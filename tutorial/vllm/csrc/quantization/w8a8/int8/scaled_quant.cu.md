# scaled_quant.cu — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/quantization/w8a8/int8/scaled_quant.cu`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Implements INT8 quantization kernels for static or dynamic scaling, with optional asymmetric zero-point output. / [CN] 实现 INT8 量化内核，支持静态/动态 scale，以及可选的非对称 zero-point 输出。

## Line-by-Line Analysis / 逐行分析
### Backend-specific saturating conversion helpers / 后端相关的饱和转换辅助函数
```cpp
static inline __device__ int8_t float_to_int8_rn(float x) {
#ifdef USE_ROCM
  float dst = std::nearbyint(x);
  dst = (dst < i8_min) ? i8_min : (dst > i8_max) ? i8_max : dst;
  return static_cast<int8_t>(dst);
#else
  uint32_t dst;
  asm volatile("cvt.rni.sat.s8.f32 %0, %1;" : "=r"(dst) : "f"(x));
  return reinterpret_cast<const int8_t&>(dst);
#endif
}
```
**EN:** The file starts by normalizing rounding and saturation behavior across CUDA and ROCm. CUDA uses inline conversion instructions, while ROCm emulates CUDA's round-to-nearest-even behavior with `nearbyint` plus manual clamping.
**CN:** 文件首先统一 CUDA 与 ROCm 下的舍入和饱和行为。CUDA 使用内联转换指令；ROCm 则通过 `nearbyint` 加手动裁剪来模拟 CUDA 的 round-to-nearest-even 行为。

### Static symmetric quantization / 静态对称量化
```cpp
template <typename scalar_t, typename scale_t>
__global__ void static_scaled_int8_quant_kernel(
    const scalar_t* __restrict__ input, int8_t* __restrict__ output,
    const scale_t* scale_ptr, const int hidden_size) {
  ...
  vectorize_with_alignment<16>(
      row_in, row_out, hidden_size, tid, stride,
      [=] __device__(int8_t& dst, const scalar_t& src) {
        dst = float_to_int8_rn(static_cast<float>(src) / scale);
      });
}
```
**EN:** This is the simplest path: one global scale is provided, and every value is quantized by dividing by it and saturating into INT8.
**CN:** 这是最简单的路径：给定一个全局 scale，所有值都通过除以该 scale 并饱和截断到 INT8 来完成量化。

### Static asymmetric quantization with zero point / 带 zero point 的静态非对称量化
```cpp
template <typename scalar_t, typename scale_t, typename azp_t>
__global__ void static_scaled_int8_azp_quant_kernel(
    const scalar_t* __restrict__ input, int8_t* __restrict__ output,
    const scale_t* scale_ptr, const azp_t* azp_ptr, const int hidden_size) {
  ...
  const auto v = static_cast<float>(src) * inv_s;
  dst = int32_to_int8(float_to_int32_rn(v) + azp);
}
```
**EN:** When an affine zero point is provided, the kernel rounds into INT32 first, adds the zero point, and then saturates to INT8. This preserves asymmetric quantization semantics.
**CN:** 当提供仿射 zero point 时，内核会先把值舍入到 INT32，再加上 zero point，最后饱和到 INT8，从而保留非对称量化语义。

### Dynamic symmetric quantization / 动态对称量化
```cpp
template <typename scalar_t, typename scale_t>
__global__ void dynamic_scaled_int8_quant_kernel(
    const scalar_t* __restrict__ input, int8_t* __restrict__ output,
    scale_t* scale_out, const int hidden_size) {
  ...
  float block_max = BlockReduce(tmp).Reduce(thread_max, CubMaxOp{}, blockDim.x);
  ...
  scale_out[blockIdx.x] = absmax / 127.f;
  ...
}
```
**EN:** Each block handles one row, finds that row's absolute maximum via CUB block reduction, stores a per-row scale, and then quantizes with `127 / absmax` as the inverse scale.
**CN:** 每个 block 处理一行，通过 CUB block reduction 找到该行的绝对值最大值，写出逐行 scale，然后用 `127 / absmax` 作为倒数 scale 执行量化。

### `MinMax` helper for affine calibration / 用于仿射标定的 `MinMax` 辅助结构
```cpp
struct MinMax {
  float min, max;
  ...
  __host__ __device__ MinMax& operator+=(float v) {
    min = fminf(min, v);
    max = fmaxf(max, v);
    return *this;
  }
```
**EN:** `MinMax` is a compact reduction payload that tracks both row minimum and maximum at once. The overloaded operators make it compatible with block-level reduction patterns.
**CN:** `MinMax` 是一个紧凑的归约载体，可同时追踪一行的最小值和最大值。重载运算符让它可以直接嵌入 block 级归约模式。

### Dynamic asymmetric quantization / 动态非对称量化
```cpp
template <typename scalar_t, typename scale_t, typename azp_t>
__global__ void dynamic_scaled_int8_azp_quant_kernel(
    const scalar_t* __restrict__ input, int8_t* __restrict__ output,
    scale_t* scale_out, azp_t* azp_out, const int hidden_size) {
  ...
  float s = (mm.max - mm.min) / 255.f;
  float zp = nearbyintf(-128.f - mm.min / s);
  scale_out[blockIdx.x] = s;
  azp_out[blockIdx.x] = azp_sh;
}
```
**EN:** This path calibrates each row using its min/max range, producing both a per-row scale and a per-row asymmetric zero point before quantization.
**CN:** 该路径使用每一行的最小/最大范围进行标定，在量化前为每一行生成一个 scale 和一个非对称 zero point。

### PyTorch-facing launchers / 面向 PyTorch 的启动器
```cpp
void static_scaled_int8_quant(torch::Tensor& out,
                              torch::Tensor const& input,
                              torch::Tensor const& scale,
                              std::optional<torch::Tensor> const& azp) {
  ...
  VLLM_DISPATCH_FLOATING_TYPES(
      input.scalar_type(), "static_scaled_int8_quant_kernel", [&] {
        if (!azp) {
          vllm::static_scaled_int8_quant_kernel<scalar_t, float><<<...>>>(...);
        } else {
          vllm::static_scaled_int8_azp_quant_kernel<scalar_t, float, int32_t><<<...>>>(...);
        }
      });
}
```
**EN:** The host functions validate contiguity and shape assumptions, choose block/grid sizes, guard the correct CUDA device, and dispatch to the symmetric or asymmetric kernel variant.
**CN:** 主机侧函数负责检查连续性与形状假设、选择 block/grid 大小、保护正确的 CUDA 设备，并分发到对称或非对称内核版本。

## Key Concepts / 关键概念
- **Cross-backend numerical consistency**: helper functions make CUDA and ROCm round/saturate in comparable ways. / **跨后端数值一致性**：辅助函数让 CUDA 与 ROCm 在舍入/饱和行为上尽量保持一致。
- **Per-row dynamic calibration**: dynamic paths compute scale (and optionally zero point) from each row's statistics. / **逐行动态标定**：动态路径会从每一行的统计量中计算 scale（以及可选 zero point）。
- **Vectorized quantization**: all kernels use 128-bit aligned vector helpers where possible. / **向量化量化**：所有内核在可能时都使用 128-bit 对齐向量化辅助函数。

## Dependencies / 依赖关系
- `dispatch_utils.h` provides floating-type dispatch macros. / `dispatch_utils.h` 提供浮点类型分发宏。
- `libtorch_stable/quantization/vectorization_utils.cuh` provides aligned vector load/store helpers used by every kernel. / `libtorch_stable/quantization/vectorization_utils.cuh` 提供所有内核共用的对齐向量读写辅助函数。
- `cub_helpers.h` supplies CUB reduction helpers such as `CubMaxOp` and `BlockReduce`. / `cub_helpers.h` 提供 `CubMaxOp`、`BlockReduce` 等 CUB 归约辅助工具。
