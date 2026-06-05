# amd-update-async-wait-count-without-token.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/amd/amd-update-async-wait-count-without-token.mlir`
- **EN:** Test for the MLIR/Triton pipeline `--tritonamdgpu-update-async-wait-count=gfx-arch=gfx1250` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `--tritonamdgpu-update-async-wait-count=gfx-arch=gfx1250` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file --tritonamdgpu-update-async-wait-count=gfx-arch=gfx1250 | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file --tritonamdgpu-update-async-wait-count=gfx-arch=gfx1250 | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-5
```mlir
1| // RUN: triton-opt %s -split-input-file --tritonamdgpu-update-async-wait-count=gfx-arch=gfx1250 | FileCheck %s
2| 
3| // The number in SSA symbolic names represents the number of generated async load operation at assembly level a ttg.async_copy_global_to_local will generate, which is counted by this pass.
4| // For example `ttg.async_copy_global_to_local %ptr2Inst, %memDesc2Inst ..` will generate two global_load_async_to_lds_b128 assembly instruction
5| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file --tritonamdgpu-update-async-wait-count=gfx-arch=gfx1250 | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file --tritonamdgpu-update-async-wait-count=gfx-arch=gfx1250 | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 6-8
```mlir
6| #blocked = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [2, 32], warpsPerCTA = [4, 1], order = [0, 1]}>
7| #shared = #ttg.swizzled_shared<{vec = 8, perPhase = 1, maxPhase = 16, order = [0, 1]}>
8| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 9-10
```mlir
 9| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
10| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 11-11
```mlir
11|   // CHECK-LABEL: simple_waitcnt
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: simple_waitcnt anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: simple_waitcnt 这样的标签用于锚定匹配范围。

### Lines 12-25
```mlir
12|   tt.func public @simple_waitcnt(
13|         %cond: i1,
14|         %arg0: i32,
15|         %memDesc2Inst: !ttg.memdesc<128x16xf16, #shared, #smem, mutable>,
16|         %ptr2Inst: tensor<128x16x!tt.ptr<f16>, #blocked>  {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>},
17|         %memDesc1Inst: !ttg.memdesc<64x16xf16, #shared, #smem, mutable>,
18|         %ptr1Inst: tensor<64x16x!tt.ptr<f16>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>}) {
19|     // Emit 1 instruction
20|     ttg.async_copy_global_to_local %ptr1Inst, %memDesc1Inst : tensor<64x16x!tt.ptr<f16>, #blocked> -> <64x16xf16, #shared, #smem, mutable>
21|     ttg.async_commit_group
22|     // Emits 2 instructions
23|     ttg.async_copy_global_to_local %ptr2Inst, %memDesc2Inst : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
24|     ttg.async_commit_group
25| 
```
**EN:** This function-oriented block defines or enters `simple_waitcnt`. Within it, the test exercises tt.func, ttg.async_copy_global_to_local, async copy commit groups, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `simple_waitcnt` 为核心。测试在其中演示 tt.func、ttg.async_copy_global_to_local、异步拷贝提交组，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 26-28
```mlir
26|     // CHECK: amdg.async_wait {num_inst = 0
27|     ttg.async_wait {num = 0 : i32}
28|     // 1 outstanding commit group (2nd): 2 instructions
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 29-31
```mlir
29|     // CHECK: amdg.async_wait {num_inst = 2
30|     ttg.async_wait {num = 1 : i32}
31|     // 2 outstanding commit groups: 2 + 1 = 3 instructions
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 32-34
```mlir
32|     // CHECK: amdg.async_wait {num_inst = 3
33|     ttg.async_wait {num = 2 : i32}
34|     // Only 2 commit groups exist, stop at function boundary
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 35-40
```mlir
35|     // CHECK: amdg.async_wait {num_inst = 3
36|     ttg.async_wait {num = 3 : i32}
37| 
38|     tt.return
39|   }
40| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on async wait synchronization, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 异步等待同步、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 41-41
```mlir
41|   // CHECK-LABEL: simple_waitcnt_non_committed_async_ops
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: simple_waitcnt_non_committed_async_ops anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: simple_waitcnt_non_committed_async_ops 这样的标签用于锚定匹配范围。

### Lines 42-52
```mlir
42|   tt.func public @simple_waitcnt_non_committed_async_ops(
43|         %cond: i1,
44|         %arg0: i32,
45|         %memDesc2Inst: !ttg.memdesc<128x16xf16, #shared, #smem, mutable>,
46|         %ptr2Inst: tensor<128x16x!tt.ptr<f16>, #blocked>  {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>},
47|         %memDesc1Inst: !ttg.memdesc<64x16xf16, #shared, #smem, mutable>,
48|         %ptr1Inst: tensor<64x16x!tt.ptr<f16>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>}) {
49|     // Emit 1 instruction
50|     ttg.async_copy_global_to_local %ptr1Inst, %memDesc1Inst : tensor<64x16x!tt.ptr<f16>, #blocked> -> <64x16xf16, #shared, #smem, mutable>
51| 
52|     // No commit groups found, walk to function boundary: 1 instruction
```
**EN:** This function-oriented block defines or enters `simple_waitcnt_non_committed_async_ops`. Within it, the test exercises tt.func, ttg.async_copy_global_to_local, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `simple_waitcnt_non_committed_async_ops` 为核心。测试在其中演示 tt.func、ttg.async_copy_global_to_local，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 53-55
```mlir
53|     // CHECK: amdg.async_wait {num_inst = 1
54|     ttg.async_wait {num = 0 : i32}
55|     // -1 means wait on all — immediate stop → conservative 0
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 56-61
```mlir
56|     // CHECK: amdg.async_wait {num_inst = 0
57|     ttg.async_wait {num = -1 : i32}
58| 
59|     tt.return
60|   }
61| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on async wait synchronization, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 异步等待同步、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 62-62
```mlir
62|   // CHECK-LABEL: wait_if_without_else
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: wait_if_without_else anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: wait_if_without_else 这样的标签用于锚定匹配范围。

### Lines 63-77
```mlir
63|   tt.func public @wait_if_without_else(
64|         %cond: i1,
65|         %arg0: i32,
66|         %memDesc2Inst: !ttg.memdesc<128x16xf16, #shared, #smem, mutable>,
67|         %ptr2Inst: tensor<128x16x!tt.ptr<f16>, #blocked>  {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>},
68|         %memDesc1Inst: !ttg.memdesc<64x16xf16, #shared, #smem, mutable>,
69|         %ptr1Inst: tensor<64x16x!tt.ptr<f16>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>}) {
70|     // Ensure we look into then but also skip the if if no else is present
71| 
72|     ttg.async_copy_global_to_local %ptr1Inst, %memDesc1Inst : tensor<64x16x!tt.ptr<f16>, #blocked> -> <64x16xf16, #shared, #smem, mutable>
73|     ttg.async_commit_group
74|     scf.if %cond {
75|       ttg.async_copy_global_to_local %ptr2Inst, %memDesc2Inst : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
76|       ttg.async_commit_group
77|     }
```
**EN:** This function-oriented block defines or enters `wait_if_without_else`. Within it, the test exercises tt.func, ttg.async_copy_global_to_local, async copy commit groups, structured conditionals, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `wait_if_without_else` 为核心。测试在其中演示 tt.func、ttg.async_copy_global_to_local、异步拷贝提交组、结构化条件分支，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 78-87
```mlir
78|     // CHECK: amdg.async_wait {num_inst = 1
79|     ttg.async_wait {num = 1: i32}
80| 
81|     ttg.async_copy_global_to_local %ptr2Inst, %memDesc2Inst : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
82|     ttg.async_commit_group
83|     scf.if %cond {
84|       ttg.async_copy_global_to_local %ptr1Inst, %memDesc1Inst : tensor<64x16x!tt.ptr<f16>, #blocked> -> <64x16xf16, #shared, #smem, mutable>
85|       ttg.async_commit_group
86|       scf.yield
87|     }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining ttg.async_copy_global_to_local, async copy commit groups, async wait synchronization, structured conditionals, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 ttg.async_copy_global_to_local、异步拷贝提交组、异步等待同步、结构化条件分支、循环/分支产出值。

### Lines 88-90
```mlir
88|     // CHECK: amdg.async_wait {num_inst = 1
89|     ttg.async_wait {num = 1: i32}
90| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 91-97
```mlir
91|     // CHECK: amdg.async_wait {num_inst = 3
92|     ttg.async_wait {num = 2: i32}
93| 
94| 
95|     tt.return
96|   }
97| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on async wait synchronization, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 异步等待同步、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 98-98
```mlir
98|   // CHECK-LABEL wait_if_with_else
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL wait_if_with_else anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL wait_if_with_else 这样的标签用于锚定匹配范围。

### Lines 99-114
```mlir
 99|   tt.func public @wait_if_with_else(
100|         %cond: i1,
101|         %arg0: i32,
102|         %memDesc2Inst: !ttg.memdesc<128x16xf16, #shared, #smem, mutable>,
103|         %ptr2Inst: tensor<128x16x!tt.ptr<f16>, #blocked>  {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>},
104|         %memDesc1Inst: !ttg.memdesc<64x16xf16, #shared, #smem, mutable>,
105|         %ptr1Inst: tensor<64x16x!tt.ptr<f16>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>}) {
106|     scf.if %cond {
107|       ttg.async_copy_global_to_local %ptr1Inst, %memDesc1Inst : tensor<64x16x!tt.ptr<f16>, #blocked> -> <64x16xf16, #shared, #smem, mutable>
108|       scf.yield
109|     } else {
110|       ttg.async_copy_global_to_local %ptr2Inst, %memDesc2Inst : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
111|       scf.yield
112|     }
113|     ttg.async_commit_group
114|     // Ensure we use the branch with less instructions (then)
```
**EN:** This function-oriented block defines or enters `wait_if_with_else`. Within it, the test exercises tt.func, ttg.async_copy_global_to_local, loop/if yielded values, structured conditionals, async copy commit groups, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `wait_if_with_else` 为核心。测试在其中演示 tt.func、ttg.async_copy_global_to_local、循环/分支产出值、结构化条件分支、异步拷贝提交组，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 115-117
```mlir
115|     // CHECK: amdg.async_wait {num_inst = 1
116|     ttg.async_wait {num = 1: i32}
117|     // Check we do not loop in an if but instead continue upwards
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 118-129
```mlir
118|     // CHECK: amdg.async_wait {num_inst = 1
119|     ttg.async_wait {num = 2: i32}
120| 
121|     scf.if %cond {
122|       ttg.async_copy_global_to_local %ptr2Inst, %memDesc2Inst : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
123|       scf.yield
124|     } else {
125|       ttg.async_copy_global_to_local %ptr1Inst, %memDesc1Inst : tensor<64x16x!tt.ptr<f16>, #blocked> -> <64x16xf16, #shared, #smem, mutable>
126|       scf.yield
127|     }
128|     ttg.async_commit_group
129|     // Ensure we use the branch with less instructions (else)
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining ttg.async_copy_global_to_local, loop/if yielded values, async wait synchronization, structured conditionals, async copy commit groups.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 ttg.async_copy_global_to_local、循环/分支产出值、异步等待同步、结构化条件分支、异步拷贝提交组。

### Lines 130-135
```mlir
130|     // CHECK: amdg.async_wait {num_inst = 1
131|     ttg.async_wait {num = 1: i32}
132| 
133|     tt.return
134|   }
135| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on async wait synchronization, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 异步等待同步、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 136-136
```mlir
136|   // CHECK-LABEL: check_wait_nested_ifs
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: check_wait_nested_ifs anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: check_wait_nested_ifs 这样的标签用于锚定匹配范围。

### Lines 137-154
```mlir
137|   tt.func public @check_wait_nested_ifs(
138|         %cond: i1,
139|         %arg0: i32,
140|         %memDesc2Inst: !ttg.memdesc<128x16xf16, #shared, #smem, mutable>,
141|         %ptr2Inst: tensor<128x16x!tt.ptr<f16>, #blocked>  {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>},
142|         %memDesc1Inst: !ttg.memdesc<64x16xf16, #shared, #smem, mutable>,
143|         %ptr1Inst: tensor<64x16x!tt.ptr<f16>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>}) {
144|     scf.if %cond {
145|       ttg.async_copy_global_to_local %ptr1Inst, %memDesc1Inst : tensor<64x16x!tt.ptr<f16>, #blocked> -> <64x16xf16, #shared, #smem, mutable>
146|       ttg.async_copy_global_to_local %ptr1Inst, %memDesc1Inst : tensor<64x16x!tt.ptr<f16>, #blocked> -> <64x16xf16, #shared, #smem, mutable>
147|       scf.if %cond {
148|         ttg.async_copy_global_to_local %ptr1Inst, %memDesc1Inst : tensor<64x16x!tt.ptr<f16>, #blocked> -> <64x16xf16, #shared, #smem, mutable>
149|         ttg.async_copy_global_to_local %ptr1Inst, %memDesc1Inst : tensor<64x16x!tt.ptr<f16>, #blocked> -> <64x16xf16, #shared, #smem, mutable>
150|         scf.yield
151|       } else {
152|         ttg.async_copy_global_to_local %ptr1Inst, %memDesc1Inst : tensor<64x16x!tt.ptr<f16>, #blocked> -> <64x16xf16, #shared, #smem, mutable>
153|         scf.yield
154|       }
```
**EN:** This function-oriented block defines or enters `check_wait_nested_ifs`. Within it, the test exercises ttg.async_copy_global_to_local, tt.func, structured conditionals, loop/if yielded values, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `check_wait_nested_ifs` 为核心。测试在其中演示 ttg.async_copy_global_to_local、tt.func、结构化条件分支、循环/分支产出值，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 155-170
```mlir
155|       ttg.async_commit_group
156|       scf.yield
157|     } else {
158|       ttg.async_copy_global_to_local %ptr1Inst, %memDesc1Inst : tensor<64x16x!tt.ptr<f16>, #blocked> -> <64x16xf16, #shared, #smem, mutable>
159|       scf.if %cond {
160|         ttg.async_copy_global_to_local %ptr1Inst, %memDesc1Inst : tensor<64x16x!tt.ptr<f16>, #blocked> -> <64x16xf16, #shared, #smem, mutable>
161|         scf.yield
162|       } else {
163|         ttg.async_copy_global_to_local %ptr1Inst, %memDesc1Inst : tensor<64x16x!tt.ptr<f16>, #blocked> -> <64x16xf16, #shared, #smem, mutable>
164|         ttg.async_copy_global_to_local %ptr1Inst, %memDesc1Inst : tensor<64x16x!tt.ptr<f16>, #blocked> -> <64x16xf16, #shared, #smem, mutable>
165|         scf.yield
166|       }
167|       ttg.async_commit_group
168|       scf.yield
169|     }
170|     // The shortest path (else->then) contains 2 async ops -> instruction count 2
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, ttg.async_copy_global_to_local, async copy commit groups, structured conditionals.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、ttg.async_copy_global_to_local、异步拷贝提交组、结构化条件分支。

### Lines 171-177
```mlir
171|     // CHECK: amdg.async_wait {num_inst = 2
172|     ttg.async_wait {num = 1: i32}
173| 
174|     tt.return
175|   }
176| 
177|   //CHECK-LABEL: for_without_async_ops
```
**EN:** This operational block carries the concrete IR steps for the test, centered on async wait synchronization, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 异步等待同步、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 178-192
```mlir
178|   tt.func public @for_without_async_ops(
179|         %cond: i1,
180|         %arg0: i32,
181|         %memDesc2Inst: !ttg.memdesc<128x16xf16, #shared, #smem, mutable>,
182|         %ptr2Inst: tensor<128x16x!tt.ptr<f16>, #blocked>  {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>},
183|         %memDesc1Inst: !ttg.memdesc<64x16xf16, #shared, #smem, mutable>,
184|         %ptr1Inst: tensor<64x16x!tt.ptr<f16>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>}) {
185| 
186|     %c0_i32 = arith.constant 0 : i32
187|     %c1_i32 = arith.constant 1 : i32
188| 
189|     ttg.async_copy_global_to_local %ptr1Inst, %memDesc1Inst : tensor<64x16x!tt.ptr<f16>, #blocked> -> <64x16xf16, #shared, #smem, mutable>
190|     ttg.async_commit_group
191| 
192|     scf.for %arg14 = %c0_i32 to %arg0 step %c1_i32 iter_args() -> () : i32 {
```
**EN:** This function-oriented block defines or enters `for_without_async_ops`. Within it, the test exercises tt.func, constants, ttg.async_copy_global_to_local, async copy commit groups, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `for_without_async_ops` 为核心。测试在其中演示 tt.func、常量、ttg.async_copy_global_to_local、异步拷贝提交组、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 193-196
```mlir
193|       // CHECK: amdg.async_wait {num_inst = 1
194|       ttg.async_wait {num = 1: i32}
195|       scf.yield
196|     }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining async wait synchronization, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 异步等待同步、循环/分支产出值。

### Lines 197-203
```mlir
197|     // CHECK: amdg.async_wait {num_inst = 1
198|     ttg.async_wait {num = 1: i32}
199| 
200|     tt.return
201|   }
202| 
203|   //CHECK-LABEL: for_with_async_ops
```
**EN:** This operational block carries the concrete IR steps for the test, centered on async wait synchronization, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 异步等待同步、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 204-221
```mlir
204|   tt.func public @for_with_async_ops(
205|         %cond: i1,
206|         %arg0: i32,
207|         %memDesc2Inst: !ttg.memdesc<128x16xf16, #shared, #smem, mutable>,
208|         %ptr2Inst: tensor<128x16x!tt.ptr<f16>, #blocked>  {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>},
209|         %memDesc1Inst: !ttg.memdesc<64x16xf16, #shared, #smem, mutable>,
210|         %ptr1Inst: tensor<64x16x!tt.ptr<f16>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>}) {
211| 
212|     %c0_i32 = arith.constant 0 : i32
213|     %c1_i32 = arith.constant 1 : i32
214| 
215|     ttg.async_copy_global_to_local %ptr2Inst, %memDesc2Inst : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
216|     ttg.async_commit_group
217|     ttg.async_copy_global_to_local %ptr2Inst, %memDesc2Inst : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
218|     ttg.async_commit_group
219|     ttg.async_copy_global_to_local %ptr2Inst, %memDesc2Inst : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
220|     ttg.async_commit_group
221|     // 3 outstanding commit groups, each ptr2Inst emits 2 instr → 6
```
**EN:** This function-oriented block defines or enters `for_with_async_ops`. Within it, the test exercises ttg.async_copy_global_to_local, async copy commit groups, tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `for_with_async_ops` 为核心。测试在其中演示 ttg.async_copy_global_to_local、异步拷贝提交组、tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 222-226
```mlir
222|     // CHECK: amdg.async_wait {num_inst = 6
223|     ttg.async_wait {num = 3: i32}
224| 
225|     scf.for %arg14 = %c0_i32 to %arg0 step %c1_i32 : i32 {
226|       // The minimum it waits are 3 loop iteration with 1 instructions per iteration. Note the prologue would lead to 6
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining async wait synchronization, structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 异步等待同步、结构化循环。

### Lines 227-233
```mlir
227|       // CHECK: amdg.async_wait {num_inst = 3
228|       ttg.async_wait {num = 3: i32}
229|       ttg.async_copy_global_to_local %ptr1Inst, %memDesc1Inst : tensor<64x16x!tt.ptr<f16>, #blocked> -> <64x16xf16, #shared, #smem, mutable>
230|       ttg.async_commit_group
231|       scf.yield
232|     }
233|     // The minimum it waits are 3 loop iteration with 1 instructions per iteration. Note the prologue would lead to 6
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining async wait synchronization, ttg.async_copy_global_to_local, async copy commit groups, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 异步等待同步、ttg.async_copy_global_to_local、异步拷贝提交组、循环/分支产出值。

### Lines 234-240
```mlir
234|     // CHECK: amdg.async_wait {num_inst = 3
235|     ttg.async_wait {num = 3: i32}
236| 
237|     tt.return
238|   }
239| 
240|   //CHECK-LABEL: for_nested_control_flow
```
**EN:** This operational block carries the concrete IR steps for the test, centered on async wait synchronization, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 异步等待同步、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 241-258
```mlir
241|   tt.func public @for_nested_control_flow(
242|         %cond: i1,
243|         %arg0: i32,
244|         %memDesc2Inst: !ttg.memdesc<128x16xf16, #shared, #smem, mutable>,
245|         %ptr2Inst: tensor<128x16x!tt.ptr<f16>, #blocked>  {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>},
246|         %memDesc1Inst: !ttg.memdesc<64x16xf16, #shared, #smem, mutable>,
247|         %ptr1Inst: tensor<64x16x!tt.ptr<f16>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>}) {
248| 
249|     %c0_i32 = arith.constant 0 : i32
250|     %c1_i32 = arith.constant 1 : i32
251| 
252|     // Prologue: 2 instructions per commit group
253|     ttg.async_copy_global_to_local %ptr2Inst, %memDesc2Inst : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
254|     ttg.async_commit_group
255|     ttg.async_copy_global_to_local %ptr2Inst, %memDesc2Inst : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
256|     ttg.async_commit_group
257|     ttg.async_copy_global_to_local %ptr2Inst, %memDesc2Inst : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
258|     ttg.async_commit_group
```
**EN:** This function-oriented block defines or enters `for_nested_control_flow`. Within it, the test exercises ttg.async_copy_global_to_local, async copy commit groups, tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `for_nested_control_flow` 为核心。测试在其中演示 ttg.async_copy_global_to_local、异步拷贝提交组、tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 259-268
```mlir
259|     ttg.async_copy_global_to_local %ptr2Inst, %memDesc2Inst : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
260|     ttg.async_commit_group
261|     ttg.async_copy_global_to_local %ptr2Inst, %memDesc2Inst : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
262|     ttg.async_commit_group
263|     ttg.async_copy_global_to_local %ptr2Inst, %memDesc2Inst : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
264|     ttg.async_commit_group
265| 
266|     // The loop has 3 commits group which produce 2,1,1 (in program order) async instructions
267|     scf.for %arg14 = %c0_i32 to %arg0 step %c1_i32 : i32 {
268|       // 2 full loop iterations => 8
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining ttg.async_copy_global_to_local, async copy commit groups, structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 ttg.async_copy_global_to_local、异步拷贝提交组、结构化循环。

### Lines 269-275
```mlir
269|       // CHECK: amdg.async_wait {num_inst = 8
270|       ttg.async_wait {num = 6: i32}
271| 
272|       ttg.async_copy_global_to_local %ptr2Inst, %memDesc2Inst : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
273|       ttg.async_commit_group
274| 
275|       // Wait on 1 full loop iteration (4) + the commit group above (2)
```
**EN:** This operational block carries the concrete IR steps for the test, centered on async wait synchronization, ttg.async_copy_global_to_local, async copy commit groups. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 异步等待同步、ttg.async_copy_global_to_local、异步拷贝提交组。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 276-293
```mlir
276|       // CHECK: amdg.async_wait {num_inst = 6
277|       ttg.async_wait {num = 4: i32}
278| 
279|       scf.if %cond {
280|         ttg.async_copy_global_to_local %ptr1Inst, %memDesc1Inst : tensor<64x16x!tt.ptr<f16>, #blocked> -> <64x16xf16, #shared, #smem, mutable>
281|       } else {
282|         ttg.async_copy_global_to_local %ptr2Inst, %memDesc2Inst : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
283|       }
284|       ttg.async_commit_group
285| 
286|       scf.if %cond {
287|         ttg.async_copy_global_to_local %ptr2Inst, %memDesc2Inst : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
288|       } else {
289|         ttg.async_copy_global_to_local %ptr1Inst, %memDesc1Inst : tensor<64x16x!tt.ptr<f16>, #blocked> -> <64x16xf16, #shared, #smem, mutable>
290|       }
291|       ttg.async_commit_group
292| 
293|       // Wait on 1 full loop iteration (4) + the commit group above (1)
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining ttg.async_copy_global_to_local, structured conditionals, async copy commit groups, async wait synchronization.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 ttg.async_copy_global_to_local、结构化条件分支、异步拷贝提交组、异步等待同步。

### Lines 294-299
```mlir
294|       // CHECK: amdg.async_wait {num_inst = 5
295|       ttg.async_wait {num = 4: i32}
296| 
297|       scf.yield
298|     }
299|     // 2 Full loop iterations (2 * 4)
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining async wait synchronization, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 异步等待同步、循环/分支产出值。

### Lines 300-305
```mlir
300|     // CHECK: amdg.async_wait {num_inst = 8
301|     ttg.async_wait {num = 6: i32}
302| 
303|     tt.return
304|   }
305| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on async wait synchronization, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 异步等待同步、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 306-306
```mlir
306|   // CHECK-LABEL: while_without_async_ops
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: while_without_async_ops anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: while_without_async_ops 这样的标签用于锚定匹配范围。

### Lines 307-321
```mlir
307|   tt.func public @while_without_async_ops(
308|         %cond: i1,
309|         %arg0: i32,
310|         %memDesc2Inst: !ttg.memdesc<128x16xf16, #shared, #smem, mutable>,
311|         %ptr2Inst: tensor<128x16x!tt.ptr<f16>, #blocked>  {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>},
312|         %memDesc1Inst: !ttg.memdesc<64x16xf16, #shared, #smem, mutable>,
313|         %ptr1Inst: tensor<64x16x!tt.ptr<f16>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>}) {
314| 
315|     %c0_i32 = arith.constant 0 : i32
316|     %c1_i32 = arith.constant 1 : i32
317| 
318|     // Check we are not getting stuck in loops with no async ops
319|     ttg.async_copy_global_to_local %ptr2Inst, %memDesc2Inst : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
320|     ttg.async_commit_group
321|     %69 = scf.while (%arg10 = %cond) : (i1) -> (i1) {
```
**EN:** This function-oriented block defines or enters `while_without_async_ops`. Within it, the test exercises tt.func, constants, ttg.async_copy_global_to_local, async copy commit groups, scf.while, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `while_without_async_ops` 为核心。测试在其中演示 tt.func、常量、ttg.async_copy_global_to_local、异步拷贝提交组、scf.while，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 322-326
```mlir
322|       // CHECK: amdg.async_wait {num_inst = 2
323|       ttg.async_wait {num = 1: i32}
324|       scf.condition(%arg10) %arg10 : i1
325|     } do {
326|     ^bb0(%arg12: i1):
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining async wait synchronization, scf.condition.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 异步等待同步、scf.condition。

### Lines 327-330
```mlir
327|       // CHECK: amdg.async_wait {num_inst = 2
328|       ttg.async_wait {num = 1: i32}
329|       scf.yield %arg12 : i1
330|     }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining async wait synchronization, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 异步等待同步、循环/分支产出值。

### Lines 331-336
```mlir
331|     // CHECK: amdg.async_wait {num_inst = 2
332|     ttg.async_wait {num = 1: i32}
333| 
334|     tt.return
335|   }
336| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on async wait synchronization, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 异步等待同步、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 337-337
```mlir
337|   // CHECK-LABEL: while_async_op_in_before_block
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: while_async_op_in_before_block anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: while_async_op_in_before_block 这样的标签用于锚定匹配范围。

### Lines 338-352
```mlir
338|   tt.func public @while_async_op_in_before_block(
339|         %cond: i1,
340|         %arg0: i32,
341|         %memDesc2Inst: !ttg.memdesc<128x16xf16, #shared, #smem, mutable>,
342|         %ptr2Inst: tensor<128x16x!tt.ptr<f16>, #blocked>  {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>},
343|         %memDesc1Inst: !ttg.memdesc<64x16xf16, #shared, #smem, mutable>,
344|         %ptr1Inst: tensor<64x16x!tt.ptr<f16>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>}) {
345| 
346|     // Check we are following control flow and count inside the before block
347|     ttg.async_copy_global_to_local %ptr2Inst, %memDesc2Inst : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
348|     ttg.async_commit_group
349|     ttg.async_copy_global_to_local %ptr2Inst, %memDesc2Inst : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
350|     ttg.async_commit_group
351|     ttg.async_copy_global_to_local %ptr2Inst, %memDesc2Inst : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
352|     ttg.async_commit_group
```
**EN:** This function-oriented block defines or enters `while_async_op_in_before_block`. Within it, the test exercises ttg.async_copy_global_to_local, async copy commit groups, tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `while_async_op_in_before_block` 为核心。测试在其中演示 ttg.async_copy_global_to_local、异步拷贝提交组、tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 353-357
```mlir
353|     // CHECK: amdg.async_wait {num_inst = 6
354|     ttg.async_wait {num = 3: i32}
355| 
356|     %70 = scf.while (%arg10 = %cond) : (i1) -> (i1) {
357|       // Count before block 3 times
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining async wait synchronization, scf.while.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 异步等待同步、scf.while。

### Lines 358-365
```mlir
358|       // CHECK: amdg.async_wait {num_inst = 3
359|       ttg.async_wait {num = 3: i32}
360|       ttg.async_copy_global_to_local %ptr1Inst, %memDesc1Inst : tensor<64x16x!tt.ptr<f16>, #blocked> -> <64x16xf16, #shared, #smem, mutable>
361|       ttg.async_commit_group
362|       scf.condition(%arg10) %arg10 : i1
363|     } do {
364|     ^bb0(%arg12: i1):
365|       // Count before block 3 times
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining async wait synchronization, ttg.async_copy_global_to_local, async copy commit groups, scf.condition.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 异步等待同步、ttg.async_copy_global_to_local、异步拷贝提交组、scf.condition。

### Lines 366-370
```mlir
366|       // CHECK: amdg.async_wait {num_inst = 3
367|       ttg.async_wait {num = 3: i32}
368|       scf.yield %arg12 : i1
369|     }
370|     // Count before block 3 times
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining async wait synchronization, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 异步等待同步、循环/分支产出值。

### Lines 371-376
```mlir
371|     // CHECK: amdg.async_wait {num_inst = 3
372|     ttg.async_wait {num = 3: i32}
373| 
374|     tt.return
375|   }
376| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on async wait synchronization, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 异步等待同步、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 377-377
```mlir
377|   // CHECK-LABEL: while_async_op_in_after_block
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: while_async_op_in_after_block anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: while_async_op_in_after_block 这样的标签用于锚定匹配范围。

### Lines 378-392
```mlir
378|   tt.func public @while_async_op_in_after_block(
379|         %cond: i1,
380|         %arg0: i32,
381|         %memDesc2Inst: !ttg.memdesc<128x16xf16, #shared, #smem, mutable>,
382|         %ptr2Inst: tensor<128x16x!tt.ptr<f16>, #blocked>  {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>},
383|         %memDesc1Inst: !ttg.memdesc<64x16xf16, #shared, #smem, mutable>,
384|         %ptr1Inst: tensor<64x16x!tt.ptr<f16>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>}) {
385| 
386|     // Check we are following control flow and count inside the after block
387|     ttg.async_copy_global_to_local %ptr2Inst, %memDesc2Inst : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
388|     ttg.async_commit_group
389|     ttg.async_copy_global_to_local %ptr2Inst, %memDesc2Inst : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
390|     ttg.async_commit_group
391|     ttg.async_copy_global_to_local %ptr2Inst, %memDesc2Inst : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
392|     ttg.async_commit_group
```
**EN:** This function-oriented block defines or enters `while_async_op_in_after_block`. Within it, the test exercises ttg.async_copy_global_to_local, async copy commit groups, tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `while_async_op_in_after_block` 为核心。测试在其中演示 ttg.async_copy_global_to_local、异步拷贝提交组、tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 393-397
```mlir
393|     // CHECK: amdg.async_wait {num_inst = 6
394|     ttg.async_wait {num = 3: i32}
395| 
396|     %71 = scf.while (%arg10 = %cond) : (i1) -> (i1) {
397|       // Count after block 3 times
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining async wait synchronization, scf.while.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 异步等待同步、scf.while。

### Lines 398-405
```mlir
398|       // CHECK: amdg.async_wait {num_inst = 3
399|       ttg.async_wait {num = 3: i32}
400|       scf.condition(%arg10) %arg10 : i1
401|     } do {
402|     ^bb0(%arg12: i1):
403|       ttg.async_copy_global_to_local %ptr1Inst, %memDesc1Inst : tensor<64x16x!tt.ptr<f16>, #blocked> -> <64x16xf16, #shared, #smem, mutable>
404|       ttg.async_commit_group
405|       // Count after block 4 times
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining async wait synchronization, scf.condition, ttg.async_copy_global_to_local, async copy commit groups.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 异步等待同步、scf.condition、ttg.async_copy_global_to_local、异步拷贝提交组。

### Lines 406-410
```mlir
406|       // CHECK: amdg.async_wait {num_inst = 4
407|       ttg.async_wait {num = 4: i32} // 4 because we moved the wait after the next prefetch
408|       scf.yield %arg12 : i1
409|     }
410|     // Count after block 3 times
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining async wait synchronization, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 异步等待同步、循环/分支产出值。

### Lines 411-417
```mlir
411|     // CHECK: amdg.async_wait {num_inst = 3
412|     ttg.async_wait {num = 3: i32}
413| 
414|     tt.return
415|   }
416| 
417|   //CHECK-LABEL: nested_loops_and_if
```
**EN:** This operational block carries the concrete IR steps for the test, centered on async wait synchronization, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 异步等待同步、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 418-435
```mlir
418|   tt.func public @nested_loops_and_if(
419|         %cond: i1,
420|         %arg0: i32,
421|         %memDesc2Inst: !ttg.memdesc<128x16xf16, #shared, #smem, mutable>,
422|         %ptr2Inst: tensor<128x16x!tt.ptr<f16>, #blocked>  {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>},
423|         %memDesc1Inst: !ttg.memdesc<64x16xf16, #shared, #smem, mutable>,
424|         %ptr1Inst: tensor<64x16x!tt.ptr<f16>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>}) {
425| 
426|     %c0_i32 = arith.constant 0 : i32
427|     %c1_i32 = arith.constant 1 : i32
428| 
429|     ttg.async_copy_global_to_local %ptr1Inst, %memDesc1Inst : tensor<64x16x!tt.ptr<f16>, #blocked> -> <64x16xf16, #shared, #smem, mutable>
430|     ttg.async_commit_group
431|     ttg.async_copy_global_to_local %ptr1Inst, %memDesc1Inst : tensor<64x16x!tt.ptr<f16>, #blocked> -> <64x16xf16, #shared, #smem, mutable>
432|     ttg.async_commit_group
433|     ttg.async_copy_global_to_local %ptr1Inst, %memDesc1Inst : tensor<64x16x!tt.ptr<f16>, #blocked> -> <64x16xf16, #shared, #smem, mutable>
434|     ttg.async_commit_group
435|     ttg.async_copy_global_to_local %ptr1Inst, %memDesc1Inst : tensor<64x16x!tt.ptr<f16>, #blocked> -> <64x16xf16, #shared, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `nested_loops_and_if`. Within it, the test exercises ttg.async_copy_global_to_local, async copy commit groups, tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `nested_loops_and_if` 为核心。测试在其中演示 ttg.async_copy_global_to_local、异步拷贝提交组、tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 436-440
```mlir
436|     ttg.async_commit_group
437|     ttg.async_copy_global_to_local %ptr1Inst, %memDesc1Inst : tensor<64x16x!tt.ptr<f16>, #blocked> -> <64x16xf16, #shared, #smem, mutable>
438|     ttg.async_commit_group
439|     ttg.async_copy_global_to_local %ptr1Inst, %memDesc1Inst : tensor<64x16x!tt.ptr<f16>, #blocked> -> <64x16xf16, #shared, #smem, mutable>
440|     ttg.async_commit_group
```
**EN:** This operational block carries the concrete IR steps for the test, centered on async copy commit groups, ttg.async_copy_global_to_local. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 异步拷贝提交组、ttg.async_copy_global_to_local。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 441-445
```mlir
441|     // CHECK: amdg.async_wait {num_inst = 6
442|     ttg.async_wait {num = 6: i32}
443| 
444|     %70 = scf.while (%arg10 = %cond) : (i1) -> (i1) {
445|       // Escape while and count prologue = 6
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining async wait synchronization, scf.while.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 异步等待同步、scf.while。

### Lines 446-454
```mlir
446|       // CHECK: amdg.async_wait {num_inst = 6
447|       ttg.async_wait {num = 6: i32}
448|       ttg.async_copy_global_to_local %ptr2Inst, %memDesc2Inst : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
449|       ttg.async_commit_group
450|       // 2 Instructions
451|       scf.condition(%arg10) %arg10 : i1
452|     } do {
453|     ^bb0(%arg12: i1):
454|       // 1 commit group in Before-block + 5 commits groups in prologue = 7
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining async wait synchronization, ttg.async_copy_global_to_local, async copy commit groups, scf.condition.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 异步等待同步、ttg.async_copy_global_to_local、异步拷贝提交组、scf.condition。

### Lines 455-465
```mlir
455|       // CHECK: amdg.async_wait {num_inst = 7
456|       ttg.async_wait {num = 6: i32}
457|       ttg.async_copy_global_to_local %ptr2Inst, %memDesc2Inst : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
458|       ttg.async_commit_group
459|       // 2 Instructions
460| 
461|       scf.for %arg14 = %c0_i32 to %arg0 step %c1_i32 : i32 {
462|         ttg.async_copy_global_to_local %ptr2Inst, %memDesc2Inst : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
463|         // 2 Instructions
464|         ttg.async_commit_group
465|         // 1 commit group(2) to escape for, 1 commits group(2) in rest of while after block, 1 commit group (2) in while before block and 3 commits group in prologue = 9
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining ttg.async_copy_global_to_local, async copy commit groups, async wait synchronization, structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 ttg.async_copy_global_to_local、异步拷贝提交组、异步等待同步、结构化循环。

### Lines 466-473
```mlir
466|         // CHECK: amdg.async_wait {num_inst = 9
467|         ttg.async_wait {num = 6: i32}
468| 
469|         scf.if %cond {
470|           ttg.async_copy_global_to_local %ptr1Inst, %memDesc1Inst : tensor<64x16x!tt.ptr<f16>, #blocked> -> <64x16xf16, #shared, #smem, mutable>
471|           ttg.async_copy_global_to_local %ptr2Inst, %memDesc2Inst : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
472| 
473|           // Same as above but we also have to count the 2 async_copies above = 9+3
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining ttg.async_copy_global_to_local, async wait synchronization, structured conditionals.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 ttg.async_copy_global_to_local、异步等待同步、结构化条件分支。

### Lines 474-487
```mlir
474|           // CHECK: amdg.async_wait {num_inst = 12
475|           ttg.async_wait {num = 6: i32}
476|         } else {
477|           ttg.async_copy_global_to_local %ptr2Inst, %memDesc2Inst : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
478|         }
479|         // 2 Instructions (else)
480|         ttg.async_commit_group
481| 
482|         scf.if %cond {
483|           ttg.async_copy_global_to_local %ptr1Inst, %memDesc1Inst : tensor<64x16x!tt.ptr<f16>, #blocked> -> <64x16xf16, #shared, #smem, mutable>
484|           ttg.async_copy_global_to_local %ptr2Inst, %memDesc2Inst : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
485|           // 3 Instructions
486|           ttg.async_commit_group
487|           // 1 commit group (3) in this block, 2 commits group in the rest of the for body (2+2), 1 commits group(2) in rest of while after block, 1 commit group (2) in while before block, 1 commit group (1) in epilogue = 12
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining ttg.async_copy_global_to_local, async copy commit groups, async wait synchronization, structured conditionals.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 ttg.async_copy_global_to_local、异步拷贝提交组、异步等待同步、结构化条件分支。

### Lines 488-491
```mlir
488|           // CHECK: amdg.async_wait {num_inst = 12
489|           ttg.async_wait {num = 6: i32}
490|         }
491|         // Same as above but skips the if (first commit group(3)) and instead counts one more in the prologue (1) = 10
```
**EN:** This operational block carries the concrete IR steps for the test, centered on async wait synchronization. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 异步等待同步。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 492-501
```mlir
492|         // CHECK: amdg.async_wait {num_inst = 10
493|         ttg.async_wait {num = 6: i32}
494|         scf.for %arg15 = %c0_i32 to %arg0 step %c1_i32 : i32 {
495|           ttg.async_copy_global_to_local %ptr1Inst, %memDesc1Inst : tensor<64x16x!tt.ptr<f16>, #blocked> -> <64x16xf16, #shared, #smem, mutable>
496|           // 1 Instruction
497|           ttg.async_commit_group
498|           ttg.async_copy_global_to_local %ptr2Inst, %memDesc2Inst : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
499|           // 2 Instructions
500|           ttg.async_commit_group
501|           // Just staying in the loop is the lowest path (3 per iteration and we do 3 iterations)
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining ttg.async_copy_global_to_local, async copy commit groups, async wait synchronization, structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 ttg.async_copy_global_to_local、异步拷贝提交组、异步等待同步、结构化循环。

### Lines 502-506
```mlir
502|           // CHECK: amdg.async_wait {num_inst = 9
503|           ttg.async_wait {num = 6: i32}
504|           scf.yield
505|         }
506|         // Just stay in the inner loop for the lowest path
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining async wait synchronization, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 异步等待同步、循环/分支产出值。

### Lines 507-513
```mlir
507|         // CHECK: amdg.async_wait {num_inst = 9
508|         ttg.async_wait {num = 6: i32}
509|         scf.yield
510|       }
511|       scf.yield %arg12 : i1
512|     }
513|     // While before-body (2) + 5 prologue groups = 7
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, async wait synchronization.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、异步等待同步。

### Lines 514-519
```mlir
514|     // CHECK: amdg.async_wait {num_inst = 7
515|     ttg.async_wait {num = 6: i32}
516| 
517|     tt.return
518|   }
519| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on async wait synchronization, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 异步等待同步、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 520-520
```mlir
520|   // CHECK-LABEL: async_wait_with_execute_regions
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: async_wait_with_execute_regions anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: async_wait_with_execute_regions 这样的标签用于锚定匹配范围。

### Lines 521-538
```mlir
521|   tt.func public @async_wait_with_execute_regions(
522|         %memDesc1Inst: !ttg.memdesc<64x16xf16, #shared, #smem, mutable>,
523|         %ptr1Inst: tensor<64x16x!tt.ptr<f16>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>},
524|         %memDesc2Inst: !ttg.memdesc<128x16xf16, #shared, #smem, mutable>,
525|         %ptr2Inst: tensor<128x16x!tt.ptr<f16>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>}) {
526| 
527|     scf.execute_region {
528|       scf.execute_region {
529|         // Emits 1 instruction
530|         ttg.async_copy_global_to_local %ptr1Inst, %memDesc1Inst : tensor<64x16x!tt.ptr<f16>, #blocked> -> <64x16xf16, #shared, #smem, mutable>
531|         ttg.async_commit_group
532|         scf.yield
533|       } {triton.warp_pipeline.stage = "stage0"}
534| 
535|       scf.execute_region {
536|         // Emits 2 instructions
537|         ttg.async_copy_global_to_local %ptr2Inst, %memDesc2Inst : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
538|         ttg.async_commit_group
```
**EN:** This function-oriented block defines or enters `async_wait_with_execute_regions`. Within it, the test exercises scf.execute_region, tt.func, ttg.async_copy_global_to_local, async copy commit groups, loop/if yielded values, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_wait_with_execute_regions` 为核心。测试在其中演示 scf.execute_region、tt.func、ttg.async_copy_global_to_local、异步拷贝提交组、循环/分支产出值，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 539-543
```mlir
539| 
540|         scf.yield
541|       } {triton.warp_pipeline.stage = "stage1"}
542| 
543|       // Wait for both execute regions
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值。

### Lines 544-547
```mlir
544|       // CHECK: amdg.async_wait {num_inst = 3
545|       ttg.async_wait {num = 2 : i32}
546| 
547|       // Check that we only traverse each execute region once
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 548-551
```mlir
548|       // CHECK: amdg.async_wait {num_inst = 3
549|       ttg.async_wait {num = 6 : i32}
550| 
551|       // Wait only for the second execute region
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 552-558
```mlir
552|       // CHECK: amdg.async_wait {num_inst = 2
553|       ttg.async_wait {num = 1 : i32}
554| 
555|       scf.yield
556|     }
557| 
558|     // Wait for both nested execute regions
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining async wait synchronization, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 异步等待同步、循环/分支产出值。

### Lines 559-566
```mlir
559|     // CHECK: amdg.async_wait {num_inst = 3
560|     ttg.async_wait {num = 2 : i32}
561| 
562|     tt.return
563|   }
564| 
565| }
566| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on async wait synchronization, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 异步等待同步、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 567-567
```mlir
567| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 568-573
```mlir
568| 
569| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
570| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
571| #smem = #ttg.shared_memory
572| #idx_i32_parent = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [0, 1]}>
573| #idx_i16_parent = #ttg.blocked<{sizePerThread = [1, 64], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [0, 1]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 574-574
```mlir
574| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 575-575
```mlir
575|   // CHECK-LABEL: tdm_gather_scatter_multiple_instructions
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tdm_gather_scatter_multiple_instructions anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tdm_gather_scatter_multiple_instructions 这样的标签用于锚定匹配范围。

### Lines 576-593
```mlir
576|   tt.func public @tdm_gather_scatter_multiple_instructions(
577|     %memDesc: !ttg.memdesc<256x128xf16, #shared, #smem, mutable>,
578|     %tensorDesc: !tt.tensordesc<64x128xf16>,
579|     %row_indices_i32: tensor<64xi32, #ttg.slice<{dim = 0, parent = #idx_i32_parent}>>,
580|     %row_indices_i16: tensor<256xi16, #ttg.slice<{dim = 0, parent = #idx_i16_parent}>>,
581|     %pred: i32
582|   ) {
583|     %c0_i32 = arith.constant 0 : i32
584| 
585|     // Gather with i32 indices: sizePerThread=16, 4 warps, maxPerInstr=8 => 2 instructions
586|     amdg.async_tdm_gather %tensorDesc[%row_indices_i32, %c0_i32] to %memDesc, pred = %pred : tensor<64xi32, #ttg.slice<{dim = 0, parent = #idx_i32_parent}>>, !ttg.memdesc<256x128xf16, #shared, #smem, mutable> -> !tt.tensordesc<64x128xf16>
587|     // Scatter with i32 indices: 2 instructions
588|     amdg.async_tdm_scatter %tensorDesc[%row_indices_i32, %c0_i32] from %memDesc : tensor<64xi32, #ttg.slice<{dim = 0, parent = #idx_i32_parent}>>, !ttg.memdesc<256x128xf16, #shared, #smem, mutable> -> !tt.tensordesc<64x128xf16>
589|     // Gather with i16 indices: sizePerThread=64, 4 warps, maxPerInstr=16 => 4 instructions
590|     amdg.async_tdm_gather %tensorDesc[%row_indices_i16, %c0_i32] to %memDesc, pred = %pred : tensor<256xi16, #ttg.slice<{dim = 0, parent = #idx_i16_parent}>>, !ttg.memdesc<256x128xf16, #shared, #smem, mutable> -> !tt.tensordesc<64x128xf16>
591|     // Scatter with i16 indices: 4 instructions
592|     amdg.async_tdm_scatter %tensorDesc[%row_indices_i16, %c0_i32] from %memDesc : tensor<256xi16, #ttg.slice<{dim = 0, parent = #idx_i16_parent}>>, !ttg.memdesc<256x128xf16, #shared, #smem, mutable> -> !tt.tensordesc<64x128xf16>
593| 
```
**EN:** This function-oriented block defines or enters `tdm_gather_scatter_multiple_instructions`. Within it, the test exercises tt.func, amdg.async_tdm_gather, amdg.async_tdm_scatter, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tdm_gather_scatter_multiple_instructions` 为核心。测试在其中演示 tt.func、amdg.async_tdm_gather、amdg.async_tdm_scatter、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 594-594
```mlir
594|     // i32 ops emit 2 instructions each, i16 ops emit 4 each => total 12 instructions
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 595-596
```mlir
595|     // CHECK: amdg.async_tdm_intrinsic_wait {count = 0
596|     amdg.async_tdm_wait {num = 0 : i32}
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 597-598
```mlir
597|     // CHECK: amdg.async_tdm_intrinsic_wait {count = 4
598|     amdg.async_tdm_wait {num = 1 : i32}
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 599-600
```mlir
599|     // CHECK: amdg.async_tdm_intrinsic_wait {count = 8
600|     amdg.async_tdm_wait {num = 2 : i32}
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 601-602
```mlir
601|     // CHECK: amdg.async_tdm_intrinsic_wait {count = 10
602|     amdg.async_tdm_wait {num = 3 : i32}
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 603-609
```mlir
603|     // CHECK: amdg.async_tdm_intrinsic_wait {count = 12
604|     amdg.async_tdm_wait {num = 4 : i32}
605| 
606|     tt.return
607|   }
608| }
609| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.async_tdm_wait, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.async_tdm_wait、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 610-610
```mlir
610| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 611-614
```mlir
611| 
612| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
613| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
614| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 615-615
```mlir
615| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 616-616
```mlir
616|   // CHECK-LABEL: tdm_load_store_single_instruction
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tdm_load_store_single_instruction anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tdm_load_store_single_instruction 这样的标签用于锚定匹配范围。

### Lines 617-628
```mlir
617|   tt.func public @tdm_load_store_single_instruction(
618|     %memDesc: !ttg.memdesc<64x128xf16, #shared, #smem, mutable>,
619|     %tensorDesc: !tt.tensordesc<64x128xf16>,
620|     %pred: i32
621|   ) {
622|     %c0_i32 = arith.constant 0 : i32
623| 
624|     %0 = amdg.async_tdm_copy_global_to_local %tensorDesc[%c0_i32, %c0_i32] into %memDesc, pred = %pred : !tt.tensordesc<64x128xf16> -> !ttg.memdesc<64x128xf16, #shared, #smem, mutable>
625|     amdg.async_tdm_copy_local_to_global %tensorDesc[%c0_i32, %c0_i32] from %memDesc : !ttg.memdesc<64x128xf16, #shared, #smem, mutable> -> !tt.tensordesc<64x128xf16>
626|     %1 = amdg.async_tdm_copy_global_to_local %tensorDesc[%c0_i32, %c0_i32] into %memDesc, pred = %pred : !tt.tensordesc<64x128xf16> -> !ttg.memdesc<64x128xf16, #shared, #smem, mutable>
627|     amdg.async_tdm_copy_local_to_global %tensorDesc[%c0_i32, %c0_i32] from %memDesc : !ttg.memdesc<64x128xf16, #shared, #smem, mutable> -> !tt.tensordesc<64x128xf16>
628| 
```
**EN:** This function-oriented block defines or enters `tdm_load_store_single_instruction`. Within it, the test exercises tt.func, amdg.async_tdm_copy_global_to_local, amdg.async_tdm_copy_local_to_global, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tdm_load_store_single_instruction` 为核心。测试在其中演示 tt.func、amdg.async_tdm_copy_global_to_local、amdg.async_tdm_copy_local_to_global、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 629-630
```mlir
629|     // CHECK: amdg.async_tdm_intrinsic_wait {count = 0
630|     amdg.async_tdm_wait {num = 0 : i32}
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 631-632
```mlir
631|     // CHECK: amdg.async_tdm_intrinsic_wait {count = 1
632|     amdg.async_tdm_wait {num = 1 : i32}
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 633-634
```mlir
633|     // CHECK: amdg.async_tdm_intrinsic_wait {count = 2
634|     amdg.async_tdm_wait {num = 2 : i32}
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 635-636
```mlir
635|     // CHECK: amdg.async_tdm_intrinsic_wait {count = 3
636|     amdg.async_tdm_wait {num = 3 : i32}
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 637-642
```mlir
637|     // CHECK: amdg.async_tdm_intrinsic_wait {count = 4
638|     amdg.async_tdm_wait {num = 4 : i32}
639| 
640|     tt.return
641|   }
642| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.async_tdm_wait, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.async_tdm_wait、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `--tritonamdgpu-update-async-wait-count=gfx-arch=gfx1250`
- **CN:** 主要 pass 选项：`-split-input-file`，`--tritonamdgpu-update-async-wait-count=gfx-arch=gfx1250`
- **EN:** Dominant operations include `ttg.async_copy_global_to_local`, `ttg.async_commit_group`, `ttg.async_wait`, `tt.func`, `scf.yield`, `tt.return`, `arith.constant`, `scf.if`, `amdg.async_tdm_wait`, `scf.for`.
- **CN:** 主要操作包括 `ttg.async_copy_global_to_local`、`ttg.async_commit_group`、`ttg.async_wait`、`tt.func`、`scf.yield`、`tt.return`、`arith.constant`、`scf.if`、`amdg.async_tdm_wait`、`scf.for`。
- **EN:** The file contains 2 independently testable section(s). Check styles used: CHECK x57, CHECK-LABEL x15. Important labels include simple_waitcnt, simple_waitcnt_non_committed_async_ops, wait_if_without_else, check_wait_nested_ifs. It validates that the selected pass pipeline rewrites each labeled test case into the checked form.
- **CN:** 该文件包含 2 个可独立测试的分段。使用的检查类型：CHECK ×57，CHECK-LABEL ×15。 关键标签包括 simple_waitcnt，simple_waitcnt_non_committed_async_ops，wait_if_without_else，check_wait_nested_ifs。 它验证所选 pass 流水线会把每个带标签的测试用例重写成检查所要求的形式。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `ttg`: TritonGPU layout, memory, and GPU-specific operations.
- **CN:** `ttg`：TritonGPU 布局、内存和 GPU 专用操作。
- **EN:** `tt`: Triton high-level tensor/pointer operations.
- **CN:** `tt`：Triton 高层张量/指针操作。
- **EN:** `scf`: Structured control flow for loops and conditionals.
- **CN:** `scf`：用于循环与条件分支的结构化控制流。
- **EN:** `arith`: MLIR arithmetic dialect for scalar/tensor math.
- **CN:** `arith`：用于标量/张量运算的 MLIR arithmetic 方言。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。