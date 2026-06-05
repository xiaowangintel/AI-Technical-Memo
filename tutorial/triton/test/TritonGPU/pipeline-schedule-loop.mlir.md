# pipeline-schedule-loop.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/pipeline-schedule-loop.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-tritongpu-schedule-loops, -canonicalize` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-tritongpu-schedule-loops, -canonicalize` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -allow-unregistered-dialect -split-input-file -tritongpu-schedule-loops -canonicalize | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -allow-unregistered-dialect -split-input-file -tritongpu-schedule-loops -canonicalize | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt %s -allow-unregistered-dialect -split-input-file -tritongpu-schedule-loops -canonicalize | FileCheck %s
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -allow-unregistered-dialect -split-input-file -tritongpu-schedule-loops -canonicalize | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -allow-unregistered-dialect -split-input-file -tritongpu-schedule-loops -canonicalize | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-11
```mlir
 3| #AL = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
 4| #BL = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
 5| #C = #ttg.nvidia_mma<{versionMajor = 2, warpsPerCTA = [4, 1], instrShape = [16, 8]}>
 6| #A = #ttg.dot_op<{opIdx = 0, parent = #C, kWidth=2}>
 7| #B = #ttg.dot_op<{opIdx = 1, parent = #C, kWidth=2}>
 8| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 16}>
 9| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = true, elementBitWidth = 16}>
10| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [8, 1], instrShape = [16, 128, 32]}>
11| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 12-12
```mlir
12| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 13-13
```mlir
13| // CHECK-LABEL: @one_dep
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @one_dep anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @one_dep 这样的标签用于锚定匹配范围。

### Lines 14-17
```mlir
14| tt.func @one_dep(%lb : index, %ub : index, %step : index,
15|                  %a_ptr_init : tensor<128x32x!tt.ptr<f16>, #A>) -> tensor<128x32xf16, #A> {
16|   %init = arith.constant dense<0.00e+00> : tensor<128x32xf16, #A>
17|   %loop = scf.for %iv = %lb to %ub step %step iter_args(%acc = %init) -> (tensor<128x32xf16, #A>) {
```
**EN:** This function-oriented block defines or enters `one_dep`. Within it, the test exercises tt.func, constants, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `one_dep` 为核心。测试在其中演示 tt.func、常量、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 18-19
```mlir
18|     // CHECK: tt.load {{.*}} {loop.cluster = 2 : i32, loop.stage = 0 : i32}
19|     %a = tt.load %a_ptr_init {tt.latency = 2 : i32} : tensor<128x32x!tt.ptr<f16>, #A>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 20-23
```mlir
20|     // CHECK: arith.addf {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
21|     %res = arith.addf %acc, %a : tensor<128x32xf16, #A>
22|     scf.yield %res : tensor<128x32xf16, #A>
23|   }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining floating-point additions, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 浮点加法、循环/分支产出值。

### Lines 24-27
```mlir
24|   // CHECK: tt.scheduled_max_stage
25|   tt.return %loop#0 : tensor<128x32xf16, #A>
26| }
27| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 28-28
```mlir
28| // CHECK-LABEL: @parallel_deps
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @parallel_deps anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @parallel_deps 这样的标签用于锚定匹配范围。

### Lines 29-33
```mlir
29| tt.func @parallel_deps(%lb : index, %ub : index, %step : index,
30|                        %a_ptr_init : tensor<128x32x!tt.ptr<f16>, #A>,
31|                        %b_ptr_init : tensor<128x32x!tt.ptr<f16>, #A>) -> (tensor<128x32xf16, #A>, tensor<128x32xf16, #A>) {
32|   %init = arith.constant dense<0.00e+00> : tensor<128x32xf16, #A>
33|   %loop:2 = scf.for %iv = %lb to %ub step %step iter_args(%acc_a = %init, %acc_b = %init) -> (tensor<128x32xf16, #A>, tensor<128x32xf16, #A>) {
```
**EN:** This function-oriented block defines or enters `parallel_deps`. Within it, the test exercises tt.func, constants, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `parallel_deps` 为核心。测试在其中演示 tt.func、常量、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 34-35
```mlir
34|     // CHECK: tt.load {{.*}} {loop.cluster = 2 : i32, loop.stage = 0 : i32}
35|     %a = tt.load %a_ptr_init {tt.latency = 2 : i32} : tensor<128x32x!tt.ptr<f16>, #A>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 36-37
```mlir
36|     // CHECK: tt.load {{.*}} {loop.cluster = 2 : i32, loop.stage = 0 : i32}
37|     %b = tt.load %a_ptr_init {tt.latency = 2 : i32} : tensor<128x32x!tt.ptr<f16>, #A>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 38-39
```mlir
38|     // CHECK: arith.addf {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
39|     %res_a = arith.addf %acc_a, %a : tensor<128x32xf16, #A>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 40-46
```mlir
40|     // CHECK: arith.addf {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
41|     %res_b = arith.addf %acc_b, %b : tensor<128x32xf16, #A>
42|     scf.yield %res_a, %res_b : tensor<128x32xf16, #A>, tensor<128x32xf16, #A>
43|   }
44|   tt.return %loop#0, %loop#1 : tensor<128x32xf16, #A>, tensor<128x32xf16, #A>
45| }
46| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining floating-point additions, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 浮点加法、循环/分支产出值、tt.return。

### Lines 47-47
```mlir
47| // CHECK-LABEL: @parallel_deps_uneven1
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @parallel_deps_uneven1 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @parallel_deps_uneven1 这样的标签用于锚定匹配范围。

### Lines 48-52
```mlir
48| tt.func @parallel_deps_uneven1(%lb : index, %ub : index, %step : index,
49|                        %a_ptr_init : tensor<128x32x!tt.ptr<f16>, #A>,
50|                        %b_ptr_init : tensor<128x32x!tt.ptr<f16>, #A>) -> (tensor<128x32xf16, #A>, tensor<128x32xf16, #A>) {
51|   %init = arith.constant dense<0.00e+00> : tensor<128x32xf16, #A>
52|   %loop:2 = scf.for %iv = %lb to %ub step %step iter_args(%acc_a = %init, %acc_b = %init) -> (tensor<128x32xf16, #A>, tensor<128x32xf16, #A>) {
```
**EN:** This function-oriented block defines or enters `parallel_deps_uneven1`. Within it, the test exercises tt.func, constants, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `parallel_deps_uneven1` 为核心。测试在其中演示 tt.func、常量、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 53-54
```mlir
53|     // CHECK: tt.load {{.*}} {loop.cluster = 2 : i32, loop.stage = 0 : i32}
54|     %a = tt.load %a_ptr_init {tt.latency = 2 : i32} : tensor<128x32x!tt.ptr<f16>, #A>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 55-56
```mlir
55|     // CHECK: tt.load {{.*}} {loop.cluster = 1 : i32, loop.stage = 1 : i32}
56|     %b = tt.load %a_ptr_init {tt.latency = 1 : i32} : tensor<128x32x!tt.ptr<f16>, #A>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 57-58
```mlir
57|     // CHECK: arith.addf {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
58|     %res_a = arith.addf %acc_a, %a : tensor<128x32xf16, #A>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 59-65
```mlir
59|     // CHECK: arith.addf {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
60|     %res_b = arith.addf %acc_b, %b : tensor<128x32xf16, #A>
61|     scf.yield %res_a, %res_b : tensor<128x32xf16, #A>, tensor<128x32xf16, #A>
62|   }
63|   tt.return %loop#0, %loop#1 : tensor<128x32xf16, #A>, tensor<128x32xf16, #A>
64| }
65| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining floating-point additions, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 浮点加法、循环/分支产出值、tt.return。

### Lines 66-66
```mlir
66| // CHECK-LABEL: @parallel_deps_uneven2
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @parallel_deps_uneven2 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @parallel_deps_uneven2 这样的标签用于锚定匹配范围。

### Lines 67-71
```mlir
67| tt.func @parallel_deps_uneven2(%lb : index, %ub : index, %step : index,
68|                        %a_ptr_init : tensor<128x32x!tt.ptr<f16>, #A>,
69|                        %b_ptr_init : tensor<128x32x!tt.ptr<f16>, #A>) -> (tensor<128x32xf16, #A>, tensor<128x32xf16, #A>) {
70|   %init = arith.constant dense<0.00e+00> : tensor<128x32xf16, #A>
71|   %loop:2 = scf.for %iv = %lb to %ub step %step iter_args(%acc_a = %init, %acc_b = %init) -> (tensor<128x32xf16, #A>, tensor<128x32xf16, #A>) {
```
**EN:** This function-oriented block defines or enters `parallel_deps_uneven2`. Within it, the test exercises tt.func, constants, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `parallel_deps_uneven2` 为核心。测试在其中演示 tt.func、常量、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 72-73
```mlir
72|     // CHECK: tt.load {{.*}} {loop.cluster = 1 : i32, loop.stage = 1 : i32}
73|     %a = tt.load %a_ptr_init {tt.latency = 1 : i32} : tensor<128x32x!tt.ptr<f16>, #A>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 74-75
```mlir
74|     // CHECK: tt.load {{.*}} {loop.cluster = 2 : i32, loop.stage = 0 : i32}
75|     %b = tt.load %a_ptr_init {tt.latency = 2 : i32} : tensor<128x32x!tt.ptr<f16>, #A>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 76-77
```mlir
76|     // CHECK: arith.addf {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
77|     %res_a = arith.addf %acc_a, %a : tensor<128x32xf16, #A>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 78-84
```mlir
78|     // CHECK: arith.addf {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
79|     %res_b = arith.addf %acc_b, %b : tensor<128x32xf16, #A>
80|     scf.yield %res_a, %res_b : tensor<128x32xf16, #A>, tensor<128x32xf16, #A>
81|   }
82|   tt.return %loop#0, %loop#1 : tensor<128x32xf16, #A>, tensor<128x32xf16, #A>
83| }
84| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining floating-point additions, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 浮点加法、循环/分支产出值、tt.return。

### Lines 85-85
```mlir
85| // CHECK-LABEL: @direct_deps
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @direct_deps anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @direct_deps 这样的标签用于锚定匹配范围。

### Lines 86-90
```mlir
86| tt.func @direct_deps(%lb : index, %ub : index, %step : index,
87|                  %a_ptr_init : tensor<128x32x!tt.ptr<f16>, #A>) -> tensor<128x32xf16, #A> {
88|   %init = arith.constant dense<0.00e+00> : tensor<128x32xf16, #A>
89|   %a_off = arith.constant dense<4> : tensor<128x32xi32, #A>
90|   %loop:2 = scf.for %iv = %lb to %ub step %step iter_args(%acc = %init, %a_ptr = %a_ptr_init) -> (tensor<128x32xf16, #A>, tensor<128x32x!tt.ptr<f16>, #A>) {
```
**EN:** This function-oriented block defines or enters `direct_deps`. Within it, the test exercises tt.func, constants, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `direct_deps` 为核心。测试在其中演示 tt.func、常量、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 91-92
```mlir
91|     // CHECK: tt.addptr {{.*}} {loop.cluster = 2 : i32, loop.stage = 0 : i32}
92|     %a_ptr_next = tt.addptr %a_ptr, %a_off : tensor<128x32x!tt.ptr<f16>, #A>, tensor<128x32xi32, #A>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 93-94
```mlir
93|     // CHECK: tt.load {{.*}} {loop.cluster = 2 : i32, loop.stage = 0 : i32}
94|     %a = tt.load %a_ptr_next {tt.latency = 2 : i32} : tensor<128x32x!tt.ptr<f16>, #A>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 95-101
```mlir
 95|     // CHECK: arith.addf {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
 96|     %res = arith.addf %acc, %a : tensor<128x32xf16, #A>
 97|     scf.yield %res, %a_ptr_next : tensor<128x32xf16, #A>, tensor<128x32x!tt.ptr<f16>, #A>
 98|   }
 99|   tt.return %loop#0 : tensor<128x32xf16, #A>
100| }
101| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining floating-point additions, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 浮点加法、循环/分支产出值、tt.return。

### Lines 102-102
```mlir
102| // CHECK-LABEL: @dist1_deps
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @dist1_deps anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @dist1_deps 这样的标签用于锚定匹配范围。

### Lines 103-107
```mlir
103| tt.func @dist1_deps(%lb : index, %ub : index, %step : index,
104|                  %a_ptr_init : tensor<128x32x!tt.ptr<f16>, #A>) -> tensor<128x32xf16, #A> {
105|   %init = arith.constant dense<0.00e+00> : tensor<128x32xf16, #A>
106|   %a_off = arith.constant dense<4> : tensor<128x32xi32, #A>
107|   %loop:2 = scf.for %iv = %lb to %ub step %step iter_args(%acc = %init, %a_ptr = %a_ptr_init) -> (tensor<128x32xf16, #A>, tensor<128x32x!tt.ptr<f16>, #A>) {
```
**EN:** This function-oriented block defines or enters `dist1_deps`. Within it, the test exercises tt.func, constants, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `dist1_deps` 为核心。测试在其中演示 tt.func、常量、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 108-109
```mlir
108|     // CHECK: tt.load {{.*}} {loop.cluster = 3 : i32, loop.stage = 0 : i32}
109|     %a = tt.load %a_ptr {tt.latency = 2 : i32} : tensor<128x32x!tt.ptr<f16>, #A>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 110-111
```mlir
110|     // CHECK: arith.addf {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
111|     %res = arith.addf %acc, %a : tensor<128x32xf16, #A>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 112-118
```mlir
112|     // CHECK: tt.addptr {{.*}} {loop.cluster = 2 : i32, loop.stage = 1 : i32}
113|     %a_ptr_next = tt.addptr %a_ptr, %a_off : tensor<128x32x!tt.ptr<f16>, #A>, tensor<128x32xi32, #A>
114|     scf.yield %res, %a_ptr_next : tensor<128x32xf16, #A>, tensor<128x32x!tt.ptr<f16>, #A>
115|   }
116|   tt.return %loop#0 : tensor<128x32xf16, #A>
117| }
118| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining pointer arithmetic, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 指针算术、循环/分支产出值、tt.return。

### Lines 119-119
```mlir
119| // CHECK-LABEL: @prologue_if
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @prologue_if anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @prologue_if 这样的标签用于锚定匹配范围。

### Lines 120-124
```mlir
120| tt.func @prologue_if(%lb : index, %ub : index, %step : index, %cnd : i1,
121|                  %a_ptr_init : tensor<128x32x!tt.ptr<f16>, #A>) -> tensor<128x32xf16, #A> {
122|   %init = arith.constant dense<0.00e+00> : tensor<128x32xf16, #A>
123|   %a_off = arith.constant dense<4> : tensor<128x32xi32, #A>
124|   %loop = scf.for %iv = %lb to %ub step %step iter_args(%acc = %init) -> (tensor<128x32xf16, #A>) {
```
**EN:** This function-oriented block defines or enters `prologue_if`. Within it, the test exercises tt.func, constants, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `prologue_if` 为核心。测试在其中演示 tt.func、常量、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 125-132
```mlir
125|     // CHECK: scf.if
126|     // CHECK: {loop.cluster = 0 : i32, loop.stage = 0 : i32}
127|     %a_ptr = scf.if %cnd -> tensor<128x32x!tt.ptr<f16>, #A> {
128|       %a_ptr_ret = tt.addptr %a_ptr_init, %a_off : tensor<128x32x!tt.ptr<f16>, #A>, tensor<128x32xi32, #A>
129|       scf.yield %a_ptr_ret : tensor<128x32x!tt.ptr<f16>, #A>
130|     } else {
131|       scf.yield %a_ptr_init : tensor<128x32x!tt.ptr<f16>, #A>
132|     }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, structured conditionals, pointer arithmetic.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、结构化条件分支、指针算术。

### Lines 133-134
```mlir
133|     // CHECK: tt.load {{.*}} {loop.cluster = 3 : i32, loop.stage = 0 : i32}
134|     %a = tt.load %a_ptr {tt.latency = 2 : i32} : tensor<128x32x!tt.ptr<f16>, #A>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 135-141
```mlir
135|     // CHECK: arith.addf {{.*}} {loop.cluster = 1 : i32, loop.stage = 2 : i32}
136|     %res = arith.addf %acc, %a : tensor<128x32xf16, #A>
137|     scf.yield %res : tensor<128x32xf16, #A>
138|   }
139|   tt.return %loop#0 : tensor<128x32xf16, #A>
140| }
141| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining floating-point additions, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 浮点加法、循环/分支产出值、tt.return。

### Lines 142-142
```mlir
142| // CHECK-LABEL: @independent_epilogue_if
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @independent_epilogue_if anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @independent_epilogue_if 这样的标签用于锚定匹配范围。

### Lines 143-147
```mlir
143| tt.func @independent_epilogue_if(%lb : index, %ub : index, %step : index, %cnd : i1,
144|                  %a_ptr_init : tensor<128x32x!tt.ptr<f16>, #A>) -> tensor<128x32xf16, #A> {
145|   %init = arith.constant dense<0.00e+00> : tensor<128x32xf16, #A>
146|   %a_off = arith.constant dense<4> : tensor<128x32xi32, #A>
147|   %loop = scf.for %iv = %lb to %ub step %step iter_args(%acc = %init) -> (tensor<128x32xf16, #A>) {
```
**EN:** This function-oriented block defines or enters `independent_epilogue_if`. Within it, the test exercises tt.func, constants, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `independent_epilogue_if` 为核心。测试在其中演示 tt.func、常量、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 148-149
```mlir
148|     // CHECK: tt.load {{.*}} {loop.cluster = 2 : i32, loop.stage = 0 : i32}
149|     %a = tt.load %a_ptr_init {tt.latency = 2 : i32} : tensor<128x32x!tt.ptr<f16>, #A>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 150-151
```mlir
150|     // CHECK: arith.addf {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
151|     %res = arith.addf %acc, %a : tensor<128x32xf16, #A>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 152-161
```mlir
152|     // CHECK: scf.if
153|     // CHECK: {loop.cluster = 4 : i32, loop.stage = 2 : i32}
154|     scf.if %cnd {
155|       tt.store %a_ptr_init, %init : tensor<128x32x!tt.ptr<f16>, #A>
156|     }
157|     scf.yield %res : tensor<128x32xf16, #A>
158|   }
159|   tt.return %loop#0 : tensor<128x32xf16, #A>
160| }
161| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured conditionals, masked or vectorized stores, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化条件分支、带掩码或向量化的存储、循环/分支产出值、tt.return。

### Lines 162-162
```mlir
162| // CHECK-LABEL: @independent_last_stage
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @independent_last_stage anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @independent_last_stage 这样的标签用于锚定匹配范围。

### Lines 163-166
```mlir
163| tt.func @independent_last_stage(%lb : index, %ub : index, %step : index,
164|                  %a_ptr_init : tensor<128x32x!tt.ptr<f16>, #A>) -> (tensor<128x32xf16, #A>, tensor<128x32xf16, #A>) {
165|   %init = arith.constant dense<0.00e+00> : tensor<128x32xf16, #A>
166|   %loop:2 = scf.for %iv = %lb to %ub step %step iter_args(%acc = %init, %acc2 = %init) -> (tensor<128x32xf16, #A>, tensor<128x32xf16, #A>) {
```
**EN:** This function-oriented block defines or enters `independent_last_stage`. Within it, the test exercises tt.func, constants, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `independent_last_stage` 为核心。测试在其中演示 tt.func、常量、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 167-168
```mlir
167|     // CHECK: tt.load {{.*}} {loop.cluster = 2 : i32, loop.stage = 0 : i32}
168|     %a = tt.load %a_ptr_init {tt.latency = 2 : i32} : tensor<128x32x!tt.ptr<f16>, #A>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 169-170
```mlir
169|     // CHECK: arith.addf {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
170|     %res = arith.addf %acc, %a : tensor<128x32xf16, #A>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 171-177
```mlir
171|     // CHECK: arith.addf {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
172|     %res2 = arith.addf %acc2, %init : tensor<128x32xf16, #A>
173|     scf.yield %res, %res2 : tensor<128x32xf16, #A>, tensor<128x32xf16, #A>
174|   }
175|   tt.return %loop#0, %loop#1 : tensor<128x32xf16, #A>, tensor<128x32xf16, #A>
176| }
177| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining floating-point additions, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 浮点加法、循环/分支产出值、tt.return。

### Lines 178-178
```mlir
178| // CHECK-LABEL: @basic_pipeline
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @basic_pipeline anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @basic_pipeline 这样的标签用于锚定匹配范围。

### Lines 179-186
```mlir
179| tt.func @basic_pipeline(%lb : index, %ub : index, %step : index,
180|                   %a_ptr_init : tensor<128x32x!tt.ptr<f16>, #AL>,
181|                   %b_ptr_init : tensor<32x128x!tt.ptr<f16>, #BL>) -> tensor<128x128xf32, #C> {
182|   %c_init = arith.constant dense<0.00e+00> : tensor<128x128xf32, #C>
183|   %a_off = arith.constant dense<4> : tensor<128x32xi32, #AL>
184|   %b_off = arith.constant dense<4> : tensor<32x128xi32, #BL>
185| 
186|   %loop:3 = scf.for %iv = %lb to %ub step %step iter_args(%a_ptr = %a_ptr_init, %b_ptr = %b_ptr_init, %prev_c = %c_init) -> (tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>) {
```
**EN:** This function-oriented block defines or enters `basic_pipeline`. Within it, the test exercises constants, tt.func, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `basic_pipeline` 为核心。测试在其中演示 常量、tt.func、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 187-188
```mlir
187|     // CHECK: tt.load {{.*}} {loop.cluster = 3 : i32, loop.stage = 0 : i32}
188|     %a_ = tt.load %a_ptr {tt.latency = 2 : i32} : tensor<128x32x!tt.ptr<f16>, #AL>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 189-190
```mlir
189|     // CHECK: ttg.convert_layout {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
190|     %a = ttg.convert_layout %a_ : tensor<128x32xf16, #AL> -> tensor<128x32xf16, #A>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 191-192
```mlir
191|     // CHECK: tt.load {{.*}} {loop.cluster = 3 : i32, loop.stage = 0 : i32}
192|     %b_ = tt.load %b_ptr {tt.latency = 2 : i32} : tensor<32x128x!tt.ptr<f16>, #BL>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 193-195
```mlir
193|     // CHECK: ttg.convert_layout {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
194|     %b = ttg.convert_layout %b_ : tensor<32x128xf16, #BL> -> tensor<32x128xf16, #B>
195| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 196-197
```mlir
196|     // CHECK: tt.dot {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
197|     %c = tt.dot %a, %b, %prev_c : tensor<128x32xf16, #A> * tensor<32x128xf16, #B> -> tensor<128x128xf32, #C>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 198-199
```mlir
198|     // CHECK: tt.addptr {{.*}} {loop.cluster = 2 : i32, loop.stage = 1 : i32}
199|     %next_a_ptr = tt.addptr %a_ptr, %a_off : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xi32, #AL>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 200-206
```mlir
200|     // CHECK: tt.addptr {{.*}} {loop.cluster = 2 : i32, loop.stage = 1 : i32}
201|     %next_b_ptr = tt.addptr %b_ptr, %b_off : tensor<32x128x!tt.ptr<f16>, #BL>, tensor<32x128xi32, #BL>
202|     scf.yield %next_a_ptr, %next_b_ptr, %c : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>
203|   }
204|   tt.return %loop#2: tensor<128x128xf32, #C>
205| }
206| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining pointer arithmetic, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 指针算术、循环/分支产出值、tt.return。

### Lines 207-207
```mlir
207| // CHECK-LABEL: @unpipelined_load
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @unpipelined_load anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @unpipelined_load 这样的标签用于锚定匹配范围。

### Lines 208-215
```mlir
208| tt.func @unpipelined_load(%lb : index, %ub : index, %step : index,
209|                   %a_ptr_init : tensor<128x32x!tt.ptr<f16>, #AL>,
210|                   %b_ptr_init : tensor<32x128x!tt.ptr<f16>, #BL>) -> tensor<128x128xf32, #C> {
211|   %c_init = arith.constant dense<0.00e+00> : tensor<128x128xf32, #C>
212|   %a_off = arith.constant dense<4> : tensor<128x32xi32, #AL>
213|   %b_off = arith.constant dense<4> : tensor<32x128xi32, #BL>
214| 
215|   %loop:3 = scf.for %iv = %lb to %ub step %step iter_args(%a_ptr = %a_ptr_init, %b_ptr = %b_ptr_init, %prev_c = %c_init) -> (tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>) {
```
**EN:** This function-oriented block defines or enters `unpipelined_load`. Within it, the test exercises constants, tt.func, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `unpipelined_load` 为核心。测试在其中演示 常量、tt.func、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 216-217
```mlir
216|     // CHECK: tt.load {{.*}} {loop.cluster = 3 : i32, loop.stage = 0 : i32}
217|     %a_ = tt.load %a_ptr {tt.latency = 2 : i32} : tensor<128x32x!tt.ptr<f16>, #AL>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 218-220
```mlir
218|     // CHECK: ttg.convert_layout {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
219|     %a = ttg.convert_layout %a_ : tensor<128x32xf16, #AL> -> tensor<128x32xf16, #A>
220|     // load below should be in the same stage as tt.dot (not pipelined)
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 221-222
```mlir
221|     // CHECK: tt.load {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
222|     %b_ = tt.load %b_ptr : tensor<32x128x!tt.ptr<f16>, #BL>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 223-225
```mlir
223|     // CHECK: ttg.convert_layout {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
224|     %b = ttg.convert_layout %b_ : tensor<32x128xf16, #BL> -> tensor<32x128xf16, #B>
225| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 226-227
```mlir
226|     // CHECK: tt.dot {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
227|     %c = tt.dot %a, %b, %prev_c : tensor<128x32xf16, #A> * tensor<32x128xf16, #B> -> tensor<128x128xf32, #C>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 228-230
```mlir
228|     // CHECK: tt.addptr {{.*}} {loop.cluster = 2 : i32, loop.stage = 1 : i32}
229|     %next_a_ptr = tt.addptr %a_ptr, %a_off : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xi32, #AL>
230|     // addptr below should be scheduled to the last stage
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 231-237
```mlir
231|     // CHECK: tt.addptr {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
232|     %next_b_ptr = tt.addptr %b_ptr, %b_off : tensor<32x128x!tt.ptr<f16>, #BL>, tensor<32x128xi32, #BL>
233|     scf.yield %next_a_ptr, %next_b_ptr, %c : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>
234|   }
235|   tt.return %loop#2: tensor<128x128xf32, #C>
236| }
237| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining pointer arithmetic, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 指针算术、循环/分支产出值、tt.return。

### Lines 238-238
```mlir
238| // CHECK-LABEL: @epilogue_if
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @epilogue_if anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @epilogue_if 这样的标签用于锚定匹配范围。

### Lines 239-247
```mlir
239| tt.func @epilogue_if(%lb : index, %ub : index, %step : index, %cnd : i1,
240|                   %a_ptr_init : tensor<128x32x!tt.ptr<f16>, #AL>,
241|                   %b_ptr_init : tensor<32x128x!tt.ptr<f16>, #BL>,
242|                   %c_ptr_store : tensor<128x128x!tt.ptr<f32>, #C>) -> tensor<128x128xf32, #C> {
243|   %c_init = arith.constant dense<0.00e+00> : tensor<128x128xf32, #C>
244|   %a_off = arith.constant dense<4> : tensor<128x32xi32, #AL>
245|   %b_off = arith.constant dense<4> : tensor<32x128xi32, #BL>
246| 
247|   %loop:3 = scf.for %iv = %lb to %ub step %step iter_args(%a_ptr = %a_ptr_init, %b_ptr = %b_ptr_init, %prev_c = %c_init) -> (tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>) {
```
**EN:** This function-oriented block defines or enters `epilogue_if`. Within it, the test exercises constants, tt.func, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `epilogue_if` 为核心。测试在其中演示 常量、tt.func、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 248-249
```mlir
248|     // CHECK: tt.load {{.*}} {loop.cluster = 3 : i32, loop.stage = 0 : i32}
249|     %a_ = tt.load %a_ptr {tt.latency = 2 : i32} : tensor<128x32x!tt.ptr<f16>, #AL>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 250-251
```mlir
250|     // CHECK: ttg.convert_layout {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
251|     %a = ttg.convert_layout %a_ : tensor<128x32xf16, #AL> -> tensor<128x32xf16, #A>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 252-253
```mlir
252|     // CHECK: tt.load {{.*}} {loop.cluster = 3 : i32, loop.stage = 0 : i32}
253|     %b_ = tt.load %b_ptr {tt.latency = 2 : i32} : tensor<32x128x!tt.ptr<f16>, #BL>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 254-256
```mlir
254|     // CHECK: ttg.convert_layout {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
255|     %b = ttg.convert_layout %b_ : tensor<32x128xf16, #BL> -> tensor<32x128xf16, #B>
256| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 257-258
```mlir
257|     // CHECK: tt.dot {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
258|     %c = tt.dot %a, %b, %prev_c : tensor<128x32xf16, #A> * tensor<32x128xf16, #B> -> tensor<128x128xf32, #C>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 259-263
```mlir
259|     // CHECK: scf.if
260|     // CHECK: {loop.cluster = 4 : i32, loop.stage = 2 : i32}
261|     scf.if %cnd {
262|       tt.store %c_ptr_store, %c : tensor<128x128x!tt.ptr<f32>, #C>
263|     }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured conditionals, masked or vectorized stores.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化条件分支、带掩码或向量化的存储。

### Lines 264-265
```mlir
264|     // CHECK: tt.addptr {{.*}} {loop.cluster = 2 : i32, loop.stage = 1 : i32}
265|     %next_a_ptr = tt.addptr %a_ptr, %a_off : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xi32, #AL>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 266-272
```mlir
266|     // CHECK: tt.addptr {{.*}} {loop.cluster = 2 : i32, loop.stage = 1 : i32}
267|     %next_b_ptr = tt.addptr %b_ptr, %b_off : tensor<32x128x!tt.ptr<f16>, #BL>, tensor<32x128xi32, #BL>
268|     scf.yield %next_a_ptr, %next_b_ptr, %c : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>
269|   }
270|   tt.return %loop#2: tensor<128x128xf32, #C>
271| }
272| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining pointer arithmetic, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 指针算术、循环/分支产出值、tt.return。

### Lines 273-273
```mlir
273| // CHECK-LABEL: @intermediate_use
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @intermediate_use anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @intermediate_use 这样的标签用于锚定匹配范围。

### Lines 274-282
```mlir
274| tt.func @intermediate_use(%lb : index, %ub : index, %step : index,
275|                   %a_ptr_init : tensor<128x32x!tt.ptr<f16>, #AL>,
276|                   %b_ptr_init : tensor<32x128x!tt.ptr<f16>, #BL>) -> tensor<128x128xf32, #C> {
277|   %c_init = arith.constant dense<0.00e+00> : tensor<128x128xf32, #C>
278|   %a_off = arith.constant dense<4> : tensor<128x32xi32, #AL>
279|   %b_off = arith.constant dense<4> : tensor<32x128xi32, #BL>
280|   %c2 = arith.constant dense<2.00> : tensor<32x128xf16, #BL>
281| 
282|   %loop:3 = scf.for %iv = %lb to %ub step %step iter_args(%a_ptr = %a_ptr_init, %b_ptr = %b_ptr_init, %prev_c = %c_init) -> (tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>) {
```
**EN:** This function-oriented block defines or enters `intermediate_use`. Within it, the test exercises constants, tt.func, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `intermediate_use` 为核心。测试在其中演示 常量、tt.func、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 283-284
```mlir
283|     // CHECK: tt.load {{.*}} {loop.cluster = 3 : i32, loop.stage = 0 : i32}
284|     %a_ = tt.load %a_ptr {tt.latency = 2 : i32} : tensor<128x32x!tt.ptr<f16>, #AL>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 285-286
```mlir
285|     // CHECK: ttg.convert_layout {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
286|     %a = ttg.convert_layout %a_ : tensor<128x32xf16, #AL> -> tensor<128x32xf16, #A>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 287-288
```mlir
287|     // CHECK: tt.load {{.*}} {loop.cluster = 3 : i32, loop.stage = 0 : i32}
288|     %b_ = tt.load %b_ptr {tt.latency = 2 : i32} : tensor<32x128x!tt.ptr<f16>, #BL>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 289-290
```mlir
289|     // CHECK: arith.mulf {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
290|     %b_2 = arith.mulf %b_ , %c2 : tensor<32x128xf16, #BL>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 291-293
```mlir
291|     // CHECK: ttg.convert_layout {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
292|     %b = ttg.convert_layout %b_2 : tensor<32x128xf16, #BL> -> tensor<32x128xf16, #B>
293| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 294-295
```mlir
294|     // CHECK: tt.dot {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
295|     %c = tt.dot %a, %b, %prev_c : tensor<128x32xf16, #A> * tensor<32x128xf16, #B> -> tensor<128x128xf32, #C>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 296-297
```mlir
296|     // CHECK: tt.addptr {{.*}} {loop.cluster = 2 : i32, loop.stage = 1 : i32}
297|     %next_a_ptr = tt.addptr %a_ptr, %a_off : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xi32, #AL>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 298-304
```mlir
298|     // CHECK: tt.addptr {{.*}} {loop.cluster = 2 : i32, loop.stage = 1 : i32}
299|     %next_b_ptr = tt.addptr %b_ptr, %b_off : tensor<32x128x!tt.ptr<f16>, #BL>, tensor<32x128xi32, #BL>
300|     scf.yield %next_a_ptr, %next_b_ptr, %c : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>
301|   }
302|   tt.return %loop#2: tensor<128x128xf32, #C>
303| }
304| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining pointer arithmetic, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 指针算术、循环/分支产出值、tt.return。

### Lines 305-305
```mlir
305| // CHECK-LABEL: @indirect_load
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @indirect_load anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @indirect_load 这样的标签用于锚定匹配范围。

### Lines 306-314
```mlir
306| tt.func @indirect_load(%lb : index, %ub : index, %step : index,
307|                   %a_ind_ptr_init : tensor<128x32x!tt.ptr<i32>, #AL>,
308|                   %a_ptr_init : tensor<128x32x!tt.ptr<f16>, #AL>,
309|                   %b_ptr_init : tensor<32x128x!tt.ptr<f16>, #BL>) -> tensor<128x128xf32, #C> {
310|   %c_init = arith.constant dense<0.00e+00> : tensor<128x128xf32, #C>
311|   %a_ind_off = arith.constant dense<4> : tensor<128x32xi32, #AL>
312|   %b_off = arith.constant dense<4> : tensor<32x128xi32, #BL>
313| 
314|   %loop:4 = scf.for %iv = %lb to %ub step %step iter_args(%a_ind_ptr = %a_ind_ptr_init, %a_ptr = %a_ptr_init, %b_ptr = %b_ptr_init, %prev_c = %c_init) -> (tensor<128x32x!tt.ptr<i32>, #AL>, tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>) {
```
**EN:** This function-oriented block defines or enters `indirect_load`. Within it, the test exercises constants, tt.func, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `indirect_load` 为核心。测试在其中演示 常量、tt.func、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 315-317
```mlir
315|     // CHECK: tt.load {{.*}} {loop.cluster = 4 : i32, loop.stage = 0 : i32}
316|     %a_off = tt.load %a_ind_ptr {tt.latency = 1 : i32} : tensor<128x32x!tt.ptr<i32>, #AL>
317|     %next_a_ind_ptr = tt.addptr %a_ind_ptr, %a_ind_off : tensor<128x32x!tt.ptr<i32>, #AL>, tensor<128x32xi32, #AL>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 318-320
```mlir
318|     // CHECK: tt.addptr {{.*}} {loop.cluster = 2 : i32, loop.stage = 1 : i32}
319|     %next_a_ptr = tt.addptr %a_ptr, %a_off : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xi32, #AL>
320|     // addptr below scheduled by scheduleDependencies to the same stage as tt.load that is using it
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 321-322
```mlir
321|     // CHECK: tt.addptr {{.*}} {loop.cluster = 2 : i32, loop.stage = 1 : i32}
322|     %next_b_ptr = tt.addptr %b_ptr, %b_off : tensor<32x128x!tt.ptr<f16>, #BL>, tensor<32x128xi32, #BL>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 323-324
```mlir
323|     // CHECK: tt.load {{.*}} {loop.cluster = 2 : i32, loop.stage = 1 : i32}
324|     %a_ = tt.load %next_a_ptr {tt.latency = 2 : i32} : tensor<128x32x!tt.ptr<f16>, #AL>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 325-326
```mlir
325|     // CHECK: ttg.convert_layout {{.*}} {loop.cluster = 0 : i32, loop.stage = 3 : i32}
326|     %a = ttg.convert_layout %a_ : tensor<128x32xf16, #AL> -> tensor<128x32xf16, #A>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 327-328
```mlir
327|     // CHECK: tt.load {{.*}} {loop.cluster = 2 : i32, loop.stage = 1 : i32}
328|     %b_ = tt.load %next_b_ptr {tt.latency = 2 : i32} : tensor<32x128x!tt.ptr<f16>, #BL>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 329-331
```mlir
329|     // CHECK: ttg.convert_layout {{.*}} {loop.cluster = 0 : i32, loop.stage = 3 : i32}
330|     %b = ttg.convert_layout %b_ : tensor<32x128xf16, #BL> -> tensor<32x128xf16, #B>
331| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 332-339
```mlir
332|     // CHECK: tt.dot {{.*}} {loop.cluster = 0 : i32, loop.stage = 3 : i32}
333|     %c = tt.dot %a, %b, %prev_c : tensor<128x32xf16, #A> * tensor<32x128xf16, #B> -> tensor<128x128xf32, #C>
334|     scf.yield %next_a_ind_ptr, %next_a_ptr, %next_b_ptr, %c : tensor<128x32x!tt.ptr<i32>, #AL>, tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>
335|   }
336|   tt.return %loop#3: tensor<128x128xf32, #C>
337| }
338| 
339| // Verify that we don't schedule/pipeline loops with barrier
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining dot-product or MMA-style math, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 点积或 MMA 风格计算、循环/分支产出值、tt.return。

### Lines 340-340
```mlir
340| // CHECK-LABEL: @gpu_barrier
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @gpu_barrier anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @gpu_barrier 这样的标签用于锚定匹配范围。

### Lines 341-344
```mlir
341| tt.func @gpu_barrier(%lb : index, %ub : index, %step : index,
342|                  %a_ptr_init : tensor<128x32x!tt.ptr<f16>, #A>) -> tensor<128x32xf16, #A> {
343|   %init = arith.constant dense<0.00e+00> : tensor<128x32xf16, #A>
344|   %loop = scf.for %iv = %lb to %ub step %step iter_args(%acc = %init) -> (tensor<128x32xf16, #A>) {
```
**EN:** This function-oriented block defines or enters `gpu_barrier`. Within it, the test exercises tt.func, constants, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `gpu_barrier` 为核心。测试在其中演示 tt.func、常量、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 345-354
```mlir
345|     // CHECK-NOT: loop.cluster
346|     %a = tt.load %a_ptr_init {tt.latency = 2 : i32} : tensor<128x32x!tt.ptr<f16>, #A>
347|     %res = arith.addf %acc, %a : tensor<128x32xf16, #A>
348|     ttg.barrier local
349|     scf.yield %res : tensor<128x32xf16, #A>
350|   }
351|   tt.return %loop#0 : tensor<128x32xf16, #A>
352| }
353| }
354| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining masked or vectorized loads, floating-point additions, ttg.barrier, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 带掩码或向量化的加载、浮点加法、ttg.barrier、循环/分支产出值、tt.return。

### Lines 355-355
```mlir
355| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 356-361
```mlir
356| 
357| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
358| #blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
359| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
360| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
361| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 362-362
```mlir
362| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 363-363
```mlir
363| // CHECK-LABEL: @tc_gen5_mma
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tc_gen5_mma anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tc_gen5_mma 这样的标签用于锚定匹配范围。

### Lines 364-369
```mlir
364| tt.func @tc_gen5_mma(%lb : index, %ub : index, %step : index,
365|                   %A_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = 16 : i32, tt.contiguity = 16 : i32},
366|                   %B: tensor<128x128xf16, #blocked1>,
367|                   %acc_tm : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory>) -> () {
368|   %true = arith.constant true
369|   scf.for %iv = %lb to %ub step %step : index {
```
**EN:** This function-oriented block defines or enters `tc_gen5_mma`. Within it, the test exercises tt.func, constants, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tc_gen5_mma` 为核心。测试在其中演示 tt.func、常量、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 370-371
```mlir
370|     // CHECK: tt.load {{.*}} {loop.cluster = 3 : i32, loop.stage = 0 : i32}
371|     %A = tt.load %A_ptr {tt.latency = 2 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked1>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 372-373
```mlir
372|     // CHECK: ttg.local_alloc {{.*}} {loop.cluster = 1 : i32, loop.stage = 2 : i32}
373|     %A_sh = ttg.local_alloc %A : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 374-375
```mlir
374|     // CHECK: ttg.local_alloc {{.*}} {loop.cluster = 1 : i32, loop.stage = 2 : i32}
375|     %B_sh = ttg.local_alloc %B : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 376-377
```mlir
376|     // CHECK: ttng.tc_gen5_mma {{.*}} {loop.cluster = 1 : i32, loop.stage = 2 : i32, tt.self_latency = 1 : i32}
377|     %mma_tok = ttng.tc_gen5_mma %A_sh, %B_sh, %acc_tm[], %true, %true {tt.latency = 1 : i32, tt.self_latency = 1 : i32} : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 378-379
```mlir
378|     // CHECK: ttng.tmem_load {{.*}} {loop.cluster = 0 : i32, loop.stage = 3 : i32}
379|     %c, %load_tok = ttng.tmem_load %acc_tm[%mma_tok] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory> -> tensor<128x128xf32, #blocked1>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 380-386
```mlir
380|     // CHECK: "use"{{.*}} {loop.cluster = 0 : i32, loop.stage = 3 : i32}
381|     "use"(%c) : (tensor<128x128xf32, #blocked1>) -> ()
382|   }
383|   tt.return
384| }
385| }
386| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 387-387
```mlir
387| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 388-393
```mlir
388| 
389| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
390| #blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
391| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
392| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
393| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 394-394
```mlir
394| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 395-395
```mlir
395| // CHECK-LABEL: @tc_gen5_mma_if_user
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tc_gen5_mma_if_user anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tc_gen5_mma_if_user 这样的标签用于锚定匹配范围。

### Lines 396-402
```mlir
396| tt.func @tc_gen5_mma_if_user(%lb : index, %ub : index, %step : index,
397|                   %A_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = 16 : i32, tt.contiguity = 16 : i32},
398|                   %B: tensor<128x128xf16, #blocked1>,
399|                   %acc_tm : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory>,
400|                   %cnd: i1) -> () {
401|   %true = arith.constant true
402|   scf.for %iv = %lb to %ub step %step : index {
```
**EN:** This function-oriented block defines or enters `tc_gen5_mma_if_user`. Within it, the test exercises tt.func, constants, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tc_gen5_mma_if_user` 为核心。测试在其中演示 tt.func、常量、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 403-404
```mlir
403|     // CHECK: tt.load {{.*}} {loop.cluster = 3 : i32, loop.stage = 0 : i32}
404|     %A = tt.load %A_ptr {tt.latency = 2 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked1>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 405-406
```mlir
405|     // CHECK: ttg.local_alloc {{.*}} {loop.cluster = 1 : i32, loop.stage = 2 : i32}
406|     %A_sh = ttg.local_alloc %A : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 407-408
```mlir
407|     // CHECK: ttg.local_alloc {{.*}} {loop.cluster = 1 : i32, loop.stage = 2 : i32}
408|     %B_sh = ttg.local_alloc %B : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 409-414
```mlir
409|     // CHECK: ttng.tc_gen5_mma {{.*}} {loop.cluster = 1 : i32, loop.stage = 2 : i32, tt.self_latency = 1 : i32}
410|     %mma_tok = ttng.tc_gen5_mma %A_sh, %B_sh, %acc_tm[], %true, %true {tt.latency = 1 : i32, tt.self_latency = 1 : i32} : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory>
411|     scf.if %cnd {
412|       %c, %load_tok = ttng.tmem_load %acc_tm[%mma_tok] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory> -> tensor<128x128xf32, #blocked1>
413|       "use"(%c) : (tensor<128x128xf32, #blocked1>) -> ()
414|     }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining Gen5 tensor-core MMA ops, structured conditionals, ttng.tmem_load.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 Gen5 张量核 MMA 操作、结构化条件分支、ttng.tmem_load。

### Lines 415-424
```mlir
415|     // CHECK: scf.if
416|     // CHECK: tmem_load
417|     // CHECK: "use"{{.*}}
418|     // CHECK-NOT: loop.cluster
419|     // CHECK: } {loop.cluster = 4 : i32, loop.stage = 3 : i32}
420|   }
421|   tt.return
422| }
423| }
424| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 tt.return。

### Lines 425-425
```mlir
425| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 426-432
```mlir
426| 
427| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
428| #blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
429| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
430| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [4, 3, 2, 1, 0]}>
431| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
432| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 433-433
```mlir
433| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 434-434
```mlir
434| // CHECK-LABEL: @tc_gen5_mma_scaled
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tc_gen5_mma_scaled anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tc_gen5_mma_scaled 这样的标签用于锚定匹配范围。

### Lines 435-442
```mlir
435| tt.func @tc_gen5_mma_scaled(%lb : index, %ub : index, %step : index,
436|                   %A_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = 16 : i32, tt.contiguity = 16 : i32},
437|                   %B: tensor<128x128xf16, #blocked1>,
438|                   %A_sc_sh: !ttg.memdesc<1x2x32x4x4xi8, #shared1, #ttg.shared_memory>,
439|                   %B_sc_sh: !ttg.memdesc<1x2x32x4x4xi8, #shared1, #ttg.shared_memory>,
440|                   %acc_tm : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory>) -> () {
441|   %true = arith.constant true
442|   scf.for %iv = %lb to %ub step %step : index {
```
**EN:** This function-oriented block defines or enters `tc_gen5_mma_scaled`. Within it, the test exercises tt.func, constants, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tc_gen5_mma_scaled` 为核心。测试在其中演示 tt.func、常量、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 443-444
```mlir
443|     // CHECK: tt.load {{.*}} {loop.cluster = 3 : i32, loop.stage = 0 : i32}
444|     %A = tt.load %A_ptr {tt.latency = 2 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked1>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 445-446
```mlir
445|     // CHECK: ttg.local_alloc {{.*}} {loop.cluster = 1 : i32, loop.stage = 2 : i32}
446|     %A_sh = ttg.local_alloc %A : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 447-448
```mlir
447|     // CHECK: ttg.local_alloc {{.*}} {loop.cluster = 1 : i32, loop.stage = 2 : i32}
448|     %B_sh = ttg.local_alloc %B : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 449-450
```mlir
449|     // CHECK: ttng.tc_gen5_mma_scaled {{.*}} {loop.cluster = 1 : i32, loop.stage = 2 : i32, tt.self_latency = 1 : i32}
450|     %mma_tok = ttng.tc_gen5_mma_scaled %A_sh, %B_sh, %acc_tm[], %A_sc_sh, %B_sc_sh, %true, %true lhs = e5m2 rhs = e5m2 {tt.latency = 1 : i32, tt.self_latency = 1 : i32} : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory>, !ttg.memdesc<1x2x32x4x4xi8, #shared1, #ttg.shared_memory>, !ttg.memdesc<1x2x32x4x4xi8, #shared1, #ttg.shared_memory>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 451-452
```mlir
451|     // CHECK: ttng.tmem_load {{.*}} {loop.cluster = 0 : i32, loop.stage = 3 : i32}
452|     %c, %load_tok = ttng.tmem_load %acc_tm[%mma_tok] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory> -> tensor<128x128xf32, #blocked1>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 453-459
```mlir
453|     // CHECK: "use"{{.*}} {loop.cluster = 0 : i32, loop.stage = 3 : i32}
454|     "use"(%c) : (tensor<128x128xf32, #blocked1>) -> ()
455|   }
456|   tt.return
457| }
458| }
459| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 460-460
```mlir
460| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 461-466
```mlir
461| 
462| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
463| #blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
464| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
465| #smem = #ttg.shared_memory
466| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 467-467
```mlir
467| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 468-468
```mlir
468|   // CHECK-LABEL: @select_after_mma
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @select_after_mma anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @select_after_mma 这样的标签用于锚定匹配范围。

### Lines 469-483
```mlir
469|   tt.func public @select_after_mma(%arg0: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = 16 : i32, tt.divisibility = 16 : i32}, %arg1: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = 16 : i32, tt.divisibility = 16 : i32}, %arg2: i32) -> tensor<128x128xf16, #blocked1> {
470|     %true = arith.constant true
471|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked1>
472|     %cst_0 = arith.constant dense<2.000000e+00> : tensor<128x128xf32, #blocked1>
473|     %c0_i32 = arith.constant 0 : i32
474|     %c1_i32 = arith.constant 1 : i32
475|     %0 = "cnd"() : () -> i1
476|     %1, %acc_tok = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
477|     %init_tok = ttng.tmem_store %cst, %1[%acc_tok], %true : tensor<128x128xf32, #blocked1> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
478|     %last_tok = scf.for %arg3 = %c0_i32 to %arg2 step %c1_i32 iter_args(%tok = %init_tok) -> !ttg.async.token : i32 {
479|       %4 = tt.load %arg0 {tt.latency = 2 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
480|       %5 = ttg.local_alloc %4 : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
481|       %6 = tt.load %arg1 {tt.latency = 2 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
482|       %7 = ttg.local_alloc %6 : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
483|       %mma_tok = ttng.tc_gen5_mma %5, %7, %1[%tok], %true, %true {tt.latency = 1 : i32, tt.self_latency = 1 : i32} : !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** This function-oriented block defines or enters `select_after_mma`. Within it, the test exercises constants, tt.func, masked or vectorized loads, shared/local memory allocation, tensor-memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `select_after_mma` 为核心。测试在其中演示 常量、tt.func、带掩码或向量化的加载、共享/本地内存分配、张量内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 484-485
```mlir
484|       // CHECK: arith.xori {{.*}} {loop.cluster = 0 : i32, loop.stage = 3 : i32}
485|       %8 = arith.xori %0, %true : i1
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 486-495
```mlir
486|       // CHECK: ttng.tmem_store {{.*}} {loop.cluster = 0 : i32, loop.stage = 3 : i32}
487|       %store_tok = ttng.tmem_store %cst_0, %1[%mma_tok], %8 : tensor<128x128xf32, #blocked1> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
488|       scf.yield %store_tok : !ttg.async.token
489|     } {tt.scheduled_max_stage = 3 : i32}
490|     %2, %res_tok = ttng.tmem_load %1[%last_tok] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
491|     %3 = arith.truncf %2 : tensor<128x128xf32, #blocked1> to tensor<128x128xf16, #blocked1>
492|     tt.return %3 : tensor<128x128xf16, #blocked1>
493|   }
494| }
495| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining ttng.tmem_store, loop/if yielded values, ttng.tmem_load, arith.truncf, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 ttng.tmem_store、循环/分支产出值、ttng.tmem_load、arith.truncf、tt.return。

### Lines 496-496
```mlir
496| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 497-502
```mlir
497| 
498| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
499| #blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
500| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
501| #smem = #ttg.shared_memory
502| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 503-503
```mlir
503| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 504-504
```mlir
504|   // CHECK-LABEL: @select_before_mma
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @select_before_mma anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @select_before_mma 这样的标签用于锚定匹配范围。

### Lines 505-514
```mlir
505|   tt.func public @select_before_mma(%arg0: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = 16 : i32, tt.divisibility = 16 : i32}, %arg1: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = 16 : i32, tt.divisibility = 16 : i32}, %arg2: i32) -> tensor<128x128xf16, #blocked1> {
506|     %true = arith.constant true
507|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked1>
508|     %cst_0 = arith.constant dense<2.000000e+00> : tensor<128x128xf32, #blocked1>
509|     %c0_i32 = arith.constant 0 : i32
510|     %c1_i32 = arith.constant 1 : i32
511|     %0 = "cnd"() : () -> i1
512|     %1, %acc_tok = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
513|     %init_tok = ttng.tmem_store %cst, %1[%acc_tok], %true : tensor<128x128xf32, #blocked1> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
514|     %last_tok = scf.for %arg3 = %c0_i32 to %arg2 step %c1_i32 iter_args(%tok = %init_tok) -> !ttg.async.token : i32 {
```
**EN:** This function-oriented block defines or enters `select_before_mma`. Within it, the test exercises constants, tt.func, tensor-memory allocation, ttng.tmem_store, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `select_before_mma` 为核心。测试在其中演示 常量、tt.func、张量内存分配、ttng.tmem_store、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 515-516
```mlir
515|       // CHECK: arith.xori {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
516|       %8 = arith.xori %0, %true : i1
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 517-522
```mlir
517|       // CHECK: ttng.tmem_store {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
518|       %store_tok = ttng.tmem_store %cst_0, %1[%tok], %8 : tensor<128x128xf32, #blocked1> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
519|       %4 = tt.load %arg0 {tt.latency = 2 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
520|       %5 = ttg.local_alloc %4 : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
521|       %6 = tt.load %arg1 {tt.latency = 2 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
522|       %7 = ttg.local_alloc %6 : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized loads, shared/local memory allocation, ttng.tmem_store. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的加载、共享/本地内存分配、ttng.tmem_store。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 523-532
```mlir
523|       // CHECK: ttng.tc_gen5_mma {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32, tt.self_latency = 1 : i32}
524|       %mma_tok = ttng.tc_gen5_mma %5, %7, %1[%store_tok], %true, %true {tt.latency = 1 : i32, tt.self_latency = 1 : i32} : !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
525|       scf.yield %mma_tok : !ttg.async.token
526|     } {tt.scheduled_max_stage = 3 : i32}
527|     %2, %res_tok = ttng.tmem_load %1[%last_tok] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
528|     %3 = arith.truncf %2 : tensor<128x128xf32, #blocked1> to tensor<128x128xf16, #blocked1>
529|     tt.return %3 : tensor<128x128xf16, #blocked1>
530|   }
531| }
532| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining Gen5 tensor-core MMA ops, loop/if yielded values, ttng.tmem_load, arith.truncf, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 Gen5 张量核 MMA 操作、循环/分支产出值、ttng.tmem_load、arith.truncf、tt.return。

### Lines 533-533
```mlir
533| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 534-538
```mlir
534| 
535| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
536| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
537| #smem = #ttg.shared_memory
538| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 539-539
```mlir
539| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 540-540
```mlir
540|   // CHECK-LABEL: @two_dots
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @two_dots anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @two_dots 这样的标签用于锚定匹配范围。

### Lines 541-547
```mlir
541|   tt.func public @two_dots(%arg0: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = 16 : i32, tt.divisibility = 16 : i32}, %arg1: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = 16 : i32, tt.divisibility = 16 : i32}, %arg2: tensor<128x128x!tt.ptr<f32>, #blocked> {tt.contiguity = 16 : i32, tt.divisibility = 16 : i32}, %arg3: tensor<128x128x!tt.ptr<f32>, #blocked> {tt.contiguity = 16 : i32, tt.divisibility = 16 : i32}, %arg4: i32) {
542|     %true = arith.constant true
543|     %c0_i32 = arith.constant 0 : i32
544|     %c1_i32 = arith.constant 1 : i32
545|     %0, %acc_tok0 = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
546|     %1, %acc_tok1 = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
547|     %last_tok:2 = scf.for %arg5 = %c0_i32 to %arg4 step %c1_i32 iter_args(%tok0 = %acc_tok0, %tok1 = %acc_tok1) -> (!ttg.async.token, !ttg.async.token) : i32 {
```
**EN:** This function-oriented block defines or enters `two_dots`. Within it, the test exercises constants, tt.func, tensor-memory allocation, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `two_dots` 为核心。测试在其中演示 常量、tt.func、张量内存分配、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 548-549
```mlir
548|       // CHECK: tt.load {{.*}} {loop.cluster = 4 : i32, loop.stage = 0 : i32}
549|       %2 = tt.load %arg0 {tt.latency = 2 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 550-551
```mlir
550|       // CHECK: ttg.local_alloc {{.*}} {loop.cluster = 2 : i32, loop.stage = 2 : i32}
551|       %3 = ttg.local_alloc %2 : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 552-553
```mlir
552|       // CHECK: tt.load {{.*}} {loop.cluster = 4 : i32, loop.stage = 0 : i32}
553|       %4 = tt.load %arg1 {tt.latency = 2 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 554-555
```mlir
554|       // CHECK: ttg.local_alloc {{.*}} {loop.cluster = 2 : i32, loop.stage = 2 : i32}
555|       %5 = ttg.local_alloc %4 : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 556-557
```mlir
556|       // CHECK: tt.load {{.*}} {loop.cluster = 2 : i32, loop.stage = 2 : i32}
557|       %6 = tt.load %arg2 : tensor<128x128x!tt.ptr<f32>, #blocked>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 558-559
```mlir
558|       // CHECK: ttng.tmem_store {{.*}} {loop.cluster = 2 : i32, loop.stage = 2 : i32}
559|       %store_tok0 = ttng.tmem_store %6, %0[%tok0], %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 560-561
```mlir
560|       // CHECK: ttng.tc_gen5_mma {{.*}} {loop.cluster = 2 : i32, loop.stage = 2 : i32, tt.self_latency = 1 : i32}
561|       %mma_tok0 = ttng.tc_gen5_mma %3, %5, %0[%store_tok0], %true, %true {tt.latency = 1 : i32, tt.self_latency = 1 : i32} : !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 562-563
```mlir
562|       // CHECK: ttng.tmem_load {{.*}} {loop.cluster = 1 : i32, loop.stage = 3 : i32}
563|       %7, %load_tok0 = ttng.tmem_load %0[%mma_tok0] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 564-565
```mlir
564|       // CHECK: ttng.tmem_store {{.*}} {loop.cluster = 1 : i32, loop.stage = 3 : i32}
565|       %store_tok1 = ttng.tmem_store %7, %1[%tok1], %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 566-567
```mlir
566|       // CHECK: ttng.tc_gen5_mma {{.*}} {loop.cluster = 1 : i32, loop.stage = 3 : i32, tt.self_latency = 1 : i32}
567|       %mma_tok1 = ttng.tc_gen5_mma %3, %5, %1[%store_tok1], %true, %true {tt.latency = 1 : i32, tt.self_latency = 1 : i32} : !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 568-569
```mlir
568|       // CHECK: ttng.tmem_load {{.*}} {loop.cluster = 0 : i32, loop.stage = 4 : i32}
569|       %8, %load_tok1 = ttng.tmem_load %1[%mma_tok1] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 570-577
```mlir
570|       // CHECK: tt.store {{.*}} {loop.cluster = 0 : i32, loop.stage = 4 : i32}
571|       tt.store %arg3, %8 : tensor<128x128x!tt.ptr<f32>, #blocked>
572|       scf.yield %load_tok0, %load_tok1 : !ttg.async.token, !ttg.async.token
573|     }
574|     tt.return
575|   }
576| }
577| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining masked or vectorized stores, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 带掩码或向量化的存储、循环/分支产出值、tt.return。

### Lines 578-578
```mlir
578| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 579-583
```mlir
579| 
580| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
581| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
582| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
583| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 584-584
```mlir
584| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 585-585
```mlir
585| // CHECK-LABEL: @tc_gen5_mma
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tc_gen5_mma anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tc_gen5_mma 这样的标签用于锚定匹配范围。

### Lines 586-591
```mlir
586| tt.func @tc_gen5_mma(%lb : index, %ub : index, %step : index,
587|                   %A_ptr: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.divisibility = 16 : i32, tt.contiguity = 16 : i32},
588|                   %B: tensor<128x128xf16, #blocked>,
589|                   %acc_tm : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory>) -> () {
590|   %true = arith.constant true
591|   scf.for %iv = %lb to %ub step %step : index {
```
**EN:** This function-oriented block defines or enters `tc_gen5_mma`. Within it, the test exercises tt.func, constants, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tc_gen5_mma` 为核心。测试在其中演示 tt.func、常量、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 592-593
```mlir
592|     // CHECK: tt.load {{.*}} {loop.cluster = 3 : i32, loop.stage = 0 : i32}
593|     %A = tt.load %A_ptr {tt.latency = 2 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 594-595
```mlir
594|     // CHECK: ttg.local_alloc {{.*}} {loop.cluster = 1 : i32, loop.stage = 2 : i32}
595|     %A_sh = ttg.local_alloc %A : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 596-597
```mlir
596|     // CHECK: ttg.local_alloc {{.*}} {loop.cluster = 1 : i32, loop.stage = 2 : i32}
597|     %B_sh = ttg.local_alloc %B : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 598-599
```mlir
598|     // CHECK: ttng.tc_gen5_mma {{.*}} {loop.cluster = 1 : i32, loop.stage = 2 : i32, tt.self_latency = 1 : i32}
599|     %mma_tok = ttng.tc_gen5_mma %A_sh, %B_sh, %acc_tm[], %true, %true {tt.latency = 1 : i32, tt.self_latency = 1 : i32} : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 600-601
```mlir
600|     // CHECK: ttng.tmem_load {{.*}} {loop.cluster = 0 : i32, loop.stage = 3 : i32}
601|     %c, %load_tok = ttng.tmem_load %acc_tm[%mma_tok] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory> -> tensor<128x128xf32, #blocked>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 602-608
```mlir
602|     // CHECK: "use"{{.*}} {loop.cluster = 0 : i32, loop.stage = 3 : i32}
603|     "use"(%c) : (tensor<128x128xf32, #blocked>) -> ()
604|   }
605|   tt.return
606| }
607| }
608| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 609-609
```mlir
609| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 610-614
```mlir
610| 
611| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
612| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
613| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
614| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 615-615
```mlir
615| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 616-616
```mlir
616| // CHECK-LABEL: @tc_gen5_mma_if_user
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tc_gen5_mma_if_user anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tc_gen5_mma_if_user 这样的标签用于锚定匹配范围。

### Lines 617-623
```mlir
617| tt.func @tc_gen5_mma_if_user(%lb : index, %ub : index, %step : index,
618|                   %A_ptr: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.divisibility = 16 : i32, tt.contiguity = 16 : i32},
619|                   %B: tensor<128x128xf16, #blocked>,
620|                   %acc_tm : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory>,
621|                   %cnd: i1) -> () {
622|   %true = arith.constant true
623|   scf.for %iv = %lb to %ub step %step : index {
```
**EN:** This function-oriented block defines or enters `tc_gen5_mma_if_user`. Within it, the test exercises tt.func, constants, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tc_gen5_mma_if_user` 为核心。测试在其中演示 tt.func、常量、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 624-625
```mlir
624|     // CHECK: tt.load {{.*}} {loop.cluster = 3 : i32, loop.stage = 0 : i32}
625|     %A = tt.load %A_ptr {tt.latency = 2 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 626-627
```mlir
626|     // CHECK: ttg.local_alloc {{.*}} {loop.cluster = 1 : i32, loop.stage = 2 : i32}
627|     %A_sh = ttg.local_alloc %A : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 628-629
```mlir
628|     // CHECK: ttg.local_alloc {{.*}} {loop.cluster = 1 : i32, loop.stage = 2 : i32}
629|     %B_sh = ttg.local_alloc %B : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 630-635
```mlir
630|     // CHECK: ttng.tc_gen5_mma {{.*}} {loop.cluster = 1 : i32, loop.stage = 2 : i32, tt.self_latency = 1 : i32}
631|     %mma_tok = ttng.tc_gen5_mma %A_sh, %B_sh, %acc_tm[], %true, %true {tt.latency = 1 : i32, tt.self_latency = 1 : i32} : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory>
632|     scf.if %cnd {
633|       %c, %load_tok = ttng.tmem_load %acc_tm[%mma_tok] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory> -> tensor<128x128xf32, #blocked>
634|       "use"(%c) : (tensor<128x128xf32, #blocked>) -> ()
635|     }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining Gen5 tensor-core MMA ops, structured conditionals, ttng.tmem_load.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 Gen5 张量核 MMA 操作、结构化条件分支、ttng.tmem_load。

### Lines 636-645
```mlir
636|     // CHECK: scf.if
637|     // CHECK: tmem_load
638|     // CHECK: "use"{{.*}}
639|     // CHECK-NOT: loop.cluster
640|     // CHECK: } {loop.cluster = 4 : i32, loop.stage = 3 : i32}
641|   }
642|   tt.return
643| }
644| }
645| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 tt.return。

### Lines 646-646
```mlir
646| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 647-652
```mlir
647| 
648| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
649| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
650| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [4, 3, 2, 1, 0]}>
651| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
652| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 653-653
```mlir
653| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 654-654
```mlir
654| // CHECK-LABEL: @tc_gen5_mma_scaled
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tc_gen5_mma_scaled anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tc_gen5_mma_scaled 这样的标签用于锚定匹配范围。

### Lines 655-662
```mlir
655| tt.func @tc_gen5_mma_scaled(%lb : index, %ub : index, %step : index,
656|                   %A_ptr: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.divisibility = 16 : i32, tt.contiguity = 16 : i32},
657|                   %B: tensor<128x128xf16, #blocked>,
658|                   %A_sc_sh: !ttg.memdesc<1x2x32x4x4xi8, #shared1, #ttg.shared_memory>,
659|                   %B_sc_sh: !ttg.memdesc<1x2x32x4x4xi8, #shared1, #ttg.shared_memory>,
660|                   %acc_tm : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory>) -> () {
661|   %true = arith.constant true
662|   scf.for %iv = %lb to %ub step %step : index {
```
**EN:** This function-oriented block defines or enters `tc_gen5_mma_scaled`. Within it, the test exercises tt.func, constants, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tc_gen5_mma_scaled` 为核心。测试在其中演示 tt.func、常量、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 663-664
```mlir
663|     // CHECK: tt.load {{.*}} {loop.cluster = 3 : i32, loop.stage = 0 : i32}
664|     %A = tt.load %A_ptr {tt.latency = 2 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 665-666
```mlir
665|     // CHECK: ttg.local_alloc {{.*}} {loop.cluster = 1 : i32, loop.stage = 2 : i32}
666|     %A_sh = ttg.local_alloc %A : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 667-668
```mlir
667|     // CHECK: ttg.local_alloc {{.*}} {loop.cluster = 1 : i32, loop.stage = 2 : i32}
668|     %B_sh = ttg.local_alloc %B : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 669-670
```mlir
669|     // CHECK: ttng.tc_gen5_mma_scaled {{.*}} {loop.cluster = 1 : i32, loop.stage = 2 : i32, tt.self_latency = 1 : i32}
670|     %mma_tok = ttng.tc_gen5_mma_scaled %A_sh, %B_sh, %acc_tm[], %A_sc_sh, %B_sc_sh, %true, %true lhs = e5m2 rhs = e5m2 {tt.latency = 1 : i32, tt.self_latency = 1 : i32} : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory>, !ttg.memdesc<1x2x32x4x4xi8, #shared1, #ttg.shared_memory>, !ttg.memdesc<1x2x32x4x4xi8, #shared1, #ttg.shared_memory>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 671-672
```mlir
671|     // CHECK: ttng.tmem_load {{.*}} {loop.cluster = 0 : i32, loop.stage = 3 : i32}
672|     %c, %load_tok = ttng.tmem_load %acc_tm[%mma_tok] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory> -> tensor<128x128xf32, #blocked>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 673-679
```mlir
673|     // CHECK: "use"{{.*}} {loop.cluster = 0 : i32, loop.stage = 3 : i32}
674|     "use"(%c) : (tensor<128x128xf32, #blocked>) -> ()
675|   }
676|   tt.return
677| }
678| }
679| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 680-680
```mlir
680| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 681-686
```mlir
681| 
682| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
683| #blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
684| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
685| #smem = #ttg.shared_memory
686| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 687-687
```mlir
687| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 688-688
```mlir
688|   // CHECK-LABEL: @select_after_mma
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @select_after_mma anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @select_after_mma 这样的标签用于锚定匹配范围。

### Lines 689-703
```mlir
689|   tt.func public @select_after_mma(%arg0: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = 16 : i32, tt.divisibility = 16 : i32}, %arg1: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = 16 : i32, tt.divisibility = 16 : i32}, %arg2: i32) -> tensor<128x128xf16, #blocked1> {
690|     %true = arith.constant true
691|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked1>
692|     %cst_0 = arith.constant dense<2.000000e+00> : tensor<128x128xf32, #blocked1>
693|     %c0_i32 = arith.constant 0 : i32
694|     %c1_i32 = arith.constant 1 : i32
695|     %0 = "cnd"() : () -> i1
696|     %1, %acc_tok = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
697|     %init_tok = ttng.tmem_store %cst, %1[%acc_tok], %true : tensor<128x128xf32, #blocked1> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
698|     %last_tok = scf.for %arg3 = %c0_i32 to %arg2 step %c1_i32 iter_args(%tok = %init_tok) -> !ttg.async.token : i32 {
699|       %4 = tt.load %arg0 {tt.latency = 2 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
700|       %5 = ttg.local_alloc %4 : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
701|       %6 = tt.load %arg1 {tt.latency = 2 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
702|       %7 = ttg.local_alloc %6 : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
703|       %mma_tok = ttng.tc_gen5_mma %5, %7, %1[%tok], %true, %true {tt.latency = 1 : i32, tt.self_latency = 1 : i32} : !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** This function-oriented block defines or enters `select_after_mma`. Within it, the test exercises constants, tt.func, masked or vectorized loads, shared/local memory allocation, tensor-memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `select_after_mma` 为核心。测试在其中演示 常量、tt.func、带掩码或向量化的加载、共享/本地内存分配、张量内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 704-705
```mlir
704|       // CHECK: arith.xori {{.*}} {loop.cluster = 0 : i32, loop.stage = 3 : i32}
705|       %8 = arith.xori %0, %true : i1
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 706-715
```mlir
706|       // CHECK: ttng.tmem_store {{.*}} {loop.cluster = 0 : i32, loop.stage = 3 : i32}
707|       %store_tok = ttng.tmem_store %cst_0, %1[%mma_tok], %8 : tensor<128x128xf32, #blocked1> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
708|       scf.yield %store_tok : !ttg.async.token
709|     } {tt.scheduled_max_stage = 3 : i32}
710|     %2, %res_tok = ttng.tmem_load %1[%last_tok] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
711|     %3 = arith.truncf %2 : tensor<128x128xf32, #blocked1> to tensor<128x128xf16, #blocked1>
712|     tt.return %3 : tensor<128x128xf16, #blocked1>
713|   }
714| }
715| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining ttng.tmem_store, loop/if yielded values, ttng.tmem_load, arith.truncf, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 ttng.tmem_store、循环/分支产出值、ttng.tmem_load、arith.truncf、tt.return。

### Lines 716-716
```mlir
716| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 717-722
```mlir
717| 
718| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
719| #blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
720| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
721| #smem = #ttg.shared_memory
722| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 723-723
```mlir
723| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 724-724
```mlir
724|   // CHECK-LABEL: @select_before_mma
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @select_before_mma anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @select_before_mma 这样的标签用于锚定匹配范围。

### Lines 725-734
```mlir
725|   tt.func public @select_before_mma(%arg0: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = 16 : i32, tt.divisibility = 16 : i32}, %arg1: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = 16 : i32, tt.divisibility = 16 : i32}, %arg2: i32) -> tensor<128x128xf16, #blocked1> {
726|     %true = arith.constant true
727|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked1>
728|     %cst_0 = arith.constant dense<2.000000e+00> : tensor<128x128xf32, #blocked1>
729|     %c0_i32 = arith.constant 0 : i32
730|     %c1_i32 = arith.constant 1 : i32
731|     %0 = "cnd"() : () -> i1
732|     %1, %acc_tok = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
733|     %init_tok = ttng.tmem_store %cst, %1[%acc_tok], %true : tensor<128x128xf32, #blocked1> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
734|     %last_tok = scf.for %arg3 = %c0_i32 to %arg2 step %c1_i32 iter_args(%tok = %init_tok) -> !ttg.async.token : i32 {
```
**EN:** This function-oriented block defines or enters `select_before_mma`. Within it, the test exercises constants, tt.func, tensor-memory allocation, ttng.tmem_store, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `select_before_mma` 为核心。测试在其中演示 常量、tt.func、张量内存分配、ttng.tmem_store、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 735-736
```mlir
735|       // CHECK: arith.xori {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
736|       %8 = arith.xori %0, %true : i1
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 737-742
```mlir
737|       // CHECK: ttng.tmem_store {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
738|       %store_tok = ttng.tmem_store %cst_0, %1[%tok], %8 : tensor<128x128xf32, #blocked1> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
739|       %4 = tt.load %arg0 {tt.latency = 2 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
740|       %5 = ttg.local_alloc %4 : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
741|       %6 = tt.load %arg1 {tt.latency = 2 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
742|       %7 = ttg.local_alloc %6 : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized loads, shared/local memory allocation, ttng.tmem_store. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的加载、共享/本地内存分配、ttng.tmem_store。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 743-752
```mlir
743|       // CHECK: ttng.tc_gen5_mma {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32, tt.self_latency = 1 : i32}
744|       %mma_tok = ttng.tc_gen5_mma %5, %7, %1[%store_tok], %true, %true {tt.latency = 1 : i32, tt.self_latency = 1 : i32} : !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
745|       scf.yield %mma_tok : !ttg.async.token
746|     } {tt.scheduled_max_stage = 3 : i32}
747|     %2, %res_tok = ttng.tmem_load %1[%last_tok] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
748|     %3 = arith.truncf %2 : tensor<128x128xf32, #blocked1> to tensor<128x128xf16, #blocked1>
749|     tt.return %3 : tensor<128x128xf16, #blocked1>
750|   }
751| }
752| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining Gen5 tensor-core MMA ops, loop/if yielded values, ttng.tmem_load, arith.truncf, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 Gen5 张量核 MMA 操作、循环/分支产出值、ttng.tmem_load、arith.truncf、tt.return。

### Lines 753-753
```mlir
753| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 754-758
```mlir
754| 
755| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
756| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
757| #smem = #ttg.shared_memory
758| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 759-759
```mlir
759| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 760-760
```mlir
760|   // CHECK-LABEL: @two_dots
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @two_dots anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @two_dots 这样的标签用于锚定匹配范围。

### Lines 761-767
```mlir
761|   tt.func public @two_dots(%arg0: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = 16 : i32, tt.divisibility = 16 : i32}, %arg1: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = 16 : i32, tt.divisibility = 16 : i32}, %arg2: tensor<128x128x!tt.ptr<f32>, #blocked> {tt.contiguity = 16 : i32, tt.divisibility = 16 : i32}, %arg3: tensor<128x128x!tt.ptr<f32>, #blocked> {tt.contiguity = 16 : i32, tt.divisibility = 16 : i32}, %arg4: i32) {
762|     %true = arith.constant true
763|     %c0_i32 = arith.constant 0 : i32
764|     %c1_i32 = arith.constant 1 : i32
765|     %0, %acc_tok0 = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
766|     %1, %acc_tok1 = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
767|     %last_tok:2 = scf.for %arg5 = %c0_i32 to %arg4 step %c1_i32 iter_args(%tok0 = %acc_tok0, %tok1 = %acc_tok1) -> (!ttg.async.token, !ttg.async.token) : i32 {
```
**EN:** This function-oriented block defines or enters `two_dots`. Within it, the test exercises constants, tt.func, tensor-memory allocation, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `two_dots` 为核心。测试在其中演示 常量、tt.func、张量内存分配、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 768-769
```mlir
768|       // CHECK: tt.load {{.*}} {loop.cluster = 4 : i32, loop.stage = 0 : i32}
769|       %2 = tt.load %arg0 {tt.latency = 2 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 770-771
```mlir
770|       // CHECK: ttg.local_alloc {{.*}} {loop.cluster = 2 : i32, loop.stage = 2 : i32}
771|       %3 = ttg.local_alloc %2 : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 772-773
```mlir
772|       // CHECK: tt.load {{.*}} {loop.cluster = 4 : i32, loop.stage = 0 : i32}
773|       %4 = tt.load %arg1 {tt.latency = 2 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 774-775
```mlir
774|       // CHECK: ttg.local_alloc {{.*}} {loop.cluster = 2 : i32, loop.stage = 2 : i32}
775|       %5 = ttg.local_alloc %4 : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 776-777
```mlir
776|       // CHECK: tt.load {{.*}} {loop.cluster = 2 : i32, loop.stage = 2 : i32}
777|       %6 = tt.load %arg2 : tensor<128x128x!tt.ptr<f32>, #blocked>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 778-779
```mlir
778|       // CHECK: ttng.tmem_store {{.*}} {loop.cluster = 2 : i32, loop.stage = 2 : i32}
779|       %store_tok0 = ttng.tmem_store %6, %0[%tok0], %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 780-781
```mlir
780|       // CHECK: ttng.tc_gen5_mma {{.*}} {loop.cluster = 2 : i32, loop.stage = 2 : i32, tt.self_latency = 1 : i32}
781|       %mma_tok0 = ttng.tc_gen5_mma %3, %5, %0[%store_tok0], %true, %true {tt.latency = 1 : i32, tt.self_latency = 1 : i32} : !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 782-783
```mlir
782|       // CHECK: ttng.tmem_load {{.*}} {loop.cluster = 1 : i32, loop.stage = 3 : i32}
783|       %7, %load_tok0 = ttng.tmem_load %0[%mma_tok0] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 784-785
```mlir
784|       // CHECK: ttng.tmem_store {{.*}} {loop.cluster = 1 : i32, loop.stage = 3 : i32}
785|       %store_tok1 = ttng.tmem_store %7, %1[%tok1], %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 786-787
```mlir
786|       // CHECK: ttng.tc_gen5_mma {{.*}} {loop.cluster = 1 : i32, loop.stage = 3 : i32, tt.self_latency = 1 : i32}
787|       %mma_tok1 = ttng.tc_gen5_mma %3, %5, %1[%store_tok1], %true, %true {tt.latency = 1 : i32, tt.self_latency = 1 : i32} : !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 788-789
```mlir
788|       // CHECK: ttng.tmem_load {{.*}} {loop.cluster = 0 : i32, loop.stage = 4 : i32}
789|       %8, %load_tok1 = ttng.tmem_load %1[%mma_tok1] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 790-797
```mlir
790|       // CHECK: tt.store {{.*}} {loop.cluster = 0 : i32, loop.stage = 4 : i32}
791|       tt.store %arg3, %8 : tensor<128x128x!tt.ptr<f32>, #blocked>
792|       scf.yield %load_tok0, %load_tok1 : !ttg.async.token, !ttg.async.token
793|     }
794|     tt.return
795|   }
796| }
797| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining masked or vectorized stores, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 带掩码或向量化的存储、循环/分支产出值、tt.return。

### Lines 798-798
```mlir
798| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 799-804
```mlir
799| 
800| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
801| #blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
802| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
803| #smem = #ttg.shared_memory
804| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 805-805
```mlir
805| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 806-806
```mlir
806|   // CHECK-LABEL: @changed_acc_before_mma
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @changed_acc_before_mma anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @changed_acc_before_mma 这样的标签用于锚定匹配范围。

### Lines 807-815
```mlir
807|   tt.func public @changed_acc_before_mma(%arg0: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = 16 : i32, tt.divisibility = 16 : i32}, %arg1: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = 16 : i32, tt.divisibility = 16 : i32}, %arg2: i32) -> tensor<128x128xf16, #blocked1> {
808|     %true = arith.constant true
809|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked1>
810|     %cst_0 = arith.constant dense<2.000000e+00> : tensor<128x128xf32, #blocked1>
811|     %c0_i32 = arith.constant 0 : i32
812|     %c1_i32 = arith.constant 1 : i32
813|     %0, %acc_tok = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
814|     %init_tok = ttng.tmem_store %cst, %0[%acc_tok], %true : tensor<128x128xf32, #blocked1> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
815|     %last_tok = scf.for %arg3 = %c0_i32 to %arg2 step %c1_i32 iter_args(%tok = %init_tok) -> !ttg.async.token : i32 {
```
**EN:** This function-oriented block defines or enters `changed_acc_before_mma`. Within it, the test exercises constants, tt.func, tensor-memory allocation, ttng.tmem_store, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `changed_acc_before_mma` 为核心。测试在其中演示 常量、tt.func、张量内存分配、ttng.tmem_store、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 816-817
```mlir
816|       // CHECK: tt.load {{.*}} {loop.cluster = 2 : i32, loop.stage = 0 : i32}
817|       %3 = tt.load %arg0 {tt.latency = 2 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 818-819
```mlir
818|       // CHECK: ttg.local_alloc {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
819|       %4 = ttg.local_alloc %3 : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 820-821
```mlir
820|       // CHECK: tt.load {{.*}} {loop.cluster = 2 : i32, loop.stage = 0 : i32}
821|       %5 = tt.load %arg1 {tt.latency = 2 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 822-823
```mlir
822|       // CHECK: ttg.local_alloc {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
823|       %6 = ttg.local_alloc %5 : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 824-825
```mlir
824|       // CHECK: ttng.tmem_load {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
825|       %7, %load_tok = ttng.tmem_load %0[%tok] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 826-827
```mlir
826|       // CHECK: arith.mulf {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
827|       %8 = arith.mulf %7, %cst_0 : tensor<128x128xf32, #blocked1>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 828-829
```mlir
828|       // CHECK: ttng.tmem_store {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
829|       %store_tok = ttng.tmem_store %8, %0[%load_tok], %true : tensor<128x128xf32, #blocked1> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 830-839
```mlir
830|       // CHECK: ttng.tc_gen5_mma {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
831|       %mma_tok = ttng.tc_gen5_mma %4, %6, %0[%store_tok], %true, %true : !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
832|       scf.yield %mma_tok : !ttg.async.token
833|     } {tt.scheduled_max_stage = 2 : i32}
834|     %1, %res_tok = ttng.tmem_load %0[%last_tok] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
835|     %2 = arith.truncf %1 : tensor<128x128xf32, #blocked1> to tensor<128x128xf16, #blocked1>
836|     tt.return %2 : tensor<128x128xf16, #blocked1>
837|   }
838| }
839| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining Gen5 tensor-core MMA ops, loop/if yielded values, ttng.tmem_load, arith.truncf, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 Gen5 张量核 MMA 操作、循环/分支产出值、ttng.tmem_load、arith.truncf、tt.return。

### Lines 840-840
```mlir
840| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 841-847
```mlir
841| 
842| #blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
843| #linear = #ttg.linear<{register = [[0, 1], [0, 2], [32, 0], [64, 0]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0]], warp = [[0, 0], [0, 0]], block = []}>
844| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 0, transposed = false, elementBitWidth = 8}>
845| #smem = #ttg.shared_memory
846| #tmem_scales = #ttng.tensor_memory_scales_encoding<>
847| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 848-849
```mlir
848| module attributes {"ttg.num-warps" = 4 : i32} {
849| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 850-850
```mlir
850| // CHECK-LABEL: @backwards_prop_existing
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @backwards_prop_existing anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @backwards_prop_existing 这样的标签用于锚定匹配范围。

### Lines 851-856
```mlir
851| tt.func public @backwards_prop_existing(%arg0: i32, %arg1: tensor<128x4x!tt.ptr<i8>, #blocked>) {
852|   %c0_i32 = arith.constant 0 : i32
853|   %c1_i32 = arith.constant 1 : i32
854|   scf.for %arg2 = %c0_i32 to %arg0 step %c1_i32  : i32 {
855|     %0 = tt.load %arg1 {loop.cluster = 2 : i32, loop.stage = 3 : i32} : tensor<128x4x!tt.ptr<i8>, #blocked>
856|     %1 = ttg.local_alloc %0 : (tensor<128x4xi8, #blocked>) -> !ttg.memdesc<128x4xi8, #shared, #smem>
```
**EN:** This function-oriented block defines or enters `backwards_prop_existing`. Within it, the test exercises tt.func, constants, structured loops, masked or vectorized loads, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `backwards_prop_existing` 为核心。测试在其中演示 tt.func、常量、结构化循环、带掩码或向量化的加载、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 857-865
```mlir
857|     // CHECK: ttg.local_load %{{.*}} {loop.cluster = 0 : i32, loop.stage = 0 : i32}
858|     %2 = ttg.local_load %1 : !ttg.memdesc<128x4xi8, #shared, #smem> -> tensor<128x4xi8, #linear>
859|     %result = ttng.tmem_alloc %2 {loop.cluster = 2 : i32, loop.stage = 3 : i32} : (tensor<128x4xi8, #linear>) -> !ttg.memdesc<128x4xi8, #tmem_scales, #ttng.tensor_memory>
860|     "use"(%result) {loop.cluster = 2 : i32, loop.stage = 3 : i32} : (!ttg.memdesc<128x4xi8, #tmem_scales, #ttng.tensor_memory>) -> ()
861|   } {tt.scheduled_max_stage = 3 : i32, tt.warp_specialize}
862|   tt.return
863| }
864| 
865| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on local/shared memory loads, tensor-memory allocation, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 本地/共享内存加载、张量内存分配、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-allow-unregistered-dialect`, `-split-input-file`, `-tritongpu-schedule-loops`, `-canonicalize`
- **CN:** 主要 pass 选项：`-allow-unregistered-dialect`，`-split-input-file`，`-tritongpu-schedule-loops`，`-canonicalize`
- **EN:** Dominant operations include `arith.constant`, `tt.func`, `tt.load`, `scf.for`, `tt.return`, `ttg.local_alloc`, `scf.yield`, `ttng.tmem_load`, `module`, `arith.addf`.
- **CN:** 主要操作包括 `arith.constant`、`tt.func`、`tt.load`、`scf.for`、`tt.return`、`ttg.local_alloc`、`scf.yield`、`ttng.tmem_load`、`module`、`arith.addf`。
- **EN:** The file contains 14 independently testable section(s). Check styles used: CHECK x154, CHECK-LABEL x29, CHECK-NOT x3. Important labels include @one_dep, @parallel_deps, @parallel_deps_uneven1, @parallel_deps_uneven2. It validates both the presence of the desired rewrite and the absence of unwanted residual operations.
- **CN:** 该文件包含 14 个可独立测试的分段。使用的检查类型：CHECK ×154，CHECK-LABEL ×29，CHECK-NOT ×3。 关键标签包括 @one_dep，@parallel_deps，@parallel_deps_uneven1，@parallel_deps_uneven2。 它同时验证目标重写已经出现，并且不再残留不需要的操作。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `tt`: Triton high-level tensor/pointer operations.
- **CN:** `tt`：Triton 高层张量/指针操作。
- **EN:** `ttg`: TritonGPU layout, memory, and GPU-specific operations.
- **CN:** `ttg`：TritonGPU 布局、内存和 GPU 专用操作。
- **EN:** `ttng`: Triton Nvidia GPU tensor-memory and MMA operations.
- **CN:** `ttng`：Triton Nvidia GPU 张量内存与 MMA 操作。
- **EN:** `arith`: MLIR arithmetic dialect for scalar/tensor math.
- **CN:** `arith`：用于标量/张量运算的 MLIR arithmetic 方言。
- **EN:** `scf`: Structured control flow for loops and conditionals.
- **CN:** `scf`：用于循环与条件分支的结构化控制流。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。