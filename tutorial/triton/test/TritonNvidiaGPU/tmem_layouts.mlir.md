# tmem_layouts.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonNvidiaGPU/tmem_layouts.mlir`
- **EN:** Test for the MLIR/Triton pipeline `--triton-nvidia-optimize-tmem-layouts` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `--triton-nvidia-optimize-tmem-layouts` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file --triton-nvidia-optimize-tmem-layouts --allow-unregistered-dialect | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file --triton-nvidia-optimize-tmem-layouts --allow-unregistered-dialect | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt %s -split-input-file --triton-nvidia-optimize-tmem-layouts --allow-unregistered-dialect | FileCheck %s
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file --triton-nvidia-optimize-tmem-layouts --allow-unregistered-dialect | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file --triton-nvidia-optimize-tmem-layouts --allow-unregistered-dialect | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-9
```mlir
3| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 2], order = [1, 0]}>
4| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1, 2], threadsPerWarp = [1, 32, 1], warpsPerCTA = [4, 2, 1], order = [2, 1, 0]}>
5| #linear = #ttg.linear<{register = [[0, 1], [0, 2], [0, 4], [0, 8], [0, 16], [0, 32]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 64]], warp = [[32, 0], [64, 0], [16, 0]], block = []}>
6| #linear1 = #ttg.linear<{register = [[0, 0, 1], [0, 0, 2], [0, 0, 4], [0, 0, 8], [0, 0, 16], [0, 0, 32]], lane = [[1, 0, 0], [2, 0, 0], [4, 0, 0], [8, 0, 0], [0, 1, 0]], warp = [[32, 0, 0], [64, 0, 0], [16, 0, 0]], block = []}>
7| #linear2 = #ttg.linear<{register = [[0, 1, 0], [0, 2, 0], [0, 4, 0], [0, 8, 0], [0, 16, 0], [0, 32, 0]], lane = [[1, 0, 0], [2, 0, 0], [4, 0, 0], [8, 0, 0], [0, 0, 1]], warp = [[32, 0, 0], [64, 0, 0], [16, 0, 0]], block = []}>
8| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
9| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 10-10
```mlir
10| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "cuda:100"} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 11-11
```mlir
11|   // CHECK-LABEL: @subtile_tmem_load
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @subtile_tmem_load anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @subtile_tmem_load 这样的标签用于锚定匹配范围。

### Lines 12-12
```mlir
12|   tt.func public @subtile_tmem_load(%arg0: !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>) -> (tensor<128x64xf32, #blocked>, tensor<128x64xf32, #blocked>) {
```
**EN:** This function-oriented block defines or enters `subtile_tmem_load`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `subtile_tmem_load` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 13-28
```mlir
13|     // CHECK: %[[S0:.+]] = ttng.tmem_subslice %{{.+}} {N = 0 : i32}
14|     // CHECK: %[[L0:.+]] = ttng.tmem_load %[[S0]] : !ttg.memdesc<128x64xf32
15|     // CHECK: %[[C0:.+]] = ttg.convert_layout %[[L0]]
16|     // CHECK: %[[S1:.+]] = ttng.tmem_subslice %{{.+}} {N = 64 : i32}
17|     // CHECK: %[[L1:.+]] = ttng.tmem_load %[[S1]] : !ttg.memdesc<128x64xf32
18|     // CHECK: %[[C1:.+]] = ttg.convert_layout %[[L1]]
19|     // CHECK: tt.return %[[C0]], %[[C1]]
20|     %0 = ttng.tmem_load %arg0 : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #linear>
21|     %1 = tt.reshape %0 : tensor<128x128xf32, #linear> -> tensor<128x2x64xf32, #linear1>
22|     %2 = tt.trans %1 {order = array<i32: 0, 2, 1>} : tensor<128x2x64xf32, #linear1> -> tensor<128x64x2xf32, #linear2>
23|     %3 = ttg.convert_layout %2 : tensor<128x64x2xf32, #linear2> -> tensor<128x64x2xf32, #blocked1>
24|     %outLHS, %outRHS = tt.split %3 : tensor<128x64x2xf32, #blocked1> -> tensor<128x64xf32, #blocked>
25|     tt.return %outLHS, %outRHS : tensor<128x64xf32, #blocked>, tensor<128x64xf32, #blocked>
26|   }
27| }
28| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_load, tensor reshaping, transpose-like layout changes, layout conversions, tt.split. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_load、张量重塑、转置类布局变换、布局转换、tt.split。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 29-29
```mlir
29| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 30-39
```mlir
30| 
31| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 2], order = [1, 0]}>
32| #blocked1 = #ttg.blocked<{sizePerThread = [1, 2, 1], threadsPerWarp = [1, 1, 32], warpsPerCTA = [4, 1, 2], order = [1, 2, 0]}>
33| #blocked2 = #ttg.blocked<{sizePerThread = [1, 1, 2], threadsPerWarp = [1, 32, 1], warpsPerCTA = [4, 2, 1], order = [2, 1, 0]}>
34| #linear = #ttg.linear<{register = [[0, 1], [0, 2], [0, 4], [0, 8], [0, 16], [0, 32], [0, 64]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 128]], warp = [[32, 0], [64, 0], [16, 0]], block = []}>
35| #linear1 = #ttg.linear<{register = [[0, 0, 1], [0, 0, 2], [0, 0, 4], [0, 0, 8], [0, 0, 16], [0, 0, 32], [0, 0, 64]], lane = [[1, 0, 0], [2, 0, 0], [4, 0, 0], [8, 0, 0], [0, 1, 0]], warp = [[32, 0, 0], [64, 0, 0], [16, 0, 0]], block = []}>
36| #linear2 = #ttg.linear<{register = [[0, 1, 0], [0, 2, 0], [0, 4, 0], [0, 8, 0], [0, 16, 0], [0, 32, 0], [0, 64, 0]], lane = [[1, 0, 0], [2, 0, 0], [4, 0, 0], [8, 0, 0], [0, 0, 1]], warp = [[32, 0, 0], [64, 0, 0], [16, 0, 0]], block = []}>
37| #linear3 = #ttg.linear<{register = [[0, 0, 1], [0, 64, 0], [4, 0, 0], [8, 0, 0], [16, 0, 0], [32, 0, 0], [64, 0, 0]], lane = [[0, 1, 0], [0, 2, 0], [0, 4, 0], [0, 8, 0], [0, 16, 0]], warp = [[0, 32, 0], [1, 0, 0], [2, 0, 0]], block = []}>
38| #linear4 = #ttg.linear<{register = [[0, 64], [4, 0], [8, 0], [16, 0], [32, 0], [64, 0]], lane = [[0, 1], [0, 2], [0, 4], [0, 8], [0, 16]], warp = [[0, 32], [1, 0], [2, 0]], block = []}>
39| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 256, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 40-40
```mlir
40| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "cuda:100"} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 41-41
```mlir
41|   // CHECK-LABEL: @subtile4_tmem_load
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @subtile4_tmem_load anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @subtile4_tmem_load 这样的标签用于锚定匹配范围。

### Lines 42-42
```mlir
42|   tt.func public @subtile4_tmem_load(%arg0: !ttg.memdesc<128x256xf32, #tmem, #ttng.tensor_memory, mutable>) -> (tensor<128x64xf32, #blocked>, tensor<128x64xf32, #blocked>, tensor<128x64xf32, #blocked>, tensor<128x64xf32, #blocked>) {
```
**EN:** This function-oriented block defines or enters `subtile4_tmem_load`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `subtile4_tmem_load` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 43-60
```mlir
43|     // CHECK: %[[S0:.+]] = ttng.tmem_subslice %{{.+}} {N = 0 : i32}
44|     // CHECK: %[[S1:.+]] = ttng.tmem_subslice %[[S0]] {N = 0 : i32}
45|     // CHECK: %[[L1:.+]] = ttng.tmem_load %[[S1]] : !ttg.memdesc<128x64xf32
46|     // CHECK: %[[C1:.+]] = ttg.convert_layout %[[L1]]
47|     // CHECK: %[[S2:.+]] = ttng.tmem_subslice %[[S0]] {N = 64 : i32}
48|     // CHECK: %[[L2:.+]] = ttng.tmem_load %[[S2]] : !ttg.memdesc<128x64xf32
49|     // CHECK: %[[C2:.+]] = ttg.convert_layout %[[L2]]
50|     // CHECK: %[[S3:.+]] = ttng.tmem_subslice %{{.+}} {N = 128 : i32}
51|     // CHECK: %[[S4:.+]] = ttng.tmem_subslice %[[S3]] {N = 0 : i32}
52|     // CHECK: %[[L4:.+]] = ttng.tmem_load %[[S4]] : !ttg.memdesc<128x64xf32
53|     // CHECK: %[[C4:.+]] = ttg.convert_layout %[[L4]]
54|     // CHECK: %[[S5:.+]] = ttng.tmem_subslice %[[S3]] {N = 64 : i32}
55|     // CHECK: %[[L5:.+]] = ttng.tmem_load %[[S5]] : !ttg.memdesc<128x64xf32
56|     // CHECK: %[[C5:.+]] = ttg.convert_layout %[[L5]]
57|     // CHECK: tt.return %[[C1]], %[[C2]], %[[C4]], %[[C5]]
58|     %result = ttng.tmem_load %arg0 : !ttg.memdesc<128x256xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x256xf32, #linear>
59|     %0 = tt.reshape %result : tensor<128x256xf32, #linear> -> tensor<128x2x128xf32, #linear1>
60|     %1 = tt.trans %0 {order = array<i32: 0, 2, 1>} : tensor<128x2x128xf32, #linear1> -> tensor<128x128x2xf32, #linear2>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_load, tensor reshaping, transpose-like layout changes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_load、张量重塑、转置类布局变换。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 61-72
```mlir
61|     %2 = ttg.convert_layout %1 : tensor<128x128x2xf32, #linear2> -> tensor<128x128x2xf32, #linear3>
62|     %outLHS, %outRHS = tt.split %2 : tensor<128x128x2xf32, #linear3> -> tensor<128x128xf32, #linear4>
63|     %3 = tt.reshape %outLHS : tensor<128x128xf32, #linear4> -> tensor<128x2x64xf32, #blocked1>
64|     %4 = tt.trans %3 {order = array<i32: 0, 2, 1>} : tensor<128x2x64xf32, #blocked1> -> tensor<128x64x2xf32, #blocked2>
65|     %outLHS_0, %outRHS_1 = tt.split %4 : tensor<128x64x2xf32, #blocked2> -> tensor<128x64xf32, #blocked>
66|     %5 = tt.reshape %outRHS : tensor<128x128xf32, #linear4> -> tensor<128x2x64xf32, #blocked1>
67|     %6 = tt.trans %5 {order = array<i32: 0, 2, 1>} : tensor<128x2x64xf32, #blocked1> -> tensor<128x64x2xf32, #blocked2>
68|     %outLHS_2, %outRHS_3 = tt.split %6 : tensor<128x64x2xf32, #blocked2> -> tensor<128x64xf32, #blocked>
69|     tt.return %outLHS_0, %outRHS_1, %outLHS_2, %outRHS_3 : tensor<128x64xf32, #blocked>, tensor<128x64xf32, #blocked>, tensor<128x64xf32, #blocked>, tensor<128x64xf32, #blocked>
70|   }
71| }
72| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.split, tensor reshaping, transpose-like layout changes, layout conversions, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.split、张量重塑、转置类布局变换、布局转换、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 73-73
```mlir
73| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 74-82
```mlir
74| 
75| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
76| #blocked5 = #ttg.blocked<{sizePerThread = [1, 64], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
77| #blocked6 = #ttg.blocked<{sizePerThread = [1, 64, 2], threadsPerWarp = [32, 1, 1], warpsPerCTA = [4, 1, 1], order = [2, 0, 1]}>
78| #blocked7 = #ttg.blocked<{sizePerThread = [1, 2, 64], threadsPerWarp = [32, 1, 1], warpsPerCTA = [4, 1, 1], order = [1, 0, 2]}>
79| #linear = #ttg.linear<{register = [[0, 64], [0, 1], [0, 2], [0, 4], [0, 8], [0, 16], [0, 32]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0]], warp = [[32, 0], [64, 0]], block = []}>
80| 
81| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
82| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 83-84
```mlir
83| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100"} {
84| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 85-85
```mlir
85|   // CHECK-LABEL: @subtile_tmem_store
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @subtile_tmem_store anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @subtile_tmem_store 这样的标签用于锚定匹配范围。

### Lines 86-90
```mlir
86|   tt.func public @subtile_tmem_store(
87|     %arg0: !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>,
88|     %arg1: tensor<128x64xf32, #blocked5>,
89|     %arg2: tensor<128x64xf32, #blocked5>
90|   ) {
```
**EN:** This function-oriented block defines or enters `subtile_tmem_store`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `subtile_tmem_store` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 91-106
```mlir
 91|     // CHECK: [[S0:%.+]] = ttng.tmem_subslice %arg0 {N = 0 : i32}
 92|     // CHECK: [[V0:%.+]] = ttg.convert_layout %arg1
 93|     // CHECK: ttng.tmem_store [[V0]], [[S0]]
 94|     // CHECK: [[S1:%.+]] = ttng.tmem_subslice %arg0 {N = 64 : i32}
 95|     // CHECK: [[V1:%.+]] = ttg.convert_layout %arg2
 96|     // CHECK: ttng.tmem_store [[V1]], [[S1]]
 97|     %true = arith.constant true
 98|     %joined = tt.join %arg1, %arg2 : tensor<128x64xf32, #blocked5> -> tensor<128x64x2xf32, #blocked6>
 99|     %trans = tt.trans %joined {order = array<i32: 0, 2, 1>} : tensor<128x64x2xf32, #blocked6> -> tensor<128x2x64xf32, #blocked7>
100|     %reshaped = tt.reshape %trans : tensor<128x2x64xf32, #blocked7> -> tensor<128x128xf32, #linear>
101|     %cvt = ttg.convert_layout %reshaped : tensor<128x128xf32, #linear> -> tensor<128x128xf32, #blocked>
102|     ttng.tmem_store %cvt, %arg0, %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
103|     tt.return
104|   }
105| }
106| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants, tt.join, transpose-like layout changes, tensor reshaping, layout conversions. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量、tt.join、转置类布局变换、张量重塑、布局转换。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 107-107
```mlir
107| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 108-115
```mlir
108| 
109| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 2], order = [1, 0]}>
110| #linear = #ttg.linear<{register = [[0, 1], [0, 2], [0, 4], [0, 8], [0, 16], [0, 32], [0, 64]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0]], warp = [[32, 0], [64, 0], [128, 0]], block = []}>
111| #blocked2 = #ttg.blocked<{sizePerThread = [1, 2, 64], threadsPerWarp = [32, 1, 1], warpsPerCTA = [8, 1, 1], order = [0, 2, 1]}>
112| #blocked3 = #ttg.blocked<{sizePerThread = [1, 64, 2], threadsPerWarp = [32, 1, 1], warpsPerCTA = [8, 1, 1], order = [0, 1, 2]}>
113| #blocked4 = #ttg.blocked<{sizePerThread = [1, 1, 2], threadsPerWarp = [1, 32, 1], warpsPerCTA = [4, 2, 1], order = [2, 1, 0]}>
114| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
115| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 116-116
```mlir
116| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "cuda:100"} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 117-119
```mlir
117|   // CHECK-LABEL: @subtile_tmem_load_256
118|   // CHECK-NOT: ttng.tmem_subslice
119|   // CHECK: tt.return
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @subtile_tmem_load_256 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @subtile_tmem_load_256 这样的标签用于锚定匹配范围。

### Lines 120-129
```mlir
120|   tt.func public @subtile_tmem_load_256(%arg0: !ttg.memdesc<256x128xf32, #tmem, #ttng.tensor_memory, mutable>) -> (tensor<256x64xf32, #blocked>, tensor<256x64xf32, #blocked>) {
121|     %0 = ttng.tmem_load %arg0 : !ttg.memdesc<256x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<256x128xf32, #linear>
122|     %1 = tt.reshape %0 : tensor<256x128xf32, #linear> -> tensor<256x2x64xf32, #blocked2>
123|     %2 = tt.trans %1 {order = array<i32: 0, 2, 1>} : tensor<256x2x64xf32, #blocked2> -> tensor<256x64x2xf32, #blocked3>
124|     %3 = ttg.convert_layout %2 : tensor<256x64x2xf32, #blocked3> -> tensor<256x64x2xf32, #blocked4>
125|     %outLHS, %outRHS = tt.split %3 : tensor<256x64x2xf32, #blocked4> -> tensor<256x64xf32, #blocked>
126|     tt.return %outLHS, %outRHS : tensor<256x64xf32, #blocked>, tensor<256x64xf32, #blocked>
127|   }
128| }
129| 
```
**EN:** This function-oriented block defines or enters `subtile_tmem_load_256`. Within it, the test exercises tt.func, ttng.tmem_load, tensor reshaping, transpose-like layout changes, layout conversions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `subtile_tmem_load_256` 为核心。测试在其中演示 tt.func、ttng.tmem_load、张量重塑、转置类布局变换、布局转换，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 130-130
```mlir
130| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 131-134
```mlir
131| 
132| #linear = #ttg.linear<{register = [[0, 1], [0, 2], [0, 4], [0, 8], [0, 16]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0]], warp = [[32, 0], [64, 0], [0, 32]], block = []}>
133| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 64, colStride = 1>
134| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 135-136
```mlir
135| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
136| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 137-138
```mlir
137| // CHECK{LITERAL}: #linear1 = #ttg.linear<{register = [[0, 1], [0, 2], [0, 4], [0, 8], [0, 16]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 32]], warp = [[32, 0], [64, 0], [16, 0]], block = []}>
138| // CHECK-LABEL: tmem_load_reduce
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tmem_load_reduce anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tmem_load_reduce 这样的标签用于锚定匹配范围。

### Lines 139-140
```mlir
139| tt.func public @tmem_load_reduce(%arg0: !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory>) -> tensor<128xf32, #ttg.slice<{dim = 1, parent = #linear}>> {
140|   %0 = ttng.tmem_load %arg0 : !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory> -> tensor<128x64xf32, #linear>
```
**EN:** This function-oriented block defines or enters `tmem_load_reduce`. Within it, the test exercises tt.func, ttng.tmem_load, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tmem_load_reduce` 为核心。测试在其中演示 tt.func、ttng.tmem_load，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 141-151
```mlir
141|   // CHECK: ttng.tmem_load %{{.*}} : !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory> -> tensor<128x64xf32, #linear1>
142|   %1 = "tt.reduce"(%0) <{axis = 1 : i32}> ({
143|   ^bb0(%arg2: f32, %arg3: f32):
144|     %2 = arith.addf %arg2, %arg3 : f32
145|     tt.reduce.return %2 : f32
146|   }) : (tensor<128x64xf32, #linear>) -> tensor<128xf32, #ttg.slice<{dim = 1, parent = #linear}>>
147|   tt.return %1 : tensor<128xf32, #ttg.slice<{dim = 1, parent = #linear}>>
148| }
149| 
150| }
151| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on floating-point additions, reductions, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 浮点加法、归约、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 152-152
```mlir
152| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 153-158
```mlir
153| 
154| #blocked = #ttg.blocked<{sizePerThread = [64, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
155| #blocked1 = #ttg.blocked<{sizePerThread = [1, 64], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
156| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
157| #smem = #ttg.shared_memory
158| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 64, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 159-159
```mlir
159| // CHECK{LITERAL}: #linear = #ttg.linear<{register = [[0, 1], [8, 0], [0, 8], [0, 16], [0, 32], [16, 0]], lane = [[0, 2], [0, 4], [1, 0], [2, 0], [4, 0]], warp = [[32, 0], [64, 0]], block = []}>
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 160-160
```mlir
160| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 161-161
```mlir
161|   // CHECK-LABLE: test_tmem_store_dist_layout
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 162-168
```mlir
162|   tt.func public @test_tmem_store_dist_layout(%arg0: f32, %arg1: !ttg.memdesc<64x128xf16, #shared, #smem, mutable>, %arg2: !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory, mutable>) {
163|     %true = arith.constant true
164|     %0 = tt.splat %arg0 : f32 -> tensor<64x128xf32, #blocked>
165|     %1 = ttg.local_load %arg1 : !ttg.memdesc<64x128xf16, #shared, #smem, mutable> -> tensor<64x128xf16, #blocked>
166|     %2 = arith.extf %1 : tensor<64x128xf16, #blocked> to tensor<64x128xf32, #blocked>
167|     %3 = arith.mulf %2, %0 : tensor<64x128xf32, #blocked>
168|     %4 = tt.trans %3 {order = array<i32: 1, 0>} : tensor<64x128xf32, #blocked> -> tensor<128x64xf32, #blocked1>
```
**EN:** This function-oriented block defines or enters `test_tmem_store_dist_layout`. Within it, the test exercises tt.func, constants, broadcasted scalars or pointers, local/shared memory loads, arith.extf, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `test_tmem_store_dist_layout` 为核心。测试在其中演示 tt.func、常量、广播后的标量或指针、本地/共享内存加载、arith.extf，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 169-175
```mlir
169|     // CHECK: %[[C:.+]] = ttg.convert_layout %{{.+}} : tensor<128x64xf32, #{{.+}}> -> tensor<128x64xf32, #linear>
170|     // CHECK: ttng.tmem_store %[[C]], %{{.+}}, %{{.+}} : tensor<128x64xf32, #linear> -> !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory, mutable>
171|     ttng.tmem_store %4, %arg2, %true : tensor<128x64xf32, #blocked1> -> !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory, mutable>
172|     tt.return
173|   }
174| }
175| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_store, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_store、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 176-176
```mlir
176| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 177-183
```mlir
177| 
178| #blocked = #ttg.blocked<{sizePerThread = [64, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
179| #blocked1 = #ttg.blocked<{sizePerThread = [1, 64], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
180| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
181| #smem = #ttg.shared_memory
182| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 64, colStride = 1>
183| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 184-184
```mlir
184| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 185-185
```mlir
185|   // CHECK-LABLE: test_tmem_store_dist_layout_negative
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 186-189
```mlir
186|   tt.func public @test_tmem_store_dist_layout_negative(%arg0: f32, %arg1: !ttg.memdesc<128x64xf16, #shared, #smem, mutable>, %arg2: !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory, mutable>) {
187|     %true = arith.constant true
188|     %1 = ttg.local_load %arg1 : !ttg.memdesc<128x64xf16, #shared, #smem, mutable> -> tensor<128x64xf16, #blocked1>
189|     %2 = arith.extf %1 : tensor<128x64xf16, #blocked1> to tensor<128x64xf32, #blocked1>
```
**EN:** This function-oriented block defines or enters `test_tmem_store_dist_layout_negative`. Within it, the test exercises tt.func, constants, local/shared memory loads, arith.extf, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `test_tmem_store_dist_layout_negative` 为核心。测试在其中演示 tt.func、常量、本地/共享内存加载、arith.extf，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 190-196
```mlir
190|     // CHECK: %[[C:.+]] = arith.extf
191|     // CHECK: ttng.tmem_store %[[C]]
192|     ttng.tmem_store %2, %arg2, %true : tensor<128x64xf32, #blocked1> -> !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory, mutable>
193|     tt.return
194|   }
195| }
196| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_store, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_store、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 197-197
```mlir
197| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 198-204
```mlir
198| 
199| #blocked = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
200| #blocked1 = #ttg.blocked<{sizePerThread = [16, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
201| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 8}>
202| #smem = #ttg.shared_memory
203| #linear = #ttg.linear<{register = [[0, 1], [0, 2], [0, 4], [0, 8], [128, 0]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0]], warp = [[32, 0], [64, 0]], block = []}>
204| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 16, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 205-205
```mlir
205| // CHECK{LITERAL}: #linear = #ttg.linear<{register = [[0, 1], [8, 0], [0, 8], [128, 0], [16, 0]], lane = [[0, 2], [0, 4], [1, 0], [2, 0], [4, 0]], warp = [[32, 0], [64, 0]], block = []}>
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 206-206
```mlir
206| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 207-207
```mlir
207|   tt.func @reshape_memedesc_negative(%arg0: !ttg.memdesc<256x16xf32, #tmem, #ttng.tensor_memory>, %arg1: !ttg.memdesc<16x256xf8E4M3FN, #shared, #smem, mutable>) {
```
**EN:** This function-oriented block defines or enters `reshape_memedesc_negative`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `reshape_memedesc_negative` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 208-217
```mlir
208|     // CHECK: %[[L:.+]] = ttng.tmem_load %{{.+}} : !ttg.memdesc<256x16xf32, #tmem, #ttng.tensor_memory> -> tensor<256x16xf32, #linear>
209|     // CHECK: ttg.convert_layout %[[L:.+]]
210|     %result = ttng.tmem_load %arg0 : !ttg.memdesc<256x16xf32, #tmem, #ttng.tensor_memory> -> tensor<256x16xf32, #linear>
211|     %0 = tt.trans %result {order = array<i32: 1, 0>} : tensor<256x16xf32, #linear> -> tensor<16x256xf32, #blocked1>
212|     %1 = tt.fp_to_fp %0, rounding = rtne : tensor<16x256xf32, #blocked1> -> tensor<16x256xf8E4M3FN, #blocked1>
213|     ttg.local_store %1, %arg1 : tensor<16x256xf8E4M3FN, #blocked1> -> !ttg.memdesc<16x256xf8E4M3FN, #shared, #smem, mutable>
214|     tt.return
215|   }
216| }
217| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_load, transpose-like layout changes, tt.fp_to_fp, local/shared memory stores, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_load、转置类布局变换、tt.fp_to_fp、本地/共享内存存储、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 218-218
```mlir
218| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 219-222
```mlir
219| 
220| #linear = #ttg.linear<{register = [[0, 1], [0, 2], [0, 4], [0, 8], [0, 16]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0]], warp = [[32, 0], [64, 0], [0, 32]], block = []}>
221| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 64, colStride = 1>
222| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 223-223
```mlir
223| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "cuda:100"} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 224-227
```mlir
224|   // CHECK-LABEL: @tmem_load_reduce_rank1
225|   // CHECK: "tt.reduce"
226|   // CHECK: "tt.reduce"
227|   // CHECK: tt.return
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tmem_load_reduce_rank1 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tmem_load_reduce_rank1 这样的标签用于锚定匹配范围。

### Lines 228-242
```mlir
228|   tt.func public @tmem_load_reduce_rank1(%arg0: !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory>) -> f32 {
229|     %0 = ttng.tmem_load %arg0 : !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory> -> tensor<128x64xf32, #linear>
230|     %1 = "tt.reduce"(%0) <{axis = 1 : i32}> ({
231|     ^bb0(%lhs: f32, %rhs: f32):
232|       %2 = arith.addf %lhs, %rhs : f32
233|       tt.reduce.return %2 : f32
234|     }) : (tensor<128x64xf32, #linear>) -> tensor<128xf32, #ttg.slice<{dim = 1, parent = #linear}>>
235|     %3 = "tt.reduce"(%1) <{axis = 0 : i32}> ({
236|     ^bb0(%lhs: f32, %rhs: f32):
237|       %4 = arith.addf %lhs, %rhs : f32
238|       tt.reduce.return %4 : f32
239|     }) : (tensor<128xf32, #ttg.slice<{dim = 1, parent = #linear}>>) -> f32
240|     tt.return %3 : f32
241|   }
242| }
```
**EN:** This function-oriented block defines or enters `tmem_load_reduce_rank1`. Within it, the test exercises tt.func, floating-point additions, reductions, ttng.tmem_load, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tmem_load_reduce_rank1` 为核心。测试在其中演示 tt.func、浮点加法、归约、ttng.tmem_load、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `--triton-nvidia-optimize-tmem-layouts`, `--allow-unregistered-dialect`
- **CN:** 主要 pass 选项：`-split-input-file`，`--triton-nvidia-optimize-tmem-layouts`，`--allow-unregistered-dialect`
- **EN:** Dominant operations include `tt.func`, `module`, `tt.return`, `tt.trans`, `ttng.tmem_load`, `tt.reshape`, `tt.split`, `ttg.convert_layout`, `arith.constant`, `ttng.tmem_store`.
- **CN:** 主要操作包括 `tt.func`、`module`、`tt.return`、`tt.trans`、`ttng.tmem_load`、`tt.reshape`、`tt.split`、`ttg.convert_layout`、`arith.constant`、`ttng.tmem_store`。
- **EN:** The file contains 8 independently testable section(s). Check styles used: CHECK x42, CHECK-LABEL x6, CHECK-LABLE x2, CHECK-NOT x1. Important labels include @subtile_tmem_load, @subtile4_tmem_load, @subtile_tmem_store, @subtile_tmem_load_256. It validates both the presence of the desired rewrite and the absence of unwanted residual operations.
- **CN:** 该文件包含 8 个可独立测试的分段。使用的检查类型：CHECK ×42，CHECK-LABEL ×6，CHECK-LABLE ×2，CHECK-NOT ×1。 关键标签包括 @subtile_tmem_load，@subtile4_tmem_load，@subtile_tmem_store，@subtile_tmem_load_256。 它同时验证目标重写已经出现，并且不再残留不需要的操作。

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
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。