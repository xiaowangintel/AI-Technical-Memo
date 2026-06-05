# nvfp4_utils.cuh — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/libtorch_stable/quantization/fp4/nvfp4_utils.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: Defines reusable NVFP4 helper routines for shape math, FP4 packing, scale placement, and fused elementwise preprocessing. / 定义可复用的 NVFP4 辅助函数，用于形状计算、FP4 打包、尺度写回以及融合逐元素预处理。

## Line-by-Line Analysis / 逐行分析
### Compile-time selection of elements per thread
```cpp
#if defined(NVFP4_ENABLE_ELTS16) && defined(CUDA_VERSION) && \
    CUDA_VERSION >= 12090
  #define ELTS_PER_THREAD 16
constexpr int CVT_FP4_ELTS_PER_THREAD = 16;
constexpr bool CVT_FP4_PACK16 = true;
#else
  #define ELTS_PER_THREAD 8
constexpr int CVT_FP4_ELTS_PER_THREAD = 8;
```
**EN:** This preprocessor block chooses whether FP4 conversion runs with 8 or 16 elements per thread, depending on compile-time flags and CUDA version support.
**CN:** 该预处理块会根据编译期开关和 CUDA 版本支持情况，决定 FP4 转换是每线程处理 8 个还是 16 个元素。

### Swizzled scale-tensor shape computation
```cpp
inline std::pair<int64_t, int64_t> computeSwizzledSFShape(int64_t m,
                                                          int64_t n) {
  int64_t rounded_m = round_up(m, static_cast<int64_t>(128));
  int64_t scale_n = n / CVT_FP4_SF_VEC_SIZE;
  int64_t rounded_n = round_up(scale_n, static_cast<int64_t>(4));
  return {rounded_m, rounded_n / 4};
}
```
**EN:** The helper rounds the row dimension to hardware-friendly tiles and converts the column dimension into scale-vector units, producing the swizzled scale-tensor shape.
**CN:** 该辅助函数会把行维度向硬件友好的 tile 对齐，并将列维度转换成尺度向量单位，从而得到 swizzled 尺度张量的形状。

### Inline PTX packing from FP32 vectors to E2M1
```cpp
inline __device__ uint32_t fp32_vec8_to_e2m1(float (&array)[8]) {
  uint32_t val;
  asm volatile(
      "{\n"
      ".reg .b8 byte0;\n"
      ".reg .b8 byte1;\n"
      ".reg .b8 byte2;\n"
      ".reg .b8 byte3;\n"
      "cvt.rn.satfinite.e2m1x2.f32   byte0, %2, %1;\n"
```
**EN:** This low-level routine uses inline PTX to pack eight FP32 values into the compact E2M1 representation, forming the basis of NVFP4 payload generation.
**CN:** 这个底层例程使用内联 PTX 将 8 个 FP32 值打包为紧凑的 E2M1 表示，是 NVFP4 载荷生成的基础。

### Addressing helper for swizzled scale output
```cpp
  int32_t mTileIdx = mIdx >> 7;         // mIdx / 128
  int32_t outerMIdx = mIdx & 31;        // mIdx % 32
  int32_t innerMIdx = (mIdx >> 5) & 3;  // (mIdx / 32) % 4
  int32_t kTileIdx = kIdx >> 2;         // kIdx / 4
  int32_t innerKIdx = kIdx & 3;         // kIdx % 4

  // Compute global SF offset: mTileIdx * (numKTiles * 512) + kTileIdx * 512 +
```
**EN:** The scale-address helper decomposes an `(m, n)` position into outer and inner tile coordinates, matching the swizzled layout expected by downstream kernels.
**CN:** 尺度地址辅助函数会把 `(m, n)` 位置拆解为外层与内层 tile 坐标，以匹配下游内核所期望的 swizzled 布局。

### Warp-level quantization and scale derivation
```cpp
  // TODO: use half as compute data type.
  float SFValue = SFScaleVal * (vecMax * reciprocal_approximate_ftz(6.0f));
  // 8 bits representation of the SF.
  uint8_t fp8SFVal;
  // Write the SF to global memory (STG.8).
  if constexpr (UE8M0_SF) {
    // Extract the 8 exponent bits from float32.
    // float 32bits = 1 sign bit + 8 exponent bits + 23 mantissa bits.
    uint32_t tmp = reinterpret_cast<uint32_t&>(SFValue) >> 23;
    fp8SFVal = tmp & 0xff;
```
**EN:** At warp scope, the converter computes the maximum magnitude, derives the shared scale factor, writes the scale in FP8-compatible form, and then packs the rescaled values into FP4.
**CN:** 在 warp 范围内，转换器会计算最大幅值、推导共享尺度因子、以兼容 FP8 的形式写出尺度，然后把重缩放后的值打包为 FP4。

### Fused SiLU helper for gate-up activations
```cpp

__device__ __forceinline__ float2 silu2(float2 x) {
  return make_float2(silu(x.x), silu(x.y));
}

template <class Type>
__inline__ __device__ PackedVec<Type, CVT_FP4_PACK16> compute_silu_mul(
    const PackedVec<Type, CVT_FP4_PACK16>& x_vec,
    const PackedVec<Type, CVT_FP4_PACK16>& y_vec) {
  PackedVec<Type, CVT_FP4_PACK16> result;
```
**EN:** The tail of the header defines scalar and vectorized SiLU helpers plus `compute_silu_mul`, which are reused by fused quantization kernels handling gate-and-up activations.
**CN:** 头文件尾部定义了标量与向量化的 SiLU 辅助函数以及 `compute_silu_mul`，供处理 gate 与 up 激活的融合量化内核复用。

## Key Concepts / 关键概念
- Compile-time control over packing granularity / 对打包粒度的编译期控制
- Swizzled scale-factor layout math / swizzled 尺度因子布局计算
- Low-level FP4 packing and fused activation helpers / 低层 FP4 打包与融合激活辅助函数

## Dependencies / 依赖关系
- `cuda_runtime.h` and `cuda_fp8.h` for device intrinsics and FP8 scale handling / 通过 `cuda_runtime.h` 与 `cuda_fp8.h` 使用设备内建函数和 FP8 尺度处理
- `cuda_vec_utils.cuh` for vector converters and packed types / 通过 `cuda_vec_utils.cuh` 获取向量转换器与打包类型
- Consumers across all FP4 quantization and GEMM kernels in this directory / 被本目录中的所有 FP4 量化与 GEMM 内核共同依赖
