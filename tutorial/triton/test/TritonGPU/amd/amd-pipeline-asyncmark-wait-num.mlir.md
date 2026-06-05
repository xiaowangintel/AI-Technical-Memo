# amd-pipeline-asyncmark-wait-num.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/amd/amd-pipeline-asyncmark-wait-num.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-tritonamdgpu-schedule-loops="num_stages=3", -tritonamdgpu-pipeline="use_async_copy=1", -canonicalize` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-tritonamdgpu-schedule-loops="num_stages=3", -tritonamdgpu-pipeline="use_async_copy=1", -canonicalize` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=3" -tritonamdgpu-pipeline="use_async_copy=1" -canonicalize | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=3" -tritonamdgpu-pipeline="use_async_copy=1" -canonicalize | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```mlir
1| // RUN: triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=3" -tritonamdgpu-pipeline="use_async_copy=1" -canonicalize | FileCheck %s
2| 
3| // On asyncmark targets (CDNA3/CDNA4) ttg.async_wait's `num` lowers straight to
4| // wait.asyncmark(N), so the pipeliner-authored num=0 ("wait for all") would
5| // serialize the SWP (PR #9883). Verify the pipeline pass runs updateWaits and
6| // rewrites the steady-state wait to a non-zero commit-group count.
7| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=3" -tritonamdgpu-pipeline="use_async_copy=1" -canonicalize | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=3" -tritonamdgpu-pipeline="use_async_copy=1" -canonicalize | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 8-9
```mlir
8| #blocked = #ttg.blocked<{sizePerThread = [2, 1], threadsPerWarp = [4, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
9| #mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [4, 1], instrShape = [16, 16, 4], isTransposed = true}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 10-10
```mlir
10| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 11-12
```mlir
11|   // CHECK-LABEL: async_wait_num_stages3
12|   // Single load, num_stages=3 -> 1 commit allowed in flight.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 13-15
```mlir
13|   // CHECK: scf.for
14|   // CHECK:   ttg.async_wait %{{.*}} {num = 1 : i32}
15|   // CHECK:   scf.yield
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 16-30
```mlir
16|   tt.func @async_wait_num_stages3(
17|       %arg0: tensor<16x32x!tt.ptr<f32>, #blocked> {tt.contiguity = dense<[1, 2]> : tensor<2xi32>, tt.divisibility = dense<16> : tensor<2xi32>},
18|       %arg1: tensor<32x32xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>>,
19|       %lb: i32, %ub: i32, %step: i32) -> tensor<16x32xf32, #mma> {
20|     %cst_acc = arith.constant dense<0.000000e+00> : tensor<16x32xf32, #mma>
21|     %result = scf.for %iv = %lb to %ub step %step iter_args(%acc = %cst_acc) -> (tensor<16x32xf32, #mma>) : i32 {
22|       %a = tt.load %arg0 : tensor<16x32x!tt.ptr<f32>, #blocked>
23|       %a_dot = ttg.convert_layout %a : tensor<16x32xf32, #blocked> -> tensor<16x32xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>>
24|       %c = tt.dot %a_dot, %arg1, %acc : tensor<16x32xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>> * tensor<32x32xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>> -> tensor<16x32xf32, #mma>
25|       scf.yield %c : tensor<16x32xf32, #mma>
26|     }
27|     tt.return %result : tensor<16x32xf32, #mma>
28|   }
29| }
30| 
```
**EN:** This function-oriented block defines or enters `async_wait_num_stages3`. Within it, the test exercises tt.func, constants, structured loops, masked or vectorized loads, layout conversions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_wait_num_stages3` 为核心。测试在其中演示 tt.func、常量、结构化循环、带掩码或向量化的加载、布局转换，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 31-31
```mlir
31| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 32-36
```mlir
32| 
33| // Two loads (gemm-shaped) at num_stages=3: each iteration emits two commit
34| // groups, the steady-state multi-token wait must allow both in flight, so
35| // updateWaits should derive num=2.
36| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 37-39
```mlir
37| #blockedA = #ttg.blocked<{sizePerThread = [2, 1], threadsPerWarp = [4, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
38| #blockedB = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [16, 4], warpsPerCTA = [1, 4], order = [0, 1]}>
39| #mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [4, 1], instrShape = [16, 16, 4], isTransposed = true}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 40-40
```mlir
40| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 41-44
```mlir
41|   // CHECK-LABEL: gemm_two_loads_stages3
42|   // CHECK: scf.for
43|   // CHECK:   ttg.async_wait %{{[^,]+}}, %{{[^,]+}} {num = 2 : i32}
44|   // CHECK:   scf.yield
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: gemm_two_loads_stages3 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: gemm_two_loads_stages3 这样的标签用于锚定匹配范围。

### Lines 45-60
```mlir
45|   tt.func @gemm_two_loads_stages3(
46|       %argA: tensor<16x32x!tt.ptr<f32>, #blockedA> {tt.contiguity = dense<[1, 2]> : tensor<2xi32>, tt.divisibility = dense<16> : tensor<2xi32>},
47|       %argB: tensor<32x32x!tt.ptr<f32>, #blockedB> {tt.contiguity = dense<[2, 1]> : tensor<2xi32>, tt.divisibility = dense<16> : tensor<2xi32>},
48|       %lb: i32, %ub: i32, %step: i32) -> tensor<16x32xf32, #mma> {
49|     %cst_acc = arith.constant dense<0.000000e+00> : tensor<16x32xf32, #mma>
50|     %result = scf.for %iv = %lb to %ub step %step iter_args(%acc = %cst_acc) -> (tensor<16x32xf32, #mma>) : i32 {
51|       %a = tt.load %argA : tensor<16x32x!tt.ptr<f32>, #blockedA>
52|       %b = tt.load %argB : tensor<32x32x!tt.ptr<f32>, #blockedB>
53|       %a_dot = ttg.convert_layout %a : tensor<16x32xf32, #blockedA> -> tensor<16x32xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>>
54|       %b_dot = ttg.convert_layout %b : tensor<32x32xf32, #blockedB> -> tensor<32x32xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>>
55|       %c = tt.dot %a_dot, %b_dot, %acc : tensor<16x32xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>> * tensor<32x32xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>> -> tensor<16x32xf32, #mma>
56|       scf.yield %c : tensor<16x32xf32, #mma>
57|     }
58|     tt.return %result : tensor<16x32xf32, #mma>
59|   }
60| }
```
**EN:** This function-oriented block defines or enters `gemm_two_loads_stages3`. Within it, the test exercises tt.func, masked or vectorized loads, layout conversions, constants, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `gemm_two_loads_stages3` 为核心。测试在其中演示 tt.func、带掩码或向量化的加载、布局转换、常量、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-tritonamdgpu-schedule-loops="num_stages=3"`, `-tritonamdgpu-pipeline="use_async_copy=1"`, `-canonicalize`
- **CN:** 主要 pass 选项：`-split-input-file`，`-tritonamdgpu-schedule-loops="num_stages=3"`，`-tritonamdgpu-pipeline="use_async_copy=1"`，`-canonicalize`
- **EN:** Dominant operations include `tt.func`, `tt.load`, `ttg.convert_layout`, `module`, `arith.constant`, `scf.for`, `tt.dot`, `scf.yield`, `tt.return`.
- **CN:** 主要操作包括 `tt.func`、`tt.load`、`ttg.convert_layout`、`module`、`arith.constant`、`scf.for`、`tt.dot`、`scf.yield`、`tt.return`。
- **EN:** The file contains 1 independently testable section(s). Check styles used: CHECK x6, CHECK-LABEL x2. Important labels include async_wait_num_stages3, gemm_two_loads_stages3. It validates that the selected pass pipeline rewrites each labeled test case into the checked form.
- **CN:** 该文件包含 1 个可独立测试的分段。使用的检查类型：CHECK ×6，CHECK-LABEL ×2。 关键标签包括 async_wait_num_stages3，gemm_two_loads_stages3。 它验证所选 pass 流水线会把每个带标签的测试用例重写成检查所要求的形式。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `tt`: Triton high-level tensor/pointer operations.
- **CN:** `tt`：Triton 高层张量/指针操作。
- **EN:** `ttg`: TritonGPU layout, memory, and GPU-specific operations.
- **CN:** `ttg`：TritonGPU 布局、内存和 GPU 专用操作。
- **EN:** `scf`: Structured control flow for loops and conditionals.
- **CN:** `scf`：用于循环与条件分支的结构化控制流。
- **EN:** `arith`: MLIR arithmetic dialect for scalar/tensor math.
- **CN:** `arith`：用于标量/张量运算的 MLIR arithmetic 方言。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。