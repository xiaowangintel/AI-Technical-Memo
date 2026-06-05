# amd-canonicalize-pointers-different-bases.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/amd/amd-canonicalize-pointers-different-bases.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-tritonamdgpu-canonicalize-pointers, -canonicalize` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-tritonamdgpu-canonicalize-pointers, -canonicalize` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file -tritonamdgpu-canonicalize-pointers -canonicalize | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file -tritonamdgpu-canonicalize-pointers -canonicalize | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt %s -split-input-file -tritonamdgpu-canonicalize-pointers -canonicalize | FileCheck %s
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file -tritonamdgpu-canonicalize-pointers -canonicalize | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file -tritonamdgpu-canonicalize-pointers -canonicalize | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-7
```mlir
3| // CHECK-LABEL: tt.func @scf_if_different_bases
4| // CHECK: [[BASE:%.*]] = arith.select %arg2, %arg0, %arg1 : !tt.ptr<f32>
5| // CHECK: [[OFFSET:%.*]] = arith.select %arg2, %c16_i32, %c32_i32 : i32
6| // CHECK: [[PTR:%.*]] = tt.addptr [[BASE]], [[OFFSET]]
7| // CHECK: tt.load [[PTR]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tt.func @scf_if_different_bases anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tt.func @scf_if_different_bases 这样的标签用于锚定匹配范围。

### Lines 8-8
```mlir
8| module attributes {"ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 9-25
```mlir
 9|   tt.func @scf_if_different_bases(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
10|                                   %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
11|                                   %arg2: i1) -> f32 {
12|     %c16_i32 = arith.constant 16 : i32
13|     %c32_i32 = arith.constant 32 : i32
14|     %0 = scf.if %arg2 -> (!tt.ptr<f32>) {
15|       %2 = tt.addptr %arg0, %c16_i32 : !tt.ptr<f32>, i32
16|       scf.yield %2 : !tt.ptr<f32>
17|     } else {
18|       %2 = tt.addptr %arg1, %c32_i32 : !tt.ptr<f32>, i32
19|       scf.yield %2 : !tt.ptr<f32>
20|     }
21|     %1 = tt.load %0 : !tt.ptr<f32>
22|     tt.return %1 : f32
23|   }
24| }
25| 
```
**EN:** This function-oriented block defines or enters `scf_if_different_bases`. Within it, the test exercises tt.func, constants, pointer arithmetic, loop/if yielded values, structured conditionals, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `scf_if_different_bases` 为核心。测试在其中演示 tt.func、常量、指针算术、循环/分支产出值、结构化条件分支，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 26-26
```mlir
26| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 27-32
```mlir
27| 
28| // CHECK-LABEL: tt.func @select_different_bases
29| // CHECK: [[BASE:%.*]] = arith.select %arg2, %arg0, %arg1 : !tt.ptr<f32>
30| // CHECK: [[OFFSET:%.*]] = arith.select %arg2, %c16_i32, %c32_i32 : i32
31| // CHECK: [[PTR:%.*]] = tt.addptr [[BASE]], [[OFFSET]]
32| // CHECK: tt.load [[PTR]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tt.func @select_different_bases anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tt.func @select_different_bases 这样的标签用于锚定匹配范围。

### Lines 33-33
```mlir
33| module attributes {"ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 34-45
```mlir
34|   tt.func @select_different_bases(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
35|                                   %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
36|                                   %arg2: i1) -> f32 {
37|     %c16_i32 = arith.constant 16 : i32
38|     %c32_i32 = arith.constant 32 : i32
39|     %2 = tt.addptr %arg0, %c16_i32 : !tt.ptr<f32>, i32
40|     %3 = tt.addptr %arg1, %c32_i32 : !tt.ptr<f32>, i32
41|     %4 = arith.select %arg2, %2, %3 : !tt.ptr<f32>
42|     %5 = tt.load %4 : !tt.ptr<f32>
43|     tt.return %5 : f32
44|   }
45| }
```
**EN:** This function-oriented block defines or enters `select_different_bases`. Within it, the test exercises tt.func, constants, pointer arithmetic, arith.select, masked or vectorized loads, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `select_different_bases` 为核心。测试在其中演示 tt.func、常量、指针算术、arith.select、带掩码或向量化的加载，这些操作构成了被优化、lower 或验证的主要数据流。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-tritonamdgpu-canonicalize-pointers`, `-canonicalize`
- **CN:** 主要 pass 选项：`-split-input-file`，`-tritonamdgpu-canonicalize-pointers`，`-canonicalize`
- **EN:** Dominant operations include `tt.func`, `arith.constant`, `tt.addptr`, `module`, `scf.yield`, `tt.load`, `tt.return`, `scf.if`, `arith.select`.
- **CN:** 主要操作包括 `tt.func`、`arith.constant`、`tt.addptr`、`module`、`scf.yield`、`tt.load`、`tt.return`、`scf.if`、`arith.select`。
- **EN:** The file contains 1 independently testable section(s). Check styles used: CHECK x8, CHECK-LABEL x2. Important labels include tt.func @scf_if_different_bases, tt.func @select_different_bases. It validates that the selected pass pipeline rewrites each labeled test case into the checked form.
- **CN:** 该文件包含 1 个可独立测试的分段。使用的检查类型：CHECK ×8，CHECK-LABEL ×2。 关键标签包括 tt.func @scf_if_different_bases，tt.func @select_different_bases。 它验证所选 pass 流水线会把每个带标签的测试用例重写成检查所要求的形式。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `tt`: Triton high-level tensor/pointer operations.
- **CN:** `tt`：Triton 高层张量/指针操作。
- **EN:** `arith`: MLIR arithmetic dialect for scalar/tensor math.
- **CN:** `arith`：用于标量/张量运算的 MLIR arithmetic 方言。
- **EN:** `scf`: Structured control flow for loops and conditionals.
- **CN:** `scf`：用于循环与条件分支的结构化控制流。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。