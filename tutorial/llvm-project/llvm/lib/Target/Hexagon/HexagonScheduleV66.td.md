# HexagonScheduleV66.td — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonScheduleV66.td`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Defines Hexagon per-architecture scheduling model data using TableGen DSL.
- Purpose / 作用 (CN): 该文件使用 TableGen DSL 定义 Hexagon 目标相关的声明式记录。
- Note / 说明: This file is written in LLVM TableGen DSL, so many lines are declarative records consumed by TableGen generators. / 该文件使用 LLVM TableGen DSL 编写，因此很多行是由 TableGen 生成器消费的声明式记录。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25 / 第 1-25 行

```tablegen
     1: //=-HexagonScheduleV66.td - HexagonV66 Scheduling Definitions *- tablegen -*-=//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: 
     9: //
    10: // ScalarItin and HVXItin contain some old itineraries
    11: // still used by a handful of instructions. Hopefully, we will be able
    12: // to get rid of them soon.
    13: 
    14: def HexagonV66ItinList : DepScalarItinV66, ScalarItin,
    15:                          DepHVXItinV66, HVXItin, PseudoItin {
    16:   list<InstrItinData> ItinList =
    17:     !listconcat(DepScalarItinV66_list, ScalarItin_list,
    18:                 DepHVXItinV66_list, HVXItin_list, PseudoItin_list);
    19: }
    20: 
    21: def HexagonItinerariesV66 :
    22:       ProcessorItineraries<[SLOT0, SLOT1, SLOT2, SLOT3, SLOT_ENDLOOP,
    23:                             CVI_ST, CVI_XLANE, CVI_SHIFT, CVI_MPY0, CVI_MPY1,
    24:                             CVI_LD, CVI_XLSHF, CVI_MPY01, CVI_ALL,
    25:                             CVI_ALL_NOMEM, CVI_ZW],
```
- EN: It defines declarative TableGen records like HexagonV66ItinList, HexagonItinerariesV66; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as listconcat, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonScheduleV66, HexagonV66, HexagonV66ItinList, HexagonItinerariesV66, showing how the code connects to sibling backend components.
- CN: 这里定义了 HexagonV66ItinList, HexagonItinerariesV66 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 listconcat 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonScheduleV66, HexagonV66, HexagonV66ItinList, HexagonItinerariesV66，说明了它与同级后端组件的连接关系。

### Lines 26-40 / 第 26-40 行

```tablegen
    26:                             [Hex_FWD, HVX_FWD],
    27:                             HexagonV66ItinList.ItinList>;
    28: 
    29: def HexagonModelV66 : SchedMachineModel {
    30:   // Max issue per cycle == bundle width.
    31:   let IssueWidth = 4;
    32:   let Itineraries = HexagonItinerariesV66;
    33:   let LoadLatency = 1;
    34:   let CompleteModel = 0;
    35: }
    36: 
    37: //===----------------------------------------------------------------------===//
    38: // Hexagon V66 Resource Definitions -
    39: //===----------------------------------------------------------------------===//
    40: 
```
- EN: It defines declarative TableGen records like HexagonModelV66; these records are consumed by TableGen instead of executed directly. Notable Hexagon symbols referenced here include HexagonV66ItinList, HexagonModelV66, HexagonItinerariesV66, showing how the code connects to sibling backend components.
- CN: 这里定义了 HexagonModelV66 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里引用的重要 Hexagon 符号包括 HexagonV66ItinList, HexagonModelV66, HexagonItinerariesV66，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- HVX vector ISA modeling / HVX 向量 ISA 建模
- instruction scheduling models / 指令调度模型
- instruction semantics / 指令语义
- generated metadata tables / 生成的元数据表
- TableGen DSL records / TableGen DSL 记录

## Dependencies / 依赖关系

- Hexagon symbols / Hexagon 符号: `HexagonScheduleV66, HexagonV66, HexagonV66ItinList, HexagonItinerariesV66, HexagonModelV66`
- TableGen pipeline / TableGen 流程: consumed by LLVM TableGen to generate target metadata and `*.inc` tables for the Hexagon backend. / 由 LLVM TableGen 消费，生成 Hexagon 后端所需的目标元数据和 `*.inc` 表。
