# rewrite-tensor-descriptor-to-pointer.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/Triton/rewrite-tensor-descriptor-to-pointer.mlir`
- **EN:** Test for the MLIR/Triton pipeline `--triton-rewrite-tensor-descriptor-to-pointer, --canonicalize, --cse, --mlir-print-debuginfo` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `--triton-rewrite-tensor-descriptor-to-pointer, --canonicalize, --cse, --mlir-print-debuginfo` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s --triton-rewrite-tensor-descriptor-to-pointer --canonicalize --cse --mlir-print-debuginfo --split-input-file | FileCheck %s --implicit-check-not \!tt.tensordesc`
- **CN:** RUN 流水线：`// RUN: triton-opt %s --triton-rewrite-tensor-descriptor-to-pointer --canonicalize --cse --mlir-print-debuginfo --split-input-file | FileCheck %s --implicit-check-not \!tt.tensordesc`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt %s --triton-rewrite-tensor-descriptor-to-pointer --canonicalize --cse --mlir-print-debuginfo --split-input-file | FileCheck %s --implicit-check-not \!tt.tensordesc
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s --triton-rewrite-tensor-descriptor-to-pointer --canonicalize --cse --mlir-print-debuginfo --split-input-file | FileCheck %s --implicit-check-not \!tt.tensordesc` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s --triton-rewrite-tensor-descriptor-to-pointer --canonicalize --cse --mlir-print-debuginfo --split-input-file | FileCheck %s --implicit-check-not \!tt.tensordesc`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-3
```mlir
3| module {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 4-15
```mlir
 4|   tt.func public @load(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg1: i32, %arg2: i32) -> (tensor<128x128xf32>) {
 5|     %c1_i64 = arith.constant 1 : i64
 6|     %c256_i64 = arith.constant 256 : i64
 7|     %c0_i32 = arith.constant 0 : i32
 8|     %c128_i32 = arith.constant 128 : i32
 9|     %c256_i32 = arith.constant 256 : i32
10|     %0 = tt.make_tensor_descriptor %arg0, [%c256_i32, %c256_i32], [%c1_i64, %c256_i64] {order = array<i32: 0>} : <f32>, <128x128xf32>
11|     %3 = tt.descriptor_load %0[%arg1, %arg2] : !tt.tensordesc<128x128xf32> -> tensor<128x128xf32>
12|     tt.return %3 : tensor<128x128xf32>
13|   }
14| }
15| 
```
**EN:** This function-oriented block defines or enters `load`. Within it, the test exercises constants, tt.func, tt.make_tensor_descriptor, tt.descriptor_load, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `load` 为核心。测试在其中演示 常量、tt.func、tt.make_tensor_descriptor、tt.descriptor_load、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 16-33
```mlir
16| // CHECK-LABEL: @load
17| // CHECK-SAME: %[[ARG0:[^:]*]]
18| // CHECK-SAME: %[[ARG1:[^:]*]]
19| // CHECK-SAME: %[[ARG2:[^:]*]]
20| // CHECK-DAG: %[[CST:.*]] = arith.constant dense<0.000000e+00> : tensor<128x128xf32>
21| // CHECK-DAG: %[[CST0:.*]] = arith.constant dense<0> : tensor<1x128xi64>
22| // CHECK-DAG: %[[CST1:.*]] = arith.constant dense<256> : tensor<128x1xi64>
23| // CHECK-DAG: %[[CST2:.*]] = arith.constant dense<0> : tensor<128x1xi64>
24| // CHECK-DAG: %[[CST3:.*]] = arith.constant dense<256> : tensor<1x128xi64>
25| 
26| // CHECK-DAG: %[[VAL0:.*]] = arith.extsi %[[ARG1]] : i32 to i64
27| // CHECK-DAG: %[[VAL1:.*]] = arith.extsi %[[ARG2]] : i32 to i64
28| // CHECK-DAG: %[[VAL2:.*]] = tt.splat %[[ARG0]] :
29| // CHECK-DAG: %[[VAL3:.*]] = tt.splat %[[VAL0]] :
30| // CHECK-DAG: %[[VAL4:.*]] = tt.make_range {end = 128 : i32, start = 0 : i32}
31| // CHECK-DAG: %[[VAL5:.*]] = arith.extsi %[[VAL4]] :
32| // CHECK-DAG: %[[VAL6:.*]] = arith.addi %[[VAL3]], %[[VAL5]] :
33| // CHECK-DAG: %[[VAL7:.*]] = tt.expand_dims %[[VAL6]] {axis = 1 : i32}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @load anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @load 这样的标签用于锚定匹配范围。

### Lines 34-51
```mlir
34| // CHECK-DAG: %[[VAL8:.*]] = tt.broadcast %[[VAL7]] : tensor<128x1xi64> -> tensor<128x128xi64>
35| // CHECK-DAG: %[[VAL9:.*]] = tt.addptr %[[VAL2]], %[[VAL8]] :
36| // CHECK-DAG: %[[VAL10:.*]] = tt.splat %[[VAL1]] :
37| // CHECK-DAG: %[[VAL11:.*]] = arith.addi %[[VAL10]], %[[VAL5]] :
38| // CHECK-DAG: %[[VAL12:.*]] = tt.expand_dims %[[VAL11]] {axis = 0 : i32}
39| // CHECK-DAG: %[[VAL13:.*]] = arith.muli %[[VAL12]], %[[CST3]] :
40| // CHECK-DAG: %[[VAL14:.*]] = tt.broadcast %[[VAL13]] : tensor<1x128xi64> -> tensor<128x128xi64>
41| // CHECK-DAG: %[[VAL15:.*]] = tt.addptr %[[VAL9]], %[[VAL14]] :
42| 
43| // CHECK-DAG: %[[VAL16:.*]] = arith.cmpi sge, %[[VAL7]], %[[CST2]]
44| // CHECK-DAG: %[[VAL17:.*]] = arith.cmpi slt, %[[VAL7]], %[[CST1]]
45| // CHECK-DAG: %[[VAL18:.*]] = arith.andi %[[VAL16]], %[[VAL17]]
46| // CHECK-DAG: %[[VAL19:.*]] = tt.broadcast %[[VAL18]] : tensor<128x1xi1> -> tensor<128x128xi1>
47| // CHECK-DAG: %[[VAL20:.*]] = arith.cmpi sge, %[[VAL12]], %[[CST0]]
48| // CHECK-DAG: %[[VAL21:.*]] = arith.cmpi slt, %[[VAL12]], %[[CST3]]
49| // CHECK-DAG: %[[VAL22:.*]] = arith.andi %[[VAL20]], %[[VAL21]]
50| // CHECK-DAG: %[[VAL23:.*]] = tt.broadcast %[[VAL22]] : tensor<1x128xi1> -> tensor<128x128xi1>
51| // CHECK-DAG: %[[VAL24:.*]] = arith.andi %[[VAL19]], %[[VAL23]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 52-55
```mlir
52| 
53| // CHECK-DAG: %[[VAL25:.*]] = tt.load %[[VAL15]], %[[VAL24]], %[[CST]]
54| // CHECK: tt.return %[[VAL25]] :
55| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 56-56
```mlir
56| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 57-58
```mlir
57| 
58| module {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 59-70
```mlir
59|   tt.func public @store(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg1: i32, %arg2: i32, %arg3: tensor<128x128xf32>) {
60|     %c1_i64 = arith.constant 1 : i64
61|     %c256_i64 = arith.constant 256 : i64
62|     %c0_i32 = arith.constant 0 : i32
63|     %c128_i32 = arith.constant 128 : i32
64|     %c256_i32 = arith.constant 256 : i32
65|     %0 = tt.make_tensor_descriptor %arg0, [%c256_i32, %c256_i32], [%c1_i64, %c256_i64] {order = array<i32: 0>} : <f32>, <128x128xf32>
66|     tt.descriptor_store %0[%arg1, %arg2], %arg3 : !tt.tensordesc<128x128xf32>, tensor<128x128xf32>
67|     tt.return
68|   }
69| }
70| 
```
**EN:** This function-oriented block defines or enters `store`. Within it, the test exercises constants, tt.func, tt.make_tensor_descriptor, tt.descriptor_store, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `store` 为核心。测试在其中演示 常量、tt.func、tt.make_tensor_descriptor、tt.descriptor_store、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 71-88
```mlir
71| // CHECK-LABEL: @store
72| // CHECK-SAME: %[[ARG0:[^:]*]]
73| // CHECK-SAME: %[[ARG1:[^:]*]]
74| // CHECK-SAME: %[[ARG2:[^:]*]]
75| // CHECK-SAME: %[[ARG3:[^:]*]]
76| // CHECK-DAG: %[[CST:.*]] = arith.constant dense<0> : tensor<1x128xi64>
77| // CHECK-DAG: %[[CST0:.*]] = arith.constant dense<256> : tensor<128x1xi64>
78| // CHECK-DAG: %[[CST1:.*]] = arith.constant dense<0> : tensor<128x1xi64>
79| // CHECK-DAG: %[[CST2:.*]] = arith.constant dense<256> : tensor<1x128xi64>
80| 
81| // CHECK-DAG: %[[VAL0:.*]] = arith.extsi %[[ARG1]] : i32 to i64
82| // CHECK-DAG: %[[VAL1:.*]] = arith.extsi %[[ARG2]] : i32 to i64
83| // CHECK-DAG: %[[VAL2:.*]] = tt.splat %[[ARG0]] :
84| // CHECK-DAG: %[[VAL3:.*]] = tt.splat %[[VAL0]] :
85| // CHECK-DAG: %[[VAL4:.*]] = tt.make_range {end = 128 : i32, start = 0 : i32}
86| // CHECK-DAG: %[[VAL5:.*]] = arith.extsi %[[VAL4]] :
87| // CHECK-DAG: %[[VAL6:.*]] = arith.addi %[[VAL3]], %[[VAL5]] :
88| // CHECK-DAG: %[[VAL7:.*]] = tt.expand_dims %[[VAL6]] {axis = 1 : i32}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @store anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @store 这样的标签用于锚定匹配范围。

### Lines 89-106
```mlir
 89| // CHECK-DAG: %[[VAL8:.*]] = tt.broadcast %[[VAL7]] : tensor<128x1xi64> -> tensor<128x128xi64>
 90| // CHECK-DAG: %[[VAL9:.*]] = tt.addptr %[[VAL2]], %[[VAL8]] :
 91| // CHECK-DAG: %[[VAL10:.*]] = tt.splat %[[VAL1]] :
 92| // CHECK-DAG: %[[VAL11:.*]] = arith.addi %[[VAL10]], %[[VAL5]] :
 93| // CHECK-DAG: %[[VAL12:.*]] = tt.expand_dims %[[VAL11]] {axis = 0 : i32}
 94| // CHECK-DAG: %[[VAL13:.*]] = arith.muli %[[VAL12]], %[[CST2]] :
 95| // CHECK-DAG: %[[VAL14:.*]] = tt.broadcast %[[VAL13]] : tensor<1x128xi64> -> tensor<128x128xi64>
 96| // CHECK-DAG: %[[VAL15:.*]] = tt.addptr %[[VAL9]], %[[VAL14]] :
 97| 
 98| // CHECK-DAG: %[[VAL16:.*]] = arith.cmpi sge, %[[VAL7]], %[[CST1]]
 99| // CHECK-DAG: %[[VAL17:.*]] = arith.cmpi slt, %[[VAL7]], %[[CST0]]
100| // CHECK-DAG: %[[VAL18:.*]] = arith.andi %[[VAL16]], %[[VAL17]]
101| // CHECK-DAG: %[[VAL19:.*]] = tt.broadcast %[[VAL18]] : tensor<128x1xi1> -> tensor<128x128xi1>
102| // CHECK-DAG: %[[VAL20:.*]] = arith.cmpi sge, %[[VAL12]], %[[CST]]
103| // CHECK-DAG: %[[VAL21:.*]] = arith.cmpi slt, %[[VAL12]], %[[CST2]]
104| // CHECK-DAG: %[[VAL22:.*]] = arith.andi %[[VAL20]], %[[VAL21]]
105| // CHECK-DAG: %[[VAL23:.*]] = tt.broadcast %[[VAL22]] : tensor<1x128xi1> -> tensor<128x128xi1>
106| // CHECK-DAG: %[[VAL24:.*]] = arith.andi %[[VAL19]], %[[VAL23]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 107-109
```mlir
107| 
108| // CHECK: tt.store %[[VAL15]], %[[ARG3]], %[[VAL24]]
109| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 110-110
```mlir
110| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 111-112
```mlir
111| 
112| #loc2 = loc("rewrite-tensor-descriptor-to-pointer.mlir":147:28)
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 113-113
```mlir
113| module {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 114-127
```mlir
114|   tt.func public @callee(%tensordesc: !tt.tensordesc<128x128xf32> loc("tensordesc"(#loc2))) -> !tt.tensordesc<128x128xf32> {
115|     tt.return %tensordesc : !tt.tensordesc<128x128xf32>
116|   }
117| 
118|   tt.func public @caller(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}) {
119|     %c1_i64 = arith.constant 1 : i64
120|     %c256_i32 = arith.constant 256 : i32
121|     %c256_i64 = arith.constant 256 : i64
122|     %0 = tt.make_tensor_descriptor %arg0, [%c256_i32, %c256_i32], [%c256_i64, %c1_i64] {order = array<i32: 0>} : <f32>, <128x128xf32>
123|     %1 = tt.call @callee(%0) : (!tt.tensordesc<128x128xf32>) -> !tt.tensordesc<128x128xf32>
124|     tt.return
125|   }
126| }
127| 
```
**EN:** This function-oriented block defines or enters `callee`. Within it, the test exercises tt.func, constants, tt.return, tt.make_tensor_descriptor, tt.call, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `callee` 为核心。测试在其中演示 tt.func、常量、tt.return、tt.make_tensor_descriptor、tt.call，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 128-145
```mlir
128| // CHECK-LABEL: @callee
129| // CHECK-SAME: %[[PTR:[^:]*]]
130| // CHECK-SAME: loc("tensordesc"(#loc{{[^,]*}}))
131| // CHECK-SAME: %[[SHAPE0:[^:]*]]
132| // CHECK-SAME: loc("tensordesc.shape.0"(#loc{{[^,]*}}))
133| // CHECK-SAME: %[[SHAPE1:[^:]*]]
134| // CHECK-SAME: loc("tensordesc.shape.1"(#loc{{[^,]*}}))
135| // CHECK-SAME: %[[STRIDE0:[^:]*]]
136| // CHECK-SAME: loc("tensordesc.stride.0"(#loc{{[^,]*}}))
137| // CHECK-SAME: %[[STRIDE1:[^:]*]]
138| // CHECK-SAME: loc("tensordesc.stride.1"(#loc{{[^,]*}}))
139| // CHECK-SAME: %[[PAD:[^:]*]]
140| // CHECK-SAME: loc("tensordesc.padding"(#loc{{[^,]*}}))
141| // CHECK-SAME: %[[ROUND:[^:]*]]
142| // CHECK-SAME: loc("tensordesc.roundF32ToTF32"(#loc{{[^,]*}}))
143| // CHECK-NEXT: tt.return %[[PTR]], %[[SHAPE0]], %[[SHAPE1]], %[[STRIDE0]], %[[STRIDE1]], %[[PAD]], %[[ROUND]]
144| 
145| // CHECK-LABEL: @caller
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @callee; CHECK-LABEL: @caller anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @callee；CHECK-LABEL: @caller 这样的标签用于锚定匹配范围。

### Lines 146-151
```mlir
146| // CHECK-SAME: %[[PTR:[^:]*]]
147| // CHECK-DAG: %[[c1:.*]] = arith.constant 1 : i64
148| // CHECK-DAG: %[[c256:.*]] = arith.constant 256 : i64
149| // CHECK: %{{.*}}:7 = tt.call @callee(%[[PTR]], %[[c256]], %[[c256]], %[[c256]], %[[c1]], %false, %false)
150| // CHECK-SAME -> (!tt.ptr<f32>, i64, i64, i64, i64, i1, i1)
151| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 152-152
```mlir
152| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 153-154
```mlir
153| 
154| module {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 155-159
```mlir
155|   tt.func public @arg_attr(%arg0: !tt.tensordesc<128x128xf32>, %arg1: i32 {tt.divisibility = 16 : i32}) {
156|     tt.return
157|   }
158| }
159| 
```
**EN:** This function-oriented block defines or enters `arg_attr`. Within it, the test exercises tt.func, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `arg_attr` 为核心。测试在其中演示 tt.func、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 160-161
```mlir
160| // CHECK-LABEL: @arg_attr
161| // CHECK-SAME: %arg7: i32 {tt.divisibility = 16 : i32} loc({{.*}})) {
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @arg_attr anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @arg_attr 这样的标签用于锚定匹配范围。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `--triton-rewrite-tensor-descriptor-to-pointer`, `--canonicalize`, `--cse`, `--mlir-print-debuginfo`, `--split-input-file`
- **CN:** 主要 pass 选项：`--triton-rewrite-tensor-descriptor-to-pointer`，`--canonicalize`，`--cse`，`--mlir-print-debuginfo`，`--split-input-file`
- **EN:** Dominant operations include `arith.constant`, `tt.func`, `tt.return`, `module`, `tt.make_tensor_descriptor`, `tt.descriptor_load`, `tt.descriptor_store`, `tt.call`.
- **CN:** 主要操作包括 `arith.constant`、`tt.func`、`tt.return`、`module`、`tt.make_tensor_descriptor`、`tt.descriptor_load`、`tt.descriptor_store`、`tt.call`。
- **EN:** The file contains 3 independently testable section(s). Check styles used: CHECK-DAG x62, CHECK-SAME x24, CHECK-LABEL x5, CHECK x3. Important labels include @load, @store, @callee, @caller. It validates that the selected pass pipeline rewrites each labeled test case into the checked form.
- **CN:** 该文件包含 3 个可独立测试的分段。使用的检查类型：CHECK-DAG ×62，CHECK-SAME ×24，CHECK-LABEL ×5，CHECK ×3。 关键标签包括 @load，@store，@callee，@caller。 它验证所选 pass 流水线会把每个带标签的测试用例重写成检查所要求的形式。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `tt`: Triton high-level tensor/pointer operations.
- **CN:** `tt`：Triton 高层张量/指针操作。
- **EN:** `arith`: MLIR arithmetic dialect for scalar/tensor math.
- **CN:** `arith`：用于标量/张量运算的 MLIR arithmetic 方言。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。