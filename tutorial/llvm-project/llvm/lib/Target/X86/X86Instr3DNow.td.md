# X86Instr3DNow.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/X86Instr3DNow.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines instruction definitions and metadata for the core X86 backend using LLVM TableGen DSL. / 使用 LLVM TableGen DSL 为X86 后端核心定义指令定义与元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: File header, licensing, and opening context / 文件头、许可证与开场上下文
```tablegen
//===-- X86Instr3DNow.td - The 3DNow! Instruction Set ------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the 3DNow! instruction set, which extends MMX to support
// floating point and also adds a few more random instructions for good measure.
//
//===----------------------------------------------------------------------===//

class I3DNow<bits<8> o, Format F, dag outs, dag ins, string asm, list<dag> pat>
      : I<o, F, outs, ins, asm, pat> {
}

class I3DNow_binop<bits<8> o, Format F, dag ins, string Mnemonic, list<dag> pat>
      : I3DNow<o, F, (outs VR64:$dst), ins,
          !strconcat(Mnemonic, "\t{$src2, $dst|$dst, $src2}"), pat>, ThreeDNow {
```
**EN:** This section uses LLVM TableGen DSL to describe file header, licensing, and opening context for the core X86 backend. Key symbols include I3DNow, I3DNow_binop. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的文件头、许可证与开场上下文。关键符号包括 I3DNow, I3DNow_binop。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 21-40: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
  let Constraints = "$src1 = $dst";
}

class I3DNow_conv<bits<8> o, Format F, dag ins, string Mnemonic, list<dag> pat>
      : I3DNow<o, F, (outs VR64:$dst), ins,
          !strconcat(Mnemonic, "\t{$src, $dst|$dst, $src}"), pat>, ThreeDNow;

multiclass I3DNow_binop_rm<bits<8> opc, string Mn,
                           X86FoldableSchedWrite sched, bit Commutable = 0> {
  let mayStore=0, hasSideEffects=0 in {
    let isCommutable = Commutable, mayLoad=0 in
    def rr : I3DNow_binop<opc, MRMSrcReg, (ins VR64:$src1, VR64:$src2), Mn,
      []>, Sched<[sched]>;
    let mayLoad=1 in
    def rm : I3DNow_binop<opc, MRMSrcMem, (ins VR64:$src1, i64mem:$src2), Mn,
      []>, Sched<[sched.Folded, sched.ReadAfterFold]>;
  }
}

multiclass I3DNow_conv_rm<bits<8> opc, string Mn,
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include I3DNow_conv, I3DNow_binop_rm, rr, rm, I3DNow_conv_rm. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 I3DNow_conv, I3DNow_binop_rm, rr, rm, I3DNow_conv_rm。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 41-60: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
                              X86FoldableSchedWrite sched> {
  let mayStore=0, hasSideEffects=0 in {
    let mayLoad=0 in
    def rr : I3DNow_conv<opc, MRMSrcReg, (ins VR64:$src), Mn,
      []>, Sched<[sched]>;
    let mayLoad=1 in
    def rm : I3DNow_conv<opc, MRMSrcMem, (ins i64mem:$src), Mn,
      []>, Sched<[sched.Folded, sched.ReadAfterFold]>;
  }
}

defm PAVGUSB  : I3DNow_binop_rm<0xBF, "pavgusb", SchedWriteVecALU.MMX, 1>;
defm PF2ID    : I3DNow_conv_rm<0x1D, "pf2id", WriteCvtPS2I>;
defm PFACC    : I3DNow_binop_rm<0xAE, "pfacc", WriteFAdd>;
defm PFADD    : I3DNow_binop_rm<0x9E, "pfadd", WriteFAdd, 1>;
defm PFCMPEQ  : I3DNow_binop_rm<0xB0, "pfcmpeq", WriteFAdd, 1>;
defm PFCMPGE  : I3DNow_binop_rm<0x90, "pfcmpge", WriteFAdd>;
defm PFCMPGT  : I3DNow_binop_rm<0xA0, "pfcmpgt", WriteFAdd>;
defm PFMAX    : I3DNow_binop_rm<0xA4, "pfmax", WriteFAdd>;
defm PFMIN    : I3DNow_binop_rm<0x94, "pfmin", WriteFAdd>;
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include rr, rm, PAVGUSB, PF2ID, PFACC, PFADD. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 rr, rm, PAVGUSB, PF2ID, PFACC, PFADD。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 61-80: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
defm PFMUL    : I3DNow_binop_rm<0xB4, "pfmul", WriteFAdd, 1>;
defm PFRCP    : I3DNow_conv_rm<0x96, "pfrcp", WriteFAdd>;
defm PFRCPIT1 : I3DNow_binop_rm<0xA6, "pfrcpit1", WriteFAdd>;
defm PFRCPIT2 : I3DNow_binop_rm<0xB6, "pfrcpit2", WriteFAdd>;
defm PFRSQIT1 : I3DNow_binop_rm<0xA7, "pfrsqit1", WriteFAdd>;
defm PFRSQRT  : I3DNow_conv_rm<0x97, "pfrsqrt", WriteFAdd>;
defm PFSUB    : I3DNow_binop_rm<0x9A, "pfsub", WriteFAdd, 1>;
defm PFSUBR   : I3DNow_binop_rm<0xAA, "pfsubr", WriteFAdd, 1>;
defm PI2FD    : I3DNow_conv_rm<0x0D, "pi2fd", WriteCvtI2PS>;
defm PMULHRW  : I3DNow_binop_rm<0xB7, "pmulhrw", SchedWriteVecIMul.MMX, 1>;

let SchedRW = [WriteEMMS], mayLoad=1, mayStore=1, hasSideEffects=1 in
def FEMMS : I3DNow<0x0E, RawFrm, (outs), (ins), "femms",
                   []>, TB;

let SchedRW = [WriteLoad], mayLoad=1, mayStore=1, hasSideEffects=0 in {
def PREFETCH : I3DNow<0x0D, MRM0m, (outs), (ins i8mem:$addr),
                      "prefetch\t$addr",
                      []>, TB;

```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include PFMUL, PFRCP, PFRCPIT1, PFRCPIT2, PFRSQIT1, PFRSQRT. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 PFMUL, PFRCP, PFRCPIT1, PFRCPIT2, PFRSQIT1, PFRSQRT。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 81-95: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
// Note: PREFETCHW is the only instruction in this file which is NOT specific to 3DNow!
def PREFETCHW : I<0x0D, MRM1m, (outs), (ins i8mem:$addr), "prefetchw\t$addr",
                  [(prefetch addr:$addr, (i32 1), (i32 PrefetchWLevel), (i32 1))]>,
                  TB, Requires<[HasPrefetchW]>;

def PREFETCHWT1 : I<0x0D, MRM2m, (outs), (ins i8mem:$addr), "prefetchwt1\t$addr",
                    []>, TB;
}

// "3DNowA" instructions
defm PF2IW    : I3DNow_conv_rm<0x1C, "pf2iw", WriteCvtPS2I>;
defm PI2FW    : I3DNow_conv_rm<0x0C, "pi2fw", WriteCvtI2PS>;
defm PFNACC   : I3DNow_binop_rm<0x8A, "pfnacc", WriteFAdd, 0>;
defm PFPNACC  : I3DNow_binop_rm<0x8E, "pfpnacc", WriteFAdd, 0>;
defm PSWAPD   : I3DNow_conv_rm<0xBB, "pswapd", SchedWriteShuffle.MMX>;
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include PREFETCHW, PREFETCHWT1, PF2IW, PI2FW, PFNACC, PFPNACC. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 PREFETCHW, PREFETCHWT1, PF2IW, PI2FW, PFNACC, PFPNACC。这些记录随后会展开为 X86 后端使用的生成表。

## Key Concepts / 关键概念
- Primary topic: instruction definitions and metadata. / 核心主题：指令定义与元数据。
- Subsystem: the core X86 backend. / 所属子系统：X86 后端核心。
- Notable symbols: I3DNow, I3DNow_binop, I3DNow_conv, I3DNow_binop_rm, rr, rm. / 重要符号：I3DNow, I3DNow_binop, I3DNow_conv, I3DNow_binop_rm, rr, rm。
- This file is written in TableGen DSL and contributes generated target tables. / 该文件使用 TableGen DSL 编写，并参与生成目标描述表。

## Dependencies / 依赖关系
- Consumed by LLVM TableGen to emit generated matcher, encoding, register, or scheduling tables. / 由 LLVM TableGen 消费，以生成匹配器、编码、寄存器或调度相关表。
- Closely coupled with sibling X86 .td fragments that assemble the full target description. / 与同级 X86 .td 片段紧密协作，共同组成完整目标描述。
