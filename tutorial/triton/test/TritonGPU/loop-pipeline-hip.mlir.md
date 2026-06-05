# loop-pipeline-hip.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/loop-pipeline-hip.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-tritonamdgpu-schedule-loops=num_stages=2, -tritonamdgpu-pipeline, -canonicalize, -tritonamdgpu-schedule-loops="num_stages=2"` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-tritonamdgpu-schedule-loops=num_stages=2, -tritonamdgpu-pipeline, -canonicalize, -tritonamdgpu-schedule-loops="num_stages=2"` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file -tritonamdgpu-schedule-loops=num_stages=2 -tritonamdgpu-pipeline -canonicalize | FileCheck %s --check-prefixes=COMMON,SYNC`; `// RUN: triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=2" -tritonamdgpu-pipeline="use_async_copy=1" -canonicalize | FileCheck %s --check-prefixes=COMMON,ASYNC`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file -tritonamdgpu-schedule-loops=num_stages=2 -tritonamdgpu-pipeline -canonicalize | FileCheck %s --check-prefixes=COMMON,SYNC`；`// RUN: triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=2" -tritonamdgpu-pipeline="use_async_copy=1" -canonicalize | FileCheck %s --check-prefixes=COMMON,ASYNC`

## Line-by-Line Analysis / 逐行分析

### Lines 1-3
```mlir
1| // RUN: triton-opt %s -split-input-file -tritonamdgpu-schedule-loops=num_stages=2 -tritonamdgpu-pipeline -canonicalize | FileCheck %s --check-prefixes=COMMON,SYNC
2| // RUN: triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=2" -tritonamdgpu-pipeline="use_async_copy=1" -canonicalize | FileCheck %s --check-prefixes=COMMON,ASYNC
3| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file -tritonamdgpu-schedule-loops=num_stages=2 -tritonamdgpu-pipeline -canonicalize | FileCheck %s --check-prefixes=COMMON,SYNC` ; ` triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=2" -tritonamdgpu-pipeline="use_async_copy=1" -canonicalize | FileCheck %s --check-prefixes=COMMON,ASYNC` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file -tritonamdgpu-schedule-loops=num_stages=2 -tritonamdgpu-pipeline -canonicalize | FileCheck %s --check-prefixes=COMMON,SYNC`；` triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=2" -tritonamdgpu-pipeline="use_async_copy=1" -canonicalize | FileCheck %s --check-prefixes=COMMON,ASYNC`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 4-9
```mlir
4| #blocked = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [8, 4], warpsPerCTA = [1, 4], order = [0, 1]}>
5| #blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
6| #mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 8]}>
7| #shared = #ttg.swizzled_shared<{vec = 8, perPhase = 1, maxPhase = 8, order = [0, 1]}>
8| #shared1 = #ttg.swizzled_shared<{vec = 8, perPhase = 1, maxPhase = 8, order = [1, 0]}>
9| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 10-10
```mlir
10| module attributes {"ttg.target" = "hip:gfx942", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 11-11
```mlir
11|   // COMMON-LABEL: tt.func @load_two_users
```
**EN:** This function-oriented block defines or enters `load_two_users`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `load_two_users` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 12-29
```mlir
12|   tt.func @load_two_users(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32}) -> (tensor<128x16xf32, #mma>, tensor<128x64xf32, #mma>) {
13|     %cst = arith.constant dense<0> : tensor<1x16xi32, #blocked>
14|     %cst_0 = arith.constant dense<0> : tensor<128x1xi32, #blocked1>
15|     %c0_i64 = arith.constant 0 : i64
16|     %c0_i32 = arith.constant 0 : i32
17|     %cst_1 = arith.constant dense<0.000000e+00> : tensor<128x16xf32, #mma>
18|     %cst_2 = arith.constant dense<0.000000e+00> : tensor<128x64xf32, #mma>
19|     %c1_i32 = arith.constant 1 : i32
20|     %c8_i32 = arith.constant 8 : i32
21|     %0 = tt.addptr %arg0, %c0_i64 : !tt.ptr<f16>, i64
22|     %1 = tt.addptr %arg1, %c0_i64 : !tt.ptr<f16>, i64
23|     %2 = tt.splat %1 : !tt.ptr<f16> -> tensor<128x1x!tt.ptr<f16>, #blocked1>
24|     %3 = tt.addptr %2, %cst_0 : tensor<128x1x!tt.ptr<f16>, #blocked1>, tensor<128x1xi32, #blocked1>
25|     %4 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
26|     %5 = tt.expand_dims %4 {axis = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked1}>> -> tensor<1x64xi32, #blocked1>
27|     %6 = tt.broadcast %3 : tensor<128x1x!tt.ptr<f16>, #blocked1> -> tensor<128x64x!tt.ptr<f16>, #blocked1>
28|     %7 = tt.broadcast %5 : tensor<1x64xi32, #blocked1> -> tensor<128x64xi32, #blocked1>
29|     %8 = tt.addptr %6, %7 : tensor<128x64x!tt.ptr<f16>, #blocked1>, tensor<128x64xi32, #blocked1>
```
**EN:** This function-oriented block defines or enters `load_two_users`. Within it, the test exercises constants, pointer arithmetic, tt.func, tensor broadcasting, broadcasted scalars or pointers, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `load_two_users` 为核心。测试在其中演示 常量、指针算术、tt.func、张量广播、广播后的标量或指针，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 30-47
```mlir
30|     %9 = tt.load %8 : tensor<128x64x!tt.ptr<f16>, #blocked1>
31|     %10 = tt.splat %0 : !tt.ptr<f16> -> tensor<1x16x!tt.ptr<f16>, #blocked>
32|     %11 = tt.addptr %10, %cst : tensor<1x16x!tt.ptr<f16>, #blocked>, tensor<1x16xi32, #blocked>
33|     %12 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
34|     %13 = tt.expand_dims %12 {axis = 1 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<64x1xi32, #blocked>
35|     %14 = tt.broadcast %11 : tensor<1x16x!tt.ptr<f16>, #blocked> -> tensor<64x16x!tt.ptr<f16>, #blocked>
36|     %15 = tt.broadcast %13 : tensor<64x1xi32, #blocked> -> tensor<64x16xi32, #blocked>
37|     %16 = tt.addptr %14, %15 : tensor<64x16x!tt.ptr<f16>, #blocked>, tensor<64x16xi32, #blocked>
38|     // SYNC: ttg.local_store
39|     // SYNC: scf.for
40|     // SYNC:   tt.load
41|     // SYNC:   tt.dot
42|     // SYNC:   tt.dot
43|     // SYNC:   ttg.local_store
44|     // SYNC:   scf.yield
45| 
46|     // ASYNC: ttg.async_copy_global_to_local
47|     // ASYNC: scf.for
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining pointer arithmetic, tensor broadcasting, masked or vectorized loads, broadcasted scalars or pointers, lane/block index ranges.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 指针算术、张量广播、带掩码或向量化的加载、广播后的标量或指针、lane/block 索引范围。

### Lines 48-65
```mlir
48|     // ASYNC:  ttg.async_wait
49|     // ASYNC:  ttg.async_copy_global_to_local
50|     // ASYNC:  tt.dot
51|     // ASYNC:  tt.dot
52|     // ASYNC:  scf.yield
53|     %17:2 = scf.for %arg2 = %c0_i32 to %c8_i32 step %c1_i32 iter_args(%arg3 = %cst_1, %arg4 = %cst_2) -> (tensor<128x16xf32, #mma>, tensor<128x64xf32, #mma>)  : i32 {
54|       %18 = tt.load %16 : tensor<64x16x!tt.ptr<f16>, #blocked>
55|       %19 = ttg.convert_layout %9 : tensor<128x64xf16, #blocked1> -> tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
56|       %20 = ttg.convert_layout %18 : tensor<64x16xf16, #blocked> -> tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>
57|       %21 = tt.dot %19, %20, %cst_1 : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<128x16xf32, #mma>
58|       %22 = arith.truncf %21 : tensor<128x16xf32, #mma> to tensor<128x16xf16, #mma>
59|       %23 = ttg.convert_layout %22 : tensor<128x16xf16, #mma> -> tensor<128x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
60|       %24 = ttg.local_alloc %18 : (tensor<64x16xf16, #blocked>) -> !ttg.memdesc<64x16xf16, #shared, #smem, mutable>
61|       %25 = ttg.memdesc_trans %24 {order=array<i32: 1,0>} : !ttg.memdesc<64x16xf16, #shared, #smem, mutable> -> !ttg.memdesc<16x64xf16, #shared1, #smem, mutable>
62|       %26 = ttg.local_load %25 : !ttg.memdesc<16x64xf16, #shared1, #smem, mutable> -> tensor<16x64xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>
63|       %27 = tt.dot %23, %26, %arg4 : tensor<128x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<16x64xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<128x64xf32, #mma>
64|       scf.yield %21, %27 : tensor<128x16xf32, #mma>, tensor<128x64xf32, #mma>
65|     }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining layout conversions, dot-product or MMA-style math, structured loops, masked or vectorized loads, arith.truncf.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 布局转换、点积或 MMA 风格计算、结构化循环、带掩码或向量化的加载、arith.truncf。

### Lines 66-69
```mlir
66|     tt.return %17#0, %17#1 : tensor<128x16xf32, #mma>, tensor<128x64xf32, #mma>
67|   }
68| }
69| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 70-70
```mlir
70| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 71-74
```mlir
71| 
72| // COMMON-LABEL: tt.func public @_jagged_hstu_attn_fwd_0d1d2d3d4d5de
73| // COMMON-NOT:  ttg.convert_layout {{.*}} : tensor<32x64xf32, #shared> -> tensor<32x64xf32, #shared1>
74| 
```
**EN:** This function-oriented block defines or enters `_jagged_hstu_attn_fwd_0d1d2d3d4d5de`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `_jagged_hstu_attn_fwd_0d1d2d3d4d5de` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 75-80
```mlir
75| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [2, 2], order = [0, 1]}>
76| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [0, 1]}>
77| #mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 8]}>
78| #shared = #ttg.swizzled_shared<{vec = 8, perPhase = 1, maxPhase = 4, order = [0, 1]}>
79| #shared1 = #ttg.swizzled_shared<{vec = 8, perPhase = 1, maxPhase = 4, order = [1, 0]}>
80| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 81-81
```mlir
81| module attributes {"ttg.target" = "hip:gfx942", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 82-99
```mlir
82|   tt.func public @_jagged_hstu_attn_fwd_0d1d2d3d4d5de(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg2: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg3: !tt.ptr<i64> {tt.divisibility = 16 : i32}, %arg4: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg5: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 8 : i32}) {
83|     %cst = arith.constant dense<0.000000e+00> : tensor<64x32xf32, #mma>
84|     %c64_i32 = arith.constant 64 : i32
85|     %c0_i32 = arith.constant 0 : i32
86|     %c32_i32 = arith.constant 32 : i32
87|     %0 = tt.get_program_id x : i32
88|     %1 = arith.muli %0, %c64_i32 : i32
89|     %2 = tt.get_program_id y : i32
90|     %3 = tt.load %arg3 : !tt.ptr<i64>
91|     %4 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
92|     %5 = tt.splat %1 : i32 -> tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
93|     %6 = arith.addi %5, %4 : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
94|     %7 = tt.expand_dims %6 {axis = 1 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<64x1xi32, #blocked>
95|     %8 = tt.splat %3 : i64 -> tensor<64x1xi64, #blocked>
96|     %9 = arith.extsi %7 : tensor<64x1xi32, #blocked> to tensor<64x1xi64, #blocked>
97|     %10 = arith.addi %8, %9 : tensor<64x1xi64, #blocked>
98|     %11 = arith.extsi %arg5 : i32 to i64
99|     %12 = tt.splat %11 : i64 -> tensor<64x1xi64, #blocked>
```
**EN:** This function-oriented block defines or enters `_jagged_hstu_attn_fwd_0d1d2d3d4d5de`. Within it, the test exercises constants, broadcasted scalars or pointers, tt.func, program IDs, integer additions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `_jagged_hstu_attn_fwd_0d1d2d3d4d5de` 为核心。测试在其中演示 常量、广播后的标量或指针、tt.func、程序 ID、整数加法，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 100-117
```mlir
100|     %13 = arith.muli %10, %12 : tensor<64x1xi64, #blocked>
101|     %14 = arith.muli %2, %arg5 : i32
102|     %15 = arith.extsi %14 : i32 to i64
103|     %16 = tt.splat %15 : i64 -> tensor<64x1xi64, #blocked>
104|     %17 = arith.addi %13, %16 : tensor<64x1xi64, #blocked>
105|     %18 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
106|     %19 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
107|     %20 = tt.expand_dims %18 {axis = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x64xi32, #blocked>
108|     %21 = tt.expand_dims %19 {axis = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked1}>> -> tensor<1x64xi32, #blocked1>
109|     %22 = tt.splat %arg5 : i32 -> tensor<1x64xi32, #blocked>
110|     %23 = tt.splat %arg5 : i32 -> tensor<1x64xi32, #blocked1>
111|     %24 = arith.muli %20, %22 : tensor<1x64xi32, #blocked>
112|     %25 = arith.muli %21, %23 : tensor<1x64xi32, #blocked1>
113|     %26 = tt.broadcast %17 : tensor<64x1xi64, #blocked> -> tensor<64x64xi64, #blocked>
114|     %27 = arith.extsi %24 : tensor<1x64xi32, #blocked> to tensor<1x64xi64, #blocked>
115|     %28 = arith.extsi %25 : tensor<1x64xi32, #blocked1> to tensor<1x64xi64, #blocked1>
116|     %29 = tt.broadcast %27 : tensor<1x64xi64, #blocked> -> tensor<64x64xi64, #blocked>
117|     %30 = arith.addi %26, %29 : tensor<64x64xi64, #blocked>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on integer multiplications, arith.extsi, broadcasted scalars or pointers, integer additions, lane/block index ranges. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 整数乘法、arith.extsi、广播后的标量或指针、整数加法、lane/block 索引范围。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 118-135
```mlir
118|     %31 = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked1}>>
119|     %32 = tt.expand_dims %31 {axis = 1 : i32} : tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked1}>> -> tensor<32x1xi32, #blocked1>
120|     %33 = tt.splat %3 : i64 -> tensor<32x1xi64, #blocked1>
121|     %34 = arith.extsi %32 : tensor<32x1xi32, #blocked1> to tensor<32x1xi64, #blocked1>
122|     %35 = arith.addi %33, %34 : tensor<32x1xi64, #blocked1>
123|     %36 = tt.splat %11 : i64 -> tensor<32x1xi64, #blocked1>
124|     %37 = arith.muli %35, %36 : tensor<32x1xi64, #blocked1>
125|     %38 = tt.splat %15 : i64 -> tensor<32x1xi64, #blocked1>
126|     %39 = arith.addi %37, %38 : tensor<32x1xi64, #blocked1>
127|     %40 = tt.broadcast %39 : tensor<32x1xi64, #blocked1> -> tensor<32x64xi64, #blocked1>
128|     %41 = tt.broadcast %28 : tensor<1x64xi64, #blocked1> -> tensor<32x64xi64, #blocked1>
129|     %42 = arith.addi %40, %41 : tensor<32x64xi64, #blocked1>
130|     %43 = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
131|     %44 = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
132|     %45 = tt.expand_dims %43 {axis = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked1}>> -> tensor<1x32xi32, #blocked1>
133|     %46 = tt.expand_dims %44 {axis = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x32xi32, #blocked>
134|     %47 = tt.splat %arg5 : i32 -> tensor<1x32xi32, #blocked1>
135|     %48 = tt.splat %arg5 : i32 -> tensor<1x32xi32, #blocked>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on broadcasted scalars or pointers, lane/block index ranges, shape expansion, integer additions, tensor broadcasting. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 广播后的标量或指针、lane/block 索引范围、形状扩展、整数加法、张量广播。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 136-153
```mlir
136|     %49 = arith.muli %45, %47 : tensor<1x32xi32, #blocked1>
137|     %50 = arith.muli %46, %48 : tensor<1x32xi32, #blocked>
138|     %51 = tt.broadcast %39 : tensor<32x1xi64, #blocked1> -> tensor<32x32xi64, #blocked1>
139|     %52 = arith.extsi %49 : tensor<1x32xi32, #blocked1> to tensor<1x32xi64, #blocked1>
140|     %53 = arith.extsi %50 : tensor<1x32xi32, #blocked> to tensor<1x32xi64, #blocked>
141|     %54 = tt.broadcast %52 : tensor<1x32xi64, #blocked1> -> tensor<32x32xi64, #blocked1>
142|     %55 = arith.addi %51, %54 : tensor<32x32xi64, #blocked1>
143|     %56 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<64x64x!tt.ptr<f32>, #blocked>
144|     %57 = tt.addptr %56, %30 : tensor<64x64x!tt.ptr<f32>, #blocked>, tensor<64x64xi64, #blocked>
145|     %58 = tt.splat %arg1 : !tt.ptr<f32> -> tensor<32x64x!tt.ptr<f32>, #blocked1>
146|     %59 = tt.addptr %58, %42 : tensor<32x64x!tt.ptr<f32>, #blocked1>, tensor<32x64xi64, #blocked1>
147|     %60 = tt.splat %arg2 : !tt.ptr<f32> -> tensor<32x32x!tt.ptr<f32>, #blocked1>
148|     %61 = tt.addptr %60, %55 : tensor<32x32x!tt.ptr<f32>, #blocked1>, tensor<32x32xi64, #blocked1>
149|     %62 = tt.load %57 : tensor<64x64x!tt.ptr<f32>, #blocked>
150|     %63 = scf.for %arg6 = %c0_i32 to %c64_i32 step %c32_i32 iter_args(%arg7 = %cst) -> (tensor<64x32xf32, #mma>)  : i32 {
151|       %70 = tt.load %59 : tensor<32x64x!tt.ptr<f32>, #blocked1>
152|       %71 = ttg.convert_layout %62 : tensor<64x64xf32, #blocked> -> tensor<64x64xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>>
153|       %72 = ttg.local_alloc %70 : (tensor<32x64xf32, #blocked1>) -> !ttg.memdesc<32x64xf32, #shared, #smem, mutable>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining broadcasted scalars or pointers, pointer arithmetic, integer multiplications, tensor broadcasting, arith.extsi.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 广播后的标量或指针、指针算术、整数乘法、张量广播、arith.extsi。

### Lines 154-171
```mlir
154|       %73 = ttg.memdesc_trans %72 {order=array<i32: 1,0>} : !ttg.memdesc<32x64xf32, #shared, #smem, mutable> -> !ttg.memdesc<64x32xf32, #shared1, #smem, mutable>
155|       %74 = ttg.local_load %73 : !ttg.memdesc<64x32xf32, #shared1, #smem, mutable> -> tensor<64x32xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>>
156|       %75 = tt.dot %71, %74, %cst, inputPrecision = tf32 : tensor<64x64xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>> * tensor<64x32xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>> -> tensor<64x32xf32, #mma>
157|       %76 = tt.load %61 : tensor<32x32x!tt.ptr<f32>, #blocked1>
158|       %77 = ttg.convert_layout %75 : tensor<64x32xf32, #mma> -> tensor<64x32xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>>
159|       %78 = ttg.convert_layout %76 : tensor<32x32xf32, #blocked1> -> tensor<32x32xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>>
160|       %79 = tt.dot %77, %78, %arg7, inputPrecision = tf32 : tensor<64x32xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>> * tensor<32x32xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>> -> tensor<64x32xf32, #mma>
161|       scf.yield %79 : tensor<64x32xf32, #mma>
162|     }
163|     %64 = tt.broadcast %17 : tensor<64x1xi64, #blocked> -> tensor<64x32xi64, #blocked>
164|     %65 = tt.broadcast %53 : tensor<1x32xi64, #blocked> -> tensor<64x32xi64, #blocked>
165|     %66 = arith.addi %64, %65 : tensor<64x32xi64, #blocked>
166|     %67 = tt.splat %arg4 : !tt.ptr<f32> -> tensor<64x32x!tt.ptr<f32>, #blocked>
167|     %68 = tt.addptr %67, %66 : tensor<64x32x!tt.ptr<f32>, #blocked>, tensor<64x32xi64, #blocked>
168|     %69 = ttg.convert_layout %63 : tensor<64x32xf32, #mma> -> tensor<64x32xf32, #blocked>
169|     tt.store %68, %69 : tensor<64x32x!tt.ptr<f32>, #blocked>
170|     tt.return
171|   }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining layout conversions, dot-product or MMA-style math, tensor broadcasting, ttg.memdesc_trans, local/shared memory loads.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 布局转换、点积或 MMA 风格计算、张量广播、ttg.memdesc_trans、本地/共享内存加载。

### Lines 172-173
```mlir
172| } // end module
173| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 174-174
```mlir
174| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 175-179
```mlir
175| 
176| // COMMON-NOT: #ttg.swizzled_shared<{{.*}} order = [2, 0, 1]
177| // COMMON: #ttg.swizzled_shared<{{.*}} order = [2, 1, 0]
178| // COMMON-NOT: #ttg.swizzled_shared<{{.*}} order = [2, 0, 1]
179| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 180-180
```mlir
180| // COMMON-LABEL: tt.func public @slowest_dim_is_batch
```
**EN:** This function-oriented block defines or enters `slowest_dim_is_batch`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `slowest_dim_is_batch` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 181-184
```mlir
181| #blocked = #ttg.blocked<{sizePerThread = [1, 1, 2], threadsPerWarp = [4, 1, 16], warpsPerCTA = [4, 1, 1], order = [2, 1, 0]}>
182| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1, 8], threadsPerWarp = [16, 1, 4], warpsPerCTA = [4, 1, 1], order = [2, 0, 1]}>
183| #blocked2 = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [1, 64], warpsPerCTA = [1, 4], order = [1, 0]}>
184| #blocked5 = #ttg.blocked<{sizePerThread = [1, 1, 2], threadsPerWarp = [16, 1, 4], warpsPerCTA = [4, 1, 1], order = [2, 0, 1]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 185-185
```mlir
185| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx90a", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx90a`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx90a`。

### Lines 186-203
```mlir
186|   tt.func public @slowest_dim_is_batch(%arg0: tensor<1x512x!tt.ptr<f32>, #blocked2>, %arg1: tensor<64x8x32x!tt.ptr<f32>, #blocked1>, %arg2: tensor<64x1x32x!tt.ptr<f32>, #blocked>) {
187|     %cst = arith.constant dense<0.000000e+00> : tensor<64x1x32xf32, #blocked>
188|     %cst_0 = arith.constant dense<512> : tensor<1x512xi32, #blocked2>
189|     %cst_1 = arith.constant dense<128> : tensor<64x8x32xi32, #blocked1>
190|     %c1_i32 = arith.constant 1 : i32
191|     %c5_i32 = arith.constant 2 : i32
192|     %c0_i32 = arith.constant 0 : i32
193|     %33:3 = scf.for %arg7 = %c0_i32 to %c5_i32 step %c1_i32 iter_args(%arg8 = %cst, %arg9 = %arg0, %arg10 = %arg1) -> (tensor<64x1x32xf32, #blocked>, tensor<1x512x!tt.ptr<f32>, #blocked2>, tensor<64x8x32x!tt.ptr<f32>, #blocked1>)  : i32 {
194|       %39 = tt.load %arg9 : tensor<1x512x!tt.ptr<f32>, #blocked2>
195|       %40 = tt.load %arg10 : tensor<64x8x32x!tt.ptr<f32>, #blocked1>
196|       %41 = tt.reshape %39 allow_reorder : tensor<1x512xf32, #blocked2> -> tensor<64x1x8xf32, #blocked5>
197|       %43 = ttg.convert_layout %41 : tensor<64x1x8xf32, #blocked5> -> tensor<64x1x8xf32, #ttg.dot_op<{opIdx = 0, parent = #blocked}>>
198|       %44 = ttg.convert_layout %40 : tensor<64x8x32xf32, #blocked1> -> tensor<64x8x32xf32, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>
199|       %45 = tt.dot %43, %44, %arg8, inputPrecision = tf32 : tensor<64x1x8xf32, #ttg.dot_op<{opIdx = 0, parent = #blocked}>> * tensor<64x8x32xf32, #ttg.dot_op<{opIdx = 1, parent = #blocked}>> -> tensor<64x1x32xf32, #blocked>
200|       %46 = tt.addptr %arg9, %cst_0 : tensor<1x512x!tt.ptr<f32>, #blocked2>, tensor<1x512xi32, #blocked2>
201|       %47 = tt.addptr %arg10, %cst_1 : tensor<64x8x32x!tt.ptr<f32>, #blocked1>, tensor<64x8x32xi32, #blocked1>
202|       scf.yield %45, %46, %47 : tensor<64x1x32xf32, #blocked>, tensor<1x512x!tt.ptr<f32>, #blocked2>, tensor<64x8x32x!tt.ptr<f32>, #blocked1>
203|     }
```
**EN:** This function-oriented block defines or enters `slowest_dim_is_batch`. Within it, the test exercises constants, tt.func, masked or vectorized loads, layout conversions, pointer arithmetic, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `slowest_dim_is_batch` 为核心。测试在其中演示 常量、tt.func、带掩码或向量化的加载、布局转换、指针算术，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 204-208
```mlir
204|     tt.store %arg2, %33#0 : tensor<64x1x32x!tt.ptr<f32>, #blocked>
205|     tt.return
206|   }
207| }
208| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized stores, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的存储、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 209-209
```mlir
209| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 210-215
```mlir
210| 
211| // Check that the stream pipeliner updates the resulting memory layout of transpose ops to mutable if immutable local buffers are replaced
212| // COMMON-LABEL: loop_with_dot_and_transpose
213| // COMMON: ttg.local_alloc {{.*}}, mutable>
214| // COMMON: ttg.memdesc_trans {{.*}}, mutable> -> {{.*}}, mutable>
215| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 216-220
```mlir
216| #blocked = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
217| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [16, 2], warpsPerCTA = [1, 4], order = [0, 1]}>
218| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0, 1]}>
219| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
220| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 221-221
```mlir
221| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1201", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1201`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1201`。

### Lines 222-239
```mlir
222|   tt.func public @loop_with_dot_and_transpose(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg1: i32, %arg4: tensor<32x32x!tt.ptr<f32>, #blocked1>, %arg5: tensor<32x32x!tt.ptr<f32>, #blocked>) {
223|     %c1_i32 = arith.constant 1 : i32
224|     %c0_i32 = arith.constant 0 : i32
225|     %cst = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #blocked>
226|     %0 = scf.for %arg2 = %c0_i32 to %arg1 step %c1_i32 iter_args(%arg3 = %cst) -> (tensor<32x32xf32, #blocked>)  : i32 {
227|       %2 = tt.load %arg4 : tensor<32x32x!tt.ptr<f32>, #blocked1>
228|       %3 = ttg.local_alloc %2 : (tensor<32x32xf32, #blocked1>) -> !ttg.memdesc<32x32xf32, #shared, #smem>
229|       %4 = ttg.memdesc_trans %3 {order = array<i32: 1, 0>} : !ttg.memdesc<32x32xf32, #shared, #smem> -> !ttg.memdesc<32x32xf32, #shared1, #smem>
230|       %5 = ttg.local_load %4 : !ttg.memdesc<32x32xf32, #shared1, #smem> -> tensor<32x32xf32, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>
231|       %6 = ttg.convert_layout %2 : tensor<32x32xf32, #blocked1> -> tensor<32x32xf32, #ttg.dot_op<{opIdx = 0, parent = #blocked}>>
232|       %7 = tt.dot %6, %5, %cst, inputPrecision = tf32 : tensor<32x32xf32, #ttg.dot_op<{opIdx = 0, parent = #blocked}>> * tensor<32x32xf32, #ttg.dot_op<{opIdx = 1, parent = #blocked}>> -> tensor<32x32xf32, #blocked>
233|       scf.yield %7 : tensor<32x32xf32, #blocked>
234|     }
235|     tt.store %arg5, %0 : tensor<32x32x!tt.ptr<f32>, #blocked>
236|     tt.return
237|   }
238| }
239| 
```
**EN:** This function-oriented block defines or enters `loop_with_dot_and_transpose`. Within it, the test exercises constants, tt.func, structured loops, masked or vectorized loads, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `loop_with_dot_and_transpose` 为核心。测试在其中演示 常量、tt.func、结构化循环、带掩码或向量化的加载、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 240-240
```mlir
240| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 241-248
```mlir
241| 
242| // Check that the stream pipeliner updates atomic op in the k-loop correctly
243| // COMMON-LABEL: _triton_gemm_kernel_atomic_rmw
244| // COMMON:  scf.for
245| // COMMON: tt.atomic_rmw fadd, acq_rel, gpu
246| // COMMON:  tt.dot
247| // COMMON: scf.yield
248| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 249-250
```mlir
249| #blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
250| #mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [4, 1], instrShape = [32, 32, 8], isTransposed = true}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 251-251
```mlir
251| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 252-269
```mlir
252|   tt.func public @_triton_gemm_kernel_atomic_rmw(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg2: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg3: i32 {tt.divisibility = 16 : i32}, %arg4: i32 {tt.divisibility = 16 : i32}) {
253|     %cst = arith.constant dense<32> : tensor<32x32xi32, #blocked>
254|     %c0_i32 = arith.constant 0 : i32
255|     %c1_i32 = arith.constant 1 : i32
256|     %c31_i32 = arith.constant 31 : i32
257|     %c32_i32 = arith.constant 32 : i32
258|     %cst_0 = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #mma>
259|     %0 = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
260|     %1 = tt.expand_dims %0 {axis = 1 : i32} : tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<32x1xi32, #blocked>
261|     %2 = tt.splat %arg4 : i32 -> tensor<32x1xi32, #blocked>
262|     %3 = arith.muli %1, %2 : tensor<32x1xi32, #blocked>
263|     %4 = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
264|     %5 = tt.expand_dims %4 {axis = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x32xi32, #blocked>
265|     %6 = tt.broadcast %3 : tensor<32x1xi32, #blocked> -> tensor<32x32xi32, #blocked>
266|     %7 = tt.broadcast %5 : tensor<1x32xi32, #blocked> -> tensor<32x32xi32, #blocked>
267|     %8 = arith.addi %6, %7 : tensor<32x32xi32, #blocked>
268|     %9 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<32x32x!tt.ptr<f16>, #blocked>
269|     %10 = tt.addptr %9, %8 : tensor<32x32x!tt.ptr<f16>, #blocked>, tensor<32x32xi32, #blocked>
```
**EN:** This function-oriented block defines or enters `_triton_gemm_kernel_atomic_rmw`. Within it, the test exercises constants, tt.func, lane/block index ranges, shape expansion, broadcasted scalars or pointers, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `_triton_gemm_kernel_atomic_rmw` 为核心。测试在其中演示 常量、tt.func、lane/block 索引范围、形状扩展、广播后的标量或指针，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 270-287
```mlir
270|     %11 = tt.splat %arg1 : !tt.ptr<f16> -> tensor<32x32x!tt.ptr<f16>, #blocked>
271|     %12 = tt.addptr %11, %8 : tensor<32x32x!tt.ptr<f16>, #blocked>, tensor<32x32xi32, #blocked>
272|     %13 = tt.splat %arg2 : !tt.ptr<f16> -> tensor<32x1x!tt.ptr<f16>, #blocked>
273|     %14 = tt.addptr %13, %3 : tensor<32x1x!tt.ptr<f16>, #blocked>, tensor<32x1xi32, #blocked>
274|     %15 = tt.broadcast %14 : tensor<32x1x!tt.ptr<f16>, #blocked> -> tensor<32x32x!tt.ptr<f16>, #blocked>
275|     %16 = tt.addptr %15, %7 : tensor<32x32x!tt.ptr<f16>, #blocked>, tensor<32x32xi32, #blocked>
276|     %17 = tt.splat %arg3 : i32 -> tensor<32x1xi32, #blocked>
277|     %18 = arith.cmpi slt, %1, %17 : tensor<32x1xi32, #blocked>
278|     %19 = tt.splat %arg3 : i32 -> tensor<1x32xi32, #blocked>
279|     %20 = arith.cmpi slt, %5, %19 : tensor<1x32xi32, #blocked>
280|     %21 = tt.broadcast %18 : tensor<32x1xi1, #blocked> -> tensor<32x32xi1, #blocked>
281|     %22 = tt.broadcast %20 : tensor<1x32xi1, #blocked> -> tensor<32x32xi1, #blocked>
282|     %23 = arith.andi %21, %22 : tensor<32x32xi1, #blocked>
283|     %24 = arith.addi %arg3, %c31_i32 : i32
284|     %25 = arith.divsi %24, %c32_i32 : i32
285|     %26 = arith.muli %arg4, %c32_i32 : i32
286|     %27 = tt.splat %26 : i32 -> tensor<32x32xi32, #blocked>
287|     %28:3 = scf.for %arg5 = %c0_i32 to %25 step %c1_i32 iter_args(%arg6 = %cst_0, %arg7 = %10, %arg8 = %12) -> (tensor<32x32xf32, #mma>, tensor<32x32x!tt.ptr<f16>, #blocked>, tensor<32x32x!tt.ptr<f16>, #blocked>)  : i32 {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining broadcasted scalars or pointers, pointer arithmetic, tensor broadcasting, integer comparisons, arith.andi.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 广播后的标量或指针、指针算术、张量广播、整数比较、arith.andi。

### Lines 288-305
```mlir
288|       %32 = tt.load %arg7 : tensor<32x32x!tt.ptr<f16>, #blocked>
289|       %33 = tt.load %arg8 : tensor<32x32x!tt.ptr<f16>, #blocked>
290|       %34 = ttg.convert_layout %32 : tensor<32x32xf16, #blocked> -> tensor<32x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>
291|       %35 = ttg.convert_layout %33 : tensor<32x32xf16, #blocked> -> tensor<32x32xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
292|       %36 = tt.dot %34, %35, %arg6 : tensor<32x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>> * tensor<32x32xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>> -> tensor<32x32xf32, #mma>
293|       %37 = tt.addptr %arg7, %cst : tensor<32x32x!tt.ptr<f16>, #blocked>, tensor<32x32xi32, #blocked>
294|       %38 = tt.addptr %arg8, %27 : tensor<32x32x!tt.ptr<f16>, #blocked>, tensor<32x32xi32, #blocked>
295|       %39 = arith.truncf %36 : tensor<32x32xf32, #mma> to tensor<32x32xf16, #mma>
296|       %40 = ttg.convert_layout %39 : tensor<32x32xf16, #mma> -> tensor<32x32xf16, #blocked>
297|       %41 = tt.atomic_rmw fadd, acq_rel, gpu, %16, %40, %23 : (tensor<32x32x!tt.ptr<f16>, #blocked>, tensor<32x32xf16, #blocked>, tensor<32x32xi1, #blocked>) -> tensor<32x32xf16, #blocked>
298|       scf.yield %36, %37, %38 : tensor<32x32xf32, #mma>, tensor<32x32x!tt.ptr<f16>, #blocked>, tensor<32x32x!tt.ptr<f16>, #blocked>
299|     }
300|     %29 = arith.truncf %28#0 : tensor<32x32xf32, #mma> to tensor<32x32xf16, #mma>
301|     %30 = ttg.convert_layout %16 : tensor<32x32x!tt.ptr<f16>, #blocked> -> tensor<32x32x!tt.ptr<f16>, #mma>
302|     %31 = ttg.convert_layout %23 : tensor<32x32xi1, #blocked> -> tensor<32x32xi1, #mma>
303|     tt.store %30, %29, %31 : tensor<32x32x!tt.ptr<f16>, #mma>
304|     tt.return
305|   }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining layout conversions, masked or vectorized loads, pointer arithmetic, arith.truncf, dot-product or MMA-style math.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 布局转换、带掩码或向量化的加载、指针算术、arith.truncf、点积或 MMA 风格计算。

### Lines 306-307
```mlir
306| }
307| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 308-308
```mlir
308| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 309-326
```mlir
309| 
310| // Check that we can pipeline scaled dot with linear layout
311| // COMMON-LABEL: mxfp8_mxfp4_matmul
312| 
313| // Prologue
314| // SYNC-3: ttg.local_alloc
315| // SYNC-3: tt.load
316| // SYNC-3: ttg.local_store
317| //
318| // ASYNC-3: ttg.async_copy_global_to_local
319| 
320| // Main loop
321| //         COMMON: scf.for
322| //          ASYNC: ttg.async_wait
323| // COMMON-COUNT-3:   ttg.local_load
324| //         COMMON:   tt.dot_scaled
325| //         COMMON:   scf.yield
326| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 327-334
```mlir
327| // Epilogue
328| //          ASYNC: ttg.async_wait
329| // COMMON-COUNT-3: ttg.local_load
330| //         COMMON: scf.if
331| //         COMMON:   tt.dot_scaled
332| // COMMON-COUNT-2:   scf.yield
333| // COMMON-COUNT-3: ttg.local_dealloc
334| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 335-341
```mlir
335| #blocked = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [4, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
336| #blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [64, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
337| #blocked2 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 64], warpsPerCTA = [1, 4], order = [1, 0]}>
338| #blocked3 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 64], warpsPerCTA = [4, 1], order = [1, 0]}>
339| #linear = #ttg.linear<{register = [[0, 2], [0, 4], [32, 0], [64, 0]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0], [0, 1]], warp = [[0, 0], [0, 0]], block = []}>
340| #linear1 = #ttg.linear<{register = [[0, 2], [0, 4], [128, 0]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0], [0, 1]], warp = [[32, 0], [64, 0]], block = []}>
341| #mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [1, 4], instrShape = [32, 32, 16], isTransposed = true}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 342-342
```mlir
342| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 343-360
```mlir
343|   tt.func public @mxfp8_mxfp4_matmul(
344|       %arg0: !tt.ptr<f8E5M2> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<i8> {tt.divisibility = 16 : i32}, %arg2: !tt.ptr<f32> {tt.divisibility = 16 : i32},
345|       %arg3: !tt.ptr<i8> {tt.divisibility = 16 : i32}, %arg4: i32 {tt.divisibility = 16 : i32}, %arg5: i32 {tt.divisibility = 16 : i32}, %arg6: i32 {tt.divisibility = 16 : i32},
346|       %arg7: i32 {tt.divisibility = 16 : i32}, %arg8: i32 {tt.divisibility = 16 : i32}, %arg9: i32 {tt.divisibility = 16 : i32}, %arg10: i32 {tt.divisibility = 16 : i32},
347|       %71: tensor<128x256x!tt.ptr<f32>, #blocked3>) {
348|     %cst = arith.constant dense<256> : tensor<128x256xi32, #blocked>
349|     %cst_0 = arith.constant dense<8> : tensor<256x8xi32, #blocked1>
350|     %c0_i32 = arith.constant 0 : i32
351|     %c1_i32 = arith.constant 1 : i32
352|     %cst_1 = arith.constant dense<127> : tensor<128x8xi8, #linear>
353|     %cst_2 = arith.constant dense<0.000000e+00> : tensor<128x256xf32, #blocked2>
354|     %cst_3 = arith.constant dense<0.000000e+00> : tensor<128x256xf32, #mma>
355|     %c127_i32 = arith.constant 127 : i32
356|     %c128_i32 = arith.constant 128 : i32
357|     %c256_i32 = arith.constant 256 : i32
358|     %c255_i32 = arith.constant 255 : i32
359|     %0 = tt.get_program_id x : i32
360|     %1 = arith.addi %arg4, %c127_i32 : i32
```
**EN:** This function-oriented block defines or enters `mxfp8_mxfp4_matmul`. Within it, the test exercises constants, tt.func, program IDs, integer additions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mxfp8_mxfp4_matmul` 为核心。测试在其中演示 常量、tt.func、程序 ID、整数加法，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 361-378
```mlir
361|     %2 = arith.divsi %1, %c128_i32 : i32
362|     %3 = arith.remsi %0, %2 : i32
363|     %4 = arith.divsi %0, %2 : i32
364|     %5 = arith.muli %3, %c128_i32 : i32
365|     %6 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
366|     %7 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked3}>>
367|     %8 = tt.splat %5 : i32 -> tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
368|     %9 = tt.splat %5 : i32 -> tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked3}>>
369|     %10 = arith.addi %8, %6 : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
370|     %11 = arith.addi %9, %7 : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked3}>>
371|     %12 = tt.splat %arg4 : i32 -> tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
372|     %13 = arith.remsi %10, %12 : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
373|     %14 = arith.muli %4, %c256_i32 : i32
374|     %15 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #ttg.slice<{dim = 1, parent = #blocked1}>>
375|     %16 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
376|     %17 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #ttg.slice<{dim = 0, parent = #blocked3}>>
377|     %18 = tt.splat %14 : i32 -> tensor<256xi32, #ttg.slice<{dim = 1, parent = #blocked1}>>
378|     %19 = tt.splat %14 : i32 -> tensor<256xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on lane/block index ranges, broadcasted scalars or pointers, arith.divsi, arith.remsi, integer multiplications. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 lane/block 索引范围、广播后的标量或指针、arith.divsi、arith.remsi、整数乘法。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 379-396
```mlir
379|     %20 = tt.splat %14 : i32 -> tensor<256xi32, #ttg.slice<{dim = 0, parent = #blocked3}>>
380|     %21 = arith.addi %18, %15 : tensor<256xi32, #ttg.slice<{dim = 1, parent = #blocked1}>>
381|     %22 = arith.addi %19, %16 : tensor<256xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
382|     %23 = arith.addi %20, %17 : tensor<256xi32, #ttg.slice<{dim = 0, parent = #blocked3}>>
383|     %24 = tt.splat %arg5 : i32 -> tensor<256xi32, #ttg.slice<{dim = 1, parent = #blocked1}>>
384|     %25 = tt.splat %arg5 : i32 -> tensor<256xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
385|     %26 = arith.remsi %21, %24 : tensor<256xi32, #ttg.slice<{dim = 1, parent = #blocked1}>>
386|     %27 = arith.remsi %22, %25 : tensor<256xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
387|     %28 = tt.expand_dims %26 {axis = 1 : i32} : tensor<256xi32, #ttg.slice<{dim = 1, parent = #blocked1}>> -> tensor<256x1xi32, #blocked1>
388|     %29 = tt.splat %arg7 : i32 -> tensor<256x1xi32, #blocked1>
389|     %30 = arith.muli %28, %29 : tensor<256x1xi32, #blocked1>
390|     %31 = tt.splat %arg3 : !tt.ptr<i8> -> tensor<256x1x!tt.ptr<i8>, #blocked1>
391|     %32 = tt.addptr %31, %30 : tensor<256x1x!tt.ptr<i8>, #blocked1>, tensor<256x1xi32, #blocked1>
392|     %33 = tt.make_range {end = 8 : i32, start = 0 : i32} : tensor<8xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
393|     %34 = tt.expand_dims %33 {axis = 0 : i32} : tensor<8xi32, #ttg.slice<{dim = 0, parent = #blocked1}>> -> tensor<1x8xi32, #blocked1>
394|     %35 = tt.broadcast %32 : tensor<256x1x!tt.ptr<i8>, #blocked1> -> tensor<256x8x!tt.ptr<i8>, #blocked1>
395|     %36 = tt.broadcast %34 : tensor<1x8xi32, #blocked1> -> tensor<256x8xi32, #blocked1>
396|     %37 = tt.addptr %35, %36 : tensor<256x8x!tt.ptr<i8>, #blocked1>, tensor<256x8xi32, #blocked1>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on broadcasted scalars or pointers, integer additions, arith.remsi, shape expansion, pointer arithmetic. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 广播后的标量或指针、整数加法、arith.remsi、形状扩展、指针算术。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 397-414
```mlir
397|     %38 = tt.expand_dims %13 {axis = 1 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<128x1xi32, #blocked>
398|     %39 = tt.splat %arg8 : i32 -> tensor<128x1xi32, #blocked>
399|     %40 = arith.muli %38, %39 : tensor<128x1xi32, #blocked>
400|     %41 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
401|     %42 = tt.expand_dims %41 {axis = 0 : i32} : tensor<256xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x256xi32, #blocked>
402|     %43 = tt.broadcast %40 : tensor<128x1xi32, #blocked> -> tensor<128x256xi32, #blocked>
403|     %44 = tt.broadcast %42 : tensor<1x256xi32, #blocked> -> tensor<128x256xi32, #blocked>
404|     %45 = arith.addi %43, %44 : tensor<128x256xi32, #blocked>
405|     %46 = tt.splat %arg0 : !tt.ptr<f8E5M2> -> tensor<128x256x!tt.ptr<f8E5M2>, #blocked>
406|     %47 = tt.addptr %46, %45 : tensor<128x256x!tt.ptr<f8E5M2>, #blocked>, tensor<128x256xi32, #blocked>
407|     %48 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
408|     %49 = tt.expand_dims %48 {axis = 1 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<128x1xi32, #blocked>
409|     %50 = tt.splat %arg9 : i32 -> tensor<128x1xi32, #blocked>
410|     %51 = arith.muli %49, %50 : tensor<128x1xi32, #blocked>
411|     %52 = tt.expand_dims %27 {axis = 0 : i32} : tensor<256xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x256xi32, #blocked>
412|     %53 = tt.broadcast %51 : tensor<128x1xi32, #blocked> -> tensor<128x256xi32, #blocked>
413|     %54 = tt.broadcast %52 : tensor<1x256xi32, #blocked> -> tensor<128x256xi32, #blocked>
414|     %55 = arith.addi %53, %54 : tensor<128x256xi32, #blocked>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shape expansion, tensor broadcasting, broadcasted scalars or pointers, integer multiplications, lane/block index ranges. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 形状扩展、张量广播、广播后的标量或指针、整数乘法、lane/block 索引范围。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 415-432
```mlir
415|     %56 = tt.splat %arg1 : !tt.ptr<i8> -> tensor<128x256x!tt.ptr<i8>, #blocked>
416|     %57 = tt.addptr %56, %55 : tensor<128x256x!tt.ptr<i8>, #blocked>, tensor<128x256xi32, #blocked>
417|     %58 = arith.addi %arg6, %c255_i32 : i32
418|     %59 = arith.divsi %58, %c256_i32 : i32
419|     %60 = arith.muli %arg9, %c128_i32 : i32
420|     %61 = tt.splat %60 : i32 -> tensor<128x256xi32, #blocked>
421|     %62:5 = scf.for %arg11 = %c0_i32 to %59 step %c1_i32 iter_args(%arg12 = %cst_2, %arg13 = %47, %arg14 = %57, %arg15 = %37, %arg16 = %cst_3)
422|       -> (tensor<128x256xf32, #blocked2>, tensor<128x256x!tt.ptr<f8E5M2>, #blocked>, tensor<128x256x!tt.ptr<i8>, #blocked>, tensor<256x8x!tt.ptr<i8>, #blocked1>, tensor<128x256xf32, #mma>)  : i32 {
423|       %80 = tt.load %arg13 : tensor<128x256x!tt.ptr<f8E5M2>, #blocked>
424|       %81 = tt.load %arg14 : tensor<128x256x!tt.ptr<i8>, #blocked>
425|       %82 = tt.load %arg15 : tensor<256x8x!tt.ptr<i8>, #blocked1>
426|       %83 = ttg.convert_layout %80 : tensor<128x256xf8E5M2, #blocked> -> tensor<128x256xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 16}>>
427|       %84 = ttg.convert_layout %81 : tensor<128x256xi8, #blocked> -> tensor<128x256xi8, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>
428|       %85 = ttg.convert_layout %82 : tensor<256x8xi8, #blocked1> -> tensor<256x8xi8, #linear1>
429|       %86 = tt.dot_scaled %83 scale %cst_1, %84 scale %85, %arg16 lhs = e5m2 rhs = e2m1 {fastMath = false} : tensor<128x256xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 16}>>, tensor<128x8xi8, #linear> * tensor<128x256xi8, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>, tensor<256x8xi8, #linear1> -> tensor<128x256xf32, #mma>
430|       %87 = ttg.convert_layout %86 : tensor<128x256xf32, #mma> -> tensor<128x256xf32, #blocked2>
431|       %88 = tt.addptr %arg13, %cst : tensor<128x256x!tt.ptr<f8E5M2>, #blocked>, tensor<128x256xi32, #blocked>
432|       %89 = tt.addptr %arg14, %61 : tensor<128x256x!tt.ptr<i8>, #blocked>, tensor<128x256xi32, #blocked>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining layout conversions, pointer arithmetic, masked or vectorized loads, broadcasted scalars or pointers, integer additions.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 布局转换、指针算术、带掩码或向量化的加载、广播后的标量或指针、整数加法。

### Lines 433-441
```mlir
433|       %90 = tt.addptr %arg15, %cst_0 : tensor<256x8x!tt.ptr<i8>, #blocked1>, tensor<256x8xi32, #blocked1>
434|       scf.yield %87, %88, %89, %90, %86 : tensor<128x256xf32, #blocked2>, tensor<128x256x!tt.ptr<f8E5M2>, #blocked>, tensor<128x256x!tt.ptr<i8>, #blocked>, tensor<256x8x!tt.ptr<i8>, #blocked1>, tensor<128x256xf32, #mma>
435|     } {tt.num_stages = 2 : i32}
436|     %79 = ttg.convert_layout %62#0 : tensor<128x256xf32, #blocked2> -> tensor<128x256xf32, #blocked3>
437|     tt.store %71, %79 : tensor<128x256x!tt.ptr<f32>, #blocked3>
438|     tt.return
439|   }
440| }
441| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining pointer arithmetic, loop/if yielded values, layout conversions, masked or vectorized stores, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 指针算术、循环/分支产出值、布局转换、带掩码或向量化的存储、tt.return。

### Lines 442-442
```mlir
442| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 443-460
```mlir
443| 
444| // Check that we can pipeline a simple matmul kernel
445| 
446| // COMMON-LABEL: simple_matmul_kernel
447| 
448| // Prologue
449| // COMMON-COUNT-2: ttg.local_alloc
450|   // SYNC-COUNT-2: tt.load
451|   // SYNC-COUNT-2: ttg.local_store
452|   //
453|   // ASYNC-COUNT-2: ttg.async_copy_global_to_local
454| 
455| // Main loop
456| //         COMMON:   scf.for
457| //
458|   // SYNC-COUNT-2:   ttg.local_load
459|   //         SYNC:   tt.dot
460|   //         SYNC:   scf.yield
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 461-476
```mlir
461|   //
462|   //         ASYNC:    ttg.async_wait
463|   //         ASYNC:    ttg.async_copy_global_to_local
464|   //         ASYNC:    ttg.local_load {{.*}} token
465|   //         ASYNC:    ttg.async_copy_global_to_local
466|   //         ASYNC:    ttg.local_load {{.*}} token
467|   //         ASYNC:    ttg.dot
468| 
469| // Epilogue
470| //          ASYNC: ttg.async_wait
471| // COMMON-COUNT-2: ttg.local_load
472| //         COMMON: scf.if
473| //         COMMON:   tt.dot
474| // COMMON-COUNT-2:   scf.yield
475| // COMMON-COUNT-2: ttg.local_dealloc
476| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 477-479
```mlir
477| #blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 8], warpsPerCTA = [8, 1], order = [1, 0]}>
478| #blocked1 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [4, 16], warpsPerCTA = [8, 1], order = [1, 0]}>
479| #mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [4, 2], instrShape = [32, 32, 8], isTransposed = true}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 480-480
```mlir
480| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 481-498
```mlir
481|   tt.func public @simple_matmul_kernel(%test: tensor<1x64xi32, #blocked1>, %arg0: tensor<64x64x!tt.ptr<f16>, #mma>, %arg1: i32, %arg2: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg3: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg4: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg5: i32 {tt.divisibility = 16 : i32}, %arg6: i32 {tt.divisibility = 16 : i32}, %arg7: i32 {tt.divisibility = 16 : i32}, %arg8: i32 {tt.divisibility = 16 : i32}, %arg9: i32 {tt.divisibility = 16 : i32}, %arg10: i32 {tt.divisibility = 16 : i32}) {
482|     %cst = arith.constant dense<32> : tensor<64x32xi32, #blocked>
483|     %cst_0 = arith.constant dense<32> : tensor<32x64xi32, #blocked1>
484|     %c64_i32 = arith.constant 64 : i32
485|     %c1_i32 = arith.constant 1 : i32
486|     %c0_i32 = arith.constant 0 : i32
487|     %cst_1 = arith.constant dense<0.000000e+00> : tensor<64x64xf32, #mma>
488|     %0 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
489|     %1 = arith.muli %arg1, %c64_i32 : i32
490|     %2 = tt.splat %1 : i32 -> tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
491|     %3 = arith.addi %2, %0 : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
492|     %4 = tt.splat %arg6 : i32 -> tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
493|     %5 = arith.remsi %3, %4 : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
494|     %6 = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
495|     %7 = tt.expand_dims %6 {axis = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x32xi32, #blocked>
496|     %8 = tt.broadcast %7 : tensor<1x32xi32, #blocked> -> tensor<64x32xi32, #blocked>
497|     %9 = tt.splat %arg2 : !tt.ptr<f16> -> tensor<64x32x!tt.ptr<f16>, #blocked>
498|     %10 = tt.addptr %9, %8 : tensor<64x32x!tt.ptr<f16>, #blocked>, tensor<64x32xi32, #blocked>
```
**EN:** This function-oriented block defines or enters `simple_matmul_kernel`. Within it, the test exercises constants, broadcasted scalars or pointers, tt.func, lane/block index ranges, integer multiplications, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `simple_matmul_kernel` 为核心。测试在其中演示 常量、广播后的标量或指针、tt.func、lane/block 索引范围、整数乘法，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 499-516
```mlir
499|     %11 = tt.expand_dims %5 {axis = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked1}>> -> tensor<1x64xi32, #blocked1>
500|     %12 = tt.broadcast %11 : tensor<1x64xi32, #blocked1> -> tensor<32x64xi32, #blocked1>
501|     %13 = tt.splat %arg3 : !tt.ptr<f16> -> tensor<32x64x!tt.ptr<f16>, #blocked1>
502|     %14 = tt.addptr %13, %12 : tensor<32x64x!tt.ptr<f16>, #blocked1>, tensor<32x64xi32, #blocked1>
503|     %15:3 = scf.for %arg11 = %c0_i32 to %arg1 step %c1_i32 iter_args(%arg12 = %cst_1, %arg13 = %10, %arg14 = %14) -> (tensor<64x64xf32, #mma>, tensor<64x32x!tt.ptr<f16>, #blocked>, tensor<32x64x!tt.ptr<f16>, #blocked1>)  : i32 {
504|       %17 = tt.load %arg13 : tensor<64x32x!tt.ptr<f16>, #blocked>
505|       %18 = tt.load %arg14 : tensor<32x64x!tt.ptr<f16>, #blocked1>
506|       %19 = ttg.convert_layout %17 : tensor<64x32xf16, #blocked> -> tensor<64x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>
507|       %20 = ttg.convert_layout %18 : tensor<32x64xf16, #blocked1> -> tensor<32x64xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
508|       %21 = tt.dot %19, %20, %arg12, inputPrecision = tf32 : tensor<64x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>> * tensor<32x64xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>> -> tensor<64x64xf32, #mma>
509|       %22 = tt.addptr %arg13, %cst : tensor<64x32x!tt.ptr<f16>, #blocked>, tensor<64x32xi32, #blocked>
510|       %23 = tt.addptr %arg14, %cst_0 : tensor<32x64x!tt.ptr<f16>, #blocked1>, tensor<32x64xi32, #blocked1>
511|       scf.yield %21, %22, %23 : tensor<64x64xf32, #mma>, tensor<64x32x!tt.ptr<f16>, #blocked>, tensor<32x64x!tt.ptr<f16>, #blocked1>
512|     }
513|     %16 = arith.truncf %15#0 : tensor<64x64xf32, #mma> to tensor<64x64xf16, #mma>
514|     tt.store %arg0, %16 : tensor<64x64x!tt.ptr<f16>, #mma>
515|     tt.return
516|   }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining pointer arithmetic, masked or vectorized loads, layout conversions, shape expansion, tensor broadcasting.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 指针算术、带掩码或向量化的加载、布局转换、形状扩展、张量广播。

### Lines 517-518
```mlir
517| }
518| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 519-519
```mlir
519| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 520-537
```mlir
520| 
521| // Check that we can pipeline small width vectors (like scale factor)
522| // COMMON-LABEL: pipeline_small_vector
523| 
524| // Prologue
525| // COMMON-COUNT-4: tt.load
526| 
527| // Main loop
528| //         COMMON: scf.for
529| // COMMON-COUNT-4:   tt.load
530| //         COMMON:   tt.dot_scaled
531| //         COMMON:   scf.yield
532| 
533| // Epilogue
534| //         COMMON: scf.if
535| //         COMMON:   tt.dot_scaled
536| //         COMMON:   scf.yield
537| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 538-544
```mlir
538| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [16, 4], warpsPerCTA = [8, 1], order = [1, 0]}>
539| #blocked1 = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [4, 16], warpsPerCTA = [8, 1], order = [1, 0]}>
540| #blocked2 = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [8, 8], warpsPerCTA = [8, 1], order = [1, 0]}>
541| #blocked3 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 64], warpsPerCTA = [2, 4], order = [1, 0]}>
542| #blocked4 = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [32, 2], warpsPerCTA = [8, 1], order = [1, 0]}>
543| #blocked5 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 64], warpsPerCTA = [8, 1], order = [1, 0]}>
544| #blocked6 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 64], warpsPerCTA = [4, 2], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 545-545
```mlir
545| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 546-563
```mlir
546|   tt.func public @pipeline_small_vector(%arg0: !tt.ptr<f8E5M2>, %arg1: !tt.ptr<f8E5M2>, %arg2: !tt.ptr<f32>, %arg3: !tt.ptr<i8>, %arg4: !tt.ptr<i8>, %arg5: i32, %arg6: i32, %arg7: i32, %arg8: i32, %arg9: i32, %arg10: i32) -> tensor<128x256xf32, #blocked3> {
547|     %c128_i32 = arith.constant 128 : i32
548|     %c256_i32 = arith.constant 256 : i32
549|     %cst = arith.constant dense<4> : tensor<128x4xi32, #blocked>
550|     %cst_0 = arith.constant dense<0.000000e+00> : tensor<128x256xf8E5M2, #blocked1>
551|     %cst_1 = arith.constant dense<0.000000e+00> : tensor<128x128xf8E5M2, #blocked2>
552|     %cst_2 = arith.constant dense<0.000000e+00> : tensor<128x256xf32, #blocked3>
553|     %c127_i32 = arith.constant 127 : i32
554|     %c1_i32 = arith.constant 1 : i32
555|     %c0_i32 = arith.constant 0 : i32
556|     %cst_3 = arith.constant dense<4> : tensor<256x4xi32, #blocked4>
557|     %cst_4 = arith.constant dense<128> : tensor<128x128xi32, #blocked2>
558|     %cst_5 = arith.constant dense<8> : tensor<256x1xi32, #blocked4>
559|     %cst_6 = arith.constant dense<8> : tensor<128x1xi32, #blocked>
560|     %0 = tt.get_program_id x : i32
561|     %1 = arith.addi %arg5, %c127_i32 : i32
562|     %2 = arith.divsi %1, %c128_i32 : i32
563|     %3 = arith.remsi %0, %2 : i32
```
**EN:** This function-oriented block defines or enters `pipeline_small_vector`. Within it, the test exercises constants, tt.func, program IDs, integer additions, arith.divsi, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `pipeline_small_vector` 为核心。测试在其中演示 常量、tt.func、程序 ID、整数加法、arith.divsi，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 564-581
```mlir
564|     %4 = arith.divsi %0, %2 : i32
565|     %5 = arith.muli %3, %c128_i32 : i32
566|     %6 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
567|     %7 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked2}>>
568|     %8 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked1}>>
569|     %9 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked5}>>
570|     %10 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked2}>>
571|     %11 = tt.splat %5 : i32 -> tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
572|     %12 = tt.splat %5 : i32 -> tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked2}>>
573|     %13 = tt.splat %5 : i32 -> tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked5}>>
574|     %14 = arith.addi %11, %6 : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
575|     %15 = arith.addi %12, %7 : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked2}>>
576|     %16 = arith.addi %13, %9 : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked5}>>
577|     %17 = tt.splat %arg5 : i32 -> tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
578|     %18 = tt.splat %arg5 : i32 -> tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked2}>>
579|     %19 = arith.remsi %14, %17 : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
580|     %20 = arith.remsi %15, %18 : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked2}>>
581|     %21 = arith.muli %4, %c256_i32 : i32
```
**EN:** This operational block carries the concrete IR steps for the test, centered on lane/block index ranges, broadcasted scalars or pointers, integer additions, integer multiplications, arith.remsi. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 lane/block 索引范围、广播后的标量或指针、整数加法、整数乘法、arith.remsi。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 582-599
```mlir
582|     %22 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #ttg.slice<{dim = 1, parent = #blocked4}>>
583|     %23 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
584|     %24 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #ttg.slice<{dim = 0, parent = #blocked5}>>
585|     %25 = tt.splat %21 : i32 -> tensor<256xi32, #ttg.slice<{dim = 1, parent = #blocked4}>>
586|     %26 = tt.splat %21 : i32 -> tensor<256xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
587|     %27 = tt.splat %21 : i32 -> tensor<256xi32, #ttg.slice<{dim = 0, parent = #blocked5}>>
588|     %28 = arith.addi %25, %22 : tensor<256xi32, #ttg.slice<{dim = 1, parent = #blocked4}>>
589|     %29 = arith.addi %26, %23 : tensor<256xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
590|     %30 = arith.addi %27, %24 : tensor<256xi32, #ttg.slice<{dim = 0, parent = #blocked5}>>
591|     %31 = tt.splat %arg6 : i32 -> tensor<256xi32, #ttg.slice<{dim = 1, parent = #blocked4}>>
592|     %32 = tt.splat %arg6 : i32 -> tensor<256xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
593|     %33 = arith.remsi %28, %31 : tensor<256xi32, #ttg.slice<{dim = 1, parent = #blocked4}>>
594|     %34 = arith.remsi %29, %32 : tensor<256xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
595|     %35 = tt.expand_dims %19 {axis = 1 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<128x1xi32, #blocked>
596|     %36 = tt.expand_dims %20 {axis = 1 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked2}>> -> tensor<128x1xi32, #blocked2>
597|     %37 = arith.muli %35, %cst_6 : tensor<128x1xi32, #blocked>
598|     %38 = tt.splat %arg3 : !tt.ptr<i8> -> tensor<128x1x!tt.ptr<i8>, #blocked>
599|     %39 = tt.addptr %38, %37 : tensor<128x1x!tt.ptr<i8>, #blocked>, tensor<128x1xi32, #blocked>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on broadcasted scalars or pointers, lane/block index ranges, integer additions, arith.remsi, shape expansion. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 广播后的标量或指针、lane/block 索引范围、整数加法、arith.remsi、形状扩展。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 600-617
```mlir
600|     %40 = tt.make_range {end = 4 : i32, start = 0 : i32} : tensor<4xi32, #ttg.slice<{dim = 0, parent = #blocked4}>>
601|     %41 = tt.make_range {end = 4 : i32, start = 0 : i32} : tensor<4xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
602|     %42 = tt.expand_dims %40 {axis = 0 : i32} : tensor<4xi32, #ttg.slice<{dim = 0, parent = #blocked4}>> -> tensor<1x4xi32, #blocked4>
603|     %43 = tt.expand_dims %41 {axis = 0 : i32} : tensor<4xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x4xi32, #blocked>
604|     %44 = tt.broadcast %39 : tensor<128x1x!tt.ptr<i8>, #blocked> -> tensor<128x4x!tt.ptr<i8>, #blocked>
605|     %45 = tt.broadcast %43 : tensor<1x4xi32, #blocked> -> tensor<128x4xi32, #blocked>
606|     %46 = tt.addptr %44, %45 : tensor<128x4x!tt.ptr<i8>, #blocked>, tensor<128x4xi32, #blocked>
607|     %47 = tt.expand_dims %33 {axis = 1 : i32} : tensor<256xi32, #ttg.slice<{dim = 1, parent = #blocked4}>> -> tensor<256x1xi32, #blocked4>
608|     %48 = arith.muli %47, %cst_5 : tensor<256x1xi32, #blocked4>
609|     %49 = tt.splat %arg4 : !tt.ptr<i8> -> tensor<256x1x!tt.ptr<i8>, #blocked4>
610|     %50 = tt.addptr %49, %48 : tensor<256x1x!tt.ptr<i8>, #blocked4>, tensor<256x1xi32, #blocked4>
611|     %51 = tt.broadcast %50 : tensor<256x1x!tt.ptr<i8>, #blocked4> -> tensor<256x4x!tt.ptr<i8>, #blocked4>
612|     %52 = tt.broadcast %42 : tensor<1x4xi32, #blocked4> -> tensor<256x4xi32, #blocked4>
613|     %53 = tt.addptr %51, %52 : tensor<256x4x!tt.ptr<i8>, #blocked4>, tensor<256x4xi32, #blocked4>
614|     %54 = tt.splat %arg8 : i32 -> tensor<128x1xi32, #blocked2>
615|     %55 = arith.muli %36, %54 : tensor<128x1xi32, #blocked2>
616|     %56 = tt.expand_dims %10 {axis = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked2}>> -> tensor<1x128xi32, #blocked2>
617|     %57 = tt.broadcast %55 : tensor<128x1xi32, #blocked2> -> tensor<128x128xi32, #blocked2>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor broadcasting, shape expansion, pointer arithmetic, lane/block index ranges, integer multiplications. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量广播、形状扩展、指针算术、lane/block 索引范围、整数乘法。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 618-635
```mlir
618|     %58 = tt.broadcast %56 : tensor<1x128xi32, #blocked2> -> tensor<128x128xi32, #blocked2>
619|     %59 = arith.addi %57, %58 : tensor<128x128xi32, #blocked2>
620|     %60 = tt.splat %arg0 : !tt.ptr<f8E5M2> -> tensor<128x128x!tt.ptr<f8E5M2>, #blocked2>
621|     %61 = tt.addptr %60, %59 : tensor<128x128x!tt.ptr<f8E5M2>, #blocked2>, tensor<128x128xi32, #blocked2>
622|     %62 = tt.expand_dims %8 {axis = 1 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked1}>> -> tensor<128x1xi32, #blocked1>
623|     %63 = tt.splat %arg9 : i32 -> tensor<128x1xi32, #blocked1>
624|     %64 = arith.muli %62, %63 : tensor<128x1xi32, #blocked1>
625|     %65 = tt.expand_dims %34 {axis = 0 : i32} : tensor<256xi32, #ttg.slice<{dim = 0, parent = #blocked1}>> -> tensor<1x256xi32, #blocked1>
626|     %66 = tt.broadcast %64 : tensor<128x1xi32, #blocked1> -> tensor<128x256xi32, #blocked1>
627|     %67 = tt.broadcast %65 : tensor<1x256xi32, #blocked1> -> tensor<128x256xi32, #blocked1>
628|     %68 = arith.addi %66, %67 : tensor<128x256xi32, #blocked1>
629|     %69 = tt.splat %arg1 : !tt.ptr<f8E5M2> -> tensor<128x256x!tt.ptr<f8E5M2>, #blocked1>
630|     %70 = tt.addptr %69, %68 : tensor<128x256x!tt.ptr<f8E5M2>, #blocked1>, tensor<128x256xi32, #blocked1>
631|     %71 = arith.addi %arg7, %c127_i32 : i32
632|     %72 = arith.divsi %71, %c128_i32 : i32
633|     %73 = arith.muli %arg9, %c128_i32 : i32
634|     %74 = tt.splat %73 : i32 -> tensor<128x256xi32, #blocked1>
635|     %75:5 = scf.for %arg11 = %c0_i32 to %72 step %c1_i32 iter_args(%arg12 = %cst_2, %arg13 = %46, %arg14 = %61, %arg15 = %70, %arg16 = %53) -> (tensor<128x256xf32, #blocked3>, tensor<128x4x!tt.ptr<i8>, #blocked>, tensor<128x128x!tt.ptr<f8E5M2>, #blocked2>, tensor<128x256x!tt.ptr<f8E5M2>, #blocked1>, tensor<256x4x!tt.ptr<i8>, #blocked4>)  : i32 {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining broadcasted scalars or pointers, tensor broadcasting, integer additions, pointer arithmetic, shape expansion.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 广播后的标量或指针、张量广播、整数加法、指针算术、形状扩展。

### Lines 636-653
```mlir
636|       %93 = arith.muli %arg11, %c128_i32 : i32
637|       %94 = arith.subi %arg7, %93 : i32
638|       %95 = tt.splat %94 : i32 -> tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked2}>>
639|       %96 = tt.splat %94 : i32 -> tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked1}>>
640|       %97 = arith.cmpi slt, %10, %95 : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked2}>>
641|       %98 = arith.cmpi slt, %8, %96 : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked1}>>
642|       %99 = tt.expand_dims %97 {axis = 0 : i32} : tensor<128xi1, #ttg.slice<{dim = 0, parent = #blocked2}>> -> tensor<1x128xi1, #blocked2>
643|       %100 = tt.broadcast %99 : tensor<1x128xi1, #blocked2> -> tensor<128x128xi1, #blocked2>
644|       %101 = tt.load %arg14, %100, %cst_1 : tensor<128x128x!tt.ptr<f8E5M2>, #blocked2>
645|       %102 = ttg.convert_layout %101 : tensor<128x128xf8E5M2, #blocked2> -> tensor<128x128xf8E5M2, #blocked6>
646|       %103 = tt.expand_dims %98 {axis = 1 : i32} : tensor<128xi1, #ttg.slice<{dim = 1, parent = #blocked1}>> -> tensor<128x1xi1, #blocked1>
647|       %104 = tt.broadcast %103 : tensor<128x1xi1, #blocked1> -> tensor<128x256xi1, #blocked1>
648|       %105 = tt.load %arg15, %104, %cst_0 : tensor<128x256x!tt.ptr<f8E5M2>, #blocked1>
649|       %106 = ttg.convert_layout %105 : tensor<128x256xf8E5M2, #blocked1> -> tensor<128x256xf8E5M2, #blocked3>
650|       %107 = tt.load %arg13 : tensor<128x4x!tt.ptr<i8>, #blocked>
651|       %108 = tt.load %arg16 : tensor<256x4x!tt.ptr<i8>, #blocked4>
652|       %109 = ttg.convert_layout %108 : tensor<256x4xi8, #blocked4> -> tensor<256x4xi8, #blocked>
653|       %110 = tt.dot_scaled %102 scale %107, %106 scale %109, %arg12 lhs = e5m2 rhs = e5m2 {fastMath = false} : tensor<128x128xf8E5M2, #blocked6>, tensor<128x4xi8, #blocked> * tensor<128x256xf8E5M2, #blocked3>, tensor<256x4xi8, #blocked> -> tensor<128x256xf32, #blocked3>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized loads, layout conversions, broadcasted scalars or pointers, integer comparisons, shape expansion. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的加载、布局转换、广播后的标量或指针、整数比较、形状扩展。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 654-663
```mlir
654|       %111 = tt.addptr %arg14, %cst_4 : tensor<128x128x!tt.ptr<f8E5M2>, #blocked2>, tensor<128x128xi32, #blocked2>
655|       %112 = tt.addptr %arg15, %74 : tensor<128x256x!tt.ptr<f8E5M2>, #blocked1>, tensor<128x256xi32, #blocked1>
656|       %113 = tt.addptr %arg13, %cst : tensor<128x4x!tt.ptr<i8>, #blocked>, tensor<128x4xi32, #blocked>
657|       %114 = tt.addptr %arg16, %cst_3 : tensor<256x4x!tt.ptr<i8>, #blocked4>, tensor<256x4xi32, #blocked4>
658|       scf.yield %110, %113, %111, %112, %114 : tensor<128x256xf32, #blocked3>, tensor<128x4x!tt.ptr<i8>, #blocked>, tensor<128x128x!tt.ptr<f8E5M2>, #blocked2>, tensor<128x256x!tt.ptr<f8E5M2>, #blocked1>, tensor<256x4x!tt.ptr<i8>, #blocked4>
659|     } {tt.num_stages = 2 : i32}
660|     tt.return %75#0 : tensor<128x256xf32, #blocked3>
661|   }
662| }
663| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining pointer arithmetic, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 指针算术、循环/分支产出值、tt.return。

### Lines 664-664
```mlir
664| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 665-668
```mlir
665| 
666| // COMMON-LABEL: pipeline_scale_memory_order
667| // ASYNC-2: ttg.async_copy_global_to_local
668| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 669-672
```mlir
669| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [64, 1], warpsPerCTA = [8, 1], order = [1, 0]}>
670| #linear = #ttg.linear<{register = [[0, 4], [16, 0], [32, 0], [64, 0]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 1], [0, 2]], warp = [[0, 0], [0, 0], [0, 0]], block = []}>
671| #linear1 = #ttg.linear<{register = [[0, 4], [128, 0], [256, 0]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 1], [0, 2]], warp = [[16, 0], [32, 0], [64, 0]], block = []}>
672| #mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [1, 8], instrShape = [16, 16, 32], isTransposed = true}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 673-673
```mlir
673| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 674-691
```mlir
674|   tt.func public @pipeline_scale_memory_order(%arg0: !tt.ptr<i8> {tt.divisibility = 16 : i32}, %arg1: i64 {tt.divisibility = 16 : i32}, %arg2: tensor<128x256xf8E4M3FN, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 16}>>, %arg3: tensor<128x512xi8, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>, %arg4: tensor<128x512x!tt.ptr<f32>, #mma>, %arg5: tensor<512x8x!tt.ptr<i8>, #blocked>) {
675|     %cst = arith.constant dense<127> : tensor<128x8xi8, #linear>
676|     %cst_0 = arith.constant dense<8> : tensor<512x8xi32, #blocked>
677|     %c256_i64 = arith.constant 256 : i64
678|     %c0_i64 = arith.constant 0 : i64
679|     %cst_1 = arith.constant dense<0.000000e+00> : tensor<128x512xf32, #mma>
680|     %0 = tt.make_range {end = 8 : i32, start = 0 : i32} : tensor<8xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
681|     %1 = arith.extsi %0 : tensor<8xi32, #ttg.slice<{dim = 0, parent = #blocked}>> to tensor<8xi64, #ttg.slice<{dim = 0, parent = #blocked}>>
682|     %2 = tt.expand_dims %1 {axis = 0 : i32} : tensor<8xi64, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x8xi64, #blocked>
683|     %3 = tt.splat %arg0 : !tt.ptr<i8> -> tensor<1x8x!tt.ptr<i8>, #blocked>
684|     %4 = tt.addptr %3, %2 : tensor<1x8x!tt.ptr<i8>, #blocked>, tensor<1x8xi64, #blocked>
685|     %5 = tt.broadcast %4 : tensor<1x8x!tt.ptr<i8>, #blocked> -> tensor<512x8x!tt.ptr<i8>, #blocked>
686|     %6:2 = scf.for %arg6 = %c0_i64 to %arg1 step %c256_i64 iter_args(%arg7 = %cst_1, %arg8 = %5) -> (tensor<128x512xf32, #mma>, tensor<512x8x!tt.ptr<i8>, #blocked>)  : i64 {
687|       %7 = tt.load %arg8 : tensor<512x8x!tt.ptr<i8>, #blocked>
688|       %8 = ttg.convert_layout %7 : tensor<512x8xi8, #blocked> -> tensor<512x8xi8, #linear1>
689|       %9 = tt.dot_scaled %arg2 scale %cst, %arg3 scale %8, %arg7 lhs = e4m3 rhs = e2m1 {fastMath = true} : tensor<128x256xf8E4M3FN, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 16}>>, tensor<128x8xi8, #linear> * tensor<128x512xi8, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>, tensor<512x8xi8, #linear1> -> tensor<128x512xf32, #mma>
690|       %10 = tt.addptr %arg8, %cst_0 : tensor<512x8x!tt.ptr<i8>, #blocked>, tensor<512x8xi32, #blocked>
691|       scf.yield %9, %10 : tensor<128x512xf32, #mma>, tensor<512x8x!tt.ptr<i8>, #blocked>
```
**EN:** This function-oriented block defines or enters `pipeline_scale_memory_order`. Within it, the test exercises constants, tt.func, pointer arithmetic, lane/block index ranges, arith.extsi, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `pipeline_scale_memory_order` 为核心。测试在其中演示 常量、tt.func、指针算术、lane/block 索引范围、arith.extsi，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 692-697
```mlir
692|     }
693|     tt.store %arg4, %6#0 : tensor<128x512x!tt.ptr<f32>, #mma>
694|     tt.return
695|   }
696| }
697| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized stores, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的存储、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 698-698
```mlir
698| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 699-705
```mlir
699| 
700| #AL = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [8, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
701| #C = #ttg.amd_mfma<{version = 4, warpsPerCTA = [4, 1], instrShape = [16, 16, 32], isTransposed = true}>
702| #A = #ttg.dot_op<{opIdx = 0, parent = #C, kWidth=2}>
703| #B = #ttg.dot_op<{opIdx = 1, parent = #C, kWidth=2}>
704| #smem = #ttg.shared_memory
705| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 706-709
```mlir
706| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
707| // Verify that we do not get AsyncCopies because we cannot lower it on gfx942 since we only have 32bit wide loads to lds
708| // COMMON-LABEL: @reject_fp64_pipelining_with_async_copy_gfx942
709| // ASYNC-NOT: ttg.async_copy_global_to_local
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 710-723
```mlir
710| tt.func @reject_fp64_pipelining_with_async_copy_gfx942(
711|                   %a_ptr : tensor<128x32x!tt.ptr<f64>, #AL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
712|                   %B : tensor<32x128xf64, #B>, %lb: i32, %ub: i32, %step: i32) -> tensor<128x128xf64, #C> {
713|   %c_init = arith.constant dense<0.00e+00> : tensor<128x128xf64, #C>
714|   %loop = scf.for %iv = %lb to %ub step %step iter_args(%prev_c = %c_init) -> (tensor<128x128xf64, #C>) : i32 {
715|     %a_ = tt.load %a_ptr : tensor<128x32x!tt.ptr<f64>, #AL>
716|     %a = ttg.convert_layout %a_ : tensor<128x32xf64, #AL> -> tensor<128x32xf64, #A>
717|     %c = tt.dot %a, %B, %prev_c : tensor<128x32xf64, #A> * tensor<32x128xf64, #B> -> tensor<128x128xf64, #C>
718|     scf.yield %c : tensor<128x128xf64, #C>
719|   }
720|   tt.return %loop: tensor<128x128xf64, #C>
721| }
722| }
723| 
```
**EN:** This function-oriented block defines or enters `reject_fp64_pipelining_with_async_copy_gfx942`. Within it, the test exercises tt.func, constants, structured loops, masked or vectorized loads, layout conversions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `reject_fp64_pipelining_with_async_copy_gfx942` 为核心。测试在其中演示 tt.func、常量、结构化循环、带掩码或向量化的加载、布局转换，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 724-724
```mlir
724| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 725-732
```mlir
725| 
726| #AL = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [8, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
727| #BL = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
728| #C = #ttg.amd_mfma<{version = 4, warpsPerCTA = [4, 1], instrShape = [16, 16, 32], isTransposed = true}>
729| #A = #ttg.dot_op<{opIdx = 0, parent = #C, kWidth=2}>
730| #B = #ttg.dot_op<{opIdx = 1, parent = #C, kWidth=2}>
731| #smem = #ttg.shared_memory
732| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 733-739
```mlir
733| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
734| // On GFX950 we can use AsyncCopy if sizePerThread >= 2 and it's contiguous because we can load 2 fp64 with one direct to lds instruction
735| // COMMON-LABEL: @pipeline_fp64_with_async_copy_gfx950
736| // ASYNC: ttg.async_copy_global_to_local
737| // ASYNC: tt.load
738| // ASYNC: ttg.async_copy_global_to_local
739| // ASYNC: tt.load
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 740-756
```mlir
740| tt.func @pipeline_fp64_with_async_copy_gfx950(
741|                   %a_ptr : tensor<128x32x!tt.ptr<f64>, #AL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
742|                   %b_ptr : tensor<32x128x!tt.ptr<f64>, #BL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 2]> : tensor<2xi32>},
743|                   %lb: i32, %ub: i32, %step: i32) -> tensor<128x128xf64, #C> {
744|   %c_init = arith.constant dense<0.00e+00> : tensor<128x128xf64, #C>
745|   %loop = scf.for %iv = %lb to %ub step %step iter_args(%prev_c = %c_init) -> (tensor<128x128xf64, #C>) : i32 {
746|     %a_ = tt.load %a_ptr : tensor<128x32x!tt.ptr<f64>, #AL>
747|     %a = ttg.convert_layout %a_ : tensor<128x32xf64, #AL> -> tensor<128x32xf64, #A>
748|     %b_ = tt.load %b_ptr : tensor<32x128x!tt.ptr<f64>, #BL>
749|     %b = ttg.convert_layout %b_ : tensor<32x128xf64, #BL> -> tensor<32x128xf64, #B>
750|     %c = tt.dot %a, %b, %prev_c : tensor<128x32xf64, #A> * tensor<32x128xf64, #B> -> tensor<128x128xf64, #C>
751|     scf.yield %c : tensor<128x128xf64, #C>
752|   }
753|   tt.return %loop: tensor<128x128xf64, #C>
754| }
755| }
756| 
```
**EN:** This function-oriented block defines or enters `pipeline_fp64_with_async_copy_gfx950`. Within it, the test exercises tt.func, masked or vectorized loads, layout conversions, constants, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `pipeline_fp64_with_async_copy_gfx950` 为核心。测试在其中演示 tt.func、带掩码或向量化的加载、布局转换、常量、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 757-757
```mlir
757| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 758-775
```mlir
758| 
759| // COMMON-LABEL: pipelining_local_load_packed_transposed
760| 
761| // Prologue
762| // COMMON: ttg.local_alloc
763| // COMMON: ttg.local_alloc
764| // ASYNC: ttg.async_copy_global_to_local
765| // SYNC: tt.load
766| // COMMON: tt.load
767| // SYNC: ttg.local_store
768| // COMMON: ttg.local_store
769| 
770| // Main loop
771| //         COMMON: scf.for
772| //         COMMON:   ttg.local_load
773| //         COMMON:   amdg.local_load_packed_tranposed
774| //         COMMON:   tt.dot_scaled
775| //         COMMON:   scf.yield
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 776-784
```mlir
776| 
777| // Epilogue
778| //         COMMON:   ttg.local_load
779| //         COMMON: amdg.local_load_packed_tranposed
780| //         COMMON: scf.if
781| //         COMMON:   tt.dot_scaled
782| // COMMON-COUNT-2:   scf.yield
783| // COMMON-COUNT-2: ttg.local_dealloc
784| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 785-790
```mlir
785| #blocked = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [8, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
786| #blocked1 = #ttg.blocked<{sizePerThread = [16, 1], threadsPerWarp = [8, 8], warpsPerCTA = [1, 4], order = [0, 1]}>
787| #blocked2 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [2, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
788| #mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [2, 2], instrShape = [32, 32, 16], isTransposed = true}>
789| #shared = #ttg.swizzled_shared<{vec = 16, perPhase = 4, maxPhase = 4, order = [1, 0]}>
790| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 791-791
```mlir
791| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 792-809
```mlir
792|   tt.func public @pipelining_local_load_packed_transposed(%a_ptr: !tt.ptr<f8E5M2> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %b_ptr: !tt.ptr<i8> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %output_ptr: !tt.ptr<f32> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %M: i32 {tt.divisibility = 16 : i32}, %N: i32 {tt.divisibility = 16 : i32}, %K: i32 {tt.divisibility = 16 : i32}, %stride_scale: i32 {tt.divisibility = 16 : i32}, %stride_am: i32 {tt.divisibility = 16 : i32}, %stride_bn: i32 {tt.divisibility = 16 : i32}, %stride_cm: i32 {tt.divisibility = 16 : i32}) {
793|     %cst = arith.constant dense<128> : tensor<128x128xi32, #blocked>
794|     %cst_0 = arith.constant dense<128> : tensor<128x64xi32, #blocked1>
795|     %c0_i32 = arith.constant 0 : i32
796|     %c1_i32 = arith.constant 1 : i32
797|     %c127_i32 = arith.constant 127 : i32
798|     %c128_i32 = arith.constant 128 : i32
799|     %c2_i32 = arith.constant 2 : i32
800|     %cst_1 = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #mma>
801|     %0 = tt.get_program_id x : i32
802|     %1 = arith.addi %M, %c127_i32 : i32
803|     %2 = arith.divsi %1, %c128_i32 : i32
804|     %3 = arith.remsi %0, %2 : i32
805|     %4 = arith.divsi %0, %2 : i32
806|     %5 = arith.muli %3, %c128_i32 : i32
807|     %6 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
808|     %7 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked2}>>
809|     %8 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked2}>>
```
**EN:** This function-oriented block defines or enters `pipelining_local_load_packed_transposed`. Within it, the test exercises constants, lane/block index ranges, tt.func, arith.divsi, program IDs, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `pipelining_local_load_packed_transposed` 为核心。测试在其中演示 常量、lane/block 索引范围、tt.func、arith.divsi、程序 ID，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 810-827
```mlir
810|     %9 = tt.splat %5 : i32 -> tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
811|     %10 = tt.splat %5 : i32 -> tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked2}>>
812|     %11 = arith.addi %9, %6 : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
813|     %12 = arith.addi %10, %7 : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked2}>>
814|     %13 = arith.muli %4, %c128_i32 : i32
815|     %14 = arith.divsi %13, %c2_i32 : i32
816|     %15 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
817|     %16 = tt.splat %14 : i32 -> tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
818|     %17 = arith.addi %16, %15 : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
819|     %18 = tt.expand_dims %11 {axis = 1 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<128x1xi32, #blocked>
820|     %19 = tt.expand_dims %12 {axis = 1 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked2}>> -> tensor<128x1xi32, #blocked2>
821|     %20 = tt.splat %stride_am : i32 -> tensor<128x1xi32, #blocked>
822|     %21 = arith.muli %18, %20 : tensor<128x1xi32, #blocked>
823|     %22 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
824|     %23 = tt.expand_dims %22 {axis = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x128xi32, #blocked>
825|     %24 = tt.broadcast %21 : tensor<128x1xi32, #blocked> -> tensor<128x128xi32, #blocked>
826|     %25 = tt.broadcast %23 : tensor<1x128xi32, #blocked> -> tensor<128x128xi32, #blocked>
827|     %26 = arith.addi %24, %25 : tensor<128x128xi32, #blocked>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on broadcasted scalars or pointers, integer additions, shape expansion, integer multiplications, lane/block index ranges. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 广播后的标量或指针、整数加法、形状扩展、整数乘法、lane/block 索引范围。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 828-845
```mlir
828|     %27 = tt.splat %a_ptr : !tt.ptr<f8E5M2> -> tensor<128x128x!tt.ptr<f8E5M2>, #blocked>
829|     %28 = tt.addptr %27, %26 : tensor<128x128x!tt.ptr<f8E5M2>, #blocked>, tensor<128x128xi32, #blocked>
830|     %29 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked1}>>
831|     %30 = tt.expand_dims %29 {axis = 1 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked1}>> -> tensor<128x1xi32, #blocked1>
832|     %31 = tt.expand_dims %17 {axis = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked1}>> -> tensor<1x64xi32, #blocked1>
833|     %32 = tt.splat %stride_bn : i32 -> tensor<1x64xi32, #blocked1>
834|     %33 = arith.muli %31, %32 : tensor<1x64xi32, #blocked1>
835|     %34 = tt.broadcast %30 : tensor<128x1xi32, #blocked1> -> tensor<128x64xi32, #blocked1>
836|     %35 = tt.broadcast %33 : tensor<1x64xi32, #blocked1> -> tensor<128x64xi32, #blocked1>
837|     %36 = arith.addi %34, %35 : tensor<128x64xi32, #blocked1>
838|     %37 = tt.splat %b_ptr : !tt.ptr<i8> -> tensor<128x64x!tt.ptr<i8>, #blocked1>
839|     %38 = tt.addptr %37, %36 : tensor<128x64x!tt.ptr<i8>, #blocked1>, tensor<128x64xi32, #blocked1>
840|     %39 = arith.addi %K, %c127_i32 : i32
841|     %40 = arith.divsi %39, %c128_i32 : i32
842|     %accumulator:3 = scf.for %accumulator_2 = %c0_i32 to %40 step %c1_i32 iter_args(%arg11 = %cst_1, %arg12 = %28, %arg13 = %38) -> (tensor<128x128xf32, #mma>, tensor<128x128x!tt.ptr<f8E5M2>, #blocked>, tensor<128x64x!tt.ptr<i8>, #blocked1>)  : i32 {
843|       %60 = tt.load %arg12 : tensor<128x128x!tt.ptr<f8E5M2>, #blocked>
844|       %61 = tt.load %arg13 : tensor<128x64x!tt.ptr<i8>, #blocked1>
845|       %62 = ttg.convert_layout %60 : tensor<128x128xf8E5M2, #blocked> -> tensor<128x128xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 16}>>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining broadcasted scalars or pointers, pointer arithmetic, shape expansion, tensor broadcasting, integer additions.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 广播后的标量或指针、指针算术、形状扩展、张量广播、整数加法。

### Lines 846-863
```mlir
846|       %63 = ttg.local_alloc %61 : (tensor<128x64xi8, #blocked1>) -> !ttg.memdesc<128x64xi8, #shared, #smem>
847|       %64 = amdg.local_load_packed_tranposed %63 : !ttg.memdesc<128x64xi8, #shared, #smem> -> tensor<64x128xi8, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>
848|       %65 = tt.dot_scaled %62, %64, %arg11 lhs = e5m2 rhs = e2m1 {fastMath = false} : tensor<128x128xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 16}>> * tensor<64x128xi8, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>> -> tensor<128x128xf32, #mma>
849|       %66 = tt.addptr %arg12, %cst : tensor<128x128x!tt.ptr<f8E5M2>, #blocked>, tensor<128x128xi32, #blocked>
850|       %67 = tt.addptr %arg13, %cst_0 : tensor<128x64x!tt.ptr<i8>, #blocked1>, tensor<128x64xi32, #blocked1>
851|       scf.yield %65, %66, %67 : tensor<128x128xf32, #mma>, tensor<128x128x!tt.ptr<f8E5M2>, #blocked>, tensor<128x64x!tt.ptr<i8>, #blocked1>
852|     } {tt.num_stages = 2 : i32}
853|     %41 = tt.splat %13 : i32 -> tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked2}>>
854|     %42 = arith.addi %41, %8 : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked2}>>
855|     %43 = tt.splat %stride_cm : i32 -> tensor<128x1xi32, #blocked2>
856|     %44 = arith.muli %43, %19 : tensor<128x1xi32, #blocked2>
857|     %45 = tt.splat %output_ptr : !tt.ptr<f32> -> tensor<128x1x!tt.ptr<f32>, #blocked2>
858|     %46 = tt.addptr %45, %44 : tensor<128x1x!tt.ptr<f32>, #blocked2>, tensor<128x1xi32, #blocked2>
859|     %47 = tt.expand_dims %42 {axis = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked2}>> -> tensor<1x128xi32, #blocked2>
860|     %48 = tt.broadcast %46 : tensor<128x1x!tt.ptr<f32>, #blocked2> -> tensor<128x128x!tt.ptr<f32>, #blocked2>
861|     %49 = tt.broadcast %47 : tensor<1x128xi32, #blocked2> -> tensor<128x128xi32, #blocked2>
862|     %50 = tt.addptr %48, %49 : tensor<128x128x!tt.ptr<f32>, #blocked2>, tensor<128x128xi32, #blocked2>
863|     %51 = tt.splat %M : i32 -> tensor<128x1xi32, #blocked2>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining pointer arithmetic, broadcasted scalars or pointers, tensor broadcasting, shared/local memory allocation, amdg.local_load_packed_tranposed.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 指针算术、广播后的标量或指针、张量广播、共享/本地内存分配、amdg.local_load_packed_tranposed。

### Lines 864-876
```mlir
864|     %52 = arith.cmpi slt, %19, %51 : tensor<128x1xi32, #blocked2>
865|     %53 = tt.splat %N : i32 -> tensor<1x128xi32, #blocked2>
866|     %54 = arith.cmpi slt, %47, %53 : tensor<1x128xi32, #blocked2>
867|     %55 = tt.broadcast %52 : tensor<128x1xi1, #blocked2> -> tensor<128x128xi1, #blocked2>
868|     %56 = tt.broadcast %54 : tensor<1x128xi1, #blocked2> -> tensor<128x128xi1, #blocked2>
869|     %57 = arith.andi %55, %56 : tensor<128x128xi1, #blocked2>
870|     %58 = ttg.convert_layout %50 : tensor<128x128x!tt.ptr<f32>, #blocked2> -> tensor<128x128x!tt.ptr<f32>, #mma>
871|     %59 = ttg.convert_layout %57 : tensor<128x128xi1, #blocked2> -> tensor<128x128xi1, #mma>
872|     tt.store %58, %accumulator#0, %59 : tensor<128x128x!tt.ptr<f32>, #mma>
873|     tt.return
874|   }
875| }
876| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on integer comparisons, tensor broadcasting, layout conversions, broadcasted scalars or pointers, arith.andi. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 整数比较、张量广播、布局转换、广播后的标量或指针、arith.andi。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 877-877
```mlir
877| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 878-890
```mlir
878| 
879| // COMMON-LABEL: bypass_lds_b_operand
880| 
881| //         SYNC: scf.for
882| //         SYNC: %[[load:.+]] = tt.load {{.*}} : tensor<8x2048x!tt.ptr<i8>, #linear>
883| //         SYNC: %[[reshape1:.+]] = tt.reshape %arg24
884| //         SYNC: %[[trans1:.+]] = tt.trans %[[reshape1]]
885| //         SYNC: %[[reshape2:.+]] = tt.reshape %[[trans1]]
886| //         SYNC: %[[trans2:.+]] = tt.trans %[[reshape2]] {{.*}} -> tensor<128x128xi8, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>
887| //         SYNC: tt.dot_scaled {{.*}}, %[[trans2]]
888| //         SYNC: scf.yield {{.*}}, %[[load]]
889| 
890| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 891-907
```mlir
891| #blocked = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [8, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
892| #blocked1 = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [1, 64], warpsPerCTA = [2, 2], order = [1, 0]}>
893| #blocked2 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 64], warpsPerCTA = [4, 1], order = [1, 0]}>
894| #blocked3 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 64], warpsPerCTA = [1, 4], order = [1, 0]}>
895| #linear = #ttg.linear<{register = [[0, 2], [0, 1]], lane = [[0, 4], [0, 8], [0, 16], [0, 32], [0, 64], [0, 128]], warp = [[0, 0], [0, 0]], block = []}>
896| #linear1 = #ttg.linear<{register = [[0, 0, 0, 0, 1, 0, 0], [0, 0, 0, 0, 0, 1, 0]], lane = [[0, 0, 0, 1, 0, 0, 0], [0, 0, 0, 2, 0, 0, 0], [0, 0, 0, 4, 0, 0, 0], [0, 0, 0, 8, 0, 0, 0], [0, 0, 1, 0, 0, 0, 0], [0, 0, 2, 0, 0, 0, 0]], warp = [[0, 0, 0, 0, 0, 0, 0], [0, 0, 0, 0, 0, 0, 0]], block = []}>
897| #linear2 = #ttg.linear<{register = [[0, 0, 0, 0, 1, 0, 0], [0, 1, 0, 0, 0, 0, 0]], lane = [[0, 0, 1, 0, 0, 0, 0], [0, 0, 2, 0, 0, 0, 0], [0, 0, 4, 0, 0, 0, 0], [0, 0, 8, 0, 0, 0, 0], [0, 0, 0, 0, 0, 1, 0], [0, 0, 0, 0, 0, 2, 0]], warp = [[0, 0, 0, 0, 0, 0, 0], [0, 0, 0, 0, 0, 0, 0]], block = []}>
898| #linear3 = #ttg.linear<{register = [[0, 4], [16, 0]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 1], [0, 2]], warp = [[0, 0], [0, 0]], block = []}>
899| #linear4 = #ttg.linear<{register = [[0, 2], [0, 1]], lane = [[0, 4], [0, 8], [0, 16], [0, 32], [0, 64], [0, 128]], warp = [[1, 0], [2, 0]], block = []}>
900| #linear5 = #ttg.linear<{register = [[0, 0, 0, 0, 1, 0, 0], [0, 0, 0, 0, 0, 1, 0]], lane = [[0, 0, 0, 1, 0, 0, 0], [0, 0, 0, 2, 0, 0, 0], [0, 0, 0, 4, 0, 0, 0], [0, 0, 0, 8, 0, 0, 0], [0, 0, 1, 0, 0, 0, 0], [0, 0, 2, 0, 0, 0, 0]], warp = [[1, 0, 0, 0, 0, 0, 0], [2, 0, 0, 0, 0, 0, 0]], block = []}>
901| #linear6 = #ttg.linear<{register = [[0, 0, 0, 0, 1, 0, 0], [0, 1, 0, 0, 0, 0, 0]], lane = [[0, 0, 1, 0, 0, 0, 0], [0, 0, 2, 0, 0, 0, 0], [0, 0, 4, 0, 0, 0, 0], [0, 0, 8, 0, 0, 0, 0], [0, 0, 0, 0, 0, 1, 0], [0, 0, 0, 0, 0, 2, 0]], warp = [[1, 0, 0, 0, 0, 0, 0], [2, 0, 0, 0, 0, 0, 0]], block = []}>
902| #linear7 = #ttg.linear<{register = [[0, 4], [16, 0]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 1], [0, 2]], warp = [[32, 0], [64, 0]], block = []}>
903| #linear8 = #ttg.linear<{register = [[0, 1], [0, 2], [0, 4], [0, 8], [0, 1024], [1, 0]], lane = [[0, 16], [0, 32], [0, 64], [0, 128], [0, 256], [0, 512]], warp = [[2, 0], [4, 0]], block = []}>
904| #linear9 = #ttg.linear<{register = [[0, 0, 0, 0, 0, 1], [0, 0, 0, 0, 0, 2], [0, 0, 0, 0, 0, 4], [0, 0, 0, 0, 0, 8], [0, 0, 4, 0, 0, 0], [0, 1, 0, 0, 0, 0]], lane = [[0, 0, 0, 0, 1, 0], [0, 0, 0, 0, 2, 0], [0, 0, 0, 0, 4, 0], [0, 0, 0, 0, 8, 0], [0, 0, 1, 0, 0, 0], [0, 0, 2, 0, 0, 0]], warp = [[0, 2, 0, 0, 0, 0], [0, 4, 0, 0, 0, 0]], block = []}>
905| #linear10 = #ttg.linear<{register = [[0, 0, 0, 0, 0, 1], [0, 0, 0, 0, 0, 2], [0, 0, 0, 0, 0, 4], [0, 0, 0, 0, 0, 8], [0, 0, 0, 4, 0, 0], [0, 1, 0, 0, 0, 0]], lane = [[0, 0, 1, 0, 0, 0], [0, 0, 2, 0, 0, 0], [0, 0, 4, 0, 0, 0], [0, 0, 8, 0, 0, 0], [0, 0, 0, 1, 0, 0], [0, 0, 0, 2, 0, 0]], warp = [[0, 2, 0, 0, 0, 0], [0, 4, 0, 0, 0, 0]], block = []}>
906| #linear11 = #ttg.linear<{register = [[0, 1], [0, 2], [0, 4], [0, 8], [0, 64], [16, 0]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 16], [0, 32]], warp = [[32, 0], [64, 0]], block = []}>
907| #mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [1, 4], tilesPerWarp = [2, 2], instrShape = [16, 16, 32], isTransposed = true}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 908-908
```mlir
908| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 909-926
```mlir
909|   tt.func public @bypass_lds_b_operand(%a_ptr: !tt.ptr<i8> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %b_ptr: !tt.ptr<i8> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %c_ptr: !tt.ptr<bf16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %a_scales_ptr: !tt.ptr<i8> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %b_scales_ptr: !tt.ptr<i8> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %M: i32 {tt.divisibility = 16 : i32}, %N: i32 {tt.divisibility = 16 : i32}, %K: i32 {tt.divisibility = 16 : i32},  %stride_am: i32 {tt.divisibility = 16 : i32}, %stride_bn: i32 {tt.divisibility = 16 : i32}, %stride_ck: i32 {tt.divisibility = 16 : i32}, %stride_cm: i32 {tt.divisibility = 16 : i32}, %stride_asm: i32 {tt.divisibility = 16 : i32}, %stride_bsn: i32 {tt.divisibility = 16 : i32})  attributes {noinline = false} {
910|     %cst = arith.constant dense<128> : tensor<32x128xi32, #blocked>
911|     %cst_0 = arith.constant dense<2048> : tensor<8x2048xi32, #blocked1>
912|     %cst_1 = arith.constant dense<256> : tensor<4x256xi32, #blocked2>
913|     %c1_i32 = arith.constant 1 : i32
914|     %pid_unified = arith.constant 7 : i32
915|     %c64_i32 = arith.constant 64 : i32
916|     %num_pid_n = arith.constant 127 : i32
917|     %cst_2 = arith.constant dense<256> : tensor<1x256xi32, #blocked3>
918|     %c128_i32 = arith.constant 128 : i32
919|     %c0_i32 = arith.constant 0 : i32
920|     %c32_i32 = arith.constant 32 : i32
921|     %c8_i32 = arith.constant 8 : i32
922|     %c4_i32 = arith.constant 4 : i32
923|     %cst_3 = arith.constant dense<0.000000e+00> : tensor<32x128xf32, #mma>
924|     %pid_unified_4 = tt.get_program_id x : i32
925|     %xcd = arith.remsi %pid_unified_4, %c8_i32 : i32
926|     %local_pid = arith.divsi %pid_unified_4, %c8_i32 : i32
```
**EN:** This function-oriented block defines or enters `bypass_lds_b_operand`. Within it, the test exercises constants, tt.func, program IDs, arith.remsi, arith.divsi, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `bypass_lds_b_operand` 为核心。测试在其中演示 常量、tt.func、程序 ID、arith.remsi、arith.divsi，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 927-944
```mlir
927|     %pid = arith.muli %xcd, %c8_i32 : i32
928|     %pid_9 = arith.addi %pid, %local_pid : i32
929|     %num_pid_n_7 = arith.addi %N, %num_pid_n : i32
930|     %num_pid_n_8 = arith.divsi %num_pid_n_7, %c128_i32 : i32
931|     %pid_n = arith.remsi %pid_9, %num_pid_n_8 : i32
932|     %offs_bn = arith.muli %pid_n, %c8_i32 : i32
933|     %offs_bn_15 = tt.make_range {end = 8 : i32, start = 0 : i32} : tensor<8xi32, #ttg.slice<{dim = 1, parent = #blocked1}>>
934|     %offs_bn_16 = tt.splat %offs_bn : i32 -> tensor<8xi32, #ttg.slice<{dim = 1, parent = #blocked1}>>
935|     %offs_bn_17 = arith.addi %offs_bn_16, %offs_bn_15 : tensor<8xi32, #ttg.slice<{dim = 1, parent = #blocked1}>>
936|     %offs_bn_18 = tt.splat %N : i32 -> tensor<8xi32, #ttg.slice<{dim = 1, parent = #blocked1}>>
937|     %offs_bn_19 = arith.remsi %offs_bn_17, %offs_bn_18 : tensor<8xi32, #ttg.slice<{dim = 1, parent = #blocked1}>>
938|     %a_ptrs_28 = tt.splat %a_ptr : !tt.ptr<i8> -> tensor<32x128x!tt.ptr<i8>, #blocked>
939|     %b_ptrs = tt.expand_dims %offs_bn_19 {axis = 1 : i32} : tensor<8xi32, #ttg.slice<{dim = 1, parent = #blocked1}>> -> tensor<8x1xi32, #blocked1>
940|     %b_ptrs_29 = tt.splat %stride_bn : i32 -> tensor<8x1xi32, #blocked1>
941|     %b_ptrs_30 = arith.muli %b_ptrs, %b_ptrs_29 : tensor<8x1xi32, #blocked1>
942|     %b_ptrs_31 = tt.make_range {end = 2048 : i32, start = 0 : i32} : tensor<2048xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
943|     %b_ptrs_32 = tt.expand_dims %b_ptrs_31 {axis = 0 : i32} : tensor<2048xi32, #ttg.slice<{dim = 0, parent = #blocked1}>> -> tensor<1x2048xi32, #blocked1>
944|     %b_ptrs_33 = tt.broadcast %b_ptrs_30 : tensor<8x1xi32, #blocked1> -> tensor<8x2048xi32, #blocked1>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on broadcasted scalars or pointers, integer multiplications, integer additions, arith.remsi, lane/block index ranges. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 广播后的标量或指针、整数乘法、整数加法、arith.remsi、lane/block 索引范围。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 945-962
```mlir
945|     %b_ptrs_34 = tt.broadcast %b_ptrs_32 : tensor<1x2048xi32, #blocked1> -> tensor<8x2048xi32, #blocked1>
946|     %b_ptrs_35 = arith.addi %b_ptrs_33, %b_ptrs_34 : tensor<8x2048xi32, #blocked1>
947|     %b_ptrs_36 = tt.splat %b_ptr : !tt.ptr<i8> -> tensor<8x2048x!tt.ptr<i8>, #blocked1>
948|     %b_ptrs_37 = tt.addptr %b_ptrs_36, %b_ptrs_35 : tensor<8x2048x!tt.ptr<i8>, #blocked1>, tensor<8x2048xi32, #blocked1>
949|     %b_scale_ptrs_53 = tt.splat %b_scales_ptr : !tt.ptr<i8> -> tensor<4x256x!tt.ptr<i8>, #blocked2>
950|     %a_scale_ptrs_56 = tt.splat %a_scales_ptr : !tt.ptr<i8> -> tensor<1x256x!tt.ptr<i8>, #blocked3>
951|     %accumulator:5 = scf.for %accumulator_83 = %c0_i32 to %c64_i32 step %c1_i32 iter_args(%a_scale_ptrs_84 = %a_scale_ptrs_56, %arg16 = %cst_3, %b_scale_ptrs_85 = %b_scale_ptrs_53, %a_ptrs_86 = %a_ptrs_28, %b_ptrs_87 = %b_ptrs_37) -> (tensor<1x256x!tt.ptr<i8>, #blocked3>, tensor<32x128xf32, #mma>, tensor<4x256x!tt.ptr<i8>, #blocked2>, tensor<32x128x!tt.ptr<i8>, #blocked>, tensor<8x2048x!tt.ptr<i8>, #blocked1>)  : i32 {
952|       %a_scales = tt.load %a_scale_ptrs_84 : tensor<1x256x!tt.ptr<i8>, #blocked3>
953|       %a_scales_88 = ttg.convert_layout %a_scales : tensor<1x256xi8, #blocked3> -> tensor<1x256xi8, #linear>
954|       %a_scales_89 = tt.reshape %a_scales_88 : tensor<1x256xi8, #linear> -> tensor<1x1x4x16x2x2x1xi8, #linear1>
955|       %a_scales_90 = tt.trans %a_scales_89 {order = array<i32: 0, 5, 3, 1, 4, 2, 6>} : tensor<1x1x4x16x2x2x1xi8, #linear1> -> tensor<1x2x16x1x2x4x1xi8, #linear2>
956|       %a_scales_91 = tt.reshape %a_scales_90 : tensor<1x2x16x1x2x4x1xi8, #linear2> -> tensor<32x8xi8, #linear3>
957|       %b_scales = tt.load %b_scale_ptrs_85 : tensor<4x256x!tt.ptr<i8>, #blocked2>
958|       %b_scales_92 = ttg.convert_layout %b_scales : tensor<4x256xi8, #blocked2> -> tensor<4x256xi8, #linear4>
959|       %b_scales_93 = tt.reshape %b_scales_92 : tensor<4x256xi8, #linear4> -> tensor<4x1x4x16x2x2x1xi8, #linear5>
960|       %b_scales_94 = tt.trans %b_scales_93 {order = array<i32: 0, 5, 3, 1, 4, 2, 6>} : tensor<4x1x4x16x2x2x1xi8, #linear5> -> tensor<4x2x16x1x2x4x1xi8, #linear6>
961|       %b_scales_95 = tt.reshape %b_scales_94 : tensor<4x2x16x1x2x4x1xi8, #linear6> -> tensor<128x8xi8, #linear7>
962|       %a = tt.load %a_ptrs_86 : tensor<32x128x!tt.ptr<i8>, #blocked>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining tensor reshaping, broadcasted scalars or pointers, masked or vectorized loads, layout conversions, transpose-like layout changes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 张量重塑、广播后的标量或指针、带掩码或向量化的加载、布局转换、转置类布局变换。

### Lines 963-980
```mlir
963|       %b = tt.load %b_ptrs_87 : tensor<8x2048x!tt.ptr<i8>, #blocked1>
964|       %accumulator_96 = ttg.convert_layout %b : tensor<8x2048xi8, #blocked1> -> tensor<8x2048xi8, #linear8>
965|       %b_97 = tt.reshape %accumulator_96 : tensor<8x2048xi8, #linear8> -> tensor<1x8x8x1x16x16xi8, #linear9>
966|       %b_98 = tt.trans %b_97 {order = array<i32: 0, 1, 4, 2, 3, 5>} : tensor<1x8x8x1x16x16xi8, #linear9> -> tensor<1x8x16x8x1x16xi8, #linear10>
967|       %b_99 = tt.reshape %b_98 : tensor<1x8x16x8x1x16xi8, #linear10> -> tensor<128x128xi8, #linear11>
968|       %b_100 = tt.trans %b_99 {order = array<i32: 1, 0>} : tensor<128x128xi8, #linear11> -> tensor<128x128xi8, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>
969|       %a_101 = ttg.convert_layout %a : tensor<32x128xi8, #blocked> -> tensor<32x128xi8, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 16}>>
970|       %accumulator_102 = tt.dot_scaled %a_101 scale %a_scales_91, %b_100 scale %b_scales_95, %cst_3 lhs = e2m1 rhs = e2m1 {fastMath = false} : tensor<32x128xi8, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 16}>>, tensor<32x8xi8, #linear3> * tensor<128x128xi8, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>, tensor<128x8xi8, #linear7> -> tensor<32x128xf32, #mma>
971|       %accumulator_103 = arith.addf %arg16, %accumulator_102 : tensor<32x128xf32, #mma>
972|       %a_ptrs_104 = tt.addptr %a_ptrs_86, %cst : tensor<32x128x!tt.ptr<i8>, #blocked>, tensor<32x128xi32, #blocked>
973|       %b_ptrs_105 = tt.addptr %b_ptrs_87, %cst_0 : tensor<8x2048x!tt.ptr<i8>, #blocked1>, tensor<8x2048xi32, #blocked1>
974|       %a_scale_ptrs_106 = tt.addptr %a_scale_ptrs_84, %cst_2 : tensor<1x256x!tt.ptr<i8>, #blocked3>, tensor<1x256xi32, #blocked3>
975|       %b_scale_ptrs_107 = tt.addptr %b_scale_ptrs_85, %cst_1 : tensor<4x256x!tt.ptr<i8>, #blocked2>, tensor<4x256xi32, #blocked2>
976|       scf.yield %a_scale_ptrs_106, %accumulator_103, %b_scale_ptrs_107, %a_ptrs_104, %b_ptrs_105 : tensor<1x256x!tt.ptr<i8>, #blocked3>, tensor<32x128xf32, #mma>, tensor<4x256x!tt.ptr<i8>, #blocked2>, tensor<32x128x!tt.ptr<i8>, #blocked>, tensor<8x2048x!tt.ptr<i8>, #blocked1>
977|     }
978|     tt.return
979|   }
980| }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining pointer arithmetic, layout conversions, tensor reshaping, transpose-like layout changes, masked or vectorized loads.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 指针算术、布局转换、张量重塑、转置类布局变换、带掩码或向量化的加载。

### Lines 982-983
```mlir
982| 
983| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 983-994
```mlir
983| 
984| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 16], warpsPerCTA = [1, 4], order = [1, 0]}>
985| #mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [4, 1], instrShape = [32, 32, 16], isTransposed = true}>
986| 
987| // ASYNC-NOT: ttg.swizzled_shared
988| // ASYNC: [[PADDED_ENC:#.*]] = #ttg.padded_shared
989| // ASYNC-SAME{LITERAL}: {offset = [[0, 1], [0, 2], [0, 4], [0, 8], [0, 16], [0, 32], [0, 64], [16, 0], [32, 0], [1, 0], [2, 0], [4, 0], [8, 0], [64, 0]], block = []}
990| // ASYNC-NOT: ttg.padded_shared
991| // ASYNC-NOT: ttg.swizzled_shared
992| 
993| // SYNC-NOT: ttg.padded_shared
994| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 995-996
```mlir
995| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
996|   // COMMON-LABEL: loop_expect_padded_layouts
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 997-1012
```mlir
 997|   tt.func public @loop_expect_padded_layouts(%arg0: i32, %arg1: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.constancy = dense<1> : tensor<2xi32>, tt.contiguity = dense<[1, 8]> : tensor<2xi32>, tt.divisibility = dense<[1, 16]> : tensor<2xi32>}, %arg2: tensor<128x128x!tt.ptr<f16>, #mma>) {
 998|     %c1_i32 = arith.constant 1 : i32
 999|     %c0_i32 = arith.constant 0 : i32
1000|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf16, #mma>
1001|     %cst_0 = arith.constant dense<0.000000e+00> : tensor<128x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>
1002|     %0 = scf.for %arg3 = %c0_i32 to %arg0 step %c1_i32 iter_args(%arg4 = %cst) -> (tensor<128x128xf16, #mma>)  : i32 {
1003|       %1 = tt.load %arg1 : tensor<128x128x!tt.ptr<f16>, #blocked>
1004|       %2 = ttg.convert_layout %1 : tensor<128x128xf16, #blocked> -> tensor<128x128xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>
1005|       %3 = tt.dot %2, %cst_0, %arg4 : tensor<128x128xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>> * tensor<128x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>> -> tensor<128x128xf16, #mma>
1006|       scf.yield %3 : tensor<128x128xf16, #mma>
1007|     }
1008|     tt.store %arg2, %0 : tensor<128x128x!tt.ptr<f16>, #mma>
1009|     tt.return
1010|   }
1011| }
1012| 
```
**EN:** This function-oriented block defines or enters `loop_expect_padded_layouts`. Within it, the test exercises constants, tt.func, structured loops, masked or vectorized loads, layout conversions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `loop_expect_padded_layouts` 为核心。测试在其中演示 常量、tt.func、结构化循环、带掩码或向量化的加载、布局转换，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1013-1013
```mlir
1013| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1014-1017
```mlir
1014| // Negative tests for padded encodings on gfx950
1015| 
1016| // Unsupported kWidth
1017| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1018-1021
```mlir
1018| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 16], warpsPerCTA = [1, 4], order = [1, 0]}>
1019| #mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [4, 1], instrShape = [32, 32, 16], isTransposed = true}>
1020| 
1021| // COMMON-NOT: ttg.padded_shared
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1022-1023
```mlir
1022| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
1023|   // COMMON-LABEL: loop_padding_too_small_vector
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 1024-1039
```mlir
1024|   tt.func public @loop_padding_too_small_vector(%arg0: i32, %arg1: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.constancy = dense<1> : tensor<2xi32>, tt.contiguity = dense<[1, 8]> : tensor<2xi32>, tt.divisibility = dense<[1, 16]> : tensor<2xi32>}, %arg2: tensor<128x128x!tt.ptr<f16>, #mma>) {
1025|     %c1_i32 = arith.constant 1 : i32
1026|     %c0_i32 = arith.constant 0 : i32
1027|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf16, #mma>
1028|     %cst_0 = arith.constant dense<0.000000e+00> : tensor<128x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>>
1029|     %0 = scf.for %arg3 = %c0_i32 to %arg0 step %c1_i32 iter_args(%arg4 = %cst) -> (tensor<128x128xf16, #mma>)  : i32 {
1030|       %1 = tt.load %arg1 : tensor<128x128x!tt.ptr<f16>, #blocked>
1031|       %2 = ttg.convert_layout %1 : tensor<128x128xf16, #blocked> -> tensor<128x128xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>>
1032|       %3 = tt.dot %2, %cst_0, %arg4 : tensor<128x128xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>> * tensor<128x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>> -> tensor<128x128xf16, #mma>
1033|       scf.yield %3 : tensor<128x128xf16, #mma>
1034|     }
1035|     tt.store %arg2, %0 : tensor<128x128x!tt.ptr<f16>, #mma>
1036|     tt.return
1037|   }
1038| }
1039| 
```
**EN:** This function-oriented block defines or enters `loop_padding_too_small_vector`. Within it, the test exercises constants, tt.func, structured loops, masked or vectorized loads, layout conversions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `loop_padding_too_small_vector` 为核心。测试在其中演示 常量、tt.func、结构化循环、带掩码或向量化的加载、布局转换，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1040-1040
```mlir
1040| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1041-1043
```mlir
1041| 
1042| // Unsupported instrShape
1043| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1044-1047
```mlir
1044| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 16], warpsPerCTA = [1, 4], order = [1, 0]}>
1045| #mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [4, 1], instrShape = [64, 4, 16], isTransposed = true}>
1046| 
1047| // COMMON-NOT: ttg.padded_shared
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1048-1049
```mlir
1048| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
1049|   // COMMON-LABEL: loop_padding_invalid_instr_shape
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 1050-1065
```mlir
1050|   tt.func public @loop_padding_invalid_instr_shape(%arg0: i32, %arg1: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.constancy = dense<1> : tensor<2xi32>, tt.contiguity = dense<[1, 8]> : tensor<2xi32>, tt.divisibility = dense<[1, 16]> : tensor<2xi32>}, %arg2: tensor<128x128x!tt.ptr<f16>, #mma>) {
1051|     %c1_i32 = arith.constant 1 : i32
1052|     %c0_i32 = arith.constant 0 : i32
1053|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf16, #mma>
1054|     %cst_0 = arith.constant dense<0.000000e+00> : tensor<128x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>>
1055|     %0 = scf.for %arg3 = %c0_i32 to %arg0 step %c1_i32 iter_args(%arg4 = %cst) -> (tensor<128x128xf16, #mma>)  : i32 {
1056|       %1 = tt.load %arg1 : tensor<128x128x!tt.ptr<f16>, #blocked>
1057|       %2 = ttg.convert_layout %1 : tensor<128x128xf16, #blocked> -> tensor<128x128xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>>
1058|       %3 = tt.dot %2, %cst_0, %arg4 : tensor<128x128xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>> * tensor<128x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>> -> tensor<128x128xf16, #mma>
1059|       scf.yield %3 : tensor<128x128xf16, #mma>
1060|     }
1061|     tt.store %arg2, %0 : tensor<128x128x!tt.ptr<f16>, #mma>
1062|     tt.return
1063|   }
1064| }
1065| 
```
**EN:** This function-oriented block defines or enters `loop_padding_invalid_instr_shape`. Within it, the test exercises constants, tt.func, structured loops, masked or vectorized loads, layout conversions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `loop_padding_invalid_instr_shape` 为核心。测试在其中演示 常量、tt.func、结构化循环、带掩码或向量化的加载、布局转换，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1066-1066
```mlir
1066| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1067-1069
```mlir
1067| 
1068| // Block size too small
1069| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1070-1073
```mlir
1070| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 16], warpsPerCTA = [1, 4], order = [1, 0]}>
1071| #mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [4, 1], instrShape = [16, 16, 32], isTransposed = true}>
1072| 
1073| // COMMON-NOT: ttg.padded_shared
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1074-1075
```mlir
1074| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
1075|   // COMMON-LABEL: loop_padding_block_size_too_small
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 1076-1091
```mlir
1076|   tt.func public @loop_padding_block_size_too_small(%arg0: i32, %arg1: tensor<16x128x!tt.ptr<f16>, #blocked> {tt.constancy = dense<1> : tensor<2xi32>, tt.contiguity = dense<[1, 8]> : tensor<2xi32>, tt.divisibility = dense<[1, 16]> : tensor<2xi32>}, %arg2: tensor<16x16x!tt.ptr<f16>, #mma>) {
1077|     %c1_i32 = arith.constant 1 : i32
1078|     %c0_i32 = arith.constant 0 : i32
1079|     %cst = arith.constant dense<0.000000e+00> : tensor<16x16xf16, #mma>
1080|     %cst_0 = arith.constant dense<0.000000e+00> : tensor<128x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>>
1081|     %0 = scf.for %arg3 = %c0_i32 to %arg0 step %c1_i32 iter_args(%arg4 = %cst) -> (tensor<16x16xf16, #mma>)  : i32 {
1082|       %1 = tt.load %arg1 : tensor<16x128x!tt.ptr<f16>, #blocked>
1083|       %2 = ttg.convert_layout %1 : tensor<16x128xf16, #blocked> -> tensor<16x128xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>>
1084|       %3 = tt.dot %2, %cst_0, %arg4 : tensor<16x128xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>> * tensor<128x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>> -> tensor<16x16xf16, #mma>
1085|       scf.yield %3 : tensor<16x16xf16, #mma>
1086|     }
1087|     tt.store %arg2, %0 : tensor<16x16x!tt.ptr<f16>, #mma>
1088|     tt.return
1089|   }
1090| }
1091| 
```
**EN:** This function-oriented block defines or enters `loop_padding_block_size_too_small`. Within it, the test exercises constants, tt.func, structured loops, masked or vectorized loads, layout conversions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `loop_padding_block_size_too_small` 为核心。测试在其中演示 常量、tt.func、结构化循环、带掩码或向量化的加载、布局转换，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1092-1092
```mlir
1092| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1093-1095
```mlir
1093| 
1094| // dtype > 2 bytes
1095| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1096-1098
```mlir
1096| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 16], warpsPerCTA = [1, 4], order = [1, 0]}>
1097| #mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [4, 1], instrShape = [16, 16, 32], isTransposed = true}>
1098| // COMMON-NOT: ttg.padded_shared
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1099-1100
```mlir
1099| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
1100|   // COMMON-LABEL: loop_padding_block_size_too_small
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 1101-1116
```mlir
1101|   tt.func public @loop_padding_block_size_too_small(%arg0: i32, %arg1: tensor<16x128x!tt.ptr<f32>, #blocked> {tt.constancy = dense<1> : tensor<2xi32>, tt.contiguity = dense<[1, 8]> : tensor<2xi32>, tt.divisibility = dense<[1, 16]> : tensor<2xi32>}, %arg2: tensor<16x16x!tt.ptr<f32>, #mma>) {
1102|     %c1_i32 = arith.constant 1 : i32
1103|     %c0_i32 = arith.constant 0 : i32
1104|     %cst = arith.constant dense<0.000000e+00> : tensor<16x16xf32, #mma>
1105|     %cst_0 = arith.constant dense<0.000000e+00> : tensor<128x16xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>>
1106|     %0 = scf.for %arg3 = %c0_i32 to %arg0 step %c1_i32 iter_args(%arg4 = %cst) -> (tensor<16x16xf32, #mma>)  : i32 {
1107|       %1 = tt.load %arg1 : tensor<16x128x!tt.ptr<f32>, #blocked>
1108|       %2 = ttg.convert_layout %1 : tensor<16x128xf32, #blocked> -> tensor<16x128xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>>
1109|       %3 = tt.dot %2, %cst_0, %arg4, inputPrecision = tf32 : tensor<16x128xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>> * tensor<128x16xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>> -> tensor<16x16xf32, #mma>
1110|       scf.yield %3 : tensor<16x16xf32, #mma>
1111|     }
1112|     tt.store %arg2, %0 : tensor<16x16x!tt.ptr<f32>, #mma>
1113|     tt.return
1114|   }
1115| }
1116| 
```
**EN:** This function-oriented block defines or enters `loop_padding_block_size_too_small`. Within it, the test exercises constants, tt.func, structured loops, masked or vectorized loads, layout conversions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `loop_padding_block_size_too_small` 为核心。测试在其中演示 常量、tt.func、结构化循环、带掩码或向量化的加载、布局转换，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1117-1117
```mlir
1117| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1118-1120
```mlir
1118| 
1119| // dtype < 2 bytes
1120| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1121-1123
```mlir
1121| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 16], warpsPerCTA = [1, 4], order = [1, 0]}>
1122| #mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [4, 1], instrShape = [16, 16, 32], isTransposed = true}>
1123| // COMMON-NOT: ttg.padded_shared
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1124-1125
```mlir
1124| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
1125|   // COMMON-LABEL: loop_padding_block_size_too_small
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 1126-1141
```mlir
1126|   tt.func public @loop_padding_block_size_too_small(%arg0: i32, %arg1: tensor<16x128x!tt.ptr<f8E5M2>, #blocked> {tt.constancy = dense<1> : tensor<2xi32>, tt.contiguity = dense<[1, 8]> : tensor<2xi32>, tt.divisibility = dense<[1, 16]> : tensor<2xi32>}, %arg2: tensor<16x16x!tt.ptr<f8E5M2>, #mma>) {
1127|     %c1_i32 = arith.constant 1 : i32
1128|     %c0_i32 = arith.constant 0 : i32
1129|     %cst = arith.constant dense<0.000000e+00> : tensor<16x16xf8E5M2, #mma>
1130|     %cst_0 = arith.constant dense<0.000000e+00> : tensor<128x16xf8E5M2, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>>
1131|     %0 = scf.for %arg3 = %c0_i32 to %arg0 step %c1_i32 iter_args(%arg4 = %cst) -> (tensor<16x16xf8E5M2, #mma>)  : i32 {
1132|       %1 = tt.load %arg1 : tensor<16x128x!tt.ptr<f8E5M2>, #blocked>
1133|       %2 = ttg.convert_layout %1 : tensor<16x128xf8E5M2, #blocked> -> tensor<16x128xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>>
1134|       %3 = tt.dot %2, %cst_0, %arg4 : tensor<16x128xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>> * tensor<128x16xf8E5M2, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>> -> tensor<16x16xf8E5M2, #mma>
1135|       scf.yield %3 : tensor<16x16xf8E5M2, #mma>
1136|     }
1137|     tt.store %arg2, %0 : tensor<16x16x!tt.ptr<f8E5M2>, #mma>
1138|     tt.return
1139|   }
1140| }
1141| 
```
**EN:** This function-oriented block defines or enters `loop_padding_block_size_too_small`. Within it, the test exercises constants, tt.func, structured loops, masked or vectorized loads, layout conversions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `loop_padding_block_size_too_small` 为核心。测试在其中演示 常量、tt.func、结构化循环、带掩码或向量化的加载、布局转换，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1142-1142
```mlir
1142| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1143-1145
```mlir
1143| 
1144| // small Block size 32x64
1145| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1146-1150
```mlir
1146| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
1147| #mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [1, 4], instrShape = [16, 16, 32], isTransposed = true}>
1148| 
1149| // ASYNC-NOT: ttg.swizzled_shared
1150| // ASYNC{LITERAL}: padded_shared<[512:+16] {offset = [[0, 1], [0, 2], [0, 4], [0, 8], [0, 16], [0, 32], [4, 0], [8, 0], [16, 0], [1, 0], [2, 0]]
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1151-1152
```mlir
1151| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
1152|   // ASYNC-LABEL: loop_padding_block_size_small
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 1153-1170
```mlir
1153|   tt.func public @loop_padding_block_size_small(%arg0: i32, %arg1: tensor<32x64x!tt.ptr<f16>, #blocked> {tt.constancy = dense<1> : tensor<2xi32>, tt.contiguity = dense<[1, 8]> : tensor<2xi32>, tt.divisibility = dense<[1, 16]> : tensor<2xi32>}, %arg2: tensor<32x64x!tt.ptr<f16>, #mma>) {
1154|     %c1_i32 = arith.constant 1 : i32
1155|     %c0_i32 = arith.constant 0 : i32
1156|     %cst = arith.constant dense<0.000000e+00> : tensor<32x64xf16, #mma>
1157|     %cst_0 = arith.constant dense<0.000000e+00> : tensor<64x64xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>
1158|     %0 = scf.for %arg3 = %c0_i32 to %arg0 step %c1_i32 iter_args(%arg4 = %cst) -> (tensor<32x64xf16, #mma>)  : i32 {
1159|       %1 = tt.load %arg1 : tensor<32x64x!tt.ptr<f16>, #blocked>
1160|       %2 = ttg.convert_layout %1 : tensor<32x64xf16, #blocked> -> tensor<32x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>
1161|       %3 = tt.dot %2, %cst_0, %arg4 : tensor<32x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>> * tensor<64x64xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>> -> tensor<32x64xf16, #mma>
1162|       scf.yield %3 : tensor<32x64xf16, #mma>
1163|     }
1164|     tt.store %arg2, %0 : tensor<32x64x!tt.ptr<f16>, #mma>
1165|     tt.return
1166|   }
1167| }
1168| 
1169| 
1170| // End of negative tests for padding on gfx950
```
**EN:** This function-oriented block defines or enters `loop_padding_block_size_small`. Within it, the test exercises constants, tt.func, structured loops, masked or vectorized loads, layout conversions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `loop_padding_block_size_small` 为核心。测试在其中演示 常量、tt.func、结构化循环、带掩码或向量化的加载、布局转换，这些操作构成了被优化、lower 或验证的主要数据流。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-tritonamdgpu-schedule-loops=num_stages=2`, `-tritonamdgpu-pipeline`, `-canonicalize`, `-tritonamdgpu-schedule-loops="num_stages=2"`, `-tritonamdgpu-pipeline="use_async_copy=1"`
- **CN:** 主要 pass 选项：`-split-input-file`，`-tritonamdgpu-schedule-loops=num_stages=2`，`-tritonamdgpu-pipeline`，`-canonicalize`，`-tritonamdgpu-schedule-loops="num_stages=2"`，`-tritonamdgpu-pipeline="use_async_copy=1"`
- **EN:** Dominant operations include `arith.constant`, `tt.splat`, `tt.addptr`, `tt.broadcast`, `ttg.convert_layout`, `arith.addi`, `tt.func`, `tt.make_range`, `tt.expand_dims`, `tt.load`.
- **CN:** 主要操作包括 `arith.constant`、`tt.splat`、`tt.addptr`、`tt.broadcast`、`ttg.convert_layout`、`arith.addi`、`tt.func`、`tt.make_range`、`tt.expand_dims`、`tt.load`。
- **EN:** The file contains 19 independently testable section(s). The file relies mainly on the transformed IR itself rather than explicit FileCheck/diagnostic annotations. It validates the semantics or lowering pattern expressed by the IR in this test.
- **CN:** 该文件包含 19 个可独立测试的分段。该文件主要依赖变换后的 IR 本身，而不是显式的 FileCheck/诊断注解。 它验证该测试中 IR 所表达的语义或 lower 模式。

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
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。