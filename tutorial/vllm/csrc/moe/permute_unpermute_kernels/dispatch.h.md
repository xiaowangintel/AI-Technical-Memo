# dispatch.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `csrc/moe/permute_unpermute_kernels/dispatch.h`
- **Repository**: vllm-project/vllm
- **Purpose**: Defines low-level permutation/unpermutation CUDA kernels and dispatch helpers for MoE routing. / 定义 MoE 路由所需的底层置换/逆置换 CUDA 内核及分派辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Preamble and includes (lines 1-3)
```cpp
#pragma once
#include <cuda_fp8.h>
#define MOE_SWITCH(TYPE, ...)                                     \
```
**EN:** This opening block pulls in the headers, feature macros, and compile-time aliases that the rest of the file depends on.
**CN:** 这一开头代码块引入了后续实现依赖的头文件、特性宏以及编译期别名。

### Compile-time setup: MOE_DISPATCH_CASE (lines 11-16)
```cpp
#define MOE_DISPATCH_CASE(enum_type, ...)                  \
  case enum_type: {                                        \
    using scalar_t = ScalarType2CudaType<enum_type>::type; \
    __VA_ARGS__();                                         \
    break;                                                 \
  }
```
**EN:** These macros encode compile-time dispatch rules, selecting the right specialization based on ISA, dtype, or backend capabilities.
**CN:** 这些宏封装了编译期分派规则，用于根据 ISA、数据类型或后端能力选择合适的专用实现。

### Compile-time setup: MOE_DISPATCH_CASE (lines 17-23)
```cpp
#define MOE_DISPATCH_FLOAT_CASE(...)                            \
  MOE_DISPATCH_CASE(at::ScalarType::Float, __VA_ARGS__)         \
  MOE_DISPATCH_CASE(at::ScalarType::Half, __VA_ARGS__)          \
  MOE_DISPATCH_CASE(at::ScalarType::BFloat16, __VA_ARGS__)      \
  MOE_DISPATCH_CASE(at::ScalarType::Float8_e5m2, __VA_ARGS__)   \
  MOE_DISPATCH_CASE(at::ScalarType::Float8_e4m3fn, __VA_ARGS__) \
  MOE_DISPATCH_CASE(at::ScalarType::Byte, __VA_ARGS__)
```
**EN:** These macros encode compile-time dispatch rules, selecting the right specialization based on ISA, dtype, or backend capabilities.
**CN:** 这些宏封装了编译期分派规则，用于根据 ISA、数据类型或后端能力选择合适的专用实现。

### Compile-time setup: MOE_DISPATCH_FLOAT_CASE (lines 25-26)
```cpp
#define MOE_DISPATCH(TYPE, ...) \
  MOE_SWITCH(TYPE, MOE_DISPATCH_FLOAT_CASE(__VA_ARGS__))
```
**EN:** These macros encode compile-time dispatch rules, selecting the right specialization based on ISA, dtype, or backend capabilities.
**CN:** 这些宏封装了编译期分派规则，用于根据 ISA、数据类型或后端能力选择合适的专用实现。

### Struct: ScalarType2CudaType (lines 29-41)
```cpp
struct ScalarType2CudaType;

template <>
struct ScalarType2CudaType<at::ScalarType::Float> {
  using type = float;
};
template <>
struct ScalarType2CudaType<at::ScalarType::Half> {
  using type = half;
};
template <>
struct ScalarType2CudaType<at::ScalarType::BFloat16> {
  using type = __nv_bfloat16;
```
**EN:** This type defines shared state, data layout, or a reusable helper abstraction for the rest of the file.
**CN:** 该类型定义了共享状态、数据布局或供本文件其他部分复用的辅助抽象。

### Struct: ScalarType2CudaType (lines 45-47)
```cpp
struct ScalarType2CudaType<at::ScalarType::Byte> {
  using type = uint8_t;
};
```
**EN:** This type defines shared state, data layout, or a reusable helper abstraction for the rest of the file.
**CN:** 该类型定义了共享状态、数据布局或供本文件其他部分复用的辅助抽象。

### Struct: ScalarType2CudaType (lines 52-54)
```cpp
struct ScalarType2CudaType<at::ScalarType::Float8_e5m2> {
  using type = __nv_fp8_e5m2;
};
```
**EN:** This type defines shared state, data layout, or a reusable helper abstraction for the rest of the file.
**CN:** 该类型定义了共享状态、数据布局或供本文件其他部分复用的辅助抽象。

### Struct: ScalarType2CudaType (lines 56-58)
```cpp
struct ScalarType2CudaType<at::ScalarType::Float8_e4m3fn> {
  using type = __nv_fp8_e4m3;
};
```
**EN:** This type defines shared state, data layout, or a reusable helper abstraction for the rest of the file.
**CN:** 该类型定义了共享状态、数据布局或供本文件其他部分复用的辅助抽象。

## Key Concepts / 关键概念
- Mixture-of-Experts routing / 混合专家路由
- Token permutation / Token 置换
- Token restoration / Token 还原
- FP8 quantization / compute / FP8 量化与计算
- Integer kernel specialization / 整数内核专用化
- CUDA programming model / CUDA 编程模型
- Template-based specialization / 基于模板的专用化

## Dependencies / 依赖关系
- **External libraries / 外部库**: CUDA runtime / CUDA headers
- **Runtime coupling / 运行时耦合**: Uses templates to specialize code paths at compile time / 通过模板在编译期专用化代码路径
