# HexagonPatternsV65.td — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonPatternsV65.td`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Defines Hexagon V65-specific instruction-selection patterns using TableGen DSL.
- Purpose / 作用 (CN): 该文件使用 TableGen DSL 定义 Hexagon 目标相关的声明式记录。 重点涉及指令语义与选择。
- Note / 说明: This file is written in LLVM TableGen DSL, so many lines are declarative records consumed by TableGen generators. / 该文件使用 LLVM TableGen DSL 编写，因此很多行是由 TableGen 生成器消费的声明式记录。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25 / 第 1-25 行

```tablegen
     1: //==- HexagonPatternsV65.td -------------------------------*- tablegen -*-===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: 
     9: multiclass vgathermh<RegisterClass RC> {
    10:   let isCodeGenOnly = 1, isPseudo = 1, mayLoad = 1,
    11:   mayStore = 1, addrMode = BaseImmOffset, accessSize = HalfWordAccess in
    12:   def NAME : CVI_GATHER_TMP_LD_Resource_NoOpcode<(outs ),
    13:                            (ins IntRegs:$_dst_, s4_0Imm:$Ii,
    14:                                 IntRegs:$Rt, ModRegs:$Mu, RC:$Vv),
    15:                            ".error \"should not emit\" ",
    16:                            []>;
    17: }
    18: 
    19: multiclass vgathermw<RegisterClass RC> {
    20:   let isCodeGenOnly = 1, isPseudo = 1, mayLoad = 1,
    21:   mayStore = 1, addrMode = BaseImmOffset, accessSize = WordAccess in
    22:   def NAME : CVI_GATHER_TMP_LD_Resource_NoOpcode<(outs ),
    23:                            (ins IntRegs:$_dst_, s4_0Imm:$Ii,
    24:                                 IntRegs:$Rt, ModRegs:$Mu, RC:$Vv),
    25:                            ".error \"should not emit\" ",
```
- EN: It defines declarative TableGen records like vgathermh, NAME, vgathermw; these records are consumed by TableGen instead of executed directly. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonPatternsV65, showing how the code connects to sibling backend components.
- CN: 这里定义了 vgathermh, NAME, vgathermw 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonPatternsV65，说明了它与同级后端组件的连接关系。

### Lines 26-50 / 第 26-50 行

```tablegen
    26:                            []>;
    27: }
    28: 
    29: multiclass vgathermhw<RegisterClass RC> {
    30:   let isCodeGenOnly = 1, isPseudo = 1, mayLoad = 1,
    31:    mayStore = 1, addrMode = BaseImmOffset, accessSize = HalfWordAccess in
    32:   def NAME : CVI_GATHER_TMP_LD_Resource_NoOpcode<(outs ),
    33:                            (ins IntRegs:$_dst_, s4_0Imm:$Ii,
    34:                                 IntRegs:$Rt, ModRegs:$Mu, RC:$Vv),
    35:                            ".error \"should not emit\" ",
    36:                            []>;
    37: }
    38: 
    39: defm V6_vgathermh_pseudo  : vgathermh<HvxVR>;
    40: defm V6_vgathermw_pseudo  : vgathermw<HvxVR>;
    41: defm V6_vgathermhw_pseudo  : vgathermhw<HvxWR>;
    42: 
    43: 
    44: multiclass vgather_scatter_mh<RegisterClass RC> {
    45:   let isCodeGenOnly = 1, isPseudo = 1, mayLoad = 1,
    46:   mayStore = 1, addrMode = BaseImmOffset, accessSize = HalfWordAccess in
    47:   def NAME : CVI_GATHER_TMP_LD_Resource_NoOpcode<(outs ),
    48:                            (ins IntRegs:$_dst_, s4_0Imm:$Ii,
    49:                                 IntRegs:$Rt, ModRegs:$Mu, RC:$Vv),
    50:                            ".error \"should not emit\" ",
```
- EN: It defines declarative TableGen records like vgathermhw, NAME, V6_vgathermh_pseudo, V6_vgathermw_pseudo, V6_vgathermhw_pseudo, ... (6 total); these records are consumed by TableGen instead of executed directly.
- CN: 这里定义了 vgathermhw, NAME, V6_vgathermh_pseudo, V6_vgathermw_pseudo, V6_vgathermhw_pseudo, ... (6 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 51-75 / 第 51-75 行

```tablegen
    51:                            []>;
    52: }
    53: 
    54: defm V6_vgather_vscatter_mh_pseudo  : vgather_scatter_mh<HvxVR>;
    55: 
    56: multiclass vgathermhq<RegisterClass RC1, RegisterClass RC2> {
    57:   let isCodeGenOnly = 1, isPseudo = 1, mayLoad = 1,
    58:   mayStore = 1, addrMode = BaseImmOffset, accessSize = HalfWordAccess in
    59:   def NAME : CVI_GATHER_TMP_LD_Resource_NoOpcode<(outs ),
    60:                            (ins IntRegs:$_dst_, s4_0Imm:$Ii,
    61:                                 RC2:$Vq, IntRegs:$Rt, ModRegs:$Mu,
    62:                                 RC1:$Vv),
    63:                            ".error \"should not emit\" ",
    64:                            []>;
    65: }
    66: 
    67: multiclass vgathermwq<RegisterClass RC1, RegisterClass RC2> {
    68:   let isCodeGenOnly = 1, isPseudo = 1, mayLoad = 1,
    69:   mayStore = 1, addrMode = BaseImmOffset, accessSize = WordAccess in
    70:   def NAME : CVI_GATHER_TMP_LD_Resource_NoOpcode<(outs ),
    71:                            (ins IntRegs:$_dst_, s4_0Imm:$Ii,
    72:                                 RC2:$Vq, IntRegs:$Rt, ModRegs:$Mu,
    73:                                 RC1:$Vv),
    74:                            ".error \"should not emit\" ",
    75:                            []>;
```
- EN: It defines declarative TableGen records like V6_vgather_vscatter_mh_pseudo, vgathermhq, NAME, vgathermwq; these records are consumed by TableGen instead of executed directly.
- CN: 这里定义了 V6_vgather_vscatter_mh_pseudo, vgathermhq, NAME, vgathermwq 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

### Lines 76-91 / 第 76-91 行

```tablegen
    76: }
    77: 
    78: multiclass vgathermhwq<RegisterClass RC1, RegisterClass RC2> {
    79:   let isCodeGenOnly = 1, isPseudo = 1, mayLoad = 1,
    80:   mayStore = 1, addrMode = BaseImmOffset, accessSize = HalfWordAccess  in
    81:   def NAME : CVI_GATHER_TMP_LD_Resource_NoOpcode<(outs ),
    82:                            (ins IntRegs:$_dst_, s4_0Imm:$Ii,
    83:                               RC2:$Vq, IntRegs:$Rt, ModRegs:$Mu,
    84:                               RC1:$Vv),
    85:                            ".error \"should not emit\" ",
    86:                            []>;
    87: }
    88: 
    89: defm V6_vgathermhq_pseudo  : vgathermhq<HvxVR, HvxQR>;
    90: defm V6_vgathermwq_pseudo  : vgathermwq<HvxVR, HvxQR>;
    91: defm V6_vgathermhwq_pseudo  : vgathermhwq<HvxWR, HvxQR>;
```
- EN: It defines declarative TableGen records like vgathermhwq, NAME, V6_vgathermhq_pseudo, V6_vgathermwq_pseudo, V6_vgathermhwq_pseudo; these records are consumed by TableGen instead of executed directly.
- CN: 这里定义了 vgathermhwq, NAME, V6_vgathermhq_pseudo, V6_vgathermwq_pseudo, V6_vgathermhwq_pseudo 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- pattern-driven instruction selection / 基于模式的指令选择
- TableGen DSL records / TableGen DSL 记录

## Dependencies / 依赖关系

- Hexagon symbols / Hexagon 符号: `HexagonPatternsV65`
- TableGen pipeline / TableGen 流程: consumed by LLVM TableGen to generate target metadata and `*.inc` tables for the Hexagon backend. / 由 LLVM TableGen 消费，生成 Hexagon 后端所需的目标元数据和 `*.inc` 表。
