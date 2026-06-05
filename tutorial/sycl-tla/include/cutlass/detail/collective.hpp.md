# collective.hpp — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `include/cutlass/detail/collective.hpp`

- **EN:** Internal helper utilities used by higher-level CUTLASS components.

- **CN:** 该头文件主要提供上层 CUTLASS 组件使用的内部辅助工具。文件级摘要：Internal helper utilities used by higher-level CUTLASS components.

## Line-by-Line Analysis / 逐行分析

### Lines 1-30

```cpp
/***************************************************************************************************
 * Copyright (c) 2023 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
 * SPDX-License-Identifier: BSD-3-Clause
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 * 1. Redistributions of source code must retain the above copyright notice, this
 * list of conditions and the following disclaimer.
 *
 * 2. Redistributions in binary form must reproduce the above copyright notice,
 * this list of conditions and the following disclaimer in the documentation
 * and/or other materials provided with the distribution.
 *
 * 3. Neither the name of the copyright holder nor the names of its
 * contributors may be used to endorse or promote products derived from
 * this software without specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
 * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
 * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
 * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
 * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
 * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
 * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
 * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 *
 **************************************************************************************************/
```

**EN:** This block records the file copyright, SPDX identifier, and redistribution disclaimer.

**CN:** 该代码块记录了文件的版权信息、SPDX 标识以及再分发免责声明。

### Lines 31-31

```cpp
#pragma once
```

**EN:** This directive uses `#pragma once` to avoid repeated inclusion of the same header.

**CN:** 这里使用 `#pragma once` 来避免同一头文件被重复包含。

### Lines 33-35

```cpp
#include "cute/container/tuple.hpp"
#include "cute/layout.hpp" // cute::size(shape)
#include "cute/arch/mma_sm100_desc.hpp" // cute::UMMA::MXF4Format, cute::UMMA::MXF8F6F4Format 
```

**EN:** This block imports dependencies such as `cute/container/tuple.hpp`, `cute/layout.hpp`, `cute/arch/mma_sm100_desc.hpp`, providing types, macros, or helper routines used later.

**CN:** 该代码块引入了 `cute/container/tuple.hpp`, `cute/layout.hpp`, `cute/arch/mma_sm100_desc.hpp` 等依赖，为后续代码提供类型、宏或辅助例程。

### Lines 36-37

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
namespace cutlass::gemm::collective {
```

**EN:** The preceding comment documents this block. This block opens the namespace scope `cutlass::gemm::collective` for the declarations that follow.

**CN:** 前面的注释说明了这个代码块。该代码块打开了 `cutlass::gemm::collective` 命名空间作用域，以容纳后续声明。

### Lines 40-41

```cpp
/////////////////////////////////////////////////////////////////////////////////////////////////
namespace detail {
```

**EN:** The preceding comment documents this block. This block opens the namespace scope `detail` for the declarations that follow.

**CN:** 前面的注释说明了这个代码块。该代码块打开了 `detail` 命名空间作用域，以容纳后续声明。

### Lines 44-45

```cpp
template <size_t I, class Tuple>
struct deduce_mixed_width_dtype {
```

**EN:** This block begins the definition of `deduce_mixed_width_dtype`, a `struct` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `deduce_mixed_width_dtype` 这个 `struct`，其成员会在后续代码中展开。

### Lines 46-47

```cpp
static_assert(I >= 0u && I <= 4u, "Valid indices are 0, 1, 2, 3 and 4, which represent "
                                  "Operand, Scale data type, Scale stride, Zero data type, and Zero Stride.");
```

**EN:** This `static_assert` checks template arguments or architectural assumptions at compile time.

**CN:** 这个 `static_assert` 会在编译期检查模板参数或架构假设。

### Lines 50-50

```cpp
  using underlying_tuple = cute::conditional_t<cute::is_tuple<Tuple>::value, Tuple, cute::tuple<Tuple>>;
```

**EN:** This alias defines `underlying_tuple` as `cute::conditional_t<cute::is_tuple<Tuple>::value, Tuple, cute::tuple<Tuple>>`, shortening later template or member declarations.

**CN:** 这里把 `underlying_tuple` 定义为 `cute::conditional_t<cute::is_tuple<Tuple>::value, Tuple, cute::tuple<Tuple>>` 的别名，以简化后续模板或成员声明。

### Lines 51-51

```cpp
  static constexpr size_t valid_index = cute::min(I, cute::tuple_size_v<underlying_tuple> - 1);
```

**EN:** This declaration defines `valid_index` and assigns it the compile-time expression `cute::min(I, cute::tuple_size_v<underlying_tuple> - 1)`.

**CN:** 这个声明定义了 `valid_index`，并把它设为编译期表达式 `cute::min(I, cute::tuple_size_v<underlying_tuple> - 1)`。

### Lines 54-56

```cpp
  using type = cute::conditional_t<(I < cute::tuple_size_v<underlying_tuple>), 
                                    cute::tuple_element_t<valid_index, underlying_tuple>,
                                    void>;
```

**EN:** This alias defines `type` as `cute::conditional_t<(I < cute::tuple_size_v<underlying_tuple>), cute::tuple_element_t<valid_index, underlying_tuple>, void>`, shortening later template or member declarations.

**CN:** 这里把 `type` 定义为 `cute::conditional_t<(I < cute::tuple_size_v<underlying_tuple>), cute::tuple_element_t<valid_index, underlying_tuple>, void>` 的别名，以简化后续模板或成员声明。

### Lines 59-60

```cpp
template <size_t I, class Tuple>
using deduce_mixed_width_dtype_t = typename deduce_mixed_width_dtype<I, Tuple>::type;
```

**EN:** This alias defines `deduce_mixed_width_dtype_t` as `typename deduce_mixed_width_dtype<I, Tuple>::type`, shortening later template or member declarations.

**CN:** 这里把 `deduce_mixed_width_dtype_t` 定义为 `typename deduce_mixed_width_dtype<I, Tuple>::type` 的别名，以简化后续模板或成员声明。

### Lines 64-71

```cpp
template <class Element>
CUTLASS_HOST_DEVICE
static constexpr bool
is_sm10x_runtime_f8f6f4() {
  return (cute::is_same_v<Element, cutlass::type_erased_dynamic_float8_t> ||
          cute::is_same_v<Element, cutlass::type_erased_dynamic_float6_t> ||
          cute::is_same_v<Element, cutlass::type_erased_dynamic_float4_t>);
}
```

**EN:** The function `is_sm10x_runtime_f8f6f4` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `is_sm10x_runtime_f8f6f4` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 73-103

```cpp
template <class ElementA, class ElementB>
CUTLASS_HOST_DEVICE
static constexpr bool
is_sm10x_f8f6f4_inputs() {
   return ( 
            
            cute::is_same_v<ElementA, cute::type_erased_dynamic_float8_t> || 
            cute::is_same_v<ElementA, cute::type_erased_dynamic_float6_t> ||
            cute::is_same_v<ElementA, cute::type_erased_dynamic_float4_t> ||
            
            cute::is_same_v<ElementA, cute::float_e4m3_t> ||
            cute::is_same_v<ElementA, cute::float_e5m2_t> 
            || cute::is_same_v<ElementA, cute::float_e3m2_t> ||
            cute::is_same_v<ElementA, cute::float_e2m3_t> ||
            cute::is_same_v<ElementA, cute::float_e2m1_t>
            
          ) &&
          ( 
            
            cute::is_same_v<ElementB, cute::type_erased_dynamic_float8_t> ||
            cute::is_same_v<ElementB, cute::type_erased_dynamic_float6_t> ||
            cute::is_same_v<ElementB, cute::type_erased_dynamic_float4_t> ||
            
            cute::is_same_v<ElementB, cute::float_e4m3_t> ||
            cute::is_same_v<ElementB, cute::float_e5m2_t> 
            || cute::is_same_v<ElementB, cute::float_e3m2_t> ||
            cute::is_same_v<ElementB, cute::float_e2m3_t> ||
            cute::is_same_v<ElementB, cute::float_e2m1_t>
            
          );
}
```

**EN:** The function `is_sm10x_f8f6f4_inputs` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `is_sm10x_f8f6f4_inputs` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 105-110

```cpp
template <class TiledMma, class ElementA, class ElementB>
CUTLASS_HOST_DEVICE
static constexpr bool
is_sm100_mma_f8f6f4() {
  return (cute::size<2>(typename TiledMma::Shape_MNK{}) == 32) && is_sm10x_f8f6f4_inputs<ElementA, ElementB>();
}
```

**EN:** The function `this entity` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `this entity` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 112-123

```cpp
template <class Element>
CUTLASS_HOST_DEVICE
static constexpr bool
is_sm10x_f8f6f4_element() {
  return (cute::is_same_v<Element, cute::float_e4m3_t> 
          || cute::is_same_v<Element, cute::float_e5m2_t> 
          || cute::is_same_v<Element, cute::float_e3m2_t>
          || cute::is_same_v<Element, cute::float_e2m3_t>
          || cute::is_same_v<Element, cute::float_e2m1_t>
          
        );
}
```

**EN:** The function `is_sm10x_f8f6f4_element` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `is_sm10x_f8f6f4_element` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 126-132

```cpp
template <class Element>
CUTLASS_HOST_DEVICE
static constexpr bool
is_sm10x_f4_element() {
  return (cute::is_same_v<Element, cute::float_e2m1_t> 
  );
}
```

**EN:** The function `is_sm10x_f4_element` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `is_sm10x_f4_element` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 134-147

```cpp
template <class ElementType>
CUTLASS_HOST_DEVICE
static constexpr bool
is_sm10x_mxf8f6f4_input() {
          // ElementType must be F8, F6, or F4
   return ( cute::is_same_v<ElementType, cutlass::type_erased_dynamic_float8_t> ||
            cute::is_same_v<ElementType, cutlass::detail::type_erased_dynamic_float6_unpacksmem_t> ||
            cute::is_same_v<ElementType, cutlass::detail::type_erased_dynamic_float4_unpacksmem_t> ||
            cute::is_same_v<ElementType, cutlass::float_e4m3_t> ||
            cute::is_same_v<ElementType, cutlass::float_e5m2_t> ||
            cute::is_same_v<ElementType, cutlass::detail::float_e2m3_unpacksmem_t> ||
            cute::is_same_v<ElementType, cutlass::detail::float_e3m2_unpacksmem_t> ||
            cute::is_same_v<ElementType, cutlass::detail::float_e2m1_unpacksmem_t>);
}
```

**EN:** The preceding comment documents this block. The function `is_sm10x_mxf8f6f4_input` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** 前面的注释说明了这个代码块。`is_sm10x_mxf8f6f4_input` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 149-157

```cpp
template <class ElementType>
CUTLASS_HOST_DEVICE
static constexpr bool
is_sm10x_mxf4nvf4_input() {
          // ElementType must be F4
   return ( cute::is_same_v<ElementType, cute::type_erased_dynamic_float4_t> ||
            cute::is_same_v<ElementType, cute::float_e2m1_t> 
          );
}
```

**EN:** The preceding comment documents this block. The function `is_sm10x_mxf4nvf4_input` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** 前面的注释说明了这个代码块。`is_sm10x_mxf4nvf4_input` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 159-160

```cpp
template <class ElementType, bool IsRuntimeDataType>
struct sm10x_block_scale_runtime_input_t {
```

**EN:** This block begins the definition of `sm10x_block_scale_runtime_input_t`, a `struct` whose members are laid out in the following lines.

**CN:** 该代码块开始定义 `sm10x_block_scale_runtime_input_t` 这个 `struct`，其成员会在后续代码中展开。

### Lines 161-161

```cpp
  static constexpr bool IsF8F6F4MmaInput = is_sm10x_mxf8f6f4_input<ElementType>();
```

**EN:** This declaration defines `IsF8F6F4MmaInput` and assigns it the compile-time expression `is_sm10x_mxf8f6f4_input<ElementType>()`.

**CN:** 这个声明定义了 `IsF8F6F4MmaInput`，并把它设为编译期表达式 `is_sm10x_mxf8f6f4_input<ElementType>()`。

### Lines 162-162

```cpp
  static constexpr bool IsF4MmaInput = is_sm10x_mxf4nvf4_input<ElementType>();
```

**EN:** This declaration defines `IsF4MmaInput` and assigns it the compile-time expression `is_sm10x_mxf4nvf4_input<ElementType>()`.

**CN:** 这个声明定义了 `IsF4MmaInput`，并把它设为编译期表达式 `is_sm10x_mxf4nvf4_input<ElementType>()`。

### Lines 164-170

```cpp
  using Type = cute::conditional_t<IsRuntimeDataType && IsF8F6F4MmaInput, 
                                   cute::UMMA::MXF8F6F4Format, 
               cute::conditional_t<IsRuntimeDataType && IsF4MmaInput, 
                                   cute::UMMA::MXF4Format, 
                                   void*
                                   >
                                  >;
```

**EN:** This alias defines `Type` as `cute::conditional_t<IsRuntimeDataType && IsF8F6F4MmaInput, cute::UMMA::MXF8F6F4Format, cute::conditional_t<IsRuntimeDataType && IsF4MmaInput, cute::UMMA::MXF4Format, void* > >`, shortening later template or member declarations.

**CN:** 这里把 `Type` 定义为 `cute::conditional_t<IsRuntimeDataType && IsF8F6F4MmaInput, cute::UMMA::MXF8F6F4Format, cute::conditional_t<IsRuntimeDataType && IsF4MmaInput, cute::UMMA::MXF4Format, void* > >` 的别名，以简化后续模板或成员声明。

### Lines 174-179

```cpp
template <class TiledMma, class ElementA, class ElementB>
CUTLASS_HOST_DEVICE
static constexpr bool
is_sm120_f8f6f4() {
  return (cute::size<2>(typename TiledMma::Shape_MNK{}) == 32) && is_sm10x_f8f6f4_inputs<ElementA, ElementB>();
}
```

**EN:** The function `this entity` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `this entity` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

### Lines 181-186

```cpp
template <class TiledMma, class ElementA, class ElementB>
CUTLASS_HOST_DEVICE
static constexpr bool
is_sm100_sparse_f8f6f4() {
  return (cute::size<2>(typename TiledMma::Shape_MNK{}) == 64) && is_sm10x_f8f6f4_inputs<ElementA, ElementB>();
}
```

**EN:** The function `this entity` implements a concrete operation in this abstraction. The body mainly computes and returns a value from the current state or inputs.

**CN:** `this entity` 函数实现了该抽象中的一个具体操作。函数体主要根据当前状态或输入计算并返回结果。

## Key Concepts / 关键概念

- **EN:** Heavy use of templates enables compile-time specialization and zero-overhead abstractions.
  **CN:** 大量使用模板，使该文件能够进行编译期特化并保持零额外开销的抽象。

- **EN:** The file handles specialized numeric formats or packed data representations used by accelerators.
  **CN:** 该文件处理加速器常用的特种数值格式或打包数据表示。

- **EN:** Type traits and compile-time checks constrain valid instantiations.
  **CN:** 类型萃取与编译期检查用于约束合法的模板实例化。

- **EN:** Conditional compilation keeps the header portable across host, device, CUDA, and SYCL builds.
  **CN:** 条件编译使该头文件能够在主机端、设备端、CUDA 与 SYCL 构建之间保持可移植性。

## Dependencies / 依赖关系

- **EN:** Direct includes: `cute/container/tuple.hpp`, `cute/layout.hpp`, `cute/arch/mma_sm100_desc.hpp`.
  **CN:** 直接包含：`cute/container/tuple.hpp`, `cute/layout.hpp`, `cute/arch/mma_sm100_desc.hpp`。

- **EN:** Primary namespaces: `cutlass::gemm::collective`, `detail`.
  **CN:** 主要命名空间：`cutlass::gemm::collective`, `detail`。

- **EN:** Important macros or compile flags: `CUTLASS_HOST_DEVICE`.
  **CN:** 重要宏或编译开关：`CUTLASS_HOST_DEVICE`。
