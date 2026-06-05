# X86InstrControl.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/X86InstrControl.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines instruction definitions and metadata for the core X86 backend using LLVM TableGen DSL. / 使用 LLVM TableGen DSL 为X86 后端核心定义指令定义与元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-50: File header, licensing, and opening context / 文件头、许可证与开场上下文
```tablegen
//===-- X86InstrControl.td - Control Flow Instructions -----*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the X86 jump, return, call, and related instructions.
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
//  Control Flow Instructions.
//

// Return instructions.
//
// The X86retglue return instructions are variadic because we may add ST0 and
// ST1 arguments when returning values on the x87 stack.
let isTerminator = 1, isReturn = 1, isBarrier = 1,
    hasCtrlDep = 1, FPForm = SpecialFP, SchedRW = [WriteJumpLd] in {
  def RET32  : I   <0xC3, RawFrm, (outs), (ins variable_ops),
                    "ret{l}", []>, OpSize32, Requires<[Not64BitMode]>;
  def RET64  : I   <0xC3, RawFrm, (outs), (ins variable_ops),
                    "ret{q}", []>, OpSize32, Requires<[In64BitMode]>;
  def RET16  : I   <0xC3, RawFrm, (outs), (ins),
                    "ret{w}", []>, OpSize16;
  def RETI32 : Ii16<0xC2, RawFrm, (outs), (ins i16imm:$amt, variable_ops),
                    "ret{l}\t$amt", []>, OpSize32, Requires<[Not64BitMode]>;
  def RETI64 : Ii16<0xC2, RawFrm, (outs), (ins i16imm:$amt, variable_ops),
                    "ret{q}\t$amt", []>, OpSize32, Requires<[In64BitMode]>;
  def RETI16 : Ii16<0xC2, RawFrm, (outs), (ins i16imm:$amt),
                    "ret{w}\t$amt", []>, OpSize16;
  def LRET32 : I   <0xCB, RawFrm, (outs), (ins),
                    "{l}ret{l|f}", []>, OpSize32;
  def LRET64 : RI  <0xCB, RawFrm, (outs), (ins),
                    "{l}ret{|f}q", []>, Requires<[In64BitMode]>;
  def LRET16 : I   <0xCB, RawFrm, (outs), (ins),
                    "{l}ret{w|f}", []>, OpSize16;
  def LRETI32 : Ii16<0xCA, RawFrm, (outs), (ins i16imm:$amt),
                     "{l}ret{l|f}\t$amt", []>, OpSize32;
  def LRETI64 : RIi16<0xCA, RawFrm, (outs), (ins i16imm:$amt),
                      "{l}ret{|f}q\t$amt", []>, Requires<[In64BitMode]>;
  def LRETI16 : Ii16<0xCA, RawFrm, (outs), (ins i16imm:$amt),
                     "{l}ret{w|f}\t$amt", []>, OpSize16;

  // The machine return from interrupt instruction, but sometimes we need to
  // perform a post-epilogue stack adjustment. Codegen emits the pseudo form
  // which expands to include an SP adjustment if necessary.
```
**EN:** This section uses LLVM TableGen DSL to describe file header, licensing, and opening context for the core X86 backend. Key symbols include RET32, RET64, RET16, RETI32, RETI64, RETI16. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的文件头、许可证与开场上下文。关键符号包括 RET32, RET64, RET16, RETI32, RETI64, RETI16。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 51-100: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
  def IRET16 : I   <0xcf, RawFrm, (outs), (ins), "iret{w}", []>,
               OpSize16;
  def IRET32 : I   <0xcf, RawFrm, (outs), (ins), "iret{l|d}", []>, OpSize32;
  def IRET64 : RI  <0xcf, RawFrm, (outs), (ins), "iretq", []>, Requires<[In64BitMode]>;
  let isCodeGenOnly = 1 in
  def IRET : PseudoI<(outs), (ins i32imm:$adj), [(X86iret timm:$adj)]>;
  def RET  : PseudoI<(outs), (ins i32imm:$adj, variable_ops), [(X86retglue timm:$adj)]>;
}

// Unconditional branches.
let isBarrier = 1, isBranch = 1, isTerminator = 1, SchedRW = [WriteJump] in {
  def JMP_1 : Ii8PCRel<0xEB, RawFrm, (outs), (ins brtarget8:$dst),
                       "jmp\t$dst", [(br bb:$dst)]>;
  let hasSideEffects = 0, isCodeGenOnly = 1, ForceDisassemble = 1 in {
    def JMP_2 : Ii16PCRel<0xE9, RawFrm, (outs), (ins brtarget16:$dst),
                          "jmp\t$dst", []>, OpSize16;
    def JMP_4 : Ii32PCRel<0xE9, RawFrm, (outs), (ins brtarget32:$dst),
                          "jmp\t$dst", []>, OpSize32;
  }
}

// Conditional Branches.
let isBranch = 1, isTerminator = 1, Uses = [EFLAGS], SchedRW = [WriteJump],
    isCodeGenOnly = 1, ForceDisassemble = 1 in {
  def JCC_1 : Ii8PCRel <0x70, AddCCFrm, (outs),
                        (ins brtarget8:$dst, ccode:$cond),
                        "j${cond}\t$dst",
                        [(X86brcond bb:$dst, timm:$cond, EFLAGS)]>;
  let hasSideEffects = 0 in {
    def JCC_2 : Ii16PCRel<0x80, AddCCFrm, (outs),
                          (ins brtarget16:$dst, ccode:$cond),
                          "j${cond}\t$dst",
                          []>, OpSize16, TB;
    def JCC_4 : Ii32PCRel<0x80, AddCCFrm, (outs),
                          (ins brtarget32:$dst, ccode:$cond),
                          "j${cond}\t$dst",
                          []>, TB, OpSize32;
  }
}

let hasSideEffects = 1, Uses = [EFLAGS], SchedRW = [WriteJump] in
def JCC_SELF : PseudoI<(outs), (ins ccode:$cond),
                       [(X86brcond_self timm:$cond, EFLAGS)]>;

// jcx/jecx/jrcx instructions.
let isBranch = 1, isTerminator = 1, hasSideEffects = 0, SchedRW = [WriteJump] in {
  // These are the 32-bit versions of this instruction for the asmparser.  In
  // 32-bit mode, the address size prefix is jcxz and the unprefixed version is
  // jecxz.
  let Uses = [CX] in
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include IRET16, IRET32, IRET64, IRET, RET, JMP_1. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 IRET16, IRET32, IRET64, IRET, RET, JMP_1。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 101-150: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
    def JCXZ : Ii8PCRel<0xE3, RawFrm, (outs), (ins i8imm_brtarget:$dst),
                        "jcxz\t$dst", []>, AdSize16, Requires<[Not64BitMode]>;
  let Uses = [ECX] in
    def JECXZ : Ii8PCRel<0xE3, RawFrm, (outs), (ins i8imm_brtarget:$dst),
                        "jecxz\t$dst", []>, AdSize32;

  let Uses = [RCX] in
    def JRCXZ : Ii8PCRel<0xE3, RawFrm, (outs), (ins i8imm_brtarget:$dst),
                         "jrcxz\t$dst", []>, AdSize64, Requires<[In64BitMode]>;
}

// Indirect branches
let isBranch = 1, isTerminator = 1, isBarrier = 1, isIndirectBranch = 1 in {
  def JMP16r     : I<0xFF, MRM4r, (outs), (ins GR16:$dst), "jmp{w}\t{*}$dst",
                     [(brind GR16:$dst)]>, Requires<[Not64BitMode]>,
                     OpSize16, Sched<[WriteJump]>;
  def JMP16m     : I<0xFF, MRM4m, (outs), (ins i16mem:$dst), "jmp{w}\t{*}$dst",
                     [(brind (loadi16 addr:$dst))]>, Requires<[Not64BitMode]>,
                     OpSize16, Sched<[WriteJumpLd]>;

  def JMP32r     : I<0xFF, MRM4r, (outs), (ins GR32:$dst), "jmp{l}\t{*}$dst",
                     [(brind GR32:$dst)]>, Requires<[Not64BitMode]>,
                     OpSize32, Sched<[WriteJump]>;
  def JMP32m     : I<0xFF, MRM4m, (outs), (ins i32mem:$dst), "jmp{l}\t{*}$dst",
                     [(brind (loadi32 addr:$dst))]>, Requires<[Not64BitMode]>,
                     OpSize32, Sched<[WriteJumpLd]>;

  def JMP64r     : I<0xFF, MRM4r, (outs), (ins GR64:$dst), "jmp{q}\t{*}$dst",
                     [(brind GR64:$dst)]>, Requires<[In64BitMode]>,
                     Sched<[WriteJump]>;
  def JMP64m     : I<0xFF, MRM4m, (outs), (ins i64mem:$dst), "jmp{q}\t{*}$dst",
                     [(brind (loadi64 addr:$dst))]>, Requires<[In64BitMode]>,
                     Sched<[WriteJumpLd]>;

  // Win64 wants indirect jumps leaving the function to have a REX_W prefix.
  // These are switched from TAILJMPr/m64_REX in MCInstLower.
  let isCodeGenOnly = 1, hasREX_W = 1 in {
    def JMP64r_REX : I<0xFF, MRM4r, (outs), (ins GR64:$dst),
                       "rex64 jmp{q}\t{*}$dst", []>, Sched<[WriteJump]>;
    let mayLoad = 1 in
    def JMP64m_REX : I<0xFF, MRM4m, (outs), (ins i64mem:$dst),
                       "rex64 jmp{q}\t{*}$dst", []>, Sched<[WriteJumpLd]>;

  }

  // Non-tracking jumps for IBT, use with caution.
  let isCodeGenOnly = 1 in {
    def JMP16r_NT : I<0xFF, MRM4r, (outs), (ins GR16 : $dst), "jmp{w}\t{*}$dst",
                      [(X86NoTrackBrind GR16 : $dst)]>, Requires<[Not64BitMode]>,
                      OpSize16, Sched<[WriteJump]>, NOTRACK;
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include JCXZ, JECXZ, JRCXZ, JMP16r, JMP16m, JMP32r. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 JCXZ, JECXZ, JRCXZ, JMP16r, JMP16m, JMP32r。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 151-200: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen

    def JMP16m_NT : I<0xFF, MRM4m, (outs), (ins i16mem : $dst), "jmp{w}\t{*}$dst",
                      [(X86NoTrackBrind (loadi16 addr : $dst))]>,
                      Requires<[Not64BitMode]>, OpSize16, Sched<[WriteJumpLd]>,
                      NOTRACK;

    def JMP32r_NT : I<0xFF, MRM4r, (outs), (ins GR32 : $dst), "jmp{l}\t{*}$dst",
                      [(X86NoTrackBrind GR32 : $dst)]>, Requires<[Not64BitMode]>,
                      OpSize32, Sched<[WriteJump]>, NOTRACK;
    def JMP32m_NT : I<0xFF, MRM4m, (outs), (ins i32mem : $dst), "jmp{l}\t{*}$dst",
                      [(X86NoTrackBrind (loadi32 addr : $dst))]>,
                      Requires<[Not64BitMode]>, OpSize32, Sched<[WriteJumpLd]>,
                      NOTRACK;

    def JMP64r_NT : I<0xFF, MRM4r, (outs), (ins GR64 : $dst), "jmp{q}\t{*}$dst",
                      [(X86NoTrackBrind GR64 : $dst)]>, Requires<[In64BitMode]>,
                      Sched<[WriteJump]>, NOTRACK;
    def JMP64m_NT : I<0xFF, MRM4m, (outs), (ins i64mem : $dst), "jmp{q}\t{*}$dst",
                      [(X86NoTrackBrind(loadi64 addr : $dst))]>,
                      Requires<[In64BitMode]>, Sched<[WriteJumpLd]>, NOTRACK;
  }

  let Predicates = [Not64BitMode], AsmVariantName = "att" in {
    def FARJMP32i  : Iseg32<0xEA, RawFrmImm16, (outs),
                            (ins i32imm:$off, i16imm:$seg),
                            "ljmp{l}\t$seg, $off", []>,
                            OpSize32, Sched<[WriteJump]>;
    def FARJMP16i  : Iseg16<0xEA, RawFrmImm16, (outs),
                            (ins i16imm:$off, i16imm:$seg),
                            "ljmp{w}\t$seg, $off", []>,
                            OpSize16, Sched<[WriteJump]>;
  }
  let mayLoad = 1 in {
    def FARJMP64m  : RI<0xFF, MRM5m, (outs), (ins opaquemem:$dst),
                        "ljmp{q}\t{*}$dst", []>, Sched<[WriteJump]>, Requires<[In64BitMode]>;

    def FARJMP32m  : I<0xFF, MRM5m, (outs), (ins opaquemem:$dst),
                       "{l}jmp{l}\t{*}$dst", []>, OpSize32, Sched<[WriteJumpLd]>;
    let AsmVariantName = "att" in
    def FARJMP16m  : I<0xFF, MRM5m, (outs), (ins opaquemem:$dst),
                       "ljmp{w}\t{*}$dst", []>, OpSize16, Sched<[WriteJumpLd]>;
  }
}

def JMPABS64i : Ii64<0xA1, RawFrm, (outs), (ins i64imm:$dst), "jmpabs\t$dst", []>,
                ExplicitREX2Prefix, Requires<[In64BitMode]>, Sched<[WriteJumpLd]>;

// Loop instructions
let isBranch = 1, isTerminator = 1, SchedRW = [WriteJump] in {
  def LOOP   : Ii8PCRel<0xE2, RawFrm, (outs), (ins i8imm_brtarget:$dst),
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include JMP16m_NT, JMP32r_NT, JMP32m_NT, JMP64r_NT, JMP64m_NT, FARJMP32i. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 JMP16m_NT, JMP32r_NT, JMP32m_NT, JMP64r_NT, JMP64m_NT, FARJMP32i。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 201-250: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
                        "loop\t$dst", []>;
  def LOOPE  : Ii8PCRel<0xE1, RawFrm, (outs), (ins i8imm_brtarget:$dst),
                        "loope\t$dst", []>;
  def LOOPNE : Ii8PCRel<0xE0, RawFrm, (outs), (ins i8imm_brtarget:$dst),
                        "loopne\t$dst", []>;
}

//===----------------------------------------------------------------------===//
//  Call Instructions...
//
let isCall = 1 in
  // All calls clobber the non-callee saved registers. ESP is marked as
  // a use to prevent stack-pointer assignments that appear immediately
  // before calls from potentially appearing dead. Uses for argument
  // registers are added manually.
  let Uses = [ESP, SSP] in {
    def CALLpcrel32 : Ii32PCRel<0xE8, RawFrm,
                           (outs), (ins i32imm_brtarget:$dst),
                           "call{l}\t$dst", []>, OpSize32,
                      Requires<[Not64BitMode]>, Sched<[WriteJump]>;
    let hasSideEffects = 0 in
      def CALLpcrel16 : Ii16PCRel<0xE8, RawFrm,
                             (outs), (ins i16imm_brtarget:$dst),
                             "call{w}\t$dst", []>, OpSize16,
                        Requires<[Not64BitMode]>, Sched<[WriteJump]>;
    def CALL16r     : I<0xFF, MRM2r, (outs), (ins GR16:$dst),
                        "call{w}\t{*}$dst", [(X86call GR16:$dst)]>,
                      OpSize16, Requires<[Not64BitMode]>, Sched<[WriteJump]>;
    def CALL16m     : I<0xFF, MRM2m, (outs), (ins i16mem:$dst),
                        "call{w}\t{*}$dst", [(X86call (loadi16 addr:$dst))]>,
                        OpSize16, Requires<[Not64BitMode,FavorMemIndirectCall]>,
                        Sched<[WriteJumpLd]>;
    def CALL32r     : I<0xFF, MRM2r, (outs), (ins GR32:$dst),
                        "call{l}\t{*}$dst", [(X86call GR32:$dst)]>, OpSize32,
                        Requires<[Not64BitMode,NotUseIndirectThunkCalls]>,
                        Sched<[WriteJump]>;
    def CALL32m     : I<0xFF, MRM2m, (outs), (ins i32mem:$dst),
                        "call{l}\t{*}$dst", [(X86call (loadi32 addr:$dst))]>,
                        OpSize32,
                        Requires<[Not64BitMode,FavorMemIndirectCall,
                                  NotUseIndirectThunkCalls]>,
                        Sched<[WriteJumpLd]>;

    // Non-tracking calls for IBT, use with caution.
    let isCodeGenOnly = 1 in {
      def CALL16r_NT : I<0xFF, MRM2r, (outs), (ins GR16 : $dst),
                        "call{w}\t{*}$dst",[(X86NoTrackCall GR16 : $dst)]>,
                        OpSize16, Requires<[Not64BitMode]>, Sched<[WriteJump]>, NOTRACK;
      def CALL16m_NT : I<0xFF, MRM2m, (outs), (ins i16mem : $dst),
                        "call{w}\t{*}$dst",[(X86NoTrackCall(loadi16 addr : $dst))]>,
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include LOOPE, LOOPNE, CALLpcrel32, CALLpcrel16, CALL16r, CALL16m. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 LOOPE, LOOPNE, CALLpcrel32, CALLpcrel16, CALL16r, CALL16m。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 251-300: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
                        OpSize16, Requires<[Not64BitMode,FavorMemIndirectCall]>,
                        Sched<[WriteJumpLd]>, NOTRACK;
      def CALL32r_NT : I<0xFF, MRM2r, (outs), (ins GR32 : $dst),
                        "call{l}\t{*}$dst",[(X86NoTrackCall GR32 : $dst)]>,
                        OpSize32, Requires<[Not64BitMode]>, Sched<[WriteJump]>, NOTRACK;
      def CALL32m_NT : I<0xFF, MRM2m, (outs), (ins i32mem : $dst),
                        "call{l}\t{*}$dst",[(X86NoTrackCall(loadi32 addr : $dst))]>,
                        OpSize32, Requires<[Not64BitMode,FavorMemIndirectCall]>,
                        Sched<[WriteJumpLd]>, NOTRACK;
    }

    let Predicates = [Not64BitMode], AsmVariantName = "att" in {
      def FARCALL32i  : Iseg32<0x9A, RawFrmImm16, (outs),
                               (ins i32imm:$off, i16imm:$seg),
                               "lcall{l}\t$seg, $off", []>,
                               OpSize32, Sched<[WriteJump]>;
      def FARCALL16i  : Iseg16<0x9A, RawFrmImm16, (outs),
                               (ins i16imm:$off, i16imm:$seg),
                               "lcall{w}\t$seg, $off", []>,
                               OpSize16, Sched<[WriteJump]>;
    }

    let mayLoad = 1 in {
      def FARCALL32m  : I<0xFF, MRM3m, (outs), (ins opaquemem:$dst),
                          "{l}call{l}\t{*}$dst", []>, OpSize32, Sched<[WriteJumpLd]>;
      def FARCALL16m  : I<0xFF, MRM3m, (outs), (ins opaquemem:$dst),
                          "lcall{w}\t{*}$dst", []>, OpSize16, Sched<[WriteJumpLd]>;
    }
  }


// Tail call stuff.
let isCall = 1, isTerminator = 1, isReturn = 1, isBarrier = 1,
    isCodeGenOnly = 1, Uses = [ESP, SSP] in {
  def TCRETURNdi : PseudoI<(outs), (ins i32imm_brtarget:$dst, i32imm:$offset),
                           []>, Sched<[WriteJump]>;
  def TCRETURNri : PseudoI<(outs), (ins GR32_TC:$dst, i32imm:$offset),
                           []>, Sched<[WriteJump]>;

  def TCRETURN_HIPE32ri : PseudoI<(outs), (ins GR32:$dst, i32imm:$offset),
                                  []>, Sched<[WriteJump]>;

  let mayLoad = 1 in
  def TCRETURNmi : PseudoI<(outs), (ins i32mem_TC:$dst, i32imm:$offset),
                           []>, Sched<[WriteJumpLd]>;

  def TAILJMPd : PseudoI<(outs), (ins i32imm_brtarget:$dst),
                         []>, Sched<[WriteJump]>;

  def TAILJMPr : PseudoI<(outs), (ins ptr_rc_tailcall:$dst),
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include CALL32r_NT, CALL32m_NT, FARCALL32i, FARCALL16i, FARCALL32m, FARCALL16m. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 CALL32r_NT, CALL32m_NT, FARCALL32i, FARCALL16i, FARCALL32m, FARCALL16m。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 301-350: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
                         []>, Sched<[WriteJump]>;
  let mayLoad = 1 in
  def TAILJMPm : PseudoI<(outs), (ins i32mem_TC:$dst),
                         []>, Sched<[WriteJumpLd]>;
}

// Conditional tail calls are similar to the above, but they are branches
// rather than barriers, and they use EFLAGS.
let isCall = 1, isTerminator = 1, isReturn = 1, isBranch = 1,
    isCodeGenOnly = 1, SchedRW = [WriteJump] in
  let Uses = [ESP, EFLAGS, SSP] in {
  def TCRETURNdicc : PseudoI<(outs),
                     (ins i32imm_brtarget:$dst, i32imm:$offset, i32imm:$cond),
                     []>;

  // This gets substituted to a conditional jump instruction in MC lowering.
  def TAILJMPd_CC : PseudoI<(outs), (ins i32imm_brtarget:$dst, i32imm:$cond), []>;
}


//===----------------------------------------------------------------------===//
//  Call Instructions...
//

// RSP is marked as a use to prevent stack-pointer assignments that appear
// immediately before calls from potentially appearing dead. Uses for argument
// registers are added manually.
let isCall = 1, Uses = [RSP, SSP], SchedRW = [WriteJump] in {
  // NOTE: this pattern doesn't match "X86call imm", because we do not know
  // that the offset between an arbitrary immediate and the call will fit in
  // the 32-bit pcrel field that we have.
  def CALL64pcrel32 : Ii32PCRel<0xE8, RawFrm,
                        (outs), (ins i64i32imm_brtarget:$dst),
                        "call{q}\t$dst", []>, OpSize32,
                      Requires<[In64BitMode]>;
  def CALL64r       : I<0xFF, MRM2r, (outs), (ins GR64:$dst),
                        "call{q}\t{*}$dst", [(X86call GR64:$dst)]>,
                      Requires<[In64BitMode,NotUseIndirectThunkCalls,ImportCallOptimizationDisabled]>;
  def CALL64m       : I<0xFF, MRM2m, (outs), (ins i64mem:$dst),
                        "call{q}\t{*}$dst", [(X86call (loadi64 addr:$dst))]>,
                      Requires<[In64BitMode,FavorMemIndirectCall,
                                NotUseIndirectThunkCalls]>;

  // Non-tracking calls for IBT, use with caution.
  let isCodeGenOnly = 1 in {
    def CALL64r_NT : I<0xFF, MRM2r, (outs), (ins GR64 : $dst),
                      "call{q}\t{*}$dst",[(X86NoTrackCall GR64 : $dst)]>,
                      Requires<[In64BitMode]>, NOTRACK;
    def CALL64m_NT : I<0xFF, MRM2m, (outs), (ins i64mem : $dst),
                       "call{q}\t{*}$dst",
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include TAILJMPm, TCRETURNdicc, TAILJMPd_CC, CALL64pcrel32, CALL64r, CALL64m. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 TAILJMPm, TCRETURNdicc, TAILJMPd_CC, CALL64pcrel32, CALL64r, CALL64m。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 351-400: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
                       [(X86NoTrackCall(loadi64 addr : $dst))]>,
                       Requires<[In64BitMode,FavorMemIndirectCall]>, NOTRACK;
  }

  let mayLoad = 1 in
  def FARCALL64m  : RI<0xFF, MRM3m, (outs), (ins opaquemem:$dst),
                       "lcall{q}\t{*}$dst", []>;
}

let isCall = 1, isTerminator = 1, isReturn = 1, isBarrier = 1,
    isCodeGenOnly = 1, Uses = [RSP, SSP] in {
  def TCRETURNdi64   : PseudoI<(outs),
                               (ins i64i32imm_brtarget:$dst, i32imm:$offset),
                               []>, Sched<[WriteJump]>;
  def TCRETURNri64   : PseudoI<(outs),
                               (ins GR64_TC:$dst, i32imm:$offset),
                               []>, Sched<[WriteJump]>;
  def TCRETURN_WIN64ri : PseudoI<(outs), (ins GR64_TCW64:$dst, i32imm:$offset),
                                []>, Sched<[WriteJump]>;

  def TCRETURNri64_ImpCall   : PseudoI<(outs),
                               (ins GR64_A:$dst, i32imm:$offset),
                               []>, Sched<[WriteJump]>;

  let mayLoad = 1 in
  def TCRETURNmi64   : PseudoI<(outs),
                               (ins i64mem_TC:$dst, i32imm:$offset),
                               []>, Sched<[WriteJumpLd]>;
  let mayLoad = 1 in
  def TCRETURN_WINmi64   : PseudoI<(outs),
                               (ins i64mem_w64TC:$dst, i32imm:$offset),
                               []>, Sched<[WriteJumpLd]>;

  def TAILJMPd64 : PseudoI<(outs), (ins i64i32imm_brtarget:$dst),
                           []>, Sched<[WriteJump]>;

  def TAILJMPr64 : PseudoI<(outs), (ins ptr_rc_tailcall:$dst),
                           []>, Sched<[WriteJump]>;

  let mayLoad = 1 in
  def TAILJMPm64 : PseudoI<(outs), (ins i64mem_TC:$dst),
                           []>, Sched<[WriteJumpLd]>;

  // Win64 wants indirect jumps leaving the function to have a REX_W prefix.
  let hasREX_W = 1 in {
    def TAILJMPr64_REX : PseudoI<(outs), (ins ptr_rc_tailcall:$dst),
                                 []>, Sched<[WriteJump]>;

    let mayLoad = 1 in
    def TAILJMPm64_REX : PseudoI<(outs), (ins i64mem_TC:$dst),
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include FARCALL64m, TCRETURNdi64, TCRETURNri64, TCRETURN_WIN64ri, TCRETURNri64_ImpCall, TCRETURNmi64. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 FARCALL64m, TCRETURNdi64, TCRETURNri64, TCRETURN_WIN64ri, TCRETURNri64_ImpCall, TCRETURNmi64。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 401-450: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
                                 []>, Sched<[WriteJumpLd]>;
  }
}

let isPseudo = 1, isCall = 1, isCodeGenOnly = 1,
    Uses = [RSP, SSP],
    usesCustomInserter = 1,
    SchedRW = [WriteJump] in {
  def INDIRECT_THUNK_CALL32 :
    PseudoI<(outs), (ins GR32:$dst), [(X86call GR32:$dst)]>,
            Requires<[Not64BitMode,UseIndirectThunkCalls]>;

  def INDIRECT_THUNK_CALL64 :
    PseudoI<(outs), (ins GR64:$dst), [(X86call GR64:$dst)]>,
            Requires<[In64BitMode,UseIndirectThunkCalls]>;

  // Indirect thunk variant of indirect tail calls.
  let isTerminator = 1, isReturn = 1, isBarrier = 1 in {
    def INDIRECT_THUNK_TCRETURN64 :
      PseudoI<(outs), (ins GR64:$dst, i32imm:$offset), []>;
    def INDIRECT_THUNK_TCRETURN32 :
      PseudoI<(outs), (ins GR32:$dst, i32imm:$offset), []>;
  }
}

let isPseudo = 1, isCall = 1, isCodeGenOnly = 1,
    Uses = [RSP, SSP],
    SchedRW = [WriteJump] in {
  def CALL64m_RVMARKER :
     PseudoI<(outs), (ins i64imm:$rvfunc, i64mem:$dst), [(X86call_rvmarker tglobaladdr:$rvfunc, (loadi64 addr:$dst))]>,
             Requires<[In64BitMode]>;

  def CALL64r_RVMARKER :
    PseudoI<(outs), (ins i64imm:$rvfunc, GR64:$dst), [(X86call_rvmarker tglobaladdr:$rvfunc, GR64:$dst)]>,
            Requires<[In64BitMode]>;

  def CALL64pcrel32_RVMARKER :
    PseudoI<(outs), (ins i64imm:$rvfunc, i64i32imm_brtarget:$dst), []>,
            Requires<[In64BitMode]>;

  def CALL64r_ImpCall :
    PseudoI<(outs), (ins GR64_A:$dst), [(X86call GR64_A:$dst)]>,
            Requires<[In64BitMode,NotUseIndirectThunkCalls,ImportCallOptimizationEnabled]>;
}

// Conditional tail calls are similar to the above, but they are branches
// rather than barriers, and they use EFLAGS.
let isCall = 1, isTerminator = 1, isReturn = 1, isBranch = 1,
    isCodeGenOnly = 1, SchedRW = [WriteJump] in
  let Uses = [RSP, EFLAGS, SSP] in {
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include INDIRECT_THUNK_CALL32, INDIRECT_THUNK_CALL64, INDIRECT_THUNK_TCRETURN64, INDIRECT_THUNK_TCRETURN32, CALL64m_RVMARKER, CALL64r_RVMARKER. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 INDIRECT_THUNK_CALL32, INDIRECT_THUNK_CALL64, INDIRECT_THUNK_TCRETURN64, INDIRECT_THUNK_TCRETURN32, CALL64m_RVMARKER, CALL64r_RVMARKER。这些记录随后会展开为 X86 后端使用的生成表。

### Lines 451-458: TableGen instruction records, predicates, or patterns / TableGen 指令记录、谓词或模式
```tablegen
  def TCRETURNdi64cc : PseudoI<(outs),
                           (ins i64i32imm_brtarget:$dst, i32imm:$offset,
                            i32imm:$cond), []>;

  // This gets substituted to a conditional jump instruction in MC lowering.
  def TAILJMPd64_CC : PseudoI<(outs),
                              (ins i64i32imm_brtarget:$dst, i32imm:$cond), []>;
}
```
**EN:** This section uses LLVM TableGen DSL to describe tablegen instruction records, predicates, or patterns for the core X86 backend. Key symbols include TCRETURNdi64cc, TAILJMPd64_CC. These records are later expanded into generated tables used by the X86 backend.

**CN:** 本段使用 LLVM TableGen DSL 描述X86 后端核心中的TableGen 指令记录、谓词或模式。关键符号包括 TCRETURNdi64cc, TAILJMPd64_CC。这些记录随后会展开为 X86 后端使用的生成表。

## Key Concepts / 关键概念
- Primary topic: instruction definitions and metadata. / 核心主题：指令定义与元数据。
- Subsystem: the core X86 backend. / 所属子系统：X86 后端核心。
- Notable symbols: RET32, RET64, RET16, RETI32, RETI64, RETI16. / 重要符号：RET32, RET64, RET16, RETI32, RETI64, RETI16。
- This file is written in TableGen DSL and contributes generated target tables. / 该文件使用 TableGen DSL 编写，并参与生成目标描述表。

## Dependencies / 依赖关系
- Consumed by LLVM TableGen to emit generated matcher, encoding, register, or scheduling tables. / 由 LLVM TableGen 消费，以生成匹配器、编码、寄存器或调度相关表。
- Closely coupled with sibling X86 .td fragments that assemble the full target description. / 与同级 X86 .td 片段紧密协作，共同组成完整目标描述。
