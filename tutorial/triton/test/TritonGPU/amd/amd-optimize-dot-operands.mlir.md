# amd-optimize-dot-operands.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/amd/amd-optimize-dot-operands.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-tritonamdgpu-optimize-dot-operands="gfx-arch=gfx950"` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-tritonamdgpu-optimize-dot-operands="gfx-arch=gfx950"` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file -tritonamdgpu-optimize-dot-operands="gfx-arch=gfx950" | FileCheck %s --check-prefixes GFX950`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file -tritonamdgpu-optimize-dot-operands="gfx-arch=gfx950" | FileCheck %s --check-prefixes GFX950`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt %s -split-input-file -tritonamdgpu-optimize-dot-operands="gfx-arch=gfx950" | FileCheck %s --check-prefixes GFX950
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file -tritonamdgpu-optimize-dot-operands="gfx-arch=gfx950" | FileCheck %s --check-prefixes GFX950` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file -tritonamdgpu-optimize-dot-operands="gfx-arch=gfx950" | FileCheck %s --check-prefixes GFX950`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-13
```mlir
 3| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [16, 4], warpsPerCTA = [2, 1], order = [1, 0]}>
 4| #linear = #ttg.linear<{register = [[0, 0, 1], [0, 0, 2], [0, 0, 4], [1, 0, 0], [2, 0, 0], [0, 32, 0], [0, 64, 0]], lane = [[0, 1, 0], [0, 2, 0], [0, 4, 0], [0, 8, 0], [0, 0, 8], [0, 0, 16]], warp = [[0, 16, 0]], block = []}>
 5| #linear1 = #ttg.linear<{register = [[0, 1], [0, 2], [32, 0], [64, 0]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 0], [0, 0]], warp = [[16, 0]], block = []}>
 6| #linear2 = #ttg.linear<{register = [[0, 1, 0], [0, 2, 0], [0, 4, 0], [1, 0, 0], [2, 0, 0], [0, 0, 32], [0, 0, 64]], lane = [[0, 0, 1], [0, 0, 2], [0, 0, 4], [0, 0, 8], [0, 8, 0], [0, 16, 0]], warp = [[0, 0, 16]], block = []}>
 7| #mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [1, 2], instrShape = [16, 16], isTransposed = true}>
 8| // GFX950{LITERAL}: #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
 9| // GFX950-LABEL: test_alloc_shared_mem_for_scaled_upcast
10| // GFX950: %[[LOAD:.+]] = tt.load
11| // GFX950: %[[ALLOC:.+]] = ttg.local_alloc %[[LOAD]] : (tensor<128x4xi8, #blocked>) -> !ttg.memdesc<128x4xi8, #shared, #smem>
12| // GFX950: %[[LOCAL_LOAD:.+]] = ttg.local_load %[[ALLOC]] : !ttg.memdesc<128x4xi8, #shared, #smem> -> tensor<128x4xi8, #linear1>
13| // GFX950: tt.trans %[[LOCAL_LOAD]] {order = array<i32: 1, 0>}
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 14-14
```mlir
14| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 2 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 15-32
```mlir
15|   tt.func public @test_alloc_shared_mem_for_scaled_upcast(
16|     %arg0: tensor<128x4x!tt.ptr<i8>, #blocked>,
17|     %arg1: tensor<128x128xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>,
18|     %out: tensor<128x128x!tt.ptr<bf16>, #blocked>,
19|     %K: i32 {tt.divisibility = 16 : i32}
20|   ) {
21|       %c0_i32 = arith.constant 0 : i32
22|       %c128_i32 = arith.constant 128 : i32
23|       %cst_0 = arith.constant dense<7> : tensor<4x128xi16, #ttg.slice<{dim = 2, parent = #linear}>>
24|       %cst_1 = arith.constant dense<0.0> : tensor<128x128xbf16, #blocked>
25| 
26|       %14:1 = scf.for %13 = %c0_i32 to %K step %c128_i32 iter_args(%15 = %cst_1) -> (tensor<128x128xbf16, #blocked>) : i32 {
27|         %1 = tt.load %arg0 : tensor<128x4x!tt.ptr<i8>, #blocked>
28|         %2 = ttg.convert_layout %1 : tensor<128x4xi8, #blocked> -> tensor<128x4xi8, #linear1>
29|         %3 = tt.trans %2 {order = array<i32: 1, 0>} : tensor<128x4xi8, #linear1> -> tensor<4x128xi8, #ttg.slice<{dim = 2, parent = #linear}>>
30|         %4 = arith.extui %3 : tensor<4x128xi8, #ttg.slice<{dim = 2, parent = #linear}>> to tensor<4x128xi16, #ttg.slice<{dim = 2, parent = #linear}>>
31|         %5 = arith.shli %4, %cst_0 : tensor<4x128xi16, #ttg.slice<{dim = 2, parent = #linear}>>
32|         %6 = tt.bitcast %5 : tensor<4x128xi16, #ttg.slice<{dim = 2, parent = #linear}>> -> tensor<4x128xbf16, #ttg.slice<{dim = 2, parent = #linear}>>
```
**EN:** This function-oriented block defines or enters `test_alloc_shared_mem_for_scaled_upcast`. Within it, the test exercises constants, tt.func, structured loops, masked or vectorized loads, layout conversions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `test_alloc_shared_mem_for_scaled_upcast` 为核心。测试在其中演示 常量、tt.func、结构化循环、带掩码或向量化的加载、布局转换，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 33-45
```mlir
33|         %7 = tt.expand_dims %6 {axis = 2 : i32} : tensor<4x128xbf16, #ttg.slice<{dim = 2, parent = #linear}>> -> tensor<4x128x1xbf16, #linear>
34|         %8 = tt.broadcast %7 : tensor<4x128x1xbf16, #linear> -> tensor<4x128x32xbf16, #linear>
35|         %9 = tt.trans %8 {order = array<i32: 0, 2, 1>} : tensor<4x128x32xbf16, #linear> -> tensor<4x32x128xbf16, #linear2>
36|         %10 = tt.reshape %9 : tensor<4x32x128xbf16, #linear2> -> tensor<128x128xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>
37|         %11 = amdg.scaled_upcast_fp8 %arg1 scale %10 : tensor<128x128xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>, tensor<128x128xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>> -> tensor<128x128xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>
38|         %12 = ttg.convert_layout %11 : tensor<128x128xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>> -> tensor<128x128xbf16, #blocked>
39|         %16 = arith.addf %15, %12 : tensor<128x128xbf16, #blocked>
40|         scf.yield %16 : tensor<128x128xbf16, #blocked>
41|       }
42|       tt.store %out, %14#0 : tensor<128x128x!tt.ptr<bf16>, #blocked>
43|       tt.return
44|   }
45| }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining shape expansion, tensor broadcasting, transpose-like layout changes, tensor reshaping, amdg.scaled_upcast_fp8.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 形状扩展、张量广播、转置类布局变换、张量重塑、amdg.scaled_upcast_fp8。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-tritonamdgpu-optimize-dot-operands="gfx-arch=gfx950"`
- **CN:** 主要 pass 选项：`-split-input-file`，`-tritonamdgpu-optimize-dot-operands="gfx-arch=gfx950"`
- **EN:** Dominant operations include `arith.constant`, `tt.func`, `ttg.convert_layout`, `tt.trans`, `module`, `scf.for`, `tt.load`, `arith.extui`, `arith.shli`, `tt.bitcast`.
- **CN:** 主要操作包括 `arith.constant`、`tt.func`、`ttg.convert_layout`、`tt.trans`、`module`、`scf.for`、`tt.load`、`arith.extui`、`arith.shli`、`tt.bitcast`。
- **EN:** The file contains 1 independently testable section(s). The file relies mainly on the transformed IR itself rather than explicit FileCheck/diagnostic annotations. It validates the semantics or lowering pattern expressed by the IR in this test.
- **CN:** 该文件包含 1 个可独立测试的分段。该文件主要依赖变换后的 IR 本身，而不是显式的 FileCheck/诊断注解。 它验证该测试中 IR 所表达的语义或 lower 模式。

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