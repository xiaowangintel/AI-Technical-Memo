# loop-pipeline-async-latencies.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/loop-pipeline-async-latencies.mlir`
- **EN:** Test for the MLIR/Triton pipeline `--tritongpu-assign-latencies, --tritongpu-schedule-loops, --tritongpu-pipeline, -canonicalize` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `--tritongpu-assign-latencies, --tritongpu-schedule-loops, --tritongpu-pipeline, -canonicalize` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s --tritongpu-assign-latencies --tritongpu-schedule-loops --tritongpu-pipeline -canonicalize -cse | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s --tritongpu-assign-latencies --tritongpu-schedule-loops --tritongpu-pipeline -canonicalize -cse | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt %s --tritongpu-assign-latencies --tritongpu-schedule-loops --tritongpu-pipeline -canonicalize -cse | FileCheck %s
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s --tritongpu-assign-latencies --tritongpu-schedule-loops --tritongpu-pipeline -canonicalize -cse | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s --tritongpu-assign-latencies --tritongpu-schedule-loops --tritongpu-pipeline -canonicalize -cse | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-8
```mlir
3| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 2], order = [1, 0]}>
4| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 8], order = [1, 0]}>
5| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [8, 1], instrShape = [16, 256, 16]}>
6| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
7| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 16}>
8| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 9-10
```mlir
 9| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
10| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 11-11
```mlir
11| // CHECK-LABEL: matmul_kernel_tma_persistent
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: matmul_kernel_tma_persistent anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: matmul_kernel_tma_persistent 这样的标签用于锚定匹配范围。

### Lines 12-18
```mlir
12| tt.func public @matmul_kernel_tma_persistent(%arg0: !tt.tensordesc<128x64xf16, #shared>, %arg1: !tt.tensordesc<256x64xf16, #shared>, %arg2: !tt.tensordesc<128x256xf16, #shared>, %arg3: i32 {tt.divisibility = 16 : i32}, %arg4: i32 {tt.divisibility = 16 : i32}, %arg5: i32 {tt.divisibility = 16 : i32}) {
13|   %c2_i32 = arith.constant 2 : i32
14|   %c1_i32 = arith.constant 1 : i32
15|   %c0_i32 = arith.constant 0 : i32
16|   %cst = arith.constant dense<0.000000e+00> : tensor<128x256xf32, #mma>
17|   %0 = arith.subi %arg3, %c2_i32 : i32
18| 
```
**EN:** This function-oriented block defines or enters `matmul_kernel_tma_persistent`. Within it, the test exercises constants, tt.func, arith.subi, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `matmul_kernel_tma_persistent` 为核心。测试在其中演示 常量、tt.func、arith.subi，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 19-36
```mlir
19|   // CHECK: [[LHS_BUFFERS:%.*]] = ttg.local_alloc : () -> !ttg.memdesc<2x128x64xf16,
20|   // CHECK: [[RHS_BUFFERS:%.*]] = ttg.local_alloc : () -> !ttg.memdesc<4x256x64xf16,
21| 
22|   // CHECK: [[LHS_BARS:%.*]] = ttg.local_alloc : () -> !ttg.memdesc<2x1xi64,
23|   // CHECK-NEXT: [[LHS_BAR0:%.*]] = ttg.memdesc_index [[LHS_BARS]]{{\[}}%c0_i32{{\]}}
24|   // CHECK-NEXT: ttng.init_barrier [[LHS_BAR0]]
25|   // CHECK-NEXT: [[LHS_BAR1:%.*]] = ttg.memdesc_index [[LHS_BARS]]{{\[}}%c1_i32{{\]}}
26|   // CHECK-NEXT: ttng.init_barrier [[LHS_BAR1]]
27| 
28|   // CHECK: [[RHS_BARS:%.*]] = ttg.local_alloc : () -> !ttg.memdesc<4x1xi64,
29|   // CHECK-NEXT: [[RHS_BAR0:%.*]] = ttg.memdesc_index [[RHS_BARS]]{{\[}}%c0_i32{{\]}}
30|   // CHECK-NEXT: ttng.init_barrier [[RHS_BAR0]]
31|   // CHECK-NEXT: [[RHS_BAR1:%.*]] = ttg.memdesc_index [[RHS_BARS]]{{\[}}%c1_i32{{\]}}
32|   // CHECK-NEXT: ttng.init_barrier [[RHS_BAR1]]
33|   // CHECK-NEXT: [[RHS_BAR2:%.*]] = ttg.memdesc_index [[RHS_BARS]]{{\[}}%c2_i32{{\]}}
34|   // CHECK-NEXT: ttng.init_barrier [[RHS_BAR2]]
35|   // CHECK-NEXT: [[RHS_BAR3:%.*]] = ttg.memdesc_index [[RHS_BARS]]{{\[}}%c3_i32{{\]}}
36|   // CHECK-NEXT: ttng.init_barrier [[RHS_BAR3]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 37-54
```mlir
37| 
38|   // CHECK: [[MASK0:%.*]] = arith.cmpi sgt, %arg3, %c0_i32
39|   // CHECK-NEXT: ttng.barrier_expect [[RHS_BAR0]], 32768, [[MASK0]]
40|   // CHECK-NEXT: [[RHS_BUF0:%.*]] = ttg.memdesc_index [[RHS_BUFFERS]]{{\[}}%c0_i32{{\]}}
41|   // CHECK-NEXT: ttng.async_tma_copy_global_to_local %arg1[%c0_i32, %c0_i32] [[RHS_BUF0]], [[RHS_BAR0]], [[MASK0]]
42| 
43|   // CHECK: [[MASK1:%.*]] = arith.cmpi sgt, %arg3, %c1_i32
44|   // CHECK-NEXT: ttng.barrier_expect [[RHS_BAR1]], 32768, [[MASK1]]
45|   // CHECK-NEXT: [[RHS_BUF1:%.*]] = ttg.memdesc_index [[RHS_BUFFERS]]{{\[}}%c1_i32{{\]}}
46|   // CHECK-NEXT: ttng.async_tma_copy_global_to_local %arg1[%c0_i32, %c1_i32] [[RHS_BUF1]], [[RHS_BAR1]], [[MASK1]]
47| 
48|   // CHECK: [[MASK2:%.*]] = arith.cmpi sgt, %arg3, %c2_i32
49| 
50|   // CHECK-NEXT: ttng.barrier_expect [[LHS_BAR0]], 16384, [[MASK0]]
51|   // CHECK-NEXT: [[LHS_BUF0:%.*]] = ttg.memdesc_index [[LHS_BUFFERS]]{{\[}}%c0_i32{{\]}}
52|   // CHECK-NEXT: ttng.async_tma_copy_global_to_local %arg0[%c0_i32, %c0_i32] [[LHS_BUF0]], [[LHS_BAR0]], [[MASK0]]
53| 
54|   // CHECK: ttng.barrier_expect [[RHS_BAR2]], 32768, [[MASK2]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 55-60
```mlir
55|   // CHECK-NEXT: [[RHS_BUF2:%.*]] = ttg.memdesc_index [[RHS_BUFFERS]]{{\[}}%c2_i32{{\]}}
56|   // CHECK-NEXT: ttng.async_tma_copy_global_to_local %arg1[%c0_i32, %c2_i32] [[RHS_BUF2]], [[RHS_BAR2]], [[MASK2]]
57| 
58|   %true = arith.constant true
59|   %false = arith.constant false
60| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 61-71
```mlir
61|   // CHECK: scf.for [[I:%.*]] = %c0_i32 to
62|   // CHECK-SAME: iter_args([[ACCUM:%arg[0-9]+]] = %cst
63| 
64|   // CHECK-SAME: [[NEXT_LHS_BUF_IDX:%arg[0-9]+]] = %c0_i32
65|   // CHECK-SAME: [[LHS_BUF_IDX:%arg[0-9]+]] = %c-1_i32
66|   // CHECK-SAME: [[LHS_PHASE_ARG:%arg[0-9]+]] = %c0_i32
67| 
68|   // CHECK-SAME: [[NEXT_RHS_BUF_IDX:%arg[0-9]+]] = %c2_i32
69|   // CHECK-SAME: [[RHS_BUF_IDX:%arg[0-9]+]] = %c-1_i32
70|   // CHECK-SAME: [[RHS_PHASE_ARG:%arg[0-9]+]] = %c0_i32
71|   %3 = scf.for %arg6 = %c0_i32 to %arg3 step %c1_i32 iter_args(%arg7 = %cst) -> (tensor<128x256xf32, #mma>)  : i32 {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 72-77
```mlir
72|     // CHECK: [[RHS_MAX_ITER:%.*]] = arith.subi %arg3, %c3_i32
73|     // CHECK-NEXT: [[RHS_MASK:%.*]] = arith.cmpi slt, [[I]], [[RHS_MAX_ITER]]
74|     // CHECK: [[LHS_MAX_ITER:%.*]] = arith.subi %arg3, %c1_i32
75|     // CHECK-NEXT: [[LHS_MASK:%.*]] = arith.cmpi slt, [[I]], [[LHS_MAX_ITER]]
76| 
77|     // Compute RHS buffer index modulo 4.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 78-82
```mlir
78|     // CHECK: [[V0:%.*]] = arith.addi [[RHS_BUF_IDX]], %c1_i32
79|     // CHECK-NEXT: [[V1:%.*]] = arith.cmpi sge, [[V0]], %c4_i32
80|     // CHECK-NEXT: [[RHS_BUF_IDX:%.*]] = arith.select [[V1]], %c0_i32, [[V0]]
81| 
82|     // Compute RHS phase index modulo 4.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 83-86
```mlir
83|     // CHECK: [[V0:%.*]] = arith.xori [[RHS_PHASE_ARG]], %c1_i32
84|     // CHECK-NEXT: [[RHS_PHASE:%.*]] = arith.select [[V1]], [[V0]], [[RHS_PHASE_ARG]]
85| 
86|     // Compute LHS buffer index modulo 2.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 87-91
```mlir
87|     // CHECK: [[V0:%.*]] = arith.addi [[LHS_BUF_IDX]], %c1_i32
88|     // CHECK-NEXT: [[V1:%.*]] = arith.cmpi sge, [[V0]], %c2_i32
89|     // CHECK-NEXT: [[LHS_BUF_IDX:%.*]] = arith.select [[V1]], %c0_i32, [[V0]]
90| 
91|     // Compute LHS phase index modulo 2.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 92-107
```mlir
 92|     // CHECK: [[V0:%.*]] = arith.xori [[LHS_PHASE_ARG]], %c1_i32
 93|     // CHECK-NEXT: [[LHS_PHASE:%.*]] = arith.select [[V1]], [[V0]], [[LHS_PHASE_ARG]]
 94| 
 95|     // CHECK: [[LHS_MBAR:%.*]] = ttg.memdesc_index [[LHS_BARS]]{{\[}}[[LHS_BUF_IDX]]{{\]}}
 96|     // CHECK-NEXT: ttng.wait_barrier [[LHS_MBAR]], [[LHS_PHASE]]
 97| 
 98|     // CHECK: [[RHS_MBAR:%.*]] = ttg.memdesc_index [[RHS_BARS]]{{\[}}[[RHS_BUF_IDX]]{{\]}}
 99|     // CHECK-NEXT: ttng.wait_barrier [[RHS_MBAR]], [[RHS_PHASE]]
100| 
101|     %4 = tt.descriptor_load %arg0[%c0_i32, %arg6] {tt.latency = 1 : i32} : !tt.tensordesc<128x64xf16, #shared> -> tensor<128x64xf16, #blocked>
102|     %5 = ttg.local_alloc %4 : (tensor<128x64xf16, #blocked>) -> !ttg.memdesc<128x64xf16, #shared, #smem>
103|     %6 = tt.descriptor_load %arg1[%c0_i32, %arg6] {tt.latency = 3 : i32} : !tt.tensordesc<256x64xf16, #shared> -> tensor<256x64xf16, #blocked>
104|     %7 = ttg.local_alloc %6 : (tensor<256x64xf16, #blocked>) -> !ttg.memdesc<256x64xf16, #shared, #smem>
105|     %8 = ttg.memdesc_trans %7 {order = array<i32: 1, 0>} : !ttg.memdesc<256x64xf16, #shared, #smem> -> !ttg.memdesc<64x256xf16, #shared1, #smem>
106|     %9 = ttng.warp_group_dot %5, %8, %arg7 {inputPrecision = 0 : i32} : !ttg.memdesc<128x64xf16, #shared, #smem> * !ttg.memdesc<64x256xf16, #shared1, #smem> -> tensor<128x256xf32, #mma>
107| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.descriptor_load, shared/local memory allocation, ttg.memdesc_trans, ttng.warp_group_dot. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.descriptor_load、共享/本地内存分配、ttg.memdesc_trans、ttng.warp_group_dot。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 108-125
```mlir
108|     // CHECK: [[V0:%.*]] = arith.addi [[NEXT_LHS_BUF_IDX]], %c1_i32
109|     // CHECK-NEXT: [[V1:%.*]] = arith.cmpi sge, [[V0]], %c2_i32
110|     // CHECK-NEXT: [[NEXT_LHS_BUF_IDX:%.*]] = arith.select [[V1]], %c0_i32, [[V0]]
111|     // CHECK-NEXT: [[NEXT_LHS_BAR:%.*]] = ttg.memdesc_index [[LHS_BARS]]{{\[}}[[NEXT_LHS_BUF_IDX]]{{\]}}
112|     // CHECK-NEXT: ttng.barrier_expect [[NEXT_LHS_BAR]], 16384, [[LHS_MASK]]
113| 
114|     // CHECK-NEXT: [[NEXT_LHS_BUF:%.*]] = ttg.memdesc_index [[LHS_BUFFERS]]{{\[}}[[NEXT_LHS_BUF_IDX]]{{\]}}
115|     // CHECK-NEXT: [[NEXT_LHS_IDX:%.*]] = arith.addi [[I]], %c1_i32
116|     // CHECK-NEXT: ttng.async_tma_copy_global_to_local %arg0[%c0_i32, [[NEXT_LHS_IDX]]] [[NEXT_LHS_BUF]], [[NEXT_LHS_BAR]], [[LHS_MASK]]
117| 
118|     // CHECK: [[V0:%.*]] = arith.addi [[NEXT_RHS_BUF_IDX]], %c1_i32
119|     // CHECK-NEXT: [[V1:%.*]] = arith.cmpi sge, [[V0]], %c4_i32
120|     // CHECK-NEXT: [[NEXT_RHS_BUF_IDX:%.*]] = arith.select [[V1]], %c0_i32, [[V0]]
121|     // CHECK-NEXT: [[NEXT_RHS_BAR:%.*]] = ttg.memdesc_index [[RHS_BARS]]{{\[}}[[NEXT_RHS_BUF_IDX]]{{\]}}
122|     // CHECK-NEXT: ttng.barrier_expect [[NEXT_RHS_BAR]], 32768, [[RHS_MASK]]
123| 
124|     // CHECK-NEXT: [[NEXT_RHS_BUF:%.*]] = ttg.memdesc_index [[RHS_BUFFERS]]{{\[}}[[NEXT_RHS_BUF_IDX]]{{\]}}
125|     // CHECK-NEXT: [[NEXT_RHS_IDX:%.*]] = arith.addi [[I]], %c3_i32
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 126-133
```mlir
126|     // CHECK-NEXT: ttng.async_tma_copy_global_to_local %arg1[%c0_i32, [[NEXT_RHS_IDX]]] [[NEXT_RHS_BUF]], [[NEXT_RHS_BAR]], [[RHS_MASK]]
127| 
128|     %10 = arith.cmpi eq, %arg3, %0 : i32
129|     scf.if %10 {
130|       %11 = arith.truncf %9 : tensor<128x256xf32, #mma> to tensor<128x256xf16, #mma>
131|       %12 = ttg.convert_layout %11 : tensor<128x256xf16, #mma> -> tensor<128x256xf16, #blocked1>
132|       tt.descriptor_store %arg2[%c0_i32, %c0_i32], %12 : !tt.tensordesc<128x256xf16, #shared>, tensor<128x256xf16, #blocked1>
133|     }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining integer comparisons, structured conditionals, arith.truncf, layout conversions, tt.descriptor_store.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 整数比较、结构化条件分支、arith.truncf、布局转换、tt.descriptor_store。

### Lines 134-140
```mlir
134|     // CHECK: yield %{{.*}}, [[NEXT_LHS_BUF_IDX]], [[LHS_BUF_IDX]], [[LHS_PHASE]], [[NEXT_RHS_BUF_IDX]], [[RHS_BUF_IDX]], [[RHS_PHASE]]
135|     scf.yield %9 : tensor<128x256xf32, #mma>
136|   } {tt.num_stages = 4 : i32}
137|   tt.return
138| }
139| 
140| }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、tt.return。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `--tritongpu-assign-latencies`, `--tritongpu-schedule-loops`, `--tritongpu-pipeline`, `-canonicalize`, `-cse`
- **CN:** 主要 pass 选项：`--tritongpu-assign-latencies`，`--tritongpu-schedule-loops`，`--tritongpu-pipeline`，`-canonicalize`，`-cse`
- **EN:** Dominant operations include `arith.constant`, `tt.func`, `tt.descriptor_load`, `ttg.local_alloc`, `module`, `arith.subi`, `scf.for`, `ttg.memdesc_trans`, `ttng.warp_group_dot`, `arith.cmpi`.
- **CN:** 主要操作包括 `arith.constant`、`tt.func`、`tt.descriptor_load`、`ttg.local_alloc`、`module`、`arith.subi`、`scf.for`、`ttg.memdesc_trans`、`ttng.warp_group_dot`、`arith.cmpi`。
- **EN:** The file contains 1 independently testable section(s). Check styles used: CHECK-NEXT x47, CHECK x20, CHECK-SAME x7, CHECK-LABEL x1. Important labels include matmul_kernel_tma_persistent. It validates that the selected pass pipeline rewrites each labeled test case into the checked form.
- **CN:** 该文件包含 1 个可独立测试的分段。使用的检查类型：CHECK-NEXT ×47，CHECK ×20，CHECK-SAME ×7，CHECK-LABEL ×1。 关键标签包括 matmul_kernel_tma_persistent。 它验证所选 pass 流水线会把每个带标签的测试用例重写成检查所要求的形式。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `ttg`: TritonGPU layout, memory, and GPU-specific operations.
- **CN:** `ttg`：TritonGPU 布局、内存和 GPU 专用操作。
- **EN:** `tt`: Triton high-level tensor/pointer operations.
- **CN:** `tt`：Triton 高层张量/指针操作。
- **EN:** `arith`: MLIR arithmetic dialect for scalar/tensor math.
- **CN:** `arith`：用于标量/张量运算的 MLIR arithmetic 方言。
- **EN:** `scf`: Structured control flow for loops and conditionals.
- **CN:** `scf`：用于循环与条件分支的结构化控制流。
- **EN:** `ttng`: Triton Nvidia GPU tensor-memory and MMA operations.
- **CN:** `ttng`：Triton Nvidia GPU 张量内存与 MMA 操作。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。