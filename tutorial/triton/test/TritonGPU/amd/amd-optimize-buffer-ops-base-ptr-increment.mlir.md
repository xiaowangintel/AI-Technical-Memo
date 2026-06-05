# amd-optimize-buffer-ops-base-ptr-increment.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/amd/amd-optimize-buffer-ops-base-ptr-increment.mlir`
- **EN:** Test for the MLIR/Triton pipeline `--tritonamdgpu-convert-buffer-ops="gfx-arch=gfx950", --tritonamdgpu-optimize-buffer-op-ptr|, --check-prefixes=CHECK` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `--tritonamdgpu-convert-buffer-ops="gfx-arch=gfx950", --tritonamdgpu-optimize-buffer-op-ptr|, --check-prefixes=CHECK` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file --tritonamdgpu-convert-buffer-ops="gfx-arch=gfx950" --tritonamdgpu-optimize-buffer-op-ptr| FileCheck %s --check-prefixes=CHECK`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file --tritonamdgpu-convert-buffer-ops="gfx-arch=gfx950" --tritonamdgpu-optimize-buffer-op-ptr| FileCheck %s --check-prefixes=CHECK`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt %s -split-input-file --tritonamdgpu-convert-buffer-ops="gfx-arch=gfx950" --tritonamdgpu-optimize-buffer-op-ptr| FileCheck %s --check-prefixes=CHECK
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file --tritonamdgpu-convert-buffer-ops="gfx-arch=gfx950" --tritonamdgpu-optimize-buffer-op-ptr| FileCheck %s --check-prefixes=CHECK` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file --tritonamdgpu-convert-buffer-ops="gfx-arch=gfx950" --tritonamdgpu-optimize-buffer-op-ptr| FileCheck %s --check-prefixes=CHECK`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-12
```mlir
 3| // CHECK-LABEL: add_after_load
 4| // CHECK-DAG: [[X_OFFSET_CST:%.*]] = arith.constant dense<123>
 5| // CHECK-DAG: [[Y_OFFSET_CST:%.*]] = arith.constant dense<321>
 6| // CHECK: scf.for {{.*}} iter_args({{.*}}, {{.*}}, [[X_BASE:%.*]] = {{.*}}, [[Y_BASE:%.*]] = {{.*}})
 7| // CHECK:   amdg.buffer_load [[X_BASE]]{{\[}}[[X_OFFSET_CST]]{{\]}} :
 8| // CHECK:   amdg.buffer_load [[Y_BASE]]{{\[}}[[Y_OFFSET_CST]]{{\]}} cacheModifier = cg :
 9| // CHECK:   [[NEXT_X_BASE:%.*]] = tt.addptr [[X_BASE]], %c64_i32
10| // CHECK:   [[NEXT_Y_BASE:%.*]] = tt.addptr [[Y_BASE]]
11| // CHECK:   scf.yield {{.*}}, [[NEXT_X_BASE]], [[NEXT_Y_BASE]]
12| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: add_after_load anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: add_after_load 这样的标签用于锚定匹配范围。

### Lines 13-15
```mlir
13| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 8], warpsPerCTA = [1, 1], order = [1, 0]}>
14| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
15| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 16-16
```mlir
16| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 17-34
```mlir
17|   tt.func public @add_after_load(%X: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %Y: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %stride: i32) attributes {noinline = false} {
18|     %cst = arith.constant dense<64> : tensor<16x64xi32, #blocked>
19|     %c0 = arith.constant 0 : index
20|     %c128 = arith.constant 128 : index
21|     %c64_i32 = arith.constant 64 : i32
22|     %c1 = arith.constant 1 : index
23| 
24|     %min_stride = arith.constant 1 : i32
25|     %max_stride = arith.constant 1024 : i32
26|     %0 = arith.cmpi sge, %stride, %min_stride : i32
27|     llvm.intr.assume %0 : i1
28|     %1 = arith.cmpi sle, %stride, %max_stride : i32
29|     llvm.intr.assume %1 : i1
30| 
31|     %Xoffset_init = arith.constant dense<123> : tensor<16x64xi32, #blocked>
32|     %Yoffset_init = arith.constant dense<321> : tensor<64x32xi32, #blocked>
33| 
34|     %x_dummy_buffer = ttg.local_alloc : () -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable, 16x64>
```
**EN:** This function-oriented block defines or enters `add_after_load`. Within it, the test exercises constants, tt.func, integer comparisons, llvm.intr, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `add_after_load` 为核心。测试在其中演示 常量、tt.func、整数比较、llvm.intr、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 35-52
```mlir
35|     %y_dummy_buffer = ttg.local_alloc : () -> !ttg.memdesc<64x32xf16, #shared, #smem, mutable, 64x32>
36| 
37|     %tmp = arith.muli %stride, %c64_i32 : i32
38|     %step = tt.splat %tmp : i32 -> tensor<64x32xi32, #blocked>
39|     %for:2 = scf.for %idx = %c0 to %c128 step %c1 iter_args(%Xoffset = %Xoffset_init, %Yoffset = %Yoffset_init) -> (tensor<16x64xi32, #blocked>, tensor<64x32xi32, #blocked>) {
40|       %x = amdg.buffer_load %X[%Xoffset] : tensor<16x64xf16, #blocked>
41|       %y = amdg.buffer_load %Y[%Yoffset] cacheModifier = cg : tensor<64x32xf16, #blocked>
42| 
43|       ttg.local_store %x, %x_dummy_buffer : tensor<16x64xf16, #blocked> -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable, 16x64>
44|       ttg.local_store %y, %y_dummy_buffer : tensor<64x32xf16, #blocked> -> !ttg.memdesc<64x32xf16, #shared, #smem, mutable, 64x32>
45| 
46|       %Xoffset_next = arith.addi %Xoffset, %cst : tensor<16x64xi32, #blocked>
47|       %Yoffset_next = arith.addi %Yoffset, %step : tensor<64x32xi32, #blocked>
48|       scf.yield %Xoffset_next, %Yoffset_next : tensor<16x64xi32, #blocked>, tensor<64x32xi32, #blocked>
49|     }
50|     tt.return
51|   }
52| }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining amdg.buffer_load, local/shared memory stores, integer additions, shared/local memory allocation, integer multiplications.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 amdg.buffer_load、本地/共享内存存储、整数加法、共享/本地内存分配、整数乘法。

### Lines 54-55
```mlir
54| 
55| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 55-62
```mlir
55| 
56| // CHECK-LABEL: buffer_load_to_local
57| // CHECK-DAG: [[X_OFFSET_CST:%.*]] = arith.constant dense<123>
58| // CHECK: scf.for {{.*}} iter_args({{.*}}, [[X_BASE:%.*]] = {{.*}}
59| // CHECK:   amdg.buffer_load_to_local [[X_BASE]]{{\[}}[[X_OFFSET_CST]]{{\]}}
60| // CHECK:   [[NEXT_X_BASE:%.*]] = tt.addptr [[X_BASE]], %c64_i32
61| // CHECK:   scf.yield {{.*}}, [[NEXT_X_BASE]]
62| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: buffer_load_to_local anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: buffer_load_to_local 这样的标签用于锚定匹配范围。

### Lines 63-65
```mlir
63| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 8], warpsPerCTA = [1, 1], order = [1, 0]}>
64| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
65| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 66-66
```mlir
66| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 67-84
```mlir
67|   tt.func public @buffer_load_to_local(%X: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) attributes {noinline = false} {
68|     %cst = arith.constant dense<64> : tensor<16x64xi32, #blocked>
69|     %c0 = arith.constant 0 : index
70|     %c128 = arith.constant 128 : index
71|     %c1 = arith.constant 1 : index
72| 
73|     %Xoffset_init = arith.constant dense<123> : tensor<16x64xi32, #blocked>
74| 
75|     %x_dummy_buffer = ttg.local_alloc : () -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable, 16x64>
76| 
77|     %for = scf.for %idx = %c0 to %c128 step %c1 iter_args(%Xoffset = %Xoffset_init) -> (tensor<16x64xi32, #blocked>) {
78|       %x = amdg.buffer_load_to_local %X[%Xoffset] into %x_dummy_buffer : <f16>[tensor<16x64xi32, #blocked>] -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable, 16x64>
79| 
80|       %Xoffset_next = arith.addi %Xoffset, %cst : tensor<16x64xi32, #blocked>
81|       scf.yield %Xoffset_next : tensor<16x64xi32, #blocked>
82|     }
83|     tt.return
84|   }
```
**EN:** This function-oriented block defines or enters `buffer_load_to_local`. Within it, the test exercises constants, tt.func, shared/local memory allocation, structured loops, amdg.buffer_load_to_local, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `buffer_load_to_local` 为核心。测试在其中演示 常量、tt.func、共享/本地内存分配、结构化循环、amdg.buffer_load_to_local，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 85-86
```mlir
85| }
86| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 87-87
```mlir
87| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 88-95
```mlir
88| 
89| // CHECK-LABEL: add_before_load
90| // CHECK-DAG: [[X_OFFSET_CST:%.*]] = arith.constant dense<123>
91| // CHECK: scf.for {{.*}} iter_args({{.*}}, [[X_BASE:%.*]] = {{.*}})
92| // CHECK:   [[NEXT_X_BASE:%.*]] = tt.addptr [[X_BASE]], %c64_i32
93| // CHECK:   amdg.buffer_load [[NEXT_X_BASE]]{{\[}}[[X_OFFSET_CST]]{{\]}}
94| // CHECK:   scf.yield {{.*}}, [[NEXT_X_BASE]]
95| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: add_before_load anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: add_before_load 这样的标签用于锚定匹配范围。

### Lines 96-98
```mlir
96| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 8], warpsPerCTA = [1, 1], order = [1, 0]}>
97| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
98| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 99-99
```mlir
99| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 100-116
```mlir
100|   tt.func public @add_before_load(%X: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %Y: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) attributes {noinline = false} {
101|     %cst = arith.constant dense<64> : tensor<16x64xi32, #blocked>
102|     %c0 = arith.constant 0 : index
103|     %c128 = arith.constant 128 : index
104|     %c1 = arith.constant 1 : index
105|     %Xoffset_init = arith.constant dense<123> : tensor<16x64xi32, #blocked>
106|     %x_dummy_buffer = ttg.local_alloc : () -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable, 16x64>
107|     %for = scf.for %idx = %c0 to %c128 step %c1 iter_args(%Xoffset = %Xoffset_init) -> (tensor<16x64xi32, #blocked>) {
108|       %Xoffset_next = arith.addi %Xoffset, %cst : tensor<16x64xi32, #blocked>
109|       %x = amdg.buffer_load %X[%Xoffset_next] : tensor<16x64xf16, #blocked>
110|       ttg.local_store %x, %x_dummy_buffer : tensor<16x64xf16, #blocked> -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable, 16x64>
111|       scf.yield %Xoffset_next : tensor<16x64xi32, #blocked>
112|     }
113|     tt.return
114|   }
115| }
116| 
```
**EN:** This function-oriented block defines or enters `add_before_load`. Within it, the test exercises constants, tt.func, shared/local memory allocation, structured loops, integer additions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `add_before_load` 为核心。测试在其中演示 常量、tt.func、共享/本地内存分配、结构化循环、整数加法，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 117-117
```mlir
117| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 118-126
```mlir
118| 
119| // CHECK-LABEL: isolated_pattern_nested_loop1
120| // CHECK: [[X_OFFSET_CST:%.*]] = arith.constant dense<123>
121| // CHECK: scf.for
122| // CHECK:   scf.for {{.*}} iter_args({{.*}}, [[X_BASE:%.*]] = {{.*}})
123| // CHECK:     amdg.buffer_load [[X_BASE]]{{\[}}[[X_OFFSET_CST]]{{\]}}
124| // CHECK:     [[NEXT_X_BASE:%.*]] = tt.addptr [[X_BASE]]
125| // CHECK:     scf.yield {{.*}}, [[NEXT_X_BASE]]
126| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: isolated_pattern_nested_loop1 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: isolated_pattern_nested_loop1 这样的标签用于锚定匹配范围。

### Lines 127-129
```mlir
127| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 8], warpsPerCTA = [1, 1], order = [1, 0]}>
128| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
129| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 130-130
```mlir
130| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 131-148
```mlir
131|   tt.func public @isolated_pattern_nested_loop1(%X: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) attributes {noinline = false} {
132|     %cst = arith.constant dense<64> : tensor<16x64xi32, #blocked>
133|     %c0 = arith.constant 0 : index
134|     %c32 = arith.constant 32 : index
135|     %c1 = arith.constant 1 : index
136| 
137|     %Xoffset_init = arith.constant dense<123> : tensor<16x64xi32, #blocked>
138|     %x_dummy_buffer = ttg.local_alloc : () -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable, 16x64>
139|     scf.for %idx_outer = %c0 to %c32 step %c1 iter_args() -> () {
140|       %for_inner = scf.for %idx_innter = %c0 to %c32 step %c1 iter_args(%Xoffset = %Xoffset_init) -> (tensor<16x64xi32, #blocked>) {
141|         %x = amdg.buffer_load %X[%Xoffset] : tensor<16x64xf16, #blocked>
142|         ttg.local_store %x, %x_dummy_buffer : tensor<16x64xf16, #blocked> -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable, 16x64>
143|         %Xoffset_next = arith.addi %Xoffset, %cst : tensor<16x64xi32, #blocked>
144|         scf.yield %Xoffset_next : tensor<16x64xi32, #blocked>
145|       }
146|       scf.yield
147|     }
148|     tt.return
```
**EN:** This function-oriented block defines or enters `isolated_pattern_nested_loop1`. Within it, the test exercises constants, tt.func, structured loops, loop/if yielded values, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `isolated_pattern_nested_loop1` 为核心。测试在其中演示 常量、tt.func、结构化循环、循环/分支产出值、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 149-151
```mlir
149|   }
150| }
151| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 152-152
```mlir
152| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 153-161
```mlir
153| 
154| // CHECK-LABEL: isolated_pattern_nested_loop2
155| // CHECK: [[X_OFFSET_CST:%.*]] = arith.constant dense<123>
156| // CHECK: scf.for {{.*}} iter_args({{.*}}, [[X_BASE:%.*]] = {{.*}})
157| // CHECK:   scf.for
158| // CHECK:     amdg.buffer_load [[X_BASE]]{{\[}}[[X_OFFSET_CST]]{{\]}}
159| // CHECK:   [[NEXT_X_BASE:%.*]] = tt.addptr [[X_BASE]]
160| // CHECK:   scf.yield {{.*}}, [[NEXT_X_BASE]]
161| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: isolated_pattern_nested_loop2 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: isolated_pattern_nested_loop2 这样的标签用于锚定匹配范围。

### Lines 162-164
```mlir
162| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 8], warpsPerCTA = [1, 1], order = [1, 0]}>
163| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
164| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 165-165
```mlir
165| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 166-183
```mlir
166|   tt.func public @isolated_pattern_nested_loop2(%X: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) attributes {noinline = false} {
167|     %cst = arith.constant dense<64> : tensor<16x64xi32, #blocked>
168|     %c0 = arith.constant 0 : index
169|     %c128 = arith.constant 128 : index
170|     %c1 = arith.constant 1 : index
171| 
172|     %Xoffset_init = arith.constant dense<123> : tensor<16x64xi32, #blocked>
173|     %x_dummy_buffer = ttg.local_alloc : () -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable, 16x64>
174|     %for_outer = scf.for %idx_outer = %c0 to %c128 step %c1 iter_args(%Xoffset = %Xoffset_init) -> (tensor<16x64xi32, #blocked>) {
175|       scf.for %idx_inner = %c0 to %c128 step %c1 iter_args() -> () {
176|         %x = amdg.buffer_load %X[%Xoffset] : tensor<16x64xf16, #blocked>
177|         ttg.local_store %x, %x_dummy_buffer : tensor<16x64xf16, #blocked> -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable, 16x64>
178|         scf.yield
179|       }
180|       %Xoffset_next = arith.addi %Xoffset, %cst : tensor<16x64xi32, #blocked>
181|       scf.yield %Xoffset_next : tensor<16x64xi32, #blocked>
182|     }
183|     tt.return
```
**EN:** This function-oriented block defines or enters `isolated_pattern_nested_loop2`. Within it, the test exercises constants, tt.func, structured loops, loop/if yielded values, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `isolated_pattern_nested_loop2` 为核心。测试在其中演示 常量、tt.func、结构化循环、循环/分支产出值、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 184-186
```mlir
184|   }
185| }
186| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 187-187
```mlir
187| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 188-195
```mlir
188| 
189| // CHECK-LABEL: convert_with_base_ptr_optimization
190| // CHECK: [[X_OFFSET_CST:%.*]] = arith.constant dense<123>
191| // CHECK: scf.for {{.*}} iter_args({{.*}}, [[X_BASE:%.*]] = {{.*}})
192| // CHECK:   amdg.buffer_load [[X_BASE]]{{\[}}[[X_OFFSET_CST]]{{\]}}
193| // CHECK:   [[NEXT_X_BASE:%.*]] = tt.addptr [[X_BASE]]
194| // CHECK:   scf.yield {{.*}}, [[NEXT_X_BASE]]
195| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: convert_with_base_ptr_optimization anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: convert_with_base_ptr_optimization 这样的标签用于锚定匹配范围。

### Lines 196-198
```mlir
196| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 8], warpsPerCTA = [1, 1], order = [1, 0]}>
197| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
198| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 199-199
```mlir
199| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 200-217
```mlir
200|   tt.func public @convert_with_base_ptr_optimization(%X: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) attributes {noinline = false} {
201|     %step = arith.constant dense<64> : tensor<16x64xi32, #blocked>
202|     %c0 = arith.constant 0 : index
203|     %c128 = arith.constant 128 : index
204|     %c1 = arith.constant 1 : index
205|     %x_base = tt.splat %X : !tt.ptr<f16> -> tensor<16x64x!tt.ptr<f16>, #blocked>
206|     %offsets_init = arith.constant dense<123> : tensor<16x64xi32, #blocked>
207|     %x_dummy_buffer = ttg.local_alloc : () -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable, 16x64>
208|     %for = scf.for %idx_outer = %c0 to %c128 step %c1 iter_args(%offsets = %offsets_init) -> (tensor<16x64xi32, #blocked>) {
209|       %X_ptr = tt.addptr %x_base, %offsets : tensor<16x64x!tt.ptr<f16>, #blocked>, tensor<16x64xi32, #blocked>
210|       %x = tt.load %X_ptr : tensor<16x64x!tt.ptr<f16>, #blocked>
211|       ttg.local_store %x, %x_dummy_buffer : tensor<16x64xf16, #blocked> -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable, 16x64>
212|       %offsets_next = arith.addi %offsets, %step : tensor<16x64xi32, #blocked>
213|       scf.yield %offsets_next : tensor<16x64xi32, #blocked>
214|     }
215|     tt.return
216|   }
217| }
```
**EN:** This function-oriented block defines or enters `convert_with_base_ptr_optimization`. Within it, the test exercises constants, tt.func, broadcasted scalars or pointers, shared/local memory allocation, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `convert_with_base_ptr_optimization` 为核心。测试在其中演示 常量、tt.func、广播后的标量或指针、共享/本地内存分配、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 219-220
```mlir
219| 
220| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 220-225
```mlir
220| 
221| // CHECK-LABEL: dynamic_base_negative
222| // CHECK:   [[X_BASE:%.*]] = tt.addptr
223| // CHECK:   amdg.buffer_load [[X_BASE]]
224| // CHECK-NOT: tt.addptr
225| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: dynamic_base_negative anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: dynamic_base_negative 这样的标签用于锚定匹配范围。

### Lines 226-228
```mlir
226| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 8], warpsPerCTA = [1, 1], order = [1, 0]}>
227| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
228| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 229-229
```mlir
229| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 230-247
```mlir
230|   tt.func public @dynamic_base_negative(%X: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) attributes {noinline = false} {
231|     %cst = arith.constant dense<64> : tensor<16x64xi32, #blocked>
232|     %c0 = arith.constant 0 : i32
233|     %c128 = arith.constant 128 : i32
234|     %c1 = arith.constant 1 : i32
235| 
236|     %Xoffset_init = arith.constant dense<123> : tensor<16x64xi32, #blocked>
237|     %x_dummy_buffer = ttg.local_alloc : () -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable, 16x64>
238|     %for = scf.for %idx = %c0 to %c128 step %c1 iter_args(%Xoffset = %Xoffset_init) -> (tensor<16x64xi32, #blocked>) : i32 {
239|       %x_base = tt.addptr %X, %idx : !tt.ptr<f16>, i32
240|       %x = amdg.buffer_load %x_base[%Xoffset] : tensor<16x64xf16, #blocked>
241|       ttg.local_store %x, %x_dummy_buffer : tensor<16x64xf16, #blocked> -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable, 16x64>
242|       %Xoffset_next = arith.addi %Xoffset, %cst : tensor<16x64xi32, #blocked>
243|       scf.yield %Xoffset_next : tensor<16x64xi32, #blocked>
244|     }
245|     tt.return
246|   }
247| }
```
**EN:** This function-oriented block defines or enters `dynamic_base_negative`. Within it, the test exercises constants, tt.func, shared/local memory allocation, structured loops, pointer arithmetic, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `dynamic_base_negative` 为核心。测试在其中演示 常量、tt.func、共享/本地内存分配、结构化循环、指针算术，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 249-250
```mlir
249| 
250| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 250-253
```mlir
250| 
251| // CHECK-LABEL: non_uniform_step_negative
252| // CHECK-NOT: tt.addptr
253| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: non_uniform_step_negative anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: non_uniform_step_negative 这样的标签用于锚定匹配范围。

### Lines 254-256
```mlir
254| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 8], warpsPerCTA = [1, 1], order = [1, 0]}>
255| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
256| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 257-257
```mlir
257| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 258-274
```mlir
258|   tt.func public @non_uniform_step_negative(%X: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %step : tensor<16x64xi32, #blocked>) attributes {noinline = false} {
259|     %c0 = arith.constant 0 : i32
260|     %c128 = arith.constant 128 : i32
261|     %c1 = arith.constant 1 : i32
262| 
263|     %Xoffset_init = arith.constant dense<123> : tensor<16x64xi32, #blocked>
264|     %x_dummy_buffer = ttg.local_alloc : () -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable, 16x64>
265|     %for = scf.for %idx = %c0 to %c128 step %c1 iter_args(%Xoffset = %Xoffset_init) -> (tensor<16x64xi32, #blocked>) : i32 {
266|       %x = amdg.buffer_load %X[%Xoffset] : tensor<16x64xf16, #blocked>
267|       ttg.local_store %x, %x_dummy_buffer : tensor<16x64xf16, #blocked> -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable, 16x64>
268|       %Xoffset_next = arith.addi %Xoffset, %step : tensor<16x64xi32, #blocked>
269|       scf.yield %Xoffset_next : tensor<16x64xi32, #blocked>
270|     }
271|     tt.return
272|   }
273| }
274| 
```
**EN:** This function-oriented block defines or enters `non_uniform_step_negative`. Within it, the test exercises constants, tt.func, shared/local memory allocation, structured loops, amdg.buffer_load, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `non_uniform_step_negative` 为核心。测试在其中演示 常量、tt.func、共享/本地内存分配、结构化循环、amdg.buffer_load，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 275-275
```mlir
275| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 276-279
```mlir
276| 
277| // CHECK-LABEL: offsets_possible_overflow_negative
278| // CHECK-NOT: tt.addptr
279| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: offsets_possible_overflow_negative anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: offsets_possible_overflow_negative 这样的标签用于锚定匹配范围。

### Lines 280-282
```mlir
280| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 8], warpsPerCTA = [1, 1], order = [1, 0]}>
281| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
282| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 283-283
```mlir
283| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 284-301
```mlir
284|   tt.func public @offsets_possible_overflow_negative(%X: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %step_scalar : i32) attributes {noinline = false} {
285|     %c0 = arith.constant 0 : i32
286|     %c128 = arith.constant 128 : i32
287|     %c1 = arith.constant 1 : i32
288| 
289|     %Xoffset_init = arith.constant dense<123> : tensor<16x64xi32, #blocked>
290|     %x_dummy_buffer = ttg.local_alloc : () -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable, 16x64>
291|     %step = tt.splat %step_scalar: i32 -> tensor<16x64xi32, #blocked>
292|     %for = scf.for %idx = %c0 to %c128 step %c1 iter_args(%Xoffset = %Xoffset_init) -> (tensor<16x64xi32, #blocked>) : i32 {
293|       %x = amdg.buffer_load %X[%Xoffset] : tensor<16x64xf16, #blocked>
294|       ttg.local_store %x, %x_dummy_buffer : tensor<16x64xf16, #blocked> -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable, 16x64>
295|       %Xoffset_next = arith.addi %Xoffset, %step : tensor<16x64xi32, #blocked>
296|       scf.yield %Xoffset_next : tensor<16x64xi32, #blocked>
297|     }
298|     tt.return
299|   }
300| }
301| 
```
**EN:** This function-oriented block defines or enters `offsets_possible_overflow_negative`. Within it, the test exercises constants, tt.func, shared/local memory allocation, broadcasted scalars or pointers, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `offsets_possible_overflow_negative` 为核心。测试在其中演示 常量、tt.func、共享/本地内存分配、广播后的标量或指针、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 302-302
```mlir
302| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 303-306
```mlir
303| 
304| // CHECK-LABEL: two_parallel_addi_negative
305| // CHECK-NOT: tt.addptr
306| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: two_parallel_addi_negative anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: two_parallel_addi_negative 这样的标签用于锚定匹配范围。

### Lines 307-309
```mlir
307| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 8], warpsPerCTA = [1, 1], order = [1, 0]}>
308| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
309| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 310-310
```mlir
310| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 311-328
```mlir
311|   tt.func public @two_parallel_addi_negative(%X: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) attributes {noinline = false} {
312|     %c0 = arith.constant 0 : i32
313|     %c128 = arith.constant 128 : i32
314|     %c1 = arith.constant 1 : i32
315| 
316|     %Xoffset_init = arith.constant dense<123> : tensor<16x64xi32, #blocked>
317|     %x_dummy_buffer = ttg.local_alloc : () -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable, 16x64>
318|     %step = arith.constant dense<64> : tensor<16x64xi32, #blocked>
319|     %for:2 = scf.for %idx = %c0 to %c128 step %c1 iter_args(%Xoffset = %Xoffset_init, %Xoffset_dummy = %Xoffset_init) -> (tensor<16x64xi32, #blocked>, tensor<16x64xi32, #blocked>) : i32 {
320|       %Xoffset_decoy = arith.addi %Xoffset, %step : tensor<16x64xi32, #blocked>
321|       %x = amdg.buffer_load %X[%Xoffset_decoy] : tensor<16x64xf16, #blocked>
322|       ttg.local_store %x, %x_dummy_buffer : tensor<16x64xf16, #blocked> -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable, 16x64>
323|       %Xoffset_next = arith.addi %Xoffset, %step : tensor<16x64xi32, #blocked>
324|       scf.yield %Xoffset_next, %Xoffset_decoy : tensor<16x64xi32, #blocked>, tensor<16x64xi32, #blocked>
325|     }
326|     tt.return
327|   }
328| }
```
**EN:** This function-oriented block defines or enters `two_parallel_addi_negative`. Within it, the test exercises constants, tt.func, integer additions, shared/local memory allocation, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `two_parallel_addi_negative` 为核心。测试在其中演示 常量、tt.func、整数加法、共享/本地内存分配、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 330-331
```mlir
330| 
331| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 331-332
```mlir
331| 
332| // Check case with three buffer ops in a loop, first and third are optimized, second is rejected because step is not uniform
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 333-341
```mlir
333| // CHECK-LABEL: mixed_optimized_rejected_optimized
334| // CHECK-DAG: [[X_OFFSET:%.*]] = arith.constant dense<123> : tensor<16x64xi32, #blocked>
335| // CHECK-DAG: [[Y_OFFSET_INIT:%.*]] = arith.constant dense<456> : tensor<64x32xi32, #blocked>
336| // CHECK-DAG: [[Z_OFFSET:%.*]] = arith.constant dense<789> : tensor<32x32xi32, #blocked>
337| // CHECK: scf.for {{.*}} iter_args({{%.*}}[[Y_OFFSET:%.*]] = [[Y_OFFSET_INIT]]
338| // CHECK-DAG: amdg.buffer_load {{%.*\[}}[[X_OFFSET]]{{\]}} : tensor<16x64xf16, #blocked>
339| // CHECK-DAG: amdg.buffer_load {{%.*\[}}[[Y_OFFSET]]{{\]}} cacheModifier = cg : tensor<64x32xf16, #blocked>
340| // CHECK-DAG: amdg.buffer_load {{%.*\[}}[[Z_OFFSET]]{{\]}} cacheModifier = cg : tensor<32x32xf16, #blocked>
341| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: mixed_optimized_rejected_optimized anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: mixed_optimized_rejected_optimized 这样的标签用于锚定匹配范围。

### Lines 342-344
```mlir
342| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 8], warpsPerCTA = [1, 1], order = [1, 0]}>
343| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
344| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 345-345
```mlir
345| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 346-363
```mlir
346|   tt.func public @mixed_optimized_rejected_optimized(
347|         %X: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
348|         %Y: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
349|         %Z: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}
350|       ) attributes {noinline = false} {
351|     %Xstep = arith.constant dense<64> : tensor<16x64xi32, #blocked>
352|     %Ystep_slice = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
353|     %Ystep_2d = tt.expand_dims %Ystep_slice {axis = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x32xi32, #blocked>
354|     %Ystep = tt.broadcast %Ystep_2d : tensor<1x32xi32, #blocked> -> tensor<64x32xi32, #blocked>
355|     %Zstep = arith.constant dense<256> : tensor<32x32xi32, #blocked>
356|     %iter_first = arith.constant 0 : index
357|     %iter_last = arith.constant 128 : index
358|     %iter_step = arith.constant 1 : index
359| 
360|     %Xoffset_init = arith.constant dense<123> : tensor<16x64xi32, #blocked>
361|     %Yoffset_init = arith.constant dense<456> : tensor<64x32xi32, #blocked>
362|     %Zoffset_init = arith.constant dense<789> : tensor<32x32xi32, #blocked>
363| 
```
**EN:** This function-oriented block defines or enters `mixed_optimized_rejected_optimized`. Within it, the test exercises constants, tt.func, lane/block index ranges, shape expansion, tensor broadcasting, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mixed_optimized_rejected_optimized` 为核心。测试在其中演示 常量、tt.func、lane/block 索引范围、形状扩展、张量广播，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 364-381
```mlir
364|     %x_dummy_buffer = ttg.local_alloc : () -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable, 16x64>
365|     %y_dummy_buffer = ttg.local_alloc : () -> !ttg.memdesc<64x32xf16, #shared, #smem, mutable, 64x32>
366|     %z_dummy_buffer = ttg.local_alloc : () -> !ttg.memdesc<32x32xf16, #shared, #smem, mutable, 32x32>
367| 
368|     %for:3 = scf.for %iter = %iter_first to %iter_last step %iter_step iter_args(%Xoffset = %Xoffset_init, %Yoffset = %Yoffset_init, %Zoffset = %Zoffset_init) -> (tensor<16x64xi32, #blocked>, tensor<64x32xi32, #blocked>, tensor<32x32xi32, #blocked>) {
369|       %x = amdg.buffer_load %X[%Xoffset] : tensor<16x64xf16, #blocked>
370|       %y = amdg.buffer_load %Y[%Yoffset] cacheModifier = cg : tensor<64x32xf16, #blocked>
371|       %z = amdg.buffer_load %Z[%Zoffset] cacheModifier = cg : tensor<32x32xf16, #blocked>
372| 
373|       ttg.local_store %x, %x_dummy_buffer : tensor<16x64xf16, #blocked> -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable, 16x64>
374|       ttg.local_store %y, %y_dummy_buffer : tensor<64x32xf16, #blocked> -> !ttg.memdesc<64x32xf16, #shared, #smem, mutable, 64x32>
375|       ttg.local_store %z, %z_dummy_buffer : tensor<32x32xf16, #blocked> -> !ttg.memdesc<32x32xf16, #shared, #smem, mutable, 32x32>
376| 
377|       %Xoffset_next = arith.addi %Xoffset, %Xstep : tensor<16x64xi32, #blocked>
378|       %Yoffset_next = arith.addi %Yoffset, %Ystep : tensor<64x32xi32, #blocked>
379|       %Zoffset_next = arith.addi %Zoffset, %Zstep : tensor<32x32xi32, #blocked>
380|       scf.yield %Xoffset_next, %Yoffset_next, %Zoffset_next : tensor<16x64xi32, #blocked>, tensor<64x32xi32, #blocked>, tensor<32x32xi32, #blocked>
381|     }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining shared/local memory allocation, amdg.buffer_load, local/shared memory stores, integer additions, structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 共享/本地内存分配、amdg.buffer_load、本地/共享内存存储、整数加法、结构化循环。

### Lines 382-385
```mlir
382|     tt.return
383|   }
384| }
385| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 386-386
```mlir
386| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 387-388
```mlir
387| 
388| // Check case with three buffer ops in a loop, only second one is optimized
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 389-397
```mlir
389| // CHECK-LABEL: mixed_rejected_optimized_rejected
390| // CHECK-DAG: [[X_OFFSET_INIT:%.*]] = arith.constant dense<123> : tensor<16x64xi32, #blocked>
391| // CHECK-DAG: [[Y_OFFSET:%.*]] = arith.constant dense<456> : tensor<64x32xi32, #blocked>
392| // CHECK-DAG: [[Z_OFFSET_INIT:%.*]] = arith.constant dense<789> : tensor<32x32xi32, #blocked>
393| // CHECK: scf.for {{.*}} iter_args([[X_OFFSET:%.*]] = [[X_OFFSET_INIT]], {{.*}}[[Z_OFFSET:%.*]] = [[Z_OFFSET_INIT]]
394| // CHECK-DAG: amdg.buffer_load {{%.*\[}}[[X_OFFSET]]{{\]}} : tensor<16x64xf16, #blocked>
395| // CHECK-DAG: amdg.buffer_load {{%.*\[}}[[Y_OFFSET]]{{\]}} cacheModifier = cg : tensor<64x32xf16, #blocked>
396| // CHECK-DAG: amdg.buffer_load {{%.*\[}}[[Z_OFFSET]]{{\]}} cacheModifier = cg : tensor<32x32xf16, #blocked>
397| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: mixed_rejected_optimized_rejected anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: mixed_rejected_optimized_rejected 这样的标签用于锚定匹配范围。

### Lines 398-400
```mlir
398| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 8], warpsPerCTA = [1, 1], order = [1, 0]}>
399| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
400| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 401-401
```mlir
401| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 402-419
```mlir
402|   tt.func public @mixed_rejected_optimized_rejected(
403|         %X: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
404|         %Y: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
405|         %Z: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
406|         %Xstride: i32,
407|         %Zstride: i32
408|       ) attributes {noinline = false} {
409|     %Xstep = tt.splat %Xstride: i32 -> tensor<16x64xi32, #blocked>
410|     %Ystep = arith.constant dense<128> : tensor<64x32xi32, #blocked>
411|     %Zstep = tt.splat %Zstride : i32 -> tensor<32x32xi32, #blocked>
412|     %iter_first = arith.constant 0 : index
413|     %iter_last = arith.constant 128 : index
414|     %iter_step = arith.constant 1 : index
415| 
416|     %Xoffset_init = arith.constant dense<123> : tensor<16x64xi32, #blocked>
417|     %Yoffset_init = arith.constant dense<456> : tensor<64x32xi32, #blocked>
418|     %Zoffset_init = arith.constant dense<789> : tensor<32x32xi32, #blocked>
419| 
```
**EN:** This function-oriented block defines or enters `mixed_rejected_optimized_rejected`. Within it, the test exercises constants, tt.func, broadcasted scalars or pointers, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mixed_rejected_optimized_rejected` 为核心。测试在其中演示 常量、tt.func、广播后的标量或指针，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 420-437
```mlir
420|     %x_dummy_buffer = ttg.local_alloc : () -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable, 16x64>
421|     %y_dummy_buffer = ttg.local_alloc : () -> !ttg.memdesc<64x32xf16, #shared, #smem, mutable, 64x32>
422|     %z_dummy_buffer = ttg.local_alloc : () -> !ttg.memdesc<32x32xf16, #shared, #smem, mutable, 32x32>
423| 
424|     %for:3 = scf.for %iter = %iter_first to %iter_last step %iter_step iter_args(%Xoffset = %Xoffset_init, %Yoffset = %Yoffset_init, %Zoffset = %Zoffset_init) -> (tensor<16x64xi32, #blocked>, tensor<64x32xi32, #blocked>, tensor<32x32xi32, #blocked>) {
425|       %x = amdg.buffer_load %X[%Xoffset] : tensor<16x64xf16, #blocked>
426|       %y = amdg.buffer_load %Y[%Yoffset] cacheModifier = cg : tensor<64x32xf16, #blocked>
427|       %z = amdg.buffer_load %Z[%Zoffset] cacheModifier = cg : tensor<32x32xf16, #blocked>
428| 
429|       ttg.local_store %x, %x_dummy_buffer : tensor<16x64xf16, #blocked> -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable, 16x64>
430|       ttg.local_store %y, %y_dummy_buffer : tensor<64x32xf16, #blocked> -> !ttg.memdesc<64x32xf16, #shared, #smem, mutable, 64x32>
431|       ttg.local_store %z, %z_dummy_buffer : tensor<32x32xf16, #blocked> -> !ttg.memdesc<32x32xf16, #shared, #smem, mutable, 32x32>
432| 
433|       %Xoffset_next = arith.addi %Xoffset, %Xstep : tensor<16x64xi32, #blocked>
434|       %Yoffset_next = arith.addi %Yoffset, %Ystep : tensor<64x32xi32, #blocked>
435|       %Zoffset_next = arith.addi %Zoffset, %Zstep : tensor<32x32xi32, #blocked>
436|       scf.yield %Xoffset_next, %Yoffset_next, %Zoffset_next : tensor<16x64xi32, #blocked>, tensor<64x32xi32, #blocked>, tensor<32x32xi32, #blocked>
437|     }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining shared/local memory allocation, amdg.buffer_load, local/shared memory stores, integer additions, structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 共享/本地内存分配、amdg.buffer_load、本地/共享内存存储、整数加法、结构化循环。

### Lines 438-441
```mlir
438|     tt.return
439|   }
440| }
441| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 442-442
```mlir
442| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 443-446
```mlir
443| 
444| // CHECK-LABEL: multiple_offset_uses
445| // CHECK: tt.addptr
446| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: multiple_offset_uses anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: multiple_offset_uses 这样的标签用于锚定匹配范围。

### Lines 447-449
```mlir
447| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 8], warpsPerCTA = [1, 1], order = [1, 0]}>
448| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
449| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 450-450
```mlir
450| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 451-468
```mlir
451|   tt.func public @multiple_offset_uses(%X: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %Y: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) attributes {noinline = false} {
452|     %offset_step = arith.constant dense<64> : tensor<16x64xi32, #blocked>
453|     %iter_first = arith.constant 0 : index
454|     %iter_last = arith.constant 128 : index
455|     %iter_step = arith.constant 1 : index
456|     %Xoffset_init = arith.constant dense<123> : tensor<16x64xi32, #blocked>
457|     %x_dummy_buffer = ttg.local_alloc : () -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable, 16x64>
458|     %offset_buffer = ttg.local_alloc : () -> !ttg.memdesc<16x64xi32, #shared, #smem, mutable, 16x64>
459|     %for = scf.for %idx = %iter_first to %iter_last step %iter_step iter_args(%Xoffset = %Xoffset_init) -> (tensor<16x64xi32, #blocked>) {
460|       ttg.local_store %Xoffset, %offset_buffer : tensor<16x64xi32, #blocked> -> !ttg.memdesc<16x64xi32, #shared, #smem, mutable, 16x64>
461|       %Xoffset_next = arith.addi %Xoffset, %offset_step : tensor<16x64xi32, #blocked>
462|       %x = amdg.buffer_load %X[%Xoffset_next] : tensor<16x64xf16, #blocked>
463|       ttg.local_store %x, %x_dummy_buffer : tensor<16x64xf16, #blocked> -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable, 16x64>
464|       ttg.local_store %Xoffset_next, %offset_buffer : tensor<16x64xi32, #blocked> -> !ttg.memdesc<16x64xi32, #shared, #smem, mutable, 16x64>
465|       scf.yield %Xoffset_next : tensor<16x64xi32, #blocked>
466|     }
467|     ttg.local_store %for, %offset_buffer : tensor<16x64xi32, #blocked> -> !ttg.memdesc<16x64xi32, #shared, #smem, mutable, 16x64>
468|     tt.return
```
**EN:** This function-oriented block defines or enters `multiple_offset_uses`. Within it, the test exercises constants, local/shared memory stores, tt.func, shared/local memory allocation, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `multiple_offset_uses` 为核心。测试在其中演示 常量、本地/共享内存存储、tt.func、共享/本地内存分配、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 469-471
```mlir
469|   }
470| }
471| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 472-472
```mlir
472| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 473-479
```mlir
473| 
474| // CHECK-LABEL: multiple_offset_uses_over_multiple_loops
475| // CHECK: scf.for
476| // CHECK:   tt.addptr
477| // CHECK:   scf.for
478| // CHECK:     tt.addptr
479| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: multiple_offset_uses_over_multiple_loops anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: multiple_offset_uses_over_multiple_loops 这样的标签用于锚定匹配范围。

### Lines 480-482
```mlir
480| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 8], warpsPerCTA = [1, 1], order = [1, 0]}>
481| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
482| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 483-483
```mlir
483| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 484-501
```mlir
484|   tt.func public @multiple_offset_uses_over_multiple_loops(%X: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %Y: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) attributes {noinline = false} {
485|     %offset_step = arith.constant dense<64> : tensor<16x64xi32, #blocked>
486|     %iter_first = arith.constant 0 : index
487|     %iter_last = arith.constant 32 : index
488|     %iter_step = arith.constant 1 : index
489|     %Xoffset_init = arith.constant dense<123> : tensor<16x64xi32, #blocked>
490|     %x_dummy_buffer = ttg.local_alloc : () -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable, 16x64>
491|     %offset_buffer = ttg.local_alloc : () -> !ttg.memdesc<16x64xi32, #shared, #smem, mutable, 16x64>
492|     %for1 = scf.for %idx = %iter_first to %iter_last step %iter_step iter_args(%Xoffset1 = %Xoffset_init) -> (tensor<16x64xi32, #blocked>) {
493|       %Xoffset_next1 = arith.addi %Xoffset1, %offset_step : tensor<16x64xi32, #blocked>
494| 
495|       %for2 = scf.for %idx2 = %iter_first to %iter_last step %iter_step iter_args(%Xoffset2 = %Xoffset_next1) -> (tensor<16x64xi32, #blocked>) {
496|         %Xoffset_next2 = arith.addi %Xoffset2, %offset_step : tensor<16x64xi32, #blocked>
497|         %x2 = amdg.buffer_load %X[%Xoffset_next2] : tensor<16x64xf16, #blocked>
498|         ttg.local_store %x2, %x_dummy_buffer : tensor<16x64xf16, #blocked> -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable, 16x64>
499|         scf.yield %Xoffset_next2 : tensor<16x64xi32, #blocked>
500|       }
501| 
```
**EN:** This function-oriented block defines or enters `multiple_offset_uses_over_multiple_loops`. Within it, the test exercises constants, tt.func, shared/local memory allocation, structured loops, integer additions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `multiple_offset_uses_over_multiple_loops` 为核心。测试在其中演示 常量、tt.func、共享/本地内存分配、结构化循环、整数加法，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 502-509
```mlir
502|       %x1 = amdg.buffer_load %X[%Xoffset_next1] : tensor<16x64xf16, #blocked>
503|       ttg.local_store %x1, %x_dummy_buffer : tensor<16x64xf16, #blocked> -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable, 16x64>
504|       scf.yield %Xoffset_next1 : tensor<16x64xi32, #blocked>
505|     }
506|     tt.return
507|   }
508| }
509| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining amdg.buffer_load, local/shared memory stores, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 amdg.buffer_load、本地/共享内存存储、循环/分支产出值、tt.return。

### Lines 510-510
```mlir
510| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 511-514
```mlir
511| 
512| // CHECK-LABEL: multiple_addi_in_sequence_negative
513| // CHECK-NOT:  tt.addptr
514| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: multiple_addi_in_sequence_negative anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: multiple_addi_in_sequence_negative 这样的标签用于锚定匹配范围。

### Lines 515-517
```mlir
515| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 8], warpsPerCTA = [1, 1], order = [1, 0]}>
516| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
517| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 518-518
```mlir
518| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 519-536
```mlir
519|   tt.func public @multiple_addi_in_sequence_negative(%X: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) attributes {noinline = false} {
520|     %cst1 = arith.constant dense<64> : tensor<16x64xi32, #blocked>
521|     %cst2 = arith.constant dense<128> : tensor<16x64xi32, #blocked>
522|     %iter_first = arith.constant 0 : index
523|     %iter_last = arith.constant 128 : index
524|     %iter_step = arith.constant 1 : index
525| 
526|     %Xoffset_init = arith.constant dense<123> : tensor<16x64xi32, #blocked>
527| 
528|     %x_dummy_buffer = ttg.local_alloc : () -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable, 16x64>
529| 
530|     %for = scf.for %idx = %iter_first to %iter_last step %iter_step iter_args(%Xoffset = %Xoffset_init) -> (tensor<16x64xi32, #blocked>) {
531|       %x = amdg.buffer_load_to_local %X[%Xoffset] into %x_dummy_buffer : <f16>[tensor<16x64xi32, #blocked>] -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable, 16x64>
532| 
533|       %Xoffset_tmp = arith.addi %Xoffset, %cst1 : tensor<16x64xi32, #blocked>
534|       %Xoffset_next = arith.addi %Xoffset_tmp, %cst2 : tensor<16x64xi32, #blocked>
535|       scf.yield %Xoffset_next : tensor<16x64xi32, #blocked>
536|     }
```
**EN:** This function-oriented block defines or enters `multiple_addi_in_sequence_negative`. Within it, the test exercises constants, tt.func, integer additions, shared/local memory allocation, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `multiple_addi_in_sequence_negative` 为核心。测试在其中演示 常量、tt.func、整数加法、共享/本地内存分配、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 537-540
```mlir
537|     tt.return
538|   }
539| }
540| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 541-541
```mlir
541| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 542-545
```mlir
542| 
543| // CHECK-LABEL: multiple_buffer_loads_on_one_addi
544| // CHECK-COUNT-2: tt.addptr
545| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: multiple_buffer_loads_on_one_addi anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: multiple_buffer_loads_on_one_addi 这样的标签用于锚定匹配范围。

### Lines 546-548
```mlir
546| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 8], warpsPerCTA = [1, 1], order = [1, 0]}>
547| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
548| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 549-549
```mlir
549| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 550-567
```mlir
550|   tt.func public @multiple_buffer_loads_on_one_addi(%X: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) attributes {noinline = false} {
551|     %cst = arith.constant dense<64> : tensor<16x64xi32, #blocked>
552|     %iter_first = arith.constant 0 : index
553|     %iter_last = arith.constant 128 : index
554|     %iter_step = arith.constant 1 : index
555| 
556|     %Xoffset_init = arith.constant dense<123> : tensor<16x64xi32, #blocked>
557| 
558|     %x_dummy_buffer = ttg.local_alloc : () -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable, 16x64>
559| 
560|     %for = scf.for %idx = %iter_first to %iter_last step %iter_step iter_args(%Xoffset = %Xoffset_init) -> (tensor<16x64xi32, #blocked>) {
561|       %x1 = amdg.buffer_load_to_local %X[%Xoffset] into %x_dummy_buffer : <f16>[tensor<16x64xi32, #blocked>] -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable, 16x64>
562|       %Xoffset_next = arith.addi %Xoffset, %cst : tensor<16x64xi32, #blocked>
563|       %x2 = amdg.buffer_load_to_local %X[%Xoffset_next] into %x_dummy_buffer : <f16>[tensor<16x64xi32, #blocked>] -> !ttg.memdesc<16x64xf16, #shared, #smem, mutable, 16x64>
564|       scf.yield %Xoffset_next : tensor<16x64xi32, #blocked>
565|     }
566|     tt.return
567|   }
```
**EN:** This function-oriented block defines or enters `multiple_buffer_loads_on_one_addi`. Within it, the test exercises constants, tt.func, amdg.buffer_load_to_local, shared/local memory allocation, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `multiple_buffer_loads_on_one_addi` 为核心。测试在其中演示 常量、tt.func、amdg.buffer_load_to_local、共享/本地内存分配、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 568-569
```mlir
568| }
569| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 570-570
```mlir
570| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 571-574
```mlir
571| 
572| // CHECK-LABEL: messed_add_chain_negative
573| // CHECK-NOT:  tt.addptr
574| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: messed_add_chain_negative anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: messed_add_chain_negative 这样的标签用于锚定匹配范围。

### Lines 575-575
```mlir
575| #blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [64], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 576-576
```mlir
576| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 577-589
```mlir
577|   tt.func public @messed_add_chain_negative(%arg0: !tt.ptr<i32> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) -> tensor<256xi32, #blocked> attributes {noinline = false} {
578|     %zero = arith.constant dense<0> : tensor<256xi32, #blocked>
579|     %iter_first = arith.constant 0 : i32
580|     %iter_last = arith.constant 15 : i32
581|     %iter_step = arith.constant 1 : i32
582| 
583|     %for:2 = scf.for %iter = %iter_first to %iter_last step %iter_step iter_args(%arg1 = %zero, %arg2 = %zero) -> (tensor<256xi32, #blocked>, tensor<256xi32, #blocked>) : i32 {
584|       %data = amdg.buffer_load %arg0[%arg1] : tensor<256xi32, #blocked>
585|       scf.yield %arg2, %data : tensor<256xi32, #blocked>, tensor<256xi32, #blocked>
586|     }
587|     tt.return %for#1 : tensor<256xi32, #blocked>
588|   }
589| }
```
**EN:** This function-oriented block defines or enters `messed_add_chain_negative`. Within it, the test exercises constants, tt.func, structured loops, amdg.buffer_load, loop/if yielded values, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `messed_add_chain_negative` 为核心。测试在其中演示 常量、tt.func、结构化循环、amdg.buffer_load、循环/分支产出值，这些操作构成了被优化、lower 或验证的主要数据流。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `--tritonamdgpu-convert-buffer-ops="gfx-arch=gfx950"`, `--tritonamdgpu-optimize-buffer-op-ptr|`, `--check-prefixes=CHECK`
- **CN:** 主要 pass 选项：`-split-input-file`，`--tritonamdgpu-convert-buffer-ops="gfx-arch=gfx950"`，`--tritonamdgpu-optimize-buffer-op-ptr|`，`--check-prefixes=CHECK`
- **EN:** Dominant operations include `arith.constant`, `tt.func`, `arith.addi`, `ttg.local_alloc`, `ttg.local_store`, `scf.for`, `scf.yield`, `amdg.buffer_load`, `module`, `tt.return`.
- **CN:** 主要操作包括 `arith.constant`、`tt.func`、`arith.addi`、`ttg.local_alloc`、`ttg.local_store`、`scf.for`、`scf.yield`、`amdg.buffer_load`、`module`、`tt.return`。
- **EN:** The file contains 16 independently testable section(s). Check styles used: CHECK x41, CHECK-LABEL x17, CHECK-DAG x16, CHECK-NOT x6. Important labels include add_after_load, buffer_load_to_local, add_before_load, isolated_pattern_nested_loop1. It validates both the presence of the desired rewrite and the absence of unwanted residual operations.
- **CN:** 该文件包含 16 个可独立测试的分段。使用的检查类型：CHECK ×41，CHECK-LABEL ×17，CHECK-DAG ×16，CHECK-NOT ×6。 关键标签包括 add_after_load，buffer_load_to_local，add_before_load，isolated_pattern_nested_loop1。 它同时验证目标重写已经出现，并且不再残留不需要的操作。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `ttg`: TritonGPU layout, memory, and GPU-specific operations.
- **CN:** `ttg`：TritonGPU 布局、内存和 GPU 专用操作。
- **EN:** `arith`: MLIR arithmetic dialect for scalar/tensor math.
- **CN:** `arith`：用于标量/张量运算的 MLIR arithmetic 方言。
- **EN:** `tt`: Triton high-level tensor/pointer operations.
- **CN:** `tt`：Triton 高层张量/指针操作。
- **EN:** `scf`: Structured control flow for loops and conditionals.
- **CN:** `scf`：用于循环与条件分支的结构化控制流。
- **EN:** `llvm`: LLVM-style low-level operations and types.
- **CN:** `llvm`：LLVM 风格底层操作与类型。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。