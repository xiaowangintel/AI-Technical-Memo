# per_token_group_quant.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/libtorch_stable/quantization/w8a8/fp8/per_token_group_quant.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Implements FP8 per-token group quantization for GPU acceleration, providing both shared-memory and register-resident kernel variants for efficient quantization. / [CN] 实现 FP8 每令牌分组量化的 GPU 加速，提供共享内存和寄存器驻留内核变体以实现高效量化。

## Line-by-Line Analysis / 逐行分析

### Include Headers / 头文件包含
```cpp
#include <torch/csrc/stable/tensor.h>
#include <torch/csrc/stable/ops.h>
#include <cuda_fp8.h>
#include "libtorch_stable/quantization/w8a8/per_token_group_quant_8bit.h"
```
**EN:** Imports PyTorch stable ABI tensor/ops APIs, CUDA FP8 types, and the shared 8-bit quantization header.  
**CN:** 导入 PyTorch 稳定 ABI 张量/操作接口、CUDA FP8 类型以及共享的 8 位量化头文件。

### GroupReduceMax Function / 分组最大值归约函数
```cpp
__device__ __forceinline__ float GroupReduceMax(float val) {
  unsigned mask = threadIdx.x % 32 >= 16 ? 0xffff0000 : 0x0000ffff;
  val = fmaxf(val, __shfl_xor_sync(mask, val, 8));
  val = fmaxf(val, __shfl_xor_sync(mask, val, 4));
  val = fmaxf(val, __shfl_xor_sync(mask, val, 2));
  val = fmaxf(val, __shfl_xor_sync(mask, val, 1));
  return val;
}
```
**EN:** Warp-level reduction to find maximum value within a 16-thread subgroup using butterfly shuffle operations for efficient parallel max computation.  
**CN:** Warp 级归约，使用蝴蝶洗牌操作在 16 线程子组内查找最大值，实现高效的并行最大值计算。

### ComputeGroupScale Template / 计算分组缩放因子模板
```cpp
template <typename T, bool SCALE_UE8M0>
__device__ __forceinline__ float ComputeGroupScale(
    const T* __restrict__ group_input, T* __restrict__ smem_group,
    const int group_size, const int lane_id, const int threads_per_group,
    const float eps, const float max_8bit) {
  float local_absmax = eps;
  // Vectorized copy from global to shared memory + compute absmax
  auto scalar_op_cache = [&] __device__(T & dst, const T& src) {
    float abs_v = fabsf(static_cast<float>(src));
    local_absmax = fmaxf(local_absmax, abs_v);
    dst = src;
  };
  vllm::vectorize_with_alignment<vec_size>(
      group_input, smem_group, group_size, lane_id, 
      threads_per_group, scalar_op_cache);
  local_absmax = GroupReduceMax(local_absmax);
  float y_s = local_absmax / max_8bit;
  if constexpr (SCALE_UE8M0) {
    y_s = exp2f(ceilf(log2f(fmaxf(fabsf(y_s), 1e-10f))));
  }
  return y_s;
}
```
**EN:** Computes quantization scale by finding absmax in the group using vectorized loads to shared memory, then optionally converts to UE8M0 format (power-of-2 scale).  
**CN:** 通过向量化加载到共享内存查找分组中的绝对最大值来计算量化缩放因子，然后可选地转换为 UE8M0 格式（2 的幂次缩放）。

### QuantizeGroup Function / 量化分组函数
```cpp
template <typename T, typename DST_DTYPE>
__device__ __forceinline__ void QuantizeGroup(
    const T* __restrict__ smem_group, DST_DTYPE* __restrict__ group_output,
    const int group_size, const int lane_id, const int threads_per_group,
    const float y_s, const float min_8bit, const float max_8bit) {
  auto scalar_op_quant = [&] __device__(DST_DTYPE & dst, const T& src) {
    float q = fminf(fmaxf(static_cast<float>(src) / y_s, min_8bit), max_8bit);
    dst = DST_DTYPE(q);
  };
  vllm::vectorize_with_alignment<vec_size>(
      smem_group, group_output, group_size, lane_id,
      threads_per_group, scalar_op_quant);
}
```
**EN:** Quantizes values from shared memory to global memory using computed scale, clamping to min/max 8-bit range with vectorized stores.  
**CN:** 使用计算的缩放因子将共享内存中的值量化到全局内存，通过向量化存储将值钳制到最小/最大 8 位范围。

### Shared-Memory Kernel / 共享内存内核
```cpp
template <typename T, typename DST_DTYPE, bool IS_COLUMN_MAJOR, bool SCALE_UE8M0>
__global__ void per_token_group_quant_8bit_kernel(
    const T* __restrict__ input, void* __restrict__ output_q,
    scale_packed_t* __restrict__ output_s, const int group_size,
    const int num_groups, const int groups_per_block, const float eps,
    const float min_8bit, const float max_8bit, ...) {
  const int threads_per_group = 16;
  const int64_t local_group_id = threadIdx.x / threads_per_group;
  const int lane_id = threadIdx.x % threads_per_group;
  
  extern __shared__ __align__(16) char smem_raw[];
  T* smem_group = reinterpret_cast<T*>(smem_raw) + local_group_id * group_size;
  
  const float y_s = ComputeGroupScale<T, SCALE_UE8M0>(
      group_input, smem_group, group_size, lane_id, 
      threads_per_group, eps, max_8bit);
  
  if (lane_id == 0) {
    *scale_output = y_s;
  }
  __syncthreads();
  
  QuantizeGroup<T, DST_DTYPE>(smem_group, group_output, group_size, 
                              lane_id, threads_per_group, y_s, min_8bit, max_8bit);
}
```
**EN:** Main kernel using shared memory to cache group data, avoiding double DRAM reads. Each block processes multiple groups (1-16) simultaneously with 16 threads per group.  
**CN:** 使用共享内存缓存分组数据的主内核，避免双重 DRAM 读取。每个块同时处理多个分组（1-16），每组 16 个线程。

### Register-Resident Fast Path / 寄存器驻留快速路径
```cpp
template <typename T, typename DST_DTYPE, int GROUP_SIZE, 
          int kGroupsPerBlockX, int kRowsPerBlock>
__global__ void per_token_group_quant_8bit_packed_register_kernel(
    const T* __restrict__ input, void* __restrict__ output_q,
    unsigned int* __restrict__ output_s_packed, ...) {
  static_assert(GROUP_SIZE == 128, "fast path supports GROUP_SIZE==128");
  constexpr int THREADS_PER_GROUP = 8;
  constexpr int VEC_SIZE = 32 / sizeof(T);  // 16 for bf16/fp16
  
  // Load 16 elements (32B) into registers as two uint4 loads
  alignas(16) T regs[VEC_SIZE];
  float local_absmax = eps;
  if (is_valid_group) {
    uint4* dst = reinterpret_cast<uint4*>(&regs[0]);
    const uint4* src = reinterpret_cast<const uint4*>(group_input);
    dst[0] = src[0];
    dst[1] = src[1];
    #pragma unroll
    for (int i = 0; i < VEC_SIZE; ++i) {
      float v = fabsf(static_cast<float>(regs[i]));
      local_absmax = fmaxf(local_absmax, v);
    }
  }
  
  // 8-lane shuffle reduce
  unsigned mask = 0xffu << (threadIdx.x & 24u);
  local_absmax = fmaxf(local_absmax, __shfl_xor_sync(mask, local_absmax, 4));
  local_absmax = fmaxf(local_absmax, __shfl_xor_sync(mask, local_absmax, 2));
  local_absmax = fmaxf(local_absmax, __shfl_xor_sync(mask, local_absmax, 1));
  
  // Compute UE8M0 scale via bit math (bit-exact with exp2f(ceilf(log2f)))
  float y_s = local_absmax / max_8bit;
  uint32_t bits = __float_as_uint(y_s);
  uint32_t exp_bits = (bits >> 23) & 0xffu;
  uint32_t mant_bits = bits & 0x7fffffu;
  uint8_t exp_byte = static_cast<uint8_t>(exp_bits + (mant_bits != 0u ? 1u : 0u));
  
  // Quantize and pack 16 bytes into uint4
  float inv_y = 1.0f / __uint_as_float(static_cast<uint32_t>(exp_byte) << 23);
  // ... pack into uint4 and write
}
```
**EN:** Optimized kernel for group_size=128 keeping all 16 elements per thread in registers throughout the pipeline. Uses bit manipulation for UE8M0 scale computation and fuses two 128-bit loads into a 256-bit load on Blackwell.  
**CN:** 针对 group_size=128 优化的内核，在整个流水线中将每个线程的所有 16 个元素保存在寄存器中。使用位操作计算 UE8M0 缩放因子，并在 Blackwell 上将两个 128 位加载融合为 256 位加载。

### Entry Point Functions / 入口函数
```cpp
void per_token_group_quant_8bit(const torch::stable::Tensor& input,
                                torch::stable::Tensor& output_q,
                                torch::stable::Tensor& output_s,
                                int64_t group_size, double eps, double min_8bit,
                                double max_8bit, bool scale_ue8m0) {
  // Launch shared-memory kernel with dynamic groups_per_block
  const int groups_per_block = GetGroupsPerBlock(num_groups);
  VLLM_STABLE_DISPATCH_FLOATING_TYPES(input.scalar_type(), ..., ([&] {
    if (dst_type == torch::headeronly::ScalarType::Float8_e4m3fn) {
      LAUNCH_KERNEL(scalar_t, __nv_fp8_e4m3);
    } else if (dst_type == torch::headeronly::ScalarType::Char) {
      LAUNCH_KERNEL(scalar_t, int8_t);
    }
  }));
}

void per_token_group_quant_8bit_packed(const torch::stable::Tensor& input,
                                       torch::stable::Tensor& output_q,
                                       torch::stable::Tensor& output_s_packed,
                                       int64_t group_size, double eps,
                                       double min_8bit, double max_8bit) {
  STD_TORCH_CHECK(group_size == 128, "only supports group_size==128");
  STD_TORCH_CHECK(in_dtype == Half || in_dtype == BFloat16, 
                  "only supports bf16/fp16 input");
  // Launch register-resident kernel with kGroupsPerBlockX and kRowsPerBlock
  VLLM_STABLE_DISPATCH_HALF_TYPES(...);
}

void per_token_group_quant_fp8(...) {
  per_token_group_quant_8bit(input, output_q, output_s, group_size, eps,
                             fp8_min, fp8_max, scale_ue8m0);
}
```
**EN:** Public API functions dispatching to appropriate kernel variant based on input shape and constraints. The packed variant enforces group_size=128 and half-precision input.  
**CN:** 公共 API 函数根据输入形状和约束分派到适当的内核变体。打包变体强制要求 group_size=128 和半精度输入。

## Key Concepts / 关键概念

### EN:
- **Per-Token Group Quantization**: Divides each token's features into groups and computes independent quantization scales per group for better accuracy than per-tensor quantization
- **Dual Kernel Strategy**: Shared-memory kernel for general case vs. register-resident kernel for the high-performance group_size=128 case
- **UE8M0 Scale Format**: Unsigned 8-bit exponent-only format (power-of-2 scales) for TMA-aligned packed scales used by DeepGEMM on Blackwell
- **Vectorized Operations**: Uses 16-byte (uint4) vectorized loads/stores to maximize memory bandwidth utilization
- **Warp Shuffle Reductions**: Exploits warp-synchronous primitives for fast intra-group max-reduce without shared memory atomics

### CN:
- **每令牌分组量化**：将每个令牌的特征划分为组，并为每组计算独立的量化缩放因子，比每张量量化更准确
- **双内核策略**：通用情况使用共享内存内核，高性能 group_size=128 情况使用寄存器驻留内核
- **UE8M0 缩放格式**：无符号 8 位仅指数格式（2 的幂次缩放），用于 Blackwell 上 DeepGEMM 使用的 TMA 对齐打包缩放
- **向量化操作**：使用 16 字节（uint4）向量化加载/存储以最大化内存带宽利用率
- **Warp 洗牌归约**：利用 warp 同步原语实现快速组内最大值归约，无需共享内存原子操作

## Dependencies / 依赖关系

### EN:
- **PyTorch Stable ABI**: `torch/csrc/stable/tensor.h`, `torch/csrc/stable/ops.h` for tensor manipulation and operation registration
- **CUDA Libraries**: `cuda_fp8.h` for FP8 type `__nv_fp8_e4m3`, CUDA runtime for kernel launch
- **vLLM Quantization Framework**: 
  - `per_token_group_quant_8bit.h` - shared header declaring common interfaces
  - `vectorization.cuh`, `vectorization_utils.cuh` - vectorized memory operation helpers
  - `dispatch_utils.h` - type dispatch macros (`VLLM_STABLE_DISPATCH_FLOATING_TYPES`)
  - `torch_utils.h` - utility functions like `get_current_cuda_stream()`
- **Hardware Requirements**: CUDA-capable GPU with compute capability supporting FP8 (SM 8.9+)

### CN:
- **PyTorch 稳定 ABI**：`torch/csrc/stable/tensor.h`、`torch/csrc/stable/ops.h` 用于张量操作和操作注册
- **CUDA 库**：`cuda_fp8.h` 提供 FP8 类型 `__nv_fp8_e4m3`，CUDA 运行时用于内核启动
- **vLLM 量化框架**：
  - `per_token_group_quant_8bit.h` - 声明通用接口的共享头文件
  - `vectorization.cuh`、`vectorization_utils.cuh` - 向量化内存操作辅助工具
  - `dispatch_utils.h` - 类型分派宏（`VLLM_STABLE_DISPATCH_FLOATING_TYPES`）
  - `torch_utils.h` - 实用函数如 `get_current_cuda_stream()`
- **硬件要求**：支持 FP8 的 CUDA GPU（计算能力 SM 8.9+）
