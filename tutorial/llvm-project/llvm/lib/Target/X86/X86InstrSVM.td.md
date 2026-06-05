# X86InstrSVM.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/X86InstrSVM.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines instruction definitions and metadata for the core X86 backend using LLVM TableGen DSL. / 使用 LLVM TableGen DSL 为X86 后端核心定义指令定义与元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: File header, licensing, and opening context / 文件头、许可证与开场上下文
```tablegen
//===-- X86InstrSVM.td - SVM Instruction Set Extension -----*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the instructions that make up the AMD SVM instruction
// set.
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
// SVM instructions

let SchedRW = [WriteSystem] in {
// 0F 01 D9
def VMMCALL : I<0x01, MRM_D9, (outs), (ins), "vmmcall", []>, TB;

```
**EN:** This section uses LLVM TableGen DSL to describe file header, licensing, and opening context for the core X86 backend. Key symbols include VMMCALL. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的文件头、许可证与开场上下文。关键符号包括 VMMCALL。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 21-40: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
// 0F 01 DC
def STGI : I<0x01, MRM_DC, (outs), (ins), "stgi", []>, TB;

// 0F 01 DD
def CLGI : I<0x01, MRM_DD, (outs), (ins), "clgi", []>, TB;

// 0F 01 DE
let Uses = [EAX] in
def SKINIT : I<0x01, MRM_DE, (outs), (ins), "skinit", []>, TB;

// 0F 01 D8
let Uses = [EAX] in
def VMRUN32 : I<0x01, MRM_D8, (outs), (ins), "vmrun", []>, TB,
                Requires<[Not64BitMode]>;
let Uses = [RAX] in
def VMRUN64 : I<0x01, MRM_D8, (outs), (ins), "vmrun", []>, TB,
                Requires<[In64BitMode]>;

// 0F 01 DA
let Uses = [EAX] in
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include STGI, CLGI, SKINIT, VMRUN32, VMRUN64. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 STGI, CLGI, SKINIT, VMRUN32, VMRUN64。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 41-60: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
def VMLOAD32 : I<0x01, MRM_DA, (outs), (ins), "vmload", []>, TB,
                 Requires<[Not64BitMode]>;
let Uses = [RAX] in
def VMLOAD64 : I<0x01, MRM_DA, (outs), (ins), "vmload", []>, TB,
                 Requires<[In64BitMode]>;

// 0F 01 DB
let Uses = [EAX] in
def VMSAVE32 : I<0x01, MRM_DB, (outs), (ins), "vmsave", []>, TB,
                 Requires<[Not64BitMode]>;
let Uses = [RAX] in
def VMSAVE64 : I<0x01, MRM_DB, (outs), (ins), "vmsave", []>, TB,
                 Requires<[In64BitMode]>;

// 0F 01 DF
let Uses = [EAX, ECX] in
def INVLPGA32 : I<0x01, MRM_DF, (outs), (ins),
                "invlpga", []>, TB, Requires<[Not64BitMode]>;
let Uses = [RAX, ECX] in
def INVLPGA64 : I<0x01, MRM_DF, (outs), (ins),
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include VMLOAD32, VMLOAD64, VMSAVE32, VMSAVE64, INVLPGA32, INVLPGA64. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 VMLOAD32, VMLOAD64, VMSAVE32, VMSAVE64, INVLPGA32, INVLPGA64。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 61-62: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
                "invlpga", []>, TB, Requires<[In64BitMode]>;
} // SchedRW
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。这些记录随后会展开为 X86 后端使用的生成表。

## Key Concepts / 关键概念
- Primary topic: instruction definitions and metadata. / 核心主题：指令定义与元数据。
- Subsystem: the core X86 backend. / 所属子系统：X86 后端核心。
- Notable symbols: VMMCALL, STGI, CLGI, SKINIT, VMRUN32, VMRUN64. / 重要符号：VMMCALL, STGI, CLGI, SKINIT, VMRUN32, VMRUN64。
- This file is written in TableGen DSL and contributes generated target tables. / 该文件使用 TableGen DSL 编写，并参与生成目标描述表。

## Dependencies / 依赖关系
- Consumed by LLVM TableGen to emit generated matcher, encoding, register, or scheduling tables. / 由 LLVM TableGen 消费，以生成匹配器、编码、寄存器或调度相关表。
- Closely coupled with sibling X86 .td fragments that assemble the full target description. / 与同级 X86 .td 片段紧密协作，共同组成完整目标描述。
