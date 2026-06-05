# loop-schedule.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/loop-schedule.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-tritongpu-assign-latencies=num-stages=3, -tritongpu-schedule-loops` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-tritongpu-assign-latencies=num-stages=3, -tritongpu-schedule-loops` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-assign-latencies=num-stages=3 -tritongpu-schedule-loops | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-assign-latencies=num-stages=3 -tritongpu-schedule-loops | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-assign-latencies=num-stages=3 -tritongpu-schedule-loops | FileCheck %s
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-assign-latencies=num-stages=3 -tritongpu-schedule-loops | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-assign-latencies=num-stages=3 -tritongpu-schedule-loops | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-10
```mlir
 3| #AL = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
 4| #BL = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
 5| #C = #ttg.nvidia_mma<{versionMajor = 2, warpsPerCTA = [4, 1], instrShape = [16, 8]}>
 6| #ALs0 = #ttg.slice<{parent=#AL, dim=0}>
 7| #BLs0 = #ttg.slice<{parent=#BL, dim=0}>
 8| #CLs0 = #ttg.slice<{parent=#C, dim=0}>
 9| #A = #ttg.dot_op<{opIdx = 0, parent = #C, kWidth=2}>
10| #B = #ttg.dot_op<{opIdx = 1, parent = #C, kWidth=2}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 11-11
```mlir
11| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 12-16
```mlir
12| // CHECK-LABLE: @matmul_loop_load_acc
13| // CHECK: tt.load %{{.*}} {loop.cluster = 3 : i32, loop.stage = 0 : i32}
14| // CHECK: tt.load %{{.*}} {loop.cluster = 3 : i32, loop.stage = 0 : i32}
15| // CHECK: tt.load %{{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
16| // CHECK: tt.dot {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 17-34
```mlir
17| tt.func @matmul_loop_load_acc(%lb : index, %ub : index, %step : index,
18|                   %A : !tt.ptr<f16> {tt.divisibility = 16 : i32},
19|                   %B : !tt.ptr<f16> {tt.divisibility = 16 : i32},
20|                   %C : !tt.ptr<f32> {tt.divisibility = 16 : i32},
21|                   %c_init: tensor<128x128xf32, #C>) -> tensor<128x128xf32, #C> {
22| 
23|   // A ptrs
24|   %a_ptr_splat = tt.splat %A : !tt.ptr<f16> -> tensor<128x32x!tt.ptr<f16>, #AL>
25|   %a_tmp0 = tt.make_range {end = 32: i32, start = 0: i32} : tensor<32xi32, #ALs0>
26|   %a_tmp1 = tt.expand_dims %a_tmp0 {axis = 0 : i32} : tensor<32xi32, #ALs0> -> tensor<1x32xi32, #AL>
27|   %a_offs = tt.broadcast %a_tmp1 : tensor<1x32xi32, #AL> -> tensor<128x32xi32, #AL>
28|   %a_ptr_init = tt.addptr %a_ptr_splat, %a_offs : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xi32, #AL>
29|   // B ptrs
30|   %b_ptr_splat = tt.splat %B : !tt.ptr<f16> -> tensor<32x128x!tt.ptr<f16>, #BL>
31|   %b_tmp0 = tt.make_range {end = 128: i32, start = 0: i32} : tensor<128xi32, #BLs0>
32|   %b_tmp1 = tt.expand_dims %b_tmp0 {axis = 0 : i32} : tensor<128xi32, #BLs0> -> tensor<1x128xi32, #BL>
33|   %b_offs = tt.broadcast %b_tmp1 : tensor<1x128xi32, #BL> -> tensor<32x128xi32, #BL>
34|   %b_ptr_init = tt.addptr %b_ptr_splat, %b_offs : tensor<32x128x!tt.ptr<f16>, #BL>, tensor<32x128xi32, #BL>
```
**EN:** This function-oriented block defines or enters `matmul_loop_load_acc`. Within it, the test exercises tt.func, broadcasted scalars or pointers, lane/block index ranges, shape expansion, tensor broadcasting, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `matmul_loop_load_acc` 为核心。测试在其中演示 tt.func、广播后的标量或指针、lane/block 索引范围、形状扩展、张量广播，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 35-52
```mlir
35|   // C ptrs
36|   %c_ptr_splat = tt.splat %C : !tt.ptr<f32> -> tensor<128x128x!tt.ptr<f32>, #C>
37|   %c_tmp0 = tt.make_range {end = 128: i32, start = 0: i32} : tensor<128xi32, #CLs0>
38|   %c_tmp1 = tt.expand_dims %c_tmp0 {axis = 0 : i32} : tensor<128xi32, #CLs0> -> tensor<1x128xi32, #C>
39|   %c_offs = tt.broadcast %c_tmp1 : tensor<1x128xi32, #C> -> tensor<128x128xi32, #C>
40|   %c_ptr_init = tt.addptr %c_ptr_splat, %c_offs : tensor<128x128x!tt.ptr<f32>, #C>, tensor<128x128xi32, #C>
41| 
42|   %a_off = arith.constant dense<4> : tensor<128x32xi32, #AL>
43|   %b_off = arith.constant dense<4> : tensor<32x128xi32, #BL>
44|   %c_off = arith.constant dense<4> : tensor<128x128xi32, #C>
45| 
46|   %loop:4 = scf.for %iv = %lb to %ub step %step iter_args(%a_ptr = %a_ptr_init, %b_ptr = %b_ptr_init, %c_ptr = %c_ptr_init, %prev_c = %c_init) -> (tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128x!tt.ptr<f32>, #C>, tensor<128x128xf32, #C>) {
47|     %a_ = tt.load %a_ptr : tensor<128x32x!tt.ptr<f16>, #AL>
48|     %a = ttg.convert_layout %a_ : tensor<128x32xf16, #AL> -> tensor<128x32xf16, #A>
49|     %b_ = tt.load %b_ptr : tensor<32x128x!tt.ptr<f16>, #BL>
50|     %b = ttg.convert_layout %b_ : tensor<32x128xf16, #BL> -> tensor<32x128xf16, #B>
51|     %c_ = tt.load %c_ptr : tensor<128x128x!tt.ptr<f32>, #C>
52|     %c = tt.dot %a, %b, %prev_c : tensor<128x32xf16, #A> * tensor<32x128xf16, #B> -> tensor<128x128xf32, #C>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining constants, masked or vectorized loads, layout conversions, broadcasted scalars or pointers, lane/block index ranges.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 常量、带掩码或向量化的加载、布局转换、广播后的标量或指针、lane/block 索引范围。

### Lines 53-62
```mlir
53| 
54|     %next_a_ptr = tt.addptr %a_ptr, %a_off : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xi32, #AL>
55|     %next_b_ptr = tt.addptr %b_ptr, %b_off : tensor<32x128x!tt.ptr<f16>, #BL>, tensor<32x128xi32, #BL>
56|     %next_c_ptr = tt.addptr %c_ptr, %c_off : tensor<128x128x!tt.ptr<f32>, #C>, tensor<128x128xi32, #C>
57|     scf.yield %next_a_ptr, %next_b_ptr, %next_c_ptr, %c : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128x!tt.ptr<f32>, #C>, tensor<128x128xf32, #C>
58|   }
59|   tt.return %loop#3: tensor<128x128xf32, #C>
60| }
61| }
62| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining pointer arithmetic, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 指针算术、循环/分支产出值、tt.return。

### Lines 63-63
```mlir
63| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 64-70
```mlir
64| 
65| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
66| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
67| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 256, 16]}>
68| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
69| #smem = #ttg.shared_memory
70| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 71-72
```mlir
71| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
72| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 73-73
```mlir
73| // CHECK-LABEL: @fused_loop
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @fused_loop anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @fused_loop 这样的标签用于锚定匹配范围。

### Lines 74-91
```mlir
74| tt.func public @fused_loop(%arg5: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg7: i32 {tt.divisibility = 16 : i32}) {
75|   %c10_i32 = arith.constant 10 : i32
76|   %false = arith.constant false
77|   %0 = ub.poison : !tt.tensordesc<64x256xf16>
78|   %cst = arith.constant dense<0> : tensor<128x1xi64, #blocked>
79|   %c-1_i32 = arith.constant -1 : i32
80|   %c1_i32 = arith.constant 1 : i32
81|   %c0_i32 = arith.constant 0 : i32
82|   %c64_i32 = arith.constant 64 : i32
83|   %c1_i64 = arith.constant 1 : i64
84|   %cst_0 = arith.constant dense<0.000000e+00> : tensor<128x256xf32, #mma>
85| 
86|   %1 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
87|   %2 = tt.expand_dims %1 {axis = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x64xi32, #blocked>
88|   %3 = arith.extsi %arg7 : i32 to i64
89|   %4 = tt.make_tensor_descriptor %arg5, [%arg7, %arg7], [%3, %c1_i64] : <f16>, <64x256xf16>
90|   %5 = tt.broadcast %2 : tensor<1x64xi32, #blocked> -> tensor<128x64xi32, #blocked>
91|   %7 = tt.splat %3 : i64 -> tensor<128x1xi64, #blocked>
```
**EN:** This function-oriented block defines or enters `fused_loop`. Within it, the test exercises constants, tt.func, ub.poison, lane/block index ranges, shape expansion, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fused_loop` 为核心。测试在其中演示 常量、tt.func、ub.poison、lane/block 索引范围、形状扩展，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 92-100
```mlir
 92| 
 93|   // CHECK: scf.for
 94|   %8:9 = scf.for %arg29 = %c0_i32 to %arg7 step %c1_i32 iter_args(%arg30 = %c-1_i32, %arg31 = %4, %arg32 = %c0_i32, %arg33 = %arg5, %arg34 = %cst_0, %arg35 = %c0_i32, %arg36 = %cst, %arg37 = %0, %arg38 = %false) -> (i32, !tt.tensordesc<64x256xf16>, i32, !tt.ptr<f16>, tensor<128x256xf32, #mma>, i32, tensor<128x1xi64, #blocked>, !tt.tensordesc<64x256xf16>, i1)  : i32 {
 95|     %9 = arith.addi %arg30, %c1_i32 : i32
 96|     %10 = arith.cmpi eq, %arg30, %c10_i32 : i32
 97|     %11 = arith.select %10, %c0_i32, %9 : i32
 98|     %12 = arith.cmpi eq, %11, %c0_i32 : i32
 99| 
100|     // This op is a distance 1 dependency of itself.
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining integer comparisons, structured loops, integer additions, arith.select.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 整数比较、结构化循环、整数加法、arith.select。

### Lines 101-118
```mlir
101|     // CHECK: {_test_marker_0, loop.cluster = 4 : i32, loop.stage = 0 : i32}
102|     %13 = arith.select %12, %c0_i32, %arg32 {_test_marker_0} : i32
103| 
104|     %14 = arith.select %12, %arg31, %arg37 : !tt.tensordesc<64x256xf16>
105|     %15 = arith.select %12, %c10_i32, %arg35 : i32
106|     %16 = scf.if %12 -> (tensor<128x1xi64, #blocked>) {
107|       %32 = arith.muli %cst, %7 : tensor<128x1xi64, #blocked>
108|       scf.yield %32 : tensor<128x1xi64, #blocked>
109|     } else {
110|       scf.yield %arg36 : tensor<128x1xi64, #blocked>
111|     }
112|     %17 = tt.splat %arg33 : !tt.ptr<f16> -> tensor<128x1x!tt.ptr<f16>, #blocked>
113|     %18 = tt.addptr %17, %16 : tensor<128x1x!tt.ptr<f16>, #blocked>, tensor<128x1xi64, #blocked>
114|     %19 = tt.broadcast %18 : tensor<128x1x!tt.ptr<f16>, #blocked> -> tensor<128x64x!tt.ptr<f16>, #blocked>
115|     %20 = tt.addptr %19, %5 : tensor<128x64x!tt.ptr<f16>, #blocked>, tensor<128x64xi32, #blocked>
116|     %21 = tt.addptr %arg33, %c64_i32 : !tt.ptr<f16>, i32
117|     %22 = tt.load %20 : tensor<128x64x!tt.ptr<f16>, #blocked>
118|     %23 = ttg.local_alloc %22 : (tensor<128x64xf16, #blocked>) -> !ttg.memdesc<128x64xf16, #shared, #smem>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining arith.select, pointer arithmetic, loop/if yielded values, structured conditionals, integer multiplications.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 arith.select、指针算术、循环/分支产出值、结构化条件分支、整数乘法。

### Lines 119-125
```mlir
119|     %24 = arith.muli %13, %c64_i32 : i32
120|     %25 = tt.descriptor_load %14[%24, %15] : !tt.tensordesc<64x256xf16> -> tensor<64x256xf16, #blocked1>
121|     %26 = ttg.local_alloc %25 : (tensor<64x256xf16, #blocked1>) -> !ttg.memdesc<64x256xf16, #shared, #smem>
122|     %27 = ttng.warp_group_dot %23, %26, %arg34, %arg38 {inputPrecision = 0 : i32} : !ttg.memdesc<128x64xf16, #shared, #smem> * !ttg.memdesc<64x256xf16, #shared, #smem> -> tensor<128x256xf32, #mma>
123|     %28 = arith.addi %13, %c1_i32 : i32
124| 
125|     // This op is in the backward slice of `_test_marker_2` and the epilogue.
```
**EN:** This operational block carries the concrete IR steps for the test, centered on integer multiplications, tt.descriptor_load, shared/local memory allocation, ttng.warp_group_dot, integer additions. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 整数乘法、tt.descriptor_load、共享/本地内存分配、ttng.warp_group_dot、整数加法。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 126-128
```mlir
126|     // CHECK: {_test_marker_1, loop.cluster = 3 : i32, loop.stage = 1 : i32}
127|     %29 = arith.cmpi eq, %11, %c10_i32 {_test_marker_1} : i32
128| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 129-135
```mlir
129|     // CHECK: {_test_marker_2, loop.cluster = 3 : i32, loop.stage = 1 : i32}
130|     %30 = arith.select %29, %arg5, %21 {_test_marker_2} : !tt.ptr<f16>
131| 
132|     %31 = arith.cmpi ne, %11, %c10_i32 : i32
133| 
134|     scf.if %29 {
135|       "use"(%27) : (tensor<128x256xf32, #mma>) -> ()
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining arith.select, integer comparisons, structured conditionals.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 arith.select、整数比较、结构化条件分支。

### Lines 136-144
```mlir
136|       // CHECK: {_test_marker_3, loop.cluster = 5 : i32, loop.stage = 2 : i32}
137|     } {_test_marker_3}
138|     scf.yield %11, %14, %28, %30, %27, %15, %16, %14, %31 : i32, !tt.tensordesc<64x256xf16>, i32, !tt.ptr<f16>, tensor<128x256xf32, #mma>, i32, tensor<128x1xi64, #blocked>, !tt.tensordesc<64x256xf16>, i1
139|   }
140|   tt.return
141| }
142| 
143| }
144| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、tt.return。

### Lines 145-145
```mlir
145| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 146-147
```mlir
146| 
147| // CHECK-LABEL: @prologue_backward_slice
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @prologue_backward_slice anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @prologue_backward_slice 这样的标签用于锚定匹配范围。

### Lines 148-151
```mlir
148| tt.func @prologue_backward_slice(%ub: i32, %cond: i1) {
149|   %c0_i32 = arith.constant 0 : i32
150|   %c1_i32 = arith.constant 1 : i32
151| 
```
**EN:** This function-oriented block defines or enters `prologue_backward_slice`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `prologue_backward_slice` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 152-153
```mlir
152|   // CHECK: scf.for
153|   scf.for %i = %c0_i32 to %ub step %c1_i32 : i32 {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 154-159
```mlir
154|     // CHECK: scf.if
155|     %0 = scf.if %cond -> i32 {
156|       scf.yield %c0_i32 : i32
157|     } else {
158|       scf.yield %c1_i32 : i32
159|     }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, structured conditionals.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、结构化条件分支。

### Lines 160-165
```mlir
160|     // CHECK: loop.cluster = 0 : i32, loop.stage = 0 : i32
161| 
162|     // CHECK: op.with_region
163|     %1 = "op.with_region"() ({
164|       "use"(%0) : (i32) -> ()
165|     }) : () -> i32
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 166-171
```mlir
166|     // CHECK: loop.cluster = 1 : i32, loop.stage = 0 : i32
167| 
168|     // CHECK: op.with_region
169|     "op.with_region"() ({
170|       "use"(%1) : (i32) -> ()
171|     }) {tt.latency = 2 : i32} : () -> ()
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 172-178
```mlir
172|     // CHECK: loop.cluster = 1 : i32, loop.stage = 0 : i32
173| 
174|   } {tt.num_stages = 3 : i32}
175| 
176|   tt.return
177| }
178| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 179-179
```mlir
179| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 180-181
```mlir
180| 
181| // CHECK-LABEL: @epilogue_forward_slice
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @epilogue_forward_slice anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @epilogue_forward_slice 这样的标签用于锚定匹配范围。

### Lines 182-185
```mlir
182| tt.func @epilogue_forward_slice(%ub: i32, %cond: i1) {
183|   %c0_i32 = arith.constant 0 : i32
184|   %c1_i32 = arith.constant 1 : i32
185| 
```
**EN:** This function-oriented block defines or enters `epilogue_forward_slice`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `epilogue_forward_slice` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 186-187
```mlir
186|   // CHECK: scf.for
187|   scf.for %i = %c0_i32 to %ub step %c1_i32 : i32 {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 188-189
```mlir
188|     // CHECK: "latency.op"() {loop.cluster = 3 : i32, loop.stage = 0 : i32
189|     %0 = "latency.op"() {tt.latency = 2 : i32} : () -> i32
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 190-195
```mlir
190|     // CHECK: scf.if
191|     %1 = scf.if %cond -> i32 {
192|       scf.yield %0 : i32
193|     } else {
194|       scf.yield %c0_i32 : i32
195|     }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, structured conditionals.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、结构化条件分支。

### Lines 196-205
```mlir
196|     // CHECK: {loop.cluster = 1 : i32, loop.stage = 2 : i32}
197| 
198|     // CHECK: "use"(%{{.*}}) {loop.cluster = 1 : i32, loop.stage = 2 : i32}
199|     "use"(%1) : (i32) -> ()
200| 
201|   } {tt.num_stages = 3 : i32}
202| 
203|   tt.return
204| }
205| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 206-206
```mlir
206| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 207-208
```mlir
207| 
208| // CHECK-LABEL: @prologue_latency
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @prologue_latency anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @prologue_latency 这样的标签用于锚定匹配范围。

### Lines 209-212
```mlir
209| tt.func @prologue_latency(%ub: i32, %cond: i1) {
210|   %c0_i32 = arith.constant 0 : i32
211|   %c1_i32 = arith.constant 1 : i32
212| 
```
**EN:** This function-oriented block defines or enters `prologue_latency`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `prologue_latency` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 213-214
```mlir
213|   // CHECK: scf.for
214|   scf.for %i = %c0_i32 to %ub step %c1_i32 : i32 {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 215-216
```mlir
215|     // CHECK: "some.op"() {loop.cluster = 0 : i32, loop.stage = 0 : i32}
216|     %0 = "some.op"() : () -> i32
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 217-222
```mlir
217|     // CHECK: scf.if
218|     %1 = scf.if %cond -> i32 {
219|       scf.yield %0 : i32
220|     } else {
221|       scf.yield %c0_i32 : i32
222|     } {tt.latency = 2 : i32}
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, structured conditionals.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、结构化条件分支。

### Lines 223-228
```mlir
223|     // CHECK: loop.cluster = 0 : i32, loop.stage = 0 : i32
224| 
225|   } {tt.num_stages = 3 : i32}
226| 
227|   tt.return
228| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-allow-unregistered-dialect`, `-tritongpu-assign-latencies=num-stages=3`, `-tritongpu-schedule-loops`
- **CN:** 主要 pass 选项：`-split-input-file`，`-allow-unregistered-dialect`，`-tritongpu-assign-latencies=num-stages=3`，`-tritongpu-schedule-loops`
- **EN:** Dominant operations include `arith.constant`, `tt.func`, `scf.yield`, `tt.addptr`, `tt.splat`, `tt.broadcast`, `scf.for`, `tt.return`, `arith.select`, `scf.if`.
- **CN:** 主要操作包括 `arith.constant`、`tt.func`、`scf.yield`、`tt.addptr`、`tt.splat`、`tt.broadcast`、`scf.for`、`tt.return`、`arith.select`、`scf.if`。
- **EN:** The file contains 4 independently testable section(s). Check styles used: CHECK x25, CHECK-LABEL x4, CHECK-LABLE x1. Important labels include @fused_loop, @prologue_backward_slice, @epilogue_forward_slice, @prologue_latency. It validates that the selected pass pipeline rewrites each labeled test case into the checked form.
- **CN:** 该文件包含 4 个可独立测试的分段。使用的检查类型：CHECK ×25，CHECK-LABEL ×4，CHECK-LABLE ×1。 关键标签包括 @fused_loop，@prologue_backward_slice，@epilogue_forward_slice，@prologue_latency。 它验证所选 pass 流水线会把每个带标签的测试用例重写成检查所要求的形式。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `tt`: Triton high-level tensor/pointer operations.
- **CN:** `tt`：Triton 高层张量/指针操作。
- **EN:** `arith`: MLIR arithmetic dialect for scalar/tensor math.
- **CN:** `arith`：用于标量/张量运算的 MLIR arithmetic 方言。
- **EN:** `ttg`: TritonGPU layout, memory, and GPU-specific operations.
- **CN:** `ttg`：TritonGPU 布局、内存和 GPU 专用操作。
- **EN:** `scf`: Structured control flow for loops and conditionals.
- **CN:** `scf`：用于循环与条件分支的结构化控制流。
- **EN:** `ttng`: Triton Nvidia GPU tensor-memory and MMA operations.
- **CN:** `ttng`：Triton Nvidia GPU 张量内存与 MMA 操作。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。