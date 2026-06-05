# amd-update-async-wait-count-asyncmark.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/amd/amd-update-async-wait-count-asyncmark.mlir`
- **EN:** Test for the MLIR/Triton pipeline `--tritonamdgpu-update-async-wait-count=gfx-arch=gfx950` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `--tritonamdgpu-update-async-wait-count=gfx-arch=gfx950` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file --tritonamdgpu-update-async-wait-count=gfx-arch=gfx950 | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file --tritonamdgpu-update-async-wait-count=gfx-arch=gfx950 | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-10
```mlir
 1| // RUN: triton-opt %s -split-input-file --tritonamdgpu-update-async-wait-count=gfx-arch=gfx950 | FileCheck %s
 2| 
 3| // For CDNA3/CDNA4, ttg.async_wait is generated from 3 sources:
 4| //   - By pipeliner, num computed within the pass.
 5| //   - By block-pingpong, num computed within the pass.
 6| //   - By gluon, num filled by user.
 7| //
 8| // On CDNA3/CDNA4, since PR #9883, UpdateAsyncWaitCount stays no-op for all 3
 9| // cases, which is checked by this test.
10| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file --tritonamdgpu-update-async-wait-count=gfx-arch=gfx950 | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file --tritonamdgpu-update-async-wait-count=gfx-arch=gfx950 | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 11-13
```mlir
11| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [32, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
12| #shared = #ttg.swizzled_shared<{vec = 8, perPhase = 8, maxPhase = 2, order = [1, 0]}>
13| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 14-14
```mlir
14| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 15-17
```mlir
15|   // CHECK-LABEL: single_token_two_crossed
16|   // Wait on %1 with %3 and %5 between - derivation would yield num=2; sentinel
17|   // num=7 must survive.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 18-18
```mlir
18|   // CHECK: ttg.async_wait %{{[^,]+}} {num = 7 : i32}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 19-30
```mlir
19|   tt.func public @single_token_two_crossed(%arg0: !ttg.memdesc<128x16xf16, #shared, #smem, mutable>, %arg1: tensor<128x16x!tt.ptr<f16>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>}) {
20|     %0 = ttg.async_copy_global_to_local %arg1, %arg0 : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
21|     %1 = ttg.async_commit_group tokens %0
22|     %2 = ttg.async_copy_global_to_local %arg1, %arg0 : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
23|     %3 = ttg.async_commit_group tokens %2
24|     %4 = ttg.async_copy_global_to_local %arg1, %arg0 : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
25|     %5 = ttg.async_commit_group tokens %4
26|     %6 = ttg.async_wait %1 {num = 7 : i32}
27|     tt.return
28|   }
29| }
30| 
```
**EN:** This function-oriented block defines or enters `single_token_two_crossed`. Within it, the test exercises ttg.async_copy_global_to_local, async copy commit groups, tt.func, async wait synchronization, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `single_token_two_crossed` 为核心。测试在其中演示 ttg.async_copy_global_to_local、异步拷贝提交组、tt.func、异步等待同步、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 31-31
```mlir
31| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 32-35
```mlir
32| 
33| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [32, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
34| #shared = #ttg.swizzled_shared<{vec = 8, perPhase = 8, maxPhase = 2, order = [1, 0]}>
35| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 36-36
```mlir
36| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 37-39
```mlir
37|   // CHECK-LABEL: tokenless_wait_preserved
38|   // Tokenless wait carries a producer-authored num that derivation cannot
39|   // recover from a def chain - num=3 stays put.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 40-40
```mlir
40|   // CHECK: ttg.async_wait {num = 3 : i32}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 41-47
```mlir
41|   tt.func public @tokenless_wait_preserved(%arg0: !ttg.memdesc<128x16xf16, #shared, #smem, mutable>, %arg1: tensor<128x16x!tt.ptr<f16>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>}) {
42|     %0 = ttg.async_copy_global_to_local %arg1, %arg0 : tensor<128x16x!tt.ptr<f16>, #blocked> -> <128x16xf16, #shared, #smem, mutable>
43|     %1 = ttg.async_commit_group tokens %0
44|     ttg.async_wait {num = 3 : i32}
45|     tt.return
46|   }
47| }
```
**EN:** This function-oriented block defines or enters `tokenless_wait_preserved`. Within it, the test exercises tt.func, ttg.async_copy_global_to_local, async copy commit groups, async wait synchronization, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tokenless_wait_preserved` 为核心。测试在其中演示 tt.func、ttg.async_copy_global_to_local、异步拷贝提交组、异步等待同步、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `--tritonamdgpu-update-async-wait-count=gfx-arch=gfx950`
- **CN:** 主要 pass 选项：`-split-input-file`，`--tritonamdgpu-update-async-wait-count=gfx-arch=gfx950`
- **EN:** Dominant operations include `tt.func`, `ttg.async_copy_global_to_local`, `ttg.async_commit_group`, `module`, `ttg.async_wait`, `tt.return`.
- **CN:** 主要操作包括 `tt.func`、`ttg.async_copy_global_to_local`、`ttg.async_commit_group`、`module`、`ttg.async_wait`、`tt.return`。
- **EN:** The file contains 1 independently testable section(s). Check styles used: CHECK-LABEL x2, CHECK x2. Important labels include single_token_two_crossed, tokenless_wait_preserved. It validates that the selected pass pipeline rewrites each labeled test case into the checked form.
- **CN:** 该文件包含 1 个可独立测试的分段。使用的检查类型：CHECK-LABEL ×2，CHECK ×2。 关键标签包括 single_token_two_crossed，tokenless_wait_preserved。 它验证所选 pass 流水线会把每个带标签的测试用例重写成检查所要求的形式。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `ttg`: TritonGPU layout, memory, and GPU-specific operations.
- **CN:** `ttg`：TritonGPU 布局、内存和 GPU 专用操作。
- **EN:** `tt`: Triton high-level tensor/pointer operations.
- **CN:** `tt`：Triton 高层张量/指针操作。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。