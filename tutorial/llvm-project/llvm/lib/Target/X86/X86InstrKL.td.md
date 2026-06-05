# X86InstrKL.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/X86InstrKL.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines instruction definitions and metadata for the core X86 backend using LLVM TableGen DSL. / 使用 LLVM TableGen DSL 为X86 后端核心定义指令定义与元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: File header, licensing, and opening context / 文件头、许可证与开场上下文
```tablegen
//===---------------------------*-tablegen-*-------------------------------===//
//===------------- X86InstrKL.td - KL Instruction Set Extension -----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the instructions that make up the Intel key locker
// instruction set.
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
// Key Locker instructions
class Encodekey<bits<8> opcode, string m>
  : I<opcode, MRMSrcReg, (outs GR32:$dst), (ins GR32:$src), m#"\t{$src, $dst|$dst, $src}", []>,
    NoCD8, XS;

```
**EN:** This section uses LLVM TableGen DSL to describe file header, licensing, and opening context for the core X86 backend. Key symbols include Encodekey. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的文件头、许可证与开场上下文。关键符号包括 Encodekey。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 21-40: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
multiclass Aesencdec<string suffix> {
 def AESENC128KL#suffix : I<0xDC, MRMSrcMem, (outs VR128:$dst),
                            (ins VR128:$src1, opaquemem:$src2),
                            "aesenc128kl\t{$src2, $src1|$src1, $src2}",
                            [(set VR128:$dst, EFLAGS, (X86aesenc128kl VR128:$src1, addr:$src2))]>,
                          NoCD8, XS;
 def AESDEC128KL#suffix : I<0xDD, MRMSrcMem, (outs VR128:$dst),
                            (ins VR128:$src1, opaquemem:$src2),
                            "aesdec128kl\t{$src2, $src1|$src1, $src2}",
                            [(set VR128:$dst, EFLAGS, (X86aesdec128kl VR128:$src1, addr:$src2))]>,
                          NoCD8, XS;
 def AESENC256KL#suffix : I<0xDE, MRMSrcMem, (outs VR128:$dst),
                            (ins VR128:$src1, opaquemem:$src2),
                            "aesenc256kl\t{$src2, $src1|$src1, $src2}",
                            [(set VR128:$dst, EFLAGS, (X86aesenc256kl VR128:$src1, addr:$src2))]>,
                          NoCD8, XS;
 def AESDEC256KL#suffix : I<0xDF, MRMSrcMem, (outs VR128:$dst),
                            (ins VR128:$src1, opaquemem:$src2),
                            "aesdec256kl\t{$src2, $src1|$src1, $src2}",
                            [(set VR128:$dst, EFLAGS, (X86aesdec256kl VR128:$src1, addr:$src2))]>,
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include Aesencdec, AESENC128KL, AESDEC128KL, AESENC256KL, AESDEC256KL. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 Aesencdec, AESENC128KL, AESDEC128KL, AESENC256KL, AESDEC256KL。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 41-60: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
                          NoCD8, XS;
}

let SchedRW = [WriteSystem] in {
  let Uses = [XMM0, EAX], Defs = [EFLAGS], Predicates = [HasKL] in {
    def LOADIWKEY : I<0xDC, MRMSrcReg, (outs), (ins VR128:$src1, VR128:$src2),
                      "loadiwkey\t{$src2, $src1|$src1, $src2}",
                      [(int_x86_loadiwkey XMM0, VR128:$src1, VR128:$src2, EAX)]>, T8, XS;
  }

  let Predicates = [HasKL] in {
    let Uses = [XMM0], Defs = [XMM0, XMM1, XMM2, XMM4, XMM5, XMM6, EFLAGS] in
      def ENCODEKEY128 : Encodekey<0xFA, "encodekey128">, T8;

    let Uses = [XMM0, XMM1], Defs = [XMM0, XMM1, XMM2, XMM3, XMM4, XMM5, XMM6, EFLAGS] in
      def ENCODEKEY256 : Encodekey<0xFB, "encodekey256">, T8;

    let Constraints = "$src1 = $dst", Defs = [EFLAGS] in
      defm "" : Aesencdec<"">, T8;
  }
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include LOADIWKEY, ENCODEKEY128, ENCODEKEY256. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 LOADIWKEY, ENCODEKEY128, ENCODEKEY256。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 61-74: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
} // SchedRW

multiclass Aesencdecwide<string suffix> {
  def AESENCWIDE128KL#suffix : I<0xD8, MRM0m, (outs), (ins opaquemem:$src), "aesencwide128kl\t$src", []>, NoCD8, XS;
  def AESDECWIDE128KL#suffix : I<0xD8, MRM1m, (outs), (ins opaquemem:$src), "aesdecwide128kl\t$src", []>, NoCD8, XS;
  def AESENCWIDE256KL#suffix : I<0xD8, MRM2m, (outs), (ins opaquemem:$src), "aesencwide256kl\t$src", []>, NoCD8, XS;
  def AESDECWIDE256KL#suffix : I<0xD8, MRM3m, (outs), (ins opaquemem:$src), "aesdecwide256kl\t$src", []>, NoCD8, XS;
}

let SchedRW = [WriteSystem], Uses = [XMM0, XMM1, XMM2, XMM3, XMM4, XMM5, XMM6, XMM7],
    Defs = [EFLAGS, XMM0, XMM1, XMM2, XMM3, XMM4, XMM5, XMM6, XMM7], mayLoad = 1 in {
  let Predicates = [HasWIDEKL] in
    defm "" : Aesencdecwide<"">, T8;
} // SchedRW
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include Aesencdecwide, AESENCWIDE128KL, AESDECWIDE128KL, AESENCWIDE256KL, AESDECWIDE256KL. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 Aesencdecwide, AESENCWIDE128KL, AESDECWIDE128KL, AESENCWIDE256KL, AESDECWIDE256KL。这些记录随后会展开为 X86 后端使用的生成表。

## Key Concepts / 关键概念
- Primary topic: instruction definitions and metadata. / 核心主题：指令定义与元数据。
- Subsystem: the core X86 backend. / 所属子系统：X86 后端核心。
- Notable symbols: Encodekey, Aesencdec, AESENC128KL, AESDEC128KL, AESENC256KL, AESDEC256KL. / 重要符号：Encodekey, Aesencdec, AESENC128KL, AESDEC128KL, AESENC256KL, AESDEC256KL。
- This file is written in TableGen DSL and contributes generated target tables. / 该文件使用 TableGen DSL 编写，并参与生成目标描述表。

## Dependencies / 依赖关系
- Consumed by LLVM TableGen to emit generated matcher, encoding, register, or scheduling tables. / 由 LLVM TableGen 消费，以生成匹配器、编码、寄存器或调度相关表。
- Closely coupled with sibling X86 .td fragments that assemble the full target description. / 与同级 X86 .td 片段紧密协作，共同组成完整目标描述。
