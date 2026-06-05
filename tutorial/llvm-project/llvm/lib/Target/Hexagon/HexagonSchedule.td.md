# HexagonSchedule.td — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonSchedule.td`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Defines Hexagon scheduling model data using TableGen DSL.
- Purpose / 作用 (CN): 该文件使用 TableGen DSL 定义 Hexagon 目标相关的声明式记录。
- Note / 说明: This file is written in LLVM TableGen DSL, so many lines are declarative records consumed by TableGen generators. / 该文件使用 LLVM TableGen DSL 编写，因此很多行是由 TableGen 生成器消费的声明式记录。

## Line-by-Line Analysis / 逐行分析

### Lines 1-25 / 第 1-25 行

```tablegen
     1: //===- HexagonSchedule.td - Hexagon Scheduling Definitions -*- tablegen -*-===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: 
     9: def Hex_FWD : Bypass;
    10: def HVX_FWD : Bypass;
    11: 
    12: // Functional Units.
    13: def SLOT0       : FuncUnit;
    14: def SLOT1       : FuncUnit;
    15: def SLOT2       : FuncUnit;
    16: def SLOT3       : FuncUnit;
    17: // Endloop is a pseudo instruction that is encoded with 2 bits in a packet
    18: // rather than taking an execution slot. This special unit is needed
    19: // to schedule an ENDLOOP with 4 other instructions.
    20: def SLOT_ENDLOOP: FuncUnit;
    21: 
    22: // CVI pipes from the "Hexagon Multimedia Co-Processor Extensions Arch Spec".
    23: def CVI_ST     : FuncUnit;
    24: def CVI_XLANE  : FuncUnit;
    25: def CVI_SHIFT  : FuncUnit;
```
- EN: It defines declarative TableGen records like Hex_FWD, HVX_FWD, SLOT0, SLOT1, SLOT2, ... (10 total); these records are consumed by TableGen instead of executed directly. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonSchedule, showing how the code connects to sibling backend components.
- CN: 这里定义了 Hex_FWD, HVX_FWD, SLOT0, SLOT1, SLOT2, ... (10 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonSchedule，说明了它与同级后端组件的连接关系。

### Lines 26-50 / 第 26-50 行

```tablegen
    26: def CVI_MPY0   : FuncUnit;
    27: def CVI_MPY1   : FuncUnit;
    28: def CVI_LD     : FuncUnit;
    29: def CVI_ZW     : FuncUnit; // Z register write port
    30: 
    31: // Combined functional units.
    32: def CVI_XLSHF  : FuncUnit;
    33: def CVI_MPY01  : FuncUnit;
    34: def CVI_ALL    : FuncUnit;
    35: def CVI_ALL_NOMEM : FuncUnit;
    36: 
    37: // Combined functional unit data.
    38: def HexagonComboFuncsV60 :
    39:     ComboFuncUnits<[
    40:       ComboFuncData<CVI_XLSHF    , [CVI_XLANE, CVI_SHIFT]>,
    41:       ComboFuncData<CVI_MPY01    , [CVI_MPY0, CVI_MPY1]>,
    42:       ComboFuncData<CVI_ALL      , [CVI_ST, CVI_XLANE, CVI_SHIFT,
    43:                                     CVI_MPY0, CVI_MPY1, CVI_LD]>,
    44:       ComboFuncData<CVI_ALL_NOMEM, [CVI_XLANE, CVI_SHIFT, CVI_MPY0, CVI_MPY1]>
    45:     ]>;
    46: 
    47: // Itinerary classes.
    48: def PSEUDO          : InstrItinClass;
    49: def PSEUDOM         : InstrItinClass;
    50: def DUPLEX          : InstrItinClass;
```
- EN: It defines declarative TableGen records like CVI_MPY0, CVI_MPY1, CVI_LD, CVI_ZW, CVI_XLSHF, ... (12 total); these records are consumed by TableGen instead of executed directly. Notable Hexagon symbols referenced here include HexagonComboFuncsV60, showing how the code connects to sibling backend components.
- CN: 这里定义了 CVI_MPY0, CVI_MPY1, CVI_LD, CVI_ZW, CVI_XLSHF, ... (12 total) 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里引用的重要 Hexagon 符号包括 HexagonComboFuncsV60，说明了它与同级后端组件的连接关系。

### Lines 51-75 / 第 51-75 行

```tablegen
    51: def tc_ENDLOOP      : InstrItinClass;
    52: 
    53: //===----------------------------------------------------------------------===//
    54: // Auto-generated itinerary classes
    55: //===----------------------------------------------------------------------===//
    56: include "HexagonDepIICScalar.td"
    57: include "HexagonDepIICHVX.td"
    58: 
    59: include "HexagonScheduleV5.td"
    60: include "HexagonScheduleV55.td"
    61: 
    62: include "HexagonIICScalar.td"
    63: include "HexagonIICHVX.td"
    64: include "HexagonScheduleV60.td"
    65: 
    66: include "HexagonScheduleV62.td"
    67: include "HexagonScheduleV65.td"
    68: include "HexagonScheduleV66.td"
    69: include "HexagonScheduleV67.td"
    70: include "HexagonScheduleV67T.td"
    71: include "HexagonScheduleV68.td"
    72: include "HexagonScheduleV69.td"
    73: include "HexagonScheduleV71.td"
    74: include "HexagonScheduleV71T.td"
    75: include "HexagonScheduleV73.td"
```
- EN: It includes TableGen fragments such as HexagonDepIICScalar.td, HexagonDepIICHVX.td, HexagonScheduleV5.td, HexagonScheduleV55.td, ... (17 total) so records can build on shared target definitions. It defines declarative TableGen records like tc_ENDLOOP; these records are consumed by TableGen instead of executed directly. Notable Hexagon symbols referenced here include HexagonDepIICScalar, HexagonDepIICHVX, HexagonScheduleV5, HexagonScheduleV55, ... (17 total), showing how the code connects to sibling backend components.
- CN: 这里包含了 HexagonDepIICScalar.td, HexagonDepIICHVX.td, HexagonScheduleV5.td, HexagonScheduleV55.td, ... (17 total) 等 TableGen 片段，使记录可以复用共享目标定义。 这里定义了 tc_ENDLOOP 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里引用的重要 Hexagon 符号包括 HexagonDepIICScalar, HexagonDepIICHVX, HexagonScheduleV5, HexagonScheduleV55, ... (17 total)，说明了它与同级后端组件的连接关系。

### Lines 76-78 / 第 76-78 行

```tablegen
    76: include "HexagonScheduleV75.td"
    77: include "HexagonScheduleV79.td"
    78: include "HexagonScheduleV81.td"
```
- EN: It includes TableGen fragments such as HexagonScheduleV75.td, HexagonScheduleV79.td, HexagonScheduleV81.td so records can build on shared target definitions. Notable Hexagon symbols referenced here include HexagonScheduleV75, HexagonScheduleV79, HexagonScheduleV81, showing how the code connects to sibling backend components.
- CN: 这里包含了 HexagonScheduleV75.td, HexagonScheduleV79.td, HexagonScheduleV81.td 等 TableGen 片段，使记录可以复用共享目标定义。 这里引用的重要 Hexagon 符号包括 HexagonScheduleV75, HexagonScheduleV79, HexagonScheduleV81，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- HVX vector ISA modeling / HVX 向量 ISA 建模
- instruction scheduling models / 指令调度模型
- instruction semantics / 指令语义
- generated metadata tables / 生成的元数据表
- TableGen DSL records / TableGen DSL 记录

## Dependencies / 依赖关系

- Included TableGen files / 包含的 TableGen 文件: `HexagonDepIICScalar.td, HexagonDepIICHVX.td, HexagonScheduleV5.td, HexagonScheduleV55.td, HexagonIICScalar.td, HexagonIICHVX.td, HexagonScheduleV60.td, HexagonScheduleV62.td, HexagonScheduleV65.td, HexagonScheduleV66.td, ... (20 total)`
- Hexagon symbols / Hexagon 符号: `HexagonSchedule, HexagonComboFuncsV60, HexagonDepIICScalar, HexagonDepIICHVX, HexagonScheduleV5, HexagonScheduleV55, HexagonIICScalar, HexagonIICHVX, HexagonScheduleV60, HexagonScheduleV62, ... (22 total)`
- TableGen pipeline / TableGen 流程: consumed by LLVM TableGen to generate target metadata and `*.inc` tables for the Hexagon backend. / 由 LLVM TableGen 消费，生成 Hexagon 后端所需的目标元数据和 `*.inc` 表。
