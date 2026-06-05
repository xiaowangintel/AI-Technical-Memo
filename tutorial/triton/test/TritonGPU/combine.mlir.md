# combine.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/combine.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-tritongpu-remove-layout-conversions, -cse` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-tritongpu-remove-layout-conversions, -cse` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-remove-layout-conversions -cse | FileCheck --dump-input-context=10 %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-remove-layout-conversions -cse | FileCheck --dump-input-context=10 %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-remove-layout-conversions -cse | FileCheck --dump-input-context=10 %s
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-remove-layout-conversions -cse | FileCheck --dump-input-context=10 %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-remove-layout-conversions -cse | FileCheck --dump-input-context=10 %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-13
```mlir
 3| #layout0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
 4| #layout1 = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
 5| 
 6| #layout2 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
 7| #layout3 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [2, 16], warpsPerCTA = [1, 4], order = [1, 0]}>
 8| 
 9| #layout4 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [2, 2], order = [0, 1]}>
10| #layout5 = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [32, 1], warpsPerCTA = [2, 2], order = [0, 1]}>
11| #linear = #ttg.linear<{register = [[0, 1], [0, 2], [0, 4], [0, 8], [0, 16]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 32]], warp = [[16, 0], [32, 0]], block = []}>
12| 
13| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 14-15
```mlir
14| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
15| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 16-17
```mlir
16| // CHECK: [[$target_layout:#.*]] = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
17| // CHECK-LABEL: cst
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: cst anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: cst 这样的标签用于锚定匹配范围。

### Lines 18-20
```mlir
18| tt.func @cst() -> tensor<1024xi32, #layout1> {
19|   %cst = arith.constant dense<0> : tensor<1024xi32, #layout0>
20|   %1 = ttg.convert_layout %cst : tensor<1024xi32, #layout0> -> tensor<1024xi32, #layout1>
```
**EN:** This function-oriented block defines or enters `cst`. Within it, the test exercises tt.func, constants, layout conversions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `cst` 为核心。测试在其中演示 tt.func、常量、布局转换，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 21-25
```mlir
21|   // CHECK-NOT: ttg.convert_layout
22|   // CHECK: tt.return %cst : tensor<1024xi32, [[$target_layout]]>
23|   tt.return %1: tensor<1024xi32, #layout1>
24| }
25| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 26-26
```mlir
26| // CHECK-LABEL: range
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: range anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: range 这样的标签用于锚定匹配范围。

### Lines 27-29
```mlir
27| tt.func @range() -> tensor<1024xi32, #layout1> {
28|   %0 = tt.make_range {end = 1024 : i32, start = 0 : i32} : tensor<1024xi32, #layout0>
29|   %1 = ttg.convert_layout %0 : tensor<1024xi32, #layout0> -> tensor<1024xi32, #layout1>
```
**EN:** This function-oriented block defines or enters `range`. Within it, the test exercises tt.func, lane/block index ranges, layout conversions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `range` 为核心。测试在其中演示 tt.func、lane/block 索引范围、布局转换，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 30-34
```mlir
30|   // CHECK-NOT: ttg.convert_layout
31|   // CHECK: tt.return %0 : tensor<1024xi32, [[$target_layout]]>
32|   tt.return %1: tensor<1024xi32, #layout1>
33| }
34| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 35-35
```mlir
35| // CHECK-LABEL: splat
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: splat anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: splat 这样的标签用于锚定匹配范围。

### Lines 36-38
```mlir
36| tt.func @splat(%arg0: i32) -> tensor<1024xi32, #layout1> {
37|   %0 = tt.splat %arg0 : i32 -> tensor<1024xi32, #layout0>
38|   %1 = ttg.convert_layout %0 : tensor<1024xi32, #layout0> -> tensor<1024xi32, #layout1>
```
**EN:** This function-oriented block defines or enters `splat`. Within it, the test exercises tt.func, broadcasted scalars or pointers, layout conversions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `splat` 为核心。测试在其中演示 tt.func、广播后的标量或指针、布局转换，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 39-43
```mlir
39|   // CHECK-NOT: ttg.convert_layout
40|   // CHECK: tt.return %0 : tensor<1024xi32, [[$target_layout]]>
41|   tt.return %1: tensor<1024xi32, #layout1>
42| }
43| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 44-44
```mlir
44| // CHECK-LABEL: remat
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: remat anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: remat 这样的标签用于锚定匹配范围。

### Lines 45-53
```mlir
45| tt.func @remat(%arg0: i32) -> tensor<1024xi32, #layout1> {
46|   %0 = tt.make_range {end = 1024 : i32, start = 0 : i32} : tensor<1024xi32, #layout0>
47|   %1 = tt.make_range {end = 1024 : i32, start = 0 : i32} : tensor<1024xi32, #layout0>
48|   %2 = arith.muli %0, %1 : tensor<1024xi32, #layout0>
49|   %3 = ttg.convert_layout %2 : tensor<1024xi32, #layout0> -> tensor<1024xi32, #layout1>
50|   %4 = tt.splat %arg0 : i32 -> tensor<1024xi32, #layout0>
51|   %5 = ttg.convert_layout %2 : tensor<1024xi32, #layout0> -> tensor<1024xi32, #layout1>
52|   %6 = arith.addi %3, %5 : tensor<1024xi32, #layout1>
53|   tt.return %6: tensor<1024xi32, #layout1>
```
**EN:** This function-oriented block defines or enters `remat`. Within it, the test exercises tt.func, lane/block index ranges, layout conversions, integer multiplications, broadcasted scalars or pointers, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `remat` 为核心。测试在其中演示 tt.func、lane/block 索引范围、布局转换、整数乘法、广播后的标量或指针，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 54-60
```mlir
54|   // CHECK: %[[A:.+]] = tt.make_range {end = 1024 : i32, start = 0 : i32} : tensor<1024xi32, [[$target_layout]]>
55|   // CHECK: %[[C:.+]] = arith.muli %[[A]], %[[A]] : tensor<1024xi32, [[$target_layout]]>
56|   // CHECK: %[[D:.+]] = arith.addi %[[C]], %[[C]] : tensor<1024xi32, [[$target_layout]]>
57|   // CHECK: tt.return %[[D]] : tensor<1024xi32, [[$target_layout]]>
58| }
59| 
60| // Always rematerialize single value loads
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 61-61
```mlir
61| // CHECK-LABEL: remat_single_value
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: remat_single_value anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: remat_single_value 这样的标签用于锚定匹配范围。

### Lines 62-64
```mlir
62| tt.func @remat_single_value(%arg: !tt.ptr<i32> {tt.divisibility = 16 : i32}) {
63|   %0 = tt.splat %arg : !tt.ptr<i32> -> tensor<1x!tt.ptr<i32>, #layout1>
64|   %1 = tt.load %0 : tensor<1x!tt.ptr<i32>, #layout1>
```
**EN:** This function-oriented block defines or enters `remat_single_value`. Within it, the test exercises tt.func, broadcasted scalars or pointers, masked or vectorized loads, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `remat_single_value` 为核心。测试在其中演示 tt.func、广播后的标量或指针、带掩码或向量化的加载，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 65-71
```mlir
65|   // CHECK-NOT: ttg.convert_layout
66|   %2 = ttg.convert_layout %1 : tensor<1xi32, #layout1> -> tensor<1xi32, #layout0>
67|   %3 = ttg.convert_layout %0 : tensor<1x!tt.ptr<i32>, #layout1> -> tensor<1x!tt.ptr<i32>, #layout0>
68|   tt.store %3, %2 : tensor<1x!tt.ptr<i32>, #layout0>
69|   tt.return
70| }
71| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, masked or vectorized stores, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、带掩码或向量化的存储、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 72-72
```mlir
72| // CHECK-LABEL: remat_fast_load
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: remat_fast_load anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: remat_fast_load 这样的标签用于锚定匹配范围。

### Lines 73-77
```mlir
73| tt.func @remat_fast_load(%arg: !tt.ptr<i32> {tt.divisibility = 16 : i32}) {
74|   %0 = tt.splat %arg : !tt.ptr<i32> -> tensor<16x!tt.ptr<i32>, #layout1>
75|   %1 = tt.make_range {end = 16 : i32, start = 0 : i32} : tensor<16xi32, #layout1>
76|   %2 = tt.addptr %0, %1 : tensor<16x!tt.ptr<i32>, #layout1>, tensor<16xi32, #layout1>
77|   %3 = tt.load %2 : tensor<16x!tt.ptr<i32>, #layout1>
```
**EN:** This function-oriented block defines or enters `remat_fast_load`. Within it, the test exercises tt.func, broadcasted scalars or pointers, lane/block index ranges, pointer arithmetic, masked or vectorized loads, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `remat_fast_load` 为核心。测试在其中演示 tt.func、广播后的标量或指针、lane/block 索引范围、指针算术、带掩码或向量化的加载，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 78-84
```mlir
78|   // CHECK-NOT: ttg.convert_layout
79|   %4 = ttg.convert_layout %3 : tensor<16xi32, #layout1> -> tensor<16xi32, #layout0>
80|   %5 = ttg.convert_layout %2 : tensor<16x!tt.ptr<i32>, #layout1> -> tensor<16x!tt.ptr<i32>, #layout0>
81|   tt.store %5, %4 : tensor<16x!tt.ptr<i32>, #layout0>
82|   tt.return
83| }
84| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, masked or vectorized stores, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、带掩码或向量化的存储、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 85-85
```mlir
85| // CHECK-LABEL: fp4_keep_convert
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: fp4_keep_convert anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: fp4_keep_convert 这样的标签用于锚定匹配范围。

### Lines 86-89
```mlir
86| tt.func @fp4_keep_convert() -> tensor<64x64xf16, #linear> {
87|   %0 = arith.constant dense<0> : tensor<64x32xi8, #layout4>
88|   %fp4 = ttg.fp4_to_fp %0 {axis = 1 : i32} : tensor<64x32xi8, #layout4> -> tensor<64x64xf16, #layout5>
89|   %converted = ttg.convert_layout %fp4 : tensor<64x64xf16, #layout5> -> tensor<64x64xf16, #linear>
```
**EN:** This function-oriented block defines or enters `fp4_keep_convert`. Within it, the test exercises tt.func, constants, ttg.fp4_to_fp, layout conversions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fp4_keep_convert` 为核心。测试在其中演示 tt.func、常量、ttg.fp4_to_fp、布局转换，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 90-95
```mlir
90|   // CHECK: ttg.fp4_to_fp
91|   // CHECK-NOT: ttg.convert_layout
92|   tt.return %converted : tensor<64x64xf16, #linear>
93| }
94| 
95| // Hoist the convert on top of ext to make it cheaper.
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 96-96
```mlir
96| // CHECK-LABEL: hoist_above_ext
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: hoist_above_ext anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: hoist_above_ext 这样的标签用于锚定匹配范围。

### Lines 97-97
```mlir
97| tt.func @hoist_above_ext(%arg0: tensor<1024xf16, #layout0>, %arg1: f32) -> tensor<1024xf32, #layout1> {
```
**EN:** This function-oriented block defines or enters `hoist_above_ext`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `hoist_above_ext` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 98-108
```mlir
 98| // CHECK: %[[CVT:.+]] = ttg.convert_layout
 99| // CHECK: arith.extf %[[CVT]]
100| // CHECK-NOT: ttg.convert_layout
101| // CHECK: tt.return
102|   %0 = arith.extf %arg0 : tensor<1024xf16, #layout0> to tensor<1024xf32, #layout0>
103|   %1 = tt.splat %arg1 : f32 -> tensor<1024xf32, #layout0>
104|   %2 = arith.addf %0, %1 : tensor<1024xf32, #layout0>
105|   %3 = ttg.convert_layout %2 : tensor<1024xf32, #layout0> -> tensor<1024xf32, #layout1>
106|   tt.return %3 : tensor<1024xf32, #layout1>
107| }
108| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on arith.extf, broadcasted scalars or pointers, floating-point additions, layout conversions, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 arith.extf、广播后的标量或指针、浮点加法、布局转换、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 109-109
```mlir
109| // CHECK-LABEL: hoist_above_ext2
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: hoist_above_ext2 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: hoist_above_ext2 这样的标签用于锚定匹配范围。

### Lines 110-110
```mlir
110| tt.func @hoist_above_ext2(%arg0: tensor<1024xf16, #layout0>, %arg1: f16) -> tensor<1024xf32, #layout1> {
```
**EN:** This function-oriented block defines or enters `hoist_above_ext2`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `hoist_above_ext2` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 111-123
```mlir
111| // CHECK: %[[CVT:.+]] = ttg.convert_layout
112| // CHECK: arith.extf %[[CVT]]
113| // CHECK-NOT: ttg.convert_layout
114| // CHECK: tt.return
115|   %0 = arith.extf %arg0 : tensor<1024xf16, #layout0> to tensor<1024xf32, #layout0>
116|   %1 = tt.splat %arg1 : f16 -> tensor<1024xf16, #layout0>
117|   %2 = arith.extf %1 : tensor<1024xf16, #layout0> to tensor<1024xf32, #layout0>
118|   %3 = arith.addf %0, %2 : tensor<1024xf32, #layout0>
119|   %4 = ttg.convert_layout %3 : tensor<1024xf32, #layout0> -> tensor<1024xf32, #layout1>
120|   tt.return %4 : tensor<1024xf32, #layout1>
121| }
122| 
123| /// CHECK-LABEL: hoist_above_fptofp
```
**EN:** This operational block carries the concrete IR steps for the test, centered on arith.extf, broadcasted scalars or pointers, floating-point additions, layout conversions, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 arith.extf、广播后的标量或指针、浮点加法、布局转换、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 124-124
```mlir
124| tt.func @hoist_above_fptofp(%arg0: tensor<1024xf8E4M3FNUZ, #layout0>) -> tensor<1024xf32, #layout1> {
```
**EN:** This function-oriented block defines or enters `hoist_above_fptofp`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `hoist_above_fptofp` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 125-134
```mlir
125| // CHECK: %[[CVT:.+]] = ttg.convert_layout
126| // CHECK: tt.fp_to_fp %[[CVT]]
127| // CHECK-NOT: ttg.convert_layout
128| // CHECK: tt.return
129|   %0 = tt.fp_to_fp %arg0, rounding = rtne : tensor<1024xf8E4M3FNUZ, #layout0> -> tensor<1024xf32, #layout0>
130|   %1 = ttg.convert_layout %0 : tensor<1024xf32, #layout0> -> tensor<1024xf32, #layout1>
131|   tt.return %1 : tensor<1024xf32, #layout1>
132| }
133| 
134| /// CHECK-LABEL: dont_hoist_above_trunc_fptofp
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.fp_to_fp, layout conversions, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.fp_to_fp、布局转换、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 135-135
```mlir
135| tt.func @dont_hoist_above_trunc_fptofp(%arg0: tensor<1024xf32, #layout0>) -> tensor<1024xf8E4M3FNUZ, #layout1> {
```
**EN:** This function-oriented block defines or enters `dont_hoist_above_trunc_fptofp`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `dont_hoist_above_trunc_fptofp` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 136-145
```mlir
136| // CHECK-NOT: ttg.convert_layout
137| // CHECK: %[[FP8:.+]] = tt.fp_to_fp
138| // CHECK: ttg.convert_layout %[[FP8]]
139| // CHECK: tt.return
140|   %0 = tt.fp_to_fp %arg0, rounding = rtne : tensor<1024xf32, #layout0> -> tensor<1024xf8E4M3FNUZ, #layout0>
141|   %1 = ttg.convert_layout %0 : tensor<1024xf8E4M3FNUZ, #layout0> -> tensor<1024xf8E4M3FNUZ, #layout1>
142|   tt.return %1 : tensor<1024xf8E4M3FNUZ, #layout1>
143| }
144| 
145| // Hoist the convert on top of broadcast to make it cheaper.
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.fp_to_fp, layout conversions, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.fp_to_fp、布局转换、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 146-146
```mlir
146| // CHECK-LABEL: hoist_above_broadcast
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: hoist_above_broadcast anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: hoist_above_broadcast 这样的标签用于锚定匹配范围。

### Lines 147-147
```mlir
147| tt.func @hoist_above_broadcast(%arg0: tensor<1024x1xf32, #layout2>, %arg1: f32) -> tensor<1024x128xf32, #layout3> {
```
**EN:** This function-oriented block defines or enters `hoist_above_broadcast`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `hoist_above_broadcast` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 148-159
```mlir
148| // CHECK: %[[CVT:.+]] = ttg.convert_layout
149| // CHECK: tt.broadcast %[[CVT]]
150| // CHECK-NOT: ttg.convert_layout
151| // CHECK: tt.return
152|   %0 = tt.broadcast %arg0 : tensor<1024x1xf32, #layout2> -> tensor<1024x128xf32, #layout2>
153|   %1 = tt.splat %arg1 : f32 -> tensor<1024x128xf32, #layout2>
154|   %2 = arith.addf %0, %1 : tensor<1024x128xf32, #layout2>
155|   %3 = ttg.convert_layout %2 : tensor<1024x128xf32, #layout2> -> tensor<1024x128xf32, #layout3>
156|   tt.return %3 : tensor<1024x128xf32, #layout3>
157| }
158| 
159| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor broadcasting, broadcasted scalars or pointers, floating-point additions, layout conversions, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量广播、广播后的标量或指针、浮点加法、布局转换、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 160-160
```mlir
160| // CHECK-LABEL: if
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: if anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: if 这样的标签用于锚定匹配范围。

### Lines 161-161
```mlir
161| tt.func @if(%arg0: i32, %arg1: !tt.ptr<i32> {tt.divisibility = 16 : i32}) {
```
**EN:** This function-oriented block defines or enters `if`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `if` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 162-176
```mlir
162|   // CHECK-NOT: ttg.convert_layout
163|   %c32_i32 = arith.constant dense<32> : tensor<1024xi32, #layout1>
164|   %0 = tt.get_program_id x : i32
165|   %1 = tt.splat %0 : i32 -> tensor<1024xi32, #layout1>
166|   %2 = arith.muli %1, %c32_i32 : tensor<1024xi32, #layout1>
167|   %3 = arith.addi %2, %c32_i32 : tensor<1024xi32, #layout1>
168|   %4 = arith.cmpi sgt, %0, %arg0 : i32
169|   %5 = tt.splat %arg1 : !tt.ptr<i32> -> tensor<1024x!tt.ptr<i32>, #layout0>
170|   scf.if %4 {
171|     %6 = ttg.convert_layout %2 : tensor<1024xi32, #layout1> -> tensor<1024xi32, #layout0>
172|     tt.store %5, %6 : tensor<1024x!tt.ptr<i32>, #layout0>
173|   }
174|   tt.return
175| }
176| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining broadcasted scalars or pointers, constants, program IDs, integer multiplications, integer additions.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 广播后的标量或指针、常量、程序 ID、整数乘法、整数加法。

### Lines 177-177
```mlir
177| // CHECK-LABEL: if_convert_else_not
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: if_convert_else_not anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: if_convert_else_not 这样的标签用于锚定匹配范围。

### Lines 178-192
```mlir
178| tt.func @if_convert_else_not(%arg0: i32, %arg1: !tt.ptr<i32> {tt.divisibility = 16 : i32}) {
179|   %c32_i32 = arith.constant dense<32> : tensor<1024xi32, #layout0>
180|   %0 = tt.get_program_id x : i32
181|   %1 = tt.splat %0 : i32 -> tensor<1024xi32, #layout0>
182|   %9 = tt.splat %0 : i32 -> tensor<1024xi32, #layout1>
183|   %2 = arith.muli %1, %c32_i32 : tensor<1024xi32, #layout0>
184|   %3 = arith.addi %2, %c32_i32 : tensor<1024xi32, #layout0>
185|   %4 = arith.cmpi sgt, %0, %arg0 : i32
186|   %5 = tt.splat %arg1 : !tt.ptr<i32> -> tensor<1024x!tt.ptr<i32>, #layout1>
187|   %8 = scf.if %4 -> tensor<1024xi32, #layout1> {
188|     %6 = ttg.convert_layout %2 : tensor<1024xi32, #layout0> -> tensor<1024xi32, #layout1>
189|     scf.yield %6 : tensor<1024xi32, #layout1>
190|   } else {
191|     scf.yield %9 : tensor<1024xi32, #layout1>
192|   }
```
**EN:** This function-oriented block defines or enters `if_convert_else_not`. Within it, the test exercises broadcasted scalars or pointers, tt.func, loop/if yielded values, constants, program IDs, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `if_convert_else_not` 为核心。测试在其中演示 广播后的标量或指针、tt.func、循环/分支产出值、常量、程序 ID，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 193-197
```mlir
193|   // CHECK-NOT: ttg.convert_layout
194|   tt.store %5, %8 : tensor<1024x!tt.ptr<i32>, #layout1>
195|   tt.return
196| }
197| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized stores, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的存储、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 198-198
```mlir
198| // CHECK-LABEL: if_not_else_convert
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: if_not_else_convert anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: if_not_else_convert 这样的标签用于锚定匹配范围。

### Lines 199-213
```mlir
199| tt.func @if_not_else_convert(%arg0: i32, %arg1: !tt.ptr<i32> {tt.divisibility = 16 : i32}) {
200|   %c32_i32 = arith.constant dense<32> : tensor<1024xi32, #layout0>
201|   %0 = tt.get_program_id x : i32
202|   %1 = tt.splat %0 : i32 -> tensor<1024xi32, #layout0>
203|   %9 = tt.splat %0 : i32 -> tensor<1024xi32, #layout1>
204|   %2 = arith.muli %1, %c32_i32 : tensor<1024xi32, #layout0>
205|   %3 = arith.addi %2, %c32_i32 : tensor<1024xi32, #layout0>
206|   %4 = arith.cmpi sgt, %0, %arg0 : i32
207|   %5 = tt.splat %arg1 : !tt.ptr<i32> -> tensor<1024x!tt.ptr<i32>, #layout1>
208|   %8 = scf.if %4 -> tensor<1024xi32, #layout1> {
209|     scf.yield %9 : tensor<1024xi32, #layout1>
210|   } else {
211|     %7 = ttg.convert_layout %3 : tensor<1024xi32, #layout0> -> tensor<1024xi32, #layout1>
212|     scf.yield %7 : tensor<1024xi32, #layout1>
213|   }
```
**EN:** This function-oriented block defines or enters `if_not_else_convert`. Within it, the test exercises broadcasted scalars or pointers, tt.func, loop/if yielded values, constants, program IDs, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `if_not_else_convert` 为核心。测试在其中演示 广播后的标量或指针、tt.func、循环/分支产出值、常量、程序 ID，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 214-218
```mlir
214|   // CHECK-NOT: ttg.convert_layout
215|   tt.store %5, %8 : tensor<1024x!tt.ptr<i32>, #layout1>
216|   tt.return
217| }
218| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized stores, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的存储、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 219-219
```mlir
219| // CHECK-LABEL: if_else_both_convert
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: if_else_both_convert anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: if_else_both_convert 这样的标签用于锚定匹配范围。

### Lines 220-236
```mlir
220| tt.func @if_else_both_convert(%arg0: i32, %arg1: !tt.ptr<i32> {tt.divisibility = 16 : i32}) {
221|   %c32_i32 = arith.constant dense<32> : tensor<1024xi32, #layout0>
222|   %0 = tt.get_program_id x : i32
223|   %1 = tt.splat %0 : i32 -> tensor<1024xi32, #layout0>
224|   %2 = arith.muli %1, %c32_i32 : tensor<1024xi32, #layout0>
225|   %3 = arith.addi %2, %c32_i32 : tensor<1024xi32, #layout0>
226|   %4 = arith.cmpi sgt, %0, %arg0 : i32
227|   %5 = tt.splat %arg1 : !tt.ptr<i32> -> tensor<1024x!tt.ptr<i32>, #layout1>
228|   %8 = scf.if %4 -> tensor<1024xi32, #layout1> {
229|     %6 = ttg.convert_layout %2 : tensor<1024xi32, #layout0> -> tensor<1024xi32, #layout1>
230|     scf.yield %6 : tensor<1024xi32, #layout1>
231|   } else {
232|     %7 = ttg.convert_layout %3 : tensor<1024xi32, #layout0> -> tensor<1024xi32, #layout1>
233|     scf.yield %7 : tensor<1024xi32, #layout1>
234|   }
235|   // TODO(csigg): seems like the whole function is converted to layout1.
236|   // disabledCHECK: ttg.convert_layout
```
**EN:** This function-oriented block defines or enters `if_else_both_convert`. Within it, the test exercises tt.func, broadcasted scalars or pointers, layout conversions, loop/if yielded values, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `if_else_both_convert` 为核心。测试在其中演示 tt.func、广播后的标量或指针、布局转换、循环/分支产出值、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 237-243
```mlir
237|   // CHECK-NOT: ttg.convert_layout
238|   tt.store %5, %8 : tensor<1024x!tt.ptr<i32>, #layout1>
239|   tt.return
240| }
241| 
242| }
243| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized stores, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的存储、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 244-244
```mlir
244| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 245-256
```mlir
245| 
246| #blocked0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
247| #blocked0a = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
248| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
249| #slice1dim1 = #ttg.slice<{dim = 1, parent = #blocked1}>
250| #blocked2 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [0, 1]}>
251| #blocked2a = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
252| #slice2dim0 = #ttg.slice<{dim = 0, parent = #blocked2}>
253| #blocked3 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [2, 16], warpsPerCTA = [1, 4], order = [1, 0]}>
254| #blocked4 = #ttg.blocked<{sizePerThread = [4, 1], threadsPerWarp = [16, 2], warpsPerCTA = [4, 1], order = [0, 1]}>
255| #blocked5 = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
256| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 257-261
```mlir
257| // CHECK-DAG: [[$row_layout:#.*]] = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [2, 16], warpsPerCTA = [1, 4], order = [1, 0]}>
258| // CHECK-DAG: [[$col_layout:#.*]] = #ttg.blocked<{sizePerThread = [4, 1], threadsPerWarp = [16, 2], warpsPerCTA = [4, 1], order = [0, 1]}>
259| // CHECK-DAG: [[$col_layout_novec:#.*]] = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
260| 
261| // CHECK-LABEL: @transpose
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @transpose anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @transpose 这样的标签用于锚定匹配范围。

### Lines 262-262
```mlir
262| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 263-263
```mlir
263| tt.func @transpose(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg1: i32 {tt.divisibility = 16 : i32}, %arg2: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg3: i32 {tt.divisibility = 16 : i32}) {
```
**EN:** This function-oriented block defines or enters `transpose`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `transpose` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 264-281
```mlir
264|   // CHECK-NOT: ttg.convert_layout
265|   // CHECK: [[loaded_val:%.*]] = tt.load {{.*}}, {{%cst.*}}, {{%cst.*}} : tensor<64x64x!tt.ptr<f32>, [[$row_layout]]>
266|   // CHECK: [[cvt_val:%.*]] = ttg.convert_layout [[loaded_val]] : tensor<64x64xf32, [[$row_layout]]> -> tensor<64x64xf32, [[$col_layout]]>
267|   // CHECK: tt.store {{.*}}, [[cvt_val]], {{%cst.*}} : tensor<64x64x!tt.ptr<f32>, [[$col_layout]]>
268|   // CHECK: tt.return
269|   %cst = arith.constant dense<0.000000e+00> : tensor<64x64xf32, #blocked1>
270|   %cst_0 = arith.constant dense<true> : tensor<64x64xi1, #blocked1>
271|   %00 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #slice1dim1>
272|   %01 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #slice2dim0>
273|   %1 = tt.expand_dims %00 {axis = 1 : i32} : tensor<64xi32, #slice1dim1> -> tensor<64x1xi32, #blocked1>
274|   %2 = tt.splat %arg1 : i32 -> tensor<64x1xi32, #blocked1>
275|   %3 = arith.muli %1, %2 : tensor<64x1xi32, #blocked1>
276|   %4 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<64x1x!tt.ptr<f32>, #blocked1>
277|   %5 = tt.addptr %4, %3 : tensor<64x1x!tt.ptr<f32>, #blocked1>, tensor<64x1xi32, #blocked1>
278|   %6 = tt.expand_dims %01 {axis = 0 : i32} : tensor<64xi32, #slice2dim0> -> tensor<1x64xi32, #blocked2>
279|   %7 = tt.broadcast %5 : tensor<64x1x!tt.ptr<f32>, #blocked1> -> tensor<64x64x!tt.ptr<f32>, #blocked1>
280|   %8 = tt.broadcast %6 : tensor<1x64xi32, #blocked2> -> tensor<64x64xi32, #blocked2>
281|   %9 = ttg.convert_layout %8 : tensor<64x64xi32, #blocked2> -> tensor<64x64xi32, #blocked1>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants, lane/block index ranges, shape expansion, broadcasted scalars or pointers, tensor broadcasting. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量、lane/block 索引范围、形状扩展、广播后的标量或指针、张量广播。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 282-299
```mlir
282|   %10 = tt.addptr %7, %9 : tensor<64x64x!tt.ptr<f32>, #blocked1>, tensor<64x64xi32, #blocked1>
283|   %11 = tt.splat %arg2 : !tt.ptr<f32> -> tensor<64x1x!tt.ptr<f32>, #blocked1>
284|   %12 = tt.addptr %11, %1 : tensor<64x1x!tt.ptr<f32>, #blocked1>, tensor<64x1xi32, #blocked1>
285|   %13 = tt.splat %arg3 : i32 -> tensor<1x64xi32, #blocked2>
286|   %14 = arith.muli %6, %13 : tensor<1x64xi32, #blocked2>
287|   %15 = tt.broadcast %12 : tensor<64x1x!tt.ptr<f32>, #blocked1> -> tensor<64x64x!tt.ptr<f32>, #blocked1>
288|   %16 = tt.broadcast %14 : tensor<1x64xi32, #blocked2> -> tensor<64x64xi32, #blocked2>
289|   %17 = ttg.convert_layout %16 : tensor<64x64xi32, #blocked2> -> tensor<64x64xi32, #blocked1>
290|   %18 = tt.addptr %15, %17 : tensor<64x64x!tt.ptr<f32>, #blocked1>, tensor<64x64xi32, #blocked1>
291|   %19 = ttg.convert_layout %10 : tensor<64x64x!tt.ptr<f32>, #blocked1> -> tensor<64x64x!tt.ptr<f32>, #blocked3>
292|   %20 = ttg.convert_layout %cst_0 : tensor<64x64xi1, #blocked1> -> tensor<64x64xi1, #blocked3>
293|   %21 = ttg.convert_layout %cst : tensor<64x64xf32, #blocked1> -> tensor<64x64xf32, #blocked3>
294|   %22 = tt.load %19, %20, %21 : tensor<64x64x!tt.ptr<f32>, #blocked3>
295|   %23 = ttg.convert_layout %22 : tensor<64x64xf32, #blocked3> -> tensor<64x64xf32, #blocked1>
296|   %24 = ttg.convert_layout %18 : tensor<64x64x!tt.ptr<f32>, #blocked1> -> tensor<64x64x!tt.ptr<f32>, #blocked4>
297|   %25 = ttg.convert_layout %23 : tensor<64x64xf32, #blocked1> -> tensor<64x64xf32, #blocked4>
298|   %26 = ttg.convert_layout %cst_0 : tensor<64x64xi1, #blocked1> -> tensor<64x64xi1, #blocked4>
299|   tt.store %24, %25, %26 : tensor<64x64x!tt.ptr<f32>, #blocked4>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, pointer arithmetic, broadcasted scalars or pointers, tensor broadcasting, integer multiplications. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、指针算术、广播后的标量或指针、张量广播、整数乘法。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 300-303
```mlir
300|   tt.return
301| }
302| }
303| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 304-304
```mlir
304| // CHECK-LABEL: loop
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: loop anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: loop 这样的标签用于锚定匹配范围。

### Lines 305-305
```mlir
305| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 306-306
```mlir
306| tt.func @loop(%arg0: !tt.ptr<f32>, %arg1: i32, %arg2: !tt.ptr<f32>, %arg3: i32, %arg4: i32) {
```
**EN:** This function-oriented block defines or enters `loop`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `loop` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 307-315
```mlir
307|   // CHECK-NOT: ttg.convert_layout
308|   // CHECK: [[loop_ret:%.*]]:2 = scf.for {{.*}} -> (tensor<64x64xf32, [[$row_layout]]>, tensor<64x64x!tt.ptr<f32>, [[$row_layout]]>)
309|   // CHECK-NEXT: {{.*}} = tt.load {{.*}} : tensor<64x64x!tt.ptr<f32>, [[$row_layout]]>
310|   // CHECK-NEXT: {{.*}} = arith.addf {{.*}} : tensor<64x64xf32, [[$row_layout]]>
311|   // CHECK-NEXT: {{.*}} = tt.addptr {{.*}} : tensor<64x64x!tt.ptr<f32>, [[$row_layout]]>, tensor<64x64xi32, [[$row_layout]]>
312|   // CHECK-NEXT: scf.yield {{.*}} : tensor<64x64xf32, [[$row_layout]]>, tensor<64x64x!tt.ptr<f32>, [[$row_layout]]>
313|   // CHECK-NEXT: }
314|   // CHECK-NOT: ttg.convert_layout
315|   //     CHECK: {{.*}} = ttg.convert_layout [[loop_ret]]#0 : tensor<64x64xf32, [[$row_layout]]> -> tensor<64x64xf32, [[$col_layout_novec]]>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 316-333
```mlir
316|   // CHECK-NOT: ttg.convert_layout
317|   //    CHECK:  tt.return
318|   %cst = arith.constant dense<true> : tensor<64x64xi1, #blocked1>
319|   %cst_0 = arith.constant dense<64> : tensor<64x64xi32, #blocked1>
320|   %c1 = arith.constant 1 : index
321|   %c32 = arith.constant 32 : index
322|   %c0 = arith.constant 0 : index
323|   %cst_1 = arith.constant dense<0.000000e+00> : tensor<64x64xf32, #blocked1>
324|   %00 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #slice1dim1>
325|   %01 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #slice2dim0>
326|   %1 = tt.expand_dims %00 {axis = 1 : i32} : tensor<64xi32, #slice1dim1> -> tensor<64x1xi32, #blocked1>
327|   %2 = tt.splat %arg1 : i32 -> tensor<64x1xi32, #blocked1>
328|   %3 = arith.muli %1, %2 : tensor<64x1xi32, #blocked1>
329|   %4 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<64x1x!tt.ptr<f32>, #blocked1>
330|   %5 = tt.addptr %4, %3 : tensor<64x1x!tt.ptr<f32>, #blocked1>, tensor<64x1xi32, #blocked1>
331|   %6 = tt.expand_dims %01 {axis = 0 : i32} : tensor<64xi32, #slice2dim0> -> tensor<1x64xi32, #blocked2>
332|   %7 = tt.broadcast %5 : tensor<64x1x!tt.ptr<f32>, #blocked1> -> tensor<64x64x!tt.ptr<f32>, #blocked1>
333|   %8 = tt.broadcast %6 : tensor<1x64xi32, #blocked2> -> tensor<64x64xi32, #blocked2>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants, lane/block index ranges, shape expansion, broadcasted scalars or pointers, tensor broadcasting. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量、lane/block 索引范围、形状扩展、广播后的标量或指针、张量广播。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 334-351
```mlir
334|   %9 = ttg.convert_layout %8 : tensor<64x64xi32, #blocked2> -> tensor<64x64xi32, #blocked1>
335|   %10 = tt.addptr %7, %9 : tensor<64x64x!tt.ptr<f32>, #blocked1>, tensor<64x64xi32, #blocked1>
336|   %11:2 = scf.for %arg5 = %c0 to %c32 step %c1 iter_args(%arg6 = %cst_1, %arg7 = %10) -> (tensor<64x64xf32, #blocked1>, tensor<64x64x!tt.ptr<f32>, #blocked1>) {
337|     %23 = ttg.convert_layout %arg7 : tensor<64x64x!tt.ptr<f32>, #blocked1> -> tensor<64x64x!tt.ptr<f32>, #blocked3>
338|     %24 = ttg.convert_layout %cst : tensor<64x64xi1, #blocked1> -> tensor<64x64xi1, #blocked3>
339|     %25 = ttg.convert_layout %cst_1 : tensor<64x64xf32, #blocked1> -> tensor<64x64xf32, #blocked3>
340|     %26 = tt.load %23, %24, %25 : tensor<64x64x!tt.ptr<f32>, #blocked3>
341|     %27 = ttg.convert_layout %26 : tensor<64x64xf32, #blocked3> -> tensor<64x64xf32, #blocked1>
342|     %28 = arith.addf %arg6, %27 : tensor<64x64xf32, #blocked1>
343|     %29 = tt.addptr %arg7, %cst_0 : tensor<64x64x!tt.ptr<f32>, #blocked1>, tensor<64x64xi32, #blocked1>
344|     scf.yield %28, %29 : tensor<64x64xf32, #blocked1>, tensor<64x64x!tt.ptr<f32>, #blocked1>
345|   }
346|   %12 = tt.splat %arg2 : !tt.ptr<f32> -> tensor<64x1x!tt.ptr<f32>, #blocked1>
347|   %13 = tt.addptr %12, %1 : tensor<64x1x!tt.ptr<f32>, #blocked1>, tensor<64x1xi32, #blocked1>
348|   %14 = tt.splat %arg3 : i32 -> tensor<1x64xi32, #blocked2>
349|   %15 = arith.muli %6, %14 : tensor<1x64xi32, #blocked2>
350|   %16 = tt.broadcast %13 : tensor<64x1x!tt.ptr<f32>, #blocked1> -> tensor<64x64x!tt.ptr<f32>, #blocked1>
351|   %17 = tt.broadcast %15 : tensor<1x64xi32, #blocked2> -> tensor<64x64xi32, #blocked2>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining layout conversions, pointer arithmetic, broadcasted scalars or pointers, tensor broadcasting, structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 布局转换、指针算术、广播后的标量或指针、张量广播、结构化循环。

### Lines 352-361
```mlir
352|   %18 = ttg.convert_layout %17 : tensor<64x64xi32, #blocked2> -> tensor<64x64xi32, #blocked1>
353|   %19 = tt.addptr %16, %18 : tensor<64x64x!tt.ptr<f32>, #blocked1>, tensor<64x64xi32, #blocked1>
354|   %20 = ttg.convert_layout %19 : tensor<64x64x!tt.ptr<f32>, #blocked1> -> tensor<64x64x!tt.ptr<f32>, #blocked1>
355|   %21 = ttg.convert_layout %11#0 : tensor<64x64xf32, #blocked1> -> tensor<64x64xf32, #blocked1>
356|   %22 = ttg.convert_layout %cst : tensor<64x64xi1, #blocked1> -> tensor<64x64xi1, #blocked1>
357|   tt.store %20, %21, %22 : tensor<64x64x!tt.ptr<f32>, #blocked1>
358|   tt.return
359| }
360| }
361| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, pointer arithmetic, masked or vectorized stores, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、指针算术、带掩码或向量化的存储、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 362-364
```mlir
362| // CHECK-LABEL: loop_if
363| // CHECK-NOT: ttg.convert_layout
364| //     CHECK: scf.for
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 365-366
```mlir
365| // CHECK-NOT: ttg.convert_layout
366| //     CHECK:   scf.if
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 367-370
```mlir
367| // CHECK-NOT: ttg.convert_layout
368| //     CHECK:     scf.yield
369| //     CHECK:   else
370| //     CHECK:     scf.yield
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 371-373
```mlir
371| // CHECK-NOT: ttg.convert_layout
372| //     CHECK:   scf.yield
373| //     CHECK: ttg.convert_layout
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 374-375
```mlir
374| // CHECK-NOT: ttg.convert_layout
375| //     CHECK: tt.store
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 376-376
```mlir
376| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 377-394
```mlir
377| tt.func @loop_if(%arg0: !tt.ptr<f32>, %arg1: i32, %arg2: !tt.ptr<f32>, %arg3: i32, %arg4: i32) {
378|   %cst = arith.constant dense<true> : tensor<64x64xi1, #blocked1>
379|   %cst_0 = arith.constant dense<64> : tensor<64x64xi32, #blocked1>
380|   %c1 = arith.constant 1 : index
381|   %c32 = arith.constant 32 : index
382|   %c0 = arith.constant 0 : index
383|   %i0 = arith.constant 0 : i32
384|   %cst_1 = arith.constant dense<0.000000e+00> : tensor<64x64xf32, #blocked1>
385|   %00 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #slice1dim1>
386|   %01 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #slice2dim0>
387|   %1 = tt.expand_dims %00 {axis = 1 : i32} : tensor<64xi32, #slice1dim1> -> tensor<64x1xi32, #blocked1>
388|   %2 = tt.splat %arg1 : i32 -> tensor<64x1xi32, #blocked1>
389|   %3 = arith.muli %1, %2 : tensor<64x1xi32, #blocked1>
390|   %4 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<64x1x!tt.ptr<f32>, #blocked1>
391|   %5 = tt.addptr %4, %3 : tensor<64x1x!tt.ptr<f32>, #blocked1>, tensor<64x1xi32, #blocked1>
392|   %6 = tt.expand_dims %01 {axis = 0 : i32} : tensor<64xi32, #slice2dim0> -> tensor<1x64xi32, #blocked2>
393|   %7 = tt.broadcast %5 : tensor<64x1x!tt.ptr<f32>, #blocked1> -> tensor<64x64x!tt.ptr<f32>, #blocked1>
394|   %8 = tt.broadcast %6 : tensor<1x64xi32, #blocked2> -> tensor<64x64xi32, #blocked2>
```
**EN:** This function-oriented block defines or enters `loop_if`. Within it, the test exercises constants, tt.func, lane/block index ranges, shape expansion, broadcasted scalars or pointers, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `loop_if` 为核心。测试在其中演示 常量、tt.func、lane/block 索引范围、形状扩展、广播后的标量或指针，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 395-412
```mlir
395|   %9 = ttg.convert_layout %8 : tensor<64x64xi32, #blocked2> -> tensor<64x64xi32, #blocked1>
396|   %10 = tt.addptr %7, %9 : tensor<64x64x!tt.ptr<f32>, #blocked1>, tensor<64x64xi32, #blocked1>
397|   %11:2 = scf.for %arg5 = %c0 to %c32 step %c1 iter_args(%arg6 = %cst_1, %arg7 = %10) -> (tensor<64x64xf32, #blocked1>, tensor<64x64x!tt.ptr<f32>, #blocked1>) {
398|     %33 = arith.cmpi "sgt", %arg5, %c0 : index
399|     %34 = scf.if %33 -> (tensor<64x64xf32, #blocked1>) {
400|       %23 = ttg.convert_layout %arg7 : tensor<64x64x!tt.ptr<f32>, #blocked1> -> tensor<64x64x!tt.ptr<f32>, #blocked3>
401|       %24 = ttg.convert_layout %cst : tensor<64x64xi1, #blocked1> -> tensor<64x64xi1, #blocked3>
402|       %25 = ttg.convert_layout %cst_1 : tensor<64x64xf32, #blocked1> -> tensor<64x64xf32, #blocked3>
403|       %26 = tt.load %23, %24, %25 : tensor<64x64x!tt.ptr<f32>, #blocked3>
404|       %27 = ttg.convert_layout %26 : tensor<64x64xf32, #blocked3> -> tensor<64x64xf32, #blocked1>
405|       scf.yield %27 : tensor<64x64xf32, #blocked1>
406|     } else {
407|       scf.yield %arg6 : tensor<64x64xf32, #blocked1>
408|     }
409|     %28 = arith.addf %arg6, %34 : tensor<64x64xf32, #blocked1>
410|     %29 = tt.addptr %arg7, %cst_0 : tensor<64x64x!tt.ptr<f32>, #blocked1>, tensor<64x64xi32, #blocked1>
411|     scf.yield %28, %29 : tensor<64x64xf32, #blocked1>, tensor<64x64x!tt.ptr<f32>, #blocked1>
412|   }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining layout conversions, loop/if yielded values, pointer arithmetic, structured loops, integer comparisons.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 布局转换、循环/分支产出值、指针算术、结构化循环、整数比较。

### Lines 413-428
```mlir
413|   %12 = tt.splat %arg2 : !tt.ptr<f32> -> tensor<64x1x!tt.ptr<f32>, #blocked1>
414|   %13 = tt.addptr %12, %1 : tensor<64x1x!tt.ptr<f32>, #blocked1>, tensor<64x1xi32, #blocked1>
415|   %14 = tt.splat %arg3 : i32 -> tensor<1x64xi32, #blocked2>
416|   %15 = arith.muli %6, %14 : tensor<1x64xi32, #blocked2>
417|   %16 = tt.broadcast %13 : tensor<64x1x!tt.ptr<f32>, #blocked1> -> tensor<64x64x!tt.ptr<f32>, #blocked1>
418|   %17 = tt.broadcast %15 : tensor<1x64xi32, #blocked2> -> tensor<64x64xi32, #blocked2>
419|   %18 = ttg.convert_layout %17 : tensor<64x64xi32, #blocked2> -> tensor<64x64xi32, #blocked1>
420|   %19 = tt.addptr %16, %18 : tensor<64x64x!tt.ptr<f32>, #blocked1>, tensor<64x64xi32, #blocked1>
421|   %20 = ttg.convert_layout %19 : tensor<64x64x!tt.ptr<f32>, #blocked1> -> tensor<64x64x!tt.ptr<f32>, #blocked1>
422|   %21 = ttg.convert_layout %11#0 : tensor<64x64xf32, #blocked1> -> tensor<64x64xf32, #blocked1>
423|   %22 = ttg.convert_layout %cst : tensor<64x64xi1, #blocked1> -> tensor<64x64xi1, #blocked1>
424|   tt.store %20, %21, %22 : tensor<64x64x!tt.ptr<f32>, #blocked1>
425|   tt.return
426| }
427| }
428| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, broadcasted scalars or pointers, pointer arithmetic, tensor broadcasting, integer multiplications. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、广播后的标量或指针、指针算术、张量广播、整数乘法。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 429-429
```mlir
429| // CHECK-LABEL: vecadd
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: vecadd anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: vecadd 这样的标签用于锚定匹配范围。

### Lines 430-430
```mlir
430| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 431-431
```mlir
431| tt.func @vecadd(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg2: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg3: i32) {
```
**EN:** This function-oriented block defines or enters `vecadd`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `vecadd` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 432-449
```mlir
432|   // CHECK-NOT: ttg.convert_layout
433|   %c256_i32 = arith.constant 256 : i32
434|   %0 = tt.get_program_id x : i32
435|   %1 = arith.muli %0, %c256_i32 : i32
436|   %2 = tt.splat %1 : i32 -> tensor<256xi32, #blocked5>
437|   %3 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #blocked5>
438|   %4 = tt.splat %1 : i32 -> tensor<256xi32, #blocked5>
439|   %5 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #blocked5>
440|   %6 = tt.splat %1 : i32 -> tensor<256xi32, #blocked5>
441|   %7 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #blocked5>
442|   %8 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<256x!tt.ptr<f32>, #blocked5>
443|   %9 = arith.addi %6, %7 : tensor<256xi32, #blocked5>
444|   %10 = tt.splat %arg1 : !tt.ptr<f32> -> tensor<256x!tt.ptr<f32>, #blocked5>
445|   %11 = arith.addi %4, %5 : tensor<256xi32, #blocked5>
446|   %12 = tt.addptr %8, %9 : tensor<256x!tt.ptr<f32>, #blocked5>, tensor<256xi32, #blocked5>
447|   %13 = tt.load %12 : tensor<256x!tt.ptr<f32>, #blocked5>
448|   %14 = ttg.convert_layout %13 : tensor<256xf32, #blocked5> -> tensor<256xf32, #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>>
449|   %15 = tt.addptr %10, %11 : tensor<256x!tt.ptr<f32>, #blocked5>, tensor<256xi32, #blocked5>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on broadcasted scalars or pointers, lane/block index ranges, integer additions, pointer arithmetic, constants. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 广播后的标量或指针、lane/block 索引范围、整数加法、指针算术、常量。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 450-462
```mlir
450|   %16 = tt.load %15 : tensor<256x!tt.ptr<f32>, #blocked5>
451|   %17 = ttg.convert_layout %16 : tensor<256xf32, #blocked5> -> tensor<256xf32, #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>>
452|   %18 = arith.addf %14, %17 : tensor<256xf32, #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>>
453|   %19 = tt.splat %arg2 : !tt.ptr<f32> -> tensor<256x!tt.ptr<f32>, #blocked5>
454|   %20 = arith.addi %2, %3 : tensor<256xi32, #blocked5>
455|   %21 = tt.addptr %19, %20 : tensor<256x!tt.ptr<f32>, #blocked5>, tensor<256xi32, #blocked5>
456|   %22 = ttg.convert_layout %18 : tensor<256xf32, #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>> -> tensor<256xf32, #blocked5>
457|   tt.store %21, %22 : tensor<256x!tt.ptr<f32>, #blocked5>
458|   tt.return
459| }
460| }
461| 
462| // Select has args with different element types
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, masked or vectorized loads, floating-point additions, broadcasted scalars or pointers, integer additions. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、带掩码或向量化的加载、浮点加法、广播后的标量或指针、整数加法。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 463-463
```mlir
463| // CHECK-LABEL: select
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: select anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: select 这样的标签用于锚定匹配范围。

### Lines 464-464
```mlir
464| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 465-465
```mlir
465| tt.func @select(%arg0: !tt.ptr<f64> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f64> {tt.divisibility = 16 : i32}, %arg2: i32 {tt.divisibility = 16 : i32}) {
```
**EN:** This function-oriented block defines or enters `select`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `select` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 466-483
```mlir
466|   // CHECK-NOT: ttg.convert_layout
467|   %cst = arith.constant dense<30000> : tensor<1x1xi32, #blocked2>
468|   %cst_0 = arith.constant dense<30000> : tensor<1x512xi32, #blocked2>
469|   %c512 = arith.constant 512 : i32
470|   %c30000 = arith.constant 30000 : i32
471|   %c0 = arith.constant 0 : i32
472|   %cst_1 = arith.constant dense<2048> : tensor<1x1xi32, #blocked2>
473|   %cst_2 = arith.constant dense<0.000000e+00> : tensor<1x512xf64, #blocked2>
474|   %0 = tt.get_program_id x : i32
475|   %1 = tt.make_range {end = 1 : i32, start = 0 : i32} : tensor<1xi32, #blocked0>
476|   %2 = ttg.convert_layout %1 : tensor<1xi32, #blocked0> -> tensor<1xi32, #ttg.slice<{dim = 1, parent = #blocked1}>>
477|   %3 = tt.expand_dims %2 {axis = 1 : i32} : tensor<1xi32, #ttg.slice<{dim = 1, parent = #blocked1}>> -> tensor<1x1xi32, #blocked1>
478|   %4 = ttg.convert_layout %3 : tensor<1x1xi32, #blocked1> -> tensor<1x1xi32, #blocked2>
479|   %5 = tt.splat %0 : i32 -> tensor<1x1xi32, #blocked2>
480|   %6 = arith.addi %5, %4 : tensor<1x1xi32, #blocked2>
481|   %7 = arith.cmpi "slt", %6, %cst_1 : tensor<1x1xi32, #blocked2>
482|   %8 = tt.make_range {end = 512 : i32, start = 0 : i32} : tensor<512xi32, #blocked0>
483|   %9 = ttg.convert_layout %8 : tensor<512xi32, #blocked0> -> tensor<512xi32, #ttg.slice<{dim = 0, parent = #blocked2}>>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants, layout conversions, lane/block index ranges, program IDs, shape expansion. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量、布局转换、lane/block 索引范围、程序 ID、形状扩展。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 484-501
```mlir
484|   %10 = tt.expand_dims %9 {axis = 0 : i32} : tensor<512xi32, #ttg.slice<{dim = 0, parent = #blocked2}>> -> tensor<1x512xi32, #blocked2>
485|   %11 = arith.muli %6, %cst : tensor<1x1xi32, #blocked2>
486|   %12 = tt.broadcast %11 : tensor<1x1xi32, #blocked2> -> tensor<1x512xi32, #blocked2>
487|   %13 = tt.splat %arg0 : !tt.ptr<f64> -> tensor<1x512x!tt.ptr<f64>, #blocked2>
488|   %14 = tt.broadcast %7 : tensor<1x1xi1, #blocked2> -> tensor<1x512xi1, #blocked2>
489|   %15 = scf.for %arg3 = %c0 to %c30000 step %c512 iter_args(%arg4 = %cst_2) -> (tensor<1x512xf64, #blocked2>) : i32 {
490|     %17 = tt.splat %arg3 : i32 -> tensor<1x512xi32, #blocked2>
491|     %18 = arith.addi %17, %10 : tensor<1x512xi32, #blocked2>
492|     %19 = arith.cmpi "slt", %18, %cst_0 : tensor<1x512xi32, #blocked2>
493|     %20 = arith.addi %18, %12 : tensor<1x512xi32, #blocked2>
494|     %21 = tt.addptr %13, %20 : tensor<1x512x!tt.ptr<f64>, #blocked2>, tensor<1x512xi32, #blocked2>
495|     %22 = arith.andi %19, %14 : tensor<1x512xi1, #blocked2>
496|     %23 = ttg.convert_layout %21 : tensor<1x512x!tt.ptr<f64>, #blocked2> -> tensor<1x512x!tt.ptr<f64>, #blocked3>
497|     %24 = ttg.convert_layout %22 : tensor<1x512xi1, #blocked2> -> tensor<1x512xi1, #blocked3>
498|     %25 = tt.load %23, %24 : tensor<1x512x!tt.ptr<f64>, #blocked3>
499|     %26 = ttg.convert_layout %25 : tensor<1x512xf64, #blocked3> -> tensor<1x512xf64, #blocked2>
500|     %27 = arith.andi %14, %19 : tensor<1x512xi1, #blocked2>
501|     %28 = arith.cmpf "olt", %arg4, %26 : tensor<1x512xf64, #blocked2>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining layout conversions, tensor broadcasting, broadcasted scalars or pointers, integer additions, arith.andi.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 布局转换、张量广播、广播后的标量或指针、整数加法、arith.andi。

### Lines 502-514
```mlir
502|     %29 = arith.andi %27, %28 : tensor<1x512xi1, #blocked2>
503|     %30 = arith.select %29, %26, %arg4 : tensor<1x512xi1, #blocked2>, tensor<1x512xf64, #blocked2>
504|     %31 = ttg.convert_layout %21 : tensor<1x512x!tt.ptr<f64>, #blocked2> -> tensor<1x512x!tt.ptr<f64>, #blocked3>
505|     %32 = ttg.convert_layout %30 : tensor<1x512xf64, #blocked2> -> tensor<1x512xf64, #blocked3>
506|     %33 = ttg.convert_layout %27 : tensor<1x512xi1, #blocked2> -> tensor<1x512xi1, #blocked3>
507|     tt.store %31, %32, %33 : tensor<1x512x!tt.ptr<f64>, #blocked3>
508|     scf.yield %30 : tensor<1x512xf64, #blocked2>
509|   }
510|   tt.return
511| }
512| }
513| 
514| // Make sure the following IR doesn't hang the compiler.
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining layout conversions, arith.andi, arith.select, masked or vectorized stores, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 布局转换、arith.andi、arith.select、带掩码或向量化的存储、循环/分支产出值。

### Lines 515-515
```mlir
515| // CHECK-LABEL: long_func
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: long_func anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: long_func 这样的标签用于锚定匹配范围。

### Lines 516-516
```mlir
516| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 517-534
```mlir
517| tt.func public @long_func(%arg0: !tt.ptr<i64> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<i64> {tt.divisibility = 16 : i32}, %arg2: !tt.ptr<i64> {tt.divisibility = 16 : i32}, %arg3: !tt.ptr<i64> {tt.divisibility = 16 : i32}, %arg4: !tt.ptr<i64> {tt.divisibility = 16 : i32}, %arg5: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg6: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg7: !tt.ptr<i64> {tt.divisibility = 16 : i32}, %arg8: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg9: !tt.ptr<f64> {tt.divisibility = 16 : i32}, %arg10: !tt.ptr<f64> {tt.divisibility = 16 : i32}, %arg11: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg12: !tt.ptr<i32> {tt.divisibility = 16 : i32}, %arg13: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg14: !tt.ptr<f64> {tt.divisibility = 16 : i32}, %arg15: !tt.ptr<f64> {tt.divisibility = 16 : i32}, %arg16: i32 {tt.divisibility = 16 : i32}) {
518|   %cst = arith.constant dense<1.000000e+00> : tensor<1024xf32, #blocked0>
519|   %cst_0 = arith.constant dense<5.000000e-04> : tensor<1024xf32, #blocked0>
520|   %cst_1 = arith.constant dense<0.999499976> : tensor<1024xf32, #blocked0>
521|   %cst_2 = arith.constant dense<1.000000e+04> : tensor<1024xf32, #blocked0>
522|   %cst_3 = arith.constant dense<5000> : tensor<1024xi32, #blocked0>
523|   %cst_4 = arith.constant dense<150> : tensor<1024xi32, #blocked0>
524|   %cst_5 = arith.constant dense<false> : tensor<1024xi1, #blocked0>
525|   %cst_6 = arith.constant dense<2> : tensor<1024xi32, #blocked0>
526|   %cst_7 = arith.constant dense<4999> : tensor<1024xi32, #blocked0>
527|   %cst_8 = arith.constant dense<2499> : tensor<1024xi32, #blocked0>
528|   %cst_9 = arith.constant dense<2500> : tensor<1024xi32, #blocked0>
529|   %cst_10 = arith.constant dense<0.91629076> : tensor<1024xf32, #blocked0>
530|   %c2499_i32 = arith.constant 2499 : i32
531|   %cst_11 = arith.constant dense<1024> : tensor<1024xi32, #blocked0>
532|   %c1024_i32 = arith.constant 1024 : i32
533|   %cst_12 = arith.constant dense<1> : tensor<1024xi32, #blocked0>
534|   %cst_13 = arith.constant dense<0.000000e+00> : tensor<1024xf32, #blocked0>
```
**EN:** This function-oriented block defines or enters `long_func`. Within it, the test exercises constants, tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `long_func` 为核心。测试在其中演示 常量、tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 535-552
```mlir
535|   %cst_14 = arith.constant dense<0> : tensor<1024xi32, #blocked0>
536|   %0 = tt.get_program_id x : i32
537|   %1 = arith.muli %0, %c1024_i32 : i32
538|   %2 = tt.make_range {end = 1024 : i32, start = 0 : i32} : tensor<1024xi32, #blocked0>
539|   %3 = tt.splat %1 : i32 -> tensor<1024xi32, #blocked0>
540|   %4 = arith.addi %3, %2 : tensor<1024xi32, #blocked0>
541|   %5 = arith.cmpi "slt", %4, %cst_11 : tensor<1024xi32, #blocked0>
542|   %6 = tt.splat %arg5 : !tt.ptr<f32> -> tensor<1024x!tt.ptr<f32>, #blocked0>
543|   %7 = tt.addptr %6, %4 : tensor<1024x!tt.ptr<f32>, #blocked0>, tensor<1024xi32, #blocked0>
544|   %8 = ttg.convert_layout %7 : tensor<1024x!tt.ptr<f32>, #blocked0> -> tensor<1024x!tt.ptr<f32>, #blocked0a>
545|   %9 = ttg.convert_layout %5 : tensor<1024xi1, #blocked0> -> tensor<1024xi1, #blocked0a>
546|   %10 = tt.load %8, %9 : tensor<1024x!tt.ptr<f32>, #blocked0a>
547|   %11 = ttg.convert_layout %10 : tensor<1024xf32, #blocked0a> -> tensor<1024xf32, #blocked0>
548|   %12 = tt.splat %arg7 : !tt.ptr<i64> -> tensor<1024x!tt.ptr<i64>, #blocked0>
549|   %13 = tt.addptr %12, %4 : tensor<1024x!tt.ptr<i64>, #blocked0>, tensor<1024xi32, #blocked0>
550|   %14 = ttg.convert_layout %13 : tensor<1024x!tt.ptr<i64>, #blocked0> -> tensor<1024x!tt.ptr<i64>, #blocked2a>
551|   %15 = ttg.convert_layout %5 : tensor<1024xi1, #blocked0> -> tensor<1024xi1, #blocked2a>
552|   %16 = tt.load %14, %15 : tensor<1024x!tt.ptr<i64>, #blocked2a>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, broadcasted scalars or pointers, pointer arithmetic, masked or vectorized loads, constants. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、广播后的标量或指针、指针算术、带掩码或向量化的加载、常量。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 553-570
```mlir
553|   %17 = ttg.convert_layout %16 : tensor<1024xi64, #blocked2a> -> tensor<1024xi64, #blocked0>
554|   %18 = tt.splat %arg8 : !tt.ptr<f32> -> tensor<1024x!tt.ptr<f32>, #blocked0>
555|   %19 = tt.addptr %18, %4 : tensor<1024x!tt.ptr<f32>, #blocked0>, tensor<1024xi32, #blocked0>
556|   %20 = ttg.convert_layout %19 : tensor<1024x!tt.ptr<f32>, #blocked0> -> tensor<1024x!tt.ptr<f32>, #blocked0a>
557|   %21 = ttg.convert_layout %5 : tensor<1024xi1, #blocked0> -> tensor<1024xi1, #blocked0a>
558|   %22 = tt.load %20, %21 : tensor<1024x!tt.ptr<f32>, #blocked0a>
559|   %23 = ttg.convert_layout %22 : tensor<1024xf32, #blocked0a> -> tensor<1024xf32, #blocked0>
560|   %24 = arith.subf %cst_13, %11 : tensor<1024xf32, #blocked0>
561|   %25 = math.exp %24 : tensor<1024xf32, #blocked0>
562|   %26 = arith.sitofp %cst_12 : tensor<1024xi32, #blocked0> to tensor<1024xf32, #blocked0>
563|   %27 = arith.addf %25, %26 : tensor<1024xf32, #blocked0>
564|   %28 = arith.divf %26, %27 : tensor<1024xf32, #blocked0>
565|   %29 = tt.addptr %arg6, %c2499_i32 : !tt.ptr<f32>, i32
566|   %30 = tt.load %29 : !tt.ptr<f32>
567|   %31 = arith.subf %11, %cst_10 : tensor<1024xf32, #blocked0>
568|   %32 = arith.subf %cst_13, %31 : tensor<1024xf32, #blocked0>
569|   %33 = math.exp %32 : tensor<1024xf32, #blocked0>
570|   %34 = arith.addf %33, %26 : tensor<1024xf32, #blocked0>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, arith.subf, pointer arithmetic, masked or vectorized loads, math.exp. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、arith.subf、指针算术、带掩码或向量化的加载、math.exp。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 571-588
```mlir
571|   %35 = arith.divf %26, %34 : tensor<1024xf32, #blocked0>
572|   %36 = tt.splat %30 : f32 -> tensor<1024xf32, #blocked0>
573|   %37 = arith.cmpf "oge", %36, %35 : tensor<1024xf32, #blocked0>
574|   %38 = arith.select %37, %cst_14, %cst_9 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
575|   %39 = arith.select %37, %cst_8, %cst_7 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
576|   %40 = arith.subi %39, %38 : tensor<1024xi32, #blocked0>
577|   %41 = arith.cmpi "slt", %40, %cst_14 : tensor<1024xi32, #blocked0>
578|   %42 = arith.cmpi "ne", %41, %cst_5 : tensor<1024xi1, #blocked0>
579|   %43 = arith.remsi %40, %cst_6 : tensor<1024xi32, #blocked0>
580|   %44 = arith.cmpi "ne", %43, %cst_14 : tensor<1024xi32, #blocked0>
581|   %45 = arith.divsi %40, %cst_6 : tensor<1024xi32, #blocked0>
582|   %46 = arith.subi %45, %cst_12 : tensor<1024xi32, #blocked0>
583|   %47 = arith.select %44, %46, %45 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
584|   %48 = arith.select %42, %47, %45 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
585|   %49 = arith.addi %38, %48 : tensor<1024xi32, #blocked0>
586|   %50 = arith.cmpi "slt", %38, %39 : tensor<1024xi32, #blocked0>
587|   %51 = arith.select %50, %49, %cst_14 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
588|   %52 = tt.splat %arg6 : !tt.ptr<f32> -> tensor<1024x!tt.ptr<f32>, #blocked0>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on arith.select, integer comparisons, broadcasted scalars or pointers, arith.subi, arith.divf. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 arith.select、整数比较、广播后的标量或指针、arith.subi、arith.divf。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 589-606
```mlir
589|   %53 = tt.addptr %52, %51 : tensor<1024x!tt.ptr<f32>, #blocked0>, tensor<1024xi32, #blocked0>
590|   %54 = ttg.convert_layout %53 : tensor<1024x!tt.ptr<f32>, #blocked0> -> tensor<1024x!tt.ptr<f32>, #blocked0>
591|   %55 = tt.load %54 : tensor<1024x!tt.ptr<f32>, #blocked0>
592|   %56 = arith.cmpf "oge", %55, %35 :tensor<1024xf32, #blocked0>
593|   %57 = arith.cmpi "eq", %56, %cst_5 : tensor<1024xi1, #blocked0>
594|   %58 = arith.andi %57, %50 : tensor<1024xi1, #blocked0>
595|   %59 = arith.addi %51, %cst_12 : tensor<1024xi32, #blocked0>
596|   %60 = arith.select %58, %59, %38 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
597|   %61 = arith.andi %56, %50 : tensor<1024xi1, #blocked0>
598|   %62 = arith.select %61, %51, %39 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
599|   %63 = arith.cmpi "slt", %60, %62 : tensor<1024xi32, #blocked0>
600|   %64 = arith.subi %62, %60 : tensor<1024xi32, #blocked0>
601|   %65 = arith.cmpi "slt", %64, %cst_14 : tensor<1024xi32, #blocked0>
602|   %66 = arith.cmpi "ne", %65, %cst_5 : tensor<1024xi1, #blocked0>
603|   %67 = arith.remsi %64, %cst_6 : tensor<1024xi32, #blocked0>
604|   %68 = arith.cmpi "ne", %67, %cst_14 : tensor<1024xi32, #blocked0>
605|   %69 = arith.divsi %64, %cst_6 : tensor<1024xi32, #blocked0>
606|   %70 = arith.subi %69, %cst_12 : tensor<1024xi32, #blocked0>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on integer comparisons, arith.andi, arith.select, arith.subi, pointer arithmetic. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 整数比较、arith.andi、arith.select、arith.subi、指针算术。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 607-624
```mlir
607|   %71 = arith.select %68, %70, %69 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
608|   %72 = arith.select %66, %71, %69 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
609|   %73 = arith.addi %60, %72 : tensor<1024xi32, #blocked0>
610|   %74 = arith.select %63, %73, %cst_14 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
611|   %75 = tt.addptr %52, %74 : tensor<1024x!tt.ptr<f32>, #blocked0>, tensor<1024xi32, #blocked0>
612|   %76 = ttg.convert_layout %75 : tensor<1024x!tt.ptr<f32>, #blocked0> -> tensor<1024x!tt.ptr<f32>, #blocked0>
613|   %77 = tt.load %76 : tensor<1024x!tt.ptr<f32>, #blocked0>
614|   %78 = arith.cmpf "oge", %77, %35 :tensor<1024xf32, #blocked0>
615|   %79 = arith.cmpi "eq", %78, %cst_5 : tensor<1024xi1, #blocked0>
616|   %80 = arith.andi %79, %63 : tensor<1024xi1, #blocked0>
617|   %81 = arith.addi %74, %cst_12 : tensor<1024xi32, #blocked0>
618|   %82 = arith.select %80, %81, %60 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
619|   %83 = arith.andi %78, %63 : tensor<1024xi1, #blocked0>
620|   %84 = arith.select %83, %74, %62 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
621|   %85 = arith.cmpi "slt", %82, %84 : tensor<1024xi32, #blocked0>
622|   %86 = arith.subi %84, %82 : tensor<1024xi32, #blocked0>
623|   %87 = arith.cmpi "slt", %86, %cst_14 : tensor<1024xi32, #blocked0>
624|   %88 = arith.cmpi "ne", %87, %cst_5 : tensor<1024xi1, #blocked0>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on arith.select, integer comparisons, integer additions, arith.andi, pointer arithmetic. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 arith.select、整数比较、整数加法、arith.andi、指针算术。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 625-642
```mlir
625|   %89 = arith.remsi %86, %cst_6 : tensor<1024xi32, #blocked0>
626|   %90 = arith.cmpi "ne", %89, %cst_14 : tensor<1024xi32, #blocked0>
627|   %91 = arith.divsi %86, %cst_6 : tensor<1024xi32, #blocked0>
628|   %92 = arith.subi %91, %cst_12 : tensor<1024xi32, #blocked0>
629|   %93 = arith.select %90, %92, %91 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
630|   %94 = arith.select %88, %93, %91 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
631|   %95 = arith.addi %82, %94 : tensor<1024xi32, #blocked0>
632|   %96 = arith.select %85, %95, %cst_14 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
633|   %97 = tt.addptr %52, %96 : tensor<1024x!tt.ptr<f32>, #blocked0>, tensor<1024xi32, #blocked0>
634|   %98 = ttg.convert_layout %97 : tensor<1024x!tt.ptr<f32>, #blocked0> -> tensor<1024x!tt.ptr<f32>, #blocked0>
635|   %99 = tt.load %98 : tensor<1024x!tt.ptr<f32>, #blocked0>
636|   %100 = arith.cmpf "oge", %99, %35 : tensor<1024xf32, #blocked0>
637|   %101 = arith.cmpi "eq", %100, %cst_5 : tensor<1024xi1, #blocked0>
638|   %102 = arith.andi %101, %85 : tensor<1024xi1, #blocked0>
639|   %103 = arith.addi %96, %cst_12 : tensor<1024xi32, #blocked0>
640|   %104 = arith.select %102, %103, %82 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
641|   %105 = arith.andi %100, %85 : tensor<1024xi1, #blocked0>
642|   %106 = arith.select %105, %96, %84 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on arith.select, integer comparisons, integer additions, arith.andi, arith.remsi. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 arith.select、整数比较、整数加法、arith.andi、arith.remsi。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 643-660
```mlir
643|   %107 = arith.cmpi "slt", %104, %106 : tensor<1024xi32, #blocked0>
644|   %108 = arith.subi %106, %104 : tensor<1024xi32, #blocked0>
645|   %109 = arith.cmpi "slt", %108, %cst_14 : tensor<1024xi32, #blocked0>
646|   %110 = arith.cmpi "ne", %109, %cst_5 : tensor<1024xi1, #blocked0>
647|   %111 = arith.remsi %108, %cst_6 : tensor<1024xi32, #blocked0>
648|   %112 = arith.cmpi "ne", %111, %cst_14 : tensor<1024xi32, #blocked0>
649|   %113 = arith.divsi %108, %cst_6 : tensor<1024xi32, #blocked0>
650|   %114 = arith.subi %113, %cst_12 : tensor<1024xi32, #blocked0>
651|   %115 = arith.select %112, %114, %113 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
652|   %116 = arith.select %110, %115, %113 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
653|   %117 = arith.addi %104, %116 : tensor<1024xi32, #blocked0>
654|   %118 = arith.select %107, %117, %cst_14 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
655|   %119 = tt.addptr %52, %118 : tensor<1024x!tt.ptr<f32>, #blocked0>, tensor<1024xi32, #blocked0>
656|   %120 = ttg.convert_layout %119 : tensor<1024x!tt.ptr<f32>, #blocked0> -> tensor<1024x!tt.ptr<f32>, #blocked0>
657|   %121 = tt.load %120 : tensor<1024x!tt.ptr<f32>, #blocked0>
658|   %122 = arith.cmpf "oge", %121, %35 : tensor<1024xf32, #blocked0>
659|   %123 = arith.cmpi "eq", %122, %cst_5 : tensor<1024xi1, #blocked0>
660|   %124 = arith.andi %123, %107 : tensor<1024xi1, #blocked0>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on integer comparisons, arith.select, arith.subi, arith.remsi, arith.divsi. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 整数比较、arith.select、arith.subi、arith.remsi、arith.divsi。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 661-678
```mlir
661|   %125 = arith.addi %118, %cst_12 : tensor<1024xi32, #blocked0>
662|   %126 = arith.select %124, %125, %104 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
663|   %127 = arith.andi %122, %107 : tensor<1024xi1, #blocked0>
664|   %128 = arith.select %127, %118, %106 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
665|   %129 = arith.cmpi "slt", %126, %128 : tensor<1024xi32, #blocked0>
666|   %130 = arith.subi %128, %126 : tensor<1024xi32, #blocked0>
667|   %131 = arith.cmpi "slt", %130, %cst_14 : tensor<1024xi32, #blocked0>
668|   %132 = arith.cmpi "ne", %131, %cst_5 : tensor<1024xi1, #blocked0>
669|   %133 = arith.remsi %130, %cst_6 : tensor<1024xi32, #blocked0>
670|   %134 = arith.cmpi "ne", %133, %cst_14 : tensor<1024xi32, #blocked0>
671|   %135 = arith.divsi %130, %cst_6 : tensor<1024xi32, #blocked0>
672|   %136 = arith.subi %135, %cst_12 : tensor<1024xi32, #blocked0>
673|   %137 = arith.select %134, %136, %135 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
674|   %138 = arith.select %132, %137, %135 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
675|   %139 = arith.addi %126, %138 : tensor<1024xi32, #blocked0>
676|   %140 = arith.select %129, %139, %cst_14 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
677|   %141 = tt.addptr %52, %140 : tensor<1024x!tt.ptr<f32>, #blocked0>, tensor<1024xi32, #blocked0>
678|   %142 = ttg.convert_layout %141 : tensor<1024x!tt.ptr<f32>, #blocked0> -> tensor<1024x!tt.ptr<f32>, #blocked0>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on arith.select, integer comparisons, integer additions, arith.subi, arith.andi. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 arith.select、整数比较、整数加法、arith.subi、arith.andi。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 679-696
```mlir
679|   %143 = tt.load %142 : tensor<1024x!tt.ptr<f32>, #blocked0>
680|   %144 = arith.cmpf "oge", %143, %35 : tensor<1024xf32, #blocked0>
681|   %145 = arith.cmpi "eq", %144, %cst_5 : tensor<1024xi1, #blocked0>
682|   %146 = arith.andi %145, %129 : tensor<1024xi1, #blocked0>
683|   %147 = arith.addi %140, %cst_12 : tensor<1024xi32, #blocked0>
684|   %148 = arith.select %146, %147, %126 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
685|   %149 = arith.andi %144, %129 : tensor<1024xi1, #blocked0>
686|   %150 = arith.select %149, %140, %128 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
687|   %151 = arith.cmpi "slt", %148, %150 : tensor<1024xi32, #blocked0>
688|   %152 = arith.subi %150, %148 : tensor<1024xi32, #blocked0>
689|   %153 = arith.cmpi "slt", %152, %cst_14 : tensor<1024xi32, #blocked0>
690|   %154 = arith.cmpi "ne", %153, %cst_5 : tensor<1024xi1, #blocked0>
691|   %155 = arith.remsi %152, %cst_6 : tensor<1024xi32, #blocked0>
692|   %156 = arith.cmpi "ne", %155, %cst_14 : tensor<1024xi32, #blocked0>
693|   %157 = arith.divsi %152, %cst_6 : tensor<1024xi32, #blocked0>
694|   %158 = arith.subi %157, %cst_12 : tensor<1024xi32, #blocked0>
695|   %159 = arith.select %156, %158, %157 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
696|   %160 = arith.select %154, %159, %157 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on integer comparisons, arith.select, arith.andi, arith.subi, masked or vectorized loads. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 整数比较、arith.select、arith.andi、arith.subi、带掩码或向量化的加载。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 697-714
```mlir
697|   %161 = arith.addi %148, %160 : tensor<1024xi32, #blocked0>
698|   %162 = arith.select %151, %161, %cst_14 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
699|   %163 = tt.addptr %52, %162 : tensor<1024x!tt.ptr<f32>, #blocked0>, tensor<1024xi32, #blocked0>
700|   %164 = ttg.convert_layout %163 : tensor<1024x!tt.ptr<f32>, #blocked0> -> tensor<1024x!tt.ptr<f32>, #blocked0>
701|   %165 = tt.load %164 : tensor<1024x!tt.ptr<f32>, #blocked0>
702|   %166 = arith.cmpf "oge", %165, %35 : tensor<1024xf32, #blocked0>
703|   %167 = arith.cmpi "eq", %166, %cst_5 : tensor<1024xi1, #blocked0>
704|   %168 = arith.andi %167, %151 : tensor<1024xi1, #blocked0>
705|   %169 = arith.addi %162, %cst_12 : tensor<1024xi32, #blocked0>
706|   %170 = arith.select %168, %169, %148 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
707|   %171 = arith.andi %166, %151 : tensor<1024xi1, #blocked0>
708|   %172 = arith.select %171, %162, %150 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
709|   %173 = arith.cmpi "slt", %170, %172 : tensor<1024xi32, #blocked0>
710|   %174 = arith.subi %172, %170 : tensor<1024xi32, #blocked0>
711|   %175 = arith.cmpi "slt", %174, %cst_14 : tensor<1024xi32, #blocked0>
712|   %176 = arith.cmpi "ne", %175, %cst_5 : tensor<1024xi1, #blocked0>
713|   %177 = arith.remsi %174, %cst_6 : tensor<1024xi32, #blocked0>
714|   %178 = arith.cmpi "ne", %177, %cst_14 : tensor<1024xi32, #blocked0>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on integer comparisons, arith.select, integer additions, arith.andi, pointer arithmetic. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 整数比较、arith.select、整数加法、arith.andi、指针算术。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 715-732
```mlir
715|   %179 = arith.divsi %174, %cst_6 : tensor<1024xi32, #blocked0>
716|   %180 = arith.subi %179, %cst_12 : tensor<1024xi32, #blocked0>
717|   %181 = arith.select %178, %180, %179 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
718|   %182 = arith.select %176, %181, %179 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
719|   %183 = arith.addi %170, %182 : tensor<1024xi32, #blocked0>
720|   %184 = arith.select %173, %183, %cst_14 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
721|   %185 = tt.addptr %52, %184 : tensor<1024x!tt.ptr<f32>, #blocked0>, tensor<1024xi32, #blocked0>
722|   %186 = ttg.convert_layout %185 : tensor<1024x!tt.ptr<f32>, #blocked0> -> tensor<1024x!tt.ptr<f32>, #blocked0>
723|   %187 = tt.load %186 : tensor<1024x!tt.ptr<f32>, #blocked0>
724|   %188 = arith.cmpf "oge", %187, %35 : tensor<1024xf32, #blocked0>
725|   %189 = arith.cmpi "eq", %188, %cst_5 : tensor<1024xi1, #blocked0>
726|   %190 = arith.andi %189, %173 : tensor<1024xi1, #blocked0>
727|   %191 = arith.addi %184, %cst_12 : tensor<1024xi32, #blocked0>
728|   %192 = arith.select %190, %191, %170 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
729|   %193 = arith.andi %188, %173 : tensor<1024xi1, #blocked0>
730|   %194 = arith.select %193, %184, %172 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
731|   %195 = arith.cmpi "slt", %192, %194 : tensor<1024xi32, #blocked0>
732|   %196 = arith.subi %194, %192 : tensor<1024xi32, #blocked0>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on arith.select, arith.subi, integer additions, integer comparisons, arith.andi. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 arith.select、arith.subi、整数加法、整数比较、arith.andi。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 733-750
```mlir
733|   %197 = arith.cmpi "slt", %196, %cst_14 : tensor<1024xi32, #blocked0>
734|   %198 = arith.cmpi "ne", %197, %cst_5 : tensor<1024xi1, #blocked0>
735|   %199 = arith.remsi %196, %cst_6 : tensor<1024xi32, #blocked0>
736|   %200 = arith.cmpi "ne", %199, %cst_14 : tensor<1024xi32, #blocked0>
737|   %201 = arith.divsi %196, %cst_6 : tensor<1024xi32, #blocked0>
738|   %202 = arith.subi %201, %cst_12 : tensor<1024xi32, #blocked0>
739|   %203 = arith.select %200, %202, %201 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
740|   %204 = arith.select %198, %203, %201 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
741|   %205 = arith.addi %192, %204 : tensor<1024xi32, #blocked0>
742|   %206 = arith.select %195, %205, %cst_14 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
743|   %207 = tt.addptr %52, %206 : tensor<1024x!tt.ptr<f32>, #blocked0>, tensor<1024xi32, #blocked0>
744|   %208 = ttg.convert_layout %207 : tensor<1024x!tt.ptr<f32>, #blocked0> -> tensor<1024x!tt.ptr<f32>, #blocked0>
745|   %209 = tt.load %208 : tensor<1024x!tt.ptr<f32>, #blocked0>
746|   %210 = arith.cmpf "oge", %209, %35 :tensor<1024xf32, #blocked0>
747|   %211 = arith.cmpi "eq", %210, %cst_5 : tensor<1024xi1, #blocked0>
748|   %212 = arith.andi %211, %195 : tensor<1024xi1, #blocked0>
749|   %213 = arith.addi %206, %cst_12 : tensor<1024xi32, #blocked0>
750|   %214 = arith.select %212, %213, %192 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on integer comparisons, arith.select, integer additions, arith.remsi, arith.divsi. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 整数比较、arith.select、整数加法、arith.remsi、arith.divsi。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 751-768
```mlir
751|   %215 = arith.andi %210, %195 : tensor<1024xi1, #blocked0>
752|   %216 = arith.select %215, %206, %194 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
753|   %217 = arith.cmpi "slt", %214, %216 : tensor<1024xi32, #blocked0>
754|   %218 = arith.subi %216, %214 : tensor<1024xi32, #blocked0>
755|   %219 = arith.cmpi "slt", %218, %cst_14 : tensor<1024xi32, #blocked0>
756|   %220 = arith.cmpi "ne", %219, %cst_5 : tensor<1024xi1, #blocked0>
757|   %221 = arith.remsi %218, %cst_6 : tensor<1024xi32, #blocked0>
758|   %222 = arith.cmpi "ne", %221, %cst_14 : tensor<1024xi32, #blocked0>
759|   %223 = arith.divsi %218, %cst_6 : tensor<1024xi32, #blocked0>
760|   %224 = arith.subi %223, %cst_12 : tensor<1024xi32, #blocked0>
761|   %225 = arith.select %222, %224, %223 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
762|   %226 = arith.select %220, %225, %223 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
763|   %227 = arith.addi %214, %226 : tensor<1024xi32, #blocked0>
764|   %228 = arith.select %217, %227, %cst_14 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
765|   %229 = tt.addptr %52, %228 : tensor<1024x!tt.ptr<f32>, #blocked0>, tensor<1024xi32, #blocked0>
766|   %230 = ttg.convert_layout %229 : tensor<1024x!tt.ptr<f32>, #blocked0> -> tensor<1024x!tt.ptr<f32>, #blocked0>
767|   %231 = tt.load %230 : tensor<1024x!tt.ptr<f32>, #blocked0>
768|   %232 = arith.cmpf "oge", %231, %35 : tensor<1024xf32, #blocked0>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on arith.select, integer comparisons, arith.subi, arith.andi, arith.remsi. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 arith.select、整数比较、arith.subi、arith.andi、arith.remsi。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 769-786
```mlir
769|   %233 = arith.cmpi "eq", %232, %cst_5 : tensor<1024xi1, #blocked0>
770|   %234 = arith.andi %233, %217 : tensor<1024xi1, #blocked0>
771|   %235 = arith.addi %228, %cst_12 : tensor<1024xi32, #blocked0>
772|   %236 = arith.select %234, %235, %214 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
773|   %237 = arith.andi %232, %217 : tensor<1024xi1, #blocked0>
774|   %238 = arith.select %237, %228, %216 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
775|   %239 = arith.cmpi "slt", %236, %238 : tensor<1024xi32, #blocked0>
776|   %240 = arith.subi %238, %236 : tensor<1024xi32, #blocked0>
777|   %241 = arith.cmpi "slt", %240, %cst_14 : tensor<1024xi32, #blocked0>
778|   %242 = arith.cmpi "ne", %241, %cst_5 : tensor<1024xi1, #blocked0>
779|   %243 = arith.remsi %240, %cst_6 : tensor<1024xi32, #blocked0>
780|   %244 = arith.cmpi "ne", %243, %cst_14 : tensor<1024xi32, #blocked0>
781|   %245 = arith.divsi %240, %cst_6 : tensor<1024xi32, #blocked0>
782|   %246 = arith.subi %245, %cst_12 : tensor<1024xi32, #blocked0>
783|   %247 = arith.select %244, %246, %245 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
784|   %248 = arith.select %242, %247, %245 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
785|   %249 = arith.addi %236, %248 : tensor<1024xi32, #blocked0>
786|   %250 = arith.select %239, %249, %cst_14 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on integer comparisons, arith.select, arith.andi, integer additions, arith.subi. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 整数比较、arith.select、arith.andi、整数加法、arith.subi。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 787-804
```mlir
787|   %251 = tt.addptr %52, %250 : tensor<1024x!tt.ptr<f32>, #blocked0>, tensor<1024xi32, #blocked0>
788|   %252 = ttg.convert_layout %251 : tensor<1024x!tt.ptr<f32>, #blocked0> -> tensor<1024x!tt.ptr<f32>, #blocked0>
789|   %253 = tt.load %252 : tensor<1024x!tt.ptr<f32>, #blocked0>
790|   %254 = arith.cmpf "oge", %253, %35 : tensor<1024xf32, #blocked0>
791|   %255 = arith.cmpi "eq", %254, %cst_5 : tensor<1024xi1, #blocked0>
792|   %256 = arith.andi %255, %239 : tensor<1024xi1, #blocked0>
793|   %257 = arith.addi %250, %cst_12 : tensor<1024xi32, #blocked0>
794|   %258 = arith.select %256, %257, %236 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
795|   %259 = arith.andi %254, %239 : tensor<1024xi1, #blocked0>
796|   %260 = arith.select %259, %250, %238 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
797|   %261 = arith.cmpi "slt", %258, %260 : tensor<1024xi32, #blocked0>
798|   %262 = arith.subi %260, %258 : tensor<1024xi32, #blocked0>
799|   %263 = arith.cmpi "slt", %262, %cst_14 : tensor<1024xi32, #blocked0>
800|   %264 = arith.cmpi "ne", %263, %cst_5 : tensor<1024xi1, #blocked0>
801|   %265 = arith.remsi %262, %cst_6 : tensor<1024xi32, #blocked0>
802|   %266 = arith.cmpi "ne", %265, %cst_14 : tensor<1024xi32, #blocked0>
803|   %267 = arith.divsi %262, %cst_6 : tensor<1024xi32, #blocked0>
804|   %268 = arith.subi %267, %cst_12 : tensor<1024xi32, #blocked0>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on integer comparisons, arith.andi, arith.select, arith.subi, pointer arithmetic. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 整数比较、arith.andi、arith.select、arith.subi、指针算术。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 805-822
```mlir
805|   %269 = arith.select %266, %268, %267 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
806|   %270 = arith.select %264, %269, %267 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
807|   %271 = arith.addi %258, %270 : tensor<1024xi32, #blocked0>
808|   %272 = arith.select %261, %271, %cst_14 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
809|   %273 = tt.addptr %52, %272 : tensor<1024x!tt.ptr<f32>, #blocked0>, tensor<1024xi32, #blocked0>
810|   %274 = ttg.convert_layout %273 : tensor<1024x!tt.ptr<f32>, #blocked0> -> tensor<1024x!tt.ptr<f32>, #blocked0>
811|   %275 = tt.load %274 : tensor<1024x!tt.ptr<f32>, #blocked0>
812|   %276 = arith.cmpf "oge", %275, %35 : tensor<1024xf32, #blocked0>
813|   %277 = arith.cmpi "eq", %276, %cst_5 : tensor<1024xi1, #blocked0>
814|   %278 = arith.andi %277, %261 : tensor<1024xi1, #blocked0>
815|   %279 = arith.addi %272, %cst_12 : tensor<1024xi32, #blocked0>
816|   %280 = arith.select %278, %279, %258 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
817|   %281 = arith.andi %276, %261 : tensor<1024xi1, #blocked0>
818|   %282 = arith.select %281, %272, %260 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
819|   %283 = arith.cmpi "slt", %280, %282 : tensor<1024xi32, #blocked0>
820|   %284 = arith.subi %282, %280 : tensor<1024xi32, #blocked0>
821|   %285 = arith.cmpi "slt", %284, %cst_14 : tensor<1024xi32, #blocked0>
822|   %286 = arith.cmpi "ne", %285, %cst_5 : tensor<1024xi1, #blocked0>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on arith.select, integer comparisons, integer additions, arith.andi, pointer arithmetic. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 arith.select、整数比较、整数加法、arith.andi、指针算术。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 823-840
```mlir
823|   %287 = arith.remsi %284, %cst_6 : tensor<1024xi32, #blocked0>
824|   %288 = arith.cmpi "ne", %287, %cst_14 : tensor<1024xi32, #blocked0>
825|   %289 = arith.divsi %284, %cst_6 : tensor<1024xi32, #blocked0>
826|   %290 = arith.subi %289, %cst_12 : tensor<1024xi32, #blocked0>
827|   %291 = arith.select %288, %290, %289 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
828|   %292 = arith.select %286, %291, %289 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
829|   %293 = arith.addi %280, %292 : tensor<1024xi32, #blocked0>
830|   %294 = arith.select %283, %293, %cst_14 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
831|   %295 = tt.addptr %52, %294 : tensor<1024x!tt.ptr<f32>, #blocked0>, tensor<1024xi32, #blocked0>
832|   %296 = ttg.convert_layout %295 : tensor<1024x!tt.ptr<f32>, #blocked0> -> tensor<1024x!tt.ptr<f32>, #blocked0>
833|   %297 = tt.load %296 : tensor<1024x!tt.ptr<f32>, #blocked0>
834|   %298 = arith.cmpf "oge", %297, %35 :tensor<1024xf32, #blocked0>
835|   %299 = arith.cmpi "eq", %298, %cst_5 : tensor<1024xi1, #blocked0>
836|   %300 = arith.andi %299, %283 : tensor<1024xi1, #blocked0>
837|   %301 = arith.addi %294, %cst_12 : tensor<1024xi32, #blocked0>
838|   %302 = arith.select %300, %301, %280 : tensor<1024xi1, #blocked0>, tensor<1024xi32, #blocked0>
839|   %303 = arith.extsi %cst_12 : tensor<1024xi32, #blocked0> to tensor<1024xi64, #blocked0>
840|   %304 = arith.cmpi "eq", %17, %303 : tensor<1024xi64, #blocked0>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on arith.select, integer comparisons, integer additions, arith.remsi, arith.divsi. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 arith.select、整数比较、整数加法、arith.remsi、arith.divsi。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 841-858
```mlir
841|   %305 = arith.fptosi %23 : tensor<1024xf32, #blocked0> to tensor<1024xi64, #blocked0>
842|   %306 = arith.extsi %cst_14 : tensor<1024xi32, #blocked0> to tensor<1024xi64, #blocked0>
843|   %307 = arith.cmpi "sgt", %306, %305 : tensor<1024xi64, #blocked0>
844|   %308 = arith.extsi %cst_4 : tensor<1024xi32, #blocked0> to tensor<1024xi64, #blocked0>
845|   %309 = arith.cmpi "sgt", %305, %308 : tensor<1024xi64, #blocked0>
846|   %310 = arith.select %309, %306, %305 : tensor<1024xi1, #blocked0>, tensor<1024xi64, #blocked0>
847|   %311 = arith.select %307, %306, %310 : tensor<1024xi1, #blocked0>, tensor<1024xi64, #blocked0>
848|   %312 = arith.select %304, %311, %306 : tensor<1024xi1, #blocked0>, tensor<1024xi64, #blocked0>
849|   %313 = arith.extsi %cst_3 : tensor<1024xi32, #blocked0> to tensor<1024xi64, #blocked0>
850|   %314 = arith.muli %312, %313 : tensor<1024xi64, #blocked0>
851|   %315 = arith.extsi %302 : tensor<1024xi32, #blocked0> to tensor<1024xi64, #blocked0>
852|   %316 = arith.addi %315, %314 : tensor<1024xi64, #blocked0>
853|   %317 = arith.trunci %316 : tensor<1024xi64, #blocked0> to tensor<1024xi32, #blocked0>
854|   %318 = arith.extsi %317 : tensor<1024xi32, #blocked0> to tensor<1024xi64, #blocked0>
855|   %319 = tt.splat %arg9 : !tt.ptr<f64> -> tensor<1024x!tt.ptr<f64>, #blocked0>
856|   %320 = tt.addptr %319, %318 : tensor<1024x!tt.ptr<f64>, #blocked0>, tensor<1024xi64, #blocked0>
857|   %321 = ttg.convert_layout %320 : tensor<1024x!tt.ptr<f64>, #blocked0> -> tensor<1024x!tt.ptr<f64>, #blocked0>
858|   %322 = tt.load %321 : tensor<1024x!tt.ptr<f64>, #blocked0>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on arith.extsi, arith.select, integer comparisons, arith.fptosi, integer multiplications. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 arith.extsi、arith.select、整数比较、arith.fptosi、整数乘法。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 859-876
```mlir
859|   %323 = arith.extf %cst_2 : tensor<1024xf32, #blocked0> to tensor<1024xf64, #blocked0>
860|   %324 = arith.cmpf "ogt", %322, %323 : tensor<1024xf64, #blocked0>
861|   %325 = tt.splat %arg10 : !tt.ptr<f64> -> tensor<1024x!tt.ptr<f64>, #blocked0>
862|   %326 = tt.addptr %325, %318 : tensor<1024x!tt.ptr<f64>, #blocked0>, tensor<1024xi64, #blocked0>
863|   %327 = ttg.convert_layout %326 : tensor<1024x!tt.ptr<f64>, #blocked0> -> tensor<1024x!tt.ptr<f64>, #blocked0>
864|   %328 = tt.load %327 : tensor<1024x!tt.ptr<f64>, #blocked0>
865|   %329 = arith.divf %328, %322 : tensor<1024xf64, #blocked0>
866|   %330 = arith.truncf %329 : tensor<1024xf64, #blocked0> to tensor<1024xf32, #blocked0>
867|   %331 = arith.mulf %330, %cst_1 : tensor<1024xf32, #blocked0>
868|   %332 = arith.mulf %35, %cst_0 : tensor<1024xf32, #blocked0>
869|   %333 = arith.addf %331, %332 : tensor<1024xf32, #blocked0>
870|   %334 = arith.select %324, %333, %35 : tensor<1024xi1, #blocked0>, tensor<1024xf32, #blocked0>
871|   %335 = tt.addptr %319, %317 : tensor<1024x!tt.ptr<f64>, #blocked0>, tensor<1024xi32, #blocked0>
872|   %336 = ttg.convert_layout %335 : tensor<1024x!tt.ptr<f64>, #blocked0> -> tensor<1024x!tt.ptr<f64>, #blocked0>
873|   %337 = tt.load %336 : tensor<1024x!tt.ptr<f64>, #blocked0>
874|   %338 = arith.extf %cst : tensor<1024xf32, #blocked0> to tensor<1024xf64, #blocked0>
875|   %339 = arith.mulf %337, %338 : tensor<1024xf64, #blocked0>
876|   %340 = tt.addptr %325, %317 : tensor<1024x!tt.ptr<f64>, #blocked0>, tensor<1024xi32, #blocked0>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on pointer arithmetic, arith.mulf, arith.extf, layout conversions, masked or vectorized loads. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 指针算术、arith.mulf、arith.extf、布局转换、带掩码或向量化的加载。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 877-894
```mlir
877|   %341 = ttg.convert_layout %340 : tensor<1024x!tt.ptr<f64>, #blocked0> -> tensor<1024x!tt.ptr<f64>, #blocked0>
878|   %342 = tt.load %341 : tensor<1024x!tt.ptr<f64>, #blocked0>
879|   %343 = arith.mulf %342, %338 : tensor<1024xf64, #blocked0>
880|   %344 = tt.splat %arg11 : !tt.ptr<f32> -> tensor<1024x!tt.ptr<f32>, #blocked0>
881|   %345 = tt.addptr %344, %4 : tensor<1024x!tt.ptr<f32>, #blocked0>, tensor<1024xi32, #blocked0>
882|   %346 = ttg.convert_layout %345 : tensor<1024x!tt.ptr<f32>, #blocked0> -> tensor<1024x!tt.ptr<f32>, #blocked0a>
883|   %347 = ttg.convert_layout %28 : tensor<1024xf32, #blocked0> -> tensor<1024xf32, #blocked0a>
884|   %348 = ttg.convert_layout %5 : tensor<1024xi1, #blocked0> -> tensor<1024xi1, #blocked0a>
885|   tt.store %346, %347, %348 : tensor<1024x!tt.ptr<f32>, #blocked0a>
886|   %349 = tt.splat %arg12 : !tt.ptr<i32> -> tensor<1024x!tt.ptr<i32>, #blocked0>
887|   %350 = tt.addptr %349, %4 : tensor<1024x!tt.ptr<i32>, #blocked0>, tensor<1024xi32, #blocked0>
888|   %351 = ttg.convert_layout %350 : tensor<1024x!tt.ptr<i32>, #blocked0> -> tensor<1024x!tt.ptr<i32>, #blocked0a>
889|   %352 = ttg.convert_layout %317 : tensor<1024xi32, #blocked0> -> tensor<1024xi32, #blocked0a>
890|   %353 = ttg.convert_layout %5 : tensor<1024xi1, #blocked0> -> tensor<1024xi1, #blocked0a>
891|   tt.store %351, %352, %353 : tensor<1024x!tt.ptr<i32>, #blocked0a>
892|   %354 = tt.splat %arg13 : !tt.ptr<f32> -> tensor<1024x!tt.ptr<f32>, #blocked0>
893|   %355 = tt.addptr %354, %4 : tensor<1024x!tt.ptr<f32>, #blocked0>, tensor<1024xi32, #blocked0>
894|   %356 = ttg.convert_layout %355 : tensor<1024x!tt.ptr<f32>, #blocked0> -> tensor<1024x!tt.ptr<f32>, #blocked0a>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, broadcasted scalars or pointers, pointer arithmetic, masked or vectorized stores, masked or vectorized loads. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、广播后的标量或指针、指针算术、带掩码或向量化的存储、带掩码或向量化的加载。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 895-912
```mlir
895|   %357 = ttg.convert_layout %334 : tensor<1024xf32, #blocked0> -> tensor<1024xf32, #blocked0a>
896|   %358 = ttg.convert_layout %5 : tensor<1024xi1, #blocked0> -> tensor<1024xi1, #blocked0a>
897|   tt.store %356, %357, %358 : tensor<1024x!tt.ptr<f32>, #blocked0a>
898|   %359 = tt.splat %arg14 : !tt.ptr<f64> -> tensor<1024x!tt.ptr<f64>, #blocked0>
899|   %360 = tt.addptr %359, %318 : tensor<1024x!tt.ptr<f64>, #blocked0>, tensor<1024xi64, #blocked0>
900|   %361 = ttg.convert_layout %360 : tensor<1024x!tt.ptr<f64>, #blocked0> -> tensor<1024x!tt.ptr<f64>, #blocked0>
901|   %362 = ttg.convert_layout %339 : tensor<1024xf64, #blocked0> -> tensor<1024xf64, #blocked0>
902|   tt.store %361, %362 : tensor<1024x!tt.ptr<f64>, #blocked0>
903|   %363 = tt.splat %arg15 : !tt.ptr<f64> -> tensor<1024x!tt.ptr<f64>, #blocked0>
904|   %364 = tt.addptr %363, %318 : tensor<1024x!tt.ptr<f64>, #blocked0>, tensor<1024xi64, #blocked0>
905|   %365 = ttg.convert_layout %364 : tensor<1024x!tt.ptr<f64>, #blocked0> -> tensor<1024x!tt.ptr<f64>, #blocked0>
906|   %366 = ttg.convert_layout %343 : tensor<1024xf64, #blocked0> -> tensor<1024xf64, #blocked0>
907|   tt.store %365, %366 : tensor<1024x!tt.ptr<f64>, #blocked0>
908|   tt.return
909| }
910| }
911| 
912| // A mnist model from torch inductor.
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, masked or vectorized stores, broadcasted scalars or pointers, pointer arithmetic, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、带掩码或向量化的存储、广播后的标量或指针、指针算术、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 913-913
```mlir
913| // Check if topological sort is working correct and there's no unnecessary convert
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 914-914
```mlir
914| // CHECK-LABEL: mnist
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: mnist anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: mnist 这样的标签用于锚定匹配范围。

### Lines 915-915
```mlir
915| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 916-916
```mlir
916| tt.func public @mnist(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg2: i32 {tt.divisibility = 16 : i32}, %arg3: i32) {
```
**EN:** This function-oriented block defines or enters `mnist`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mnist` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 917-934
```mlir
917|   // CHECK-NOT: ttg.convert_layout
918|   %cst = arith.constant dense<10> : tensor<16x1xi32, #blocked2>
919|   %cst_0 = arith.constant dense<10> : tensor<1x16xi32, #blocked3>
920|   %c16_i32 = arith.constant 16 : i32
921|   %cst_1 = arith.constant dense<64> : tensor<16x1xi32, #blocked2>
922|   %cst_2 = arith.constant dense<0xFF800000> : tensor<16x16xf32, #blocked2>
923|   %cst_3 = arith.constant dense<0.000000e+00> : tensor<16x16xf32, #blocked2>
924|   %cst_4 = arith.constant dense<0> : tensor<16x16xi32, #blocked2>
925|   %0 = tt.get_program_id x : i32
926|   %1 = arith.muli %0, %c16_i32 : i32
927|   %2 = tt.make_range {end = 16 : i32, start = 0 : i32} : tensor<16xi32, #blocked0>
928|   %3 = ttg.convert_layout %2 : tensor<16xi32, #blocked0> -> tensor<16xi32, #ttg.slice<{dim = 1, parent = #blocked1}>>
929|   %4 = tt.expand_dims %3 {axis = 1 : i32} : tensor<16xi32, #ttg.slice<{dim = 1, parent = #blocked1}>> -> tensor<16x1xi32, #blocked1>
930|   %5 = ttg.convert_layout %4 : tensor<16x1xi32, #blocked1> -> tensor<16x1xi32, #blocked2>
931|   %6 = tt.splat %1 : i32 -> tensor<16x1xi32, #blocked2>
932|   %7 = arith.addi %6, %5 : tensor<16x1xi32, #blocked2>
933|   %8 = arith.cmpi "slt", %7, %cst_1 : tensor<16x1xi32, #blocked2>
934|   %9 = ttg.convert_layout %2 : tensor<16xi32, #blocked0> -> tensor<16xi32, #ttg.slice<{dim = 0, parent = #blocked3}>>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants, layout conversions, program IDs, integer multiplications, lane/block index ranges. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量、布局转换、程序 ID、整数乘法、lane/block 索引范围。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 935-952
```mlir
935|   %10 = tt.expand_dims %9 {axis = 0 : i32} : tensor<16xi32, #ttg.slice<{dim = 0, parent = #blocked3}>> -> tensor<1x16xi32, #blocked3>
936|   %11 = arith.cmpi "slt", %10, %cst_0 : tensor<1x16xi32, #blocked3>
937|   %12 = arith.muli %7, %cst : tensor<16x1xi32, #blocked2>
938|   %13 = tt.broadcast %10 : tensor<1x16xi32, #blocked3> -> tensor<16x16xi32, #blocked3>
939|   %14 = ttg.convert_layout %13 : tensor<16x16xi32, #blocked3> -> tensor<16x16xi32, #blocked2>
940|   %15 = tt.broadcast %12 : tensor<16x1xi32, #blocked2> -> tensor<16x16xi32, #blocked2>
941|   %16 = arith.addi %14, %15 : tensor<16x16xi32, #blocked2>
942|   %17 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<16x16x!tt.ptr<f32>, #blocked2>
943|   %18 = tt.addptr %17, %16 : tensor<16x16x!tt.ptr<f32>, #blocked2>, tensor<16x16xi32, #blocked2>
944|   %19 = tt.broadcast %11 : tensor<1x16xi1, #blocked3> -> tensor<16x16xi1, #blocked3>
945|   %20 = ttg.convert_layout %19 : tensor<16x16xi1, #blocked3> -> tensor<16x16xi1, #blocked2>
946|   %21 = tt.broadcast %8 : tensor<16x1xi1, #blocked2> -> tensor<16x16xi1, #blocked2>
947|   %22 = arith.andi %20, %21 : tensor<16x16xi1, #blocked2>
948|   %23 = ttg.convert_layout %18 : tensor<16x16x!tt.ptr<f32>, #blocked2> -> tensor<16x16x!tt.ptr<f32>, #blocked4>
949|   %24 = ttg.convert_layout %22 : tensor<16x16xi1, #blocked2> -> tensor<16x16xi1, #blocked4>
950|   %25 = tt.load %23, %24 : tensor<16x16x!tt.ptr<f32>, #blocked4>
951|   %26 = ttg.convert_layout %25 : tensor<16x16xf32, #blocked4> -> tensor<16x16xf32, #blocked2>
952|   %27 = arith.cmpf "olt", %cst_2, %26 : tensor<16x16xf32, #blocked2>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, tensor broadcasting, shape expansion, integer comparisons, integer multiplications. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、张量广播、形状扩展、整数比较、整数乘法。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 953-970
```mlir
953|   %28 = arith.andi %22, %27 : tensor<16x16xi1, #blocked2>
954|   %29 = arith.select %28, %26, %cst_2 : tensor<16x16xi1, #blocked2>, tensor<16x16xf32, #blocked2>
955|   %30 = "tt.reduce" (%29) ({
956|   ^bb0(%arg4: f32, %arg5: f32):
957|     %max = arith.maximumf %arg4, %arg5 : f32
958|     tt.reduce.return %max : f32
959|   }) {axis = 1 : i32} : (tensor<16x16xf32, #blocked2>) -> tensor<16xf32, #ttg.slice<{dim = 1, parent = #blocked2}>>
960|   %31 = ttg.convert_layout %30 : tensor<16xf32, #ttg.slice<{dim = 1, parent = #blocked2}>> -> tensor<16xf32, #blocked0>
961|   %32 = ttg.convert_layout %31 : tensor<16xf32, #blocked0> -> tensor<16xf32, #ttg.slice<{dim = 1, parent = #blocked1}>>
962|   %33 = tt.expand_dims %32 {axis = 1 : i32} : tensor<16xf32, #ttg.slice<{dim = 1, parent = #blocked1}>> -> tensor<16x1xf32, #blocked1>
963|   %34 = ttg.convert_layout %33 : tensor<16x1xf32, #blocked1> -> tensor<16x1xf32, #blocked2>
964|   %35 = arith.sitofp %cst_4 : tensor<16x16xi32, #blocked2> to tensor<16x16xf32, #blocked2>
965|   %36 = arith.addf %35, %cst_3 : tensor<16x16xf32, #blocked2>
966|   %37 = ttg.convert_layout %18 : tensor<16x16x!tt.ptr<f32>, #blocked2> -> tensor<16x16x!tt.ptr<f32>, #blocked4>
967|   %38 = ttg.convert_layout %22 : tensor<16x16xi1, #blocked2> -> tensor<16x16xi1, #blocked4>
968|   %39 = tt.load %37, %38 : tensor<16x16x!tt.ptr<f32>, #blocked4>
969|   %40 = ttg.convert_layout %39 : tensor<16x16xf32, #blocked4> -> tensor<16x16xf32, #blocked2>
970|   %41 = tt.broadcast %34 : tensor<16x1xf32, #blocked2> -> tensor<16x16xf32, #blocked2>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, arith.andi, arith.select, arith.maximumf, reductions. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、arith.andi、arith.select、arith.maximumf、归约。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 971-988
```mlir
971|   %42 = arith.subf %40, %41 : tensor<16x16xf32, #blocked2>
972|   %43 = math.exp %42 : tensor<16x16xf32, #blocked2>
973|   %44 = arith.addf %36, %43 : tensor<16x16xf32, #blocked2>
974|   %45 = arith.select %22, %44, %36 : tensor<16x16xi1, #blocked2>, tensor<16x16xf32, #blocked2>
975|   %46 = "tt.reduce" (%45) ({
976|   ^bb0(%arg4: f32, %arg5: f32):
977|     %add = arith.addf %arg4, %arg5 : f32
978|     tt.reduce.return %add : f32
979|   }) {axis = 1 : i32} : (tensor<16x16xf32, #blocked2>) -> tensor<16xf32, #ttg.slice<{dim = 1, parent = #blocked2}>>
980|   %47 = ttg.convert_layout %46 : tensor<16xf32, #ttg.slice<{dim = 1, parent = #blocked2}>> -> tensor<16xf32, #blocked0>
981|   %48 = ttg.convert_layout %47 : tensor<16xf32, #blocked0> -> tensor<16xf32, #ttg.slice<{dim = 1, parent = #blocked1}>>
982|   %49 = tt.expand_dims %48 {axis = 1 : i32} : tensor<16xf32, #ttg.slice<{dim = 1, parent = #blocked1}>> -> tensor<16x1xf32, #blocked1>
983|   %50 = ttg.convert_layout %49 : tensor<16x1xf32, #blocked1> -> tensor<16x1xf32, #blocked2>
984|   %51 = ttg.convert_layout %18 : tensor<16x16x!tt.ptr<f32>, #blocked2> -> tensor<16x16x!tt.ptr<f32>, #blocked4>
985|   %52 = ttg.convert_layout %22 : tensor<16x16xi1, #blocked2> -> tensor<16x16xi1, #blocked4>
986|   %53 = tt.load %51, %52 : tensor<16x16x!tt.ptr<f32>, #blocked4>
987|   %54 = ttg.convert_layout %53 : tensor<16x16xf32, #blocked4> -> tensor<16x16xf32, #blocked2>
988|   %55 = arith.subf %54, %41 : tensor<16x16xf32, #blocked2>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, arith.subf, floating-point additions, math.exp, arith.select. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、arith.subf、浮点加法、math.exp、arith.select。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 989-1001
```mlir
 989|   %56 = math.log %50 : tensor<16x1xf32, #blocked2>
 990|   %57 = tt.broadcast %56 : tensor<16x1xf32, #blocked2> -> tensor<16x16xf32, #blocked2>
 991|   %58 = arith.subf %55, %57 : tensor<16x16xf32, #blocked2>
 992|   %59 = tt.splat %arg1 : !tt.ptr<f32> -> tensor<16x16x!tt.ptr<f32>, #blocked2>
 993|   %60 = tt.addptr %59, %16 : tensor<16x16x!tt.ptr<f32>, #blocked2>, tensor<16x16xi32, #blocked2>
 994|   %61 = ttg.convert_layout %60 : tensor<16x16x!tt.ptr<f32>, #blocked2> -> tensor<16x16x!tt.ptr<f32>, #blocked4>
 995|   %62 = ttg.convert_layout %58 : tensor<16x16xf32, #blocked2> -> tensor<16x16xf32, #blocked4>
 996|   %63 = ttg.convert_layout %22 : tensor<16x16xi1, #blocked2> -> tensor<16x16xi1, #blocked4>
 997|   tt.store %61, %62, %63 : tensor<16x16x!tt.ptr<f32>, #blocked4>
 998|   tt.return
 999| }
1000| }
1001| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, math.log, tensor broadcasting, arith.subf, broadcasted scalars or pointers. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、math.log、张量广播、arith.subf、广播后的标量或指针。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1002-1002
```mlir
1002| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1003-1010
```mlir
1003| 
1004| #blocked0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [8], order = [0]}>
1005| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [8, 1], order = [0, 1]}>
1006| #blocked2 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [2, 4], order = [0, 1]}>
1007| #blocked3 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 8], order = [0, 1]}>
1008| #blocked4 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [8, 1], order = [1, 0]}>
1009| #blocked5 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [2, 16], warpsPerCTA = [8, 1], order = [1, 0]}>
1010| // cmpf and cmpi have different operands and result types
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1011-1011
```mlir
1011| // CHECK-LABEL: cmp
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: cmp anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: cmp 这样的标签用于锚定匹配范围。

### Lines 1012-1012
```mlir
1012| module attributes {"ttg.num-warps" = 8 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 1013-1030
```mlir
1013| tt.func public @cmp(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg2: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg3: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg4: i32 {tt.divisibility = 16 : i32}, %arg5: i32 {tt.divisibility = 16 : i32}) {
1014|   %c64 = arith.constant 64 : i32
1015|   %c2048 = arith.constant 2048 : i32
1016|   %c0 = arith.constant 0 : i32
1017|   %c64_i32 = arith.constant 64 : i32
1018|   %cst = arith.constant dense<-3.40282347E+38> : tensor<64x64xf32, #blocked2>
1019|   %cst_0 = arith.constant dense<4194304> : tensor<64x1xi32, #blocked2>
1020|   %cst_1 = arith.constant dense<12> : tensor<64x1xi32, #blocked2>
1021|   %cst_2 = arith.constant dense<2048> : tensor<1x64xi32, #blocked3>
1022|   %cst_3 = arith.constant dense<0> : tensor<64x64xi32, #blocked2>
1023|   %cst_4 = arith.constant dense<2048> : tensor<64x1xi32, #blocked2>
1024|   %cst_5 = arith.constant dense<49152> : tensor<64x1xi32, #blocked2>
1025|   %cst_6 = arith.constant dense<0.000000e+00> : tensor<64x64xf32, #blocked2>
1026|   %0 = tt.get_program_id x : i32
1027|   %1 = arith.muli %0, %c64_i32 : i32
1028|   %2 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #blocked0>
1029|   %3 = ttg.convert_layout %2 : tensor<64xi32, #blocked0> -> tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked1}>>
1030|   %4 = tt.expand_dims %3 {axis = 1 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked1}>> -> tensor<64x1xi32, #blocked1>
```
**EN:** This function-oriented block defines or enters `cmp`. Within it, the test exercises constants, tt.func, program IDs, integer multiplications, lane/block index ranges, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `cmp` 为核心。测试在其中演示 常量、tt.func、程序 ID、整数乘法、lane/block 索引范围，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1031-1048
```mlir
1031|   %5 = ttg.convert_layout %4 : tensor<64x1xi32, #blocked1> -> tensor<64x1xi32, #blocked2>
1032|   %6 = tt.splat %1 : i32 -> tensor<64x1xi32, #blocked2>
1033|   %7 = arith.addi %6, %5 : tensor<64x1xi32, #blocked2>
1034|   %8 = arith.cmpi "slt", %7, %cst_5 : tensor<64x1xi32, #blocked2>
1035|   %9 = ttg.convert_layout %2 : tensor<64xi32, #blocked0> -> tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked3}>>
1036|   %10 = tt.expand_dims %9 {axis = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked3}>> -> tensor<1x64xi32, #blocked3>
1037|   %11 = arith.remsi %7, %cst_4 : tensor<64x1xi32, #blocked2>
1038|   %12 = arith.divsi %7, %cst_4 : tensor<64x1xi32, #blocked2>
1039|   %13 = arith.sitofp %cst_3 : tensor<64x64xi32, #blocked2> to tensor<64x64xf32, #blocked2>
1040|   %14 = arith.addf %13, %cst_6 : tensor<64x64xf32, #blocked2>
1041|   %15 = arith.muli %7, %cst_4 : tensor<64x1xi32, #blocked2>
1042|   %16 = tt.broadcast %15 : tensor<64x1xi32, #blocked2> -> tensor<64x64xi32, #blocked2>
1043|   %17 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<64x64x!tt.ptr<f16>, #blocked2>
1044|   %18 = tt.broadcast %8 : tensor<64x1xi1, #blocked2> -> tensor<64x64xi1, #blocked2>
1045|   %19 = arith.muli %11, %cst_4 : tensor<64x1xi32, #blocked2>
1046|   %20 = tt.broadcast %19 : tensor<64x1xi32, #blocked2> -> tensor<64x64xi32, #blocked2>
1047|   %21 = arith.divsi %12, %cst_1 : tensor<64x1xi32, #blocked2>
1048|   %22 = arith.muli %21, %cst_0 : tensor<64x1xi32, #blocked2>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on integer multiplications, tensor broadcasting, layout conversions, broadcasted scalars or pointers, arith.divsi. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 整数乘法、张量广播、布局转换、广播后的标量或指针、arith.divsi。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1049-1066
```mlir
1049|   %23 = tt.broadcast %22 : tensor<64x1xi32, #blocked2> -> tensor<64x64xi32, #blocked2>
1050|   %24 = tt.splat %arg1 : !tt.ptr<f32> -> tensor<64x64x!tt.ptr<f32>, #blocked2>
1051|   %25 = scf.for %arg6 = %c0 to %c2048 step %c64 iter_args(%arg7 = %14) -> (tensor<64x64xf32, #blocked2>) : i32 {
1052|     %45 = tt.splat %arg6 : i32 -> tensor<1x64xi32, #blocked3>
1053|     %46 = arith.addi %45, %10 : tensor<1x64xi32, #blocked3>
1054|     %47 = arith.cmpi "slt", %46, %cst_2 : tensor<1x64xi32, #blocked3>
1055|     %48 = tt.broadcast %46 : tensor<1x64xi32, #blocked3> -> tensor<64x64xi32, #blocked3>
1056|     %49 = ttg.convert_layout %48 : tensor<64x64xi32, #blocked3> -> tensor<64x64xi32, #blocked2>
1057|     %50 = arith.addi %49, %16 : tensor<64x64xi32, #blocked2>
1058|     %51 = tt.addptr %17, %50 : tensor<64x64x!tt.ptr<f16>, #blocked2>, tensor<64x64xi32, #blocked2>
1059|     %52 = tt.broadcast %47 : tensor<1x64xi1, #blocked3> -> tensor<64x64xi1, #blocked3>
1060|     %53 = ttg.convert_layout %52 : tensor<64x64xi1, #blocked3> -> tensor<64x64xi1, #blocked2>
1061|     %54 = arith.andi %53, %18 : tensor<64x64xi1, #blocked2>
1062|     %55 = ttg.convert_layout %51 : tensor<64x64x!tt.ptr<f16>, #blocked2> -> tensor<64x64x!tt.ptr<f16>, #blocked4>
1063|     %56 = ttg.convert_layout %54 : tensor<64x64xi1, #blocked2> -> tensor<64x64xi1, #blocked4>
1064|     %57 = tt.load %55, %56 : tensor<64x64x!tt.ptr<f16>, #blocked4>
1065|     %58 = ttg.convert_layout %57 : tensor<64x64xf16, #blocked4> -> tensor<64x64xf16, #blocked2>
1066|     %59 = arith.extf %58 : tensor<64x64xf16, #blocked2> to tensor<64x64xf32, #blocked2>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining layout conversions, tensor broadcasting, broadcasted scalars or pointers, integer additions, structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 布局转换、张量广播、广播后的标量或指针、整数加法、结构化循环。

### Lines 1067-1084
```mlir
1067|     %60 = arith.addi %49, %20 : tensor<64x64xi32, #blocked2>
1068|     %61 = arith.addi %60, %23 : tensor<64x64xi32, #blocked2>
1069|     %62 = tt.addptr %24, %61 : tensor<64x64x!tt.ptr<f32>, #blocked2>, tensor<64x64xi32, #blocked2>
1070|     %63 = ttg.convert_layout %62 : tensor<64x64x!tt.ptr<f32>, #blocked2> -> tensor<64x64x!tt.ptr<f32>, #blocked5>
1071|     %64 = ttg.convert_layout %54 : tensor<64x64xi1, #blocked2> -> tensor<64x64xi1, #blocked5>
1072|     %65 = tt.load %63, %64 : tensor<64x64x!tt.ptr<f32>, #blocked5>
1073|     %66 = ttg.convert_layout %65 : tensor<64x64xf32, #blocked5> -> tensor<64x64xf32, #blocked2>
1074|     %67 = arith.addf %59, %66 : tensor<64x64xf32, #blocked2>
1075|     %68 = arith.cmpf "une", %67, %67 : tensor<64x64xf32, #blocked2>
1076|     %69 = arith.cmpf "ogt", %67, %cst : tensor<64x64xf32, #blocked2>
1077|     %70 = arith.select %69, %67, %cst : tensor<64x64xi1, #blocked2>, tensor<64x64xf32, #blocked2>
1078|     %71 = arith.select %68, %67, %70 : tensor<64x64xi1, #blocked2>, tensor<64x64xf32, #blocked2>
1079|     %72 = math.exp %71 : tensor<64x64xf32, #blocked2>
1080|     %73 = arith.addf %arg7, %72 : tensor<64x64xf32, #blocked2>
1081|     %74 = arith.select %54, %73, %arg7 : tensor<64x64xi1, #blocked2>, tensor<64x64xf32, #blocked2>
1082|     scf.yield %74 : tensor<64x64xf32, #blocked2>
1083|   }
1084|   %26 = "tt.reduce" (%25) ({
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining layout conversions, arith.select, integer additions, floating-point additions, floating-point comparisons.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 布局转换、arith.select、整数加法、浮点加法、浮点比较。

### Lines 1085-1102
```mlir
1085|   ^bb0(%arg8: f32, %arg9: f32):
1086|     %add = arith.addf %arg8, %arg9 : f32
1087|     tt.reduce.return %add : f32
1088|   }) {axis = 1 : i32} : (tensor<64x64xf32, #blocked2>) -> tensor<64xf32, #ttg.slice<{dim = 1, parent = #blocked2}>>
1089|   %27 = ttg.convert_layout %26 : tensor<64xf32, #ttg.slice<{dim = 1, parent = #blocked2}>> -> tensor<64xf32, #blocked0>
1090|   %28 = ttg.convert_layout %27 : tensor<64xf32, #blocked0> -> tensor<64xf32, #ttg.slice<{dim = 1, parent = #blocked1}>>
1091|   %29 = tt.expand_dims %28 {axis = 1 : i32} : tensor<64xf32, #ttg.slice<{dim = 1, parent = #blocked1}>> -> tensor<64x1xf32, #blocked1>
1092|   %30 = ttg.convert_layout %29 : tensor<64x1xf32, #blocked1> -> tensor<64x1xf32, #blocked2>
1093|   %31 = arith.muli %7, %cst_4 : tensor<64x1xi32, #blocked2>
1094|   %32 = tt.broadcast %31 : tensor<64x1xi32, #blocked2> -> tensor<64x64xi32, #blocked2>
1095|   %33 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<64x64x!tt.ptr<f16>, #blocked2>
1096|   %34 = tt.broadcast %8 : tensor<64x1xi1, #blocked2> -> tensor<64x64xi1, #blocked2>
1097|   %35 = arith.muli %11, %cst_4 : tensor<64x1xi32, #blocked2>
1098|   %36 = tt.broadcast %35 : tensor<64x1xi32, #blocked2> -> tensor<64x64xi32, #blocked2>
1099|   %37 = arith.divsi %12, %cst_1 : tensor<64x1xi32, #blocked2>
1100|   %38 = arith.muli %37, %cst_0 : tensor<64x1xi32, #blocked2>
1101|   %39 = tt.broadcast %38 : tensor<64x1xi32, #blocked2> -> tensor<64x64xi32, #blocked2>
1102|   %40 = tt.splat %arg1 : !tt.ptr<f32> -> tensor<64x64x!tt.ptr<f32>, #blocked2>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor broadcasting, layout conversions, integer multiplications, broadcasted scalars or pointers, floating-point additions. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量广播、布局转换、整数乘法、广播后的标量或指针、浮点加法。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1103-1120
```mlir
1103|   %41 = tt.broadcast %30 : tensor<64x1xf32, #blocked2> -> tensor<64x64xf32, #blocked2>
1104|   %42 = tt.splat %arg2 : !tt.ptr<f32> -> tensor<64x64x!tt.ptr<f32>, #blocked2>
1105|   %43 = tt.splat %arg3 : !tt.ptr<f16> -> tensor<64x64x!tt.ptr<f16>, #blocked2>
1106|   scf.for %arg6 = %c0 to %c2048 step %c64 : i32 {
1107|     %45 = tt.splat %arg6 : i32 -> tensor<1x64xi32, #blocked3>
1108|     %46 = arith.addi %45, %10 : tensor<1x64xi32, #blocked3>
1109|     %47 = arith.cmpi "slt", %46, %cst_2 : tensor<1x64xi32, #blocked3>
1110|     %48 = tt.broadcast %46 : tensor<1x64xi32, #blocked3> -> tensor<64x64xi32, #blocked3>
1111|     %49 = ttg.convert_layout %48 : tensor<64x64xi32, #blocked3> -> tensor<64x64xi32, #blocked2>
1112|     %50 = arith.addi %49, %32 : tensor<64x64xi32, #blocked2>
1113|     %51 = tt.addptr %33, %50 : tensor<64x64x!tt.ptr<f16>, #blocked2>, tensor<64x64xi32, #blocked2>
1114|     %52 = tt.broadcast %47 : tensor<1x64xi1, #blocked3> -> tensor<64x64xi1, #blocked3>
1115|     %53 = ttg.convert_layout %52 : tensor<64x64xi1, #blocked3> -> tensor<64x64xi1, #blocked2>
1116|     %54 = arith.andi %53, %34 : tensor<64x64xi1, #blocked2>
1117|     %55 = ttg.convert_layout %51 : tensor<64x64x!tt.ptr<f16>, #blocked2> -> tensor<64x64x!tt.ptr<f16>, #blocked4>
1118|     %56 = ttg.convert_layout %54 : tensor<64x64xi1, #blocked2> -> tensor<64x64xi1, #blocked4>
1119|     %57 = tt.load %55, %56 : tensor<64x64x!tt.ptr<f16>, #blocked4>
1120|     %58 = ttg.convert_layout %57 : tensor<64x64xf16, #blocked4> -> tensor<64x64xf16, #blocked2>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining layout conversions, tensor broadcasting, broadcasted scalars or pointers, integer additions, structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 布局转换、张量广播、广播后的标量或指针、整数加法、结构化循环。

### Lines 1121-1138
```mlir
1121|     %59 = arith.extf %58 : tensor<64x64xf16, #blocked2> to tensor<64x64xf32, #blocked2>
1122|     %60 = arith.addi %49, %36 : tensor<64x64xi32, #blocked2>
1123|     %61 = arith.addi %60, %39 : tensor<64x64xi32, #blocked2>
1124|     %62 = tt.addptr %40, %61 : tensor<64x64x!tt.ptr<f32>, #blocked2>, tensor<64x64xi32, #blocked2>
1125|     %63 = ttg.convert_layout %62 : tensor<64x64x!tt.ptr<f32>, #blocked2> -> tensor<64x64x!tt.ptr<f32>, #blocked5>
1126|     %64 = ttg.convert_layout %54 : tensor<64x64xi1, #blocked2> -> tensor<64x64xi1, #blocked5>
1127|     %65 = tt.load %63, %64 : tensor<64x64x!tt.ptr<f32>, #blocked5>
1128|     %66 = ttg.convert_layout %65 : tensor<64x64xf32, #blocked5> -> tensor<64x64xf32, #blocked2>
1129|     %67 = arith.addf %59, %66 : tensor<64x64xf32, #blocked2>
1130|     %68 = arith.cmpf "une", %67, %67 : tensor<64x64xf32, #blocked2>
1131|     %69 = arith.cmpf "ogt", %67, %cst : tensor<64x64xf32, #blocked2>
1132|     %70 = arith.select %69, %67, %cst : tensor<64x64xi1, #blocked2>, tensor<64x64xf32, #blocked2>
1133|     %71 = arith.select %68, %67, %70 : tensor<64x64xi1, #blocked2>, tensor<64x64xf32, #blocked2>
1134|     %72 = math.exp %71 : tensor<64x64xf32, #blocked2>
1135|     %73 = arith.divf %72, %41 : tensor<64x64xf32, #blocked2>
1136|     %74 = tt.addptr %42, %50 : tensor<64x64x!tt.ptr<f32>, #blocked2>, tensor<64x64xi32, #blocked2>
1137|     %75 = ttg.convert_layout %74 : tensor<64x64x!tt.ptr<f32>, #blocked2> -> tensor<64x64x!tt.ptr<f32>, #blocked5>
1138|     %76 = ttg.convert_layout %73 : tensor<64x64xf32, #blocked2> -> tensor<64x64xf32, #blocked5>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, integer additions, pointer arithmetic, floating-point comparisons, arith.select. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、整数加法、指针算术、浮点比较、arith.select。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1139-1151
```mlir
1139|     %77 = ttg.convert_layout %54 : tensor<64x64xi1, #blocked2> -> tensor<64x64xi1, #blocked5>
1140|     tt.store %75, %76, %77 : tensor<64x64x!tt.ptr<f32>, #blocked5>
1141|     %78 = tt.addptr %43, %50 : tensor<64x64x!tt.ptr<f16>, #blocked2>, tensor<64x64xi32, #blocked2>
1142|     %79 = arith.truncf %73 : tensor<64x64xf32, #blocked2> to tensor<64x64xf16, #blocked2>
1143|     %80 = ttg.convert_layout %78 : tensor<64x64x!tt.ptr<f16>, #blocked2> -> tensor<64x64x!tt.ptr<f16>, #blocked4>
1144|     %81 = ttg.convert_layout %79 : tensor<64x64xf16, #blocked2> -> tensor<64x64xf16, #blocked4>
1145|     %82 = ttg.convert_layout %54 : tensor<64x64xi1, #blocked2> -> tensor<64x64xi1, #blocked4>
1146|     tt.store %80, %81, %82 : tensor<64x64x!tt.ptr<f16>, #blocked4>
1147|   }
1148|   tt.return
1149| }
1150| }
1151| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, masked or vectorized stores, pointer arithmetic, arith.truncf, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、带掩码或向量化的存储、指针算术、arith.truncf、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1152-1152
```mlir
1152| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1153-1154
```mlir
1153| 
1154| // Just make sure it doesn't crash on non-tensor types.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1155-1155
```mlir
1155| // CHECK-LABEL: if_no_tensor
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: if_no_tensor anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: if_no_tensor 这样的标签用于锚定匹配范围。

### Lines 1156-1156
```mlir
1156| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 1157-1157
```mlir
1157| tt.func public @if_no_tensor(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg2: i32 {tt.divisibility = 16 : i32}, %arg3: !tt.ptr<i64> {tt.divisibility = 16 : i32}) {
```
**EN:** This function-oriented block defines or enters `if_no_tensor`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `if_no_tensor` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1158-1175
```mlir
1158|   // CHECK-NOT: ttg.convert_layout
1159|   %c-1_i64 = arith.constant -1 : i64
1160|   %cst = arith.constant 0.000000e+00 : f32
1161|   %c-1_i32 = arith.constant -1 : i32
1162|   %0 = tt.get_program_id x : i32
1163|   %1 = tt.addptr %arg3, %0 : !tt.ptr<i64>, i32
1164|   %2 = tt.load %1 : !tt.ptr<i64>
1165|   %3 = arith.cmpi eq, %2, %c-1_i64 : i64
1166|   %4 = arith.select %3, %c-1_i32, %arg2 : i32
1167|   %5 = scf.if %3 -> (!tt.ptr<f32>) {
1168|     scf.yield %arg0 : !tt.ptr<f32>
1169|   } else {
1170|     %10 = tt.addptr %arg0, %2 : !tt.ptr<f32>, i64
1171|     scf.yield %10 : !tt.ptr<f32>
1172|   }
1173|   %6 = arith.extsi %4 : i32 to i64
1174|   %7 = arith.cmpi slt, %2, %6 : i64
1175|   %8 = tt.load %5, %7, %cst : !tt.ptr<f32>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining constants, pointer arithmetic, masked or vectorized loads, integer comparisons, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 常量、指针算术、带掩码或向量化的加载、整数比较、循环/分支产出值。

### Lines 1176-1181
```mlir
1176|   %9 = tt.addptr %arg1, %0 : !tt.ptr<f32>, i32
1177|   tt.store %9, %8 : !tt.ptr<f32>
1178|   tt.return
1179| }
1180| }
1181| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on pointer arithmetic, masked or vectorized stores, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 指针算术、带掩码或向量化的存储、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1182-1182
```mlir
1182| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1183-1184
```mlir
1183| 
1184| // Check if the SimplifyReduceCvt rewriter pattern doesn't hang.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1185-1186
```mlir
1185| // CHECK-LABEL: reduce_cvt
1186| // CHECK-NOT: ttg.convert_layout
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: reduce_cvt anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: reduce_cvt 这样的标签用于锚定匹配范围。

### Lines 1187-1190
```mlir
1187| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 2], order = [0, 1]}>
1188| #blocked1 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [2], order = [0]}>
1189| #blocked2 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [2, 1], order = [0, 1]}>
1190| #blocked3 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [16, 2], warpsPerCTA = [2, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1191-1191
```mlir
1191| module attributes {"ttg.num-warps" = 2 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 1192-1209
```mlir
1192|   tt.func public @reduce_cvt1(%arg0: !tt.ptr<i64> {tt.divisibility = 16 : i32}, %arg1: i32, %arg2: i32) {
1193|     %cst = arith.constant dense<0> : tensor<1x2xi32, #blocked>
1194|     %cst_0 = arith.constant dense<2> : tensor<1x2xi32, #blocked>
1195|     %0 = tt.make_range {end = 2 : i32, start = 0 : i32} : tensor<2xi32, #blocked1>
1196|     %1 = ttg.convert_layout %0 : tensor<2xi32, #blocked1> -> tensor<2xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
1197|     %2 = tt.expand_dims %1 {axis = 0 : i32} : tensor<2xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x2xi32, #blocked>
1198|     %3 = arith.cmpi "slt", %2, %cst_0 : tensor<1x2xi32, #blocked>
1199|     %4 = "tt.reduce" (%cst) ({
1200|     ^bb0(%arg3: i32, %arg4: i32):
1201|       %add = arith.addi %arg3, %arg4 : i32
1202|       tt.reduce.return %add : i32
1203|     }) {axis = 1 : i32} : (tensor<1x2xi32, #blocked>) -> tensor<1xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
1204|     %5 = ttg.convert_layout %4 : tensor<1xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<1xi32, #blocked1>
1205|     %6 = ttg.convert_layout %5 : tensor<1xi32, #blocked1> -> tensor<1xi32, #ttg.slice<{dim = 1, parent = #blocked2}>>
1206|     %7 = tt.expand_dims %6 {axis = 1 : i32} : tensor<1xi32, #ttg.slice<{dim = 1, parent = #blocked2}>> -> tensor<1x1xi32, #blocked2>
1207|     %8 = ttg.convert_layout %7 : tensor<1x1xi32, #blocked2> -> tensor<1x1xi32, #blocked>
1208|     %9 = tt.splat %arg0 : !tt.ptr<i64> -> tensor<1x2x!tt.ptr<i64>, #blocked>
1209|     %10 = tt.addptr %9, %2 : tensor<1x2x!tt.ptr<i64>, #blocked>, tensor<1x2xi32, #blocked>
```
**EN:** This function-oriented block defines or enters `reduce_cvt1`. Within it, the test exercises layout conversions, tt.func, constants, shape expansion, lane/block index ranges, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `reduce_cvt1` 为核心。测试在其中演示 布局转换、tt.func、常量、形状扩展、lane/block 索引范围，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1210-1219
```mlir
1210|     %11 = tt.broadcast %8 : tensor<1x1xi32, #blocked> -> tensor<1x2xi32, #blocked>
1211|     %12 = arith.extsi %11 : tensor<1x2xi32, #blocked> to tensor<1x2xi64, #blocked>
1212|     %13 = ttg.convert_layout %10 : tensor<1x2x!tt.ptr<i64>, #blocked> -> tensor<1x2x!tt.ptr<i64>, #blocked3>
1213|     %14 = ttg.convert_layout %12 : tensor<1x2xi64, #blocked> -> tensor<1x2xi64, #blocked3>
1214|     %15 = ttg.convert_layout %3 : tensor<1x2xi1, #blocked> -> tensor<1x2xi1, #blocked3>
1215|     tt.store %13, %14, %15 : tensor<1x2x!tt.ptr<i64>, #blocked3>
1216|     tt.return
1217|   }
1218| }
1219| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, tensor broadcasting, arith.extsi, masked or vectorized stores, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、张量广播、arith.extsi、带掩码或向量化的存储、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1220-1220
```mlir
1220| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1221-1223
```mlir
1221| 
1222| // CHECK-LABEL: reduce_cvt2
1223| // Match the reduction
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1224-1231
```mlir
1224| // CHECK-NOT: ttg.convert_layout
1225| // CHECK: tt.reduce
1226| // CHECK-SAME: axis = 1
1227| // CHECK: (tensor<1x256xf32, #{{.*}}>) -> tensor<1xf32, #ttg.slice<{dim = 1, parent = #{{.*}}}>>
1228| // CHECK: ttg.convert_layout
1229| // CHECK: tt.expand_dims
1230| // CHECK-NOT: ttg.convert_layout
1231| // CHECK: tt.return
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 1232-1235
```mlir
1232| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [0, 1]}>
1233| #blocked1 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
1234| #blocked2 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
1235| #blocked3 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [2, 16], warpsPerCTA = [1, 4], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1236-1236
```mlir
1236| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 1237-1254
```mlir
1237|   tt.func public @reduce_cvt2(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg2: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg3: i32 {tt.divisibility = 16 : i32}, %arg4: i32 {tt.divisibility = 16 : i32}) {
1238|     %cst = arith.constant dense<0.000000e+00> : tensor<1x256xf32, #blocked>
1239|     %c3136_i32 = arith.constant 3136 : i32
1240|     %c256_i32 = arith.constant 256 : i32
1241|     %c0_i32 = arith.constant 0 : i32
1242|     %cst_0 = arith.constant dense<3.136000e+03> : tensor<1x1xf32, #blocked>
1243|     %cst_1 = arith.constant dense<50176> : tensor<1x256xi32, #blocked>
1244|     %cst_2 = arith.constant dense<196> : tensor<1x1xi32, #blocked>
1245|     %cst_3 = arith.constant dense<196> : tensor<1x256xi32, #blocked>
1246|     %cst_4 = arith.constant dense<3136> : tensor<1x256xi32, #blocked>
1247|     %cst_5 = arith.constant dense<256> : tensor<1x1xi32, #blocked>
1248|     %0 = tt.get_program_id x : i32
1249|     %1 = tt.make_range {end = 1 : i32, start = 0 : i32} : tensor<1xi32, #blocked1>
1250|     %2 = ttg.convert_layout %1 : tensor<1xi32, #blocked1> -> tensor<1xi32, #ttg.slice<{dim = 1, parent = #blocked2}>>
1251|     %3 = tt.expand_dims %2 {axis = 1 : i32} : tensor<1xi32, #ttg.slice<{dim = 1, parent = #blocked2}>> -> tensor<1x1xi32, #blocked2>
1252|     %4 = ttg.convert_layout %3 : tensor<1x1xi32, #blocked2> -> tensor<1x1xi32, #blocked>
1253|     %5 = tt.splat %0 : i32 -> tensor<1x1xi32, #blocked>
1254|     %6 = arith.addi %5, %4 : tensor<1x1xi32, #blocked>
```
**EN:** This function-oriented block defines or enters `reduce_cvt2`. Within it, the test exercises constants, tt.func, layout conversions, program IDs, lane/block index ranges, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `reduce_cvt2` 为核心。测试在其中演示 常量、tt.func、布局转换、程序 ID、lane/block 索引范围，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1255-1272
```mlir
1255|     %7 = arith.cmpi "slt", %6, %cst_5 : tensor<1x1xi32, #blocked>
1256|     %8 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #blocked1>
1257|     %9 = ttg.convert_layout %8 : tensor<256xi32, #blocked1> -> tensor<256xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
1258|     %10 = tt.expand_dims %9 {axis = 0 : i32} : tensor<256xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x256xi32, #blocked>
1259|     %11 = arith.muli %6, %cst_2 : tensor<1x1xi32, #blocked>
1260|     %12 = tt.broadcast %11 : tensor<1x1xi32, #blocked> -> tensor<1x256xi32, #blocked>
1261|     %13 = tt.splat %arg1 : !tt.ptr<f32> -> tensor<1x256x!tt.ptr<f32>, #blocked>
1262|     %14 = tt.broadcast %7 : tensor<1x1xi1, #blocked> -> tensor<1x256xi1, #blocked>
1263|     %15 = scf.for %arg5 = %c0_i32 to %c3136_i32 step %c256_i32 iter_args(%arg6 = %cst) -> (tensor<1x256xf32, #blocked>) : i32 {
1264|       %43 = tt.splat %arg5 : i32 -> tensor<1x256xi32, #blocked>
1265|       %44 = arith.addi %43, %10 : tensor<1x256xi32, #blocked>
1266|       %45 = arith.cmpi "slt", %44, %cst_4 : tensor<1x256xi32, #blocked>
1267|       %46 = arith.remsi %44, %cst_3 : tensor<1x256xi32, #blocked>
1268|       %47 = arith.divsi %44, %cst_3 : tensor<1x256xi32, #blocked>
1269|       %48 = arith.addi %46, %12 : tensor<1x256xi32, #blocked>
1270|       %49 = arith.muli %47, %cst_1 : tensor<1x256xi32, #blocked>
1271|       %50 = arith.addi %48, %49 : tensor<1x256xi32, #blocked>
1272|       %51 = tt.addptr %13, %50 : tensor<1x256x!tt.ptr<f32>, #blocked>, tensor<1x256xi32, #blocked>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining integer additions, integer comparisons, integer multiplications, tensor broadcasting, broadcasted scalars or pointers.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 整数加法、整数比较、整数乘法、张量广播、广播后的标量或指针。

### Lines 1273-1290
```mlir
1273|       %52 = arith.andi %45, %14 : tensor<1x256xi1, #blocked>
1274|       %53 = ttg.convert_layout %51 : tensor<1x256x!tt.ptr<f32>, #blocked> -> tensor<1x256x!tt.ptr<f32>, #blocked3>
1275|       %54 = ttg.convert_layout %52 : tensor<1x256xi1, #blocked> -> tensor<1x256xi1, #blocked3>
1276|       %55 = ttg.convert_layout %cst : tensor<1x256xf32, #blocked> -> tensor<1x256xf32, #blocked3>
1277|       %56 = tt.load %53, %54, %55 : tensor<1x256x!tt.ptr<f32>, #blocked3>
1278|       %57 = ttg.convert_layout %56 : tensor<1x256xf32, #blocked3> -> tensor<1x256xf32, #blocked>
1279|       %58 = arith.addf %arg6, %57 : tensor<1x256xf32, #blocked>
1280|       %59 = arith.select %52, %58, %arg6 : tensor<1x256xi1, #blocked>, tensor<1x256xf32, #blocked>
1281|       scf.yield %59 : tensor<1x256xf32, #blocked>
1282|     }
1283|     %16 = "tt.reduce" (%15) ({
1284|     ^bb0(%arg7: f32, %arg8: f32):
1285|       %add = arith.addf %arg7, %arg8 : f32
1286|       tt.reduce.return %add : f32
1287| 
1288|     }) {axis = 1 : i32} : (tensor<1x256xf32, #blocked>) -> tensor<1xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
1289|     %17 = ttg.convert_layout %16 : tensor<1xf32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<1xf32, #blocked1>
1290|     %18 = ttg.convert_layout %17 : tensor<1xf32, #blocked1> -> tensor<1xf32, #ttg.slice<{dim = 1, parent = #blocked2}>>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining layout conversions, floating-point additions, arith.andi, masked or vectorized loads, arith.select.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 布局转换、浮点加法、arith.andi、带掩码或向量化的加载、arith.select。

### Lines 1291-1303
```mlir
1291|     %19 = tt.expand_dims %18 {axis = 1 : i32} : tensor<1xf32, #ttg.slice<{dim = 1, parent = #blocked2}>> -> tensor<1x1xf32, #blocked2>
1292|     %20 = ttg.convert_layout %19 : tensor<1x1xf32, #blocked2> -> tensor<1x1xf32, #blocked>
1293|     %21 = arith.divf %20, %cst_0 : tensor<1x1xf32, #blocked>
1294|     %22 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<1x1x!tt.ptr<f32>, #blocked>
1295|     %23 = tt.addptr %22, %6 : tensor<1x1x!tt.ptr<f32>, #blocked>, tensor<1x1xi32, #blocked>
1296|     %24 = ttg.convert_layout %23 : tensor<1x1x!tt.ptr<f32>, #blocked> -> tensor<1x1x!tt.ptr<f32>, #blocked>
1297|     %25 = ttg.convert_layout %21 : tensor<1x1xf32, #blocked> -> tensor<1x1xf32, #blocked>
1298|     %26 = ttg.convert_layout %7 : tensor<1x1xi1, #blocked> -> tensor<1x1xi1, #blocked>
1299|     tt.store %24, %25, %26 : tensor<1x1x!tt.ptr<f32>, #blocked>
1300|     tt.return
1301|   }
1302| }
1303| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, shape expansion, arith.divf, broadcasted scalars or pointers, pointer arithmetic. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、形状扩展、arith.divf、广播后的标量或指针、指针算术。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1304-1304
```mlir
1304| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1305-1306
```mlir
1305| 
1306| // Ensure that RematerializeForward doesn't apply when a convert has multiple uses
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1307-1307
```mlir
1307| // CHECK-LABEL: loop_convert_multi_uses
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: loop_convert_multi_uses anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: loop_convert_multi_uses 这样的标签用于锚定匹配范围。

### Lines 1308-1312
```mlir
1308| #blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
1309| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [16, 2], warpsPerCTA = [1, 4], order = [0, 1]}>
1310| #blocked2 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
1311| #blocked3 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [0, 1]}>
1312| #blocked4 = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1313-1313
```mlir
1313| module attributes {"ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 1314-1331
```mlir
1314|   tt.func public @loop_convert_multi_uses(%arg0: i32 {tt.divisibility = 16 : i32}, %arg1: i32, %arg2: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg3: i32 {tt.divisibility = 16 : i32}, %arg4: i32 {tt.divisibility = 16 : i32}, %arg5: i32 {tt.divisibility = 16 : i32}, %arg6: i32 {tt.divisibility = 16 : i32}, %arg7: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg8: i32 {tt.divisibility = 16 : i32}, %arg9: i32, %arg10: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg11: i32 {tt.divisibility = 16 : i32}, %arg12: i32, %arg13: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg14: i32 {tt.divisibility = 16 : i32}, %arg15: i32 {tt.divisibility = 16 : i32}, %arg16: i32 {tt.divisibility = 16 : i32}) {
1315|     %cst = arith.constant dense<0xFF800000> : tensor<16xf32, #blocked>
1316|     %c1_i32 = arith.constant 1 : i32
1317|     %c0_i32 = arith.constant 0 : i32
1318|     %cst_0 = arith.constant dense<0.000000e+00> : tensor<16xf32, #blocked>
1319|     %cst_1 = arith.constant dense<1> : tensor<16xi32, #blocked>
1320|     %cst_2 = arith.constant dense<0.000000e+00> : tensor<16x16xf32, #blocked1>
1321|     %cst_3 = arith.constant dense<1> : tensor<16x1xi32, #blocked1>
1322|     %c16_i32 = arith.constant 16 : i32
1323|     %0 = tt.get_program_id x : i32
1324|     %1 = tt.get_program_id y : i32
1325|     %2 = arith.divsi %1, %arg0 : i32
1326|     %3 = arith.remsi %1, %arg0 : i32
1327|     %4 = tt.make_range {end = 16 : i32, start = 0 : i32} : tensor<16xi32, #blocked>
1328|     %5 = arith.muli %0, %c16_i32 : i32
1329|     %6 = tt.splat %5 : i32 -> tensor<16xi32, #blocked>
1330|     %7 = arith.addi %6, %4 : tensor<16xi32, #blocked>
1331|     %8 = arith.muli %2, %arg3 : i32
```
**EN:** This function-oriented block defines or enters `loop_convert_multi_uses`. Within it, the test exercises constants, tt.func, program IDs, integer multiplications, arith.divsi, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `loop_convert_multi_uses` 为核心。测试在其中演示 常量、tt.func、程序 ID、整数乘法、arith.divsi，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1332-1349
```mlir
1332|     %9 = arith.muli %3, %arg4 : i32
1333|     %10 = arith.addi %8, %9 : i32
1334|     %11 = ttg.convert_layout %7 : tensor<16xi32, #blocked> -> tensor<16xi32, #ttg.slice<{dim = 1, parent = #blocked2}>>
1335|     %12 = tt.expand_dims %11 {axis = 1 : i32} : tensor<16xi32, #ttg.slice<{dim = 1, parent = #blocked2}>> -> tensor<16x1xi32, #blocked2>
1336|     %13 = ttg.convert_layout %12 : tensor<16x1xi32, #blocked2> -> tensor<16x1xi32, #blocked1>
1337|     %14 = tt.splat %arg6 : i32 -> tensor<16x1xi32, #blocked1>
1338|     %15 = arith.muli %13, %14 : tensor<16x1xi32, #blocked1>
1339|     %16 = ttg.convert_layout %4 : tensor<16xi32, #blocked> -> tensor<16xi32, #ttg.slice<{dim = 0, parent = #blocked3}>>
1340|     %17 = tt.expand_dims %16 {axis = 0 : i32} : tensor<16xi32, #ttg.slice<{dim = 0, parent = #blocked3}>> -> tensor<1x16xi32, #blocked3>
1341|     %18 = tt.broadcast %15 : tensor<16x1xi32, #blocked1> -> tensor<16x16xi32, #blocked1>
1342|     %19 = tt.broadcast %17 : tensor<1x16xi32, #blocked3> -> tensor<16x16xi32, #blocked3>
1343|     %20 = ttg.convert_layout %19 : tensor<16x16xi32, #blocked3> -> tensor<16x16xi32, #blocked1>
1344|     %21 = arith.addi %18, %20 : tensor<16x16xi32, #blocked1>
1345|     %22 = tt.splat %arg2 : !tt.ptr<f16> -> tensor<16x16x!tt.ptr<f16>, #blocked1>
1346|     %23 = arith.cmpi "slt", %13, %cst_3 : tensor<16x1xi32, #blocked1>
1347|     %24 = tt.broadcast %23 : tensor<16x1xi1, #blocked1> -> tensor<16x16xi1, #blocked1>
1348|     %25 = arith.truncf %cst_2 : tensor<16x16xf32, #blocked1> to tensor<16x16xf16, #blocked1>
1349|     %26 = arith.muli %2, %arg11 : i32
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, integer multiplications, tensor broadcasting, integer additions, shape expansion. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、整数乘法、张量广播、整数加法、形状扩展。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1350-1367
```mlir
1350|     %27 = arith.muli %3, %arg12 : i32
1351|     %28 = arith.addi %26, %27 : i32
1352|     %29 = tt.splat %arg10 : !tt.ptr<f32> -> tensor<16x!tt.ptr<f32>, #blocked>
1353|     %30 = arith.cmpi "slt", %7, %cst_1 : tensor<16xi32, #blocked>
1354|     %31 = arith.muli %2, %arg8 : i32
1355|     %32 = arith.muli %3, %arg9 : i32
1356|     %33 = arith.addi %31, %32 : i32
1357|     %34 = tt.splat %arg7 : !tt.ptr<f32> -> tensor<16x!tt.ptr<f32>, #blocked>
1358|     %35:3 = scf.for %arg17 = %c0_i32 to %arg1 step %c1_i32 iter_args(%arg18 = %cst_2, %arg19 = %cst_0, %arg20 = %cst) -> (tensor<16x16xf32, #blocked1>, tensor<16xf32, #blocked>, tensor<16xf32, #blocked>)  : i32 {
1359|       %60 = arith.muli %arg17, %arg5 : i32
1360|       %61 = arith.addi %10, %60 : i32
1361|       %62 = tt.splat %61 : i32 -> tensor<16x16xi32, #blocked1>
1362|       %63 = arith.addi %62, %21 : tensor<16x16xi32, #blocked1>
1363|       %64 = tt.addptr %22, %63 : tensor<16x16x!tt.ptr<f16>, #blocked1>, tensor<16x16xi32, #blocked1>
1364|       %65 = ttg.convert_layout %64 : tensor<16x16x!tt.ptr<f16>, #blocked1> -> tensor<16x16x!tt.ptr<f16>, #blocked4>
1365|       %66 = ttg.convert_layout %24 : tensor<16x16xi1, #blocked1> -> tensor<16x16xi1, #blocked4>
1366|       %67 = ttg.convert_layout %25 : tensor<16x16xf16, #blocked1> -> tensor<16x16xf16, #blocked4>
1367|       %68 = tt.load %65, %66, %67 : tensor<16x16x!tt.ptr<f16>, #blocked4>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining integer multiplications, integer additions, broadcasted scalars or pointers, layout conversions, integer comparisons.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 整数乘法、整数加法、广播后的标量或指针、布局转换、整数比较。

### Lines 1368-1385
```mlir
1368|       %69 = ttg.convert_layout %68 : tensor<16x16xf16, #blocked4> -> tensor<16x16xf16, #blocked1>
1369|       %70 = arith.addi %28, %arg17 : i32
1370|       %71 = tt.splat %70 : i32 -> tensor<16xi32, #blocked>
1371|       %72 = arith.addi %71, %7 : tensor<16xi32, #blocked>
1372|       %73 = tt.addptr %29, %72 : tensor<16x!tt.ptr<f32>, #blocked>, tensor<16xi32, #blocked>
1373|       %74 = ttg.convert_layout %73 : tensor<16x!tt.ptr<f32>, #blocked> -> tensor<16x!tt.ptr<f32>, #blocked>
1374|       %75 = ttg.convert_layout %30 : tensor<16xi1, #blocked> -> tensor<16xi1, #blocked>
1375|       %76 = ttg.convert_layout %cst_0 : tensor<16xf32, #blocked> -> tensor<16xf32, #blocked>
1376|       %77 = tt.load %74, %75, %76 : tensor<16x!tt.ptr<f32>, #blocked>
1377|       %78 = arith.addi %33, %arg17 : i32
1378|       %79 = tt.splat %78 : i32 -> tensor<16xi32, #blocked>
1379|       %80 = arith.addi %79, %7 : tensor<16xi32, #blocked>
1380|       %81 = tt.addptr %34, %80 : tensor<16x!tt.ptr<f32>, #blocked>, tensor<16xi32, #blocked>
1381|       %82 = ttg.convert_layout %81 : tensor<16x!tt.ptr<f32>, #blocked> -> tensor<16x!tt.ptr<f32>, #blocked>
1382|       %83 = ttg.convert_layout %30 : tensor<16xi1, #blocked> -> tensor<16xi1, #blocked>
1383|       %84 = ttg.convert_layout %cst_0 : tensor<16xf32, #blocked> -> tensor<16xf32, #blocked>
1384|       %85 = tt.load %82, %83, %84 : tensor<16x!tt.ptr<f32>, #blocked>
1385|       %86 = arith.cmpf "ogt", %arg20, %85 : tensor<16xf32, #blocked>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, integer additions, broadcasted scalars or pointers, pointer arithmetic, masked or vectorized loads. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、整数加法、广播后的标量或指针、指针算术、带掩码或向量化的加载。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1386-1403
```mlir
1386|       %87 = arith.select %86, %arg20, %85 : tensor<16xi1, #blocked>, tensor<16xf32, #blocked>
1387|       %88 = arith.subf %arg20, %87 : tensor<16xf32, #blocked>
1388|       %89 = math.exp %88 : tensor<16xf32, #blocked>
1389|       %90 = arith.subf %85, %87 : tensor<16xf32, #blocked>
1390|       %91 = math.exp %90 : tensor<16xf32, #blocked>
1391|       %92 = arith.mulf %89, %arg19 : tensor<16xf32, #blocked>
1392|       %93 = arith.mulf %91, %77 : tensor<16xf32, #blocked>
1393|       %94 = arith.addf %92, %93 : tensor<16xf32, #blocked>
1394|       %95 = arith.divf %91, %94 : tensor<16xf32, #blocked>
1395|       %96 = arith.divf %arg19, %94 : tensor<16xf32, #blocked>
1396|       %97 = arith.mulf %96, %89 : tensor<16xf32, #blocked>
1397|       %98 = ttg.convert_layout %97 : tensor<16xf32, #blocked> -> tensor<16xf32, #ttg.slice<{dim = 1, parent = #blocked2}>>
1398|       %99 = tt.expand_dims %98 {axis = 1 : i32} : tensor<16xf32, #ttg.slice<{dim = 1, parent = #blocked2}>> -> tensor<16x1xf32, #blocked2>
1399|       %100 = ttg.convert_layout %99 : tensor<16x1xf32, #blocked2> -> tensor<16x1xf32, #blocked1>
1400|       %101 = tt.broadcast %100 : tensor<16x1xf32, #blocked1> -> tensor<16x16xf32, #blocked1>
1401|       %102 = arith.mulf %arg18, %101 : tensor<16x16xf32, #blocked1>
1402|       %103 = ttg.convert_layout %95 : tensor<16xf32, #blocked> -> tensor<16xf32, #ttg.slice<{dim = 1, parent = #blocked2}>>
1403|       %104 = tt.expand_dims %103 {axis = 1 : i32} : tensor<16xf32, #ttg.slice<{dim = 1, parent = #blocked2}>> -> tensor<16x1xf32, #blocked2>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on arith.mulf, layout conversions, arith.subf, math.exp, arith.divf. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 arith.mulf、布局转换、arith.subf、math.exp、arith.divf。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1404-1421
```mlir
1404|       %105 = ttg.convert_layout %104 : tensor<16x1xf32, #blocked2> -> tensor<16x1xf32, #blocked1>
1405|       %106 = tt.broadcast %105 : tensor<16x1xf32, #blocked1> -> tensor<16x16xf32, #blocked1>
1406|       %107 = arith.extf %69 : tensor<16x16xf16, #blocked1> to tensor<16x16xf32, #blocked1>
1407|       %108 = arith.mulf %107, %106 : tensor<16x16xf32, #blocked1>
1408|       %109 = arith.addf %102, %108 : tensor<16x16xf32, #blocked1>
1409|       scf.yield %109, %94, %87 : tensor<16x16xf32, #blocked1>, tensor<16xf32, #blocked>, tensor<16xf32, #blocked>
1410|     }
1411|     %36 = arith.muli %2, %arg14 : i32
1412|     %37 = arith.muli %3, %arg15 : i32
1413|     %38 = arith.addi %36, %37 : i32
1414|     %39 = ttg.convert_layout %7 : tensor<16xi32, #blocked> -> tensor<16xi32, #ttg.slice<{dim = 1, parent = #blocked2}>>
1415|     %40 = tt.expand_dims %39 {axis = 1 : i32} : tensor<16xi32, #ttg.slice<{dim = 1, parent = #blocked2}>> -> tensor<16x1xi32, #blocked2>
1416|     %41 = ttg.convert_layout %40 : tensor<16x1xi32, #blocked2> -> tensor<16x1xi32, #blocked1>
1417|     %42 = tt.splat %arg16 : i32 -> tensor<16x1xi32, #blocked1>
1418|     %43 = arith.muli %41, %42 : tensor<16x1xi32, #blocked1>
1419|     %44 = ttg.convert_layout %4 : tensor<16xi32, #blocked> -> tensor<16xi32, #ttg.slice<{dim = 0, parent = #blocked3}>>
1420|     %45 = tt.expand_dims %44 {axis = 0 : i32} : tensor<16xi32, #ttg.slice<{dim = 0, parent = #blocked3}>> -> tensor<1x16xi32, #blocked3>
1421|     %46 = tt.broadcast %43 : tensor<16x1xi32, #blocked1> -> tensor<16x16xi32, #blocked1>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining layout conversions, integer multiplications, tensor broadcasting, shape expansion, arith.extf.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 布局转换、整数乘法、张量广播、形状扩展、arith.extf。

### Lines 1422-1439
```mlir
1422|     %47 = tt.broadcast %45 : tensor<1x16xi32, #blocked3> -> tensor<16x16xi32, #blocked3>
1423|     %48 = ttg.convert_layout %47 : tensor<16x16xi32, #blocked3> -> tensor<16x16xi32, #blocked1>
1424|     %49 = arith.addi %46, %48 : tensor<16x16xi32, #blocked1>
1425|     %50 = tt.splat %38 : i32 -> tensor<16x16xi32, #blocked1>
1426|     %51 = arith.addi %50, %49 : tensor<16x16xi32, #blocked1>
1427|     %52 = tt.splat %arg13 : !tt.ptr<f16> -> tensor<16x16x!tt.ptr<f16>, #blocked1>
1428|     %53 = tt.addptr %52, %51 : tensor<16x16x!tt.ptr<f16>, #blocked1>, tensor<16x16xi32, #blocked1>
1429|     %54 = arith.cmpi "slt", %41, %cst_3 : tensor<16x1xi32, #blocked1>
1430|     %55 = tt.broadcast %54 : tensor<16x1xi1, #blocked1> -> tensor<16x16xi1, #blocked1>
1431|     %56 = arith.truncf %35#0 : tensor<16x16xf32, #blocked1> to tensor<16x16xf16, #blocked1>
1432|     %57 = ttg.convert_layout %53 : tensor<16x16x!tt.ptr<f16>, #blocked1> -> tensor<16x16x!tt.ptr<f16>, #blocked4>
1433|     %58 = ttg.convert_layout %56 : tensor<16x16xf16, #blocked1> -> tensor<16x16xf16, #blocked4>
1434|     %59 = ttg.convert_layout %55 : tensor<16x16xi1, #blocked1> -> tensor<16x16xi1, #blocked4>
1435|     tt.store %57, %58, %59 : tensor<16x16x!tt.ptr<f16>, #blocked4>
1436|     tt.return
1437|   }
1438| }
1439| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, tensor broadcasting, integer additions, broadcasted scalars or pointers, pointer arithmetic. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、张量广播、整数加法、广播后的标量或指针、指针算术。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1440-1440
```mlir
1440| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1441-1442
```mlir
1441| 
1442| // Check if MoveConvertOutOfLoop hangs because of adding additional conversions
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1443-1445
```mlir
1443| // CHECK-LABEL: @loop_print
1444| // CHECK-NOT: ttg.convert_layout
1445| //     CHECK: tt.return
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1446-1451
```mlir
1446| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [0, 1]}>
1447| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
1448| #blocked2 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
1449| #blocked3 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [0, 1]}>
1450| #blocked4 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
1451| #blocked5 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1452-1452
```mlir
1452| module attributes {"ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 1453-1470
```mlir
1453|   tt.func public @loop_print(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg2: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg3: i32 {tt.divisibility = 16 : i32}, %arg4: i32 {tt.divisibility = 16 : i32}, %arg5: i32 {tt.divisibility = 16 : i32}, %arg6: i32 {tt.divisibility = 16 : i32}) {
1454|     %c32_i32 = arith.constant 32 : i32
1455|     %c31_i32 = arith.constant 31 : i32
1456|     %c0_i32 = arith.constant 0 : i32
1457|     %c1_i32 = arith.constant 1 : i32
1458|     %cst = arith.constant dense<32> : tensor<32x128xi32, #blocked>
1459|     %cst_0 = arith.constant dense<32> : tensor<128x32xi32, #blocked1>
1460|     %cst_1 = arith.constant 0.000000e+00 : f32
1461|     %0 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #blocked2>
1462|     %1 = ttg.convert_layout %0 : tensor<128xi32, #blocked2> -> tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked1}>>
1463|     %2 = tt.expand_dims %1 {axis = 1 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked1}>> -> tensor<128x1xi32, #blocked1>
1464|     %3 = tt.splat %arg6 : i32 -> tensor<128x1xi32, #blocked1>
1465|     %4 = arith.muli %2, %3 : tensor<128x1xi32, #blocked1>
1466|     %5 = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #blocked2>
1467|     %6 = ttg.convert_layout %5 : tensor<32xi32, #blocked2> -> tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked3}>>
1468|     %7 = tt.expand_dims %6 {axis = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked3}>> -> tensor<1x32xi32, #blocked3>
1469|     %8 = tt.broadcast %4 : tensor<128x1xi32, #blocked1> -> tensor<128x32xi32, #blocked1>
1470|     %9 = tt.broadcast %7 : tensor<1x32xi32, #blocked3> -> tensor<128x32xi32, #blocked3>
```
**EN:** This function-oriented block defines or enters `loop_print`. Within it, the test exercises constants, tt.func, lane/block index ranges, layout conversions, shape expansion, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `loop_print` 为核心。测试在其中演示 常量、tt.func、lane/block 索引范围、布局转换、形状扩展，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1471-1488
```mlir
1471|     %10 = ttg.convert_layout %9 : tensor<128x32xi32, #blocked3> -> tensor<128x32xi32, #blocked1>
1472|     %11 = arith.addi %8, %10 : tensor<128x32xi32, #blocked1>
1473|     %12 = ttg.convert_layout %5 : tensor<32xi32, #blocked2> -> tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked1}>>
1474|     %13 = tt.expand_dims %12 {axis = 1 : i32} : tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked1}>> -> tensor<32x1xi32, #blocked1>
1475|     %14 = ttg.convert_layout %13 : tensor<32x1xi32, #blocked1> -> tensor<32x1xi32, #blocked>
1476|     %15 = ttg.convert_layout %0 : tensor<128xi32, #blocked2> -> tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked3}>>
1477|     %16 = tt.expand_dims %15 {axis = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked3}>> -> tensor<1x128xi32, #blocked3>
1478|     %17 = tt.broadcast %14 : tensor<32x1xi32, #blocked> -> tensor<32x128xi32, #blocked>
1479|     %18 = tt.broadcast %16 : tensor<1x128xi32, #blocked3> -> tensor<32x128xi32, #blocked3>
1480|     %19 = ttg.convert_layout %18 : tensor<32x128xi32, #blocked3> -> tensor<32x128xi32, #blocked>
1481|     %20 = arith.addi %17, %19 : tensor<32x128xi32, #blocked>
1482|     %21 = arith.addi %arg5, %c31_i32 : i32
1483|     %22 = arith.divsi %21, %c32_i32 : i32
1484|     %23 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<128x32x!tt.ptr<f16>, #blocked1>
1485|     %24 = tt.splat %arg1 : !tt.ptr<f16> -> tensor<32x128x!tt.ptr<f16>, #blocked>
1486|     %25:3 = scf.for %arg7 = %c0_i32 to %22 step %c1_i32 iter_args(%arg8 = %cst_1, %arg9 = %11, %arg10 = %20) -> (f32, tensor<128x32xi32, #blocked1>, tensor<32x128xi32, #blocked>)  : i32 {
1487|       tt.print "a_offsets: " { hex = false, isSigned = array<i32: 0> } : %arg9 : tensor<128x32xi32, #blocked1>
1488|       %27 = tt.addptr %23, %arg9 : tensor<128x32x!tt.ptr<f16>, #blocked1>, tensor<128x32xi32, #blocked1>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining layout conversions, integer additions, shape expansion, tensor broadcasting, broadcasted scalars or pointers.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 布局转换、整数加法、形状扩展、张量广播、广播后的标量或指针。

### Lines 1489-1506
```mlir
1489|       %28 = ttg.convert_layout %27 : tensor<128x32x!tt.ptr<f16>, #blocked1> -> tensor<128x32x!tt.ptr<f16>, #blocked4>
1490|       %29 = tt.load %28 : tensor<128x32x!tt.ptr<f16>, #blocked4>
1491|       %30 = ttg.convert_layout %29 : tensor<128x32xf16, #blocked4> -> tensor<128x32xf16, #blocked1>
1492|       %31 = tt.addptr %24, %arg10 : tensor<32x128x!tt.ptr<f16>, #blocked>, tensor<32x128xi32, #blocked>
1493|       %32 = ttg.convert_layout %31 : tensor<32x128x!tt.ptr<f16>, #blocked> -> tensor<32x128x!tt.ptr<f16>, #blocked5>
1494|       %33 = tt.load %32 : tensor<32x128x!tt.ptr<f16>, #blocked5>
1495|       %34 = ttg.convert_layout %33 : tensor<32x128xf16, #blocked5> -> tensor<32x128xf16, #blocked>
1496|       %35 = "tt.reduce"(%30) <{axis = 0 : i32}> ({
1497|       ^bb0(%arg11: f16, %arg12: f16):
1498|         %46 = arith.addf %arg11, %arg12 : f16
1499|         tt.reduce.return %46 : f16
1500|       }) : (tensor<128x32xf16, #blocked1>) -> tensor<32xf16, #ttg.slice<{dim = 0, parent = #blocked1}>>
1501|       %36 = ttg.convert_layout %35 : tensor<32xf16, #ttg.slice<{dim = 0, parent = #blocked1}>> -> tensor<32xf16, #blocked2>
1502|       %37 = "tt.reduce"(%36) <{axis = 0 : i32}> ({
1503|       ^bb0(%arg11: f16, %arg12: f16):
1504|         %46 = arith.addf %arg11, %arg12 : f16
1505|         tt.reduce.return %46 : f16
1506|       }) : (tensor<32xf16, #blocked2>) -> f16
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, masked or vectorized loads, floating-point additions, reductions, pointer arithmetic. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、带掩码或向量化的加载、浮点加法、归约、指针算术。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1507-1524
```mlir
1507|       %38 = "tt.reduce"(%34) <{axis = 0 : i32}> ({
1508|       ^bb0(%arg11: f16, %arg12: f16):
1509|         %46 = arith.addf %arg11, %arg12 : f16
1510|         tt.reduce.return %46 : f16
1511|       }) : (tensor<32x128xf16, #blocked>) -> tensor<128xf16, #ttg.slice<{dim = 0, parent = #blocked}>>
1512|       %39 = ttg.convert_layout %38 : tensor<128xf16, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<128xf16, #blocked2>
1513|       %40 = "tt.reduce"(%39) <{axis = 0 : i32}> ({
1514|       ^bb0(%arg11: f16, %arg12: f16):
1515|         %46 = arith.addf %arg11, %arg12 : f16
1516|         tt.reduce.return %46 : f16
1517|       }) : (tensor<128xf16, #blocked2>) -> f16
1518|       %41 = arith.addf %37, %40 : f16
1519|       %42 = arith.extf %41 : f16 to f32
1520|       %43 = arith.addf %arg8, %42 : f32
1521|       %44 = arith.addi %arg9, %cst_0 : tensor<128x32xi32, #blocked1>
1522|       %45 = arith.addi %arg10, %cst : tensor<32x128xi32, #blocked>
1523|       scf.yield %43, %44, %45 : f32, tensor<128x32xi32, #blocked1>, tensor<32x128xi32, #blocked>
1524|     }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining floating-point additions, reductions, integer additions, layout conversions, arith.extf.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 浮点加法、归约、整数加法、布局转换、arith.extf。

### Lines 1525-1530
```mlir
1525|     %26 = arith.truncf %25#0 : f32 to f16
1526|     tt.store %arg2, %26 : !tt.ptr<f16>
1527|     tt.return
1528|   }
1529| }
1530| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on arith.truncf, masked or vectorized stores, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 arith.truncf、带掩码或向量化的存储、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1531-1531
```mlir
1531| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1532-1533
```mlir
1532| 
1533| // Check if SimplifyReduceCvt handles the cvt,reduce->reduce,cvt conversion but not the general push forward conversion
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1534-1537
```mlir
1534| // CHECK-LABEL: reduce_cvt3
1535| // CHECK: tt.dot
1536| // CHECK-NEXT: tt.reduce
1537| // CHECK: ttg.convert_layout
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: reduce_cvt3 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: reduce_cvt3 这样的标签用于锚定匹配范围。

### Lines 1538-1546
```mlir
1538| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [0, 1]}>
1539| #blocked1 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
1540| #blocked2 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
1541| #blocked3 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [0, 1]}>
1542| #blocked4 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
1543| #blocked5 = #ttg.blocked<{sizePerThread = [2, 2], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
1544| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0, 1]}>
1545| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
1546| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1547-1547
```mlir
1547| module attributes {"ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 1548-1565
```mlir
1548|   tt.func public @reduce_cvt3(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg2: !tt.ptr<i32> {tt.divisibility = 16 : i32}) {
1549|     %cst = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #blocked>
1550|     %cst_0 = arith.constant dense<32> : tensor<32x1xi32, #blocked>
1551|     %0 = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #blocked1>
1552|     %1 = ttg.convert_layout %0 : tensor<32xi32, #blocked1> -> tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked2}>>
1553|     %2 = tt.expand_dims %1 {axis = 1 : i32} : tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked2}>> -> tensor<32x1xi32, #blocked2>
1554|     %3 = ttg.convert_layout %2 : tensor<32x1xi32, #blocked2> -> tensor<32x1xi32, #blocked>
1555|     %4 = arith.muli %3, %cst_0 : tensor<32x1xi32, #blocked>
1556|     %5 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<32x1x!tt.ptr<f16>, #blocked>
1557|     %6 = tt.addptr %5, %4 : tensor<32x1x!tt.ptr<f16>, #blocked>, tensor<32x1xi32, #blocked>
1558|     %7 = ttg.convert_layout %0 : tensor<32xi32, #blocked1> -> tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked3}>>
1559|     %8 = tt.expand_dims %7 {axis = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked3}>> -> tensor<1x32xi32, #blocked3>
1560|     %9 = tt.broadcast %6 : tensor<32x1x!tt.ptr<f16>, #blocked> -> tensor<32x32x!tt.ptr<f16>, #blocked>
1561|     %10 = tt.broadcast %8 : tensor<1x32xi32, #blocked3> -> tensor<32x32xi32, #blocked3>
1562|     %11 = ttg.convert_layout %10 : tensor<32x32xi32, #blocked3> -> tensor<32x32xi32, #blocked>
1563|     %12 = tt.addptr %9, %11 : tensor<32x32x!tt.ptr<f16>, #blocked>, tensor<32x32xi32, #blocked>
1564|     %13 = tt.splat %arg1 : !tt.ptr<f16> -> tensor<32x1x!tt.ptr<f16>, #blocked>
1565|     %14 = tt.addptr %13, %4 : tensor<32x1x!tt.ptr<f16>, #blocked>, tensor<32x1xi32, #blocked>
```
**EN:** This function-oriented block defines or enters `reduce_cvt3`. Within it, the test exercises layout conversions, pointer arithmetic, tt.func, constants, shape expansion, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `reduce_cvt3` 为核心。测试在其中演示 布局转换、指针算术、tt.func、常量、形状扩展，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1566-1583
```mlir
1566|     %15 = tt.broadcast %14 : tensor<32x1x!tt.ptr<f16>, #blocked> -> tensor<32x32x!tt.ptr<f16>, #blocked>
1567|     %16 = tt.addptr %15, %11 : tensor<32x32x!tt.ptr<f16>, #blocked>, tensor<32x32xi32, #blocked>
1568|     %17 = ttg.convert_layout %12 : tensor<32x32x!tt.ptr<f16>, #blocked> -> tensor<32x32x!tt.ptr<f16>, #blocked4>
1569|     %18 = tt.load %17 : tensor<32x32x!tt.ptr<f16>, #blocked4>
1570|     %19 = ttg.convert_layout %18 : tensor<32x32xf16, #blocked4> -> tensor<32x32xf16, #blocked>
1571|     %20 = ttg.convert_layout %16 : tensor<32x32x!tt.ptr<f16>, #blocked> -> tensor<32x32x!tt.ptr<f16>, #blocked4>
1572|     %21 = tt.load %20 : tensor<32x32x!tt.ptr<f16>, #blocked4>
1573|     %22 = ttg.convert_layout %21 : tensor<32x32xf16, #blocked4> -> tensor<32x32xf16, #blocked>
1574|     %23 = ttg.local_alloc %22 : (tensor<32x32xf16, #blocked>) -> !ttg.memdesc<32x32xf16, #shared, #smem>
1575|     %24 = ttg.memdesc_trans %23 {order=array<i32: 1,0>} : !ttg.memdesc<32x32xf16, #shared, #smem> -> !ttg.memdesc<32x32xf16, #shared1, #smem>
1576|     %25 = ttg.local_load %24 : !ttg.memdesc<32x32xf16, #shared1, #smem> -> tensor<32x32xf16, #blocked>
1577|     %26 = ttg.convert_layout %19 : tensor<32x32xf16, #blocked> -> tensor<32x32xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked5}>>
1578|     %27 = ttg.convert_layout %25 : tensor<32x32xf16, #blocked> -> tensor<32x32xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked5}>>
1579|     %28 = ttg.convert_layout %cst : tensor<32x32xf32, #blocked> -> tensor<32x32xf32, #blocked5>
1580|     %29 = tt.dot %26, %27, %28 : tensor<32x32xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked5}>> * tensor<32x32xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked5}>> -> tensor<32x32xf32, #blocked5>
1581|     %30 = ttg.convert_layout %29 : tensor<32x32xf32, #blocked5> -> tensor<32x32xf32, #blocked>
1582|     %31:2 = "tt.reduce"(%30, %11) <{axis = 1 : i32}> ({
1583|     ^bb0(%arg3: f32, %arg4: i32, %arg5: f32, %arg6: i32):
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, masked or vectorized loads, tensor broadcasting, pointer arithmetic, shared/local memory allocation. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、带掩码或向量化的加载、张量广播、指针算术、共享/本地内存分配。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1584-1601
```mlir
1584|       %37 = arith.cmpf "oeq", %arg3, %arg5 : f32
1585|       %38 = arith.cmpi "slt", %arg4, %arg6 : i32
1586|       %39 = arith.andi %37, %38 : i1
1587|       %40 = arith.cmpf "ogt", %arg3, %arg5 : f32
1588|       %41 = arith.ori %40, %39 : i1
1589|       %42 = arith.select %41, %arg3, %arg5 : f32
1590|       %43 = arith.select %41, %arg4, %arg6 : i32
1591|       tt.reduce.return %42, %43 : f32, i32
1592|     }) : (tensor<32x32xf32, #blocked>, tensor<32x32xi32, #blocked>) -> (tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>, tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked}>>)
1593|     %32 = ttg.convert_layout %31#1 : tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<32xi32, #blocked1>
1594|     %33 = tt.splat %arg2 : !tt.ptr<i32> -> tensor<32x!tt.ptr<i32>, #blocked1>
1595|     %34 = tt.addptr %33, %0 : tensor<32x!tt.ptr<i32>, #blocked1>, tensor<32xi32, #blocked1>
1596|     %35 = ttg.convert_layout %34 : tensor<32x!tt.ptr<i32>, #blocked1> -> tensor<32x!tt.ptr<i32>, #blocked1>
1597|     %36 = ttg.convert_layout %32 : tensor<32xi32, #blocked1> -> tensor<32xi32, #blocked1>
1598|     tt.store %35, %36 : tensor<32x!tt.ptr<i32>, #blocked1>
1599|     tt.return
1600|   }
1601| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, floating-point comparisons, arith.select, integer comparisons, arith.andi. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、浮点比较、arith.select、整数比较、arith.andi。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1604-1606
```mlir
1604| 
1605| 
1606| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1605-1606
```mlir
1605| 
1606| // Check that we don't have extra convert for flash attention IR.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1607-1619
```mlir
1607| #blocked = #ttg.blocked<{sizePerThread = [4, 4], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
1608| #blocked1 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
1609| #blocked2 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
1610| #blocked3 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
1611| #blocked3a = #ttg.blocked<{sizePerThread = [8], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
1612| #blocked4 = #ttg.blocked<{sizePerThread = [1, 1, 8], threadsPerWarp = [4, 1, 8], warpsPerCTA = [4, 1, 1], order = [1, 2, 0]}>
1613| #blocked4a = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
1614| #blocked5 = #ttg.blocked<{sizePerThread = [1, 1, 8], threadsPerWarp = [1, 4, 8], warpsPerCTA = [1, 4, 1], order = [0, 2, 1]}>
1615| #blocked6a = #ttg.blocked<{sizePerThread = [8], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
1616| #blocked6 = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [8, 4], warpsPerCTA = [1, 4], order = [0, 1]}>
1617| #blocked7 = #ttg.blocked<{sizePerThread = [8, 1, 1], threadsPerWarp = [8, 1, 4], warpsPerCTA = [1, 1, 4], order = [1, 0, 2]}>
1618| #blocked8 = #ttg.blocked<{sizePerThread = [1, 8, 1], threadsPerWarp = [1, 8, 4], warpsPerCTA = [1, 1, 4], order = [0, 1, 2]}>
1619| #blocked9 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1620-1620
```mlir
1620| module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 1621-1638
```mlir
1621|   tt.func public @attention_fw(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg2: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg3: f32, %arg4: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg5: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg6: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 8 : i32}, %arg7: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 8 : i32}, %arg8: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 8 : i32}, %arg9: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 8 : i32}, %arg10: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 8 : i32}, %arg11: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 8 : i32}, %arg12: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 8 : i32}, %arg13: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 8 : i32}, %arg14: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 8 : i32}, %arg15: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 8 : i32}, %arg16: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 8 : i32}, %arg17: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 8 : i32}, %arg18: i32, %arg19: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 8 : i32}, %arg20: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 8 : i32}, %arg21: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 8 : i32}) {
1622|     %c0_i64 = arith.constant 0 : i64
1623|     %c64_i64 = arith.constant 64 : i64
1624|     %cst = arith.constant dense<0.000000e+00> : tensor<128x64xf16, #blocked>
1625|     %cst_0 = arith.constant dense<0xFF800000> : tensor<128xf32, #blocked1>
1626|     %cst_1 = arith.constant dense<0.000000e+00> : tensor<128xf32, #blocked1>
1627|     %c64_i32 = arith.constant 64 : i32
1628|     %c0_i32 = arith.constant 0 : i32
1629|     %cst_2 = arith.constant dense<0.000000e+00> : tensor<128x64xf32, #blocked2>
1630|     %cst_3 = arith.constant 1.44269502 : f32
1631|     %c128_i32 = arith.constant 128 : i32
1632|     %0 = tt.get_program_id x : i32
1633|     %1 = tt.get_program_id y : i32
1634|     %2 = arith.muli %1, %arg7 : i32
1635|     %3 = arith.muli %1, %arg10 : i32
1636|     %4 = tt.addptr %arg0, %2 : !tt.ptr<f16>, i32
1637|     %5 = arith.muli %0, %c128_i32 : i32
1638|     %6 = arith.extsi %arg8 : i32 to i64
```
**EN:** This function-oriented block defines or enters `attention_fw`. Within it, the test exercises constants, integer multiplications, tt.func, program IDs, pointer arithmetic, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `attention_fw` 为核心。测试在其中演示 常量、整数乘法、tt.func、程序 ID、指针算术，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1639-1656
```mlir
1639|     %7 = arith.extsi %5 : i32 to i64
1640|     %8 = tt.addptr %arg1, %3 : !tt.ptr<f16>, i32
1641|     %9 = arith.addi %arg20, %arg21 : i32
1642|     %10 = arith.extsi %arg11 : i32 to i64
1643|     %11 = tt.addptr %arg2, %3 : !tt.ptr<f16>, i32
1644|     %12 = arith.extsi %arg14 : i32 to i64
1645|     %13 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #blocked1>
1646|     %14 = tt.splat %5 : i32 -> tensor<128xi32, #blocked1>
1647|     %15 = arith.addi %14, %13 : tensor<128xi32, #blocked1>
1648|     %16 = arith.mulf %arg3, %cst_3 : f32
1649|     %17 = tt.splat %4 : !tt.ptr<f16> -> tensor<128x64x!tt.ptr<f16>, #blocked3>
1650|     %18 = tt.splat %7 : i64 -> tensor<128xi64, #blocked3a>
1651|     %19 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #blocked3a>
1652|     %20 = arith.extsi %19 : tensor<128xi32, #blocked3a> to tensor<128xi64, #blocked3a>
1653|     %21 = arith.addi %18, %20 : tensor<128xi64, #blocked3a>
1654|     %22 = ttg.convert_layout %21 : tensor<128xi64, #blocked3a> -> tensor<128xi64, #ttg.slice<{dim = 1, parent = #blocked4a}>>
1655|     %23 = tt.expand_dims %22 {axis = 1 : i32} : tensor<128xi64, #ttg.slice<{dim = 1, parent = #blocked4a}>> -> tensor<128x1xi64, #blocked4a>
1656|     %24 = tt.splat %6 : i64 -> tensor<128x1xi64, #blocked4a>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on arith.extsi, broadcasted scalars or pointers, integer additions, pointer arithmetic, lane/block index ranges. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 arith.extsi、广播后的标量或指针、整数加法、指针算术、lane/block 索引范围。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1657-1673
```mlir
1657|     %25 = arith.muli %23, %24 : tensor<128x1xi64, #blocked4a>
1658|     %26 = tt.broadcast %25 : tensor<128x1xi64, #blocked4a> -> tensor<128x64xi64, #blocked4a>
1659|     %27 = ttg.convert_layout %26 : tensor<128x64xi64, #blocked4a> -> tensor<128x64xi64, #blocked3>
1660|     %28 = tt.addptr %17, %27 : tensor<128x64x!tt.ptr<f16>, #blocked3>, tensor<128x64xi64, #blocked3>
1661|     %29 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #blocked3a>
1662|     %30 = arith.extsi %29 : tensor<64xi32, #blocked3a> to tensor<64xi64, #blocked3a>
1663|     %31 = ttg.convert_layout %30 : tensor<64xi64, #blocked3a> -> tensor<64xi64, #ttg.slice<{dim = 0, parent = #blocked4a}>>
1664|     %32 = tt.expand_dims %31 {axis = 0 : i32} : tensor<64xi64, #ttg.slice<{dim = 0, parent = #blocked4a}>> -> tensor<1x64xi64, #blocked4a>
1665|     %33 = tt.broadcast %32 : tensor<1x64xi64, #blocked4a> -> tensor<128x64xi64, #blocked4a>
1666|     %34 = ttg.convert_layout %33 : tensor<128x64xi64, #blocked4a> -> tensor<128x64xi64, #blocked3>
1667|     %35 = tt.addptr %28, %34 : tensor<128x64x!tt.ptr<f16>, #blocked3>, tensor<128x64xi64, #blocked3>
1668|     %36 = tt.load %35 : tensor<128x64x!tt.ptr<f16>, #blocked3>
1669|     %37 = ttg.convert_layout %36 : tensor<128x64xf16, #blocked3> -> tensor<128x64xf16, #blocked2>
1670|     %38 = tt.splat %16 : f32 -> tensor<128x64xf32, #blocked2>
1671|     %39 = arith.extf %37 : tensor<128x64xf16, #blocked2> to tensor<128x64xf32, #blocked2>
1672|     %40 = arith.mulf %39, %38 : tensor<128x64xf32, #blocked2>
1673|     %41 = arith.truncf %40 : tensor<128x64xf32, #blocked2> to tensor<128x64xf16, #blocked2>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, tensor broadcasting, pointer arithmetic, integer multiplications, lane/block index ranges. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、张量广播、指针算术、整数乘法、lane/block 索引范围。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1674-1675
```mlir
1674| // CHECK-NOT: ttg.convert_layout
1675| //     CHECK: scf.for
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 1676-1678
```mlir
1676| // CHECK-NOT:   ttg.convert_layout
1677| //     CHECK:   ttg.convert_layout %{{.*}} #ttg.dot_op
1678| //     CHECK:   ttg.convert_layout %{{.*}} #ttg.dot_op
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1679-1680
```mlir
1679| // CHECK-NOT:   ttg.convert_layout
1680| //     CHECK:   tt.dot
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1681-1683
```mlir
1681| // CHECK-NOT:   ttg.convert_layout
1682| //     CHECK:   ttg.convert_layout %{{.*}} #ttg.dot_op
1683| //     CHECK:   ttg.convert_layout %{{.*}} #ttg.dot_op
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1684-1701
```mlir
1684| // CHECK-NOT:   ttg.convert_layout
1685| //     CHECK:   tt.dot
1686| //     CHECK:   scf.yield
1687|     %42:5 = scf.for %arg22 = %c0_i32 to %9 step %c64_i32 iter_args(%arg23 = %cst_2, %arg24 = %cst_1, %arg25 = %cst_0, %arg26 = %c0_i64, %arg27 = %c0_i64) -> (tensor<128x64xf32, #blocked2>, tensor<128xf32, #blocked1>, tensor<128xf32, #blocked1>, i64, i64)  : i32 {
1688|       %78 = tt.splat %8 : !tt.ptr<f16> -> tensor<64x64x!tt.ptr<f16>, #blocked6>
1689|       %79 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #blocked6a>
1690|       %80 = arith.extsi %79 : tensor<64xi32, #blocked6a> to tensor<64xi64, #blocked6a>
1691|       %81 = ttg.convert_layout %80 : tensor<64xi64, #blocked6a> -> tensor<64xi64, #ttg.slice<{dim = 1, parent = #blocked6}>>
1692|       %82 = tt.expand_dims %81 {axis = 1 : i32} : tensor<64xi64, #ttg.slice<{dim = 1, parent = #blocked6}>> -> tensor<64x1xi64, #blocked6>
1693|       %83 = tt.broadcast %82 : tensor<64x1xi64, #blocked6> -> tensor<64x64xi64, #blocked6>
1694|       %84 = ttg.convert_layout %83 : tensor<64x64xi64, #blocked6> -> tensor<64x64xi64, #blocked6>
1695|       %85 = tt.addptr %78, %84 : tensor<64x64x!tt.ptr<f16>, #blocked6>, tensor<64x64xi64, #blocked6>
1696|       %86 = tt.splat %arg26 : i64 -> tensor<64xi64, #blocked6a>
1697|       %87 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #blocked6a>
1698|       %88 = arith.extsi %87 : tensor<64xi32, #blocked6a> to tensor<64xi64, #blocked6a>
1699|       %89 = arith.addi %86, %88 : tensor<64xi64, #blocked6a>
1700|       %90 = ttg.convert_layout %89 : tensor<64xi64, #blocked6a> -> tensor<64xi64, #ttg.slice<{dim = 0, parent = #blocked6}>>
1701|       %91 = tt.expand_dims %90 {axis = 0 : i32} : tensor<64xi64, #ttg.slice<{dim = 0, parent = #blocked6}>> -> tensor<1x64xi64, #blocked6>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining layout conversions, broadcasted scalars or pointers, lane/block index ranges, arith.extsi, shape expansion.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 布局转换、广播后的标量或指针、lane/block 索引范围、arith.extsi、形状扩展。

### Lines 1702-1719
```mlir
1702|       %92 = tt.splat %10 : i64 -> tensor<1x64xi64, #blocked6>
1703|       %93 = arith.muli %91, %92 : tensor<1x64xi64, #blocked6>
1704|       %94 = tt.broadcast %93 : tensor<1x64xi64, #blocked6> -> tensor<64x64xi64, #blocked6>
1705|       %95 = ttg.convert_layout %94 : tensor<64x64xi64, #blocked6> -> tensor<64x64xi64, #blocked6>
1706|       %96 = tt.addptr %85, %95 : tensor<64x64x!tt.ptr<f16>, #blocked6>, tensor<64x64xi64, #blocked6>
1707|       %97 = tt.load %96 : tensor<64x64x!tt.ptr<f16>, #blocked6>
1708|       %98 = tt.splat %11 : !tt.ptr<f16> -> tensor<64x64x!tt.ptr<f16>, #blocked3>
1709|       %99 = tt.splat %arg27 : i64 -> tensor<64xi64, #blocked3a>
1710|       %100 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #blocked3a>
1711|       %101 = arith.extsi %100 : tensor<64xi32, #blocked3a> to tensor<64xi64, #blocked3a>
1712|       %102 = arith.addi %99, %101 : tensor<64xi64, #blocked3a>
1713|       %103 = ttg.convert_layout %102 : tensor<64xi64, #blocked3a> -> tensor<64xi64, #ttg.slice<{dim = 1, parent = #blocked3}>>
1714|       %104 = tt.expand_dims %103 {axis = 1 : i32} : tensor<64xi64, #ttg.slice<{dim = 1, parent = #blocked3}>> -> tensor<64x1xi64, #blocked3>
1715|       %105 = tt.splat %12 : i64 -> tensor<64x1xi64, #blocked3>
1716|       %106 = arith.muli %104, %105 : tensor<64x1xi64, #blocked3>
1717|       %107 = tt.broadcast %106 : tensor<64x1xi64, #blocked3> -> tensor<64x64xi64, #blocked3>
1718|       %108 = ttg.convert_layout %107 : tensor<64x64xi64, #blocked3> -> tensor<64x64xi64, #blocked3>
1719|       %109 = tt.addptr %98, %108 : tensor<64x64x!tt.ptr<f16>, #blocked3>, tensor<64x64xi64, #blocked3>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on broadcasted scalars or pointers, layout conversions, integer multiplications, tensor broadcasting, pointer arithmetic. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 广播后的标量或指针、布局转换、整数乘法、张量广播、指针算术。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1720-1737
```mlir
1720|       %110 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #blocked3a>
1721|       %111 = arith.extsi %110 : tensor<64xi32, #blocked3a> to tensor<64xi64, #blocked3a>
1722|       %112 = ttg.convert_layout %111 : tensor<64xi64, #blocked3a> -> tensor<64xi64, #ttg.slice<{dim = 0, parent = #blocked4a}>>
1723|       %113 = tt.expand_dims %112 {axis = 0 : i32} : tensor<64xi64, #ttg.slice<{dim = 0, parent = #blocked4a}>> -> tensor<1x64xi64, #blocked4a>
1724|       %114 = tt.broadcast %113 : tensor<1x64xi64, #blocked4a> -> tensor<64x64xi64, #blocked4a>
1725|       %115 = ttg.convert_layout %114 : tensor<64x64xi64, #blocked4a> -> tensor<64x64xi64, #blocked3>
1726|       %116 = tt.addptr %109, %115 : tensor<64x64x!tt.ptr<f16>, #blocked3>, tensor<64x64xi64, #blocked3>
1727|       %117 = tt.load %116 : tensor<64x64x!tt.ptr<f16>, #blocked3>
1728|       %118 = ttg.convert_layout %41 : tensor<128x64xf16, #blocked2> -> tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked}>>
1729|       %119 = ttg.convert_layout %97 : tensor<64x64xf16, #blocked6> -> tensor<64x64xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>
1730|       %120 = tt.dot %118, %119, %cst : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked}>> * tensor<64x64xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked}>> -> tensor<128x64xf16, #blocked>
1731|       %121 = ttg.convert_layout %120 : tensor<128x64xf16, #blocked> -> tensor<128x64xf16, #blocked2>
1732|       %122 = arith.extf %121 : tensor<128x64xf16, #blocked2> to tensor<128x64xf32, #blocked2>
1733|       %123 = "tt.reduce"(%122) <{axis = 1 : i32}> ({
1734|       ^bb0(%arg28: f32, %arg29: f32):
1735|         %153 = arith.maximumf %arg28, %arg29 : f32
1736|         tt.reduce.return %153 : f32
1737|       }) : (tensor<128x64xf32, #blocked2>) -> tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked2}>>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, lane/block index ranges, arith.extsi, shape expansion, tensor broadcasting. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、lane/block 索引范围、arith.extsi、形状扩展、张量广播。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1738-1755
```mlir
1738|       %124 = ttg.convert_layout %123 : tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked2}>> -> tensor<128xf32, #blocked1>
1739|       %125 = arith.maximumf %arg25, %124 : tensor<128xf32, #blocked1>
1740|       %126 = arith.subf %arg25, %125 : tensor<128xf32, #blocked1>
1741|       %127 = tt.extern_elementwise %126 {pure = true, libname = "libdevice", libpath = "/root/.pyenv/versions/3.9.9/lib/python3.9/site-packages/triton/language/../third_party/cuda/lib/libdevice.10.bc", symbol = "__nv_exp2f"} : (tensor<128xf32, #blocked1>) -> tensor<128xf32, #blocked1>
1742|       %128 = ttg.convert_layout %125 : tensor<128xf32, #blocked1> -> tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked9}>>
1743|       %129 = tt.expand_dims %128 {axis = 1 : i32} : tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked9}>> -> tensor<128x1xf32, #blocked9>
1744|       %130 = ttg.convert_layout %129 : tensor<128x1xf32, #blocked9> -> tensor<128x1xf32, #blocked2>
1745|       %131 = tt.broadcast %130 : tensor<128x1xf32, #blocked2> -> tensor<128x64xf32, #blocked2>
1746|       %132 = arith.subf %122, %131 : tensor<128x64xf32, #blocked2>
1747|       %133 = tt.extern_elementwise %132 {pure = true, libname = "libdevice", libpath = "/root/.pyenv/versions/3.9.9/lib/python3.9/site-packages/triton/language/../third_party/cuda/lib/libdevice.10.bc", symbol = "__nv_exp2f"} : (tensor<128x64xf32, #blocked2>) -> tensor<128x64xf32, #blocked2>
1748|       %134 = arith.mulf %arg24, %cst_1 : tensor<128xf32, #blocked1>
1749|       %135 = arith.addf %134, %127 : tensor<128xf32, #blocked1>
1750|       %136 = ttg.convert_layout %135 : tensor<128xf32, #blocked1> -> tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked9}>>
1751|       %137 = tt.expand_dims %136 {axis = 1 : i32} : tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked9}>> -> tensor<128x1xf32, #blocked9>
1752|       %138 = ttg.convert_layout %137 : tensor<128x1xf32, #blocked9> -> tensor<128x1xf32, #blocked2>
1753|       %139 = tt.broadcast %138 : tensor<128x1xf32, #blocked2> -> tensor<128x64xf32, #blocked2>
1754|       %140 = arith.mulf %arg23, %139 : tensor<128x64xf32, #blocked2>
1755|       %141 = arith.truncf %133 : tensor<128x64xf32, #blocked2> to tensor<128x64xf16, #blocked2>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, arith.subf, tt.extern_elementwise, shape expansion, tensor broadcasting. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、arith.subf、tt.extern_elementwise、形状扩展、张量广播。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1756-1773
```mlir
1756|       %142 = ttg.convert_layout %141 : tensor<128x64xf16, #blocked2> -> tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked}>>
1757|       %143 = ttg.convert_layout %117 : tensor<64x64xf16, #blocked3> -> tensor<64x64xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>
1758|       %144 = ttg.convert_layout %140 : tensor<128x64xf32, #blocked2> -> tensor<128x64xf32, #blocked>
1759|       %145 = tt.dot %142, %143, %144 : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked}>> * tensor<64x64xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked}>> -> tensor<128x64xf32, #blocked>
1760|       %146 = ttg.convert_layout %145 : tensor<128x64xf32, #blocked> -> tensor<128x64xf32, #blocked2>
1761|       %147 = arith.mulf %arg24, %127 : tensor<128xf32, #blocked1>
1762|       %148 = "tt.reduce"(%133) <{axis = 1 : i32}> ({
1763|       ^bb0(%arg28: f32, %arg29: f32):
1764|         %153 = arith.addf %arg28, %arg29 : f32
1765|         tt.reduce.return %153 : f32
1766|       }) : (tensor<128x64xf32, #blocked2>) -> tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked2}>>
1767|       %149 = ttg.convert_layout %148 : tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked2}>> -> tensor<128xf32, #blocked1>
1768|       %150 = arith.addf %147, %149 : tensor<128xf32, #blocked1>
1769|       %151 = arith.addi %arg26, %c64_i64 : i64
1770|       %152 = arith.addi %arg27, %c64_i64 : i64
1771|       scf.yield %146, %150, %125, %151, %152 : tensor<128x64xf32, #blocked2>, tensor<128xf32, #blocked1>, tensor<128xf32, #blocked1>, i64, i64
1772|     }
1773|     %43 = ttg.convert_layout %42#1 : tensor<128xf32, #blocked1> -> tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked9}>>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining layout conversions, floating-point additions, integer additions, dot-product or MMA-style math, arith.mulf.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 布局转换、浮点加法、整数加法、点积或 MMA 风格计算、arith.mulf。

### Lines 1774-1791
```mlir
1774|     %44 = tt.expand_dims %43 {axis = 1 : i32} : tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked9}>> -> tensor<128x1xf32, #blocked9>
1775|     %45 = ttg.convert_layout %44 : tensor<128x1xf32, #blocked9> -> tensor<128x1xf32, #blocked2>
1776|     %46 = tt.broadcast %45 : tensor<128x1xf32, #blocked2> -> tensor<128x64xf32, #blocked2>
1777|     %47 = arith.divf %42#0, %46 : tensor<128x64xf32, #blocked2>
1778|     %48 = arith.muli %1, %arg20 : i32
1779|     %49 = tt.addptr %arg4, %48 : !tt.ptr<f32>, i32
1780|     %50 = tt.splat %49 : !tt.ptr<f32> -> tensor<128x!tt.ptr<f32>, #blocked1>
1781|     %51 = tt.addptr %50, %15 : tensor<128x!tt.ptr<f32>, #blocked1>, tensor<128xi32, #blocked1>
1782|     %52 = tt.extern_elementwise %42#1 {pure = true, libname = "libdevice", libpath = "/root/.pyenv/versions/3.9.9/lib/python3.9/site-packages/triton/language/../third_party/cuda/lib/libdevice.10.bc", symbol = "__nv_log2f"} : (tensor<128xf32, #blocked1>) -> tensor<128xf32, #blocked1>
1783|     %53 = arith.addf %42#2, %52 : tensor<128xf32, #blocked1>
1784|     tt.store %51, %53 : tensor<128x!tt.ptr<f32>, #blocked1>
1785|     %54 = tt.addptr %arg5, %2 : !tt.ptr<f16>, i32
1786|     %55 = arith.extsi %arg17 : i32 to i64
1787|     %56 = arith.extsi %5 : i32 to i64
1788|     %57 = arith.truncf %47 : tensor<128x64xf32, #blocked2> to tensor<128x64xf16, #blocked2>
1789|     %58 = ttg.convert_layout %57 : tensor<128x64xf16, #blocked2> -> tensor<128x64xf16, #blocked3>
1790|     %59 = tt.splat %54 : !tt.ptr<f16> -> tensor<128x64x!tt.ptr<f16>, #blocked3>
1791|     %60 = tt.splat %56 : i64 -> tensor<128xi64, #blocked3a>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on pointer arithmetic, broadcasted scalars or pointers, layout conversions, arith.extsi, shape expansion. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 指针算术、广播后的标量或指针、布局转换、arith.extsi、形状扩展。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1792-1809
```mlir
1792|     %61 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #blocked3a>
1793|     %62 = arith.extsi %61 : tensor<128xi32, #blocked3a> to tensor<128xi64, #blocked3a>
1794|     %63 = arith.addi %60, %62 : tensor<128xi64, #blocked3a>
1795|     %64 = ttg.convert_layout %63 : tensor<128xi64, #blocked3a> -> tensor<128xi64, #ttg.slice<{dim = 1, parent = #blocked4a}>>
1796|     %65 = tt.expand_dims %64 {axis = 1 : i32} : tensor<128xi64, #ttg.slice<{dim = 1, parent = #blocked4a}>> -> tensor<128x1xi64, #blocked4a>
1797|     %66 = tt.splat %55 : i64 -> tensor<128x1xi64, #blocked4a>
1798|     %67 = arith.muli %65, %66 : tensor<128x1xi64, #blocked4a>
1799|     %68 = tt.broadcast %67 : tensor<128x1xi64, #blocked4a> -> tensor<128x64xi64, #blocked4a>
1800|     %69 = ttg.convert_layout %68 : tensor<128x64xi64, #blocked4a> -> tensor<128x64xi64, #blocked3>
1801|     %70 = tt.addptr %59, %69 : tensor<128x64x!tt.ptr<f16>, #blocked3>, tensor<128x64xi64, #blocked3>
1802|     %71 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #blocked3a>
1803|     %72 = arith.extsi %71 : tensor<64xi32, #blocked3a> to tensor<64xi64, #blocked3a>
1804|     %73 = ttg.convert_layout %72 : tensor<64xi64, #blocked3a> -> tensor<64xi64, #ttg.slice<{dim = 0, parent = #blocked6}>>
1805|     %74 = tt.expand_dims %73 {axis = 0 : i32} : tensor<64xi64, #ttg.slice<{dim = 0, parent = #blocked6}>> -> tensor<1x64xi64, #blocked6>
1806|     %75 = tt.broadcast %74 : tensor<1x64xi64, #blocked6> -> tensor<128x64xi64, #blocked6>
1807|     %76 = ttg.convert_layout %75 : tensor<128x64xi64, #blocked6> -> tensor<128x64xi64, #blocked3>
1808|     %77 = tt.addptr %70, %76 : tensor<128x64x!tt.ptr<f16>, #blocked3>, tensor<128x64xi64, #blocked3>
1809|     tt.store %77, %58 : tensor<128x64x!tt.ptr<f16>, #blocked3>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, lane/block index ranges, arith.extsi, shape expansion, tensor broadcasting. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、lane/block 索引范围、arith.extsi、形状扩展、张量广播。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1810-1813
```mlir
1810|     tt.return
1811|   }
1812| }
1813| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 1814-1814
```mlir
1814| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1815-1817
```mlir
1815| 
1816| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [0, 1]}>
1817| #blocked1 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1818-1818
```mlir
1818| // CHECK-LABEL: axis_mismatch
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: axis_mismatch anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: axis_mismatch 这样的标签用于锚定匹配范围。

### Lines 1819-1819
```mlir
1819| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 1820-1820
```mlir
1820| tt.func @axis_mismatch(%arg0: f32) -> tensor<1xf32, #ttg.slice<{dim = 0, parent = #blocked}>> {
```
**EN:** This function-oriented block defines or enters `axis_mismatch`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `axis_mismatch` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1821-1835
```mlir
1821| // CHECK: %[[R:.+]] = "tt.reduce"(%0) <{axis = 1 : i32}>
1822| // CHECK: %[[C:.+]] = ttg.convert_layout %[[R]]
1823| // CHECK: tt.return %[[C]]
1824|   %0 = tt.splat %arg0 : f32 -> tensor<1x16xf32, #blocked>
1825|   %1 = "tt.reduce"(%0) <{axis = 1 : i32}> ({
1826|     ^bb0(%arg9: f32, %arg10: f32):
1827|     %60 = arith.addf %arg9, %arg10 : f32
1828|     tt.reduce.return %60 : f32
1829|   }) : (tensor<1x16xf32, #blocked>) -> tensor<1xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
1830|   %2 = ttg.convert_layout %1 : tensor<1xf32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<1xf32, #blocked1>
1831|   %3 = ttg.convert_layout %2 : tensor<1xf32, #blocked1> -> tensor<1xf32, #ttg.slice<{dim = 0, parent = #blocked}>>
1832|   tt.return %3: tensor<1xf32, #ttg.slice<{dim = 0, parent = #blocked}>>
1833| }
1834| }
1835| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, broadcasted scalars or pointers, floating-point additions, reductions, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、广播后的标量或指针、浮点加法、归约、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1836-1836
```mlir
1836| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1837-1839
```mlir
1837| 
1838| #blocked = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
1839| #blocked1 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1840-1840
```mlir
1840| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 1841-1843
```mlir
1841| // CHECK-LABEL: reduce_to_scalar
1842| //   CHECK-NOT:   ttg.convert_layout
1843| //       CHECK:   tt.return
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1844-1861
```mlir
1844| tt.func @reduce_to_scalar(%ptr: tensor<1024x!tt.ptr<f32>, #blocked>) -> (f32, i32) {
1845|   %0 = tt.load %ptr : tensor<1024x!tt.ptr<f32>, #blocked>
1846|   %1 = ttg.convert_layout %0 : tensor<1024xf32, #blocked> -> tensor<1024xf32, #blocked1>
1847|   %2 = tt.make_range {end = 1024 : i32, start = 0 : i32} : tensor<1024xi32, #blocked1>
1848|   %3:2 = "tt.reduce"(%1, %2) <{axis = 0 : i32}> ({
1849|     ^bb0(%arg7: f32, %arg8: i32, %arg9: f32, %arg10: i32):
1850|     %51 = arith.cmpf "oeq", %arg7, %arg9 : f32
1851|     %52 = arith.cmpi "slt", %arg8, %arg10 : i32
1852|     %53 = arith.andi %51, %52 : i1
1853|     %54 = arith.cmpf "ogt", %arg7, %arg9 : f32
1854|     %55 = arith.ori %54, %53 : i1
1855|     %56 = arith.select %55, %arg7, %arg9 : f32
1856|     %57 = arith.select %55, %arg8, %arg10 : i32
1857|     tt.reduce.return %56, %57 : f32, i32
1858|   }) : (tensor<1024xf32, #blocked1>, tensor<1024xi32, #blocked1>) -> (f32, i32)
1859|   tt.return %3#0, %3#1: f32, i32
1860| }
1861| }
```
**EN:** This function-oriented block defines or enters `reduce_to_scalar`. Within it, the test exercises tt.func, floating-point comparisons, arith.select, masked or vectorized loads, layout conversions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `reduce_to_scalar` 为核心。测试在其中演示 tt.func、浮点比较、arith.select、带掩码或向量化的加载、布局转换，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1863-1864
```mlir
1863| 
1864| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1864-1866
```mlir
1864| 
1865| #blocked = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
1866| #blocked1 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1867-1867
```mlir
1867| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 1868-1877
```mlir
1868| // CHECK-LABEL: whileop
1869| //       CHECK: %[[L:.+]] = tt.load %{{.*}} : tensor<1024x!tt.ptr<f32>, #blocked>
1870| //       CHECK: %[[W:.+]] = scf.while (%[[I:.+]] = %[[L]], %{{.*}} = %{{.*}}) : (tensor<1024xf32, #blocked>, i1) -> tensor<1024xf32, #blocked> {
1871| //       CHECK:   scf.condition(%{{.*}}) %[[I]] : tensor<1024xf32, #blocked>
1872| //       CHECK: } do {
1873| //       CHECK: ^bb0(%[[ARG1:.+]]: tensor<1024xf32, #blocked>):
1874| //       CHECK:    %[[ADD:.+]] = arith.addf %[[ARG1]], %[[ARG1]] : tensor<1024xf32, #blocked>
1875| //       CHECK:    scf.yield %[[ADD]], %{{.*}} : tensor<1024xf32, #blocked>, i1
1876| //       CHECK:  }
1877| //       CHECK:  tt.store %{{.*}}, %[[W]] : tensor<1024x!tt.ptr<f32>, #blocked>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 1878-1895
```mlir
1878| tt.func @whileop(%ptr: tensor<1024x!tt.ptr<f32>, #blocked>, %cond: i1) {
1879|   %0 = tt.load %ptr : tensor<1024x!tt.ptr<f32>, #blocked>
1880|   %1 = ttg.convert_layout %0 : tensor<1024xf32, #blocked> -> tensor<1024xf32, #blocked1>
1881|   %2 = scf.while (%arg0 = %1, %arg1 = %cond) : (tensor<1024xf32, #blocked1>, i1) -> (tensor<1024xf32, #blocked1>) {
1882|       scf.condition(%arg1) %arg0 : tensor<1024xf32, #blocked1>
1883|     } do {
1884|     ^bb0(%arg0: tensor<1024xf32, #blocked1>):
1885|       %4 = ttg.convert_layout %arg0 : tensor<1024xf32, #blocked1> -> tensor<1024xf32, #blocked>
1886|       %5 = arith.addf %4, %4 : tensor<1024xf32, #blocked>
1887|       %6 = ttg.convert_layout %5 : tensor<1024xf32, #blocked> -> tensor<1024xf32, #blocked1>
1888|       scf.yield %6, %cond : tensor<1024xf32, #blocked1>, i1
1889|     }
1890|   %3 = ttg.convert_layout %2 : tensor<1024xf32, #blocked1> -> tensor<1024xf32, #blocked>
1891|   tt.store %ptr, %3 : tensor<1024x!tt.ptr<f32>, #blocked>
1892|   tt.return
1893| }
1894| }
1895| 
```
**EN:** This function-oriented block defines or enters `whileop`. Within it, the test exercises layout conversions, tt.func, masked or vectorized loads, scf.while, scf.condition, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `whileop` 为核心。测试在其中演示 布局转换、tt.func、带掩码或向量化的加载、scf.while、scf.condition，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1896-1896
```mlir
1896| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1897-1899
```mlir
1897| 
1898| #blocked = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
1899| #blocked1 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1900-1900
```mlir
1900| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 1901-1904
```mlir
1901| // CHECK-LABEL: whileop_backward_negative
1902| // CHECK: scf.while
1903| // CHECK:  scf.yield
1904| // CHECK: ttg.convert_layout
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: whileop_backward_negative anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: whileop_backward_negative 这样的标签用于锚定匹配范围。

### Lines 1905-1921
```mlir
1905| tt.func @whileop_backward_negative(%ptr: tensor<1024x!tt.ptr<i32>, #blocked>, %cond: i1) {
1906|   %1 = tt.make_range {end = 1024 : i32, start = 0 : i32} : tensor<1024xi32, #blocked1>
1907|   %2 = scf.while (%arg0 = %1, %arg1 = %cond) : (tensor<1024xi32, #blocked1>, i1) -> (tensor<1024xi32, #blocked1>) {
1908|       scf.condition(%arg1) %arg0 : tensor<1024xi32, #blocked1>
1909|     } do {
1910|     ^bb0(%arg0: tensor<1024xi32, #blocked1>):
1911|       %4 = ttg.convert_layout %arg0 : tensor<1024xi32, #blocked1> -> tensor<1024xi32, #blocked>
1912|       %5 = arith.addi %4, %4 : tensor<1024xi32, #blocked>
1913|       %6 = ttg.convert_layout %5 : tensor<1024xi32, #blocked> -> tensor<1024xi32, #blocked1>
1914|       scf.yield %6, %cond : tensor<1024xi32, #blocked1>, i1
1915|     }
1916|   %3 = ttg.convert_layout %2 : tensor<1024xi32, #blocked1> -> tensor<1024xi32, #blocked>
1917|   tt.store %ptr, %3 : tensor<1024x!tt.ptr<i32>, #blocked>
1918|   tt.return
1919| }
1920| }
1921| 
```
**EN:** This function-oriented block defines or enters `whileop_backward_negative`. Within it, the test exercises layout conversions, tt.func, lane/block index ranges, scf.while, scf.condition, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `whileop_backward_negative` 为核心。测试在其中演示 布局转换、tt.func、lane/block 索引范围、scf.while、scf.condition，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1922-1922
```mlir
1922| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1923-1936
```mlir
1923| 
1924| // Suppose we have a loop which yields a value from outside the loop:
1925| //   %x = ...
1926| //   %y = ...
1927| //   %z = for iter_args(%unused = %x) {
1928| //     yield %y
1929| //   }
1930| //   return %z
1931| //
1932| // This loop returns %y if it runs 1 or more times; otherwise, it returns %x.
1933| //
1934| // Check that we don't transform this loop into `yield %x` on the incorrect
1935| // theory that the yield is dead unless %x = %y.
1936| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1937-1938
```mlir
1937| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
1938| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 1939-1939
```mlir
1939| // CHECK-LABEL @yield_outside_loop1
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL @yield_outside_loop1 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL @yield_outside_loop1 这样的标签用于锚定匹配范围。

### Lines 1940-1950
```mlir
1940| tt.func public @yield_outside_loop1(%arg0: i32, %arg1: i32) -> (i32) {
1941|   %c0 = arith.constant 0 : index
1942|   %c5 = arith.constant 5 : index
1943|   %c1 = arith.constant 1 : index
1944|   %0 = scf.for %i = %c0 to %c5 step %c1 iter_args(%arg3 = %arg0) -> (i32) {
1945|     scf.yield %arg1 : i32
1946|   }
1947| 
1948|   // We should return %arg1, not %arg0.  (It would also be OK to return %0, if
1949|   // the loop didn't get eliminated.)
1950|   //
```
**EN:** This function-oriented block defines or enters `yield_outside_loop1`. Within it, the test exercises constants, tt.func, structured loops, loop/if yielded values, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `yield_outside_loop1` 为核心。测试在其中演示 常量、tt.func、结构化循环、循环/分支产出值，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1951-1954
```mlir
1951|   // CHECK: tt.return %arg1
1952|   tt.return %0 : i32
1953| }  // end function
1954| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1955-1955
```mlir
1955| // CHECK-LABEL @yield_outside_loop2
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL @yield_outside_loop2 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL @yield_outside_loop2 这样的标签用于锚定匹配范围。

### Lines 1956-1961
```mlir
1956| tt.func public @yield_outside_loop2(%arg0: i32, %arg1: i32) -> (i32, i32) {
1957|   %c0 = arith.constant 0 : index
1958|   %c5 = arith.constant 5 : index
1959|   %c1 = arith.constant 1 : index
1960|   %i0 = arith.constant 0 : i32
1961|   // Only yield a single value
```
**EN:** This function-oriented block defines or enters `yield_outside_loop2`. Within it, the test exercises constants, tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `yield_outside_loop2` 为核心。测试在其中演示 常量、tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1962-1972
```mlir
1962|   // CHECK: scf.yield %{{.*}} : i32
1963|   %0, %1 = scf.for %i = %c0 to %c5 step %c1 iter_args(%arg3 = %arg0, %sum = %i0) -> (i32, i32) {
1964|     %sum1 = arith.addi %sum, %arg3 : i32
1965|     scf.yield %arg0, %sum1 : i32, i32
1966|   }
1967| 
1968|   tt.return %0, %1 : i32, i32
1969| }  // end function
1970| 
1971| }  // end module
1972| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops, integer additions, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环、整数加法、循环/分支产出值、tt.return。

### Lines 1973-1973
```mlir
1973| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1974-1977
```mlir
1974| 
1975| // Check that we handle corner cases when hoisting conversions on top of extf because conversion operations on a smaller type are faster.
1976| // For complex slices we may hoist convert on top of extf while the source of extf has multiple uses in the slice.
1977| // In this case we want to make sure we don't replace other uses of extf source.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1978-1985
```mlir
1978| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
1979| #blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
1980| #blocked2 = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
1981| #blocked3 = #ttg.blocked<{sizePerThread = [4, 4], threadsPerWarp = [1, 32], warpsPerCTA = [2, 2], order = [1, 0]}>
1982| #mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [1, 4], instrShape = [16, 8]}>
1983| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0, 1]}>
1984| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
1985| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1986-1986
```mlir
1986| module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 1987-1990
```mlir
1987| // CHECK: [[$BLOCKED:#.*]] = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
1988| // CHECK: [[$MMA:#.*]] = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [1, 4], instrShape = [16, 8]}>
1989| 
1990| // CHECK-LABEL: @hoist_convert_above_extf_and_remat
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @hoist_convert_above_extf_and_remat anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @hoist_convert_above_extf_and_remat 这样的标签用于锚定匹配范围。

### Lines 1991-2008
```mlir
1991|   tt.func public @hoist_convert_above_extf_and_remat(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg2: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg3: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg4: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg5: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg6: !tt.ptr<f16>) {
1992|     %cst = arith.constant dense<256> : tensor<32x1xi32, #blocked>
1993|     %cst_0 = arith.constant dense<256> : tensor<32x1xi32, #blocked1>
1994|     %cst_1 = arith.constant dense<256> : tensor<256x1xi32, #blocked>
1995|     %c64_i32 = arith.constant 64 : i32
1996|     %c256_i32 = arith.constant 256 : i32
1997|     %c0_i32 = arith.constant 0 : i32
1998|     %cst_2 = arith.constant dense<1.000000e-03> : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked2}>>
1999|     %cst_3 = arith.constant dense<2.560000e+02> : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked2}>>
2000|     %cst_4 = arith.constant dense<0.000000e+00> : tensor<32x256xf32, #blocked3>
2001|     %c32_i32 = arith.constant 32 : i32
2002|     %0 = tt.get_program_id x : i32
2003|     %1 = arith.muli %0, %c32_i32 : i32
2004|     %2 = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
2005|     %3 = tt.expand_dims %2 {axis = 1 : i32} : tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<32x1xi32, #blocked>
2006|     %4 = tt.splat %1 : i32 -> tensor<32x1xi32, #blocked>
2007|     %5 = arith.addi %4, %3 : tensor<32x1xi32, #blocked>
2008|     %6 = arith.muli %5, %cst : tensor<32x1xi32, #blocked>
```
**EN:** This function-oriented block defines or enters `hoist_convert_above_extf_and_remat`. Within it, the test exercises constants, tt.func, integer multiplications, program IDs, lane/block index ranges, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `hoist_convert_above_extf_and_remat` 为核心。测试在其中演示 常量、tt.func、整数乘法、程序 ID、lane/block 索引范围，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 2009-2026
```mlir
2009|     %7 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
2010|     %8 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
2011|     %9 = tt.expand_dims %7 {axis = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x64xi32, #blocked>
2012|     %10 = tt.expand_dims %8 {axis = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x64xi32, #blocked>
2013|     %11 = tt.broadcast %9 : tensor<1x64xi32, #blocked> -> tensor<32x64xi32, #blocked>
2014|     %12 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
2015|     %13 = tt.expand_dims %12 {axis = 1 : i32} : tensor<256xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<256x1xi32, #blocked>
2016|     %14 = arith.muli %13, %cst_1 : tensor<256x1xi32, #blocked>
2017|     %15 = tt.broadcast %10 : tensor<1x64xi32, #blocked> -> tensor<256x64xi32, #blocked>
2018|     %16 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<32x64x!tt.ptr<f16>, #blocked>
2019|     %17 = tt.splat %arg1 : !tt.ptr<f16> -> tensor<256x64x!tt.ptr<f16>, #blocked>
2020|     %18 = scf.for %arg7 = %c0_i32 to %c256_i32 step %c64_i32 iter_args(%arg8 = %cst_4) -> (tensor<32x256xf32, #blocked3>)  : i32 {
2021|       %58 = tt.splat %arg7 : i32 -> tensor<32x1xi32, #blocked>
2022|       %59 = arith.addi %6, %58 : tensor<32x1xi32, #blocked>
2023|       %60 = tt.broadcast %59 : tensor<32x1xi32, #blocked> -> tensor<32x64xi32, #blocked>
2024|       %61 = arith.addi %60, %11 : tensor<32x64xi32, #blocked>
2025|       %62 = tt.splat %arg7 : i32 -> tensor<256x1xi32, #blocked>
2026|       %63 = arith.addi %14, %62 : tensor<256x1xi32, #blocked>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining broadcasted scalars or pointers, lane/block index ranges, shape expansion, tensor broadcasting, integer additions.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 广播后的标量或指针、lane/block 索引范围、形状扩展、张量广播、整数加法。

### Lines 2027-2044
```mlir
2027|       %64 = tt.broadcast %63 : tensor<256x1xi32, #blocked> -> tensor<256x64xi32, #blocked>
2028|       %65 = arith.addi %64, %15 : tensor<256x64xi32, #blocked>
2029|       %66 = tt.addptr %16, %61 : tensor<32x64x!tt.ptr<f16>, #blocked>, tensor<32x64xi32, #blocked>
2030|       %67 = tt.load %66 : tensor<32x64x!tt.ptr<f16>, #blocked>
2031|       %68 = tt.addptr %17, %65 : tensor<256x64x!tt.ptr<f16>, #blocked>, tensor<256x64xi32, #blocked>
2032|       %69 = tt.load %68 : tensor<256x64x!tt.ptr<f16>, #blocked>
2033|       %70 = ttg.local_alloc %69 : (tensor<256x64xf16, #blocked>) -> !ttg.memdesc<256x64xf16, #shared, #smem>
2034|       %71 = ttg.memdesc_trans %70 {order=array<i32: 1,0>} : !ttg.memdesc<256x64xf16, #shared, #smem> -> !ttg.memdesc<64x256xf16, #shared1, #smem>
2035|       %72 = ttg.convert_layout %67 : tensor<32x64xf16, #blocked> -> tensor<32x64xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked3}>>
2036|       %73 = ttg.local_load %71 : !ttg.memdesc<64x256xf16, #shared1, #smem> -> tensor<64x256xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked3}>>
2037|       %74 = ttg.convert_layout %arg8 : tensor<32x256xf32, #blocked3> -> tensor<32x256xf32, #mma>
2038|       %75 = ttg.convert_layout %72 : tensor<32x64xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked3}>> -> tensor<32x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
2039|       %76 = ttg.convert_layout %73 : tensor<64x256xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked3}>> -> tensor<64x256xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>
2040|       %77 = tt.dot %75, %76, %74 : tensor<32x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<64x256xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<32x256xf32, #mma>
2041|       %78 = ttg.convert_layout %77 : tensor<32x256xf32, #mma> -> tensor<32x256xf32, #blocked3>
2042|       scf.yield %78 : tensor<32x256xf32, #blocked3>
2043|     }
2044|     %19 = arith.truncf %18 : tensor<32x256xf32, #blocked3> to tensor<32x256xf16, #blocked3>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining layout conversions, pointer arithmetic, masked or vectorized loads, tensor broadcasting, integer additions.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 布局转换、指针算术、带掩码或向量化的加载、张量广播、整数加法。

### Lines 2045-2054
```mlir
2045|     %20 = ttg.convert_layout %19 : tensor<32x256xf16, #blocked3> -> tensor<32x256xf16, #blocked2>
2046|     %21 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #ttg.slice<{dim = 0, parent = #blocked2}>>
2047|     %22 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
2048|     %23 = tt.expand_dims %21 {axis = 0 : i32} : tensor<256xi32, #ttg.slice<{dim = 0, parent = #blocked2}>> -> tensor<1x256xi32, #blocked2>
2049|     %24 = tt.expand_dims %22 {axis = 0 : i32} : tensor<256xi32, #ttg.slice<{dim = 0, parent = #blocked1}>> -> tensor<1x256xi32, #blocked1>
2050|     %25 = tt.splat %arg2 : !tt.ptr<f16> -> tensor<1x256x!tt.ptr<f16>, #blocked2>
2051|     %26 = tt.addptr %25, %23 : tensor<1x256x!tt.ptr<f16>, #blocked2>, tensor<1x256xi32, #blocked2>
2052|     %27 = tt.load %26 : tensor<1x256x!tt.ptr<f16>, #blocked2>
2053|     %28 = tt.broadcast %27 : tensor<1x256xf16, #blocked2> -> tensor<32x256xf16, #blocked2>
2054|     %29 = arith.addf %20, %28 : tensor<32x256xf16, #blocked2>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on lane/block index ranges, shape expansion, layout conversions, broadcasted scalars or pointers, pointer arithmetic. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 lane/block 索引范围、形状扩展、布局转换、广播后的标量或指针、指针算术。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 2055-2072
```mlir
2055| // CHECK: %[[A:.+]] = ttg.convert_layout {{.*}} : tensor<1x256xf16, [[$BLOCKED]]> -> tensor<1x256xf16, [[$MMA]]>
2056| // CHECK: %[[B:.+]] = tt.broadcast %[[A]]
2057| // CHECK: %[[C:.+]] = arith.addf %[[B:.+]], {{.*}}
2058| // CHECK: arith.extf %[[C]] : tensor<32x256xf16, [[$MMA]]> to tensor<32x256xf32, [[$MMA]]>
2059|     %30 = arith.extf %29 : tensor<32x256xf16, #blocked2> to tensor<32x256xf32, #blocked2>
2060|     %31 = "tt.reduce"(%30) <{axis = 1 : i32}> ({
2061|     ^bb0(%arg7: f32, %arg8: f32):
2062|       %58 = arith.addf %arg7, %arg8 : f32
2063|       tt.reduce.return %58 : f32
2064|     }) : (tensor<32x256xf32, #blocked2>) -> tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked2}>>
2065|     %32 = arith.divf %31, %cst_3 : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked2}>>
2066|     %33 = arith.mulf %30, %30 : tensor<32x256xf32, #blocked2>
2067|     %34 = "tt.reduce"(%33) <{axis = 1 : i32}> ({
2068|     ^bb0(%arg7: f32, %arg8: f32):
2069|       %58 = arith.addf %arg7, %arg8 : f32
2070|       tt.reduce.return %58 : f32
2071|     }) : (tensor<32x256xf32, #blocked2>) -> tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked2}>>
2072|     %35 = arith.divf %34, %cst_3 : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked2}>>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on floating-point additions, reductions, arith.divf, arith.extf, arith.mulf. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 浮点加法、归约、arith.divf、arith.extf、arith.mulf。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 2073-2090
```mlir
2073|     %36 = arith.mulf %32, %32 : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked2}>>
2074|     %37 = arith.subf %35, %36 : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked2}>>
2075|     %38 = math.sqrt %37 : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked2}>>
2076|     %39 = arith.addf %38, %cst_2 : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked2}>>
2077|     %40 = tt.expand_dims %32 {axis = 1 : i32} : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked2}>> -> tensor<32x1xf32, #blocked2>
2078|     %41 = tt.expand_dims %39 {axis = 1 : i32} : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked2}>> -> tensor<32x1xf32, #blocked2>
2079|     %42 = tt.broadcast %40 : tensor<32x1xf32, #blocked2> -> tensor<32x256xf32, #blocked2>
2080|     %43 = arith.subf %30, %42 : tensor<32x256xf32, #blocked2>
2081|     %44 = tt.broadcast %41 : tensor<32x1xf32, #blocked2> -> tensor<32x256xf32, #blocked2>
2082|     %45 = arith.divf %43, %44 : tensor<32x256xf32, #blocked2>
2083|     %46 = arith.truncf %45 : tensor<32x256xf32, #blocked2> to tensor<32x256xf16, #blocked2>
2084|     %47 = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked1}>>
2085|     %48 = tt.expand_dims %47 {axis = 1 : i32} : tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked1}>> -> tensor<32x1xi32, #blocked1>
2086|     %49 = arith.muli %48, %cst_0 : tensor<32x1xi32, #blocked1>
2087|     %50 = tt.splat %1 : i32 -> tensor<32x1xi32, #blocked1>
2088|     %51 = arith.addi %50, %49 : tensor<32x1xi32, #blocked1>
2089|     %52 = tt.broadcast %51 : tensor<32x1xi32, #blocked1> -> tensor<32x256xi32, #blocked1>
2090|     %53 = tt.broadcast %24 : tensor<1x256xi32, #blocked1> -> tensor<32x256xi32, #blocked1>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor broadcasting, shape expansion, arith.subf, arith.mulf, math.sqrt. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量广播、形状扩展、arith.subf、arith.mulf、math.sqrt。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 2091-2099
```mlir
2091|     %54 = arith.addi %52, %53 : tensor<32x256xi32, #blocked1>
2092|     %55 = tt.splat %arg5 : !tt.ptr<f16> -> tensor<32x256x!tt.ptr<f16>, #blocked1>
2093|     %56 = tt.addptr %55, %54 : tensor<32x256x!tt.ptr<f16>, #blocked1>, tensor<32x256xi32, #blocked1>
2094|     %57 = ttg.convert_layout %46 : tensor<32x256xf16, #blocked2> -> tensor<32x256xf16, #blocked1>
2095|     tt.store %56, %57 : tensor<32x256x!tt.ptr<f16>, #blocked1>
2096|     tt.return
2097|   }
2098| }
2099| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on integer additions, broadcasted scalars or pointers, pointer arithmetic, layout conversions, masked or vectorized stores. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 整数加法、广播后的标量或指针、指针算术、布局转换、带掩码或向量化的存储。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 2100-2100
```mlir
2100| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 2101-2107
```mlir
2101| 
2102| // Minimal repro for https://github.com/pytorch/pytorch/issues/154933
2103| //
2104| // Check that if, during hoisting conversions over ext and broadcast ops,
2105| // we see multiple different layouts assigned to the same value, then we
2106| // skip propagation of that layout.
2107| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 2108-2108
```mlir
2108| // CHECK-LABEL: @hoist_on_ext_broadcast_mismatch
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @hoist_on_ext_broadcast_mismatch anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @hoist_on_ext_broadcast_mismatch 这样的标签用于锚定匹配范围。

### Lines 2109-2110
```mlir
2109| #blockedX = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
2110| #blockedY = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 2111-2111
```mlir
2111| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 2112-2122
```mlir
2112|   tt.func public @hoist_on_ext_broadcast_mismatch(%arg0: !tt.ptr<i32> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<i32> {tt.divisibility = 16 : i32}) -> tensor<4x1xi64, #blockedY> {
2113|     %c1_i32 = arith.constant 1 : i32
2114|     %c4_i32 = arith.constant 4 : i32
2115|     %c0_i32 = arith.constant 0 : i32
2116|     %0 = tt.make_range {end = 4 : i32, start = 0 : i32} : tensor<4xi32, #ttg.slice<{dim = 1, parent = #blockedX}>>
2117|     %cast0 = arith.extsi %0 : tensor<4xi32, #ttg.slice<{dim = 1, parent = #blockedX}>> to tensor<4xi64, #ttg.slice<{dim = 1, parent = #blockedX}>>
2118|     %1 = tt.splat %arg0 : !tt.ptr<i32> -> tensor<4x!tt.ptr<i32>, #ttg.slice<{dim = 1, parent = #blockedX}>>
2119|     %2 = tt.expand_dims %cast0 {axis = 1 : i32} : tensor<4xi64, #ttg.slice<{dim = 1, parent = #blockedX}>> -> tensor<4x1xi64, #blockedX>
2120|     %3 = tt.addptr %1, %cast0 : tensor<4x!tt.ptr<i32>, #ttg.slice<{dim = 1, parent = #blockedX}>>, tensor<4xi64, #ttg.slice<{dim = 1, parent = #blockedX}>>
2121|     %4 = tt.load %3 : tensor<4x!tt.ptr<i32>, #ttg.slice<{dim = 1, parent = #blockedX}>>
2122|     %5 = tt.reshape %4 : tensor<4xi32, #ttg.slice<{dim = 1, parent = #blockedX}>> -> tensor<4x1xi32, #blockedX>
```
**EN:** This function-oriented block defines or enters `hoist_on_ext_broadcast_mismatch`. Within it, the test exercises constants, tt.func, lane/block index ranges, arith.extsi, broadcasted scalars or pointers, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `hoist_on_ext_broadcast_mismatch` 为核心。测试在其中演示 常量、tt.func、lane/block 索引范围、arith.extsi、广播后的标量或指针，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 2123-2129
```mlir
2123|     // CHECK: arith.extsi
2124|     %6 = arith.extsi %5 : tensor<4x1xi32, #blockedX> to tensor<4x1xi64, #blockedX>
2125|     %7 = arith.addi %2, %6 : tensor<4x1xi64, #blockedX>
2126|     // for loop prevents fully hoisting the conversion.
2127|     %8 = scf.for %arg2 = %c0_i32 to %c4_i32 step %c1_i32 iter_args(%arg3 = %5) -> (tensor<4x1xi32, #blockedX>) : i32 {
2128|       scf.yield %5 : tensor<4x1xi32, #blockedX>
2129|     }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining arith.extsi, integer additions, structured loops, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 arith.extsi、整数加法、结构化循环、循环/分支产出值。

### Lines 2130-2137
```mlir
2130|     // CHECK: ttg.convert_layout
2131|     %9 = arith.extsi %8 : tensor<4x1xi32, #blockedX> to tensor<4x1xi64, #blockedX>
2132|     %10 = arith.addi %7, %9 : tensor<4x1xi64, #blockedX>
2133|     %11 = ttg.convert_layout %10 : tensor<4x1xi64, #blockedX> -> tensor<4x1xi64, #blockedY>
2134|     tt.return %11 : tensor<4x1xi64, #blockedY>
2135|   }
2136| }
2137| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on arith.extsi, integer additions, layout conversions, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 arith.extsi、整数加法、布局转换、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 2138-2138
```mlir
2138| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 2139-2142
```mlir
2139| 
2140| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [2, 1], order = [0, 1]}>
2141| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [2, 1], order = [1, 0]}>
2142| #blocked2 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 2], order = [0, 1]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 2143-2143
```mlir
2143| module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 2 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 2144-2146
```mlir
2144| // CHECK-LABEL: @backward_reduce_multiple_results
2145| //   CHECK-NOT:   ttg.convert_layout
2146| //       CHECK:   tt.return
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 2147-2162
```mlir
2147|   tt.func public @backward_reduce_multiple_results() -> tensor<1xi32, #ttg.slice<{dim = 1, parent = #blocked}>> {
2148|     %cst = arith.constant dense<0xFFF0000000000000> : tensor<1x32xf64, #blocked1>
2149|     %0 = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked2}>>
2150|     %1 = tt.expand_dims %0 {axis = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked2}>> -> tensor<1x32xi32, #blocked2>
2151|     %2 = ttg.convert_layout %1 : tensor<1x32xi32, #blocked2> -> tensor<1x32xi32, #blocked1>
2152|     %3:2 = "tt.reduce"(%cst, %2) <{axis = 1 : i32}> ({
2153|     ^bb0(%arg0: f64, %arg1: i32, %arg2: f64, %arg3: i32):
2154|       %5 = arith.addi %arg1, %arg3 : i32
2155|       %6 = arith.addf %arg0, %arg2 : f64
2156|       tt.reduce.return %6, %5 : f64, i32
2157|     }) : (tensor<1x32xf64, #blocked1>, tensor<1x32xi32, #blocked1>) -> (tensor<1xf64, #ttg.slice<{dim = 1, parent = #blocked1}>>, tensor<1xi32, #ttg.slice<{dim = 1, parent = #blocked1}>>)
2158|     %4 = ttg.convert_layout %3#1 : tensor<1xi32, #ttg.slice<{dim = 1, parent = #blocked1}>> -> tensor<1xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
2159|     tt.return %4 : tensor<1xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
2160| }
2161| }  // end module
2162| 
```
**EN:** This function-oriented block defines or enters `backward_reduce_multiple_results`. Within it, the test exercises tt.func, layout conversions, constants, lane/block index ranges, shape expansion, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `backward_reduce_multiple_results` 为核心。测试在其中演示 tt.func、布局转换、常量、lane/block 索引范围、形状扩展，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 2163-2163
```mlir
2163| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 2164-2169
```mlir
2164| 
2165| #blocked = #ttg.blocked<{sizePerThread = [1,2], threadsPerWarp = [32,1], warpsPerCTA = [1,1], order = [1,0]}>
2166| #blocked1 = #ttg.blocked<{sizePerThread = [1,1], threadsPerWarp = [16,2], warpsPerCTA = [1,1], order = [1,0]}>
2167| #blocked2 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
2168| #blocked3 = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
2169| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 2170-2170
```mlir
2170| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 2171-2171
```mlir
2171|   // CHECK-LABEL: @reshape_propagate
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @reshape_propagate anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @reshape_propagate 这样的标签用于锚定匹配范围。

### Lines 2172-2172
```mlir
2172|   tt.func public @reshape_propagate(%arg0: tensor<16x2xf32, #blocked>) -> tensor<32xf32, #blocked3> {
```
**EN:** This function-oriented block defines or enters `reshape_propagate`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `reshape_propagate` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 2173-2180
```mlir
2173|     // CHECK-NOT: ttg.convert_layout
2174|     %a = ttg.convert_layout %arg0 : tensor<16x2xf32, #blocked> -> tensor<16x2xf32, #blocked1>
2175|     %b = tt.reshape %a : tensor<16x2xf32, #blocked1> -> tensor<32xf32, #blocked2>
2176|     %c = ttg.convert_layout %b : tensor<32xf32, #blocked2> -> tensor<32xf32, #blocked3>
2177|     tt.return %c : tensor<32xf32, #blocked3>
2178|   }
2179| }
2180| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, tensor reshaping, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、张量重塑、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 2181-2181
```mlir
2181| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 2182-2186
```mlir
2182| 
2183| #blocked = #ttg.blocked<{sizePerThread = [1,2], threadsPerWarp = [32,1], warpsPerCTA = [1,1], order = [1,0]}>
2184| #blocked1 = #ttg.blocked<{sizePerThread = [1,1], threadsPerWarp = [16,2], warpsPerCTA = [1,1], order = [1,0]}>
2185| #blocked2 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
2186| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 2187-2187
```mlir
2187| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 2188-2188
```mlir
2188|   // CHECK-LABEL: @reshape_sink_convert
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @reshape_sink_convert anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @reshape_sink_convert 这样的标签用于锚定匹配范围。

### Lines 2189-2189
```mlir
2189|   tt.func public @reshape_sink_convert(%arg0: tensor<16x2xf32, #blocked>) -> tensor<32xf32, #blocked2> {
```
**EN:** This function-oriented block defines or enters `reshape_sink_convert`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `reshape_sink_convert` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 2190-2198
```mlir
2190|     // CHECK-NOT: ttg.convert_layout
2191|     // CHECK: tt.reshape
2192|     // CHECK: ttg.convert_layout
2193|     %a = ttg.convert_layout %arg0 : tensor<16x2xf32, #blocked> -> tensor<16x2xf32, #blocked1>
2194|     %b = tt.reshape %a : tensor<16x2xf32, #blocked1> -> tensor<32xf32, #blocked2>
2195|     tt.return %b : tensor<32xf32, #blocked2>
2196|   }
2197| }
2198| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, tensor reshaping, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、张量重塑、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 2199-2199
```mlir
2199| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 2200-2204
```mlir
2200| 
2201| #blocked = #ttg.blocked<{sizePerThread = [1,2], threadsPerWarp = [32,1], warpsPerCTA = [1,1], order = [1,0]}>
2202| #blocked1 = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
2203| #blocked2 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
2204| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 2205-2205
```mlir
2205| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 2206-2206
```mlir
2206|   // CHECK-LABEL: @permuting_reshape_propagate
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @permuting_reshape_propagate anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @permuting_reshape_propagate 这样的标签用于锚定匹配范围。

### Lines 2207-2207
```mlir
2207|   tt.func public @permuting_reshape_propagate(%arg0: tensor<16x2xf32, #blocked>) -> tensor<32xf16, #blocked2> {
```
**EN:** This function-oriented block defines or enters `permuting_reshape_propagate`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `permuting_reshape_propagate` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 2208-2217
```mlir
2208|     // CHECK-NOT: ttg.convert_layout
2209|     // CHECK: arith.truncf
2210|     // CHECK: ttg.convert_layout
2211|     %a = tt.reshape %arg0 allow_reorder efficient_layout : tensor<16x2xf32, #blocked> -> tensor<32xf32, #blocked1>
2212|     %b = ttg.convert_layout %a : tensor<32xf32, #blocked1> -> tensor<32xf32, #blocked2>
2213|     %c = arith.truncf %b : tensor<32xf32, #blocked2> to tensor<32xf16, #blocked2>
2214|     tt.return %c : tensor<32xf16, #blocked2>
2215|   }
2216| }
2217| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor reshaping, layout conversions, arith.truncf, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量重塑、布局转换、arith.truncf、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 2218-2218
```mlir
2218| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 2219-2224
```mlir
2219| 
2220| #blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [0, 1]}>
2221| #blocked3 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [16, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
2222| #blocked4 = #ttg.blocked<{sizePerThread = [2, 2], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
2223| #blocked1 = #ttg.slice<{dim = 0, parent = #blocked}>
2224| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 2225-2225
```mlir
2225| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 2226-2228
```mlir
2226|   // CHECK-LABEL: @permuting_reshape_backward_remat
2227|   // CHECK-NOT: ttg.convert_layout
2228|   // CHECK: tt.return
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @permuting_reshape_backward_remat anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @permuting_reshape_backward_remat 这样的标签用于锚定匹配范围。

### Lines 2229-2238
```mlir
2229|   tt.func public @permuting_reshape_backward_remat(%arg0: !tt.ptr<i32> {tt.divisibility = 16 : i32}) -> tensor<8x2xi32, #blocked3> {
2230|     %0 = tt.make_range {end = 16 : i32, start = 0 : i32} : tensor<16xi32, #blocked1>
2231|     %1 = tt.splat %arg0 : !tt.ptr<i32> -> tensor<16x!tt.ptr<i32>, #blocked1>
2232|     %2 = tt.addptr %1, %0 : tensor<16x!tt.ptr<i32>, #blocked1>, tensor<16xi32, #blocked1>
2233|     %3 = tt.load %2 : tensor<16x!tt.ptr<i32>, #blocked1>
2234|     %4 = tt.reshape %3 allow_reorder : tensor<16xi32, #blocked1> -> tensor<8x2xi32, #blocked4>
2235|     %5 = ttg.convert_layout %4 : tensor<8x2xi32, #blocked4> -> tensor<8x2xi32, #blocked3>
2236|     tt.return %5 : tensor<8x2xi32, #blocked3>
2237|   }
2238| 
```
**EN:** This function-oriented block defines or enters `permuting_reshape_backward_remat`. Within it, the test exercises tt.func, lane/block index ranges, broadcasted scalars or pointers, pointer arithmetic, masked or vectorized loads, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `permuting_reshape_backward_remat` 为核心。测试在其中演示 tt.func、lane/block 索引范围、广播后的标量或指针、指针算术、带掩码或向量化的加载，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 2239-2241
```mlir
2239|   // CHECK-LABEL: @permuting_reshape_no_backward_remat_efficient_layout
2240|   // CHECK: ttg.convert_layout
2241|   // CHECK: tt.return
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @permuting_reshape_no_backward_remat_efficient_layout anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @permuting_reshape_no_backward_remat_efficient_layout 这样的标签用于锚定匹配范围。

### Lines 2242-2252
```mlir
2242|   tt.func public @permuting_reshape_no_backward_remat_efficient_layout(%arg0: !tt.ptr<i32> {tt.divisibility = 16 : i32}) -> tensor<8x2xi32, #blocked3> {
2243|     %0 = tt.make_range {end = 16 : i32, start = 0 : i32} : tensor<16xi32, #blocked1>
2244|     %1 = tt.splat %arg0 : !tt.ptr<i32> -> tensor<16x!tt.ptr<i32>, #blocked1>
2245|     %2 = tt.addptr %1, %0 : tensor<16x!tt.ptr<i32>, #blocked1>, tensor<16xi32, #blocked1>
2246|     %3 = tt.load %2 : tensor<16x!tt.ptr<i32>, #blocked1>
2247|     %4 = tt.reshape %3 allow_reorder efficient_layout : tensor<16xi32, #blocked1> -> tensor<8x2xi32, #blocked4>
2248|     %5 = ttg.convert_layout %4 : tensor<8x2xi32, #blocked4> -> tensor<8x2xi32, #blocked3>
2249|     tt.return %5 : tensor<8x2xi32, #blocked3>
2250|   }
2251| }
2252| 
```
**EN:** This function-oriented block defines or enters `permuting_reshape_no_backward_remat_efficient_layout`. Within it, the test exercises tt.func, lane/block index ranges, broadcasted scalars or pointers, pointer arithmetic, masked or vectorized loads, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `permuting_reshape_no_backward_remat_efficient_layout` 为核心。测试在其中演示 tt.func、lane/block 索引范围、广播后的标量或指针、指针算术、带掩码或向量化的加载，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 2253-2253
```mlir
2253| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 2254-2257
```mlir
2254| 
2255| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
2256| #slice1dim1 = #ttg.slice<{dim = 1, parent = #blocked1}>
2257| #blocked2 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 2258-2259
```mlir
2258| module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
2259| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 2260-2260
```mlir
2260| // CHECK-LABEL: scan_propagation
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: scan_propagation anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: scan_propagation 这样的标签用于锚定匹配范围。

### Lines 2261-2269
```mlir
2261| tt.func @scan_propagation(%arg: tensor<1024xi32, #slice1dim1>) -> tensor<1024xi32, #slice1dim1> {
2262|   %1 = ttg.convert_layout %arg : tensor<1024xi32, #slice1dim1> -> tensor<1024xi32, #blocked2>
2263|   %2 = "tt.scan" (%1) ({
2264|   ^bb0(%arg3: i32, %arg4: i32):
2265|       %add = arith.addi %arg3, %arg4 : i32
2266|       tt.scan.return %add : i32
2267|   }) {axis = 0 : i32, reverse = false} : (tensor<1024xi32, #blocked2>) -> tensor<1024xi32, #blocked2>
2268|   %3 = ttg.convert_layout %2 : tensor<1024xi32, #blocked2> -> tensor<1024xi32, #slice1dim1>
2269|   // don't allow non blocked layout to be propagated to scan
```
**EN:** This function-oriented block defines or enters `scan_propagation`. Within it, the test exercises tt.func, layout conversions, integer additions, tt.scan, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `scan_propagation` 为核心。测试在其中演示 tt.func、布局转换、整数加法、tt.scan，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 2270-2277
```mlir
2270|   // CHECK: ttg.convert_layout
2271|   // CHECK: tt.scan
2272|   // CHECK: ttg.convert_layout
2273|   // CHECK: tt.return
2274|   tt.return %3: tensor<1024xi32, #slice1dim1>
2275| }
2276| }
2277| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 2278-2278
```mlir
2278| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 2279-2281
```mlir
2279| 
2280| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
2281| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 2282-2282
```mlir
2282| module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 2283-2283
```mlir
2283| // CHECK-LABEL: fw_propagate_for_op
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: fw_propagate_for_op anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: fw_propagate_for_op 这样的标签用于锚定匹配范围。

### Lines 2284-2288
```mlir
2284|   tt.func public @fw_propagate_for_op(%arg0: tensor<1024x4xi32, #blocked>, %arg1: tensor<1024x4x!tt.ptr<i32>, #blocked1>) {
2285|     %c0_i32 = arith.constant 0 : i32
2286|     %c2_i32 = arith.constant 2 : i32
2287|     %c1_i32 = arith.constant 1 : i32
2288| 
```
**EN:** This function-oriented block defines or enters `fw_propagate_for_op`. Within it, the test exercises constants, tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fw_propagate_for_op` 为核心。测试在其中演示 常量、tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 2289-2305
```mlir
2289|   // CHECK-NOT: ttg.convert_layout
2290|   // CHECK: arith.muli
2291|   // CHECK: scf.for
2292|   // CHECK:   scf.yield
2293|   // CHECK: ttg.convert_layout
2294|   // CHECK: tt.store
2295|     %0 = ttg.convert_layout %arg0 : tensor<1024x4xi32, #blocked> -> tensor<1024x4xi32, #blocked1>
2296|     %1 = arith.muli %0, %0 : tensor<1024x4xi32, #blocked1>
2297|     %2 = scf.for %arg2 = %c0_i32 to %c2_i32 step %c1_i32 iter_args(%arg3 = %1) -> (tensor<1024x4xi32, #blocked1>)  : i32 {
2298|       %3 = arith.addi %arg3, %arg3 : tensor<1024x4xi32, #blocked1>
2299|       scf.yield %3 : tensor<1024x4xi32, #blocked1>
2300|     }
2301|     tt.store %arg1, %2 : tensor<1024x4x!tt.ptr<i32>, #blocked1>
2302|     tt.return
2303|   }
2304| }
2305| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining layout conversions, integer multiplications, structured loops, integer additions, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 布局转换、整数乘法、结构化循环、整数加法、循环/分支产出值。

### Lines 2306-2306
```mlir
2306| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 2307-2310
```mlir
2307| 
2308| #blocked = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
2309| #blocked1 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
2310| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 2311-2311
```mlir
2311| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 2312-2312
```mlir
2312| // CHECK-LABEL: @rematerialize_through_if
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @rematerialize_through_if anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @rematerialize_through_if 这样的标签用于锚定匹配范围。

### Lines 2313-2313
```mlir
2313|   tt.func public @rematerialize_through_if(%arg0: i1, %arg1: f32) -> tensor<32xf32, #blocked> {
```
**EN:** This function-oriented block defines or enters `rematerialize_through_if`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `rematerialize_through_if` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 2314-2331
```mlir
2314|     // CHECK: arith.constant {{.*}} : tensor<32xf32, #blocked>
2315|     // CHECK: arith.constant {{.*}} : tensor<32xf32, #blocked>
2316|     // CHECK: scf.if %arg0 -> (tensor<32xf32, #blocked>) {
2317|     // CHECK-NOT: ttg.convert_layout
2318|     %cst = arith.constant dense<1.000000e+00> : tensor<32xf32, #blocked1>
2319|     %cst_0 = arith.constant dense<2.000000e+00> : tensor<32xf32, #blocked1>
2320|     %0 = tt.splat %arg1 : f32 -> tensor<32xf32, #blocked1>
2321|     %3 = scf.if %arg0 -> (tensor<32xf32, #blocked1>) {
2322|       %1 = arith.addf %cst, %0 : tensor<32xf32, #blocked1>
2323|       scf.yield %1 : tensor<32xf32, #blocked1>
2324|     } else {
2325|       %2 = arith.addf %cst_0, %0 : tensor<32xf32, #blocked1>
2326|       scf.yield %2 : tensor<32xf32, #blocked1>
2327|     }
2328|     %4 = ttg.convert_layout %3 : tensor<32xf32, #blocked1> -> tensor<32xf32, #blocked>
2329|     tt.return %4 : tensor<32xf32, #blocked>
2330|   }
2331| }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining constants, floating-point additions, loop/if yielded values, broadcasted scalars or pointers, structured conditionals.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 常量、浮点加法、循环/分支产出值、广播后的标量或指针、结构化条件分支。

### Lines 2333-2334
```mlir
2333| 
2334| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 2334-2337
```mlir
2334| 
2335| #blocked = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
2336| #blocked1 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
2337| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 2338-2338
```mlir
2338| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 2339-2339
```mlir
2339| // CHECK-LABEL: @rematerialize_if_inside_loop
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @rematerialize_if_inside_loop anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @rematerialize_if_inside_loop 这样的标签用于锚定匹配范围。

### Lines 2340-2340
```mlir
2340|   tt.func public @rematerialize_if_inside_loop() -> (tensor<32xf32, #blocked>, tensor<32xf32, #blocked>) {
```
**EN:** This function-oriented block defines or enters `rematerialize_if_inside_loop`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `rematerialize_if_inside_loop` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 2341-2358
```mlir
2341|     // CHECK: arith.constant {{.*}} : tensor<32xf32, #blocked>
2342|     // CHECK: arith.constant {{.*}} : tensor<32xf32, #blocked>
2343|     // CHECK-NOT: ttg.convert_layout
2344|     // CHECK: %[[for:[0-9]*]]:2 = scf.for {{.*}} -> (tensor<32xf32, #blocked>, tensor<32xf32, #blocked>)
2345| 
2346|     // CHECK-NOT: ttg.convert_layout
2347|     // CHECK: scf.if %{{.*}} -> (tensor<32xf32, #blocked>, tensor<32xf32, #blocked>)
2348|     // CHECK-NOT: ttg.convert_layout
2349|     // CHECK: scf.yield {{.*}} : tensor<32xf32, #blocked>, tensor<32xf32, #blocked>
2350|     // CHECK: scf.yield {{.*}} : tensor<32xf32, #blocked>, tensor<32xf32, #blocked>
2351|     // CHECK-NOT: ttg.convert_layout
2352|     // CHECK: tt.return %[[for]]#1, %[[for]]#0
2353|     %cst = arith.constant dense<1.000000e+00> : tensor<32xf32, #blocked1>
2354|     %cst_0 = arith.constant dense<2.000000e+00> : tensor<32xf32, #blocked>
2355|     %c0_i32 = arith.constant 0 : i32
2356|     %c32_i32 = arith.constant 32 : i32
2357|     %c4096_i32 = arith.constant 4096 : i32
2358|     %1:2 = scf.for %arg0 = %c0_i32 to %c4096_i32 step %c32_i32 iter_args(%arg1 = %cst, %arg3 = %cst_0) -> (tensor<32xf32, #blocked1>, tensor<32xf32, #blocked>) : i32 {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining constants, structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 常量、结构化循环。

### Lines 2359-2374
```mlir
2359|       %2 = arith.cmpi eq, %arg0, %c0_i32 : i32
2360|       %3:2 = scf.if %2 -> (tensor<32xf32, #blocked1>, tensor<32xf32, #blocked>) {
2361|         scf.yield %cst, %cst_0 : tensor<32xf32, #blocked1>, tensor<32xf32, #blocked>
2362|       } else {
2363|         %4 = arith.addf %arg1, %cst : tensor<32xf32, #blocked1>
2364|         %5 = ttg.convert_layout %4 : tensor<32xf32, #blocked1> -> tensor<32xf32, #blocked>
2365|         %6 = arith.mulf %arg3, %5 : tensor<32xf32, #blocked>
2366|         scf.yield %4, %6 : tensor<32xf32, #blocked1>, tensor<32xf32, #blocked>
2367|       }
2368|       scf.yield %3#0, %3#1 : tensor<32xf32, #blocked1>, tensor<32xf32, #blocked>
2369|     }
2370|     %7 = ttg.convert_layout %1#0 : tensor<32xf32, #blocked1> -> tensor<32xf32, #blocked>
2371|     tt.return %7, %1#1 : tensor<32xf32, #blocked>, tensor<32xf32, #blocked>
2372|   }
2373| }
2374| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, layout conversions, integer comparisons, structured conditionals, floating-point additions.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、布局转换、整数比较、结构化条件分支、浮点加法。

### Lines 2375-2375
```mlir
2375| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 2376-2378
```mlir
2376| 
2377| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [8, 1], order = [1, 0]}>
2378| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 8], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 2379-2379
```mlir
2379| module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 2380-2380
```mlir
2380|   // CHECK-LABEL: rematerialize_loop_arg
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: rematerialize_loop_arg anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: rematerialize_loop_arg 这样的标签用于锚定匹配范围。

### Lines 2381-2381
```mlir
2381|   tt.func public @rematerialize_loop_arg(%arg0: !tt.ptr<f16>) {
```
**EN:** This function-oriented block defines or enters `rematerialize_loop_arg`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `rematerialize_loop_arg` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 2382-2389
```mlir
2382|     // CHECK-NOT: ttg.convert_layout
2383|     %c1_i32 = arith.constant 1 : i32
2384|     %c0_i32 = arith.constant 0 : i32
2385|     %c128_i32 = arith.constant 128 : i32
2386|     %cst_0 = arith.constant dense<0.000000e+00> : tensor<128x64xf16, #blocked1>
2387|     %cst_1 = arith.constant dense<64> : tensor<128x64xi32, #blocked>
2388|     %cst_2 = arith.constant dense<128> : tensor<128x64xi32, #blocked>
2389|     %0 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<128x64x!tt.ptr<f16>, #blocked>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants, broadcasted scalars or pointers. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量、广播后的标量或指针。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 2390-2406
```mlir
2390|     // CHECK: scf.for %{{.*}} iter_args(%{{.*}} = %0) -> (tensor<128x64x!tt.ptr<f16>, #blocked>)
2391|     // CHECK-NOT: ttg.convert_layout
2392|     // CHECK: scf.yield %{{.*}} : tensor<128x64x!tt.ptr<f16>, #blocked>
2393|     %1 = scf.for %arg1 = %c0_i32 to %c128_i32 step %c1_i32 iter_args(%arg2 = %0) -> (tensor<128x64x!tt.ptr<f16>, #blocked>)  : i32 {
2394|       %2 = tt.addptr %arg2, %cst_1 : tensor<128x64x!tt.ptr<f16>, #blocked>, tensor<128x64xi32, #blocked>
2395|       %3 = ttg.convert_layout %2 : tensor<128x64x!tt.ptr<f16>, #blocked> -> tensor<128x64x!tt.ptr<f16>, #blocked1>
2396|       tt.store %3, %cst_0 : tensor<128x64x!tt.ptr<f16>, #blocked1>
2397|       %4 = tt.addptr %arg2, %cst_2 : tensor<128x64x!tt.ptr<f16>, #blocked>, tensor<128x64xi32, #blocked>
2398|       %5 = ttg.convert_layout %4 : tensor<128x64x!tt.ptr<f16>, #blocked> -> tensor<128x64x!tt.ptr<f16>, #blocked1>
2399|       tt.store %5, %cst_0 : tensor<128x64x!tt.ptr<f16>, #blocked1>
2400|       scf.yield %2 : tensor<128x64x!tt.ptr<f16>, #blocked>
2401|     }
2402|     tt.return
2403|   }
2404| }
2405| 
2406| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining pointer arithmetic, layout conversions, masked or vectorized stores, structured loops, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 指针算术、布局转换、带掩码或向量化的存储、结构化循环、循环/分支产出值。

### Lines 2407-2407
```mlir
2407| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 2408-2411
```mlir
2408| 
2409| #blocked = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
2410| #blocked1 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
2411| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 2412-2412
```mlir
2412| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 2413-2416
```mlir
2413| // CHECK-LABEL: assertop
2414| // CHECK: %[[L:.+]] = tt.load %{{.*}} : tensor<1024x!tt.ptr<i1>, #blocked>
2415| // CHECK: tt.assert %[[L]]
2416| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: assertop anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: assertop 这样的标签用于锚定匹配范围。

### Lines 2417-2424
```mlir
2417| tt.func @assertop(%ptr: tensor<1024x!tt.ptr<i1>, #blocked>) {
2418|   %0 = tt.load %ptr : tensor<1024x!tt.ptr<i1>, #blocked>
2419|   %1 = ttg.convert_layout %0 : tensor<1024xi1, #blocked> -> tensor<1024xi1, #blocked1>
2420|   tt.assert %1, "cond must be true " : tensor<1024xi1, #blocked1>
2421|   tt.return
2422| }
2423| }
2424| 
```
**EN:** This function-oriented block defines or enters `assertop`. Within it, the test exercises tt.func, masked or vectorized loads, layout conversions, tt.assert, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `assertop` 为核心。测试在其中演示 tt.func、带掩码或向量化的加载、布局转换、tt.assert、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 2425-2425
```mlir
2425| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 2426-2430
```mlir
2426| #blocked = #ttg.blocked<{sizePerThread = [1,2], threadsPerWarp = [32,1], warpsPerCTA = [1,1], order = [1,0]}>
2427| #blocked1 = #ttg.blocked<{sizePerThread = [1,1], threadsPerWarp = [16,2], warpsPerCTA = [1,1], order = [1,0]}>
2428| #blocked2 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
2429| #blocked3 = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
2430| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 2431-2431
```mlir
2431| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 2432-2432
```mlir
2432|   // CHECK-LABEL: @warp_group_dot_wait_propagate
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @warp_group_dot_wait_propagate anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @warp_group_dot_wait_propagate 这样的标签用于锚定匹配范围。

### Lines 2433-2433
```mlir
2433|   tt.func public @warp_group_dot_wait_propagate(%arg0: tensor<16x2xf32, #blocked>) -> tensor<16x2xf32, #blocked> {
```
**EN:** This function-oriented block defines or enters `warp_group_dot_wait_propagate`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `warp_group_dot_wait_propagate` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 2434-2441
```mlir
2434|     // CHECK-NOT: ttg.convert_layout
2435|     %a = ttg.convert_layout %arg0 : tensor<16x2xf32, #blocked> -> tensor<16x2xf32, #blocked1>
2436|     %b = ttng.warp_group_dot_wait %a {pendings = 0 : i32} : tensor<16x2xf32, #blocked1>
2437|     %c = ttg.convert_layout %b : tensor<16x2xf32, #blocked1> -> tensor<16x2xf32, #blocked>
2438|     tt.return %c : tensor<16x2xf32, #blocked>
2439|   }
2440| }
2441| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, ttng.warp_group_dot_wait, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、ttng.warp_group_dot_wait、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 2442-2442
```mlir
2442| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 2443-2446
```mlir
2443| #blocked = #ttg.blocked<{sizePerThread = [1,2], threadsPerWarp = [32,1], warpsPerCTA = [1,1], order = [1,0]}>
2444| #blocked1 = #ttg.blocked<{sizePerThread = [2,4], threadsPerWarp = [16,2], warpsPerCTA = [1,1], order = [1,0]}>
2445| #blocked2 = #ttg.blocked<{sizePerThread = [4,2], threadsPerWarp = [2,16], warpsPerCTA = [1,1], order = [0,1]}>
2446| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 2447-2447
```mlir
2447| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 2448-2448
```mlir
2448|   // CHECK-LABEL: @trans_propagate
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @trans_propagate anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @trans_propagate 这样的标签用于锚定匹配范围。

### Lines 2449-2449
```mlir
2449|   tt.func public @trans_propagate(%arg0: tensor<16x2xf32, #blocked>) -> tensor<2x16xf32, #blocked2> {
```
**EN:** This function-oriented block defines or enters `trans_propagate`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `trans_propagate` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 2450-2458
```mlir
2450|     // CHECK: tt.trans
2451|     // CHECK: ttg.convert_layout
2452|     %a = ttg.convert_layout %arg0 : tensor<16x2xf32, #blocked> -> tensor<16x2xf32, #blocked1>
2453|     %b = tt.trans %a {order=array<i32: 1,0>} : tensor<16x2xf32, #blocked1> -> tensor<2x16xf32, #blocked2>
2454|     tt.return %b : tensor<2x16xf32, #blocked2>
2455|   }
2456| }
2457| 
2458| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, transpose-like layout changes, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、转置类布局变换、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 2459-2459
```mlir
2459| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 2460-2463
```mlir
2460| #blocked = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [2, 16], warpsPerCTA = [8, 1], order = [1, 0]}>
2461| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [8, 1], instrShape = [16, 128, 32]}>
2462| #mma1 = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [8, 1], instrShape = [16, 256, 32]}>
2463| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 2464-2467
```mlir
2464| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, "ttg.threads-per-warp" = 32 : i32} {
2465|   // Verify that we don't hoist the convert on top of the broadcast. In general we should hoist the convert to reduce its cost
2466|   // but because this would combine the 1st and 2nd convert and since the 1st convert is known to be a no-op this would
2467|   // generate more expensive code.
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 2468-2468
```mlir
2468|   // CHECK-LABEL: @hoist_with_free_convert
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @hoist_with_free_convert anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @hoist_with_free_convert 这样的标签用于锚定匹配范围。

### Lines 2469-2469
```mlir
2469|   tt.func public @hoist_with_free_convert(%arg0: tensor<128x256xf32, #mma1>, %arg1: tensor<128x1xf32, #mma>) -> tensor<128x256xf32, #blocked> {
```
**EN:** This function-oriented block defines or enters `hoist_with_free_convert`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `hoist_with_free_convert` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 2470-2481
```mlir
2470|     // CHECK: ttg.convert_layout
2471|     // CHECK: tt.broadcast
2472|     // CHECK: ttg.convert_layout
2473|     // CHECK: tt.return
2474|     %0 = ttg.convert_layout %arg0 : tensor<128x256xf32, #mma1> -> tensor<128x256xf32, #mma>
2475|     %1 = tt.broadcast %arg1 : tensor<128x1xf32, #mma> -> tensor<128x256xf32, #mma>
2476|     %2 = arith.addf %0, %1 : tensor<128x256xf32, #mma>
2477|     %3 = ttg.convert_layout %2 : tensor<128x256xf32, #mma> -> tensor<128x256xf32, #blocked>
2478|     tt.return %3 : tensor<128x256xf32, #blocked>
2479|   }
2480| }
2481| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, tensor broadcasting, floating-point additions, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、张量广播、浮点加法、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 2482-2482
```mlir
2482| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 2483-2486
```mlir
2483| 
2484| #blocked = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
2485| #blocked1 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
2486| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 2487-2487
```mlir
2487| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 2488-2488
```mlir
2488|   // CHECK-LABEL: @rematerialize_loop_arg
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @rematerialize_loop_arg anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @rematerialize_loop_arg 这样的标签用于锚定匹配范围。

### Lines 2489-2494
```mlir
2489|   tt.func public @rematerialize_loop_arg() -> (tensor<32xf32, #blocked>, tensor<32xf32, #blocked>, tensor<32xf32, #blocked1>) {
2490|     %cst = arith.constant dense<1.000000e+00> : tensor<32xf32, #blocked1>
2491|     %cst_0 = arith.constant dense<2.000000e+00> : tensor<32xf32, #blocked>
2492|     %c0_i32 = arith.constant 0 : i32
2493|     %c32_i32 = arith.constant 32 : i32
2494|     %c4096_i32 = arith.constant 4096 : i32
```
**EN:** This function-oriented block defines or enters `rematerialize_loop_arg`. Within it, the test exercises constants, tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `rematerialize_loop_arg` 为核心。测试在其中演示 常量、tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 2495-2512
```mlir
2495|     // CHECK: %[[F:.+]]:3 = scf.for
2496|     // CHECK:   %[[R:.+]] = arith.addf
2497|     // CHECK:   arith.addf
2498|     // CHECK:   scf.yield %{{.+}}, %{{.+}}, %[[R]]
2499|     // CHECK: }
2500|     // CHECK: tt.return %[[F]]#2, %[[F]]#1, %[[F]]#0
2501|     %1:3 = scf.for %arg0 = %c0_i32 to %c4096_i32 step %c32_i32 iter_args(%arg1 = %cst, %arg3 = %cst_0, %arg4 = %cst) -> (tensor<32xf32, #blocked1>, tensor<32xf32, #blocked>, tensor<32xf32, #blocked1>) : i32 {
2502|       %4 = arith.addf %arg1, %cst : tensor<32xf32, #blocked1>
2503|       %5 = ttg.convert_layout %4 : tensor<32xf32, #blocked1> -> tensor<32xf32, #blocked>
2504|       %6 = arith.mulf %arg3, %5 : tensor<32xf32, #blocked>
2505|       scf.yield %4, %6, %4 : tensor<32xf32, #blocked1>, tensor<32xf32, #blocked>, tensor<32xf32, #blocked1>
2506|     }
2507|     %7 = ttg.convert_layout %1#0 : tensor<32xf32, #blocked1> -> tensor<32xf32, #blocked>
2508|     tt.return %7, %1#1, %1#2 : tensor<32xf32, #blocked>, tensor<32xf32, #blocked>, tensor<32xf32, #blocked1>
2509| 
2510|   }
2511| }
2512| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining layout conversions, structured loops, floating-point additions, arith.mulf, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 布局转换、结构化循环、浮点加法、arith.mulf、循环/分支产出值。

### Lines 2513-2513
```mlir
2513| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 2514-2518
```mlir
2514| 
2515| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
2516| #blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
2517| #blocked2 = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [8, 4], warpsPerCTA = [1, 4], order = [0, 1]}>
2518| #mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 8]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 2519-2523
```mlir
2519| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
2520|   // Regression test:
2521|   // Rematerialization of multiple loop-carried variables, where one is
2522|   // rematerialized to the same layout by multiple users.
2523|   // Previously this didn't interact correctly with the de-duplication mechanism.
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 2524-2524
```mlir
2524|   // CHECK-LABEL: @multi_rematerialize_loop_arg
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @multi_rematerialize_loop_arg anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @multi_rematerialize_loop_arg 这样的标签用于锚定匹配范围。

### Lines 2525-2537
```mlir
2525|   tt.func public  @multi_rematerialize_loop_arg(%arg0: !tt.ptr<f16>, %arg1: !tt.ptr<i8>) -> (tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>, tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>) {
2526|     %c0_i32 = arith.constant 0 : i32
2527|     %c64_i32 = arith.constant 64 : i32
2528|     %c2048_i32 = arith.constant 2048 : i32
2529|     %cst = arith.constant dense<0.000000e+00> : tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
2530|     %cst_0 = arith.constant dense<0xFF800000> : tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
2531|     %cst_1 = arith.constant dense<0xFF800000> : tensor<128x64xf32, #mma>
2532|     %cst_2 = arith.constant dense<0.000000e+00> : tensor<128x64xf32, #mma>
2533|     %0 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<128x64x!tt.ptr<f16>, #blocked1>
2534|     %1 = tt.load %0 : tensor<128x64x!tt.ptr<f16>, #blocked1>
2535|     %2 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<64x64x!tt.ptr<f16>, #blocked2>
2536|     %3 = tt.splat %arg1 : !tt.ptr<i8> -> tensor<128x64x!tt.ptr<i8>, #blocked>
2537|     %4 = tt.splat %arg1 : !tt.ptr<i8> -> tensor<128x64x!tt.ptr<i8>, #blocked>
```
**EN:** This function-oriented block defines or enters `multi_rematerialize_loop_arg`. Within it, the test exercises constants, broadcasted scalars or pointers, tt.func, masked or vectorized loads, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `multi_rematerialize_loop_arg` 为核心。测试在其中演示 常量、广播后的标量或指针、tt.func、带掩码或向量化的加载，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 2538-2555
```mlir
2538|     // CHECK: %[[F:.+]]:3 = scf.for {{.*}} -> (tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>, tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>, tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>)
2539|     // CHECK-COUNT-4: convert_layout
2540|     // CHECK-NOT: convert_layout
2541|     // CHECK:   scf.yield {{.*}} : tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>, tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>, tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
2542|     // CHECK: }
2543|     // CHECK: tt.return %[[F]]#0, %[[F]]#1 : tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>, tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
2544|      %5:3 = scf.for %arg2 = %c0_i32 to %c2048_i32 step %c64_i32 iter_args(%arg3 = %cst_2, %arg4 = %cst, %arg5 = %cst_0) -> (tensor<128x64xf32, #mma>, tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>, tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>)  : i32 {
2545|       %6 = tt.load %2 : tensor<64x64x!tt.ptr<f16>, #blocked2>
2546|       %7 = ttg.convert_layout %1 : tensor<128x64xf16, #blocked1> -> tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
2547|       %8 = ttg.convert_layout %6 : tensor<64x64xf16, #blocked2> -> tensor<64x64xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>
2548|       %9 = tt.dot %7, %8, %cst_2, inputPrecision = tf32 : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<64x64xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<128x64xf32, #mma>
2549|       %10 = tt.load %3 : tensor<128x64x!tt.ptr<i8>, #blocked>
2550|       %11 = tt.load %4 : tensor<128x64x!tt.ptr<i8>, #blocked>
2551|       %12 = arith.cmpi eq, %10, %11 : tensor<128x64xi8, #blocked>
2552|       %13 = ttg.convert_layout %12 : tensor<128x64xi1, #blocked> -> tensor<128x64xi1, #mma>
2553|       %14 = arith.select %13, %9, %cst_1 : tensor<128x64xi1, #mma>, tensor<128x64xf32, #mma>
2554|       %15 = ttg.convert_layout %14 : tensor<128x64xf32, #mma> -> tensor<128x64xf32, #blocked>
2555|       %16 = "tt.reduce"(%15) <{axis = 1 : i32}> ({
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining layout conversions, masked or vectorized loads, structured loops, dot-product or MMA-style math, integer comparisons.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 布局转换、带掩码或向量化的加载、结构化循环、点积或 MMA 风格计算、整数比较。

### Lines 2556-2573
```mlir
2556|       ^bb0(%arg6: f32, %arg7: f32):
2557|         %34 = arith.maxnumf %arg6, %arg7 : f32
2558|         tt.reduce.return %34 : f32
2559|       }) : (tensor<128x64xf32, #blocked>) -> tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
2560|       %17 = arith.maxnumf %arg5, %16 : tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
2561|       %18 = arith.cmpf oeq, %17, %cst_0 : tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
2562|       %19 = ttg.convert_layout %18 : tensor<128xi1, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<128xi1, #ttg.slice<{dim = 1, parent = #mma}>>
2563|       %20 = arith.select %18, %cst, %17 : tensor<128xi1, #ttg.slice<{dim = 1, parent = #blocked}>>, tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
2564|       %21 = tt.expand_dims %19 {axis = 1 : i32} : tensor<128xi1, #ttg.slice<{dim = 1, parent = #mma}>> -> tensor<128x1xi1, #mma>
2565|       %22 = tt.broadcast %21 : tensor<128x1xi1, #mma> -> tensor<128x64xi1, #mma>
2566|       %23 = arith.select %22, %cst_2, %14 : tensor<128x64xi1, #mma>, tensor<128x64xf32, #mma>
2567|       %24 = ttg.convert_layout %23 : tensor<128x64xf32, #mma> -> tensor<128x64xf32, #blocked>
2568|       %25 = arith.mulf %arg4, %cst : tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
2569|       %26 = ttg.convert_layout %25 : tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<128xf32, #ttg.slice<{dim = 1, parent = #mma}>>
2570|       %27 = tt.expand_dims %26 {axis = 1 : i32} : tensor<128xf32, #ttg.slice<{dim = 1, parent = #mma}>> -> tensor<128x1xf32, #mma>
2571|       %28 = tt.broadcast %27 : tensor<128x1xf32, #mma> -> tensor<128x64xf32, #mma>
2572|       %29 = arith.mulf %arg3, %28 : tensor<128x64xf32, #mma>
2573|       %30 = ttg.convert_layout %23 : tensor<128x64xf32, #mma> -> tensor<128x64xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, arith.maxnumf, arith.select, shape expansion, tensor broadcasting. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、arith.maxnumf、arith.select、形状扩展、张量广播。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 2574-2586
```mlir
2574|       %31 = arith.mulf %arg4, %20 : tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
2575|       %32 = "tt.reduce"(%24) <{axis = 1 : i32}> ({
2576|       ^bb0(%arg6: f32, %arg7: f32):
2577|         %34 = arith.addf %arg6, %arg7 : f32
2578|         tt.reduce.return %34 : f32
2579|       }) : (tensor<128x64xf32, #blocked>) -> tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
2580|       %33 = arith.addf %31, %32 : tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
2581|       scf.yield %29, %33, %17 : tensor<128x64xf32, #mma>, tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>, tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
2582|     }
2583|     tt.return %5#1, %5#2 : tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>, tensor<128xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
2584|   }
2585| }
2586| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining floating-point additions, arith.mulf, reductions, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 浮点加法、arith.mulf、归约、循环/分支产出值、tt.return。

### Lines 2587-2587
```mlir
2587| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 2588-2591
```mlir
2588| 
2589| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
2590| #blocked2 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [2, 2], order = [1, 0]}>
2591| #blocked7 = #ttg.blocked<{sizePerThread = [4, 4], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 2592-2595
```mlir
2592| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
2593|   // Regression test:
2594|   // The while loop use the result of the for loop as an argument.
2595|   // When propagating the layout, we should only "forward" propagate the layout to the argument and the result of the while loop
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 2596-2596
```mlir
2596|   // CHECK-LABEL: @while_use_for
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @while_use_for anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @while_use_for 这样的标签用于锚定匹配范围。

### Lines 2597-2614
```mlir
2597|   tt.func public @while_use_for(%arg0: !tt.ptr<f16>, %arg3: !tt.ptr<f32>, %arg6: i32) {
2598|     %c0_i32 = arith.constant 0 : i32
2599|     %c1_i32 = arith.constant 1 : i32
2600|     %c8_i32 = arith.constant 8 : i32
2601|     %c0_i1 = arith.constant 1 : i1
2602|     %cst_0 = arith.constant dense<0.000000e+00> : tensor<256x128xf32, #blocked1>
2603|     %1000 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<256x64x!tt.ptr<f16>, #blocked2>
2604|     %1001 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<64x128x!tt.ptr<f16>, #blocked1>
2605|     %1002 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<256x128x!tt.ptr<f16>, #blocked1>
2606|     %1003 = tt.splat %arg3 : !tt.ptr<f32> -> tensor<256x128x!tt.ptr<f32>, #blocked1>
2607|     %74 = tt.load %1000 : tensor<256x64x!tt.ptr<f16>, #blocked2>
2608|     %67:2 = scf.for %arg11 = %c0_i32 to %c8_i32 step %c1_i32 iter_args(%arg12 = %cst_0, %arg14 = %1001) -> (tensor<256x128xf32, #blocked1>, tensor<64x128x!tt.ptr<f16>, #blocked1>)  : i32 {
2609|       %76 = tt.load %arg14 : tensor<64x128x!tt.ptr<f16>, #blocked1>
2610|       %78 = ttg.convert_layout %74 : tensor<256x64xf16, #blocked2> -> tensor<256x64xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked7}>>
2611|       %79 = ttg.convert_layout %76 : tensor<64x128xf16, #blocked1> -> tensor<64x128xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked7}>>
2612|       %80 = ttg.convert_layout %arg12 : tensor<256x128xf32, #blocked1> -> tensor<256x128xf32, #blocked7>
2613|       %81 = tt.dot %78, %79, %80, inputPrecision = tf32 : tensor<256x64xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked7}>> * tensor<64x128xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked7}>> -> tensor<256x128xf32, #blocked7>
2614|       %82 = ttg.convert_layout %81 : tensor<256x128xf32, #blocked7> -> tensor<256x128xf32, #blocked1>
```
**EN:** This function-oriented block defines or enters `while_use_for`. Within it, the test exercises constants, broadcasted scalars or pointers, layout conversions, tt.func, masked or vectorized loads, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `while_use_for` 为核心。测试在其中演示 常量、广播后的标量或指针、布局转换、tt.func、带掩码或向量化的加载，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 2615-2632
```mlir
2615|       scf.yield %82, %arg14 : tensor<256x128xf32, #blocked1>, tensor<64x128x!tt.ptr<f16>, #blocked1>
2616|     }
2617|     %68:2 = scf.while (%arg11 = %67#0, %arg12 = %c1_i32) : (tensor<256x128xf32, #blocked1>, i32) -> (tensor<256x128xf32, #blocked1>, i32) {
2618|       scf.condition(%c0_i1) %arg11, %arg12 : tensor<256x128xf32, #blocked1>, i32
2619|     } do {
2620|     ^bb0(%arg11: tensor<256x128xf32, #blocked1>, %arg12: i32):
2621|       %80 = ttg.convert_layout %1003 : tensor<256x128x!tt.ptr<f32>, #blocked1> -> tensor<256x128x!tt.ptr<f32>, #blocked1>
2622|       %81 = tt.load %80 : tensor<256x128x!tt.ptr<f32>, #blocked1>
2623|       %82 = arith.addf %arg11, %81 : tensor<256x128xf32, #blocked1>
2624|       %83 = arith.addi %arg12, %c1_i32 : i32
2625|       scf.yield %82, %83 : tensor<256x128xf32, #blocked1>, i32
2626|     }
2627|     %69 = arith.truncf %68#0 : tensor<256x128xf32, #blocked1> to tensor<256x128xf16, #blocked1>
2628|     %71 = ttg.convert_layout %69 : tensor<256x128xf16, #blocked1> -> tensor<256x128xf16, #blocked1>
2629|     tt.store %1002, %71 : tensor<256x128x!tt.ptr<f16>, #blocked1>
2630|     tt.return
2631|   }
2632| }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, layout conversions, scf.while, scf.condition, masked or vectorized loads.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、布局转换、scf.while、scf.condition、带掩码或向量化的加载。

### Lines 2634-2635
```mlir
2634| 
2635| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 2635-2637
```mlir
2635| // Minimized reproducer for https://github.com/pytorch/pytorch/issues/130101
2636| // Check that backward rematerialization bails out when the same tensor requires two different layouts
2637| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 2638-2642
```mlir
2638| // CHECK-LABEL: double_remat
2639| // CHECK: %[[res:.*]] = ttg.convert_layout
2640| // CHECK: tt.broadcast %[[res]]
2641| // CHECK-NOT: ttg.convert_layout
2642| // CHECK: tt.return
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: double_remat anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: double_remat 这样的标签用于锚定匹配范围。

### Lines 2643-2645
```mlir
2643| #blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 32], warpsPerCTA = [1, 2], order = [1, 0]}>
2644| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 2], order = [1, 0]}>
2645| #blocked2 = #ttg.blocked<{sizePerThread = [1, 1, 1], threadsPerWarp = [1, 1, 32], warpsPerCTA = [1, 1, 2], order = [2, 1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 2646-2646
```mlir
2646| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 2 : i32, ttg.target = "cuda:86", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:86`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:86`。

### Lines 2647-2662
```mlir
2647|   tt.func public @double_remat() -> tensor<1x256xi32, #blocked> {
2648|     %cst = arith.constant dense<0> : tensor<1x256xi32, #blocked1>
2649|     %0 = tt.make_range {end = 2 : i32, start = 0 : i32} : tensor<2xi32, #ttg.slice<{dim = 0, parent = #ttg.slice<{dim = 2, parent = #blocked2}>}>>
2650|     %1 = tt.expand_dims %0 {axis = 0 : i32} : tensor<2xi32, #ttg.slice<{dim = 0, parent = #ttg.slice<{dim = 2, parent = #blocked2}>}>> -> tensor<1x2xi32, #ttg.slice<{dim = 2, parent = #blocked2}>>
2651|     %2 = tt.expand_dims %1 {axis = 2 : i32} : tensor<1x2xi32, #ttg.slice<{dim = 2, parent = #blocked2}>> -> tensor<1x2x1xi32, #blocked2>
2652|     %3 = tt.broadcast %2 : tensor<1x2x1xi32, #blocked2> -> tensor<1x2x128xi32, #blocked2>
2653|     %4 = tt.reshape %3 : tensor<1x2x128xi32, #blocked2> -> tensor<1x256xi32, #blocked1>
2654|     %5 = tt.broadcast %2 : tensor<1x2x1xi32, #blocked2> -> tensor<2x2x64xi32, #blocked2>
2655|     %6 = tt.reshape %5 : tensor<2x2x64xi32, #blocked2> -> tensor<1x256xi32, #blocked1>
2656|     %7 = arith.cmpi ne, %4, %cst : tensor<1x256xi32, #blocked1>
2657|     %8 = arith.select %7, %6, %cst : tensor<1x256xi1, #blocked1>, tensor<1x256xi32, #blocked1>
2658|     %9 = ttg.convert_layout %8 : tensor<1x256xi32, #blocked1> -> tensor<1x256xi32, #blocked>
2659|     tt.return %9 : tensor<1x256xi32, #blocked>
2660|   }
2661| }
2662| 
```
**EN:** This function-oriented block defines or enters `double_remat`. Within it, the test exercises tt.func, shape expansion, tensor broadcasting, tensor reshaping, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `double_remat` 为核心。测试在其中演示 tt.func、形状扩展、张量广播、张量重塑、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 2663-2663
```mlir
2663| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 2664-2665
```mlir
2664| #blocked = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
2665| #blocked1 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 2666-2666
```mlir
2666| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 2667-2669
```mlir
2667|   // CHECK-LABEL: @if_condition_not_dead_inside_loop
2668|   // CHECK: scf.if
2669|   // CHECK-NOT: convert_layout
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @if_condition_not_dead_inside_loop anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @if_condition_not_dead_inside_loop 这样的标签用于锚定匹配范围。

### Lines 2670-2687
```mlir
2670|   tt.func public @if_condition_not_dead_inside_loop(%arg0: i32) -> (tensor<32xf32, #blocked>, tensor<32xf32, #blocked>) {
2671|     %true = arith.constant true
2672|     %cst = arith.constant dense<1.000000e+00> : tensor<32xf32, #blocked1>
2673|     %cst_0 = arith.constant dense<2.000000e+00> : tensor<32xf32, #blocked>
2674|     %c0_i32 = arith.constant 0 : i32
2675|     %c32_i32 = arith.constant 32 : i32
2676|     %c4096_i32 = arith.constant 4096 : i32
2677|     %1:3 = scf.for %arg10 = %c0_i32 to %c4096_i32 step %c32_i32 iter_args(%arg1 = %cst, %arg3 = %cst_0, %arg4 = %true) -> (tensor<32xf32, #blocked1>, tensor<32xf32, #blocked>, i1) : i32 {
2678|       %3:2 = scf.if %arg4 -> (tensor<32xf32, #blocked1>, tensor<32xf32, #blocked>) {
2679|         scf.yield %cst, %cst_0 : tensor<32xf32, #blocked1>, tensor<32xf32, #blocked>
2680|       } else {
2681|         %4 = arith.addf %arg1, %cst : tensor<32xf32, #blocked1>
2682|         %5 = ttg.convert_layout %4 : tensor<32xf32, #blocked1> -> tensor<32xf32, #blocked>
2683|         %6 = arith.mulf %arg3, %5 : tensor<32xf32, #blocked>
2684|         scf.yield %4, %6 : tensor<32xf32, #blocked1>, tensor<32xf32, #blocked>
2685|       }
2686|       %119 = arith.cmpi eq, %arg10, %arg0 : i32
2687|       scf.yield %3#0, %3#1, %119 : tensor<32xf32, #blocked1>, tensor<32xf32, #blocked>, i1
```
**EN:** This function-oriented block defines or enters `if_condition_not_dead_inside_loop`. Within it, the test exercises constants, loop/if yielded values, tt.func, structured loops, structured conditionals, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `if_condition_not_dead_inside_loop` 为核心。测试在其中演示 常量、循环/分支产出值、tt.func、结构化循环、结构化条件分支，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 2688-2693
```mlir
2688|     }
2689|     %7 = ttg.convert_layout %1#0 : tensor<32xf32, #blocked1> -> tensor<32xf32, #blocked>
2690|     tt.return %7, %1#1 : tensor<32xf32, #blocked>, tensor<32xf32, #blocked>
2691|   }
2692| }
2693| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 2694-2694
```mlir
2694| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 2695-2696
```mlir
2695| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [8, 1], instrShape = [16, 32, 16]}>
2696| #mma1 = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 2], instrShape = [16, 64, 16]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 2697-2697
```mlir
2697| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 2698-2698
```mlir
2698|   // CHECK-LABEL: @dot_wait
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @dot_wait anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @dot_wait 这样的标签用于锚定匹配范围。

### Lines 2699-2701
```mlir
2699|   tt.func public @dot_wait(%arg0: tensor<64x64xf32, #mma>, %arg1: tensor<64x128xf32, #mma1>) -> (tensor<64x64xf32, #mma>, tensor<64x128xf32, #mma1>) {
2700|     %0:2 = ttng.warp_group_dot_wait %arg0, %arg1 {pendings = 0 : i32} : tensor<64x64xf32, #mma>, tensor<64x128xf32, #mma1>
2701|     tt.return %0#0, %0#1 : tensor<64x64xf32, #mma>, tensor<64x128xf32, #mma1>
```
**EN:** This function-oriented block defines or enters `dot_wait`. Within it, the test exercises tt.func, ttng.warp_group_dot_wait, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `dot_wait` 为核心。测试在其中演示 tt.func、ttng.warp_group_dot_wait、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 2702-2706
```mlir
2702|     // CHECK: %[[W:.+]]:2 = ttng.warp_group_dot_wait
2703|     // CHECK: tt.return %[[W]]#0, %[[W]]#1 : tensor<64x64xf32, #mma>, tensor<64x128xf32, #mma1>
2704|   }
2705| }
2706| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 2707-2707
```mlir
2707| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 2708-2711
```mlir
2708| 
2709| #blocked = #ttg.blocked<{sizePerThread = [1, 64, 2], threadsPerWarp = [32, 1, 1], warpsPerCTA = [4, 1, 1], order = [0, 1, 2]}>
2710| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [2, 2], order = [1, 0]}>
2711| #blocked2 = #ttg.blocked<{sizePerThread = [1, 1, 2], threadsPerWarp = [1, 32, 1], warpsPerCTA = [2, 2, 1], order = [2, 1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 2712-2712
```mlir
2712| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 2713-2717
```mlir
2713|   // CHECK-LABEL: @split_propagation
2714|   // CHECK-SAME: (%[[ARG:.+]]: tensor<128x64x2xf32
2715|   //      CHECK: %[[S:.+]], %{{.+}} = tt.split %[[ARG]]
2716|   //      CHECK: %[[C:.+]] = ttg.convert_layout %[[S]]
2717|   //      CHECK: tt.return %[[C]]
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 2718-2724
```mlir
2718|   tt.func public @split_propagation(%arg0: tensor<128x64x2xf32, #blocked>) -> tensor<128x64xf32, #blocked1> {
2719|     %0 = ttg.convert_layout %arg0 : tensor<128x64x2xf32, #blocked> -> tensor<128x64x2xf32, #blocked2>
2720|     %outLHS, %outRHS = tt.split %0 : tensor<128x64x2xf32, #blocked2> -> tensor<128x64xf32, #blocked1>
2721|     tt.return %outLHS : tensor<128x64xf32, #blocked1>
2722|   }
2723| }
2724| 
```
**EN:** This function-oriented block defines or enters `split_propagation`. Within it, the test exercises tt.func, layout conversions, tt.split, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `split_propagation` 为核心。测试在其中演示 tt.func、布局转换、tt.split、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 2725-2725
```mlir
2725| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 2726-2727
```mlir
2726| 
2727| // Test split with a weird layout
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 2728-2730
```mlir
2728| #blocked = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [32, 1], warpsPerCTA = [1, 1], order = [1, 0]}>
2729| #blocked1 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
2730| #linear = #ttg.linear<{register = [[1, 0], [4, 0], [0, 0], [0, 0], [8, 0], [0, 1], [2, 0]], lane = [[0, 0], [0, 0], [0, 0], [0, 0], [0, 0]], warp = [], block = []}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 2731-2731
```mlir
2731| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 2732-2736
```mlir
2732|   // CHECK-LABEL: @split_propagation_linear
2733|   // CHECK-SAME: (%[[ARG:.+]]: tensor<16x2xf32
2734|   //      CHECK: %[[S:.+]], %{{.+}} = tt.split %[[ARG]]
2735|   //      CHECK: %[[C:.+]] = ttg.convert_layout %[[S]]
2736|   //      CHECK: tt.return %[[C]]
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 2737-2743
```mlir
2737|   tt.func public @split_propagation_linear(%arg0: tensor<16x2xf32, #linear>) -> tensor<16xf32, #blocked1> {
2738|     %0 = ttg.convert_layout %arg0 : tensor<16x2xf32, #linear> -> tensor<16x2xf32, #blocked>
2739|     %outLHS, %outRHS = tt.split %0 : tensor<16x2xf32, #blocked> -> tensor<16xf32, #blocked1>
2740|     tt.return %outLHS : tensor<16xf32, #blocked1>
2741|   }
2742| }
2743| 
```
**EN:** This function-oriented block defines or enters `split_propagation_linear`. Within it, the test exercises tt.func, layout conversions, tt.split, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `split_propagation_linear` 为核心。测试在其中演示 tt.func、布局转换、tt.split、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 2744-2744
```mlir
2744| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 2745-2748
```mlir
2745| 
2746| #blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
2747| #blocked1 = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
2748| #blocked2 = #ttg.blocked<{sizePerThread = [4, 4], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 2749-2749
```mlir
2749| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 2750-2752
```mlir
2750|   // CHECK-DAG: [[LINEAR:#.*]] = #ttg.linear
2751|   // CHECK-DAG: [[BLOCKED:#.*]] = #ttg.blocked<{sizePerThread = [4, 4], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
2752|   // CHECK: tt.split {{.*}} : tensor<32x2xf32, [[LINEAR]]> -> tensor<32xf32, #ttg.slice<{dim = 1, parent = [[BLOCKED]]}>>
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 2753-2760
```mlir
2753|   tt.func public @split_slice_backward_propagation() -> tensor<32xf32, #ttg.slice<{dim=1, parent=#blocked2}>> {
2754|     %cst = arith.constant dense<0.0> : tensor<32x2xf32, #blocked1>
2755|     %outLHS, %outRHS = tt.split %cst : tensor<32x2xf32, #blocked1> -> tensor<32xf32, #blocked>
2756|     %62 = ttg.convert_layout %outLHS : tensor<32xf32, #blocked> -> tensor<32xf32, #ttg.slice<{dim=1, parent=#blocked2}>>
2757|     tt.return %62 : tensor<32xf32, #ttg.slice<{dim=1, parent=#blocked2}>>
2758|   }
2759| }
2760| 
```
**EN:** This function-oriented block defines or enters `split_slice_backward_propagation`. Within it, the test exercises tt.func, constants, tt.split, layout conversions, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `split_slice_backward_propagation` 为核心。测试在其中演示 tt.func、常量、tt.split、布局转换、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 2761-2761
```mlir
2761| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 2762-2766
```mlir
2762| 
2763| #blocked = #ttg.blocked<{sizePerThread = [1, 1, 1, 1, 4], threadsPerWarp = [2, 1, 16, 1, 1], warpsPerCTA = [1, 1, 2, 2, 1], order = [4, 0, 1, 2, 3]}>
2764| #blocked2 = #ttg.blocked<{sizePerThread = [1, 1, 1, 1, 4], threadsPerWarp = [1, 1, 32, 1, 1], warpsPerCTA = [1, 1, 1, 1, 4], order = [4, 3, 2, 1, 0]}>
2765| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1, 1, 1, 4], threadsPerWarp = [2, 1, 16, 1, 1], warpsPerCTA = [1, 2, 2, 1, 1], order = [4, 0, 3, 2, 1]}>
2766| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [4, 0, 1, 2, 3]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 2767-2767
```mlir
2767| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 2768-2770
```mlir
2768|   // CHECK-LABEL: lift_convert_to_local_load
2769|   // CHECK-NOT: convert_layout
2770|   // CHECK: tt.return
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: lift_convert_to_local_load anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: lift_convert_to_local_load 这样的标签用于锚定匹配范围。

### Lines 2771-2778
```mlir
2771|   tt.func public @lift_convert_to_local_load(%arg0 : !ttg.memdesc<2x1x32x4x4xi8, #shared, #ttg.shared_memory, mutable>) -> tensor<2x4x32x1x4xi8, #blocked2> {
2772|     %1 = ttg.local_load %arg0 : !ttg.memdesc<2x1x32x4x4xi8, #shared, #ttg.shared_memory, mutable> -> tensor<2x1x32x4x4xi8, #blocked>
2773|     %2 = tt.trans %1 {order = array<i32: 0, 3, 2, 1, 4>} : tensor<2x1x32x4x4xi8, #blocked> -> tensor<2x4x32x1x4xi8, #blocked1>
2774|     %3 = ttg.convert_layout %2 : tensor<2x4x32x1x4xi8, #blocked1> -> tensor<2x4x32x1x4xi8, #blocked2>
2775|     tt.return %3 : tensor<2x4x32x1x4xi8, #blocked2>
2776|   }
2777| }
2778| 
```
**EN:** This function-oriented block defines or enters `lift_convert_to_local_load`. Within it, the test exercises tt.func, local/shared memory loads, transpose-like layout changes, layout conversions, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `lift_convert_to_local_load` 为核心。测试在其中演示 tt.func、本地/共享内存加载、转置类布局变换、布局转换、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 2779-2779
```mlir
2779| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 2780-2787
```mlir
2780| 
2781| #AL = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
2782| #BL = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
2783| #CL = #ttg.blocked<{sizePerThread = [4, 4], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
2784| #C = #ttg.nvidia_mma<{versionMajor = 2, warpsPerCTA = [4, 1], instrShape = [16, 8]}>
2785| #A_DOT = #ttg.dot_op<{opIdx = 0, parent = #C, kWidth = 2}>
2786| #B_DOT = #ttg.dot_op<{opIdx = 1, parent = #C, kWidth = 2}>
2787| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 2788-2788
```mlir
2788| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 2789-2789
```mlir
2789|   // CHECK-LABEL: matmul_add
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: matmul_add anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: matmul_add 这样的标签用于锚定匹配范围。

### Lines 2790-2805
```mlir
2790|   tt.func @matmul_add(%lb : index, %ub : index, %step : index, %A : !tt.ptr<f16>, %B : !tt.ptr<f16>, %C : !tt.ptr<f32>) {
2791|     %a_ptr_init = tt.splat %A : !tt.ptr<f16> -> tensor<128x32x!tt.ptr<f16>, #AL>
2792|     %b_ptr_init = tt.splat %B : !tt.ptr<f16> -> tensor<32x128x!tt.ptr<f16>, #BL>
2793|     %c_ptr_init = tt.splat %C : !tt.ptr<f32> -> tensor<128x128x!tt.ptr<f32>, #CL>
2794|     %c_init = arith.constant dense<0.00e+00> : tensor<128x128xf32, #CL>
2795|     %cst = arith.constant dense<0.00e+00> : tensor<128x128xf32, #C>
2796|     %a_off = arith.constant dense<4> : tensor<128x32xi32, #AL>
2797|     %b_off = arith.constant dense<4> : tensor<32x128xi32, #BL>
2798| 
2799|     %100:3 = scf.for %iv = %lb to %ub step %step iter_args(%a_ptr = %a_ptr_init, %b_ptr = %b_ptr_init, %prev_c = %c_init) -> (tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #CL>) {
2800|       %a_ = tt.load %a_ptr : tensor<128x32x!tt.ptr<f16>, #AL>
2801|       %a = ttg.convert_layout %a_ : tensor<128x32xf16, #AL> -> tensor<128x32xf16, #A_DOT>
2802|       %b_ = tt.load %b_ptr : tensor<32x128x!tt.ptr<f16>, #BL>
2803|       %b = ttg.convert_layout %b_ : tensor<32x128xf16, #BL> -> tensor<32x128xf16, #B_DOT>
2804|       %c = tt.dot %a, %b, %cst : tensor<128x32xf16, #A_DOT> * tensor<32x128xf16, #B_DOT> -> tensor<128x128xf32, #C>
2805|       %t = ttg.convert_layout %c : tensor<128x128xf32, #C> -> tensor<128x128xf32, #CL>
```
**EN:** This function-oriented block defines or enters `matmul_add`. Within it, the test exercises constants, broadcasted scalars or pointers, layout conversions, tt.func, masked or vectorized loads, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `matmul_add` 为核心。测试在其中演示 常量、广播后的标量或指针、布局转换、tt.func、带掩码或向量化的加载，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 2806-2810
```mlir
2806|       // CHECK: %[[T0:.*]] = tt.dot
2807|       // CHECK: arith.addf %{{.*}}, %[[T0]] : tensor<128x128xf32, #mma>
2808|       %t2 = arith.addf %prev_c, %t : tensor<128x128xf32, #CL>
2809|       %next_a_ptr = tt.addptr %a_ptr, %a_off : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xi32, #AL>
2810|       %next_b_ptr = tt.addptr %b_ptr, %b_off : tensor<32x128x!tt.ptr<f16>, #BL>, tensor<32x128xi32, #BL>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on pointer arithmetic, floating-point additions. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 指针算术、浮点加法。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 2811-2814
```mlir
2811|       // CHECK: scf.yield
2812|       scf.yield %next_a_ptr, %next_b_ptr, %t2 : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #CL>
2813|     }
2814| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值。

### Lines 2815-2820
```mlir
2815|     // CHECK: ttg.convert_layout {{.*}} : tensor<128x128xf32, #mma> -> tensor<128x128xf32, #blocked
2816|     tt.store %c_ptr_init, %100#2 : tensor<128x128x!tt.ptr<f32>, #CL>
2817|     tt.return
2818|   }
2819| }
2820| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized stores, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的存储、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 2821-2821
```mlir
2821| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 2822-2826
```mlir
2822| 
2823| // Minimized reproducer for compiler crash during remove layouts conversions pass:
2824| // If dot result transformed into tensor with shape smaller than one MFMA instruction size, it triggers various asserts.
2825| // This is a smoke test that checks that compiler do not crash.
2826| //
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 2827-2828
```mlir
2827| // CHECK-LABEL: small_tensor_mfma
2828| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: small_tensor_mfma anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: small_tensor_mfma 这样的标签用于锚定匹配范围。

### Lines 2829-2831
```mlir
2829| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [16, 4], warpsPerCTA = [1, 1], order = [0, 1]}>
2830| #mma = #ttg.amd_mfma<{version = 2, warpsPerCTA = [1, 1], instrShape = [32, 32, 8], isTransposed = true}>
2831| #mma1 = #ttg.amd_mfma<{version = 2, warpsPerCTA = [1, 1], instrShape = [16, 16, 16], isTransposed = true}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 2832-2832
```mlir
2832| module attributes {"ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 2833-2850
```mlir
2833|   tt.func public @small_tensor_mfma(%arg0: !tt.ptr<f32>) {
2834|     %cst = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #mma>
2835|     %cst_0 = arith.constant dense<1.230000e+02> : tensor<32x32xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
2836|     %cst_1 = arith.constant dense<1.230000e+02> : tensor<32x32xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>
2837|     %cst_2 = arith.constant dense<1.230000e+02> : tensor<32x32xf32, #ttg.dot_op<{opIdx = 0, parent = #mma1, kWidth = 4}>>
2838|     %cst_3 = arith.constant dense<1.230000e+02> : tensor<32x16xf32, #mma1>
2839|     %0 = tt.dot %cst_0, %cst_1, %cst, inputPrecision = tf32 : tensor<32x32xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<32x32xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<32x32xf32, #mma>
2840|     %1 = ttg.convert_layout %0 : tensor<32x32xf32, #mma> -> tensor<32x32xf32, #blocked>
2841|     %2 = "tt.reduce" (%1) ({
2842|     ^bb0(%arg1: f32, %arg2: f32):
2843|       %3 = arith.addf %arg1, %arg2 : f32
2844|       tt.reduce.return %3 : f32
2845|     }) {axis = 1 : i32} : (tensor<32x32xf32, #blocked>) -> tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
2846|     %4 = tt.expand_dims %2 {axis = 1 : i32} : tensor<32xf32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<32x1xf32, #blocked>
2847|     %5 = tt.broadcast %4 : tensor<32x1xf32, #blocked> -> tensor<32x16xf32, #blocked>
2848|     %6 = ttg.convert_layout %5 : tensor<32x16xf32, #blocked> -> tensor<32x16xf32, #ttg.dot_op<{opIdx = 1, parent = #mma1, kWidth = 4}>>
2849|     %7 = tt.dot %cst_2, %6, %cst_3, inputPrecision = tf32 : tensor<32x32xf32, #ttg.dot_op<{opIdx = 0, parent = #mma1, kWidth = 4}>> * tensor<32x16xf32, #ttg.dot_op<{opIdx = 1, parent = #mma1, kWidth = 4}>> -> tensor<32x16xf32, #mma1>
2850|     %addr = tt.splat %arg0 : !tt.ptr<f32> -> tensor<32x16x!tt.ptr<f32>, #blocked>
```
**EN:** This function-oriented block defines or enters `small_tensor_mfma`. Within it, the test exercises constants, tt.func, dot-product or MMA-style math, layout conversions, floating-point additions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `small_tensor_mfma` 为核心。测试在其中演示 常量、tt.func、点积或 MMA 风格计算、布局转换、浮点加法，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 2851-2856
```mlir
2851|     %8 = ttg.convert_layout %7 : tensor<32x16xf32, #mma1> -> tensor<32x16xf32, #blocked>
2852|     tt.store %addr, %8 : tensor<32x16x!tt.ptr<f32>, #blocked>
2853|     tt.return
2854|   }
2855| }
2856| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, masked or vectorized stores, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、带掩码或向量化的存储、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 2857-2857
```mlir
2857| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 2858-2863
```mlir
2858| 
2859| #blocked = #ttg.blocked<{sizePerThread = [1, 1, 1, 1, 4], threadsPerWarp = [2, 1, 16, 1, 1], warpsPerCTA = [1, 1, 2, 2, 1], order = [4, 0, 1, 2, 3]}>
2860| #blocked2 = #ttg.blocked<{sizePerThread = [1, 1, 1, 1, 4], threadsPerWarp = [1, 1, 32, 1, 1], warpsPerCTA = [1, 1, 1, 1, 4], order = [4, 3, 2, 1, 0]}>
2861| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1, 1, 1, 4], threadsPerWarp = [2, 1, 16, 1, 1], warpsPerCTA = [1, 2, 2, 1, 1], order = [4, 0, 3, 2, 1]}>
2862| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [4, 0, 1, 2, 3]}>
2863| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 2864-2864
```mlir
2864| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 2865-2867
```mlir
2865|   // CHECK-LABEL: lift_convert_to_local_load
2866|   // CHECK-NOT: convert_layout
2867|   // CHECK: tt.return
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: lift_convert_to_local_load anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: lift_convert_to_local_load 这样的标签用于锚定匹配范围。

### Lines 2868-2875
```mlir
2868|   tt.func public @lift_convert_to_local_load(%arg0 : !ttg.memdesc<2x1x32x4x4xi8, #shared, #smem, mutable>) -> tensor<2x4x32x1x4xi8, #blocked2> {
2869|     %1 = ttg.local_load %arg0 : !ttg.memdesc<2x1x32x4x4xi8, #shared, #smem, mutable> -> tensor<2x1x32x4x4xi8, #blocked>
2870|     %2 = tt.trans %1 {order = array<i32: 0, 3, 2, 1, 4>} : tensor<2x1x32x4x4xi8, #blocked> -> tensor<2x4x32x1x4xi8, #blocked1>
2871|     %3 = ttg.convert_layout %2 : tensor<2x4x32x1x4xi8, #blocked1> -> tensor<2x4x32x1x4xi8, #blocked2>
2872|     tt.return %3 : tensor<2x4x32x1x4xi8, #blocked2>
2873|   }
2874| }
2875| 
```
**EN:** This function-oriented block defines or enters `lift_convert_to_local_load`. Within it, the test exercises tt.func, local/shared memory loads, transpose-like layout changes, layout conversions, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `lift_convert_to_local_load` 为核心。测试在其中演示 tt.func、本地/共享内存加载、转置类布局变换、布局转换、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 2876-2876
```mlir
2876| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 2877-2881
```mlir
2877| 
2878| #blocked = #ttg.blocked<{sizePerThread = [2, 2], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
2879| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
2880| #blocked2 = #ttg.blocked<{sizePerThread = [4, 1], threadsPerWarp = [16, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
2881| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 2882-2883
```mlir
2882| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
2883| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 2884-2884
```mlir
2884| tt.func @forward_propagate_layout_gather(%arg0: tensor<1024x256xi32, #blocked>, %arg1: tensor<128x256xf32, #blocked1>) -> tensor<1024x256xf32, #blocked> {
```
**EN:** This function-oriented block defines or enters `forward_propagate_layout_gather`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `forward_propagate_layout_gather` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 2885-2893
```mlir
2885|   // CHECK-LABEL: forward_propagate_layout_gather
2886| 
2887|   // CHECK-NOT: convert_layout
2888|   %0 = ttg.convert_layout %arg0 : tensor<1024x256xi32, #blocked> -> tensor<1024x256xi32, #blocked2>
2889|   %1 = tt.gather %arg1[%0] {axis = 0 : i32} : (tensor<128x256xf32, #blocked1>, tensor<1024x256xi32, #blocked2>) -> tensor<1024x256xf32, #blocked2>
2890|   %2 = ttg.convert_layout %1 : tensor<1024x256xf32, #blocked2> -> tensor<1024x256xf32, #blocked>
2891|   tt.return %2 : tensor<1024x256xf32, #blocked>
2892| }
2893| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, tt.gather, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、tt.gather、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 2894-2894
```mlir
2894| tt.func @forward_only_propagation(%arg0: tensor<1024x256xi32, #blocked>, %arg1: tensor<128x256xf32, #blocked1>) -> tensor<1024x256xf32, #blocked1> {
```
**EN:** This function-oriented block defines or enters `forward_only_propagation`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `forward_only_propagation` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 2895-2899
```mlir
2895|   // CHECK-LABEL: forward_only_propagation
2896| 
2897|   // CHECK-NEXT: [[GATHER:%.*]] = tt.gather
2898|   %0 = ttg.convert_layout %arg0 : tensor<1024x256xi32, #blocked> -> tensor<1024x256xi32, #blocked2>
2899|   %1 = tt.gather %arg1[%0] {axis = 0 : i32} : (tensor<128x256xf32, #blocked1>, tensor<1024x256xi32, #blocked2>) -> tensor<1024x256xf32, #blocked2>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, tt.gather. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、tt.gather。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 2900-2901
```mlir
2900|   // CHECK-NEXT: [[RES:%.*]] = ttg.convert_layout [[GATHER]] : tensor<1024x256xf32, #blocked> -> tensor<1024x256xf32, #blocked1>
2901|   %2 = ttg.convert_layout %1 : tensor<1024x256xf32, #blocked2> -> tensor<1024x256xf32, #blocked1>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 2902-2905
```mlir
2902|   // CHECK-NEXT: return [[RES]]
2903|   tt.return %2 : tensor<1024x256xf32, #blocked1>
2904| }
2905| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 2906-2906
```mlir
2906| tt.func @backward_remat_gather_layout(%arg0: tensor<64x64xf32, #blocked1>) -> tensor<1x64xf32, #blocked1> {
```
**EN:** This function-oriented block defines or enters `backward_remat_gather_layout`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `backward_remat_gather_layout` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 2907-2912
```mlir
2907|   // CHECK-LABEL: backward_remat_gather_layout
2908| 
2909|   %0 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
2910|   %1 = tt.expand_dims %0 {axis = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x64xi32, #blocked>
2911|   %2 = tt.gather %arg0[%1] {axis = 0 : i32} : (tensor<64x64xf32, #blocked1>, tensor<1x64xi32, #blocked>) -> tensor<1x64xf32, #blocked>
2912| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on lane/block index ranges, shape expansion, tt.gather. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 lane/block 索引范围、形状扩展、tt.gather。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 2913-2917
```mlir
2913|   // CHECK-NOT: convert_layout
2914|   %3 = ttg.convert_layout %2 : tensor<1x64xf32, #blocked> -> tensor<1x64xf32, #blocked1>
2915|   tt.return %3 : tensor<1x64xf32, #blocked1>
2916| }
2917| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 2918-2918
```mlir
2918| tt.func @do_not_propagate(%arg0: tensor<1024x256xi32, #blocked>, %arg1: tensor<128x256xf32, #blocked1>) -> tensor<1024x256xf32, #blocked> {
```
**EN:** This function-oriented block defines or enters `do_not_propagate`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `do_not_propagate` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 2919-2921
```mlir
2919|   // CHECK-LABEL: do_not_propagate
2920| 
2921|   %0 = ttg.convert_layout %arg0 : tensor<1024x256xi32, #blocked> -> tensor<1024x256xi32, #blocked2>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 2922-2927
```mlir
2922|   // CHECK: tt.gather {{.*}} (tensor<128x256xf32, #blocked1>, tensor<1024x256xi32, #blocked2>) -> tensor<1024x256xf32, #blocked2>
2923|   %1 = tt.gather %arg1[%0] {axis = 0 : i32, efficient_layout} : (tensor<128x256xf32, #blocked1>, tensor<1024x256xi32, #blocked2>) -> tensor<1024x256xf32, #blocked2>
2924|   %2 = ttg.convert_layout %1 : tensor<1024x256xf32, #blocked2> -> tensor<1024x256xf32, #blocked>
2925|   tt.return %2 : tensor<1024x256xf32, #blocked>
2926| }
2927| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.gather, layout conversions, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.gather、布局转换、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 2928-2928
```mlir
2928| tt.func @do_not_remat(%arg0: tensor<64x64xf32, #blocked1>) -> tensor<1x64xf32, #blocked1> {
```
**EN:** This function-oriented block defines or enters `do_not_remat`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `do_not_remat` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 2929-2932
```mlir
2929|   // CHECK-LABEL: do_not_remat
2930| 
2931|   %0 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
2932|   %1 = tt.expand_dims %0 {axis = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x64xi32, #blocked>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on lane/block index ranges, shape expansion. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 lane/block 索引范围、形状扩展。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 2933-2941
```mlir
2933|   // CHECK: tt.gather {{.*}} (tensor<64x64xf32, #blocked1>, tensor<1x64xi32, #blocked>) -> tensor<1x64xf32, #blocked>
2934|   %2 = tt.gather %arg0[%1] {axis = 0 : i32, efficient_layout} : (tensor<64x64xf32, #blocked1>, tensor<1x64xi32, #blocked>) -> tensor<1x64xf32, #blocked>
2935| 
2936|   %3 = ttg.convert_layout %2 : tensor<1x64xf32, #blocked> -> tensor<1x64xf32, #blocked1>
2937|   tt.return %3 : tensor<1x64xf32, #blocked1>
2938| }
2939| 
2940| }
2941| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.gather, layout conversions, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.gather、布局转换、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 2942-2942
```mlir
2942| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 2943-2946
```mlir
2943| 
2944| #blocked = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
2945| #blocked1 = #ttg.blocked<{sizePerThread = [2, 1], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [0, 1]}>
2946| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 2947-2948
```mlir
2947| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
2948| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 2949-2949
```mlir
2949| // CHECK-LABEL: reuse_layout_conversion
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: reuse_layout_conversion anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: reuse_layout_conversion 这样的标签用于锚定匹配范围。

### Lines 2950-2950
```mlir
2950| tt.func @reuse_layout_conversion(%arg0: tensor<64x64xf32, #blocked>) -> (tensor<64x64xf32, #blocked>, tensor<64x64xf32, #blocked>) {
```
**EN:** This function-oriented block defines or enters `reuse_layout_conversion`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `reuse_layout_conversion` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 2951-2952
```mlir
2951|   // CHECK-NEXT: %cst = arith.constant {{.*}} tensor<64x64xf32, #blocked>
2952|   %cst = arith.constant dense<2.000000e+00> : tensor<64x64xf32, #blocked1>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 2953-2954
```mlir
2953|   // CHECK-NEXT: [[TRANS:%.*]] = tt.trans %arg0 {{.*}} tensor<64x64xf32, #blocked> -> tensor<64x64xf32, #blocked1>
2954|   %0 = tt.trans %arg0 {order = array<i32: 1, 0>} : tensor<64x64xf32, #blocked> -> tensor<64x64xf32, #blocked1>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on transpose-like layout changes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 转置类布局变换。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 2955-2956
```mlir
2955|   // CHECK-NEXT: [[CVT:%.*]] = ttg.convert_layout [[TRANS]] : tensor<64x64xf32, #blocked1> -> tensor<64x64xf32, #blocked>
2956|   %1 = ttg.convert_layout %0 : tensor<64x64xf32, #blocked1> -> tensor<64x64xf32, #blocked>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 2957-2959
```mlir
2957|   // CHECK-NEXT: [[RESULT:%.*]] = arith.mulf [[CVT]], %cst : tensor<64x64xf32, #blocked>
2958|   %2 = arith.mulf %0, %cst : tensor<64x64xf32, #blocked1>
2959|   %3 = ttg.convert_layout %2 : tensor<64x64xf32, #blocked1> -> tensor<64x64xf32, #blocked>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on arith.mulf, layout conversions. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 arith.mulf、布局转换。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 2960-2963
```mlir
2960|   // CHECK-NEXT: return [[CVT]], [[RESULT]]
2961|   tt.return %1, %3 : tensor<64x64xf32, #blocked>, tensor<64x64xf32, #blocked>
2962| }
2963| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 2964-2964
```mlir
2964| // CHECK-LABEL: respect_dominance
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: respect_dominance anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: respect_dominance 这样的标签用于锚定匹配范围。

### Lines 2965-2967
```mlir
2965| tt.func @respect_dominance(%arg0: tensor<64x64xf32, #blocked>) -> (tensor<64x64xf32, #blocked>, tensor<64x64xf32, #blocked>) {
2966|   %cst = arith.constant dense<2.000000e+00> : tensor<64x64xf32, #blocked1>
2967| 
```
**EN:** This function-oriented block defines or enters `respect_dominance`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `respect_dominance` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 2968-2976
```mlir
2968|   // CHECK-COUNT-2: convert_layout
2969|   %0 = tt.trans %arg0 {order = array<i32: 1, 0>} : tensor<64x64xf32, #blocked> -> tensor<64x64xf32, #blocked1>
2970| 
2971|   %2 = arith.mulf %0, %cst : tensor<64x64xf32, #blocked1>
2972|   %1 = ttg.convert_layout %0 : tensor<64x64xf32, #blocked1> -> tensor<64x64xf32, #blocked>
2973|   %3 = ttg.convert_layout %2 : tensor<64x64xf32, #blocked1> -> tensor<64x64xf32, #blocked>
2974|   tt.return %1, %3 : tensor<64x64xf32, #blocked>, tensor<64x64xf32, #blocked>
2975| }
2976| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, transpose-like layout changes, arith.mulf, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、转置类布局变换、arith.mulf、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 2977-2977
```mlir
2977| // CHECK-LABEL: remat_across_regions
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: remat_across_regions anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: remat_across_regions 这样的标签用于锚定匹配范围。

### Lines 2978-2978
```mlir
2978| tt.func @remat_across_regions(%arg0: i1, %arg1: tensor<8x8xf32, #blocked>) {
```
**EN:** This function-oriented block defines or enters `remat_across_regions`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `remat_across_regions` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 2979-2980
```mlir
2979|   // CHECK-NEXT: scf.if
2980|   scf.if %arg0 {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured conditionals.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化条件分支。

### Lines 2981-2983
```mlir
2981|     // CHECK-NEXT: convert_layout
2982|     %0 = ttg.convert_layout %arg1 : tensor<8x8xf32, #blocked> -> tensor<8x8xf32, #blocked1>
2983|     "test.keep"(%0) : (tensor<8x8xf32, #blocked1>) -> ()
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 2984-2986
```mlir
2984|   // CHECK: else
2985|   } else {
2986|     %0 = "test.dummy"() : () -> i32
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 2987-2989
```mlir
2987|     // CHECK: convert_layout
2988|     %1 = ttg.convert_layout %arg1 : tensor<8x8xf32, #blocked> -> tensor<8x8xf32, #blocked1>
2989|     "test.keep"(%1) : (tensor<8x8xf32, #blocked1>) -> ()
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 2990-2991
```mlir
2990|   // CHECK: }
2991|   }
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 2992-2997
```mlir
2992|   // CHECK-NEXT: return
2993|   tt.return
2994| }
2995| 
2996| }
2997| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 2998-2998
```mlir
2998| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 2999-3002
```mlir
2999| 
3000| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
3001| #mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 16]}>
3002| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 3003-3004
```mlir
3003| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100"} {
3004| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 3005-3005
```mlir
3005| // CHECK-LABEL: @hoist_one_conditional
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @hoist_one_conditional anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @hoist_one_conditional 这样的标签用于锚定匹配范围。

### Lines 3006-3010
```mlir
3006| tt.func @hoist_one_conditional(
3007|     %arg0: i1,
3008|     %arg1: tensor<128x32x!tt.ptr<f32>, #blocked>
3009| ) -> tensor<128x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> {
3010| 
```
**EN:** This function-oriented block defines or enters `hoist_one_conditional`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `hoist_one_conditional` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 3011-3012
```mlir
3011|   // CHECK: arith.constant {{.*}} tensor<128x32xf32, #blocked>
3012|   %cst = arith.constant dense<0.000000e+00> : tensor<128x32xf32, #blocked>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 3013-3014
```mlir
3013|   // CHECK: scf.if
3014|   %0 = scf.if %arg0 -> (tensor<128x32xf32, #blocked>) {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured conditionals.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化条件分支。

### Lines 3015-3016
```mlir
3015|     // CHECK-NEXT: [[RES:%.*]] = tt.load
3016|     %3 = tt.load %arg1 : tensor<128x32x!tt.ptr<f32>, #blocked>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized loads. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的加载。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 3017-3021
```mlir
3017|     // CHECK-NEXT: yield [[RES]]
3018|     scf.yield %3 : tensor<128x32xf32, #blocked>
3019|   } else {
3020|     scf.yield %cst : tensor<128x32xf32, #blocked>
3021|   }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值。

### Lines 3022-3023
```mlir
3022|   // CHECK: [[TRUNC:%.*]] = arith.truncf
3023|   %1 = arith.truncf %0 : tensor<128x32xf32, #blocked> to tensor<128x32xf16, #blocked>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on arith.truncf. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 arith.truncf。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 3024-3028
```mlir
3024|   // CHECK-NEXT: convert_layout [[TRUNC]]
3025|   %2 = ttg.convert_layout %1 : tensor<128x32xf16, #blocked> -> tensor<128x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
3026|   tt.return %2 : tensor<128x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
3027| }
3028| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 3029-3029
```mlir
3029| // CHECK-LABEL: @hoist_multiple_conditional
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @hoist_multiple_conditional anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @hoist_multiple_conditional 这样的标签用于锚定匹配范围。

### Lines 3030-3037
```mlir
3030| tt.func @hoist_multiple_conditional(
3031|     %arg0: i1,
3032|     %arg1: i1,
3033|     %arg2: tensor<128x32x!tt.ptr<f32>, #blocked>,
3034|     %arg3: tensor<128x32x!tt.ptr<f32>, #blocked>,
3035|     %arg4: tensor<32x128xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>,
3036|     %arg5: tensor<128x128xf32, #mma>
3037| ) -> tensor<128x128xf32, #mma> {
```
**EN:** This function-oriented block defines or enters `hoist_multiple_conditional`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `hoist_multiple_conditional` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 3038-3055
```mlir
3038|   // CHECK-COUNT-1: ttg.convert_layout
3039|   %cst0 = arith.constant dense<1.0> : tensor<128x32xf32, #blocked>
3040|   %cst1 = arith.constant dense<2.0> : tensor<128x32xf32, #blocked>
3041|   %0 = scf.if %arg0 -> (tensor<128x32xf32, #blocked>) {
3042|     %3 = tt.load %arg2 : tensor<128x32x!tt.ptr<f32>, #blocked>
3043|     scf.yield %3 : tensor<128x32xf32, #blocked>
3044|   } else {
3045|     scf.yield %cst0 : tensor<128x32xf32, #blocked>
3046|   }
3047|   %1 = scf.if %arg1 -> (tensor<128x32xf32, #blocked>) {
3048|     %4 = tt.load %arg3 : tensor<128x32x!tt.ptr<f32>, #blocked>
3049|     scf.yield %4 : tensor<128x32xf32, #blocked>
3050|   } else {
3051|     scf.yield %cst1 : tensor<128x32xf32, #blocked>
3052|   }
3053|   %2 = arith.addf %0, %1 : tensor<128x32xf32, #blocked>
3054|   %3 = ttg.convert_layout %2 : tensor<128x32xf32, #blocked> -> tensor<128x32xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
3055|   %4 = tt.dot %3, %arg4, %arg5, inputPrecision = tf32 : tensor<128x32xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<32x128xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<128x128xf32, #mma>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, constants, structured conditionals, masked or vectorized loads, floating-point additions.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、常量、结构化条件分支、带掩码或向量化的加载、浮点加法。

### Lines 3056-3058
```mlir
3056|   tt.return %4 : tensor<128x128xf32, #mma>
3057| }
3058| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 3059-3059
```mlir
3059| // CHECK-LABEL: @hoist_across_loop
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @hoist_across_loop anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @hoist_across_loop 这样的标签用于锚定匹配范围。

### Lines 3060-3065
```mlir
3060| tt.func @hoist_across_loop(
3061|     %arg0: i1,
3062|     %arg1: tensor<128x32x!tt.ptr<f32>, #blocked>,
3063|     %arg2: tensor<32x128xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>,
3064|     %arg3: tensor<128x128xf32, #mma>
3065| ) -> tensor<128x128xf32, #mma> {
```
**EN:** This function-oriented block defines or enters `hoist_across_loop`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `hoist_across_loop` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 3066-3070
```mlir
3066|   // CHECK: arith.constant {{.*}} tensor<128x32xf32, #ttg.dot_op
3067|   %cst = arith.constant dense<1.0> : tensor<128x32xf32, #blocked>
3068|   %c0_i32 = arith.constant 0 : i32
3069|   %c1_i32 = arith.constant 1 : i32
3070|   %c32_i32 = arith.constant 32 : i32
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 3071-3072
```mlir
3071|   // CHECK: scf.for
3072|   %0:2 = scf.for %i = %c0_i32 to %c32_i32 step %c1_i32 iter_args(%arg4 = %cst, %acc = %arg3) -> (tensor<128x32xf32, #blocked>, tensor<128x128xf32, #mma>) : i32 {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 3073-3074
```mlir
3073|     // CHECK-NEXT: scf.if
3074|     %1 = scf.if %arg0 -> (tensor<128x32xf32, #blocked>) {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured conditionals.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化条件分支。

### Lines 3075-3081
```mlir
3075|       // CHECK-NEXT: [[RES:%.*]] = tt.load
3076|       // CHECK-NEXT: ttg.convert_layout [[RES]]
3077|       %3 = tt.load %arg1 : tensor<128x32x!tt.ptr<f32>, #blocked>
3078|       scf.yield %3 : tensor<128x32xf32, #blocked>
3079|     } else {
3080|       scf.yield %arg4 : tensor<128x32xf32, #blocked>
3081|     }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, masked or vectorized loads.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、带掩码或向量化的加载。

### Lines 3082-3089
```mlir
3082|     // CHECK-NOT: ttg.convert_layout
3083|     %2 = ttg.convert_layout %1 : tensor<128x32xf32, #blocked> -> tensor<128x32xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
3084|     %3 = tt.dot %2, %arg2, %acc, inputPrecision = tf32 : tensor<128x32xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<32x128xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<128x128xf32, #mma>
3085|     scf.yield %1, %3 : tensor<128x32xf32, #blocked>, tensor<128x128xf32, #mma>
3086|   }
3087|   tt.return %0#1 : tensor<128x128xf32, #mma>
3088| }
3089| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining layout conversions, dot-product or MMA-style math, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 布局转换、点积或 MMA 风格计算、循环/分支产出值、tt.return。

### Lines 3090-3090
```mlir
3090| // CHECK-LABEL: @chained_if
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @chained_if anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @chained_if 这样的标签用于锚定匹配范围。

### Lines 3091-3091
```mlir
3091| tt.func @chained_if(%arg0: i1, %arg1: i1, %arg2: tensor<32x32x!tt.ptr<f32>, #blocked>, %arg3: tensor<32x32x!tt.ptr<f32>, #blocked>) -> tensor<32x32xf32, #mma> {
```
**EN:** This function-oriented block defines or enters `chained_if`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `chained_if` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 3092-3109
```mlir
3092|   // CHECK-COUNT-1: ttg.convert_layout
3093|   %cst = arith.constant dense<1.0> : tensor<32x32xf32, #blocked>
3094|   %0 = scf.if %arg0 -> tensor<32x32xf32, #blocked> {
3095|     %anchor = tt.load %arg2 : tensor<32x32x!tt.ptr<f32>, #blocked>
3096|     scf.yield %anchor : tensor<32x32xf32, #blocked>
3097|   } else {
3098|     scf.yield %cst : tensor<32x32xf32, #blocked>
3099|   }
3100|   %1 = scf.if %arg1 -> tensor<32x32xf32, #blocked> {
3101|     %anchor = tt.load %arg3 : tensor<32x32x!tt.ptr<f32>, #blocked>
3102|     scf.yield %anchor : tensor<32x32xf32, #blocked>
3103|   } else {
3104|     scf.yield %0 : tensor<32x32xf32, #blocked>
3105|   }
3106|   %2 = ttg.convert_layout %1 : tensor<32x32xf32, #blocked> -> tensor<32x32xf32, #mma>
3107|   tt.return %2 : tensor<32x32xf32, #mma>
3108| }
3109| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, structured conditionals, masked or vectorized loads, constants, layout conversions.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、结构化条件分支、带掩码或向量化的加载、常量、布局转换。

### Lines 3110-3111
```mlir
3110| }
3111| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 3112-3112
```mlir
3112| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 3113-3116
```mlir
3113| 
3114| #blocked = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
3115| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
3116| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 3117-3118
```mlir
3117| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
3118| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 3119-3119
```mlir
3119| // CHECK-LABEL: @cvt_in_peeled_prologue
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @cvt_in_peeled_prologue anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @cvt_in_peeled_prologue 这样的标签用于锚定匹配范围。

### Lines 3120-3123
```mlir
3120| tt.func @cvt_in_peeled_prologue(%arg0: tensor<32x32x!tt.ptr<bf16>, #blocked>, %arg1: i1, %arg2: i32, %arg3: i32, %arg4: i1) {
3121|   %c1_i32 = arith.constant 1 : i32
3122|   %cst = arith.constant dense<0.0> : tensor<32x32xbf16, #blocked1>
3123| 
```
**EN:** This function-oriented block defines or enters `cvt_in_peeled_prologue`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `cvt_in_peeled_prologue` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 3124-3125
```mlir
3124|   // CHECK: scf.if
3125|   %0 = scf.if %arg1 -> (tensor<32x32xbf16, #blocked1>) {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured conditionals.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化条件分支。

### Lines 3126-3128
```mlir
3126|     // CHECK-NEXT: tt.load
3127|     %1 = tt.load %arg0 : tensor<32x32x!tt.ptr<bf16>, #blocked>
3128|     %2 = ttg.convert_layout %1 : tensor<32x32xbf16, #blocked> -> tensor<32x32xbf16, #blocked1>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized loads, layout conversions. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的加载、布局转换。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 3129-3130
```mlir
3129|     // CHECK-NEXT: yield
3130|     scf.yield %2 : tensor<32x32xbf16, #blocked1>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值。

### Lines 3131-3132
```mlir
3131|     // CHECK-NEXT: else
3132|   } else {
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 3133-3134
```mlir
3133|     // CHECK-NEXT: yield
3134|     scf.yield %cst : tensor<32x32xbf16, #blocked1>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值。

### Lines 3135-3137
```mlir
3135|   // CHECK-NEXT: }
3136|   }
3137| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 3138-3139
```mlir
3138|   // CHECK: [[PEEL1:%.*]] = scf.if
3139|   %1 = scf.if %arg4 -> (tensor<32x32xbf16, #blocked1>) {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured conditionals.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化条件分支。

### Lines 3140-3142
```mlir
3140|     // CHECK-NEXT: tt.load
3141|     %2 = tt.load %arg0 : tensor<32x32x!tt.ptr<bf16>, #blocked>
3142|     %3 = ttg.convert_layout %2 : tensor<32x32xbf16, #blocked> -> tensor<32x32xbf16, #blocked1>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized loads, layout conversions. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的加载、布局转换。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 3143-3144
```mlir
3143|     // CHECK-NEXT: yield
3144|     scf.yield %3 : tensor<32x32xbf16, #blocked1>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值。

### Lines 3145-3146
```mlir
3145|     // CHECK-NEXT: else
3146|   } else {
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 3147-3148
```mlir
3147|     // CHECK-NEXT: yield
3148|     scf.yield %0 : tensor<32x32xbf16, #blocked1>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值。

### Lines 3149-3151
```mlir
3149|   // CHECK-NEXT: }
3150|   }
3151| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 3152-3154
```mlir
3152|   // CHECK-NEXT: [[CVT:%.*]] = ttg.convert_layout [[PEEL1]]
3153|   // CHECK-NEXT: scf.for {{.*}} iter_args(%{{arg[0-9]+}} = [[CVT]])
3154|   %3 = scf.for %i = %arg2 to %arg3 step %c1_i32 iter_args(%k = %1) -> (tensor<32x32xbf16, #blocked1>) : i32 {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 3155-3156
```mlir
3155|     // CHECK-NEXT: scf.if
3156|     %4 = scf.if %arg1 -> (tensor<32x32xbf16, #blocked1>) {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured conditionals.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化条件分支。

### Lines 3157-3158
```mlir
3157|       // CHECK-NEXT: tt.load
3158|       %5 = tt.load %arg0 : tensor<32x32x!tt.ptr<bf16>, #blocked>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized loads. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的加载。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 3159-3167
```mlir
3159|       // CHECK-NEXT: ttg.convert_layout
3160|       %6 = ttg.convert_layout %5 : tensor<32x32xbf16, #blocked> -> tensor<32x32xbf16, #blocked1>
3161|       scf.yield %6 : tensor<32x32xbf16, #blocked1>
3162|     } else {
3163|       scf.yield %k : tensor<32x32xbf16, #blocked1>
3164|     }
3165|     "use.it"(%4) : (tensor<32x32xbf16, #blocked1>) -> ()
3166|     scf.yield %4 : tensor<32x32xbf16, #blocked1>
3167|   }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, layout conversions.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、布局转换。

### Lines 3168-3171
```mlir
3168|   // CHECK-NOT: ttg.convert_layout
3169|   tt.return
3170| }
3171| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 3172-3172
```mlir
3172| // CHECK-LABEL: @cvt_in_loop_if_slice
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @cvt_in_loop_if_slice anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @cvt_in_loop_if_slice 这样的标签用于锚定匹配范围。

### Lines 3173-3176
```mlir
3173| tt.func @cvt_in_loop_if_slice(%arg0: tensor<32x32x!tt.ptr<bf16>, #blocked>, %arg1: i1, %arg2: i32, %arg3: i32, %arg4: i1) {
3174|   %c1_i32 = arith.constant 1 : i32
3175|   %cst = arith.constant dense<0.0> : tensor<32x32xbf16, #blocked>
3176| 
```
**EN:** This function-oriented block defines or enters `cvt_in_loop_if_slice`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `cvt_in_loop_if_slice` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 3177-3178
```mlir
3177|   // CHECK: [[IF_OUT:%.*]] = scf.if
3178|   %0 = scf.if %arg1 -> (tensor<32x32xbf16, #blocked>) {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured conditionals.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化条件分支。

### Lines 3179-3180
```mlir
3179|     // CHECK-NEXT: tt.load
3180|     %1 = tt.load %arg0 : tensor<32x32x!tt.ptr<bf16>, #blocked>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized loads. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的加载。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 3181-3182
```mlir
3181|     // CHECK-NEXT: yield
3182|     scf.yield %1 : tensor<32x32xbf16, #blocked>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值。

### Lines 3183-3184
```mlir
3183|     // CHECK-NEXT: else
3184|   } else {
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 3185-3186
```mlir
3185|     // CHECK-NEXT: yield
3186|     scf.yield %cst : tensor<32x32xbf16, #blocked>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值。

### Lines 3187-3189
```mlir
3187|   // CHECK-NEXT: }
3188|   }
3189| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 3190-3192
```mlir
3190|   // CHECK-NEXT: [[CVT:%.*]] = ttg.convert_layout [[IF_OUT]]
3191|   // CHECK-NEXT: scf.for
3192|   %1 = scf.for %i = %arg2 to %arg3 step %c1_i32 iter_args(%k = %cst) -> tensor<32x32xbf16, #blocked> : i32 {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 3193-3194
```mlir
3193|     // CHECK-NEXT: scf.if
3194|     %4 = scf.if %arg4 -> (tensor<32x32xbf16, #blocked>) {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured conditionals.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化条件分支。

### Lines 3195-3196
```mlir
3195|       // CHECK-NEXT: tt.load
3196|       %5 = tt.load %arg0 : tensor<32x32x!tt.ptr<bf16>, #blocked>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized loads. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的加载。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 3197-3202
```mlir
3197|       // CHECK-NEXT: ttg.convert_layout
3198|       scf.yield %5 : tensor<32x32xbf16, #blocked>
3199|     } else {
3200|       scf.yield %k : tensor<32x32xbf16, #blocked>
3201|     }
3202|     %6 = arith.addf %4, %0 : tensor<32x32xbf16, #blocked>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, floating-point additions.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、浮点加法。

### Lines 3203-3213
```mlir
3203|     // CHECK-NOT: ttg.convert_layout
3204|     %7 = ttg.convert_layout %6 : tensor<32x32xbf16, #blocked> -> tensor<32x32xbf16, #blocked1>
3205|     "use.it"(%7) : (tensor<32x32xbf16, #blocked1>) -> ()
3206|     scf.yield %6 : tensor<32x32xbf16, #blocked>
3207|   }
3208| 
3209|   tt.return
3210| }
3211| 
3212| }
3213| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining layout conversions, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 布局转换、循环/分支产出值、tt.return。

### Lines 3214-3214
```mlir
3214| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 3215-3218
```mlir
3215| 
3216| #linear = #ttg.linear<{register = [[1, 0], [0, 8], [0, 16]], lane = [[2, 0], [4, 0], [8, 0], [16, 0], [0, 1]], warp = [[0, 2], [0, 4]], block = []}>
3217| #blocked = #ttg.blocked<{sizePerThread = [2, 4], threadsPerWarp = [16, 2], warpsPerCTA = [1, 4], order = [1, 0]}>
3218| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 3219-3220
```mlir
3219| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32}  {
3220| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 3221-3221
```mlir
3221| // CHECK-LABEL: reduce_linear_layouts
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: reduce_linear_layouts anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: reduce_linear_layouts 这样的标签用于锚定匹配范围。

### Lines 3222-3222
```mlir
3222| tt.func @reduce_linear_layouts(%arg0: tensor<32x32xi32, #linear>) -> tensor<32xi32, #ttg.slice<{dim = 1, parent = #linear}>> {
```
**EN:** This function-oriented block defines or enters `reduce_linear_layouts`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `reduce_linear_layouts` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 3223-3224
```mlir
3223|   // CHECK-NOT: convert_layout
3224|   %0 = ttg.convert_layout %arg0 : tensor<32x32xi32, #linear> -> tensor<32x32xi32, #blocked>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 3225-3228
```mlir
3225|   // CHECK-NEXT: tt.reduce
3226|   %1 = "tt.reduce" (%0) ({
3227|   ^bb0(%arg1: i32, %arg2: i32):
3228|     tt.reduce.return %arg1 : i32
```
**EN:** This operational block carries the concrete IR steps for the test, centered on reductions. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 归约。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 3229-3236
```mlir
3229|   // CHECK: (tensor<32x32xi32, #linear>) -> tensor<32xi32, #ttg.slice<{dim = 1, parent = #linear}>
3230|   }) {axis = 1 : i32} : (tensor<32x32xi32, #blocked>) -> tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
3231|   %2 = ttg.convert_layout %1 : tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<32xi32, #ttg.slice<{dim = 1, parent = #linear}>>
3232|   tt.return %2 : tensor<32xi32, #ttg.slice<{dim = 1, parent = #linear}>>
3233| }
3234| 
3235| }
3236| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 3237-3237
```mlir
3237| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 3238-3246
```mlir
3238| 
3239| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [2, 2], order = [1, 0]}>
3240| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
3241| #blocked2 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [16, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
3242| #blocked3 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
3243| #blocked4 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [2, 2], order = [0, 1]}>
3244| #blocked5 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [0, 1]}>
3245| #linear = #ttg.linear<{register = [[16, 0]], lane = [[0, 1], [1, 0], [2, 0], [4, 0], [8, 0]], warp = [[0, 0], [0, 0]], block = []}>
3246| #mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [1, 4], instrShape = [16, 8]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 3247-3249
```mlir
3247| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
3248| 
3249|   // Test that after dot_scaled with rhs scales is decomposed, we are able to get rid of the redundant convert_layout
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 3250-3250
```mlir
3250|   // CHECK-LABEL: dot_scale_transpose
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: dot_scale_transpose anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: dot_scale_transpose 这样的标签用于锚定匹配范围。

### Lines 3251-3265
```mlir
3251|   tt.func public @dot_scale_transpose(%arg0: tensor<128x64xf8E4M3FN, #blocked>, %arg1: tensor<32x32xi8, #blocked1>, %arg2: tensor<128x32x!tt.ptr<bf16>, #blocked3>) {
3252|     %cst = arith.constant dense<0.000000e+00> : tensor<128x32xf32, #blocked1>
3253|     %c1_i32 = arith.constant 1 : i32
3254|     %c100_i32 = arith.constant 100 : i32
3255|     %c0_i32 = arith.constant 0 : i32
3256|     %0 = scf.for %arg4 = %c0_i32 to %c100_i32 step %c1_i32 iter_args(%arg5 = %cst) -> (tensor<128x32xf32, #blocked1>)  : i32 {
3257|       %3 = tt.trans %arg0 {order = array<i32: 1, 0>} : tensor<128x64xf8E4M3FN, #blocked> -> tensor<64x128xf8E4M3FN, #blocked4>
3258|       %4 = tt.trans %arg1 {order = array<i32: 1, 0>} : tensor<32x32xi8, #blocked1> -> tensor<32x32xi8, #blocked5>
3259|       %5 = tt.trans %arg5 {order = array<i32: 1, 0>} : tensor<128x32xf32, #blocked1> -> tensor<32x128xf32, #blocked5>
3260|       %6 = ttg.convert_layout %5 : tensor<32x128xf32, #blocked5> -> tensor<32x128xf32, #mma>
3261|       %7 = ttg.convert_layout %4 : tensor<32x32xi8, #blocked5> -> tensor<32x32xi8, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>
3262|       %9 = ttg.fp4_to_fp %7 {axis = 1 : i32} : tensor<32x32xi8, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>> -> tensor<32x64xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>
3263|       %10 = ttg.convert_layout %3 : tensor<64x128xf8E4M3FN, #blocked4> -> tensor<64x128xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>
3264|       %11 = tt.fp_to_fp %10 : tensor<64x128xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>> -> tensor<64x128xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>
3265|       %12 = tt.dot %9, %11, %6 : tensor<32x64xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>> * tensor<64x128xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>> -> tensor<32x128xf32, #mma>
```
**EN:** This function-oriented block defines or enters `dot_scale_transpose`. Within it, the test exercises constants, transpose-like layout changes, layout conversions, tt.func, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `dot_scale_transpose` 为核心。测试在其中演示 常量、转置类布局变换、布局转换、tt.func、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 3266-3272
```mlir
3266|       // CHECK: tt.dot
3267|       // CHECK-NOT: ttg.convert_layout
3268|       // CHECK: scf.yield
3269|       %13 = ttg.convert_layout %12 : tensor<32x128xf32, #mma> -> tensor<32x128xf32, #blocked5>
3270|       %14 = tt.trans %13 {order = array<i32: 1, 0>} : tensor<32x128xf32, #blocked5> -> tensor<128x32xf32, #blocked1>
3271|       scf.yield %14 : tensor<128x32xf32, #blocked1>
3272|     }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining layout conversions, transpose-like layout changes, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 布局转换、转置类布局变换、循环/分支产出值。

### Lines 3273-3282
```mlir
3273|     // CHECK: arith.truncf
3274|     // CHECK-NEXT: ttg.convert_layout
3275|     // CHECK-NEXT: tt.store
3276|     %1 = arith.truncf %0 : tensor<128x32xf32, #blocked1> to tensor<128x32xbf16, #blocked1>
3277|     %2 = ttg.convert_layout %1 : tensor<128x32xbf16, #blocked1> -> tensor<128x32xbf16, #blocked3>
3278|     tt.store %arg2, %2 : tensor<128x32x!tt.ptr<bf16>, #blocked3>
3279|     tt.return
3280|   }
3281| }
3282| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on arith.truncf, layout conversions, masked or vectorized stores, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 arith.truncf、布局转换、带掩码或向量化的存储、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 3283-3283
```mlir
3283| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 3284-3289
```mlir
3284| 
3285| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [2, 16], warpsPerCTA = [8, 1], order = [1, 0]}>
3286| #blocked1 = #ttg.blocked<{sizePerThread = [2, 2], threadsPerWarp = [2, 16], warpsPerCTA = [8, 1], order = [1, 0]}>
3287| #blocked2 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [8], order = [0]}>
3288| #blocked3 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [8, 1], order = [0, 1]}>
3289| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 3290-3291
```mlir
3290| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
3291| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 3292-3300
```mlir
3292| tt.func public @reshape_slice_dot_enc(%arg0: tensor<4x16xi32, #blocked>) -> tensor<64x1xi32, #ttg.dot_op<{opIdx = 1, parent = #blocked1}>> {
3293|   %0 = tt.reshape %arg0 : tensor<4x16xi32, #blocked> -> tensor<64xi32, #blocked2>
3294|   %1 = ttg.convert_layout %0 : tensor<64xi32, #blocked2> -> tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked3}>>
3295|   %2 = tt.expand_dims %1 {axis = 1 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked3}>> -> tensor<64x1xi32, #blocked3>
3296|   %3 = ttg.convert_layout %2 : tensor<64x1xi32, #blocked3> -> tensor<64x1xi32, #ttg.dot_op<{opIdx = 1, parent = #blocked1}>>
3297|   tt.return %3 : tensor<64x1xi32, #ttg.dot_op<{opIdx = 1, parent = #blocked1}>>
3298| }
3299| 
3300| }
```
**EN:** This function-oriented block defines or enters `reshape_slice_dot_enc`. Within it, the test exercises tt.func, layout conversions, tensor reshaping, shape expansion, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `reshape_slice_dot_enc` 为核心。测试在其中演示 tt.func、布局转换、张量重塑、形状扩展、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 3301-3312
```mlir
3301| #Cv2 = #ttg.nvidia_mma<{versionMajor = 2, warpsPerCTA = [4, 1], instrShape = [16, 8]}>
3302| #Av2k1 = #ttg.dot_op<{opIdx = 0, parent = #Cv2, kWidth=1}>
3303| #Bv2k1 = #ttg.dot_op<{opIdx = 1, parent = #Cv2, kWidth=1}>
3304| #Av2k2 = #ttg.dot_op<{opIdx = 0, parent = #Cv2, kWidth=2}>
3305| #Bv2k2 = #ttg.dot_op<{opIdx = 1, parent = #Cv2, kWidth=2}>
3306| #Av2k4 = #ttg.dot_op<{opIdx = 0, parent = #Cv2, kWidth=4}>
3307| #Bv2k4 = #ttg.dot_op<{opIdx = 1, parent = #Cv2, kWidth=4}>
3308| #ALR = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
3309| #ALC = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [0, 1]}>
3310| #BLR = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
3311| #BLC = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [0, 1]}>
3312| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 3313-3314
```mlir
3313| module attributes {"ttg.num-warps" = 4 : i32, "ttg.target" = "cuda:80"} {
3314| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 3315-3323
```mlir
3315| // CHECK: tt.func @push_elementwise
3316| // CHECK: %[[ALOAD:.*]] = tt.load %arg0
3317| // CHECK: %[[ACVT:.*]] = ttg.convert_layout %[[ALOAD]] {{.*}} #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>
3318| // CHECK: %[[AF8E5:.*]] = tt.bitcast %[[ACVT]] {{.*}} #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>
3319| // CHECK: %[[AF16:.*]] = tt.fp_to_fp %[[AF8E5]] {{.*}} #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>
3320| // CHECK: %[[BCVT:.*]] = ttg.convert_layout %{{.*}} : {{.*}} tensor<16x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
3321| // CHECK: %[[C:.*]] = tt.dot %[[AF16]], %[[BCVT]]
3322| // CHECK-SAME: tensor<16x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>> * tensor<16x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>> -> tensor<16x16xf32, #mma>
3323| // CHECK: tt.return %[[C]] : tensor<16x16xf32, #mma>
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 3324-3338
```mlir
3324| tt.func @push_elementwise(
3325|                    %pa: tensor<16x16x!tt.ptr<i8>, #ALR> {tt.divisibility=16: i32, tt.contiguity=2 : i32},
3326|                    %pb: tensor<16x16x!tt.ptr<f16>, #BLC> {tt.divisibility=16: i32, tt.contiguity=2 : i32},
3327|                    %c: tensor<16x16xf32, #Cv2>) -> tensor<16x16xf32, #Cv2>{
3328|   %ai8 = tt.load %pa : tensor<16x16x!tt.ptr<i8>, #ALR>
3329|   %b = tt.load %pb : tensor<16x16x!tt.ptr<f16>, #BLC>
3330|   %af8 = tt.bitcast %ai8: tensor<16x16xi8, #ALR> -> tensor<16x16xf8E5M2, #ALR>
3331|   %a = tt.fp_to_fp %af8: tensor<16x16xf8E5M2, #ALR> -> tensor<16x16xf16, #ALR>
3332|   %dota = ttg.convert_layout %a : tensor<16x16xf16, #ALR> -> tensor<16x16xf16, #Av2k4>
3333|   %dotb = ttg.convert_layout %b : tensor<16x16xf16, #BLC> -> tensor<16x16xf16, #Bv2k4>
3334|   %newc = tt.dot %dota, %dotb, %c : tensor<16x16xf16, #Av2k4> * tensor<16x16xf16, #Bv2k4> -> tensor<16x16xf32, #Cv2>
3335|   tt.return %newc : tensor<16x16xf32, #Cv2>
3336| }
3337| 
3338| 
```
**EN:** This function-oriented block defines or enters `push_elementwise`. Within it, the test exercises tt.func, masked or vectorized loads, layout conversions, tt.bitcast, tt.fp_to_fp, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `push_elementwise` 为核心。测试在其中演示 tt.func、带掩码或向量化的加载、布局转换、tt.bitcast、tt.fp_to_fp，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 3339-3345
```mlir
3339| // CHECK: tt.func @succeeds_if_arg_is_not_convert_layout
3340| // CHECK: %[[ALOAD:.*]] = tt.load %arg0
3341| // CHECK: %[[ACVT:.*]] = ttg.convert_layout %[[ALOAD]]
3342| // CHECK: %[[AF8E5:.*]] = tt.bitcast %[[ACVT]]
3343| // CHECK: %[[AF16:.*]] = tt.fp_to_fp %[[AF8E5]]
3344| // CHECK: %[[C:.*]] = tt.dot %[[AF16]]
3345| // CHECK: tt.return %[[C]] : tensor<16x16xf32, #mma>
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 3346-3359
```mlir
3346| tt.func @succeeds_if_arg_is_not_convert_layout(
3347|                    %pa: tensor<16x16x!tt.ptr<i8>, #ALR> {tt.divisibility=16: i32, tt.contiguity=2 : i32},
3348|                    %pb: tensor<16x16x!tt.ptr<f16>, #BLC> {tt.divisibility=16: i32, tt.contiguity=2 : i32},
3349|                    %c: tensor<16x16xf32, #Cv2>) -> tensor<16x16xf32, #Cv2>{
3350|   %ai8 = tt.load %pa : tensor<16x16x!tt.ptr<i8>, #ALR>
3351|   %dotai8 = ttg.convert_layout %ai8 : tensor<16x16xi8, #ALR> -> tensor<16x16xi8, #Av2k4>
3352|   %b = tt.load %pb : tensor<16x16x!tt.ptr<f16>, #BLC>
3353|   %dotaf8 = tt.bitcast %dotai8 : tensor<16x16xi8, #Av2k4> -> tensor<16x16xf8E5M2, #Av2k4>
3354|   %dota = tt.fp_to_fp %dotaf8 : tensor<16x16xf8E5M2, #Av2k4> -> tensor<16x16xf16, #Av2k4>
3355|   %dotb = ttg.convert_layout %b : tensor<16x16xf16, #BLC> -> tensor<16x16xf16, #Bv2k4>
3356|   %newc = tt.dot %dota, %dotb, %c : tensor<16x16xf16, #Av2k4> * tensor<16x16xf16, #Bv2k4> -> tensor<16x16xf32, #Cv2>
3357|   tt.return %newc : tensor<16x16xf32, #Cv2>
3358| }
3359| 
```
**EN:** This function-oriented block defines or enters `succeeds_if_arg_is_not_convert_layout`. Within it, the test exercises tt.func, masked or vectorized loads, layout conversions, tt.bitcast, tt.fp_to_fp, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `succeeds_if_arg_is_not_convert_layout` 为核心。测试在其中演示 tt.func、带掩码或向量化的加载、布局转换、tt.bitcast、tt.fp_to_fp，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 3360-3366
```mlir
3360| // CHECK: tt.func @push_inline_asm_op
3361| // CHECK: %[[ALOAD:.*]] = tt.load %arg0
3362| // CHECK: %[[ACVT:.*]] = ttg.convert_layout %[[ALOAD]]
3363| // CHECK: %[[AF8E5:.*]] = tt.bitcast %[[ACVT]]
3364| // CHECK: %[[AF16:.*]] = tt.elementwise_inline_asm {{.*}} %[[AF8E5]]
3365| // CHECK: %[[C:.*]] = tt.dot %[[AF16]]
3366| // CHECK: tt.return %[[C]] : tensor<16x16xf32, #mma>
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 3367-3379
```mlir
3367| tt.func @push_inline_asm_op(
3368|                    %pa: tensor<16x16x!tt.ptr<i8>, #ALR> {tt.divisibility=16: i32, tt.contiguity=2 : i32},
3369|                    %dotb: tensor<16x16xf16, #Bv2k4>,
3370|                    %c: tensor<16x16xf32, #Cv2>) -> tensor<16x16xf32, #Cv2>{
3371|   %ai8 = tt.load %pa : tensor<16x16x!tt.ptr<i8>, #ALR>
3372|   %dotaf8 = tt.bitcast %ai8 : tensor<16x16xi8, #ALR> -> tensor<16x16xf8E5M2, #ALR>
3373|   %dota = tt.elementwise_inline_asm "{ cvt.rn.satfinite.e4m3x2.f16x2 $0, $1; }" {constraints = "=r,r", packed_element = 2 : i32, pure = true} %dotaf8 : tensor<16x16xf8E5M2, #ALR> -> tensor<16x16xf16, #ALR>
3374|   %dota_cvt = ttg.convert_layout %dota : tensor<16x16xf16, #ALR> -> tensor<16x16xf16, #Av2k4>
3375|   %newc = tt.dot %dota_cvt, %dotb, %c : tensor<16x16xf16, #Av2k4> * tensor<16x16xf16, #Bv2k4> -> tensor<16x16xf32, #Cv2>
3376|   tt.return %newc : tensor<16x16xf32, #Cv2>
3377| }
3378| }
3379| 
```
**EN:** This function-oriented block defines or enters `push_inline_asm_op`. Within it, the test exercises tt.func, masked or vectorized loads, tt.bitcast, tt.elementwise_inline_asm, layout conversions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `push_inline_asm_op` 为核心。测试在其中演示 tt.func、带掩码或向量化的加载、tt.bitcast、tt.elementwise_inline_asm、布局转换，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 3380-3380
```mlir
3380| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 3381-3384
```mlir
3381| 
3382| #blockedA = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
3383| #blockedB = #ttg.blocked<{sizePerThread = [2, 1], threadsPerWarp = [8, 4], warpsPerCTA = [1, 4], order = [0, 1]}>
3384| #mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [1, 4], instrShape = [16, 8]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 3385-3386
```mlir
3385| module attributes {"ttg.num-warps" = 4 : i32, "ttg.target" = "cuda:80"} {
3386| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 3387-3390
```mlir
3387| // CHECK: #[[BA:.*]] = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
3388| // CHECK: #[[BB:.*]] = #ttg.blocked<{sizePerThread = [2, 1], threadsPerWarp = [8, 4], warpsPerCTA = [1, 4], order = [0, 1]}>
3389| // CHECK: #[[MMA:.*]] = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [1, 4], instrShape = [16, 8]}>
3390| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 3391-3398
```mlir
3391| // CHECK: tt.func @push_convert_both_operands
3392| // CHECK-DAG: %[[ALOAD:.*]] = tt.load %{{.*}} : tensor<16x16x!tt.ptr<f16>, #[[BA]]>
3393| // CHECK-DAG: %[[BLOAD:.*]] = tt.load %{{.*}} : tensor<16x16x!tt.ptr<f16>, #[[BB]]>
3394| // CHECK-DAG: %[[ACVT:.*]] = ttg.convert_layout %[[ALOAD]] : tensor<16x16xf16, #[[BA]]> -> tensor<16x16xf16, #ttg.dot_op<{opIdx = 0, parent = #[[MMA]], kWidth = 2}>>
3395| // CHECK-DAG: %[[AEXT:.*]] = arith.extf %[[ACVT]] : tensor<16x16xf16, #ttg.dot_op<{opIdx = 0, parent = #[[MMA]], kWidth = 2}>> to tensor<16x16xf32, #ttg.dot_op<{opIdx = 0, parent = #[[MMA]], kWidth = 2}>>
3396| // CHECK-DAG: %[[BCVT:.*]] = ttg.convert_layout %[[BLOAD]] : tensor<16x16xf16, #[[BB]]> -> tensor<16x16xf16, #ttg.dot_op<{opIdx = 1, parent = #[[MMA]], kWidth = 2}>>
3397| // CHECK-DAG: %[[BEXT:.*]] = arith.extf %[[BCVT]] : tensor<16x16xf16, #ttg.dot_op<{opIdx = 1, parent = #[[MMA]], kWidth = 2}>> to tensor<16x16xf32, #ttg.dot_op<{opIdx = 1, parent = #[[MMA]], kWidth = 2}>>
3398| // CHECK-DAG: tt.dot %[[AEXT]], %[[BEXT]], %{{.*}}, inputPrecision = tf32 : tensor<16x16xf32, #ttg.dot_op<{opIdx = 0, parent = #[[MMA]], kWidth = 2}>> * tensor<16x16xf32, #ttg.dot_op<{opIdx = 1, parent = #[[MMA]], kWidth = 2}>> -> tensor<16x16xf32, #mma>
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 3399-3414
```mlir
3399| tt.func @push_convert_both_operands(
3400|                    %pa: tensor<16x16x!tt.ptr<f16>, #blockedA> {tt.divisibility=16: i32, tt.contiguity=2 : i32},
3401|                    %pb: tensor<16x16x!tt.ptr<f16>, #blockedB> {tt.divisibility=16: i32, tt.contiguity=2 : i32},
3402|                    %c: tensor<16x16xf32, #mma>) -> tensor<16x16xf32, #mma>{
3403|   %a = tt.load %pa : tensor<16x16x!tt.ptr<f16>, #blockedA>
3404|   %b = tt.load %pb : tensor<16x16x!tt.ptr<f16>, #blockedB>
3405|   %ae = arith.extf %a : tensor<16x16xf16, #blockedA> to tensor<16x16xf32, #blockedA>
3406|   %be = arith.extf %b : tensor<16x16xf16, #blockedB> to tensor<16x16xf32, #blockedB>
3407|   %al = ttg.convert_layout %ae : tensor<16x16xf32, #blockedA> -> tensor<16x16xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
3408|   %bl = ttg.convert_layout %be : tensor<16x16xf32, #blockedB> -> tensor<16x16xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>
3409|   %r = tt.dot %al, %bl, %c, inputPrecision = tf32 : tensor<16x16xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<16x16xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<16x16xf32, #mma>
3410|   tt.return %r : tensor<16x16xf32, #mma>
3411| }
3412| 
3413| }
3414| 
```
**EN:** This function-oriented block defines or enters `push_convert_both_operands`. Within it, the test exercises tt.func, masked or vectorized loads, arith.extf, layout conversions, dot-product or MMA-style math, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `push_convert_both_operands` 为核心。测试在其中演示 tt.func、带掩码或向量化的加载、arith.extf、布局转换、点积或 MMA 风格计算，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 3415-3415
```mlir
3415| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 3416-3419
```mlir
3416| 
3417| #blockedA = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
3418| #blockedB = #ttg.blocked<{sizePerThread = [2, 1], threadsPerWarp = [8, 4], warpsPerCTA = [1, 4], order = [0, 1]}>
3419| #mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [1, 4], instrShape = [16, 8]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 3420-3421
```mlir
3420| module attributes {"ttg.num-warps" = 4 : i32, "ttg.target" = "cuda:80"} {
3421| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 3422-3425
```mlir
3422| // CHECK: #[[BA:.*]] = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
3423| // CHECK: #[[BB:.*]] = #ttg.blocked<{sizePerThread = [2, 1], threadsPerWarp = [8, 4], warpsPerCTA = [1, 4], order = [0, 1]}>
3424| // CHECK: #[[MMA:.*]] = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [1, 4], instrShape = [16, 8]}>
3425| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 3426-3435
```mlir
3426| // CHECK: tt.func @update_kwidth_slice
3427| // CHECK: %[[CST:.+]] = arith.constant dense<1.000000e+00> : tensor<16x16xf32, #ttg.dot_op<{opIdx = 1, parent = #[[MMA]], kWidth = 2}>>
3428| // CHECK-DAG: %[[ALOAD:.*]] = tt.load %{{.*}} : tensor<16x16x!tt.ptr<f16>, #[[BA]]>
3429| // CHECK-DAG: %[[BLOAD:.*]] = tt.load %{{.*}} : tensor<16x16x!tt.ptr<f16>, #[[BB]]>
3430| // CHECK-DAG: %[[ACVT:.*]] = ttg.convert_layout %[[ALOAD]] : tensor<16x16xf16, #[[BA]]> -> tensor<16x16xf16, #ttg.dot_op<{opIdx = 0, parent = #[[MMA]], kWidth = 2}>>
3431| // CHECK-DAG: %[[AEXT:.*]] = arith.extf %[[ACVT]] : tensor<16x16xf16, #ttg.dot_op<{opIdx = 0, parent = #[[MMA]], kWidth = 2}>> to tensor<16x16xf32, #ttg.dot_op<{opIdx = 0, parent = #[[MMA]], kWidth = 2}>>
3432| // CHECK-DAG: %[[BCVT:.*]] = ttg.convert_layout %[[BLOAD]] : tensor<16x16xf16, #[[BB]]> -> tensor<16x16xf16, #ttg.dot_op<{opIdx = 1, parent = #[[MMA]], kWidth = 2}>>
3433| // CHECK-DAG: %[[BEXT:.*]] = arith.extf %[[BCVT]] : tensor<16x16xf16, #ttg.dot_op<{opIdx = 1, parent = #[[MMA]], kWidth = 2}>> to tensor<16x16xf32, #ttg.dot_op<{opIdx = 1, parent = #[[MMA]], kWidth = 2}>>
3434| // CHECK-DAG: %[[ADD:.+]] = arith.addf %[[BEXT]], %[[CST]] : tensor<16x16xf32, #ttg.dot_op<{opIdx = 1, parent = #[[MMA]], kWidth = 2}>>
3435| // CHECK-DAG: tt.dot %[[AEXT]], %[[ADD]], %{{.*}}, inputPrecision = tf32 : tensor<16x16xf32, #ttg.dot_op<{opIdx = 0, parent = #[[MMA]], kWidth = 2}>> * tensor<16x16xf32, #ttg.dot_op<{opIdx = 1, parent = #[[MMA]], kWidth = 2}>> -> tensor<16x16xf32, #mma>
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 3436-3452
```mlir
3436| tt.func @update_kwidth_slice(
3437|                    %pa: tensor<16x16x!tt.ptr<f16>, #blockedA> {tt.divisibility=16: i32, tt.contiguity=2 : i32},
3438|                    %pb: tensor<16x16x!tt.ptr<f16>, #blockedB> {tt.divisibility=16: i32, tt.contiguity=2 : i32},
3439|                    %c: tensor<16x16xf32, #mma>) -> tensor<16x16xf32, #mma>{
3440|   %cst = arith.constant dense<1.000000e+00> : tensor<16x16xf32, #blockedB>
3441|   %a = tt.load %pa : tensor<16x16x!tt.ptr<f16>, #blockedA>
3442|   %b = tt.load %pb : tensor<16x16x!tt.ptr<f16>, #blockedB>
3443|   %ae = arith.extf %a : tensor<16x16xf16, #blockedA> to tensor<16x16xf32, #blockedA>
3444|   %be = arith.extf %b : tensor<16x16xf16, #blockedB> to tensor<16x16xf32, #blockedB>
3445|   %add = arith.addf %be, %cst : tensor<16x16xf32, #blockedB>
3446|   %al = ttg.convert_layout %ae : tensor<16x16xf32, #blockedA> -> tensor<16x16xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
3447|   %bl = ttg.convert_layout %add : tensor<16x16xf32, #blockedB> -> tensor<16x16xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>
3448|   %r = tt.dot %al, %bl, %c, inputPrecision = tf32 : tensor<16x16xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<16x16xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<16x16xf32, #mma>
3449|   tt.return %r : tensor<16x16xf32, #mma>
3450| }
3451| }
3452| 
```
**EN:** This function-oriented block defines or enters `update_kwidth_slice`. Within it, the test exercises tt.func, masked or vectorized loads, arith.extf, layout conversions, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `update_kwidth_slice` 为核心。测试在其中演示 tt.func、带掩码或向量化的加载、arith.extf、布局转换、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 3453-3453
```mlir
3453| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 3454-3456
```mlir
3454| 
3455| #blocked = #ttg.blocked<{sizePerThread = [2, 2], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
3456| #mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 8]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 3457-3457
```mlir
3457| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 3458-3459
```mlir
3458|   // CHECK: tt.func @propagate_dot_op_to_constant()
3459|   // CHECK: arith.constant dense<1.000000e+00> : tensor<64x32xf32, #mma>
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 3460-3470
```mlir
3460|   tt.func @propagate_dot_op_to_constant() -> tensor<64x32xf32, #mma> {
3461|     %cst = arith.constant dense<1.000000e+00> : tensor<128x32xf32, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>
3462|     %cst1 = arith.constant dense<1.000000e+00> : tensor<64x128xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>>
3463|     %cst2 = arith.constant dense<1.000000e+00> : tensor<64x32xf32, #mma>
3464|     %0 = tt.elementwise_inline_asm "cvt.rna.tf32.f32 $0, $1;" {constraints = "=r,r", packed_element = 1 : i32, pure = true} %cst : tensor<128x32xf32, #ttg.dot_op<{opIdx = 1, parent = #blocked}>> -> tensor<128x32xf32, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>
3465|     %1 = ttg.convert_layout %0 : tensor<128x32xf32, #ttg.dot_op<{opIdx = 1, parent = #blocked}>> -> tensor<128x32xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>>
3466|     %2 = tt.dot %cst1, %1, %cst2, inputPrecision = tf32 : tensor<64x128xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>> * tensor<128x32xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>> -> tensor<64x32xf32, #mma>
3467|     tt.return %2 : tensor<64x32xf32, #mma>
3468|   }
3469| }
3470| 
```
**EN:** This function-oriented block defines or enters `propagate_dot_op_to_constant`. Within it, the test exercises constants, tt.func, tt.elementwise_inline_asm, layout conversions, dot-product or MMA-style math, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `propagate_dot_op_to_constant` 为核心。测试在其中演示 常量、tt.func、tt.elementwise_inline_asm、布局转换、点积或 MMA 风格计算，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 3471-3471
```mlir
3471| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 3472-3474
```mlir
3472| 
3473| #blocked = #ttg.blocked<{sizePerThread = [2, 2], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
3474| #mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 8]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 3475-3475
```mlir
3475| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 3476-3477
```mlir
3476|   // CHECK: tt.func @propagate_dot_op_to_constant_above_for()
3477|   // CHECK: arith.constant dense<1.000000e+00> : tensor<32x128xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>>
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 3478-3495
```mlir
3478|   tt.func @propagate_dot_op_to_constant_above_for() -> tensor<32x128xf32, #mma> {
3479|     %cst = arith.constant dense<1.000000e+00> : tensor<32x128xf32, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>
3480|     %cst_0 = arith.constant dense<1.000000e+00> : tensor<32x32xf32, #ttg.dot_op<{opIdx = 0, parent = #blocked}>>
3481|     %cst_1 = arith.constant dense<0.000000e+00> : tensor<32x128xf32, #mma>
3482|     %c0_i32 = arith.constant 0 : i32
3483|     %c32_i32 = arith.constant 32 : i32
3484|     %c128_i32 = arith.constant 128 : i32
3485|     %loop:1 = scf.for %arg2 = %c0_i32 to %c128_i32 step %c32_i32 iter_args(%arg0 = %cst_1) -> (tensor<32x128xf32, #mma>)  : i32 {
3486|       %0 = tt.elementwise_inline_asm "cvt.rna.tf32.f32 $0, $1;" {constraints = "=r,r", packed_element = 1 : i32, pure = true} %cst : tensor<32x128xf32, #ttg.dot_op<{opIdx = 1, parent = #blocked}>> -> tensor<32x128xf32, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>
3487|       %1 = ttg.convert_layout %0 : tensor<32x128xf32, #ttg.dot_op<{opIdx = 1, parent = #blocked}>> -> tensor<32x128xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>>
3488|       %2 = ttg.convert_layout %cst_0 : tensor<32x32xf32, #ttg.dot_op<{opIdx = 0, parent = #blocked}>> -> tensor<32x32xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>>
3489|       %3 = tt.dot %2, %1, %arg0, inputPrecision = tf32 : tensor<32x32xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>> * tensor<32x128xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>> -> tensor<32x128xf32, #mma>
3490|       scf.yield %3 : tensor<32x128xf32, #mma>
3491|     }
3492|     tt.return %loop#0 : tensor<32x128xf32, #mma>
3493|   }
3494| }
3495| 
```
**EN:** This function-oriented block defines or enters `propagate_dot_op_to_constant_above_for`. Within it, the test exercises constants, tt.func, layout conversions, structured loops, tt.elementwise_inline_asm, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `propagate_dot_op_to_constant_above_for` 为核心。测试在其中演示 常量、tt.func、布局转换、结构化循环、tt.elementwise_inline_asm，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 3496-3496
```mlir
3496| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 3497-3499
```mlir
3497| 
3498| #blocked = #ttg.blocked<{sizePerThread = [2, 2], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
3499| #mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 8]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 3500-3504
```mlir
3500| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
3501|   // We currently don't propagate through block arguments on hoistDotOperand
3502|   // that being said, https://github.com/triton-lang/triton/pull/5350
3503|   // allowed to lift DotOperand(opIdx=1), which might be alright
3504| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 3505-3507
```mlir
3505|   // CHECK: tt.func @do_not_propagate_through_block_arguments()
3506|   // CHECK: %[[THROUGH_FOR_OP:.*]] = arith.constant dense<1.000000e+00> : tensor<32x128xf32, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>
3507|   // CHECK: scf.for {{.*}} iter_args(%{{.*}} = %[[THROUGH_FOR_OP]],
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 3508-3525
```mlir
3508|   tt.func @do_not_propagate_through_block_arguments() -> tensor<32x128xf32, #mma> {
3509|     %cst = arith.constant dense<1.000000e+00> : tensor<32x128xf32, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>
3510|     %cst_0 = arith.constant dense<1.000000e+00> : tensor<32x32xf32, #ttg.dot_op<{opIdx = 0, parent = #blocked}>>
3511|     %cst_1 = arith.constant dense<0.000000e+00> : tensor<32x128xf32, #mma>
3512|     %c0_i32 = arith.constant 0 : i32
3513|     %c32_i32 = arith.constant 32 : i32
3514|     %c128_i32 = arith.constant 128 : i32
3515|     %loop:2 = scf.for %arg2 = %c0_i32 to %c128_i32 step %c32_i32 iter_args(%arg0 = %cst, %arg1 = %cst_1) -> (tensor<32x128xf32, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>, tensor<32x128xf32, #mma>)  : i32 {
3516|       %0 = arith.addf %cst, %arg0 : tensor<32x128xf32, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>
3517|       %1 = ttg.convert_layout %0 : tensor<32x128xf32, #ttg.dot_op<{opIdx = 1, parent = #blocked}>> -> tensor<32x128xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>>
3518|       %2 = ttg.convert_layout %cst_0 : tensor<32x32xf32, #ttg.dot_op<{opIdx = 0, parent = #blocked}>> -> tensor<32x32xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>>
3519|       %3 = tt.dot %2, %1, %arg1, inputPrecision = tf32 : tensor<32x32xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>> * tensor<32x128xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>> -> tensor<32x128xf32, #mma>
3520|       scf.yield %0, %3 : tensor<32x128xf32, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>, tensor<32x128xf32, #mma>
3521|     }
3522|     tt.return %loop#1 : tensor<32x128xf32, #mma>
3523|   }
3524| }
3525| 
```
**EN:** This function-oriented block defines or enters `do_not_propagate_through_block_arguments`. Within it, the test exercises constants, tt.func, layout conversions, structured loops, floating-point additions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `do_not_propagate_through_block_arguments` 为核心。测试在其中演示 常量、tt.func、布局转换、结构化循环、浮点加法，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 3526-3526
```mlir
3526| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 3527-3529
```mlir
3527| 
3528| #blocked = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
3529| #mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [1, 4], instrShape = [16, 8]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 3530-3530
```mlir
3530| module attributes {"ttg.num-warps" = 4 : i32, "ttg.target" = "cuda:80"} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 3531-3534
```mlir
3531|   tt.func @dot_op_hoisted_to_load_with_unsupported_op_and_initializer_above_slice(
3532|                     %pa: tensor<16x16x!tt.ptr<f16>, #blocked> {tt.divisibility=16: i32, tt.contiguity=2 : i32},
3533|                     %b: tensor<16x16xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>,
3534|                     %c: tensor<16x16xf32, #mma>) -> tensor<16x16xf32, #mma>{
```
**EN:** This function-oriented block defines or enters `dot_op_hoisted_to_load_with_unsupported_op_and_initializer_above_slice`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `dot_op_hoisted_to_load_with_unsupported_op_and_initializer_above_slice` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 3535-3538
```mlir
3535|     // CHECK: tt.func @dot_op_hoisted_to_load_with_unsupported_op_and_initializer_above_slice
3536|     // This checks that we propagate dot op layout given the following:
3537|     // initializer -> unsupported op -> initializer -> supported ops -> convert,
3538|     // where initializers can be constants or loads.
```
**EN:** This function-oriented block defines or enters `dot_op_hoisted_to_load_with_unsupported_op_and_initializer_above_slice`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `dot_op_hoisted_to_load_with_unsupported_op_and_initializer_above_slice` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 3539-3551
```mlir
3539|     // CHECK: %[[LOAD1:.*]] = tt.load
3540|     // CHECK: ttg.convert_layout %[[LOAD1]]
3541|     %offset = arith.constant dense<16> : tensor<16x1xi32, #blocked>
3542|     %broadcast = tt.broadcast %offset : tensor<16x1xi32, #blocked> -> tensor<16x16xi32, #blocked>
3543|     %pa2 = tt.addptr %pa, %broadcast : tensor<16x16x!tt.ptr<f16>, #blocked>, tensor<16x16xi32, #blocked>
3544|     %a = tt.load %pa2 : tensor<16x16x!tt.ptr<f16>, #blocked>
3545|     %ae = arith.extf %a : tensor<16x16xf16, #blocked> to tensor<16x16xf32, #blocked>
3546|     %ac = ttg.convert_layout %ae : tensor<16x16xf32, #blocked> -> tensor<16x16xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
3547|     %r = tt.dot %ac, %b, %c, inputPrecision = tf32 : tensor<16x16xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<16x16xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<16x16xf32, #mma>
3548|     tt.return %r : tensor<16x16xf32, #mma>
3549|   }
3550| }
3551| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants, tensor broadcasting, pointer arithmetic, masked or vectorized loads, arith.extf. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量、张量广播、指针算术、带掩码或向量化的加载、arith.extf。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 3552-3552
```mlir
3552| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 3553-3557
```mlir
3553| 
3554| #blocked = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
3555| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 64, 16]}>
3556| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
3557| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 3558-3558
```mlir
3558| module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 3559-3563
```mlir
3559| // CHECK: tt.func @mma_v3_reg_push_elementwise
3560| //    CHECK: %[[A_BLOCK:.*]] = tt.load %{{.*}} : tensor<128x64x!tt.ptr<bf16>, #blocked>
3561| //    CHECK: %[[A_DOTOP:.*]] = ttg.convert_layout %[[A_BLOCK]] : tensor<128x64xbf16, #blocked> -> tensor<128x64xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
3562| //    CHECK: %[[A_CASTED:.*]] = tt.fp_to_fp %[[A_DOTOP]] : tensor<128x64xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> -> tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
3563| //    CHECK: %[[R:.*]] = ttng.warp_group_dot %[[A_CASTED]], %{{.*}}, %{{.*}} : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * !ttg.memdesc<64x64xf16, #shared, #smem> -> tensor<128x64xf32, #mma>
```
**EN:** This function-oriented block defines or enters `mma_v3_reg_push_elementwise`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mma_v3_reg_push_elementwise` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 3564-3572
```mlir
3564|   tt.func @mma_v3_reg_push_elementwise(%pa: tensor<128x64x!tt.ptr<bf16>, #blocked>, %dotb: !ttg.memdesc<64x64xf16, #shared, #smem>, %dotc: tensor<128x64xf32, #mma>) -> tensor<128x64xf32, #mma>{
3565|     %a_bf16 = tt.load %pa : tensor<128x64x!tt.ptr<bf16>, #blocked>
3566|     %a = tt.fp_to_fp %a_bf16 : tensor<128x64xbf16, #blocked> -> tensor<128x64xf16, #blocked>
3567|     %dota = ttg.convert_layout %a: tensor<128x64xf16, #blocked> -> tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
3568|     %r = ttng.warp_group_dot %dota, %dotb, %dotc : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * !ttg.memdesc<64x64xf16, #shared, #smem> -> tensor<128x64xf32, #mma>
3569|     tt.return %r : tensor<128x64xf32, #mma>
3570|   }
3571| }
3572| 
```
**EN:** This function-oriented block defines or enters `mma_v3_reg_push_elementwise`. Within it, the test exercises tt.func, masked or vectorized loads, tt.fp_to_fp, layout conversions, ttng.warp_group_dot, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mma_v3_reg_push_elementwise` 为核心。测试在其中演示 tt.func、带掩码或向量化的加载、tt.fp_to_fp、布局转换、ttng.warp_group_dot，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 3573-3573
```mlir
3573| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 3574-3578
```mlir
3574| 
3575| #blocked = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
3576| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 64, 16]}>
3577| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
3578| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 3579-3579
```mlir
3579| module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 3580-3587
```mlir
3580| // CHECK: tt.func @mma_v3_reg_push_elementwise_chained
3581| //    CHECK: %[[CST_DOTOP:.*]] = arith.constant dense<0.000000e+00> : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
3582| //    CHECK: %[[A_BLOCK:.*]] = tt.load %{{.*}} : tensor<128x64x!tt.ptr<i8>, #blocked>
3583| //    CHECK: %[[A_DOTOP:.*]] = ttg.convert_layout %[[A_BLOCK]] : tensor<128x64xi8, #blocked> -> tensor<128x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
3584| //    CHECK: %[[A_CASTED:.*]] = arith.sitofp %[[A_DOTOP]] : tensor<128x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> to tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
3585| //    CHECK: %[[A_SCALED:.*]] = arith.mulf %[[A_CASTED]], %[[CST_DOTOP]] : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
3586| //    CHECK: %[[A_NEGATED:.*]] = arith.negf %[[A_SCALED]] : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
3587| //    CHECK: %[[R:.*]] = ttng.warp_group_dot %[[A_NEGATED]], %{{.*}}, %{{.*}} : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * !ttg.memdesc<64x64xf16, #shared, #smem> -> tensor<128x64xf32, #mma>
```
**EN:** This function-oriented block defines or enters `mma_v3_reg_push_elementwise_chained`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mma_v3_reg_push_elementwise_chained` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 3588-3599
```mlir
3588|   tt.func @mma_v3_reg_push_elementwise_chained(%pa: tensor<128x64x!tt.ptr<i8>, #blocked>, %dotb: !ttg.memdesc<64x64xf16, #shared, #smem>, %dotc: tensor<128x64xf32, #mma>) -> tensor<128x64xf32, #mma>{
3589|     %cst = arith.constant dense<0.000000e+00> : tensor<128x64xf16, #blocked>
3590|     %a_i8 = tt.load %pa : tensor<128x64x!tt.ptr<i8>, #blocked>
3591|     %a_f16 = arith.sitofp %a_i8 : tensor<128x64xi8, #blocked> to tensor<128x64xf16, #blocked>
3592|     %a_scaled = arith.mulf %a_f16, %cst : tensor<128x64xf16, #blocked>
3593|     %a_negated = arith.negf %a_scaled : tensor<128x64xf16, #blocked>
3594|     %dota = ttg.convert_layout %a_negated: tensor<128x64xf16, #blocked> -> tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
3595|     %r = ttng.warp_group_dot %dota, %dotb, %dotc : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * !ttg.memdesc<64x64xf16, #shared, #smem> -> tensor<128x64xf32, #mma>
3596|     tt.return %r : tensor<128x64xf32, #mma>
3597|   }
3598| 
3599| 
```
**EN:** This function-oriented block defines or enters `mma_v3_reg_push_elementwise_chained`. Within it, the test exercises tt.func, constants, masked or vectorized loads, arith.sitofp, arith.mulf, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mma_v3_reg_push_elementwise_chained` 为核心。测试在其中演示 tt.func、常量、带掩码或向量化的加载、arith.sitofp、arith.mulf，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 3600-3607
```mlir
3600|   // CHECK: tt.func @mma_v3_reg_push_elementwise_chained_descritor_load
3601|   //    CHECK: %[[CST_DOTOP:.*]] = arith.constant dense<0.000000e+00> : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
3602|   //    CHECK: %[[A_BLOCK:.*]] = tt.descriptor_load %{{.*}} : !tt.tensordesc<128x64xsi8> -> tensor<128x64xi8, #blocked>
3603|   //    CHECK: %[[A_DOTOP:.*]] = ttg.convert_layout %[[A_BLOCK]] : tensor<128x64xi8, #blocked> -> tensor<128x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
3604|   //    CHECK: %[[A_CASTED:.*]] = arith.sitofp %[[A_DOTOP]] : tensor<128x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> to tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
3605|   //    CHECK: %[[A_SCALED:.*]] = arith.mulf %[[A_CASTED]], %[[CST_DOTOP]] : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
3606|   //    CHECK: %[[A_NEGATED:.*]] = arith.negf %[[A_SCALED]] : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
3607|   //    CHECK: %[[R:.*]] = ttng.warp_group_dot %[[A_NEGATED]], %{{.*}}, %{{.*}} : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * !ttg.memdesc<64x64xf16, #shared, #smem> -> tensor<128x64xf32, #mma>
```
**EN:** This function-oriented block defines or enters `mma_v3_reg_push_elementwise_chained_descritor_load`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mma_v3_reg_push_elementwise_chained_descritor_load` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 3608-3619
```mlir
3608|   tt.func @mma_v3_reg_push_elementwise_chained_descritor_load(%pa: !tt.tensordesc<128x64xsi8>, %dotb: !ttg.memdesc<64x64xf16, #shared, #smem>, %dotc: tensor<128x64xf32, #mma>, %A_dim1: i32, %A_dim2: i32) -> tensor<128x64xf32, #mma>{
3609|     %cst = arith.constant dense<0.000000e+00> : tensor<128x64xf16, #blocked>
3610|     %a_i8 = tt.descriptor_load %pa[%A_dim1, %A_dim2]: !tt.tensordesc<128x64xsi8> -> tensor<128x64xi8, #blocked>
3611|     %a_f16 = arith.sitofp %a_i8 : tensor<128x64xi8, #blocked> to tensor<128x64xf16, #blocked>
3612|     %a_scaled = arith.mulf %a_f16, %cst : tensor<128x64xf16, #blocked>
3613|     %a_negated = arith.negf %a_scaled : tensor<128x64xf16, #blocked>
3614|     %dota = ttg.convert_layout %a_negated: tensor<128x64xf16, #blocked> -> tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
3615|     %r = ttng.warp_group_dot %dota, %dotb, %dotc : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * !ttg.memdesc<64x64xf16, #shared, #smem> -> tensor<128x64xf32, #mma>
3616|     tt.return %r : tensor<128x64xf32, #mma>
3617|   }
3618| }
3619| 
```
**EN:** This function-oriented block defines or enters `mma_v3_reg_push_elementwise_chained_descritor_load`. Within it, the test exercises tt.func, constants, tt.descriptor_load, arith.sitofp, arith.mulf, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mma_v3_reg_push_elementwise_chained_descritor_load` 为核心。测试在其中演示 tt.func、常量、tt.descriptor_load、arith.sitofp、arith.mulf，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 3620-3620
```mlir
3620| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 3621-3623
```mlir
3621| 
3622| #blocked = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
3623| #mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [1, 4], instrShape = [16, 8]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 3624-3624
```mlir
3624| module attributes {"ttg.num-warps" = 4 : i32, "ttg.target" = "cuda:80"} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 3625-3629
```mlir
3625|   tt.func @dot_op_hoisted_to_load_with_unsupported_op_and_initializer_above_slice(
3626|                     %pa1: tensor<16x1x!tt.ptr<f16>, #blocked> {tt.divisibility=16: i32, tt.contiguity=2 : i32},
3627|                     %pa2: tensor<16x16x!tt.ptr<f16>, #blocked> {tt.divisibility=16: i32, tt.contiguity=2 : i32},
3628|                     %b: tensor<16x16xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>,
3629|                     %c: tensor<16x16xf32, #mma>) -> tensor<16x16xf32, #mma>{
```
**EN:** This function-oriented block defines or enters `dot_op_hoisted_to_load_with_unsupported_op_and_initializer_above_slice`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `dot_op_hoisted_to_load_with_unsupported_op_and_initializer_above_slice` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 3630-3631
```mlir
3630|     // CHECK: tt.func @dot_op_hoisted_to_load_with_unsupported_op_and_initializer_above_slice
3631|     // Confirm that both loads feed directly into a convert_layout.
```
**EN:** This function-oriented block defines or enters `dot_op_hoisted_to_load_with_unsupported_op_and_initializer_above_slice`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `dot_op_hoisted_to_load_with_unsupported_op_and_initializer_above_slice` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 3632-3646
```mlir
3632|     // CHECK: %[[LOAD1:.*]] = tt.load
3633|     // CHECK: ttg.convert_layout %[[LOAD1]]
3634|     // CHECK: %[[LOAD2:.*]] = tt.load
3635|     // CHECK: ttg.convert_layout %[[LOAD2]]
3636|     %a1 = tt.load %pa1 : tensor<16x1x!tt.ptr<f16>, #blocked>
3637|     %a2 = tt.load %pa2 : tensor<16x16x!tt.ptr<f16>, #blocked>
3638|     %ab = tt.broadcast %a1 : tensor<16x1xf16, #blocked> -> tensor<16x16xf16, #blocked>
3639|     %aa = arith.addf %ab, %a2 : tensor<16x16xf16, #blocked>
3640|     %ae = arith.extf %aa : tensor<16x16xf16, #blocked> to tensor<16x16xf32, #blocked>
3641|     %ac = ttg.convert_layout %ae : tensor<16x16xf32, #blocked> -> tensor<16x16xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
3642|     %r = tt.dot %ac, %b, %c, inputPrecision = tf32 : tensor<16x16xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<16x16xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<16x16xf32, #mma>
3643|     tt.return %r : tensor<16x16xf32, #mma>
3644|   }
3645| }
3646| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized loads, tensor broadcasting, floating-point additions, arith.extf, layout conversions. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的加载、张量广播、浮点加法、arith.extf、布局转换。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 3647-3647
```mlir
3647| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 3648-3658
```mlir
3648| 
3649| #blocked = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [1, 32], warpsPerCTA = [2, 2], order = [1, 0]}>
3650| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
3651| #blocked2 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
3652| #blocked3 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
3653| #blocked4 = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
3654| #blocked5 = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [16, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
3655| #blocked6 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [2, 2], order = [1, 0]}>
3656| #blocked7 = #ttg.blocked<{sizePerThread = [1, 1, 1], threadsPerWarp = [8, 4, 1], warpsPerCTA = [4, 1, 1], order = [2, 1, 0]}>
3657| #linear = #ttg.linear<{register = [[0, 1], [0, 2], [0, 4], [0, 8], [0, 16]], lane = [[0, 32], [0, 64], [1, 0], [2, 0], [4, 0]], warp = [[8, 0], [16, 0]], block = []}>
3658| #mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [2, 2], instrShape = [16, 8]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 3659-3659
```mlir
3659| module attributes {"ttg.num-warps" = 4 : i32, "ttg.target" = "cuda:80"} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 3660-3671
```mlir
3660|   // CHECK: @remove_layout_dot_scaled
3661|   // CHECK: %[[LOAD1:.*]] = tt.load
3662|   // CHECK: ttg.convert_layout %[[LOAD1]]
3663|   // CHECK: %[[LOAD2:.*]] = tt.load
3664|   // CHECK: ttg.convert_layout %[[LOAD2]]
3665|   // CHECK: %[[LOAD3:.*]] = tt.load
3666|   // CHECK: ttg.convert_layout %[[LOAD3]]
3667|   // CHECK-NOT: ttg.convert_layout
3668|   // CHECK: tt.dot
3669|   // CHECK-NOT: ttg.convert_layout
3670|   // CHECK: %[[STORE:.*]] = ttg.convert_layout
3671|   // CHECK: tt.store %[[PTR:.+]], %[[STORE]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 3672-3689
```mlir
3672|   tt.func @remove_layout_dot_scaled(%arg0: !tt.ptr<i8> {tt.divisibility = 16 : i32}, %arg1: i32 {tt.divisibility = 16 : i32}, %arg2: !tt.ptr<i8> {tt.divisibility = 16 : i32}, %arg3: !tt.ptr<i8> {tt.divisibility = 16 : i32}, %arg4: i32 {tt.divisibility = 16 : i32}, %arg5: !tt.ptr<bf16> {tt.divisibility = 16 : i32}) {
3673|     %cst = arith.constant dense<0x7FC0> : tensor<32x128xbf16, #blocked>
3674|     %cst_0 = arith.constant dense<-1> : tensor<32x4xi8, #blocked1>
3675|     %cst_1 = arith.constant dense<7> : tensor<32x4xi16, #blocked1>
3676|     %cst_2 = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #blocked2>
3677|     %cst_3 = arith.constant dense<32> : tensor<32x1xi32, #blocked3>
3678|     %cst_4 = arith.constant dense<4> : tensor<32x1xi32, #blocked1>
3679|     %0 = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked4}>>
3680|     %1 = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked1}>>
3681|     %2 = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked3}>>
3682|     %3 = tt.expand_dims %0 {axis = 1 : i32} : tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked4}>> -> tensor<32x1xi32, #blocked4>
3683|     %4 = tt.expand_dims %1 {axis = 1 : i32} : tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked1}>> -> tensor<32x1xi32, #blocked1>
3684|     %5 = tt.expand_dims %2 {axis = 1 : i32} : tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked3}>> -> tensor<32x1xi32, #blocked3>
3685|     %6 = tt.splat %arg1 : i32 -> tensor<32x1xi32, #blocked4>
3686|     %7 = arith.muli %3, %6 : tensor<32x1xi32, #blocked4>
3687|     %8 = tt.splat %arg0 : !tt.ptr<i8> -> tensor<32x1x!tt.ptr<i8>, #blocked4>
3688|     %9 = tt.addptr %8, %7 : tensor<32x1x!tt.ptr<i8>, #blocked4>, tensor<32x1xi32, #blocked4>
3689|     %10 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked4}>>
```
**EN:** This function-oriented block defines or enters `remove_layout_dot_scaled`. Within it, the test exercises constants, lane/block index ranges, shape expansion, tt.func, broadcasted scalars or pointers, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `remove_layout_dot_scaled` 为核心。测试在其中演示 常量、lane/block 索引范围、形状扩展、tt.func、广播后的标量或指针，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 3690-3707
```mlir
3690|     %11 = tt.expand_dims %10 {axis = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked4}>> -> tensor<1x64xi32, #blocked4>
3691|     %12 = tt.broadcast %9 : tensor<32x1x!tt.ptr<i8>, #blocked4> -> tensor<32x64x!tt.ptr<i8>, #blocked4>
3692|     %13 = tt.broadcast %11 : tensor<1x64xi32, #blocked4> -> tensor<32x64xi32, #blocked4>
3693|     %14 = tt.addptr %12, %13 : tensor<32x64x!tt.ptr<i8>, #blocked4>, tensor<32x64xi32, #blocked4>
3694|     %15 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked5}>>
3695|     %16 = tt.expand_dims %15 {axis = 1 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked5}>> -> tensor<128x1xi32, #blocked5>
3696|     %17 = tt.splat %arg4 : i32 -> tensor<128x1xi32, #blocked5>
3697|     %18 = arith.muli %16, %17 : tensor<128x1xi32, #blocked5>
3698|     %19 = tt.splat %arg3 : !tt.ptr<i8> -> tensor<128x1x!tt.ptr<i8>, #blocked5>
3699|     %20 = tt.addptr %19, %18 : tensor<128x1x!tt.ptr<i8>, #blocked5>, tensor<128x1xi32, #blocked5>
3700|     %21 = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked5}>>
3701|     %22 = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked3}>>
3702|     %23 = tt.expand_dims %21 {axis = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked5}>> -> tensor<1x32xi32, #blocked5>
3703|     %24 = tt.expand_dims %22 {axis = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked3}>> -> tensor<1x32xi32, #blocked3>
3704|     %25 = tt.broadcast %20 : tensor<128x1x!tt.ptr<i8>, #blocked5> -> tensor<128x32x!tt.ptr<i8>, #blocked5>
3705|     %26 = tt.broadcast %23 : tensor<1x32xi32, #blocked5> -> tensor<128x32xi32, #blocked5>
3706|     %27 = tt.addptr %25, %26 : tensor<128x32x!tt.ptr<i8>, #blocked5>, tensor<128x32xi32, #blocked5>
3707|     %28 = tt.load %14 : tensor<32x64x!tt.ptr<i8>, #blocked4>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shape expansion, tensor broadcasting, pointer arithmetic, lane/block index ranges, broadcasted scalars or pointers. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 形状扩展、张量广播、指针算术、lane/block 索引范围、广播后的标量或指针。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 3708-3725
```mlir
3708|     %29 = ttg.convert_layout %28 : tensor<32x64xi8, #blocked4> -> tensor<32x64xi8, #blocked6>
3709|     %30 = tt.load %27 : tensor<128x32x!tt.ptr<i8>, #blocked5>
3710|     %31 = arith.muli %4, %cst_4 : tensor<32x1xi32, #blocked1>
3711|     %32 = tt.splat %arg2 : !tt.ptr<i8> -> tensor<32x1x!tt.ptr<i8>, #blocked1>
3712|     %33 = tt.addptr %32, %31 : tensor<32x1x!tt.ptr<i8>, #blocked1>, tensor<32x1xi32, #blocked1>
3713|     %34 = tt.make_range {end = 4 : i32, start = 0 : i32} : tensor<4xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
3714|     %35 = tt.expand_dims %34 {axis = 0 : i32} : tensor<4xi32, #ttg.slice<{dim = 0, parent = #blocked1}>> -> tensor<1x4xi32, #blocked1>
3715|     %36 = tt.broadcast %33 : tensor<32x1x!tt.ptr<i8>, #blocked1> -> tensor<32x4x!tt.ptr<i8>, #blocked1>
3716|     %37 = tt.broadcast %35 : tensor<1x4xi32, #blocked1> -> tensor<32x4xi32, #blocked1>
3717|     %38 = tt.addptr %36, %37 : tensor<32x4x!tt.ptr<i8>, #blocked1>, tensor<32x4xi32, #blocked1>
3718|     %39 = tt.load %38 : tensor<32x4x!tt.ptr<i8>, #blocked1>
3719|     %40 = tt.bitcast %30 : tensor<128x32xi8, #blocked5> -> tensor<128x32xf8E4M3FN, #blocked5>
3720|     %41 = ttg.convert_layout %40 : tensor<128x32xf8E4M3FN, #blocked5> -> tensor<128x32xf8E4M3FN, #blocked2>
3721|     %42 = ttg.fp4_to_fp %29 {axis = 1 : i32} : tensor<32x64xi8, #blocked6> -> tensor<32x128xbf16, #blocked>
3722|     %43 = arith.extui %39 : tensor<32x4xi8, #blocked1> to tensor<32x4xi16, #blocked1>
3723|     %44 = arith.shli %43, %cst_1 : tensor<32x4xi16, #blocked1>
3724|     %45 = tt.bitcast %44 : tensor<32x4xi16, #blocked1> -> tensor<32x4xbf16, #blocked1>
3725|     %46 = ttg.convert_layout %45 : tensor<32x4xbf16, #blocked1> -> tensor<32x4xbf16, #ttg.slice<{dim = 2, parent = #blocked7}>>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, masked or vectorized loads, pointer arithmetic, tensor broadcasting, tt.bitcast. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、带掩码或向量化的加载、指针算术、张量广播、tt.bitcast。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 3726-3743
```mlir
3726|     %47 = tt.expand_dims %46 {axis = 2 : i32} : tensor<32x4xbf16, #ttg.slice<{dim = 2, parent = #blocked7}>> -> tensor<32x4x1xbf16, #blocked7>
3727|     %48 = tt.broadcast %47 : tensor<32x4x1xbf16, #blocked7> -> tensor<32x4x32xbf16, #blocked7>
3728|     %49 = tt.reshape %48 : tensor<32x4x32xbf16, #blocked7> -> tensor<32x128xbf16, #linear>
3729|     %50 = ttg.convert_layout %49 : tensor<32x128xbf16, #linear> -> tensor<32x128xbf16, #blocked>
3730|     %51 = arith.mulf %42, %50 : tensor<32x128xbf16, #blocked>
3731|     %52 = arith.cmpi eq, %39, %cst_0 : tensor<32x4xi8, #blocked1>
3732|     %53 = ttg.convert_layout %52 : tensor<32x4xi1, #blocked1> -> tensor<32x4xi1, #ttg.slice<{dim = 2, parent = #blocked7}>>
3733|     %54 = tt.expand_dims %53 {axis = 2 : i32} : tensor<32x4xi1, #ttg.slice<{dim = 2, parent = #blocked7}>> -> tensor<32x4x1xi1, #blocked7>
3734|     %55 = tt.broadcast %54 : tensor<32x4x1xi1, #blocked7> -> tensor<32x4x32xi1, #blocked7>
3735|     %56 = tt.reshape %55 : tensor<32x4x32xi1, #blocked7> -> tensor<32x128xi1, #linear>
3736|     %57 = ttg.convert_layout %56 : tensor<32x128xi1, #linear> -> tensor<32x128xi1, #blocked>
3737|     %58 = arith.select %57, %cst, %51 : tensor<32x128xi1, #blocked>, tensor<32x128xbf16, #blocked>
3738|     %59 = ttg.convert_layout %58 : tensor<32x128xbf16, #blocked> -> tensor<32x128xbf16, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>>
3739|     %60 = tt.fp_to_fp %41 : tensor<128x32xf8E4M3FN, #blocked2> -> tensor<128x32xbf16, #blocked2>
3740|     %61 = ttg.convert_layout %60 : tensor<128x32xbf16, #blocked2> -> tensor<128x32xbf16, #ttg.dot_op<{opIdx = 1, parent = #blocked2}>>
3741|     %62 = ttg.convert_layout %cst_2 : tensor<32x32xf32, #blocked2> -> tensor<32x32xf32, #mma>
3742|     %63 = ttg.convert_layout %59 : tensor<32x128xbf16, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>> -> tensor<32x128xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>
3743|     %64 = ttg.convert_layout %61 : tensor<128x32xbf16, #ttg.dot_op<{opIdx = 1, parent = #blocked2}>> -> tensor<128x32xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, shape expansion, tensor broadcasting, tensor reshaping, arith.mulf. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、形状扩展、张量广播、张量重塑、arith.mulf。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 3744-3759
```mlir
3744|     %65 = tt.dot %63, %64, %62 : tensor<32x128xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>> * tensor<128x32xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>> -> tensor<32x32xf32, #mma>
3745|     %66 = ttg.convert_layout %65 : tensor<32x32xf32, #mma> -> tensor<32x32xf32, #blocked2>
3746|     %67 = ttg.convert_layout %66 : tensor<32x32xf32, #blocked2> -> tensor<32x32xf32, #blocked2>
3747|     %68 = arith.muli %5, %cst_3 : tensor<32x1xi32, #blocked3>
3748|     %69 = tt.splat %arg5 : !tt.ptr<bf16> -> tensor<32x1x!tt.ptr<bf16>, #blocked3>
3749|     %70 = tt.addptr %69, %68 : tensor<32x1x!tt.ptr<bf16>, #blocked3>, tensor<32x1xi32, #blocked3>
3750|     %71 = tt.broadcast %70 : tensor<32x1x!tt.ptr<bf16>, #blocked3> -> tensor<32x32x!tt.ptr<bf16>, #blocked3>
3751|     %72 = tt.broadcast %24 : tensor<1x32xi32, #blocked3> -> tensor<32x32xi32, #blocked3>
3752|     %73 = tt.addptr %71, %72 : tensor<32x32x!tt.ptr<bf16>, #blocked3>, tensor<32x32xi32, #blocked3>
3753|     %74 = arith.truncf %67 : tensor<32x32xf32, #blocked2> to tensor<32x32xbf16, #blocked2>
3754|     %75 = ttg.convert_layout %74 : tensor<32x32xbf16, #blocked2> -> tensor<32x32xbf16, #blocked3>
3755|     tt.store %73, %75 : tensor<32x32x!tt.ptr<bf16>, #blocked3>
3756|     tt.return
3757|   }
3758| }
3759| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, pointer arithmetic, tensor broadcasting, dot-product or MMA-style math, integer multiplications. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、指针算术、张量广播、点积或 MMA 风格计算、整数乘法。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 3760-3760
```mlir
3760| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 3761-3764
```mlir
3761| 
3762| // Check that we can hoist ttg.convert_layout ops that eventually feed into dot
3763| // for decomposed mxfp emulation for AMD GPUs.
3764| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 3765-3775
```mlir
3765| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [4, 16], warpsPerCTA = [1, 4], order = [0, 1]}>
3766| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 64], warpsPerCTA = [2, 2], order = [1, 0]}>
3767| #blocked2 = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [32, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
3768| #blocked3 = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [8, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
3769| #blocked4 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [2, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
3770| #blocked5 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [16, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
3771| #blocked6 = #ttg.blocked<{sizePerThread = [1, 1, 1], threadsPerWarp = [1, 64, 1], warpsPerCTA = [2, 2, 1], order = [2, 1, 0]}>
3772| #blocked7 = #ttg.blocked<{sizePerThread = [1, 1, 1], threadsPerWarp = [1, 1, 64], warpsPerCTA = [2, 1, 2], order = [1, 2, 0]}>
3773| #linear = #ttg.linear<{register = [[1, 0], [4, 0], [8, 0], [16, 0], [32, 0], [64, 0], [128, 0]], lane = [[0, 1], [0, 2], [0, 4], [0, 8], [0, 16], [0, 32]], warp = [[0, 64], [2, 0]], block = []}>
3774| #linear1 = #ttg.linear<{register = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0], [64, 0]], lane = [[0, 1], [0, 2], [0, 4], [0, 8], [0, 16], [0, 32]], warp = [[0, 64], [32, 0]], block = []}>
3775| #mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [2, 2], instrShape = [32, 32, 8], isTransposed = true}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 3776-3776
```mlir
3776| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 3777-3777
```mlir
3777|   // CHECK-LABEL: @fp8_mxfp4_matmul_decompose
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @fp8_mxfp4_matmul_decompose anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @fp8_mxfp4_matmul_decompose 这样的标签用于锚定匹配范围。

### Lines 3778-3793
```mlir
3778|   tt.func public @fp8_mxfp4_matmul_decompose(%59: i32, %71: tensor<128x128x!tt.ptr<f32>, #blocked4>, %47: tensor<128x128x!tt.ptr<f8E5M2>, #blocked3>, %57: tensor<64x128x!tt.ptr<i8>, #blocked3>, %37: tensor<128x4x!tt.ptr<i8>, #blocked2>, %61: tensor<64x128xi32, #blocked3>) {
3779|     %c0_i32 = arith.constant 0 : i32
3780|     %c1_i32 = arith.constant 1 : i32
3781|     %cst = arith.constant dense<0x7FC0> : tensor<128x128xbf16, #linear>
3782|     %cst_0 = arith.constant dense<-1> : tensor<4x128xi8, #blocked>
3783|     %cst_1 = arith.constant dense<7> : tensor<4x128xi16, #blocked>
3784|     %cst_2 = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked1>
3785|     %cst_3 = arith.constant dense<4> : tensor<128x4xi32, #blocked2>
3786|     %cst_4 = arith.constant dense<128> : tensor<128x128xi32, #blocked3>
3787|     //     CHECK: scf.for
3788|     //     CHECK:   tt.load
3789|     //     CHECK:   ttg.convert_layout
3790|     //     CHECK:   tt.load
3791|     //     CHECK:   ttg.convert_layout
3792|     //     CHECK:   tt.load
3793|     //     CHECK:   ttg.convert_layout
```
**EN:** This function-oriented block defines or enters `fp8_mxfp4_matmul_decompose`. Within it, the test exercises constants, tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fp8_mxfp4_matmul_decompose` 为核心。测试在其中演示 常量、tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 3794-3811
```mlir
3794|     // CHECK-NOT:   ttg.convert_layout
3795|     //     CHECK:   scf.yield
3796|     %62:4 = scf.for %arg11 = %c0_i32 to %59 step %c1_i32 iter_args(%arg12 = %cst_2, %arg13 = %47, %arg14 = %57, %arg15 = %37) -> (tensor<128x128xf32, #blocked1>, tensor<128x128x!tt.ptr<f8E5M2>, #blocked3>, tensor<64x128x!tt.ptr<i8>, #blocked3>, tensor<128x4x!tt.ptr<i8>, #blocked2>)  : i32 {
3797|       %80 = tt.load %arg13 : tensor<128x128x!tt.ptr<f8E5M2>, #blocked3>
3798|       %81 = ttg.convert_layout %80 : tensor<128x128xf8E5M2, #blocked3> -> tensor<128x128xf8E5M2, #blocked1>
3799|       %82 = tt.load %arg14 : tensor<64x128x!tt.ptr<i8>, #blocked3>
3800|       %83 = ttg.convert_layout %82 : tensor<64x128xi8, #blocked3> -> tensor<64x128xi8, #blocked1>
3801|       %84 = tt.load %arg15 : tensor<128x4x!tt.ptr<i8>, #blocked2>
3802|       %85 = ttg.convert_layout %84 : tensor<128x4xi8, #blocked2> -> tensor<128x4xi8, #blocked5>
3803|       %86 = tt.fp_to_fp %81 : tensor<128x128xf8E5M2, #blocked1> -> tensor<128x128xbf16, #blocked1>
3804|       %87 = ttg.convert_layout %86 : tensor<128x128xbf16, #blocked1> -> tensor<128x128xbf16, #ttg.dot_op<{opIdx = 0, parent = #blocked1}>>
3805|       %88 = ttg.fp4_to_fp %83 {axis = 0 : i32} : tensor<64x128xi8, #blocked1> -> tensor<128x128xbf16, #linear>
3806|       %89 = tt.trans %85 {order = array<i32: 1, 0>} : tensor<128x4xi8, #blocked5> -> tensor<4x128xi8, #blocked>
3807|       %90 = arith.extui %89 : tensor<4x128xi8, #blocked> to tensor<4x128xi16, #blocked>
3808|       %91 = arith.shli %90, %cst_1 : tensor<4x128xi16, #blocked>
3809|       %92 = tt.bitcast %91 : tensor<4x128xi16, #blocked> -> tensor<4x128xbf16, #blocked>
3810|       %93 = ttg.convert_layout %92 : tensor<4x128xbf16, #blocked> -> tensor<4x128xbf16, #ttg.slice<{dim = 2, parent = #blocked6}>>
3811|       %94 = tt.expand_dims %93 {axis = 2 : i32} : tensor<4x128xbf16, #ttg.slice<{dim = 2, parent = #blocked6}>> -> tensor<4x128x1xbf16, #blocked6>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining layout conversions, masked or vectorized loads, structured loops, tt.fp_to_fp, ttg.fp4_to_fp.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 布局转换、带掩码或向量化的加载、结构化循环、tt.fp_to_fp、ttg.fp4_to_fp。

### Lines 3812-3829
```mlir
3812|       %95 = tt.broadcast %94 : tensor<4x128x1xbf16, #blocked6> -> tensor<4x128x32xbf16, #blocked6>
3813|       %96 = tt.trans %95 {order = array<i32: 0, 2, 1>} : tensor<4x128x32xbf16, #blocked6> -> tensor<4x32x128xbf16, #blocked7>
3814|       %97 = tt.reshape %96 : tensor<4x32x128xbf16, #blocked7> -> tensor<128x128xbf16, #linear1>
3815|       %98 = ttg.convert_layout %97 : tensor<128x128xbf16, #linear1> -> tensor<128x128xbf16, #linear>
3816|       %99 = arith.mulf %88, %98 : tensor<128x128xbf16, #linear>
3817|       %100 = arith.cmpi eq, %89, %cst_0 : tensor<4x128xi8, #blocked>
3818|       %101 = ttg.convert_layout %100 : tensor<4x128xi1, #blocked> -> tensor<4x128xi1, #ttg.slice<{dim = 2, parent = #blocked6}>>
3819|       %102 = tt.expand_dims %101 {axis = 2 : i32} : tensor<4x128xi1, #ttg.slice<{dim = 2, parent = #blocked6}>> -> tensor<4x128x1xi1, #blocked6>
3820|       %103 = tt.broadcast %102 : tensor<4x128x1xi1, #blocked6> -> tensor<4x128x32xi1, #blocked6>
3821|       %104 = tt.trans %103 {order = array<i32: 0, 2, 1>} : tensor<4x128x32xi1, #blocked6> -> tensor<4x32x128xi1, #blocked7>
3822|       %105 = tt.reshape %104 : tensor<4x32x128xi1, #blocked7> -> tensor<128x128xi1, #linear1>
3823|       %106 = ttg.convert_layout %105 : tensor<128x128xi1, #linear1> -> tensor<128x128xi1, #linear>
3824|       %107 = arith.select %106, %cst, %99 : tensor<128x128xi1, #linear>, tensor<128x128xbf16, #linear>
3825|       %108 = ttg.convert_layout %107 : tensor<128x128xbf16, #linear> -> tensor<128x128xbf16, #ttg.dot_op<{opIdx = 1, parent = #blocked1}>>
3826|       %109 = ttg.convert_layout %arg12 : tensor<128x128xf32, #blocked1> -> tensor<128x128xf32, #mma>
3827|       %110 = ttg.convert_layout %87 : tensor<128x128xbf16, #ttg.dot_op<{opIdx = 0, parent = #blocked1}>> -> tensor<128x128xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>
3828|       %111 = ttg.convert_layout %108 : tensor<128x128xbf16, #ttg.dot_op<{opIdx = 1, parent = #blocked1}>> -> tensor<128x128xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
3829|       %112 = tt.dot %110, %111, %109 : tensor<128x128xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>> * tensor<128x128xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>> -> tensor<128x128xf32, #mma>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, tensor broadcasting, transpose-like layout changes, tensor reshaping, arith.mulf. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、张量广播、转置类布局变换、张量重塑、arith.mulf。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 3830-3842
```mlir
3830|       %113 = ttg.convert_layout %112 : tensor<128x128xf32, #mma> -> tensor<128x128xf32, #blocked1>
3831|       %114 = ttg.convert_layout %113 : tensor<128x128xf32, #blocked1> -> tensor<128x128xf32, #blocked1>
3832|       %115 = tt.addptr %arg13, %cst_4 : tensor<128x128x!tt.ptr<f8E5M2>, #blocked3>, tensor<128x128xi32, #blocked3>
3833|       %116 = tt.addptr %arg14, %61 : tensor<64x128x!tt.ptr<i8>, #blocked3>, tensor<64x128xi32, #blocked3>
3834|       %117 = tt.addptr %arg15, %cst_3 : tensor<128x4x!tt.ptr<i8>, #blocked2>, tensor<128x4xi32, #blocked2>
3835|       scf.yield %114, %115, %116, %117 : tensor<128x128xf32, #blocked1>, tensor<128x128x!tt.ptr<f8E5M2>, #blocked3>, tensor<64x128x!tt.ptr<i8>, #blocked3>, tensor<128x4x!tt.ptr<i8>, #blocked2>
3836|     } {tt.num_stages = 2 : i32}
3837|     %79 = ttg.convert_layout %62#0 : tensor<128x128xf32, #blocked1> -> tensor<128x128xf32, #blocked4>
3838|     tt.store %71, %79 : tensor<128x128x!tt.ptr<f32>, #blocked4>
3839|     tt.return
3840|   }
3841| }
3842| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining layout conversions, pointer arithmetic, loop/if yielded values, masked or vectorized stores, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 布局转换、指针算术、循环/分支产出值、带掩码或向量化的存储、tt.return。

### Lines 3843-3843
```mlir
3843| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 3844-3849
```mlir
3844| 
3845| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
3846| #blocked2 = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
3847| #blocked3 = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [16, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
3848| #linear = #ttg.linear<{register = [[0, 0, 1], [0, 0, 2], [0, 0, 4], [8, 0, 0], [0, 1, 0], [0, 2, 0]], lane = [[0, 0, 8], [0, 0, 16], [1, 0, 0], [2, 0, 0], [4, 0, 0]], warp = [[0, 0, 0], [16, 0, 0]], block = []}>
3849| #mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [2, 2], instrShape = [16, 8]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 3850-3852
```mlir
3850| module attributes {"ttg.num-warps" = 4 : i32, "ttg.target" = "cuda:80"} {
3851|   // Check that the remove-layout-conversions pass is idempotent
3852|   // in that it keeps the convert_layout ops next to the loads
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 3853-3853
```mlir
3853|   // CHECK: tt.func @remove_layout_is_idempotent
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 3854-3858
```mlir
3854|   tt.func @remove_layout_is_idempotent(%14: tensor<32x64x!tt.ptr<i8>, #blocked2>, %39: tensor<32x4x!tt.ptr<i8>, #blocked>, %27: tensor<128x32x!tt.ptr<i8>, #blocked3>) -> tensor<32x32xf32, #mma> {
3855|     %cst = arith.constant dense<0x7FC0> : tensor<32x128xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>
3856|     %cst_0 = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #mma>
3857|     %cst_3 = arith.constant dense<7> : tensor<32x4xi16, #ttg.slice<{dim = 2, parent = #linear}>>
3858|     %cst_4 = arith.constant dense<-1> : tensor<32x4xi8, #ttg.slice<{dim = 2, parent = #linear}>>
```
**EN:** This function-oriented block defines or enters `remove_layout_is_idempotent`. Within it, the test exercises constants, tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `remove_layout_is_idempotent` 为核心。测试在其中演示 常量、tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 3859-3876
```mlir
3859|     // CHECK: %[[LOAD1:.*]] = tt.load
3860|     // CHECK: ttg.convert_layout %[[LOAD1]]
3861|     // CHECK: %[[LOAD2:.*]] = tt.load
3862|     // CHECK: ttg.convert_layout %[[LOAD2]]
3863|     // CHECK: %[[LOAD3:.*]] = tt.load
3864|     // CHECK: ttg.convert_layout %[[LOAD3]]
3865|     %28 = tt.load %14 : tensor<32x64x!tt.ptr<i8>, #blocked2>
3866|     %29 = ttg.convert_layout %28 : tensor<32x64xi8, #blocked2> -> tensor<32x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>
3867|     %30 = tt.load %27 : tensor<128x32x!tt.ptr<i8>, #blocked3>
3868|     %31 = ttg.convert_layout %30 : tensor<128x32xi8, #blocked3> -> tensor<128x32xi8, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>
3869|     %40 = tt.load %39 : tensor<32x4x!tt.ptr<i8>, #blocked>
3870|     %41 = ttg.convert_layout %40 : tensor<32x4xi8, #blocked> -> tensor<32x4xi8, #ttg.slice<{dim = 2, parent = #linear}>>
3871|     %42 = tt.bitcast %31 : tensor<128x32xi8, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>> -> tensor<128x32xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>
3872|     %43 = ttg.fp4_to_fp %29 {axis = 1 : i32} : tensor<32x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>> -> tensor<32x128xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>
3873|     %44 = arith.extui %41 : tensor<32x4xi8, #ttg.slice<{dim = 2, parent = #linear}>> to tensor<32x4xi16, #ttg.slice<{dim = 2, parent = #linear}>>
3874|     %45 = arith.shli %44, %cst_3 : tensor<32x4xi16, #ttg.slice<{dim = 2, parent = #linear}>>
3875|     %46 = tt.bitcast %45 : tensor<32x4xi16, #ttg.slice<{dim = 2, parent = #linear}>> -> tensor<32x4xbf16, #ttg.slice<{dim = 2, parent = #linear}>>
3876|     %47 = tt.expand_dims %46 {axis = 2 : i32} : tensor<32x4xbf16, #ttg.slice<{dim = 2, parent = #linear}>> -> tensor<32x4x1xbf16, #linear>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized loads, layout conversions, tt.bitcast, ttg.fp4_to_fp, arith.extui. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的加载、布局转换、tt.bitcast、ttg.fp4_to_fp、arith.extui。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 3877-3890
```mlir
3877|     %48 = tt.broadcast %47 : tensor<32x4x1xbf16, #linear> -> tensor<32x4x32xbf16, #linear>
3878|     %49 = tt.reshape %48 : tensor<32x4x32xbf16, #linear> -> tensor<32x128xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>
3879|     %50 = arith.mulf %43, %49 : tensor<32x128xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>
3880|     %51 = arith.cmpi eq, %41, %cst_4 : tensor<32x4xi8, #ttg.slice<{dim = 2, parent = #linear}>>
3881|     %52 = tt.expand_dims %51 {axis = 2 : i32} : tensor<32x4xi1, #ttg.slice<{dim = 2, parent = #linear}>> -> tensor<32x4x1xi1, #linear>
3882|     %53 = tt.broadcast %52 : tensor<32x4x1xi1, #linear> -> tensor<32x4x32xi1, #linear>
3883|     %54 = tt.reshape %53 : tensor<32x4x32xi1, #linear> -> tensor<32x128xi1, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>
3884|     %55 = arith.select %54, %cst, %50 : tensor<32x128xi1, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>, tensor<32x128xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>
3885|     %56 = tt.fp_to_fp %42 : tensor<128x32xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>> -> tensor<128x32xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>
3886|     %57 = tt.dot %55, %56, %cst_0 : tensor<32x128xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>> * tensor<128x32xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>> -> tensor<32x32xf32, #mma>
3887|     tt.return %57 : tensor<32x32xf32, #mma>
3888|   }
3889| }
3890| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor broadcasting, tensor reshaping, arith.mulf, integer comparisons, shape expansion. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量广播、张量重塑、arith.mulf、整数比较、形状扩展。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 3891-3891
```mlir
3891| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 3892-3898
```mlir
3892| 
3893| #blocked = #ttg.blocked<{sizePerThread = [4, 4], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
3894| #blocked2 = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [16, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
3895| #blocked4 = #ttg.blocked<{sizePerThread = [1, 16, 2], threadsPerWarp = [16, 2, 1], warpsPerCTA = [4, 1, 1], order = [2, 1, 0]}>
3896| #blocked5 = #ttg.blocked<{sizePerThread = [1, 32], threadsPerWarp = [16, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
3897| #blocked6 = #ttg.blocked<{sizePerThread = [32, 1], threadsPerWarp = [2, 16], warpsPerCTA = [1, 4], order = [0, 1]}>
3898| #mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [1, 4], instrShape = [16, 8]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 3899-3899
```mlir
3899| module attributes {"ttg.num-warps" = 4 : i32, "ttg.target" = "cuda:80"} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 3900-3901
```mlir
3900|   tt.func @join_reshape_dot(%112: tensor<128x32x!tt.ptr<i8>, #blocked2>, %117: tensor<128x32xi1, #blocked2>, %128: tensor<16x64xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>) -> tensor<16x128xf32, #mma> {
3901|       %cst = arith.constant dense<0.000000e+00> : tensor<16x128xf32, #blocked>
```
**EN:** This function-oriented block defines or enters `join_reshape_dot`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `join_reshape_dot` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 3902-3916
```mlir
3902|       // CHECK: %[[LOAD_I8:.*]] = tt.load {{.*}} tensor<128x32x!tt.ptr<i8>
3903|       // CHECK: ttg.convert_layout %[[LOAD_I8]] {{.*}} #linear
3904|       %118 = tt.load %112, %117 : tensor<128x32x!tt.ptr<i8>, #blocked2>
3905|       %121:2 = tt.elementwise_inline_asm "" {constraints = "=r,=r,=r,=r,r", packed_element = 4 : i32, pure = true} %118 : tensor<128x32xi8, #blocked2> -> tensor<128x32xbf16, #blocked2>, tensor<128x32xbf16, #blocked2>
3906|       %122 = tt.join %121#0, %121#1 : tensor<128x32xbf16, #blocked2> -> tensor<128x32x2xbf16, #blocked4>
3907|       %123 = tt.reshape %122 : tensor<128x32x2xbf16, #blocked4> -> tensor<128x64xbf16, #blocked5>
3908|       %124 = tt.trans %123 {order = array<i32: 1, 0>} : tensor<128x64xbf16, #blocked5> -> tensor<64x128xbf16, #blocked6>
3909|       %126 = ttg.convert_layout %124 : tensor<64x128xbf16, #blocked6> -> tensor<64x128xbf16, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>
3910|       %127 = ttg.convert_layout %cst : tensor<16x128xf32, #blocked> -> tensor<16x128xf32, #mma>
3911|       %129 = ttg.convert_layout %126 : tensor<64x128xbf16, #ttg.dot_op<{opIdx = 1, parent = #blocked}>> -> tensor<64x128xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>
3912|       %130 = tt.dot %128, %129, %127, inputPrecision = tf32 : tensor<16x64xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<64x128xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<16x128xf32, #mma>
3913|       tt.return %130 : tensor<16x128xf32, #mma>
3914|   }
3915| }
3916| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, masked or vectorized loads, tt.elementwise_inline_asm, tt.join, tensor reshaping. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、带掩码或向量化的加载、tt.elementwise_inline_asm、tt.join、张量重塑。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 3917-3917
```mlir
3917| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 3918-3921
```mlir
3918| 
3919| // CHECK-DAG: [[BLOCKED_OUT:#.*]] = #ttg.blocked<{sizePerThread = [1, 1, 2]
3920| // CHECK-DAG: [[BLOCKED_JOIN:#.*]] = #ttg.blocked<{sizePerThread = [1, 2, 2]
3921| // CHECK-DAG: [[BLOCKED_IN:#.*]] = #ttg.blocked<{sizePerThread = [1, 2]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 3922-3924
```mlir
3922| #blocked = #ttg.blocked<{sizePerThread = [1, 1, 2], threadsPerWarp = [2, 16, 1], warpsPerCTA = [1, 1, 1], order = [2, 1, 0]}>
3923| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [2, 16], warpsPerCTA = [1, 1], order = [1, 0]}>
3924| #blocked2 = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [2, 16], warpsPerCTA = [1, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 3925-3925
```mlir
3925| module attributes {"ttg.num-warps" = 1 : i32, ttg.target = "cuda:80"} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:80`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:80`。

### Lines 3926-3926
```mlir
3926|   tt.func @join_forward(%arg0: tensor<2x16xf32, #blocked2>) -> tensor<2x16x2xf32, #blocked> {
```
**EN:** This function-oriented block defines or enters `join_forward`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `join_forward` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 3927-3935
```mlir
3927|     // CHECK: [[JOIN:%.*]] = tt.join %arg0, %arg0 : tensor<2x16xf32, [[BLOCKED_IN]]> -> tensor<2x16x2xf32, [[BLOCKED_JOIN]]>
3928|     // CHECK: [[RES:%.*]] = ttg.convert_layout [[JOIN]] : tensor<2x16x2xf32, [[BLOCKED_JOIN]]> -> tensor<2x16x2xf32, [[BLOCKED_OUT]]
3929|     // CHECK: tt.return [[RES]]
3930|     %0 = ttg.convert_layout %arg0 : tensor<2x16xf32, #blocked2> -> tensor<2x16xf32, #blocked1>
3931|     %1 = tt.join %0, %0 : tensor<2x16xf32, #blocked1> -> tensor<2x16x2xf32, #blocked>
3932|     tt.return %1 : tensor<2x16x2xf32, #blocked>
3933|   }
3934| }
3935| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, tt.join, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、tt.join、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 3936-3936
```mlir
3936| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 3937-3940
```mlir
3937| 
3938| #blocked = #ttg.blocked<{sizePerThread = [1, 32], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
3939| #blocked1 = #ttg.blocked<{sizePerThread = [1, 32, 2], threadsPerWarp = [32, 1, 1], warpsPerCTA = [4, 1, 1], order = [0, 1, 2]}>
3940| #blocked2 = #ttg.blocked<{sizePerThread = [1, 32, 2], threadsPerWarp = [32, 1, 1], warpsPerCTA = [4, 1, 1], order = [2, 0, 1]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 3941-3941
```mlir
3941| module attributes {"ttg.num-warps" = 4 : i32, ttg.target = "cuda:80"} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:80`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:80`。

### Lines 3942-3942
```mlir
3942|   // CHECK-LABEL: join_backward_blocked
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: join_backward_blocked anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: join_backward_blocked 这样的标签用于锚定匹配范围。

### Lines 3943-3943
```mlir
3943|   tt.func @join_backward_blocked(%arg0: tensor<128x32xf16, #blocked>, %arg1: tensor<128x32xf16, #blocked>) -> tensor<128x32x2xf16, #blocked1> {
```
**EN:** This function-oriented block defines or enters `join_backward_blocked`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `join_backward_blocked` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 3944-3951
```mlir
3944|     // CHECK: %[[JOIN:.*]] = tt.join %arg0, %arg1
3945|     // CHECK: tt.return %[[JOIN]]
3946|     %0 = tt.join %arg0, %arg1 : tensor<128x32xf16, #blocked> -> tensor<128x32x2xf16, #blocked2>
3947|     %1 = ttg.convert_layout %0 : tensor<128x32x2xf16, #blocked2> -> tensor<128x32x2xf16, #blocked1>
3948|     tt.return %1 : tensor<128x32x2xf16, #blocked1>
3949|   }
3950| }
3951| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.join, layout conversions, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.join、布局转换、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 3952-3952
```mlir
3952| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 3953-3955
```mlir
3953| 
3954| #blocked1 = #ttg.blocked<{sizePerThread = [1, 32, 2], threadsPerWarp = [32, 1, 1], warpsPerCTA = [4, 1, 1], order = [0, 1, 2]}>
3955| #blocked2 = #ttg.blocked<{sizePerThread = [1, 32, 2], threadsPerWarp = [32, 1, 1], warpsPerCTA = [4, 1, 1], order = [2, 0, 1]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 3956-3956
```mlir
3956| module attributes {"ttg.num-warps" = 4 : i32, ttg.target = "cuda:80"} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:80`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:80`。

### Lines 3957-3957
```mlir
3957|   // CHECK-LABEL: join_backward_slice
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: join_backward_slice anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: join_backward_slice 这样的标签用于锚定匹配范围。

### Lines 3958-3958
```mlir
3958|   tt.func @join_backward_slice(%arg0: tensor<128x32xf16, #ttg.slice<{dim=2, parent=#blocked1}>>, %arg1: tensor<128x32xf16, #ttg.slice<{dim=2, parent=#blocked1}>>) -> tensor<128x32x2xf16, #blocked1> {
```
**EN:** This function-oriented block defines or enters `join_backward_slice`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `join_backward_slice` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 3959-3966
```mlir
3959|     // CHECK: %[[JOIN:.*]] = tt.join
3960|     // CHECK: tt.return %[[JOIN]]
3961|     %0 = tt.join %arg0, %arg1 : tensor<128x32xf16, #ttg.slice<{dim=2, parent=#blocked1}>> -> tensor<128x32x2xf16, #blocked2>
3962|     %1 = ttg.convert_layout %0 : tensor<128x32x2xf16, #blocked2> -> tensor<128x32x2xf16, #blocked1>
3963|     tt.return %1 : tensor<128x32x2xf16, #blocked1>
3964|   }
3965| }
3966| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.join, layout conversions, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.join、布局转换、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 3967-3967
```mlir
3967| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 3968-3975
```mlir
3968| 
3969| #linear = #ttg.linear<{register = [[0, 2], [64, 0]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0], [0, 1]], warp = [[0, 0], [32, 0]], block = []}>
3970| #linear1 = #ttg.linear<{register = [[0, 2], [64, 0]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0], [0, 1]], warp = [[32, 0], [0, 0]], block = []}>
3971| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 64], warpsPerCTA = [2, 2], order = [1, 0]}>
3972| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 64], warpsPerCTA = [1, 4], order = [1, 0]}>
3973| #mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [2, 2], instrShape = [32, 32, 64], isTransposed = true}>
3974| #dot_op_a = #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 16}>
3975| #dot_op_b = #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 3976-3977
```mlir
3976| // CHECK: [[$BLOCK:.+]] = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 64], warpsPerCTA = [2, 2], order = [1, 0]}>
3977| // CHECK-LABEL: mfma_dot_scaled_no_redundant_convert_layout
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: mfma_dot_scaled_no_redundant_convert_layout anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: mfma_dot_scaled_no_redundant_convert_layout 这样的标签用于锚定匹配范围。

### Lines 3978-3978
```mlir
3978| module attributes {"ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx950`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx950`。

### Lines 3979-3990
```mlir
3979|   tt.func public @mfma_dot_scaled_no_redundant_convert_layout(
3980|         %arg0: tensor<128x128xf8E4M3FN, #dot_op_a>,
3981|         %arg1: tensor<128x128xf8E4M3FN, #dot_op_b>,
3982|         %arg2: tensor<128x4xi8, #linear>,
3983|         %arg3: tensor<128x4xi8, #linear1>,
3984|         %arg4: tensor<128x128x!tt.ptr<f32>, #blocked>
3985|       ) {
3986|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #mma>
3987|     %cst0 = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked1>
3988|     %c0 = arith.constant 0 : index
3989|     %c1 = arith.constant 1 : index
3990|     %c32 = arith.constant 32 : index
```
**EN:** This function-oriented block defines or enters `mfma_dot_scaled_no_redundant_convert_layout`. Within it, the test exercises constants, tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `mfma_dot_scaled_no_redundant_convert_layout` 为核心。测试在其中演示 常量、tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 3991-4007
```mlir
3991|     // CHECK: %[[RET:.+]] = scf.for
3992|     // CHECK-NEXT: %[[DOT_RET:.+]] = tt.dot_scaled %arg0 scale %arg2, %arg1 scale %arg3, %cst lhs = e4m3 rhs = e4m3 {fastMath = false}
3993|     // CHECK-NEXT: scf.yield %[[DOT_RET]]
3994|     // CHECK-NEXT: }
3995|     // CHECK-NEXT: ttg.convert_layout %[[RET]] : tensor<128x128xf32, #mma> -> tensor<128x128xf32, [[$BLOCK]]>
3996|     // CHECK-NEXT: tt.store
3997|     %1 = scf.for %arg5 = %c0 to %c32 step %c1 iter_args(%arg6 = %cst0) -> (tensor<128x128xf32, #blocked1>) {
3998|       %4 = tt.dot_scaled %arg0 scale %arg2, %arg1 scale %arg3, %cst lhs = e4m3 rhs = e4m3 {fastMath = false} : tensor<128x128xf8E4M3FN, #dot_op_a>, tensor<128x4xi8, #linear> * tensor<128x128xf8E4M3FN, #dot_op_b>, tensor<128x4xi8, #linear1> -> tensor<128x128xf32, #mma>
3999|       %5 = ttg.convert_layout %4 : tensor<128x128xf32, #mma> -> tensor<128x128xf32, #blocked1>
4000|       scf.yield %5 : tensor<128x128xf32, #blocked1>
4001|     }
4002|     %7 = ttg.convert_layout %1 : tensor<128x128xf32, #blocked1> -> tensor<128x128xf32, #blocked>
4003|     tt.store %arg4, %7 : tensor<128x128x!tt.ptr<f32>, #blocked>
4004|     tt.return
4005|   }
4006| }
4007| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining layout conversions, structured loops, tt.dot_scaled, loop/if yielded values, masked or vectorized stores.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 布局转换、结构化循环、tt.dot_scaled、循环/分支产出值、带掩码或向量化的存储。

### Lines 4008-4008
```mlir
4008| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 4009-4013
```mlir
4009| 
4010| // Test that when we attempt to hoist layout conversions into one branch of an
4011| // if/else, we validate that the layouts required by different conditionals or
4012| // different branches do not conflict.
4013| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 4014-4015
```mlir
4014| #blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
4015| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 4016-4016
```mlir
4016| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 4017-4034
```mlir
4017|   tt.func public @hoist_into_cond_layout_conflict(%arg0: !tt.ptr<i32> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<i32> {tt.divisibility = 16 : i32}, %arg2: i1) -> tensor<4x1xi64, #blocked> {
4018|     %c1_i32 = arith.constant 1 : i32
4019|     %c4_i32 = arith.constant 4 : i32
4020|     %c0_i32 = arith.constant 0 : i32
4021|     %0 = tt.make_range {end = 4 : i32, start = 0 : i32} : tensor<4xi32, #ttg.slice<{dim = 1, parent = #blocked1}>>
4022|     %1 = arith.extsi %0 : tensor<4xi32, #ttg.slice<{dim = 1, parent = #blocked1}>> to tensor<4xi64, #ttg.slice<{dim = 1, parent = #blocked1}>>
4023|     %2 = tt.splat %arg0 : !tt.ptr<i32> -> tensor<4x!tt.ptr<i32>, #ttg.slice<{dim = 1, parent = #blocked1}>>
4024|     %3 = tt.expand_dims %1 {axis = 1 : i32} : tensor<4xi64, #ttg.slice<{dim = 1, parent = #blocked1}>> -> tensor<4x1xi64, #blocked1>
4025|     %4 = tt.addptr %2, %1 : tensor<4x!tt.ptr<i32>, #ttg.slice<{dim = 1, parent = #blocked1}>>, tensor<4xi64, #ttg.slice<{dim = 1, parent = #blocked1}>>
4026|     %5 = tt.load %4 : tensor<4x!tt.ptr<i32>, #ttg.slice<{dim = 1, parent = #blocked1}>>
4027|     %6 = tt.reshape %5 : tensor<4xi32, #ttg.slice<{dim = 1, parent = #blocked1}>> -> tensor<4x1xi32, #blocked1>
4028|     %7 = arith.extsi %6 : tensor<4x1xi32, #blocked1> to tensor<4x1xi64, #blocked1>
4029|     %cst = arith.constant dense<0> : tensor<4x1xi64, #blocked>
4030|     %8 = scf.if %arg2 -> (tensor<4x1xi64, #blocked1>) {
4031|       // The backward slice from this extsi will produce a non-sliced layout for
4032|       // %1.
4033|       scf.yield %7 : tensor<4x1xi64, #blocked1>
4034|     } else {
```
**EN:** This function-oriented block defines or enters `hoist_into_cond_layout_conflict`. Within it, the test exercises constants, tt.func, arith.extsi, lane/block index ranges, broadcasted scalars or pointers, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `hoist_into_cond_layout_conflict` 为核心。测试在其中演示 常量、tt.func、arith.extsi、lane/block 索引范围、广播后的标量或指针，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 4035-4037
```mlir
4035|       // The backward slice from this add will produce a sliced layout for %1.
4036|       scf.yield %3 : tensor<4x1xi64, #blocked1>
4037|     }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值。

### Lines 4038-4055
```mlir
4038|     // CHECK: scf.for
4039|     // CHECK-NEXT: scf.if
4040|     // CHECK-NOT: ttg.convert_layout
4041|     // CHECK: } else {
4042|     // CHECK: ttg.convert_layout
4043|     // CHECK-NOT: ttg.convert-layout
4044|     %9 = scf.for %arg3 = %c0_i32 to %c4_i32 step %c1_i32 iter_args(%arg4 = %cst) -> (tensor<4x1xi64, #blocked>)  : i32 {
4045|       %10 = scf.if %arg2 -> (tensor<4x1xi64, #blocked1>) {
4046|         // The backward slice from this extsi will produce a non-sliced layout
4047|         // for %1 when it is rematerialized conflicting with the sliced layout
4048|         // produced by %3 in the else arm of the other if.
4049|         %14 = arith.extsi %6 : tensor<4x1xi32, #blocked1> to tensor<4x1xi64, #blocked1>
4050|         scf.yield %14 : tensor<4x1xi64, #blocked1>
4051|       } else {
4052|         // The backward slice from this add will produce conflicting layouts for
4053|         // %1, so we try to hoist the convert into this arm.
4054|         %14 = arith.addi %7, %3 : tensor<4x1xi64, #blocked1>
4055|         scf.yield %14 : tensor<4x1xi64, #blocked1>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, structured loops, structured conditionals, arith.extsi, integer additions.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、结构化循环、结构化条件分支、arith.extsi、整数加法。

### Lines 4056-4065
```mlir
4056|       }
4057|       %11 = arith.addi %8, %10 : tensor<4x1xi64, #blocked1>
4058|       %12 = ttg.convert_layout %11 : tensor<4x1xi64, #blocked1> -> tensor<4x1xi64, #blocked>
4059|       %13 = arith.addi %arg4, %12 : tensor<4x1xi64, #blocked>
4060|       scf.yield %13 : tensor<4x1xi64, #blocked>
4061|     }
4062|     tt.return %9 : tensor<4x1xi64, #blocked>
4063|   }
4064| }
4065| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining integer additions, layout conversions, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 整数加法、布局转换、循环/分支产出值、tt.return。

### Lines 4066-4066
```mlir
4066| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 4067-4070
```mlir
4067| 
4068| // Test that we do not hoist a convert if the resulting convert would  be more
4069| // expensive than the original.
4070| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 4071-4072
```mlir
4071| #blocked = #ttg.blocked<{sizePerThread = [1, 1, 1], threadsPerWarp = [1, 8, 4], warpsPerCTA = [1, 4, 1], order = [2, 1, 0]}>
4072| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1, 1], threadsPerWarp = [1, 4, 8], warpsPerCTA = [1, 1, 4], order = [2, 1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 4073-4073
```mlir
4073| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 4074-4074
```mlir
4074|   tt.func public @hoist_convert_unprofitable(%arg0: tensor<1x32x32xf32, #blocked>) -> tensor<32xf32, #ttg.slice<{dim = 0, parent = #ttg.slice<{dim = 0, parent = #blocked1}>}>> {
```
**EN:** This function-oriented block defines or enters `hoist_convert_unprofitable`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `hoist_convert_unprofitable` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 4075-4092
```mlir
4075|     // CHECK: tt.broadcast
4076|     // CHECK: tt.reduce
4077|     // CHECK: tt.reduce
4078|     // CHECK: ttg.convert_layout
4079|     %0 = tt.broadcast %arg0 : tensor<1x32x32xf32, #blocked> -> tensor<32x32x32xf32, #blocked>
4080|     %1 = "tt.reduce"(%0) <{axis = 0 : i32}> ({
4081|     ^bb0(%arg1: f32, %arg2: f32):
4082|       %4 = arith.addf %arg1, %arg2 : f32
4083|       tt.reduce.return %4 : f32
4084|     }) : (tensor<32x32x32xf32, #blocked>) -> tensor<32x32xf32, #ttg.slice<{dim = 0, parent = #blocked}>>
4085|     %2 = "tt.reduce"(%1) <{axis = 0 : i32}> ({
4086|     ^bb0(%arg1: f32, %arg2: f32):
4087|       %4 = arith.addf %arg1, %arg2 : f32
4088|       tt.reduce.return %4 : f32
4089|     }) : (tensor<32x32xf32, #ttg.slice<{dim = 0, parent = #blocked}>>) -> tensor<32xf32, #ttg.slice<{dim = 0, parent = #ttg.slice<{dim = 0, parent = #blocked}>}>>
4090|     %3 = ttg.convert_layout %2 : tensor<32xf32, #ttg.slice<{dim = 0, parent = #ttg.slice<{dim = 0, parent = #blocked}>}>> -> tensor<32xf32, #ttg.slice<{dim = 0, parent = #ttg.slice<{dim = 0, parent = #blocked1}>}>>
4091|     tt.return %3 : tensor<32xf32, #ttg.slice<{dim = 0, parent = #ttg.slice<{dim = 0, parent = #blocked1}>}>>
4092|   }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on floating-point additions, reductions, tensor broadcasting, layout conversions, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 浮点加法、归约、张量广播、布局转换、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 4093-4094
```mlir
4093| }
4094| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 4095-4095
```mlir
4095| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 4096-4099
```mlir
4096| 
4097| // There was previously a bug where one of the layout conversions would be
4098| // incorrectly reused during backward rematerialization as an operand to an
4099| // instruction that preceded it.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 4100-4101
```mlir
4100| #blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
4101| #blocked1 = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 4102-4102
```mlir
4102| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 4103-4120
```mlir
4103|   tt.func public @kernel(%arg0: !tt.ptr<f32>) -> (tensor<8xf32, #blocked>, tensor<8xf32, #blocked>, tensor<8xf32, #blocked>) attributes {noinline = false} {
4104|     %0 = tt.make_range {end = 8 : i32, start = 0 : i32} : tensor<8xi32, #blocked1>
4105|     %1 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<8x!tt.ptr<f32>, #blocked1>
4106|     %2 = tt.addptr %1, %0 : tensor<8x!tt.ptr<f32>, #blocked1>, tensor<8xi32, #blocked1>
4107|     %3 = tt.load %2 : tensor<8x!tt.ptr<f32>, #blocked1>
4108|     %4 = math.exp %3 : tensor<8xf32, #blocked1>
4109|     %5 = math.exp %4 : tensor<8xf32, #blocked1>
4110|     %6 = math.exp %5 : tensor<8xf32, #blocked1>
4111|     %7 = math.exp %6 : tensor<8xf32, #blocked1>
4112|     %8 = math.exp %7 : tensor<8xf32, #blocked1>
4113|     %9 = math.exp %8 : tensor<8xf32, #blocked1>
4114|     %10 = math.exp %9 : tensor<8xf32, #blocked1>
4115|     %11 = math.exp %10 : tensor<8xf32, #blocked1>
4116|     %12 = math.exp %11 : tensor<8xf32, #blocked1>
4117|     %13 = math.exp %12 : tensor<8xf32, #blocked1>
4118|     %14 = math.exp %13 : tensor<8xf32, #blocked1>
4119|     %15 = math.exp %14 : tensor<8xf32, #blocked1>
4120|     %16 = math.exp %15 : tensor<8xf32, #blocked1>
```
**EN:** This function-oriented block defines or enters `kernel`. Within it, the test exercises math.exp, tt.func, lane/block index ranges, broadcasted scalars or pointers, pointer arithmetic, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `kernel` 为核心。测试在其中演示 math.exp、tt.func、lane/block 索引范围、广播后的标量或指针、指针算术，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 4121-4137
```mlir
4121|     %17 = math.exp %16 : tensor<8xf32, #blocked1>
4122|     %18 = math.exp %17 : tensor<8xf32, #blocked1>
4123|     %19 = math.exp %18 : tensor<8xf32, #blocked1>
4124|     %20 = math.exp %19 : tensor<8xf32, #blocked1>
4125|     // %21 is too expensive to rematerialize, so we just record a mapping
4126|     // %19 -> %21 for future rematerializations.
4127|     %21 = ttg.convert_layout %19 : tensor<8xf32, #blocked1> -> tensor<8xf32, #blocked>
4128|     // %22 is just below the cost threshold, so we rematerialize the whole chain ending in %18.
4129|     %22 = ttg.convert_layout %18 : tensor<8xf32, #blocked1> -> tensor<8xf32, #blocked>
4130|     // Now that %18 is rematerialized in blocked1, the chain ending %20 is cheap
4131|     // enough to rematerialize. However, when rematerializing %19 as part of
4132|     // this chain, we must not consider %21, as it does not dominate %20.
4133|     %23 = ttg.convert_layout %20 : tensor<8xf32, #blocked1> -> tensor<8xf32, #blocked>
4134|     tt.return %21, %22, %23 : tensor<8xf32, #blocked>, tensor<8xf32, #blocked>, tensor<8xf32, #blocked>
4135|   }
4136| }
4137| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on math.exp, layout conversions, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 math.exp、布局转换、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 4138-4139
```mlir
4138| // CHECK-NOT: ttg.convert_layout
4139| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 4140-4140
```mlir
4140| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 4141-4144
```mlir
4141| 
4142| // Test that we correctly process existing rematerializations when hoisting
4143| // converts on top of ext/broadcast ops.
4144| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 4145-4146
```mlir
4145| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
4146| #blocked1 = #ttg.blocked<{sizePerThread = [4, 4], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 4147-4147
```mlir
4147| module attributes {"ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 4148-4151
```mlir
4148|   tt.func @test_hoist_cvt_existing_remat(%arg0: tensor<128x64xi32, #blocked>) -> (tensor<128x64xi64, #blocked1>, tensor<128x64xi64, #blocked1>) {
4149|     %0 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
4150|     %1 = tt.expand_dims %0 {axis = 1 : i32} : tensor<128xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<128x1xi32, #blocked>
4151|     // TODO: We should be able to reuse the first conversion so there is just 1.
```
**EN:** This function-oriented block defines or enters `test_hoist_cvt_existing_remat`. Within it, the test exercises tt.func, lane/block index ranges, shape expansion, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `test_hoist_cvt_existing_remat` 为核心。测试在其中演示 tt.func、lane/block 索引范围、形状扩展，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 4152-4166
```mlir
4152|     // CHECK: ttg.convert_layout %arg0
4153|     // CHECK-NOT: ttg.convert_layout
4154|     %2 = arith.extsi %arg0 : tensor<128x64xi32, #blocked> to tensor<128x64xi64, #blocked>
4155|     %3 = arith.extsi %1 : tensor<128x1xi32, #blocked> to tensor<128x1xi64, #blocked>
4156|     %4 = tt.broadcast %3 : tensor<128x1xi64, #blocked> -> tensor<128x64xi64, #blocked>
4157|     %5 = arith.muli %2, %4 : tensor<128x64xi64, #blocked>
4158|     %6 = ttg.convert_layout %5 : tensor<128x64xi64, #blocked> -> tensor<128x64xi64, #blocked1>
4159|     %7 = arith.extsi %1 : tensor<128x1xi32, #blocked> to tensor<128x1xi64, #blocked>
4160|     %8 = tt.broadcast %7 : tensor<128x1xi64, #blocked> -> tensor<128x64xi64, #blocked>
4161|     %9 = arith.muli %2, %8 : tensor<128x64xi64, #blocked>
4162|     %10 = ttg.convert_layout %9 : tensor<128x64xi64, #blocked> -> tensor<128x64xi64, #blocked1>
4163|     tt.return %6, %10 : tensor<128x64xi64, #blocked1>, tensor<128x64xi64, #blocked1>
4164|   }
4165| }
4166| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on arith.extsi, tensor broadcasting, integer multiplications, layout conversions, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 arith.extsi、张量广播、整数乘法、布局转换、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 4167-4167
```mlir
4167| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 4168-4171
```mlir
4168| 
4169| // We had a bug where iter args used as the bound of a nested loop would be
4170| // treated as dead and eliminated. Check that this is not the case.
4171| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 4172-4172
```mlir
4172| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 4173-4173
```mlir
4173|   // CHECK-LABEL: @for_arg_used_in_nested_for_bound
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @for_arg_used_in_nested_for_bound anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @for_arg_used_in_nested_for_bound 这样的标签用于锚定匹配范围。

### Lines 4174-4176
```mlir
4174|   tt.func public @for_arg_used_in_nested_for_bound(%arg0: i32, %arg1: i32) -> i32 {
4175|     %c0_i32 = arith.constant 0 : i32
4176|     %c1_i32 = arith.constant 1 : i32
```
**EN:** This function-oriented block defines or enters `for_arg_used_in_nested_for_bound`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `for_arg_used_in_nested_for_bound` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 4177-4179
```mlir
4177|     // CHECK: scf.for
4178|     // CHECK-SAME: iter_args(%{{.*}} = %arg0, %[[OUTER_ARG1:.*]] = %c0_i32)
4179|     %0:2 = scf.for %arg2 = %c0_i32 to %arg1 step %c1_i32 iter_args(%arg3 = %arg0, %arg4 = %c0_i32) -> (i32, i32)  : i32 {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 4180-4181
```mlir
4180|       // CHECK-NEXT: %[[ADD_1:.+]] = arith.addi %[[OUTER_ARG1]], %c1_i32 : i32
4181|       %1 = arith.addi %arg4, %c1_i32 : i32
```
**EN:** This operational block carries the concrete IR steps for the test, centered on integer additions. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 整数加法。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 4182-4193
```mlir
4182|       // CHECK-NEXT: scf.for %arg5 = %c0_i32 to %[[ADD_1:.+]] step %c1_i32
4183|       %2 = scf.for %arg5 = %c0_i32 to %1 step %c1_i32 iter_args(%arg6 = %arg3) -> (i32)  : i32 {
4184|         %4 = arith.addi %arg6, %arg5 : i32
4185|         scf.yield %4 : i32
4186|       }
4187|       %3 = arith.addi %arg3, %c1_i32 : i32
4188|       scf.yield %2, %3 : i32, i32
4189|     }
4190|     tt.return %0#0 : i32
4191|   }
4192| }
4193| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining integer additions, loop/if yielded values, structured loops, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 整数加法、循环/分支产出值、结构化循环、tt.return。

### Lines 4194-4194
```mlir
4194| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 4195-4198
```mlir
4195| 
4196| #src = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [1, 1], warpsPerCTA = [1, 1], order = [0, 1]}>
4197| #dst = #ttg.blocked<{sizePerThread = [1, 2, 2], threadsPerWarp = [1, 1, 1], warpsPerCTA = [1, 1, 1], order = [0, 1, 2]}>
4198| #lin = #ttg.linear<{register = [[0, 1, 0]], lane = [], warp = [], block = []}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 4199-4199
```mlir
4199| module attributes {"ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 4200-4202
```mlir
4200|   // CHECK-LABEL: @test_existing_layout_conflict
4201|   // CHECK: ttg.convert_layout
4202|   // CHECK: tt.return
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @test_existing_layout_conflict anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @test_existing_layout_conflict 这样的标签用于锚定匹配范围。

### Lines 4203-4214
```mlir
4203|   tt.func @test_existing_layout_conflict() -> tensor<1x2x2xi32, #dst> {
4204|     %r = tt.make_range {end = 2 : i32, start = 0 : i32} : tensor<2xi32, #ttg.slice<{dim = 0, parent = #src}>>
4205|     %v = tt.expand_dims %r {axis = 0 : i32} : tensor<2xi32, #ttg.slice<{dim = 0, parent = #src}>> -> tensor<1x2xi32, #src>
4206|     %j = tt.join %v, %v : tensor<1x2xi32, #src> -> tensor<1x2x2xi32, #lin>
4207|     %r3 = tt.reshape %v : tensor<1x2xi32, #src> -> tensor<1x2x1xi32, #lin>
4208|     %b = tt.broadcast %r3 : tensor<1x2x1xi32, #lin> -> tensor<1x2x2xi32, #lin>
4209|     %s = arith.addi %j, %b : tensor<1x2x2xi32, #lin>
4210|     %o = ttg.convert_layout %s : tensor<1x2x2xi32, #lin> -> tensor<1x2x2xi32, #dst>
4211|     tt.return %o : tensor<1x2x2xi32, #dst>
4212|   }
4213| }
4214| 
```
**EN:** This function-oriented block defines or enters `test_existing_layout_conflict`. Within it, the test exercises tt.func, lane/block index ranges, shape expansion, tt.join, tensor reshaping, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `test_existing_layout_conflict` 为核心。测试在其中演示 tt.func、lane/block 索引范围、形状扩展、tt.join、张量重塑，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 4215-4215
```mlir
4215| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 4216-4220
```mlir
4216| 
4217| #blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
4218| #blocked2 = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
4219| #linear = #ttg.linear<{register = [[1], [16]], lane = [[0], [0], [2], [4], [8]], warp = [[0], [0]], block = []}>
4220| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 4221-4221
```mlir
4221| module attributes {"ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 4222-4222
```mlir
4222|   // CHECK-LABEL: @cat_incompatible_target_keeps_convert
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @cat_incompatible_target_keeps_convert anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @cat_incompatible_target_keeps_convert 这样的标签用于锚定匹配范围。

### Lines 4223-4225
```mlir
4223|   tt.func public @cat_incompatible_target_keeps_convert(%out: !tt.ptr<i32>) {
4224|     %lhs = arith.constant dense<0> : tensor<16xi32, #blocked>
4225|     %rhs = arith.constant dense<1> : tensor<16xi32, #blocked>
```
**EN:** This function-oriented block defines or enters `cat_incompatible_target_keeps_convert`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `cat_incompatible_target_keeps_convert` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 4226-4227
```mlir
4226|     // CHECK: %[[CAT:[^ ]+]] = tt.cat
4227|     %cat = tt.cat %lhs, %rhs : tensor<16xi32, #blocked> -> tensor<32xi32, #blocked2>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.cat. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.cat。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 4228-4230
```mlir
4228|     // CHECK: %[[CVT:[^ ]+]] = ttg.convert_layout %[[CAT]]
4229|     %cvt = ttg.convert_layout %cat {allocation.offset = 0 : i32} : tensor<32xi32, #blocked2> -> tensor<32xi32, #linear>
4230|     %ptr = tt.splat %out : !tt.ptr<i32> -> tensor<32x!tt.ptr<i32>, #linear>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, broadcasted scalars or pointers. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、广播后的标量或指针。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 4231-4236
```mlir
4231|     // CHECK: tt.store {{.*}}, %[[CVT]]
4232|     tt.store %ptr, %cvt : tensor<32x!tt.ptr<i32>, #linear>
4233|     tt.return
4234|   }
4235| }
4236| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized stores, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的存储、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 4237-4237
```mlir
4237| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 4238-4242
```mlir
4238| 
4239| #blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
4240| #blocked2 = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
4241| #linear_bcast = #ttg.linear<{register = [[0]], lane = [[1], [2], [4], [8], [16]], warp = [[0], [0]], block = []}>
4242| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 4243-4243
```mlir
4243| module attributes {"ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 4244-4244
```mlir
4244|   // CHECK-LABEL: @cat_target_adds_broadcasting_keeps_convert
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @cat_target_adds_broadcasting_keeps_convert anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @cat_target_adds_broadcasting_keeps_convert 这样的标签用于锚定匹配范围。

### Lines 4245-4247
```mlir
4245|   tt.func public @cat_target_adds_broadcasting_keeps_convert(%out: !tt.ptr<i32>) {
4246|     %lhs = arith.constant dense<0> : tensor<16xi32, #blocked>
4247|     %rhs = arith.constant dense<1> : tensor<16xi32, #blocked>
```
**EN:** This function-oriented block defines or enters `cat_target_adds_broadcasting_keeps_convert`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `cat_target_adds_broadcasting_keeps_convert` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 4248-4249
```mlir
4248|     // CHECK: %[[CAT:[^ ]+]] = tt.cat
4249|     %cat = tt.cat %lhs, %rhs : tensor<16xi32, #blocked> -> tensor<32xi32, #blocked2>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.cat. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.cat。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 4250-4252
```mlir
4250|     // CHECK: %[[CVT:[^ ]+]] = ttg.convert_layout %[[CAT]]
4251|     %cvt = ttg.convert_layout %cat : tensor<32xi32, #blocked2> -> tensor<32xi32, #linear_bcast>
4252|     %ptr = tt.splat %out : !tt.ptr<i32> -> tensor<32x!tt.ptr<i32>, #linear_bcast>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, broadcasted scalars or pointers. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、广播后的标量或指针。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 4253-4258
```mlir
4253|     // CHECK: tt.store {{.*}}, %[[CVT]]
4254|     tt.store %ptr, %cvt : tensor<32x!tt.ptr<i32>, #linear_bcast>
4255|     tt.return
4256|   }
4257| }
4258| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized stores, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的存储、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 4259-4259
```mlir
4259| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 4260-4263
```mlir
4260| 
4261| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 1], order = [0, 1]}>
4262| #blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 1], order = [1, 0]}>
4263| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 4264-4264
```mlir
4264| module attributes {"ttg.num-warps" = 1 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 4265-4265
```mlir
4265|   // CHECK-LABEL: @register_reorder_convert_kept
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @register_reorder_convert_kept anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @register_reorder_convert_kept 这样的标签用于锚定匹配范围。

### Lines 4266-4270
```mlir
4266|   tt.func public @register_reorder_convert_kept() -> (tensor<1x32xf32, #blocked1>, tensor<1x32xf32, #blocked>) {
4267|     %range = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
4268|     %expanded = tt.expand_dims %range {axis = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x32xi32, #blocked>
4269|     %fp = arith.sitofp %expanded : tensor<1x32xi32, #blocked> to tensor<1x32xf32, #blocked>
4270|     %exp = math.exp2 %fp : tensor<1x32xf32, #blocked>
```
**EN:** This function-oriented block defines or enters `register_reorder_convert_kept`. Within it, the test exercises tt.func, lane/block index ranges, shape expansion, arith.sitofp, math.exp2, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `register_reorder_convert_kept` 为核心。测试在其中演示 tt.func、lane/block 索引范围、形状扩展、arith.sitofp、math.exp2，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 4271-4272
```mlir
4271|     // CHECK: ttg.convert_layout
4272|     %cvt = ttg.convert_layout %exp : tensor<1x32xf32, #blocked> -> tensor<1x32xf32, #blocked1>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 4273-4277
```mlir
4273|     // CHECK: tt.return
4274|     tt.return %cvt, %exp : tensor<1x32xf32, #blocked1>, tensor<1x32xf32, #blocked>
4275|   }
4276| }
4277| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 4278-4278
```mlir
4278| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 4279-4282
```mlir
4279| 
4280| #blocked = #ttg.blocked<{sizePerThread = [1, 1, 4], threadsPerWarp = [1, 1, 32], warpsPerCTA = [1, 1, 4], order = [2, 1, 0]}>
4281| #blocked1 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
4282| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 4283-4283
```mlir
4283| module attributes {"ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.num-ctas" = 1 : i32, ttg.target = "cuda:90"} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 4284-4284
```mlir
4284|   // CHECK-LABEL: @remove_layout_avoids_per_lane_broadcast
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @remove_layout_avoids_per_lane_broadcast anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @remove_layout_avoids_per_lane_broadcast 这样的标签用于锚定匹配范围。

### Lines 4285-4285
```mlir
4285|   tt.func public @remove_layout_avoids_per_lane_broadcast(%arg0: tensor<2x1x1xf32, #blocked>) -> tensor<2xf32, #ttg.slice<{dim = 1, parent = #ttg.slice<{dim = 2, parent = #blocked}>}>> {
```
**EN:** This function-oriented block defines or enters `remove_layout_avoids_per_lane_broadcast`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `remove_layout_avoids_per_lane_broadcast` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 4286-4287
```mlir
4286|     // CHECK: tt.broadcast
4287|     %bcast = tt.broadcast %arg0 : tensor<2x1x1xf32, #blocked> -> tensor<2x4x1024xf32, #blocked>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor broadcasting. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量广播。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 4288-4290
```mlir
4288|     // CHECK-NOT: tensor<2x4x1024xf32, #linear
4289|     // CHECK: arith.addf
4290|     %add = arith.addf %bcast, %bcast : tensor<2x4x1024xf32, #blocked>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on floating-point additions. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 浮点加法。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 4291-4293
```mlir
4291|     // CHECK-NOT: tensor<2x4x1024xf32, #linear
4292|     // CHECK: tt.reshape
4293|     %reshape = tt.reshape %add : tensor<2x4x1024xf32, #blocked> -> tensor<2x4096xf32, #blocked1>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor reshaping. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量重塑。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 4294-4299
```mlir
4294|     // CHECK: "tt.reduce"
4295|     %sum = "tt.reduce"(%reshape) <{axis = 1 : i32}> ({
4296|     ^bb0(%lhs: f32, %rhs: f32):
4297|       %sumf = arith.addf %lhs, %rhs : f32
4298|       tt.reduce.return %sumf : f32
4299|     }) : (tensor<2x4096xf32, #blocked1>) -> tensor<2xf32, #ttg.slice<{dim = 1, parent = #blocked1}>>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on floating-point additions, reductions. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 浮点加法、归约。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 4300-4301
```mlir
4300|     // CHECK: ttg.convert_layout
4301|     %sum_cvt = ttg.convert_layout %sum : tensor<2xf32, #ttg.slice<{dim = 1, parent = #blocked1}>> -> tensor<2xf32, #ttg.slice<{dim = 1, parent = #ttg.slice<{dim = 2, parent = #blocked}>}>>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 4302-4306
```mlir
4302|     // CHECK: tt.return
4303|     tt.return %sum_cvt : tensor<2xf32, #ttg.slice<{dim = 1, parent = #ttg.slice<{dim = 2, parent = #blocked}>}>>
4304|   }
4305| }
4306| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 4307-4307
```mlir
4307| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 4308-4310
```mlir
4308| 
4309| // CHECK-LABEL: remat_cycle_single_use
4310| // CHECK-NOT: ttg.convert_layout
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: remat_cycle_single_use anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: remat_cycle_single_use 这样的标签用于锚定匹配范围。

### Lines 4311-4312
```mlir
4311| #blocked = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
4312| #blocked1 = #ttg.blocked<{sizePerThread = [2, 1], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [0, 1]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 4313-4313
```mlir
4313| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:80", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:80`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:80`。

### Lines 4314-4331
```mlir
4314|   tt.func public @remat_cycle_single_use(%arg0: !tt.ptr<f32>, %arg1: i32) {
4315|     %c0_i32 = arith.constant 0 : i32
4316|     %c1_i32 = arith.constant 1 : i32
4317|     %cst = arith.constant dense<0.000000e+00> : tensor<8x8xf32, #blocked>
4318|     %0 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<8x8x!tt.ptr<f32>, #blocked>
4319|     %1 = scf.for %arg2 = %c0_i32 to %arg1 step %c1_i32 iter_args(%arg3 = %cst) -> (tensor<8x8xf32, #blocked>)  : i32 {
4320|       %2 = tt.load %0 : tensor<8x8x!tt.ptr<f32>, #blocked>
4321|       %3 = math.exp %2 : tensor<8x8xf32, #blocked>
4322|       %4 = math.exp %3 : tensor<8x8xf32, #blocked>
4323|       %5 = math.exp %4 : tensor<8x8xf32, #blocked>
4324|       %6 = math.exp %5 : tensor<8x8xf32, #blocked>
4325|       %7 = math.exp %6 : tensor<8x8xf32, #blocked>
4326|       %8 = arith.addf %arg3, %7 : tensor<8x8xf32, #blocked>
4327|       %9 = ttg.convert_layout %8 : tensor<8x8xf32, #blocked> -> tensor<8x8xf32, #blocked1>
4328|       "use"(%9) : (tensor<8x8xf32, #blocked1>) -> ()
4329|       scf.yield %8 : tensor<8x8xf32, #blocked>
4330|     }
4331|     tt.return
```
**EN:** This function-oriented block defines or enters `remat_cycle_single_use`. Within it, the test exercises math.exp, constants, tt.func, broadcasted scalars or pointers, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `remat_cycle_single_use` 为核心。测试在其中演示 math.exp、常量、tt.func、广播后的标量或指针、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 4332-4334
```mlir
4332|   }
4333| }
4334| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 4335-4335
```mlir
4335| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 4336-4338
```mlir
4336| 
4337| // CHECK-LABEL: remat_for_iter_arg
4338| // CHECK-NOT: ttg.convert_layout
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: remat_for_iter_arg anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: remat_for_iter_arg 这样的标签用于锚定匹配范围。

### Lines 4339-4340
```mlir
4339| #blocked = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
4340| #blocked1 = #ttg.blocked<{sizePerThread = [2, 1], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [0, 1]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 4341-4341
```mlir
4341| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:80", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:80`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:80`。

### Lines 4342-4359
```mlir
4342|   tt.func public @remat_for_iter_arg(%arg0: !tt.ptr<f32>, %arg1: i32) {
4343|     %c0_i32 = arith.constant 0 : i32
4344|     %c1_i32 = arith.constant 1 : i32
4345|     %ptrs = tt.splat %arg0 : !tt.ptr<f32> -> tensor<8x8x!tt.ptr<f32>, #blocked>
4346|     %0 = tt.load %ptrs : tensor<8x8x!tt.ptr<f32>, #blocked>
4347|     %1 = math.exp %0 : tensor<8x8xf32, #blocked>
4348|     %2 = math.exp %1 : tensor<8x8xf32, #blocked>
4349|     %3 = math.exp %2 : tensor<8x8xf32, #blocked>
4350|     %4 = math.exp %3 : tensor<8x8xf32, #blocked>
4351|     %5 = math.exp %4 : tensor<8x8xf32, #blocked>
4352|     %6 = scf.for %arg2 = %c0_i32 to %arg1 step %c1_i32 iter_args(%arg3 = %5) -> (tensor<8x8xf32, #blocked>)  : i32 {
4353|       %7 = ttg.convert_layout %arg3 : tensor<8x8xf32, #blocked> -> tensor<8x8xf32, #blocked1>
4354|       "use"(%7) : (tensor<8x8xf32, #blocked1>) -> ()
4355|       scf.yield %arg3 : tensor<8x8xf32, #blocked>
4356|     }
4357|     tt.return
4358|   }
4359| }
```
**EN:** This function-oriented block defines or enters `remat_for_iter_arg`. Within it, the test exercises math.exp, tt.func, constants, broadcasted scalars or pointers, masked or vectorized loads, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `remat_for_iter_arg` 为核心。测试在其中演示 math.exp、tt.func、常量、广播后的标量或指针、带掩码或向量化的加载，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 4361-4362
```mlir
4361| 
4362| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 4362-4364
```mlir
4362| 
4363| // CHECK-LABEL: remat_if_yield
4364| // CHECK-NOT: ttg.convert_layout
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: remat_if_yield anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: remat_if_yield 这样的标签用于锚定匹配范围。

### Lines 4365-4366
```mlir
4365| #blocked = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
4366| #blocked1 = #ttg.blocked<{sizePerThread = [2, 1], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [0, 1]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 4367-4367
```mlir
4367| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:80", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:80`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:80`。

### Lines 4368-4385
```mlir
4368|   tt.func public @remat_if_yield(%arg0: !tt.ptr<f32>, %cond: i1) -> tensor<8x8xf32, #blocked1> {
4369|     %0 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<8x8x!tt.ptr<f32>, #blocked>
4370|     %1 = tt.load %0 : tensor<8x8x!tt.ptr<f32>, #blocked>
4371|     %2 = math.exp %1 : tensor<8x8xf32, #blocked>
4372|     %3 = math.exp %2 : tensor<8x8xf32, #blocked>
4373|     %4 = math.exp %3 : tensor<8x8xf32, #blocked>
4374|     %5 = math.exp %4 : tensor<8x8xf32, #blocked>
4375|     %6 = math.exp %5 : tensor<8x8xf32, #blocked>
4376|     %7 = scf.if %cond -> tensor<8x8xf32, #blocked> {
4377|       scf.yield %6 : tensor<8x8xf32, #blocked>
4378|     } else {
4379|       scf.yield %6 : tensor<8x8xf32, #blocked>
4380|     }
4381|     %8 = ttg.convert_layout %7 : tensor<8x8xf32, #blocked> -> tensor<8x8xf32, #blocked1>
4382|     tt.return %8: tensor<8x8xf32, #blocked1>
4383|   }
4384| }
4385| 
```
**EN:** This function-oriented block defines or enters `remat_if_yield`. Within it, the test exercises math.exp, tt.func, loop/if yielded values, broadcasted scalars or pointers, masked or vectorized loads, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `remat_if_yield` 为核心。测试在其中演示 math.exp、tt.func、循环/分支产出值、广播后的标量或指针、带掩码或向量化的加载，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 4386-4386
```mlir
4386| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 4387-4389
```mlir
4387| 
4388| // CHECK-LABEL: remat_if_nested_yield
4389| // CHECK-NOT: ttg.convert_layout
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: remat_if_nested_yield anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: remat_if_nested_yield 这样的标签用于锚定匹配范围。

### Lines 4390-4391
```mlir
4390| #blocked = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
4391| #blocked1 = #ttg.blocked<{sizePerThread = [2, 1], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [0, 1]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 4392-4392
```mlir
4392| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:80", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:80`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:80`。

### Lines 4393-4410
```mlir
4393|   tt.func public @remat_if_nested_yield(%arg0: !tt.ptr<f32>, %cond1: i1, %cond2: i1) -> tensor<8x8xf32, #blocked1> {
4394|     %0 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<8x8x!tt.ptr<f32>, #blocked>
4395|     %load = tt.load %0 : tensor<8x8x!tt.ptr<f32>, #blocked>
4396|     %outer = scf.if %cond1 -> tensor<8x8xf32, #blocked> {
4397|       %inner = scf.if %cond2 -> tensor<8x8xf32, #blocked> {
4398|         %1 = math.exp %load : tensor<8x8xf32, #blocked>
4399|         %2 = math.exp %1 : tensor<8x8xf32, #blocked>
4400|         %3 = math.exp %2 : tensor<8x8xf32, #blocked>
4401|         %4 = math.exp %3 : tensor<8x8xf32, #blocked>
4402|         %5 = math.exp %4 : tensor<8x8xf32, #blocked>
4403|         scf.yield %5 : tensor<8x8xf32, #blocked>
4404|       } else {
4405|         scf.yield %load : tensor<8x8xf32, #blocked>
4406|       }
4407|       scf.yield %inner : tensor<8x8xf32, #blocked>
4408|     } else {
4409|       scf.yield %load : tensor<8x8xf32, #blocked>
4410|     }
```
**EN:** This function-oriented block defines or enters `remat_if_nested_yield`. Within it, the test exercises math.exp, loop/if yielded values, tt.func, structured conditionals, broadcasted scalars or pointers, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `remat_if_nested_yield` 为核心。测试在其中演示 math.exp、循环/分支产出值、tt.func、结构化条件分支、广播后的标量或指针，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 4411-4415
```mlir
4411|     %cvt = ttg.convert_layout %outer : tensor<8x8xf32, #blocked> -> tensor<8x8xf32, #blocked1>
4412|     tt.return %cvt : tensor<8x8xf32, #blocked1>
4413|   }
4414| }
4415| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 4416-4416
```mlir
4416| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 4417-4422
```mlir
4417| 
4418| // Test that when the result of an IfOp is used outside the slice being
4419| // rematerialized, we are able to propagate that information back to the yield
4420| // operands. This prevents eliminating the convert_layout, because the cost is
4421| // too high.
4422| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 4423-4424
```mlir
4423| // CHECK-LABEL: remat_if_yield_in_branch_multi_use_negative
4424| // CHECK: ttg.convert_layout
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: remat_if_yield_in_branch_multi_use_negative anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: remat_if_yield_in_branch_multi_use_negative 这样的标签用于锚定匹配范围。

### Lines 4425-4426
```mlir
4425| #blocked = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
4426| #blocked1 = #ttg.blocked<{sizePerThread = [2, 1], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [0, 1]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 4427-4427
```mlir
4427| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:80", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:80`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:80`。

### Lines 4428-4445
```mlir
4428|   tt.func public @remat_if_yield_in_branch_multi_use_negative(%arg0: !tt.ptr<f32>, %cond: i1) -> (tensor<8x8xf32, #blocked>, tensor<8x8xf32, #blocked1>) {
4429|     %0 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<8x8x!tt.ptr<f32>, #blocked>
4430|     %load = tt.load %0 : tensor<8x8x!tt.ptr<f32>, #blocked>
4431|     %result = scf.if %cond -> tensor<8x8xf32, #blocked> {
4432|       %1 = math.exp %load : tensor<8x8xf32, #blocked>
4433|       %2 = math.exp %1 : tensor<8x8xf32, #blocked>
4434|       %3 = math.exp %2 : tensor<8x8xf32, #blocked>
4435|       %4 = math.exp %3 : tensor<8x8xf32, #blocked>
4436|       %5 = math.exp %4 : tensor<8x8xf32, #blocked>
4437|       scf.yield %5 : tensor<8x8xf32, #blocked>
4438|     } else {
4439|       scf.yield %load : tensor<8x8xf32, #blocked>
4440|     }
4441|     %cvt = ttg.convert_layout %result : tensor<8x8xf32, #blocked> -> tensor<8x8xf32, #blocked1>
4442|     tt.return %result, %cvt: tensor<8x8xf32, #blocked>, tensor<8x8xf32, #blocked1>
4443|   }
4444| }
4445| 
```
**EN:** This function-oriented block defines or enters `remat_if_yield_in_branch_multi_use_negative`. Within it, the test exercises math.exp, tt.func, loop/if yielded values, broadcasted scalars or pointers, masked or vectorized loads, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `remat_if_yield_in_branch_multi_use_negative` 为核心。测试在其中演示 math.exp、tt.func、循环/分支产出值、广播后的标量或指针、带掩码或向量化的加载，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 4446-4446
```mlir
4446| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 4447-4451
```mlir
4447| 
4448| // Test that when the block arg is used outside the slice being rematerialized,
4449| // we are able to propagate that information back to the loop operands. This
4450| // prevents eliminating the convert_layout, because the cost is too high.
4451| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 4452-4453
```mlir
4452| // CHECK-LABEL: remat_for_iter_arg_multi_use_negative
4453| // CHECK: ttg.convert_layout
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: remat_for_iter_arg_multi_use_negative anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: remat_for_iter_arg_multi_use_negative 这样的标签用于锚定匹配范围。

### Lines 4454-4455
```mlir
4454| #blocked = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
4455| #blocked1 = #ttg.blocked<{sizePerThread = [2, 1], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [0, 1]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 4456-4456
```mlir
4456| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:80", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:80`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:80`。

### Lines 4457-4474
```mlir
4457|   tt.func public @remat_for_iter_arg_multi_use_negative(%arg0: !tt.ptr<f32>, %arg1: i32) {
4458|     %c0_i32 = arith.constant 0 : i32
4459|     %c1_i32 = arith.constant 1 : i32
4460|     %ptrs = tt.splat %arg0 : !tt.ptr<f32> -> tensor<8x8x!tt.ptr<f32>, #blocked>
4461|     %0 = tt.load %ptrs : tensor<8x8x!tt.ptr<f32>, #blocked>
4462|     %1 = math.exp %0 : tensor<8x8xf32, #blocked>
4463|     %2 = math.exp %1 : tensor<8x8xf32, #blocked>
4464|     %3 = math.exp %2 : tensor<8x8xf32, #blocked>
4465|     %4 = math.exp %3 : tensor<8x8xf32, #blocked>
4466|     %5 = math.exp %4 : tensor<8x8xf32, #blocked>
4467|     %6 = scf.for %arg2 = %c0_i32 to %arg1 step %c1_i32 iter_args(%arg3 = %5) -> (tensor<8x8xf32, #blocked>)  : i32 {
4468|       %7 = ttg.convert_layout %arg3 : tensor<8x8xf32, #blocked> -> tensor<8x8xf32, #blocked1>
4469|       "use"(%7) : (tensor<8x8xf32, #blocked1>) -> ()
4470|       "other_use"(%arg3) : (tensor<8x8xf32, #blocked>) -> ()
4471|       scf.yield %arg3 : tensor<8x8xf32, #blocked>
4472|     }
4473|     tt.return
4474|   }
```
**EN:** This function-oriented block defines or enters `remat_for_iter_arg_multi_use_negative`. Within it, the test exercises math.exp, tt.func, constants, broadcasted scalars or pointers, masked or vectorized loads, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `remat_for_iter_arg_multi_use_negative` 为核心。测试在其中演示 math.exp、tt.func、常量、广播后的标量或指针、带掩码或向量化的加载，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 4475-4475
```mlir
4475| }
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-allow-unregistered-dialect`, `-tritongpu-remove-layout-conversions`, `-cse`
- **CN:** 主要 pass 选项：`-split-input-file`，`-allow-unregistered-dialect`，`-tritongpu-remove-layout-conversions`，`-cse`
- **EN:** Dominant operations include `ttg.convert_layout`, `arith.constant`, `tt.func`, `tt.splat`, `tt.return`, `tt.load`, `tt.addptr`, `arith.addi`, `tt.broadcast`, `arith.cmpi`.
- **CN:** 主要操作包括 `ttg.convert_layout`、`arith.constant`、`tt.func`、`tt.splat`、`tt.return`、`tt.load`、`tt.addptr`、`arith.addi`、`tt.broadcast`、`arith.cmpi`。
- **EN:** The file contains 83 independently testable section(s). Check styles used: CHECK x295, CHECK-LABEL x97, CHECK-NOT x77, CHECK-NEXT x59. Important labels include cst, range, splat, remat. It validates both the presence of the desired rewrite and the absence of unwanted residual operations.
- **CN:** 该文件包含 83 个可独立测试的分段。使用的检查类型：CHECK ×295，CHECK-LABEL ×97，CHECK-NOT ×77，CHECK-NEXT ×59。 关键标签包括 cst，range，splat，remat。 它同时验证目标重写已经出现，并且不再残留不需要的操作。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `tt`: Triton high-level tensor/pointer operations.
- **CN:** `tt`：Triton 高层张量/指针操作。
- **EN:** `ttg`: TritonGPU layout, memory, and GPU-specific operations.
- **CN:** `ttg`：TritonGPU 布局、内存和 GPU 专用操作。
- **EN:** `arith`: MLIR arithmetic dialect for scalar/tensor math.
- **CN:** `arith`：用于标量/张量运算的 MLIR arithmetic 方言。
- **EN:** `scf`: Structured control flow for loops and conditionals.
- **CN:** `scf`：用于循环与条件分支的结构化控制流。
- **EN:** `ttng`: Triton Nvidia GPU tensor-memory and MMA operations.
- **CN:** `ttng`：Triton Nvidia GPU 张量内存与 MMA 操作。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。