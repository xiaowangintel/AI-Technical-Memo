# X86InstrRAOINT.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/X86InstrRAOINT.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines instruction definitions and metadata for the core X86 backend using LLVM TableGen DSL. / 使用 LLVM TableGen DSL 为X86 后端核心定义指令定义与元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: File header, licensing, and opening context / 文件头、许可证与开场上下文
```tablegen
//===---- X86InstrRAOINT.td -------------------------------*- tablegen -*--===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the instructions that make up the Intel RAO-INT
// instruction set.
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
// RAO-INT instructions

def SDTRAOBinaryArith : SDTypeProfile<0, 2, [SDTCisPtrTy<0>, SDTCisInt<1>]>;

// RAO arithmetic instructions.
// OUTCHAIN = AADD(INCHAIN, PTR, RHS)
```
**EN:** This section uses LLVM TableGen DSL to describe file header, licensing, and opening context for the core X86 backend. Key symbols include SDTRAOBinaryArith. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的文件头、许可证与开场上下文。关键符号包括 SDTRAOBinaryArith。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 21-40: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
def X86rao_add  : SDNode<"X86ISD::AADD", SDTRAOBinaryArith,
                         [SDNPHasChain, SDNPMayStore, SDNPMemOperand]>;
def X86rao_or   : SDNode<"X86ISD::AOR",  SDTRAOBinaryArith,
                         [SDNPHasChain, SDNPMayStore, SDNPMemOperand]>;
def X86rao_xor  : SDNode<"X86ISD::AXOR", SDTRAOBinaryArith,
                         [SDNPHasChain, SDNPMayStore, SDNPMemOperand]>;
def X86rao_and  : SDNode<"X86ISD::AAND", SDTRAOBinaryArith,
                         [SDNPHasChain, SDNPMayStore, SDNPMemOperand]>;

multiclass RaoInt<string m, string suffix = ""> {
  let Pattern = [(!cast<SDNode>("X86rao_" # m) addr:$src1, GR32:$src2)] in
    def 32mr#suffix : BinOpMR_M<0xfc, "a" # m, Xi32>;
  let Pattern = [(!cast<SDNode>("X86rao_" # m) addr:$src1, GR64:$src2)] in
    def 64mr#suffix : BinOpMR_M<0xfc, "a" # m, Xi64>;
}

let Predicates = [HasRAOINT, NoEGPR] in {
  defm AADD : RaoInt<"add">, T8;
  defm AAND : RaoInt<"and">, T8, PD;
  defm AOR  : RaoInt<"or" >, T8, XD;
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include X86rao_add, X86rao_or, X86rao_xor, X86rao_and, RaoInt, AADD. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 X86rao_add, X86rao_or, X86rao_xor, X86rao_and, RaoInt, AADD。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 41-49: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
  defm AXOR : RaoInt<"xor">, T8, XS;
}

let Predicates = [HasRAOINT, HasEGPR, In64BitMode] in {
  defm AADD : RaoInt<"add", "_EVEX">, EVEX, T_MAP4;
  defm AAND : RaoInt<"and", "_EVEX">, EVEX, T_MAP4, PD;
  defm AOR  : RaoInt<"or",  "_EVEX">, EVEX, T_MAP4, XD;
  defm AXOR : RaoInt<"xor", "_EVEX">, EVEX, T_MAP4, XS;
}
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include AXOR, AADD, AAND, AOR. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 AXOR, AADD, AAND, AOR。这些记录随后会展开为 X86 后端使用的生成表。

## Key Concepts / 关键概念
- Primary topic: instruction definitions and metadata. / 核心主题：指令定义与元数据。
- Subsystem: the core X86 backend. / 所属子系统：X86 后端核心。
- Notable symbols: SDTRAOBinaryArith, X86rao_add, X86rao_or, X86rao_xor, X86rao_and, RaoInt. / 重要符号：SDTRAOBinaryArith, X86rao_add, X86rao_or, X86rao_xor, X86rao_and, RaoInt。
- This file is written in TableGen DSL and contributes generated target tables. / 该文件使用 TableGen DSL 编写，并参与生成目标描述表。

## Dependencies / 依赖关系
- Consumed by LLVM TableGen to emit generated matcher, encoding, register, or scheduling tables. / 由 LLVM TableGen 消费，以生成匹配器、编码、寄存器或调度相关表。
- Closely coupled with sibling X86 .td fragments that assemble the full target description. / 与同级 X86 .td 片段紧密协作，共同组成完整目标描述。
