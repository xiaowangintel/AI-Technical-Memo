# amd-coalesce-async-copy.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/amd/amd-coalesce-async-copy.mlir`
- **EN:** Test for the MLIR/Triton pipeline `--tritonamdgpu-coalesce-async-copy=gfx-arch=gfx950` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `--tritonamdgpu-coalesce-async-copy=gfx-arch=gfx950` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file --tritonamdgpu-coalesce-async-copy=gfx-arch=gfx950 | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file --tritonamdgpu-coalesce-async-copy=gfx-arch=gfx950 | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt %s -split-input-file --tritonamdgpu-coalesce-async-copy=gfx-arch=gfx950 | FileCheck %s
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file --tritonamdgpu-coalesce-async-copy=gfx-arch=gfx950 | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file --tritonamdgpu-coalesce-async-copy=gfx-arch=gfx950 | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-5
```mlir
3| #blocked = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [64], warpsPerCTA = [4], order = [0]}>
4| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
5| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 6-7
```mlir
6| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.target" = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
7| // sizePerThread = [1] because we have no information about contiguity of src pointers
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 8-9
```mlir
8| // CHECK: #[[$NEW_BLOCKED:.*]] = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [64], warpsPerCTA = [4], order = [0]}>
9| // CHECK-LABEL: async_copy_1d
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: async_copy_1d anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: async_copy_1d 这样的标签用于锚定匹配范围。

### Lines 10-11
```mlir
10| tt.func @async_copy_1d(%input: tensor<1024x!tt.ptr<f32>, #blocked>,
11|     %view: !ttg.memdesc<1024xf32, #shared, #smem, mutable>) {
```
**EN:** This function-oriented block defines or enters `async_copy_1d`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_copy_1d` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 12-18
```mlir
12|   // CHECK: %{{.*}} = ttg.convert_layout %{{.*}} : {{.*}} -> tensor<1024x!tt.ptr<f32>, #[[$NEW_BLOCKED]]>
13|   // CHECK: %{{.*}} = ttg.async_copy_global_to_local %{{.*}}: tensor<1024x!tt.ptr<f32>, #[[$NEW_BLOCKED]]>
14|   %token = ttg.async_copy_global_to_local %input, %view: tensor<1024x!tt.ptr<f32>, #blocked> -> <1024xf32, #shared, #smem, mutable>
15|   tt.return
16| }
17| }
18| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.async_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.async_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 19-19
```mlir
19| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 20-23
```mlir
20| 
21| #blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [64], warpsPerCTA = [4], order = [0]}>
22| #shared = #ttg.padded_shared<[256:+4] {order = [0], shape = [1024]}>
23| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 24-25
```mlir
24| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.target" = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
25| // Padded encoding with an identity mapping does produce coalesced writes so we should not change the blocked encoding
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 26-27
```mlir
26| // CHECK: #[[$NEW_BLOCKED:.*]] = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [64], warpsPerCTA = [4], order = [0]}>
27| // CHECK-LABEL: async_copy_with_padding
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: async_copy_with_padding anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: async_copy_with_padding 这样的标签用于锚定匹配范围。

### Lines 28-29
```mlir
28| tt.func @async_copy_with_padding(%input: tensor<1024x!tt.ptr<f32>, #blocked>,
29|     %view: !ttg.memdesc<1024xf32, #shared, #smem, mutable>) {
```
**EN:** This function-oriented block defines or enters `async_copy_with_padding`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_copy_with_padding` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 30-36
```mlir
30|   // CHECK-NOT: ttg.convert_layout
31|   // CHECK: %{{.*}} = ttg.async_copy_global_to_local %{{.*}}: tensor<1024x!tt.ptr<f32>, #[[$NEW_BLOCKED]]>
32|   %token = ttg.async_copy_global_to_local %input, %view: tensor<1024x!tt.ptr<f32>, #blocked> -> <1024xf32, #shared, #smem, mutable>
33|   tt.return
34| }
35| }
36| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.async_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.async_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 37-37
```mlir
37| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 38-41
```mlir
38| 
39| #blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
40| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
41| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 42-43
```mlir
42| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.target" = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
43| // sizePerThread = [1, 1] because we have no information about contiguity of src pointers
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 44-45
```mlir
44| // CHECK: #[[$NEW_BLOCKED:.*]] = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 64], warpsPerCTA = [4, 1], order = [1, 0]}>
45| // CHECK-LABEL: async_copy_2d
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: async_copy_2d anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: async_copy_2d 这样的标签用于锚定匹配范围。

### Lines 46-47
```mlir
46| tt.func @async_copy_2d(%input: tensor<64x64x!tt.ptr<f32>, #blocked>,
47|     %view: !ttg.memdesc<64x64xf32, #shared, #smem, mutable>) {
```
**EN:** This function-oriented block defines or enters `async_copy_2d`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_copy_2d` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 48-54
```mlir
48|   // CHECK: %{{.*}} = ttg.convert_layout %{{.*}} : {{.*}} -> tensor<64x64x!tt.ptr<f32>, #[[$NEW_BLOCKED]]>
49|   // CHECK: %{{.*}} = ttg.async_copy_global_to_local %{{.*}}: tensor<64x64x!tt.ptr<f32>, #[[$NEW_BLOCKED]]>
50|   %token = ttg.async_copy_global_to_local %input, %view: tensor<64x64x!tt.ptr<f32>, #blocked> -> <64x64xf32, #shared, #smem, mutable>
51|   tt.return
52| }
53| }
54| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.async_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.async_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 55-55
```mlir
55| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 56-59
```mlir
56| 
57| #blocked = #ttg.blocked<{sizePerThread = [8, 1, 1], threadsPerWarp = [64, 1, 1], warpsPerCTA = [1,2,2], order = [0,1,2]}>
58| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0,1,2]}>
59| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 60-61
```mlir
60| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.target" = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
61| // sizePerThread = [1, 1, 1] because we have no information about contiguity of src pointers
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 62-63
```mlir
62| // CHECK: #[[$NEW_BLOCKED:.*]] = #ttg.blocked<{sizePerThread = [1, 1, 1], threadsPerWarp = [64, 1, 1], warpsPerCTA = [4, 1, 1], order = [0, 1, 2]}>
63| // CHECK-LABEL: async_copy_3d
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: async_copy_3d anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: async_copy_3d 这样的标签用于锚定匹配范围。

### Lines 64-65
```mlir
64| tt.func @async_copy_3d(%input: tensor<1024x1024x1024x!tt.ptr<f32>, #blocked>,
65|     %view: !ttg.memdesc<1024x1024x1024xf32, #shared, #smem, mutable>) {
```
**EN:** This function-oriented block defines or enters `async_copy_3d`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_copy_3d` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 66-72
```mlir
66|   // CHECK: %{{.*}} = ttg.convert_layout %{{.*}} : {{.*}} -> tensor<1024x1024x1024x!tt.ptr<f32>, #[[$NEW_BLOCKED]]>
67|   // CHECK: %{{.*}} = ttg.async_copy_global_to_local %{{.*}}: tensor<1024x1024x1024x!tt.ptr<f32>, #[[$NEW_BLOCKED]]>
68|   %token = ttg.async_copy_global_to_local %input, %view: tensor<1024x1024x1024x!tt.ptr<f32>, #blocked> -> <1024x1024x1024xf32, #shared, #smem, mutable>
69|   tt.return
70| }
71| }
72| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.async_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.async_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 73-73
```mlir
73| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 74-77
```mlir
74| 
75| #blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
76| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
77| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 78-78
```mlir
78| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.target" = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 79-80
```mlir
79| // CHECK: #[[$NEW_BLOCKED:.*]] = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 64], warpsPerCTA = [4, 1], order = [1, 0]}>
80| // CHECK-LABEL: async_copy_with_mask_and_other
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: async_copy_with_mask_and_other anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: async_copy_with_mask_and_other 这样的标签用于锚定匹配范围。

### Lines 81-84
```mlir
81| tt.func @async_copy_with_mask_and_other(%input: tensor<64x64x!tt.ptr<f32>, #blocked>,
82|     %view: !ttg.memdesc<64x64xf32, #shared, #smem, mutable>,
83|     %mask: tensor<64x64xi1, #blocked>,
84|     %other: tensor<64x64xf32, #blocked>) {
```
**EN:** This function-oriented block defines or enters `async_copy_with_mask_and_other`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_copy_with_mask_and_other` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 85-93
```mlir
85|   // CHECK: %{{.*}} = ttg.convert_layout %{{.*}} : {{.*}} -> tensor<64x64x!tt.ptr<f32>, #[[$NEW_BLOCKED]]>
86|   // CHECK: %{{.*}} = ttg.convert_layout %{{.*}} : {{.*}} -> tensor<64x64xi1, #[[$NEW_BLOCKED]]>
87|   // CHECK: %{{.*}} = ttg.convert_layout %{{.*}} : {{.*}} -> tensor<64x64xf32, #[[$NEW_BLOCKED]]>
88|   // CHECK: %{{.*}} = ttg.async_copy_global_to_local %{{.*}}: tensor<64x64x!tt.ptr<f32>, #[[$NEW_BLOCKED]]>
89|   %token = ttg.async_copy_global_to_local %input, %view mask %mask other %other: tensor<64x64x!tt.ptr<f32>, #blocked> -> <64x64xf32, #shared, #smem, mutable>
90|   tt.return
91| }
92| }
93| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.async_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.async_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 94-94
```mlir
94| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 95-98
```mlir
95| 
96| #blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [4, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
97| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
98| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 99-100
```mlir
 99| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
100|   // Clip to vector size 2 (32bit) because we do not support 64 bit loads to lds
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 101-102
```mlir
101|   // CHECK: #[[$NEW_BLOCKED:.*]] = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [2, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
102|   // CHECK-LABEL: async_copy_vector_size_2
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: async_copy_vector_size_2 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: async_copy_vector_size_2 这样的标签用于锚定匹配范围。

### Lines 103-112
```mlir
103|   tt.func public @async_copy_vector_size_2(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
104|                                 %arg1: i32 {tt.divisibility = 16 : i32},
105|                                 %arg2: !ttg.memdesc<32x64xf16, #shared, #smem, mutable>) {
106|     // We need the index calculation so AxisAnalysis sees that we can vectorize the load
107|     %1 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
108|     %2 = tt.expand_dims %1 {axis = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x64xi32, #blocked>
109|     %3 = tt.broadcast %2 : tensor<1x64xi32, #blocked> -> tensor<32x64xi32, #blocked>
110|     %4 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<32x64x!tt.ptr<f16>, #blocked>
111|     %5 = tt.addptr %4, %3 : tensor<32x64x!tt.ptr<f16>, #blocked>, tensor<32x64xi32, #blocked>
112| 
```
**EN:** This function-oriented block defines or enters `async_copy_vector_size_2`. Within it, the test exercises tt.func, lane/block index ranges, shape expansion, tensor broadcasting, broadcasted scalars or pointers, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_copy_vector_size_2` 为核心。测试在其中演示 tt.func、lane/block 索引范围、形状扩展、张量广播、广播后的标量或指针，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 113-119
```mlir
113|     // CHECK: %{{.*}} = ttg.convert_layout %{{.*}} : {{.*}} -> tensor<32x64x!tt.ptr<f16>, #[[$NEW_BLOCKED]]>
114|     // CHECK: %{{.*}} = ttg.async_copy_global_to_local %{{.*}}: tensor<32x64x!tt.ptr<f16>, #[[$NEW_BLOCKED]]>
115|     %6 = ttg.async_copy_global_to_local %5, %arg2 : tensor<32x64x!tt.ptr<f16>, #blocked> -> <32x64xf16, #shared, #smem, mutable>
116|     tt.return
117|   }
118| }
119| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.async_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.async_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 120-120
```mlir
120| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 121-124
```mlir
121| 
122| #blocked = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [32, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
123| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
124| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 125-126
```mlir
125| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
126|   // Clip to vector size 4 (128bit) which is the largest supported load width
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 127-128
```mlir
127|   // CHECK: #[[$NEW_BLOCKED:.*]] = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
128|   // CHECK-LABEL: async_copy_vector_size_8
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: async_copy_vector_size_8 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: async_copy_vector_size_8 这样的标签用于锚定匹配范围。

### Lines 129-138
```mlir
129|   tt.func public @async_copy_vector_size_8(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
130|                                 %arg1: i32 {tt.divisibility = 16 : i32},
131|                                 %arg2: !ttg.memdesc<32x64xf16, #shared, #smem, mutable>) {
132|     // We need the index calculation so AxisAnalysis sees that we can vectorize the load based on the src contiguity
133|     %1 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
134|     %2 = tt.expand_dims %1 {axis = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x64xi32, #blocked>
135|     %3 = tt.broadcast %2 : tensor<1x64xi32, #blocked> -> tensor<32x64xi32, #blocked>
136|     %4 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<32x64x!tt.ptr<f16>, #blocked>
137|     %5 = tt.addptr %4, %3 : tensor<32x64x!tt.ptr<f16>, #blocked>, tensor<32x64xi32, #blocked>
138| 
```
**EN:** This function-oriented block defines or enters `async_copy_vector_size_8`. Within it, the test exercises tt.func, lane/block index ranges, shape expansion, tensor broadcasting, broadcasted scalars or pointers, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_copy_vector_size_8` 为核心。测试在其中演示 tt.func、lane/block 索引范围、形状扩展、张量广播、广播后的标量或指针，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 139-145
```mlir
139|     // CHECK: %{{.*}} = ttg.convert_layout %{{.*}} : {{.*}} -> tensor<32x64x!tt.ptr<f16>, #[[$NEW_BLOCKED]]>
140|     // CHECK: %{{.*}} = ttg.async_copy_global_to_local %{{.*}}: tensor<32x64x!tt.ptr<f16>, #[[$NEW_BLOCKED]]>
141|     %6 = ttg.async_copy_global_to_local %5, %arg2 : tensor<32x64x!tt.ptr<f16>, #blocked> -> <32x64xf16, #shared, #smem, mutable>
142|     tt.return
143|   }
144| }
145| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.async_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.async_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 146-146
```mlir
146| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 147-150
```mlir
147| 
148| #blocked = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [32, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
149| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0, 1]}>
150| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 151-152
```mlir
151| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
152|   // The order of #blocked and #shared are different so we need to clip to 1 element
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 153-154
```mlir
153|   // CHECK: #[[$NEW_BLOCKED:.*]] = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 64], warpsPerCTA = [4, 1], order = [1, 0]}>
154|   // CHECK-LABEL: async_copy_different_order
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: async_copy_different_order anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: async_copy_different_order 这样的标签用于锚定匹配范围。

### Lines 155-164
```mlir
155|   tt.func public @async_copy_different_order(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
156|                                 %arg1: i32 {tt.divisibility = 16 : i32},
157|                                 %arg2: !ttg.memdesc<32x64xf32, #shared, #smem, mutable>) {
158|     // We need the index calculation so AxisAnalysis sees that we can vectorize the load based on the src contiguity
159|     %1 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
160|     %2 = tt.expand_dims %1 {axis = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x64xi32, #blocked>
161|     %3 = tt.broadcast %2 : tensor<1x64xi32, #blocked> -> tensor<32x64xi32, #blocked>
162|     %4 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<32x64x!tt.ptr<f32>, #blocked>
163|     %5 = tt.addptr %4, %3 : tensor<32x64x!tt.ptr<f32>, #blocked>, tensor<32x64xi32, #blocked>
164| 
```
**EN:** This function-oriented block defines or enters `async_copy_different_order`. Within it, the test exercises tt.func, lane/block index ranges, shape expansion, tensor broadcasting, broadcasted scalars or pointers, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_copy_different_order` 为核心。测试在其中演示 tt.func、lane/block 索引范围、形状扩展、张量广播、广播后的标量或指针，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 165-171
```mlir
165|     // CHECK: %{{.*}} = ttg.convert_layout %{{.*}} : {{.*}} -> tensor<32x64x!tt.ptr<f32>, #[[$NEW_BLOCKED]]>
166|     // CHECK: %{{.*}} = ttg.async_copy_global_to_local %{{.*}}: tensor<32x64x!tt.ptr<f32>, #[[$NEW_BLOCKED]]>
167|     %6 = ttg.async_copy_global_to_local %5, %arg2 : tensor<32x64x!tt.ptr<f32>, #blocked> -> <32x64xf32, #shared, #smem, mutable>
168|     tt.return
169|   }
170| }
171| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.async_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.async_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 172-172
```mlir
172| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 173-176
```mlir
173| 
174| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 64], warpsPerCTA = [2, 2], order = [1, 0]}>
175| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 2, maxPhase = 4, order = [1, 0]}>
176| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 177-178
```mlir
177| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.target" = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
178| // The shared layout should not be changed
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 179-181
```mlir
179| // CHECK: #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 2, maxPhase = 4, order = [1, 0]}>
180| // CHECK-NOT: #shared1
181| // CHECK-LABEL: async_copy_2d_swizzled
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: async_copy_2d_swizzled anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: async_copy_2d_swizzled 这样的标签用于锚定匹配范围。

### Lines 182-183
```mlir
182| tt.func @async_copy_2d_swizzled(%input: tensor<64x64x!tt.ptr<f16>, #blocked>,
183|     %view: !ttg.memdesc<64x64xf16, #shared, #smem, mutable>) {
```
**EN:** This function-oriented block defines or enters `async_copy_2d_swizzled`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_copy_2d_swizzled` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 184-189
```mlir
184|   // CHECK: %{{.*}} = ttg.async_copy_global_to_local {{.*}} -> <64x64xf16, #shared, #smem, mutable>
185|   %token = ttg.async_copy_global_to_local %input, %view: tensor<64x64x!tt.ptr<f16>, #blocked> -> <64x64xf16, #shared, #smem, mutable>
186|   tt.return
187| }
188| }
189| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.async_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.async_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 190-190
```mlir
190| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 191-194
```mlir
191| 
192| #blocked = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [64], warpsPerCTA = [1], order = [0]}>
193| #shared = #ttg.padded_shared<[64:+4] {order = [0], shape = [256]}>
194| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 195-196
```mlir
195| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.target" = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
196| // Padded encoding with an identity mapping has vec=1 whereas the blocked has vec=4 so we need to rewrite it
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 197-199
```mlir
197| // CHECK: #[[$NEW_SRC_ENCODING:.*]] = #ttg.linear
198| // CHECK-SAME{LITERAL}: register = [[64], [128]], lane = [[1], [2], [4], [8], [16], [32]], warp = [], block = []
199| // CHECK-LABEL: async_copy_with_padding_different_vec
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: async_copy_with_padding_different_vec anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: async_copy_with_padding_different_vec 这样的标签用于锚定匹配范围。

### Lines 200-201
```mlir
200| tt.func @async_copy_with_padding_different_vec(%input: tensor<256x!tt.ptr<f32>, #blocked>,
201|     %view: !ttg.memdesc<256xf32, #shared, #smem, mutable>) {
```
**EN:** This function-oriented block defines or enters `async_copy_with_padding_different_vec`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_copy_with_padding_different_vec` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 202-207
```mlir
202|   // CHECK: %{{.*}} = ttg.async_copy_global_to_local %{{.*}}: tensor<256x!tt.ptr<f32>, #[[$NEW_SRC_ENCODING]]>
203|   %token = ttg.async_copy_global_to_local %input, %view: tensor<256x!tt.ptr<f32>, #blocked> -> <256xf32, #shared, #smem, mutable>
204|   tt.return
205| }
206| }
207| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.async_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.async_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 208-208
```mlir
208| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 209-212
```mlir
209| 
210| #blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [64], warpsPerCTA = [4], order = [0]}>
211| #shared = #ttg.padded_shared<[64:+4] {offset = [[1], [2], [4], [8], [64], [128], [16], [32]], block = []}>
212| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 213-214
```mlir
213| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.target" = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
214| // We rearrange in 4 blocks of 16 elements, check that we transfer it to the src encoding to write coalesced to lds
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 215-217
```mlir
215| // CHECK: #[[$NEW_SRC_ENCODING:.*]] = #ttg.linear
216| // CHECK-SAME{LITERAL}: register = [], lane = [[1], [2], [4], [8], [64], [128]], warp = [[16], [32]], block = []
217| // CHECK-LABEL: async_copy_padded_layout_with_simple_rearanging
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: async_copy_padded_layout_with_simple_rearanging anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: async_copy_padded_layout_with_simple_rearanging 这样的标签用于锚定匹配范围。

### Lines 218-219
```mlir
218| tt.func @async_copy_padded_layout_with_simple_rearanging(%input: tensor<256x!tt.ptr<f32>, #blocked>,
219|     %view: !ttg.memdesc<256xf32, #shared, #smem, mutable>) {
```
**EN:** This function-oriented block defines or enters `async_copy_padded_layout_with_simple_rearanging`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_copy_padded_layout_with_simple_rearanging` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 220-225
```mlir
220|   // CHECK: %{{.*}} = ttg.async_copy_global_to_local %{{.*}}: tensor<256x!tt.ptr<f32>, #[[$NEW_SRC_ENCODING]]>
221|   %token = ttg.async_copy_global_to_local %input, %view: tensor<256x!tt.ptr<f32>, #blocked> -> <256xf32, #shared, #smem, mutable>
222|   tt.return
223| }
224| }
225| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.async_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.async_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 226-226
```mlir
226| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 227-230
```mlir
227| 
228| #blocked = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [64], warpsPerCTA = [4], order = [0]}>
229| #shared = #ttg.padded_shared<[256:+4] {offset = [[1], [2], [4], [8], [16], [32], [256], [512], [64], [128]], block = []}>
230| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 231-232
```mlir
231| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.target" = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
232| // We rearrange in 4 blocks of 16 elements, check that we transfer it to the src encoding to write coalesced to lds
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 233-235
```mlir
233| // CHECK: #[[$NEW_SRC_ENCODING:.*]] = #ttg.linear
234| // CHECK-SAME{LITERAL}: register = [[1], [2]], lane = [[4], [8], [16], [32], [256], [512]], warp = [[64], [128]], block = []
235| // CHECK-LABEL: async_copy_padded_layout_with_vectorization_and_rearanging
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: async_copy_padded_layout_with_vectorization_and_rearanging anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: async_copy_padded_layout_with_vectorization_and_rearanging 这样的标签用于锚定匹配范围。

### Lines 236-237
```mlir
236| tt.func @async_copy_padded_layout_with_vectorization_and_rearanging(%input: tensor<1024x!tt.ptr<f32>, #blocked> {tt.contiguity = 4 : i32, tt.divisibility = 16 : i32},
237|     %view: !ttg.memdesc<1024xf32, #shared, #smem, mutable>) {
```
**EN:** This function-oriented block defines or enters `async_copy_padded_layout_with_vectorization_and_rearanging`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_copy_padded_layout_with_vectorization_and_rearanging` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 238-243
```mlir
238|   // CHECK: %{{.*}} = ttg.async_copy_global_to_local %{{.*}}: tensor<1024x!tt.ptr<f32>, #[[$NEW_SRC_ENCODING]]>
239|   %token = ttg.async_copy_global_to_local %input, %view: tensor<1024x!tt.ptr<f32>, #blocked> -> <1024xf32, #shared, #smem, mutable>
240|   tt.return
241| }
242| }
243| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.async_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.async_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 244-244
```mlir
244| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 245-248
```mlir
245| 
246| #blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [64], warpsPerCTA = [4], order = [0]}>
247| #shared = #ttg.padded_shared<[64:+4] {offset = [[1], [2], [4], [8], [64], [16], [32]], block = []}>
248| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 249-250
```mlir
249| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.target" = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
250| // Check that we add a broadcast in case not each lane in the WG can read unique data
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 251-253
```mlir
251| // CHECK: #[[$NEW_SRC_ENCODING:.*]] = #ttg.linear
252| // CHECK-SAME{LITERAL}: register = [], lane = [[1], [2], [4], [8], [64], [16]], warp = [[32], [0]], block = []
253| // CHECK-LABEL: async_copy_padded_layout_requiring_broadcasting
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: async_copy_padded_layout_requiring_broadcasting anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: async_copy_padded_layout_requiring_broadcasting 这样的标签用于锚定匹配范围。

### Lines 254-255
```mlir
254| tt.func @async_copy_padded_layout_requiring_broadcasting(%input: tensor<128x!tt.ptr<f32>, #blocked>,
255|     %view: !ttg.memdesc<128xf32, #shared, #smem, mutable>) {
```
**EN:** This function-oriented block defines or enters `async_copy_padded_layout_requiring_broadcasting`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_copy_padded_layout_requiring_broadcasting` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 256-261
```mlir
256|   // CHECK: %{{.*}} = ttg.async_copy_global_to_local %{{.*}}: tensor<128x!tt.ptr<f32>, #[[$NEW_SRC_ENCODING]]>
257|   %token = ttg.async_copy_global_to_local %input, %view: tensor<128x!tt.ptr<f32>, #blocked> -> <128xf32, #shared, #smem, mutable>
258|   tt.return
259| }
260| }
261| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.async_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.async_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 262-262
```mlir
262| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 263-266
```mlir
263| 
264| #blocked = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [64], warpsPerCTA = [1], order = [0]}>
265| #shared = #ttg.padded_shared<[16:+4] {order = [0], shape = [256]}>
266| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 267-268
```mlir
267| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.target" = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
268| // Padded encoding with a small padding interval cannot write warp coalesced so we should not change the encoding
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 269-270
```mlir
269| // CHECK: #[[$NEW_BLOCKED:.*]] = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [64], warpsPerCTA = [1], order = [0]}>
270| // CHECK-LABEL: async_copy_with_padding_different_vec
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: async_copy_with_padding_different_vec anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: async_copy_with_padding_different_vec 这样的标签用于锚定匹配范围。

### Lines 271-272
```mlir
271| tt.func @async_copy_with_padding_different_vec(%input: tensor<256x!tt.ptr<f32>, #blocked>,
272|     %view: !ttg.memdesc<256xf32, #shared, #smem, mutable>) {
```
**EN:** This function-oriented block defines or enters `async_copy_with_padding_different_vec`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_copy_with_padding_different_vec` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 273-277
```mlir
273|   // CHECK: %{{.*}} = ttg.async_copy_global_to_local %{{.*}}: tensor<256x!tt.ptr<f32>, #[[$NEW_BLOCKED]]>
274|   %token = ttg.async_copy_global_to_local %input, %view: tensor<256x!tt.ptr<f32>, #blocked> -> <256xf32, #shared, #smem, mutable>
275|   tt.return
276| }
277| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.async_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.async_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `--tritonamdgpu-coalesce-async-copy=gfx-arch=gfx950`
- **CN:** 主要 pass 选项：`-split-input-file`，`--tritonamdgpu-coalesce-async-copy=gfx-arch=gfx950`
- **EN:** Dominant operations include `tt.func`, `module`, `ttg.async_copy_global_to_local`, `tt.return`, `tt.make_range`, `tt.expand_dims`, `tt.broadcast`, `tt.splat`, `tt.addptr`.
- **CN:** 主要操作包括 `tt.func`、`module`、`ttg.async_copy_global_to_local`、`tt.return`、`tt.make_range`、`tt.expand_dims`、`tt.broadcast`、`tt.splat`、`tt.addptr`。
- **EN:** The file contains 13 independently testable section(s). Check styles used: CHECK x37, CHECK-LABEL x14, CHECK-SAME x4, CHECK-NOT x2. Important labels include async_copy_1d, async_copy_with_padding, async_copy_2d, async_copy_3d. It validates both the presence of the desired rewrite and the absence of unwanted residual operations.
- **CN:** 该文件包含 13 个可独立测试的分段。使用的检查类型：CHECK ×37，CHECK-LABEL ×14，CHECK-SAME ×4，CHECK-NOT ×2。 关键标签包括 async_copy_1d，async_copy_with_padding，async_copy_2d，async_copy_3d。 它同时验证目标重写已经出现，并且不再残留不需要的操作。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `tt`: Triton high-level tensor/pointer operations.
- **CN:** `tt`：Triton 高层张量/指针操作。
- **EN:** `ttg`: TritonGPU layout, memory, and GPU-specific operations.
- **CN:** `ttg`：TritonGPU 布局、内存和 GPU 专用操作。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。