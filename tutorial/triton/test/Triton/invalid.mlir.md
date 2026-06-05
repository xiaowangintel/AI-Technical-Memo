# invalid.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/Triton/invalid.mlir`
- **EN:** Negative/diagnostic test that checks verifier or pass failures for invalid Triton IR.
- **CN:** 这是一个负向/诊断测试，用来检查无效 Triton IR 是否会触发 verifier 或 pass 失败。
- **EN:** RUN pipeline(s): `// RUN: triton-opt --split-input-file %s --verify-diagnostics`
- **CN:** RUN 流水线：`// RUN: triton-opt --split-input-file %s --verify-diagnostics`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt --split-input-file %s --verify-diagnostics
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt --split-input-file %s --verify-diagnostics` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt --split-input-file %s --verify-diagnostics`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-4
```mlir
3| tt.func @fn(%v: i32) {
4|   %b = tt.splat %v : i32 -> tensor<128xi32>
```
**EN:** This function-oriented block defines or enters `fn`. Within it, the test exercises tt.func, broadcasted scalars or pointers, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fn` 为核心。测试在其中演示 tt.func、广播后的标量或指针，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 5-9
```mlir
5|   // expected-error @+1 {{rank of source must be same as rank of result}}
6|   %c = tt.broadcast %b : tensor<128xi32> -> tensor<128x32xi32>
7|   tt.return
8| }
9| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor broadcasting, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量广播、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 10-10
```mlir
10| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 11-12
```mlir
11| 
12| // Invalid bitcast between types of different bit width.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 13-13
```mlir
13| tt.func public @fn(%arg0: tensor<128xf32>) {
```
**EN:** This function-oriented block defines or enters `fn`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fn` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 14-17
```mlir
14|     // expected-error @+1 {{Cannot bitcast data-type of size}}
15|     %a = tt.bitcast %arg0 : tensor<128xf32> -> tensor<128xi16>
16|     tt.return
17| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.bitcast, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.bitcast、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 18-18
```mlir
18| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 19-20
```mlir
19| 
20| // Invalid bitcast between pointer and non-pointer type.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 21-21
```mlir
21| tt.func public @fn(%arg0: !tt.ptr<f32>) {
```
**EN:** This function-oriented block defines or enters `fn`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fn` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 22-25
```mlir
22|     // expected-error @+1 {{Cannot bitcast pointer to non-pointer type}}
23|     %a = tt.bitcast %arg0 : !tt.ptr<f32> -> i32
24|     tt.return
25| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.bitcast, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.bitcast、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 26-26
```mlir
26| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 27-29
```mlir
27| 
28| tt.func @fn(%v: i32) {
29|   %b = tt.splat %v : i32 -> tensor<2x32xi32>
```
**EN:** This function-oriented block defines or enters `fn`. Within it, the test exercises tt.func, broadcasted scalars or pointers, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fn` 为核心。测试在其中演示 tt.func、广播后的标量或指针，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 30-34
```mlir
30|   // expected-error @+1 {{Different dimensions at index 0 between source and result.  Broadcast requires the source dimension to be 1.}}
31|   %c = tt.broadcast %b : tensor<2x32xi32> -> tensor<128x32xi32>
32|   tt.return
33| }
34| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor broadcasting, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量广播、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 35-35
```mlir
35| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 36-37
```mlir
36| 
37| tt.func public @fn(%arg0: tensor<128xf32>) {
```
**EN:** This function-oriented block defines or enters `fn`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fn` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 38-43
```mlir
38|     // expected-error @+1 {{packed_element}}
39|     %a = tt.elementwise_inline_asm ""
40|       {constraints = "=r,r", packed_element=3:i32, pure=true} %arg0 : tensor<128xf32> -> tensor<128xf32>
41|     tt.return
42| }
43| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.elementwise_inline_asm, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.elementwise_inline_asm、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 44-44
```mlir
44| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 45-46
```mlir
45| 
46| tt.func public @fn(%arg0: tensor<128xf32>, %arg1: tensor<64xf32>) {
```
**EN:** This function-oriented block defines or enters `fn`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fn` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 47-52
```mlir
47|     // expected-error @+1 {{same shape}}
48|     %a = tt.elementwise_inline_asm ""
49|       {constraints = "=r,r,r", packed_element=1:i32, pure=true}
50|       %arg0, %arg1: tensor<128xf32>, tensor<64xf32> -> tensor<128xf32>
51|     tt.return
52| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.elementwise_inline_asm, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.elementwise_inline_asm、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 53-53
```mlir
53| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 54-55
```mlir
54| 
55| tt.func public @reshape_different_num_elements(%arg0: tensor<32x128xf16>) {
```
**EN:** This function-oriented block defines or enters `reshape_different_num_elements`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `reshape_different_num_elements` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 56-60
```mlir
56|     // expected-error @+1 {{number of src and dst elements of reshape must be the same}}
57|     %a = tt.reshape %arg0 : tensor<32x128xf16> -> tensor<64x32xf16>
58|     tt.return
59| }
60| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor reshaping, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量重塑、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 61-61
```mlir
61| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 62-63
```mlir
62| 
63| // expected-note @+1 {{prior use}}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 64-64
```mlir
64| tt.func public @fn(%arg0: tensor<32xf32>, %arg1: tensor<33xf32>) {
```
**EN:** This function-oriented block defines or enters `fn`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fn` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 65-69
```mlir
65|     // expected-error @+1 {{expects different type}}
66|     %a = tt.join %arg0, %arg1 : tensor<32xf32> -> tensor<32x2xf32>
67|     tt.return
68| }
69| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.join, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.join、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 70-70
```mlir
70| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 71-72
```mlir
71| 
72| // expected-note @+1 {{prior use}}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 73-73
```mlir
73| tt.func public @fn(%arg0: tensor<32x32xf32>, %arg1: tensor<32x32xf16>) {
```
**EN:** This function-oriented block defines or enters `fn`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fn` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 74-78
```mlir
74|     // expected-error @+1 {{expects different type}}
75|     %a = tt.join %arg0, %arg1 : tensor<32x32xf32> -> tensor<32x32x2xf32>
76|     tt.return
77| }
78| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.join, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.join、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 79-79
```mlir
79| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 80-81
```mlir
80| 
81| tt.func public @fn(%arg0: tensor<32xf32>, %arg1: tensor<32xf32>) {
```
**EN:** This function-oriented block defines or enters `fn`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fn` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 82-86
```mlir
82|     // expected-error @+1 {{op result shape must be (32, 2), but got 64}}
83|     %a = tt.join %arg0, %arg1 : tensor<32xf32> -> tensor<64xf32>
84|     tt.return
85| }
86| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.join, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.join、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 87-87
```mlir
87| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 88-89
```mlir
88| 
89| tt.func public @fn(%arg0: tensor<32x32xf32>, %arg1: tensor<32x32xf32>) {
```
**EN:** This function-oriented block defines or enters `fn`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fn` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 90-94
```mlir
90|     // expected-error @+1 {{result shape must be (32, 32, 2), but got 32, 64}}
91|     %a = tt.join %arg0, %arg1 : tensor<32x32xf32> -> tensor<32x64xf32>
92|     tt.return
93| }
94| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.join, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.join、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 95-95
```mlir
95| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 96-97
```mlir
96| 
97| // This one is OK
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 98-102
```mlir
 98| tt.func public @fn(%arg0: tensor<f32>, %arg1: tensor<f32>) {
 99|     %a = tt.join %arg0, %arg1 : tensor<f32> -> tensor<2xf32>
100|     tt.return
101| }
102| 
```
**EN:** This function-oriented block defines or enters `fn`. Within it, the test exercises tt.func, tt.join, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fn` 为核心。测试在其中演示 tt.func、tt.join、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 103-103
```mlir
103| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 104-105
```mlir
104| 
105| tt.func public @fn(%arg0: f32, %arg1: f32) {
```
**EN:** This function-oriented block defines or enters `fn`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fn` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 106-110
```mlir
106|     // expected-error @+1 {{kind of type}}
107|     %a = tt.join %arg0, %arg1 : f32 -> tensor<2xf32>
108|     tt.return
109| }
110| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.join, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.join、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 111-111
```mlir
111| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 112-113
```mlir
112| 
113| tt.func public @fn(%v: tensor<4x128xf64>) {
```
**EN:** This function-oriented block defines or enters `fn`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fn` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 114-122
```mlir
114|     // expected-error @+1 {{operand types and result types}}
115|     %a = "tt.reduce" (%v) ({
116|     ^bb0(%arg0: f32, %arg1: f32):
117|       %add = arith.addf %arg0, %arg1 : f32
118|       tt.reduce.return %add : f32
119|     }) {axis = 0 : i32}  : (tensor<4x128xf64>) -> tensor<128xf32>
120|     tt.return
121| }
122| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on floating-point additions, reductions, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 浮点加法、归约、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 123-123
```mlir
123| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 124-125
```mlir
124| 
125| tt.func public @fn(%v: tensor<4x128xf32>) {
```
**EN:** This function-oriented block defines or enters `fn`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fn` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 126-134
```mlir
126|     // expected-error @+1 {{axis out of bounds}}
127|     %a = "tt.reduce" (%v) ({
128|     ^bb0(%arg0: f32, %arg1: f32):
129|       %add = arith.addf %arg0, %arg1 : f32
130|       tt.reduce.return %add : f32
131|     }) {axis = 2 : i32}  : (tensor<4x128xf32>) -> tensor<4xf32>
132|     tt.return
133| }
134| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on floating-point additions, reductions, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 浮点加法、归约、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 135-135
```mlir
135| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 136-137
```mlir
136| 
137| tt.func @reduce_different_input_shapes(%arg0: tensor<32x32x64xf32>, %arg1: tensor<16x32x64xf32>) -> (tensor<32x64xf32>, tensor<16x64xf32>) {
```
**EN:** This function-oriented block defines or enters `reduce_different_input_shapes`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `reduce_different_input_shapes` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 138-147
```mlir
138|     // expected-error @below {{op requires the same shape for all operands}}
139|     %0:2 = "tt.reduce" (%arg0, %arg1) <{axis = 1 : i32}> ({
140|     ^bb0(%acc0: f32, %acc1: f32, %cur0: f32, %cur1: f32):
141|       %1 = arith.addf %acc0, %cur0 : f32
142|       %2 = arith.addf %acc1, %cur1 : f32
143|       tt.reduce.return %1, %2 : f32, f32
144|     }) : (tensor<32x32x64xf32>, tensor<16x32x64xf32>) -> (tensor<32x64xf32>, tensor<16x64xf32>)
145|     tt.return %0#0, %0#1 : tensor<32x64xf32>, tensor<16x64xf32>
146| }
147| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on floating-point additions, reductions, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 浮点加法、归约、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 148-148
```mlir
148| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 149-150
```mlir
149| 
150| tt.func public @fn(%v: tensor<4x128xf32>) {
```
**EN:** This function-oriented block defines or enters `fn`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fn` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 151-159
```mlir
151|     // expected-error @+1 {{requires the same shape}}
152|     %a = "tt.scan" (%v) ({
153|     ^bb0(%arg0: f32, %arg1: f32):
154|       %add = arith.addf %arg0, %arg1 : f32
155|       tt.scan.return %add : f32
156|     }) {axis = 0 : i32, reverse = false}  : (tensor<4x128xf32>) -> tensor<128xf32>
157|     tt.return
158| }
159| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on floating-point additions, tt.scan, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 浮点加法、tt.scan、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 160-160
```mlir
160| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 161-162
```mlir
161| 
162| tt.func public @fn(%v1: tensor<4x128xf32>, %v2: tensor<4x128xi64>) {
```
**EN:** This function-oriented block defines or enters `fn`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fn` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 163-171
```mlir
163|     // expected-error @+1 {{operand types and result types}}
164|     %a, %b = "tt.scan" (%v1, %v2) ({
165|     ^bb0(%arg0: f32, %arg1: i32, %arg2: f32, %arg3: i32):
166|       %add = arith.addf %arg0, %arg2 : f32
167|       tt.scan.return %add, %arg1 : f32, i32
168|     }) {axis = 0 : i32, reverse = false}  : (tensor<4x128xf32>, tensor<4x128xi64>) -> (tensor<4x128xi64>, tensor<4x128xf32>)
169|     tt.return
170| }
171| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on floating-point additions, tt.scan, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 浮点加法、tt.scan、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 172-172
```mlir
172| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 173-174
```mlir
173| 
174| tt.func public @fn(%v1: tensor<4x128xf32>, %v2: tensor<4x128xi64>) {
```
**EN:** This function-oriented block defines or enters `fn`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fn` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 175-183
```mlir
175|     // expected-error @+1 {{operand types and result types}}
176|     %a, %b = "tt.reduce" (%v1, %v2) ({
177|     ^bb0(%arg0: f32, %arg1: i32, %arg2: f32, %arg3: i32):
178|       %add = arith.addf %arg0, %arg2 : f32
179|       tt.reduce.return %add, %arg1 : f32, i32
180|     }) {axis = 0 : i32}  : (tensor<4x128xf32>, tensor<4x128xi64>) -> (tensor<128xi64>, tensor<128xf32>)
181|     tt.return
182| }
183| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on floating-point additions, reductions, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 浮点加法、归约、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 184-184
```mlir
184| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 185-186
```mlir
185| 
186| #blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 187-187
```mlir
187| module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 188-188
```mlir
188| tt.func public @fn(%arg0: tensor<32xf32, #blocked>) {
```
**EN:** This function-oriented block defines or enters `fn`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fn` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 189-194
```mlir
189|     // expected-error @+1 {{op result encoding must be specified}}
190|     %a = tt.join %arg0, %arg0 : tensor<32xf32, #blocked> -> tensor<32x2xf32>
191|     tt.return
192| }
193| }  // end module
194| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.join, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.join、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 195-195
```mlir
195| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 196-197
```mlir
196| 
197| // Bad order; should be [1,0]
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 198-199
```mlir
198| #blocked  = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
199| #blocked1 = #ttg.blocked<{sizePerThread = [1,2], threadsPerWarp = [32,1], warpsPerCTA = [1,1], order = [0,1]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 200-200
```mlir
200| module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 201-201
```mlir
201| tt.func public @fn(%arg0: tensor<32xf32, #blocked>) {
```
**EN:** This function-oriented block defines or enters `fn`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fn` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 202-207
```mlir
202|     // expected-error @+1 {{op incompatible join layout}}
203|     %a = tt.join %arg0, %arg0 : tensor<32xf32, #blocked> -> tensor<32x2xf32, #blocked1>
204|     tt.return
205| }
206| }  // end module
207| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.join, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.join、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 208-208
```mlir
208| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 209-210
```mlir
209| 
210| tt.func public @fn(%arg0: tensor<32xf32>) {
```
**EN:** This function-oriented block defines or enters `fn`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fn` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 211-216
```mlir
211|     // expected-error @+2 {{last dimension}}
212|     // expected-error @+1 {{op failed to infer returned types}}
213|     %a, %b = tt.split %arg0 : tensor<32xf32> -> tensor<16xf32>
214|     tt.return
215| }
216| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.split, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.split、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 217-217
```mlir
217| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 218-219
```mlir
218| 
219| tt.func public @fn(%arg0: tensor<32x2xf32>) {
```
**EN:** This function-oriented block defines or enters `fn`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fn` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 220-225
```mlir
220|     // expected-error @+2 {{op inferred type}}
221|     // expected-error @+1 {{op failed to infer returned types}}
222|     %a, %b = tt.split %arg0 : tensor<32x2xf32> -> tensor<32xf16>
223|     tt.return
224| }
225| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.split, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.split、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 226-226
```mlir
226| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 227-228
```mlir
227| 
228| tt.func public @fn(%arg0: f32) {
```
**EN:** This function-oriented block defines or enters `fn`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fn` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 229-232
```mlir
229|     // expected-error @+1 {{invalid kind of type}}
230|     %a, %b = tt.split %arg0 : f32 -> f16
231|     tt.return
232| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.split, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.split、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 233-233
```mlir
233| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 234-239
```mlir
234| 
235| tt.func public @fn(%arg0: tensor<2xf32>) {
236|     %a, %b = tt.split %arg0 : tensor<2xf32> -> tensor<f32> // OK
237|     tt.return
238| }
239| 
```
**EN:** This function-oriented block defines or enters `fn`. Within it, the test exercises tt.func, tt.split, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fn` 为核心。测试在其中演示 tt.func、tt.split、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 240-240
```mlir
240| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 241-243
```mlir
241| 
242| #blocked  = #ttg.blocked<{sizePerThread = [1,2,2], threadsPerWarp = [1,32,1], warpsPerCTA = [1,1,1], order = [2,0,1]}>
243| // Bad order, should be [1,0].
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 244-245
```mlir
244| #blocked1 = #ttg.blocked<{sizePerThread = [1,1], threadsPerWarp = [1,32], warpsPerCTA = [1,1], order = [1,0]}>
245| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 246-246
```mlir
246| module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 247-247
```mlir
247| tt.func public @fn(%arg0: tensor<2x2x2xf32, #blocked>) {
```
**EN:** This function-oriented block defines or enters `fn`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fn` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 248-254
```mlir
248|     // expected-error @+2 {{op inferred type}}
249|     // expected-error @+1 {{op failed to infer returned types}}
250|     %a, %b = tt.split %arg0 : tensor<2x2x2xf32, #blocked> -> tensor<2x2xf32, #blocked1>
251|     tt.return
252| }
253| }  // end module
254| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.split, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.split、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 255-255
```mlir
255| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 256-258
```mlir
256| 
257| #blocked  = #ttg.blocked<{sizePerThread = [1,1,2], threadsPerWarp = [1,32,1], warpsPerCTA = [1,1,1], order = [2,0,1]}>
258| // bad sizePerThread; should be [1,1].
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 259-260
```mlir
259| #blocked1 = #ttg.blocked<{sizePerThread = [1,2], threadsPerWarp = [1,32], warpsPerCTA = [1,1], order = [0,1]}>
260| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 261-261
```mlir
261| module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 262-262
```mlir
262| tt.func public @fn(%arg0: tensor<2x2x2xf32, #blocked>) {
```
**EN:** This function-oriented block defines or enters `fn`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fn` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 263-269
```mlir
263|     // expected-error @+2 {{op inferred type}}
264|     // expected-error @+1 {{op failed to infer returned types}}
265|     %a, %b = tt.split %arg0 : tensor<2x2x2xf32, #blocked> -> tensor<2x2xf32, #blocked1>
266|     tt.return
267| }
268| }  // end module
269| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.split, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.split、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 270-270
```mlir
270| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 271-272
```mlir
271| 
272| // Valid ops.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 273-273
```mlir
273| module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 274-280
```mlir
274| tt.func public @fn(%arg0: tensor<16x32x64xf32>) {
275|     %a = tt.trans %arg0 {order = array<i32: 0, 1, 2>} : tensor<16x32x64xf32> -> tensor<16x32x64xf32>
276|     %b = tt.trans %arg0 {order = array<i32: 1, 0, 2>} : tensor<16x32x64xf32> -> tensor<32x16x64xf32>
277|     tt.return
278| }
279| }  // end module
280| 
```
**EN:** This function-oriented block defines or enters `fn`. Within it, the test exercises tt.func, transpose-like layout changes, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fn` 为核心。测试在其中演示 tt.func、转置类布局变换、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 281-281
```mlir
281| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 282-283
```mlir
282| 
283| // Valid op with blocked encoding.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 284-285
```mlir
284| #blocked2 = #ttg.blocked<{sizePerThread = [1,2,4], threadsPerWarp = [2,4,4], warpsPerCTA = [2,4,8], order = [0,1,2], CGALayout = [[0, 1, 0], [0, 0, 1], [0, 0, 2]]}>
285| #blocked3 = #ttg.blocked<{sizePerThread = [2,1,4], threadsPerWarp = [4,2,4], warpsPerCTA = [4,2,8], order = [1,0,2], CGALayout = [[1, 0, 0], [0, 0, 1], [0, 0, 2]]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 286-286
```mlir
286| module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 8 : i32, "ttg.num-warps" = 64 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 287-292
```mlir
287| tt.func public @fn(%arg0: tensor<16x32x64xf32, #blocked2>) {
288|     %b = tt.trans %arg0 {order = array<i32: 1, 0, 2>} : tensor<16x32x64xf32, #blocked2> -> tensor<32x16x64xf32, #blocked3>
289|     tt.return
290| }
291| }  // end module
292| 
```
**EN:** This function-oriented block defines or enters `fn`. Within it, the test exercises tt.func, transpose-like layout changes, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fn` 为核心。测试在其中演示 tt.func、转置类布局变换、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 293-293
```mlir
293| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 294-295
```mlir
294| 
295| // Valid op with shared encoding.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 296-300
```mlir
296| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [3, 2, 1, 0], CGALayout = [[0, 0, 0, 0], [0, 0, 0, 0], [0, 0, 0, 0]]}>
297| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 2, 0, 3], CGALayout = [[0, 0, 0, 0], [0, 0, 0, 0], [0, 0, 0, 0]]}>
298| #shared2 = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = false, elementBitWidth = 32, CGALayout = [[1, 0], [0, 1], [0, 2]]}>
299| #shared3 = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = true, elementBitWidth = 32, CGALayout = [[0, 1], [1, 0], [2, 0]]}>
300| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 301-301
```mlir
301| module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 8 : i32, "ttg.num-warps" = 64 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 302-308
```mlir
302| tt.func public @fn(%arg0: !ttg.memdesc<2x4x8x16xf32, #shared, #smem>, %arg1: !ttg.memdesc<16x32xf32, #shared2, #smem>) {
303|     %a = ttg.memdesc_trans %arg0 {order = array<i32: 1, 3, 2, 0>} : !ttg.memdesc<2x4x8x16xf32, #shared, #smem> -> !ttg.memdesc<4x16x8x2xf32, #shared1, #smem>
304|     %b = ttg.memdesc_trans %arg1 {order = array<i32: 1, 0>} : !ttg.memdesc<16x32xf32, #shared2, #smem> -> !ttg.memdesc<32x16xf32, #shared3, #smem>
305|     tt.return
306| }
307| }  // end module
308| 
```
**EN:** This function-oriented block defines or enters `fn`. Within it, the test exercises tt.func, ttg.memdesc_trans, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fn` 为核心。测试在其中演示 tt.func、ttg.memdesc_trans、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 309-309
```mlir
309| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 310-311
```mlir
310| 
311| // Invalid blocked encoding.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 312-313
```mlir
312| #blocked  = #ttg.blocked<{sizePerThread = [1,2,4], threadsPerWarp = [2,4,4], warpsPerCTA = [2,4,8], order = [0,1,2], CGALayout = [[0, 1, 0], [0, 0, 1], [0, 0, 2]]}>
313| #blocked1 = #ttg.blocked<{sizePerThread = [1,2,4], threadsPerWarp = [4,2,4], warpsPerCTA = [4,2,8], order = [1,0,2], CGALayout = [[1, 0, 0], [0, 0, 1], [0, 0, 2]]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 314-314
```mlir
314| module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 8 : i32, "ttg.num-warps" = 64 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 315-315
```mlir
315| tt.func public @fn(%arg0: tensor<16x32x64xf32, #blocked>) {
```
**EN:** This function-oriented block defines or enters `fn`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fn` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 316-321
```mlir
316|     // expected-error @+1 {{type}}
317|     %a = tt.trans %arg0 {order = array<i32: 1, 0, 2>} : tensor<16x32x64xf32, #blocked> -> tensor<32x16x64xf32, #blocked1>
318|     tt.return
319| }
320| }  // end module
321| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on transpose-like layout changes, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 转置类布局变换、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 322-322
```mlir
322| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 323-324
```mlir
323| 
324| // Invalid shared encoding.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 325-326
```mlir
325| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0, 1, 2]}>
326| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [2, 0, 1]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 327-327
```mlir
327| module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 8 : i32, "ttg.num-warps" = 64 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 328-328
```mlir
328| tt.func public @fn(%arg0: tensor<16x32x64xf32, #shared>) {
```
**EN:** This function-oriented block defines or enters `fn`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fn` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 329-334
```mlir
329|     // expected-error @+1 {{type}}
330|     %a = tt.trans %arg0 {order = array<i32: 1, 0, 2>} : tensor<16x32x64xf32, #shared> -> tensor<32x16x64xf32, #shared1>
331|     tt.return
332| }
333| }  // end module
334| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on transpose-like layout changes, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 转置类布局变换、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 335-335
```mlir
335| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 336-337
```mlir
336| 
337| module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 338-338
```mlir
338| tt.func public @fn(%arg0: tensor<16x32xf32>) {
```
**EN:** This function-oriented block defines or enters `fn`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fn` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 339-344
```mlir
339|     // expected-error @+1 {{order}}
340|     %a = tt.trans %arg0 {order = array<i32: 0>} : tensor<16x32xf32> -> tensor<32x16xf32>
341|     tt.return
342| }
343| }  // end module
344| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on transpose-like layout changes, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 转置类布局变换、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 345-345
```mlir
345| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 346-347
```mlir
346| 
347| module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 348-348
```mlir
348| tt.func public @fn(%arg0: tensor<16x32xf32>) {
```
**EN:** This function-oriented block defines or enters `fn`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fn` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 349-354
```mlir
349|     // expected-error @+1 {{order}}
350|     %a = tt.trans %arg0 {order = array<i32: 2, 1, 0>} : tensor<16x32xf32> -> tensor<32x16xf32>
351|     tt.return
352| }
353| }  // end module
354| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on transpose-like layout changes, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 转置类布局变换、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 355-355
```mlir
355| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 356-357
```mlir
356| 
357| module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 358-358
```mlir
358| tt.func public @fn(%arg0: tensor<16x32xf32>) {
```
**EN:** This function-oriented block defines or enters `fn`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fn` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 359-364
```mlir
359|     // expected-error @+1 {{order must be a permutation}}
360|     %a = tt.trans %arg0 {order = array<i32: 0, 0>} : tensor<16x32xf32> -> tensor<32x16xf32>
361|     tt.return
362| }
363| }  // end module
364| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on transpose-like layout changes, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 转置类布局变换、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 365-365
```mlir
365| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 366-367
```mlir
366| 
367| // Invalid tensor with shared encoding.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 368-369
```mlir
368| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0, 1, 2]}>
369| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [2, 0, 1]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 370-370
```mlir
370| module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 64 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 371-371
```mlir
371| tt.func public @fn(%arg0: tensor<16x32x64xf32, #shared>) {
```
**EN:** This function-oriented block defines or enters `fn`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fn` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 372-377
```mlir
372|     // expected-error @+1 {{Non-distributed layout is not allowed in tensor type.}}
373|     %a = tt.trans %arg0 {order = array<i32: 1, 0, 2>} : tensor<16x32x64xf32, #shared> -> tensor<32x16x64xf32, #shared1>
374|     tt.return
375| }
376| }  // end module
377| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on transpose-like layout changes, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 转置类布局变换、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 378-378
```mlir
378| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 379-380
```mlir
379| 
380| tt.func @gather_op(%arg0: tensor<128x16xf32>, %arg1: tensor<512x4xi32>) {
```
**EN:** This function-oriented block defines or enters `gather_op`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `gather_op` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 381-385
```mlir
381|   // expected-error @below {{indices and output shapes must match}}
382|   %0 = tt.gather %arg0[%arg1] {axis = 0 : i32} : (tensor<128x16xf32>, tensor<512x4xi32>) -> tensor<512xf32>
383|   tt.return
384| }
385| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.gather, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.gather、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 386-386
```mlir
386| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 387-389
```mlir
387| 
388| #blocked  = #ttg.blocked<{sizePerThread = [2, 1], threadsPerWarp = [32, 1], warpsPerCTA = [1, 1], order = [0, 1]}>
389| #blocked1 = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [32, 1], warpsPerCTA = [1, 1], order = [0, 1]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 390-390
```mlir
390| module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 391-391
```mlir
391| tt.func @gather_op(%arg0: tensor<128x16xf32>, %arg1: tensor<512x4xi32, #blocked>) {
```
**EN:** This function-oriented block defines or enters `gather_op`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `gather_op` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 392-397
```mlir
392|   // expected-error @below {{indices and output encodings must match}}
393|   %0 = tt.gather %arg0[%arg1] {axis = 0 : i32} : (tensor<128x16xf32>, tensor<512x4xi32, #blocked>) -> tensor<512x4xf32, #blocked1>
394|   tt.return
395| }
396| }
397| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.gather, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.gather、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 398-398
```mlir
398| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 399-400
```mlir
399| 
400| tt.func @gather_op(%arg0: tensor<128x16xf16>, %arg1: tensor<512x4xi32>) {
```
**EN:** This function-oriented block defines or enters `gather_op`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `gather_op` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 401-405
```mlir
401|   // expected-error @below {{input and output element types must match}}
402|   %0 = tt.gather %arg0[%arg1] {axis = 0 : i32} : (tensor<128x16xf16>, tensor<512x4xi32>) -> tensor<512x4xf32>
403|   tt.return
404| }
405| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.gather, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.gather、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 406-406
```mlir
406| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 407-408
```mlir
407| 
408| tt.func @gather_op(%arg0: tensor<128xf32>, %arg1: tensor<512x4xi32>) {
```
**EN:** This function-oriented block defines or enters `gather_op`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `gather_op` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 409-413
```mlir
409|   // expected-error @below {{input and indices ranks must match}}
410|   %0 = tt.gather %arg0[%arg1] {axis = 0 : i32} : (tensor<128xf32>, tensor<512x4xi32>) -> tensor<512x4xf32>
411|   tt.return
412| }
413| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.gather, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.gather、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 414-414
```mlir
414| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 415-416
```mlir
415| 
416| tt.func @gather_op(%arg0: tensor<128x16xf32>, %arg1: tensor<512x32xi32>) {
```
**EN:** This function-oriented block defines or enters `gather_op`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `gather_op` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 417-420
```mlir
417|   // expected-error @below {{indices dimension 1 must match the corresponding input dimension}}
418|   %0 = tt.gather %arg0[%arg1] {axis = 0 : i32} : (tensor<128x16xf32>, tensor<512x32xi32>) -> tensor<512x32xf32>
419|   tt.return
420| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.gather, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.gather、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 421-421
```mlir
421| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 422-423
```mlir
422| 
423| tt.func @gather_op(%arg0: tensor<128x16xf32>, %arg1: tensor<512x4xi32>) {
```
**EN:** This function-oriented block defines or enters `gather_op`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `gather_op` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 424-428
```mlir
424|   // expected-error @below {{gather dimension must be less than the input rank}}
425|   %0 = tt.gather %arg0[%arg1] {axis = 3 : i32} : (tensor<128x16xf32>, tensor<512x4xi32>) -> tensor<512x4xf32>
426|   tt.return
427| }
428| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.gather, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.gather、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 429-429
```mlir
429| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 430-432
```mlir
430| 
431| tt.func @invalid_desc_load(%arg0: !tt.tensordesc<16x16xf32>) {
432|   %c = arith.constant 0 : i32
```
**EN:** This function-oriented block defines or enters `invalid_desc_load`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `invalid_desc_load` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 433-437
```mlir
433|   // expected-error @below {{descriptor block and tensor must have the same number of elements}}
434|   tt.descriptor_load %arg0[%c, %c] : !tt.tensordesc<16x16xf32> -> tensor<16xf32>
435|   tt.return
436| }
437| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.descriptor_load, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.descriptor_load、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 438-438
```mlir
438| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 439-441
```mlir
439| 
440| tt.func @invalid_desc_load(%arg0: !tt.tensordesc<16x16xf32>) {
441|   %c = arith.constant 0 : i32
```
**EN:** This function-oriented block defines or enters `invalid_desc_load`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `invalid_desc_load` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 442-446
```mlir
442|   // expected-error @below {{descriptor block and tensor element types must match}}
443|   tt.descriptor_load %arg0[%c, %c] : !tt.tensordesc<16x16xf32> -> tensor<16x16xf16>
444|   tt.return
445| }
446| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.descriptor_load, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.descriptor_load、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 447-447
```mlir
447| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 448-450
```mlir
448| 
449| tt.func @invalid_desc_store(%arg0: !tt.tensordesc<16x16xf32>, %arg1: tensor<32x16xf32>) {
450|   %c = arith.constant 0 : i32
```
**EN:** This function-oriented block defines or enters `invalid_desc_store`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `invalid_desc_store` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 451-455
```mlir
451|   // expected-error @below {{descriptor block and tensor must have the same number of elements}}
452|   tt.descriptor_store %arg0[%c, %c], %arg1 : !tt.tensordesc<16x16xf32>, tensor<32x16xf32>
453|   tt.return
454| }
455| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.descriptor_store, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.descriptor_store、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 456-456
```mlir
456| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 457-458
```mlir
457| 
458| tt.func @invalid_tma_gather(%arg0: !tt.tensordesc<128xbf16>, %arg1: tensor<32xi32>, %arg2: i32) {
```
**EN:** This function-oriented block defines or enters `invalid_tma_gather`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `invalid_tma_gather` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 459-463
```mlir
459|   // expected-error @below {{block must be a 2D tensor}}
460|   %0 = tt.descriptor_gather %arg0[%arg1, %arg2] : (!tt.tensordesc<128xbf16>, tensor<32xi32>, i32) -> tensor<32xbf16>
461|   tt.return
462| }
463| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.descriptor_gather, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.descriptor_gather、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 464-464
```mlir
464| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 465-466
```mlir
465| 
466| tt.func @invalid_tma_gather(%arg0: !tt.tensordesc<2x128xbf16>, %arg1: tensor<32xi32>, %arg2: i32) {
```
**EN:** This function-oriented block defines or enters `invalid_tma_gather`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `invalid_tma_gather` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 467-471
```mlir
467|   // expected-error @below {{block must have exactly 1 row}}
468|   %0 = tt.descriptor_gather %arg0[%arg1, %arg2] : (!tt.tensordesc<2x128xbf16>, tensor<32xi32>, i32) -> tensor<32x128xbf16>
469|   tt.return
470| }
471| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.descriptor_gather, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.descriptor_gather、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 472-472
```mlir
472| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 473-474
```mlir
473| 
474| tt.func @invalid_tma_gather(%arg0: !tt.tensordesc<1x128xbf16>, %arg1: tensor<1x32xi32>, %arg2: i32) {
```
**EN:** This function-oriented block defines or enters `invalid_tma_gather`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `invalid_tma_gather` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 475-479
```mlir
475|   // expected-error @below {{x offsets must be a 1D tensor}}
476|   %0 = tt.descriptor_gather %arg0[%arg1, %arg2] : (!tt.tensordesc<1x128xbf16>, tensor<1x32xi32>, i32) -> tensor<32x128xbf16>
477|   tt.return
478| }
479| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.descriptor_gather, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.descriptor_gather、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 480-480
```mlir
480| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 481-482
```mlir
481| 
482| tt.func @invalid_tma_gather(%arg0: !tt.tensordesc<1x128xbf16>, %arg1: tensor<32xi32>, %arg2: i32) {
```
**EN:** This function-oriented block defines or enters `invalid_tma_gather`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `invalid_tma_gather` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 483-487
```mlir
483|   // expected-error @below {{result must be a 2D tensor}}
484|   %0 = tt.descriptor_gather %arg0[%arg1, %arg2] : (!tt.tensordesc<1x128xbf16>, tensor<32xi32>, i32) -> tensor<128xbf16>
485|   tt.return
486| }
487| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.descriptor_gather, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.descriptor_gather、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 488-488
```mlir
488| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 489-490
```mlir
489| 
490| tt.func @invalid_tma_gather(%arg0: !tt.tensordesc<1x128xbf16>, %arg1: tensor<32xi32>, %arg2: i32) {
```
**EN:** This function-oriented block defines or enters `invalid_tma_gather`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `invalid_tma_gather` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 491-495
```mlir
491|   // expected-error @below {{result tensor number of columns must match block (128)}}
492|   %0 = tt.descriptor_gather %arg0[%arg1, %arg2] : (!tt.tensordesc<1x128xbf16>, tensor<32xi32>, i32) -> tensor<32x64xbf16>
493|   tt.return
494| }
495| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.descriptor_gather, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.descriptor_gather、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 496-496
```mlir
496| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 497-498
```mlir
497| 
498| tt.func @invalid_tma_gather(%arg0: !tt.tensordesc<1x128xbf16>, %arg1: tensor<32xi32>, %arg2: i32) {
```
**EN:** This function-oriented block defines or enters `invalid_tma_gather`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `invalid_tma_gather` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 499-503
```mlir
499|   // expected-error @below {{result tensor must have as many rows as indices (32)}}
500|   %0 = tt.descriptor_gather %arg0[%arg1, %arg2] : (!tt.tensordesc<1x128xbf16>, tensor<32xi32>, i32) -> tensor<64x128xbf16>
501|   tt.return
502| }
503| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.descriptor_gather, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.descriptor_gather、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 504-504
```mlir
504| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 505-506
```mlir
505| 
506| tt.func @invalid_tma_gather(%arg0: !tt.tensordesc<1x128xbf16>, %arg1: tensor<32xi32>, %arg2: i32) {
```
**EN:** This function-oriented block defines or enters `invalid_tma_gather`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `invalid_tma_gather` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 507-511
```mlir
507|   // expected-error @below {{result tensor element type must match block ('bf16')}}
508|   %0 = tt.descriptor_gather %arg0[%arg1, %arg2] : (!tt.tensordesc<1x128xbf16>, tensor<32xi32>, i32) -> tensor<32x128xf32>
509|   tt.return
510| }
511| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.descriptor_gather, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.descriptor_gather、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 512-512
```mlir
512| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 513-517
```mlir
513| 
514| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [0, 1]}>
515| #mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [2, 2], instrShape = [16, 8]}>
516| #shared = #ttg.swizzled_shared<{vec = 8, perPhase = 1, maxPhase = 4, order = [0, 1]}>
517| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 518-518
```mlir
518| module attributes {"ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 519-527
```mlir
519|   tt.func public @invalid_dot(%arg0: tensor<32x32x!tt.ptr<f32>, #blocked>, %arg1: tensor<16x32x!tt.ptr<f32>, #blocked>) {
520|     %9 = tt.load %arg0 : tensor<32x32x!tt.ptr<f32>, #blocked>
521|     %10 = tt.load %arg1 : tensor<16x32x!tt.ptr<f32>, #blocked>
522|     %11 = ttg.local_alloc %9 : (tensor<32x32xf32, #blocked>) -> !ttg.memdesc<32x32xf32, #shared, #smem>
523|     %12 = ttg.local_alloc %10 : (tensor<16x32xf32, #blocked>) -> !ttg.memdesc<16x32xf32, #shared, #smem>
524|     %13 = ttg.local_load %11 : !ttg.memdesc<32x32xf32, #shared, #smem> -> tensor<32x32xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>>
525|     %14 = ttg.local_load %12 : !ttg.memdesc<16x32xf32, #shared, #smem> -> tensor<16x32xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>>
526|     %15 = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #mma>
527| 
```
**EN:** This function-oriented block defines or enters `invalid_dot`. Within it, the test exercises tt.func, masked or vectorized loads, shared/local memory allocation, local/shared memory loads, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `invalid_dot` 为核心。测试在其中演示 tt.func、带掩码或向量化的加载、共享/本地内存分配、本地/共享内存加载、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 528-535
```mlir
528|     // expected-error @below {{'tt.dot' op expected the last dimension of the first operand to be equal to the second-to-last dimension of the second operand}}
529|     %16 = tt.dot %13, %14, %15 : tensor<32x32xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>> * tensor<16x32xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>> -> tensor<32x32xf32, #mma>
530|     %17 = ttg.convert_layout %16 : tensor<32x32xf32, #mma> -> tensor<32x32xf32, #blocked>
531|     tt.store %arg0, %17 : tensor<32x32x!tt.ptr<f32>, #blocked>
532|     tt.return
533|   }
534| }
535| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on dot-product or MMA-style math, layout conversions, masked or vectorized stores, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 点积或 MMA 风格计算、布局转换、带掩码或向量化的存储、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 536-536
```mlir
536| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 537-540
```mlir
537| 
538| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
539| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [16, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
540| #blocked2 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [2, 2], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 541-541
```mlir
541| module attributes {"ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 542-547
```mlir
542|   tt.func @dot_scaled_fp8(
543|     %a: tensor<128x32xi8, #blocked2>,
544|     %scale: tensor<128x2xi8, #blocked1>,
545|     %b_fp8: tensor<128x128xf8E4M3FN, #blocked>
546|     ) -> tensor<128x128xf32, #blocked> {
547|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
```
**EN:** This function-oriented block defines or enters `dot_scaled_fp8`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `dot_scaled_fp8` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 548-553
```mlir
548|     // expected-error @below {{'tt.dot_scaled' op expected the last dimension of the first operand to be equal to the second-to-last dimension of the second operand}}
549|     %result = tt.dot_scaled %a scale %scale, %b_fp8, %cst lhs = e2m1 rhs = e4m3 {fastMath = true} : tensor<128x32xi8, #blocked2>, tensor<128x2xi8, #blocked1> * tensor<128x128xf8E4M3FN, #blocked> -> tensor<128x128xf32, #blocked>
550|     tt.return %result : tensor<128x128xf32, #blocked>
551|   }
552| }
553| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.dot_scaled, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.dot_scaled、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 554-554
```mlir
554| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 555-556
```mlir
555| 
556| module {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 557-562
```mlir
557|   tt.func @dot_scaled_invalid_dims(
558|     %a: tensor<128x128xf8E4M3FN>,
559|     %b: tensor<128x128xf8E4M3FN>,
560|     %a_scale: tensor<128x128xi8>,
561|     %b_scale: tensor<128x4xi8>) -> tensor<128x128xf32> {
562|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32>
```
**EN:** This function-oriented block defines or enters `dot_scaled_invalid_dims`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `dot_scaled_invalid_dims` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 563-568
```mlir
563|     // expected-error @below {{scale factor must be 16 or 32. Got 1}}
564|     %result = tt.dot_scaled %a scale %a_scale, %b scale %b_scale, %cst lhs = e4m3 rhs = e4m3 {fastMath = true} : tensor<128x128xf8E4M3FN>, tensor<128x128xi8>  * tensor<128x128xf8E4M3FN>, tensor<128x4xi8>-> tensor<128x128xf32>
565|     tt.return %result : tensor<128x128xf32>
566|   }
567| }
568| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.dot_scaled, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.dot_scaled、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 569-569
```mlir
569| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 570-571
```mlir
570| 
571| tt.func @unsplat_invalid(%arg0: tensor<128xf32>) {
```
**EN:** This function-oriented block defines or enters `unsplat_invalid`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `unsplat_invalid` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 572-576
```mlir
572|   // expected-error @below {{source tensor must have exactly one element}}
573|   %0 = tt.unsplat %arg0 : tensor<128xf32>
574|   tt.return
575| }
576| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.unsplat, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.unsplat、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 577-577
```mlir
577| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 578-580
```mlir
578| 
579| tt.func @atomic_cas_different_elem_types(%arg0: tensor<128x!tt.ptr<f32>>, %arg1: tensor<128xi32>) {
580|   %cmp = arith.constant dense<0> : tensor<128xi32>
```
**EN:** This function-oriented block defines or enters `atomic_cas_different_elem_types`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `atomic_cas_different_elem_types` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 581-585
```mlir
581|   // expected-error @below {{'tt.atomic_cas' op failed to verify that ptr type matches cmp type}}
582|   %0 = tt.atomic_cas relaxed, gpu, %arg0, %cmp, %arg1 : (tensor<128x!tt.ptr<f32>>, tensor<128xi32>, tensor<128xi32>) -> tensor<128xi32>
583|   tt.return
584| }
585| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.atomic_cas, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.atomic_cas、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 586-586
```mlir
586| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 587-589
```mlir
587| 
588| tt.func @atomic_cas_different_elem_types(%arg0: tensor<128x!tt.ptr<f32>>, %arg1: tensor<128xi32>) {
589|   %cmp = arith.constant dense<0.0> : tensor<128xf32>
```
**EN:** This function-oriented block defines or enters `atomic_cas_different_elem_types`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `atomic_cas_different_elem_types` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 590-594
```mlir
590|   // expected-error @below {{'tt.atomic_cas' op failed to verify that ptr type matches value type}}
591|   %0 = tt.atomic_cas relaxed, gpu, %arg0, %cmp, %arg1 : (tensor<128x!tt.ptr<f32>>, tensor<128xf32>, tensor<128xi32>) -> tensor<128xi32>
592|   tt.return
593| }
594| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.atomic_cas, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.atomic_cas、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 595-595
```mlir
595| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 596-598
```mlir
596| 
597| tt.func @map_elementwise_arg_num_mismatch() {
598|   %cst = arith.constant dense<0> : tensor<256xi32>
```
**EN:** This function-oriented block defines or enters `map_elementwise_arg_num_mismatch`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `map_elementwise_arg_num_mismatch` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 599-606
```mlir
599|   // expected-error @below {{region has wrong number of arguments}}
600|   "tt.map_elementwise" (%cst) <{pack = 1 : i32}> ({
601|   ^bb0(%arg0: i64, %arg1 : i32):
602|      tt.map_elementwise.return %arg1 : i32
603|   }) : (tensor<256xi32>) -> (tensor<256xi32>)
604|   tt.return
605| }
606| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.map_elementwise, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.map_elementwise、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 607-607
```mlir
607| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 608-610
```mlir
608| 
609| tt.func @map_elementwise_arg_mismatch() {
610|   %cst = arith.constant dense<0> : tensor<256xi32>
```
**EN:** This function-oriented block defines or enters `map_elementwise_arg_mismatch`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `map_elementwise_arg_mismatch` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 611-618
```mlir
611|   // expected-error @below {{argument types did not match}}
612|   "tt.map_elementwise" (%cst) <{pack = 1 : i32}> ({
613|   ^bb0(%arg0: i64):
614|      tt.map_elementwise.return %arg0 : i64
615|   }) : (tensor<256xi32>) -> (tensor<256xi64>)
616|   tt.return
617| }
618| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.map_elementwise, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.map_elementwise、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 619-619
```mlir
619| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 620-624
```mlir
620| 
621| tt.func @map_elementwise_return_mismatch() {
622|   %cst = arith.constant dense<0> : tensor<256xi32>
623|   "tt.map_elementwise" (%cst) <{pack = 1 : i32}> ({
624|   ^bb0(%arg0: i32):
```
**EN:** This function-oriented block defines or enters `map_elementwise_return_mismatch`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `map_elementwise_return_mismatch` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 625-630
```mlir
625|      // expected-error @below {{region return does not match map_elementwise result}}
626|      tt.map_elementwise.return %arg0 : i32
627|   }) : (tensor<256xi32>) -> (tensor<256xi64>)
628|   tt.return
629| }
630| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.map_elementwise, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.map_elementwise、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 631-631
```mlir
631| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 632-636
```mlir
632| 
633| tt.func @map_elementwise_store(%ptr: tensor<256x!tt.ptr<i32>>) {
634|   %cst = arith.constant dense<0> : tensor<256xi32>
635|   "tt.map_elementwise" (%ptr, %cst) <{pack = 1 : i32}> ({
636|   ^bb0(%arg0: !tt.ptr<i32>, %arg1: i32):
```
**EN:** This function-oriented block defines or enters `map_elementwise_store`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `map_elementwise_store` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 637-643
```mlir
637|      // expected-error @below {{Stores are not supported inside map_elementwise}}
638|      tt.store %arg0, %arg1 : !tt.ptr<i32>
639|      tt.map_elementwise.return %arg1 : i32
640|   }) : (tensor<256x!tt.ptr<i32>>, tensor<256xi32>) -> (tensor<256xi32>)
641|   tt.return
642| }
643| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized stores, tt.map_elementwise, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的存储、tt.map_elementwise、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 644-644
```mlir
644| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 645-647
```mlir
645| 
646| // Test that DotOp with f32 inputs but without TF32 precision is rejected for MMAv2
647| // MMAv2 requires TF32 input precision for f32 operands
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 648-651
```mlir
648| #mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [2, 2], instrShape = [16, 8]}>
649| #dot_operand_a = #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>
650| #dot_operand_b = #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>
651| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 652-652
```mlir
652| module attributes {"ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32, ttg.target = "cuda:80"} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:80`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:80`。

### Lines 653-654
```mlir
653|   tt.func @dot_f32_without_tf32_mma_v2(%a: tensor<16x16xf32, #dot_operand_a>, %b: tensor<16x16xf32, #dot_operand_b>) {
654|     %cst = arith.constant dense<0.000000e+00> : tensor<16x16xf32, #mma>
```
**EN:** This function-oriented block defines or enters `dot_f32_without_tf32_mma_v2`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `dot_f32_without_tf32_mma_v2` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 655-659
```mlir
655|     // expected-error @below {{unsupported MMA version}}
656|     %result = tt.dot %a, %b, %cst, inputPrecision = ieee : tensor<16x16xf32, #dot_operand_a> * tensor<16x16xf32, #dot_operand_b> -> tensor<16x16xf32, #mma>
657|     tt.return
658|   }
659| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on dot-product or MMA-style math, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 点积或 MMA 风格计算、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `--split-input-file`, `--verify-diagnostics`
- **CN:** 主要 pass 选项：`--split-input-file`，`--verify-diagnostics`
- **EN:** Dominant operations include `tt.func`, `tt.return`, `module`, `arith.constant`, `tt.trans`, `tt.join`, `arith.addf`, `tt.descriptor_gather`, `tt.split`, `tt.gather`.
- **CN:** 主要操作包括 `tt.func`、`tt.return`、`module`、`arith.constant`、`tt.trans`、`tt.join`、`arith.addf`、`tt.descriptor_gather`、`tt.split`、`tt.gather`。
- **EN:** The file contains 62 independently testable section(s). Expected diagnostic comments specify the exact verifier/pass failures. It validates that invalid IR is rejected with the intended diagnostics.
- **CN:** 该文件包含 62 个可独立测试的分段。expected 诊断注释给出了 verifier/pass 需要触发的精确报错。 它验证无效 IR 会被拒绝，并产生预期诊断。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `tt`: Triton high-level tensor/pointer operations.
- **CN:** `tt`：Triton 高层张量/指针操作。
- **EN:** `ttg`: TritonGPU layout, memory, and GPU-specific operations.
- **CN:** `ttg`：TritonGPU 布局、内存和 GPU 专用操作。
- **EN:** `arith`: MLIR arithmetic dialect for scalar/tensor math.
- **CN:** `arith`：用于标量/张量运算的 MLIR arithmetic 方言。
- **EN:** Diagnostic verification is part of the contract, so parser/verifier error strings are also dependencies of the test.
- **CN:** 诊断验证也是该测试契约的一部分，因此 parser/verifier 的报错文本同样是依赖项。