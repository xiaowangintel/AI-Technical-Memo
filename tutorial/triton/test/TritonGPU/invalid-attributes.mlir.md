# invalid-attributes.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/invalid-attributes.mlir`
- **EN:** Negative/diagnostic test that checks verifier or pass failures for invalid Triton IR.
- **CN:** 这是一个负向/诊断测试，用来检查无效 Triton IR 是否会触发 verifier 或 pass 失败。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file -verify-diagnostics`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file -verify-diagnostics`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt %s -split-input-file -verify-diagnostics
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file -verify-diagnostics` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file -verify-diagnostics`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-3
```mlir
3| // expected-error@+2 {{ttg.dot_op opIdx parameter can be 0 or 1, got: 2}}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 4-6
```mlir
4| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 8], warpsPerCTA = [1, 1], order = [1, 0]}>
5| #dot_op = #ttg.dot_op<{opIdx = 2, parent = #blocked, kWidth = 2}>
6| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 7-7
```mlir
7| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 8-9
```mlir
8| 
9| // expected-error@+2 {{ttg.dot_op kWidth parameter is not supported when the parent is a blocked layout}}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 10-12
```mlir
10| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 8], warpsPerCTA = [1, 1], order = [1, 0]}>
11| #dot_op = #ttg.dot_op<{opIdx = 1, parent = #blocked, kWidth = 8}>
12| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 13-13
```mlir
13| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 14-15
```mlir
14| 
15| // expected-error@+2 {{ttg.dot_op kWidth parameter can only be non-zero for Ampere or Hopper MMA parent}}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 16-18
```mlir
16| #mma = #ttg.nvidia_mma<{versionMajor = 1, warpsPerCTA = [1, 1], instrShape = [16, 8]}>
17| #dot_op = #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>
18| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 19-19
```mlir
19| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 20-21
```mlir
20| 
21| // expected-error@+2 {{ttg.dot_op kWidth parameter is mandatory for Ampere or Hopper MMA parent}}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 22-24
```mlir
22| #mma = #ttg.nvidia_mma<{versionMajor = 2, warpsPerCTA = [1, 1], instrShape = [16, 8]}>
23| #dot_op = #ttg.dot_op<{opIdx = 0, parent = #mma}>
24| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 25-25
```mlir
25| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 26-27
```mlir
26| 
27| // expected-error@+2 {{ttg.dot_op kWidth parameter is mandatory for Ampere or Hopper MMA parent}}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 28-30
```mlir
28| #mma = #ttg.nvidia_mma<{versionMajor = 3, warpsPerCTA = [1, 1], instrShape = [16, 8]}>
29| #dot_op = #ttg.dot_op<{opIdx = 0, parent = #mma}>
30| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 31-31
```mlir
31| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 32-33
```mlir
32| 
33| // expected-error@+2 {{ttg.dot_op opIdx parameter must be 0 for Hopper MMA parent, since Hopper WGMMA only allows first operand to be in registers}}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 34-36
```mlir
34| #mma = #ttg.nvidia_mma<{versionMajor = 3, warpsPerCTA = [1, 1], instrShape = [16, 8]}>
35| #dot_op = #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>
36| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 37-37
```mlir
37| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 38-39
```mlir
38| 
39| // expected-error@+2 {{ttg.dot_op kWidth parameter is mandatory for MFMA parent}}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 40-42
```mlir
40| #mfma = #ttg.amd_mfma<{version = 2, warpsPerCTA = [1, 1, 1], instrShape = [32, 32, 8], isTransposed = false}>
41| #dot_op = #ttg.dot_op<{opIdx = 1, parent = #mfma}>
42| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 43-43
```mlir
43| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 44-45
```mlir
44| 
45| // expected-error@+2 {{ttg.dot_op kWidth parameter must be 8/16 for WMMA v1 (including packed cases for `scaled_dot`)}}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 46-48
```mlir
46| #wmma = #ttg.amd_wmma<{version = 1, ctaLayout = {warp = [[0, 1], [0, 2]]}}>
47| #dot_op = #ttg.dot_op<{opIdx = 1, parent = #wmma}>
48| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 49-49
```mlir
49| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 50-51
```mlir
50| 
51| // expected-error@+2 {{ttg.dot_op kWidth parameter must be 4/8/16 for WMMA v2 (including packed cases for `scaled_dot`)}}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 52-54
```mlir
52| #wmma = #ttg.amd_wmma<{version = 2, ctaLayout = {warp = [[0, 1], [0, 2]]}}>
53| #dot_op = #ttg.dot_op<{opIdx = 1, parent = #wmma, kWidth = 32}>
54| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 55-55
```mlir
55| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 56-57
```mlir
56| 
57| // expected-error@+1 {{invalid WMMA v1 instruction shape}}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 58-59
```mlir
58| #wmma = #ttg.amd_wmma<{version = 1, ctaLayout = {warp = []}, instrShape = [16, 16, 32]}>
59| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 60-60
```mlir
60| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 61-62
```mlir
61| 
62| // expected-error@+1 {{invalid WMMA v2 instruction shape}}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 63-64
```mlir
63| #wmma = #ttg.amd_wmma<{version = 2, ctaLayout = {warp = []}, instrShape = [16, 16, 64]}>
64| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 65-65
```mlir
65| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 66-67
```mlir
66| 
67| // expected-error@+1 {{invalid WMMA v3 instruction shape}}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 68-69
```mlir
68| #wmma = #ttg.amd_wmma<{version = 3, ctaLayout = {warp = []}, instrShape = [16, 16, 16]}>
69| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 70-70
```mlir
70| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 71-72
```mlir
71| 
72| // expected-error@+1 {{version must be in the [0, 4] range}}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 73-74
```mlir
73| #mfma = #ttg.amd_mfma<{version = 10, warpsPerCTA = [1, 1, 1], instrShape = [32, 32, 8], isTransposed = false}>
74| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 75-75
```mlir
75| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 76-77
```mlir
76| 
77| // expected-error@+1 {{invalid (mDim, nDim) combination}}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 78-79
```mlir
78| #mfma = #ttg.amd_mfma<{version = 2, warpsPerCTA = [1, 1, 1], instrShape = [16, 8, 8], isTransposed = false}>
79| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 80-80
```mlir
80| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 81-82
```mlir
81| 
82| // expected-error@+1 {{elementBitWidth must be 32 or 64}}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 83-83
```mlir
83| #mfma = #ttg.amd_mfma<{version = 2, warpsPerCTA = [1, 1, 1], instrShape = [16, 16, 16], isTransposed = false, elementBitWidth = 16}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-verify-diagnostics`
- **CN:** 主要 pass 选项：`-split-input-file`，`-verify-diagnostics`
- **EN:** The file contains 14 independently testable section(s). Expected diagnostic comments specify the exact verifier/pass failures. It validates that invalid IR is rejected with the intended diagnostics.
- **CN:** 该文件包含 14 个可独立测试的分段。expected 诊断注释给出了 verifier/pass 需要触发的精确报错。 它验证无效 IR 会被拒绝，并产生预期诊断。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `ttg`: TritonGPU layout, memory, and GPU-specific operations.
- **CN:** `ttg`：TritonGPU 布局、内存和 GPU 专用操作。
- **EN:** Diagnostic verification is part of the contract, so parser/verifier error strings are also dependencies of the test.
- **CN:** 诊断验证也是该测试契约的一部分，因此 parser/verifier 的报错文本同样是依赖项。