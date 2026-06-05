# tmem_barrier_insertion.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonNvidiaGPU/tmem_barrier_insertion.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-triton-nvidia-gpu-tmem-barrier-insertion` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-triton-nvidia-gpu-tmem-barrier-insertion` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -triton-nvidia-gpu-tmem-barrier-insertion | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -triton-nvidia-gpu-tmem-barrier-insertion | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt %s -triton-nvidia-gpu-tmem-barrier-insertion | FileCheck %s
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -triton-nvidia-gpu-tmem-barrier-insertion | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -triton-nvidia-gpu-tmem-barrier-insertion | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-12
```mlir
 3| #shared_a = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = false, elementBitWidth = 16}>
 4| #shared_b = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = true, elementBitWidth = 16}>
 5| #shared_copy = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
 6| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
 7| #blocked_scales = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [1, 0]}>
 8| #linear64 = #ttg.linear<{register = [[0, 1], [0, 2], [0, 4], [0, 8], [0, 16], [0, 32]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 64]], warp = [[16, 0], [32, 0]], block = []}>
 9| #tmem128 = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
10| #tmem64 = #ttng.tensor_memory_encoding<blockM = 64, blockN = 128, colStride = 1>
11| #tmem_scales = #ttng.tensor_memory_scales_encoding<>
12| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 13-13
```mlir
13| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 14-17
```mlir
14|   // CHECK-LABEL: @alloc_then_alloc
15|   // CHECK: ttng.tmem_alloc
16|   // CHECK-NEXT: ttg.barrier local
17|   // CHECK-NEXT: ttng.tmem_alloc
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @alloc_then_alloc anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @alloc_then_alloc 这样的标签用于锚定匹配范围。

### Lines 18-23
```mlir
18|   tt.func @alloc_then_alloc(%arg0: tensor<128x128xf32, #blocked>) {
19|     %0 = ttng.tmem_alloc %arg0 {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : (tensor<128x128xf32, #blocked>) -> !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
20|     %1 = ttng.tmem_alloc %arg0 {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : (tensor<128x128xf32, #blocked>) -> !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
21|     tt.return
22|   }
23| 
```
**EN:** This function-oriented block defines or enters `alloc_then_alloc`. Within it, the test exercises tt.func, tensor-memory allocation, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `alloc_then_alloc` 为核心。测试在其中演示 tt.func、张量内存分配、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 24-27
```mlir
24|   // CHECK-LABEL: @alloc_then_ld
25|   // CHECK: ttng.tmem_alloc
26|   // CHECK-NEXT: ttg.barrier local
27|   // CHECK-NEXT: ttng.tmem_load
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @alloc_then_ld anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @alloc_then_ld 这样的标签用于锚定匹配范围。

### Lines 28-33
```mlir
28|   tt.func @alloc_then_ld(%arg0: tensor<128x128xf32, #blocked>) {
29|     %0 = ttng.tmem_alloc %arg0 {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : (tensor<128x128xf32, #blocked>) -> !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
30|     %1 = ttng.tmem_load %0 : !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
31|     tt.return
32|   }
33| 
```
**EN:** This function-oriented block defines or enters `alloc_then_ld`. Within it, the test exercises tt.func, tensor-memory allocation, ttng.tmem_load, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `alloc_then_ld` 为核心。测试在其中演示 tt.func、张量内存分配、ttng.tmem_load、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 34-37
```mlir
34|   // CHECK-LABEL: @alloc_then_st
35|   // CHECK: ttng.tmem_alloc
36|   // CHECK-NEXT: ttg.barrier local
37|   // CHECK-NEXT: ttng.tmem_store
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @alloc_then_st anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @alloc_then_st 这样的标签用于锚定匹配范围。

### Lines 38-44
```mlir
38|   tt.func @alloc_then_st(%arg0: tensor<128x128xf32, #blocked>) {
39|     %true = arith.constant true
40|     %0 = ttng.tmem_alloc %arg0 {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : (tensor<128x128xf32, #blocked>) -> !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
41|     ttng.tmem_store %arg0, %0, %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
42|     tt.return
43|   }
44| 
```
**EN:** This function-oriented block defines or enters `alloc_then_st`. Within it, the test exercises tt.func, constants, tensor-memory allocation, ttng.tmem_store, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `alloc_then_st` 为核心。测试在其中演示 tt.func、常量、张量内存分配、ttng.tmem_store、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 45-48
```mlir
45|   // CHECK-LABEL: @alloc_then_mma
46|   // CHECK: ttng.tmem_alloc
47|   // CHECK-NEXT: ttg.barrier local
48|   // CHECK-NEXT: ttng.tc_gen5_mma
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @alloc_then_mma anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @alloc_then_mma 这样的标签用于锚定匹配范围。

### Lines 49-61
```mlir
49|   tt.func @alloc_then_mma(%arg0: tensor<128x128xf32, #blocked>,
50|                           %arg1: !ttg.memdesc<128x128xf16, #shared_a, #ttg.shared_memory>,
51|                           %arg2: !ttg.memdesc<128x128xf16, #shared_b, #ttg.shared_memory>) {
52|     %false = arith.constant false
53|     %true = arith.constant true
54|     %0 = ttng.tmem_alloc %arg0 {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : (tensor<128x128xf32, #blocked>) -> !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
55|     ttng.tc_gen5_mma %arg1, %arg2, %0, %false, %true :
56|       !ttg.memdesc<128x128xf16, #shared_a, #ttg.shared_memory>,
57|       !ttg.memdesc<128x128xf16, #shared_b, #ttg.shared_memory>,
58|       !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
59|     tt.return
60|   }
61| 
```
**EN:** This function-oriented block defines or enters `alloc_then_mma`. Within it, the test exercises tt.func, constants, tensor-memory allocation, Gen5 tensor-core MMA ops, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `alloc_then_mma` 为核心。测试在其中演示 tt.func、常量、张量内存分配、Gen5 张量核 MMA 操作、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 62-65
```mlir
62|   // CHECK-LABEL: @ld_then_alloc
63|   // CHECK: ttng.tmem_load
64|   // CHECK-NEXT: ttg.barrier local
65|   // CHECK-NEXT: ttng.tmem_alloc
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @ld_then_alloc anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @ld_then_alloc 这样的标签用于锚定匹配范围。

### Lines 66-73
```mlir
66|   tt.func @ld_then_alloc(%arg0: tensor<128x128xf32, #blocked>) {
67|     %0 = ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
68|     ttg.barrier local
69|     %1 = ttng.tmem_load %0 : !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
70|     %2 = ttng.tmem_alloc %arg0 {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : (tensor<128x128xf32, #blocked>) -> !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
71|     tt.return
72|   }
73| 
```
**EN:** This function-oriented block defines or enters `ld_then_alloc`. Within it, the test exercises tt.func, tensor-memory allocation, ttg.barrier, ttng.tmem_load, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `ld_then_alloc` 为核心。测试在其中演示 tt.func、张量内存分配、ttg.barrier、ttng.tmem_load、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 74-76
```mlir
74|   // CHECK-LABEL: @ld_then_ld
75|   // CHECK: ttng.tmem_load
76|   // CHECK-NEXT: ttng.tmem_load
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @ld_then_ld anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @ld_then_ld 这样的标签用于锚定匹配范围。

### Lines 77-84
```mlir
77|   tt.func @ld_then_ld() {
78|     %0 = ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
79|     ttg.barrier local
80|     %1 = ttng.tmem_load %0 : !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
81|     %2 = ttng.tmem_load %0 : !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
82|     tt.return
83|   }
84| 
```
**EN:** This function-oriented block defines or enters `ld_then_ld`. Within it, the test exercises tt.func, ttng.tmem_load, tensor-memory allocation, ttg.barrier, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `ld_then_ld` 为核心。测试在其中演示 tt.func、ttng.tmem_load、张量内存分配、ttg.barrier、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 85-88
```mlir
85|   // CHECK-LABEL: @ld_then_st
86|   // CHECK: ttng.tmem_load
87|   // CHECK-NEXT: ttg.barrier local
88|   // CHECK-NEXT: ttng.tmem_store
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @ld_then_st anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @ld_then_st 这样的标签用于锚定匹配范围。

### Lines 89-97
```mlir
89|   tt.func @ld_then_st(%arg0: tensor<128x128xf32, #blocked>) {
90|     %true = arith.constant true
91|     %0 = ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
92|     ttg.barrier local
93|     %1 = ttng.tmem_load %0 : !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
94|     ttng.tmem_store %arg0, %0, %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
95|     tt.return
96|   }
97| 
```
**EN:** This function-oriented block defines or enters `ld_then_st`. Within it, the test exercises tt.func, constants, tensor-memory allocation, ttg.barrier, ttng.tmem_load, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `ld_then_st` 为核心。测试在其中演示 tt.func、常量、张量内存分配、ttg.barrier、ttng.tmem_load，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 98-101
```mlir
 98|   // CHECK-LABEL: @ld_then_mma
 99|   // CHECK: ttng.tmem_load
100|   // CHECK-NEXT: ttg.barrier local
101|   // CHECK-NEXT: ttng.tc_gen5_mma
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @ld_then_mma anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @ld_then_mma 这样的标签用于锚定匹配范围。

### Lines 102-115
```mlir
102|   tt.func @ld_then_mma(%arg0: !ttg.memdesc<128x128xf16, #shared_a, #ttg.shared_memory>,
103|                        %arg1: !ttg.memdesc<128x128xf16, #shared_b, #ttg.shared_memory>) {
104|     %false = arith.constant false
105|     %true = arith.constant true
106|     %0 = ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
107|     ttg.barrier local
108|     %1 = ttng.tmem_load %0 : !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
109|     ttng.tc_gen5_mma %arg0, %arg1, %0, %false, %true :
110|       !ttg.memdesc<128x128xf16, #shared_a, #ttg.shared_memory>,
111|       !ttg.memdesc<128x128xf16, #shared_b, #ttg.shared_memory>,
112|       !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
113|     tt.return
114|   }
115| 
```
**EN:** This function-oriented block defines or enters `ld_then_mma`. Within it, the test exercises tt.func, constants, tensor-memory allocation, ttg.barrier, ttng.tmem_load, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `ld_then_mma` 为核心。测试在其中演示 tt.func、常量、张量内存分配、ttg.barrier、ttng.tmem_load，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 116-118
```mlir
116|   // CHECK-LABEL: @st_then_alloc
117|   // CHECK: ttng.tmem_store
118|   // CHECK-NEXT: ttng.tmem_alloc
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @st_then_alloc anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @st_then_alloc 这样的标签用于锚定匹配范围。

### Lines 119-127
```mlir
119|   tt.func @st_then_alloc(%arg0: tensor<128x128xf32, #blocked>) {
120|     %true = arith.constant true
121|     %0 = ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
122|     ttg.barrier local
123|     ttng.tmem_store %arg0, %0, %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
124|     %1 = ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
125|     tt.return
126|   }
127| 
```
**EN:** This function-oriented block defines or enters `st_then_alloc`. Within it, the test exercises tt.func, tensor-memory allocation, constants, ttg.barrier, ttng.tmem_store, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `st_then_alloc` 为核心。测试在其中演示 tt.func、张量内存分配、常量、ttg.barrier、ttng.tmem_store，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 128-131
```mlir
128|   // CHECK-LABEL: @st_then_ld
129|   // CHECK: ttng.tmem_store
130|   // CHECK-NEXT: ttg.barrier local
131|   // CHECK-NEXT: ttng.tmem_load
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @st_then_ld anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @st_then_ld 这样的标签用于锚定匹配范围。

### Lines 132-140
```mlir
132|   tt.func @st_then_ld(%arg0: tensor<128x128xf32, #blocked>) {
133|     %true = arith.constant true
134|     %0 = ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
135|     ttg.barrier local
136|     ttng.tmem_store %arg0, %0, %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
137|     %1 = ttng.tmem_load %0 : !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
138|     tt.return
139|   }
140| 
```
**EN:** This function-oriented block defines or enters `st_then_ld`. Within it, the test exercises tt.func, constants, tensor-memory allocation, ttg.barrier, ttng.tmem_store, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `st_then_ld` 为核心。测试在其中演示 tt.func、常量、张量内存分配、ttg.barrier、ttng.tmem_store，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 141-144
```mlir
141|   // CHECK-LABEL: @st_then_st
142|   // CHECK: ttng.tmem_store
143|   // CHECK-NEXT: ttg.barrier local
144|   // CHECK-NEXT: ttng.tmem_store
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @st_then_st anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @st_then_st 这样的标签用于锚定匹配范围。

### Lines 145-153
```mlir
145|   tt.func @st_then_st(%arg0: tensor<128x128xf32, #blocked>) {
146|     %true = arith.constant true
147|     %0 = ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
148|     ttg.barrier local
149|     ttng.tmem_store %arg0, %0, %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
150|     ttng.tmem_store %arg0, %0, %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
151|     tt.return
152|   }
153| 
```
**EN:** This function-oriented block defines or enters `st_then_st`. Within it, the test exercises tt.func, ttng.tmem_store, constants, tensor-memory allocation, ttg.barrier, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `st_then_st` 为核心。测试在其中演示 tt.func、ttng.tmem_store、常量、张量内存分配、ttg.barrier，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 154-157
```mlir
154|   // CHECK-LABEL: @st_then_mma
155|   // CHECK: ttng.tmem_store
156|   // CHECK-NEXT: ttg.barrier local
157|   // CHECK-NEXT: ttng.tc_gen5_mma
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @st_then_mma anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @st_then_mma 这样的标签用于锚定匹配范围。

### Lines 158-172
```mlir
158|   tt.func @st_then_mma(%arg0: tensor<128x128xf32, #blocked>,
159|                        %arg1: !ttg.memdesc<128x128xf16, #shared_a, #ttg.shared_memory>,
160|                        %arg2: !ttg.memdesc<128x128xf16, #shared_b, #ttg.shared_memory>) {
161|     %false = arith.constant false
162|     %true = arith.constant true
163|     %0 = ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
164|     ttg.barrier local
165|     ttng.tmem_store %arg0, %0, %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
166|     ttng.tc_gen5_mma %arg1, %arg2, %0, %false, %true :
167|       !ttg.memdesc<128x128xf16, #shared_a, #ttg.shared_memory>,
168|       !ttg.memdesc<128x128xf16, #shared_b, #ttg.shared_memory>,
169|       !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
170|     tt.return
171|   }
172| 
```
**EN:** This function-oriented block defines or enters `st_then_mma`. Within it, the test exercises tt.func, constants, tensor-memory allocation, ttg.barrier, ttng.tmem_store, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `st_then_mma` 为核心。测试在其中演示 tt.func、常量、张量内存分配、ttg.barrier、ttng.tmem_store，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 173-175
```mlir
173|   // CHECK-LABEL: @mma_then_alloc
174|   // CHECK: ttng.tc_gen5_mma
175|   // CHECK-NEXT: ttng.tmem_alloc
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @mma_then_alloc anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @mma_then_alloc 这样的标签用于锚定匹配范围。

### Lines 176-189
```mlir
176|   tt.func @mma_then_alloc(%arg0: !ttg.memdesc<128x128xf16, #shared_a, #ttg.shared_memory>,
177|                           %arg1: !ttg.memdesc<128x128xf16, #shared_b, #ttg.shared_memory>) {
178|     %false = arith.constant false
179|     %true = arith.constant true
180|     %0 = ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
181|     ttg.barrier local
182|     ttng.tc_gen5_mma %arg0, %arg1, %0, %false, %true :
183|       !ttg.memdesc<128x128xf16, #shared_a, #ttg.shared_memory>,
184|       !ttg.memdesc<128x128xf16, #shared_b, #ttg.shared_memory>,
185|       !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
186|     %1 = ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
187|     tt.return
188|   }
189| 
```
**EN:** This function-oriented block defines or enters `mma_then_alloc`. Within it, the test exercises tt.func, constants, tensor-memory allocation, ttg.barrier, Gen5 tensor-core MMA ops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mma_then_alloc` 为核心。测试在其中演示 tt.func、常量、张量内存分配、ttg.barrier、Gen5 张量核 MMA 操作，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 190-192
```mlir
190|   // CHECK-LABEL: @mma_then_ld
191|   // CHECK: ttng.tc_gen5_mma
192|   // CHECK-NEXT: ttng.tmem_load
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @mma_then_ld anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @mma_then_ld 这样的标签用于锚定匹配范围。

### Lines 193-206
```mlir
193|   tt.func @mma_then_ld(%arg0: !ttg.memdesc<128x128xf16, #shared_a, #ttg.shared_memory>,
194|                        %arg1: !ttg.memdesc<128x128xf16, #shared_b, #ttg.shared_memory>) {
195|     %false = arith.constant false
196|     %true = arith.constant true
197|     %0 = ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
198|     ttg.barrier local
199|     ttng.tc_gen5_mma %arg0, %arg1, %0, %false, %true :
200|       !ttg.memdesc<128x128xf16, #shared_a, #ttg.shared_memory>,
201|       !ttg.memdesc<128x128xf16, #shared_b, #ttg.shared_memory>,
202|       !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
203|     %1 = ttng.tmem_load %0 : !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
204|     tt.return
205|   }
206| 
```
**EN:** This function-oriented block defines or enters `mma_then_ld`. Within it, the test exercises tt.func, constants, tensor-memory allocation, ttg.barrier, Gen5 tensor-core MMA ops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mma_then_ld` 为核心。测试在其中演示 tt.func、常量、张量内存分配、ttg.barrier、Gen5 张量核 MMA 操作，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 207-209
```mlir
207|   // CHECK-LABEL: @mma_then_st
208|   // CHECK: ttng.tc_gen5_mma
209|   // CHECK-NEXT: ttng.tmem_store
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @mma_then_st anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @mma_then_st 这样的标签用于锚定匹配范围。

### Lines 210-224
```mlir
210|   tt.func @mma_then_st(%arg0: tensor<128x128xf32, #blocked>,
211|                        %arg1: !ttg.memdesc<128x128xf16, #shared_a, #ttg.shared_memory>,
212|                        %arg2: !ttg.memdesc<128x128xf16, #shared_b, #ttg.shared_memory>) {
213|     %false = arith.constant false
214|     %true = arith.constant true
215|     %0 = ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
216|     ttg.barrier local
217|     ttng.tc_gen5_mma %arg1, %arg2, %0, %false, %true :
218|       !ttg.memdesc<128x128xf16, #shared_a, #ttg.shared_memory>,
219|       !ttg.memdesc<128x128xf16, #shared_b, #ttg.shared_memory>,
220|       !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
221|     ttng.tmem_store %arg0, %0, %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
222|     tt.return
223|   }
224| 
```
**EN:** This function-oriented block defines or enters `mma_then_st`. Within it, the test exercises tt.func, constants, tensor-memory allocation, ttg.barrier, Gen5 tensor-core MMA ops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mma_then_st` 为核心。测试在其中演示 tt.func、常量、张量内存分配、ttg.barrier、Gen5 张量核 MMA 操作，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 225-227
```mlir
225|   // CHECK-LABEL: @mma_then_mma
226|   // CHECK: ttng.tc_gen5_mma
227|   // CHECK-NEXT: ttng.tc_gen5_mma
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @mma_then_mma anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @mma_then_mma 这样的标签用于锚定匹配范围。

### Lines 228-244
```mlir
228|   tt.func @mma_then_mma(%arg0: !ttg.memdesc<128x128xf16, #shared_a, #ttg.shared_memory>,
229|                         %arg1: !ttg.memdesc<128x128xf16, #shared_b, #ttg.shared_memory>) {
230|     %false = arith.constant false
231|     %true = arith.constant true
232|     %0 = ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
233|     ttg.barrier local
234|     ttng.tc_gen5_mma %arg0, %arg1, %0, %false, %true :
235|       !ttg.memdesc<128x128xf16, #shared_a, #ttg.shared_memory>,
236|       !ttg.memdesc<128x128xf16, #shared_b, #ttg.shared_memory>,
237|       !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
238|     ttng.tc_gen5_mma %arg0, %arg1, %0, %false, %true :
239|       !ttg.memdesc<128x128xf16, #shared_a, #ttg.shared_memory>,
240|       !ttg.memdesc<128x128xf16, #shared_b, #ttg.shared_memory>,
241|       !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
242|     tt.return
243|   }
244| 
```
**EN:** This function-oriented block defines or enters `mma_then_mma`. Within it, the test exercises tt.func, constants, Gen5 tensor-core MMA ops, tensor-memory allocation, ttg.barrier, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mma_then_mma` 为核心。测试在其中演示 tt.func、常量、Gen5 张量核 MMA 操作、张量内存分配、ttg.barrier，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 245-247
```mlir
245|   // CHECK-LABEL: @ld_then_st_non_aliasing
246|   // CHECK: ttng.tmem_load
247|   // CHECK-NEXT: ttng.tmem_store
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @ld_then_st_non_aliasing anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @ld_then_st_non_aliasing 这样的标签用于锚定匹配范围。

### Lines 248-257
```mlir
248|   tt.func @ld_then_st_non_aliasing(%arg0: tensor<128x128xf32, #blocked>) {
249|     %true = arith.constant true
250|     %0 = ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
251|     %2 = ttng.tmem_alloc {tensor_memory_col_offset = 128 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
252|     ttg.barrier local
253|     %1 = ttng.tmem_load %0 : !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
254|     ttng.tmem_store %arg0, %2, %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
255|     tt.return
256|   }
257| 
```
**EN:** This function-oriented block defines or enters `ld_then_st_non_aliasing`. Within it, the test exercises tt.func, tensor-memory allocation, constants, ttg.barrier, ttng.tmem_load, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `ld_then_st_non_aliasing` 为核心。测试在其中演示 tt.func、张量内存分配、常量、ttg.barrier、ttng.tmem_load，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 258-262
```mlir
258|   // CHECK-LABEL: @ld_then_alloc_then_st_aliases_second_row
259|   // CHECK: ttng.tmem_load
260|   // CHECK-NEXT: ttng.tmem_alloc
261|   // CHECK-NEXT: ttg.barrier local
262|   // CHECK-NEXT: ttng.tmem_store
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @ld_then_alloc_then_st_aliases_second_row anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @ld_then_alloc_then_st_aliases_second_row 这样的标签用于锚定匹配范围。

### Lines 263-272
```mlir
263|   tt.func @ld_then_alloc_then_st_aliases_second_row(%arg0: tensor<64x128xf32, #linear64>) {
264|     %true = arith.constant true
265|     %0 = ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
266|     ttg.barrier local
267|     %1 = ttng.tmem_load %0 : !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
268|     %2 = ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 16 : i32} : () -> !ttg.memdesc<64x128xf32, #tmem64, #ttng.tensor_memory, mutable>
269|     ttng.tmem_store %arg0, %2, %true : tensor<64x128xf32, #linear64> -> !ttg.memdesc<64x128xf32, #tmem64, #ttng.tensor_memory, mutable>
270|     tt.return
271|   }
272| 
```
**EN:** This function-oriented block defines or enters `ld_then_alloc_then_st_aliases_second_row`. Within it, the test exercises tt.func, tensor-memory allocation, constants, ttg.barrier, ttng.tmem_load, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `ld_then_alloc_then_st_aliases_second_row` 为核心。测试在其中演示 tt.func、张量内存分配、常量、ttg.barrier、ttng.tmem_load，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 273-276
```mlir
273|   // CHECK-LABEL: @alloc_then_alloc_partial_overlap
274|   // CHECK: ttng.tmem_alloc
275|   // CHECK-NEXT: ttg.barrier local
276|   // CHECK-NEXT: ttng.tmem_alloc
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @alloc_then_alloc_partial_overlap anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @alloc_then_alloc_partial_overlap 这样的标签用于锚定匹配范围。

### Lines 277-282
```mlir
277|   tt.func @alloc_then_alloc_partial_overlap(%arg0: tensor<128x128xf32, #blocked>) {
278|     %0 = ttng.tmem_alloc %arg0 {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : (tensor<128x128xf32, #blocked>) -> !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
279|     %1 = ttng.tmem_alloc %arg0 {tensor_memory_col_offset = 64 : i32, tensor_memory_row_offset = 0 : i32} : (tensor<128x128xf32, #blocked>) -> !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
280|     tt.return
281|   }
282| 
```
**EN:** This function-oriented block defines or enters `alloc_then_alloc_partial_overlap`. Within it, the test exercises tt.func, tensor-memory allocation, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `alloc_then_alloc_partial_overlap` 为核心。测试在其中演示 tt.func、张量内存分配、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 283-286
```mlir
283|   // CHECK-LABEL: @st_then_mma_scaled_scale_operand
284|   // CHECK: ttng.tmem_store
285|   // CHECK-NEXT: ttg.barrier local
286|   // CHECK-NEXT: ttng.tc_gen5_mma_scaled
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @st_then_mma_scaled_scale_operand anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @st_then_mma_scaled_scale_operand 这样的标签用于锚定匹配范围。

### Lines 287-304
```mlir
287|   tt.func @st_then_mma_scaled_scale_operand(
288|       %arg0: tensor<128x1xi8, #blocked_scales>,
289|       %arg1: !ttg.memdesc<128x128xf16, #shared_a, #ttg.shared_memory>,
290|       %arg2: !ttg.memdesc<128x128xf16, #shared_b, #ttg.shared_memory>,
291|       %arg3: !ttg.memdesc<64x1xi8, #tmem_scales, #ttng.tensor_memory>) {
292|     %true = arith.constant true
293|     %d = ttng.tmem_alloc {tensor_memory_col_offset = 64 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
294|     %a_scale = ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<128x1xi8, #tmem_scales, #ttng.tensor_memory, mutable>
295|     ttg.barrier local
296|     ttng.tmem_store %arg0, %a_scale, %true : tensor<128x1xi8, #blocked_scales> -> !ttg.memdesc<128x1xi8, #tmem_scales, #ttng.tensor_memory, mutable>
297|     ttng.tc_gen5_mma_scaled %arg1, %arg2, %d, %a_scale, %arg3, %true, %true lhs = e5m2 rhs = e5m2 :
298|       !ttg.memdesc<128x128xf16, #shared_a, #ttg.shared_memory>,
299|       !ttg.memdesc<128x128xf16, #shared_b, #ttg.shared_memory>,
300|       !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>,
301|       !ttg.memdesc<128x1xi8, #tmem_scales, #ttng.tensor_memory, mutable>,
302|       !ttg.memdesc<64x1xi8, #tmem_scales, #ttng.tensor_memory>
303|     tt.return
304|   }
```
**EN:** This function-oriented block defines or enters `st_then_mma_scaled_scale_operand`. Within it, the test exercises tt.func, tensor-memory allocation, constants, ttg.barrier, ttng.tmem_store, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `st_then_mma_scaled_scale_operand` 为核心。测试在其中演示 tt.func、张量内存分配、常量、ttg.barrier、ttng.tmem_store，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 305-309
```mlir
305| 
306|   // CHECK-LABEL: @ld_then_tmem_copy
307|   // CHECK: ttng.tmem_load
308|   // CHECK-NEXT: ttg.barrier local
309|   // CHECK-NEXT: ttng.tmem_copy
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @ld_then_tmem_copy anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @ld_then_tmem_copy 这样的标签用于锚定匹配范围。

### Lines 310-318
```mlir
310|   tt.func @ld_then_tmem_copy(
311|       %arg0: !ttg.memdesc<128x128xf32, #shared_copy, #ttg.shared_memory>) {
312|     %0 = ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
313|     ttg.barrier local
314|     %1 = ttng.tmem_load %0 : !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
315|     ttng.tmem_copy %arg0, %0 : !ttg.memdesc<128x128xf32, #shared_copy, #ttg.shared_memory>, !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
316|     tt.return
317|   }
318| 
```
**EN:** This function-oriented block defines or enters `ld_then_tmem_copy`. Within it, the test exercises tt.func, tensor-memory allocation, ttg.barrier, ttng.tmem_load, tensor-memory copies, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `ld_then_tmem_copy` 为核心。测试在其中演示 tt.func、张量内存分配、ttg.barrier、ttng.tmem_load、张量内存拷贝，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 319-322
```mlir
319|   // CHECK-LABEL: @st_then_tmem_copy
320|   // CHECK: ttng.tmem_store
321|   // CHECK-NEXT: ttg.barrier local
322|   // CHECK-NEXT: ttng.tmem_copy
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @st_then_tmem_copy anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @st_then_tmem_copy 这样的标签用于锚定匹配范围。

### Lines 323-333
```mlir
323|   tt.func @st_then_tmem_copy(
324|       %arg0: tensor<128x128xf32, #blocked>,
325|       %arg1: !ttg.memdesc<128x128xf32, #shared_copy, #ttg.shared_memory>) {
326|     %true = arith.constant true
327|     %0 = ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
328|     ttg.barrier local
329|     ttng.tmem_store %arg0, %0, %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
330|     ttng.tmem_copy %arg1, %0 : !ttg.memdesc<128x128xf32, #shared_copy, #ttg.shared_memory>, !ttg.memdesc<128x128xf32, #tmem128, #ttng.tensor_memory, mutable>
331|     tt.return
332|   }
333| }
```
**EN:** This function-oriented block defines or enters `st_then_tmem_copy`. Within it, the test exercises tt.func, constants, tensor-memory allocation, ttg.barrier, ttng.tmem_store, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `st_then_tmem_copy` 为核心。测试在其中演示 tt.func、常量、张量内存分配、ttg.barrier、ttng.tmem_store，这些操作构成了被优化、lower 或验证的主要数据流。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-triton-nvidia-gpu-tmem-barrier-insertion`
- **CN:** 主要 pass 选项：`-triton-nvidia-gpu-tmem-barrier-insertion`
- **EN:** Dominant operations include `tt.func`, `ttng.tmem_alloc`, `arith.constant`, `tt.return`, `ttg.barrier`, `ttng.tmem_store`, `ttng.tmem_load`, `ttng.tc_gen5_mma`, `ttng.tmem_copy`, `module`.
- **CN:** 主要操作包括 `tt.func`、`ttng.tmem_alloc`、`arith.constant`、`tt.return`、`ttg.barrier`、`ttng.tmem_store`、`ttng.tmem_load`、`ttng.tc_gen5_mma`、`ttng.tmem_copy`、`module`。
- **EN:** The file contains 1 independently testable section(s). Check styles used: CHECK-NEXT x38, CHECK-LABEL x22, CHECK x22. Important labels include @alloc_then_alloc, @alloc_then_ld, @alloc_then_st, @alloc_then_mma. It validates that the selected pass pipeline rewrites each labeled test case into the checked form.
- **CN:** 该文件包含 1 个可独立测试的分段。使用的检查类型：CHECK-NEXT ×38，CHECK-LABEL ×22，CHECK ×22。 关键标签包括 @alloc_then_alloc，@alloc_then_ld，@alloc_then_st，@alloc_then_mma。 它验证所选 pass 流水线会把每个带标签的测试用例重写成检查所要求的形式。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `ttg`: TritonGPU layout, memory, and GPU-specific operations.
- **CN:** `ttg`：TritonGPU 布局、内存和 GPU 专用操作。
- **EN:** `ttng`: Triton Nvidia GPU tensor-memory and MMA operations.
- **CN:** `ttng`：Triton Nvidia GPU 张量内存与 MMA 操作。
- **EN:** `tt`: Triton high-level tensor/pointer operations.
- **CN:** `tt`：Triton 高层张量/指针操作。
- **EN:** `arith`: MLIR arithmetic dialect for scalar/tensor math.
- **CN:** `arith`：用于标量/张量运算的 MLIR arithmetic 方言。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。