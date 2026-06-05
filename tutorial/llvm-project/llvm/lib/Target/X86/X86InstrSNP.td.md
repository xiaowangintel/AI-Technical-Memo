# X86InstrSNP.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/X86InstrSNP.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines instruction definitions and metadata for the core X86 backend using LLVM TableGen DSL. / 使用 LLVM TableGen DSL 为X86 后端核心定义指令定义与元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: File header, licensing, and opening context / 文件头、许可证与开场上下文
```tablegen
//===-- X86InstrSNP.td - SNP Instruction Set Extension -----*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the instructions that make up the AMD Secure Nested
// Paging (SNP) instruction set.
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
// SNP instructions

let SchedRW = [WriteSystem] in {
// F3 0F 01 FF
let Uses = [RAX], Defs = [EAX, EFLAGS] in
def PSMASH: I<0x01, MRM_FF, (outs), (ins), "psmash", []>, TB, XS,
```
**EN:** This section uses LLVM TableGen DSL to describe file header, licensing, and opening context for the core X86 backend. Key symbols include PSMASH. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的文件头、许可证与开场上下文。关键符号包括 PSMASH。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 21-40: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
            Requires<[In64BitMode]>;

// F2 0F 01 FF
let Uses = [RAX, RCX, RDX], Defs = [EAX, EFLAGS] in
def PVALIDATE64: I<0x01, MRM_FF, (outs), (ins), "pvalidate",[]>,
                 TB, XD, Requires<[In64BitMode]>;

let Uses = [EAX, ECX, EDX], Defs = [EAX, EFLAGS] in
def PVALIDATE32: I<0x01, MRM_FF, (outs), (ins), "pvalidate",[]>,
                 TB, XD, Requires<[Not64BitMode]>;

// F2 0F 01 FE
let Uses = [RAX, RCX], Defs = [EAX, EFLAGS] in
def RMPUPDATE: I<0x01, MRM_FE, (outs), (ins), "rmpupdate", []>, TB, XD,
               Requires<[In64BitMode]>;

// F3 0F 01 FE
let Uses = [RAX, RCX, RDX], Defs = [EAX, EFLAGS] in
def RMPADJUST: I<0x01, MRM_FE, (outs), (ins), "rmpadjust", []>, TB, XS,
               Requires<[In64BitMode]>;
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include PVALIDATE64, PVALIDATE32, RMPUPDATE, RMPADJUST. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 PVALIDATE64, PVALIDATE32, RMPUPDATE, RMPADJUST。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 41-53: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen

// F3 0F 01 FD
let Uses = [RAX, RDX], Defs = [RAX, RCX, RDX, EFLAGS] in
def RMPQUERY: I<0x01, MRM_FD, (outs), (ins), "rmpquery", []>, TB, XS,
               Requires<[In64BitMode]>;
} // SchedRW

def : InstAlias<"psmash\t{%rax|rax}", (PSMASH)>, Requires<[In64BitMode]>;
def : InstAlias<"pvalidate\t{%rax, %rcx, %rdx|rdx, rcx, rax|}", (PVALIDATE64)>, Requires<[In64BitMode]>;
def : InstAlias<"pvalidate\t{%eax, %ecx, %edx|edx, ecx, eax|}", (PVALIDATE32)>, Requires<[Not64BitMode]>;
def : InstAlias<"rmpupdate\t{%rax, %rcx|rcx, rax|}", (RMPUPDATE)>, Requires<[In64BitMode]>;
def : InstAlias<"rmpadjust\t{%rax, %rcx, %rdx|rdx, rcx, rax|}", (RMPADJUST)>, Requires<[In64BitMode]>;
def : InstAlias<"rmpquery\t{%rax, %rdx|rdx, rax|}", (RMPQUERY)>, Requires<[In64BitMode]>;
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include RMPQUERY. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 RMPQUERY。这些记录随后会展开为 X86 后端使用的生成表。

## Key Concepts / 关键概念
- Primary topic: instruction definitions and metadata. / 核心主题：指令定义与元数据。
- Subsystem: the core X86 backend. / 所属子系统：X86 后端核心。
- Notable symbols: PSMASH, PVALIDATE64, PVALIDATE32, RMPUPDATE, RMPADJUST, RMPQUERY. / 重要符号：PSMASH, PVALIDATE64, PVALIDATE32, RMPUPDATE, RMPADJUST, RMPQUERY。
- This file is written in TableGen DSL and contributes generated target tables. / 该文件使用 TableGen DSL 编写，并参与生成目标描述表。

## Dependencies / 依赖关系
- Consumed by LLVM TableGen to emit generated matcher, encoding, register, or scheduling tables. / 由 LLVM TableGen 消费，以生成匹配器、编码、寄存器或调度相关表。
- Closely coupled with sibling X86 .td fragments that assemble the full target description. / 与同级 X86 .td 片段紧密协作，共同组成完整目标描述。
