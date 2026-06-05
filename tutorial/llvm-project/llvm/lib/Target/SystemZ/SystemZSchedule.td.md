# SystemZSchedule.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZSchedule.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines declarative SystemZ backend records and patterns using LLVM TableGen DSL.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义声明式的 SystemZ 后端记录与匹配模式。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12 / 第 1-12 行
```tablegen
   1: //==-- SystemZSchedule.td - SystemZ Scheduling Definitions ----*- tblgen -*-==//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: // Scheduler resources
  10: 
  11: // These resources are used to express decoder grouping rules.  The number of
  12: // decoder slots needed by an instructions is normally one, but there are
```
- **EN**: This range is mainly descriptive commentary or banner text that frames the file before executable definitions begin.
- **CN**: 这一段主要是说明性注释或横幅文本，用于在可执行定义开始前说明文件背景。

### Lines 13-24 / 第 13-24 行
```tablegen
  13: // exceptions.
  14: def NormalGr    : SchedWrite;
  15: def Cracked     : SchedWrite;
  16: def GroupAlone  : SchedWrite;
  17: def GroupAlone2 : SchedWrite;
  18: def GroupAlone3 : SchedWrite;
  19: def BeginGroup  : SchedWrite;
  20: def EndGroup    : SchedWrite;
  21: 
  22: // A SchedWrite added to other SchedWrites to make LSU latency parameterizable.
  23: def LSULatency : SchedWrite;
  24: 
```
- **EN**: This block declares or refines TableGen records such as `NormalGr`, `Cracked`, `GroupAlone`, `GroupAlone2`, `GroupAlone3`, `BeginGroup`.
- **CN**: 该代码块声明或细化了 `NormalGr`, `Cracked`, `GroupAlone`, `GroupAlone2`, `GroupAlone3`, `BeginGroup` 等 TableGen 记录。

### Lines 25-36 / 第 25-36 行
```tablegen
  25: // Operand WriteLatencies.
  26: foreach L = 1 - 30 in def "WLat"#L : SchedWrite;
  27: 
  28: foreach L = 1 - 16 in
  29:   def "WLat"#L#"LSU" : WriteSequence<[!cast<SchedWrite>("WLat"#L),
  30:                                       LSULatency]>;
  31: 
  32: // ReadAdvances, used for the register operand next to a memory operand,
  33: // modelling that the register operand is needed later than the address
  34: // operands.
  35: def RegReadAdv : SchedRead;
  36: 
```
- **EN**: This block declares or refines TableGen records such as `RegReadAdv`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 该代码块声明或细化了 `RegReadAdv` 等 TableGen 记录。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 37-48 / 第 37-48 行
```tablegen
  37: foreach Num = ["", "2", "3", "4", "5", "6"] in {
  38:   // Fixed-point units
  39:   def "FXa"#Num : SchedWrite;
  40:   def "FXb"#Num : SchedWrite;
  41:   def "FXU"#Num : SchedWrite;
  42:   // Load/store unit
  43:   def "LSU"#Num : SchedWrite;
  44:   // Vector sub units (z13 and later)
  45:   def "VecBF"#Num : SchedWrite;
  46:   def "VecDF"#Num : SchedWrite;
  47:   def "VecDFX"#Num : SchedWrite;
  48:   def "VecMul"#Num : SchedWrite;
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 49-60 / 第 49-60 行
```tablegen
  49:   def "VecStr"#Num : SchedWrite;
  50:   def "VecXsPm"#Num : SchedWrite;
  51:   // Floating point unit (zEC12 and earlier)
  52:   def "FPU"#Num : SchedWrite;
  53:   def "DFU"#Num : SchedWrite;
  54: }
  55: 
  56: def VecFPd   : SchedWrite; // Blocking BFP div/sqrt unit (30 cycles).
  57: def VecFPd20 : SchedWrite; // Blocking BFP div/sqrt unit, 20 cycles.
  58: 
  59: def VBU : SchedWrite; // Virtual branching unit
  60: 
```
- **EN**: This block declares or refines TableGen records such as `VecFPd`, `VecFPd20`, `VBU`.
- **CN**: 该代码块声明或细化了 `VecFPd`, `VecFPd20`, `VBU` 等 TableGen 记录。

### Lines 61-69 / 第 61-69 行
```tablegen
  61: def MCD : SchedWrite; // Millicode
  62: 
  63: include "SystemZScheduleZ17.td"
  64: include "SystemZScheduleZ16.td"
  65: include "SystemZScheduleZ15.td"
  66: include "SystemZScheduleZ14.td"
  67: include "SystemZScheduleZ13.td"
  68: include "SystemZScheduleZEC12.td"
  69: include "SystemZScheduleZ196.td"
```
- **EN**: It composes TableGen records by including `SystemZScheduleZ17.td`, `SystemZScheduleZ16.td`, `SystemZScheduleZ15.td`, `SystemZScheduleZ14.td`, `SystemZScheduleZ13.td`, `SystemZScheduleZEC12.td` and reusing previously declared backend fragments. This block declares or refines TableGen records such as `MCD`.
- **CN**: 它通过包含 `SystemZScheduleZ17.td`, `SystemZScheduleZ16.td`, `SystemZScheduleZ15.td`, `SystemZScheduleZ14.td`, `SystemZScheduleZ13.td`, `SystemZScheduleZEC12.td` 组合 TableGen 记录，并复用此前声明的后端片段。 该代码块声明或细化了 `MCD` 等 TableGen 记录。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。
- **Scheduling model**: Provides instruction itineraries or scheduling classes for performance modelling. / 提供指令行程或调度类以支持性能建模。

## Dependencies / 依赖关系
- `SystemZScheduleZ17.td`
- `SystemZScheduleZ16.td`
- `SystemZScheduleZ15.td`
- `SystemZScheduleZ14.td`
- `SystemZScheduleZ13.td`
- `SystemZScheduleZEC12.td`
- `SystemZScheduleZ196.td`
