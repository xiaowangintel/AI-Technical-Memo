# fpsan.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/fpsan.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-tritoninstrument-fp-sanitizer, -canonicalize` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-tritoninstrument-fp-sanitizer, -canonicalize` 的行为。
- **EN:** RUN pipeline(s): `// RUN: split-file %s %t`; `// RUN: triton-opt %t/success.mlir -split-input-file -tritoninstrument-fp-sanitizer | FileCheck %t/success.mlir`; `// RUN: triton-opt %t/canonicalize.mlir -canonicalize | FileCheck %t/canonicalize.mlir`; `// RUN: not triton-opt %t/unsupported.mlir -tritoninstrument-fp-sanitizer 2>&1 | FileCheck %t/unsupported.mlir --check-prefix=FPSANERR`
- **CN:** RUN 流水线：`// RUN: split-file %s %t`；`// RUN: triton-opt %t/success.mlir -split-input-file -tritoninstrument-fp-sanitizer | FileCheck %t/success.mlir`；`// RUN: triton-opt %t/canonicalize.mlir -canonicalize | FileCheck %t/canonicalize.mlir`；`// RUN: not triton-opt %t/unsupported.mlir -tritoninstrument-fp-sanitizer 2>&1 | FileCheck %t/unsupported.mlir --check-prefix=FPSANERR`

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```mlir
1| // RUN: split-file %s %t
2| // RUN: triton-opt %t/success.mlir -split-input-file -tritoninstrument-fp-sanitizer | FileCheck %t/success.mlir
3| // RUN: triton-opt %t/canonicalize.mlir -canonicalize | FileCheck %t/canonicalize.mlir
4| // RUN: not triton-opt %t/unsupported.mlir -tritoninstrument-fp-sanitizer 2>&1 | FileCheck %t/unsupported.mlir --check-prefix=FPSANERR
5| 
6| //--- success.mlir
7| 
```
**EN:** This header defines how the test is executed. It runs `split-file %s %t` ; ` triton-opt %t/success.mlir -split-input-file -tritoninstrument-fp-sanitizer | FileCheck %t/success.mlir` ; ` triton-opt %t/canonicalize.mlir -canonicalize | FileCheck %t/canonicalize.mlir` ; ` not triton-opt %t/unsupported.mlir -tritoninstrument-fp-sanitizer 2>&1 | FileCheck %t/unsupported.mlir --check-prefix=FPSANERR` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `split-file %s %t`；` triton-opt %t/success.mlir -split-input-file -tritoninstrument-fp-sanitizer | FileCheck %t/success.mlir`；` triton-opt %t/canonicalize.mlir -canonicalize | FileCheck %t/canonicalize.mlir`；` not triton-opt %t/unsupported.mlir -tritoninstrument-fp-sanitizer 2>&1 | FileCheck %t/unsupported.mlir --check-prefix=FPSANERR`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 8-10
```mlir
 8| #blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [64, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
 9| #dot_operand_a = #ttg.dot_op<{opIdx = 0, parent = #blocked}>
10| #dot_operand_b = #ttg.dot_op<{opIdx = 1, parent = #blocked}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 11-11
```mlir
11| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 12-12
```mlir
12|   // CHECK-LABEL: @dot_emulation
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @dot_emulation anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @dot_emulation 这样的标签用于锚定匹配范围。

### Lines 13-13
```mlir
13|   tt.func public @dot_emulation() -> tensor<16x16xf32, #blocked> {
```
**EN:** This function-oriented block defines or enters `dot_emulation`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `dot_emulation` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 14-25
```mlir
14|     // CHECK: scf.for
15|     // CHECK-NOT: tt.dot
16|     // CHECK-NOT: ttg.convert_layout
17|     %cst = arith.constant 1.000000e+00 : f16
18|     %zero = arith.constant dense<0.000000e+00> : tensor<16x16xf32, #blocked>
19|     %a = tt.splat %cst : f16 -> tensor<16x16xf16, #dot_operand_a>
20|     %b = tt.splat %cst : f16 -> tensor<16x16xf16, #dot_operand_b>
21|     %out = tt.dot %a, %b, %zero : tensor<16x16xf16, #dot_operand_a> * tensor<16x16xf16, #dot_operand_b> -> tensor<16x16xf32, #blocked>
22|     tt.return %out : tensor<16x16xf32, #blocked>
23|   }
24| }
25| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining constants, broadcasted scalars or pointers, dot-product or MMA-style math, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 常量、广播后的标量或指针、点积或 MMA 风格计算、tt.return。

### Lines 26-26
```mlir
26| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 27-30
```mlir
27| 
28| #blocked = #ttg.blocked<{sizePerThread = [1, 1, 1], threadsPerWarp = [1, 32, 1], warpsPerCTA = [1, 4, 1], order = [2, 1, 0]}>
29| #dot_operand_a = #ttg.dot_op<{opIdx = 0, parent = #blocked}>
30| #dot_operand_b = #ttg.dot_op<{opIdx = 1, parent = #blocked}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 31-31
```mlir
31| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 32-32
```mlir
32|   // CHECK-LABEL: @rank3_dot_emulation
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @rank3_dot_emulation anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @rank3_dot_emulation 这样的标签用于锚定匹配范围。

### Lines 33-33
```mlir
33|   tt.func public @rank3_dot_emulation() -> tensor<2x16x16xf32, #blocked> {
```
**EN:** This function-oriented block defines or enters `rank3_dot_emulation`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `rank3_dot_emulation` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 34-46
```mlir
34|     // CHECK: scf.for
35|     // CHECK: scf.for
36|     // CHECK-NOT: tt.dot
37|     // CHECK-NOT: ttg.convert_layout
38|     %one = arith.constant 1.000000e+00 : f16
39|     %zero = arith.constant dense<0.000000e+00> : tensor<2x16x16xf32, #blocked>
40|     %a = tt.splat %one : f16 -> tensor<2x16x16xf16, #dot_operand_a>
41|     %b = tt.splat %one : f16 -> tensor<2x16x16xf16, #dot_operand_b>
42|     %out = tt.dot %a, %b, %zero : tensor<2x16x16xf16, #dot_operand_a> * tensor<2x16x16xf16, #dot_operand_b> -> tensor<2x16x16xf32, #blocked>
43|     tt.return %out : tensor<2x16x16xf32, #blocked>
44|   }
45| }
46| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining constants, broadcasted scalars or pointers, dot-product or MMA-style math, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 常量、广播后的标量或指针、点积或 MMA 风格计算、tt.return。

### Lines 47-47
```mlir
47| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 48-51
```mlir
48| 
49| #blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [32, 2], warpsPerCTA = [4, 1], order = [0, 1]}>
50| #dot_A = #ttg.dot_op<{opIdx = 0, parent = #blocked}>
51| #dot_B = #ttg.dot_op<{opIdx = 1, parent = #blocked}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 52-52
```mlir
52| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 53-53
```mlir
53|   // CHECK-LABEL: @dot_scaled_emulation
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @dot_scaled_emulation anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @dot_scaled_emulation 这样的标签用于锚定匹配范围。

### Lines 54-54
```mlir
54|   tt.func public @dot_scaled_emulation() -> tensor<16x16xf32, #blocked> {
```
**EN:** This function-oriented block defines or enters `dot_scaled_emulation`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `dot_scaled_emulation` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 55-68
```mlir
55|     // CHECK: ttg.barrier global_read|global_write
56|     // CHECK: scf.for
57|     // CHECK: ttg.barrier global_read|global_write
58|     // CHECK-NOT: ttg.dot_scaled
59|     // CHECK-NOT: ttg.convert_layout
60|      %cst = arith.constant 1.000000e+00 : f16
61|      %zero = arith.constant dense<0.000000e+00> : tensor<16x16xf32, #blocked>
62|      %a = tt.splat %cst : f16 -> tensor<16x16xf16, #dot_A>
63|      %b = tt.splat %cst : f16 -> tensor<16x16xf16, #dot_B>
64|      %out = tt.dot_scaled %a, %b, %zero lhs = fp16 rhs = fp16 {fastMath = false} : tensor<16x16xf16, #dot_A> * tensor<16x16xf16, #dot_B> -> tensor<16x16xf32, #blocked>
65|      tt.return %out : tensor<16x16xf32, #blocked>
66|   }
67| }
68| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining constants, broadcasted scalars or pointers, tt.dot_scaled, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 常量、广播后的标量或指针、tt.dot_scaled、tt.return。

### Lines 69-69
```mlir
69| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 70-73
```mlir
70| 
71| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 32, 16]}>
72| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
73| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 74-74
```mlir
74| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 75-75
```mlir
75|   // CHECK-LABEL: @warp_group_dot_emulation
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @warp_group_dot_emulation anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @warp_group_dot_emulation 这样的标签用于锚定匹配范围。

### Lines 76-76
```mlir
76|   tt.func public @warp_group_dot_emulation() -> tensor<64x32xf32, #mma> {
```
**EN:** This function-oriented block defines or enters `warp_group_dot_emulation`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `warp_group_dot_emulation` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 77-94
```mlir
77|     // CHECK: ttg.local_load
78|     // CHECK: tti.experimental_fpsan_embed
79|     // CHECK: tt.store
80|     // CHECK: ttg.barrier global_read|global_write
81|     // CHECK: scf.for
82|     // CHECK: ttg.barrier global_read|global_write
83|     // CHECK: %[[RAW:.*]] = tt.load
84|     // CHECK: %[[OUT:.*]] = tti.experimental_fpsan_unembed %[[RAW]]
85|     // CHECK-NOT: ttng.warp_group_dot {{.*}} :
86|     // CHECK: ttng.warp_group_dot_wait %[[OUT]]
87|     %a = ttg.local_alloc : () -> !ttg.memdesc<64x32xf32, #shared, #smem, mutable>
88|     %b = ttg.local_alloc : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
89|     %c = arith.constant dense<0.000000e+00> : tensor<64x32xf32, #mma>
90|     %true = arith.constant true
91|     %d = ttng.warp_group_dot %a, %b, %c, %true {inputPrecision = 1 : i32, isAsync = true} : !ttg.memdesc<64x32xf32, #shared, #smem, mutable> * !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<64x32xf32, #mma>
92|     %wait = ttng.warp_group_dot_wait %d {pendings = 0 : i32} : tensor<64x32xf32, #mma>
93|     tt.return %wait : tensor<64x32xf32, #mma>
94|   }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining shared/local memory allocation, constants, ttng.warp_group_dot, ttng.warp_group_dot_wait, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 共享/本地内存分配、常量、ttng.warp_group_dot、ttng.warp_group_dot_wait、tt.return。

### Lines 95-96
```mlir
95| }
96| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 97-97
```mlir
97| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 98-100
```mlir
 98| 
 99| #tmem_linear = #ttg.linear<{register = [[0, 1], [0, 2], [0, 4], [0, 8], [0, 16], [0, 32]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0]], warp = [[32, 0], [64, 0]], block = []}>
100| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 64, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 101-101
```mlir
101| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 102-102
```mlir
102|   // CHECK-LABEL: @tmem_scratch_payloads
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tmem_scratch_payloads anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tmem_scratch_payloads 这样的标签用于锚定匹配范围。

### Lines 103-103
```mlir
103|   tt.func public @tmem_scratch_payloads(%arg0: tensor<128x64xf32, #tmem_linear>) -> tensor<128x64xf32, #tmem_linear> {
```
**EN:** This function-oriented block defines or enters `tmem_scratch_payloads`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tmem_scratch_payloads` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 104-118
```mlir
104|     // CHECK: %[[PAYLOAD:.*]] = tti.experimental_fpsan_embed %arg0
105|     // CHECK: tt.store {{.*}}, %[[PAYLOAD]]
106|     // CHECK: %[[RAW:.*]] = tt.load
107|     // CHECK: %[[OUT:.*]] = tti.experimental_fpsan_unembed %[[RAW]]
108|     // CHECK: tt.return %[[OUT]]
109|     // CHECK-NOT: ttng.tmem_store
110|     // CHECK-NOT: ttng.tmem_load
111|     %true = arith.constant true
112|     %tmem = ttng.tmem_alloc : () -> !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory, mutable>
113|     ttng.tmem_store %arg0, %tmem, %true : tensor<128x64xf32, #tmem_linear> -> !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory, mutable>
114|     %out = ttng.tmem_load %tmem : !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x64xf32, #tmem_linear>
115|     tt.return %out : tensor<128x64xf32, #tmem_linear>
116|   }
117| }
118| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants, tensor-memory allocation, ttng.tmem_store, ttng.tmem_load, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量、张量内存分配、ttng.tmem_store、ttng.tmem_load、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 119-119
```mlir
119| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 120-121
```mlir
120| 
121| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 122-122
```mlir
122|   // CHECK-LABEL: @binary_ops
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @binary_ops anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @binary_ops 这样的标签用于锚定匹配范围。

### Lines 123-123
```mlir
123|   tt.func public @binary_ops(%a: tensor<4xf32>, %b: tensor<4xf32>) -> tensor<4xf32> {
```
**EN:** This function-oriented block defines or enters `binary_ops`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `binary_ops` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 124-139
```mlir
124|     // CHECK: tti.experimental_fpsan_embed
125|     // CHECK: arith.addi
126|     // CHECK: arith.subi
127|     // CHECK: arith.muli
128|     // CHECK-NOT: arith.addf
129|     // CHECK-NOT: arith.subf
130|     // CHECK-NOT: arith.mulf
131|     %add = arith.addf %a, %b : tensor<4xf32>
132|     %sub = arith.subf %a, %b : tensor<4xf32>
133|     %mul = arith.mulf %a, %b : tensor<4xf32>
134|     %sum = arith.addf %add, %sub : tensor<4xf32>
135|     %out = arith.mulf %sum, %mul : tensor<4xf32>
136|     tt.return %out : tensor<4xf32>
137|   }
138| }
139| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on floating-point additions, arith.mulf, arith.subf, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 浮点加法、arith.mulf、arith.subf、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 140-140
```mlir
140| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 141-142
```mlir
141| 
142| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 143-143
```mlir
143|   // CHECK-LABEL: @neg_op
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @neg_op anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @neg_op 这样的标签用于锚定匹配范围。

### Lines 144-144
```mlir
144|   tt.func public @neg_op(%a: tensor<4xf32>) -> tensor<4xf32> {
```
**EN:** This function-oriented block defines or enters `neg_op`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `neg_op` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 145-154
```mlir
145|     // CHECK-DAG: %[[A:.*]] = tti.experimental_fpsan_embed %arg0 : (tensor<4xf32>) -> tensor<4xi32>
146|     // CHECK-DAG: %[[ZERO:.*]] = arith.constant dense<0> : tensor<4xi32>
147|     // CHECK: %[[NEG:.*]] = arith.subi %[[ZERO]], %[[A]] : tensor<4xi32>
148|     // CHECK: %[[OUT:.*]] = tti.experimental_fpsan_unembed %[[NEG]] : (tensor<4xi32>) -> tensor<4xf32>
149|     // CHECK-NOT: arith.negf
150|     %neg = arith.negf %a : tensor<4xf32>
151|     tt.return %neg : tensor<4xf32>
152|   }
153| }
154| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on arith.negf, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 arith.negf、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 155-155
```mlir
155| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 156-157
```mlir
156| 
157| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 158-158
```mlir
158|   // CHECK-LABEL: @chained_ops
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @chained_ops anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @chained_ops 这样的标签用于锚定匹配范围。

### Lines 159-159
```mlir
159|   tt.func public @chained_ops(%a: tensor<4xf32>, %b: tensor<4xf32>, %c: tensor<4xf32>) -> tensor<4xf32> {
```
**EN:** This function-oriented block defines or enters `chained_ops`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `chained_ops` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 160-172
```mlir
160|     // CHECK: %[[A:.*]] = tti.experimental_fpsan_embed %arg0 : (tensor<4xf32>) -> tensor<4xi32>
161|     // CHECK: %[[B:.*]] = tti.experimental_fpsan_embed %arg1 : (tensor<4xf32>) -> tensor<4xi32>
162|     // CHECK: %[[SUM0:.*]] = arith.addi %[[A]], %[[B]] : tensor<4xi32>
163|     // CHECK: %[[C:.*]] = tti.experimental_fpsan_embed %arg2 : (tensor<4xf32>) -> tensor<4xi32>
164|     // CHECK: %[[SUM1:.*]] = arith.addi %[[SUM0]], %[[C]] : tensor<4xi32>
165|     // CHECK: %[[OUT:.*]] = tti.experimental_fpsan_unembed %[[SUM1]] : (tensor<4xi32>) -> tensor<4xf32>
166|     // CHECK: tt.return %[[OUT]] : tensor<4xf32>
167|     %sum0 = arith.addf %a, %b : tensor<4xf32>
168|     %sum1 = arith.addf %sum0, %c : tensor<4xf32>
169|     tt.return %sum1 : tensor<4xf32>
170|   }
171| }
172| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on floating-point additions, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 浮点加法、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 173-173
```mlir
173| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 174-175
```mlir
174| 
175| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 176-176
```mlir
176|   // CHECK-LABEL: @div_rem_ops
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @div_rem_ops anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @div_rem_ops 这样的标签用于锚定匹配范围。

### Lines 177-177
```mlir
177|   tt.func public @div_rem_ops(%a: tensor<4xf32>, %b: tensor<4xf32>) -> tensor<4xf32> {
```
**EN:** This function-oriented block defines or enters `div_rem_ops`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `div_rem_ops` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 178-188
```mlir
178|     // CHECK: tti.experimental_fpsan_embed
179|     // CHECK: arith.muli
180|     // CHECK-NOT: arith.divf
181|     // CHECK-NOT: arith.remf
182|     %div = arith.divf %a, %b : tensor<4xf32>
183|     %rem = arith.remf %a, %b : tensor<4xf32>
184|     %out = arith.addf %div, %rem : tensor<4xf32>
185|     tt.return %out : tensor<4xf32>
186|   }
187| }
188| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on arith.divf, arith.remf, floating-point additions, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 arith.divf、arith.remf、浮点加法、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 189-189
```mlir
189| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 190-191
```mlir
190| 
191| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 192-192
```mlir
192|   // CHECK-LABEL: @fma_op
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @fma_op anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @fma_op 这样的标签用于锚定匹配范围。

### Lines 193-193
```mlir
193|   tt.func public @fma_op(%a: tensor<4xf32>, %b: tensor<4xf32>, %c: tensor<4xf32>) -> tensor<4xf32> {
```
**EN:** This function-oriented block defines or enters `fma_op`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fma_op` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 194-201
```mlir
194|     // CHECK: arith.muli
195|     // CHECK: arith.addi
196|     // CHECK-NOT: math.fma
197|     %fma = math.fma %a, %b, %c : tensor<4xf32>
198|     tt.return %fma : tensor<4xf32>
199|   }
200| }
201| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on math.fma, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 math.fma、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 202-202
```mlir
202| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 203-204
```mlir
203| 
204| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 205-205
```mlir
205|   // CHECK-LABEL: @unary_ops
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @unary_ops anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @unary_ops 这样的标签用于锚定匹配范围。

### Lines 206-206
```mlir
206|   tt.func public @unary_ops(%a: tensor<4xf32>) -> tensor<4xf32> {
```
**EN:** This function-oriented block defines or enters `unary_ops`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `unary_ops` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 207-219
```mlir
207|     // CHECK-DAG: arith.constant dense<314159>
208|     // CHECK: tti.experimental_fpsan_embed
209|     // CHECK: arith.muli
210|     // CHECK: arith.xori
211|     // CHECK: arith.xori
212|     // CHECK-NOT: math.log
213|     // CHECK-NOT: math.sqrt
214|     %l = math.log %a : tensor<4xf32>
215|     %s = math.sqrt %l : tensor<4xf32>
216|     tt.return %s : tensor<4xf32>
217|   }
218| }
219| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on math.log, math.sqrt, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 math.log、math.sqrt、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 220-220
```mlir
220| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 221-222
```mlir
221| 
222| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 223-223
```mlir
223|   // CHECK-LABEL: @exp_ops
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @exp_ops anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @exp_ops 这样的标签用于锚定匹配范围。

### Lines 224-224
```mlir
224|   tt.func public @exp_ops(%a: tensor<4xf32>) -> (tensor<4xf32>, tensor<4xf32>) {
```
**EN:** This function-oriented block defines or enters `exp_ops`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `exp_ops` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 225-241
```mlir
225|     // CHECK-DAG: arith.constant dense<594471359>
226|     // CHECK-DAG: arith.constant dense<1>
227|     // CHECK-DAG: arith.constant dense<0>
228|     // CHECK-DAG: arith.constant dense<-1555856531>
229|     // CHECK: tti.experimental_fpsan_embed
230|     // CHECK: arith.muli
231|     // CHECK: arith.andi
232|     // CHECK: arith.cmpi
233|     // CHECK: arith.select
234|     // CHECK-NOT: math.exp
235|     // CHECK-NOT: math.exp2
236|     %0 = math.exp %a : tensor<4xf32>
237|     %1 = math.exp2 %a : tensor<4xf32>
238|     tt.return %0, %1 : tensor<4xf32>, tensor<4xf32>
239|   }
240| }
241| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on math.exp, math.exp2, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 math.exp、math.exp2、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 242-242
```mlir
242| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 243-244
```mlir
243| 
244| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 245-245
```mlir
245|   // CHECK-LABEL: @cast_extf
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @cast_extf anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @cast_extf 这样的标签用于锚定匹配范围。

### Lines 246-246
```mlir
246|   tt.func public @cast_extf(%a: tensor<4xf16>) -> tensor<4xf32> {
```
**EN:** This function-oriented block defines or enters `cast_extf`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `cast_extf` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 247-254
```mlir
247|     // CHECK: tti.experimental_fpsan_embed
248|     // CHECK: arith.extsi
249|     // CHECK-NOT: arith.extf
250|     %0 = arith.extf %a : tensor<4xf16> to tensor<4xf32>
251|     tt.return %0 : tensor<4xf32>
252|   }
253| }
254| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on arith.extf, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 arith.extf、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 255-255
```mlir
255| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 256-257
```mlir
256| 
257| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 258-258
```mlir
258|   // CHECK-LABEL: @cast_truncf
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @cast_truncf anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @cast_truncf 这样的标签用于锚定匹配范围。

### Lines 259-259
```mlir
259|   tt.func public @cast_truncf(%a: tensor<4xf32>) -> tensor<4xf16> {
```
**EN:** This function-oriented block defines or enters `cast_truncf`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `cast_truncf` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 260-267
```mlir
260|     // CHECK: tti.experimental_fpsan_embed
261|     // CHECK: arith.trunci
262|     // CHECK-NOT: arith.truncf
263|     %0 = arith.truncf %a : tensor<4xf32> to tensor<4xf16>
264|     tt.return %0 : tensor<4xf16>
265|   }
266| }
267| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on arith.truncf, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 arith.truncf、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 268-268
```mlir
268| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 269-270
```mlir
269| 
270| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 271-271
```mlir
271|   // CHECK-LABEL: @cast_fp_to_fp
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @cast_fp_to_fp anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @cast_fp_to_fp 这样的标签用于锚定匹配范围。

### Lines 272-272
```mlir
272|   tt.func public @cast_fp_to_fp(%a: tensor<4xf8E4M3FN>) -> tensor<4xf16> {
```
**EN:** This function-oriented block defines or enters `cast_fp_to_fp`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `cast_fp_to_fp` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 273-280
```mlir
273|     // CHECK: tti.experimental_fpsan_embed
274|     // CHECK: arith.extsi
275|     // CHECK-NOT: tt.fp_to_fp
276|     %0 = tt.fp_to_fp %a : tensor<4xf8E4M3FN> -> tensor<4xf16>
277|     tt.return %0 : tensor<4xf16>
278|   }
279| }
280| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.fp_to_fp, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.fp_to_fp、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 281-281
```mlir
281| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 282-283
```mlir
282| 
283| #blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 284-284
```mlir
284| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 285-285
```mlir
285|   // CHECK-LABEL: @cast_fp4_to_fp
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @cast_fp4_to_fp anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @cast_fp4_to_fp 这样的标签用于锚定匹配范围。

### Lines 286-286
```mlir
286|   tt.func public @cast_fp4_to_fp(%a: tensor<16x8xi8, #blocked>) -> tensor<16x16xf16, #blocked> {
```
**EN:** This function-oriented block defines or enters `cast_fp4_to_fp`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `cast_fp4_to_fp` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 287-297
```mlir
287|     // CHECK: arith.andi
288|     // CHECK: arith.shrui
289|     // CHECK: tt.join
290|     // CHECK: tt.reshape
291|     // CHECK-NOT: tt.trans
292|     // CHECK-NOT: ttg.fp4_to_fp
293|     %0 = ttg.fp4_to_fp %a {axis = 1 : i32} : tensor<16x8xi8, #blocked> -> tensor<16x16xf16, #blocked>
294|     tt.return %0 : tensor<16x16xf16, #blocked>
295|   }
296| }
297| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.fp4_to_fp, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.fp4_to_fp、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 298-298
```mlir
298| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 299-300
```mlir
299| 
300| // CHECK-LABEL: @extern_unary
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @extern_unary anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @extern_unary 这样的标签用于锚定匹配范围。

### Lines 301-301
```mlir
301| tt.func public @extern_unary(%a: tensor<4xf32>) -> tensor<4xf32> {
```
**EN:** This function-oriented block defines or enters `extern_unary`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `extern_unary` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 302-308
```mlir
302|   // CHECK: tti.experimental_fpsan_embed
303|   // CHECK: arith.xori
304|   // CHECK-NOT: tt.extern_elementwise
305|   %0 = tt.extern_elementwise %a {libname = "", libpath = "", pure = true, symbol = "__nv_tanf"} : (tensor<4xf32>) -> tensor<4xf32>
306|   tt.return %0 : tensor<4xf32>
307| }
308| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.extern_elementwise, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.extern_elementwise、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 309-309
```mlir
309| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 310-311
```mlir
310| 
311| // CHECK-LABEL: @extern_binary
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @extern_binary anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @extern_binary 这样的标签用于锚定匹配范围。

### Lines 312-312
```mlir
312| tt.func public @extern_binary(%a: tensor<4xf32>, %b: tensor<4xf32>) -> tensor<4xf32> {
```
**EN:** This function-oriented block defines or enters `extern_binary`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `extern_binary` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 313-320
```mlir
313|   // CHECK: tti.experimental_fpsan_embed
314|   // CHECK: arith.addi
315|   // CHECK: arith.xori
316|   // CHECK-NOT: tt.extern_elementwise
317|   %0 = tt.extern_elementwise %a, %b {libname = "", libpath = "", pure = true, symbol = "__nv_atan2f"} : (tensor<4xf32>, tensor<4xf32>) -> tensor<4xf32>
318|   tt.return %0 : tensor<4xf32>
319| }
320| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.extern_elementwise, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.extern_elementwise、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 321-321
```mlir
321| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 322-323
```mlir
322| 
323| // CHECK-LABEL: @extern_ternary
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @extern_ternary anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @extern_ternary 这样的标签用于锚定匹配范围。

### Lines 324-324
```mlir
324| tt.func public @extern_ternary(%a: tensor<4xf32>, %b: tensor<4xf32>, %c: tensor<4xf32>) -> tensor<4xf32> {
```
**EN:** This function-oriented block defines or enters `extern_ternary`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `extern_ternary` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 325-332
```mlir
325|   // CHECK: tti.experimental_fpsan_embed
326|   // CHECK: arith.addi
327|   // CHECK: arith.xori
328|   // CHECK-NOT: tt.extern_elementwise
329|   %0 = tt.extern_elementwise %a, %b, %c {libname = "", libpath = "", pure = true, symbol = "__nv_fmaf"} : (tensor<4xf32>, tensor<4xf32>, tensor<4xf32>) -> tensor<4xf32>
330|   tt.return %0 : tensor<4xf32>
331| }
332| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.extern_elementwise, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.extern_elementwise、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 333-333
```mlir
333| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 334-335
```mlir
334| 
335| // CHECK-LABEL: @extern_mixed
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @extern_mixed anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @extern_mixed 这样的标签用于锚定匹配范围。

### Lines 336-336
```mlir
336| tt.func public @extern_mixed(%a: tensor<4xf32>, %b: tensor<4xi32>) -> tensor<4xf32> {
```
**EN:** This function-oriented block defines or enters `extern_mixed`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `extern_mixed` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 337-346
```mlir
337|   // CHECK: tti.experimental_fpsan_embed
338|   // CHECK: arith.addi
339|   // CHECK: arith.xori
340|   // CHECK-NOT: tt.extern_elementwise
341|   %0 = tt.extern_elementwise %a, %b {libname = "", libpath = "", pure = true, symbol = "__nv_ldexpf"} : (tensor<4xf32>, tensor<4xi32>) -> tensor<4xf32>
342|   tt.return %0 : tensor<4xf32>
343| }
344| 
345| //--- canonicalize.mlir
346| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.extern_elementwise, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.extern_elementwise、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 347-347
```mlir
347| module {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 348-348
```mlir
348|   // CHECK-LABEL: @fold_fpsan_embedding_roundtrips
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @fold_fpsan_embedding_roundtrips anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @fold_fpsan_embedding_roundtrips 这样的标签用于锚定匹配范围。

### Lines 349-349
```mlir
349|   tt.func public @fold_fpsan_embedding_roundtrips(%arg0: tensor<4xi32>, %arg1: tensor<4xf32>) -> (tensor<4xi32>, tensor<4xf32>) {
```
**EN:** This function-oriented block defines or enters `fold_fpsan_embedding_roundtrips`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fold_fpsan_embedding_roundtrips` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 350-359
```mlir
350|     // CHECK-NOT: tti.experimental_fpsan
351|     // CHECK: tt.return %arg0, %arg1
352|     %0 = tti.experimental_fpsan_unembed %arg0 : (tensor<4xi32>) -> tensor<4xf32>
353|     %1 = tti.experimental_fpsan_embed %0 : (tensor<4xf32>) -> tensor<4xi32>
354|     %2 = tti.experimental_fpsan_embed %arg1 : (tensor<4xf32>) -> tensor<4xi32>
355|     %3 = tti.experimental_fpsan_unembed %2 : (tensor<4xi32>) -> tensor<4xf32>
356|     tt.return %1, %3 : tensor<4xi32>, tensor<4xf32>
357|   }
358| }
359| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tti.experimental_fpsan_unembed, tti.experimental_fpsan_embed, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tti.experimental_fpsan_unembed、tti.experimental_fpsan_embed、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 360-360
```mlir
360| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 361-363
```mlir
361| 
362| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
363| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [0, 1]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 364-364
```mlir
364| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 365-365
```mlir
365|   // CHECK-LABEL: @push_unembed_through_views
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @push_unembed_through_views anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @push_unembed_through_views 这样的标签用于锚定匹配范围。

### Lines 366-373
```mlir
366|   tt.func public @push_unembed_through_views(
367|       %arg0: tensor<16x16xi32, #blocked>,
368|       %arg1: tensor<2x4xi32>,
369|       %arg2: tensor<8xi32>,
370|       %arg3: tensor<1x4xi32>,
371|       %arg4: tensor<4xi32>) ->
372|       (tensor<16x16xf32, #blocked1>, tensor<4x2xf32>, tensor<2x4xf32>,
373|        tensor<2x4xf32>, tensor<1x4xf32>) {
```
**EN:** This function-oriented block defines or enters `push_unembed_through_views`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `push_unembed_through_views` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 374-391
```mlir
374|     // CHECK: %[[CVT:.*]] = ttg.convert_layout %arg0 : tensor<16x16xi32, #blocked> -> tensor<16x16xi32, #blocked1>
375|     // CHECK: %[[CVT_OUT:.*]] = tti.experimental_fpsan_unembed %[[CVT]] : (tensor<16x16xi32, #blocked1>) -> tensor<16x16xf32, #blocked1>
376|     // CHECK: %[[TRANS:.*]] = tt.trans %arg1 {order = array<i32: 1, 0>} : tensor<2x4xi32> -> tensor<4x2xi32>
377|     // CHECK: %[[TRANS_OUT:.*]] = tti.experimental_fpsan_unembed %[[TRANS]] : (tensor<4x2xi32>) -> tensor<4x2xf32>
378|     // CHECK: %[[RESHAPE:.*]] = tt.reshape %arg2 : tensor<8xi32> -> tensor<2x4xi32>
379|     // CHECK: %[[RESHAPE_OUT:.*]] = tti.experimental_fpsan_unembed %[[RESHAPE]] : (tensor<2x4xi32>) -> tensor<2x4xf32>
380|     // CHECK: %[[BCAST:.*]] = tt.broadcast %arg3 : tensor<1x4xi32> -> tensor<2x4xi32>
381|     // CHECK: %[[BCAST_OUT:.*]] = tti.experimental_fpsan_unembed %[[BCAST]] : (tensor<2x4xi32>) -> tensor<2x4xf32>
382|     // CHECK: %[[EXPAND:.*]] = tt.expand_dims %arg4 {axis = 0 : i32} : tensor<4xi32> -> tensor<1x4xi32>
383|     // CHECK: %[[EXPAND_OUT:.*]] = tti.experimental_fpsan_unembed %[[EXPAND]] : (tensor<1x4xi32>) -> tensor<1x4xf32>
384|     %0 = tti.experimental_fpsan_unembed %arg0 : (tensor<16x16xi32, #blocked>) -> tensor<16x16xf32, #blocked>
385|     %1 = ttg.convert_layout %0 : tensor<16x16xf32, #blocked> -> tensor<16x16xf32, #blocked1>
386|     %2 = tti.experimental_fpsan_unembed %arg1 : (tensor<2x4xi32>) -> tensor<2x4xf32>
387|     %3 = tt.trans %2 {order = array<i32: 1, 0>} : tensor<2x4xf32> -> tensor<4x2xf32>
388|     %4 = tti.experimental_fpsan_unembed %arg2 : (tensor<8xi32>) -> tensor<8xf32>
389|     %5 = tt.reshape %4 : tensor<8xf32> -> tensor<2x4xf32>
390|     %6 = tti.experimental_fpsan_unembed %arg3 : (tensor<1x4xi32>) -> tensor<1x4xf32>
391|     %7 = tt.broadcast %6 : tensor<1x4xf32> -> tensor<2x4xf32>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tti.experimental_fpsan_unembed, layout conversions, transpose-like layout changes, tensor reshaping, tensor broadcasting. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tti.experimental_fpsan_unembed、布局转换、转置类布局变换、张量重塑、张量广播。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 392-400
```mlir
392|     %8 = tti.experimental_fpsan_unembed %arg4 : (tensor<4xi32>) -> tensor<4xf32>
393|     %9 = tt.expand_dims %8 {axis = 0 : i32} : tensor<4xf32> -> tensor<1x4xf32>
394|     tt.return %1, %3, %5, %7, %9 : tensor<16x16xf32, #blocked1>, tensor<4x2xf32>,
395|                                           tensor<2x4xf32>, tensor<2x4xf32>, tensor<1x4xf32>
396|   }
397| }
398| 
399| //--- unsupported.mlir
400| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tti.experimental_fpsan_unembed, shape expansion, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tti.experimental_fpsan_unembed、形状扩展、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 401-401
```mlir
401| module {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 402-407
```mlir
402|   tt.func public @dot_no_encoding(%a: tensor<16x16xf32>, %b: tensor<16x16xf32>, %c: tensor<16x16xf32>) -> tensor<16x16xf32> {
403|     // FPSANERR: error: 'tt.dot' op unsupported by fpsan
404|     %out = tt.dot %a, %b, %c : tensor<16x16xf32> * tensor<16x16xf32> -> tensor<16x16xf32>
405|     tt.return %out : tensor<16x16xf32>
406|   }
407| }
```
**EN:** This function-oriented block defines or enters `dot_no_encoding`. Within it, the test exercises tt.func, dot-product or MMA-style math, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `dot_no_encoding` 为核心。测试在其中演示 tt.func、点积或 MMA 风格计算、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-tritoninstrument-fp-sanitizer`, `-canonicalize`
- **CN:** 主要 pass 选项：`-split-input-file`，`-tritoninstrument-fp-sanitizer`，`-canonicalize`
- **EN:** Dominant operations include `tt.func`, `tt.return`, `module`, `arith.constant`, `tti.experimental_fpsan_unembed`, `tt.splat`, `arith.addf`, `tt.extern_elementwise`, `tt.dot`, `ttg.local_alloc`.
- **CN:** 主要操作包括 `tt.func`、`tt.return`、`module`、`arith.constant`、`tti.experimental_fpsan_unembed`、`tt.splat`、`arith.addf`、`tt.extern_elementwise`、`tt.dot`、`ttg.local_alloc`。
- **EN:** The file contains 20 independently testable section(s). Check styles used: CHECK x78, CHECK-NOT x30, CHECK-LABEL x22, CHECK-DAG x7. Important labels include @dot_emulation, @rank3_dot_emulation, @dot_scaled_emulation, @warp_group_dot_emulation. It validates both the presence of the desired rewrite and the absence of unwanted residual operations.
- **CN:** 该文件包含 20 个可独立测试的分段。使用的检查类型：CHECK ×78，CHECK-NOT ×30，CHECK-LABEL ×22，CHECK-DAG ×7。 关键标签包括 @dot_emulation，@rank3_dot_emulation，@dot_scaled_emulation，@warp_group_dot_emulation。 它同时验证目标重写已经出现，并且不再残留不需要的操作。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `tt`: Triton high-level tensor/pointer operations.
- **CN:** `tt`：Triton 高层张量/指针操作。
- **EN:** `ttg`: TritonGPU layout, memory, and GPU-specific operations.
- **CN:** `ttg`：TritonGPU 布局、内存和 GPU 专用操作。
- **EN:** `arith`: MLIR arithmetic dialect for scalar/tensor math.
- **CN:** `arith`：用于标量/张量运算的 MLIR arithmetic 方言。
- **EN:** `ttng`: Triton Nvidia GPU tensor-memory and MMA operations.
- **CN:** `ttng`：Triton Nvidia GPU 张量内存与 MMA 操作。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。