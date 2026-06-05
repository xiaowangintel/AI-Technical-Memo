# HexagonScheduleV71.td — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonScheduleV71.td`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): This file is distributed under the University of Illinois Open Source
- Purpose / 作用 (CN): 该文件使用 TableGen DSL 定义 Hexagon 目标相关的声明式记录。
- Note / 说明: This file is written in LLVM TableGen DSL, so many lines are declarative records consumed by TableGen generators. / 该文件使用 LLVM TableGen DSL 编写，因此很多行是由 TableGen 生成器消费的声明式记录。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25 / 第 1-25 行

```tablegen
     1: //=-HexagonScheduleV71.td - HexagonV71 Scheduling Definitions *- tablegen -*-=//
     2: //
     3: //                     The LLVM Compiler Infrastructure
     4: //
     5: // This file is distributed under the University of Illinois Open Source
     6: // License. See LICENSE.TXT for details.
     7: //
     8: //===----------------------------------------------------------------------===//
     9: 
    10: //
    11: // ScalarItin and HVXItin contain some old itineraries still used by a handful
    12: // of instructions. Hopefully, we will be able to get rid of them soon.
    13: def HexagonV71ItinList : DepScalarItinV71, ScalarItin,
    14:                          DepHVXItinV71, HVXItin, PseudoItin {
    15:   list<InstrItinData> ItinList =
    16:     !listconcat(DepScalarItinV71_list, ScalarItin_list,
    17:                 DepHVXItinV71_list, HVXItin_list, PseudoItin_list);
    18: }
    19: 
    20: def HexagonItinerariesV71 :
    21:       ProcessorItineraries<[SLOT0, SLOT1, SLOT2, SLOT3, SLOT_ENDLOOP,
    22:                             CVI_ST, CVI_XLANE, CVI_SHIFT, CVI_MPY0, CVI_MPY1,
    23:                             CVI_LD, CVI_XLSHF, CVI_MPY01, CVI_ALL,
    24:                             CVI_ALL_NOMEM, CVI_ZW],
    25:                             [Hex_FWD, HVX_FWD],
```
- EN: It defines declarative TableGen records like HexagonV71ItinList, HexagonItinerariesV71; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as listconcat, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonScheduleV71, HexagonV71, HexagonV71ItinList, HexagonItinerariesV71, showing how the code connects to sibling backend components.
- CN: 这里定义了 HexagonV71ItinList, HexagonItinerariesV71 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 listconcat 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonScheduleV71, HexagonV71, HexagonV71ItinList, HexagonItinerariesV71，说明了它与同级后端组件的连接关系。

### Lines 26-39 / 第 26-39 行

```tablegen
    26:                             HexagonV71ItinList.ItinList>;
    27: 
    28: def HexagonModelV71 : SchedMachineModel {
    29:   // Max issue per cycle == bundle width.
    30:   let IssueWidth = 4;
    31:   let Itineraries = HexagonItinerariesV71;
    32:   let LoadLatency = 1;
    33:   let CompleteModel = 0;
    34: }
    35: 
    36: //===----------------------------------------------------------------------===//
    37: // Hexagon V71 Resource Definitions -
    38: //===----------------------------------------------------------------------===//
    39: 
```
- EN: It defines declarative TableGen records like HexagonModelV71; these records are consumed by TableGen instead of executed directly. Notable Hexagon symbols referenced here include HexagonV71ItinList, HexagonModelV71, HexagonItinerariesV71, showing how the code connects to sibling backend components.
- CN: 这里定义了 HexagonModelV71 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里引用的重要 Hexagon 符号包括 HexagonV71ItinList, HexagonModelV71, HexagonItinerariesV71，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- HVX vector ISA modeling / HVX 向量 ISA 建模
- instruction scheduling models / 指令调度模型
- instruction semantics / 指令语义
- generated metadata tables / 生成的元数据表
- TableGen DSL records / TableGen DSL 记录

## Dependencies / 依赖关系

- Hexagon symbols / Hexagon 符号: `HexagonScheduleV71, HexagonV71, HexagonV71ItinList, HexagonItinerariesV71, HexagonModelV71`
- TableGen pipeline / TableGen 流程: consumed by LLVM TableGen to generate target metadata and `*.inc` tables for the Hexagon backend. / 由 LLVM TableGen 消费，生成 Hexagon 后端所需的目标元数据和 `*.inc` 表。
