# amd-extractslice-op.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/amd/amd-extractslice-op.mlir`
- **EN:** Test for the MLIR/Triton pipeline `--convert-triton-amdgpu-to-llvm="gfx-arch=gfx942"` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `--convert-triton-amdgpu-to-llvm="gfx-arch=gfx942"` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file --convert-triton-amdgpu-to-llvm="gfx-arch=gfx942" | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file --convert-triton-amdgpu-to-llvm="gfx-arch=gfx942" | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt %s -split-input-file --convert-triton-amdgpu-to-llvm="gfx-arch=gfx942" | FileCheck %s
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file --convert-triton-amdgpu-to-llvm="gfx-arch=gfx942" | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file --convert-triton-amdgpu-to-llvm="gfx-arch=gfx942" | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-4
```mlir
3| #blocked1 = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [4, 16], warpsPerCTA = [8, 1], order = [1, 0]}>
4| #blocked2 = #ttg.blocked<{sizePerThread = [4, 1], threadsPerWarp = [4, 16], warpsPerCTA = [8, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 5-5
```mlir
5| module attributes {"ttg.compute-capability" = 0 : i32, "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 6-6
```mlir
6|   tt.func @extract_2d_blocked_tensor(%arg0: tensor<256x128xi32, #blocked1> {tt.divisibility = 16 : i32}) {
```
**EN:** This function-oriented block defines or enters `extract_2d_blocked_tensor`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `extract_2d_blocked_tensor` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 7-14
```mlir
 7|     // CHECK-LABEL: llvm.func @extract_2d_blocked_tensor
 8|     // CHECK-COUNT-64: %{{.*}} = llvm.extractvalue  %{{.*}} : !llvm.struct
 9|     // CHECK-COUNT-8:  %{{.*}} = llvm.insertvalue %{{.*}} : !llvm.struct
10|     %72 = amdg.extract_slice %arg0 [0,0] : tensor<256x128xi32, #blocked1> to tensor<256x16xi32, #blocked1>
11|     tt.return
12|   }
13| }
14| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.extract_slice, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.extract_slice、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 15-15
```mlir
15| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 16-19
```mlir
16| 
17| #ll1 = #ttg.linear<{register = [[1, 0], [2, 0], [4, 0], [0, 16], [0, 32], [0, 64]], lane = [[0, 1], [0, 2], [0, 4], [0, 8], [8, 0], [16, 0]], warp = [[32, 0], [64, 0], [128, 0]], block = []}>
18| #ll2 = #ttg.linear<{register = [[1, 0], [2, 0], [4, 0]], lane = [[0, 1], [0, 2], [0, 4], [0, 8], [8, 0], [16, 0]], warp = [[32, 0], [64, 0], [128, 0]], block = []}>
19| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 20-20
```mlir
20| module attributes {"ttg.compute-capability" = 0 : i32, "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 21-21
```mlir
21|   tt.func @extract_2d_linear_tensor(%arg0: tensor<256x128xi32, #ll1> {tt.divisibility = 16 : i32}) {
```
**EN:** This function-oriented block defines or enters `extract_2d_linear_tensor`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `extract_2d_linear_tensor` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 22-29
```mlir
22|     // CHECK-LABEL: llvm.func @extract_2d_linear_tensor
23|     // CHECK-COUNT-64: %{{.*}} = llvm.extractvalue  %arg0[{{[0-9]*}}] : !llvm.struct
24|     // CHECK-COUNT-8:  %{{.*}} = llvm.insertvalue %{{.*}} : !llvm.struct
25|     %72 = amdg.extract_slice %arg0 [0,0] : tensor<256x128xi32, #ll1> to tensor<256x16xi32, #ll2>
26|     tt.return
27|   }
28| }
29| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.extract_slice, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.extract_slice、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 30-30
```mlir
30| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 31-34
```mlir
31| 
32| #ll1 = #ttg.linear<{register = [[0, 1, 0], [0, 2, 0], [0, 4, 0], [0, 0, 16], [0, 0, 32], [0, 0, 64], [1, 0, 0]], lane = [[0, 0, 1], [0, 0, 2], [0, 0, 4], [0, 0, 8], [0, 8, 0], [0, 16, 0]], warp = [[0, 32, 0], [0, 64, 0], [0, 128, 0]], block = []}>
33| #ll2 = #ttg.linear<{register = [[0, 1, 0], [0, 2, 0], [0, 4, 0], [0, 0, 16], [0, 0, 32], [0, 0, 64]], lane = [[0, 0, 1], [0, 0, 2], [0, 0, 4], [0, 0, 8], [0, 8, 0], [0, 16, 0]], warp = [[0, 32, 0], [0, 64, 0], [0, 128, 0]], block = []}>
34| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 35-35
```mlir
35| module attributes {"ttg.compute-capability" = 0 : i32, "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 36-36
```mlir
36|   tt.func @extract_3d_linear_tensor(%arg0: tensor<2x256x128xi32, #ll1> {tt.divisibility = 16 : i32}) {
```
**EN:** This function-oriented block defines or enters `extract_3d_linear_tensor`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `extract_3d_linear_tensor` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 37-44
```mlir
37|     // CHECK-LABEL: llvm.func @extract_3d_linear_tensor
38|     // CHECK-COUNT-128: %{{.*}} = llvm.extractvalue %arg0[{{.*}}] : !llvm.struct
39|     // CHECK-COUNT-64: %{{[0-9]*}} = llvm.insertvalue %{{.*}} : !llvm.struct
40|     %72 = amdg.extract_slice %arg0 [0,0,0] : tensor<2x256x128xi32, #ll1> to tensor<1x256x128xi32, #ll2>
41|     tt.return
42|   }
43| }
44| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.extract_slice, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.extract_slice、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 45-45
```mlir
45| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 46-48
```mlir
46| 
47| #ll1 = #ttg.linear<{register=[[1], [256], [512]], lane=[[2], [4], [8], [16], [32], [64]], warp=[[128]], block=[]}>
48| #ll2 = #ttg.linear<{register=[[1]], lane=[[2], [4], [8], [16], [32], [64]], warp=[[128]], block=[]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 49-49
```mlir
49| module attributes {"ttg.compute-capability" = 0 : i32, "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 50-50
```mlir
50|   tt.func @extract_1d_linear_tensor(%arg0: tensor<1024xi32, #ll1> {tt.divisibility = 16 : i32}) {
```
**EN:** This function-oriented block defines or enters `extract_1d_linear_tensor`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `extract_1d_linear_tensor` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 51-58
```mlir
51|     // CHECK-LABEL: llvm.func @extract_1d_linear_tensor
52|     // CHECK-COUNT-8: %{{.*}} = llvm.extractvalue %arg0[{{.*}}] : !llvm.struct
53|     // CHECK-COUNT-2: %{{[0-9]*}} = llvm.insertvalue %{{.*}} : !llvm.struct
54|     %72 = amdg.extract_slice %arg0 [0] : tensor<1024xi32, #ll1> to tensor<256xi32, #ll2>
55|     tt.return
56|   }
57| }
58| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.extract_slice, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.extract_slice、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 59-59
```mlir
59| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 60-63
```mlir
60| 
61| // Input tensor broadcasts 4 registers along dimension 1, resulting in total 32 values in tensor and 16 values per [128x1] tile.
62| // Output tensor do not have redundancy in register and holds 4 values.
63| // Test checks that extract slice copies only 4 values from input to output.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 64-65
```mlir
64| #blocked1 = #ttg.linear<{register=[[0, 0], [0, 0], [1, 0], [2, 0], [128, 0]], lane=[[0, 0], [0, 0], [0, 0], [4, 0], [8, 0], [16, 0]], warp=[[0, 0], [32, 0], [64, 0]], block=[]}>
65| #blocked2 = #ttg.linear<{register=[                [1, 0], [2, 0]],           lane=[[0, 0], [0, 0], [0, 0], [4, 0], [8, 0], [16, 0]], warp=[[0, 0], [32, 0], [64, 0]], block=[]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 66-66
```mlir
66| module attributes {"ttg.compute-capability" = 0 : i32, "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 67-67
```mlir
67|   tt.func @extract_from_broadcasted_tensor(%arg0: tensor<256x1xi32, #blocked1> {tt.divisibility = 16 : i32}) {
```
**EN:** This function-oriented block defines or enters `extract_from_broadcasted_tensor`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `extract_from_broadcasted_tensor` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 68-75
```mlir
68|     // CHECK-LABEL: llvm.func @extract_from_broadcasted_tensor
69|     // CHECK-COUNT-32: %{{.*}} = llvm.extractvalue  %{{.*}} : !llvm.struct
70|     // CHECK-COUNT-4:  %{{.*}} = llvm.insertvalue %{{.*}} : !llvm.struct
71|     %0 = amdg.extract_slice %arg0 [0,0] : tensor<256x1xi32, #blocked1> to tensor<128x1xi32, #blocked2>
72|     tt.return
73|   }
74| }
75| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.extract_slice, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.extract_slice、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 76-76
```mlir
76| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 77-80
```mlir
77| 
78| // Input tensor do not have broadcasted registers, resulting in total 8 values in tensor and 4 values per [128x1] tile.
79| // Output tensor broadcasts 4 registers along dimension 1 and total 16 values.
80| // Test checks that extract slice duplicates 4 values from input in 16 output values.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 81-82
```mlir
81| #blocked1 = #ttg.linear<{register=[                [1, 0], [2, 0], [128, 0]], lane=[[0, 0], [0, 0], [0, 0], [4, 0], [8, 0], [16, 0]], warp=[[0, 0], [32, 0], [64, 0]], block=[]}>
82| #blocked2 = #ttg.linear<{register=[[0, 0], [0, 0], [1, 0], [2, 0]],           lane=[[0, 0], [0, 0], [0, 0], [4, 0], [8, 0], [16, 0]], warp=[[0, 0], [32, 0], [64, 0]], block=[]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 83-83
```mlir
83| module attributes {"ttg.compute-capability" = 0 : i32, "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 84-84
```mlir
84|   tt.func @extract_to_broadcasted_tensor(%arg0: tensor<256x1xi32, #blocked1> {tt.divisibility = 16 : i32}) {
```
**EN:** This function-oriented block defines or enters `extract_to_broadcasted_tensor`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `extract_to_broadcasted_tensor` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 85-91
```mlir
85|     // CHECK-LABEL: llvm.func @extract_to_broadcasted_tensor
86|     // CHECK-COUNT-8: %{{.*}} = llvm.extractvalue  %{{.*}} : !llvm.struct
87|     // CHECK-COUNT-16:  %{{.*}} = llvm.insertvalue %{{.*}} : !llvm.struct
88|     %72 = amdg.extract_slice %arg0 [0,0] : tensor<256x1xi32, #blocked1> to tensor<128x1xi32, #blocked2>
89|     tt.return
90|   }
91| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.extract_slice, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.extract_slice、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `--convert-triton-amdgpu-to-llvm="gfx-arch=gfx942"`
- **CN:** 主要 pass 选项：`-split-input-file`，`--convert-triton-amdgpu-to-llvm="gfx-arch=gfx942"`
- **EN:** Dominant operations include `tt.func`, `module`, `amdg.extract_slice`, `tt.return`.
- **CN:** 主要操作包括 `tt.func`、`module`、`amdg.extract_slice`、`tt.return`。
- **EN:** The file contains 5 independently testable section(s). Check styles used: CHECK-COUNT x12, CHECK-LABEL x6. Important labels include llvm.func @extract_2d_blocked_tensor, llvm.func @extract_2d_linear_tensor, llvm.func @extract_3d_linear_tensor, llvm.func @extract_1d_linear_tensor. It validates that the selected pass pipeline rewrites each labeled test case into the checked form.
- **CN:** 该文件包含 5 个可独立测试的分段。使用的检查类型：CHECK-COUNT ×12，CHECK-LABEL ×6。 关键标签包括 llvm.func @extract_2d_blocked_tensor，llvm.func @extract_2d_linear_tensor，llvm.func @extract_3d_linear_tensor，llvm.func @extract_1d_linear_tensor。 它验证所选 pass 流水线会把每个带标签的测试用例重写成检查所要求的形式。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `tt`: Triton high-level tensor/pointer operations.
- **CN:** `tt`：Triton 高层张量/指针操作。
- **EN:** `ttg`: TritonGPU layout, memory, and GPU-specific operations.
- **CN:** `ttg`：TritonGPU 布局、内存和 GPU 专用操作。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。