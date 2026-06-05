# amd-fpsan.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/amd/amd-fpsan.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-tritonamdgpu-fp-sanitizer` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-tritonamdgpu-fp-sanitizer` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file -tritonamdgpu-fp-sanitizer | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file -tritonamdgpu-fp-sanitizer | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt %s -split-input-file -tritonamdgpu-fp-sanitizer | FileCheck %s
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file -tritonamdgpu-fp-sanitizer | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file -tritonamdgpu-fp-sanitizer | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-3
```mlir
3| #blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [64, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 4-4
```mlir
4| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 5-5
```mlir
5|   // CHECK-LABEL: @scaled_upcast_fp8
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @scaled_upcast_fp8 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @scaled_upcast_fp8 这样的标签用于锚定匹配范围。

### Lines 6-6
```mlir
6|   tt.func public @scaled_upcast_fp8(%src: tensor<32x128xf8E4M3FN, #blocked>, %scale: tensor<32x128xbf16, #blocked>) -> tensor<32x128xbf16, #blocked> {
```
**EN:** This function-oriented block defines or enters `scaled_upcast_fp8`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `scaled_upcast_fp8` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 7-14
```mlir
 7|     // CHECK: tt.fp_to_fp
 8|     // CHECK: arith.mulf
 9|     // CHECK-NOT: amdg.scaled_upcast_fp8
10|     %0 = amdg.scaled_upcast_fp8 %src scale %scale : tensor<32x128xf8E4M3FN, #blocked>, tensor<32x128xbf16, #blocked> -> tensor<32x128xbf16, #blocked>
11|     tt.return %0 : tensor<32x128xbf16, #blocked>
12|   }
13| }
14| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.scaled_upcast_fp8, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.scaled_upcast_fp8、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 15-15
```mlir
15| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 16-17
```mlir
16| 
17| #blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [64, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 18-18
```mlir
18| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 19-19
```mlir
19|   // CHECK-LABEL: @scaled_upcast_fp4
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @scaled_upcast_fp4 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @scaled_upcast_fp4 这样的标签用于锚定匹配范围。

### Lines 20-20
```mlir
20|   tt.func public @scaled_upcast_fp4(%src: tensor<16x32xi8, #blocked>, %scale: tensor<16x64xbf16, #blocked>) -> tensor<16x64xbf16, #blocked> {
```
**EN:** This function-oriented block defines or enters `scaled_upcast_fp4`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `scaled_upcast_fp4` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 21-27
```mlir
21|     // CHECK: ttg.fp4_to_fp
22|     // CHECK: arith.mulf
23|     // CHECK-NOT: amdg.scaled_upcast_fp4
24|     %0 = amdg.scaled_upcast_fp4 %src scale %scale {axis = 1 : i32} : tensor<16x32xi8, #blocked>, tensor<16x64xbf16, #blocked> -> tensor<16x64xbf16, #blocked>
25|     tt.return %0 : tensor<16x64xbf16, #blocked>
26|   }
27| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.scaled_upcast_fp4, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.scaled_upcast_fp4、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-tritonamdgpu-fp-sanitizer`
- **CN:** 主要 pass 选项：`-split-input-file`，`-tritonamdgpu-fp-sanitizer`
- **EN:** Dominant operations include `tt.func`, `module`, `tt.return`, `amdg.scaled_upcast_fp8`, `amdg.scaled_upcast_fp4`.
- **CN:** 主要操作包括 `tt.func`、`module`、`tt.return`、`amdg.scaled_upcast_fp8`、`amdg.scaled_upcast_fp4`。
- **EN:** The file contains 1 independently testable section(s). Check styles used: CHECK x4, CHECK-LABEL x2, CHECK-NOT x2. Important labels include @scaled_upcast_fp8, @scaled_upcast_fp4. It validates both the presence of the desired rewrite and the absence of unwanted residual operations.
- **CN:** 该文件包含 1 个可独立测试的分段。使用的检查类型：CHECK ×4，CHECK-LABEL ×2，CHECK-NOT ×2。 关键标签包括 @scaled_upcast_fp8，@scaled_upcast_fp4。 它同时验证目标重写已经出现，并且不再残留不需要的操作。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `tt`: Triton high-level tensor/pointer operations.
- **CN:** `tt`：Triton 高层张量/指针操作。
- **EN:** `ttg`: TritonGPU layout, memory, and GPU-specific operations.
- **CN:** `ttg`：TritonGPU 布局、内存和 GPU 专用操作。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。