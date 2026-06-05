# arm_neon.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/include/clang/Basic/arm_neon.td`
- **Repository / 仓库**: `/root/xw/llvm-project`
- **Purpose (EN)**: ARM NEON compiler interface.
- **Purpose (CN)**: 声明与 `arm_neon` 相关的编译器基础元数据、诊断信息、目标描述或 builtin 定义。
- **Line Count / 行数**: 2238

## Line-by-Line Analysis / 逐行分析

### Lines 1-28

````tablegen
//===--- arm_neon.td - ARM NEON compiler interface ------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
//  This file defines the TableGen definitions from which the ARM NEON header
//  file will be generated.  See ARM document DUI0348B.
//
//===----------------------------------------------------------------------===//

include "arm_neon_incl.td"

def OP_ADD      : Op<(op "+", $p0, $p1)>;
def OP_ADDL     : Op<(op "+", (call "vmovl", $p0), (call "vmovl", $p1))>;
def OP_ADDLHi   : Op<(op "+", (call "vmovl_high", $p0),
                              (call "vmovl_high", $p1))>;
def OP_ADDW     : Op<(op "+", $p0, (call "vmovl", $p1))>;
def OP_ADDWHi   : Op<(op "+", $p0, (call "vmovl_high", $p1))>;
def OP_SUB      : Op<(op "-", $p0, $p1)>;
def OP_SUBL     : Op<(op "-", (call "vmovl", $p0), (call "vmovl", $p1))>;
def OP_SUBLHi   : Op<(op "-", (call "vmovl_high", $p0),
                              (call "vmovl_high", $p1))>;
def OP_SUBW     : Op<(op "-", $p0, (call "vmovl", $p1))>;
def OP_SUBWHi   : Op<(op "-", $p0, (call "vmovl_high", $p1))>;
def OP_MUL      : Op<(op "*", $p0, $p1)>;
````
- **L1 EN**: Banner comment marking a file or section boundary.
  **L1 CN**: 横幅注释，用于标记文件或章节边界。
- **L2 EN**: Separator comment used for visual grouping.
  **L2 CN**: 用于视觉分组的分隔注释。
- **L3 EN**: Comment explains nearby logic, constraints, or intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L3 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4 EN**: Comment explains nearby logic, constraints, or intent: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释解释附近代码的逻辑、约束或设计意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment explains nearby logic, constraints, or intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Separator comment used for visual grouping.
  **L6 CN**: 用于视觉分组的分隔注释。
- **L7 EN**: Banner comment marking a file or section boundary.
  **L7 CN**: 横幅注释，用于标记文件或章节边界。
- **L8 EN**: Separator comment used for visual grouping.
  **L8 CN**: 用于视觉分组的分隔注释。
- **L9 EN**: Comment explains nearby logic, constraints, or intent: `This file defines the TableGen definitions from which the ARM NEON header`.
  **L9 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This file defines the TableGen definitions from which the ARM NEON header`。
- **L10 EN**: Comment explains nearby logic, constraints, or intent: `file will be generated. See ARM document DUI0348B.`.
  **L10 CN**: 注释解释附近代码的逻辑、约束或设计意图：`file will be generated. See ARM document DUI0348B.`。
- **L11 EN**: Separator comment used for visual grouping.
  **L11 CN**: 用于视觉分组的分隔注释。
- **L12 EN**: Banner comment marking a file or section boundary.
  **L12 CN**: 横幅注释，用于标记文件或章节边界。
- **L13 EN**: Blank line separating nearby declarations or logic blocks.
  **L13 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L14 EN**: Includes TableGen file `"arm_neon_incl.td"` so later records can reuse shared definitions.
  **L14 CN**: 引入 TableGen 文件 `"arm_neon_incl.td"`，以便后续记录复用共享定义。
- **L15 EN**: Blank line separating nearby declarations or logic blocks.
  **L15 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L16 EN**: Declares TableGen def record `OP_ADD`.
  **L16 CN**: 声明 TableGen def 记录 `OP_ADD`。
- **L17 EN**: Declares TableGen def record `OP_ADDL`.
  **L17 CN**: 声明 TableGen def 记录 `OP_ADDL`。
- **L18 EN**: Declares TableGen def record `OP_ADDLHi`.
  **L18 CN**: 声明 TableGen def 记录 `OP_ADDLHi`。
- **L19 EN**: Executes a call or declaration centered on `statement`.
  **L19 CN**: 执行以 `statement` 为核心的调用或声明。
- **L20 EN**: Declares TableGen def record `OP_ADDW`.
  **L20 CN**: 声明 TableGen def 记录 `OP_ADDW`。
- **L21 EN**: Declares TableGen def record `OP_ADDWHi`.
  **L21 CN**: 声明 TableGen def 记录 `OP_ADDWHi`。
- **L22 EN**: Declares TableGen def record `OP_SUB`.
  **L22 CN**: 声明 TableGen def 记录 `OP_SUB`。
- **L23 EN**: Declares TableGen def record `OP_SUBL`.
  **L23 CN**: 声明 TableGen def 记录 `OP_SUBL`。
- **L24 EN**: Declares TableGen def record `OP_SUBLHi`.
  **L24 CN**: 声明 TableGen def 记录 `OP_SUBLHi`。
- **L25 EN**: Executes a call or declaration centered on `statement`.
  **L25 CN**: 执行以 `statement` 为核心的调用或声明。
- **L26 EN**: Declares TableGen def record `OP_SUBW`.
  **L26 CN**: 声明 TableGen def 记录 `OP_SUBW`。
- **L27 EN**: Declares TableGen def record `OP_SUBWHi`.
  **L27 CN**: 声明 TableGen def 记录 `OP_SUBWHi`。
- **L28 EN**: Declares TableGen def record `OP_MUL`.
  **L28 CN**: 声明 TableGen def 记录 `OP_MUL`。

### Lines 29-56

````tablegen
def OP_MLA      : Op<(op "+", $p0, (op "*", $p1, $p2))>;
def OP_MLAL     : Op<(op "+", $p0, (call "vmull", $p1, $p2))>;
def OP_MULLHi   : Op<(call "vmull", (call "vget_high", $p0),
                                    (call "vget_high", $p1))>;
def OP_MULLHi_P64 : Op<(call "vmull",
                         (bitcast "poly64_t", (call "vget_high", $p0)),
                         (bitcast "poly64_t", (call "vget_high", $p1)))>;
def OP_MULLHi_N : Op<(call "vmull_n", (call "vget_high", $p0), $p1)>;
def OP_MLALHi   : Op<(call "vmlal", $p0, (call "vget_high", $p1),
                                         (call "vget_high", $p2))>;
def OP_MLALHi_N : Op<(call "vmlal_n", $p0, (call "vget_high", $p1), $p2)>;
def OP_MLS      : Op<(op "-", $p0, (op "*", $p1, $p2))>;
def OP_FMLS     : Op<(call "vfma", $p0, (op "-", $p1), $p2)>;
def OP_MLSL     : Op<(op "-", $p0, (call "vmull", $p1, $p2))>;
def OP_MLSLHi   : Op<(call "vmlsl", $p0, (call "vget_high", $p1),
                                         (call "vget_high", $p2))>;
def OP_MLSLHi_N : Op<(call "vmlsl_n", $p0, (call "vget_high", $p1), $p2)>;
def OP_MUL_N    : Op<(op "*", $p0, (dup $p1))>;
def OP_MULX_N   : Op<(call "vmulx", $p0, (dup $p1))>;
def OP_MLA_N    : Op<(op "+", $p0, (op "*", $p1, (dup $p2)))>;
def OP_MLS_N    : Op<(op "-", $p0, (op "*", $p1, (dup $p2)))>;
def OP_FMLA_N   : Op<(call "vfma", $p0, $p1, (dup $p2))>;
def OP_FMLS_N   : Op<(call "vfma", $p0, (op "-", $p1), (dup $p2))>;
def OP_MLAL_N   : Op<(op "+", $p0, (call "vmull", $p1, (dup $p2)))>;
def OP_MLSL_N   : Op<(op "-", $p0, (call "vmull", $p1, (dup $p2)))>;
def OP_MUL_LN   : Op<(op "*", $p0, (call_mangled "splat_lane", $p1, $p2))>;
def OP_MULX_LN  : Op<(call "vmulx", $p0, (call_mangled "splat_lane", $p1, $p2))>;
def OP_MULL_N  : Op<(call "vmull", $p0, (dup $p1))>;
````
- **L29 EN**: Declares TableGen def record `OP_MLA`.
  **L29 CN**: 声明 TableGen def 记录 `OP_MLA`。
- **L30 EN**: Declares TableGen def record `OP_MLAL`.
  **L30 CN**: 声明 TableGen def 记录 `OP_MLAL`。
- **L31 EN**: Declares TableGen def record `OP_MULLHi`.
  **L31 CN**: 声明 TableGen def 记录 `OP_MULLHi`。
- **L32 EN**: Executes a call or declaration centered on `statement`.
  **L32 CN**: 执行以 `statement` 为核心的调用或声明。
- **L33 EN**: Declares TableGen def record `OP_MULLHi_P64`.
  **L33 CN**: 声明 TableGen def 记录 `OP_MULLHi_P64`。
- **L34 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(bitcast "poly64_t", (call "vget_high", $p0)),`.
  **L34 CN**: 继续一个多行参数列表、初始化器或聚合项：`(bitcast "poly64_t", (call "vget_high", $p0)),`。
- **L35 EN**: Executes a call or declaration centered on `statement`.
  **L35 CN**: 执行以 `statement` 为核心的调用或声明。
- **L36 EN**: Declares TableGen def record `OP_MULLHi_N`.
  **L36 CN**: 声明 TableGen def 记录 `OP_MULLHi_N`。
- **L37 EN**: Declares TableGen def record `OP_MLALHi`.
  **L37 CN**: 声明 TableGen def 记录 `OP_MLALHi`。
- **L38 EN**: Executes a call or declaration centered on `statement`.
  **L38 CN**: 执行以 `statement` 为核心的调用或声明。
- **L39 EN**: Declares TableGen def record `OP_MLALHi_N`.
  **L39 CN**: 声明 TableGen def 记录 `OP_MLALHi_N`。
- **L40 EN**: Declares TableGen def record `OP_MLS`.
  **L40 CN**: 声明 TableGen def 记录 `OP_MLS`。
- **L41 EN**: Declares TableGen def record `OP_FMLS`.
  **L41 CN**: 声明 TableGen def 记录 `OP_FMLS`。
- **L42 EN**: Declares TableGen def record `OP_MLSL`.
  **L42 CN**: 声明 TableGen def 记录 `OP_MLSL`。
- **L43 EN**: Declares TableGen def record `OP_MLSLHi`.
  **L43 CN**: 声明 TableGen def 记录 `OP_MLSLHi`。
- **L44 EN**: Executes a call or declaration centered on `statement`.
  **L44 CN**: 执行以 `statement` 为核心的调用或声明。
- **L45 EN**: Declares TableGen def record `OP_MLSLHi_N`.
  **L45 CN**: 声明 TableGen def 记录 `OP_MLSLHi_N`。
- **L46 EN**: Declares TableGen def record `OP_MUL_N`.
  **L46 CN**: 声明 TableGen def 记录 `OP_MUL_N`。
- **L47 EN**: Declares TableGen def record `OP_MULX_N`.
  **L47 CN**: 声明 TableGen def 记录 `OP_MULX_N`。
- **L48 EN**: Declares TableGen def record `OP_MLA_N`.
  **L48 CN**: 声明 TableGen def 记录 `OP_MLA_N`。
- **L49 EN**: Declares TableGen def record `OP_MLS_N`.
  **L49 CN**: 声明 TableGen def 记录 `OP_MLS_N`。
- **L50 EN**: Declares TableGen def record `OP_FMLA_N`.
  **L50 CN**: 声明 TableGen def 记录 `OP_FMLA_N`。
- **L51 EN**: Declares TableGen def record `OP_FMLS_N`.
  **L51 CN**: 声明 TableGen def 记录 `OP_FMLS_N`。
- **L52 EN**: Declares TableGen def record `OP_MLAL_N`.
  **L52 CN**: 声明 TableGen def 记录 `OP_MLAL_N`。
- **L53 EN**: Declares TableGen def record `OP_MLSL_N`.
  **L53 CN**: 声明 TableGen def 记录 `OP_MLSL_N`。
- **L54 EN**: Declares TableGen def record `OP_MUL_LN`.
  **L54 CN**: 声明 TableGen def 记录 `OP_MUL_LN`。
- **L55 EN**: Declares TableGen def record `OP_MULX_LN`.
  **L55 CN**: 声明 TableGen def 记录 `OP_MULX_LN`。
- **L56 EN**: Declares TableGen def record `OP_MULL_N`.
  **L56 CN**: 声明 TableGen def 记录 `OP_MULL_N`。

### Lines 57-84

````tablegen
def OP_MULL_LN  : Op<(call "vmull", $p0, (call_mangled "splat_lane", $p1, $p2))>;
def OP_MULLHi_LN: Op<(call "vmull", (call "vget_high", $p0), (call_mangled "splat_lane", $p1, $p2))>;
def OP_MLA_LN   : Op<(op "+", $p0, (op "*", $p1, (call_mangled "splat_lane", $p2, $p3)))>;
def OP_MLS_LN   : Op<(op "-", $p0, (op "*", $p1, (call_mangled "splat_lane", $p2, $p3)))>;
def OP_MLAL_LN  : Op<(op "+", $p0, (call "vmull", $p1, (call_mangled "splat_lane", $p2, $p3)))>;
def OP_MLALHi_LN: Op<(op "+", $p0, (call "vmull", (call "vget_high", $p1),
                                                  (call_mangled "splat_lane", $p2, $p3)))>;
def OP_MLSL_LN  : Op<(op "-", $p0, (call "vmull", $p1, (call_mangled "splat_lane", $p2, $p3)))>;
def OP_MLSLHi_LN : Op<(op "-", $p0, (call "vmull", (call "vget_high", $p1),
                                                   (call_mangled "splat_lane", $p2, $p3)))>;
def OP_QDMULL_N : Op<(call "vqdmull", $p0, (dup $p1))>;
def OP_QDMULL_LN : Op<(call "vqdmull", $p0, (call_mangled "splat_lane", $p1, $p2))>;
def OP_QDMULLHi_LN : Op<(call "vqdmull", (call "vget_high", $p0),
                                         (call_mangled "splat_lane", $p1, $p2))>;
def OP_QDMLAL_N : Op<(call "vqdmlal", $p0, $p1, (dup $p2))>;
def OP_QDMLAL_LN : Op<(call "vqdmlal", $p0, $p1, (call_mangled "splat_lane", $p2, $p3))>;
def OP_QDMLALHi_LN : Op<(call "vqdmlal", $p0, (call "vget_high", $p1),
                                              (call_mangled "splat_lane", $p2, $p3))>;
def OP_QDMLSL_N : Op<(call "vqdmlsl", $p0, $p1, (dup $p2))>;
def OP_QDMLSL_LN : Op<(call "vqdmlsl", $p0, $p1, (call_mangled "splat_lane", $p2, $p3))>;
def OP_QDMLSLHi_LN : Op<(call "vqdmlsl", $p0, (call "vget_high", $p1),
                                              (call_mangled "splat_lane", $p2, $p3))>;
def OP_QDMULH_N : Op<(call "vqdmulh", $p0, (dup $p1))>;
def OP_QDMULH_LN : Op<(call "vqdmulh", $p0, (call_mangled "splat_lane", $p1, $p2))>;
def OP_QRDMULH_LN : Op<(call "vqrdmulh", $p0, (call_mangled "splat_lane", $p1, $p2))>;
def OP_QRDMULH_N : Op<(call "vqrdmulh", $p0, (dup $p1))>;
def OP_QRDMLAH_LN : Op<(call "vqrdmlah", $p0, $p1, (call_mangled "splat_lane", $p2, $p3))>;
def OP_QRDMLSH_LN : Op<(call "vqrdmlsh", $p0, $p1, (call_mangled "splat_lane", $p2, $p3))>;
````
- **L57 EN**: Declares TableGen def record `OP_MULL_LN`.
  **L57 CN**: 声明 TableGen def 记录 `OP_MULL_LN`。
- **L58 EN**: Declares TableGen def record `OP_MULLHi_LN`.
  **L58 CN**: 声明 TableGen def 记录 `OP_MULLHi_LN`。
- **L59 EN**: Declares TableGen def record `OP_MLA_LN`.
  **L59 CN**: 声明 TableGen def 记录 `OP_MLA_LN`。
- **L60 EN**: Declares TableGen def record `OP_MLS_LN`.
  **L60 CN**: 声明 TableGen def 记录 `OP_MLS_LN`。
- **L61 EN**: Declares TableGen def record `OP_MLAL_LN`.
  **L61 CN**: 声明 TableGen def 记录 `OP_MLAL_LN`。
- **L62 EN**: Declares TableGen def record `OP_MLALHi_LN`.
  **L62 CN**: 声明 TableGen def 记录 `OP_MLALHi_LN`。
- **L63 EN**: Executes a call or declaration centered on `statement`.
  **L63 CN**: 执行以 `statement` 为核心的调用或声明。
- **L64 EN**: Declares TableGen def record `OP_MLSL_LN`.
  **L64 CN**: 声明 TableGen def 记录 `OP_MLSL_LN`。
- **L65 EN**: Declares TableGen def record `OP_MLSLHi_LN`.
  **L65 CN**: 声明 TableGen def 记录 `OP_MLSLHi_LN`。
- **L66 EN**: Executes a call or declaration centered on `statement`.
  **L66 CN**: 执行以 `statement` 为核心的调用或声明。
- **L67 EN**: Declares TableGen def record `OP_QDMULL_N`.
  **L67 CN**: 声明 TableGen def 记录 `OP_QDMULL_N`。
- **L68 EN**: Declares TableGen def record `OP_QDMULL_LN`.
  **L68 CN**: 声明 TableGen def 记录 `OP_QDMULL_LN`。
- **L69 EN**: Declares TableGen def record `OP_QDMULLHi_LN`.
  **L69 CN**: 声明 TableGen def 记录 `OP_QDMULLHi_LN`。
- **L70 EN**: Executes a call or declaration centered on `statement`.
  **L70 CN**: 执行以 `statement` 为核心的调用或声明。
- **L71 EN**: Declares TableGen def record `OP_QDMLAL_N`.
  **L71 CN**: 声明 TableGen def 记录 `OP_QDMLAL_N`。
- **L72 EN**: Declares TableGen def record `OP_QDMLAL_LN`.
  **L72 CN**: 声明 TableGen def 记录 `OP_QDMLAL_LN`。
- **L73 EN**: Declares TableGen def record `OP_QDMLALHi_LN`.
  **L73 CN**: 声明 TableGen def 记录 `OP_QDMLALHi_LN`。
- **L74 EN**: Executes a call or declaration centered on `statement`.
  **L74 CN**: 执行以 `statement` 为核心的调用或声明。
- **L75 EN**: Declares TableGen def record `OP_QDMLSL_N`.
  **L75 CN**: 声明 TableGen def 记录 `OP_QDMLSL_N`。
- **L76 EN**: Declares TableGen def record `OP_QDMLSL_LN`.
  **L76 CN**: 声明 TableGen def 记录 `OP_QDMLSL_LN`。
- **L77 EN**: Declares TableGen def record `OP_QDMLSLHi_LN`.
  **L77 CN**: 声明 TableGen def 记录 `OP_QDMLSLHi_LN`。
- **L78 EN**: Executes a call or declaration centered on `statement`.
  **L78 CN**: 执行以 `statement` 为核心的调用或声明。
- **L79 EN**: Declares TableGen def record `OP_QDMULH_N`.
  **L79 CN**: 声明 TableGen def 记录 `OP_QDMULH_N`。
- **L80 EN**: Declares TableGen def record `OP_QDMULH_LN`.
  **L80 CN**: 声明 TableGen def 记录 `OP_QDMULH_LN`。
- **L81 EN**: Declares TableGen def record `OP_QRDMULH_LN`.
  **L81 CN**: 声明 TableGen def 记录 `OP_QRDMULH_LN`。
- **L82 EN**: Declares TableGen def record `OP_QRDMULH_N`.
  **L82 CN**: 声明 TableGen def 记录 `OP_QRDMULH_N`。
- **L83 EN**: Declares TableGen def record `OP_QRDMLAH_LN`.
  **L83 CN**: 声明 TableGen def 记录 `OP_QRDMLAH_LN`。
- **L84 EN**: Declares TableGen def record `OP_QRDMLSH_LN`.
  **L84 CN**: 声明 TableGen def 记录 `OP_QRDMLSH_LN`。

### Lines 85-112

````tablegen
def OP_FMS_LN   : Op<(call "vfma_lane", $p0, (op "-", $p1), $p2, $p3)>;
def OP_FMS_LNQ  : Op<(call "vfma_laneq", $p0, (op "-", $p1), $p2, $p3)>;
def OP_TRN1     : Op<(shuffle $p0, $p1, (interleave (decimate mask0, 2),
                                                    (decimate mask1, 2)))>;
def OP_ZIP1     : Op<(shuffle $p0, $p1, (lowhalf (interleave mask0, mask1)))>;
def OP_UZP1     : Op<(shuffle $p0, $p1, (add (decimate mask0, 2),
                                             (decimate mask1, 2)))>;
def OP_TRN2     : Op<(shuffle $p0, $p1, (interleave
                                          (decimate (rotl mask0, 1), 2),
                                          (decimate (rotl mask1, 1), 2)))>;
def OP_ZIP2     : Op<(shuffle $p0, $p1, (highhalf (interleave mask0, mask1)))>;
def OP_UZP2     : Op<(shuffle $p0, $p1, (add (decimate (rotl mask0, 1), 2),
                                             (decimate (rotl mask1, 1), 2)))>;
def OP_EQ       : Op<(bitcast "R", (op "==", $p0, $p1))>;
def OP_GE       : Op<(bitcast "R", (op ">=", $p0, $p1))>;
def OP_LE       : Op<(bitcast "R", (op "<=", $p0, $p1))>;
def OP_GT       : Op<(bitcast "R", (op ">", $p0, $p1))>;
def OP_LT       : Op<(bitcast "R", (op "<", $p0, $p1))>;
def OP_NEG      : Op<(op "-", $p0)>;
def OP_NOT      : Op<(op "~", $p0)>;
def OP_AND      : Op<(op "&", $p0, $p1)>;
def OP_OR       : Op<(op "|", $p0, $p1)>;
def OP_XOR      : Op<(op "^", $p0, $p1)>;
def OP_ANDN     : Op<(op "&", $p0, (op "~", $p1))>;
def OP_ORN      : Op<(op "|", $p0, (op "~", $p1))>;
def OP_CAST     : LOp<[(save_temp $promote, $p0),
                       (bitcast "R", $promote)]>;
def OP_HI       : Op<(shuffle $p0, $p0, (highhalf mask0))>;
````
- **L85 EN**: Declares TableGen def record `OP_FMS_LN`.
  **L85 CN**: 声明 TableGen def 记录 `OP_FMS_LN`。
- **L86 EN**: Declares TableGen def record `OP_FMS_LNQ`.
  **L86 CN**: 声明 TableGen def 记录 `OP_FMS_LNQ`。
- **L87 EN**: Declares TableGen def record `OP_TRN1`.
  **L87 CN**: 声明 TableGen def 记录 `OP_TRN1`。
- **L88 EN**: Executes a call or declaration centered on `statement`.
  **L88 CN**: 执行以 `statement` 为核心的调用或声明。
- **L89 EN**: Declares TableGen def record `OP_ZIP1`.
  **L89 CN**: 声明 TableGen def 记录 `OP_ZIP1`。
- **L90 EN**: Declares TableGen def record `OP_UZP1`.
  **L90 CN**: 声明 TableGen def 记录 `OP_UZP1`。
- **L91 EN**: Executes a call or declaration centered on `statement`.
  **L91 CN**: 执行以 `statement` 为核心的调用或声明。
- **L92 EN**: Declares TableGen def record `OP_TRN2`.
  **L92 CN**: 声明 TableGen def 记录 `OP_TRN2`。
- **L93 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(decimate (rotl mask0, 1), 2),`.
  **L93 CN**: 继续一个多行参数列表、初始化器或聚合项：`(decimate (rotl mask0, 1), 2),`。
- **L94 EN**: Executes a call or declaration centered on `statement`.
  **L94 CN**: 执行以 `statement` 为核心的调用或声明。
- **L95 EN**: Declares TableGen def record `OP_ZIP2`.
  **L95 CN**: 声明 TableGen def 记录 `OP_ZIP2`。
- **L96 EN**: Declares TableGen def record `OP_UZP2`.
  **L96 CN**: 声明 TableGen def 记录 `OP_UZP2`。
- **L97 EN**: Executes a call or declaration centered on `statement`.
  **L97 CN**: 执行以 `statement` 为核心的调用或声明。
- **L98 EN**: Declares TableGen def record `OP_EQ`.
  **L98 CN**: 声明 TableGen def 记录 `OP_EQ`。
- **L99 EN**: Declares TableGen def record `OP_GE`.
  **L99 CN**: 声明 TableGen def 记录 `OP_GE`。
- **L100 EN**: Declares TableGen def record `OP_LE`.
  **L100 CN**: 声明 TableGen def 记录 `OP_LE`。
- **L101 EN**: Declares TableGen def record `OP_GT`.
  **L101 CN**: 声明 TableGen def 记录 `OP_GT`。
- **L102 EN**: Declares TableGen def record `OP_LT`.
  **L102 CN**: 声明 TableGen def 记录 `OP_LT`。
- **L103 EN**: Declares TableGen def record `OP_NEG`.
  **L103 CN**: 声明 TableGen def 记录 `OP_NEG`。
- **L104 EN**: Declares TableGen def record `OP_NOT`.
  **L104 CN**: 声明 TableGen def 记录 `OP_NOT`。
- **L105 EN**: Declares TableGen def record `OP_AND`.
  **L105 CN**: 声明 TableGen def 记录 `OP_AND`。
- **L106 EN**: Declares TableGen def record `OP_OR`.
  **L106 CN**: 声明 TableGen def 记录 `OP_OR`。
- **L107 EN**: Declares TableGen def record `OP_XOR`.
  **L107 CN**: 声明 TableGen def 记录 `OP_XOR`。
- **L108 EN**: Declares TableGen def record `OP_ANDN`.
  **L108 CN**: 声明 TableGen def 记录 `OP_ANDN`。
- **L109 EN**: Declares TableGen def record `OP_ORN`.
  **L109 CN**: 声明 TableGen def 记录 `OP_ORN`。
- **L110 EN**: Declares TableGen def record `OP_CAST`.
  **L110 CN**: 声明 TableGen def 记录 `OP_CAST`。
- **L111 EN**: Executes a call or declaration centered on `statement`.
  **L111 CN**: 执行以 `statement` 为核心的调用或声明。
- **L112 EN**: Declares TableGen def record `OP_HI`.
  **L112 CN**: 声明 TableGen def 记录 `OP_HI`。

### Lines 113-140

````tablegen
def OP_LO       : Op<(shuffle $p0, $p0, (lowhalf mask0))>;
def OP_CONC     : Op<(shuffle $p0, $p1, (add mask0, mask1))>;
def OP_DUP      : Op<(dup $p0)>;
def OP_DUP_LN   : Op<(call_mangled "splat_lane", $p0, $p1)>;
def OP_SEL      : Op<(bitcast "R", (op "|",
                                    (op "&", $p0, (bitcast $p0, $p1)),
                                    (op "&", (op "~", $p0), (bitcast $p0, $p2))))>;
def OP_REV16    : Op<(shuffle $p0, $p0, (rev 16, mask0))>;
def OP_REV32    : Op<(shuffle $p0, $p0, (rev 32, mask0))>;
def OP_REV64    : Op<(shuffle $p0, $p0, (rev 64, mask0))>;
def OP_XTN      : Op<(call "vcombine", $p0, (call "vmovn", $p1))>;
def OP_SQXTUN   : Op<(call "vcombine", (bitcast $p0, "U", $p0),
                                       (call "vqmovun", $p1))>;
def OP_QXTN     : Op<(call "vcombine", $p0, (call "vqmovn", $p1))>;
def OP_VCVT_NA_HI_F16 : Op<(call "vcombine", $p0, (call "vcvt_f16_f32", $p1))>;
def OP_VCVT_NA_HI_F32 : Op<(call "vcombine", $p0, (call "vcvt_f32_f64", $p1))>;
def OP_VCVT_EX_HI_F32 : Op<(call "vcvt_f32_f16", (call "vget_high", $p0))>;
def OP_VCVT_EX_HI_F64 : Op<(call "vcvt_f64_f32", (call "vget_high", $p0))>;
def OP_VCVTX_HI : Op<(call "vcombine", $p0, (call "vcvtx_f32", $p1))>;
def OP_REINT    : Op<(bitcast "R", $p0)>;
def OP_ADDHNHi  : Op<(call "vcombine", $p0, (call "vaddhn", $p1, $p2))>;
def OP_RADDHNHi : Op<(call "vcombine", $p0, (call "vraddhn", $p1, $p2))>;
def OP_SUBHNHi  : Op<(call "vcombine", $p0, (call "vsubhn", $p1, $p2))>;
def OP_RSUBHNHi : Op<(call "vcombine", $p0, (call "vrsubhn", $p1, $p2))>;
def OP_ABDL     : Op<(bitcast "R", (call "vmovl", (bitcast $p0, "U",
                                                     (call "vabd", $p0, $p1))))>;
def OP_ABDLHi   : Op<(call "vabdl", (call "vget_high", $p0),
                                    (call "vget_high", $p1))>;
````
- **L113 EN**: Declares TableGen def record `OP_LO`.
  **L113 CN**: 声明 TableGen def 记录 `OP_LO`。
- **L114 EN**: Declares TableGen def record `OP_CONC`.
  **L114 CN**: 声明 TableGen def 记录 `OP_CONC`。
- **L115 EN**: Declares TableGen def record `OP_DUP`.
  **L115 CN**: 声明 TableGen def 记录 `OP_DUP`。
- **L116 EN**: Declares TableGen def record `OP_DUP_LN`.
  **L116 CN**: 声明 TableGen def 记录 `OP_DUP_LN`。
- **L117 EN**: Declares TableGen def record `OP_SEL`.
  **L117 CN**: 声明 TableGen def 记录 `OP_SEL`。
- **L118 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(op "&", $p0, (bitcast $p0, $p1)),`.
  **L118 CN**: 继续一个多行参数列表、初始化器或聚合项：`(op "&", $p0, (bitcast $p0, $p1)),`。
- **L119 EN**: Executes a call or declaration centered on `statement`.
  **L119 CN**: 执行以 `statement` 为核心的调用或声明。
- **L120 EN**: Declares TableGen def record `OP_REV16`.
  **L120 CN**: 声明 TableGen def 记录 `OP_REV16`。
- **L121 EN**: Declares TableGen def record `OP_REV32`.
  **L121 CN**: 声明 TableGen def 记录 `OP_REV32`。
- **L122 EN**: Declares TableGen def record `OP_REV64`.
  **L122 CN**: 声明 TableGen def 记录 `OP_REV64`。
- **L123 EN**: Declares TableGen def record `OP_XTN`.
  **L123 CN**: 声明 TableGen def 记录 `OP_XTN`。
- **L124 EN**: Declares TableGen def record `OP_SQXTUN`.
  **L124 CN**: 声明 TableGen def 记录 `OP_SQXTUN`。
- **L125 EN**: Executes a call or declaration centered on `statement`.
  **L125 CN**: 执行以 `statement` 为核心的调用或声明。
- **L126 EN**: Declares TableGen def record `OP_QXTN`.
  **L126 CN**: 声明 TableGen def 记录 `OP_QXTN`。
- **L127 EN**: Declares TableGen def record `OP_VCVT_NA_HI_F16`.
  **L127 CN**: 声明 TableGen def 记录 `OP_VCVT_NA_HI_F16`。
- **L128 EN**: Declares TableGen def record `OP_VCVT_NA_HI_F32`.
  **L128 CN**: 声明 TableGen def 记录 `OP_VCVT_NA_HI_F32`。
- **L129 EN**: Declares TableGen def record `OP_VCVT_EX_HI_F32`.
  **L129 CN**: 声明 TableGen def 记录 `OP_VCVT_EX_HI_F32`。
- **L130 EN**: Declares TableGen def record `OP_VCVT_EX_HI_F64`.
  **L130 CN**: 声明 TableGen def 记录 `OP_VCVT_EX_HI_F64`。
- **L131 EN**: Declares TableGen def record `OP_VCVTX_HI`.
  **L131 CN**: 声明 TableGen def 记录 `OP_VCVTX_HI`。
- **L132 EN**: Declares TableGen def record `OP_REINT`.
  **L132 CN**: 声明 TableGen def 记录 `OP_REINT`。
- **L133 EN**: Declares TableGen def record `OP_ADDHNHi`.
  **L133 CN**: 声明 TableGen def 记录 `OP_ADDHNHi`。
- **L134 EN**: Declares TableGen def record `OP_RADDHNHi`.
  **L134 CN**: 声明 TableGen def 记录 `OP_RADDHNHi`。
- **L135 EN**: Declares TableGen def record `OP_SUBHNHi`.
  **L135 CN**: 声明 TableGen def 记录 `OP_SUBHNHi`。
- **L136 EN**: Declares TableGen def record `OP_RSUBHNHi`.
  **L136 CN**: 声明 TableGen def 记录 `OP_RSUBHNHi`。
- **L137 EN**: Declares TableGen def record `OP_ABDL`.
  **L137 CN**: 声明 TableGen def 记录 `OP_ABDL`。
- **L138 EN**: Executes a call or declaration centered on `statement`.
  **L138 CN**: 执行以 `statement` 为核心的调用或声明。
- **L139 EN**: Declares TableGen def record `OP_ABDLHi`.
  **L139 CN**: 声明 TableGen def 记录 `OP_ABDLHi`。
- **L140 EN**: Executes a call or declaration centered on `statement`.
  **L140 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 141-168

````tablegen
def OP_ABA      : Op<(op "+", $p0, (call "vabd", $p1, $p2))>;
def OP_ABAL     : Op<(op "+", $p0, (call "vabdl", $p1, $p2))>;
def OP_ABALHi   : Op<(call "vabal", $p0, (call "vget_high", $p1),
                                       (call "vget_high", $p2))>;
def OP_QDMULLHi : Op<(call "vqdmull", (call "vget_high", $p0),
                                      (call "vget_high", $p1))>;
def OP_QDMULLHi_N : Op<(call "vqdmull_n", (call "vget_high", $p0), $p1)>;
def OP_QDMLALHi : Op<(call "vqdmlal", $p0, (call "vget_high", $p1),
                                           (call "vget_high", $p2))>;
def OP_QDMLALHi_N : Op<(call "vqdmlal_n", $p0, (call "vget_high", $p1), $p2)>;
def OP_QDMLSLHi : Op<(call "vqdmlsl", $p0, (call "vget_high", $p1),
                                           (call "vget_high", $p2))>;
def OP_QDMLSLHi_N : Op<(call "vqdmlsl_n", $p0, (call "vget_high", $p1), $p2)>;
def OP_DIV  : Op<(op "/", $p0, $p1)>;
def OP_LONG_HI : Op<(bitcast "R", (call (name_replace "_high_", "_"),
                                                (call "vget_high", $p0), $p1))>;
def OP_NARROW_HI : Op<(bitcast "R", (call "vcombine",
                                       (bitcast "R", "H", $p0),
                                       (bitcast "R", "H",
                                           (call (name_replace "_high_", "_"),
                                                 $p1, $p2))))>;
def OP_MOVL_HI  : LOp<[(save_temp $a1, (call "vget_high", $p0)),
                       (bitcast "R",
                            (call "vshll_n", $a1, (literal "int32_t", "0")))]>;
def OP_COPY_LN : Op<(call "vset_lane", (call "vget_lane", $p2, $p3), $p0, $p1)>;
def OP_SCALAR_MUL_LN : Op<(op "*", $p0, (call "vget_lane", $p1, $p2))>;
def OP_SCALAR_MULX_LN : Op<(call "vmulx", $p0, (call "vget_lane", $p1, $p2))>;
def OP_SCALAR_VMULX_LN : LOp<[(save_temp $x, (call "vget_lane", $p0,
````
- **L141 EN**: Declares TableGen def record `OP_ABA`.
  **L141 CN**: 声明 TableGen def 记录 `OP_ABA`。
- **L142 EN**: Declares TableGen def record `OP_ABAL`.
  **L142 CN**: 声明 TableGen def 记录 `OP_ABAL`。
- **L143 EN**: Declares TableGen def record `OP_ABALHi`.
  **L143 CN**: 声明 TableGen def 记录 `OP_ABALHi`。
- **L144 EN**: Executes a call or declaration centered on `statement`.
  **L144 CN**: 执行以 `statement` 为核心的调用或声明。
- **L145 EN**: Declares TableGen def record `OP_QDMULLHi`.
  **L145 CN**: 声明 TableGen def 记录 `OP_QDMULLHi`。
- **L146 EN**: Executes a call or declaration centered on `statement`.
  **L146 CN**: 执行以 `statement` 为核心的调用或声明。
- **L147 EN**: Declares TableGen def record `OP_QDMULLHi_N`.
  **L147 CN**: 声明 TableGen def 记录 `OP_QDMULLHi_N`。
- **L148 EN**: Declares TableGen def record `OP_QDMLALHi`.
  **L148 CN**: 声明 TableGen def 记录 `OP_QDMLALHi`。
- **L149 EN**: Executes a call or declaration centered on `statement`.
  **L149 CN**: 执行以 `statement` 为核心的调用或声明。
- **L150 EN**: Declares TableGen def record `OP_QDMLALHi_N`.
  **L150 CN**: 声明 TableGen def 记录 `OP_QDMLALHi_N`。
- **L151 EN**: Declares TableGen def record `OP_QDMLSLHi`.
  **L151 CN**: 声明 TableGen def 记录 `OP_QDMLSLHi`。
- **L152 EN**: Executes a call or declaration centered on `statement`.
  **L152 CN**: 执行以 `statement` 为核心的调用或声明。
- **L153 EN**: Declares TableGen def record `OP_QDMLSLHi_N`.
  **L153 CN**: 声明 TableGen def 记录 `OP_QDMLSLHi_N`。
- **L154 EN**: Declares TableGen def record `OP_DIV`.
  **L154 CN**: 声明 TableGen def 记录 `OP_DIV`。
- **L155 EN**: Declares TableGen def record `OP_LONG_HI`.
  **L155 CN**: 声明 TableGen def 记录 `OP_LONG_HI`。
- **L156 EN**: Executes a call or declaration centered on `statement`.
  **L156 CN**: 执行以 `statement` 为核心的调用或声明。
- **L157 EN**: Declares TableGen def record `OP_NARROW_HI`.
  **L157 CN**: 声明 TableGen def 记录 `OP_NARROW_HI`。
- **L158 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(bitcast "R", "H", $p0),`.
  **L158 CN**: 继续一个多行参数列表、初始化器或聚合项：`(bitcast "R", "H", $p0),`。
- **L159 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(bitcast "R", "H",`.
  **L159 CN**: 继续一个多行参数列表、初始化器或聚合项：`(bitcast "R", "H",`。
- **L160 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(call (name_replace "_high_", "_"),`.
  **L160 CN**: 继续一个多行参数列表、初始化器或聚合项：`(call (name_replace "_high_", "_"),`。
- **L161 EN**: Adds a standalone statement or declaration: `$p1, $p2))))>;`.
  **L161 CN**: 添加一条独立语句或声明：`$p1, $p2))))>;`。
- **L162 EN**: Declares TableGen def record `OP_MOVL_HI`.
  **L162 CN**: 声明 TableGen def 记录 `OP_MOVL_HI`。
- **L163 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(bitcast "R",`.
  **L163 CN**: 继续一个多行参数列表、初始化器或聚合项：`(bitcast "R",`。
- **L164 EN**: Executes a call or declaration centered on `statement`.
  **L164 CN**: 执行以 `statement` 为核心的调用或声明。
- **L165 EN**: Declares TableGen def record `OP_COPY_LN`.
  **L165 CN**: 声明 TableGen def 记录 `OP_COPY_LN`。
- **L166 EN**: Declares TableGen def record `OP_SCALAR_MUL_LN`.
  **L166 CN**: 声明 TableGen def 记录 `OP_SCALAR_MUL_LN`。
- **L167 EN**: Declares TableGen def record `OP_SCALAR_MULX_LN`.
  **L167 CN**: 声明 TableGen def 记录 `OP_SCALAR_MULX_LN`。
- **L168 EN**: Declares TableGen def record `OP_SCALAR_VMULX_LN`.
  **L168 CN**: 声明 TableGen def 记录 `OP_SCALAR_VMULX_LN`。

### Lines 169-196

````tablegen
                                                    (literal "int32_t", "0"))),
                              (save_temp $y, (call "vget_lane", $p1, $p2)),
                              (save_temp $z, (call "vmulx", $x, $y)),
                              (call "vset_lane", $z, $p0, $p2)]>;
def OP_SCALAR_VMULX_LNQ : LOp<[(save_temp $x, (call "vget_lane", $p0,
                                                     (literal "int32_t", "0"))),
                               (save_temp $y, (call "vget_lane", $p1, $p2)),
                               (save_temp $z, (call "vmulx", $x, $y)),
                               (call "vset_lane", $z, $p0, (literal "int32_t",
                                                                     "0"))]>;
class ScalarMulOp<string opname> :
  Op<(call opname, $p0, (call "vget_lane", $p1, $p2))>;

def OP_SCALAR_QDMULL_LN : ScalarMulOp<"vqdmull">;
def OP_SCALAR_QDMULH_LN : ScalarMulOp<"vqdmulh">;
def OP_SCALAR_QRDMULH_LN : ScalarMulOp<"vqrdmulh">;

def OP_SCALAR_QRDMLAH_LN : Op<(call "vqrdmlah", $p0, $p1,
                                (call "vget_lane", $p2, $p3))>;
def OP_SCALAR_QRDMLSH_LN : Op<(call "vqrdmlsh", $p0, $p1,
                                (call "vget_lane", $p2, $p3))>;

def OP_SCALAR_HALF_GET_LN : Op<(bitcast "float16_t",
                                   (call "vget_lane",
                                         (bitcast "int16x4_t", $p0), $p1))>;
def OP_SCALAR_HALF_GET_LNQ : Op<(bitcast "float16_t",
                                    (call "vget_lane",
                                          (bitcast "int16x8_t", $p0), $p1))>;
````
- **L169 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(literal "int32_t", "0"))),`.
  **L169 CN**: 继续一个多行参数列表、初始化器或聚合项：`(literal "int32_t", "0"))),`。
- **L170 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(save_temp $y, (call "vget_lane", $p1, $p2)),`.
  **L170 CN**: 继续一个多行参数列表、初始化器或聚合项：`(save_temp $y, (call "vget_lane", $p1, $p2)),`。
- **L171 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(save_temp $z, (call "vmulx", $x, $y)),`.
  **L171 CN**: 继续一个多行参数列表、初始化器或聚合项：`(save_temp $z, (call "vmulx", $x, $y)),`。
- **L172 EN**: Executes a call or declaration centered on `statement`.
  **L172 CN**: 执行以 `statement` 为核心的调用或声明。
- **L173 EN**: Declares TableGen def record `OP_SCALAR_VMULX_LNQ`.
  **L173 CN**: 声明 TableGen def 记录 `OP_SCALAR_VMULX_LNQ`。
- **L174 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(literal "int32_t", "0"))),`.
  **L174 CN**: 继续一个多行参数列表、初始化器或聚合项：`(literal "int32_t", "0"))),`。
- **L175 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(save_temp $y, (call "vget_lane", $p1, $p2)),`.
  **L175 CN**: 继续一个多行参数列表、初始化器或聚合项：`(save_temp $y, (call "vget_lane", $p1, $p2)),`。
- **L176 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(save_temp $z, (call "vmulx", $x, $y)),`.
  **L176 CN**: 继续一个多行参数列表、初始化器或聚合项：`(save_temp $z, (call "vmulx", $x, $y)),`。
- **L177 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(call "vset_lane", $z, $p0, (literal "int32_t",`.
  **L177 CN**: 继续一个多行参数列表、初始化器或聚合项：`(call "vset_lane", $z, $p0, (literal "int32_t",`。
- **L178 EN**: Adds a standalone statement or declaration: `"0"))]>;`.
  **L178 CN**: 添加一条独立语句或声明：`"0"))]>;`。
- **L179 EN**: Declares TableGen class record `ScalarMulOp`.
  **L179 CN**: 声明 TableGen class 记录 `ScalarMulOp`。
- **L180 EN**: Executes a call or declaration centered on `Op<`.
  **L180 CN**: 执行以 `Op<` 为核心的调用或声明。
- **L181 EN**: Blank line separating nearby declarations or logic blocks.
  **L181 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L182 EN**: Declares TableGen def record `OP_SCALAR_QDMULL_LN`.
  **L182 CN**: 声明 TableGen def 记录 `OP_SCALAR_QDMULL_LN`。
- **L183 EN**: Declares TableGen def record `OP_SCALAR_QDMULH_LN`.
  **L183 CN**: 声明 TableGen def 记录 `OP_SCALAR_QDMULH_LN`。
- **L184 EN**: Declares TableGen def record `OP_SCALAR_QRDMULH_LN`.
  **L184 CN**: 声明 TableGen def 记录 `OP_SCALAR_QRDMULH_LN`。
- **L185 EN**: Blank line separating nearby declarations or logic blocks.
  **L185 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L186 EN**: Declares TableGen def record `OP_SCALAR_QRDMLAH_LN`.
  **L186 CN**: 声明 TableGen def 记录 `OP_SCALAR_QRDMLAH_LN`。
- **L187 EN**: Executes a call or declaration centered on `statement`.
  **L187 CN**: 执行以 `statement` 为核心的调用或声明。
- **L188 EN**: Declares TableGen def record `OP_SCALAR_QRDMLSH_LN`.
  **L188 CN**: 声明 TableGen def 记录 `OP_SCALAR_QRDMLSH_LN`。
- **L189 EN**: Executes a call or declaration centered on `statement`.
  **L189 CN**: 执行以 `statement` 为核心的调用或声明。
- **L190 EN**: Blank line separating nearby declarations or logic blocks.
  **L190 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L191 EN**: Declares TableGen def record `OP_SCALAR_HALF_GET_LN`.
  **L191 CN**: 声明 TableGen def 记录 `OP_SCALAR_HALF_GET_LN`。
- **L192 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(call "vget_lane",`.
  **L192 CN**: 继续一个多行参数列表、初始化器或聚合项：`(call "vget_lane",`。
- **L193 EN**: Executes a call or declaration centered on `statement`.
  **L193 CN**: 执行以 `statement` 为核心的调用或声明。
- **L194 EN**: Declares TableGen def record `OP_SCALAR_HALF_GET_LNQ`.
  **L194 CN**: 声明 TableGen def 记录 `OP_SCALAR_HALF_GET_LNQ`。
- **L195 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(call "vget_lane",`.
  **L195 CN**: 继续一个多行参数列表、初始化器或聚合项：`(call "vget_lane",`。
- **L196 EN**: Executes a call or declaration centered on `statement`.
  **L196 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 197-224

````tablegen
def OP_SCALAR_HALF_SET_LN : Op<(bitcast "float16x4_t",
                                   (call "vset_lane",
                                         (bitcast "int16_t", $p0),
                                         (bitcast "int16x4_t", $p1), $p2))>;
def OP_SCALAR_HALF_SET_LNQ : Op<(bitcast "float16x8_t",
                                    (call "vset_lane",
                                          (bitcast "int16_t", $p0),
                                          (bitcast "int16x8_t", $p1), $p2))>;

def OP_DOT_LN
    : Op<(call "vdot", $p0, $p1,
          (bitcast $p1, (call_mangled "splat_lane", (bitcast "32", $p2), $p3)))>;
def OP_DOT_LNQ
    : Op<(call "vdot", $p0, $p1,
          (bitcast $p1, (call_mangled "splat_lane", (bitcast "32", $p2), $p3)))>;

def OP_FMLAL_LN     : Op<(call "vfmlal_low", $p0, $p1,
                           (dup_typed $p1, (call "vget_lane", $p2, $p3)))>;
def OP_FMLSL_LN     : Op<(call "vfmlsl_low", $p0, $p1,
                           (dup_typed $p1, (call "vget_lane", $p2, $p3)))>;
def OP_FMLAL_LN_Hi  : Op<(call "vfmlal_high", $p0, $p1,
                           (dup_typed $p1, (call "vget_lane", $p2, $p3)))>;
def OP_FMLSL_LN_Hi  : Op<(call "vfmlsl_high", $p0, $p1,
                           (dup_typed $p1, (call "vget_lane", $p2, $p3)))>;

def OP_USDOT_LN
    : Op<(call "vusdot", $p0, $p1,
          (bitcast "8", "S", (call_mangled "splat_lane", (bitcast "int32x2_t", $p2), $p3)))>;
````
- **L197 EN**: Declares TableGen def record `OP_SCALAR_HALF_SET_LN`.
  **L197 CN**: 声明 TableGen def 记录 `OP_SCALAR_HALF_SET_LN`。
- **L198 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(call "vset_lane",`.
  **L198 CN**: 继续一个多行参数列表、初始化器或聚合项：`(call "vset_lane",`。
- **L199 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(bitcast "int16_t", $p0),`.
  **L199 CN**: 继续一个多行参数列表、初始化器或聚合项：`(bitcast "int16_t", $p0),`。
- **L200 EN**: Executes a call or declaration centered on `statement`.
  **L200 CN**: 执行以 `statement` 为核心的调用或声明。
- **L201 EN**: Declares TableGen def record `OP_SCALAR_HALF_SET_LNQ`.
  **L201 CN**: 声明 TableGen def 记录 `OP_SCALAR_HALF_SET_LNQ`。
- **L202 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(call "vset_lane",`.
  **L202 CN**: 继续一个多行参数列表、初始化器或聚合项：`(call "vset_lane",`。
- **L203 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(bitcast "int16_t", $p0),`.
  **L203 CN**: 继续一个多行参数列表、初始化器或聚合项：`(bitcast "int16_t", $p0),`。
- **L204 EN**: Executes a call or declaration centered on `statement`.
  **L204 CN**: 执行以 `statement` 为核心的调用或声明。
- **L205 EN**: Blank line separating nearby declarations or logic blocks.
  **L205 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L206 EN**: Declares TableGen def record `OP_DOT_LN`.
  **L206 CN**: 声明 TableGen def 记录 `OP_DOT_LN`。
- **L207 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Op<(call "vdot", $p0, $p1,`.
  **L207 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Op<(call "vdot", $p0, $p1,`。
- **L208 EN**: Executes a call or declaration centered on `statement`.
  **L208 CN**: 执行以 `statement` 为核心的调用或声明。
- **L209 EN**: Declares TableGen def record `OP_DOT_LNQ`.
  **L209 CN**: 声明 TableGen def 记录 `OP_DOT_LNQ`。
- **L210 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Op<(call "vdot", $p0, $p1,`.
  **L210 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Op<(call "vdot", $p0, $p1,`。
- **L211 EN**: Executes a call or declaration centered on `statement`.
  **L211 CN**: 执行以 `statement` 为核心的调用或声明。
- **L212 EN**: Blank line separating nearby declarations or logic blocks.
  **L212 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L213 EN**: Declares TableGen def record `OP_FMLAL_LN`.
  **L213 CN**: 声明 TableGen def 记录 `OP_FMLAL_LN`。
- **L214 EN**: Executes a call or declaration centered on `statement`.
  **L214 CN**: 执行以 `statement` 为核心的调用或声明。
- **L215 EN**: Declares TableGen def record `OP_FMLSL_LN`.
  **L215 CN**: 声明 TableGen def 记录 `OP_FMLSL_LN`。
- **L216 EN**: Executes a call or declaration centered on `statement`.
  **L216 CN**: 执行以 `statement` 为核心的调用或声明。
- **L217 EN**: Declares TableGen def record `OP_FMLAL_LN_Hi`.
  **L217 CN**: 声明 TableGen def 记录 `OP_FMLAL_LN_Hi`。
- **L218 EN**: Executes a call or declaration centered on `statement`.
  **L218 CN**: 执行以 `statement` 为核心的调用或声明。
- **L219 EN**: Declares TableGen def record `OP_FMLSL_LN_Hi`.
  **L219 CN**: 声明 TableGen def 记录 `OP_FMLSL_LN_Hi`。
- **L220 EN**: Executes a call or declaration centered on `statement`.
  **L220 CN**: 执行以 `statement` 为核心的调用或声明。
- **L221 EN**: Blank line separating nearby declarations or logic blocks.
  **L221 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L222 EN**: Declares TableGen def record `OP_USDOT_LN`.
  **L222 CN**: 声明 TableGen def 记录 `OP_USDOT_LN`。
- **L223 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Op<(call "vusdot", $p0, $p1,`.
  **L223 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Op<(call "vusdot", $p0, $p1,`。
- **L224 EN**: Executes a call or declaration centered on `statement`.
  **L224 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 225-252

````tablegen
def OP_USDOT_LNQ
    : Op<(call "vusdot", $p0, $p1,
          (bitcast "8", "S", (call_mangled "splat_lane", (bitcast "int32x4_t", $p2), $p3)))>;

// sudot splats the second vector and then calls vusdot
def OP_SUDOT_LN
    : Op<(call "vusdot", $p0,
          (bitcast "8", "U", (call_mangled "splat_lane", (bitcast "int32x2_t", $p2), $p3)), $p1)>;
def OP_SUDOT_LNQ
    : Op<(call "vusdot", $p0,
          (bitcast "8", "U", (call_mangled "splat_lane", (bitcast "int32x4_t", $p2), $p3)), $p1)>;

def OP_BFDOT_LN
    : Op<(call "vbfdot", $p0, $p1,
          (bitcast $p1, (call_mangled "splat_lane", (bitcast "float32x2_t", $p2), $p3)))>;

def OP_BFDOT_LNQ
    : Op<(call "vbfdot", $p0, $p1,
          (bitcast $p1, (call_mangled "splat_lane", (bitcast "float32x4_t", $p2), $p3)))>;

def OP_BFMLALB_LN
    : Op<(call "vbfmlalb", $p0, $p1,
          (dup_typed $p1, (call "vget_lane", $p2, $p3)))>;

def OP_BFMLALT_LN
    : Op<(call "vbfmlalt", $p0, $p1,
          (dup_typed $p1, (call "vget_lane", $p2, $p3)))>;

````
- **L225 EN**: Declares TableGen def record `OP_USDOT_LNQ`.
  **L225 CN**: 声明 TableGen def 记录 `OP_USDOT_LNQ`。
- **L226 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Op<(call "vusdot", $p0, $p1,`.
  **L226 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Op<(call "vusdot", $p0, $p1,`。
- **L227 EN**: Executes a call or declaration centered on `statement`.
  **L227 CN**: 执行以 `statement` 为核心的调用或声明。
- **L228 EN**: Blank line separating nearby declarations or logic blocks.
  **L228 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L229 EN**: Comment explains nearby logic, constraints, or intent: `sudot splats the second vector and then calls vusdot`.
  **L229 CN**: 注释解释附近代码的逻辑、约束或设计意图：`sudot splats the second vector and then calls vusdot`。
- **L230 EN**: Declares TableGen def record `OP_SUDOT_LN`.
  **L230 CN**: 声明 TableGen def 记录 `OP_SUDOT_LN`。
- **L231 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Op<(call "vusdot", $p0,`.
  **L231 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Op<(call "vusdot", $p0,`。
- **L232 EN**: Executes a call or declaration centered on `statement`.
  **L232 CN**: 执行以 `statement` 为核心的调用或声明。
- **L233 EN**: Declares TableGen def record `OP_SUDOT_LNQ`.
  **L233 CN**: 声明 TableGen def 记录 `OP_SUDOT_LNQ`。
- **L234 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Op<(call "vusdot", $p0,`.
  **L234 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Op<(call "vusdot", $p0,`。
- **L235 EN**: Executes a call or declaration centered on `statement`.
  **L235 CN**: 执行以 `statement` 为核心的调用或声明。
- **L236 EN**: Blank line separating nearby declarations or logic blocks.
  **L236 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L237 EN**: Declares TableGen def record `OP_BFDOT_LN`.
  **L237 CN**: 声明 TableGen def 记录 `OP_BFDOT_LN`。
- **L238 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Op<(call "vbfdot", $p0, $p1,`.
  **L238 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Op<(call "vbfdot", $p0, $p1,`。
- **L239 EN**: Executes a call or declaration centered on `statement`.
  **L239 CN**: 执行以 `statement` 为核心的调用或声明。
- **L240 EN**: Blank line separating nearby declarations or logic blocks.
  **L240 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L241 EN**: Declares TableGen def record `OP_BFDOT_LNQ`.
  **L241 CN**: 声明 TableGen def 记录 `OP_BFDOT_LNQ`。
- **L242 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Op<(call "vbfdot", $p0, $p1,`.
  **L242 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Op<(call "vbfdot", $p0, $p1,`。
- **L243 EN**: Executes a call or declaration centered on `statement`.
  **L243 CN**: 执行以 `statement` 为核心的调用或声明。
- **L244 EN**: Blank line separating nearby declarations or logic blocks.
  **L244 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L245 EN**: Declares TableGen def record `OP_BFMLALB_LN`.
  **L245 CN**: 声明 TableGen def 记录 `OP_BFMLALB_LN`。
- **L246 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Op<(call "vbfmlalb", $p0, $p1,`.
  **L246 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Op<(call "vbfmlalb", $p0, $p1,`。
- **L247 EN**: Executes a call or declaration centered on `statement`.
  **L247 CN**: 执行以 `statement` 为核心的调用或声明。
- **L248 EN**: Blank line separating nearby declarations or logic blocks.
  **L248 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L249 EN**: Declares TableGen def record `OP_BFMLALT_LN`.
  **L249 CN**: 声明 TableGen def 记录 `OP_BFMLALT_LN`。
- **L250 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Op<(call "vbfmlalt", $p0, $p1,`.
  **L250 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Op<(call "vbfmlalt", $p0, $p1,`。
- **L251 EN**: Executes a call or declaration centered on `statement`.
  **L251 CN**: 执行以 `statement` 为核心的调用或声明。
- **L252 EN**: Blank line separating nearby declarations or logic blocks.
  **L252 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 253-280

````tablegen
def OP_VCVT_F32_BF16
    : Op<(bitcast "R",
          (call "vshll_n", (bitcast "uint16x4_t", $p0),
                           (literal "int32_t", "16")))>;
def OP_VCVT_F32_BF16_LO
    : Op<(call "vcvt_f32_bf16", (call "vget_low", $p0))>;
def OP_VCVT_F32_BF16_HI
    : Op<(call "vcvt_f32_bf16", (call "vget_high", $p0))>;

def OP_VCVT_BF16_F32_A32
    : Op<(call "__a32_vcvt_bf16", $p0)>;

def OP_VCVT_BF16_F32_LO_A32
    : Op<(call "vcombine", (bitcast "bfloat16x4_t", (literal "uint64_t", "0ULL")),
                           (call "__a32_vcvt_bf16", $p0))>;
def OP_VCVT_BF16_F32_HI_A32
    : Op<(call "vcombine", (call "__a32_vcvt_bf16", $p1),
                           (call "vget_low", $p0))>;

def OP_CVT_F32_BF16
    : Op<(bitcast "R", (op "<<", (cast "uint32_t", (bitcast "uint16_t", $p0)),
                                 (literal "uint32_t", "16")))>;

//===----------------------------------------------------------------------===//
// Auxiliary Instructions
//===----------------------------------------------------------------------===//

// Splat operation - performs a range-checked splat over a vector
````
- **L253 EN**: Declares TableGen def record `OP_VCVT_F32_BF16`.
  **L253 CN**: 声明 TableGen def 记录 `OP_VCVT_F32_BF16`。
- **L254 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Op<(bitcast "R",`.
  **L254 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Op<(bitcast "R",`。
- **L255 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(call "vshll_n", (bitcast "uint16x4_t", $p0),`.
  **L255 CN**: 继续一个多行参数列表、初始化器或聚合项：`(call "vshll_n", (bitcast "uint16x4_t", $p0),`。
- **L256 EN**: Executes a call or declaration centered on `statement`.
  **L256 CN**: 执行以 `statement` 为核心的调用或声明。
- **L257 EN**: Declares TableGen def record `OP_VCVT_F32_BF16_LO`.
  **L257 CN**: 声明 TableGen def 记录 `OP_VCVT_F32_BF16_LO`。
- **L258 EN**: Executes a call or declaration centered on `Op<`.
  **L258 CN**: 执行以 `Op<` 为核心的调用或声明。
- **L259 EN**: Declares TableGen def record `OP_VCVT_F32_BF16_HI`.
  **L259 CN**: 声明 TableGen def 记录 `OP_VCVT_F32_BF16_HI`。
- **L260 EN**: Executes a call or declaration centered on `Op<`.
  **L260 CN**: 执行以 `Op<` 为核心的调用或声明。
- **L261 EN**: Blank line separating nearby declarations or logic blocks.
  **L261 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L262 EN**: Declares TableGen def record `OP_VCVT_BF16_F32_A32`.
  **L262 CN**: 声明 TableGen def 记录 `OP_VCVT_BF16_F32_A32`。
- **L263 EN**: Executes a call or declaration centered on `Op<`.
  **L263 CN**: 执行以 `Op<` 为核心的调用或声明。
- **L264 EN**: Blank line separating nearby declarations or logic blocks.
  **L264 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L265 EN**: Declares TableGen def record `OP_VCVT_BF16_F32_LO_A32`.
  **L265 CN**: 声明 TableGen def 记录 `OP_VCVT_BF16_F32_LO_A32`。
- **L266 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Op<(call "vcombine", (bitcast "bfloat16x4_t", (literal "uint64_t", "0ULL")),`.
  **L266 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Op<(call "vcombine", (bitcast "bfloat16x4_t", (literal "uint64_t", "0ULL")),`。
- **L267 EN**: Executes a call or declaration centered on `statement`.
  **L267 CN**: 执行以 `statement` 为核心的调用或声明。
- **L268 EN**: Declares TableGen def record `OP_VCVT_BF16_F32_HI_A32`.
  **L268 CN**: 声明 TableGen def 记录 `OP_VCVT_BF16_F32_HI_A32`。
- **L269 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Op<(call "vcombine", (call "__a32_vcvt_bf16", $p1),`.
  **L269 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Op<(call "vcombine", (call "__a32_vcvt_bf16", $p1),`。
- **L270 EN**: Executes a call or declaration centered on `statement`.
  **L270 CN**: 执行以 `statement` 为核心的调用或声明。
- **L271 EN**: Blank line separating nearby declarations or logic blocks.
  **L271 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L272 EN**: Declares TableGen def record `OP_CVT_F32_BF16`.
  **L272 CN**: 声明 TableGen def 记录 `OP_CVT_F32_BF16`。
- **L273 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Op<(bitcast "R", (op "<<", (cast "uint32_t", (bitcast "uint16_t", $p0)),`.
  **L273 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Op<(bitcast "R", (op "<<", (cast "uint32_t", (bitcast "uint16_t", $p0)),`。
- **L274 EN**: Executes a call or declaration centered on `statement`.
  **L274 CN**: 执行以 `statement` 为核心的调用或声明。
- **L275 EN**: Blank line separating nearby declarations or logic blocks.
  **L275 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L276 EN**: Banner comment marking a file or section boundary.
  **L276 CN**: 横幅注释，用于标记文件或章节边界。
- **L277 EN**: Comment explains nearby logic, constraints, or intent: `Auxiliary Instructions`.
  **L277 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Auxiliary Instructions`。
- **L278 EN**: Banner comment marking a file or section boundary.
  **L278 CN**: 横幅注释，用于标记文件或章节边界。
- **L279 EN**: Blank line separating nearby declarations or logic blocks.
  **L279 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L280 EN**: Comment explains nearby logic, constraints, or intent: `Splat operation - performs a range-checked splat over a vector`.
  **L280 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Splat operation - performs a range-checked splat over a vector`。

### Lines 281-308

````tablegen
def SPLAT  : WInst<"splat_lane", ".(!q)I",
                   "UcUsUicsilPcPsfQUcQUsQUiQcQsQiQPcQPsQflUlQlQUlhdQhQdPlQPlmQm",
                    [ImmCheck<1, ImmCheckLaneIndex, 0>]>;
def SPLATQ : WInst<"splat_laneq", ".(!Q)I",
                   "UcUsUicsilPcPsfQUcQUsQUiQcQsQiQPcQPsQflUlQlQUlhdQhQdPlQPlmQm",
                   [ImmCheck<1, ImmCheckLaneIndex, 0>]>;

let TargetGuard = "bf16,neon" in {
  def SPLAT_BF  : WInst<"splat_lane", ".(!q)I", "bQb",
                      [ImmCheck<1, ImmCheckLaneIndex, 0>]>;
  def SPLATQ_BF : WInst<"splat_laneq", ".(!Q)I", "bQb",
                      [ImmCheck<1, ImmCheckLaneIndex, 0>]>;
}

//===----------------------------------------------------------------------===//
// Intrinsics
//===----------------------------------------------------------------------===//

////////////////////////////////////////////////////////////////////////////////
// E.3.1 Addition
def VADD    : IOpInst<"vadd", "...",
                      "csilfUcUsUiUlQcQsQiQlQfQUcQUsQUiQUl", OP_ADD>;
def VADDL   : SOpInst<"vaddl", "(>Q)..", "csiUcUsUi", OP_ADDL>;
def VADDW   : SOpInst<"vaddw", "(>Q)(>Q).", "csiUcUsUi", OP_ADDW>;
def VHADD   : SInst<"vhadd", "...", "csiUcUsUiQcQsQiQUcQUsQUi">;
def VRHADD  : SInst<"vrhadd", "...", "csiUcUsUiQcQsQiQUcQUsQUi">;
def VQADD   : SInst<"vqadd", "...", "csilUcUsUiUlQcQsQiQlQUcQUsQUiQUl">;
def VADDHN  : IInst<"vaddhn", "<QQ", "silUsUiUl">;
````
- **L281 EN**: Declares TableGen def record `SPLAT`.
  **L281 CN**: 声明 TableGen def 记录 `SPLAT`。
- **L282 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"UcUsUicsilPcPsfQUcQUsQUiQcQsQiQPcQPsQflUlQlQUlhdQhQdPlQPlmQm",`.
  **L282 CN**: 继续一个多行参数列表、初始化器或聚合项：`"UcUsUicsilPcPsfQUcQUsQUiQcQsQiQPcQPsQflUlQlQUlhdQhQdPlQPlmQm",`。
- **L283 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheckLaneIndex, 0>]>;`.
  **L283 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheckLaneIndex, 0>]>;`。
- **L284 EN**: Declares TableGen def record `SPLATQ`.
  **L284 CN**: 声明 TableGen def 记录 `SPLATQ`。
- **L285 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"UcUsUicsilPcPsfQUcQUsQUiQcQsQiQPcQPsQflUlQlQUlhdQhQdPlQPlmQm",`.
  **L285 CN**: 继续一个多行参数列表、初始化器或聚合项：`"UcUsUicsilPcPsfQUcQUsQUiQcQsQiQPcQPsQflUlQlQUlhdQhQdPlQPlmQm",`。
- **L286 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheckLaneIndex, 0>]>;`.
  **L286 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheckLaneIndex, 0>]>;`。
- **L287 EN**: Blank line separating nearby declarations or logic blocks.
  **L287 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L288 EN**: Assigns a TableGen property that affects following records or inherited fields: `let TargetGuard = "bf16,neon" in {`.
  **L288 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let TargetGuard = "bf16,neon" in {`。
- **L289 EN**: Declares TableGen def record `SPLAT_BF`.
  **L289 CN**: 声明 TableGen def 记录 `SPLAT_BF`。
- **L290 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheckLaneIndex, 0>]>;`.
  **L290 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheckLaneIndex, 0>]>;`。
- **L291 EN**: Declares TableGen def record `SPLATQ_BF`.
  **L291 CN**: 声明 TableGen def 记录 `SPLATQ_BF`。
- **L292 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheckLaneIndex, 0>]>;`.
  **L292 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheckLaneIndex, 0>]>;`。
- **L293 EN**: Closes the current lexical scope or compound statement.
  **L293 CN**: 结束当前词法作用域或复合语句块。
- **L294 EN**: Blank line separating nearby declarations or logic blocks.
  **L294 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L295 EN**: Banner comment marking a file or section boundary.
  **L295 CN**: 横幅注释，用于标记文件或章节边界。
- **L296 EN**: Comment explains nearby logic, constraints, or intent: `Intrinsics`.
  **L296 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Intrinsics`。
- **L297 EN**: Banner comment marking a file or section boundary.
  **L297 CN**: 横幅注释，用于标记文件或章节边界。
- **L298 EN**: Blank line separating nearby declarations or logic blocks.
  **L298 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L299 EN**: Separator comment used for visual grouping.
  **L299 CN**: 用于视觉分组的分隔注释。
- **L300 EN**: Comment explains nearby logic, constraints, or intent: `E.3.1 Addition`.
  **L300 CN**: 注释解释附近代码的逻辑、约束或设计意图：`E.3.1 Addition`。
- **L301 EN**: Declares TableGen def record `VADD`.
  **L301 CN**: 声明 TableGen def 记录 `VADD`。
- **L302 EN**: Adds a standalone statement or declaration: `"csilfUcUsUiUlQcQsQiQlQfQUcQUsQUiQUl", OP_ADD>;`.
  **L302 CN**: 添加一条独立语句或声明：`"csilfUcUsUiUlQcQsQiQlQfQUcQUsQUiQUl", OP_ADD>;`。
- **L303 EN**: Declares TableGen def record `VADDL`.
  **L303 CN**: 声明 TableGen def 记录 `VADDL`。
- **L304 EN**: Declares TableGen def record `VADDW`.
  **L304 CN**: 声明 TableGen def 记录 `VADDW`。
- **L305 EN**: Declares TableGen def record `VHADD`.
  **L305 CN**: 声明 TableGen def 记录 `VHADD`。
- **L306 EN**: Declares TableGen def record `VRHADD`.
  **L306 CN**: 声明 TableGen def 记录 `VRHADD`。
- **L307 EN**: Declares TableGen def record `VQADD`.
  **L307 CN**: 声明 TableGen def 记录 `VQADD`。
- **L308 EN**: Declares TableGen def record `VADDHN`.
  **L308 CN**: 声明 TableGen def 记录 `VADDHN`。

### Lines 309-336

````tablegen
def VRADDHN : IInst<"vraddhn", "<QQ", "silUsUiUl">;

////////////////////////////////////////////////////////////////////////////////
// E.3.2 Multiplication
def VMUL     : IOpInst<"vmul", "...", "csifUcUsUiQcQsQiQfQUcQUsQUi", OP_MUL>;
def VMULP    : SInst<"vmul", "...", "PcQPc">;
def VMLA     : IOpInst<"vmla", "....", "csifUcUsUiQcQsQiQfQUcQUsQUi", OP_MLA>;
def VMLAL    : SOpInst<"vmlal", "(>Q)(>Q)..", "csiUcUsUi", OP_MLAL>;
def VMLS     : IOpInst<"vmls", "....", "csifUcUsUiQcQsQiQfQUcQUsQUi", OP_MLS>;
def VMLSL    : SOpInst<"vmlsl", "(>Q)(>Q)..", "csiUcUsUi", OP_MLSL>;
def VQDMULH  : SInst<"vqdmulh", "...", "siQsQi">;
def VQRDMULH : SInst<"vqrdmulh", "...", "siQsQi">;

let TargetGuard = "v8.1a,neon" in {
def VQRDMLAH : SInst<"vqrdmlah", "....", "siQsQi">;
def VQRDMLSH : SInst<"vqrdmlsh", "....", "siQsQi">;
}

def VQDMLAL  : SInst<"vqdmlal", "(>Q)(>Q)..", "si">;
def VQDMLSL  : SInst<"vqdmlsl", "(>Q)(>Q)..", "si">;
def VMULL    : SInst<"vmull", "(>Q)..", "csiUcUsUiPc">;
def VQDMULL  : SInst<"vqdmull", "(>Q)..", "si">;

////////////////////////////////////////////////////////////////////////////////
// E.3.3 Subtraction
def VSUB    : IOpInst<"vsub", "...",
                      "csilfUcUsUiUlQcQsQiQlQfQUcQUsQUiQUl", OP_SUB>;
def VSUBL   : SOpInst<"vsubl", "(>Q)..", "csiUcUsUi", OP_SUBL>;
````
- **L309 EN**: Declares TableGen def record `VRADDHN`.
  **L309 CN**: 声明 TableGen def 记录 `VRADDHN`。
- **L310 EN**: Blank line separating nearby declarations or logic blocks.
  **L310 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L311 EN**: Separator comment used for visual grouping.
  **L311 CN**: 用于视觉分组的分隔注释。
- **L312 EN**: Comment explains nearby logic, constraints, or intent: `E.3.2 Multiplication`.
  **L312 CN**: 注释解释附近代码的逻辑、约束或设计意图：`E.3.2 Multiplication`。
- **L313 EN**: Declares TableGen def record `VMUL`.
  **L313 CN**: 声明 TableGen def 记录 `VMUL`。
- **L314 EN**: Declares TableGen def record `VMULP`.
  **L314 CN**: 声明 TableGen def 记录 `VMULP`。
- **L315 EN**: Declares TableGen def record `VMLA`.
  **L315 CN**: 声明 TableGen def 记录 `VMLA`。
- **L316 EN**: Declares TableGen def record `VMLAL`.
  **L316 CN**: 声明 TableGen def 记录 `VMLAL`。
- **L317 EN**: Declares TableGen def record `VMLS`.
  **L317 CN**: 声明 TableGen def 记录 `VMLS`。
- **L318 EN**: Declares TableGen def record `VMLSL`.
  **L318 CN**: 声明 TableGen def 记录 `VMLSL`。
- **L319 EN**: Declares TableGen def record `VQDMULH`.
  **L319 CN**: 声明 TableGen def 记录 `VQDMULH`。
- **L320 EN**: Declares TableGen def record `VQRDMULH`.
  **L320 CN**: 声明 TableGen def 记录 `VQRDMULH`。
- **L321 EN**: Blank line separating nearby declarations or logic blocks.
  **L321 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L322 EN**: Assigns a TableGen property that affects following records or inherited fields: `let TargetGuard = "v8.1a,neon" in {`.
  **L322 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let TargetGuard = "v8.1a,neon" in {`。
- **L323 EN**: Declares TableGen def record `VQRDMLAH`.
  **L323 CN**: 声明 TableGen def 记录 `VQRDMLAH`。
- **L324 EN**: Declares TableGen def record `VQRDMLSH`.
  **L324 CN**: 声明 TableGen def 记录 `VQRDMLSH`。
- **L325 EN**: Closes the current lexical scope or compound statement.
  **L325 CN**: 结束当前词法作用域或复合语句块。
- **L326 EN**: Blank line separating nearby declarations or logic blocks.
  **L326 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L327 EN**: Declares TableGen def record `VQDMLAL`.
  **L327 CN**: 声明 TableGen def 记录 `VQDMLAL`。
- **L328 EN**: Declares TableGen def record `VQDMLSL`.
  **L328 CN**: 声明 TableGen def 记录 `VQDMLSL`。
- **L329 EN**: Declares TableGen def record `VMULL`.
  **L329 CN**: 声明 TableGen def 记录 `VMULL`。
- **L330 EN**: Declares TableGen def record `VQDMULL`.
  **L330 CN**: 声明 TableGen def 记录 `VQDMULL`。
- **L331 EN**: Blank line separating nearby declarations or logic blocks.
  **L331 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L332 EN**: Separator comment used for visual grouping.
  **L332 CN**: 用于视觉分组的分隔注释。
- **L333 EN**: Comment explains nearby logic, constraints, or intent: `E.3.3 Subtraction`.
  **L333 CN**: 注释解释附近代码的逻辑、约束或设计意图：`E.3.3 Subtraction`。
- **L334 EN**: Declares TableGen def record `VSUB`.
  **L334 CN**: 声明 TableGen def 记录 `VSUB`。
- **L335 EN**: Adds a standalone statement or declaration: `"csilfUcUsUiUlQcQsQiQlQfQUcQUsQUiQUl", OP_SUB>;`.
  **L335 CN**: 添加一条独立语句或声明：`"csilfUcUsUiUlQcQsQiQlQfQUcQUsQUiQUl", OP_SUB>;`。
- **L336 EN**: Declares TableGen def record `VSUBL`.
  **L336 CN**: 声明 TableGen def 记录 `VSUBL`。

### Lines 337-364

````tablegen
def VSUBW   : SOpInst<"vsubw", "(>Q)(>Q).", "csiUcUsUi", OP_SUBW>;
def VQSUB   : SInst<"vqsub", "...", "csilUcUsUiUlQcQsQiQlQUcQUsQUiQUl">;
def VHSUB   : SInst<"vhsub", "...", "csiUcUsUiQcQsQiQUcQUsQUi">;
def VSUBHN  : IInst<"vsubhn", "<QQ", "silUsUiUl">;
def VRSUBHN : IInst<"vrsubhn", "<QQ", "silUsUiUl">;

////////////////////////////////////////////////////////////////////////////////
// E.3.4 Comparison
def VCEQ  : IOpInst<"vceq", "U..", "csifUcUsUiPcQcQsQiQfQUcQUsQUiQPc", OP_EQ>;
def VCGE  : SOpInst<"vcge", "U..", "csifUcUsUiQcQsQiQfQUcQUsQUi", OP_GE>;
let InstName = "vcge" in
def VCLE  : SOpInst<"vcle", "U..", "csifUcUsUiQcQsQiQfQUcQUsQUi", OP_LE>;
def VCGT  : SOpInst<"vcgt", "U..", "csifUcUsUiQcQsQiQfQUcQUsQUi", OP_GT>;
let InstName = "vcgt" in
def VCLT  : SOpInst<"vclt", "U..", "csifUcUsUiQcQsQiQfQUcQUsQUi", OP_LT>;
let InstName = "vacge" in {
def VCAGE : IInst<"vcage", "U..", "fQf">;
def VCALE : IInst<"vcale", "U..", "fQf">;
}
let InstName = "vacgt" in {
def VCAGT : IInst<"vcagt", "U..", "fQf">;
def VCALT : IInst<"vcalt", "U..", "fQf">;
}
def VTST  : WInst<"vtst", "U..", "csiUcUsUiPcPsQcQsQiQUcQUsQUiQPcQPs">;

////////////////////////////////////////////////////////////////////////////////
// E.3.5 Absolute Difference
def VABD  : SInst<"vabd", "...",  "csiUcUsUifQcQsQiQUcQUsQUiQf">;
````
- **L337 EN**: Declares TableGen def record `VSUBW`.
  **L337 CN**: 声明 TableGen def 记录 `VSUBW`。
- **L338 EN**: Declares TableGen def record `VQSUB`.
  **L338 CN**: 声明 TableGen def 记录 `VQSUB`。
- **L339 EN**: Declares TableGen def record `VHSUB`.
  **L339 CN**: 声明 TableGen def 记录 `VHSUB`。
- **L340 EN**: Declares TableGen def record `VSUBHN`.
  **L340 CN**: 声明 TableGen def 记录 `VSUBHN`。
- **L341 EN**: Declares TableGen def record `VRSUBHN`.
  **L341 CN**: 声明 TableGen def 记录 `VRSUBHN`。
- **L342 EN**: Blank line separating nearby declarations or logic blocks.
  **L342 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L343 EN**: Separator comment used for visual grouping.
  **L343 CN**: 用于视觉分组的分隔注释。
- **L344 EN**: Comment explains nearby logic, constraints, or intent: `E.3.4 Comparison`.
  **L344 CN**: 注释解释附近代码的逻辑、约束或设计意图：`E.3.4 Comparison`。
- **L345 EN**: Declares TableGen def record `VCEQ`.
  **L345 CN**: 声明 TableGen def 记录 `VCEQ`。
- **L346 EN**: Declares TableGen def record `VCGE`.
  **L346 CN**: 声明 TableGen def 记录 `VCGE`。
- **L347 EN**: Assigns a TableGen property that affects following records or inherited fields: `let InstName = "vcge" in`.
  **L347 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let InstName = "vcge" in`。
- **L348 EN**: Declares TableGen def record `VCLE`.
  **L348 CN**: 声明 TableGen def 记录 `VCLE`。
- **L349 EN**: Declares TableGen def record `VCGT`.
  **L349 CN**: 声明 TableGen def 记录 `VCGT`。
- **L350 EN**: Assigns a TableGen property that affects following records or inherited fields: `let InstName = "vcgt" in`.
  **L350 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let InstName = "vcgt" in`。
- **L351 EN**: Declares TableGen def record `VCLT`.
  **L351 CN**: 声明 TableGen def 记录 `VCLT`。
- **L352 EN**: Assigns a TableGen property that affects following records or inherited fields: `let InstName = "vacge" in {`.
  **L352 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let InstName = "vacge" in {`。
- **L353 EN**: Declares TableGen def record `VCAGE`.
  **L353 CN**: 声明 TableGen def 记录 `VCAGE`。
- **L354 EN**: Declares TableGen def record `VCALE`.
  **L354 CN**: 声明 TableGen def 记录 `VCALE`。
- **L355 EN**: Closes the current lexical scope or compound statement.
  **L355 CN**: 结束当前词法作用域或复合语句块。
- **L356 EN**: Assigns a TableGen property that affects following records or inherited fields: `let InstName = "vacgt" in {`.
  **L356 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let InstName = "vacgt" in {`。
- **L357 EN**: Declares TableGen def record `VCAGT`.
  **L357 CN**: 声明 TableGen def 记录 `VCAGT`。
- **L358 EN**: Declares TableGen def record `VCALT`.
  **L358 CN**: 声明 TableGen def 记录 `VCALT`。
- **L359 EN**: Closes the current lexical scope or compound statement.
  **L359 CN**: 结束当前词法作用域或复合语句块。
- **L360 EN**: Declares TableGen def record `VTST`.
  **L360 CN**: 声明 TableGen def 记录 `VTST`。
- **L361 EN**: Blank line separating nearby declarations or logic blocks.
  **L361 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L362 EN**: Separator comment used for visual grouping.
  **L362 CN**: 用于视觉分组的分隔注释。
- **L363 EN**: Comment explains nearby logic, constraints, or intent: `E.3.5 Absolute Difference`.
  **L363 CN**: 注释解释附近代码的逻辑、约束或设计意图：`E.3.5 Absolute Difference`。
- **L364 EN**: Declares TableGen def record `VABD`.
  **L364 CN**: 声明 TableGen def 记录 `VABD`。

### Lines 365-392

````tablegen
def VABDL : SOpInst<"vabdl", "(>Q)..",  "csiUcUsUi", OP_ABDL>;
def VABA  : SOpInst<"vaba", "....", "csiUcUsUiQcQsQiQUcQUsQUi", OP_ABA>;
def VABAL : SOpInst<"vabal", "(>Q)(>Q)..", "csiUcUsUi", OP_ABAL>;

////////////////////////////////////////////////////////////////////////////////
// E.3.6 Max/Min
def VMAX : SInst<"vmax", "...", "csiUcUsUifQcQsQiQUcQUsQUiQf">;
def VMIN : SInst<"vmin", "...", "csiUcUsUifQcQsQiQUcQUsQUiQf">;

////////////////////////////////////////////////////////////////////////////////
// E.3.7 Pairwise Addition
def VPADD  : IInst<"vpadd", "...", "csiUcUsUif">;
def VPADDL : SInst<"vpaddl", ">.",  "csiUcUsUiQcQsQiQUcQUsQUi">;
def VPADAL : SInst<"vpadal", ">>.", "csiUcUsUiQcQsQiQUcQUsQUi">;

////////////////////////////////////////////////////////////////////////////////
// E.3.8-9 Folding Max/Min
def VPMAX : SInst<"vpmax", "...", "csiUcUsUif">;
def VPMIN : SInst<"vpmin", "...", "csiUcUsUif">;

////////////////////////////////////////////////////////////////////////////////
// E.3.10 Reciprocal/Sqrt
def VRECPS  : IInst<"vrecps", "...", "fQf">;
def VRSQRTS : IInst<"vrsqrts", "...", "fQf">;

////////////////////////////////////////////////////////////////////////////////
// E.3.11 Shifts by signed variable
def VSHL   : SInst<"vshl", "..S", "csilUcUsUiUlQcQsQiQlQUcQUsQUiQUl">;
````
- **L365 EN**: Declares TableGen def record `VABDL`.
  **L365 CN**: 声明 TableGen def 记录 `VABDL`。
- **L366 EN**: Declares TableGen def record `VABA`.
  **L366 CN**: 声明 TableGen def 记录 `VABA`。
- **L367 EN**: Declares TableGen def record `VABAL`.
  **L367 CN**: 声明 TableGen def 记录 `VABAL`。
- **L368 EN**: Blank line separating nearby declarations or logic blocks.
  **L368 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L369 EN**: Separator comment used for visual grouping.
  **L369 CN**: 用于视觉分组的分隔注释。
- **L370 EN**: Comment explains nearby logic, constraints, or intent: `E.3.6 Max/Min`.
  **L370 CN**: 注释解释附近代码的逻辑、约束或设计意图：`E.3.6 Max/Min`。
- **L371 EN**: Declares TableGen def record `VMAX`.
  **L371 CN**: 声明 TableGen def 记录 `VMAX`。
- **L372 EN**: Declares TableGen def record `VMIN`.
  **L372 CN**: 声明 TableGen def 记录 `VMIN`。
- **L373 EN**: Blank line separating nearby declarations or logic blocks.
  **L373 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L374 EN**: Separator comment used for visual grouping.
  **L374 CN**: 用于视觉分组的分隔注释。
- **L375 EN**: Comment explains nearby logic, constraints, or intent: `E.3.7 Pairwise Addition`.
  **L375 CN**: 注释解释附近代码的逻辑、约束或设计意图：`E.3.7 Pairwise Addition`。
- **L376 EN**: Declares TableGen def record `VPADD`.
  **L376 CN**: 声明 TableGen def 记录 `VPADD`。
- **L377 EN**: Declares TableGen def record `VPADDL`.
  **L377 CN**: 声明 TableGen def 记录 `VPADDL`。
- **L378 EN**: Declares TableGen def record `VPADAL`.
  **L378 CN**: 声明 TableGen def 记录 `VPADAL`。
- **L379 EN**: Blank line separating nearby declarations or logic blocks.
  **L379 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L380 EN**: Separator comment used for visual grouping.
  **L380 CN**: 用于视觉分组的分隔注释。
- **L381 EN**: Comment explains nearby logic, constraints, or intent: `E.3.8-9 Folding Max/Min`.
  **L381 CN**: 注释解释附近代码的逻辑、约束或设计意图：`E.3.8-9 Folding Max/Min`。
- **L382 EN**: Declares TableGen def record `VPMAX`.
  **L382 CN**: 声明 TableGen def 记录 `VPMAX`。
- **L383 EN**: Declares TableGen def record `VPMIN`.
  **L383 CN**: 声明 TableGen def 记录 `VPMIN`。
- **L384 EN**: Blank line separating nearby declarations or logic blocks.
  **L384 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L385 EN**: Separator comment used for visual grouping.
  **L385 CN**: 用于视觉分组的分隔注释。
- **L386 EN**: Comment explains nearby logic, constraints, or intent: `E.3.10 Reciprocal/Sqrt`.
  **L386 CN**: 注释解释附近代码的逻辑、约束或设计意图：`E.3.10 Reciprocal/Sqrt`。
- **L387 EN**: Declares TableGen def record `VRECPS`.
  **L387 CN**: 声明 TableGen def 记录 `VRECPS`。
- **L388 EN**: Declares TableGen def record `VRSQRTS`.
  **L388 CN**: 声明 TableGen def 记录 `VRSQRTS`。
- **L389 EN**: Blank line separating nearby declarations or logic blocks.
  **L389 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L390 EN**: Separator comment used for visual grouping.
  **L390 CN**: 用于视觉分组的分隔注释。
- **L391 EN**: Comment explains nearby logic, constraints, or intent: `E.3.11 Shifts by signed variable`.
  **L391 CN**: 注释解释附近代码的逻辑、约束或设计意图：`E.3.11 Shifts by signed variable`。
- **L392 EN**: Declares TableGen def record `VSHL`.
  **L392 CN**: 声明 TableGen def 记录 `VSHL`。

### Lines 393-420

````tablegen
def VQSHL  : SInst<"vqshl", "..S", "csilUcUsUiUlQcQsQiQlQUcQUsQUiQUl">;
def VRSHL  : SInst<"vrshl", "..S", "csilUcUsUiUlQcQsQiQlQUcQUsQUiQUl">;
def VQRSHL : SInst<"vqrshl", "..S", "csilUcUsUiUlQcQsQiQlQUcQUsQUiQUl">;

////////////////////////////////////////////////////////////////////////////////
// E.3.12 Shifts by constant
let isShift = 1 in {


def VSHR_N     : SInst<"vshr_n", "..I", "csilUcUsUiUlQcQsQiQlQUcQUsQUiQUl",
                      [ImmCheck<1, ImmCheckShiftRight>]>;
def VSHL_N     : IInst<"vshl_n", "..I", "csilUcUsUiUlQcQsQiQlQUcQUsQUiQUl",
                      [ImmCheck<1, ImmCheckShiftLeft>]>;
def VRSHR_N    : SInst<"vrshr_n", "..I", "csilUcUsUiUlQcQsQiQlQUcQUsQUiQUl",
                      [ImmCheck<1, ImmCheckShiftRight>]>;
def VSRA_N     : SInst<"vsra_n", "...I", "csilUcUsUiUlQcQsQiQlQUcQUsQUiQUl",
                      [ImmCheck<2, ImmCheckShiftRight>]>;
def VRSRA_N    : SInst<"vrsra_n", "...I", "csilUcUsUiUlQcQsQiQlQUcQUsQUiQUl",
                      [ImmCheck<2, ImmCheckShiftRight>]>;
def VQSHL_N    : SInst<"vqshl_n", "..I", "csilUcUsUiUlQcQsQiQlQUcQUsQUiQUl",
                      [ImmCheck<1, ImmCheckShiftLeft>]>;
def VQSHLU_N   : SInst<"vqshlu_n", "U.I", "csilQcQsQiQl",
                      [ImmCheck<1, ImmCheckShiftLeft>]>;

// Narrowing right shifts should have an immediate range of 1..(sizeinbits(arg)/2).
// However, as the overloaded type code that is supplied to a polymorphic builtin
// is that of the return type (half as wide as the argument in this case), using
// ImmCheckShiftRightNarrow would return in an upper bound of (sizeinbits(arg)/2)/2.
````
- **L393 EN**: Declares TableGen def record `VQSHL`.
  **L393 CN**: 声明 TableGen def 记录 `VQSHL`。
- **L394 EN**: Declares TableGen def record `VRSHL`.
  **L394 CN**: 声明 TableGen def 记录 `VRSHL`。
- **L395 EN**: Declares TableGen def record `VQRSHL`.
  **L395 CN**: 声明 TableGen def 记录 `VQRSHL`。
- **L396 EN**: Blank line separating nearby declarations or logic blocks.
  **L396 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L397 EN**: Separator comment used for visual grouping.
  **L397 CN**: 用于视觉分组的分隔注释。
- **L398 EN**: Comment explains nearby logic, constraints, or intent: `E.3.12 Shifts by constant`.
  **L398 CN**: 注释解释附近代码的逻辑、约束或设计意图：`E.3.12 Shifts by constant`。
- **L399 EN**: Assigns a TableGen property that affects following records or inherited fields: `let isShift = 1 in {`.
  **L399 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let isShift = 1 in {`。
- **L400 EN**: Blank line separating nearby declarations or logic blocks.
  **L400 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L401 EN**: Blank line separating nearby declarations or logic blocks.
  **L401 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L402 EN**: Declares TableGen def record `VSHR_N`.
  **L402 CN**: 声明 TableGen def 记录 `VSHR_N`。
- **L403 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheckShiftRight>]>;`.
  **L403 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheckShiftRight>]>;`。
- **L404 EN**: Declares TableGen def record `VSHL_N`.
  **L404 CN**: 声明 TableGen def 记录 `VSHL_N`。
- **L405 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheckShiftLeft>]>;`.
  **L405 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheckShiftLeft>]>;`。
- **L406 EN**: Declares TableGen def record `VRSHR_N`.
  **L406 CN**: 声明 TableGen def 记录 `VRSHR_N`。
- **L407 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheckShiftRight>]>;`.
  **L407 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheckShiftRight>]>;`。
- **L408 EN**: Declares TableGen def record `VSRA_N`.
  **L408 CN**: 声明 TableGen def 记录 `VSRA_N`。
- **L409 EN**: Adds a standalone statement or declaration: `[ImmCheck<2, ImmCheckShiftRight>]>;`.
  **L409 CN**: 添加一条独立语句或声明：`[ImmCheck<2, ImmCheckShiftRight>]>;`。
- **L410 EN**: Declares TableGen def record `VRSRA_N`.
  **L410 CN**: 声明 TableGen def 记录 `VRSRA_N`。
- **L411 EN**: Adds a standalone statement or declaration: `[ImmCheck<2, ImmCheckShiftRight>]>;`.
  **L411 CN**: 添加一条独立语句或声明：`[ImmCheck<2, ImmCheckShiftRight>]>;`。
- **L412 EN**: Declares TableGen def record `VQSHL_N`.
  **L412 CN**: 声明 TableGen def 记录 `VQSHL_N`。
- **L413 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheckShiftLeft>]>;`.
  **L413 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheckShiftLeft>]>;`。
- **L414 EN**: Declares TableGen def record `VQSHLU_N`.
  **L414 CN**: 声明 TableGen def 记录 `VQSHLU_N`。
- **L415 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheckShiftLeft>]>;`.
  **L415 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheckShiftLeft>]>;`。
- **L416 EN**: Blank line separating nearby declarations or logic blocks.
  **L416 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L417 EN**: Comment explains nearby logic, constraints, or intent: `Narrowing right shifts should have an immediate range of 1..(sizeinbits(arg)/2).`.
  **L417 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Narrowing right shifts should have an immediate range of 1..(sizeinbits(arg)/2).`。
- **L418 EN**: Comment explains nearby logic, constraints, or intent: `However, as the overloaded type code that is supplied to a polymorphic builtin`.
  **L418 CN**: 注释解释附近代码的逻辑、约束或设计意图：`However, as the overloaded type code that is supplied to a polymorphic builtin`。
- **L419 EN**: Comment explains nearby logic, constraints, or intent: `is that of the return type (half as wide as the argument in this case), using`.
  **L419 CN**: 注释解释附近代码的逻辑、约束或设计意图：`is that of the return type (half as wide as the argument in this case), using`。
- **L420 EN**: Comment explains nearby logic, constraints, or intent: `ImmCheckShiftRightNarrow would return in an upper bound of (sizeinbits(arg)/2)/2.`.
  **L420 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ImmCheckShiftRightNarrow would return in an upper bound of (sizeinbits(arg)/2)/2.`。

### Lines 421-448

````tablegen
// ImmCheckShiftRight produces the correct behavior here.
def VSHRN_N    : IInst<"vshrn_n", "<QI", "silUsUiUl",
                      [ImmCheck<1, ImmCheckShiftRight>]>;
def VQSHRUN_N  : SInst<"vqshrun_n", "(<U)QI", "sil",
                      [ImmCheck<1, ImmCheckShiftRight>]>;
def VQRSHRUN_N : SInst<"vqrshrun_n", "(<U)QI", "sil",
                      [ImmCheck<1, ImmCheckShiftRight>]>;
def VQSHRN_N   : SInst<"vqshrn_n", "<QI", "silUsUiUl",
                      [ImmCheck<1, ImmCheckShiftRight>]>;
def VRSHRN_N   : IInst<"vrshrn_n", "<QI", "silUsUiUl",
                      [ImmCheck<1, ImmCheckShiftRight>]>;
def VQRSHRN_N  : SInst<"vqrshrn_n", "<QI", "silUsUiUl",
                      [ImmCheck<1, ImmCheckShiftRight>]>;

// Widening left-shifts should have a range of 0..(sizeinbits(arg)-1).
// This polymorphic builtin is supplied the wider return type as it's overloaded
// base type, so the range here is actually 0..(sizeinbits(arg)*2).
// This cannot be rectified currently due to a use of vshll_n_s16 with an
// out-of-bounds immediate in the defintiion of vcvt_f32_bf16.
def VSHLL_N    : SInst<"vshll_n", "(>Q).I", "csiUcUsUi",
                      [ImmCheck<1, ImmCheckShiftLeft>]>;

////////////////////////////////////////////////////////////////////////////////
// E.3.13 Shifts with insert
def VSRI_N : WInst<"vsri_n", "...I",
                   "csilUcUsUiUlPcPsQcQsQiQlQUcQUsQUiQUlQPcQPs",
                    [ImmCheck<2, ImmCheckShiftRight, 0>]>;
def VSLI_N : WInst<"vsli_n", "...I",
````
- **L421 EN**: Comment explains nearby logic, constraints, or intent: `ImmCheckShiftRight produces the correct behavior here.`.
  **L421 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ImmCheckShiftRight produces the correct behavior here.`。
- **L422 EN**: Declares TableGen def record `VSHRN_N`.
  **L422 CN**: 声明 TableGen def 记录 `VSHRN_N`。
- **L423 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheckShiftRight>]>;`.
  **L423 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheckShiftRight>]>;`。
- **L424 EN**: Declares TableGen def record `VQSHRUN_N`.
  **L424 CN**: 声明 TableGen def 记录 `VQSHRUN_N`。
- **L425 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheckShiftRight>]>;`.
  **L425 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheckShiftRight>]>;`。
- **L426 EN**: Declares TableGen def record `VQRSHRUN_N`.
  **L426 CN**: 声明 TableGen def 记录 `VQRSHRUN_N`。
- **L427 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheckShiftRight>]>;`.
  **L427 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheckShiftRight>]>;`。
- **L428 EN**: Declares TableGen def record `VQSHRN_N`.
  **L428 CN**: 声明 TableGen def 记录 `VQSHRN_N`。
- **L429 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheckShiftRight>]>;`.
  **L429 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheckShiftRight>]>;`。
- **L430 EN**: Declares TableGen def record `VRSHRN_N`.
  **L430 CN**: 声明 TableGen def 记录 `VRSHRN_N`。
- **L431 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheckShiftRight>]>;`.
  **L431 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheckShiftRight>]>;`。
- **L432 EN**: Declares TableGen def record `VQRSHRN_N`.
  **L432 CN**: 声明 TableGen def 记录 `VQRSHRN_N`。
- **L433 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheckShiftRight>]>;`.
  **L433 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheckShiftRight>]>;`。
- **L434 EN**: Blank line separating nearby declarations or logic blocks.
  **L434 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L435 EN**: Comment explains nearby logic, constraints, or intent: `Widening left-shifts should have a range of 0..(sizeinbits(arg)-1).`.
  **L435 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Widening left-shifts should have a range of 0..(sizeinbits(arg)-1).`。
- **L436 EN**: Comment explains nearby logic, constraints, or intent: `This polymorphic builtin is supplied the wider return type as it's overloaded`.
  **L436 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This polymorphic builtin is supplied the wider return type as it's overloaded`。
- **L437 EN**: Comment explains nearby logic, constraints, or intent: `base type, so the range here is actually 0..(sizeinbits(arg)*2).`.
  **L437 CN**: 注释解释附近代码的逻辑、约束或设计意图：`base type, so the range here is actually 0..(sizeinbits(arg)*2).`。
- **L438 EN**: Comment explains nearby logic, constraints, or intent: `This cannot be rectified currently due to a use of vshll_n_s16 with an`.
  **L438 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This cannot be rectified currently due to a use of vshll_n_s16 with an`。
- **L439 EN**: Comment explains nearby logic, constraints, or intent: `out-of-bounds immediate in the defintiion of vcvt_f32_bf16.`.
  **L439 CN**: 注释解释附近代码的逻辑、约束或设计意图：`out-of-bounds immediate in the defintiion of vcvt_f32_bf16.`。
- **L440 EN**: Declares TableGen def record `VSHLL_N`.
  **L440 CN**: 声明 TableGen def 记录 `VSHLL_N`。
- **L441 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheckShiftLeft>]>;`.
  **L441 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheckShiftLeft>]>;`。
- **L442 EN**: Blank line separating nearby declarations or logic blocks.
  **L442 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L443 EN**: Separator comment used for visual grouping.
  **L443 CN**: 用于视觉分组的分隔注释。
- **L444 EN**: Comment explains nearby logic, constraints, or intent: `E.3.13 Shifts with insert`.
  **L444 CN**: 注释解释附近代码的逻辑、约束或设计意图：`E.3.13 Shifts with insert`。
- **L445 EN**: Declares TableGen def record `VSRI_N`.
  **L445 CN**: 声明 TableGen def 记录 `VSRI_N`。
- **L446 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"csilUcUsUiUlPcPsQcQsQiQlQUcQUsQUiQUlQPcQPs",`.
  **L446 CN**: 继续一个多行参数列表、初始化器或聚合项：`"csilUcUsUiUlPcPsQcQsQiQlQUcQUsQUiQUlQPcQPs",`。
- **L447 EN**: Adds a standalone statement or declaration: `[ImmCheck<2, ImmCheckShiftRight, 0>]>;`.
  **L447 CN**: 添加一条独立语句或声明：`[ImmCheck<2, ImmCheckShiftRight, 0>]>;`。
- **L448 EN**: Declares TableGen def record `VSLI_N`.
  **L448 CN**: 声明 TableGen def 记录 `VSLI_N`。

### Lines 449-476

````tablegen
                   "csilUcUsUiUlPcPsQcQsQiQlQUcQUsQUiQUlQPcQPs",
                   [ImmCheck<2, ImmCheckShiftLeft, 0>]>;
}

////////////////////////////////////////////////////////////////////////////////
// E.3.14 Loads and stores of a single vector
def VLD1      : WInst<"vld1", ".(c*!)",
                      "QUcQUsQUiQUlQcQsQiQlQfQPcQPsUcUsUiUlcsilfPcPs">;
def VLD1_X2   : WInst<"vld1_x2", "2(c*!)",
                      "cfilsUcUiUlUsQcQfQiQlQsQUcQUiQUlQUsPcPsQPcQPs">;
def VLD1_X3   : WInst<"vld1_x3", "3(c*!)",
                      "cfilsUcUiUlUsQcQfQiQlQsQUcQUiQUlQUsPcPsQPcQPs">;
def VLD1_X4   : WInst<"vld1_x4", "4(c*!)",
                      "cfilsUcUiUlUsQcQfQiQlQsQUcQUiQUlQUsPcPsQPcQPs">;
def VLD1_LANE : WInst<"vld1_lane", ".(c*!).I",
                      "QUcQUsQUiQUlQcQsQiQlQfQPcQPsUcUsUiUlcsilfPcPs",
                      [ImmCheck<2, ImmCheckLaneIndex, 1>]>;
def VLD1_DUP  : WInst<"vld1_dup", ".(c*!)",
                      "QUcQUsQUiQUlQcQsQiQlQfQPcQPsUcUsUiUlcsilfPcPs">;
def VST1      : WInst<"vst1", "v*(.!)",
                      "QUcQUsQUiQUlQcQsQiQlQfQPcQPsUcUsUiUlcsilfPcPs">;
def VST1_X2   : WInst<"vst1_x2", "v*(2!)",
                      "cfilsUcUiUlUsQcQfQiQlQsQUcQUiQUlQUsPcPsQPcQPs">;
def VST1_X3   : WInst<"vst1_x3", "v*(3!)",
                      "cfilsUcUiUlUsQcQfQiQlQsQUcQUiQUlQUsPcPsQPcQPs">;
def VST1_X4   : WInst<"vst1_x4", "v*(4!)",
                      "cfilsUcUiUlUsQcQfQiQlQsQUcQUiQUlQUsPcPsQPcQPs">;
def VST1_LANE : WInst<"vst1_lane", "v*(.!)I",
````
- **L449 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"csilUcUsUiUlPcPsQcQsQiQlQUcQUsQUiQUlQPcQPs",`.
  **L449 CN**: 继续一个多行参数列表、初始化器或聚合项：`"csilUcUsUiUlPcPsQcQsQiQlQUcQUsQUiQUlQPcQPs",`。
- **L450 EN**: Adds a standalone statement or declaration: `[ImmCheck<2, ImmCheckShiftLeft, 0>]>;`.
  **L450 CN**: 添加一条独立语句或声明：`[ImmCheck<2, ImmCheckShiftLeft, 0>]>;`。
- **L451 EN**: Closes the current lexical scope or compound statement.
  **L451 CN**: 结束当前词法作用域或复合语句块。
- **L452 EN**: Blank line separating nearby declarations or logic blocks.
  **L452 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L453 EN**: Separator comment used for visual grouping.
  **L453 CN**: 用于视觉分组的分隔注释。
- **L454 EN**: Comment explains nearby logic, constraints, or intent: `E.3.14 Loads and stores of a single vector`.
  **L454 CN**: 注释解释附近代码的逻辑、约束或设计意图：`E.3.14 Loads and stores of a single vector`。
- **L455 EN**: Declares TableGen def record `VLD1`.
  **L455 CN**: 声明 TableGen def 记录 `VLD1`。
- **L456 EN**: Adds a standalone statement or declaration: `"QUcQUsQUiQUlQcQsQiQlQfQPcQPsUcUsUiUlcsilfPcPs">;`.
  **L456 CN**: 添加一条独立语句或声明：`"QUcQUsQUiQUlQcQsQiQlQfQPcQPsUcUsUiUlcsilfPcPs">;`。
- **L457 EN**: Declares TableGen def record `VLD1_X2`.
  **L457 CN**: 声明 TableGen def 记录 `VLD1_X2`。
- **L458 EN**: Adds a standalone statement or declaration: `"cfilsUcUiUlUsQcQfQiQlQsQUcQUiQUlQUsPcPsQPcQPs">;`.
  **L458 CN**: 添加一条独立语句或声明：`"cfilsUcUiUlUsQcQfQiQlQsQUcQUiQUlQUsPcPsQPcQPs">;`。
- **L459 EN**: Declares TableGen def record `VLD1_X3`.
  **L459 CN**: 声明 TableGen def 记录 `VLD1_X3`。
- **L460 EN**: Adds a standalone statement or declaration: `"cfilsUcUiUlUsQcQfQiQlQsQUcQUiQUlQUsPcPsQPcQPs">;`.
  **L460 CN**: 添加一条独立语句或声明：`"cfilsUcUiUlUsQcQfQiQlQsQUcQUiQUlQUsPcPsQPcQPs">;`。
- **L461 EN**: Declares TableGen def record `VLD1_X4`.
  **L461 CN**: 声明 TableGen def 记录 `VLD1_X4`。
- **L462 EN**: Adds a standalone statement or declaration: `"cfilsUcUiUlUsQcQfQiQlQsQUcQUiQUlQUsPcPsQPcQPs">;`.
  **L462 CN**: 添加一条独立语句或声明：`"cfilsUcUiUlUsQcQfQiQlQsQUcQUiQUlQUsPcPsQPcQPs">;`。
- **L463 EN**: Declares TableGen def record `VLD1_LANE`.
  **L463 CN**: 声明 TableGen def 记录 `VLD1_LANE`。
- **L464 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"QUcQUsQUiQUlQcQsQiQlQfQPcQPsUcUsUiUlcsilfPcPs",`.
  **L464 CN**: 继续一个多行参数列表、初始化器或聚合项：`"QUcQUsQUiQUlQcQsQiQlQfQPcQPsUcUsUiUlcsilfPcPs",`。
- **L465 EN**: Adds a standalone statement or declaration: `[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`.
  **L465 CN**: 添加一条独立语句或声明：`[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`。
- **L466 EN**: Declares TableGen def record `VLD1_DUP`.
  **L466 CN**: 声明 TableGen def 记录 `VLD1_DUP`。
- **L467 EN**: Adds a standalone statement or declaration: `"QUcQUsQUiQUlQcQsQiQlQfQPcQPsUcUsUiUlcsilfPcPs">;`.
  **L467 CN**: 添加一条独立语句或声明：`"QUcQUsQUiQUlQcQsQiQlQfQPcQPsUcUsUiUlcsilfPcPs">;`。
- **L468 EN**: Declares TableGen def record `VST1`.
  **L468 CN**: 声明 TableGen def 记录 `VST1`。
- **L469 EN**: Adds a standalone statement or declaration: `"QUcQUsQUiQUlQcQsQiQlQfQPcQPsUcUsUiUlcsilfPcPs">;`.
  **L469 CN**: 添加一条独立语句或声明：`"QUcQUsQUiQUlQcQsQiQlQfQPcQPsUcUsUiUlcsilfPcPs">;`。
- **L470 EN**: Declares TableGen def record `VST1_X2`.
  **L470 CN**: 声明 TableGen def 记录 `VST1_X2`。
- **L471 EN**: Adds a standalone statement or declaration: `"cfilsUcUiUlUsQcQfQiQlQsQUcQUiQUlQUsPcPsQPcQPs">;`.
  **L471 CN**: 添加一条独立语句或声明：`"cfilsUcUiUlUsQcQfQiQlQsQUcQUiQUlQUsPcPsQPcQPs">;`。
- **L472 EN**: Declares TableGen def record `VST1_X3`.
  **L472 CN**: 声明 TableGen def 记录 `VST1_X3`。
- **L473 EN**: Adds a standalone statement or declaration: `"cfilsUcUiUlUsQcQfQiQlQsQUcQUiQUlQUsPcPsQPcQPs">;`.
  **L473 CN**: 添加一条独立语句或声明：`"cfilsUcUiUlUsQcQfQiQlQsQUcQUiQUlQUsPcPsQPcQPs">;`。
- **L474 EN**: Declares TableGen def record `VST1_X4`.
  **L474 CN**: 声明 TableGen def 记录 `VST1_X4`。
- **L475 EN**: Adds a standalone statement or declaration: `"cfilsUcUiUlUsQcQfQiQlQsQUcQUiQUlQUsPcPsQPcQPs">;`.
  **L475 CN**: 添加一条独立语句或声明：`"cfilsUcUiUlUsQcQfQiQlQsQUcQUiQUlQUsPcPsQPcQPs">;`。
- **L476 EN**: Declares TableGen def record `VST1_LANE`.
  **L476 CN**: 声明 TableGen def 记录 `VST1_LANE`。

### Lines 477-504

````tablegen
                      "QUcQUsQUiQUlQcQsQiQlQfQPcQPsUcUsUiUlcsilfPcPs",
                      [ImmCheck<2, ImmCheckLaneIndex, 1>]>;

let ArchGuard = "(__ARM_FP & 2)" in {
def VLD1_F16      : WInst<"vld1", ".(c*!)", "hQh">;
def VLD1_X2_F16   : WInst<"vld1_x2", "2(c*!)", "hQh">;
def VLD1_X3_F16   : WInst<"vld1_x3", "3(c*!)", "hQh">;
def VLD1_X4_F16   : WInst<"vld1_x4", "4(c*!)", "hQh">;
def VLD1_LANE_F16 : WInst<"vld1_lane", ".(c*!).I", "hQh",
                          [ImmCheck<2, ImmCheckLaneIndex, 1>]>;
def VLD1_DUP_F16  : WInst<"vld1_dup", ".(c*!)", "hQh">;
def VST1_F16      : WInst<"vst1", "v*(.!)", "hQh">;
def VST1_X2_F16   : WInst<"vst1_x2", "v*(2!)", "hQh">;
def VST1_X3_F16   : WInst<"vst1_x3", "v*(3!)", "hQh">;
def VST1_X4_F16   : WInst<"vst1_x4", "v*(4!)", "hQh">;
def VST1_LANE_F16 : WInst<"vst1_lane", "v*(.!)I", "hQh",
                          [ImmCheck<2, ImmCheckLaneIndex, 1>]>;
}

////////////////////////////////////////////////////////////////////////////////
// E.3.15 Loads and stores of an N-element structure
def VLD2 : WInst<"vld2", "2(c*!)", "QUcQUsQUiQcQsQiQfQPcQPsUcUsUiUlcsilfPcPs">;
def VLD3 : WInst<"vld3", "3(c*!)", "QUcQUsQUiQcQsQiQfQPcQPsUcUsUiUlcsilfPcPs">;
def VLD4 : WInst<"vld4", "4(c*!)", "QUcQUsQUiQcQsQiQfQPcQPsUcUsUiUlcsilfPcPs">;
def VLD2_DUP  : WInst<"vld2_dup", "2(c*!)",
                      "UcUsUiUlcsilfPcPsQcQfQiQlQsQPcQPsQUcQUiQUlQUs">;
def VLD3_DUP  : WInst<"vld3_dup", "3(c*!)",
                      "UcUsUiUlcsilfPcPsQcQfQiQlQsQPcQPsQUcQUiQUlQUs">;
````
- **L477 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"QUcQUsQUiQUlQcQsQiQlQfQPcQPsUcUsUiUlcsilfPcPs",`.
  **L477 CN**: 继续一个多行参数列表、初始化器或聚合项：`"QUcQUsQUiQUlQcQsQiQlQfQPcQPsUcUsUiUlcsilfPcPs",`。
- **L478 EN**: Adds a standalone statement or declaration: `[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`.
  **L478 CN**: 添加一条独立语句或声明：`[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`。
- **L479 EN**: Blank line separating nearby declarations or logic blocks.
  **L479 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L480 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "(__ARM_FP & 2)" in {`.
  **L480 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "(__ARM_FP & 2)" in {`。
- **L481 EN**: Declares TableGen def record `VLD1_F16`.
  **L481 CN**: 声明 TableGen def 记录 `VLD1_F16`。
- **L482 EN**: Declares TableGen def record `VLD1_X2_F16`.
  **L482 CN**: 声明 TableGen def 记录 `VLD1_X2_F16`。
- **L483 EN**: Declares TableGen def record `VLD1_X3_F16`.
  **L483 CN**: 声明 TableGen def 记录 `VLD1_X3_F16`。
- **L484 EN**: Declares TableGen def record `VLD1_X4_F16`.
  **L484 CN**: 声明 TableGen def 记录 `VLD1_X4_F16`。
- **L485 EN**: Declares TableGen def record `VLD1_LANE_F16`.
  **L485 CN**: 声明 TableGen def 记录 `VLD1_LANE_F16`。
- **L486 EN**: Adds a standalone statement or declaration: `[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`.
  **L486 CN**: 添加一条独立语句或声明：`[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`。
- **L487 EN**: Declares TableGen def record `VLD1_DUP_F16`.
  **L487 CN**: 声明 TableGen def 记录 `VLD1_DUP_F16`。
- **L488 EN**: Declares TableGen def record `VST1_F16`.
  **L488 CN**: 声明 TableGen def 记录 `VST1_F16`。
- **L489 EN**: Declares TableGen def record `VST1_X2_F16`.
  **L489 CN**: 声明 TableGen def 记录 `VST1_X2_F16`。
- **L490 EN**: Declares TableGen def record `VST1_X3_F16`.
  **L490 CN**: 声明 TableGen def 记录 `VST1_X3_F16`。
- **L491 EN**: Declares TableGen def record `VST1_X4_F16`.
  **L491 CN**: 声明 TableGen def 记录 `VST1_X4_F16`。
- **L492 EN**: Declares TableGen def record `VST1_LANE_F16`.
  **L492 CN**: 声明 TableGen def 记录 `VST1_LANE_F16`。
- **L493 EN**: Adds a standalone statement or declaration: `[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`.
  **L493 CN**: 添加一条独立语句或声明：`[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`。
- **L494 EN**: Closes the current lexical scope or compound statement.
  **L494 CN**: 结束当前词法作用域或复合语句块。
- **L495 EN**: Blank line separating nearby declarations or logic blocks.
  **L495 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L496 EN**: Separator comment used for visual grouping.
  **L496 CN**: 用于视觉分组的分隔注释。
- **L497 EN**: Comment explains nearby logic, constraints, or intent: `E.3.15 Loads and stores of an N-element structure`.
  **L497 CN**: 注释解释附近代码的逻辑、约束或设计意图：`E.3.15 Loads and stores of an N-element structure`。
- **L498 EN**: Declares TableGen def record `VLD2`.
  **L498 CN**: 声明 TableGen def 记录 `VLD2`。
- **L499 EN**: Declares TableGen def record `VLD3`.
  **L499 CN**: 声明 TableGen def 记录 `VLD3`。
- **L500 EN**: Declares TableGen def record `VLD4`.
  **L500 CN**: 声明 TableGen def 记录 `VLD4`。
- **L501 EN**: Declares TableGen def record `VLD2_DUP`.
  **L501 CN**: 声明 TableGen def 记录 `VLD2_DUP`。
- **L502 EN**: Adds a standalone statement or declaration: `"UcUsUiUlcsilfPcPsQcQfQiQlQsQPcQPsQUcQUiQUlQUs">;`.
  **L502 CN**: 添加一条独立语句或声明：`"UcUsUiUlcsilfPcPsQcQfQiQlQsQPcQPsQUcQUiQUlQUs">;`。
- **L503 EN**: Declares TableGen def record `VLD3_DUP`.
  **L503 CN**: 声明 TableGen def 记录 `VLD3_DUP`。
- **L504 EN**: Adds a standalone statement or declaration: `"UcUsUiUlcsilfPcPsQcQfQiQlQsQPcQPsQUcQUiQUlQUs">;`.
  **L504 CN**: 添加一条独立语句或声明：`"UcUsUiUlcsilfPcPsQcQfQiQlQsQPcQPsQUcQUiQUlQUs">;`。

### Lines 505-532

````tablegen
def VLD4_DUP  : WInst<"vld4_dup", "4(c*!)",
                      "UcUsUiUlcsilfPcPsQcQfQiQlQsQPcQPsQUcQUiQUlQUs">;
def VLD2_LANE : WInst<"vld2_lane", "2(c*!)2I", "QUsQUiQsQiQfQPsUcUsUicsifPcPs",
                      [ImmCheck<4, ImmCheckLaneIndex, 1>]>;
def VLD3_LANE : WInst<"vld3_lane", "3(c*!)3I", "QUsQUiQsQiQfQPsUcUsUicsifPcPs",
                      [ImmCheck<5, ImmCheckLaneIndex, 1>]>;
def VLD4_LANE : WInst<"vld4_lane", "4(c*!)4I", "QUsQUiQsQiQfQPsUcUsUicsifPcPs",
                      [ImmCheck<6, ImmCheckLaneIndex, 1>]>;
def VST2 : WInst<"vst2", "v*(2!)", "QUcQUsQUiQcQsQiQfQPcQPsUcUsUiUlcsilfPcPs">;
def VST3 : WInst<"vst3", "v*(3!)", "QUcQUsQUiQcQsQiQfQPcQPsUcUsUiUlcsilfPcPs">;
def VST4 : WInst<"vst4", "v*(4!)", "QUcQUsQUiQcQsQiQfQPcQPsUcUsUiUlcsilfPcPs">;
def VST2_LANE : WInst<"vst2_lane", "v*(2!)I", "QUsQUiQsQiQfQPsUcUsUicsifPcPs",
                      [ImmCheck<3, ImmCheckLaneIndex, 1>]>;
def VST3_LANE : WInst<"vst3_lane", "v*(3!)I", "QUsQUiQsQiQfQPsUcUsUicsifPcPs",
                      [ImmCheck<4, ImmCheckLaneIndex, 1>]>;
def VST4_LANE : WInst<"vst4_lane", "v*(4!)I", "QUsQUiQsQiQfQPsUcUsUicsifPcPs",
                      [ImmCheck<5, ImmCheckLaneIndex, 1>]>;
let ArchGuard = "(__ARM_FP & 2)" in {
def VLD2_F16      : WInst<"vld2", "2(c*!)", "hQh">;
def VLD3_F16      : WInst<"vld3", "3(c*!)", "hQh">;
def VLD4_F16      : WInst<"vld4", "4(c*!)", "hQh">;
def VLD2_DUP_F16  : WInst<"vld2_dup", "2(c*!)", "hQh">;
def VLD3_DUP_F16  : WInst<"vld3_dup", "3(c*!)", "hQh">;
def VLD4_DUP_F16  : WInst<"vld4_dup", "4(c*!)", "hQh">;
def VLD2_LANE_F16 : WInst<"vld2_lane", "2(c*!)2I", "hQh",
                          [ImmCheck<4, ImmCheckLaneIndex, 1>]>;
def VLD3_LANE_F16 : WInst<"vld3_lane", "3(c*!)3I", "hQh",
                          [ImmCheck<5, ImmCheckLaneIndex, 1>]>;
````
- **L505 EN**: Declares TableGen def record `VLD4_DUP`.
  **L505 CN**: 声明 TableGen def 记录 `VLD4_DUP`。
- **L506 EN**: Adds a standalone statement or declaration: `"UcUsUiUlcsilfPcPsQcQfQiQlQsQPcQPsQUcQUiQUlQUs">;`.
  **L506 CN**: 添加一条独立语句或声明：`"UcUsUiUlcsilfPcPsQcQfQiQlQsQPcQPsQUcQUiQUlQUs">;`。
- **L507 EN**: Declares TableGen def record `VLD2_LANE`.
  **L507 CN**: 声明 TableGen def 记录 `VLD2_LANE`。
- **L508 EN**: Adds a standalone statement or declaration: `[ImmCheck<4, ImmCheckLaneIndex, 1>]>;`.
  **L508 CN**: 添加一条独立语句或声明：`[ImmCheck<4, ImmCheckLaneIndex, 1>]>;`。
- **L509 EN**: Declares TableGen def record `VLD3_LANE`.
  **L509 CN**: 声明 TableGen def 记录 `VLD3_LANE`。
- **L510 EN**: Adds a standalone statement or declaration: `[ImmCheck<5, ImmCheckLaneIndex, 1>]>;`.
  **L510 CN**: 添加一条独立语句或声明：`[ImmCheck<5, ImmCheckLaneIndex, 1>]>;`。
- **L511 EN**: Declares TableGen def record `VLD4_LANE`.
  **L511 CN**: 声明 TableGen def 记录 `VLD4_LANE`。
- **L512 EN**: Adds a standalone statement or declaration: `[ImmCheck<6, ImmCheckLaneIndex, 1>]>;`.
  **L512 CN**: 添加一条独立语句或声明：`[ImmCheck<6, ImmCheckLaneIndex, 1>]>;`。
- **L513 EN**: Declares TableGen def record `VST2`.
  **L513 CN**: 声明 TableGen def 记录 `VST2`。
- **L514 EN**: Declares TableGen def record `VST3`.
  **L514 CN**: 声明 TableGen def 记录 `VST3`。
- **L515 EN**: Declares TableGen def record `VST4`.
  **L515 CN**: 声明 TableGen def 记录 `VST4`。
- **L516 EN**: Declares TableGen def record `VST2_LANE`.
  **L516 CN**: 声明 TableGen def 记录 `VST2_LANE`。
- **L517 EN**: Adds a standalone statement or declaration: `[ImmCheck<3, ImmCheckLaneIndex, 1>]>;`.
  **L517 CN**: 添加一条独立语句或声明：`[ImmCheck<3, ImmCheckLaneIndex, 1>]>;`。
- **L518 EN**: Declares TableGen def record `VST3_LANE`.
  **L518 CN**: 声明 TableGen def 记录 `VST3_LANE`。
- **L519 EN**: Adds a standalone statement or declaration: `[ImmCheck<4, ImmCheckLaneIndex, 1>]>;`.
  **L519 CN**: 添加一条独立语句或声明：`[ImmCheck<4, ImmCheckLaneIndex, 1>]>;`。
- **L520 EN**: Declares TableGen def record `VST4_LANE`.
  **L520 CN**: 声明 TableGen def 记录 `VST4_LANE`。
- **L521 EN**: Adds a standalone statement or declaration: `[ImmCheck<5, ImmCheckLaneIndex, 1>]>;`.
  **L521 CN**: 添加一条独立语句或声明：`[ImmCheck<5, ImmCheckLaneIndex, 1>]>;`。
- **L522 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "(__ARM_FP & 2)" in {`.
  **L522 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "(__ARM_FP & 2)" in {`。
- **L523 EN**: Declares TableGen def record `VLD2_F16`.
  **L523 CN**: 声明 TableGen def 记录 `VLD2_F16`。
- **L524 EN**: Declares TableGen def record `VLD3_F16`.
  **L524 CN**: 声明 TableGen def 记录 `VLD3_F16`。
- **L525 EN**: Declares TableGen def record `VLD4_F16`.
  **L525 CN**: 声明 TableGen def 记录 `VLD4_F16`。
- **L526 EN**: Declares TableGen def record `VLD2_DUP_F16`.
  **L526 CN**: 声明 TableGen def 记录 `VLD2_DUP_F16`。
- **L527 EN**: Declares TableGen def record `VLD3_DUP_F16`.
  **L527 CN**: 声明 TableGen def 记录 `VLD3_DUP_F16`。
- **L528 EN**: Declares TableGen def record `VLD4_DUP_F16`.
  **L528 CN**: 声明 TableGen def 记录 `VLD4_DUP_F16`。
- **L529 EN**: Declares TableGen def record `VLD2_LANE_F16`.
  **L529 CN**: 声明 TableGen def 记录 `VLD2_LANE_F16`。
- **L530 EN**: Adds a standalone statement or declaration: `[ImmCheck<4, ImmCheckLaneIndex, 1>]>;`.
  **L530 CN**: 添加一条独立语句或声明：`[ImmCheck<4, ImmCheckLaneIndex, 1>]>;`。
- **L531 EN**: Declares TableGen def record `VLD3_LANE_F16`.
  **L531 CN**: 声明 TableGen def 记录 `VLD3_LANE_F16`。
- **L532 EN**: Adds a standalone statement or declaration: `[ImmCheck<5, ImmCheckLaneIndex, 1>]>;`.
  **L532 CN**: 添加一条独立语句或声明：`[ImmCheck<5, ImmCheckLaneIndex, 1>]>;`。

### Lines 533-560

````tablegen
def VLD4_LANE_F16 : WInst<"vld4_lane", "4(c*!)4I", "hQh",
                          [ImmCheck<6, ImmCheckLaneIndex, 1>]>;
def VST2_F16      : WInst<"vst2", "v*(2!)", "hQh">;
def VST3_F16      : WInst<"vst3", "v*(3!)", "hQh">;
def VST4_F16      : WInst<"vst4", "v*(4!)", "hQh">;
def VST2_LANE_F16 : WInst<"vst2_lane", "v*(2!)I", "hQh",
                          [ImmCheck<3, ImmCheckLaneIndex, 1>]>;
def VST3_LANE_F16 : WInst<"vst3_lane", "v*(3!)I", "hQh",
                         [ImmCheck<4, ImmCheckLaneIndex, 1>]>;
def VST4_LANE_F16 : WInst<"vst4_lane", "v*(4!)I", "hQh",
                          [ImmCheck<5, ImmCheckLaneIndex, 1>]>;
}

////////////////////////////////////////////////////////////////////////////////
// E.3.16 Extract lanes from a vector
let InstName = "vmov" in
def VGET_LANE : IInst<"vget_lane", "1.I",
                      "UcUsUicsiPcPsfQUcQUsQUiQcQsQiQPcQPsQflUlQlQUlmQm",
                      [ImmCheck<1, ImmCheckLaneIndex, 0>]>;

////////////////////////////////////////////////////////////////////////////////
// E.3.17 Set lanes within a vector
let InstName = "vmov" in
def VSET_LANE : IInst<"vset_lane", ".1.I",
                      "UcUsUicsiPcPsfQUcQUsQUiQcQsQiQPcQPsQflUlQlQUlmQm",
                      [ImmCheck<2, ImmCheckLaneIndex, 1>]>;

////////////////////////////////////////////////////////////////////////////////
````
- **L533 EN**: Declares TableGen def record `VLD4_LANE_F16`.
  **L533 CN**: 声明 TableGen def 记录 `VLD4_LANE_F16`。
- **L534 EN**: Adds a standalone statement or declaration: `[ImmCheck<6, ImmCheckLaneIndex, 1>]>;`.
  **L534 CN**: 添加一条独立语句或声明：`[ImmCheck<6, ImmCheckLaneIndex, 1>]>;`。
- **L535 EN**: Declares TableGen def record `VST2_F16`.
  **L535 CN**: 声明 TableGen def 记录 `VST2_F16`。
- **L536 EN**: Declares TableGen def record `VST3_F16`.
  **L536 CN**: 声明 TableGen def 记录 `VST3_F16`。
- **L537 EN**: Declares TableGen def record `VST4_F16`.
  **L537 CN**: 声明 TableGen def 记录 `VST4_F16`。
- **L538 EN**: Declares TableGen def record `VST2_LANE_F16`.
  **L538 CN**: 声明 TableGen def 记录 `VST2_LANE_F16`。
- **L539 EN**: Adds a standalone statement or declaration: `[ImmCheck<3, ImmCheckLaneIndex, 1>]>;`.
  **L539 CN**: 添加一条独立语句或声明：`[ImmCheck<3, ImmCheckLaneIndex, 1>]>;`。
- **L540 EN**: Declares TableGen def record `VST3_LANE_F16`.
  **L540 CN**: 声明 TableGen def 记录 `VST3_LANE_F16`。
- **L541 EN**: Adds a standalone statement or declaration: `[ImmCheck<4, ImmCheckLaneIndex, 1>]>;`.
  **L541 CN**: 添加一条独立语句或声明：`[ImmCheck<4, ImmCheckLaneIndex, 1>]>;`。
- **L542 EN**: Declares TableGen def record `VST4_LANE_F16`.
  **L542 CN**: 声明 TableGen def 记录 `VST4_LANE_F16`。
- **L543 EN**: Adds a standalone statement or declaration: `[ImmCheck<5, ImmCheckLaneIndex, 1>]>;`.
  **L543 CN**: 添加一条独立语句或声明：`[ImmCheck<5, ImmCheckLaneIndex, 1>]>;`。
- **L544 EN**: Closes the current lexical scope or compound statement.
  **L544 CN**: 结束当前词法作用域或复合语句块。
- **L545 EN**: Blank line separating nearby declarations or logic blocks.
  **L545 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L546 EN**: Separator comment used for visual grouping.
  **L546 CN**: 用于视觉分组的分隔注释。
- **L547 EN**: Comment explains nearby logic, constraints, or intent: `E.3.16 Extract lanes from a vector`.
  **L547 CN**: 注释解释附近代码的逻辑、约束或设计意图：`E.3.16 Extract lanes from a vector`。
- **L548 EN**: Assigns a TableGen property that affects following records or inherited fields: `let InstName = "vmov" in`.
  **L548 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let InstName = "vmov" in`。
- **L549 EN**: Declares TableGen def record `VGET_LANE`.
  **L549 CN**: 声明 TableGen def 记录 `VGET_LANE`。
- **L550 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"UcUsUicsiPcPsfQUcQUsQUiQcQsQiQPcQPsQflUlQlQUlmQm",`.
  **L550 CN**: 继续一个多行参数列表、初始化器或聚合项：`"UcUsUicsiPcPsfQUcQUsQUiQcQsQiQPcQPsQflUlQlQUlmQm",`。
- **L551 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheckLaneIndex, 0>]>;`.
  **L551 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheckLaneIndex, 0>]>;`。
- **L552 EN**: Blank line separating nearby declarations or logic blocks.
  **L552 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L553 EN**: Separator comment used for visual grouping.
  **L553 CN**: 用于视觉分组的分隔注释。
- **L554 EN**: Comment explains nearby logic, constraints, or intent: `E.3.17 Set lanes within a vector`.
  **L554 CN**: 注释解释附近代码的逻辑、约束或设计意图：`E.3.17 Set lanes within a vector`。
- **L555 EN**: Assigns a TableGen property that affects following records or inherited fields: `let InstName = "vmov" in`.
  **L555 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let InstName = "vmov" in`。
- **L556 EN**: Declares TableGen def record `VSET_LANE`.
  **L556 CN**: 声明 TableGen def 记录 `VSET_LANE`。
- **L557 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"UcUsUicsiPcPsfQUcQUsQUiQcQsQiQPcQPsQflUlQlQUlmQm",`.
  **L557 CN**: 继续一个多行参数列表、初始化器或聚合项：`"UcUsUicsiPcPsfQUcQUsQUiQcQsQiQPcQPsQflUlQlQUlmQm",`。
- **L558 EN**: Adds a standalone statement or declaration: `[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`.
  **L558 CN**: 添加一条独立语句或声明：`[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`。
- **L559 EN**: Blank line separating nearby declarations or logic blocks.
  **L559 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L560 EN**: Separator comment used for visual grouping.
  **L560 CN**: 用于视觉分组的分隔注释。

### Lines 561-588

````tablegen
// E.3.18 Initialize a vector from bit pattern
def VCREATE : NoTestOpInst<"vcreate", ".(IU>)", "csihfUcUsUiUlPcPslm", OP_CAST> {
  let BigEndianSafe = 1;
}

////////////////////////////////////////////////////////////////////////////////
// E.3.19 Set all lanes to same value
let InstName = "vmov" in {
def VDUP_N   : WOpInst<"vdup_n", ".1",
                       "UcUsUicsiPcPshfQUcQUsQUiQcQsQiQPcQPsQhQflUlQlQUlmQm",
                       OP_DUP>;
def VMOV_N   : WOpInst<"vmov_n", ".1",
                       "UcUsUicsiPcPshfQUcQUsQUiQcQsQiQPcQPsQhQflUlQlQUlmQm",
                       OP_DUP>;
}
let InstName = "" in
def VDUP_LANE: WOpInst<"vdup_lane", ".qI",
                       "UcUsUicsiPcPshfQUcQUsQUiQcQsQiQPcQPsQhQflUlQlQUlmQm",
                       OP_DUP_LN>;

////////////////////////////////////////////////////////////////////////////////
// E.3.20 Combining vectors
def VCOMBINE : NoTestOpInst<"vcombine", "Q..", "csilhfUcUsUiUlPcPsm", OP_CONC>;

////////////////////////////////////////////////////////////////////////////////
// E.3.21 Splitting vectors
// Note that the ARM NEON Reference 2.0 mistakenly document the vget_high_f16()
// and vget_low_f16() intrinsics as AArch64-only. We (and GCC) support all
````
- **L561 EN**: Comment explains nearby logic, constraints, or intent: `E.3.18 Initialize a vector from bit pattern`.
  **L561 CN**: 注释解释附近代码的逻辑、约束或设计意图：`E.3.18 Initialize a vector from bit pattern`。
- **L562 EN**: Declares TableGen def record `VCREATE`.
  **L562 CN**: 声明 TableGen def 记录 `VCREATE`。
- **L563 EN**: Assigns a TableGen property that affects following records or inherited fields: `let BigEndianSafe = 1;`.
  **L563 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let BigEndianSafe = 1;`。
- **L564 EN**: Closes the current lexical scope or compound statement.
  **L564 CN**: 结束当前词法作用域或复合语句块。
- **L565 EN**: Blank line separating nearby declarations or logic blocks.
  **L565 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L566 EN**: Separator comment used for visual grouping.
  **L566 CN**: 用于视觉分组的分隔注释。
- **L567 EN**: Comment explains nearby logic, constraints, or intent: `E.3.19 Set all lanes to same value`.
  **L567 CN**: 注释解释附近代码的逻辑、约束或设计意图：`E.3.19 Set all lanes to same value`。
- **L568 EN**: Assigns a TableGen property that affects following records or inherited fields: `let InstName = "vmov" in {`.
  **L568 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let InstName = "vmov" in {`。
- **L569 EN**: Declares TableGen def record `VDUP_N`.
  **L569 CN**: 声明 TableGen def 记录 `VDUP_N`。
- **L570 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"UcUsUicsiPcPshfQUcQUsQUiQcQsQiQPcQPsQhQflUlQlQUlmQm",`.
  **L570 CN**: 继续一个多行参数列表、初始化器或聚合项：`"UcUsUicsiPcPshfQUcQUsQUiQcQsQiQPcQPsQhQflUlQlQUlmQm",`。
- **L571 EN**: Adds a standalone statement or declaration: `OP_DUP>;`.
  **L571 CN**: 添加一条独立语句或声明：`OP_DUP>;`。
- **L572 EN**: Declares TableGen def record `VMOV_N`.
  **L572 CN**: 声明 TableGen def 记录 `VMOV_N`。
- **L573 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"UcUsUicsiPcPshfQUcQUsQUiQcQsQiQPcQPsQhQflUlQlQUlmQm",`.
  **L573 CN**: 继续一个多行参数列表、初始化器或聚合项：`"UcUsUicsiPcPshfQUcQUsQUiQcQsQiQPcQPsQhQflUlQlQUlmQm",`。
- **L574 EN**: Adds a standalone statement or declaration: `OP_DUP>;`.
  **L574 CN**: 添加一条独立语句或声明：`OP_DUP>;`。
- **L575 EN**: Closes the current lexical scope or compound statement.
  **L575 CN**: 结束当前词法作用域或复合语句块。
- **L576 EN**: Assigns a TableGen property that affects following records or inherited fields: `let InstName = "" in`.
  **L576 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let InstName = "" in`。
- **L577 EN**: Declares TableGen def record `VDUP_LANE`.
  **L577 CN**: 声明 TableGen def 记录 `VDUP_LANE`。
- **L578 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"UcUsUicsiPcPshfQUcQUsQUiQcQsQiQPcQPsQhQflUlQlQUlmQm",`.
  **L578 CN**: 继续一个多行参数列表、初始化器或聚合项：`"UcUsUicsiPcPshfQUcQUsQUiQcQsQiQPcQPsQhQflUlQlQUlmQm",`。
- **L579 EN**: Adds a standalone statement or declaration: `OP_DUP_LN>;`.
  **L579 CN**: 添加一条独立语句或声明：`OP_DUP_LN>;`。
- **L580 EN**: Blank line separating nearby declarations or logic blocks.
  **L580 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L581 EN**: Separator comment used for visual grouping.
  **L581 CN**: 用于视觉分组的分隔注释。
- **L582 EN**: Comment explains nearby logic, constraints, or intent: `E.3.20 Combining vectors`.
  **L582 CN**: 注释解释附近代码的逻辑、约束或设计意图：`E.3.20 Combining vectors`。
- **L583 EN**: Declares TableGen def record `VCOMBINE`.
  **L583 CN**: 声明 TableGen def 记录 `VCOMBINE`。
- **L584 EN**: Blank line separating nearby declarations or logic blocks.
  **L584 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L585 EN**: Separator comment used for visual grouping.
  **L585 CN**: 用于视觉分组的分隔注释。
- **L586 EN**: Comment explains nearby logic, constraints, or intent: `E.3.21 Splitting vectors`.
  **L586 CN**: 注释解释附近代码的逻辑、约束或设计意图：`E.3.21 Splitting vectors`。
- **L587 EN**: Comment highlights an implementation note: `Note that the ARM NEON Reference 2.0 mistakenly document the vget_high_f16()`.
  **L587 CN**: 注释强调一条实现说明：`Note that the ARM NEON Reference 2.0 mistakenly document the vget_high_f16()`。
- **L588 EN**: Comment explains nearby logic, constraints, or intent: `and vget_low_f16() intrinsics as AArch64-only. We (and GCC) support all`.
  **L588 CN**: 注释解释附近代码的逻辑、约束或设计意图：`and vget_low_f16() intrinsics as AArch64-only. We (and GCC) support all`。

### Lines 589-616

````tablegen
// versions of these intrinsics in both AArch32 and AArch64 architectures. See
// D45668 for more details.
let InstName = "vmov" in {
def VGET_HIGH : NoTestOpInst<"vget_high", ".Q", "csilhfUcUsUiUlPcPsm", OP_HI>;
def VGET_LOW  : NoTestOpInst<"vget_low", ".Q", "csilhfUcUsUiUlPcPsm", OP_LO>;
}

////////////////////////////////////////////////////////////////////////////////
// E.3.22 Converting vectors

let ArchGuard = "(__ARM_FP & 2)" in {
  def VCVT_F16_F32 : SInst<"vcvt_f16_f32", "(<q)(.!)", "Hf">;
  def VCVT_F32_F16 : SInst<"vcvt_f32_f16", "(>Q)(.!)", "h">;
}

def VCVT_S32     : SInst<"vcvt_s32", "S.",  "fQf">;
def VCVT_U32     : SInst<"vcvt_u32", "U.",  "fQf">;
def VCVT_F32     : SInst<"vcvt_f32", "F(.!)",  "iUiQiQUi">;
def VCVT_N_S32   : SInst<"vcvt_n_s32", "S.I", "fQf",
                        [ImmCheck<1, ImmCheck1_32>]>;
def VCVT_N_U32   : SInst<"vcvt_n_u32", "U.I", "fQf",
                        [ImmCheck<1, ImmCheck1_32>]>;
def VCVT_N_F32   : SInst<"vcvt_n_f32", "F(.!)I", "iUiQiQUi",
                        [ImmCheck<1, ImmCheck1_32>]>;

def VMOVN        : IInst<"vmovn", "<Q",  "silUsUiUl">;
def VMOVL        : SInst<"vmovl", "(>Q).",  "csiUcUsUi">;
def VQMOVN       : SInst<"vqmovn", "<Q",  "silUsUiUl">;
````
- **L589 EN**: Comment explains nearby logic, constraints, or intent: `versions of these intrinsics in both AArch32 and AArch64 architectures. See`.
  **L589 CN**: 注释解释附近代码的逻辑、约束或设计意图：`versions of these intrinsics in both AArch32 and AArch64 architectures. See`。
- **L590 EN**: Comment explains nearby logic, constraints, or intent: `D45668 for more details.`.
  **L590 CN**: 注释解释附近代码的逻辑、约束或设计意图：`D45668 for more details.`。
- **L591 EN**: Assigns a TableGen property that affects following records or inherited fields: `let InstName = "vmov" in {`.
  **L591 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let InstName = "vmov" in {`。
- **L592 EN**: Declares TableGen def record `VGET_HIGH`.
  **L592 CN**: 声明 TableGen def 记录 `VGET_HIGH`。
- **L593 EN**: Declares TableGen def record `VGET_LOW`.
  **L593 CN**: 声明 TableGen def 记录 `VGET_LOW`。
- **L594 EN**: Closes the current lexical scope or compound statement.
  **L594 CN**: 结束当前词法作用域或复合语句块。
- **L595 EN**: Blank line separating nearby declarations or logic blocks.
  **L595 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L596 EN**: Separator comment used for visual grouping.
  **L596 CN**: 用于视觉分组的分隔注释。
- **L597 EN**: Comment explains nearby logic, constraints, or intent: `E.3.22 Converting vectors`.
  **L597 CN**: 注释解释附近代码的逻辑、约束或设计意图：`E.3.22 Converting vectors`。
- **L598 EN**: Blank line separating nearby declarations or logic blocks.
  **L598 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L599 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "(__ARM_FP & 2)" in {`.
  **L599 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "(__ARM_FP & 2)" in {`。
- **L600 EN**: Declares TableGen def record `VCVT_F16_F32`.
  **L600 CN**: 声明 TableGen def 记录 `VCVT_F16_F32`。
- **L601 EN**: Declares TableGen def record `VCVT_F32_F16`.
  **L601 CN**: 声明 TableGen def 记录 `VCVT_F32_F16`。
- **L602 EN**: Closes the current lexical scope or compound statement.
  **L602 CN**: 结束当前词法作用域或复合语句块。
- **L603 EN**: Blank line separating nearby declarations or logic blocks.
  **L603 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L604 EN**: Declares TableGen def record `VCVT_S32`.
  **L604 CN**: 声明 TableGen def 记录 `VCVT_S32`。
- **L605 EN**: Declares TableGen def record `VCVT_U32`.
  **L605 CN**: 声明 TableGen def 记录 `VCVT_U32`。
- **L606 EN**: Declares TableGen def record `VCVT_F32`.
  **L606 CN**: 声明 TableGen def 记录 `VCVT_F32`。
- **L607 EN**: Declares TableGen def record `VCVT_N_S32`.
  **L607 CN**: 声明 TableGen def 记录 `VCVT_N_S32`。
- **L608 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheck1_32>]>;`.
  **L608 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheck1_32>]>;`。
- **L609 EN**: Declares TableGen def record `VCVT_N_U32`.
  **L609 CN**: 声明 TableGen def 记录 `VCVT_N_U32`。
- **L610 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheck1_32>]>;`.
  **L610 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheck1_32>]>;`。
- **L611 EN**: Declares TableGen def record `VCVT_N_F32`.
  **L611 CN**: 声明 TableGen def 记录 `VCVT_N_F32`。
- **L612 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheck1_32>]>;`.
  **L612 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheck1_32>]>;`。
- **L613 EN**: Blank line separating nearby declarations or logic blocks.
  **L613 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L614 EN**: Declares TableGen def record `VMOVN`.
  **L614 CN**: 声明 TableGen def 记录 `VMOVN`。
- **L615 EN**: Declares TableGen def record `VMOVL`.
  **L615 CN**: 声明 TableGen def 记录 `VMOVL`。
- **L616 EN**: Declares TableGen def record `VQMOVN`.
  **L616 CN**: 声明 TableGen def 记录 `VQMOVN`。

### Lines 617-644

````tablegen
def VQMOVUN      : SInst<"vqmovun", "(<U)Q",  "sil">;

////////////////////////////////////////////////////////////////////////////////
// E.3.23-24 Table lookup, Extended table lookup
let InstName = "vtbl" in {
def VTBL1 : WInst<"vtbl1", "..p",  "UccPcm">;
def VTBL2 : WInst<"vtbl2", ".2p",  "UccPcm">;
def VTBL3 : WInst<"vtbl3", ".3p",  "UccPcm">;
def VTBL4 : WInst<"vtbl4", ".4p",  "UccPcm">;
}
let InstName = "vtbx" in {
def VTBX1 : WInst<"vtbx1", "...p", "UccPcm">;
def VTBX2 : WInst<"vtbx2", "..2p", "UccPcm">;
def VTBX3 : WInst<"vtbx3", "..3p", "UccPcm">;
def VTBX4 : WInst<"vtbx4", "..4p", "UccPcm">;
}

////////////////////////////////////////////////////////////////////////////////
// E.3.25 Operations with a scalar value
def VMLA_LANE     : IOpInst<"vmla_lane", "...qI",
                            "siUsUifQsQiQUsQUiQf", OP_MLA_LN>;
def VMLAL_LANE    : SOpInst<"vmlal_lane", "(>Q)(>Q)..I", "siUsUi", OP_MLAL_LN>;
def VQDMLAL_LANE  : SOpInst<"vqdmlal_lane", "(>Q)(>Q)..I", "si", OP_QDMLAL_LN>;
def VMLS_LANE     : IOpInst<"vmls_lane", "...qI",
                            "siUsUifQsQiQUsQUiQf", OP_MLS_LN>;
def VMLSL_LANE    : SOpInst<"vmlsl_lane", "(>Q)(>Q)..I", "siUsUi", OP_MLSL_LN>;
def VQDMLSL_LANE  : SOpInst<"vqdmlsl_lane", "(>Q)(>Q)..I", "si", OP_QDMLSL_LN>;
def VMUL_N        : IOpInst<"vmul_n", "..1", "sifUsUiQsQiQfQUsQUi", OP_MUL_N>;
````
- **L617 EN**: Declares TableGen def record `VQMOVUN`.
  **L617 CN**: 声明 TableGen def 记录 `VQMOVUN`。
- **L618 EN**: Blank line separating nearby declarations or logic blocks.
  **L618 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L619 EN**: Separator comment used for visual grouping.
  **L619 CN**: 用于视觉分组的分隔注释。
- **L620 EN**: Comment explains nearby logic, constraints, or intent: `E.3.23-24 Table lookup, Extended table lookup`.
  **L620 CN**: 注释解释附近代码的逻辑、约束或设计意图：`E.3.23-24 Table lookup, Extended table lookup`。
- **L621 EN**: Assigns a TableGen property that affects following records or inherited fields: `let InstName = "vtbl" in {`.
  **L621 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let InstName = "vtbl" in {`。
- **L622 EN**: Declares TableGen def record `VTBL1`.
  **L622 CN**: 声明 TableGen def 记录 `VTBL1`。
- **L623 EN**: Declares TableGen def record `VTBL2`.
  **L623 CN**: 声明 TableGen def 记录 `VTBL2`。
- **L624 EN**: Declares TableGen def record `VTBL3`.
  **L624 CN**: 声明 TableGen def 记录 `VTBL3`。
- **L625 EN**: Declares TableGen def record `VTBL4`.
  **L625 CN**: 声明 TableGen def 记录 `VTBL4`。
- **L626 EN**: Closes the current lexical scope or compound statement.
  **L626 CN**: 结束当前词法作用域或复合语句块。
- **L627 EN**: Assigns a TableGen property that affects following records or inherited fields: `let InstName = "vtbx" in {`.
  **L627 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let InstName = "vtbx" in {`。
- **L628 EN**: Declares TableGen def record `VTBX1`.
  **L628 CN**: 声明 TableGen def 记录 `VTBX1`。
- **L629 EN**: Declares TableGen def record `VTBX2`.
  **L629 CN**: 声明 TableGen def 记录 `VTBX2`。
- **L630 EN**: Declares TableGen def record `VTBX3`.
  **L630 CN**: 声明 TableGen def 记录 `VTBX3`。
- **L631 EN**: Declares TableGen def record `VTBX4`.
  **L631 CN**: 声明 TableGen def 记录 `VTBX4`。
- **L632 EN**: Closes the current lexical scope or compound statement.
  **L632 CN**: 结束当前词法作用域或复合语句块。
- **L633 EN**: Blank line separating nearby declarations or logic blocks.
  **L633 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L634 EN**: Separator comment used for visual grouping.
  **L634 CN**: 用于视觉分组的分隔注释。
- **L635 EN**: Comment explains nearby logic, constraints, or intent: `E.3.25 Operations with a scalar value`.
  **L635 CN**: 注释解释附近代码的逻辑、约束或设计意图：`E.3.25 Operations with a scalar value`。
- **L636 EN**: Declares TableGen def record `VMLA_LANE`.
  **L636 CN**: 声明 TableGen def 记录 `VMLA_LANE`。
- **L637 EN**: Adds a standalone statement or declaration: `"siUsUifQsQiQUsQUiQf", OP_MLA_LN>;`.
  **L637 CN**: 添加一条独立语句或声明：`"siUsUifQsQiQUsQUiQf", OP_MLA_LN>;`。
- **L638 EN**: Declares TableGen def record `VMLAL_LANE`.
  **L638 CN**: 声明 TableGen def 记录 `VMLAL_LANE`。
- **L639 EN**: Declares TableGen def record `VQDMLAL_LANE`.
  **L639 CN**: 声明 TableGen def 记录 `VQDMLAL_LANE`。
- **L640 EN**: Declares TableGen def record `VMLS_LANE`.
  **L640 CN**: 声明 TableGen def 记录 `VMLS_LANE`。
- **L641 EN**: Adds a standalone statement or declaration: `"siUsUifQsQiQUsQUiQf", OP_MLS_LN>;`.
  **L641 CN**: 添加一条独立语句或声明：`"siUsUifQsQiQUsQUiQf", OP_MLS_LN>;`。
- **L642 EN**: Declares TableGen def record `VMLSL_LANE`.
  **L642 CN**: 声明 TableGen def 记录 `VMLSL_LANE`。
- **L643 EN**: Declares TableGen def record `VQDMLSL_LANE`.
  **L643 CN**: 声明 TableGen def 记录 `VQDMLSL_LANE`。
- **L644 EN**: Declares TableGen def record `VMUL_N`.
  **L644 CN**: 声明 TableGen def 记录 `VMUL_N`。

### Lines 645-672

````tablegen
def VMUL_LANE     : IOpInst<"vmul_lane", "..qI",
                            "sifUsUiQsQiQfQUsQUi", OP_MUL_LN>;
def VMULL_N       : SOpInst<"vmull_n", "(>Q).1", "siUsUi", OP_MULL_N>;
def VMULL_LANE    : SOpInst<"vmull_lane", "(>Q)..I", "siUsUi", OP_MULL_LN>;
def VQDMULL_N     : SOpInst<"vqdmull_n", "(>Q).1", "si", OP_QDMULL_N>;
def VQDMULL_LANE  : SOpInst<"vqdmull_lane", "(>Q)..I", "si", OP_QDMULL_LN>;
def VQDMULH_N     : SOpInst<"vqdmulh_n", "..1", "siQsQi", OP_QDMULH_N>;
def VQRDMULH_N    : SOpInst<"vqrdmulh_n", "..1", "siQsQi", OP_QRDMULH_N>;

let ArchGuard = "!defined(__aarch64__) && !defined(__arm64ec__)" in {
def VQDMULH_LANE  : SOpInst<"vqdmulh_lane", "..qI", "siQsQi", OP_QDMULH_LN>;
def VQRDMULH_LANE : SOpInst<"vqrdmulh_lane", "..qI", "siQsQi", OP_QRDMULH_LN>;
}
let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)" in {
def A64_VQDMULH_LANE  : SInst<"vqdmulh_lane", "..(!q)I", "siQsQi",
                              [ImmCheck<2, ImmCheckLaneIndex, 1>]>;
def A64_VQRDMULH_LANE : SInst<"vqrdmulh_lane", "..(!q)I", "siQsQi",
                              [ImmCheck<2, ImmCheckLaneIndex, 1>]>;
}

let TargetGuard = "v8.1a,neon" in {
def VQRDMLAH_LANE : SOpInst<"vqrdmlah_lane", "...qI", "siQsQi", OP_QRDMLAH_LN>;
def VQRDMLSH_LANE : SOpInst<"vqrdmlsh_lane", "...qI", "siQsQi", OP_QRDMLSH_LN>;
}

def VMLA_N        : IOpInst<"vmla_n", "...1", "siUsUifQsQiQUsQUiQf", OP_MLA_N>;
def VMLAL_N       : SOpInst<"vmlal_n", "(>Q)(>Q).1", "siUsUi", OP_MLAL_N>;
def VQDMLAL_N     : SOpInst<"vqdmlal_n", "(>Q)(>Q).1", "si", OP_QDMLAL_N>;
````
- **L645 EN**: Declares TableGen def record `VMUL_LANE`.
  **L645 CN**: 声明 TableGen def 记录 `VMUL_LANE`。
- **L646 EN**: Adds a standalone statement or declaration: `"sifUsUiQsQiQfQUsQUi", OP_MUL_LN>;`.
  **L646 CN**: 添加一条独立语句或声明：`"sifUsUiQsQiQfQUsQUi", OP_MUL_LN>;`。
- **L647 EN**: Declares TableGen def record `VMULL_N`.
  **L647 CN**: 声明 TableGen def 记录 `VMULL_N`。
- **L648 EN**: Declares TableGen def record `VMULL_LANE`.
  **L648 CN**: 声明 TableGen def 记录 `VMULL_LANE`。
- **L649 EN**: Declares TableGen def record `VQDMULL_N`.
  **L649 CN**: 声明 TableGen def 记录 `VQDMULL_N`。
- **L650 EN**: Declares TableGen def record `VQDMULL_LANE`.
  **L650 CN**: 声明 TableGen def 记录 `VQDMULL_LANE`。
- **L651 EN**: Declares TableGen def record `VQDMULH_N`.
  **L651 CN**: 声明 TableGen def 记录 `VQDMULH_N`。
- **L652 EN**: Declares TableGen def record `VQRDMULH_N`.
  **L652 CN**: 声明 TableGen def 记录 `VQRDMULH_N`。
- **L653 EN**: Blank line separating nearby declarations or logic blocks.
  **L653 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L654 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "!defined(__aarch64__) && !defined(__arm64ec__)" in {`.
  **L654 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "!defined(__aarch64__) && !defined(__arm64ec__)" in {`。
- **L655 EN**: Declares TableGen def record `VQDMULH_LANE`.
  **L655 CN**: 声明 TableGen def 记录 `VQDMULH_LANE`。
- **L656 EN**: Declares TableGen def record `VQRDMULH_LANE`.
  **L656 CN**: 声明 TableGen def 记录 `VQRDMULH_LANE`。
- **L657 EN**: Closes the current lexical scope or compound statement.
  **L657 CN**: 结束当前词法作用域或复合语句块。
- **L658 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)" in {`.
  **L658 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)" in {`。
- **L659 EN**: Declares TableGen def record `A64_VQDMULH_LANE`.
  **L659 CN**: 声明 TableGen def 记录 `A64_VQDMULH_LANE`。
- **L660 EN**: Adds a standalone statement or declaration: `[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`.
  **L660 CN**: 添加一条独立语句或声明：`[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`。
- **L661 EN**: Declares TableGen def record `A64_VQRDMULH_LANE`.
  **L661 CN**: 声明 TableGen def 记录 `A64_VQRDMULH_LANE`。
- **L662 EN**: Adds a standalone statement or declaration: `[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`.
  **L662 CN**: 添加一条独立语句或声明：`[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`。
- **L663 EN**: Closes the current lexical scope or compound statement.
  **L663 CN**: 结束当前词法作用域或复合语句块。
- **L664 EN**: Blank line separating nearby declarations or logic blocks.
  **L664 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L665 EN**: Assigns a TableGen property that affects following records or inherited fields: `let TargetGuard = "v8.1a,neon" in {`.
  **L665 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let TargetGuard = "v8.1a,neon" in {`。
- **L666 EN**: Declares TableGen def record `VQRDMLAH_LANE`.
  **L666 CN**: 声明 TableGen def 记录 `VQRDMLAH_LANE`。
- **L667 EN**: Declares TableGen def record `VQRDMLSH_LANE`.
  **L667 CN**: 声明 TableGen def 记录 `VQRDMLSH_LANE`。
- **L668 EN**: Closes the current lexical scope or compound statement.
  **L668 CN**: 结束当前词法作用域或复合语句块。
- **L669 EN**: Blank line separating nearby declarations or logic blocks.
  **L669 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L670 EN**: Declares TableGen def record `VMLA_N`.
  **L670 CN**: 声明 TableGen def 记录 `VMLA_N`。
- **L671 EN**: Declares TableGen def record `VMLAL_N`.
  **L671 CN**: 声明 TableGen def 记录 `VMLAL_N`。
- **L672 EN**: Declares TableGen def record `VQDMLAL_N`.
  **L672 CN**: 声明 TableGen def 记录 `VQDMLAL_N`。

### Lines 673-700

````tablegen
def VMLS_N        : IOpInst<"vmls_n", "...1", "siUsUifQsQiQUsQUiQf", OP_MLS_N>;
def VMLSL_N       : SOpInst<"vmlsl_n", "(>Q)(>Q).1", "siUsUi", OP_MLSL_N>;
def VQDMLSL_N     : SOpInst<"vqdmlsl_n", "(>Q)(>Q).1", "si", OP_QDMLSL_N>;

////////////////////////////////////////////////////////////////////////////////
// E.3.26 Vector Extract
def VEXT : WInst<"vext", "...I",
                 "cUcPcsUsPsiUilUlfQcQUcQPcQsQUsQPsQiQUiQlQUlQfmQm",
                 [ImmCheck<2, ImmCheckLaneIndex, 0>]>;

////////////////////////////////////////////////////////////////////////////////
// E.3.27 Reverse vector elements
def VREV64 : WOpInst<"vrev64", "..", "csiUcUsUiPcPsfQcQsQiQUcQUsQUiQPcQPsQfmQm",
                  OP_REV64>;
def VREV32 : WOpInst<"vrev32", "..", "csUcUsPcPsQcQsQUcQUsQPcQPsmQm", OP_REV32>;
def VREV16 : WOpInst<"vrev16", "..", "cUcPcQcQUcQPcmQm", OP_REV16>;

////////////////////////////////////////////////////////////////////////////////
// E.3.28 Other single operand arithmetic
def VABS    : SInst<"vabs", "..", "csifQcQsQiQf">;
def VQABS   : SInst<"vqabs", "..", "csiQcQsQi">;
def VNEG    : SOpInst<"vneg", "..", "csifQcQsQiQf", OP_NEG>;
def VQNEG   : SInst<"vqneg", "..", "csiQcQsQi">;
def VCLS    : SInst<"vcls", "S.", "csiUcUsUiQcQsQiQUcQUsQUi">;
def VCLZ    : IInst<"vclz", "..", "csiUcUsUiQcQsQiQUcQUsQUi">;
def VCNT    : WInst<"vcnt", "..", "UccPcQUcQcQPc">;
def VRECPE  : SInst<"vrecpe", "..", "fUiQfQUi">;
def VRSQRTE : SInst<"vrsqrte", "..", "fUiQfQUi">;
````
- **L673 EN**: Declares TableGen def record `VMLS_N`.
  **L673 CN**: 声明 TableGen def 记录 `VMLS_N`。
- **L674 EN**: Declares TableGen def record `VMLSL_N`.
  **L674 CN**: 声明 TableGen def 记录 `VMLSL_N`。
- **L675 EN**: Declares TableGen def record `VQDMLSL_N`.
  **L675 CN**: 声明 TableGen def 记录 `VQDMLSL_N`。
- **L676 EN**: Blank line separating nearby declarations or logic blocks.
  **L676 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L677 EN**: Separator comment used for visual grouping.
  **L677 CN**: 用于视觉分组的分隔注释。
- **L678 EN**: Comment explains nearby logic, constraints, or intent: `E.3.26 Vector Extract`.
  **L678 CN**: 注释解释附近代码的逻辑、约束或设计意图：`E.3.26 Vector Extract`。
- **L679 EN**: Declares TableGen def record `VEXT`.
  **L679 CN**: 声明 TableGen def 记录 `VEXT`。
- **L680 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"cUcPcsUsPsiUilUlfQcQUcQPcQsQUsQPsQiQUiQlQUlQfmQm",`.
  **L680 CN**: 继续一个多行参数列表、初始化器或聚合项：`"cUcPcsUsPsiUilUlfQcQUcQPcQsQUsQPsQiQUiQlQUlQfmQm",`。
- **L681 EN**: Adds a standalone statement or declaration: `[ImmCheck<2, ImmCheckLaneIndex, 0>]>;`.
  **L681 CN**: 添加一条独立语句或声明：`[ImmCheck<2, ImmCheckLaneIndex, 0>]>;`。
- **L682 EN**: Blank line separating nearby declarations or logic blocks.
  **L682 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L683 EN**: Separator comment used for visual grouping.
  **L683 CN**: 用于视觉分组的分隔注释。
- **L684 EN**: Comment explains nearby logic, constraints, or intent: `E.3.27 Reverse vector elements`.
  **L684 CN**: 注释解释附近代码的逻辑、约束或设计意图：`E.3.27 Reverse vector elements`。
- **L685 EN**: Declares TableGen def record `VREV64`.
  **L685 CN**: 声明 TableGen def 记录 `VREV64`。
- **L686 EN**: Adds a standalone statement or declaration: `OP_REV64>;`.
  **L686 CN**: 添加一条独立语句或声明：`OP_REV64>;`。
- **L687 EN**: Declares TableGen def record `VREV32`.
  **L687 CN**: 声明 TableGen def 记录 `VREV32`。
- **L688 EN**: Declares TableGen def record `VREV16`.
  **L688 CN**: 声明 TableGen def 记录 `VREV16`。
- **L689 EN**: Blank line separating nearby declarations or logic blocks.
  **L689 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L690 EN**: Separator comment used for visual grouping.
  **L690 CN**: 用于视觉分组的分隔注释。
- **L691 EN**: Comment explains nearby logic, constraints, or intent: `E.3.28 Other single operand arithmetic`.
  **L691 CN**: 注释解释附近代码的逻辑、约束或设计意图：`E.3.28 Other single operand arithmetic`。
- **L692 EN**: Declares TableGen def record `VABS`.
  **L692 CN**: 声明 TableGen def 记录 `VABS`。
- **L693 EN**: Declares TableGen def record `VQABS`.
  **L693 CN**: 声明 TableGen def 记录 `VQABS`。
- **L694 EN**: Declares TableGen def record `VNEG`.
  **L694 CN**: 声明 TableGen def 记录 `VNEG`。
- **L695 EN**: Declares TableGen def record `VQNEG`.
  **L695 CN**: 声明 TableGen def 记录 `VQNEG`。
- **L696 EN**: Declares TableGen def record `VCLS`.
  **L696 CN**: 声明 TableGen def 记录 `VCLS`。
- **L697 EN**: Declares TableGen def record `VCLZ`.
  **L697 CN**: 声明 TableGen def 记录 `VCLZ`。
- **L698 EN**: Declares TableGen def record `VCNT`.
  **L698 CN**: 声明 TableGen def 记录 `VCNT`。
- **L699 EN**: Declares TableGen def record `VRECPE`.
  **L699 CN**: 声明 TableGen def 记录 `VRECPE`。
- **L700 EN**: Declares TableGen def record `VRSQRTE`.
  **L700 CN**: 声明 TableGen def 记录 `VRSQRTE`。

### Lines 701-728

````tablegen

////////////////////////////////////////////////////////////////////////////////
// E.3.29 Logical operations
def VMVN : LOpInst<"vmvn", "..", "csiUcUsUiPcQcQsQiQUcQUsQUiQPc", OP_NOT>;
def VAND : LOpInst<"vand", "...", "csilUcUsUiUlQcQsQiQlQUcQUsQUiQUl", OP_AND>;
def VORR : LOpInst<"vorr", "...", "csilUcUsUiUlQcQsQiQlQUcQUsQUiQUl", OP_OR>;
def VEOR : LOpInst<"veor", "...", "csilUcUsUiUlQcQsQiQlQUcQUsQUiQUl", OP_XOR>;
def VBIC : LOpInst<"vbic", "...", "csilUcUsUiUlQcQsQiQlQUcQUsQUiQUl", OP_ANDN>;
def VORN : LOpInst<"vorn", "...", "csilUcUsUiUlQcQsQiQlQUcQUsQUiQUl", OP_ORN>;
let isHiddenLInst = 1 in
def VBSL : SInst<"vbsl", ".U..",
                "csilUcUsUiUlfPcPsQcQsQiQlQUcQUsQUiQUlQfQPcQPsmQm">;

////////////////////////////////////////////////////////////////////////////////
// E.3.30 Transposition operations
def VTRN : WInst<"vtrn", "2..", "csiUcUsUifPcPsQcQsQiQUcQUsQUiQfQPcQPsmQm">;
def VZIP : WInst<"vzip", "2..", "csiUcUsUifPcPsQcQsQiQUcQUsQUiQfQPcQPsmQm">;
def VUZP : WInst<"vuzp", "2..", "csiUcUsUifPcPsQcQsQiQUcQUsQUiQfQPcQPsmQm">;

////////////////////////////////////////////////////////////////////////////////

class REINTERPRET_CROSS_SELF<string Types> :
  NoTestOpInst<"vreinterpret", "..", Types, OP_REINT> {
    let CartesianProductWith = Types;
}

multiclass REINTERPRET_CROSS_TYPES<string TypesA, string TypesB> {
  def AXB: NoTestOpInst<"vreinterpret", "..", TypesA, OP_REINT> {
````
- **L701 EN**: Blank line separating nearby declarations or logic blocks.
  **L701 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L702 EN**: Separator comment used for visual grouping.
  **L702 CN**: 用于视觉分组的分隔注释。
- **L703 EN**: Comment explains nearby logic, constraints, or intent: `E.3.29 Logical operations`.
  **L703 CN**: 注释解释附近代码的逻辑、约束或设计意图：`E.3.29 Logical operations`。
- **L704 EN**: Declares TableGen def record `VMVN`.
  **L704 CN**: 声明 TableGen def 记录 `VMVN`。
- **L705 EN**: Declares TableGen def record `VAND`.
  **L705 CN**: 声明 TableGen def 记录 `VAND`。
- **L706 EN**: Declares TableGen def record `VORR`.
  **L706 CN**: 声明 TableGen def 记录 `VORR`。
- **L707 EN**: Declares TableGen def record `VEOR`.
  **L707 CN**: 声明 TableGen def 记录 `VEOR`。
- **L708 EN**: Declares TableGen def record `VBIC`.
  **L708 CN**: 声明 TableGen def 记录 `VBIC`。
- **L709 EN**: Declares TableGen def record `VORN`.
  **L709 CN**: 声明 TableGen def 记录 `VORN`。
- **L710 EN**: Assigns a TableGen property that affects following records or inherited fields: `let isHiddenLInst = 1 in`.
  **L710 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let isHiddenLInst = 1 in`。
- **L711 EN**: Declares TableGen def record `VBSL`.
  **L711 CN**: 声明 TableGen def 记录 `VBSL`。
- **L712 EN**: Adds a standalone statement or declaration: `"csilUcUsUiUlfPcPsQcQsQiQlQUcQUsQUiQUlQfQPcQPsmQm">;`.
  **L712 CN**: 添加一条独立语句或声明：`"csilUcUsUiUlfPcPsQcQsQiQlQUcQUsQUiQUlQfQPcQPsmQm">;`。
- **L713 EN**: Blank line separating nearby declarations or logic blocks.
  **L713 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L714 EN**: Separator comment used for visual grouping.
  **L714 CN**: 用于视觉分组的分隔注释。
- **L715 EN**: Comment explains nearby logic, constraints, or intent: `E.3.30 Transposition operations`.
  **L715 CN**: 注释解释附近代码的逻辑、约束或设计意图：`E.3.30 Transposition operations`。
- **L716 EN**: Declares TableGen def record `VTRN`.
  **L716 CN**: 声明 TableGen def 记录 `VTRN`。
- **L717 EN**: Declares TableGen def record `VZIP`.
  **L717 CN**: 声明 TableGen def 记录 `VZIP`。
- **L718 EN**: Declares TableGen def record `VUZP`.
  **L718 CN**: 声明 TableGen def 记录 `VUZP`。
- **L719 EN**: Blank line separating nearby declarations or logic blocks.
  **L719 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L720 EN**: Separator comment used for visual grouping.
  **L720 CN**: 用于视觉分组的分隔注释。
- **L721 EN**: Blank line separating nearby declarations or logic blocks.
  **L721 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L722 EN**: Declares TableGen class record `REINTERPRET_CROSS_SELF`.
  **L722 CN**: 声明 TableGen class 记录 `REINTERPRET_CROSS_SELF`。
- **L723 EN**: Continues the surrounding expression or declaration: `NoTestOpInst<"vreinterpret", "..", Types, OP_REINT> {`.
  **L723 CN**: 继续构造周围的表达式或声明：`NoTestOpInst<"vreinterpret", "..", Types, OP_REINT> {`。
- **L724 EN**: Assigns a TableGen property that affects following records or inherited fields: `let CartesianProductWith = Types;`.
  **L724 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let CartesianProductWith = Types;`。
- **L725 EN**: Closes the current lexical scope or compound statement.
  **L725 CN**: 结束当前词法作用域或复合语句块。
- **L726 EN**: Blank line separating nearby declarations or logic blocks.
  **L726 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L727 EN**: Declares TableGen multiclass record `REINTERPRET_CROSS_TYPES`.
  **L727 CN**: 声明 TableGen multiclass 记录 `REINTERPRET_CROSS_TYPES`。
- **L728 EN**: Declares TableGen def record `AXB`.
  **L728 CN**: 声明 TableGen def 记录 `AXB`。

### Lines 729-756

````tablegen
    let CartesianProductWith = TypesB;
  }
  def BXA: NoTestOpInst<"vreinterpret", "..", TypesB, OP_REINT> {
    let CartesianProductWith = TypesA;
  }
}

// E.3.31 Vector reinterpret cast operations
def VREINTERPRET : REINTERPRET_CROSS_SELF<"csilUcUsUiUlhfPcPsQcQsQiQlQUcQUsQUiQUlQhQfQPcQPs"> {
  let ArchGuard = "!defined(__aarch64__) && !defined(__arm64ec__)";
  let BigEndianSafe = 1;
}

////////////////////////////////////////////////////////////////////////////////
// Vector fused multiply-add operations

let ArchGuard = "defined(__ARM_FEATURE_FMA)" in {
  def VFMA : SInst<"vfma", "....", "fQf">;
  def VFMS : SOpInst<"vfms", "....", "fQf", OP_FMLS>;
  def FMLA_N_F32 : SOpInst<"vfma_n", "...1", "fQf", OP_FMLA_N>;
}

////////////////////////////////////////////////////////////////////////////////
// fp16 vector operations
def SCALAR_HALF_GET_LANE : IOpInst<"vget_lane", "1.I", "h", OP_SCALAR_HALF_GET_LN>;
def SCALAR_HALF_SET_LANE : IOpInst<"vset_lane", ".1.I", "h", OP_SCALAR_HALF_SET_LN>;
def SCALAR_HALF_GET_LANEQ : IOpInst<"vget_lane", "1.I", "Qh", OP_SCALAR_HALF_GET_LNQ>;
def SCALAR_HALF_SET_LANEQ : IOpInst<"vset_lane", ".1.I", "Qh", OP_SCALAR_HALF_SET_LNQ>;
````
- **L729 EN**: Assigns a TableGen property that affects following records or inherited fields: `let CartesianProductWith = TypesB;`.
  **L729 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let CartesianProductWith = TypesB;`。
- **L730 EN**: Closes the current lexical scope or compound statement.
  **L730 CN**: 结束当前词法作用域或复合语句块。
- **L731 EN**: Declares TableGen def record `BXA`.
  **L731 CN**: 声明 TableGen def 记录 `BXA`。
- **L732 EN**: Assigns a TableGen property that affects following records or inherited fields: `let CartesianProductWith = TypesA;`.
  **L732 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let CartesianProductWith = TypesA;`。
- **L733 EN**: Closes the current lexical scope or compound statement.
  **L733 CN**: 结束当前词法作用域或复合语句块。
- **L734 EN**: Closes the current lexical scope or compound statement.
  **L734 CN**: 结束当前词法作用域或复合语句块。
- **L735 EN**: Blank line separating nearby declarations or logic blocks.
  **L735 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L736 EN**: Comment explains nearby logic, constraints, or intent: `E.3.31 Vector reinterpret cast operations`.
  **L736 CN**: 注释解释附近代码的逻辑、约束或设计意图：`E.3.31 Vector reinterpret cast operations`。
- **L737 EN**: Declares TableGen def record `VREINTERPRET`.
  **L737 CN**: 声明 TableGen def 记录 `VREINTERPRET`。
- **L738 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "!defined(__aarch64__) && !defined(__arm64ec__)";`.
  **L738 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "!defined(__aarch64__) && !defined(__arm64ec__)";`。
- **L739 EN**: Assigns a TableGen property that affects following records or inherited fields: `let BigEndianSafe = 1;`.
  **L739 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let BigEndianSafe = 1;`。
- **L740 EN**: Closes the current lexical scope or compound statement.
  **L740 CN**: 结束当前词法作用域或复合语句块。
- **L741 EN**: Blank line separating nearby declarations or logic blocks.
  **L741 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L742 EN**: Separator comment used for visual grouping.
  **L742 CN**: 用于视觉分组的分隔注释。
- **L743 EN**: Comment explains nearby logic, constraints, or intent: `Vector fused multiply-add operations`.
  **L743 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Vector fused multiply-add operations`。
- **L744 EN**: Blank line separating nearby declarations or logic blocks.
  **L744 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L745 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "defined(__ARM_FEATURE_FMA)" in {`.
  **L745 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "defined(__ARM_FEATURE_FMA)" in {`。
- **L746 EN**: Declares TableGen def record `VFMA`.
  **L746 CN**: 声明 TableGen def 记录 `VFMA`。
- **L747 EN**: Declares TableGen def record `VFMS`.
  **L747 CN**: 声明 TableGen def 记录 `VFMS`。
- **L748 EN**: Declares TableGen def record `FMLA_N_F32`.
  **L748 CN**: 声明 TableGen def 记录 `FMLA_N_F32`。
- **L749 EN**: Closes the current lexical scope or compound statement.
  **L749 CN**: 结束当前词法作用域或复合语句块。
- **L750 EN**: Blank line separating nearby declarations or logic blocks.
  **L750 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L751 EN**: Separator comment used for visual grouping.
  **L751 CN**: 用于视觉分组的分隔注释。
- **L752 EN**: Comment explains nearby logic, constraints, or intent: `fp16 vector operations`.
  **L752 CN**: 注释解释附近代码的逻辑、约束或设计意图：`fp16 vector operations`。
- **L753 EN**: Declares TableGen def record `SCALAR_HALF_GET_LANE`.
  **L753 CN**: 声明 TableGen def 记录 `SCALAR_HALF_GET_LANE`。
- **L754 EN**: Declares TableGen def record `SCALAR_HALF_SET_LANE`.
  **L754 CN**: 声明 TableGen def 记录 `SCALAR_HALF_SET_LANE`。
- **L755 EN**: Declares TableGen def record `SCALAR_HALF_GET_LANEQ`.
  **L755 CN**: 声明 TableGen def 记录 `SCALAR_HALF_GET_LANEQ`。
- **L756 EN**: Declares TableGen def record `SCALAR_HALF_SET_LANEQ`.
  **L756 CN**: 声明 TableGen def 记录 `SCALAR_HALF_SET_LANEQ`。

### Lines 757-784

````tablegen

////////////////////////////////////////////////////////////////////////////////
// Non poly128_t vaddp for Arm and AArch64
// TODO: poly128_t not implemented on arm32
def VADDP   : WInst<"vadd", "...", "PcPsPlQPcQPsQPl">;

////////////////////////////////////////////////////////////////////////////////
// AArch64 Intrinsics

let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)" in {

////////////////////////////////////////////////////////////////////////////////
// Load/Store
def LD1 : WInst<"vld1", ".(c*!)", "dQdPlQPlmQm">;
def LD2 : WInst<"vld2", "2(c*!)", "QUlQldQdPlQPlmQm">;
def LD3 : WInst<"vld3", "3(c*!)", "QUlQldQdPlQPlmQm">;
def LD4 : WInst<"vld4", "4(c*!)", "QUlQldQdPlQPlmQm">;
def ST1 : WInst<"vst1", "v*(.!)", "dQdPlQPlmQm">;
def ST2 : WInst<"vst2", "v*(2!)", "QUlQldQdPlQPlmQm">;
def ST3 : WInst<"vst3", "v*(3!)", "QUlQldQdPlQPlmQm">;
def ST4 : WInst<"vst4", "v*(4!)", "QUlQldQdPlQPlmQm">;

def LD1_X2 : WInst<"vld1_x2", "2(c*!)",
                   "dQdPlQPlmQm">;
def LD1_X3 : WInst<"vld1_x3", "3(c*!)",
                   "dQdPlQPlmQm">;
def LD1_X4 : WInst<"vld1_x4", "4(c*!)",
                   "dQdPlQPlmQm">;
````
- **L757 EN**: Blank line separating nearby declarations or logic blocks.
  **L757 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L758 EN**: Separator comment used for visual grouping.
  **L758 CN**: 用于视觉分组的分隔注释。
- **L759 EN**: Comment explains nearby logic, constraints, or intent: `Non poly128_t vaddp for Arm and AArch64`.
  **L759 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Non poly128_t vaddp for Arm and AArch64`。
- **L760 EN**: Comment records a pending task or caution: `TODO: poly128_t not implemented on arm32`.
  **L760 CN**: 注释记录待办事项或注意点：`TODO: poly128_t not implemented on arm32`。
- **L761 EN**: Declares TableGen def record `VADDP`.
  **L761 CN**: 声明 TableGen def 记录 `VADDP`。
- **L762 EN**: Blank line separating nearby declarations or logic blocks.
  **L762 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L763 EN**: Separator comment used for visual grouping.
  **L763 CN**: 用于视觉分组的分隔注释。
- **L764 EN**: Comment explains nearby logic, constraints, or intent: `AArch64 Intrinsics`.
  **L764 CN**: 注释解释附近代码的逻辑、约束或设计意图：`AArch64 Intrinsics`。
- **L765 EN**: Blank line separating nearby declarations or logic blocks.
  **L765 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L766 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)" in {`.
  **L766 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)" in {`。
- **L767 EN**: Blank line separating nearby declarations or logic blocks.
  **L767 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L768 EN**: Separator comment used for visual grouping.
  **L768 CN**: 用于视觉分组的分隔注释。
- **L769 EN**: Comment explains nearby logic, constraints, or intent: `Load/Store`.
  **L769 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Load/Store`。
- **L770 EN**: Declares TableGen def record `LD1`.
  **L770 CN**: 声明 TableGen def 记录 `LD1`。
- **L771 EN**: Declares TableGen def record `LD2`.
  **L771 CN**: 声明 TableGen def 记录 `LD2`。
- **L772 EN**: Declares TableGen def record `LD3`.
  **L772 CN**: 声明 TableGen def 记录 `LD3`。
- **L773 EN**: Declares TableGen def record `LD4`.
  **L773 CN**: 声明 TableGen def 记录 `LD4`。
- **L774 EN**: Declares TableGen def record `ST1`.
  **L774 CN**: 声明 TableGen def 记录 `ST1`。
- **L775 EN**: Declares TableGen def record `ST2`.
  **L775 CN**: 声明 TableGen def 记录 `ST2`。
- **L776 EN**: Declares TableGen def record `ST3`.
  **L776 CN**: 声明 TableGen def 记录 `ST3`。
- **L777 EN**: Declares TableGen def record `ST4`.
  **L777 CN**: 声明 TableGen def 记录 `ST4`。
- **L778 EN**: Blank line separating nearby declarations or logic blocks.
  **L778 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L779 EN**: Declares TableGen def record `LD1_X2`.
  **L779 CN**: 声明 TableGen def 记录 `LD1_X2`。
- **L780 EN**: Adds a standalone statement or declaration: `"dQdPlQPlmQm">;`.
  **L780 CN**: 添加一条独立语句或声明：`"dQdPlQPlmQm">;`。
- **L781 EN**: Declares TableGen def record `LD1_X3`.
  **L781 CN**: 声明 TableGen def 记录 `LD1_X3`。
- **L782 EN**: Adds a standalone statement or declaration: `"dQdPlQPlmQm">;`.
  **L782 CN**: 添加一条独立语句或声明：`"dQdPlQPlmQm">;`。
- **L783 EN**: Declares TableGen def record `LD1_X4`.
  **L783 CN**: 声明 TableGen def 记录 `LD1_X4`。
- **L784 EN**: Adds a standalone statement or declaration: `"dQdPlQPlmQm">;`.
  **L784 CN**: 添加一条独立语句或声明：`"dQdPlQPlmQm">;`。

### Lines 785-812

````tablegen

def ST1_X2 : WInst<"vst1_x2", "v*(2!)", "dQdPlQPlmQm">;
def ST1_X3 : WInst<"vst1_x3", "v*(3!)", "dQdPlQPlmQm">;
def ST1_X4 : WInst<"vst1_x4", "v*(4!)", "dQdPlQPlmQm">;

def LD1_LANE : WInst<"vld1_lane", ".(c*!).I", "dQdPlQPlmQm",
                    [ImmCheck<2, ImmCheckLaneIndex, 1>]>;
def LD2_LANE : WInst<"vld2_lane", "2(c*!)2I", "lUlQcQUcQPcQlQUldQdPlQPlmQm",
                    [ImmCheck<4, ImmCheckLaneIndex, 1>]>;
def LD3_LANE : WInst<"vld3_lane", "3(c*!)3I", "lUlQcQUcQPcQlQUldQdPlQPlmQm",
                    [ImmCheck<5, ImmCheckLaneIndex, 1>]>;
def LD4_LANE : WInst<"vld4_lane", "4(c*!)4I", "lUlQcQUcQPcQlQUldQdPlQPlmQm",
                    [ImmCheck<6, ImmCheckLaneIndex, 1>]>;
def ST1_LANE : WInst<"vst1_lane", "v*(.!)I", "dQdPlQPlmQm",
                    [ImmCheck<2, ImmCheckLaneIndex, 1>]>;
def ST2_LANE : WInst<"vst2_lane", "v*(2!)I", "lUlQcQUcQPcQlQUldQdPlQPlmQm",
                    [ImmCheck<3, ImmCheckLaneIndex, 1>]>;
def ST3_LANE : WInst<"vst3_lane", "v*(3!)I", "lUlQcQUcQPcQlQUldQdPlQPlmQm",
                    [ImmCheck<4, ImmCheckLaneIndex, 1>]>;
def ST4_LANE : WInst<"vst4_lane", "v*(4!)I", "lUlQcQUcQPcQlQUldQdPlQPlmQm",
                    [ImmCheck<5, ImmCheckLaneIndex, 1>]>;

def LD1_DUP  : WInst<"vld1_dup", ".(c*!)", "dQdPlQPlmQm">;
def LD2_DUP  : WInst<"vld2_dup", "2(c*!)", "dQdPlQPlmQm">;
def LD3_DUP  : WInst<"vld3_dup", "3(c*!)", "dQdPlQPlmQm">;
def LD4_DUP  : WInst<"vld4_dup", "4(c*!)", "dQdPlQPlmQm">;

def VLDRQ : WInst<"vldrq", "1(c*!)", "Pk">;
````
- **L785 EN**: Blank line separating nearby declarations or logic blocks.
  **L785 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L786 EN**: Declares TableGen def record `ST1_X2`.
  **L786 CN**: 声明 TableGen def 记录 `ST1_X2`。
- **L787 EN**: Declares TableGen def record `ST1_X3`.
  **L787 CN**: 声明 TableGen def 记录 `ST1_X3`。
- **L788 EN**: Declares TableGen def record `ST1_X4`.
  **L788 CN**: 声明 TableGen def 记录 `ST1_X4`。
- **L789 EN**: Blank line separating nearby declarations or logic blocks.
  **L789 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L790 EN**: Declares TableGen def record `LD1_LANE`.
  **L790 CN**: 声明 TableGen def 记录 `LD1_LANE`。
- **L791 EN**: Adds a standalone statement or declaration: `[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`.
  **L791 CN**: 添加一条独立语句或声明：`[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`。
- **L792 EN**: Declares TableGen def record `LD2_LANE`.
  **L792 CN**: 声明 TableGen def 记录 `LD2_LANE`。
- **L793 EN**: Adds a standalone statement or declaration: `[ImmCheck<4, ImmCheckLaneIndex, 1>]>;`.
  **L793 CN**: 添加一条独立语句或声明：`[ImmCheck<4, ImmCheckLaneIndex, 1>]>;`。
- **L794 EN**: Declares TableGen def record `LD3_LANE`.
  **L794 CN**: 声明 TableGen def 记录 `LD3_LANE`。
- **L795 EN**: Adds a standalone statement or declaration: `[ImmCheck<5, ImmCheckLaneIndex, 1>]>;`.
  **L795 CN**: 添加一条独立语句或声明：`[ImmCheck<5, ImmCheckLaneIndex, 1>]>;`。
- **L796 EN**: Declares TableGen def record `LD4_LANE`.
  **L796 CN**: 声明 TableGen def 记录 `LD4_LANE`。
- **L797 EN**: Adds a standalone statement or declaration: `[ImmCheck<6, ImmCheckLaneIndex, 1>]>;`.
  **L797 CN**: 添加一条独立语句或声明：`[ImmCheck<6, ImmCheckLaneIndex, 1>]>;`。
- **L798 EN**: Declares TableGen def record `ST1_LANE`.
  **L798 CN**: 声明 TableGen def 记录 `ST1_LANE`。
- **L799 EN**: Adds a standalone statement or declaration: `[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`.
  **L799 CN**: 添加一条独立语句或声明：`[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`。
- **L800 EN**: Declares TableGen def record `ST2_LANE`.
  **L800 CN**: 声明 TableGen def 记录 `ST2_LANE`。
- **L801 EN**: Adds a standalone statement or declaration: `[ImmCheck<3, ImmCheckLaneIndex, 1>]>;`.
  **L801 CN**: 添加一条独立语句或声明：`[ImmCheck<3, ImmCheckLaneIndex, 1>]>;`。
- **L802 EN**: Declares TableGen def record `ST3_LANE`.
  **L802 CN**: 声明 TableGen def 记录 `ST3_LANE`。
- **L803 EN**: Adds a standalone statement or declaration: `[ImmCheck<4, ImmCheckLaneIndex, 1>]>;`.
  **L803 CN**: 添加一条独立语句或声明：`[ImmCheck<4, ImmCheckLaneIndex, 1>]>;`。
- **L804 EN**: Declares TableGen def record `ST4_LANE`.
  **L804 CN**: 声明 TableGen def 记录 `ST4_LANE`。
- **L805 EN**: Adds a standalone statement or declaration: `[ImmCheck<5, ImmCheckLaneIndex, 1>]>;`.
  **L805 CN**: 添加一条独立语句或声明：`[ImmCheck<5, ImmCheckLaneIndex, 1>]>;`。
- **L806 EN**: Blank line separating nearby declarations or logic blocks.
  **L806 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L807 EN**: Declares TableGen def record `LD1_DUP`.
  **L807 CN**: 声明 TableGen def 记录 `LD1_DUP`。
- **L808 EN**: Declares TableGen def record `LD2_DUP`.
  **L808 CN**: 声明 TableGen def 记录 `LD2_DUP`。
- **L809 EN**: Declares TableGen def record `LD3_DUP`.
  **L809 CN**: 声明 TableGen def 记录 `LD3_DUP`。
- **L810 EN**: Declares TableGen def record `LD4_DUP`.
  **L810 CN**: 声明 TableGen def 记录 `LD4_DUP`。
- **L811 EN**: Blank line separating nearby declarations or logic blocks.
  **L811 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L812 EN**: Declares TableGen def record `VLDRQ`.
  **L812 CN**: 声明 TableGen def 记录 `VLDRQ`。

### Lines 813-840

````tablegen
def VSTRQ : WInst<"vstrq", "v*(1!)", "Pk">;

////////////////////////////////////////////////////////////////////////////////
// Addition
def ADD : IOpInst<"vadd", "...", "dQd", OP_ADD>;

////////////////////////////////////////////////////////////////////////////////
// Subtraction
def SUB : IOpInst<"vsub", "...", "dQd", OP_SUB>;

////////////////////////////////////////////////////////////////////////////////
// Multiplication
def MUL     : IOpInst<"vmul", "...", "dQd", OP_MUL>;
def MLA     : IOpInst<"vmla", "....", "dQd", OP_MLA>;
def MLS     : IOpInst<"vmls", "....", "dQd", OP_MLS>;

////////////////////////////////////////////////////////////////////////////////
// Multiplication Extended
def MULX : SInst<"vmulx", "...", "fdQfQd">;

////////////////////////////////////////////////////////////////////////////////
// Division
def FDIV : IOpInst<"vdiv", "...",  "fdQfQd", OP_DIV>;

////////////////////////////////////////////////////////////////////////////////
// Vector fused multiply-add operations
def FMLA : SInst<"vfma", "....", "dQd">;
def FMLS : SOpInst<"vfms", "....", "dQd", OP_FMLS>;
````
- **L813 EN**: Declares TableGen def record `VSTRQ`.
  **L813 CN**: 声明 TableGen def 记录 `VSTRQ`。
- **L814 EN**: Blank line separating nearby declarations or logic blocks.
  **L814 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L815 EN**: Separator comment used for visual grouping.
  **L815 CN**: 用于视觉分组的分隔注释。
- **L816 EN**: Comment explains nearby logic, constraints, or intent: `Addition`.
  **L816 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Addition`。
- **L817 EN**: Declares TableGen def record `ADD`.
  **L817 CN**: 声明 TableGen def 记录 `ADD`。
- **L818 EN**: Blank line separating nearby declarations or logic blocks.
  **L818 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L819 EN**: Separator comment used for visual grouping.
  **L819 CN**: 用于视觉分组的分隔注释。
- **L820 EN**: Comment explains nearby logic, constraints, or intent: `Subtraction`.
  **L820 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Subtraction`。
- **L821 EN**: Declares TableGen def record `SUB`.
  **L821 CN**: 声明 TableGen def 记录 `SUB`。
- **L822 EN**: Blank line separating nearby declarations or logic blocks.
  **L822 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L823 EN**: Separator comment used for visual grouping.
  **L823 CN**: 用于视觉分组的分隔注释。
- **L824 EN**: Comment explains nearby logic, constraints, or intent: `Multiplication`.
  **L824 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Multiplication`。
- **L825 EN**: Declares TableGen def record `MUL`.
  **L825 CN**: 声明 TableGen def 记录 `MUL`。
- **L826 EN**: Declares TableGen def record `MLA`.
  **L826 CN**: 声明 TableGen def 记录 `MLA`。
- **L827 EN**: Declares TableGen def record `MLS`.
  **L827 CN**: 声明 TableGen def 记录 `MLS`。
- **L828 EN**: Blank line separating nearby declarations or logic blocks.
  **L828 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L829 EN**: Separator comment used for visual grouping.
  **L829 CN**: 用于视觉分组的分隔注释。
- **L830 EN**: Comment explains nearby logic, constraints, or intent: `Multiplication Extended`.
  **L830 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Multiplication Extended`。
- **L831 EN**: Declares TableGen def record `MULX`.
  **L831 CN**: 声明 TableGen def 记录 `MULX`。
- **L832 EN**: Blank line separating nearby declarations or logic blocks.
  **L832 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L833 EN**: Separator comment used for visual grouping.
  **L833 CN**: 用于视觉分组的分隔注释。
- **L834 EN**: Comment explains nearby logic, constraints, or intent: `Division`.
  **L834 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Division`。
- **L835 EN**: Declares TableGen def record `FDIV`.
  **L835 CN**: 声明 TableGen def 记录 `FDIV`。
- **L836 EN**: Blank line separating nearby declarations or logic blocks.
  **L836 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L837 EN**: Separator comment used for visual grouping.
  **L837 CN**: 用于视觉分组的分隔注释。
- **L838 EN**: Comment explains nearby logic, constraints, or intent: `Vector fused multiply-add operations`.
  **L838 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Vector fused multiply-add operations`。
- **L839 EN**: Declares TableGen def record `FMLA`.
  **L839 CN**: 声明 TableGen def 记录 `FMLA`。
- **L840 EN**: Declares TableGen def record `FMLS`.
  **L840 CN**: 声明 TableGen def 记录 `FMLS`。

### Lines 841-868

````tablegen

////////////////////////////////////////////////////////////////////////////////
// MUL, MLA, MLS, FMA, FMS definitions with scalar argument
def VMUL_N_A64 : IOpInst<"vmul_n", "..1", "Qd", OP_MUL_N>;

def FMLA_N : SOpInst<"vfma_n", "...1", "dQd", OP_FMLA_N>;
def FMLS_N : SOpInst<"vfms_n", "...1", "fdQfQd", OP_FMLS_N>;

////////////////////////////////////////////////////////////////////////////////
// Logical operations
def BSL : SInst<"vbsl", ".U..", "dPlQdQPl">;

////////////////////////////////////////////////////////////////////////////////
// Absolute Difference
def ABD  : SInst<"vabd", "...",  "dQd">;

////////////////////////////////////////////////////////////////////////////////
// saturating absolute/negate
def ABS    : SInst<"vabs", "..", "dQdlQl">;
def QABS   : SInst<"vqabs", "..", "lQl">;
def NEG    : SOpInst<"vneg", "..", "dlQdQl", OP_NEG>;
def QNEG   : SInst<"vqneg", "..", "lQl">;

////////////////////////////////////////////////////////////////////////////////
// Signed Saturating Accumulated of Unsigned Value
def SUQADD : SInst<"vuqadd", "..U", "csilQcQsQiQl">;

////////////////////////////////////////////////////////////////////////////////
````
- **L841 EN**: Blank line separating nearby declarations or logic blocks.
  **L841 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L842 EN**: Separator comment used for visual grouping.
  **L842 CN**: 用于视觉分组的分隔注释。
- **L843 EN**: Comment explains nearby logic, constraints, or intent: `MUL, MLA, MLS, FMA, FMS definitions with scalar argument`.
  **L843 CN**: 注释解释附近代码的逻辑、约束或设计意图：`MUL, MLA, MLS, FMA, FMS definitions with scalar argument`。
- **L844 EN**: Declares TableGen def record `VMUL_N_A64`.
  **L844 CN**: 声明 TableGen def 记录 `VMUL_N_A64`。
- **L845 EN**: Blank line separating nearby declarations or logic blocks.
  **L845 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L846 EN**: Declares TableGen def record `FMLA_N`.
  **L846 CN**: 声明 TableGen def 记录 `FMLA_N`。
- **L847 EN**: Declares TableGen def record `FMLS_N`.
  **L847 CN**: 声明 TableGen def 记录 `FMLS_N`。
- **L848 EN**: Blank line separating nearby declarations or logic blocks.
  **L848 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L849 EN**: Separator comment used for visual grouping.
  **L849 CN**: 用于视觉分组的分隔注释。
- **L850 EN**: Comment explains nearby logic, constraints, or intent: `Logical operations`.
  **L850 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Logical operations`。
- **L851 EN**: Declares TableGen def record `BSL`.
  **L851 CN**: 声明 TableGen def 记录 `BSL`。
- **L852 EN**: Blank line separating nearby declarations or logic blocks.
  **L852 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L853 EN**: Separator comment used for visual grouping.
  **L853 CN**: 用于视觉分组的分隔注释。
- **L854 EN**: Comment explains nearby logic, constraints, or intent: `Absolute Difference`.
  **L854 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Absolute Difference`。
- **L855 EN**: Declares TableGen def record `ABD`.
  **L855 CN**: 声明 TableGen def 记录 `ABD`。
- **L856 EN**: Blank line separating nearby declarations or logic blocks.
  **L856 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L857 EN**: Separator comment used for visual grouping.
  **L857 CN**: 用于视觉分组的分隔注释。
- **L858 EN**: Comment explains nearby logic, constraints, or intent: `saturating absolute/negate`.
  **L858 CN**: 注释解释附近代码的逻辑、约束或设计意图：`saturating absolute/negate`。
- **L859 EN**: Declares TableGen def record `ABS`.
  **L859 CN**: 声明 TableGen def 记录 `ABS`。
- **L860 EN**: Declares TableGen def record `QABS`.
  **L860 CN**: 声明 TableGen def 记录 `QABS`。
- **L861 EN**: Declares TableGen def record `NEG`.
  **L861 CN**: 声明 TableGen def 记录 `NEG`。
- **L862 EN**: Declares TableGen def record `QNEG`.
  **L862 CN**: 声明 TableGen def 记录 `QNEG`。
- **L863 EN**: Blank line separating nearby declarations or logic blocks.
  **L863 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L864 EN**: Separator comment used for visual grouping.
  **L864 CN**: 用于视觉分组的分隔注释。
- **L865 EN**: Comment explains nearby logic, constraints, or intent: `Signed Saturating Accumulated of Unsigned Value`.
  **L865 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Signed Saturating Accumulated of Unsigned Value`。
- **L866 EN**: Declares TableGen def record `SUQADD`.
  **L866 CN**: 声明 TableGen def 记录 `SUQADD`。
- **L867 EN**: Blank line separating nearby declarations or logic blocks.
  **L867 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L868 EN**: Separator comment used for visual grouping.
  **L868 CN**: 用于视觉分组的分隔注释。

### Lines 869-896

````tablegen
// Unsigned Saturating Accumulated of Signed Value
def USQADD : SInst<"vsqadd", "..S", "UcUsUiUlQUcQUsQUiQUl">;

////////////////////////////////////////////////////////////////////////////////
// Reciprocal/Sqrt
def FRECPS  : IInst<"vrecps", "...", "dQd">;
def FRSQRTS : IInst<"vrsqrts", "...", "dQd">;
def FRECPE  : SInst<"vrecpe", "..", "dQd">;
def FRSQRTE : SInst<"vrsqrte", "..", "dQd">;
def FSQRT   : SInst<"vsqrt", "..", "fdQfQd">;

////////////////////////////////////////////////////////////////////////////////
// bitwise reverse
def RBIT : IInst<"vrbit", "..", "cUcPcQcQUcQPc">;

////////////////////////////////////////////////////////////////////////////////
// Integer extract and narrow to high
def XTN2 : SOpInst<"vmovn_high", "(<Q)<Q", "silUsUiUl", OP_XTN>;

////////////////////////////////////////////////////////////////////////////////
// Signed integer saturating extract and unsigned narrow to high
def SQXTUN2 : SOpInst<"vqmovun_high", "(<U)(<Uq).", "HsHiHl", OP_SQXTUN>;

////////////////////////////////////////////////////////////////////////////////
// Integer saturating extract and narrow to high
def QXTN2 : SOpInst<"vqmovn_high", "(<Q)<Q", "silUsUiUl", OP_QXTN>;

////////////////////////////////////////////////////////////////////////////////
````
- **L869 EN**: Comment explains nearby logic, constraints, or intent: `Unsigned Saturating Accumulated of Signed Value`.
  **L869 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Unsigned Saturating Accumulated of Signed Value`。
- **L870 EN**: Declares TableGen def record `USQADD`.
  **L870 CN**: 声明 TableGen def 记录 `USQADD`。
- **L871 EN**: Blank line separating nearby declarations or logic blocks.
  **L871 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L872 EN**: Separator comment used for visual grouping.
  **L872 CN**: 用于视觉分组的分隔注释。
- **L873 EN**: Comment explains nearby logic, constraints, or intent: `Reciprocal/Sqrt`.
  **L873 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Reciprocal/Sqrt`。
- **L874 EN**: Declares TableGen def record `FRECPS`.
  **L874 CN**: 声明 TableGen def 记录 `FRECPS`。
- **L875 EN**: Declares TableGen def record `FRSQRTS`.
  **L875 CN**: 声明 TableGen def 记录 `FRSQRTS`。
- **L876 EN**: Declares TableGen def record `FRECPE`.
  **L876 CN**: 声明 TableGen def 记录 `FRECPE`。
- **L877 EN**: Declares TableGen def record `FRSQRTE`.
  **L877 CN**: 声明 TableGen def 记录 `FRSQRTE`。
- **L878 EN**: Declares TableGen def record `FSQRT`.
  **L878 CN**: 声明 TableGen def 记录 `FSQRT`。
- **L879 EN**: Blank line separating nearby declarations or logic blocks.
  **L879 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L880 EN**: Separator comment used for visual grouping.
  **L880 CN**: 用于视觉分组的分隔注释。
- **L881 EN**: Comment explains nearby logic, constraints, or intent: `bitwise reverse`.
  **L881 CN**: 注释解释附近代码的逻辑、约束或设计意图：`bitwise reverse`。
- **L882 EN**: Declares TableGen def record `RBIT`.
  **L882 CN**: 声明 TableGen def 记录 `RBIT`。
- **L883 EN**: Blank line separating nearby declarations or logic blocks.
  **L883 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L884 EN**: Separator comment used for visual grouping.
  **L884 CN**: 用于视觉分组的分隔注释。
- **L885 EN**: Comment explains nearby logic, constraints, or intent: `Integer extract and narrow to high`.
  **L885 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Integer extract and narrow to high`。
- **L886 EN**: Declares TableGen def record `XTN2`.
  **L886 CN**: 声明 TableGen def 记录 `XTN2`。
- **L887 EN**: Blank line separating nearby declarations or logic blocks.
  **L887 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L888 EN**: Separator comment used for visual grouping.
  **L888 CN**: 用于视觉分组的分隔注释。
- **L889 EN**: Comment explains nearby logic, constraints, or intent: `Signed integer saturating extract and unsigned narrow to high`.
  **L889 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Signed integer saturating extract and unsigned narrow to high`。
- **L890 EN**: Declares TableGen def record `SQXTUN2`.
  **L890 CN**: 声明 TableGen def 记录 `SQXTUN2`。
- **L891 EN**: Blank line separating nearby declarations or logic blocks.
  **L891 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L892 EN**: Separator comment used for visual grouping.
  **L892 CN**: 用于视觉分组的分隔注释。
- **L893 EN**: Comment explains nearby logic, constraints, or intent: `Integer saturating extract and narrow to high`.
  **L893 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Integer saturating extract and narrow to high`。
- **L894 EN**: Declares TableGen def record `QXTN2`.
  **L894 CN**: 声明 TableGen def 记录 `QXTN2`。
- **L895 EN**: Blank line separating nearby declarations or logic blocks.
  **L895 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L896 EN**: Separator comment used for visual grouping.
  **L896 CN**: 用于视觉分组的分隔注释。

### Lines 897-924

````tablegen
// Converting vectors

def VCVT_F32_F64 : SInst<"vcvt_f32_f64", "(<q).", "Qd">;
def VCVT_F64_F32 : SInst<"vcvt_f64_f32", "(>Q).", "f">;

def VCVT_S64 : SInst<"vcvt_s64", "S.",  "dQd">;
def VCVT_U64 : SInst<"vcvt_u64", "U.",  "dQd">;
def VCVT_F64 : SInst<"vcvt_f64", "F(.!)",  "lUlQlQUl">;

def VCVT_HIGH_F16_F32 : SOpInst<"vcvt_high_f16", "<(<q!)Q", "Hf", OP_VCVT_NA_HI_F16>;
def VCVT_HIGH_F32_F16 : SOpInst<"vcvt_high_f32", "(>Q)(Q!)", "h", OP_VCVT_EX_HI_F32>;
def VCVT_HIGH_F32_F64 : SOpInst<"vcvt_high_f32", "(<Q)(F<!)Q", "d", OP_VCVT_NA_HI_F32>;
def VCVT_HIGH_F64_F32 : SOpInst<"vcvt_high_f64", "(>Q)(Q!)", "f", OP_VCVT_EX_HI_F64>;

def VCVTX_F32_F64      : SInst<"vcvtx_f32", "(F<)(Q!)",  "d">;
def VCVTX_HIGH_F32_F64 : SOpInst<"vcvtx_high_f32", "(<Q)(F<!)Q", "d", OP_VCVTX_HI>;

////////////////////////////////////////////////////////////////////////////////
// Comparison
def FCAGE : IInst<"vcage", "U..", "dQd">;
def FCAGT : IInst<"vcagt", "U..", "dQd">;
def FCALE : IInst<"vcale", "U..", "dQd">;
def FCALT : IInst<"vcalt", "U..", "dQd">;
def CMTST  : WInst<"vtst", "U..", "lUlPlQlQUlQPl">;
def CFMEQ  : SOpInst<"vceq", "U..", "lUldQdQlQUlPlQPl", OP_EQ>;
def CFMGE  : SOpInst<"vcge", "U..", "lUldQdQlQUl", OP_GE>;
def CFMLE  : SOpInst<"vcle", "U..", "lUldQdQlQUl", OP_LE>;
def CFMGT  : SOpInst<"vcgt", "U..", "lUldQdQlQUl", OP_GT>;
````
- **L897 EN**: Comment explains nearby logic, constraints, or intent: `Converting vectors`.
  **L897 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Converting vectors`。
- **L898 EN**: Blank line separating nearby declarations or logic blocks.
  **L898 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L899 EN**: Declares TableGen def record `VCVT_F32_F64`.
  **L899 CN**: 声明 TableGen def 记录 `VCVT_F32_F64`。
- **L900 EN**: Declares TableGen def record `VCVT_F64_F32`.
  **L900 CN**: 声明 TableGen def 记录 `VCVT_F64_F32`。
- **L901 EN**: Blank line separating nearby declarations or logic blocks.
  **L901 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L902 EN**: Declares TableGen def record `VCVT_S64`.
  **L902 CN**: 声明 TableGen def 记录 `VCVT_S64`。
- **L903 EN**: Declares TableGen def record `VCVT_U64`.
  **L903 CN**: 声明 TableGen def 记录 `VCVT_U64`。
- **L904 EN**: Declares TableGen def record `VCVT_F64`.
  **L904 CN**: 声明 TableGen def 记录 `VCVT_F64`。
- **L905 EN**: Blank line separating nearby declarations or logic blocks.
  **L905 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L906 EN**: Declares TableGen def record `VCVT_HIGH_F16_F32`.
  **L906 CN**: 声明 TableGen def 记录 `VCVT_HIGH_F16_F32`。
- **L907 EN**: Declares TableGen def record `VCVT_HIGH_F32_F16`.
  **L907 CN**: 声明 TableGen def 记录 `VCVT_HIGH_F32_F16`。
- **L908 EN**: Declares TableGen def record `VCVT_HIGH_F32_F64`.
  **L908 CN**: 声明 TableGen def 记录 `VCVT_HIGH_F32_F64`。
- **L909 EN**: Declares TableGen def record `VCVT_HIGH_F64_F32`.
  **L909 CN**: 声明 TableGen def 记录 `VCVT_HIGH_F64_F32`。
- **L910 EN**: Blank line separating nearby declarations or logic blocks.
  **L910 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L911 EN**: Declares TableGen def record `VCVTX_F32_F64`.
  **L911 CN**: 声明 TableGen def 记录 `VCVTX_F32_F64`。
- **L912 EN**: Declares TableGen def record `VCVTX_HIGH_F32_F64`.
  **L912 CN**: 声明 TableGen def 记录 `VCVTX_HIGH_F32_F64`。
- **L913 EN**: Blank line separating nearby declarations or logic blocks.
  **L913 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L914 EN**: Separator comment used for visual grouping.
  **L914 CN**: 用于视觉分组的分隔注释。
- **L915 EN**: Comment explains nearby logic, constraints, or intent: `Comparison`.
  **L915 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Comparison`。
- **L916 EN**: Declares TableGen def record `FCAGE`.
  **L916 CN**: 声明 TableGen def 记录 `FCAGE`。
- **L917 EN**: Declares TableGen def record `FCAGT`.
  **L917 CN**: 声明 TableGen def 记录 `FCAGT`。
- **L918 EN**: Declares TableGen def record `FCALE`.
  **L918 CN**: 声明 TableGen def 记录 `FCALE`。
- **L919 EN**: Declares TableGen def record `FCALT`.
  **L919 CN**: 声明 TableGen def 记录 `FCALT`。
- **L920 EN**: Declares TableGen def record `CMTST`.
  **L920 CN**: 声明 TableGen def 记录 `CMTST`。
- **L921 EN**: Declares TableGen def record `CFMEQ`.
  **L921 CN**: 声明 TableGen def 记录 `CFMEQ`。
- **L922 EN**: Declares TableGen def record `CFMGE`.
  **L922 CN**: 声明 TableGen def 记录 `CFMGE`。
- **L923 EN**: Declares TableGen def record `CFMLE`.
  **L923 CN**: 声明 TableGen def 记录 `CFMLE`。
- **L924 EN**: Declares TableGen def record `CFMGT`.
  **L924 CN**: 声明 TableGen def 记录 `CFMGT`。

### Lines 925-952

````tablegen
def CFMLT  : SOpInst<"vclt", "U..", "lUldQdQlQUl", OP_LT>;

def CMEQ  : SInst<"vceqz", "U(.!)",
                  "csilfUcUsUiUlPcPlQcQsQiQlQfQUcQUsQUiQUlQPcdQdQPl">;
def CMGE  : SInst<"vcgez", "U(.!)", "csilfdQcQsQiQlQfQd">;
def CMLE  : SInst<"vclez", "U(.!)", "csilfdQcQsQiQlQfQd">;
def CMGT  : SInst<"vcgtz", "U(.!)", "csilfdQcQsQiQlQfQd">;
def CMLT  : SInst<"vcltz", "U(.!)", "csilfdQcQsQiQlQfQd">;

////////////////////////////////////////////////////////////////////////////////
// Max/Min Integer
def MAX : SInst<"vmax", "...", "dQd">;
def MIN : SInst<"vmin", "...", "dQd">;

////////////////////////////////////////////////////////////////////////////////
// Pairwise Max/Min
def MAXP : SInst<"vpmax", "...", "QcQsQiQUcQUsQUiQfQd">;
def MINP : SInst<"vpmin", "...", "QcQsQiQUcQUsQUiQfQd">;

////////////////////////////////////////////////////////////////////////////////
// Pairwise MaxNum/MinNum Floating Point
def FMAXNMP : SInst<"vpmaxnm", "...", "fQfQd">;
def FMINNMP : SInst<"vpminnm", "...", "fQfQd">;

////////////////////////////////////////////////////////////////////////////////
// Pairwise Addition
def ADDP  : IInst<"vpadd", "...", "QcQsQiQlQUcQUsQUiQUlQfQd">;

````
- **L925 EN**: Declares TableGen def record `CFMLT`.
  **L925 CN**: 声明 TableGen def 记录 `CFMLT`。
- **L926 EN**: Blank line separating nearby declarations or logic blocks.
  **L926 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L927 EN**: Declares TableGen def record `CMEQ`.
  **L927 CN**: 声明 TableGen def 记录 `CMEQ`。
- **L928 EN**: Adds a standalone statement or declaration: `"csilfUcUsUiUlPcPlQcQsQiQlQfQUcQUsQUiQUlQPcdQdQPl">;`.
  **L928 CN**: 添加一条独立语句或声明：`"csilfUcUsUiUlPcPlQcQsQiQlQfQUcQUsQUiQUlQPcdQdQPl">;`。
- **L929 EN**: Declares TableGen def record `CMGE`.
  **L929 CN**: 声明 TableGen def 记录 `CMGE`。
- **L930 EN**: Declares TableGen def record `CMLE`.
  **L930 CN**: 声明 TableGen def 记录 `CMLE`。
- **L931 EN**: Declares TableGen def record `CMGT`.
  **L931 CN**: 声明 TableGen def 记录 `CMGT`。
- **L932 EN**: Declares TableGen def record `CMLT`.
  **L932 CN**: 声明 TableGen def 记录 `CMLT`。
- **L933 EN**: Blank line separating nearby declarations or logic blocks.
  **L933 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L934 EN**: Separator comment used for visual grouping.
  **L934 CN**: 用于视觉分组的分隔注释。
- **L935 EN**: Comment explains nearby logic, constraints, or intent: `Max/Min Integer`.
  **L935 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Max/Min Integer`。
- **L936 EN**: Declares TableGen def record `MAX`.
  **L936 CN**: 声明 TableGen def 记录 `MAX`。
- **L937 EN**: Declares TableGen def record `MIN`.
  **L937 CN**: 声明 TableGen def 记录 `MIN`。
- **L938 EN**: Blank line separating nearby declarations or logic blocks.
  **L938 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L939 EN**: Separator comment used for visual grouping.
  **L939 CN**: 用于视觉分组的分隔注释。
- **L940 EN**: Comment explains nearby logic, constraints, or intent: `Pairwise Max/Min`.
  **L940 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Pairwise Max/Min`。
- **L941 EN**: Declares TableGen def record `MAXP`.
  **L941 CN**: 声明 TableGen def 记录 `MAXP`。
- **L942 EN**: Declares TableGen def record `MINP`.
  **L942 CN**: 声明 TableGen def 记录 `MINP`。
- **L943 EN**: Blank line separating nearby declarations or logic blocks.
  **L943 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L944 EN**: Separator comment used for visual grouping.
  **L944 CN**: 用于视觉分组的分隔注释。
- **L945 EN**: Comment explains nearby logic, constraints, or intent: `Pairwise MaxNum/MinNum Floating Point`.
  **L945 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Pairwise MaxNum/MinNum Floating Point`。
- **L946 EN**: Declares TableGen def record `FMAXNMP`.
  **L946 CN**: 声明 TableGen def 记录 `FMAXNMP`。
- **L947 EN**: Declares TableGen def record `FMINNMP`.
  **L947 CN**: 声明 TableGen def 记录 `FMINNMP`。
- **L948 EN**: Blank line separating nearby declarations or logic blocks.
  **L948 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L949 EN**: Separator comment used for visual grouping.
  **L949 CN**: 用于视觉分组的分隔注释。
- **L950 EN**: Comment explains nearby logic, constraints, or intent: `Pairwise Addition`.
  **L950 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Pairwise Addition`。
- **L951 EN**: Declares TableGen def record `ADDP`.
  **L951 CN**: 声明 TableGen def 记录 `ADDP`。
- **L952 EN**: Blank line separating nearby declarations or logic blocks.
  **L952 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 953-980

````tablegen
////////////////////////////////////////////////////////////////////////////////
// Shifts by constant
let isShift = 1 in {
// Left shift long high
def SHLL_HIGH_N    : SOpInst<"vshll_high_n", ">.I", "HcHsHiHUcHUsHUi",
                             OP_LONG_HI>;

////////////////////////////////////////////////////////////////////////////////
def SRI_N : WInst<"vsri_n", "...I", "PlQPl", [ImmCheck<2, ImmCheckShiftRight, 0>]>;
def SLI_N : WInst<"vsli_n", "...I", "PlQPl", [ImmCheck<2, ImmCheckShiftLeft, 0>]>;

// Right shift narrow high
def SHRN_HIGH_N    : IOpInst<"vshrn_high_n", "<(<q).I",
                             "HsHiHlHUsHUiHUl", OP_NARROW_HI>;
def QSHRUN_HIGH_N  : SOpInst<"vqshrun_high_n", "(<U)(<Uq).I",
                             "HsHiHl", OP_NARROW_HI>;
def RSHRN_HIGH_N   : IOpInst<"vrshrn_high_n", "<(<q).I",
                             "HsHiHlHUsHUiHUl", OP_NARROW_HI>;
def QRSHRUN_HIGH_N : SOpInst<"vqrshrun_high_n", "(<U)(<Uq).I",
                             "HsHiHl", OP_NARROW_HI>;
def QSHRN_HIGH_N   : SOpInst<"vqshrn_high_n", "<(<q).I",
                             "HsHiHlHUsHUiHUl", OP_NARROW_HI>;
def QRSHRN_HIGH_N  : SOpInst<"vqrshrn_high_n", "<(<q).I",
                             "HsHiHlHUsHUiHUl", OP_NARROW_HI>;
}

////////////////////////////////////////////////////////////////////////////////
// Converting vectors
````
- **L953 EN**: Separator comment used for visual grouping.
  **L953 CN**: 用于视觉分组的分隔注释。
- **L954 EN**: Comment explains nearby logic, constraints, or intent: `Shifts by constant`.
  **L954 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Shifts by constant`。
- **L955 EN**: Assigns a TableGen property that affects following records or inherited fields: `let isShift = 1 in {`.
  **L955 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let isShift = 1 in {`。
- **L956 EN**: Comment explains nearby logic, constraints, or intent: `Left shift long high`.
  **L956 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Left shift long high`。
- **L957 EN**: Declares TableGen def record `SHLL_HIGH_N`.
  **L957 CN**: 声明 TableGen def 记录 `SHLL_HIGH_N`。
- **L958 EN**: Adds a standalone statement or declaration: `OP_LONG_HI>;`.
  **L958 CN**: 添加一条独立语句或声明：`OP_LONG_HI>;`。
- **L959 EN**: Blank line separating nearby declarations or logic blocks.
  **L959 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L960 EN**: Separator comment used for visual grouping.
  **L960 CN**: 用于视觉分组的分隔注释。
- **L961 EN**: Declares TableGen def record `SRI_N`.
  **L961 CN**: 声明 TableGen def 记录 `SRI_N`。
- **L962 EN**: Declares TableGen def record `SLI_N`.
  **L962 CN**: 声明 TableGen def 记录 `SLI_N`。
- **L963 EN**: Blank line separating nearby declarations or logic blocks.
  **L963 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L964 EN**: Comment explains nearby logic, constraints, or intent: `Right shift narrow high`.
  **L964 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Right shift narrow high`。
- **L965 EN**: Declares TableGen def record `SHRN_HIGH_N`.
  **L965 CN**: 声明 TableGen def 记录 `SHRN_HIGH_N`。
- **L966 EN**: Adds a standalone statement or declaration: `"HsHiHlHUsHUiHUl", OP_NARROW_HI>;`.
  **L966 CN**: 添加一条独立语句或声明：`"HsHiHlHUsHUiHUl", OP_NARROW_HI>;`。
- **L967 EN**: Declares TableGen def record `QSHRUN_HIGH_N`.
  **L967 CN**: 声明 TableGen def 记录 `QSHRUN_HIGH_N`。
- **L968 EN**: Adds a standalone statement or declaration: `"HsHiHl", OP_NARROW_HI>;`.
  **L968 CN**: 添加一条独立语句或声明：`"HsHiHl", OP_NARROW_HI>;`。
- **L969 EN**: Declares TableGen def record `RSHRN_HIGH_N`.
  **L969 CN**: 声明 TableGen def 记录 `RSHRN_HIGH_N`。
- **L970 EN**: Adds a standalone statement or declaration: `"HsHiHlHUsHUiHUl", OP_NARROW_HI>;`.
  **L970 CN**: 添加一条独立语句或声明：`"HsHiHlHUsHUiHUl", OP_NARROW_HI>;`。
- **L971 EN**: Declares TableGen def record `QRSHRUN_HIGH_N`.
  **L971 CN**: 声明 TableGen def 记录 `QRSHRUN_HIGH_N`。
- **L972 EN**: Adds a standalone statement or declaration: `"HsHiHl", OP_NARROW_HI>;`.
  **L972 CN**: 添加一条独立语句或声明：`"HsHiHl", OP_NARROW_HI>;`。
- **L973 EN**: Declares TableGen def record `QSHRN_HIGH_N`.
  **L973 CN**: 声明 TableGen def 记录 `QSHRN_HIGH_N`。
- **L974 EN**: Adds a standalone statement or declaration: `"HsHiHlHUsHUiHUl", OP_NARROW_HI>;`.
  **L974 CN**: 添加一条独立语句或声明：`"HsHiHlHUsHUiHUl", OP_NARROW_HI>;`。
- **L975 EN**: Declares TableGen def record `QRSHRN_HIGH_N`.
  **L975 CN**: 声明 TableGen def 记录 `QRSHRN_HIGH_N`。
- **L976 EN**: Adds a standalone statement or declaration: `"HsHiHlHUsHUiHUl", OP_NARROW_HI>;`.
  **L976 CN**: 添加一条独立语句或声明：`"HsHiHlHUsHUiHUl", OP_NARROW_HI>;`。
- **L977 EN**: Closes the current lexical scope or compound statement.
  **L977 CN**: 结束当前词法作用域或复合语句块。
- **L978 EN**: Blank line separating nearby declarations or logic blocks.
  **L978 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L979 EN**: Separator comment used for visual grouping.
  **L979 CN**: 用于视觉分组的分隔注释。
- **L980 EN**: Comment explains nearby logic, constraints, or intent: `Converting vectors`.
  **L980 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Converting vectors`。

### Lines 981-1008

````tablegen
def VMOVL_HIGH   : SOpInst<"vmovl_high", ">.", "HcHsHiHUcHUsHUi", OP_MOVL_HI>;

def CVTF_N_F64   : SInst<"vcvt_n_f64", "F(.!)I", "lUlQlQUl",
                        [ImmCheck<1, ImmCheck1_64>]>;
def FCVTZS_N_S64 : SInst<"vcvt_n_s64", "S.I", "dQd",
                        [ImmCheck<1, ImmCheck1_64>]>;
def FCVTZS_N_U64 : SInst<"vcvt_n_u64", "U.I", "dQd",
                        [ImmCheck<1, ImmCheck1_64>]>;

////////////////////////////////////////////////////////////////////////////////
// 3VDiff class using high 64-bit in operands
def VADDL_HIGH   : SOpInst<"vaddl_high", "(>Q)QQ", "csiUcUsUi", OP_ADDLHi>;
def VADDW_HIGH   : SOpInst<"vaddw_high", "(>Q)(>Q)Q", "csiUcUsUi", OP_ADDWHi>;
def VSUBL_HIGH   : SOpInst<"vsubl_high", "(>Q)QQ", "csiUcUsUi", OP_SUBLHi>;
def VSUBW_HIGH   : SOpInst<"vsubw_high", "(>Q)(>Q)Q", "csiUcUsUi", OP_SUBWHi>;

def VABDL_HIGH   : SOpInst<"vabdl_high", "(>Q)QQ",  "csiUcUsUi", OP_ABDLHi>;
def VABAL_HIGH   : SOpInst<"vabal_high", "(>Q)(>Q)QQ", "csiUcUsUi", OP_ABALHi>;

def VMULL_HIGH   : SOpInst<"vmull_high", "(>Q)QQ", "csiUcUsUiPc", OP_MULLHi>;
def VMULL_HIGH_N : SOpInst<"vmull_high_n", "(>Q)Q1", "siUsUi", OP_MULLHi_N>;
def VMLAL_HIGH   : SOpInst<"vmlal_high", "(>Q)(>Q)QQ", "csiUcUsUi", OP_MLALHi>;
def VMLAL_HIGH_N : SOpInst<"vmlal_high_n", "(>Q)(>Q)Q1", "siUsUi", OP_MLALHi_N>;
def VMLSL_HIGH   : SOpInst<"vmlsl_high", "(>Q)(>Q)QQ", "csiUcUsUi", OP_MLSLHi>;
def VMLSL_HIGH_N : SOpInst<"vmlsl_high_n", "(>Q)(>Q)Q1", "siUsUi", OP_MLSLHi_N>;

def VADDHN_HIGH  : SOpInst<"vaddhn_high", "(<Q)<QQ", "silUsUiUl", OP_ADDHNHi>;
def VRADDHN_HIGH : SOpInst<"vraddhn_high", "(<Q)<QQ", "silUsUiUl", OP_RADDHNHi>;
````
- **L981 EN**: Declares TableGen def record `VMOVL_HIGH`.
  **L981 CN**: 声明 TableGen def 记录 `VMOVL_HIGH`。
- **L982 EN**: Blank line separating nearby declarations or logic blocks.
  **L982 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L983 EN**: Declares TableGen def record `CVTF_N_F64`.
  **L983 CN**: 声明 TableGen def 记录 `CVTF_N_F64`。
- **L984 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheck1_64>]>;`.
  **L984 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheck1_64>]>;`。
- **L985 EN**: Declares TableGen def record `FCVTZS_N_S64`.
  **L985 CN**: 声明 TableGen def 记录 `FCVTZS_N_S64`。
- **L986 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheck1_64>]>;`.
  **L986 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheck1_64>]>;`。
- **L987 EN**: Declares TableGen def record `FCVTZS_N_U64`.
  **L987 CN**: 声明 TableGen def 记录 `FCVTZS_N_U64`。
- **L988 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheck1_64>]>;`.
  **L988 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheck1_64>]>;`。
- **L989 EN**: Blank line separating nearby declarations or logic blocks.
  **L989 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L990 EN**: Separator comment used for visual grouping.
  **L990 CN**: 用于视觉分组的分隔注释。
- **L991 EN**: Comment explains nearby logic, constraints, or intent: `3VDiff class using high 64-bit in operands`.
  **L991 CN**: 注释解释附近代码的逻辑、约束或设计意图：`3VDiff class using high 64-bit in operands`。
- **L992 EN**: Declares TableGen def record `VADDL_HIGH`.
  **L992 CN**: 声明 TableGen def 记录 `VADDL_HIGH`。
- **L993 EN**: Declares TableGen def record `VADDW_HIGH`.
  **L993 CN**: 声明 TableGen def 记录 `VADDW_HIGH`。
- **L994 EN**: Declares TableGen def record `VSUBL_HIGH`.
  **L994 CN**: 声明 TableGen def 记录 `VSUBL_HIGH`。
- **L995 EN**: Declares TableGen def record `VSUBW_HIGH`.
  **L995 CN**: 声明 TableGen def 记录 `VSUBW_HIGH`。
- **L996 EN**: Blank line separating nearby declarations or logic blocks.
  **L996 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L997 EN**: Declares TableGen def record `VABDL_HIGH`.
  **L997 CN**: 声明 TableGen def 记录 `VABDL_HIGH`。
- **L998 EN**: Declares TableGen def record `VABAL_HIGH`.
  **L998 CN**: 声明 TableGen def 记录 `VABAL_HIGH`。
- **L999 EN**: Blank line separating nearby declarations or logic blocks.
  **L999 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1000 EN**: Declares TableGen def record `VMULL_HIGH`.
  **L1000 CN**: 声明 TableGen def 记录 `VMULL_HIGH`。
- **L1001 EN**: Declares TableGen def record `VMULL_HIGH_N`.
  **L1001 CN**: 声明 TableGen def 记录 `VMULL_HIGH_N`。
- **L1002 EN**: Declares TableGen def record `VMLAL_HIGH`.
  **L1002 CN**: 声明 TableGen def 记录 `VMLAL_HIGH`。
- **L1003 EN**: Declares TableGen def record `VMLAL_HIGH_N`.
  **L1003 CN**: 声明 TableGen def 记录 `VMLAL_HIGH_N`。
- **L1004 EN**: Declares TableGen def record `VMLSL_HIGH`.
  **L1004 CN**: 声明 TableGen def 记录 `VMLSL_HIGH`。
- **L1005 EN**: Declares TableGen def record `VMLSL_HIGH_N`.
  **L1005 CN**: 声明 TableGen def 记录 `VMLSL_HIGH_N`。
- **L1006 EN**: Blank line separating nearby declarations or logic blocks.
  **L1006 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1007 EN**: Declares TableGen def record `VADDHN_HIGH`.
  **L1007 CN**: 声明 TableGen def 记录 `VADDHN_HIGH`。
- **L1008 EN**: Declares TableGen def record `VRADDHN_HIGH`.
  **L1008 CN**: 声明 TableGen def 记录 `VRADDHN_HIGH`。

### Lines 1009-1036

````tablegen
def VSUBHN_HIGH  : SOpInst<"vsubhn_high", "(<Q)<QQ", "silUsUiUl", OP_SUBHNHi>;
def VRSUBHN_HIGH : SOpInst<"vrsubhn_high", "(<Q)<QQ", "silUsUiUl", OP_RSUBHNHi>;

def VQDMULL_HIGH : SOpInst<"vqdmull_high", "(>Q)QQ", "si", OP_QDMULLHi>;
def VQDMULL_HIGH_N : SOpInst<"vqdmull_high_n", "(>Q)Q1", "si", OP_QDMULLHi_N>;
def VQDMLAL_HIGH : SOpInst<"vqdmlal_high", "(>Q)(>Q)QQ", "si", OP_QDMLALHi>;
def VQDMLAL_HIGH_N : SOpInst<"vqdmlal_high_n", "(>Q)(>Q)Q1", "si", OP_QDMLALHi_N>;
def VQDMLSL_HIGH : SOpInst<"vqdmlsl_high", "(>Q)(>Q)QQ", "si", OP_QDMLSLHi>;
def VQDMLSL_HIGH_N : SOpInst<"vqdmlsl_high_n", "(>Q)(>Q)Q1", "si", OP_QDMLSLHi_N>;
let TargetGuard = "aes,neon" in {
  def VMULL_P64    : SInst<"vmull", "(1>)11", "Pl">;
  def VMULL_HIGH_P64 : SOpInst<"vmull_high", "(1>)..", "HPl", OP_MULLHi_P64>;
}


////////////////////////////////////////////////////////////////////////////////
// Extract or insert element from vector
def GET_LANE : IInst<"vget_lane", "1.I", "dQdPlQPl",
                      [ImmCheck<1, ImmCheckLaneIndex, 0>]>;
def SET_LANE : IInst<"vset_lane", ".1.I", "dQdPlQPl",
                      [ImmCheck<2, ImmCheckLaneIndex, 1>]>;
def COPY_LANE : IOpInst<"vcopy_lane", "..I.I",
                        "csilUcUsUiUlPcPsPlfdm", OP_COPY_LN>;
def COPYQ_LANE : IOpInst<"vcopy_lane", "..IqI",
                        "QcQsQiQlQUcQUsQUiQUlQPcQPsQfQdQPlQm", OP_COPY_LN>;
def COPY_LANEQ : IOpInst<"vcopy_laneq", "..IQI",
                     "csilPcPsPlUcUsUiUlfdm", OP_COPY_LN>;
def COPYQ_LANEQ : IOpInst<"vcopy_laneq", "..I.I",
````
- **L1009 EN**: Declares TableGen def record `VSUBHN_HIGH`.
  **L1009 CN**: 声明 TableGen def 记录 `VSUBHN_HIGH`。
- **L1010 EN**: Declares TableGen def record `VRSUBHN_HIGH`.
  **L1010 CN**: 声明 TableGen def 记录 `VRSUBHN_HIGH`。
- **L1011 EN**: Blank line separating nearby declarations or logic blocks.
  **L1011 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1012 EN**: Declares TableGen def record `VQDMULL_HIGH`.
  **L1012 CN**: 声明 TableGen def 记录 `VQDMULL_HIGH`。
- **L1013 EN**: Declares TableGen def record `VQDMULL_HIGH_N`.
  **L1013 CN**: 声明 TableGen def 记录 `VQDMULL_HIGH_N`。
- **L1014 EN**: Declares TableGen def record `VQDMLAL_HIGH`.
  **L1014 CN**: 声明 TableGen def 记录 `VQDMLAL_HIGH`。
- **L1015 EN**: Declares TableGen def record `VQDMLAL_HIGH_N`.
  **L1015 CN**: 声明 TableGen def 记录 `VQDMLAL_HIGH_N`。
- **L1016 EN**: Declares TableGen def record `VQDMLSL_HIGH`.
  **L1016 CN**: 声明 TableGen def 记录 `VQDMLSL_HIGH`。
- **L1017 EN**: Declares TableGen def record `VQDMLSL_HIGH_N`.
  **L1017 CN**: 声明 TableGen def 记录 `VQDMLSL_HIGH_N`。
- **L1018 EN**: Assigns a TableGen property that affects following records or inherited fields: `let TargetGuard = "aes,neon" in {`.
  **L1018 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let TargetGuard = "aes,neon" in {`。
- **L1019 EN**: Declares TableGen def record `VMULL_P64`.
  **L1019 CN**: 声明 TableGen def 记录 `VMULL_P64`。
- **L1020 EN**: Declares TableGen def record `VMULL_HIGH_P64`.
  **L1020 CN**: 声明 TableGen def 记录 `VMULL_HIGH_P64`。
- **L1021 EN**: Closes the current lexical scope or compound statement.
  **L1021 CN**: 结束当前词法作用域或复合语句块。
- **L1022 EN**: Blank line separating nearby declarations or logic blocks.
  **L1022 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1023 EN**: Blank line separating nearby declarations or logic blocks.
  **L1023 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1024 EN**: Separator comment used for visual grouping.
  **L1024 CN**: 用于视觉分组的分隔注释。
- **L1025 EN**: Comment explains nearby logic, constraints, or intent: `Extract or insert element from vector`.
  **L1025 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extract or insert element from vector`。
- **L1026 EN**: Declares TableGen def record `GET_LANE`.
  **L1026 CN**: 声明 TableGen def 记录 `GET_LANE`。
- **L1027 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheckLaneIndex, 0>]>;`.
  **L1027 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheckLaneIndex, 0>]>;`。
- **L1028 EN**: Declares TableGen def record `SET_LANE`.
  **L1028 CN**: 声明 TableGen def 记录 `SET_LANE`。
- **L1029 EN**: Adds a standalone statement or declaration: `[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`.
  **L1029 CN**: 添加一条独立语句或声明：`[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`。
- **L1030 EN**: Declares TableGen def record `COPY_LANE`.
  **L1030 CN**: 声明 TableGen def 记录 `COPY_LANE`。
- **L1031 EN**: Adds a standalone statement or declaration: `"csilUcUsUiUlPcPsPlfdm", OP_COPY_LN>;`.
  **L1031 CN**: 添加一条独立语句或声明：`"csilUcUsUiUlPcPsPlfdm", OP_COPY_LN>;`。
- **L1032 EN**: Declares TableGen def record `COPYQ_LANE`.
  **L1032 CN**: 声明 TableGen def 记录 `COPYQ_LANE`。
- **L1033 EN**: Adds a standalone statement or declaration: `"QcQsQiQlQUcQUsQUiQUlQPcQPsQfQdQPlQm", OP_COPY_LN>;`.
  **L1033 CN**: 添加一条独立语句或声明：`"QcQsQiQlQUcQUsQUiQUlQPcQPsQfQdQPlQm", OP_COPY_LN>;`。
- **L1034 EN**: Declares TableGen def record `COPY_LANEQ`.
  **L1034 CN**: 声明 TableGen def 记录 `COPY_LANEQ`。
- **L1035 EN**: Adds a standalone statement or declaration: `"csilPcPsPlUcUsUiUlfdm", OP_COPY_LN>;`.
  **L1035 CN**: 添加一条独立语句或声明：`"csilPcPsPlUcUsUiUlfdm", OP_COPY_LN>;`。
- **L1036 EN**: Declares TableGen def record `COPYQ_LANEQ`.
  **L1036 CN**: 声明 TableGen def 记录 `COPYQ_LANEQ`。

### Lines 1037-1064

````tablegen
                     "QcQsQiQlQUcQUsQUiQUlQPcQPsQfQdQPlQm", OP_COPY_LN>;

////////////////////////////////////////////////////////////////////////////////
// Set all lanes to same value
def VDUP_LANE1: WOpInst<"vdup_lane", ".qI", "dQdPlQPl", OP_DUP_LN>;
def VDUP_LANE2: WOpInst<"vdup_laneq", ".QI",
                  "csilUcUsUiUlPcPshfdQcQsQiQlQPcQPsQUcQUsQUiQUlQhQfQdPlQPlmQm",
                        OP_DUP_LN>;
def DUP_N   : WOpInst<"vdup_n", ".1", "dQdPlQPl", OP_DUP>;
def MOV_N   : WOpInst<"vmov_n", ".1", "dQdPlQPl", OP_DUP>;

////////////////////////////////////////////////////////////////////////////////
def COMBINE : NoTestOpInst<"vcombine", "Q..", "dPl", OP_CONC>;

////////////////////////////////////////////////////////////////////////////////
//Initialize a vector from bit pattern
def CREATE : NoTestOpInst<"vcreate", ".(IU>)", "dPl", OP_CAST> {
  let BigEndianSafe = 1;
}

////////////////////////////////////////////////////////////////////////////////

def VMLA_LANEQ   : IOpInst<"vmla_laneq", "...QI",
                           "siUsUifQsQiQUsQUiQf", OP_MLA_LN>;
def VMLS_LANEQ   : IOpInst<"vmls_laneq", "...QI",
                           "siUsUifQsQiQUsQUiQf", OP_MLS_LN>;
def VFMA_LANE    : IInst<"vfma_lane", "...qI", "fdQfQd",
                        [ImmCheck<3, ImmCheckLaneIndex, 2>]>;
````
- **L1037 EN**: Adds a standalone statement or declaration: `"QcQsQiQlQUcQUsQUiQUlQPcQPsQfQdQPlQm", OP_COPY_LN>;`.
  **L1037 CN**: 添加一条独立语句或声明：`"QcQsQiQlQUcQUsQUiQUlQPcQPsQfQdQPlQm", OP_COPY_LN>;`。
- **L1038 EN**: Blank line separating nearby declarations or logic blocks.
  **L1038 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1039 EN**: Separator comment used for visual grouping.
  **L1039 CN**: 用于视觉分组的分隔注释。
- **L1040 EN**: Comment explains nearby logic, constraints, or intent: `Set all lanes to same value`.
  **L1040 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Set all lanes to same value`。
- **L1041 EN**: Declares TableGen def record `VDUP_LANE1`.
  **L1041 CN**: 声明 TableGen def 记录 `VDUP_LANE1`。
- **L1042 EN**: Declares TableGen def record `VDUP_LANE2`.
  **L1042 CN**: 声明 TableGen def 记录 `VDUP_LANE2`。
- **L1043 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"csilUcUsUiUlPcPshfdQcQsQiQlQPcQPsQUcQUsQUiQUlQhQfQdPlQPlmQm",`.
  **L1043 CN**: 继续一个多行参数列表、初始化器或聚合项：`"csilUcUsUiUlPcPshfdQcQsQiQlQPcQPsQUcQUsQUiQUlQhQfQdPlQPlmQm",`。
- **L1044 EN**: Adds a standalone statement or declaration: `OP_DUP_LN>;`.
  **L1044 CN**: 添加一条独立语句或声明：`OP_DUP_LN>;`。
- **L1045 EN**: Declares TableGen def record `DUP_N`.
  **L1045 CN**: 声明 TableGen def 记录 `DUP_N`。
- **L1046 EN**: Declares TableGen def record `MOV_N`.
  **L1046 CN**: 声明 TableGen def 记录 `MOV_N`。
- **L1047 EN**: Blank line separating nearby declarations or logic blocks.
  **L1047 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1048 EN**: Separator comment used for visual grouping.
  **L1048 CN**: 用于视觉分组的分隔注释。
- **L1049 EN**: Declares TableGen def record `COMBINE`.
  **L1049 CN**: 声明 TableGen def 记录 `COMBINE`。
- **L1050 EN**: Blank line separating nearby declarations or logic blocks.
  **L1050 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1051 EN**: Separator comment used for visual grouping.
  **L1051 CN**: 用于视觉分组的分隔注释。
- **L1052 EN**: Comment explains nearby logic, constraints, or intent: `Initialize a vector from bit pattern`.
  **L1052 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Initialize a vector from bit pattern`。
- **L1053 EN**: Declares TableGen def record `CREATE`.
  **L1053 CN**: 声明 TableGen def 记录 `CREATE`。
- **L1054 EN**: Assigns a TableGen property that affects following records or inherited fields: `let BigEndianSafe = 1;`.
  **L1054 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let BigEndianSafe = 1;`。
- **L1055 EN**: Closes the current lexical scope or compound statement.
  **L1055 CN**: 结束当前词法作用域或复合语句块。
- **L1056 EN**: Blank line separating nearby declarations or logic blocks.
  **L1056 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1057 EN**: Separator comment used for visual grouping.
  **L1057 CN**: 用于视觉分组的分隔注释。
- **L1058 EN**: Blank line separating nearby declarations or logic blocks.
  **L1058 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1059 EN**: Declares TableGen def record `VMLA_LANEQ`.
  **L1059 CN**: 声明 TableGen def 记录 `VMLA_LANEQ`。
- **L1060 EN**: Adds a standalone statement or declaration: `"siUsUifQsQiQUsQUiQf", OP_MLA_LN>;`.
  **L1060 CN**: 添加一条独立语句或声明：`"siUsUifQsQiQUsQUiQf", OP_MLA_LN>;`。
- **L1061 EN**: Declares TableGen def record `VMLS_LANEQ`.
  **L1061 CN**: 声明 TableGen def 记录 `VMLS_LANEQ`。
- **L1062 EN**: Adds a standalone statement or declaration: `"siUsUifQsQiQUsQUiQf", OP_MLS_LN>;`.
  **L1062 CN**: 添加一条独立语句或声明：`"siUsUifQsQiQUsQUiQf", OP_MLS_LN>;`。
- **L1063 EN**: Declares TableGen def record `VFMA_LANE`.
  **L1063 CN**: 声明 TableGen def 记录 `VFMA_LANE`。
- **L1064 EN**: Adds a standalone statement or declaration: `[ImmCheck<3, ImmCheckLaneIndex, 2>]>;`.
  **L1064 CN**: 添加一条独立语句或声明：`[ImmCheck<3, ImmCheckLaneIndex, 2>]>;`。

### Lines 1065-1092

````tablegen
def VFMA_LANEQ   : IInst<"vfma_laneq", "...QI", "fdQfQd",
                        [ImmCheck<3, ImmCheckLaneIndex, 2>]>;
def VFMS_LANE    : IOpInst<"vfms_lane", "...qI", "fdQfQd", OP_FMS_LN>;
def VFMS_LANEQ   : IOpInst<"vfms_laneq", "...QI", "fdQfQd", OP_FMS_LNQ>;

def VMLAL_LANEQ  : SOpInst<"vmlal_laneq", "(>Q)(>Q).QI", "siUsUi", OP_MLAL_LN>;
def VMLAL_HIGH_LANE   : SOpInst<"vmlal_high_lane", "(>Q)(>Q)Q.I", "siUsUi",
                                OP_MLALHi_LN>;
def VMLAL_HIGH_LANEQ  : SOpInst<"vmlal_high_laneq", "(>Q)(>Q)QQI", "siUsUi",
                                OP_MLALHi_LN>;
def VMLSL_LANEQ  : SOpInst<"vmlsl_laneq", "(>Q)(>Q).QI", "siUsUi", OP_MLSL_LN>;
def VMLSL_HIGH_LANE   : SOpInst<"vmlsl_high_lane", "(>Q)(>Q)Q.I", "siUsUi",
                                OP_MLSLHi_LN>;
def VMLSL_HIGH_LANEQ  : SOpInst<"vmlsl_high_laneq", "(>Q)(>Q)QQI", "siUsUi",
                                OP_MLSLHi_LN>;
def VQDMLAL_LANEQ  : SOpInst<"vqdmlal_laneq", "(>Q)(>Q).QI", "si", OP_QDMLAL_LN>;
def VQDMLAL_HIGH_LANE   : SOpInst<"vqdmlal_high_lane", "(>Q)(>Q)Q.I", "si",
                                OP_QDMLALHi_LN>;
def VQDMLAL_HIGH_LANEQ  : SOpInst<"vqdmlal_high_laneq", "(>Q)(>Q)QQI", "si",
                                OP_QDMLALHi_LN>;
def VQDMLSL_LANEQ  : SOpInst<"vqdmlsl_laneq", "(>Q)(>Q).QI", "si", OP_QDMLSL_LN>;
def VQDMLSL_HIGH_LANE   : SOpInst<"vqdmlsl_high_lane", "(>Q)(>Q)Q.I", "si",
                                OP_QDMLSLHi_LN>;
def VQDMLSL_HIGH_LANEQ  : SOpInst<"vqdmlsl_high_laneq", "(>Q)(>Q)QQI", "si",
                                OP_QDMLSLHi_LN>;

// Newly add double parameter for vmul_lane in aarch64
// Note: d type is handled by SCALAR_VMUL_LANE
````
- **L1065 EN**: Declares TableGen def record `VFMA_LANEQ`.
  **L1065 CN**: 声明 TableGen def 记录 `VFMA_LANEQ`。
- **L1066 EN**: Adds a standalone statement or declaration: `[ImmCheck<3, ImmCheckLaneIndex, 2>]>;`.
  **L1066 CN**: 添加一条独立语句或声明：`[ImmCheck<3, ImmCheckLaneIndex, 2>]>;`。
- **L1067 EN**: Declares TableGen def record `VFMS_LANE`.
  **L1067 CN**: 声明 TableGen def 记录 `VFMS_LANE`。
- **L1068 EN**: Declares TableGen def record `VFMS_LANEQ`.
  **L1068 CN**: 声明 TableGen def 记录 `VFMS_LANEQ`。
- **L1069 EN**: Blank line separating nearby declarations or logic blocks.
  **L1069 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1070 EN**: Declares TableGen def record `VMLAL_LANEQ`.
  **L1070 CN**: 声明 TableGen def 记录 `VMLAL_LANEQ`。
- **L1071 EN**: Declares TableGen def record `VMLAL_HIGH_LANE`.
  **L1071 CN**: 声明 TableGen def 记录 `VMLAL_HIGH_LANE`。
- **L1072 EN**: Adds a standalone statement or declaration: `OP_MLALHi_LN>;`.
  **L1072 CN**: 添加一条独立语句或声明：`OP_MLALHi_LN>;`。
- **L1073 EN**: Declares TableGen def record `VMLAL_HIGH_LANEQ`.
  **L1073 CN**: 声明 TableGen def 记录 `VMLAL_HIGH_LANEQ`。
- **L1074 EN**: Adds a standalone statement or declaration: `OP_MLALHi_LN>;`.
  **L1074 CN**: 添加一条独立语句或声明：`OP_MLALHi_LN>;`。
- **L1075 EN**: Declares TableGen def record `VMLSL_LANEQ`.
  **L1075 CN**: 声明 TableGen def 记录 `VMLSL_LANEQ`。
- **L1076 EN**: Declares TableGen def record `VMLSL_HIGH_LANE`.
  **L1076 CN**: 声明 TableGen def 记录 `VMLSL_HIGH_LANE`。
- **L1077 EN**: Adds a standalone statement or declaration: `OP_MLSLHi_LN>;`.
  **L1077 CN**: 添加一条独立语句或声明：`OP_MLSLHi_LN>;`。
- **L1078 EN**: Declares TableGen def record `VMLSL_HIGH_LANEQ`.
  **L1078 CN**: 声明 TableGen def 记录 `VMLSL_HIGH_LANEQ`。
- **L1079 EN**: Adds a standalone statement or declaration: `OP_MLSLHi_LN>;`.
  **L1079 CN**: 添加一条独立语句或声明：`OP_MLSLHi_LN>;`。
- **L1080 EN**: Declares TableGen def record `VQDMLAL_LANEQ`.
  **L1080 CN**: 声明 TableGen def 记录 `VQDMLAL_LANEQ`。
- **L1081 EN**: Declares TableGen def record `VQDMLAL_HIGH_LANE`.
  **L1081 CN**: 声明 TableGen def 记录 `VQDMLAL_HIGH_LANE`。
- **L1082 EN**: Adds a standalone statement or declaration: `OP_QDMLALHi_LN>;`.
  **L1082 CN**: 添加一条独立语句或声明：`OP_QDMLALHi_LN>;`。
- **L1083 EN**: Declares TableGen def record `VQDMLAL_HIGH_LANEQ`.
  **L1083 CN**: 声明 TableGen def 记录 `VQDMLAL_HIGH_LANEQ`。
- **L1084 EN**: Adds a standalone statement or declaration: `OP_QDMLALHi_LN>;`.
  **L1084 CN**: 添加一条独立语句或声明：`OP_QDMLALHi_LN>;`。
- **L1085 EN**: Declares TableGen def record `VQDMLSL_LANEQ`.
  **L1085 CN**: 声明 TableGen def 记录 `VQDMLSL_LANEQ`。
- **L1086 EN**: Declares TableGen def record `VQDMLSL_HIGH_LANE`.
  **L1086 CN**: 声明 TableGen def 记录 `VQDMLSL_HIGH_LANE`。
- **L1087 EN**: Adds a standalone statement or declaration: `OP_QDMLSLHi_LN>;`.
  **L1087 CN**: 添加一条独立语句或声明：`OP_QDMLSLHi_LN>;`。
- **L1088 EN**: Declares TableGen def record `VQDMLSL_HIGH_LANEQ`.
  **L1088 CN**: 声明 TableGen def 记录 `VQDMLSL_HIGH_LANEQ`。
- **L1089 EN**: Adds a standalone statement or declaration: `OP_QDMLSLHi_LN>;`.
  **L1089 CN**: 添加一条独立语句或声明：`OP_QDMLSLHi_LN>;`。
- **L1090 EN**: Blank line separating nearby declarations or logic blocks.
  **L1090 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1091 EN**: Comment explains nearby logic, constraints, or intent: `Newly add double parameter for vmul_lane in aarch64`.
  **L1091 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Newly add double parameter for vmul_lane in aarch64`。
- **L1092 EN**: Comment highlights an implementation note: `Note: d type is handled by SCALAR_VMUL_LANE`.
  **L1092 CN**: 注释强调一条实现说明：`Note: d type is handled by SCALAR_VMUL_LANE`。

### Lines 1093-1120

````tablegen
def VMUL_LANE_A64 : IOpInst<"vmul_lane", "..qI", "Qd", OP_MUL_LN>;

// Note: d type is handled by SCALAR_VMUL_LANEQ
def VMUL_LANEQ   : IOpInst<"vmul_laneq", "..QI",
                           "sifUsUiQsQiQUsQUiQfQd", OP_MUL_LN>;
def VMULL_LANEQ  : SOpInst<"vmull_laneq", "(>Q).QI", "siUsUi", OP_MULL_LN>;
def VMULL_HIGH_LANE   : SOpInst<"vmull_high_lane", "(>Q)Q.I", "siUsUi",
                                OP_MULLHi_LN>;
def VMULL_HIGH_LANEQ  : SOpInst<"vmull_high_laneq", "(>Q)QQI", "siUsUi",
                                OP_MULLHi_LN>;
def VQDMULL_LANEQ  : SOpInst<"vqdmull_laneq", "(>Q).QI", "si", OP_QDMULL_LN>;
def VQDMULL_HIGH_LANE   : SOpInst<"vqdmull_high_lane", "(>Q)Q.I", "si",
                                  OP_QDMULLHi_LN>;
def VQDMULL_HIGH_LANEQ  : SOpInst<"vqdmull_high_laneq", "(>Q)QQI", "si",
                                  OP_QDMULLHi_LN>;
def VQDMULH_LANEQ  : SInst<"vqdmulh_laneq", "..QI", "siQsQi",
                          [ImmCheck<2, ImmCheckLaneIndex, 1>]>;
def VQRDMULH_LANEQ : SInst<"vqrdmulh_laneq", "..QI", "siQsQi",
                          [ImmCheck<2, ImmCheckLaneIndex, 1>]>;

let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "v8.1a,neon" in {
def VQRDMLAH_LANEQ : SOpInst<"vqrdmlah_laneq", "...QI", "siQsQi", OP_QRDMLAH_LN>;
def VQRDMLSH_LANEQ : SOpInst<"vqrdmlsh_laneq", "...QI", "siQsQi", OP_QRDMLSH_LN>;
} // ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "v8.1a"

// Note: d type implemented by SCALAR_VMULX_LANE
def VMULX_LANE : IOpInst<"vmulx_lane", "..qI", "fQfQd", OP_MULX_LN>;
// Note: d type is implemented by SCALAR_VMULX_LANEQ
````
- **L1093 EN**: Declares TableGen def record `VMUL_LANE_A64`.
  **L1093 CN**: 声明 TableGen def 记录 `VMUL_LANE_A64`。
- **L1094 EN**: Blank line separating nearby declarations or logic blocks.
  **L1094 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1095 EN**: Comment highlights an implementation note: `Note: d type is handled by SCALAR_VMUL_LANEQ`.
  **L1095 CN**: 注释强调一条实现说明：`Note: d type is handled by SCALAR_VMUL_LANEQ`。
- **L1096 EN**: Declares TableGen def record `VMUL_LANEQ`.
  **L1096 CN**: 声明 TableGen def 记录 `VMUL_LANEQ`。
- **L1097 EN**: Adds a standalone statement or declaration: `"sifUsUiQsQiQUsQUiQfQd", OP_MUL_LN>;`.
  **L1097 CN**: 添加一条独立语句或声明：`"sifUsUiQsQiQUsQUiQfQd", OP_MUL_LN>;`。
- **L1098 EN**: Declares TableGen def record `VMULL_LANEQ`.
  **L1098 CN**: 声明 TableGen def 记录 `VMULL_LANEQ`。
- **L1099 EN**: Declares TableGen def record `VMULL_HIGH_LANE`.
  **L1099 CN**: 声明 TableGen def 记录 `VMULL_HIGH_LANE`。
- **L1100 EN**: Adds a standalone statement or declaration: `OP_MULLHi_LN>;`.
  **L1100 CN**: 添加一条独立语句或声明：`OP_MULLHi_LN>;`。
- **L1101 EN**: Declares TableGen def record `VMULL_HIGH_LANEQ`.
  **L1101 CN**: 声明 TableGen def 记录 `VMULL_HIGH_LANEQ`。
- **L1102 EN**: Adds a standalone statement or declaration: `OP_MULLHi_LN>;`.
  **L1102 CN**: 添加一条独立语句或声明：`OP_MULLHi_LN>;`。
- **L1103 EN**: Declares TableGen def record `VQDMULL_LANEQ`.
  **L1103 CN**: 声明 TableGen def 记录 `VQDMULL_LANEQ`。
- **L1104 EN**: Declares TableGen def record `VQDMULL_HIGH_LANE`.
  **L1104 CN**: 声明 TableGen def 记录 `VQDMULL_HIGH_LANE`。
- **L1105 EN**: Adds a standalone statement or declaration: `OP_QDMULLHi_LN>;`.
  **L1105 CN**: 添加一条独立语句或声明：`OP_QDMULLHi_LN>;`。
- **L1106 EN**: Declares TableGen def record `VQDMULL_HIGH_LANEQ`.
  **L1106 CN**: 声明 TableGen def 记录 `VQDMULL_HIGH_LANEQ`。
- **L1107 EN**: Adds a standalone statement or declaration: `OP_QDMULLHi_LN>;`.
  **L1107 CN**: 添加一条独立语句或声明：`OP_QDMULLHi_LN>;`。
- **L1108 EN**: Declares TableGen def record `VQDMULH_LANEQ`.
  **L1108 CN**: 声明 TableGen def 记录 `VQDMULH_LANEQ`。
- **L1109 EN**: Adds a standalone statement or declaration: `[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`.
  **L1109 CN**: 添加一条独立语句或声明：`[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`。
- **L1110 EN**: Declares TableGen def record `VQRDMULH_LANEQ`.
  **L1110 CN**: 声明 TableGen def 记录 `VQRDMULH_LANEQ`。
- **L1111 EN**: Adds a standalone statement or declaration: `[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`.
  **L1111 CN**: 添加一条独立语句或声明：`[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`。
- **L1112 EN**: Blank line separating nearby declarations or logic blocks.
  **L1112 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1113 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "v8.1a,neon" in {`.
  **L1113 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "v8.1a,neon" in {`。
- **L1114 EN**: Declares TableGen def record `VQRDMLAH_LANEQ`.
  **L1114 CN**: 声明 TableGen def 记录 `VQRDMLAH_LANEQ`。
- **L1115 EN**: Declares TableGen def record `VQRDMLSH_LANEQ`.
  **L1115 CN**: 声明 TableGen def 记录 `VQRDMLSH_LANEQ`。
- **L1116 EN**: Continues logic associated with callable symbol `defined`.
  **L1116 CN**: 继续与可调用符号 `defined` 相关的逻辑。
- **L1117 EN**: Blank line separating nearby declarations or logic blocks.
  **L1117 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1118 EN**: Comment highlights an implementation note: `Note: d type implemented by SCALAR_VMULX_LANE`.
  **L1118 CN**: 注释强调一条实现说明：`Note: d type implemented by SCALAR_VMULX_LANE`。
- **L1119 EN**: Declares TableGen def record `VMULX_LANE`.
  **L1119 CN**: 声明 TableGen def 记录 `VMULX_LANE`。
- **L1120 EN**: Comment highlights an implementation note: `Note: d type is implemented by SCALAR_VMULX_LANEQ`.
  **L1120 CN**: 注释强调一条实现说明：`Note: d type is implemented by SCALAR_VMULX_LANEQ`。

### Lines 1121-1148

````tablegen
def VMULX_LANEQ : IOpInst<"vmulx_laneq", "..QI", "fQfQd", OP_MULX_LN>;

////////////////////////////////////////////////////////////////////////////////
// Across vectors class
def VADDLV  : SInst<"vaddlv", "(1>).", "csiUcUsUiQcQsQiQUcQUsQUi">;
def VMAXV   : SInst<"vmaxv", "1.", "csifUcUsUiQcQsQiQUcQUsQUiQfQd">;
def VMINV   : SInst<"vminv", "1.", "csifUcUsUiQcQsQiQUcQUsQUiQfQd">;
def VADDV   : SInst<"vaddv", "1.", "csifUcUsUiQcQsQiQUcQUsQUiQfQdQlQUl">;
def FMAXNMV : SInst<"vmaxnmv", "1.", "fQfQd">;
def FMINNMV : SInst<"vminnmv", "1.", "fQfQd">;

////////////////////////////////////////////////////////////////////////////////
// Newly added Vector Extract for f64
def VEXT_A64 : WInst<"vext", "...I", "dQdPlQPl",
                    [ImmCheck<2, ImmCheckLaneIndex, 0>]>;

////////////////////////////////////////////////////////////////////////////////
// Crypto
let ArchGuard = "__ARM_ARCH >= 8", TargetGuard = "aes,neon" in {
def AESE : SInst<"vaese", "...", "QUc">;
def AESD : SInst<"vaesd", "...", "QUc">;
def AESMC : SInst<"vaesmc", "..", "QUc">;
def AESIMC : SInst<"vaesimc", "..", "QUc">;
}

let ArchGuard = "__ARM_ARCH >= 8", TargetGuard = "sha2,neon" in {
def SHA1H : SInst<"vsha1h", "11", "Ui">;
def SHA1SU1 : SInst<"vsha1su1", "...", "QUi">;
````
- **L1121 EN**: Declares TableGen def record `VMULX_LANEQ`.
  **L1121 CN**: 声明 TableGen def 记录 `VMULX_LANEQ`。
- **L1122 EN**: Blank line separating nearby declarations or logic blocks.
  **L1122 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1123 EN**: Separator comment used for visual grouping.
  **L1123 CN**: 用于视觉分组的分隔注释。
- **L1124 EN**: Comment explains nearby logic, constraints, or intent: `Across vectors class`.
  **L1124 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Across vectors class`。
- **L1125 EN**: Declares TableGen def record `VADDLV`.
  **L1125 CN**: 声明 TableGen def 记录 `VADDLV`。
- **L1126 EN**: Declares TableGen def record `VMAXV`.
  **L1126 CN**: 声明 TableGen def 记录 `VMAXV`。
- **L1127 EN**: Declares TableGen def record `VMINV`.
  **L1127 CN**: 声明 TableGen def 记录 `VMINV`。
- **L1128 EN**: Declares TableGen def record `VADDV`.
  **L1128 CN**: 声明 TableGen def 记录 `VADDV`。
- **L1129 EN**: Declares TableGen def record `FMAXNMV`.
  **L1129 CN**: 声明 TableGen def 记录 `FMAXNMV`。
- **L1130 EN**: Declares TableGen def record `FMINNMV`.
  **L1130 CN**: 声明 TableGen def 记录 `FMINNMV`。
- **L1131 EN**: Blank line separating nearby declarations or logic blocks.
  **L1131 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1132 EN**: Separator comment used for visual grouping.
  **L1132 CN**: 用于视觉分组的分隔注释。
- **L1133 EN**: Comment explains nearby logic, constraints, or intent: `Newly added Vector Extract for f64`.
  **L1133 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Newly added Vector Extract for f64`。
- **L1134 EN**: Declares TableGen def record `VEXT_A64`.
  **L1134 CN**: 声明 TableGen def 记录 `VEXT_A64`。
- **L1135 EN**: Adds a standalone statement or declaration: `[ImmCheck<2, ImmCheckLaneIndex, 0>]>;`.
  **L1135 CN**: 添加一条独立语句或声明：`[ImmCheck<2, ImmCheckLaneIndex, 0>]>;`。
- **L1136 EN**: Blank line separating nearby declarations or logic blocks.
  **L1136 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1137 EN**: Separator comment used for visual grouping.
  **L1137 CN**: 用于视觉分组的分隔注释。
- **L1138 EN**: Comment explains nearby logic, constraints, or intent: `Crypto`.
  **L1138 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Crypto`。
- **L1139 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "__ARM_ARCH >= 8", TargetGuard = "aes,neon" in {`.
  **L1139 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "__ARM_ARCH >= 8", TargetGuard = "aes,neon" in {`。
- **L1140 EN**: Declares TableGen def record `AESE`.
  **L1140 CN**: 声明 TableGen def 记录 `AESE`。
- **L1141 EN**: Declares TableGen def record `AESD`.
  **L1141 CN**: 声明 TableGen def 记录 `AESD`。
- **L1142 EN**: Declares TableGen def record `AESMC`.
  **L1142 CN**: 声明 TableGen def 记录 `AESMC`。
- **L1143 EN**: Declares TableGen def record `AESIMC`.
  **L1143 CN**: 声明 TableGen def 记录 `AESIMC`。
- **L1144 EN**: Closes the current lexical scope or compound statement.
  **L1144 CN**: 结束当前词法作用域或复合语句块。
- **L1145 EN**: Blank line separating nearby declarations or logic blocks.
  **L1145 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1146 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "__ARM_ARCH >= 8", TargetGuard = "sha2,neon" in {`.
  **L1146 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "__ARM_ARCH >= 8", TargetGuard = "sha2,neon" in {`。
- **L1147 EN**: Declares TableGen def record `SHA1H`.
  **L1147 CN**: 声明 TableGen def 记录 `SHA1H`。
- **L1148 EN**: Declares TableGen def record `SHA1SU1`.
  **L1148 CN**: 声明 TableGen def 记录 `SHA1SU1`。

### Lines 1149-1176

````tablegen
def SHA256SU0 : SInst<"vsha256su0", "...", "QUi">;

def SHA1C : SInst<"vsha1c", "..1.", "QUi">;
def SHA1P : SInst<"vsha1p", "..1.", "QUi">;
def SHA1M : SInst<"vsha1m", "..1.", "QUi">;
def SHA1SU0 : SInst<"vsha1su0", "....", "QUi">;
def SHA256H : SInst<"vsha256h", "....", "QUi">;
def SHA256H2 : SInst<"vsha256h2", "....", "QUi">;
def SHA256SU1 : SInst<"vsha256su1", "....", "QUi">;
}

let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "sha3,neon" in {
def BCAX : SInst<"vbcax", "....", "QUcQUsQUiQUlQcQsQiQl">;
def EOR3 : SInst<"veor3", "....", "QUcQUsQUiQUlQcQsQiQl">;
def RAX1 : SInst<"vrax1", "...", "QUl">;
def XAR :  SInst<"vxar", "...I", "QUl", [ImmCheck<2, ImmCheck0_63>]>;
}

let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "sha3,neon" in {
def SHA512SU0 : SInst<"vsha512su0", "...", "QUl">;
def SHA512su1 : SInst<"vsha512su1", "....", "QUl">;
def SHA512H : SInst<"vsha512h", "....", "QUl">;
def SHA512H2 : SInst<"vsha512h2", "....", "QUl">;
}

let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "sm4,neon" in {
def SM3SS1 : SInst<"vsm3ss1", "....", "QUi">;
def SM3TT1A : SInst<"vsm3tt1a", "....I", "QUi", [ImmCheck<3, ImmCheck0_3>]>;
````
- **L1149 EN**: Declares TableGen def record `SHA256SU0`.
  **L1149 CN**: 声明 TableGen def 记录 `SHA256SU0`。
- **L1150 EN**: Blank line separating nearby declarations or logic blocks.
  **L1150 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1151 EN**: Declares TableGen def record `SHA1C`.
  **L1151 CN**: 声明 TableGen def 记录 `SHA1C`。
- **L1152 EN**: Declares TableGen def record `SHA1P`.
  **L1152 CN**: 声明 TableGen def 记录 `SHA1P`。
- **L1153 EN**: Declares TableGen def record `SHA1M`.
  **L1153 CN**: 声明 TableGen def 记录 `SHA1M`。
- **L1154 EN**: Declares TableGen def record `SHA1SU0`.
  **L1154 CN**: 声明 TableGen def 记录 `SHA1SU0`。
- **L1155 EN**: Declares TableGen def record `SHA256H`.
  **L1155 CN**: 声明 TableGen def 记录 `SHA256H`。
- **L1156 EN**: Declares TableGen def record `SHA256H2`.
  **L1156 CN**: 声明 TableGen def 记录 `SHA256H2`。
- **L1157 EN**: Declares TableGen def record `SHA256SU1`.
  **L1157 CN**: 声明 TableGen def 记录 `SHA256SU1`。
- **L1158 EN**: Closes the current lexical scope or compound statement.
  **L1158 CN**: 结束当前词法作用域或复合语句块。
- **L1159 EN**: Blank line separating nearby declarations or logic blocks.
  **L1159 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1160 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "sha3,neon" in {`.
  **L1160 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "sha3,neon" in {`。
- **L1161 EN**: Declares TableGen def record `BCAX`.
  **L1161 CN**: 声明 TableGen def 记录 `BCAX`。
- **L1162 EN**: Declares TableGen def record `EOR3`.
  **L1162 CN**: 声明 TableGen def 记录 `EOR3`。
- **L1163 EN**: Declares TableGen def record `RAX1`.
  **L1163 CN**: 声明 TableGen def 记录 `RAX1`。
- **L1164 EN**: Declares TableGen def record `XAR`.
  **L1164 CN**: 声明 TableGen def 记录 `XAR`。
- **L1165 EN**: Closes the current lexical scope or compound statement.
  **L1165 CN**: 结束当前词法作用域或复合语句块。
- **L1166 EN**: Blank line separating nearby declarations or logic blocks.
  **L1166 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1167 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "sha3,neon" in {`.
  **L1167 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "sha3,neon" in {`。
- **L1168 EN**: Declares TableGen def record `SHA512SU0`.
  **L1168 CN**: 声明 TableGen def 记录 `SHA512SU0`。
- **L1169 EN**: Declares TableGen def record `SHA512su1`.
  **L1169 CN**: 声明 TableGen def 记录 `SHA512su1`。
- **L1170 EN**: Declares TableGen def record `SHA512H`.
  **L1170 CN**: 声明 TableGen def 记录 `SHA512H`。
- **L1171 EN**: Declares TableGen def record `SHA512H2`.
  **L1171 CN**: 声明 TableGen def 记录 `SHA512H2`。
- **L1172 EN**: Closes the current lexical scope or compound statement.
  **L1172 CN**: 结束当前词法作用域或复合语句块。
- **L1173 EN**: Blank line separating nearby declarations or logic blocks.
  **L1173 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1174 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "sm4,neon" in {`.
  **L1174 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "sm4,neon" in {`。
- **L1175 EN**: Declares TableGen def record `SM3SS1`.
  **L1175 CN**: 声明 TableGen def 记录 `SM3SS1`。
- **L1176 EN**: Declares TableGen def record `SM3TT1A`.
  **L1176 CN**: 声明 TableGen def 记录 `SM3TT1A`。

### Lines 1177-1204

````tablegen
def SM3TT1B : SInst<"vsm3tt1b", "....I", "QUi", [ImmCheck<3, ImmCheck0_3>]>;
def SM3TT2A : SInst<"vsm3tt2a", "....I", "QUi", [ImmCheck<3, ImmCheck0_3>]>;
def SM3TT2B : SInst<"vsm3tt2b", "....I", "QUi", [ImmCheck<3, ImmCheck0_3>]>;
def SM3PARTW1 : SInst<"vsm3partw1", "....", "QUi">;
def SM3PARTW2 : SInst<"vsm3partw2", "....", "QUi">;
}

let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "sm4,neon" in {
def SM4E : SInst<"vsm4e", "...", "QUi">;
def SM4EKEY : SInst<"vsm4ekey", "...", "QUi">;
}

////////////////////////////////////////////////////////////////////////////////
// poly128_t vadd for AArch64 only see VADDP for the rest
def VADDP_Q   : WInst<"vadd", "...", "QPk">;

////////////////////////////////////////////////////////////////////////////////
// Float -> Int conversions with explicit rounding mode

let ArchGuard = "__ARM_ARCH >= 8" in {
def FCVTNS_S32 : SInst<"vcvtn_s32", "S.", "fQf">;
def FCVTNU_S32 : SInst<"vcvtn_u32", "U.", "fQf">;
def FCVTPS_S32 : SInst<"vcvtp_s32", "S.", "fQf">;
def FCVTPU_S32 : SInst<"vcvtp_u32", "U.", "fQf">;
def FCVTMS_S32 : SInst<"vcvtm_s32", "S.", "fQf">;
def FCVTMU_S32 : SInst<"vcvtm_u32", "U.", "fQf">;
def FCVTAS_S32 : SInst<"vcvta_s32", "S.", "fQf">;
def FCVTAU_S32 : SInst<"vcvta_u32", "U.", "fQf">;
````
- **L1177 EN**: Declares TableGen def record `SM3TT1B`.
  **L1177 CN**: 声明 TableGen def 记录 `SM3TT1B`。
- **L1178 EN**: Declares TableGen def record `SM3TT2A`.
  **L1178 CN**: 声明 TableGen def 记录 `SM3TT2A`。
- **L1179 EN**: Declares TableGen def record `SM3TT2B`.
  **L1179 CN**: 声明 TableGen def 记录 `SM3TT2B`。
- **L1180 EN**: Declares TableGen def record `SM3PARTW1`.
  **L1180 CN**: 声明 TableGen def 记录 `SM3PARTW1`。
- **L1181 EN**: Declares TableGen def record `SM3PARTW2`.
  **L1181 CN**: 声明 TableGen def 记录 `SM3PARTW2`。
- **L1182 EN**: Closes the current lexical scope or compound statement.
  **L1182 CN**: 结束当前词法作用域或复合语句块。
- **L1183 EN**: Blank line separating nearby declarations or logic blocks.
  **L1183 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1184 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "sm4,neon" in {`.
  **L1184 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "sm4,neon" in {`。
- **L1185 EN**: Declares TableGen def record `SM4E`.
  **L1185 CN**: 声明 TableGen def 记录 `SM4E`。
- **L1186 EN**: Declares TableGen def record `SM4EKEY`.
  **L1186 CN**: 声明 TableGen def 记录 `SM4EKEY`。
- **L1187 EN**: Closes the current lexical scope or compound statement.
  **L1187 CN**: 结束当前词法作用域或复合语句块。
- **L1188 EN**: Blank line separating nearby declarations or logic blocks.
  **L1188 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1189 EN**: Separator comment used for visual grouping.
  **L1189 CN**: 用于视觉分组的分隔注释。
- **L1190 EN**: Comment explains nearby logic, constraints, or intent: `poly128_t vadd for AArch64 only see VADDP for the rest`.
  **L1190 CN**: 注释解释附近代码的逻辑、约束或设计意图：`poly128_t vadd for AArch64 only see VADDP for the rest`。
- **L1191 EN**: Declares TableGen def record `VADDP_Q`.
  **L1191 CN**: 声明 TableGen def 记录 `VADDP_Q`。
- **L1192 EN**: Blank line separating nearby declarations or logic blocks.
  **L1192 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1193 EN**: Separator comment used for visual grouping.
  **L1193 CN**: 用于视觉分组的分隔注释。
- **L1194 EN**: Comment explains nearby logic, constraints, or intent: `Float -> Int conversions with explicit rounding mode`.
  **L1194 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Float -> Int conversions with explicit rounding mode`。
- **L1195 EN**: Blank line separating nearby declarations or logic blocks.
  **L1195 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1196 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "__ARM_ARCH >= 8" in {`.
  **L1196 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "__ARM_ARCH >= 8" in {`。
- **L1197 EN**: Declares TableGen def record `FCVTNS_S32`.
  **L1197 CN**: 声明 TableGen def 记录 `FCVTNS_S32`。
- **L1198 EN**: Declares TableGen def record `FCVTNU_S32`.
  **L1198 CN**: 声明 TableGen def 记录 `FCVTNU_S32`。
- **L1199 EN**: Declares TableGen def record `FCVTPS_S32`.
  **L1199 CN**: 声明 TableGen def 记录 `FCVTPS_S32`。
- **L1200 EN**: Declares TableGen def record `FCVTPU_S32`.
  **L1200 CN**: 声明 TableGen def 记录 `FCVTPU_S32`。
- **L1201 EN**: Declares TableGen def record `FCVTMS_S32`.
  **L1201 CN**: 声明 TableGen def 记录 `FCVTMS_S32`。
- **L1202 EN**: Declares TableGen def record `FCVTMU_S32`.
  **L1202 CN**: 声明 TableGen def 记录 `FCVTMU_S32`。
- **L1203 EN**: Declares TableGen def record `FCVTAS_S32`.
  **L1203 CN**: 声明 TableGen def 记录 `FCVTAS_S32`。
- **L1204 EN**: Declares TableGen def record `FCVTAU_S32`.
  **L1204 CN**: 声明 TableGen def 记录 `FCVTAU_S32`。

### Lines 1205-1232

````tablegen
}

let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)" in {
def FCVTNS_S64 : SInst<"vcvtn_s64", "S.", "dQd">;
def FCVTNU_S64 : SInst<"vcvtn_u64", "U.", "dQd">;
def FCVTPS_S64 : SInst<"vcvtp_s64", "S.", "dQd">;
def FCVTPU_S64 : SInst<"vcvtp_u64", "U.", "dQd">;
def FCVTMS_S64 : SInst<"vcvtm_s64", "S.", "dQd">;
def FCVTMU_S64 : SInst<"vcvtm_u64", "U.", "dQd">;
def FCVTAS_S64 : SInst<"vcvta_s64", "S.", "dQd">;
def FCVTAU_S64 : SInst<"vcvta_u64", "U.", "dQd">;
}

////////////////////////////////////////////////////////////////////////////////
// Round to Integral

let ArchGuard = "__ARM_ARCH >= 8 && defined(__ARM_FEATURE_DIRECTED_ROUNDING)" in {
def FRINTN_S32 : SInst<"vrndn", "..", "fQf">;
def FRINTA_S32 : SInst<"vrnda", "..", "fQf">;
def FRINTP_S32 : SInst<"vrndp", "..", "fQf">;
def FRINTM_S32 : SInst<"vrndm", "..", "fQf">;
def FRINTX_S32 : SInst<"vrndx", "..", "fQf">;
def FRINTZ_S32 : SInst<"vrnd", "..", "fQf">;
def FRINTI_S32 : SInst<"vrndi", "..", "fQf">;
}

let ArchGuard = "(defined(__aarch64__) || defined(__arm64ec__)) && defined(__ARM_FEATURE_DIRECTED_ROUNDING)" in {
def FRINTN_S64 : SInst<"vrndn", "..", "dQd">;
````
- **L1205 EN**: Closes the current lexical scope or compound statement.
  **L1205 CN**: 结束当前词法作用域或复合语句块。
- **L1206 EN**: Blank line separating nearby declarations or logic blocks.
  **L1206 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1207 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)" in {`.
  **L1207 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)" in {`。
- **L1208 EN**: Declares TableGen def record `FCVTNS_S64`.
  **L1208 CN**: 声明 TableGen def 记录 `FCVTNS_S64`。
- **L1209 EN**: Declares TableGen def record `FCVTNU_S64`.
  **L1209 CN**: 声明 TableGen def 记录 `FCVTNU_S64`。
- **L1210 EN**: Declares TableGen def record `FCVTPS_S64`.
  **L1210 CN**: 声明 TableGen def 记录 `FCVTPS_S64`。
- **L1211 EN**: Declares TableGen def record `FCVTPU_S64`.
  **L1211 CN**: 声明 TableGen def 记录 `FCVTPU_S64`。
- **L1212 EN**: Declares TableGen def record `FCVTMS_S64`.
  **L1212 CN**: 声明 TableGen def 记录 `FCVTMS_S64`。
- **L1213 EN**: Declares TableGen def record `FCVTMU_S64`.
  **L1213 CN**: 声明 TableGen def 记录 `FCVTMU_S64`。
- **L1214 EN**: Declares TableGen def record `FCVTAS_S64`.
  **L1214 CN**: 声明 TableGen def 记录 `FCVTAS_S64`。
- **L1215 EN**: Declares TableGen def record `FCVTAU_S64`.
  **L1215 CN**: 声明 TableGen def 记录 `FCVTAU_S64`。
- **L1216 EN**: Closes the current lexical scope or compound statement.
  **L1216 CN**: 结束当前词法作用域或复合语句块。
- **L1217 EN**: Blank line separating nearby declarations or logic blocks.
  **L1217 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1218 EN**: Separator comment used for visual grouping.
  **L1218 CN**: 用于视觉分组的分隔注释。
- **L1219 EN**: Comment explains nearby logic, constraints, or intent: `Round to Integral`.
  **L1219 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Round to Integral`。
- **L1220 EN**: Blank line separating nearby declarations or logic blocks.
  **L1220 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1221 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "__ARM_ARCH >= 8 && defined(__ARM_FEATURE_DIRECTED_ROUNDING)" in {`.
  **L1221 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "__ARM_ARCH >= 8 && defined(__ARM_FEATURE_DIRECTED_ROUNDING)" in {`。
- **L1222 EN**: Declares TableGen def record `FRINTN_S32`.
  **L1222 CN**: 声明 TableGen def 记录 `FRINTN_S32`。
- **L1223 EN**: Declares TableGen def record `FRINTA_S32`.
  **L1223 CN**: 声明 TableGen def 记录 `FRINTA_S32`。
- **L1224 EN**: Declares TableGen def record `FRINTP_S32`.
  **L1224 CN**: 声明 TableGen def 记录 `FRINTP_S32`。
- **L1225 EN**: Declares TableGen def record `FRINTM_S32`.
  **L1225 CN**: 声明 TableGen def 记录 `FRINTM_S32`。
- **L1226 EN**: Declares TableGen def record `FRINTX_S32`.
  **L1226 CN**: 声明 TableGen def 记录 `FRINTX_S32`。
- **L1227 EN**: Declares TableGen def record `FRINTZ_S32`.
  **L1227 CN**: 声明 TableGen def 记录 `FRINTZ_S32`。
- **L1228 EN**: Declares TableGen def record `FRINTI_S32`.
  **L1228 CN**: 声明 TableGen def 记录 `FRINTI_S32`。
- **L1229 EN**: Closes the current lexical scope or compound statement.
  **L1229 CN**: 结束当前词法作用域或复合语句块。
- **L1230 EN**: Blank line separating nearby declarations or logic blocks.
  **L1230 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1231 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "(defined(__aarch64__) || defined(__arm64ec__)) && defined(__ARM_FEATURE_DIRECTED_ROUNDING)" in {`.
  **L1231 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "(defined(__aarch64__) || defined(__arm64ec__)) && defined(__ARM_FEATURE_DIRECTED_ROUNDING)" in {`。
- **L1232 EN**: Declares TableGen def record `FRINTN_S64`.
  **L1232 CN**: 声明 TableGen def 记录 `FRINTN_S64`。

### Lines 1233-1260

````tablegen
def FRINTA_S64 : SInst<"vrnda", "..", "dQd">;
def FRINTP_S64 : SInst<"vrndp", "..", "dQd">;
def FRINTM_S64 : SInst<"vrndm", "..", "dQd">;
def FRINTX_S64 : SInst<"vrndx", "..", "dQd">;
def FRINTZ_S64 : SInst<"vrnd", "..", "dQd">;
def FRINTI_S64 : SInst<"vrndi", "..", "dQd">;
}

let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "v8.5a,neon" in {
def FRINT32X_S32 : SInst<"vrnd32x", "..", "fQf">;
def FRINT32Z_S32 : SInst<"vrnd32z", "..", "fQf">;
def FRINT64X_S32 : SInst<"vrnd64x", "..", "fQf">;
def FRINT64Z_S32 : SInst<"vrnd64z", "..", "fQf">;

def FRINT32X_S64 : SInst<"vrnd32x", "..", "dQd">;
def FRINT32Z_S64 : SInst<"vrnd32z", "..", "dQd">;
def FRINT64X_S64 : SInst<"vrnd64x", "..", "dQd">;
def FRINT64Z_S64 : SInst<"vrnd64z", "..", "dQd">;
}

////////////////////////////////////////////////////////////////////////////////
// MaxNum/MinNum Floating Point

let ArchGuard = "__ARM_ARCH >= 8 && defined(__ARM_FEATURE_NUMERIC_MAXMIN)" in {
def FMAXNM_S32 : SInst<"vmaxnm", "...", "fQf">;
def FMINNM_S32 : SInst<"vminnm", "...", "fQf">;
}

````
- **L1233 EN**: Declares TableGen def record `FRINTA_S64`.
  **L1233 CN**: 声明 TableGen def 记录 `FRINTA_S64`。
- **L1234 EN**: Declares TableGen def record `FRINTP_S64`.
  **L1234 CN**: 声明 TableGen def 记录 `FRINTP_S64`。
- **L1235 EN**: Declares TableGen def record `FRINTM_S64`.
  **L1235 CN**: 声明 TableGen def 记录 `FRINTM_S64`。
- **L1236 EN**: Declares TableGen def record `FRINTX_S64`.
  **L1236 CN**: 声明 TableGen def 记录 `FRINTX_S64`。
- **L1237 EN**: Declares TableGen def record `FRINTZ_S64`.
  **L1237 CN**: 声明 TableGen def 记录 `FRINTZ_S64`。
- **L1238 EN**: Declares TableGen def record `FRINTI_S64`.
  **L1238 CN**: 声明 TableGen def 记录 `FRINTI_S64`。
- **L1239 EN**: Closes the current lexical scope or compound statement.
  **L1239 CN**: 结束当前词法作用域或复合语句块。
- **L1240 EN**: Blank line separating nearby declarations or logic blocks.
  **L1240 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1241 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "v8.5a,neon" in {`.
  **L1241 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "v8.5a,neon" in {`。
- **L1242 EN**: Declares TableGen def record `FRINT32X_S32`.
  **L1242 CN**: 声明 TableGen def 记录 `FRINT32X_S32`。
- **L1243 EN**: Declares TableGen def record `FRINT32Z_S32`.
  **L1243 CN**: 声明 TableGen def 记录 `FRINT32Z_S32`。
- **L1244 EN**: Declares TableGen def record `FRINT64X_S32`.
  **L1244 CN**: 声明 TableGen def 记录 `FRINT64X_S32`。
- **L1245 EN**: Declares TableGen def record `FRINT64Z_S32`.
  **L1245 CN**: 声明 TableGen def 记录 `FRINT64Z_S32`。
- **L1246 EN**: Blank line separating nearby declarations or logic blocks.
  **L1246 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1247 EN**: Declares TableGen def record `FRINT32X_S64`.
  **L1247 CN**: 声明 TableGen def 记录 `FRINT32X_S64`。
- **L1248 EN**: Declares TableGen def record `FRINT32Z_S64`.
  **L1248 CN**: 声明 TableGen def 记录 `FRINT32Z_S64`。
- **L1249 EN**: Declares TableGen def record `FRINT64X_S64`.
  **L1249 CN**: 声明 TableGen def 记录 `FRINT64X_S64`。
- **L1250 EN**: Declares TableGen def record `FRINT64Z_S64`.
  **L1250 CN**: 声明 TableGen def 记录 `FRINT64Z_S64`。
- **L1251 EN**: Closes the current lexical scope or compound statement.
  **L1251 CN**: 结束当前词法作用域或复合语句块。
- **L1252 EN**: Blank line separating nearby declarations or logic blocks.
  **L1252 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1253 EN**: Separator comment used for visual grouping.
  **L1253 CN**: 用于视觉分组的分隔注释。
- **L1254 EN**: Comment explains nearby logic, constraints, or intent: `MaxNum/MinNum Floating Point`.
  **L1254 CN**: 注释解释附近代码的逻辑、约束或设计意图：`MaxNum/MinNum Floating Point`。
- **L1255 EN**: Blank line separating nearby declarations or logic blocks.
  **L1255 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1256 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "__ARM_ARCH >= 8 && defined(__ARM_FEATURE_NUMERIC_MAXMIN)" in {`.
  **L1256 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "__ARM_ARCH >= 8 && defined(__ARM_FEATURE_NUMERIC_MAXMIN)" in {`。
- **L1257 EN**: Declares TableGen def record `FMAXNM_S32`.
  **L1257 CN**: 声明 TableGen def 记录 `FMAXNM_S32`。
- **L1258 EN**: Declares TableGen def record `FMINNM_S32`.
  **L1258 CN**: 声明 TableGen def 记录 `FMINNM_S32`。
- **L1259 EN**: Closes the current lexical scope or compound statement.
  **L1259 CN**: 结束当前词法作用域或复合语句块。
- **L1260 EN**: Blank line separating nearby declarations or logic blocks.
  **L1260 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1261-1288

````tablegen
let ArchGuard = "(defined(__aarch64__)  || defined(__arm64ec__)) && defined(__ARM_FEATURE_NUMERIC_MAXMIN)" in {
def FMAXNM_S64 : SInst<"vmaxnm", "...", "dQd">;
def FMINNM_S64 : SInst<"vminnm", "...", "dQd">;
}

////////////////////////////////////////////////////////////////////////////////
// Permutation
def VTRN1 : SOpInst<"vtrn1", "...",
                    "csiUcUsUifPcPsQcQsQiQlQUcQUsQUiQUlQfQdQPcQPsQPlmQm", OP_TRN1>;
def VZIP1 : SOpInst<"vzip1", "...",
                    "csiUcUsUifPcPsQcQsQiQlQUcQUsQUiQUlQfQdQPcQPsQPlmQm", OP_ZIP1>;
def VUZP1 : SOpInst<"vuzp1", "...",
                    "csiUcUsUifPcPsQcQsQiQlQUcQUsQUiQUlQfQdQPcQPsQPlmQm", OP_UZP1>;
def VTRN2 : SOpInst<"vtrn2", "...",
                    "csiUcUsUifPcPsQcQsQiQlQUcQUsQUiQUlQfQdQPcQPsQPlmQm", OP_TRN2>;
def VZIP2 : SOpInst<"vzip2", "...",
                    "csiUcUsUifPcPsQcQsQiQlQUcQUsQUiQUlQfQdQPcQPsQPlmQm", OP_ZIP2>;
def VUZP2 : SOpInst<"vuzp2", "...",
                    "csiUcUsUifPcPsQcQsQiQlQUcQUsQUiQUlQfQdQPcQPsQPlmQm", OP_UZP2>;

////////////////////////////////////////////////////////////////////////////////
// Table lookup
let InstName = "vtbl" in {
def VQTBL1_A64 : WInst<"vqtbl1", ".QU",  "UccPcQUcQcQPcmQm">;
def VQTBL2_A64 : WInst<"vqtbl2", ".(2Q)U",  "UccPcQUcQcQPcmQm">;
def VQTBL3_A64 : WInst<"vqtbl3", ".(3Q)U",  "UccPcQUcQcQPcmQm">;
def VQTBL4_A64 : WInst<"vqtbl4", ".(4Q)U",  "UccPcQUcQcQPcmQm">;
}
````
- **L1261 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "(defined(__aarch64__)  || defined(__arm64ec__)) && defined(__ARM_FEATURE_NUMERIC_MAXMIN)" in {`.
  **L1261 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "(defined(__aarch64__)  || defined(__arm64ec__)) && defined(__ARM_FEATURE_NUMERIC_MAXMIN)" in {`。
- **L1262 EN**: Declares TableGen def record `FMAXNM_S64`.
  **L1262 CN**: 声明 TableGen def 记录 `FMAXNM_S64`。
- **L1263 EN**: Declares TableGen def record `FMINNM_S64`.
  **L1263 CN**: 声明 TableGen def 记录 `FMINNM_S64`。
- **L1264 EN**: Closes the current lexical scope or compound statement.
  **L1264 CN**: 结束当前词法作用域或复合语句块。
- **L1265 EN**: Blank line separating nearby declarations or logic blocks.
  **L1265 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1266 EN**: Separator comment used for visual grouping.
  **L1266 CN**: 用于视觉分组的分隔注释。
- **L1267 EN**: Comment explains nearby logic, constraints, or intent: `Permutation`.
  **L1267 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Permutation`。
- **L1268 EN**: Declares TableGen def record `VTRN1`.
  **L1268 CN**: 声明 TableGen def 记录 `VTRN1`。
- **L1269 EN**: Adds a standalone statement or declaration: `"csiUcUsUifPcPsQcQsQiQlQUcQUsQUiQUlQfQdQPcQPsQPlmQm", OP_TRN1>;`.
  **L1269 CN**: 添加一条独立语句或声明：`"csiUcUsUifPcPsQcQsQiQlQUcQUsQUiQUlQfQdQPcQPsQPlmQm", OP_TRN1>;`。
- **L1270 EN**: Declares TableGen def record `VZIP1`.
  **L1270 CN**: 声明 TableGen def 记录 `VZIP1`。
- **L1271 EN**: Adds a standalone statement or declaration: `"csiUcUsUifPcPsQcQsQiQlQUcQUsQUiQUlQfQdQPcQPsQPlmQm", OP_ZIP1>;`.
  **L1271 CN**: 添加一条独立语句或声明：`"csiUcUsUifPcPsQcQsQiQlQUcQUsQUiQUlQfQdQPcQPsQPlmQm", OP_ZIP1>;`。
- **L1272 EN**: Declares TableGen def record `VUZP1`.
  **L1272 CN**: 声明 TableGen def 记录 `VUZP1`。
- **L1273 EN**: Adds a standalone statement or declaration: `"csiUcUsUifPcPsQcQsQiQlQUcQUsQUiQUlQfQdQPcQPsQPlmQm", OP_UZP1>;`.
  **L1273 CN**: 添加一条独立语句或声明：`"csiUcUsUifPcPsQcQsQiQlQUcQUsQUiQUlQfQdQPcQPsQPlmQm", OP_UZP1>;`。
- **L1274 EN**: Declares TableGen def record `VTRN2`.
  **L1274 CN**: 声明 TableGen def 记录 `VTRN2`。
- **L1275 EN**: Adds a standalone statement or declaration: `"csiUcUsUifPcPsQcQsQiQlQUcQUsQUiQUlQfQdQPcQPsQPlmQm", OP_TRN2>;`.
  **L1275 CN**: 添加一条独立语句或声明：`"csiUcUsUifPcPsQcQsQiQlQUcQUsQUiQUlQfQdQPcQPsQPlmQm", OP_TRN2>;`。
- **L1276 EN**: Declares TableGen def record `VZIP2`.
  **L1276 CN**: 声明 TableGen def 记录 `VZIP2`。
- **L1277 EN**: Adds a standalone statement or declaration: `"csiUcUsUifPcPsQcQsQiQlQUcQUsQUiQUlQfQdQPcQPsQPlmQm", OP_ZIP2>;`.
  **L1277 CN**: 添加一条独立语句或声明：`"csiUcUsUifPcPsQcQsQiQlQUcQUsQUiQUlQfQdQPcQPsQPlmQm", OP_ZIP2>;`。
- **L1278 EN**: Declares TableGen def record `VUZP2`.
  **L1278 CN**: 声明 TableGen def 记录 `VUZP2`。
- **L1279 EN**: Adds a standalone statement or declaration: `"csiUcUsUifPcPsQcQsQiQlQUcQUsQUiQUlQfQdQPcQPsQPlmQm", OP_UZP2>;`.
  **L1279 CN**: 添加一条独立语句或声明：`"csiUcUsUifPcPsQcQsQiQlQUcQUsQUiQUlQfQdQPcQPsQPlmQm", OP_UZP2>;`。
- **L1280 EN**: Blank line separating nearby declarations or logic blocks.
  **L1280 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1281 EN**: Separator comment used for visual grouping.
  **L1281 CN**: 用于视觉分组的分隔注释。
- **L1282 EN**: Comment explains nearby logic, constraints, or intent: `Table lookup`.
  **L1282 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Table lookup`。
- **L1283 EN**: Assigns a TableGen property that affects following records or inherited fields: `let InstName = "vtbl" in {`.
  **L1283 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let InstName = "vtbl" in {`。
- **L1284 EN**: Declares TableGen def record `VQTBL1_A64`.
  **L1284 CN**: 声明 TableGen def 记录 `VQTBL1_A64`。
- **L1285 EN**: Declares TableGen def record `VQTBL2_A64`.
  **L1285 CN**: 声明 TableGen def 记录 `VQTBL2_A64`。
- **L1286 EN**: Declares TableGen def record `VQTBL3_A64`.
  **L1286 CN**: 声明 TableGen def 记录 `VQTBL3_A64`。
- **L1287 EN**: Declares TableGen def record `VQTBL4_A64`.
  **L1287 CN**: 声明 TableGen def 记录 `VQTBL4_A64`。
- **L1288 EN**: Closes the current lexical scope or compound statement.
  **L1288 CN**: 结束当前词法作用域或复合语句块。

### Lines 1289-1316

````tablegen
let InstName = "vtbx" in {
def VQTBX1_A64 : WInst<"vqtbx1", "..QU", "UccPcQUcQcQPcmQm">;
def VQTBX2_A64 : WInst<"vqtbx2", "..(2Q)U", "UccPcQUcQcQPcmQm">;
def VQTBX3_A64 : WInst<"vqtbx3", "..(3Q)U", "UccPcQUcQcQPcmQm">;
def VQTBX4_A64 : WInst<"vqtbx4", "..(4Q)U", "UccPcQUcQcQPcmQm">;
}

////////////////////////////////////////////////////////////////////////////////
// Vector reinterpret cast operations

// NeonEmitter implicitly takes the cartesian product of the type string with
// itself during generation so, unlike all other intrinsics, this one should
// include *all* types, not just additional ones.
def VVREINTERPRET : REINTERPRET_CROSS_SELF<"csilUcUsUiUlmhfdPcPsPlQcQsQiQlQUcQUsQUiQUlQmQhQfQdQPcQPsQPlQPk"> {
  let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)";
  let BigEndianSafe = 1;
}

////////////////////////////////////////////////////////////////////////////////
// Scalar Intrinsics
// Scalar Arithmetic

// Scalar Addition
def SCALAR_ADD : SInst<"vadd", "111",  "SlSUl">;
// Scalar  Saturating Add
def SCALAR_QADD   : SInst<"vqadd", "111", "ScSsSiSlSUcSUsSUiSUl">;

// Scalar Subtraction
````
- **L1289 EN**: Assigns a TableGen property that affects following records or inherited fields: `let InstName = "vtbx" in {`.
  **L1289 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let InstName = "vtbx" in {`。
- **L1290 EN**: Declares TableGen def record `VQTBX1_A64`.
  **L1290 CN**: 声明 TableGen def 记录 `VQTBX1_A64`。
- **L1291 EN**: Declares TableGen def record `VQTBX2_A64`.
  **L1291 CN**: 声明 TableGen def 记录 `VQTBX2_A64`。
- **L1292 EN**: Declares TableGen def record `VQTBX3_A64`.
  **L1292 CN**: 声明 TableGen def 记录 `VQTBX3_A64`。
- **L1293 EN**: Declares TableGen def record `VQTBX4_A64`.
  **L1293 CN**: 声明 TableGen def 记录 `VQTBX4_A64`。
- **L1294 EN**: Closes the current lexical scope or compound statement.
  **L1294 CN**: 结束当前词法作用域或复合语句块。
- **L1295 EN**: Blank line separating nearby declarations or logic blocks.
  **L1295 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1296 EN**: Separator comment used for visual grouping.
  **L1296 CN**: 用于视觉分组的分隔注释。
- **L1297 EN**: Comment explains nearby logic, constraints, or intent: `Vector reinterpret cast operations`.
  **L1297 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Vector reinterpret cast operations`。
- **L1298 EN**: Blank line separating nearby declarations or logic blocks.
  **L1298 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1299 EN**: Comment explains nearby logic, constraints, or intent: `NeonEmitter implicitly takes the cartesian product of the type string with`.
  **L1299 CN**: 注释解释附近代码的逻辑、约束或设计意图：`NeonEmitter implicitly takes the cartesian product of the type string with`。
- **L1300 EN**: Comment explains nearby logic, constraints, or intent: `itself during generation so, unlike all other intrinsics, this one should`.
  **L1300 CN**: 注释解释附近代码的逻辑、约束或设计意图：`itself during generation so, unlike all other intrinsics, this one should`。
- **L1301 EN**: Comment explains nearby logic, constraints, or intent: `include *all* types, not just additional ones.`.
  **L1301 CN**: 注释解释附近代码的逻辑、约束或设计意图：`include *all* types, not just additional ones.`。
- **L1302 EN**: Declares TableGen def record `VVREINTERPRET`.
  **L1302 CN**: 声明 TableGen def 记录 `VVREINTERPRET`。
- **L1303 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)";`.
  **L1303 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)";`。
- **L1304 EN**: Assigns a TableGen property that affects following records or inherited fields: `let BigEndianSafe = 1;`.
  **L1304 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let BigEndianSafe = 1;`。
- **L1305 EN**: Closes the current lexical scope or compound statement.
  **L1305 CN**: 结束当前词法作用域或复合语句块。
- **L1306 EN**: Blank line separating nearby declarations or logic blocks.
  **L1306 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1307 EN**: Separator comment used for visual grouping.
  **L1307 CN**: 用于视觉分组的分隔注释。
- **L1308 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Intrinsics`.
  **L1308 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Intrinsics`。
- **L1309 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Arithmetic`.
  **L1309 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Arithmetic`。
- **L1310 EN**: Blank line separating nearby declarations or logic blocks.
  **L1310 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1311 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Addition`.
  **L1311 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Addition`。
- **L1312 EN**: Declares TableGen def record `SCALAR_ADD`.
  **L1312 CN**: 声明 TableGen def 记录 `SCALAR_ADD`。
- **L1313 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Saturating Add`.
  **L1313 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Saturating Add`。
- **L1314 EN**: Declares TableGen def record `SCALAR_QADD`.
  **L1314 CN**: 声明 TableGen def 记录 `SCALAR_QADD`。
- **L1315 EN**: Blank line separating nearby declarations or logic blocks.
  **L1315 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1316 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Subtraction`.
  **L1316 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Subtraction`。

### Lines 1317-1344

````tablegen
def SCALAR_SUB : SInst<"vsub", "111",  "SlSUl">;
// Scalar  Saturating Sub
def SCALAR_QSUB   : SInst<"vqsub", "111", "ScSsSiSlSUcSUsSUiSUl">;

let InstName = "vmov" in {
def VGET_HIGH_A64 : NoTestOpInst<"vget_high", ".Q", "dPl", OP_HI>;
def VGET_LOW_A64  : NoTestOpInst<"vget_low", ".Q", "dPl", OP_LO>;
}

////////////////////////////////////////////////////////////////////////////////
// Scalar Shift
// Scalar Shift Left
def SCALAR_SHL: SInst<"vshl", "11(S1)", "SlSUl">;
// Scalar Saturating Shift Left
def SCALAR_QSHL: SInst<"vqshl", "11(S1)", "ScSsSiSlSUcSUsSUiSUl">;
// Scalar Saturating Rounding Shift Left
def SCALAR_QRSHL: SInst<"vqrshl", "11(S1)", "ScSsSiSlSUcSUsSUiSUl">;
// Scalar Shift Rounding Left
def SCALAR_RSHL: SInst<"vrshl", "11(S1)", "SlSUl">;

////////////////////////////////////////////////////////////////////////////////
// Scalar Shift (Immediate)
let isScalarShift = 1 in {
// Signed/Unsigned Shift Right (Immediate)
def SCALAR_SSHR_N: SInst<"vshr_n", "11I", "SlSUl",
                        [ImmCheck<1, ImmCheckShiftRight, 0>]>;
// Signed/Unsigned Rounding Shift Right (Immediate)
def SCALAR_SRSHR_N: SInst<"vrshr_n", "11I", "SlSUl",
````
- **L1317 EN**: Declares TableGen def record `SCALAR_SUB`.
  **L1317 CN**: 声明 TableGen def 记录 `SCALAR_SUB`。
- **L1318 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Saturating Sub`.
  **L1318 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Saturating Sub`。
- **L1319 EN**: Declares TableGen def record `SCALAR_QSUB`.
  **L1319 CN**: 声明 TableGen def 记录 `SCALAR_QSUB`。
- **L1320 EN**: Blank line separating nearby declarations or logic blocks.
  **L1320 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1321 EN**: Assigns a TableGen property that affects following records or inherited fields: `let InstName = "vmov" in {`.
  **L1321 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let InstName = "vmov" in {`。
- **L1322 EN**: Declares TableGen def record `VGET_HIGH_A64`.
  **L1322 CN**: 声明 TableGen def 记录 `VGET_HIGH_A64`。
- **L1323 EN**: Declares TableGen def record `VGET_LOW_A64`.
  **L1323 CN**: 声明 TableGen def 记录 `VGET_LOW_A64`。
- **L1324 EN**: Closes the current lexical scope or compound statement.
  **L1324 CN**: 结束当前词法作用域或复合语句块。
- **L1325 EN**: Blank line separating nearby declarations or logic blocks.
  **L1325 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1326 EN**: Separator comment used for visual grouping.
  **L1326 CN**: 用于视觉分组的分隔注释。
- **L1327 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Shift`.
  **L1327 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Shift`。
- **L1328 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Shift Left`.
  **L1328 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Shift Left`。
- **L1329 EN**: Declares TableGen def record `SCALAR_SHL`.
  **L1329 CN**: 声明 TableGen def 记录 `SCALAR_SHL`。
- **L1330 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Saturating Shift Left`.
  **L1330 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Saturating Shift Left`。
- **L1331 EN**: Declares TableGen def record `SCALAR_QSHL`.
  **L1331 CN**: 声明 TableGen def 记录 `SCALAR_QSHL`。
- **L1332 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Saturating Rounding Shift Left`.
  **L1332 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Saturating Rounding Shift Left`。
- **L1333 EN**: Declares TableGen def record `SCALAR_QRSHL`.
  **L1333 CN**: 声明 TableGen def 记录 `SCALAR_QRSHL`。
- **L1334 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Shift Rounding Left`.
  **L1334 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Shift Rounding Left`。
- **L1335 EN**: Declares TableGen def record `SCALAR_RSHL`.
  **L1335 CN**: 声明 TableGen def 记录 `SCALAR_RSHL`。
- **L1336 EN**: Blank line separating nearby declarations or logic blocks.
  **L1336 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1337 EN**: Separator comment used for visual grouping.
  **L1337 CN**: 用于视觉分组的分隔注释。
- **L1338 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Shift (Immediate)`.
  **L1338 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Shift (Immediate)`。
- **L1339 EN**: Assigns a TableGen property that affects following records or inherited fields: `let isScalarShift = 1 in {`.
  **L1339 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let isScalarShift = 1 in {`。
- **L1340 EN**: Comment explains nearby logic, constraints, or intent: `Signed/Unsigned Shift Right (Immediate)`.
  **L1340 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Signed/Unsigned Shift Right (Immediate)`。
- **L1341 EN**: Declares TableGen def record `SCALAR_SSHR_N`.
  **L1341 CN**: 声明 TableGen def 记录 `SCALAR_SSHR_N`。
- **L1342 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheckShiftRight, 0>]>;`.
  **L1342 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheckShiftRight, 0>]>;`。
- **L1343 EN**: Comment explains nearby logic, constraints, or intent: `Signed/Unsigned Rounding Shift Right (Immediate)`.
  **L1343 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Signed/Unsigned Rounding Shift Right (Immediate)`。
- **L1344 EN**: Declares TableGen def record `SCALAR_SRSHR_N`.
  **L1344 CN**: 声明 TableGen def 记录 `SCALAR_SRSHR_N`。

### Lines 1345-1372

````tablegen
                          [ImmCheck<1, ImmCheckShiftRight, 0>]>;

// Signed/Unsigned Shift Right and Accumulate (Immediate)
def SCALAR_SSRA_N: SInst<"vsra_n", "111I", "SlSUl",
                        [ImmCheck<2, ImmCheckShiftRight, 0>]>;
// Signed/Unsigned Rounding Shift Right and Accumulate (Immediate)
def SCALAR_SRSRA_N: SInst<"vrsra_n", "111I", "SlSUl",
                        [ImmCheck<2, ImmCheckShiftRight, 0>]>;

// Shift Left (Immediate)
def SCALAR_SHL_N: SInst<"vshl_n", "11I", "SlSUl",
                      [ImmCheck<1, ImmCheckShiftLeft, 0>]>;
// Signed/Unsigned Saturating Shift Left (Immediate)
def SCALAR_SQSHL_N: SInst<"vqshl_n", "11I", "ScSsSiSlSUcSUsSUiSUl",
                      [ImmCheck<1, ImmCheckShiftLeft, 0>]>;
// Signed Saturating Shift Left Unsigned (Immediate)
def SCALAR_SQSHLU_N: SInst<"vqshlu_n", "11I", "ScSsSiSl",
                      [ImmCheck<1, ImmCheckShiftLeft, 0>]>;

// Shift Right And Insert (Immediate)
def SCALAR_SRI_N: SInst<"vsri_n", "111I", "SlSUl",
                        [ImmCheck<2, ImmCheckShiftRight, 0>]>;
// Shift Left And Insert (Immediate)
def SCALAR_SLI_N: SInst<"vsli_n", "111I", "SlSUl",
                        [ImmCheck<2, ImmCheckShiftLeft, 0>]>;

let isScalarNarrowShift = 1 in {
  // Signed/Unsigned Saturating Shift Right Narrow (Immediate)
````
- **L1345 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheckShiftRight, 0>]>;`.
  **L1345 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheckShiftRight, 0>]>;`。
- **L1346 EN**: Blank line separating nearby declarations or logic blocks.
  **L1346 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1347 EN**: Comment explains nearby logic, constraints, or intent: `Signed/Unsigned Shift Right and Accumulate (Immediate)`.
  **L1347 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Signed/Unsigned Shift Right and Accumulate (Immediate)`。
- **L1348 EN**: Declares TableGen def record `SCALAR_SSRA_N`.
  **L1348 CN**: 声明 TableGen def 记录 `SCALAR_SSRA_N`。
- **L1349 EN**: Adds a standalone statement or declaration: `[ImmCheck<2, ImmCheckShiftRight, 0>]>;`.
  **L1349 CN**: 添加一条独立语句或声明：`[ImmCheck<2, ImmCheckShiftRight, 0>]>;`。
- **L1350 EN**: Comment explains nearby logic, constraints, or intent: `Signed/Unsigned Rounding Shift Right and Accumulate (Immediate)`.
  **L1350 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Signed/Unsigned Rounding Shift Right and Accumulate (Immediate)`。
- **L1351 EN**: Declares TableGen def record `SCALAR_SRSRA_N`.
  **L1351 CN**: 声明 TableGen def 记录 `SCALAR_SRSRA_N`。
- **L1352 EN**: Adds a standalone statement or declaration: `[ImmCheck<2, ImmCheckShiftRight, 0>]>;`.
  **L1352 CN**: 添加一条独立语句或声明：`[ImmCheck<2, ImmCheckShiftRight, 0>]>;`。
- **L1353 EN**: Blank line separating nearby declarations or logic blocks.
  **L1353 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1354 EN**: Comment explains nearby logic, constraints, or intent: `Shift Left (Immediate)`.
  **L1354 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Shift Left (Immediate)`。
- **L1355 EN**: Declares TableGen def record `SCALAR_SHL_N`.
  **L1355 CN**: 声明 TableGen def 记录 `SCALAR_SHL_N`。
- **L1356 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheckShiftLeft, 0>]>;`.
  **L1356 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheckShiftLeft, 0>]>;`。
- **L1357 EN**: Comment explains nearby logic, constraints, or intent: `Signed/Unsigned Saturating Shift Left (Immediate)`.
  **L1357 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Signed/Unsigned Saturating Shift Left (Immediate)`。
- **L1358 EN**: Declares TableGen def record `SCALAR_SQSHL_N`.
  **L1358 CN**: 声明 TableGen def 记录 `SCALAR_SQSHL_N`。
- **L1359 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheckShiftLeft, 0>]>;`.
  **L1359 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheckShiftLeft, 0>]>;`。
- **L1360 EN**: Comment explains nearby logic, constraints, or intent: `Signed Saturating Shift Left Unsigned (Immediate)`.
  **L1360 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Signed Saturating Shift Left Unsigned (Immediate)`。
- **L1361 EN**: Declares TableGen def record `SCALAR_SQSHLU_N`.
  **L1361 CN**: 声明 TableGen def 记录 `SCALAR_SQSHLU_N`。
- **L1362 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheckShiftLeft, 0>]>;`.
  **L1362 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheckShiftLeft, 0>]>;`。
- **L1363 EN**: Blank line separating nearby declarations or logic blocks.
  **L1363 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1364 EN**: Comment explains nearby logic, constraints, or intent: `Shift Right And Insert (Immediate)`.
  **L1364 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Shift Right And Insert (Immediate)`。
- **L1365 EN**: Declares TableGen def record `SCALAR_SRI_N`.
  **L1365 CN**: 声明 TableGen def 记录 `SCALAR_SRI_N`。
- **L1366 EN**: Adds a standalone statement or declaration: `[ImmCheck<2, ImmCheckShiftRight, 0>]>;`.
  **L1366 CN**: 添加一条独立语句或声明：`[ImmCheck<2, ImmCheckShiftRight, 0>]>;`。
- **L1367 EN**: Comment explains nearby logic, constraints, or intent: `Shift Left And Insert (Immediate)`.
  **L1367 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Shift Left And Insert (Immediate)`。
- **L1368 EN**: Declares TableGen def record `SCALAR_SLI_N`.
  **L1368 CN**: 声明 TableGen def 记录 `SCALAR_SLI_N`。
- **L1369 EN**: Adds a standalone statement or declaration: `[ImmCheck<2, ImmCheckShiftLeft, 0>]>;`.
  **L1369 CN**: 添加一条独立语句或声明：`[ImmCheck<2, ImmCheckShiftLeft, 0>]>;`。
- **L1370 EN**: Blank line separating nearby declarations or logic blocks.
  **L1370 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1371 EN**: Assigns a TableGen property that affects following records or inherited fields: `let isScalarNarrowShift = 1 in {`.
  **L1371 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let isScalarNarrowShift = 1 in {`。
- **L1372 EN**: Comment explains nearby logic, constraints, or intent: `Signed/Unsigned Saturating Shift Right Narrow (Immediate)`.
  **L1372 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Signed/Unsigned Saturating Shift Right Narrow (Immediate)`。

### Lines 1373-1400

````tablegen
  def SCALAR_SQSHRN_N: SInst<"vqshrn_n", "(1<)1I", "SsSiSlSUsSUiSUl",
                            [ImmCheck<1, ImmCheckShiftRightNarrow, 0>]>;
  // Signed/Unsigned Saturating Rounded Shift Right Narrow (Immediate)
  def SCALAR_SQRSHRN_N: SInst<"vqrshrn_n", "(1<)1I", "SsSiSlSUsSUiSUl",
                            [ImmCheck<1, ImmCheckShiftRightNarrow, 0>]>;
  // Signed Saturating Shift Right Unsigned Narrow (Immediate)
  def SCALAR_SQSHRUN_N: SInst<"vqshrun_n", "(1<U)1I", "SsSiSl",
                            [ImmCheck<1, ImmCheckShiftRightNarrow, 0>]>;
  // Signed Saturating Rounded Shift Right Unsigned Narrow (Immediate)
  def SCALAR_SQRSHRUN_N: SInst<"vqrshrun_n", "(1<U)1I", "SsSiSl",
                            [ImmCheck<1, ImmCheckShiftRightNarrow, 0>]>;
}

////////////////////////////////////////////////////////////////////////////////
// Scalar Signed/Unsigned Fixed-point Convert To Floating-Point (Immediate)
def SCALAR_SCVTF_N_F32: SInst<"vcvt_n_f32", "(1F)(1!)I", "SiSUi",
                              [ImmCheck<1, ImmCheck1_32>]>;
def SCALAR_SCVTF_N_F64: SInst<"vcvt_n_f64", "(1F)(1!)I", "SlSUl",
                              [ImmCheck<1, ImmCheck1_64>]>;

////////////////////////////////////////////////////////////////////////////////
// Scalar Floating-point Convert To Signed/Unsigned Fixed-point (Immediate)
def SCALAR_FCVTZS_N_S32 : SInst<"vcvt_n_s32", "(1S)1I", "Sf",
                                [ImmCheck<1, ImmCheck1_32>]>;
def SCALAR_FCVTZU_N_U32 : SInst<"vcvt_n_u32", "(1U)1I", "Sf",
                                [ImmCheck<1, ImmCheck1_32>]>;
def SCALAR_FCVTZS_N_S64 : SInst<"vcvt_n_s64", "(1S)1I", "Sd",
                                [ImmCheck<1, ImmCheck1_64>]>;
````
- **L1373 EN**: Declares TableGen def record `SCALAR_SQSHRN_N`.
  **L1373 CN**: 声明 TableGen def 记录 `SCALAR_SQSHRN_N`。
- **L1374 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheckShiftRightNarrow, 0>]>;`.
  **L1374 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheckShiftRightNarrow, 0>]>;`。
- **L1375 EN**: Comment explains nearby logic, constraints, or intent: `Signed/Unsigned Saturating Rounded Shift Right Narrow (Immediate)`.
  **L1375 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Signed/Unsigned Saturating Rounded Shift Right Narrow (Immediate)`。
- **L1376 EN**: Declares TableGen def record `SCALAR_SQRSHRN_N`.
  **L1376 CN**: 声明 TableGen def 记录 `SCALAR_SQRSHRN_N`。
- **L1377 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheckShiftRightNarrow, 0>]>;`.
  **L1377 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheckShiftRightNarrow, 0>]>;`。
- **L1378 EN**: Comment explains nearby logic, constraints, or intent: `Signed Saturating Shift Right Unsigned Narrow (Immediate)`.
  **L1378 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Signed Saturating Shift Right Unsigned Narrow (Immediate)`。
- **L1379 EN**: Declares TableGen def record `SCALAR_SQSHRUN_N`.
  **L1379 CN**: 声明 TableGen def 记录 `SCALAR_SQSHRUN_N`。
- **L1380 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheckShiftRightNarrow, 0>]>;`.
  **L1380 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheckShiftRightNarrow, 0>]>;`。
- **L1381 EN**: Comment explains nearby logic, constraints, or intent: `Signed Saturating Rounded Shift Right Unsigned Narrow (Immediate)`.
  **L1381 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Signed Saturating Rounded Shift Right Unsigned Narrow (Immediate)`。
- **L1382 EN**: Declares TableGen def record `SCALAR_SQRSHRUN_N`.
  **L1382 CN**: 声明 TableGen def 记录 `SCALAR_SQRSHRUN_N`。
- **L1383 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheckShiftRightNarrow, 0>]>;`.
  **L1383 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheckShiftRightNarrow, 0>]>;`。
- **L1384 EN**: Closes the current lexical scope or compound statement.
  **L1384 CN**: 结束当前词法作用域或复合语句块。
- **L1385 EN**: Blank line separating nearby declarations or logic blocks.
  **L1385 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1386 EN**: Separator comment used for visual grouping.
  **L1386 CN**: 用于视觉分组的分隔注释。
- **L1387 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Signed/Unsigned Fixed-point Convert To Floating-Point (Immediate)`.
  **L1387 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Signed/Unsigned Fixed-point Convert To Floating-Point (Immediate)`。
- **L1388 EN**: Declares TableGen def record `SCALAR_SCVTF_N_F32`.
  **L1388 CN**: 声明 TableGen def 记录 `SCALAR_SCVTF_N_F32`。
- **L1389 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheck1_32>]>;`.
  **L1389 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheck1_32>]>;`。
- **L1390 EN**: Declares TableGen def record `SCALAR_SCVTF_N_F64`.
  **L1390 CN**: 声明 TableGen def 记录 `SCALAR_SCVTF_N_F64`。
- **L1391 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheck1_64>]>;`.
  **L1391 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheck1_64>]>;`。
- **L1392 EN**: Blank line separating nearby declarations or logic blocks.
  **L1392 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1393 EN**: Separator comment used for visual grouping.
  **L1393 CN**: 用于视觉分组的分隔注释。
- **L1394 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Floating-point Convert To Signed/Unsigned Fixed-point (Immediate)`.
  **L1394 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Floating-point Convert To Signed/Unsigned Fixed-point (Immediate)`。
- **L1395 EN**: Declares TableGen def record `SCALAR_FCVTZS_N_S32`.
  **L1395 CN**: 声明 TableGen def 记录 `SCALAR_FCVTZS_N_S32`。
- **L1396 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheck1_32>]>;`.
  **L1396 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheck1_32>]>;`。
- **L1397 EN**: Declares TableGen def record `SCALAR_FCVTZU_N_U32`.
  **L1397 CN**: 声明 TableGen def 记录 `SCALAR_FCVTZU_N_U32`。
- **L1398 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheck1_32>]>;`.
  **L1398 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheck1_32>]>;`。
- **L1399 EN**: Declares TableGen def record `SCALAR_FCVTZS_N_S64`.
  **L1399 CN**: 声明 TableGen def 记录 `SCALAR_FCVTZS_N_S64`。
- **L1400 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheck1_64>]>;`.
  **L1400 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheck1_64>]>;`。

### Lines 1401-1428

````tablegen
def SCALAR_FCVTZU_N_U64 : SInst<"vcvt_n_u64", "(1U)1I", "Sd",
                                [ImmCheck<1, ImmCheck1_64>]>;
}

////////////////////////////////////////////////////////////////////////////////
// Scalar Floating-point Round to Integral
let ArchGuard = "__ARM_ARCH >= 8 && defined(__ARM_FEATURE_DIRECTED_ROUNDING)" in {
def SCALAR_FRINTN_S32 : SInst<"vrndn", "11", "Sf">;
}

////////////////////////////////////////////////////////////////////////////////
// Scalar Reduce Pairwise Addition (Scalar and Floating Point)
def SCALAR_ADDP  : SInst<"vpadd", "1.", "SfSHlSHdSHUl">;

////////////////////////////////////////////////////////////////////////////////
// Scalar Reduce Floating Point Pairwise Max/Min
def SCALAR_FMAXP : SInst<"vpmax", "1.", "SfSQd">;

def SCALAR_FMINP : SInst<"vpmin", "1.", "SfSQd">;

////////////////////////////////////////////////////////////////////////////////
// Scalar Reduce Floating Point Pairwise maxNum/minNum
def SCALAR_FMAXNMP : SInst<"vpmaxnm", "1.", "SfSQd">;
def SCALAR_FMINNMP : SInst<"vpminnm", "1.", "SfSQd">;

////////////////////////////////////////////////////////////////////////////////
// Scalar Integer Saturating Doubling Multiply Half High
def SCALAR_SQDMULH : SInst<"vqdmulh", "111", "SsSi">;
````
- **L1401 EN**: Declares TableGen def record `SCALAR_FCVTZU_N_U64`.
  **L1401 CN**: 声明 TableGen def 记录 `SCALAR_FCVTZU_N_U64`。
- **L1402 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheck1_64>]>;`.
  **L1402 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheck1_64>]>;`。
- **L1403 EN**: Closes the current lexical scope or compound statement.
  **L1403 CN**: 结束当前词法作用域或复合语句块。
- **L1404 EN**: Blank line separating nearby declarations or logic blocks.
  **L1404 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1405 EN**: Separator comment used for visual grouping.
  **L1405 CN**: 用于视觉分组的分隔注释。
- **L1406 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Floating-point Round to Integral`.
  **L1406 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Floating-point Round to Integral`。
- **L1407 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "__ARM_ARCH >= 8 && defined(__ARM_FEATURE_DIRECTED_ROUNDING)" in {`.
  **L1407 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "__ARM_ARCH >= 8 && defined(__ARM_FEATURE_DIRECTED_ROUNDING)" in {`。
- **L1408 EN**: Declares TableGen def record `SCALAR_FRINTN_S32`.
  **L1408 CN**: 声明 TableGen def 记录 `SCALAR_FRINTN_S32`。
- **L1409 EN**: Closes the current lexical scope or compound statement.
  **L1409 CN**: 结束当前词法作用域或复合语句块。
- **L1410 EN**: Blank line separating nearby declarations or logic blocks.
  **L1410 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1411 EN**: Separator comment used for visual grouping.
  **L1411 CN**: 用于视觉分组的分隔注释。
- **L1412 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Reduce Pairwise Addition (Scalar and Floating Point)`.
  **L1412 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Reduce Pairwise Addition (Scalar and Floating Point)`。
- **L1413 EN**: Declares TableGen def record `SCALAR_ADDP`.
  **L1413 CN**: 声明 TableGen def 记录 `SCALAR_ADDP`。
- **L1414 EN**: Blank line separating nearby declarations or logic blocks.
  **L1414 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1415 EN**: Separator comment used for visual grouping.
  **L1415 CN**: 用于视觉分组的分隔注释。
- **L1416 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Reduce Floating Point Pairwise Max/Min`.
  **L1416 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Reduce Floating Point Pairwise Max/Min`。
- **L1417 EN**: Declares TableGen def record `SCALAR_FMAXP`.
  **L1417 CN**: 声明 TableGen def 记录 `SCALAR_FMAXP`。
- **L1418 EN**: Blank line separating nearby declarations or logic blocks.
  **L1418 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1419 EN**: Declares TableGen def record `SCALAR_FMINP`.
  **L1419 CN**: 声明 TableGen def 记录 `SCALAR_FMINP`。
- **L1420 EN**: Blank line separating nearby declarations or logic blocks.
  **L1420 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1421 EN**: Separator comment used for visual grouping.
  **L1421 CN**: 用于视觉分组的分隔注释。
- **L1422 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Reduce Floating Point Pairwise maxNum/minNum`.
  **L1422 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Reduce Floating Point Pairwise maxNum/minNum`。
- **L1423 EN**: Declares TableGen def record `SCALAR_FMAXNMP`.
  **L1423 CN**: 声明 TableGen def 记录 `SCALAR_FMAXNMP`。
- **L1424 EN**: Declares TableGen def record `SCALAR_FMINNMP`.
  **L1424 CN**: 声明 TableGen def 记录 `SCALAR_FMINNMP`。
- **L1425 EN**: Blank line separating nearby declarations or logic blocks.
  **L1425 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1426 EN**: Separator comment used for visual grouping.
  **L1426 CN**: 用于视觉分组的分隔注释。
- **L1427 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Integer Saturating Doubling Multiply Half High`.
  **L1427 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Integer Saturating Doubling Multiply Half High`。
- **L1428 EN**: Declares TableGen def record `SCALAR_SQDMULH`.
  **L1428 CN**: 声明 TableGen def 记录 `SCALAR_SQDMULH`。

### Lines 1429-1456

````tablegen

////////////////////////////////////////////////////////////////////////////////
// Scalar Integer Saturating Rounding Doubling Multiply Half High
def SCALAR_SQRDMULH : SInst<"vqrdmulh", "111", "SsSi">;

let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "v8.1a,neon" in {
////////////////////////////////////////////////////////////////////////////////
// Signed Saturating Rounding Doubling Multiply Accumulate Returning High Half
def SCALAR_SQRDMLAH : SInst<"vqrdmlah", "1111", "SsSi">;

////////////////////////////////////////////////////////////////////////////////
// Signed Saturating Rounding Doubling Multiply Subtract Returning High Half
def SCALAR_SQRDMLSH : SInst<"vqrdmlsh", "1111", "SsSi">;
} // ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "v8.1a"

////////////////////////////////////////////////////////////////////////////////
// Scalar Floating-point Multiply Extended
def SCALAR_FMULX : IInst<"vmulx", "111", "SfSd">;

////////////////////////////////////////////////////////////////////////////////
// Scalar Floating-point Reciprocal Step
def SCALAR_FRECPS : IInst<"vrecps", "111", "SfSd">;

////////////////////////////////////////////////////////////////////////////////
// Scalar Floating-point Reciprocal Square Root Step
def SCALAR_FRSQRTS : IInst<"vrsqrts", "111", "SfSd">;

////////////////////////////////////////////////////////////////////////////////
````
- **L1429 EN**: Blank line separating nearby declarations or logic blocks.
  **L1429 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1430 EN**: Separator comment used for visual grouping.
  **L1430 CN**: 用于视觉分组的分隔注释。
- **L1431 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Integer Saturating Rounding Doubling Multiply Half High`.
  **L1431 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Integer Saturating Rounding Doubling Multiply Half High`。
- **L1432 EN**: Declares TableGen def record `SCALAR_SQRDMULH`.
  **L1432 CN**: 声明 TableGen def 记录 `SCALAR_SQRDMULH`。
- **L1433 EN**: Blank line separating nearby declarations or logic blocks.
  **L1433 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1434 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "v8.1a,neon" in {`.
  **L1434 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "v8.1a,neon" in {`。
- **L1435 EN**: Separator comment used for visual grouping.
  **L1435 CN**: 用于视觉分组的分隔注释。
- **L1436 EN**: Comment explains nearby logic, constraints, or intent: `Signed Saturating Rounding Doubling Multiply Accumulate Returning High Half`.
  **L1436 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Signed Saturating Rounding Doubling Multiply Accumulate Returning High Half`。
- **L1437 EN**: Declares TableGen def record `SCALAR_SQRDMLAH`.
  **L1437 CN**: 声明 TableGen def 记录 `SCALAR_SQRDMLAH`。
- **L1438 EN**: Blank line separating nearby declarations or logic blocks.
  **L1438 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1439 EN**: Separator comment used for visual grouping.
  **L1439 CN**: 用于视觉分组的分隔注释。
- **L1440 EN**: Comment explains nearby logic, constraints, or intent: `Signed Saturating Rounding Doubling Multiply Subtract Returning High Half`.
  **L1440 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Signed Saturating Rounding Doubling Multiply Subtract Returning High Half`。
- **L1441 EN**: Declares TableGen def record `SCALAR_SQRDMLSH`.
  **L1441 CN**: 声明 TableGen def 记录 `SCALAR_SQRDMLSH`。
- **L1442 EN**: Continues logic associated with callable symbol `defined`.
  **L1442 CN**: 继续与可调用符号 `defined` 相关的逻辑。
- **L1443 EN**: Blank line separating nearby declarations or logic blocks.
  **L1443 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1444 EN**: Separator comment used for visual grouping.
  **L1444 CN**: 用于视觉分组的分隔注释。
- **L1445 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Floating-point Multiply Extended`.
  **L1445 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Floating-point Multiply Extended`。
- **L1446 EN**: Declares TableGen def record `SCALAR_FMULX`.
  **L1446 CN**: 声明 TableGen def 记录 `SCALAR_FMULX`。
- **L1447 EN**: Blank line separating nearby declarations or logic blocks.
  **L1447 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1448 EN**: Separator comment used for visual grouping.
  **L1448 CN**: 用于视觉分组的分隔注释。
- **L1449 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Floating-point Reciprocal Step`.
  **L1449 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Floating-point Reciprocal Step`。
- **L1450 EN**: Declares TableGen def record `SCALAR_FRECPS`.
  **L1450 CN**: 声明 TableGen def 记录 `SCALAR_FRECPS`。
- **L1451 EN**: Blank line separating nearby declarations or logic blocks.
  **L1451 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1452 EN**: Separator comment used for visual grouping.
  **L1452 CN**: 用于视觉分组的分隔注释。
- **L1453 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Floating-point Reciprocal Square Root Step`.
  **L1453 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Floating-point Reciprocal Square Root Step`。
- **L1454 EN**: Declares TableGen def record `SCALAR_FRSQRTS`.
  **L1454 CN**: 声明 TableGen def 记录 `SCALAR_FRSQRTS`。
- **L1455 EN**: Blank line separating nearby declarations or logic blocks.
  **L1455 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1456 EN**: Separator comment used for visual grouping.
  **L1456 CN**: 用于视觉分组的分隔注释。

### Lines 1457-1484

````tablegen
// Scalar Signed Integer Convert To Floating-point
def SCALAR_SCVTFS : SInst<"vcvt_f32", "(1F)(1!)", "Si">;
def SCALAR_SCVTFD : SInst<"vcvt_f64", "(1F)(1!)", "Sl">;

////////////////////////////////////////////////////////////////////////////////
// Scalar Unsigned Integer Convert To Floating-point
def SCALAR_UCVTFS : SInst<"vcvt_f32", "(1F)(1!)", "SUi">;
def SCALAR_UCVTFD : SInst<"vcvt_f64", "(1F)(1!)", "SUl">;

////////////////////////////////////////////////////////////////////////////////
// Scalar Floating-point Converts
def SCALAR_FCVTXN  : IInst<"vcvtx_f32", "(1F<)(1!)", "Sd">;

def SCALAR_FCVTN_F32toSS  : SInst<"vcvtn_s32", "(1S)1", "Sf">;
def SCALAR_FCVTN_F32toUS  : SInst<"vcvtn_u32", "(1U)1", "Sf">;
def SCALAR_FCVTN_F64toSS  : SInst<"vcvtn_s32", "(1S<)1", "Sd">;
def SCALAR_FCVTN_F64toUS  : SInst<"vcvtn_u32", "(1U<)1", "Sd">;
def SCALAR_FCVTN_F32toSD  : SInst<"vcvtn_s64", "(1S>)1", "Sf">;
def SCALAR_FCVTN_F32toUD  : SInst<"vcvtn_u64", "(1U>)1", "Sf">;
def SCALAR_FCVTN_F64toSD  : SInst<"vcvtn_s64", "(1S)1", "Sd">;
def SCALAR_FCVTN_F64toUD  : SInst<"vcvtn_u64", "(1U)1", "Sd">;

def SCALAR_FCVTM_F32toSS  : SInst<"vcvtm_s32", "(1S)1", "Sf">;
def SCALAR_FCVTM_F32toUS  : SInst<"vcvtm_u32", "(1U)1", "Sf">;
def SCALAR_FCVTM_F64toSS  : SInst<"vcvtm_s32", "(1S<)1", "Sd">;
def SCALAR_FCVTM_F64toUS  : SInst<"vcvtm_u32", "(1U<)1", "Sd">;
def SCALAR_FCVTM_F32toSD  : SInst<"vcvtm_s64", "(1S>)1", "Sf">;
def SCALAR_FCVTM_F32toUD  : SInst<"vcvtm_u64", "(1U>)1", "Sf">;
````
- **L1457 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Signed Integer Convert To Floating-point`.
  **L1457 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Signed Integer Convert To Floating-point`。
- **L1458 EN**: Declares TableGen def record `SCALAR_SCVTFS`.
  **L1458 CN**: 声明 TableGen def 记录 `SCALAR_SCVTFS`。
- **L1459 EN**: Declares TableGen def record `SCALAR_SCVTFD`.
  **L1459 CN**: 声明 TableGen def 记录 `SCALAR_SCVTFD`。
- **L1460 EN**: Blank line separating nearby declarations or logic blocks.
  **L1460 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1461 EN**: Separator comment used for visual grouping.
  **L1461 CN**: 用于视觉分组的分隔注释。
- **L1462 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Unsigned Integer Convert To Floating-point`.
  **L1462 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Unsigned Integer Convert To Floating-point`。
- **L1463 EN**: Declares TableGen def record `SCALAR_UCVTFS`.
  **L1463 CN**: 声明 TableGen def 记录 `SCALAR_UCVTFS`。
- **L1464 EN**: Declares TableGen def record `SCALAR_UCVTFD`.
  **L1464 CN**: 声明 TableGen def 记录 `SCALAR_UCVTFD`。
- **L1465 EN**: Blank line separating nearby declarations or logic blocks.
  **L1465 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1466 EN**: Separator comment used for visual grouping.
  **L1466 CN**: 用于视觉分组的分隔注释。
- **L1467 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Floating-point Converts`.
  **L1467 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Floating-point Converts`。
- **L1468 EN**: Declares TableGen def record `SCALAR_FCVTXN`.
  **L1468 CN**: 声明 TableGen def 记录 `SCALAR_FCVTXN`。
- **L1469 EN**: Blank line separating nearby declarations or logic blocks.
  **L1469 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1470 EN**: Declares TableGen def record `SCALAR_FCVTN_F32toSS`.
  **L1470 CN**: 声明 TableGen def 记录 `SCALAR_FCVTN_F32toSS`。
- **L1471 EN**: Declares TableGen def record `SCALAR_FCVTN_F32toUS`.
  **L1471 CN**: 声明 TableGen def 记录 `SCALAR_FCVTN_F32toUS`。
- **L1472 EN**: Declares TableGen def record `SCALAR_FCVTN_F64toSS`.
  **L1472 CN**: 声明 TableGen def 记录 `SCALAR_FCVTN_F64toSS`。
- **L1473 EN**: Declares TableGen def record `SCALAR_FCVTN_F64toUS`.
  **L1473 CN**: 声明 TableGen def 记录 `SCALAR_FCVTN_F64toUS`。
- **L1474 EN**: Declares TableGen def record `SCALAR_FCVTN_F32toSD`.
  **L1474 CN**: 声明 TableGen def 记录 `SCALAR_FCVTN_F32toSD`。
- **L1475 EN**: Declares TableGen def record `SCALAR_FCVTN_F32toUD`.
  **L1475 CN**: 声明 TableGen def 记录 `SCALAR_FCVTN_F32toUD`。
- **L1476 EN**: Declares TableGen def record `SCALAR_FCVTN_F64toSD`.
  **L1476 CN**: 声明 TableGen def 记录 `SCALAR_FCVTN_F64toSD`。
- **L1477 EN**: Declares TableGen def record `SCALAR_FCVTN_F64toUD`.
  **L1477 CN**: 声明 TableGen def 记录 `SCALAR_FCVTN_F64toUD`。
- **L1478 EN**: Blank line separating nearby declarations or logic blocks.
  **L1478 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1479 EN**: Declares TableGen def record `SCALAR_FCVTM_F32toSS`.
  **L1479 CN**: 声明 TableGen def 记录 `SCALAR_FCVTM_F32toSS`。
- **L1480 EN**: Declares TableGen def record `SCALAR_FCVTM_F32toUS`.
  **L1480 CN**: 声明 TableGen def 记录 `SCALAR_FCVTM_F32toUS`。
- **L1481 EN**: Declares TableGen def record `SCALAR_FCVTM_F64toSS`.
  **L1481 CN**: 声明 TableGen def 记录 `SCALAR_FCVTM_F64toSS`。
- **L1482 EN**: Declares TableGen def record `SCALAR_FCVTM_F64toUS`.
  **L1482 CN**: 声明 TableGen def 记录 `SCALAR_FCVTM_F64toUS`。
- **L1483 EN**: Declares TableGen def record `SCALAR_FCVTM_F32toSD`.
  **L1483 CN**: 声明 TableGen def 记录 `SCALAR_FCVTM_F32toSD`。
- **L1484 EN**: Declares TableGen def record `SCALAR_FCVTM_F32toUD`.
  **L1484 CN**: 声明 TableGen def 记录 `SCALAR_FCVTM_F32toUD`。

### Lines 1485-1512

````tablegen
def SCALAR_FCVTM_F64toSD  : SInst<"vcvtm_s64", "(1S)1", "Sd">;
def SCALAR_FCVTM_F64toUD  : SInst<"vcvtm_u64", "(1U)1", "Sd">;

def SCALAR_FCVTA_F32toSS  : SInst<"vcvta_s32", "(1S)1", "Sf">;
def SCALAR_FCVTA_F32toUS  : SInst<"vcvta_u32", "(1U)1", "Sf">;
def SCALAR_FCVTA_F64toSS  : SInst<"vcvta_s32", "(1S<)1", "Sd">;
def SCALAR_FCVTA_F64toUS  : SInst<"vcvta_u32", "(1U<)1", "Sd">;
def SCALAR_FCVTA_F32toSD  : SInst<"vcvta_s64", "(1S>)1", "Sf">;
def SCALAR_FCVTA_F32toUD  : SInst<"vcvta_u64", "(1U>)1", "Sf">;
def SCALAR_FCVTA_F64toSD  : SInst<"vcvta_s64", "(1S)1", "Sd">;
def SCALAR_FCVTA_F64toUD  : SInst<"vcvta_u64", "(1U)1", "Sd">;

def SCALAR_FCVTP_F32toSS  : SInst<"vcvtp_s32", "(1S)1", "Sf">;
def SCALAR_FCVTP_F32toUS  : SInst<"vcvtp_u32", "(1U)1", "Sf">;
def SCALAR_FCVTP_F64toSS  : SInst<"vcvtp_s32", "(1S<)1", "Sd">;
def SCALAR_FCVTP_F64toUS  : SInst<"vcvtp_u32", "(1U<)1", "Sd">;
def SCALAR_FCVTP_F32toSD  : SInst<"vcvtp_s64", "(1S>)1", "Sf">;
def SCALAR_FCVTP_F32toUD  : SInst<"vcvtp_u64", "(1U>)1", "Sf">;
def SCALAR_FCVTP_F64toSD  : SInst<"vcvtp_s64", "(1S)1", "Sd">;
def SCALAR_FCVTP_F64toUD  : SInst<"vcvtp_u64", "(1U)1", "Sd">;

def SCALAR_FCVTZ_F32toSS  : SInst<"vcvt_s32", "(1S)1", "Sf">;
def SCALAR_FCVTZ_F32toUS  : SInst<"vcvt_u32", "(1U)1", "Sf">;
def SCALAR_FCVTZ_F64toSS  : SInst<"vcvt_s32", "(1S<)1", "Sd">;
def SCALAR_FCVTZ_F64toUS  : SInst<"vcvt_u32", "(1U<)1", "Sd">;
def SCALAR_FCVTZ_F32toSD  : SInst<"vcvt_s64", "(1S>)1", "Sf">;
def SCALAR_FCVTZ_F32toUD  : SInst<"vcvt_u64", "(1U>)1", "Sf">;
def SCALAR_FCVTZ_F64toSD  : SInst<"vcvt_s64", "(1S)1", "Sd">;
````
- **L1485 EN**: Declares TableGen def record `SCALAR_FCVTM_F64toSD`.
  **L1485 CN**: 声明 TableGen def 记录 `SCALAR_FCVTM_F64toSD`。
- **L1486 EN**: Declares TableGen def record `SCALAR_FCVTM_F64toUD`.
  **L1486 CN**: 声明 TableGen def 记录 `SCALAR_FCVTM_F64toUD`。
- **L1487 EN**: Blank line separating nearby declarations or logic blocks.
  **L1487 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1488 EN**: Declares TableGen def record `SCALAR_FCVTA_F32toSS`.
  **L1488 CN**: 声明 TableGen def 记录 `SCALAR_FCVTA_F32toSS`。
- **L1489 EN**: Declares TableGen def record `SCALAR_FCVTA_F32toUS`.
  **L1489 CN**: 声明 TableGen def 记录 `SCALAR_FCVTA_F32toUS`。
- **L1490 EN**: Declares TableGen def record `SCALAR_FCVTA_F64toSS`.
  **L1490 CN**: 声明 TableGen def 记录 `SCALAR_FCVTA_F64toSS`。
- **L1491 EN**: Declares TableGen def record `SCALAR_FCVTA_F64toUS`.
  **L1491 CN**: 声明 TableGen def 记录 `SCALAR_FCVTA_F64toUS`。
- **L1492 EN**: Declares TableGen def record `SCALAR_FCVTA_F32toSD`.
  **L1492 CN**: 声明 TableGen def 记录 `SCALAR_FCVTA_F32toSD`。
- **L1493 EN**: Declares TableGen def record `SCALAR_FCVTA_F32toUD`.
  **L1493 CN**: 声明 TableGen def 记录 `SCALAR_FCVTA_F32toUD`。
- **L1494 EN**: Declares TableGen def record `SCALAR_FCVTA_F64toSD`.
  **L1494 CN**: 声明 TableGen def 记录 `SCALAR_FCVTA_F64toSD`。
- **L1495 EN**: Declares TableGen def record `SCALAR_FCVTA_F64toUD`.
  **L1495 CN**: 声明 TableGen def 记录 `SCALAR_FCVTA_F64toUD`。
- **L1496 EN**: Blank line separating nearby declarations or logic blocks.
  **L1496 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1497 EN**: Declares TableGen def record `SCALAR_FCVTP_F32toSS`.
  **L1497 CN**: 声明 TableGen def 记录 `SCALAR_FCVTP_F32toSS`。
- **L1498 EN**: Declares TableGen def record `SCALAR_FCVTP_F32toUS`.
  **L1498 CN**: 声明 TableGen def 记录 `SCALAR_FCVTP_F32toUS`。
- **L1499 EN**: Declares TableGen def record `SCALAR_FCVTP_F64toSS`.
  **L1499 CN**: 声明 TableGen def 记录 `SCALAR_FCVTP_F64toSS`。
- **L1500 EN**: Declares TableGen def record `SCALAR_FCVTP_F64toUS`.
  **L1500 CN**: 声明 TableGen def 记录 `SCALAR_FCVTP_F64toUS`。
- **L1501 EN**: Declares TableGen def record `SCALAR_FCVTP_F32toSD`.
  **L1501 CN**: 声明 TableGen def 记录 `SCALAR_FCVTP_F32toSD`。
- **L1502 EN**: Declares TableGen def record `SCALAR_FCVTP_F32toUD`.
  **L1502 CN**: 声明 TableGen def 记录 `SCALAR_FCVTP_F32toUD`。
- **L1503 EN**: Declares TableGen def record `SCALAR_FCVTP_F64toSD`.
  **L1503 CN**: 声明 TableGen def 记录 `SCALAR_FCVTP_F64toSD`。
- **L1504 EN**: Declares TableGen def record `SCALAR_FCVTP_F64toUD`.
  **L1504 CN**: 声明 TableGen def 记录 `SCALAR_FCVTP_F64toUD`。
- **L1505 EN**: Blank line separating nearby declarations or logic blocks.
  **L1505 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1506 EN**: Declares TableGen def record `SCALAR_FCVTZ_F32toSS`.
  **L1506 CN**: 声明 TableGen def 记录 `SCALAR_FCVTZ_F32toSS`。
- **L1507 EN**: Declares TableGen def record `SCALAR_FCVTZ_F32toUS`.
  **L1507 CN**: 声明 TableGen def 记录 `SCALAR_FCVTZ_F32toUS`。
- **L1508 EN**: Declares TableGen def record `SCALAR_FCVTZ_F64toSS`.
  **L1508 CN**: 声明 TableGen def 记录 `SCALAR_FCVTZ_F64toSS`。
- **L1509 EN**: Declares TableGen def record `SCALAR_FCVTZ_F64toUS`.
  **L1509 CN**: 声明 TableGen def 记录 `SCALAR_FCVTZ_F64toUS`。
- **L1510 EN**: Declares TableGen def record `SCALAR_FCVTZ_F32toSD`.
  **L1510 CN**: 声明 TableGen def 记录 `SCALAR_FCVTZ_F32toSD`。
- **L1511 EN**: Declares TableGen def record `SCALAR_FCVTZ_F32toUD`.
  **L1511 CN**: 声明 TableGen def 记录 `SCALAR_FCVTZ_F32toUD`。
- **L1512 EN**: Declares TableGen def record `SCALAR_FCVTZ_F64toSD`.
  **L1512 CN**: 声明 TableGen def 记录 `SCALAR_FCVTZ_F64toSD`。

### Lines 1513-1540

````tablegen
def SCALAR_FCVTZ_F64toUD  : SInst<"vcvt_u64", "(1U)1", "Sd">;

////////////////////////////////////////////////////////////////////////////////
// Scalar Floating-point Reciprocal Estimate
def SCALAR_FRECPE : IInst<"vrecpe", "11", "SfSd">;

////////////////////////////////////////////////////////////////////////////////
// Scalar Floating-point Reciprocal Exponent
def SCALAR_FRECPX : IInst<"vrecpx", "11", "SfSd">;

////////////////////////////////////////////////////////////////////////////////
// Scalar Floating-point Reciprocal Square Root Estimate
def SCALAR_FRSQRTE : IInst<"vrsqrte", "11", "SfSd">;

////////////////////////////////////////////////////////////////////////////////
// Scalar Integer Comparison
def SCALAR_CMEQ : SInst<"vceq", "(U1)11", "SlSUl">;
def SCALAR_CMEQZ : SInst<"vceqz", "(U1)1", "SlSUl">;
def SCALAR_CMGE : SInst<"vcge", "(U1)11", "Sl">;
def SCALAR_CMGEZ : SInst<"vcgez", "(U1)1", "Sl">;
def SCALAR_CMHS : SInst<"vcge", "(U1)11", "SUl">;
def SCALAR_CMLE : SInst<"vcle", "(U1)11", "SlSUl">;
def SCALAR_CMLEZ : SInst<"vclez", "(U1)1", "Sl">;
def SCALAR_CMLT : SInst<"vclt", "(U1)11", "SlSUl">;
def SCALAR_CMLTZ : SInst<"vcltz", "(U1)1", "Sl">;
def SCALAR_CMGT : SInst<"vcgt", "(U1)11", "Sl">;
def SCALAR_CMGTZ : SInst<"vcgtz", "(U1)1", "Sl">;
def SCALAR_CMHI : SInst<"vcgt", "(U1)11", "SUl">;
````
- **L1513 EN**: Declares TableGen def record `SCALAR_FCVTZ_F64toUD`.
  **L1513 CN**: 声明 TableGen def 记录 `SCALAR_FCVTZ_F64toUD`。
- **L1514 EN**: Blank line separating nearby declarations or logic blocks.
  **L1514 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1515 EN**: Separator comment used for visual grouping.
  **L1515 CN**: 用于视觉分组的分隔注释。
- **L1516 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Floating-point Reciprocal Estimate`.
  **L1516 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Floating-point Reciprocal Estimate`。
- **L1517 EN**: Declares TableGen def record `SCALAR_FRECPE`.
  **L1517 CN**: 声明 TableGen def 记录 `SCALAR_FRECPE`。
- **L1518 EN**: Blank line separating nearby declarations or logic blocks.
  **L1518 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1519 EN**: Separator comment used for visual grouping.
  **L1519 CN**: 用于视觉分组的分隔注释。
- **L1520 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Floating-point Reciprocal Exponent`.
  **L1520 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Floating-point Reciprocal Exponent`。
- **L1521 EN**: Declares TableGen def record `SCALAR_FRECPX`.
  **L1521 CN**: 声明 TableGen def 记录 `SCALAR_FRECPX`。
- **L1522 EN**: Blank line separating nearby declarations or logic blocks.
  **L1522 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1523 EN**: Separator comment used for visual grouping.
  **L1523 CN**: 用于视觉分组的分隔注释。
- **L1524 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Floating-point Reciprocal Square Root Estimate`.
  **L1524 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Floating-point Reciprocal Square Root Estimate`。
- **L1525 EN**: Declares TableGen def record `SCALAR_FRSQRTE`.
  **L1525 CN**: 声明 TableGen def 记录 `SCALAR_FRSQRTE`。
- **L1526 EN**: Blank line separating nearby declarations or logic blocks.
  **L1526 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1527 EN**: Separator comment used for visual grouping.
  **L1527 CN**: 用于视觉分组的分隔注释。
- **L1528 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Integer Comparison`.
  **L1528 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Integer Comparison`。
- **L1529 EN**: Declares TableGen def record `SCALAR_CMEQ`.
  **L1529 CN**: 声明 TableGen def 记录 `SCALAR_CMEQ`。
- **L1530 EN**: Declares TableGen def record `SCALAR_CMEQZ`.
  **L1530 CN**: 声明 TableGen def 记录 `SCALAR_CMEQZ`。
- **L1531 EN**: Declares TableGen def record `SCALAR_CMGE`.
  **L1531 CN**: 声明 TableGen def 记录 `SCALAR_CMGE`。
- **L1532 EN**: Declares TableGen def record `SCALAR_CMGEZ`.
  **L1532 CN**: 声明 TableGen def 记录 `SCALAR_CMGEZ`。
- **L1533 EN**: Declares TableGen def record `SCALAR_CMHS`.
  **L1533 CN**: 声明 TableGen def 记录 `SCALAR_CMHS`。
- **L1534 EN**: Declares TableGen def record `SCALAR_CMLE`.
  **L1534 CN**: 声明 TableGen def 记录 `SCALAR_CMLE`。
- **L1535 EN**: Declares TableGen def record `SCALAR_CMLEZ`.
  **L1535 CN**: 声明 TableGen def 记录 `SCALAR_CMLEZ`。
- **L1536 EN**: Declares TableGen def record `SCALAR_CMLT`.
  **L1536 CN**: 声明 TableGen def 记录 `SCALAR_CMLT`。
- **L1537 EN**: Declares TableGen def record `SCALAR_CMLTZ`.
  **L1537 CN**: 声明 TableGen def 记录 `SCALAR_CMLTZ`。
- **L1538 EN**: Declares TableGen def record `SCALAR_CMGT`.
  **L1538 CN**: 声明 TableGen def 记录 `SCALAR_CMGT`。
- **L1539 EN**: Declares TableGen def record `SCALAR_CMGTZ`.
  **L1539 CN**: 声明 TableGen def 记录 `SCALAR_CMGTZ`。
- **L1540 EN**: Declares TableGen def record `SCALAR_CMHI`.
  **L1540 CN**: 声明 TableGen def 记录 `SCALAR_CMHI`。

### Lines 1541-1568

````tablegen
def SCALAR_CMTST : SInst<"vtst", "(U1)11", "SlSUl">;

////////////////////////////////////////////////////////////////////////////////
// Scalar Floating-point Comparison
def SCALAR_FCMEQ : IInst<"vceq", "(1U)11", "SfSd">;
def SCALAR_FCMEQZ : IInst<"vceqz", "(1U)1", "SfSd">;
def SCALAR_FCMGE : IInst<"vcge", "(1U)11", "SfSd">;
def SCALAR_FCMGEZ : IInst<"vcgez", "(1U)1", "SfSd">;
def SCALAR_FCMGT : IInst<"vcgt", "(1U)11", "SfSd">;
def SCALAR_FCMGTZ : IInst<"vcgtz", "(1U)1", "SfSd">;
def SCALAR_FCMLE : IInst<"vcle", "(1U)11", "SfSd">;
def SCALAR_FCMLEZ : IInst<"vclez", "(1U)1", "SfSd">;
def SCALAR_FCMLT : IInst<"vclt", "(1U)11", "SfSd">;
def SCALAR_FCMLTZ : IInst<"vcltz", "(1U)1", "SfSd">;

////////////////////////////////////////////////////////////////////////////////
// Scalar Floating-point Absolute Compare Mask Greater Than Or Equal
def SCALAR_FACGE : IInst<"vcage", "(1U)11", "SfSd">;
def SCALAR_FACLE : IInst<"vcale", "(1U)11", "SfSd">;

////////////////////////////////////////////////////////////////////////////////
// Scalar Floating-point Absolute Compare Mask Greater Than
def SCALAR_FACGT : IInst<"vcagt", "(1U)11", "SfSd">;
def SCALAR_FACLT : IInst<"vcalt", "(1U)11", "SfSd">;

////////////////////////////////////////////////////////////////////////////////
// Scalar Absolute Value
def SCALAR_ABS : SInst<"vabs", "11", "Sl">;
````
- **L1541 EN**: Declares TableGen def record `SCALAR_CMTST`.
  **L1541 CN**: 声明 TableGen def 记录 `SCALAR_CMTST`。
- **L1542 EN**: Blank line separating nearby declarations or logic blocks.
  **L1542 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1543 EN**: Separator comment used for visual grouping.
  **L1543 CN**: 用于视觉分组的分隔注释。
- **L1544 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Floating-point Comparison`.
  **L1544 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Floating-point Comparison`。
- **L1545 EN**: Declares TableGen def record `SCALAR_FCMEQ`.
  **L1545 CN**: 声明 TableGen def 记录 `SCALAR_FCMEQ`。
- **L1546 EN**: Declares TableGen def record `SCALAR_FCMEQZ`.
  **L1546 CN**: 声明 TableGen def 记录 `SCALAR_FCMEQZ`。
- **L1547 EN**: Declares TableGen def record `SCALAR_FCMGE`.
  **L1547 CN**: 声明 TableGen def 记录 `SCALAR_FCMGE`。
- **L1548 EN**: Declares TableGen def record `SCALAR_FCMGEZ`.
  **L1548 CN**: 声明 TableGen def 记录 `SCALAR_FCMGEZ`。
- **L1549 EN**: Declares TableGen def record `SCALAR_FCMGT`.
  **L1549 CN**: 声明 TableGen def 记录 `SCALAR_FCMGT`。
- **L1550 EN**: Declares TableGen def record `SCALAR_FCMGTZ`.
  **L1550 CN**: 声明 TableGen def 记录 `SCALAR_FCMGTZ`。
- **L1551 EN**: Declares TableGen def record `SCALAR_FCMLE`.
  **L1551 CN**: 声明 TableGen def 记录 `SCALAR_FCMLE`。
- **L1552 EN**: Declares TableGen def record `SCALAR_FCMLEZ`.
  **L1552 CN**: 声明 TableGen def 记录 `SCALAR_FCMLEZ`。
- **L1553 EN**: Declares TableGen def record `SCALAR_FCMLT`.
  **L1553 CN**: 声明 TableGen def 记录 `SCALAR_FCMLT`。
- **L1554 EN**: Declares TableGen def record `SCALAR_FCMLTZ`.
  **L1554 CN**: 声明 TableGen def 记录 `SCALAR_FCMLTZ`。
- **L1555 EN**: Blank line separating nearby declarations or logic blocks.
  **L1555 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1556 EN**: Separator comment used for visual grouping.
  **L1556 CN**: 用于视觉分组的分隔注释。
- **L1557 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Floating-point Absolute Compare Mask Greater Than Or Equal`.
  **L1557 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Floating-point Absolute Compare Mask Greater Than Or Equal`。
- **L1558 EN**: Declares TableGen def record `SCALAR_FACGE`.
  **L1558 CN**: 声明 TableGen def 记录 `SCALAR_FACGE`。
- **L1559 EN**: Declares TableGen def record `SCALAR_FACLE`.
  **L1559 CN**: 声明 TableGen def 记录 `SCALAR_FACLE`。
- **L1560 EN**: Blank line separating nearby declarations or logic blocks.
  **L1560 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1561 EN**: Separator comment used for visual grouping.
  **L1561 CN**: 用于视觉分组的分隔注释。
- **L1562 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Floating-point Absolute Compare Mask Greater Than`.
  **L1562 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Floating-point Absolute Compare Mask Greater Than`。
- **L1563 EN**: Declares TableGen def record `SCALAR_FACGT`.
  **L1563 CN**: 声明 TableGen def 记录 `SCALAR_FACGT`。
- **L1564 EN**: Declares TableGen def record `SCALAR_FACLT`.
  **L1564 CN**: 声明 TableGen def 记录 `SCALAR_FACLT`。
- **L1565 EN**: Blank line separating nearby declarations or logic blocks.
  **L1565 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1566 EN**: Separator comment used for visual grouping.
  **L1566 CN**: 用于视觉分组的分隔注释。
- **L1567 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Absolute Value`.
  **L1567 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Absolute Value`。
- **L1568 EN**: Declares TableGen def record `SCALAR_ABS`.
  **L1568 CN**: 声明 TableGen def 记录 `SCALAR_ABS`。

### Lines 1569-1596

````tablegen

////////////////////////////////////////////////////////////////////////////////
// Scalar Absolute Difference
def SCALAR_ABD : IInst<"vabd", "111", "SfSd">;

////////////////////////////////////////////////////////////////////////////////
// Scalar Signed Saturating Absolute Value
def SCALAR_SQABS : SInst<"vqabs", "11", "ScSsSiSl">;

////////////////////////////////////////////////////////////////////////////////
// Scalar Negate
def SCALAR_NEG : SInst<"vneg", "11", "Sl">;

////////////////////////////////////////////////////////////////////////////////
// Scalar Signed Saturating Negate
def SCALAR_SQNEG : SInst<"vqneg", "11", "ScSsSiSl">;

////////////////////////////////////////////////////////////////////////////////
// Scalar Signed Saturating Accumulated of Unsigned Value
def SCALAR_SUQADD : SInst<"vuqadd", "11(1U)", "ScSsSiSl">;

////////////////////////////////////////////////////////////////////////////////
// Scalar Unsigned Saturating Accumulated of Signed Value
def SCALAR_USQADD : SInst<"vsqadd", "11(1S)", "SUcSUsSUiSUl">;

////////////////////////////////////////////////////////////////////////////////
// Signed Saturating Doubling Multiply-Add Long
def SCALAR_SQDMLAL : SInst<"vqdmlal", "(1>)(1>)11", "SsSi">;
````
- **L1569 EN**: Blank line separating nearby declarations or logic blocks.
  **L1569 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1570 EN**: Separator comment used for visual grouping.
  **L1570 CN**: 用于视觉分组的分隔注释。
- **L1571 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Absolute Difference`.
  **L1571 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Absolute Difference`。
- **L1572 EN**: Declares TableGen def record `SCALAR_ABD`.
  **L1572 CN**: 声明 TableGen def 记录 `SCALAR_ABD`。
- **L1573 EN**: Blank line separating nearby declarations or logic blocks.
  **L1573 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1574 EN**: Separator comment used for visual grouping.
  **L1574 CN**: 用于视觉分组的分隔注释。
- **L1575 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Signed Saturating Absolute Value`.
  **L1575 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Signed Saturating Absolute Value`。
- **L1576 EN**: Declares TableGen def record `SCALAR_SQABS`.
  **L1576 CN**: 声明 TableGen def 记录 `SCALAR_SQABS`。
- **L1577 EN**: Blank line separating nearby declarations or logic blocks.
  **L1577 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1578 EN**: Separator comment used for visual grouping.
  **L1578 CN**: 用于视觉分组的分隔注释。
- **L1579 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Negate`.
  **L1579 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Negate`。
- **L1580 EN**: Declares TableGen def record `SCALAR_NEG`.
  **L1580 CN**: 声明 TableGen def 记录 `SCALAR_NEG`。
- **L1581 EN**: Blank line separating nearby declarations or logic blocks.
  **L1581 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1582 EN**: Separator comment used for visual grouping.
  **L1582 CN**: 用于视觉分组的分隔注释。
- **L1583 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Signed Saturating Negate`.
  **L1583 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Signed Saturating Negate`。
- **L1584 EN**: Declares TableGen def record `SCALAR_SQNEG`.
  **L1584 CN**: 声明 TableGen def 记录 `SCALAR_SQNEG`。
- **L1585 EN**: Blank line separating nearby declarations or logic blocks.
  **L1585 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1586 EN**: Separator comment used for visual grouping.
  **L1586 CN**: 用于视觉分组的分隔注释。
- **L1587 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Signed Saturating Accumulated of Unsigned Value`.
  **L1587 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Signed Saturating Accumulated of Unsigned Value`。
- **L1588 EN**: Declares TableGen def record `SCALAR_SUQADD`.
  **L1588 CN**: 声明 TableGen def 记录 `SCALAR_SUQADD`。
- **L1589 EN**: Blank line separating nearby declarations or logic blocks.
  **L1589 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1590 EN**: Separator comment used for visual grouping.
  **L1590 CN**: 用于视觉分组的分隔注释。
- **L1591 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Unsigned Saturating Accumulated of Signed Value`.
  **L1591 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Unsigned Saturating Accumulated of Signed Value`。
- **L1592 EN**: Declares TableGen def record `SCALAR_USQADD`.
  **L1592 CN**: 声明 TableGen def 记录 `SCALAR_USQADD`。
- **L1593 EN**: Blank line separating nearby declarations or logic blocks.
  **L1593 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1594 EN**: Separator comment used for visual grouping.
  **L1594 CN**: 用于视觉分组的分隔注释。
- **L1595 EN**: Comment explains nearby logic, constraints, or intent: `Signed Saturating Doubling Multiply-Add Long`.
  **L1595 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Signed Saturating Doubling Multiply-Add Long`。
- **L1596 EN**: Declares TableGen def record `SCALAR_SQDMLAL`.
  **L1596 CN**: 声明 TableGen def 记录 `SCALAR_SQDMLAL`。

### Lines 1597-1624

````tablegen

////////////////////////////////////////////////////////////////////////////////
// Signed Saturating Doubling Multiply-Subtract Long
def SCALAR_SQDMLSL : SInst<"vqdmlsl", "(1>)(1>)11", "SsSi">;

////////////////////////////////////////////////////////////////////////////////
// Signed Saturating Doubling Multiply Long
def SCALAR_SQDMULL : SInst<"vqdmull", "(1>)11", "SsSi">;

////////////////////////////////////////////////////////////////////////////////
// Scalar Signed Saturating Extract Unsigned Narrow
def SCALAR_SQXTUN : SInst<"vqmovun", "(U1<)1", "SsSiSl">;

////////////////////////////////////////////////////////////////////////////////
// Scalar Signed Saturating Extract Narrow
def SCALAR_SQXTN : SInst<"vqmovn", "(1<)1", "SsSiSl">;

////////////////////////////////////////////////////////////////////////////////
// Scalar Unsigned Saturating Extract Narrow
def SCALAR_UQXTN : SInst<"vqmovn", "(1<)1", "SUsSUiSUl">;

// Scalar Floating Point  multiply (scalar, by element)
def SCALAR_FMUL_LANE : IOpInst<"vmul_lane", "11.I", "SfSd", OP_SCALAR_MUL_LN>;
def SCALAR_FMUL_LANEQ : IOpInst<"vmul_laneq", "11QI", "SfSd", OP_SCALAR_MUL_LN>;

// Scalar Floating Point  multiply extended (scalar, by element)
def SCALAR_FMULX_LANE : IOpInst<"vmulx_lane", "11.I", "SfSd", OP_SCALAR_MULX_LN>;
def SCALAR_FMULX_LANEQ : IOpInst<"vmulx_laneq", "11QI", "SfSd", OP_SCALAR_MULX_LN>;
````
- **L1597 EN**: Blank line separating nearby declarations or logic blocks.
  **L1597 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1598 EN**: Separator comment used for visual grouping.
  **L1598 CN**: 用于视觉分组的分隔注释。
- **L1599 EN**: Comment explains nearby logic, constraints, or intent: `Signed Saturating Doubling Multiply-Subtract Long`.
  **L1599 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Signed Saturating Doubling Multiply-Subtract Long`。
- **L1600 EN**: Declares TableGen def record `SCALAR_SQDMLSL`.
  **L1600 CN**: 声明 TableGen def 记录 `SCALAR_SQDMLSL`。
- **L1601 EN**: Blank line separating nearby declarations or logic blocks.
  **L1601 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1602 EN**: Separator comment used for visual grouping.
  **L1602 CN**: 用于视觉分组的分隔注释。
- **L1603 EN**: Comment explains nearby logic, constraints, or intent: `Signed Saturating Doubling Multiply Long`.
  **L1603 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Signed Saturating Doubling Multiply Long`。
- **L1604 EN**: Declares TableGen def record `SCALAR_SQDMULL`.
  **L1604 CN**: 声明 TableGen def 记录 `SCALAR_SQDMULL`。
- **L1605 EN**: Blank line separating nearby declarations or logic blocks.
  **L1605 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1606 EN**: Separator comment used for visual grouping.
  **L1606 CN**: 用于视觉分组的分隔注释。
- **L1607 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Signed Saturating Extract Unsigned Narrow`.
  **L1607 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Signed Saturating Extract Unsigned Narrow`。
- **L1608 EN**: Declares TableGen def record `SCALAR_SQXTUN`.
  **L1608 CN**: 声明 TableGen def 记录 `SCALAR_SQXTUN`。
- **L1609 EN**: Blank line separating nearby declarations or logic blocks.
  **L1609 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1610 EN**: Separator comment used for visual grouping.
  **L1610 CN**: 用于视觉分组的分隔注释。
- **L1611 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Signed Saturating Extract Narrow`.
  **L1611 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Signed Saturating Extract Narrow`。
- **L1612 EN**: Declares TableGen def record `SCALAR_SQXTN`.
  **L1612 CN**: 声明 TableGen def 记录 `SCALAR_SQXTN`。
- **L1613 EN**: Blank line separating nearby declarations or logic blocks.
  **L1613 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1614 EN**: Separator comment used for visual grouping.
  **L1614 CN**: 用于视觉分组的分隔注释。
- **L1615 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Unsigned Saturating Extract Narrow`.
  **L1615 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Unsigned Saturating Extract Narrow`。
- **L1616 EN**: Declares TableGen def record `SCALAR_UQXTN`.
  **L1616 CN**: 声明 TableGen def 记录 `SCALAR_UQXTN`。
- **L1617 EN**: Blank line separating nearby declarations or logic blocks.
  **L1617 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1618 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Floating Point multiply (scalar, by element)`.
  **L1618 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Floating Point multiply (scalar, by element)`。
- **L1619 EN**: Declares TableGen def record `SCALAR_FMUL_LANE`.
  **L1619 CN**: 声明 TableGen def 记录 `SCALAR_FMUL_LANE`。
- **L1620 EN**: Declares TableGen def record `SCALAR_FMUL_LANEQ`.
  **L1620 CN**: 声明 TableGen def 记录 `SCALAR_FMUL_LANEQ`。
- **L1621 EN**: Blank line separating nearby declarations or logic blocks.
  **L1621 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1622 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Floating Point multiply extended (scalar, by element)`.
  **L1622 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Floating Point multiply extended (scalar, by element)`。
- **L1623 EN**: Declares TableGen def record `SCALAR_FMULX_LANE`.
  **L1623 CN**: 声明 TableGen def 记录 `SCALAR_FMULX_LANE`。
- **L1624 EN**: Declares TableGen def record `SCALAR_FMULX_LANEQ`.
  **L1624 CN**: 声明 TableGen def 记录 `SCALAR_FMULX_LANEQ`。

### Lines 1625-1652

````tablegen

def SCALAR_VMUL_N : IInst<"vmul_n", "..1", "d">;

// VMUL_LANE_A64 d type implemented using scalar mul lane
def SCALAR_VMUL_LANE : IInst<"vmul_lane", "..qI", "d",
                            [ImmCheck<2, ImmCheckLaneIndex, 1>]>;

// VMUL_LANEQ d type implemented using scalar mul lane
def SCALAR_VMUL_LANEQ : IInst<"vmul_laneq", "..QI", "d",
                              [ImmCheck<2, ImmCheckLaneIndex, 1>]>;
// VMULX_LANE d type implemented using scalar vmulx_lane
def SCALAR_VMULX_LANE : IOpInst<"vmulx_lane", "..qI", "d", OP_SCALAR_VMULX_LN>;

// VMULX_LANEQ d type implemented using scalar vmulx_laneq
def SCALAR_VMULX_LANEQ : IOpInst<"vmulx_laneq", "..QI", "d", OP_SCALAR_VMULX_LNQ>;
// Scalar Floating Point fused multiply-add (scalar, by element)
def SCALAR_FMLA_LANE : IInst<"vfma_lane", "111.I", "SfSd",
                            [ImmCheck<3, ImmCheckLaneIndex, 2>]>;
def SCALAR_FMLA_LANEQ : IInst<"vfma_laneq", "111QI", "SfSd",
                            [ImmCheck<3, ImmCheckLaneIndex, 2>]>;

// Scalar Floating Point fused multiply-subtract (scalar, by element)
def SCALAR_FMLS_LANE : IOpInst<"vfms_lane", "111.I", "SfSd", OP_FMS_LN>;
def SCALAR_FMLS_LANEQ : IOpInst<"vfms_laneq", "111QI", "SfSd", OP_FMS_LNQ>;

// Signed Saturating Doubling Multiply Long (scalar by element)
def SCALAR_SQDMULL_LANE : SOpInst<"vqdmull_lane", "(1>)1.I", "SsSi", OP_SCALAR_QDMULL_LN>;
def SCALAR_SQDMULL_LANEQ : SOpInst<"vqdmull_laneq", "(1>)1QI", "SsSi", OP_SCALAR_QDMULL_LN>;
````
- **L1625 EN**: Blank line separating nearby declarations or logic blocks.
  **L1625 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1626 EN**: Declares TableGen def record `SCALAR_VMUL_N`.
  **L1626 CN**: 声明 TableGen def 记录 `SCALAR_VMUL_N`。
- **L1627 EN**: Blank line separating nearby declarations or logic blocks.
  **L1627 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1628 EN**: Comment explains nearby logic, constraints, or intent: `VMUL_LANE_A64 d type implemented using scalar mul lane`.
  **L1628 CN**: 注释解释附近代码的逻辑、约束或设计意图：`VMUL_LANE_A64 d type implemented using scalar mul lane`。
- **L1629 EN**: Declares TableGen def record `SCALAR_VMUL_LANE`.
  **L1629 CN**: 声明 TableGen def 记录 `SCALAR_VMUL_LANE`。
- **L1630 EN**: Adds a standalone statement or declaration: `[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`.
  **L1630 CN**: 添加一条独立语句或声明：`[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`。
- **L1631 EN**: Blank line separating nearby declarations or logic blocks.
  **L1631 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1632 EN**: Comment explains nearby logic, constraints, or intent: `VMUL_LANEQ d type implemented using scalar mul lane`.
  **L1632 CN**: 注释解释附近代码的逻辑、约束或设计意图：`VMUL_LANEQ d type implemented using scalar mul lane`。
- **L1633 EN**: Declares TableGen def record `SCALAR_VMUL_LANEQ`.
  **L1633 CN**: 声明 TableGen def 记录 `SCALAR_VMUL_LANEQ`。
- **L1634 EN**: Adds a standalone statement or declaration: `[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`.
  **L1634 CN**: 添加一条独立语句或声明：`[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`。
- **L1635 EN**: Comment explains nearby logic, constraints, or intent: `VMULX_LANE d type implemented using scalar vmulx_lane`.
  **L1635 CN**: 注释解释附近代码的逻辑、约束或设计意图：`VMULX_LANE d type implemented using scalar vmulx_lane`。
- **L1636 EN**: Declares TableGen def record `SCALAR_VMULX_LANE`.
  **L1636 CN**: 声明 TableGen def 记录 `SCALAR_VMULX_LANE`。
- **L1637 EN**: Blank line separating nearby declarations or logic blocks.
  **L1637 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1638 EN**: Comment explains nearby logic, constraints, or intent: `VMULX_LANEQ d type implemented using scalar vmulx_laneq`.
  **L1638 CN**: 注释解释附近代码的逻辑、约束或设计意图：`VMULX_LANEQ d type implemented using scalar vmulx_laneq`。
- **L1639 EN**: Declares TableGen def record `SCALAR_VMULX_LANEQ`.
  **L1639 CN**: 声明 TableGen def 记录 `SCALAR_VMULX_LANEQ`。
- **L1640 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Floating Point fused multiply-add (scalar, by element)`.
  **L1640 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Floating Point fused multiply-add (scalar, by element)`。
- **L1641 EN**: Declares TableGen def record `SCALAR_FMLA_LANE`.
  **L1641 CN**: 声明 TableGen def 记录 `SCALAR_FMLA_LANE`。
- **L1642 EN**: Adds a standalone statement or declaration: `[ImmCheck<3, ImmCheckLaneIndex, 2>]>;`.
  **L1642 CN**: 添加一条独立语句或声明：`[ImmCheck<3, ImmCheckLaneIndex, 2>]>;`。
- **L1643 EN**: Declares TableGen def record `SCALAR_FMLA_LANEQ`.
  **L1643 CN**: 声明 TableGen def 记录 `SCALAR_FMLA_LANEQ`。
- **L1644 EN**: Adds a standalone statement or declaration: `[ImmCheck<3, ImmCheckLaneIndex, 2>]>;`.
  **L1644 CN**: 添加一条独立语句或声明：`[ImmCheck<3, ImmCheckLaneIndex, 2>]>;`。
- **L1645 EN**: Blank line separating nearby declarations or logic blocks.
  **L1645 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1646 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Floating Point fused multiply-subtract (scalar, by element)`.
  **L1646 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Floating Point fused multiply-subtract (scalar, by element)`。
- **L1647 EN**: Declares TableGen def record `SCALAR_FMLS_LANE`.
  **L1647 CN**: 声明 TableGen def 记录 `SCALAR_FMLS_LANE`。
- **L1648 EN**: Declares TableGen def record `SCALAR_FMLS_LANEQ`.
  **L1648 CN**: 声明 TableGen def 记录 `SCALAR_FMLS_LANEQ`。
- **L1649 EN**: Blank line separating nearby declarations or logic blocks.
  **L1649 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1650 EN**: Comment explains nearby logic, constraints, or intent: `Signed Saturating Doubling Multiply Long (scalar by element)`.
  **L1650 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Signed Saturating Doubling Multiply Long (scalar by element)`。
- **L1651 EN**: Declares TableGen def record `SCALAR_SQDMULL_LANE`.
  **L1651 CN**: 声明 TableGen def 记录 `SCALAR_SQDMULL_LANE`。
- **L1652 EN**: Declares TableGen def record `SCALAR_SQDMULL_LANEQ`.
  **L1652 CN**: 声明 TableGen def 记录 `SCALAR_SQDMULL_LANEQ`。

### Lines 1653-1680

````tablegen

// Signed Saturating Doubling Multiply-Add Long (scalar by element)
def SCALAR_SQDMLAL_LANE : SInst<"vqdmlal_lane", "(1>)(1>)1.I", "SsSi",
                                [ImmCheck<3, ImmCheckLaneIndex, 1>]>;
def SCALAR_SQDMLAL_LANEQ : SInst<"vqdmlal_laneq", "(1>)(1>)1QI", "SsSi",
                                [ImmCheck<3, ImmCheckLaneIndex, 2>]>;

// Signed Saturating Doubling Multiply-Subtract Long (scalar by element)
def SCALAR_SQDMLS_LANE : SInst<"vqdmlsl_lane", "(1>)(1>)1.I", "SsSi",
                              [ImmCheck<3, ImmCheckLaneIndex, 1>]>;
def SCALAR_SQDMLS_LANEQ : SInst<"vqdmlsl_laneq", "(1>)(1>)1QI", "SsSi",
                              [ImmCheck<3, ImmCheckLaneIndex, 2>]>;
// Scalar Integer Saturating Doubling Multiply Half High (scalar by element)
def SCALAR_SQDMULH_LANE : SOpInst<"vqdmulh_lane", "11.I", "SsSi", OP_SCALAR_QDMULH_LN>;
def SCALAR_SQDMULH_LANEQ : SOpInst<"vqdmulh_laneq", "11QI", "SsSi", OP_SCALAR_QDMULH_LN>;

// Scalar Integer Saturating Rounding Doubling Multiply Half High
def SCALAR_SQRDMULH_LANE : SOpInst<"vqrdmulh_lane", "11.I", "SsSi", OP_SCALAR_QRDMULH_LN>;
def SCALAR_SQRDMULH_LANEQ : SOpInst<"vqrdmulh_laneq", "11QI", "SsSi", OP_SCALAR_QRDMULH_LN>;

let TargetGuard = "v8.1a,neon" in {
// Signed Saturating Rounding Doubling Multiply Accumulate Returning High Half
def SCALAR_SQRDMLAH_LANE : SOpInst<"vqrdmlah_lane", "111.I", "SsSi", OP_SCALAR_QRDMLAH_LN>;
def SCALAR_SQRDMLAH_LANEQ : SOpInst<"vqrdmlah_laneq", "111QI", "SsSi", OP_SCALAR_QRDMLAH_LN>;
// Signed Saturating Rounding Doubling Multiply Subtract Returning High Half
def SCALAR_SQRDMLSH_LANE : SOpInst<"vqrdmlsh_lane", "111.I", "SsSi", OP_SCALAR_QRDMLSH_LN>;
def SCALAR_SQRDMLSH_LANEQ : SOpInst<"vqrdmlsh_laneq", "111QI", "SsSi", OP_SCALAR_QRDMLSH_LN>;
} // TargetGuard = "v8.1a"
````
- **L1653 EN**: Blank line separating nearby declarations or logic blocks.
  **L1653 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1654 EN**: Comment explains nearby logic, constraints, or intent: `Signed Saturating Doubling Multiply-Add Long (scalar by element)`.
  **L1654 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Signed Saturating Doubling Multiply-Add Long (scalar by element)`。
- **L1655 EN**: Declares TableGen def record `SCALAR_SQDMLAL_LANE`.
  **L1655 CN**: 声明 TableGen def 记录 `SCALAR_SQDMLAL_LANE`。
- **L1656 EN**: Adds a standalone statement or declaration: `[ImmCheck<3, ImmCheckLaneIndex, 1>]>;`.
  **L1656 CN**: 添加一条独立语句或声明：`[ImmCheck<3, ImmCheckLaneIndex, 1>]>;`。
- **L1657 EN**: Declares TableGen def record `SCALAR_SQDMLAL_LANEQ`.
  **L1657 CN**: 声明 TableGen def 记录 `SCALAR_SQDMLAL_LANEQ`。
- **L1658 EN**: Adds a standalone statement or declaration: `[ImmCheck<3, ImmCheckLaneIndex, 2>]>;`.
  **L1658 CN**: 添加一条独立语句或声明：`[ImmCheck<3, ImmCheckLaneIndex, 2>]>;`。
- **L1659 EN**: Blank line separating nearby declarations or logic blocks.
  **L1659 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1660 EN**: Comment explains nearby logic, constraints, or intent: `Signed Saturating Doubling Multiply-Subtract Long (scalar by element)`.
  **L1660 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Signed Saturating Doubling Multiply-Subtract Long (scalar by element)`。
- **L1661 EN**: Declares TableGen def record `SCALAR_SQDMLS_LANE`.
  **L1661 CN**: 声明 TableGen def 记录 `SCALAR_SQDMLS_LANE`。
- **L1662 EN**: Adds a standalone statement or declaration: `[ImmCheck<3, ImmCheckLaneIndex, 1>]>;`.
  **L1662 CN**: 添加一条独立语句或声明：`[ImmCheck<3, ImmCheckLaneIndex, 1>]>;`。
- **L1663 EN**: Declares TableGen def record `SCALAR_SQDMLS_LANEQ`.
  **L1663 CN**: 声明 TableGen def 记录 `SCALAR_SQDMLS_LANEQ`。
- **L1664 EN**: Adds a standalone statement or declaration: `[ImmCheck<3, ImmCheckLaneIndex, 2>]>;`.
  **L1664 CN**: 添加一条独立语句或声明：`[ImmCheck<3, ImmCheckLaneIndex, 2>]>;`。
- **L1665 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Integer Saturating Doubling Multiply Half High (scalar by element)`.
  **L1665 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Integer Saturating Doubling Multiply Half High (scalar by element)`。
- **L1666 EN**: Declares TableGen def record `SCALAR_SQDMULH_LANE`.
  **L1666 CN**: 声明 TableGen def 记录 `SCALAR_SQDMULH_LANE`。
- **L1667 EN**: Declares TableGen def record `SCALAR_SQDMULH_LANEQ`.
  **L1667 CN**: 声明 TableGen def 记录 `SCALAR_SQDMULH_LANEQ`。
- **L1668 EN**: Blank line separating nearby declarations or logic blocks.
  **L1668 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1669 EN**: Comment explains nearby logic, constraints, or intent: `Scalar Integer Saturating Rounding Doubling Multiply Half High`.
  **L1669 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar Integer Saturating Rounding Doubling Multiply Half High`。
- **L1670 EN**: Declares TableGen def record `SCALAR_SQRDMULH_LANE`.
  **L1670 CN**: 声明 TableGen def 记录 `SCALAR_SQRDMULH_LANE`。
- **L1671 EN**: Declares TableGen def record `SCALAR_SQRDMULH_LANEQ`.
  **L1671 CN**: 声明 TableGen def 记录 `SCALAR_SQRDMULH_LANEQ`。
- **L1672 EN**: Blank line separating nearby declarations or logic blocks.
  **L1672 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1673 EN**: Assigns a TableGen property that affects following records or inherited fields: `let TargetGuard = "v8.1a,neon" in {`.
  **L1673 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let TargetGuard = "v8.1a,neon" in {`。
- **L1674 EN**: Comment explains nearby logic, constraints, or intent: `Signed Saturating Rounding Doubling Multiply Accumulate Returning High Half`.
  **L1674 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Signed Saturating Rounding Doubling Multiply Accumulate Returning High Half`。
- **L1675 EN**: Declares TableGen def record `SCALAR_SQRDMLAH_LANE`.
  **L1675 CN**: 声明 TableGen def 记录 `SCALAR_SQRDMLAH_LANE`。
- **L1676 EN**: Declares TableGen def record `SCALAR_SQRDMLAH_LANEQ`.
  **L1676 CN**: 声明 TableGen def 记录 `SCALAR_SQRDMLAH_LANEQ`。
- **L1677 EN**: Comment explains nearby logic, constraints, or intent: `Signed Saturating Rounding Doubling Multiply Subtract Returning High Half`.
  **L1677 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Signed Saturating Rounding Doubling Multiply Subtract Returning High Half`。
- **L1678 EN**: Declares TableGen def record `SCALAR_SQRDMLSH_LANE`.
  **L1678 CN**: 声明 TableGen def 记录 `SCALAR_SQRDMLSH_LANE`。
- **L1679 EN**: Declares TableGen def record `SCALAR_SQRDMLSH_LANEQ`.
  **L1679 CN**: 声明 TableGen def 记录 `SCALAR_SQRDMLSH_LANEQ`。
- **L1680 EN**: Continues the surrounding expression or declaration: `} // TargetGuard = "v8.1a"`.
  **L1680 CN**: 继续构造周围的表达式或声明：`} // TargetGuard = "v8.1a"`。

### Lines 1681-1708

````tablegen

def SCALAR_VDUP_LANE : IInst<"vdup_lane", "1.I", "ScSsSiSlSfSdSUcSUsSUiSUlSPcSPsSm",
                            [ImmCheck<1, ImmCheckLaneIndex, 0>]>;
def SCALAR_VDUP_LANEQ : IInst<"vdup_laneq", "1QI", "ScSsSiSlSfSdSUcSUsSUiSUlSPcSPsSm",
                            [ImmCheck<1, ImmCheckLaneIndex, 0>]>;

} // ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)"

// ARMv8.2-A FP16 vector intrinsics for A32/A64.
let TargetGuard = "fullfp16,neon" in {

  // ARMv8.2-A FP16 one-operand vector intrinsics.

  // Comparison
  def CMEQH    : SInst<"vceqz", "U(.!)", "hQh">;
  def CMGEH    : SInst<"vcgez", "U(.!)", "hQh">;
  def CMGTH    : SInst<"vcgtz", "U(.!)", "hQh">;
  def CMLEH    : SInst<"vclez", "U(.!)", "hQh">;
  def CMLTH    : SInst<"vcltz", "U(.!)", "hQh">;

  // Vector conversion
  def VCVT_F16     : SInst<"vcvt_f16", "F(.!)",  "sUsQsQUs">;
  def VCVT_S16     : SInst<"vcvt_s16", "S.",  "hQh">;
  def VCVT_U16     : SInst<"vcvt_u16", "U.",  "hQh">;
  def VCVTA_S16    : SInst<"vcvta_s16", "S.", "hQh">;
  def VCVTA_U16    : SInst<"vcvta_u16", "U.", "hQh">;
  def VCVTM_S16    : SInst<"vcvtm_s16", "S.", "hQh">;
  def VCVTM_U16    : SInst<"vcvtm_u16", "U.", "hQh">;
````
- **L1681 EN**: Blank line separating nearby declarations or logic blocks.
  **L1681 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1682 EN**: Declares TableGen def record `SCALAR_VDUP_LANE`.
  **L1682 CN**: 声明 TableGen def 记录 `SCALAR_VDUP_LANE`。
- **L1683 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheckLaneIndex, 0>]>;`.
  **L1683 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheckLaneIndex, 0>]>;`。
- **L1684 EN**: Declares TableGen def record `SCALAR_VDUP_LANEQ`.
  **L1684 CN**: 声明 TableGen def 记录 `SCALAR_VDUP_LANEQ`。
- **L1685 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheckLaneIndex, 0>]>;`.
  **L1685 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheckLaneIndex, 0>]>;`。
- **L1686 EN**: Blank line separating nearby declarations or logic blocks.
  **L1686 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1687 EN**: Continues logic associated with callable symbol `defined`.
  **L1687 CN**: 继续与可调用符号 `defined` 相关的逻辑。
- **L1688 EN**: Blank line separating nearby declarations or logic blocks.
  **L1688 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1689 EN**: Comment explains nearby logic, constraints, or intent: `ARMv8.2-A FP16 vector intrinsics for A32/A64.`.
  **L1689 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ARMv8.2-A FP16 vector intrinsics for A32/A64.`。
- **L1690 EN**: Assigns a TableGen property that affects following records or inherited fields: `let TargetGuard = "fullfp16,neon" in {`.
  **L1690 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let TargetGuard = "fullfp16,neon" in {`。
- **L1691 EN**: Blank line separating nearby declarations or logic blocks.
  **L1691 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1692 EN**: Comment explains nearby logic, constraints, or intent: `ARMv8.2-A FP16 one-operand vector intrinsics.`.
  **L1692 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ARMv8.2-A FP16 one-operand vector intrinsics.`。
- **L1693 EN**: Blank line separating nearby declarations or logic blocks.
  **L1693 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1694 EN**: Comment explains nearby logic, constraints, or intent: `Comparison`.
  **L1694 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Comparison`。
- **L1695 EN**: Declares TableGen def record `CMEQH`.
  **L1695 CN**: 声明 TableGen def 记录 `CMEQH`。
- **L1696 EN**: Declares TableGen def record `CMGEH`.
  **L1696 CN**: 声明 TableGen def 记录 `CMGEH`。
- **L1697 EN**: Declares TableGen def record `CMGTH`.
  **L1697 CN**: 声明 TableGen def 记录 `CMGTH`。
- **L1698 EN**: Declares TableGen def record `CMLEH`.
  **L1698 CN**: 声明 TableGen def 记录 `CMLEH`。
- **L1699 EN**: Declares TableGen def record `CMLTH`.
  **L1699 CN**: 声明 TableGen def 记录 `CMLTH`。
- **L1700 EN**: Blank line separating nearby declarations or logic blocks.
  **L1700 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1701 EN**: Comment explains nearby logic, constraints, or intent: `Vector conversion`.
  **L1701 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Vector conversion`。
- **L1702 EN**: Declares TableGen def record `VCVT_F16`.
  **L1702 CN**: 声明 TableGen def 记录 `VCVT_F16`。
- **L1703 EN**: Declares TableGen def record `VCVT_S16`.
  **L1703 CN**: 声明 TableGen def 记录 `VCVT_S16`。
- **L1704 EN**: Declares TableGen def record `VCVT_U16`.
  **L1704 CN**: 声明 TableGen def 记录 `VCVT_U16`。
- **L1705 EN**: Declares TableGen def record `VCVTA_S16`.
  **L1705 CN**: 声明 TableGen def 记录 `VCVTA_S16`。
- **L1706 EN**: Declares TableGen def record `VCVTA_U16`.
  **L1706 CN**: 声明 TableGen def 记录 `VCVTA_U16`。
- **L1707 EN**: Declares TableGen def record `VCVTM_S16`.
  **L1707 CN**: 声明 TableGen def 记录 `VCVTM_S16`。
- **L1708 EN**: Declares TableGen def record `VCVTM_U16`.
  **L1708 CN**: 声明 TableGen def 记录 `VCVTM_U16`。

### Lines 1709-1736

````tablegen
  def VCVTN_S16    : SInst<"vcvtn_s16", "S.", "hQh">;
  def VCVTN_U16    : SInst<"vcvtn_u16", "U.", "hQh">;
  def VCVTP_S16    : SInst<"vcvtp_s16", "S.", "hQh">;
  def VCVTP_U16    : SInst<"vcvtp_u16", "U.", "hQh">;

  // Vector rounding
  let ArchGuard = "__ARM_ARCH >= 8 && defined(__ARM_FEATURE_DIRECTED_ROUNDING)", TargetGuard = "fullfp16,neon" in {
    def FRINTZH      : SInst<"vrnd",  "..", "hQh">;
    def FRINTNH      : SInst<"vrndn", "..", "hQh">;
    def FRINTAH      : SInst<"vrnda", "..", "hQh">;
    def FRINTPH      : SInst<"vrndp", "..", "hQh">;
    def FRINTMH      : SInst<"vrndm", "..", "hQh">;
    def FRINTXH      : SInst<"vrndx", "..", "hQh">;
  }

  // Misc.
  def VABSH        : SInst<"vabs", "..", "hQh">;
  def VNEGH        : SOpInst<"vneg", "..", "hQh", OP_NEG>;
  def VRECPEH      : SInst<"vrecpe", "..", "hQh">;
  def FRSQRTEH     : SInst<"vrsqrte", "..", "hQh">;

  // ARMv8.2-A FP16 two-operands vector intrinsics.

  // Misc.
  def VADDH        : SOpInst<"vadd", "...", "hQh", OP_ADD>;
  def VABDH        : SInst<"vabd", "...",  "hQh">;
  def VSUBH         : SOpInst<"vsub", "...", "hQh", OP_SUB>;

````
- **L1709 EN**: Declares TableGen def record `VCVTN_S16`.
  **L1709 CN**: 声明 TableGen def 记录 `VCVTN_S16`。
- **L1710 EN**: Declares TableGen def record `VCVTN_U16`.
  **L1710 CN**: 声明 TableGen def 记录 `VCVTN_U16`。
- **L1711 EN**: Declares TableGen def record `VCVTP_S16`.
  **L1711 CN**: 声明 TableGen def 记录 `VCVTP_S16`。
- **L1712 EN**: Declares TableGen def record `VCVTP_U16`.
  **L1712 CN**: 声明 TableGen def 记录 `VCVTP_U16`。
- **L1713 EN**: Blank line separating nearby declarations or logic blocks.
  **L1713 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1714 EN**: Comment explains nearby logic, constraints, or intent: `Vector rounding`.
  **L1714 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Vector rounding`。
- **L1715 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "__ARM_ARCH >= 8 && defined(__ARM_FEATURE_DIRECTED_ROUNDING)", TargetGuard = "fullfp16,neon" in {`.
  **L1715 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "__ARM_ARCH >= 8 && defined(__ARM_FEATURE_DIRECTED_ROUNDING)", TargetGuard = "fullfp16,neon" in {`。
- **L1716 EN**: Declares TableGen def record `FRINTZH`.
  **L1716 CN**: 声明 TableGen def 记录 `FRINTZH`。
- **L1717 EN**: Declares TableGen def record `FRINTNH`.
  **L1717 CN**: 声明 TableGen def 记录 `FRINTNH`。
- **L1718 EN**: Declares TableGen def record `FRINTAH`.
  **L1718 CN**: 声明 TableGen def 记录 `FRINTAH`。
- **L1719 EN**: Declares TableGen def record `FRINTPH`.
  **L1719 CN**: 声明 TableGen def 记录 `FRINTPH`。
- **L1720 EN**: Declares TableGen def record `FRINTMH`.
  **L1720 CN**: 声明 TableGen def 记录 `FRINTMH`。
- **L1721 EN**: Declares TableGen def record `FRINTXH`.
  **L1721 CN**: 声明 TableGen def 记录 `FRINTXH`。
- **L1722 EN**: Closes the current lexical scope or compound statement.
  **L1722 CN**: 结束当前词法作用域或复合语句块。
- **L1723 EN**: Blank line separating nearby declarations or logic blocks.
  **L1723 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1724 EN**: Comment explains nearby logic, constraints, or intent: `Misc.`.
  **L1724 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Misc.`。
- **L1725 EN**: Declares TableGen def record `VABSH`.
  **L1725 CN**: 声明 TableGen def 记录 `VABSH`。
- **L1726 EN**: Declares TableGen def record `VNEGH`.
  **L1726 CN**: 声明 TableGen def 记录 `VNEGH`。
- **L1727 EN**: Declares TableGen def record `VRECPEH`.
  **L1727 CN**: 声明 TableGen def 记录 `VRECPEH`。
- **L1728 EN**: Declares TableGen def record `FRSQRTEH`.
  **L1728 CN**: 声明 TableGen def 记录 `FRSQRTEH`。
- **L1729 EN**: Blank line separating nearby declarations or logic blocks.
  **L1729 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1730 EN**: Comment explains nearby logic, constraints, or intent: `ARMv8.2-A FP16 two-operands vector intrinsics.`.
  **L1730 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ARMv8.2-A FP16 two-operands vector intrinsics.`。
- **L1731 EN**: Blank line separating nearby declarations or logic blocks.
  **L1731 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1732 EN**: Comment explains nearby logic, constraints, or intent: `Misc.`.
  **L1732 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Misc.`。
- **L1733 EN**: Declares TableGen def record `VADDH`.
  **L1733 CN**: 声明 TableGen def 记录 `VADDH`。
- **L1734 EN**: Declares TableGen def record `VABDH`.
  **L1734 CN**: 声明 TableGen def 记录 `VABDH`。
- **L1735 EN**: Declares TableGen def record `VSUBH`.
  **L1735 CN**: 声明 TableGen def 记录 `VSUBH`。
- **L1736 EN**: Blank line separating nearby declarations or logic blocks.
  **L1736 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1737-1764

````tablegen
  // Comparison
  let InstName = "vacge" in {
    def VCAGEH     : SInst<"vcage", "U..", "hQh">;
    def VCALEH     : SInst<"vcale", "U..", "hQh">;
  }
  let InstName = "vacgt" in {
    def VCAGTH     : SInst<"vcagt", "U..", "hQh">;
    def VCALTH     : SInst<"vcalt", "U..", "hQh">;
  }
  def VCEQH        : SOpInst<"vceq", "U..", "hQh", OP_EQ>;
  def VCGEH        : SOpInst<"vcge", "U..", "hQh", OP_GE>;
  def VCGTH        : SOpInst<"vcgt", "U..", "hQh", OP_GT>;
  let InstName = "vcge" in
    def VCLEH      : SOpInst<"vcle", "U..", "hQh", OP_LE>;
  let InstName = "vcgt" in
    def VCLTH      : SOpInst<"vclt", "U..", "hQh", OP_LT>;

  // Vector conversion
    def VCVT_N_F16 : SInst<"vcvt_n_f16", "F(.!)I", "sUsQsQUs",
                          [ImmCheck<1, ImmCheck1_16>]>;
    def VCVT_N_S16 : SInst<"vcvt_n_s16", "S.I", "hQh",
                          [ImmCheck<1, ImmCheck1_16>]>;
    def VCVT_N_U16 : SInst<"vcvt_n_u16", "U.I", "hQh",
                          [ImmCheck<1, ImmCheck1_16>]>;

  // Max/Min
  def VMAXH         : SInst<"vmax", "...", "hQh">;
  def VMINH         : SInst<"vmin", "...", "hQh">;
````
- **L1737 EN**: Comment explains nearby logic, constraints, or intent: `Comparison`.
  **L1737 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Comparison`。
- **L1738 EN**: Assigns a TableGen property that affects following records or inherited fields: `let InstName = "vacge" in {`.
  **L1738 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let InstName = "vacge" in {`。
- **L1739 EN**: Declares TableGen def record `VCAGEH`.
  **L1739 CN**: 声明 TableGen def 记录 `VCAGEH`。
- **L1740 EN**: Declares TableGen def record `VCALEH`.
  **L1740 CN**: 声明 TableGen def 记录 `VCALEH`。
- **L1741 EN**: Closes the current lexical scope or compound statement.
  **L1741 CN**: 结束当前词法作用域或复合语句块。
- **L1742 EN**: Assigns a TableGen property that affects following records or inherited fields: `let InstName = "vacgt" in {`.
  **L1742 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let InstName = "vacgt" in {`。
- **L1743 EN**: Declares TableGen def record `VCAGTH`.
  **L1743 CN**: 声明 TableGen def 记录 `VCAGTH`。
- **L1744 EN**: Declares TableGen def record `VCALTH`.
  **L1744 CN**: 声明 TableGen def 记录 `VCALTH`。
- **L1745 EN**: Closes the current lexical scope or compound statement.
  **L1745 CN**: 结束当前词法作用域或复合语句块。
- **L1746 EN**: Declares TableGen def record `VCEQH`.
  **L1746 CN**: 声明 TableGen def 记录 `VCEQH`。
- **L1747 EN**: Declares TableGen def record `VCGEH`.
  **L1747 CN**: 声明 TableGen def 记录 `VCGEH`。
- **L1748 EN**: Declares TableGen def record `VCGTH`.
  **L1748 CN**: 声明 TableGen def 记录 `VCGTH`。
- **L1749 EN**: Assigns a TableGen property that affects following records or inherited fields: `let InstName = "vcge" in`.
  **L1749 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let InstName = "vcge" in`。
- **L1750 EN**: Declares TableGen def record `VCLEH`.
  **L1750 CN**: 声明 TableGen def 记录 `VCLEH`。
- **L1751 EN**: Assigns a TableGen property that affects following records or inherited fields: `let InstName = "vcgt" in`.
  **L1751 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let InstName = "vcgt" in`。
- **L1752 EN**: Declares TableGen def record `VCLTH`.
  **L1752 CN**: 声明 TableGen def 记录 `VCLTH`。
- **L1753 EN**: Blank line separating nearby declarations or logic blocks.
  **L1753 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1754 EN**: Comment explains nearby logic, constraints, or intent: `Vector conversion`.
  **L1754 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Vector conversion`。
- **L1755 EN**: Declares TableGen def record `VCVT_N_F16`.
  **L1755 CN**: 声明 TableGen def 记录 `VCVT_N_F16`。
- **L1756 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheck1_16>]>;`.
  **L1756 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheck1_16>]>;`。
- **L1757 EN**: Declares TableGen def record `VCVT_N_S16`.
  **L1757 CN**: 声明 TableGen def 记录 `VCVT_N_S16`。
- **L1758 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheck1_16>]>;`.
  **L1758 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheck1_16>]>;`。
- **L1759 EN**: Declares TableGen def record `VCVT_N_U16`.
  **L1759 CN**: 声明 TableGen def 记录 `VCVT_N_U16`。
- **L1760 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheck1_16>]>;`.
  **L1760 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheck1_16>]>;`。
- **L1761 EN**: Blank line separating nearby declarations or logic blocks.
  **L1761 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1762 EN**: Comment explains nearby logic, constraints, or intent: `Max/Min`.
  **L1762 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Max/Min`。
- **L1763 EN**: Declares TableGen def record `VMAXH`.
  **L1763 CN**: 声明 TableGen def 记录 `VMAXH`。
- **L1764 EN**: Declares TableGen def record `VMINH`.
  **L1764 CN**: 声明 TableGen def 记录 `VMINH`。

### Lines 1765-1792

````tablegen
  let ArchGuard = "__ARM_ARCH >= 8 && defined(__ARM_FEATURE_NUMERIC_MAXMIN)", TargetGuard = "fullfp16,neon" in {
    def FMAXNMH       : SInst<"vmaxnm", "...", "hQh">;
    def FMINNMH       : SInst<"vminnm", "...", "hQh">;
  }

  // Multiplication/Division
  def VMULH         : SOpInst<"vmul", "...", "hQh", OP_MUL>;

  // Pairwise addition
  def VPADDH        : SInst<"vpadd", "...", "h">;

  // Pairwise Max/Min
  def VPMAXH        : SInst<"vpmax", "...", "h">;
  def VPMINH        : SInst<"vpmin", "...", "h">;

  // Reciprocal/Sqrt
  def VRECPSH       : SInst<"vrecps", "...", "hQh">;
  def VRSQRTSH      : SInst<"vrsqrts", "...", "hQh">;

  // ARMv8.2-A FP16 three-operands vector intrinsics.

  // Vector fused multiply-add operations
  def VFMAH        : SInst<"vfma", "....", "hQh">;
  def VFMSH        : SOpInst<"vfms", "....", "hQh", OP_FMLS>;

  // ARMv8.2-A FP16 lane vector intrinsics.

  // Mul lane
````
- **L1765 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "__ARM_ARCH >= 8 && defined(__ARM_FEATURE_NUMERIC_MAXMIN)", TargetGuard = "fullfp16,neon" in {`.
  **L1765 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "__ARM_ARCH >= 8 && defined(__ARM_FEATURE_NUMERIC_MAXMIN)", TargetGuard = "fullfp16,neon" in {`。
- **L1766 EN**: Declares TableGen def record `FMAXNMH`.
  **L1766 CN**: 声明 TableGen def 记录 `FMAXNMH`。
- **L1767 EN**: Declares TableGen def record `FMINNMH`.
  **L1767 CN**: 声明 TableGen def 记录 `FMINNMH`。
- **L1768 EN**: Closes the current lexical scope or compound statement.
  **L1768 CN**: 结束当前词法作用域或复合语句块。
- **L1769 EN**: Blank line separating nearby declarations or logic blocks.
  **L1769 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1770 EN**: Comment explains nearby logic, constraints, or intent: `Multiplication/Division`.
  **L1770 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Multiplication/Division`。
- **L1771 EN**: Declares TableGen def record `VMULH`.
  **L1771 CN**: 声明 TableGen def 记录 `VMULH`。
- **L1772 EN**: Blank line separating nearby declarations or logic blocks.
  **L1772 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1773 EN**: Comment explains nearby logic, constraints, or intent: `Pairwise addition`.
  **L1773 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Pairwise addition`。
- **L1774 EN**: Declares TableGen def record `VPADDH`.
  **L1774 CN**: 声明 TableGen def 记录 `VPADDH`。
- **L1775 EN**: Blank line separating nearby declarations or logic blocks.
  **L1775 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1776 EN**: Comment explains nearby logic, constraints, or intent: `Pairwise Max/Min`.
  **L1776 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Pairwise Max/Min`。
- **L1777 EN**: Declares TableGen def record `VPMAXH`.
  **L1777 CN**: 声明 TableGen def 记录 `VPMAXH`。
- **L1778 EN**: Declares TableGen def record `VPMINH`.
  **L1778 CN**: 声明 TableGen def 记录 `VPMINH`。
- **L1779 EN**: Blank line separating nearby declarations or logic blocks.
  **L1779 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1780 EN**: Comment explains nearby logic, constraints, or intent: `Reciprocal/Sqrt`.
  **L1780 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Reciprocal/Sqrt`。
- **L1781 EN**: Declares TableGen def record `VRECPSH`.
  **L1781 CN**: 声明 TableGen def 记录 `VRECPSH`。
- **L1782 EN**: Declares TableGen def record `VRSQRTSH`.
  **L1782 CN**: 声明 TableGen def 记录 `VRSQRTSH`。
- **L1783 EN**: Blank line separating nearby declarations or logic blocks.
  **L1783 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1784 EN**: Comment explains nearby logic, constraints, or intent: `ARMv8.2-A FP16 three-operands vector intrinsics.`.
  **L1784 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ARMv8.2-A FP16 three-operands vector intrinsics.`。
- **L1785 EN**: Blank line separating nearby declarations or logic blocks.
  **L1785 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1786 EN**: Comment explains nearby logic, constraints, or intent: `Vector fused multiply-add operations`.
  **L1786 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Vector fused multiply-add operations`。
- **L1787 EN**: Declares TableGen def record `VFMAH`.
  **L1787 CN**: 声明 TableGen def 记录 `VFMAH`。
- **L1788 EN**: Declares TableGen def record `VFMSH`.
  **L1788 CN**: 声明 TableGen def 记录 `VFMSH`。
- **L1789 EN**: Blank line separating nearby declarations or logic blocks.
  **L1789 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1790 EN**: Comment explains nearby logic, constraints, or intent: `ARMv8.2-A FP16 lane vector intrinsics.`.
  **L1790 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ARMv8.2-A FP16 lane vector intrinsics.`。
- **L1791 EN**: Blank line separating nearby declarations or logic blocks.
  **L1791 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1792 EN**: Comment explains nearby logic, constraints, or intent: `Mul lane`.
  **L1792 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Mul lane`。

### Lines 1793-1820

````tablegen
  def VMUL_LANEH    : IOpInst<"vmul_lane", "..qI", "hQh", OP_MUL_LN>;
  def VMUL_NH       : IOpInst<"vmul_n", "..1", "hQh", OP_MUL_N>;
}

// Data processing intrinsics - section 5. Do not require fullfp16.

// Logical operations
let isHiddenLInst = 1 in
def VBSLH    : SInst<"vbsl", ".U..", "hQh">;
// Transposition operations
def VZIPH    : WInst<"vzip", "2..", "hQh">;
def VUZPH    : WInst<"vuzp", "2..", "hQh">;
def VTRNH    : WInst<"vtrn", "2..", "hQh">;
// Vector Extract
def VEXTH      : WInst<"vext", "...I", "hQh", [ImmCheck<2, ImmCheckLaneIndex, 0>]>;
// Reverse vector elements
def VREV64H    : WOpInst<"vrev64", "..", "hQh", OP_REV64>;

// ARMv8.2-A FP16 vector intrinsics for A64 only.
let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "fullfp16,neon" in {

  // Vector rounding
  def FRINTIH      : SInst<"vrndi", "..", "hQh">;

  // Misc.
  def FSQRTH       : SInst<"vsqrt", "..", "hQh">;

  // Multiplication/Division
````
- **L1793 EN**: Declares TableGen def record `VMUL_LANEH`.
  **L1793 CN**: 声明 TableGen def 记录 `VMUL_LANEH`。
- **L1794 EN**: Declares TableGen def record `VMUL_NH`.
  **L1794 CN**: 声明 TableGen def 记录 `VMUL_NH`。
- **L1795 EN**: Closes the current lexical scope or compound statement.
  **L1795 CN**: 结束当前词法作用域或复合语句块。
- **L1796 EN**: Blank line separating nearby declarations or logic blocks.
  **L1796 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1797 EN**: Comment explains nearby logic, constraints, or intent: `Data processing intrinsics - section 5. Do not require fullfp16.`.
  **L1797 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Data processing intrinsics - section 5. Do not require fullfp16.`。
- **L1798 EN**: Blank line separating nearby declarations or logic blocks.
  **L1798 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1799 EN**: Comment explains nearby logic, constraints, or intent: `Logical operations`.
  **L1799 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Logical operations`。
- **L1800 EN**: Assigns a TableGen property that affects following records or inherited fields: `let isHiddenLInst = 1 in`.
  **L1800 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let isHiddenLInst = 1 in`。
- **L1801 EN**: Declares TableGen def record `VBSLH`.
  **L1801 CN**: 声明 TableGen def 记录 `VBSLH`。
- **L1802 EN**: Comment explains nearby logic, constraints, or intent: `Transposition operations`.
  **L1802 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Transposition operations`。
- **L1803 EN**: Declares TableGen def record `VZIPH`.
  **L1803 CN**: 声明 TableGen def 记录 `VZIPH`。
- **L1804 EN**: Declares TableGen def record `VUZPH`.
  **L1804 CN**: 声明 TableGen def 记录 `VUZPH`。
- **L1805 EN**: Declares TableGen def record `VTRNH`.
  **L1805 CN**: 声明 TableGen def 记录 `VTRNH`。
- **L1806 EN**: Comment explains nearby logic, constraints, or intent: `Vector Extract`.
  **L1806 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Vector Extract`。
- **L1807 EN**: Declares TableGen def record `VEXTH`.
  **L1807 CN**: 声明 TableGen def 记录 `VEXTH`。
- **L1808 EN**: Comment explains nearby logic, constraints, or intent: `Reverse vector elements`.
  **L1808 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Reverse vector elements`。
- **L1809 EN**: Declares TableGen def record `VREV64H`.
  **L1809 CN**: 声明 TableGen def 记录 `VREV64H`。
- **L1810 EN**: Blank line separating nearby declarations or logic blocks.
  **L1810 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1811 EN**: Comment explains nearby logic, constraints, or intent: `ARMv8.2-A FP16 vector intrinsics for A64 only.`.
  **L1811 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ARMv8.2-A FP16 vector intrinsics for A64 only.`。
- **L1812 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "fullfp16,neon" in {`.
  **L1812 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "fullfp16,neon" in {`。
- **L1813 EN**: Blank line separating nearby declarations or logic blocks.
  **L1813 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1814 EN**: Comment explains nearby logic, constraints, or intent: `Vector rounding`.
  **L1814 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Vector rounding`。
- **L1815 EN**: Declares TableGen def record `FRINTIH`.
  **L1815 CN**: 声明 TableGen def 记录 `FRINTIH`。
- **L1816 EN**: Blank line separating nearby declarations or logic blocks.
  **L1816 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1817 EN**: Comment explains nearby logic, constraints, or intent: `Misc.`.
  **L1817 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Misc.`。
- **L1818 EN**: Declares TableGen def record `FSQRTH`.
  **L1818 CN**: 声明 TableGen def 记录 `FSQRTH`。
- **L1819 EN**: Blank line separating nearby declarations or logic blocks.
  **L1819 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1820 EN**: Comment explains nearby logic, constraints, or intent: `Multiplication/Division`.
  **L1820 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Multiplication/Division`。

### Lines 1821-1848

````tablegen
  def MULXH         : SInst<"vmulx", "...", "hQh">;
  def FDIVH         : IOpInst<"vdiv", "...",  "hQh", OP_DIV>;

  // Pairwise addition
  def VPADDH1       : SInst<"vpadd", "...", "Qh">;

  // Pairwise Max/Min
  def VPMAXH1       : SInst<"vpmax", "...", "Qh">;
  def VPMINH1       : SInst<"vpmin", "...", "Qh">;

  // Pairwise MaxNum/MinNum
  def FMAXNMPH      : SInst<"vpmaxnm", "...", "hQh">;
  def FMINNMPH      : SInst<"vpminnm", "...", "hQh">;

  // ARMv8.2-A FP16 lane vector intrinsics.

  // FMA lane
  def VFMA_LANEH   : IInst<"vfma_lane", "...qI", "hQh",
                          [ImmCheck<3, ImmCheckLaneIndex, 2>]>;
  def VFMA_LANEQH  : IInst<"vfma_laneq", "...QI", "hQh",
                          [ImmCheck<3, ImmCheckLaneIndex, 2>]>;

  // FMA lane with scalar argument
  def FMLA_NH      : SOpInst<"vfma_n", "...1", "hQh", OP_FMLA_N>;
  // Scalar floating point fused multiply-add (scalar, by element)
  def SCALAR_FMLA_LANEH  : IInst<"vfma_lane", "111.I", "Sh",
                                [ImmCheck<3, ImmCheckLaneIndex, 2>]>;
  def SCALAR_FMLA_LANEQH : IInst<"vfma_laneq", "111QI", "Sh",
````
- **L1821 EN**: Declares TableGen def record `MULXH`.
  **L1821 CN**: 声明 TableGen def 记录 `MULXH`。
- **L1822 EN**: Declares TableGen def record `FDIVH`.
  **L1822 CN**: 声明 TableGen def 记录 `FDIVH`。
- **L1823 EN**: Blank line separating nearby declarations or logic blocks.
  **L1823 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1824 EN**: Comment explains nearby logic, constraints, or intent: `Pairwise addition`.
  **L1824 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Pairwise addition`。
- **L1825 EN**: Declares TableGen def record `VPADDH1`.
  **L1825 CN**: 声明 TableGen def 记录 `VPADDH1`。
- **L1826 EN**: Blank line separating nearby declarations or logic blocks.
  **L1826 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1827 EN**: Comment explains nearby logic, constraints, or intent: `Pairwise Max/Min`.
  **L1827 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Pairwise Max/Min`。
- **L1828 EN**: Declares TableGen def record `VPMAXH1`.
  **L1828 CN**: 声明 TableGen def 记录 `VPMAXH1`。
- **L1829 EN**: Declares TableGen def record `VPMINH1`.
  **L1829 CN**: 声明 TableGen def 记录 `VPMINH1`。
- **L1830 EN**: Blank line separating nearby declarations or logic blocks.
  **L1830 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1831 EN**: Comment explains nearby logic, constraints, or intent: `Pairwise MaxNum/MinNum`.
  **L1831 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Pairwise MaxNum/MinNum`。
- **L1832 EN**: Declares TableGen def record `FMAXNMPH`.
  **L1832 CN**: 声明 TableGen def 记录 `FMAXNMPH`。
- **L1833 EN**: Declares TableGen def record `FMINNMPH`.
  **L1833 CN**: 声明 TableGen def 记录 `FMINNMPH`。
- **L1834 EN**: Blank line separating nearby declarations or logic blocks.
  **L1834 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1835 EN**: Comment explains nearby logic, constraints, or intent: `ARMv8.2-A FP16 lane vector intrinsics.`.
  **L1835 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ARMv8.2-A FP16 lane vector intrinsics.`。
- **L1836 EN**: Blank line separating nearby declarations or logic blocks.
  **L1836 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1837 EN**: Comment explains nearby logic, constraints, or intent: `FMA lane`.
  **L1837 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FMA lane`。
- **L1838 EN**: Declares TableGen def record `VFMA_LANEH`.
  **L1838 CN**: 声明 TableGen def 记录 `VFMA_LANEH`。
- **L1839 EN**: Adds a standalone statement or declaration: `[ImmCheck<3, ImmCheckLaneIndex, 2>]>;`.
  **L1839 CN**: 添加一条独立语句或声明：`[ImmCheck<3, ImmCheckLaneIndex, 2>]>;`。
- **L1840 EN**: Declares TableGen def record `VFMA_LANEQH`.
  **L1840 CN**: 声明 TableGen def 记录 `VFMA_LANEQH`。
- **L1841 EN**: Adds a standalone statement or declaration: `[ImmCheck<3, ImmCheckLaneIndex, 2>]>;`.
  **L1841 CN**: 添加一条独立语句或声明：`[ImmCheck<3, ImmCheckLaneIndex, 2>]>;`。
- **L1842 EN**: Blank line separating nearby declarations or logic blocks.
  **L1842 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1843 EN**: Comment explains nearby logic, constraints, or intent: `FMA lane with scalar argument`.
  **L1843 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FMA lane with scalar argument`。
- **L1844 EN**: Declares TableGen def record `FMLA_NH`.
  **L1844 CN**: 声明 TableGen def 记录 `FMLA_NH`。
- **L1845 EN**: Comment explains nearby logic, constraints, or intent: `Scalar floating point fused multiply-add (scalar, by element)`.
  **L1845 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar floating point fused multiply-add (scalar, by element)`。
- **L1846 EN**: Declares TableGen def record `SCALAR_FMLA_LANEH`.
  **L1846 CN**: 声明 TableGen def 记录 `SCALAR_FMLA_LANEH`。
- **L1847 EN**: Adds a standalone statement or declaration: `[ImmCheck<3, ImmCheckLaneIndex, 2>]>;`.
  **L1847 CN**: 添加一条独立语句或声明：`[ImmCheck<3, ImmCheckLaneIndex, 2>]>;`。
- **L1848 EN**: Declares TableGen def record `SCALAR_FMLA_LANEQH`.
  **L1848 CN**: 声明 TableGen def 记录 `SCALAR_FMLA_LANEQH`。

### Lines 1849-1876

````tablegen
                                [ImmCheck<3, ImmCheckLaneIndex, 2>]>;

  // FMS lane
  def VFMS_LANEH   : IOpInst<"vfms_lane", "...qI", "hQh", OP_FMS_LN>;
  def VFMS_LANEQH  : IOpInst<"vfms_laneq", "...QI", "hQh", OP_FMS_LNQ>;
  // FMS lane with scalar argument
  def FMLS_NH      : SOpInst<"vfms_n", "...1", "hQh", OP_FMLS_N>;
  // Scalar floating foint fused multiply-subtract (scalar, by element)
  def SCALAR_FMLS_LANEH  : IOpInst<"vfms_lane", "111.I", "Sh", OP_FMS_LN>;
  def SCALAR_FMLS_LANEQH : IOpInst<"vfms_laneq", "111QI", "Sh", OP_FMS_LNQ>;
  // Mul lane
  def VMUL_LANEQH   : IOpInst<"vmul_laneq", "..QI", "hQh", OP_MUL_LN>;
  // Scalar floating point  multiply (scalar, by element)
  def SCALAR_FMUL_LANEH  : IOpInst<"vmul_lane", "11.I", "Sh", OP_SCALAR_MUL_LN>;
  def SCALAR_FMUL_LANEQH : IOpInst<"vmul_laneq", "11QI", "Sh", OP_SCALAR_MUL_LN>;

  // Mulx lane
  def VMULX_LANEH   : IOpInst<"vmulx_lane", "..qI", "hQh", OP_MULX_LN>;
  def VMULX_LANEQH  : IOpInst<"vmulx_laneq", "..QI", "hQh", OP_MULX_LN>;
  def VMULX_NH      : IOpInst<"vmulx_n", "..1", "hQh", OP_MULX_N>;
  // Scalar floating point  mulx (scalar, by element)
  def SCALAR_FMULX_LANEH : IInst<"vmulx_lane", "11.I", "Sh",
                                [ImmCheck<2, ImmCheckLaneIndex, 1>]>;
  def SCALAR_FMULX_LANEQH : IInst<"vmulx_laneq", "11QI", "Sh",
                                [ImmCheck<2, ImmCheckLaneIndex, 1>]>;
  // ARMv8.2-A FP16 reduction vector intrinsics.
  def VMAXVH   : SInst<"vmaxv", "1.", "hQh">;
  def VMINVH   : SInst<"vminv", "1.", "hQh">;
````
- **L1849 EN**: Adds a standalone statement or declaration: `[ImmCheck<3, ImmCheckLaneIndex, 2>]>;`.
  **L1849 CN**: 添加一条独立语句或声明：`[ImmCheck<3, ImmCheckLaneIndex, 2>]>;`。
- **L1850 EN**: Blank line separating nearby declarations or logic blocks.
  **L1850 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1851 EN**: Comment explains nearby logic, constraints, or intent: `FMS lane`.
  **L1851 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FMS lane`。
- **L1852 EN**: Declares TableGen def record `VFMS_LANEH`.
  **L1852 CN**: 声明 TableGen def 记录 `VFMS_LANEH`。
- **L1853 EN**: Declares TableGen def record `VFMS_LANEQH`.
  **L1853 CN**: 声明 TableGen def 记录 `VFMS_LANEQH`。
- **L1854 EN**: Comment explains nearby logic, constraints, or intent: `FMS lane with scalar argument`.
  **L1854 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FMS lane with scalar argument`。
- **L1855 EN**: Declares TableGen def record `FMLS_NH`.
  **L1855 CN**: 声明 TableGen def 记录 `FMLS_NH`。
- **L1856 EN**: Comment explains nearby logic, constraints, or intent: `Scalar floating foint fused multiply-subtract (scalar, by element)`.
  **L1856 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar floating foint fused multiply-subtract (scalar, by element)`。
- **L1857 EN**: Declares TableGen def record `SCALAR_FMLS_LANEH`.
  **L1857 CN**: 声明 TableGen def 记录 `SCALAR_FMLS_LANEH`。
- **L1858 EN**: Declares TableGen def record `SCALAR_FMLS_LANEQH`.
  **L1858 CN**: 声明 TableGen def 记录 `SCALAR_FMLS_LANEQH`。
- **L1859 EN**: Comment explains nearby logic, constraints, or intent: `Mul lane`.
  **L1859 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Mul lane`。
- **L1860 EN**: Declares TableGen def record `VMUL_LANEQH`.
  **L1860 CN**: 声明 TableGen def 记录 `VMUL_LANEQH`。
- **L1861 EN**: Comment explains nearby logic, constraints, or intent: `Scalar floating point multiply (scalar, by element)`.
  **L1861 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar floating point multiply (scalar, by element)`。
- **L1862 EN**: Declares TableGen def record `SCALAR_FMUL_LANEH`.
  **L1862 CN**: 声明 TableGen def 记录 `SCALAR_FMUL_LANEH`。
- **L1863 EN**: Declares TableGen def record `SCALAR_FMUL_LANEQH`.
  **L1863 CN**: 声明 TableGen def 记录 `SCALAR_FMUL_LANEQH`。
- **L1864 EN**: Blank line separating nearby declarations or logic blocks.
  **L1864 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1865 EN**: Comment explains nearby logic, constraints, or intent: `Mulx lane`.
  **L1865 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Mulx lane`。
- **L1866 EN**: Declares TableGen def record `VMULX_LANEH`.
  **L1866 CN**: 声明 TableGen def 记录 `VMULX_LANEH`。
- **L1867 EN**: Declares TableGen def record `VMULX_LANEQH`.
  **L1867 CN**: 声明 TableGen def 记录 `VMULX_LANEQH`。
- **L1868 EN**: Declares TableGen def record `VMULX_NH`.
  **L1868 CN**: 声明 TableGen def 记录 `VMULX_NH`。
- **L1869 EN**: Comment explains nearby logic, constraints, or intent: `Scalar floating point mulx (scalar, by element)`.
  **L1869 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar floating point mulx (scalar, by element)`。
- **L1870 EN**: Declares TableGen def record `SCALAR_FMULX_LANEH`.
  **L1870 CN**: 声明 TableGen def 记录 `SCALAR_FMULX_LANEH`。
- **L1871 EN**: Adds a standalone statement or declaration: `[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`.
  **L1871 CN**: 添加一条独立语句或声明：`[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`。
- **L1872 EN**: Declares TableGen def record `SCALAR_FMULX_LANEQH`.
  **L1872 CN**: 声明 TableGen def 记录 `SCALAR_FMULX_LANEQH`。
- **L1873 EN**: Adds a standalone statement or declaration: `[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`.
  **L1873 CN**: 添加一条独立语句或声明：`[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`。
- **L1874 EN**: Comment explains nearby logic, constraints, or intent: `ARMv8.2-A FP16 reduction vector intrinsics.`.
  **L1874 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ARMv8.2-A FP16 reduction vector intrinsics.`。
- **L1875 EN**: Declares TableGen def record `VMAXVH`.
  **L1875 CN**: 声明 TableGen def 记录 `VMAXVH`。
- **L1876 EN**: Declares TableGen def record `VMINVH`.
  **L1876 CN**: 声明 TableGen def 记录 `VMINVH`。

### Lines 1877-1904

````tablegen
  def FMAXNMVH : SInst<"vmaxnmv", "1.", "hQh">;
  def FMINNMVH : SInst<"vminnmv", "1.", "hQh">;
}

let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)" in {
  // Permutation
  def VTRN1H     : SOpInst<"vtrn1", "...", "hQh", OP_TRN1>;
  def VZIP1H     : SOpInst<"vzip1", "...", "hQh", OP_ZIP1>;
  def VUZP1H     : SOpInst<"vuzp1", "...", "hQh", OP_UZP1>;
  def VTRN2H     : SOpInst<"vtrn2", "...", "hQh", OP_TRN2>;
  def VZIP2H     : SOpInst<"vzip2", "...", "hQh", OP_ZIP2>;
  def VUZP2H     : SOpInst<"vuzp2", "...", "hQh", OP_UZP2>;

  def SCALAR_VDUP_LANEH  : IInst<"vdup_lane", "1.I", "Sh",
                                [ImmCheck<1, ImmCheckLaneIndex, 0>]>;
  def SCALAR_VDUP_LANEQH : IInst<"vdup_laneq", "1QI", "Sh",
                                [ImmCheck<1, ImmCheckLaneIndex, 0>]>;
}

// v8.2-A dot product instructions.
let TargetGuard = "dotprod,neon" in {
  def DOT : SInst<"vdot", "..(<<)(<<)", "iQiUiQUi">;
  def DOT_LANE : SOpInst<"vdot_lane", "..(<<)(<<q)I", "iUiQiQUi", OP_DOT_LN>;
}
let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "dotprod,neon" in {
  // Variants indexing into a 128-bit vector are A64 only.
  def UDOT_LANEQ : SOpInst<"vdot_laneq", "..(<<)(<<Q)I", "iUiQiQUi", OP_DOT_LNQ>;
}
````
- **L1877 EN**: Declares TableGen def record `FMAXNMVH`.
  **L1877 CN**: 声明 TableGen def 记录 `FMAXNMVH`。
- **L1878 EN**: Declares TableGen def record `FMINNMVH`.
  **L1878 CN**: 声明 TableGen def 记录 `FMINNMVH`。
- **L1879 EN**: Closes the current lexical scope or compound statement.
  **L1879 CN**: 结束当前词法作用域或复合语句块。
- **L1880 EN**: Blank line separating nearby declarations or logic blocks.
  **L1880 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1881 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)" in {`.
  **L1881 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)" in {`。
- **L1882 EN**: Comment explains nearby logic, constraints, or intent: `Permutation`.
  **L1882 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Permutation`。
- **L1883 EN**: Declares TableGen def record `VTRN1H`.
  **L1883 CN**: 声明 TableGen def 记录 `VTRN1H`。
- **L1884 EN**: Declares TableGen def record `VZIP1H`.
  **L1884 CN**: 声明 TableGen def 记录 `VZIP1H`。
- **L1885 EN**: Declares TableGen def record `VUZP1H`.
  **L1885 CN**: 声明 TableGen def 记录 `VUZP1H`。
- **L1886 EN**: Declares TableGen def record `VTRN2H`.
  **L1886 CN**: 声明 TableGen def 记录 `VTRN2H`。
- **L1887 EN**: Declares TableGen def record `VZIP2H`.
  **L1887 CN**: 声明 TableGen def 记录 `VZIP2H`。
- **L1888 EN**: Declares TableGen def record `VUZP2H`.
  **L1888 CN**: 声明 TableGen def 记录 `VUZP2H`。
- **L1889 EN**: Blank line separating nearby declarations or logic blocks.
  **L1889 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1890 EN**: Declares TableGen def record `SCALAR_VDUP_LANEH`.
  **L1890 CN**: 声明 TableGen def 记录 `SCALAR_VDUP_LANEH`。
- **L1891 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheckLaneIndex, 0>]>;`.
  **L1891 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheckLaneIndex, 0>]>;`。
- **L1892 EN**: Declares TableGen def record `SCALAR_VDUP_LANEQH`.
  **L1892 CN**: 声明 TableGen def 记录 `SCALAR_VDUP_LANEQH`。
- **L1893 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheckLaneIndex, 0>]>;`.
  **L1893 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheckLaneIndex, 0>]>;`。
- **L1894 EN**: Closes the current lexical scope or compound statement.
  **L1894 CN**: 结束当前词法作用域或复合语句块。
- **L1895 EN**: Blank line separating nearby declarations or logic blocks.
  **L1895 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1896 EN**: Comment explains nearby logic, constraints, or intent: `v8.2-A dot product instructions.`.
  **L1896 CN**: 注释解释附近代码的逻辑、约束或设计意图：`v8.2-A dot product instructions.`。
- **L1897 EN**: Assigns a TableGen property that affects following records or inherited fields: `let TargetGuard = "dotprod,neon" in {`.
  **L1897 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let TargetGuard = "dotprod,neon" in {`。
- **L1898 EN**: Declares TableGen def record `DOT`.
  **L1898 CN**: 声明 TableGen def 记录 `DOT`。
- **L1899 EN**: Declares TableGen def record `DOT_LANE`.
  **L1899 CN**: 声明 TableGen def 记录 `DOT_LANE`。
- **L1900 EN**: Closes the current lexical scope or compound statement.
  **L1900 CN**: 结束当前词法作用域或复合语句块。
- **L1901 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "dotprod,neon" in {`.
  **L1901 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "dotprod,neon" in {`。
- **L1902 EN**: Comment explains nearby logic, constraints, or intent: `Variants indexing into a 128-bit vector are A64 only.`.
  **L1902 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Variants indexing into a 128-bit vector are A64 only.`。
- **L1903 EN**: Declares TableGen def record `UDOT_LANEQ`.
  **L1903 CN**: 声明 TableGen def 记录 `UDOT_LANEQ`。
- **L1904 EN**: Closes the current lexical scope or compound statement.
  **L1904 CN**: 结束当前词法作用域或复合语句块。

### Lines 1905-1932

````tablegen

// v8.2-A FP16 fused multiply-add long instructions.
let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "fp16fml,neon" in {
  def VFMLAL_LOW  : SInst<"vfmlal_low",  ">>..", "hQh">;
  def VFMLSL_LOW  : SInst<"vfmlsl_low",  ">>..", "hQh">;
  def VFMLAL_HIGH : SInst<"vfmlal_high", ">>..", "hQh">;
  def VFMLSL_HIGH : SInst<"vfmlsl_high", ">>..", "hQh">;

  def VFMLAL_LANE_LOW  : SOpInst<"vfmlal_lane_low",  "(F>)(F>)F(Fq)I", "hQh", OP_FMLAL_LN>;
  def VFMLSL_LANE_LOW  : SOpInst<"vfmlsl_lane_low",  "(F>)(F>)F(Fq)I", "hQh", OP_FMLSL_LN>;
  def VFMLAL_LANE_HIGH : SOpInst<"vfmlal_lane_high", "(F>)(F>)F(Fq)I", "hQh", OP_FMLAL_LN_Hi>;
  def VFMLSL_LANE_HIGH : SOpInst<"vfmlsl_lane_high", "(F>)(F>)F(Fq)I", "hQh", OP_FMLSL_LN_Hi>;

  def VFMLAL_LANEQ_LOW  : SOpInst<"vfmlal_laneq_low",  "(F>)(F>)F(FQ)I", "hQh", OP_FMLAL_LN>;
  def VFMLSL_LANEQ_LOW  : SOpInst<"vfmlsl_laneq_low",  "(F>)(F>)F(FQ)I", "hQh", OP_FMLSL_LN>;
  def VFMLAL_LANEQ_HIGH : SOpInst<"vfmlal_laneq_high", "(F>)(F>)F(FQ)I", "hQh", OP_FMLAL_LN_Hi>;
  def VFMLSL_LANEQ_HIGH : SOpInst<"vfmlsl_laneq_high", "(F>)(F>)F(FQ)I", "hQh", OP_FMLSL_LN_Hi>;
}

let ArchGuard = "defined(__aarch64__)", TargetGuard = "f8f16mm,neon" in {
  def VMMLA_F16_MF8 : VInst<"vmmla_f16_mf8_fpm", "(>F)(>F)..V", "Qm">;
}

let ArchGuard = "defined(__aarch64__)", TargetGuard = "f8f32mm,neon" in {
  def VMMLA_F32_MF8 : VInst<"vmmla_f32_mf8_fpm", "(>>F)(>>F)..V", "Qm">;
}

let ArchGuard = "defined(__aarch64__)", TargetGuard = "f16mm,neon" in {
````
- **L1905 EN**: Blank line separating nearby declarations or logic blocks.
  **L1905 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1906 EN**: Comment explains nearby logic, constraints, or intent: `v8.2-A FP16 fused multiply-add long instructions.`.
  **L1906 CN**: 注释解释附近代码的逻辑、约束或设计意图：`v8.2-A FP16 fused multiply-add long instructions.`。
- **L1907 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "fp16fml,neon" in {`.
  **L1907 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "fp16fml,neon" in {`。
- **L1908 EN**: Declares TableGen def record `VFMLAL_LOW`.
  **L1908 CN**: 声明 TableGen def 记录 `VFMLAL_LOW`。
- **L1909 EN**: Declares TableGen def record `VFMLSL_LOW`.
  **L1909 CN**: 声明 TableGen def 记录 `VFMLSL_LOW`。
- **L1910 EN**: Declares TableGen def record `VFMLAL_HIGH`.
  **L1910 CN**: 声明 TableGen def 记录 `VFMLAL_HIGH`。
- **L1911 EN**: Declares TableGen def record `VFMLSL_HIGH`.
  **L1911 CN**: 声明 TableGen def 记录 `VFMLSL_HIGH`。
- **L1912 EN**: Blank line separating nearby declarations or logic blocks.
  **L1912 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1913 EN**: Declares TableGen def record `VFMLAL_LANE_LOW`.
  **L1913 CN**: 声明 TableGen def 记录 `VFMLAL_LANE_LOW`。
- **L1914 EN**: Declares TableGen def record `VFMLSL_LANE_LOW`.
  **L1914 CN**: 声明 TableGen def 记录 `VFMLSL_LANE_LOW`。
- **L1915 EN**: Declares TableGen def record `VFMLAL_LANE_HIGH`.
  **L1915 CN**: 声明 TableGen def 记录 `VFMLAL_LANE_HIGH`。
- **L1916 EN**: Declares TableGen def record `VFMLSL_LANE_HIGH`.
  **L1916 CN**: 声明 TableGen def 记录 `VFMLSL_LANE_HIGH`。
- **L1917 EN**: Blank line separating nearby declarations or logic blocks.
  **L1917 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1918 EN**: Declares TableGen def record `VFMLAL_LANEQ_LOW`.
  **L1918 CN**: 声明 TableGen def 记录 `VFMLAL_LANEQ_LOW`。
- **L1919 EN**: Declares TableGen def record `VFMLSL_LANEQ_LOW`.
  **L1919 CN**: 声明 TableGen def 记录 `VFMLSL_LANEQ_LOW`。
- **L1920 EN**: Declares TableGen def record `VFMLAL_LANEQ_HIGH`.
  **L1920 CN**: 声明 TableGen def 记录 `VFMLAL_LANEQ_HIGH`。
- **L1921 EN**: Declares TableGen def record `VFMLSL_LANEQ_HIGH`.
  **L1921 CN**: 声明 TableGen def 记录 `VFMLSL_LANEQ_HIGH`。
- **L1922 EN**: Closes the current lexical scope or compound statement.
  **L1922 CN**: 结束当前词法作用域或复合语句块。
- **L1923 EN**: Blank line separating nearby declarations or logic blocks.
  **L1923 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1924 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "defined(__aarch64__)", TargetGuard = "f8f16mm,neon" in {`.
  **L1924 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "defined(__aarch64__)", TargetGuard = "f8f16mm,neon" in {`。
- **L1925 EN**: Declares TableGen def record `VMMLA_F16_MF8`.
  **L1925 CN**: 声明 TableGen def 记录 `VMMLA_F16_MF8`。
- **L1926 EN**: Closes the current lexical scope or compound statement.
  **L1926 CN**: 结束当前词法作用域或复合语句块。
- **L1927 EN**: Blank line separating nearby declarations or logic blocks.
  **L1927 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1928 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "defined(__aarch64__)", TargetGuard = "f8f32mm,neon" in {`.
  **L1928 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "defined(__aarch64__)", TargetGuard = "f8f32mm,neon" in {`。
- **L1929 EN**: Declares TableGen def record `VMMLA_F32_MF8`.
  **L1929 CN**: 声明 TableGen def 记录 `VMMLA_F32_MF8`。
- **L1930 EN**: Closes the current lexical scope or compound statement.
  **L1930 CN**: 结束当前词法作用域或复合语句块。
- **L1931 EN**: Blank line separating nearby declarations or logic blocks.
  **L1931 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1932 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "defined(__aarch64__)", TargetGuard = "f16mm,neon" in {`.
  **L1932 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "defined(__aarch64__)", TargetGuard = "f16mm,neon" in {`。

### Lines 1933-1960

````tablegen
  def VMMLA_F16_F16 : SInst<"vmmla_f16", "....", "Qh">;
}

let ArchGuard = "defined(__aarch64__)", TargetGuard = "f16f32mm,neon" in {
  def VMMLA_F32_F16 : SInst<"vmmla_f32", ">>..", "Qh">;
}

let TargetGuard = "i8mm,neon" in {
  def VMMLA   : SInst<"vmmla", "..(<<)(<<)", "QUiQi">;
  def VUSMMLA : SInst<"vusmmla", "..(<<U)(<<)", "Qi">;

  def VUSDOT  : SInst<"vusdot", "..(<<U)(<<)", "iQi">;

  def VUSDOT_LANE  : SOpInst<"vusdot_lane", "..(<<U)(<<q)I", "iQi", OP_USDOT_LN>;
  def VSUDOT_LANE  : SOpInst<"vsudot_lane", "..(<<)(<<qU)I", "iQi", OP_SUDOT_LN>;

  let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)" in {
    def VUSDOT_LANEQ  : SOpInst<"vusdot_laneq", "..(<<U)(<<Q)I", "iQi", OP_USDOT_LNQ>;
    def VSUDOT_LANEQ  : SOpInst<"vsudot_laneq", "..(<<)(<<QU)I", "iQi", OP_SUDOT_LNQ>;
  }
}

let TargetGuard = "bf16,neon" in {
  def VDOT_BF : SInst<"vbfdot", "..BB", "fQf">;
  def VDOT_LANE_BF : SOpInst<"vbfdot_lane", "..B(Bq)I", "fQf", OP_BFDOT_LN>;
  def VDOT_LANEQ_BF : SOpInst<"vbfdot_laneq", "..B(BQ)I", "fQf", OP_BFDOT_LNQ>;

  def VFMMLA_BF : SInst<"vbfmmla", "..BB", "Qf">;
````
- **L1933 EN**: Declares TableGen def record `VMMLA_F16_F16`.
  **L1933 CN**: 声明 TableGen def 记录 `VMMLA_F16_F16`。
- **L1934 EN**: Closes the current lexical scope or compound statement.
  **L1934 CN**: 结束当前词法作用域或复合语句块。
- **L1935 EN**: Blank line separating nearby declarations or logic blocks.
  **L1935 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1936 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "defined(__aarch64__)", TargetGuard = "f16f32mm,neon" in {`.
  **L1936 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "defined(__aarch64__)", TargetGuard = "f16f32mm,neon" in {`。
- **L1937 EN**: Declares TableGen def record `VMMLA_F32_F16`.
  **L1937 CN**: 声明 TableGen def 记录 `VMMLA_F32_F16`。
- **L1938 EN**: Closes the current lexical scope or compound statement.
  **L1938 CN**: 结束当前词法作用域或复合语句块。
- **L1939 EN**: Blank line separating nearby declarations or logic blocks.
  **L1939 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1940 EN**: Assigns a TableGen property that affects following records or inherited fields: `let TargetGuard = "i8mm,neon" in {`.
  **L1940 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let TargetGuard = "i8mm,neon" in {`。
- **L1941 EN**: Declares TableGen def record `VMMLA`.
  **L1941 CN**: 声明 TableGen def 记录 `VMMLA`。
- **L1942 EN**: Declares TableGen def record `VUSMMLA`.
  **L1942 CN**: 声明 TableGen def 记录 `VUSMMLA`。
- **L1943 EN**: Blank line separating nearby declarations or logic blocks.
  **L1943 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1944 EN**: Declares TableGen def record `VUSDOT`.
  **L1944 CN**: 声明 TableGen def 记录 `VUSDOT`。
- **L1945 EN**: Blank line separating nearby declarations or logic blocks.
  **L1945 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1946 EN**: Declares TableGen def record `VUSDOT_LANE`.
  **L1946 CN**: 声明 TableGen def 记录 `VUSDOT_LANE`。
- **L1947 EN**: Declares TableGen def record `VSUDOT_LANE`.
  **L1947 CN**: 声明 TableGen def 记录 `VSUDOT_LANE`。
- **L1948 EN**: Blank line separating nearby declarations or logic blocks.
  **L1948 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1949 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)" in {`.
  **L1949 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)" in {`。
- **L1950 EN**: Declares TableGen def record `VUSDOT_LANEQ`.
  **L1950 CN**: 声明 TableGen def 记录 `VUSDOT_LANEQ`。
- **L1951 EN**: Declares TableGen def record `VSUDOT_LANEQ`.
  **L1951 CN**: 声明 TableGen def 记录 `VSUDOT_LANEQ`。
- **L1952 EN**: Closes the current lexical scope or compound statement.
  **L1952 CN**: 结束当前词法作用域或复合语句块。
- **L1953 EN**: Closes the current lexical scope or compound statement.
  **L1953 CN**: 结束当前词法作用域或复合语句块。
- **L1954 EN**: Blank line separating nearby declarations or logic blocks.
  **L1954 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1955 EN**: Assigns a TableGen property that affects following records or inherited fields: `let TargetGuard = "bf16,neon" in {`.
  **L1955 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let TargetGuard = "bf16,neon" in {`。
- **L1956 EN**: Declares TableGen def record `VDOT_BF`.
  **L1956 CN**: 声明 TableGen def 记录 `VDOT_BF`。
- **L1957 EN**: Declares TableGen def record `VDOT_LANE_BF`.
  **L1957 CN**: 声明 TableGen def 记录 `VDOT_LANE_BF`。
- **L1958 EN**: Declares TableGen def record `VDOT_LANEQ_BF`.
  **L1958 CN**: 声明 TableGen def 记录 `VDOT_LANEQ_BF`。
- **L1959 EN**: Blank line separating nearby declarations or logic blocks.
  **L1959 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1960 EN**: Declares TableGen def record `VFMMLA_BF`.
  **L1960 CN**: 声明 TableGen def 记录 `VFMMLA_BF`。

### Lines 1961-1988

````tablegen

  def VFMLALB_BF : SInst<"vbfmlalb", "..BB", "Qf">;
  def VFMLALT_BF : SInst<"vbfmlalt", "..BB", "Qf">;

  def VFMLALB_LANE_BF : SOpInst<"vbfmlalb_lane", "..B(Bq)I", "Qf", OP_BFMLALB_LN>;
  def VFMLALB_LANEQ_BF : SOpInst<"vbfmlalb_laneq", "..B(BQ)I", "Qf", OP_BFMLALB_LN>;

  def VFMLALT_LANE_BF : SOpInst<"vbfmlalt_lane", "..B(Bq)I", "Qf", OP_BFMLALT_LN>;
  def VFMLALT_LANEQ_BF : SOpInst<"vbfmlalt_laneq", "..B(BQ)I", "Qf", OP_BFMLALT_LN>;
}

multiclass VCMLA_ROTS<string type, string lanety, string laneqty> {
  foreach ROT = ["", "_rot90", "_rot180", "_rot270" ] in {
    def   : SInst<"vcmla" # ROT, "....", type # "Q" # type>;

    // vcmla{ROT}_lane
    def : SOpInst<"vcmla" # ROT # "_lane", "...qI", type, Op<(call "vcmla" # ROT, $p0, $p1,
           (bitcast $p0, (dup_typed lanety , (call "vget_lane", (bitcast lanety, $p2), $p3))))>>;
    // vcmlaq{ROT}_lane
    def : SOpInst<"vcmla" # ROT # "_lane", "...qI", "Q" # type, Op<(call "vcmla" # ROT, $p0, $p1,
           (bitcast $p0, (dup_typed laneqty , (call "vget_lane", (bitcast lanety, $p2), $p3))))>>;

    // vcmla{ROT}_laneq
    def : SOpInst<"vcmla" # ROT # "_laneq", "...QI", type,  Op<(call "vcmla" # ROT, $p0, $p1,
            (bitcast $p0, (dup_typed lanety, (call "vget_lane", (bitcast laneqty, $p2), $p3))))>>;
    // vcmlaq{ROT}_laneq
    def : SOpInst<"vcmla" # ROT # "_laneq", "...QI", "Q" # type, Op<(call "vcmla" # ROT, $p0, $p1,
            (bitcast $p0, (dup_typed laneqty , (call "vget_lane", (bitcast laneqty, $p2), $p3))))>>;
````
- **L1961 EN**: Blank line separating nearby declarations or logic blocks.
  **L1961 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1962 EN**: Declares TableGen def record `VFMLALB_BF`.
  **L1962 CN**: 声明 TableGen def 记录 `VFMLALB_BF`。
- **L1963 EN**: Declares TableGen def record `VFMLALT_BF`.
  **L1963 CN**: 声明 TableGen def 记录 `VFMLALT_BF`。
- **L1964 EN**: Blank line separating nearby declarations or logic blocks.
  **L1964 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1965 EN**: Declares TableGen def record `VFMLALB_LANE_BF`.
  **L1965 CN**: 声明 TableGen def 记录 `VFMLALB_LANE_BF`。
- **L1966 EN**: Declares TableGen def record `VFMLALB_LANEQ_BF`.
  **L1966 CN**: 声明 TableGen def 记录 `VFMLALB_LANEQ_BF`。
- **L1967 EN**: Blank line separating nearby declarations or logic blocks.
  **L1967 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1968 EN**: Declares TableGen def record `VFMLALT_LANE_BF`.
  **L1968 CN**: 声明 TableGen def 记录 `VFMLALT_LANE_BF`。
- **L1969 EN**: Declares TableGen def record `VFMLALT_LANEQ_BF`.
  **L1969 CN**: 声明 TableGen def 记录 `VFMLALT_LANEQ_BF`。
- **L1970 EN**: Closes the current lexical scope or compound statement.
  **L1970 CN**: 结束当前词法作用域或复合语句块。
- **L1971 EN**: Blank line separating nearby declarations or logic blocks.
  **L1971 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1972 EN**: Declares TableGen multiclass record `VCMLA_ROTS`.
  **L1972 CN**: 声明 TableGen multiclass 记录 `VCMLA_ROTS`。
- **L1973 EN**: Starts a TableGen iteration used to generate repeated records: `foreach ROT = ["", "_rot90", "_rot180", "_rot270" ] in {`.
  **L1973 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach ROT = ["", "_rot90", "_rot180", "_rot270" ] in {`。
- **L1974 EN**: Declares TableGen def record `def`.
  **L1974 CN**: 声明 TableGen def 记录 `def`。
- **L1975 EN**: Blank line separating nearby declarations or logic blocks.
  **L1975 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1976 EN**: Comment explains nearby logic, constraints, or intent: `vcmla{ROT}_lane`.
  **L1976 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vcmla{ROT}_lane`。
- **L1977 EN**: Declares TableGen def record `def`.
  **L1977 CN**: 声明 TableGen def 记录 `def`。
- **L1978 EN**: Executes a call or declaration centered on `statement`.
  **L1978 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1979 EN**: Comment explains nearby logic, constraints, or intent: `vcmlaq{ROT}_lane`.
  **L1979 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vcmlaq{ROT}_lane`。
- **L1980 EN**: Declares TableGen def record `def`.
  **L1980 CN**: 声明 TableGen def 记录 `def`。
- **L1981 EN**: Executes a call or declaration centered on `statement`.
  **L1981 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1982 EN**: Blank line separating nearby declarations or logic blocks.
  **L1982 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1983 EN**: Comment explains nearby logic, constraints, or intent: `vcmla{ROT}_laneq`.
  **L1983 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vcmla{ROT}_laneq`。
- **L1984 EN**: Declares TableGen def record `def`.
  **L1984 CN**: 声明 TableGen def 记录 `def`。
- **L1985 EN**: Executes a call or declaration centered on `statement`.
  **L1985 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1986 EN**: Comment explains nearby logic, constraints, or intent: `vcmlaq{ROT}_laneq`.
  **L1986 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vcmlaq{ROT}_laneq`。
- **L1987 EN**: Declares TableGen def record `def`.
  **L1987 CN**: 声明 TableGen def 记录 `def`。
- **L1988 EN**: Executes a call or declaration centered on `statement`.
  **L1988 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 1989-2016

````tablegen
  }
}

// v8.3-A Vector complex addition intrinsics
let TargetGuard = "v8.3a,fullfp16,neon" in {
  def VCADD_ROT90_FP16   : SInst<"vcadd_rot90", "...", "h">;
  def VCADD_ROT270_FP16  : SInst<"vcadd_rot270", "...", "h">;
  def VCADDQ_ROT90_FP16  : SInst<"vcaddq_rot90", "QQQ", "h">;
  def VCADDQ_ROT270_FP16 : SInst<"vcaddq_rot270", "QQQ", "h">;

  defm VCMLA_FP16  : VCMLA_ROTS<"h", "uint32x2_t", "uint32x4_t">;
}
let TargetGuard = "v8.3a,neon" in {
  def VCADD_ROT90   : SInst<"vcadd_rot90", "...", "f">;
  def VCADD_ROT270  : SInst<"vcadd_rot270", "...", "f">;
  def VCADDQ_ROT90  : SInst<"vcaddq_rot90", "QQQ", "f">;
  def VCADDQ_ROT270 : SInst<"vcaddq_rot270", "QQQ", "f">;

  defm VCMLA_F32    : VCMLA_ROTS<"f", "uint64x1_t", "uint64x2_t">;
}
let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "v8.3a,neon" in {
  def VCADDQ_ROT90_FP64  : SInst<"vcaddq_rot90", "QQQ", "d">;
  def VCADDQ_ROT270_FP64 : SInst<"vcaddq_rot270", "QQQ", "d">;

  def VCMLAQ_FP64        : SInst<"vcmlaq", "QQQQ", "d">;
  def VCMLAQ_ROT90_FP64  : SInst<"vcmlaq_rot90", "QQQQ", "d">;
  def VCMLAQ_ROT180_FP64 : SInst<"vcmlaq_rot180", "QQQQ", "d">;
  def VCMLAQ_ROT270_FP64 : SInst<"vcmlaq_rot270", "QQQQ", "d">;
````
- **L1989 EN**: Closes the current lexical scope or compound statement.
  **L1989 CN**: 结束当前词法作用域或复合语句块。
- **L1990 EN**: Closes the current lexical scope or compound statement.
  **L1990 CN**: 结束当前词法作用域或复合语句块。
- **L1991 EN**: Blank line separating nearby declarations or logic blocks.
  **L1991 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1992 EN**: Comment explains nearby logic, constraints, or intent: `v8.3-A Vector complex addition intrinsics`.
  **L1992 CN**: 注释解释附近代码的逻辑、约束或设计意图：`v8.3-A Vector complex addition intrinsics`。
- **L1993 EN**: Assigns a TableGen property that affects following records or inherited fields: `let TargetGuard = "v8.3a,fullfp16,neon" in {`.
  **L1993 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let TargetGuard = "v8.3a,fullfp16,neon" in {`。
- **L1994 EN**: Declares TableGen def record `VCADD_ROT90_FP16`.
  **L1994 CN**: 声明 TableGen def 记录 `VCADD_ROT90_FP16`。
- **L1995 EN**: Declares TableGen def record `VCADD_ROT270_FP16`.
  **L1995 CN**: 声明 TableGen def 记录 `VCADD_ROT270_FP16`。
- **L1996 EN**: Declares TableGen def record `VCADDQ_ROT90_FP16`.
  **L1996 CN**: 声明 TableGen def 记录 `VCADDQ_ROT90_FP16`。
- **L1997 EN**: Declares TableGen def record `VCADDQ_ROT270_FP16`.
  **L1997 CN**: 声明 TableGen def 记录 `VCADDQ_ROT270_FP16`。
- **L1998 EN**: Blank line separating nearby declarations or logic blocks.
  **L1998 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1999 EN**: Declares TableGen defm record `VCMLA_FP16`.
  **L1999 CN**: 声明 TableGen defm 记录 `VCMLA_FP16`。
- **L2000 EN**: Closes the current lexical scope or compound statement.
  **L2000 CN**: 结束当前词法作用域或复合语句块。
- **L2001 EN**: Assigns a TableGen property that affects following records or inherited fields: `let TargetGuard = "v8.3a,neon" in {`.
  **L2001 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let TargetGuard = "v8.3a,neon" in {`。
- **L2002 EN**: Declares TableGen def record `VCADD_ROT90`.
  **L2002 CN**: 声明 TableGen def 记录 `VCADD_ROT90`。
- **L2003 EN**: Declares TableGen def record `VCADD_ROT270`.
  **L2003 CN**: 声明 TableGen def 记录 `VCADD_ROT270`。
- **L2004 EN**: Declares TableGen def record `VCADDQ_ROT90`.
  **L2004 CN**: 声明 TableGen def 记录 `VCADDQ_ROT90`。
- **L2005 EN**: Declares TableGen def record `VCADDQ_ROT270`.
  **L2005 CN**: 声明 TableGen def 记录 `VCADDQ_ROT270`。
- **L2006 EN**: Blank line separating nearby declarations or logic blocks.
  **L2006 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2007 EN**: Declares TableGen defm record `VCMLA_F32`.
  **L2007 CN**: 声明 TableGen defm 记录 `VCMLA_F32`。
- **L2008 EN**: Closes the current lexical scope or compound statement.
  **L2008 CN**: 结束当前词法作用域或复合语句块。
- **L2009 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "v8.3a,neon" in {`.
  **L2009 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "v8.3a,neon" in {`。
- **L2010 EN**: Declares TableGen def record `VCADDQ_ROT90_FP64`.
  **L2010 CN**: 声明 TableGen def 记录 `VCADDQ_ROT90_FP64`。
- **L2011 EN**: Declares TableGen def record `VCADDQ_ROT270_FP64`.
  **L2011 CN**: 声明 TableGen def 记录 `VCADDQ_ROT270_FP64`。
- **L2012 EN**: Blank line separating nearby declarations or logic blocks.
  **L2012 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2013 EN**: Declares TableGen def record `VCMLAQ_FP64`.
  **L2013 CN**: 声明 TableGen def 记录 `VCMLAQ_FP64`。
- **L2014 EN**: Declares TableGen def record `VCMLAQ_ROT90_FP64`.
  **L2014 CN**: 声明 TableGen def 记录 `VCMLAQ_ROT90_FP64`。
- **L2015 EN**: Declares TableGen def record `VCMLAQ_ROT180_FP64`.
  **L2015 CN**: 声明 TableGen def 记录 `VCMLAQ_ROT180_FP64`。
- **L2016 EN**: Declares TableGen def record `VCMLAQ_ROT270_FP64`.
  **L2016 CN**: 声明 TableGen def 记录 `VCMLAQ_ROT270_FP64`。

### Lines 2017-2044

````tablegen
}

// V8.2-A BFloat intrinsics
let TargetGuard = "bf16,neon" in {
  def VCREATE_BF : NoTestOpInst<"vcreate", ".(IU>)", "b", OP_CAST> {
    let BigEndianSafe = 1;
  }

  def VDUP_N_BF    : WOpInst<"vdup_n", ".1", "bQb", OP_DUP>;

  def VDUP_LANE_BF : WOpInst<"vdup_lane", ".qI", "bQb", OP_DUP_LN>;
  def VDUP_LANEQ_BF: WOpInst<"vdup_laneq", ".QI", "bQb", OP_DUP_LN>;

  def VCOMBINE_BF  : NoTestOpInst<"vcombine", "Q..", "b", OP_CONC>;

  def VGET_HIGH_BF : NoTestOpInst<"vget_high", ".Q", "b", OP_HI>;
  def VGET_LOW_BF  : NoTestOpInst<"vget_low", ".Q", "b", OP_LO>;

  def VGET_LANE_BF : IInst<"vget_lane", "1.I", "bQb",
                          [ImmCheck<1, ImmCheckLaneIndex, 0>]>;
  def VSET_LANE_BF : IInst<"vset_lane", ".1.I", "bQb",
                          [ImmCheck<2, ImmCheckLaneIndex, 1>]>;
  def SCALAR_VDUP_LANE_BF : IInst<"vdup_lane", "1.I", "Sb",
                          [ImmCheck<1, ImmCheckLaneIndex, 0>]>;
  def SCALAR_VDUP_LANEQ_BF : IInst<"vdup_laneq", "1QI", "Sb",
                          [ImmCheck<1, ImmCheckLaneIndex, 0>]>;

  def VLD1_BF : WInst<"vld1", ".(c*!)", "bQb">;
````
- **L2017 EN**: Closes the current lexical scope or compound statement.
  **L2017 CN**: 结束当前词法作用域或复合语句块。
- **L2018 EN**: Blank line separating nearby declarations or logic blocks.
  **L2018 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2019 EN**: Comment explains nearby logic, constraints, or intent: `V8.2-A BFloat intrinsics`.
  **L2019 CN**: 注释解释附近代码的逻辑、约束或设计意图：`V8.2-A BFloat intrinsics`。
- **L2020 EN**: Assigns a TableGen property that affects following records or inherited fields: `let TargetGuard = "bf16,neon" in {`.
  **L2020 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let TargetGuard = "bf16,neon" in {`。
- **L2021 EN**: Declares TableGen def record `VCREATE_BF`.
  **L2021 CN**: 声明 TableGen def 记录 `VCREATE_BF`。
- **L2022 EN**: Assigns a TableGen property that affects following records or inherited fields: `let BigEndianSafe = 1;`.
  **L2022 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let BigEndianSafe = 1;`。
- **L2023 EN**: Closes the current lexical scope or compound statement.
  **L2023 CN**: 结束当前词法作用域或复合语句块。
- **L2024 EN**: Blank line separating nearby declarations or logic blocks.
  **L2024 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2025 EN**: Declares TableGen def record `VDUP_N_BF`.
  **L2025 CN**: 声明 TableGen def 记录 `VDUP_N_BF`。
- **L2026 EN**: Blank line separating nearby declarations or logic blocks.
  **L2026 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2027 EN**: Declares TableGen def record `VDUP_LANE_BF`.
  **L2027 CN**: 声明 TableGen def 记录 `VDUP_LANE_BF`。
- **L2028 EN**: Declares TableGen def record `VDUP_LANEQ_BF`.
  **L2028 CN**: 声明 TableGen def 记录 `VDUP_LANEQ_BF`。
- **L2029 EN**: Blank line separating nearby declarations or logic blocks.
  **L2029 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2030 EN**: Declares TableGen def record `VCOMBINE_BF`.
  **L2030 CN**: 声明 TableGen def 记录 `VCOMBINE_BF`。
- **L2031 EN**: Blank line separating nearby declarations or logic blocks.
  **L2031 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2032 EN**: Declares TableGen def record `VGET_HIGH_BF`.
  **L2032 CN**: 声明 TableGen def 记录 `VGET_HIGH_BF`。
- **L2033 EN**: Declares TableGen def record `VGET_LOW_BF`.
  **L2033 CN**: 声明 TableGen def 记录 `VGET_LOW_BF`。
- **L2034 EN**: Blank line separating nearby declarations or logic blocks.
  **L2034 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2035 EN**: Declares TableGen def record `VGET_LANE_BF`.
  **L2035 CN**: 声明 TableGen def 记录 `VGET_LANE_BF`。
- **L2036 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheckLaneIndex, 0>]>;`.
  **L2036 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheckLaneIndex, 0>]>;`。
- **L2037 EN**: Declares TableGen def record `VSET_LANE_BF`.
  **L2037 CN**: 声明 TableGen def 记录 `VSET_LANE_BF`。
- **L2038 EN**: Adds a standalone statement or declaration: `[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`.
  **L2038 CN**: 添加一条独立语句或声明：`[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`。
- **L2039 EN**: Declares TableGen def record `SCALAR_VDUP_LANE_BF`.
  **L2039 CN**: 声明 TableGen def 记录 `SCALAR_VDUP_LANE_BF`。
- **L2040 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheckLaneIndex, 0>]>;`.
  **L2040 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheckLaneIndex, 0>]>;`。
- **L2041 EN**: Declares TableGen def record `SCALAR_VDUP_LANEQ_BF`.
  **L2041 CN**: 声明 TableGen def 记录 `SCALAR_VDUP_LANEQ_BF`。
- **L2042 EN**: Adds a standalone statement or declaration: `[ImmCheck<1, ImmCheckLaneIndex, 0>]>;`.
  **L2042 CN**: 添加一条独立语句或声明：`[ImmCheck<1, ImmCheckLaneIndex, 0>]>;`。
- **L2043 EN**: Blank line separating nearby declarations or logic blocks.
  **L2043 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2044 EN**: Declares TableGen def record `VLD1_BF`.
  **L2044 CN**: 声明 TableGen def 记录 `VLD1_BF`。

### Lines 2045-2072

````tablegen
  def VLD2_BF : WInst<"vld2", "2(c*!)", "bQb">;
  def VLD3_BF : WInst<"vld3", "3(c*!)", "bQb">;
  def VLD4_BF : WInst<"vld4", "4(c*!)", "bQb">;

  def VST1_BF : WInst<"vst1", "v*(.!)", "bQb">;
  def VST2_BF : WInst<"vst2", "v*(2!)", "bQb">;
  def VST3_BF : WInst<"vst3", "v*(3!)", "bQb">;
  def VST4_BF : WInst<"vst4", "v*(4!)", "bQb">;

  def VLD1_X2_BF : WInst<"vld1_x2", "2(c*!)", "bQb">;
  def VLD1_X3_BF : WInst<"vld1_x3", "3(c*!)", "bQb">;
  def VLD1_X4_BF : WInst<"vld1_x4", "4(c*!)", "bQb">;

  def VST1_X2_BF : WInst<"vst1_x2", "v*(2!)", "bQb">;
  def VST1_X3_BF : WInst<"vst1_x3", "v*(3!)", "bQb">;
  def VST1_X4_BF : WInst<"vst1_x4", "v*(4!)", "bQb">;

  def VLD1_LANE_BF : WInst<"vld1_lane", ".(c*!).I", "bQb",
                          [ImmCheck<2, ImmCheckLaneIndex, 1>]>;
  def VLD2_LANE_BF : WInst<"vld2_lane", "2(c*!)2I", "bQb",
                          [ImmCheck<4, ImmCheckLaneIndex, 1>]>;
  def VLD3_LANE_BF : WInst<"vld3_lane", "3(c*!)3I", "bQb",
                          [ImmCheck<5, ImmCheckLaneIndex, 1>]>;
  def VLD4_LANE_BF : WInst<"vld4_lane", "4(c*!)4I", "bQb",
                          [ImmCheck<6, ImmCheckLaneIndex, 1>]>;
  def VST1_LANE_BF : WInst<"vst1_lane", "v*(.!)I", "bQb",
                          [ImmCheck<2, ImmCheckLaneIndex, 1>]>;
  def VST2_LANE_BF : WInst<"vst2_lane", "v*(2!)I", "bQb",
````
- **L2045 EN**: Declares TableGen def record `VLD2_BF`.
  **L2045 CN**: 声明 TableGen def 记录 `VLD2_BF`。
- **L2046 EN**: Declares TableGen def record `VLD3_BF`.
  **L2046 CN**: 声明 TableGen def 记录 `VLD3_BF`。
- **L2047 EN**: Declares TableGen def record `VLD4_BF`.
  **L2047 CN**: 声明 TableGen def 记录 `VLD4_BF`。
- **L2048 EN**: Blank line separating nearby declarations or logic blocks.
  **L2048 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2049 EN**: Declares TableGen def record `VST1_BF`.
  **L2049 CN**: 声明 TableGen def 记录 `VST1_BF`。
- **L2050 EN**: Declares TableGen def record `VST2_BF`.
  **L2050 CN**: 声明 TableGen def 记录 `VST2_BF`。
- **L2051 EN**: Declares TableGen def record `VST3_BF`.
  **L2051 CN**: 声明 TableGen def 记录 `VST3_BF`。
- **L2052 EN**: Declares TableGen def record `VST4_BF`.
  **L2052 CN**: 声明 TableGen def 记录 `VST4_BF`。
- **L2053 EN**: Blank line separating nearby declarations or logic blocks.
  **L2053 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2054 EN**: Declares TableGen def record `VLD1_X2_BF`.
  **L2054 CN**: 声明 TableGen def 记录 `VLD1_X2_BF`。
- **L2055 EN**: Declares TableGen def record `VLD1_X3_BF`.
  **L2055 CN**: 声明 TableGen def 记录 `VLD1_X3_BF`。
- **L2056 EN**: Declares TableGen def record `VLD1_X4_BF`.
  **L2056 CN**: 声明 TableGen def 记录 `VLD1_X4_BF`。
- **L2057 EN**: Blank line separating nearby declarations or logic blocks.
  **L2057 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2058 EN**: Declares TableGen def record `VST1_X2_BF`.
  **L2058 CN**: 声明 TableGen def 记录 `VST1_X2_BF`。
- **L2059 EN**: Declares TableGen def record `VST1_X3_BF`.
  **L2059 CN**: 声明 TableGen def 记录 `VST1_X3_BF`。
- **L2060 EN**: Declares TableGen def record `VST1_X4_BF`.
  **L2060 CN**: 声明 TableGen def 记录 `VST1_X4_BF`。
- **L2061 EN**: Blank line separating nearby declarations or logic blocks.
  **L2061 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2062 EN**: Declares TableGen def record `VLD1_LANE_BF`.
  **L2062 CN**: 声明 TableGen def 记录 `VLD1_LANE_BF`。
- **L2063 EN**: Adds a standalone statement or declaration: `[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`.
  **L2063 CN**: 添加一条独立语句或声明：`[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`。
- **L2064 EN**: Declares TableGen def record `VLD2_LANE_BF`.
  **L2064 CN**: 声明 TableGen def 记录 `VLD2_LANE_BF`。
- **L2065 EN**: Adds a standalone statement or declaration: `[ImmCheck<4, ImmCheckLaneIndex, 1>]>;`.
  **L2065 CN**: 添加一条独立语句或声明：`[ImmCheck<4, ImmCheckLaneIndex, 1>]>;`。
- **L2066 EN**: Declares TableGen def record `VLD3_LANE_BF`.
  **L2066 CN**: 声明 TableGen def 记录 `VLD3_LANE_BF`。
- **L2067 EN**: Adds a standalone statement or declaration: `[ImmCheck<5, ImmCheckLaneIndex, 1>]>;`.
  **L2067 CN**: 添加一条独立语句或声明：`[ImmCheck<5, ImmCheckLaneIndex, 1>]>;`。
- **L2068 EN**: Declares TableGen def record `VLD4_LANE_BF`.
  **L2068 CN**: 声明 TableGen def 记录 `VLD4_LANE_BF`。
- **L2069 EN**: Adds a standalone statement or declaration: `[ImmCheck<6, ImmCheckLaneIndex, 1>]>;`.
  **L2069 CN**: 添加一条独立语句或声明：`[ImmCheck<6, ImmCheckLaneIndex, 1>]>;`。
- **L2070 EN**: Declares TableGen def record `VST1_LANE_BF`.
  **L2070 CN**: 声明 TableGen def 记录 `VST1_LANE_BF`。
- **L2071 EN**: Adds a standalone statement or declaration: `[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`.
  **L2071 CN**: 添加一条独立语句或声明：`[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`。
- **L2072 EN**: Declares TableGen def record `VST2_LANE_BF`.
  **L2072 CN**: 声明 TableGen def 记录 `VST2_LANE_BF`。

### Lines 2073-2100

````tablegen
                          [ImmCheck<3, ImmCheckLaneIndex, 1>]>;
  def VST3_LANE_BF : WInst<"vst3_lane", "v*(3!)I", "bQb",
                          [ImmCheck<4, ImmCheckLaneIndex, 1>]>;
  def VST4_LANE_BF : WInst<"vst4_lane", "v*(4!)I", "bQb",
                          [ImmCheck<5, ImmCheckLaneIndex, 1>]>;

  def VLD1_DUP_BF : WInst<"vld1_dup", ".(c*!)", "bQb">;
  def VLD2_DUP_BF : WInst<"vld2_dup", "2(c*!)", "bQb">;
  def VLD3_DUP_BF : WInst<"vld3_dup", "3(c*!)", "bQb">;
  def VLD4_DUP_BF : WInst<"vld4_dup", "4(c*!)", "bQb">;

  def VCVT_F32_BF16 : SOpInst<"vcvt_f32_bf16", "(F>)(Bq!)",  "Qb", OP_VCVT_F32_BF16>;
  def VCVT_LOW_F32_BF16 : SOpInst<"vcvt_low_f32", "(F>)(BQ!)",  "Qb", OP_VCVT_F32_BF16_LO>;
  def VCVT_HIGH_F32_BF16 : SOpInst<"vcvt_high_f32", "(F>)(BQ!)", "Qb", OP_VCVT_F32_BF16_HI>;

  def SCALAR_CVT_BF16_F32 : SInst<"vcvth_bf16", "(1B)1", "f">;
  def SCALAR_CVT_F32_BF16 : SOpInst<"vcvtah_f32", "(1F>)(1!)", "b", OP_CVT_F32_BF16>;
}

let ArchGuard = "!defined(__aarch64__) && !defined(__arm64ec__)", TargetGuard = "bf16,neon" in {
  def VCVT_BF16_F32_A32_INTERNAL : WInst<"__a32_vcvt_bf16", "BQ", "f">;
  def VCVT_BF16_F32_A32 : SOpInst<"vcvt_bf16", "BQ", "f", OP_VCVT_BF16_F32_A32>;
  def VCVT_LOW_BF16_F32_A32 : SOpInst<"vcvt_low_bf16",  "BQ", "Qf", OP_VCVT_BF16_F32_LO_A32>;
  def VCVT_HIGH_BF16_F32_A32 : SOpInst<"vcvt_high_bf16", "BBQ", "Qf", OP_VCVT_BF16_F32_HI_A32>;
}

let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "bf16,neon" in {
  def VCVT_LOW_BF16_F32_A64 : SInst<"vcvt_low_bf16", "BQ", "Qf">;
````
- **L2073 EN**: Adds a standalone statement or declaration: `[ImmCheck<3, ImmCheckLaneIndex, 1>]>;`.
  **L2073 CN**: 添加一条独立语句或声明：`[ImmCheck<3, ImmCheckLaneIndex, 1>]>;`。
- **L2074 EN**: Declares TableGen def record `VST3_LANE_BF`.
  **L2074 CN**: 声明 TableGen def 记录 `VST3_LANE_BF`。
- **L2075 EN**: Adds a standalone statement or declaration: `[ImmCheck<4, ImmCheckLaneIndex, 1>]>;`.
  **L2075 CN**: 添加一条独立语句或声明：`[ImmCheck<4, ImmCheckLaneIndex, 1>]>;`。
- **L2076 EN**: Declares TableGen def record `VST4_LANE_BF`.
  **L2076 CN**: 声明 TableGen def 记录 `VST4_LANE_BF`。
- **L2077 EN**: Adds a standalone statement or declaration: `[ImmCheck<5, ImmCheckLaneIndex, 1>]>;`.
  **L2077 CN**: 添加一条独立语句或声明：`[ImmCheck<5, ImmCheckLaneIndex, 1>]>;`。
- **L2078 EN**: Blank line separating nearby declarations or logic blocks.
  **L2078 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2079 EN**: Declares TableGen def record `VLD1_DUP_BF`.
  **L2079 CN**: 声明 TableGen def 记录 `VLD1_DUP_BF`。
- **L2080 EN**: Declares TableGen def record `VLD2_DUP_BF`.
  **L2080 CN**: 声明 TableGen def 记录 `VLD2_DUP_BF`。
- **L2081 EN**: Declares TableGen def record `VLD3_DUP_BF`.
  **L2081 CN**: 声明 TableGen def 记录 `VLD3_DUP_BF`。
- **L2082 EN**: Declares TableGen def record `VLD4_DUP_BF`.
  **L2082 CN**: 声明 TableGen def 记录 `VLD4_DUP_BF`。
- **L2083 EN**: Blank line separating nearby declarations or logic blocks.
  **L2083 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2084 EN**: Declares TableGen def record `VCVT_F32_BF16`.
  **L2084 CN**: 声明 TableGen def 记录 `VCVT_F32_BF16`。
- **L2085 EN**: Declares TableGen def record `VCVT_LOW_F32_BF16`.
  **L2085 CN**: 声明 TableGen def 记录 `VCVT_LOW_F32_BF16`。
- **L2086 EN**: Declares TableGen def record `VCVT_HIGH_F32_BF16`.
  **L2086 CN**: 声明 TableGen def 记录 `VCVT_HIGH_F32_BF16`。
- **L2087 EN**: Blank line separating nearby declarations or logic blocks.
  **L2087 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2088 EN**: Declares TableGen def record `SCALAR_CVT_BF16_F32`.
  **L2088 CN**: 声明 TableGen def 记录 `SCALAR_CVT_BF16_F32`。
- **L2089 EN**: Declares TableGen def record `SCALAR_CVT_F32_BF16`.
  **L2089 CN**: 声明 TableGen def 记录 `SCALAR_CVT_F32_BF16`。
- **L2090 EN**: Closes the current lexical scope or compound statement.
  **L2090 CN**: 结束当前词法作用域或复合语句块。
- **L2091 EN**: Blank line separating nearby declarations or logic blocks.
  **L2091 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2092 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "!defined(__aarch64__) && !defined(__arm64ec__)", TargetGuard = "bf16,neon" in {`.
  **L2092 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "!defined(__aarch64__) && !defined(__arm64ec__)", TargetGuard = "bf16,neon" in {`。
- **L2093 EN**: Declares TableGen def record `VCVT_BF16_F32_A32_INTERNAL`.
  **L2093 CN**: 声明 TableGen def 记录 `VCVT_BF16_F32_A32_INTERNAL`。
- **L2094 EN**: Declares TableGen def record `VCVT_BF16_F32_A32`.
  **L2094 CN**: 声明 TableGen def 记录 `VCVT_BF16_F32_A32`。
- **L2095 EN**: Declares TableGen def record `VCVT_LOW_BF16_F32_A32`.
  **L2095 CN**: 声明 TableGen def 记录 `VCVT_LOW_BF16_F32_A32`。
- **L2096 EN**: Declares TableGen def record `VCVT_HIGH_BF16_F32_A32`.
  **L2096 CN**: 声明 TableGen def 记录 `VCVT_HIGH_BF16_F32_A32`。
- **L2097 EN**: Closes the current lexical scope or compound statement.
  **L2097 CN**: 结束当前词法作用域或复合语句块。
- **L2098 EN**: Blank line separating nearby declarations or logic blocks.
  **L2098 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2099 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "bf16,neon" in {`.
  **L2099 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "bf16,neon" in {`。
- **L2100 EN**: Declares TableGen def record `VCVT_LOW_BF16_F32_A64`.
  **L2100 CN**: 声明 TableGen def 记录 `VCVT_LOW_BF16_F32_A64`。

### Lines 2101-2128

````tablegen
  def VCVT_HIGH_BF16_F32_A64 : SInst<"vcvt_high_bf16", "BBQ", "Qf">;
  def VCVT_BF16_F32 : SInst<"vcvt_bf16", "BQ", "f">;

  def COPY_LANE_BF16 : IOpInst<"vcopy_lane", "..I.I", "b", OP_COPY_LN>;
  def COPYQ_LANE_BF16 : IOpInst<"vcopy_lane", "..IqI", "Qb", OP_COPY_LN>;
  def COPY_LANEQ_BF16 : IOpInst<"vcopy_laneq", "..IQI", "b", OP_COPY_LN>;
  def COPYQ_LANEQ_BF16 : IOpInst<"vcopy_laneq", "..I.I", "Qb", OP_COPY_LN>;
}

let ArchGuard = "!defined(__aarch64__) && !defined(__arm64ec__)", TargetGuard = "bf16,neon" in {
  let BigEndianSafe = 1 in {
    defm VREINTERPRET_BF : REINTERPRET_CROSS_TYPES<
        "csilUcUsUiUlhfPcPsPlQcQsQiQlQUcQUsQUiQUlQhQfQPcQPsQPl", "bQb">;
  }
}

let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "bf16,neon" in {
  let BigEndianSafe = 1 in {
    defm VVREINTERPRET_BF : REINTERPRET_CROSS_TYPES<
        "csilUcUsUiUlhfdPcPsPlQcQsQiQlQUcQUsQUiQUlQhQfQdQPcQPsQPlQPk", "bQb">;
  }
}

// v8.9a/v9.4a LRCPC3 intrinsics
let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "rcpc3,neon" in {
  def VLDAP1_LANE : WInst<"vldap1_lane", ".(c*!).I", "QUlQlUlldQdPlQPl",
                        [ImmCheck<2, ImmCheckLaneIndex, 1>]>;
  def VSTL1_LANE  : WInst<"vstl1_lane", "v*(.!)I", "QUlQlUlldQdPlQPl",
````
- **L2101 EN**: Declares TableGen def record `VCVT_HIGH_BF16_F32_A64`.
  **L2101 CN**: 声明 TableGen def 记录 `VCVT_HIGH_BF16_F32_A64`。
- **L2102 EN**: Declares TableGen def record `VCVT_BF16_F32`.
  **L2102 CN**: 声明 TableGen def 记录 `VCVT_BF16_F32`。
- **L2103 EN**: Blank line separating nearby declarations or logic blocks.
  **L2103 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2104 EN**: Declares TableGen def record `COPY_LANE_BF16`.
  **L2104 CN**: 声明 TableGen def 记录 `COPY_LANE_BF16`。
- **L2105 EN**: Declares TableGen def record `COPYQ_LANE_BF16`.
  **L2105 CN**: 声明 TableGen def 记录 `COPYQ_LANE_BF16`。
- **L2106 EN**: Declares TableGen def record `COPY_LANEQ_BF16`.
  **L2106 CN**: 声明 TableGen def 记录 `COPY_LANEQ_BF16`。
- **L2107 EN**: Declares TableGen def record `COPYQ_LANEQ_BF16`.
  **L2107 CN**: 声明 TableGen def 记录 `COPYQ_LANEQ_BF16`。
- **L2108 EN**: Closes the current lexical scope or compound statement.
  **L2108 CN**: 结束当前词法作用域或复合语句块。
- **L2109 EN**: Blank line separating nearby declarations or logic blocks.
  **L2109 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2110 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "!defined(__aarch64__) && !defined(__arm64ec__)", TargetGuard = "bf16,neon" in {`.
  **L2110 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "!defined(__aarch64__) && !defined(__arm64ec__)", TargetGuard = "bf16,neon" in {`。
- **L2111 EN**: Assigns a TableGen property that affects following records or inherited fields: `let BigEndianSafe = 1 in {`.
  **L2111 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let BigEndianSafe = 1 in {`。
- **L2112 EN**: Declares TableGen defm record `VREINTERPRET_BF`.
  **L2112 CN**: 声明 TableGen defm 记录 `VREINTERPRET_BF`。
- **L2113 EN**: Adds a standalone statement or declaration: `"csilUcUsUiUlhfPcPsPlQcQsQiQlQUcQUsQUiQUlQhQfQPcQPsQPl", "bQb">;`.
  **L2113 CN**: 添加一条独立语句或声明：`"csilUcUsUiUlhfPcPsPlQcQsQiQlQUcQUsQUiQUlQhQfQPcQPsQPl", "bQb">;`。
- **L2114 EN**: Closes the current lexical scope or compound statement.
  **L2114 CN**: 结束当前词法作用域或复合语句块。
- **L2115 EN**: Closes the current lexical scope or compound statement.
  **L2115 CN**: 结束当前词法作用域或复合语句块。
- **L2116 EN**: Blank line separating nearby declarations or logic blocks.
  **L2116 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2117 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "bf16,neon" in {`.
  **L2117 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "bf16,neon" in {`。
- **L2118 EN**: Assigns a TableGen property that affects following records or inherited fields: `let BigEndianSafe = 1 in {`.
  **L2118 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let BigEndianSafe = 1 in {`。
- **L2119 EN**: Declares TableGen defm record `VVREINTERPRET_BF`.
  **L2119 CN**: 声明 TableGen defm 记录 `VVREINTERPRET_BF`。
- **L2120 EN**: Adds a standalone statement or declaration: `"csilUcUsUiUlhfdPcPsPlQcQsQiQlQUcQUsQUiQUlQhQfQdQPcQPsQPlQPk", "bQb">;`.
  **L2120 CN**: 添加一条独立语句或声明：`"csilUcUsUiUlhfdPcPsPlQcQsQiQlQUcQUsQUiQUlQhQfQdQPcQPsQPlQPk", "bQb">;`。
- **L2121 EN**: Closes the current lexical scope or compound statement.
  **L2121 CN**: 结束当前词法作用域或复合语句块。
- **L2122 EN**: Closes the current lexical scope or compound statement.
  **L2122 CN**: 结束当前词法作用域或复合语句块。
- **L2123 EN**: Blank line separating nearby declarations or logic blocks.
  **L2123 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2124 EN**: Comment explains nearby logic, constraints, or intent: `v8.9a/v9.4a LRCPC3 intrinsics`.
  **L2124 CN**: 注释解释附近代码的逻辑、约束或设计意图：`v8.9a/v9.4a LRCPC3 intrinsics`。
- **L2125 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "rcpc3,neon" in {`.
  **L2125 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "defined(__aarch64__) || defined(__arm64ec__)", TargetGuard = "rcpc3,neon" in {`。
- **L2126 EN**: Declares TableGen def record `VLDAP1_LANE`.
  **L2126 CN**: 声明 TableGen def 记录 `VLDAP1_LANE`。
- **L2127 EN**: Adds a standalone statement or declaration: `[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`.
  **L2127 CN**: 添加一条独立语句或声明：`[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`。
- **L2128 EN**: Declares TableGen def record `VSTL1_LANE`.
  **L2128 CN**: 声明 TableGen def 记录 `VSTL1_LANE`。

### Lines 2129-2156

````tablegen
                        [ImmCheck<2, ImmCheckLaneIndex, 1>]>;
}

// Lookup table read with 2-bit/4-bit indices
let ArchGuard = "defined(__aarch64__)", TargetGuard = "lut" in {
  def VLUTI2_B    : SInst<"vluti2_lane", "Q.(qU)I", "cUcPcmQcQUcQPcQm",
                         [ImmCheck<2, ImmCheck0_1>]>;
  def VLUTI2_B_Q  : SInst<"vluti2_laneq", "Q.(QU)I", "cUcPcmQcQUcQPcQm",
                         [ImmCheck<2, ImmCheck0_3>]>;
  def VLUTI2_H    : SInst<"vluti2_lane", "Q.(<qU)I", "sUsPshQsQUsQPsQh",
                         [ImmCheck<2, ImmCheck0_3>]>;
  def VLUTI2_H_Q  : SInst<"vluti2_laneq", "Q.(<QU)I", "sUsPshQsQUsQPsQh",
                         [ImmCheck<2, ImmCheck0_7>]>;
  def VLUTI4_B    : SInst<"vluti4_lane", "..(qU)I", "QcQUcQPcQm",
                         [ImmCheck<2, ImmCheck0_0>]>;
  def VLUTI4_B_Q  : SInst<"vluti4_laneq", "..UI", "QcQUcQPcQm",
                         [ImmCheck<2, ImmCheck0_1>]>;
  def VLUTI4_H_X2 : SInst<"vluti4_lane_x2", ".2(<qU)I", "QsQUsQPsQh",
                          [ImmCheck<3, ImmCheck0_1>]>;
  def VLUTI4_H_X2_Q : SInst<"vluti4_laneq_x2", ".2(<U)I", "QsQUsQPsQh",
                          [ImmCheck<3, ImmCheck0_3>]>;

  let TargetGuard = "lut,bf16" in {
    def VLUTI2_BF      : SInst<"vluti2_lane", "Q.(<qU)I", "bQb",
                              [ImmCheck<2, ImmCheck0_3>]>;
    def VLUTI2_BF_Q    : SInst<"vluti2_laneq", "Q.(<QU)I", "bQb",
                              [ImmCheck<2, ImmCheck0_7>]>;
    def VLUTI4_BF_X2   : SInst<"vluti4_lane_x2", ".2(<qU)I", "Qb",
````
- **L2129 EN**: Adds a standalone statement or declaration: `[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`.
  **L2129 CN**: 添加一条独立语句或声明：`[ImmCheck<2, ImmCheckLaneIndex, 1>]>;`。
- **L2130 EN**: Closes the current lexical scope or compound statement.
  **L2130 CN**: 结束当前词法作用域或复合语句块。
- **L2131 EN**: Blank line separating nearby declarations or logic blocks.
  **L2131 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2132 EN**: Comment explains nearby logic, constraints, or intent: `Lookup table read with 2-bit/4-bit indices`.
  **L2132 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Lookup table read with 2-bit/4-bit indices`。
- **L2133 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "defined(__aarch64__)", TargetGuard = "lut" in {`.
  **L2133 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "defined(__aarch64__)", TargetGuard = "lut" in {`。
- **L2134 EN**: Declares TableGen def record `VLUTI2_B`.
  **L2134 CN**: 声明 TableGen def 记录 `VLUTI2_B`。
- **L2135 EN**: Adds a standalone statement or declaration: `[ImmCheck<2, ImmCheck0_1>]>;`.
  **L2135 CN**: 添加一条独立语句或声明：`[ImmCheck<2, ImmCheck0_1>]>;`。
- **L2136 EN**: Declares TableGen def record `VLUTI2_B_Q`.
  **L2136 CN**: 声明 TableGen def 记录 `VLUTI2_B_Q`。
- **L2137 EN**: Adds a standalone statement or declaration: `[ImmCheck<2, ImmCheck0_3>]>;`.
  **L2137 CN**: 添加一条独立语句或声明：`[ImmCheck<2, ImmCheck0_3>]>;`。
- **L2138 EN**: Declares TableGen def record `VLUTI2_H`.
  **L2138 CN**: 声明 TableGen def 记录 `VLUTI2_H`。
- **L2139 EN**: Adds a standalone statement or declaration: `[ImmCheck<2, ImmCheck0_3>]>;`.
  **L2139 CN**: 添加一条独立语句或声明：`[ImmCheck<2, ImmCheck0_3>]>;`。
- **L2140 EN**: Declares TableGen def record `VLUTI2_H_Q`.
  **L2140 CN**: 声明 TableGen def 记录 `VLUTI2_H_Q`。
- **L2141 EN**: Adds a standalone statement or declaration: `[ImmCheck<2, ImmCheck0_7>]>;`.
  **L2141 CN**: 添加一条独立语句或声明：`[ImmCheck<2, ImmCheck0_7>]>;`。
- **L2142 EN**: Declares TableGen def record `VLUTI4_B`.
  **L2142 CN**: 声明 TableGen def 记录 `VLUTI4_B`。
- **L2143 EN**: Adds a standalone statement or declaration: `[ImmCheck<2, ImmCheck0_0>]>;`.
  **L2143 CN**: 添加一条独立语句或声明：`[ImmCheck<2, ImmCheck0_0>]>;`。
- **L2144 EN**: Declares TableGen def record `VLUTI4_B_Q`.
  **L2144 CN**: 声明 TableGen def 记录 `VLUTI4_B_Q`。
- **L2145 EN**: Adds a standalone statement or declaration: `[ImmCheck<2, ImmCheck0_1>]>;`.
  **L2145 CN**: 添加一条独立语句或声明：`[ImmCheck<2, ImmCheck0_1>]>;`。
- **L2146 EN**: Declares TableGen def record `VLUTI4_H_X2`.
  **L2146 CN**: 声明 TableGen def 记录 `VLUTI4_H_X2`。
- **L2147 EN**: Adds a standalone statement or declaration: `[ImmCheck<3, ImmCheck0_1>]>;`.
  **L2147 CN**: 添加一条独立语句或声明：`[ImmCheck<3, ImmCheck0_1>]>;`。
- **L2148 EN**: Declares TableGen def record `VLUTI4_H_X2_Q`.
  **L2148 CN**: 声明 TableGen def 记录 `VLUTI4_H_X2_Q`。
- **L2149 EN**: Adds a standalone statement or declaration: `[ImmCheck<3, ImmCheck0_3>]>;`.
  **L2149 CN**: 添加一条独立语句或声明：`[ImmCheck<3, ImmCheck0_3>]>;`。
- **L2150 EN**: Blank line separating nearby declarations or logic blocks.
  **L2150 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2151 EN**: Assigns a TableGen property that affects following records or inherited fields: `let TargetGuard = "lut,bf16" in {`.
  **L2151 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let TargetGuard = "lut,bf16" in {`。
- **L2152 EN**: Declares TableGen def record `VLUTI2_BF`.
  **L2152 CN**: 声明 TableGen def 记录 `VLUTI2_BF`。
- **L2153 EN**: Adds a standalone statement or declaration: `[ImmCheck<2, ImmCheck0_3>]>;`.
  **L2153 CN**: 添加一条独立语句或声明：`[ImmCheck<2, ImmCheck0_3>]>;`。
- **L2154 EN**: Declares TableGen def record `VLUTI2_BF_Q`.
  **L2154 CN**: 声明 TableGen def 记录 `VLUTI2_BF_Q`。
- **L2155 EN**: Adds a standalone statement or declaration: `[ImmCheck<2, ImmCheck0_7>]>;`.
  **L2155 CN**: 添加一条独立语句或声明：`[ImmCheck<2, ImmCheck0_7>]>;`。
- **L2156 EN**: Declares TableGen def record `VLUTI4_BF_X2`.
  **L2156 CN**: 声明 TableGen def 记录 `VLUTI4_BF_X2`。

### Lines 2157-2184

````tablegen
                              [ImmCheck<3, ImmCheck0_1>]>;
    def VLUTI4_BF_X2_Q   : SInst<"vluti4_laneq_x2", ".2(<U)I", "Qb",
                              [ImmCheck<3, ImmCheck0_3>]>;
  }
}

let ArchGuard = "defined(__aarch64__)", TargetGuard = "fp8,neon" in {
  def VBF1CVT_BF16_MF8        : VInst<"vcvt1_bf16_mf8_fpm",      "(QB).V", "m">;
  def VBF1CVT_LOW_BF16_MF8    : VInst<"vcvt1_low_bf16_mf8_fpm",  "B.V",    "Hm">;
  def VBF2CVTL_BF16_MF8       : VInst<"vcvt2_bf16_mf8_fpm",      "(QB).V", "m">;
  def VBF2CVTL_LOW_BF16_MF8   : VInst<"vcvt2_low_bf16_mf8_fpm",  "B.V",    "Hm">;
  def VBF1CVTL2_HIGH_BF16_MF8 : VInst<"vcvt1_high_bf16_mf8_fpm", "B.V",    "Hm">;
  def VBF2CVTL2_HIGH_BF16_MF8 : VInst<"vcvt2_high_bf16_mf8_fpm", "B.V",    "Hm">;
}

let ArchGuard = "defined(__aarch64__)", TargetGuard = "fp8,neon" in {
  def VF1CVT_F16_MF8        : VInst<"vcvt1_f16_mf8_fpm",      "(>QF).V", "m">;
  def VF1CVT_LOW_F16_MF8    : VInst<"vcvt1_low_f16_mf8_fpm",  "(>F).V",  "Hm">;
  def VF2CVTL_F16_MF8       : VInst<"vcvt2_f16_mf8_fpm",      "(>QF).V", "m">;
  def VF2CVTL_LOW_F16_MF8   : VInst<"vcvt2_low_f16_mf8_fpm",  "(>F).V",  "Hm">;
  def VF1CVTL2_HIGH_F16_MF8 : VInst<"vcvt1_high_f16_mf8_fpm", "(>F).V",  "Hm">;
  def VF2CVTL2_HIGH_F16_MF8 : VInst<"vcvt2_high_f16_mf8_fpm", "(>F).V",  "Hm">;

  def VCVTN_LOW_F8_F32  : VInst<"vcvt_mf8_f32_fpm",      ".(>>QF)(>>QF)V",  "m">;
  def VCVTN_HIGH_F8_F32 : VInst<"vcvt_high_mf8_f32_fpm", ".(q)(>>F)(>>F)V", "Hm">;
  def VCVTN_F8_F16      : VInst<"vcvt_mf8_f16_fpm",      ".(>F)(>F)V",      "mQm">;
}

````
- **L2157 EN**: Adds a standalone statement or declaration: `[ImmCheck<3, ImmCheck0_1>]>;`.
  **L2157 CN**: 添加一条独立语句或声明：`[ImmCheck<3, ImmCheck0_1>]>;`。
- **L2158 EN**: Declares TableGen def record `VLUTI4_BF_X2_Q`.
  **L2158 CN**: 声明 TableGen def 记录 `VLUTI4_BF_X2_Q`。
- **L2159 EN**: Adds a standalone statement or declaration: `[ImmCheck<3, ImmCheck0_3>]>;`.
  **L2159 CN**: 添加一条独立语句或声明：`[ImmCheck<3, ImmCheck0_3>]>;`。
- **L2160 EN**: Closes the current lexical scope or compound statement.
  **L2160 CN**: 结束当前词法作用域或复合语句块。
- **L2161 EN**: Closes the current lexical scope or compound statement.
  **L2161 CN**: 结束当前词法作用域或复合语句块。
- **L2162 EN**: Blank line separating nearby declarations or logic blocks.
  **L2162 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2163 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "defined(__aarch64__)", TargetGuard = "fp8,neon" in {`.
  **L2163 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "defined(__aarch64__)", TargetGuard = "fp8,neon" in {`。
- **L2164 EN**: Declares TableGen def record `VBF1CVT_BF16_MF8`.
  **L2164 CN**: 声明 TableGen def 记录 `VBF1CVT_BF16_MF8`。
- **L2165 EN**: Declares TableGen def record `VBF1CVT_LOW_BF16_MF8`.
  **L2165 CN**: 声明 TableGen def 记录 `VBF1CVT_LOW_BF16_MF8`。
- **L2166 EN**: Declares TableGen def record `VBF2CVTL_BF16_MF8`.
  **L2166 CN**: 声明 TableGen def 记录 `VBF2CVTL_BF16_MF8`。
- **L2167 EN**: Declares TableGen def record `VBF2CVTL_LOW_BF16_MF8`.
  **L2167 CN**: 声明 TableGen def 记录 `VBF2CVTL_LOW_BF16_MF8`。
- **L2168 EN**: Declares TableGen def record `VBF1CVTL2_HIGH_BF16_MF8`.
  **L2168 CN**: 声明 TableGen def 记录 `VBF1CVTL2_HIGH_BF16_MF8`。
- **L2169 EN**: Declares TableGen def record `VBF2CVTL2_HIGH_BF16_MF8`.
  **L2169 CN**: 声明 TableGen def 记录 `VBF2CVTL2_HIGH_BF16_MF8`。
- **L2170 EN**: Closes the current lexical scope or compound statement.
  **L2170 CN**: 结束当前词法作用域或复合语句块。
- **L2171 EN**: Blank line separating nearby declarations or logic blocks.
  **L2171 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2172 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "defined(__aarch64__)", TargetGuard = "fp8,neon" in {`.
  **L2172 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "defined(__aarch64__)", TargetGuard = "fp8,neon" in {`。
- **L2173 EN**: Declares TableGen def record `VF1CVT_F16_MF8`.
  **L2173 CN**: 声明 TableGen def 记录 `VF1CVT_F16_MF8`。
- **L2174 EN**: Declares TableGen def record `VF1CVT_LOW_F16_MF8`.
  **L2174 CN**: 声明 TableGen def 记录 `VF1CVT_LOW_F16_MF8`。
- **L2175 EN**: Declares TableGen def record `VF2CVTL_F16_MF8`.
  **L2175 CN**: 声明 TableGen def 记录 `VF2CVTL_F16_MF8`。
- **L2176 EN**: Declares TableGen def record `VF2CVTL_LOW_F16_MF8`.
  **L2176 CN**: 声明 TableGen def 记录 `VF2CVTL_LOW_F16_MF8`。
- **L2177 EN**: Declares TableGen def record `VF1CVTL2_HIGH_F16_MF8`.
  **L2177 CN**: 声明 TableGen def 记录 `VF1CVTL2_HIGH_F16_MF8`。
- **L2178 EN**: Declares TableGen def record `VF2CVTL2_HIGH_F16_MF8`.
  **L2178 CN**: 声明 TableGen def 记录 `VF2CVTL2_HIGH_F16_MF8`。
- **L2179 EN**: Blank line separating nearby declarations or logic blocks.
  **L2179 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2180 EN**: Declares TableGen def record `VCVTN_LOW_F8_F32`.
  **L2180 CN**: 声明 TableGen def 记录 `VCVTN_LOW_F8_F32`。
- **L2181 EN**: Declares TableGen def record `VCVTN_HIGH_F8_F32`.
  **L2181 CN**: 声明 TableGen def 记录 `VCVTN_HIGH_F8_F32`。
- **L2182 EN**: Declares TableGen def record `VCVTN_F8_F16`.
  **L2182 CN**: 声明 TableGen def 记录 `VCVTN_F8_F16`。
- **L2183 EN**: Closes the current lexical scope or compound statement.
  **L2183 CN**: 结束当前词法作用域或复合语句块。
- **L2184 EN**: Blank line separating nearby declarations or logic blocks.
  **L2184 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 2185-2212

````tablegen
let ArchGuard = "defined(__aarch64__)", TargetGuard = "fp8dot2,neon" in {
  def VDOT_F16_MF8 : VInst<"vdot_f16_mf8_fpm", "(>F)(>F)..V", "mQm">;

  def VDOT_LANE_F16_MF8  : VInst<"vdot_lane_f16_mf8_fpm",  "(>F)(>F)..IV", "m", [ImmCheck<3, ImmCheck0_3, 0>]>;
  def VDOT_LANEQ_F16_MF8 : VInst<"vdot_laneq_f16_mf8_fpm", "(>F)(>F).QIV", "m", [ImmCheck<3, ImmCheck0_7, 0>]>;

  def VDOTQ_LANE_F16_MF8  : VInst<"vdot_lane_f16_mf8_fpm",  "(>F)(>F).qIV", "Qm", [ImmCheck<3, ImmCheck0_3, 0>]>;
  def VDOTQ_LANEQ_F16_MF8 : VInst<"vdot_laneq_f16_mf8_fpm", "(>F)(>F)..IV", "Qm", [ImmCheck<3, ImmCheck0_7, 0>]>;
}

let ArchGuard = "defined(__aarch64__)", TargetGuard = "fp8dot4,neon" in {
  def VDOT_F32_MF8 : VInst<"vdot_f32_mf8_fpm", "(>>F)(>>F)..V", "mQm">;

  def VDOT_LANE_F32_MF8  : VInst<"vdot_lane_f32_mf8_fpm",  "(>>F)(>>F)..IV", "m", [ImmCheck<3, ImmCheck0_1, 0>]>;
  def VDOT_LANEQ_F32_MF8 : VInst<"vdot_laneq_f32_mf8_fpm", "(>>F)(>>F).QIV", "m", [ImmCheck<3, ImmCheck0_3, 0>]>;

  def VDOTQ_LANE_F32_MF8  : VInst<"vdot_lane_f32_mf8_fpm",  "(>>F)(>>F).qIV", "Qm", [ImmCheck<3, ImmCheck0_1, 0>]>;
  def VDOTQ_LANEQ_F32_MF8 : VInst<"vdot_laneq_f32_mf8_fpm", "(>>F)(>>F)..IV", "Qm", [ImmCheck<3, ImmCheck0_3, 0>]>;
}

let ArchGuard = "defined(__aarch64__)", TargetGuard = "fp8fma,neon" in {
  def VMLALB_F16_F8 : VInst<"vmlalb_f16_mf8_fpm", "(>F)(>F)..V", "Qm">;
  def VMLALT_F16_F8 : VInst<"vmlalt_f16_mf8_fpm", "(>F)(>F)..V", "Qm">;

  def VMLALLBB_F32_F8 : VInst<"vmlallbb_f32_mf8_fpm", "(>>F)(>>F)..V", "Qm">;
  def VMLALLBT_F32_F8 : VInst<"vmlallbt_f32_mf8_fpm", "(>>F)(>>F)..V", "Qm">;
  def VMLALLTB_F32_F8 : VInst<"vmlalltb_f32_mf8_fpm", "(>>F)(>>F)..V", "Qm">;
  def VMLALLTT_F32_F8 : VInst<"vmlalltt_f32_mf8_fpm", "(>>F)(>>F)..V", "Qm">;
````
- **L2185 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "defined(__aarch64__)", TargetGuard = "fp8dot2,neon" in {`.
  **L2185 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "defined(__aarch64__)", TargetGuard = "fp8dot2,neon" in {`。
- **L2186 EN**: Declares TableGen def record `VDOT_F16_MF8`.
  **L2186 CN**: 声明 TableGen def 记录 `VDOT_F16_MF8`。
- **L2187 EN**: Blank line separating nearby declarations or logic blocks.
  **L2187 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2188 EN**: Declares TableGen def record `VDOT_LANE_F16_MF8`.
  **L2188 CN**: 声明 TableGen def 记录 `VDOT_LANE_F16_MF8`。
- **L2189 EN**: Declares TableGen def record `VDOT_LANEQ_F16_MF8`.
  **L2189 CN**: 声明 TableGen def 记录 `VDOT_LANEQ_F16_MF8`。
- **L2190 EN**: Blank line separating nearby declarations or logic blocks.
  **L2190 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2191 EN**: Declares TableGen def record `VDOTQ_LANE_F16_MF8`.
  **L2191 CN**: 声明 TableGen def 记录 `VDOTQ_LANE_F16_MF8`。
- **L2192 EN**: Declares TableGen def record `VDOTQ_LANEQ_F16_MF8`.
  **L2192 CN**: 声明 TableGen def 记录 `VDOTQ_LANEQ_F16_MF8`。
- **L2193 EN**: Closes the current lexical scope or compound statement.
  **L2193 CN**: 结束当前词法作用域或复合语句块。
- **L2194 EN**: Blank line separating nearby declarations or logic blocks.
  **L2194 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2195 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "defined(__aarch64__)", TargetGuard = "fp8dot4,neon" in {`.
  **L2195 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "defined(__aarch64__)", TargetGuard = "fp8dot4,neon" in {`。
- **L2196 EN**: Declares TableGen def record `VDOT_F32_MF8`.
  **L2196 CN**: 声明 TableGen def 记录 `VDOT_F32_MF8`。
- **L2197 EN**: Blank line separating nearby declarations or logic blocks.
  **L2197 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2198 EN**: Declares TableGen def record `VDOT_LANE_F32_MF8`.
  **L2198 CN**: 声明 TableGen def 记录 `VDOT_LANE_F32_MF8`。
- **L2199 EN**: Declares TableGen def record `VDOT_LANEQ_F32_MF8`.
  **L2199 CN**: 声明 TableGen def 记录 `VDOT_LANEQ_F32_MF8`。
- **L2200 EN**: Blank line separating nearby declarations or logic blocks.
  **L2200 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2201 EN**: Declares TableGen def record `VDOTQ_LANE_F32_MF8`.
  **L2201 CN**: 声明 TableGen def 记录 `VDOTQ_LANE_F32_MF8`。
- **L2202 EN**: Declares TableGen def record `VDOTQ_LANEQ_F32_MF8`.
  **L2202 CN**: 声明 TableGen def 记录 `VDOTQ_LANEQ_F32_MF8`。
- **L2203 EN**: Closes the current lexical scope or compound statement.
  **L2203 CN**: 结束当前词法作用域或复合语句块。
- **L2204 EN**: Blank line separating nearby declarations or logic blocks.
  **L2204 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2205 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "defined(__aarch64__)", TargetGuard = "fp8fma,neon" in {`.
  **L2205 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "defined(__aarch64__)", TargetGuard = "fp8fma,neon" in {`。
- **L2206 EN**: Declares TableGen def record `VMLALB_F16_F8`.
  **L2206 CN**: 声明 TableGen def 记录 `VMLALB_F16_F8`。
- **L2207 EN**: Declares TableGen def record `VMLALT_F16_F8`.
  **L2207 CN**: 声明 TableGen def 记录 `VMLALT_F16_F8`。
- **L2208 EN**: Blank line separating nearby declarations or logic blocks.
  **L2208 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2209 EN**: Declares TableGen def record `VMLALLBB_F32_F8`.
  **L2209 CN**: 声明 TableGen def 记录 `VMLALLBB_F32_F8`。
- **L2210 EN**: Declares TableGen def record `VMLALLBT_F32_F8`.
  **L2210 CN**: 声明 TableGen def 记录 `VMLALLBT_F32_F8`。
- **L2211 EN**: Declares TableGen def record `VMLALLTB_F32_F8`.
  **L2211 CN**: 声明 TableGen def 记录 `VMLALLTB_F32_F8`。
- **L2212 EN**: Declares TableGen def record `VMLALLTT_F32_F8`.
  **L2212 CN**: 声明 TableGen def 记录 `VMLALLTT_F32_F8`。

### Lines 2213-2238

````tablegen

  def VMLALB_F16_F8_LANE  : VInst<"vmlalb_lane_f16_mf8_fpm",  "(>F)(>F).qIV", "Qm", [ImmCheck<3, ImmCheck0_7,  0>]>;
  def VMLALB_F16_F8_LANEQ : VInst<"vmlalb_laneq_f16_mf8_fpm", "(>F)(>F)..IV", "Qm", [ImmCheck<3, ImmCheck0_15, 0>]>;
  def VMLALT_F16_F8_LANE  : VInst<"vmlalt_lane_f16_mf8_fpm",  "(>F)(>F).qIV", "Qm", [ImmCheck<3, ImmCheck0_7,  0>]>;
  def VMLALT_F16_F8_LANEQ : VInst<"vmlalt_laneq_f16_mf8_fpm", "(>F)(>F)..IV", "Qm", [ImmCheck<3, ImmCheck0_15, 0>]>;

  def VMLALLBB_F32_F8_LANE  : VInst<"vmlallbb_lane_f32_mf8_fpm",  "(>>F)(>>F).qIV", "Qm", [ImmCheck<3, ImmCheck0_7,  0>]>;
  def VMLALLBB_F32_F8_LANEQ : VInst<"vmlallbb_laneq_f32_mf8_fpm", "(>>F)(>>F)..IV", "Qm", [ImmCheck<3, ImmCheck0_15, 0>]>;
  def VMLALLBT_F32_F8_LANE  : VInst<"vmlallbt_lane_f32_mf8_fpm",  "(>>F)(>>F).qIV", "Qm", [ImmCheck<3, ImmCheck0_7,  0>]>;
  def VMLALLBT_F32_F8_LANEQ : VInst<"vmlallbt_laneq_f32_mf8_fpm", "(>>F)(>>F)..IV", "Qm", [ImmCheck<3, ImmCheck0_15, 0>]>;
  def VMLALLTB_F32_F8_LANE  : VInst<"vmlalltb_lane_f32_mf8_fpm",  "(>>F)(>>F).qIV", "Qm", [ImmCheck<3, ImmCheck0_7,  0>]>;
  def VMLALLTB_F32_F8_LANEQ : VInst<"vmlalltb_laneq_f32_mf8_fpm", "(>>F)(>>F)..IV", "Qm", [ImmCheck<3, ImmCheck0_15, 0>]>;
  def VMLALLTT_F32_F8_LANE  : VInst<"vmlalltt_lane_f32_mf8_fpm",  "(>>F)(>>F).qIV", "Qm", [ImmCheck<3, ImmCheck0_7,  0>]>;
  def VMLALLTT_F32_F8_LANEQ : VInst<"vmlalltt_laneq_f32_mf8_fpm", "(>>F)(>>F)..IV", "Qm", [ImmCheck<3, ImmCheck0_15, 0>]>;
}

let ArchGuard = "defined(__aarch64__)", TargetGuard = "neon,faminmax" in {
  def FAMIN : WInst<"vamin", "...", "fhQdQfQh">;
  def FAMAX : WInst<"vamax", "...", "fhQdQfQh">;
}

let ArchGuard = "defined(__aarch64__)", TargetGuard = "fp8,neon" in {
  // fscale
  def FSCALE_V128 : WInst<"vscale", "..(.S)", "QdQfQh">;
  def FSCALE_V64 : WInst<"vscale", "(.q)(.q)(.qS)", "fh">;
}
````
- **L2213 EN**: Blank line separating nearby declarations or logic blocks.
  **L2213 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2214 EN**: Declares TableGen def record `VMLALB_F16_F8_LANE`.
  **L2214 CN**: 声明 TableGen def 记录 `VMLALB_F16_F8_LANE`。
- **L2215 EN**: Declares TableGen def record `VMLALB_F16_F8_LANEQ`.
  **L2215 CN**: 声明 TableGen def 记录 `VMLALB_F16_F8_LANEQ`。
- **L2216 EN**: Declares TableGen def record `VMLALT_F16_F8_LANE`.
  **L2216 CN**: 声明 TableGen def 记录 `VMLALT_F16_F8_LANE`。
- **L2217 EN**: Declares TableGen def record `VMLALT_F16_F8_LANEQ`.
  **L2217 CN**: 声明 TableGen def 记录 `VMLALT_F16_F8_LANEQ`。
- **L2218 EN**: Blank line separating nearby declarations or logic blocks.
  **L2218 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2219 EN**: Declares TableGen def record `VMLALLBB_F32_F8_LANE`.
  **L2219 CN**: 声明 TableGen def 记录 `VMLALLBB_F32_F8_LANE`。
- **L2220 EN**: Declares TableGen def record `VMLALLBB_F32_F8_LANEQ`.
  **L2220 CN**: 声明 TableGen def 记录 `VMLALLBB_F32_F8_LANEQ`。
- **L2221 EN**: Declares TableGen def record `VMLALLBT_F32_F8_LANE`.
  **L2221 CN**: 声明 TableGen def 记录 `VMLALLBT_F32_F8_LANE`。
- **L2222 EN**: Declares TableGen def record `VMLALLBT_F32_F8_LANEQ`.
  **L2222 CN**: 声明 TableGen def 记录 `VMLALLBT_F32_F8_LANEQ`。
- **L2223 EN**: Declares TableGen def record `VMLALLTB_F32_F8_LANE`.
  **L2223 CN**: 声明 TableGen def 记录 `VMLALLTB_F32_F8_LANE`。
- **L2224 EN**: Declares TableGen def record `VMLALLTB_F32_F8_LANEQ`.
  **L2224 CN**: 声明 TableGen def 记录 `VMLALLTB_F32_F8_LANEQ`。
- **L2225 EN**: Declares TableGen def record `VMLALLTT_F32_F8_LANE`.
  **L2225 CN**: 声明 TableGen def 记录 `VMLALLTT_F32_F8_LANE`。
- **L2226 EN**: Declares TableGen def record `VMLALLTT_F32_F8_LANEQ`.
  **L2226 CN**: 声明 TableGen def 记录 `VMLALLTT_F32_F8_LANEQ`。
- **L2227 EN**: Closes the current lexical scope or compound statement.
  **L2227 CN**: 结束当前词法作用域或复合语句块。
- **L2228 EN**: Blank line separating nearby declarations or logic blocks.
  **L2228 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2229 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "defined(__aarch64__)", TargetGuard = "neon,faminmax" in {`.
  **L2229 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "defined(__aarch64__)", TargetGuard = "neon,faminmax" in {`。
- **L2230 EN**: Declares TableGen def record `FAMIN`.
  **L2230 CN**: 声明 TableGen def 记录 `FAMIN`。
- **L2231 EN**: Declares TableGen def record `FAMAX`.
  **L2231 CN**: 声明 TableGen def 记录 `FAMAX`。
- **L2232 EN**: Closes the current lexical scope or compound statement.
  **L2232 CN**: 结束当前词法作用域或复合语句块。
- **L2233 EN**: Blank line separating nearby declarations or logic blocks.
  **L2233 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2234 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArchGuard = "defined(__aarch64__)", TargetGuard = "fp8,neon" in {`.
  **L2234 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArchGuard = "defined(__aarch64__)", TargetGuard = "fp8,neon" in {`。
- **L2235 EN**: Comment explains nearby logic, constraints, or intent: `fscale`.
  **L2235 CN**: 注释解释附近代码的逻辑、约束或设计意图：`fscale`。
- **L2236 EN**: Declares TableGen def record `FSCALE_V128`.
  **L2236 CN**: 声明 TableGen def 记录 `FSCALE_V128`。
- **L2237 EN**: Declares TableGen def record `FSCALE_V64`.
  **L2237 CN**: 声明 TableGen def 记录 `FSCALE_V64`。
- **L2238 EN**: Closes the current lexical scope or compound statement.
  **L2238 CN**: 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **Compiler configuration / 编译器配置**
  - **EN**: Holds low-level enums, constants, feature switches, and descriptive metadata used across Clang.
  - **CN**: 保存 Clang 各处复用的底层枚举、常量、特性开关与描述性元数据。
- **Target and language metadata / 目标与语言元数据**
  - **EN**: Models ABI, address-space, builtin, and target-feature information shared by many subsystems.
  - **CN**: 建模多个子系统共享的 ABI、地址空间、builtin 与目标特性信息。
- **Generated definition tables / 生成式定义表**
  - **EN**: Many Basic headers use `.td` or `.def` files to describe families of diagnostics, attributes, or builtins declaratively.
  - **CN**: 许多 Basic 头文件使用 `.td` 或 `.def` 文件以声明式方式描述诊断、属性或 builtin 家族。
- **TableGen declarative modeling / TableGen 声明式建模**
  - **EN**: Uses TableGen records, inheritance, and generators to describe families of compiler entities compactly.
  - **CN**: 使用 TableGen 记录、继承与生成器紧凑地描述一组编译器实体。
- **Builtin descriptions / Builtin 描述**
  - **EN**: Describes compiler builtins, their signatures, properties, or target-specific availability.
  - **CN**: 描述编译器 builtin 的签名、属性或目标相关可用性。
- **Type-system support / 类型系统支撑**
  - **EN**: Provides metadata or helpers used to model Clang types and related dispatch logic.
  - **CN**: 提供建模 Clang 类型及相关分发逻辑所需的元数据或辅助工具。
- **Indexing and tooling / 索引与工具支持**
  - **EN**: Exposes APIs used by source indexing, diagnostics retrieval, and IDE tooling.
  - **CN**: 暴露源码索引、诊断提取与 IDE 工具使用的 API。
- **AArch64 target support / AArch64 目标支持**
  - **EN**: Encodes AArch64-specific compiler metadata or builtin descriptions.
  - **CN**: 编码 AArch64 专用的编译器元数据或 builtin 描述。
- **Arm target support / Arm 目标支持**
  - **EN**: Encodes Arm-specific language extensions, intrinsics, or target metadata.
  - **CN**: 编码 Arm 专用语言扩展、intrinsic 或目标元数据。
- **Macro-driven generation / 宏驱动生成**
  - **EN**: Relies on macros or generated include fragments to keep large definition sets synchronized.
  - **CN**: 依赖宏或生成式 include 片段来同步大规模定义集合。

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: None detected. / 未检测到。
- **Macros / 宏**: `GET_LANE`
- **Types / 类型**: `ScalarMulOp`, `REINTERPRET_CROSS_SELF`, `using`
- **Functions or callables / 函数或可调用对象**: `Op<`, `decimate`, `call`, `sizeinbits`, `type`, `of`, `vget_high_f16`, `vget_low_f16`, `F`, `defined`, `U`, `Shift`
- **TableGen records / TableGen 记录**: `OP_ADD`, `OP_ADDL`, `OP_ADDLHi`, `OP_ADDW`, `OP_ADDWHi`, `OP_SUB`, `OP_SUBL`, `OP_SUBLHi`, `OP_SUBW`, `OP_SUBWHi`, `OP_MUL`, `OP_MLA`, `OP_MLAL`, `OP_MULLHi`, `OP_MULLHi_P64`, `OP_MULLHi_N`
- **Namespaces / 命名空间**: No explicit namespaces detected. / 未检测到显式命名空间。
