# loop-pipeline.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/loop-pipeline.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-tritongpu-assign-latencies, -tritongpu-schedule-loops, -tritongpu-pipeline=num-stages=3, -canonicalize` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-tritongpu-assign-latencies, -tritongpu-schedule-loops, -tritongpu-pipeline=num-stages=3, -canonicalize` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file -tritongpu-assign-latencies -tritongpu-schedule-loops -tritongpu-pipeline=num-stages=3 -canonicalize | FileCheck %s --check-prefixes=COMMON,CHECK`; `// RUN: triton-opt %s -split-input-file -tritonamdgpu-schedule-loops=num_stages=2 -tritonamdgpu-pipeline -canonicalize | FileCheck %s --check-prefixes=COMMON,AMD`; `// RUN: triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=3" -tritonamdgpu-pipeline -canonicalize | FileCheck %s --check-prefixes=COMMON,AMD_3_STAGES`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file -tritongpu-assign-latencies -tritongpu-schedule-loops -tritongpu-pipeline=num-stages=3 -canonicalize | FileCheck %s --check-prefixes=COMMON,CHECK`；`// RUN: triton-opt %s -split-input-file -tritonamdgpu-schedule-loops=num_stages=2 -tritonamdgpu-pipeline -canonicalize | FileCheck %s --check-prefixes=COMMON,AMD`；`// RUN: triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=3" -tritonamdgpu-pipeline -canonicalize | FileCheck %s --check-prefixes=COMMON,AMD_3_STAGES`

## Line-by-Line Analysis / 逐行分析

### Lines 1-6
```mlir
1| // RUN: triton-opt %s -split-input-file -tritongpu-assign-latencies -tritongpu-schedule-loops -tritongpu-pipeline=num-stages=3 -canonicalize | FileCheck %s --check-prefixes=COMMON,CHECK
2| // RUN: triton-opt %s -split-input-file -tritonamdgpu-schedule-loops=num_stages=2 -tritonamdgpu-pipeline -canonicalize | FileCheck %s --check-prefixes=COMMON,AMD
3| // RUN: triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=3" -tritonamdgpu-pipeline -canonicalize | FileCheck %s --check-prefixes=COMMON,AMD_3_STAGES
4| 
5| // 4 warps
6| // matmul: 128x32 @ 32x128 -> 128x128
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file -tritongpu-assign-latencies -tritongpu-schedule-loops -tritongpu-pipeline=num-stages=3 -canonicalize | FileCheck %s --check-prefixes=COMMON,CHECK` ; ` triton-opt %s -split-input-file -tritonamdgpu-schedule-loops=num_stages=2 -tritonamdgpu-pipeline -canonicalize | FileCheck %s --check-prefixes=COMMON,AMD` ; ` triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=3" -tritonamdgpu-pipeline -canonicalize | FileCheck %s --check-prefixes=COMMON,AMD_3_STAGES` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file -tritongpu-assign-latencies -tritongpu-schedule-loops -tritongpu-pipeline=num-stages=3 -canonicalize | FileCheck %s --check-prefixes=COMMON,CHECK`；` triton-opt %s -split-input-file -tritonamdgpu-schedule-loops=num_stages=2 -tritonamdgpu-pipeline -canonicalize | FileCheck %s --check-prefixes=COMMON,AMD`；` triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=3" -tritonamdgpu-pipeline -canonicalize | FileCheck %s --check-prefixes=COMMON,AMD_3_STAGES`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 7-16
```mlir
 7| #AL = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
 8| #BL = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
 9| #ALs0 = #ttg.slice<{parent=#AL, dim=0}>
10| #BLs0 = #ttg.slice<{parent=#BL, dim=0}>
11| #BLs1 = #ttg.slice<{parent=#BL, dim=1}>
12| #C = #ttg.nvidia_mma<{versionMajor = 2, warpsPerCTA = [4, 1], instrShape = [16, 8]}>
13| #A = #ttg.dot_op<{opIdx = 0, parent = #C, kWidth=2}>
14| #B = #ttg.dot_op<{opIdx = 1, parent = #C, kWidth=2}>
15| #smem = #ttg.shared_memory
16| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 17-34
```mlir
17| // CHECK-LABEL: tt.func @matmul_loop
18| // CHECK-DAG: %[[CONSTANT_NEG1:.*]] = arith.constant -1 : i32
19| // CHECK-DAG: %[[CONSTANT_0:.*]] = arith.constant 0 : i32
20| // CHECK-DAG: %[[CONSTANT_1:.*]] = arith.constant 1 : i32
21| // CHECK-DAG: %[[CONSTANT_2:.*]] = arith.constant 2 : i32
22| // CHECK: %[[ABUFFER:.*]] = ttg.local_alloc
23| // CHECK: %[[BBUFFER:.*]] = ttg.local_alloc
24| // CHECK-DAG: %[[LOOP_COND_0:.*]] = arith.cmpi slt, %[[LB:.*]], %[[UB:.*]]
25| // CHECK-DAG: %[[LOOP_COND_0_SPLAT_A:.*]] = tt.splat %[[LOOP_COND_0]]
26| // CHECK-DAG: %[[ASUB:.*]] = ttg.memdesc_index %[[ABUFFER]]{{\[}}%[[CONSTANT_0]]{{\]}}
27| // CHECK: %[[T_A0:.*]] = ttg.async_copy_global_to_local %{{.*}}, %[[ASUB]] mask %[[LOOP_COND_0_SPLAT_A]]
28| // CHECK-DAG: %[[LOOP_COND_0_SPLAT_B:.*]] = tt.splat %[[LOOP_COND_0]]
29| // CHECK-DAG: %[[BSUB:.*]] = ttg.memdesc_index %[[BBUFFER]]{{\[}}%[[CONSTANT_0]]{{\]}}
30| // CHECK: %[[T_B0:.*]] = ttg.async_copy_global_to_local %{{.*}}, %[[BSUB]] mask %[[LOOP_COND_0_SPLAT_B]] other %{{.*}}
31| // CHECK-DAG: %[[IV_1:.*]] = arith.addi %[[LB]], %[[STEP:.*]]
32| // CHECK-DAG: %[[LOOP_COND_1:.*]] = arith.cmpi slt, %[[IV_1]], %[[UB]]
33| // CHECK-DAG: %[[LOOP_COND_1_SPLAT_A:.*]] = tt.splat %[[LOOP_COND_1]]
34| // CHECK-DAG: %[[ASUB1:.*]] = ttg.memdesc_index %[[ABUFFER]]{{\[}}%[[CONSTANT_1]]{{\]}}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tt.func @matmul_loop anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tt.func @matmul_loop 这样的标签用于锚定匹配范围。

### Lines 35-52
```mlir
35| // CHECK: %[[T_A1:.*]] = ttg.async_copy_global_to_local %{{.*}}, %[[ASUB1]] mask %[[LOOP_COND_1_SPLAT_A]]
36| // CHECK-DAG: %[[LOOP_COND_1_SPLAT_B:.*]] = tt.splat %[[LOOP_COND_1]]
37| // CHECK-DAG: %[[BSUB1:.*]] = ttg.memdesc_index %[[BBUFFER]]{{\[}}%[[CONSTANT_1]]{{\]}}
38| // CHECK: %[[T_B1:.*]] = ttg.async_copy_global_to_local %{{.*}}, %[[BSUB1]] mask %[[LOOP_COND_1_SPLAT_B]]
39| // CHECK: scf.for {{.*}} iter_args({{.*}}, %[[INS_IDX:.*]] = %[[CONSTANT_1]], %[[EXT_IDX:.*]] = %[[CONSTANT_NEG1]]
40| // CHECK-DAG: %[[EXT_IDX_2:.*]] = arith.addi %[[EXT_IDX]], %[[CONSTANT_1]] : i32
41| // CHECK-DAG: %[[CMP_EXT:.*]] = arith.cmpi sge, %[[EXT_IDX_2]], %[[CONSTANT_2]]
42| // CHECK-DAG: %[[EXT_IDX_3:.*]] = arith.select %[[CMP_EXT]], %[[CONSTANT_0]], %[[EXT_IDX_2]]
43| // CHECK-DAG: ttg.async_wait {{.*}} {num = 2 : i32}
44| // CHECK-DAG: %[[A0:.*]] = ttg.memdesc_index %[[ABUFFER]]{{\[}}%[[EXT_IDX_3]]{{\]}}
45| // CHECK:   %[[arg_a0_dot_op:.*]] = ttg.local_load %[[A0]]
46| // CHECK-DAG: %[[B0:.*]] = ttg.memdesc_index %[[BBUFFER]]{{\[}}%[[EXT_IDX_3]]{{\]}}
47| // CHECK:   %[[arg_b0_dot_op_0:.*]] = ttg.local_load %[[B0]]
48| // CHECK:   %[[arg_b0_dot_op_1:.*]] = arith.mulf %[[arg_b0_dot_op_0]]
49| // CHECK:   tt.dot %[[arg_a0_dot_op]], %[[arg_b0_dot_op_1]], {{.*}}
50| // CHECK-DAG: %[[INS_IDX_2:.*]] = arith.addi %[[INS_IDX]], %[[CONSTANT_1]] : i32
51| // CHECK-DAG: %[[CMP_INS:.*]] = arith.cmpi sge, %[[INS_IDX_2]], %[[CONSTANT_2]]
52| // CHECK-DAG: %[[INS_IDX_3:.*]] = arith.select %[[CMP_INS]], %[[CONSTANT_0]], %[[INS_IDX_2]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 53-58
```mlir
53| // CHECK:   %[[ASUB3:.*]] = ttg.memdesc_index %[[ABUFFER]]{{\[}}%[[INS_IDX_3]]{{\]}}
54| // CHECK:   %[[NEXT_A_BUFFER:.*]] = ttg.async_copy_global_to_local {{.*}}, %[[ASUB3]]
55| // CHECK:   %[[BSUB3:.*]] = ttg.memdesc_index %[[BBUFFER]]{{\[}}%[[INS_IDX_3]]{{\]}}
56| // CHECK:   %[[NEXT_B_BUFFER:.*]] = ttg.async_copy_global_to_local {{.*}}, %[[BSUB3]]
57| // CHECK:   scf.yield {{.*}}, %[[INS_IDX_3]], %[[EXT_IDX_3]]
58| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 59-76
```mlir
59| // AMD-LABEL:  tt.func @matmul_loop
60| //   AMD-DAG:   %[[CM1:.*]] = arith.constant -1 : index
61| //   AMD-DAG:   %[[C1:.*]] = arith.constant 1 : index
62| //   AMD-DAG:   %[[C0:.*]] = arith.constant 0 : index
63| //       AMD:   %[[UB1:.*]] = arith.subi %[[UB:.*]], %arg2 : index
64| //       AMD:   %[[FOR:.*]]:6 = scf.for %[[ARG5:.*]] = %[[LB:.*]] to %[[UB1]] step %[[STEP:.*]] iter_args(%[[ARG6:.*]] = %{{.*}}, %[[ARG7:.*]] = %{{.*}}, %[[ARG8:.*]] = %{{.*}}, %[[ARG9:.*]] = %{{.*}}, %[[ARG10:.*]] = %{{.*}}, %[[ARG11:.*]] = %{{.*}})
65| //       AMD:     %[[ADDPTR_34:.*]] = tt.addptr %[[ARG6]], %{{.*}}
66| //       AMD:     %[[ADDPTR_35:.*]] = tt.addptr %[[ARG7]], %{{.*}}
67| //       AMD:     %[[LOAD_36:.*]] = tt.load %[[ADDPTR_34]]
68| //       AMD:     %[[LOCAL_LOAD_37:.*]] = ttg.local_load %[[ARG10]]
69| //       AMD:     %[[LOAD_38:.*]] = tt.load %[[ADDPTR_35]]
70| //       AMD:     %[[LOCAL_LOAD_39:.*]] = ttg.local_load %[[ARG11]]
71| //       AMD:     %[[MULF_40:.*]] = arith.mulf %[[LOCAL_LOAD_39]], %{{.*}}
72| //       AMD:     %[[DOT_41:.*]] = tt.dot %[[LOCAL_LOAD_37]], %[[MULF_40]], %[[ARG8]]
73| //       AMD:     %[[ADDI_42:.*]] = arith.addi %[[ARG9]], %{{.*}}
74| //       AMD:     %[[CMPI_43:.*]] = arith.cmpi slt, %[[ADDI_42]], %{{.*}}
75| //       AMD:     %[[SELECT_44:.*]] = arith.select %[[CMPI_43]], %[[ADDI_42]], %{{.*}}
76| //       AMD:     %[[MEMDESC_SUBVIEW_45:.*]] = ttg.memdesc_index %{{.*}}{{\[}}%[[SELECT_44]]{{\]}}
```
**EN:** This function-oriented block defines or enters `matmul_loop`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `matmul_loop` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 77-94
```mlir
77| //       AMD:     ttg.local_store %[[LOAD_36]], %[[MEMDESC_SUBVIEW_45]]
78| //       AMD:     %[[MEMDESC_SUBVIEW_46:.*]] = ttg.memdesc_index %{{.*}}{{\[}}%[[SELECT_44]]{{\]}}
79| //       AMD:     ttg.local_store %[[LOAD_38]], %[[MEMDESC_SUBVIEW_46]]
80| //       AMD:     scf.yield %[[ADDPTR_34]], %[[ADDPTR_35]], %[[DOT_41]], %[[SELECT_44]], %[[MEMDESC_SUBVIEW_45]], %[[MEMDESC_SUBVIEW_46]]
81| //       AMD:   }
82| //       AMD:   %[[CMPI_21:.*]] = arith.cmpi slt, %[[STEP]], %[[C0]]
83| //       AMD:   %[[SELECT_22:.*]] = arith.select %[[CMPI_21]], %[[C1]], %[[CM1]]
84| //       AMD:   %[[SUBI_23:.*]] = arith.subi %[[UB]], %[[LB]]
85| //       AMD:   %[[ADDI_24:.*]] = arith.addi %[[SUBI_23]], %[[STEP]]
86| //       AMD:   %[[ADDI_25:.*]] = arith.addi %[[ADDI_24]], %[[SELECT_22]]
87| //       AMD:   %[[DIVSI_26:.*]] = arith.divsi %[[ADDI_25]], %[[STEP]]
88| //       AMD:   %[[CMPI_27:.*]] = arith.cmpi sge, %[[DIVSI_26]], %{{.*}}
89| //       AMD:   %[[LOCAL_LOAD_28:.*]] = ttg.local_load %{{.*}}#4
90| //       AMD:   %[[LOCAL_LOAD_29:.*]] = ttg.local_load %{{.*}}#5
91| //       AMD:   %[[MULF_30:.*]] = arith.mulf %[[LOCAL_LOAD_29]], %{{.*}}
92| //       AMD:   %[[IF_31:.*]] = scf.if %[[CMPI_27]]
93| //       AMD:     %[[DOT_33:.*]] = tt.dot %[[LOCAL_LOAD_28]], %[[MULF_30]], %{{.*}}#2
94| //       AMD:     scf.yield %[[DOT_33]]
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 95-101
```mlir
 95| //       AMD:   } else {
 96| //       AMD:     scf.yield %{{.*}}#2
 97| //       AMD:   }
 98| //       AMD:   %[[SELECT_32:.*]] = arith.select %[[CMPI_27]], %[[IF_31]], %{{.*}}#2
 99| //       AMD:   ttg.local_dealloc %{{.*}}
100| //       AMD:   ttg.local_dealloc %{{.*}}
101| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 102-119
```mlir
102| // AMD_3_STAGES-LABEL: tt.func @matmul_loop
103| //       AMD_3_STAGES:   ttg.local_alloc
104| //       AMD_3_STAGES:   ttg.local_alloc
105| //       AMD_3_STAGES:   tt.load
106| //       AMD_3_STAGES:   tt.load
107| //       AMD_3_STAGES:   ttg.local_store
108| //       AMD_3_STAGES:   ttg.local_store
109| //       AMD_3_STAGES:   tt.load
110| //       AMD_3_STAGES:   tt.load
111| //       AMD_3_STAGES:   ttg.local_store
112| //       AMD_3_STAGES:   ttg.local_store
113| //       AMD_3_STAGES:   scf.for
114| //       AMD_3_STAGES:     tt.load
115| //       AMD_3_STAGES:     ttg.local_load
116| //       AMD_3_STAGES:     tt.load
117| //       AMD_3_STAGES:     ttg.local_load
118| //       AMD_3_STAGES:     tt.dot
119| //       AMD_3_STAGES:     ttg.local_store
```
**EN:** This function-oriented block defines or enters `matmul_loop`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `matmul_loop` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 120-125
```mlir
120| //       AMD_3_STAGES:     ttg.local_store
121| //       AMD_3_STAGES:     scf.yield
122| //       AMD_3_STAGES:   tt.dot
123| //       AMD_3_STAGES:   tt.dot
124| //       AMD_3_STAGES:   tt.return
125| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 126-126
```mlir
126| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 127-144
```mlir
127| tt.func @matmul_loop(%lb : index, %ub : index, %step : index,
128|                   %A : !tt.ptr<f16> {tt.divisibility = 16 : i32},
129|                   %B : !tt.ptr<f16> {tt.divisibility = 16 : i32}) -> tensor<128x128xf32, #C> {
130|   // A ptrs
131|   %a_ptr_splat = tt.splat %A : !tt.ptr<f16> -> tensor<128x32x!tt.ptr<f16>, #AL>
132|   %a_tmp0 = tt.make_range {end = 32: i32, start = 0: i32} : tensor<32xi32, #ALs0>
133|   %a_tmp1 = tt.expand_dims %a_tmp0 {axis = 0 : i32} : tensor<32xi32, #ALs0> -> tensor<1x32xi32, #AL>
134|   %a_offs = tt.broadcast %a_tmp1 : tensor<1x32xi32, #AL> -> tensor<128x32xi32, #AL>
135|   %a_ptr_init = tt.addptr %a_ptr_splat, %a_offs : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xi32, #AL>
136|   // B ptrs
137|   %b_ptr_splat = tt.splat %B : !tt.ptr<f16> -> tensor<32x128x!tt.ptr<f16>, #BL>
138|   %b_tmp0 = tt.make_range {end = 128: i32, start = 0: i32} : tensor<128xi32, #BLs0>
139|   %b_tmp1 = tt.expand_dims %b_tmp0 {axis = 0 : i32} : tensor<128xi32, #BLs0> -> tensor<1x128xi32, #BL>
140|   %b_offs = tt.broadcast %b_tmp1 : tensor<1x128xi32, #BL> -> tensor<32x128xi32, #BL>
141|   %b_ptr_init = tt.addptr %b_ptr_splat, %b_offs : tensor<32x128x!tt.ptr<f16>, #BL>, tensor<32x128xi32, #BL>
142| 
143| 
144|   %a_mask = arith.constant dense<true> : tensor<128x32xi1, #AL>
```
**EN:** This function-oriented block defines or enters `matmul_loop`. Within it, the test exercises tt.func, broadcasted scalars or pointers, lane/block index ranges, shape expansion, tensor broadcasting, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `matmul_loop` 为核心。测试在其中演示 tt.func、广播后的标量或指针、lane/block 索引范围、形状扩展、张量广播，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 145-162
```mlir
145|   %a_other = arith.constant dense<0.00e+00> : tensor<128x32xf16, #AL>
146|   %b_mask = arith.constant dense<true> : tensor<32x128xi1, #BL>
147|   %b_other = arith.constant dense<0.00e+00> : tensor<32x128xf16, #BL>
148|   %c_init = arith.constant dense<0.00e+00> : tensor<128x128xf32, #C>
149| 
150|   %a_off = arith.constant dense<4> : tensor<128x32xi32, #AL>
151|   %b_off = arith.constant dense<4> : tensor<32x128xi32, #BL>
152| 
153|   %b_scale = arith.constant dense<4.> : tensor<32x128xf16, #B>
154| 
155|   %loop:3 = scf.for %iv = %lb to %ub step %step iter_args(%a_ptr = %a_ptr_init, %b_ptr = %b_ptr_init, %prev_c = %c_init) -> (tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>) {
156|     %a_ = tt.load %a_ptr : tensor<128x32x!tt.ptr<f16>, #AL>
157|     %a = ttg.convert_layout %a_ : tensor<128x32xf16, #AL> -> tensor<128x32xf16, #A>
158|     %b__ = tt.load %b_ptr, %b_mask, %b_other : tensor<32x128x!tt.ptr<f16>, #BL>
159|     %b_ = ttg.convert_layout %b__ : tensor<32x128xf16, #BL> -> tensor<32x128xf16, #B>
160|     %b = arith.mulf %b_, %b_scale: tensor<32x128xf16, #B>
161| 
162|     %c = tt.dot %a, %b, %prev_c : tensor<128x32xf16, #A> * tensor<32x128xf16, #B> -> tensor<128x128xf32, #C>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining constants, masked or vectorized loads, layout conversions, structured loops, arith.mulf.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 常量、带掩码或向量化的加载、布局转换、结构化循环、arith.mulf。

### Lines 163-170
```mlir
163| 
164|     %next_a_ptr = tt.addptr %a_ptr, %a_off : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xi32, #AL>
165|     %next_b_ptr = tt.addptr %b_ptr, %b_off : tensor<32x128x!tt.ptr<f16>, #BL>, tensor<32x128xi32, #BL>
166|     scf.yield %next_a_ptr, %next_b_ptr, %c : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>
167|   }
168|   tt.return %loop#2: tensor<128x128xf32, #C>
169| }
170| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining pointer arithmetic, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 指针算术、循环/分支产出值、tt.return。

### Lines 171-188
```mlir
171| // CHECK-LABEL: tt.func @matmul_loop_nested
172| // CHECK-DAG: %[[CONSTANT_NEG1:.*]] = arith.constant -1 : i32
173| // CHECK-DAG: %[[CONSTANT_0:.*]] = arith.constant 0 : i32
174| // CHECK-DAG: %[[CONSTANT_1:.*]] = arith.constant 1 : i32
175| // CHECK-DAG: %[[CONSTANT_2:.*]] = arith.constant 2 : i32
176| // CHECK: scf.for
177| // CHECK:   %[[ABUFFER:.*]] = ttg.local_alloc
178| // CHECK:   %[[BBUFFER:.*]] = ttg.local_alloc
179| // CHECK:   ttg.memdesc_index %[[ABUFFER]]{{\[}}%[[CONSTANT_0]]{{\]}}
180| // CHECK:   ttg.async_copy_global_to_local
181| // CHECK:   ttg.memdesc_index %[[BBUFFER]]{{\[}}%[[CONSTANT_0]]{{\]}}
182| // CHECK:   ttg.async_copy_global_to_local
183| // CHECK:   ttg.memdesc_index %[[ABUFFER]]{{\[}}%[[CONSTANT_1]]{{\]}}
184| // CHECK:   ttg.async_copy_global_to_local
185| // CHECK:   ttg.memdesc_index %[[BBUFFER]]{{\[}}%[[CONSTANT_1]]{{\]}}
186| // CHECK:   ttg.async_copy_global_to_local
187| // CHECK:   scf.for {{.*}} iter_args({{.*}}, %[[INS_IDX:.*]] = %[[CONSTANT_1]], %[[EXT_IDX:.*]] = %[[CONSTANT_NEG1]]{{.*}}
188| // CHECK:     %[[EXT_IDX_2:.*]] = arith.addi %[[EXT_IDX]], %[[CONSTANT_1]] : i32
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tt.func @matmul_loop_nested anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tt.func @matmul_loop_nested 这样的标签用于锚定匹配范围。

### Lines 189-206
```mlir
189| // CHECK:     %[[CMP_EXT:.*]] = arith.cmpi sge, %[[EXT_IDX_2]], %[[CONSTANT_2]]
190| // CHECK:     %[[EXT_IDX_3:.*]] = arith.select %[[CMP_EXT]], %[[CONSTANT_0]], %[[EXT_IDX_2]]
191| // CHECK:     ttg.async_wait {{.*}} {num = 2 : i32}
192| // CHECK:     %[[A:.*]] = ttg.memdesc_index %[[ABUFFER]]{{\[}}%[[EXT_IDX_3]]{{\]}}
193| // CHECK:     %[[arg_a0_dot_op:.*]] = ttg.local_load %[[A]]
194| // CHECK:     %[[B:.*]] = ttg.memdesc_index %[[BBUFFER]]{{\[}}%[[EXT_IDX_3]]{{\]}}
195| // CHECK:     %[[arg_b0_dot_op_0:.*]] = ttg.local_load %[[B]]
196| // CHECK:     tt.dot %[[arg_a0_dot_op]], %[[arg_b0_dot_op_0]], {{.*}}
197| // CHECK-DAG: %[[INS_IDX_2:.*]] = arith.addi %[[INS_IDX]], %[[CONSTANT_1]] : i32
198| // CHECK-DAG: %[[CMP_INS:.*]] = arith.cmpi sge, %[[INS_IDX_2]], %[[CONSTANT_2]]
199| // CHECK-DAG: %[[INS_IDX_3:.*]] = arith.select %[[CMP_INS]], %[[CONSTANT_0]], %[[INS_IDX_2]]
200| // CHECK:     ttg.memdesc_index %[[ABUFFER]]{{\[}}%[[INS_IDX_3]]{{\]}}
201| // CHECK:     ttg.async_copy_global_to_local
202| // CHECK:     ttg.memdesc_index %[[BBUFFER]]{{\[}}%[[INS_IDX_3]]{{\]}}
203| // CHECK:     ttg.async_copy_global_to_local
204| // CHECK:   scf.yield {{.*}}, %[[INS_IDX_3]], %[[EXT_IDX_3]]
205| // CHECK:   ttg.async_wait {num = 0 : i32}
206| // CHECK    scf.yield
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 207-224
```mlir
207| 
208| //   AMD-LABEL:  tt.func @matmul_loop_nested
209| //         AMD:  scf.for
210| // AMD-COUNT-2:  ttg.local_alloc
211| // AMD-COUNT-2:  tt.load
212| //         AMD:  %[[SUBVIEW0:.*]] = ttg.memdesc_index
213| //         AMD:  ttg.local_store %{{.+}}, %[[SUBVIEW0]]
214| //         AMD:  %[[SUBVIEW1:.*]] = ttg.memdesc_index
215| //         AMD:  ttg.local_store %{{.+}}, %[[SUBVIEW1]]
216| //         AMD:  %[[FOR:.*]]:6 = scf.for
217| // AMD-COUNT-2:    tt.addptr
218| //         AMD:    tt.load
219| //         AMD:    ttg.local_load
220| //         AMD:    tt.load
221| //         AMD:    ttg.local_load
222| //         AMD:    tt.dot
223| //         AMD:    %[[SUBVIEW0:.*]] = ttg.memdesc_index
224| //         AMD:    ttg.local_store %{{.+}}, %[[SUBVIEW0]]
```
**EN:** This function-oriented block defines or enters `matmul_loop_nested`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `matmul_loop_nested` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 225-235
```mlir
225| //         AMD:    %[[SUBVIEW1:.*]] = ttg.memdesc_index
226| //         AMD:    ttg.local_store %{{.+}}, %[[SUBVIEW1]]
227| //         AMD:    scf.yield
228| // AMD-COUNT-2:  ttg.local_load
229| //         AMD:  %[[IF1:.*]] = scf.if
230| //         AMD:  %[[DOT1:.*]] = tt.dot
231| //         AMD:  scf.yield %[[DOT1]]
232| //         AMD:  %[[SEL1:.*]] = arith.select %{{.*}}, %[[IF1]], %[[FOR]]#2
233| // AMD-COUNT-2:  ttg.local_dealloc
234| //         AMD:  scf.yield %[[SEL1]]
235| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 236-237
```mlir
236| // AMD_3_STAGES-LABEL: tt.func @matmul_loop_nested
237| 
```
**EN:** This function-oriented block defines or enters `matmul_loop_nested`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `matmul_loop_nested` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 238-255
```mlir
238| tt.func @matmul_loop_nested(%lb : index, %ub : index, %step : index,
239|                          %A : !tt.ptr<f16> {tt.divisibility = 16 : i32},
240|                          %B : !tt.ptr<f16> {tt.divisibility = 16 : i32}) -> tensor<128x128xf32, #C>{
241| 
242|   %c_start = arith.constant dense<0.00e+00> : tensor<128x128xf32, #C>
243|   %loop1:1 = scf.for %iv0 = %lb to %ub step %step iter_args(%c_init = %c_start) -> (tensor<128x128xf32, #C>) {
244|     // A ptrs
245|     %a_ptr_splat = tt.splat %A : !tt.ptr<f16> -> tensor<128x32x!tt.ptr<f16>, #AL>
246|     %a_tmp0 = tt.make_range {end = 32: i32, start = 0: i32} : tensor<32xi32, #ALs0>
247|     %a_tmp1 = tt.expand_dims %a_tmp0 {axis = 0 : i32} : tensor<32xi32, #ALs0> -> tensor<1x32xi32, #AL>
248|     %a_offs = tt.broadcast %a_tmp1 : tensor<1x32xi32, #AL> -> tensor<128x32xi32, #AL>
249|     %a_ptr_init = tt.addptr %a_ptr_splat, %a_offs : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xi32, #AL>
250|     // B ptrs
251|     %b_ptr_splat = tt.splat %B : !tt.ptr<f16> -> tensor<32x128x!tt.ptr<f16>, #BL>
252|     %b_tmp0 = tt.make_range {end = 128: i32, start = 0: i32} : tensor<128xi32, #BLs0>
253|     %b_tmp1 = tt.expand_dims %b_tmp0 {axis = 0 : i32} : tensor<128xi32, #BLs0> -> tensor<1x128xi32, #BL>
254|     %b_offs = tt.broadcast %b_tmp1 : tensor<1x128xi32, #BL> -> tensor<32x128xi32, #BL>
255|     %b_ptr_init = tt.addptr %b_ptr_splat, %b_offs : tensor<32x128x!tt.ptr<f16>, #BL>, tensor<32x128xi32, #BL>
```
**EN:** This function-oriented block defines or enters `matmul_loop_nested`. Within it, the test exercises tt.func, broadcasted scalars or pointers, lane/block index ranges, shape expansion, tensor broadcasting, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `matmul_loop_nested` 为核心。测试在其中演示 tt.func、广播后的标量或指针、lane/block 索引范围、形状扩展、张量广播，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 256-273
```mlir
256| 
257|     %a_mask = arith.constant dense<true> : tensor<128x32xi1, #AL>
258|     %a_other = arith.constant dense<0.00e+00> : tensor<128x32xf16, #AL>
259|     %b_mask = arith.constant dense<true> : tensor<32x128xi1, #BL>
260|     %b_other = arith.constant dense<0.00e+00> : tensor<32x128xf16, #BL>
261| 
262|     %a_off = arith.constant dense<4> : tensor<128x32xi32, #AL>
263|     %b_off = arith.constant dense<4> : tensor<32x128xi32, #BL>
264| 
265|     %loop2:3 = scf.for %iv = %lb to %ub step %step iter_args(%a_ptr = %a_ptr_init, %b_ptr = %b_ptr_init, %prev_c = %c_init) -> (tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>) {
266|       %a_ = tt.load %a_ptr, %a_mask, %a_other : tensor<128x32x!tt.ptr<f16>, #AL>
267|       %a = ttg.convert_layout %a_ : tensor<128x32xf16, #AL> -> tensor<128x32xf16, #A>
268|       %b_ = tt.load %b_ptr, %b_mask, %b_other : tensor<32x128x!tt.ptr<f16>, #BL>
269|       %b = ttg.convert_layout %b_ : tensor<32x128xf16, #BL> -> tensor<32x128xf16, #B>
270| 
271|       %c = tt.dot %a, %b, %prev_c : tensor<128x32xf16, #A> * tensor<32x128xf16, #B> -> tensor<128x128xf32, #C>
272| 
273|       %next_a_ptr = tt.addptr %a_ptr, %a_off : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xi32, #AL>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining constants, masked or vectorized loads, layout conversions, structured loops, dot-product or MMA-style math.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 常量、带掩码或向量化的加载、布局转换、结构化循环、点积或 MMA 风格计算。

### Lines 274-282
```mlir
274|       %next_b_ptr = tt.addptr %b_ptr, %b_off : tensor<32x128x!tt.ptr<f16>, #BL>, tensor<32x128xi32, #BL>
275|       scf.yield %next_a_ptr, %next_b_ptr, %c : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>
276|     }
277| 
278|     scf.yield %loop2#2 : tensor<128x128xf32, #C>
279|   }
280|   tt.return %loop1#0 : tensor<128x128xf32, #C>
281| }
282| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining loop/if yielded values, pointer arithmetic, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 循环/分支产出值、指针算术、tt.return。

### Lines 283-300
```mlir
283| // CHECK-LABEL: tt.func @matmul_loop_single_pipeline
284| // CHECK-DAG: %[[CONSTANT_NEG1:.*]] = arith.constant -1 : i32
285| // CHECK-DAG: %[[CONSTANT_0:.*]] = arith.constant 0 : i32
286| // CHECK-DAG: %[[CONSTANT_1:.*]] = arith.constant 1 : i32
287| // CHECK-DAG: %[[CONSTANT_2:.*]] = arith.constant 2 : i32
288| // CHECK: %[[BBUFFER:.*]] = ttg.local_alloc
289| // CHECK: ttg.memdesc_index %[[BBUFFER]]{{\[}}%[[CONSTANT_0]]{{\]}}
290| // CHECK: ttg.async_copy_global_to_local
291| // CHECK: ttg.memdesc_index %[[BBUFFER]]{{\[}}%[[CONSTANT_1]]{{\]}}
292| // CHECK: ttg.async_copy_global_to_local
293| // CHECK:   scf.for {{.*}} iter_args({{.*}}, %[[INS_IDX:.*]] = %[[CONSTANT_1]], %[[EXT_IDX:.*]] = %[[CONSTANT_NEG1]]
294| // CHECK:     %[[EXT_IDX_2:.*]] = arith.addi %[[EXT_IDX]], %[[CONSTANT_1]] : i32
295| // CHECK:     %[[CMP_EXT:.*]] = arith.cmpi sge, %[[EXT_IDX_2]], %[[CONSTANT_2]]
296| // CHECK:     %[[EXT_IDX_3:.*]] = arith.select %[[CMP_EXT]], %[[CONSTANT_0]], %[[EXT_IDX_2]]
297| // CHECK:     ttg.async_wait {{.*}} {num = 1 : i32}
298| // CHECK:     %[[B0:.*]] = ttg.memdesc_index %[[BBUFFER]]{{\[}}%[[EXT_IDX_3]]{{\]}}
299| // CHECK:     %[[arg_b0_dot_op:.*]] = ttg.local_load %[[B0]]
300| // CHECK:     tt.dot {{.*}}, %[[arg_b0_dot_op]], {{.*}}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tt.func @matmul_loop_single_pipeline anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tt.func @matmul_loop_single_pipeline 这样的标签用于锚定匹配范围。

### Lines 301-307
```mlir
301| // CHECK-DAG: %[[INS_IDX_2:.*]] = arith.addi %[[INS_IDX]], %[[CONSTANT_1]] : i32
302| // CHECK-DAG: %[[CMP_INS:.*]] = arith.cmpi sge, %[[INS_IDX_2]], %[[CONSTANT_2]]
303| // CHECK-DAG: %[[INS_IDX_3:.*]] = arith.select %[[CMP_INS]], %[[CONSTANT_0]], %[[INS_IDX_2]]
304| // CHECK:     ttg.memdesc_index %[[BBUFFER]]{{\[}}%[[INS_IDX_3]]{{\]}}
305| // CHECK:     ttg.async_copy_global_to_local
306| // CHECK:   scf.yield {{.*}}, %[[INS_IDX_3]], %[[EXT_IDX_3]]
307| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 308-325
```mlir
308| // AMD-LABEL:  tt.func @matmul_loop_single_pipeline
309| //       AMD:   %[[LOAD_10:.*]] = tt.load %{{.*}}
310| //       AMD:   %[[CONVERT_LAYOUT_11:.*]] = ttg.convert_layout %[[LOAD_10]]
311| //       AMD:   %[[LOCAL_ALLOC_12:.*]] = ttg.local_alloc
312| //       AMD:   %[[CMPI_13:.*]] = arith.cmpi slt, %{{.*}}, %{{.*}}
313| //       AMD:   %[[SPLAT_14:.*]] = tt.splat %[[CMPI_13]]
314| //       AMD:   %[[LOAD_15:.*]] = tt.load %{{.*}}, %[[SPLAT_14]], %{{.*}}
315| //       AMD:   %[[MEMDESC_SUBVIEW_16:.*]] = ttg.memdesc_index %[[LOCAL_ALLOC_12]]{{\[}}%{{.*}}{{\]}}
316| //       AMD:   ttg.local_store %[[LOAD_15]], %[[MEMDESC_SUBVIEW_16]]
317| //       AMD:   %[[SUBI_17:.*]] = arith.subi %{{.*}}, %{{.*}}
318| //       AMD:   %{{.*}}:4 = scf.for %[[ARG5:.*]] = %{{.*}} to %[[SUBI_17]] step %{{.*}} iter_args(%[[ARG6:.*]] = %{{.*}}, %[[ARG7:.*]] = %{{.*}}, %[[ARG8:.*]] = %{{.*}}, %[[ARG9:.*]] = %[[MEMDESC_SUBVIEW_16]])
319| //       AMD:       %[[ADDPTR_32:.*]] = tt.addptr %[[ARG6]], %{{.*}}
320| //       AMD:       %[[LOAD_33:.*]] = tt.load %[[ADDPTR_32]]
321| //       AMD:       %[[LOCAL_LOAD_30:.*]] = ttg.local_load %[[ARG9]]
322| //       AMD:       %[[DOT_31:.*]] = tt.dot %[[CONVERT_LAYOUT_11]], %[[LOCAL_LOAD_30]], %[[ARG7]]
323| //       AMD:       %[[ADDI_34:.*]] = arith.addi %[[ARG8]], %{{.*}}
324| //       AMD:       %[[CMPI_35:.*]] = arith.cmpi slt, %[[ADDI_34]], %{{.*}}
325| //       AMD:       %[[SELECT_36:.*]] = arith.select %[[CMPI_35]], %[[ADDI_34]], %{{.*}}
```
**EN:** This function-oriented block defines or enters `matmul_loop_single_pipeline`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `matmul_loop_single_pipeline` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 326-330
```mlir
326| //       AMD:       %[[MEMDESC_SUBVIEW_37:.*]] = ttg.memdesc_index %[[LOCAL_ALLOC_12]]{{\[}}%[[SELECT_36]]{{\]}}
327| //       AMD:       ttg.local_store %[[LOAD_33]], %[[MEMDESC_SUBVIEW_37]]
328| //       AMD:       scf.yield %[[ADDPTR_32]], %[[DOT_31]], %[[SELECT_36]], %[[MEMDESC_SUBVIEW_37]]
329| //       AMD:  ttg.local_dealloc %[[LOCAL_ALLOC_12]]
330| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 331-346
```mlir
331| // AMD_3_STAGES-LABEL: tt.func @matmul_loop_single_pipeline
332| //       AMD_3_STAGES:   ttg.local_alloc
333| //       AMD_3_STAGES:   tt.load
334| //       AMD_3_STAGES:   ttg.local_store
335| //       AMD_3_STAGES:   tt.load
336| //       AMD_3_STAGES:   ttg.local_store
337| //       AMD_3_STAGES:   scf.for
338| //       AMD_3_STAGES:     tt.load
339| //       AMD_3_STAGES:     ttg.local_load
340| //       AMD_3_STAGES:     tt.dot
341| //       AMD_3_STAGES:     ttg.local_store
342| //       AMD_3_STAGES:     scf.yield
343| //       AMD_3_STAGES:   tt.dot
344| //       AMD_3_STAGES:   tt.dot
345| //       AMD_3_STAGES:   tt.return
346| 
```
**EN:** This function-oriented block defines or enters `matmul_loop_single_pipeline`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `matmul_loop_single_pipeline` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 347-364
```mlir
347| tt.func @matmul_loop_single_pipeline(%lb : index, %ub : index, %step : index,
348|                                   %A : !tt.ptr<f16> {tt.divisibility = 16 : i32},
349|                                   %B : !tt.ptr<f16> {tt.divisibility = 16 : i32}) -> tensor<128x128xf32, #C> {
350|   // A ptrs
351|   %a_ptr_splat = tt.splat %A : !tt.ptr<f16> -> tensor<128x32x!tt.ptr<f16>, #AL>
352|   %a_tmp0 = tt.make_range {end = 32: i32, start = 0: i32} : tensor<32xi32, #ALs0>
353|   %a_tmp1 = tt.expand_dims %a_tmp0 {axis = 0 : i32} : tensor<32xi32, #ALs0> -> tensor<1x32xi32, #AL>
354|   %a_offs = tt.broadcast %a_tmp1 : tensor<1x32xi32, #AL> -> tensor<128x32xi32, #AL>
355|   %a_ptr_init = tt.addptr %a_ptr_splat, %a_offs : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xi32, #AL>
356|   // B ptrs
357|   %b_ptr_splat = tt.splat %B : !tt.ptr<f16> -> tensor<32x128x!tt.ptr<f16>, #BL>
358|   %b_tmp0 = tt.make_range {end = 128: i32, start = 0: i32} : tensor<128xi32, #BLs0>
359|   %b_tmp1 = tt.expand_dims %b_tmp0 {axis = 0 : i32} : tensor<128xi32, #BLs0> -> tensor<1x128xi32, #BL>
360|   %b_offs = tt.broadcast %b_tmp1 : tensor<1x128xi32, #BL> -> tensor<32x128xi32, #BL>
361|   %b_ptr_init = tt.addptr %b_ptr_splat, %b_offs : tensor<32x128x!tt.ptr<f16>, #BL>, tensor<32x128xi32, #BL>
362| 
363|   %a_mask = arith.constant dense<true> : tensor<128x32xi1, #AL>
364|   %a_other = arith.constant dense<0.00e+00> : tensor<128x32xf16, #AL>
```
**EN:** This function-oriented block defines or enters `matmul_loop_single_pipeline`. Within it, the test exercises tt.func, broadcasted scalars or pointers, lane/block index ranges, shape expansion, tensor broadcasting, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `matmul_loop_single_pipeline` 为核心。测试在其中演示 tt.func、广播后的标量或指针、lane/block 索引范围、形状扩展、张量广播，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 365-382
```mlir
365| 
366|   %a_ = tt.load %a_ptr_init, %a_mask, %a_other : tensor<128x32x!tt.ptr<f16>, #AL>
367|   %a = ttg.convert_layout %a_ : tensor<128x32xf16, #AL> -> tensor<128x32xf16, #A>
368| 
369|   %b_mask = arith.constant dense<true> : tensor<32x128xi1, #BL>
370|   %b_other = arith.constant dense<0.00e+00> : tensor<32x128xf16, #BL>
371|   %c_init = arith.constant dense<0.00e+00> : tensor<128x128xf32, #C>
372| 
373|   %b_off = arith.constant dense<4> : tensor<32x128xi32, #BL>
374| 
375|   %loop:2 = scf.for %iv = %lb to %ub step %step iter_args(%b_ptr = %b_ptr_init, %prev_c = %c_init) -> (tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>) {
376|     %b_ = tt.load %b_ptr, %b_mask, %b_other : tensor<32x128x!tt.ptr<f16>, #BL>
377|     %b = ttg.convert_layout %b_ : tensor<32x128xf16, #BL> -> tensor<32x128xf16, #B>
378|     %c = tt.dot %a, %b, %prev_c : tensor<128x32xf16, #A> * tensor<32x128xf16, #B> -> tensor<128x128xf32, #C>
379|     %next_b_ptr = tt.addptr %b_ptr, %b_off : tensor<32x128x!tt.ptr<f16>, #BL>, tensor<32x128xi32, #BL>
380|     scf.yield %next_b_ptr, %c : tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>
381|   }
382|   tt.return %loop#1 : tensor<128x128xf32, #C>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining constants, masked or vectorized loads, layout conversions, structured loops, dot-product or MMA-style math.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 常量、带掩码或向量化的加载、布局转换、结构化循环、点积或 MMA 风格计算。

### Lines 383-384
```mlir
383| }
384| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 385-402
```mlir
385| // CHECK-LABEL: tt.func @indirect_bmm_scalar
386| // CHECK: ttg.async_copy_global_to_local
387| // CHECK: ttg.async_copy_global_to_local
388| // CHECK: ttg.async_copy_global_to_local
389| // CHECK: ttg.async_copy_global_to_local
390| // CHECK: ttg.async_commit_group
391| // CHECK: scf.for
392| // CHECK: ttg.async_wait {{.*}} {num = 1 : i32}
393| // CHECK: %[[NEXT_BUFFER_1:.*]] = tt.addptr %{{.*}}, {{.*}}
394| // CHECK: ttg.async_copy_global_to_local %[[NEXT_BUFFER_1]]
395| // CHECK: ttg.async_wait {{.*}} {num = 1 : i32}
396| // CHECK: %[[IND_BUFFER_0_T:.*]] = ttg.local_load
397| // CHECK: %[[IND_BUFFER_0:.*]] = tt.unsplat %[[IND_BUFFER_0_T]] : tensor<1xi64
398| // CHECK: %[[IND_BUFFER_1:.*]] = arith.muli {{.*}}, %[[IND_BUFFER_0]]
399| // CHECK: %[[IND_BUFFER_2:.*]] = tt.splat %[[IND_BUFFER_1]]
400| // CHECK: %[[NEXT_BUFFER_0:.*]] = tt.addptr {{.*}}, %[[IND_BUFFER_2]]
401| // CHECK: ttg.async_copy_global_to_local %[[NEXT_BUFFER_0]]
402| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tt.func @indirect_bmm_scalar anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tt.func @indirect_bmm_scalar 这样的标签用于锚定匹配范围。

### Lines 403-420
```mlir
403| // AMD-LABEL:   tt.func @indirect_bmm_scalar
404| //       AMD:     %[[LOCAL_ALLOC_0:.*]] = ttg.local_alloc
405| //       AMD:     %[[LOCAL_ALLOC_1:.*]] = ttg.local_alloc
406| //       AMD:     %[[CMPI_2:.*]] = arith.cmpi sgt, %{{.*}}, %{{.*}}
407| //       AMD:     %[[LOAD_5:.*]] = tt.load %{{.*}}, %[[CMPI_2]] {amd.pipeliner_part = "prologue"}
408| //       AMD:     %[[SPLAT_3:.*]] = tt.splat %[[CMPI_2]]
409| //       AMD:     %[[LOAD_4:.*]] = tt.load %{{.*}}, %[[SPLAT_3]] {amd.pipeliner_part = "prologue"}
410| //       AMD:     %[[MULI_6:.*]] = arith.muli %{{.*}}, %[[LOAD_5]]
411| //       AMD:     %[[SPLAT_7:.*]] = tt.splat %[[MULI_6]]
412| //       AMD:     %[[ADDPTR_8:.*]] = tt.addptr %{{.*}}, %[[SPLAT_7]]
413| //       AMD:     %[[SPLAT_9:.*]] = tt.splat %[[CMPI_2]]
414| //       AMD:     %[[LOAD_10:.*]] = tt.load %[[ADDPTR_8]], %[[SPLAT_9]] {amd.pipeliner_part = "prologue"}
415| //       AMD:     %[[MEMDESC_SUBVIEW_11:.*]] = ttg.memdesc_index %[[LOCAL_ALLOC_0]]{{\[}}%{{.*}}{{\]}}
416| //       AMD:     ttg.local_store %[[LOAD_4]], %[[MEMDESC_SUBVIEW_11]]
417| //       AMD:     %[[MEMDESC_SUBVIEW_12:.*]] = ttg.memdesc_index %[[LOCAL_ALLOC_1]]{{\[}}%{{.*}}{{\]}}
418| //       AMD:     ttg.local_store %[[LOAD_10]], %[[MEMDESC_SUBVIEW_12]]
419| //       AMD:     %[[SUBI_26:.*]] = arith.subi %{{.*}}, %{{.*}}
420| //       AMD:     %{{.*}}:7 = scf.for %[[ARG6:.*]] = %{{.*}} to %[[SUBI_26]] step %{{.*}} iter_args(%[[ARG7:.*]] = %{{.*}}, %[[ARG8:.*]] = %{{.*}}, %[[ARG9:.*]] = %{{.*}}, %[[ARG10:.*]] = %{{.*}}, %[[ARG11:.*]] = %[[MEMDESC_SUBVIEW_11]], %[[ARG12:.*]] = %{{.*}}, %[[ARG13:.*]] = %[[MEMDESC_SUBVIEW_12]])
```
**EN:** This function-oriented block defines or enters `indirect_bmm_scalar`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `indirect_bmm_scalar` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 421-438
```mlir
421| //       AMD:       %[[ADDPTR_38:.*]] = tt.addptr %[[ARG8]], %{{.*}}
422| //       AMD:       %[[ADDPTR_39:.*]] = tt.addptr %[[ARG9]], %{{.*}}
423| //       AMD:       %[[LOAD_40:.*]] = tt.load %[[ADDPTR_38]]
424| //       AMD:       %[[LOCAL_LOAD_41:.*]] = ttg.local_load %[[ARG11]]
425| //       AMD:       %[[LOAD_42:.*]] = tt.load %[[ADDPTR_39]]
426| //       AMD:       %[[MULI_43:.*]] = arith.muli %{{.*}}, %[[ARG12]]
427| //       AMD:       %[[SPLAT_44:.*]] = tt.splat %[[MULI_43]]
428| //       AMD:       %[[ADDPTR_45:.*]] = tt.addptr %{{.*}}, %[[SPLAT_44]]
429| //       AMD:       %[[LOAD_46:.*]] = tt.load %[[ADDPTR_45]]
430| //       AMD:       %[[LOCAL_LOAD_47:.*]] = ttg.local_load %[[ARG13]]
431| //       AMD:       %[[DOT_48:.*]] = tt.dot %[[LOCAL_LOAD_41]], %[[LOCAL_LOAD_47]], %[[ARG7]]
432| //       AMD:       %[[ADDI_49:.*]] = arith.addi %[[ARG10]], %{{.*}}
433| //       AMD:       %[[CMPI_50:.*]] = arith.cmpi slt, %[[ADDI_49]], %{{.*}}
434| //       AMD:       %[[SELECT_51:.*]] = arith.select %[[CMPI_50]], %[[ADDI_49]], %{{.*}}
435| //       AMD:       %[[MEMDESC_SUBVIEW_52:.*]] = ttg.memdesc_index %[[LOCAL_ALLOC_0]]{{\[}}%[[SELECT_51]]{{\]}}
436| //       AMD:       ttg.local_store %[[LOAD_40]], %[[MEMDESC_SUBVIEW_52]]
437| //       AMD:       %[[MEMDESC_SUBVIEW_53:.*]] = ttg.memdesc_index %[[LOCAL_ALLOC_1]]{{\[}}%[[SELECT_51]]{{\]}}
438| //       AMD:       ttg.local_store %[[LOAD_46]], %[[MEMDESC_SUBVIEW_53]]
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 439-456
```mlir
439| //       AMD:       scf.yield %[[DOT_48]], %[[ADDPTR_38]], %[[ADDPTR_39]], %[[SELECT_51]], %[[MEMDESC_SUBVIEW_52]], %[[LOAD_42]], %[[MEMDESC_SUBVIEW_53]]
440| //       AMD:     } {tt.num_stages = 3
441| //       AMD:     %[[CMPI_28:.*]] = arith.cmpi sge, %{{.*}}, %{{.*}}
442| //       AMD:     %[[CMPI_29:.*]] = arith.cmpi sge, %{{.*}}, %{{.*}}
443| //       AMD:     %[[LOCAL_LOAD_30:.*]] = ttg.local_load %{{.*}}#4
444| //       AMD:     %[[LOCAL_LOAD_31:.*]] = ttg.local_load %{{.*}}#6
445| //       AMD:     %[[IF_32:.*]] = scf.if %[[CMPI_28]]
446| //       AMD:       %[[DOT_38:.*]] = tt.dot %[[LOCAL_LOAD_30]], %[[LOCAL_LOAD_31]], %{{.*}}#0
447| //       AMD:       scf.yield %[[DOT_38]]
448| //       AMD:     } else {
449| //       AMD:       scf.yield %{{.*}}#0
450| //       AMD:     }
451| //       AMD:     %[[SELECT_33:.*]] = arith.select %[[CMPI_28]], %[[IF_32]], %{{.*}}#0
452| //       AMD:     %[[LOCAL_LOAD_34:.*]] = ttg.local_load %{{.*}}
453| //       AMD:     %[[LOCAL_LOAD_35:.*]] = ttg.local_load %{{.*}}
454| //       AMD:     %[[IF_36:.*]] = scf.if %[[CMPI_29]]
455| //       AMD:       %[[DOT_38:.*]] = tt.dot %[[LOCAL_LOAD_34]], %[[LOCAL_LOAD_35]], %[[SELECT_33]]
456| //       AMD:       scf.yield %[[DOT_38]]
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 457-462
```mlir
457| //       AMD:     } else {
458| //       AMD:       scf.yield %[[SELECT_33]]
459| //       AMD:     }
460| //       AMD:     %[[SELECT_37:.*]] = arith.select %[[CMPI_29]], %[[IF_36]], %[[SELECT_33]]
461| //       AMD-DAG:     ttg.local_dealloc %[[LOCAL_ALLOC_0]]
462| //       AMD-DAG:     ttg.local_dealloc %[[LOCAL_ALLOC_1]]
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 463-480
```mlir
463| tt.func @indirect_bmm_scalar(%77: i64 {tt.divisibility=16: i32},
464|                    %76: index,
465|                    %49: tensor<16x16x!tt.ptr<f16>, #AL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 2]> : tensor<2xi32>},
466|                    %75: !tt.ptr<i64>,
467|                    %78: tensor<16x16xi32, #AL> {tt.constancy = dense<[16, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>},
468|                    %60: tensor<16x16x!tt.ptr<f16>, #BL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>}) -> tensor<16x16xf32, #C> {
469|   %cst = arith.constant dense<0.000000e+00> : tensor<16x16xf32, #C>
470|   %c4_i32 = arith.constant 4 : i32
471|   %c1 = arith.constant 1 : index
472|   %c0 = arith.constant 0 : index
473|   %c0_i64 = arith.constant 0 : i64
474|   %c1_i32 = arith.constant 1 : i32
475|   %79:3 = scf.for %arg18 = %c0 to %76 step %c1 iter_args(%arg19 = %cst, %arg20 = %49, %arg21 = %75) -> (tensor<16x16xf32, #C>, tensor<16x16x!tt.ptr<f16>, #AL>, !tt.ptr<i64>) {
476|     %82 = tt.load %arg20 : tensor<16x16x!tt.ptr<f16>, #AL>
477|     %83 = tt.load %arg21 : !tt.ptr<i64>
478|     %84 = arith.muli %77, %83 : i64
479|     %85 = tt.splat %84 : i64 -> tensor<16x16xi64, #BL>
480|     %86 = tt.addptr %60, %85 : tensor<16x16x!tt.ptr<f16>, #BL>, tensor<16x16xi64, #BL>
```
**EN:** This function-oriented block defines or enters `indirect_bmm_scalar`. Within it, the test exercises constants, tt.func, masked or vectorized loads, structured loops, integer multiplications, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `indirect_bmm_scalar` 为核心。测试在其中演示 常量、tt.func、带掩码或向量化的加载、结构化循环、整数乘法，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 481-491
```mlir
481|     %87 = tt.load %86 : tensor<16x16x!tt.ptr<f16>, #BL>
482|     %88 = ttg.convert_layout %82 : tensor<16x16xf16, #AL> -> tensor<16x16xf16, #A>
483|     %89 = ttg.convert_layout %87 : tensor<16x16xf16, #BL> -> tensor<16x16xf16, #B>
484|     %90 = tt.dot %88, %89, %arg19 : tensor<16x16xf16, #A> * tensor<16x16xf16, #B> -> tensor<16x16xf32, #C>
485|     %91 = tt.addptr %arg20, %78 : tensor<16x16x!tt.ptr<f16>, #AL>, tensor<16x16xi32, #AL>
486|     %92 = tt.addptr %arg21, %c1_i32 : !tt.ptr<i64>, i32
487|     scf.yield %90, %91, %92 : tensor<16x16xf32, #C>, tensor<16x16x!tt.ptr<f16>, #AL>, !tt.ptr<i64>
488|   } {tt.num_stages = 3 : i32}
489|   tt.return %79#0 : tensor<16x16xf32, #C>
490| }
491| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining layout conversions, pointer arithmetic, masked or vectorized loads, dot-product or MMA-style math, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 布局转换、指针算术、带掩码或向量化的加载、点积或 MMA 风格计算、循环/分支产出值。

### Lines 492-508
```mlir
492| // CHECK-LABEL: tt.func @indirect_bmm_scalar_dist_one
493| // CHECK: ttg.async_copy_global_to_local
494| // CHECK: ttg.async_copy_global_to_local
495| // CHECK: ttg.async_copy_global_to_local
496| // CHECK: ttg.async_copy_global_to_local
497| // CHECK: ttg.async_commit_group
498| // CHECK: scf.for %{{.*}} iter_args(%{{[^,]*}}, %{{[^,]*}}, %{{[^,]*}}, %[[IND_BUFFER_PREV:[^,]*]] = {{[^,]*}}
499| // CHECK: ttg.async_wait {{.*}} {num = 2 : i32}
500| // CHECK: %[[NEXT_BUFFER_1:.*]] = tt.addptr %{{.*}}, {{.*}}
501| // CHECK: ttg.async_copy_global_to_local %[[NEXT_BUFFER_1]]
502| // CHECK: %[[IND_BUFFER_0:.*]] = tt.load %{{.*}}, {{.*}}
503| // CHECK: %[[IND_BUFFER_1:.*]] = arith.muli {{.*}}, %[[IND_BUFFER_PREV]]
504| // CHECK: %[[IND_BUFFER_2:.*]] = tt.splat %[[IND_BUFFER_1]]
505| // CHECK: %[[NEXT_BUFFER_0:.*]] = tt.addptr {{.*}}, %[[IND_BUFFER_2]]
506| // CHECK: ttg.async_copy_global_to_local %[[NEXT_BUFFER_0]]
507| // CHECK: scf.yield {{.*}}, {{.*}}, {{.*}}, %[[IND_BUFFER_0]]
508| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tt.func @indirect_bmm_scalar_dist_one anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tt.func @indirect_bmm_scalar_dist_one 这样的标签用于锚定匹配范围。

### Lines 509-516
```mlir
509| // AMD-LABEL:  tt.func @indirect_bmm_scalar_dist_one
510| // AMD-COUNT-4:  tt.load
511| //       AMD:  scf.for
512| //       AMD:    tt.load
513| //       AMD:    tt.dot
514| //       AMD:    ttg.local_store
515| //       AMD:    scf.yield
516| 
```
**EN:** This function-oriented block defines or enters `indirect_bmm_scalar_dist_one`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `indirect_bmm_scalar_dist_one` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 517-518
```mlir
517| // AMD_3_STAGES-LABEL: tt.func @indirect_bmm_scalar_dist_one
518| 
```
**EN:** This function-oriented block defines or enters `indirect_bmm_scalar_dist_one`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `indirect_bmm_scalar_dist_one` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 519-536
```mlir
519| tt.func @indirect_bmm_scalar_dist_one(%77: i64 {tt.divisibility=16: i32},
520|                    %76: index,
521|                    %49: tensor<16x16x!tt.ptr<f16>, #AL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 2]> : tensor<2xi32>},
522|                    %75: !tt.ptr<i64>,
523|                    %78: tensor<16x16xi32, #AL> {tt.constancy = dense<[16, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>},
524|                    %60: tensor<16x16x!tt.ptr<f16>, #BL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>}) -> tensor<16x16xf32, #C> {
525|   %cst = arith.constant dense<0.000000e+00> : tensor<16x16xf32, #C>
526|   %c4_i32 = arith.constant 4 : i32
527|   %c1 = arith.constant 1 : index
528|   %c0 = arith.constant 0 : index
529|   %c0_i64 = arith.constant 0 : i64
530|   %c1_i32 = arith.constant 1 : i32
531|   %50 = tt.load %75 : !tt.ptr<i64>
532|   %51 = tt.addptr %75, %c1_i32 : !tt.ptr<i64>, i32
533|   %79:4 = scf.for %arg18 = %c0 to %76 step %c1 iter_args(%arg19 = %cst, %arg20 = %49, %arg21 = %51, %arg22 = %50) -> (tensor<16x16xf32, #C>, tensor<16x16x!tt.ptr<f16>, #AL>, !tt.ptr<i64>, i64) {
534|     %82 = tt.load %arg20 : tensor<16x16x!tt.ptr<f16>, #AL>
535|     %83 = tt.load %arg21 : !tt.ptr<i64>
536|     %84 = arith.muli %77, %arg22 : i64
```
**EN:** This function-oriented block defines or enters `indirect_bmm_scalar_dist_one`. Within it, the test exercises constants, masked or vectorized loads, tt.func, pointer arithmetic, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `indirect_bmm_scalar_dist_one` 为核心。测试在其中演示 常量、带掩码或向量化的加载、tt.func、指针算术、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 537-549
```mlir
537|     %85 = tt.splat %84 : i64 -> tensor<16x16xi64, #BL>
538|     %86 = tt.addptr %60, %85 : tensor<16x16x!tt.ptr<f16>, #BL>, tensor<16x16xi64, #BL>
539|     %87 = tt.load %86 : tensor<16x16x!tt.ptr<f16>, #BL>
540|     %88 = ttg.convert_layout %82 : tensor<16x16xf16, #AL> -> tensor<16x16xf16, #A>
541|     %89 = ttg.convert_layout %87 : tensor<16x16xf16, #BL> -> tensor<16x16xf16, #B>
542|     %90 = tt.dot %88, %89, %arg19 : tensor<16x16xf16, #A> * tensor<16x16xf16, #B> -> tensor<16x16xf32, #C>
543|     %91 = tt.addptr %arg20, %78 : tensor<16x16x!tt.ptr<f16>, #AL>, tensor<16x16xi32, #AL>
544|     %92 = tt.addptr %arg21, %c1_i32 : !tt.ptr<i64>, i32
545|     scf.yield %90, %91, %92, %83 : tensor<16x16xf32, #C>, tensor<16x16x!tt.ptr<f16>, #AL>, !tt.ptr<i64>, i64
546|   }
547|   tt.return %79#0 : tensor<16x16xf32, #C>
548| }
549| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining pointer arithmetic, layout conversions, broadcasted scalars or pointers, masked or vectorized loads, dot-product or MMA-style math.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 指针算术、布局转换、广播后的标量或指针、带掩码或向量化的加载、点积或 MMA 风格计算。

### Lines 550-567
```mlir
550| // CHECK-LABEL: tt.func @indirect_bmm_vector
551| // CHECK: ttg.async_copy_global_to_local
552| // CHECK: ttg.async_copy_global_to_local
553| // CHECK: ttg.async_copy_global_to_local
554| // CHECK: ttg.async_copy_global_to_local
555| // CHECK: ttg.async_commit_group
556| // CHECK: scf.for
557| // CHECK: ttg.async_wait {{.*}} {num = 1 : i32}
558| // CHECK: tt.dot
559| // CHECK: %[[NEXT_BUFFER_1:.*]] = tt.addptr %{{.*}}, {{.*}}
560| // CHECK: ttg.async_copy_global_to_local %[[NEXT_BUFFER_1]]
561| // CHECK-DAG: %[[IND_BUFFER_WAIT_TOKEN:.*]] = ttg.async_wait {{.*}} {num = 1 : i32}
562| // CHECK-DAG: %[[IND_BUFFER_0:.*]] = ttg.memdesc_index
563| // CHECK: %[[IND_BUFFER_1:.*]] = ttg.local_load %[[IND_BUFFER_0]] token %[[IND_BUFFER_WAIT_TOKEN]]
564| // CHECK: %[[IND_BUFFER_2:.*]] = tt.expand_dims %[[IND_BUFFER_1]] {axis = 1 : i32}
565| // CHECK: %[[IND_BUFFER_3:.*]] = tt.broadcast %[[IND_BUFFER_2]]
566| // CHECK: %[[IND_BUFFER_4:.*]] = arith.muli {{.*}}, %[[IND_BUFFER_3]]
567| // CHECK: %[[NEXT_BUFFER_0:.*]] = tt.addptr {{.*}}, %[[IND_BUFFER_4]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tt.func @indirect_bmm_vector anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tt.func @indirect_bmm_vector 这样的标签用于锚定匹配范围。

### Lines 568-570
```mlir
568| // CHECK: ttg.async_copy_global_to_local %[[NEXT_BUFFER_0]]
569| // CHECK: scf.yield
570| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 571-588
```mlir
571| // AMD-LABEL:  tt.func @indirect_bmm_vector
572| //       AMD:   %[[LOCAL_ALLOC_0:.*]] = ttg.local_alloc
573| //       AMD:   %[[LOCAL_ALLOC_1:.*]] = ttg.local_alloc
574| //       AMD:   %[[CMPI_2:.*]] = arith.cmpi sgt, %{{.*}}, %{{.*}}
575| //       AMD:   %[[SPLAT_3:.*]] = tt.splat %[[CMPI_2]]
576| //       AMD:   %[[LOAD_4:.*]] = tt.load %{{.*}}, %[[SPLAT_3]]
577| //       AMD:   %[[CMPI_5:.*]] = arith.cmpi sgt, %{{.*}}, %{{.*}}
578| //       AMD:   %[[ADDPTR_6:.*]] = tt.addptr %{{.*}}, %{{.*}}
579| //       AMD:   %[[SPLAT_7:.*]] = tt.splat %[[CMPI_2]]
580| //       AMD:   %[[LOAD_8:.*]] = tt.load %{{.*}}, %[[SPLAT_7]]
581| //       AMD:   %[[SPLAT_9:.*]] = tt.splat %[[CMPI_5]]
582| //       AMD:   %[[LOAD_10:.*]] = tt.load %[[ADDPTR_6]], %[[SPLAT_9]]
583| //       AMD:   %[[EXPAND_DIMS_11:.*]] = tt.expand_dims %[[LOAD_4]] {axis = 1 : i32}
584| //       AMD:   %[[BROADCAST_12:.*]] = tt.broadcast %[[EXPAND_DIMS_11]]
585| //       AMD:   %[[MULI_13:.*]] = arith.muli %{{.*}}, %[[BROADCAST_12]]
586| //       AMD:   %[[ADDPTR_14:.*]] = tt.addptr %{{.*}}, %[[MULI_13]]
587| //       AMD:   %[[SPLAT_15:.*]] = tt.splat %[[CMPI_2]]
588| //       AMD:   %[[LOAD_16:.*]] = tt.load %[[ADDPTR_14]], %[[SPLAT_15]]
```
**EN:** This function-oriented block defines or enters `indirect_bmm_vector`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `indirect_bmm_vector` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 589-606
```mlir
589| //       AMD:   %[[MEMDESC_SUBVIEW_17:.*]] = ttg.memdesc_index %[[LOCAL_ALLOC_0]]{{\[}}%{{.*}}{{\]}}
590| //       AMD:   ttg.local_store %[[LOAD_8]], %[[MEMDESC_SUBVIEW_17]]
591| //       AMD:   %[[MEMDESC_SUBVIEW_18:.*]] = ttg.memdesc_index %[[LOCAL_ALLOC_1]]{{\[}}%{{.*}}{{\]}}
592| //       AMD:   ttg.local_store %[[LOAD_16]], %[[MEMDESC_SUBVIEW_18]]
593| //       AMD:   %[[SUBI_19:.*]] = arith.subi %{{.*}}, %{{.*}}
594| //       AMD:   %{{.*}}:7 = scf.for %[[ARG6:.*]] = %{{.*}} to %[[SUBI_19]] step %{{.*}} iter_args(%[[ARG7:.*]] = %{{.*}}, %[[ARG8:.*]] = %{{.*}}, %[[ARG9:.*]] = %[[ADDPTR_6]], %[[ARG10:.*]] = %{{.*}}, %[[ARG11:.*]] = %[[MEMDESC_SUBVIEW_17]], %[[ARG12:.*]] = %[[LOAD_10]], %[[ARG13:.*]] = %[[MEMDESC_SUBVIEW_18]])
595| //       AMD:     %[[ADDPTR_47:.*]] = tt.addptr %[[ARG8]], %{{.*}}
596| //       AMD:     %[[ADDPTR_48:.*]] = tt.addptr %[[ARG9]], %{{.*}}
597| //       AMD:     %[[LOAD_49:.*]] = tt.load %[[ADDPTR_47]]
598| //       AMD:     %[[LOCAL_LOAD_50:.*]] = ttg.local_load %[[ARG11]]
599| //       AMD:     %[[LOAD_51:.*]] = tt.load %[[ADDPTR_48]]
600| //       AMD:     %[[EXPAND_DIMS_52:.*]] = tt.expand_dims %[[ARG12]] {axis = 1 : i32}
601| //       AMD:     %[[BROADCAST_53:.*]] = tt.broadcast %[[EXPAND_DIMS_52]]
602| //       AMD:     %[[MULI_54:.*]] = arith.muli %{{.*}}, %[[BROADCAST_53]]
603| //       AMD:     %[[ADDPTR_55:.*]] = tt.addptr %{{.*}}, %[[MULI_54]]
604| //       AMD:     %[[LOAD_56:.*]] = tt.load %[[ADDPTR_55]]
605| //       AMD:     %[[LOCAL_LOAD_57:.*]] = ttg.local_load %[[ARG13]]
606| //       AMD:     %[[DOT_58:.*]] = tt.dot %[[LOCAL_LOAD_50]], %[[LOCAL_LOAD_57]], %[[ARG7]]
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 607-615
```mlir
607| //       AMD:     %[[ADDI_59:.*]] = arith.addi %[[ARG10]], %{{.*}}
608| //       AMD:     %[[CMPI_60:.*]] = arith.cmpi slt, %[[ADDI_59]], %{{.*}}
609| //       AMD:     %[[SELECT_61:.*]] = arith.select %[[CMPI_60]], %[[ADDI_59]], %{{.*}}
610| //       AMD:     %[[MEMDESC_SUBVIEW_62:.*]] = ttg.memdesc_index %[[LOCAL_ALLOC_0]]{{\[}}%[[SELECT_61]]{{\]}}
611| //       AMD:     ttg.local_store %[[LOAD_49]], %[[MEMDESC_SUBVIEW_62]]
612| //       AMD:     %[[MEMDESC_SUBVIEW_63:.*]] = ttg.memdesc_index %[[LOCAL_ALLOC_1]]{{\[}}%[[SELECT_61]]{{\]}}
613| //       AMD:     ttg.local_store %[[LOAD_56]], %[[MEMDESC_SUBVIEW_63]]
614| //       AMD:     scf.yield %[[DOT_58]], %[[ADDPTR_47]], %[[ADDPTR_48]], %[[SELECT_61]], %[[MEMDESC_SUBVIEW_62]], %[[LOAD_51]], %[[MEMDESC_SUBVIEW_63]]
615| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 616-617
```mlir
616| // AMD_3_STAGES-LABEL: tt.func @indirect_bmm_vector
617| 
```
**EN:** This function-oriented block defines or enters `indirect_bmm_vector`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `indirect_bmm_vector` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 618-635
```mlir
618| tt.func @indirect_bmm_vector(%77: tensor<16x16xi64, #BL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.constancy = dense<[16, 16]> : tensor<2xi32>},
619|                    %76: index,
620|                    %49: tensor<16x16x!tt.ptr<f16>, #AL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 2]> : tensor<2xi32>},
621|                    %75: tensor<16x!tt.ptr<i64>, #BLs1>,
622|                    %78: tensor<16x16xi32, #AL> {tt.constancy = dense<[16, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>},
623|                    %60: tensor<16x16x!tt.ptr<f16>, #BL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>}) -> tensor<16x16xf32, #C> {
624|   %cst = arith.constant dense<0.000000e+00> : tensor<16x16xf32, #C>
625|   %c4_i32 = arith.constant 4 : i32
626|   %c1 = arith.constant 1 : index
627|   %c0 = arith.constant 0 : index
628|   %c0_i64 = arith.constant 0 : i64
629|   %c1_i32 = arith.constant 1 : i32
630|   %c1_i32_splat = tt.splat %c1_i32 : i32 -> tensor<16xi32, #BLs1>
631|   %79:3 = scf.for %arg18 = %c0 to %76 step %c1 iter_args(%arg19 = %cst, %arg20 = %49, %arg21 = %75) -> (tensor<16x16xf32, #C>, tensor<16x16x!tt.ptr<f16>, #AL>, tensor<16x!tt.ptr<i64>, #BLs1>) {
632|     %82 = tt.load %arg20 : tensor<16x16x!tt.ptr<f16>, #AL>
633|     %83 = tt.load %arg21 : tensor<16x!tt.ptr<i64>, #BLs1>
634|     %84 = tt.expand_dims %83 {axis=1: i32}: tensor<16xi64, #BLs1> -> tensor<16x1xi64, #BL>
635|     %850 = tt.broadcast %84 : tensor<16x1xi64, #BL> -> tensor<16x16xi64, #BL>
```
**EN:** This function-oriented block defines or enters `indirect_bmm_vector`. Within it, the test exercises constants, tt.func, masked or vectorized loads, broadcasted scalars or pointers, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `indirect_bmm_vector` 为核心。测试在其中演示 常量、tt.func、带掩码或向量化的加载、广播后的标量或指针、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 636-653
```mlir
636|     %85 = arith.muli %77, %850 : tensor<16x16xi64, #BL>
637|     %86 = tt.addptr %60, %85 : tensor<16x16x!tt.ptr<f16>, #BL>, tensor<16x16xi64, #BL>
638|     %87 = tt.load %86 : tensor<16x16x!tt.ptr<f16>, #BL>
639|     %88 = ttg.convert_layout %82 : tensor<16x16xf16, #AL> -> tensor<16x16xf16, #A>
640|     %89 = ttg.convert_layout %87 : tensor<16x16xf16, #BL> -> tensor<16x16xf16, #B>
641|     %90 = tt.dot %88, %89, %arg19 : tensor<16x16xf16, #A> * tensor<16x16xf16, #B> -> tensor<16x16xf32, #C>
642|     %91 = tt.addptr %arg20, %78 : tensor<16x16x!tt.ptr<f16>, #AL>, tensor<16x16xi32, #AL>
643|     %92 = tt.addptr %arg21, %c1_i32_splat : tensor<16x!tt.ptr<i64>, #BLs1>, tensor<16xi32, #BLs1>
644|     scf.yield %90, %91, %92 : tensor<16x16xf32, #C>, tensor<16x16x!tt.ptr<f16>, #AL>, tensor<16x!tt.ptr<i64>, #BLs1>
645|   } {tt.num_stages = 3 : i32}
646|   tt.return %79#0 : tensor<16x16xf32, #C>
647| }
648| 
649| // COMMON-LABEL: tt.func @post_load_inv
650| // COMMON: scf.for
651| // COMMON-DAG: %[[IV:.*]] = arith.index_cast
652| // COMMON: %[[NEXT_IV:.*]] = arith.addi %[[IV]], %c1_i32 : i32
653| // COMMON: arith.index_cast
```
**EN:** This function-oriented block defines or enters `post_load_inv`. Within it, the test exercises pointer arithmetic, layout conversions, integer multiplications, masked or vectorized loads, dot-product or MMA-style math, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `post_load_inv` 为核心。测试在其中演示 指针算术、布局转换、整数乘法、带掩码或向量化的加载、点积或 MMA 风格计算，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 654-654
```mlir
654| // COMMON-NOT: arith.addi %[[NEXT_IV]]
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 655-672
```mlir
655| tt.func @post_load_inv(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32},
656|                        %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32},
657|                        %arg2: !tt.ptr<f32> {tt.divisibility = 16 : i32},
658|                        %arg3: i32 {tt.divisibility = 16 : i32},
659|                        %arg4: i32 {tt.divisibility = 16 : i32},
660|                        %arg5: i32 {tt.divisibility = 16 : i32},
661|                        %arg6: i32 {tt.divisibility = 16 : i32},
662|                        %arg7: i32 {tt.divisibility = 16 : i32},
663|                        %arg8: i32 {tt.divisibility = 16 : i32}) -> tensor<32x32xf32, #C> {
664|   %c0_index = arith.constant 0 : index
665|   %c1_index = arith.constant 1 : index
666|   %c1_i32 = arith.constant 1 : i32
667|   %c32_i32 = arith.constant 32 : i32
668|   %84 = arith.constant 900 : index
669|   %cst = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #C>
670|   %cst_0 = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #AL>
671|   %50 = tt.splat %arg3 : i32 -> tensor<1x32xi32, #AL>
672|   %59 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<32x32x!tt.ptr<f32>, #AL>
```
**EN:** This function-oriented block defines or enters `post_load_inv`. Within it, the test exercises constants, tt.func, broadcasted scalars or pointers, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `post_load_inv` 为核心。测试在其中演示 常量、tt.func、广播后的标量或指针，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 673-690
```mlir
673|   %81 = tt.splat %arg1 : !tt.ptr<f32> -> tensor<32x32x!tt.ptr<f32>, #AL>
674|   %66 = tt.splat %arg4 : i32 -> tensor<32x1xi32, #AL>
675|   %60 = tt.splat %arg2 : !tt.ptr<f32> -> tensor<32x32x!tt.ptr<f32>, #AL>
676|   %82 = tt.splat %arg2 : !tt.ptr<f32> -> tensor<32x32x!tt.ptr<f32>, #AL>
677|   %85:3 = scf.for %arg9 = %c0_index to %84 step %c1_index iter_args(%arg10 = %cst, %arg11 = %59, %arg12 = %81) -> (tensor<32x32xf32, #C>, tensor<32x32x!tt.ptr<f32>, #AL>, tensor<32x32x!tt.ptr<f32>, #AL>)  {
678|     %130 = arith.index_cast %arg9 : index to i32
679|     %107 = arith.muli %130, %c32_i32 : i32
680|     %108 = arith.subi %arg5, %107 : i32
681|     %109 = tt.splat %108 : i32 -> tensor<1x32xi32, #AL>
682|     %110 = arith.cmpi "slt", %50, %109 : tensor<1x32xi32, #AL>
683|     %111 = tt.broadcast %110 : tensor<1x32xi1, #AL> -> tensor<32x32xi1, #AL>
684|     %112 = tt.load %arg11, %111, %cst_0 : tensor<32x32x!tt.ptr<f32>, #AL>
685|     %113 = tt.splat %108 : i32 -> tensor<32x1xi32, #AL>
686|     %114 = arith.cmpi "slt", %66, %113 : tensor<32x1xi32, #AL>
687|     %115 = tt.broadcast %114 : tensor<32x1xi1, #AL> -> tensor<32x32xi1, #AL>
688|     %116 = tt.load %arg12, %115, %cst_0 : tensor<32x32x!tt.ptr<f32>, #AL>
689|     %117 = ttg.convert_layout %112 : tensor<32x32xf32, #AL> -> tensor<32x32xf32, #ttg.dot_op<{opIdx = 0, parent = #C, kWidth = 1}>>
690|     %118 = ttg.convert_layout %116 : tensor<32x32xf32, #AL> -> tensor<32x32xf32, #ttg.dot_op<{opIdx = 1, parent = #C, kWidth = 1}>>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining broadcasted scalars or pointers, integer comparisons, tensor broadcasting, masked or vectorized loads, layout conversions.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 广播后的标量或指针、整数比较、张量广播、带掩码或向量化的加载、布局转换。

### Lines 691-708
```mlir
691|     %119 = tt.dot %117, %118, %arg10, inputPrecision = tf32 : tensor<32x32xf32, #ttg.dot_op<{opIdx = 0, parent = #C, kWidth = 1}>> * tensor<32x32xf32, #ttg.dot_op<{opIdx = 1, parent = #C, kWidth = 1}>> -> tensor<32x32xf32, #C>
692|     %131 = arith.index_cast %arg9 : index to i32
693|     %120 = arith.addi %131, %c1_i32 : i32
694|     %121 = arith.muli %120, %c32_i32 : i32
695|     %122 = tt.splat %121 : i32 -> tensor<32x32xi32, #AL>
696|     %123 = tt.addptr %60, %122 : tensor<32x32x!tt.ptr<f32>, #AL>, tensor<32x32xi32, #AL>
697|     %124 = arith.muli %121, %arg7 : i32
698|     %125 = tt.splat %124 : i32 -> tensor<32x32xi32, #AL>
699|     %126 = tt.addptr %82, %125 : tensor<32x32x!tt.ptr<f32>, #AL>, tensor<32x32xi32, #AL>
700|     scf.yield %119, %123, %126 : tensor<32x32xf32, #C>, tensor<32x32x!tt.ptr<f32>, #AL>, tensor<32x32x!tt.ptr<f32>, #AL>
701|   }
702|   tt.return %85#0 : tensor<32x32xf32, #C>
703| }
704| 
705| // COMMON-LABEL: tt.func @cross_iter_dep
706| // TODO: enable pipelining with distance of 2
707| // COMMON-NOT: ttg.async_commit_group
708| // COMMON: scf.for
```
**EN:** This function-oriented block defines or enters `cross_iter_dep`. Within it, the test exercises integer multiplications, broadcasted scalars or pointers, pointer arithmetic, dot-product or MMA-style math, arith.index_cast, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `cross_iter_dep` 为核心。测试在其中演示 整数乘法、广播后的标量或指针、指针算术、点积或 MMA 风格计算、arith.index_cast，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 709-710
```mlir
709| // COMMON: scf.yield
710| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 711-728
```mlir
711| tt.func @cross_iter_dep(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32},
712|                         %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32},
713|                         %arg2: !tt.ptr<f32> {tt.divisibility = 16 : i32},
714|                         %arg3: i32 {tt.divisibility = 16 : i32},
715|                         %arg4: i32 {tt.divisibility = 16 : i32},
716|                         %arg5: i32 {tt.divisibility = 16 : i32},
717|                         %arg6: i32 {tt.divisibility = 16 : i32},
718|                         %arg7: i32 {tt.divisibility = 16 : i32},
719|                         %arg8: i32 {tt.divisibility = 16 : i32}) -> tensor<32x32xf32, #C> {
720|   %c0_i32 = arith.constant 0 : index
721|   %118 = arith.constant 32 : index
722|   %c1_i32 = arith.constant 1 : index
723|   %c2_i32 = arith.constant 2 : i32
724|   %c32_i32 = arith.constant 32 : i32
725|   %cst = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #C>
726|   %cst_1 = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #AL>
727|   %78 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<32x32x!tt.ptr<f32>, #AL>
728|   %110 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<32x32x!tt.ptr<f32>, #AL>
```
**EN:** This function-oriented block defines or enters `cross_iter_dep`. Within it, the test exercises constants, tt.func, broadcasted scalars or pointers, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `cross_iter_dep` 为核心。测试在其中演示 常量、tt.func、广播后的标量或指针，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 729-746
```mlir
729|   %112 = tt.splat %arg1 : !tt.ptr<f32> -> tensor<32x32x!tt.ptr<f32>, #AL>
730|   %113 = tt.splat %arg1 : !tt.ptr<f32> -> tensor<32x32x!tt.ptr<f32>, #AL>
731|   %116 = tt.splat %arg2 : !tt.ptr<f32> -> tensor<32x32x!tt.ptr<f32>, #AL>
732|   %65 = tt.splat %arg3 : i32 -> tensor<1x32xi32, #AL>
733|   %88 = tt.splat %arg4 : i32 -> tensor<32x1xi32, #AL>
734|   %80 = tt.splat %arg2 : !tt.ptr<f32> -> tensor<32x32x!tt.ptr<f32>, #AL>
735|   %119:5 = scf.for %arg9 = %c0_i32 to %118 step %c1_i32 iter_args(%arg10 = %cst, %arg11 = %78, %arg12 = %110, %arg13 = %113, %arg14 = %116) -> (tensor<32x32xf32, #C>, tensor<32x32x!tt.ptr<f32>, #AL>, tensor<32x32x!tt.ptr<f32>, #AL>, tensor<32x32x!tt.ptr<f32>, #AL>, tensor<32x32x!tt.ptr<f32>, #AL>)  {
736|     %161 = arith.index_cast %arg9 : index to i32
737|     %141 = arith.muli %161, %c32_i32 : i32
738|     %142 = arith.subi %arg5, %141 : i32
739|     %143 = tt.splat %142 : i32 -> tensor<1x32xi32, #AL>
740|     %144 = arith.cmpi "slt", %65, %143 : tensor<1x32xi32, #AL>
741|     %145 = tt.broadcast %144 : tensor<1x32xi1, #AL> -> tensor<32x32xi1, #AL>
742|     %146 = tt.load %arg11, %145, %cst_1 : tensor<32x32x!tt.ptr<f32>, #AL>
743|     %147 = tt.splat %142 : i32 -> tensor<32x1xi32, #AL>
744|     %148 = arith.cmpi "slt", %88, %147 : tensor<32x1xi32, #AL>
745|     %149 = tt.broadcast %148 : tensor<32x1xi1, #AL> -> tensor<32x32xi1, #AL>
746|     %150 = tt.load %arg12, %149, %cst_1 : tensor<32x32x!tt.ptr<f32>, #AL>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining broadcasted scalars or pointers, integer comparisons, tensor broadcasting, masked or vectorized loads, structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 广播后的标量或指针、整数比较、张量广播、带掩码或向量化的加载、结构化循环。

### Lines 747-764
```mlir
747|     %151 = ttg.convert_layout %146 : tensor<32x32xf32, #AL> -> tensor<32x32xf32, #ttg.dot_op<{opIdx = 0, parent = #C, kWidth = 1}>>
748|     %152 = ttg.convert_layout %150 : tensor<32x32xf32, #AL> -> tensor<32x32xf32, #ttg.dot_op<{opIdx = 1, parent = #C, kWidth = 1}>>
749|     %153 = tt.dot %151, %152, %arg10, inputPrecision = tf32 : tensor<32x32xf32, #ttg.dot_op<{opIdx = 0, parent = #C, kWidth = 1}>> * tensor<32x32xf32, #ttg.dot_op<{opIdx = 1, parent = #C, kWidth = 1}>> -> tensor<32x32xf32, #C>
750|     %162 = arith.index_cast %arg9 : index to i32
751|     %154 = arith.addi %162, %c2_i32 : i32
752|     %155 = arith.muli %154, %c32_i32 : i32
753|     %156 = tt.splat %155 : i32 -> tensor<32x32xi32, #AL>
754|     %157 = tt.addptr %80, %156 : tensor<32x32x!tt.ptr<f32>, #AL>, tensor<32x32xi32, #AL>
755|     %158 = arith.muli %155, %arg7 : i32
756|     %159 = tt.splat %158 : i32 -> tensor<32x32xi32, #AL>
757|     %160 = tt.addptr %112, %159 : tensor<32x32x!tt.ptr<f32>, #AL>, tensor<32x32xi32, #AL>
758|     scf.yield %153, %arg13, %arg14, %157, %160 : tensor<32x32xf32, #C>, tensor<32x32x!tt.ptr<f32>, #AL>, tensor<32x32x!tt.ptr<f32>, #AL>, tensor<32x32x!tt.ptr<f32>, #AL>, tensor<32x32x!tt.ptr<f32>, #AL>
759|   }
760|   tt.return %119#0 : tensor<32x32xf32, #C>
761| }
762| 
763| // COMMON-LABEL: tt.func @dep_arg_two_uses
764| // COMMON: tt.expand_dims
```
**EN:** This function-oriented block defines or enters `dep_arg_two_uses`. Within it, the test exercises layout conversions, integer multiplications, broadcasted scalars or pointers, pointer arithmetic, dot-product or MMA-style math, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `dep_arg_two_uses` 为核心。测试在其中演示 布局转换、整数乘法、广播后的标量或指针、指针算术、点积或 MMA 风格计算，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 765-769
```mlir
765| // COMMON: tt.expand_dims
766| // COMMON: tt.expand_dims %arg5
767| // COMMON: %[[PTR0:.*]] = tt.splat %arg6
768| // COMMON: %[[PTR1:.*]] = tt.addptr %[[PTR0]]
769| // COMMON-NEXT: tt.load %[[PTR1]]
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 770-787
```mlir
770| tt.func @dep_arg_two_uses(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32},
771|                           %arg1: !tt.ptr<i32> {tt.divisibility = 16 : i32},
772|                           %arg2: !tt.ptr<f16> {tt.divisibility = 16 : i32}) -> tensor<128x128xf32, #C> {
773|   %23 = arith.constant 100 : index
774|   %c64 = arith.constant 64 : i64
775|   %56 = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 0, parent = #AL}>>
776|   %57 = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 0, parent = #AL}>>
777|   %58 = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 1, parent = #BL}>>
778|   %83 = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 0, parent = #AL}>>
779|   %85 = tt.splat %c64 : i64 -> tensor<1x32xi64, #AL>
780|   %86 = tt.splat %c64 : i64 -> tensor<1x32xi64, #AL>
781|   %68 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<128x32x!tt.ptr<f16>, #AL>
782|   %c32_index = arith.constant 32 : index
783|   %c32_i32 = arith.index_cast %c32_index : index to i32
784|   %80 = tt.splat %arg2 : !tt.ptr<f16> -> tensor<32x128x!tt.ptr<f16>, #BL>
785|   %cst_6 = arith.constant dense<0.000000e+00> : tensor<32x128xf32, #BL>
786|   %88 = arith.truncf %cst_6 : tensor<32x128xf32, #BL> to tensor<32x128xf16, #BL>
787|   %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #C>
```
**EN:** This function-oriented block defines or enters `dep_arg_two_uses`. Within it, the test exercises constants, lane/block index ranges, broadcasted scalars or pointers, tt.func, arith.index_cast, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `dep_arg_two_uses` 为核心。测试在其中演示 常量、lane/block 索引范围、广播后的标量或指针、tt.func、arith.index_cast，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 788-805
```mlir
788|   %90 = tt.splat %c64 : i64 -> tensor<32x128xi64, #BL>
789|   %92 = tt.addptr %arg1, %c32_i32 : !tt.ptr<i32>, i32
790|   %c0_index = arith.constant 0 : index
791|   %91:5 = scf.for %arg19 = %c0_index to %23 step %c32_index iter_args(%arg20 = %68, %arg21 = %83, %arg22 = %92, %arg23 = %cst, %arg24 = %80) -> (tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32xi32, #ttg.slice<{dim = 0, parent = #AL}>>, !tt.ptr<i32>, tensor<128x128xf32, #C>, tensor<32x128x!tt.ptr<f16>, #BL>)   {
792|     %1750 = arith.subi %23, %arg19 : index
793|     %175 = arith.index_cast %1750 : index to i32
794|     %176 = tt.splat %175 : i32 -> tensor<32xi32, #ttg.slice<{dim = 0, parent = #AL}>>
795|     %177 = tt.splat %175 : i32 -> tensor<32xi32, #ttg.slice<{dim = 1, parent = #BL}>>
796|     %178 = arith.cmpi "slt", %57, %176 : tensor<32xi32, #ttg.slice<{dim = 0, parent = #AL}>>
797|     %179 = arith.cmpi "slt", %58, %177 : tensor<32xi32, #ttg.slice<{dim = 1, parent = #BL}>>
798|     %180 = tt.expand_dims %178 {axis = 0 : i32} : tensor<32xi1, #ttg.slice<{dim = 0, parent = #AL}>> -> tensor<1x32xi1, #AL>
799|     %181 = tt.expand_dims %179 {axis = 1 : i32} : tensor<32xi1, #ttg.slice<{dim = 1, parent = #BL}>> -> tensor<32x1xi1, #BL>
800|     %182 = tt.expand_dims %arg21 {axis = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 0, parent = #AL}>> -> tensor<1x32xi32, #AL>
801|     %183 = tt.expand_dims %arg21 {axis = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 0, parent = #AL}>> -> tensor<1x32xi32, #AL>
802|     %184 = arith.extsi %182 : tensor<1x32xi32, #AL> to tensor<1x32xi64, #AL>
803|     %185 = arith.extsi %183 : tensor<1x32xi32, #AL> to tensor<1x32xi64, #AL>
804|     %186 = arith.muli %184, %85 : tensor<1x32xi64, #AL>
805|     %187 = arith.muli %185, %86 : tensor<1x32xi64, #AL>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining shape expansion, broadcasted scalars or pointers, integer comparisons, arith.extsi, integer multiplications.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 形状扩展、广播后的标量或指针、整数比较、arith.extsi、整数乘法。

### Lines 806-823
```mlir
806|     %188 = tt.broadcast %186 : tensor<1x32xi64, #AL> -> tensor<128x32xi64, #AL>
807|     %189 = tt.broadcast %187 : tensor<1x32xi64, #AL> -> tensor<128x32xi64, #AL>
808|     %190 = tt.addptr %arg20, %188 : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xi64, #AL>
809|     %191 = tt.addptr %arg20, %189 : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xi64, #AL>
810|     %192 = tt.broadcast %180 : tensor<1x32xi1, #AL> -> tensor<128x32xi1, #AL>
811|     %193 = tt.load %191, %192 : tensor<128x32x!tt.ptr<f16>, #AL>
812|     %194 = tt.splat %arg22 : !tt.ptr<i32> -> tensor<32x!tt.ptr<i32>, #ttg.slice<{dim = 0, parent = #AL}>>
813|     %195 = tt.addptr %194, %56 : tensor<32x!tt.ptr<i32>, #ttg.slice<{dim = 0, parent = #AL}>>, tensor<32xi32, #ttg.slice<{dim = 0, parent = #AL}>>
814|     %196 = tt.load %195 : tensor<32x!tt.ptr<i32>, #ttg.slice<{dim = 0, parent = #AL}>>
815|     %197 = tt.addptr %arg22, %c32_i32 : !tt.ptr<i32>, i32
816|     %198 = tt.broadcast %181 : tensor<32x1xi1, #BL> -> tensor<32x128xi1, #BL>
817|     %199 = tt.load %arg24, %198, %88 : tensor<32x128x!tt.ptr<f16>, #BL>
818|     %200 = ttg.convert_layout %193 : tensor<128x32xf16, #AL> -> tensor<128x32xf16, #ttg.dot_op<{opIdx = 0, parent = #C, kWidth = 2}>>
819|     %201 = ttg.convert_layout %199 : tensor<32x128xf16, #BL> -> tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #C, kWidth = 2}>>
820|     %202 = tt.dot %200, %201, %arg23 : tensor<128x32xf16, #ttg.dot_op<{opIdx = 0, parent = #C, kWidth = 2}>> * tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #C, kWidth = 2}>> -> tensor<128x128xf32, #C>
821|     %203 = tt.addptr %arg24, %90 : tensor<32x128x!tt.ptr<f16>, #BL>, tensor<32x128xi64, #BL>
822|     scf.yield %190, %196, %197, %202, %203 : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32xi32, #ttg.slice<{dim = 0, parent = #AL}>>, !tt.ptr<i32>, tensor<128x128xf32, #C>, tensor<32x128x!tt.ptr<f16>, #BL>
823|   }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining pointer arithmetic, tensor broadcasting, masked or vectorized loads, layout conversions, broadcasted scalars or pointers.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 指针算术、张量广播、带掩码或向量化的加载、布局转换、广播后的标量或指针。

### Lines 824-827
```mlir
824|   tt.return %91#3 : tensor<128x128xf32, #C>
825| }
826| }  // end module
827| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 828-828
```mlir
828| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 829-835
```mlir
829| 
830| #blocked = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [8, 4], warpsPerCTA = [1, 4], order = [0, 1]}>
831| #blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
832| #mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 8]}>
833| #shared = #ttg.swizzled_shared<{vec = 4, perPhase = 1, maxPhase = 2, order = [0, 1]}>
834| #shared1 = #ttg.swizzled_shared<{vec = 4, perPhase = 1, maxPhase = 2, order = [1, 0]}>
835| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 836-836
```mlir
836| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 837-837
```mlir
837| // COMMON-LABEL: tt.func @load_two_users_incompatible_layouts
```
**EN:** This function-oriented block defines or enters `load_two_users_incompatible_layouts`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `load_two_users_incompatible_layouts` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 838-855
```mlir
838|   tt.func @load_two_users_incompatible_layouts(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32}) -> (tensor<128x16xf32, #mma>, tensor<128x64xf32, #mma>) {
839|     %cst = arith.constant dense<0> : tensor<1x16xi32, #blocked>
840|     %cst_0 = arith.constant dense<0> : tensor<128x1xi32, #blocked1>
841|     %c0_i64 = arith.constant 0 : i64
842|     %c0_i32 = arith.constant 0 : i32
843|     %cst_1 = arith.constant dense<0.000000e+00> : tensor<128x16xf32, #mma>
844|     %cst_2 = arith.constant dense<0.000000e+00> : tensor<128x64xf32, #mma>
845|     %c1_i32 = arith.constant 1 : i32
846|     %c8_i32 = arith.constant 8 : i32
847|     %0 = tt.addptr %arg0, %c0_i64 : !tt.ptr<f16>, i64
848|     %1 = tt.addptr %arg1, %c0_i64 : !tt.ptr<f16>, i64
849|     %2 = tt.splat %1 : !tt.ptr<f16> -> tensor<128x1x!tt.ptr<f16>, #blocked1>
850|     %3 = tt.addptr %2, %cst_0 : tensor<128x1x!tt.ptr<f16>, #blocked1>, tensor<128x1xi32, #blocked1>
851|     %4 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
852|     %5 = tt.expand_dims %4 {axis = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked1}>> -> tensor<1x64xi32, #blocked1>
853|     %6 = tt.broadcast %3 : tensor<128x1x!tt.ptr<f16>, #blocked1> -> tensor<128x64x!tt.ptr<f16>, #blocked1>
854|     %7 = tt.broadcast %5 : tensor<1x64xi32, #blocked1> -> tensor<128x64xi32, #blocked1>
855|     %8 = tt.addptr %6, %7 : tensor<128x64x!tt.ptr<f16>, #blocked1>, tensor<128x64xi32, #blocked1>
```
**EN:** This function-oriented block defines or enters `load_two_users_incompatible_layouts`. Within it, the test exercises constants, pointer arithmetic, tt.func, tensor broadcasting, broadcasted scalars or pointers, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `load_two_users_incompatible_layouts` 为核心。测试在其中演示 常量、指针算术、tt.func、张量广播、广播后的标量或指针，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 856-873
```mlir
856|     %9 = tt.load %8 : tensor<128x64x!tt.ptr<f16>, #blocked1>
857|     %10 = tt.splat %0 : !tt.ptr<f16> -> tensor<1x16x!tt.ptr<f16>, #blocked>
858|     %11 = tt.addptr %10, %cst : tensor<1x16x!tt.ptr<f16>, #blocked>, tensor<1x16xi32, #blocked>
859|     %12 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
860|     %13 = tt.expand_dims %12 {axis = 1 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<64x1xi32, #blocked>
861|     %14 = tt.broadcast %11 : tensor<1x16x!tt.ptr<f16>, #blocked> -> tensor<64x16x!tt.ptr<f16>, #blocked>
862|     %15 = tt.broadcast %13 : tensor<64x1xi32, #blocked> -> tensor<64x16xi32, #blocked>
863|     %16 = tt.addptr %14, %15 : tensor<64x16x!tt.ptr<f16>, #blocked>, tensor<64x16xi32, #blocked>
864|     // check that the load didn't get pipelined.
865|     // COMMON-NOT: alloc
866|     // COMMON: scf.for
867|     %17:2 = scf.for %arg2 = %c0_i32 to %c8_i32 step %c1_i32 iter_args(%arg3 = %cst_1, %arg4 = %cst_2) -> (tensor<128x16xf32, #mma>, tensor<128x64xf32, #mma>)  : i32 {
868|       %18 = tt.load %16 : tensor<64x16x!tt.ptr<f16>, #blocked>
869|       %19 = ttg.convert_layout %9 : tensor<128x64xf16, #blocked1> -> tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
870|       %20 = ttg.convert_layout %18 : tensor<64x16xf16, #blocked> -> tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>
871|       %21 = tt.dot %19, %20, %cst_1 : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<128x16xf32, #mma>
872|       %22 = arith.truncf %21 : tensor<128x16xf32, #mma> to tensor<128x16xf16, #mma>
873|       %23 = ttg.convert_layout %22 : tensor<128x16xf16, #mma> -> tensor<128x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining layout conversions, masked or vectorized loads, pointer arithmetic, tensor broadcasting, broadcasted scalars or pointers.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 布局转换、带掩码或向量化的加载、指针算术、张量广播、广播后的标量或指针。

### Lines 874-885
```mlir
874|       %24 = ttg.local_alloc %18 : (tensor<64x16xf16, #blocked>) -> !ttg.memdesc<64x16xf16, #shared, #smem>
875|       %25 = ttg.memdesc_trans %24 {order=array<i32: 1,0>} : !ttg.memdesc<64x16xf16, #shared, #smem> -> !ttg.memdesc<16x64xf16, #shared1, #smem>
876|       %26 = ttg.local_load %25 : !ttg.memdesc<16x64xf16, #shared1, #smem> -> tensor<16x64xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>
877|       %27 = tt.dot %23, %26, %arg4 : tensor<128x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<16x64xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<128x64xf32, #mma>
878|       // COMMON: scf.yield
879|       scf.yield %21, %27 : tensor<128x16xf32, #mma>, tensor<128x64xf32, #mma>
880|     }
881|     // COMMON-NOT: alloc
882|     tt.return %17#0, %17#1 : tensor<128x16xf32, #mma>, tensor<128x64xf32, #mma>
883|   }
884| }
885| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining shared/local memory allocation, ttg.memdesc_trans, local/shared memory loads, dot-product or MMA-style math, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 共享/本地内存分配、ttg.memdesc_trans、本地/共享内存加载、点积或 MMA 风格计算、循环/分支产出值。

### Lines 886-886
```mlir
886| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 887-898
```mlir
887| 
888| // CHECK-LABEL: nested_loops
889| // CHECK: scf.for
890| // CHECK:   ttg.local_alloc
891| // CHECK:   ttg.async_copy_global_to_local
892| // CHECK:   ttg.async_commit_group
893| // CHECK:   ttg.async_copy_global_to_local
894| // CHECK:   ttg.async_commit_group
895| // CHECK:   scf.for
896| // CHECK:     scf.yield
897| // CHECK:   ttg.async_wait {num = 0 : i32}
898| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: nested_loops anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: nested_loops 这样的标签用于锚定匹配范围。

### Lines 899-916
```mlir
899| // AMD-LABEL: tt.func public @nested_loops
900| //       AMD: scf.for
901| //       AMD:   ttg.local_alloc
902| //   AMD-NOT:   ttg.local_alloc
903| //       AMD:   scf.for
904| //       AMD:     scf.yield
905| //   AMD-DIS:   scf.yield
906| 
907| //
908| // The following code has the structure:
909| //
910| // ```
911| // for {
912| //   %a = load()
913| //   for {
914| //     %b = load()
915| //     dot(%a, %b)
916| //   }
```
**EN:** This function-oriented block defines or enters `nested_loops`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `nested_loops` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 917-922
```mlir
917| // }
918| // ```
919| //
920| // For CUDA, we pipeline the inner loop first then pipeline the outer
921| // loop to prefetch the async copy after the inner loop.
922| // For HIP, we only pipeline the inner loop for now.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 923-925
```mlir
923| #blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
924| #mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [2, 2], instrShape = [16, 8]}>
925| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 926-926
```mlir
926| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 927-944
```mlir
927|   tt.func public @nested_loops(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg2: !tt.ptr<i32> {tt.divisibility = 16 : i32}, %arg3: !tt.ptr<f32> {tt.divisibility = 16 : i32}) {
928|     %cst = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #mma>
929|     %cst_0 = arith.constant dense<320> : tensor<32x1xi32, #blocked>
930|     %c0_i32 = arith.constant 0 : i32
931|     %c1_i32 = arith.constant 1 : i32
932|     %c32_i32 = arith.constant 32 : i32
933|     %c10_i32 = arith.constant 10 : i32
934|     %0 = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
935|     %1 = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
936|     %2 = tt.expand_dims %1 {axis = 1 : i32} : tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<32x1xi32, #blocked>
937|     %3 = arith.muli %2, %cst_0 : tensor<32x1xi32, #blocked>
938|     %4 = tt.splat %arg1 : !tt.ptr<f32> -> tensor<32x1x!tt.ptr<f32>, #blocked>
939|     %5 = tt.addptr %4, %3 : tensor<32x1x!tt.ptr<f32>, #blocked>, tensor<32x1xi32, #blocked>
940|     %6 = tt.broadcast %5 : tensor<32x1x!tt.ptr<f32>, #blocked> -> tensor<32x32x!tt.ptr<f32>, #blocked>
941|     %7 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<32x1x!tt.ptr<f32>, #blocked>
942|     %8 = tt.splat %arg3 : !tt.ptr<f32> -> tensor<32x1x!tt.ptr<f32>, #blocked>
943|     scf.for %arg4 = %c0_i32 to %c10_i32 step %c1_i32  : i32 {
944|       %9 = arith.muli %arg4, %c32_i32 : i32
```
**EN:** This function-oriented block defines or enters `nested_loops`. Within it, the test exercises constants, broadcasted scalars or pointers, tt.func, lane/block index ranges, integer multiplications, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `nested_loops` 为核心。测试在其中演示 常量、广播后的标量或指针、tt.func、lane/block 索引范围、整数乘法，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 945-962
```mlir
945|       %10 = tt.splat %9 : i32 -> tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
946|       %11 = tt.splat %9 : i32 -> tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
947|       %12 = arith.addi %10, %0 : tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
948|       %13 = arith.addi %11, %1 : tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
949|       %14 = tt.expand_dims %12 {axis = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x32xi32, #blocked>
950|       %15 = tt.broadcast %14 : tensor<1x32xi32, #blocked> -> tensor<32x32xi32, #blocked>
951|       %16 = tt.addptr %6, %15 : tensor<32x32x!tt.ptr<f32>, #blocked>, tensor<32x32xi32, #blocked>
952|       %17 = tt.load %16 : tensor<32x32x!tt.ptr<f32>, #blocked>
953|       %18 = tt.expand_dims %13 {axis = 1 : i32} : tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<32x1xi32, #blocked>
954|       %19 = arith.muli %18, %cst_0 : tensor<32x1xi32, #blocked>
955|       %20 = tt.addptr %7, %19 : tensor<32x1x!tt.ptr<f32>, #blocked>, tensor<32x1xi32, #blocked>
956|       %21 = tt.broadcast %20 : tensor<32x1x!tt.ptr<f32>, #blocked> -> tensor<32x32x!tt.ptr<f32>, #blocked>
957|       %22 = tt.addptr %8, %19 : tensor<32x1x!tt.ptr<f32>, #blocked>, tensor<32x1xi32, #blocked>
958|       %23 = tt.broadcast %22 : tensor<32x1x!tt.ptr<f32>, #blocked> -> tensor<32x32x!tt.ptr<f32>, #blocked>
959|       scf.for %arg5 = %c0_i32 to %c10_i32 step %c1_i32  : i32 {
960|         %24 = arith.muli %arg5, %c32_i32 : i32
961|         %25 = tt.splat %24 : i32 -> tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
962|         %26 = arith.addi %25, %0 : tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining broadcasted scalars or pointers, integer additions, tensor broadcasting, pointer arithmetic, shape expansion.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 广播后的标量或指针、整数加法、张量广播、指针算术、形状扩展。

### Lines 963-979
```mlir
963|         %27 = tt.expand_dims %26 {axis = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x32xi32, #blocked>
964|         %28 = tt.broadcast %27 : tensor<1x32xi32, #blocked> -> tensor<32x32xi32, #blocked>
965|         %29 = tt.addptr %21, %28 : tensor<32x32x!tt.ptr<f32>, #blocked>, tensor<32x32xi32, #blocked>
966|         %30 = tt.load %29 : tensor<32x32x!tt.ptr<f32>, #blocked>
967|         %31 = ttg.convert_layout %30 : tensor<32x32xf32, #blocked> -> tensor<32x32xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>>
968|         %32 = ttg.convert_layout %17 : tensor<32x32xf32, #blocked> -> tensor<32x32xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>>
969|         %33 = tt.dot %31, %32, %cst, inputPrecision = tf32 : tensor<32x32xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>> * tensor<32x32xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>> -> tensor<32x32xf32, #mma>
970|         %34 = tt.addptr %23, %28 : tensor<32x32x!tt.ptr<f32>, #blocked>, tensor<32x32xi32, #blocked>
971|         %35 = ttg.convert_layout %33 : tensor<32x32xf32, #mma> -> tensor<32x32xf32, #blocked>
972|         tt.store %34, %35 : tensor<32x32x!tt.ptr<f32>, #blocked>
973|       }
974|     }
975|     tt.return
976|   }
977| }  // end module
978| 
979| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on layout conversions, pointer arithmetic, shape expansion, tensor broadcasting, masked or vectorized loads. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 布局转换、指针算术、形状扩展、张量广播、带掩码或向量化的加载。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 980-980
```mlir
980| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 981-981
```mlir
981| // CHECK: #[[$SHARED_LAYOUT:shared.*]] = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 982-995
```mlir
982| // CHECK-LABEL: tt.func @indirect_load_shared_layout
983| // CHECK: scf.for
984| // CHECK: ttg.async_wait {{.*}} {num = 1 : i32}
985| // CHECK: %[[NEXT_BUFFER_1:.*]] = tt.addptr %{{.*}}, {{.*}}
986| // CHECK: ttg.async_copy_global_to_local %[[NEXT_BUFFER_1]]
987| // CHECK: %[[IND_BUFFER_0:.*]] = ttg.memdesc_index {{.*}} : !ttg.memdesc<1x16xi64, #[[$SHARED_LAYOUT]], #smem, mutable> -> !ttg.memdesc<16xi64, #[[$SHARED_LAYOUT]], #smem, mutable>
988| // CHECK: %[[IND_BUFFER_1:.*]] = ttg.local_load %[[IND_BUFFER_0]]
989| // CHECK: %[[IND_BUFFER_2:.*]] = tt.expand_dims %[[IND_BUFFER_1]] {axis = 1 : i32}
990| // CHECK: %[[IND_BUFFER_3:.*]] = tt.broadcast %[[IND_BUFFER_2]]
991| // CHECK: %[[IND_BUFFER_4:.*]] = arith.muli {{.*}}, %[[IND_BUFFER_3]]
992| // CHECK: %[[NEXT_BUFFER_0:.*]] = tt.addptr {{.*}}, %[[IND_BUFFER_4]]
993| // CHECK: ttg.async_copy_global_to_local %[[NEXT_BUFFER_0]]
994| 
995| //   AMD-DIS: #[[$SHARED_LAYOUT:shared.*]] = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
```
**EN:** This function-oriented block defines or enters `indirect_load_shared_layout`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `indirect_load_shared_layout` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 996-1013
```mlir
 996| // AMD-LABEL: tt.func @indirect_load_shared_layout
 997| //       AMD:   %[[LOCAL_ALLOC_0:.*]] = ttg.local_alloc
 998| //       AMD:   %[[LOCAL_ALLOC_1:.*]] = ttg.local_alloc
 999| //       AMD:   %{{.*}}:7 = scf.for %[[ARG6:.*]] = %{{.*}} to %{{.*}} step %{{.*}} iter_args(%[[ARG7:.*]] = %{{.*}}, %[[ARG8:.*]] = %{{.*}}, %[[ARG9:.*]] = %{{.*}}, %[[ARG10:.*]] = %{{.*}}, %[[ARG11:.*]] = %{{.*}}, %[[ARG12:.*]] = %{{.*}}, %[[ARG13:.*]] = %{{.*}})
1000| //       AMD:     %[[ADDPTR_47:.*]] = tt.addptr %[[ARG8]], %{{.*}}
1001| //       AMD:     %[[ADDPTR_48:.*]] = tt.addptr %[[ARG9]], %{{.*}}
1002| //       AMD:     %[[LOAD_49:.*]] = tt.load %[[ADDPTR_47]]
1003| //       AMD:     %[[LOCAL_LOAD_50:.*]] = ttg.local_load %[[ARG11]]
1004| //       AMD:     %[[LOAD_51:.*]] = tt.load %[[ADDPTR_48]]
1005| //       AMD:     %[[EXPAND_DIMS_52:.*]] = tt.expand_dims %[[ARG12]] {axis = 1 : i32}
1006| //       AMD:     %[[BROADCAST_53:.*]] = tt.broadcast %[[EXPAND_DIMS_52]]
1007| //       AMD:     %[[MULI_54:.*]] = arith.muli %{{.*}}, %[[BROADCAST_53]]
1008| //       AMD:     %[[ADDPTR_55:.*]] = tt.addptr %{{.*}}, %[[MULI_54]]
1009| //       AMD:     %[[LOAD_56:.*]] = tt.load %[[ADDPTR_55]]
1010| //       AMD:     %[[LOCAL_LOAD_57:.*]] = ttg.local_load %[[ARG13]]
1011| //       AMD:     %[[DOT_58:.*]] = tt.dot %[[LOCAL_LOAD_50]], %[[LOCAL_LOAD_57]], %[[ARG7]]
1012| //       AMD:     %[[ADDI_59:.*]] = arith.addi %[[ARG10]], %{{.*}}
1013| //       AMD:     %[[CMPI_60:.*]] = arith.cmpi slt, %[[ADDI_59]], %{{.*}}
```
**EN:** This function-oriented block defines or enters `indirect_load_shared_layout`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `indirect_load_shared_layout` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1014-1031
```mlir
1014| //       AMD:     %[[SELECT_61:.*]] = arith.select %[[CMPI_60]], %[[ADDI_59]], %{{.*}}
1015| //       AMD:     %[[MEMDESC_SUBVIEW_62:.*]] = ttg.memdesc_index %{{.*}}{{\[}}%[[SELECT_61]]{{\]}}
1016| //       AMD:     ttg.local_store %[[LOAD_49]], %[[MEMDESC_SUBVIEW_62]]
1017| //       AMD:     %[[MEMDESC_SUBVIEW_63:.*]] = ttg.memdesc_index %{{.*}}{{\[}}%[[SELECT_61]]{{\]}}
1018| //       AMD:     ttg.local_store %[[LOAD_56]], %[[MEMDESC_SUBVIEW_63]]
1019| //       AMD:     scf.yield %[[DOT_58]], %[[ADDPTR_47]], %[[ADDPTR_48]], %[[SELECT_61]], %[[MEMDESC_SUBVIEW_62]], %[[LOAD_51]], %[[MEMDESC_SUBVIEW_63]]
1020| //       AMD:   }
1021| //       AMD:     %[[CMPI_21:.*]] = arith.cmpi sge, %{{.*}}, %{{.*}}
1022| //       AMD:     %[[CMPI_22:.*]] = arith.cmpi sge, %{{.*}}, %{{.*}}
1023| //       AMD:     %[[ADDPTR_23:.*]] = tt.addptr %{{.*}}#1, %{{.*}}
1024| //       AMD:     %[[SPLAT_24:.*]] = tt.splat %[[CMPI_22]]
1025| //       AMD:     %[[LOAD_25:.*]] = tt.load %[[ADDPTR_23]], %[[SPLAT_24]]
1026| //       AMD:     %[[LOCAL_LOAD_26:.*]] = ttg.local_load %{{.*}}#4
1027| //       AMD:     %[[EXPAND_DIMS_27:.*]] = tt.expand_dims %{{.*}}#5 {axis = 1 : i32}
1028| //       AMD:     %[[BROADCAST_28:.*]] = tt.broadcast %[[EXPAND_DIMS_27]]
1029| //       AMD:     %[[MULI_29:.*]] = arith.muli %{{.*}}, %[[BROADCAST_28]]
1030| //       AMD:     %[[ADDPTR_30:.*]] = tt.addptr %{{.*}}, %[[MULI_29]]
1031| //       AMD:     %[[SPLAT_31:.*]] = tt.splat %[[CMPI_22]]
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 1032-1049
```mlir
1032| //       AMD:     %[[LOAD_32:.*]] = tt.load %[[ADDPTR_30]], %[[SPLAT_31]]
1033| //       AMD:     %[[LOCAL_LOAD_33:.*]] = ttg.local_load %{{.*}}#6
1034| //       AMD:     %[[IF_34:.*]] = scf.if %[[CMPI_21]]
1035| //       AMD:       %[[DOT_45:.*]] = tt.dot %[[LOCAL_LOAD_26]], %[[LOCAL_LOAD_33]], %{{.*}}#0
1036| //       AMD:       scf.yield %[[DOT_45]]
1037| //       AMD:     } else {
1038| //       AMD:       scf.yield %{{.*}}#0
1039| //       AMD:     }
1040| //       AMD:     %[[ADDI_35:.*]] = arith.addi %{{.*}}#3, %{{.*}}
1041| //       AMD:     %[[CMPI_36:.*]] = arith.cmpi slt, %[[ADDI_35]], %{{.*}}
1042| //       AMD:     %[[SELECT_37:.*]] = arith.select %[[CMPI_36]], %[[ADDI_35]], %{{.*}}
1043| //       AMD:     %[[MEMDESC_SUBVIEW_38:.*]] = ttg.memdesc_index %{{.*}}{{\[}}%[[SELECT_37]]{{\]}}
1044| //       AMD:     ttg.local_store %[[LOAD_25]], %[[MEMDESC_SUBVIEW_38]]
1045| //       AMD:     %[[MEMDESC_SUBVIEW_39:.*]] = ttg.memdesc_index %{{.*}}{{\[}}%[[SELECT_37]]{{\]}}
1046| //       AMD:     ttg.local_store %[[LOAD_32]], %[[MEMDESC_SUBVIEW_39]]
1047| //       AMD:     %[[SELECT_40:.*]] = arith.select %[[CMPI_21]], %[[IF_34]], %{{.*}}#0
1048| //       AMD:     %[[LOCAL_LOAD_41:.*]] = ttg.local_load %[[MEMDESC_SUBVIEW_38]]
1049| //       AMD:     %[[LOCAL_LOAD_42:.*]] = ttg.local_load %[[MEMDESC_SUBVIEW_39]]
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 1050-1059
```mlir
1050| //       AMD:     %[[IF_43:.*]] = scf.if %[[CMPI_22]]
1051| //       AMD:       %[[DOT_45:.*]] = tt.dot %[[LOCAL_LOAD_41]], %[[LOCAL_LOAD_42]], %[[SELECT_40]]
1052| //       AMD:       scf.yield %[[DOT_45]]
1053| //       AMD:     } else {
1054| //       AMD:       scf.yield %[[SELECT_40]]
1055| //       AMD:     }
1056| //       AMD:     %[[SELECT_44:.*]] = arith.select %[[CMPI_22]], %[[IF_43]], %[[SELECT_40]]
1057| //       AMD:     ttg.local_dealloc %{{.*}}
1058| //       AMD:     ttg.local_dealloc %{{.*}}
1059| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 1060-1065
```mlir
1060| #AL = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
1061| #BL = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
1062| #BLs1 = #ttg.slice<{parent=#BL, dim=1}>
1063| #C = #ttg.nvidia_mma<{versionMajor = 2, warpsPerCTA = [4, 1], instrShape = [16, 8]}>
1064| #A = #ttg.dot_op<{opIdx = 0, parent = #C, kWidth=2}>
1065| #B = #ttg.dot_op<{opIdx = 1, parent = #C, kWidth=2}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1066-1066
```mlir
1066| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 1067-1084
```mlir
1067| tt.func @indirect_load_shared_layout(%77: tensor<16x16xi64, #BL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.constancy = dense<[16, 16]> : tensor<2xi32>},
1068|                    %76: index,
1069|                    %49: tensor<16x16x!tt.ptr<f16>, #AL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 2]> : tensor<2xi32>},
1070|                    %75: tensor<16x!tt.ptr<i64>, #BLs1>,
1071|                    %78: tensor<16x16xi32, #AL> {tt.constancy = dense<[16, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>},
1072|                    %60: tensor<16x16x!tt.ptr<f16>, #BL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>}) -> tensor<16x16xf32, #C> {
1073|   %cst = arith.constant dense<0.000000e+00> : tensor<16x16xf32, #C>
1074|   %c4_i32 = arith.constant 4 : i32
1075|   %c1 = arith.constant 1 : index
1076|   %c0 = arith.constant 0 : index
1077|   %c0_i64 = arith.constant 0 : i64
1078|   %c1_i32 = arith.constant 1 : i32
1079|   %c1_i32_splat = tt.splat %c1_i32 : i32 -> tensor<16xi32, #BLs1>
1080|   %79:3 = scf.for %arg18 = %c0 to %76 step %c1 iter_args(%arg19 = %cst, %arg20 = %49, %arg21 = %75) -> (tensor<16x16xf32, #C>, tensor<16x16x!tt.ptr<f16>, #AL>, tensor<16x!tt.ptr<i64>, #BLs1>) {
1081|     %82 = tt.load %arg20 : tensor<16x16x!tt.ptr<f16>, #AL>
1082|     %83 = tt.load %arg21 : tensor<16x!tt.ptr<i64>, #BLs1>
1083|     %84 = tt.expand_dims %83 {axis=1: i32}: tensor<16xi64, #BLs1> -> tensor<16x1xi64, #BL>
1084|     %850 = tt.broadcast %84 : tensor<16x1xi64, #BL> -> tensor<16x16xi64, #BL>
```
**EN:** This function-oriented block defines or enters `indirect_load_shared_layout`. Within it, the test exercises constants, tt.func, masked or vectorized loads, broadcasted scalars or pointers, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `indirect_load_shared_layout` 为核心。测试在其中演示 常量、tt.func、带掩码或向量化的加载、广播后的标量或指针、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1085-1099
```mlir
1085|     %85 = arith.muli %77, %850 : tensor<16x16xi64, #BL>
1086|     %86 = tt.addptr %60, %85 : tensor<16x16x!tt.ptr<f16>, #BL>, tensor<16x16xi64, #BL>
1087|     %87 = tt.load %86 : tensor<16x16x!tt.ptr<f16>, #BL>
1088|     %88 = ttg.convert_layout %82 : tensor<16x16xf16, #AL> -> tensor<16x16xf16, #A>
1089|     %89 = ttg.convert_layout %87 : tensor<16x16xf16, #BL> -> tensor<16x16xf16, #B>
1090|     %90 = tt.dot %88, %89, %arg19 : tensor<16x16xf16, #A> * tensor<16x16xf16, #B> -> tensor<16x16xf32, #C>
1091|     %91 = tt.addptr %arg20, %78 : tensor<16x16x!tt.ptr<f16>, #AL>, tensor<16x16xi32, #AL>
1092|     %92 = tt.addptr %arg21, %c1_i32_splat : tensor<16x!tt.ptr<i64>, #BLs1>, tensor<16xi32, #BLs1>
1093|     scf.yield %90, %91, %92 : tensor<16x16xf32, #C>, tensor<16x16x!tt.ptr<f16>, #AL>, tensor<16x!tt.ptr<i64>, #BLs1>
1094|   } {tt.num_stages = 3 : i32}
1095|   tt.return %79#0 : tensor<16x16xf32, #C>
1096| }
1097| }
1098| 
1099| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining pointer arithmetic, layout conversions, integer multiplications, masked or vectorized loads, dot-product or MMA-style math.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 指针算术、布局转换、整数乘法、带掩码或向量化的加载、点积或 MMA 风格计算。

### Lines 1100-1100
```mlir
1100| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1101-1117
```mlir
1101| 
1102| // CHECK-LABEL: @kernel_yield_constant
1103| // CHECK: ttg.async_copy_global_to_local
1104| // CHECK: scf.for
1105| // CHECK: ttg.memdesc_index
1106| // CHECK: ttg.async_copy_global_to_local
1107| // CHECK: tt.return
1108| 
1109| // AMD-LABEL: @kernel_yield_constant
1110| // AMD: tt.load
1111| // AMD: ttg.memdesc_index
1112| // AMD: ttg.local_store
1113| // AMD: scf.for
1114| // AMD: tt.load
1115| // AMD: ttg.memdesc_index
1116| // AMD: ttg.local_store
1117| // AMD: tt.return
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 1118-1119
```mlir
1118| #blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
1119| #mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [2, 2], instrShape = [16, 8]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1120-1120
```mlir
1120| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 1121-1138
```mlir
1121|   tt.func public @kernel_yield_constant(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg2: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 16 : i32}, %arg3: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 16 : i32}, %arg4: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 16 : i32}, %arg5: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 16 : i32}, %arg6: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 16 : i32}) {
1122|     %cst = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #mma>
1123|     %cst1 = arith.constant dense<1.000000e+00> : tensor<32x32xf32, #mma>
1124|     %c0_i32 = arith.constant 0 : i32
1125|     %c1_i32 = arith.constant 1 : i32
1126|     %cst_0 = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #blocked>
1127|     %c32_i32 = arith.constant 32 : i32
1128|     %c31_i32 = arith.constant 31 : i32
1129|     %cst_1 = arith.constant dense<2.000000e+00> : tensor<32x32xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>>
1130|     %0 = tt.get_program_id x : i32
1131|     %7 = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
1132|     %12 = arith.addi %arg4, %c31_i32 : i32
1133|     %13 = arith.divsi %12, %c32_i32 : i32
1134|     %14 = tt.expand_dims %7 {axis = 1 : i32} : tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<32x1xi32, #blocked>
1135|     %22 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<32x32x!tt.ptr<f32>, #blocked>
1136|     %34 = tt.splat %arg1 : !tt.ptr<f32> -> tensor<32x32x!tt.ptr<f32>, #blocked>
1137|     %42 = scf.for %arg7 = %c0_i32 to %13 step %c1_i32 iter_args(%arg8 = %cst) -> (tensor<32x32xf32, #mma>)  : i32 {
1138|       %43 = arith.muli %arg7, %c32_i32 : i32
```
**EN:** This function-oriented block defines or enters `kernel_yield_constant`. Within it, the test exercises constants, tt.func, broadcasted scalars or pointers, program IDs, lane/block index ranges, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `kernel_yield_constant` 为核心。测试在其中演示 常量、tt.func、广播后的标量或指针、程序 ID、lane/block 索引范围，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1139-1156
```mlir
1139|       %44 = arith.muli %43, %arg5 : i32
1140|       %45 = tt.splat %44 : i32 -> tensor<32x32xi32, #blocked>
1141|       %46 = tt.addptr %22, %45 : tensor<32x32x!tt.ptr<f32>, #blocked>, tensor<32x32xi32, #blocked>
1142|       %47 = arith.subi %arg4, %43 : i32
1143|       %48 = tt.splat %47 : i32 -> tensor<32x1xi32, #blocked>
1144|       %49 = arith.cmpi slt, %14, %48 : tensor<32x1xi32, #blocked>
1145|       %50 = tt.broadcast %49 : tensor<32x1xi1, #blocked> -> tensor<32x32xi1, #blocked>
1146|       %51 = tt.load %46, %50, %cst_0 : tensor<32x32x!tt.ptr<f32>, #blocked>
1147|       %52 = ttg.convert_layout %51 : tensor<32x32xf32, #blocked> -> tensor<32x32xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>>
1148|       %53 = tt.dot %cst_1, %52, %arg8, inputPrecision = tf32 : tensor<32x32xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>> * tensor<32x32xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>> -> tensor<32x32xf32, #mma>
1149|       %54 = ttg.convert_layout %53 : tensor<32x32xf32, #mma> -> tensor<32x32xf32, #blocked>
1150|       tt.store %34, %54 : tensor<32x32x!tt.ptr<f32>, #blocked>
1151|       scf.yield %cst1 : tensor<32x32xf32, #mma>
1152|     }
1153|     tt.return
1154|   }
1155| }
1156| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining broadcasted scalars or pointers, layout conversions, integer multiplications, pointer arithmetic, arith.subi.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 广播后的标量或指针、布局转换、整数乘法、指针算术、arith.subi。

### Lines 1158-1159
```mlir
1158| 
1159| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1159-1174
```mlir
1159| 
1160| // CHECK-LABEL: @add_kernel
1161| // CHECK-DAG: %[[CONSTANT_0:.*]] = arith.constant 0 : i32
1162| // CHECK-DAG: %[[CONSTANT_1:.*]] = arith.constant 1 : i32
1163| // CHECK:   %[[ABUFFER:.*]] = ttg.local_alloc
1164| // CHECK:   %[[BBUFFER:.*]] = ttg.local_alloc
1165| // CHECK:   %[[A0BUFFER:.*]] = ttg.memdesc_index %[[ABUFFER]]{{\[}}%[[CONSTANT_0]]{{\]}}
1166| // CHECK:   ttg.async_copy_global_to_local {{.*}}, %[[A0BUFFER]]
1167| // CHECK:   %[[B0BUFFER:.*]] = ttg.memdesc_index %[[BBUFFER]]{{\[}}%[[CONSTANT_0]]{{\]}}
1168| // CHECK:   ttg.async_copy_global_to_local {{.*}}, %[[B0BUFFER]]
1169| // CHECK:   %[[A1BUFFER:.*]] = ttg.memdesc_index %[[ABUFFER]]{{\[}}%[[CONSTANT_1]]{{\]}}
1170| // CHECK:   ttg.async_copy_global_to_local {{.*}}, %[[A1BUFFER]]
1171| // CHECK:   %[[B1BUFFER:.*]] = ttg.memdesc_index %[[BBUFFER]]{{\[}}%[[CONSTANT_1]]{{\]}}
1172| // CHECK:   ttg.async_copy_global_to_local {{.*}}, %[[B1BUFFER]]
1173| // CHECK:   scf.for
1174| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @add_kernel anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @add_kernel 这样的标签用于锚定匹配范围。

### Lines 1175-1187
```mlir
1175| // AMD-LABEL:  tt.func public @add_kernel
1176| // AMD:  %[[LOAD_11:.*]] = tt.load %{{.*}}, %{{.*}}
1177| // AMD:  %[[ADDPTR_12:.*]] = tt.addptr %{{.*}}, %{{.*}}
1178| // AMD:  %[[LOAD_13:.*]] = tt.load %[[ADDPTR_12]], %{{.*}}
1179| // AMD:  %[[ADDI_14:.*]] = arith.addi %{{.*}}, %{{.*}}
1180| // AMD:  %[[SPLAT_15:.*]] = tt.splat %[[ADDI_14]]
1181| // AMD:  %[[ADDI_16:.*]] = arith.addi %[[SPLAT_15]], %{{.*}}
1182| // AMD:  %[[CMPI_17:.*]] = arith.cmpi slt, %[[ADDI_16]], %{{.*}}
1183| // AMD:  %[[ADDPTR_18:.*]] = tt.addptr %{{.*}}, %[[ADDI_16]]
1184| // AMD:  %[[LOAD_19:.*]] = tt.load %[[ADDPTR_18]], %[[CMPI_17]]
1185| // AMD:  %[[ADDPTR_20:.*]] = tt.addptr %{{.*}}, %[[ADDI_16]]
1186| // AMD:  %[[LOAD_21:.*]] = tt.load %[[ADDPTR_20]], %[[CMPI_17]]
1187| // AMD:  scf.for
```
**EN:** This function-oriented block defines or enters `add_kernel`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `add_kernel` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1188-1188
```mlir
1188| #blocked = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1189-1189
```mlir
1189| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 1190-1207
```mlir
1190|   tt.func public @add_kernel(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg2: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg3: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 16 : i32}) {
1191|     %c1024_i32 = arith.constant 1024 : i32
1192|     %c0_i32 = arith.constant 0 : i32
1193|     %c1016800_i32 = arith.constant 1016800 : i32
1194|     %0 = tt.get_program_id x : i32
1195|     %1 = arith.muli %0, %c1016800_i32 : i32
1196|     %2 = tt.make_range {end = 1024 : i32, start = 0 : i32} : tensor<1024xi32, #blocked>
1197|     %3 = tt.splat %arg3 : i32 -> tensor<1024xi32, #blocked>
1198|     %4 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<1024x!tt.ptr<f32>, #blocked>
1199|     %5 = tt.splat %arg1 : !tt.ptr<f32> -> tensor<1024x!tt.ptr<f32>, #blocked>
1200|     %6 = tt.splat %arg2 : !tt.ptr<f32> -> tensor<1024x!tt.ptr<f32>, #blocked>
1201|     scf.for %arg4 = %c0_i32 to %c1016800_i32 step %c1024_i32  : i32 {
1202|       %7 = arith.addi %1, %arg4 : i32
1203|       %8 = tt.splat %7 : i32 -> tensor<1024xi32, #blocked>
1204|       %9 = arith.addi %8, %2 : tensor<1024xi32, #blocked>
1205|       %10 = arith.cmpi slt, %9, %3 : tensor<1024xi32, #blocked>
1206|       %11 = tt.addptr %4, %9 : tensor<1024x!tt.ptr<f32>, #blocked>, tensor<1024xi32, #blocked>
1207|       %12 = tt.load %11, %10 : tensor<1024x!tt.ptr<f32>, #blocked>
```
**EN:** This function-oriented block defines or enters `add_kernel`. Within it, the test exercises broadcasted scalars or pointers, constants, tt.func, integer additions, program IDs, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `add_kernel` 为核心。测试在其中演示 广播后的标量或指针、常量、tt.func、整数加法、程序 ID，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1208-1218
```mlir
1208|       %13 = tt.addptr %5, %9 : tensor<1024x!tt.ptr<f32>, #blocked>, tensor<1024xi32, #blocked>
1209|       %14 = tt.load %13, %10 : tensor<1024x!tt.ptr<f32>, #blocked>
1210|       %15 = arith.addf %12, %14 : tensor<1024xf32, #blocked>
1211|       %16 = tt.addptr %6, %9 : tensor<1024x!tt.ptr<f32>, #blocked>, tensor<1024xi32, #blocked>
1212|       tt.store %16, %15, %10 : tensor<1024x!tt.ptr<f32>, #blocked>
1213|     } {tt.num_stages = 3 : i32}
1214|     tt.return
1215|   }
1216| }
1217| 
1218| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on pointer arithmetic, masked or vectorized loads, floating-point additions, masked or vectorized stores, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 指针算术、带掩码或向量化的加载、浮点加法、带掩码或向量化的存储、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1219-1219
```mlir
1219| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1220-1237
```mlir
1220| 
1221| // CHECK-LABEL: @nested_loops
1222| // CHECK: tt.addptr %{{.*}}, {{.*}}
1223| // CHECK: %[[NEXT_BUFFER_1:.*]] = tt.addptr %{{.*}}, {{.*}}
1224| // CHECK: scf.for
1225| // CHECK:   %[[LOAD_1:.*]] = tt.load %[[NEXT_BUFFER_1]]
1226| // CHECK:   %[[BUFFER_2:.*]] = ttg.local_alloc %[[LOAD_1]]
1227| // CHECK:   %[[TRANS:.*]] = ttg.memdesc_trans %[[BUFFER_2]]
1228| // CHECK:   %[[LOCAL_LOAD_1:.*]] = ttg.local_load %[[TRANS]]
1229| // CHECK:   %[[BUFFER_1:.*]] = ttg.local_alloc : ()
1230| // CHECK:   %[[SUBVIEW_1:.*]] = ttg.memdesc_index %[[BUFFER_1]]
1231| // CHECK:   %[[ASYNC_COPY_1:.*]] = ttg.async_copy_global_to_local %[[NEXT_BUFFER_1]], %[[SUBVIEW_1]]
1232| // CHECK:   ttg.async_commit_group tokens %[[ASYNC_COPY_1]]
1233| // CHECK:   %[[SUBVIEW_2:.*]] = ttg.memdesc_index %[[BUFFER_1]]
1234| // CHECK:   %[[ASYNC_COPY_2:.*]] = ttg.async_copy_global_to_local %[[NEXT_BUFFER_1]], %[[SUBVIEW_2]]
1235| // CHECK:   ttg.async_commit_group tokens %[[ASYNC_COPY_2]]
1236| // CHECK:   scf.for
1237| // CHECK:     ttg.async_wait
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @nested_loops anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @nested_loops 这样的标签用于锚定匹配范围。

### Lines 1238-1246
```mlir
1238| // CHECK:     ttg.memdesc_index %[[BUFFER_1]]
1239| // CHECK:     %[[LOCAL_LOAD_2:.*]] = ttg.local_load
1240| // CHECK:     %[[DOT:.*]] = tt.dot %[[LOCAL_LOAD_2]], %[[LOCAL_LOAD_1]]
1241| // CHECK:     %[[CONVERT_LAYOUT_3:.*]] = ttg.convert_layout %[[DOT]]
1242| // CHECK:     %[[SUBVIEW_4:.*]] = ttg.memdesc_index %[[BUFFER_1]]
1243| // CHECK:     %[[ASYNC_COPY_3:.*]] = ttg.async_copy_global_to_local %[[NEXT_BUFFER_1]], %[[SUBVIEW_4]]
1244| // CHECK:     ttg.async_commit_group tokens %[[ASYNC_COPY_3]]
1245| // CHECK: ttg.local_dealloc %[[BUFFER_1]]
1246| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 1247-1257
```mlir
1247| // AMD-LABEL:  tt.func public @nested_loops
1248| // AMD-NOT:  ttg.local_alloc
1249| // AMD:      scf.for
1250| // AMD:        ttg.local_alloc
1251| // AMD:        scf.for
1252| // AMD:          ttg.local_load
1253| // AMD:          tt.dot
1254| // AMD:          ttg.local_store
1255| // AMD:          scf.yield
1256| // AMD:        ttg.local_dealloc
1257| 
```
**EN:** This function-oriented block defines or enters `nested_loops`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `nested_loops` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1258-1273
```mlir
1258| // AMD_3_STAGES-LABEL:  tt.func public @nested_loops
1259| // AMD_3_STAGES-NOT:  ttg.local_alloc
1260| // AMD_3_STAGES:      scf.for
1261| // AMD_3_STAGES:        ttg.local_alloc
1262| // AMD_3_STAGES:        tt.load
1263| // AMD_3_STAGES:        ttg.local_store
1264| // AMD_3_STAGES:        tt.load
1265| // AMD_3_STAGES:        ttg.local_store
1266| // AMD_3_STAGES:        scf.for
1267| // AMD_3_STAGES:          tt.load
1268| // AMD_3_STAGES:          ttg.local_load
1269| // AMD_3_STAGES:          tt.dot
1270| // AMD_3_STAGES:          ttg.local_store
1271| // AMD_3_STAGES:          scf.yield
1272| // AMD_3_STAGES:        ttg.local_dealloc
1273| 
```
**EN:** This function-oriented block defines or enters `nested_loops`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `nested_loops` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1274-1278
```mlir
1274| #blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 4], warpsPerCTA = [2, 1], order = [1, 0]}>
1275| #mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [1, 2], instrShape = [16, 8]}>
1276| #shared = #ttg.swizzled_shared<{vec = 4, perPhase = 2, maxPhase = 4, order = [1, 0]}>
1277| #shared1 = #ttg.swizzled_shared<{vec = 4, perPhase = 2, maxPhase = 4, order = [0, 1]}>
1278| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1279-1279
```mlir
1279| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 2 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 1280-1297
```mlir
1280|   tt.func public @nested_loops(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}) {
1281|     %cst = arith.constant dense<0.000000e+00> : tensor<16x16xf32, #mma>
1282|     %c1_i32 = arith.constant 1 : i32
1283|     %c2_i32 = arith.constant 2 : i32
1284|     %c0_i32 = arith.constant 0 : i32
1285|     %cst_0 = arith.constant dense<16> : tensor<16x1xi32, #blocked>
1286|     %0 = tt.make_range {end = 16 : i32, start = 0 : i32} : tensor<16xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
1287|     %1 = tt.expand_dims %0 {axis = 1 : i32} : tensor<16xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<16x1xi32, #blocked>
1288|     %2 = arith.muli %1, %cst_0 : tensor<16x1xi32, #blocked>
1289|     %3 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<16x1x!tt.ptr<f32>, #blocked>
1290|     %4 = tt.addptr %3, %2 : tensor<16x1x!tt.ptr<f32>, #blocked>, tensor<16x1xi32, #blocked>
1291|     %5 = tt.make_range {end = 16 : i32, start = 0 : i32} : tensor<16xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
1292|     %6 = tt.expand_dims %5 {axis = 0 : i32} : tensor<16xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x16xi32, #blocked>
1293|     %7 = tt.broadcast %4 : tensor<16x1x!tt.ptr<f32>, #blocked> -> tensor<16x16x!tt.ptr<f32>, #blocked>
1294|     %8 = tt.broadcast %6 : tensor<1x16xi32, #blocked> -> tensor<16x16xi32, #blocked>
1295|     %9 = tt.addptr %7, %8 : tensor<16x16x!tt.ptr<f32>, #blocked>, tensor<16x16xi32, #blocked>
1296|     scf.for %arg1 = %c0_i32 to %c2_i32 step %c1_i32  : i32 {
1297|       %10 = tt.load %9 : tensor<16x16x!tt.ptr<f32>, #blocked>
```
**EN:** This function-oriented block defines or enters `nested_loops`. Within it, the test exercises constants, tt.func, lane/block index ranges, shape expansion, pointer arithmetic, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `nested_loops` 为核心。测试在其中演示 常量、tt.func、lane/block 索引范围、形状扩展、指针算术，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1298-1312
```mlir
1298|       %11 = ttg.local_alloc %10 : (tensor<16x16xf32, #blocked>) -> !ttg.memdesc<16x16xf32, #shared, #smem>
1299|       %12 = ttg.memdesc_trans %11 {order = array<i32: 1, 0>} : !ttg.memdesc<16x16xf32, #shared, #smem> -> !ttg.memdesc<16x16xf32, #shared1, #smem>
1300|       %13 = ttg.local_load %12 : !ttg.memdesc<16x16xf32, #shared1, #smem> -> tensor<16x16xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>>
1301|       scf.for %arg2 = %c0_i32 to %c2_i32 step %c1_i32  : i32 {
1302|         %14 = tt.load %9 : tensor<16x16x!tt.ptr<f32>, #blocked>
1303|         %15 = ttg.convert_layout %14 : tensor<16x16xf32, #blocked> -> tensor<16x16xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>>
1304|         %16 = tt.dot %15, %13, %cst, inputPrecision = tf32 : tensor<16x16xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>> * tensor<16x16xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>> -> tensor<16x16xf32, #mma>
1305|         %17 = ttg.convert_layout %16 : tensor<16x16xf32, #mma> -> tensor<16x16xf32, #blocked>
1306|         tt.store %9, %17 : tensor<16x16x!tt.ptr<f32>, #blocked>
1307|       }
1308|     }
1309|     tt.return
1310|   }
1311| }
1312| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining layout conversions, shared/local memory allocation, ttg.memdesc_trans, local/shared memory loads, structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 布局转换、共享/本地内存分配、ttg.memdesc_trans、本地/共享内存加载、结构化循环。

### Lines 1313-1313
```mlir
1313| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1314-1315
```mlir
1314| 
1315|   // CHECK-LABEL: @int4_matmul_ampere
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @int4_matmul_ampere anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @int4_matmul_ampere 这样的标签用于锚定匹配范围。

### Lines 1316-1322
```mlir
1316| #blocked = #ttg.blocked<{sizePerThread = [16, 1], threadsPerWarp = [4, 8], warpsPerCTA = [1, 8], order = [0, 1]}>
1317| #blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [8, 1], order = [1, 0]}>
1318| #blocked2 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [1, 32], warpsPerCTA = [8, 1], order = [1, 0]}>
1319| #blocked3 = #ttg.blocked<{sizePerThread = [16, 1, 2], threadsPerWarp = [4, 8, 1], warpsPerCTA = [1, 8, 1], order = [2, 0, 1]}>
1320| #blocked4 = #ttg.blocked<{sizePerThread = [16, 2, 1], threadsPerWarp = [4, 1, 8], warpsPerCTA = [1, 1, 8], order = [1, 0, 2]}>
1321| #blocked5 = #ttg.blocked<{sizePerThread = [32, 1], threadsPerWarp = [4, 8], warpsPerCTA = [1, 8], order = [0, 1]}>
1322| #mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [1, 8], instrShape = [16, 8]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1323-1323
```mlir
1323| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 1324-1341
```mlir
1324|   tt.func public @int4_matmul_ampere(
1325|     %arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32},
1326|     %arg1: !tt.ptr<i8> {tt.divisibility = 16 : i32}
1327|   ) -> tensor<16x256xf32, #mma> {
1328|     %cst = arith.constant dense<64> : tensor<64x256xi32, #blocked>
1329|     %cst_0 = arith.constant dense<128> : tensor<16x128xi32, #blocked1>
1330|     %c256_i32 = arith.constant 256 : i32
1331|     %c16_i32 = arith.constant 16 : i32
1332|     %c128_i32 = arith.constant 128 : i32
1333|     %cst_1 = arith.constant dense<0.000000e+00> : tensor<16x128xf16, #blocked1>
1334|     %c0_i32 = arith.constant 0 : i32
1335|     %c1_i32 = arith.constant 1 : i32
1336|     %c255_i32 = arith.constant 255 : i32
1337|     %c15_i32 = arith.constant 15 : i32
1338|     %cst_2 = arith.constant dense<4> : tensor<64x256xi8, #blocked>
1339|     %cst_3 = arith.constant dense<0.000000e+00> : tensor<16x256xf32, #mma>
1340| 
1341|     %35 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
```
**EN:** This function-oriented block defines or enters `int4_matmul_ampere`. Within it, the test exercises constants, tt.func, lane/block index ranges, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `int4_matmul_ampere` 为核心。测试在其中演示 常量、tt.func、lane/block 索引范围，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1342-1353
```mlir
1342|     %36 = tt.expand_dims %35 {axis = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = #blocked1}>> -> tensor<1x128xi32, #blocked1>
1343|     %38 = tt.broadcast %36 : tensor<1x128xi32, #blocked1> -> tensor<16x128xi32, #blocked1>
1344|     %40 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<16x128x!tt.ptr<f16>, #blocked1>
1345|     %41 = tt.addptr %40, %38 : tensor<16x128x!tt.ptr<f16>, #blocked1>, tensor<16x128xi32, #blocked1>
1346| 
1347|     %42 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
1348|     %43 = tt.expand_dims %42 {axis = 1 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<64x1xi32, #blocked>
1349|     %47 = tt.broadcast %43 : tensor<64x1xi32, #blocked> -> tensor<64x256xi32, #blocked>
1350|     %50 = tt.splat %arg1 : !tt.ptr<i8> -> tensor<64x256x!tt.ptr<i8>, #blocked>
1351|     %51 = tt.addptr %50, %47 : tensor<64x256x!tt.ptr<i8>, #blocked>, tensor<64x256xi32, #blocked>
1352| 
1353|     // Check that both loads in the loop are pipelined.
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shape expansion, tensor broadcasting, broadcasted scalars or pointers, pointer arithmetic, lane/block index ranges. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 形状扩展、张量广播、广播后的标量或指针、指针算术、lane/block 索引范围。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1354-1371
```mlir
1354|     // CHECK: scf.for
1355|     // CHECK-NOT: tt.load
1356|     // CHECK: ttg.async_copy_global_to_local
1357|     // CHECK-NOT: tt.load
1358|     // CHECK: ttg.async_copy_global_to_local
1359|     // CHECK-NOT: tt.load
1360|     // CHECK: scf.yield
1361|     %54:3 = scf.for %arg9 = %c0_i32 to %c16_i32 step %c1_i32 iter_args(%arg10 = %cst_3, %arg11 = %41, %arg12 = %51) -> (tensor<16x256xf32, #mma>, tensor<16x128x!tt.ptr<f16>, #blocked1>, tensor<64x256x!tt.ptr<i8>, #blocked>)  : i32 {
1362|       %78 = tt.load %arg11 : tensor<16x128x!tt.ptr<f16>, #blocked1>
1363|       %79 = tt.load %arg12 : tensor<64x256x!tt.ptr<i8>, #blocked>
1364|       %80 = arith.shli %79, %cst_2 : tensor<64x256xi8, #blocked>
1365|       %81 = arith.shrsi %80, %cst_2 : tensor<64x256xi8, #blocked>
1366|       %82 = arith.shrsi %79, %cst_2 : tensor<64x256xi8, #blocked>
1367|       %83 = arith.sitofp %81 : tensor<64x256xi8, #blocked> to tensor<64x256xf16, #blocked>
1368|       %84 = arith.sitofp %82 : tensor<64x256xi8, #blocked> to tensor<64x256xf16, #blocked>
1369|       %85 = tt.join %83, %84 : tensor<64x256xf16, #blocked> -> tensor<64x256x2xf16, #blocked3>
1370|       %86 = tt.trans %85 {order = array<i32: 0, 2, 1>} : tensor<64x256x2xf16, #blocked3> -> tensor<64x2x256xf16, #blocked4>
1371|       %87 = tt.reshape %86 : tensor<64x2x256xf16, #blocked4> -> tensor<128x256xf16, #blocked5>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining masked or vectorized loads, arith.shrsi, arith.sitofp, structured loops, arith.shli.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 带掩码或向量化的加载、arith.shrsi、arith.sitofp、结构化循环、arith.shli。

### Lines 1372-1383
```mlir
1372|       %88 = ttg.convert_layout %78 : tensor<16x128xf16, #blocked1> -> tensor<16x128xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
1373|       %89 = ttg.convert_layout %87 : tensor<128x256xf16, #blocked5> -> tensor<128x256xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>
1374|       %90 = tt.dot %88, %89, %arg10 : tensor<16x128xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<128x256xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<16x256xf32, #mma>
1375|       %91 = tt.addptr %arg11, %cst_0 : tensor<16x128x!tt.ptr<f16>, #blocked1>, tensor<16x128xi32, #blocked1>
1376|       %92 = tt.addptr %arg12, %cst : tensor<64x256x!tt.ptr<i8>, #blocked>, tensor<64x256xi32, #blocked>
1377|       scf.yield %90, %91, %92 : tensor<16x256xf32, #mma>, tensor<16x128x!tt.ptr<f16>, #blocked1>, tensor<64x256x!tt.ptr<i8>, #blocked>
1378|     }
1379|     tt.return %54#0 : tensor<16x256xf32, #mma>
1380|   }
1381| }
1382| 
1383| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining layout conversions, pointer arithmetic, dot-product or MMA-style math, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 布局转换、指针算术、点积或 MMA 风格计算、循环/分支产出值、tt.return。

### Lines 1384-1384
```mlir
1384| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1385-1388
```mlir
1385| 
1386| // This test triggered some failure in the verifier, so we only
1387| // included a simple check for the kernel name.
1388| // COMMON-LABEL: @load_convert_layout
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1389-1397
```mlir
1389| #AL = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
1390| #BL = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
1391| #ALs0 = #ttg.slice<{parent=#AL, dim=0}>
1392| #BLs0 = #ttg.slice<{parent=#BL, dim=0}>
1393| #BLs1 = #ttg.slice<{parent=#BL, dim=1}>
1394| #C = #ttg.nvidia_mma<{versionMajor = 2, warpsPerCTA = [4, 1], instrShape = [16, 8]}>
1395| #A = #ttg.dot_op<{opIdx = 0, parent = #C, kWidth=2}>
1396| #B = #ttg.dot_op<{opIdx = 1, parent = #C, kWidth=2}>
1397| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1398-1398
```mlir
1398| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 1399-1416
```mlir
1399| tt.func @load_convert_layout(%77: tensor<16x16xi64, #BL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.constancy = dense<[16, 16]> : tensor<2xi32>},
1400|                    %76: index,
1401|                    %49: tensor<16x16x!tt.ptr<f16>, #AL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 2]> : tensor<2xi32>},
1402|                    %75: tensor<16x!tt.ptr<i64>, #BLs1>,
1403|                    %78: tensor<16x16xi32, #AL> {tt.constancy = dense<[16, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>},
1404|                    %60: tensor<16x16x!tt.ptr<f16>, #BL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>}) -> tensor<16x16xf32, #C> {
1405|   %1 = tt.make_range {end = 16 : i32, start = 0 : i32} : tensor<16xi32, #BLs1>
1406|   %cst = arith.constant dense<0.000000e+00> : tensor<16x16xf32, #C>
1407|   %cst_0 = arith.constant dense<2> : tensor<16xi32, #BLs1>
1408|   %c4_i32 = arith.constant 4 : i32
1409|   %c1 = arith.constant 1 : index
1410|   %c0 = arith.constant 0 : index
1411|   %c0_i64 = arith.constant 0 : i64
1412|   %c1_i32 = arith.constant 1 : i32
1413|   %c1_i32_splat = tt.splat %c1_i32 : i32 -> tensor<16xi32, #BLs1>
1414|   %15 = arith.cmpi slt, %1, %cst_0 : tensor<16xi32, #BLs1>
1415|   %79:3 = scf.for %arg18 = %c0 to %76 step %c1 iter_args(%arg19 = %cst, %arg20 = %49, %arg21 = %75) -> (tensor<16x16xf32, #C>, tensor<16x16x!tt.ptr<f16>, #AL>, tensor<16x!tt.ptr<i64>, #BLs1>) {
1416|     %82 = tt.load %arg20 : tensor<16x16x!tt.ptr<f16>, #AL>
```
**EN:** This function-oriented block defines or enters `load_convert_layout`. Within it, the test exercises constants, tt.func, lane/block index ranges, broadcasted scalars or pointers, integer comparisons, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `load_convert_layout` 为核心。测试在其中演示 常量、tt.func、lane/block 索引范围、广播后的标量或指针、整数比较，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1417-1434
```mlir
1417|     %83 = tt.load %arg21, %15 : tensor<16x!tt.ptr<i64>, #BLs1>
1418|     %84 = tt.expand_dims %83 {axis=1: i32}: tensor<16xi64, #BLs1> -> tensor<16x1xi64, #BL>
1419|     %850 = tt.broadcast %84 : tensor<16x1xi64, #BL> -> tensor<16x16xi64, #BL>
1420|     %85 = arith.muli %77, %850 : tensor<16x16xi64, #BL>
1421|     %86 = tt.addptr %60, %85 : tensor<16x16x!tt.ptr<f16>, #BL>, tensor<16x16xi64, #BL>
1422|     %87 = tt.load %86 : tensor<16x16x!tt.ptr<f16>, #BL>
1423|     %88 = ttg.convert_layout %82 : tensor<16x16xf16, #AL> -> tensor<16x16xf16, #A>
1424|     %89 = ttg.convert_layout %87 : tensor<16x16xf16, #BL> -> tensor<16x16xf16, #B>
1425|     %90 = tt.dot %88, %89, %arg19 : tensor<16x16xf16, #A> * tensor<16x16xf16, #B> -> tensor<16x16xf32, #C>
1426|     %91 = tt.addptr %arg20, %78 : tensor<16x16x!tt.ptr<f16>, #AL>, tensor<16x16xi32, #AL>
1427|     %92 = tt.addptr %arg21, %c1_i32_splat : tensor<16x!tt.ptr<i64>, #BLs1>, tensor<16xi32, #BLs1>
1428|     scf.yield %90, %91, %92 : tensor<16x16xf32, #C>, tensor<16x16x!tt.ptr<f16>, #AL>, tensor<16x!tt.ptr<i64>, #BLs1>
1429|   } {tt.num_stages = 3 : i32}
1430|   tt.return %79#0 : tensor<16x16xf32, #C>
1431| }
1432| }
1433| 
1434| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining pointer arithmetic, masked or vectorized loads, layout conversions, shape expansion, tensor broadcasting.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 指针算术、带掩码或向量化的加载、布局转换、形状扩展、张量广播。

### Lines 1435-1435
```mlir
1435| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1436-1439
```mlir
1436| 
1437| // This test captured some ICE in MatmulLoopPipeline pass, so we only
1438| // included a simple check for the kernel name.
1439| // COMMON-LABEL: @matmul_indirect_pipeline
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1440-1441
```mlir
1440| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [1, 2], order = [0, 1]}>
1441| #mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [2, 1], instrShape = [16, 8]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1442-1442
```mlir
1442| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 2 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 1443-1460
```mlir
1443|   tt.func public @matmul_indirect_pipeline(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<i64> {tt.divisibility = 16 : i32}, %arg2: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg3: !tt.ptr<f32> {tt.divisibility = 16 : i32}) {
1444|     %cst = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #mma>
1445|     %c1_i32 = arith.constant 1 : i32
1446|     %c2_i32 = arith.constant 2 : i32
1447|     %c0_i32 = arith.constant 0 : i32
1448|     %0 = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
1449|     %1 = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
1450|     %2 = tt.expand_dims %1 {axis = 1 : i32} : tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<32x1xi32, #blocked>
1451|     %3 = tt.expand_dims %0 {axis = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x32xi32, #blocked>
1452|     %4 = tt.broadcast %2 : tensor<32x1xi32, #blocked> -> tensor<32x32xi32, #blocked>
1453|     %5 = tt.broadcast %3 : tensor<1x32xi32, #blocked> -> tensor<32x32xi32, #blocked>
1454|     %6 = arith.addi %4, %5 : tensor<32x32xi32, #blocked>
1455|     %7 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<32x32x!tt.ptr<f32>, #blocked>
1456|     %8 = tt.addptr %7, %6 : tensor<32x32x!tt.ptr<f32>, #blocked>, tensor<32x32xi32, #blocked>
1457|     %9 = tt.load %8 : tensor<32x32x!tt.ptr<f32>, #blocked>
1458|     %10 = tt.splat %arg3 : !tt.ptr<f32> -> tensor<32x32x!tt.ptr<f32>, #blocked>
1459|     %11 = tt.addptr %10, %6 : tensor<32x32x!tt.ptr<f32>, #blocked>, tensor<32x32xi32, #blocked>
1460|     %12 = tt.splat %arg1 : !tt.ptr<i64> -> tensor<32x!tt.ptr<i64>, #ttg.slice<{dim = 0, parent = #blocked}>>
```
**EN:** This function-oriented block defines or enters `matmul_indirect_pipeline`. Within it, the test exercises constants, broadcasted scalars or pointers, tt.func, lane/block index ranges, shape expansion, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `matmul_indirect_pipeline` 为核心。测试在其中演示 常量、广播后的标量或指针、tt.func、lane/block 索引范围、形状扩展，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1461-1478
```mlir
1461|     %13 = tt.addptr %12, %0 : tensor<32x!tt.ptr<i64>, #ttg.slice<{dim = 0, parent = #blocked}>>, tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
1462|     %14 = tt.splat %arg2 : !tt.ptr<f32> -> tensor<32x!tt.ptr<f32>, #ttg.slice<{dim = 0, parent = #blocked}>>
1463|     scf.for %arg4 = %c0_i32 to %c2_i32 step %c1_i32  : i32 {
1464|       %15 = tt.load %13 : tensor<32x!tt.ptr<i64>, #ttg.slice<{dim = 0, parent = #blocked}>>
1465|       %16 = tt.addptr %14, %15 : tensor<32x!tt.ptr<f32>, #ttg.slice<{dim = 0, parent = #blocked}>>, tensor<32xi64, #ttg.slice<{dim = 0, parent = #blocked}>>
1466|       %17 = tt.load %16 : tensor<32x!tt.ptr<f32>, #ttg.slice<{dim = 0, parent = #blocked}>>
1467|       %18 = tt.expand_dims %17 {axis = 0 : i32} : tensor<32xf32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x32xf32, #blocked>
1468|       %19 = tt.broadcast %18 : tensor<1x32xf32, #blocked> -> tensor<32x32xf32, #blocked>
1469|       %20 = arith.addf %9, %19 : tensor<32x32xf32, #blocked>
1470|       %21 = ttg.convert_layout %9 : tensor<32x32xf32, #blocked> -> tensor<32x32xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>>
1471|       %22 = ttg.convert_layout %20 : tensor<32x32xf32, #blocked> -> tensor<32x32xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>>
1472|       %23 = tt.dot %21, %22, %cst, inputPrecision = tf32 : tensor<32x32xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>> * tensor<32x32xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>> -> tensor<32x32xf32, #mma>
1473|       %24 = ttg.convert_layout %23 : tensor<32x32xf32, #mma> -> tensor<32x32xf32, #blocked>
1474|       tt.store %11, %24 : tensor<32x32x!tt.ptr<f32>, #blocked>
1475|     } {tt.num_stages = 3 : i32}
1476|     tt.return
1477|   }
1478| }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining layout conversions, pointer arithmetic, masked or vectorized loads, broadcasted scalars or pointers, structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 布局转换、指针算术、带掩码或向量化的加载、广播后的标量或指针、结构化循环。

### Lines 1480-1481
```mlir
1480| 
1481| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1481-1483
```mlir
1481| 
1482| // COMMON-LABEL: @dont_pipeline_128x1
1483| // AMD-NOT: local_load{{.*}}128x1
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1484-1484
```mlir
1484| // CHECK: local_load{{.*}}128x1
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 1485-1486
```mlir
1485| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
1486| #mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 8]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1487-1487
```mlir
1487| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 1488-1505
```mlir
1488|   tt.func public @dont_pipeline_128x1(%arg6: !tt.ptr<i32> {tt.divisibility = 16 : i32}) {
1489|     %cst = arith.constant dense<0.000000e+00> : tensor<128x64xf32, #mma>
1490|     %c128_i32 = arith.constant 128 : i32
1491|     %c0_i32 = arith.constant 0 : i32
1492|     %c64_i32 = arith.constant 64 : i32
1493|     %cst_4 = arith.constant dense<-1.000000e+30> : tensor<128xf32, #ttg.slice<{dim = 1, parent = #mma}>>
1494| 
1495|     %99:1 = scf.for %arg25 = %c0_i32 to %c128_i32 step %c64_i32 iter_args(%arg31 = %cst_4) -> (tensor<128xf32, #ttg.slice<{dim = 1, parent = #mma}>>)  : i32 {
1496|       %94 = tt.splat %arg6 : !tt.ptr<i32> -> tensor<128x1x!tt.ptr<i32>, #blocked>
1497|       %151 = tt.load %94 : tensor<128x1x!tt.ptr<i32>, #blocked>
1498|       %161 = ttg.convert_layout %151 : tensor<128x1xi32, #blocked> -> tensor<128x1xi32, #mma>
1499|       %162 = tt.broadcast %161 : tensor<128x1xi32, #mma> -> tensor<128x64xi32, #mma>
1500|       %170 = arith.sitofp %162 : tensor<128x64xi32, #mma> to tensor<128x64xf32, #mma>
1501| 
1502|       %173 = "tt.reduce"(%170) <{axis = 1 : i32}> ({
1503|       ^bb0(%arg33: f32, %arg34: f32):
1504|         %207 = arith.maxnumf %arg33, %arg34 : f32
1505|         tt.reduce.return %207 : f32
```
**EN:** This function-oriented block defines or enters `dont_pipeline_128x1`. Within it, the test exercises constants, tt.func, structured loops, broadcasted scalars or pointers, masked or vectorized loads, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `dont_pipeline_128x1` 为核心。测试在其中演示 常量、tt.func、结构化循环、广播后的标量或指针、带掩码或向量化的加载，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1506-1521
```mlir
1506|       }) : (tensor<128x64xf32, #mma>) -> tensor<128xf32, #ttg.slice<{dim = 1, parent = #mma}>>
1507|       %175 = arith.maxnumf %arg31, %173 : tensor<128xf32, #ttg.slice<{dim = 1, parent = #mma}>>
1508| 
1509|       %201 = arith.truncf %170 : tensor<128x64xf32, #mma> to tensor<128x64xf16, #mma>
1510|       %202 = ttg.convert_layout %201 : tensor<128x64xf16, #mma> -> tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
1511| 
1512|       %192 = arith.constant dense<0.> : tensor<128x64xf32, #mma>
1513|       %203 = arith.constant dense<0.> : tensor<64x64xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>
1514|       %204 = tt.dot %202, %203, %192 : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<64x64xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<128x64xf32, #mma>
1515| 
1516|       scf.yield %175 : tensor<128xf32, #ttg.slice<{dim = 1, parent = #mma}>>
1517|     }
1518|     tt.return
1519|   }
1520| }
1521| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining constants, arith.maxnumf, arith.truncf, layout conversions, dot-product or MMA-style math.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 常量、arith.maxnumf、arith.truncf、布局转换、点积或 MMA 风格计算。

### Lines 1522-1522
```mlir
1522| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1523-1528
```mlir
1523| 
1524| // Check that the dependencies across ops of different nesting does not cause crash or
1525| // incorrect schedule that fails to pipeline.
1526| // COMMON-LABEL: @matmul_nested_ops
1527| // COMMON: ttg.local_load
1528| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1529-1537
```mlir
1529| #AL = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
1530| #BL = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
1531| #ALs0 = #ttg.slice<{parent=#AL, dim=0}>
1532| #BLs0 = #ttg.slice<{parent=#BL, dim=0}>
1533| #BLs1 = #ttg.slice<{parent=#BL, dim=1}>
1534| #C = #ttg.nvidia_mma<{versionMajor = 2, warpsPerCTA = [4, 1], instrShape = [16, 8]}>
1535| #A = #ttg.dot_op<{opIdx = 0, parent = #C, kWidth=2}>
1536| #B = #ttg.dot_op<{opIdx = 1, parent = #C, kWidth=2}>
1537| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1538-1538
```mlir
1538| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 1539-1556
```mlir
1539| tt.func @matmul_nested_ops(%lb : index, %ub : index, %step : index,
1540|                   %A : !tt.ptr<f16> {tt.divisibility = 16 : i32},
1541|                   %B : !tt.ptr<f16> {tt.divisibility = 16 : i32},
1542|                   %ext : index) -> tensor<128x128xf32, #C> {
1543|   // A ptrs
1544|   %a_ptr_splat = tt.splat %A : !tt.ptr<f16> -> tensor<128x32x!tt.ptr<f16>, #AL>
1545|   %a_tmp0 = tt.make_range {end = 32: i32, start = 0: i32} : tensor<32xi32, #ALs0>
1546|   %a_tmp1 = tt.expand_dims %a_tmp0 {axis = 0 : i32} : tensor<32xi32, #ALs0> -> tensor<1x32xi32, #AL>
1547|   %a_offs = tt.broadcast %a_tmp1 : tensor<1x32xi32, #AL> -> tensor<128x32xi32, #AL>
1548|   %a_ptr_init = tt.addptr %a_ptr_splat, %a_offs : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xi32, #AL>
1549|   // B ptrs
1550|   %b_ptr_splat = tt.splat %B : !tt.ptr<f16> -> tensor<32x128x!tt.ptr<f16>, #BL>
1551|   %b_tmp0 = tt.make_range {end = 128: i32, start = 0: i32} : tensor<128xi32, #BLs0>
1552|   %b_tmp1 = tt.expand_dims %b_tmp0 {axis = 0 : i32} : tensor<128xi32, #BLs0> -> tensor<1x128xi32, #BL>
1553|   %b_offs = tt.broadcast %b_tmp1 : tensor<1x128xi32, #BL> -> tensor<32x128xi32, #BL>
1554|   %b_ptr = tt.addptr %b_ptr_splat, %b_offs : tensor<32x128x!tt.ptr<f16>, #BL>, tensor<32x128xi32, #BL>
1555| 
1556|   %a_mask = arith.constant dense<true> : tensor<128x32xi1, #AL>
```
**EN:** This function-oriented block defines or enters `matmul_nested_ops`. Within it, the test exercises tt.func, broadcasted scalars or pointers, lane/block index ranges, shape expansion, tensor broadcasting, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `matmul_nested_ops` 为核心。测试在其中演示 tt.func、广播后的标量或指针、lane/block 索引范围、形状扩展、张量广播，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1557-1574
```mlir
1557|   %a_other = arith.constant dense<0.00e+00> : tensor<128x32xf16, #AL>
1558|   %b_mask = arith.constant dense<true> : tensor<32x128xi1, #BL>
1559|   %b_other = arith.constant dense<0.00e+00> : tensor<32x128xf16, #BL>
1560|   %c_init = arith.constant dense<0.00e+00> : tensor<128x128xf32, #C>
1561| 
1562|   %a_off = arith.constant dense<4> : tensor<128x32xi32, #AL>
1563| 
1564|   %b_ = tt.load %b_ptr, %b_mask, %b_other : tensor<32x128x!tt.ptr<f16>, #BL>
1565|   %b = ttg.convert_layout %b_ : tensor<32x128xf16, #BL> -> tensor<32x128xf16, #B>
1566| 
1567|   %loop:2 = scf.for %iv = %lb to %ub step %step iter_args(%a_ptr = %a_ptr_init, %prev_c = %c_init) -> (tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x128xf32, #C>) {
1568|     %cnd = arith.cmpi slt, %iv, %ext : index
1569|     %inc_a_ptr = scf.if %cnd -> (tensor<128x32x!tt.ptr<f16>, #AL>) {
1570|       %a_ptr_ = tt.addptr %a_ptr, %a_off : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xi32, #AL>
1571|       scf.yield %a_ptr_ : tensor<128x32x!tt.ptr<f16>, #AL>
1572|     } else {
1573|       scf.yield %a_ptr : tensor<128x32x!tt.ptr<f16>, #AL>
1574|     }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining constants, loop/if yielded values, masked or vectorized loads, layout conversions, structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 常量、循环/分支产出值、带掩码或向量化的加载、布局转换、结构化循环。

### Lines 1575-1586
```mlir
1575|     %a_ = tt.load %inc_a_ptr : tensor<128x32x!tt.ptr<f16>, #AL>
1576|     %a = ttg.convert_layout %a_ : tensor<128x32xf16, #AL> -> tensor<128x32xf16, #A>
1577| 
1578|     %c = tt.dot %a, %b, %prev_c : tensor<128x32xf16, #A> * tensor<32x128xf16, #B> -> tensor<128x128xf32, #C>
1579| 
1580|     %next_a_ptr = tt.addptr %inc_a_ptr, %a_off : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xi32, #AL>
1581|     scf.yield %next_a_ptr, %c : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x128xf32, #C>
1582|   }
1583|   tt.return %loop#1: tensor<128x128xf32, #C>
1584| }
1585| }
1586| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining masked or vectorized loads, layout conversions, dot-product or MMA-style math, pointer arithmetic, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 带掩码或向量化的加载、布局转换、点积或 MMA 风格计算、指针算术、循环/分支产出值。

### Lines 1587-1587
```mlir
1587| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1588-1605
```mlir
1588| 
1589| // CHECK-LABEL: @masked_add_kernel
1590| // CHECK: %[[CONSTANT:.*]] = arith.constant dense<0xFF800000>
1591| // CHECK:   scf.for
1592| // CHECK: %[[A:.*]] = ttg.local_load
1593| // CHECK: arith.select {{.*}}, %[[A]], %[[CONSTANT]]
1594| // CHECK: %[[B:.*]] = ttg.local_load
1595| // CHECK: arith.select {{.*}}, %[[B]], %[[CONSTANT]]
1596| 
1597| // AMD-LABEL: @masked_add_kernel
1598| // AMD: %[[CONSTANT:.*]] = arith.constant dense<0xFF800000>
1599| // AMD: tt.load {{.*}}, %{{.*}}, %[[CONSTANT]]
1600| // AMD: tt.load {{.*}}, %{{.*}}, %[[CONSTANT]]
1601| // AMD: tt.load {{.*}}, %{{.*}}, %[[CONSTANT]]
1602| // AMD: tt.load {{.*}}, %{{.*}}, %[[CONSTANT]]
1603| // AMD: scf.for
1604| // AMD:   arith.select
1605| // AMD:   %[[A:.*]] = tt.load {{.*}}, %{{.*}}, %[[CONSTANT]]
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 1606-1623
```mlir
1606| // AMD:   %[[B:.*]] = tt.load {{.*}}, %{{.*}}, %[[CONSTANT]]
1607| // AMD:   arith.addf
1608| // AMD:   tt.store
1609| // AMD:   scf.yield
1610| // AMD: tt.store
1611| // AMD: tt.store
1612| 
1613| // AMD_3_STAGES-LABEL: @masked_add_kernel
1614| // AMD_3_STAGES: %[[CONSTANT:.*]] = arith.constant dense<0xFF800000>
1615| // AMD_3_STAGES-COUNT-4: tt.load {{.*}}, %{{.*}}, %[[CONSTANT]]
1616| // AMD_3_STAGES: scf.for
1617| // AMD_3_STAGES:   arith.select
1618| // AMD_3_STAGES:   %[[A:.*]] = tt.load {{.*}}, %{{.*}}, %[[CONSTANT]]
1619| // AMD_3_STAGES:   %[[B:.*]] = tt.load {{.*}}, %{{.*}}, %[[CONSTANT]]
1620| // AMD_3_STAGES:   arith.addf
1621| // AMD_3_STAGES:   tt.store
1622| // AMD_3_STAGES:   scf.yield
1623| // AMD_3_STAGES: tt.store
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining IR structure and attributes.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 IR 结构与属性。

### Lines 1624-1625
```mlir
1624| // AMD_3_STAGES: tt.store
1625| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1626-1626
```mlir
1626| #blocked = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1627-1627
```mlir
1627| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 1628-1645
```mlir
1628|   tt.func public @masked_add_kernel(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg2: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg3: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 16 : i32}) {
1629|     %c1024_i32 = arith.constant 1024 : i32
1630|     %c0_i32 = arith.constant 0 : i32
1631|     %c1016800_i32 = arith.constant 1016800 : i32
1632|     %cst = arith.constant dense<0xFF800000> : tensor<1024xf32, #blocked>
1633|     %0 = tt.get_program_id x : i32
1634|     %1 = arith.muli %0, %c1016800_i32 : i32
1635|     %2 = tt.make_range {end = 1024 : i32, start = 0 : i32} : tensor<1024xi32, #blocked>
1636|     %3 = tt.splat %arg3 : i32 -> tensor<1024xi32, #blocked>
1637|     %4 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<1024x!tt.ptr<f32>, #blocked>
1638|     %5 = tt.splat %arg1 : !tt.ptr<f32> -> tensor<1024x!tt.ptr<f32>, #blocked>
1639|     %6 = tt.splat %arg2 : !tt.ptr<f32> -> tensor<1024x!tt.ptr<f32>, #blocked>
1640|     scf.for %arg4 = %c0_i32 to %c1016800_i32 step %c1024_i32  : i32 {
1641|       %7 = arith.addi %1, %arg4 : i32
1642|       %8 = tt.splat %7 : i32 -> tensor<1024xi32, #blocked>
1643|       %9 = arith.addi %8, %2 : tensor<1024xi32, #blocked>
1644|       %10 = arith.cmpi slt, %9, %3 : tensor<1024xi32, #blocked>
1645|       %11 = tt.addptr %4, %9 : tensor<1024x!tt.ptr<f32>, #blocked>, tensor<1024xi32, #blocked>
```
**EN:** This function-oriented block defines or enters `masked_add_kernel`. Within it, the test exercises broadcasted scalars or pointers, constants, tt.func, integer additions, program IDs, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `masked_add_kernel` 为核心。测试在其中演示 广播后的标量或指针、常量、tt.func、整数加法、程序 ID，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1646-1656
```mlir
1646|       %12 = tt.load %11, %10, %cst : tensor<1024x!tt.ptr<f32>, #blocked>
1647|       %13 = tt.addptr %5, %9 : tensor<1024x!tt.ptr<f32>, #blocked>, tensor<1024xi32, #blocked>
1648|       %14 = tt.load %13, %10, %cst : tensor<1024x!tt.ptr<f32>, #blocked>
1649|       %15 = arith.addf %12, %14 : tensor<1024xf32, #blocked>
1650|       %16 = tt.addptr %6, %9 : tensor<1024x!tt.ptr<f32>, #blocked>, tensor<1024xi32, #blocked>
1651|       tt.store %16, %15, %10 : tensor<1024x!tt.ptr<f32>, #blocked>
1652|     }{tt.num_stages = 3 : i32}
1653|     tt.return
1654|   }
1655| }
1656| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on masked or vectorized loads, pointer arithmetic, floating-point additions, masked or vectorized stores, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带掩码或向量化的加载、指针算术、浮点加法、带掩码或向量化的存储、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1657-1657
```mlir
1657| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1658-1659
```mlir
1658| 
1659| #blocked = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1660-1660
```mlir
1660| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 1661-1663
```mlir
1661|   // CHECK-LABEL: @predicate_stage1
1662|   // CHECK: scf.for %[[IV:.*]] = %[[LB:.*]] to %[[UB:.*]] step %[[STEP:.*]] iter_args
1663|   // CHECK: ttg.predicate_stage %[[IV]], %[[UB]], %[[STEP]] maxStage 2 stage 0 : i32 -> i1
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @predicate_stage1 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @predicate_stage1 这样的标签用于锚定匹配范围。

### Lines 1664-1681
```mlir
1664|   tt.func public @predicate_stage1(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg2: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg3: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 16 : i32}) {
1665|     %c1024_i32 = arith.constant 1024 : i32
1666|     %c0_i32 = arith.constant 0 : i32
1667|     %c1016800_i32 = arith.constant 1016800 : i32
1668|     %0 = tt.get_program_id x : i32
1669|     %1 = arith.muli %0, %c1016800_i32 : i32
1670|     %2 = tt.make_range {end = 1024 : i32, start = 0 : i32} : tensor<1024xi32, #blocked>
1671|     %3 = tt.splat %arg3 : i32 -> tensor<1024xi32, #blocked>
1672|     %4 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<1024x!tt.ptr<f32>, #blocked>
1673|     %5 = tt.splat %arg1 : !tt.ptr<f32> -> tensor<1024x!tt.ptr<f32>, #blocked>
1674|     %6 = tt.splat %arg2 : !tt.ptr<f32> -> tensor<1024x!tt.ptr<f32>, #blocked>
1675|     scf.for %arg4 = %c0_i32 to %c1016800_i32 step %c1024_i32  : i32 {
1676|       %7 = arith.addi %1, %arg4 : i32
1677|       %8 = tt.splat %7 : i32 -> tensor<1024xi32, #blocked>
1678|       %9 = arith.addi %8, %2 : tensor<1024xi32, #blocked>
1679|       %10 = arith.cmpi slt, %9, %3 : tensor<1024xi32, #blocked>
1680|       %11 = tt.addptr %4, %9 : tensor<1024x!tt.ptr<f32>, #blocked>, tensor<1024xi32, #blocked>
1681|       %12 = tt.load %11, %10 : tensor<1024x!tt.ptr<f32>, #blocked>
```
**EN:** This function-oriented block defines or enters `predicate_stage1`. Within it, the test exercises broadcasted scalars or pointers, constants, tt.func, integer additions, program IDs, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `predicate_stage1` 为核心。测试在其中演示 广播后的标量或指针、常量、tt.func、整数加法、程序 ID，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1682-1691
```mlir
1682|       %13 = tt.addptr %5, %9 : tensor<1024x!tt.ptr<f32>, #blocked>, tensor<1024xi32, #blocked>
1683|       %14 = tt.load %13, %10 : tensor<1024x!tt.ptr<f32>, #blocked>
1684|       %15 = arith.addf %12, %14 : tensor<1024xf32, #blocked>
1685|       %16 = tt.addptr %6, %9 : tensor<1024x!tt.ptr<f32>, #blocked>, tensor<1024xi32, #blocked>
1686|       tt.store %16, %15, %10 : tensor<1024x!tt.ptr<f32>, #blocked>
1687|     } {tt.num_stages = 3 : i32, __test_keep_predicate_stage}
1688|     tt.return
1689|   }
1690| }
1691| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on pointer arithmetic, masked or vectorized loads, floating-point additions, masked or vectorized stores, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 指针算术、带掩码或向量化的加载、浮点加法、带掩码或向量化的存储、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1692-1692
```mlir
1692| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1693-1699
```mlir
1693| 
1694| #AL = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
1695| #C = #ttg.nvidia_mma<{versionMajor = 2, warpsPerCTA = [4, 1], instrShape = [16, 8]}>
1696| #A = #ttg.dot_op<{opIdx = 0, parent = #C, kWidth=2}>
1697| #B = #ttg.dot_op<{opIdx = 1, parent = #C, kWidth=2}>
1698| #smem = #ttg.shared_memory
1699| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1700-1701
```mlir
1700| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
1701| // Verify that statically dead prologue iterations are properly predicated
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 1702-1708
```mlir
1702| // CHECK-LABEL: @peeled_prologue_statically_dead
1703| // CHECK-DAG: %[[FALSE:.*]] = arith.constant dense<false>
1704| // CHECK-DAG: %[[TRUE:.*]] = arith.constant dense<true>
1705| // CHECK: ttg.async_copy_global_to_local {{.*}} mask %[[TRUE]]
1706| // CHECK: ttg.async_copy_global_to_local {{.*}} mask %[[TRUE]]
1707| // CHECK: ttg.async_copy_global_to_local {{.*}} mask %[[FALSE]]
1708| // CHECK: scf.for
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @peeled_prologue_statically_dead anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @peeled_prologue_statically_dead 这样的标签用于锚定匹配范围。

### Lines 1709-1726
```mlir
1709| tt.func @peeled_prologue_statically_dead(
1710|                   %a_ptr : tensor<128x32x!tt.ptr<f16>, #AL> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[1, 16]> : tensor<2xi32>},
1711|                   %B : tensor<32x128xf16, #B>) -> tensor<128x128xf32, #C> {
1712|   %lb = arith.constant 0 : i32
1713|   %ub = arith.constant 2 : i32
1714|   %step = arith.constant 1 : i32
1715| 
1716|   %c_init = arith.constant dense<0.00e+00> : tensor<128x128xf32, #C>
1717| 
1718|   %loop = scf.for %iv = %lb to %ub step %step iter_args(%prev_c = %c_init) -> (tensor<128x128xf32, #C>) : i32 {
1719|     %a_ = tt.load %a_ptr : tensor<128x32x!tt.ptr<f16>, #AL>
1720|     %a = ttg.convert_layout %a_ : tensor<128x32xf16, #AL> -> tensor<128x32xf16, #A>
1721|     %c = tt.dot %a, %B, %prev_c : tensor<128x32xf16, #A> * tensor<32x128xf16, #B> -> tensor<128x128xf32, #C>
1722|     scf.yield %c : tensor<128x128xf32, #C>
1723|   } {tt.num_stages = 4 : i32}
1724|   tt.return %loop: tensor<128x128xf32, #C>
1725| }
1726| 
```
**EN:** This function-oriented block defines or enters `peeled_prologue_statically_dead`. Within it, the test exercises constants, tt.func, structured loops, masked or vectorized loads, layout conversions, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `peeled_prologue_statically_dead` 为核心。测试在其中演示 常量、tt.func、结构化循环、带掩码或向量化的加载、布局转换，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1727-1728
```mlir
1727| }
1728| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 1729-1729
```mlir
1729| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1730-1732
```mlir
1730| 
1731| // Disable pipelining for loops that contain barriers.
1732| //   Barriers are problematic since they are not chained to any other operation.
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1733-1740
```mlir
1733| // COMMON-LABEL: tt.func public @barrier_in_loop_kernel
1734| // COMMON:  scf.for
1735| // COMMON:    tt.load
1736| // COMMON:    ttg.barrier local
1737| // COMMON:    tt.store
1738| // COMMON-NOT:  ttg.barrier local
1739| // COMMON:  tt.return
1740| 
```
**EN:** This function-oriented block defines or enters `barrier_in_loop_kernel`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `barrier_in_loop_kernel` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1741-1741
```mlir
1741| #blocked = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1742-1742
```mlir
1742| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 1743-1754
```mlir
1743|   tt.func public @barrier_in_loop_kernel(%arg1: tensor<1024x!tt.ptr<f32>, #blocked> {tt.divisibility = 16 : i32, tt.max_divisibility = 16 : i32},  %arg2: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 16 : i32}) {
1744|     %c1024_i32 = arith.constant 1024 : i32
1745|     %c0_i32 = arith.constant 0 : i32
1746|     scf.for %arg4 = %c0_i32 to %arg2 step %c1024_i32  : i32 {
1747|       %12 = tt.load %arg1 : tensor<1024x!tt.ptr<f32>, #blocked>
1748|       ttg.barrier local
1749|       tt.store %arg1, %12 : tensor<1024x!tt.ptr<f32>, #blocked>
1750|     } {tt.num_stages = 2 : i32}
1751|     tt.return
1752|   }
1753| }
1754| 
```
**EN:** This function-oriented block defines or enters `barrier_in_loop_kernel`. Within it, the test exercises tt.func, constants, structured loops, masked or vectorized loads, ttg.barrier, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `barrier_in_loop_kernel` 为核心。测试在其中演示 tt.func、常量、结构化循环、带掩码或向量化的加载、ttg.barrier，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1755-1755
```mlir
1755| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1756-1757
```mlir
1756| 
1757| // Disable pipelining for loops that contain asserts because we should not reorder them
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1758-1764
```mlir
1758| // COMMON-LABEL: tt.func public @assert_in_loop_kernel
1759| // COMMON:  scf.for
1760| // COMMON:    tt.load
1761| // COMMON:    tt.assert
1762| // COMMON:    tt.store
1763| // COMMON-NOT:  tt.assert
1764| // COMMON:  tt.return
```
**EN:** This function-oriented block defines or enters `assert_in_loop_kernel`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `assert_in_loop_kernel` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1765-1765
```mlir
1765| #blocked = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1766-1766
```mlir
1766| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 1767-1778
```mlir
1767|   tt.func public @assert_in_loop_kernel(%arg1: tensor<1024x!tt.ptr<f32>, #blocked> {tt.divisibility = 16 : i32, tt.max_divisibility = 16 : i32},  %arg2: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 16 : i32}, %arg3: i1) {
1768|     %c1024_i32 = arith.constant 1024 : i32
1769|     %c0_i32 = arith.constant 0 : i32
1770|     scf.for %arg4 = %c0_i32 to %arg2 step %c1024_i32  : i32 {
1771|       %12 = tt.load %arg1 : tensor<1024x!tt.ptr<f32>, #blocked>
1772|       tt.assert %arg3, "some assert" : i1
1773|       tt.store %arg1, %12 : tensor<1024x!tt.ptr<f32>, #blocked>
1774|     } {tt.num_stages = 2 : i32}
1775|     tt.return
1776|   }
1777| }
1778| 
```
**EN:** This function-oriented block defines or enters `assert_in_loop_kernel`. Within it, the test exercises tt.func, constants, structured loops, masked or vectorized loads, tt.assert, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `assert_in_loop_kernel` 为核心。测试在其中演示 tt.func、常量、结构化循环、带掩码或向量化的加载、tt.assert，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1779-1779
```mlir
1779| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1780-1781
```mlir
1780| 
1781| // Disable pipelining for loops that contain prints because we should not reorder them
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1782-1788
```mlir
1782| // COMMON-LABEL: tt.func public @print_in_loop_kernel
1783| // COMMON:  scf.for
1784| // COMMON:    tt.load
1785| // COMMON:    tt.print
1786| // COMMON:    tt.store
1787| // COMMON-NOT:  tt.print
1788| // COMMON:  tt.return
```
**EN:** This function-oriented block defines or enters `print_in_loop_kernel`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `print_in_loop_kernel` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1789-1789
```mlir
1789| #blocked = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1790-1790
```mlir
1790| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 1791-1801
```mlir
1791|   tt.func public @print_in_loop_kernel(%arg1: tensor<1024x!tt.ptr<f32>, #blocked> {tt.divisibility = 16 : i32, tt.max_divisibility = 16 : i32},  %arg2: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 16 : i32}, %arg3: i32) {
1792|     %c1024_i32 = arith.constant 1024 : i32
1793|     %c0_i32 = arith.constant 0 : i32
1794|     scf.for %arg4 = %c0_i32 to %arg2 step %c1024_i32  : i32 {
1795|       %12 = tt.load %arg1 : tensor<1024x!tt.ptr<f32>, #blocked>
1796|       tt.print "some print" {hex = false, isSigned = array<i32: 0>} : %arg3 : i32
1797|       tt.store %arg1, %12 : tensor<1024x!tt.ptr<f32>, #blocked>
1798|     } {tt.num_stages = 2 : i32}
1799|     tt.return
1800|   }
1801| }
```
**EN:** This function-oriented block defines or enters `print_in_loop_kernel`. Within it, the test exercises tt.func, constants, structured loops, masked or vectorized loads, tt.print, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `print_in_loop_kernel` 为核心。测试在其中演示 tt.func、常量、结构化循环、带掩码或向量化的加载、tt.print，这些操作构成了被优化、lower 或验证的主要数据流。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-tritongpu-assign-latencies`, `-tritongpu-schedule-loops`, `-tritongpu-pipeline=num-stages=3`, `-canonicalize`, `-tritonamdgpu-schedule-loops=num_stages=2`, `-tritonamdgpu-pipeline`, `-tritonamdgpu-schedule-loops="num_stages=3"`
- **CN:** 主要 pass 选项：`-split-input-file`，`-tritongpu-assign-latencies`，`-tritongpu-schedule-loops`，`-tritongpu-pipeline=num-stages=3`，`-canonicalize`，`-tritonamdgpu-schedule-loops=num_stages=2`，`-tritonamdgpu-pipeline`，`-tritonamdgpu-schedule-loops="num_stages=3"`
- **EN:** Dominant operations include `arith.constant`, `tt.splat`, `tt.addptr`, `tt.load`, `tt.func`, `ttg.convert_layout`, `tt.broadcast`, `tt.expand_dims`, `scf.for`, `tt.make_range`.
- **CN:** 主要操作包括 `arith.constant`、`tt.splat`、`tt.addptr`、`tt.load`、`tt.func`、`ttg.convert_layout`、`tt.broadcast`、`tt.expand_dims`、`scf.for`、`tt.make_range`。
- **EN:** The file contains 17 independently testable section(s). Check styles used: CHECK x187, CHECK-DAG x44, CHECK-LABEL x15, CHECK-NOT x3. Important labels include tt.func @matmul_loop, tt.func @matmul_loop_nested, tt.func @matmul_loop_single_pipeline, tt.func @indirect_bmm_scalar. It validates both the presence of the desired rewrite and the absence of unwanted residual operations.
- **CN:** 该文件包含 17 个可独立测试的分段。使用的检查类型：CHECK ×187，CHECK-DAG ×44，CHECK-LABEL ×15，CHECK-NOT ×3。 关键标签包括 tt.func @matmul_loop，tt.func @matmul_loop_nested，tt.func @matmul_loop_single_pipeline，tt.func @indirect_bmm_scalar。 它同时验证目标重写已经出现，并且不再残留不需要的操作。

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
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。