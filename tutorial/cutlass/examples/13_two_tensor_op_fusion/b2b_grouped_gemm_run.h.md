# b2b_grouped_gemm_run.h — Code Analysis / 代码分析

**Source / 源文件**: `examples/13_two_tensor_op_fusion/b2b_grouped_gemm_run.h`  
**Purpose / 用途**: Provides host-side helpers for grouped fused back-to-back GEMM experiments. / 提供分组式融合双 GEMM 实验的主机侧辅助代码。

---

## Line-by-Line Analysis / 逐行分析

### Lines 1-31 / 第1-31行

```cpp
/***************************************************************************************************
 * Copyright (c) 2023 - 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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
/*! \file
```

**EN**: This is the standard BSD-3-Clause license header. It documents redistribution terms before any executable code appears.  
**CN**: 这里是标准的 BSD-3-Clause 许可证头，在任何可执行代码之前先说明再分发条款。

### Lines 32-33 / 第32-33行

```cpp
    \brief Containers for running grouped back-to-back GEMMs
*/
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 35-35 / 第35-35行

```cpp
#pragma once
```

**EN**: This pragma makes the header idempotent so repeated inclusion does not create duplicate definitions.  
**CN**: 这个 pragma 保证头文件只会被处理一次，避免重复包含产生重复定义。

### Lines 37-39 / 第37-39行

```cpp
#include <iostream>
#include <fstream>
#include <sstream>
```

**EN**: These headers pull in the building blocks required by this file. Console stream utilities for logging progress and results. File stream utilities for reading or writing benchmark data. String-stream helpers for assembling formatted text.  
**CN**: 这些头文件引入了当前文件所需的构建模块。用于记录进度和结果的控制台流工具。用于读写基准数据的文件流工具。用于拼接格式化文本的字符串流工具。

### Lines 41-50 / 第41-50行

```cpp
#include "cutlass/util/device_memory.h"
#include "cutlass/util/host_tensor.h"
#include "cutlass/util/tensor_view_io.h"
#include "cutlass/util/distribution.h"
#include "cutlass/util/reference/host/tensor_fill.h"
#include "cutlass/util/reference/host/tensor_copy.h"
#include "cutlass/util/reference/host/tensor_compare.h"
#include "cutlass/util/reference/host/tensor_norm.h"
#include "cutlass/util/reference/device/gemm.h"
#include "cutlass/util/reference/device/tensor_relu.h"
```

**EN**: These headers pull in the building blocks required by this file. RAII-style device-memory helpers. Host/device tensor wrapper used to allocate storage and transfer data. Tensor printing helpers for debugging layouts and values. Random-data distribution utilities for initializing test tensors.  
**CN**: 这些头文件引入了当前文件所需的构建模块。RAII 风格的设备内存辅助工具。主机/设备张量封装，用于分配存储并传输数据。用于调试布局和值的张量打印辅助工具。用于初始化测试张量的随机分布工具。

### Lines 52-53 / 第52-53行

```cpp
#include "reference/device/tensor_scale_bias.h"
#include "helper.h"
```

**EN**: These headers pull in the building blocks required by this file. Example-local reference helper used to validate fused results. Provides `helper.h` so this file can use the related API or helper utilities.  
**CN**: 这些头文件引入了当前文件所需的构建模块。示例本地参考辅助组件，用于验证融合结果。提供 `helper.h`，使本文件能够使用相关 API 或辅助工具。

### Lines 55-55 / 第55-55行

```cpp
#define CHECK_GT(val1, val2) \
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 56-57 / 第56-57行

```cpp
    if((val1) <= (val2)) \
        std::cerr << __FILE__ << " " << __LINE__ << ": CHECK_GT failed\n";
```

**EN**: This control-flow block selects a path based on runtime state, such as argument validation, loop traversal, or pass/fail reporting.  
**CN**: 这个控制流代码块会根据运行时状态选择执行路径，例如参数校验、循环遍历或通过/失败报告。

### Lines 58-58 / 第58-58行

```cpp
#define CHECK_TRUE(val) \
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 59-60 / 第59-60行

```cpp
    if(!(val)) \
        std::cerr << __FILE__ << " " << __LINE__ << ": CHECK_TRUE failed\n";
```

**EN**: This control-flow block selects a path based on runtime state, such as argument validation, loop traversal, or pass/fail reporting.  
**CN**: 这个控制流代码块会根据运行时状态选择执行路径，例如参数校验、循环遍历或通过/失败报告。

### Lines 62-62 / 第62-62行

```cpp
////////////////////////////////////////////////////////////////////////////////
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 64-66 / 第64-66行

```cpp
template <typename B2bGemm_>
struct B2bFusedGroupedGemmRun
{
```

**EN**: This declaration introduces a template or type shell that the rest of the file will specialize or fill in. Much of CUTLASS customization happens through such templates. Grouped execution lets one launch process many problems with different shapes and uses a scheduler to distribute them across thread blocks.  
**CN**: 这个声明引入了一个模板或类型骨架，供后续代码进行特化或填充。CUTLASS 的大量定制都通过这类模板完成。 分组执行允许一次启动处理多个不同形状的问题，并通过调度器把它们分配给线程块。

### Lines 68-68 / 第68-68行

```cpp
  using B2bGemm = B2bGemm_;
```

**EN**: This type alias assembles a specialized CUTLASS kernel configuration. The template arguments choose layouts, scalar types, architecture tags, tile shapes, iterator policies, epilogues, swizzles, and staging depth.  
**CN**: 这个类型别名组装出了一个专门化的 CUTLASS 内核配置。模板参数会选择布局、标量类型、架构标签、分块形状、迭代器策略、epilogue、swizzle 以及流水深度。

### Lines 69-69 / 第69-69行

```cpp
  using ElementAccumulator = typename B2bGemm::ElementAccumulator;
```

**EN**: This type alias assembles a specialized CUTLASS kernel configuration. The template arguments choose layouts, scalar types, architecture tags, tile shapes, iterator policies, epilogues, swizzles, and staging depth.  
**CN**: 这个类型别名组装出了一个专门化的 CUTLASS 内核配置。模板参数会选择布局、标量类型、架构标签、分块形状、迭代器策略、epilogue、swizzle 以及流水深度。

### Lines 70-70 / 第70-70行

```cpp
  using ElementCompute = typename B2bGemm::BaseKernel::Epilogue::OutputOp::ElementCompute;
```

**EN**: This type alias assembles a specialized CUTLASS kernel configuration. The template arguments choose layouts, scalar types, architecture tags, tile shapes, iterator policies, epilogues, swizzles, and staging depth.  
**CN**: 这个类型别名组装出了一个专门化的 CUTLASS 内核配置。模板参数会选择布局、标量类型、架构标签、分块形状、迭代器策略、epilogue、swizzle 以及流水深度。

### Lines 72-72 / 第72-72行

```cpp
  /// Initialization
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 73-73 / 第73-73行

```cpp
  cutlass::Distribution::Kind init_A;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 74-74 / 第74-74行

```cpp
  cutlass::Distribution::Kind init_B;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 75-75 / 第75-75行

```cpp
  cutlass::Distribution::Kind init_C;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 76-76 / 第76-76行

```cpp
  cutlass::Distribution::Kind init_Scale;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 77-77 / 第77-77行

```cpp
  cutlass::Distribution::Kind init_Bias;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 78-78 / 第78-78行

```cpp
  uint64_t seed;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 80-82 / 第80-82行

```cpp
  //
  // Methods
  //
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 84-93 / 第84-93行

```cpp
  B2bFusedGroupedGemmRun(
    cutlass::Distribution::Kind init_A_ = cutlass::Distribution::Uniform, 
    cutlass::Distribution::Kind init_B_ = cutlass::Distribution::Uniform, 
    cutlass::Distribution::Kind init_C_ = cutlass::Distribution::Uniform, 
    cutlass::Distribution::Kind init_Scale_ = cutlass::Distribution::Uniform,
    cutlass::Distribution::Kind init_Bias_ = cutlass::Distribution::Uniform,
    uint64_t seed_ = 2080
  ):
    init_A(init_A_), init_B(init_B_), init_C(init_C_),
    init_Scale(init_Scale_), init_Bias(init_Bias_), seed(seed_) { }
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps. Grouped execution lets one launch process many problems with different shapes and uses a scheduler to distribute them across thread blocks.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。 分组执行允许一次启动处理多个不同形状的问题，并通过调度器把它们分配给线程块。

### Lines 95-95 / 第95-95行

```cpp
  /// Helper to initialize a tensor view
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 96-100 / 第96-100行

```cpp
  template <typename Element, typename Layout>
  bool initialize_tensor(
    cutlass::TensorView<Element, Layout> view, 
    cutlass::Distribution::Kind dist_kind,
    uint64_t seed) {
```

**EN**: This declaration introduces a template or type shell that the rest of the file will specialize or fill in. Much of CUTLASS customization happens through such templates.  
**CN**: 这个声明引入了一个模板或类型骨架，供后续代码进行特化或填充。CUTLASS 的大量定制都通过这类模板完成。

### Lines 102-104 / 第102-104行

```cpp
    if (dist_kind == cutlass::Distribution::Uniform) {
      cutlass::reference::host::TensorFillRandomUniform(
        view, seed, 1, -1, 0);
```

**EN**: This control-flow block selects a path based on runtime state, such as argument validation, loop traversal, or pass/fail reporting.  
**CN**: 这个控制流代码块会根据运行时状态选择执行路径，例如参数校验、循环遍历或通过/失败报告。

### Lines 106-106 / 第106-106行

```cpp
    } 
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 107-108 / 第107-108行

```cpp
    else if (dist_kind == cutlass::Distribution::Identity) {
      cutlass::reference::host::TensorFillIdentity(view);
```

**EN**: This control-flow block selects a path based on runtime state, such as argument validation, loop traversal, or pass/fail reporting.  
**CN**: 这个控制流代码块会根据运行时状态选择执行路径，例如参数校验、循环遍历或通过/失败报告。

### Lines 110-110 / 第110-110行

```cpp
    } 
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 111-112 / 第111-112行

```cpp
    else if (dist_kind == cutlass::Distribution::Gaussian) {
      cutlass::reference::host::TensorFillRandomGaussian(view, seed, 0, 0.5);
```

**EN**: This control-flow block selects a path based on runtime state, such as argument validation, loop traversal, or pass/fail reporting.  
**CN**: 这个控制流代码块会根据运行时状态选择执行路径，例如参数校验、循环遍历或通过/失败报告。

### Lines 114-114 / 第114-114行

```cpp
    }
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 115-117 / 第115-117行

```cpp
    else if (dist_kind == cutlass::Distribution::Sequential) {
      cutlass::reference::host::BlockFillSequential(
        view.data(), view.capacity());
```

**EN**: This control-flow block selects a path based on runtime state, such as argument validation, loop traversal, or pass/fail reporting.  
**CN**: 这个控制流代码块会根据运行时状态选择执行路径，例如参数校验、循环遍历或通过/失败报告。

### Lines 119-119 / 第119-119行

```cpp
    }
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 120-121 / 第120-121行

```cpp
    else if (dist_kind == cutlass::Distribution::AllZeros) {
      cutlass::reference::host::TensorFill(view, Element(0));
```

**EN**: This control-flow block selects a path based on runtime state, such as argument validation, loop traversal, or pass/fail reporting.  
**CN**: 这个控制流代码块会根据运行时状态选择执行路径，例如参数校验、循环遍历或通过/失败报告。

### Lines 122-122 / 第122-122行

```cpp
    }
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 123-124 / 第123-124行

```cpp
    else if (dist_kind == cutlass::Distribution::AllOnes) {
      cutlass::reference::host::TensorFill(view, Element(1));
```

**EN**: This control-flow block selects a path based on runtime state, such as argument validation, loop traversal, or pass/fail reporting.  
**CN**: 这个控制流代码块会根据运行时状态选择执行路径，例如参数校验、循环遍历或通过/失败报告。

### Lines 125-125 / 第125-125行

```cpp
    }
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 126-127 / 第126-127行

```cpp
    else {
      std::cerr << "Not implemented\n";
```

**EN**: This control-flow block selects a path based on runtime state, such as argument validation, loop traversal, or pass/fail reporting.  
**CN**: 这个控制流代码块会根据运行时状态选择执行路径，例如参数校验、循环遍历或通过/失败报告。

### Lines 128-128 / 第128-128行

```cpp
      return false;
```

**EN**: This return statement propagates the current result back to the caller so the surrounding harness can continue its control flow.  
**CN**: 这个返回语句会把当前结果传回调用者，使外围测试框架能够继续执行后续控制流程。

### Lines 129-129 / 第129-129行

```cpp
    }
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 131-131 / 第131-131行

```cpp
    return true;
```

**EN**: This return statement propagates the current result back to the caller so the surrounding harness can continue its control flow.  
**CN**: 这个返回语句会把当前结果传回调用者，使外围测试框架能够继续执行后续控制流程。

### Lines 132-132 / 第132-132行

```cpp
  }
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 134-134 / 第134-134行

```cpp
  /// Executes one test
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 135-145 / 第135-145行

```cpp
  bool run(
    std::vector<cutlass::gemm::GemmCoord> problem_sizes_0,
    std::vector<cutlass::gemm::GemmCoord> problem_sizes_1,
    ElementCompute alpha0 = ElementCompute(1),
    ElementCompute beta0 = ElementCompute(0),
    ElementCompute alpha1 = ElementCompute(1),
    ElementCompute beta1 = ElementCompute(0),
    bool relu = true,
    int warm_ups = 1,
    int runs = 100) {
    using HostTensorA = cutlass::HostTensor<typename B2bGemm::ElementA, typename B2bGemm::LayoutA>;
```

**EN**: This block exercises the configured CUTLASS object at runtime. Typical calls include initialization, capability checks, workspace queries, kernel launches, or reference validation.  
**CN**: 这段代码在运行时驱动已配置好的 CUTLASS 对象。常见调用包括初始化、能力检查、工作区查询、内核启动以及参考结果验证。

### Lines 147-147 / 第147-147行

```cpp
    using HostTensorB = cutlass::HostTensor<typename B2bGemm::ElementB, typename B2bGemm::LayoutB>;
```

**EN**: This type alias assembles a specialized CUTLASS kernel configuration. The template arguments choose layouts, scalar types, architecture tags, tile shapes, iterator policies, epilogues, swizzles, and staging depth.  
**CN**: 这个类型别名组装出了一个专门化的 CUTLASS 内核配置。模板参数会选择布局、标量类型、架构标签、分块形状、迭代器策略、epilogue、swizzle 以及流水深度。

### Lines 148-148 / 第148-148行

```cpp
    using HostTensorC = cutlass::HostTensor<typename B2bGemm::ElementC, typename B2bGemm::LayoutC>;
```

**EN**: This type alias assembles a specialized CUTLASS kernel configuration. The template arguments choose layouts, scalar types, architecture tags, tile shapes, iterator policies, epilogues, swizzles, and staging depth.  
**CN**: 这个类型别名组装出了一个专门化的 CUTLASS 内核配置。模板参数会选择布局、标量类型、架构标签、分块形状、迭代器策略、epilogue、swizzle 以及流水深度。

### Lines 149-149 / 第149-149行

```cpp
    using HostTensorScale = cutlass::HostTensor<ElementCompute, typename B2bGemm::LayoutC>;
```

**EN**: This type alias assembles a specialized CUTLASS kernel configuration. The template arguments choose layouts, scalar types, architecture tags, tile shapes, iterator policies, epilogues, swizzles, and staging depth.  
**CN**: 这个类型别名组装出了一个专门化的 CUTLASS 内核配置。模板参数会选择布局、标量类型、架构标签、分块形状、迭代器策略、epilogue、swizzle 以及流水深度。

### Lines 150-150 / 第150-150行

```cpp
    using HostTensorZ = cutlass::HostTensor<ElementAccumulator, typename B2bGemm::LayoutC>;
```

**EN**: This type alias assembles a specialized CUTLASS kernel configuration. The template arguments choose layouts, scalar types, architecture tags, tile shapes, iterator policies, epilogues, swizzles, and staging depth.  
**CN**: 这个类型别名组装出了一个专门化的 CUTLASS 内核配置。模板参数会选择布局、标量类型、架构标签、分块形状、迭代器策略、epilogue、swizzle 以及流水深度。

### Lines 151-151 / 第151-151行

```cpp
    using HostTensorBias = cutlass::HostTensor<ElementCompute, typename B2bGemm::LayoutC>;
```

**EN**: This type alias assembles a specialized CUTLASS kernel configuration. The template arguments choose layouts, scalar types, architecture tags, tile shapes, iterator policies, epilogues, swizzles, and staging depth.  
**CN**: 这个类型别名组装出了一个专门化的 CUTLASS 内核配置。模板参数会选择布局、标量类型、架构标签、分块形状、迭代器策略、epilogue、swizzle 以及流水深度。

### Lines 153-153 / 第153-153行

```cpp
    int problem_count = (int)problem_sizes_0.size();
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 155-155 / 第155-155行

```cpp
    std::vector<HostTensorA> host_tensor_A0(problem_count);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 156-156 / 第156-156行

```cpp
    std::vector<HostTensorB> host_tensor_B0(problem_count);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 157-157 / 第157-157行

```cpp
    std::vector<HostTensorC> host_tensor_C0(problem_count);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 158-158 / 第158-158行

```cpp
    std::vector<HostTensorScale> host_tensor_Scale0(problem_count);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 159-159 / 第159-159行

```cpp
    std::vector<HostTensorScale> host_tensor_Bias0(problem_count);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 160-160 / 第160-160行

```cpp
    std::vector<HostTensorB> host_tensor_B1(problem_count);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 161-161 / 第161-161行

```cpp
    std::vector<HostTensorC> host_tensor_C1(problem_count);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 162-162 / 第162-162行

```cpp
    std::vector<HostTensorBias> host_tensor_Bias1(problem_count);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 163-163 / 第163-163行

```cpp
    std::vector<HostTensorC> host_tensor_D1(problem_count);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 164-164 / 第164-164行

```cpp
    std::vector<HostTensorZ> host_tensor_Z(problem_count);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 165-165 / 第165-165行

```cpp
    std::vector<HostTensorC> host_tensor_ref_D0(problem_count);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 166-166 / 第166-166行

```cpp
    std::vector<HostTensorC> host_tensor_ref_D1(problem_count);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 168-168 / 第168-168行

```cpp
    std::vector<typename HostTensorA::TensorRef> ref_A0(problem_count);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 169-169 / 第169-169行

```cpp
    std::vector<typename HostTensorB::TensorRef> ref_B0(problem_count);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 170-170 / 第170-170行

```cpp
    std::vector<typename HostTensorC::TensorRef> ref_C0(problem_count);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 171-171 / 第171-171行

```cpp
    std::vector<typename HostTensorScale::TensorRef> ref_Scale0(problem_count);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 172-172 / 第172-172行

```cpp
    std::vector<typename HostTensorScale::TensorRef> ref_Bias0(problem_count);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 173-173 / 第173-173行

```cpp
    std::vector<typename HostTensorB::TensorRef> ref_B1(problem_count);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 174-174 / 第174-174行

```cpp
    std::vector<typename HostTensorC::TensorRef> ref_C1(problem_count);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 175-175 / 第175-175行

```cpp
    std::vector<typename HostTensorBias::TensorRef> ref_Bias1(problem_count);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 176-176 / 第176-176行

```cpp
    std::vector<typename HostTensorC::TensorRef> ref_D1(problem_count);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 177-177 / 第177-177行

```cpp
    std::vector<typename HostTensorZ::TensorRef> ref_Z(problem_count);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 178-178 / 第178-178行

```cpp
    std::vector<typename HostTensorC::TensorRef> ref_ref_D0(problem_count);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 179-179 / 第179-179行

```cpp
    std::vector<typename HostTensorC::TensorRef> ref_ref_D1(problem_count);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 181-240 / 第181-240行

```cpp
    for (int i = 0; i < problem_count; ++i) {
      //
      // Allocate the GEMM workspace
      //
      auto problem_size_0 = problem_sizes_0[i];
      auto problem_size_1 = problem_sizes_1[i];
      host_tensor_A0.at(i) = HostTensorA(problem_size_0.mk());
      host_tensor_B0.at(i) = HostTensorB(problem_size_0.kn());
      host_tensor_C0.at(i) = HostTensorC(problem_size_0.mn());
      if (alpha0 == ElementCompute(0)) //per-channel scale
        host_tensor_Scale0.at(i) = HostTensorScale(typename HostTensorZ::Layout::TensorCoord{1, problem_size_0.n()});
      host_tensor_Bias0.at(i) = HostTensorScale(typename HostTensorBias::Layout::TensorCoord{1, problem_size_0.n()});
      host_tensor_Z.at(i) = HostTensorZ(problem_size_0.mn());
      host_tensor_ref_D0.at(i) = HostTensorC(problem_size_0.mn());
      host_tensor_B1.at(i) = HostTensorB(problem_size_1.kn());
      host_tensor_C1.at(i) = HostTensorC(problem_size_1.mn());
      host_tensor_Bias1.at(i) = HostTensorScale(typename HostTensorBias::Layout::TensorCoord{1, problem_size_1.n()});
      host_tensor_D1.at(i) = HostTensorC(problem_size_1.mn());
      host_tensor_ref_D1.at(i) = HostTensorC(problem_size_1.mn());
      CHECK_TRUE(initialize_tensor(host_tensor_A0.at(i).host_view(), init_A, seed + 2019));
      CHECK_TRUE(initialize_tensor(host_tensor_B0.at(i).host_view(), init_B, seed + 2018));
      CHECK_TRUE(initialize_tensor(host_tensor_C0.at(i).host_view(), init_C, seed + 2017));
      if (alpha0 == ElementCompute(0)) //per-channel scale
        CHECK_TRUE(initialize_tensor(host_tensor_Scale0.at(i).host_view(), init_Scale, seed + 2014));
      CHECK_TRUE(initialize_tensor(host_tensor_Bias0.at(i).host_view(), init_Bias, seed + 2013));
      CHECK_TRUE(initialize_tensor(host_tensor_B1.at(i).host_view(), init_B, seed + 2016));
      CHECK_TRUE(initialize_tensor(host_tensor_C1.at(i).host_view(), init_C, seed + 2015));
      CHECK_TRUE(initialize_tensor(host_tensor_Bias1.at(i).host_view(), init_Bias, seed + 2012));
      cutlass::reference::host::TensorFill(
        host_tensor_D1.at(i).host_view());
      cutlass::reference::host::TensorFill(
        host_tensor_ref_D0.at(i).host_view());
      cutlass::reference::host::TensorFill(
        host_tensor_ref_D1.at(i).host_view());
      host_tensor_A0.at(i).sync_device();
      host_tensor_B0.at(i).sync_device();
      host_tensor_C0.at(i).sync_device();
      if (alpha0 == ElementCompute(0)) //per-channel scale
        host_tensor_Scale0.at(i).sync_device();
      host_tensor_Bias0.at(i).sync_device();
      host_tensor_B1.at(i).sync_device();
      host_tensor_C1.at(i).sync_device();
      host_tensor_Bias1.at(i).sync_device();
      host_tensor_D1.at(i).sync_device();
      host_tensor_ref_D0.at(i).sync_device();
      host_tensor_ref_D1.at(i).sync_device();
      ref_A0.at(i) = (host_tensor_A0.at(i).device_ref());
      ref_B0.at(i) = (host_tensor_B0.at(i).device_ref());
      ref_C0.at(i) = (host_tensor_C0.at(i).device_ref());
      if (alpha0 == ElementCompute(0)) //per-channel scale
        ref_Scale0.at(i) = (host_tensor_Scale0.at(i).device_ref());
      ref_Bias0.at(i) = (host_tensor_Bias0.at(i).device_ref());
      ref_B1.at(i) = (host_tensor_B1.at(i).device_ref());
      ref_C1.at(i) = {host_tensor_Bias1.at(i).device_data(), typename B2bGemm::LayoutC::Stride(0)};
      ref_Bias1.at(i) = (host_tensor_Bias1.at(i).device_ref());
      ref_D1.at(i) = (host_tensor_D1.at(i).device_ref());
      ref_Z.at(i) = (host_tensor_Z.at(i).device_ref());
      ref_ref_D0.at(i) = (host_tensor_ref_D0.at(i).device_ref());
      ref_ref_D1.at(i) = (host_tensor_ref_D1.at(i).device_ref());
    }
```

**EN**: This control-flow block selects a path based on runtime state, such as argument validation, loop traversal, or pass/fail reporting.  
**CN**: 这个控制流代码块会根据运行时状态选择执行路径，例如参数校验、循环遍历或通过/失败报告。

### Lines 248-250 / 第248-250行

```cpp
    //
    // Initialize the GEMM operator
    //
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 252-252 / 第252-252行

```cpp
    cutlass::DeviceAllocation<typename HostTensorA::TensorRef> device_ref_A0(problem_count);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 253-253 / 第253-253行

```cpp
    device_ref_A0.copy_from_host(ref_A0.data());
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 254-254 / 第254-254行

```cpp
    cutlass::DeviceAllocation<typename HostTensorB::TensorRef> device_ref_B0(problem_count);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 255-255 / 第255-255行

```cpp
    device_ref_B0.copy_from_host(ref_B0.data());
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 256-256 / 第256-256行

```cpp
    cutlass::DeviceAllocation<typename HostTensorC::TensorRef> device_ref_C0(problem_count);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 257-257 / 第257-257行

```cpp
    device_ref_C0.copy_from_host(ref_C0.data());
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 258-258 / 第258-258行

```cpp
    cutlass::DeviceAllocation<typename HostTensorScale::TensorRef> device_ref_Scale0(problem_count);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 259-259 / 第259-259行

```cpp
    device_ref_Scale0.copy_from_host(ref_Scale0.data());
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 260-260 / 第260-260行

```cpp
    cutlass::DeviceAllocation<typename HostTensorScale::TensorRef> device_ref_Bias0(problem_count);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 261-261 / 第261-261行

```cpp
    device_ref_Bias0.copy_from_host(ref_Bias0.data());
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 262-262 / 第262-262行

```cpp
    cutlass::DeviceAllocation<typename HostTensorB::TensorRef> device_ref_B1(problem_count);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 263-263 / 第263-263行

```cpp
    device_ref_B1.copy_from_host(ref_B1.data());
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 264-264 / 第264-264行

```cpp
    cutlass::DeviceAllocation<typename HostTensorC::TensorRef> device_ref_C1(problem_count);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 265-265 / 第265-265行

```cpp
    device_ref_C1.copy_from_host(ref_C1.data());
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 266-266 / 第266-266行

```cpp
    cutlass::DeviceAllocation<typename HostTensorBias::TensorRef> device_ref_Bias1(problem_count);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 267-267 / 第267-267行

```cpp
    device_ref_Bias1.copy_from_host(ref_Bias1.data());
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 268-268 / 第268-268行

```cpp
    cutlass::DeviceAllocation<typename HostTensorC::TensorRef> device_ref_D1(problem_count);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 269-269 / 第269-269行

```cpp
    device_ref_D1.copy_from_host(ref_D1.data());
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 271-271 / 第271-271行

```cpp
    cutlass::DeviceAllocation<cutlass::gemm::GemmCoord> device_problem_sizes_0(problem_count);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 272-272 / 第272-272行

```cpp
    device_problem_sizes_0.copy_from_host(problem_sizes_0.data());
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 273-273 / 第273-273行

```cpp
    cutlass::DeviceAllocation<cutlass::gemm::GemmCoord> device_problem_sizes_1(problem_count);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 274-274 / 第274-274行

```cpp
    device_problem_sizes_1.copy_from_host(problem_sizes_1.data());
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 276-276 / 第276-276行

```cpp
    B2bGemm b2b_gemm_op;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 278-278 / 第278-278行

```cpp
    int threadblock_count = B2bGemm::sufficient(problem_sizes_1.data(), problem_count);
```

**EN**: This block fills in runtime state, object fields, or helper variables needed by later setup and execution steps.  
**CN**: 这段代码填充运行时状态、对象字段或辅助变量，为后续的配置与执行步骤做准备。

### Lines 279-280 / 第279-280行

```cpp
    if (!threadblock_count) {
      std::cout << "Active CUDA device lacks hardware resources to run CUTLASS Grouped GEMM kernel." << std::endl;
```

**EN**: This control-flow block selects a path based on runtime state, such as argument validation, loop traversal, or pass/fail reporting. Grouped execution lets one launch process many problems with different shapes and uses a scheduler to distribute them across thread blocks.  
**CN**: 这个控制流代码块会根据运行时状态选择执行路径，例如参数校验、循环遍历或通过/失败报告。 分组执行允许一次启动处理多个不同形状的问题，并通过调度器把它们分配给线程块。

### Lines 281-281 / 第281-281行

```cpp
      return false;
```

**EN**: This return statement propagates the current result back to the caller so the surrounding harness can continue its control flow.  
**CN**: 这个返回语句会把当前结果传回调用者，使外围测试框架能够继续执行后续控制流程。

### Lines 282-282 / 第282-282行

```cpp
    }
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 284-299 / 第284-299行

```cpp
    typename B2bGemm::Arguments arguments{
      problem_count,
      device_problem_sizes_0.get(),
      device_problem_sizes_1.get(),
      device_ref_A0.get(),
      device_ref_B0.get(),
      device_ref_C0.get(),
      device_ref_Scale0.get(),
      device_ref_Bias0.get(),
      device_ref_B1.get(),
      device_ref_C1.get(),
      device_ref_D1.get(),
      {alpha0, beta0},
      {alpha1, beta1},
      threadblock_count
    };
```

**EN**: This line starts a function or callable scope. The body that follows typically configures a kernel, launches it, or checks its result.  
**CN**: 这一行开始定义一个函数或可调用作用域。随后的函数体通常会配置内核、启动执行或检查结果。

### Lines 301-301 / 第301-301行

```cpp
    cutlass::Status status = b2b_gemm_op.can_implement(arguments);
```

**EN**: This block exercises the configured CUTLASS object at runtime. Typical calls include initialization, capability checks, workspace queries, kernel launches, or reference validation.  
**CN**: 这段代码在运行时驱动已配置好的 CUTLASS 对象。常见调用包括初始化、能力检查、工作区查询、内核启动以及参考结果验证。

### Lines 303-309 / 第303-309行

```cpp
    if(status != cutlass::Status::kSuccess) {
        std::cout << "Problem sizes not supported.\n"
                << "Requirments:\n"
                << "    problem_size_0.M = problem_size_1.M\n"
                << "    problem_size_0.N = problem_size_1.K\n"
                << "    ThreadblockShape0::kN = problem_size_0.N\n"
                << "    ThreadblockShape1::kN = problem_size_1.N" << std::endl;
```

**EN**: This control-flow block selects a path based on runtime state, such as argument validation, loop traversal, or pass/fail reporting. `ThreadblockShape` selects the CTA-sized tile computed by one thread block.  
**CN**: 这个控制流代码块会根据运行时状态选择执行路径，例如参数校验、循环遍历或通过/失败报告。 `ThreadblockShape` 选择单个线程块负责计算的 CTA 级分块。

### Lines 310-310 / 第310-310行

```cpp
    }
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 312-312 / 第312-312行

```cpp
    status = b2b_gemm_op.initialize(arguments);
```

**EN**: This block exercises the configured CUTLASS object at runtime. Typical calls include initialization, capability checks, workspace queries, kernel launches, or reference validation.  
**CN**: 这段代码在运行时驱动已配置好的 CUTLASS 对象。常见调用包括初始化、能力检查、工作区查询、内核启动以及参考结果验证。

### Lines 314-314 / 第314-314行

```cpp
    CUTLASS_CHECK(status);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 316-319 / 第316-319行

```cpp
    for(int i = 0; i < warm_ups; i++) {
        status = b2b_gemm_op();
        CUTLASS_CHECK(status);
    }
```

**EN**: This control-flow block selects a path based on runtime state, such as argument validation, loop traversal, or pass/fail reporting.  
**CN**: 这个控制流代码块会根据运行时状态选择执行路径，例如参数校验、循环遍历或通过/失败报告。

### Lines 321-323 / 第321-323行

```cpp
    //
    // Run the GEMM
    //
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

### Lines 325-325 / 第325-325行

```cpp
    cudaEvent_t start, stop;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 326-326 / 第326-326行

```cpp
    cudaEventCreate(&start);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 327-327 / 第327-327行

```cpp
    cudaEventCreate(&stop);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 329-329 / 第329-329行

```cpp
    cudaEventRecord(start);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 331-334 / 第331-334行

```cpp
    for(int i = 0; i < runs; i++) {
        status = b2b_gemm_op();
        CUTLASS_CHECK(status);
    }
```

**EN**: This control-flow block selects a path based on runtime state, such as argument validation, loop traversal, or pass/fail reporting.  
**CN**: 这个控制流代码块会根据运行时状态选择执行路径，例如参数校验、循环遍历或通过/失败报告。

### Lines 336-336 / 第336-336行

```cpp
    cudaEventRecord(stop);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 337-337 / 第337-337行

```cpp
    cudaDeviceSynchronize();
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 338-338 / 第338-338行

```cpp
    float gemmTime;
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 339-339 / 第339-339行

```cpp
    cudaEventElapsedTime(&gemmTime, start, stop);
```

**EN**: This block contributes one logical step of the file, such as wiring together types, state, helper objects, or execution flow.  
**CN**: 这段代码构成了文件中的一个逻辑步骤，例如连接类型、状态、辅助对象或执行流程。

### Lines 340-340 / 第340-340行

```cpp
    std::cout << "Fusion time " << gemmTime / (float)runs << " ms\n";
```

**EN**: This block prints human-readable status so it is clear which configuration is running and whether validation passed.  
**CN**: 这段代码打印可读的状态信息，让用户清楚当前运行的是哪种配置，以及验证是否通过。

### Lines 342-430 / 第342-430行

```cpp
    for (int i = 0; i < problem_count; ++i) {
      host_tensor_D1.at(i).sync_host();
      //
      // Verify
      //
      cutlass::reference::device::Gemm<
          typename B2bGemm::ElementA, typename B2bGemm::LayoutA,
          typename B2bGemm::ElementB, typename B2bGemm::LayoutB,
          ElementAccumulator, typename B2bGemm::LayoutC, 
          ElementAccumulator, ElementAccumulator>
          reference_gemm_0;
      cutlass::reference::device::Gemm<
          typename B2bGemm::ElementA, typename B2bGemm::LayoutA,
          typename B2bGemm::ElementB, typename B2bGemm::LayoutB,
          typename B2bGemm::ElementC, typename B2bGemm::LayoutC, ElementCompute,
          ElementAccumulator>
          reference_gemm_1;
      auto problem_size_0 = problem_sizes_0[i];
      auto problem_size_1 = problem_sizes_1[i];
      reference_gemm_0(
        problem_size_0,
        ElementAccumulator(1), //intermediate alpha=1
        ref_A0.at(i), 
        ref_B0.at(i), 
        ElementAccumulator(0), //beta = 0
        ref_Z.at(i),
        ref_Z.at(i),
        ElementAccumulator(0)
      );
      cutlass::reference::device::TensorScaleBiasGemm<
        ElementAccumulator, typename B2bGemm::ElementC, typename B2bGemm::LayoutC,
        ElementCompute, typename B2bGemm::LayoutC
      > (
        problem_size_0,
        ref_Z.at(i),
        ref_ref_D0.at(i),
        alpha0,
        ref_Scale0.at(i),
        ref_Bias0.at(i)
      );
      if(relu) {
        cutlass::reference::device::TensorReLu(host_tensor_ref_D0.at(i).device_view()); 
      }
      reference_gemm_1(
        problem_size_1,
        alpha1, 
        ref_ref_D0.at(i), 
        ref_B1.at(i), 
        beta1, 
        {host_tensor_Bias1.at(i).device_data(), typename B2bGemm::LayoutC::Stride(0)},
        ref_ref_D1.at(i)
      );
      if(relu) {
        cutlass::reference::device::TensorReLu(host_tensor_ref_D1.at(i).device_view()); 
      }
      cudaDeviceSynchronize();
      host_tensor_ref_D0.at(i).sync_host();
      host_tensor_ref_D1.at(i).sync_host();
      CHECK_GT(cutlass::reference::host::TensorNorm(host_tensor_ref_D0.at(i).host_view()), 0);
      CHECK_GT(cutlass::reference::host::TensorNorm(host_tensor_D1.at(i).host_view()), 0);
      CHECK_GT(cutlass::reference::host::TensorNorm(host_tensor_ref_D1.at(i).host_view()), 0);
      bool passed = cutlass::reference::host::TensorEquals(
        host_tensor_ref_D1.at(i).host_view(), 
        host_tensor_D1.at(i).host_view());
      CHECK_TRUE(passed);
      if (!passed)
      {
        std::stringstream fname;
        fname << "error_B2bGemm_device_fused.txt";
        std::cerr << "Check failed for GEMM " << i << " in the group." << std::endl;
        std::cerr << "Dumping results in " << fname.str() << "\n";
        std::ofstream file(fname.str());
        file 
          << "GEMM " << i << " in group\n"
          << "A0 =\n" << host_tensor_A0.at(i).host_view()
          << "\nB0 =\n" << host_tensor_B0.at(i).host_view()
          << "\nC0 =\n" << host_tensor_C0.at(i).host_view()
          << "\nScale0:\n" << host_tensor_Scale0.at(i).host_view() << "\n"
          << "\nBias0:\n" << host_tensor_Bias0.at(i).host_view() << "\n"
          << "\nB1 =\n" << host_tensor_B1.at(i).host_view()
          << "\nC1 =\n" << host_tensor_C1.at(i).host_view()
          << "\nBias1:\n" << host_tensor_Bias1.at(i).host_view() << "\n"
          << "\n\nReference =\n" << host_tensor_ref_D1.at(i).host_view()
          << "\nComputed =\n" << host_tensor_D1.at(i).host_view();
        return false;
      }
    }
    return true;
  }
```

**EN**: This control-flow block selects a path based on runtime state, such as argument validation, loop traversal, or pass/fail reporting. `cutlass::gemm::device::Gemm` packages layouts, types, tile sizes, epilogue policy, swizzle, and stage count into a callable kernel object.  
**CN**: 这个控制流代码块会根据运行时状态选择执行路径，例如参数校验、循环遍历或通过/失败报告。 `cutlass::gemm::device::Gemm` 会把布局、类型、分块大小、epilogue 策略、swizzle 与流水级数打包成可调用的内核对象。

### Lines 448-448 / 第448-448行

```cpp
};
```

**EN**: This closes the current scope or namespace and keeps the surrounding structure balanced.  
**CN**: 这里关闭当前作用域或命名空间，以保持外围结构的配对完整。

### Lines 450-450 / 第450-450行

```cpp
////////////////////////////////////////////////////////////////////////////////
```

**EN**: This comment separates sections and documents the purpose of the code that follows.  
**CN**: 这段注释用于分隔不同章节，并说明后续代码的用途。

---

## Key Concepts / 关键概念

- Back-to-back GEMM fusion / 前后相接 GEMM 融合
- Grouped scheduling for heterogeneous problems / 面向异构问题的分组调度
- Hierarchical tiling: threadblock / warp / instruction / 分层分块：threadblock / warp / instruction
- Epilogue fusion and output operators / Epilogue 融合与输出算子

## Dependencies / 依赖项

- `<iostream>` — Console stream utilities for logging progress and results. / 用于记录进度和结果的控制台流工具。
- `<fstream>` — File stream utilities for reading or writing benchmark data. / 用于读写基准数据的文件流工具。
- `<sstream>` — String-stream helpers for assembling formatted text. / 用于拼接格式化文本的字符串流工具。
- `"cutlass/util/device_memory.h"` — RAII-style device-memory helpers. / RAII 风格的设备内存辅助工具。
- `"cutlass/util/host_tensor.h"` — Host/device tensor wrapper used to allocate storage and transfer data. / 主机/设备张量封装，用于分配存储并传输数据。
- `"cutlass/util/tensor_view_io.h"` — Tensor printing helpers for debugging layouts and values. / 用于调试布局和值的张量打印辅助工具。
- `"cutlass/util/distribution.h"` — Random-data distribution utilities for initializing test tensors. / 用于初始化测试张量的随机分布工具。
- `"cutlass/util/reference/host/tensor_fill.h"` — Host-side reference implementation used for correctness checking. / 用于正确性检查的主机端参考实现。
- `"cutlass/util/reference/host/tensor_copy.h"` — Host-side reference implementation used for correctness checking. / 用于正确性检查的主机端参考实现。
- `"cutlass/util/reference/host/tensor_compare.h"` — Host-side reference implementation used for correctness checking. / 用于正确性检查的主机端参考实现。
- `"cutlass/util/reference/host/tensor_norm.h"` — Host-side reference implementation used for correctness checking. / 用于正确性检查的主机端参考实现。
- `"cutlass/util/reference/device/gemm.h"` — Device-side reference helper used in validation flows. / 验证流程中使用的设备端参考辅助工具。
- `"cutlass/util/reference/device/tensor_relu.h"` — Device-side reference helper used in validation flows. / 验证流程中使用的设备端参考辅助工具。
- `"reference/device/tensor_scale_bias.h"` — Example-local reference helper used to validate fused results. / 示例本地参考辅助组件，用于验证融合结果。
- `"helper.h"` — Provides `helper.h` so this file can use the related API or helper utilities. / 提供 `helper.h`，使本文件能够使用相关 API 或辅助工具。
