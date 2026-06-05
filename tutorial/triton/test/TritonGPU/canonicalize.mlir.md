# canonicalize.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/canonicalize.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-canonicalize` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-canonicalize` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file -canonicalize -allow-unregistered-dialect | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file -canonicalize -allow-unregistered-dialect | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-3
```mlir
1| // RUN: triton-opt %s -split-input-file -canonicalize -allow-unregistered-dialect | FileCheck %s
2| 
3| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file -canonicalize -allow-unregistered-dialect | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file -canonicalize -allow-unregistered-dialect | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 4-8
```mlir
4| // CHECK-LABEL: @test_canonicalize_convert_view
5| // CHECK-SAME: (%[[ARG:.+]]: tensor<64x64xf32
6| //   CHECK-NOT:   ttg.convert_layout
7| //       CHECK:   %[[V:.+]] = tt.reshape %[[ARG]] allow_reorder
8| //       CHECK:   tt.return %[[V]]
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 9-12
```mlir
 9| #blocked0 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [8, 1], order = [1, 0]}>
10| #blocked1 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [8], order = [0]}>
11| #blocked2 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [4, 8], warpsPerCTA = [8, 1], order = [0, 1]}>
12| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 13-13
```mlir
13| module attributes {"ttg.num-warps" = 8 : i32, "ttg.num-ctas" = 1 : i32, "ttg.target" = "cuda:80"} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 14-20
```mlir
14| tt.func @test_canonicalize_convert_view(%arg0: tensor<64x64xf32, #blocked0>) -> tensor<4096xf32, #blocked1> {
15|     %c = ttg.convert_layout %arg0 : tensor<64x64xf32, #blocked0> -> tensor<64x64xf32, #blocked2>
16|     %r = tt.reshape %c allow_reorder : tensor<64x64xf32, #blocked2> -> tensor<4096xf32, #blocked1>
17|     tt.return %r : tensor<4096xf32, #blocked1>
18| }
19| }  // end module
20| 
```
**EN:** This function-oriented block defines or enters `test_canonicalize_convert_view`. Within it, the test exercises tt.func, layout conversions, tensor reshaping, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `test_canonicalize_convert_view` 为核心。测试在其中演示 tt.func、布局转换、张量重塑、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 21-21
```mlir
21| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 22-24
```mlir
22| 
23| // test that the convert doesn't get combined with view if the resulting operations
24| // is an expensive view which would require moving data across threads.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 25-29
```mlir
25| // CHECK-LABEL: @test_canonicalize_convert_expensive_view
26| // CHECK-SAME: (%[[ARG:.+]]: tensor<256x16xf32
27| //       CHECK:   %[[C:.+]] = ttg.convert_layout %[[ARG]]
28| //       CHECK:   %[[V:.+]] = tt.reshape %[[C]] allow_reorder
29| //       CHECK:   tt.return %[[V]]
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 30-32
```mlir
30| #blocked0 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [8, 1], order = [1, 0]}>
31| #blocked1 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [8], order = [0]}>
32| #blocked2 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [8, 1], order = [0, 1]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 33-33
```mlir
33| module attributes {"ttg.num-warps" = 8 : i32, "ttg.num-ctas" = 1 : i32, "ttg.target" = "cuda:80"} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 34-40
```mlir
34| tt.func @test_canonicalize_convert_expensive_view(%arg0: tensor<256x16xf32, #blocked0>) -> tensor<4096xf32, #blocked1> {
35|     %c = ttg.convert_layout %arg0 : tensor<256x16xf32, #blocked0> -> tensor<256x16xf32, #blocked2>
36|     %r = tt.reshape %c allow_reorder : tensor<256x16xf32, #blocked2> -> tensor<4096xf32, #blocked1>
37|     tt.return %r : tensor<4096xf32, #blocked1>
38| }
39| }  // end module
40| 
```
**EN:** This function-oriented block defines or enters `test_canonicalize_convert_expensive_view`. Within it, the test exercises tt.func, layout conversions, tensor reshaping, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `test_canonicalize_convert_expensive_view` 为核心。测试在其中演示 tt.func、布局转换、张量重塑、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 41-41
```mlir
41| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 42-44
```mlir
42| 
43| // test that the convert doesn't get combined with view if the resulting operations
44| // is an expensive view which would require moving data across threads.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 45-49
```mlir
45| // CHECK-LABEL: @test_canonicalize_convert_expensive_view
46| // CHECK-SAME: (%[[ARG:.+]]: tensor<2xf32
47| //       CHECK:   %[[C:.+]] = ttg.convert_layout %[[ARG]]
48| //       CHECK:   %[[V:.+]] = tt.reshape %[[C]] allow_reorder
49| //       CHECK:   tt.return %[[V]]
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 50-51
```mlir
50| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
51| #blocked1 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 52-52
```mlir
52| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:80"} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:80`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:80`。

### Lines 53-59
```mlir
53|   tt.func @test_canonicalize_convert_expensive_view2(%arg0: tensor<2xf32, #ttg.slice<{dim = 1, parent = #blocked}>>) -> tensor<2xf32, #blocked1> {
54|     %c = ttg.convert_layout %arg0 : tensor<2xf32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<2xf32, #blocked1>
55|     %r = tt.reshape %c allow_reorder : tensor<2xf32, #blocked1> -> tensor<2xf32, #blocked1>
56|     tt.return %r : tensor<2xf32, #blocked1>
57|   }
58| }
59| 
```
**EN:** This function-oriented block defines or enters `test_canonicalize_convert_expensive_view2`. Within it, the test exercises tt.func, layout conversions, tensor reshaping, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `test_canonicalize_convert_expensive_view2` 为核心。测试在其中演示 tt.func、布局转换、张量重塑、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 60-60
```mlir
60| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 61-63
```mlir
61| 
62| // test that the convert does get combined with the view even if the resulting operation
63| // is an efficient view.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 64-68
```mlir
64| // CHECK-LABEL: @test_canonicalize_convert_view
65| // CHECK-SAME: (%[[ARG:.+]]: tensor<64x64xf32
66| //   CHECK-NOT:   ttg.convert_layout
67| //       CHECK:   %[[V:.+]] = tt.reshape %[[ARG]] allow_reorder
68| //       CHECK:   tt.return %[[V]]
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 69-72
```mlir
69| #blocked0 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [8, 1], order = [1, 0]}>
70| #blocked1 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [8], order = [0]}>
71| #blocked2 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [4, 8], warpsPerCTA = [8, 1], order = [0, 1]}>
72| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 73-73
```mlir
73| module attributes {"ttg.num-warps" = 8 : i32, "ttg.num-ctas" = 1 : i32, "ttg.target" = "cuda:80"} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 74-80
```mlir
74| tt.func @test_canonicalize_convert_view(%arg0: tensor<64x64xf32, #blocked0>) -> tensor<4096xf32, #blocked1> {
75|     %c = ttg.convert_layout %arg0 : tensor<64x64xf32, #blocked0> -> tensor<64x64xf32, #blocked2>
76|     %r = tt.reshape %c allow_reorder efficient_layout : tensor<64x64xf32, #blocked2> -> tensor<4096xf32, #blocked1>
77|     tt.return %r : tensor<4096xf32, #blocked1>
78| }
79| }  // end module
80| 
```
**EN:** This function-oriented block defines or enters `test_canonicalize_convert_view`. Within it, the test exercises tt.func, layout conversions, tensor reshaping, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `test_canonicalize_convert_view` 为核心。测试在其中演示 tt.func、布局转换、张量重塑、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 81-81
```mlir
81| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 82-89
```mlir
82| 
83| // CHECK-LABEL: @test_canonicalize_convert_histogram
84| // CHECK-SAME: (%[[SRC:.+]]: tensor<256xi32
85| // CHECK-SAME: %[[MASK:.+]]: tensor<256xi1
86| //       CHECK:   %[[M:.+]] = ttg.convert_layout %[[MASK]]
87| //       CHECK:   %[[V:.+]] = tt.histogram %[[SRC]], %[[M]]
88| //   CHECK-NOT:   ttg.convert_layout
89| //       CHECK:   tt.return %[[V]]
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 90-92
```mlir
90| #blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
91| #blocked1 = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
92| #blocked2 = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 93-93
```mlir
93| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32, "ttg.target" = "cuda:80"} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 94-102
```mlir
 94| tt.func @test_canonicalize_convert_histogram(%arg0: tensor<256xi32, #blocked1>, %arg1: tensor<256xi1, #blocked2>) -> tensor<512xi32, #blocked2> {
 95|     %0 = ttg.convert_layout %arg0 : tensor<256xi32, #blocked1> -> tensor<256xi32, #blocked>
 96|     %1 = ttg.convert_layout %arg1 : tensor<256xi1, #blocked2> -> tensor<256xi1, #blocked>
 97|     %2 = tt.histogram %0, %1 : tensor<256xi32, #blocked> -> tensor<512xi32, #blocked>
 98|     %3 = ttg.convert_layout %2 : tensor<512xi32, #blocked> -> tensor<512xi32, #blocked2>
 99|     tt.return %3 : tensor<512xi32, #blocked2>
100| }
101| }  // end module
102| 
```
**EN:** This function-oriented block defines or enters `test_canonicalize_convert_histogram`. Within it, the test exercises layout conversions, tt.func, tt.histogram, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `test_canonicalize_convert_histogram` 为核心。测试在其中演示 布局转换、tt.func、tt.histogram、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 103-103
```mlir
103| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 104-110
```mlir
104| 
105| // CHECK-LABEL: @test_canonicalize_convert_local_load
106| // CHECK-NOT:   ttg.barrier local
107| // CHECK: %[[V:.+]] = ttg.local_load {{.*}} token %arg0
108| // CHECK-NEXT:  ttg.barrier local
109| // CHECK-NEXT: tt.return %[[V]]
110| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @test_canonicalize_convert_local_load anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @test_canonicalize_convert_local_load 这样的标签用于锚定匹配范围。

### Lines 111-114
```mlir
111| #blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
112| #blocked1 = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
113| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
114| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 115-115
```mlir
115| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32, "ttg.compute-capability" = 80} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 116-124
```mlir
116| tt.func @test_canonicalize_convert_local_load(%arg0: !ttg.async.token) -> tensor<256xi32, #blocked1> {
117|     %0 = ttg.local_alloc : () -> !ttg.memdesc<256xi32, #shared, #smem, mutable>
118|     %1 = ttg.local_load %0 token %arg0: !ttg.memdesc<256xi32, #shared, #smem, mutable> -> tensor<256xi32, #blocked>
119|     ttg.barrier local
120|     %2 = ttg.convert_layout %1 : tensor<256xi32, #blocked> -> tensor<256xi32, #blocked1>
121|     tt.return %2 : tensor<256xi32, #blocked1>
122| }
123| }  // end module
124| 
```
**EN:** This function-oriented block defines or enters `test_canonicalize_convert_local_load`. Within it, the test exercises tt.func, shared/local memory allocation, local/shared memory loads, ttg.barrier, layout conversions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `test_canonicalize_convert_local_load` 为核心。测试在其中演示 tt.func、共享/本地内存分配、本地/共享内存加载、ttg.barrier、布局转换，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 125-125
```mlir
125| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 126-129
```mlir
126| 
127| #blocked = #ttg.blocked<{sizePerThread = [1, 32], threadsPerWarp = [32, 1], warpsPerCTA = [4, 2], order = [0, 1]}>
128| #linear = #ttg.linear<{register = [[0, 1], [0, 2], [0, 4], [0, 8], [0, 16]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0]], warp = [[32, 0], [64, 0], [0, 32]], block = []}>
129| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 64, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 130-130
```mlir
130| // CHECK-LABEL: test_canonicalize_convert_tmem_store
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: test_canonicalize_convert_tmem_store anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: test_canonicalize_convert_tmem_store 这样的标签用于锚定匹配范围。

### Lines 131-131
```mlir
131| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 132-136
```mlir
132|   tt.func @test_canonicalize_convert_tmem_store(
133|     %arg0: tensor<128x64xbf16, #linear>,
134|     %arg1: !ttg.memdesc<128x64xbf16, #tmem, #ttng.tensor_memory, mutable>
135|   ) {
136|       %true = arith.constant true
```
**EN:** This function-oriented block defines or enters `test_canonicalize_convert_tmem_store`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `test_canonicalize_convert_tmem_store` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 137-138
```mlir
137|       // CHECK-NOT: ttg.convert_layout
138|       %1 = ttg.convert_layout %arg0 : tensor<128x64xbf16, #linear> -> tensor<128x64xbf16, #blocked>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 139-144
```mlir
139|       // CHECK: ttng.tmem_store %{{.*}} : tensor<128x64xbf16, #linear> ->
140|       ttng.tmem_store %1, %arg1, %true : tensor<128x64xbf16, #blocked> -> !ttg.memdesc<128x64xbf16, #tmem, #ttng.tensor_memory, mutable>
141|       tt.return
142|   }
143| }
144| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_store, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_store、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 145-145
```mlir
145| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 146-149
```mlir
146| 
147| #blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 4], warpsPerCTA = [1, 1], order = [1, 0]}>
148| #shared = #ttg.swizzled_shared<{vec = 1, perPhase=2, maxPhase=8, order = [1, 0]}>
149| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 150-150
```mlir
150| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 151-151
```mlir
151|   // CHECK-LABEL: local_alloc_nofold1
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: local_alloc_nofold1 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: local_alloc_nofold1 这样的标签用于锚定匹配范围。

### Lines 152-152
```mlir
152|   tt.func @local_alloc_nofold1(%arg0: tensor<16x16xf16, #blocked>) -> !ttg.memdesc<16x16xf16, #shared, #smem> {
```
**EN:** This function-oriented block defines or enters `local_alloc_nofold1`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `local_alloc_nofold1` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 153-164
```mlir
153|     // CHECK: %[[ARG:.+]] = ttg.local_alloc
154|     // CHECK-NEXT: %[[ARG2:.+]] = ttg.local_load %[[ARG]]
155|     // CHECK-NEXT: %[[ARG3:.+]] = ttg.local_alloc %[[ARG2]]
156|     // CHECK-NEXT: tt.return %[[ARG3]]
157|     %0 = ttg.local_alloc %arg0 : (tensor<16x16xf16, #blocked>) -> !ttg.memdesc<16x16xf16, #shared, #smem, mutable>
158|     %1 = ttg.local_load %0 : !ttg.memdesc<16x16xf16, #shared, #smem, mutable> -> tensor<16x16xf16, #blocked>
159|     %2 = ttg.local_alloc %1 : (tensor<16x16xf16, #blocked>) -> !ttg.memdesc<16x16xf16, #shared, #smem>
160|     tt.return %2 : !ttg.memdesc<16x16xf16, #shared, #smem>
161|   }
162| }  // end module
163| 
164| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, local/shared memory loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、本地/共享内存加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 165-165
```mlir
165| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 166-170
```mlir
166| 
167| #blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 4], warpsPerCTA = [1, 1], order = [1, 0]}>
168| #shared = #ttg.swizzled_shared<{vec = 1, perPhase=2, maxPhase=8, order = [1, 0]}>
169| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase=1, maxPhase=1, order = [1, 0]}>
170| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 171-171
```mlir
171| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 172-172
```mlir
172|   // CHECK-LABEL: local_alloc_nofold2
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: local_alloc_nofold2 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: local_alloc_nofold2 这样的标签用于锚定匹配范围。

### Lines 173-173
```mlir
173|   tt.func @local_alloc_nofold2(%arg0: tensor<16x16xf16, #blocked>) -> !ttg.memdesc<16x16xf16, #shared1, #smem> {
```
**EN:** This function-oriented block defines or enters `local_alloc_nofold2`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `local_alloc_nofold2` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 174-185
```mlir
174|     // CHECK: %[[ARG:.+]] = ttg.local_alloc
175|     // CHECK-NEXT: %[[ARG2:.+]] = ttg.local_load %[[ARG]]
176|     // CHECK-NEXT: %[[ARG3:.+]] = ttg.local_alloc %[[ARG2]]
177|     // CHECK-NEXT: tt.return %[[ARG3]]
178|     %0 = ttg.local_alloc %arg0 : (tensor<16x16xf16, #blocked>) -> !ttg.memdesc<16x16xf16, #shared, #smem>
179|     %1 = ttg.local_load %0 : !ttg.memdesc<16x16xf16, #shared, #smem> -> tensor<16x16xf16, #blocked>
180|     %2 = ttg.local_alloc %1 : (tensor<16x16xf16, #blocked>) -> !ttg.memdesc<16x16xf16, #shared1, #smem>
181|     tt.return %2 : !ttg.memdesc<16x16xf16, #shared1, #smem>
182|   }
183| }  // end module
184| 
185| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, local/shared memory loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、本地/共享内存加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 186-186
```mlir
186| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 187-190
```mlir
187| 
188| #blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 4], warpsPerCTA = [1, 1], order = [1, 0]}>
189| #shared = #ttg.swizzled_shared<{vec = 1, perPhase=2, maxPhase=8, order = [1, 0]}>
190| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 191-191
```mlir
191| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 192-192
```mlir
192|   tt.func @local_alloc_fold(%arg0: tensor<16x16xf16, #blocked>) -> !ttg.memdesc<16x16xf16, #shared, #smem> {
```
**EN:** This function-oriented block defines or enters `local_alloc_fold`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `local_alloc_fold` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 193-202
```mlir
193|     // CHECK-LABEL: local_alloc_fold
194|     // CHECK-NEXT: %[[ARG:.+]] = ttg.local_alloc
195|     // CHECK-NEXT: tt.return %[[ARG]]
196|     %0 = ttg.local_alloc %arg0 : (tensor<16x16xf16, #blocked>) -> !ttg.memdesc<16x16xf16, #shared, #smem>
197|     %1 = ttg.local_load %0 : !ttg.memdesc<16x16xf16, #shared, #smem> -> tensor<16x16xf16, #blocked>
198|     %2 = ttg.local_alloc %1 : (tensor<16x16xf16, #blocked>) -> !ttg.memdesc<16x16xf16, #shared, #smem>
199|     tt.return %2 : !ttg.memdesc<16x16xf16, #shared, #smem>
200|   }
201| }  // end module
202| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, local/shared memory loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、本地/共享内存加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 203-203
```mlir
203| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 204-206
```mlir
204| 
205| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [8, 1], order = [1, 0]}>
206| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [8, 1], order = [0, 1]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 207-207
```mlir
207| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 208-208
```mlir
208|   // CHECK-LABEL: convert_layout_gather_src
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: convert_layout_gather_src anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: convert_layout_gather_src 这样的标签用于锚定匹配范围。

### Lines 209-210
```mlir
209|   tt.func @convert_layout_gather_src(%arg0: tensor<16x16xf16, #blocked>, %arg1: tensor<16x16xi32, #blocked>) -> tensor<16x16xf16, #blocked> {
210|     %0 = ttg.convert_layout %arg0 : tensor<16x16xf16, #blocked> -> tensor<16x16xf16, #blocked1>
```
**EN:** This function-oriented block defines or enters `convert_layout_gather_src`. Within it, the test exercises tt.func, layout conversions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `convert_layout_gather_src` 为核心。测试在其中演示 tt.func、布局转换，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 211-215
```mlir
211|     // CHECK-NEXT: tt.gather %arg0[%arg1]
212|     %1 = tt.gather %0[%arg1] {axis = 0 : i32} : (tensor<16x16xf16, #blocked1>, tensor<16x16xi32, #blocked>) -> tensor<16x16xf16, #blocked>
213|     tt.return %1 : tensor<16x16xf16, #blocked>
214|   }
215| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.gather, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.gather、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 216-216
```mlir
216|   // CHECK-LABEL: gather_efficient_layout
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: gather_efficient_layout anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: gather_efficient_layout 这样的标签用于锚定匹配范围。

### Lines 217-217
```mlir
217|   tt.func @gather_efficient_layout(%arg0: tensor<16x16xf16, #blocked>, %arg1: tensor<16x16xi32, #blocked>) -> tensor<16x16xf16, #blocked> {
```
**EN:** This function-oriented block defines or enters `gather_efficient_layout`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `gather_efficient_layout` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 218-219
```mlir
218|     // CHECK-NEXT: convert_layout
219|     %0 = ttg.convert_layout %arg0 : tensor<16x16xf16, #blocked> -> tensor<16x16xf16, #blocked1>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 220-225
```mlir
220|     // CHECK-NEXT: tt.gather {{.*}} (tensor<16x16xf16, #blocked1>
221|     %1 = tt.gather %0[%arg1] {axis = 0 : i32, efficient_layout} : (tensor<16x16xf16, #blocked1>, tensor<16x16xi32, #blocked>) -> tensor<16x16xf16, #blocked>
222|     tt.return %1 : tensor<16x16xf16, #blocked>
223|   }
224| }
225| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.gather, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.gather、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 226-226
```mlir
226| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 227-231
```mlir
227| 
228| #linear = #ttg.linear<{register = [[0, 1], [8, 0], [16, 0]], lane = [[0, 2], [0, 4], [1, 0], [2, 0], [4, 0]], warp = [[0, 8], [0, 16]], block = []}>
229| #blocked = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [8, 4], warpsPerCTA = [1, 4], order = [1, 0]}>
230| #blocked_trans = #ttg.blocked<{sizePerThread = [2, 1], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [0, 1]}>
231| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 232-233
```mlir
232| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
233| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 234-234
```mlir
234| // CHECK-LABEL: @infer_trans
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @infer_trans anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @infer_trans 这样的标签用于锚定匹配范围。

### Lines 235-235
```mlir
235| tt.func @infer_trans(%arg0: tensor<32x32xf32, #linear>) -> tensor<32x32xf32, #blocked_trans> {
```
**EN:** This function-oriented block defines or enters `infer_trans`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `infer_trans` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 236-243
```mlir
236|   // CHECK-NOT: ttg.convert_layout
237|   %0 = ttg.convert_layout %arg0 : tensor<32x32xf32, #linear> -> tensor<32x32xf32, #blocked>
238|   %1 = tt.trans %0  {order = array<i32: 1, 0>} : tensor<32x32xf32, #blocked> -> tensor<32x32xf32, #blocked_trans>
239|   tt.return %1 : tensor<32x32xf32, #blocked_trans>
240| }
241| 
242| }
243| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, transpose-like layout changes, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、转置类布局变换、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 244-244
```mlir
244| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 245-249
```mlir
245| 
246| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 16, 16]}>
247| #dot_t = #ttg.linear<{register = [[1, 0], [0, 8], [8, 0], [16, 0], [32, 0], [64, 0], [128, 0], [0, 64], [0, 128]], lane = [[2, 0], [4, 0], [0, 1], [0, 2], [0, 4]], warp = [[0, 16], [0, 32]], block = []}>
248| #dot_linear = #ttg.linear<{register = [[0, 1], [8, 0], [0, 8], [0, 16], [0, 32], [0, 64], [0, 128], [64, 0], [128, 0]], lane = [[0, 2], [0, 4], [1, 0], [2, 0], [4, 0]], warp = [[16, 0], [32, 0]], block = []}>
249| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 250-250
```mlir
250| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 251-251
```mlir
251|   // CHECK-LABEL: @simplify_trans_trans
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @simplify_trans_trans anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @simplify_trans_trans 这样的标签用于锚定匹配范围。

### Lines 252-252
```mlir
252|   tt.func public @simplify_trans_trans(%arg0: tensor<256x256xf32, #dot_linear>) -> tensor<256x256xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> {
```
**EN:** This function-oriented block defines or enters `simplify_trans_trans`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `simplify_trans_trans` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 253-259
```mlir
253|     // CHECK-NEXT: ttg.convert_layout
254|     %a = tt.trans %arg0 {order=array<i32: 1,0>} : tensor<256x256xf32, #dot_linear> -> tensor<256x256xf32, #dot_t>
255|     %b = tt.trans %a {order=array<i32: 1,0>} : tensor<256x256xf32, #dot_t> -> tensor<256x256xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
256|     tt.return %b : tensor<256x256xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
257|   }
258| }
259| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on transpose-like layout changes, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 转置类布局变换、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 260-260
```mlir
260| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 261-262
```mlir
261| 
262| // CHECK-LABEL: @warp_specialize_with_no_uses_and_effects
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @warp_specialize_with_no_uses_and_effects anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @warp_specialize_with_no_uses_and_effects 这样的标签用于锚定匹配范围。

### Lines 263-272
```mlir
263| tt.func @warp_specialize_with_no_uses_and_effects(%arg0: i32) {
264|   %0 = ttg.warp_specialize(%arg0)
265|   default {
266|     %1 = arith.addi %arg0, %arg0 : i32
267|     ttg.warp_yield %1 : i32
268|   }
269|   partition0(%arg1: i32) num_warps(4) {
270|     arith.addi %arg1, %arg1 : i32
271|     ttg.warp_return
272|   } : (i32) -> i32
```
**EN:** This function-oriented block defines or enters `warp_specialize_with_no_uses_and_effects`. Within it, the test exercises tt.func, integer additions, ttg.warp_specialize, ttg.warp_yield, ttg.warp_return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `warp_specialize_with_no_uses_and_effects` 为核心。测试在其中演示 tt.func、整数加法、ttg.warp_specialize、ttg.warp_yield、ttg.warp_return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 273-276
```mlir
273|   // CHECK-NEXT: tt.return
274|   tt.return
275| }
276| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 277-277
```mlir
277| // CHECK-LABEL: @canonicalize_within_warp_specialize
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @canonicalize_within_warp_specialize anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @canonicalize_within_warp_specialize 这样的标签用于锚定匹配范围。

### Lines 278-282
```mlir
278| tt.func @canonicalize_within_warp_specialize(%arg0: i32) -> i32 {
279|   %c0_i32 = arith.constant 0 : i32
280|   %0 = ttg.warp_specialize()
281|   default {
282|     %1 = arith.addi %arg0, %c0_i32 : i32
```
**EN:** This function-oriented block defines or enters `canonicalize_within_warp_specialize`. Within it, the test exercises tt.func, constants, ttg.warp_specialize, integer additions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `canonicalize_within_warp_specialize` 为核心。测试在其中演示 tt.func、常量、ttg.warp_specialize、整数加法，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 283-285
```mlir
283|     // CHECK: warp_yield %arg0
284|     ttg.warp_yield %1 : i32
285|   }
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 286-288
```mlir
286|   // CHECK: partition0
287|   partition0() num_warps(4) {
288|     %c0_i32_0 = arith.constant 0 : i32
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 289-294
```mlir
289|     // CHECK-NEXT: warp_return
290|     ttg.warp_return
291|   } : () -> i32
292|   tt.return %0 : i32
293| }
294| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.warp_return, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.warp_return、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 295-295
```mlir
295| // CHECK-LABEL: @unused_warp_specialize_results
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @unused_warp_specialize_results anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @unused_warp_specialize_results 这样的标签用于锚定匹配范围。

### Lines 296-296
```mlir
296| tt.func @unused_warp_specialize_results(%arg0: i32, %arg1: i32, %arg2: i32) -> (i32, i32) {
```
**EN:** This function-oriented block defines or enters `unused_warp_specialize_results`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `unused_warp_specialize_results` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 297-298
```mlir
297|   // CHECK-NEXT: [[OUTS:%.*]]:2 = ttg.warp_specialize
298|   %0:3 = ttg.warp_specialize()
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.warp_specialize. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.warp_specialize。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 299-300
```mlir
299|   // CHECK-NEXT: default
300|   default {
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 301-302
```mlir
301|     // CHECK-NEXT: ttg.warp_yield %arg0, %arg2 : i32, i32
302|     ttg.warp_yield %arg0, %arg1, %arg2 : i32, i32, i32
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.warp_yield. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.warp_yield。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 303-304
```mlir
303|   // CHECK-NEXT: () -> (i32, i32)
304|   } : () -> (i32, i32, i32)
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 305-309
```mlir
305|   // CHECK-NEXT: return [[OUTS]]#0, [[OUTS]]#1 : i32, i32
306|   tt.return %0#0, %0#2 : i32, i32
307| }
308| 
309| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 310-310
```mlir
310| // CHECK-LABEL: @unused_warp_specialize_captures
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @unused_warp_specialize_captures anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @unused_warp_specialize_captures 这样的标签用于锚定匹配范围。

### Lines 311-311
```mlir
311| tt.func @unused_warp_specialize_captures(%arg0: i32, %arg1: i32, %arg2: i32) {
```
**EN:** This function-oriented block defines or enters `unused_warp_specialize_captures`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `unused_warp_specialize_captures` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 312-316
```mlir
312|   // CHECK-NEXT: ttg.warp_specialize(%arg0, %arg2)
313|   ttg.warp_specialize(%arg0, %arg1, %arg2)
314|   default {
315|     ttg.warp_yield
316|   }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.warp_specialize, ttg.warp_yield. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.warp_specialize、ttg.warp_yield。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 317-318
```mlir
317|   // CHECK: partition0(%arg3: i32, %arg4: i32)
318|   partition0(%arg3: i32, %arg4: i32, %arg5: i32) num_warps(4) {
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 319-321
```mlir
319|     // CHECK-NEXT: "use"(%arg3, %arg4) : (i32, i32) -> ()
320|     "use"(%arg3, %arg5) : (i32, i32) -> ()
321|     ttg.warp_return
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.warp_return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.warp_return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 322-326
```mlir
322|   // CHECK: (i32, i32) -> ()
323|   } : (i32, i32, i32) -> ()
324|   tt.return
325| }
326| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 327-327
```mlir
327| // CHECK-LABEL: @unused_warp_specialize_captures_and_results
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @unused_warp_specialize_captures_and_results anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @unused_warp_specialize_captures_and_results 这样的标签用于锚定匹配范围。

### Lines 328-328
```mlir
328| tt.func @unused_warp_specialize_captures_and_results(%arg0: i32, %arg1: i32, %arg2: i32) -> (i32, i32) {
```
**EN:** This function-oriented block defines or enters `unused_warp_specialize_captures_and_results`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `unused_warp_specialize_captures_and_results` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 329-330
```mlir
329|   // CHECK-NEXT: [[OUTS:%.*]]:2 = ttg.warp_specialize
330|   %0:3 = ttg.warp_specialize(%arg0, %arg1, %arg2)
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.warp_specialize. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.warp_specialize。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 331-332
```mlir
331|   // CHECK-NEXT: default
332|   default {
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 333-335
```mlir
333|     // CHECK-NEXT: ttg.warp_yield %arg0, %arg2 : i32, i32
334|     ttg.warp_yield %arg0, %arg1, %arg2 : i32, i32, i32
335|   }
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 336-337
```mlir
336|   // CHECK: partition0(%arg3: i32, %arg4: i32)
337|   partition0(%arg3: i32, %arg4: i32, %arg5: i32) num_warps(4) {
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 338-340
```mlir
338|     // CHECK-NEXT: "use"(%arg3, %arg4) : (i32, i32) -> ()
339|     "use"(%arg3, %arg5) : (i32, i32) -> ()
340|     ttg.warp_return
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.warp_return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.warp_return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 341-342
```mlir
341|   // CHECK: (i32, i32) -> (i32, i32)
342|   } : (i32, i32, i32) -> (i32, i32, i32)
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 343-346
```mlir
343|   // CHECK-NEXT: return [[OUTS]]#0, [[OUTS]]#1 : i32, i32
344|   tt.return %0#0, %0#2 : i32, i32
345| }
346| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 347-347
```mlir
347| // CHECK-LABEL: @duplicate_warp_specialize_captures
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @duplicate_warp_specialize_captures anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @duplicate_warp_specialize_captures 这样的标签用于锚定匹配范围。

### Lines 348-348
```mlir
348| tt.func @duplicate_warp_specialize_captures(%arg0: i32, %arg1: i32, %arg2: i32) {
```
**EN:** This function-oriented block defines or enters `duplicate_warp_specialize_captures`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `duplicate_warp_specialize_captures` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 349-353
```mlir
349|   // CHECK-NEXT: ttg.warp_specialize(%arg0, %arg1)
350|   ttg.warp_specialize(%arg0, %arg1, %arg1, %arg2, %arg0)
351|   default {
352|     ttg.warp_yield
353|   }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.warp_specialize, ttg.warp_yield. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.warp_specialize、ttg.warp_yield。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 354-355
```mlir
354|   // CHECK: partition0(%arg3: i32, %arg4: i32)
355|   partition0(%arg3: i32, %arg4: i32, %arg5: i32, %arg6: i32, %arg7: i32) num_warps(4) {
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 356-362
```mlir
356|     // CHECK-NEXT: "use"(%arg3, %arg4, %arg4, %arg3)
357|     "use"(%arg3, %arg4, %arg5, %arg7) : (i32, i32, i32, i32) -> ()
358|     ttg.warp_return
359|   } : (i32, i32, i32, i32, i32) -> ()
360|   tt.return
361| }
362| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.warp_return, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.warp_return、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 363-363
```mlir
363| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 364-367
```mlir
364| 
365| #shared = #ttg.swizzled_shared<{vec = 16, perPhase = 2, maxPhase = 8, order = [0, 1]}>
366| #smem = #ttg.shared_memory
367| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 368-368
```mlir
368| // CHECK-LABEL: @fold_subslice_chain
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @fold_subslice_chain anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @fold_subslice_chain 这样的标签用于锚定匹配范围。

### Lines 369-369
```mlir
369| tt.func @fold_subslice_chain() {
```
**EN:** This function-oriented block defines or enters `fold_subslice_chain`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fold_subslice_chain` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 370-371
```mlir
370|   // CHECK: %[[ALLOC:.*]] = ttg.local_alloc
371|   %alloc = ttg.local_alloc : () -> !ttg.memdesc<32x64xf8E5M2, #shared, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 372-373
```mlir
372|   // CHECK-NOT: ttg.memdesc_subslice %[[ALLOC]][16, 32]
373|   %subslice = ttg.memdesc_subslice %alloc[16, 32] : !ttg.memdesc<32x64xf8E5M2, #shared, #smem, mutable> -> !ttg.memdesc<16x32xf8E5M2, #shared, #smem, mutable, 32x64>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.memdesc_subslice. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.memdesc_subslice。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 374-376
```mlir
374|   // CHECK: %[[SUBSLICE:.*]] = ttg.memdesc_subslice %[[ALLOC]][24, 48]
375|   %subslice2 = ttg.memdesc_subslice %subslice[8, 16] : !ttg.memdesc<16x32xf8E5M2, #shared, #smem, mutable, 32x64> -> !ttg.memdesc<8x16xf8E5M2, #shared, #smem, mutable, 32x64>
376|   %dummy_value = arith.constant dense<0.000000e+00> : tensor<8x16xf8E5M2>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.memdesc_subslice, constants. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.memdesc_subslice、常量。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 377-380
```mlir
377|   // CHECK: ttg.local_store %{{.*}}, %[[SUBSLICE]]
378|   ttg.local_store %dummy_value, %subslice2 : tensor<8x16xf8E5M2> -> !ttg.memdesc<8x16xf8E5M2, #shared, #smem, mutable, 32x64>
379|   tt.return
380| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on local/shared memory stores, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 本地/共享内存存储、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-canonicalize`, `-allow-unregistered-dialect`
- **CN:** 主要 pass 选项：`-split-input-file`，`-canonicalize`，`-allow-unregistered-dialect`
- **EN:** Dominant operations include `tt.func`, `tt.return`, `module`, `ttg.convert_layout`, `ttg.local_alloc`, `ttg.warp_specialize`, `ttg.warp_yield`, `ttg.warp_return`, `tt.reshape`, `ttg.local_load`.
- **CN:** 主要操作包括 `tt.func`、`tt.return`、`module`、`ttg.convert_layout`、`ttg.local_alloc`、`ttg.warp_specialize`、`ttg.warp_yield`、`ttg.warp_return`、`tt.reshape`、`ttg.local_load`。
- **EN:** The file contains 14 independently testable section(s). Check styles used: CHECK-NEXT x30, CHECK x27, CHECK-LABEL x21, CHECK-NOT x7. Important labels include @test_canonicalize_convert_view, @test_canonicalize_convert_expensive_view, @test_canonicalize_convert_expensive_view, @test_canonicalize_convert_view. It validates both the presence of the desired rewrite and the absence of unwanted residual operations.
- **CN:** 该文件包含 14 个可独立测试的分段。使用的检查类型：CHECK-NEXT ×30，CHECK ×27，CHECK-LABEL ×21，CHECK-NOT ×7。 关键标签包括 @test_canonicalize_convert_view，@test_canonicalize_convert_expensive_view，@test_canonicalize_convert_expensive_view，@test_canonicalize_convert_view。 它同时验证目标重写已经出现，并且不再残留不需要的操作。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `ttg`: TritonGPU layout, memory, and GPU-specific operations.
- **CN:** `ttg`：TritonGPU 布局、内存和 GPU 专用操作。
- **EN:** `tt`: Triton high-level tensor/pointer operations.
- **CN:** `tt`：Triton 高层张量/指针操作。
- **EN:** `arith`: MLIR arithmetic dialect for scalar/tensor math.
- **CN:** `arith`：用于标量/张量运算的 MLIR arithmetic 方言。
- **EN:** `ttng`: Triton Nvidia GPU tensor-memory and MMA operations.
- **CN:** `ttng`：Triton Nvidia GPU 张量内存与 MMA 操作。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。