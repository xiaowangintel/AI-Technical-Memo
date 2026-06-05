# amd-convert-buffer-ops.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/amd/amd-convert-buffer-ops.mlir`
- **EN:** Test for the MLIR/Triton pipeline `--tritonamdgpu-convert-buffer-ops="gfx-arch=gfx942, --check-prefixes=COMMON,GFX942-ONLY,CDNA, --tritonamdgpu-convert-buffer-ops="gfx-arch=gfx950, --check-prefixes=COMMON,GFX950-PLUS,CDNA` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `--tritonamdgpu-convert-buffer-ops="gfx-arch=gfx942, --check-prefixes=COMMON,GFX942-ONLY,CDNA, --tritonamdgpu-convert-buffer-ops="gfx-arch=gfx950, --check-prefixes=COMMON,GFX950-PLUS,CDNA` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file --tritonamdgpu-convert-buffer-ops="gfx-arch=gfx942 analyze-small-tensor-ofst=true"| FileCheck %s --check-prefixes=COMMON,GFX942-ONLY,CDNA`; `// RUN: triton-opt %s -split-input-file --tritonamdgpu-convert-buffer-ops="gfx-arch=gfx950 analyze-small-tensor-ofst=true"| FileCheck %s --check-prefixes=COMMON,GFX950-PLUS,CDNA`; `// RUN: triton-opt %s -split-input-file --tritonamdgpu-convert-buffer-ops="gfx-arch=gfx1250 analyze-small-tensor-ofst=true"| FileCheck %s --check-prefixes=COMMON,GFX950-PLUS`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file --tritonamdgpu-convert-buffer-ops="gfx-arch=gfx942 analyze-small-tensor-ofst=true"| FileCheck %s --check-prefixes=COMMON,GFX942-ONLY,CDNA`；`// RUN: triton-opt %s -split-input-file --tritonamdgpu-convert-buffer-ops="gfx-arch=gfx950 analyze-small-tensor-ofst=true"| FileCheck %s --check-prefixes=COMMON,GFX950-PLUS,CDNA`；`// RUN: triton-opt %s -split-input-file --tritonamdgpu-convert-buffer-ops="gfx-arch=gfx1250 analyze-small-tensor-ofst=true"| FileCheck %s --check-prefixes=COMMON,GFX950-PLUS`

## Line-by-Line Analysis / 逐行分析

### Lines 1-4
```mlir
1| // RUN: triton-opt %s -split-input-file --tritonamdgpu-convert-buffer-ops="gfx-arch=gfx942 analyze-small-tensor-ofst=true"| FileCheck %s --check-prefixes=COMMON,GFX942-ONLY,CDNA
2| // RUN: triton-opt %s -split-input-file --tritonamdgpu-convert-buffer-ops="gfx-arch=gfx950 analyze-small-tensor-ofst=true"| FileCheck %s --check-prefixes=COMMON,GFX950-PLUS,CDNA
3| // RUN: triton-opt %s -split-input-file --tritonamdgpu-convert-buffer-ops="gfx-arch=gfx1250 analyze-small-tensor-ofst=true"| FileCheck %s --check-prefixes=COMMON,GFX950-PLUS
4| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file --tritonamdgpu-convert-buffer-ops="gfx-arch=gfx942 analyze-small-tensor-ofst=true"| FileCheck %s --check-prefixes=COMMON,GFX942-ONLY,CDNA` ; ` triton-opt %s -split-input-file --tritonamdgpu-convert-buffer-ops="gfx-arch=gfx950 analyze-small-tensor-ofst=true"| FileCheck %s --check-prefixes=COMMON,GFX950-PLUS,CDNA` ; ` triton-opt %s -split-input-file --tritonamdgpu-convert-buffer-ops="gfx-arch=gfx1250 analyze-small-tensor-ofst=true"| FileCheck %s --check-prefixes=COMMON,GFX950-PLUS` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file --tritonamdgpu-convert-buffer-ops="gfx-arch=gfx942 analyze-small-tensor-ofst=true"| FileCheck %s --check-prefixes=COMMON,GFX942-ONLY,CDNA`；` triton-opt %s -split-input-file --tritonamdgpu-convert-buffer-ops="gfx-arch=gfx950 analyze-small-tensor-ofst=true"| FileCheck %s --check-prefixes=COMMON,GFX950-PLUS,CDNA`；` triton-opt %s -split-input-file --tritonamdgpu-convert-buffer-ops="gfx-arch=gfx1250 analyze-small-tensor-ofst=true"| FileCheck %s --check-prefixes=COMMON,GFX950-PLUS`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 5-5
```mlir
5| #blocked0 = #ttg.blocked<{sizePerThread = [8], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 6-7
```mlir
6| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
7|   // COMMON-LABEL: simple
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 8-25
```mlir
 8|     tt.func @simple(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32, tt.pointer_range = 32 :i32}, %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg2: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg3: i32) {
 9|     %c256_i32 = arith.constant 256 : i32
10|     %0 = tt.get_program_id x : i32
11|     %1 = arith.muli %0, %c256_i32 : i32
12|     %2 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #blocked0>
13|     %3 = tt.splat %1 : i32 -> tensor<256xi32, #blocked0>
14|     // COMMON: %[[offset:.*]] = arith.addi
15|     %4 = arith.addi %3, %2 : tensor<256xi32, #blocked0>
16|     %5 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<256x!tt.ptr<f32>, #blocked0>
17|     %6 = tt.addptr %5, %4 : tensor<256x!tt.ptr<f32>, #blocked0>, tensor<256xi32, #blocked0>
18|     %7 = tt.splat %arg1 : !tt.ptr<f32> -> tensor<256x!tt.ptr<f32>, #blocked0>
19|     %8 = tt.addptr %7, %4 : tensor<256x!tt.ptr<f32>, #blocked0>, tensor<256xi32, #blocked0>
20|     // Note: large-tensor with elemIdx=pid*256 + arange(0, 256), elemIdx ∈ [0, smax]
21|     // COMMON-NOT: buffer_load
22|     %9 = tt.load %6 : tensor<256x!tt.ptr<f32>, #blocked0>
23|     // COMMON-NOT: buffer_load
24|     %10 = tt.load %8 : tensor<256x!tt.ptr<f32>, #blocked0>
25|     // COMMON: %[[data:.*]] = arith.addf
```
**EN:** This function-oriented block defines or enters `simple`. Within it, the test exercises broadcasted scalars or pointers, tt.func, pointer arithmetic, masked or vectorized loads, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `simple` 为核心。测试在其中演示 广播后的标量或指针、tt.func、指针算术、带掩码或向量化的加载、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 26-35
```mlir
26|     %11 = arith.addf %9, %10 : tensor<256xf32, #blocked0>
27|     %12 = tt.splat %arg2 : !tt.ptr<f32> -> tensor<256x!tt.ptr<f32>, #blocked0>
28|     %13 = tt.addptr %12, %4 : tensor<256x!tt.ptr<f32>, #blocked0>, tensor<256xi32, #blocked0>
29|     // Note: large-tensor with elemIdx ∈ [0, smax]
30|     // COMMON-NOT: buffer_store
31|     tt.store %13, %11 : tensor<256x!tt.ptr<f32>, #blocked0>
32|     tt.return
33|   }
34| }
35| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on floating-point additions, broadcasted scalars or pointers, pointer arithmetic, masked or vectorized stores, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 浮点加法、广播后的标量或指针、指针算术、带掩码或向量化的存储、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 36-36
```mlir
36| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 37-38
```mlir
37| 
38| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [8, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 39-40
```mlir
39| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
40| // COMMON-LABEL: buffer_stride
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 41-58
```mlir
41|   tt.func public @buffer_stride(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg2: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg3: i32 {tt.divisibility = 16 : i32}, %arg4: i32 {tt.divisibility = 16 : i32}, %arg5: i32 {tt.divisibility = 16 : i32}, %arg6: i32 {tt.divisibility = 16 : i32}, %arg7: i32 {tt.divisibility = 16 : i32}, %arg8: i32 {tt.divisibility = 16 : i32}, %arg9: i32 {tt.divisibility = 16 : i32}) {
42|     %c48_i32 = arith.constant 48 : i32
43|     %c32_i32 = arith.constant 32 : i32
44|     %c0_i32 = arith.constant 0 : i32
45|     %0 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
46|     %1 = tt.expand_dims %0 {axis = 1 : i32} : tensor<256xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<256x1xi32, #blocked>
47|     %cmp = arith.cmpi sgt, %arg6, %c0_i32 : i32
48|     llvm.intr.assume %cmp : i1
49|     %arg6_upper = arith.constant 4194304 : i32
50|     %cmp2 = arith.cmpi slt, %arg6, %arg6_upper : i32
51|     llvm.intr.assume %cmp2 : i1
52|     %2 = tt.splat %arg6 : i32 -> tensor<256x1xi32, #blocked>
53|     %3 = arith.muli %1, %2 : tensor<256x1xi32, #blocked>
54|     %4 = tt.addptr %arg0, %c32_i32 : !tt.ptr<f16>, i32
55|     %5 = tt.broadcast %3 : tensor<256x1xi32, #blocked> -> tensor<256x64xi32, #blocked>
56|     %6 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
57|     %7 = tt.expand_dims %6 {axis = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x64xi32, #blocked>
58|     %8 = tt.broadcast %7 : tensor<1x64xi32, #blocked> -> tensor<256x64xi32, #blocked>
```
**EN:** This function-oriented block defines or enters `buffer_stride`. Within it, the test exercises constants, tt.func, lane/block index ranges, shape expansion, integer comparisons, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `buffer_stride` 为核心。测试在其中演示 常量、tt.func、lane/block 索引范围、形状扩展、整数比较，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 59-76
```mlir
59|     %9 = arith.addi %8, %5 : tensor<256x64xi32, #blocked>
60|     %10 = tt.splat %4 : !tt.ptr<f16> -> tensor<256x64x!tt.ptr<f16>, #blocked>
61|     %11 = tt.addptr %10, %9 : tensor<256x64x!tt.ptr<f16>, #blocked>, tensor<256x64xi32, #blocked>
62| 
63|     // COMMON: %[[splat:.*]] = tt.splat %arg[[#stride:]]
64|     // COMMON: %[[mul:.*]] = arith.muli %[[#]], %[[splat]]
65|     // COMMON: %[[ptr:.*]] = tt.addptr %arg0
66|     // COMMON: %[[bcast1:.*]] = tt.broadcast %[[mul]]
67|     // COMMON: %[[bcast0:.*]] = tt.broadcast %[[#]]
68|     // COMMON: %[[offset:.*]] = arith.addi %[[bcast0]], %[[bcast1]]
69|     // COMMON: %[[buffer:.*]] = amdg.buffer_load %[[ptr]][%[[offset]]] stride = %arg[[#stride]]
70| 
71|     %12 = tt.load %11 : tensor<256x64x!tt.ptr<f16>, #blocked>
72|     %13 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
73|     %14 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
74|     %15 = tt.expand_dims %13 {axis = 1 : i32} : tensor<256xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<256x1xi32, #blocked>
75|     %16 = tt.expand_dims %14 {axis = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x64xi32, #blocked>
76|     %cmp1 = arith.cmpi sgt, %arg8, %c0_i32 : i32
```
**EN:** This operational block carries the concrete IR steps for the test, centered on lane/block index ranges, shape expansion, integer additions, broadcasted scalars or pointers, pointer arithmetic. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 lane/block 索引范围、形状扩展、整数加法、广播后的标量或指针、指针算术。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 77-94
```mlir
77|     llvm.intr.assume %cmp1 : i1
78|     %17 = tt.splat %arg8 : i32 -> tensor<256x1xi32, #blocked>
79|     %18 = arith.muli %17, %15 : tensor<256x1xi32, #blocked>
80|     %19 = tt.addptr %arg2, %c48_i32 : !tt.ptr<f16>, i32
81|     %20 = tt.broadcast %18 : tensor<256x1xi32, #blocked> -> tensor<256x64xi32, #blocked>
82|     %21 = tt.broadcast %16 : tensor<1x64xi32, #blocked> -> tensor<256x64xi32, #blocked>
83|     %22 = tt.addptr %19, %c48_i32 : !tt.ptr<f16>, i32
84|     %23 = arith.addi %21, %20 : tensor<256x64xi32, #blocked>
85|     %24 = tt.splat %22 : !tt.ptr<f16> -> tensor<256x64x!tt.ptr<f16>, #blocked>
86|     %25 = tt.addptr %24, %23 : tensor<256x64x!tt.ptr<f16>, #blocked>, tensor<256x64xi32, #blocked>
87|     %ofst_upper = arith.constant 1073741823 : i32
88|     %cmp3 = arith.cmpi slt, %ofst_upper, %ofst_upper : i32
89|     llvm.intr.assume %cmp3 : i1
90| 
91|     // COMMON: %[[splatb:.*]] = tt.splat %arg[[#strideb:]]
92|     // COMMON: %[[mulb:.*]] = arith.muli %[[splatb]], %[[#]]
93|     // COMMON: %[[bcast1b:.*]] = tt.broadcast %[[mulb]]
94|     // COMMON: %[[bcast0b:.*]] = tt.broadcast %[[#]]
```
**EN:** This operational block carries the concrete IR steps for the test, centered on pointer arithmetic, llvm.intr, broadcasted scalars or pointers, tensor broadcasting, integer multiplications. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 指针算术、llvm.intr、广播后的标量或指针、张量广播、整数乘法。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 95-104
```mlir
 95|     // COMMON: %[[ptrb:.*]] = tt.addptr
 96|     // COMMON: %[[offsetb:.*]] = arith.addi %[[bcast0b]], %[[bcast1b]]
 97|     // COMMON-NOT: buffer_store
 98| 
 99|     tt.store %25, %12 : tensor<256x64x!tt.ptr<f16>, #blocked>
100|     tt.return
101|   }
102| }
103| 
104| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized stores, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的存储、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 105-105
```mlir
105| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 106-107
```mlir
106| 
107| #blocked = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 108-109
```mlir
108| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
109|   // COMMON-LABEL: assume_positive_offset
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 110-127
```mlir
110|   tt.func @assume_positive_offset(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}) ->  tensor<1024xf32, #blocked>{
111|     %c1024_i32 = arith.constant 1024 : i32
112|     %c128_i32 = arith.constant 128 : i32
113|     %c0_i32 = arith.constant 0 : i32
114|     %0 = tt.get_program_id x : i32
115|     %1 = arith.muli %0, %c1024_i32 : i32
116|     %sub = arith.subi %1, %c128_i32 : i32
117|     %cmp = arith.cmpi sgt, %sub, %c0_i32 : i32
118|     llvm.intr.assume %cmp : i1
119|     %2 = tt.splat %sub : i32 -> tensor<1024xi32, #blocked>
120|     %3 = tt.make_range {end = 1024 : i32, start = 0 : i32} : tensor<1024xi32, #blocked>
121|     // COMMON: %[[offset:.*]] = arith.addi
122|     %4 = arith.addi %2, %3 : tensor<1024xi32, #blocked>
123|     // COMMON: %[[scalar_ptr:.*]] = tt.addptr %arg0
124|     %5 = tt.addptr %arg0, %1 : !tt.ptr<f32>, i32
125|     %8 = tt.splat %5 : !tt.ptr<f32> -> tensor<1024x!tt.ptr<f32>, #blocked>
126|     %9 = tt.addptr %8, %4 : tensor<1024x!tt.ptr<f32>, #blocked>, tensor<1024xi32, #blocked>
127|     // COMMON-NOT: buffer_load %[[scalar_ptr]][%[[offset]]]
```
**EN:** This function-oriented block defines or enters `assume_positive_offset`. Within it, the test exercises constants, tt.func, broadcasted scalars or pointers, pointer arithmetic, program IDs, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `assume_positive_offset` 为核心。测试在其中演示 常量、tt.func、广播后的标量或指针、指针算术、程序 ID，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 128-132
```mlir
128|     %10 = tt.load %9 : tensor<1024x!tt.ptr<f32>, #blocked>
129|     tt.return %10 : tensor<1024xf32, #blocked>
130|   }
131| }
132| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 133-133
```mlir
133| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 134-135
```mlir
134| 
135| #blocked = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 136-137
```mlir
136| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32}  {
137|   // COMMON-LABEL: offset_64_bits
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 138-155
```mlir
138|   tt.func @offset_64_bits(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}) -> tensor<1024xf32, #blocked> {
139|     %c1024_i32 = arith.constant 1024 : i32
140|     %c128_i32 = arith.constant 128 : i32
141|     %0 = tt.get_program_id x : i32
142|     %1 = arith.muli %0, %c1024_i32 : i32
143|     %sub = arith.subi %1, %c128_i32 : i32
144|     %2 = tt.splat %sub : i32 -> tensor<1024xi32, #blocked>
145|     %3 = tt.make_range {end = 1024 : i32, start = 0 : i32} : tensor<1024xi32, #blocked>
146|     %ext2 = arith.extsi %2 : tensor<1024xi32, #blocked> to tensor<1024xi64, #blocked>
147|     %ext3 = arith.extsi %3 : tensor<1024xi32, #blocked> to tensor<1024xi64, #blocked>
148|     %4 = arith.addi %ext2, %ext3 : tensor<1024xi64, #blocked>
149|     %5 = tt.addptr %arg0, %1 : !tt.ptr<f32>, i32
150|     %8 = tt.splat %5 : !tt.ptr<f32> -> tensor<1024x!tt.ptr<f32>, #blocked>
151|     %9 = tt.addptr %8, %4 : tensor<1024x!tt.ptr<f32>, #blocked>, tensor<1024xi64, #blocked>
152|     // COMMON: tt.load
153|     %10 = tt.load %9 : tensor<1024x!tt.ptr<f32>, #blocked>
154|     tt.return %10 : tensor<1024xf32, #blocked>
155|   }
```
**EN:** This function-oriented block defines or enters `offset_64_bits`. Within it, the test exercises tt.func, constants, broadcasted scalars or pointers, arith.extsi, pointer arithmetic, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `offset_64_bits` 为核心。测试在其中演示 tt.func、常量、广播后的标量或指针、arith.extsi、指针算术，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 156-157
```mlir
156| }
157| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 158-158
```mlir
158| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 159-160
```mlir
159| 
160| #blocked = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 161-162
```mlir
161| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32}  {
162|   // COMMON-LABEL: offset_64_bits_narrow
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 163-180
```mlir
163|   tt.func public @offset_64_bits_narrow(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg2: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg3: i32 {tt.divisibility = 16 : i32}) -> tensor<1024xf32, #blocked> {
164|     %c1024_i32 = arith.constant 1024 : i32
165|     %c128_i32 = arith.constant 128 : i32
166|     %0 = tt.get_program_id x : i32
167|     %1 = arith.muli %0, %c1024_i32 : i32
168|     %2 = tt.splat %1: i32 -> tensor<1024xi32, #blocked>
169|     %3 = tt.make_range {end = 1024 : i32, start = 0 : i32} : tensor<1024xi32, #blocked>
170|     %ext2 = arith.extsi %2 : tensor<1024xi32, #blocked> to tensor<1024xi64, #blocked>
171|     %ext3 = arith.extsi %3 : tensor<1024xi32, #blocked> to tensor<1024xi64, #blocked>
172|     %4 = arith.addi %ext2, %ext3 : tensor<1024xi64, #blocked>
173|     // COMMON: %[[scalar_ptr:.*]] = tt.addptr %arg0
174|     %5 = tt.addptr %arg0, %1 : !tt.ptr<f32>, i32
175|     %8 = tt.splat %5 : !tt.ptr<f32> -> tensor<1024x!tt.ptr<f32>, #blocked>
176|     // COMMON: %[[offset_32_bit:.*]] = arith.trunci
177|     %narrow4 = arith.trunci %4 : tensor<1024xi64, #blocked> to tensor <1024xi32, #blocked>
178|     %9 = tt.addptr %8, %narrow4 : tensor<1024x!tt.ptr<f32>, #blocked>, tensor<1024xi32, #blocked>
179|     // Note: base is arg0 which is large-tensor, the offset=int(long(pid*1024) * long(arange(0, 1024))
180|     // offset is in [0, i32-max].
```
**EN:** This function-oriented block defines or enters `offset_64_bits_narrow`. Within it, the test exercises tt.func, constants, broadcasted scalars or pointers, arith.extsi, pointer arithmetic, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `offset_64_bits_narrow` 为核心。测试在其中演示 tt.func、常量、广播后的标量或指针、arith.extsi、指针算术，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 181-186
```mlir
181|     // COMMON-NOT: buffer_load
182|     %10 = tt.load %9 : tensor<1024x!tt.ptr<f32>, #blocked>
183|     tt.return %10 : tensor<1024xf32, #blocked>
184|   }
185| }
186| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 187-187
```mlir
187| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 188-189
```mlir
188| 
189| #blocked = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 190-191
```mlir
190| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32}  {
191|   // COMMON-LABEL: non_canonical_ptr
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 192-200
```mlir
192|   tt.func @non_canonical_ptr(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg1: tensor<1024xi32, #blocked>) -> tensor<1024xf32, #blocked>{
193|     %8 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<1024x!tt.ptr<f32>, #blocked>
194|     %9 = tt.addptr %8, %arg1: tensor<1024x!tt.ptr<f32>, #blocked>, tensor<1024xi32, #blocked>
195|     // COMMON: tt.load
196|     %10 = tt.load %9 : tensor<1024x!tt.ptr<f32>, #blocked>
197|     tt.return %10 : tensor<1024xf32, #blocked>
198|   }
199| }
200| 
```
**EN:** This function-oriented block defines or enters `non_canonical_ptr`. Within it, the test exercises tt.func, broadcasted scalars or pointers, pointer arithmetic, masked or vectorized loads, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `non_canonical_ptr` 为核心。测试在其中演示 tt.func、广播后的标量或指针、指针算术、带掩码或向量化的加载、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 201-201
```mlir
201| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 202-203
```mlir
202| 
203| #blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 204-205
```mlir
204| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
205|   // COMMON-LABEL: assume_eq_non_neg
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 206-223
```mlir
206|   tt.func @assume_eq_non_neg(%arg0: !tt.ptr<bf16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg1: !tt.ptr<bf16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg2: i32) {
207|     %c10_i32 = arith.constant 10 : i32
208|     %0 = arith.cmpi eq, %arg2, %c10_i32 : i32
209|     llvm.intr.assume %0 : i1
210|     // COMMON: %[[range:.*]] = tt.make_range
211|     %1 = tt.make_range {end = 16 : i32, start = 0 : i32} : tensor<16xi32, #blocked>
212|     // COMMON: %[[ptr:.*]] = tt.addptr %arg0, %arg2
213|     %2 = tt.addptr %arg0, %arg2: !tt.ptr<bf16>, i32
214|     %3 = tt.splat %2 : !tt.ptr<bf16> -> tensor<16x!tt.ptr<bf16>, #blocked>
215|     %4 = tt.addptr %3, %1 : tensor<16x!tt.ptr<bf16>, #blocked>, tensor<16xi32, #blocked>
216|     %5 = tt.splat %arg1 : !tt.ptr<bf16> -> tensor<16x!tt.ptr<bf16>, #blocked>
217|     %6 = tt.addptr %5, %1 : tensor<16x!tt.ptr<bf16>, #blocked>, tensor<16xi32, #blocked>
218|     // COMMON: %[[loaded:.*]] = amdg.buffer_load %arg1[%1]
219|     %7 = tt.load %6 : tensor<16x!tt.ptr<bf16>, #blocked>
220|     // COMMON: amdg.buffer_store %[[loaded]], %[[ptr]][%[[range]]]
221|     tt.store %4, %7 : tensor<16x!tt.ptr<bf16>, #blocked>
222|     tt.return
223|   }
```
**EN:** This function-oriented block defines or enters `assume_eq_non_neg`. Within it, the test exercises pointer arithmetic, tt.func, broadcasted scalars or pointers, constants, integer comparisons, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `assume_eq_non_neg` 为核心。测试在其中演示 指针算术、tt.func、广播后的标量或指针、常量、整数比较，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 224-225
```mlir
224| }
225| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 226-226
```mlir
226| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 227-228
```mlir
227| 
228| #blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 229-230
```mlir
229| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
230|   // COMMON-LABEL: assume_nonneg_less
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 231-248
```mlir
231|   tt.func @assume_nonneg_less(%arg0: !tt.ptr<bf16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg1: !tt.ptr<bf16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg2: i32) {
232|     %c10_i32 = arith.constant 5 : i32
233|     %0 = arith.cmpi slt, %c10_i32, %arg2 : i32
234|     llvm.intr.assume %0 : i1
235|     // COMMON: %[[range:.*]] = tt.make_range
236|     %1 = tt.make_range {end = 16 : i32, start = 0 : i32} : tensor<16xi32, #blocked>
237|     // COMMON: %[[ptr:.*]] = tt.addptr %arg0, %arg2
238|     %2 = tt.addptr %arg0, %arg2: !tt.ptr<bf16>, i32
239|     %3 = tt.splat %2 : !tt.ptr<bf16> -> tensor<16x!tt.ptr<bf16>, #blocked>
240|     %4 = tt.addptr %3, %1 : tensor<16x!tt.ptr<bf16>, #blocked>, tensor<16xi32, #blocked>
241|     %5 = tt.splat %arg1 : !tt.ptr<bf16> -> tensor<16x!tt.ptr<bf16>, #blocked>
242|     %6 = tt.addptr %5, %1 : tensor<16x!tt.ptr<bf16>, #blocked>, tensor<16xi32, #blocked>
243|     // COMMON: %[[loaded:.*]] = amdg.buffer_load %arg1[%1]
244|     %7 = tt.load %6 : tensor<16x!tt.ptr<bf16>, #blocked>
245|     // COMMON: amdg.buffer_store %[[loaded]], %[[ptr]][%[[range]]]
246|     tt.store %4, %7 : tensor<16x!tt.ptr<bf16>, #blocked>
247|     tt.return
248|   }
```
**EN:** This function-oriented block defines or enters `assume_nonneg_less`. Within it, the test exercises pointer arithmetic, tt.func, broadcasted scalars or pointers, constants, integer comparisons, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `assume_nonneg_less` 为核心。测试在其中演示 指针算术、tt.func、广播后的标量或指针、常量、整数比较，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 249-250
```mlir
249| }
250| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 251-251
```mlir
251| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 252-253
```mlir
252| 
253| #blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 254-255
```mlir
254| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
255|   // COMMON-LABEL: assume_cmp_non_const
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 256-273
```mlir
256|   tt.func @assume_cmp_non_const(%arg0: !tt.ptr<bf16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg1: !tt.ptr<bf16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg2: i32, %arg3 : i32, %arg4 : i32, %arg5 : i32, %arg6 : i32) {
257|     %0 = arith.cmpi sgt, %arg2, %arg3 : i32
258|     llvm.intr.assume %0 : i1
259|     %1 = arith.subi %arg2, %arg3 : i32
260|     %2 = arith.cmpi sge, %1, %arg4 : i32
261|     llvm.intr.assume %2 : i1
262|     %3 = arith.subi %1, %arg4 : i32
263|     %4 = arith.cmpi slt, %3, %arg5 : i32
264|     llvm.intr.assume %4 : i1
265|     %5 = arith.subi %arg5, %3 : i32
266|     %6 = arith.cmpi sle, %5, %arg6 : i32
267|     llvm.intr.assume %6 : i1
268|     %7 = arith.subi %arg6, %5 : i32
269|     %8 = arith.minsi %1, %3 : i32
270|     %9 = arith.minsi %8, %5 : i32
271|     %10 = arith.minsi %9, %7 : i32
272|     // COMMON: %[[range:.*]] = tt.make_range
273|     %11 = tt.make_range {end = 16 : i32, start = 0 : i32} : tensor<16xi32, #blocked>
```
**EN:** This function-oriented block defines or enters `assume_cmp_non_const`. Within it, the test exercises integer comparisons, llvm.intr, arith.subi, arith.minsi, tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `assume_cmp_non_const` 为核心。测试在其中演示 整数比较、llvm.intr、arith.subi、arith.minsi、tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 274-288
```mlir
274|     %12 = tt.splat %10 : i32 -> tensor<16xi32, #blocked>
275|     // COMMON: %[[offsets:.*]] = arith.addi
276|     %offsets = arith.addi %11, %12 : tensor<16xi32, #blocked>
277|     %13 = tt.splat %arg0 : !tt.ptr<bf16> -> tensor<16x!tt.ptr<bf16>, #blocked>
278|     %14 = tt.addptr %13, %11 : tensor<16x!tt.ptr<bf16>, #blocked>, tensor<16xi32, #blocked>
279|     %15 = tt.splat %arg1 : !tt.ptr<bf16> -> tensor<16x!tt.ptr<bf16>, #blocked>
280|     %16 = tt.addptr %15, %offsets : tensor<16x!tt.ptr<bf16>, #blocked>, tensor<16xi32, #blocked>
281|     // COMMON-NOT: amdg.buffer_load
282|     %17 = tt.load %16 : tensor<16x!tt.ptr<bf16>, #blocked>
283|     // COMMON: amdg.buffer_store
284|     tt.store %14, %17 : tensor<16x!tt.ptr<bf16>, #blocked>
285|     tt.return
286|   }
287| }
288| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on broadcasted scalars or pointers, pointer arithmetic, integer additions, masked or vectorized loads, masked or vectorized stores. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 广播后的标量或指针、指针算术、整数加法、带掩码或向量化的加载、带掩码或向量化的存储。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 289-289
```mlir
289| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 290-295
```mlir
290| 
291| #blockedsrc = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [0, 1]}>
292| #blocked = #ttg.blocked<{sizePerThread = [2, 2], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
293| #blockedtrans = #ttg.blocked<{sizePerThread = [2, 2], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
294| #blocked1 = #ttg.slice<{dim=0, parent=#blockedsrc}>
295| #blocked2 = #ttg.slice<{dim=0, parent=#blockedtrans}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 296-297
```mlir
296| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
297|   // COMMON-LABEL: unary_triton_ops_transitive_nonneg
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 298-315
```mlir
298|   tt.func @unary_triton_ops_transitive_nonneg(%arg0: !tt.ptr<bf16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg1: !tt.ptr<bf16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
299|     %c10_i32 = arith.constant 5 : i32
300|     %0 = tt.make_range {end = 16 : i32, start = 0 : i32} : tensor<16xi32, #blocked1>
301|     %1 = tt.expand_dims %0 {axis = 0 : i32} : tensor<16xi32, #blocked1> -> tensor<1x16xi32, #blockedsrc>
302|     %2 = tt.reshape %1 allow_reorder : tensor<1x16xi32, #blockedsrc> -> tensor<8x2xi32, #blocked>
303|     %3 = tt.reshape %1 allow_reorder : tensor<1x16xi32, #blockedsrc> -> tensor<2x8xi32, #blockedtrans>
304|     %4 = tt.trans %3 {order = array<i32: 1, 0>} : tensor<2x8xi32, #blockedtrans> -> tensor<8x2xi32, #blocked>
305|     %5 = ttg.convert_layout %4 : tensor<8x2xi32, #blocked> -> tensor<8x2xi32, #blocked>
306|     %6 = arith.addi %5, %2 : tensor<8x2xi32, #blocked>
307|     %7 = tt.make_range {end = 10 : i32, start = 2 : i32} : tensor<8xi32, #blocked2>
308|     %8 = ttg.convert_layout %7 : tensor<8xi32, #blocked2> -> tensor<8xi32, #blocked2>
309|     %9 = tt.expand_dims %8 {axis = 0 : i32} : tensor<8xi32, #blocked2> -> tensor<1x8xi32, #blockedtrans>
310|     %10 = tt.broadcast %9 : tensor<1x8xi32, #blockedtrans> -> tensor<2x8xi32, #blockedtrans>
311|     %11 = tt.reshape %10 allow_reorder : tensor<2x8xi32, #blockedtrans> -> tensor<8x2xi32, #blocked>
312|     %12 = tt.splat %c10_i32 : i32 -> tensor<8x2xi32, #blocked>
313|     %13 = arith.addi %11, %12 : tensor<8x2xi32, #blocked>
314|     %14 = arith.minsi %13, %5 : tensor<8x2xi32, #blocked>
315|     // COMMON: %[[lhs:.*]], %[[rhs:.*]] = tt.split
```
**EN:** This function-oriented block defines or enters `unary_triton_ops_transitive_nonneg`. Within it, the test exercises tensor reshaping, tt.func, lane/block index ranges, shape expansion, layout conversions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `unary_triton_ops_transitive_nonneg` 为核心。测试在其中演示 张量重塑、tt.func、lane/block 索引范围、形状扩展、布局转换，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 316-333
```mlir
316|     %15, %16 = tt.split %11: tensor<8x2xi32, #blocked> -> tensor<8xi32, #blocked2>
317|     %17 = tt.splat %arg0 : !tt.ptr<bf16> -> tensor<8x!tt.ptr<bf16>, #blocked2>
318|     %18 = tt.addptr %17, %15 : tensor<8x!tt.ptr<bf16>, #blocked2>, tensor<8xi32, #blocked2>
319|     // COMMON: %[[loaded:.*]] = amdg.buffer_load %arg0[%[[lhs]]]
320|     %19 = tt.load %18 : tensor<8x!tt.ptr<bf16>, #blocked2>
321|     %20 = tt.addptr %17, %16 : tensor<8x!tt.ptr<bf16>, #blocked2>, tensor<8xi32, #blocked2>
322|     // COMMON: %[[loaded2:.*]] = amdg.buffer_load %arg0[%[[rhs]]]
323|     %21 = tt.load %20 : tensor<8x!tt.ptr<bf16>, #blocked2>
324|     // COMMON: %[[added:.*]] = arith.addf %[[loaded]], %[[loaded2]]
325|     %22 = arith.addf %19, %21 : tensor<8xbf16, #blocked2>
326|     %23 = tt.splat %arg1 : !tt.ptr<bf16> -> tensor<8x!tt.ptr<bf16>, #blocked2>
327|     %24 = tt.addptr %23, %7 : tensor<8x!tt.ptr<bf16>, #blocked2>, tensor<8xi32, #blocked2>
328|     // COMMON: amdg.buffer_store %[[added]], %arg1[%{{.*}}]
329|     tt.store %24, %22 : tensor<8x!tt.ptr<bf16>, #blocked2>
330|     tt.return
331|   }
332| }
333| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on pointer arithmetic, broadcasted scalars or pointers, masked or vectorized loads, tt.split, floating-point additions. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 指针算术、广播后的标量或指针、带掩码或向量化的加载、tt.split、浮点加法。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 334-334
```mlir
334| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 335-340
```mlir
335| 
336| 
337| #blocked = #ttg.blocked<{sizePerThread = [2, 1], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
338| #blocked1 = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
339| #blocked2 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
340| #blocked3 = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 341-342
```mlir
341| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
342|   // COMMON-LABEL: join_cat_transitive_nonneg
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 343-360
```mlir
343|   tt.func @join_cat_transitive_nonneg(%arg0: !tt.ptr<bf16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg1: !tt.ptr<bf16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
344|     %0 = tt.make_range {end = 8 : i32, start = 0 : i32} : tensor<8xi32, #blocked1>
345|     %1 = tt.make_range {end = 10 : i32, start = 2 : i32} : tensor<8xi32, #blocked1>
346|     %2 = tt.join %0, %1 : tensor<8xi32, #blocked1> -> tensor<8x2xi32, #blocked>
347|     %3 = tt.make_range {end = 4 : i32, start = 0 : i32} : tensor<4xi32, #blocked2>
348|     %4 = tt.make_range {end = 8 : i32, start = 4 : i32} : tensor<4xi32, #blocked2>
349|     %5 = tt.join %3, %4 : tensor<4xi32, #blocked2> -> tensor<4x2xi32, #blocked3>
350|     %6 = tt.cat %5, %5 : tensor<4x2xi32, #blocked3> -> tensor<8x2xi32, #blocked>
351|     %7 = arith.addi %2, %6 : tensor<8x2xi32, #blocked>
352|     %zeros = arith.constant dense<0> : tensor<8x1xi32, #blocked>
353|     %ones = arith.constant dense<1> : tensor<8x1xi32, #blocked>
354|     %8 = tt.gather %7[%zeros] {axis = 1 : i32} : (tensor<8x2xi32, #blocked>, tensor<8x1xi32, #blocked>) -> tensor<8x1xi32, #blocked>
355|     %9 = tt.gather %7[%ones] {axis = 1 : i32} : (tensor<8x2xi32, #blocked>, tensor<8x1xi32, #blocked>) -> tensor<8x1xi32, #blocked>
356|     %10 = arith.addi %8, %9 : tensor<8x1xi32, #blocked>
357|     %11 = tt.reshape %10 allow_reorder : tensor<8x1xi32, #blocked> -> tensor<8xi32, #blocked1>
358|     %12 = tt.splat %arg0 : !tt.ptr<bf16> -> tensor<8x!tt.ptr<bf16>, #blocked1>
359|     %14 = tt.addptr %12, %11 : tensor<8x!tt.ptr<bf16>, #blocked1>, tensor<8xi32, #blocked1>
360|     // COMMON: %[[loaded:.*]] = amdg.buffer_load %arg0[%{{.*}}]
```
**EN:** This function-oriented block defines or enters `join_cat_transitive_nonneg`. Within it, the test exercises lane/block index ranges, tt.func, tt.join, integer additions, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `join_cat_transitive_nonneg` 为核心。测试在其中演示 lane/block 索引范围、tt.func、tt.join、整数加法、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 361-369
```mlir
361|     %15 = tt.load %14 : tensor<8x!tt.ptr<bf16>, #blocked1>
362|     %16 = tt.splat %arg1 : !tt.ptr<bf16> -> tensor<8x!tt.ptr<bf16>, #blocked1>
363|     %17 = tt.addptr %16, %0 : tensor<8x!tt.ptr<bf16>, #blocked1>, tensor<8xi32, #blocked1>
364|     // COMMON: amdg.buffer_store %[[loaded]], %arg1[%{{.*}}]
365|     tt.store %17, %15 : tensor<8x!tt.ptr<bf16>, #blocked1>
366|     tt.return
367|   }
368| }
369| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized loads, broadcasted scalars or pointers, pointer arithmetic, masked or vectorized stores, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的加载、广播后的标量或指针、指针算术、带掩码或向量化的存储、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 370-370
```mlir
370| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 371-372
```mlir
371| 
372| #blocked = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 373-374
```mlir
373| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
374|   // COMMON-LABEL: histo_nonneg
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 375-392
```mlir
375|   tt.func @histo_nonneg(%arg0: !tt.ptr<bf16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg1: !tt.ptr<bf16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg2 : tensor<256xi32, #blocked>) {
376|     /// Purposely specify %arg2 so that we can't statically determine the input
377|     /// data is nonneg.
378|     // COMMON: tt.histogram
379|     %0 = tt.histogram %arg2 : tensor<256xi32, #blocked> -> tensor<8xi32, #blocked>
380|     %1 = tt.make_range {end = 8 : i32, start = 0 : i32} : tensor<8xi32, #blocked>
381|     %2 = tt.splat %arg0 : !tt.ptr<bf16> -> tensor<8x!tt.ptr<bf16>, #blocked>
382|     %3 = tt.addptr %2, %0 : tensor<8x!tt.ptr<bf16>, #blocked>, tensor<8xi32, #blocked>
383|     // Note: index is tt.histogram ∈ [0, smax)
384|     // COMMON-NOT: amdg.buffer_load
385|     %4 = tt.load %3 : tensor<8x!tt.ptr<bf16>, #blocked>
386|     %5 = tt.splat %arg1 : !tt.ptr<bf16> -> tensor<8x!tt.ptr<bf16>, #blocked>
387|     %6 = tt.addptr %5, %1 : tensor<8x!tt.ptr<bf16>, #blocked>, tensor<8xi32, #blocked>
388|     // Note: index is tt.histogram ∈ [0, smax)
389|     // COMMON: amdg.buffer_store
390|     tt.store %6, %4 : tensor<8x!tt.ptr<bf16>, #blocked>
391|     tt.return
392|   }
```
**EN:** This function-oriented block defines or enters `histo_nonneg`. Within it, the test exercises tt.func, broadcasted scalars or pointers, pointer arithmetic, tt.histogram, lane/block index ranges, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `histo_nonneg` 为核心。测试在其中演示 tt.func、广播后的标量或指针、指针算术、tt.histogram、lane/block 索引范围，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 393-394
```mlir
393| }
394| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 395-395
```mlir
395| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 396-397
```mlir
396| 
397| #blocked = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 398-399
```mlir
398| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
399|   // COMMON-LABEL: get_num_prog_nonneg
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 400-417
```mlir
400|   tt.func @get_num_prog_nonneg(%arg0: !tt.ptr<bf16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg1: !tt.ptr<bf16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg2 : i32) {
401|     %0 = tt.get_num_programs x : i32
402|     %1 = tt.get_num_programs y : i32
403|     %2 = tt.get_num_programs z : i32
404|     %3 = arith.minsi %0, %1 : i32
405|     %4 = arith.minsi %2, %3 : i32
406|     %5 = arith.maxsi %arg2, %4 : i32
407|     %6 = tt.splat %5 : i32 -> tensor<8xi32, #blocked>
408|     %7 = tt.make_range {end = 8 : i32, start = 0 : i32} : tensor<8xi32, #blocked>
409|     %8 = arith.addi %6, %7 : tensor<8xi32, #blocked>
410|     %9 = tt.splat %arg0 : !tt.ptr<bf16> -> tensor<8x!tt.ptr<bf16>, #blocked>
411|     %10 = tt.addptr %9, %8 : tensor<8x!tt.ptr<bf16>, #blocked>, tensor<8xi32, #blocked>
412|     // COMMON-NOT: amdg.buffer_load
413|     %11 = tt.load %10 : tensor<8x!tt.ptr<bf16>, #blocked>
414|     %12 = tt.splat %arg1 : !tt.ptr<bf16> -> tensor<8x!tt.ptr<bf16>, #blocked>
415|     %13 = tt.addptr %12, %7 : tensor<8x!tt.ptr<bf16>, #blocked>, tensor<8xi32, #blocked>
416|     // COMMON: amdg.buffer_store
417|     tt.store %13, %11 : tensor<8x!tt.ptr<bf16>, #blocked>
```
**EN:** This function-oriented block defines or enters `get_num_prog_nonneg`. Within it, the test exercises tt.get_num_programs, broadcasted scalars or pointers, tt.func, arith.minsi, pointer arithmetic, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `get_num_prog_nonneg` 为核心。测试在其中演示 tt.get_num_programs、广播后的标量或指针、tt.func、arith.minsi、指针算术，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 418-421
```mlir
418|     tt.return
419|   }
420| }
421| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 422-422
```mlir
422| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 423-424
```mlir
423| 
424| #blocked = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 425-426
```mlir
425| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
426|   // COMMON-LABEL: unsigned_ops
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 427-444
```mlir
427|   tt.func @unsigned_ops(%arg0: !tt.ptr<bf16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg1: !tt.ptr<bf16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg2 : i32, %arg3 : i32, %arg4 : f32) {
428|     %c5_i32 = arith.constant 5 : i32
429|     %0 = arith.ceildivui %arg2, %c5_i32 : i32
430|     %1 = arith.divui %arg3, %c5_i32 : i32
431|     %2 = arith.fptoui %arg4 : f32 to i32
432|     %4 = arith.maxui %arg2, %arg3 : i32
433|     %5 = arith.minui %arg2, %arg3 : i32
434|     %6 = arith.remui %arg2, %c5_i32 : i32
435|     %7 = arith.shrui %arg3, %c5_i32 : i32
436|     %8 = arith.addi %0, %1 : i32
437|     %10 = arith.addi %4, %5 : i32
438|     %11 = arith.addi %6, %7 : i32
439|     %12 = arith.addi %8, %2 : i32
440|     %13 = arith.addi %10, %11 : i32
441|     %14 = arith.addi %8, %13 : i32
442|     %15 = tt.splat %14 : i32 -> tensor<8xi32, #blocked>
443|     %16 = tt.make_range {end = 8 : i32, start = 0 : i32} : tensor<8xi32, #blocked>
444|     %17 = arith.addi %15, %16 : tensor<8xi32, #blocked>
```
**EN:** This function-oriented block defines or enters `unsigned_ops`. Within it, the test exercises integer additions, tt.func, constants, arith.ceildivui, arith.divui, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `unsigned_ops` 为核心。测试在其中演示 整数加法、tt.func、常量、arith.ceildivui、arith.divui，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 445-457
```mlir
445|     %18 = tt.splat %arg0 : !tt.ptr<bf16> -> tensor<8x!tt.ptr<bf16>, #blocked>
446|     %19 = tt.addptr %18, %17 : tensor<8x!tt.ptr<bf16>, #blocked>, tensor<8xi32, #blocked>
447|     // Note: above operations can only prove elmtIdx >= 0 not don't reveal its upper bound.
448|     // COMMON-NOT: amdg.buffer_load
449|     %20 = tt.load %19 : tensor<8x!tt.ptr<bf16>, #blocked>
450|     %21 = tt.splat %arg1 : !tt.ptr<bf16> -> tensor<8x!tt.ptr<bf16>, #blocked>
451|     %22 = tt.addptr %21, %16 : tensor<8x!tt.ptr<bf16>, #blocked>, tensor<8xi32, #blocked>
452|     // COMMON: amdg.buffer_store
453|     tt.store %22, %20 : tensor<8x!tt.ptr<bf16>, #blocked>
454|     tt.return
455|   }
456| }
457| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on broadcasted scalars or pointers, pointer arithmetic, masked or vectorized loads, masked or vectorized stores, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 广播后的标量或指针、指针算术、带掩码或向量化的加载、带掩码或向量化的存储、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 458-458
```mlir
458| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 459-460
```mlir
459| 
460| #blocked = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 461-462
```mlir
461| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
462|   // COMMON-LABEL: extui_nonneg
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 463-480
```mlir
463|   tt.func @extui_nonneg(%arg0: !tt.ptr<bf16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg1: !tt.ptr<bf16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg2 : i32) {
464|     %0 = arith.extui %arg2 : i32 to i64
465|     %1 = tt.splat %0 : i64 -> tensor<8xi64, #blocked>
466|     %2 = tt.make_range {end = 8 : i32, start = 0 : i32} : tensor<8xi32, #blocked>
467|     %3 = arith.extui %2 : tensor<8xi32, #blocked> to tensor<8xi64, #blocked>
468|     %4 = arith.addi %1, %3 : tensor<8xi64, #blocked>
469|     %5 = arith.trunci %4 : tensor<8xi64, #blocked> to tensor<8xi32, #blocked>
470|     %6 = tt.splat %arg0 : !tt.ptr<bf16> -> tensor<8x!tt.ptr<bf16>, #blocked>
471|     %7 = tt.addptr %6, %5 : tensor<8x!tt.ptr<bf16>, #blocked>, tensor<8xi32, #blocked>
472|     // Note: elemIdx is (int32)(arange(0, 8) + (uint64)(uint32)arg2)
473|     // elemIdx is not necessarilly >=0
474|     // COMMON-NOT: amdg.buffer_load
475|     %8 = tt.load %7: tensor<8x!tt.ptr<bf16>, #blocked>
476|     %9 = tt.splat %arg1 : !tt.ptr<bf16> -> tensor<8x!tt.ptr<bf16>, #blocked>
477|     %10 = tt.addptr %9, %2 : tensor<8x!tt.ptr<bf16>, #blocked>, tensor<8xi32, #blocked>
478|     // COMMON: amdg.buffer_store
479|     tt.store %10, %8 : tensor<8x!tt.ptr<bf16>, #blocked>
480|     tt.return
```
**EN:** This function-oriented block defines or enters `extui_nonneg`. Within it, the test exercises broadcasted scalars or pointers, tt.func, arith.extui, pointer arithmetic, lane/block index ranges, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `extui_nonneg` 为核心。测试在其中演示 广播后的标量或指针、tt.func、arith.extui、指针算术、lane/block 索引范围，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 481-483
```mlir
481|   }
482| }
483| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 484-484
```mlir
484| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 485-486
```mlir
485| 
486| #blocked = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 487-488
```mlir
487| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
488|   // COMMON-LABEL: traverse_if
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 489-506
```mlir
489|   tt.func @traverse_if(%arg0: !tt.ptr<bf16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg1: !tt.ptr<bf16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg2 : i32, %arg3 : i32) {
490|     %c0_i32 = arith.constant 0 : i32
491|     %c2_i32 = arith.constant 2 : i32
492|     %c5_i32 = arith.constant 7 : i32
493|     %c7_i32 = arith.constant 5 : i32
494|     %0 = arith.extui %arg2 : i32 to i64
495|     %1 = arith.remui %arg2, %c2_i32 : i32
496|     %2 = arith.cmpi eq, %1, %c0_i32 : i32
497|     %3 = scf.if %2 -> tensor<8xi64, #blocked> {
498|       %20 = tt.make_range {end = 8 : i32, start = 0 : i32} : tensor<8xi32, #blocked>
499|       %21 = arith.extui %20 : tensor<8xi32, #blocked> to tensor<8xi64, #blocked>
500|       %22 = tt.splat %arg3 : i32 -> tensor<8xi32, #blocked>
501|       %23 = arith.extui %22 : tensor<8xi32, #blocked> to tensor<8xi64, #blocked>
502|       %24 = arith.addi %21, %23 : tensor<8xi64, #blocked>
503|       scf.yield %24 : tensor<8xi64, #blocked>
504|     } else {
505|       %30 = tt.make_range {end = 16 : i32, start = 8 : i32} : tensor<8xi32, #blocked>
506|       %31 = arith.extui %30 : tensor<8xi32, #blocked> to tensor<8xi64, #blocked>
```
**EN:** This function-oriented block defines or enters `traverse_if`. Within it, the test exercises constants, arith.extui, tt.func, lane/block index ranges, arith.remui, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `traverse_if` 为核心。测试在其中演示 常量、arith.extui、tt.func、lane/block 索引范围、arith.remui，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 507-524
```mlir
507|       %32 = tt.splat %0 : i64 -> tensor<8xi64, #blocked>
508|       %33 = arith.addi %31, %32 : tensor<8xi64, #blocked>
509|       scf.yield %33 : tensor<8xi64, #blocked>
510|     }
511|     %4 = arith.trunci %3 : tensor<8xi64, #blocked> to tensor<8xi32, #blocked>
512|     %5 = tt.splat %arg0 : !tt.ptr<bf16> -> tensor<8x!tt.ptr<bf16>, #blocked>
513|     %6 = tt.addptr %5, %4 : tensor<8x!tt.ptr<bf16>, #blocked>, tensor<8xi32, #blocked>
514|     // Note: It's not able to prove that the value range of elmtIdx in [0,1G].
515|     // testing case traverse_if_2nd, traverse_if_2nd_v2 and traverse_if_2nd_v3
516|     // works better than this case for this purpose.
517|     // COMMON-NOT:amdg.buffer_load
518|     %7 = tt.load %6: tensor<8x!tt.ptr<bf16>, #blocked>
519|     %8 = tt.make_range {end = 8 : i32, start = 0 : i32} : tensor<8xi32, #blocked>
520|     %9 = tt.splat %arg1 : !tt.ptr<bf16> -> tensor<8x!tt.ptr<bf16>, #blocked>
521|     %10 = tt.addptr %9, %8 : tensor<8x!tt.ptr<bf16>, #blocked>, tensor<8xi32, #blocked>
522|     // COMMON: amdg.buffer_store
523|     tt.store %10, %7 : tensor<8x!tt.ptr<bf16>, #blocked>
524|     tt.return
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining broadcasted scalars or pointers, pointer arithmetic, integer additions, loop/if yielded values, arith.trunci.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 广播后的标量或指针、指针算术、整数加法、循环/分支产出值、arith.trunci。

### Lines 525-527
```mlir
525|   }
526| }
527| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 528-528
```mlir
528| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 529-530
```mlir
529| 
530| #blocked = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 531-532
```mlir
531| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
532|   // COMMON-LABEL: traverse_if_2nd
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 533-550
```mlir
533|   tt.func @traverse_if_2nd(%arg0: !tt.ptr<bf16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg1: !tt.ptr<bf16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg2 : i32, %arg3 : i32) {
534|     %c0_i32 = arith.constant 0 : i32
535|     %c2_i32 = arith.constant 2 : i32
536|     %c5_i32 = arith.constant 7 : i32
537|     %c7_i32 = arith.constant 5 : i32
538|     %zeros = arith.constant dense<0> : tensor<8xi32, #blocked>
539|     %0 = arith.extui %arg2 : i32 to i64
540|     %1 = arith.remui %arg2, %c2_i32 : i32
541|     %2 = arith.cmpi eq, %1, %c0_i32 : i32
542|     %3, %4 = scf.if %2 -> (tensor<8xi64, #blocked>, tensor<8xi32, #blocked>) {
543|       %20 = tt.make_range {end = 8 : i32, start = 0 : i32} : tensor<8xi32, #blocked>
544|       %21 = arith.extui %20 : tensor<8xi32, #blocked> to tensor<8xi64, #blocked>
545|       %22 = tt.splat %arg3 : i32 -> tensor<8xi32, #blocked>
546|       %23 = arith.extui %22 : tensor<8xi32, #blocked> to tensor<8xi64, #blocked>
547|       %24 = arith.addi %21, %23 : tensor<8xi64, #blocked>
548|       %25 = tt.make_range {end = 9 : i32, start = 1 : i32} : tensor<8xi32, #blocked>
549|       scf.yield %24, %25 : tensor<8xi64, #blocked>, tensor<8xi32, #blocked>
550|     } else {
```
**EN:** This function-oriented block defines or enters `traverse_if_2nd`. Within it, the test exercises constants, arith.extui, tt.func, lane/block index ranges, arith.remui, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `traverse_if_2nd` 为核心。测试在其中演示 常量、arith.extui、tt.func、lane/block 索引范围、arith.remui，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 551-568
```mlir
551|       %30 = tt.make_range {end = 16 : i32, start = 8 : i32} : tensor<8xi32, #blocked>
552|       %31 = arith.extui %30 : tensor<8xi32, #blocked> to tensor<8xi64, #blocked>
553|       %32 = tt.splat %0 : i64 -> tensor<8xi64, #blocked>
554|       %33 = arith.addi %31, %32 : tensor<8xi64, #blocked>
555|       scf.yield %33, %zeros : tensor<8xi64, #blocked>, tensor<8xi32, #blocked>
556|     }
557|     %5 = arith.trunci %3 : tensor<8xi64, #blocked> to tensor<8xi32, #blocked>
558|     %6 = arith.addi %4, %5 : tensor<8xi32, #blocked>
559|     %7 = tt.splat %arg0 : !tt.ptr<bf16> -> tensor<8x!tt.ptr<bf16>, #blocked>
560|     %8 = tt.addptr %7, %6 : tensor<8x!tt.ptr<bf16>, #blocked>, tensor<8xi32, #blocked>
561|     // COMMON-NOT: amdg.buffer_load
562|     %9 = tt.load %8: tensor<8x!tt.ptr<bf16>, #blocked>
563|     %10 = tt.make_range {end = 8 : i32, start = 0 : i32} : tensor<8xi32, #blocked>
564|     %11 = tt.splat %arg1 : !tt.ptr<bf16> -> tensor<8x!tt.ptr<bf16>, #blocked>
565|     %12 = tt.addptr %11, %10 : tensor<8x!tt.ptr<bf16>, #blocked>, tensor<8xi32, #blocked>
566|     // COMMON: amdg.buffer_store
567|     tt.store %12, %9 : tensor<8x!tt.ptr<bf16>, #blocked>
568|     tt.return
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining broadcasted scalars or pointers, lane/block index ranges, integer additions, pointer arithmetic, arith.extui.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 广播后的标量或指针、lane/block 索引范围、整数加法、指针算术、arith.extui。

### Lines 569-571
```mlir
569|   }
570| }
571| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 572-572
```mlir
572| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 573-574
```mlir
573| 
574| #blocked = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 575-576
```mlir
575| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
576|   // COMMON-LABEL: traverse_if_2nd_v2
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 577-594
```mlir
577|   tt.func @traverse_if_2nd_v2(%arg0: !tt.ptr<bf16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg1: !tt.ptr<bf16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg2 : i32, %arg3 : i32) {
578|     %c0_i32 = arith.constant 0 : i32
579|     %c2_i32 = arith.constant 2 : i32
580|     %c5_i32 = arith.constant 7 : i32
581|     %c7_i32 = arith.constant 5 : i32
582|     %zeros = arith.constant dense<0> : tensor<8xi32, #blocked>
583|     %0 = arith.extui %arg2 : i32 to i64
584|     %1 = arith.remui %arg2, %c2_i32 : i32
585|     %2 = arith.cmpi eq, %1, %c0_i32 : i32
586|     %3, %4 = scf.if %2 -> (tensor<8xi64, #blocked>, tensor<8xi32, #blocked>) {
587|       %20 = tt.make_range {end = 8 : i32, start = 0 : i32} : tensor<8xi32, #blocked>
588|       %21 = arith.extui %20 : tensor<8xi32, #blocked> to tensor<8xi64, #blocked>
589|       %22 = tt.splat %arg3 : i32 -> tensor<8xi32, #blocked>
590|       %23 = arith.extui %22 : tensor<8xi32, #blocked> to tensor<8xi64, #blocked>
591|       %24 = arith.addi %21, %23 : tensor<8xi64, #blocked>
592|       %25 = tt.make_range {end = 9 : i32, start = 1 : i32} : tensor<8xi32, #blocked>
593|       scf.yield %24, %25 : tensor<8xi64, #blocked>, tensor<8xi32, #blocked>
594|     } else {
```
**EN:** This function-oriented block defines or enters `traverse_if_2nd_v2`. Within it, the test exercises constants, arith.extui, tt.func, lane/block index ranges, arith.remui, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `traverse_if_2nd_v2` 为核心。测试在其中演示 常量、arith.extui、tt.func、lane/block 索引范围、arith.remui，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 595-612
```mlir
595|       %30 = tt.make_range {end = 16 : i32, start = 8 : i32} : tensor<8xi32, #blocked>
596|       %31 = arith.extui %30 : tensor<8xi32, #blocked> to tensor<8xi64, #blocked>
597|       %32 = tt.splat %0 : i64 -> tensor<8xi64, #blocked>
598|       %33 = arith.addi %31, %32 : tensor<8xi64, #blocked>
599|       scf.yield %33, %zeros : tensor<8xi64, #blocked>, tensor<8xi32, #blocked>
600|     }
601|     %5 = arith.trunci %3 : tensor<8xi64, #blocked> to tensor<8xi32, #blocked>
602|     %6 = arith.addi %4, %5 : tensor<8xi32, #blocked>
603|     %7 = tt.splat %arg0 : !tt.ptr<bf16> -> tensor<8x!tt.ptr<bf16>, #blocked>
604|     %8 = tt.addptr %7, %6 : tensor<8x!tt.ptr<bf16>, #blocked>, tensor<8xi32, #blocked>
605| 
606|     // Note:
607|     // elmtIdx = %6 = %4 + %5, value-range(%4) = [0,7], value-range(%5) = [0, umax]
608|     // %5 = max([0,8] + arg3, [8,16) + arg2), to make %6 * sizeof(bf16) <= 2G - 2byte
609|     // arg3 ∈ [0, 1G-1-8-7 = 1073741808), arg2 ∈  [-8, 1G-1-15-8=1073741800]
610|     %cmp1 = arith.cmpi sge, %arg2, %c0_i32 : i32
611|     llvm.intr.assume %cmp1 : i1
612|     %cmp2 = arith.cmpi sge, %arg3, %c0_i32 : i32
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining broadcasted scalars or pointers, integer additions, integer comparisons, lane/block index ranges, arith.extui.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 广播后的标量或指针、整数加法、整数比较、lane/block 索引范围、arith.extui。

### Lines 613-630
```mlir
613|     llvm.intr.assume %cmp2 : i1
614|     %arg_up2 = arith.constant 1073741800 : i32
615|     %arg_up3 = arith.constant 1073741808 : i32
616|     %cmp3 = arith.cmpi slt, %arg2, %arg_up2 : i32
617|     %cmp4 = arith.cmpi slt, %arg3, %arg_up3 : i32
618|     llvm.intr.assume %cmp3 : i1
619|     llvm.intr.assume %cmp4 : i1
620| 
621|     // COMMON: %[[loaded:.*]] = amdg.buffer_load %arg0[%{{.*}}]
622|     %9 = tt.load %8: tensor<8x!tt.ptr<bf16>, #blocked>
623|     %10 = tt.make_range {end = 8 : i32, start = 0 : i32} : tensor<8xi32, #blocked>
624|     %11 = tt.splat %arg1 : !tt.ptr<bf16> -> tensor<8x!tt.ptr<bf16>, #blocked>
625|     %12 = tt.addptr %11, %10 : tensor<8x!tt.ptr<bf16>, #blocked>, tensor<8xi32, #blocked>
626|     // COMMON: amdg.buffer_store %[[loaded]], %arg1[%{{.*}}]
627|     tt.store %12, %9 : tensor<8x!tt.ptr<bf16>, #blocked>
628|     tt.return
629|   }
630| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on llvm.intr, constants, integer comparisons, masked or vectorized loads, lane/block index ranges. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 llvm.intr、常量、整数比较、带掩码或向量化的加载、lane/block 索引范围。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 632-633
```mlir
632| 
633| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 633-634
```mlir
633| 
634| #blocked = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 635-636
```mlir
635| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
636|   // COMMON-LABEL: traverse_if_2nd_v3
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 637-654
```mlir
637|   tt.func @traverse_if_2nd_v3(%arg0: !tt.ptr<bf16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg1: !tt.ptr<bf16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg2 : i32, %arg3 : i32) {
638|     %c0_i32 = arith.constant 0 : i32
639|     %c2_i32 = arith.constant 2 : i32
640|     %c5_i32 = arith.constant 7 : i32
641|     %c7_i32 = arith.constant 5 : i32
642|     %zeros = arith.constant dense<0> : tensor<8xi32, #blocked>
643|     %0 = arith.extui %arg2 : i32 to i64
644|     %1 = arith.remui %arg2, %c2_i32 : i32
645|     %2 = arith.cmpi eq, %1, %c0_i32 : i32
646|     %3, %4 = scf.if %2 -> (tensor<8xi64, #blocked>, tensor<8xi32, #blocked>) {
647|       %20 = tt.make_range {end = 8 : i32, start = 0 : i32} : tensor<8xi32, #blocked>
648|       %21 = arith.extui %20 : tensor<8xi32, #blocked> to tensor<8xi64, #blocked>
649|       %22 = tt.splat %arg3 : i32 -> tensor<8xi32, #blocked>
650|       %23 = arith.extui %22 : tensor<8xi32, #blocked> to tensor<8xi64, #blocked>
651|       %24 = arith.addi %21, %23 : tensor<8xi64, #blocked>
652|       %25 = tt.make_range {end = 9 : i32, start = 1 : i32} : tensor<8xi32, #blocked>
653|       scf.yield %24, %25 : tensor<8xi64, #blocked>, tensor<8xi32, #blocked>
654|     } else {
```
**EN:** This function-oriented block defines or enters `traverse_if_2nd_v3`. Within it, the test exercises constants, arith.extui, tt.func, lane/block index ranges, arith.remui, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `traverse_if_2nd_v3` 为核心。测试在其中演示 常量、arith.extui、tt.func、lane/block 索引范围、arith.remui，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 655-672
```mlir
655|       %30 = tt.make_range {end = 16 : i32, start = 8 : i32} : tensor<8xi32, #blocked>
656|       %31 = arith.extui %30 : tensor<8xi32, #blocked> to tensor<8xi64, #blocked>
657|       %32 = tt.splat %0 : i64 -> tensor<8xi64, #blocked>
658|       %33 = arith.addi %31, %32 : tensor<8xi64, #blocked>
659|       scf.yield %33, %zeros : tensor<8xi64, #blocked>, tensor<8xi32, #blocked>
660|     }
661|     %5 = arith.trunci %3 : tensor<8xi64, #blocked> to tensor<8xi32, #blocked>
662|     %6 = arith.addi %4, %5 : tensor<8xi32, #blocked>
663|     %7 = tt.splat %arg0 : !tt.ptr<bf16> -> tensor<8x!tt.ptr<bf16>, #blocked>
664|     %8 = tt.addptr %7, %6 : tensor<8x!tt.ptr<bf16>, #blocked>, tensor<8xi32, #blocked>
665| 
666|     // Note:
667|     // elmtIdx = %6 = %4 + %5, value-range(%4) = [0,7], value-range(%5) = [0, umax]
668|     // %5 = max([0,8] + arg3, [8,16) + arg2), to make %6 * sizeof(bf16) <= 2G - 2byte
669|     // arg3 ∈ [0, 1G-1-8-7 = 1073741808), arg2 ∈  [-8, 1G-1-15-8=1073741800]
670|     %cmp1 = arith.cmpi sge, %arg2, %c0_i32 : i32
671|     llvm.intr.assume %cmp1 : i1
672|     %cmp2 = arith.cmpi sge, %arg3, %c0_i32 : i32
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining broadcasted scalars or pointers, integer additions, integer comparisons, lane/block index ranges, arith.extui.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 广播后的标量或指针、整数加法、整数比较、lane/block 索引范围、arith.extui。

### Lines 673-690
```mlir
673|     llvm.intr.assume %cmp2 : i1
674|     // the only difference between traverse_if_2nd_v3 and traverse_if_2nd_v2
675|     // is arg_up2. In v3 the upper bound is bumped by 1.
676|     %arg_up2 = arith.constant 1073741801 : i32
677|     %arg_up3 = arith.constant 1073741808 : i32
678|     %cmp3 = arith.cmpi slt, %arg2, %arg_up2 : i32
679|     %cmp4 = arith.cmpi slt, %arg3, %arg_up3 : i32
680|     llvm.intr.assume %cmp3 : i1
681|     llvm.intr.assume %cmp4 : i1
682| 
683|     // COMMON-NOT: amdg.buffer_load
684|     %9 = tt.load %8: tensor<8x!tt.ptr<bf16>, #blocked>
685|     %10 = tt.make_range {end = 8 : i32, start = 0 : i32} : tensor<8xi32, #blocked>
686|     %11 = tt.splat %arg1 : !tt.ptr<bf16> -> tensor<8x!tt.ptr<bf16>, #blocked>
687|     %12 = tt.addptr %11, %10 : tensor<8x!tt.ptr<bf16>, #blocked>, tensor<8xi32, #blocked>
688|     // COMMON: amdg.buffer_store
689|     tt.store %12, %9 : tensor<8x!tt.ptr<bf16>, #blocked>
690|     tt.return
```
**EN:** This operational block carries the concrete IR steps for the test, centered on llvm.intr, constants, integer comparisons, masked or vectorized loads, lane/block index ranges. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 llvm.intr、常量、整数比较、带掩码或向量化的加载、lane/block 索引范围。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 691-693
```mlir
691|   }
692| }
693| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 694-694
```mlir
694| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 695-696
```mlir
695| 
696| #blocked = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [64], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 697-698
```mlir
697| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
698|   // COMMON-LABEL: atomic_add_bf16
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 699-715
```mlir
699|   tt.func public @atomic_add_bf16(%arg0: !tt.ptr<bf16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
700|     %cst = arith.constant dense<true> : tensor<512xi1, #blocked>
701|     %cst_0 = arith.constant dense<1.000000e+00> : tensor<512xbf16, #blocked>
702|     %c512_i32 = arith.constant 512 : i32
703|     %0 = tt.get_program_id x : i32
704|     %1 = arith.muli %0, %c512_i32 : i32
705|     %2 = tt.make_range {end = 512 : i32, start = 0 : i32} : tensor<512xi32, #blocked>
706|     %3 = tt.addptr %arg0, %1 : !tt.ptr<bf16>, i32
707|     %4 = tt.splat %3 : !tt.ptr<bf16> -> tensor<512x!tt.ptr<bf16>, #blocked>
708|     %5 = tt.addptr %4, %2 : tensor<512x!tt.ptr<bf16>, #blocked>, tensor<512xi32, #blocked>
709|     // GFX942-ONLY-NOT: amdg.buffer_atomic_rmw
710|     // GFX950-PLUS: amdg.buffer_atomic_rmw
711|     %6 = tt.atomic_rmw fadd, acq_rel, gpu, %5, %cst_0, %cst : (tensor<512x!tt.ptr<bf16>, #blocked>, tensor<512xbf16, #blocked>, tensor<512xi1, #blocked>) -> tensor<512xbf16, #blocked>
712|     tt.return
713|   }
714| }
715| 
```
**EN:** This function-oriented block defines or enters `atomic_add_bf16`. Within it, the test exercises constants, tt.func, pointer arithmetic, program IDs, integer multiplications, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `atomic_add_bf16` 为核心。测试在其中演示 常量、tt.func、指针算术、程序 ID、整数乘法，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 716-716
```mlir
716| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 717-718
```mlir
717| 
718| #blocked = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 719-720
```mlir
719| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
720|   // COMMON-LABEL: assume_positive_offset_buffer_atomic
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 721-738
```mlir
721|   tt.func @assume_positive_offset_buffer_atomic(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg1: tensor<1024xf32, #blocked>) ->  tensor<1024xf32, #blocked>{
722|     %c1024_i32 = arith.constant 1024 : i32
723|     %c128_i32 = arith.constant 128 : i32
724|     %c0_i32 = arith.constant 0 : i32
725|     %0 = tt.get_program_id x : i32
726|     %1 = arith.muli %0, %c1024_i32 : i32
727|     %sub = arith.subi %1, %c128_i32 : i32
728|     %cmp = arith.cmpi sgt, %sub, %c0_i32 : i32
729|     llvm.intr.assume %cmp : i1
730|     %2 = tt.splat %sub : i32 -> tensor<1024xi32, #blocked>
731|     %3 = tt.make_range {end = 1024 : i32, start = 0 : i32} : tensor<1024xi32, #blocked>
732|     // COMMON: %[[offset:.*]] = arith.addi
733|     %4 = arith.addi %2, %3 : tensor<1024xi32, #blocked>
734|     // COMMON: %[[scalar_ptr:.*]] = tt.addptr %arg0
735|     %5 = tt.addptr %arg0, %1 : !tt.ptr<f32>, i32
736|     %6 = tt.splat %5 : !tt.ptr<f32> -> tensor<1024x!tt.ptr<f32>, #blocked>
737|     %7 = tt.addptr %6, %4 : tensor<1024x!tt.ptr<f32>, #blocked>, tensor<1024xi32, #blocked>
738|     // Note: the large tensor is accessed, offset is in the range of [0, smax].
```
**EN:** This function-oriented block defines or enters `assume_positive_offset_buffer_atomic`. Within it, the test exercises constants, tt.func, broadcasted scalars or pointers, pointer arithmetic, program IDs, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `assume_positive_offset_buffer_atomic` 为核心。测试在其中演示 常量、tt.func、广播后的标量或指针、指针算术、程序 ID，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 739-745
```mlir
739|     // without tl.assume the range would be [-128, smax]
740|     // COMMON-NOT: amdg.buffer_atomic_rmw
741|     %8 = tt.atomic_rmw fadd, acq_rel, gpu, %7, %arg1 : (tensor<1024x!tt.ptr<f32>, #blocked>, tensor<1024xf32, #blocked>) -> tensor<1024xf32, #blocked>
742|     tt.return %8 : tensor<1024xf32, #blocked>
743|   }
744| }
745| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.atomic_rmw, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.atomic_rmw、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 746-746
```mlir
746| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 747-750
```mlir
747| 
748| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [8, 1], order = [1, 0]}>
749| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
750| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 751-752
```mlir
751| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
752| // COMMON-LABEL: buffer_load_to_local
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 753-770
```mlir
753|   tt.func public @buffer_load_to_local(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg2: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg3: i32 {tt.divisibility = 16 : i32}, %arg4: i32 {tt.divisibility = 16 : i32}, %arg5: i32 {tt.divisibility = 16 : i32}, %arg6: i32 {tt.divisibility = 16 : i32}, %arg7: i32 {tt.divisibility = 16 : i32}, %arg8: i32 {tt.divisibility = 16 : i32}, %arg9: i32 {tt.divisibility = 16 : i32},
754|                                        %arg10: !ttg.memdesc<256x64xf16, #shared, #smem, mutable>, %arg11: tensor<256x64xi1, #blocked>, %arg12: tensor<256x64xf16, #blocked>) {
755|     %c48_i32 = arith.constant 48 : i32
756|     %c32_i32 = arith.constant 32 : i32
757|     %c0_i32 = arith.constant 0 : i32
758|     %0 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
759|     %1 = tt.expand_dims %0 {axis = 1 : i32} : tensor<256xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<256x1xi32, #blocked>
760|     %cmp = arith.cmpi sgt, %arg6, %c0_i32 : i32
761|     llvm.intr.assume %cmp : i1
762|     %2 = tt.splat %arg6 : i32 -> tensor<256x1xi32, #blocked>
763|     %3 = arith.muli %1, %2 : tensor<256x1xi32, #blocked>
764|     %4 = tt.addptr %arg0, %c32_i32 : !tt.ptr<f16>, i32
765|     %5 = tt.broadcast %3 : tensor<256x1xi32, #blocked> -> tensor<256x64xi32, #blocked>
766|     %6 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
767|     %7 = tt.expand_dims %6 {axis = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x64xi32, #blocked>
768|     %8 = tt.broadcast %7 : tensor<1x64xi32, #blocked> -> tensor<256x64xi32, #blocked>
769|     %9 = arith.addi %8, %5 : tensor<256x64xi32, #blocked>
770|     %10 = tt.splat %4 : !tt.ptr<f16> -> tensor<256x64x!tt.ptr<f16>, #blocked>
```
**EN:** This function-oriented block defines or enters `buffer_load_to_local`. Within it, the test exercises constants, tt.func, lane/block index ranges, shape expansion, broadcasted scalars or pointers, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `buffer_load_to_local` 为核心。测试在其中演示 常量、tt.func、lane/block 索引范围、形状扩展、广播后的标量或指针，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 771-788
```mlir
771|     %11 = tt.addptr %10, %9 : tensor<256x64x!tt.ptr<f16>, #blocked>, tensor<256x64xi32, #blocked>
772| 
773|     // COMMON: %[[splat:.*]] = tt.splat %arg[[#stride:]]
774|     // COMMON: %[[mul:.*]] = arith.muli %[[#]], %[[splat]]
775|     // COMMON: %[[ptr:.*]] = tt.addptr %arg0
776|     // COMMON: %[[bcast1:.*]] = tt.broadcast %[[mul]]
777|     // COMMON: %[[bcast0:.*]] = tt.broadcast %[[#]]
778|     // COMMON: %[[offset:.*]] = arith.addi %[[bcast0]], %[[bcast1]]
779| 
780|     // Note: offset(i.e. elmtIdx) = bcast0 + bcast1
781|     //   = arange(0, 64) + arg6 * arange(0, 256)
782|     // to make elmtIdx * sizeof(f16) ∈  [0, 2G], arg6 must be in [0, 4210752]
783|     %arg6_up = arith.constant 4210752: i32
784|     %cmp2 = arith.cmpi slt, %arg6, %arg6_up : i32
785|     llvm.intr.assume %cmp2 : i1
786| 
787|     // CDNA: %[[buffer:.*]] = amdg.buffer_load_to_local %[[ptr]][%[[offset]]] stride = %arg[[#stride]] into %arg10
788|     %12 = ttg.async_copy_global_to_local %11, %arg10 : tensor<256x64x!tt.ptr<f16>, #blocked> -> <256x64xf16, #shared, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on pointer arithmetic, constants, integer comparisons, llvm.intr, ttg.async_copy_global_to_local. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 指针算术、常量、整数比较、llvm.intr、ttg.async_copy_global_to_local。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 789-806
```mlir
789| 
790|     // CDNA: %[[buffer:.*]] = amdg.buffer_load_to_local %[[ptr]][%[[offset]]] other = %arg12 stride = %arg[[#stride]] into %arg10
791|     %13 = ttg.async_copy_global_to_local %11, %arg10 other %arg12: tensor<256x64x!tt.ptr<f16>, #blocked> -> <256x64xf16, #shared, #smem, mutable>
792| 
793|     // CDNA: %[[buffer:.*]] = amdg.buffer_load_to_local %[[ptr]][%[[offset]]] mask = %arg11 stride = %arg[[#stride]] into %arg10
794|     %14 = ttg.async_copy_global_to_local %11, %arg10 mask %arg11: tensor<256x64x!tt.ptr<f16>, #blocked> -> <256x64xf16, #shared, #smem, mutable>
795| 
796|     // CDNA: %[[buffer:.*]] = amdg.buffer_load_to_local %[[ptr]][%[[offset]]] mask = %arg11 other = %arg12 stride = %arg[[#stride]] into %arg10
797|     %15 = ttg.async_copy_global_to_local %11, %arg10 mask %arg11 other %arg12 : tensor<256x64x!tt.ptr<f16>, #blocked> -> <256x64xf16, #shared, #smem, mutable>
798| 
799|     // CDNA: %[[buffer:.*]] = amdg.buffer_load_to_local %[[ptr]][%[[offset]]] mask = %arg11 other = %arg12 stride = %arg[[#stride]] cacheModifier = ca into %arg10
800|     %16 = ttg.async_copy_global_to_local %11, %arg10 mask %arg11 other %arg12 cacheModifier = ca: tensor<256x64x!tt.ptr<f16>, #blocked> -> <256x64xf16, #shared, #smem, mutable>
801| 
802|     // CDNA: %[[buffer:.*]] = amdg.buffer_load_to_local %[[ptr]][%[[offset]]] mask = %arg11 other = %arg12 stride = %arg[[#stride]] cacheModifier = cg into %arg10
803|     %17 = ttg.async_copy_global_to_local %11, %arg10 mask %arg11 other %arg12 cacheModifier = cg: tensor<256x64x!tt.ptr<f16>, #blocked> -> <256x64xf16, #shared, #smem, mutable>
804| 
805|     // CDNA: %[[buffer:.*]] = amdg.buffer_load_to_local %[[ptr]][%[[offset]]] mask = %arg11 other = %arg12 stride = %arg[[#stride]] cacheModifier = cv into %arg10
806|     %18 = ttg.async_copy_global_to_local %11, %arg10 mask %arg11 other %arg12 cacheModifier = cv: tensor<256x64x!tt.ptr<f16>, #blocked> -> <256x64xf16, #shared, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.async_copy_global_to_local. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.async_copy_global_to_local。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 807-813
```mlir
807| 
808|     // CDNA: %[[buffer:.*]] = amdg.buffer_load_to_local %[[ptr]][%[[offset]]] stride = %arg[[#stride]] into %arg10 {contiguity = 8 : i32
809|     %19 = ttg.async_copy_global_to_local %11, %arg10 {contiguity = 8 : i32} : tensor<256x64x!tt.ptr<f16>, #blocked> -> <256x64xf16, #shared, #smem, mutable>
810|     tt.return
811|   }
812| }
813| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.async_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.async_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 814-814
```mlir
814| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 815-817
```mlir
815| 
816| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 16], warpsPerCTA = [2, 2], order = [1, 0]}>
817| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 818-818
```mlir
818| module attributes {"ttg.compute-capability" = 0 : i32, "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 819-836
```mlir
819|   tt.func @extract_slice(%arg0: !tt.ptr<f32>) -> tensor<128x256xf32, #blocked> {
820|     %0 = arith.constant dense<0> : tensor<256x256xi64, #blocked>
821|     %1 = amdg.extract_slice %0 [0, 0] : tensor<256x256xi64, #blocked> to tensor<128x256xi64, #blocked>
822|     %2 = arith.trunci %1 : tensor<128x256xi64, #blocked> to tensor<128x256xi32, #blocked>
823|     %3 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<128x256x!tt.ptr<f32>, #blocked>
824|     %4 = tt.addptr %3, %2 : tensor<128x256x!tt.ptr<f32>, #blocked>, tensor<128x256xi32, #blocked>
825|     %5 = tt.load %4 : tensor<128x256x!tt.ptr<f32>, #blocked>
826|     tt.return %5 : tensor<128x256xf32, #blocked>
827|   }
828| }
829| 
830| // COMMON-LABEL: tt.func @extract_slice(
831| // COMMON-SAME:    %[[ARG_0:.*]]: !tt.ptr<f32>) -> tensor<128x256xf32, #blocked> {
832| // COMMON:    %[[VAR_0:.*]] = arith.constant dense<0> : tensor<256x256xi64, #blocked>
833| // COMMON:    %[[VAR_1:.*]] = amdg.extract_slice %[[VAR_0]] [0, 0] : tensor<256x256xi64, #blocked> to tensor<128x256xi64, #blocked>
834| // COMMON:    %[[VAR_2:.*]] = arith.trunci %[[VAR_1]] : tensor<128x256xi64, #blocked> to tensor<128x256xi32, #blocked>
835| // COMMON:    %[[VAR_3:.*]] = amdg.buffer_load %[[ARG_0]][%[[VAR_2]]] : tensor<128x256xf32, #blocked>
836| // COMMON:    tt.return %[[VAR_3]] : tensor<128x256xf32, #blocked>
```
**EN:** This function-oriented block defines or enters `extract_slice`. Within it, the test exercises tt.func, constants, amdg.extract_slice, arith.trunci, broadcasted scalars or pointers, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `extract_slice` 为核心。测试在其中演示 tt.func、常量、amdg.extract_slice、arith.trunci、广播后的标量或指针，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 837-838
```mlir
837| // COMMON:  }
838| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 839-839
```mlir
839| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 840-841
```mlir
840| 
841| #blocked = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [64], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 842-843
```mlir
842| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
843|   // COMMON-LABEL: buffer_atomic_cas_i64
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 844-861
```mlir
844|   tt.func public @buffer_atomic_cas_i64(%arg0: !tt.ptr<i64> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32} , %arg1: !tt.ptr<i64> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
845|     // COMMON: %[[val:.*]] = arith.constant dense<2>
846|     %cst = arith.constant dense<2> : tensor<1024xi64, #blocked>
847|     // COMMON: %[[cmp:.*]] = arith.constant dense<0>
848|     %cst_0 = arith.constant dense<0> : tensor<1024xi64, #blocked>
849|     %c1024_i32 = arith.constant 1024 : i32
850|     %0 = tt.get_program_id x : i32
851|     %1 = arith.muli %0, %c1024_i32 : i32
852|     // COMMON: %[[offset:.*]] = tt.make_range
853|     %2 = tt.make_range {end = 1024 : i32, start = 0 : i32} : tensor<1024xi32, #blocked>
854|     // COMMON: %[[scalar_ptr:.*]] = tt.addptr %arg0
855|     %3 = tt.addptr %arg0, %1 : !tt.ptr<i64>, i32
856|     %4 = tt.splat %3 : !tt.ptr<i64> -> tensor<1024x!tt.ptr<i64>, #blocked>
857|     %5 = tt.addptr %4, %2 : tensor<1024x!tt.ptr<i64>, #blocked>, tensor<1024xi32, #blocked>
858|     // COMMON: amdg.buffer_atomic_cas acq_rel, gpu, %[[cmp]], %[[val]], %[[scalar_ptr]][%[[offset]]]
859|     %6 = tt.atomic_cas acq_rel, gpu, %5, %cst_0, %cst : (tensor<1024x!tt.ptr<i64>, #blocked>, tensor<1024xi64, #blocked>, tensor<1024xi64, #blocked>) -> tensor<1024xi64, #blocked>
860|     %7 = tt.addptr %arg1, %1 : !tt.ptr<i64>, i32
861|     %8 = tt.splat %7 : !tt.ptr<i64> -> tensor<1024x!tt.ptr<i64>, #blocked>
```
**EN:** This function-oriented block defines or enters `buffer_atomic_cas_i64`. Within it, the test exercises constants, pointer arithmetic, tt.func, broadcasted scalars or pointers, program IDs, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `buffer_atomic_cas_i64` 为核心。测试在其中演示 常量、指针算术、tt.func、广播后的标量或指针、程序 ID，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 862-867
```mlir
862|     %9 = tt.addptr %8, %2 : tensor<1024x!tt.ptr<i64>, #blocked>, tensor<1024xi32, #blocked>
863|     tt.store %9, %6 : tensor<1024x!tt.ptr<i64>, #blocked>
864|     tt.return
865|   }
866| }
867| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on pointer arithmetic, masked or vectorized stores, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 指针算术、带掩码或向量化的存储、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 868-868
```mlir
868| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 869-873
```mlir
869| 
870| // The following two regression tests (all_false_mask and all_true_mask) are to
871| // make sure that a buffer-op does not have to take mask-operand if and only if
872| // its mask operand is a all-true-predicate.
873| //
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 874-874
```mlir
874| #blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [64], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 875-876
```mlir
875| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
876|   // COMMON-LABEL: all_false_mask
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 877-894
```mlir
877|   tt.func public @all_false_mask(%in_ptr: !tt.ptr<f32> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
878|                                  %idx_ptr: !tt.ptr<i64> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
879|                                  %out_ptr: !tt.ptr<f32> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
880|                                  %shape0: i32, %shape1: i32) {
881|     %cst = arith.constant dense<false> : tensor<64xi1, #blocked>
882|     %c64_i32 = arith.constant 64 : i32
883|     %0 = tt.get_program_id x : i32
884|     %1 = arith.muli %0, %c64_i32 : i32
885|     %2 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #blocked>
886|     %3 = tt.splat %1 : i32 -> tensor<64xi32, #blocked>
887|     %4 = arith.addi %3, %2 : tensor<64xi32, #blocked>
888|     %5 = tt.splat %shape1 : i32 -> tensor<64xi32, #blocked>
889|     %6 = arith.divsi %4, %5 : tensor<64xi32, #blocked>
890|     %7 = arith.muli %5, %6 : tensor<64xi32, #blocked>
891|     %8 = tt.addptr %idx_ptr, %1 : !tt.ptr<i64>, i32
892|     %9 = tt.splat %8 : !tt.ptr<i64> -> tensor<64x!tt.ptr<i64>, #blocked>
893|     %10 = tt.addptr %9, %2 : tensor<64x!tt.ptr<i64>, #blocked>, tensor<64xi32, #blocked>
894|     %11 = tt.load %10, %cst : tensor<64x!tt.ptr<i64>, #blocked>
```
**EN:** This function-oriented block defines or enters `all_false_mask`. Within it, the test exercises broadcasted scalars or pointers, tt.func, constants, integer multiplications, pointer arithmetic, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `all_false_mask` 为核心。测试在其中演示 广播后的标量或指针、tt.func、常量、整数乘法、指针算术，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 895-910
```mlir
895|     // COMMON: amdg.buffer_load %[[ptr1:.*]][%[[ofst1:.*]]], %[[mask1:.*]] : tensor<64xi64, #blocked>
896|     %12 = tt.addptr %in_ptr, %1 : !tt.ptr<f32>, i32
897|     %13 = tt.splat %12 : !tt.ptr<f32> -> tensor<64x!tt.ptr<f32>, #blocked>
898|     %14 = tt.addptr %13, %2 : tensor<64x!tt.ptr<f32>, #blocked>, tensor<64xi32, #blocked>
899|     %15 = tt.load %14, %cst : tensor<64x!tt.ptr<f32>, #blocked>
900|     // COMMON: amdg.buffer_load %[[ptr2:.*]][%[[ofst2:.*]]], %[[mask2:.*]] : tensor<64xf32, #blocked>
901|     %16 = arith.extsi %7 : tensor<64xi32, #blocked> to tensor<64xi64, #blocked>
902|     %17 = arith.addi %11, %16 : tensor<64xi64, #blocked>
903|     %18 = arith.trunci %17 : tensor<64xi64, #blocked> to tensor<64xi32, #blocked>
904|     %19 = tt.splat %out_ptr : !tt.ptr<f32> -> tensor<64x!tt.ptr<f32>, #blocked>
905|     %20 = tt.addptr %19, %18 : tensor<64x!tt.ptr<f32>, #blocked>, tensor<64xi32, #blocked>
906|     %21 = tt.atomic_rmw fadd, relaxed, gpu, %20, %15, %cst : (tensor<64x!tt.ptr<f32>, #blocked>, tensor<64xf32, #blocked>, tensor<64xi1, #blocked>) -> tensor<64xf32, #blocked>
907|     tt.return
908|   }
909| }
910| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on pointer arithmetic, broadcasted scalars or pointers, masked or vectorized loads, arith.extsi, integer additions. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 指针算术、广播后的标量或指针、带掩码或向量化的加载、arith.extsi、整数加法。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 911-911
```mlir
911| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 912-913
```mlir
912| 
913| #blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [64], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 914-915
```mlir
914| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
915|   // COMMON-LABEL: all_true_mask
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 916-933
```mlir
916|   tt.func public @all_true_mask(%in_ptr: !tt.ptr<f32> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
917|                                 %idx_ptr: !tt.ptr<i64> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
918|                                 %out_ptr: !tt.ptr<f32> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
919|                                 %shape0: i32, %shape1: i32) {
920|     %cst = arith.constant dense<true> : tensor<64xi1, #blocked>
921|     %c64_i32 = arith.constant 64 : i32
922|     %0 = tt.get_program_id x : i32
923|     %1 = arith.muli %0, %c64_i32 : i32
924|     %2 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #blocked>
925|     %3 = tt.splat %1 : i32 -> tensor<64xi32, #blocked>
926|     %4 = arith.addi %3, %2 : tensor<64xi32, #blocked>
927|     %5 = tt.splat %shape1 : i32 -> tensor<64xi32, #blocked>
928|     %6 = arith.divsi %4, %5 : tensor<64xi32, #blocked>
929|     %7 = arith.muli %5, %6 : tensor<64xi32, #blocked>
930|     %8 = tt.addptr %idx_ptr, %1 : !tt.ptr<i64>, i32
931|     %9 = tt.splat %8 : !tt.ptr<i64> -> tensor<64x!tt.ptr<i64>, #blocked>
932|     %10 = tt.addptr %9, %2 : tensor<64x!tt.ptr<i64>, #blocked>, tensor<64xi32, #blocked>
933|     %11 = tt.load %10, %cst : tensor<64x!tt.ptr<i64>, #blocked>
```
**EN:** This function-oriented block defines or enters `all_true_mask`. Within it, the test exercises broadcasted scalars or pointers, tt.func, constants, integer multiplications, pointer arithmetic, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `all_true_mask` 为核心。测试在其中演示 广播后的标量或指针、tt.func、常量、整数乘法、指针算术，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 934-948
```mlir
934|     // COMMON: amdg.buffer_load %[[ptr1:.*]][%[[ofst1:.*]]] : tensor<64xi64, #blocked>
935|     %12 = tt.addptr %in_ptr, %1 : !tt.ptr<f32>, i32
936|     %13 = tt.splat %12 : !tt.ptr<f32> -> tensor<64x!tt.ptr<f32>, #blocked>
937|     %14 = tt.addptr %13, %2 : tensor<64x!tt.ptr<f32>, #blocked>, tensor<64xi32, #blocked>
938|     %15 = tt.load %14, %cst : tensor<64x!tt.ptr<f32>, #blocked>
939|     // COMMON: amdg.buffer_load %[[ptr2:.*]][%[[ofst2:.*]]] : tensor<64xf32, #blocked>
940|     %16 = arith.extsi %7 : tensor<64xi32, #blocked> to tensor<64xi64, #blocked>
941|     %17 = arith.addi %11, %16 : tensor<64xi64, #blocked>
942|     %18 = arith.trunci %17 : tensor<64xi64, #blocked> to tensor<64xi32, #blocked>
943|     %19 = tt.splat %out_ptr : !tt.ptr<f32> -> tensor<64x!tt.ptr<f32>, #blocked>
944|     %20 = tt.addptr %19, %18 : tensor<64x!tt.ptr<f32>, #blocked>, tensor<64xi32, #blocked>
945|     %21 = tt.atomic_rmw fadd, relaxed, gpu, %20, %15, %cst : (tensor<64x!tt.ptr<f32>, #blocked>, tensor<64xf32, #blocked>, tensor<64xi1, #blocked>) -> tensor<64xf32, #blocked>
946|     tt.return
947|   }
948| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on pointer arithmetic, broadcasted scalars or pointers, masked or vectorized loads, arith.extsi, integer additions. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 指针算术、广播后的标量或指针、带掩码或向量化的加载、arith.extsi、整数加法。这些行共同表达了 pass 流水线必须保持或重写的行为。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `--tritonamdgpu-convert-buffer-ops="gfx-arch=gfx942`, `--check-prefixes=COMMON,GFX942-ONLY,CDNA`, `--tritonamdgpu-convert-buffer-ops="gfx-arch=gfx950`, `--check-prefixes=COMMON,GFX950-PLUS,CDNA`, `--tritonamdgpu-convert-buffer-ops="gfx-arch=gfx1250`, `--check-prefixes=COMMON,GFX950-PLUS`
- **CN:** 主要 pass 选项：`-split-input-file`，`--tritonamdgpu-convert-buffer-ops="gfx-arch=gfx942`，`--check-prefixes=COMMON,GFX942-ONLY,CDNA`，`--tritonamdgpu-convert-buffer-ops="gfx-arch=gfx950`，`--check-prefixes=COMMON,GFX950-PLUS,CDNA`，`--tritonamdgpu-convert-buffer-ops="gfx-arch=gfx1250`，`--check-prefixes=COMMON,GFX950-PLUS`
- **EN:** Dominant operations include `tt.splat`, `tt.addptr`, `arith.constant`, `tt.func`, `tt.make_range`, `arith.addi`, `module`, `tt.load`, `tt.return`, `arith.cmpi`.
- **CN:** 主要操作包括 `tt.splat`、`tt.addptr`、`arith.constant`、`tt.func`、`tt.make_range`、`arith.addi`、`module`、`tt.load`、`tt.return`、`arith.cmpi`。
- **EN:** The file contains 25 independently testable section(s). The file relies mainly on the transformed IR itself rather than explicit FileCheck/diagnostic annotations. It validates the semantics or lowering pattern expressed by the IR in this test.
- **CN:** 该文件包含 25 个可独立测试的分段。该文件主要依赖变换后的 IR 本身，而不是显式的 FileCheck/诊断注解。 它验证该测试中 IR 所表达的语义或 lower 模式。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `tt`: Triton high-level tensor/pointer operations.
- **CN:** `tt`：Triton 高层张量/指针操作。
- **EN:** `arith`: MLIR arithmetic dialect for scalar/tensor math.
- **CN:** `arith`：用于标量/张量运算的 MLIR arithmetic 方言。
- **EN:** `ttg`: TritonGPU layout, memory, and GPU-specific operations.
- **CN:** `ttg`：TritonGPU 布局、内存和 GPU 专用操作。
- **EN:** `llvm`: LLVM-style low-level operations and types.
- **CN:** `llvm`：LLVM 风格底层操作与类型。
- **EN:** `scf`: Structured control flow for loops and conditionals.
- **CN:** `scf`：用于循环与条件分支的结构化控制流。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。