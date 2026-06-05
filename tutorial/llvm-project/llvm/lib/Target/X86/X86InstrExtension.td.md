# X86InstrExtension.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/X86InstrExtension.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines instruction definitions and metadata for the core X86 backend using LLVM TableGen DSL. / 使用 LLVM TableGen DSL 为X86 后端核心定义指令定义与元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: File header, licensing, and opening context / 文件头、许可证与开场上下文
```tablegen
//===-- X86InstrExtension.td - Sign and Zero Extensions ----*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the sign and zero extension operations.
//
//===----------------------------------------------------------------------===//

let hasSideEffects = 0 in {
  let Defs = [AX], Uses = [AL] in // AX = signext(AL)
  def CBW : I<0x98, RawFrm, (outs), (ins),
              "{cbtw|cbw}", []>, OpSize16, Sched<[WriteALU]>;
  let Defs = [EAX], Uses = [AX] in // EAX = signext(AX)
  def CWDE : I<0x98, RawFrm, (outs), (ins),
              "{cwtl|cwde}", []>, OpSize32, Sched<[WriteALU]>;
  let Defs = [RAX], Uses = [EAX] in // RAX = signext(EAX)
  def CDQE : RI<0x98, RawFrm, (outs), (ins),
               "{cltq|cdqe}", []>, Sched<[WriteALU]>, Requires<[In64BitMode]>;

  // FIXME: CWD/CDQ/CQO shouldn't Def the A register, but the fast register
  // allocator crashes if you remove it.
  let Defs = [AX,DX], Uses = [AX] in // DX:AX = signext(AX)
  def CWD : I<0x99, RawFrm, (outs), (ins),
              "{cwtd|cwd}", []>, OpSize16, Sched<[WriteALU]>;
  let Defs = [EAX,EDX], Uses = [EAX] in // EDX:EAX = signext(EAX)
  def CDQ : I<0x99, RawFrm, (outs), (ins),
```
**EN:** This section uses LLVM TableGen DSL to describe file header, licensing, and opening context for the core X86 backend. Key symbols include CBW, CWDE, CDQE, CWD, CDQ. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的文件头、许可证与开场上下文。关键符号包括 CBW, CWDE, CDQE, CWD, CDQ。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 31-60: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
              "{cltd|cdq}", []>, OpSize32, Sched<[WriteALU]>;
  let Defs = [RAX,RDX], Uses = [RAX] in // RDX:RAX = signext(RAX)
  def CQO  : RI<0x99, RawFrm, (outs), (ins),
                "{cqto|cqo}", []>, Sched<[WriteALU]>, Requires<[In64BitMode]>;
}

// Sign/Zero extenders
let hasSideEffects = 0 in {
def MOVSX16rr8 : I<0xBE, MRMSrcReg, (outs GR16:$dst), (ins GR8:$src),
                   "movs{bw|x}\t{$src, $dst|$dst, $src}", []>,
                   TB, OpSize16, Sched<[WriteALU]>;
let mayLoad = 1 in
def MOVSX16rm8 : I<0xBE, MRMSrcMem, (outs GR16:$dst), (ins i8mem:$src),
                   "movs{bw|x}\t{$src, $dst|$dst, $src}", []>,
                   TB, OpSize16, Sched<[WriteLoad]>;
} // hasSideEffects = 0
def MOVSX32rr8 : I<0xBE, MRMSrcReg, (outs GR32:$dst), (ins GR8:$src),
                   "movs{bl|x}\t{$src, $dst|$dst, $src}",
                   [(set GR32:$dst, (sext GR8:$src))]>, TB,
                   OpSize32, Sched<[WriteALU]>;
def MOVSX32rm8 : I<0xBE, MRMSrcMem, (outs GR32:$dst), (ins i8mem :$src),
                   "movs{bl|x}\t{$src, $dst|$dst, $src}",
                   [(set GR32:$dst, (sextloadi32i8 addr:$src))]>, TB,
                   OpSize32, Sched<[WriteLoad]>;
def MOVSX32rr16: I<0xBF, MRMSrcReg, (outs GR32:$dst), (ins GR16:$src),
                   "movs{wl|x}\t{$src, $dst|$dst, $src}",
                   [(set GR32:$dst, (sext GR16:$src))]>, TB,
                   OpSize32, Sched<[WriteALU]>;
def MOVSX32rm16: I<0xBF, MRMSrcMem, (outs GR32:$dst), (ins i16mem:$src),
                   "movs{wl|x}\t{$src, $dst|$dst, $src}",
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include CQO, MOVSX16rr8, MOVSX16rm8, MOVSX32rr8, MOVSX32rm8, MOVSX32rr16. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 CQO, MOVSX16rr8, MOVSX16rm8, MOVSX32rr8, MOVSX32rm8, MOVSX32rr16。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 61-90: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
                   [(set GR32:$dst, (sextloadi32i16 addr:$src))]>,
                   OpSize32, TB, Sched<[WriteLoad]>;

let hasSideEffects = 0 in {
def MOVZX16rr8 : I<0xB6, MRMSrcReg, (outs GR16:$dst), (ins GR8:$src),
                   "movz{bw|x}\t{$src, $dst|$dst, $src}", []>,
                   TB, OpSize16, Sched<[WriteALU]>;
let mayLoad = 1 in
def MOVZX16rm8 : I<0xB6, MRMSrcMem, (outs GR16:$dst), (ins i8mem:$src),
                   "movz{bw|x}\t{$src, $dst|$dst, $src}", []>,
                   TB, OpSize16, Sched<[WriteLoad]>;
} // hasSideEffects = 0
def MOVZX32rr8 : I<0xB6, MRMSrcReg, (outs GR32:$dst), (ins GR8 :$src),
                   "movz{bl|x}\t{$src, $dst|$dst, $src}",
                   [(set GR32:$dst, (zext GR8:$src))]>, TB,
                   OpSize32, Sched<[WriteALU]>;
def MOVZX32rm8 : I<0xB6, MRMSrcMem, (outs GR32:$dst), (ins i8mem :$src),
                   "movz{bl|x}\t{$src, $dst|$dst, $src}",
                   [(set GR32:$dst, (zextloadi32i8 addr:$src))]>, TB,
                   OpSize32, Sched<[WriteLoad]>;
def MOVZX32rr16: I<0xB7, MRMSrcReg, (outs GR32:$dst), (ins GR16:$src),
                   "movz{wl|x}\t{$src, $dst|$dst, $src}",
                   [(set GR32:$dst, (zext GR16:$src))]>, TB,
                   OpSize32, Sched<[WriteALU]>;
def MOVZX32rm16: I<0xB7, MRMSrcMem, (outs GR32:$dst), (ins i16mem:$src),
                   "movz{wl|x}\t{$src, $dst|$dst, $src}",
                   [(set GR32:$dst, (zextloadi32i16 addr:$src))]>,
                   TB, OpSize32, Sched<[WriteLoad]>;

// These instructions exist as a consequence of operand size prefix having
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include MOVZX16rr8, MOVZX16rm8, MOVZX32rr8, MOVZX32rm8, MOVZX32rr16, MOVZX32rm16. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 MOVZX16rr8, MOVZX16rm8, MOVZX32rr8, MOVZX32rm8, MOVZX32rr16, MOVZX32rm16。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 91-120: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
// control of the destination size, but not the input size. Only support them
// for the disassembler.
let isCodeGenOnly = 1, ForceDisassemble = 1, hasSideEffects = 0 in {
def MOVSX16rr16: I<0xBF, MRMSrcReg, (outs GR16:$dst), (ins GR16:$src),
                   "movs{ww|x}\t{$src, $dst|$dst, $src}",
                   []>, TB, OpSize16, Sched<[WriteALU]>;
def MOVZX16rr16: I<0xB7, MRMSrcReg, (outs GR16:$dst), (ins GR16:$src),
                   "movz{ww|x}\t{$src, $dst|$dst, $src}",
                   []>, TB, OpSize16, Sched<[WriteALU]>;
let mayLoad = 1 in {
def MOVSX16rm16: I<0xBF, MRMSrcMem, (outs GR16:$dst), (ins i16mem:$src),
                   "movs{ww|x}\t{$src, $dst|$dst, $src}",
                   []>, OpSize16, TB, Sched<[WriteLoad]>;
def MOVZX16rm16: I<0xB7, MRMSrcMem, (outs GR16:$dst), (ins i16mem:$src),
                   "movz{ww|x}\t{$src, $dst|$dst, $src}",
                   []>, TB, OpSize16, Sched<[WriteLoad]>;
} // mayLoad = 1
} // isCodeGenOnly = 1, ForceDisassemble = 1, hasSideEffects = 0

// These are the same as the regular MOVZX32rr8 and MOVZX32rm8
// except that they use GR32_NOREX for the output operand register class
// instead of GR32. This allows them to operate on h registers on x86-64.
let hasSideEffects = 0, isCodeGenOnly = 1 in {
def MOVZX32rr8_NOREX : I<0xB6, MRMSrcReg,
                         (outs GR32_NOREX:$dst), (ins GR8_NOREX:$src),
                         "movz{bl|x}\t{$src, $dst|$dst, $src}",
                         []>, TB, OpSize32, Sched<[WriteALU]>;
let mayLoad = 1 in
def MOVZX32rm8_NOREX : I<0xB6, MRMSrcMem,
                         (outs GR32_NOREX:$dst), (ins i8mem_NOREX:$src),
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include MOVSX16rr16, MOVZX16rr16, MOVSX16rm16, MOVZX16rm16, MOVZX32rr8_NOREX, MOVZX32rm8_NOREX. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 MOVSX16rr16, MOVZX16rr16, MOVSX16rm16, MOVZX16rm16, MOVZX32rr8_NOREX, MOVZX32rm8_NOREX。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 121-150: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
                         "movz{bl|x}\t{$src, $dst|$dst, $src}",
                         []>, TB, OpSize32, Sched<[WriteLoad]>;

def MOVSX32rr8_NOREX : I<0xBE, MRMSrcReg,
                         (outs GR32_NOREX:$dst), (ins GR8_NOREX:$src),
                         "movs{bl|x}\t{$src, $dst|$dst, $src}",
                         []>, TB, OpSize32, Sched<[WriteALU]>;
let mayLoad = 1 in
def MOVSX32rm8_NOREX : I<0xBE, MRMSrcMem,
                         (outs GR32_NOREX:$dst), (ins i8mem_NOREX:$src),
                         "movs{bl|x}\t{$src, $dst|$dst, $src}",
                         []>, TB, OpSize32, Sched<[WriteLoad]>;
}

// MOVSX64rr8 always has a REX prefix and it has an 8-bit register
// operand, which makes it a rare instruction with an 8-bit register
// operand that can never access an h register. If support for h registers
// were generalized, this would require a special register class.
def MOVSX64rr8 : RI<0xBE, MRMSrcReg, (outs GR64:$dst), (ins GR8 :$src),
                    "movs{bq|x}\t{$src, $dst|$dst, $src}",
                    [(set GR64:$dst, (sext GR8:$src))]>, TB,
                    Sched<[WriteALU]>;
def MOVSX64rm8 : RI<0xBE, MRMSrcMem, (outs GR64:$dst), (ins i8mem :$src),
                    "movs{bq|x}\t{$src, $dst|$dst, $src}",
                    [(set GR64:$dst, (sextloadi64i8 addr:$src))]>,
                    TB, Sched<[WriteLoad]>;
def MOVSX64rr16: RI<0xBF, MRMSrcReg, (outs GR64:$dst), (ins GR16:$src),
                    "movs{wq|x}\t{$src, $dst|$dst, $src}",
                    [(set GR64:$dst, (sext GR16:$src))]>, TB,
                    Sched<[WriteALU]>;
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include MOVSX32rr8_NOREX, MOVSX32rm8_NOREX, MOVSX64rr8, MOVSX64rm8, MOVSX64rr16. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 MOVSX32rr8_NOREX, MOVSX32rm8_NOREX, MOVSX64rr8, MOVSX64rm8, MOVSX64rr16。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 151-180: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
def MOVSX64rm16: RI<0xBF, MRMSrcMem, (outs GR64:$dst), (ins i16mem:$src),
                    "movs{wq|x}\t{$src, $dst|$dst, $src}",
                    [(set GR64:$dst, (sextloadi64i16 addr:$src))]>,
                    TB, Sched<[WriteLoad]>;
def MOVSX64rr32: RI<0x63, MRMSrcReg, (outs GR64:$dst), (ins GR32:$src),
                    "movs{lq|xd}\t{$src, $dst|$dst, $src}",
                    [(set GR64:$dst, (sext GR32:$src))]>,
                    Sched<[WriteALU]>, Requires<[In64BitMode]>;
def MOVSX64rm32: RI<0x63, MRMSrcMem, (outs GR64:$dst), (ins i32mem:$src),
                    "movs{lq|xd}\t{$src, $dst|$dst, $src}",
                    [(set GR64:$dst, (sextloadi64i32 addr:$src))]>,
                    Sched<[WriteLoad]>, Requires<[In64BitMode]>;

// These instructions exist as a consequence of operand size prefix having
// control of the destination size, but not the input size. Only support them
// for the disassembler.
let isCodeGenOnly = 1, ForceDisassemble = 1, hasSideEffects = 0 in {
def MOVSX16rr32: I<0x63, MRMSrcReg, (outs GR16:$dst), (ins GR32:$src),
                   "movs{lq|xd}\t{$src, $dst|$dst, $src}", []>,
                   Sched<[WriteALU]>, OpSize16, Requires<[In64BitMode]>;
def MOVSX32rr32: I<0x63, MRMSrcReg, (outs GR32:$dst), (ins GR32:$src),
                   "movs{lq|xd}\t{$src, $dst|$dst, $src}", []>,
                   Sched<[WriteALU]>, OpSize32, Requires<[In64BitMode]>;
let mayLoad = 1 in {
def MOVSX16rm32: I<0x63, MRMSrcMem, (outs GR16:$dst), (ins i32mem:$src),
                   "movs{lq|xd}\t{$src, $dst|$dst, $src}", []>,
                   Sched<[WriteLoad]>, OpSize16, Requires<[In64BitMode]>;
def MOVSX32rm32: I<0x63, MRMSrcMem, (outs GR32:$dst), (ins i32mem:$src),
                   "movs{lq|xd}\t{$src, $dst|$dst, $src}", []>,
                   Sched<[WriteLoad]>, OpSize32, Requires<[In64BitMode]>;
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include MOVSX64rm16, MOVSX64rr32, MOVSX64rm32, MOVSX16rr32, MOVSX32rr32, MOVSX16rm32. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 MOVSX64rm16, MOVSX64rr32, MOVSX64rm32, MOVSX16rr32, MOVSX32rr32, MOVSX16rm32。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 181-210: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
} // mayLoad = 1
} // isCodeGenOnly = 1, ForceDisassemble = 1, hasSideEffects = 0

// movzbq and movzwq encodings for the disassembler
let hasSideEffects = 0 in {
def MOVZX64rr8 : RI<0xB6, MRMSrcReg, (outs GR64:$dst), (ins GR8:$src),
                     "movz{bq|x}\t{$src, $dst|$dst, $src}", []>,
                     TB, Sched<[WriteALU]>;
let mayLoad = 1 in
def MOVZX64rm8 : RI<0xB6, MRMSrcMem, (outs GR64:$dst), (ins i8mem:$src),
                     "movz{bq|x}\t{$src, $dst|$dst, $src}", []>,
                     TB, Sched<[WriteLoad]>;
def MOVZX64rr16 : RI<0xB7, MRMSrcReg, (outs GR64:$dst), (ins GR16:$src),
                     "movz{wq|x}\t{$src, $dst|$dst, $src}", []>,
                     TB, Sched<[WriteALU]>;
let mayLoad = 1 in
def MOVZX64rm16 : RI<0xB7, MRMSrcMem, (outs GR64:$dst), (ins i16mem:$src),
                     "movz{wq|x}\t{$src, $dst|$dst, $src}", []>,
                     TB, Sched<[WriteLoad]>;
}

// 64-bit zero-extension patterns use SUBREG_TO_REG and an operation writing a
// 32-bit register.
def : Pat<(i64 (zext GR8:$src)),
          (SUBREG_TO_REG (MOVZX32rr8 GR8:$src), sub_32bit)>;
def : Pat<(zextloadi64i8 addr:$src),
          (SUBREG_TO_REG (MOVZX32rm8 addr:$src), sub_32bit)>;

def : Pat<(i64 (zext GR16:$src)),
          (SUBREG_TO_REG (MOVZX32rr16 GR16:$src), sub_32bit)>;
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include MOVZX64rr8, MOVZX64rm8, MOVZX64rr16, MOVZX64rm16. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 MOVZX64rr8, MOVZX64rm8, MOVZX64rr16, MOVZX64rm16。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 211-222: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
def : Pat<(zextloadi64i16 addr:$src),
          (SUBREG_TO_REG (MOVZX32rm16 addr:$src), sub_32bit)>;

// The preferred way to do 32-bit-to-64-bit zero extension on x86-64 is to use a
// SUBREG_TO_REG to utilize implicit zero-extension, however this isn't possible
// when the 32-bit value is defined by a truncate or is copied from something
// where the high bits aren't necessarily all zero. In such cases, we fall back
// to these explicit zext instructions.
def : Pat<(i64 (zext GR32:$src)),
          (SUBREG_TO_REG (MOV32rr GR32:$src), sub_32bit)>;
def : Pat<(i64 (zextloadi64i32 addr:$src)),
          (SUBREG_TO_REG (MOV32rm addr:$src), sub_32bit)>;
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。这些记录随后会展开为 X86 后端使用的生成表。

## Key Concepts / 关键概念
- Primary topic: instruction definitions and metadata. / 核心主题：指令定义与元数据。
- Subsystem: the core X86 backend. / 所属子系统：X86 后端核心。
- Notable symbols: CBW, CWDE, CDQE, CWD, CDQ, CQO. / 重要符号：CBW, CWDE, CDQE, CWD, CDQ, CQO。
- This file is written in TableGen DSL and contributes generated target tables. / 该文件使用 TableGen DSL 编写，并参与生成目标描述表。

## Dependencies / 依赖关系
- Consumed by LLVM TableGen to emit generated matcher, encoding, register, or scheduling tables. / 由 LLVM TableGen 消费，以生成匹配器、编码、寄存器或调度相关表。
- Closely coupled with sibling X86 .td fragments that assemble the full target description. / 与同级 X86 .td 片段紧密协作，共同组成完整目标描述。
