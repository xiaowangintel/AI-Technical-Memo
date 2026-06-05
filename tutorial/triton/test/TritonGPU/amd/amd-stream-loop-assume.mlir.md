# amd-stream-loop-assume.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/amd/amd-stream-loop-assume.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-tritonamdgpu-schedule-loops="num_stages=2", -tritonamdgpu-pipeline, -canonicalize` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-tritonamdgpu-schedule-loops="num_stages=2", -tritonamdgpu-pipeline, -canonicalize` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=2" -tritonamdgpu-pipeline -canonicalize | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=2" -tritonamdgpu-pipeline -canonicalize | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-3
```mlir
1| // RUN: triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=2" -tritonamdgpu-pipeline -canonicalize | FileCheck %s
2| 
3| // matmul: 128x32 @ 32x128 -> 128x128
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=2" -tritonamdgpu-pipeline -canonicalize | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=2" -tritonamdgpu-pipeline -canonicalize | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 4-12
```mlir
 4| #AL = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
 5| #BL = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
 6| #ALs0 = #ttg.slice<{parent=#AL, dim=0}>
 7| #BLs0 = #ttg.slice<{parent=#BL, dim=0}>
 8| #BLs1 = #ttg.slice<{parent=#BL, dim=1}>
 9| #C = #ttg.amd_mfma<{version = 4, warpsPerCTA = [4, 1], instrShape = [32, 32, 16], isTransposed = true}>
10| #A = #ttg.dot_op<{opIdx = 0, parent = #C, kWidth = 4}>
11| #B = #ttg.dot_op<{opIdx = 1, parent = #C, kWidth = 4}>
12| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 13-29
```mlir
13| // CHECK-LABEL: tt.func @assume_matmul
14| // CHECK-COUNT-2: tt.load
15| // CHECK-COUNT-2: ttg.local_store
16| // CHECK: scf.for
17| // CHECK: llvm.intr.assume
18| // CHECK: tt.load
19| // CHECK: ttg.local_load
20| // CHECK: tt.load
21| // CHECK: ttg.local_load
22| // CHECK: tt.dot
23| // CHECK-COUNT-2: ttg.local_store
24| // CHECK: scf.yield
25| // CHECK: llvm.intr.assume
26| // CHECK-COUNT-2: ttg.local_load
27| // CHECK: tt.dot
28| // CHECK-NOT: tt.dot
29| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tt.func @assume_matmul anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tt.func @assume_matmul 这样的标签用于锚定匹配范围。

### Lines 30-30
```mlir
30| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 31-48
```mlir
31| tt.func @assume_matmul(%lb : index, %ub : index, %step : index,
32|                   %A : !tt.ptr<f16> {tt.divisibility = 16 : i32},
33|                   %B : !tt.ptr<f16> {tt.divisibility = 16 : i32}) -> tensor<128x128xf32, #C> {
34|   // A ptrs
35|   %a_ptr_splat = tt.splat %A : !tt.ptr<f16> -> tensor<128x32x!tt.ptr<f16>, #AL>
36|   %a_tmp0 = tt.make_range {end = 32: i32, start = 0: i32} : tensor<32xi32, #ALs0>
37|   %a_tmp1 = tt.expand_dims %a_tmp0 {axis = 0 : i32} : tensor<32xi32, #ALs0> -> tensor<1x32xi32, #AL>
38|   %a_offs = tt.broadcast %a_tmp1 : tensor<1x32xi32, #AL> -> tensor<128x32xi32, #AL>
39|   %a_ptr_init = tt.addptr %a_ptr_splat, %a_offs : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xi32, #AL>
40|   // B ptrs
41|   %b_ptr_splat = tt.splat %B : !tt.ptr<f16> -> tensor<32x128x!tt.ptr<f16>, #BL>
42|   %b_tmp0 = tt.make_range {end = 128: i32, start = 0: i32} : tensor<128xi32, #BLs0>
43|   %b_tmp1 = tt.expand_dims %b_tmp0 {axis = 0 : i32} : tensor<128xi32, #BLs0> -> tensor<1x128xi32, #BL>
44|   %b_offs = tt.broadcast %b_tmp1 : tensor<1x128xi32, #BL> -> tensor<32x128xi32, #BL>
45|   %b_ptr_init = tt.addptr %b_ptr_splat, %b_offs : tensor<32x128x!tt.ptr<f16>, #BL>, tensor<32x128xi32, #BL>
46| 
47| 
48|   %a_mask = arith.constant dense<true> : tensor<128x32xi1, #AL>
```
**EN:** This function-oriented block defines or enters `assume_matmul`. Within it, the test exercises tt.func, broadcasted scalars or pointers, lane/block index ranges, shape expansion, tensor broadcasting, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `assume_matmul` 为核心。测试在其中演示 tt.func、广播后的标量或指针、lane/block 索引范围、形状扩展、张量广播，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 49-66
```mlir
49|   %a_other = arith.constant dense<0.00e+00> : tensor<128x32xf16, #AL>
50|   %b_mask = arith.constant dense<true> : tensor<32x128xi1, #BL>
51|   %b_other = arith.constant dense<0.00e+00> : tensor<32x128xf16, #BL>
52|   %c_init = arith.constant dense<0.00e+00> : tensor<128x128xf32, #C>
53| 
54|   %a_off = arith.constant dense<4> : tensor<128x32xi32, #AL>
55|   %b_off = arith.constant dense<4> : tensor<32x128xi32, #BL>
56| 
57|   %b_scale = arith.constant dense<4.> : tensor<32x128xf16, #B>
58|   %c_true = arith.constant 1: i1
59| 
60|   %loop:3 = scf.for %iv = %lb to %ub step %step iter_args(%a_ptr = %a_ptr_init, %b_ptr = %b_ptr_init, %prev_c = %c_init) -> (tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>) {
61|     // Note: This isn't a meaningful assumption here, but it acts
62|     // as a placeholder for a user generated assume in a loop.
63|     llvm.intr.assume %c_true : i1
64|     %a_ = tt.load %a_ptr : tensor<128x32x!tt.ptr<f16>, #AL>
65|     %a = ttg.convert_layout %a_ : tensor<128x32xf16, #AL> -> tensor<128x32xf16, #A>
66|     %b__ = tt.load %b_ptr, %b_mask, %b_other : tensor<32x128x!tt.ptr<f16>, #BL>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining constants, masked or vectorized loads, structured loops, llvm.intr, layout conversions.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 常量、带掩码或向量化的加载、结构化循环、llvm.intr、布局转换。

### Lines 67-78
```mlir
67|     %b_ = ttg.convert_layout %b__ : tensor<32x128xf16, #BL> -> tensor<32x128xf16, #B>
68|     %b = arith.mulf %b_, %b_scale: tensor<32x128xf16, #B>
69| 
70|     %c = tt.dot %a, %b, %prev_c : tensor<128x32xf16, #A> * tensor<32x128xf16, #B> -> tensor<128x128xf32, #C>
71| 
72|     %next_a_ptr = tt.addptr %a_ptr, %a_off : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xi32, #AL>
73|     %next_b_ptr = tt.addptr %b_ptr, %b_off : tensor<32x128x!tt.ptr<f16>, #BL>, tensor<32x128xi32, #BL>
74|     scf.yield %next_a_ptr, %next_b_ptr, %c : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>
75|   }
76|   tt.return %loop#2: tensor<128x128xf32, #C>
77| }
78| }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining pointer arithmetic, layout conversions, arith.mulf, dot-product or MMA-style math, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 指针算术、布局转换、arith.mulf、点积或 MMA 风格计算、循环/分支产出值。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-tritonamdgpu-schedule-loops="num_stages=2"`, `-tritonamdgpu-pipeline`, `-canonicalize`
- **CN:** 主要 pass 选项：`-split-input-file`，`-tritonamdgpu-schedule-loops="num_stages=2"`，`-tritonamdgpu-pipeline`，`-canonicalize`
- **EN:** Dominant operations include `arith.constant`, `tt.addptr`, `tt.func`, `tt.splat`, `tt.make_range`, `tt.expand_dims`, `tt.broadcast`, `tt.load`, `ttg.convert_layout`, `module`.
- **CN:** 主要操作包括 `arith.constant`、`tt.addptr`、`tt.func`、`tt.splat`、`tt.make_range`、`tt.expand_dims`、`tt.broadcast`、`tt.load`、`ttg.convert_layout`、`module`。
- **EN:** The file contains 1 independently testable section(s). Check styles used: CHECK x10, CHECK-COUNT x4, CHECK-LABEL x1, CHECK-NOT x1. Important labels include tt.func @assume_matmul. It validates both the presence of the desired rewrite and the absence of unwanted residual operations.
- **CN:** 该文件包含 1 个可独立测试的分段。使用的检查类型：CHECK ×10，CHECK-COUNT ×4，CHECK-LABEL ×1，CHECK-NOT ×1。 关键标签包括 tt.func @assume_matmul。 它同时验证目标重写已经出现，并且不再残留不需要的操作。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `tt`: Triton high-level tensor/pointer operations.
- **CN:** `tt`：Triton 高层张量/指针操作。
- **EN:** `ttg`: TritonGPU layout, memory, and GPU-specific operations.
- **CN:** `ttg`：TritonGPU 布局、内存和 GPU 专用操作。
- **EN:** `arith`: MLIR arithmetic dialect for scalar/tensor math.
- **CN:** `arith`：用于标量/张量运算的 MLIR arithmetic 方言。
- **EN:** `scf`: Structured control flow for loops and conditionals.
- **CN:** `scf`：用于循环与条件分支的结构化控制流。
- **EN:** `llvm`: LLVM-style low-level operations and types.
- **CN:** `llvm`：LLVM 风格底层操作与类型。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。