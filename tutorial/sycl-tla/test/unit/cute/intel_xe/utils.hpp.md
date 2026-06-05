# utils.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `test/unit/cute/intel_xe/utils.hpp`
- **EN:** Common helper utilities for Intel Xe tests, including data generation and result verification.
- **CN:** 本文件围绕 `utils` 相关功能编写单元测试或辅助基架。

## Line-by-Line Analysis / 逐行分析
### Lines 1-31
```cpp
   1: /***************************************************************************************************
   2:  * Copyright (c) 2024 - 2024 Codeplay Software Ltd. All rights reserved.
   3:  * Copyright (C) 2025 Intel Corporation, All rights reserved.
   4:  * SPDX-License-Identifier: BSD-3-Clause
   5:  *
   6:  * Redistribution and use in source and binary forms, with or without
   7:  * modification, are permitted provided that the following conditions are met:
   8:  *
   9:  * 1. Redistributions of source code must retain the above copyright notice, this
  10:  * list of conditions and the following disclaimer.
  11:  *
  12:  * 2. Redistributions in binary form must reproduce the above copyright notice,
  13:  * this list of conditions and the following disclaimer in the documentation
  14:  * and/or other materials provided with the distribution.
  15:  *
  16:  * 3. Neither the name of the copyright holder nor the names of its
  17:  * contributors may be used to endorse or promote products derived from
  18:  * this software without specific prior written permission.
  19:  *
  20:  * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
  21:  * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
  22:  * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
  23:  * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
  24:  * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
  25:  * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
  26:  * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
  27:  * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
  28:  * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
  29:  * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
  30:  *
  31:  **************************************************************************************************/
```
**EN:** Records copyright ownership and the BSD-3-Clause licensing terms for this source file.
**CN:** 记录该源文件的版权归属以及 BSD-3-Clause 许可条款。

### Lines 33
```cpp
  33: #pragma once
```
**EN:** Marks this header as include-once so helper declarations are not processed repeatedly.
**CN:** 通过 include-once 语义避免该头文件被重复处理。

### Lines 35
```cpp
  35: #include "cutlass/detail/layout.hpp"
```
**EN:** Pulls in the unit-test harness together with the standard, CuTe, or CUTLASS headers used below.
**CN:** 引入单元测试框架，以及下文使用的标准库、CuTe 或 CUTLASS 头文件。

### Lines 37-39
```cpp
  37: #include <cute/tensor.hpp>
  38: #include <sycl/sycl.hpp>
  39: #include <cute/util/compat.hpp>
```
**EN:** Pulls in the unit-test harness together with the standard, CuTe, or CUTLASS headers used below.
**CN:** 引入单元测试框架，以及下文使用的标准库、CuTe 或 CUTLASS 头文件。

### Lines 41
```cpp
  41: #include "cutlass_unit_test.h"
```
**EN:** Pulls in the unit-test harness together with the standard, CuTe, or CUTLASS headers used below.
**CN:** 引入单元测试框架，以及下文使用的标准库、CuTe 或 CUTLASS 头文件。

### Lines 43-44
```cpp
  43: using namespace cute;
  44: using namespace cute::detail;
```
**EN:** Introduces namespace aliases or using-directives to keep later template code readable.
**CN:** 引入命名空间别名或 using 声明，提升后续模板代码的可读性。

### Lines 46-48
```cpp
  46: using namespace cutlass;
  47: using namespace cutlass::layout;
  48: using namespace cutlass::detail;
```
**EN:** Introduces namespace aliases or using-directives to keep later template code readable.
**CN:** 引入命名空间别名或 using 声明，提升后续模板代码的可读性。

### Lines 50
```cpp
  50: using namespace compat::experimental;
```
**EN:** Introduces namespace aliases or using-directives to keep later template code readable. Keeps a SYCL-compatible code path alongside CUDA-specific logic.
**CN:** 引入命名空间别名或 using 声明，提升后续模板代码的可读性。 在 CUDA 专用逻辑旁保留兼容 SYCL 的实现路径。

### Lines 52
```cpp
  52: #define SUBGROUP_SIZE (16)
```
**EN:** Continues the procedural logic of the current helper or test case.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。

### Lines 54-57
```cpp
  54: #define CUTLASS_ENABLE_DEBUG_PRINTS (0)
  55: #define LOG_GROUP (0)
  56: #undef LOG_THREAD
  57: #define LOG_THREAD (0)
```
**EN:** Continues the procedural logic of the current helper or test case.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。

### Lines 59-63
```cpp
  59: template <class atype, class btype, class ctype>
  60: void verify(uint32_t m, uint32_t n, uint32_t k, atype *A, btype *B, ctype *C,
  61:             bool row_a = true, bool row_b = true) {
  62:   int cnt = 0;
  63:   bool is_normal = true;
```
**EN:** Defines helper type `atype` used by the surrounding tests or kernels.
**CN:** 定义辅助类型 `atype`，供周围测试或内核复用。

### Lines 65-72
```cpp
  65:   for (int i = 0; i < m; i++) {
  66:     for (int j = 0; j < n; j++) {
  67:       ctype expect = ctype(0);
  68:       for (int z = 0; z < k; z++) {
  69:         auto a = row_a ? A[i * k + z] : A[i + z * m];
  70:         auto b = row_b ? B[z * n + j] : B[z + j * k];
  71:         expect += a * b;
  72:       }
```
**EN:** Continues the procedural logic of the current helper or test case. Initializes deterministic or randomized input data used by the test scenario.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 初始化测试场景所需的确定性或随机输入数据。

### Lines 74
```cpp
  74:       ctype val = C[i * n + j];
```
**EN:** Continues the procedural logic of the current helper or test case.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。

### Lines 76-92
```cpp
  76:       if constexpr(std::is_floating_point_v<ctype>) {
  77:         if (isnormal(val) && isnormal(expect)) {
  78:           auto error = std::abs((expect - val) / val);
  79:           if (error > 0.01f) {
  80:             cnt++;
  81:           }
  82:         } else {
  83:           // TODO(codeplay): Assert that at least some values are non-zero.
  84:           if(!(expect == 0 && val == 0)) is_normal = false;
  85:         }
  86:       } else {
  87:         if (val != expect) {
  88:           cnt++;
  89:         }
  90:       }
  91:     }
  92:   }
```
**EN:** Continues the procedural logic of the current helper or test case.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。

### Lines 94-96
```cpp
  94:   EXPECT_EQ(cnt, 0);
  95:   EXPECT_EQ(is_normal, true);
  96: }
```
**EN:** Performs validation and reports mismatches through compile-time or runtime assertions. Validates results immediately so the test fails close to the source of an error.
**CN:** 执行校验，并通过编译期或运行期断言报告不匹配。 通过即时断言验证结果，使测试在最接近错误源的位置失败。

### Lines 98-100
```cpp
  98: template <typename T> static void fill_matrix(cutlass::host_vector<T> &M) {
  99:   std::random_device dev;
 100:   std::mt19937 rng(dev());
```
**EN:** Introduces templated helper `fill_matrix` so one code path can cover many types or layouts. Allocates host/device buffers and moves test data between them. Initializes deterministic or randomized input data used by the test scenario.
**CN:** 引入模板辅助实体 `fill_matrix`，让同一代码路径覆盖多种类型或布局。 分配主机/设备缓冲区，并在两者之间搬运测试数据。 初始化测试场景所需的确定性或随机输入数据。

### Lines 102
```cpp
 102:   T start, end;
```
**EN:** Continues the procedural logic of the current helper or test case.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。

### Lines 104-116
```cpp
 104:   if constexpr (std::is_same_v<T, tfloat32_t> || std::is_same_v<T, half_t>
 105:                  || std::is_same_v<T, bfloat16_t> || std::is_same_v<T, float>) {
 106:     start = (T)0.0;
 107:     end = (T)1.0;
 108:   } else if constexpr (std::is_same_v<T, int8_t>) {
 109:     start = (T)(-5);
 110:     end = (T)5;
 111:   } else if constexpr (std::is_same_v<T, uint8_t>) {
 112:     start = (T)0;
 113:     end = (T)5;
 114:   } else {
 115:     CUTE_STATIC_ASSERT(false, "you must set coreect start/end value to initialize data");
 116:   }
```
**EN:** Continues the procedural logic of the current helper or test case.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。

### Lines 118-121
```cpp
 118:   std::uniform_real_distribution<float> dist((T)start, (T)end);
 119:   for (int i = 0; i < M.size(); i++)
 120:     M[i] = static_cast<T>(dist(rng));
 121: }
```
**EN:** Continues the procedural logic of the current helper or test case. Initializes deterministic or randomized input data used by the test scenario.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 初始化测试场景所需的确定性或随机输入数据。

### Lines 123
```cpp
 123: template <class kernel> void run(uint32_t m, uint32_t n, uint32_t k) {
```
**EN:** Defines helper type `kernel` used by the surrounding tests or kernels.
**CN:** 定义辅助类型 `kernel`，供周围测试或内核复用。

### Lines 125-127
```cpp
 125:   using TA = typename kernel::TA;
 126:   using TB = typename kernel::TB;
 127:   using TC = typename kernel::TC;
```
**EN:** Continues the procedural logic of the current helper or test case.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。

### Lines 129-131
```cpp
 129:   cutlass::host_vector<TA> h_A(m * k);
 130:   cutlass::host_vector<TB> h_B(n * k);
 131:   cutlass::host_vector<TC> h_C(m * n);
```
**EN:** Continues the procedural logic of the current helper or test case. Allocates host/device buffers and moves test data between them.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 分配主机/设备缓冲区，并在两者之间搬运测试数据。

### Lines 133-134
```cpp
 133:   fill_matrix(h_A);
 134:   fill_matrix(h_B);
```
**EN:** Continues the procedural logic of the current helper or test case.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。

### Lines 136-138
```cpp
 136:   cutlass::device_vector<TA> d_A = h_A;
 137:   cutlass::device_vector<TB> d_B = h_B;
 138:   cutlass::device_vector<TC> d_C = h_C;
```
**EN:** Continues the procedural logic of the current helper or test case. Allocates host/device buffers and moves test data between them.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 分配主机/设备缓冲区，并在两者之间搬运测试数据。

### Lines 140-144
```cpp
 140:   auto dimBlock = compat::dim3(
 141:       ceil_div(kernel::wg_tile_m, kernel::sg_tile_m),
 142:       SUBGROUP_SIZE * ceil_div(kernel::wg_tile_n, kernel::sg_tile_n));
 143:   auto dimGrid = compat::dim3(size(ceil_div(m, kernel::wg_tile_m)),
 144:                                   size(ceil_div(n, kernel::wg_tile_n)));
```
**EN:** Continues the procedural logic of the current helper or test case. Keeps a SYCL-compatible code path alongside CUDA-specific logic.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 在 CUDA 专用逻辑旁保留兼容 SYCL 的实现路径。

### Lines 146-149
```cpp
 146:   launch<kernel::func>(
 147:       launch_policy{dimGrid, dimBlock,
 148:                     kernel_properties{sycl_exp::sub_group_size<SUBGROUP_SIZE>}},
 149:       d_A.data(), d_B.data(), d_C.data(), m, n, k);
```
**EN:** Continues the procedural logic of the current helper or test case. Launches a device kernel through the SYCL or CUDA execution path.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 通过 SYCL 或 CUDA 执行路径启动设备内核。

### Lines 151
```cpp
 151:   compat::wait();
```
**EN:** Continues the procedural logic of the current helper or test case. Keeps a SYCL-compatible code path alongside CUDA-specific logic.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 在 CUDA 专用逻辑旁保留兼容 SYCL 的实现路径。

### Lines 153-156
```cpp
 153:   h_C = d_C;
 154:   verify(m, n, k, h_A.data(), h_B.data(), h_C.data(), kernel::is_a_row_major,
 155:          kernel::is_b_row_major);
 156: }
```
**EN:** Continues the procedural logic of the current helper or test case.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。

## Key Concepts / 关键概念
- **EN:** Intel Xe plus SYCL backend coverage
  **CN:** 覆盖 Intel Xe 与 SYCL 后端相关行为。
- **EN:** CuTe tensors and layouts
  **CN:** 使用 CuTe 张量与布局抽象描述数据形状、步幅和坐标映射。
- **EN:** Unit-test driven validation
  **CN:** 通过单元测试断言直接验证行为是否正确。
- **EN:** Host/device round-trip checking
  **CN:** 通过主机与设备之间的往返数据检查结果正确性。

## Dependencies / 依赖关系
- **EN:** Direct headers: `"cutlass/detail/layout.hpp"`, `<cute/tensor.hpp>`, `<sycl/sycl.hpp>`, `<cute/util/compat.hpp>`, `"cutlass_unit_test.h"`.
  **CN:** 直接头文件依赖：`"cutlass/detail/layout.hpp"`, `<cute/tensor.hpp>`, `<sycl/sycl.hpp>`, `<cute/util/compat.hpp>`, `"cutlass_unit_test.h"`。
- **EN:** Primary test harness: `cutlass_unit_test.h` and GoogleTest-style assertion macros.
  **CN:** `cutlass_unit_test.h` 与 GoogleTest 风格断言宏构成主要测试框架。
- **EN:** Key APIs referenced here: `copy`, `EXPECT_EQ`.
  **CN:** 此处反复使用的关键 API：`copy`, `EXPECT_EQ`。
