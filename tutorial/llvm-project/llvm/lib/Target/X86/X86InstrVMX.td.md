# X86InstrVMX.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/X86InstrVMX.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines instruction definitions and metadata for the core X86 backend using LLVM TableGen DSL. / 使用 LLVM TableGen DSL 为X86 后端核心定义指令定义与元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: File header, licensing, and opening context / 文件头、许可证与开场上下文
```tablegen
//===-- X86InstrVMX.td - VMX Instruction Set Extension -----*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the instructions that make up the Intel VMX instruction
// set.
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
// VMX instructions

let SchedRW = [WriteSystem] in {
// 66 0F 38 80
def INVEPT32 : I<0x80, MRMSrcMem, (outs), (ins GR32:$src1, i128mem:$src2),
               "invept\t{$src2, $src1|$src1, $src2}", []>, T8, PD,
```
**EN:** This section uses LLVM TableGen DSL to describe file header, licensing, and opening context for the core X86 backend. Key symbols include INVEPT32. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的文件头、许可证与开场上下文。关键符号包括 INVEPT32。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 21-40: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
               Requires<[Not64BitMode]>;
def INVEPT64 : I<0x80, MRMSrcMem, (outs), (ins GR64:$src1, i128mem:$src2),
               "invept\t{$src2, $src1|$src1, $src2}", []>, T8, PD,
               Requires<[In64BitMode]>;
def INVEPT64_EVEX : I<0xF0, MRMSrcMem, (outs), (ins GR64:$src1, i128mem:$src2),
                      "invept\t{$src2, $src1|$src1, $src2}", []>,
                    EVEX, NoCD8, T_MAP4, XS, WIG, Requires<[In64BitMode]>;

// 66 0F 38 81
def INVVPID32 : I<0x81, MRMSrcMem, (outs), (ins GR32:$src1, i128mem:$src2),
                "invvpid\t{$src2, $src1|$src1, $src2}", []>, T8, PD,
                Requires<[Not64BitMode]>;
def INVVPID64 : I<0x81, MRMSrcMem, (outs), (ins GR64:$src1, i128mem:$src2),
                "invvpid\t{$src2, $src1|$src1, $src2}", []>, T8, PD,
                Requires<[In64BitMode]>;
def INVVPID64_EVEX : I<0xF1, MRMSrcMem, (outs), (ins GR64:$src1, i128mem:$src2),
                       "invvpid\t{$src2, $src1|$src1, $src2}", []>,
                     EVEX, NoCD8, T_MAP4, XS, WIG, Requires<[In64BitMode]>;

// 0F 01 C1
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include INVEPT64, INVEPT64_EVEX, INVVPID32, INVVPID64, INVVPID64_EVEX. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 INVEPT64, INVEPT64_EVEX, INVVPID32, INVVPID64, INVVPID64_EVEX。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 41-60: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
def VMCALL : I<0x01, MRM_C1, (outs), (ins), "vmcall", []>, TB;
def VMCLEARm : I<0xC7, MRM6m, (outs), (ins i64mem:$vmcs),
  "vmclear\t$vmcs", []>, TB, PD;

// OF 01 D4
def VMFUNC : I<0x01, MRM_D4, (outs), (ins), "vmfunc", []>, TB;

// 0F 01 C2
def VMLAUNCH : I<0x01, MRM_C2, (outs), (ins), "vmlaunch", []>, TB;

// 0F 01 C3
def VMRESUME : I<0x01, MRM_C3, (outs), (ins), "vmresume", []>, TB;
def VMPTRLDm : I<0xC7, MRM6m, (outs), (ins i64mem:$vmcs),
  "vmptrld\t$vmcs", []>, TB;
def VMPTRSTm : I<0xC7, MRM7m, (outs), (ins i64mem:$vmcs),
  "vmptrst\t$vmcs", []>, TB;
def VMREAD64rr : I<0x78, MRMDestReg, (outs GR64:$dst), (ins GR64:$src),
  "vmread{q}\t{$src, $dst|$dst, $src}", []>, TB, Requires<[In64BitMode]>;
def VMREAD32rr : I<0x78, MRMDestReg, (outs GR32:$dst), (ins GR32:$src),
  "vmread{l}\t{$src, $dst|$dst, $src}", []>, TB, Requires<[Not64BitMode]>;
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include VMCALL, VMCLEARm, VMFUNC, VMLAUNCH, VMRESUME, VMPTRLDm. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 VMCALL, VMCLEARm, VMFUNC, VMLAUNCH, VMRESUME, VMPTRLDm。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 61-80: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen

let mayStore = 1 in {
def VMREAD64mr : I<0x78, MRMDestMem, (outs), (ins i64mem:$dst, GR64:$src),
  "vmread{q}\t{$src, $dst|$dst, $src}", []>, TB, Requires<[In64BitMode]>;
def VMREAD32mr : I<0x78, MRMDestMem, (outs), (ins i32mem:$dst, GR32:$src),
  "vmread{l}\t{$src, $dst|$dst, $src}", []>, TB, Requires<[Not64BitMode]>;
} // mayStore

def VMWRITE64rr : I<0x79, MRMSrcReg, (outs GR64:$dst), (ins GR64:$src),
  "vmwrite{q}\t{$src, $dst|$dst, $src}", []>, TB, Requires<[In64BitMode]>;
def VMWRITE32rr : I<0x79, MRMSrcReg, (outs GR32:$dst), (ins GR32:$src),
  "vmwrite{l}\t{$src, $dst|$dst, $src}", []>, TB, Requires<[Not64BitMode]>;

let mayLoad = 1 in {
def VMWRITE64rm : I<0x79, MRMSrcMem, (outs GR64:$dst), (ins i64mem:$src),
  "vmwrite{q}\t{$src, $dst|$dst, $src}", []>, TB, Requires<[In64BitMode]>;
def VMWRITE32rm : I<0x79, MRMSrcMem, (outs GR32:$dst), (ins i32mem:$src),
  "vmwrite{l}\t{$src, $dst|$dst, $src}", []>, TB, Requires<[Not64BitMode]>;
} // mayLoad

```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include VMREAD64mr, VMREAD32mr, VMWRITE64rr, VMWRITE32rr, VMWRITE64rm, VMWRITE32rm. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 VMREAD64mr, VMREAD32mr, VMWRITE64rr, VMWRITE32rr, VMWRITE64rm, VMWRITE32rm。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 81-85: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
// 0F 01 C4
def VMXOFF : I<0x01, MRM_C4, (outs), (ins), "vmxoff", []>, TB;
def VMXON : I<0xC7, MRM6m, (outs), (ins i64mem:$vmxon),
  "vmxon\t$vmxon", []>, TB, XS;
} // SchedRW
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include VMXOFF, VMXON. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 VMXOFF, VMXON。这些记录随后会展开为 X86 后端使用的生成表。

## Key Concepts / 关键概念
- Primary topic: instruction definitions and metadata. / 核心主题：指令定义与元数据。
- Subsystem: the core X86 backend. / 所属子系统：X86 后端核心。
- Notable symbols: INVEPT32, INVEPT64, INVEPT64_EVEX, INVVPID32, INVVPID64, INVVPID64_EVEX. / 重要符号：INVEPT32, INVEPT64, INVEPT64_EVEX, INVVPID32, INVVPID64, INVVPID64_EVEX。
- This file is written in TableGen DSL and contributes generated target tables. / 该文件使用 TableGen DSL 编写，并参与生成目标描述表。

## Dependencies / 依赖关系
- Consumed by LLVM TableGen to emit generated matcher, encoding, register, or scheduling tables. / 由 LLVM TableGen 消费，以生成匹配器、编码、寄存器或调度相关表。
- Closely coupled with sibling X86 .td fragments that assemble the full target description. / 与同级 X86 .td 片段紧密协作，共同组成完整目标描述。
