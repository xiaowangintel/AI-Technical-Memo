# tma_lowering.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonNvidiaGPU/tma_lowering.mlir`
- **EN:** Test for the MLIR/Triton pipeline `--triton-nvidia-tma-lowering` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `--triton-nvidia-tma-lowering` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file --triton-nvidia-tma-lowering | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file --triton-nvidia-tma-lowering | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-1
```mlir
1| // RUN: triton-opt %s -split-input-file --triton-nvidia-tma-lowering | FileCheck %s
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file --triton-nvidia-tma-lowering | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file --triton-nvidia-tma-lowering | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 2-4
```mlir
2| #nvmma_128 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
3| 
4| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 5-5
```mlir
5| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 6-13
```mlir
 6| // CHECK-LABEL: tma_load
 7| // CHECK: ttg.local_alloc : ()
 8| // CHECK: ttg.local_alloc : ()
 9| // CHECK: ttng.init_barrier
10| // CHECK: ttng.async_tma_copy_global_to_local
11| // CHECK: ttng.wait_barrier
12| // CHECK: ttng.inval_barrier
13| // CHECK: ttg.local_load
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tma_load anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tma_load 这样的标签用于锚定匹配范围。

### Lines 14-19
```mlir
14|   tt.func public @tma_load(%arg0: !tt.tensordesc<128x64xf16, #nvmma_128>, %arg1: i32) -> tensor<128x64xf16, #blocked> {
15|     %l = tt.descriptor_load %arg0[%arg1, %arg1] : !tt.tensordesc<128x64xf16, #nvmma_128> -> tensor<128x64xf16, #blocked>
16|     tt.return %l : tensor<128x64xf16, #blocked>
17|   }
18| }
19| 
```
**EN:** This function-oriented block defines or enters `tma_load`. Within it, the test exercises tt.func, tt.descriptor_load, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tma_load` 为核心。测试在其中演示 tt.func、tt.descriptor_load、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 20-20
```mlir
20| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 21-23
```mlir
21| #nvmma_128 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
22| 
23| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 24-24
```mlir
24| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 25-28
```mlir
25| // CHECK-LABEL: tma_store
26| //       CHECK: ttg.local_alloc {{.*}} -> !ttg.memdesc<128x256xf32, #shared, #smem>
27| //       CHECK: ttng.fence_async_shared {bCluster = false}
28| //       CHECK: ttng.async_tma_copy_local_to_global
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 29-34
```mlir
29|   tt.func public @tma_store(%arg0: !tt.tensordesc<128x256xf32, #nvmma_128>, %arg1: i32 {tt.divisibility = 16 : i32}, %arg2: tensor<128x256xf32, #blocked>) {
30|     tt.descriptor_store %arg0[%arg1, %arg1], %arg2 : !tt.tensordesc<128x256xf32, #nvmma_128>, tensor<128x256xf32, #blocked>
31|     tt.return
32|   }
33| }
34| 
```
**EN:** This function-oriented block defines or enters `tma_store`. Within it, the test exercises tt.func, tt.descriptor_store, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tma_store` 为核心。测试在其中演示 tt.func、tt.descriptor_store、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 35-35
```mlir
35| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 36-37
```mlir
36| #nvmma_32 = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = false, elementBitWidth = 8}>
37| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 38-38
```mlir
38| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 39-44
```mlir
39|   // CHECK-LABEL: make_tensor_descriptor
40|   // CHECK: %0 = arith.extsi %arg2 : i32 to i64
41|   // CHECK: %1 = ttg.global_scratch_alloc {alignment = 128 : i32, nbytes = 128 : i32} : !tt.ptr<i8>
42|   // CHECK: ttng.tensormap_create %1, %arg0, [%c32_i32, %c8_i32], [%arg2, %arg1], [%0], [%c1_i32, %c1_i32] {elem_type = 0 : i32, fill_mode = 0 : i32, interleave_layout = 0 : i32, swizzle_mode = 1 : i32} : (!tt.ptr<i8>, !tt.ptr<i8>, i32, i32, i32, i32, i64, i32, i32) -> ()
43|   // CHECK: ttng.tensormap_fenceproxy_acquire %1 : !tt.ptr<i8>
44|   // CHECK: ttng.reinterpret_tensor_descriptor %1 : !tt.ptr<i8> to !tt.tensordesc<8x32xi8, #shared>
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: make_tensor_descriptor anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: make_tensor_descriptor 这样的标签用于锚定匹配范围。

### Lines 45-55
```mlir
45|   tt.func public @make_tensor_descriptor(%arg0: !tt.ptr<i8> {tt.divisibility = 16 : i32}, %arg1: i32 {tt.divisibility = 16 : i32}, %arg2: i32 {tt.divisibility = 16 : i32} ) -> !tt.tensordesc<8x32xi8, #nvmma_32> {
46|     %c1_i64 = arith.constant 1 : i64
47|     %cst = arith.constant dense<32> : tensor<8x1xi32>
48|     %c64_i32 = arith.constant 64 : i32
49|     %c8_i32 = arith.constant 8 : i32
50|     %0 = arith.extsi %arg2 : i32 to i64
51|     %1 = tt.make_tensor_descriptor %arg0, [%arg1, %arg2], [%0, %c1_i64] : !tt.ptr<i8>, !tt.tensordesc<8x32xi8, #nvmma_32>
52|     tt.return %1 : !tt.tensordesc<8x32xi8, #nvmma_32>
53|   }
54| }
55| 
```
**EN:** This function-oriented block defines or enters `make_tensor_descriptor`. Within it, the test exercises constants, tt.func, arith.extsi, tt.make_tensor_descriptor, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `make_tensor_descriptor` 为核心。测试在其中演示 常量、tt.func、arith.extsi、tt.make_tensor_descriptor、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 56-56
```mlir
56| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 57-59
```mlir
57| #nvmma_128 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
58| 
59| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 60-60
```mlir
60| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 61-64
```mlir
61| // CHECK-LABEL: tma_reduce
62| //       CHECK: ttg.local_alloc {{.*}} -> !ttg.memdesc<128x256xf32, #shared, #smem>
63| //       CHECK: ttng.fence_async_shared {bCluster = false}
64| //       CHECK: ttng.async_tma_reduce add
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 65-70
```mlir
65|   tt.func public @tma_reduce(%arg0: !tt.tensordesc<128x256xf32, #nvmma_128>, %arg1: i32 {tt.divisibility = 16 : i32}, %arg2: tensor<128x256xf32, #blocked>) {
66|     tt.descriptor_reduce add, %arg0[%arg1, %arg1], %arg2 : !tt.tensordesc<128x256xf32, #nvmma_128>, tensor<128x256xf32, #blocked>
67|     tt.return
68|   }
69| }
70| 
```
**EN:** This function-oriented block defines or enters `tma_reduce`. Within it, the test exercises tt.func, tt.descriptor_reduce, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tma_reduce` 为核心。测试在其中演示 tt.func、tt.descriptor_reduce、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 71-71
```mlir
71| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 72-77
```mlir
72| 
73| #blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [1, 0]}>
74| #offsets = #ttg.slice<{dim = 0, parent = #blocked}>
75| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
76| #nvmma_128 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
77| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 78-79
```mlir
78| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100"} {
79| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 80-80
```mlir
80| // CHECK-LABEL: @tma_gather
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tma_gather anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tma_gather 这样的标签用于锚定匹配范围。

### Lines 81-81
```mlir
81| tt.func @tma_gather(%arg0: !tt.tensordesc<1x128xbf16, #nvmma_128>, %arg1: tensor<32xi32, #offsets>, %arg2: i32) -> tensor<32x128xbf16, #blocked1> {
```
**EN:** This function-oriented block defines or enters `tma_gather`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tma_gather` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 82-89
```mlir
82|   // CHECK: [[RESULT:%.*]] = ttg.local_alloc
83|   // CHECK: [[BARRIER:%.*]] = ttg.local_alloc
84|   // CHECK: ttng.init_barrier [[BARRIER]]
85|   // CHECK: ttng.async_tma_gather %arg0[%arg1, %arg2] [[RESULT]], [[BARRIER]], %true
86|   // CHECK: ttng.wait_barrier [[BARRIER]]
87|   // CHECK: ttng.inval_barrier [[BARRIER]]
88|   // CHECK: [[OUT:%.*]] = ttg.local_load [[RESULT]]
89|   %0 = tt.descriptor_gather %arg0[%arg1, %arg2] : (!tt.tensordesc<1x128xbf16, #nvmma_128>, tensor<32xi32, #offsets>, i32) -> tensor<32x128xbf16, #blocked1>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.descriptor_gather. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.descriptor_gather。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 90-93
```mlir
90|   // CHECK: return [[OUT]]
91|   tt.return %0 : tensor<32x128xbf16, #blocked1>
92| }
93| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 94-94
```mlir
94| // CHECK-LABEL: @tma_scatter
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tma_scatter anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tma_scatter 这样的标签用于锚定匹配范围。

### Lines 95-95
```mlir
95| tt.func @tma_scatter(%arg0: !tt.tensordesc<1x128xbf16, #nvmma_128>, %arg1: tensor<32xi32, #offsets>, %arg2: i32, %arg3: tensor<32x128xbf16, #blocked1>) {
```
**EN:** This function-oriented block defines or enters `tma_scatter`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tma_scatter` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 96-105
```mlir
 96|   // CHECK-NEXT: [[SRC:%.*]] = ttg.local_alloc %arg3
 97|   // CHECK-NEXT: ttng.fence_async_shared {bCluster = false}
 98|   // CHECK-NEXT: ttng.async_tma_scatter %arg0[%arg1, %arg2] [[SRC]]
 99|   // CHECK-NEXT: ttng.async_tma_store_wait
100|   tt.descriptor_scatter %arg0[%arg1, %arg2], %arg3 : !tt.tensordesc<1x128xbf16, #nvmma_128>, tensor<32xi32, #offsets>, i32, tensor<32x128xbf16, #blocked1>
101|   tt.return
102| }
103| 
104| }
105| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.descriptor_scatter, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.descriptor_scatter、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 106-106
```mlir
106| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 107-109
```mlir
107| 
108| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [8, 1], order = [1, 0]}>
109| #nvmma_128 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 110-110
```mlir
110| // CHECK: #[[$NVMMA:.+]] = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 111-111
```mlir
111| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 112-112
```mlir
112|   // CHECK-LABLE: @rank_reducing_load
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 113-114
```mlir
113|   tt.func public @rank_reducing_load(%arg0: !tt.tensordesc<1x256x32xf32, #nvmma_128>) -> tensor<256x32xf32, #blocked> {
114|       %c32_i32 = arith.constant 32 : i32
```
**EN:** This function-oriented block defines or enters `rank_reducing_load`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `rank_reducing_load` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 115-121
```mlir
115|       // CHECK: %[[A:.+]] = ttg.local_alloc : () -> !ttg.memdesc<256x32xf32, #[[$NVMMA]], #smem, mutable>
116|       // CHECK: tng.async_tma_copy_global_to_local %{{.+}}[%{{.+}}, %{{.+}}, %{{.+}}] %[[A]],
117|       %l = tt.descriptor_load %arg0[%c32_i32, %c32_i32, %c32_i32] : !tt.tensordesc<1x256x32xf32, #nvmma_128> -> tensor<256x32xf32, #blocked>
118|       tt.return %l : tensor<256x32xf32, #blocked>
119|   }
120| }
121| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.descriptor_load, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.descriptor_load、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 122-122
```mlir
122| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 123-127
```mlir
123| 
124| #blocked = #ttg.blocked<{sizePerThread = [4, 4], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
125| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
126| #nvmma_128 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
127| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 128-128
```mlir
128| // CHECK: #[[$NVMMA:.+]] = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 129-129
```mlir
129| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 130-130
```mlir
130|   // CHECK-LABEL: @tma_load_alloc_user
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tma_load_alloc_user anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tma_load_alloc_user 这样的标签用于锚定匹配范围。

### Lines 131-132
```mlir
131|   tt.func public @tma_load_alloc_user(%arg0: !tt.tensordesc<64x64xf32, #nvmma_128>, %arg1: i32) -> (tensor<64x64xf32, #blocked>, !ttg.memdesc<64x64xf32, #shared, #smem, mutable>) {
132|     %0 = tt.descriptor_load %arg0[%arg1, %arg1] : !tt.tensordesc<64x64xf32, #nvmma_128> -> tensor<64x64xf32, #blocked>
```
**EN:** This function-oriented block defines or enters `tma_load_alloc_user`. Within it, the test exercises tt.func, tt.descriptor_load, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tma_load_alloc_user` 为核心。测试在其中演示 tt.func、tt.descriptor_load，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 133-135
```mlir
133|     // CHECK: %[[A:.+]] = ttg.local_alloc : () -> !ttg.memdesc<64x64xf32, #[[$NVMMA]], #smem, mutable>
134|     // CHECK: tng.async_tma_copy_global_to_local %{{.+}}[%{{.+}}, %{{.+}}] %[[A]],
135|     %1 = ttg.local_alloc %0 : (tensor<64x64xf32, #blocked>) -> !ttg.memdesc<64x64xf32, #shared, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 136-142
```mlir
136|     // CHECK: %[[L:.+]] = ttg.local_load %[[A]] :
137|     // CHECK: %[[S:.+]] = ttg.local_alloc %[[L]] :
138|     // CHECK: tt.return %[[L]], %[[S]] :
139|     tt.return %0, %1 : tensor<64x64xf32, #blocked>, !ttg.memdesc<64x64xf32, #shared, #smem, mutable>
140|   }
141| }
142| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 143-143
```mlir
143| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 144-151
```mlir
144| 
145| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
146| #mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [1, 4], instrShape = [16, 8]}>
147| #mma1 = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 64, 16]}>
148| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 16}>
149| #shared1 = #ttg.swizzled_shared<{vec = 8, perPhase = 2, maxPhase = 4, order = [1, 0]}>
150| #shared2 = #ttg.swizzled_shared<{vec = 8, perPhase = 2, maxPhase = 4, order = [0, 1]}>
151| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 152-152
```mlir
152| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 153-153
```mlir
153|   // CHECK-LABEL: @tma_load_double_use
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tma_load_double_use anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tma_load_double_use 这样的标签用于锚定匹配范围。

### Lines 154-158
```mlir
154|   tt.func public @tma_load_double_use(%arg0: !tt.tensordesc<64x32xf32, #shared>, %arg1: !tt.tensordesc<64x64xf32, #shared1>) -> tensor<64x32xf32, #mma1> {
155|     %cst = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #mma>
156|     %cst_0 = arith.constant dense<0.000000e+00> : tensor<64x32xf32, #mma1>
157|     %c32_i32 = arith.constant 32 : i32
158|     %c64_i32 = arith.constant 64 : i32
```
**EN:** This function-oriented block defines or enters `tma_load_double_use`. Within it, the test exercises constants, tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tma_load_double_use` 为核心。测试在其中演示 常量、tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 159-160
```mlir
159|     // CHECK: %[[A:.+]] = ttg.local_alloc : () -> !ttg.memdesc<64x32xf32
160|     %0 = tt.descriptor_load %arg0[%c64_i32, %c32_i32] : !tt.tensordesc<64x32xf32, #shared> -> tensor<64x32xf32, #blocked>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.descriptor_load. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.descriptor_load。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 161-163
```mlir
161|     // CHECK: %[[B:.+]] = ttg.local_load %[[A]]
162|     // CHECK: %[[C:.+]] = ttg.local_alloc %[[B]]
163|     %1 = ttg.local_alloc %0 : (tensor<64x32xf32, #blocked>) -> !ttg.memdesc<64x32xf32, #shared1, #smem>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 164-166
```mlir
164|     // CHECK: %[[D:.+]] = ttg.memdesc_trans %[[C]]
165|     %2 = ttg.memdesc_trans %1 {order = array<i32: 1, 0>} : !ttg.memdesc<64x32xf32, #shared1, #smem> -> !ttg.memdesc<32x64xf32, #shared2, #smem>
166|     %3 = ttg.local_alloc %0 : (tensor<64x32xf32, #blocked>) -> !ttg.memdesc<64x32xf32, #shared, #smem>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.memdesc_trans, shared/local memory allocation. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.memdesc_trans、共享/本地内存分配。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 167-168
```mlir
167|     // CHECK: %[[E:.+]] = ttg.local_load %[[D]]
168|     %4 = ttg.local_load %2 : !ttg.memdesc<32x64xf32, #shared2, #smem> -> tensor<32x64xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on local/shared memory loads. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 本地/共享内存加载。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 169-170
```mlir
169|     // CHECK: %[[F:.+]] = ttg.local_load %[[A]]
170|     %5 = ttg.local_load %3 : !ttg.memdesc<64x32xf32, #shared, #smem> -> tensor<64x32xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on local/shared memory loads. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 本地/共享内存加载。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 171-172
```mlir
171|     // CHECK: %[[G:.+]] = tt.dot %[[E]], %[[F]]
172|     %6 = tt.dot %4, %5, %cst, inputPrecision = tf32 : tensor<32x64xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<64x32xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<32x32xf32, #mma>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on dot-product or MMA-style math. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 点积或 MMA 风格计算。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 173-174
```mlir
173|     // CHECK: %[[H:.+]] = ttg.local_alloc %[[G]]
174|     %7 = ttg.local_alloc %6 : (tensor<32x32xf32, #mma>) -> !ttg.memdesc<32x32xf32, #shared, #smem>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 175-180
```mlir
175|     // CHECK: {{.*}} = ttng.warp_group_dot %[[A]], %[[H]]
176|     %8 = ttng.warp_group_dot %3, %7, %cst_0 {isAsync = true} : !ttg.memdesc<64x32xf32, #shared, #smem> * !ttg.memdesc<32x32xf32, #shared, #smem> -> tensor<64x32xf32, #mma1>
177|     %9:3 = ttng.warp_group_dot_wait %8, %3, %7 {pendings = 0 : i32} : tensor<64x32xf32, #mma1>, !ttg.memdesc<64x32xf32, #shared, #smem>, !ttg.memdesc<32x32xf32, #shared, #smem>
178|     tt.return %9 : tensor<64x32xf32, #mma1>
179|   }
180| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.warp_group_dot, ttng.warp_group_dot_wait, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.warp_group_dot、ttng.warp_group_dot_wait、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `--triton-nvidia-tma-lowering`
- **CN:** 主要 pass 选项：`-split-input-file`，`--triton-nvidia-tma-lowering`
- **EN:** Dominant operations include `tt.func`, `tt.return`, `arith.constant`, `module`, `tt.descriptor_load`, `ttg.local_alloc`, `ttg.local_load`, `tt.descriptor_store`, `arith.extsi`, `tt.make_tensor_descriptor`.
- **CN:** 主要操作包括 `tt.func`、`tt.return`、`arith.constant`、`module`、`tt.descriptor_load`、`ttg.local_alloc`、`ttg.local_load`、`tt.descriptor_store`、`arith.extsi`、`tt.make_tensor_descriptor`。
- **EN:** The file contains 7 independently testable section(s). Check styles used: CHECK x44, CHECK-LABEL x8, CHECK-NEXT x4, CHECK-LABLE x1. Important labels include tma_load, tma_store, make_tensor_descriptor, tma_reduce. It validates that the selected pass pipeline rewrites each labeled test case into the checked form.
- **CN:** 该文件包含 7 个可独立测试的分段。使用的检查类型：CHECK ×44，CHECK-LABEL ×8，CHECK-NEXT ×4，CHECK-LABLE ×1。 关键标签包括 tma_load，tma_store，make_tensor_descriptor，tma_reduce。 它验证所选 pass 流水线会把每个带标签的测试用例重写成检查所要求的形式。

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