# amd-convert-buffer-ops-i64-offset.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/amd/amd-convert-buffer-ops-i64-offset.mlir`
- **EN:** Test for the MLIR/Triton pipeline `--tritonamdgpu-convert-buffer-ops="gfx-arch=gfx942"` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `--tritonamdgpu-convert-buffer-ops="gfx-arch=gfx942"` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file --tritonamdgpu-convert-buffer-ops="gfx-arch=gfx942" | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file --tritonamdgpu-convert-buffer-ops="gfx-arch=gfx942" | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-5
```mlir
1| // RUN: triton-opt %s -split-input-file --tritonamdgpu-convert-buffer-ops="gfx-arch=gfx942" | FileCheck %s
2| 
3| // Test that tt.load with i64 offsets derived from provably bounded non-negative
4| // expressions is converted to amdg.buffer_load with an arith.trunci from i64 to i32.
5| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file --tritonamdgpu-convert-buffer-ops="gfx-arch=gfx942" | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file --tritonamdgpu-convert-buffer-ops="gfx-arch=gfx942" | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 6-7
```mlir
6| #blocked = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [64], warpsPerCTA = [1], order = [0]}>
7| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 8-8
```mlir
8| // CHECK-LABEL: @load_i64_offset_bounded
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @load_i64_offset_bounded anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @load_i64_offset_bounded 这样的标签用于锚定匹配范围。

### Lines 9-9
```mlir
9| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 10-17
```mlir
10|   tt.func @load_i64_offset_bounded(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}) -> tensor<256xf32, #blocked> {
11|     %range = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #blocked>
12|     %range_ext = arith.extsi %range : tensor<256xi32, #blocked> to tensor<256xi64, #blocked>
13|     %c1024_i64 = arith.constant 1024 : i64
14|     %stride = tt.splat %c1024_i64 : i64 -> tensor<256xi64, #blocked>
15|     %offset = arith.muli %range_ext, %stride : tensor<256xi64, #blocked>
16|     %base = tt.splat %arg0 : !tt.ptr<f32> -> tensor<256x!tt.ptr<f32>, #blocked>
17|     %ptr = tt.addptr %base, %offset : tensor<256x!tt.ptr<f32>, #blocked>, tensor<256xi64, #blocked>
```
**EN:** This function-oriented block defines or enters `load_i64_offset_bounded`. Within it, the test exercises tt.func, broadcasted scalars or pointers, lane/block index ranges, arith.extsi, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `load_i64_offset_bounded` 为核心。测试在其中演示 tt.func、广播后的标量或指针、lane/block 索引范围、arith.extsi、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 18-26
```mlir
18|     // CHECK: arith.trunci
19|     // CHECK-SAME: tensor<256xi64,
20|     // CHECK-SAME: to tensor<256xi32,
21|     // CHECK: amdg.buffer_load
22|     %val = tt.load %ptr : tensor<256x!tt.ptr<f32>, #blocked>
23|     tt.return %val : tensor<256xf32, #blocked>
24|   }
25| }
26| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 27-27
```mlir
27| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 28-30
```mlir
28| 
29| // Test that i64 offset loads are NOT converted when the offset may be negative.
30| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 31-32
```mlir
31| #blocked1 = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [64], warpsPerCTA = [1], order = [0]}>
32| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 33-33
```mlir
33| // CHECK-LABEL: @load_i64_offset_possibly_negative
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @load_i64_offset_possibly_negative anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @load_i64_offset_possibly_negative 这样的标签用于锚定匹配范围。

### Lines 34-34
```mlir
34| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 35-38
```mlir
35|   tt.func @load_i64_offset_possibly_negative(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg1: i64) -> tensor<256xf32, #blocked1> {
36|     %splat_off = tt.splat %arg1 : i64 -> tensor<256xi64, #blocked1>
37|     %base = tt.splat %arg0 : !tt.ptr<f32> -> tensor<256x!tt.ptr<f32>, #blocked1>
38|     %ptr = tt.addptr %base, %splat_off : tensor<256x!tt.ptr<f32>, #blocked1>, tensor<256xi64, #blocked1>
```
**EN:** This function-oriented block defines or enters `load_i64_offset_possibly_negative`. Within it, the test exercises tt.func, broadcasted scalars or pointers, pointer arithmetic, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `load_i64_offset_possibly_negative` 为核心。测试在其中演示 tt.func、广播后的标量或指针、指针算术，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 39-45
```mlir
39|     // CHECK-NOT: amdg.buffer_load
40|     // CHECK: tt.load
41|     %val = tt.load %ptr : tensor<256x!tt.ptr<f32>, #blocked1>
42|     tt.return %val : tensor<256xf32, #blocked1>
43|   }
44| }
45| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 46-46
```mlir
46| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 47-49
```mlir
47| 
48| // Test that i64 offset stores are converted with trunci when offset is bounded.
49| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 50-51
```mlir
50| #blocked2 = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [64], warpsPerCTA = [1], order = [0]}>
51| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 52-52
```mlir
52| // CHECK-LABEL: @store_i64_offset_bounded
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @store_i64_offset_bounded anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @store_i64_offset_bounded 这样的标签用于锚定匹配范围。

### Lines 53-53
```mlir
53| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 54-61
```mlir
54|   tt.func @store_i64_offset_bounded(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %data: tensor<256xf32, #blocked2>) {
55|     %range = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #blocked2>
56|     %range_ext = arith.extsi %range : tensor<256xi32, #blocked2> to tensor<256xi64, #blocked2>
57|     %c512_i64 = arith.constant 512 : i64
58|     %stride = tt.splat %c512_i64 : i64 -> tensor<256xi64, #blocked2>
59|     %offset = arith.muli %range_ext, %stride : tensor<256xi64, #blocked2>
60|     %base = tt.splat %arg0 : !tt.ptr<f32> -> tensor<256x!tt.ptr<f32>, #blocked2>
61|     %ptr = tt.addptr %base, %offset : tensor<256x!tt.ptr<f32>, #blocked2>, tensor<256xi64, #blocked2>
```
**EN:** This function-oriented block defines or enters `store_i64_offset_bounded`. Within it, the test exercises tt.func, broadcasted scalars or pointers, lane/block index ranges, arith.extsi, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `store_i64_offset_bounded` 为核心。测试在其中演示 tt.func、广播后的标量或指针、lane/block 索引范围、arith.extsi、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 62-70
```mlir
62|     // CHECK: arith.trunci
63|     // CHECK-SAME: tensor<256xi64,
64|     // CHECK-SAME: to tensor<256xi32,
65|     // CHECK: amdg.buffer_store
66|     tt.store %ptr, %data : tensor<256x!tt.ptr<f32>, #blocked2>
67|     tt.return
68|   }
69| }
70| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized stores, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的存储、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 71-71
```mlir
71| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 72-74
```mlir
72| 
73| // Test that i64 offset loads with tt.pointer_range=32 attribute are converted.
74| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 75-76
```mlir
75| #blocked3 = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [64], warpsPerCTA = [1], order = [0]}>
76| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 77-77
```mlir
77| // CHECK-LABEL: @load_i64_offset_pointer_range_32
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @load_i64_offset_pointer_range_32 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @load_i64_offset_pointer_range_32 这样的标签用于锚定匹配范围。

### Lines 78-78
```mlir
78| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 79-82
```mlir
79|   tt.func @load_i64_offset_pointer_range_32(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg1: i64) -> tensor<256xf32, #blocked3> {
80|     %splat_off = tt.splat %arg1 : i64 -> tensor<256xi64, #blocked3>
81|     %base = tt.splat %arg0 : !tt.ptr<f32> -> tensor<256x!tt.ptr<f32>, #blocked3>
82|     %ptr = tt.addptr %base, %splat_off : tensor<256x!tt.ptr<f32>, #blocked3>, tensor<256xi64, #blocked3>
```
**EN:** This function-oriented block defines or enters `load_i64_offset_pointer_range_32`. Within it, the test exercises tt.func, broadcasted scalars or pointers, pointer arithmetic, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `load_i64_offset_pointer_range_32` 为核心。测试在其中演示 tt.func、广播后的标量或指针、指针算术，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 83-91
```mlir
83|     // CHECK: arith.trunci
84|     // CHECK-SAME: tensor<256xi64,
85|     // CHECK-SAME: to tensor<256xi32,
86|     // CHECK: amdg.buffer_load
87|     %val = tt.load %ptr : tensor<256x!tt.ptr<f32>, #blocked3>
88|     tt.return %val : tensor<256xf32, #blocked3>
89|   }
90| }
91| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 92-92
```mlir
92| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 93-96
```mlir
93| 
94| // Test that multiple loads sharing the same tt.addptr with i64 offset are both
95| // converted without SSA dominance violations (regression test for #9907).
96| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 97-98
```mlir
97| #blocked4 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 8], warpsPerCTA = [2, 1], order = [1, 0]}>
98| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 99-99
```mlir
99| // CHECK-LABEL: @multi_load_shared_addptr_i64
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @multi_load_shared_addptr_i64 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @multi_load_shared_addptr_i64 这样的标签用于锚定匹配范围。

### Lines 100-100
```mlir
100| module attributes {"ttg.num-warps" = 2 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 101-109
```mlir
101|   tt.func @multi_load_shared_addptr_i64(%arg0: !tt.ptr<f32>, %arg1: !tt.ptr<f32>) {
102|     %r = tt.make_range {end = 8 : i32, start = 0 : i32} : tensor<8xi32, #ttg.slice<{dim = 0, parent = #blocked4}>>
103|     %ext = arith.extsi %r : tensor<8xi32, #ttg.slice<{dim = 0, parent = #blocked4}>> to tensor<8xi64, #ttg.slice<{dim = 0, parent = #blocked4}>>
104|     %offset = tt.expand_dims %ext {axis = 0 : i32} : tensor<8xi64, #ttg.slice<{dim = 0, parent = #blocked4}>> -> tensor<1x8xi64, #blocked4>
105|     %base = tt.splat %arg0 : !tt.ptr<f32> -> tensor<1x8x!tt.ptr<f32>, #blocked4>
106|     %ptr = tt.addptr %base, %offset : tensor<1x8x!tt.ptr<f32>, #blocked4>, tensor<1x8xi64, #blocked4>
107|     %v1 = tt.load %ptr : tensor<1x8x!tt.ptr<f32>, #blocked4>
108|     %v2 = tt.load %ptr : tensor<1x8x!tt.ptr<f32>, #blocked4>
109|     // Each trunci is inserted right before its corresponding load.
```
**EN:** This function-oriented block defines or enters `multi_load_shared_addptr_i64`. Within it, the test exercises tt.func, masked or vectorized loads, lane/block index ranges, arith.extsi, shape expansion, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `multi_load_shared_addptr_i64` 为核心。测试在其中演示 tt.func、带掩码或向量化的加载、lane/block 索引范围、arith.extsi、形状扩展，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 110-121
```mlir
110|     // CHECK: arith.trunci {{.*}} tensor<1x8xi64, {{.*}}> to tensor<1x8xi32, {{.*}}>
111|     // CHECK: amdg.buffer_load
112|     // CHECK: arith.trunci {{.*}} tensor<1x8xi64, {{.*}}> to tensor<1x8xi32, {{.*}}>
113|     // CHECK: amdg.buffer_load
114|     // CHECK-NOT: tt.load
115|     %sum = arith.addf %v1, %v2 : tensor<1x8xf32, #blocked4>
116|     %st = tt.splat %arg1 : !tt.ptr<f32> -> tensor<1x8x!tt.ptr<f32>, #blocked4>
117|     tt.store %st, %sum : tensor<1x8x!tt.ptr<f32>, #blocked4>
118|     tt.return
119|   }
120| }
121| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on floating-point additions, broadcasted scalars or pointers, masked or vectorized stores, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 浮点加法、广播后的标量或指针、带掩码或向量化的存储、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 122-122
```mlir
122| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 123-127
```mlir
123| 
124| // Test that an atomic RMW with i64 offset and unsupported type (i8) is NOT
125| // converted. canUseBufferOps is pure, so no stale trunci leaks into the IR
126| // when the pattern bails on the type check.
127| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 128-129
```mlir
128| #blocked5 = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [64], warpsPerCTA = [1], order = [0]}>
129| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 130-130
```mlir
130| // CHECK-LABEL: @atomic_rmw_i64_offset_unsupported_type
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @atomic_rmw_i64_offset_unsupported_type anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @atomic_rmw_i64_offset_unsupported_type 这样的标签用于锚定匹配范围。

### Lines 131-131
```mlir
131| module attributes {"ttg.num-warps" = 1 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 132-141
```mlir
132|   tt.func @atomic_rmw_i64_offset_unsupported_type(%arg0: !tt.ptr<i8> {tt.divisibility = 16 : i32}) {
133|     %range = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #blocked5>
134|     %range_ext = arith.extsi %range : tensor<256xi32, #blocked5> to tensor<256xi64, #blocked5>
135|     %c1024_i64 = arith.constant 1024 : i64
136|     %stride = tt.splat %c1024_i64 : i64 -> tensor<256xi64, #blocked5>
137|     %offset = arith.muli %range_ext, %stride : tensor<256xi64, #blocked5>
138|     %base = tt.splat %arg0 : !tt.ptr<i8> -> tensor<256x!tt.ptr<i8>, #blocked5>
139|     %ptr = tt.addptr %base, %offset : tensor<256x!tt.ptr<i8>, #blocked5>, tensor<256xi64, #blocked5>
140|     %val = arith.constant dense<1> : tensor<256xi8, #blocked5>
141|     %result = tt.atomic_rmw add, relaxed, gpu, %ptr, %val : (tensor<256x!tt.ptr<i8>, #blocked5>, tensor<256xi8, #blocked5>) -> tensor<256xi8, #blocked5>
```
**EN:** This function-oriented block defines or enters `atomic_rmw_i64_offset_unsupported_type`. Within it, the test exercises tt.func, constants, broadcasted scalars or pointers, lane/block index ranges, arith.extsi, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `atomic_rmw_i64_offset_unsupported_type` 为核心。测试在其中演示 tt.func、常量、广播后的标量或指针、lane/block 索引范围、arith.extsi，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 142-148
```mlir
142|     // CHECK-NOT: arith.trunci
143|     // CHECK-NOT: amdg.buffer_atomic_rmw
144|     // CHECK: tt.atomic_rmw
145|     tt.return
146|   }
147| }
148| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 149-149
```mlir
149| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 150-154
```mlir
150| 
151| // 2D offset = row * stride + col with i64 stride; extracted block stride is the
152| // scalar splat source (i64) and must be truncated to i32 for amdg.buffer_load
153| // operand #2 (regression: verifier expected i32, got i64).
154| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 155-156
```mlir
155| #blocked6 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [8, 1], order = [1, 0]}>
156| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 157-157
```mlir
157| // CHECK-LABEL: @stride_i64_minimal
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @stride_i64_minimal anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @stride_i64_minimal 这样的标签用于锚定匹配范围。

### Lines 158-158
```mlir
158| module attributes {"ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 159-171
```mlir
159|   tt.func @stride_i64_minimal(
160|     %ptr: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
161|     %stride: i64,
162|     %row: tensor<256x1xi64, #blocked6>,
163|     %col: tensor<1x64xi64, #blocked6>
164|   ) -> tensor<256x64xf16, #blocked6> {
165|     %s = tt.splat %stride : i64 -> tensor<256x1xi64, #blocked6>
166|     %mul = arith.muli %row, %s : tensor<256x1xi64, #blocked6>
167|     %bc0 = tt.broadcast %mul : tensor<256x1xi64, #blocked6> -> tensor<256x64xi64, #blocked6>
168|     %bc1 = tt.broadcast %col : tensor<1x64xi64, #blocked6> -> tensor<256x64xi64, #blocked6>
169|     %off = arith.addi %bc1, %bc0 : tensor<256x64xi64, #blocked6>
170|     %base = tt.splat %ptr : !tt.ptr<f16> -> tensor<256x64x!tt.ptr<f16>, #blocked6>
171|     %p = tt.addptr %base, %off : tensor<256x64x!tt.ptr<f16>, #blocked6>, tensor<256x64xi64, #blocked6>
```
**EN:** This function-oriented block defines or enters `stride_i64_minimal`. Within it, the test exercises tt.func, broadcasted scalars or pointers, tensor broadcasting, integer multiplications, integer additions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `stride_i64_minimal` 为核心。测试在其中演示 tt.func、广播后的标量或指针、张量广播、整数乘法、整数加法，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 172-179
```mlir
172|     // CHECK: arith.trunci {{.*}} : tensor<256x64xi64, {{.*}}> to tensor<256x64xi32, {{.*}}>
173|     // CHECK: arith.trunci {{.*}} : i64 to i32
174|     // CHECK: amdg.buffer_load
175|     // CHECK-NOT: tt.load
176|     %v = tt.load %p : tensor<256x64x!tt.ptr<f16>, #blocked6>
177|     tt.return %v : tensor<256x64xf16, #blocked6>
178|   }
179| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `--tritonamdgpu-convert-buffer-ops="gfx-arch=gfx942"`
- **CN:** 主要 pass 选项：`-split-input-file`，`--tritonamdgpu-convert-buffer-ops="gfx-arch=gfx942"`
- **EN:** Dominant operations include `tt.func`, `tt.splat`, `module`, `tt.addptr`, `tt.return`, `tt.load`, `tt.make_range`, `arith.extsi`, `arith.constant`, `arith.muli`.
- **CN:** 主要操作包括 `tt.func`、`tt.splat`、`module`、`tt.addptr`、`tt.return`、`tt.load`、`tt.make_range`、`arith.extsi`、`arith.constant`、`arith.muli`。
- **EN:** The file contains 6 independently testable section(s). Check styles used: CHECK x15, CHECK-LABEL x7, CHECK-SAME x6, CHECK-NOT x5. Important labels include @load_i64_offset_bounded, @load_i64_offset_possibly_negative, @store_i64_offset_bounded, @load_i64_offset_pointer_range_32. It validates both the presence of the desired rewrite and the absence of unwanted residual operations.
- **CN:** 该文件包含 6 个可独立测试的分段。使用的检查类型：CHECK ×15，CHECK-LABEL ×7，CHECK-SAME ×6，CHECK-NOT ×5。 关键标签包括 @load_i64_offset_bounded，@load_i64_offset_possibly_negative，@store_i64_offset_bounded，@load_i64_offset_pointer_range_32。 它同时验证目标重写已经出现，并且不再残留不需要的操作。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `tt`: Triton high-level tensor/pointer operations.
- **CN:** `tt`：Triton 高层张量/指针操作。
- **EN:** `arith`: MLIR arithmetic dialect for scalar/tensor math.
- **CN:** `arith`：用于标量/张量运算的 MLIR arithmetic 方言。
- **EN:** `ttg`: TritonGPU layout, memory, and GPU-specific operations.
- **CN:** `ttg`：TritonGPU 布局、内存和 GPU 专用操作。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。