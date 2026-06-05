# mfma-double-rate.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/amd/mfma-double-rate.mlir`
- **EN:** Test for the MLIR/Triton pipeline `--convert-triton-amdgpu-to-llvm="gfx-arch=gfx950"` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `--convert-triton-amdgpu-to-llvm="gfx-arch=gfx950"` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s  -split-input-file --convert-triton-amdgpu-to-llvm="gfx-arch=gfx950" | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s  -split-input-file --convert-triton-amdgpu-to-llvm="gfx-arch=gfx950" | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt %s  -split-input-file --convert-triton-amdgpu-to-llvm="gfx-arch=gfx950" | FileCheck %s
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s  -split-input-file --convert-triton-amdgpu-to-llvm="gfx-arch=gfx950" | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s  -split-input-file --convert-triton-amdgpu-to-llvm="gfx-arch=gfx950" | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-4
```mlir
3| // CHECK-LABEL:mfma_16x16x32_f16
4| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL:mfma_16x16x32_f16 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL:mfma_16x16x32_f16 这样的标签用于锚定匹配范围。

### Lines 5-5
```mlir
5| #mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [4, 1], instrShape = [16, 16, 32], isTransposed = false}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 6-6
```mlir
6| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 7-9
```mlir
7|   tt.func public @mfma_16x16x32_f16(%arg0: tensor<16x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>,
8|                          %arg1: tensor<32x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>) {
9|     %cst = arith.constant dense<0.000000e+00> : tensor<16x16xf32, #mma>
```
**EN:** This function-oriented block defines or enters `mfma_16x16x32_f16`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mfma_16x16x32_f16` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 10-15
```mlir
10|     // CHECK: rocdl.mfma.f32.16x16x32.f16 {{.*}} : (vector<8xf16>, vector<8xf16>
11|     %dot = tt.dot %arg0, %arg1, %cst : tensor<16x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>> * tensor<32x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>> -> tensor<16x16xf32, #mma>
12|     tt.return
13|  }
14| }
15| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on dot-product or MMA-style math, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 点积或 MMA 风格计算、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 16-16
```mlir
16| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 17-19
```mlir
17| 
18| // CHECK-LABEL:mfma_16x16x32_bf16
19| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL:mfma_16x16x32_bf16 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL:mfma_16x16x32_bf16 这样的标签用于锚定匹配范围。

### Lines 20-20
```mlir
20| #mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [4, 1], instrShape = [16, 16, 32], isTransposed = false}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 21-21
```mlir
21| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 22-24
```mlir
22|   tt.func public @mfma_16x16x32_bf16(%arg0: tensor<16x32xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>,
23|                          %arg1: tensor<32x16xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>) {
24|     %cst = arith.constant dense<0.000000e+00> : tensor<16x16xf32, #mma>
```
**EN:** This function-oriented block defines or enters `mfma_16x16x32_bf16`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mfma_16x16x32_bf16` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 25-30
```mlir
25|     // CHECK: rocdl.mfma.f32.16x16x32.bf16 {{.*}} : (vector<8xbf16>, vector<8xbf16>
26|     %dot = tt.dot %arg0, %arg1, %cst : tensor<16x32xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>> * tensor<32x16xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>> -> tensor<16x16xf32, #mma>
27|     tt.return
28|  }
29| }
30| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on dot-product or MMA-style math, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 点积或 MMA 风格计算、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 31-31
```mlir
31| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 32-34
```mlir
32| 
33| // CHECK-LABEL:mfma_32x32x16_f16
34| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL:mfma_32x32x16_f16 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL:mfma_32x32x16_f16 这样的标签用于锚定匹配范围。

### Lines 35-35
```mlir
35| #mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [4, 1], instrShape = [32, 32, 16], isTransposed = false}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 36-36
```mlir
36| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 37-39
```mlir
37|   tt.func public @mfma_32x32x16_f16(%arg0: tensor<32x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>,
38|                          %arg1: tensor<16x32xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>) {
39|     %cst = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #mma>
```
**EN:** This function-oriented block defines or enters `mfma_32x32x16_f16`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mfma_32x32x16_f16` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 40-46
```mlir
40|     // CHECK: rocdl.mfma.f32.32x32x16.f16 {{.*}} : (vector<8xf16>, vector<8xf16>
41|     %dot = tt.dot %arg0, %arg1, %cst : tensor<32x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>> * tensor<16x32xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>> -> tensor<32x32xf32, #mma>
42|     tt.return
43|  }
44| }
45| 
46| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on dot-product or MMA-style math, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 点积或 MMA 风格计算、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 47-47
```mlir
47| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 48-50
```mlir
48| 
49| // CHECK-LABEL:mfma_32x32x16_bf16
50| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL:mfma_32x32x16_bf16 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL:mfma_32x32x16_bf16 这样的标签用于锚定匹配范围。

### Lines 51-51
```mlir
51| #mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [4, 1], instrShape = [32, 32, 16], isTransposed = false}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 52-52
```mlir
52| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 53-55
```mlir
53|   tt.func public @mfma_32x32x16_bf16(%arg0: tensor<32x16xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>,
54|                          %arg1: tensor<16x32xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>) {
55|     %cst = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #mma>
```
**EN:** This function-oriented block defines or enters `mfma_32x32x16_bf16`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mfma_32x32x16_bf16` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 56-61
```mlir
56|     // CHECK: rocdl.mfma.f32.32x32x16.bf16 {{.*}} : (vector<8xbf16>, vector<8xbf16>
57|     %dot = tt.dot %arg0, %arg1, %cst : tensor<32x16xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>> * tensor<16x32xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>> -> tensor<32x32xf32, #mma>
58|     tt.return
59|  }
60| }
61| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on dot-product or MMA-style math, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 点积或 MMA 风格计算、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 62-62
```mlir
62| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 63-65
```mlir
63| 
64| // When kWidth is set to 4, still generate double rated mfma instructions.
65| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 66-67
```mlir
66| // CHECK-LABEL:mfma_16x16x32_f16
67| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL:mfma_16x16x32_f16 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL:mfma_16x16x32_f16 这样的标签用于锚定匹配范围。

### Lines 68-70
```mlir
68| #mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [4, 1], instrShape = [16, 16, 32], isTransposed = true}>
69| #dotOp0 = #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>
70| #dotOp1 = #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 71-71
```mlir
71| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 72-75
```mlir
72|   tt.func public @mfma_16x16x32_f16(
73|       %q: tensor<128x128xf16, #dotOp0>,
74|       %k: tensor<128x128xf16, #dotOp1>) {
75|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #mma>
```
**EN:** This function-oriented block defines or enters `mfma_16x16x32_f16`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mfma_16x16x32_f16` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 76-81
```mlir
76|     // CHECK: rocdl.mfma.f32.16x16x32.f16 {{.*}} : (vector<8xf16>, vector<8xf16>
77|     %qk = tt.dot %q, %k, %cst : tensor<128x128xf16, #dotOp0> * tensor<128x128xf16, #dotOp1> -> tensor<128x128xf32, #mma>
78|     tt.return
79|  }
80| }
81| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on dot-product or MMA-style math, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 点积或 MMA 风格计算、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 82-82
```mlir
82| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 83-85
```mlir
83| 
84| // CHECK-LABEL:mfma_16x16x32_bf16
85| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL:mfma_16x16x32_bf16 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL:mfma_16x16x32_bf16 这样的标签用于锚定匹配范围。

### Lines 86-88
```mlir
86| #mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [4, 1], instrShape = [16, 16, 32], isTransposed = true}>
87| #dotOp0 = #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>
88| #dotOp1 = #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 89-89
```mlir
89| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 90-93
```mlir
90|   tt.func public @mfma_16x16x32_bf16(
91|       %q: tensor<128x128xbf16, #dotOp0>,
92|       %k: tensor<128x128xbf16, #dotOp1>) {
93|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #mma>
```
**EN:** This function-oriented block defines or enters `mfma_16x16x32_bf16`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mfma_16x16x32_bf16` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 94-99
```mlir
94|     // CHECK: rocdl.mfma.f32.16x16x32.bf16 {{.*}} : (vector<8xbf16>, vector<8xbf16>
95|     %qk = tt.dot %q, %k, %cst : tensor<128x128xbf16, #dotOp0> * tensor<128x128xbf16, #dotOp1> -> tensor<128x128xf32, #mma>
96|     tt.return
97|  }
98| }
99| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on dot-product or MMA-style math, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 点积或 MMA 风格计算、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 100-100
```mlir
100| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 101-103
```mlir
101| 
102| // CHECK-LABEL:mfma_32x32x16_f16
103| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL:mfma_32x32x16_f16 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL:mfma_32x32x16_f16 这样的标签用于锚定匹配范围。

### Lines 104-106
```mlir
104| #mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [4, 1], instrShape = [32, 32, 16], isTransposed = true}>
105| #dotOp0 = #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>
106| #dotOp1 = #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 107-107
```mlir
107| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 108-111
```mlir
108|   tt.func public @mfma_32x32x16_f16(
109|       %q: tensor<128x128xf16, #dotOp0>,
110|       %k: tensor<128x128xf16, #dotOp1>) {
111|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #mma>
```
**EN:** This function-oriented block defines or enters `mfma_32x32x16_f16`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mfma_32x32x16_f16` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 112-117
```mlir
112|     // CHECK: rocdl.mfma.f32.32x32x16.f16 {{.*}} : (vector<8xf16>, vector<8xf16>
113|     %qk = tt.dot %q, %k, %cst : tensor<128x128xf16, #dotOp0> * tensor<128x128xf16, #dotOp1> -> tensor<128x128xf32, #mma>
114|     tt.return
115|  }
116| }
117| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on dot-product or MMA-style math, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 点积或 MMA 风格计算、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 118-118
```mlir
118| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 119-121
```mlir
119| 
120| // CHECK-LABEL:mfma_32x32x16_bf16
121| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL:mfma_32x32x16_bf16 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL:mfma_32x32x16_bf16 这样的标签用于锚定匹配范围。

### Lines 122-124
```mlir
122| #mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [4, 1], instrShape = [32, 32, 16], isTransposed = true}>
123| #dotOp0 = #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>
124| #dotOp1 = #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 125-125
```mlir
125| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 126-129
```mlir
126|   tt.func public @mfma_32x32x16_bf16(
127|       %q: tensor<128x128xbf16, #dotOp0>,
128|       %k: tensor<128x128xbf16, #dotOp1>) {
129|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #mma>
```
**EN:** This function-oriented block defines or enters `mfma_32x32x16_bf16`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mfma_32x32x16_bf16` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 130-135
```mlir
130|     // CHECK: rocdl.mfma.f32.32x32x16.bf16 {{.*}} : (vector<8xbf16>, vector<8xbf16>
131|     %qk = tt.dot %q, %k, %cst : tensor<128x128xbf16, #dotOp0> * tensor<128x128xbf16, #dotOp1> -> tensor<128x128xf32, #mma>
132|     tt.return
133|  }
134| }
135| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on dot-product or MMA-style math, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 点积或 MMA 风格计算、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 136-136
```mlir
136| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 137-138
```mlir
137| 
138| // CHECK-LABEL:mxfp4_2step
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL:mxfp4_2step anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL:mxfp4_2step 这样的标签用于锚定匹配范围。

### Lines 139-141
```mlir
139| #linear = #ttg.linear<{register = [[0, 4], [32, 0], [64, 0], [128, 0]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 1], [0, 2]], warp = [[0, 0], [0, 0], [16, 0]], block = []}>
140| #linear1 = #ttg.linear<{register = [[0, 4], [64, 0], [128, 0]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 1], [0, 2]], warp = [[16, 0], [32, 0], [0, 0]], block = []}>
141| #mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [2, 4], instrShape = [16, 16, 128], isTransposed = true}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 142-142
```mlir
142| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 143-143
```mlir
143|   tt.func public @mxfp4_2step(%arg0: tensor<256x128xi8, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 16}>>, %arg1: tensor<256x8xi8, #linear>, %arg2: tensor<128x256xi8, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>, %arg3: tensor<256x8xi8, #linear1>) {
```
**EN:** This function-oriented block defines or enters `mxfp4_2step`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mxfp4_2step` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 144-153
```mlir
144|     // CHECK-COUNT-32: rocdl.mfma.scale.f32.16x16x128.f8f6f4
145|     // CHECK: rocdl.sched.barrier 0
146|     // CHECK: rocdl.s.barrier
147|     // CHECK: rocdl.sched.barrier 0
148|     // CHECK-COUNT-32: rocdl.mfma.scale.f32.16x16x128.f8f6f4
149|     %cst = arith.constant dense<0.000000e+00> : tensor<256x256xf32, #mma>
150|     %dots = tt.dot_scaled %arg0 scale %arg1, %arg2 scale %arg3, %cst lhs = e2m1 rhs = e2m1 {fastMath = false, pingpong_2step} : tensor<256x128xi8, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 16}>>, tensor<256x8xi8, #linear> * tensor<128x256xi8, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>, tensor<256x8xi8, #linear1> -> tensor<256x256xf32, #mma>
151|     tt.return
152|  }
153| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants, tt.dot_scaled, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量、tt.dot_scaled、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `--convert-triton-amdgpu-to-llvm="gfx-arch=gfx950"`
- **CN:** 主要 pass 选项：`-split-input-file`，`--convert-triton-amdgpu-to-llvm="gfx-arch=gfx950"`
- **EN:** Dominant operations include `tt.func`, `module`, `arith.constant`, `tt.return`, `tt.dot`, `tt.dot_scaled`.
- **CN:** 主要操作包括 `tt.func`、`module`、`arith.constant`、`tt.return`、`tt.dot`、`tt.dot_scaled`。
- **EN:** The file contains 8 independently testable section(s). Check styles used: CHECK x11, CHECK-LABEL x9, CHECK-COUNT x2. Important labels include mfma_16x16x32_f16, mfma_16x16x32_bf16, mfma_32x32x16_f16, mfma_32x32x16_bf16. It validates that the selected pass pipeline rewrites each labeled test case into the checked form.
- **CN:** 该文件包含 8 个可独立测试的分段。使用的检查类型：CHECK ×11，CHECK-LABEL ×9，CHECK-COUNT ×2。 关键标签包括 mfma_16x16x32_f16，mfma_16x16x32_bf16，mfma_32x32x16_f16，mfma_32x32x16_bf16。 它验证所选 pass 流水线会把每个带标签的测试用例重写成检查所要求的形式。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `ttg`: TritonGPU layout, memory, and GPU-specific operations.
- **CN:** `ttg`：TritonGPU 布局、内存和 GPU 专用操作。
- **EN:** `tt`: Triton high-level tensor/pointer operations.
- **CN:** `tt`：Triton 高层张量/指针操作。
- **EN:** `arith`: MLIR arithmetic dialect for scalar/tensor math.
- **CN:** `arith`：用于标量/张量运算的 MLIR arithmetic 方言。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。