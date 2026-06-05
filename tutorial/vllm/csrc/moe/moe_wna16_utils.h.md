# moe_wna16_utils.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/moe/moe_wna16_utils.h`
- **Repository**: vllm-project/vllm
- **Purpose**: Defines helper types and utilities for weight-only MoE kernels. / 定义仅权重 MoE 内核使用的辅助类型与工具函数。

## Line-by-Line Analysis / 逐行分析
### Preamble and includes (lines 2-3)
```cpp
#include <cuda_fp16.h>
#include <cuda_bf16.h>
```
**EN:** This opening block pulls in the headers, feature macros, and compile-time aliases that the rest of the file depends on.
**CN:** 这一开头代码块引入了后续实现依赖的头文件、特性宏以及编译期别名。

### Class: ScalarType (lines 9-41)
```cpp
class ScalarType<half> {
 public:
  using scalar_t = half;
  using scalar_t2 = half2;

  static __device__ float inline num2float(const half x) {
    return __half2float(x);
  }

  static __device__ half2 inline num2num2(const half x) {
    return __half2half2(x);
  }

  static __device__ half2 inline nums2num2(const half x1, const half x2) {
// ...
    return __float22half2_rn(x);
  }
};
```
**EN:** This type defines shared state, data layout, or a reusable helper abstraction for the rest of the file.
**CN:** 该类型定义了共享状态、数据布局或供本文件其他部分复用的辅助抽象。

### Class: ScalarType (lines 44-79)
```cpp
class ScalarType<nv_bfloat16> {
 public:
  using scalar_t = nv_bfloat16;
  using scalar_t2 = nv_bfloat162;

#if defined(__CUDA_ARCH__) && __CUDA_ARCH__ >= 800
  static __device__ float inline num2float(const nv_bfloat16 x) {
    return __bfloat162float(x);
  }

  static __device__ nv_bfloat162 inline num2num2(const nv_bfloat16 x) {
    return __bfloat162bfloat162(x);
  }

// ...
  }
#endif
};
```
**EN:** This type defines shared state, data layout, or a reusable helper abstraction for the rest of the file.
**CN:** 该类型定义了共享状态、数据布局或供本文件其他部分复用的辅助抽象。

### Function / Kernel: dequant (lines 99-100)
```cpp
template <typename scalar_t2, int bit>
__device__ inline void dequant(int q, scalar_t2* res) {}
```
**EN:** This function quantizes or dequantizes expert data, typically preparing compact low-precision buffers for faster compute.
**CN:** 该函数对专家数据进行量化或反量化，通常用于准备更紧凑的低精度缓冲区以提升计算效率。

### Function / Kernel: template <> __device__ inline void dequant<half2, 4>(int q,  (lines 102-127)
```cpp
template <>
__device__ inline void dequant<half2, 4>(int q, half2* res) {
  const int LO = 0x000f000f;
  const int HI = 0x00f000f0;
  const int EX = 0x64006400;
  const int SUB = 0x64006400;
  const int MUL = 0x2c002c00;
  const int ADD = 0xd400d400;

  int lo0 = lop3<(0xf0 & 0xcc) | 0xaa>(q, LO, EX);
  int hi0 = lop3<(0xf0 & 0xcc) | 0xaa>(q, HI, EX);
  q >>= 8;
  int lo1 = lop3<(0xf0 & 0xcc) | 0xaa>(q, LO, EX);
  int hi1 = lop3<(0xf0 & 0xcc) | 0xaa>(q, HI, EX);
// ...
                   *reinterpret_cast<const half2*>(&MUL),
                   *reinterpret_cast<const half2*>(&ADD));
}
```
**EN:** This helper converts data between storage formats while preserving the layout assumptions expected by later compute kernels.
**CN:** 该辅助函数在不同存储格式之间进行转换，同时保持后续计算内核所需的数据布局假设。

### Function / Kernel: template <> __device__ inline void dequant<half2, 8>(int q,  (lines 129-144)
```cpp
template <>
__device__ inline void dequant<half2, 8>(int q, half2* res) {
  static constexpr uint32_t mask_for_elt_01 = 0x5250;
  static constexpr uint32_t mask_for_elt_23 = 0x5351;
  static constexpr uint32_t start_byte_for_fp16 = 0x64646464;

  uint32_t lo = prmt<start_byte_for_fp16, mask_for_elt_01>(q);
  uint32_t hi = prmt<start_byte_for_fp16, mask_for_elt_23>(q);

  static constexpr uint32_t I8s_TO_F16s_MAGIC_NUM = 0x64006400;

  res[0] = __hsub2(*reinterpret_cast<half2*>(&lo),
                   *reinterpret_cast<const half2*>(&I8s_TO_F16s_MAGIC_NUM));
  res[1] = __hsub2(*reinterpret_cast<half2*>(&hi),
                   *reinterpret_cast<const half2*>(&I8s_TO_F16s_MAGIC_NUM));
}
```
**EN:** This helper converts data between storage formats while preserving the layout assumptions expected by later compute kernels.
**CN:** 该辅助函数在不同存储格式之间进行转换，同时保持后续计算内核所需的数据布局假设。

### Function / Kernel: template <> __device__ inline void dequant<nv_bfloat162, 4>( (lines 147-175)
```cpp
template <>
__device__ inline void dequant<nv_bfloat162, 4>(int q, nv_bfloat162* res) {
  static constexpr uint32_t MASK = 0x000f000f;
  static constexpr uint32_t EX = 0x43004300;

  int lo0 = lop3<(0xf0 & 0xcc) | 0xaa>(q, MASK, EX);
  q >>= 4;
  int hi0 = lop3<(0xf0 & 0xcc) | 0xaa>(q, MASK, EX);
  q >>= 4;
  int lo1 = lop3<(0xf0 & 0xcc) | 0xaa>(q, MASK, EX);
  q >>= 4;
  int hi1 = lop3<(0xf0 & 0xcc) | 0xaa>(q, MASK, EX);

  static constexpr uint32_t MUL = 0x3F803F80;
// ...
                   *reinterpret_cast<const nv_bfloat162*>(&MUL),
                   *reinterpret_cast<const nv_bfloat162*>(&ADD));
}
```
**EN:** This helper converts data between storage formats while preserving the layout assumptions expected by later compute kernels.
**CN:** 该辅助函数在不同存储格式之间进行转换，同时保持后续计算内核所需的数据布局假设。

### Function / Kernel: template <> __device__ inline void dequant<nv_bfloat162, 8>( (lines 177-199)
```cpp
template <>
__device__ inline void dequant<nv_bfloat162, 8>(int q, nv_bfloat162* res) {
  float fp32_intermediates[4];
  uint32_t* fp32_intermediates_casted =
      reinterpret_cast<uint32_t*>(fp32_intermediates);

  static constexpr uint32_t fp32_base = 0x4B000000;
  fp32_intermediates_casted[0] = __byte_perm(q, fp32_base, 0x7650);
  fp32_intermediates_casted[1] = __byte_perm(q, fp32_base, 0x7652);
  fp32_intermediates_casted[2] = __byte_perm(q, fp32_base, 0x7651);
  fp32_intermediates_casted[3] = __byte_perm(q, fp32_base, 0x7653);

  fp32_intermediates[0] -= 8388608.f;
  fp32_intermediates[1] -= 8388608.f;
// ...
  bf16_result_ptr[1] = __byte_perm(fp32_intermediates_casted[2],
                                   fp32_intermediates_casted[3], 0x7632);
}
```
**EN:** This helper converts data between storage formats while preserving the layout assumptions expected by later compute kernels.
**CN:** 该辅助函数在不同存储格式之间进行转换，同时保持后续计算内核所需的数据布局假设。

## Key Concepts / 关键概念
- Mixture-of-Experts routing / 混合专家路由
- Weight-only quantization / 仅权重量化
- CUDA programming model / CUDA 编程模型
- Template-based specialization / 基于模板的专用化

## Dependencies / 依赖关系
- **External libraries / 外部库**: CUDA runtime / CUDA headers
- **Runtime coupling / 运行时耦合**: Uses templates to specialize code paths at compile time / 通过模板在编译期专用化代码路径
