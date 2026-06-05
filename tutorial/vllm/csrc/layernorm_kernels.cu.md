# layernorm_kernels.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/layernorm_kernels.cu`
- **Repository**: `vllm-project/vllm`
- **Purpose**: [EN] Implements CUDA RMSNorm and fused residual-add + RMSNorm kernels, including vectorized fast paths for FP16/BF16 and host launch logic. / [CN] 实现 CUDA RMSNorm 与 fused residual-add + RMSNorm 内核，并包含 FP16/BF16 的向量化快速路径以及主机侧启动逻辑。

## Line-by-Line Analysis / 逐行分析

### Includes and dispatch helpers / 头文件与分发辅助
```cpp
#include "type_convert.cuh"
#include "dispatch_utils.h"
#include "cub_helpers.h"
#include "core/batch_invariant.hpp"
#include "libtorch_stable/quantization/vectorization_utils.cuh"
```
**EN:** These headers provide type conversion helpers, macro-based dtype/vector-size dispatch, CUB reductions, the batch-invariant runtime check, and aligned vectorized reads. Together they define the performance-critical infrastructure used by the kernels below.
**CN:** 这些头文件提供类型转换辅助、基于宏的数据类型/向量宽度分发、CUB 归约、batch-invariant 运行时检查以及对齐的向量化读取能力。它们共同构成下面内核所依赖的性能基础设施。

### RMSNorm input row selection / RMSNorm 输入行定位
```cpp
template <typename scalar_t, int VEC_SIZE, int NUM_DIMS>
__global__ void rms_norm_kernel(
    scalar_t* __restrict__ out,
    const scalar_t* __restrict__ input,
    const int64_t input_stride_d2,
    const int64_t input_stride_d3,
    const int64_t input_stride_d4,
    const int64_t input_shape_d2,
    const int64_t input_shape_d3,
    const scalar_t* __restrict__ weight,
    const float epsilon, const int num_tokens, const int hidden_size) {
  __shared__ float s_variance;
  float variance = 0.0f;
  const scalar_t* input_row;
  if constexpr (NUM_DIMS == 2) {
    input_row = input + blockIdx.x * input_stride_d2;
  } else if constexpr (NUM_DIMS == 3) {
    int batch_idx = blockIdx.x / input_shape_d2;
    int head_idx = blockIdx.x % input_shape_d2;
    input_row =
        input + batch_idx * input_stride_d3 + head_idx * input_stride_d2;
  } else if constexpr (NUM_DIMS == 4) {
    int batch_idx = blockIdx.x / (input_shape_d3 * input_shape_d2);
    int remaining = blockIdx.x % (input_shape_d3 * input_shape_d2);
    int seq_idx = remaining / input_shape_d2;
    int head_idx = remaining % input_shape_d2;
    input_row = input + batch_idx * input_stride_d4 +
                seq_idx * input_stride_d3 + head_idx * input_stride_d2;
  }
```
**EN:** A single templated kernel handles 2D, 3D, and 4D tensor layouts. `blockIdx.x` identifies one logical token/head row, and compile-time `NUM_DIMS` selects the correct stride math without runtime branching inside the generated specialization.
**CN:** 该模板内核统一处理 2D、3D 和 4D 张量布局。`blockIdx.x` 对应一个逻辑 token/head 行，编译期常量 `NUM_DIMS` 决定具体步长计算方式，从而避免在生成后的特化内核中出现额外运行时分支。

### Vectorized variance reduction / 向量化方差归约
```cpp
  auto vec_op = [&variance](const vec_n_t<scalar_t, VEC_SIZE>& vec) {
#pragma unroll
    for (int i = 0; i < VEC_SIZE; ++i) {
      float x = static_cast<float>(vec.val[i]);
      variance += x * x;
    }
  };
  auto scalar_op = [&variance](const scalar_t& val) {
    float x = static_cast<float>(val);
    variance += x * x;
  };
  vllm::vectorize_read_with_alignment<VEC_SIZE>(
      input_row, hidden_size, threadIdx.x, blockDim.x, vec_op, scalar_op);

  using BlockReduce = cub::BlockReduce<float, 1024>;
  __shared__ typename BlockReduce::TempStorage reduceStore;
  variance = BlockReduce(reduceStore).Reduce(variance, CubAddOp{}, blockDim.x);

  if (threadIdx.x == 0) {
    s_variance = rsqrtf(variance / hidden_size + epsilon);
  }
```
**EN:** Each thread accumulates a local sum of squares, preferably through aligned vector loads. CUB then reduces those partial sums to a block-wide value, and thread 0 converts it into the RMS scaling factor `1/sqrt(mean(x^2)+eps)` stored in shared memory.
**CN:** 每个线程先累加本地平方和，并尽量通过对齐向量加载提升吞吐。随后使用 CUB 将各线程部分和归约为 block 级总和，再由线程 0 计算 RMS 缩放系数 `1/sqrt(mean(x^2)+eps)` 并存入共享内存。

### RMSNorm output writeback / RMSNorm 输出写回
```cpp
  scalar_t* out_row = out + blockIdx.x * hidden_size;
  auto* v_in = reinterpret_cast<const vec_n_t<scalar_t, VEC_SIZE>*>(input_row);
  auto* v_w = reinterpret_cast<const vec_n_t<scalar_t, VEC_SIZE>*>(weight);
  auto* v_out = reinterpret_cast<vec_n_t<scalar_t, VEC_SIZE>*>(out_row);
  for (int i = threadIdx.x; i < hidden_size / VEC_SIZE; i += blockDim.x) {
    vec_n_t<scalar_t, VEC_SIZE> dst;
    vec_n_t<scalar_t, VEC_SIZE> src1 = v_in[i];
    vec_n_t<scalar_t, VEC_SIZE> src2 = v_w[i];
#pragma unroll
    for (int j = 0; j < VEC_SIZE; j++) {
      float x = static_cast<float>(src1.val[j]);
      float w = static_cast<float>(src2.val[j]);
      dst.val[j] = static_cast<scalar_t>(x * s_variance * w);
    }
    v_out[i] = dst;
  }
}
```
**EN:** After the normalization factor is known, the kernel multiplies input values by both the RMS scale and learned weight. The vector reinterpret casts keep the final pass bandwidth-efficient.
**CN:** 在归一化因子确定后，内核将输入同时乘以 RMS 缩放系数和可学习权重。这里通过向量类型重解释来维持最终写回阶段的带宽效率。

### Vectorized fused residual + RMSNorm path / 向量化 fused residual + RMSNorm 路径
```cpp
template <typename scalar_t, int width>
__global__ std::enable_if_t<(width > 0) && _typeConvert<scalar_t>::exists>
fused_add_rms_norm_kernel(
    scalar_t* __restrict__ input,
    const int64_t input_stride,
    scalar_t* __restrict__ residual,
    const scalar_t* __restrict__ weight,
    const float epsilon, const int num_tokens, const int hidden_size) {
  static_assert(std::is_pod_v<_f16Vec<scalar_t, width>>);
  static_assert(sizeof(_f16Vec<scalar_t, width>) == sizeof(scalar_t) * width);

  const int vec_hidden_size = hidden_size / width;
  const int64_t vec_input_stride = input_stride / width;
  __shared__ float s_variance;
  float variance = 0.0f;
  auto* __restrict__ input_v =
      reinterpret_cast<_f16Vec<scalar_t, width>*>(input);
  auto* __restrict__ residual_v =
      reinterpret_cast<_f16Vec<scalar_t, width>*>(residual);
  auto* __restrict__ weight_v =
      reinterpret_cast<const _f16Vec<scalar_t, width>*>(weight);
```
**EN:** This specialization is selected for FP16/BF16-like types that support `_typeConvert`. It loads packed vectors, adds the residual into `input`, accumulates squared sums from the packed data, writes the summed residual back, and then emits normalized results using vector math.
**CN:** 这个特化用于支持 `_typeConvert` 的 FP16/BF16 类类型。它以打包向量方式读取数据，把 residual 加到 `input` 上，对打包数据累加平方和，将相加后的结果写回 residual，再用向量化计算输出归一化结果。

### Generic fallback specialization / 通用回退特化
```cpp
template <typename scalar_t, int width>
__global__ std::enable_if_t<(width == 0) || !_typeConvert<scalar_t>::exists>
fused_add_rms_norm_kernel(
    scalar_t* __restrict__ input,
    const int64_t input_stride,
    scalar_t* __restrict__ residual,
    const scalar_t* __restrict__ weight,
    const float epsilon, const int num_tokens, const int hidden_size) {
  __shared__ float s_variance;
  float variance = 0.0f;

  for (int idx = threadIdx.x; idx < hidden_size; idx += blockDim.x) {
    scalar_t z = input[blockIdx.x * input_stride + idx];
    z += residual[blockIdx.x * hidden_size + idx];
    float x = (float)z;
    variance += x * x;
    residual[blockIdx.x * hidden_size + idx] = z;
  }
```
**EN:** When vector packing is not possible, the kernel falls back to scalar iteration. The algorithm stays the same—residual addition, variance accumulation, block reduction, and weighted normalization—but loses the memory coalescing advantage of packed loads/stores.
**CN:** 当无法进行向量打包时，内核退回到标量循环。算法流程保持不变：先做 residual 相加、再做方差累加和 block 归约、最后执行带权重的归一化，只是失去了打包读写带来的更好内存访问效率。

### Host-side launch policy / 主机侧启动策略
```cpp
void rms_norm(torch::Tensor& out, torch::Tensor& input,
              torch::Tensor& weight, double epsilon) {
  TORCH_CHECK(out.is_contiguous());
  if (input.stride(-1) != 1) {
    input = input.contiguous();
  }
  ...
  const int max_block_size = (num_tokens < 256) ? 1024 : 256;
  ...
  VLLM_DISPATCH_RANK234(num_dims, [&] {
    VLLM_DISPATCH_FLOATING_TYPES(input.scalar_type(), "rms_norm_kernel", [&] {
      const int calculated_vec_size =
          std::gcd(16 / sizeof(scalar_t), hidden_size);
```
**EN:** The wrapper enforces layout assumptions, selects a smaller block size for large token counts to improve occupancy, dispatches on tensor rank and scalar type, and computes the best vector width from alignment-compatible 128-bit access granularity.
**CN:** 该包装函数负责保证布局前提、在 token 数较大时选用更小 block 以提升占用率，并根据张量维度与数据类型分发内核；同时依据 128-bit 对齐访问能力计算合适的向量宽度。

### Alignment-gated fused launch / 基于对齐条件的 fused 启动
```cpp
auto inp_ptr = reinterpret_cast<std::uintptr_t>(input.data_ptr());
auto res_ptr = reinterpret_cast<std::uintptr_t>(residual.data_ptr());
auto wt_ptr = reinterpret_cast<std::uintptr_t>(weight.data_ptr());
constexpr int vector_width = 8;
constexpr int req_alignment_bytes = vector_width * 2;
bool ptrs_are_aligned = inp_ptr % req_alignment_bytes == 0 &&
                        res_ptr % req_alignment_bytes == 0 &&
                        wt_ptr % req_alignment_bytes == 0;
bool offsets_are_multiple_of_vector_width =
    hidden_size % vector_width == 0 && input_stride % vector_width == 0;
bool batch_invariant_launch = vllm::vllm_is_batch_invariant();
if (ptrs_are_aligned && offsets_are_multiple_of_vector_width &&
    !batch_invariant_launch) {
  LAUNCH_FUSED_ADD_RMS_NORM(8);
} else {
  LAUNCH_FUSED_ADD_RMS_NORM(0);
}
```
**EN:** The fast fused kernel is only used when pointers and strides support 16-byte vector loads and the launch is not in a batch-invariant mode that changes execution assumptions. Otherwise the safer scalar fallback is chosen.
**CN:** 只有在指针对齐、步长满足 16 字节向量加载要求，且当前不是会改变执行假设的 batch-invariant 模式时，才会启用快速 fused 内核；否则退回更稳妥的标量版本。

## Key Concepts / 关键概念
- **EN:** RMSNorm here computes `x / sqrt(mean(x^2)+eps) * weight`; unlike LayerNorm it does not subtract the mean.  
  **CN:** 这里的 RMSNorm 计算的是 `x / sqrt(mean(x^2)+eps) * weight`；与 LayerNorm 不同，它不会减去均值。
- **EN:** Performance comes from vectorized aligned reads, CUB block reduction, and specialization for tensor ranks and dtypes.  
  **CN:** 性能主要来自对齐向量读取、CUB 的 block 归约，以及对张量维度和数据类型的特化分发。
- **EN:** The fused kernel updates `residual` in-place with `input + residual`, then writes normalized output back to `input`.  
  **CN:** fused 内核会先将 `input + residual` 原地写回 `residual`，再把归一化结果写回 `input`。

## Dependencies / 依赖关系
- **EN:** Depends on `type_convert.cuh` for packed half/bfloat helpers and conversions.  
  **CN:** 依赖 `type_convert.cuh` 提供半精度/BFloat16 的打包辅助和类型转换。
- **EN:** Depends on `dispatch_utils.h` and `vectorization_utils.cuh` for compile-time launch specialization.  
  **CN:** 依赖 `dispatch_utils.h` 与 `vectorization_utils.cuh` 完成编译期启动特化与向量化读取。
- **EN:** Uses CUB reduction utilities from `cub_helpers.h` and CUDA stream/device guards from PyTorch CUDA headers.  
  **CN:** 使用 `cub_helpers.h` 中的 CUB 归约工具，以及 PyTorch CUDA 头文件提供的设备/流守卫。
