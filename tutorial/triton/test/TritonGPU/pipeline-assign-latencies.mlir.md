# pipeline-assign-latencies.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/pipeline-assign-latencies.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-tritongpu-assign-latencies=num-stages=3, -canonicalize` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-tritongpu-assign-latencies=num-stages=3, -canonicalize` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-assign-latencies=num-stages=3 -canonicalize | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-assign-latencies=num-stages=3 -canonicalize | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-assign-latencies=num-stages=3 -canonicalize | FileCheck %s
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-assign-latencies=num-stages=3 -canonicalize | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-assign-latencies=num-stages=3 -canonicalize | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-12
```mlir
 3| #AL = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
 4| #BL = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
 5| #C = #ttg.nvidia_mma<{versionMajor = 2, warpsPerCTA = [4, 1], instrShape = [16, 8]}>
 6| #A = #ttg.dot_op<{opIdx = 0, parent = #C, kWidth=2}>
 7| #B = #ttg.dot_op<{opIdx = 1, parent = #C, kWidth=2}>
 8| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 16}>
 9| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 16}>
10| #shared2 = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = true, elementBitWidth = 16}>
11| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 128, 32]}>
12| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 13-13
```mlir
13| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 14-14
```mlir
14| // CHECK-LABEL: @default_stages
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @default_stages anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @default_stages 这样的标签用于锚定匹配范围。

### Lines 15-22
```mlir
15| tt.func @default_stages(%lb : index, %ub : index, %step : index,
16|                   %a_ptr_init : tensor<128x32x!tt.ptr<f16>, #AL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 32]> : tensor<2xi32>},
17|                   %b_ptr_init : tensor<32x128x!tt.ptr<f16>, #BL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 32]> : tensor<2xi32>}) -> tensor<128x128xf32, #C> {
18|   %c_init = arith.constant dense<0.00e+00> : tensor<128x128xf32, #C>
19|   %a_off = arith.constant dense<4> : tensor<128x32xi32, #AL>
20|   %b_off = arith.constant dense<4> : tensor<32x128xi32, #BL>
21| 
22|   %loop:3 = scf.for %iv = %lb to %ub step %step iter_args(%a_ptr = %a_ptr_init, %b_ptr = %b_ptr_init, %prev_c = %c_init) -> (tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>) {
```
**EN:** This function-oriented block defines or enters `default_stages`. Within it, the test exercises constants, tt.func, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `default_stages` 为核心。测试在其中演示 常量、tt.func、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 23-25
```mlir
23|     // CHECK: tt.load {{.*}} {tt.latency = 2 : i32}
24|     %a_ = tt.load %a_ptr : tensor<128x32x!tt.ptr<f16>, #AL>
25|     %a = ttg.convert_layout %a_ : tensor<128x32xf16, #AL> -> tensor<128x32xf16, #A>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 26-38
```mlir
26|     // CHECK: tt.load {{.*}} {tt.latency = 2 : i32}
27|     %b_ = tt.load %b_ptr : tensor<32x128x!tt.ptr<f16>, #BL>
28|     %b = ttg.convert_layout %b_ : tensor<32x128xf16, #BL> -> tensor<32x128xf16, #B>
29| 
30|     %c = tt.dot %a, %b, %prev_c : tensor<128x32xf16, #A> * tensor<32x128xf16, #B> -> tensor<128x128xf32, #C>
31| 
32|     %next_a_ptr = tt.addptr %a_ptr, %a_off : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xi32, #AL>
33|     %next_b_ptr = tt.addptr %b_ptr, %b_off : tensor<32x128x!tt.ptr<f16>, #BL>, tensor<32x128xi32, #BL>
34|     scf.yield %next_a_ptr, %next_b_ptr, %c : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>
35|   }
36|   tt.return %loop#2: tensor<128x128xf32, #C>
37| }
38| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining pointer arithmetic, masked or vectorized loads, layout conversions, dot-product or MMA-style math, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 指针算术、带掩码或向量化的加载、布局转换、点积或 MMA 风格计算、循环/分支产出值。

### Lines 39-40
```mlir
39| // CHECK-LABEL: @small_load
40| // We should *not* assign latency to the load of b_ptr.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 41-48
```mlir
41| tt.func @small_load(%lb : index, %ub : index, %step : index,
42|                   %a_ptr_init : tensor<128x32x!tt.ptr<f16>, #AL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 32]> : tensor<2xi32>},
43|                   %b_ptr_init : tensor<32x128x!tt.ptr<f16>, #BL>) -> tensor<128x128xf32, #C> {
44|   %c_init = arith.constant dense<0.00e+00> : tensor<128x128xf32, #C>
45|   %a_off = arith.constant dense<4> : tensor<128x32xi32, #AL>
46|   %b_off = arith.constant dense<4> : tensor<32x128xi32, #BL>
47| 
48|   %loop:3 = scf.for %iv = %lb to %ub step %step iter_args(%a_ptr = %a_ptr_init, %b_ptr = %b_ptr_init, %prev_c = %c_init) -> (tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>) {
```
**EN:** This function-oriented block defines or enters `small_load`. Within it, the test exercises constants, tt.func, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `small_load` 为核心。测试在其中演示 常量、tt.func、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 49-51
```mlir
49|     // CHECK: tt.load {{.*}} {tt.latency = 2 : i32}
50|     %a_ = tt.load %a_ptr : tensor<128x32x!tt.ptr<f16>, #AL>
51|     %a = ttg.convert_layout %a_ : tensor<128x32xf16, #AL> -> tensor<128x32xf16, #A>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 52-65
```mlir
52|     // CHECK: tt.load {{.*}}
53|     // CHECK-NOT: tt.latency
54|     %b_ = tt.load %b_ptr : tensor<32x128x!tt.ptr<f16>, #BL>
55|     %b = ttg.convert_layout %b_ : tensor<32x128xf16, #BL> -> tensor<32x128xf16, #B>
56| 
57|     %c = tt.dot %a, %b, %prev_c : tensor<128x32xf16, #A> * tensor<32x128xf16, #B> -> tensor<128x128xf32, #C>
58| 
59|     %next_a_ptr = tt.addptr %a_ptr, %a_off : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xi32, #AL>
60|     %next_b_ptr = tt.addptr %b_ptr, %b_off : tensor<32x128x!tt.ptr<f16>, #BL>, tensor<32x128xi32, #BL>
61|     scf.yield %next_a_ptr, %next_b_ptr, %c : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>
62|   }
63|   tt.return %loop#2: tensor<128x128xf32, #C>
64| }
65| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining pointer arithmetic, masked or vectorized loads, layout conversions, dot-product or MMA-style math, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 指针算术、带掩码或向量化的加载、布局转换、点积或 MMA 风格计算、循环/分支产出值。

### Lines 66-66
```mlir
66| // CHECK-LABEL: @load_into_shared
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @load_into_shared anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @load_into_shared 这样的标签用于锚定匹配范围。

### Lines 67-74
```mlir
67| tt.func @load_into_shared(%lb : index, %ub : index, %step : index,
68|                   %a_ptr_init : tensor<128x32x!tt.ptr<f16>, #AL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 32]> : tensor<2xi32>},
69|                   %b_ptr_init : tensor<32x128x!tt.ptr<f16>, #BL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 32]> : tensor<2xi32>}) -> tensor<128x128xf32, #mma> {
70|   %c_init = arith.constant dense<0.00e+00> : tensor<128x128xf32, #mma>
71|   %a_off = arith.constant dense<4> : tensor<128x32xi32, #AL>
72|   %b_off = arith.constant dense<4> : tensor<32x128xi32, #BL>
73| 
74|   %loop:3 = scf.for %iv = %lb to %ub step %step iter_args(%a_ptr = %a_ptr_init, %b_ptr = %b_ptr_init, %prev_c = %c_init) -> (tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #mma>) {
```
**EN:** This function-oriented block defines or enters `load_into_shared`. Within it, the test exercises constants, tt.func, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `load_into_shared` 为核心。测试在其中演示 常量、tt.func、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 75-77
```mlir
75|     // CHECK: tt.load {{.*}} {tt.latency = 2 : i32}
76|     %a_ = tt.load %a_ptr : tensor<128x32x!tt.ptr<f16>, #AL>
77|     %a = ttg.local_alloc %a_ : (tensor<128x32xf16, #AL>) -> !ttg.memdesc<128x32xf16, #shared, #ttg.shared_memory>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 78-90
```mlir
78|     // CHECK: tt.load {{.*}} {tt.latency = 2 : i32}
79|     %b_ = tt.load %b_ptr : tensor<32x128x!tt.ptr<f16>, #BL>
80|     %b = ttg.local_alloc %b_ : (tensor<32x128xf16, #BL>) -> !ttg.memdesc<32x128xf16, #shared1, #ttg.shared_memory>
81| 
82|     %c = ttng.warp_group_dot %a, %b, %prev_c {maxNumImpreciseAcc = 1073741824 : i32} : !ttg.memdesc<128x32xf16, #shared, #ttg.shared_memory> * !ttg.memdesc<32x128xf16, #shared1, #ttg.shared_memory> -> tensor<128x128xf32, #mma>
83| 
84|     %next_a_ptr = tt.addptr %a_ptr, %a_off : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xi32, #AL>
85|     %next_b_ptr = tt.addptr %b_ptr, %b_off : tensor<32x128x!tt.ptr<f16>, #BL>, tensor<32x128xi32, #BL>
86|     scf.yield %next_a_ptr, %next_b_ptr, %c : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #mma>
87|   }
88|   tt.return %loop#2: tensor<128x128xf32, #mma>
89| }
90| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining pointer arithmetic, masked or vectorized loads, shared/local memory allocation, ttng.warp_group_dot, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 指针算术、带掩码或向量化的加载、共享/本地内存分配、ttng.warp_group_dot、循环/分支产出值。

### Lines 91-91
```mlir
91| // CHECK-LABEL: @load_into_lt_4b
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @load_into_lt_4b anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @load_into_lt_4b 这样的标签用于锚定匹配范围。

### Lines 92-99
```mlir
92| tt.func @load_into_lt_4b(%lb : index, %ub : index, %step : index,
93|                   %a_ptr_init : tensor<128x32x!tt.ptr<f16>, #AL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 32]> : tensor<2xi32>},
94|                   %b_ptr_init : tensor<32x128x!tt.ptr<f16>, #BL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 32]> : tensor<2xi32>}) -> tensor<128x128xf32, #mma> {
95|   %c_init = arith.constant dense<0.00e+00> : tensor<128x128xf32, #mma>
96|   %a_off = arith.constant dense<4> : tensor<128x32xi32, #AL>
97|   %b_off = arith.constant dense<4> : tensor<32x128xi32, #BL>
98| 
99|   %loop:3 = scf.for %iv = %lb to %ub step %step iter_args(%a_ptr = %a_ptr_init, %b_ptr = %b_ptr_init, %prev_c = %c_init) -> (tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #mma>) {
```
**EN:** This function-oriented block defines or enters `load_into_lt_4b`. Within it, the test exercises constants, tt.func, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `load_into_lt_4b` 为核心。测试在其中演示 常量、tt.func、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 100-103
```mlir
100|     // CHECK: tt.load {{.*}} {tt.latency = 2 : i32}
101|     %a_ = tt.load %a_ptr : tensor<128x32x!tt.ptr<f16>, #AL>
102|     %a = ttg.local_alloc %a_ : (tensor<128x32xf16, #AL>) -> !ttg.memdesc<128x32xf16, #shared, #ttg.shared_memory>
103|     // Do not pipeline if cp.async would read less than 4 consecutive bytes
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized loads, shared/local memory allocation. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的加载、共享/本地内存分配。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 104-117
```mlir
104|     // CHECK: tt.load
105|     // CHECK-NOT: {tt.latency = 2 : i32}
106|     %b_ = tt.load %b_ptr : tensor<32x128x!tt.ptr<f16>, #BL>
107|     %b = ttg.local_alloc %b_ : (tensor<32x128xf16, #BL>) -> !ttg.memdesc<32x128xf16, #shared2, #ttg.shared_memory>
108| 
109|     %c = ttng.warp_group_dot %a, %b, %prev_c {maxNumImpreciseAcc = 1073741824 : i32} : !ttg.memdesc<128x32xf16, #shared, #ttg.shared_memory> * !ttg.memdesc<32x128xf16, #shared2, #ttg.shared_memory> -> tensor<128x128xf32, #mma>
110| 
111|     %next_a_ptr = tt.addptr %a_ptr, %a_off : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xi32, #AL>
112|     %next_b_ptr = tt.addptr %b_ptr, %b_off : tensor<32x128x!tt.ptr<f16>, #BL>, tensor<32x128xi32, #BL>
113|     scf.yield %next_a_ptr, %next_b_ptr, %c : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #mma>
114|   }
115|   tt.return %loop#2: tensor<128x128xf32, #mma>
116| }
117| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining pointer arithmetic, masked or vectorized loads, shared/local memory allocation, ttng.warp_group_dot, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 指针算术、带掩码或向量化的加载、共享/本地内存分配、ttng.warp_group_dot、循环/分支产出值。

### Lines 118-118
```mlir
118| // CHECK-LABEL: @intermediate_use
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @intermediate_use anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @intermediate_use 这样的标签用于锚定匹配范围。

### Lines 119-127
```mlir
119| tt.func @intermediate_use(%lb : index, %ub : index, %step : index,
120|                   %a_ptr_init : tensor<128x32x!tt.ptr<f16>, #AL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 32]> : tensor<2xi32>},
121|                   %b_ptr_init : tensor<32x128x!tt.ptr<f16>, #BL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 32]> : tensor<2xi32>}) -> tensor<128x128xf32, #C> {
122|   %c_init = arith.constant dense<0.00e+00> : tensor<128x128xf32, #C>
123|   %a_off = arith.constant dense<4> : tensor<128x32xi32, #AL>
124|   %b_off = arith.constant dense<4> : tensor<32x128xi32, #BL>
125|   %c2 = arith.constant dense<2.00> : tensor<32x128xf16, #BL>
126| 
127|   %loop:3 = scf.for %iv = %lb to %ub step %step iter_args(%a_ptr = %a_ptr_init, %b_ptr = %b_ptr_init, %prev_c = %c_init) -> (tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>) {
```
**EN:** This function-oriented block defines or enters `intermediate_use`. Within it, the test exercises constants, tt.func, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `intermediate_use` 为核心。测试在其中演示 常量、tt.func、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 128-130
```mlir
128|     // CHECK: tt.load {{.*}} {tt.latency = 2 : i32}
129|     %a_ = tt.load %a_ptr : tensor<128x32x!tt.ptr<f16>, #AL>
130|     %a = ttg.convert_layout %a_ : tensor<128x32xf16, #AL> -> tensor<128x32xf16, #A>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 131-144
```mlir
131|     // CHECK: tt.load {{.*}} {tt.latency = 2 : i32}
132|     %b_ = tt.load %b_ptr : tensor<32x128x!tt.ptr<f16>, #BL>
133|     %b_2 = arith.mulf %b_ , %c2 : tensor<32x128xf16, #BL>
134|     %b = ttg.convert_layout %b_2 : tensor<32x128xf16, #BL> -> tensor<32x128xf16, #B>
135| 
136|     %c = tt.dot %a, %b, %prev_c : tensor<128x32xf16, #A> * tensor<32x128xf16, #B> -> tensor<128x128xf32, #C>
137| 
138|     %next_a_ptr = tt.addptr %a_ptr, %a_off : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xi32, #AL>
139|     %next_b_ptr = tt.addptr %b_ptr, %b_off : tensor<32x128x!tt.ptr<f16>, #BL>, tensor<32x128xi32, #BL>
140|     scf.yield %next_a_ptr, %next_b_ptr, %c : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>
141|   }
142|   tt.return %loop#2: tensor<128x128xf32, #C>
143| }
144| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining pointer arithmetic, masked or vectorized loads, arith.mulf, layout conversions, dot-product or MMA-style math.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 指针算术、带掩码或向量化的加载、arith.mulf、布局转换、点积或 MMA 风格计算。

### Lines 145-145
```mlir
145| // CHECK-LABEL: @indirect_load
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @indirect_load anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @indirect_load 这样的标签用于锚定匹配范围。

### Lines 146-155
```mlir
146| tt.func @indirect_load(%lb : index, %ub : index, %step : index,
147|                   %a_ind_ptr_init : tensor<128x32x!tt.ptr<i32>, #AL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 32]> : tensor<2xi32>},
148|                   %b_ind_ptr_init : tensor<32x128x!tt.ptr<i32>, #BL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 32]> : tensor<2xi32>},
149|                   %a_ptr_init : tensor<128x32x!tt.ptr<f16>, #AL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 32]> : tensor<2xi32>},
150|                   %b_ptr_init : tensor<32x128x!tt.ptr<f16>, #BL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 32]> : tensor<2xi32>}) -> tensor<128x128xf32, #C> {
151|   %c_init = arith.constant dense<0.00e+00> : tensor<128x128xf32, #C>
152|   %a_ind_off = arith.constant dense<4> : tensor<128x32xi32, #AL>
153|   %b_ind_off = arith.constant dense<4> : tensor<32x128xi32, #BL>
154| 
155|   %loop:5 = scf.for %iv = %lb to %ub step %step iter_args(%a_ind_ptr = %a_ind_ptr_init, %b_ind_ptr = %b_ind_ptr_init, %a_ptr = %a_ptr_init, %b_ptr = %b_ptr_init, %prev_c = %c_init) -> (tensor<128x32x!tt.ptr<i32>, #AL>, tensor<32x128x!tt.ptr<i32>, #BL>, tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>) {
```
**EN:** This function-oriented block defines or enters `indirect_load`. Within it, the test exercises constants, tt.func, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `indirect_load` 为核心。测试在其中演示 常量、tt.func、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 156-157
```mlir
156|     // CHECK: tt.load {{.*}} {tt.latency = 1 : i32}
157|     %a_off = tt.load %a_ind_ptr : tensor<128x32x!tt.ptr<i32>, #AL>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 158-163
```mlir
158|     // CHECK: tt.load {{.*}} {tt.latency = 1 : i32}
159|     %b_off = tt.load %b_ind_ptr : tensor<32x128x!tt.ptr<i32>, #BL>
160|     %next_a_ind_ptr = tt.addptr %a_ind_ptr, %a_ind_off : tensor<128x32x!tt.ptr<i32>, #AL>, tensor<128x32xi32, #AL>
161|     %next_b_ind_ptr = tt.addptr %b_ind_ptr, %b_ind_off : tensor<32x128x!tt.ptr<i32>, #BL>, tensor<32x128xi32, #BL>
162|     %next_a_ptr = tt.addptr %a_ptr, %a_off {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 32]> : tensor<2xi32>, tt.constancy = dense<[1, 1]> : tensor<2xi32>} : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xi32, #AL>
163|     %next_b_ptr = tt.addptr %b_ptr, %b_off {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 32]> : tensor<2xi32>, tt.constancy = dense<[1, 1]> : tensor<2xi32>} : tensor<32x128x!tt.ptr<f16>, #BL>, tensor<32x128xi32, #BL>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on pointer arithmetic, masked or vectorized loads. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 指针算术、带掩码或向量化的加载。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 164-166
```mlir
164|     // CHECK: tt.load {{.*}} {tt.latency = 1 : i32}
165|     %a_ = tt.load %next_a_ptr : tensor<128x32x!tt.ptr<f16>, #AL>
166|     %a = ttg.convert_layout %a_ : tensor<128x32xf16, #AL> -> tensor<128x32xf16, #A>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 167-176
```mlir
167|     // CHECK: tt.load {{.*}} {tt.latency = 1 : i32}
168|     %b_ = tt.load %next_b_ptr : tensor<32x128x!tt.ptr<f16>, #BL>
169|     %b = ttg.convert_layout %b_ : tensor<32x128xf16, #BL> -> tensor<32x128xf16, #B>
170| 
171|     %c = tt.dot %a, %b, %prev_c : tensor<128x32xf16, #A> * tensor<32x128xf16, #B> -> tensor<128x128xf32, #C>
172|     scf.yield %next_a_ind_ptr, %next_b_ind_ptr, %next_a_ptr, %next_b_ptr, %c : tensor<128x32x!tt.ptr<i32>, #AL>, tensor<32x128x!tt.ptr<i32>, #BL>, tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>
173|   }
174|   tt.return %loop#4: tensor<128x128xf32, #C>
175| }
176| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining masked or vectorized loads, layout conversions, dot-product or MMA-style math, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 带掩码或向量化的加载、布局转换、点积或 MMA 风格计算、循环/分支产出值、tt.return。

### Lines 177-177
```mlir
177| // CHECK-LABEL: @mixed_loads
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @mixed_loads anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @mixed_loads 这样的标签用于锚定匹配范围。

### Lines 178-186
```mlir
178| tt.func @mixed_loads(%lb : index, %ub : index, %step : index,
179|                   %a_ind_ptr_init : tensor<128x32x!tt.ptr<i32>, #AL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 32]> : tensor<2xi32>},
180|                   %a_ptr_init : tensor<128x32x!tt.ptr<f16>, #AL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 32]> : tensor<2xi32>},
181|                   %b_ptr_init : tensor<32x128x!tt.ptr<f16>, #BL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 32]> : tensor<2xi32>}) -> tensor<128x128xf32, #C> {
182|   %c_init = arith.constant dense<0.00e+00> : tensor<128x128xf32, #C>
183|   %a_ind_off = arith.constant dense<4> : tensor<128x32xi32, #AL>
184|   %b_off = arith.constant dense<4> : tensor<32x128xi32, #BL>
185| 
186|   %loop:4 = scf.for %iv = %lb to %ub step %step iter_args(%a_ind_ptr = %a_ind_ptr_init, %a_ptr = %a_ptr_init, %b_ptr = %b_ptr_init, %prev_c = %c_init) -> (tensor<128x32x!tt.ptr<i32>, #AL>, tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>) {
```
**EN:** This function-oriented block defines or enters `mixed_loads`. Within it, the test exercises constants, tt.func, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mixed_loads` 为核心。测试在其中演示 常量、tt.func、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 187-191
```mlir
187|     // CHECK: tt.load {{.*}} {tt.latency = 1 : i32}
188|     %a_off = tt.load %a_ind_ptr : tensor<128x32x!tt.ptr<i32>, #AL>
189|     %next_a_ind_ptr = tt.addptr %a_ind_ptr, %a_ind_off : tensor<128x32x!tt.ptr<i32>, #AL>, tensor<128x32xi32, #AL>
190|     %next_a_ptr = tt.addptr %a_ptr, %a_off {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 32]> : tensor<2xi32>, tt.constancy = dense<[1, 1]> : tensor<2xi32>} : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xi32, #AL>
191|     %next_b_ptr = tt.addptr %b_ptr, %b_off : tensor<32x128x!tt.ptr<f16>, #BL>, tensor<32x128xi32, #BL>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on pointer arithmetic, masked or vectorized loads. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 指针算术、带掩码或向量化的加载。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 192-194
```mlir
192|     // CHECK: tt.load {{.*}} {tt.latency = 1 : i32}
193|     %a_ = tt.load %next_a_ptr : tensor<128x32x!tt.ptr<f16>, #AL>
194|     %a = ttg.convert_layout %a_ : tensor<128x32xf16, #AL> -> tensor<128x32xf16, #A>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 195-204
```mlir
195|     // CHECK: tt.load {{.*}} {tt.latency = 1 : i32}
196|     %b_ = tt.load %next_b_ptr : tensor<32x128x!tt.ptr<f16>, #BL>
197|     %b = ttg.convert_layout %b_ : tensor<32x128xf16, #BL> -> tensor<32x128xf16, #B>
198| 
199|     %c = tt.dot %a, %b, %prev_c : tensor<128x32xf16, #A> * tensor<32x128xf16, #B> -> tensor<128x128xf32, #C>
200|     scf.yield %next_a_ind_ptr, %next_a_ptr, %next_b_ptr, %c : tensor<128x32x!tt.ptr<i32>, #AL>, tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>
201|   }
202|   tt.return %loop#3: tensor<128x128xf32, #C>
203| }
204| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining masked or vectorized loads, layout conversions, dot-product or MMA-style math, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 带掩码或向量化的加载、布局转换、点积或 MMA 风格计算、循环/分支产出值、tt.return。

### Lines 205-205
```mlir
205| // CHECK-LABEL: @per_loop_stages
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @per_loop_stages anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @per_loop_stages 这样的标签用于锚定匹配范围。

### Lines 206-213
```mlir
206| tt.func @per_loop_stages(%lb : index, %ub : index, %step : index,
207|                   %a_ptr_init : tensor<128x32x!tt.ptr<f16>, #AL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 32]> : tensor<2xi32>},
208|                   %b_ptr_init : tensor<32x128x!tt.ptr<f16>, #BL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 32]> : tensor<2xi32>}) -> (tensor<128x128xf32, #C>, tensor<128x128xf32, #C>) {
209|   %c_init = arith.constant dense<0.00e+00> : tensor<128x128xf32, #C>
210|   %a_off = arith.constant dense<4> : tensor<128x32xi32, #AL>
211|   %b_off = arith.constant dense<4> : tensor<32x128xi32, #BL>
212| 
213|   %loop_cust_stages:4 = scf.for %iv = %lb to %ub step %step iter_args(%a_ptr = %a_ptr_init, %b_ptr = %b_ptr_init, %prev_c = %c_init, %l_ptr = %a_ptr_init) -> (tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>, tensor<128x32x!tt.ptr<f16>, #AL>) {
```
**EN:** This function-oriented block defines or enters `per_loop_stages`. Within it, the test exercises constants, tt.func, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `per_loop_stages` 为核心。测试在其中演示 常量、tt.func、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 214-216
```mlir
214|     // CHECK: tt.load {{.*}} {tt.latency = 3 : i32}
215|     %a_ = tt.load %a_ptr : tensor<128x32x!tt.ptr<f16>, #AL>
216|     %a = ttg.convert_layout %a_ : tensor<128x32xf16, #AL> -> tensor<128x32xf16, #A>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 217-222
```mlir
217|     // CHECK: tt.load {{.*}} {tt.latency = 3 : i32}
218|     %b_ = tt.load %b_ptr : tensor<32x128x!tt.ptr<f16>, #BL>
219|     %b = ttg.convert_layout %b_ : tensor<32x128xf16, #BL> -> tensor<32x128xf16, #B>
220| 
221|     %c = tt.dot %a, %b, %prev_c : tensor<128x32xf16, #A> * tensor<32x128xf16, #B> -> tensor<128x128xf32, #C>
222| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized loads, layout conversions, dot-product or MMA-style math. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的加载、布局转换、点积或 MMA 风格计算。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 223-232
```mlir
223|     // CHECK: tt.load {{.*}} {tt.latency = 3 : i32}
224|     %l = tt.load %l_ptr : tensor<128x32x!tt.ptr<f16>, #AL>
225|     "use"(%l) : (tensor<128x32xf16, #AL>) -> ()
226|     %next_a_ptr = tt.addptr %a_ptr, %a_off : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xi32, #AL>
227|     %next_b_ptr = tt.addptr %b_ptr, %b_off : tensor<32x128x!tt.ptr<f16>, #BL>, tensor<32x128xi32, #BL>
228|     %next_l_ptr = tt.addptr %l_ptr, %a_off : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xi32, #AL>
229|     scf.yield %next_a_ptr, %next_b_ptr, %c, %next_l_ptr : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>, tensor<128x32x!tt.ptr<f16>, #AL>
230|   } {tt.num_stages = 4 : i32}
231| 
232|   %loop:3 = scf.for %iv = %lb to %ub step %step iter_args(%a_ptr = %a_ptr_init, %b_ptr = %b_ptr_init, %prev_c = %c_init) -> (tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>) {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining pointer arithmetic, masked or vectorized loads, loop/if yielded values, structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 指针算术、带掩码或向量化的加载、循环/分支产出值、结构化循环。

### Lines 233-235
```mlir
233|     // CHECK: tt.load {{.*}} {tt.latency = 2 : i32}
234|     %a_ = tt.load %a_ptr : tensor<128x32x!tt.ptr<f16>, #AL>
235|     %a = ttg.convert_layout %a_ : tensor<128x32xf16, #AL> -> tensor<128x32xf16, #A>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 236-248
```mlir
236|     // CHECK: tt.load {{.*}} {tt.latency = 2 : i32}
237|     %b_ = tt.load %b_ptr : tensor<32x128x!tt.ptr<f16>, #BL>
238|     %b = ttg.convert_layout %b_ : tensor<32x128xf16, #BL> -> tensor<32x128xf16, #B>
239| 
240|     %c = tt.dot %a, %b, %prev_c : tensor<128x32xf16, #A> * tensor<32x128xf16, #B> -> tensor<128x128xf32, #C>
241| 
242|     %next_a_ptr = tt.addptr %a_ptr, %a_off : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xi32, #AL>
243|     %next_b_ptr = tt.addptr %b_ptr, %b_off : tensor<32x128x!tt.ptr<f16>, #BL>, tensor<32x128xi32, #BL>
244|     scf.yield %next_a_ptr, %next_b_ptr, %c : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>
245|   }
246|   tt.return %loop_cust_stages#2, %loop#2: tensor<128x128xf32, #C>, tensor<128x128xf32, #C>
247| }
248| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining pointer arithmetic, masked or vectorized loads, layout conversions, dot-product or MMA-style math, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 指针算术、带掩码或向量化的加载、布局转换、点积或 MMA 风格计算、循环/分支产出值。

### Lines 249-249
```mlir
249| // CHECK-LABEL: @indirect_load_cust_stages
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @indirect_load_cust_stages anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @indirect_load_cust_stages 这样的标签用于锚定匹配范围。

### Lines 250-259
```mlir
250| tt.func @indirect_load_cust_stages(%lb : index, %ub : index, %step : index,
251|                   %a_ind_ptr_init : tensor<128x32x!tt.ptr<i32>, #AL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 32]> : tensor<2xi32>},
252|                   %b_ind_ptr_init : tensor<32x128x!tt.ptr<i32>, #BL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 32]> : tensor<2xi32>},
253|                   %a_ptr_init : tensor<128x32x!tt.ptr<f16>, #AL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 32]> : tensor<2xi32>},
254|                   %b_ptr_init : tensor<32x128x!tt.ptr<f16>, #BL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 32]> : tensor<2xi32>}) -> tensor<128x128xf32, #C> {
255|   %c_init = arith.constant dense<0.00e+00> : tensor<128x128xf32, #C>
256|   %a_ind_off = arith.constant dense<4> : tensor<128x32xi32, #AL>
257|   %b_ind_off = arith.constant dense<4> : tensor<32x128xi32, #BL>
258| 
259|   %loop:5 = scf.for %iv = %lb to %ub step %step iter_args(%a_ind_ptr = %a_ind_ptr_init, %b_ind_ptr = %b_ind_ptr_init, %a_ptr = %a_ptr_init, %b_ptr = %b_ptr_init, %prev_c = %c_init) -> (tensor<128x32x!tt.ptr<i32>, #AL>, tensor<32x128x!tt.ptr<i32>, #BL>, tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>) {
```
**EN:** This function-oriented block defines or enters `indirect_load_cust_stages`. Within it, the test exercises constants, tt.func, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `indirect_load_cust_stages` 为核心。测试在其中演示 常量、tt.func、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 260-261
```mlir
260|     // CHECK: tt.load {{.*}} {tt.latency = 2 : i32}
261|     %a_off = tt.load %a_ind_ptr : tensor<128x32x!tt.ptr<i32>, #AL>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 262-267
```mlir
262|     // CHECK: tt.load {{.*}} {tt.latency = 2 : i32}
263|     %b_off = tt.load %b_ind_ptr : tensor<32x128x!tt.ptr<i32>, #BL>
264|     %next_a_ind_ptr = tt.addptr %a_ind_ptr, %a_ind_off : tensor<128x32x!tt.ptr<i32>, #AL>, tensor<128x32xi32, #AL>
265|     %next_b_ind_ptr = tt.addptr %b_ind_ptr, %b_ind_off : tensor<32x128x!tt.ptr<i32>, #BL>, tensor<32x128xi32, #BL>
266|     %next_a_ptr = tt.addptr %a_ptr, %a_off {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 32]> : tensor<2xi32>, tt.constancy = dense<[1, 1]> : tensor<2xi32>} : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xi32, #AL>
267|     %next_b_ptr = tt.addptr %b_ptr, %b_off {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 32]> : tensor<2xi32>, tt.constancy = dense<[1, 1]> : tensor<2xi32>} : tensor<32x128x!tt.ptr<f16>, #BL>, tensor<32x128xi32, #BL>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on pointer arithmetic, masked or vectorized loads. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 指针算术、带掩码或向量化的加载。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 268-270
```mlir
268|     // CHECK: tt.load {{.*}} {tt.latency = 2 : i32}
269|     %a_ = tt.load %next_a_ptr : tensor<128x32x!tt.ptr<f16>, #AL>
270|     %a = ttg.convert_layout %a_ : tensor<128x32xf16, #AL> -> tensor<128x32xf16, #A>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 271-280
```mlir
271|     // CHECK: tt.load {{.*}} {tt.latency = 2 : i32}
272|     %b_ = tt.load %next_b_ptr : tensor<32x128x!tt.ptr<f16>, #BL>
273|     %b = ttg.convert_layout %b_ : tensor<32x128xf16, #BL> -> tensor<32x128xf16, #B>
274| 
275|     %c = tt.dot %a, %b, %prev_c : tensor<128x32xf16, #A> * tensor<32x128xf16, #B> -> tensor<128x128xf32, #C>
276|     scf.yield %next_a_ind_ptr, %next_b_ind_ptr, %next_a_ptr, %next_b_ptr, %c : tensor<128x32x!tt.ptr<i32>, #AL>, tensor<32x128x!tt.ptr<i32>, #BL>, tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>
277|   } {tt.num_stages = 5 : i32}
278|   tt.return %loop#4: tensor<128x128xf32, #C>
279| }
280| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining masked or vectorized loads, layout conversions, dot-product or MMA-style math, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 带掩码或向量化的加载、布局转换、点积或 MMA 风格计算、循环/分支产出值、tt.return。

### Lines 281-281
```mlir
281| // CHECK-LABEL: @indirect_load_few_stages
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @indirect_load_few_stages anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @indirect_load_few_stages 这样的标签用于锚定匹配范围。

### Lines 282-291
```mlir
282| tt.func @indirect_load_few_stages(%lb : index, %ub : index, %step : index,
283|                   %a_ind_ptr_init : tensor<128x32x!tt.ptr<i32>, #AL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 32]> : tensor<2xi32>},
284|                   %b_ind_ptr_init : tensor<32x128x!tt.ptr<i32>, #BL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 32]> : tensor<2xi32>},
285|                   %a_ptr_init : tensor<128x32x!tt.ptr<f16>, #AL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 32]> : tensor<2xi32>},
286|                   %b_ptr_init : tensor<32x128x!tt.ptr<f16>, #BL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 32]> : tensor<2xi32>}) -> tensor<128x128xf32, #C> {
287|   %c_init = arith.constant dense<0.00e+00> : tensor<128x128xf32, #C>
288|   %a_ind_off = arith.constant dense<4> : tensor<128x32xi32, #AL>
289|   %b_ind_off = arith.constant dense<4> : tensor<32x128xi32, #BL>
290| 
291|   %loop:5 = scf.for %iv = %lb to %ub step %step iter_args(%a_ind_ptr = %a_ind_ptr_init, %b_ind_ptr = %b_ind_ptr_init, %a_ptr = %a_ptr_init, %b_ptr = %b_ptr_init, %prev_c = %c_init) -> (tensor<128x32x!tt.ptr<i32>, #AL>, tensor<32x128x!tt.ptr<i32>, #BL>, tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>) {
```
**EN:** This function-oriented block defines or enters `indirect_load_few_stages`. Within it, the test exercises constants, tt.func, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `indirect_load_few_stages` 为核心。测试在其中演示 常量、tt.func、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 292-294
```mlir
292|     // CHECK: tt.load
293|     // CHECK-NOT: tt.latency
294|     %a_off = tt.load %a_ind_ptr : tensor<128x32x!tt.ptr<i32>, #AL>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized loads. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的加载。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 295-301
```mlir
295|     // CHECK: tt.load
296|     // CHECK-NOT: tt.latency
297|     %b_off = tt.load %b_ind_ptr : tensor<32x128x!tt.ptr<i32>, #BL>
298|     %next_a_ind_ptr = tt.addptr %a_ind_ptr, %a_ind_off : tensor<128x32x!tt.ptr<i32>, #AL>, tensor<128x32xi32, #AL>
299|     %next_b_ind_ptr = tt.addptr %b_ind_ptr, %b_ind_off : tensor<32x128x!tt.ptr<i32>, #BL>, tensor<32x128xi32, #BL>
300|     %next_a_ptr = tt.addptr %a_ptr, %a_off {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 32]> : tensor<2xi32>, tt.constancy = dense<[1, 1]> : tensor<2xi32>} : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xi32, #AL>
301|     %next_b_ptr = tt.addptr %b_ptr, %b_off {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 32]> : tensor<2xi32>, tt.constancy = dense<[1, 1]> : tensor<2xi32>} : tensor<32x128x!tt.ptr<f16>, #BL>, tensor<32x128xi32, #BL>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on pointer arithmetic, masked or vectorized loads. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 指针算术、带掩码或向量化的加载。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 302-304
```mlir
302|     // CHECK: tt.load {{.*}} {tt.latency = 1 : i32}
303|     %a_ = tt.load %next_a_ptr : tensor<128x32x!tt.ptr<f16>, #AL>
304|     %a = ttg.convert_layout %a_ : tensor<128x32xf16, #AL> -> tensor<128x32xf16, #A>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 305-314
```mlir
305|     // CHECK: tt.load {{.*}} {tt.latency = 1 : i32}
306|     %b_ = tt.load %next_b_ptr : tensor<32x128x!tt.ptr<f16>, #BL>
307|     %b = ttg.convert_layout %b_ : tensor<32x128xf16, #BL> -> tensor<32x128xf16, #B>
308| 
309|     %c = tt.dot %a, %b, %prev_c : tensor<128x32xf16, #A> * tensor<32x128xf16, #B> -> tensor<128x128xf32, #C>
310|     scf.yield %next_a_ind_ptr, %next_b_ind_ptr, %next_a_ptr, %next_b_ptr, %c : tensor<128x32x!tt.ptr<i32>, #AL>, tensor<32x128x!tt.ptr<i32>, #BL>, tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>
311|   } {tt.num_stages = 2 : i32}
312|   tt.return %loop#4: tensor<128x128xf32, #C>
313| }
314| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining masked or vectorized loads, layout conversions, dot-product or MMA-style math, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 带掩码或向量化的加载、布局转换、点积或 MMA 风格计算、循环/分支产出值、tt.return。

### Lines 315-315
```mlir
315| // CHECK-LABEL: @non_dot_pipeline
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @non_dot_pipeline anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @non_dot_pipeline 这样的标签用于锚定匹配范围。

### Lines 316-321
```mlir
316| tt.func @non_dot_pipeline(%lb : index, %ub : index, %step : index,
317|                   %a_ptr_init : tensor<128x32x!tt.ptr<f16>, #AL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 32]> : tensor<2xi32>}) -> tensor<128x32xf16, #A> {
318|   %c_init = arith.constant dense<0.00e+00> : tensor<128x32xf16, #A>
319|   %a_off = arith.constant dense<4> : tensor<128x32xi32, #AL>
320| 
321|   %loop:2 = scf.for %iv = %lb to %ub step %step iter_args(%a_ptr = %a_ptr_init, %prev_c = %c_init) -> (tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xf16, #A>) {
```
**EN:** This function-oriented block defines or enters `non_dot_pipeline`. Within it, the test exercises tt.func, constants, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `non_dot_pipeline` 为核心。测试在其中演示 tt.func、常量、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 322-333
```mlir
322|     // CHECK: tt.load {{.*}} {tt.latency = 2 : i32}
323|     %a_ = tt.load %a_ptr : tensor<128x32x!tt.ptr<f16>, #AL>
324|     %a = ttg.convert_layout %a_ : tensor<128x32xf16, #AL> -> tensor<128x32xf16, #A>
325| 
326|     %c = arith.addf %a, %prev_c : tensor<128x32xf16, #A>
327| 
328|     %next_a_ptr = tt.addptr %a_ptr, %a_off : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xi32, #AL>
329|     scf.yield %next_a_ptr, %c : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xf16, #A>
330|   } {tt.num_stages = 3 : i32}
331|   tt.return %loop#1: tensor<128x32xf16, #A>
332| }
333| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining masked or vectorized loads, layout conversions, floating-point additions, pointer arithmetic, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 带掩码或向量化的加载、布局转换、浮点加法、指针算术、循环/分支产出值。

### Lines 334-334
```mlir
334| // CHECK-LABEL: @no_pipeline
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @no_pipeline anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @no_pipeline 这样的标签用于锚定匹配范围。

### Lines 335-340
```mlir
335| tt.func @no_pipeline(%lb : index, %ub : index, %step : index,
336|                   %a_ptr_init : tensor<128x32x!tt.ptr<f16>, #AL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 32]> : tensor<2xi32>}) -> tensor<128x32xf16, #A> {
337|   %c_init = arith.constant dense<0.00e+00> : tensor<128x32xf16, #A>
338|   %a_off = arith.constant dense<4> : tensor<128x32xi32, #AL>
339| 
340|   %loop:2 = scf.for %iv = %lb to %ub step %step iter_args(%a_ptr = %a_ptr_init, %prev_c = %c_init) -> (tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xf16, #A>) {
```
**EN:** This function-oriented block defines or enters `no_pipeline`. Within it, the test exercises tt.func, constants, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `no_pipeline` 为核心。测试在其中演示 tt.func、常量、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 341-353
```mlir
341|     // CHECK: tt.load
342|     // CHECK-NOT: tt.latency
343|     %a_ = tt.load %a_ptr : tensor<128x32x!tt.ptr<f16>, #AL>
344|     %a = ttg.convert_layout %a_ : tensor<128x32xf16, #AL> -> tensor<128x32xf16, #A>
345| 
346|     %c = arith.addf %a, %prev_c : tensor<128x32xf16, #A>
347| 
348|     %next_a_ptr = tt.addptr %a_ptr, %a_off : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xi32, #AL>
349|     scf.yield %next_a_ptr, %c : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xf16, #A>
350|   }
351|   tt.return %loop#1: tensor<128x32xf16, #A>
352| }
353| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining masked or vectorized loads, layout conversions, floating-point additions, pointer arithmetic, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 带掩码或向量化的加载、布局转换、浮点加法、指针算术、循环/分支产出值。

### Lines 354-354
```mlir
354| // CHECK-LABEL: @intermediate_use
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @intermediate_use anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @intermediate_use 这样的标签用于锚定匹配范围。

### Lines 355-363
```mlir
355| tt.func @intermediate_use_cust_stages(%lb : index, %ub : index, %step : index,
356|                   %a_ptr_init : tensor<128x32x!tt.ptr<f16>, #AL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 32]> : tensor<2xi32>},
357|                   %b_ptr_init : tensor<32x128x!tt.ptr<f16>, #BL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 32]> : tensor<2xi32>}) -> tensor<128x128xf32, #C> {
358|   %c_init = arith.constant dense<0.00e+00> : tensor<128x128xf32, #C>
359|   %a_off = arith.constant dense<4> : tensor<128x32xi32, #AL>
360|   %b_off = arith.constant dense<4> : tensor<32x128xi32, #BL>
361|   %c2 = arith.constant dense<2.00> : tensor<32x128xf16, #BL>
362| 
363|   %loop:3 = scf.for %iv = %lb to %ub step %step iter_args(%a_ptr = %a_ptr_init, %b_ptr = %b_ptr_init, %prev_c = %c_init) -> (tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>) {
```
**EN:** This function-oriented block defines or enters `intermediate_use_cust_stages`. Within it, the test exercises constants, tt.func, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `intermediate_use_cust_stages` 为核心。测试在其中演示 常量、tt.func、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 364-366
```mlir
364|     // CHECK: tt.load {{.*}} {tt.latency = 2 : i32}
365|     %a_ = tt.load %a_ptr : tensor<128x32x!tt.ptr<f16>, #AL>
366|     %a = ttg.convert_layout %a_ : tensor<128x32xf16, #AL> -> tensor<128x32xf16, #A>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 367-381
```mlir
367|     // CHECK: tt.load {{.*}} {tt.latency = 2 : i32}
368|     %b_ = tt.load %b_ptr : tensor<32x128x!tt.ptr<f16>, #BL>
369|     %b_2 = arith.mulf %b_ , %c2 : tensor<32x128xf16, #BL>
370|     %b = ttg.convert_layout %b_2 : tensor<32x128xf16, #BL> -> tensor<32x128xf16, #B>
371| 
372|     %c = tt.dot %a, %b, %prev_c : tensor<128x32xf16, #A> * tensor<32x128xf16, #B> -> tensor<128x128xf32, #C>
373| 
374|     %next_a_ptr = tt.addptr %a_ptr, %a_off : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xi32, #AL>
375|     %next_b_ptr = tt.addptr %b_ptr, %b_off : tensor<32x128x!tt.ptr<f16>, #BL>, tensor<32x128xi32, #BL>
376|     scf.yield %next_a_ptr, %next_b_ptr, %c : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>
377|   } {tt.num_stages = 3 : i32}
378|   tt.return %loop#2: tensor<128x128xf32, #C>
379| }
380| }
381| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining pointer arithmetic, masked or vectorized loads, arith.mulf, layout conversions, dot-product or MMA-style math.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 指针算术、带掩码或向量化的加载、arith.mulf、布局转换、点积或 MMA 风格计算。

### Lines 382-382
```mlir
382| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 383-388
```mlir
383| 
384| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
385| #blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
386| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
387| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
388| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 389-389
```mlir
389| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 390-390
```mlir
390| // CHECK-LABEL: @tc_gen5_mma_overwrite_acc
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tc_gen5_mma_overwrite_acc anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tc_gen5_mma_overwrite_acc 这样的标签用于锚定匹配范围。

### Lines 391-397
```mlir
391| tt.func @tc_gen5_mma_overwrite_acc(%lb : index, %ub : index, %step : index,
392|                   %A_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
393|                   %B_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
394|                   %acc_init : tensor<128x128xf32, #blocked1>) -> () {
395|   %true = arith.constant true
396|   %acc_tm = ttng.tmem_alloc %acc_init : (tensor<128x128xf32, #blocked1>) -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
397|   scf.for %iv = %lb to %ub step %step : index {
```
**EN:** This function-oriented block defines or enters `tc_gen5_mma_overwrite_acc`. Within it, the test exercises tt.func, constants, tensor-memory allocation, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tc_gen5_mma_overwrite_acc` 为核心。测试在其中演示 tt.func、常量、张量内存分配、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 398-399
```mlir
398|     // CHECK: tt.load {{.*}} {tt.latency = 2 : i32}
399|     %A = tt.load %A_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 400-403
```mlir
400|     // CHECK: tt.load {{.*}} {tt.latency = 2 : i32}
401|     %B = tt.load %B_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
402|     %A_sh = ttg.local_alloc %A : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
403|     %B_sh = ttg.local_alloc %B : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, masked or vectorized loads. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、带掩码或向量化的加载。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 404-413
```mlir
404|     // CHECK: ttng.tc_gen5_mma {{.*}} {tt.latency = 1 : i32, tt.self_latency = 1 : i32}
405|     ttng.tmem_store %acc_init, %acc_tm, %true : tensor<128x128xf32, #blocked1> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
406|     ttng.tc_gen5_mma %A_sh, %B_sh, %acc_tm, %true, %true : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
407|     %acc_res = ttng.tmem_load %acc_tm : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
408|     "use"(%acc_res) : (tensor<128x128xf32, #blocked1>) -> ()
409|   }
410|   tt.return
411| }
412| }
413| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_store, Gen5 tensor-core MMA ops, ttng.tmem_load, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_store、Gen5 张量核 MMA 操作、ttng.tmem_load、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 414-414
```mlir
414| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 415-420
```mlir
415| 
416| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
417| #blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
418| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
419| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
420| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 421-421
```mlir
421| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 422-422
```mlir
422| // CHECK-LABEL: @tc_gen5_mma_acc_use_false
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tc_gen5_mma_acc_use_false anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tc_gen5_mma_acc_use_false 这样的标签用于锚定匹配范围。

### Lines 423-430
```mlir
423| tt.func @tc_gen5_mma_acc_use_false(%lb : index, %ub : index, %step : index,
424|                   %A_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
425|                   %B_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
426|                   %acc_init : tensor<128x128xf32, #blocked1>) -> () {
427|   %true = arith.constant true
428|   %false = arith.constant false
429|   %acc_tm = ttng.tmem_alloc %acc_init : (tensor<128x128xf32, #blocked1>) -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
430|   scf.for %iv = %lb to %ub step %step : index {
```
**EN:** This function-oriented block defines or enters `tc_gen5_mma_acc_use_false`. Within it, the test exercises tt.func, constants, tensor-memory allocation, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tc_gen5_mma_acc_use_false` 为核心。测试在其中演示 tt.func、常量、张量内存分配、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 431-432
```mlir
431|     // CHECK: tt.load {{.*}} {tt.latency = 2 : i32}
432|     %A = tt.load %A_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 433-436
```mlir
433|     // CHECK: tt.load {{.*}} {tt.latency = 2 : i32}
434|     %B = tt.load %B_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
435|     %A_sh = ttg.local_alloc %A : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
436|     %B_sh = ttg.local_alloc %B : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, masked or vectorized loads. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、带掩码或向量化的加载。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 437-445
```mlir
437|     // CHECK: ttng.tc_gen5_mma {{.*}} {tt.latency = 1 : i32, tt.self_latency = 1 : i32}
438|     ttng.tc_gen5_mma %A_sh, %B_sh, %acc_tm, %false, %true : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
439|     %acc_res = ttng.tmem_load %acc_tm : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
440|     "use"(%acc_res) : (tensor<128x128xf32, #blocked1>) -> ()
441|   }
442|   tt.return
443| }
444| }
445| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on Gen5 tensor-core MMA ops, ttng.tmem_load, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 Gen5 张量核 MMA 操作、ttng.tmem_load、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 446-446
```mlir
446| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 447-452
```mlir
447| 
448| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
449| #blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
450| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
451| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
452| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 453-453
```mlir
453| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 454-454
```mlir
454| // CHECK-LABEL: @tc_gen5_mma_acc_use_false
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tc_gen5_mma_acc_use_false anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tc_gen5_mma_acc_use_false 这样的标签用于锚定匹配范围。

### Lines 455-463
```mlir
455| tt.func @tc_gen5_mma_acc_use_false(%lb : index, %ub : index, %step : index,
456|                   %A_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
457|                   %B_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
458|                   %acc_init : tensor<128x128xf32, #blocked1>,
459|                   %acc_use_init : i1) -> () {
460|   %true = arith.constant true
461|   %false = arith.constant false
462|   %acc_tm = ttng.tmem_alloc %acc_init : (tensor<128x128xf32, #blocked1>) -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
463|   scf.for %iv = %lb to %ub step %step : index {
```
**EN:** This function-oriented block defines or enters `tc_gen5_mma_acc_use_false`. Within it, the test exercises tt.func, constants, tensor-memory allocation, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tc_gen5_mma_acc_use_false` 为核心。测试在其中演示 tt.func、常量、张量内存分配、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 464-465
```mlir
464|     // CHECK: tt.load {{.*}} {tt.latency = 2 : i32}
465|     %A = tt.load %A_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 466-470
```mlir
466|     // CHECK: tt.load {{.*}} {tt.latency = 2 : i32}
467|     %B = tt.load %B_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
468|     %A_sh = ttg.local_alloc %A : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
469|     %B_sh = ttg.local_alloc %B : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
470|     %acc_use = arith.xori %acc_use_init, %true : i1
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, masked or vectorized loads, arith.xori. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、带掩码或向量化的加载、arith.xori。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 471-479
```mlir
471|     // CHECK: ttng.tc_gen5_mma {{.*}} {tt.latency = 1 : i32, tt.self_latency = 1 : i32}
472|     ttng.tc_gen5_mma %A_sh, %B_sh, %acc_tm, %acc_use, %true : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
473|     %acc_res = ttng.tmem_load %acc_tm : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
474|     "use"(%acc_res) : (tensor<128x128xf32, #blocked1>) -> ()
475|   }
476|   tt.return
477| }
478| }
479| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on Gen5 tensor-core MMA ops, ttng.tmem_load, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 Gen5 张量核 MMA 操作、ttng.tmem_load、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 480-480
```mlir
480| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 481-486
```mlir
481| 
482| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
483| #blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
484| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
485| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
486| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 487-487
```mlir
487| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 488-488
```mlir
488| // CHECK-LABEL: @tc_gen5_mma_acc_use_false_dist_1
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tc_gen5_mma_acc_use_false_dist_1 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tc_gen5_mma_acc_use_false_dist_1 这样的标签用于锚定匹配范围。

### Lines 489-497
```mlir
489| tt.func @tc_gen5_mma_acc_use_false_dist_1(%lb : index, %ub : index, %step : index,
490|                   %A_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
491|                   %B_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
492|                   %acc_init : tensor<128x128xf32, #blocked1>,
493|                   %acc_use_init : i1) -> () {
494|   %true = arith.constant true
495|   %false = arith.constant false
496|   %acc_tm = ttng.tmem_alloc %acc_init : (tensor<128x128xf32, #blocked1>) -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
497|   scf.for %iv = %lb to %ub step %step iter_args(%acc_use = %acc_use_init) -> (i1) {
```
**EN:** This function-oriented block defines or enters `tc_gen5_mma_acc_use_false_dist_1`. Within it, the test exercises tt.func, constants, tensor-memory allocation, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tc_gen5_mma_acc_use_false_dist_1` 为核心。测试在其中演示 tt.func、常量、张量内存分配、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 498-499
```mlir
498|     // CHECK: tt.load {{.*}} {tt.latency = 2 : i32}
499|     %A = tt.load %A_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 500-503
```mlir
500|     // CHECK: tt.load {{.*}} {tt.latency = 2 : i32}
501|     %B = tt.load %B_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
502|     %A_sh = ttg.local_alloc %A : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
503|     %B_sh = ttg.local_alloc %B : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, masked or vectorized loads. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、带掩码或向量化的加载。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 504-514
```mlir
504|     // CHECK: ttng.tc_gen5_mma {{.*}} {tt.latency = 1 : i32, tt.self_latency = 1 : i32}
505|     ttng.tc_gen5_mma %A_sh, %B_sh, %acc_tm, %acc_use, %true : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
506|     %acc_res = ttng.tmem_load %acc_tm : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
507|     "use"(%acc_res) : (tensor<128x128xf32, #blocked1>) -> ()
508|     %acc_use_next = arith.xori %acc_use, %true : i1
509|     scf.yield %acc_use_next : i1
510|   }
511|   tt.return
512| }
513| }
514| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining Gen5 tensor-core MMA ops, ttng.tmem_load, arith.xori, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 Gen5 张量核 MMA 操作、ttng.tmem_load、arith.xori、循环/分支产出值、tt.return。

### Lines 515-515
```mlir
515| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 516-521
```mlir
516| 
517| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
518| #blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
519| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
520| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
521| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 522-522
```mlir
522| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 523-523
```mlir
523| // CHECK-LABEL: @tc_gen5_mma_acc_use_false_outside_loop
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tc_gen5_mma_acc_use_false_outside_loop anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tc_gen5_mma_acc_use_false_outside_loop 这样的标签用于锚定匹配范围。

### Lines 524-533
```mlir
524| tt.func @tc_gen5_mma_acc_use_false_outside_loop(%lb : index, %ub : index, %step : index,
525|                   %A_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
526|                   %B_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
527|                   %acc_init : tensor<128x128xf32, #blocked1>,
528|                   %acc_use_init : i1) -> () {
529|   %true = arith.constant true
530|   %false = arith.constant false
531|   %acc_tm = ttng.tmem_alloc %acc_init : (tensor<128x128xf32, #blocked1>) -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
532|   %acc_use = arith.xori %acc_use_init, %true : i1
533|   scf.for %iv = %lb to %ub step %step : index {
```
**EN:** This function-oriented block defines or enters `tc_gen5_mma_acc_use_false_outside_loop`. Within it, the test exercises tt.func, constants, tensor-memory allocation, arith.xori, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tc_gen5_mma_acc_use_false_outside_loop` 为核心。测试在其中演示 tt.func、常量、张量内存分配、arith.xori、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 534-535
```mlir
534|     // CHECK: tt.load {{.*}} {tt.latency = 2 : i32}
535|     %A = tt.load %A_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 536-539
```mlir
536|     // CHECK: tt.load {{.*}} {tt.latency = 2 : i32}
537|     %B = tt.load %B_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
538|     %A_sh = ttg.local_alloc %A : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
539|     %B_sh = ttg.local_alloc %B : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, masked or vectorized loads. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、带掩码或向量化的加载。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 540-548
```mlir
540|     // CHECK: ttng.tc_gen5_mma {{.*}} {tt.self_latency = 1 : i32}
541|     ttng.tc_gen5_mma %A_sh, %B_sh, %acc_tm, %acc_use, %true : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
542|     %acc_res = ttng.tmem_load %acc_tm : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
543|     "use"(%acc_res) : (tensor<128x128xf32, #blocked1>) -> ()
544|   }
545|   tt.return
546| }
547| }
548| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on Gen5 tensor-core MMA ops, ttng.tmem_load, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 Gen5 张量核 MMA 操作、ttng.tmem_load、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 549-549
```mlir
549| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 550-555
```mlir
550| 
551| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
552| #blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
553| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
554| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
555| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 556-556
```mlir
556| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 557-557
```mlir
557| // CHECK-LABEL: @tc_gen5_mma_overwrite_acc_outside_loop
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tc_gen5_mma_overwrite_acc_outside_loop anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tc_gen5_mma_overwrite_acc_outside_loop 这样的标签用于锚定匹配范围。

### Lines 558-565
```mlir
558| tt.func @tc_gen5_mma_overwrite_acc_outside_loop(%lb : index, %ub : index, %step : index,
559|                   %A_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
560|                   %B_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
561|                   %acc_init : tensor<128x128xf32, #blocked1>) -> () {
562|   %true = arith.constant true
563|   %acc_tm = ttng.tmem_alloc %acc_init : (tensor<128x128xf32, #blocked1>) -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
564|   ttng.tmem_store %acc_init, %acc_tm, %true : tensor<128x128xf32, #blocked1> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
565|   scf.for %iv = %lb to %ub step %step : index {
```
**EN:** This function-oriented block defines or enters `tc_gen5_mma_overwrite_acc_outside_loop`. Within it, the test exercises tt.func, constants, tensor-memory allocation, ttng.tmem_store, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tc_gen5_mma_overwrite_acc_outside_loop` 为核心。测试在其中演示 tt.func、常量、张量内存分配、ttng.tmem_store、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 566-567
```mlir
566|     // CHECK: tt.load {{.*}} {tt.latency = 2 : i32}
567|     %A = tt.load %A_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 568-571
```mlir
568|     // CHECK: tt.load {{.*}} {tt.latency = 2 : i32}
569|     %B = tt.load %B_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
570|     %A_sh = ttg.local_alloc %A : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
571|     %B_sh = ttg.local_alloc %B : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, masked or vectorized loads. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、带掩码或向量化的加载。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 572-580
```mlir
572|     // CHECK: ttng.tc_gen5_mma {{.*}} {tt.self_latency = 1 : i32}
573|     ttng.tc_gen5_mma %A_sh, %B_sh, %acc_tm, %true, %true : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
574|     %acc_res = ttng.tmem_load %acc_tm : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
575|     "use"(%acc_res) : (tensor<128x128xf32, #blocked1>) -> ()
576|   }
577|   tt.return
578| }
579| }
580| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on Gen5 tensor-core MMA ops, ttng.tmem_load, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 Gen5 张量核 MMA 操作、ttng.tmem_load、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 581-581
```mlir
581| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 582-587
```mlir
582| 
583| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
584| #blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
585| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
586| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
587| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 588-588
```mlir
588| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 589-589
```mlir
589| // CHECK-LABEL: @tc_gen5_mma_overwrite_acc
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tc_gen5_mma_overwrite_acc anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tc_gen5_mma_overwrite_acc 这样的标签用于锚定匹配范围。

### Lines 590-596
```mlir
590| tt.func @tc_gen5_mma_overwrite_acc_small_load(%lb : index, %ub : index, %step : index,
591|                   %A_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1>,
592|                   %B_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1>,
593|                   %acc_init : tensor<128x128xf32, #blocked1>) -> () {
594|   %true = arith.constant true
595|   %acc_tm = ttng.tmem_alloc %acc_init : (tensor<128x128xf32, #blocked1>) -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
596|   scf.for %iv = %lb to %ub step %step : index {
```
**EN:** This function-oriented block defines or enters `tc_gen5_mma_overwrite_acc_small_load`. Within it, the test exercises tt.func, constants, tensor-memory allocation, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tc_gen5_mma_overwrite_acc_small_load` 为核心。测试在其中演示 tt.func、常量、张量内存分配、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 597-599
```mlir
597|     // CHECK: tt.load
598|     // CHECK-NOT: tt.latency
599|     %A = tt.load %A_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized loads. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的加载。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 600-604
```mlir
600|     // CHECK: tt.load
601|     // CHECK-NOT: tt.latency
602|     %B = tt.load %B_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
603|     %A_sh = ttg.local_alloc %A : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
604|     %B_sh = ttg.local_alloc %B : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, masked or vectorized loads. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、带掩码或向量化的加载。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 605-616
```mlir
605|     // CHECK: ttng.tc_gen5_mma
606|     // CHECK-NOT: tt.latency
607|     // CHECK-NOT: tt.self_latency
608|     ttng.tmem_store %acc_init, %acc_tm, %true : tensor<128x128xf32, #blocked1> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
609|     ttng.tc_gen5_mma %A_sh, %B_sh, %acc_tm, %true, %true : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
610|     %acc_res = ttng.tmem_load %acc_tm : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
611|     "use"(%acc_res) : (tensor<128x128xf32, #blocked1>) -> ()
612|   }
613|   tt.return
614| }
615| }
616| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_store, Gen5 tensor-core MMA ops, ttng.tmem_load, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_store、Gen5 张量核 MMA 操作、ttng.tmem_load、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 617-617
```mlir
617| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 618-623
```mlir
618| 
619| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
620| #blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
621| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
622| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
623| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 624-624
```mlir
624| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 625-625
```mlir
625| // CHECK-LABEL: @tc_gen5_mma_B_outside
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tc_gen5_mma_B_outside anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tc_gen5_mma_B_outside 这样的标签用于锚定匹配范围。

### Lines 626-635
```mlir
626| tt.func @tc_gen5_mma_B_outside(%lb : index, %ub : index, %step : index,
627|                   %A_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
628|                   %B: tensor<128x128xf16, #blocked1>,
629|                   %acc_init : tensor<128x128xf32, #blocked1>) -> () {
630|   %true = arith.constant true
631|   %acc_tm = ttng.tmem_alloc %acc_init : (tensor<128x128xf32, #blocked1>) -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
632|   scf.for %iv = %lb to %ub step %step : index {
633|     %A = tt.load %A_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
634|     %A_sh = ttg.local_alloc %A : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
635|     %B_sh = ttg.local_alloc %B : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
```
**EN:** This function-oriented block defines or enters `tc_gen5_mma_B_outside`. Within it, the test exercises tt.func, shared/local memory allocation, constants, tensor-memory allocation, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tc_gen5_mma_B_outside` 为核心。测试在其中演示 tt.func、共享/本地内存分配、常量、张量内存分配、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 636-645
```mlir
636|     // CHECK: ttng.tc_gen5_mma {{.*}} {tt.latency = 1 : i32, tt.self_latency = 1 : i32}
637|     ttng.tmem_store %acc_init, %acc_tm, %true : tensor<128x128xf32, #blocked1> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
638|     ttng.tc_gen5_mma %A_sh, %B_sh, %acc_tm, %true, %true : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
639|     %acc_res = ttng.tmem_load %acc_tm : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
640|     "use"(%acc_res) : (tensor<128x128xf32, #blocked1>) -> ()
641|   }
642|   tt.return
643| }
644| }
645| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_store, Gen5 tensor-core MMA ops, ttng.tmem_load, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_store、Gen5 张量核 MMA 操作、ttng.tmem_load、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 646-646
```mlir
646| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 647-652
```mlir
647| 
648| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
649| #blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
650| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
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
654| // CHECK-LABEL: @tc_gen5_mma_disallow_multibuffer
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tc_gen5_mma_disallow_multibuffer anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tc_gen5_mma_disallow_multibuffer 这样的标签用于锚定匹配范围。

### Lines 655-664
```mlir
655| tt.func @tc_gen5_mma_disallow_multibuffer(%lb : index, %ub : index, %step : index,
656|                   %A_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
657|                   %B: tensor<128x128xf16, #blocked1>,
658|                   %acc_init : tensor<128x128xf32, #blocked1>) -> () {
659|   %true = arith.constant true
660|   %acc_tm = ttng.tmem_alloc %acc_init : (tensor<128x128xf32, #blocked1>) -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
661|   scf.for %iv = %lb to %ub step %step : index {
662|     %A = tt.load %A_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
663|     %A_sh = ttg.local_alloc %A : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
664|     %B_sh = ttg.local_alloc %B : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
```
**EN:** This function-oriented block defines or enters `tc_gen5_mma_disallow_multibuffer`. Within it, the test exercises tt.func, shared/local memory allocation, constants, tensor-memory allocation, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tc_gen5_mma_disallow_multibuffer` 为核心。测试在其中演示 tt.func、共享/本地内存分配、常量、张量内存分配、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 665-674
```mlir
665|     // CHECK: ttng.tc_gen5_mma {{.*}} {tt.self_latency = 1 : i32}
666|     ttng.tmem_store %acc_init, %acc_tm, %true : tensor<128x128xf32, #blocked1> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
667|     ttng.tc_gen5_mma %A_sh, %B_sh, %acc_tm, %true, %true : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
668|     %acc_res = ttng.tmem_load %acc_tm : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
669|     "use"(%acc_res) : (tensor<128x128xf32, #blocked1>) -> ()
670|   } {tt.disallow_acc_multi_buffer}
671|   tt.return
672| }
673| }
674| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_store, Gen5 tensor-core MMA ops, ttng.tmem_load, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_store、Gen5 张量核 MMA 操作、ttng.tmem_load、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 675-675
```mlir
675| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 676-681
```mlir
676| 
677| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
678| #blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
679| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
680| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
681| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 682-682
```mlir
682| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 683-683
```mlir
683| // CHECK-LABEL: @tc_gen5_mma_B_outside2
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tc_gen5_mma_B_outside2 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tc_gen5_mma_B_outside2 这样的标签用于锚定匹配范围。

### Lines 684-692
```mlir
684| tt.func @tc_gen5_mma_B_outside2(%lb : index, %ub : index, %step : index,
685|                   %A_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
686|                   %B_sh: !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>,
687|                   %acc_init : tensor<128x128xf32, #blocked1>) -> () {
688|   %true = arith.constant true
689|   %acc_tm = ttng.tmem_alloc %acc_init : (tensor<128x128xf32, #blocked1>) -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
690|   scf.for %iv = %lb to %ub step %step : index {
691|     %A = tt.load %A_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
692|     %A_sh = ttg.local_alloc %A : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
```
**EN:** This function-oriented block defines or enters `tc_gen5_mma_B_outside2`. Within it, the test exercises tt.func, constants, tensor-memory allocation, structured loops, masked or vectorized loads, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tc_gen5_mma_B_outside2` 为核心。测试在其中演示 tt.func、常量、张量内存分配、结构化循环、带掩码或向量化的加载，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 693-702
```mlir
693|     // CHECK: ttng.tc_gen5_mma {{.*}} {tt.latency = 1 : i32, tt.self_latency = 1 : i32}
694|     ttng.tmem_store %acc_init, %acc_tm, %true : tensor<128x128xf32, #blocked1> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
695|     ttng.tc_gen5_mma %A_sh, %B_sh, %acc_tm, %true, %true : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
696|     %acc_res = ttng.tmem_load %acc_tm : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
697|     "use"(%acc_res) : (tensor<128x128xf32, #blocked1>) -> ()
698|   }
699|   tt.return
700| }
701| }
702| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_store, Gen5 tensor-core MMA ops, ttng.tmem_load, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_store、Gen5 张量核 MMA 操作、ttng.tmem_load、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 703-703
```mlir
703| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 704-709
```mlir
704| 
705| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
706| #blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
707| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
708| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
709| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 710-710
```mlir
710| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 711-711
```mlir
711| // CHECK-LABEL: @tc_gen5_mma_non_load_operand1
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tc_gen5_mma_non_load_operand1 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tc_gen5_mma_non_load_operand1 这样的标签用于锚定匹配范围。

### Lines 712-721
```mlir
712| tt.func @tc_gen5_mma_non_load_operand1(%lb : index, %ub : index, %step : index,
713|                   %A_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
714|                   %acc_init : tensor<128x128xf32, #blocked1>) -> () {
715|   %true = arith.constant true
716|   %acc_tm = ttng.tmem_alloc %acc_init : (tensor<128x128xf32, #blocked1>) -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
717|   scf.for %iv = %lb to %ub step %step : index {
718|     %A = tt.load %A_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
719|     %A_sh = ttg.local_alloc %A : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
720|     %B = "producer"() : () -> tensor<128x128xf16, #blocked1>
721|     %B_sh = ttg.local_alloc %B : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
```
**EN:** This function-oriented block defines or enters `tc_gen5_mma_non_load_operand1`. Within it, the test exercises tt.func, shared/local memory allocation, constants, tensor-memory allocation, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tc_gen5_mma_non_load_operand1` 为核心。测试在其中演示 tt.func、共享/本地内存分配、常量、张量内存分配、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 722-732
```mlir
722|     // CHECK: ttng.tc_gen5_mma
723|     // CHECK-NOT: tt.latency
724|     ttng.tmem_store %acc_init, %acc_tm, %true : tensor<128x128xf32, #blocked1> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
725|     ttng.tc_gen5_mma %A_sh, %B_sh, %acc_tm, %true, %true : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
726|     %acc_res = ttng.tmem_load %acc_tm : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
727|     "use"(%acc_res) : (tensor<128x128xf32, #blocked1>) -> ()
728|   }
729|   tt.return
730| }
731| }
732| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_store, Gen5 tensor-core MMA ops, ttng.tmem_load, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_store、Gen5 张量核 MMA 操作、ttng.tmem_load、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 733-733
```mlir
733| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 734-739
```mlir
734| 
735| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
736| #blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
737| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
738| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
739| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 740-740
```mlir
740| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 741-741
```mlir
741| // CHECK-LABEL: @tc_gen5_mma_non_load_operand2
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tc_gen5_mma_non_load_operand2 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tc_gen5_mma_non_load_operand2 这样的标签用于锚定匹配范围。

### Lines 742-750
```mlir
742| tt.func @tc_gen5_mma_non_load_operand2(%lb : index, %ub : index, %step : index,
743|                   %A_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
744|                   %acc_init : tensor<128x128xf32, #blocked1>) -> () {
745|   %true = arith.constant true
746|   %acc_tm = ttng.tmem_alloc %acc_init : (tensor<128x128xf32, #blocked1>) -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
747|   scf.for %iv = %lb to %ub step %step : index {
748|     %A = tt.load %A_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
749|     %A_sh = ttg.local_alloc %A : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
750|     %B_sh = "producer"() : () -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
```
**EN:** This function-oriented block defines or enters `tc_gen5_mma_non_load_operand2`. Within it, the test exercises tt.func, constants, tensor-memory allocation, structured loops, masked or vectorized loads, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tc_gen5_mma_non_load_operand2` 为核心。测试在其中演示 tt.func、常量、张量内存分配、结构化循环、带掩码或向量化的加载，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 751-762
```mlir
751|     // CHECK: ttng.tc_gen5_mma
752|     // CHECK-NOT: tt.latency
753|     // CHECK-NOT: tt.self_latency
754|     ttng.tmem_store %acc_init, %acc_tm, %true : tensor<128x128xf32, #blocked1> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
755|     ttng.tc_gen5_mma %A_sh, %B_sh, %acc_tm, %true, %true : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
756|     %acc_res = ttng.tmem_load %acc_tm : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
757|     "use"(%acc_res) : (tensor<128x128xf32, #blocked1>) -> ()
758|   }
759|   tt.return
760| }
761| }
762| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_store, Gen5 tensor-core MMA ops, ttng.tmem_load, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_store、Gen5 张量核 MMA 操作、ttng.tmem_load、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 763-763
```mlir
763| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 764-769
```mlir
764| 
765| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
766| #blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
767| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
768| #smem = #ttg.shared_memory
769| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 770-770
```mlir
770| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 771-771
```mlir
771|   // CHECK-LABEL: @select_after_mma
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @select_after_mma anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @select_after_mma 这样的标签用于锚定匹配范围。

### Lines 772-785
```mlir
772|   tt.func public @select_after_mma(%arg0: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>}, %arg1: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>}, %arg2: i32) -> tensor<128x128xf16, #blocked1> {
773|     %true = arith.constant true
774|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked1>
775|     %cst_0 = arith.constant dense<2.000000e+00> : tensor<128x128xf32, #blocked1>
776|     %c0_i32 = arith.constant 0 : i32
777|     %c1_i32 = arith.constant 1 : i32
778|     %0 = "cnd"() : () -> i1
779|     %1 = ttng.tmem_alloc  : () -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
780|     ttng.tmem_store %cst, %1, %true : tensor<128x128xf32, #blocked1> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
781|     scf.for %arg3 = %c0_i32 to %arg2 step %c1_i32  : i32 {
782|       %4 = tt.load %arg0 : tensor<128x128x!tt.ptr<f16>, #blocked>
783|       %5 = ttg.local_alloc %4 : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
784|       %6 = tt.load %arg1 : tensor<128x128x!tt.ptr<f16>, #blocked>
785|       %7 = ttg.local_alloc %6 : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `select_after_mma`. Within it, the test exercises constants, tt.func, masked or vectorized loads, shared/local memory allocation, tensor-memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `select_after_mma` 为核心。测试在其中演示 常量、tt.func、带掩码或向量化的加载、共享/本地内存分配、张量内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 786-796
```mlir
786|       // CHECK: ttng.tc_gen5_mma {{.*}} {tt.latency = 1 : i32, tt.self_latency = 1 : i32}
787|       ttng.tc_gen5_mma %5, %7, %1, %true, %true : !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
788|       %8 = arith.xori %0, %true : i1
789|       ttng.tmem_store %cst_0, %1, %8 : tensor<128x128xf32, #blocked1> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
790|     } {tt.scheduled_max_stage = 3 : i32}
791|     %2 = ttng.tmem_load %1 : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
792|     %3 = arith.truncf %2 : tensor<128x128xf32, #blocked1> to tensor<128x128xf16, #blocked1>
793|     tt.return %3 : tensor<128x128xf16, #blocked1>
794|   }
795| }
796| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on Gen5 tensor-core MMA ops, arith.xori, ttng.tmem_store, ttng.tmem_load, arith.truncf. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 Gen5 张量核 MMA 操作、arith.xori、ttng.tmem_store、ttng.tmem_load、arith.truncf。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 797-797
```mlir
797| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 798-807
```mlir
798| 
799| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
800| #blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
801| #blocked2 = #ttg.blocked<{sizePerThread = [1, 1, 1, 1, 4], threadsPerWarp = [1, 1, 8, 4, 1], warpsPerCTA = [1, 1, 4, 1, 1], order = [4, 3, 2, 1, 0]}>
802| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
803| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [4, 3, 2, 1, 0]}>
804| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
805| #tmem_scales = #ttng.tensor_memory_scales_encoding<>
806| #smem = #ttg.shared_memory
807| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 808-808
```mlir
808| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 809-809
```mlir
809| // CHECK-LABEL: @tc_gen5_mma_scaled
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tc_gen5_mma_scaled anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tc_gen5_mma_scaled 这样的标签用于锚定匹配范围。

### Lines 810-827
```mlir
810| tt.func @tc_gen5_mma_scaled(%lb : index, %ub : index, %step : index,
811|                   %A_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
812|                   %B_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
813|                   %A_sc_ptr: tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked2> {tt.divisibility = dense<[16, 16, 16, 16, 16]> : tensor<5xi32>, tt.contiguity = dense<[1, 1, 1, 1, 16]> : tensor<5xi32>},
814|                   %B_sc_ptr: tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked2> {tt.divisibility = dense<[16, 16, 16, 16, 16]> : tensor<5xi32>, tt.contiguity = dense<[1, 1, 1, 1, 16]> : tensor<5xi32>},
815|                   %acc_init : tensor<128x128xf32, #blocked1>) -> () {
816|   %true = arith.constant true
817|   %acc_tm = ttng.tmem_alloc %acc_init : (tensor<128x128xf32, #blocked1>) -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
818|   scf.for %iv = %lb to %ub step %step : index {
819|     %A = tt.load %A_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
820|     %A_sh = ttg.local_alloc %A : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
821|     %B = tt.load %B_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
822|     %B_sh = ttg.local_alloc %B : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
823| 
824|     %A_sc = tt.load %A_sc_ptr : tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked2>
825|     %A_sc_sh = ttg.local_alloc %A_sc : (tensor<1x2x32x4x4xi8, #blocked2>) -> !ttg.memdesc<1x2x32x4x4xi8, #shared1, #smem>
826| 
827|     %B_sc = tt.load %B_sc_ptr : tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked2>
```
**EN:** This function-oriented block defines or enters `tc_gen5_mma_scaled`. Within it, the test exercises masked or vectorized loads, shared/local memory allocation, tt.func, constants, tensor-memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tc_gen5_mma_scaled` 为核心。测试在其中演示 带掩码或向量化的加载、共享/本地内存分配、tt.func、常量、张量内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 828-829
```mlir
828|     %B_sc_sh = ttg.local_alloc %B_sc : (tensor<1x2x32x4x4xi8, #blocked2>) -> !ttg.memdesc<1x2x32x4x4xi8, #shared1, #smem>
829| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 830-839
```mlir
830|     // CHECK: ttng.tc_gen5_mma_scaled {{.*}} {tt.latency = 1 : i32, tt.self_latency = 1 : i32}
831|     ttng.tmem_store %acc_init, %acc_tm, %true : tensor<128x128xf32, #blocked1> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
832|     ttng.tc_gen5_mma_scaled %A_sh, %B_sh, %acc_tm, %A_sc_sh, %B_sc_sh, %true, %true lhs = e5m2 rhs = e5m2 : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.memdesc<1x2x32x4x4xi8, #shared1, #smem>, !ttg.memdesc<1x2x32x4x4xi8, #shared1, #smem>
833|     %acc_res = ttng.tmem_load %acc_tm : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
834|     "use"(%acc_res) : (tensor<128x128xf32, #blocked1>) -> ()
835|   }
836|   tt.return
837| }
838| }
839| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_store, scaled tensor-core MMA ops, ttng.tmem_load, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_store、带缩放的张量核 MMA 操作、ttng.tmem_load、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 840-840
```mlir
840| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 841-850
```mlir
841| 
842| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
843| #blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
844| #scales = #ttg.linear<{register = [[0, 1], [0, 2], [32, 0], [64, 0], [0, 4]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0]], warp = [[0, 0], [0, 0]], block = []}>
845| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
846| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
847| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
848| #tmem_scales = #ttng.tensor_memory_scales_encoding<>
849| #smem = #ttg.shared_memory
850| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 851-851
```mlir
851| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 852-852
```mlir
852| // CHECK-LABEL: @tc_gen5_mma_scaled_tmem_scales
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tc_gen5_mma_scaled_tmem_scales anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tc_gen5_mma_scaled_tmem_scales 这样的标签用于锚定匹配范围。

### Lines 853-870
```mlir
853| tt.func @tc_gen5_mma_scaled_tmem_scales(%lb : index, %ub : index, %step : index,
854|                   %A_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
855|                   %B_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
856|                   %A_sc_ptr: tensor<128x8x!tt.ptr<i8>, #scales> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
857|                   %B_sc_ptr: tensor<128x8x!tt.ptr<i8>, #scales> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
858|                   %acc_init : tensor<128x128xf32, #blocked1>) -> () {
859|   %true = arith.constant true
860|   %acc_tm = ttng.tmem_alloc %acc_init : (tensor<128x128xf32, #blocked1>) -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
861|   scf.for %iv = %lb to %ub step %step : index {
862|     %A = tt.load %A_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
863|     %A_sh = ttg.local_alloc %A : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
864|     %B = tt.load %B_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
865|     %B_sh = ttg.local_alloc %B : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
866| 
867|     %A_sc = tt.load %A_sc_ptr : tensor<128x8x!tt.ptr<i8>, #scales>
868|     %A_sc_sh = ttg.local_alloc %A_sc : (tensor<128x8xi8, #scales>) -> !ttg.memdesc<128x8xi8, #shared1, #smem>
869| 
870|     %B_sc = tt.load %B_sc_ptr : tensor<128x8x!tt.ptr<i8>, #scales>
```
**EN:** This function-oriented block defines or enters `tc_gen5_mma_scaled_tmem_scales`. Within it, the test exercises masked or vectorized loads, shared/local memory allocation, tt.func, constants, tensor-memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tc_gen5_mma_scaled_tmem_scales` 为核心。测试在其中演示 带掩码或向量化的加载、共享/本地内存分配、tt.func、常量、张量内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 871-872
```mlir
871|     %B_sc_tm = ttng.tmem_alloc %B_sc : (tensor<128x8xi8, #scales>) -> !ttg.memdesc<128x8xi8, #tmem_scales, #ttng.tensor_memory>
872| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor-memory allocation. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量内存分配。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 873-884
```mlir
873|     // CHECK: ttng.tc_gen5_mma_scaled {{.*}}
874|     // CHECK-NOT: tt.latency
875|     // CHECK-NOT: tt.self_latency
876|     ttng.tmem_store %acc_init, %acc_tm, %true : tensor<128x128xf32, #blocked1> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
877|     ttng.tc_gen5_mma_scaled %A_sh, %B_sh, %acc_tm, %A_sc_sh, %B_sc_tm, %true, %true lhs = e5m2 rhs = e5m2 : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.memdesc<128x8xi8, #shared1, #smem>, !ttg.memdesc<128x8xi8, #tmem_scales, #ttng.tensor_memory>
878|     %acc_res = ttng.tmem_load %acc_tm : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
879|     "use"(%acc_res) : (tensor<128x128xf32, #blocked1>) -> ()
880|   }
881|   tt.return
882| }
883| }
884| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_store, scaled tensor-core MMA ops, ttng.tmem_load, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_store、带缩放的张量核 MMA 操作、ttng.tmem_load、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 885-885
```mlir
885| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 886-894
```mlir
886| 
887| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
888| #blocked1 = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
889| #blocked2 = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
890| #blocked3 = #ttg.blocked<{sizePerThread = [1, 1, 1, 1, 4], threadsPerWarp = [1, 1, 8, 4, 1], warpsPerCTA = [1, 1, 4, 1, 1], order = [4, 3, 2, 1, 0]}>
891| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 8}>
892| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [4, 3, 2, 1, 0]}>
893| #smem = #ttg.shared_memory
894| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 895-895
```mlir
895| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 896-896
```mlir
896|   // CHECK-LABEL: @block_scale_mxfp_matmul
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @block_scale_mxfp_matmul anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @block_scale_mxfp_matmul 这样的标签用于锚定匹配范围。

### Lines 897-914
```mlir
897|   tt.func public @block_scale_mxfp_matmul(%arg0: index, %arg1: index, %arg2: index, %arg3: !tt.ptr<f8E5M2> {tt.divisibility = 16 : i32}, %arg4: !tt.ptr<f8E5M2> {tt.divisibility = 16 : i32}, %arg5: !tt.ptr<i8> {tt.divisibility = 16 : i32}, %arg6: !tt.ptr<i8> {tt.divisibility = 16 : i32}) -> tensor<128x128xf32, #blocked> {
898|     %true = arith.constant true
899|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
900|     %cst_0 = arith.constant dense<4> : tensor<128x256xi32, #blocked1>
901|     %cst_1 = arith.constant dense<4> : tensor<256x128xi32, #blocked2>
902|     %cst_2 = arith.constant dense<4> : tensor<1x2x32x4x4xi32, #blocked3>
903|     %0 = tt.splat %arg3 : !tt.ptr<f8E5M2> -> tensor<128x256x!tt.ptr<f8E5M2>, #blocked1>
904|     %1 = tt.splat %arg4 : !tt.ptr<f8E5M2> -> tensor<256x128x!tt.ptr<f8E5M2>, #blocked2>
905|     %2 = tt.splat %arg5 : !tt.ptr<i8> -> tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked3>
906|     %3 = tt.splat %arg6 : !tt.ptr<i8> -> tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked3>
907|     %4 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
908|     %5 = tt.expand_dims %4 {axis = 0 : i32} : tensor<256xi32, #ttg.slice<{dim = 0, parent = #blocked1}>> -> tensor<1x256xi32, #blocked1>
909|     %6 = tt.broadcast %5 : tensor<1x256xi32, #blocked1> -> tensor<128x256xi32, #blocked1>
910|     %7 = tt.addptr %0, %6 : tensor<128x256x!tt.ptr<f8E5M2>, #blocked1>, tensor<128x256xi32, #blocked1>
911|     %8 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked2}>>
912|     %9 = tt.expand_dims %8 {axis = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked2}>> -> tensor<1x128xi32, #blocked2>
913|     %10 = tt.broadcast %9 : tensor<1x128xi32, #blocked2> -> tensor<256x128xi32, #blocked2>
914|     %11 = tt.addptr %1, %10 : tensor<256x128x!tt.ptr<f8E5M2>, #blocked2>, tensor<256x128xi32, #blocked2>
```
**EN:** This function-oriented block defines or enters `block_scale_mxfp_matmul`. Within it, the test exercises constants, broadcasted scalars or pointers, tt.func, lane/block index ranges, shape expansion, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `block_scale_mxfp_matmul` 为核心。测试在其中演示 常量、广播后的标量或指针、tt.func、lane/block 索引范围、形状扩展，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 915-925
```mlir
915|     %12 = tt.make_range {end = 4 : i32, start = 0 : i32} : tensor<4xi32, #ttg.slice<{dim = 0, parent = #ttg.slice<{dim = 1, parent = #ttg.slice<{dim = 2, parent = #ttg.slice<{dim = 3, parent = #blocked3}>}>}>}>>
916|     %13 = tt.expand_dims %12 {axis = 0 : i32} : tensor<4xi32, #ttg.slice<{dim = 0, parent = #ttg.slice<{dim = 1, parent = #ttg.slice<{dim = 2, parent = #ttg.slice<{dim = 3, parent = #blocked3}>}>}>}>> -> tensor<1x4xi32, #ttg.slice<{dim = 1, parent = #ttg.slice<{dim = 2, parent = #ttg.slice<{dim = 3, parent = #blocked3}>}>}>>
917|     %14 = tt.expand_dims %13 {axis = 1 : i32} : tensor<1x4xi32, #ttg.slice<{dim = 1, parent = #ttg.slice<{dim = 2, parent = #ttg.slice<{dim = 3, parent = #blocked3}>}>}>> -> tensor<1x1x4xi32, #ttg.slice<{dim = 2, parent = #ttg.slice<{dim = 3, parent = #blocked3}>}>>
918|     %15 = tt.expand_dims %14 {axis = 2 : i32} : tensor<1x1x4xi32, #ttg.slice<{dim = 2, parent = #ttg.slice<{dim = 3, parent = #blocked3}>}>> -> tensor<1x1x1x4xi32, #ttg.slice<{dim = 3, parent = #blocked3}>>
919|     %16 = tt.expand_dims %15 {axis = 3 : i32} : tensor<1x1x1x4xi32, #ttg.slice<{dim = 3, parent = #blocked3}>> -> tensor<1x1x1x1x4xi32, #blocked3>
920|     %17 = tt.broadcast %16 : tensor<1x1x1x1x4xi32, #blocked3> -> tensor<1x2x32x4x4xi32, #blocked3>
921|     %18 = tt.addptr %2, %17 : tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked3>, tensor<1x2x32x4x4xi32, #blocked3>
922|     %19 = tt.addptr %3, %17 : tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked3>, tensor<1x2x32x4x4xi32, #blocked3>
923|     %20 = ttng.tmem_alloc  : () -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
924|     ttng.tmem_store %cst, %20, %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
925|     %21:4 = scf.for %arg7 = %arg0 to %arg1 step %arg2 iter_args(%arg8 = %7, %arg9 = %11, %arg10 = %18, %arg11 = %19) -> (tensor<128x256x!tt.ptr<f8E5M2>, #blocked1>, tensor<256x128x!tt.ptr<f8E5M2>, #blocked2>, tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked3>, tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked3>) {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining shape expansion, pointer arithmetic, lane/block index ranges, tensor broadcasting, tensor-memory allocation.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 形状扩展、指针算术、lane/block 索引范围、张量广播、张量内存分配。

### Lines 926-928
```mlir
926|       // CHECK: tt.load {{.*}} {tt.latency = 2 : i32}
927|       %22 = tt.load %arg8 : tensor<128x256x!tt.ptr<f8E5M2>, #blocked1>
928|       %23 = ttg.local_alloc %22 : (tensor<128x256xf8E5M2, #blocked1>) -> !ttg.memdesc<128x256xf8E5M2, #shared, #smem>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 929-931
```mlir
929|       // CHECK: tt.load {{.*}} {tt.latency = 2 : i32}
930|       %24 = tt.load %arg9 : tensor<256x128x!tt.ptr<f8E5M2>, #blocked2>
931|       %25 = ttg.local_alloc %24 : (tensor<256x128xf8E5M2, #blocked2>) -> !ttg.memdesc<256x128xf8E5M2, #shared, #smem>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 932-933
```mlir
932|       // CHECK: tt.load {{.*}} {tt.latency = 2 : i32}
933|       %26 = tt.load %arg10 : tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked3>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 934-937
```mlir
934|       // CHECK: tt.load {{.*}} {tt.latency = 2 : i32}
935|       %27 = tt.load %arg11 : tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked3>
936|       %28 = ttg.local_alloc %26 : (tensor<1x2x32x4x4xi8, #blocked3>) -> !ttg.memdesc<1x2x32x4x4xi8, #shared1, #smem>
937|       %29 = ttg.local_alloc %27 : (tensor<1x2x32x4x4xi8, #blocked3>) -> !ttg.memdesc<1x2x32x4x4xi8, #shared1, #smem>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, masked or vectorized loads. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、带掩码或向量化的加载。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 938-949
```mlir
938|       // CHECK: ttng.tc_gen5_mma_scaled {{.*}} {tt.latency = 1 : i32, tt.self_latency = 1 : i32}
939|       ttng.tc_gen5_mma_scaled %23, %25, %20, %28, %29, %true, %true lhs = e5m2 rhs = e5m2 : !ttg.memdesc<128x256xf8E5M2, #shared, #smem>, !ttg.memdesc<256x128xf8E5M2, #shared, #smem>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.memdesc<1x2x32x4x4xi8, #shared1, #smem>, !ttg.memdesc<1x2x32x4x4xi8, #shared1, #smem>
940|       %30 = tt.addptr %arg8, %cst_0 : tensor<128x256x!tt.ptr<f8E5M2>, #blocked1>, tensor<128x256xi32, #blocked1>
941|       %31 = tt.addptr %arg9, %cst_1 : tensor<256x128x!tt.ptr<f8E5M2>, #blocked2>, tensor<256x128xi32, #blocked2>
942|       %32 = tt.addptr %arg10, %cst_2 : tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked3>, tensor<1x2x32x4x4xi32, #blocked3>
943|       %33 = tt.addptr %arg11, %cst_2 : tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked3>, tensor<1x2x32x4x4xi32, #blocked3>
944|       scf.yield %30, %31, %32, %33 : tensor<128x256x!tt.ptr<f8E5M2>, #blocked1>, tensor<256x128x!tt.ptr<f8E5M2>, #blocked2>, tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked3>, tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked3>
945|     } {tt.num_stages = 3 : i32}
946|     tt.return %cst : tensor<128x128xf32, #blocked>
947|   }
948| }
949| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining pointer arithmetic, scaled tensor-core MMA ops, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 指针算术、带缩放的张量核 MMA 操作、循环/分支产出值、tt.return。

### Lines 950-950
```mlir
950| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 951-955
```mlir
951| 
952| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
953| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
954| #smem = #ttg.shared_memory
955| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 956-956
```mlir
956| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 957-957
```mlir
957|   // CHECK-LABEL: @two_dots
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @two_dots anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @two_dots 这样的标签用于锚定匹配范围。

### Lines 958-964
```mlir
958|   tt.func public @two_dots(%arg0: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>}, %arg1: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>}, %arg2: tensor<128x128x!tt.ptr<f32>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>}, %arg3: tensor<128x128x!tt.ptr<f32>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>}, %arg4: i32) {
959|     %true = arith.constant true
960|     %c0_i32 = arith.constant 0 : i32
961|     %c1_i32 = arith.constant 1 : i32
962|     %0 = ttng.tmem_alloc  : () -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
963|     %1 = ttng.tmem_alloc  : () -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
964|     scf.for %arg5 = %c0_i32 to %arg4 step %c1_i32  : i32 {
```
**EN:** This function-oriented block defines or enters `two_dots`. Within it, the test exercises constants, tt.func, tensor-memory allocation, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `two_dots` 为核心。测试在其中演示 常量、tt.func、张量内存分配、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 965-967
```mlir
965|       // CHECK: tt.load {{.*}} {tt.latency = 2 : i32}
966|       %2 = tt.load %arg0 : tensor<128x128x!tt.ptr<f16>, #blocked>
967|       %3 = ttg.local_alloc %2 : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 968-972
```mlir
968|       // CHECK: tt.load {{.*}} {tt.latency = 2 : i32}
969|       %4 = tt.load %arg1 : tensor<128x128x!tt.ptr<f16>, #blocked>
970|       %5 = ttg.local_alloc %4 : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
971|       %6 = tt.load %arg2 : tensor<128x128x!tt.ptr<f32>, #blocked>
972|       ttng.tmem_store %6, %0, %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized loads, shared/local memory allocation, ttng.tmem_store. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的加载、共享/本地内存分配、ttng.tmem_store。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 973-976
```mlir
973|       // CHECK: ttng.tc_gen5_mma {{.*}} {tt.latency = 1 : i32, tt.self_latency = 1 : i32}
974|       ttng.tc_gen5_mma %3, %5, %0, %true, %true : !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
975|       %7 = ttng.tmem_load %0 : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
976|       ttng.tmem_store %7, %1, %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on Gen5 tensor-core MMA ops, ttng.tmem_load, ttng.tmem_store. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 Gen5 张量核 MMA 操作、ttng.tmem_load、ttng.tmem_store。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 977-985
```mlir
977|       // CHECK: ttng.tc_gen5_mma {{.*}} {tt.latency = 1 : i32, tt.self_latency = 1 : i32}
978|       ttng.tc_gen5_mma %3, %5, %1, %true, %true : !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
979|       %8 = ttng.tmem_load %1 : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
980|       tt.store %arg3, %8 : tensor<128x128x!tt.ptr<f32>, #blocked>
981|     }
982|     tt.return
983|   }
984| }
985| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on Gen5 tensor-core MMA ops, ttng.tmem_load, masked or vectorized stores, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 Gen5 张量核 MMA 操作、ttng.tmem_load、带掩码或向量化的存储、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 986-986
```mlir
986| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 987-992
```mlir
987| 
988| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
989| #blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
990| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
991| #smem = #ttg.shared_memory
992| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 993-993
```mlir
993| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 994-994
```mlir
994|   // CHECK-LABEL: @changed_acc_before_mma
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @changed_acc_before_mma anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @changed_acc_before_mma 这样的标签用于锚定匹配范围。

### Lines 995-1010
```mlir
 995|   tt.func public @changed_acc_before_mma(%arg0: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>}, %arg1: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>}, %arg2: i32) -> tensor<128x128xf16, #blocked1> {
 996|     %true = arith.constant true
 997|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked1>
 998|     %cst_0 = arith.constant dense<2.000000e+00> : tensor<128x128xf32, #blocked1>
 999|     %c0_i32 = arith.constant 0 : i32
1000|     %c1_i32 = arith.constant 1 : i32
1001|     %0, %acc_tok = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
1002|     %init_tok = ttng.tmem_store %cst, %0[%acc_tok], %true : tensor<128x128xf32, #blocked1> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1003|     %last_tok = scf.for %arg3 = %c0_i32 to %arg2 step %c1_i32 iter_args(%tok = %init_tok) -> !ttg.async.token : i32 {
1004|       %3 = tt.load %arg0 : tensor<128x128x!tt.ptr<f16>, #blocked>
1005|       %4 = ttg.local_alloc %3 : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
1006|       %5 = tt.load %arg1 : tensor<128x128x!tt.ptr<f16>, #blocked>
1007|       %6 = ttg.local_alloc %5 : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
1008|       %7, %load_tok = ttng.tmem_load %0[%tok] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
1009|       %8 = arith.mulf %7, %cst_0 : tensor<128x128xf32, #blocked1>
1010|       %store_tok = ttng.tmem_store %8, %0[%load_tok], %true : tensor<128x128xf32, #blocked1> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** This function-oriented block defines or enters `changed_acc_before_mma`. Within it, the test exercises constants, tt.func, ttng.tmem_store, masked or vectorized loads, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `changed_acc_before_mma` 为核心。测试在其中演示 常量、tt.func、ttng.tmem_store、带掩码或向量化的加载、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1011-1020
```mlir
1011|       // CHECK: ttng.tc_gen5_mma {{.*}} {tt.latency = 1 : i32, tt.self_latency = 1 : i32}
1012|       %mma_tok = ttng.tc_gen5_mma %4, %6, %0[%store_tok], %true, %true : !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1013|       scf.yield %mma_tok : !ttg.async.token
1014|     } {tt.scheduled_max_stage = 2 : i32}
1015|     %1, %res_tok = ttng.tmem_load %0[%last_tok] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
1016|     %2 = arith.truncf %1 : tensor<128x128xf32, #blocked1> to tensor<128x128xf16, #blocked1>
1017|     tt.return %2 : tensor<128x128xf16, #blocked1>
1018|   }
1019| }
1020| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining Gen5 tensor-core MMA ops, loop/if yielded values, ttng.tmem_load, arith.truncf, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 Gen5 张量核 MMA 操作、循环/分支产出值、ttng.tmem_load、arith.truncf、tt.return。

### Lines 1021-1021
```mlir
1021| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1022-1030
```mlir
1022| 
1023| #blocked = #ttg.blocked<{sizePerThread = [1, 64], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
1024| #load_blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [2, 2], order = [1, 0]}>
1025| 
1026| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
1027| #shared_T = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 16}>
1028| 
1029| #smem = #ttg.shared_memory
1030| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 64, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1031-1032
```mlir
1031| module attributes {"ttg.num-warps" = 4 : i32, ttg.target = "cuda:100"} {
1032| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 1033-1033
```mlir
1033| // CHECK-LABEL: @attention_forward
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @attention_forward anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @attention_forward 这样的标签用于锚定匹配范围。

### Lines 1034-1051
```mlir
1034| tt.func public @attention_forward(
1035|   %Q_shared: !ttg.memdesc<256x64xf16, #shared, #smem>,
1036|   %K_desc: !tt.tensordesc<64x64xf16, #shared>,
1037|   %V_desc: !tt.tensordesc<64x64xf16, #shared>,
1038|   %qk_scale: f32,
1039|   %n_tiles: i32
1040| ) {
1041|   %true = arith.constant true
1042|   %false = arith.constant false
1043|   %c0_i32 = arith.constant 0 : i32
1044|   %c64_i32 = arith.constant 64 : i32
1045| 
1046|   %neg_inf = arith.constant dense<0xFF800000> : tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
1047|   %zero = arith.constant dense<0.0> : tensor<256x64xf32, #blocked>
1048|   %one = arith.constant dense<1.0> : tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
1049| 
1050|   %QK_tmem, %QK_tok = ttng.tmem_alloc : () -> (!ttg.memdesc<256x64xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
1051| 
```
**EN:** This function-oriented block defines or enters `attention_forward`. Within it, the test exercises constants, tt.func, tensor-memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `attention_forward` 为核心。测试在其中演示 常量、tt.func、张量内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1052-1060
```mlir
1052|   %loop_outs:3 = scf.for %i = %c0_i32 to %n_tiles step %c64_i32 iter_args(
1053|     %l_i = %one,
1054|     %acc = %zero,
1055|     %m_i = %neg_inf
1056|   ) -> (
1057|     tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>,
1058|     tensor<256x64xf32, #blocked>,
1059|     tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
1060|   ) : i32 {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 1061-1064
```mlir
1061|     // CHECK: descriptor_load {{.*}} {tt.latency = 2 : i32}
1062|     %K = tt.descriptor_load %K_desc[%i, %c0_i32] : !tt.tensordesc<64x64xf16, #shared> -> tensor<64x64xf16, #load_blocked>
1063|     %K_shared = ttg.local_alloc %K : (tensor<64x64xf16, #load_blocked>) -> !ttg.memdesc<64x64xf16, #shared, #smem>
1064|     %K_trans = ttg.memdesc_trans %K_shared {order = array<i32: 1, 0>} : !ttg.memdesc<64x64xf16, #shared, #smem> -> !ttg.memdesc<64x64xf16, #shared_T, #smem>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.descriptor_load, shared/local memory allocation, ttg.memdesc_trans. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.descriptor_load、共享/本地内存分配、ttg.memdesc_trans。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1065-1072
```mlir
1065|     // CHECK: tc_gen5_mma {{.*}} {tt.latency = 2 : i32, tt.self_latency = 0 : i32}
1066|     %QK_mma_tok = ttng.tc_gen5_mma %Q_shared, %K_trans, %QK_tmem[%QK_tok], %false, %true : !ttg.memdesc<256x64xf16, #shared, #smem>, !ttg.memdesc<64x64xf16, #shared_T, #smem>, !ttg.memdesc<256x64xf32, #tmem, #ttng.tensor_memory, mutable>
1067|     %QK, %QK_load_tok = ttng.tmem_load %QK_tmem[%QK_mma_tok] : !ttg.memdesc<256x64xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<256x64xf32, #blocked>
1068| 
1069|     %alpha_1, %P, %next_l_i, %row_max = "softmax_work"(%QK, %l_i, %m_i, %qk_scale) : (tensor<256x64xf32, #blocked>, tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>, tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>, f32) -> (tensor<256x64xf32, #blocked>, tensor<256x64xf16, #blocked>, tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>, tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>)
1070| 
1071|     %acc_corrected = arith.mulf %acc, %alpha_1 : tensor<256x64xf32, #blocked>
1072| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on Gen5 tensor-core MMA ops, ttng.tmem_load, arith.mulf. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 Gen5 张量核 MMA 操作、ttng.tmem_load、arith.mulf。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1073-1077
```mlir
1073|     // CHECK: descriptor_load {{.*}} {tt.latency = 2 : i32}
1074|     %V = tt.descriptor_load %V_desc[%i, %c0_i32] : !tt.tensordesc<64x64xf16, #shared> -> tensor<64x64xf16, #load_blocked>
1075|     %V_shared = ttg.local_alloc %V : (tensor<64x64xf16, #load_blocked>) -> !ttg.memdesc<64x64xf16, #shared, #smem>
1076|     %P_tmem = ttng.tmem_alloc %P : (tensor<256x64xf16, #blocked>) -> !ttg.memdesc<256x64xf16, #tmem, #ttng.tensor_memory>
1077|     %acc_tmem, %acc_tok = ttng.tmem_alloc %acc_corrected : (tensor<256x64xf32, #blocked>) -> (!ttg.memdesc<256x64xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor-memory allocation, tt.descriptor_load, shared/local memory allocation. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量内存分配、tt.descriptor_load、共享/本地内存分配。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1078-1091
```mlir
1078|     // CHECK: tc_gen5_mma {{.*}} {tt.self_latency = 0 : i32}
1079|     %PV_mma_tok = ttng.tc_gen5_mma %P_tmem, %V_shared, %acc_tmem[%acc_tok], %true, %true : !ttg.memdesc<256x64xf16, #tmem, #ttng.tensor_memory>, !ttg.memdesc<64x64xf16, #shared, #smem>, !ttg.memdesc<256x64xf32, #tmem, #ttng.tensor_memory, mutable>
1080|     %O, %O_tok = ttng.tmem_load %acc_tmem[%PV_mma_tok] : !ttg.memdesc<256x64xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<256x64xf32, #blocked>
1081| 
1082|     scf.yield %next_l_i, %O, %row_max : tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>, tensor<256x64xf32, #blocked>, tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
1083|   } {tt.warp_specialize}
1084| 
1085|   "use"(%loop_outs#0, %loop_outs#1, %loop_outs#2) : (tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>, tensor<256x64xf32, #blocked>, tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>) -> ()
1086| 
1087|   tt.return
1088| }
1089| 
1090| }
1091| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining Gen5 tensor-core MMA ops, ttng.tmem_load, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 Gen5 张量核 MMA 操作、ttng.tmem_load、循环/分支产出值、tt.return。

### Lines 1092-1092
```mlir
1092| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1093-1100
```mlir
1093| 
1094| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
1095| #blocked1 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
1096| #blocked2 = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
1097| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 8}>
1098| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 8}>
1099| #smem = #ttg.shared_memory
1100| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1101-1101
```mlir
1101| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 1102-1119
```mlir
1102|   tt.func public @attention_persistent_inner_loop_kernel(%desc_q: !tt.tensordesc<128x128xf16, #shared>, %desc_q_0: i32, %desc_q_1: i32, %desc_q_2: i64, %desc_q_3: i64, %desc_k: !tt.tensordesc<128x128xf16, #shared>, %desc_k_4: i32, %desc_k_5: i32, %desc_k_6: i64, %desc_k_7: i64, %desc_v: !tt.tensordesc<128x128xf16, #shared>, %desc_v_8: i32, %desc_v_9: i32, %desc_v_10: i64, %desc_v_11: i64, %desc_acc: !tt.tensordesc<128x128xf16, #shared>, %desc_acc_12: i32, %desc_acc_13: i32, %desc_acc_14: i64, %desc_acc_15: i64, %l_i_ptr: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %m_i_ptr: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %M: i32 {tt.divisibility = 16 : i32}, %N: i32 {tt.divisibility = 16 : i32}, %qk_scale: f32) attributes {noinline = false} {
1103|     %false = arith.constant false
1104|     %true = arith.constant true
1105|     %c1_i32 = arith.constant 1 : i32
1106|     %c0_i32 = arith.constant 0 : i32
1107|     %c128_i32 = arith.constant 128 : i32
1108|     %cst = arith.constant dense<1.000000e+00> : tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
1109|     %cst_16 = arith.constant dense<0xFF800000> : tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
1110|     %prog_id = tt.get_program_id x : i32
1111|     %num_sm = tt.get_num_programs x : i32
1112|     %num_tiles = arith.divsi %M, %c128_i32 : i32
1113|     %tiles_per_sm = arith.divsi %num_tiles, %num_sm : i32
1114|     %tile_idx = scf.for %_ = %c0_i32 to %tiles_per_sm step %c1_i32 iter_args(%tile_idx_20 = %prog_id) -> (i32)  : i32 {
1115|       %off_m = arith.muli %tile_idx_20, %c128_i32 : i32
1116|       %q = tt.descriptor_load %desc_q[%off_m, %c0_i32] : !tt.tensordesc<128x128xf16, #shared> -> tensor<128x128xf16, #blocked2>
1117|       %q_21 = ttg.local_alloc %q : (tensor<128x128xf16, #blocked2>) -> !ttg.memdesc<128x128xf16, #shared, #smem>
1118|       %qk_22, %qk_23 = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
1119|       %acc, %acc_24 = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
```
**EN:** This function-oriented block defines or enters `attention_persistent_inner_loop_kernel`. Within it, the test exercises constants, tt.func, arith.divsi, tensor-memory allocation, program IDs, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `attention_persistent_inner_loop_kernel` 为核心。测试在其中演示 常量、tt.func、arith.divsi、张量内存分配、程序 ID，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1120-1120
```mlir
1120|       %acc_26:4 = scf.for %acc_30 = %c0_i32 to %N step %c128_i32 iter_args(%arg28 = %cst_16, %arg29 = %cst, %qk_31 = %qk_23, %acc_32 = %acc_24) -> (tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>, tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>, !ttg.async.token, !ttg.async.token)  : i32 {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 1121-1124
```mlir
1121|         // CHECK: tt.descriptor_load {{.*}} {tt.latency = 2 : i32}
1122|         %k = tt.descriptor_load %desc_k[%acc_30, %c0_i32] : !tt.tensordesc<128x128xf16, #shared> -> tensor<128x128xf16, #blocked2>
1123|         %k_33 = ttg.local_alloc %k : (tensor<128x128xf16, #blocked2>) -> !ttg.memdesc<128x128xf16, #shared, #smem>
1124|         %k_34 = ttg.memdesc_trans %k_33 {order = array<i32: 1, 0>} : !ttg.memdesc<128x128xf16, #shared, #smem> -> !ttg.memdesc<128x128xf16, #shared1, #smem>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.descriptor_load, shared/local memory allocation, ttg.memdesc_trans. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.descriptor_load、共享/本地内存分配、ttg.memdesc_trans。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1125-1134
```mlir
1125|         // CHECK: ttng.tc_gen5_mma {{.*}}, {{.*}}, {{.*}}, {{.*}}, {{.*}} {tt.latency = 2 : i32, tt.self_latency = 0 : i32}
1126|         %qk_35 = ttng.tc_gen5_mma %q_21, %k_34, %qk_22[%qk_31], %false, %true : !ttg.memdesc<128x128xf16, #shared, #smem>, !ttg.memdesc<128x128xf16, #shared1, #smem>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1127|         %qk_36, %qk_37 = ttng.tmem_load %qk_22[%qk_35] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
1128| 
1129|         %acc_47, %p, %next_l_i, %row_max = "softmax_work"(%qk_36, %arg29, %arg28) : (tensor<128x128xf32, #blocked>, tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>, tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>) -> (tensor<128x128xf32, #blocked>, tensor<128x128xf16, #blocked>, tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>, tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>)
1130| 
1131|         %acc_48, %acc_49 = ttng.tmem_load %acc[%acc_32] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
1132|         %acc_50 = arith.mulf %acc_48, %acc_47 : tensor<128x128xf32, #blocked>
1133|         %p_53 = ttg.local_alloc %p : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem>
1134|         %acc_54 = ttng.tmem_store %acc_50, %acc[%acc_49], %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_load, Gen5 tensor-core MMA ops, arith.mulf, shared/local memory allocation, ttng.tmem_store. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_load、Gen5 张量核 MMA 操作、arith.mulf、共享/本地内存分配、ttng.tmem_store。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1135-1138
```mlir
1135|         // CHECK: tt.descriptor_load {{.*}} {tt.latency = 2 : i32}
1136|         %v = tt.descriptor_load %desc_v[%acc_30, %c0_i32] : !tt.tensordesc<128x128xf16, #shared> -> tensor<128x128xf16, #blocked2>
1137|         %v_51 = ttg.local_alloc %v : (tensor<128x128xf16, #blocked2>) -> !ttg.memdesc<128x128xf16, #shared, #smem>
1138| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 1139-1150
```mlir
1139|         // CHECK: ttng.tc_gen5_mma {{.*}}, {{.*}}, {{.*}}, {{.*}}, {{.*}} {tt.self_latency = 0 : i32}
1140|         %acc_55 = ttng.tc_gen5_mma %p_53, %v_51, %acc[%acc_54], %true, %true : !ttg.memdesc<128x128xf16, #shared, #smem>, !ttg.memdesc<128x128xf16, #shared, #smem>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1141| 
1142|         scf.yield %row_max, %next_l_i, %qk_37, %acc_55 : tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>, tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>, !ttg.async.token, !ttg.async.token
1143|       }
1144|       %tile_idx_29 = arith.addi %tile_idx_20, %num_sm : i32
1145|       scf.yield %tile_idx_29 : i32
1146|     } {tt.num_stages = 3 : i32, tt.warp_specialize}
1147|     tt.return
1148|   }
1149| }
1150| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, Gen5 tensor-core MMA ops, integer additions, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、Gen5 张量核 MMA 操作、整数加法、tt.return。

### Lines 1151-1151
```mlir
1151| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1152-1154
```mlir
1152| 
1153| // Test that ub.poison producing a memdesc does not get treated like a tensor
1154| // value in AxisInfo analysis.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1155-1156
```mlir
1155| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
1156| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1157-1157
```mlir
1157| module attributes {"ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 1158-1168
```mlir
1158|   tt.func public @minimal_crash(%lb: i32, %ub: i32) -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable> {
1159|     %c1 = arith.constant 1 : i32
1160|     %poison = ub.poison : !ttg.memdesc<64x64xf16, #shared, #smem, mutable>
1161|     %normal = ttg.local_alloc : () -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable>
1162|     %result = scf.for %i = %lb to %ub step %c1 iter_args(%current = %poison) -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable> : i32 {
1163|       scf.yield %normal : !ttg.memdesc<64x64xf16, #shared, #smem, mutable>
1164|     }
1165|     tt.return %result : !ttg.memdesc<64x64xf16, #shared, #smem, mutable>
1166|   }
1167| }
1168| 
```
**EN:** This function-oriented block defines or enters `minimal_crash`. Within it, the test exercises tt.func, constants, ub.poison, shared/local memory allocation, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `minimal_crash` 为核心。测试在其中演示 tt.func、常量、ub.poison、共享/本地内存分配、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1169-1169
```mlir
1169| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1170-1175
```mlir
1170| 
1171| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
1172| #blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
1173| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
1174| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
1175| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1176-1176
```mlir
1176| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 1177-1177
```mlir
1177| // CHECK-LABEL: @tc_gen5_mma_alloc_block_arg
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tc_gen5_mma_alloc_block_arg anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tc_gen5_mma_alloc_block_arg 这样的标签用于锚定匹配范围。

### Lines 1178-1184
```mlir
1178| tt.func @tc_gen5_mma_alloc_block_arg(%lb : index, %ub : index, %step : index,
1179|                   %A_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
1180|                   %B_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
1181|                   %acc_init : tensor<128x128xf32, #blocked1>) -> () {
1182|   %true = arith.constant true
1183|   %acc_tm = ttng.tmem_alloc %acc_init : (tensor<128x128xf32, #blocked1>) -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1184|   %zero = arith.constant dense<0.0> : tensor<128x128xf16, #blocked1>
```
**EN:** This function-oriented block defines or enters `tc_gen5_mma_alloc_block_arg`. Within it, the test exercises tt.func, constants, tensor-memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tc_gen5_mma_alloc_block_arg` 为核心。测试在其中演示 tt.func、常量、张量内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1185-1191
```mlir
1185|   // CHECK: ttng.tmem_alloc
1186|   // CHECK: scf.for
1187|   scf.for %iv = %lb to %ub step %step iter_args(%A = %zero, %B = %zero) -> (tensor<128x128xf16, #blocked1>, tensor<128x128xf16, #blocked1>) : index {
1188|     // Ensure this doesn't crash.
1189|     %A_sh = ttg.local_alloc %A : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
1190|     %B_sh = ttg.local_alloc %B : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
1191|     ttng.tmem_store %acc_init, %acc_tm, %true : tensor<128x128xf32, #blocked1> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining shared/local memory allocation, structured loops, ttng.tmem_store.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 共享/本地内存分配、结构化循环、ttng.tmem_store。

### Lines 1192-1193
```mlir
1192|     // CHECK: ttng.tc_gen5_mma
1193|     ttng.tc_gen5_mma %A_sh, %B_sh, %acc_tm, %true, %true : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on Gen5 tensor-core MMA ops. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 Gen5 张量核 MMA 操作。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1194-1203
```mlir
1194|     // CHECK: ttng.tmem_load
1195|     %acc_res = ttng.tmem_load %acc_tm : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
1196|     "use"(%acc_res) : (tensor<128x128xf32, #blocked1>) -> ()
1197|     %A_next = tt.load %A_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
1198|     %B_next = tt.load %B_ptr : tensor<128x128x!tt.ptr<f16>, #blocked1>
1199|     scf.yield %A_next, %B_next : tensor<128x128xf16, #blocked1>, tensor<128x128xf16, #blocked1>
1200|   }
1201|   tt.return
1202| }
1203| }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining masked or vectorized loads, ttng.tmem_load, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 带掩码或向量化的加载、ttng.tmem_load、循环/分支产出值、tt.return。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-allow-unregistered-dialect`, `-tritongpu-assign-latencies=num-stages=3`, `-canonicalize`
- **CN:** 主要 pass 选项：`-split-input-file`，`-allow-unregistered-dialect`，`-tritongpu-assign-latencies=num-stages=3`，`-canonicalize`
- **EN:** Dominant operations include `arith.constant`, `tt.load`, `tt.func`, `ttg.local_alloc`, `tt.addptr`, `scf.for`, `tt.return`, `ttng.tmem_alloc`, `ttng.tmem_load`, `module`.
- **CN:** 主要操作包括 `arith.constant`、`tt.load`、`tt.func`、`ttg.local_alloc`、`tt.addptr`、`scf.for`、`tt.return`、`ttng.tmem_alloc`、`ttng.tmem_load`、`module`。
- **EN:** The file contains 22 independently testable section(s). Check styles used: CHECK x85, CHECK-LABEL x33, CHECK-NOT x14. Important labels include @default_stages, @small_load, @load_into_shared, @load_into_lt_4b. It validates both the presence of the desired rewrite and the absence of unwanted residual operations.
- **CN:** 该文件包含 22 个可独立测试的分段。使用的检查类型：CHECK ×85，CHECK-LABEL ×33，CHECK-NOT ×14。 关键标签包括 @default_stages，@small_load，@load_into_shared，@load_into_lt_4b。 它同时验证目标重写已经出现，并且不再残留不需要的操作。

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