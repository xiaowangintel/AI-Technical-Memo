# HexagonScheduleV71T.td — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonScheduleV71T.td`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): This file is distributed under the University of Illinois Open Source
- Purpose / 作用 (CN): 该文件使用 TableGen DSL 定义 Hexagon 目标相关的声明式记录。
- Note / 说明: This file is written in LLVM TableGen DSL, so many lines are declarative records consumed by TableGen generators. / 该文件使用 LLVM TableGen DSL 编写，因此很多行是由 TableGen 生成器消费的声明式记录。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25 / 第 1-25 行

```tablegen
     1: //=-HexagonScheduleV71T.td - Hexagon V71 Tiny Core Scheduling Definition ----=//
     2: //
     3: //                     The LLVM Compiler Infrastructure
     4: //
     5: // This file is distributed under the University of Illinois Open Source
     6: // License. See LICENSE.TXT for details.
     7: //
     8: //===----------------------------------------------------------------------===//
     9: 
    10: class HexagonV71TPseudoItin {
    11:   list<InstrItinData> V71TPseudoItin_list = [
    12:     InstrItinData<PSEUDO, [InstrStage<1, [SLOT0, SLOT2, SLOT3]>], [2, 1, 1],
    13:                           [Hex_FWD, Hex_FWD, Hex_FWD]>,
    14:     InstrItinData<PSEUDOM, [InstrStage<1, [SLOT2, SLOT3], 0>,
    15:                             InstrStage<1, [SLOT2, SLOT3]>],
    16:                            [2, 1, 1],
    17:                            [Hex_FWD, Hex_FWD, Hex_FWD]>,
    18:     InstrItinData<DUPLEX, [InstrStage<1, [SLOT0]>],
    19:                           [2, 1, 1]>,
    20:     InstrItinData<tc_ENDLOOP, [InstrStage<1, [SLOT_ENDLOOP]>], [2]>
    21:   ];
    22: }
    23: 
    24: //
    25: // HVXItin contains some old itineraries still used by a handful of
```
- EN: It declares types such as HexagonV71TPseudoItin, which carry the state or API of this component. It defines declarative TableGen records like HexagonV71TPseudoItin; these records are consumed by TableGen instead of executed directly. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonScheduleV71T, HexagonV71TPseudoItin, showing how the code connects to sibling backend components.
- CN: 这里声明了 HexagonV71TPseudoItin 等类型，用来承载该组件的状态或接口。 这里定义了 HexagonV71TPseudoItin 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonScheduleV71T, HexagonV71TPseudoItin，说明了它与同级后端组件的连接关系。

### Lines 26-50 / 第 26-50 行

```tablegen
    26: // instructions. Hopefully, we will be able to get rid of them soon.
    27: def HexagonV71TItinList : DepScalarItinV71T, DepHVXItinV71, HVXItin,
    28:                           HexagonV71TPseudoItin {
    29:   list<InstrItinData> V71TItin_list = [
    30:     InstrItinData<LD_tc_ld_SLOT01, [InstrStage<1, [SLOT0]>],
    31:                                    [3, 1, 1],
    32:                                    [Hex_FWD, Hex_FWD, Hex_FWD]>,
    33:     InstrItinData<ST_tc_st_SLOT01, [InstrStage<1, [SLOT0]>],
    34:                                    [1, 1, 3, 3],
    35:                                    [Hex_FWD, Hex_FWD]>
    36:   ];
    37:   list<InstrItinData> ItinList =
    38:     !listconcat(DepScalarItinV71T_list, V71TItin_list, DepHVXItinV71_list,
    39:                 HVXItin_list, V71TPseudoItin_list);
    40: }
    41: 
    42: def HexagonItinerariesV71T :
    43:       ProcessorItineraries<[SLOT0, SLOT1, SLOT2, SLOT3, SLOT_ENDLOOP,
    44:                             CVI_ST, CVI_XLANE, CVI_SHIFT, CVI_MPY0, CVI_MPY1,
    45:                             CVI_LD, CVI_XLSHF, CVI_MPY01, CVI_ALL,
    46:                             CVI_ALL_NOMEM, CVI_ZW],
    47:                             [Hex_FWD, HVX_FWD],
    48:                             HexagonV71TItinList.ItinList>;
    49: 
    50: def HexagonModelV71T : SchedMachineModel {
```
- EN: It defines declarative TableGen records like HexagonV71TItinList, HexagonItinerariesV71T, HexagonModelV71T; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as listconcat, translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonV71TItinList, HexagonV71TPseudoItin, HexagonItinerariesV71T, HexagonModelV71T, showing how the code connects to sibling backend components.
- CN: 这里定义了 HexagonV71TItinList, HexagonItinerariesV71T, HexagonModelV71T 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 listconcat 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonV71TItinList, HexagonV71TPseudoItin, HexagonItinerariesV71T, HexagonModelV71T，说明了它与同级后端组件的连接关系。

### Lines 51-59 / 第 51-59 行

```tablegen
    51:   let IssueWidth = 3;
    52:   let Itineraries = HexagonItinerariesV71T;
    53:   let LoadLatency = 1;
    54:   let CompleteModel = 0;
    55: }
    56: 
    57: //===----------------------------------------------------------------------===//
    58: // Hexagon V71 Tiny Core Resource Definitions -
    59: //===----------------------------------------------------------------------===//
```
- EN: Notable Hexagon symbols referenced here include HexagonItinerariesV71T, showing how the code connects to sibling backend components.
- CN: 这里引用的重要 Hexagon 符号包括 HexagonItinerariesV71T，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- HVX vector ISA modeling / HVX 向量 ISA 建模
- instruction scheduling models / 指令调度模型
- instruction semantics / 指令语义
- generated metadata tables / 生成的元数据表
- TableGen DSL records / TableGen DSL 记录

## Dependencies / 依赖关系

- Hexagon symbols / Hexagon 符号: `HexagonScheduleV71T, HexagonV71TPseudoItin, HexagonV71TItinList, HexagonItinerariesV71T, HexagonModelV71T`
- TableGen pipeline / TableGen 流程: consumed by LLVM TableGen to generate target metadata and `*.inc` tables for the Hexagon backend. / 由 LLVM TableGen 消费，生成 Hexagon 后端所需的目标元数据和 `*.inc` 表。
