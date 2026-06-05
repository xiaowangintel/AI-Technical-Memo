# ops.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/ops.mlir`
- **EN:** Test case centered on `ops` behavior in Triton/TritonGPU IR.
- **CN:** 该测试围绕 Triton/TritonGPU IR 中的 `ops` 行为展开。
- **EN:** RUN pipeline(s): `// RUN: triton-opt --split-input-file %s | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt --split-input-file %s | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt --split-input-file %s | FileCheck %s
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt --split-input-file %s | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt --split-input-file %s | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-5
```mlir
3| // CHECK: #[[$WMMA_GEN1:.*]] = #ttg.amd_wmma<{{.*}}version = 1{{.*}}>
4| // CHECK: #[[$WMMA_GEN2:.*]] = #ttg.amd_wmma<{{.*}}version = 2{{.*}}>
5| // CHECK: #[[$WMMA_GEN3:.*]] = #ttg.amd_wmma<{{.*}}version = 3{{.*}}>
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 6-7
```mlir
6| #blocked = #ttg.blocked<{sizePerThread = [2, 2], threadsPerWarp = [4, 8], warpsPerCTA = [1, 1], order = [1, 0]}>
7| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 8-8
```mlir
8| module attributes {"ttg.target" = "cuda:0", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 9-9
```mlir
9|   // CHECK-LABEL: wmma_layout
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: wmma_layout anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: wmma_layout 这样的标签用于锚定匹配范围。

### Lines 10-11
```mlir
10|   tt.func @wmma_layout(%0: tensor<16x16xf16, #blocked>) {
11|     %1 = ttg.convert_layout %0 : tensor<16x16xf16, #blocked> -> tensor<16x16xf16, #ttg.amd_wmma<{version = 1, ctaLayout = {register = [], warp = []}}>>
```
**EN:** This function-oriented block defines or enters `wmma_layout`. Within it, the test exercises tt.func, layout conversions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `wmma_layout` 为核心。测试在其中演示 tt.func、布局转换，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 12-15
```mlir
12|     // CHECK:  %{{.+}} = ttg.convert_layout %{{.+}} : tensor<16x16xf16, #{{.+}}> -> tensor<16x16xf16, #[[$WMMA_GEN1]]>
13|     tt.return
14|   }
15| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 16-16
```mlir
16|   // CHECK-LABEL: wmma_dot_op_layout
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: wmma_dot_op_layout anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: wmma_dot_op_layout 这样的标签用于锚定匹配范围。

### Lines 17-18
```mlir
17|   tt.func @wmma_dot_op_layout(%0: tensor<16x16xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>) {
18|     %1 = ttg.convert_layout %0 : tensor<16x16xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked}>> -> tensor<16x16xf16, #ttg.dot_op<{opIdx = 1, parent = #ttg.amd_wmma<{version = 1, ctaLayout = {register = [], warp = []}}>, kWidth = 16}>>
```
**EN:** This function-oriented block defines or enters `wmma_dot_op_layout`. Within it, the test exercises tt.func, layout conversions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `wmma_dot_op_layout` 为核心。测试在其中演示 tt.func、布局转换，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 19-22
```mlir
19|     // CHECK:  %{{.+}} = ttg.convert_layout %{{.+}} : tensor<16x16xf16, #ttg.dot_op<{opIdx = 1, parent = #{{.+}}}>> -> tensor<16x16xf16, #ttg.dot_op<{opIdx = 1, parent = #[[$WMMA_GEN1]], kWidth = 16}>>
20|     tt.return
21|   }
22| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 23-23
```mlir
23|   // CHECK-LABEL: wmma_gen2_layout
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: wmma_gen2_layout anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: wmma_gen2_layout 这样的标签用于锚定匹配范围。

### Lines 24-25
```mlir
24|   tt.func @wmma_gen2_layout(%0: tensor<16x16xf16, #blocked>) {
25|     %1 = ttg.convert_layout %0 : tensor<16x16xf16, #blocked> -> tensor<16x16xf16, #ttg.amd_wmma<{version = 2, ctaLayout = {warp = []}}>>
```
**EN:** This function-oriented block defines or enters `wmma_gen2_layout`. Within it, the test exercises tt.func, layout conversions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `wmma_gen2_layout` 为核心。测试在其中演示 tt.func、布局转换，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 26-29
```mlir
26|     // CHECK:  %{{.+}} = ttg.convert_layout %{{.+}} : tensor<16x16xf16, #{{.+}}> -> tensor<16x16xf16, #[[$WMMA_GEN2]]>
27|     tt.return
28|   }
29| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 30-30
```mlir
30|   // CHECK-LABEL: wmma_gen2_dot_op_layout
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: wmma_gen2_dot_op_layout anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: wmma_gen2_dot_op_layout 这样的标签用于锚定匹配范围。

### Lines 31-32
```mlir
31|   tt.func @wmma_gen2_dot_op_layout(%0: tensor<16x16xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>) {
32|     %1 = ttg.convert_layout %0 : tensor<16x16xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked}>> -> tensor<16x16xf16, #ttg.dot_op<{opIdx = 1, parent = #ttg.amd_wmma<{version = 2, ctaLayout = {warp = []}}>, kWidth = 8}>>
```
**EN:** This function-oriented block defines or enters `wmma_gen2_dot_op_layout`. Within it, the test exercises tt.func, layout conversions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `wmma_gen2_dot_op_layout` 为核心。测试在其中演示 tt.func、布局转换，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 33-36
```mlir
33|     // CHECK:  %{{.+}} = ttg.convert_layout %{{.+}} : tensor<16x16xf16, #ttg.dot_op<{opIdx = 1, parent = #{{.+}}}>> -> tensor<16x16xf16, #ttg.dot_op<{opIdx = 1, parent = #[[$WMMA_GEN2]], kWidth = 8}>>
34|     tt.return
35|   }
36| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 37-37
```mlir
37|   // CHECK-LABEL: wmma_gen3_layout
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: wmma_gen3_layout anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: wmma_gen3_layout 这样的标签用于锚定匹配范围。

### Lines 38-39
```mlir
38|   tt.func @wmma_gen3_layout(%0: tensor<16x16xf32, #blocked>) {
39|     %1 = ttg.convert_layout %0 : tensor<16x16xf32, #blocked> -> tensor<16x16xf32, #ttg.amd_wmma<{version = 3, ctaLayout = {warp = []}, instrShape = [16, 16, 32]}>>
```
**EN:** This function-oriented block defines or enters `wmma_gen3_layout`. Within it, the test exercises tt.func, layout conversions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `wmma_gen3_layout` 为核心。测试在其中演示 tt.func、布局转换，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 40-43
```mlir
40|     // CHECK:  %{{.+}} = ttg.convert_layout %{{.+}} : tensor<16x16xf32, #{{.+}}> -> tensor<16x16xf32, #[[$WMMA_GEN3]]>
41|     tt.return
42|   }
43| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 44-44
```mlir
44|   // CHECK-LABEL: wmma_gen3_dot_op_layout
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: wmma_gen3_dot_op_layout anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: wmma_gen3_dot_op_layout 这样的标签用于锚定匹配范围。

### Lines 45-46
```mlir
45|   tt.func @wmma_gen3_dot_op_layout(%0: tensor<16x32xbf16, #ttg.dot_op<{opIdx = 0, parent = #blocked}>>) {
46|     %1 = ttg.convert_layout %0 : tensor<16x32xbf16, #ttg.dot_op<{opIdx = 0, parent = #blocked}>> -> tensor<16x32xbf16, #ttg.dot_op<{opIdx = 0, parent = #ttg.amd_wmma<{version = 3, ctaLayout = {warp = []}, instrShape = [16, 16, 32]}>, kWidth = 8}>>
```
**EN:** This function-oriented block defines or enters `wmma_gen3_dot_op_layout`. Within it, the test exercises tt.func, layout conversions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `wmma_gen3_dot_op_layout` 为核心。测试在其中演示 tt.func、布局转换，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 47-50
```mlir
47|     // CHECK:  %{{.+}} = ttg.convert_layout %{{.+}} : tensor<16x32xbf16, #ttg.dot_op<{opIdx = 0, parent = #{{.+}}}>> -> tensor<16x32xbf16, #ttg.dot_op<{opIdx = 0, parent = #[[$WMMA_GEN3]], kWidth = 8}>>
48|     tt.return
49|   }
50| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 51-51
```mlir
51| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 52-53
```mlir
52| 
53| #blocked= #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 54-55
```mlir
54| // CHECK: #[[$LINEAR:.*]] = #ttg.linear<{{.*}}>
55| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 56-56
```mlir
56| module attributes {"ttg.target" = "cuda:0", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 57-57
```mlir
57|   // CHECK-LABEL: @blocked_to_linear
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @blocked_to_linear anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @blocked_to_linear 这样的标签用于锚定匹配范围。

### Lines 58-60
```mlir
58|   tt.func @blocked_to_linear(%input: tensor<32x4xi8, #blocked>) {
59|     // The layout is the basic layout generated by DecomposeScaledBlocked
60|     %output = ttg.convert_layout %input {allocation.offset = 0 : i32} : tensor<32x4xi8, #blocked> -> tensor<32x4xi8, #ttg.linear<{register = [], lane = [[0, 1], [1, 0], [2, 0], [4, 0], [8, 0]], warp = [[0, 0], [16, 0]], block = []}>>
```
**EN:** This function-oriented block defines or enters `blocked_to_linear`. Within it, the test exercises tt.func, layout conversions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `blocked_to_linear` 为核心。测试在其中演示 tt.func、布局转换，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 61-65
```mlir
61|     // CHECK:  %{{.+}} = ttg.convert_layout %{{.+}} : tensor<32x4xi8, #blocked> -> tensor<32x4xi8, #[[$LINEAR]]>
62|     tt.return
63|   }
64| }
65| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 66-66
```mlir
66| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 67-69
```mlir
67| 
68| #shared0 = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = false, elementBitWidth = 16}>
69| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 70-70
```mlir
70| module attributes {"ttg.target" = "cuda:0", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 71-72
```mlir
71|   // CHECK-LABEL: memdesc
72|   // CHECK-SAME: !ttg.memdesc<1x64x16xf16, #{{.+}}>
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: memdesc anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: memdesc 这样的标签用于锚定匹配范围。

### Lines 73-76
```mlir
73|   tt.func @memdesc(%d : !ttg.memdesc<1x64x16xf16, #shared0, #smem>) {
74|     tt.return
75|   }
76| 
```
**EN:** This function-oriented block defines or enters `memdesc`. Within it, the test exercises tt.func, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `memdesc` 为核心。测试在其中演示 tt.func、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 77-78
```mlir
77|   // CHECK-LABEL: memdesc_with_alloc_shape
78|   // CHECK-SAME: !ttg.memdesc<64x16xf16, #{{.+}}, mutable, 2x64x16>
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: memdesc_with_alloc_shape anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: memdesc_with_alloc_shape 这样的标签用于锚定匹配范围。

### Lines 79-83
```mlir
79|   tt.func @memdesc_with_alloc_shape(%d : !ttg.memdesc<64x16xf16, #shared0, #smem, mutable, 2x64x16>){
80|     tt.return
81|   }
82| }
83| 
```
**EN:** This function-oriented block defines or enters `memdesc_with_alloc_shape`. Within it, the test exercises tt.func, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `memdesc_with_alloc_shape` 为核心。测试在其中演示 tt.func、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 84-84
```mlir
84| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 85-90
```mlir
85| 
86| #shared_cga_01 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0], CGALayout = [[0, 1]]}>
87| #shared_cga_10 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0], CGALayout = [[1, 0]]}>
88| #shared_cga_00 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0], CGALayout = [[0, 0]]}>
89| #shared = #ttg.shared_linear<{offset = [[0, 1], [0, 2], [1, 0], [4, 0]], block = [[2, 0]]}, alignment = 16>
90| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 91-91
```mlir
91| module attributes {"ttg.target" = "cuda:0", "ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 92-92
```mlir
92|   // CHECK-LABEL: @subslice_non_trivial_block_cga_01
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @subslice_non_trivial_block_cga_01 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @subslice_non_trivial_block_cga_01 这样的标签用于锚定匹配范围。

### Lines 93-93
```mlir
93|   tt.func @subslice_non_trivial_block_cga_01(%arg0: !ttg.memdesc<8x16xf32, #shared_cga_01, #smem>) {
```
**EN:** This function-oriented block defines or enters `subslice_non_trivial_block_cga_01`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `subslice_non_trivial_block_cga_01` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 94-98
```mlir
94|     // CHECK: ttg.memdesc_subslice %{{.*}}[0, 0]
95|     %0 = ttg.memdesc_subslice %arg0 [0, 0] : !ttg.memdesc<8x16xf32, #shared_cga_01, #smem> -> !ttg.memdesc<4x16xf32, #shared_cga_01, #smem>
96|     tt.return
97|   }
98| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.memdesc_subslice, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.memdesc_subslice、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 99-99
```mlir
99|   // CHECK-LABEL: @subslice_non_trivial_block_cga_10
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @subslice_non_trivial_block_cga_10 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @subslice_non_trivial_block_cga_10 这样的标签用于锚定匹配范围。

### Lines 100-100
```mlir
100|   tt.func @subslice_non_trivial_block_cga_10(%arg0: !ttg.memdesc<8x16xf32, #shared_cga_10, #smem>) {
```
**EN:** This function-oriented block defines or enters `subslice_non_trivial_block_cga_10`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `subslice_non_trivial_block_cga_10` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 101-105
```mlir
101|     // CHECK: ttg.memdesc_subslice %{{.*}}[0, 0]
102|     %0 = ttg.memdesc_subslice %arg0 [0, 0] : !ttg.memdesc<8x16xf32, #shared_cga_10, #smem> -> !ttg.memdesc<8x8xf32, #shared_cga_10, #smem>
103|     tt.return
104|   }
105| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.memdesc_subslice, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.memdesc_subslice、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 106-106
```mlir
106|   // CHECK-LABEL: @subslice_broadcasted_block_cga_00
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @subslice_broadcasted_block_cga_00 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @subslice_broadcasted_block_cga_00 这样的标签用于锚定匹配范围。

### Lines 107-107
```mlir
107|   tt.func @subslice_broadcasted_block_cga_00(%arg0: !ttg.memdesc<8x16xf32, #shared_cga_00, #smem>) {
```
**EN:** This function-oriented block defines or enters `subslice_broadcasted_block_cga_00`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `subslice_broadcasted_block_cga_00` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 108-112
```mlir
108|     // CHECK: ttg.memdesc_subslice %{{.*}}[0, 0]
109|     %0 = ttg.memdesc_subslice %arg0 [0, 0] : !ttg.memdesc<8x16xf32, #shared_cga_00, #smem> -> !ttg.memdesc<4x16xf32, #shared_cga_00, #smem>
110|     tt.return
111|   }
112| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.memdesc_subslice, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.memdesc_subslice、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 113-113
```mlir
113|   // CHECK-LABEL: @subview_split_on_cta_dim
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @subview_split_on_cta_dim anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @subview_split_on_cta_dim 这样的标签用于锚定匹配范围。

### Lines 114-114
```mlir
114|   tt.func @subview_split_on_cta_dim(%arg0: !ttg.memdesc<8x4xf32, #shared, #smem>) {
```
**EN:** This function-oriented block defines or enters `subview_split_on_cta_dim`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `subview_split_on_cta_dim` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 115-120
```mlir
115|     // CHECK: ttg.memdesc_subslice %{{.*}}[0, 0]
116|     %a = ttg.memdesc_subslice %arg0 [0, 0] : !ttg.memdesc<8x4xf32, #shared, #smem> -> !ttg.memdesc<4x4xf32, #shared, #smem, 8x4>
117|     tt.return
118|   }
119| }
120| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.memdesc_subslice, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.memdesc_subslice、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 121-121
```mlir
121| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 122-124
```mlir
122| 
123| #shared = #ttg.padded_shared<[4:+4] {offset=[[1, 0], [2, 0], [0, 1], [0, 2]], block=[]}>
124| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 125-125
```mlir
125| module attributes {"ttg.target" = "gfx950", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 126-126
```mlir
126|   // CHECK-LABEL: memdesc_padded_same_rank_than_shape
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: memdesc_padded_same_rank_than_shape anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: memdesc_padded_same_rank_than_shape 这样的标签用于锚定匹配范围。

### Lines 127-130
```mlir
127|   tt.func @memdesc_padded_same_rank_than_shape(%d : !ttg.memdesc<4x4xf16, #shared, #smem, mutable, 3x4x4>) {
128|     tt.return
129|   }
130| 
```
**EN:** This function-oriented block defines or enters `memdesc_padded_same_rank_than_shape`. Within it, the test exercises tt.func, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `memdesc_padded_same_rank_than_shape` 为核心。测试在其中演示 tt.func、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 131-131
```mlir
131|   // CHECK-LABEL: memdesc_padded_with_pipeline_dim
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: memdesc_padded_with_pipeline_dim anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: memdesc_padded_with_pipeline_dim 这样的标签用于锚定匹配范围。

### Lines 132-136
```mlir
132|   tt.func @memdesc_padded_with_pipeline_dim(%d : !ttg.memdesc<3x4x4xf32, #shared, #smem, mutable>){
133|     tt.return
134|   }
135| }
136| 
```
**EN:** This function-oriented block defines or enters `memdesc_padded_with_pipeline_dim`. Within it, the test exercises tt.func, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `memdesc_padded_with_pipeline_dim` 为核心。测试在其中演示 tt.func、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 137-137
```mlir
137| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 138-144
```mlir
138| 
139| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 16, rank = 4}>
140| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 16}>
141| #shared2 = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = true, elementBitWidth = 32}>
142| #shared_linear_16 = #ttg.shared_linear<{offset = [[0, 1], [0, 2], [0, 4], [0, 8], [1, 0], [2, 4], [4, 8], [8, 0]]}, alignment = 512>
143| #shared_linear_equiv = #ttg.shared_linear<{offset = [[0, 0, 1, 0], [0, 1, 0, 0], [0, 2, 0, 0], [0, 4, 0, 0], [0, 0, 0, 1], [0, 2, 0, 2], [0, 4, 0, 4], [0, 0, 0, 8]]}, alignment = 512>
144| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 145-145
```mlir
145| module attributes {"ttg.target" = "cuda:0", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 146-147
```mlir
146|   // CHECK-LABEL: memdesc_reshape
147|   // CHECK: !ttg.memdesc<128x64xf16, #{{.+}}, mutable>
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: memdesc_reshape anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: memdesc_reshape 这样的标签用于锚定匹配范围。

### Lines 148-152
```mlir
148|   tt.func @memdesc_reshape(%d : !ttg.memdesc<32x1x4x64xf16, #shared, #smem, mutable>){
149|     %1 = ttg.memdesc_reshape %d : !ttg.memdesc<32x1x4x64xf16, #shared, #smem, mutable> -> !ttg.memdesc<128x64xf16, #shared1, #smem, mutable>
150|     tt.return
151|   }
152| 
```
**EN:** This function-oriented block defines or enters `memdesc_reshape`. Within it, the test exercises tt.func, ttg.memdesc_reshape, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `memdesc_reshape` 为核心。测试在其中演示 tt.func、ttg.memdesc_reshape、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 153-154
```mlir
153|   // CHECK-LABEL: memdesc_reshape_equiv
154|   // CHECK: %[[R:.*]] = ttg.memdesc_reshape %{{.*}} : !ttg.memdesc<1x8x2x16xf32, #{{.*}}, #smem> -> !ttg.memdesc<16x16xf32, #{{.*}}, #smem>
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: memdesc_reshape_equiv anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: memdesc_reshape_equiv 这样的标签用于锚定匹配范围。

### Lines 155-159
```mlir
155|   tt.func @memdesc_reshape_equiv(%arg0 : !ttg.memdesc<1x8x2x16xf32, #shared_linear_equiv, #smem>) {
156|     %0 = ttg.memdesc_reshape %arg0 : !ttg.memdesc<1x8x2x16xf32, #shared_linear_equiv, #smem> -> !ttg.memdesc<16x16xf32, #shared2, #smem>
157|     tt.return
158|   }
159| 
```
**EN:** This function-oriented block defines or enters `memdesc_reshape_equiv`. Within it, the test exercises tt.func, ttg.memdesc_reshape, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `memdesc_reshape_equiv` 为核心。测试在其中演示 tt.func、ttg.memdesc_reshape、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 160-161
```mlir
160|   // CHECK-LABEL: memdesc_trans_equiv
161|   // CHECK: %[[T:.*]] = ttg.memdesc_trans %{{.*}} {order = array<i32: 1, 0>} : !ttg.memdesc<16x16xf32, #{{.*}}, #smem> -> !ttg.memdesc<16x16xf32, #{{.*}}, #smem>
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: memdesc_trans_equiv anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: memdesc_trans_equiv 这样的标签用于锚定匹配范围。

### Lines 162-167
```mlir
162|   tt.func @memdesc_trans_equiv(%arg0 : !ttg.memdesc<16x16xf32, #shared_linear_16, #smem>) {
163|     %0 = ttg.memdesc_trans %arg0 {order = array<i32: 1, 0>} : !ttg.memdesc<16x16xf32, #shared_linear_16, #smem> -> !ttg.memdesc<16x16xf32, #shared2, #smem>
164|     tt.return
165|   }
166| }
167| 
```
**EN:** This function-oriented block defines or enters `memdesc_trans_equiv`. Within it, the test exercises tt.func, ttg.memdesc_trans, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `memdesc_trans_equiv` 为核心。测试在其中演示 tt.func、ttg.memdesc_trans、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 168-168
```mlir
168| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 169-172
```mlir
169| 
170| #shared1d = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
171| #shared2d = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
172| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 173-173
```mlir
173| module attributes {"ttg.target" = "cuda:0", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 174-175
```mlir
174|   // CHECK-LABEL: memdesc_reinterpret_layout_rank_increase
175|   // CHECK: ttg.memdesc_reinterpret
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: memdesc_reinterpret_layout_rank_increase anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: memdesc_reinterpret_layout_rank_increase 这样的标签用于锚定匹配范围。

### Lines 176-180
```mlir
176|   tt.func @memdesc_reinterpret_layout_rank_increase(%arg0 : !ttg.memdesc<32x2xi32, #shared1d, #smem, mutable>) {
177|     %0 = ttg.memdesc_reinterpret %arg0 : !ttg.memdesc<32x2xi32, #shared1d, #smem, mutable> -> !ttg.memdesc<32x2xi32, #shared2d, #smem, mutable>
178|     tt.return
179|   }
180| 
```
**EN:** This function-oriented block defines or enters `memdesc_reinterpret_layout_rank_increase`. Within it, the test exercises tt.func, ttg.memdesc_reinterpret, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `memdesc_reinterpret_layout_rank_increase` 为核心。测试在其中演示 tt.func、ttg.memdesc_reinterpret、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 181-182
```mlir
181|   // CHECK-LABEL: memdesc_reinterpret_layout_rank_decrease
182|   // CHECK: ttg.memdesc_reinterpret
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: memdesc_reinterpret_layout_rank_decrease anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: memdesc_reinterpret_layout_rank_decrease 这样的标签用于锚定匹配范围。

### Lines 183-188
```mlir
183|   tt.func @memdesc_reinterpret_layout_rank_decrease(%arg0 : !ttg.memdesc<32x2xi32, #shared2d, #smem, mutable>) {
184|     %0 = ttg.memdesc_reinterpret %arg0 : !ttg.memdesc<32x2xi32, #shared2d, #smem, mutable> -> !ttg.memdesc<32x2xi32, #shared1d, #smem, mutable>
185|     tt.return
186|   }
187| }
188| 
```
**EN:** This function-oriented block defines or enters `memdesc_reinterpret_layout_rank_decrease`. Within it, the test exercises tt.func, ttg.memdesc_reinterpret, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `memdesc_reinterpret_layout_rank_decrease` 为核心。测试在其中演示 tt.func、ttg.memdesc_reinterpret、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 189-189
```mlir
189| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 190-191
```mlir
190| 
191| // CHECK: #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 16, CGALayout = {{\[\[1, 0, 0, 0, 0\]\]}}}>
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 192-193
```mlir
192| #shared_rank_5 = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 16, CGALayout = [[1, 0, 0, 0, 0]]}>
193| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 194-194
```mlir
194| module attributes {"ttg.target" = "cuda:0", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 195-200
```mlir
195|   tt.func @nvmma_rank_inference(%arg0 : !ttg.memdesc<1x1x1x1x32xf16, #shared_rank_5, #smem>) {
196|     tt.return
197|   }
198| }
199| 
200| 
```
**EN:** This function-oriented block defines or enters `nvmma_rank_inference`. Within it, the test exercises tt.func, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `nvmma_rank_inference` 为核心。测试在其中演示 tt.func、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 201-201
```mlir
201| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 202-203
```mlir
202| 
203| // CHECK-LABEL: @warp_specialize_nothing
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @warp_specialize_nothing anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @warp_specialize_nothing 这样的标签用于锚定匹配范围。

### Lines 204-204
```mlir
204| tt.func @warp_specialize_nothing() {
```
**EN:** This function-oriented block defines or enters `warp_specialize_nothing`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `warp_specialize_nothing` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 205-206
```mlir
205|   // CHECK-NEXT: ttg.warp_specialize()
206|   ttg.warp_specialize()
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.warp_specialize. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.warp_specialize。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 207-208
```mlir
207|   // CHECK-NEXT: default {
208|   default {
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 209-210
```mlir
209|     // CHECK-NEXT: ttg.warp_yield
210|     ttg.warp_yield
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.warp_yield. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.warp_yield。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 211-215
```mlir
211|   // CHECK-NEXT: } : () -> ()
212|   } : () -> ()
213|   tt.return
214| }
215| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 216-216
```mlir
216| // CHECK-LABEL: @warp_specialize_no_partitions
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @warp_specialize_no_partitions anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @warp_specialize_no_partitions 这样的标签用于锚定匹配范围。

### Lines 217-217
```mlir
217| tt.func @warp_specialize_no_partitions(%arg0: i32, %arg1: i64) -> i64 {
```
**EN:** This function-oriented block defines or enters `warp_specialize_no_partitions`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `warp_specialize_no_partitions` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 218-219
```mlir
218|   // CHECK-NEXT: %0 = ttg.warp_specialize(%arg0)
219|   %0 = ttg.warp_specialize(%arg0)
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.warp_specialize. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.warp_specialize。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 220-221
```mlir
220|   // CHECK-NEXT: default {
221|   default {
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 222-223
```mlir
222|     // CHECK-NEXT: ttg.warp_yield %arg1 : i64
223|     ttg.warp_yield %arg1 : i64
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.warp_yield. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.warp_yield。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 224-228
```mlir
224|   // CHECK-NEXT: } : (i32) -> i64
225|   } : (i32) -> i64
226|   tt.return %0 : i64
227| }
228| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 229-229
```mlir
229| // CHECK-LABEL: @warp_specialize_partitions
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @warp_specialize_partitions anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @warp_specialize_partitions 这样的标签用于锚定匹配范围。

### Lines 230-230
```mlir
230| tt.func @warp_specialize_partitions(%arg0: i32, %arg1: i64) -> i64 {
```
**EN:** This function-oriented block defines or enters `warp_specialize_partitions`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `warp_specialize_partitions` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 231-232
```mlir
231|   // CHECK-NEXT: %0 = ttg.warp_specialize(%arg0)
232|   %0 = ttg.warp_specialize(%arg0)
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.warp_specialize. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.warp_specialize。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 233-234
```mlir
233|   // CHECK-NEXT: default {
234|   default {
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 235-236
```mlir
235|     // CHECK-NEXT: ttg.warp_yield %arg1 : i64
236|     ttg.warp_yield %arg1 : i64
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.warp_yield. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.warp_yield。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 237-238
```mlir
237|   // CHECK-NEXT: }
238|   }
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 239-240
```mlir
239|   // CHECK-NEXT: partition0(%arg2: i32) num_warps(4) {
240|   partition0(%arg2: i32) num_warps(4) {
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 241-242
```mlir
241|     // CHECK-NEXT: arith.addi %arg2, %arg2 : i32
242|     %1 = arith.addi %arg2, %arg2 : i32
```
**EN:** This operational block carries the concrete IR steps for the test, centered on integer additions. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 整数加法。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 243-244
```mlir
243|     // CHECK-NEXT: ttg.warp_return
244|     ttg.warp_return
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.warp_return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.warp_return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 245-246
```mlir
245|   // CHECK-NEXT: }
246|   }
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 247-248
```mlir
247|   // CHECK-NEXT: partition1(%arg2: i32) num_warps(1) {
248|   partition1(%arg2: i32) num_warps(1) {
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 249-250
```mlir
249|     // CHECK-NEXT: ttg.warp_return
250|     ttg.warp_return
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.warp_return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.warp_return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 251-252
```mlir
251|   // CHECK-NEXT: }
252|   }
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 253-254
```mlir
253|   // CHECK-NEXT: partition2(%arg2: i32) num_warps(8) {
254|   partition2(%arg2: i32) num_warps(8) {
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 255-256
```mlir
255|     // CHECK-NEXT: arith.muli
256|     %1 = arith.muli %arg2, %arg2 : i32
```
**EN:** This operational block carries the concrete IR steps for the test, centered on integer multiplications. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 整数乘法。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 257-258
```mlir
257|     // CHECK-NEXT: ttg.warp_return
258|     ttg.warp_return
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.warp_return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.warp_return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 259-263
```mlir
259|   // CHECK-NEXT: } : (i32) -> i64
260|   } : (i32) -> i64
261|   tt.return %0 : i64
262| }
263| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 264-264
```mlir
264| // CHECK-LABEL: @warp_specialize_multiple_args
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @warp_specialize_multiple_args anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @warp_specialize_multiple_args 这样的标签用于锚定匹配范围。

### Lines 265-265
```mlir
265| tt.func @warp_specialize_multiple_args_res(%arg0: i32, %arg1: i32) -> (i32, i32) {
```
**EN:** This function-oriented block defines or enters `warp_specialize_multiple_args_res`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `warp_specialize_multiple_args_res` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 266-267
```mlir
266|   // CHECK-NEXT: %0:2 = ttg.warp_specialize(%arg0, %arg1)
267|   %0:2 = ttg.warp_specialize(%arg0, %arg1)
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.warp_specialize. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.warp_specialize。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 268-269
```mlir
268|   // CHECK-NEXT: default {
269|   default {
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 270-271
```mlir
270|     // CHECK-NEXT: ttg.warp_yield %arg0, %arg1 : i32, i32
271|     ttg.warp_yield %arg0, %arg1 : i32, i32
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.warp_yield. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.warp_yield。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 272-273
```mlir
272|   // CHECK-NEXT: }
273|   }
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 274-275
```mlir
274|   // CHECK-NEXT: partition0(%arg2: i32, %arg3: i32) num_warps(4) {
275|   partition0(%arg2: i32, %arg3: i32) num_warps(4) {
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 276-277
```mlir
276|     // CHECK-NEXT: arith.addi %arg2, %arg3 : i32
277|     %1 = arith.addi %arg2, %arg3 : i32
```
**EN:** This operational block carries the concrete IR steps for the test, centered on integer additions. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 整数加法。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 278-279
```mlir
278|     // CHECK-NEXT: ttg.warp_return
279|     ttg.warp_return
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.warp_return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.warp_return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 280-284
```mlir
280|   // CHECK-NEXT: } : (i32, i32) -> (i32, i32)
281|   } : (i32, i32) -> (i32, i32)
282|   tt.return %0#0, %0#1 : i32, i32
283| }
284| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 285-285
```mlir
285| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 286-287
```mlir
286| 
287| // CHECK-DAG: [[BLOCKED_1_WARPS:#.*]] = #ttg.blocked{{.*}} warpsPerCTA = [1]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 288-288
```mlir
288| #blocked_1_warps = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 289-289
```mlir
289| // CHECK-DAG: [[BLOCKED_2_WARPS:#.*]] = #ttg.blocked{{.*}} warpsPerCTA = [2]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 290-290
```mlir
290| #blocked_2_warps = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [2], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 291-291
```mlir
291| // CHECK-DAG: [[BLOCKED_4_WARPS:#.*]] = #ttg.blocked{{.*}} warpsPerCTA = [4]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 292-292
```mlir
292| #blocked_4_warps = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 293-293
```mlir
293| // CHECK-DAG: [[BLOCKED_8_WARPS:#.*]] = #ttg.blocked{{.*}} warpsPerCTA = [8]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 294-295
```mlir
294| #blocked_8_warps = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [8], order = [0]}>
295| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 296-297
```mlir
296| module attributes {"ttg.num-warps" = 4 : i32} {
297| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 298-298
```mlir
298| // CHECK: @function_scope
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 299-299
```mlir
299| tt.func @function_scope() attributes {"ttg.num-warps" = 8 : i32} {
```
**EN:** This function-oriented block defines or enters `function_scope`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `function_scope` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 300-304
```mlir
300|   // CHECK-NEXT: tt.make_range {{.*}} tensor<128xi32, [[BLOCKED_8_WARPS]]>
301|   tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #blocked_8_warps>
302|   tt.return
303| }
304| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on lane/block index ranges, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 lane/block 索引范围、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 305-305
```mlir
305| // CHECK: @function_no_scope
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 306-306
```mlir
306| tt.func @function_no_scope() {
```
**EN:** This function-oriented block defines or enters `function_no_scope`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `function_no_scope` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 307-308
```mlir
307|   // CHECK-NEXT: tt.make_range {{.*}} tensor<128xi32, [[BLOCKED_4_WARPS]]>
308|   tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #blocked_4_warps>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on lane/block index ranges. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 lane/block 索引范围。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 309-313
```mlir
309|   // CHECK-NEXT: ttg.warp_specialize()
310|   ttg.warp_specialize()
311|   default {
312|     ttg.warp_yield
313|   }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.warp_specialize, ttg.warp_yield. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.warp_specialize、ttg.warp_yield。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 314-315
```mlir
314|   // CHECK: partition0() num_warps(2)
315|   partition0() num_warps(2) {
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 316-319
```mlir
316|     // CHECK-NEXT: tt.make_range {{.*}} tensor<128xi32, [[BLOCKED_2_WARPS]]>
317|     tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #blocked_2_warps>
318|     ttg.warp_return
319|   }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on lane/block index ranges, ttg.warp_return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 lane/block 索引范围、ttg.warp_return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 320-321
```mlir
320|   // CHECK: partition1() num_warps(1)
321|   partition1() num_warps(1) {
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 322-330
```mlir
322|     // CHECK-NEXT: tt.make_range {{.*}} tensor<128xi32, [[BLOCKED_1_WARPS]]>
323|     tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #blocked_1_warps>
324|     ttg.warp_return
325|   } : () -> ()
326|   tt.return
327| }
328| 
329| }
330| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on lane/block index ranges, ttg.warp_return, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 lane/block 索引范围、ttg.warp_return、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 331-331
```mlir
331| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 332-333
```mlir
332| 
333| // CHECK-DAG: [[$BLOCKED:#.*]] = #ttg.blocked
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 334-334
```mlir
334| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 335-335
```mlir
335| // CHECK-DAG: [[$LINEAR:#.*]] = #ttg.linear
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 336-337
```mlir
336| #linear = #ttg.linear<{register = [[0, 1], [16, 0], [32, 0], [64, 0]], lane = [[0, 0], [0, 0], [0, 0], [1, 0], [2, 0]], warp = [[4, 0], [8, 0]], block = []}>
337| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 338-338
```mlir
338| module attributes {"ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 339-339
```mlir
339| // CHECK-LABEL: @split_join_linear_mix
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @split_join_linear_mix anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @split_join_linear_mix 这样的标签用于锚定匹配范围。

### Lines 340-340
```mlir
340| tt.func @split_join_linear_mix(%arg: tensor<128x2xf32, #linear>) attributes {"ttg.num-warps" = 4 : i32} {
```
**EN:** This function-oriented block defines or enters `split_join_linear_mix`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `split_join_linear_mix` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 341-342
```mlir
341|   // CHECK-NEXT: tt.split %{{.*}} : tensor<128x2xf32, [[$LINEAR]]> -> tensor<128xf32, #ttg.slice<{dim = 1, parent = [[$BLOCKED]]}>>
342|   %lhs, %rhs = tt.split %arg : tensor<128x2xf32, #linear> -> tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.split. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.split。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 343-348
```mlir
343|   // CHECK-NEXT: tt.join %{{.*}}, %{{.*}} : tensor<128xf32, #ttg.slice<{dim = 1, parent = [[$BLOCKED]]}>> -> tensor<128x2xf32, [[$LINEAR]]>
344|   %j = tt.join %lhs, %rhs : tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<128x2xf32, #linear>
345|   tt.return
346| }
347| }
348| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.join, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.join、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 349-349
```mlir
349| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 350-351
```mlir
350| 
351| // CHECK-LABEL: @async_commit_group
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @async_commit_group anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @async_commit_group 这样的标签用于锚定匹配范围。

### Lines 352-352
```mlir
352| tt.func @async_commit_group(%arg0: !ttg.async.token) {
```
**EN:** This function-oriented block defines or enters `async_commit_group`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_commit_group` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 353-354
```mlir
353|   // CHECK-NEXT: ttg.async_commit_group
354|   ttg.async_commit_group
```
**EN:** This operational block carries the concrete IR steps for the test, centered on async copy commit groups. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 异步拷贝提交组。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 355-356
```mlir
355|   // CHECK-NEXT: ttg.async_commit_group tokens %arg0
356|   %0 = ttg.async_commit_group tokens %arg0
```
**EN:** This operational block carries the concrete IR steps for the test, centered on async copy commit groups. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 异步拷贝提交组。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 357-361
```mlir
357|   // CHECK-NEXT: ttg.async_commit_group
358|   %1 = ttg.async_commit_group
359|   tt.return
360| }
361| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on async copy commit groups, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 异步拷贝提交组、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 362-362
```mlir
362| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 363-367
```mlir
363| 
364| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [2, 2], warpsPerCTA = [1, 1], order = [1, 0]}>
365| #shared = #ttg.shared_linear<{offset = [[0, 1], [0, 2], [1, 0], [2, 2]]}, alignment = 16>
366| #smem = #ttg.shared_memory
367| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 368-368
```mlir
368| module attributes {"ttg.threads-per-warp" = 4 : i32, "ttg.num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 369-369
```mlir
369|   tt.func @round_trip(%arg0: tensor<4x4xf32, #blocked>) -> tensor<4x4xf32, #blocked> {
```
**EN:** This function-oriented block defines or enters `round_trip`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `round_trip` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 370-377
```mlir
370|     // CHECK: ttg.local_alloc
371|     // CHECK-SAME: !ttg.memdesc<4x4xf32, #shared
372|     %alloc = ttg.local_alloc %arg0 : (tensor<4x4xf32, #blocked>) -> !ttg.memdesc<4x4xf32, #shared, #smem, mutable>
373|     ttg.local_store %arg0, %alloc : tensor<4x4xf32, #blocked> -> !ttg.memdesc<4x4xf32, #shared, #smem, mutable>
374|     %loaded = ttg.local_load %alloc : !ttg.memdesc<4x4xf32, #shared, #smem, mutable> -> tensor<4x4xf32, #blocked>
375|     tt.return %loaded : tensor<4x4xf32, #blocked>
376|   }
377| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, local/shared memory stores, local/shared memory loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、本地/共享内存存储、本地/共享内存加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `--split-input-file`
- **CN:** 主要 pass 选项：`--split-input-file`
- **EN:** Dominant operations include `tt.func`, `tt.return`, `module`, `ttg.convert_layout`, `ttg.warp_return`, `ttg.warp_specialize`, `ttg.warp_yield`, `ttg.memdesc_subslice`, `tt.make_range`, `ttg.async_commit_group`.
- **CN:** 主要操作包括 `tt.func`、`tt.return`、`module`、`ttg.convert_layout`、`ttg.warp_return`、`ttg.warp_specialize`、`ttg.warp_yield`、`ttg.memdesc_subslice`、`tt.make_range`、`ttg.async_commit_group`。
- **EN:** The file contains 12 independently testable section(s). Check styles used: CHECK-NEXT x41, CHECK x26, CHECK-LABEL x26, CHECK-DAG x6. Important labels include wmma_layout, wmma_dot_op_layout, wmma_gen2_layout, wmma_gen2_dot_op_layout. It validates that the selected pass pipeline rewrites each labeled test case into the checked form.
- **CN:** 该文件包含 12 个可独立测试的分段。使用的检查类型：CHECK-NEXT ×41，CHECK ×26，CHECK-LABEL ×26，CHECK-DAG ×6。 关键标签包括 wmma_layout，wmma_dot_op_layout，wmma_gen2_layout，wmma_gen2_dot_op_layout。 它验证所选 pass 流水线会把每个带标签的测试用例重写成检查所要求的形式。

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