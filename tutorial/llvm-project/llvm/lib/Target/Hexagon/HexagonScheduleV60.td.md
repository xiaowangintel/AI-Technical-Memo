# HexagonScheduleV60.td — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonScheduleV60.td`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): This file describes that machine information.
- Purpose / 作用 (CN): 该文件使用 TableGen DSL 定义 Hexagon 目标相关的声明式记录。
- Note / 说明: This file is written in LLVM TableGen DSL, so many lines are declarative records consumed by TableGen generators. / 该文件使用 LLVM TableGen DSL 编写，因此很多行是由 TableGen 生成器消费的声明式记录。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25 / 第 1-25 行

```tablegen
     1: //=-HexagonScheduleV60.td - HexagonV60 Scheduling Definitions *- tablegen -*-=//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: 
     9: 
    10: // There are four SLOTS (four parallel pipelines) in Hexagon V60 machine.
    11: // This file describes that machine information.
    12: //
    13: //    |===========|==================================================|
    14: //    | PIPELINE  |              Instruction Classes                 |
    15: //    |===========|==================================================|
    16: //    | SLOT0     |  LD       ST    ALU32     MEMOP     NV    SYSTEM |
    17: //    |-----------|--------------------------------------------------|
    18: //    | SLOT1     |  LD       ST    ALU32                            |
    19: //    |-----------|--------------------------------------------------|
    20: //    | SLOT2     |  XTYPE          ALU32     J         JR           |
    21: //    |-----------|--------------------------------------------------|
    22: //    | SLOT3     |  XTYPE          ALU32     J         CR           |
    23: //    |===========|==================================================|
    24: //
    25: //
```
- EN: This range is dominated by comments or banner text that documents the surrounding section. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonScheduleV60, HexagonV60, showing how the code connects to sibling backend components.
- CN: 这一段主要由注释或分隔说明组成，用来解释后续代码区域。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonScheduleV60, HexagonV60，说明了它与同级后端组件的连接关系。

### Lines 26-50 / 第 26-50 行

```tablegen
    26: // In addition to using the above SLOTS, there are also six vector pipelines
    27: // in the CVI co-processor in the Hexagon V60 machine.
    28: //
    29: //      |=========| |=========| |=========| |=========| |=========| |=========|
    30: // SLOT | CVI_LD  | |CVI_MPY3 | |CVI_MPY2 | |CVI_SHIFT| |CVI_XLANE| | CVI_ST  |
    31: // ==== |=========| |=========| |=========| |=========| |=========| |=========|
    32: // S0-3 |         | | CVI_VA  | | CVI_VA  | | CVI_VA  | | CVI_VA  | |         |
    33: // S2-3 |         | | CVI_VX  | | CVI_VX  | |         | |         | |         |
    34: // S0-3 |         | |         | |         | |         | | CVI_VP  | |         |
    35: // S0-3 |         | |         | |         | | CVI_VS  | |         | |         |
    36: // S0-1 |(CVI_LD) | | CVI_LD  | | CVI_LD  | | CVI_LD  | | CVI_LD  | |         |
    37: // S0-1 |(C*TMP_LD) |         | |         | |         | |         | |         |
    38: // S01  |(C*_LDU) | |         | |         | |         | | C*_LDU  | |         |
    39: // S0   |         | | CVI_ST  | | CVI_ST  | | CVI_ST  | | CVI_ST  | |(CVI_ST) |
    40: // S0   |         | |         | |         | |         | |         | |(C*TMP_ST)
    41: // S01  |         | |         | |         | |         | | VSTU    | |(C*_STU) |
    42: //      |=========| |=========| |=========| |=========| |=========| |=========|
    43: //                  |=====================| |=====================|
    44: //                  | CVI_MPY2 & CVI_MPY3 | |CVI_XLANE & CVI_SHIFT|
    45: //                  |=====================| |=====================|
    46: // S0-3             | CVI_VA_DV           | | CVI_VA_DV           |
    47: // S0-3             |                     | | CVI_VP_DV           |
    48: // S2-3             | CVI_VX_DV           | |                     |
    49: //                  |=====================| |=====================|
    50: //      |=====================================================================|
```
- EN: This range is dominated by comments or banner text that documents the surrounding section.
- CN: 这一段主要由注释或分隔说明组成，用来解释后续代码区域。

### Lines 51-75 / 第 51-75 行

```tablegen
    51: // S0-3 | CVI_HIST   Histogram                                                |
    52: // S0123| CVI_VA_EXT Extract                                                  |
    53: //      |=====================================================================|
    54: 
    55: def HexagonV60ItinList : DepScalarItinV60, ScalarItin,
    56:                          DepHVXItinV60,
    57:                          HVXItin, PseudoItin {
    58:   list<InstrItinData> ItinList =
    59:     !listconcat(DepScalarItinV60_list, ScalarItin_list,
    60:                 DepHVXItinV60_list, HVXItin_list, PseudoItin_list);
    61: }
    62: 
    63: def HexagonItinerariesV60 :
    64:       ProcessorItineraries<[SLOT0, SLOT1, SLOT2, SLOT3, SLOT_ENDLOOP,
    65:                             CVI_ST, CVI_XLANE, CVI_SHIFT, CVI_MPY0, CVI_MPY1,
    66:                             CVI_LD, CVI_XLSHF, CVI_MPY01, CVI_ALL,
    67:                             CVI_ALL_NOMEM, CVI_ZW],
    68:                             [Hex_FWD, HVX_FWD], HexagonV60ItinList.ItinList>;
    69: 
    70: def HexagonModelV60 : SchedMachineModel {
    71:   // Max issue per cycle == bundle width.
    72:   let IssueWidth = 4;
    73:   let Itineraries = HexagonItinerariesV60;
    74:   let LoadLatency = 1;
    75:   let CompleteModel = 0;
```
- EN: It defines declarative TableGen records like HexagonV60ItinList, HexagonItinerariesV60, HexagonModelV60; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as listconcat, translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonV60ItinList, HexagonItinerariesV60, HexagonModelV60, showing how the code connects to sibling backend components.
- CN: 这里定义了 HexagonV60ItinList, HexagonItinerariesV60, HexagonModelV60 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 listconcat 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonV60ItinList, HexagonItinerariesV60, HexagonModelV60，说明了它与同级后端组件的连接关系。

### Lines 76-80 / 第 76-80 行

```tablegen
    76: }
    77: 
    78: //===----------------------------------------------------------------------===//
    79: // Hexagon V60 Resource Definitions -
    80: //===----------------------------------------------------------------------===//
```
- EN: This range is dominated by comments or banner text that documents the surrounding section.
- CN: 这一段主要由注释或分隔说明组成，用来解释后续代码区域。

## Key Concepts / 关键概念

- HVX vector ISA modeling / HVX 向量 ISA 建模
- instruction scheduling models / 指令调度模型
- instruction semantics / 指令语义
- generated metadata tables / 生成的元数据表
- TableGen DSL records / TableGen DSL 记录

## Dependencies / 依赖关系

- Hexagon symbols / Hexagon 符号: `HexagonScheduleV60, HexagonV60, HexagonV60ItinList, HexagonItinerariesV60, HexagonModelV60`
- TableGen pipeline / TableGen 流程: consumed by LLVM TableGen to generate target metadata and `*.inc` tables for the Hexagon backend. / 由 LLVM TableGen 消费，生成 Hexagon 后端所需的目标元数据和 `*.inc` 表。
