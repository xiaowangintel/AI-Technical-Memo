# HexagonScheduleV5.td — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonScheduleV5.td`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Defines Hexagon per-architecture scheduling model data using TableGen DSL.
- Purpose / 作用 (CN): 该文件使用 TableGen DSL 定义 Hexagon 目标相关的声明式记录。
- Note / 说明: This file is written in LLVM TableGen DSL, so many lines are declarative records consumed by TableGen generators. / 该文件使用 LLVM TableGen DSL 编写，因此很多行是由 TableGen 生成器消费的声明式记录。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25 / 第 1-25 行

```tablegen
     1: //=-HexagonScheduleV5.td - HexagonV5 Scheduling Definitions --*- tablegen -*-=//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: 
     9: def LD_tc_ld_SLOT01 : InstrItinClass;
    10: def ST_tc_st_SLOT01 : InstrItinClass;
    11: 
    12: class HexagonV5PseudoItin {
    13:   list<InstrItinData> V5PseudoItin_list = [
    14:     InstrItinData<PSEUDO,     [InstrStage<1, [SLOT0, SLOT1, SLOT2, SLOT3]>]>,
    15:     InstrItinData<PSEUDOM,    [InstrStage<1, [SLOT2, SLOT3], 0>,
    16:                                InstrStage<1, [SLOT2, SLOT3]>]>,
    17:     InstrItinData<DUPLEX,     [InstrStage<1, [SLOT0]>]>,
    18:     InstrItinData<tc_ENDLOOP, [InstrStage<1, [SLOT_ENDLOOP]>]>
    19:   ];
    20: }
    21: 
    22: def HexagonV5ItinList : DepScalarItinV5, HexagonV5PseudoItin {
    23:   list<InstrItinData> V5Itin_list = [
    24:     InstrItinData<LD_tc_ld_SLOT01, [InstrStage<1, [SLOT0, SLOT1]>]>,
    25:     InstrItinData<ST_tc_st_SLOT01, [InstrStage<1, [SLOT0, SLOT1]>]>
```
- EN: It declares types such as HexagonV5PseudoItin, which carry the state or API of this component. It defines declarative TableGen records like LD_tc_ld_SLOT01, ST_tc_st_SLOT01, HexagonV5PseudoItin, HexagonV5ItinList; these records are consumed by TableGen instead of executed directly. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonScheduleV5, HexagonV5, HexagonV5PseudoItin, HexagonV5ItinList, showing how the code connects to sibling backend components.
- CN: 这里声明了 HexagonV5PseudoItin 等类型，用来承载该组件的状态或接口。 这里定义了 LD_tc_ld_SLOT01, ST_tc_st_SLOT01, HexagonV5PseudoItin, HexagonV5ItinList 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonScheduleV5, HexagonV5, HexagonV5PseudoItin, HexagonV5ItinList，说明了它与同级后端组件的连接关系。

### Lines 26-45 / 第 26-45 行

```tablegen
    26:   ];
    27:   list<InstrItinData> ItinList =
    28:     !listconcat(V5Itin_list, DepScalarItinV5_list, V5PseudoItin_list);
    29: }
    30: 
    31: def HexagonItinerariesV5 :
    32:       ProcessorItineraries<[SLOT0, SLOT1, SLOT2, SLOT3, SLOT_ENDLOOP],
    33:                            [Hex_FWD], HexagonV5ItinList.ItinList>;
    34: 
    35: def HexagonModelV5 : SchedMachineModel {
    36:   // Max issue per cycle == bundle width.
    37:   let IssueWidth = 4;
    38:   let Itineraries = HexagonItinerariesV5;
    39:   let LoadLatency = 1;
    40:   let CompleteModel = 0;
    41: }
    42: 
    43: //===----------------------------------------------------------------------===//
    44: // Hexagon V5 Resource Definitions -
    45: //===----------------------------------------------------------------------===//
```
- EN: It defines declarative TableGen records like HexagonItinerariesV5, HexagonModelV5; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as listconcat, translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonItinerariesV5, HexagonV5ItinList, HexagonModelV5, showing how the code connects to sibling backend components.
- CN: 这里定义了 HexagonItinerariesV5, HexagonModelV5 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 listconcat 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonItinerariesV5, HexagonV5ItinList, HexagonModelV5，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- instruction scheduling models / 指令调度模型
- instruction semantics / 指令语义
- generated metadata tables / 生成的元数据表
- TableGen DSL records / TableGen DSL 记录

## Dependencies / 依赖关系

- Hexagon symbols / Hexagon 符号: `HexagonScheduleV5, HexagonV5, HexagonV5PseudoItin, HexagonV5ItinList, HexagonItinerariesV5, HexagonModelV5`
- TableGen pipeline / TableGen 流程: consumed by LLVM TableGen to generate target metadata and `*.inc` tables for the Hexagon backend. / 由 LLVM TableGen 消费，生成 Hexagon 后端所需的目标元数据和 `*.inc` 表。
