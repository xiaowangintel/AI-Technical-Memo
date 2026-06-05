# nvidia-fpsan.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/nvidia-fpsan.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-tritoninstrument-fp-sanitizer` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-tritoninstrument-fp-sanitizer` 的行为。
- **EN:** RUN pipeline(s): `// RUN: split-file %s %t`; `// RUN: triton-opt %t/success.mlir -split-input-file -allow-unregistered-dialect -tritoninstrument-fp-sanitizer | FileCheck %t/success.mlir`
- **CN:** RUN 流水线：`// RUN: split-file %s %t`；`// RUN: triton-opt %t/success.mlir -split-input-file -allow-unregistered-dialect -tritoninstrument-fp-sanitizer | FileCheck %t/success.mlir`

## Line-by-Line Analysis / 逐行分析

### Lines 1-5
```mlir
1| // RUN: split-file %s %t
2| // RUN: triton-opt %t/success.mlir -split-input-file -allow-unregistered-dialect -tritoninstrument-fp-sanitizer | FileCheck %t/success.mlir
3| 
4| //--- success.mlir
5| 
```
**EN:** This header defines how the test is executed. It runs `split-file %s %t` ; ` triton-opt %t/success.mlir -split-input-file -allow-unregistered-dialect -tritoninstrument-fp-sanitizer | FileCheck %t/success.mlir` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `split-file %s %t`；` triton-opt %t/success.mlir -split-input-file -allow-unregistered-dialect -tritoninstrument-fp-sanitizer | FileCheck %t/success.mlir`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 6-7
```mlir
6| #blocked = #ttg.blocked<{sizePerThread = [1, 64], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
7| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 8-8
```mlir
8| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.tensor_memory_size = 0 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 9-9
```mlir
9|   // CHECK-LABEL: @tmem_load_store
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tmem_load_store anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tmem_load_store 这样的标签用于锚定匹配范围。

### Lines 10-10
```mlir
10|   tt.func public @tmem_load_store() {
```
**EN:** This function-oriented block defines or enters `tmem_load_store`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tmem_load_store` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 11-25
```mlir
11|     // CHECK: ttg.global_scratch_alloc
12|     // CHECK: tt.store
13|     // CHECK: tt.load
14|     // CHECK-NOT: ttng.tmem_load
15|     // CHECK-NOT: ttng.tmem_store
16|     %true = arith.constant true
17|     %zero = arith.constant dense<0.0> : tensor<128x128xf32, #blocked>
18|     %buf = ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
19|     ttng.tmem_store %zero, %buf, %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
20|     %val = ttng.tmem_load %buf : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
21|     ttng.tmem_store %val, %buf, %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
22|     tt.return
23|   }
24| }
25| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants, ttng.tmem_store, tensor-memory allocation, ttng.tmem_load, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量、ttng.tmem_store、张量内存分配、ttng.tmem_load、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 26-26
```mlir
26| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 27-31
```mlir
27| 
28| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
29| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
30| #smem = #ttg.shared_memory
31| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 32-32
```mlir
32| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.tensor_memory_size = 0 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 33-33
```mlir
33|   // CHECK-LABEL: @tmem_copy_subslice
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tmem_copy_subslice anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tmem_copy_subslice 这样的标签用于锚定匹配范围。

### Lines 34-34
```mlir
34|   tt.func public @tmem_copy_subslice() {
```
**EN:** This function-oriented block defines or enters `tmem_copy_subslice`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tmem_copy_subslice` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 35-45
```mlir
35|     // CHECK: ttg.global_scratch_alloc
36|     // CHECK-NOT: ttng.tmem_copy
37|     // CHECK-NOT: ttng.tmem_subslice
38|     %src = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<128x128xf32, #shared, #smem, mutable>
39|     %dst = ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
40|     %sub = ttng.tmem_subslice %dst {N = 0 : i32} : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
41|     ttng.tmem_copy %src, %sub : !ttg.memdesc<128x128xf32, #shared, #smem, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
42|     tt.return
43|   }
44| }
45| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, tensor-memory allocation, ttng.tmem_subslice, tensor-memory copies, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、张量内存分配、ttng.tmem_subslice、张量内存拷贝、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 46-46
```mlir
46| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 47-49
```mlir
47| 
48| #blocked = #ttg.blocked<{sizePerThread = [1, 64], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
49| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 50-50
```mlir
50| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.tensor_memory_size = 0 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 51-51
```mlir
51|   // CHECK-LABEL: @tmem_subslice_load_store
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tmem_subslice_load_store anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tmem_subslice_load_store 这样的标签用于锚定匹配范围。

### Lines 52-52
```mlir
52|   tt.func public @tmem_subslice_load_store() {
```
**EN:** This function-oriented block defines or enters `tmem_subslice_load_store`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tmem_subslice_load_store` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 53-66
```mlir
53|     // CHECK: ttg.global_scratch_alloc
54|     // CHECK-NOT: ttng.tmem_subslice
55|     // CHECK-NOT: ttng.tmem_load
56|     // CHECK-NOT: ttng.tmem_store
57|     %true = arith.constant true
58|     %zero = arith.constant dense<0.0> : tensor<128x128xf32, #blocked>
59|     %buf = ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<128x256xf32, #tmem, #ttng.tensor_memory, mutable>
60|     %sub = ttng.tmem_subslice %buf {N = 128 : i32} : !ttg.memdesc<128x256xf32, #tmem, #ttng.tensor_memory, mutable> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable, 128x256>
61|     ttng.tmem_store %zero, %sub, %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable, 128x256>
62|     %val = ttng.tmem_load %sub : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable, 128x256> -> tensor<128x128xf32, #blocked>
63|     tt.return
64|   }
65| }
66| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants, tensor-memory allocation, ttng.tmem_subslice, ttng.tmem_store, ttng.tmem_load. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量、张量内存分配、ttng.tmem_subslice、ttng.tmem_store、ttng.tmem_load。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 67-67
```mlir
67| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 68-72
```mlir
68| 
69| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
70| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
71| #smem = #ttg.shared_memory
72| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 73-73
```mlir
73| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.tensor_memory_size = 0 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 74-74
```mlir
74|   // CHECK-LABEL: @tcgen05_mma
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tcgen05_mma anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tcgen05_mma 这样的标签用于锚定匹配范围。

### Lines 75-75
```mlir
75|   tt.func public @tcgen05_mma() {
```
**EN:** This function-oriented block defines or enters `tcgen05_mma`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tcgen05_mma` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 76-93
```mlir
76|     // CHECK: ttg.global_scratch_alloc
77|     // CHECK: ttg.barrier global_read|global_write
78|     // CHECK-NEXT: scf.for
79|     // CHECK: ttg.barrier global_read|global_write
80|     // CHECK: tt.store
81|     // CHECK-NEXT: }
82|     // CHECK-NEXT: }
83|     // CHECK-NEXT: ttg.barrier global_read|global_write
84|     // CHECK-NEXT: ttng.arrive_barrier
85|     // CHECK-NOT: ttng.tc_gen5_mma
86|     %true = arith.constant true
87|     %a = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
88|     %b = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
89|     %d = ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
90|     %bar = ttg.local_alloc {allocation.offset = 8192 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
91|     ttng.tc_gen5_mma %a, %b, %d, %true, %true, %bar[%true] {is_async} : !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.memdesc<1xi64, #shared1, #smem, mutable>
92|     tt.return
93|   }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining shared/local memory allocation, constants, tensor-memory allocation, Gen5 tensor-core MMA ops, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 共享/本地内存分配、常量、张量内存分配、Gen5 张量核 MMA 操作、tt.return。

### Lines 94-95
```mlir
94| }
95| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 96-96
```mlir
96| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 97-101
```mlir
 97| 
 98| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
 99| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
100| #smem = #ttg.shared_memory
101| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 102-102
```mlir
102| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.tensor_memory_size = 0 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 103-103
```mlir
103|   // CHECK-LABEL: @tcgen05_mma_pred
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tcgen05_mma_pred anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tcgen05_mma_pred 这样的标签用于锚定匹配范围。

### Lines 104-104
```mlir
104|   tt.func public @tcgen05_mma_pred() {
```
**EN:** This function-oriented block defines or enters `tcgen05_mma_pred`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tcgen05_mma_pred` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 105-122
```mlir
105|     // CHECK: ttg.global_scratch_alloc
106|     // CHECK: ttg.barrier global_read|global_write
107|     // CHECK-NEXT: scf.for
108|     // CHECK: ttg.barrier global_read|global_write
109|     // CHECK: tt.store
110|     // CHECK-NEXT: }
111|     // CHECK-NEXT: }
112|     // CHECK-NEXT: ttg.barrier global_read|global_write
113|     // CHECK-NEXT: ttng.arrive_barrier
114|     // CHECK-NOT: ttng.tc_gen5_mma
115|     %false = arith.constant false
116|     %a = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
117|     %b = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
118|     %d = ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
119|     %bar = ttg.local_alloc {allocation.offset = 8192 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
120|     ttng.tc_gen5_mma %a, %b, %d, %false, %false, %bar[%false] {is_async} : !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.memdesc<1xi64, #shared1, #smem, mutable>
121|     tt.return
122|   }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining shared/local memory allocation, constants, tensor-memory allocation, Gen5 tensor-core MMA ops, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 共享/本地内存分配、常量、张量内存分配、Gen5 张量核 MMA 操作、tt.return。

### Lines 123-124
```mlir
123| }
124| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 125-125
```mlir
125| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 126-132
```mlir
126| 
127| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 8}>
128| #sharedT = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 8}>
129| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 0, transposed = false, elementBitWidth = 8}>
130| #shared2 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
131| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 64, colStride = 1>
132| #tmem_scales = #ttng.tensor_memory_scales_encoding<>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 133-133
```mlir
133| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.tensor_memory_size = 0 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 134-134
```mlir
134|   // CHECK-LABEL: @tcgen05_mma_scaled
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tcgen05_mma_scaled anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tcgen05_mma_scaled 这样的标签用于锚定匹配范围。

### Lines 135-135
```mlir
135|   tt.func public @tcgen05_mma_scaled() {
```
**EN:** This function-oriented block defines or enters `tcgen05_mma_scaled`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tcgen05_mma_scaled` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 136-153
```mlir
136|     // CHECK: ttg.global_scratch_alloc
137|     // CHECK: ttg.barrier global_read|global_write
138|     // CHECK-NEXT: scf.for
139|     // CHECK: ttg.barrier global_read|global_write
140|     // CHECK: tt.store
141|     // CHECK-NEXT: }
142|     // CHECK-NEXT: }
143|     // CHECK-NEXT: ttg.barrier global_read|global_write
144|     // CHECK-NEXT: ttng.arrive_barrier
145|     // CHECK-NOT: ttng.tc_gen5_mma_scaled
146|     %true = arith.constant true
147|     %a = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<128x256xi8, #shared, #ttg.shared_memory, mutable>
148|     %b = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<256x64xi8, #sharedT, #ttg.shared_memory, mutable>
149|     %d = ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory, mutable>
150|     %a_scale = ttng.tmem_alloc : () -> !ttg.memdesc<128x8xf8E4M3FN, #tmem_scales, #ttng.tensor_memory, mutable>
151|     %b_scale = ttng.tmem_alloc : () -> !ttg.memdesc<64x8xf8E4M3FN, #tmem_scales, #ttng.tensor_memory, mutable>
152|     %bar = ttg.local_alloc {allocation.offset = 8192 : i32} : () -> !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory, mutable>
153|     ttng.tc_gen5_mma_scaled %a, %b, %d, %a_scale, %b_scale, %true, %true lhs = e2m1 rhs = e2m1, %bar[%true] {is_async} : !ttg.memdesc<128x256xi8, #shared, #ttg.shared_memory, mutable>, !ttg.memdesc<256x64xi8, #sharedT, #ttg.shared_memory, mutable>, !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.memdesc<128x8xf8E4M3FN, #tmem_scales, #ttng.tensor_memory, mutable>, !ttg.memdesc<64x8xf8E4M3FN, #tmem_scales, #ttng.tensor_memory, mutable>, !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory, mutable>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining shared/local memory allocation, tensor-memory allocation, constants, scaled tensor-core MMA ops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 共享/本地内存分配、张量内存分配、常量、带缩放的张量核 MMA 操作。

### Lines 154-157
```mlir
154|     tt.return
155|   }
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

### Lines 159-161
```mlir
159| 
160| #blocked = #ttg.blocked<{sizePerThread = [1, 64], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
161| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 162-162
```mlir
162| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.tensor_memory_size = 0 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 163-163
```mlir
163|   // CHECK-LABEL: @tmem_memdesc_index
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tmem_memdesc_index anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tmem_memdesc_index 这样的标签用于锚定匹配范围。

### Lines 164-164
```mlir
164|   tt.func public @tmem_memdesc_index() {
```
**EN:** This function-oriented block defines or enters `tmem_memdesc_index`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tmem_memdesc_index` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 165-179
```mlir
165|     // CHECK: ttg.global_scratch_alloc
166|     // CHECK-NOT: ttng.tmem_load
167|     // CHECK-NOT: ttng.tmem_store
168|     // CHECK-NOT: ttg.memdesc_index
169|     %true = arith.constant true
170|     %c0_i32 = arith.constant 0 : i32
171|     %zero = arith.constant dense<0.0> : tensor<128x128xf32, #blocked>
172|     %buf = ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<2x128x128xf32, #tmem, #ttng.tensor_memory, mutable>
173|     %view = ttg.memdesc_index %buf[%c0_i32] : !ttg.memdesc<2x128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
174|     ttng.tmem_store %zero, %view, %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
175|     %val = ttng.tmem_load %view : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
176|     tt.return
177|   }
178| }
179| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants, tensor-memory allocation, ttg.memdesc_index, ttng.tmem_store, ttng.tmem_load. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量、张量内存分配、ttg.memdesc_index、ttng.tmem_store、ttng.tmem_load。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 180-180
```mlir
180| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 181-183
```mlir
181| 
182| #blocked = #ttg.blocked<{sizePerThread = [1, 64], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
183| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 184-184
```mlir
184| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.tensor_memory_size = 0 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 185-185
```mlir
185|   // CHECK-LABEL: @tmem_memdesc_index_nonzero
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tmem_memdesc_index_nonzero anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tmem_memdesc_index_nonzero 这样的标签用于锚定匹配范围。

### Lines 186-186
```mlir
186|   tt.func public @tmem_memdesc_index_nonzero() {
```
**EN:** This function-oriented block defines or enters `tmem_memdesc_index_nonzero`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tmem_memdesc_index_nonzero` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 187-200
```mlir
187|     // CHECK: ttg.global_scratch_alloc
188|     // CHECK-NOT: ttng.tmem_load
189|     // CHECK-NOT: ttng.tmem_store
190|     %true = arith.constant true
191|     %c1_i32 = arith.constant 1 : i32
192|     %zero = arith.constant dense<0.0> : tensor<128x128xf32, #blocked>
193|     %buf = ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<2x128x128xf32, #tmem, #ttng.tensor_memory, mutable>
194|     %view = ttg.memdesc_index %buf[%c1_i32] : !ttg.memdesc<2x128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
195|     ttng.tmem_store %zero, %view, %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
196|     %val = ttng.tmem_load %view : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
197|     tt.return
198|   }
199| }
200| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants, tensor-memory allocation, ttg.memdesc_index, ttng.tmem_store, ttng.tmem_load. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量、张量内存分配、ttg.memdesc_index、ttng.tmem_store、ttng.tmem_load。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 201-201
```mlir
201| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 202-204
```mlir
202| 
203| #blocked = #ttg.blocked<{sizePerThread = [1, 64], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
204| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 205-205
```mlir
205| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.tensor_memory_size = 0 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 206-206
```mlir
206|   // CHECK-LABEL: @tmem_reinterpret
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tmem_reinterpret anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tmem_reinterpret 这样的标签用于锚定匹配范围。

### Lines 207-207
```mlir
207|   tt.func public @tmem_reinterpret() {
```
**EN:** This function-oriented block defines or enters `tmem_reinterpret`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tmem_reinterpret` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 208-220
```mlir
208|     // CHECK: ttg.global_scratch_alloc
209|     // CHECK-NOT: ttng.tmem_load
210|     // CHECK-NOT: ttng.tmem_store
211|     %true = arith.constant true
212|     %zero = arith.constant dense<0.0> : tensor<128x128xf32, #blocked>
213|     %buf = ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
214|     %view = ttg.memdesc_reinterpret %buf : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
215|     ttng.tmem_store %zero, %view, %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
216|     %val = ttng.tmem_load %view : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
217|     tt.return
218|   }
219| }
220| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants, tensor-memory allocation, ttg.memdesc_reinterpret, ttng.tmem_store, ttng.tmem_load. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量、张量内存分配、ttg.memdesc_reinterpret、ttng.tmem_store、ttng.tmem_load。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 221-221
```mlir
221| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 222-227
```mlir
222| 
223| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
224| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
225| #smem = #ttg.shared_memory
226| #blocked = #ttg.blocked<{sizePerThread = [1, 64], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
227| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 228-228
```mlir
228| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.tensor_memory_size = 0 : i32, "ttg.total-num-warps" = 8 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 229-229
```mlir
229|   // CHECK-LABEL: @ws_partition_tmem_load
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @ws_partition_tmem_load anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @ws_partition_tmem_load 这样的标签用于锚定匹配范围。

### Lines 230-230
```mlir
230|   tt.func public @ws_partition_tmem_load() {
```
**EN:** This function-oriented block defines or enters `ws_partition_tmem_load`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `ws_partition_tmem_load` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 231-248
```mlir
231|     // CHECK: %[[SCRATCH:.*]] = ttg.global_scratch_alloc
232|     // CHECK: ttg.warp_specialize(%{{.*}}, %{{.*}}, %{{.*}}, %[[SCRATCH]])
233|     // CHECK: partition0(%{{.*}}: !ttg.memdesc<1xi64, #{{[^,>]+}}, #smem, mutable>, %{{.*}}: !ttg.memdesc<128x128xf32, #{{[^,>]+}}, #smem, mutable>, %{{.*}}: !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, %[[SCRATCH_ARG:.*]]: !tt.ptr<i32>) num_warps(4)
234|     // CHECK: %[[PTRS:.*]] = tt.splat %[[SCRATCH_ARG]] : !tt.ptr<i32> -> tensor<128x128x!tt.ptr<i32>, #blocked>
235|     // CHECK: tt.load
236|     // CHECK: ttg.local_store
237|     // CHECK-NOT: ttng.tmem_load
238|     %bar = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
239|     %smem = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<128x128xf32, #shared, #smem, mutable>
240|     %buf = ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
241|     ttg.warp_specialize(%bar, %smem, %buf) attributes {actualRegisters = array<i32: 32, 32>, allocation.offset = 512 : i32, requestedRegisters = array<i32: 32>, warpGroupStartIds = array<i32: 4>}
242|     default {
243|       ttg.warp_yield
244|     }
245|     partition0(%arg0: !ttg.memdesc<1xi64, #shared1, #smem, mutable>, %arg1: !ttg.memdesc<128x128xf32, #shared, #smem, mutable>, %arg2: !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>) num_warps(4) {
246|       %val = ttng.tmem_load %arg2 : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
247|       ttg.local_store %val, %arg1 : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #shared, #smem, mutable>
248|       ttg.warp_return
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, tensor-memory allocation, ttg.warp_specialize, ttg.warp_yield, ttng.tmem_load. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、张量内存分配、ttg.warp_specialize、ttg.warp_yield、ttng.tmem_load。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 249-252
```mlir
249|     } : (!ttg.memdesc<1xi64, #shared1, #smem, mutable>, !ttg.memdesc<128x128xf32, #shared, #smem, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>) -> ()
250|     tt.return
251|   }
252| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-allow-unregistered-dialect`, `-tritoninstrument-fp-sanitizer`
- **CN:** 主要 pass 选项：`-split-input-file`，`-allow-unregistered-dialect`，`-tritoninstrument-fp-sanitizer`
- **EN:** Dominant operations include `tt.func`, `arith.constant`, `ttng.tmem_alloc`, `ttg.local_alloc`, `module`, `tt.return`, `ttng.tmem_store`, `ttng.tmem_load`, `ttng.tmem_subslice`, `ttng.tc_gen5_mma`.
- **CN:** 主要操作包括 `tt.func`、`arith.constant`、`ttng.tmem_alloc`、`ttg.local_alloc`、`module`、`tt.return`、`ttng.tmem_store`、`ttng.tmem_load`、`ttng.tmem_subslice`、`ttng.tc_gen5_mma`。
- **EN:** The file contains 9 independently testable section(s). Check styles used: CHECK x26, CHECK-NOT x18, CHECK-NEXT x15, CHECK-LABEL x10. Important labels include @tmem_load_store, @tmem_copy_subslice, @tmem_subslice_load_store, @tcgen05_mma. It validates both the presence of the desired rewrite and the absence of unwanted residual operations.
- **CN:** 该文件包含 9 个可独立测试的分段。使用的检查类型：CHECK ×26，CHECK-NOT ×18，CHECK-NEXT ×15，CHECK-LABEL ×10。 关键标签包括 @tmem_load_store，@tmem_copy_subslice，@tmem_subslice_load_store，@tcgen05_mma。 它同时验证目标重写已经出现，并且不再残留不需要的操作。

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