# amd-update-async-wait-count.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/amd/amd-update-async-wait-count.mlir`
- **EN:** Test for the MLIR/Triton pipeline `--tritonamdgpu-update-async-wait-count=gfx-arch=gfx1250` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `--tritonamdgpu-update-async-wait-count=gfx-arch=gfx1250` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file --tritonamdgpu-update-async-wait-count=gfx-arch=gfx1250 | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file --tritonamdgpu-update-async-wait-count=gfx-arch=gfx1250 | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-4
```mlir
1| // RUN: triton-opt %s -split-input-file --tritonamdgpu-update-async-wait-count=gfx-arch=gfx1250 | FileCheck %s
2| 
3| // Simple case without any branching
4| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file --tritonamdgpu-update-async-wait-count=gfx-arch=gfx1250 | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file --tritonamdgpu-update-async-wait-count=gfx-arch=gfx1250 | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 5-9
```mlir
5| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [32, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
6| #blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
7| #shared = #ttg.swizzled_shared<{vec = 8, perPhase = 8, maxPhase = 2, order = [1, 0]}>
8| #shared1 = #ttg.padded_shared<[4:+4] {order = [1, 0], shape=[16, 256]}>
9| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 10-10
```mlir
10| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 11-11
```mlir
11|   // CHECK-LABEL: simple_waitcnt
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: simple_waitcnt anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: simple_waitcnt 这样的标签用于锚定匹配范围。

### Lines 12-20
```mlir
12|   tt.func public @simple_waitcnt(%arg1: !ttg.memdesc<128x16xf16, #shared, #smem, mutable>, %arg2: !ttg.memdesc<16x256xf16, #shared1, #smem, mutable>, %arg3: tensor<128x16x!tt.ptr<f16>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>}, %arg4: tensor<16x256x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>}) {
13|     // Emits 1 direct to lds instruction
14|     %0 = ttg.async_copy_global_to_local %arg3, %arg1 : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
15|     %1 = ttg.async_commit_group tokens %0
16|     // Emits 4 direct to lds instructions (padding interval limits vec to 4)
17|     %2 = ttg.async_copy_global_to_local %arg4, %arg2 : tensor<16x256x!tt.ptr<f16>, #blocked1> -> <16x256xf16, #shared1, #smem, mutable>
18|     %3 = ttg.async_commit_group tokens %2
19| 
20|     // Wait on token %1: 2 instructions outstanding (second async_copy emits 2)
```
**EN:** This function-oriented block defines or enters `simple_waitcnt`. Within it, the test exercises tt.func, ttg.async_copy_global_to_local, async copy commit groups, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `simple_waitcnt` 为核心。测试在其中演示 tt.func、ttg.async_copy_global_to_local、异步拷贝提交组，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 21-23
```mlir
21|     // CHECK: amdg.async_wait {{.*}} {num_inst = 4
22|     %9 = ttg.async_wait %1 {num = 0 : i32}
23|     // Wait on token %3: 0 instructions outstanding
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 24-29
```mlir
24|     // CHECK: amdg.async_wait {{.*}} {num_inst = 0
25|     %10 = ttg.async_wait %3 {num = 0 : i32}
26|     tt.return
27|   }
28| }
29| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on async wait synchronization, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 异步等待同步、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 30-30
```mlir
30| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 31-33
```mlir
31| 
32| // Simple case with amdg.buffer_load_to_local
33| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 34-38
```mlir
34| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [32, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
35| #blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
36| #shared = #ttg.swizzled_shared<{vec = 8, perPhase = 8, maxPhase = 2, order = [1, 0]}>
37| #shared1 = #ttg.padded_shared<[16:+4] {order = [1, 0], shape = [16, 256]}>
38| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 39-39
```mlir
39| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 40-40
```mlir
40|   // CHECK-LABEL: simple_buffer_load_to_local_waitcnt
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: simple_buffer_load_to_local_waitcnt anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: simple_buffer_load_to_local_waitcnt 这样的标签用于锚定匹配范围。

### Lines 41-48
```mlir
41|   tt.func public @simple_buffer_load_to_local_waitcnt(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg1: tensor<128x16xi32, #blocked> {tt.contiguity = dense<16> : tensor<2xi32>, tt.divisibility = dense<16> : tensor<2xi32>}, %arg2: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg3: tensor<16x256xi32, #blocked1> {tt.contiguity = dense<16> : tensor<2xi32>, tt.divisibility = dense<16> : tensor<2xi32>}, %arg4: !ttg.memdesc<128x16xf16, #shared, #smem, mutable>, %arg5: !ttg.memdesc<16x256xf16, #shared1, #smem, mutable>) {
42|     // Emits 1 direct to lds instruction
43|     %0 = amdg.buffer_load_to_local %arg0[%arg1] into %arg4 : <f16>[tensor<128x16xi32, #blocked>]  -> <128x16xf16, #shared, #smem, mutable>
44|     %1 = ttg.async_commit_group tokens %0
45|     // Emits 2 direct to lds instructions
46|     %2 = amdg.buffer_load_to_local %arg2[%arg3] into %arg5 : <f16>[tensor<16x256xi32, #blocked1>]  -> <16x256xf16, #shared1, #smem, mutable>
47|     %3 = ttg.async_commit_group tokens %2
48|     // Wait on token %1: 2 instructions outstanding (second buffer_load emits 2)
```
**EN:** This function-oriented block defines or enters `simple_buffer_load_to_local_waitcnt`. Within it, the test exercises tt.func, amdg.buffer_load_to_local, async copy commit groups, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `simple_buffer_load_to_local_waitcnt` 为核心。测试在其中演示 tt.func、amdg.buffer_load_to_local、异步拷贝提交组，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 49-51
```mlir
49|     // CHECK: amdg.async_wait {{.*}} {num_inst = 2
50|     %4 = ttg.async_wait %1 {num = 0 : i32}
51|     // Wait on token %3: 0 instructions outstanding
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 52-57
```mlir
52|     // CHECK: amdg.async_wait {{.*}} {num_inst = 0
53|     %5 = ttg.async_wait %3 {num = 0 : i32}
54|     tt.return
55|   }
56| }
57| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on async wait synchronization, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 异步等待同步、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 58-58
```mlir
58| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 59-61
```mlir
59| 
60| // Same as simple_waitcnt but swapped async_waits
61| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 62-66
```mlir
62| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [32, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
63| #blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
64| #shared = #ttg.swizzled_shared<{vec = 8, perPhase = 8, maxPhase = 2, order = [1, 0]}>
65| #shared1 = #ttg.swizzled_shared<{vec = 8, perPhase = 1, maxPhase = 16, order = [1, 0]}>
66| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 67-67
```mlir
67| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 68-68
```mlir
68|   // CHECK-LABEL: simple_waitcnt_reversed
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: simple_waitcnt_reversed anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: simple_waitcnt_reversed 这样的标签用于锚定匹配范围。

### Lines 69-77
```mlir
69|   tt.func public @simple_waitcnt_reversed(%arg1: !ttg.memdesc<128x16xf16, #shared, #smem, mutable>, %arg2: !ttg.memdesc<16x256xf16, #shared1, #smem, mutable>, %arg3: tensor<128x16x!tt.ptr<f16>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>}, %arg4: tensor<16x256x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>}) {
70|     // Emits 1 direct to lds instruction
71|     %0 = ttg.async_copy_global_to_local %arg3, %arg1 : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
72|     %1 = ttg.async_commit_group tokens %0
73|     // Emits 2 direct to lds instructions
74|     %2 = ttg.async_copy_global_to_local %arg4, %arg2 : tensor<16x256x!tt.ptr<f16>, #blocked1> -> <16x256xf16, #shared1, #smem, mutable>
75|     %3 = ttg.async_commit_group tokens %2
76| 
77|     // Wait on token %3: 0 instructions outstanding (nothing after %3)
```
**EN:** This function-oriented block defines or enters `simple_waitcnt_reversed`. Within it, the test exercises tt.func, ttg.async_copy_global_to_local, async copy commit groups, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `simple_waitcnt_reversed` 为核心。测试在其中演示 tt.func、ttg.async_copy_global_to_local、异步拷贝提交组，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 78-80
```mlir
78|     // CHECK: amdg.async_wait {{.*}} {num_inst = 0
79|     %9 = ttg.async_wait %3 {num = 0 : i32}
80|     // Wait on token %1: 2 instructions outstanding (second async_copy emits 2)
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 81-86
```mlir
81|     // CHECK: amdg.async_wait {{.*}} {num_inst = 2
82|     %10 = ttg.async_wait %1 {num = 0 : i32}
83|     tt.return
84|   }
85| }
86| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on async wait synchronization, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 异步等待同步、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 87-87
```mlir
87| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 88-90
```mlir
88| 
89| // We should ignore tt.loads when counting
90| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 91-95
```mlir
91| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [32, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
92| #blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
93| #shared = #ttg.swizzled_shared<{vec = 8, perPhase = 8, maxPhase = 2, order = [1, 0]}>
94| #shared1 = #ttg.swizzled_shared<{vec = 8, perPhase = 1, maxPhase = 16, order = [1, 0]}>
95| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 96-96
```mlir
96| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 97-97
```mlir
97|   // CHECK-LABEL: simple_waitcnt_with_tt_load
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: simple_waitcnt_with_tt_load anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: simple_waitcnt_with_tt_load 这样的标签用于锚定匹配范围。

### Lines 98-108
```mlir
 98|   tt.func public @simple_waitcnt_with_tt_load(%arg1: !ttg.memdesc<128x16xf16, #shared, #smem, mutable>, %arg2: !ttg.memdesc<16x256xf16, #shared1, #smem, mutable>, %arg3: tensor<128x16x!tt.ptr<f16>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>}, %arg4: tensor<16x256x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>}) {
 99|     // Emits 1 direct to lds instruction
100|     %0 = ttg.async_copy_global_to_local %arg3, %arg1 : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
101|     %1 = ttg.async_commit_group tokens %0
102|     // Emits 2 direct to lds instructions
103|     %2 = ttg.async_copy_global_to_local %arg4, %arg2 : tensor<16x256x!tt.ptr<f16>, #blocked1> -> <16x256xf16, #shared1, #smem, mutable>
104|     %3 = ttg.async_commit_group tokens %2
105| 
106|     %4 = tt.load %arg3 : tensor<128x16x!tt.ptr<f16>, #blocked>
107| 
108|     // Wait on token %1: 2 instructions outstanding (tt.load ignored, second async_copy emits 2)
```
**EN:** This function-oriented block defines or enters `simple_waitcnt_with_tt_load`. Within it, the test exercises tt.func, ttg.async_copy_global_to_local, async copy commit groups, masked or vectorized loads, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `simple_waitcnt_with_tt_load` 为核心。测试在其中演示 tt.func、ttg.async_copy_global_to_local、异步拷贝提交组、带掩码或向量化的加载，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 109-111
```mlir
109|     // CHECK: amdg.async_wait {{.*}} {num_inst = 2
110|     %9 = ttg.async_wait %1 {num = 0 : i32}
111|     // Wait on token %3: 0 instructions outstanding
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 112-117
```mlir
112|     // CHECK: amdg.async_wait {{.*}} {num_inst = 0
113|     %10 = ttg.async_wait %3 {num = 0 : i32}
114|     tt.return
115|   }
116| }
117| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on async wait synchronization, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 异步等待同步、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 118-118
```mlir
118| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 119-121
```mlir
119| 
120| // Simple loop without any interleaving loads so we expect waitcnt 0
121| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 122-126
```mlir
122| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [32, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
123| #blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
124| #shared = #ttg.swizzled_shared<{vec = 8, perPhase = 8, maxPhase = 2, order = [1, 0]}>
125| #shared1 = #ttg.swizzled_shared<{vec = 8, perPhase = 1, maxPhase = 16, order = [1, 0]}>
126| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 127-127
```mlir
127| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 128-128
```mlir
128|   // CHECK-LABEL wait_in_for_loop
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL wait_in_for_loop anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL wait_in_for_loop 这样的标签用于锚定匹配范围。

### Lines 129-139
```mlir
129|   tt.func public @wait_in_for_loop(%arg0: i32, %arg1: !ttg.memdesc<128x16xf16, #shared, #smem, mutable>, %arg2: !ttg.memdesc<16x256xf16, #shared1, #smem, mutable>, %arg3: tensor<128x16x!tt.ptr<f16>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>}, %arg4: tensor<16x256x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>}) {
130|     %c0_i32 = arith.constant 0 : i32
131|     %c1_i32 = arith.constant 1 : i32
132|     // Emits 1 direct to lds instruction
133|     %0 = ttg.async_copy_global_to_local %arg3, %arg1 : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
134|     %1 = ttg.async_commit_group tokens %0
135|     // Emits 2 direct to lds instructions
136|     %2 = ttg.async_copy_global_to_local %arg4, %arg2 : tensor<16x256x!tt.ptr<f16>, #blocked1> -> <16x256xf16, #shared1, #smem, mutable>
137|     %3 = ttg.async_commit_group tokens %2
138|     %8:2 = scf.for %arg14 = %c0_i32 to %arg0 step %c1_i32 iter_args(%arg15 = %1, %arg16 = %3) -> (!ttg.async.token, !ttg.async.token)  : i32 {
139|       // min over tokens: %arg16 has 0 outstanding → overall 0
```
**EN:** This function-oriented block defines or enters `wait_in_for_loop`. Within it, the test exercises tt.func, constants, ttg.async_copy_global_to_local, async copy commit groups, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `wait_in_for_loop` 为核心。测试在其中演示 tt.func、常量、ttg.async_copy_global_to_local、异步拷贝提交组、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 140-147
```mlir
140|       // CHECK: amdg.async_wait {{.*}}, {{.*}} {num_inst = 0
141|       %10 = ttg.async_wait %arg15, %arg16 {num = 2 : i32}
142|       %11 = ttg.async_copy_global_to_local %arg3, %arg1 : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
143|       %12 = ttg.async_commit_group tokens %11
144|       %13 = ttg.async_copy_global_to_local %arg4, %arg2 : tensor<16x256x!tt.ptr<f16>, #blocked1> -> <16x256xf16, #shared1, #smem, mutable>
145|       %14 = ttg.async_commit_group tokens %13
146|       scf.yield %12, %14: !ttg.async.token, !ttg.async.token
147|     }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining ttg.async_copy_global_to_local, async copy commit groups, async wait synchronization, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 ttg.async_copy_global_to_local、异步拷贝提交组、异步等待同步、循环/分支产出值。

### Lines 148-153
```mlir
148|     // CHECK: amdg.async_wait {{.*}}, {{.*}} {num_inst = 0
149|     %9 = ttg.async_wait %8#0, %8#1 {num = 0 : i32}
150|     tt.return
151|   }
152| }
153| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on async wait synchronization, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 异步等待同步、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 154-154
```mlir
154| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 155-157
```mlir
155| 
156| // Double buffering for 2 loads where the first one will emit 2 instructions and the second 1 instruction so we expect waitcnt 3 inside the loop and 0 in the epilogue
157| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 158-162
```mlir
158| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [32, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
159| #blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
160| #shared = #ttg.swizzled_shared<{vec = 8, perPhase = 8, maxPhase = 2, order = [1, 0]}>
161| #shared1 = #ttg.swizzled_shared<{vec = 8, perPhase = 1, maxPhase = 16, order = [1, 0]}>
162| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 163-163
```mlir
163| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 164-164
```mlir
164|   // CHECK-LABEL double_buffering
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL double_buffering anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL double_buffering 这样的标签用于锚定匹配范围。

### Lines 165-179
```mlir
165|   tt.func public @double_buffering(%arg0: i32, %arg1: !ttg.memdesc<128x16xf16, #shared, #smem, mutable>, %arg2: !ttg.memdesc<16x256xf16, #shared1, #smem, mutable>, %arg3: tensor<128x16x!tt.ptr<f16>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>}, %arg4: tensor<16x256x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>}) {
166|     %c0_i32 = arith.constant 0 : i32
167|     %c1_i32 = arith.constant 1 : i32
168|     // Emits 1 direct to lds instruction
169|     %0 = ttg.async_copy_global_to_local %arg3, %arg1 : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
170|     %1 = ttg.async_commit_group tokens %0
171|     // Emits 2 direct to lds instructions
172|     %2 = ttg.async_copy_global_to_local %arg4, %arg2 : tensor<16x256x!tt.ptr<f16>, #blocked1> -> <16x256xf16, #shared1, #smem, mutable>
173|     %3 = ttg.async_commit_group tokens %2
174|     %4 = ttg.async_copy_global_to_local %arg3, %arg1 : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
175|     %5 = ttg.async_commit_group tokens %4
176|     %6 = ttg.async_copy_global_to_local %arg4, %arg2 : tensor<16x256x!tt.ptr<f16>, #blocked1> -> <16x256xf16, #shared1, #smem, mutable>
177|     %7 = ttg.async_commit_group tokens %6
178|     %8:4 = scf.for %arg14 = %c0_i32 to %arg0 step %c1_i32 iter_args(%arg15 = %1, %arg16 = %5, %arg17 = %3, %arg18 = %7) -> (!ttg.async.token, !ttg.async.token, !ttg.async.token, !ttg.async.token)  : i32 {
179|       // 2 loads per buffer: first emits 1, second emits 2 → 3 per iteration
```
**EN:** This function-oriented block defines or enters `double_buffering`. Within it, the test exercises ttg.async_copy_global_to_local, async copy commit groups, tt.func, constants, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `double_buffering` 为核心。测试在其中演示 ttg.async_copy_global_to_local、异步拷贝提交组、tt.func、常量、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 180-187
```mlir
180|       // CHECK: amdg.async_wait {{.*}}, {{.*}} {num_inst = 3
181|       %10 = ttg.async_wait %arg15, %arg17 {num = 2 : i32}
182|       %11 = ttg.async_copy_global_to_local %arg3, %arg1 : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
183|       %12 = ttg.async_commit_group tokens %11
184|       %13 = ttg.async_copy_global_to_local %arg4, %arg2 : tensor<16x256x!tt.ptr<f16>, #blocked1> -> <16x256xf16, #shared1, #smem, mutable>
185|       %14 = ttg.async_commit_group tokens %13
186|       scf.yield %arg16, %12, %arg18, %14 : !ttg.async.token, !ttg.async.token, !ttg.async.token, !ttg.async.token
187|     }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining ttg.async_copy_global_to_local, async copy commit groups, async wait synchronization, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 ttg.async_copy_global_to_local、异步拷贝提交组、异步等待同步、循环/分支产出值。

### Lines 188-192
```mlir
188|     // CHECK: amdg.async_wait {{.*}}, {{.*}} {num_inst = 0
189|     %9 = ttg.async_wait %8#0, %8#1, %8#2, %8#3 {num = 0 : i32}
190|     tt.return
191|   }
192| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on async wait synchronization, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 异步等待同步、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 193-193
```mlir
193| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 194-196
```mlir
194| 
195| // Double buffering with async_wait inside scf.if
196| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 197-201
```mlir
197| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [32, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
198| #blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
199| #shared = #ttg.swizzled_shared<{vec = 8, perPhase = 8, maxPhase = 2, order = [1, 0]}>
200| #shared1 = #ttg.swizzled_shared<{vec = 8, perPhase = 1, maxPhase = 16, order = [1, 0]}>
201| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 202-202
```mlir
202| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 203-203
```mlir
203|   // CHECK-LABEL: double_buffering_wait_in_if
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: double_buffering_wait_in_if anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: double_buffering_wait_in_if 这样的标签用于锚定匹配范围。

### Lines 204-218
```mlir
204|   tt.func public @double_buffering_wait_in_if(%cond: i1, %arg0: i32, %arg1: !ttg.memdesc<128x16xf16, #shared, #smem, mutable>, %arg2: !ttg.memdesc<16x256xf16, #shared1, #smem, mutable>, %arg3: tensor<128x16x!tt.ptr<f16>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>}, %arg4: tensor<16x256x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>}) {
205|     %c0_i32 = arith.constant 0 : i32
206|     %c1_i32 = arith.constant 1 : i32
207|     // Emits 1 direct to lds instruction
208|     %0 = ttg.async_copy_global_to_local %arg3, %arg1 : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
209|     %1 = ttg.async_commit_group tokens %0
210|     // Emits 2 direct to lds instructions
211|     %2 = ttg.async_copy_global_to_local %arg4, %arg2 : tensor<16x256x!tt.ptr<f16>, #blocked1> -> <16x256xf16, #shared1, #smem, mutable>
212|     %3 = ttg.async_commit_group tokens %2
213|     %4 = ttg.async_copy_global_to_local %arg3, %arg1 : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
214|     %5 = ttg.async_commit_group tokens %4
215|     %6 = ttg.async_copy_global_to_local %arg4, %arg2 : tensor<16x256x!tt.ptr<f16>, #blocked1> -> <16x256xf16, #shared1, #smem, mutable>
216|     %7 = ttg.async_commit_group tokens %6
217|     %8:4 = scf.for %arg14 = %c0_i32 to %arg0 step %c1_i32 iter_args(%arg15 = %1, %arg16 = %5, %arg17 = %3, %arg18 = %7) -> (!ttg.async.token, !ttg.async.token, !ttg.async.token, !ttg.async.token) : i32 {
218|       %103 = scf.if %cond -> (!ttg.async.token) {
```
**EN:** This function-oriented block defines or enters `double_buffering_wait_in_if`. Within it, the test exercises ttg.async_copy_global_to_local, async copy commit groups, tt.func, constants, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `double_buffering_wait_in_if` 为核心。测试在其中演示 ttg.async_copy_global_to_local、异步拷贝提交组、tt.func、常量、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 219-222
```mlir
219|         // CHECK: amdg.async_wait {{.*}}, {{.*}} {num_inst = 3
220|         %token1 = ttg.async_wait %arg15, %arg17 {num = 2 : i32}
221|         scf.yield %token1 : !ttg.async.token
222|       } else {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining async wait synchronization, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 异步等待同步、循环/分支产出值。

### Lines 223-232
```mlir
223|         // CHECK: amdg.async_wait {{.*}} {num_inst = 5
224|         %token2 = ttg.async_wait %arg15 {num = 1 : i32}
225|         scf.yield %token2 : !ttg.async.token
226|       }
227|       %11 = ttg.async_copy_global_to_local %arg3, %arg1 : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
228|       %12 = ttg.async_commit_group tokens %11
229|       %13 = ttg.async_copy_global_to_local %arg4, %arg2 : tensor<16x256x!tt.ptr<f16>, #blocked1> -> <16x256xf16, #shared1, #smem, mutable>
230|       %14 = ttg.async_commit_group tokens %13
231|       scf.yield %arg16, %12, %arg18, %14 : !ttg.async.token, !ttg.async.token, !ttg.async.token, !ttg.async.token
232|     }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, ttg.async_copy_global_to_local, async copy commit groups, async wait synchronization.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、ttg.async_copy_global_to_local、异步拷贝提交组、异步等待同步。

### Lines 233-238
```mlir
233|     // CHECK: amdg.async_wait {{.*}}, {{.*}} {num_inst = 0
234|     %9 = ttg.async_wait %8#0, %8#1, %8#2, %8#3 {num = 0 : i32}
235|     tt.return
236|   }
237| }
238| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on async wait synchronization, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 异步等待同步、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 239-239
```mlir
239| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 240-242
```mlir
240| 
241| // Double buffering with async_wait and additional async_loads inside the scf.if
242| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 243-247
```mlir
243| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [32, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
244| #blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
245| #shared = #ttg.swizzled_shared<{vec = 8, perPhase = 8, maxPhase = 2, order = [1, 0]}>
246| #shared1 = #ttg.swizzled_shared<{vec = 8, perPhase = 1, maxPhase = 16, order = [1, 0]}>
247| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 248-248
```mlir
248| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 249-249
```mlir
249|   // CHECK-LABEL: doube_buffering_wait_loads_in_if
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: doube_buffering_wait_loads_in_if anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: doube_buffering_wait_loads_in_if 这样的标签用于锚定匹配范围。

### Lines 250-266
```mlir
250|   tt.func public @doube_buffering_wait_loads_in_if(%cond: i1, %arg0: i32, %arg1: !ttg.memdesc<128x16xf16, #shared, #smem, mutable>, %arg2: !ttg.memdesc<16x256xf16, #shared1, #smem, mutable>, %arg3: tensor<128x16x!tt.ptr<f16>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>}, %arg4: tensor<16x256x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>}) {
251|     %c0_i32 = arith.constant 0 : i32
252|     %c1_i32 = arith.constant 1 : i32
253|     // Emits 1 direct to lds instruction
254|     %0 = ttg.async_copy_global_to_local %arg3, %arg1 : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
255|     %1 = ttg.async_commit_group tokens %0
256|     // Emits 2 direct to lds instructions
257|     %2 = ttg.async_copy_global_to_local %arg4, %arg2 : tensor<16x256x!tt.ptr<f16>, #blocked1> -> <16x256xf16, #shared1, #smem, mutable>
258|     %3 = ttg.async_commit_group tokens %2
259|     %4 = ttg.async_copy_global_to_local %arg3, %arg1 : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
260|     %5 = ttg.async_commit_group tokens %4
261|     %6 = ttg.async_copy_global_to_local %arg4, %arg2 : tensor<16x256x!tt.ptr<f16>, #blocked1> -> <16x256xf16, #shared1, #smem, mutable>
262|     %7 = ttg.async_commit_group tokens %6
263|     %8:4 = scf.for %arg14 = %c0_i32 to %arg0 step %c1_i32 iter_args(%arg15 = %1, %arg16 = %5, %arg17 = %3, %arg18 = %7) -> (!ttg.async.token, !ttg.async.token, !ttg.async.token, !ttg.async.token)  : i32 {
264|       %103 = scf.if %cond -> (!ttg.async.token) {
265|         %cond_load = ttg.async_copy_global_to_local %arg4, %arg2 : tensor<16x256x!tt.ptr<f16>, #blocked1> -> <16x256xf16, #shared1, #smem, mutable>
266|         %cond_load_commit = ttg.async_commit_group tokens %cond_load
```
**EN:** This function-oriented block defines or enters `doube_buffering_wait_loads_in_if`. Within it, the test exercises ttg.async_copy_global_to_local, async copy commit groups, tt.func, constants, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `doube_buffering_wait_loads_in_if` 为核心。测试在其中演示 ttg.async_copy_global_to_local、异步拷贝提交组、tt.func、常量、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 267-278
```mlir
267|         // CHECK: amdg.async_wait {{.*}}, {{.*}} {num_inst = 5
268|         %token1 = ttg.async_wait %arg15, %arg17 {num = 2 : i32}
269|         scf.yield %token1 : !ttg.async.token
270|       } else {
271|         scf.yield %arg15 : !ttg.async.token
272|       }
273|       %11 = ttg.async_copy_global_to_local %arg3, %arg1 : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
274|       %12 = ttg.async_commit_group tokens %11
275|       %13 = ttg.async_copy_global_to_local %arg4, %arg2 : tensor<16x256x!tt.ptr<f16>, #blocked1> -> <16x256xf16, #shared1, #smem, mutable>
276|       %14 = ttg.async_commit_group tokens %13
277|       scf.yield %arg16, %12, %arg18, %14 : !ttg.async.token, !ttg.async.token, !ttg.async.token, !ttg.async.token
278|     }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, ttg.async_copy_global_to_local, async copy commit groups, async wait synchronization.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、ttg.async_copy_global_to_local、异步拷贝提交组、异步等待同步。

### Lines 279-284
```mlir
279|     // CHECK: amdg.async_wait {{.*}}, {{.*}} {num_inst = 0
280|     %9 = ttg.async_wait %8#0, %8#1, %8#2, %8#3 {num = 0 : i32}
281|     tt.return
282|   }
283| }
284| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on async wait synchronization, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 异步等待同步、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 285-285
```mlir
285| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 286-288
```mlir
286| 
287| // Double buffering with different number of async_copies inside scf.if then and else block. Check that we take the lower number from both blocks
288| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 289-293
```mlir
289| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [32, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
290| #blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
291| #shared = #ttg.swizzled_shared<{vec = 8, perPhase = 8, maxPhase = 2, order = [1, 0]}>
292| #shared1 = #ttg.swizzled_shared<{vec = 8, perPhase = 1, maxPhase = 16, order = [1, 0]}>
293| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 294-294
```mlir
294| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 295-295
```mlir
295|   // CHECK-LABEL: double_buffering_uneven_then_else
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: double_buffering_uneven_then_else anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: double_buffering_uneven_then_else 这样的标签用于锚定匹配范围。

### Lines 296-309
```mlir
296|   tt.func public @double_buffering_uneven_then_else(%cond: i1, %arg0: i32, %arg1: !ttg.memdesc<128x16xf16, #shared, #smem, mutable>, %arg2: !ttg.memdesc<16x256xf16, #shared1, #smem, mutable>, %arg3: tensor<128x16x!tt.ptr<f16>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>}, %arg4: tensor<16x256x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>}) {
297|     %c0_i32 = arith.constant 0 : i32
298|     %c1_i32 = arith.constant 1 : i32
299|     // Emits 1 direct to lds instruction
300|     %0 = ttg.async_copy_global_to_local %arg3, %arg1 : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
301|     %1 = ttg.async_commit_group tokens %0
302|     // Emits 2 direct to lds instructions
303|     %2 = ttg.async_copy_global_to_local %arg4, %arg2 : tensor<16x256x!tt.ptr<f16>, #blocked1> -> <16x256xf16, #shared1, #smem, mutable>
304|     %3 = ttg.async_commit_group tokens %2
305|     %4 = ttg.async_copy_global_to_local %arg3, %arg1 : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
306|     %5 = ttg.async_commit_group tokens %4
307|     %6 = ttg.async_copy_global_to_local %arg4, %arg2 : tensor<16x256x!tt.ptr<f16>, #blocked1> -> <16x256xf16, #shared1, #smem, mutable>
308|     %7 = ttg.async_commit_group tokens %6
309|     %8:4 = scf.for %arg14 = %c0_i32 to %arg0 step %c1_i32 iter_args(%arg15 = %1, %arg16 = %5, %arg17 = %3, %arg18 = %7) -> (!ttg.async.token, !ttg.async.token, !ttg.async.token, !ttg.async.token)  : i32 {
```
**EN:** This function-oriented block defines or enters `double_buffering_uneven_then_else`. Within it, the test exercises ttg.async_copy_global_to_local, async copy commit groups, tt.func, constants, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `double_buffering_uneven_then_else` 为核心。测试在其中演示 ttg.async_copy_global_to_local、异步拷贝提交组、tt.func、常量、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 310-326
```mlir
310|       // CHECK: amdg.async_wait {{.*}}, {{.*}} {num_inst = 3
311|       %token1 = ttg.async_wait %arg15, %arg17 {num = 2 : i32}
312| 
313|       %103 = scf.if %cond -> (!ttg.async.token) {
314|         %11 = ttg.async_copy_global_to_local %arg3, %arg1 : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
315|         %110 = ttg.async_copy_global_to_local %arg3, %arg1 : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
316|         %12 = ttg.async_commit_group tokens %11, %110
317|         scf.yield %12 : !ttg.async.token
318|       } else {
319|         %11 = ttg.async_copy_global_to_local %arg3, %arg1 : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
320|         %12 = ttg.async_commit_group tokens %11
321|         scf.yield %12 : !ttg.async.token
322|       }
323|       %13 = ttg.async_copy_global_to_local %arg4, %arg2 : tensor<16x256x!tt.ptr<f16>, #blocked1> -> <16x256xf16, #shared1, #smem, mutable>
324|       %14 = ttg.async_commit_group tokens %13
325|       scf.yield %arg16, %103, %arg18, %14 : !ttg.async.token, !ttg.async.token, !ttg.async.token, !ttg.async.token
326|     }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining ttg.async_copy_global_to_local, async copy commit groups, loop/if yielded values, async wait synchronization, structured conditionals.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 ttg.async_copy_global_to_local、异步拷贝提交组、循环/分支产出值、异步等待同步、结构化条件分支。

### Lines 327-332
```mlir
327|     // CHECK: amdg.async_wait {{.*}}, {{.*}} {num_inst = 0
328|     %9 = ttg.async_wait %8#0, %8#1, %8#2, %8#3 {num = 0 : i32}
329|     tt.return
330|   }
331| }
332| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on async wait synchronization, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 异步等待同步、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 333-333
```mlir
333| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 334-336
```mlir
334| 
335| // Test for dynamic loop in def chain
336| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 337-339
```mlir
337| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [32, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
338| #shared = #ttg.swizzled_shared<{vec = 8, perPhase = 8, maxPhase = 2, order = [1, 0]}>
339| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 340-340
```mlir
340| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 341-341
```mlir
341|   // CHECK-LABEL: dynamic_loop_in_def_chain
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: dynamic_loop_in_def_chain anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: dynamic_loop_in_def_chain 这样的标签用于锚定匹配范围。

### Lines 342-353
```mlir
342|   tt.func public @dynamic_loop_in_def_chain(%arg0: i32, %arg1: !ttg.memdesc<128x16xf16, #shared, #smem, mutable>, %arg3: tensor<128x16x!tt.ptr<f16>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>}) {
343|     %c0_i32 = arith.constant 0 : i32
344|     %c1_i32 = arith.constant 1 : i32
345|     %c4_i32 = arith.constant 4 : i32
346|     // Emits 1 direct to lds instruction
347|     %0 = ttg.async_copy_global_to_local %arg3, %arg1 : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
348|     %1 = ttg.async_commit_group tokens %0
349|     // Emits 1 direct to lds instruction
350|     %6 = ttg.async_copy_global_to_local %arg3, %arg1 : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
351|     %7 = ttg.async_commit_group tokens %6
352|     // Dynamic iteration count
353|     %30 = scf.for %arg21 = %c0_i32 to %arg0 step %c1_i32 iter_args(%arg30 = %6) -> (!ttg.async.token) : i32 {
```
**EN:** This function-oriented block defines or enters `dynamic_loop_in_def_chain`. Within it, the test exercises constants, tt.func, ttg.async_copy_global_to_local, async copy commit groups, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `dynamic_loop_in_def_chain` 为核心。测试在其中演示 常量、tt.func、ttg.async_copy_global_to_local、异步拷贝提交组、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 354-360
```mlir
354|       // CHECK: amdg.async_wait {{.*}} {num_inst = 0
355|       %31 = ttg.async_wait %arg30 {num = 1 : i32}
356|       // Emits 1 direct to lds instruction
357|       %32 = ttg.async_copy_global_to_local %arg3, %arg1 : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
358|       %33 = ttg.async_commit_group tokens %32
359|       scf.yield %33 : !ttg.async.token
360|     }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining async wait synchronization, ttg.async_copy_global_to_local, async copy commit groups, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 异步等待同步、ttg.async_copy_global_to_local、异步拷贝提交组、循环/分支产出值。

### Lines 361-366
```mlir
361|     // CHECK: amdg.async_wait {{.*}} {num_inst = 1
362|     %10 = ttg.async_wait %1 {num = 1 : i32}
363|     tt.return
364|   }
365| }
366| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on async wait synchronization, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 异步等待同步、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 367-367
```mlir
367| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 368-370
```mlir
368| 
369| // Test loop in def chain with constant iteration count
370| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 371-373
```mlir
371| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [32, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
372| #shared = #ttg.swizzled_shared<{vec = 8, perPhase = 8, maxPhase = 2, order = [1, 0]}>
373| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 374-374
```mlir
374| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 375-375
```mlir
375|   // CHECK-LABEL: constant_loop_in_def_chain
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: constant_loop_in_def_chain anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: constant_loop_in_def_chain 这样的标签用于锚定匹配范围。

### Lines 376-386
```mlir
376|   tt.func public @constant_loop_in_def_chain(%arg1: !ttg.memdesc<128x16xf16, #shared, #smem, mutable>, %arg3: tensor<128x16x!tt.ptr<f16>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>}) {
377|     %c0_i32 = arith.constant 0 : i32
378|     %c1_i32 = arith.constant 1 : i32
379|     %c4_i32 = arith.constant 4 : i32
380|     // Emits 1 direct to lds instruction
381|     %0 = ttg.async_copy_global_to_local %arg3, %arg1 : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
382|     %1 = ttg.async_commit_group tokens %0
383|     // Emits 1 direct to lds instruction
384|     %6 = ttg.async_copy_global_to_local %arg3, %arg1 : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
385|     %7 = ttg.async_commit_group tokens %6
386|     %30 = scf.for %arg21 = %c0_i32 to %c4_i32 step %c1_i32 iter_args(%arg30 = %6) -> (!ttg.async.token) : i32 {
```
**EN:** This function-oriented block defines or enters `constant_loop_in_def_chain`. Within it, the test exercises constants, tt.func, ttg.async_copy_global_to_local, async copy commit groups, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `constant_loop_in_def_chain` 为核心。测试在其中演示 常量、tt.func、ttg.async_copy_global_to_local、异步拷贝提交组、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 387-393
```mlir
387|       // CHECK: amdg.async_wait {{.*}} {num_inst = 0
388|       %31 = ttg.async_wait %arg30 {num = 1 : i32}
389|       // Emits 1 direct to lds instruction
390|       %32 = ttg.async_copy_global_to_local %arg3, %arg1 : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
391|       %33 = ttg.async_commit_group tokens %32
392|       scf.yield %33 : !ttg.async.token
393|     }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining async wait synchronization, ttg.async_copy_global_to_local, async copy commit groups, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 异步等待同步、ttg.async_copy_global_to_local、异步拷贝提交组、循环/分支产出值。

### Lines 394-399
```mlir
394|     // CHECK: amdg.async_wait {{.*}} {num_inst = 5
395|     %10 = ttg.async_wait %1 {num = 1 : i32}
396|     tt.return
397|   }
398| }
399| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on async wait synchronization, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 异步等待同步、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 400-400
```mlir
400| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 401-403
```mlir
401| 
402| // Simple case with TDM
403| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 404-406
```mlir
404| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
405| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
406| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 407-407
```mlir
407| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 408-408
```mlir
408|   // CHECK-LABEL: simple_tdm_waitcnt
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: simple_tdm_waitcnt anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: simple_tdm_waitcnt 这样的标签用于锚定匹配范围。

### Lines 409-417
```mlir
409|   tt.func public @simple_tdm_waitcnt(%memDesc: !ttg.memdesc<128x16xf16, #shared, #smem, mutable>, %tensorDesc: !tt.tensordesc<128x16xf16>, %mask: i32
410|   ) {
411|     %c0_i32 = arith.constant 0 : i32
412| 
413|     // Each async_tdm_copy only emits a single instruction (-> counts 1)
414|     %1 = amdg.async_tdm_copy_global_to_local %tensorDesc[%c0_i32, %c0_i32] into %memDesc, pred = %mask : !tt.tensordesc<128x16xf16> -> !ttg.memdesc<128x16xf16, #shared, #smem, mutable>
415|     %2 = amdg.async_tdm_copy_global_to_local %tensorDesc[%c0_i32, %c0_i32] into %memDesc, pred = %mask : !tt.tensordesc<128x16xf16> -> !ttg.memdesc<128x16xf16, #shared, #smem, mutable>
416| 
417|     // Do not wait on the second tdm => waitcnt 1
```
**EN:** This function-oriented block defines or enters `simple_tdm_waitcnt`. Within it, the test exercises tt.func, amdg.async_tdm_copy_global_to_local, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `simple_tdm_waitcnt` 为核心。测试在其中演示 tt.func、amdg.async_tdm_copy_global_to_local、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 418-420
```mlir
418|     // CHECK: amdg.async_tdm_intrinsic_wait {{.*}} {count = 1
419|     %w1 = amdg.async_tdm_wait %1 {num = 0 : i32}
420|     // No async_copies in between => waitcnt 0
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 421-426
```mlir
421|     // CHECK: amdg.async_tdm_intrinsic_wait {{.*}} {count = 0
422|     %w2 = amdg.async_tdm_wait %2 {num = 0 : i32}
423|     tt.return
424|   }
425| }
426| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.async_tdm_wait, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.async_tdm_wait、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 427-427
```mlir
427| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 428-431
```mlir
428| 
429| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
430| #shared = #ttg.swizzled_shared<{vec = 8, perPhase = 8, maxPhase = 2, order = [1, 0]}>
431| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 432-432
```mlir
432| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 433-433
```mlir
433|   // CHECK-LABEL: contiguity_hint_waitcnt
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: contiguity_hint_waitcnt anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: contiguity_hint_waitcnt 这样的标签用于锚定匹配范围。

### Lines 434-445
```mlir
434|   tt.func public @contiguity_hint_waitcnt(%arg1: !ttg.memdesc<64x32xf16, #shared, #smem, mutable>, %arg2: tensor<64x32x!tt.ptr<f16>, #blocked>) {
435|     // Dummy commit group to wait on both loads
436|     %0 = ttg.async_commit_group
437| 
438|     // 16 loads since vec=1 and no hints
439|     %1 = ttg.async_copy_global_to_local %arg2, %arg1 : tensor<64x32x!tt.ptr<f16>, #blocked> -> <64x32xf16, #shared, #smem, mutable>
440|     %2 = ttg.async_commit_group tokens %1
441| 
442|     // Contig hint bumps contig to 8 -> 2 loads
443|     %3 = ttg.async_copy_global_to_local %arg2, %arg1 {contiguity = 8 : i32} : tensor<64x32x!tt.ptr<f16>, #blocked> -> <64x32xf16, #shared, #smem, mutable>
444|     %4 = ttg.async_commit_group tokens %3
445| 
```
**EN:** This function-oriented block defines or enters `contiguity_hint_waitcnt`. Within it, the test exercises async copy commit groups, tt.func, ttg.async_copy_global_to_local, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `contiguity_hint_waitcnt` 为核心。测试在其中演示 异步拷贝提交组、tt.func、ttg.async_copy_global_to_local，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 446-447
```mlir
446|     // CHECK: amdg.async_wait {{.*}} {num_inst = 2
447|     %5 = ttg.async_wait %2 {num = 0 : i32}
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 448-454
```mlir
448|     // CHECK: amdg.async_wait {{.*}} {num_inst = 18
449|     %6 = ttg.async_wait %0 {num = 0 : i32}
450| 
451|     tt.return
452|   }
453| }
454| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on async wait synchronization, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 异步等待同步、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 455-455
```mlir
455| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 456-458
```mlir
456| 
457| // Test async_copy_local_to_global on GFX1250
458| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 459-461
```mlir
459| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
460| #shared = #ttg.swizzled_shared<{vec = 8, perPhase = 8, maxPhase = 2, order = [1, 0]}>
461| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 462-462
```mlir
462| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 463-463
```mlir
463|   // CHECK-LABEL: simple_local_to_global_waitcnt
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: simple_local_to_global_waitcnt anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: simple_local_to_global_waitcnt 这样的标签用于锚定匹配范围。

### Lines 464-472
```mlir
464|   tt.func public @simple_local_to_global_waitcnt(%arg1: !ttg.memdesc<32x32xf32, #shared, #smem, mutable>, %arg2: tensor<32x32x!tt.ptr<f32>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>}) {
465|     // Emits 2 async store instructions (256 bits per thread, split into 2x128-bit stores)
466|     %0 = amdg.async_copy_local_to_global %arg1, %arg2 : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32x!tt.ptr<f32>, #blocked>
467|     %1 = ttg.async_commit_group tokens %0
468|     // Emits 2 async store instructions
469|     %2 = amdg.async_copy_local_to_global %arg1, %arg2 : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32x!tt.ptr<f32>, #blocked>
470|     %3 = ttg.async_commit_group tokens %2
471| 
472|     // Wait on token %1: 2 store instructions outstanding
```
**EN:** This function-oriented block defines or enters `simple_local_to_global_waitcnt`. Within it, the test exercises tt.func, amdg.async_copy_local_to_global, async copy commit groups, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `simple_local_to_global_waitcnt` 为核心。测试在其中演示 tt.func、amdg.async_copy_local_to_global、异步拷贝提交组，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 473-475
```mlir
473|     // CHECK: amdg.async_wait {{.*}} {num_inst = 2
474|     %9 = ttg.async_wait %1 {num = 0 : i32}
475|     // Wait on token %3: 0 outstanding
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 476-481
```mlir
476|     // CHECK: amdg.async_wait {{.*}} {num_inst = 0
477|     %10 = ttg.async_wait %3 {num = 0 : i32}
478|     tt.return
479|   }
480| }
481| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on async wait synchronization, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 异步等待同步、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 482-482
```mlir
482| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 483-486
```mlir
483| 
484| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
485| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
486| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 487-487
```mlir
487| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 488-488
```mlir
488|   // CHECK-LABEL: bf16_local_to_global_split_waitcnt
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: bf16_local_to_global_split_waitcnt anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: bf16_local_to_global_split_waitcnt 这样的标签用于锚定匹配范围。

### Lines 489-496
```mlir
489|   tt.func public @bf16_local_to_global_split_waitcnt(%arg1: !ttg.memdesc<4x32xbf16, #shared, #smem, mutable>, %arg2: tensor<4x32x!tt.ptr<bf16>, #blocked>) {
490|     // Emits 2 async store intrinsics (16-bit store split into two 8-bit stores)
491|     %0 = amdg.async_copy_local_to_global %arg1, %arg2 : !ttg.memdesc<4x32xbf16, #shared, #smem, mutable> -> tensor<4x32x!tt.ptr<bf16>, #blocked>
492|     %1 = ttg.async_commit_group tokens %0
493|     // Emits 2 async store intrinsics
494|     %2 = amdg.async_copy_local_to_global %arg1, %arg2 : !ttg.memdesc<4x32xbf16, #shared, #smem, mutable> -> tensor<4x32x!tt.ptr<bf16>, #blocked>
495|     %3 = ttg.async_commit_group tokens %2
496| 
```
**EN:** This function-oriented block defines or enters `bf16_local_to_global_split_waitcnt`. Within it, the test exercises tt.func, amdg.async_copy_local_to_global, async copy commit groups, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `bf16_local_to_global_split_waitcnt` 为核心。测试在其中演示 tt.func、amdg.async_copy_local_to_global、异步拷贝提交组，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 497-498
```mlir
497|     // CHECK: amdg.async_wait {{.*}} {num_inst = 2
498|     %9 = ttg.async_wait %1 {num = 0 : i32}
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 499-504
```mlir
499|     // CHECK: amdg.async_wait {{.*}} {num_inst = 0
500|     %10 = ttg.async_wait %3 {num = 0 : i32}
501|     tt.return
502|   }
503| }
504| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on async wait synchronization, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 异步等待同步、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 505-505
```mlir
505| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 506-508
```mlir
506| 
507| // Test mixing async_copy_global_to_local and async_copy_local_to_global on GFX1250
508| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 509-511
```mlir
509| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
510| #shared = #ttg.swizzled_shared<{vec = 8, perPhase = 8, maxPhase = 2, order = [1, 0]}>
511| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 512-512
```mlir
512| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 513-513
```mlir
513|   // CHECK-LABEL: mix_global_to_local_and_local_to_global
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: mix_global_to_local_and_local_to_global anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: mix_global_to_local_and_local_to_global 这样的标签用于锚定匹配范围。

### Lines 514-522
```mlir
514|   tt.func public @mix_global_to_local_and_local_to_global(%arg1: !ttg.memdesc<32x32xf32, #shared, #smem, mutable>, %arg2: tensor<32x32x!tt.ptr<f32>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>}) {
515|     // Emits 2 async load instructions
516|     %0 = ttg.async_copy_global_to_local %arg2, %arg1 : tensor<32x32x!tt.ptr<f32>, #blocked> -> <32x32xf32, #shared, #smem, mutable>
517|     %1 = ttg.async_commit_group tokens %0
518|     // Emits 2 async store instructions
519|     %2 = amdg.async_copy_local_to_global %arg1, %arg2 : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32x!tt.ptr<f32>, #blocked>
520|     %3 = ttg.async_commit_group tokens %2
521| 
522|     // Wait on token %1: 2 store instructions outstanding
```
**EN:** This function-oriented block defines or enters `mix_global_to_local_and_local_to_global`. Within it, the test exercises tt.func, async copy commit groups, ttg.async_copy_global_to_local, amdg.async_copy_local_to_global, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mix_global_to_local_and_local_to_global` 为核心。测试在其中演示 tt.func、异步拷贝提交组、ttg.async_copy_global_to_local、amdg.async_copy_local_to_global，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 523-525
```mlir
523|     // CHECK: amdg.async_wait {{.*}} {num_inst = 2
524|     %9 = ttg.async_wait %1 {num = 0 : i32}
525|     // Wait on token %3: 0 outstanding
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 526-531
```mlir
526|     // CHECK: amdg.async_wait {{.*}} {num_inst = 0
527|     %10 = ttg.async_wait %3 {num = 0 : i32}
528|     tt.return
529|   }
530| }
531| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on async wait synchronization, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 异步等待同步、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 532-532
```mlir
532| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 533-535
```mlir
533| 
534| // Test mixing async_copy and async_tdm_copy
535| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 536-538
```mlir
536| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
537| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
538| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 539-539
```mlir
539| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 540-540
```mlir
540|   // CHECK-LABEL: mix_async_copy_and_async_tdm_copy
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: mix_async_copy_and_async_tdm_copy anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: mix_async_copy_and_async_tdm_copy 这样的标签用于锚定匹配范围。

### Lines 541-556
```mlir
541|   tt.func public @mix_async_copy_and_async_tdm_copy(%memDesc: !ttg.memdesc<128x8xf16, #shared, #smem, mutable>, %tensorDesc: !tt.tensordesc<128x8xf16>, %mask: i32, %ptr: tensor<128x8x!tt.ptr<f16>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>}
542|   ) {
543|     %c0_i32 = arith.constant 0 : i32
544| 
545|     // Each async_tdm_copy only emits a single instruction (-> counts 1)
546|     %1 = amdg.async_tdm_copy_global_to_local %tensorDesc[%c0_i32, %c0_i32] into %memDesc, pred = %mask : !tt.tensordesc<128x8xf16> -> !ttg.memdesc<128x8xf16, #shared, #smem, mutable>
547| 
548|     %2 = ttg.async_copy_global_to_local %ptr, %memDesc : tensor<128x8x!tt.ptr<f16>, #blocked> -> <128x8xf16, #shared, #smem, mutable>
549|     %21 = ttg.async_commit_group tokens %2
550| 
551|     %3 = amdg.async_tdm_copy_global_to_local %tensorDesc[%c0_i32, %c0_i32] into %memDesc, pred = %mask : !tt.tensordesc<128x8xf16> -> !ttg.memdesc<128x8xf16, #shared, #smem, mutable>
552| 
553|     %4 = ttg.async_copy_global_to_local %ptr, %memDesc : tensor<128x8x!tt.ptr<f16>, #blocked> -> <128x8xf16, #shared, #smem, mutable>
554|     %5 = ttg.async_copy_global_to_local %ptr, %memDesc : tensor<128x8x!tt.ptr<f16>, #blocked> -> <128x8xf16, #shared, #smem, mutable>
555|     %51 = ttg.async_commit_group tokens %4, %5
556| 
```
**EN:** This function-oriented block defines or enters `mix_async_copy_and_async_tdm_copy`. Within it, the test exercises ttg.async_copy_global_to_local, tt.func, amdg.async_tdm_copy_global_to_local, async copy commit groups, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mix_async_copy_and_async_tdm_copy` 为核心。测试在其中演示 ttg.async_copy_global_to_local、tt.func、amdg.async_tdm_copy_global_to_local、异步拷贝提交组、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 557-560
```mlir
557|     // CHECK: amdg.async_tdm_intrinsic_wait {{.*}} {count = 1
558|     %tw1 = amdg.async_tdm_wait %1 {num = 0 : i32}
559| 
560|     // Wait on token %21: 2 async_copy instructions outstanding
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 561-563
```mlir
561|     // CHECK: amdg.async_wait {{.*}} {num_inst = 2
562|     %cw1 = ttg.async_wait %21 {num = 0 : i32}
563| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 564-566
```mlir
564|     // CHECK: amdg.async_tdm_intrinsic_wait {{.*}} {count = 0
565|     %w2 = amdg.async_tdm_wait %3 {num = 0 : i32}
566| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 567-572
```mlir
567|     // CHECK: amdg.async_wait {{.*}} {num_inst = 0
568|     %cw2 = ttg.async_wait %51 {num = 0 : i32}
569|     tt.return
570|   }
571| }
572| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on async wait synchronization, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 异步等待同步、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 573-573
```mlir
573| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 574-576
```mlir
574| 
575| // Test scf.if without else region in def chain
576| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 577-579
```mlir
577| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [32, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
578| #shared = #ttg.swizzled_shared<{vec = 8, perPhase = 8, maxPhase = 2, order = [1, 0]}>
579| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 580-580
```mlir
580| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 581-581
```mlir
581|   // CHECK-LABEL: scf_if_without_else
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: scf_if_without_else anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: scf_if_without_else 这样的标签用于锚定匹配范围。

### Lines 582-595
```mlir
582|   tt.func public @scf_if_without_else(%arg1: !ttg.memdesc<128x16xf16, #shared, #smem, mutable>, %arg3: tensor<128x16x!tt.ptr<f16>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>}, %cond: i1) {
583|     // Emits 1 direct to lds instruction
584|     %0 = ttg.async_copy_global_to_local %arg3, %arg1 : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
585|     %1 = ttg.async_commit_group tokens %0
586| 
587|     // For scf.if without else region, the else path contributes 0 instructions;
588|     // so the minimum across both paths is 0.
589|     scf.if %cond {
590|       // Emits 1 direct to lds instruction inside the if
591|       %inner = ttg.async_copy_global_to_local %arg3, %arg1 : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
592|       %inner_commit = ttg.async_commit_group tokens %inner
593|     }
594| 
595|     // Wait on token %1: if path contributes 1, else (skip) contributes 0 → min = 0
```
**EN:** This function-oriented block defines or enters `scf_if_without_else`. Within it, the test exercises tt.func, ttg.async_copy_global_to_local, async copy commit groups, structured conditionals, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `scf_if_without_else` 为核心。测试在其中演示 tt.func、ttg.async_copy_global_to_local、异步拷贝提交组、结构化条件分支，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 596-601
```mlir
596|     // CHECK: amdg.async_wait {{.*}} {num_inst = 0
597|     %10 = ttg.async_wait %1 {num = 0 : i32}
598|     tt.return
599|   }
600| }
601| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on async wait synchronization, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 异步等待同步、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 602-602
```mlir
602| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 603-608
```mlir
603| 
604| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
605| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
606| #smem = #ttg.shared_memory
607| #idx_i32_parent = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [0, 1]}>
608| #idx_i16_parent = #ttg.blocked<{sizePerThread = [1, 64], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [0, 1]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 609-609
```mlir
609| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 610-610
```mlir
610|   // CHECK-LABEL: tdm_gather_scatter_multiple_instructions
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tdm_gather_scatter_multiple_instructions anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tdm_gather_scatter_multiple_instructions 这样的标签用于锚定匹配范围。

### Lines 611-628
```mlir
611|   tt.func public @tdm_gather_scatter_multiple_instructions(
612|     %memDesc: !ttg.memdesc<256x128xf16, #shared, #smem, mutable>,
613|     %tensorDesc: !tt.tensordesc<64x128xf16>,
614|     %row_indices_i32: tensor<64xi32, #ttg.slice<{dim = 0, parent = #idx_i32_parent}>>,
615|     %row_indices_i16: tensor<256xi16, #ttg.slice<{dim = 0, parent = #idx_i16_parent}>>,
616|     %pred: i32
617|   ) {
618|     %c0_i32 = arith.constant 0 : i32
619| 
620|     // Gather with i32 indices: sizePerThread=16, 4 warps, maxPerInstr=8 => 2 instructions
621|     %token1 = amdg.async_tdm_gather %tensorDesc[%row_indices_i32, %c0_i32] to %memDesc, pred = %pred : tensor<64xi32, #ttg.slice<{dim = 0, parent = #idx_i32_parent}>>, !ttg.memdesc<256x128xf16, #shared, #smem, mutable> -> !tt.tensordesc<64x128xf16>
622|     // Scatter with i32 indices: 2 instructions
623|     %token2 = amdg.async_tdm_scatter %tensorDesc[%row_indices_i32, %c0_i32] from %memDesc : tensor<64xi32, #ttg.slice<{dim = 0, parent = #idx_i32_parent}>>, !ttg.memdesc<256x128xf16, #shared, #smem, mutable> -> !tt.tensordesc<64x128xf16>
624|     // Gather with i16 indices: sizePerThread=64, 4 warps, maxPerInstr=16 => 4 instructions
625|     %token3 = amdg.async_tdm_gather %tensorDesc[%row_indices_i16, %c0_i32] to %memDesc, pred = %pred : tensor<256xi16, #ttg.slice<{dim = 0, parent = #idx_i16_parent}>>, !ttg.memdesc<256x128xf16, #shared, #smem, mutable> -> !tt.tensordesc<64x128xf16>
626|     // Scatter with i16 indices: 4 instructions
627|     %token4 = amdg.async_tdm_scatter %tensorDesc[%row_indices_i16, %c0_i32] from %memDesc : tensor<256xi16, #ttg.slice<{dim = 0, parent = #idx_i16_parent}>>, !ttg.memdesc<256x128xf16, #shared, #smem, mutable> -> !tt.tensordesc<64x128xf16>
628| 
```
**EN:** This function-oriented block defines or enters `tdm_gather_scatter_multiple_instructions`. Within it, the test exercises tt.func, amdg.async_tdm_gather, amdg.async_tdm_scatter, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tdm_gather_scatter_multiple_instructions` 为核心。测试在其中演示 tt.func、amdg.async_tdm_gather、amdg.async_tdm_scatter、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 629-630
```mlir
629|     // CHECK: amdg.async_tdm_intrinsic_wait {{.*}} {count = 0
630|     %w1 = amdg.async_tdm_wait %token4 {num = 0 : i32}
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 631-632
```mlir
631|     // CHECK: amdg.async_tdm_intrinsic_wait {{.*}} {count = 4
632|     %w2 = amdg.async_tdm_wait %token3 {num = 0 : i32}
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 633-634
```mlir
633|     // CHECK: amdg.async_tdm_intrinsic_wait {{.*}} {count = 8
634|     %w3 = amdg.async_tdm_wait %token2 {num = 0 : i32}
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 635-641
```mlir
635|     // CHECK: amdg.async_tdm_intrinsic_wait {{.*}} {count = 10
636|     %w4 = amdg.async_tdm_wait %token1 {num = 0 : i32}
637| 
638|     tt.return
639|   }
640| }
641| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.async_tdm_wait, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.async_tdm_wait、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 642-642
```mlir
642| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 643-647
```mlir
643| 
644| // Test bug fix: With warp = [[0], [0]] the warp dimension has free variables,
645| // so the load should contribute 0 instructions — non-canonical warps skip the
646| // load entirely.
647| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 648-650
```mlir
648| #linear_warp_free = #ttg.linear<{register = [[0]], lane = [[1], [2], [4], [8], [16], [32]], warp = [[0], [0]], block = []}>
649| #shared_simple = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
650| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 651-651
```mlir
651| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 652-652
```mlir
652|   // CHECK-LABEL: warp_free_variable_returns_zero
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: warp_free_variable_returns_zero anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: warp_free_variable_returns_zero 这样的标签用于锚定匹配范围。

### Lines 653-660
```mlir
653|   tt.func public @warp_free_variable_returns_zero(
654|       %ptr: !tt.ptr<i32> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
655|       %offsets: tensor<64xi32, #linear_warp_free>,
656|       %dest: !ttg.memdesc<64xi32, #shared_simple, #smem, mutable>) {
657|     %0 = amdg.buffer_load_to_local %ptr[%offsets] into %dest : <i32>[tensor<64xi32, #linear_warp_free>]  -> <64xi32, #shared_simple, #smem, mutable>
658|     %1 = ttg.async_commit_group
659| 
660|     // Warp free variable means 0 instructions emitted for this warp config
```
**EN:** This function-oriented block defines or enters `warp_free_variable_returns_zero`. Within it, the test exercises tt.func, amdg.buffer_load_to_local, async copy commit groups, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `warp_free_variable_returns_zero` 为核心。测试在其中演示 tt.func、amdg.buffer_load_to_local、异步拷贝提交组，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 661-666
```mlir
661|     // CHECK: amdg.async_wait {num_inst = 0
662|     %2 = ttg.async_wait {num = 1 : i32}
663|     tt.return
664|   }
665| }
666| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on async wait synchronization, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 异步等待同步、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 667-667
```mlir
667| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 668-670
```mlir
668| 
669| // Test bug fix: register zero bases should not inflate instruction count.
670| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 671-673
```mlir
671| #linear_reg_zero = #ttg.linear<{register = [[0]], lane = [[1], [2], [4], [8], [16], [32]], warp = [[64], [128]], block = []}>
672| #shared_simple2 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
673| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 674-674
```mlir
674| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 675-675
```mlir
675|   // CHECK-LABEL: register_zero_bases_not_inflated
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: register_zero_bases_not_inflated anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: register_zero_bases_not_inflated 这样的标签用于锚定匹配范围。

### Lines 676-682
```mlir
676|   tt.func public @register_zero_bases_not_inflated(
677|       %ptr: !tt.ptr<i32> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
678|       %offsets: tensor<256xi32, #linear_reg_zero>,
679|       %dest: !ttg.memdesc<256xi32, #shared_simple2, #smem, mutable>) {
680|     %0 = amdg.buffer_load_to_local %ptr[%offsets] into %dest : <i32>[tensor<256xi32, #linear_reg_zero>]  -> <256xi32, #shared_simple2, #smem, mutable>
681|     %1 = ttg.async_commit_group
682|     // Register zero bases should not inflate count: 1 instruction
```
**EN:** This function-oriented block defines or enters `register_zero_bases_not_inflated`. Within it, the test exercises tt.func, amdg.buffer_load_to_local, async copy commit groups, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `register_zero_bases_not_inflated` 为核心。测试在其中演示 tt.func、amdg.buffer_load_to_local、异步拷贝提交组，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 683-688
```mlir
683|     // CHECK: amdg.async_wait {num_inst = 1
684|     %2 = ttg.async_wait {num = 1 : i32}
685|     tt.return
686|   }
687| }
688| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on async wait synchronization, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 异步等待同步、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 689-689
```mlir
689| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 690-692
```mlir
690| 
691| // Test TDM with partitioned shared encoding where each copy emits multiple instructions
692| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 693-696
```mlir
693| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
694| #shared_inner = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
695| #partitioned = #ttg.partitioned_shared<{numPartitions = 2, numGroups = 4, partitionDim = 0, partitionLayout = #shared_inner}>
696| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 697-697
```mlir
697| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 698-698
```mlir
698|   // CHECK-LABEL: tdm_partitioned_shared_waitcnt
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tdm_partitioned_shared_waitcnt anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tdm_partitioned_shared_waitcnt 这样的标签用于锚定匹配范围。

### Lines 699-712
```mlir
699|   tt.func public @tdm_partitioned_shared_waitcnt(
700|     %memDesc: !ttg.memdesc<128x16xf16, #partitioned, #smem, mutable>,
701|     %tensorDesc: !tt.tensordesc<128x16xf16>,
702|     %mask: i32
703|   ) {
704|     %c0_i32 = arith.constant 0 : i32
705| 
706|     // numLogicalPieces = numPartitions * numGroups = 2 * 4 = 8
707|     // warpsAlongPartition = gcd(numWarps=4, numLogicalPieces=8) = 4
708|     // Each async_tdm_copy emits divideCeil(8, 4) = 2 instructions
709|     %1 = amdg.async_tdm_copy_global_to_local %tensorDesc[%c0_i32, %c0_i32] into %memDesc, pred = %mask : !tt.tensordesc<128x16xf16> -> !ttg.memdesc<128x16xf16, #partitioned, #smem, mutable>
710|     %2 = amdg.async_tdm_copy_global_to_local %tensorDesc[%c0_i32, %c0_i32] into %memDesc, pred = %mask : !tt.tensordesc<128x16xf16> -> !ttg.memdesc<128x16xf16, #partitioned, #smem, mutable>
711| 
712|     // Skip second copy (2 instructions) => count = 2
```
**EN:** This function-oriented block defines or enters `tdm_partitioned_shared_waitcnt`. Within it, the test exercises tt.func, amdg.async_tdm_copy_global_to_local, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tdm_partitioned_shared_waitcnt` 为核心。测试在其中演示 tt.func、amdg.async_tdm_copy_global_to_local、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 713-715
```mlir
713|     // CHECK: amdg.async_tdm_intrinsic_wait {{.*}} {count = 2
714|     %w1 = amdg.async_tdm_wait %1 {num = 0 : i32}
715|     // Nothing in between => count = 0
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 716-720
```mlir
716|     // CHECK: amdg.async_tdm_intrinsic_wait {{.*}} {count = 0
717|     %w2 = amdg.async_tdm_wait %2 {num = 0 : i32}
718|     tt.return
719|   }
720| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.async_tdm_wait, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.async_tdm_wait、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `--tritonamdgpu-update-async-wait-count=gfx-arch=gfx1250`
- **CN:** 主要 pass 选项：`-split-input-file`，`--tritonamdgpu-update-async-wait-count=gfx-arch=gfx1250`
- **EN:** Dominant operations include `ttg.async_commit_group`, `ttg.async_copy_global_to_local`, `tt.func`, `ttg.async_wait`, `module`, `tt.return`, `arith.constant`, `scf.yield`, `amdg.async_tdm_wait`, `scf.for`.
- **CN:** 主要操作包括 `ttg.async_commit_group`、`ttg.async_copy_global_to_local`、`tt.func`、`ttg.async_wait`、`module`、`tt.return`、`arith.constant`、`scf.yield`、`amdg.async_tdm_wait`、`scf.for`。
- **EN:** The file contains 21 independently testable section(s). Check styles used: CHECK x46, CHECK-LABEL x22. Important labels include simple_waitcnt, simple_buffer_load_to_local_waitcnt, simple_waitcnt_reversed, simple_waitcnt_with_tt_load. It validates that the selected pass pipeline rewrites each labeled test case into the checked form.
- **CN:** 该文件包含 21 个可独立测试的分段。使用的检查类型：CHECK ×46，CHECK-LABEL ×22。 关键标签包括 simple_waitcnt，simple_buffer_load_to_local_waitcnt，simple_waitcnt_reversed，simple_waitcnt_with_tt_load。 它验证所选 pass 流水线会把每个带标签的测试用例重写成检查所要求的形式。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `ttg`: TritonGPU layout, memory, and GPU-specific operations.
- **CN:** `ttg`：TritonGPU 布局、内存和 GPU 专用操作。
- **EN:** `tt`: Triton high-level tensor/pointer operations.
- **CN:** `tt`：Triton 高层张量/指针操作。
- **EN:** `scf`: Structured control flow for loops and conditionals.
- **CN:** `scf`：用于循环与条件分支的结构化控制流。
- **EN:** `arith`: MLIR arithmetic dialect for scalar/tensor math.
- **CN:** `arith`：用于标量/张量运算的 MLIR arithmetic 方言。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。