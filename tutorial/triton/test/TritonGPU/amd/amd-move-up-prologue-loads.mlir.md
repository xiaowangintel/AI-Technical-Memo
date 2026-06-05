# amd-move-up-prologue-loads.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/amd/amd-move-up-prologue-loads.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-tritonamdgpu-move-up-prologue-loads` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-tritonamdgpu-move-up-prologue-loads` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file -tritonamdgpu-move-up-prologue-loads | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file -tritonamdgpu-move-up-prologue-loads | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt %s -split-input-file -tritonamdgpu-move-up-prologue-loads | FileCheck %s
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file -tritonamdgpu-move-up-prologue-loads | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file -tritonamdgpu-move-up-prologue-loads | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-7
```mlir
3| // CHECK-LABEL: move_up_slice
4| // CHECK: arith.cmpi
5| // CHECK: tt.splat
6| // CHECK: tt.load
7| // CHECK: ttg.local_alloc
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: move_up_slice anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: move_up_slice 这样的标签用于锚定匹配范围。

### Lines 8-10
```mlir
 8| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
 9| #shared = #ttg.swizzled_shared<{vec = 8, perPhase = 1, maxPhase = 16, order = [1, 0]}>
10| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 11-11
```mlir
11| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 12-21
```mlir
12|   tt.func @move_up_slice(%arg0: tensor<32x128x!tt.ptr<f16>, #blocked>, %arg1: i32) {
13|     %c0_i32 = arith.constant 0 : i32
14|     %0 = ttg.local_alloc : () -> !ttg.memdesc<1x32x128xf16, #shared, #smem, mutable>
15|     %1 = arith.cmpi sgt, %arg1, %c0_i32 : i32
16|     %2 = tt.splat %1 : i1 -> tensor<32x128xi1, #blocked>
17|     %3 = tt.load %arg0, %2 {amd.pipeliner_part = "prologue"} : tensor<32x128x!tt.ptr<f16>, #blocked>
18|     tt.return
19|   }
20| }
21| 
```
**EN:** This function-oriented block defines or enters `move_up_slice`. Within it, the test exercises tt.func, constants, shared/local memory allocation, integer comparisons, broadcasted scalars or pointers, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `move_up_slice` 为核心。测试在其中演示 tt.func、常量、共享/本地内存分配、整数比较、广播后的标量或指针，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 22-22
```mlir
22| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 23-32
```mlir
23| 
24| // CHECK-LABEL: keep_load_order
25| // CHECK: arith.cmpi sgt
26| // CHECK: tt.splat
27| // CHECK: tt.load %arg0
28| // CHECK: tt.addptr
29| // CHECK: arith.cmpi slt
30| // CHECK: tt.splat
31| // CHECK: tt.load
32| // CHECK: ttg.local_alloc
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: keep_load_order anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: keep_load_order 这样的标签用于锚定匹配范围。

### Lines 33-35
```mlir
33| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
34| #shared = #ttg.swizzled_shared<{vec = 8, perPhase = 1, maxPhase = 16, order = [1, 0]}>
35| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 36-36
```mlir
36| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 37-51
```mlir
37|   tt.func @keep_load_order(%arg0: tensor<32x128x!tt.ptr<f16>, #blocked>, %arg1: i32, %arg2: i32) {
38|     %c0_i32 = arith.constant 0 : i32
39|     %cst = arith.constant dense<128> : tensor<32x128xi32, #blocked>
40|     %0 = tt.addptr %arg0, %cst : tensor<32x128x!tt.ptr<f16>, #blocked>, tensor<32x128xi32, #blocked>
41|     %1 = ttg.local_alloc : () -> !ttg.memdesc<1x32x128xf16, #shared, #smem, mutable>
42|     %2 = arith.cmpi sgt, %arg1, %c0_i32 : i32
43|     %3 = tt.splat %2 : i1 -> tensor<32x128xi1, #blocked>
44|     %4 = tt.load %arg0, %3 {amd.pipeliner_part = "prologue"} : tensor<32x128x!tt.ptr<f16>, #blocked>
45|     %5 = arith.cmpi slt, %arg2, %c0_i32 : i32
46|     %6 = tt.splat %5 : i1 -> tensor<32x128xi1, #blocked>
47|     %7 = tt.load %0, %6 {amd.pipeliner_part = "prologue"} : tensor<32x128x!tt.ptr<f16>, #blocked>
48|     tt.return
49|   }
50| }
51| 
```
**EN:** This function-oriented block defines or enters `keep_load_order`. Within it, the test exercises tt.func, constants, integer comparisons, broadcasted scalars or pointers, masked or vectorized loads, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `keep_load_order` 为核心。测试在其中演示 tt.func、常量、整数比较、广播后的标量或指针、带掩码或向量化的加载，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 52-52
```mlir
52| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 53-59
```mlir
53| 
54| // CHECK-LABEL: break_at_atomic
55| // CHECK: tt.atomic_rmw
56| // CHECK: arith.cmpi
57| // CHECK: tt.splat
58| // CHECK: tt.load
59| // CHECK: ttg.local_alloc
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: break_at_atomic anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: break_at_atomic 这样的标签用于锚定匹配范围。

### Lines 60-62
```mlir
60| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
61| #shared = #ttg.swizzled_shared<{vec = 8, perPhase = 1, maxPhase = 16, order = [1, 0]}>
62| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 63-63
```mlir
63| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 64-75
```mlir
64|   tt.func @break_at_atomic(%arg0: tensor<32x128x!tt.ptr<f16>, #blocked>, %arg1: i32, %arg2: !tt.ptr<i32>) {
65|     %c0_i32 = arith.constant 0 : i32
66|     %c1_i32 = arith.constant 1 : i32
67|     %0 = tt.atomic_rmw fadd, relaxed, gpu, %arg2, %c1_i32 : (!tt.ptr<i32>, i32) -> i32
68|     %1 = ttg.local_alloc : () -> !ttg.memdesc<1x32x128xf16, #shared, #smem, mutable>
69|     %2 = arith.cmpi sgt, %arg1, %c0_i32 : i32
70|     %3 = tt.splat %2 : i1 -> tensor<32x128xi1, #blocked>
71|     %4 = tt.load %arg0, %3 {amd.pipeliner_part = "prologue"} : tensor<32x128x!tt.ptr<f16>, #blocked>
72|     tt.return
73|   }
74| }
75| 
```
**EN:** This function-oriented block defines or enters `break_at_atomic`. Within it, the test exercises tt.func, constants, tt.atomic_rmw, shared/local memory allocation, integer comparisons, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `break_at_atomic` 为核心。测试在其中演示 tt.func、常量、tt.atomic_rmw、共享/本地内存分配、整数比较，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 76-76
```mlir
76| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 77-83
```mlir
77| 
78| // CHECK-LABEL: break_at_barrier
79| // CHECK: gpu.barrier
80| // CHECK: arith.cmpi
81| // CHECK: tt.splat
82| // CHECK: tt.load
83| // CHECK: ttg.local_alloc
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: break_at_barrier anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: break_at_barrier 这样的标签用于锚定匹配范围。

### Lines 84-86
```mlir
84| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
85| #shared = #ttg.swizzled_shared<{vec = 8, perPhase = 1, maxPhase = 16, order = [1, 0]}>
86| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 87-87
```mlir
87| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 88-98
```mlir
88|   tt.func @break_at_barrier(%arg0: tensor<32x128x!tt.ptr<f16>, #blocked>, %arg1: i32) {
89|     %c0_i32 = arith.constant 0 : i32
90|     gpu.barrier
91|     %0 = ttg.local_alloc : () -> !ttg.memdesc<1x32x128xf16, #shared, #smem, mutable>
92|     %1 = arith.cmpi sgt, %arg1, %c0_i32 : i32
93|     %2 = tt.splat %1 : i1 -> tensor<32x128xi1, #blocked>
94|     %3 = tt.load %arg0, %2 {amd.pipeliner_part = "prologue"} : tensor<32x128x!tt.ptr<f16>, #blocked>
95|     tt.return
96|   }
97| }
98| 
```
**EN:** This function-oriented block defines or enters `break_at_barrier`. Within it, the test exercises tt.func, constants, gpu.barrier, shared/local memory allocation, integer comparisons, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `break_at_barrier` 为核心。测试在其中演示 tt.func、常量、gpu.barrier、共享/本地内存分配、整数比较，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 99-99
```mlir
99| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 100-104
```mlir
100| 
101| // CHECK-LABEL: break_at_loop
102| // CHECK: scf.for
103| // CHECK: tt.load %arg0
104| // CHECK: ttg.local_alloc
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: break_at_loop anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: break_at_loop 这样的标签用于锚定匹配范围。

### Lines 105-107
```mlir
105| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
106| #shared = #ttg.swizzled_shared<{vec = 8, perPhase = 1, maxPhase = 16, order = [1, 0]}>
107| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 108-108
```mlir
108| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 109-119
```mlir
109|   tt.func @break_at_loop(%arg0: tensor<32x128x!tt.ptr<f16>, #blocked>, %arg1: i32) {
110|     %c0_i32 = arith.constant 0 : i32
111|     %c1_i32 = arith.constant 1 : i32
112|     scf.for %arg2 = %c0_i32 to %arg1 step %c1_i32  : i32 {
113|     }
114|     %0 = ttg.local_alloc : () -> !ttg.memdesc<1x32x128xf16, #shared, #smem, mutable>
115|     %1 = tt.load %arg0 {amd.pipeliner_part = "prologue"} : tensor<32x128x!tt.ptr<f16>, #blocked>
116|     tt.return
117|   }
118| }
119| 
```
**EN:** This function-oriented block defines or enters `break_at_loop`. Within it, the test exercises tt.func, constants, structured loops, shared/local memory allocation, masked or vectorized loads, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `break_at_loop` 为核心。测试在其中演示 tt.func、常量、结构化循环、共享/本地内存分配、带掩码或向量化的加载，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 120-120
```mlir
120| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 121-122
```mlir
121| 
122| // Negative test: load without amd.pipeliner_part attribute should not be moved
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 123-127
```mlir
123| // CHECK-LABEL: no_prologue_attribute
124| // CHECK: ttg.local_alloc
125| // CHECK: arith.cmpi
126| // CHECK: tt.splat
127| // CHECK: tt.load
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: no_prologue_attribute anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: no_prologue_attribute 这样的标签用于锚定匹配范围。

### Lines 128-130
```mlir
128| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
129| #shared = #ttg.swizzled_shared<{vec = 8, perPhase = 1, maxPhase = 16, order = [1, 0]}>
130| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 131-131
```mlir
131| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 132-140
```mlir
132|   tt.func @no_prologue_attribute(%arg0: tensor<32x128x!tt.ptr<f16>, #blocked>, %arg1: i32) {
133|     %c0_i32 = arith.constant 0 : i32
134|     %0 = ttg.local_alloc : () -> !ttg.memdesc<1x32x128xf16, #shared, #smem, mutable>
135|     %1 = arith.cmpi sgt, %arg1, %c0_i32 : i32
136|     %2 = tt.splat %1 : i1 -> tensor<32x128xi1, #blocked>
137|     %3 = tt.load %arg0, %2 : tensor<32x128x!tt.ptr<f16>, #blocked>
138|     tt.return
139|   }
140| }
```
**EN:** This function-oriented block defines or enters `no_prologue_attribute`. Within it, the test exercises tt.func, constants, shared/local memory allocation, integer comparisons, broadcasted scalars or pointers, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `no_prologue_attribute` 为核心。测试在其中演示 tt.func、常量、共享/本地内存分配、整数比较、广播后的标量或指针，这些操作构成了被优化、lower 或验证的主要数据流。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-tritonamdgpu-move-up-prologue-loads`
- **CN:** 主要 pass 选项：`-split-input-file`，`-tritonamdgpu-move-up-prologue-loads`
- **EN:** Dominant operations include `tt.func`, `arith.constant`, `tt.load`, `module`, `ttg.local_alloc`, `arith.cmpi`, `tt.splat`, `tt.return`, `tt.addptr`, `tt.atomic_rmw`.
- **CN:** 主要操作包括 `tt.func`、`arith.constant`、`tt.load`、`module`、`ttg.local_alloc`、`arith.cmpi`、`tt.splat`、`tt.return`、`tt.addptr`、`tt.atomic_rmw`。
- **EN:** The file contains 5 independently testable section(s). Check styles used: CHECK x29, CHECK-LABEL x6. Important labels include move_up_slice, keep_load_order, break_at_atomic, break_at_barrier. It validates that the selected pass pipeline rewrites each labeled test case into the checked form.
- **CN:** 该文件包含 5 个可独立测试的分段。使用的检查类型：CHECK ×29，CHECK-LABEL ×6。 关键标签包括 move_up_slice，keep_load_order，break_at_atomic，break_at_barrier。 它验证所选 pass 流水线会把每个带标签的测试用例重写成检查所要求的形式。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `tt`: Triton high-level tensor/pointer operations.
- **CN:** `tt`：Triton 高层张量/指针操作。
- **EN:** `ttg`: TritonGPU layout, memory, and GPU-specific operations.
- **CN:** `ttg`：TritonGPU 布局、内存和 GPU 专用操作。
- **EN:** `arith`: MLIR arithmetic dialect for scalar/tensor math.
- **CN:** `arith`：用于标量/张量运算的 MLIR arithmetic 方言。
- **EN:** `gpu`: GPU runtime or synchronization operations.
- **CN:** `gpu`：GPU 运行时或同步操作。
- **EN:** `scf`: Structured control flow for loops and conditionals.
- **CN:** `scf`：用于循环与条件分支的结构化控制流。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。