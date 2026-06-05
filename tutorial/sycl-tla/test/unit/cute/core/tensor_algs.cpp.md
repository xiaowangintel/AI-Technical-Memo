# tensor_algs.cpp — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `test/unit/cute/core/tensor_algs.cpp`
- **EN:** Core tests for tensor algorithms such as copy, transform, and reduction-style helpers.
- **CN:** 本文件围绕 `tensor_algs` 相关功能编写单元测试或辅助基架。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30
```cpp
   1: /***************************************************************************************************
   2:  * Copyright (c) 2017 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
   3:  * SPDX-License-Identifier: BSD-3-Clause
   4:  *
   5:  * Redistribution and use in source and binary forms, with or without
   6:  * modification, are permitted provided that the following conditions are met:
   7:  *
   8:  * 1. Redistributions of source code must retain the above copyright notice, this
   9:  * list of conditions and the following disclaimer.
  10:  *
  11:  * 2. Redistributions in binary form must reproduce the above copyright notice,
  12:  * this list of conditions and the following disclaimer in the documentation
  13:  * and/or other materials provided with the distribution.
  14:  *
  15:  * 3. Neither the name of the copyright holder nor the names of its
  16:  * contributors may be used to endorse or promote products derived from
  17:  * this software without specific prior written permission.
  18:  *
  19:  * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
  20:  * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
  21:  * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
  22:  * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
  23:  * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
  24:  * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
  25:  * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
  26:  * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
  27:  * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
  28:  * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
  29:  *
  30:  **************************************************************************************************/
```
**EN:** Records copyright ownership and the BSD-3-Clause licensing terms for this source file.
**CN:** 记录该源文件的版权归属以及 BSD-3-Clause 许可条款。

### Lines 32
```cpp
  32: #include "cutlass_unit_test.h"
```
**EN:** Pulls in the unit-test harness together with the standard, CuTe, or CUTLASS headers used below.
**CN:** 引入单元测试框架，以及下文使用的标准库、CuTe 或 CUTLASS 头文件。

### Lines 34-36
```cpp
  34: #include <cute/algorithm/tensor_algorithms.hpp>
  35: #include <cute/algorithm/tensor_reduce.hpp>
  36: #include <cute/numeric/complex.hpp>
```
**EN:** Pulls in the unit-test harness together with the standard, CuTe, or CUTLASS headers used below.
**CN:** 引入单元测试框架，以及下文使用的标准库、CuTe 或 CUTLASS 头文件。

### Lines 38-48
```cpp
  38: TEST(CuTe_algorithm, TensorTransform) {
  39:   using namespace cute;
  40:   complex<float> array[4] = {{0,0}, {1,0}, {0,1}, {1,1}};
  41:   complex<float> correct[4] = {{0,0}, {1,0}, {0,-1}, {1,-1}};
  42:   Tensor tensor = make_tensor(static_cast<complex<float>*>(array), make_layout(make_shape(4)));
  43:   conjugate conj;
  44:   transform(tensor, conj);
  45:   for (int i = 0; i < 4; ++i) {
  46:     EXPECT_EQ(tensor(i), correct[i]);
  47:   }
  48: }
```
**EN:** Introduces namespace aliases or using-directives to keep later template code readable. Wraps raw storage in CuTe tensor views so later code can use explicit shape and stride semantics. Builds layout metadata that maps logical coordinates onto physical memory addresses. Validates results immediately so the test fails close to the source of an error.
**CN:** 引入命名空间别名或 using 声明，提升后续模板代码的可读性。 将原始存储包装成 CuTe 张量视图，使后续代码能够按显式形状与步幅访问数据。 构造布局元数据，把逻辑坐标映射到实际内存地址。 通过即时断言验证结果，使测试在最接近错误源的位置失败。

### Lines 50-51
```cpp
  50: TEST(CuTe_algorithm, TensorBatchReduce) {
  51:   using namespace cute;
```
**EN:** Introduces namespace aliases or using-directives to keep later template code readable.
**CN:** 引入命名空间别名或 using 声明，提升后续模板代码的可读性。

### Lines 53-56
```cpp
  53:   int src_vals[16] = {0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15};
  54:   Tensor src_tensor = make_tensor(static_cast<int*>(src_vals),
  55:                                   make_layout(make_shape (make_shape (2,2), make_shape (2,2)),
  56:                                               make_stride(make_stride(2,8), make_stride(1,4))));
```
**EN:** Continues the procedural logic of the current helper or test case. Wraps raw storage in CuTe tensor views so later code can use explicit shape and stride semantics. Builds layout metadata that maps logical coordinates onto physical memory addresses.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 将原始存储包装成 CuTe 张量视图，使后续代码能够按显式形状与步幅访问数据。 构造布局元数据，把逻辑坐标映射到实际内存地址。

### Lines 58-60
```cpp
  58:   array<int, 4> dst_vals;
  59:   fill(dst_vals, 0);
  60:   Tensor dst_tensor = make_tensor(dst_vals.begin(), make_shape(2,2));
```
**EN:** Continues the procedural logic of the current helper or test case. Wraps raw storage in CuTe tensor views so later code can use explicit shape and stride semantics. Builds layout metadata that maps logical coordinates onto physical memory addresses. Initializes deterministic or randomized input data used by the test scenario.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 将原始存储包装成 CuTe 张量视图，使后续代码能够按显式形状与步幅访问数据。 构造布局元数据，把逻辑坐标映射到实际内存地址。 初始化测试场景所需的确定性或随机输入数据。

### Lines 62
```cpp
  62:   batch_reduce(src_tensor, dst_tensor);
```
**EN:** Continues the procedural logic of the current helper or test case.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。

### Lines 64-69
```cpp
  64:   int correct[4] = {20,24,36,40};
  65:   for (int i = 0; i < 4; ++i) {
  66:     //printf("%d  %d\n", dst_tensor(i), correct[i]);
  67:     EXPECT_EQ(dst_tensor(i), correct[i]);
  68:   }
  69: }
```
**EN:** Performs validation and reports mismatches through compile-time or runtime assertions. Validates results immediately so the test fails close to the source of an error. Initializes deterministic or randomized input data used by the test scenario.
**CN:** 执行校验，并通过编译期或运行期断言报告不匹配。 通过即时断言验证结果，使测试在最接近错误源的位置失败。 初始化测试场景所需的确定性或随机输入数据。

### Lines 72-73
```cpp
  72: TEST(CuTe_algorithm, TensorLogicalReduce) {
  73:   using namespace cute;
```
**EN:** Introduces namespace aliases or using-directives to keep later template code readable.
**CN:** 引入命名空间别名或 using 声明，提升后续模板代码的可读性。

### Lines 75-80
```cpp
  75:   { // Reduce each column of a matrix
  76:     Tensor src_tensor = make_tensor(counting_iterator<int>{0},
  77:                                     Layout<Shape <_32, Shape <_12,_6>>,
  78:                                            Stride< _1, Stride<_64,_1>>>{});
  79:     auto slicer = make_coord(0_c, _);
  80:     Tensor dst_tensor = make_tensor_like(src_tensor(slicer));
```
**EN:** Continues the procedural logic of the current helper or test case. Wraps raw storage in CuTe tensor views so later code can use explicit shape and stride semantics. Builds layout metadata that maps logical coordinates onto physical memory addresses.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 将原始存储包装成 CuTe 张量视图，使后续代码能够按显式形状与步幅访问数据。 构造布局元数据，把逻辑坐标映射到实际内存地址。

### Lines 82
```cpp
  82:     logical_reduce(src_tensor, dst_tensor, slicer);
```
**EN:** Continues the procedural logic of the current helper or test case.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。

### Lines 84-87
```cpp
  84:     for (int i = 0; i < size(dst_tensor); ++i) {
  85:       EXPECT_EQ(dst_tensor(i), reduce(src_tensor(_,i), int(0)));
  86:     }
  87:   }
```
**EN:** Performs validation and reports mismatches through compile-time or runtime assertions. Validates results immediately so the test fails close to the source of an error. Initializes deterministic or randomized input data used by the test scenario.
**CN:** 执行校验，并通过编译期或运行期断言报告不匹配。 通过即时断言验证结果，使测试在最接近错误源的位置失败。 初始化测试场景所需的确定性或随机输入数据。

### Lines 89-94
```cpp
  89:   { // Reduce each row of a matrix
  90:     Tensor src_tensor = make_tensor(counting_iterator<int>{0},
  91:                                     Layout<Shape <_32, Shape <_12,_6>>,
  92:                                            Stride< _1, Stride<_64,_1>>>{});
  93:     auto slicer = make_coord(_, 0_c);
  94:     Tensor dst_tensor = make_tensor_like(src_tensor(slicer));
```
**EN:** Continues the procedural logic of the current helper or test case. Wraps raw storage in CuTe tensor views so later code can use explicit shape and stride semantics. Builds layout metadata that maps logical coordinates onto physical memory addresses.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 将原始存储包装成 CuTe 张量视图，使后续代码能够按显式形状与步幅访问数据。 构造布局元数据，把逻辑坐标映射到实际内存地址。

### Lines 96
```cpp
  96:     logical_reduce(src_tensor, dst_tensor, slicer);
```
**EN:** Continues the procedural logic of the current helper or test case.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。

### Lines 98-101
```cpp
  98:     for (int i = 0; i < size(dst_tensor); ++i) {
  99:       EXPECT_EQ(dst_tensor(i), reduce(src_tensor(i,_), int(0)));
 100:     }
 101:   }
```
**EN:** Performs validation and reports mismatches through compile-time or runtime assertions. Validates results immediately so the test fails close to the source of an error. Initializes deterministic or randomized input data used by the test scenario.
**CN:** 执行校验，并通过编译期或运行期断言报告不匹配。 通过即时断言验证结果，使测试在最接近错误源的位置失败。 初始化测试场景所需的确定性或随机输入数据。

### Lines 103-108
```cpp
 103:   { // 1 profile
 104:     Tensor src_tensor = make_tensor(counting_iterator<int>{0},
 105:                                     Layout<Shape<_32>, Stride<_1>>{});
 106:     array<int, 1> dst_vals;
 107:     fill(dst_vals, 0);
 108:     Tensor dst_tensor = make_tensor(dst_vals.begin(), Layout<_1,_0>{});
```
**EN:** Continues the procedural logic of the current helper or test case. Wraps raw storage in CuTe tensor views so later code can use explicit shape and stride semantics. Builds layout metadata that maps logical coordinates onto physical memory addresses. Initializes deterministic or randomized input data used by the test scenario.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 将原始存储包装成 CuTe 张量视图，使后续代码能够按显式形状与步幅访问数据。 构造布局元数据，把逻辑坐标映射到实际内存地址。 初始化测试场景所需的确定性或随机输入数据。

### Lines 110
```cpp
 110:     logical_reduce(src_tensor, dst_tensor, 1);
```
**EN:** Continues the procedural logic of the current helper or test case.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。

### Lines 112-115
```cpp
 112:     for (int i = 0; i < size(dst_tensor); ++i) {
 113:       EXPECT_EQ(dst_tensor(i), reduce(src_tensor, int(0)));
 114:     }
 115:   }
```
**EN:** Performs validation and reports mismatches through compile-time or runtime assertions. Validates results immediately so the test fails close to the source of an error. Initializes deterministic or randomized input data used by the test scenario.
**CN:** 执行校验，并通过编译期或运行期断言报告不匹配。 通过即时断言验证结果，使测试在最接近错误源的位置失败。 初始化测试场景所需的确定性或随机输入数据。

### Lines 117-121
```cpp
 117:   { // _ profile
 118:     Tensor src_tensor = make_tensor(counting_iterator<int>{0},
 119:                                     Layout<Shape<_32>, Stride<_1>>{});
 120:     auto slicer = _;
 121:     Tensor dst_tensor = make_tensor_like(src_tensor(slicer));
```
**EN:** Continues the procedural logic of the current helper or test case. Wraps raw storage in CuTe tensor views so later code can use explicit shape and stride semantics. Builds layout metadata that maps logical coordinates onto physical memory addresses.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 将原始存储包装成 CuTe 张量视图，使后续代码能够按显式形状与步幅访问数据。 构造布局元数据，把逻辑坐标映射到实际内存地址。

### Lines 123
```cpp
 123:     logical_reduce(src_tensor, dst_tensor, slicer);
```
**EN:** Continues the procedural logic of the current helper or test case.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。

### Lines 125-128
```cpp
 125:     for (int i = 0; i < size(dst_tensor); ++i) {
 126:       EXPECT_EQ(dst_tensor(i), src_tensor(i));
 127:     }
 128:   }
```
**EN:** Performs validation and reports mismatches through compile-time or runtime assertions. Validates results immediately so the test fails close to the source of an error. Initializes deterministic or randomized input data used by the test scenario.
**CN:** 执行校验，并通过编译期或运行期断言报告不匹配。 通过即时断言验证结果，使测试在最接近错误源的位置失败。 初始化测试场景所需的确定性或随机输入数据。

### Lines 130-137
```cpp
 130:   { // (1,1) profile
 131:     Tensor src_tensor = make_tensor(counting_iterator<int>{0},
 132:                                     Layout<Shape <_32, Shape <_12,_6>>,
 133:                                            Stride< _1, Stride<_192,_32>>>{});
 134:     auto slicer = make_coord(1, 1);
 135:     array<int, 1> dst_vals;
 136:     fill(dst_vals, 0);
 137:     Tensor dst_tensor = make_tensor(dst_vals.begin(), Layout<_1,_0>{});
```
**EN:** Continues the procedural logic of the current helper or test case. Wraps raw storage in CuTe tensor views so later code can use explicit shape and stride semantics. Builds layout metadata that maps logical coordinates onto physical memory addresses. Initializes deterministic or randomized input data used by the test scenario.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 将原始存储包装成 CuTe 张量视图，使后续代码能够按显式形状与步幅访问数据。 构造布局元数据，把逻辑坐标映射到实际内存地址。 初始化测试场景所需的确定性或随机输入数据。

### Lines 139
```cpp
 139:     logical_reduce(src_tensor, dst_tensor, slicer);
```
**EN:** Continues the procedural logic of the current helper or test case.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。

### Lines 141-144
```cpp
 141:     for (int i = 0; i < size(dst_tensor); ++i) {
 142:       EXPECT_EQ(dst_tensor(i), reduce(src_tensor, int(0)));
 143:     }
 144:   }
```
**EN:** Performs validation and reports mismatches through compile-time or runtime assertions. Validates results immediately so the test fails close to the source of an error. Initializes deterministic or randomized input data used by the test scenario.
**CN:** 执行校验，并通过编译期或运行期断言报告不匹配。 通过即时断言验证结果，使测试在最接近错误源的位置失败。 初始化测试场景所需的确定性或随机输入数据。

### Lines 146-151
```cpp
 146:   { // (_,_) profile
 147:     Tensor src_tensor = make_tensor(counting_iterator<int>{0},
 148:                                     Layout<Shape <_32, Shape <_12,_6>>,
 149:                                            Stride< _1, Stride<_192,_32>>>{});
 150:     auto slicer = make_coord(_,_);
 151:     Tensor dst_tensor = make_tensor_like(src_tensor(slicer));
```
**EN:** Continues the procedural logic of the current helper or test case. Wraps raw storage in CuTe tensor views so later code can use explicit shape and stride semantics. Builds layout metadata that maps logical coordinates onto physical memory addresses.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 将原始存储包装成 CuTe 张量视图，使后续代码能够按显式形状与步幅访问数据。 构造布局元数据，把逻辑坐标映射到实际内存地址。

### Lines 153
```cpp
 153:     logical_reduce(src_tensor, dst_tensor, slicer);
```
**EN:** Continues the procedural logic of the current helper or test case.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。

### Lines 155-158
```cpp
 155:     for (int i = 0; i < size(dst_tensor); ++i) {
 156:       EXPECT_EQ(dst_tensor(i), src_tensor(i));
 157:     }
 158:   }
```
**EN:** Performs validation and reports mismatches through compile-time or runtime assertions. Validates results immediately so the test fails close to the source of an error. Initializes deterministic or randomized input data used by the test scenario.
**CN:** 执行校验，并通过编译期或运行期断言报告不匹配。 通过即时断言验证结果，使测试在最接近错误源的位置失败。 初始化测试场景所需的确定性或随机输入数据。

### Lines 160-163
```cpp
 160:   {
 161:     Tensor src_tensor = make_tensor(counting_iterator<int>{0},
 162:                                     make_layout(make_shape (2,2,2,2),
 163:                                                 make_stride(1,2,4,8)));
```
**EN:** Continues the procedural logic of the current helper or test case. Wraps raw storage in CuTe tensor views so later code can use explicit shape and stride semantics. Builds layout metadata that maps logical coordinates onto physical memory addresses.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 将原始存储包装成 CuTe 张量视图，使后续代码能够按显式形状与步幅访问数据。 构造布局元数据，把逻辑坐标映射到实际内存地址。

### Lines 165-167
```cpp
 165:     array<int, 4> dst_vals;
 166:     fill(dst_vals, 0);
 167:     Tensor dst_tensor = make_tensor(dst_vals.begin(), make_shape(2,2));
```
**EN:** Continues the procedural logic of the current helper or test case. Wraps raw storage in CuTe tensor views so later code can use explicit shape and stride semantics. Builds layout metadata that maps logical coordinates onto physical memory addresses. Initializes deterministic or randomized input data used by the test scenario.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 将原始存储包装成 CuTe 张量视图，使后续代码能够按显式形状与步幅访问数据。 构造布局元数据，把逻辑坐标映射到实际内存地址。 初始化测试场景所需的确定性或随机输入数据。

### Lines 169
```cpp
 169:     auto target_profile = make_coord(_,1,_,1);
```
**EN:** Continues the procedural logic of the current helper or test case.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。

### Lines 171
```cpp
 171:     logical_reduce(src_tensor, dst_tensor, target_profile);
```
**EN:** Continues the procedural logic of the current helper or test case.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。

### Lines 173-178
```cpp
 173:     int correct[4] = {20,24,36,40};
 174:     for (int i = 0; i < 4; ++i) {
 175:       //printf("%d  %d\n", dst_tensor(i), correct[i]);
 176:       EXPECT_EQ(dst_tensor(i), correct[i]);
 177:     }
 178:   }
```
**EN:** Performs validation and reports mismatches through compile-time or runtime assertions. Validates results immediately so the test fails close to the source of an error. Initializes deterministic or randomized input data used by the test scenario.
**CN:** 执行校验，并通过编译期或运行期断言报告不匹配。 通过即时断言验证结果，使测试在最接近错误源的位置失败。 初始化测试场景所需的确定性或随机输入数据。

### Lines 180-183
```cpp
 180:   {
 181:     Tensor src_tensor = make_tensor(counting_iterator<int>{0},
 182:                                     make_layout(make_shape (2,make_shape (2,2),2),
 183:                                                 make_stride(1,make_stride(2,4),8)));
```
**EN:** Continues the procedural logic of the current helper or test case. Wraps raw storage in CuTe tensor views so later code can use explicit shape and stride semantics. Builds layout metadata that maps logical coordinates onto physical memory addresses.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 将原始存储包装成 CuTe 张量视图，使后续代码能够按显式形状与步幅访问数据。 构造布局元数据，把逻辑坐标映射到实际内存地址。

### Lines 185-187
```cpp
 185:     array<int, 4> dst_vals;
 186:     fill(dst_vals, 0);
 187:     Tensor dst_tensor = make_tensor(dst_vals.begin(), make_shape(2,2));
```
**EN:** Continues the procedural logic of the current helper or test case. Wraps raw storage in CuTe tensor views so later code can use explicit shape and stride semantics. Builds layout metadata that maps logical coordinates onto physical memory addresses. Initializes deterministic or randomized input data used by the test scenario.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。 将原始存储包装成 CuTe 张量视图，使后续代码能够按显式形状与步幅访问数据。 构造布局元数据，把逻辑坐标映射到实际内存地址。 初始化测试场景所需的确定性或随机输入数据。

### Lines 189
```cpp
 189:     auto target_profile = make_coord(_,make_coord(1,_),1);
```
**EN:** Continues the procedural logic of the current helper or test case.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。

### Lines 191
```cpp
 191:     logical_reduce(src_tensor, dst_tensor, target_profile);
```
**EN:** Continues the procedural logic of the current helper or test case.
**CN:** 继续执行当前辅助例程或测试用例的过程逻辑。

### Lines 193-200
```cpp
 193:     int correct[4] = {20,24,36,40};
 194:     for (int i = 0; i < 4; ++i) {
 195:       //printf("%d  %d\n", dst_tensor(i), correct[i]);
 196:       EXPECT_EQ(dst_tensor(i), correct[i]);
 197:     }
 198:   }
 200: }
```
**EN:** Performs validation and reports mismatches through compile-time or runtime assertions. Validates results immediately so the test fails close to the source of an error. Initializes deterministic or randomized input data used by the test scenario.
**CN:** 执行校验，并通过编译期或运行期断言报告不匹配。 通过即时断言验证结果，使测试在最接近错误源的位置失败。 初始化测试场景所需的确定性或随机输入数据。

## Key Concepts / 关键概念
- **EN:** Core CuTe metaprogramming coverage
  **CN:** 覆盖 CuTe 核心模板元编程与布局代数行为。
- **EN:** CuTe tensors and layouts
  **CN:** 使用 CuTe 张量与布局抽象描述数据形状、步幅和坐标映射。
- **EN:** Unit-test driven validation
  **CN:** 通过单元测试断言直接验证行为是否正确。

## Dependencies / 依赖关系
- **EN:** Direct headers: `"cutlass_unit_test.h"`, `<cute/algorithm/tensor_algorithms.hpp>`, `<cute/algorithm/tensor_reduce.hpp>`, `<cute/numeric/complex.hpp>`.
  **CN:** 直接头文件依赖：`"cutlass_unit_test.h"`, `<cute/algorithm/tensor_algorithms.hpp>`, `<cute/algorithm/tensor_reduce.hpp>`, `<cute/numeric/complex.hpp>`。
- **EN:** Primary test harness: `cutlass_unit_test.h` and GoogleTest-style assertion macros.
  **CN:** `cutlass_unit_test.h` 与 GoogleTest 风格断言宏构成主要测试框架。
- **EN:** Key APIs referenced here: `make_tensor`, `make_layout`, `make_shape`, `copy`, `EXPECT_EQ`.
  **CN:** 此处反复使用的关键 API：`make_tensor`, `make_layout`, `make_shape`, `copy`, `EXPECT_EQ`。
