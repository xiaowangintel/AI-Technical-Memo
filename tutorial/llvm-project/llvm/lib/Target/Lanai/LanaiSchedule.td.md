# LanaiSchedule.td — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/Lanai/LanaiSchedule.td`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Describes instruction scheduling models and processor resources in TableGen.
- 目的（中文）: 使用 TableGen 描述指令调度模型与处理器资源。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```tablegen
   1: //=-LanaiSchedule.td - Lanai Scheduling Definitions --*- tablegen -*-=========//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: def ALU_FU  : FuncUnit;
  10: def LDST_FU : FuncUnit;
  11: 
  12: def IIC_ALU  : InstrItinClass;
  13: def IIC_LD   : InstrItinClass;
  14: def IIC_ST   : InstrItinClass;
  15: def IIC_LDSW : InstrItinClass;
  16: def IIC_STSW : InstrItinClass;
  17: 
  18: def LanaiItinerary : ProcessorItineraries<[ALU_FU, LDST_FU],[],[
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. This range defines declarative TableGen records such as ALU_FU, LDST_FU, IIC_ALU, capturing target metadata instead of imperative runtime logic.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这一段定义了 ALU_FU, LDST_FU, IIC_ALU 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。

### Lines 19-36

```tablegen
  19:   InstrItinData<IIC_LD,   [InstrStage<1, [LDST_FU]>]>,
  20:   InstrItinData<IIC_ST,   [InstrStage<1, [LDST_FU]>]>,
  21:   InstrItinData<IIC_LDSW, [InstrStage<2, [LDST_FU]>]>,
  22:   InstrItinData<IIC_STSW, [InstrStage<2, [LDST_FU]>]>,
  23:   InstrItinData<IIC_ALU,  [InstrStage<1, [ALU_FU]>]>
  24: ]>;
  25: 
  26: def LanaiSchedModel : SchedMachineModel {
  27:   // Cycles for loads to access the cache [default = -1]
  28:   let LoadLatency = 2;
  29: 
  30:   // Max micro-ops that can be buffered for optimized loop dispatch/execution.
  31:   // [default = -1]
  32:   let LoopMicroOpBufferSize = 0;
  33: 
  34:   // Allow scheduler to assign default model to any unrecognized opcodes.
  35:   // [default = 1]
  36:   let CompleteModel = 0;
```

- EN: This range defines declarative TableGen records such as LanaiSchedModel, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 LanaiSchedModel 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 37-54

```tablegen
  37: 
  38:   // Max micro-ops that may be scheduled per cycle. [default = 1]
  39:   let IssueWidth = 1;
  40: 
  41:   // Extra cycles for a mispredicted branch. [default = -1]
  42:   let MispredictPenalty = 10;
  43: 
  44:   // Enable Post RegAlloc Scheduler pass. [default = 0]
  45:   let PostRAScheduler = 0;
  46: 
  47:   // Max micro-ops that can be buffered. [default = -1]
  48:   let MicroOpBufferSize = 0;
  49: 
  50:   // Per-cycle resources tables. [default = NoItineraries]
  51:   let Itineraries = LanaiItinerary;
  52: }
  53: 
  54: def ALU : ProcResource<1> { let BufferSize = 0; }
```

- EN: This range defines declarative TableGen records such as ALU, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 ALU 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

### Lines 55-69

```tablegen
  55: def LdSt : ProcResource<1> { let BufferSize = 0; }
  56: 
  57: def WriteLD   : SchedWrite;
  58: def WriteST   : SchedWrite;
  59: def WriteLDSW : SchedWrite;
  60: def WriteSTSW : SchedWrite;
  61: def WriteALU  : SchedWrite;
  62: 
  63: let SchedModel = LanaiSchedModel in {
  64:   def : WriteRes<WriteLD, [LdSt]>   { let Latency = 2; }
  65:   def : WriteRes<WriteST, [LdSt]>   { let Latency = 2; }
  66:   def : WriteRes<WriteLDSW, [LdSt]> { let Latency = 2; }
  67:   def : WriteRes<WriteSTSW, [LdSt]> { let Latency = 4; }
  68:   def : WriteRes<WriteALU, [ALU]>   { let Latency = 1; }
  69: }
```

- EN: This range defines declarative TableGen records such as LdSt, WriteLD, WriteST, capturing target metadata instead of imperative runtime logic. `let` clauses override inherited fields, refining encodings, predicates, or scheduling attributes for a group of records.
- 中文: 这一段定义了 LdSt, WriteLD, WriteST 等声明式 TableGen 记录，用于表达目标元数据，而不是命令式运行时代码。 `let` 子句用于覆盖继承字段，从而细化一组记录的编码、谓词或调度属性。

## Key Concepts / 关键概念

- TableGen DSL / TableGen DSL
- Processor resources / 处理器资源
- Latency modelling / 时延建模
- Scheduling model / 调度模型

## Dependencies / 依赖关系

- LLVM subsystems / LLVM 子系统: TableGen
