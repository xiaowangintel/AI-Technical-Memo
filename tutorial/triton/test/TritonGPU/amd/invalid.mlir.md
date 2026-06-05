# invalid.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/amd/invalid.mlir`
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

### Lines 3-3
```mlir
3| // expected-error @+1 {{WMMA version must be in the [1, 3] range}}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 4-4
```mlir
4| #wmma = #ttg.amd_wmma<{version = 0, isTranspose = false, ctaLayout = {warp = [[0, 1], [1, 0]]}}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 5-5
```mlir
5| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 6-11
```mlir
 6|     tt.func public @fn(%arg0: !tt.ptr<i32>) {
 7|         %t = tt.splat %arg0 : !tt.ptr<i32,1> -> tensor<32x32x!tt.ptr<i32,1>, #wmma>
 8|         tt.return
 9|     }
10| }
11| 
```
**EN:** This function-oriented block defines or enters `fn`. Within it, the test exercises tt.func, broadcasted scalars or pointers, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fn` 为核心。测试在其中演示 tt.func、广播后的标量或指针、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 12-12
```mlir
12| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 13-15
```mlir
13| 
14| #blocked = #ttg.blocked<{sizePerThread = [4, 4], threadsPerWarp = [8, 8], warpsPerCTA = [1, 1], order = [1, 0]}>
15| #linear = #ttg.linear<{register = [[0, 1], [0, 2], [1, 0], [2, 0]], lane = [[0, 4], [0, 8], [0, 16], [4, 0], [8, 0], [16, 0]], warp = [], block = []}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 16-16
```mlir
16| module attributes {"ttg.target" = "hip:gfx942", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 17-17
```mlir
17|   tt.func @amd_in_thread_transpose_wrong_output_encoding(%arg0: tensor<32x32xf16, #blocked>) {
```
**EN:** This function-oriented block defines or enters `amd_in_thread_transpose_wrong_output_encoding`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `amd_in_thread_transpose_wrong_output_encoding` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 18-35
```mlir
18| // expected-error-re @+15 {{Expect output layout to be transposed per thread:{{.*}}- register=1 -> (1, 0){{.*}}register=2 -> (2, 0){{.*}}register=4 -> (0, 1){{.*}}register=8 -> (0, 2)}}
19| // Full expected layout is following:
20| // - register=1 -> (1, 0)
21| //   register=2 -> (2, 0)
22| //   register=4 -> (0, 1)
23| //   register=8 -> (0, 2)}}
24| // - lane=1 -> (0, 4)
25| //   lane=2 -> (0, 8)
26| //   lane=4 -> (0, 16)
27| //   lane=8 -> (4, 0)
28| //   lane=16 -> (8, 0)
29| //   lane=32 -> (16, 0)
30| // - warp is a size 1 dimension
31| // - block is a size 1 dimension
32| // where out dims are: [dim0 (size 32), dim1 (size 32)]
33|     %0 = amdg.in_thread_transpose %arg0 : tensor<32x32xf16, #blocked> -> tensor<32x32xf16, #linear>
34|     tt.return
35|   }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.in_thread_transpose, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.in_thread_transpose、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 36-37
```mlir
36| }
37| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 38-38
```mlir
38| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 39-41
```mlir
39| 
40| #mfma = #ttg.amd_mfma<{version = 2, warpsPerCTA = [4, 1], instrShape = [16, 16, 16], isTransposed = true}>
41| #linear = #ttg.linear<{register = [[1, 0], [2, 0], [0, 1], [0, 2]], lane = [[0, 4], [0, 8], [0, 16], [4, 0], [8, 0], [16, 0]], warp = [], block = []}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 42-42
```mlir
42| module attributes {"ttg.target" = "hip:gfx942", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 43-43
```mlir
43|   tt.func @amd_in_thread_transpose_wrong_input_encoding(%arg0: tensor<32x32xf16, #mfma>) {
```
**EN:** This function-oriented block defines or enters `amd_in_thread_transpose_wrong_input_encoding`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `amd_in_thread_transpose_wrong_input_encoding` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 44-49
```mlir
44| // expected-error @+1 {{Expect input tensor in Blocked encoding}}
45|     %0 = amdg.in_thread_transpose %arg0 : tensor<32x32xf16, #mfma> -> tensor<32x32xf16, #linear>
46|     tt.return
47|   }
48| }
49| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.in_thread_transpose, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.in_thread_transpose、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 50-50
```mlir
50| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 51-53
```mlir
51| 
52| #blocked = #ttg.blocked<{sizePerThread = [4, 4], threadsPerWarp = [8, 8], warpsPerCTA = [1, 1], order = [1, 0]}>
53| #linear = #ttg.linear<{register = [[1, 0], [2, 0], [0, 1], [0, 2]], lane = [[0, 4], [0, 8], [0, 16], [4, 0], [8, 0], [16, 0]], warp = [], block = []}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 54-54
```mlir
54| module attributes {"ttg.target" = "hip:gfx942", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 55-55
```mlir
55|   tt.func @amd_in_thread_transpose_wrong_shape(%arg0: tensor<64x64xf16, #blocked>) {
```
**EN:** This function-oriented block defines or enters `amd_in_thread_transpose_wrong_shape`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `amd_in_thread_transpose_wrong_shape` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 56-61
```mlir
56| // expected-error @+1 {{Expect equal input and output shapes}}
57|     %0 = amdg.in_thread_transpose %arg0 : tensor<64x64xf16, #blocked> -> tensor<32x32xf16, #linear>
58|     tt.return
59|   }
60| }
61| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.in_thread_transpose, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.in_thread_transpose、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 62-62
```mlir
62| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 63-65
```mlir
63| 
64| #blocked = #ttg.blocked<{sizePerThread = [4, 4], threadsPerWarp = [8, 8], warpsPerCTA = [1, 1], order = [1, 0]}>
65| #linear = #ttg.linear<{register = [[1, 0], [2, 0], [0, 1], [0, 2]], lane = [[0, 4], [0, 8], [0, 16], [4, 0], [8, 0], [16, 0]], warp = [], block = []}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 66-66
```mlir
66| module attributes {"ttg.target" = "hip:gfx942", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 67-67
```mlir
67|   tt.func @amd_in_thread_transpose_wrong_dtype(%arg0: tensor<32x32xf16, #blocked>) {
```
**EN:** This function-oriented block defines or enters `amd_in_thread_transpose_wrong_dtype`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `amd_in_thread_transpose_wrong_dtype` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 68-73
```mlir
68| // expected-error @+1 {{Expect input and output tensor to have same dtype}}
69|     %0 = amdg.in_thread_transpose %arg0 : tensor<32x32xf16, #blocked> -> tensor<32x32xf32, #linear>
70|     tt.return
71|   }
72| }
73| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.in_thread_transpose, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.in_thread_transpose、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 74-74
```mlir
74| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 75-77
```mlir
75| 
76| #blocked = #ttg.blocked<{sizePerThread = [1, 4, 4], threadsPerWarp = [1, 8, 8], warpsPerCTA = [1, 1, 1], order = [2, 1, 0]}>
77| #linear = #ttg.linear<{register = [[0, 1, 0], [0, 2, 0], [0, 0, 1], [0, 0, 2]], lane = [[0, 0, 4], [0, 0, 8], [0, 0, 16], [0, 4, 0], [0, 8, 0], [0, 16, 0]], warp = [], block = []}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 78-78
```mlir
78| module attributes {"ttg.target" = "hip:gfx942", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 79-79
```mlir
79|   tt.func @amd_in_thread_transpose_3d_shape(%arg0: tensor<2x32x32xf16, #blocked>) {
```
**EN:** This function-oriented block defines or enters `amd_in_thread_transpose_3d_shape`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `amd_in_thread_transpose_3d_shape` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 80-85
```mlir
80| // expected-error @+1 {{Expect 2d tensor}}
81|     %0 = amdg.in_thread_transpose %arg0 : tensor<2x32x32xf16, #blocked> -> tensor<2x32x32xf16, #linear>
82|     tt.return
83|   }
84| }
85| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.in_thread_transpose, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.in_thread_transpose、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 86-86
```mlir
86| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 87-93
```mlir
87| 
88| #mma32 = #ttg.amd_mfma<{version = 4, warpsPerCTA = [2, 2], instrShape = [32, 32, 16], isTransposed = true}>
89| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0, 1]}>
90| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
91| #smem = #ttg.shared_memory
92| #blocked = #ttg.blocked<{sizePerThread = [4, 4], threadsPerWarp = [8, 8], warpsPerCTA = [1, 1], order = [1, 0]}>
93| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 94-94
```mlir
94| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 95-95
```mlir
95|   tt.func @local_load_packed_tranposed_wrong_op_idx(%arg0: !ttg.memdesc<16x64xi8, #shared, #smem, mutable>, %arg1: !ttg.memdesc<64x16xi8, #shared1, #smem, mutable>) {
```
**EN:** This function-oriented block defines or enters `local_load_packed_tranposed_wrong_op_idx`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `local_load_packed_tranposed_wrong_op_idx` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 96-100
```mlir
 96| // expected-error @+1 {{Order of dimensions don't match expected}}
 97|     %1 = amdg.local_load_packed_tranposed %arg0 : !ttg.memdesc<16x64xi8, #shared, #smem, mutable> -> tensor<32x32xi8, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 16}>>
 98|     tt.return
 99|   }
100| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.local_load_packed_tranposed, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.local_load_packed_tranposed、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 101-101
```mlir
101|   tt.func @local_load_packed_tranposed_wrong_op_idx2(%arg0: !ttg.memdesc<64x16xi8, #shared, #smem, mutable>) {
```
**EN:** This function-oriented block defines or enters `local_load_packed_tranposed_wrong_op_idx2`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `local_load_packed_tranposed_wrong_op_idx2` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 102-106
```mlir
102| // expected-error @+1 {{Input and output dimensions don't match after packing changes}}
103|     %1 = amdg.local_load_packed_tranposed %arg0 : !ttg.memdesc<64x16xi8, #shared, #smem, mutable> -> tensor<32x32xi8, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 16}>>
104|     tt.return
105|   }
106|   //  CHECK-LABEL: ds_transpose_t_fp4_mfma16
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.local_load_packed_tranposed, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.local_load_packed_tranposed、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 107-107
```mlir
107|   tt.func @local_load_packed_tranposed_wrong_shape(%arg0: !ttg.memdesc<8x128xi8, #shared, #smem, mutable>, %arg1: !ttg.memdesc<128x8xi8, #shared1, #smem, mutable>) {
```
**EN:** This function-oriented block defines or enters `local_load_packed_tranposed_wrong_shape`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `local_load_packed_tranposed_wrong_shape` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 108-114
```mlir
108| // expected-error @+1 {{only works with DotOperandEncodingAttr dst encoding}}
109|     %1 = amdg.local_load_packed_tranposed %arg0 : !ttg.memdesc<8x128xi8, #shared, #smem, mutable> -> tensor<256x128xi32, #blocked>
110|     tt.return
111|   }
112| 
113| }
114| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.local_load_packed_tranposed, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.local_load_packed_tranposed、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 115-115
```mlir
115| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 116-121
```mlir
116| 
117| #wmma_v3 = #ttg.amd_wmma<{version = 3, ctaLayout = {warp = [[0, 1], [1, 0]]}, instrShape = [16, 16, 32]}>
118| #wmma_v2 = #ttg.amd_wmma<{version = 2, ctaLayout = {warp = [[0, 1], [1, 0]]}, instrShape = [16, 16, 32]}>
119| #wmma_diff_warp = #ttg.amd_wmma<{version = 3, ctaLayout = {warp = [[0, 1], [0, 0]]}, instrShape = [16, 16, 32]}>
120| #wmma_diff_shape = #ttg.amd_wmma<{version = 3, ctaLayout = {warp = [[0, 1], [1, 0]]}, instrShape = [16, 16, 64]}>
121| #wmma_diff_transpose = #ttg.amd_wmma<{version = 3, ctaLayout = {warp = [[0, 1], [1, 0]]}, instrShape = [16, 16, 64], isTransposed = true}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 122-122
```mlir
122| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 123-127
```mlir
123|   tt.func @wmma_dot_incompatible_versions(
124|               %arg0: tensor<16x16xf16, #ttg.dot_op<{opIdx = 0, parent = #wmma_v3, kWidth = 8}>>,
125|               %arg1: tensor<16x16xf16, #ttg.dot_op<{opIdx = 1, parent = #wmma_v2, kWidth = 8}>>,
126|               %dst: tensor<16x16xf32, #wmma_v3>
127|   ) {
```
**EN:** This function-oriented block defines or enters `wmma_dot_incompatible_versions`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `wmma_dot_incompatible_versions` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 128-133
```mlir
128|     // expected-error @+2 {{'tt.dot' op failed to infer returned types}}
129|     // expected-error @+1 {{Incompatible parent encoding}}
130|     %0 = tt.dot %arg0, %arg1, %dst : tensor<16x16xf16, #ttg.dot_op<{opIdx = 0, parent = #wmma_v3, kWidth = 8}>> * tensor<16x16xf16, #ttg.dot_op<{opIdx = 1, parent = #wmma_v2, kWidth = 8}>> -> tensor<16x16xf32, #wmma_v3>
131|     tt.return
132|   }
133| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on dot-product or MMA-style math, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 点积或 MMA 风格计算、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 134-138
```mlir
134|   tt.func @wmma_dot_incompatible_warp_layouts(
135|               %arg0: tensor<16x16xf16, #ttg.dot_op<{opIdx = 0, parent = #wmma_v3, kWidth = 8}>>,
136|               %arg1: tensor<16x16xf16, #ttg.dot_op<{opIdx = 1, parent = #wmma_diff_warp, kWidth = 8}>>,
137|               %dst: tensor<16x16xf32, #wmma_v3>
138|   ) {
```
**EN:** This function-oriented block defines or enters `wmma_dot_incompatible_warp_layouts`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `wmma_dot_incompatible_warp_layouts` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 139-144
```mlir
139|     // expected-error @+2 {{'tt.dot' op failed to infer returned types}}
140|     // expected-error @+1 {{Incompatible parent encoding}}
141|     %0 = tt.dot %arg0, %arg1, %dst : tensor<16x16xf16, #ttg.dot_op<{opIdx = 0, parent = #wmma_v3, kWidth = 8}>> * tensor<16x16xf16, #ttg.dot_op<{opIdx = 1, parent = #wmma_diff_warp, kWidth = 8}>> -> tensor<16x16xf32, #wmma_v3>
142|     tt.return
143|   }
144| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on dot-product or MMA-style math, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 点积或 MMA 风格计算、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 145-149
```mlir
145|   tt.func @wmma_dot_incomptible_shapes(
146|               %arg0: tensor<16x16xf16, #ttg.dot_op<{opIdx = 0, parent = #wmma_v3, kWidth = 8}>>,
147|               %arg1: tensor<16x16xf16, #ttg.dot_op<{opIdx = 1, parent = #wmma_diff_shape, kWidth = 8}>>,
148|               %dst: tensor<16x16xf32, #wmma_v3>
149|   ) {
```
**EN:** This function-oriented block defines or enters `wmma_dot_incomptible_shapes`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `wmma_dot_incomptible_shapes` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 150-155
```mlir
150|     // expected-error @+2 {{'tt.dot' op failed to infer returned types}}
151|     // expected-error @+1 {{Incompatible parent encoding}}
152|     %0 = tt.dot %arg0, %arg1, %dst : tensor<16x16xf16, #ttg.dot_op<{opIdx = 0, parent = #wmma_v3, kWidth = 8}>> * tensor<16x16xf16, #ttg.dot_op<{opIdx = 1, parent = #wmma_diff_shape, kWidth = 8}>> -> tensor<16x16xf32, #wmma_v3>
153|     tt.return
154|   }
155| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on dot-product or MMA-style math, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 点积或 MMA 风格计算、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 156-160
```mlir
156|   tt.func @wmma_dot_incomptible_transpose(
157|               %arg0: tensor<16x16xf16, #ttg.dot_op<{opIdx = 0, parent = #wmma_v3, kWidth = 8}>>,
158|               %arg1: tensor<16x16xf16, #ttg.dot_op<{opIdx = 1, parent = #wmma_diff_transpose, kWidth = 8}>>,
159|               %dst: tensor<16x16xf32, #wmma_v3>
160|   ) {
```
**EN:** This function-oriented block defines or enters `wmma_dot_incomptible_transpose`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `wmma_dot_incomptible_transpose` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 161-167
```mlir
161|     // expected-error @+2 {{'tt.dot' op failed to infer returned types}}
162|     // expected-error @+1 {{Incompatible parent encoding}}
163|     %0 = tt.dot %arg0, %arg1, %dst : tensor<16x16xf16, #ttg.dot_op<{opIdx = 0, parent = #wmma_v3, kWidth = 8}>> * tensor<16x16xf16, #ttg.dot_op<{opIdx = 1, parent = #wmma_diff_transpose, kWidth = 8}>> -> tensor<16x16xf32, #wmma_v3>
164|     tt.return
165|   }
166| }
167| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on dot-product or MMA-style math, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 点积或 MMA 风格计算、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 168-168
```mlir
168| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 169-172
```mlir
169| 
170| #wmma_acc = #ttg.amd_wmma<{version = 3, isTranspose = true, ctaLayout = {warp = [[0, 1], [1, 0]]}, CGALayout = [[1, 0], [0, 1]], instrShape = [16, 16, 32]}>
171| #wmma_dim1 = #ttg.amd_wmma<{version = 3, isTranspose = true, ctaLayout = {warp = [[0, 1], [1, 0]]}, CGALayout = [[1, 0], [0, 0]], instrShape = [16, 16, 32]}>
172| #wmma_dim2 = #ttg.amd_wmma<{version = 3, isTranspose = true, ctaLayout = {warp = [[0, 1], [1, 0]]}, CGALayout = [[0, 0], [0, 1]], instrShape = [16, 16, 32]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 173-173
```mlir
173| module attributes {"ttg.num-ctas" = 4 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 174-178
```mlir
174|   tt.func @wmma_invalid_cga_split_operand_0(
175|               %arg0: tensor<32x32xf16, #ttg.dot_op<{opIdx = 0, parent = #wmma_dim2, kWidth = 8}>>,
176|               %arg1: tensor<32x32xf16, #ttg.dot_op<{opIdx = 1, parent = #wmma_dim2, kWidth = 8}>>,
177|               %dst: tensor<32x32xf32, #wmma_acc>
178|   ) {
```
**EN:** This function-oriented block defines or enters `wmma_invalid_cga_split_operand_0`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `wmma_invalid_cga_split_operand_0` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 179-183
```mlir
179|     // expected-error @+1 {{Incompatible CGA layout for operand 0}}
180|     %0 = tt.dot %arg0, %arg1, %dst : tensor<32x32xf16, #ttg.dot_op<{opIdx = 0, parent = #wmma_dim2, kWidth = 8}>> * tensor<32x32xf16, #ttg.dot_op<{opIdx = 1, parent = #wmma_dim2, kWidth = 8}>> -> tensor<32x32xf32, #wmma_acc>
181|     tt.return
182|   }
183| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on dot-product or MMA-style math, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 点积或 MMA 风格计算、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 184-188
```mlir
184|   tt.func @wmma_invalid_cga_split_operand_1(
185|               %arg0: tensor<32x32xf16, #ttg.dot_op<{opIdx = 0, parent = #wmma_dim1, kWidth = 8}>>,
186|               %arg1: tensor<32x32xf16, #ttg.dot_op<{opIdx = 1, parent = #wmma_dim1, kWidth = 8}>>,
187|               %dst: tensor<32x32xf32, #wmma_acc>
188|   ) {
```
**EN:** This function-oriented block defines or enters `wmma_invalid_cga_split_operand_1`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `wmma_invalid_cga_split_operand_1` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 189-193
```mlir
189|     // expected-error @+1 {{Incompatible CGA layout for operand 1}}
190|     %0 = tt.dot %arg0, %arg1, %dst : tensor<32x32xf16, #ttg.dot_op<{opIdx = 0, parent = #wmma_dim1, kWidth = 8}>> * tensor<32x32xf16, #ttg.dot_op<{opIdx = 1, parent = #wmma_dim1, kWidth = 8}>> -> tensor<32x32xf32, #wmma_acc>
191|     tt.return
192|   }
193| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on dot-product or MMA-style math, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 点积或 MMA 风格计算、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 194-198
```mlir
194|   tt.func @wmma_invalid_cga_split_accumulator(
195|               %arg0: tensor<32x32xf16, #ttg.dot_op<{opIdx = 0, parent = #wmma_dim2, kWidth = 8}>>,
196|               %arg1: tensor<32x32xf16, #ttg.dot_op<{opIdx = 1, parent = #wmma_dim2, kWidth = 8}>>,
197|               %dst: tensor<32x32xf32, #wmma_dim1>
198|   ) {
```
**EN:** This function-oriented block defines or enters `wmma_invalid_cga_split_accumulator`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `wmma_invalid_cga_split_accumulator` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 199-204
```mlir
199|     // expected-error @+1 {{Accumulator CGA layout should not broadcast or have repeated rows}}
200|     %0 = tt.dot %arg0, %arg1, %dst : tensor<32x32xf16, #ttg.dot_op<{opIdx = 0, parent = #wmma_dim2, kWidth = 8}>> * tensor<32x32xf16, #ttg.dot_op<{opIdx = 1, parent = #wmma_dim2, kWidth = 8}>> -> tensor<32x32xf32, #wmma_dim1>
201|     tt.return
202|   }
203| }
204| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on dot-product or MMA-style math, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 点积或 MMA 风格计算、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 205-205
```mlir
205| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 206-209
```mlir
206| 
207| #shared_32 = #ttg.padded_shared<[32:+4] {order = [1, 0], shape = [128, 64]}>
208| #shared_2_intervals = #ttg.padded_shared<[64:+4, 128:+4] {order = [1, 0], shape = [128, 64]}>
209| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 210-210
```mlir
210| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 211-215
```mlir
211|   tt.func public @interval_not_matching_innermost_block_dimension(
212|     %tensorDesc: !tt.tensordesc<128x64xf16>,
213|     %memDesc: !ttg.memdesc<128x64xf16, #shared_32, #smem, mutable>
214|   ) {
215|     %c0_i32 = arith.constant 0 : i32
```
**EN:** This function-oriented block defines or enters `interval_not_matching_innermost_block_dimension`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `interval_not_matching_innermost_block_dimension` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 216-220
```mlir
216|     // expected-error @+1 {{TDM store padding is only supported when padding interval equals the innermost block dimension}}
217|     amdg.async_tdm_copy_local_to_global %tensorDesc[%c0_i32, %c0_i32] from %memDesc: !ttg.memdesc<128x64xf16, #shared_32, #smem, mutable> -> !tt.tensordesc<128x64xf16>
218|     tt.return
219|   }
220| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.async_tdm_copy_local_to_global, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.async_tdm_copy_local_to_global、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 221-225
```mlir
221|   tt.func public @tdm_store_two_padding_intervals(
222|     %tensorDesc: !tt.tensordesc<128x64xf16>,
223|     %memDesc: !ttg.memdesc<128x64xf16, #shared_2_intervals, #smem, mutable>
224|   ) {
225|     %c0_i32 = arith.constant 0 : i32
```
**EN:** This function-oriented block defines or enters `tdm_store_two_padding_intervals`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tdm_store_two_padding_intervals` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 226-231
```mlir
226|     // expected-error @+1 {{TDM store only supports single interval paddings}}
227|     amdg.async_tdm_copy_local_to_global %tensorDesc[%c0_i32, %c0_i32] from %memDesc: !ttg.memdesc<128x64xf16, #shared_2_intervals, #smem, mutable> -> !tt.tensordesc<128x64xf16>
228|     tt.return
229|   }
230| }
231| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.async_tdm_copy_local_to_global, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.async_tdm_copy_local_to_global、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 232-232
```mlir
232| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 233-236
```mlir
233| 
234| // Gather with an index layout that distributes values across lanes (invalid).
235| // parent blocked: threadsPerWarp = [32, 1] → lanes map to dim 0.
236| // slice dim 1 → 1D tensor where each lane holds a different value.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 237-240
```mlir
237| #blocked_lane_dist = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [1, 1], order = [1, 0]}>
238| #slice_lane_dist = #ttg.slice<{dim = 1, parent = #blocked_lane_dist}>
239| #shared_gather = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
240| #smem_gather = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 241-241
```mlir
241| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 242-248
```mlir
242|   tt.func @tdm_gather_invalid_lane_distribution(
243|     %memDesc: !ttg.memdesc<32x128xf16, #shared_gather, #smem_gather, mutable>,
244|     %tensorDesc: !tt.tensordesc<32x128xf16>,
245|     %row_indices: tensor<32xi32, #slice_lane_dist>,
246|     %pred: i32
247|   ) {
248|     %c0_i32 = arith.constant 0 : i32
```
**EN:** This function-oriented block defines or enters `tdm_gather_invalid_lane_distribution`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tdm_gather_invalid_lane_distribution` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 249-254
```mlir
249|     // expected-error @+1 {{index layout distributes values across lanes}}
250|     %token = amdg.async_tdm_gather %tensorDesc[%row_indices, %c0_i32] to %memDesc, pred = %pred : tensor<32xi32, #slice_lane_dist>, !ttg.memdesc<32x128xf16, #shared_gather, #smem_gather, mutable> -> !tt.tensordesc<32x128xf16>
251|     tt.return
252|   }
253| }
254| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.async_tdm_gather, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.async_tdm_gather、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 255-255
```mlir
255| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 256-257
```mlir
256| 
257| // Scatter with padded shared layout where padding interval != innermost block dimension.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 258-259
```mlir
258| #shared_scatter_32 = #ttg.padded_shared<[32:+4] {order = [1, 0], shape = [8, 64]}>
259| // Scatter with two padding intervals (only single interval is supported).
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 260-261
```mlir
260| #shared_scatter_2_intervals = #ttg.padded_shared<[64:+4, 128:+4] {order = [1, 0], shape = [8, 64]}>
261| #smem_scatter = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 262-262
```mlir
262| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 263-268
```mlir
263|   tt.func public @scatter_interval_not_matching_innermost_block_dimension(
264|     %tensorDesc: !tt.tensordesc<8x64xf16>,
265|     %memDesc: !ttg.memdesc<8x64xf16, #shared_scatter_32, #smem_scatter, mutable>,
266|     %row_indices: tensor<8xi32>
267|   ) {
268|     %c0_i32 = arith.constant 0 : i32
```
**EN:** This function-oriented block defines or enters `scatter_interval_not_matching_innermost_block_dimension`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `scatter_interval_not_matching_innermost_block_dimension` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 269-273
```mlir
269|     // expected-error @+1 {{TDM scatter padding is only supported when padding interval equals the innermost block dimension}}
270|     amdg.async_tdm_scatter %tensorDesc[%row_indices, %c0_i32] from %memDesc : tensor<8xi32>, !ttg.memdesc<8x64xf16, #shared_scatter_32, #smem_scatter, mutable> -> !tt.tensordesc<8x64xf16>
271|     tt.return
272|   }
273| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.async_tdm_scatter, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.async_tdm_scatter、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 274-279
```mlir
274|   tt.func public @scatter_two_padding_intervals(
275|     %tensorDesc: !tt.tensordesc<8x64xf16>,
276|     %memDesc: !ttg.memdesc<8x64xf16, #shared_scatter_2_intervals, #smem_scatter, mutable>,
277|     %row_indices: tensor<8xi32>
278|   ) {
279|     %c0_i32 = arith.constant 0 : i32
```
**EN:** This function-oriented block defines or enters `scatter_two_padding_intervals`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `scatter_two_padding_intervals` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 280-285
```mlir
280|     // expected-error @+1 {{TDM scatter only supports single interval paddings}}
281|     amdg.async_tdm_scatter %tensorDesc[%row_indices, %c0_i32] from %memDesc : tensor<8xi32>, !ttg.memdesc<8x64xf16, #shared_scatter_2_intervals, #smem_scatter, mutable> -> !tt.tensordesc<8x64xf16>
282|     tt.return
283|   }
284| }
285| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.async_tdm_scatter, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.async_tdm_scatter、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 286-286
```mlir
286| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 287-288
```mlir
287| 
288| // warp_used_hint validation tests
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 289-290
```mlir
289| #shared_wb = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
290| #smem_wb = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 291-292
```mlir
291| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
292|   // hint == 0 has no active warps; rejected.
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 293-298
```mlir
293|   tt.func @warp_used_hint_zero(
294|     %tensorDesc: !tt.tensordesc<256x64xf16>,
295|     %memDesc: !ttg.memdesc<256x64xf16, #shared_wb, #smem_wb, mutable>,
296|     %pred: i32
297|   ) {
298|     %c0 = arith.constant 0 : i32
```
**EN:** This function-oriented block defines or enters `warp_used_hint_zero`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `warp_used_hint_zero` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 299-306
```mlir
299|     // expected-error @+1 {{warp_used_hint must have at least one bit set}}
300|     %0 = amdg.async_tdm_copy_global_to_local %tensorDesc[%c0, %c0] into %memDesc, pred = %pred {warp_used_hint = 0 : i32} : !tt.tensordesc<256x64xf16> -> !ttg.memdesc<256x64xf16, #shared_wb, #smem_wb, mutable>
301|     tt.return
302|   }
303| 
304|   // 0x69 (warps 0,3,5,6) is rejected: K=4 is a power of two but the
305|   // active set spans 3 warpId bit positions, not log2(K) = 2 -- a
306|   // non axis-aligned pattern is not supported.
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.async_tdm_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.async_tdm_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 307-312
```mlir
307|   tt.func @warp_used_hint_non_axis_aligned(
308|     %tensorDesc: !tt.tensordesc<256x64xf16>,
309|     %memDesc: !ttg.memdesc<256x64xf16, #shared_wb, #smem_wb, mutable>,
310|     %pred: i32
311|   ) {
312|     %c0 = arith.constant 0 : i32
```
**EN:** This function-oriented block defines or enters `warp_used_hint_non_axis_aligned`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `warp_used_hint_non_axis_aligned` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 313-319
```mlir
313|     // expected-error @+1 {{is not axis-aligned}}
314|     %0 = amdg.async_tdm_copy_global_to_local %tensorDesc[%c0, %c0] into %memDesc, pred = %pred {warp_used_hint = 105 : i32} : !tt.tensordesc<256x64xf16> -> !ttg.memdesc<256x64xf16, #shared_wb, #smem_wb, mutable>
315|     tt.return
316|   }
317| 
318|   // popcount must be a power of two.  0x07 has K=3 -- rejected even
319|   // though warps 0..2 are otherwise contiguous.
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.async_tdm_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.async_tdm_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 320-325
```mlir
320|   tt.func @warp_used_hint_non_pow2_k(
321|     %tensorDesc: !tt.tensordesc<256x64xf16>,
322|     %memDesc: !ttg.memdesc<256x64xf16, #shared_wb, #smem_wb, mutable>,
323|     %pred: i32
324|   ) {
325|     %c0 = arith.constant 0 : i32
```
**EN:** This function-oriented block defines or enters `warp_used_hint_non_pow2_k`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `warp_used_hint_non_pow2_k` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 326-332
```mlir
326|     // expected-error @+1 {{popcount(warp_used_hint) = 3 must be a power of two}}
327|     %0 = amdg.async_tdm_copy_global_to_local %tensorDesc[%c0, %c0] into %memDesc, pred = %pred {warp_used_hint = 7 : i32} : !tt.tensordesc<256x64xf16> -> !ttg.memdesc<256x64xf16, #shared_wb, #smem_wb, mutable>
328|     tt.return
329|   }
330| 
331|   // hint sets all 16 low bits but num_warps = 8 so bits 8..15 don't
332|   // correspond to any warp.  Reported by the bits-beyond check.
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.async_tdm_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.async_tdm_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 333-338
```mlir
333|   tt.func @warp_used_hint_exceeds_num_warps(
334|     %tensorDesc: !tt.tensordesc<256x64xf16>,
335|     %memDesc: !ttg.memdesc<256x64xf16, #shared_wb, #smem_wb, mutable>,
336|     %pred: i32
337|   ) {
338|     %c0 = arith.constant 0 : i32
```
**EN:** This function-oriented block defines or enters `warp_used_hint_exceeds_num_warps`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `warp_used_hint_exceeds_num_warps` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 339-345
```mlir
339|     // expected-error @+1 {{warp_used_hint = 0xffff sets bits beyond num_warps = 8}}
340|     %0 = amdg.async_tdm_copy_global_to_local %tensorDesc[%c0, %c0] into %memDesc, pred = %pred {warp_used_hint = 65535 : i32} : !tt.tensordesc<256x64xf16> -> !ttg.memdesc<256x64xf16, #shared_wb, #smem_wb, mutable>
341|     tt.return
342|   }
343| 
344|   // Bits outside [0, num_warps) must be zero.  K=2 is otherwise valid,
345|   // but warp index 9 is not in [0, 8).
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.async_tdm_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.async_tdm_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 346-351
```mlir
346|   tt.func @warp_used_hint_bits_beyond_num_warps(
347|     %tensorDesc: !tt.tensordesc<256x64xf16>,
348|     %memDesc: !ttg.memdesc<256x64xf16, #shared_wb, #smem_wb, mutable>,
349|     %pred: i32
350|   ) {
351|     %c0 = arith.constant 0 : i32
```
**EN:** This function-oriented block defines or enters `warp_used_hint_bits_beyond_num_warps`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `warp_used_hint_bits_beyond_num_warps` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 352-357
```mlir
352|     // expected-error @+1 {{sets bits beyond num_warps = 8}}
353|     %0 = amdg.async_tdm_copy_global_to_local %tensorDesc[%c0, %c0] into %memDesc, pred = %pred {warp_used_hint = 513 : i32} : !tt.tensordesc<256x64xf16> -> !ttg.memdesc<256x64xf16, #shared_wb, #smem_wb, mutable>
354|     tt.return
355|   }
356| }
357| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.async_tdm_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.async_tdm_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 358-358
```mlir
358| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 359-362
```mlir
359| 
360| #fp4_src = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 8], warpsPerCTA = [1, 1], order = [1, 0]}>
361| #fp4_dst = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [1, 1], order = [1, 0]}>
362| #fp4_scale_bad = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 16], warpsPerCTA = [1, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 363-363
```mlir
363| module attributes {"ttg.target" = "hip:gfx950", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 364-364
```mlir
364|   tt.func @scaled_upcast_fp4_incompatible_scale_encoding(%src: tensor<16x32xi8, #fp4_src>, %scale: tensor<16x64xbf16, #fp4_scale_bad>) {
```
**EN:** This function-oriented block defines or enters `scaled_upcast_fp4_incompatible_scale_encoding`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `scaled_upcast_fp4_incompatible_scale_encoding` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 365-370
```mlir
365|     // expected-error @+1 {{scale and output encodings are not compatible}}
366|     %0 = amdg.scaled_upcast_fp4 %src scale %scale {axis = 1 : i32} : tensor<16x32xi8, #fp4_src>, tensor<16x64xbf16, #fp4_scale_bad> -> tensor<16x64xbf16, #fp4_dst>
367|     tt.return
368|   }
369| }
370| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.scaled_upcast_fp4, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.scaled_upcast_fp4、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 371-371
```mlir
371| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 372-375
```mlir
372| 
373| // Partitioned encoding requires K to be a multiple of numLogicalPieces
374| // (= numPartitions*numGroups = 4) so the hinted copy fits in a single
375| // TDM instruction.  Here K=2 < numLogicalPieces=4 is rejected.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 376-378
```mlir
376| #shared_inner_mi = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
377| #partitioned_mi = #ttg.partitioned_shared<{numPartitions = 2, numGroups = 2, partitionDim = 0, partitionLayout = #shared_inner_mi}>
378| #smem_mi = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 379-379
```mlir
379| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 380-385
```mlir
380|   tt.func @warp_used_hint_partitioned_insufficient(
381|     %tensorDesc: !tt.tensordesc<128x16xf16>,
382|     %memDesc: !ttg.memdesc<128x16xf16, #partitioned_mi, #smem_mi, mutable>,
383|     %pred: i32
384|   ) {
385|     %c0 = arith.constant 0 : i32
```
**EN:** This function-oriented block defines or enters `warp_used_hint_partitioned_insufficient`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `warp_used_hint_partitioned_insufficient` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 386-391
```mlir
386|     // expected-error @+1 {{warp_used_hint with a partitioned shared encoding must select K active warps}}
387|     %0 = amdg.async_tdm_copy_global_to_local %tensorDesc[%c0, %c0] into %memDesc, pred = %pred {warp_used_hint = 3 : i32} : !tt.tensordesc<128x16xf16> -> !ttg.memdesc<128x16xf16, #partitioned_mi, #smem_mi, mutable>
388|     tt.return
389|   }
390| }
391| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.async_tdm_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.async_tdm_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 392-392
```mlir
392| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 393-396
```mlir
393| 
394| #fp4_src = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 8], warpsPerCTA = [1, 1], order = [1, 0]}>
395| #fp4_dst = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [1, 1], order = [1, 0]}>
396| #fp4_dst_bad = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 8], warpsPerCTA = [1, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 397-397
```mlir
397| module attributes {"ttg.target" = "hip:gfx950", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 398-398
```mlir
398|   tt.func @scaled_upcast_fp4_incompatible_src_encoding(%src: tensor<16x32xi8, #fp4_src>, %scale: tensor<16x64xbf16, #fp4_dst_bad>) {
```
**EN:** This function-oriented block defines or enters `scaled_upcast_fp4_incompatible_src_encoding`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `scaled_upcast_fp4_incompatible_src_encoding` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 399-404
```mlir
399|     // expected-error @+1 {{Src and Dst encodings are not compatible}}
400|     %0 = amdg.scaled_upcast_fp4 %src scale %scale {axis = 1 : i32} : tensor<16x32xi8, #fp4_src>, tensor<16x64xbf16, #fp4_dst_bad> -> tensor<16x64xbf16, #fp4_dst_bad>
401|     tt.return
402|   }
403| }
404| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.scaled_upcast_fp4, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.scaled_upcast_fp4、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 405-405
```mlir
405| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 406-408
```mlir
406| 
407| #fp4_src = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 8], warpsPerCTA = [1, 1], order = [1, 0]}>
408| #fp4_dst = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [1, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 409-409
```mlir
409| module attributes {"ttg.target" = "hip:gfx950", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 410-410
```mlir
410|   tt.func @scaled_upcast_fp4_invalid_result_type(%src: tensor<16x32xi8, #fp4_src>, %scale: tensor<16x64xbf16, #fp4_dst>) {
```
**EN:** This function-oriented block defines or enters `scaled_upcast_fp4_invalid_result_type`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `scaled_upcast_fp4_invalid_result_type` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 411-416
```mlir
411|     // expected-error @+1 {{must be ranked tensor of 16-bit float or bfloat16 type values}}
412|     %0 = amdg.scaled_upcast_fp4 %src scale %scale {axis = 1 : i32} : tensor<16x32xi8, #fp4_src>, tensor<16x64xbf16, #fp4_dst> -> tensor<16x64xf32, #fp4_dst>
413|     tt.return
414|   }
415| }
416| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.scaled_upcast_fp4, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.scaled_upcast_fp4、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 417-417
```mlir
417| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 418-419
```mlir
418| 
419| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [1, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 420-420
```mlir
420| module attributes {"ttg.target" = "hip:gfx950", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 421-421
```mlir
421|   tt.func @scaled_upcast_fp8_invalid_result_type(%src: tensor<16x64xf8E4M3FN, #blocked>, %scale: tensor<16x64xbf16, #blocked>) {
```
**EN:** This function-oriented block defines or enters `scaled_upcast_fp8_invalid_result_type`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `scaled_upcast_fp8_invalid_result_type` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 422-427
```mlir
422|     // expected-error @+1 {{must be ranked tensor of 16-bit float or bfloat16 type values}}
423|     %0 = amdg.scaled_upcast_fp8 %src scale %scale : tensor<16x64xf8E4M3FN, #blocked>, tensor<16x64xbf16, #blocked> -> tensor<16x64xf32, #blocked>
424|     tt.return
425|   }
426| }
427| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.scaled_upcast_fp8, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.scaled_upcast_fp8、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 428-428
```mlir
428| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 429-430
```mlir
429| 
430| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 431-431
```mlir
431| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 432-434
```mlir
432|   tt.func public @update_tensor_descriptor_wrong_offset_count(
433|     %desc: !tt.tensordesc<64x64xf16, #shared>, %dx: i32
434|   ) -> !tt.tensordesc<64x64xf16, #shared> {
```
**EN:** This function-oriented block defines or enters `update_tensor_descriptor_wrong_offset_count`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `update_tensor_descriptor_wrong_offset_count` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 435-440
```mlir
435|     // expected-error @+1 {{expected 2 add_offsets to match descriptor rank, got 1}}
436|     %result = amdg.update_tensor_descriptor %desc add_offsets = [%dx] : !tt.tensordesc<64x64xf16, #shared>
437|     tt.return %result : !tt.tensordesc<64x64xf16, #shared>
438|   }
439| }
440| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.update_tensor_descriptor, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.update_tensor_descriptor、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 441-441
```mlir
441| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 442-443
```mlir
442| 
443| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 444-444
```mlir
444| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 445-447
```mlir
445|   tt.func public @update_tensor_descriptor_wrong_bounds_count(
446|     %desc: !tt.tensordesc<64x64xf16, #shared>, %m: i32, %n: i32, %k: i32
447|   ) -> !tt.tensordesc<64x64xf16, #shared> {
```
**EN:** This function-oriented block defines or enters `update_tensor_descriptor_wrong_bounds_count`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `update_tensor_descriptor_wrong_bounds_count` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 448-453
```mlir
448|     // expected-error @+1 {{expected 2 set_bounds to match descriptor rank, got 3}}
449|     %result = amdg.update_tensor_descriptor %desc set_bounds = [%m, %n, %k] : !tt.tensordesc<64x64xf16, #shared>
450|     tt.return %result : !tt.tensordesc<64x64xf16, #shared>
451|   }
452| }
453| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.update_tensor_descriptor, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.update_tensor_descriptor、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 454-454
```mlir
454| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 455-456
```mlir
455| 
456| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 457-457
```mlir
457| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 458-460
```mlir
458|   tt.func public @update_tensor_descriptor_no_kwargs(
459|     %desc: !tt.tensordesc<64x64xf16, #shared>
460|   ) -> !tt.tensordesc<64x64xf16, #shared> {
```
**EN:** This function-oriented block defines or enters `update_tensor_descriptor_no_kwargs`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `update_tensor_descriptor_no_kwargs` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 461-465
```mlir
461|     // expected-error @+1 {{must provide at least one of add_offsets, set_bounds, dest, pred, or barrier}}
462|     %result = amdg.update_tensor_descriptor %desc : !tt.tensordesc<64x64xf16, #shared>
463|     tt.return %result : !tt.tensordesc<64x64xf16, #shared>
464|   }
465| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.update_tensor_descriptor, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.update_tensor_descriptor、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `--split-input-file`, `--verify-diagnostics`
- **CN:** 主要 pass 选项：`--split-input-file`，`--verify-diagnostics`
- **EN:** Dominant operations include `tt.func`, `tt.return`, `module`, `arith.constant`, `tt.dot`, `amdg.async_tdm_copy_global_to_local`, `amdg.in_thread_transpose`, `amdg.local_load_packed_tranposed`, `amdg.scaled_upcast_fp4`, `amdg.update_tensor_descriptor`.
- **CN:** 主要操作包括 `tt.func`、`tt.return`、`module`、`arith.constant`、`tt.dot`、`amdg.async_tdm_copy_global_to_local`、`amdg.in_thread_transpose`、`amdg.local_load_packed_tranposed`、`amdg.scaled_upcast_fp4`、`amdg.update_tensor_descriptor`。
- **EN:** The file contains 20 independently testable section(s). Check styles used: CHECK-LABEL x1. Important labels include ds_transpose_t_fp4_mfma16. Expected diagnostic comments specify the exact verifier/pass failures. It validates that invalid IR is rejected with the intended diagnostics.
- **CN:** 该文件包含 20 个可独立测试的分段。使用的检查类型：CHECK-LABEL ×1。 关键标签包括 ds_transpose_t_fp4_mfma16。 expected 诊断注释给出了 verifier/pass 需要触发的精确报错。 它验证无效 IR 会被拒绝，并产生预期诊断。

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