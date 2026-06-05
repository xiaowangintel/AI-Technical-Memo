# X86InstrSGX.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/X86InstrSGX.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines instruction definitions and metadata for the core X86 backend using LLVM TableGen DSL. / 使用 LLVM TableGen DSL 为X86 后端核心定义指令定义与元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: File header, licensing, and opening context / 文件头、许可证与开场上下文
```tablegen
//===-- X86InstrSGX.td - SGX Instruction Set Extension -----*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the instructions that make up the Intel SGX instruction
// set.
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
// SGX instructions

let SchedRW = [WriteSystem], Predicates = [HasSGX] in {
// ENCLS - Execute an Enclave System Function of Specified Leaf Number
def ENCLS : I<0x01, MRM_CF, (outs), (ins),
             "encls", []>, TB;
```
**EN:** This section uses LLVM TableGen DSL to describe file header, licensing, and opening context for the core X86 backend. Key symbols include ENCLS. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的文件头、许可证与开场上下文。关键符号包括 ENCLS。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 21-29: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen

// ENCLU - Execute an Enclave User Function of Specified Leaf Number
def ENCLU : I<0x01, MRM_D7, (outs), (ins),
             "enclu", []>, TB;

// ENCLV - Execute an Enclave VMM Function of Specified Leaf Number
def ENCLV : I<0x01, MRM_C0, (outs), (ins),
             "enclv", []>, TB;
} // SchedRW
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include ENCLU, ENCLV. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 ENCLU, ENCLV。这些记录随后会展开为 X86 后端使用的生成表。

## Key Concepts / 关键概念
- Primary topic: instruction definitions and metadata. / 核心主题：指令定义与元数据。
- Subsystem: the core X86 backend. / 所属子系统：X86 后端核心。
- Notable symbols: ENCLS, ENCLU, ENCLV. / 重要符号：ENCLS, ENCLU, ENCLV。
- This file is written in TableGen DSL and contributes generated target tables. / 该文件使用 TableGen DSL 编写，并参与生成目标描述表。

## Dependencies / 依赖关系
- Consumed by LLVM TableGen to emit generated matcher, encoding, register, or scheduling tables. / 由 LLVM TableGen 消费，以生成匹配器、编码、寄存器或调度相关表。
- Closely coupled with sibling X86 .td fragments that assemble the full target description. / 与同级 X86 .td 片段紧密协作，共同组成完整目标描述。
