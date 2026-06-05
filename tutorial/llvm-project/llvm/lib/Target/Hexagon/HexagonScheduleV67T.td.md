# HexagonScheduleV67T.td — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonScheduleV67T.td`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Defines Hexagon per-architecture scheduling model data using TableGen DSL.
- Purpose / 作用 (CN): 该文件使用 TableGen DSL 定义 Hexagon 目标相关的声明式记录。
- Note / 说明: This file is written in LLVM TableGen DSL, so many lines are declarative records consumed by TableGen generators. / 该文件使用 LLVM TableGen DSL 编写，因此很多行是由 TableGen 生成器消费的声明式记录。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25 / 第 1-25 行

```tablegen
     1: //=- HexagonScheduleV67T.td - Hexagon V67 Tiny Core Scheduling Definitions --=//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: 
     9: class HexagonV67TPseudoItin {
    10:   list<InstrItinData> V67TPseudoItin_list = [
    11:     InstrItinData<PSEUDO, [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [2, 1, 1],
    12:                           [Hex_FWD, Hex_FWD, Hex_FWD]>,
    13:     InstrItinData<PSEUDOM, [InstrStage<1, [SLOT2, SLOT3], 0>,
    14:                             InstrStage<1, [SLOT2, SLOT3]>],
    15:                            [2, 1, 1],
    16:                            [Hex_FWD, Hex_FWD, Hex_FWD]>,
    17:     InstrItinData<DUPLEX, [InstrStage<1, [SLOT0]>],
    18:                           [2, 1, 1]>,
    19:     InstrItinData<tc_ENDLOOP, [InstrStage<1, [SLOT_ENDLOOP]>], [2]>
    20:   ];
    21: }
    22: 
    23: // V67TItin_list and HVXItin contain some old itineraries
    24: // still used by a handful of instructions. Hopefully, we will be able to
    25: // get rid of them soon.
```
- EN: It declares types such as HexagonV67TPseudoItin, which carry the state or API of this component. It defines declarative TableGen records like HexagonV67TPseudoItin; these records are consumed by TableGen instead of executed directly. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonScheduleV67T, HexagonV67TPseudoItin, showing how the code connects to sibling backend components.
- CN: 这里声明了 HexagonV67TPseudoItin 等类型，用来承载该组件的状态或接口。 这里定义了 HexagonV67TPseudoItin 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonScheduleV67T, HexagonV67TPseudoItin，说明了它与同级后端组件的连接关系。

### Lines 26-50 / 第 26-50 行

```tablegen
    26: def HexagonV67TItinList : DepScalarItinV67T,
    27:                           DepHVXItinV67, HVXItin, HexagonV67TPseudoItin {
    28:   list<InstrItinData> V67TItin_list = [
    29:     InstrItinData<LD_tc_ld_SLOT01, [InstrStage<1, [SLOT0]>],
    30:                                    [3, 1, 1],
    31:                                    [Hex_FWD, Hex_FWD, Hex_FWD]>,
    32:     InstrItinData<ST_tc_st_SLOT01, [InstrStage<1, [SLOT0]>],
    33:                                    [1, 1, 3, 3],
    34:                                    [Hex_FWD, Hex_FWD]>
    35:   ];
    36: 
    37:   list<InstrItinData> ItinList =
    38:     !listconcat(DepScalarItinV67T_list,
    39:                 DepHVXItinV67_list, V67TItin_list,
    40:                 HVXItin_list, V67TPseudoItin_list);
    41: }
    42: 
    43: def HexagonItinerariesV67T :
    44:       ProcessorItineraries<[SLOT0, SLOT1, SLOT2, SLOT3, SLOT_ENDLOOP,
    45:                             CVI_ST, CVI_XLANE, CVI_SHIFT, CVI_MPY0, CVI_MPY1,
    46:                             CVI_LD, CVI_XLSHF, CVI_MPY01, CVI_ALL,
    47:                             CVI_ALL_NOMEM, CVI_ZW],
    48:                             [Hex_FWD, HVX_FWD],
    49:                             HexagonV67TItinList.ItinList>;
    50: 
```
- EN: It defines declarative TableGen records like HexagonV67TItinList, HexagonItinerariesV67T; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as listconcat, translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonV67TItinList, HexagonV67TPseudoItin, HexagonItinerariesV67T, showing how the code connects to sibling backend components.
- CN: 这里定义了 HexagonV67TItinList, HexagonItinerariesV67T 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 listconcat 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonV67TItinList, HexagonV67TPseudoItin, HexagonItinerariesV67T，说明了它与同级后端组件的连接关系。

### Lines 51-61 / 第 51-61 行

```tablegen
    51: 
    52: def HexagonModelV67T : SchedMachineModel {
    53:   let IssueWidth = 3;
    54:   let Itineraries = HexagonItinerariesV67T;
    55:   let LoadLatency = 1;
    56:   let CompleteModel = 0;
    57: }
    58: 
    59: //===----------------------------------------------------------------------===//
    60: // Hexagon V67 Tiny Core Resource Definitions -
    61: //===----------------------------------------------------------------------===//
```
- EN: It defines declarative TableGen records like HexagonModelV67T; these records are consumed by TableGen instead of executed directly. Notable Hexagon symbols referenced here include HexagonModelV67T, HexagonItinerariesV67T, showing how the code connects to sibling backend components.
- CN: 这里定义了 HexagonModelV67T 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里引用的重要 Hexagon 符号包括 HexagonModelV67T, HexagonItinerariesV67T，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- HVX vector ISA modeling / HVX 向量 ISA 建模
- instruction scheduling models / 指令调度模型
- instruction semantics / 指令语义
- generated metadata tables / 生成的元数据表
- TableGen DSL records / TableGen DSL 记录

## Dependencies / 依赖关系

- Hexagon symbols / Hexagon 符号: `HexagonScheduleV67T, HexagonV67TPseudoItin, HexagonV67TItinList, HexagonItinerariesV67T, HexagonModelV67T`
- TableGen pipeline / TableGen 流程: consumed by LLVM TableGen to generate target metadata and `*.inc` tables for the Hexagon backend. / 由 LLVM TableGen 消费，生成 Hexagon 后端所需的目标元数据和 `*.inc` 表。
