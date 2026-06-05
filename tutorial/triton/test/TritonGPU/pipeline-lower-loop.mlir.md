# pipeline-lower-loop.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/pipeline-lower-loop.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-tritongpu-test-pipeline-lower-loop, -canonicalize` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-tritongpu-test-pipeline-lower-loop, -canonicalize` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-test-pipeline-lower-loop -canonicalize | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-test-pipeline-lower-loop -canonicalize | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-1
```mlir
1| // RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-test-pipeline-lower-loop -canonicalize | FileCheck %s
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-test-pipeline-lower-loop -canonicalize | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-test-pipeline-lower-loop -canonicalize | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 2-5
```mlir
2| #nvmma_128 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
3| 
4| #A = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
5| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 6-6
```mlir
6| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 7-10
```mlir
 7| // CHECK-LABEL: @unscheduled_loop
 8| // CHECK: scf.for
 9| // CHECK:   tt.load
10| // CHECK:   "use"
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @unscheduled_loop anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @unscheduled_loop 这样的标签用于锚定匹配范围。

### Lines 11-20
```mlir
11| tt.func @unscheduled_loop(%lb : index, %ub : index, %step : index,
12|                  %a_ptr_init : tensor<128x32x!tt.ptr<f16>, #A>) -> () {
13|   scf.for %iv = %lb to %ub step %step : index {
14|     %a = tt.load %a_ptr_init : tensor<128x32x!tt.ptr<f16>, #A>
15|     "use"(%a) : (tensor<128x32xf16, #A>) -> ()
16|   }
17|   tt.return
18| }
19| }
20| 
```
**EN:** This function-oriented block defines or enters `unscheduled_loop`. Within it, the test exercises tt.func, structured loops, masked or vectorized loads, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `unscheduled_loop` 为核心。测试在其中演示 tt.func、结构化循环、带掩码或向量化的加载、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 21-21
```mlir
21| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 22-24
```mlir
22| 
23| #A = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
24| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 25-25
```mlir
25| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 26-43
```mlir
26| // CHECK-LABEL: @one_dep_async
27| // CHECK-DAG: %[[MINUS_ONE:.*]] = arith.constant -1
28| // CHECK-DAG: %[[ZERO:.*]] = arith.constant 0
29| // CHECK-DAG: %[[ONE:.*]] = arith.constant 1
30| // CHECK-DAG: %[[A:.*]] = ttg.local_alloc : () -> !ttg.memdesc<2x128x32
31| // CHECK-DAG: %[[NUM_BUFS:.*]] = arith.constant {{.*}} 2 : i32
32| // CHECK: scf.for {{.*}} iter_args(%[[INS:.*]] = %[[MINUS_ONE]], %[[EXT:.*]] = %[[MINUS_ONE]])
33| // CHECK:   %[[INS_P1:.*]] = arith.addi %[[INS]], %[[ONE]] {loop.cluster = 2 : i32, loop.stage = 0 : i32}  : i32
34| // CHECK:   %[[INS_CMP:.*]] = arith.cmpi sge, %[[INS_P1]], %[[NUM_BUFS]] {loop.cluster = 2 : i32, loop.stage = 0 : i32}  : i32
35| // CHECK:   %[[INS_NEXT:.*]] = arith.select %[[INS_CMP]], %[[ZERO]], %[[INS_P1]] {loop.cluster = 2 : i32, loop.stage = 0 : i32}  : i32
36| // CHECK:   %[[EXT_P1:.*]] = arith.addi %[[EXT]], %[[ONE]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}  : i32
37| // CHECK:   %[[EXT_CMP:.*]] = arith.cmpi sge, %[[EXT_P1]], %[[NUM_BUFS]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}  : i32
38| // CHECK:   %[[EXT_NEXT:.*]] = arith.select %[[EXT_CMP]], %[[ZERO]], %[[EXT_P1]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}  : i32
39| // CHECK:   %[[A_INS:.*]] = ttg.memdesc_index %[[A]]{{\[}}%[[INS_NEXT]]{{\]}} {loop.cluster = 2 : i32, loop.stage = 0 : i32}
40| // CHECK:   %[[A_TOK:.*]] = ttg.async_copy_global_to_local %{{.*}}, %[[A_INS]] {contiguity = 8 : i32, loop.cluster = 2 : i32, loop.stage = 0 : i32}
41| // CHECK:   %[[A_TOK2:.*]] = ttg.async_commit_group tokens %[[A_TOK]] {loop.cluster = 2 : i32, loop.stage = 0 : i32}
42| // CHECK:   %[[A_TOK3:.*]] = ttg.async_wait %[[A_TOK2]] {loop.cluster = 0 : i32, loop.stage = 2 : i32, num = 0 : i32}
43| // CHECK:   %[[A_EXT:.*]] = ttg.memdesc_index %[[A]]{{\[}}%[[EXT_NEXT]]{{\]}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @one_dep_async anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @one_dep_async 这样的标签用于锚定匹配范围。

### Lines 44-49
```mlir
44| // CHECK:   %[[A_VAL:.*]] = ttg.local_load %[[A_EXT]] token %[[A_TOK3]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
45| // CHECK:   "use"(%[[A_VAL]]) {loop.cluster = 0 : i32, loop.stage = 2 : i32}
46| // CHECK:   scf.yield %[[INS_NEXT]], %[[EXT_NEXT]]
47| // CHECK-DAG:   ttg.local_dealloc %[[A]]
48| // CHECK-DAG:   ttg.async_wait  {num = 0 : i32}
49| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 50-59
```mlir
50| tt.func @one_dep_async(%lb : index, %ub : index, %step : index,
51|                  %a_ptr_init : tensor<128x32x!tt.ptr<f16>, #A> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>}) -> () {
52|   scf.for %iv = %lb to %ub step %step : index {
53|     %a = tt.load %a_ptr_init {loop.cluster = 2 : i32, loop.stage = 0 : i32} : tensor<128x32x!tt.ptr<f16>, #A>
54|     "use"(%a) {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x32xf16, #A>) -> ()
55|   } {tt.scheduled_max_stage = 2 : i32}
56|   tt.return
57| }
58| }
59| 
```
**EN:** This function-oriented block defines or enters `one_dep_async`. Within it, the test exercises tt.func, structured loops, masked or vectorized loads, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `one_dep_async` 为核心。测试在其中演示 tt.func、结构化循环、带掩码或向量化的加载、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 60-60
```mlir
60| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 61-65
```mlir
61| 
62| #A = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
63| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
64| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
65| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 66-66
```mlir
66| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 67-68
```mlir
67| // CHECK-LABEL: @one_dep_barrier_wait
68| // CHECK: ttg.local_alloc : () -> !ttg.memdesc<3x128x64
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @one_dep_barrier_wait anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @one_dep_barrier_wait 这样的标签用于锚定匹配范围。

### Lines 69-82
```mlir
69| tt.func @one_dep_barrier_wait(%lb : index, %ub : index, %step : index,
70|                  %a_ptr_init : tensor<128x64x!tt.ptr<f16>, #A> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
71|                  %bar : !ttg.memdesc<1xi64, #shared1, #ttg.shared_memory, mutable>,
72|                  %phase : i32) -> () {
73|   scf.for %iv = %lb to %ub step %step : index {
74|     %a = tt.load %a_ptr_init {loop.cluster = 2 : i32, loop.stage = 0 : i32} : tensor<128x64x!tt.ptr<f16>, #A>
75|     %sh = ttg.local_alloc %a {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x64xf16, #A>) -> !ttg.memdesc<128x64xf16, #shared, #ttg.shared_memory>
76|     "use"(%a) {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x64xf16, #A>) -> ()
77|     ttng.wait_barrier %bar, %phase deps %sh {loop.cluster = 3 : i32, loop.stage = 3 : i32} : !ttg.memdesc<1xi64, #shared1, #ttg.shared_memory, mutable>, !ttg.memdesc<128x64xf16, #shared, #ttg.shared_memory>
78|   } {tt.scheduled_max_stage = 3 : i32}
79|   tt.return
80| }
81| }
82| 
```
**EN:** This function-oriented block defines or enters `one_dep_barrier_wait`. Within it, the test exercises tt.func, structured loops, masked or vectorized loads, shared/local memory allocation, ttng.wait_barrier, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `one_dep_barrier_wait` 为核心。测试在其中演示 tt.func、结构化循环、带掩码或向量化的加载、共享/本地内存分配、ttng.wait_barrier，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 83-83
```mlir
83| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 84-89
```mlir
84| 
85| #A = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
86| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
87| #shared2 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 16}>
88| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
89| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 90-90
```mlir
90| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 91-92
```mlir
91| // CHECK-LABEL: @one_dep_barrier_wait_trans
92| // CHECK: ttg.local_alloc : () -> !ttg.memdesc<3x128x64
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @one_dep_barrier_wait_trans anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @one_dep_barrier_wait_trans 这样的标签用于锚定匹配范围。

### Lines 93-107
```mlir
 93| tt.func @one_dep_barrier_wait_trans(%lb : index, %ub : index, %step : index,
 94|                  %a_ptr_init : tensor<128x64x!tt.ptr<f16>, #A> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
 95|                  %bar : !ttg.memdesc<1xi64, #shared1, #ttg.shared_memory, mutable>,
 96|                  %phase : i32) -> () {
 97|   scf.for %iv = %lb to %ub step %step : index {
 98|     %a = tt.load %a_ptr_init {loop.cluster = 2 : i32, loop.stage = 0 : i32} : tensor<128x64x!tt.ptr<f16>, #A>
 99|     %sh = ttg.local_alloc %a {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x64xf16, #A>) -> !ttg.memdesc<128x64xf16, #shared, #ttg.shared_memory>
100|     %trans = ttg.memdesc_trans %sh {order = array<i32: 1, 0>, loop.cluster = 0 : i32, loop.stage = 3 : i32} : !ttg.memdesc<128x64xf16, #shared, #ttg.shared_memory> -> !ttg.memdesc<64x128xf16, #shared2, #ttg.shared_memory>
101|     "use"(%a) {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x64xf16, #A>) -> ()
102|     ttng.wait_barrier %bar, %phase deps %trans {loop.cluster = 3 : i32, loop.stage = 3 : i32} : !ttg.memdesc<1xi64, #shared1, #ttg.shared_memory, mutable>, !ttg.memdesc<64x128xf16, #shared2, #ttg.shared_memory>
103|   } {tt.scheduled_max_stage = 3 : i32}
104|   tt.return
105| }
106| }
107| 
```
**EN:** This function-oriented block defines or enters `one_dep_barrier_wait_trans`. Within it, the test exercises tt.func, structured loops, masked or vectorized loads, shared/local memory allocation, ttg.memdesc_trans, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `one_dep_barrier_wait_trans` 为核心。测试在其中演示 tt.func、结构化循环、带掩码或向量化的加载、共享/本地内存分配、ttg.memdesc_trans，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 108-108
```mlir
108| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 109-111
```mlir
109| 
110| #A = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
111| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 112-112
```mlir
112| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 113-120
```mlir
113| // CHECK-LABEL: @different_use_stages
114| // CHECK: scf.for
115| // CHECK:   ttg.async_copy_global_to_local %{{.*}} {contiguity = 8 : i32, loop.cluster = 2 : i32, loop.stage = 0 : i32}
116| // CHECK:   ttg.async_wait {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32, num = 0 : i32}
117| // CHECK:   ttg.memdesc_index {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
118| // CHECK:   %[[A_VAL:.*]] = ttg.local_load {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
119| // CHECK:   "use1"(%[[A_VAL]]) {loop.cluster = 0 : i32, loop.stage = 2 : i32}
120| // CHECK:   "use2"(%[[A_VAL]]) {loop.cluster = 0 : i32, loop.stage = 3 : i32}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @different_use_stages anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @different_use_stages 这样的标签用于锚定匹配范围。

### Lines 121-131
```mlir
121| tt.func @different_use_stages(%lb : index, %ub : index, %step : index,
122|                  %a_ptr_init : tensor<128x32x!tt.ptr<f16>, #A> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>}) -> () {
123|   scf.for %iv = %lb to %ub step %step : index {
124|     %a = tt.load %a_ptr_init {loop.cluster = 2 : i32, loop.stage = 0 : i32} : tensor<128x32x!tt.ptr<f16>, #A>
125|     "use1"(%a) {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x32xf16, #A>) -> ()
126|     "use2"(%a) {loop.cluster = 0 : i32, loop.stage = 3 : i32} : (tensor<128x32xf16, #A>) -> ()
127|   } {tt.scheduled_max_stage = 3 : i32}
128|   tt.return
129| }
130| }
131| 
```
**EN:** This function-oriented block defines or enters `different_use_stages`. Within it, the test exercises tt.func, structured loops, masked or vectorized loads, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `different_use_stages` 为核心。测试在其中演示 tt.func、结构化循环、带掩码或向量化的加载、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 132-132
```mlir
132| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 133-135
```mlir
133| 
134| #A = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
135| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 136-136
```mlir
136| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 137-145
```mlir
137| // CHECK-LABEL: @used_by_if_yield
138| // CHECK-DAG: %[[A:.*]] = ttg.local_alloc : () -> !ttg.memdesc<2x128x32
139| // CHECK: scf.for
140| // CHECK:   %[[A_TOK:.*]] = ttg.async_copy_global_to_local %{{.*}} {contiguity = 8 : i32, loop.cluster = 2 : i32, loop.stage = 0 : i32}
141| // CHECK:   %[[A_TOK2:.*]] = ttg.async_commit_group tokens %[[A_TOK]] {loop.cluster = 2 : i32, loop.stage = 0 : i32}
142| // CHECK:   %[[A_TOK3:.*]] = ttg.async_wait %[[A_TOK2]] {loop.cluster = 0 : i32, loop.stage = 2 : i32, num = 0 : i32}
143| // CHECK:   ttg.local_load {{.*}} token %[[A_TOK3]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
144| // CHECK:   "use"{{.*}} {loop.cluster = 0 : i32, loop.stage = 3 : i32}
145| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @used_by_if_yield anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @used_by_if_yield 这样的标签用于锚定匹配范围。

### Lines 146-161
```mlir
146| tt.func @used_by_if_yield(%lb : index, %ub : index, %step : index,
147|                  %a_ptr_init : tensor<128x32x!tt.ptr<f16>, #A> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
148|                  %init_a : tensor<128x32xf16, #A>,
149|                  %cnd : i1) -> () {
150|   scf.for %iv = %lb to %ub step %step : index {
151|     %a = tt.load %a_ptr_init {loop.cluster = 2 : i32, loop.stage = 0 : i32} : tensor<128x32x!tt.ptr<f16>, #A>
152|     %a_if = scf.if %cnd -> tensor<128x32xf16, #A> {
153|       scf.yield %a : tensor<128x32xf16, #A>
154|     } else {
155|       scf.yield %init_a : tensor<128x32xf16, #A>
156|     } {loop.cluster = 0 : i32, loop.stage = 2 : i32}
157|     "use"(%a_if) {loop.cluster = 0 : i32, loop.stage = 3 : i32} : (tensor<128x32xf16, #A>) -> ()
158|   } {tt.scheduled_max_stage = 3 : i32}
159|   tt.return
160| }
161| }
```
**EN:** This function-oriented block defines or enters `used_by_if_yield`. Within it, the test exercises tt.func, loop/if yielded values, structured loops, masked or vectorized loads, structured conditionals, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `used_by_if_yield` 为核心。测试在其中演示 tt.func、循环/分支产出值、结构化循环、带掩码或向量化的加载、结构化条件分支，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 162-162
```mlir
162| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 163-165
```mlir
163| 
164| #A = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
165| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 166-166
```mlir
166| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 167-167
```mlir
167| // CHECK-LABEL: @dist1_load
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @dist1_load anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @dist1_load 这样的标签用于锚定匹配范围。

### Lines 168-180
```mlir
168| tt.func @dist1_load(%lb : index, %ub : index, %step : index,
169|                  %a_ptr_init : tensor<128x32x!tt.ptr<f16>, #A> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
170|                  %init_a : tensor<128x32xf16, #A>) -> () {
171|   %_ = scf.for %iv = %lb to %ub step %step iter_args(%prev_a = %init_a) -> (tensor<128x32xf16, #A>) : index {
172|     "use_next_iter"(%prev_a) {loop.cluster = 2 : i32, loop.stage = 0 : i32} : (tensor<128x32xf16, #A>) -> ()
173|     %a = tt.load %a_ptr_init {loop.cluster = 2 : i32, loop.stage = 0 : i32} : tensor<128x32x!tt.ptr<f16>, #A>
174|     "use"(%a) {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x32xf16, #A>) -> ()
175|     scf.yield %a : tensor<128x32xf16, #A>
176|   } {tt.scheduled_max_stage = 2 : i32}
177|   tt.return
178| }
179| }
180| 
```
**EN:** This function-oriented block defines or enters `dist1_load`. Within it, the test exercises tt.func, structured loops, masked or vectorized loads, loop/if yielded values, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `dist1_load` 为核心。测试在其中演示 tt.func、结构化循环、带掩码或向量化的加载、循环/分支产出值、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 181-181
```mlir
181| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 182-184
```mlir
182| 
183| #A = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
184| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 185-185
```mlir
185| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 186-188
```mlir
186| // CHECK-LABEL: @one_dep_sync
187| // CHECK: scf.for
188| // CHECK:   tt.load {{.*}} {loop.cluster = 2 : i32, loop.stage = 0 : i32}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @one_dep_sync anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @one_dep_sync 这样的标签用于锚定匹配范围。

### Lines 189-198
```mlir
189| tt.func @one_dep_sync(%lb : index, %ub : index, %step : index,
190|                  %a_ptr_init : tensor<1x!tt.ptr<f16>, #A> {tt.divisibility = dense<[16]> : tensor<1xi32>, tt.contiguity = dense<[16]> : tensor<1xi32>}) -> () {
191|   scf.for %iv = %lb to %ub step %step : index {
192|     %a = tt.load %a_ptr_init {loop.cluster = 2 : i32, loop.stage = 0 : i32} : tensor<1x!tt.ptr<f16>, #A>
193|     "use"(%a) {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<1xf16, #A>) -> ()
194|   } {tt.scheduled_max_stage = 2 : i32}
195|   tt.return
196| }
197| }
198| 
```
**EN:** This function-oriented block defines or enters `one_dep_sync`. Within it, the test exercises tt.func, structured loops, masked or vectorized loads, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `one_dep_sync` 为核心。测试在其中演示 tt.func、结构化循环、带掩码或向量化的加载、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 199-199
```mlir
199| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 200-203
```mlir
200| 
201| #A = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
202| #shared = #ttg.swizzled_shared<{vec = 8, perPhase = 2, maxPhase = 4, order = [1, 0]}>
203| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 204-204
```mlir
204| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 205-222
```mlir
205| // CHECK: #[[SHARED:.*]] = #ttg.swizzled_shared
206| // CHECK-DAG: %[[MINUS_ONE:.*]] = arith.constant -1
207| // CHECK-DAG: %[[ZERO:.*]] = arith.constant 0
208| // CHECK-DAG: %[[ONE:.*]] = arith.constant 1
209| // CHECK-DAG: %[[A:.*]] = ttg.local_alloc : () -> !ttg.memdesc<2x128x32
210| // CHECK-DAG: %[[NUM_BUFS:.*]] = arith.constant {{.*}} 2 : i32
211| // CHECK: scf.for {{.*}} iter_args(%[[INS:.*]] = %[[MINUS_ONE]], %[[EXT:.*]] = %[[MINUS_ONE]])
212| // CHECK:   %[[INS_P1:.*]] = arith.addi %[[INS]], %[[ONE]] {loop.cluster = 2 : i32, loop.stage = 0 : i32}  : i32
213| // CHECK:   %[[INS_CMP:.*]] = arith.cmpi sge, %[[INS_P1]], %[[NUM_BUFS]] {loop.cluster = 2 : i32, loop.stage = 0 : i32}  : i32
214| // CHECK:   %[[INS_NEXT:.*]] = arith.select %[[INS_CMP]], %[[ZERO]], %[[INS_P1]] {loop.cluster = 2 : i32, loop.stage = 0 : i32}  : i32
215| // CHECK:   %[[EXT_P1:.*]] = arith.addi %[[EXT]], %[[ONE]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}  : i32
216| // CHECK:   %[[EXT_CMP:.*]] = arith.cmpi sge, %[[EXT_P1]], %[[NUM_BUFS]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}  : i32
217| // CHECK:   %[[EXT_NEXT:.*]] = arith.select %[[EXT_CMP]], %[[ZERO]], %[[EXT_P1]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}  : i32
218| // CHECK:   %[[A_INS:.*]] = ttg.memdesc_index %[[A]]{{\[}}%[[INS_NEXT]]{{\]}} {loop.cluster = 2 : i32, loop.stage = 0 : i32}
219| // CHECK:   %[[A_TOK:.*]] = ttg.async_copy_global_to_local %{{.*}}, %[[A_INS]] {contiguity = 8 : i32, loop.cluster = 2 : i32, loop.stage = 0 : i32}
220| // CHECK:   %[[A_TOK2:.*]] = ttg.async_commit_group tokens %[[A_TOK]] {loop.cluster = 2 : i32, loop.stage = 0 : i32}
221| // CHECK:   %[[A_TOK3:.*]] = ttg.async_wait %[[A_TOK2]] {loop.cluster = 0 : i32, loop.stage = 2 : i32, num = 0 : i32}
222| // CHECK:   %[[A_EXT:.*]] = ttg.memdesc_index %[[A]]{{\[}}%[[EXT_NEXT]]{{\]}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 223-227
```mlir
223| // CHECK:   %[[A_VAL:.*]] = ttg.local_load %[[A_EXT]] {loop.cluster = 0 : i32, loop.stage = 2 : i32} : !ttg.memdesc<128x32xf16, #[[SHARED]], #
224| // CHECK:   "use"(%[[A_VAL]]) {loop.cluster = 0 : i32, loop.stage = 2 : i32}
225| // CHECK:   scf.yield %[[INS_NEXT]], %[[EXT_NEXT]]
226| // CHECK-DAG:   ttg.local_dealloc %[[A]]
227| // CHECK-DAG:   ttg.async_wait  {num = 0 : i32}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 228-239
```mlir
228| tt.func @one_dep_local_alloc(%lb : index, %ub : index, %step : index,
229|                  %a_ptr_init : tensor<128x32x!tt.ptr<f16>, #A> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>}) -> () {
230|   scf.for %iv = %lb to %ub step %step : index {
231|     %a = tt.load %a_ptr_init {loop.cluster = 2 : i32, loop.stage = 0 : i32} : tensor<128x32x!tt.ptr<f16>, #A>
232|     %a_alloc = ttg.local_alloc %a {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x32xf16, #A>) -> !ttg.memdesc<128x32xf16, #shared, #ttg.shared_memory, mutable>
233|     %a_load = ttg.local_load %a_alloc {loop.cluster = 0 : i32, loop.stage = 2 : i32} : !ttg.memdesc<128x32xf16, #shared, #ttg.shared_memory, mutable> -> tensor<128x32xf16, #A>
234|     "use"(%a_load) {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x32xf16, #A>) -> ()
235|   } {tt.scheduled_max_stage = 2 : i32}
236|   tt.return
237| }
238| }
239| 
```
**EN:** This function-oriented block defines or enters `one_dep_local_alloc`. Within it, the test exercises tt.func, structured loops, masked or vectorized loads, shared/local memory allocation, local/shared memory loads, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `one_dep_local_alloc` 为核心。测试在其中演示 tt.func、结构化循环、带掩码或向量化的加载、共享/本地内存分配、本地/共享内存加载，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 240-240
```mlir
240| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 241-243
```mlir
241| 
242| #A = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
243| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 244-244
```mlir
244| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 245-245
```mlir
245| // CHECK-LABEL: @one_load_group
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @one_load_group anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @one_load_group 这样的标签用于锚定匹配范围。

### Lines 246-248
```mlir
246| tt.func @one_load_group(%lb : index, %ub : index, %step : index,
247|                        %a_ptr_init : tensor<128x32x!tt.ptr<f32>, #A> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
248|                        %b_ptr_init : tensor<128x32x!tt.ptr<f32>, #A> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>}) -> () {
```
**EN:** This function-oriented block defines or enters `one_load_group`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `one_load_group` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 249-253
```mlir
249|   // CHECK-DAG: %[[MINUS_ONE:.*]] = arith.constant -1
250|   // CHECK-DAG: %[[ZERO:.*]] = arith.constant 0
251|   // CHECK-DAG: %[[ONE:.*]] = arith.constant 1
252|   // CHECK-DAG: %[[NUM_BUFS:.*]] = arith.constant {{.*}} 2 : i32
253|   // Only one insert and extract index is used.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 254-255
```mlir
254|   // CHECK: scf.for {{.*}} iter_args(%[[INS:.*]] = %[[MINUS_ONE]], %[[EXT:.*]] = %[[MINUS_ONE]]) ->
255|   scf.for %iv = %lb to %ub step %step : index {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 256-270
```mlir
256|     // CHECK: %[[INS_P1:.*]] = arith.addi %[[INS]], %[[ONE]]
257|     // CHECK: %[[INS_CMP:.*]] = arith.cmpi sge, %[[INS_P1]], %[[NUM_BUFS]]
258|     // CHECK: %[[INS_NEXT:.*]] = arith.select %[[INS_CMP]], %[[ZERO]], %[[INS_P1]]
259|     // CHECK: %[[EXT_P1:.*]] = arith.addi %[[EXT]], %[[ONE]]
260|     // CHECK: %[[EXT_CMP:.*]] = arith.cmpi sge, %[[EXT_P1]], %[[NUM_BUFS]]
261|     // CHECK: %[[EXT_NEXT:.*]] = arith.select %[[EXT_CMP]], %[[ZERO]], %[[EXT_P1]]
262|     %a = tt.load %a_ptr_init {loop.cluster = 2 : i32, loop.stage = 0 : i32} : tensor<128x32x!tt.ptr<f32>, #A>
263|     %b = tt.load %a_ptr_init {loop.cluster = 2 : i32, loop.stage = 0 : i32} : tensor<128x32x!tt.ptr<f32>, #A>
264|     "use1"(%a){loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x32xf32, #A>) -> ()
265|     "use2"(%b){loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x32xf32, #A>) -> ()
266|   } {tt.scheduled_max_stage = 2 : i32}
267|   tt.return
268| }
269| }
270| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 271-271
```mlir
271| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 272-274
```mlir
272| 
273| #A = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
274| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 275-275
```mlir
275| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 276-276
```mlir
276| // CHECK-LABEL: @two_load_groups
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @two_load_groups anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @two_load_groups 这样的标签用于锚定匹配范围。

### Lines 277-280
```mlir
277| tt.func @two_load_groups(%lb : index, %ub : index, %step : index,
278|                        %a_ptr_init : tensor<128x32x!tt.ptr<f32>, #A> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
279|                        %b_ptr_init : tensor<128x32x!tt.ptr<f32>, #A> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
280|                        %c_ptr_init : tensor<128x32x!tt.ptr<f32>, #A> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>}) -> () {
```
**EN:** This function-oriented block defines or enters `two_load_groups`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `two_load_groups` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 281-286
```mlir
281|   // CHECK-DAG: %[[MINUS_ONE:.*]] = arith.constant -1
282|   // CHECK-DAG: %[[ZERO:.*]] = arith.constant 0
283|   // CHECK-DAG: %[[ONE:.*]] = arith.constant 1
284|   // CHECK-DAG: %[[NUM_BUFS2:.*]] = arith.constant {{.*}} 2 : i32
285|   // CHECK-DAG: %[[NUM_BUFS3:.*]] = arith.constant {{.*}} 3 : i32
286|   // Two insert and extract indices are used.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 287-288
```mlir
287|   // CHECK: scf.for {{.*}} iter_args(%[[INS2:.*]] = %[[MINUS_ONE]], %[[EXT2:.*]] = %[[MINUS_ONE]], %[[INS3:.*]] = %[[MINUS_ONE]], %[[EXT3:.*]] = %[[MINUS_ONE]]) ->
288|   scf.for %iv = %lb to %ub step %step : index {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 289-306
```mlir
289|     // CHECK-DAG: %[[INS3_P1:.*]] = arith.addi %[[INS3]], %[[ONE]]
290|     // CHECK-DAG: %[[INS3_CMP:.*]] = arith.cmpi sge, %[[INS3_P1]], %[[NUM_BUFS3]]
291|     // CHECK-DAG: %[[INS3_NEXT:.*]] = arith.select %[[INS3_CMP]], %[[ZERO]], %[[INS3_P1]]
292|     // CHECK-DAG: %[[EXT3_P1:.*]] = arith.addi %[[EXT3]], %[[ONE]]
293|     // CHECK-DAG: %[[EXT3_CMP:.*]] = arith.cmpi sge, %[[EXT3_P1]], %[[NUM_BUFS3]]
294|     // CHECK-DAG: %[[EXT3_NEXT:.*]] = arith.select %[[EXT3_CMP]], %[[ZERO]], %[[EXT3_P1]]
295|     // CHECK-DAG: %[[INS2_P1:.*]] = arith.addi %[[INS2]], %[[ONE]]
296|     // CHECK-DAG: %[[INS2_CMP:.*]] = arith.cmpi sge, %[[INS2_P1]], %[[NUM_BUFS2]]
297|     // CHECK-DAG: %[[INS2_NEXT:.*]] = arith.select %[[INS2_CMP]], %[[ZERO]], %[[INS2_P1]]
298|     // CHECK-DAG: %[[EXT2_P1:.*]] = arith.addi %[[EXT2]], %[[ONE]]
299|     // CHECK-DAG: %[[EXT2_CMP:.*]] = arith.cmpi sge, %[[EXT2_P1]], %[[NUM_BUFS2]]
300|     // CHECK-DAG: %[[EXT2_NEXT:.*]] = arith.select %[[EXT2_CMP]], %[[ZERO]], %[[EXT2_P1]]
301|     %a = tt.load %a_ptr_init {loop.cluster = 3 : i32, loop.stage = 0 : i32} : tensor<128x32x!tt.ptr<f32>, #A>
302|     %b = tt.load %a_ptr_init {loop.cluster = 3 : i32, loop.stage = 0 : i32} : tensor<128x32x!tt.ptr<f32>, #A>
303|     %c = tt.load %a_ptr_init {loop.cluster = 3 : i32, loop.stage = 0 : i32} : tensor<128x32x!tt.ptr<f32>, #A>
304|     "use1"(%a){loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x32xf32, #A>) -> ()
305|     "use2"(%b){loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x32xf32, #A>) -> ()
306|     "use3"(%c){loop.cluster = 0 : i32, loop.stage = 3 : i32} : (tensor<128x32xf32, #A>) -> ()
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized loads. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的加载。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 307-311
```mlir
307|   } {tt.scheduled_max_stage = 3 : i32}
308|   tt.return
309| }
310| }
311| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 312-312
```mlir
312| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 313-315
```mlir
313| 
314| #A = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
315| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 316-316
```mlir
316| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 317-317
```mlir
317| // CHECK-LABEL: @dependent_loads
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @dependent_loads anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @dependent_loads 这样的标签用于锚定匹配范围。

### Lines 318-319
```mlir
318| tt.func @dependent_loads(%lb : index, %ub : index, %step : index,
319|                        %a_ptr_init : tensor<128x32x!tt.ptr<f32>, #A> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>}) -> () {
```
**EN:** This function-oriented block defines or enters `dependent_loads`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `dependent_loads` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 320-337
```mlir
320|   // CHECK-DAG: %[[MINUS_ONE:.*]] = arith.constant -1
321|   // CHECK-DAG: %[[ZERO:.*]] = arith.constant 0
322|   // CHECK-DAG: %[[ONE:.*]] = arith.constant 1
323|   // CHECK-DAG: %[[NUM_BUFS:.*]] = arith.constant {{.*}} 2 : i32
324|   // CHECK: %[[A:.*]] = ttg.local_alloc : () -> !ttg.memdesc<2x128x32xf32
325|   // CHECK: %[[C:.*]] = ttg.local_alloc : () -> !ttg.memdesc<2x128x32xf32
326|   // CHECK: scf.for {{.*}} iter_args(%[[INS:.*]] = %[[MINUS_ONE]], %[[EXT:.*]] = %[[MINUS_ONE]]) ->
327|   // CHECK: %[[INS_P1:.*]] = arith.addi %[[INS]], %[[ONE]] {loop.cluster = 4 : i32, loop.stage = 0 : i32}
328|   // CHECK: %[[INS_CMP:.*]] = arith.cmpi sge, %[[INS_P1]], %[[NUM_BUFS]] {loop.cluster = 4 : i32, loop.stage = 0 : i32}
329|   // CHECK: %[[INS_NEXT:.*]] = arith.select %[[INS_CMP]], %[[ZERO]], %[[INS_P1]] {loop.cluster = 4 : i32, loop.stage = 0 : i32}
330|   // CHECK: %[[EXT_P1:.*]] = arith.addi %[[EXT]], %[[ONE]] {loop.cluster = 2 : i32, loop.stage = 2 : i32}
331|   // CHECK: %[[EXT_CMP:.*]] = arith.cmpi sge, %[[EXT_P1]], %[[NUM_BUFS]] {loop.cluster = 2 : i32, loop.stage = 2 : i32}
332|   // CHECK: %[[EXT_NEXT:.*]] = arith.select %[[EXT_CMP]], %[[ZERO]], %[[EXT_P1]] {loop.cluster = 2 : i32, loop.stage = 2 : i32}
333|   // CHECK: %[[A_INS:.*]] = ttg.memdesc_index %[[A]]{{\[}}%[[INS_NEXT]]{{\]}} {loop.cluster = 4 : i32, loop.stage = 0 : i32}
334|   // CHECK: %[[A_TOK:.*]] = ttg.async_copy_global_to_local %{{.*}}, %[[A_INS]] {contiguity = 4 : i32, loop.cluster = 4 : i32, loop.stage = 0 : i32}
335|   // CHECK: %[[A_TOK2:.*]] = ttg.async_commit_group tokens %[[A_TOK]] {loop.cluster = 4 : i32, loop.stage = 0 : i32}
336|   // CHECK: %[[A_TOK3:.*]] = ttg.async_wait %[[A_TOK2]] {loop.cluster = 2 : i32, loop.stage = 2 : i32, num = 0 : i32}
337|   // CHECK: %[[A_EXT:.*]] = ttg.memdesc_index %[[A]]{{\[}}%[[EXT_NEXT]]{{\]}} {loop.cluster = 2 : i32, loop.stage = 2 : i32}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 338-355
```mlir
338|   // CHECK: %[[A_VAL:.*]] = ttg.local_load %[[A_EXT]] token %[[A_TOK3]] {loop.cluster = 2 : i32, loop.stage = 2 : i32}
339|   // CHECK: %[[B:.*]] = "pointerize"(%[[A_VAL]]) {loop.cluster = 2 : i32, loop.stage = 2 : i32}
340|   // CHECK: %[[C_INS:.*]] = ttg.memdesc_index %[[C]]{{\[}}%[[INS_NEXT]]{{\]}} {loop.cluster = 2 : i32, loop.stage = 2 : i32}
341|   // CHECK: %[[C_TOK:.*]] = ttg.async_copy_global_to_local %[[B]], %[[C_INS]] {loop.cluster = 2 : i32, loop.stage = 2 : i32}
342|   // CHECK: %[[C_TOK2:.*]] = ttg.async_commit_group tokens %[[C_TOK]] {loop.cluster = 2 : i32, loop.stage = 2 : i32}
343|   // CHECK: %[[C_TOK3:.*]] = ttg.async_wait %[[C_TOK2]] {loop.cluster = 0 : i32, loop.stage = 4 : i32, num = 0 : i32}
344|   // CHECK: %[[C_EXT:.*]] = ttg.memdesc_index %[[C]]{{\[}}%[[EXT_NEXT]]{{\]}} {loop.cluster = 0 : i32, loop.stage = 4 : i32}
345|   // CHECK: %[[C_VAL:.*]] = ttg.local_load %[[C_EXT]] token %[[C_TOK3]] {loop.cluster = 0 : i32, loop.stage = 4 : i32}
346|   // CHECK: "use1"(%[[C_VAL]]) {loop.cluster = 0 : i32, loop.stage = 4 : i32}
347|   // CHECK: scf.yield
348|   // CHECK-DAG: ttg.local_dealloc %[[A]]
349|   // CHECK-DAG: ttg.local_dealloc %[[C]]
350|   // CHECK-DAG:   ttg.async_wait  {num = 0 : i32}
351|   scf.for %iv = %lb to %ub step %step : index {
352|     %a = tt.load %a_ptr_init {loop.cluster = 4 : i32, loop.stage = 0 : i32} : tensor<128x32x!tt.ptr<f32>, #A>
353|     %b = "pointerize"(%a) {loop.cluster = 2 : i32, loop.stage = 2 : i32} : (tensor<128x32xf32, #A>) -> tensor<128x32x!tt.ptr<f32>, #A>
354|     %c = tt.load %b {loop.cluster = 2 : i32, loop.stage = 2 : i32} : tensor<128x32x!tt.ptr<f32>, #A>
355|     "use1"(%c){loop.cluster = 0 : i32, loop.stage = 4 : i32} : (tensor<128x32xf32, #A>) -> ()
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining masked or vectorized loads, structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 带掩码或向量化的加载、结构化循环。

### Lines 356-360
```mlir
356|   } {tt.scheduled_max_stage = 4 : i32}
357|   tt.return
358| }
359| }
360| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 361-361
```mlir
361| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 362-364
```mlir
362| 
363| #A = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
364| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 365-365
```mlir
365| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 366-367
```mlir
366| // CHECK-LABEL: @dependent_loads_asymmetric
367| // Loads have different latencies, should create two load groups.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 368-369
```mlir
368| tt.func @dependent_loads_asymmetric(%lb : index, %ub : index, %step : index,
369|                        %a_ptr_init : tensor<128x32x!tt.ptr<f32>, #A> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>}) -> () {
```
**EN:** This function-oriented block defines or enters `dependent_loads_asymmetric`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `dependent_loads_asymmetric` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 370-387
```mlir
370|   // CHECK-DAG: %[[MINUS_ONE:.*]] = arith.constant -1
371|   // CHECK-DAG: %[[ZERO:.*]] = arith.constant 0
372|   // CHECK-DAG: %[[ONE:.*]] = arith.constant 1
373|   // CHECK-DAG: %[[NUM_BUFS2:.*]] = arith.constant {{.*}} 2 : i32
374|   // CHECK-DAG: %[[NUM_BUFS3:.*]] = arith.constant {{.*}} 3 : i32
375|   // CHECK: %[[A:.*]] = ttg.local_alloc : () -> !ttg.memdesc<2x128x32xf32
376|   // CHECK: %[[C:.*]] = ttg.local_alloc : () -> !ttg.memdesc<3x128x32xf32
377|   // CHECK: scf.for {{.*}} iter_args(%[[INS2:.*]] = %[[MINUS_ONE]], %[[EXT2:.*]] = %[[MINUS_ONE]], %[[INS3:.*]] = %[[MINUS_ONE]], %[[EXT3:.*]] = %[[MINUS_ONE]]) ->
378|   // CHECK-DAG: %[[INS3_P1:.*]] = arith.addi %[[INS3]], %[[ONE]] {loop.cluster = 2 : i32, loop.stage = 2 : i32}
379|   // CHECK-DAG: %[[INS3_CMP:.*]] = arith.cmpi sge, %[[INS3_P1]], %[[NUM_BUFS3]] {loop.cluster = 2 : i32, loop.stage = 2 : i32}
380|   // CHECK-DAG: %[[INS3_NEXT:.*]] = arith.select %[[INS3_CMP]], %[[ZERO]], %[[INS3_P1]] {loop.cluster = 2 : i32, loop.stage = 2 : i32}
381|   // CHECK-DAG: %[[EXT3_P1:.*]] = arith.addi %[[EXT3]], %[[ONE]] {loop.cluster = 0 : i32, loop.stage = 5 : i32}
382|   // CHECK-DAG: %[[EXT3_CMP:.*]] = arith.cmpi sge, %[[EXT3_P1]], %[[NUM_BUFS3]] {loop.cluster = 0 : i32, loop.stage = 5 : i32}
383|   // CHECK-DAG: %[[EXT3_NEXT:.*]] = arith.select %[[EXT3_CMP]], %[[ZERO]], %[[EXT3_P1]] {loop.cluster = 0 : i32, loop.stage = 5 : i32}
384|   // CHECK-DAG: %[[INS2_P1:.*]] = arith.addi %[[INS2]], %[[ONE]] {loop.cluster = 4 : i32, loop.stage = 0 : i32}
385|   // CHECK-DAG: %[[INS2_CMP:.*]] = arith.cmpi sge, %[[INS2_P1]], %[[NUM_BUFS2]] {loop.cluster = 4 : i32, loop.stage = 0 : i32}
386|   // CHECK-DAG: %[[INS2_NEXT:.*]] = arith.select %[[INS2_CMP]], %[[ZERO]], %[[INS2_P1]] {loop.cluster = 4 : i32, loop.stage = 0 : i32}
387|   // CHECK-DAG: %[[EXT2_P1:.*]] = arith.addi %[[EXT2]], %[[ONE]] {loop.cluster = 2 : i32, loop.stage = 2 : i32}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 388-405
```mlir
388|   // CHECK-DAG: %[[EXT2_CMP:.*]] = arith.cmpi sge, %[[EXT2_P1]], %[[NUM_BUFS2]] {loop.cluster = 2 : i32, loop.stage = 2 : i32}
389|   // CHECK-DAG: %[[EXT2_NEXT:.*]] = arith.select %[[EXT2_CMP]], %[[ZERO]], %[[EXT2_P1]] {loop.cluster = 2 : i32, loop.stage = 2 : i32}
390|   // CHECK: %[[A_INS:.*]] = ttg.memdesc_index %[[A]]{{\[}}%[[INS2_NEXT]]{{\]}} {loop.cluster = 4 : i32, loop.stage = 0 : i32}
391|   // CHECK: %[[A_TOK:.*]] = ttg.async_copy_global_to_local %{{.*}}, %[[A_INS]] {contiguity = 4 : i32, loop.cluster = 4 : i32, loop.stage = 0 : i32}
392|   // CHECK: %[[A_TOK2:.*]] = ttg.async_commit_group tokens %[[A_TOK]] {loop.cluster = 4 : i32, loop.stage = 0 : i32}
393|   // CHECK: %[[A_TOK3:.*]] = ttg.async_wait %[[A_TOK2]] {loop.cluster = 2 : i32, loop.stage = 2 : i32, num = 0 : i32}
394|   // CHECK: %[[A_EXT:.*]] = ttg.memdesc_index %[[A]]{{\[}}%[[EXT2_NEXT]]{{\]}} {loop.cluster = 2 : i32, loop.stage = 2 : i32}
395|   // CHECK: %[[A_VAL:.*]] = ttg.local_load %[[A_EXT]] token %[[A_TOK3]] {loop.cluster = 2 : i32, loop.stage = 2 : i32}
396|   // CHECK: %[[B:.*]] = "pointerize"(%[[A_VAL]]) {loop.cluster = 2 : i32, loop.stage = 2 : i32}
397|   // CHECK: %[[C_INS:.*]] = ttg.memdesc_index
398|   // CHECK: %[[C_TOK:.*]] = ttg.async_copy_global_to_local %[[B]], %[[C_INS]] {loop.cluster = 2 : i32, loop.stage = 2 : i32}
399|   // CHECK: %[[C_TOK2:.*]] = ttg.async_commit_group tokens %[[C_TOK]] {loop.cluster = 2 : i32, loop.stage = 2 : i32}
400|   // CHECK: %[[C_TOK3:.*]] = ttg.async_wait %[[C_TOK2]] {loop.cluster = 0 : i32, loop.stage = 5 : i32, num = 0 : i32}
401|   // CHECK: %[[C_EXT:.*]] = ttg.memdesc_index %[[C]]{{\[}}%[[EXT3_NEXT]]{{\]}} {loop.cluster = 0 : i32, loop.stage = 5 : i32}
402|   // CHECK: %[[C_VAL:.*]] = ttg.local_load %[[C_EXT]] token %[[C_TOK3]] {loop.cluster = 0 : i32, loop.stage = 5 : i32}
403|   // CHECK: "use1"(%[[C_VAL]]) {loop.cluster = 0 : i32, loop.stage = 5 : i32}
404|   // CHECK: scf.yield
405|   // CHECK-DAG: ttg.local_dealloc %[[A]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 406-417
```mlir
406|   // CHECK-DAG: ttg.local_dealloc %[[C]]
407|   // CHECK-DAG: ttg.async_wait  {num = 0 : i32}
408|   scf.for %iv = %lb to %ub step %step : index {
409|     %a = tt.load %a_ptr_init {loop.cluster = 4 : i32, loop.stage = 0 : i32} : tensor<128x32x!tt.ptr<f32>, #A>
410|     %b = "pointerize"(%a) {loop.cluster = 2 : i32, loop.stage = 2 : i32} : (tensor<128x32xf32, #A>) -> tensor<128x32x!tt.ptr<f32>, #A>
411|     %c = tt.load %b {loop.cluster = 2 : i32, loop.stage = 2 : i32} : tensor<128x32x!tt.ptr<f32>, #A>
412|     "use1"(%c){loop.cluster = 0 : i32, loop.stage = 5 : i32} : (tensor<128x32xf32, #A>) -> ()
413|   } {tt.scheduled_max_stage = 5 : i32}
414|   tt.return
415| }
416| }
417| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining masked or vectorized loads, structured loops, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 带掩码或向量化的加载、结构化循环、tt.return。

### Lines 418-418
```mlir
418| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 419-421
```mlir
419| 
420| #A = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
421| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 422-422
```mlir
422| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 423-423
```mlir
423| // CHECK-LABEL: @unused_load
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @unused_load anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @unused_load 这样的标签用于锚定匹配范围。

### Lines 424-425
```mlir
424| tt.func @unused_load(%lb : index, %ub : index, %step : index,
425|                        %a_ptr_init : tensor<128x32x!tt.ptr<f32>, #A> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>}) -> () {
```
**EN:** This function-oriented block defines or enters `unused_load`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `unused_load` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 426-427
```mlir
426|   // CHECK: scf.for
427|   scf.for %iv = %lb to %ub step %step : index {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 428-435
```mlir
428|     // CHECK: dummy
429|     %a = tt.load %a_ptr_init {loop.cluster = 0 : i32, loop.stage = 1 : i32} : tensor<128x32x!tt.ptr<f32>, #A>
430|     "dummy"() : () -> ()
431|   } {tt.scheduled_max_stage = 1 : i32}
432|   tt.return
433| }
434| }
435| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 436-436
```mlir
436| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 437-441
```mlir
437| 
438| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 16, 16]}>
439| #blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
440| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
441| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 16}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 442-442
```mlir
442| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 443-460
```mlir
443|   // CHECK-LABEL: @shmem_pipelining_mmav3
444|   // CHECK-DAG: %[[INIT:.*]] = arith.constant dense<0.000000e+00>
445|   // CHECK-DAG: %[[MINUS_ONE:.*]] = arith.constant -1
446|   // CHECK-DAG: %[[ZERO:.*]] = arith.constant 0
447|   // CHECK-DAG: %[[ONE:.*]] = arith.constant 1
448|   // CHECK-DAG: %[[NUM_BUFS:.*]] = arith.constant {{.*}} 3 : i32
449|   // CHECK: %[[A:.*]] = ttg.local_alloc : () -> !ttg.memdesc<3x128x128
450|   // CHECK: %[[B:.*]] = ttg.local_alloc : () -> !ttg.memdesc<3x128x128
451|   // CHECK: scf.for {{.*}} iter_args(%[[ACC:.*]] = %[[INIT]], %[[INS:.*]] = %[[MINUS_ONE]], %[[EXT:.*]] = %[[MINUS_ONE]])
452|   // CHECK:   %[[INS_P1:.*]] = arith.addi %[[INS]], %[[ONE]] {loop.cluster = 2 : i32, loop.stage = 0 : i32}  : i32
453|   // CHECK:   %[[INS_CMP:.*]] = arith.cmpi sge, %[[INS_P1]], %[[NUM_BUFS]] {loop.cluster = 2 : i32, loop.stage = 0 : i32}  : i32
454|   // CHECK:   %[[INS_NEXT:.*]] = arith.select %[[INS_CMP]], %[[ZERO]], %[[INS_P1]] {loop.cluster = 2 : i32, loop.stage = 0 : i32}  : i32
455|   // CHECK:   %[[EXT_P1:.*]] = arith.addi %[[EXT]], %[[ONE]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}  : i32
456|   // CHECK:   %[[EXT_CMP:.*]] = arith.cmpi sge, %[[EXT_P1]], %[[NUM_BUFS]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}  : i32
457|   // CHECK:   %[[EXT_NEXT:.*]] = arith.select %[[EXT_CMP]], %[[ZERO]], %[[EXT_P1]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}  : i32
458|   // CHECK:   %[[A_INS:.*]] = ttg.memdesc_index %[[A]]{{\[}}%[[INS_NEXT]]{{\]}} {loop.cluster = 2 : i32, loop.stage = 0 : i32}
459|   // CHECK:   %[[A_TOK:.*]] = ttg.async_copy_global_to_local %{{.*}}, %[[A_INS]] {contiguity = 8 : i32, loop.cluster = 2 : i32, loop.stage = 0 : i32}
460|   // CHECK:   %[[A_TOK2:.*]] = ttg.async_commit_group tokens %[[A_TOK]] {loop.cluster = 2 : i32, loop.stage = 0 : i32}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @shmem_pipelining_mmav3 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @shmem_pipelining_mmav3 这样的标签用于锚定匹配范围。

### Lines 461-473
```mlir
461|   // CHECK:   %[[A_TOK3:.*]] = ttg.async_wait %[[A_TOK2]] {loop.cluster = 0 : i32, loop.stage = 2 : i32, num = 0 : i32}
462|   // CHECK:   %[[A_EXT:.*]] = ttg.memdesc_index %[[A]]{{\[}}%[[EXT_NEXT]]{{\]}}{{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
463|   // CHECK:   %[[B_INS:.*]] = ttg.memdesc_index %[[B]]{{\[}}%[[INS_NEXT]]{{\]}}{{.*}} {loop.cluster = 2 : i32, loop.stage = 0 : i32}
464|   // CHECK:   %[[B_TOK:.*]] = ttg.async_copy_global_to_local %{{.*}}, %[[B_INS]] {contiguity = 8 : i32, loop.cluster = 2 : i32, loop.stage = 0 : i32}
465|   // CHECK:   %[[B_TOK2:.*]] = ttg.async_commit_group tokens %[[B_TOK]] {loop.cluster = 2 : i32, loop.stage = 0 : i32}
466|   // CHECK:   %[[B_TOK3:.*]] = ttg.async_wait %[[B_TOK2]] {loop.cluster = 0 : i32, loop.stage = 2 : i32, num = 0 : i32}
467|   // CHECK:   %[[B_EXT:.*]] = ttg.memdesc_index %[[B]]{{\[}}%[[EXT_NEXT]]{{\]}}{{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
468|   // CHECK:   %[[A_EXT_TRANSP:.*]] = ttg.memdesc_trans %[[A_EXT]] {loop.cluster = 0 : i32, loop.stage = 2 : i32, order = array<i32: 1, 0>}
469|   // CHECK:   ttng.warp_group_dot %[[A_EXT_TRANSP]], %[[B_EXT]], %{{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
470|   // CHECK:   scf.yield {{.*}}, %[[INS_NEXT]], %[[EXT_NEXT]]
471|   // CHECK-DAG: ttg.local_dealloc %[[A]]
472|   // CHECK-DAG: ttg.local_dealloc %[[B]]
473|   // CHECK-DAG: ttg.async_wait  {num = 0 : i32}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 474-491
```mlir
474|   tt.func public @shmem_pipelining_mmav3(%lb : index, %ub : index, %step : index,
475|                                               %A_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
476|                                               %B_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>}) -> tensor<128x128xf16, #mma> {
477|     %true = arith.constant true
478|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #mma>
479|     %c0_i32 = arith.constant 0 : i32
480|     %res = scf.for %i = %lb to %ub step %step iter_args(%acc = %cst) -> (tensor<128x128xf32, #mma>) : index {
481|       %A = tt.load %A_ptr  {loop.cluster = 2 : i32, loop.stage = 0 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked1>
482|       %B = tt.load %B_ptr  {loop.cluster = 2 : i32, loop.stage = 0 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked1>
483|       %A_sh = ttg.local_alloc %A {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
484|       %B_sh = ttg.local_alloc %B {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
485|       %A_transp = ttg.memdesc_trans %A_sh {order = array<i32: 1, 0>} : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory> -> !ttg.memdesc<128x128xf16, #shared1, #ttg.shared_memory>
486|       %acc_res = ttng.warp_group_dot %A_transp, %B_sh, %acc {loop.cluster = 0 : i32, loop.stage = 2 : i32} : !ttg.memdesc<128x128xf16, #shared1, #ttg.shared_memory> * !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory> -> tensor<128x128xf32, #mma>
487|       scf.yield %acc_res : tensor<128x128xf32, #mma>
488|     } {tt.scheduled_max_stage = 2 : i32}
489|     %res_f16 = arith.truncf %res : tensor<128x128xf32, #mma> to tensor<128x128xf16, #mma>
490|     tt.return %res_f16 : tensor<128x128xf16, #mma>
491|   }
```
**EN:** This function-oriented block defines or enters `shmem_pipelining_mmav3`. Within it, the test exercises constants, tt.func, masked or vectorized loads, shared/local memory allocation, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `shmem_pipelining_mmav3` 为核心。测试在其中演示 常量、tt.func、带掩码或向量化的加载、共享/本地内存分配、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 492-493
```mlir
492| }
493| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 494-494
```mlir
494| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 495-499
```mlir
495| 
496| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 16, 16]}>
497| #blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
498| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
499| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 16}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 500-500
```mlir
500| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 501-503
```mlir
501|   // CHECK-LABEL: @shmem_pipelining_mmav3_two_users
502|   // CHECK: ttg.local_alloc : () -> !ttg.memdesc<3x128x128
503|   // CHECK: ttg.local_alloc : () -> !ttg.memdesc<3x128x128
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @shmem_pipelining_mmav3_two_users anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @shmem_pipelining_mmav3_two_users 这样的标签用于锚定匹配范围。

### Lines 504-521
```mlir
504|   tt.func public @shmem_pipelining_mmav3_two_users(%lb : index, %ub : index, %step : index,
505|                                               %A_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
506|                                               %B_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>}) -> tensor<128x128xf16, #mma> {
507|     %true = arith.constant true
508|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #mma>
509|     %c0_i32 = arith.constant 0 : i32
510|     %res = scf.for %i = %lb to %ub step %step iter_args(%acc = %cst) -> (tensor<128x128xf32, #mma>) : index {
511|       %A = tt.load %A_ptr  {loop.cluster = 2 : i32, loop.stage = 0 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked1>
512|       %B = tt.load %B_ptr  {loop.cluster = 2 : i32, loop.stage = 0 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked1>
513|       %A_sh = ttg.local_alloc %A {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
514|       %B_sh = ttg.local_alloc %B {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
515|       %A_transp = ttg.memdesc_trans %A_sh {order = array<i32: 1, 0>} : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory> -> !ttg.memdesc<128x128xf16, #shared1, #ttg.shared_memory>
516|       %acc_res = ttng.warp_group_dot %A_transp, %B_sh, %acc {loop.cluster = 0 : i32, loop.stage = 2 : i32} : !ttg.memdesc<128x128xf16, #shared1, #ttg.shared_memory> * !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory> -> tensor<128x128xf32, #mma>
517|       %acc_res2 = ttng.warp_group_dot %A_transp, %B_sh, %acc {loop.cluster = 0 : i32, loop.stage = 2 : i32} : !ttg.memdesc<128x128xf16, #shared1, #ttg.shared_memory> * !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory> -> tensor<128x128xf32, #mma>
518|       scf.yield %acc_res : tensor<128x128xf32, #mma>
519|     } {tt.scheduled_max_stage = 2 : i32}
520|     %res_f16 = arith.truncf %res : tensor<128x128xf32, #mma> to tensor<128x128xf16, #mma>
521|     tt.return %res_f16 : tensor<128x128xf16, #mma>
```
**EN:** This function-oriented block defines or enters `shmem_pipelining_mmav3_two_users`. Within it, the test exercises constants, tt.func, masked or vectorized loads, shared/local memory allocation, ttng.warp_group_dot, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `shmem_pipelining_mmav3_two_users` 为核心。测试在其中演示 常量、tt.func、带掩码或向量化的加载、共享/本地内存分配、ttng.warp_group_dot，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 522-524
```mlir
522|   }
523| }
524| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 525-525
```mlir
525| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 526-531
```mlir
526| 
527| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 16, 16]}>
528| #blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
529| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
530| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 32}>
531| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 532-534
```mlir
532| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
533|   // The combination of blocked and shared layouts for operand B would result in cp.async with less than 4 bytes size.
534|   // We can't pipeline that using shared memory buffer.
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 535-552
```mlir
535|   // CHECK-LABEL: @no_shmem_pipelining_incompat_layout
536|   // CHECK-DAG: %[[INIT:.*]] = arith.constant dense<0.000000e+00>
537|   // CHECK-DAG: %[[MINUS_ONE:.*]] = arith.constant -1
538|   // CHECK-DAG: %[[ZERO:.*]] = arith.constant 0
539|   // CHECK-DAG: %[[ONE:.*]] = arith.constant 1
540|   // CHECK-DAG: %[[NUM_BUFS:.*]] = arith.constant {{.*}} 3 : i32
541|   // CHECK: %[[A:.*]] = ttg.local_alloc : () -> !ttg.memdesc<3x128x128
542|   // CHECK: scf.for {{.*}} iter_args(%[[ACC:.*]] = %[[INIT]], %[[INS:.*]] = %[[MINUS_ONE]], %[[EXT:.*]] = %[[MINUS_ONE]])
543|   // CHECK:   %[[INS_P1:.*]] = arith.addi %[[INS]], %[[ONE]] {loop.cluster = 2 : i32, loop.stage = 0 : i32}  : i32
544|   // CHECK:   %[[INS_CMP:.*]] = arith.cmpi sge, %[[INS_P1]], %[[NUM_BUFS]] {loop.cluster = 2 : i32, loop.stage = 0 : i32}  : i32
545|   // CHECK:   %[[INS_NEXT:.*]] = arith.select %[[INS_CMP]], %[[ZERO]], %[[INS_P1]] {loop.cluster = 2 : i32, loop.stage = 0 : i32}  : i32
546|   // CHECK:   %[[EXT_P1:.*]] = arith.addi %[[EXT]], %[[ONE]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}  : i32
547|   // CHECK:   %[[EXT_CMP:.*]] = arith.cmpi sge, %[[EXT_P1]], %[[NUM_BUFS]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}  : i32
548|   // CHECK:   %[[EXT_NEXT:.*]] = arith.select %[[EXT_CMP]], %[[ZERO]], %[[EXT_P1]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}  : i32
549|   // CHECK:   %[[A_INS:.*]] = ttg.memdesc_index %[[A]]{{\[}}%[[INS_NEXT]]{{\]}} {loop.cluster = 2 : i32, loop.stage = 0 : i32}
550|   // CHECK:   %[[A_TOK:.*]] = ttg.async_copy_global_to_local %{{.*}}, %[[A_INS]] {contiguity = 8 : i32, loop.cluster = 2 : i32, loop.stage = 0 : i32}
551|   // CHECK:   %[[A_TOK2:.*]] = ttg.async_commit_group tokens %[[A_TOK]] {loop.cluster = 2 : i32, loop.stage = 0 : i32}
552|   // CHECK:   %[[A_TOK3:.*]] = ttg.async_wait %[[A_TOK2]] {loop.cluster = 0 : i32, loop.stage = 2 : i32, num = 0 : i32}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @no_shmem_pipelining_incompat_layout anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @no_shmem_pipelining_incompat_layout 这样的标签用于锚定匹配范围。

### Lines 553-559
```mlir
553|   // CHECK:   %[[A_EXT:.*]] = ttg.memdesc_index %[[A]]{{\[}}%[[EXT_NEXT]]{{\]}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
554|   // CHECK:   %[[B:.*]] = tt.load {{.*}} {loop.cluster = 2 : i32, loop.stage = 0 : i32}
555|   // CHECK:   %[[B_SH:.*]] = ttg.local_alloc %[[B]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
556|   // CHECK:   ttng.warp_group_dot %[[A_EXT]], %[[B_SH]], %{{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
557|   // CHECK:   scf.yield {{.*}}, %[[INS_NEXT]], %[[EXT_NEXT]]
558|   // CHECK-DAG:   ttg.local_dealloc %[[A]]
559|   // CHECK-DAG:   ttg.async_wait  {num = 0 : i32}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 560-577
```mlir
560|   tt.func public @no_shmem_pipelining_incompat_layout(
561|                     %lb : index, %ub : index, %step : index,
562|                     %A_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
563|                     %B_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>}) -> tensor<128x128xf32, #mma> {
564|     %true = arith.constant true
565|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #mma>
566|     %c0_i32 = arith.constant 0 : i32
567|     %res = scf.for %i = %lb to %ub step %step iter_args(%acc = %cst) -> (tensor<128x128xf32, #mma>) : index {
568|       %A = tt.load %A_ptr  {loop.cluster = 2 : i32, loop.stage = 0 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked1>
569|       %B = tt.load %B_ptr  {loop.cluster = 2 : i32, loop.stage = 0 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked1>
570|       %A_sh = ttg.local_alloc %A {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
571|       %B_sh = ttg.local_alloc %B {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared1, #ttg.shared_memory>
572|       %acc_res = ttng.warp_group_dot %A_sh, %B_sh, %acc {loop.cluster = 0 : i32, loop.stage = 2 : i32} : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory> * !ttg.memdesc<128x128xf16, #shared1, #ttg.shared_memory> -> tensor<128x128xf32, #mma>
573|       scf.yield %acc_res : tensor<128x128xf32, #mma>
574|     } {tt.scheduled_max_stage = 2 : i32}
575|     tt.return %res : tensor<128x128xf32, #mma>
576|   }
577| }
```
**EN:** This function-oriented block defines or enters `no_shmem_pipelining_incompat_layout`. Within it, the test exercises constants, tt.func, masked or vectorized loads, shared/local memory allocation, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `no_shmem_pipelining_incompat_layout` 为核心。测试在其中演示 常量、tt.func、带掩码或向量化的加载、共享/本地内存分配、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 579-580
```mlir
579| 
580| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 580-584
```mlir
580| 
581| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 16, 16]}>
582| #blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
583| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
584| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 585-587
```mlir
585| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
586|   // non-zero "other" value is used in the load, while cp.async does not support it.
587|   // We can't feed the shared memory values directly to mma, we need other values being filled in the registers.
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 588-605
```mlir
588|   // CHECK-LABEL: @no_shmem_pipelining_other_used
589|   // CHECK-DAG: %[[INIT:.*]] = arith.constant dense<0.000000e+00>
590|   // CHECK-DAG: %[[MINUS_ONE:.*]] = arith.constant -1
591|   // CHECK-DAG: %[[ZERO:.*]] = arith.constant 0
592|   // CHECK-DAG: %[[ONE:.*]] = arith.constant 1
593|   // CHECK-DAG: %[[NUM_BUFS:.*]] = arith.constant {{.*}} 2 : i32
594|   // CHECK: %[[A:.*]] = ttg.local_alloc : () -> !ttg.memdesc<2x128x128
595|   // CHECK: %[[B:.*]] = ttg.local_alloc : () -> !ttg.memdesc<2x128x128
596|   // CHECK: scf.for {{.*}} iter_args(%[[ACC:[^,]*]] = %[[INIT]], %[[INS:[^,]*]] = %[[MINUS_ONE]], %[[EXT:[^,]*]] = %[[MINUS_ONE]])
597|   // CHECK:   %[[INS_P1:.*]] = arith.addi %[[INS]], %[[ONE]] {loop.cluster = 2 : i32, loop.stage = 0 : i32}  : i32
598|   // CHECK:   %[[INS_CMP:.*]] = arith.cmpi sge, %[[INS_P1]], %[[NUM_BUFS]] {loop.cluster = 2 : i32, loop.stage = 0 : i32}  : i32
599|   // CHECK:   %[[INS_NEXT:.*]] = arith.select %[[INS_CMP]], %[[ZERO]], %[[INS_P1]] {loop.cluster = 2 : i32, loop.stage = 0 : i32}  : i32
600|   // CHECK:   %[[EXT_P1:.*]] = arith.addi %[[EXT]], %[[ONE]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}  : i32
601|   // CHECK:   %[[EXT_CMP:.*]] = arith.cmpi sge, %[[EXT_P1]], %[[NUM_BUFS]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}  : i32
602|   // CHECK:   %[[EXT_NEXT:.*]] = arith.select %[[EXT_CMP]], %[[ZERO]], %[[EXT_P1]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}  : i32
603|   // CHECK:   %[[A_INS:.*]] = ttg.memdesc_index %[[A]]{{\[}}%[[INS_NEXT]]{{\]}} {loop.cluster = 2 : i32, loop.stage = 0 : i32}
604|   // CHECK:   %[[A_TOK:.*]] = ttg.async_copy_global_to_local %{{.*}}, %[[A_INS]] {{.*}} {contiguity = 8 : i32, loop.cluster = 2 : i32, loop.stage = 0 : i32}
605|   // CHECK:   %[[A_TOK2:.*]] = ttg.async_commit_group tokens %[[A_TOK]] {loop.cluster = 2 : i32, loop.stage = 0 : i32}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @no_shmem_pipelining_other_used anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @no_shmem_pipelining_other_used 这样的标签用于锚定匹配范围。

### Lines 606-623
```mlir
606|   // CHECK:   %[[A_TOK3:.*]] = ttg.async_wait %[[A_TOK2]] {loop.cluster = 0 : i32, loop.stage = 2 : i32, num = 0 : i32}
607|   // CHECK:   %[[A_EXT:.*]] = ttg.memdesc_index %[[A]]{{\[}}%[[EXT_NEXT]]{{\]}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
608|   // CHECK:   %[[A_LOAD:.*]] = ttg.local_load %[[A_EXT]] {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
609|   // CHECK:   %[[A_MASKED:.*]] = arith.select {{.*}}, %[[A_LOAD]], {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
610|   // CHECK:   %[[B_INS:.*]] = ttg.memdesc_index %[[B]]{{\[}}%[[INS_NEXT]]{{\]}} {loop.cluster = 2 : i32, loop.stage = 0 : i32}
611|   // CHECK:   %[[B_TOK:.*]] = ttg.async_copy_global_to_local %{{.*}}, %[[B_INS]] {{.*}} {contiguity = 8 : i32, loop.cluster = 2 : i32, loop.stage = 0 : i32}
612|   // CHECK:   %[[B_TOK2:.*]] = ttg.async_commit_group tokens %[[B_TOK]] {loop.cluster = 2 : i32, loop.stage = 0 : i32}
613|   // CHECK:   %[[B_TOK3:.*]] = ttg.async_wait %[[B_TOK2]] {loop.cluster = 0 : i32, loop.stage = 2 : i32, num = 0 : i32}
614|   // CHECK:   %[[B_EXT:.*]] = ttg.memdesc_index %[[B]]{{\[}}%[[EXT_NEXT]]{{\]}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
615|   // CHECK:   %[[B_LOAD:.*]] = ttg.local_load %[[B_EXT]] {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
616|   // CHECK:   %[[B_MASKED:.*]] = arith.select {{.*}}, %[[B_LOAD]], {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
617|   // CHECK:   %[[A_SH:.*]] = ttg.local_alloc %[[A_MASKED]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
618|   // CHECK:   %[[B_SH:.*]] = ttg.local_alloc %[[B_MASKED]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
619|   // CHECK:   ttng.warp_group_dot %[[A_SH]], %[[B_SH]], %{{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
620|   // CHECK:   scf.yield {{.*}}, %[[INS_NEXT]], %[[EXT_NEXT]]
621|   // CHECK-DAG: ttg.local_dealloc %[[A]]
622|   // CHECK-DAG: ttg.local_dealloc %[[B]]
623|   // CHECK-DAG: ttg.async_wait  {num = 0 : i32}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 624-641
```mlir
624|   tt.func public @no_shmem_pipelining_other_used(
625|                       %lb : index, %ub : index, %step : index,
626|                       %A_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
627|                       %B_ptr: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
628|                       %mask: tensor<128x128xi1, #blocked1> {tt.constancy = dense<[128, 128]> : tensor<2xi32>},
629|                       %other: tensor<128x128xf16, #blocked1> {tt.constancy = dense<[128, 128]> : tensor<2xi32>}) -> tensor<128x128xf16, #mma> {
630|     %true = arith.constant true
631|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #mma>
632|     %c0_i32 = arith.constant 0 : i32
633|     %res = scf.for %i = %lb to %ub step %step iter_args(%acc = %cst) -> (tensor<128x128xf32, #mma>) : index {
634|       %A = tt.load %A_ptr, %mask, %other  {loop.cluster = 2 : i32, loop.stage = 0 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked1>
635|       %B = tt.load %B_ptr, %mask, %other {loop.cluster = 2 : i32, loop.stage = 0 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked1>
636|       %A_sh = ttg.local_alloc %A {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
637|       %B_sh = ttg.local_alloc %B {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
638|       %acc_res = ttng.warp_group_dot %A_sh, %B_sh, %acc {loop.cluster = 0 : i32, loop.stage = 2 : i32} : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory> * !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory> -> tensor<128x128xf32, #mma>
639|       scf.yield %acc_res : tensor<128x128xf32, #mma>
640|     } {tt.scheduled_max_stage = 2 : i32}
641|     %res_f16 = arith.truncf %res : tensor<128x128xf32, #mma> to tensor<128x128xf16, #mma>
```
**EN:** This function-oriented block defines or enters `no_shmem_pipelining_other_used`. Within it, the test exercises constants, tt.func, masked or vectorized loads, shared/local memory allocation, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `no_shmem_pipelining_other_used` 为核心。测试在其中演示 常量、tt.func、带掩码或向量化的加载、共享/本地内存分配、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 642-645
```mlir
642|     tt.return %res_f16 : tensor<128x128xf16, #mma>
643|   }
644| }
645| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 646-646
```mlir
646| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 647-651
```mlir
647| 
648| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
649| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
650| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
651| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 652-652
```mlir
652| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 653-670
```mlir
653|   // CHECK-LABEL: @shmem_pipelining_mmav5
654|   // CHECK-DAG: %[[INIT:.*]] = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
655|   // CHECK-DAG: %[[MINUS_ONE:.*]] = arith.constant -1
656|   // CHECK-DAG: %[[ZERO:.*]] = arith.constant 0
657|   // CHECK-DAG: %[[ONE:.*]] = arith.constant 1
658|   // CHECK-DAG: %[[TWO:.*]] = arith.constant{{.*}} 2 : i32
659|   // CHECK-DAG: %[[NUM_BUFS:.*]] = arith.constant{{.*}}3 : i32
660|   // CHECK: %[[ACC_TM:.*]], %[[ACC_TOK:.*]] = ttng.tmem_alloc : ()
661|   // CHECK: %[[INIT_TOK:.*]] = ttng.tmem_store %[[INIT]], %[[ACC_TM]][%[[ACC_TOK]]]
662|   // CHECK: %[[BAR:.*]] = ttg.local_alloc  : () -> !ttg.memdesc<2x1xi64
663|   // CHECK: %[[BAR_SUB1:.*]] = ttg.memdesc_index %[[BAR]]{{\[}}%[[ZERO]]{{\]}}
664|   // CHECK: ttng.init_barrier %[[BAR_SUB1]], 1
665|   // CHECK: %[[BAR_SUB2:.*]] = ttg.memdesc_index %[[BAR]]{{\[}}%[[ONE]]{{\]}}
666|   // CHECK: ttng.init_barrier %[[BAR_SUB2]], 1
667|   // CHECK: %[[A:.*]] = ttg.local_alloc : () -> !ttg.memdesc<3x128x128
668|   // CHECK: %[[B:.*]] = ttg.local_alloc : () -> !ttg.memdesc<3x128x128
669|   // CHECK: %[[FOR_RET:.*]] = scf.for {{.*}} iter_args(%[[TOK:.*]] = %[[INIT_TOK]], %[[PHASE:.*]] = %[[ZERO]], %[[BAR_IDX:.*]] = %[[ZERO]], %[[INS:.*]] = %[[MINUS_ONE]], %[[EXT:.*]] = %[[MINUS_ONE]])
670|   // CHECK:   %[[INS_P1:.*]] = arith.addi %[[INS]], %[[ONE]] {loop.cluster = 2 : i32, loop.stage = 0 : i32}  : i32
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @shmem_pipelining_mmav5 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @shmem_pipelining_mmav5 这样的标签用于锚定匹配范围。

### Lines 671-688
```mlir
671|   // CHECK:   %[[INS_CMP:.*]] = arith.cmpi sge, %[[INS_P1]], %[[NUM_BUFS]] {loop.cluster = 2 : i32, loop.stage = 0 : i32}  : i32
672|   // CHECK:   %[[INS_NEXT:.*]] = arith.select %[[INS_CMP]], %[[ZERO]], %[[INS_P1]] {loop.cluster = 2 : i32, loop.stage = 0 : i32}  : i32
673|   // CHECK:   %[[EXT_P1:.*]] = arith.addi %[[EXT]], %[[ONE]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}  : i32
674|   // CHECK:   %[[EXT_CMP:.*]] = arith.cmpi sge, %[[EXT_P1]], %[[NUM_BUFS]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}  : i32
675|   // CHECK:   %[[EXT_NEXT:.*]] = arith.select %[[EXT_CMP]], %[[ZERO]], %[[EXT_P1]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}  : i32
676|   // CHECK:   %[[A_INS:.*]] = ttg.memdesc_index %[[A]]{{\[}}%[[INS_NEXT]]{{\]}} {loop.cluster = 2 : i32, loop.stage = 0 : i32}
677|   // CHECK:   %[[A_TOK:.*]] = ttg.async_copy_global_to_local %{{.*}}, %[[A_INS]] {contiguity = 8 : i32, loop.cluster = 2 : i32, loop.stage = 0 : i32}
678|   // CHECK:   %[[A_TOK2:.*]] = ttg.async_commit_group tokens %[[A_TOK]] {loop.cluster = 2 : i32, loop.stage = 0 : i32}
679|   // CHECK:   %[[A_TOK3:.*]] = ttg.async_wait %[[A_TOK2]] {loop.cluster = 0 : i32, loop.stage = 2 : i32, num = 0 : i32}
680|   // CHECK:   %[[A_EXT:.*]] = ttg.memdesc_index %[[A]]{{\[}}%[[EXT_NEXT]]{{\]}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
681|   // CHECK:   %[[B_INS:.*]] = ttg.memdesc_index %[[B]]{{\[}}%[[INS_NEXT]]{{\]}} {loop.cluster = 2 : i32, loop.stage = 0 : i32}
682|   // CHECK:   %[[B_TOK:.*]] = ttg.async_copy_global_to_local %{{.*}}, %[[B_INS]] {contiguity = 8 : i32, loop.cluster = 2 : i32, loop.stage = 0 : i32}
683|   // CHECK:   %[[B_TOK2:.*]] = ttg.async_commit_group tokens %[[B_TOK]] {loop.cluster = 2 : i32, loop.stage = 0 : i32}
684|   // CHECK:   %[[B_TOK3:.*]] = ttg.async_wait %[[B_TOK2]] {loop.cluster = 0 : i32, loop.stage = 2 : i32, num = 0 : i32}
685|   // CHECK:   %[[B_EXT:.*]] = ttg.memdesc_index %[[B]]{{\[}}%[[EXT_NEXT]]{{\]}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
686|   // CHECK:   %[[BAR_SUB:.*]] = ttg.memdesc_index %[[BAR]]{{\[}}%[[BAR_IDX]]{{\]}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
687|   // CHECK:   %[[MMA_TOK:.*]] = ttng.tc_gen5_mma %[[A_EXT]], %[[B_EXT]], %{{.*}}[%[[TOK]]], {{.*}} {is_async, loop.cluster = 0 : i32, loop.stage = 2 : i32}
688|   // CHECK:   ttng.wait_barrier %[[BAR_SUB]], %[[PHASE]] deps %[[A_EXT]], %[[B_EXT]] {loop.cluster = 0 : i32, loop.stage = 3 : i32}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 689-702
```mlir
689|   // CHECK:   %[[PHASE_NEG:.*]] = arith.xori %[[PHASE]], %[[ONE]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
690|   // CHECK:   %[[BAR_IDX_P1:.*]] = arith.addi %[[BAR_IDX]], %[[ONE]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
691|   // CHECK:   %[[BAR_IDX_CMP:.*]] = arith.cmpi sge, %[[BAR_IDX_P1]], %[[TWO]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
692|   // CHECK:   %[[BAR_IDX_NEXT:.*]] = arith.select %[[BAR_IDX_CMP]], %[[ZERO]], %[[BAR_IDX_P1]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
693|   // CHECK:   %[[PHASE_NEXT:.*]] = arith.select %[[BAR_IDX_CMP]], %[[PHASE_NEG]], %[[PHASE]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
694|   // CHECK:   scf.yield %[[MMA_TOK]], %[[PHASE_NEXT]], %[[BAR_IDX_NEXT]], %[[INS_NEXT]], %[[EXT_NEXT]]
695|   // CHECK-DAG: ttg.local_dealloc %[[A]]
696|   // CHECK-DAG: ttg.local_dealloc %[[B]]
697|   // CHECK-DAG: %[[BAR_SUB1:.*]] = ttg.memdesc_index %[[BAR]]{{\[}}%[[ZERO]]{{\]}}
698|   // CHECK-DAG: ttng.inval_barrier %[[BAR_SUB1]]
699|   // CHECK-DAG: %[[BAR_SUB2:.*]] = ttg.memdesc_index %[[BAR]]{{\[}}%[[ONE]]{{\]}}
700|   // CHECK-DAG: ttng.inval_barrier %[[BAR_SUB2]]
701|   // CHECK-DAG: ttg.local_dealloc %[[BAR]]
702|   // CHECK-DAG: ttg.async_wait {num = 0 : i32}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 703-720
```mlir
703|   tt.func public @shmem_pipelining_mmav5(%lb : index, %ub : index, %step : index,
704|                                               %A_ptr: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
705|                                               %B_ptr: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>}) -> tensor<128x128xf16, #blocked> {
706|     %true = arith.constant true
707|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
708|     %c0_i32 = arith.constant 0 : i32
709|     %acc_tm, %acc_tok = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
710|     %init_tok = ttng.tmem_store %cst, %acc_tm[%acc_tok], %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
711|     %last_tok = scf.for %i = %lb to %ub step %step iter_args(%tok = %init_tok) -> !ttg.async.token : index {
712|       %A = tt.load %A_ptr {loop.cluster = 2 : i32, loop.stage = 0 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
713|       %B = tt.load %B_ptr {loop.cluster = 2 : i32, loop.stage = 0 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
714|       %A_sh = ttg.local_alloc %A {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
715|       %B_sh = ttg.local_alloc %B {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
716|       %mma_tok = ttng.tc_gen5_mma %A_sh, %B_sh, %acc_tm[%tok], %true, %true {loop.cluster = 0 : i32, loop.stage = 2 : i32, tt.self_latency = 1 : i32} : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
717|       scf.yield %mma_tok : !ttg.async.token
718|     } {tt.scheduled_max_stage = 3 : i32}
719|     %res, %res_tok = ttng.tmem_load %acc_tm[%last_tok] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
720|     %res_f16 = arith.truncf %res : tensor<128x128xf32, #blocked> to tensor<128x128xf16, #blocked>
```
**EN:** This function-oriented block defines or enters `shmem_pipelining_mmav5`. Within it, the test exercises constants, tt.func, masked or vectorized loads, shared/local memory allocation, tensor-memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `shmem_pipelining_mmav5` 为核心。测试在其中演示 常量、tt.func、带掩码或向量化的加载、共享/本地内存分配、张量内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 721-724
```mlir
721|     tt.return %res_f16 : tensor<128x128xf16, #blocked>
722|   }
723| }
724| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 725-725
```mlir
725| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 726-729
```mlir
726| 
727| #A = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
728| #nvmma_64 = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 16}>
729| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 730-730
```mlir
730| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 731-748
```mlir
731| // CHECK-LABEL: @tma_load_lowering
732| // CHECK-DAG: %[[TRUE:.*]] = arith.constant {{.*}} true
733| // CHECK-DAG: %[[MINUS_ONE:.*]] = arith.constant -1 : i32
734| // CHECK-DAG: %[[ZERO:.*]] = arith.constant 0 : i32
735| // CHECK-DAG: %[[ONE:.*]] = arith.constant 1 : i32
736| // CHECK-DAG: %[[NUM_BUFS:.*]] = arith.constant {{.*}} 2 : i32
737| // CHECK-DAG: %[[A:.*]] = ttg.local_alloc : () -> !ttg.memdesc<2x128x32
738| // CHECK-DAG: %[[BARRIER:.*]] = ttg.local_alloc : () -> !ttg.memdesc<2x1xi64
739| // CHECK: %[[BAR1_VIEW:.*]] = ttg.memdesc_index %[[BARRIER]]{{\[}}%[[ZERO]]{{\]}}
740| // CHECK: ttng.init_barrier %[[BAR1_VIEW]], 1
741| // CHECK: %[[BAR2_VIEW:.*]] = ttg.memdesc_index %[[BARRIER]]{{\[}}%[[ONE]]{{\]}}
742| // CHECK: ttng.init_barrier %[[BAR2_VIEW]], 1
743| // CHECK: scf.for {{.*}} iter_args(%[[INS:.*]] = %[[MINUS_ONE]], %[[EXT:.*]] = %[[MINUS_ONE]], %[[PHASE:.*]] = %[[ZERO]])
744| // CHECK:   %[[INS_P1:.*]] = arith.addi %[[INS]], %[[ONE]] {loop.cluster = 2 : i32, loop.stage = 0 : i32}
745| // CHECK:   %[[INS_CMP:.*]] = arith.cmpi sge, %[[INS_P1]], %[[NUM_BUFS]] {loop.cluster = 2 : i32, loop.stage = 0 : i32}
746| // CHECK:   %[[INS_NEXT:.*]] = arith.select %[[INS_CMP]], %[[ZERO]], %[[INS_P1]] {loop.cluster = 2 : i32, loop.stage = 0 : i32}
747| // CHECK:   %[[EXT_P1:.*]] = arith.addi %[[EXT]], %[[ONE]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
748| // CHECK:   %[[EXT_CMP:.*]] = arith.cmpi sge, %[[EXT_P1]], %[[NUM_BUFS]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tma_load_lowering anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tma_load_lowering 这样的标签用于锚定匹配范围。

### Lines 749-766
```mlir
749| // CHECK:   %[[EXT_NEXT:.*]] = arith.select %[[EXT_CMP]], %[[ZERO]], %[[EXT_P1]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
750| // CHECK:   %[[PHASE_XOR:.*]] = arith.xori %[[PHASE]], %[[ONE]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
751| // CHECK:   %[[PHASE_NEXT:.*]] = arith.select %[[EXT_CMP]], %[[PHASE_XOR]], %[[PHASE]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
752| // CHECK:   %[[BAR_INS:.*]] = ttg.memdesc_index %[[BARRIER]]{{\[}}%[[INS_NEXT]]{{\]}} {loop.cluster = 2 : i32, loop.stage = 0 : i32}
753| // CHECK:   ttng.barrier_expect %[[BAR_INS]], 8192 {loop.cluster = 2 : i32, loop.stage = 0 : i32}, %[[TRUE]]
754| // CHECK:   %[[A_INS:.*]] = ttg.memdesc_index %[[A]]{{\[}}%[[INS_NEXT]]{{\]}} {loop.cluster = 2 : i32, loop.stage = 0 : i32}
755| // CHECK:   ttng.async_tma_copy_global_to_local {{.*}}[{{.*}}] %[[A_INS]], %[[BAR_INS]], %[[TRUE]] {loop.cluster = 2 : i32, loop.stage = 0 : i32}
756| // CHECK:   %[[BAR_EXT:.*]] = ttg.memdesc_index %[[BARRIER]]{{\[}}%[[EXT_NEXT]]{{\]}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
757| // CHECK:   ttng.wait_barrier %[[BAR_EXT]], %[[PHASE_NEXT]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
758| // CHECK:   %[[A_EXT:.*]] = ttg.memdesc_index %[[A]]{{\[}}%[[EXT_NEXT]]{{\]}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
759| // CHECK:   %[[A_LOAD:.*]] = ttg.local_load %[[A_EXT]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
760| // CHECK:   "use"(%[[A_LOAD]]) {loop.cluster = 0 : i32, loop.stage = 2 : i32}
761| // CHECK:   scf.yield %[[INS_NEXT]], %[[EXT_NEXT]], %[[PHASE_NEXT]] : i32, i32, i32
762| // CHECK:  %[[BAR1_VIEW:.*]] = ttg.memdesc_index %[[BARRIER]]{{\[}}%[[ZERO]]{{\]}}
763| // CHECK:  ttng.inval_barrier %[[BAR1_VIEW]]
764| // CHECK:  %[[BAR2_VIEW:.*]] = ttg.memdesc_index %[[BARRIER]]{{\[}}%[[ONE]]{{\]}}
765| // CHECK:  ttng.inval_barrier %[[BAR2_VIEW]]
766| // CHECK:  ttg.local_dealloc %[[BARRIER]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 767-767
```mlir
767| // CHECK:  ttg.local_dealloc %[[A]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 768-778
```mlir
768| tt.func @tma_load_lowering(%lb : index, %ub : index, %step : index,
769|                  %desc : !tt.tensordesc<128x32xf16, #nvmma_64>,
770|                  %offs : i32) -> () {
771|   scf.for %iv = %lb to %ub step %step : index {
772|     %a = tt.descriptor_load %desc[%offs, %offs] {loop.cluster = 2 : i32, loop.stage = 0 : i32} : !tt.tensordesc<128x32xf16, #nvmma_64> -> tensor<128x32xf16, #A>
773|     "use"(%a) {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x32xf16, #A>) -> ()
774|   } {tt.scheduled_max_stage = 2 : i32}
775|   tt.return
776| }
777| }
778| 
```
**EN:** This function-oriented block defines or enters `tma_load_lowering`. Within it, the test exercises tt.func, structured loops, tt.descriptor_load, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tma_load_lowering` 为核心。测试在其中演示 tt.func、结构化循环、tt.descriptor_load、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 779-779
```mlir
779| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 780-785
```mlir
780| 
781| #A = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
782| #offsets_parent = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [1, 0]}>
783| #offsets = #ttg.slice<{dim = 0, parent = #offsets_parent}>
784| #nvmma_128 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
785| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 786-786
```mlir
786| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 787-804
```mlir
787| // CHECK-LABEL: @tma_gather_lowering
788| // CHECK-DAG: %[[TRUE:.*]] = arith.constant {{.*}} true
789| // CHECK-DAG: %[[MINUS_ONE:.*]] = arith.constant -1 : i32
790| // CHECK-DAG: %[[ZERO:.*]] = arith.constant 0 : i32
791| // CHECK-DAG: %[[ONE:.*]] = arith.constant 1 : i32
792| // CHECK-DAG: %[[NUM_BUFS:.*]] = arith.constant {{.*}} 2 : i32
793| // CHECK-DAG: %[[A:.*]] = ttg.local_alloc : () -> !ttg.memdesc<2x32x128
794| // CHECK-DAG: %[[BARRIER:.*]] = ttg.local_alloc : () -> !ttg.memdesc<2x1xi64
795| // CHECK: %[[BAR1_VIEW:.*]] = ttg.memdesc_index %[[BARRIER]]{{\[}}%[[ZERO]]{{\]}}
796| // CHECK: ttng.init_barrier %[[BAR1_VIEW]], 1
797| // CHECK: %[[BAR2_VIEW:.*]] = ttg.memdesc_index %[[BARRIER]]{{\[}}%[[ONE]]{{\]}}
798| // CHECK: ttng.init_barrier %[[BAR2_VIEW]], 1
799| // CHECK: scf.for {{.*}} iter_args(%[[INS:.*]] = %[[MINUS_ONE]], %[[EXT:.*]] = %[[MINUS_ONE]], %[[PHASE:.*]] = %[[ZERO]])
800| // CHECK:   %[[INS_P1:.*]] = arith.addi %[[INS]], %[[ONE]] {loop.cluster = 2 : i32, loop.stage = 0 : i32}
801| // CHECK:   %[[INS_CMP:.*]] = arith.cmpi sge, %[[INS_P1]], %[[NUM_BUFS]] {loop.cluster = 2 : i32, loop.stage = 0 : i32}
802| // CHECK:   %[[INS_NEXT:.*]] = arith.select %[[INS_CMP]], %[[ZERO]], %[[INS_P1]] {loop.cluster = 2 : i32, loop.stage = 0 : i32}
803| // CHECK:   %[[EXT_P1:.*]] = arith.addi %[[EXT]], %[[ONE]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
804| // CHECK:   %[[EXT_CMP:.*]] = arith.cmpi sge, %[[EXT_P1]], %[[NUM_BUFS]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tma_gather_lowering anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tma_gather_lowering 这样的标签用于锚定匹配范围。

### Lines 805-822
```mlir
805| // CHECK:   %[[EXT_NEXT:.*]] = arith.select %[[EXT_CMP]], %[[ZERO]], %[[EXT_P1]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
806| // CHECK:   %[[PHASE_XOR:.*]] = arith.xori %[[PHASE]], %[[ONE]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
807| // CHECK:   %[[PHASE_NEXT:.*]] = arith.select %[[EXT_CMP]], %[[PHASE_XOR]], %[[PHASE]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
808| // CHECK:   %[[BAR_INS:.*]] = ttg.memdesc_index %[[BARRIER]]{{\[}}%[[INS_NEXT]]{{\]}} {loop.cluster = 2 : i32, loop.stage = 0 : i32}
809| // CHECK:   ttng.barrier_expect %[[BAR_INS]], 16384 {loop.cluster = 2 : i32, loop.stage = 0 : i32}, %[[TRUE]]
810| // CHECK:   %[[A_INS:.*]] = ttg.memdesc_index %[[A]]{{\[}}%[[INS_NEXT]]{{\]}} {loop.cluster = 2 : i32, loop.stage = 0 : i32}
811| // CHECK:   ttng.async_tma_gather {{.*}}[{{.*}}] %[[A_INS]], %[[BAR_INS]], %[[TRUE]] {loop.cluster = 2 : i32, loop.stage = 0 : i32}
812| // CHECK:   %[[BAR_EXT:.*]] = ttg.memdesc_index %[[BARRIER]]{{\[}}%[[EXT_NEXT]]{{\]}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
813| // CHECK:   ttng.wait_barrier %[[BAR_EXT]], %[[PHASE_NEXT]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
814| // CHECK:   %[[A_EXT:.*]] = ttg.memdesc_index %[[A]]{{\[}}%[[EXT_NEXT]]{{\]}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
815| // CHECK:   %[[A_LOAD:.*]] = ttg.local_load %[[A_EXT]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
816| // CHECK:   "use"(%[[A_LOAD]]) {loop.cluster = 0 : i32, loop.stage = 2 : i32}
817| // CHECK:   scf.yield %[[INS_NEXT]], %[[EXT_NEXT]], %[[PHASE_NEXT]] : i32, i32, i32
818| // CHECK:  %[[BAR1_VIEW:.*]] = ttg.memdesc_index %[[BARRIER]]{{\[}}%[[ZERO]]{{\]}}
819| // CHECK:  ttng.inval_barrier %[[BAR1_VIEW]]
820| // CHECK:  %[[BAR2_VIEW:.*]] = ttg.memdesc_index %[[BARRIER]]{{\[}}%[[ONE]]{{\]}}
821| // CHECK:  ttng.inval_barrier %[[BAR2_VIEW]]
822| // CHECK-DAG: ttg.local_dealloc %[[BARRIER]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 823-823
```mlir
823| // CHECK-DAG: ttg.local_dealloc %[[A]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 824-835
```mlir
824| tt.func @tma_gather_lowering(%lb : index, %ub : index, %step : index,
825|                  %desc : !tt.tensordesc<1x128xf32, #nvmma_128>,
826|                  %x : tensor<32xi32, #offsets>,
827|                  %y : i32) -> () {
828|   scf.for %iv = %lb to %ub step %step : index {
829|     %a = tt.descriptor_gather %desc[%x, %y] {loop.cluster = 2 : i32, loop.stage = 0 : i32} : (!tt.tensordesc<1x128xf32, #nvmma_128>, tensor<32xi32, #offsets>, i32) -> tensor<32x128xf32, #A>
830|     "use"(%a) {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<32x128xf32, #A>) -> ()
831|   } {tt.scheduled_max_stage = 2 : i32}
832|   tt.return
833| }
834| }
835| 
```
**EN:** This function-oriented block defines or enters `tma_gather_lowering`. Within it, the test exercises tt.func, structured loops, tt.descriptor_gather, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tma_gather_lowering` 为核心。测试在其中演示 tt.func、结构化循环、tt.descriptor_gather、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 836-836
```mlir
836| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 837-840
```mlir
837| 
838| #A = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
839| #nvmma_64 = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 16}>
840| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 841-841
```mlir
841| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 842-854
```mlir
842| // CHECK-LABEL: @tma_reuse_barrier
843| // CHECK: scf.for
844| // CHECK:   ttng.barrier_expect {{.*}}, 16384
845| // CHECK:   ttng.async_tma_copy_global_to_local
846| // CHECK-NOT: ttng.wait_barrier
847| // CHECK:   ttng.async_tma_copy_global_to_local
848| // CHECK:   ttng.wait_barrier
849| // CHECK:   "use1"
850| // CHECK:   "use2"
851| // CHECK:   ttng.barrier_expect {{.*}}, 8192
852| // CHECK:   ttng.async_tma_copy_global_to_local
853| // CHECK:   ttng.wait_barrier
854| // CHECK:   "use3"
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tma_reuse_barrier anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tma_reuse_barrier 这样的标签用于锚定匹配范围。

### Lines 855-871
```mlir
855| tt.func @tma_reuse_barrier(%lb : index, %ub : index, %step : index,
856|                  %descA : !tt.tensordesc<128x32xf16, #nvmma_64>,
857|                  %descB : !tt.tensordesc<128x32xf16, #nvmma_64>,
858|                  %descC : !tt.tensordesc<128x32xf16, #nvmma_64>,
859|                  %offs : i32) -> () {
860|   scf.for %iv = %lb to %ub step %step : index {
861|     %a = tt.descriptor_load %descA[%offs, %offs] {loop.cluster = 2 : i32, loop.stage = 0 : i32} : !tt.tensordesc<128x32xf16, #nvmma_64> -> tensor<128x32xf16, #A>
862|     %b = tt.descriptor_load %descB[%offs, %offs] {loop.cluster = 2 : i32, loop.stage = 0 : i32} : !tt.tensordesc<128x32xf16, #nvmma_64> -> tensor<128x32xf16, #A>
863|     "use1"(%a) {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x32xf16, #A>) -> ()
864|     "use2"(%b) {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x32xf16, #A>) -> ()
865|     %c = tt.descriptor_load %descC[%offs, %offs] {loop.cluster = 2 : i32, loop.stage = 0 : i32} : !tt.tensordesc<128x32xf16, #nvmma_64> -> tensor<128x32xf16, #A>
866|     "use3"(%c) {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x32xf16, #A>) -> ()
867|   } {tt.scheduled_max_stage = 2 : i32}
868|   tt.return
869| }
870| }
871| 
```
**EN:** This function-oriented block defines or enters `tma_reuse_barrier`. Within it, the test exercises tt.descriptor_load, tt.func, structured loops, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tma_reuse_barrier` 为核心。测试在其中演示 tt.descriptor_load、tt.func、结构化循环、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 872-872
```mlir
872| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 873-877
```mlir
873| 
874| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 16, 16]}>
875| #blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
876| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
877| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 878-878
```mlir
878| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 879-890
```mlir
879|   // CHECK-LABEL: @tma_pipelining_mmav3
880|   // CHECK: ttg.local_alloc : () -> !ttg.memdesc<3x128x128
881|   // CHECK: ttg.local_alloc : () -> !ttg.memdesc<3x128x128
882|   // CHECK: ttg.local_alloc : () -> !ttg.memdesc<3x1xi64
883|   // CHECK: scf.for
884|   // CHECK:   ttng.barrier_expect
885|   // CHECK:   ttng.async_tma_copy_global_to_local
886|   // CHECK-NOT: ttng.wait_barrier
887|   // CHECK:   ttng.async_tma_copy_global_to_local
888|   // CHECK:   ttng.wait_barrier
889|   // CHECK-NOT: ttg.local_alloc
890|   // CHECK:   ttng.warp_group_dot
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tma_pipelining_mmav3 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tma_pipelining_mmav3 这样的标签用于锚定匹配范围。

### Lines 891-908
```mlir
891|   tt.func public @tma_pipelining_mmav3(%lb : index, %ub : index, %step : index,
892|                                               %descA : !tt.tensordesc<128x128xf16, #shared>,
893|                                               %descB : !tt.tensordesc<128x128xf16, #shared>,
894|                                               %offs : i32) -> tensor<128x128xf16, #mma> {
895|     %true = arith.constant true
896|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #mma>
897|     %c0_i32 = arith.constant 0 : i32
898|     %res = scf.for %i = %lb to %ub step %step iter_args(%acc = %cst) -> (tensor<128x128xf32, #mma>) : index {
899|       %A = tt.descriptor_load %descA[%offs, %offs] {loop.cluster = 2 : i32, loop.stage = 0 : i32} : !tt.tensordesc<128x128xf16, #shared> -> tensor<128x128xf16, #blocked1>
900|       %A_sh = ttg.local_alloc %A {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
901|       %B = tt.descriptor_load %descB[%offs, %offs] {loop.cluster = 2 : i32, loop.stage = 0 : i32} : !tt.tensordesc<128x128xf16, #shared> -> tensor<128x128xf16, #blocked1>
902|       %B_sh = ttg.local_alloc %B {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>
903|       %acc_res = ttng.warp_group_dot %A_sh, %B_sh, %acc {loop.cluster = 0 : i32, loop.stage = 2 : i32} : !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory> * !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory> -> tensor<128x128xf32, #mma>
904|       scf.yield %acc_res : tensor<128x128xf32, #mma>
905|     } {tt.scheduled_max_stage = 2 : i32}
906|     %res_f16 = arith.truncf %res : tensor<128x128xf32, #mma> to tensor<128x128xf16, #mma>
907|     tt.return %res_f16 : tensor<128x128xf16, #mma>
908|   }
```
**EN:** This function-oriented block defines or enters `tma_pipelining_mmav3`. Within it, the test exercises constants, tt.func, tt.descriptor_load, shared/local memory allocation, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tma_pipelining_mmav3` 为核心。测试在其中演示 常量、tt.func、tt.descriptor_load、共享/本地内存分配、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 909-910
```mlir
909| }
910| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 911-911
```mlir
911| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 912-913
```mlir
912| #nvmma_128 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
913| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 914-914
```mlir
914| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 915-929
```mlir
915|   // CHECK-LABEL: @tensor_descriptor_lowering
916|   // CHECK-DAG: %[[ZERO:.*]] = arith.constant 0 : i32
917|   // CHECK-DAG: %[[ONE:.*]] = arith.constant 1 : i32
918|   // CHECK-DAG: %[[_128:.*]] = arith.constant{{.*}} 128 : i32
919|   // CHECK: %[[GLOBAL_ALLOC:.*]] = ttg.global_scratch_alloc {alignment = 128 : i32, nbytes = 128 : i32} : !tt.ptr<i8>
920|   // CHECK: scf.for {{.*}} iter_args(%[[IDX:.*]] = %[[ZERO]])
921|   // CHECK:   %[[OFFS:.*]] = arith.muli %[[IDX]], %[[_128]] {loop.cluster = 0 : i32, loop.stage = 1 : i32}
922|   // CHECK:   %[[DESC_PTR:.*]] = tt.addptr %[[GLOBAL_ALLOC]], %[[OFFS]] {loop.cluster = 0 : i32, loop.stage = 1 : i32}
923|   // CHECK:   ttng.tensormap_create %[[DESC_PTR]]{{.*}} loop.cluster = 0 : i32, loop.stage = 1 : i32
924|   // CHECK:   ttng.tensormap_fenceproxy_acquire %[[DESC_PTR]] {loop.cluster = 0 : i32, loop.stage = 1 : i32}
925|   // CHECK:   %[[DESC:.*]] = ttng.reinterpret_tensor_descriptor %[[DESC_PTR]] {loop.cluster = 0 : i32, loop.stage = 1 : i32}
926|   // CHECK:   %[[IDX_P1:.*]] = arith.addi %[[IDX]], %[[ONE]] {loop.cluster = 0 : i32, loop.stage = 1 : i32}
927|   // CHECK:   %[[IDX_CMP:.*]] = arith.cmpi sge, %[[IDX_P1]], %[[ONE]] {loop.cluster = 0 : i32, loop.stage = 1 : i32}
928|   // CHECK:   %[[IDX_NEXT:.*]] = arith.select %[[IDX_CMP]], %[[ZERO]], %[[IDX_P1]] {loop.cluster = 0 : i32, loop.stage = 1 : i32}
929|   // CHECK:   "use"(%[[DESC]]) {loop.cluster = 0 : i32, loop.stage = 1 : i32}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tensor_descriptor_lowering anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tensor_descriptor_lowering 这样的标签用于锚定匹配范围。

### Lines 930-944
```mlir
930|   tt.func @tensor_descriptor_lowering(
931|     %lb : index, %ub : index, %step : index,
932|     %A: !tt.ptr<f16>,
933|     %shape_x: i32,
934|     %shape_y: i32,
935|     %strides_x: i64,
936|     %strides_y: i64) -> (){
937|     scf.for %iv = %lb to %ub step %step : index {
938|       %desc = tt.make_tensor_descriptor %A, [%shape_x, %shape_y], [%strides_x, %strides_y] {loop.cluster = 0 : i32, loop.stage = 1 : i32} : <f16>, <128x128xf16, #nvmma_128>
939|       "use"(%desc) {loop.cluster = 0 : i32, loop.stage = 1 : i32} : (!tt.tensordesc<128x128xf16, #nvmma_128>) -> ()
940|     } {tt.scheduled_max_stage = 1 : i32}
941|     tt.return
942|   }
943| }
944| 
```
**EN:** This function-oriented block defines or enters `tensor_descriptor_lowering`. Within it, the test exercises tt.func, structured loops, tt.make_tensor_descriptor, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tensor_descriptor_lowering` 为核心。测试在其中演示 tt.func、结构化循环、tt.make_tensor_descriptor、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 945-945
```mlir
945| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 946-954
```mlir
946| 
947| #blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
948| #blocked2 = #ttg.blocked<{sizePerThread = [1, 1, 1, 1, 4], threadsPerWarp = [1, 1, 8, 4, 1], warpsPerCTA = [1, 1, 4, 1, 1], order = [4, 3, 2, 1, 0]}>
949| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
950| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [4, 3, 2, 1, 0]}>
951| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
952| #tmem_scales = #ttng.tensor_memory_scales_encoding<>
953| #smem = #ttg.shared_memory
954| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 955-955
```mlir
955| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 956-960
```mlir
956|   // CHECK-LABEL: @pipelining_mmav5_scaled
957|   // CHECK: ttg.local_alloc : () -> !ttg.memdesc<3x128x128xf8E5M2
958|   // CHECK: ttg.local_alloc : () -> !ttg.memdesc<3x128x128xf8E5M2
959|   // CHECK: ttg.local_alloc : () -> !ttg.memdesc<3x1x2x32x4x4xi8
960|   // CHECK: ttg.local_alloc : () -> !ttg.memdesc<3x1x2x32x4x4xi8
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @pipelining_mmav5_scaled anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @pipelining_mmav5_scaled 这样的标签用于锚定匹配范围。

### Lines 961-978
```mlir
961|   tt.func public @pipelining_mmav5_scaled(%lb : index, %ub : index, %step : index,
962|                                               %A_ptr: tensor<128x128x!tt.ptr<f8E5M2>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
963|                                               %B_ptr: tensor<128x128x!tt.ptr<f8E5M2>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
964|                                               %A_sc_ptr: tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked2> {tt.divisibility = dense<[16, 16, 16, 16, 16]> : tensor<5xi32>, tt.contiguity = dense<[1, 1, 1, 1, 16]> : tensor<5xi32>},
965|                                               %B_sc_ptr: tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked2> {tt.divisibility = dense<[16, 16, 16, 16, 16]> : tensor<5xi32>, tt.contiguity = dense<[1, 1, 1, 1, 16]> : tensor<5xi32>}) -> tensor<128x128xf32, #blocked1> {
966|     %true = arith.constant true
967|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked1>
968|     %c0_i32 = arith.constant 0 : i32
969|     %acc_tm, %acc_tok = ttng.tmem_alloc %cst {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x128xf32, #blocked1>) -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
970|     %last_tok = scf.for %i = %lb to %ub step %step iter_args(%tok = %acc_tok) -> !ttg.async.token : index {
971|       %A = tt.load %A_ptr  {loop.cluster = 2 : i32, loop.stage = 0 : i32} : tensor<128x128x!tt.ptr<f8E5M2>, #blocked1>
972|       %B = tt.load %B_ptr  {loop.cluster = 2 : i32, loop.stage = 0 : i32} : tensor<128x128x!tt.ptr<f8E5M2>, #blocked1>
973|       %A_sh = ttg.local_alloc %A {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x128xf8E5M2, #blocked1>) -> !ttg.memdesc<128x128xf8E5M2, #shared, #ttg.shared_memory>
974|       %B_sh = ttg.local_alloc %B {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x128xf8E5M2, #blocked1>) -> !ttg.memdesc<128x128xf8E5M2, #shared, #ttg.shared_memory>
975| 
976|       %A_sc = tt.load %A_sc_ptr {loop.cluster = 2 : i32, loop.stage = 0 : i32} : tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked2>
977|       %A_sc_sh = ttg.local_alloc %A_sc {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<1x2x32x4x4xi8, #blocked2>) -> !ttg.memdesc<1x2x32x4x4xi8, #shared1, #smem>
978| 
```
**EN:** This function-oriented block defines or enters `pipelining_mmav5_scaled`. Within it, the test exercises constants, masked or vectorized loads, shared/local memory allocation, tt.func, tensor-memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `pipelining_mmav5_scaled` 为核心。测试在其中演示 常量、带掩码或向量化的加载、共享/本地内存分配、tt.func、张量内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 979-989
```mlir
979|       %B_sc = tt.load %B_sc_ptr {loop.cluster = 2 : i32, loop.stage = 0 : i32} : tensor<1x2x32x4x4x!tt.ptr<i8>, #blocked2>
980|       %B_sc_sh = ttg.local_alloc %B_sc {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<1x2x32x4x4xi8, #blocked2>) -> !ttg.memdesc<1x2x32x4x4xi8, #shared1, #smem>
981| 
982|       %mma_tok = ttng.tc_gen5_mma_scaled %A_sh, %B_sh, %acc_tm[%tok], %A_sc_sh, %B_sc_sh, %true, %true lhs = e5m2 rhs = e5m2 {loop.cluster = 0 : i32, loop.stage = 2 : i32, tt.self_latency = 1 : i32} : !ttg.memdesc<128x128xf8E5M2, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf8E5M2, #shared, #ttg.shared_memory>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.memdesc<1x2x32x4x4xi8, #shared1, #smem>, !ttg.memdesc<1x2x32x4x4xi8, #shared1, #smem>
983|       scf.yield %mma_tok : !ttg.async.token
984|     } {tt.scheduled_max_stage = 3 : i32}
985|     %res, %res_tok = ttng.tmem_load %acc_tm[%last_tok] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
986|     tt.return %res : tensor<128x128xf32, #blocked1>
987|   }
988| }
989| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining masked or vectorized loads, shared/local memory allocation, scaled tensor-core MMA ops, loop/if yielded values, ttng.tmem_load.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 带掩码或向量化的加载、共享/本地内存分配、带缩放的张量核 MMA 操作、循环/分支产出值、ttng.tmem_load。

### Lines 990-990
```mlir
990| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 991-996
```mlir
991| 
992| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
993| #blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
994| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
995| #smem = #ttg.shared_memory
996| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 997-997
```mlir
997| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 998-998
```mlir
998|   // CHECK-LABEL: @cnd_store_before_mma
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @cnd_store_before_mma anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @cnd_store_before_mma 这样的标签用于锚定匹配范围。

### Lines 999-1007
```mlir
 999|   tt.func public @cnd_store_before_mma(%arg0: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>}, %arg1: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>}, %arg2: i32) -> tensor<128x128xf16, #blocked1> {
1000|     %true = arith.constant true
1001|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked1>
1002|     %cst_0 = arith.constant dense<2.000000e+00> : tensor<128x128xf32, #blocked1>
1003|     %c0_i32 = arith.constant 0 : i32
1004|     %c1_i32 = arith.constant 1 : i32
1005|     %0 = "cnd"() : () -> i1
1006|     %1, %acc_tok = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
1007|     // Do not multibuffer tmem, as all the tmem uses are in the same stage.
```
**EN:** This function-oriented block defines or enters `cnd_store_before_mma`. Within it, the test exercises constants, tt.func, tensor-memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `cnd_store_before_mma` 为核心。测试在其中演示 常量、tt.func、张量内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1008-1025
```mlir
1008|     // CHECK: %[[ACC_TM:.*]], %[[ACC_TOK:.*]] = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32
1009|     %init_tok = ttng.tmem_store %cst, %1[%acc_tok], %true : tensor<128x128xf32, #blocked1> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1010|     %last_tok = scf.for %arg3 = %c0_i32 to %arg2 step %c1_i32 iter_args(%tok = %init_tok) -> !ttg.async.token : i32 {
1011|       %4 = arith.xori %0, %true {loop.cluster = 0 : i32, loop.stage = 2 : i32} : i1
1012|       %store_tok = ttng.tmem_store %cst_0, %1[%tok], %4 {loop.cluster = 0 : i32, loop.stage = 2 : i32} : tensor<128x128xf32, #blocked1> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1013|       %5 = tt.load %arg0 {loop.cluster = 2 : i32, loop.stage = 0 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
1014|       %6 = ttg.local_alloc %5 {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
1015|       %7 = tt.load %arg1 {loop.cluster = 2 : i32, loop.stage = 0 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
1016|       %8 = ttg.local_alloc %7 {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
1017|       %mma_tok = ttng.tc_gen5_mma %6, %8, %1[%store_tok], %true, %true {loop.cluster = 0 : i32, loop.stage = 2 : i32} : !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1018|       scf.yield %mma_tok : !ttg.async.token
1019|     } {tt.scheduled_max_stage = 2 : i32}
1020|     %2, %load_tok = ttng.tmem_load %1[%last_tok] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
1021|     %3 = arith.truncf %2 : tensor<128x128xf32, #blocked1> to tensor<128x128xf16, #blocked1>
1022|     tt.return %3 : tensor<128x128xf16, #blocked1>
1023|   }
1024| }
1025| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining ttng.tmem_store, masked or vectorized loads, shared/local memory allocation, structured loops, arith.xori.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 ttng.tmem_store、带掩码或向量化的加载、共享/本地内存分配、结构化循环、arith.xori。

### Lines 1026-1026
```mlir
1026| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1027-1032
```mlir
1027| 
1028| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
1029| #blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
1030| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
1031| #smem = #ttg.shared_memory
1032| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1033-1033
```mlir
1033| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 1034-1051
```mlir
1034|   // CHECK-LABEL: @simple_persistent_mmav5
1035|   // CHECK-DAG: %[[TRUE:.*]] = arith.constant true
1036|   // CHECK-DAG: %[[INIT_ACC:.*]] = "init_acc"()
1037|   // CHECK-DAG: %[[OVERRIDE_ACC:.*]] = "override_acc"()
1038|   // CHECK-DAG: %[[CND:.*]] = "cnd"()
1039|   // CHECK-DAG: %[[C_N1:.*]] = arith.constant -1 : i32
1040|   // CHECK-DAG: %[[C_0:.*]] = arith.constant 0 : i32
1041|   // CHECK-DAG: %[[C_1:.*]] = arith.constant 1 : i32
1042|   // CHECK-DAG: %[[C_2:.*]] = arith.constant 2 : i32
1043|   // CHECK: %[[ACC_TM:.*]], %[[ACC_TOK:.*]] = ttng.tmem_alloc  : () -> (!ttg.memdesc<2x128x128xf32
1044|   // CHECK: %[[ACC_TM_SLICE:.*]] = ttg.memdesc_index %[[ACC_TM]]{{\[}}%[[C_0]]{{\]}}
1045|   // CHECK: %[[INIT_TOK:.*]] = ttng.tmem_store %[[INIT_ACC]], %[[ACC_TM_SLICE]][], %[[TRUE]]
1046|   // CHECK: %[[BAR:.*]] = ttg.local_alloc  : () -> !ttg.memdesc<2x1xi64
1047|   // CHECK: %[[BAR_SLICE:.*]] = ttg.memdesc_index %[[BAR]]{{\[}}%[[C_0]]{{\]}}
1048|   // CHECK: ttng.init_barrier %[[BAR_SLICE]], 1
1049|   // CHECK: %[[BAR_SLICE_2:.*]] = ttg.memdesc_index %[[BAR]]{{\[}}%[[C_1]]{{\]}}
1050|   // CHECK: ttng.init_barrier %[[BAR_SLICE_2]], 1
1051|   // CHECK: %[[FOR_RES:.*]]:5 = scf.for {{.*}} iter_args(%[[PHASE:.*]] = %[[C_0]], %[[BAR_IDX:.*]] = %[[C_0]], %[[BUF_IDX:.*]] = %[[C_N1]], %[[INSERT_IDX:.*]] = %[[C_N1]], %[[EXTRACT_IDX:.*]] = %[[C_N1]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @simple_persistent_mmav5 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @simple_persistent_mmav5 这样的标签用于锚定匹配范围。

### Lines 1052-1069
```mlir
1052|   // CHECK:   %[[BUF_IDX_P1:.*]] = arith.addi %[[BUF_IDX]], %[[C_1]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
1053|   // CHECK:   %[[BUF_IDX_CND:.*]] = arith.cmpi sge, %[[BUF_IDX_P1]], %[[C_2]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
1054|   // CHECK:   %[[BUF_IDX_NEXT:.*]] = arith.select %[[BUF_IDX_CND]], %[[C_0]], %[[BUF_IDX_P1]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
1055|   // CHECK:   %[[BUF_IDX_NEXT_CND:.*]] = arith.select %[[CND]], %[[BUF_IDX]], %[[BUF_IDX_NEXT]]
1056|   // CHECK:   %[[TM_SLICE:.*]] = ttg.memdesc_index %[[ACC_TM]]{{\[}}%[[BUF_IDX_NEXT_CND]]{{\]}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
1057|   // CHECK:   %[[STORE_TOK:.*]] = ttng.tmem_store %[[OVERRIDE_ACC]], %[[TM_SLICE]][], {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
1058|   // CHECK:   %[[BAR_SLICE:.*]] = ttg.memdesc_index %[[BAR]]{{\[}}%[[BAR_IDX]]{{\]}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
1059|   // CHECK:   %[[ACC_TM_SLICE:.*]] = ttg.memdesc_index %[[ACC_TM]]{{\[}}%[[BUF_IDX_NEXT_CND]]{{\]}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
1060|   // CHECK:   %[[MMA_TOK:.*]] = ttng.tc_gen5_mma %{{.*}}, %{{.*}}, %[[ACC_TM_SLICE]][], %[[TRUE]], %[[TRUE]], %[[BAR_SLICE]][%true] {is_async, loop.cluster = 0 : i32, loop.stage = 2 : i32}
1061|   // CHECK:   ttng.wait_barrier %[[BAR_SLICE]], %[[PHASE]] deps %{{.*}}, %{{.*}} {loop.cluster = 0 : i32, loop.stage = 3 : i32}
1062|   // CHECK:   scf.if
1063|   // CHECK:     %[[ACC_TM_SLICE:.*]] = ttg.memdesc_index %[[ACC_TM]]{{\[}}%[[BUF_IDX_NEXT_CND]]{{\]}}
1064|   // CHECK:     %[[LOAD_ACC:.*]], %[[USER_TOK:.*]] = ttng.tmem_load %[[ACC_TM_SLICE]][]
1065|   // CHECK:     "use"(%[[LOAD_ACC]])
1066|   // CHECK:   }
1067|   // CHECK:   %[[PHASE_NEG:.*]] = arith.xori %[[PHASE]], %[[C_1]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
1068|   // CHECK:   %[[BAR_IDX_P1:.*]] = arith.addi %[[BAR_IDX]], %[[C_1]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
1069|   // CHECK:   %[[BAR_IDX_CND:.*]] = arith.cmpi sge, %[[BAR_IDX_P1]], %[[C_2]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 1070-1075
```mlir
1070|   // CHECK:   %[[BAR_IDX_NEXT:.*]] = arith.select %[[BAR_IDX_CND]], %[[C_0]], %[[BAR_IDX_P1]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
1071|   // CHECK:   %[[PHASE_NEXT:.*]] = arith.select %[[BAR_IDX_CND]], %[[PHASE_NEG]], %[[PHASE]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
1072|   // CHECK:   scf.yield %[[PHASE_NEXT]], %[[BAR_IDX_NEXT]], %[[BUF_IDX_NEXT_CND]]
1073|   // CHECK: } {tt.scheduled_max_stage = 3 : i32}
1074|   // CHECK: %[[ACC_TM_SLICE:.*]] = ttg.memdesc_index %[[ACC_TM]]{{\[}}%[[FOR_RES]]#2{{\]}}
1075|   // CHECK: %[[LOAD_ACC:.*]], %[[RES_TOK:.*]] = ttng.tmem_load %[[ACC_TM_SLICE]][]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 1076-1093
```mlir
1076|   tt.func public @simple_persistent_mmav5(%arg0: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>}, %arg1: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>}, %arg2: i32) -> tensor<128x128xf16, #blocked1> {
1077|     %true = arith.constant true
1078|     %cst = "init_acc"() : () -> tensor<128x128xf32, #blocked1>
1079|     %cst_0 = "override_acc"() : () -> tensor<128x128xf32, #blocked1>
1080|     %c0_i32 = arith.constant 0 : i32
1081|     %c1_i32 = arith.constant 1 : i32
1082|     %0 = "cnd"() : () -> i1
1083|     %1, %acc_tok = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
1084|     %init_tok = ttng.tmem_store %cst, %1[%acc_tok], %true : tensor<128x128xf32, #blocked1> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1085|     %last_tok = scf.for %arg3 = %c0_i32 to %arg2 step %c1_i32 iter_args(%tok = %init_tok) -> !ttg.async.token : i32 {
1086|       %4 = arith.xori %0, %true {loop.cluster = 0 : i32, loop.stage = 2 : i32} : i1
1087|       %store_tok = ttng.tmem_store %cst_0, %1[%tok], %4 {loop.cluster = 0 : i32, loop.stage = 2 : i32} : tensor<128x128xf32, #blocked1> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1088|       %5 = tt.load %arg0 {loop.cluster = 2 : i32, loop.stage = 0 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
1089|       %6 = ttg.local_alloc %5 {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
1090|       %7 = tt.load %arg1 {loop.cluster = 2 : i32, loop.stage = 0 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
1091|       %8 = ttg.local_alloc %7 {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
1092|       %mma_tok = ttng.tc_gen5_mma %6, %8, %1[%store_tok], %true, %true {loop.cluster = 0 : i32, loop.stage = 2 : i32, tt.self_latency = 1 : i32} : !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1093|       %cnd_tok = scf.if %0 -> !ttg.async.token {
```
**EN:** This function-oriented block defines or enters `simple_persistent_mmav5`. Within it, the test exercises constants, tt.func, ttng.tmem_store, masked or vectorized loads, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `simple_persistent_mmav5` 为核心。测试在其中演示 常量、tt.func、ttng.tmem_store、带掩码或向量化的加载、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1094-1107
```mlir
1094|         %9, %user_tok = ttng.tmem_load %1[%mma_tok] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
1095|         "use"(%9) : (tensor<128x128xf32, #blocked1>) -> ()
1096|         scf.yield %user_tok : !ttg.async.token
1097|       } else {
1098|         scf.yield %mma_tok : !ttg.async.token
1099|       } {loop.cluster = 3 : i32, loop.stage = 3 : i32}
1100|       scf.yield %cnd_tok : !ttg.async.token
1101|     } {tt.scheduled_max_stage = 2 : i32}
1102|     %2, %res_tok = ttng.tmem_load %1[%last_tok] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
1103|     %3 = arith.truncf %2 : tensor<128x128xf32, #blocked1> to tensor<128x128xf16, #blocked1>
1104|     tt.return %3 : tensor<128x128xf16, #blocked1>
1105|   }
1106| }
1107| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, ttng.tmem_load, arith.truncf, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、ttng.tmem_load、arith.truncf、tt.return。

### Lines 1108-1108
```mlir
1108| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1109-1114
```mlir
1109| 
1110| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
1111| #blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
1112| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
1113| #smem = #ttg.shared_memory
1114| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1115-1115
```mlir
1115| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 1116-1133
```mlir
1116|   // CHECK-LABEL: @simple_persistent_mmav5_acc_flag
1117|   // CHECK-DAG: %[[TRUE:.*]] = arith.constant true
1118|   // CHECK-DAG: %[[INIT_ACC:.*]] = "init_acc"()
1119|   // CHECK-DAG: %[[OVERRIDE_ACC:.*]] = "override_acc"()
1120|   // CHECK-DAG: %[[CND:.*]] = "cnd"()
1121|   // CHECK-DAG: %[[C_N1:.*]] = arith.constant -1 : i32
1122|   // CHECK-DAG: %[[C_0:.*]] = arith.constant 0 : i32
1123|   // CHECK-DAG: %[[C_1:.*]] = arith.constant 1 : i32
1124|   // CHECK-DAG: %[[C_2:.*]] = arith.constant 2 : i32
1125|   // CHECK: %[[ACC_TM:.*]], %[[ACC_TOK:.*]] = ttng.tmem_alloc : () -> (!ttg.memdesc<2x128x128xf32
1126|   // CHECK: %[[ACC_TM_SLICE:.*]] = ttg.memdesc_index %[[ACC_TM]]{{\[}}%[[C_0]]{{\]}}
1127|   // CHECK: %[[INIT_TOK:.*]] = ttng.tmem_store %[[INIT_ACC]], %[[ACC_TM_SLICE]][], %[[TRUE]]
1128|   // CHECK: %[[BAR:.*]] = ttg.local_alloc  : () -> !ttg.memdesc<2x1xi64
1129|   // CHECK: %[[BAR_SLICE:.*]] = ttg.memdesc_index %[[BAR]]{{\[}}%[[C_0]]{{\]}}
1130|   // CHECK: ttng.init_barrier %[[BAR_SLICE]], 1
1131|   // CHECK: %[[BAR_SLICE_2:.*]] = ttg.memdesc_index %[[BAR]]{{\[}}%[[C_1]]{{\]}}
1132|   // CHECK: ttng.init_barrier %[[BAR_SLICE_2]], 1
1133|   // CHECK: %[[FOR_RES:.*]]:5 = scf.for {{.*}} iter_args(%[[PHASE:.*]] = %[[C_0]], %[[BAR_IDX:.*]] = %[[C_0]], %[[BUF_IDX:.*]] = %[[C_N1]], %[[INSERT_IDX:.*]] = %[[C_N1]], %[[EXTRACT_IDX:.*]] = %[[C_N1]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @simple_persistent_mmav5_acc_flag anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @simple_persistent_mmav5_acc_flag 这样的标签用于锚定匹配范围。

### Lines 1134-1151
```mlir
1134|   // CHECK:   %[[BAR_SLICE:.*]] = ttg.memdesc_index %[[BAR]]{{\[}}%[[BAR_IDX]]{{\]}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
1135|   // CHECK:   %[[BUF_IDX_P1:.*]] = arith.addi %[[BUF_IDX]], %[[C_1]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
1136|   // CHECK:   %[[BUF_IDX_CND:.*]] = arith.cmpi sge, %[[BUF_IDX_P1]], %[[C_2]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
1137|   // CHECK:   %[[BUF_IDX_NEXT:.*]] = arith.select %[[BUF_IDX_CND]], %[[C_0]], %[[BUF_IDX_P1]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
1138|   // CHECK:   %[[BUF_IDX_NEXT_CND:.*]] = arith.select %[[CND]], %[[BUF_IDX]], %[[BUF_IDX_NEXT]]
1139|   // CHECK:   %[[ACC_TM_SLICE:.*]] = ttg.memdesc_index %[[ACC_TM]]{{\[}}%[[BUF_IDX_NEXT_CND]]{{\]}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
1140|   // CHECK:   %[[MMA_TOK:.*]] = ttng.tc_gen5_mma %{{.*}}, %{{.*}}, %[[ACC_TM_SLICE]][], %[[CND]], %[[TRUE]], %[[BAR_SLICE]][%true] {is_async, loop.cluster = 0 : i32, loop.stage = 2 : i32}
1141|   // CHECK:   ttng.wait_barrier %[[BAR_SLICE]], %[[PHASE]] deps %{{.*}}, %{{.*}} {loop.cluster = 0 : i32, loop.stage = 3 : i32}
1142|   // CHECK:   scf.if
1143|   // CHECK:     %[[ACC_TM_SLICE:.*]] = ttg.memdesc_index %[[ACC_TM]]{{\[}}%[[BUF_IDX_NEXT_CND]]{{\]}}
1144|   // CHECK:     %[[LOAD_ACC:.*]], %[[USER_TOK:.*]] = ttng.tmem_load %[[ACC_TM_SLICE]][]
1145|   // CHECK:     "use"(%[[LOAD_ACC]])
1146|   // CHECK:   }
1147|   // CHECK:   %[[PHASE_NEG:.*]] = arith.xori %[[PHASE]], %[[C_1]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
1148|   // CHECK:   %[[BAR_IDX_P1:.*]] = arith.addi %[[BAR_IDX]], %[[C_1]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
1149|   // CHECK:   %[[BAR_IDX_CND:.*]] = arith.cmpi sge, %[[BAR_IDX_P1]], %[[C_2]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
1150|   // CHECK:   %[[BAR_IDX_NEXT:.*]] = arith.select %[[BAR_IDX_CND]], %[[C_0]], %[[BAR_IDX_P1]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
1151|   // CHECK:   %[[PHASE_NEXT:.*]] = arith.select %[[BAR_IDX_CND]], %[[PHASE_NEG]], %[[PHASE]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 1152-1155
```mlir
1152|   // CHECK:   scf.yield %[[PHASE_NEXT]], %[[BAR_IDX_NEXT]], %[[BUF_IDX_NEXT_CND]]
1153|   // CHECK: } {tt.scheduled_max_stage = 3 : i32}
1154|   // CHECK: %[[ACC_TM_SLICE:.*]] = ttg.memdesc_index %[[ACC_TM]]{{\[}}%[[FOR_RES]]#2{{\]}}
1155|   // CHECK: %[[LOAD_ACC:.*]], %[[RES_TOK:.*]] = ttng.tmem_load %[[ACC_TM_SLICE]][]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 1156-1173
```mlir
1156|   tt.func public @simple_persistent_mmav5_acc_flag(%arg0: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>}, %arg1: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>}, %arg2: i32) -> tensor<128x128xf16, #blocked1> {
1157|     %true = arith.constant true
1158|     %cst = "init_acc"() : () -> tensor<128x128xf32, #blocked1>
1159|     %cst_0 = "override_acc"() : () -> tensor<128x128xf32, #blocked1>
1160|     %c0_i32 = arith.constant 0 : i32
1161|     %c1_i32 = arith.constant 1 : i32
1162|     %0 = "cnd"() : () -> i1
1163|     %1, %acc_tok = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
1164|     %init_tok = ttng.tmem_store %cst, %1[%acc_tok], %true : tensor<128x128xf32, #blocked1> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1165|     %last_tok = scf.for %arg3 = %c0_i32 to %arg2 step %c1_i32 iter_args(%tok = %init_tok) -> !ttg.async.token : i32 {
1166|       %5 = tt.load %arg0 {loop.cluster = 2 : i32, loop.stage = 0 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
1167|       %6 = ttg.local_alloc %5 {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
1168|       %7 = tt.load %arg1 {loop.cluster = 2 : i32, loop.stage = 0 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
1169|       %8 = ttg.local_alloc %7 {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
1170|       %mma_tok = ttng.tc_gen5_mma %6, %8, %1[%tok], %0, %true {loop.cluster = 0 : i32, loop.stage = 2 : i32, tt.self_latency = 1 : i32} : !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1171|       %cnd_tok = scf.if %0 -> !ttg.async.token {
1172|         %9, %user_tok = ttng.tmem_load %1[%mma_tok] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
1173|         "use"(%9) : (tensor<128x128xf32, #blocked1>) -> ()
```
**EN:** This function-oriented block defines or enters `simple_persistent_mmav5_acc_flag`. Within it, the test exercises constants, tt.func, masked or vectorized loads, shared/local memory allocation, tensor-memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `simple_persistent_mmav5_acc_flag` 为核心。测试在其中演示 常量、tt.func、带掩码或向量化的加载、共享/本地内存分配、张量内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1174-1185
```mlir
1174|         scf.yield %user_tok : !ttg.async.token
1175|       } else {
1176|         scf.yield %mma_tok : !ttg.async.token
1177|       } {loop.cluster = 3 : i32, loop.stage = 3 : i32}
1178|       scf.yield %cnd_tok : !ttg.async.token
1179|     } {tt.scheduled_max_stage = 2 : i32}
1180|     %2, %res_tok = ttng.tmem_load %1[%last_tok] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
1181|     %3 = arith.truncf %2 : tensor<128x128xf32, #blocked1> to tensor<128x128xf16, #blocked1>
1182|     tt.return %3 : tensor<128x128xf16, #blocked1>
1183|   }
1184| }
1185| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, ttng.tmem_load, arith.truncf, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、ttng.tmem_load、arith.truncf、tt.return。

### Lines 1186-1186
```mlir
1186| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1187-1191
```mlir
1187| 
1188| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
1189| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
1190| #smem = #ttg.shared_memory
1191| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1192-1192
```mlir
1192| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 1193-1193
```mlir
1193|   // CHECK-LABEL: @mmav5_load_in_different_cluster
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @mmav5_load_in_different_cluster anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @mmav5_load_in_different_cluster 这样的标签用于锚定匹配范围。

### Lines 1194-1206
```mlir
1194|   tt.func public @mmav5_load_in_different_cluster(%arg0: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>}, %arg1: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>}, %arg2: i32, %arg3: tensor<128x128x!tt.ptr<f32>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>}, %arg4: i1) -> tensor<128x128xf16, #blocked> {
1195|     %true = arith.constant true
1196|     %false = arith.constant false
1197|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
1198|     %c0_i32 = arith.constant 0 : i32
1199|     %c1_i32 = arith.constant 1 : i32
1200|     %0, %acc_tok = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
1201|     %init_tok = ttng.tmem_store %cst, %0[%acc_tok], %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1202|     %last_tok = scf.for %arg5 = %c0_i32 to %arg2 step %c1_i32 iter_args(%tok = %init_tok) -> !ttg.async.token : i32 {
1203|       %3 = tt.load %arg0 {loop.cluster = 3 : i32, loop.stage = 0 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
1204|       %4 = ttg.local_alloc %3 {loop.cluster = 1 : i32, loop.stage = 2 : i32} : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
1205|       %5 = tt.load %arg1 {loop.cluster = 3 : i32, loop.stage = 0 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
1206|       %6 = ttg.local_alloc %5 {loop.cluster = 1 : i32, loop.stage = 2 : i32} : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `mmav5_load_in_different_cluster`. Within it, the test exercises constants, tt.func, masked or vectorized loads, shared/local memory allocation, tensor-memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mmav5_load_in_different_cluster` 为核心。测试在其中演示 常量、tt.func、带掩码或向量化的加载、共享/本地内存分配、张量内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1207-1208
```mlir
1207|       // CHECK: ttng.tc_gen5_mma {{.*}} {is_async, loop.cluster = 2 : i32, loop.stage = 2 : i32}
1208|       // Wait should be in the cluster right before the load
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1209-1221
```mlir
1209|       // CHECK: ttng.wait_barrier {{.*}} {loop.cluster = 0 : i32, loop.stage = 3 : i32}
1210|       // CHECK: ttng.tmem_load {{.*}} {loop.cluster = 1 : i32, loop.stage = 3 : i32}
1211|       %mma_tok = ttng.tc_gen5_mma %4, %6, %0[%tok], %false, %true {loop.cluster = 1 : i32, loop.stage = 2 : i32, tt.self_latency = 1 : i32} : !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1212|       %7, %load_tok = ttng.tmem_load %0[%mma_tok] {loop.cluster = 0 : i32, loop.stage = 3 : i32} : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
1213|       "use"(%7) {loop.cluster = 0 : i32, loop.stage = 3 : i32} : (tensor<128x128xf32, #blocked>) -> ()
1214|       scf.yield %load_tok : !ttg.async.token
1215|     } {tt.scheduled_max_stage = 2 : i32}
1216|     %1, %res_tok = ttng.tmem_load %0[%last_tok] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
1217|     %2 = arith.truncf %1 : tensor<128x128xf32, #blocked> to tensor<128x128xf16, #blocked>
1218|     tt.return %2 : tensor<128x128xf16, #blocked>
1219|   }
1220| }
1221| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining ttng.tmem_load, Gen5 tensor-core MMA ops, loop/if yielded values, arith.truncf, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 ttng.tmem_load、Gen5 张量核 MMA 操作、循环/分支产出值、arith.truncf、tt.return。

### Lines 1222-1222
```mlir
1222| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1223-1227
```mlir
1223| 
1224| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
1225| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
1226| #smem = #ttg.shared_memory
1227| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1228-1228
```mlir
1228| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 1229-1246
```mlir
1229|   // CHECK-LABEL: @chained_dot_wait_before_store
1230|   // CHECK-DAG: %[[C0_F:.+]] = arith.constant dense<0.000000e+00>
1231|   // CHECK-DAG: %[[TRUE:.+]] = arith.constant true
1232|   // CHECK-DAG: %[[CN1:.+]] = arith.constant -1 : i32
1233|   // CHECK-DAG: %[[C0:.+]] = arith.constant 0 : i32
1234|   // CHECK-DAG: %[[C1:.+]] = arith.constant 1 : i32
1235|   // CHECK-DAG: %[[C2:.+]] = arith.constant{{.*}} 2 : i32
1236|   // CHECK: %[[TMEM_BUF:.+]], %[[ACC_TOK:.+]] = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32
1237|   // CHECK: %[[INIT_TOK:.+]] = ttng.tmem_store %[[C0_F]], %[[TMEM_BUF]][%[[ACC_TOK]]]
1238|   // CHECK: %[[BAR_BUF:.+]] = ttg.local_alloc : () -> !ttg.memdesc<2x1xi64
1239|   // CHECK: %[[BAR_SLICE0:.+]] = ttg.memdesc_index %[[BAR_BUF]]{{\[}}%[[C0]]{{\]}}
1240|   // CHECK: ttng.init_barrier %[[BAR_SLICE0]], 1
1241|   // CHECK: %[[BAR_SLICE1:.+]] = ttg.memdesc_index %[[BAR_BUF]]{{\[}}%[[C1]]{{\]}}
1242|   // CHECK: ttng.init_barrier %[[BAR_SLICE1]], 1
1243|   // CHECK: %[[LHS_BUFS:.+]] = ttg.local_alloc
1244|   // CHECK: %[[RHS_BUFS:.+]] = ttg.local_alloc
1245|   // CHECK: %[[FOR_RES:.+]]:5 = scf.for {{.*}} iter_args(%[[TOK:[^,]+]] = %[[INIT_TOK]], %[[PHASE:[^,]+]] = %[[C0]], %[[BAR_IDX:[^,]+]] = %[[C0]],
1246|   // CHECK:   %[[IDX0:.+]] = arith.select
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @chained_dot_wait_before_store anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @chained_dot_wait_before_store 这样的标签用于锚定匹配范围。

### Lines 1247-1264
```mlir
1247|   // CHECK:   %[[IDX1:.+]] = arith.select
1248|   // CHECK:   %[[LHS_DEP:.+]] = ttg.memdesc_index %[[LHS_BUFS]]{{\[}}%[[IDX1]]{{\]}}
1249|   // CHECK:   %[[RHS_DEP:.+]] = ttg.memdesc_index %[[RHS_BUFS]]{{\[}}%[[IDX1]]{{\]}}
1250|   // CHECK:   %[[BAR_SLICE:.+]] = ttg.memdesc_index %[[BAR_BUF]]{{\[}}%[[BAR_IDX]]{{\]}} {loop.cluster = 0 : i32, loop.stage = 2 : i32}
1251|   // CHECK:   %[[MMA_TOK:.+]] = ttng.tc_gen5_mma {{.*}}, {{.*}}, %[[TMEM_BUF]][%[[TOK]]], %[[TRUE]], %[[TRUE]], %[[BAR_SLICE]][%true] {is_async, loop.cluster = 0 : i32, loop.stage = 2 : i32}
1252|   // CHECK:   ttng.wait_barrier %[[BAR_SLICE]], %[[PHASE]] deps %[[LHS_DEP]], %[[RHS_DEP]] {loop.cluster = 0 : i32, loop.stage = 3 : i32}
1253|   // CHECK:   %[[CND_TOK:.+]] = scf.if
1254|   // CHECK:     ttng.wait_barrier %[[BAR_SLICE]], %[[PHASE]] deps %[[LHS_DEP]], %[[RHS_DEP]]
1255|   // CHECK:     %[[ACC_RES:.+]], %[[USER_TOK:.+]] = ttng.tmem_load %[[TMEM_BUF]][%[[MMA_TOK]]]
1256|   // CHECK:     tt.store %{{.*}}, %[[ACC_RES]]
1257|   // CHECK:     yield %[[USER_TOK]]
1258|   // CHECK:   } else {
1259|   // CHECK:     yield %[[MMA_TOK]]
1260|   // CHECK:   } {loop.cluster = 3 : i32, loop.stage = 2 : i32}
1261|   // CHECK:   %[[PHASE_XOR:.+]] = arith.xori %[[PHASE]], %[[C1]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
1262|   // CHECK:   %[[BAR_IDX_P1:.+]] = arith.addi %[[BAR_IDX]], %[[C1]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
1263|   // CHECK:   %[[BAR_WRAP:.+]] = arith.cmpi sge, %[[BAR_IDX_P1]], %[[C2]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
1264|   // CHECK:   %[[BAR_IDX_NEXT:.+]] = arith.select %[[BAR_WRAP]], %[[C0]], %[[BAR_IDX_P1]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 1265-1272
```mlir
1265|   // CHECK:   %[[PHASE_NEXT:.+]] = arith.select %[[BAR_WRAP]], %[[PHASE_XOR]], %[[PHASE]] {loop.cluster = 0 : i32, loop.stage = 2 : i32}
1266|   // CHECK:   yield %[[CND_TOK]]
1267|   // CHECK: %[[BAR_SLICE0:.+]] = ttg.memdesc_index %[[BAR_BUF]]{{\[}}%[[C0]]{{\]}}
1268|   // CHECK: ttng.inval_barrier %[[BAR_SLICE0]]
1269|   // CHECK: %[[BAR_SLICE1:.+]] = ttg.memdesc_index %[[BAR_BUF]]{{\[}}%[[C1]]{{\]}}
1270|   // CHECK: ttng.inval_barrier %[[BAR_SLICE1]]
1271|   // CHECK: ttg.local_dealloc %[[BAR_BUF]]
1272|   // CHECK: ttng.tmem_load %[[TMEM_BUF]][%[[FOR_RES]]#0]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 1273-1290
```mlir
1273|   tt.func public @chained_dot_wait_before_store(%arg0: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>}, %arg1: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>}, %arg2: i32, %arg3: tensor<128x128x!tt.ptr<f32>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>}, %arg4: i1) -> tensor<128x128xf16, #blocked> {
1274|     %true = arith.constant true
1275|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
1276|     %c0_i32 = arith.constant 0 : i32
1277|     %c1_i32 = arith.constant 1 : i32
1278|     %0, %acc_tok = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
1279|     %init_tok = ttng.tmem_store %cst, %0[%acc_tok], %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1280|     %last_tok = scf.for %arg5 = %c0_i32 to %arg2 step %c1_i32 iter_args(%tok = %init_tok) -> !ttg.async.token : i32 {
1281|       %3 = tt.load %arg0 {loop.cluster = 2 : i32, loop.stage = 0 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
1282|       %4 = ttg.local_alloc %3 {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
1283|       %5 = tt.load %arg1 {loop.cluster = 2 : i32, loop.stage = 0 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
1284|       %6 = ttg.local_alloc %5 {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
1285|       %mma_tok = ttng.tc_gen5_mma %4, %6, %0[%tok], %true, %true {loop.cluster = 0 : i32, loop.stage = 2 : i32, tt.self_latency = 1 : i32} : !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1286|       %cnd_tok = scf.if %arg4 -> !ttg.async.token {
1287|         %7, %user_tok = ttng.tmem_load %0[%mma_tok] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
1288|         tt.store %arg3, %7 : tensor<128x128x!tt.ptr<f32>, #blocked>
1289|         scf.yield %user_tok : !ttg.async.token
1290|       } else {
```
**EN:** This function-oriented block defines or enters `chained_dot_wait_before_store`. Within it, the test exercises constants, tt.func, masked or vectorized loads, shared/local memory allocation, tensor-memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `chained_dot_wait_before_store` 为核心。测试在其中演示 常量、tt.func、带掩码或向量化的加载、共享/本地内存分配、张量内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1291-1300
```mlir
1291|         scf.yield %mma_tok : !ttg.async.token
1292|       } {loop.cluster = 3 : i32, loop.stage = 2 : i32}
1293|       scf.yield %cnd_tok : !ttg.async.token
1294|     } {tt.scheduled_max_stage = 2 : i32}
1295|     %1, %res_tok = ttng.tmem_load %0[%last_tok] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
1296|     %2 = arith.truncf %1 : tensor<128x128xf32, #blocked> to tensor<128x128xf16, #blocked>
1297|     tt.return %2 : tensor<128x128xf16, #blocked>
1298|   }
1299| }
1300| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, ttng.tmem_load, arith.truncf, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、ttng.tmem_load、arith.truncf、tt.return。

### Lines 1301-1301
```mlir
1301| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1302-1306
```mlir
1302| 
1303| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
1304| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
1305| #smem = #ttg.shared_memory
1306| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1307-1307
```mlir
1307| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 1308-1308
```mlir
1308|   // CHECK-LABEL: @multibuf_tmem1
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @multibuf_tmem1 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @multibuf_tmem1 这样的标签用于锚定匹配范围。

### Lines 1309-1313
```mlir
1309|   tt.func public @multibuf_tmem1(%arg0: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>}, %arg1: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>}, %arg2: tensor<128x128x!tt.ptr<f32>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>}, %arg3: tensor<128x128x!tt.ptr<f32>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>}, %arg4: i32) {
1310|     %true = arith.constant true
1311|     %c0_i32 = arith.constant 0 : i32
1312|     %c1_i32 = arith.constant 1 : i32
1313|     // Multibuffer tmem as users are scheduled after defs
```
**EN:** This function-oriented block defines or enters `multibuf_tmem1`. Within it, the test exercises constants, tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `multibuf_tmem1` 为核心。测试在其中演示 常量、tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1314-1330
```mlir
1314|     // CHECK: ttng.tmem_alloc : () -> (!ttg.memdesc<2x128x128xf32
1315|     %0, %acc_tok = ttng.tmem_alloc  : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
1316|     scf.for %arg5 = %c0_i32 to %arg4 step %c1_i32 iter_args(%tok = %acc_tok) -> !ttg.async.token : i32 {
1317|       %2 = tt.load %arg0 {loop.cluster = 4 : i32, loop.stage = 0 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
1318|       %3 = ttg.local_alloc %2 {loop.cluster = 2 : i32, loop.stage = 2 : i32} : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
1319|       %4 = tt.load %arg1 {loop.cluster = 4 : i32, loop.stage = 0 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
1320|       %5 = ttg.local_alloc %4 {loop.cluster = 2 : i32, loop.stage = 2 : i32} : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
1321|       %6 = tt.load %arg2 {loop.cluster = 2 : i32, loop.stage = 2 : i32} : tensor<128x128x!tt.ptr<f32>, #blocked>
1322|       %store_tok = ttng.tmem_store %6, %0[%tok], %true {loop.cluster = 2 : i32, loop.stage = 2 : i32} : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1323|       %mma_tok = ttng.tc_gen5_mma %3, %5, %0[%store_tok], %true, %true {loop.cluster = 2 : i32, loop.stage = 2 : i32, tt.self_latency = 1 : i32} : !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1324|       %res, %load_tok = ttng.tmem_load %0[%mma_tok] {loop.cluster = 2 : i32, loop.stage = 3 : i32} : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
1325|       scf.yield %load_tok : !ttg.async.token
1326|     } {tt.scheduled_max_stage = 3 : i32}
1327|     tt.return
1328|   }
1329| }
1330| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining masked or vectorized loads, shared/local memory allocation, tensor-memory allocation, structured loops, ttng.tmem_store.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 带掩码或向量化的加载、共享/本地内存分配、张量内存分配、结构化循环、ttng.tmem_store。

### Lines 1331-1331
```mlir
1331| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1332-1336
```mlir
1332| 
1333| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
1334| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
1335| #smem = #ttg.shared_memory
1336| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1337-1337
```mlir
1337| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 1338-1338
```mlir
1338|   // CHECK-LABEL: @multibuf_tmem2
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @multibuf_tmem2 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @multibuf_tmem2 这样的标签用于锚定匹配范围。

### Lines 1339-1343
```mlir
1339|   tt.func public @multibuf_tmem2(%arg0: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>}, %arg1: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>}, %arg2: tensor<128x128x!tt.ptr<f32>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>}, %arg3: tensor<128x128x!tt.ptr<f32>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>}, %arg4: i32) {
1340|     %true = arith.constant true
1341|     %c0_i32 = arith.constant 0 : i32
1342|     %c1_i32 = arith.constant 1 : i32
1343|     // Multibuffer tmem as users are scheduled after defs
```
**EN:** This function-oriented block defines or enters `multibuf_tmem2`. Within it, the test exercises constants, tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `multibuf_tmem2` 为核心。测试在其中演示 常量、tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1344-1360
```mlir
1344|     // CHECK: ttng.tmem_alloc : () -> (!ttg.memdesc<2x128x128xf32
1345|     %0, %acc_tok = ttng.tmem_alloc  : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
1346|     scf.for %arg5 = %c0_i32 to %arg4 step %c1_i32 iter_args(%tok = %acc_tok) -> !ttg.async.token : i32 {
1347|       %2 = tt.load %arg0 {loop.cluster = 4 : i32, loop.stage = 0 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
1348|       %3 = ttg.local_alloc %2 {loop.cluster = 2 : i32, loop.stage = 2 : i32} : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
1349|       %4 = tt.load %arg1 {loop.cluster = 4 : i32, loop.stage = 0 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
1350|       %5 = ttg.local_alloc %4 {loop.cluster = 2 : i32, loop.stage = 2 : i32} : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
1351|       %6 = tt.load %arg2 {loop.cluster = 2 : i32, loop.stage = 2 : i32} : tensor<128x128x!tt.ptr<f32>, #blocked>
1352|       %store_tok = ttng.tmem_store %6, %0[%tok], %true {loop.cluster = 2 : i32, loop.stage = 2 : i32} : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1353|       %mma_tok = ttng.tc_gen5_mma %3, %5, %0[%store_tok], %true, %true {loop.cluster = 2 : i32, loop.stage = 2 : i32, tt.self_latency = 1 : i32} : !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1354|       %res, %load_tok = ttng.tmem_load %0[%mma_tok] {loop.cluster = 3 : i32, loop.stage = 3 : i32} : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
1355|       scf.yield %load_tok : !ttg.async.token
1356|     } {tt.scheduled_max_stage = 3 : i32}
1357|     tt.return
1358|   }
1359| }
1360| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining masked or vectorized loads, shared/local memory allocation, tensor-memory allocation, structured loops, ttng.tmem_store.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 带掩码或向量化的加载、共享/本地内存分配、张量内存分配、结构化循环、ttng.tmem_store。

### Lines 1361-1361
```mlir
1361| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1362-1366
```mlir
1362| 
1363| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
1364| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
1365| #smem = #ttg.shared_memory
1366| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1367-1367
```mlir
1367| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 1368-1368
```mlir
1368|   // CHECK-LABEL: @two_dots
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @two_dots anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @two_dots 这样的标签用于锚定匹配范围。

### Lines 1369-1373
```mlir
1369|   tt.func public @two_dots(%arg0: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>}, %arg1: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>}, %arg2: tensor<128x128x!tt.ptr<f32>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>}, %arg3: tensor<128x128x!tt.ptr<f32>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>}, %arg4: i32) {
1370|     %true = arith.constant true
1371|     %c0_i32 = arith.constant 0 : i32
1372|     %c1_i32 = arith.constant 1 : i32
1373|     // Do not multi buffer tmem as uses are scheduled before defs
```
**EN:** This function-oriented block defines or enters `two_dots`. Within it, the test exercises constants, tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `two_dots` 为核心。测试在其中演示 常量、tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1374-1391
```mlir
1374|     // CHECK: ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32
1375|     // CHECK: ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32
1376|     // CHECK: scf.for
1377|     // CHECK: ttng.tc_gen5_mma {{.*}} {is_async, loop.cluster = 4 : i32, loop.stage = 2 : i32}
1378|     // CHECK: ttng.wait_barrier {{.*}} {loop.cluster = 2 : i32, loop.stage = 3 : i32}
1379|     // CHECK: ttng.tc_gen5_mma {{.*}} {is_async, loop.cluster = 3 : i32, loop.stage = 3 : i32}
1380|     // CHECK: ttng.wait_barrier {{.*}} {loop.cluster = 0 : i32, loop.stage = 4 : i32}
1381|     %0, %acc_tok0 = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
1382|     %1, %acc_tok1 = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
1383|     scf.for %arg5 = %c0_i32 to %arg4 step %c1_i32 iter_args(%tok0 = %acc_tok0, %tok1 = %acc_tok1) -> (!ttg.async.token, !ttg.async.token) : i32 {
1384|       %2 = tt.load %arg0 {loop.cluster = 4 : i32, loop.stage = 0 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
1385|       %3 = ttg.local_alloc %2 {loop.cluster = 2 : i32, loop.stage = 2 : i32} : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
1386|       %4 = tt.load %arg1 {loop.cluster = 4 : i32, loop.stage = 0 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
1387|       %5 = ttg.local_alloc %4 {loop.cluster = 2 : i32, loop.stage = 2 : i32} : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
1388|       %6 = tt.load %arg2 {loop.cluster = 2 : i32, loop.stage = 2 : i32} : tensor<128x128x!tt.ptr<f32>, #blocked>
1389| 
1390|       %store_tok0 = ttng.tmem_store %6, %0[%tok0], %true {loop.cluster = 2 : i32, loop.stage = 2 : i32} : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1391|       %mma_tok0 = ttng.tc_gen5_mma %3, %5, %0[%store_tok0], %true, %true {loop.cluster = 2 : i32, loop.stage = 2 : i32, tt.self_latency = 1 : i32} : !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining masked or vectorized loads, tensor-memory allocation, shared/local memory allocation, structured loops, ttng.tmem_store.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 带掩码或向量化的加载、张量内存分配、共享/本地内存分配、结构化循环、ttng.tmem_store。

### Lines 1392-1405
```mlir
1392|       %7, %load_tok0 = ttng.tmem_load %0[%mma_tok0] {loop.cluster = 1 : i32, loop.stage = 3 : i32} : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
1393| 
1394|       %store_tok1 = ttng.tmem_store %7, %1[%tok1], %true {loop.cluster = 1 : i32, loop.stage = 3 : i32} : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1395|       %mma_tok1 = ttng.tc_gen5_mma %3, %5, %1[%store_tok1], %true, %true {loop.cluster = 1 : i32, loop.stage = 3 : i32, tt.self_latency = 1 : i32} : !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1396|       %8, %load_tok1 = ttng.tmem_load %1[%mma_tok1] {loop.cluster = 0 : i32, loop.stage = 4 : i32} : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
1397| 
1398|       tt.store %arg3, %8 {loop.cluster = 0 : i32, loop.stage = 4 : i32} : tensor<128x128x!tt.ptr<f32>, #blocked>
1399| 
1400|       scf.yield %load_tok0, %load_tok1 : !ttg.async.token, !ttg.async.token
1401|     } {tt.scheduled_max_stage = 4 : i32}
1402|     tt.return
1403|   }
1404| }
1405| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining ttng.tmem_load, ttng.tmem_store, Gen5 tensor-core MMA ops, masked or vectorized stores, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 ttng.tmem_load、ttng.tmem_store、Gen5 张量核 MMA 操作、带掩码或向量化的存储、循环/分支产出值。

### Lines 1406-1406
```mlir
1406| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1407-1423
```mlir
1407| 
1408| #blocked = #ttg.blocked<{sizePerThread = [1, 256], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
1409| #blocked1 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [1, 0]}>
1410| #blocked2 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
1411| #blocked3 = #ttg.blocked<{sizePerThread = [1, 1, 1, 1, 1], threadsPerWarp = [1, 1, 1, 2, 16], warpsPerCTA = [1, 1, 1, 4, 1], order = [4, 3, 2, 1, 0]}>
1412| #blocked4 = #ttg.blocked<{sizePerThread = [1, 1, 1, 1, 1], threadsPerWarp = [1, 1, 2, 4, 4], warpsPerCTA = [1, 1, 4, 1, 1], order = [4, 3, 2, 1, 0]}>
1413| #blocked5 = #ttg.blocked<{sizePerThread = [1, 1, 1, 1, 1], threadsPerWarp = [1, 4, 2, 1, 4], warpsPerCTA = [1, 1, 4, 1, 1], order = [4, 1, 2, 3, 0]}>
1414| #linear = #ttg.linear<{register = [[0, 1], [0, 2], [32, 0], [64, 0]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0]], warp = [[0, 0], [0, 0]], block = []}>
1415| #linear1 = #ttg.linear<{register = [[0, 0, 0, 0, 1], [0, 0, 0, 0, 2], [0, 1, 0, 0, 0], [0, 2, 0, 0, 0], [1, 0, 0, 0, 0]], lane = [[0, 0, 1, 0, 0], [0, 0, 2, 0, 0], [0, 0, 4, 0, 0], [0, 0, 8, 0, 0], [0, 0, 16, 0, 0]], warp = [[0, 0, 0, 0, 0], [0, 0, 0, 0, 0]], block = []}>
1416| #linear2 = #ttg.linear<{register = [[0, 1], [0, 2], [32, 0], [64, 0], [128, 0]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0]], warp = [[0, 0], [0, 0]], block = []}>
1417| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 8}>
1418| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 8, rank = 3}>
1419| #shared2 = #ttg.nvmma_shared<{swizzlingByteWidth = 0, transposed = false, elementBitWidth = 8, rank = 5}>
1420| #shared3 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 8, fp4Padded = true}>
1421| #smem = #ttg.shared_memory
1422| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 256, colStride = 1>
1423| #tmem_scales = #ttng.tensor_memory_scales_encoding<>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1424-1424
```mlir
1424| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 1425-1442
```mlir
1425|   tt.func public @scaled_mmav5_unswizzled(%arg0: !tt.ptr<f8E4M3FN> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f8E4M3FN> {tt.divisibility = 16 : i32}, %arg2: i32 {tt.divisibility = 16 : i32}, %arg3: i32 {tt.divisibility = 16 : i32}, %arg4: i32 {tt.divisibility = 16 : i32}, %arg5: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg6: !tt.ptr<f8E4M3FN> {tt.divisibility = 16 : i32}, %arg7: i32 {tt.divisibility = 16 : i32}, %arg8: i32 {tt.divisibility = 16 : i32}, %arg9: !tt.ptr<i8> {tt.divisibility = 16 : i32}, %arg10: i32 {tt.divisibility = 16 : i32}, %arg11: i32 {tt.divisibility = 16 : i32}, %arg12: !tt.ptr<i8> {tt.divisibility = 16 : i32}, %arg13: i32 {tt.divisibility = 16 : i32}, %arg14: i32 {tt.divisibility = 16 : i32}, %arg15: i32 {tt.divisibility = 16 : i32}, %arg16: i32 {tt.divisibility = 16 : i32}, %arg17: i32 {tt.divisibility = 16 : i32}, %arg18: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg19: !tt.ptr<i32> {tt.divisibility = 16 : i32}, %arg20: i32 {tt.divisibility = 16 : i32}, %arg21: !tt.ptr<i32> {tt.divisibility = 16 : i32}, %arg22: i32 {tt.divisibility = 16 : i32}, %arg23: !tt.ptr<i32> {tt.divisibility = 16 : i32}, %arg24: !tt.ptr<i32> {tt.divisibility = 16 : i32}, %arg25: !tt.ptr<i32>, %arg26: !tt.ptr<i32>, %arg27: i32, %arg28: i32, %arg29: i32) {
1426|     %cst = arith.constant dense<0.000000e+00> : tensor<128x256xf32, #blocked>
1427|     %true = arith.constant true
1428|     %c16_i64 = arith.constant 16 : i64
1429|     %c1_i32 = arith.constant 1 : i32
1430|     %c1_i64 = arith.constant 1 : i64
1431|     %c0_i32 = arith.constant 0 : i32
1432|     %c16_i32 = arith.constant 16 : i32
1433|     %c32_i32 = arith.constant 32 : i32
1434|     %c32_i64 = arith.constant 32 : i64
1435|     %cst_0 = arith.constant dense<127> : tensor<128x4xi8, #linear>
1436|     %0 = tt.make_tensor_descriptor %arg6, [%c32_i32, %c32_i32], [%c32_i64, %c1_i64] : <f8E4M3FN>, <1x128xf8E4M3FN, #shared>
1437|     %1 = tt.make_tensor_descriptor %arg9, [%c32_i32, %c32_i32, %c32_i32], [%c32_i64, %c32_i64, %c1_i64] : <i8>, <1x64x256xi8, #shared1>
1438|     %2 = tt.make_tensor_descriptor %arg12, [%c32_i32, %c32_i32, %c32_i32, %c32_i32, %c16_i32], [%c32_i64, %c32_i64, %c32_i64, %c16_i64, %c1_i64] : <i8>, <1x2x1x32x16xi8, #shared2>
1439|     %3 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
1440|     %4 = ttng.tmem_alloc %cst_0 : (tensor<128x4xi8, #linear>) -> !ttg.memdesc<128x4xi8, #tmem_scales, #ttng.tensor_memory>
1441|     %5, %acc_tok = ttng.tmem_alloc : () -> (!ttg.memdesc<128x256xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
1442|     %init_tok = ttng.tmem_store %cst, %5[%acc_tok], %true : tensor<128x256xf32, #blocked> -> !ttg.memdesc<128x256xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** This function-oriented block defines or enters `scaled_mmav5_unswizzled`. Within it, the test exercises constants, tt.make_tensor_descriptor, tt.func, tensor-memory allocation, lane/block index ranges, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `scaled_mmav5_unswizzled` 为核心。测试在其中演示 常量、tt.make_tensor_descriptor、tt.func、张量内存分配、lane/block 索引范围，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1443-1444
```mlir
1443|     %last_tok = scf.for %arg30 = %c0_i32 to %c32_i32 step %c1_i32 iter_args(%tok = %init_tok) -> !ttg.async.token : i32 {
1444|       // Scale format is not tma compatible, so we have a local_alloc inside the loop
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 1445-1462
```mlir
1445|       // CHECK: ttng.wait_barrier
1446|       // CHECK: ttg.local_load
1447|       // CHECK: ttg.local_alloc
1448|       // CHECK: ttng.wait_barrier
1449|       // CHECK: ttng.tmem_alloc
1450|       // CHECK: ttng.tc_gen5_mma_scaled
1451| 
1452|       %7 = tt.descriptor_gather %0[%3, %c0_i32] {loop.cluster = 2 : i32, loop.stage = 0 : i32} : (!tt.tensordesc<1x128xf8E4M3FN, #shared>, tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>, i32) -> tensor<128x128xf8E4M3FN, #blocked2>
1453|       %8 = ttg.local_alloc %7 {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x128xf8E4M3FN, #blocked2>) -> !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem>
1454|       %9 = tt.descriptor_load %1[%arg30, %c0_i32, %c0_i32] {loop.cluster = 2 : i32, loop.stage = 0 : i32} : !tt.tensordesc<1x64x256xi8, #shared1> -> tensor<64x256xi8, #blocked2>
1455|       %10 = ttg.local_alloc %9 {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<64x256xi8, #blocked2>) -> !ttg.memdesc<64x256xi8, #shared3, #smem>
1456|       %11 = tt.descriptor_load %2[%arg30, %c0_i32, %c0_i32, %c0_i32, %c0_i32] {loop.cluster = 2 : i32, loop.stage = 0 : i32} : !tt.tensordesc<1x2x1x32x16xi8, #shared2> -> tensor<1x2x1x32x16xi8, #blocked3>
1457|       %12 = tt.reshape %11 {loop.cluster = 0 : i32, loop.stage = 2 : i32} : tensor<1x2x1x32x16xi8, #blocked3> -> tensor<2x1x32x4x4xi8, #blocked4>
1458|       %13 = tt.trans %12 {loop.cluster = 0 : i32, loop.stage = 2 : i32, order = array<i32: 0, 3, 2, 1, 4>} : tensor<2x1x32x4x4xi8, #blocked4> -> tensor<2x4x32x1x4xi8, #blocked5>
1459|       %14 = ttg.convert_layout %13 {loop.cluster = 0 : i32, loop.stage = 2 : i32} : tensor<2x4x32x1x4xi8, #blocked5> -> tensor<2x4x32x1x4xi8, #linear1>
1460|       %15 = tt.reshape %14 {loop.cluster = 0 : i32, loop.stage = 2 : i32} : tensor<2x4x32x1x4xi8, #linear1> -> tensor<256x4xi8, #linear2>
1461| 
1462|       %16 = ttng.tmem_alloc %15 {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<256x4xi8, #linear2>) -> !ttg.memdesc<256x4xi8, #tmem_scales, #ttng.tensor_memory>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, tt.descriptor_load, tensor reshaping, tt.descriptor_gather, transpose-like layout changes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、tt.descriptor_load、张量重塑、tt.descriptor_gather、转置类布局变换。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1463-1471
```mlir
1463|       %mma_tok = ttng.tc_gen5_mma_scaled %8, %10, %5[%tok], %4, %16, %true, %true lhs = e4m3 rhs = e2m1 {loop.cluster = 0 : i32, loop.stage = 2 : i32} : !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem>, !ttg.memdesc<64x256xi8, #shared3, #smem>, !ttg.memdesc<128x256xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.memdesc<128x4xi8, #tmem_scales, #ttng.tensor_memory>, !ttg.memdesc<256x4xi8, #tmem_scales, #ttng.tensor_memory>
1464|       scf.yield %mma_tok : !ttg.async.token
1465|     } {tt.disallow_acc_multi_buffer, tt.scheduled_max_stage = 2 : i32}
1466| 
1467|     %6, %res_tok = ttng.tmem_load %5[%last_tok] : !ttg.memdesc<128x256xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x256xf32, #blocked>
1468|     tt.return
1469|   }
1470| }
1471| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining scaled tensor-core MMA ops, loop/if yielded values, ttng.tmem_load, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 带缩放的张量核 MMA 操作、循环/分支产出值、ttng.tmem_load、tt.return。

### Lines 1472-1472
```mlir
1472| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1473-1478
```mlir
1473| 
1474| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
1475| #blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
1476| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
1477| #smem = #ttg.shared_memory
1478| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1479-1479
```mlir
1479| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 1480-1493
```mlir
1480|   // CHECK-LABEL: @changed_acc_before_mma
1481|   // CHECK-DAG: %[[TRUE:.+]] = arith.constant true
1482|   // CHECK: %[[TMEM_BUF:.+]], %[[ACC_TOK:.+]] = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32
1483|   // CHECK: %[[BAR_BUF:.+]] = ttg.local_alloc : () -> !ttg.memdesc<2x1xi64
1484|   // CHECK: %[[A_BUF:.+]] = ttg.local_alloc : () -> !ttg.memdesc<3x128x128xf16
1485|   // CHECK: %[[B_BUF:.+]] = ttg.local_alloc : () -> !ttg.memdesc<3x128x128xf16
1486|   // CHECK: scf.for
1487|   // CHECK:   %[[ACC1:.*]], %[[LOAD_TOK:.+]] = ttng.tmem_load %[[TMEM_BUF]][%{{.*}}] {loop.cluster = 2 : i32, loop.stage = 2 : i32}
1488|   // CHECK:   %[[MUL:.*]] = arith.mulf %[[ACC1]], {{.*}} {loop.cluster = 2 : i32, loop.stage = 2 : i32}
1489|   // CHECK:   %[[STORE_TOK:.+]] = ttng.tmem_store %[[MUL]], %[[TMEM_BUF]][%[[LOAD_TOK]]], {{.*}} {loop.cluster = 2 : i32, loop.stage = 2 : i32}
1490|   // CHECK:   %[[BAR_SLICE:.*]] = ttg.memdesc_index %[[BAR_BUF]]
1491|   // CHECK:   %[[MMA_TOK:.+]] = ttng.tc_gen5_mma %[[A_SLICE:.*]], %[[B_SLICE:.*]], %[[TMEM_BUF]][%[[STORE_TOK]]], {{.*}}, {{.*}}, %[[BAR_SLICE]][%true] {is_async, loop.cluster = 2 : i32, loop.stage = 2 : i32}
1492|   // CHECK:   ttng.wait_barrier %[[BAR_SLICE]], {{.*}} {loop.cluster = 1 : i32, loop.stage = 3 : i32}
1493|   // CHECK:   scf.yield %[[MMA_TOK]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @changed_acc_before_mma anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @changed_acc_before_mma 这样的标签用于锚定匹配范围。

### Lines 1494-1511
```mlir
1494|   tt.func public @changed_acc_before_mma(%arg0: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>}, %arg1: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>}, %arg2: i32) -> tensor<128x128xf16, #blocked1> {
1495|     %true = arith.constant true
1496|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked1>
1497|     %cst_0 = arith.constant dense<2.000000e+00> : tensor<128x128xf32, #blocked1>
1498|     %c0_i32 = arith.constant 0 : i32
1499|     %c1_i32 = arith.constant 1 : i32
1500|     %0, %acc_tok = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
1501|     %init_tok = ttng.tmem_store %cst, %0[%acc_tok], %true : tensor<128x128xf32, #blocked1> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1502|     %last_tok = scf.for %arg3 = %c0_i32 to %arg2 step %c1_i32 iter_args(%tok = %init_tok) -> !ttg.async.token : i32 {
1503|       %3 = tt.load %arg0 {loop.cluster = 2 : i32, loop.stage = 0 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
1504|       %4 = ttg.local_alloc %3 {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
1505|       %5 = tt.load %arg1 {loop.cluster = 2 : i32, loop.stage = 0 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
1506|       %6 = ttg.local_alloc %5 {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
1507|       %7, %load_tok = ttng.tmem_load %0[%tok] {loop.cluster = 0 : i32, loop.stage = 2 : i32} : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
1508|       %8 = arith.mulf %7, %cst_0 {loop.cluster = 0 : i32, loop.stage = 2 : i32} : tensor<128x128xf32, #blocked1>
1509|       %store_tok = ttng.tmem_store %8, %0[%load_tok], %true {loop.cluster = 0 : i32, loop.stage = 2 : i32} : tensor<128x128xf32, #blocked1> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1510|       %mma_tok = ttng.tc_gen5_mma %4, %6, %0[%store_tok], %true, %true {loop.cluster = 0 : i32, loop.stage = 2 : i32, tt.self_latency = 1 : i32} : !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1511|       scf.yield %mma_tok : !ttg.async.token
```
**EN:** This function-oriented block defines or enters `changed_acc_before_mma`. Within it, the test exercises constants, tt.func, ttng.tmem_store, masked or vectorized loads, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `changed_acc_before_mma` 为核心。测试在其中演示 常量、tt.func、ttng.tmem_store、带掩码或向量化的加载、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1512-1518
```mlir
1512|     } {tt.scheduled_max_stage = 2 : i32}
1513|     %1, %res_tok = ttng.tmem_load %0[%last_tok] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
1514|     %2 = arith.truncf %1 : tensor<128x128xf32, #blocked1> to tensor<128x128xf16, #blocked1>
1515|     tt.return %2 : tensor<128x128xf16, #blocked1>
1516|   }
1517| }
1518| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_load, arith.truncf, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_load、arith.truncf、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1519-1519
```mlir
1519| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1520-1526
```mlir
1520| 
1521| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
1522| #blocked1 = #ttg.blocked<{sizePerThread = [1, 64], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
1523| #blocked2 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
1524| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
1525| #smem = #ttg.shared_memory
1526| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 64, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1527-1529
```mlir
1527| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
1528|   // Check that wait is pushed to the next stage, right before the tmem_load, and after the prologue,
1529|   // despite mma being impossible to pipeline.
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 1530-1536
```mlir
1530|   // CHECK-LABEL: @changed_acc_unpipelineable_operand
1531|   // CHECK: scf.for
1532|   // CHECK: "prologue"() {loop.cluster = 0 : i32, loop.stage = 2 : i32}
1533|   // CHECK: ttng.tmem_load {{.*}} {loop.cluster = 2 : i32, loop.stage = 2 : i32}
1534|   // CHECK: ttg.async_copy_global_to_local {{.*}} {contiguity = 8 : i32, loop.cluster = 2 : i32, loop.stage = 2 : i32}
1535|   // CHECK: ttng.tc_gen5_mma {{.*}} {is_async, loop.cluster = 2 : i32, loop.stage = 2 : i32}
1536|   // CHECK: ttng.wait_barrier {{.*}} {loop.cluster = 1 : i32, loop.stage = 3 : i32}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @changed_acc_unpipelineable_operand anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @changed_acc_unpipelineable_operand 这样的标签用于锚定匹配范围。

### Lines 1537-1554
```mlir
1537|   tt.func public @changed_acc_unpipelineable_operand(%A: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>},
1538|                                                      %B: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>},
1539|                                                       %arg1: i32, %arg2: i32, %arg3: i32) {
1540|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked1>
1541|     %true = arith.constant true
1542|     %0 = ttng.tmem_alloc : () -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1543|     ttng.tmem_store %cst, %0, %true : tensor<128x128xf32, #blocked1> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1544|     scf.for %arg4 = %arg1 to %arg2 step %arg3  : i32 {
1545|       %2 = "prologue"() {loop.cluster = 0 : i32, loop.stage = 2 : i32} : () -> tensor<128x128xf16, #blocked2>
1546|       %3 = ttng.tmem_load %0 {loop.cluster = 0 : i32, loop.stage = 2 : i32} : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
1547|       %4 = "acc_modify"(%3, %2) {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x128xf32, #blocked1>, tensor<128x128xf16, #blocked2>) -> tensor<128x128xf32, #blocked1>
1548|       %5 = tt.load %B {loop.cluster = 0 : i32, loop.stage = 2 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
1549|       %6 = ttg.local_alloc %5 {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem>
1550|       %7 = tt.load %A {loop.cluster = 2 : i32, loop.stage = 0 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
1551|       %8 = ttg.local_alloc %7 {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem>
1552|       ttng.tmem_store %4, %0, %true {loop.cluster = 0 : i32, loop.stage = 2 : i32} : tensor<128x128xf32, #blocked1> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1553|       ttng.tc_gen5_mma %6, %8, %0, %true, %true {loop.cluster = 0 : i32, loop.stage = 2 : i32, tt.self_latency = 1 : i32} : !ttg.memdesc<128x128xf16, #shared, #smem>, !ttg.memdesc<128x128xf16, #shared, #smem>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1554|     } {tt.scheduled_max_stage = 2 : i32}
```
**EN:** This function-oriented block defines or enters `changed_acc_unpipelineable_operand`. Within it, the test exercises tt.func, constants, ttng.tmem_store, masked or vectorized loads, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `changed_acc_unpipelineable_operand` 为核心。测试在其中演示 tt.func、常量、ttng.tmem_store、带掩码或向量化的加载、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1555-1560
```mlir
1555|     %1 = ttng.tmem_load %0 : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
1556|     "use"(%1) : (tensor<128x128xf32, #blocked1>) -> ()
1557|     tt.return
1558|   }
1559| }
1560| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_load, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_load、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1561-1561
```mlir
1561| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1562-1568
```mlir
1562| 
1563| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
1564| #blocked1 = #ttg.blocked<{sizePerThread = [1, 64], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
1565| #blocked2 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
1566| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
1567| #smem = #ttg.shared_memory
1568| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 64, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1569-1571
```mlir
1569| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
1570|   // Check that wait is pushed to the next stage, right before the tmem_load, and after the prologue,
1571|   // despite mma being impossible to pipeline.
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 1572-1578
```mlir
1572|   // CHECK-LABEL: @changed_acc_unpipelineable_operand2
1573|   // CHECK: scf.for
1574|   // CHECK: "prologue"() {loop.cluster = 0 : i32, loop.stage = 2 : i32}
1575|   // CHECK: ttg.async_copy_global_to_local {{.*}} {contiguity = 8 : i32, loop.cluster = 2 : i32, loop.stage = 2 : i32}
1576|   // CHECK: ttng.tmem_load {{.*}} {loop.cluster = 2 : i32, loop.stage = 2 : i32}
1577|   // CHECK: ttng.tc_gen5_mma {{.*}} {is_async, loop.cluster = 2 : i32, loop.stage = 2 : i32}
1578|   // CHECK: ttng.wait_barrier {{.*}} {loop.cluster = 1 : i32, loop.stage = 3 : i32}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @changed_acc_unpipelineable_operand2 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @changed_acc_unpipelineable_operand2 这样的标签用于锚定匹配范围。

### Lines 1579-1596
```mlir
1579|   tt.func public @changed_acc_unpipelineable_operand2(%A: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>},
1580|                                                      %B: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>},
1581|                                                       %arg1: i32, %arg2: i32, %arg3: i32) {
1582|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked1>
1583|     %true = arith.constant true
1584|     %0 = ttng.tmem_alloc : () -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1585|     ttng.tmem_store %cst, %0, %true : tensor<128x128xf32, #blocked1> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1586|     scf.for %arg4 = %arg1 to %arg2 step %arg3  : i32 {
1587|       %2 = "prologue"() {loop.cluster = 0 : i32, loop.stage = 2 : i32} : () -> tensor<128x128xf16, #blocked2>
1588|       %5 = tt.load %B {loop.cluster = 0 : i32, loop.stage = 2 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
1589|       %6 = ttg.local_alloc %5 {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem>
1590|       %3 = ttng.tmem_load %0 {loop.cluster = 0 : i32, loop.stage = 2 : i32} : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
1591|       %4 = "acc_modify"(%3, %2) {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x128xf32, #blocked1>, tensor<128x128xf16, #blocked2>) -> tensor<128x128xf32, #blocked1>
1592| 
1593|       %7 = tt.load %A {loop.cluster = 2 : i32, loop.stage = 0 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
1594|       %8 = ttg.local_alloc %7 {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem>
1595|       ttng.tmem_store %4, %0, %true {loop.cluster = 0 : i32, loop.stage = 2 : i32} : tensor<128x128xf32, #blocked1> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1596|       ttng.tc_gen5_mma %6, %8, %0, %true, %true {loop.cluster = 0 : i32, loop.stage = 2 : i32, tt.self_latency = 1 : i32} : !ttg.memdesc<128x128xf16, #shared, #smem>, !ttg.memdesc<128x128xf16, #shared, #smem>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** This function-oriented block defines or enters `changed_acc_unpipelineable_operand2`. Within it, the test exercises tt.func, constants, ttng.tmem_store, masked or vectorized loads, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `changed_acc_unpipelineable_operand2` 为核心。测试在其中演示 tt.func、常量、ttng.tmem_store、带掩码或向量化的加载、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1597-1603
```mlir
1597|     } {tt.scheduled_max_stage = 2 : i32}
1598|     %1 = ttng.tmem_load %0 : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
1599|     "use"(%1) : (tensor<128x128xf32, #blocked1>) -> ()
1600|     tt.return
1601|   }
1602| }
1603| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_load, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_load、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1604-1604
```mlir
1604| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1605-1611
```mlir
1605| 
1606| #blocked1 = #ttg.blocked<{sizePerThread = [1, 64], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
1607| #blocked2 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
1608| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
1609| #smem = #ttg.shared_memory
1610| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
1611| #tmem_f16 = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1612-1614
```mlir
1612| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
1613|   // Check that wait is pushed to the next stage, right before the tmem_alloc, and after the prologue.
1614|   // Check that tmem is hoisted out of the loop.
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 1615-1623
```mlir
1615|   // CHECK-LABEL: @wait_before_tmem_alloc
1616|   // CHECK: ttng.tmem_alloc
1617|   // CHECK: %[[TMEM_BUF:.+]] = ttng.tmem_alloc
1618|   // CHECK: scf.for
1619|   // CHECK: "prologue"() {loop.cluster = 0 : i32, loop.stage = 2 : i32}
1620|   // CHECK: ttng.tmem_store {{.*}}, %[[TMEM_BUF]], {{.*}} {loop.cluster = 2 : i32, loop.stage = 2 : i32}
1621|   // CHECK: ttg.async_copy_global_to_local {{.*}} {contiguity = 8 : i32, loop.cluster = 2 : i32, loop.stage = 2 : i32}
1622|   // CHECK: ttng.tc_gen5_mma {{.*}} {is_async, loop.cluster = 2 : i32, loop.stage = 2 : i32}
1623|   // CHECK: ttng.wait_barrier {{.*}} {loop.cluster = 1 : i32, loop.stage = 3 : i32}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @wait_before_tmem_alloc anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @wait_before_tmem_alloc 这样的标签用于锚定匹配范围。

### Lines 1624-1641
```mlir
1624|   tt.func public @wait_before_tmem_alloc(%A: tensor<128x128xf16, #blocked1> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>},
1625|                                          %B: tensor<128x128x!tt.ptr<f16>, #blocked1> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>},
1626|                                          %arg1: i32, %arg2: i32, %arg3: i32) {
1627|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked1>
1628|     %true = arith.constant true
1629|     %0 = ttng.tmem_alloc : () -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1630|     ttng.tmem_store %cst, %0, %true : tensor<128x128xf32, #blocked1> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1631|     scf.for %arg4 = %arg1 to %arg2 step %arg3  : i32 {
1632|       %2 = "prologue"() {loop.cluster = 0 : i32, loop.stage = 2 : i32} : () -> tensor<128x128xf16, #blocked2>
1633|       %8 = ttng.tmem_alloc %A {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #tmem_f16, #ttng.tensor_memory>
1634|       %5 = tt.load %B {loop.cluster = 0 : i32, loop.stage = 2 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked1>
1635|       %6 = ttg.local_alloc %5 {loop.cluster = 0 : i32, loop.stage = 2 : i32} : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #shared, #smem>
1636|       ttng.tc_gen5_mma %8, %6, %0, %true, %true {loop.cluster = 0 : i32, loop.stage = 2 : i32, tt.self_latency = 1 : i32} : !ttg.memdesc<128x128xf16, #tmem_f16, #ttng.tensor_memory>, !ttg.memdesc<128x128xf16, #shared, #smem>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1637|     } {tt.scheduled_max_stage = 2 : i32}
1638|     %1 = ttng.tmem_load %0 : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
1639|     "use"(%1) : (tensor<128x128xf32, #blocked1>) -> ()
1640|     tt.return
1641|   }
```
**EN:** This function-oriented block defines or enters `wait_before_tmem_alloc`. Within it, the test exercises tt.func, constants, tensor-memory allocation, ttng.tmem_store, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `wait_before_tmem_alloc` 为核心。测试在其中演示 tt.func、常量、张量内存分配、ttng.tmem_store、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1642-1643
```mlir
1642| }
1643| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 1644-1644
```mlir
1644| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1645-1647
```mlir
1645| 
1646| #A = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
1647| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1648-1648
```mlir
1648| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 1649-1652
```mlir
1649| // CHECK-LABEL: @load_cant_use_async_cp
1650| // CHECK: scf.for
1651| // CHECK:   tt.load
1652| // CHECK:   "use"
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @load_cant_use_async_cp anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @load_cant_use_async_cp 这样的标签用于锚定匹配范围。

### Lines 1653-1662
```mlir
1653| tt.func @load_cant_use_async_cp(%lb : index, %ub : index, %step : index,
1654|                  %a_ptr_init : tensor<128x32x!tt.ptr<f16>, #A>) -> () {
1655|   scf.for %iv = %lb to %ub step %step : index {
1656|     %a = tt.load %a_ptr_init {loop.cluster = 1 : i32, loop.stage = 0 : i32} : tensor<128x32x!tt.ptr<f16>, #A>
1657|     "use"(%a) {loop.cluster = 2 : i32, loop.stage = 3 : i32} : (tensor<128x32xf16, #A>) -> ()
1658|   } {tt.scheduled_max_stage = 3 : i32}
1659|   tt.return
1660| }
1661| }
1662| 
```
**EN:** This function-oriented block defines or enters `load_cant_use_async_cp`. Within it, the test exercises tt.func, structured loops, masked or vectorized loads, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `load_cant_use_async_cp` 为核心。测试在其中演示 tt.func、结构化循环、带掩码或向量化的加载、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1663-1663
```mlir
1663| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1664-1665
```mlir
1664| 
1665| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 1666-1666
```mlir
1666| // CHECK-LABEL: @scalar_load
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @scalar_load anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @scalar_load 这样的标签用于锚定匹配范围。

### Lines 1667-1669
```mlir
1667| tt.func @scalar_load(%lb : index, %ub : index, %step : index,
1668|                      %a_ptr_init : !tt.ptr<i32>) -> () {
1669|   scf.for %iv = %lb to %ub step %step : index {
```
**EN:** This function-oriented block defines or enters `scalar_load`. Within it, the test exercises tt.func, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `scalar_load` 为核心。测试在其中演示 tt.func、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1670-1683
```mlir
1670|     // CHECK: %[[PTR:.+]] = tt.splat %{{.*}} {loop.cluster = 0 : i32, loop.stage = 0 : i32} : !tt.ptr<i32>
1671|     // CHECK: %[[CP:.+]] = ttg.async_copy_global_to_local %[[PTR]], %{{.+}} {loop.cluster = 0 : i32, loop.stage = 0 : i32}
1672|     // CHECK: %[[T0:.+]] = ttg.async_commit_group tokens %[[CP]] {loop.cluster = 0 : i32, loop.stage = 0 : i32}
1673|     // CHECK: %[[T1:.+]] = ttg.async_wait %[[T0]] {loop.cluster = 1 : i32, loop.stage = 3 : i32, num = 0 : i32}
1674|     // CHECK: %[[L:.+]] = ttg.local_load %{{.+}} token %[[T1]] {loop.cluster = 1 : i32, loop.stage = 3 : i32}
1675|     // CHECK: %[[R:.+]] = tt.unsplat %[[L]] {loop.cluster = 1 : i32, loop.stage = 3 : i32}
1676|     // CHECK: "use"(%[[R]]) {loop.cluster = 1 : i32, loop.stage = 3 : i32} : (i32) -> ()
1677|     %a = tt.load %a_ptr_init {loop.cluster = 1 : i32, loop.stage = 0 : i32} : !tt.ptr<i32>
1678|     "use"(%a) {loop.cluster = 2 : i32, loop.stage = 3 : i32} : (i32) -> ()
1679|   } {tt.scheduled_max_stage = 3 : i32}
1680|   tt.return
1681| }
1682| }
1683| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1684-1684
```mlir
1684| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1685-1687
```mlir
1685| 
1686| #blocked1 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
1687| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1688-1692
```mlir
1688| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
1689| // Test for conditional store pipelining bugfix
1690| // This test reproduces the race condition where conditional code (scf.if) gets moved to
1691| // epilogue cluster, causing users of loads to be scheduled in later clusters than the loads themselves.
1692| // The fix allocates extra buffer space when this situation is detected.
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 1693-1696
```mlir
1693| // CHECK-LABEL: @conditional_store_race_fix
1694| // CHECK: ttg.local_alloc : () -> !ttg.memdesc<3x{{.*}}>
1695| // CHECK: scf.if %{{.*}} {
1696| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @conditional_store_race_fix anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @conditional_store_race_fix 这样的标签用于锚定匹配范围。

### Lines 1697-1714
```mlir
1697| tt.func @conditional_store_race_fix(%lb : index, %ub : index, %step : index,
1698|                  %a_ptr_init : tensor<128x32x!tt.ptr<f16>, #blocked1> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
1699|                  %out_ptr : tensor<128x32x!tt.ptr<f16>, #blocked1>,
1700|                  %cnd : i1) -> () {
1701|   scf.for %iv = %lb to %ub step %step : index {
1702|     // Load is in cluster 0, stage 0 (early cluster)
1703|     %a = tt.load %a_ptr_init {loop.cluster = 0 : i32, loop.stage = 0 : i32} : tensor<128x32x!tt.ptr<f16>, #blocked1>
1704|     // Conditional store is in cluster 2, stage 2 (later cluster than load: 2 > 0)
1705|     // This creates the race condition where the local load happens after
1706|     // the global-to-local copy for the next pipeline stage starts
1707|     scf.if %cnd {
1708|       tt.store %out_ptr, %a {loop.cluster = 2 : i32, loop.stage = 2 : i32} : tensor<128x32x!tt.ptr<f16>, #blocked1>
1709|     } {loop.cluster = 2 : i32, loop.stage = 2 : i32}
1710|   } {tt.scheduled_max_stage = 2 : i32}
1711|   tt.return
1712| }
1713| }
1714| 
```
**EN:** This function-oriented block defines or enters `conditional_store_race_fix`. Within it, the test exercises tt.func, structured loops, masked or vectorized loads, structured conditionals, masked or vectorized stores, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `conditional_store_race_fix` 为核心。测试在其中演示 tt.func、结构化循环、带掩码或向量化的加载、结构化条件分支、带掩码或向量化的存储，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1715-1715
```mlir
1715| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1716-1722
```mlir
1716| 
1717| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [2, 2], order = [1, 0]}>
1718| #blocked1 = #ttg.blocked<{sizePerThread = [1, 32], threadsPerWarp = [16, 2], warpsPerCTA = [4, 1], order = [0, 1]}>
1719| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
1720| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
1721| #smem = #ttg.shared_memory
1722| #tmem = #ttng.tensor_memory_encoding<blockM = 64, blockN = 64, colStride=1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1723-1723
```mlir
1723| module attributes {ttg.max_reg_auto_ws = 152 : i32, ttg.min_reg_auto_ws = 24 : i32, "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 1724-1724
```mlir
1724|   // CHECK-LABEL: @non_pipelined_op
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @non_pipelined_op anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @non_pipelined_op 这样的标签用于锚定匹配范围。

### Lines 1725-1734
```mlir
1725|   tt.func public @non_pipelined_op(%x_desc: !tt.tensordesc<64x64xbf16, #shared>, %x_desc_0: i32, %x_desc_1: i32, %x_desc_2: i64, %x_desc_3: i64, %y_desc: !tt.tensordesc<64x64xbf16, #shared>, %y_desc_4: i32, %y_desc_5: i32, %y_desc_6: i64, %y_desc_7: i64, %out_desc: !tt.tensordesc<64x64xf32, #shared1>, %out_desc_8: i32, %out_desc_9: i32, %out_desc_10: i64, %out_desc_11: i64, %N: i32 {tt.divisibility = 16 : i32}) attributes {noinline = false} {
1726|     %acc = arith.constant false
1727|     %true = arith.constant true
1728|     %c1_i32 = arith.constant 1 : i32
1729|     %BLOCK_N = arith.constant 64 : i32
1730|     %c0_i32 = arith.constant 0 : i32
1731|     %x = tt.descriptor_load %x_desc[%c0_i32, %c0_i32] : !tt.tensordesc<64x64xbf16, #shared> -> tensor<64x64xbf16, #blocked>
1732|     %x_12 = ttg.local_alloc %x : (tensor<64x64xbf16, #blocked>) -> !ttg.memdesc<64x64xbf16, #shared, #smem>
1733|     %num_slices = arith.divsi %N, %BLOCK_N : i32
1734|     %acc_13, %acc_14 = ttng.tmem_alloc : () -> (!ttg.memdesc<64x64xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
```
**EN:** This function-oriented block defines or enters `non_pipelined_op`. Within it, the test exercises constants, tt.func, tt.descriptor_load, shared/local memory allocation, arith.divsi, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `non_pipelined_op` 为核心。测试在其中演示 常量、tt.func、tt.descriptor_load、共享/本地内存分配、arith.divsi，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1735-1738
```mlir
1735|     // CHECK-COUNT-3: ttng.init_barrier {{.*}}
1736|     // CHECK: scf.for
1737|     %0 = scf.for %i = %c0_i32 to %num_slices step %c1_i32 iter_args(%acc_15 = %acc_14) -> (!ttg.async.token)  : i32 {
1738|       %y = arith.muli %i, %BLOCK_N {loop.cluster = 1 : i32, loop.stage = 0 : i32} : i32
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops, integer multiplications.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环、整数乘法。

### Lines 1739-1741
```mlir
1739|       // CHECK: ttng.barrier_expect {{.*}} {loop.cluster = 2 : i32, loop.stage = 0 : i32}, {{.*}}
1740|       // CHECK: ttng.async_tma_copy_global_to_local {{.*}} {loop.cluster = 2 : i32, loop.stage = 0 : i32} {{.*}}
1741|       %y_16 = tt.descriptor_load %y_desc[%c0_i32, %y] {loop.cluster = 1 : i32, loop.stage = 0 : i32} : !tt.tensordesc<64x64xbf16, #shared> -> tensor<64x64xbf16, #blocked>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 1742-1743
```mlir
1742|       // CHECK: ttng.wait_barrier {{.*}} {loop.cluster = 2 : i32, loop.stage = 0 : i32} {{.*}}
1743|       %y_17 = ttg.local_alloc %y_16 {loop.cluster = 1 : i32, loop.stage = 0 : i32} : (tensor<64x64xbf16, #blocked>) -> !ttg.memdesc<64x64xbf16, #shared, #smem>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 1744-1745
```mlir
1744|       // CHECK:{{.*}} = ttng.tc_gen5_mma {{.*}} {is_async, loop.cluster = 2 : i32, loop.stage = 0 : i32} {{.*}}
1745|       %acc_18 = ttng.tc_gen5_mma %x_12, %y_17, %acc_13[%acc_15], %acc, %true {loop.cluster = 1 : i32, loop.stage = 0 : i32, tt.self_latency = 1 : i32} : !ttg.memdesc<64x64xbf16, #shared, #smem>, !ttg.memdesc<64x64xbf16, #shared, #smem>, !ttg.memdesc<64x64xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 1746-1749
```mlir
1746|       // CHECK: ttng.wait_barrier {{.*}} {loop.cluster = 0 : i32, loop.stage = 1 : i32} {{.*}}
1747|       // CHECK: {{.*}} = ttng.tmem_load {{.*}} {loop.cluster = 1 : i32, loop.stage = 1 : i32} {{.*}}
1748|       %acc_19, %acc_20 = ttng.tmem_load %acc_13[%acc_18] {loop.cluster = 0 : i32, loop.stage = 1 : i32} : !ttg.memdesc<64x64xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<64x64xf32, #blocked1>
1749|       %1 = ttg.convert_layout %acc_19 {loop.cluster = 0 : i32, loop.stage = 1 : i32} : tensor<64x64xf32, #blocked1> -> tensor<64x64xf32, #blocked>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_load, layout conversions. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_load、布局转换。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1750-1757
```mlir
1750|       // CHECK: tt.descriptor_store {{.*}} {loop.cluster = 1 : i32, loop.stage = 1 : i32} {{.*}}
1751|       tt.descriptor_store %out_desc[%c0_i32, %y], %1 {loop.cluster = 0 : i32, loop.stage = 1 : i32} : !tt.tensordesc<64x64xf32, #shared1>, tensor<64x64xf32, #blocked>
1752|       scf.yield %acc_20 : !ttg.async.token
1753|     } {tt.scheduled_max_stage = 1 : i32}
1754|     tt.return
1755|   }
1756| }
1757| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining tt.descriptor_store, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 tt.descriptor_store、循环/分支产出值、tt.return。

### Lines 1758-1758
```mlir
1758| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1759-1765
```mlir
1759| 
1760| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [2, 2], order = [1, 0]}>
1761| #blocked1 = #ttg.blocked<{sizePerThread = [1, 32], threadsPerWarp = [16, 2], warpsPerCTA = [4, 1], order = [0, 1]}>
1762| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
1763| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
1764| #smem = #ttg.shared_memory
1765| #tmem = #ttng.tensor_memory_encoding<blockM = 64, blockN = 64, colStride=1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1766-1766
```mlir
1766| module attributes {ttg.max_reg_auto_ws = 152 : i32, ttg.min_reg_auto_ws = 24 : i32, "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 1767-1767
```mlir
1767|   // CHECK-LABEL: @non_pipelined_op_two_stage
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @non_pipelined_op_two_stage anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @non_pipelined_op_two_stage 这样的标签用于锚定匹配范围。

### Lines 1768-1777
```mlir
1768|   tt.func public @non_pipelined_op_two_stage(%x_desc: !tt.tensordesc<64x64xbf16, #shared>, %x_desc_0: i32, %x_desc_1: i32, %x_desc_2: i64, %x_desc_3: i64, %y_desc: !tt.tensordesc<64x64xbf16, #shared>, %y_desc_4: i32, %y_desc_5: i32, %y_desc_6: i64, %y_desc_7: i64, %out_desc: !tt.tensordesc<64x64xf32, #shared1>, %out_desc_8: i32, %out_desc_9: i32, %out_desc_10: i64, %out_desc_11: i64, %N: i32 {tt.divisibility = 16 : i32}) attributes {noinline = false} {
1769|     %acc = arith.constant false
1770|     %true = arith.constant true
1771|     %c1_i32 = arith.constant 1 : i32
1772|     %BLOCK_N = arith.constant 64 : i32
1773|     %c0_i32 = arith.constant 0 : i32
1774|     %x = tt.descriptor_load %x_desc[%c0_i32, %c0_i32] : !tt.tensordesc<64x64xbf16, #shared> -> tensor<64x64xbf16, #blocked>
1775|     %x_12 = ttg.local_alloc %x : (tensor<64x64xbf16, #blocked>) -> !ttg.memdesc<64x64xbf16, #shared, #smem>
1776|     %num_slices = arith.divsi %N, %BLOCK_N : i32
1777|     %acc_13, %acc_14 = ttng.tmem_alloc : () -> (!ttg.memdesc<64x64xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
```
**EN:** This function-oriented block defines or enters `non_pipelined_op_two_stage`. Within it, the test exercises constants, tt.func, tt.descriptor_load, shared/local memory allocation, arith.divsi, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `non_pipelined_op_two_stage` 为核心。测试在其中演示 常量、tt.func、tt.descriptor_load、共享/本地内存分配、arith.divsi，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1778-1781
```mlir
1778|     // CHECK-COUNT-3: ttng.init_barrier {{.*}}
1779|     // CHECK: scf.for
1780|     %0 = scf.for %i = %c0_i32 to %num_slices step %c1_i32 iter_args(%acc_15 = %acc_14) -> (!ttg.async.token)  : i32 {
1781|       %y = arith.muli %i, %BLOCK_N {loop.cluster = 1 : i32, loop.stage = 0 : i32} : i32
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops, integer multiplications.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环、整数乘法。

### Lines 1782-1784
```mlir
1782|       // CHECK: ttng.barrier_expect {{.*}} {loop.cluster = 3 : i32, loop.stage = 0 : i32}, {{.*}}
1783|       // CHECK: ttng.async_tma_copy_global_to_local {{.*}} {loop.cluster = 3 : i32, loop.stage = 0 : i32} {{.*}}
1784|       %y_16 = tt.descriptor_load %y_desc[%c0_i32, %y] {loop.cluster = 1 : i32, loop.stage = 0 : i32} : !tt.tensordesc<64x64xbf16, #shared> -> tensor<64x64xbf16, #blocked>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 1785-1786
```mlir
1785|       // CHECK: ttng.wait_barrier {{.*}} {loop.cluster = 3 : i32, loop.stage = 0 : i32} {{.*}}
1786|       %y_17 = ttg.local_alloc %y_16 {loop.cluster = 1 : i32, loop.stage = 0 : i32} : (tensor<64x64xbf16, #blocked>) -> !ttg.memdesc<64x64xbf16, #shared, #smem>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 1787-1788
```mlir
1787|       // CHECK:{{.*}} = ttng.tc_gen5_mma {{.*}} {is_async, loop.cluster = 3 : i32, loop.stage = 0 : i32} {{.*}}
1788|       %acc_18 = ttng.tc_gen5_mma %x_12, %y_17, %acc_13[%acc_15], %acc, %true {loop.cluster = 1 : i32, loop.stage = 0 : i32, tt.self_latency = 1 : i32} : !ttg.memdesc<64x64xbf16, #shared, #smem>, !ttg.memdesc<64x64xbf16, #shared, #smem>, !ttg.memdesc<64x64xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 1789-1792
```mlir
1789|       // CHECK: ttng.wait_barrier {{.*}} {loop.cluster = 2 : i32, loop.stage = 1 : i32} {{.*}}
1790|       // CHECK: {{.*}} = ttng.tmem_load {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32} {{.*}}
1791|       %acc_19, %acc_20 = ttng.tmem_load %acc_13[%acc_18] {loop.cluster = 0 : i32, loop.stage = 2 : i32} : !ttg.memdesc<64x64xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<64x64xf32, #blocked1>
1792|       %1 = ttg.convert_layout %acc_19 {loop.cluster = 0 : i32, loop.stage = 2 : i32} : tensor<64x64xf32, #blocked1> -> tensor<64x64xf32, #blocked>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_load, layout conversions. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_load、布局转换。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1793-1799
```mlir
1793|       // CHECK: tt.descriptor_store {{.*}} {loop.cluster = 0 : i32, loop.stage = 2 : i32} {{.*}}
1794|       tt.descriptor_store %out_desc[%c0_i32, %y], %1 {loop.cluster = 0 : i32, loop.stage = 2 : i32} : !tt.tensordesc<64x64xf32, #shared1>, tensor<64x64xf32, #blocked>
1795|       scf.yield %acc_20 : !ttg.async.token
1796|     } {tt.scheduled_max_stage = 2 : i32}
1797|     tt.return
1798|   }
1799| }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining tt.descriptor_store, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 tt.descriptor_store、循环/分支产出值、tt.return。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-allow-unregistered-dialect`, `-tritongpu-test-pipeline-lower-loop`, `-canonicalize`
- **CN:** 主要 pass 选项：`-split-input-file`，`-allow-unregistered-dialect`，`-tritongpu-test-pipeline-lower-loop`，`-canonicalize`
- **EN:** Dominant operations include `tt.func`, `arith.constant`, `tt.load`, `ttg.local_alloc`, `module`, `scf.for`, `tt.return`, `scf.yield`, `ttng.tmem_load`, `ttng.tmem_alloc`.
- **CN:** 主要操作包括 `tt.func`、`arith.constant`、`tt.load`、`ttg.local_alloc`、`module`、`scf.for`、`tt.return`、`scf.yield`、`ttng.tmem_load`、`ttng.tmem_alloc`。
- **EN:** The file contains 42 independently testable section(s). Check styles used: CHECK x477, CHECK-DAG x142, CHECK-LABEL x41, CHECK-NOT x3. Important labels include @unscheduled_loop, @one_dep_async, @one_dep_barrier_wait, @one_dep_barrier_wait_trans. It validates both the presence of the desired rewrite and the absence of unwanted residual operations.
- **CN:** 该文件包含 42 个可独立测试的分段。使用的检查类型：CHECK ×477，CHECK-DAG ×142，CHECK-LABEL ×41，CHECK-NOT ×3。 关键标签包括 @unscheduled_loop，@one_dep_async，@one_dep_barrier_wait，@one_dep_barrier_wait_trans。 它同时验证目标重写已经出现，并且不再残留不需要的操作。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `ttg`: TritonGPU layout, memory, and GPU-specific operations.
- **CN:** `ttg`：TritonGPU 布局、内存和 GPU 专用操作。
- **EN:** `tt`: Triton high-level tensor/pointer operations.
- **CN:** `tt`：Triton 高层张量/指针操作。
- **EN:** `ttng`: Triton Nvidia GPU tensor-memory and MMA operations.
- **CN:** `ttng`：Triton Nvidia GPU 张量内存与 MMA 操作。
- **EN:** `arith`: MLIR arithmetic dialect for scalar/tensor math.
- **CN:** `arith`：用于标量/张量运算的 MLIR arithmetic 方言。
- **EN:** `scf`: Structured control flow for loops and conditionals.
- **CN:** `scf`：用于循环与条件分支的结构化控制流。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。