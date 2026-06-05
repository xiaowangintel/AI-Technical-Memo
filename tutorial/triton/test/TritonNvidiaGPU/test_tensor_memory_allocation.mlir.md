# test_tensor_memory_allocation.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonNvidiaGPU/test_tensor_memory_allocation.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-triton-tensor-memory-allocation` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-triton-tensor-memory-allocation` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -triton-tensor-memory-allocation | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -triton-tensor-memory-allocation | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -triton-tensor-memory-allocation | FileCheck %s
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file -allow-unregistered-dialect -triton-tensor-memory-allocation | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file -allow-unregistered-dialect -triton-tensor-memory-allocation | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-11
```mlir
 3| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
 4| #blocked1 = #ttg.blocked<{sizePerThread = [1, 32], threadsPerWarp = [16, 2], warpsPerCTA = [4, 1], order = [0, 1]}>
 5| #blocked2 = #ttg.blocked<{sizePerThread = [1, 64], threadsPerWarp = [16, 2], warpsPerCTA = [4, 1], order = [0, 1]}>
 6| #tmem_f32 = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
 7| #tmem_f16 = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 2>
 8| #tmem1 = #ttng.tensor_memory_encoding<blockM = 64, blockN = 64, colStride = 2>
 9| #tmem2 = #ttng.tensor_memory_encoding<blockM = 64, blockN = 128, colStride = 2>
10| #tmem_scales = #ttng.tensor_memory_scales_encoding<>
11| #linear = #ttg.linear<{register = [[0, 1], [0, 2], [32, 0]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0]], warp = [[0, 0], [0, 0]], block = []}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 12-12
```mlir
12| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65536 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 13-14
```mlir
13|   // CHECK: ttg.tensor_memory_size = 512
14|   // CHECK: alloc_tensor_memory
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 15-23
```mlir
15|   tt.func public @alloc_tensor_memory(%arg0: !tt.ptr<f16>, %arg1: !tt.ptr<f16>, %arg2: !tt.ptr<f16>) {
16|     %true = arith.constant true
17|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
18|     %cst0 = arith.constant dense<0.000000e+00> : tensor<128x128xf16, #blocked>
19|     %cst1 = arith.constant dense<0.000000e+00> : tensor<64x64xf16, #blocked1>
20|     %cst2 = arith.constant dense<0.000000e+00> : tensor<64x128xf16, #blocked2>
21|     %cst3 = arith.constant dense<0> : tensor<64x4xi8, #linear>
22|     %cst4 = arith.constant dense<0.000000e+00> : tensor<64x128xf16, #blocked2>
23| 
```
**EN:** This function-oriented block defines or enters `alloc_tensor_memory`. Within it, the test exercises constants, tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `alloc_tensor_memory` 为核心。测试在其中演示 常量、tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 24-25
```mlir
24|     // CHECK: ttng.tmem_alloc %{{.+}} {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32}
25|     %0 = ttng.tmem_alloc %cst : (tensor<128x128xf32, #blocked>) -> !ttg.memdesc<128x128xf32, #tmem_f32, #ttng.tensor_memory, mutable>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 26-27
```mlir
26|     // CHECK: ttng.tmem_alloc %{{.+}} {tensor_memory_col_offset = 128 : i32, tensor_memory_row_offset = 0 : i32}
27|     %1 = ttng.tmem_alloc %cst0 : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #tmem_f16, #ttng.tensor_memory, mutable>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 28-29
```mlir
28|     // CHECK: ttng.tmem_alloc %{{.+}} {tensor_memory_col_offset = 256 : i32, tensor_memory_row_offset = 0 : i32}
29|     %2 = ttng.tmem_alloc %cst1 : (tensor<64x64xf16, #blocked1>) -> !ttg.memdesc<64x64xf16, #tmem1, #ttng.tensor_memory, mutable>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 30-37
```mlir
30|     // CHECK: ttng.tmem_alloc %{{.+}} {tensor_memory_col_offset = 320 : i32, tensor_memory_row_offset = 0 : i32}
31|     %3 = ttng.tmem_alloc %cst : (tensor<128x128xf32, #blocked>) -> !ttg.memdesc<128x128xf32, #tmem_f32, #ttng.tensor_memory, mutable>
32| 
33|     ttng.tmem_store %cst, %0, %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem_f32, #ttng.tensor_memory, mutable>
34|     ttng.tmem_store %cst0, %1, %true : tensor<128x128xf16, #blocked> -> !ttg.memdesc<128x128xf16, #tmem_f16, #ttng.tensor_memory, mutable>
35|     ttng.tmem_store %cst1, %2, %true : tensor<64x64xf16, #blocked1> -> !ttg.memdesc<64x64xf16, #tmem1, #ttng.tensor_memory, mutable>
36|     ttng.tmem_store %cst, %3, %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem_f32, #ttng.tensor_memory, mutable>
37| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_store, tensor-memory allocation. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_store、张量内存分配。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 38-39
```mlir
38|     // CHECK: ttng.tmem_alloc %{{.+}} {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32}
39|     %4 = ttng.tmem_alloc %cst4 : (tensor<64x128xf16, #blocked2>) -> !ttg.memdesc<64x128xf16, #tmem2, #ttng.tensor_memory, mutable>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 40-41
```mlir
40|     // CHECK: ttng.tmem_alloc %{{.+}} {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 16 : i32}
41|     %5 = ttng.tmem_alloc %cst4 : (tensor<64x128xf16, #blocked2>) -> !ttg.memdesc<64x128xf16, #tmem2, #ttng.tensor_memory, mutable>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 42-49
```mlir
42|     // CHECK: ttng.tmem_alloc %{{.+}} {tensor_memory_col_offset = 128 : i32, tensor_memory_row_offset = 0 : i32}
43|     %6 = ttng.tmem_alloc %cst : (tensor<128x128xf32, #blocked>) -> !ttg.memdesc<128x128xf32, #tmem_f32, #ttng.tensor_memory, mutable>
44| 
45|     ttng.tmem_store %cst2, %4, %true : tensor<64x128xf16, #blocked2> -> !ttg.memdesc<64x128xf16, #tmem2, #ttng.tensor_memory, mutable>
46|     ttng.tmem_store %cst2, %5, %true : tensor<64x128xf16, #blocked2> -> !ttg.memdesc<64x128xf16, #tmem2, #ttng.tensor_memory, mutable>
47|     ttng.tmem_store %cst, %6, %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem_f32, #ttng.tensor_memory, mutable>
48| 
49|     %7 = ttng.tmem_alloc : () -> !ttg.memdesc<64x4xi8, #tmem_scales, #ttng.tensor_memory, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_store, tensor-memory allocation. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_store、张量内存分配。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 50-51
```mlir
50|     // CHECK: ttng.tmem_alloc  {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32}
51|     %8 = ttng.tmem_alloc : () -> !ttg.memdesc<64x4xi8, #tmem_scales, #ttng.tensor_memory, mutable>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 52-61
```mlir
52|     // CHECK: ttng.tmem_alloc  {tensor_memory_col_offset = 4 : i32, tensor_memory_row_offset = 0 : i32}
53| 
54|     ttng.tmem_store %cst3, %7, %true : tensor<64x4xi8, #linear> -> !ttg.memdesc<64x4xi8, #tmem_scales, #ttng.tensor_memory, mutable>
55|     ttng.tmem_store %cst3, %8, %true : tensor<64x4xi8, #linear> -> !ttg.memdesc<64x4xi8, #tmem_scales, #ttng.tensor_memory, mutable>
56| 
57| 
58|     tt.return
59|   }
60| }
61| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_store, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_store、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 62-62
```mlir
62| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 63-67
```mlir
63| 
64| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
65| #blocked1 = #ttg.blocked<{sizePerThread = [1, 64], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
66| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
67| #tmem1 = #ttng.tensor_memory_encoding<blockM = 128, blockN = 64, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 68-68
```mlir
68| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65536 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 69-70
```mlir
69|   // CHECK: ttg.tensor_memory_size = 512
70|   // CHECK: alloc_tensor_memory_re_use
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 71-79
```mlir
71|   tt.func public @alloc_tensor_memory_re_use(%arg0: !tt.ptr<f16>, %arg1: !tt.ptr<f16>, %arg2: !tt.ptr<f16>) {
72|     %true = arith.constant true
73|     %c1 = arith.constant 1 : i32
74|     %c0 = arith.constant 0 : i32
75|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
76|     %cst0 = arith.constant dense<0.000000e+00> : tensor<128x256xf32, #blocked>
77|     %cst1 = arith.constant dense<0.000000e+00> : tensor<64x64xf16, #blocked>
78|     %cst2 = arith.constant dense<0.000000e+00> : tensor<128x64xf32, #blocked1>
79| 
```
**EN:** This function-oriented block defines or enters `alloc_tensor_memory_re_use`. Within it, the test exercises constants, tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `alloc_tensor_memory_re_use` 为核心。测试在其中演示 常量、tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 80-82
```mlir
80|     // CHECK: ttng.tmem_alloc %{{.+}} {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32}
81|     %a = ttng.tmem_alloc %cst0 : (tensor<128x256xf32, #blocked>) -> !ttg.memdesc<128x256xf32, #tmem, #ttng.tensor_memory, mutable>
82| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 83-85
```mlir
83|     // CHECK: ttng.tmem_alloc %{{.+}} {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32}
84|     %0 = ttng.tmem_alloc %cst : (tensor<128x128xf32, #blocked>) -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
85| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 86-87
```mlir
86|     // CHECK: ttng.tmem_alloc %{{.+}} {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32}
87|     %1 = ttng.tmem_alloc %cst2 : (tensor<128x64xf32, #blocked1>) -> !ttg.memdesc<128x64xf32, #tmem1, #ttng.tensor_memory, mutable>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 88-93
```mlir
88|     // CHECK: ttng.tmem_alloc %{{.+}} {tensor_memory_col_offset = 64 : i32, tensor_memory_row_offset = 0 : i32}
89|     %2 = ttng.tmem_alloc %cst2 : (tensor<128x64xf32, #blocked1>) -> !ttg.memdesc<128x64xf32, #tmem1, #ttng.tensor_memory, mutable>
90|     ttng.tmem_store %cst2, %1, %true : tensor<128x64xf32, #blocked1> -> !ttg.memdesc<128x64xf32, #tmem1, #ttng.tensor_memory, mutable>
91|     ttng.tmem_store %cst2, %2, %true : tensor<128x64xf32, #blocked1> -> !ttg.memdesc<128x64xf32, #tmem1, #ttng.tensor_memory, mutable>
92| 
93|     // Test that the 2 allocations above are re-used.
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_store, tensor-memory allocation. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_store、张量内存分配。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 94-96
```mlir
94|     // CHECK: ttng.tmem_alloc %{{.+}} {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32}
95|     %3 = ttng.tmem_alloc %cst0 : (tensor<128x256xf32, #blocked>) -> !ttg.memdesc<128x256xf32, #tmem, #ttng.tensor_memory, mutable>
96| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 97-98
```mlir
97|     // CHECK: ttng.tmem_alloc %{{.+}} {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32}
98|     %4 = ttng.tmem_alloc %cst2 : (tensor<128x64xf32, #blocked1>) -> !ttg.memdesc<128x64xf32, #tmem1, #ttng.tensor_memory, mutable>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 99-102
```mlir
 99|     // CHECK: ttng.tmem_alloc %{{.+}} {tensor_memory_col_offset = 64 : i32, tensor_memory_row_offset = 0 : i32}
100|     %5 = ttng.tmem_alloc %cst2 : (tensor<128x64xf32, #blocked1>) -> !ttg.memdesc<128x64xf32, #tmem1, #ttng.tensor_memory, mutable>
101|     ttng.tmem_store %cst2, %4, %true : tensor<128x64xf32, #blocked1> -> !ttg.memdesc<128x64xf32, #tmem1, #ttng.tensor_memory, mutable>
102| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 103-106
```mlir
103|     // CHECK: ttng.tmem_alloc {tensor_memory_col_offset = 128 : i32, tensor_memory_row_offset = 0 : i32}
104|     %6 = ttng.tmem_alloc : () -> !ttg.memdesc<2x128x128xf32, #tmem, #ttng.tensor_memory, mutable>
105|     %s = ttg.memdesc_index %6[%c1] : !ttg.memdesc<2x128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
106| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 107-108
```mlir
107|     // CHECK: ttng.tmem_alloc %{{.+}} {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32}
108|     %7 = ttng.tmem_alloc %cst2 : (tensor<128x64xf32, #blocked1>) -> !ttg.memdesc<128x64xf32, #tmem1, #ttng.tensor_memory, mutable>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 109-118
```mlir
109|     // CHECK: ttng.tmem_alloc %{{.+}} {tensor_memory_col_offset = 384 : i32, tensor_memory_row_offset = 0 : i32}
110|     %8 = ttng.tmem_alloc %cst2 : (tensor<128x64xf32, #blocked1>) -> !ttg.memdesc<128x64xf32, #tmem1, #ttng.tensor_memory, mutable>
111| 
112|     ttng.tmem_store %cst, %s, %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
113|     ttng.tmem_store %cst2, %7, %true : tensor<128x64xf32, #blocked1> -> !ttg.memdesc<128x64xf32, #tmem1, #ttng.tensor_memory, mutable>
114|     ttng.tmem_store %cst2, %5, %true : tensor<128x64xf32, #blocked1> -> !ttg.memdesc<128x64xf32, #tmem1, #ttng.tensor_memory, mutable>
115|     tt.return
116|   }
117| }
118| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_store, tensor-memory allocation, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_store、张量内存分配、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 119-119
```mlir
119| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 120-122
```mlir
120| 
121| #blocked = #ttg.blocked<{sizePerThread = [1, 32], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
122| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 64, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 123-123
```mlir
123| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65536 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 124-125
```mlir
124|   // CHECK: ttg.tensor_memory_size = 128
125|   // CHECK: alloc_tensor_memory_re_use_liverange_end_collision
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 126-136
```mlir
126|   tt.func public @alloc_tensor_memory_re_use_liverange_end_collision(
127|                                              %arg0: !tt.ptr<f16>, %arg1: !tt.ptr<f16>, %arg2: !tt.ptr<f16>,
128|                                              %lb: index, %ub: index, %step: index) {
129|     %true = arith.constant true
130|     %c1 = arith.constant 1 : i32
131|     %c0 = arith.constant 0 : i32
132|     %cst = arith.constant dense<0.000000e+00> : tensor<128x64xf32, #blocked>
133|     %cst0 = arith.constant dense<0.000000e+00> : tensor<128x64xf32, #blocked>
134|     %cst1 = arith.constant dense<0.000000e+00> : tensor<128x64xf32, #blocked>
135|     %cst2 = arith.constant dense<0.000000e+00> : tensor<128x64xf32, #blocked>
136| 
```
**EN:** This function-oriented block defines or enters `alloc_tensor_memory_re_use_liverange_end_collision`. Within it, the test exercises constants, tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `alloc_tensor_memory_re_use_liverange_end_collision` 为核心。测试在其中演示 常量、tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 137-139
```mlir
137|     // CHECK: ttng.tmem_alloc %{{.+}} {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32}
138|     %a = ttng.tmem_alloc %cst0 : (tensor<128x64xf32, #blocked>) -> !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory, mutable>
139| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 140-149
```mlir
140|     // CHECK: ttng.tmem_alloc %{{.+}} {tensor_memory_col_offset = 64 : i32, tensor_memory_row_offset = 0 : i32}
141|     %b = ttng.tmem_alloc %cst : (tensor<128x64xf32, #blocked>) -> !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory, mutable>
142| 
143|     scf.for %i = %lb to %ub step %step {
144|       ttng.tmem_store %cst2, %a, %true : tensor<128x64xf32, #blocked> -> !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory, mutable>
145|       ttng.tmem_store %cst2, %b, %true : tensor<128x64xf32, #blocked> -> !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory, mutable>
146|       scf.yield
147|     }
148|     // Liveranges of both allocations end at the same time, at the boundary of the loop. Make sure we can handle this case.
149| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining ttng.tmem_store, tensor-memory allocation, structured loops, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 ttng.tmem_store、张量内存分配、结构化循环、循环/分支产出值。

### Lines 150-152
```mlir
150|     // CHECK: ttng.tmem_alloc %{{.+}} {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32}
151|     %c = ttng.tmem_alloc %cst0 : (tensor<128x64xf32, #blocked>) -> !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory, mutable>
152| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 153-162
```mlir
153|     // CHECK: ttng.tmem_alloc %{{.+}} {tensor_memory_col_offset = 64 : i32, tensor_memory_row_offset = 0 : i32}
154|     %d = ttng.tmem_alloc %cst : (tensor<128x64xf32, #blocked>) -> !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory, mutable>
155| 
156|     ttng.tmem_store %cst2, %c, %true : tensor<128x64xf32, #blocked> -> !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory, mutable>
157|     ttng.tmem_store %cst2, %d, %true : tensor<128x64xf32, #blocked> -> !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory, mutable>
158| 
159|     tt.return
160|   }
161| }
162| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_store, tensor-memory allocation, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_store、张量内存分配、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 163-163
```mlir
163| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 164-168
```mlir
164| 
165| #blocked = #ttg.blocked<{sizePerThread = [1, 32], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1], CGALayout = [[0, 1]]}>
166| #blocked1 = #ttg.blocked<{sizePerThread = [1, 64], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1], CGALayout = [[1, 0]]}>
167| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 2, CGALayout = [[1, 0]]>
168| #tmem1 = #ttng.tensor_memory_encoding<blockM = 128, blockN = 64, colStride = 2, CGALayout = [[0, 1]]>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 169-169
```mlir
169| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32, ttg.shared = 65536 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 170-170
```mlir
170|   // CHECK-LABEL: multi_ctas
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: multi_ctas anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: multi_ctas 这样的标签用于锚定匹配范围。

### Lines 171-175
```mlir
171|   tt.func public @multi_ctas() {
172|     %true = arith.constant true
173|     %cst0 = arith.constant dense<0.000000e+00> : tensor<256x128xf16, #blocked>
174|     %cst1 = arith.constant dense<0.000000e+00> : tensor<256x128xf16, #blocked1>
175| 
```
**EN:** This function-oriented block defines or enters `multi_ctas`. Within it, the test exercises constants, tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `multi_ctas` 为核心。测试在其中演示 常量、tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 176-177
```mlir
176|     // CHECK: ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32}
177|     %0 = ttng.tmem_alloc : () -> !ttg.memdesc<256x128xf16, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 178-179
```mlir
178|     // CHECK: ttng.tmem_alloc {tensor_memory_col_offset = 128 : i32, tensor_memory_row_offset = 0 : i32}
179|     %1 = ttng.tmem_alloc : () -> !ttg.memdesc<256x128xf16, #tmem1, #ttng.tensor_memory, mutable>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 180-189
```mlir
180|     // CHECK: ttng.tmem_alloc {tensor_memory_col_offset = 256 : i32, tensor_memory_row_offset = 0 : i32}
181|     %2 = ttng.tmem_alloc : () -> !ttg.memdesc<256x128xf16, #tmem, #ttng.tensor_memory, mutable>
182| 
183|     ttng.tmem_store %cst1, %0, %true : tensor<256x128xf16, #blocked1> -> !ttg.memdesc<256x128xf16, #tmem, #ttng.tensor_memory, mutable>
184|     ttng.tmem_store %cst0, %1, %true : tensor<256x128xf16, #blocked> -> !ttg.memdesc<256x128xf16, #tmem1, #ttng.tensor_memory, mutable>
185|     ttng.tmem_store %cst1, %2, %true : tensor<256x128xf16, #blocked1> -> !ttg.memdesc<256x128xf16, #tmem, #ttng.tensor_memory, mutable>
186|     tt.return
187|   }
188| }
189| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_store, tensor-memory allocation, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_store、张量内存分配、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 190-190
```mlir
190| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 191-194
```mlir
191| 
192| #layout = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
193| #tmem = #ttng.tensor_memory
194| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 195-196
```mlir
195| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32, ttg.shared = 65536 : i32, ttg.target = "cuda:100"} {
196| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 197-197
```mlir
197| // CHECK-LABEL: @alloc_warp_specialize
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @alloc_warp_specialize anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @alloc_warp_specialize 这样的标签用于锚定匹配范围。

### Lines 198-198
```mlir
198| tt.func @alloc_warp_specialize() {
```
**EN:** This function-oriented block defines or enters `alloc_warp_specialize`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `alloc_warp_specialize` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 199-202
```mlir
199|   // CHECK: ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32}
200|   %0 = ttng.tmem_alloc : () -> !ttg.memdesc<128x128xf32, #layout, #tmem, mutable>
201|   ttg.warp_specialize()
202|   default {
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor-memory allocation, ttg.warp_specialize. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量内存分配、ttg.warp_specialize。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 203-204
```mlir
203|     // CHECK: ttng.tmem_alloc {tensor_memory_col_offset = 128 : i32, tensor_memory_row_offset = 0 : i32}
204|     %1 = ttng.tmem_alloc : () -> !ttg.memdesc<128x128xf32, #layout, #tmem, mutable>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 205-209
```mlir
205|     // CHECK: ttng.tmem_alloc {tensor_memory_col_offset = 128 : i32, tensor_memory_row_offset = 0 : i32}
206|     %2 = ttng.tmem_alloc : () -> !ttg.memdesc<128x128xf32, #layout, #tmem, mutable>
207|     ttg.warp_yield
208|   }
209|   partition0() num_warps(1) {
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor-memory allocation, ttg.warp_yield. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量内存分配、ttg.warp_yield。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 210-211
```mlir
210|     // CHECK: ttng.tmem_alloc {tensor_memory_col_offset = 256 : i32, tensor_memory_row_offset = 0 : i32}
211|     %1 = ttng.tmem_alloc : () -> !ttg.memdesc<128x128xf32, #layout, #tmem, mutable>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 212-220
```mlir
212|     // CHECK: ttng.tmem_alloc {tensor_memory_col_offset = 384 : i32, tensor_memory_row_offset = 0 : i32}
213|     %2 = ttng.tmem_alloc : () -> !ttg.memdesc<128x128xf32, #layout, #tmem, mutable>
214|     "use"(%1) : (!ttg.memdesc<128x128xf32, #layout, #tmem, mutable>) -> ()
215|     ttg.warp_return
216|   } : () -> ()
217|   "use"(%0) : (!ttg.memdesc<128x128xf32, #layout, #tmem, mutable>) -> ()
218|   tt.return
219| }
220| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor-memory allocation, ttg.warp_return, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量内存分配、ttg.warp_return、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 221-221
```mlir
221| // CHECK-LABEL: @alloc_warp_specialize_explicit_capture
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @alloc_warp_specialize_explicit_capture anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @alloc_warp_specialize_explicit_capture 这样的标签用于锚定匹配范围。

### Lines 222-222
```mlir
222| tt.func @alloc_warp_specialize_explicit_capture() {
```
**EN:** This function-oriented block defines or enters `alloc_warp_specialize_explicit_capture`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `alloc_warp_specialize_explicit_capture` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 223-226
```mlir
223|   // CHECK: ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32}
224|   %0 = ttng.tmem_alloc : () -> !ttg.memdesc<128x128xf32, #layout, #tmem, mutable>
225|   ttg.warp_specialize(%0)
226|   default {
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor-memory allocation, ttg.warp_specialize. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量内存分配、ttg.warp_specialize。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 227-231
```mlir
227|     // CHECK: ttng.tmem_alloc {tensor_memory_col_offset = 128 : i32, tensor_memory_row_offset = 0 : i32}
228|     %1 = ttng.tmem_alloc : () -> !ttg.memdesc<128x128xf32, #layout, #tmem, mutable>
229|     ttg.warp_yield
230|   }
231|   partition0(%arg0: !ttg.memdesc<128x128xf32, #layout, #tmem, mutable>) num_warps(1) {
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor-memory allocation, ttg.warp_yield. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量内存分配、ttg.warp_yield。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 232-240
```mlir
232|     // CHECK: ttng.tmem_alloc {tensor_memory_col_offset = 256 : i32, tensor_memory_row_offset = 0 : i32}
233|     %1 = ttng.tmem_alloc : () -> !ttg.memdesc<128x128xf32, #layout, #tmem, mutable>
234|     ttg.warp_return
235|   } : (!ttg.memdesc<128x128xf32, #layout, #tmem, mutable>) -> ()
236|   tt.return
237| }
238| 
239| }
240| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor-memory allocation, ttg.warp_return, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量内存分配、ttg.warp_return、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 241-241
```mlir
241| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 242-249
```mlir
242| 
243| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 8}>
244| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = true, elementBitWidth = 8}>
245| #shared2 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
246| #tmem_f16 = #ttng.tensor_memory_encoding<blockM = 128, blockN = 64, colStride = 1>
247| #tmem_f32 = #ttng.tensor_memory_encoding<blockM = 128, blockN = 64, colStride = 1>
248| #tmem_scales = #ttng.tensor_memory_scales_encoding<>
249| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 250-251
```mlir
250| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65536 : i32} {
251| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 252-252
```mlir
252| // CHECK-LABEL: @mma_lhs_tmem
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @mma_lhs_tmem anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @mma_lhs_tmem 这样的标签用于锚定匹配范围。

### Lines 253-259
```mlir
253| tt.func @mma_lhs_tmem(
254|   %b: !ttg.memdesc<64x64xf16, #shared1, #ttg.shared_memory>,
255|   %useAcc: i1,
256|   %pred: i1,
257|   %barrier: !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory>,
258|   %barrierPred: i1
259| ) {
```
**EN:** This function-oriented block defines or enters `mma_lhs_tmem`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mma_lhs_tmem` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 260-271
```mlir
260|   // CHECK-COUNT-2: ttng.tmem_alloc {{.*}} tensor_memory_row_offset = 0 : i32
261|   // CHECK-NOT: tensor_memory_row_offset
262|   %a = ttng.tmem_alloc : () -> !ttg.memdesc<128x64xf16, #tmem_f16, #ttng.tensor_memory, mutable>
263|   %c = ttng.tmem_alloc : () -> !ttg.memdesc<128x64xf32, #tmem_f32, #ttng.tensor_memory, mutable>
264|   ttng.tc_gen5_mma %a, %b, %c, %useAcc, %pred, %barrier[%barrierPred] {is_async} :
265|     !ttg.memdesc<128x64xf16, #tmem_f16, #ttng.tensor_memory, mutable>,
266|     !ttg.memdesc<64x64xf16, #shared1, #ttg.shared_memory>,
267|     !ttg.memdesc<128x64xf32, #tmem_f32, #ttng.tensor_memory, mutable>,
268|     !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory>
269|   tt.return
270| }
271| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor-memory allocation, Gen5 tensor-core MMA ops, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量内存分配、Gen5 张量核 MMA 操作、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 272-272
```mlir
272| // CHECK-LABEL: @mma_scaled_lhs_tmem
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @mma_scaled_lhs_tmem anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @mma_scaled_lhs_tmem 这样的标签用于锚定匹配范围。

### Lines 273-281
```mlir
273| tt.func @mma_scaled_lhs_tmem(
274|   %b: !ttg.memdesc<64x64xf16, #shared1, #ttg.shared_memory>,
275|   %scale_a: !ttg.memdesc<128x8xf8E4M3FN, #tmem_scales, #ttng.tensor_memory>,
276|   %scale_b: !ttg.memdesc<256x8xf8E4M3FN, #tmem_scales, #ttng.tensor_memory>,
277|   %useAcc: i1,
278|   %pred: i1,
279|   %barrier: !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory>,
280|   %barrierPred: i1
281| ) {
```
**EN:** This function-oriented block defines or enters `mma_scaled_lhs_tmem`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mma_scaled_lhs_tmem` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 282-297
```mlir
282|   // CHECK-COUNT-2: ttng.tmem_alloc {{.*}} tensor_memory_row_offset = 0 : i32
283|   // CHECK-NOT: tensor_memory_row_offset
284|   %a = ttng.tmem_alloc : () -> !ttg.memdesc<128x64xf16, #tmem_f16, #ttng.tensor_memory, mutable>
285|   %c = ttng.tmem_alloc : () -> !ttg.memdesc<128x64xf32, #tmem_f32, #ttng.tensor_memory, mutable>
286|   ttng.tc_gen5_mma_scaled %a, %b, %c, %scale_a, %scale_b, %useAcc, %pred lhs = e2m1 rhs = e2m1, %barrier[%barrierPred] {is_async} :
287|     !ttg.memdesc<128x64xf16, #tmem_f16, #ttng.tensor_memory, mutable>,
288|     !ttg.memdesc<64x64xf16, #shared1, #ttg.shared_memory>,
289|     !ttg.memdesc<128x64xf32, #tmem_f32, #ttng.tensor_memory, mutable>,
290|     !ttg.memdesc<128x8xf8E4M3FN, #tmem_scales, #ttng.tensor_memory>,
291|     !ttg.memdesc<256x8xf8E4M3FN, #tmem_scales, #ttng.tensor_memory>,
292|     !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory>
293|   tt.return
294| }
295| 
296| }
297| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor-memory allocation, scaled tensor-core MMA ops, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量内存分配、带缩放的张量核 MMA 操作、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 298-298
```mlir
298| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 299-303
```mlir
299| 
300| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
301| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
302| #smem = #ttg.shared_memory
303| #tmem = #ttng.tensor_memory_encoding<blockM = 64, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 304-305
```mlir
304| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32, ttg.shared = 65536 : i32, ttg.target = "cuda:100"} {
305| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 306-306
```mlir
306| // CHECK-LABEL: @alloc_warp_specialize_explicit_capture_subview
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @alloc_warp_specialize_explicit_capture_subview anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @alloc_warp_specialize_explicit_capture_subview 这样的标签用于锚定匹配范围。

### Lines 307-307
```mlir
307| tt.func @alloc_warp_specialize_explicit_capture_subview() {
```
**EN:** This function-oriented block defines or enters `alloc_warp_specialize_explicit_capture_subview`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `alloc_warp_specialize_explicit_capture_subview` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 308-311
```mlir
308|   // CHECK: ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32}
309|   %0 = ttg.local_alloc {allocation.offset = 196880 : i32} : () -> !ttg.memdesc<2x1xi64, #shared, #smem, mutable>
310|   %1 = ttng.tmem_alloc : () -> !ttg.memdesc<1x64x128xbf16, #tmem, #ttng.tensor_memory, mutable>
311|   %2 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<2x128x128xbf16, #shared1, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, tensor-memory allocation. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、张量内存分配。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 312-329
```mlir
312|   // CHECK: ttng.tmem_alloc {tensor_memory_col_offset = 64 : i32, tensor_memory_row_offset = 0 : i32}
313|   %3 = ttng.tmem_alloc : () -> !ttg.memdesc<1x64x128xf32, #tmem, #ttng.tensor_memory, mutable>
314|   ttg.warp_specialize(%2, %1, %3, %0)
315|   default {
316|     ttg.warp_yield
317|   }
318|   partition0(%arg0: !ttg.memdesc<2x128x128xbf16, #shared1, #smem, mutable>, %arg1: !ttg.memdesc<1x64x128xbf16, #tmem, #ttng.tensor_memory, mutable>, %arg2: !ttg.memdesc<1x64x128xf32, #tmem, #ttng.tensor_memory, mutable>, %arg3: !ttg.memdesc<2x1xi64, #shared, #smem, mutable>) num_warps(1) {
319|     %true = arith.constant true
320|     %c0_i32 = arith.constant 0 : i32
321| 
322|     %b = ttg.memdesc_index %arg0[%c0_i32] : !ttg.memdesc<2x128x128xbf16, #shared1, #smem, mutable> -> !ttg.memdesc<128x128xbf16, #shared1, #smem>
323|     %a = ttg.memdesc_index %arg1[%c0_i32] : !ttg.memdesc<1x64x128xbf16, #tmem, #ttng.tensor_memory, mutable> -> !ttg.memdesc<64x128xbf16, #tmem, #ttng.tensor_memory, mutable>
324|     %d = ttg.memdesc_index %arg2[%c0_i32] : !ttg.memdesc<1x64x128xf32, #tmem, #ttng.tensor_memory, mutable> -> !ttg.memdesc<64x128xf32, #tmem, #ttng.tensor_memory, mutable>
325|     %barrier = ttg.memdesc_index %arg3[%c0_i32] : !ttg.memdesc<2x1xi64, #shared, #smem, mutable> -> !ttg.memdesc<1xi64, #shared, #smem, mutable>
326| 
327|     ttng.tc_gen5_mma %a, %b, %d, %true, %true, %barrier[%true] {is_async} : !ttg.memdesc<64x128xbf16, #tmem, #ttng.tensor_memory, mutable>, !ttg.memdesc<128x128xbf16, #shared1, #smem>, !ttg.memdesc<64x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.memdesc<1xi64, #shared, #smem, mutable>
328|     ttg.warp_return
329|   } : (!ttg.memdesc<2x128x128xbf16, #shared1, #smem, mutable>, !ttg.memdesc<1x64x128xbf16, #tmem, #ttng.tensor_memory, mutable>, !ttg.memdesc<1x64x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.memdesc<2x1xi64, #shared, #smem, mutable>) -> ()
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.memdesc_index, constants, tensor-memory allocation, ttg.warp_specialize, ttg.warp_yield. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.memdesc_index、常量、张量内存分配、ttg.warp_specialize、ttg.warp_yield。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 330-332
```mlir
330|   tt.return
331| }
332| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 333-333
```mlir
333| // CHECK-LABEL: @alloc_warp_specialize_explicit_capture
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @alloc_warp_specialize_explicit_capture anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @alloc_warp_specialize_explicit_capture 这样的标签用于锚定匹配范围。

### Lines 334-334
```mlir
334| tt.func @alloc_warp_specialize_explicit_capture() {
```
**EN:** This function-oriented block defines or enters `alloc_warp_specialize_explicit_capture`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `alloc_warp_specialize_explicit_capture` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 335-338
```mlir
335|   // CHECK: ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32}
336|   %0 = ttg.local_alloc {allocation.offset = 196880 : i32} : () -> !ttg.memdesc<2x1xi64, #shared, #smem, mutable>
337|   %1 = ttng.tmem_alloc : () -> !ttg.memdesc<64x128xbf16, #tmem, #ttng.tensor_memory, mutable>
338|   %2 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<2x128x128xbf16, #shared1, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, tensor-memory allocation. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、张量内存分配。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 339-356
```mlir
339|   // CHECK: ttng.tmem_alloc {tensor_memory_col_offset = 64 : i32, tensor_memory_row_offset = 0 : i32}
340|   %3 = ttng.tmem_alloc : () -> !ttg.memdesc<64x128xf32, #tmem, #ttng.tensor_memory, mutable>
341|   ttg.warp_specialize(%2, %1, %3, %0)
342|   default {
343|     ttg.warp_yield
344|   }
345|   partition0(%arg0: !ttg.memdesc<2x128x128xbf16, #shared1, #smem, mutable>, %arg1: !ttg.memdesc<64x128xbf16, #tmem, #ttng.tensor_memory, mutable>, %arg2: !ttg.memdesc<64x128xf32, #tmem, #ttng.tensor_memory, mutable>, %arg3: !ttg.memdesc<2x1xi64, #shared, #smem, mutable>) num_warps(1) {
346|     %true = arith.constant true
347|     %c0_i32 = arith.constant 0 : i32
348| 
349|     %b = ttg.memdesc_index %arg0[%c0_i32] : !ttg.memdesc<2x128x128xbf16, #shared1, #smem, mutable> -> !ttg.memdesc<128x128xbf16, #shared1, #smem>
350|     %barrier = ttg.memdesc_index %arg3[%c0_i32] : !ttg.memdesc<2x1xi64, #shared, #smem, mutable> -> !ttg.memdesc<1xi64, #shared, #smem, mutable>
351| 
352|     ttng.tc_gen5_mma %arg1, %b, %arg2, %true, %true, %barrier[%true] {is_async} : !ttg.memdesc<64x128xbf16, #tmem, #ttng.tensor_memory, mutable>, !ttg.memdesc<128x128xbf16, #shared1, #smem>, !ttg.memdesc<64x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.memdesc<1xi64, #shared, #smem, mutable>
353|     ttg.warp_return
354|   } : (!ttg.memdesc<2x128x128xbf16, #shared1, #smem, mutable>, !ttg.memdesc<64x128xbf16, #tmem, #ttng.tensor_memory, mutable>, !ttg.memdesc<64x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.memdesc<2x1xi64, #shared, #smem, mutable>) -> ()
355|   tt.return
356| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants, ttg.memdesc_index, tensor-memory allocation, ttg.warp_specialize, ttg.warp_yield. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量、ttg.memdesc_index、张量内存分配、ttg.warp_specialize、ttg.warp_yield。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 357-359
```mlir
357| 
358| }
359| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 360-360
```mlir
360| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 361-368
```mlir
361| 
362| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 8}>
363| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = true, elementBitWidth = 8}>
364| #shared2 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
365| #tmem_f16 = #ttng.tensor_memory_encoding<blockM = 64, blockN = 64, colStride = 1>
366| #tmem_f32 = #ttng.tensor_memory_encoding<blockM = 64, blockN = 64, colStride = 1>
367| #tmem_scales = #ttng.tensor_memory_scales_encoding<>
368| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 369-370
```mlir
369| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65536 : i32} {
370| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 371-371
```mlir
371| // CHECK-LABEL: @mma_lhs_tmem
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @mma_lhs_tmem anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @mma_lhs_tmem 这样的标签用于锚定匹配范围。

### Lines 372-378
```mlir
372| tt.func @mma_lhs_tmem(
373|   %b: !ttg.memdesc<64x64xf16, #shared1, #ttg.shared_memory>,
374|   %useAcc: i1,
375|   %pred: i1,
376|   %barrier: !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory>,
377|   %barrierPred: i1
378| ) {
```
**EN:** This function-oriented block defines or enters `mma_lhs_tmem`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mma_lhs_tmem` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 379-396
```mlir
379|   // CHECK-COUNT-4: ttng.tmem_alloc {{.*}} tensor_memory_row_offset = 0 : i32
380|   // CHECK-NOT: tensor_memory_row_offset
381|   %a0 = ttng.tmem_alloc : () -> !ttg.memdesc<64x64xf16, #tmem_f16, #ttng.tensor_memory, mutable>
382|   %a1 = ttng.tmem_alloc : () -> !ttg.memdesc<64x64xf16, #tmem_f16, #ttng.tensor_memory, mutable>
383|   %a2 = ttng.tmem_alloc : () -> !ttg.memdesc<64x64xf16, #tmem_f16, #ttng.tensor_memory, mutable>
384|   %c = ttng.tmem_alloc : () -> !ttg.memdesc<64x64xf32, #tmem_f32, #ttng.tensor_memory, mutable>
385| 
386|   %a = arith.select %barrierPred, %a0, %a1 : !ttg.memdesc<64x64xf16, #tmem_f16, #ttng.tensor_memory, mutable>
387| 
388|   cf.cond_br %barrierPred, ^switch, ^bb1(%a : !ttg.memdesc<64x64xf16, #tmem_f16, #ttng.tensor_memory, mutable>)
389| 
390| ^switch:
391|   cf.br ^bb1(%a2 : !ttg.memdesc<64x64xf16, #tmem_f16, #ttng.tensor_memory, mutable>)
392| 
393| ^bb1(%lhs: !ttg.memdesc<64x64xf16, #tmem_f16, #ttng.tensor_memory, mutable>):
394|   ttng.tc_gen5_mma %lhs, %b, %c, %useAcc, %pred, %barrier[%barrierPred] {is_async} :
395|     !ttg.memdesc<64x64xf16, #tmem_f16, #ttng.tensor_memory, mutable>,
396|     !ttg.memdesc<64x64xf16, #shared1, #ttg.shared_memory>,
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining tensor-memory allocation, arith.select, cf.cond_br, cf.br, Gen5 tensor-core MMA ops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 张量内存分配、arith.select、cf.cond_br、cf.br、Gen5 张量核 MMA 操作。

### Lines 397-402
```mlir
397|     !ttg.memdesc<64x64xf32, #tmem_f32, #ttng.tensor_memory, mutable>,
398|     !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory>
399|   tt.return
400| }
401| 
402| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-allow-unregistered-dialect`, `-triton-tensor-memory-allocation`
- **CN:** 主要 pass 选项：`-split-input-file`，`-allow-unregistered-dialect`，`-triton-tensor-memory-allocation`
- **EN:** Dominant operations include `ttng.tmem_alloc`, `arith.constant`, `tt.func`, `ttng.tmem_store`, `tt.return`, `module`, `ttg.memdesc_index`, `ttg.warp_specialize`, `ttg.warp_yield`, `ttg.warp_return`.
- **CN:** 主要操作包括 `ttng.tmem_alloc`、`arith.constant`、`tt.func`、`ttng.tmem_store`、`tt.return`、`module`、`ttg.memdesc_index`、`ttg.warp_specialize`、`ttg.warp_yield`、`ttg.warp_return`。
- **EN:** The file contains 7 independently testable section(s). Check styles used: CHECK x44, CHECK-LABEL x8, CHECK-COUNT x3, CHECK-NOT x3. Important labels include multi_ctas, @alloc_warp_specialize, @alloc_warp_specialize_explicit_capture, @mma_lhs_tmem. It validates both the presence of the desired rewrite and the absence of unwanted residual operations.
- **CN:** 该文件包含 7 个可独立测试的分段。使用的检查类型：CHECK ×44，CHECK-LABEL ×8，CHECK-COUNT ×3，CHECK-NOT ×3。 关键标签包括 multi_ctas，@alloc_warp_specialize，@alloc_warp_specialize_explicit_capture，@mma_lhs_tmem。 它同时验证目标重写已经出现，并且不再残留不需要的操作。

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
- **EN:** `scf`: Structured control flow for loops and conditionals.
- **CN:** `scf`：用于循环与条件分支的结构化控制流。
- **EN:** `cf`: Control-flow dialect for basic branches.
- **CN:** `cf`：基础分支控制流方言。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。