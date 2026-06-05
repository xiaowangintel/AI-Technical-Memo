# amd-prepare-if-combining.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/amd/amd-prepare-if-combining.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-tritonamdgpu-prepare-if-combining, -canonicalize` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-tritonamdgpu-prepare-if-combining, -canonicalize` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file -tritonamdgpu-prepare-if-combining | FileCheck %s`; `// RUN: triton-opt %s -split-input-file -tritonamdgpu-prepare-if-combining -canonicalize | FileCheck %s --check-prefix=CANON`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file -tritonamdgpu-prepare-if-combining | FileCheck %s`；`// RUN: triton-opt %s -split-input-file -tritonamdgpu-prepare-if-combining -canonicalize | FileCheck %s --check-prefix=CANON`

## Line-by-Line Analysis / 逐行分析

### Lines 1-4
```mlir
1| // RUN: triton-opt %s -split-input-file -tritonamdgpu-prepare-if-combining | FileCheck %s
2| // RUN: triton-opt %s -split-input-file -tritonamdgpu-prepare-if-combining -canonicalize | FileCheck %s --check-prefix=CANON
3| 
4| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file -tritonamdgpu-prepare-if-combining | FileCheck %s` ; ` triton-opt %s -split-input-file -tritonamdgpu-prepare-if-combining -canonicalize | FileCheck %s --check-prefix=CANON` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file -tritonamdgpu-prepare-if-combining | FileCheck %s`；` triton-opt %s -split-input-file -tritonamdgpu-prepare-if-combining -canonicalize | FileCheck %s --check-prefix=CANON`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 5-11
```mlir
 5| // CHECK-LABEL: op_between_ifs
 6| //       CHECK: %[[LOAD:.+]] = ttg.local_load
 7| //  CHECK-NEXT: tt.trans %[[LOAD]]
 8| //  CHECK-NEXT: scf.if
 9| // CANON-LABEL: op_between_ifs
10| //       CANON: scf.if
11| //   CANON-NOT: scf.if
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 12-15
```mlir
12| #blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
13| #blocked_transposed = #ttg.blocked<{sizePerThread = [4, 1], threadsPerWarp = [8, 8], warpsPerCTA = [1, 4], order = [0, 1]}>
14| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
15| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 16-16
```mlir
16| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 17-34
```mlir
17|   tt.func public @op_between_ifs(%cond: i1, %smem: !ttg.memdesc<32x32xf32, #shared, #smem>, %a: tensor<32x32xf32, #blocked>) -> tensor<32x32xf32, #blocked> {
18|     %0 = ttg.local_load %smem : !ttg.memdesc<32x32xf32, #shared, #smem> -> tensor<32x32xf32, #blocked>
19|     %1 = scf.if %cond -> tensor<32x32xf32, #blocked> {
20|       %mul = arith.mulf %a, %a : tensor<32x32xf32, #blocked>
21|       scf.yield %mul : tensor<32x32xf32, #blocked>
22|     } else {
23|       %div = arith.divf %a, %a : tensor<32x32xf32, #blocked>
24|       scf.yield %div : tensor<32x32xf32, #blocked>
25|     }
26|     %2 = tt.trans %0 {order = array<i32: 1, 0>} : tensor<32x32xf32, #blocked> -> tensor<32x32xf32, #blocked_transposed>
27|     %3 = scf.if %cond -> tensor<32x32xf32, #blocked_transposed> {
28|       %add = arith.addf %2, %2 : tensor<32x32xf32, #blocked_transposed>
29|       scf.yield %add : tensor<32x32xf32, #blocked_transposed>
30|     } else {
31|       %sub = arith.subf %2, %2 : tensor<32x32xf32, #blocked_transposed>
32|       scf.yield %sub : tensor<32x32xf32, #blocked_transposed>
33|     }
34|     %4 = ttg.convert_layout %3 : tensor<32x32xf32, #blocked_transposed> -> tensor<32x32xf32, #blocked>
```
**EN:** This function-oriented block defines or enters `op_between_ifs`. Within it, the test exercises loop/if yielded values, tt.func, structured conditionals, local/shared memory loads, arith.mulf, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `op_between_ifs` 为核心。测试在其中演示 循环/分支产出值、tt.func、结构化条件分支、本地/共享内存加载、arith.mulf，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 35-39
```mlir
35|     %5 = arith.addf %4, %1 : tensor<32x32xf32, #blocked>
36|     tt.return %5 : tensor<32x32xf32, #blocked>
37|   }
38| }
39| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on floating-point additions, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 浮点加法、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 40-40
```mlir
40| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 41-49
```mlir
41| 
42| // CHECK-LABEL: multiple_ops_between_ifs
43| //       CHECK: %[[LOAD:.+]] = ttg.local_load
44| //  CHECK-NEXT: %[[CVT:.+]] = ttg.convert_layout %[[LOAD]]
45| //  CHECK-NEXT: tt.trans %[[CVT]]
46| //  CHECK-NEXT: scf.if
47| // CANON-LABEL: multiple_ops_between_ifs
48| //       CANON: scf.if
49| //   CANON-NOT: scf.if
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 50-53
```mlir
50| #blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
51| #blocked_transposed = #ttg.blocked<{sizePerThread = [4, 1], threadsPerWarp = [8, 8], warpsPerCTA = [1, 4], order = [0, 1]}>
52| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
53| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 54-54
```mlir
54| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 55-72
```mlir
55|   tt.func public @multiple_ops_between_ifs(%cond: i1, %smem: !ttg.memdesc<32x32xf32, #shared, #smem>, %a: tensor<32x32xf32, #blocked>) -> tensor<32x32xf32, #blocked> {
56|     %0 = ttg.local_load %smem : !ttg.memdesc<32x32xf32, #shared, #smem> -> tensor<32x32xf32, #blocked>
57|     %1 = scf.if %cond -> tensor<32x32xf32, #blocked> {
58|       %mul = arith.mulf %a, %a : tensor<32x32xf32, #blocked>
59|       scf.yield %mul : tensor<32x32xf32, #blocked>
60|     } else {
61|       %div = arith.divf %a, %a : tensor<32x32xf32, #blocked>
62|       scf.yield %div : tensor<32x32xf32, #blocked>
63|     }
64|     %2 = ttg.convert_layout %0 : tensor<32x32xf32, #blocked> -> tensor<32x32xf32, #blocked_transposed>
65|     %3 = tt.trans %2 {order = array<i32: 1, 0>} : tensor<32x32xf32, #blocked_transposed> -> tensor<32x32xf32, #blocked>
66|     %4 = scf.if %cond -> tensor<32x32xf32, #blocked> {
67|       %add = arith.addf %3, %3 : tensor<32x32xf32, #blocked>
68|       scf.yield %add : tensor<32x32xf32, #blocked>
69|     } else {
70|       %sub = arith.subf %3, %3 : tensor<32x32xf32, #blocked>
71|       scf.yield %sub : tensor<32x32xf32, #blocked>
72|     }
```
**EN:** This function-oriented block defines or enters `multiple_ops_between_ifs`. Within it, the test exercises loop/if yielded values, tt.func, structured conditionals, local/shared memory loads, arith.mulf, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `multiple_ops_between_ifs` 为核心。测试在其中演示 循环/分支产出值、tt.func、结构化条件分支、本地/共享内存加载、arith.mulf，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 73-77
```mlir
73|     %5 = arith.addf %4, %1 : tensor<32x32xf32, #blocked>
74|     tt.return %5 : tensor<32x32xf32, #blocked>
75|   }
76| }
77| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on floating-point additions, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 浮点加法、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 78-78
```mlir
78| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 79-88
```mlir
79| 
80| // CHECK-LABEL: op_between_ifs_inside_for
81| //       CHECK: %[[LOAD:.+]] = ttg.local_load
82| //       CHECK: scf.for
83| //  CHECK-NEXT: tt.trans %[[LOAD]]
84| //  CHECK-NEXT: scf.if
85| // CANON-LABEL: op_between_ifs_inside_for
86| //       CANON: scf.for
87| //       CANON: scf.if
88| //   CANON-NOT: scf.if
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 89-92
```mlir
89| #blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
90| #blocked_transposed = #ttg.blocked<{sizePerThread = [4, 1], threadsPerWarp = [8, 8], warpsPerCTA = [1, 4], order = [0, 1]}>
91| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
92| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 93-93
```mlir
93| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 94-111
```mlir
 94|   tt.func public @op_between_ifs_inside_for(%cond: i1, %smem: !ttg.memdesc<32x32xf32, #shared, #smem>, %a: tensor<32x32xf32, #blocked>) -> tensor<32x32xf32, #blocked> {
 95|     %c0 = arith.constant 0 : index
 96|     %c1 = arith.constant 1 : index
 97|     %c4 = arith.constant 4 : index
 98|     %x = ttg.local_load %smem : !ttg.memdesc<32x32xf32, #shared, #smem> -> tensor<32x32xf32, #blocked>
 99|     %result = scf.for %i = %c0 to %c4 step %c1 iter_args(%acc = %a) -> tensor<32x32xf32, #blocked> {
100|       %0 = scf.if %cond -> tensor<32x32xf32, #blocked> {
101|         %mul = arith.mulf %acc, %acc : tensor<32x32xf32, #blocked>
102|         scf.yield %mul : tensor<32x32xf32, #blocked>
103|       } else {
104|         %div = arith.divf %acc, %acc : tensor<32x32xf32, #blocked>
105|         scf.yield %div : tensor<32x32xf32, #blocked>
106|       }
107|       %1 = tt.trans %x {order = array<i32: 1, 0>} : tensor<32x32xf32, #blocked> -> tensor<32x32xf32, #blocked_transposed>
108|       %2 = scf.if %cond -> tensor<32x32xf32, #blocked_transposed> {
109|         %add = arith.addf %1, %1 : tensor<32x32xf32, #blocked_transposed>
110|         scf.yield %add : tensor<32x32xf32, #blocked_transposed>
111|       } else {
```
**EN:** This function-oriented block defines or enters `op_between_ifs_inside_for`. Within it, the test exercises constants, loop/if yielded values, tt.func, structured conditionals, local/shared memory loads, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `op_between_ifs_inside_for` 为核心。测试在其中演示 常量、循环/分支产出值、tt.func、结构化条件分支、本地/共享内存加载，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 112-122
```mlir
112|         %sub = arith.subf %1, %1 : tensor<32x32xf32, #blocked_transposed>
113|         scf.yield %sub : tensor<32x32xf32, #blocked_transposed>
114|       }
115|       %3 = ttg.convert_layout %2 : tensor<32x32xf32, #blocked_transposed> -> tensor<32x32xf32, #blocked>
116|       %4 = arith.addf %3, %0 : tensor<32x32xf32, #blocked>
117|       scf.yield %4 : tensor<32x32xf32, #blocked>
118|     }
119|     tt.return %result : tensor<32x32xf32, #blocked>
120|   }
121| }
122| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, arith.subf, layout conversions, floating-point additions, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、arith.subf、布局转换、浮点加法、tt.return。

### Lines 123-123
```mlir
123| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 124-132
```mlir
124| 
125| // CHECK-LABEL: sibling_ifs_with_nested_if
126| //       CHECK: %[[LOAD:.+]] = ttg.local_load
127| //  CHECK-NEXT: tt.trans %[[LOAD]]
128| //  CHECK-NEXT: scf.if
129| // CANON-LABEL: sibling_ifs_with_nested_if
130| //       CANON: scf.if %arg0
131| //       CANON:   scf.if %arg1
132| //   CANON-NOT: scf.if
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 133-136
```mlir
133| #blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
134| #blocked_transposed = #ttg.blocked<{sizePerThread = [4, 1], threadsPerWarp = [8, 8], warpsPerCTA = [1, 4], order = [0, 1]}>
135| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
136| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 137-137
```mlir
137| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 138-155
```mlir
138|   tt.func public @sibling_ifs_with_nested_if(%cond: i1, %cond2: i1, %smem: !ttg.memdesc<32x32xf32, #shared, #smem>, %a: tensor<32x32xf32, #blocked>) -> tensor<32x32xf32, #blocked> {
139|     %x = ttg.local_load %smem : !ttg.memdesc<32x32xf32, #shared, #smem> -> tensor<32x32xf32, #blocked>
140|     %0 = scf.if %cond -> tensor<32x32xf32, #blocked> {
141|       %inner = scf.if %cond2 -> tensor<32x32xf32, #blocked> {
142|         %mul = arith.mulf %a, %a : tensor<32x32xf32, #blocked>
143|         scf.yield %mul : tensor<32x32xf32, #blocked>
144|       } else {
145|         scf.yield %a : tensor<32x32xf32, #blocked>
146|       }
147|       scf.yield %inner : tensor<32x32xf32, #blocked>
148|     } else {
149|       %div = arith.divf %a, %a : tensor<32x32xf32, #blocked>
150|       scf.yield %div : tensor<32x32xf32, #blocked>
151|     }
152|     %1 = tt.trans %x {order = array<i32: 1, 0>} : tensor<32x32xf32, #blocked> -> tensor<32x32xf32, #blocked_transposed>
153|     %2 = scf.if %cond -> tensor<32x32xf32, #blocked_transposed> {
154|       %add = arith.addf %1, %1 : tensor<32x32xf32, #blocked_transposed>
155|       scf.yield %add : tensor<32x32xf32, #blocked_transposed>
```
**EN:** This function-oriented block defines or enters `sibling_ifs_with_nested_if`. Within it, the test exercises loop/if yielded values, structured conditionals, tt.func, local/shared memory loads, arith.mulf, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `sibling_ifs_with_nested_if` 为核心。测试在其中演示 循环/分支产出值、结构化条件分支、tt.func、本地/共享内存加载、arith.mulf，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 156-165
```mlir
156|     } else {
157|       %sub = arith.subf %1, %1 : tensor<32x32xf32, #blocked_transposed>
158|       scf.yield %sub : tensor<32x32xf32, #blocked_transposed>
159|     }
160|     %3 = ttg.convert_layout %2 : tensor<32x32xf32, #blocked_transposed> -> tensor<32x32xf32, #blocked>
161|     %4 = arith.addf %3, %0 : tensor<32x32xf32, #blocked>
162|     tt.return %4 : tensor<32x32xf32, #blocked>
163|   }
164| }
165| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining arith.subf, loop/if yielded values, layout conversions, floating-point additions, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 arith.subf、循环/分支产出值、布局转换、浮点加法、tt.return。

### Lines 166-166
```mlir
166| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 167-168
```mlir
167| 
168| // Negative test: ifs in different blocks
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 169-174
```mlir
169| // CHECK-LABEL: ifs_in_different_blocks
170| //       CHECK: scf.if
171| //       CHECK: scf.for
172| //  CHECK-NEXT: tt.trans
173| //  CHECK-NEXT: scf.if
174| // CANON-LABEL: ifs_in_different_blocks
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 175-178
```mlir
175| #blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
176| #blocked_transposed = #ttg.blocked<{sizePerThread = [4, 1], threadsPerWarp = [8, 8], warpsPerCTA = [1, 4], order = [0, 1]}>
177| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
178| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 179-179
```mlir
179| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 180-197
```mlir
180|   tt.func public @ifs_in_different_blocks(%cond: i1, %smem: !ttg.memdesc<32x32xf32, #shared, #smem>, %a: tensor<32x32xf32, #blocked>) -> tensor<32x32xf32, #blocked> {
181|     %c0 = arith.constant 0 : index
182|     %c1 = arith.constant 1 : index
183|     %c4 = arith.constant 4 : index
184|     %x = ttg.local_load %smem : !ttg.memdesc<32x32xf32, #shared, #smem> -> tensor<32x32xf32, #blocked>
185|     %0 = scf.if %cond -> tensor<32x32xf32, #blocked> {
186|       %mul = arith.mulf %a, %a : tensor<32x32xf32, #blocked>
187|       scf.yield %mul : tensor<32x32xf32, #blocked>
188|     } else {
189|       %div = arith.divf %a, %a : tensor<32x32xf32, #blocked>
190|       scf.yield %div : tensor<32x32xf32, #blocked>
191|     }
192|     %result = scf.for %i = %c0 to %c4 step %c1 iter_args(%acc = %0) -> tensor<32x32xf32, #blocked> {
193|       %1 = tt.trans %x {order = array<i32: 1, 0>} : tensor<32x32xf32, #blocked> -> tensor<32x32xf32, #blocked_transposed>
194|       %2 = scf.if %cond -> tensor<32x32xf32, #blocked_transposed> {
195|         %add = arith.addf %1, %1 : tensor<32x32xf32, #blocked_transposed>
196|         scf.yield %add : tensor<32x32xf32, #blocked_transposed>
197|       } else {
```
**EN:** This function-oriented block defines or enters `ifs_in_different_blocks`. Within it, the test exercises constants, loop/if yielded values, tt.func, structured conditionals, local/shared memory loads, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `ifs_in_different_blocks` 为核心。测试在其中演示 常量、循环/分支产出值、tt.func、结构化条件分支、本地/共享内存加载，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 198-208
```mlir
198|         %sub = arith.subf %1, %1 : tensor<32x32xf32, #blocked_transposed>
199|         scf.yield %sub : tensor<32x32xf32, #blocked_transposed>
200|       }
201|       %3 = ttg.convert_layout %2 : tensor<32x32xf32, #blocked_transposed> -> tensor<32x32xf32, #blocked>
202|       %4 = arith.addf %3, %acc : tensor<32x32xf32, #blocked>
203|       scf.yield %4 : tensor<32x32xf32, #blocked>
204|     }
205|     tt.return %result : tensor<32x32xf32, #blocked>
206|   }
207| }
208| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, arith.subf, layout conversions, floating-point additions, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、arith.subf、布局转换、浮点加法、tt.return。

### Lines 209-209
```mlir
209| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 210-211
```mlir
210| 
211| // Negative test: nested ifs
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 212-216
```mlir
212| // CHECK-LABEL: nested_ifs
213| //       CHECK: scf.if
214| //  CHECK-NEXT: tt.trans
215| //  CHECK-NEXT: scf.if
216| // CANON-LABEL: nested_ifs
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 217-220
```mlir
217| #blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
218| #blocked_transposed = #ttg.blocked<{sizePerThread = [4, 1], threadsPerWarp = [8, 8], warpsPerCTA = [1, 4], order = [0, 1]}>
219| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
220| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 221-221
```mlir
221| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 222-239
```mlir
222|   tt.func public @nested_ifs(%cond: i1, %cond2: i1, %smem: !ttg.memdesc<32x32xf32, #shared, #smem>, %a: tensor<32x32xf32, #blocked>) -> tensor<32x32xf32, #blocked> {
223|     %x = ttg.local_load %smem : !ttg.memdesc<32x32xf32, #shared, #smem> -> tensor<32x32xf32, #blocked>
224|     %0 = scf.if %cond -> tensor<32x32xf32, #blocked> {
225|       %1 = tt.trans %x {order = array<i32: 1, 0>} : tensor<32x32xf32, #blocked> -> tensor<32x32xf32, #blocked_transposed>
226|       %2 = scf.if %cond2 -> tensor<32x32xf32, #blocked_transposed> {
227|         %add = arith.addf %1, %1 : tensor<32x32xf32, #blocked_transposed>
228|         scf.yield %add : tensor<32x32xf32, #blocked_transposed>
229|       } else {
230|         %sub = arith.subf %1, %1 : tensor<32x32xf32, #blocked_transposed>
231|         scf.yield %sub : tensor<32x32xf32, #blocked_transposed>
232|       }
233|       %3 = ttg.convert_layout %2 : tensor<32x32xf32, #blocked_transposed> -> tensor<32x32xf32, #blocked>
234|       scf.yield %3 : tensor<32x32xf32, #blocked>
235|     } else {
236|       %div = arith.divf %a, %a : tensor<32x32xf32, #blocked>
237|       scf.yield %div : tensor<32x32xf32, #blocked>
238|     }
239|     tt.return %0 : tensor<32x32xf32, #blocked>
```
**EN:** This function-oriented block defines or enters `nested_ifs`. Within it, the test exercises loop/if yielded values, tt.func, structured conditionals, local/shared memory loads, transpose-like layout changes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `nested_ifs` 为核心。测试在其中演示 循环/分支产出值、tt.func、结构化条件分支、本地/共享内存加载、转置类布局变换，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 240-242
```mlir
240|   }
241| }
242| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 243-243
```mlir
243| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 244-245
```mlir
244| 
245| // Negative test: ifs with different conditions
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 246-251
```mlir
246| // CHECK-LABEL: different_conditions
247| //       CHECK: scf.if
248| //  CHECK-NEXT: arith.mulf
249| //       CHECK: tt.trans
250| //  CHECK-NEXT: scf.if
251| // CANON-LABEL: different_conditions
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 252-255
```mlir
252| #blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
253| #blocked_transposed = #ttg.blocked<{sizePerThread = [4, 1], threadsPerWarp = [8, 8], warpsPerCTA = [1, 4], order = [0, 1]}>
254| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
255| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 256-256
```mlir
256| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 257-274
```mlir
257|   tt.func public @different_conditions(%cond1: i1, %cond2: i1, %smem: !ttg.memdesc<32x32xf32, #shared, #smem>, %a: tensor<32x32xf32, #blocked>) -> tensor<32x32xf32, #blocked> {
258|     %x = ttg.local_load %smem : !ttg.memdesc<32x32xf32, #shared, #smem> -> tensor<32x32xf32, #blocked>
259|     %0 = scf.if %cond1 -> tensor<32x32xf32, #blocked> {
260|       %mul = arith.mulf %a, %a : tensor<32x32xf32, #blocked>
261|       scf.yield %mul : tensor<32x32xf32, #blocked>
262|     } else {
263|       %div = arith.divf %a, %a : tensor<32x32xf32, #blocked>
264|       scf.yield %div : tensor<32x32xf32, #blocked>
265|     }
266|     %1 = tt.trans %x {order = array<i32: 1, 0>} : tensor<32x32xf32, #blocked> -> tensor<32x32xf32, #blocked_transposed>
267|     %2 = scf.if %cond2 -> tensor<32x32xf32, #blocked_transposed> {
268|       %add = arith.addf %1, %1 : tensor<32x32xf32, #blocked_transposed>
269|       scf.yield %add : tensor<32x32xf32, #blocked_transposed>
270|     } else {
271|       %sub = arith.subf %1, %1 : tensor<32x32xf32, #blocked_transposed>
272|       scf.yield %sub : tensor<32x32xf32, #blocked_transposed>
273|     }
274|     %3 = ttg.convert_layout %2 : tensor<32x32xf32, #blocked_transposed> -> tensor<32x32xf32, #blocked>
```
**EN:** This function-oriented block defines or enters `different_conditions`. Within it, the test exercises loop/if yielded values, tt.func, structured conditionals, local/shared memory loads, arith.mulf, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `different_conditions` 为核心。测试在其中演示 循环/分支产出值、tt.func、结构化条件分支、本地/共享内存加载、arith.mulf，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 275-279
```mlir
275|     %4 = arith.addf %3, %0 : tensor<32x32xf32, #blocked>
276|     tt.return %4 : tensor<32x32xf32, #blocked>
277|   }
278| }
279| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on floating-point additions, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 浮点加法、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 280-280
```mlir
280| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 281-282
```mlir
281| 
282| // Negative test: non-pure op (local_store) between ifs
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 283-288
```mlir
283| // CHECK-LABEL: non_pure_op_between_ifs
284| //       CHECK: scf.if
285| //       CHECK: tt.trans
286| //  CHECK-NEXT: ttg.local_store
287| //  CHECK-NEXT: scf.if
288| // CANON-LABEL: non_pure_op_between_ifs
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 289-293
```mlir
289| #blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
290| #blocked_transposed = #ttg.blocked<{sizePerThread = [4, 1], threadsPerWarp = [8, 8], warpsPerCTA = [1, 4], order = [0, 1]}>
291| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
292| #shared_transposed = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0, 1]}>
293| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 294-294
```mlir
294| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 295-312
```mlir
295|   tt.func public @non_pure_op_between_ifs(%cond: i1, %smem: !ttg.memdesc<32x32xf32, #shared, #smem>, %smem_trans: !ttg.memdesc<32x32xf32, #shared_transposed, #smem, mutable>, %a: tensor<32x32xf32, #blocked>) -> tensor<32x32xf32, #blocked> {
296|     %x = ttg.local_load %smem : !ttg.memdesc<32x32xf32, #shared, #smem> -> tensor<32x32xf32, #blocked>
297|     %0 = scf.if %cond -> tensor<32x32xf32, #blocked> {
298|       %mul = arith.mulf %a, %a : tensor<32x32xf32, #blocked>
299|       scf.yield %mul : tensor<32x32xf32, #blocked>
300|     } else {
301|       %div = arith.divf %a, %a : tensor<32x32xf32, #blocked>
302|       scf.yield %div : tensor<32x32xf32, #blocked>
303|     }
304|     %1 = tt.trans %x {order = array<i32: 1, 0>} : tensor<32x32xf32, #blocked> -> tensor<32x32xf32, #blocked_transposed>
305|     ttg.local_store %1, %smem_trans : tensor<32x32xf32, #blocked_transposed> -> !ttg.memdesc<32x32xf32, #shared_transposed, #smem, mutable>
306|     %2 = scf.if %cond -> tensor<32x32xf32, #blocked_transposed> {
307|       %add = arith.addf %1, %1 : tensor<32x32xf32, #blocked_transposed>
308|       scf.yield %add : tensor<32x32xf32, #blocked_transposed>
309|     } else {
310|       %sub = arith.subf %1, %1 : tensor<32x32xf32, #blocked_transposed>
311|       scf.yield %sub : tensor<32x32xf32, #blocked_transposed>
312|     }
```
**EN:** This function-oriented block defines or enters `non_pure_op_between_ifs`. Within it, the test exercises loop/if yielded values, tt.func, structured conditionals, local/shared memory loads, arith.mulf, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `non_pure_op_between_ifs` 为核心。测试在其中演示 循环/分支产出值、tt.func、结构化条件分支、本地/共享内存加载、arith.mulf，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 313-317
```mlir
313|     %3 = ttg.convert_layout %2 : tensor<32x32xf32, #blocked_transposed> -> tensor<32x32xf32, #blocked>
314|     %4 = arith.addf %3, %0 : tensor<32x32xf32, #blocked>
315|     tt.return %4 : tensor<32x32xf32, #blocked>
316|   }
317| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, floating-point additions, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、浮点加法、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-tritonamdgpu-prepare-if-combining`, `-canonicalize`
- **CN:** 主要 pass 选项：`-split-input-file`，`-tritonamdgpu-prepare-if-combining`，`-canonicalize`
- **EN:** Dominant operations include `scf.yield`, `scf.if`, `tt.func`, `arith.addf`, `module`, `ttg.local_load`, `arith.divf`, `tt.trans`, `arith.subf`, `ttg.convert_layout`.
- **CN:** 主要操作包括 `scf.yield`、`scf.if`、`tt.func`、`arith.addf`、`module`、`ttg.local_load`、`arith.divf`、`tt.trans`、`arith.subf`、`ttg.convert_layout`。
- **EN:** The file contains 7 independently testable section(s). Check styles used: CHECK-NEXT x17, CHECK x12, CHECK-LABEL x8. Important labels include op_between_ifs, multiple_ops_between_ifs, op_between_ifs_inside_for, sibling_ifs_with_nested_if. It validates that the selected pass pipeline rewrites each labeled test case into the checked form.
- **CN:** 该文件包含 7 个可独立测试的分段。使用的检查类型：CHECK-NEXT ×17，CHECK ×12，CHECK-LABEL ×8。 关键标签包括 op_between_ifs，multiple_ops_between_ifs，op_between_ifs_inside_for，sibling_ifs_with_nested_if。 它验证所选 pass 流水线会把每个带标签的测试用例重写成检查所要求的形式。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `ttg`: TritonGPU layout, memory, and GPU-specific operations.
- **CN:** `ttg`：TritonGPU 布局、内存和 GPU 专用操作。
- **EN:** `scf`: Structured control flow for loops and conditionals.
- **CN:** `scf`：用于循环与条件分支的结构化控制流。
- **EN:** `arith`: MLIR arithmetic dialect for scalar/tensor math.
- **CN:** `arith`：用于标量/张量运算的 MLIR arithmetic 方言。
- **EN:** `tt`: Triton high-level tensor/pointer operations.
- **CN:** `tt`：Triton 高层张量/指针操作。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。