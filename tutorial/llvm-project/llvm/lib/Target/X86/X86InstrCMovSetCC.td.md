# X86InstrCMovSetCC.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/X86InstrCMovSetCC.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines instruction definitions and metadata for the core X86 backend using LLVM TableGen DSL. / 使用 LLVM TableGen DSL 为X86 后端核心定义指令定义与元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: File header, licensing, and opening context / 文件头、许可证与开场上下文
```tablegen
//===-- X86InstrCMovSetCC.td - Conditional Move and SetCC --*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the X86 conditional move and set on condition
// instructions.
//
//===----------------------------------------------------------------------===//


// CMOV instructions.
multiclass Cmov<X86TypeInfo t, string args, bit ndd = 0, string suffix = ""> {
let isCommutable = 1, SchedRW = [WriteCMOV] in
  def rr#suffix : ITy<0x40, MRMSrcRegCC, t, (outs t.RegClass:$dst),
                      (ins t.RegClass:$src1, t.RegClass:$src2, ccode:$cond),
                      "cmov${cond}", args,
                      [(set t.RegClass:$dst, (X86cmov t.RegClass:$src1,
                                        t.RegClass:$src2, timm:$cond, EFLAGS))]>, UseEFLAGS, NDD<ndd>;
let SchedRW = [WriteCMOV.Folded, WriteCMOV.ReadAfterFold] in
  def rm#suffix : ITy<0x40, MRMSrcMemCC, t, (outs t.RegClass:$dst),
                      (ins t.RegClass:$src1, t.MemOperand:$src2, ccode:$cond),
                      "cmov${cond}", args,
                      [(set t.RegClass:$dst, (X86cmov t.RegClass:$src1,
                                    (t.LoadNode !if(ndd, ndd_addr, addr):$src2), timm:$cond, EFLAGS))]>, UseEFLAGS, NDD<ndd>;
}

```
**EN:** This section uses LLVM TableGen DSL to describe file header, licensing, and opening context for the core X86 backend. Key symbols include Cmov, rr, rm. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的文件头、许可证与开场上下文。关键符号包括 Cmov, rr, rm。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 31-60: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
multiclass Cfcmov<X86TypeInfo t> {
let isCommutable = 1, SchedRW = [WriteCMOV] in {
let Predicates = [HasCMOV, HasCF, In64BitMode] in {
  def rr : ITy<0x40, MRMDestRegCC, t, (outs t.RegClass:$dst),
               (ins t.RegClass:$src1, ccode:$cond),
               "cfcmov${cond}", unaryop_ndd_args,
               [(set t.RegClass:$dst,
                 (X86cmov 0, t.RegClass:$src1, timm:$cond, EFLAGS))]>, UseEFLAGS, NF;
  def rr_REV : ITy<0x40, MRMSrcRegCC, t, (outs t.RegClass:$dst),
                   (ins t.RegClass:$src1, ccode:$cond),
                   "cfcmov${cond}", unaryop_ndd_args,
                   []>, UseEFLAGS, EVEX, T_MAP4;
}
let Predicates = [HasCMOV, HasCF, HasNDD, In64BitMode] in
  def rr_ND : ITy<0x40, MRMSrcRegCC, t, (outs t.RegClass:$dst),
                  (ins t.RegClass:$src1, t.RegClass:$src2, ccode:$cond),
                  "cfcmov${cond}", binop_ndd_args, []>, UseEFLAGS, NDD<1>, NF;
}
let SchedRW = [WriteCMOV.Folded, WriteCMOV.ReadAfterFold] in {
  let Predicates = [HasCMOV, HasCF, In64BitMode], mayLoad = 1 in
    def rm : ITy<0x40, MRMSrcMemCC, t, (outs t.RegClass:$dst),
                 (ins t.MemOperand:$src1, ccode:$cond),
                 "cfcmov${cond}", unaryop_ndd_args, []>, UseEFLAGS, EVEX, T_MAP4;
  let Predicates = [HasCMOV, HasCF, HasNDD, In64BitMode], mayLoad = 1 in
    def rm_ND : ITy<0x40, MRMSrcMemCC, t, (outs t.RegClass:$dst),
                    (ins t.RegClass:$src1, t.MemOperand:$src2, ccode:$cond),
                    "cfcmov${cond}", binop_ndd_args, []>, UseEFLAGS, NDD<1>, NF;
}
let SchedRW = [WriteCMOV, ReadDefault, ReadDefault, ReadDefault, ReadDefault, ReadDefault],
    Predicates = [HasCMOV, HasCF, In64BitMode], mayStore = 1 in
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include Cfcmov, rr, rr_REV, rr_ND, rm, rm_ND. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 Cfcmov, rr, rr_REV, rr_ND, rm, rm_ND。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 61-90: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
  def mr : ITy<0x40, MRMDestMemCC, t, (outs),
                (ins t.MemOperand:$dst, t.RegClass:$src1, ccode:$cond),
                "cfcmov${cond}", unaryop_ndd_args, []>, UseEFLAGS, NF;
}

let isCodeGenOnly = 1, ForceDisassemble = 1 in {
  let Predicates = [HasCMOV, NoNDD], Constraints = "$dst = $src1" in {
    defm CMOV16 : Cmov<Xi16, binop_args>, OpSize16, TB;
    defm CMOV32 : Cmov<Xi32, binop_args>, OpSize32, TB;
    defm CMOV64 : Cmov<Xi64, binop_args>, TB;
  }

  let Predicates = [HasCMOV, HasNDD, In64BitMode] in {
    defm CMOV16 : Cmov<Xi16, binop_ndd_args, 1, "_ND">, PD;
    defm CMOV32 : Cmov<Xi32, binop_ndd_args, 1, "_ND">;
    defm CMOV64 : Cmov<Xi64, binop_ndd_args, 1, "_ND">;
  }

  defm CFCMOV16 : Cfcmov<Xi16>, PD;
  defm CFCMOV32 : Cfcmov<Xi32>;
  defm CFCMOV64 : Cfcmov<Xi64>;
} // isCodeGenOnly = 1, ForceDisassemble = 1

def inv_cond_XFORM : SDNodeXForm<imm, [{
  X86::CondCode CC = static_cast<X86::CondCode>(N->getZExtValue());
  return CurDAG->getTargetConstant(X86::GetOppositeBranchCondition(CC),
                                   SDLoc(N), MVT::i8);
}]>;

// Conditional moves with folded loads with operands swapped and conditions
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include mr, CMOV16, CMOV32, CMOV64, CFCMOV16, CFCMOV32. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 mr, CMOV16, CMOV32, CMOV64, CFCMOV16, CFCMOV32。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 91-120: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
// inverted.
let Predicates = [HasCMOV, NoNDD] in {
  def : Pat<(X86cmov (loadi16 addr:$src1), GR16:$src2, timm:$cond, EFLAGS),
            (CMOV16rm GR16:$src2, addr:$src1, (inv_cond_XFORM timm:$cond))>;
  def : Pat<(X86cmov (loadi32 addr:$src1), GR32:$src2, timm:$cond, EFLAGS),
            (CMOV32rm GR32:$src2, addr:$src1, (inv_cond_XFORM timm:$cond))>;
  def : Pat<(X86cmov (loadi64 addr:$src1), GR64:$src2, timm:$cond, EFLAGS),
            (CMOV64rm GR64:$src2, addr:$src1, (inv_cond_XFORM timm:$cond))>;
}

let Predicates = [HasCMOV, HasNDD] in {
  def : Pat<(X86cmov (loadi16 ndd_addr:$src1), GR16:$src2, timm:$cond, EFLAGS),
            (CMOV16rm_ND GR16:$src2, ndd_addr:$src1, (inv_cond_XFORM timm:$cond))>;
  def : Pat<(X86cmov (loadi32 ndd_addr:$src1), GR32:$src2, timm:$cond, EFLAGS),
            (CMOV32rm_ND GR32:$src2, ndd_addr:$src1, (inv_cond_XFORM timm:$cond))>;
  def : Pat<(X86cmov (loadi64 ndd_addr:$src1), GR64:$src2, timm:$cond, EFLAGS),
            (CMOV64rm_ND GR64:$src2, ndd_addr:$src1, (inv_cond_XFORM timm:$cond))>;
}
let Predicates = [HasCMOV, HasCF] in {
  def : Pat<(X86cmov GR16:$src1, 0, timm:$cond, EFLAGS),
            (CFCMOV16rr GR16:$src1, (inv_cond_XFORM timm:$cond))>;
  def : Pat<(X86cmov GR32:$src1, 0, timm:$cond, EFLAGS),
            (CFCMOV32rr GR32:$src1, (inv_cond_XFORM timm:$cond))>;
  def : Pat<(X86cmov GR64:$src1, 0, timm:$cond, EFLAGS),
            (CFCMOV64rr GR64:$src1, (inv_cond_XFORM timm:$cond))>;

  def : Pat<(X86cload addr:$src1, 0, timm:$cond, EFLAGS),
            (CFCMOV16rm addr:$src1, timm:$cond)>;
  def : Pat<(X86cload addr:$src1, 0, timm:$cond, EFLAGS),
            (CFCMOV32rm addr:$src1, timm:$cond)>;
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 121-150: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
  def : Pat<(X86cload addr:$src1, 0, timm:$cond, EFLAGS),
            (CFCMOV64rm addr:$src1, timm:$cond)>;

  def : Pat<(X86cload addr:$src2, GR16:$src1, timm:$cond, EFLAGS),
            (CFCMOV16rm_ND GR16:$src1, addr:$src2, timm:$cond)>;
  def : Pat<(X86cload addr:$src2, GR32:$src1, timm:$cond, EFLAGS),
            (CFCMOV32rm_ND GR32:$src1, addr:$src2, timm:$cond)>;
  def : Pat<(X86cload addr:$src2, GR64:$src1, timm:$cond, EFLAGS),
            (CFCMOV64rm_ND GR64:$src1, addr:$src2, timm:$cond)>;

  def : Pat<(X86cstore GR16:$src2, addr:$src1, timm:$cond, EFLAGS),
            (CFCMOV16mr addr:$src1, GR16:$src2, timm:$cond)>;
  def : Pat<(X86cstore GR32:$src2, addr:$src1, timm:$cond, EFLAGS),
            (CFCMOV32mr addr:$src1, GR32:$src2, timm:$cond)>;
  def : Pat<(X86cstore GR64:$src2, addr:$src1, timm:$cond, EFLAGS),
            (CFCMOV64mr addr:$src1, GR64:$src2, timm:$cond)>;
}

// SetCC instructions.
let Uses = [EFLAGS], isCodeGenOnly = 1, ForceDisassemble = 1,
  Predicates = [PreferLegacySetCC] in
  def SETCCr : I<0x90, MRMXrCC, (outs GR8:$dst), (ins ccode:$cond),
                "set${cond}\t$dst",
                [(set GR8:$dst, (X86setcc timm:$cond, EFLAGS))]>,
                TB, Sched<[WriteSETCC]>;
let Uses = [EFLAGS], isCodeGenOnly = 1, ForceDisassemble = 1 in
  def SETCCm : I<0x90, MRMXmCC, (outs), (ins i8mem:$dst, ccode:$cond),
                "set${cond}\t$dst",
                [(store (X86setcc timm:$cond, EFLAGS), addr:$dst)]>,
                TB, Sched<[WriteSETCCStore]>;
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include SETCCr, SETCCm. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 SETCCr, SETCCm。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 151-179: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen

// SetZUCC and promoted SetCC instructions.
let Uses = [EFLAGS], isCodeGenOnly = 1, ForceDisassemble = 1,
  hasSideEffects = 0, Predicates = [In64BitMode, PreferNoLegacySetCC] in
  def SETZUCCr : I<0x40, MRMXrCC, (outs GR8:$dst), (ins ccode:$cond),
                "setzu${cond}\t$dst",
                [(set GR8:$dst, (X86setcc timm:$cond, EFLAGS))]>,
                XD, ZU, NoCD8, Sched<[WriteSETCC]>;
let Uses = [EFLAGS], isCodeGenOnly = 1, ForceDisassemble = 1,
  hasSideEffects = 0, Predicates = [In64BitMode] in {
  def SETCCr_EVEX : I<0x40, MRMXrCC, (outs GR8:$dst), (ins ccode:$cond),
                "set${cond}\t$dst", []>,
                XD, PL, Sched<[WriteSETCC]>;
  let mayStore = 1 in {
    def SETZUCCm : I<0x40, MRMXmCC, (outs), (ins i8mem:$dst, ccode:$cond),
                  "setzu${cond}\t$dst", []>,
                  XD, ZU, NoCD8, Sched<[WriteSETCCStore]>;
    def SETCCm_EVEX : I<0x40, MRMXmCC, (outs), (ins i8mem:$dst, ccode:$cond),
                  "set${cond}\t$dst", []>,
                  XD, PL, Sched<[WriteSETCCStore]>;
  }
}

// SALC is an undocumented instruction. Information for this instruction can be found
// here http://www.rcollins.org/secrets/opcodes/SALC.html
// Set AL if carry. 
let Uses = [EFLAGS], Defs = [AL], SchedRW = [WriteALU] in {
  def SALC : I<0xD6, RawFrm, (outs), (ins), "salc", []>, Requires<[Not64BitMode]>;
}
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include SETZUCCr, SETCCr_EVEX, SETZUCCm, SETCCm_EVEX, SALC. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 SETZUCCr, SETCCr_EVEX, SETZUCCm, SETCCm_EVEX, SALC。这些记录随后会展开为 X86 后端使用的生成表。

## Key Concepts / 关键概念
- Primary topic: instruction definitions and metadata. / 核心主题：指令定义与元数据。
- Subsystem: the core X86 backend. / 所属子系统：X86 后端核心。
- Notable symbols: Cmov, rr, rm, Cfcmov, rr_REV, rr_ND. / 重要符号：Cmov, rr, rm, Cfcmov, rr_REV, rr_ND。
- This file is written in TableGen DSL and contributes generated target tables. / 该文件使用 TableGen DSL 编写，并参与生成目标描述表。

## Dependencies / 依赖关系
- Consumed by LLVM TableGen to emit generated matcher, encoding, register, or scheduling tables. / 由 LLVM TableGen 消费，以生成匹配器、编码、寄存器或调度相关表。
- Closely coupled with sibling X86 .td fragments that assemble the full target description. / 与同级 X86 .td 片段紧密协作，共同组成完整目标描述。
