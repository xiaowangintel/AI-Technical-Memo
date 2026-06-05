# amd-stream-lds-layout-selection.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/amd/amd-stream-lds-layout-selection.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-tritonamdgpu-schedule-loops="num_stages=2", -tritonamdgpu-pipeline, -canonicalize` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-tritonamdgpu-schedule-loops="num_stages=2", -tritonamdgpu-pipeline, -canonicalize` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=2" -tritonamdgpu-pipeline -canonicalize | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=2" -tritonamdgpu-pipeline -canonicalize | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-3
```mlir
1| // RUN: triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=2" -tritonamdgpu-pipeline -canonicalize | FileCheck %s
2| 
3| // Pick a common shared memory layout with vec = max kWidth of all users.
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=2" -tritonamdgpu-pipeline -canonicalize | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=2" -tritonamdgpu-pipeline -canonicalize | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 4-21
```mlir
 4| // CHECK{LITERAL}: #shared = #ttg.swizzled_shared<{vec = 8, perPhase = 2, maxPhase = 8, order = [0, 1]}>
 5| // CHECK-NOT: #ttg.swizzled_shared
 6| // CHECK{LITERAL}: #smem = #ttg.shared_memory
 7| // CHECK-LABEL: test_lds_layout_selection
 8| 
 9| // CHECK: %[[ALLOC:.+]] = ttg.local_alloc : () -> !ttg.memdesc<1x64x16xf16, #shared, #smem, mutable>
10| // CHECK: %[[MEMDESC_IDX:.+]] = ttg.memdesc_index %[[ALLOC]]
11| 
12| // CHECK: scf.for {{.+}} iter_args({{.*}}, %[[MEMDESC_IDX_ITER:.+]] = %[[MEMDESC_IDX]]) -> ({{.+}})
13| //  CHECK: %[[LOAD:.+]] = tt.load {{.+}} : tensor<64x16x!tt.ptr<f16>, #blocked>
14| //  CHECK: %[[LOCAL_LOAD_TRANS:.+]] = ttg.local_load %[[MEMDESC_IDX_ITER]] : {{.+}} -> tensor<64x16xf16, #linear>
15| //  CHECK: %[[LOCAL_LOAD_DIRECT:.+]] = ttg.local_load %[[MEMDESC_IDX_ITER]] : {{.+}} -> tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
16| //  CHECK: tt.dot {{.+}}, %[[LOCAL_LOAD_DIRECT]], {{.+}}
17| //  CHECK: %[[TRANS:.+]] = tt.trans %[[LOCAL_LOAD_TRANS]] {{.+}} : {{.+}} -> tensor<16x64xf16, #ttg.dot_op<{opIdx = 1, parent = #mma1, kWidth = 8}>>
18| //  CHECK: tt.dot {{.+}}, %[[TRANS]], {{.+}}
19| //  CHECK: %[[MEMDESC_IDX:.+]] = ttg.memdesc_index %[[ALLOC]]
20| //  CHECK: ttg.local_store %[[LOAD]], %[[MEMDESC_IDX]]
21| //  CHECK: scf.yield
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 22-27
```mlir
22| 
23| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [64, 1], warpsPerCTA = [1, 4], order = [0, 1]}>
24| #linear = #ttg.linear<{register = [[0, 1], [0, 2], [0, 4], [32, 0]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0], [0, 8]], warp = [[0, 0], [0, 0]], block = []}>
25| #mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [4, 1], instrShape = [32, 32, 16], isTransposed = true}>
26| #mma1 = #ttg.amd_mfma<{version = 4, warpsPerCTA = [4, 1], instrShape = [16, 16, 32], isTransposed = true}>
27| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 28-28
```mlir
28| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 29-46
```mlir
29|   tt.func public @test_lds_layout_selection(
30|     %arg0: tensor<64x16x!tt.ptr<f16>, #blocked>,
31|     %out0 : tensor<128x16x!tt.ptr<f32>, #blocked>,
32|     %out1 : tensor<128x64x!tt.ptr<f32>, #blocked>
33|   ) {
34|     %cst_0 = arith.constant dense<0.000000e+00> : tensor<128x16xf32, #mma1>
35|     %cst_1 = arith.constant dense<0.693147182> : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma1, kWidth = 4}>>
36|     %cst_2 = arith.constant dense<0.581374812> : tensor<128x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>
37|     %cst_3 = arith.constant dense<0.000000e+00> : tensor<128x64xf32, #mma>
38|     %c0_i32 = arith.constant 0 : i32
39|     %c1_i32 = arith.constant 1 : i32
40|     %c8_i32 = arith.constant 8 : i32
41| 
42|     %0:2 = scf.for %arg1 = %c0_i32 to %c8_i32 step %c1_i32 iter_args(%arg2 = %cst_0, %arg3 = %cst_3) -> (tensor<128x16xf32, #mma1>, tensor<128x64xf32, #mma>)  : i32 {
43|       %1 = tt.load %arg0 : tensor<64x16x!tt.ptr<f16>, #blocked>
44|       %2 = ttg.convert_layout %1 : tensor<64x16xf16, #blocked> -> tensor<64x16xf16, #linear>
45|       %3 = ttg.convert_layout %1 : tensor<64x16xf16, #blocked> -> tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma1, kWidth = 4}>>
46|       %4 = tt.dot %cst_1, %3, %arg2 : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma1, kWidth = 4}>> * tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma1, kWidth = 4}>> -> tensor<128x16xf32, #mma1>
```
**EN:** This function-oriented block defines or enters `test_lds_layout_selection`. Within it, the test exercises constants, tt.func, layout conversions, structured loops, masked or vectorized loads, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `test_lds_layout_selection` 为核心。测试在其中演示 常量、tt.func、布局转换、结构化循环、带掩码或向量化的加载，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 47-58
```mlir
47|       %5 = tt.trans %2 {order = array<i32: 1, 0>} : tensor<64x16xf16, #linear> -> tensor<16x64xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>
48|       %6 = tt.dot %cst_2, %5, %arg3 : tensor<128x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>> * tensor<16x64xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>> -> tensor<128x64xf32, #mma>
49|       scf.yield %4, %6 : tensor<128x16xf32, #mma1>, tensor<128x64xf32, #mma>
50|     }
51| 
52|     %7 = ttg.convert_layout %0#0 : tensor<128x16xf32, #mma1> -> tensor<128x16xf32, #blocked>
53|     %8 = ttg.convert_layout %0#1 : tensor<128x64xf32, #mma> -> tensor<128x64xf32, #blocked>
54|     tt.store %out0, %7 : tensor<128x16x!tt.ptr<f32>, #blocked>
55|     tt.store %out1, %8 : tensor<128x64x!tt.ptr<f32>, #blocked>
56|     tt.return
57|   }
58| }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining layout conversions, masked or vectorized stores, transpose-like layout changes, dot-product or MMA-style math, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 布局转换、带掩码或向量化的存储、转置类布局变换、点积或 MMA 风格计算、循环/分支产出值。

### Lines 59-59
```mlir
59| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 60-61
```mlir
60| 
61| // Verify that a common shared memory layout is chosen for users with different kWidth and opIdx.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 62-79
```mlir
62| // CHECK{LITERAL}: #shared = #ttg.swizzled_shared<{vec = 8, perPhase = 2, maxPhase = 8, order = [0, 1]}>
63| // CHECK-NOT: #ttg.swizzled_shared
64| // CHECK{LITERAL}: #smem = #ttg.shared_memory
65| // CHECK-LABEL: test_lds_layout_selection_different_opIdx
66| 
67| // CHECK: %[[ALLOC:.+]] = ttg.local_alloc : () -> !ttg.memdesc<1x64x16xf16, #shared, #smem, mutable>
68| // CHECK: %[[MEMDESC_IDX:.+]] = ttg.memdesc_index %[[ALLOC]]
69| 
70| // CHECK: scf.for {{.+}} iter_args({{.*}}, %[[MEMDESC_IDX_ITER:.+]] = %[[MEMDESC_IDX]]) -> ({{.+}})
71| //  CHECK: %[[LOAD:.+]] = tt.load {{.+}} : tensor<64x16x!tt.ptr<f16>, #blocked>
72| //  CHECK: %[[LOCAL_LOAD_TRANS:.+]] = ttg.local_load %[[MEMDESC_IDX_ITER]] : {{.+}} -> tensor<64x16xf16, #linear>
73| //  CHECK: %[[LOCAL_LOAD_DIRECT:.+]] = ttg.local_load %[[MEMDESC_IDX_ITER]] : {{.+}} -> tensor<64x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>
74| //  CHECK: tt.dot %[[LOCAL_LOAD_DIRECT]], {{.+}}
75| //  CHECK: %[[TRANS:.+]] = tt.trans %[[LOCAL_LOAD_TRANS]] {{.+}} : {{.+}} -> tensor<16x64xf16, #ttg.dot_op<{opIdx = 1, parent = #mma1, kWidth = 8}>>
76| //  CHECK: tt.dot {{.+}}, %[[TRANS]], {{.+}}
77| //  CHECK: %[[MEMDESC_IDX:.+]] = ttg.memdesc_index %[[ALLOC]]
78| //  CHECK: ttg.local_store %[[LOAD]], %[[MEMDESC_IDX]]
79| //  CHECK: scf.yield
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 80-85
```mlir
80| 
81| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [64, 1], warpsPerCTA = [1, 4], order = [0, 1]}>
82| #linear = #ttg.linear<{register = [[0, 1], [0, 2], [0, 4], [32, 0]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0], [0, 8]], warp = [[0, 0], [0, 0]], block = []}>
83| #mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [4, 1], instrShape = [32, 32, 16], isTransposed = true}>
84| #mma1 = #ttg.amd_mfma<{version = 4, warpsPerCTA = [4, 1], instrShape = [16, 16, 32], isTransposed = true}>
85| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 86-86
```mlir
86| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 87-104
```mlir
 87|   tt.func public @test_lds_layout_selection_different_opIdx(
 88|     %arg0: tensor<64x16x!tt.ptr<f16>, #blocked>,
 89|     %out0 : tensor<64x64x!tt.ptr<f32>, #blocked>,
 90|     %out1 : tensor<128x64x!tt.ptr<f32>, #blocked>
 91|   ) {
 92|     %cst_0 = arith.constant dense<0.000000e+00> : tensor<64x64xf32, #mma1>
 93|     %cst_1 = arith.constant dense<0.693147182> : tensor<16x64xf16, #ttg.dot_op<{opIdx = 1, parent = #mma1, kWidth = 4}>>
 94|     %cst_2 = arith.constant dense<0.581374812> : tensor<128x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>
 95|     %cst_3 = arith.constant dense<0.000000e+00> : tensor<128x64xf32, #mma>
 96|     %c0_i32 = arith.constant 0 : i32
 97|     %c1_i32 = arith.constant 1 : i32
 98|     %c8_i32 = arith.constant 8 : i32
 99| 
100|     %0:2 = scf.for %arg1 = %c0_i32 to %c8_i32 step %c1_i32 iter_args(%arg2 = %cst_0, %arg3 = %cst_3) -> (tensor<64x64xf32, #mma1>, tensor<128x64xf32, #mma>)  : i32 {
101|       %1 = tt.load %arg0 : tensor<64x16x!tt.ptr<f16>, #blocked>
102|       %2 = ttg.convert_layout %1 : tensor<64x16xf16, #blocked> -> tensor<64x16xf16, #linear>
103|       %3 = ttg.convert_layout %1 : tensor<64x16xf16, #blocked> -> tensor<64x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma1, kWidth = 4}>>
104|       %4 = tt.dot %3, %cst_1, %arg2 : tensor<64x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma1, kWidth = 4}>> * tensor<16x64xf16, #ttg.dot_op<{opIdx = 1, parent = #mma1, kWidth = 4}>> -> tensor<64x64xf32, #mma1>
```
**EN:** This function-oriented block defines or enters `test_lds_layout_selection_different_opIdx`. Within it, the test exercises constants, tt.func, layout conversions, structured loops, masked or vectorized loads, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `test_lds_layout_selection_different_opIdx` 为核心。测试在其中演示 常量、tt.func、布局转换、结构化循环、带掩码或向量化的加载，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 105-116
```mlir
105|       %5 = tt.trans %2 {order = array<i32: 1, 0>} : tensor<64x16xf16, #linear> -> tensor<16x64xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>
106|       %6 = tt.dot %cst_2, %5, %arg3 : tensor<128x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>> * tensor<16x64xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>> -> tensor<128x64xf32, #mma>
107|       scf.yield %4, %6 : tensor<64x64xf32, #mma1>, tensor<128x64xf32, #mma>
108|     }
109| 
110|     %7 = ttg.convert_layout %0#0 : tensor<64x64xf32, #mma1> -> tensor<64x64xf32, #blocked>
111|     %8 = ttg.convert_layout %0#1 : tensor<128x64xf32, #mma> -> tensor<128x64xf32, #blocked>
112|     tt.store %out0, %7 : tensor<64x64x!tt.ptr<f32>, #blocked>
113|     tt.store %out1, %8 : tensor<128x64x!tt.ptr<f32>, #blocked>
114|     tt.return
115|   }
116| }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining layout conversions, masked or vectorized stores, transpose-like layout changes, dot-product or MMA-style math, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 布局转换、带掩码或向量化的存储、转置类布局变换、点积或 MMA 风格计算、循环/分支产出值。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-tritonamdgpu-schedule-loops="num_stages=2"`, `-tritonamdgpu-pipeline`, `-canonicalize`
- **CN:** 主要 pass 选项：`-split-input-file`，`-tritonamdgpu-schedule-loops="num_stages=2"`，`-tritonamdgpu-pipeline`，`-canonicalize`
- **EN:** Dominant operations include `arith.constant`, `ttg.convert_layout`, `tt.func`, `tt.dot`, `tt.store`, `module`, `scf.for`, `tt.load`, `tt.trans`, `scf.yield`.
- **CN:** 主要操作包括 `arith.constant`、`ttg.convert_layout`、`tt.func`、`tt.dot`、`tt.store`、`module`、`scf.for`、`tt.load`、`tt.trans`、`scf.yield`。
- **EN:** The file contains 1 independently testable section(s). Check styles used: CHECK x28, CHECK-NOT x2, CHECK-LABEL x2. Important labels include test_lds_layout_selection, test_lds_layout_selection_different_opIdx. It validates both the presence of the desired rewrite and the absence of unwanted residual operations.
- **CN:** 该文件包含 1 个可独立测试的分段。使用的检查类型：CHECK ×28，CHECK-NOT ×2，CHECK-LABEL ×2。 关键标签包括 test_lds_layout_selection，test_lds_layout_selection_different_opIdx。 它同时验证目标重写已经出现，并且不再残留不需要的操作。

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
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。