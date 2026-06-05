# M68kInstrCompiler.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/M68k/M68kInstrCompiler.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines declarative M68k backend records and patterns using LLVM TableGen DSL.
- **用途 (CN)**: 使用 LLVM TableGen DSL 定义声明式的 M68k 后端记录与匹配模式。
- **Note / 说明**: This file is written in LLVM TableGen DSL rather than standard C++ source. / 该文件使用 LLVM TableGen DSL，而不是普通 C++ 源码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```tablegen
   1: //===-- M68kInstrCompiler.td - Pseudos and Patterns --------*- tablegen -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: ///
   9: /// \file
  10: /// This file describes the various pseudo instructions used by the compiler,
  11: /// as well as Pat patterns used during instruction selection.
  12: ///
  13: //===----------------------------------------------------------------------===//
  14: 
  15: //===----------------------------------------------------------------------===//
  16: // ConstantPool, GlobalAddress, ExternalSymbol, and JumpTable
  17: //===----------------------------------------------------------------------===//
  18: 
```
- **EN**: This range is mainly descriptive commentary or banner text that frames the file before executable definitions begin.
- **CN**: 这一段主要是说明性注释或横幅文本，用于在可执行定义开始前说明文件背景。

### Lines 19-36 / 第 19-36 行
```tablegen
  19: def : Pat<(i32 (MxWrapper tconstpool    :$src)), (MOV32ri tconstpool    :$src)>;
  20: def : Pat<(i32 (MxWrapper tglobaladdr   :$src)), (MOV32ri tglobaladdr   :$src)>;
  21: def : Pat<(i32 (MxWrapper texternalsym  :$src)), (MOV32ri texternalsym  :$src)>;
  22: def : Pat<(i32 (MxWrapper tjumptable    :$src)), (MOV32ri tjumptable    :$src)>;
  23: def : Pat<(i32 (MxWrapper tblockaddress :$src)), (MOV32ri tblockaddress :$src)>;
  24: 
  25: def : Pat<(add MxDRD32:$src, (MxWrapper tconstpool:$opd)),
  26:           (ADD32di MxDRD32:$src, tconstpool:$opd)>;
  27: def : Pat<(add MxARD32:$src, (MxWrapper tjumptable:$opd)),
  28:           (ADD32ai MxARD32:$src, tjumptable:$opd)>;
  29: def : Pat<(add MxARD32:$src, (MxWrapper tglobaladdr :$opd)),
  30:           (ADD32ai MxARD32:$src, tglobaladdr:$opd)>;
  31: def : Pat<(add MxARD32:$src, (MxWrapper texternalsym:$opd)),
  32:           (ADD32ai MxARD32:$src, texternalsym:$opd)>;
  33: def : Pat<(add MxARD32:$src, (MxWrapper tblockaddress:$opd)),
  34:           (ADD32ai MxARD32:$src, tblockaddress:$opd)>;
  35: 
  36: def : Pat<(store (i32 (MxWrapper tglobaladdr:$src)), iPTR:$dst),
```
- **EN**: This span continues the file's main responsibility: defines declarative M68k backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 37-54 / 第 37-54 行
```tablegen
  37:           (MOV32ji MxARI32:$dst, tglobaladdr:$src)>;
  38: def : Pat<(store (i32 (MxWrapper texternalsym:$src)), iPTR:$dst),
  39:           (MOV32ji MxARI32:$dst, texternalsym:$src)>;
  40: def : Pat<(store (i32 (MxWrapper tblockaddress:$src)), iPTR:$dst),
  41:           (MOV32ji MxARI32:$dst, tblockaddress:$src)>;
  42: 
  43: def : Pat<(i32 (MxWrapperPC tconstpool    :$src)), (LEA32q tconstpool    :$src)>;
  44: def : Pat<(i32 (MxWrapperPC tglobaladdr   :$src)), (LEA32q tglobaladdr   :$src)>;
  45: def : Pat<(i32 (MxWrapperPC texternalsym  :$src)), (LEA32q texternalsym  :$src)>;
  46: def : Pat<(i32 (MxWrapperPC tjumptable    :$src)), (LEA32q tjumptable    :$src)>;
  47: def : Pat<(i32 (MxWrapperPC tblockaddress :$src)), (LEA32q tblockaddress :$src)>;
  48: 
  49: 
  50: //===----------------------------------------------------------------------===//
  51: // Conditional Move Pseudo Instructions
  52: //
  53: // CMOV* - Used to implement the SELECT DAG operation. Expanded after
  54: // instruction selection into a branch sequence.
```
- **EN**: This span continues the file's main responsibility: defines declarative M68k backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 55-72 / 第 55-72 行
```tablegen
  55: //===----------------------------------------------------------------------===//
  56: 
  57: let usesCustomInserter = 1, Uses = [CCR] in
  58: class MxCMove<MxType TYPE>
  59:     : MxPseudo<(outs TYPE.ROp:$dst), (ins TYPE.ROp:$t, TYPE.ROp:$f, i8imm:$cond),
  60:                [(set TYPE.VT:$dst,
  61:                      (TYPE.VT (MxCmov TYPE.VT:$t, TYPE.VT:$f, imm:$cond, CCR)))]>;
  62: 
  63: def CMOV8d  : MxCMove<MxType8d>;
  64: def CMOV16d : MxCMove<MxType16d>;
  65: def CMOV32r : MxCMove<MxType32r>;
  66: 
  67: 
  68: //===----------------------------------------------------------------------===//
  69: // Calls
  70: //===----------------------------------------------------------------------===//
  71: 
  72: // ADJCALLSTACKDOWN/UP implicitly use/def %SP because they may be expanded into
```
- **EN**: This block declares or refines TableGen records such as `MxCMove`, `CMOV8d`, `CMOV16d`, `CMOV32r`.
- **CN**: 该代码块声明或细化了 `MxCMove`, `CMOV8d`, `CMOV16d`, `CMOV32r` 等 TableGen 记录。

### Lines 73-90 / 第 73-90 行
```tablegen
  73: // a stack adjustment and the codegen must know that they may modify the stack
  74: // pointer before prolog-epilog rewriting occurs.
  75: // Pessimistically assume ADJCALLSTACKDOWN / ADJCALLSTACKUP will become
  76: // sub / add which can clobber CCR.
  77: let Defs = [SP, CCR], Uses = [SP] in {
  78: 
  79:   def ADJCALLSTACKDOWN
  80:     : MxPseudo<(outs), (ins i32imm:$amt1, i32imm:$amt2),
  81:                [(MxCallSeqStart timm:$amt1, timm:$amt2)]>;
  82: 
  83:   def ADJCALLSTACKUP
  84:     : MxPseudo<(outs), (ins i32imm:$amt1, i32imm:$amt2),
  85:                [(MxCallSeqEnd timm:$amt1, timm:$amt2)]>;
  86: 
  87: } // Defs
  88: 
  89: //===----------------------------------------------------------------------===//
  90: // Tail Call
```
- **EN**: This block declares or refines TableGen records such as `ADJCALLSTACKDOWN`, `ADJCALLSTACKUP`.
- **CN**: 该代码块声明或细化了 `ADJCALLSTACKDOWN`, `ADJCALLSTACKUP` 等 TableGen 记录。

### Lines 91-108 / 第 91-108 行
```tablegen
  91: //===----------------------------------------------------------------------===//
  92: 
  93: // Tailcall stuff. The TCRETURN instructions execute after the epilog, so they
  94: // can never use callee-saved registers. That is the purpose of the XR32_TC
  95: // register classes.
  96: 
  97: // FIXME TC is disabled for PIC mode because the global base
  98: // register which is part of the address mode may be assigned a
  99: // callee-saved register.
 100: def : Pat<(MxTCRet (load MxCP_ARII:$dst), imm:$adj),
 101:           (TCRETURNj (MOV32af_TC MxARII32:$dst), imm:$adj)>,
 102:       Requires<[IsNotPIC]>;
 103: 
 104: def : Pat<(MxTCRet AR32_TC:$dst, imm:$adj),
 105:           (TCRETURNj MxARI32_TC:$dst, imm:$adj)>;
 106: 
 107: def : Pat<(MxTCRet (i32 tglobaladdr:$dst), imm:$adj),
 108:           (TCRETURNq MxPCD32:$dst, imm:$adj)>;
```
- **EN**: This span continues the file's main responsibility: defines declarative M68k backend records and patterns using LLVM TableGen DSL.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 109-126 / 第 109-126 行
```tablegen
 109: 
 110: def : Pat<(MxTCRet (i32 texternalsym:$dst), imm:$adj),
 111:           (TCRETURNq MxPCD32:$dst, imm:$adj)>;
 112: 
 113: 
 114: //===----------------------------------------------------------------------===//
 115: // Segmented Stack
 116: //
 117: // When using segmented stacks these are lowered into instructions which first
 118: // check if the current stacklet has enough free memory. If it does, memory is
 119: // allocated by bumping the stack pointer. Otherwise memory is allocated from
 120: // the heap.
 121: //===----------------------------------------------------------------------===//
 122: 
 123: let Defs = [SP, CCR], Uses = [SP] in
 124: let usesCustomInserter = 1 in
 125: def SALLOCA : MxPseudo<(outs MxARD32:$dst), (ins MxARD32:$size),
 126:                        [(set iPTR:$dst, (MxSegAlloca iPTR:$size))]>;
```
- **EN**: This block declares or refines TableGen records such as `SALLOCA`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `SALLOCA` 等 TableGen 记录。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。

## Dependencies / 依赖关系
- No explicit include directives were found; dependencies are primarily implicit through LLVM build structure or generated files. / 未发现显式 include 指令；依赖主要通过 LLVM 构建结构或生成文件隐式提供。
