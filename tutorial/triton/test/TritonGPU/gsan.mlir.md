# gsan.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/gsan.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-tritoninstrument-global-sanitizer` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-tritoninstrument-global-sanitizer` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -tritoninstrument-global-sanitizer | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -tritoninstrument-global-sanitizer | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -tritoninstrument-global-sanitizer | FileCheck %s
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file -allow-unregistered-dialect -tritoninstrument-global-sanitizer | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file -allow-unregistered-dialect -tritoninstrument-global-sanitizer | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-4
```mlir
3| #blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
4| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 5-5
```mlir
5| module attributes {"ttg.num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 6-6
```mlir
6|   // CHECK-LABEL: tt.func @instrumented
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tt.func @instrumented anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tt.func @instrumented 这样的标签用于锚定匹配范围。

### Lines 7-10
```mlir
 7|   tt.func @instrumented(%ptrs: tensor<128x!tt.ptr<f32>, #blocked>,
 8|                         %mask: tensor<128xi1, #blocked>,
 9|                         %other: tensor<128xf32, #blocked>,
10|                         %vals: tensor<128xf32, #blocked>) {
```
**EN:** This function-oriented block defines or enters `instrumented`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `instrumented` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 11-13
```mlir
11|     // CHECK: tti.experimental_gsan_tensor_access %{{.*}}, false, %{{.*}}
12|     // CHECK-NEXT: %[[LD:.*]] = tt.load
13|     %0 = tt.load %ptrs, %mask, %other : tensor<128x!tt.ptr<f32>, #blocked>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 14-20
```mlir
14|     // CHECK: tti.experimental_gsan_tensor_access %{{.*}}, true, %{{.*}}
15|     // CHECK-NEXT: tt.store
16|     tt.store %ptrs, %vals, %mask : tensor<128x!tt.ptr<f32>, #blocked>
17|     tt.return
18|   }
19| }
20| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized stores, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的存储、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 21-21
```mlir
21| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 22-26
```mlir
22| 
23| #blocked = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [32], warpsPerCTA = [2], order = [0]}>
24| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
25| #smem = #ttg.shared_memory
26| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 27-27
```mlir
27| module attributes {"ttg.num-warps" = 2 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 28-28
```mlir
28|   // CHECK-LABEL: tt.func @instrumented_async_copy
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tt.func @instrumented_async_copy anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tt.func @instrumented_async_copy 这样的标签用于锚定匹配范围。

### Lines 29-31
```mlir
29|   tt.func @instrumented_async_copy(%ptrs: tensor<128x!tt.ptr<f16>, #blocked>,
30|                                    %mask: tensor<128xi1, #blocked>) {
31|     %buf = ttg.local_alloc : () -> !ttg.memdesc<128xf16, #shared, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `instrumented_async_copy`. Within it, the test exercises tt.func, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `instrumented_async_copy` 为核心。测试在其中演示 tt.func、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 32-38
```mlir
32|     // CHECK: tti.experimental_gsan_tensor_access %[[PTRS:.*]], false, %[[MASK:.*]] :
33|     // CHECK-NEXT: ttg.async_copy_global_to_local %[[PTRS]], {{.*}} mask %[[MASK]]
34|     %tok = ttg.async_copy_global_to_local %ptrs, %buf mask %mask : tensor<128x!tt.ptr<f16>, #blocked> -> <128xf16, #shared, #smem, mutable>
35|     tt.return
36|   }
37| }
38| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.async_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.async_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 39-39
```mlir
39| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 40-44
```mlir
40| 
41| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
42| #bar = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
43| #smem = #ttg.shared_memory
44| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 45-45
```mlir
45| module attributes {"ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 46-46
```mlir
46|   // CHECK-LABEL: tt.func @instrumented_async_tma_copy
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tt.func @instrumented_async_tma_copy anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tt.func @instrumented_async_tma_copy 这样的标签用于锚定匹配范围。

### Lines 47-51
```mlir
47|   tt.func @instrumented_async_tma_copy(%desc: !tt.tensordesc<32x32xf32, #shared>) {
48|     %true = arith.constant true
49|     %c0_i32 = arith.constant 0 : i32
50|     %buf = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
51|     %barrier = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<1xi64, #bar, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `instrumented_async_tma_copy`. Within it, the test exercises tt.func, constants, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `instrumented_async_tma_copy` 为核心。测试在其中演示 tt.func、常量、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 52-55
```mlir
52|     // CHECK: tti.experimental_gsan_tensordesc_info %arg0
53|     // CHECK: tti.experimental_gsan_tensor_access %{{.*}}, false, %{{.*}}
54|     // CHECK-NEXT: ttng.async_tma_copy_global_to_local
55|     ttng.async_tma_copy_global_to_local %desc[%c0_i32, %c0_i32] %buf, %barrier, %true : !tt.tensordesc<32x32xf32, #shared>, !ttg.memdesc<1xi64, #bar, #smem, mutable> -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.async_tma_copy_global_to_local. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.async_tma_copy_global_to_local。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 56-63
```mlir
56|     // CHECK: tti.experimental_gsan_tensordesc_info %arg0
57|     // CHECK: tti.experimental_gsan_tensor_access %{{.*}}, true, %{{.*}}
58|     // CHECK-NEXT: ttng.async_tma_copy_local_to_global
59|     ttng.async_tma_copy_local_to_global %desc[%c0_i32, %c0_i32] %buf : !tt.tensordesc<32x32xf32, #shared>, !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
60|     tt.return
61|   }
62| }
63| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.async_tma_copy_local_to_global, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.async_tma_copy_local_to_global、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 64-64
```mlir
64| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 65-71
```mlir
65| 
66| #blocked_rows_parent = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [32, 1], warpsPerCTA = [1, 1], order = [1, 0]}>
67| #blocked_rows = #ttg.slice<{dim = 0, parent = #blocked_rows_parent}>
68| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
69| #bar = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
70| #smem = #ttg.shared_memory
71| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 72-72
```mlir
72| module attributes {"ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 73-73
```mlir
73|   // CHECK-LABEL: tt.func @instrumented_async_tma_gather_scatter
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tt.func @instrumented_async_tma_gather_scatter anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tt.func @instrumented_async_tma_gather_scatter 这样的标签用于锚定匹配范围。

### Lines 74-79
```mlir
74|   tt.func @instrumented_async_tma_gather_scatter(%desc: !tt.tensordesc<1x32xf32, #shared>) {
75|     %true = arith.constant true
76|     %c0_i32 = arith.constant 0 : i32
77|     %x_offsets = arith.constant dense<1> : tensor<32xi32, #blocked_rows>
78|     %buf = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
79|     %barrier = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<1xi64, #bar, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `instrumented_async_tma_gather_scatter`. Within it, the test exercises constants, tt.func, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `instrumented_async_tma_gather_scatter` 为核心。测试在其中演示 常量、tt.func、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 80-83
```mlir
80|     // CHECK: tti.experimental_gsan_tensordesc_info %arg0
81|     // CHECK: tti.experimental_gsan_tensor_access %{{.*}}, false, %{{.*}}
82|     // CHECK-NEXT: ttng.async_tma_gather
83|     ttng.async_tma_gather %desc[%x_offsets, %c0_i32] %buf, %barrier, %true : !tt.tensordesc<1x32xf32, #shared>, tensor<32xi32, #blocked_rows>, i32, !ttg.memdesc<1xi64, #bar, #smem, mutable>, !ttg.memdesc<32x32xf32, #shared, #smem, mutable>, i1
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.async_tma_gather. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.async_tma_gather。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 84-91
```mlir
84|     // CHECK: tti.experimental_gsan_tensordesc_info %arg0
85|     // CHECK: tti.experimental_gsan_tensor_access %{{.*}}, true, %{{.*}}
86|     // CHECK-NEXT: ttng.async_tma_scatter
87|     ttng.async_tma_scatter %desc[%x_offsets, %c0_i32] %buf : !tt.tensordesc<1x32xf32, #shared>, tensor<32xi32, #blocked_rows>, i32, !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
88|     tt.return
89|   }
90| }
91| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.async_tma_scatter, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.async_tma_scatter、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 92-92
```mlir
92| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 93-97
```mlir
93| 
94| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
95| #bar = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
96| #smem = #ttg.shared_memory
97| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 98-98
```mlir
98| module attributes {"ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 99-99
```mlir
99|   // CHECK-LABEL: tt.func @instrumented_async_tma_copy_device_desc
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tt.func @instrumented_async_tma_copy_device_desc anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tt.func @instrumented_async_tma_copy_device_desc 这样的标签用于锚定匹配范围。

### Lines 100-108
```mlir
100|   tt.func @instrumented_async_tma_copy_device_desc(%raw_desc: !tt.ptr<i8>,
101|                                                    %base: !tt.ptr<f32>,
102|                                                    %shape0: i32, %shape1: i32,
103|                                                    %stride0: i64) {
104|     %true = arith.constant true
105|     %c0_i32 = arith.constant 0 : i32
106|     %c1_i32 = arith.constant 1 : i32
107|     %c32_i32 = arith.constant 32 : i32
108|     ttng.tensormap_create %raw_desc, %base, [%c32_i32, %c32_i32], [%shape1, %shape0], [%stride0], [%c1_i32, %c1_i32] {elem_type = 0 : i32, fill_mode = 0 : i32, interleave_layout = 0 : i32, swizzle_mode = 0 : i32} : (!tt.ptr<i8>, !tt.ptr<f32>, i32, i32, i32, i32, i64, i32, i32) -> ()
```
**EN:** This function-oriented block defines or enters `instrumented_async_tma_copy_device_desc`. Within it, the test exercises constants, tt.func, ttng.tensormap_create, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `instrumented_async_tma_copy_device_desc` 为核心。测试在其中演示 常量、tt.func、ttng.tensormap_create，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 109-112
```mlir
109|     // CHECK: %[[DESC:.*]] = ttng.reinterpret_tensor_descriptor %arg0
110|     %desc = ttng.reinterpret_tensor_descriptor %raw_desc : !tt.ptr<i8> to !tt.tensordesc<32x32xf32, #shared>
111|     %buf = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
112|     %barrier = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<1xi64, #bar, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, ttng.reinterpret_tensor_descriptor. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、ttng.reinterpret_tensor_descriptor。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 113-119
```mlir
113|     // CHECK: tti.experimental_gsan_tensordesc_info %[[DESC]]
114|     // CHECK: tti.experimental_gsan_tensor_access %{{.*}}, false, %{{.*}}
115|     // CHECK-NEXT: ttng.async_tma_copy_global_to_local
116|     ttng.async_tma_copy_global_to_local %desc[%c0_i32, %c0_i32] %buf, %barrier, %true : !tt.tensordesc<32x32xf32, #shared>, !ttg.memdesc<1xi64, #bar, #smem, mutable> -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
117|     tt.return
118|   }
119| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.async_tma_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.async_tma_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-allow-unregistered-dialect`, `-tritoninstrument-global-sanitizer`
- **CN:** 主要 pass 选项：`-split-input-file`，`-allow-unregistered-dialect`，`-tritoninstrument-global-sanitizer`
- **EN:** Dominant operations include `tt.func`, `arith.constant`, `ttg.local_alloc`, `module`, `tt.return`, `ttng.async_tma_copy_global_to_local`, `tt.load`, `tt.store`, `ttg.async_copy_global_to_local`, `ttng.async_tma_copy_local_to_global`.
- **CN:** 主要操作包括 `tt.func`、`arith.constant`、`ttg.local_alloc`、`module`、`tt.return`、`ttng.async_tma_copy_global_to_local`、`tt.load`、`tt.store`、`ttg.async_copy_global_to_local`、`ttng.async_tma_copy_local_to_global`。
- **EN:** The file contains 4 independently testable section(s). Check styles used: CHECK x14, CHECK-NEXT x8, CHECK-LABEL x5. Important labels include tt.func @instrumented, tt.func @instrumented_async_copy, tt.func @instrumented_async_tma_copy, tt.func @instrumented_async_tma_gather_scatter. It validates that the selected pass pipeline rewrites each labeled test case into the checked form.
- **CN:** 该文件包含 4 个可独立测试的分段。使用的检查类型：CHECK ×14，CHECK-NEXT ×8，CHECK-LABEL ×5。 关键标签包括 tt.func @instrumented，tt.func @instrumented_async_copy，tt.func @instrumented_async_tma_copy，tt.func @instrumented_async_tma_gather_scatter。 它验证所选 pass 流水线会把每个带标签的测试用例重写成检查所要求的形式。

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