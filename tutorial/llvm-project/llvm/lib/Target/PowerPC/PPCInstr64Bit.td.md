# PPCInstr64Bit.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCInstr64Bit.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: PPCInstr64Bit.td - The PowerPC 64-bit Support. It is written in LLVM TableGen DSL and feeds generated backend tables.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCInstr64Bit.td`，主要负责 PowerPC 后端的该后端的 TableGen 声明。 它使用 LLVM TableGen DSL 编写，用于描述声明式目标信息并驱动生成代码。
- **Language**: TableGen DSL / TableGen 领域特定语言

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```tablegen
//===-- PPCInstr64Bit.td - The PowerPC 64-bit Support ------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-11

```tablegen
//===----------------------------------------------------------------------===//
//
// This file describes the PowerPC 64-bit instructions.  These patterns are used
// both when in ppc64 mode and when in "use 64-bit extensions in 32-bit" mode.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This file describes the PowerPC 64-bit instructions.  These patterns are used".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This file describes the PowerPC 64-bit instructions.  These patterns are used”。

### Lines 12-13

```tablegen
//===----------------------------------------------------------------------===//
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 14-20

```tablegen
//===----------------------------------------------------------------------===//
// 64-bit operands.
//
def tocentry : Operand<iPTR> {
  let MIOperandInfo = (ops i64imm:$imm);
}
def tlsreg : Operand<i64> {
```
- **EN**: Adds declarative TableGen records such as `tocentry`, `tlsreg` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `tocentry`, `tlsreg`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 21-27

```tablegen
  let EncoderMethod = "getTLSRegEncoding";
  let ParserMatchClass = PPCTLSRegOperand;
}
def tlsgd : Operand<i64> {}
def tlscall : Operand<i64> {
  let PrintMethod = "printTLSCall";
  let MIOperandInfo = (ops calltarget:$func, tlsgd:$sym);
```
- **EN**: Adds declarative TableGen records such as `tlsgd`, `tlscall` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `tlsgd`, `tlscall`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 28-30

```tablegen
  let EncoderMethod = "getTLSCallEncoding";
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 31-39

```tablegen
//===----------------------------------------------------------------------===//
// 64-bit transformation functions.
//

def SHL64 : SDNodeXForm<imm, [{
  // Transformation function: 63 - imm
  return getI32Imm(63 - N->getZExtValue(), SDLoc(N));
}]>;
```
- **EN**: Adds declarative TableGen records such as `SHL64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SHL64`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 40-46

```tablegen
def SRL64 : SDNodeXForm<imm, [{
  // Transformation function: 64 - imm
  return N->getZExtValue() ? getI32Imm(64 - N->getZExtValue(), SDLoc(N))
                           : getI32Imm(0, SDLoc(N));
}]>;
```
- **EN**: Adds declarative TableGen records such as `SRL64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SRL64`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 47-53

```tablegen
//===----------------------------------------------------------------------===//
// Calls.
//

let Interpretation64Bit = 1, isCodeGenOnly = 1 in {
let isTerminator = 1, isBarrier = 1, PPC970_Unit = 7, hasSideEffects = 0 in {
  let isReturn = 1, isPredicable = 1, Uses = [LR8, RM] in
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 54-60

```tablegen
    def BLR8 : XLForm_2_ext<19, 16, 20, 0, 0, (outs), (ins), "blr", IIC_BrB,
                            [(PPCretglue)]>, Requires<[IsPPC64]>;
  let isBranch = 1, isIndirectBranch = 1, Uses = [CTR8] in {
    let isPredicable = 1 in
      def BCTR8 : XLForm_2_ext<19, 528, 20, 0, 0, (outs), (ins), "bctr", IIC_BrB,
                               []>,
          Requires<[IsPPC64]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 61-68

```tablegen
    def BCCCTR8 : XLForm_2_br<19, 528, 0, (outs), (ins (pred $BIBO, $CR):$cond),
                              "b${cond:cc}ctr${cond:pm} ${cond:reg}", IIC_BrB,
                              []>,
        Requires<[IsPPC64]>;

    def BCCTR8  : XLForm_2_br2<19, 528, 12, 0, (outs), (ins crbitrc:$BI),
                               "bcctr 12, $BI, 0", IIC_BrB, []>,
        Requires<[IsPPC64]>;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 69-75

```tablegen
    def BCCTR8n : XLForm_2_br2<19, 528, 4, 0, (outs), (ins crbitrc:$BI),
                               "bcctr 4, $BI, 0", IIC_BrB, []>,
        Requires<[IsPPC64]>;
  }
}

let Defs = [LR8] in
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 76-82

```tablegen
  def MovePCtoLR8 : PPCEmitTimePseudo<(outs), (ins), "#MovePCtoLR8", []>,
                    PPC970_Unit_BRU;

let isBranch = 1, isTerminator = 1, hasCtrlDep = 1, PPC970_Unit = 7, hasSideEffects = 0 in {
  let Defs = [CTR8], Uses = [CTR8] in {
    def BDZ8  : BForm_1<16, 18, 0, 0, (outs), (ins condbrtarget:$BD),
                        "bdz $BD">;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 83-89

```tablegen
    def BDNZ8 : BForm_1<16, 16, 0, 0, (outs), (ins condbrtarget:$BD),
                        "bdnz $BD">;
  }

  let isReturn = 1, Defs = [CTR8], Uses = [CTR8, LR8, RM] in {
    def BDZLR8  : XLForm_2_ext<19, 16, 18, 0, 0, (outs), (ins),
                              "bdzlr", IIC_BrB, []>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 90-96

```tablegen
    def BDNZLR8 : XLForm_2_ext<19, 16, 16, 0, 0, (outs), (ins),
                              "bdnzlr", IIC_BrB, []>;
  }
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 97-105

```tablegen
let isCall = 1, PPC970_Unit = 7, Defs = [LR8], hasSideEffects = 0 in {
  // Convenient aliases for call instructions
  let Uses = [RM] in {
    def BL8  : IForm<18, 0, 1, (outs), (ins calltarget:$LI),
                     "bl $LI", IIC_BrB, []>;  // See Pat patterns below.

    def BL8_TLS  : IForm<18, 0, 1, (outs), (ins tlscall:$LI),
                         "bl $LI", IIC_BrB, []>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 106-113

```tablegen
    def BLA8 : IForm<18, 1, 1, (outs), (ins abscalltarget:$LI),
                     "bla $LI", IIC_BrB, [(PPCcall (i64 imm:$LI))]>;
  }
  let Uses = [RM], isCodeGenOnly = 1 in {
    def BL8_NOP  : IForm_and_DForm_4_zero<18, 0, 1, 24,
                             (outs), (ins calltarget:$LI),
                             "bl $LI\n\tnop", IIC_BrB, []>;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 114-121

```tablegen
    def BL8_NOP_TLS : IForm_and_DForm_4_zero<18, 0, 1, 24,
                                  (outs), (ins tlscall:$LI),
                                  "bl $LI\n\tnop", IIC_BrB, []>;

    def BLA8_NOP : IForm_and_DForm_4_zero<18, 1, 1, 24,
                             (outs), (ins abscalltarget:$LI),
                             "bla $LI\n\tnop", IIC_BrB,
                             [(PPCcall_nop (i64 imm:$LI))]>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 122-129

```tablegen
    let Predicates = [PCRelativeMemops] in {
      // BL8_NOTOC means that the caller does not use the TOC pointer and if
      // it does use R2 then it is just a caller saved register. Therefore it is
      // safe to emit only the bl and not the nop for this instruction. The
      // linker will not try to restore R2 after the call.
      def BL8_NOTOC : IForm<18, 0, 1, (outs),
                            (ins calltarget:$LI),
                            "bl $LI", IIC_BrB, []>;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "BL8_NOTOC means that the caller does not use the TOC pointer and if". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“BL8_NOTOC means that the caller does not use the TOC pointer and if”。 这些声明会进入生成式模式匹配逻辑。

### Lines 130-136

```tablegen
      def BL8_NOTOC_TLS : IForm<18, 0, 1, (outs),
                                (ins tlscall:$LI),
                                "bl $LI", IIC_BrB, []>;
    }
  }
  let Uses = [CTR8, RM] in {
    let isPredicable = 1 in
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 137-146

```tablegen
      def BCTRL8 : XLForm_2_ext<19, 528, 20, 0, 1, (outs), (ins),
                                "bctrl", IIC_BrB, [(PPCbctrl)]>,
                   Requires<[IsPPC64]>;

    let isCodeGenOnly = 1 in {
      def BCCCTRL8 : XLForm_2_br<19, 528, 1, (outs), (ins (pred $BIBO, $CR):$cond),
                                 "b${cond:cc}ctrl${cond:pm} ${cond:reg}", IIC_BrB,
                                 []>,
          Requires<[IsPPC64]>;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 147-156

```tablegen
      def BCCTRL8  : XLForm_2_br2<19, 528, 12, 1, (outs), (ins crbitrc:$BI),
                                  "bcctrl 12, $BI, 0", IIC_BrB, []>,
          Requires<[IsPPC64]>;
      def BCCTRL8n : XLForm_2_br2<19, 528, 4, 1, (outs), (ins crbitrc:$BI),
                                  "bcctrl 4, $BI, 0", IIC_BrB, []>,
          Requires<[IsPPC64]>;
    }
  }
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 157-164

```tablegen
let isCall = 1, PPC970_Unit = 7, Defs = [LR8, RM], hasSideEffects = 0,
    isCodeGenOnly = 1, Uses = [RM] in {
  // Convenient aliases for call instructions
  def BL8_RM  : IForm<18, 0, 1, (outs), (ins calltarget:$LI),
                      "bl $LI", IIC_BrB, []>;  // See Pat patterns below.

  def BLA8_RM : IForm<18, 1, 1, (outs), (ins abscalltarget:$LI),
                      "bla $LI", IIC_BrB, [(PPCcall_rm (i64 imm:$LI))]>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 165-172

```tablegen
  def BL8_NOP_RM  : IForm_and_DForm_4_zero<18, 0, 1, 24,
                           (outs), (ins calltarget:$LI),
                           "bl $LI\n\tnop", IIC_BrB, []>;

  def BLA8_NOP_RM : IForm_and_DForm_4_zero<18, 1, 1, 24,
                           (outs), (ins abscalltarget:$LI),
                           "bla $LI\n\tnop", IIC_BrB,
                           [(PPCcall_nop_rm (i64 imm:$LI))]>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 173-181

```tablegen
  let Predicates = [PCRelativeMemops] in {
    // BL8_NOTOC means that the caller does not use the TOC pointer and if
    // it does use R2 then it is just a caller saved register. Therefore it is
    // safe to emit only the bl and not the nop for this instruction. The
    // linker will not try to restore R2 after the call.
    def BL8_NOTOC_RM : IForm<18, 0, 1, (outs),
                             (ins calltarget:$LI),
                             "bl $LI", IIC_BrB, []>;
  }
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "BL8_NOTOC means that the caller does not use the TOC pointer and if". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“BL8_NOTOC means that the caller does not use the TOC pointer and if”。 这些声明会进入生成式模式匹配逻辑。

### Lines 182-189

```tablegen
  let Uses = [CTR8, RM] in {
    let isPredicable = 1 in
      def BCTRL8_RM : XLForm_2_ext<19, 528, 20, 0, 1, (outs), (ins),
                                   "bctrl", IIC_BrB, [(PPCbctrl_rm)]>,
                   Requires<[IsPPC64]>;
  }
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 190-199

```tablegen
let isCall = 1, PPC970_Unit = 7, isCodeGenOnly = 1,
    Defs = [LR8, X2], Uses = [CTR8, RM], RST = 2 in {
  def BCTRL8_LDinto_toc :
    XLForm_2_ext_and_DSForm_1<19, 528, 20, 0, 1, 58, 0, (outs),
                              (ins (memrix $D, $RA):$src),
                              "bctrl\n\tld 2, $src", IIC_BrB,
                              [(PPCbctrl_load_toc iaddrX4:$src)]>,
    Requires<[IsPPC64]>;
}
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 200-209

```tablegen
let isCall = 1, PPC970_Unit = 7, isCodeGenOnly = 1,
    Defs = [LR8, X2, RM], Uses = [CTR8, RM], RST = 2 in {
  def BCTRL8_LDinto_toc_RM :
    XLForm_2_ext_and_DSForm_1<19, 528, 20, 0, 1, 58, 0, (outs),
                              (ins (memrix $D, $RA):$src),
                              "bctrl\n\tld 2, $src", IIC_BrB,
                              [(PPCbctrl_load_toc_rm iaddrX4:$src)]>,
    Requires<[IsPPC64]>;
}
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 210-219

```tablegen
let Predicates = [IsAIX] in {
  let isCall = 1, PPC970_Unit = 7, isCodeGenOnly = 1,
      Defs = [LR8, X2], Uses = [RM, X1], RST = 2, RA = 1, D = 40 in {
    def BL8_LDinto_toc : IForm_and_DForm_1<18, 0, 1, 58,
                        (outs), (ins calltarget:$LI),
                        "bl $LI\n\tld 2, 40(1)", IIC_BrB,
                        []>,
                         Requires<[IsPPC64]>;
  }
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 220-234

```tablegen
  let isCall = 1, PPC970_Unit = 7, isCodeGenOnly = 1,
      Defs = [LR8, X2, RM], Uses = [RM, X1], RST = 2, RA = 1, D = 40 in {
    def BL8_LDinto_toc_RM : IForm_and_DForm_1<18, 0, 1, 58,
                          (outs), (ins calltarget:$LI),
                          "bl $LI\n\tld 2, 40(1)", IIC_BrB,
                          []>,
                           Requires<[IsPPC64]>;
  }
}

} // Interpretation64Bit

// FIXME: Duplicating this for the asm parser should be unnecessary, but the
// previous definition must be marked as CodeGen only to prevent decoding
// conflicts.
```
- **EN**: Contains iterative logic that walks instructions, operands, or tables to apply a backend transformation step by step.
- **CN**: 这里包含迭代逻辑，会逐步遍历指令、操作数或表项来执行后端变换。

### Lines 235-242

```tablegen
let Interpretation64Bit = 1, isAsmParserOnly = 1, hasSideEffects = 0 in
let isCall = 1, PPC970_Unit = 7, Defs = [LR8], Uses = [RM] in
def BL8_TLS_ : IForm<18, 0, 1, (outs), (ins tlscall:$LI),
                     "bl $LI", IIC_BrB, []>;

// Calls
def : Pat<(PPCcall (i64 tglobaladdr:$dst)),
          (BL8 tglobaladdr:$dst)>;
```
- **EN**: Adds declarative TableGen records such as `BL8_TLS_` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `BL8_TLS_`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 243-250

```tablegen
def : Pat<(PPCcall_nop (i64 tglobaladdr:$dst)),
          (BL8_NOP tglobaladdr:$dst)>;

def : Pat<(PPCcall (i64 texternalsym:$dst)),
          (BL8 texternalsym:$dst)>;
def : Pat<(PPCcall_nop (i64 texternalsym:$dst)),
          (BL8_NOP texternalsym:$dst)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 251-257

```tablegen
def : Pat<(PPCcall_notoc (i64 tglobaladdr:$dst)),
          (BL8_NOTOC tglobaladdr:$dst)>;
def : Pat<(PPCcall_notoc (i64 texternalsym:$dst)),
          (BL8_NOTOC texternalsym:$dst)>;

def : Pat<(PPCcall_rm (i64 tglobaladdr:$dst)),
          (BL8_RM tglobaladdr:$dst)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 258-265

```tablegen
def : Pat<(PPCcall_nop_rm (i64 tglobaladdr:$dst)),
          (BL8_NOP_RM tglobaladdr:$dst)>;

def : Pat<(PPCcall_rm (i64 texternalsym:$dst)),
          (BL8_RM texternalsym:$dst)>;
def : Pat<(PPCcall_nop_rm (i64 texternalsym:$dst)),
          (BL8_NOP_RM texternalsym:$dst)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 266-273

```tablegen
def : Pat<(PPCcall_notoc_rm (i64 tglobaladdr:$dst)),
          (BL8_NOTOC_RM tglobaladdr:$dst)>;
def : Pat<(PPCcall_notoc_rm (i64 texternalsym:$dst)),
          (BL8_NOTOC_RM texternalsym:$dst)>;

// Calls for AIX
def : Pat<(PPCcall (i64 mcsym:$dst)),
          (BL8 mcsym:$dst)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 274-281

```tablegen
def : Pat<(PPCcall_nop (i64 mcsym:$dst)),
          (BL8_NOP mcsym:$dst)>;

def : Pat<(PPCcall_rm (i64 mcsym:$dst)),
          (BL8_RM mcsym:$dst)>;
def : Pat<(PPCcall_nop_rm (i64 mcsym:$dst)),
          (BL8_NOP_RM mcsym:$dst)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 282-295

```tablegen
let Predicates = [IsAIX] in {
  def : Pat<(PPCbl_load_toc (i64 texternalsym:$dst)),
            (BL8_LDinto_toc texternalsym:$dst)>;

  def : Pat<(PPCbl_load_toc_rm (i64 texternalsym:$dst)),
            (BL8_LDinto_toc_RM texternalsym:$dst)>;
}

// Atomic operations
// FIXME: some of these might be used with constant operands. This will result
// in constant materialization instructions that may be redundant. We currently
// clean this up in PPCMIPeephole with calls to
// PPCInstrInfo::convertToImmediateForm() but we should probably not emit them
// in the first place.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Atomic operations". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Atomic operations”。 这些声明会进入生成式模式匹配逻辑。

### Lines 296-304

```tablegen
foreach op = ["load_add", "load_sub", "load_and", "load_or", "load_xor",
              "load_nand", "load_min", "load_max", "load_umax", "load_umin",
              "swap"] in {
  defvar pat = !cast<PatFrag>("atomic_"#op#"_i64");
  defvar pseudo = "ATOMIC_"#!toupper(op)#"_I64";
  let Defs = [CR0] in
    def pseudo : PPCCustomInserterPseudo<
        (outs g8rc:$dst), (ins memrr:$ptr, i32imm:$sz, g8rc:$incr),
        "#" # NAME, []>;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend. These declarations feed generated pattern-matching logic.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。 这些声明会进入生成式模式匹配逻辑。

### Lines 305-313

```tablegen
  def : Pat<(i64 (pat ForceXForm:$ptr, g8rc:$incr)),
          (!cast<Instruction>(pseudo) memrr:$ptr, 8, g8rc:$incr)>;
}

let Defs = [CR0] in
  def ATOMIC_CMP_SWAP_I64 : PPCCustomInserterPseudo<
    (outs g8rc:$dst), (ins memrr:$ptr, g8rc:$old, g8rc:$new),
    "#" # NAME, []>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 314-321

```tablegen
def : Pat<(i64 (atomic_cmp_swap_i64 ForceXForm:$ptr, g8rc:$old, g8rc:$new)),
          (ATOMIC_CMP_SWAP_I64 memrr:$ptr, i64:$old, i64:$new)>;

// Instructions to support atomic operations
let mayLoad = 1, mayStore = 1, hasSideEffects = 1 in {
def LDARX : XForm_1_memOp<31,  84, (outs g8rc:$RST), (ins (memrr $RA, $RB):$addr),
                          "ldarx $RST, $addr", IIC_LdStLDARX, []>;
// TODO: Add scheduling info.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Instructions to support atomic operations". Notable symbols in this range include `LDARX`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Instructions to support atomic operations”。 该区间中较显眼的符号包括 `LDARX`。

### Lines 322-330

```tablegen
let hasNoSchedulingInfo = 1 in
def LQARX : XForm_1_memOp<31, 276, (outs g8prc:$RST), (ins (memrr $RA, $RB):$addr),
                          "lqarx $RST, $addr", IIC_LdStLQARX, []>, isPPC64;

// Instruction to support lock versions of atomics
// (EH=1 - see Power ISA 2.07 Book II 4.4.2)
def LDARXL : XForm_1<31,  84, (outs g8rc:$RST), (ins (memrr $RA, $RB):$addr),
                     "ldarx $RST, $addr, 1", IIC_LdStLDARX, []>, isRecordForm;
// TODO: Add scheduling info.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Instruction to support lock versions of atomics". Scheduling or processor-model metadata is part of this section.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Instruction to support lock versions of atomics”。 这一段包含调度或处理器模型元数据。

### Lines 331-337

```tablegen
let hasNoSchedulingInfo = 1 in
// FIXME: We have to seek a way to remove isRecordForm since
// LQARXL is not really altering CR0.
def LQARXL : XForm_1<31, 276, (outs g8prc:$RST), (ins (memrr $RA, $RB):$addr),
                     "lqarx $RST, $addr, 1", IIC_LdStLQARX, []>,
                     isPPC64, isRecordForm;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "FIXME: We have to seek a way to remove isRecordForm since". Scheduling or processor-model metadata is part of this section.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“FIXME: We have to seek a way to remove isRecordForm since”。 这一段包含调度或处理器模型元数据。

### Lines 338-345

```tablegen
let hasExtraDefRegAllocReq = 1, mayStore = 1 in
def LDAT : X_RD5_RS5_IM5<31, 614, (outs g8prc:$RST), (ins g8prc:$RSTi, ptr_rc_nor0:$RA, u5imm:$RB),
                         "ldat $RST, $RA, $RB", IIC_LdStLoad>, isPPC64,
           Requires<[IsISA3_0]>,
           RegConstraint<"$RSTi = $RST">;

let hasExtraSrcRegAllocReq = 1, mayStore = 1,
      isCodeGenOnly = 1, hasNoSchedulingInfo = 1, RB = 16 in
```
- **EN**: Adds declarative TableGen records such as `LDAT` that LLVM later expands into generated tables or helper code. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LDAT`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段包含调度或处理器模型元数据。

### Lines 346-353

```tablegen
def LDAT_CSNE : X_RD5_RS5_IM5<31, 614, (outs g8rc:$RST), (ins ptr_rc_nor0:$RA),
                         "ldat $RST, $RA, 16", IIC_LdStLoad>,
           Requires<[IsISA3_0]>;
}

def : Pat<(int_ppc_amo_ldat ptr_rc_nor0:$ptr, g8rc:$val, u5imm_timm:$fc),
          (EVEN8 (LDAT (PAIR8 (i64 (IMPLICIT_DEF)), $val), $ptr, $fc))>;
```
- **EN**: Adds declarative TableGen records such as `LDAT_CSNE` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LDAT_CSNE`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 354-363

```tablegen
def : Pat<(int_ppc_amo_ldat_cond ptr_rc_nor0:$ptr, u5imm_timm:$fc),
          (EVEN8 (LDAT (PAIR8 (i64 (IMPLICIT_DEF)), (i64 (IMPLICIT_DEF))), $ptr, $fc))>;

let Defs = [X8, X9, X10], Uses = [X9, X10] in
def LDAT_CSNE_PSEUDO : PPCPostRAExpPseudo<
    (outs g8rc:$dst),
    (ins ptr_rc_nor0:$ptr),
    "#LDAT_CSNE_PSEUDO",
    [(set i64:$dst, (int_ppc_amo_ldat_csne ptr_rc_nor0:$ptr, X9, X10))]>;
```
- **EN**: Adds declarative TableGen records such as `LDAT_CSNE_PSEUDO` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LDAT_CSNE_PSEUDO`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 364-373

```tablegen
let Defs = [CR0], mayStore = 1, mayLoad = 1, hasSideEffects = 1 in {
def STDCX : XForm_1_memOp<31, 214, (outs), (ins g8rc:$RST, (memrr $RA, $RB):$addr),
                          "stdcx. $RST, $addr", IIC_LdStSTDCX, []>, isRecordForm;
// TODO: Add scheduling info.
let hasNoSchedulingInfo = 1 in
def STQCX : XForm_1_memOp<31, 182, (outs), (ins g8prc:$RST, (memrr $RA, $RB):$addr),
                          "stqcx. $RST, $addr", IIC_LdStSTQCX, []>,
                          isPPC64, isRecordForm;
}
```
- **EN**: Adds declarative TableGen records such as `STDCX`, `STQCX` that LLVM later expands into generated tables or helper code. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STDCX`, `STQCX`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段包含调度或处理器模型元数据。

### Lines 374-382

```tablegen
def SPLIT_QUADWORD : PPCCustomInserterPseudo<(outs g8rc:$lo, g8rc:$hi),
                                             (ins g8prc:$src),
                                             "#SPLIT_QUADWORD", []>;
class AtomicRMW128<string asmstr>
  : PPCPostRAExpPseudo<(outs g8prc:$RTp, g8prc:$scratch),
                       (ins memrr:$ptr, g8rc:$incr_lo, g8rc:$incr_hi),
                       asmstr, []>;
// We have to keep values in MI's uses during LL/SC looping as they are,
// so set both $RTp and $scratch earlyclobber.
```
- **EN**: Declares a backend-facing type `SPLIT_QUADWORD`, `AtomicRMW128` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `SPLIT_QUADWORD`, `AtomicRMW128`，并勾勒出周边代码会依赖的接口或状态。

### Lines 383-389

```tablegen
let mayStore = 1, mayLoad = 1,
    Defs = [CR0],
    Constraints = "@earlyclobber $scratch,@earlyclobber $RTp" in {
// Atomic pseudo instructions expanded post-ra.
def ATOMIC_SWAP_I128      : AtomicRMW128<"#ATOMIC_SWAP_I128">;
def ATOMIC_LOAD_ADD_I128  : AtomicRMW128<"#ATOMIC_LOAD_ADD_I128">;
def ATOMIC_LOAD_SUB_I128  : AtomicRMW128<"#ATOMIC_LOAD_SUB_I128">;
```
- **EN**: Adds declarative TableGen records such as `ATOMIC_SWAP_I128`, `ATOMIC_LOAD_ADD_I128`, `ATOMIC_LOAD_SUB_I128` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ATOMIC_SWAP_I128`, `ATOMIC_LOAD_ADD_I128`, `ATOMIC_LOAD_SUB_I128`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 390-401

```tablegen
def ATOMIC_LOAD_AND_I128  : AtomicRMW128<"#ATOMIC_LOAD_AND_I128">;
def ATOMIC_LOAD_XOR_I128  : AtomicRMW128<"#ATOMIC_LOAD_XOR_I128">;
def ATOMIC_LOAD_OR_I128   : AtomicRMW128<"#ATOMIC_LOAD_OR_I128">;
def ATOMIC_LOAD_NAND_I128 : AtomicRMW128<"#ATOMIC_LOAD_NAND_I128">;

def ATOMIC_CMP_SWAP_I128 : PPCPostRAExpPseudo<
                              (outs g8prc:$RTp, g8prc:$scratch),
                              (ins memrr:$ptr, g8rc:$cmp_lo, g8rc:$cmp_hi,
                                   g8rc:$new_lo, g8rc:$new_hi),
                              "#ATOMIC_CMP_SWAP_I128", []>;
}
```
- **EN**: Adds declarative TableGen records such as `ATOMIC_LOAD_AND_I128`, `ATOMIC_LOAD_XOR_I128`, `ATOMIC_LOAD_OR_I128` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ATOMIC_LOAD_AND_I128`, `ATOMIC_LOAD_XOR_I128`, `ATOMIC_LOAD_OR_I128`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 402-409

```tablegen
class PatAtomicRMWI128<SDPatternOperator OpNode, AtomicRMW128 Inst> :
      Pat<(OpNode ForceXForm:$ptr,
                  i64:$incr_lo,
                  i64:$incr_hi),
          (SPLIT_QUADWORD (Inst memrr:$ptr,
                                g8rc:$incr_lo,
                                g8rc:$incr_hi))>;
```
- **EN**: Declares a backend-facing type `PatAtomicRMWI128` and outlines the API or state that nearby code will rely on. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明面向后端的类型 `PatAtomicRMWI128`，并勾勒出周边代码会依赖的接口或状态。 这些声明会进入生成式模式匹配逻辑。

### Lines 410-416

```tablegen
def : PatAtomicRMWI128<int_ppc_atomicrmw_add_i128,  ATOMIC_LOAD_ADD_I128>;
def : PatAtomicRMWI128<int_ppc_atomicrmw_sub_i128,  ATOMIC_LOAD_SUB_I128>;
def : PatAtomicRMWI128<int_ppc_atomicrmw_xor_i128,  ATOMIC_LOAD_XOR_I128>;
def : PatAtomicRMWI128<int_ppc_atomicrmw_and_i128,  ATOMIC_LOAD_AND_I128>;
def : PatAtomicRMWI128<int_ppc_atomicrmw_nand_i128, ATOMIC_LOAD_NAND_I128>;
def : PatAtomicRMWI128<int_ppc_atomicrmw_or_i128,   ATOMIC_LOAD_OR_I128>;
def : PatAtomicRMWI128<int_ppc_atomicrmw_xchg_i128, ATOMIC_SWAP_I128>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 417-428

```tablegen
def : Pat<(int_ppc_cmpxchg_i128 ForceXForm:$ptr,
                                i64:$cmp_lo,
                                i64:$cmp_hi,
                                i64:$new_lo,
                                i64:$new_hi),
          (SPLIT_QUADWORD (ATOMIC_CMP_SWAP_I128
                           memrr:$ptr,
                           g8rc:$cmp_lo,
                           g8rc:$cmp_hi,
                           g8rc:$new_lo,
                           g8rc:$new_hi))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 429-436

```tablegen
let mayStore = 1, mayLoad = 1, hasSideEffects = 0 in
def STDAT : XForm_base_r3xo_memOp<31, 742, (outs),
                                  (ins g8rc:$RST, ptr_rc_nor0:$RA, u5imm:$RB),
                                  "stdat $RST, $RA, $RB", IIC_LdStStore,
                                  [(int_ppc_amo_stdat ptr_rc_nor0:$RA, i64:$RST,
                                            u5imm_timm:$RB)]>, isPPC64,
            Requires<[IsISA3_0]>;
```
- **EN**: Adds declarative TableGen records such as `STDAT` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STDAT`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 437-443

```tablegen
let Interpretation64Bit = 1, isCodeGenOnly = 1 in {
let isCall = 1, isTerminator = 1, isReturn = 1, isBarrier = 1, Uses = [RM] in
def TCRETURNdi8 :PPCEmitTimePseudo< (outs),
                        (ins calltarget:$dst, i32imm:$offset),
                 "#TC_RETURNd8 $dst $offset",
                 []>;
```
- **EN**: Adds declarative TableGen records such as `TCRETURNdi8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `TCRETURNdi8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 444-453

```tablegen
let isCall = 1, isTerminator = 1, isReturn = 1, isBarrier = 1, Uses = [RM] in
def TCRETURNai8 :PPCEmitTimePseudo<(outs), (ins abscalltarget:$func, i32imm:$offset),
                 "#TC_RETURNa8 $func $offset",
                 [(PPCtc_return (i64 imm:$func), imm:$offset)]>;

let isCall = 1, isTerminator = 1, isReturn = 1, isBarrier = 1, Uses = [RM] in
def TCRETURNri8 : PPCEmitTimePseudo<(outs), (ins CTRRC8:$dst, i32imm:$offset),
                 "#TC_RETURNr8 $dst $offset",
                 []>;
```
- **EN**: Adds declarative TableGen records such as `TCRETURNai8`, `TCRETURNri8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `TCRETURNai8`, `TCRETURNri8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 454-460

```tablegen
let hasSideEffects = 0 in {
let isTerminator = 1, isBarrier = 1, PPC970_Unit = 7, isBranch = 1,
    isIndirectBranch = 1, isCall = 1, isReturn = 1, Uses = [CTR8, RM] in
def TAILBCTR8 : XLForm_2_ext<19, 528, 20, 0, 0, (outs), (ins), "bctr", IIC_BrB,
                             []>,
    Requires<[IsPPC64]>;
```
- **EN**: Adds declarative TableGen records such as `TAILBCTR8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `TAILBCTR8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 461-468

```tablegen
let isBranch = 1, isTerminator = 1, hasCtrlDep = 1, PPC970_Unit = 7,
    isBarrier = 1, isCall = 1, isReturn = 1, Uses = [RM] in
def TAILB8   : IForm<18, 0, 0, (outs), (ins calltarget:$LI),
                  "b $LI", IIC_BrB,
                  []>;

let isBranch = 1, isTerminator = 1, hasCtrlDep = 1, PPC970_Unit = 7,
    isBarrier = 1, isCall = 1, isReturn = 1, Uses = [RM] in
```
- **EN**: Adds declarative TableGen records such as `TAILB8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `TAILB8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 469-477

```tablegen
def TAILBA8   : IForm<18, 0, 0, (outs), (ins abscalltarget:$LI),
                  "ba $LI", IIC_BrB,
                  []>;
}
} // Interpretation64Bit

def : Pat<(PPCtc_return (i64 tglobaladdr:$dst),  imm:$imm),
          (TCRETURNdi8 tglobaladdr:$dst, imm:$imm)>;
```
- **EN**: Adds declarative TableGen records such as `TAILBA8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `TAILBA8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 478-485

```tablegen
def : Pat<(PPCtc_return (i64 texternalsym:$dst), imm:$imm),
          (TCRETURNdi8 texternalsym:$dst, imm:$imm)>;

def : Pat<(PPCtc_return CTRRC8:$dst, imm:$imm),
          (TCRETURNri8 CTRRC8:$dst, imm:$imm)>;


// 64-bit CR instructions
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 486-497

```tablegen
let Interpretation64Bit = 1, isCodeGenOnly = 1 in {
let hasSideEffects = 0 in {
// mtocrf's input needs to be prepared by shifting by an amount dependent
// on the cr register selected. Thus, post-ra anti-dep breaking must not
// later change that register assignment.
let hasExtraDefRegAllocReq = 1 in {
def MTOCRF8: XFXForm_5a<31, 144, (outs crbitm:$FXM), (ins g8rc:$RST),
                        "mtocrf $FXM, $RST", IIC_BrMCRX>,
            PPC970_DGroup_First, PPC970_Unit_CRU;

// Similarly to mtocrf, the mask for mtcrf must be prepared in a way that
// is dependent on the cr fields being set.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "mtocrf's input needs to be prepared by shifting by an amount dependent". Notable symbols in this range include `MTOCRF8`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“mtocrf's input needs to be prepared by shifting by an amount dependent”。 该区间中较显眼的符号包括 `MTOCRF8`。

### Lines 498-505

```tablegen
def MTCRF8 : XFXForm_5<31, 144, (outs), (ins i32imm:$FXM, g8rc:$RST),
                      "mtcrf $FXM, $RST", IIC_BrMCRX>,
            PPC970_MicroCode, PPC970_Unit_CRU;
} // hasExtraDefRegAllocReq = 1

// mfocrf's input needs to be prepared by shifting by an amount dependent
// on the cr register selected. Thus, post-ra anti-dep breaking must not
// later change that register assignment.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "mfocrf's input needs to be prepared by shifting by an amount dependent". Notable symbols in this range include `MTCRF8`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“mfocrf's input needs to be prepared by shifting by an amount dependent”。 该区间中较显眼的符号包括 `MTCRF8`。

### Lines 506-512

```tablegen
let hasExtraSrcRegAllocReq = 1 in {
def MFOCRF8: XFXForm_5a<31, 19, (outs g8rc:$RST), (ins crbitm:$FXM),
                        "mfocrf $RST, $FXM", IIC_SprMFCRF>,
             PPC970_DGroup_First, PPC970_Unit_CRU;

// Similarly to mfocrf, the mask for mfcrf must be prepared in a way that
// is dependent on the cr fields being copied.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Similarly to mfocrf, the mask for mfcrf must be prepared in a way that". Notable symbols in this range include `MFOCRF8`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Similarly to mfocrf, the mask for mfcrf must be prepared in a way that”。 该区间中较显眼的符号包括 `MFOCRF8`。

### Lines 513-520

```tablegen
def MFCR8 : XFXForm_3<31, 19, (outs g8rc:$RT), (ins),
                     "mfcr $RT", IIC_SprMFCR>,
                     PPC970_MicroCode, PPC970_Unit_CRU;
} // hasExtraSrcRegAllocReq = 1
} // hasSideEffects = 0

// While longjmp is a control-flow barrier (fallthrough isn't allowed), setjmp
// is not.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "While longjmp is a control-flow barrier (fallthrough isn't allowed), setjmp". Notable symbols in this range include `MFCR8`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“While longjmp is a control-flow barrier (fallthrough isn't allowed), setjmp”。 该区间中较显眼的符号包括 `MFCR8`。

### Lines 521-528

```tablegen
let hasSideEffects = 1 in {
  let Defs = [CTR8] in
  def EH_SjLj_SetJmp64  : PPCCustomInserterPseudo<(outs gprc:$dst), (ins memr:$buf),
                            "#EH_SJLJ_SETJMP64",
                            [(set i32:$dst, (PPCeh_sjlj_setjmp addr:$buf))]>,
                          Requires<[IsPPC64]>;
}
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 529-536

```tablegen
let hasSideEffects = 1, isBarrier = 1 in {
  let isTerminator = 1 in
  def EH_SjLj_LongJmp64 : PPCCustomInserterPseudo<(outs), (ins memr:$buf),
                            "#EH_SJLJ_LONGJMP64",
                            [(PPCeh_sjlj_longjmp addr:$buf)]>,
                          Requires<[IsPPC64]>;
}
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 537-542

```tablegen
def MFSPR8 : XFXForm_1<31, 339, (outs g8rc:$RST), (ins i32imm:$SPR),
                       "mfspr $RST, $SPR", IIC_SprMFSPR>;
def MTSPR8 : XFXForm_1<31, 467, (outs), (ins i32imm:$SPR, g8rc:$RST),
                       "mtspr $SPR, $RST", IIC_SprMTSPR>;
```
- **EN**: Adds declarative TableGen records such as `MFSPR8`, `MTSPR8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MFSPR8`, `MTSPR8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 543-550

```tablegen
//===----------------------------------------------------------------------===//
// 64-bit SPR manipulation instrs.

let Uses = [CTR8] in {
def MFCTR8 : XFXForm_1_ext<31, 339, 9, (outs g8rc:$RST), (ins),
                           "mfctr $RST", IIC_SprMFSPR>,
             PPC970_DGroup_First, PPC970_Unit_FXU;
}
```
- **EN**: Adds declarative TableGen records such as `MFCTR8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MFCTR8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 551-557

```tablegen
let Pattern = [(PPCmtctr i64:$RST)], Defs = [CTR8] in {
def MTCTR8 : XFXForm_1_ext<31, 467, 9, (outs), (ins g8rc:$RST),
                           "mtctr $RST", IIC_SprMTSPR>,
             PPC970_DGroup_First, PPC970_Unit_FXU;
}
// MTCTR[8|]loop must be inside a loop-preheader, duplicating
// the loop-preheader block will break this assumption.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "MTCTR[8|]loop must be inside a loop-preheader, duplicating". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“MTCTR[8|]loop must be inside a loop-preheader, duplicating”。 这些声明会进入生成式模式匹配逻辑。

### Lines 558-564

```tablegen
let hasSideEffects = 1, isNotDuplicable = 1, Defs = [CTR8] in {
let Pattern = [(int_set_loop_iterations i64:$RST)] in
def MTCTR8loop : XFXForm_1_ext<31, 467, 9, (outs), (ins g8rc:$RST),
                               "mtctr $RST", IIC_SprMTSPR>,
                 PPC970_DGroup_First, PPC970_Unit_FXU;
}
```
- **EN**: Adds declarative TableGen records such as `MTCTR8loop` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MTCTR8loop`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 565-577

```tablegen
let hasSideEffects = 1, hasNoSchedulingInfo = 1, isNotDuplicable = 1, Uses = [CTR8], Defs = [CTR8] in
def DecreaseCTR8loop : PPCEmitTimePseudo<(outs crbitrc:$rT), (ins i64imm:$stride),
                                        "#DecreaseCTR8loop", [(set i1:$rT, (int_loop_decrement (i64 imm:$stride)))]>;

let Pattern = [(set i64:$RST, readcyclecounter)] in
def MFTB8 : XFXForm_1_ext<31, 339, 268, (outs g8rc:$RST), (ins),
                          "mfspr $RST, 268", IIC_SprMFTB>,
            PPC970_DGroup_First, PPC970_Unit_FXU;
// Note that encoding mftb using mfspr is now the preferred form,
// and has been since at least ISA v2.03. The mftb instruction has
// now been phased out. Using mfspr, however, is known not to work on
// the POWER3.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Note that encoding mftb using mfspr is now the preferred form,". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Note that encoding mftb using mfspr is now the preferred form,”。 这些声明会进入生成式模式匹配逻辑。

### Lines 578-584

```tablegen
let Defs = [X1], Uses = [X1] in
def DYNALLOC8 : PPCEmitTimePseudo<(outs g8rc:$result), (ins g8rc:$negsize, memri:$fpsi),"#DYNALLOC8",
                       [(set i64:$result,
                             (PPCdynalloc i64:$negsize, iaddr:$fpsi))]>;
def DYNAREAOFFSET8 : PPCEmitTimePseudo<(outs i64imm:$result), (ins memri:$fpsi), "#DYNAREAOFFSET8",
                       [(set i64:$result, (PPCdynareaoffset iaddr:$fpsi))]>;
// Probed alloca to support stack clash protection.
```
- **EN**: Adds declarative TableGen records such as `DYNALLOC8`, `DYNAREAOFFSET8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DYNALLOC8`, `DYNAREAOFFSET8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 585-592

```tablegen
let Defs = [X1], Uses = [X1], hasNoSchedulingInfo = 1 in {
def PROBED_ALLOCA_64 : PPCCustomInserterPseudo<(outs g8rc:$result),
                         (ins g8rc:$negsize, memri:$fpsi), "#PROBED_ALLOCA_64",
                           [(set i64:$result,
                             (PPCprobedalloca i64:$negsize, iaddr:$fpsi))]>;
def PREPARE_PROBED_ALLOCA_64 : PPCEmitTimePseudo<(outs
    g8rc:$fp, g8rc:$actual_negsize),
    (ins g8rc:$negsize, memri:$fpsi), "#PREPARE_PROBED_ALLOCA_64", []>;
```
- **EN**: Adds declarative TableGen records such as `PROBED_ALLOCA_64`, `PREPARE_PROBED_ALLOCA_64` that LLVM later expands into generated tables or helper code. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PROBED_ALLOCA_64`, `PREPARE_PROBED_ALLOCA_64`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段包含调度或处理器模型元数据。

### Lines 593-602

```tablegen
def PREPARE_PROBED_ALLOCA_NEGSIZE_SAME_REG_64 : PPCEmitTimePseudo<(outs
    g8rc:$fp, g8rc:$actual_negsize),
    (ins g8rc:$negsize, memri:$fpsi),
    "#PREPARE_PROBED_ALLOCA_NEGSIZE_SAME_REG_64", []>,
    RegConstraint<"$actual_negsize = $negsize">;
def PROBED_STACKALLOC_64 : PPCEmitTimePseudo<(outs g8rc:$scratch, g8rc:$temp),
    (ins i64imm:$stacksize),
    "#PROBED_STACKALLOC_64", []>;
}
```
- **EN**: Adds declarative TableGen records such as `PREPARE_PROBED_ALLOCA_NEGSIZE_SAME_REG_64`, `PROBED_STACKALLOC_64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PREPARE_PROBED_ALLOCA_NEGSIZE_SAME_REG_64`, `PROBED_STACKALLOC_64`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 603-609

```tablegen
let hasSideEffects = 0 in {
let Defs = [LR8] in {
def MTLR8  : XFXForm_1_ext<31, 467, 8, (outs), (ins g8rc:$RST),
                           "mtlr $RST", IIC_SprMTSPR>,
             PPC970_DGroup_First, PPC970_Unit_FXU;
}
let Uses = [LR8] in {
```
- **EN**: Adds declarative TableGen records such as `MTLR8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MTLR8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 610-616

```tablegen
def MFLR8  : XFXForm_1_ext<31, 339, 8, (outs g8rc:$RST), (ins),
                           "mflr $RST", IIC_SprMFSPR>,
             PPC970_DGroup_First, PPC970_Unit_FXU;
}
} // Interpretation64Bit
}
```
- **EN**: Adds declarative TableGen records such as `MFLR8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MFLR8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 617-623

```tablegen
//===----------------------------------------------------------------------===//
// Fixed point instructions.
//

let PPC970_Unit = 1 in {  // FXU Operations.
let Interpretation64Bit = 1 in {
let hasSideEffects = 0 in {
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 624-635

```tablegen
let isCodeGenOnly = 1 in {

let isReMaterializable = 1, isAsCheapAsAMove = 1, isMoveImm = 1 in {
def LI8  : DForm_2_r0<14, (outs g8rc:$RST), (ins s16imm64:$D),
                      "li $RST, $D", IIC_IntSimple,
                      [(set i64:$RST, imm64SExt16:$D)]>, SExt32To64;
def LIS8 : DForm_2_r0<15, (outs g8rc:$RST), (ins s17imm64:$D),
                      "lis $RST, $D", IIC_IntSimple,
                      [(set i64:$RST, imm16ShiftedSExt:$D)]>, SExt32To64;
}

// Logical ops.
```
- **EN**: Adds declarative TableGen records such as `LI8`, `LIS8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LI8`, `LIS8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 636-643

```tablegen
let isCommutable = 1 in {
defm NAND8: XForm_6r<31, 476, (outs g8rc:$RA), (ins g8rc:$RST, g8rc:$RB),
                     "nand", "$RA, $RST, $RB", IIC_IntSimple,
                     [(set i64:$RA, (not (and i64:$RST, i64:$RB)))]>;
defm AND8 : XForm_6r<31,  28, (outs g8rc:$RA), (ins g8rc:$RST, g8rc:$RB),
                     "and", "$RA, $RST, $RB", IIC_IntSimple,
                     [(set i64:$RA, (and i64:$RST, i64:$RB))]>;
} // isCommutable
```
- **EN**: Adds declarative TableGen records such as `NAND8`, `AND8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `NAND8`, `AND8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 644-650

```tablegen
defm ANDC8: XForm_6r<31,  60, (outs g8rc:$RA), (ins g8rc:$RST, g8rc:$RB),
                     "andc", "$RA, $RST, $RB", IIC_IntSimple,
                     [(set i64:$RA, (and i64:$RST, (not i64:$RB)))]>;
let isCommutable = 1 in {
defm OR8  : XForm_6r<31, 444, (outs g8rc:$RA), (ins g8rc:$RST, g8rc:$RB),
                     "or", "$RA, $RST, $RB", IIC_IntSimple,
                     [(set i64:$RA, (or i64:$RST, i64:$RB))]>;
```
- **EN**: Adds declarative TableGen records such as `ANDC8`, `OR8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ANDC8`, `OR8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 651-657

```tablegen
defm NOR8 : XForm_6r<31, 124, (outs g8rc:$RA), (ins g8rc:$RST, g8rc:$RB),
                     "nor", "$RA, $RST, $RB", IIC_IntSimple,
                     [(set i64:$RA, (not (or i64:$RST, i64:$RB)))]>;
} // isCommutable
defm ORC8 : XForm_6r<31, 412, (outs g8rc:$RA), (ins g8rc:$RST, g8rc:$RB),
                     "orc", "$RA, $RST, $RB", IIC_IntSimple,
                     [(set i64:$RA, (or i64:$RST, (not i64:$RB)))]>;
```
- **EN**: Adds declarative TableGen records such as `NOR8`, `ORC8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `NOR8`, `ORC8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 658-667

```tablegen
let isCommutable = 1 in {
defm EQV8 : XForm_6r<31, 284, (outs g8rc:$RA), (ins g8rc:$RST, g8rc:$RB),
                     "eqv", "$RA, $RST, $RB", IIC_IntSimple,
                     [(set i64:$RA, (not (xor i64:$RST, i64:$RB)))]>;
defm XOR8 : XForm_6r<31, 316, (outs g8rc:$RA), (ins g8rc:$RST, g8rc:$RB),
                     "xor", "$RA, $RST, $RB", IIC_IntSimple,
                     [(set i64:$RA, (xor i64:$RST, i64:$RB))]>;
} // let isCommutable = 1

// Logical ops with immediate.
```
- **EN**: Adds declarative TableGen records such as `EQV8`, `XOR8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `EQV8`, `XOR8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 668-677

```tablegen
let Defs = [CR0] in {
def ANDI8_rec  : DForm_4<28, (outs g8rc:$RA), (ins g8rc:$RST, u16imm64:$D),
                      "andi. $RA, $RST, $D", IIC_IntGeneral,
                      [(set i64:$RA, (and i64:$RST, immZExt16:$D))]>,
                      isRecordForm, SExt32To64, ZExt32To64;
def ANDIS8_rec : DForm_4<29, (outs g8rc:$RA), (ins g8rc:$RST, u16imm64:$D),
                     "andis. $RA, $RST, $D", IIC_IntGeneral,
                    [(set i64:$RA, (and i64:$RST, imm16ShiftedZExt:$D))]>,
                     isRecordForm, ZExt32To64;
}
```
- **EN**: Adds declarative TableGen records such as `ANDI8_rec`, `ANDIS8_rec` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ANDI8_rec`, `ANDIS8_rec`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 678-686

```tablegen
def ORI8    : DForm_4<24, (outs g8rc:$RA), (ins g8rc:$RST, u16imm64:$D),
                      "ori $RA, $RST, $D", IIC_IntSimple,
                      [(set i64:$RA, (or i64:$RST, immZExt16:$D))]>;
def ORIS8   : DForm_4<25, (outs g8rc:$RA), (ins g8rc:$RST, u16imm64:$D),
                      "oris $RA, $RST, $D", IIC_IntSimple,
                    [(set i64:$RA, (or i64:$RST, imm16ShiftedZExt:$D))]>;
def XORI8   : DForm_4<26, (outs g8rc:$RA), (ins g8rc:$RST, u16imm64:$D),
                      "xori $RA, $RST, $D", IIC_IntSimple,
                      [(set i64:$RA, (xor i64:$RST, immZExt16:$D))]>;
```
- **EN**: Adds declarative TableGen records such as `ORI8`, `ORIS8`, `XORI8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ORI8`, `ORIS8`, `XORI8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 687-698

```tablegen
def XORIS8  : DForm_4<27, (outs g8rc:$RA), (ins g8rc:$RST, u16imm64:$D),
                      "xoris $RA, $RST, $D", IIC_IntSimple,
                   [(set i64:$RA, (xor i64:$RST, imm16ShiftedZExt:$D))]>;

let isCommutable = 1 in
defm ADD8  : XOForm_1rx<31, 266, (outs g8rc:$RT), (ins g8rc:$RA, g8rc:$RB),
                        "add", "$RT, $RA, $RB", IIC_IntSimple,
                        [(set i64:$RT, (add i64:$RA, i64:$RB))]>;
// ADD8 has a special form: reg = ADD8(reg, sym@tls) for use by the
// initial-exec thread-local storage model.  We need to forbid r0 here -
// while it works for add just fine, the linker can relax this to local-exec
// addi, which won't work for r0.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "ADD8 has a special form: reg = ADD8(reg, sym@tls) for use by the". Notable symbols in this range include `XORIS8`, `ADD8`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“ADD8 has a special form: reg = ADD8(reg, sym@tls) for use by the”。 该区间中较显眼的符号包括 `XORIS8`, `ADD8`。

### Lines 699-706

```tablegen
def ADD8TLS  : XOForm_1<31, 266, 0, (outs g8rc:$RT), (ins g8rc_nox0:$RA, tlsreg:$RB),
                        "add $RT, $RA, $RB", IIC_IntSimple,
                        [(set i64:$RT, (add i64:$RA, tglobaltlsaddr:$RB))]>;
let mayLoad = 1 in {
def LBZXTLS : XForm_1<31,  87, (outs g8rc:$RST), (ins ptr_rc_nor0:$RA, tlsreg:$RB),
                      "lbzx $RST, $RA, $RB", IIC_LdStLoad, []>;
def LHZXTLS : XForm_1<31, 279, (outs g8rc:$RST), (ins ptr_rc_nor0:$RA, tlsreg:$RB),
                      "lhzx $RST, $RA, $RB", IIC_LdStLoad, []>;
```
- **EN**: Adds declarative TableGen records such as `ADD8TLS`, `LBZXTLS`, `LHZXTLS` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ADD8TLS`, `LBZXTLS`, `LHZXTLS`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 707-714

```tablegen
def LHAXTLS : XForm_1<31, 343, (outs g8rc:$RST), (ins ptr_rc_nor0:$RA, tlsreg:$RB),
                      "lhax $RST, $RA, $RB", IIC_LdStLoad, []>;
def LWZXTLS : XForm_1<31,  23, (outs g8rc:$RST), (ins ptr_rc_nor0:$RA, tlsreg:$RB),
                      "lwzx $RST, $RA, $RB", IIC_LdStLoad, []>;
def LWAXTLS : XForm_1<31, 341, (outs g8rc:$RST), (ins ptr_rc_nor0:$RA, tlsreg:$RB),
                      "lwax $RST, $RA, $RB", IIC_LdStLoad, []>;
def LDXTLS  : XForm_1<31,  21, (outs g8rc:$RST), (ins ptr_rc_nor0:$RA, tlsreg:$RB),
                      "ldx $RST, $RA, $RB", IIC_LdStLD, []>, isPPC64;
```
- **EN**: Adds declarative TableGen records such as `LHAXTLS`, `LWZXTLS`, `LWAXTLS` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LHAXTLS`, `LWZXTLS`, `LWAXTLS`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 715-722

```tablegen
def LBZXTLS_32 : XForm_1<31,  87, (outs gprc:$RST), (ins ptr_rc_nor0:$RA, tlsreg:$RB),
                         "lbzx $RST, $RA, $RB", IIC_LdStLoad, []>;
def LHZXTLS_32 : XForm_1<31, 279, (outs gprc:$RST), (ins ptr_rc_nor0:$RA, tlsreg:$RB),
                         "lhzx $RST, $RA, $RB", IIC_LdStLoad, []>;
def LHAXTLS_32 : XForm_1<31, 343, (outs gprc:$RST), (ins ptr_rc_nor0:$RA, tlsreg:$RB),
                         "lhax $RST, $RA, $RB", IIC_LdStLoad, []>;
def LWZXTLS_32 : XForm_1<31,  23, (outs gprc:$RST), (ins ptr_rc_nor0:$RA, tlsreg:$RB),
                         "lwzx $RST, $RA, $RB", IIC_LdStLoad, []>;
```
- **EN**: Adds declarative TableGen records such as `LBZXTLS_32`, `LHZXTLS_32`, `LHAXTLS_32` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LBZXTLS_32`, `LHZXTLS_32`, `LHAXTLS_32`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 723-729

```tablegen
def LWAXTLS_32 : XForm_1<31, 341, (outs gprc:$RST), (ins ptr_rc_nor0:$RA, tlsreg:$RB),
                         "lwax $RST, $RA, $RB", IIC_LdStLoad, []>;

}
let mayLoad = 1, Predicates = [HasFPU] in {
def LFSXTLS : XForm_25<31, 535, (outs f4rc:$RST), (ins ptr_rc_nor0:$RA, tlsreg:$RB),
                       "lfsx $RST, $RA, $RB", IIC_LdStLFD, []>;
```
- **EN**: Adds declarative TableGen records such as `LWAXTLS_32`, `LFSXTLS` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LWAXTLS_32`, `LFSXTLS`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 730-737

```tablegen
def LFDXTLS : XForm_25<31, 599, (outs f8rc:$RST), (ins ptr_rc_nor0:$RA, tlsreg:$RB),
                       "lfdx $RST, $RA, $RB", IIC_LdStLFD, []>;
}

let mayStore = 1 in {
def STBXTLS : XForm_8<31, 215, (outs), (ins g8rc:$RST, ptr_rc_nor0:$RA, tlsreg:$RB),
                      "stbx $RST, $RA, $RB", IIC_LdStStore, []>,
                      PPC970_DGroup_Cracked;
```
- **EN**: Adds declarative TableGen records such as `LFDXTLS`, `STBXTLS` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LFDXTLS`, `STBXTLS`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 738-746

```tablegen
def STHXTLS : XForm_8<31, 407, (outs), (ins g8rc:$RST, ptr_rc_nor0:$RA, tlsreg:$RB),
                      "sthx $RST, $RA, $RB", IIC_LdStStore, []>,
                      PPC970_DGroup_Cracked;
def STWXTLS : XForm_8<31, 151, (outs), (ins g8rc:$RST, ptr_rc_nor0:$RA, tlsreg:$RB),
                      "stwx $RST, $RA, $RB", IIC_LdStStore, []>,
                      PPC970_DGroup_Cracked;
def STDXTLS  : XForm_8<31, 149, (outs), (ins g8rc:$RST, ptr_rc_nor0:$RA, tlsreg:$RB),
                       "stdx $RST, $RA, $RB", IIC_LdStSTD, []>, isPPC64,
                       PPC970_DGroup_Cracked;
```
- **EN**: Adds declarative TableGen records such as `STHXTLS`, `STWXTLS`, `STDXTLS` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STHXTLS`, `STWXTLS`, `STDXTLS`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 747-757

```tablegen
def STBXTLS_32 : XForm_8<31, 215, (outs), (ins gprc:$RST, ptr_rc_nor0:$RA, tlsreg:$RB),
                         "stbx $RST, $RA, $RB", IIC_LdStStore, []>,
                         PPC970_DGroup_Cracked;
def STHXTLS_32 : XForm_8<31, 407, (outs), (ins gprc:$RST, ptr_rc_nor0:$RA, tlsreg:$RB),
                         "sthx $RST, $RA, $RB", IIC_LdStStore, []>,
                         PPC970_DGroup_Cracked;
def STWXTLS_32 : XForm_8<31, 151, (outs), (ins gprc:$RST, ptr_rc_nor0:$RA, tlsreg:$RB),
                         "stwx $RST, $RA, $RB", IIC_LdStStore, []>,
                         PPC970_DGroup_Cracked;

}
```
- **EN**: Adds declarative TableGen records such as `STBXTLS_32`, `STHXTLS_32`, `STWXTLS_32` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STBXTLS_32`, `STHXTLS_32`, `STWXTLS_32`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 758-766

```tablegen
let mayStore = 1, Predicates = [HasFPU] in {
def STFSXTLS : XForm_8<31, 663, (outs), (ins f4rc:$RST, ptr_rc_nor0:$RA, tlsreg:$RB),
                       "stfsx $RST, $RA, $RB", IIC_LdStSTFD, []>,
                       PPC970_DGroup_Cracked;
def STFDXTLS : XForm_8<31, 727, (outs), (ins f8rc:$RST, ptr_rc_nor0:$RA, tlsreg:$RB),
                       "stfdx $RST, $RA, $RB", IIC_LdStSTFD, []>,
                       PPC970_DGroup_Cracked;
}
```
- **EN**: Adds declarative TableGen records such as `STFSXTLS`, `STFDXTLS` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STFSXTLS`, `STFDXTLS`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 767-773

```tablegen
let isCommutable = 1 in
defm ADDC8 : XOForm_1rc<31, 10, 0, (outs g8rc:$RT), (ins g8rc:$RA, g8rc:$RB),
                        "addc", "$RT, $RA, $RB", IIC_IntGeneral,
                        [(set i64:$RT, (PPCaddc i64:$RA, i64:$RB))]>,
                        PPC970_DGroup_Cracked;

let Defs = [CARRY] in
```
- **EN**: Adds declarative TableGen records such as `ADDC8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ADDC8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 774-783

```tablegen
def ADDIC8 : DForm_2<12, (outs g8rc:$RST), (ins g8rc:$RA, s16imm64:$D),
                     "addic $RST, $RA, $D", IIC_IntGeneral,
                     [(set i64:$RST, (PPCaddc i64:$RA, imm64SExt16:$D))]>;
def ADDI8  : DForm_2<14, (outs g8rc:$RST), (ins g8rc_nox0:$RA, s16imm64:$D),
                     "addi $RST, $RA, $D", IIC_IntSimple,
                     [(set i64:$RST, (add i64:$RA, imm64SExt16:$D))]>;
def ADDIS8 : DForm_2<15, (outs g8rc:$RST), (ins g8rc_nox0:$RA, s17imm64:$D),
                     "addis $RST, $RA, $D", IIC_IntSimple,
                     [(set i64:$RST, (add i64:$RA, imm16ShiftedSExt:$D))]>;
```
- **EN**: Adds declarative TableGen records such as `ADDIC8`, `ADDI8`, `ADDIS8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ADDIC8`, `ADDI8`, `ADDIS8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 784-793

```tablegen
def LA8     : DForm_2<14, (outs g8rc:$RST), (ins g8rc_nox0:$RA, s16imm64:$D),
                     "la $RST, $D($RA)", IIC_IntGeneral,
                     [(set i64:$RST, (add i64:$RA,
                                    (PPClo tglobaladdr:$D, 0)))]>, MemriOp;

let Defs = [CARRY] in {
def SUBFIC8: DForm_2< 8, (outs g8rc:$RST), (ins g8rc:$RA, s16imm64:$D),
                     "subfic $RST, $RA, $D", IIC_IntGeneral,
                     [(set i64:$RST, (PPCsubc imm64SExt16:$D, i64:$RA))]>;
}
```
- **EN**: Adds declarative TableGen records such as `LA8`, `SUBFIC8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LA8`, `SUBFIC8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 794-800

```tablegen
defm SUBFC8 : XOForm_1rc<31, 8, 0, (outs g8rc:$RT), (ins g8rc:$RA, g8rc:$RB),
                        "subfc", "$RT, $RA, $RB", IIC_IntGeneral,
                        [(set i64:$RT, (PPCsubc i64:$RB, i64:$RA))]>,
                        PPC970_DGroup_Cracked;
defm SUBF8 : XOForm_1rx<31, 40, (outs g8rc:$RT), (ins g8rc:$RA, g8rc:$RB),
                        "subf", "$RT, $RA, $RB", IIC_IntGeneral,
                        [(set i64:$RT, (sub i64:$RB, i64:$RA))]>;
```
- **EN**: Adds declarative TableGen records such as `SUBFC8`, `SUBF8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SUBFC8`, `SUBF8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 801-808

```tablegen
defm NEG8    : XOForm_3r<31, 104, 0, (outs g8rc:$RT), (ins g8rc:$RA),
                        "neg", "$RT, $RA", IIC_IntSimple,
                        [(set i64:$RT, (ineg i64:$RA))]>;
let Uses = [CARRY] in {
let isCommutable = 1 in
defm ADDE8   : XOForm_1rc<31, 138, 0, (outs g8rc:$RT), (ins g8rc:$RA, g8rc:$RB),
                          "adde", "$RT, $RA, $RB", IIC_IntGeneral,
                          [(set i64:$RT, (PPCadde i64:$RA, i64:$RB, CARRY))]>;
```
- **EN**: Adds declarative TableGen records such as `NEG8`, `ADDE8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `NEG8`, `ADDE8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 809-817

```tablegen
defm ADDME8  : XOForm_3rc<31, 234, 0, (outs g8rc:$RT), (ins g8rc:$RA),
                          "addme", "$RT, $RA", IIC_IntGeneral,
                          [(set i64:$RT, (PPCadde i64:$RA, -1, CARRY))]>;
defm ADDZE8  : XOForm_3rc<31, 202, 0, (outs g8rc:$RT), (ins g8rc:$RA),
                          "addze", "$RT, $RA", IIC_IntGeneral,
                          [(set i64:$RT, (PPCadde i64:$RA, 0, CARRY))]>;
defm SUBFE8  : XOForm_1rc<31, 136, 0, (outs g8rc:$RT), (ins g8rc:$RA, g8rc:$RB),
                          "subfe", "$RT, $RA, $RB", IIC_IntGeneral,
                          [(set i64:$RT, (PPCsube i64:$RB, i64:$RA, CARRY))]>;
```
- **EN**: Adds declarative TableGen records such as `ADDME8`, `ADDZE8`, `SUBFE8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ADDME8`, `ADDZE8`, `SUBFE8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 818-829

```tablegen
defm SUBFME8 : XOForm_3rc<31, 232, 0, (outs g8rc:$RT), (ins g8rc:$RA),
                          "subfme", "$RT, $RA", IIC_IntGeneral,
                          [(set i64:$RT, (PPCsube -1, i64:$RA, CARRY))]>;
defm SUBFZE8 : XOForm_3rc<31, 200, 0, (outs g8rc:$RT), (ins g8rc:$RA),
                          "subfze", "$RT, $RA", IIC_IntGeneral,
                          [(set i64:$RT, (PPCsube 0, i64:$RA, CARRY))]>;
}
} // isCodeGenOnly

// FIXME: Duplicating this for the asm parser should be unnecessary, but the
// previous definition must be marked as CodeGen only to prevent decoding
// conflicts.
```
- **EN**: Adds declarative TableGen records such as `SUBFME8`, `SUBFZE8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SUBFME8`, `SUBFZE8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 830-836

```tablegen
let isAsmParserOnly = 1 in {
def ADD8TLS_ : XOForm_1<31, 266, 0, (outs g8rc:$RT), (ins g8rc:$RA, tlsreg:$RB),
                        "add $RT, $RA, $RB", IIC_IntSimple, []>;

let mayLoad = 1 in {
def LBZXTLS_ : XForm_1<31,  87, (outs g8rc:$RST), (ins ptr_rc_nor0:$RA, tlsreg:$RB),
                      "lbzx $RST, $RA, $RB", IIC_LdStLoad, []>;
```
- **EN**: Adds declarative TableGen records such as `ADD8TLS_`, `LBZXTLS_` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ADD8TLS_`, `LBZXTLS_`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 837-844

```tablegen
def LHZXTLS_ : XForm_1<31, 279, (outs g8rc:$RST), (ins ptr_rc_nor0:$RA, tlsreg:$RB),
                      "lhzx $RST, $RA, $RB", IIC_LdStLoad, []>;
def LHAXTLS_ : XForm_1<31, 343, (outs g8rc:$RST), (ins ptr_rc_nor0:$RA, tlsreg:$RB),
                       "lhax $RST, $RA, $RB", IIC_LdStLoad, []>;
def LWZXTLS_ : XForm_1<31,  23, (outs g8rc:$RST), (ins ptr_rc_nor0:$RA, tlsreg:$RB),
                      "lwzx $RST, $RA, $RB", IIC_LdStLoad, []>;
def LWAXTLS_ : XForm_1<31, 341, (outs g8rc:$RST), (ins ptr_rc_nor0:$RA, tlsreg:$RB),
                       "lwax $RST, $RA, $RB", IIC_LdStLoad, []>;
```
- **EN**: Adds declarative TableGen records such as `LHZXTLS_`, `LHAXTLS_`, `LWZXTLS_` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LHZXTLS_`, `LHAXTLS_`, `LWZXTLS_`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 845-851

```tablegen
def LDXTLS_  : XForm_1<31,  21, (outs g8rc:$RST), (ins ptr_rc_nor0:$RA, tlsreg:$RB),
                      "ldx $RST, $RA, $RB", IIC_LdStLD, []>, isPPC64;
}

let mayLoad = 1, Predicates = [HasFPU] in {
def LFSXTLS_ : XForm_25<31, 535, (outs f4rc:$RST), (ins ptr_rc_nor0:$RA, tlsreg:$RB),
                        "lfsx $RST, $RA, $RB", IIC_LdStLFD, []>;
```
- **EN**: Adds declarative TableGen records such as `LDXTLS_`, `LFSXTLS_` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LDXTLS_`, `LFSXTLS_`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 852-859

```tablegen
def LFDXTLS_ : XForm_25<31, 599, (outs f8rc:$RST), (ins ptr_rc_nor0:$RA, tlsreg:$RB),
                        "lfdx $RST, $RA, $RB", IIC_LdStLFD, []>;
}

let mayStore = 1 in {
def STBXTLS_ : XForm_8<31, 215, (outs), (ins g8rc:$RST, ptr_rc_nor0:$RA, tlsreg:$RB),
                      "stbx $RST, $RA, $RB", IIC_LdStStore, []>,
                      PPC970_DGroup_Cracked;
```
- **EN**: Adds declarative TableGen records such as `LFDXTLS_`, `STBXTLS_` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LFDXTLS_`, `STBXTLS_`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 860-870

```tablegen
def STHXTLS_ : XForm_8<31, 407, (outs), (ins g8rc:$RST, ptr_rc_nor0:$RA, tlsreg:$RB),
                      "sthx $RST, $RA, $RB", IIC_LdStStore, []>,
                      PPC970_DGroup_Cracked;
def STWXTLS_ : XForm_8<31, 151, (outs), (ins g8rc:$RST, ptr_rc_nor0:$RA, tlsreg:$RB),
                      "stwx $RST, $RA, $RB", IIC_LdStStore, []>,
                      PPC970_DGroup_Cracked;
def STDXTLS_  : XForm_8<31, 149, (outs), (ins g8rc:$RST, ptr_rc_nor0:$RA, tlsreg:$RB),
                       "stdx $RST, $RA, $RB", IIC_LdStSTD, []>, isPPC64,
                       PPC970_DGroup_Cracked;
}
```
- **EN**: Adds declarative TableGen records such as `STHXTLS_`, `STWXTLS_`, `STDXTLS_` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STHXTLS_`, `STWXTLS_`, `STDXTLS_`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 871-880

```tablegen
let mayStore = 1, Predicates = [HasFPU] in {
def STFSXTLS_ : XForm_8<31, 663, (outs), (ins f4rc:$RST, ptr_rc_nor0:$RA, tlsreg:$RB),
                        "stfsx $RST, $RA, $RB", IIC_LdStSTFD, []>,
                        PPC970_DGroup_Cracked;
def STFDXTLS_ : XForm_8<31, 727, (outs), (ins f8rc:$RST, ptr_rc_nor0:$RA, tlsreg:$RB),
                        "stfdx $RST, $RA, $RB", IIC_LdStSTFD, []>,
                        PPC970_DGroup_Cracked;
}
}
```
- **EN**: Adds declarative TableGen records such as `STFSXTLS_`, `STFDXTLS_` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STFSXTLS_`, `STFDXTLS_`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 881-891

```tablegen
let isCommutable = 1 in {
defm MULHD : XOForm_1r<31, 73, 0, (outs g8rc:$RT), (ins g8rc:$RA, g8rc:$RB),
                       "mulhd", "$RT, $RA, $RB", IIC_IntMulHW,
                       [(set i64:$RT, (mulhs i64:$RA, i64:$RB))]>;
defm MULHDU : XOForm_1r<31, 9, 0, (outs g8rc:$RT), (ins g8rc:$RA, g8rc:$RB),
                       "mulhdu", "$RT, $RA, $RB", IIC_IntMulHWU,
                       [(set i64:$RT, (mulhu i64:$RA, i64:$RB))]>;
} // isCommutable
}
} // Interpretation64Bit
```
- **EN**: Adds declarative TableGen records such as `MULHD`, `MULHDU` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MULHD`, `MULHDU`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 892-898

```tablegen
let isCompare = 1, hasSideEffects = 0 in {
  def CMPD   : XForm_16_ext<31, 0, (outs crrc:$BF), (ins g8rc:$RA, g8rc:$RB),
                            "cmpd $BF, $RA, $RB", IIC_IntCompare>, isPPC64;
  def CMPLD  : XForm_16_ext<31, 32, (outs crrc:$BF), (ins g8rc:$RA, g8rc:$RB),
                            "cmpld $BF, $RA, $RB", IIC_IntCompare>, isPPC64;
  def CMPDI  : DForm_5_ext<11, (outs crrc:$BF), (ins g8rc:$RA, s16imm64:$D),
                           "cmpdi $BF, $RA, $D", IIC_IntCompare>, isPPC64;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 899-906

```tablegen
  def CMPLDI : DForm_6_ext<10, (outs crrc:$BF), (ins g8rc:$RA, u16imm64:$D),
                           "cmpldi $BF, $RA, $D",
                           IIC_IntCompare>, isPPC64;
  let Interpretation64Bit = 1, isCodeGenOnly = 1 in
  def CMPRB8 : X_BF3_L1_RS5_RS5<31, 192, (outs crrc:$BF),
                                (ins u1imm:$L, g8rc:$RA, g8rc:$RB),
                                "cmprb $BF, $L, $RA, $RB", IIC_IntCompare, []>,
               Requires<[IsISA3_0]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 907-915

```tablegen
  def CMPEQB : X_BF3_RS5_RS5<31, 224, (outs crrc:$BF),
                             (ins g8rc:$RA, g8rc:$RB), "cmpeqb $BF, $RA, $RB",
                             IIC_IntCompare, []>, Requires<[IsISA3_0]>;
}

let hasSideEffects = 0 in {
defm SLD  : XForm_6r<31,  27, (outs g8rc:$RA), (ins g8rc:$RST, gprc:$RB),
                     "sld", "$RA, $RST, $RB", IIC_IntRotateD,
                     [(set i64:$RA, (PPCshl i64:$RST, i32:$RB))]>, isPPC64;
```
- **EN**: Adds declarative TableGen records such as `SLD` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SLD`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 916-922

```tablegen
defm SRD  : XForm_6r<31, 539, (outs g8rc:$RA), (ins g8rc:$RST, gprc:$RB),
                     "srd", "$RA, $RST, $RB", IIC_IntRotateD,
                     [(set i64:$RA, (PPCsrl i64:$RST, i32:$RB))]>, isPPC64;
defm SRAD : XForm_6rc<31, 794, (outs g8rc:$RA), (ins g8rc:$RST, gprc:$RB),
                      "srad", "$RA, $RST, $RB", IIC_IntRotateD,
                      [(set i64:$RA, (PPCsra i64:$RST, i32:$RB))]>, isPPC64;
```
- **EN**: Adds declarative TableGen records such as `SRD`, `SRAD` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SRD`, `SRAD`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 923-930

```tablegen
let Interpretation64Bit = 1, isCodeGenOnly = 1 in {
defm CNTLZW8 : XForm_11r<31,  26, (outs g8rc:$RA), (ins g8rc:$RST),
                        "cntlzw", "$RA, $RST", IIC_IntGeneral, []>,
                        ZExt32To64, SExt32To64;
defm CNTTZW8 : XForm_11r<31, 538, (outs g8rc:$RA), (ins g8rc:$RST),
                        "cnttzw", "$RA, $RST", IIC_IntGeneral, []>,
               Requires<[IsISA3_0]>, ZExt32To64, SExt32To64;
```
- **EN**: Adds declarative TableGen records such as `CNTLZW8`, `CNTTZW8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CNTLZW8`, `CNTTZW8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 931-937

```tablegen
defm EXTSB8 : XForm_11r<31, 954, (outs g8rc:$RA), (ins g8rc:$RST),
                        "extsb", "$RA, $RST", IIC_IntSimple,
                        [(set i64:$RA, (sext_inreg i64:$RST, i8))]>, SExt32To64;
defm EXTSH8 : XForm_11r<31, 922, (outs g8rc:$RA), (ins g8rc:$RST),
                        "extsh", "$RA, $RST", IIC_IntSimple,
                        [(set i64:$RA, (sext_inreg i64:$RST, i16))]>, SExt32To64;
```
- **EN**: Adds declarative TableGen records such as `EXTSB8`, `EXTSH8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `EXTSB8`, `EXTSH8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 938-946

```tablegen
defm SLW8  : XForm_6r<31,  24, (outs g8rc:$RA), (ins g8rc:$RST, g8rc:$RB),
                      "slw", "$RA, $RST, $RB", IIC_IntGeneral, []>, ZExt32To64;
defm SRW8  : XForm_6r<31, 536, (outs g8rc:$RA), (ins g8rc:$RST, g8rc:$RB),
                      "srw", "$RA, $RST, $RB", IIC_IntGeneral, []>, ZExt32To64;

defm SRAW8 : XForm_6rc<31, 792, (outs g8rc:$RA), (ins g8rc:$RST, g8rc:$RB),
                      "sraw", "$RA, $RST, $RB", IIC_IntShift,
                      []>, SExt32To64;
```
- **EN**: Adds declarative TableGen records such as `SLW8`, `SRW8`, `SRAW8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SLW8`, `SRW8`, `SRAW8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 947-953

```tablegen
defm SRAWI8 : XForm_10rc<31, 824, (outs g8rc:$RA), (ins g8rc:$RST, u5imm:$RB),
            "srawi", "$RA, $RST, $RB", IIC_IntShift, []>, SExt32To64;

} // Interpretation64Bit

// For fast-isel:
let isCodeGenOnly = 1 in {
```
- **EN**: Adds declarative TableGen records such as `SRAWI8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SRAWI8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 954-961

```tablegen
def EXTSB8_32_64 : XForm_11<31, 954, (outs g8rc:$RA), (ins gprc:$RST),
                           "extsb $RA, $RST", IIC_IntSimple, []>, isPPC64,
                           SExt32To64;
def EXTSH8_32_64 : XForm_11<31, 922, (outs g8rc:$RA), (ins gprc:$RST),
                           "extsh $RA, $RST", IIC_IntSimple, []>, isPPC64,
                           SExt32To64;
} // isCodeGenOnly for fast-isel
```
- **EN**: Adds declarative TableGen records such as `EXTSB8_32_64`, `EXTSH8_32_64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `EXTSB8_32_64`, `EXTSH8_32_64`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 962-970

```tablegen
defm EXTSW  : XForm_11r<31, 986, (outs g8rc:$RA), (ins g8rc:$RST),
                        "extsw", "$RA, $RST", IIC_IntSimple,
                        [(set i64:$RA, (sext_inreg i64:$RST, i32))]>, isPPC64,
                        SExt32To64;
let Interpretation64Bit = 1, isCodeGenOnly = 1 in
defm EXTSW_32_64 : XForm_11r<31, 986, (outs g8rc:$RA), (ins gprc:$RST),
                             "extsw", "$RA, $RST", IIC_IntSimple,
                             [(set i64:$RA, (sext i32:$RST))]>, isPPC64,
                             SExt32To64;
```
- **EN**: Adds declarative TableGen records such as `EXTSW`, `EXTSW_32_64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `EXTSW`, `EXTSW_32_64`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 971-979

```tablegen
let isCodeGenOnly = 1 in
def EXTSW_32 : XForm_11<31, 986, (outs gprc:$RA), (ins gprc:$RST),
                        "extsw $RA, $RST", IIC_IntSimple,
                        []>, isPPC64;

defm SRADI  : XSForm_1rc<31, 413, (outs g8rc:$RA), (ins g8rc:$RS, u6imm:$SH),
                         "sradi", "$RA, $RS, $SH", IIC_IntRotateDI,
                         [(set i64:$RA, (sra i64:$RS, (i32 imm:$SH)))]>, isPPC64;
```
- **EN**: Adds declarative TableGen records such as `EXTSW_32`, `SRADI` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `EXTSW_32`, `SRADI`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 980-987

```tablegen
let Interpretation64Bit = 1, isCodeGenOnly = 1 in
defm EXTSWSLI_32_64 : XSForm_1r<31, 445, (outs g8rc:$RA),
                                (ins gprc:$RS, u6imm:$SH),
                                "extswsli", "$RA, $RS, $SH", IIC_IntRotateDI,
                                [(set i64:$RA,
                                      (PPCextswsli i32:$RS, (i32 imm:$SH)))]>,
                                isPPC64, Requires<[IsISA3_0]>;
```
- **EN**: Adds declarative TableGen records such as `EXTSWSLI_32_64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `EXTSWSLI_32_64`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 988-996

```tablegen
defm EXTSWSLI : XSForm_1rc<31, 445, (outs g8rc:$RA), (ins g8rc:$RS, u6imm:$SH),
                           "extswsli", "$RA, $RS, $SH", IIC_IntRotateDI,
                           []>, isPPC64, Requires<[IsISA3_0]>;

// For fast-isel:
let isCodeGenOnly = 1, Defs = [CARRY] in
def SRADI_32  : XSForm_1<31, 413, (outs gprc:$RA), (ins gprc:$RS, u6imm:$SH),
                         "sradi $RA, $RS, $SH", IIC_IntRotateDI, []>, isPPC64;
```
- **EN**: Adds declarative TableGen records such as `EXTSWSLI`, `SRADI_32` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `EXTSWSLI`, `SRADI_32`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 997-1004

```tablegen
defm CNTLZD : XForm_11r<31,  58, (outs g8rc:$RA), (ins g8rc:$RST),
                        "cntlzd", "$RA, $RST", IIC_IntGeneral,
                        [(set i64:$RA, (ctlz i64:$RST))]>,
                        ZExt32To64, SExt32To64;
defm CNTTZD : XForm_11r<31, 570, (outs g8rc:$RA), (ins g8rc:$RST),
                        "cnttzd", "$RA, $RST", IIC_IntGeneral,
                        [(set i64:$RA, (cttz i64:$RST))]>, Requires<[IsISA3_0]>,
                        ZExt32To64, SExt32To64;
```
- **EN**: Adds declarative TableGen records such as `CNTLZD`, `CNTTZD` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CNTLZD`, `CNTTZD`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1005-1013

```tablegen
def POPCNTD : XForm_11<31, 506, (outs g8rc:$RA), (ins g8rc:$RST),
                       "popcntd $RA, $RST", IIC_IntGeneral,
                       [(set i64:$RA, (ctpop i64:$RST))]>,
                       ZExt32To64, SExt32To64;
def BPERMD : XForm_6<31, 252, (outs g8rc:$RA), (ins g8rc:$RST, g8rc:$RB),
                     "bpermd $RA, $RST, $RB", IIC_IntGeneral,
                     [(set i64:$RA, (int_ppc_bpermd g8rc:$RST, g8rc:$RB))]>,
                     isPPC64, Requires<[HasBPERMD]>;
```
- **EN**: Adds declarative TableGen records such as `POPCNTD`, `BPERMD` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `POPCNTD`, `BPERMD`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1014-1021

```tablegen
let isCodeGenOnly = 1, isCommutable = 1 in
def CMPB8 : XForm_6<31, 508, (outs g8rc:$RA), (ins g8rc:$RST, g8rc:$RB),
                    "cmpb $RA, $RST, $RB", IIC_IntGeneral,
                    [(set i64:$RA, (PPCcmpb i64:$RST, i64:$RB))]>;

// popcntw also does a population count on the high 32 bits (storing the
// results in the high 32-bits of the output). We'll ignore that here (which is
// safe because we never separately use the high part of the 64-bit registers).
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "popcntw also does a population count on the high 32 bits (storing the". Notable symbols in this range include `CMPB8`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“popcntw also does a population count on the high 32 bits (storing the”。 该区间中较显眼的符号包括 `CMPB8`。

### Lines 1022-1030

```tablegen
def POPCNTW : XForm_11<31, 378, (outs gprc:$RA), (ins gprc:$RST),
                       "popcntw $RA, $RST", IIC_IntGeneral,
                       [(set i32:$RA, (ctpop i32:$RST))]>;

let isCodeGenOnly = 1 in {
def POPCNTB8 : XForm_11<31, 122, (outs g8rc:$RA), (ins g8rc:$RST),
                        "popcntb $RA, $RST", IIC_IntGeneral,
                        [(set i64:$RA, (int_ppc_popcntb i64:$RST))]>;
```
- **EN**: Adds declarative TableGen records such as `POPCNTW`, `POPCNTB8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `POPCNTW`, `POPCNTB8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1031-1041

```tablegen
def CDTBCD8 : XForm_11<31, 282, (outs g8rc:$RA), (ins g8rc:$RST),
                       "cdtbcd $RA, $RST", IIC_IntGeneral,
                       [(set i64:$RA, (int_ppc_cdtbcdd i64:$RST))]>;
def CBCDTD8 : XForm_11<31, 314, (outs g8rc:$RA), (ins g8rc:$RST),
                       "cbcdtd $RA, $RST", IIC_IntGeneral,
                       [(set i64:$RA, (int_ppc_cbcdtdd i64:$RST))]>;
def ADDG6S8 : XOForm_1<31, 74, 0, (outs g8rc:$RT), (ins g8rc:$RA, g8rc:$RB),
                       "addg6s $RT, $RA, $RB", IIC_IntGeneral,
                       [(set i64:$RT, (int_ppc_addg6sd i64:$RA, i64:$RB))]>;
}
```
- **EN**: Adds declarative TableGen records such as `CDTBCD8`, `CBCDTD8`, `ADDG6S8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CDTBCD8`, `CBCDTD8`, `ADDG6S8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1042-1052

```tablegen
defm DIVD  : XOForm_1rcr<31, 489, 0, (outs g8rc:$RT), (ins g8rc:$RA, g8rc:$RB),
                          "divd", "$RT, $RA, $RB", IIC_IntDivD,
                          [(set i64:$RT, (sdiv i64:$RA, i64:$RB))]>, isPPC64;
defm DIVDU : XOForm_1rcr<31, 457, 0, (outs g8rc:$RT), (ins g8rc:$RA, g8rc:$RB),
                          "divdu", "$RT, $RA, $RB", IIC_IntDivD,
                          [(set i64:$RT, (udiv i64:$RA, i64:$RB))]>, isPPC64;
defm DIVDE : XOForm_1rcr<31, 425, 0, (outs g8rc:$RT), (ins g8rc:$RA, g8rc:$RB),
                         "divde", "$RT, $RA, $RB", IIC_IntDivD,
                         [(set i64:$RT, (int_ppc_divde g8rc:$RA, g8rc:$RB))]>,
                         isPPC64, Requires<[HasExtDiv]>;
```
- **EN**: Adds declarative TableGen records such as `DIVD`, `DIVDU`, `DIVDE` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DIVD`, `DIVDU`, `DIVDE`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1053-1062

```tablegen
let Predicates = [IsISA3_0] in {
def MADDHD : VAForm_1a<48, (outs g8rc:$RT), (ins g8rc:$RA, g8rc:$RB, g8rc:$RC),
                       "maddhd $RT, $RA, $RB, $RC", IIC_IntMulHD, []>, isPPC64;
def MADDHDU : VAForm_1a<49,
                       (outs g8rc:$RT), (ins g8rc:$RA, g8rc:$RB, g8rc:$RC),
                       "maddhdu $RT, $RA, $RB, $RC", IIC_IntMulHD, []>, isPPC64;
def MADDLD : VAForm_1a<51, (outs gprc:$RT), (ins gprc:$RA, gprc:$RB, gprc:$RC),
                       "maddld $RT, $RA, $RB, $RC", IIC_IntMulHD,
                       [(set i32:$RT, (add_without_simm16 (mul_without_simm16 i32:$RA, i32:$RB), i32:$RC))]>,
                       isPPC64;
```
- **EN**: Adds declarative TableGen records such as `MADDHD`, `MADDHDU`, `MADDLD` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MADDHD`, `MADDHDU`, `MADDLD`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1063-1071

```tablegen
let Interpretation64Bit = 1, isCodeGenOnly = 1 in {
  def MADDLD8 : VAForm_1a<51,
                       (outs g8rc:$RT), (ins g8rc:$RA, g8rc:$RB, g8rc:$RC),
                       "maddld $RT, $RA, $RB, $RC", IIC_IntMulHD,
                       [(set i64:$RT, (add_without_simm16 (mul_without_simm16 i64:$RA, i64:$RB), i64:$RC))]>,
                       isPPC64;
  def SETB8 : XForm_44<31, 128, (outs g8rc:$RT), (ins crrc:$BFA),
                       "setb $RT, $BFA", IIC_IntGeneral>, isPPC64, SExt32To64;
}
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1072-1081

```tablegen
def ADDPCIS : DXForm<19, 2, (outs g8rc:$RT), (ins i32imm:$D),
                     "addpcis $RT, $D", IIC_BrB, []>, isPPC64;
def MODSD : XForm_8<31, 777, (outs g8rc:$RST), (ins g8rc:$RA, g8rc:$RB),
                        "modsd $RST, $RA, $RB", IIC_IntDivW,
                        [(set i64:$RST, (srem i64:$RA, i64:$RB))]>;
def MODUD : XForm_8<31, 265, (outs g8rc:$RST), (ins g8rc:$RA, g8rc:$RB),
                        "modud $RST, $RA, $RB", IIC_IntDivW,
                        [(set i64:$RST, (urem i64:$RA, i64:$RB))]>;
}
```
- **EN**: Adds declarative TableGen records such as `ADDPCIS`, `MODSD`, `MODUD` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ADDPCIS`, `MODSD`, `MODUD`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1082-1089

```tablegen
defm DIVDEU : XOForm_1rcr<31, 393, 0, (outs g8rc:$RT), (ins g8rc:$RA, g8rc:$RB),
                          "divdeu", "$RT, $RA, $RB", IIC_IntDivD,
                          [(set i64:$RT, (int_ppc_divdeu g8rc:$RA, g8rc:$RB))]>,
                          isPPC64, Requires<[HasExtDiv]>;
let isCommutable = 1 in
defm MULLD : XOForm_1rx<31, 233, (outs g8rc:$RT), (ins g8rc:$RA, g8rc:$RB),
                        "mulld", "$RT, $RA, $RB", IIC_IntMulHD,
                        [(set i64:$RT, (mul i64:$RA, i64:$RB))]>, isPPC64;
```
- **EN**: Adds declarative TableGen records such as `DIVDEU`, `MULLD` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DIVDEU`, `MULLD`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1090-1096

```tablegen
let Interpretation64Bit = 1, isCodeGenOnly = 1 in
def MULLI8 : DForm_2<7, (outs g8rc:$RST), (ins g8rc:$RA, s16imm64:$D),
                       "mulli $RST, $RA, $D", IIC_IntMulLI,
                       [(set i64:$RST, (mul i64:$RA, imm64SExt16:$D))]>;
}

let hasSideEffects = 1 in {
```
- **EN**: Adds declarative TableGen records such as `MULLI8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MULLI8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1097-1107

```tablegen
def DARN : XForm_45<31, 755, (outs g8rc:$RT), (ins u2imm:$L),
                    "darn $RT, $L", IIC_LdStLD>, isPPC64;
}

let hasSideEffects = 0 in {
defm RLDIMI : MDForm_1r<30, 3, (outs g8rc:$RA),
                        (ins g8rc:$RAi, g8rc:$RS, u6imm:$SH, u6imm:$MBE),
                        "rldimi", "$RA, $RS, $SH, $MBE", IIC_IntRotateDI,
                        []>, isPPC64, RegConstraint<"$RAi = $RA">;

// Rotate instructions.
```
- **EN**: Adds declarative TableGen records such as `DARN`, `RLDIMI` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DARN`, `RLDIMI`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1108-1115

```tablegen
defm RLDCL  : MDSForm_1r<30, 8,
                        (outs g8rc:$RA), (ins g8rc:$RS, gprc:$RB, u6imm:$MBE),
                        "rldcl", "$RA, $RS, $RB, $MBE", IIC_IntRotateD,
                        []>, isPPC64;
defm RLDCR  : MDSForm_1r<30, 9,
                        (outs g8rc:$RA), (ins g8rc:$RS, gprc:$RB, u6imm:$MBE),
                        "rldcr", "$RA, $RS, $RB, $MBE", IIC_IntRotateD,
                        []>, isPPC64;
```
- **EN**: Adds declarative TableGen records such as `RLDCL`, `RLDCR` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `RLDCL`, `RLDCR`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1116-1127

```tablegen
defm RLDICL : MDForm_1r<30, 0,
                        (outs g8rc:$RA), (ins g8rc:$RS, u6imm:$SH, u6imm:$MBE),
                        "rldicl", "$RA, $RS, $SH, $MBE", IIC_IntRotateDI,
                        []>, isPPC64;
// For fast-isel:
let isCodeGenOnly = 1 in
def RLDICL_32_64 : MDForm_1<30, 0,
                            (outs g8rc:$RA),
                            (ins gprc:$RS, u6imm:$SH, u6imm:$MBE),
                            "rldicl $RA, $RS, $SH, $MBE", IIC_IntRotateDI,
                            []>, isPPC64;
// End fast-isel.
```
- **EN**: Adds declarative TableGen records such as `RLDICL`, `RLDICL_32_64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `RLDICL`, `RLDICL_32_64`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1128-1137

```tablegen
let Interpretation64Bit = 1, isCodeGenOnly = 1 in
defm RLDICL_32 : MDForm_1r<30, 0,
                           (outs gprc:$RA),
                           (ins gprc:$RS, u6imm:$SH, u6imm:$MBE),
                           "rldicl", "$RA, $RS, $SH, $MBE", IIC_IntRotateDI,
                           []>, isPPC64;
defm RLDICR : MDForm_1r<30, 1,
                        (outs g8rc:$RA), (ins g8rc:$RS, u6imm:$SH, u6imm:$MBE),
                        "rldicr", "$RA, $RS, $SH, $MBE", IIC_IntRotateDI,
                        []>, isPPC64;
```
- **EN**: Adds declarative TableGen records such as `RLDICL_32`, `RLDICR` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `RLDICL_32`, `RLDICR`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1138-1147

```tablegen
let isCodeGenOnly = 1 in
def RLDICR_32 : MDForm_1<30, 1,
                         (outs gprc:$RA), (ins gprc:$RS, u6imm:$SH, u6imm:$MBE),
                         "rldicr $RA, $RS, $SH, $MBE", IIC_IntRotateDI,
                         []>, isPPC64;
defm RLDIC  : MDForm_1r<30, 2,
                        (outs g8rc:$RA), (ins g8rc:$RS, u6imm:$SH, u6imm:$MBE),
                        "rldic", "$RA, $RS, $SH, $MBE", IIC_IntRotateDI,
                        []>, isPPC64;
```
- **EN**: Adds declarative TableGen records such as `RLDICR_32`, `RLDIC` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `RLDICR_32`, `RLDIC`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1148-1159

```tablegen
let Interpretation64Bit = 1, isCodeGenOnly = 1 in {
defm RLWINM8 : MForm_2r<21, (outs g8rc:$RA),
                        (ins g8rc:$RS, u5imm:$SH, u5imm:$MB, u5imm:$ME),
                        "rlwinm", "$RA, $RS, $SH, $MB, $ME", IIC_IntGeneral,
                        []>;

defm RLWNM8  : MForm_1r<23, (outs g8rc:$RA),
                        (ins g8rc:$RS, g8rc:$RB, u5imm:$MB, u5imm:$ME),
                        "rlwnm", "$RA, $RS, $RB, $MB, $ME", IIC_IntGeneral,
                        []>;

// RLWIMI can be commuted if the rotate amount is zero.
```
- **EN**: Adds declarative TableGen records such as `RLWINM8`, `RLWNM8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `RLWINM8`, `RLWNM8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1160-1166

```tablegen
let Interpretation64Bit = 1, isCodeGenOnly = 1 in
defm RLWIMI8 : MForm_2r<20, (outs g8rc:$RA),
                        (ins g8rc:$RAi, g8rc:$RS, u5imm:$SH, u5imm:$MB,
                        u5imm:$ME), "rlwimi", "$RA, $RS, $SH, $MB, $ME",
                        IIC_IntRotate, []>, PPC970_DGroup_Cracked,
                        RegConstraint<"$RAi = $RA">;
```
- **EN**: Adds declarative TableGen records such as `RLWIMI8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `RLWIMI8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1167-1175

```tablegen
let isSelect = 1 in
def ISEL8   : AForm_4<31, 15,
                     (outs g8rc:$RT), (ins g8rc_nox0:$RA, g8rc:$RB, crbitrc:$COND),
                     "isel $RT, $RA, $RB, $COND", IIC_IntISEL,
                     []>;
}  // Interpretation64Bit
}  // hasSideEffects = 0
}  // End FXU Operations.
```
- **EN**: Adds declarative TableGen records such as `ISEL8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ISEL8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1176-1182

```tablegen
def : InstAlias<"li $rD, $imm", (ADDI8 g8rc:$rD, ZERO8, s16imm64:$imm)>;
def : InstAlias<"lis $rD, $imm", (ADDIS8 g8rc:$rD, ZERO8, s17imm64:$imm)>;

def : InstAlias<"mr $rA, $rB", (OR8 g8rc:$rA, g8rc:$rB, g8rc:$rB)>;
def : InstAlias<"mr. $rA, $rB", (OR8_rec g8rc:$rA, g8rc:$rB, g8rc:$rB)>;

def : InstAlias<"not $rA, $rB", (NOR8 g8rc:$rA, g8rc:$rB, g8rc:$rB)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1183-1189

```tablegen
def : InstAlias<"not. $rA, $rB", (NOR8_rec g8rc:$rA, g8rc:$rB, g8rc:$rB)>;

def : InstAlias<"mtcr $rA", (MTCRF8 255, g8rc:$rA)>;

def : InstAlias<"sub $rA, $rB, $rC", (SUBF8 g8rc:$rA, g8rc:$rC, g8rc:$rB)>;
def : InstAlias<"sub. $rA, $rB, $rC", (SUBF8_rec g8rc:$rA, g8rc:$rC, g8rc:$rB)>;
def : InstAlias<"subc $rA, $rB, $rC", (SUBFC8 g8rc:$rA, g8rc:$rC, g8rc:$rB)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1190-1196

```tablegen
def : InstAlias<"subc. $rA, $rB, $rC", (SUBFC8_rec g8rc:$rA, g8rc:$rC, g8rc:$rB)>;

def : InstAlias<"rotlwi $rA, $rS, $n", (RLWINM8 g8rc:$rA, g8rc:$rS, u5imm:$n, 0, 31)>;
def : InstAlias<"rotlwi. $rA, $rS, $n", (RLWINM8_rec g8rc:$rA, g8rc:$rS, u5imm:$n, 0, 31)>;
def : InstAlias<"rotlw $rA, $rS, $rB", (RLWNM8 g8rc:$rA, g8rc:$rS, g8rc:$rB, 0, 31)>;
def : InstAlias<"rotlw. $rA, $rS, $rB", (RLWNM8_rec g8rc:$rA, g8rc:$rS, g8rc:$rB, 0, 31)>;
def : InstAlias<"clrlwi $rA, $rS, $n", (RLWINM8 g8rc:$rA, g8rc:$rS, 0, u5imm:$n, 31)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1197-1205

```tablegen
def : InstAlias<"clrlwi. $rA, $rS, $n", (RLWINM8_rec g8rc:$rA, g8rc:$rS, 0, u5imm:$n, 31)>;

def : InstAlias<"isellt $rT, $rA, $rB",
                (ISEL8 g8rc:$rT, g8rc_nox0:$rA, g8rc:$rB, CR0LT)>;
def : InstAlias<"iselgt $rT, $rA, $rB",
                (ISEL8 g8rc:$rT, g8rc_nox0:$rA, g8rc:$rB, CR0GT)>;
def : InstAlias<"iseleq $rT, $rA, $rB",
                (ISEL8 g8rc:$rT, g8rc_nox0:$rA, g8rc:$rB, CR0EQ)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1206-1212

```tablegen
def : InstAlias<"nop", (ORI8 X0, X0, 0)>;
def : InstAlias<"xnop", (XORI8 X0, X0, 0)>;

def : InstAlias<"cntlzw $rA, $rS", (CNTLZW8 g8rc:$rA, g8rc:$rS)>;
def : InstAlias<"cntlzw. $rA, $rS", (CNTLZW8_rec g8rc:$rA, g8rc:$rS)>;

def : InstAlias<"mtxer $Rx", (MTSPR8 1, g8rc:$Rx)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1213-1220

```tablegen
def : InstAlias<"mfxer $Rx", (MFSPR8 g8rc:$Rx, 1)>;

//Disable this alias on AIX for now because as does not support them.
let Predicates = [ModernAs] in {

def : InstAlias<"mtudscr $Rx", (MTSPR8 3, g8rc:$Rx)>;
def : InstAlias<"mfudscr $Rx", (MFSPR8 g8rc:$Rx, 3)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1221-1227

```tablegen
def : InstAlias<"mfrtcu $Rx", (MFSPR8 g8rc:$Rx, 4)>;
def : InstAlias<"mfrtcl $Rx", (MFSPR8 g8rc:$Rx, 5)>;

def : InstAlias<"mtlr $Rx", (MTSPR8 8, g8rc:$Rx)>;
def : InstAlias<"mflr $Rx", (MFSPR8 g8rc:$Rx, 8)>;

def : InstAlias<"mtctr $Rx", (MTSPR8 9, g8rc:$Rx)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1228-1235

```tablegen
def : InstAlias<"mfctr $Rx", (MFSPR8 g8rc:$Rx, 9)>;

def : InstAlias<"mtuamr $Rx", (MTSPR8 13, g8rc:$Rx)>;
def : InstAlias<"mfuamr $Rx", (MFSPR8 g8rc:$Rx, 13)>;

def : InstAlias<"mtdscr $Rx", (MTSPR8 17, g8rc:$Rx)>;
def : InstAlias<"mfdscr $Rx", (MFSPR8 g8rc:$Rx, 17)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1236-1242

```tablegen
def : InstAlias<"mtdsisr $Rx", (MTSPR8 18, g8rc:$Rx)>;
def : InstAlias<"mfdsisr $Rx", (MFSPR8 g8rc:$Rx, 18)>;

def : InstAlias<"mtdar $Rx", (MTSPR8 19, g8rc:$Rx)>;
def : InstAlias<"mfdar $Rx", (MFSPR8 g8rc:$Rx, 19)>;

def : InstAlias<"mtdec $Rx", (MTSPR8 22, g8rc:$Rx)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1243-1250

```tablegen
def : InstAlias<"mfdec $Rx", (MFSPR8 g8rc:$Rx, 22)>;

def : InstAlias<"mtsdr1 $Rx", (MTSPR8 25, g8rc:$Rx)>;
def : InstAlias<"mfsdr1 $Rx", (MFSPR8 g8rc:$Rx, 25)>;

def : InstAlias<"mtsrr0 $Rx", (MTSPR8 26, g8rc:$Rx)>;
def : InstAlias<"mfsrr0 $Rx", (MFSPR8 g8rc:$Rx, 26)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1251-1257

```tablegen
def : InstAlias<"mtsrr1 $Rx", (MTSPR8 27, g8rc:$Rx)>;
def : InstAlias<"mfsrr1 $Rx", (MFSPR8 g8rc:$Rx, 27)>;

def : InstAlias<"mtcfar $Rx", (MTSPR8 28, g8rc:$Rx)>;
def : InstAlias<"mfcfar $Rx", (MFSPR8 g8rc:$Rx, 28)>;

def : InstAlias<"mtamr $Rx", (MTSPR8 29, g8rc:$Rx)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1258-1266

```tablegen
def : InstAlias<"mfamr $Rx", (MFSPR8 g8rc:$Rx, 29)>;

foreach SPRG = 0-3 in {
  def : InstAlias<"mfsprg $RT, "#SPRG, (MFSPR8 g8rc:$RT, !add(SPRG, 272))>;
  def : InstAlias<"mfsprg"#SPRG#" $RT", (MFSPR8 g8rc:$RT, !add(SPRG, 272))>;
  def : InstAlias<"mtsprg "#SPRG#", $RT", (MTSPR8 !add(SPRG, 272), g8rc:$RT)>;
  def : InstAlias<"mtsprg"#SPRG#" $RT", (MTSPR8 !add(SPRG, 272), g8rc:$RT)>;
}
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1267-1274

```tablegen
def : InstAlias<"mfasr $RT", (MFSPR8 g8rc:$RT, 280)>;
def : InstAlias<"mtasr $RT", (MTSPR8 280, g8rc:$RT)>;

def : InstAlias<"mttbl $Rx", (MTSPR8 284, g8rc:$Rx)>;
def : InstAlias<"mttbu $Rx", (MTSPR8 285, g8rc:$Rx)>;

def : InstAlias<"mfpvr $RT", (MFSPR8 g8rc:$RT, 287)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1275-1279

```tablegen
def : InstAlias<"mfspefscr $Rx", (MFSPR8 g8rc:$Rx, 512)>;
def : InstAlias<"mtspefscr $Rx", (MTSPR8 512, g8rc:$Rx)>;

}
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1280-1286

```tablegen
//===----------------------------------------------------------------------===//
// Load/Store instructions.
//


// Sign extending loads.
let PPC970_Unit = 2 in {
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Load/Store instructions.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Load/Store instructions.”。

### Lines 1287-1296

```tablegen
let Interpretation64Bit = 1, isCodeGenOnly = 1 in
def LHA8: DForm_1<42, (outs g8rc:$RST), (ins (memri $D, $RA):$addr),
                  "lha $RST, $addr", IIC_LdStLHA,
                  [(set i64:$RST, (sextloadi16 DForm:$addr))]>,
                  PPC970_DGroup_Cracked, SExt32To64;
def LWA  : DSForm_1<58, 2, (outs g8rc:$RST), (ins (memrix $D, $RA):$addr),
                    "lwa $RST, $addr", IIC_LdStLWA,
                    [(set i64:$RST,
                          (sextloadi32 DSForm:$addr))]>, isPPC64,
                    PPC970_DGroup_Cracked, SExt32To64;
```
- **EN**: Adds declarative TableGen records such as `LHA8`, `LWA` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LHA8`, `LWA`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1297-1306

```tablegen
let Interpretation64Bit = 1, isCodeGenOnly = 1 in
def LHAX8: XForm_1_memOp<31, 343, (outs g8rc:$RST), (ins (memrr $RA, $RB):$addr),
                        "lhax $RST, $addr", IIC_LdStLHA,
                        [(set i64:$RST, (sextloadi16 XForm:$addr))]>,
                        PPC970_DGroup_Cracked, SExt32To64;
def LWAX : XForm_1_memOp<31, 341, (outs g8rc:$RST), (ins (memrr $RA, $RB):$addr),
                        "lwax $RST, $addr", IIC_LdStLHA,
                        [(set i64:$RST, (sextloadi32 XForm:$addr))]>, isPPC64,
                        PPC970_DGroup_Cracked, SExt32To64;
// For fast-isel:
```
- **EN**: Adds declarative TableGen records such as `LHAX8`, `LWAX` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LHAX8`, `LWAX`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1307-1316

```tablegen
let isCodeGenOnly = 1, mayLoad = 1, hasSideEffects = 0 in {
def LWA_32  : DSForm_1<58, 2, (outs gprc:$RST), (ins (memrix $D, $RA):$addr),
                      "lwa $RST, $addr", IIC_LdStLWA, []>, isPPC64,
                      PPC970_DGroup_Cracked, SExt32To64;
def LWAX_32 : XForm_1_memOp<31, 341, (outs gprc:$RST), (ins (memrr $RA, $RB):$addr),
                            "lwax $RST, $addr", IIC_LdStLHA, []>, isPPC64,
                            PPC970_DGroup_Cracked, SExt32To64;
} // end fast-isel isCodeGenOnly

// Update forms.
```
- **EN**: Adds declarative TableGen records such as `LWA_32`, `LWAX_32` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LWA_32`, `LWAX_32`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1317-1324

```tablegen
let mayLoad = 1, hasSideEffects = 0 in {
let Interpretation64Bit = 1, isCodeGenOnly = 1 in
def LHAU8 : DForm_1<43, (outs g8rc:$RST, ptr_rc_nor0:$ea_result),
                    (ins (memri $D, $RA):$addr),
                    "lhau $RST, $addr", IIC_LdStLHAU,
                    []>, RegConstraint<"$addr.reg = $ea_result">;
// NO LWAU!
```
- **EN**: Adds declarative TableGen records such as `LHAU8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LHAU8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1325-1336

```tablegen
let Interpretation64Bit = 1, isCodeGenOnly = 1 in
def LHAUX8 : XForm_1_memOp<31, 375, (outs g8rc:$RST, ptr_rc_nor0:$ea_result),
                          (ins (memrr $RA, $RB):$addr),
                          "lhaux $RST, $addr", IIC_LdStLHAUX,
                          []>, RegConstraint<"$addr.ptrreg = $ea_result">;
def LWAUX : XForm_1_memOp<31, 373, (outs g8rc:$RST, ptr_rc_nor0:$ea_result),
                          (ins (memrr $RA, $RB):$addr),
                          "lwaux $RST, $addr", IIC_LdStLHAUX,
                          []>, RegConstraint<"$addr.ptrreg = $ea_result">, isPPC64;
}
}
```
- **EN**: Adds declarative TableGen records such as `LHAUX8`, `LWAUX` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LHAUX8`, `LWAUX`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1337-1343

```tablegen
let Interpretation64Bit = 1, isCodeGenOnly = 1 in {
// Zero extending loads.
let PPC970_Unit = 2 in {
def LBZ8 : DForm_1<34, (outs g8rc:$RST), (ins (memri $D, $RA):$addr),
                  "lbz $RST, $addr", IIC_LdStLoad,
                  [(set i64:$RST, (zextloadi8 DForm:$addr))]>, ZExt32To64,
                  SExt32To64;
```
- **EN**: Adds declarative TableGen records such as `LBZ8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LBZ8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1344-1352

```tablegen
def LHZ8 : DForm_1<40, (outs g8rc:$RST), (ins (memri $D, $RA):$addr),
                  "lhz $RST, $addr", IIC_LdStLoad,
                  [(set i64:$RST, (zextloadi16 DForm:$addr))]>, ZExt32To64,
                  SExt32To64;
def LWZ8 : DForm_1<32, (outs g8rc:$RST), (ins (memri $D, $RA):$addr),
                  "lwz $RST, $addr", IIC_LdStLoad,
                  [(set i64:$RST, (zextloadi32 DForm:$addr))]>, isPPC64,
                  ZExt32To64;
```
- **EN**: Adds declarative TableGen records such as `LHZ8`, `LWZ8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LHZ8`, `LWZ8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1353-1360

```tablegen
def LBZX8 : XForm_1_memOp<31,  87, (outs g8rc:$RST), (ins (memrr $RA, $RB):$addr),
                          "lbzx $RST, $addr", IIC_LdStLoad,
                          [(set i64:$RST, (zextloadi8 XForm:$addr))]>, ZExt32To64,
                          SExt32To64;
def LHZX8 : XForm_1_memOp<31, 279, (outs g8rc:$RST), (ins (memrr $RA, $RB):$addr),
                          "lhzx $RST, $addr", IIC_LdStLoad,
                          [(set i64:$RST, (zextloadi16 XForm:$addr))]>,
                          ZExt32To64, SExt32To64;
```
- **EN**: Adds declarative TableGen records such as `LBZX8`, `LHZX8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LBZX8`, `LHZX8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1361-1367

```tablegen
def LWZX8 : XForm_1_memOp<31,  23, (outs g8rc:$RST), (ins (memrr $RA, $RB):$addr),
                          "lwzx $RST, $addr", IIC_LdStLoad,
                          [(set i64:$RST, (zextloadi32 XForm:$addr))]>,
                          ZExt32To64;


// Update forms.
```
- **EN**: Adds declarative TableGen records such as `LWZX8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LWZX8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1368-1376

```tablegen
let mayLoad = 1, hasSideEffects = 0 in {
def LBZU8 : DForm_1<35, (outs g8rc:$RST, ptr_rc_nor0:$ea_result),
                    (ins (memri $D, $RA):$addr),
                    "lbzu $RST, $addr", IIC_LdStLoadUpd,
                    []>, RegConstraint<"$addr.reg = $ea_result">;
def LHZU8 : DForm_1<41, (outs g8rc:$RST, ptr_rc_nor0:$ea_result),
                    (ins (memri $D, $RA):$addr),
                    "lhzu $RST, $addr", IIC_LdStLoadUpd,
                    []>, RegConstraint<"$addr.reg = $ea_result">;
```
- **EN**: Adds declarative TableGen records such as `LBZU8`, `LHZU8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LBZU8`, `LHZU8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1377-1385

```tablegen
def LWZU8 : DForm_1<33, (outs g8rc:$RST, ptr_rc_nor0:$ea_result),
                    (ins (memri $D, $RA):$addr),
                    "lwzu $RST, $addr", IIC_LdStLoadUpd,
                    []>, RegConstraint<"$addr.reg = $ea_result">;

def LBZUX8 : XForm_1_memOp<31, 119, (outs g8rc:$RST, ptr_rc_nor0:$ea_result),
                          (ins (memrr $RA, $RB):$addr),
                          "lbzux $RST, $addr", IIC_LdStLoadUpdX,
                          []>, RegConstraint<"$addr.ptrreg = $ea_result">;
```
- **EN**: Adds declarative TableGen records such as `LWZU8`, `LBZUX8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LWZU8`, `LBZUX8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1386-1399

```tablegen
def LHZUX8 : XForm_1_memOp<31, 311, (outs g8rc:$RST, ptr_rc_nor0:$ea_result),
                          (ins (memrr $RA, $RB):$addr),
                          "lhzux $RST, $addr", IIC_LdStLoadUpdX,
                          []>, RegConstraint<"$addr.ptrreg = $ea_result">;
def LWZUX8 : XForm_1_memOp<31, 55, (outs g8rc:$RST, ptr_rc_nor0:$ea_result),
                          (ins (memrr $RA, $RB):$addr),
                          "lwzux $RST, $addr", IIC_LdStLoadUpdX,
                          []>, RegConstraint<"$addr.ptrreg = $ea_result">;
}
}
} // Interpretation64Bit


// Full 8-byte loads.
```
- **EN**: Adds declarative TableGen records such as `LHZUX8`, `LWZUX8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LHZUX8`, `LWZUX8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1400-1406

```tablegen
let PPC970_Unit = 2 in {
def LD   : DSForm_1<58, 0, (outs g8rc:$RST), (ins (memrix $D, $RA):$addr),
                    "ld $RST, $addr", IIC_LdStLD,
                    [(set i64:$RST, (load DSForm:$addr))]>, isPPC64;
// The following four definitions are selected for small code model only.
// Otherwise, we need to create two instructions to form a 32-bit offset,
// so we have a custom matcher for TOC_ENTRY in PPCDAGToDAGIsel::Select().
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "The following four definitions are selected for small code model only.". Notable symbols in this range include `LD`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“The following four definitions are selected for small code model only.”。 该区间中较显眼的符号包括 `LD`。

### Lines 1407-1414

```tablegen
def LDtoc: PPCEmitTimePseudo<(outs g8rc:$rD), (ins tocentry:$disp, g8rc:$reg),
                  "#LDtoc",
                  [(set i64:$rD,
                     (PPCtoc_entry tglobaladdr:$disp, i64:$reg))]>, isPPC64;
def LDtocJTI: PPCEmitTimePseudo<(outs g8rc:$rD), (ins tocentry:$disp, g8rc:$reg),
                  "#LDtocJTI",
                  [(set i64:$rD,
                     (PPCtoc_entry tjumptable:$disp, i64:$reg))]>, isPPC64;
```
- **EN**: Adds declarative TableGen records such as `LDtoc`, `LDtocJTI` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LDtoc`, `LDtocJTI`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1415-1423

```tablegen
def LDtocCPT: PPCEmitTimePseudo<(outs g8rc:$rD), (ins tocentry:$disp, g8rc:$reg),
                  "#LDtocCPT",
                  [(set i64:$rD,
                     (PPCtoc_entry tconstpool:$disp, i64:$reg))]>, isPPC64;
def LDtocBA: PPCEmitTimePseudo<(outs g8rc:$rD), (ins tocentry:$disp, g8rc:$reg),
                  "#LDtocCPT",
                  [(set i64:$rD,
                     (PPCtoc_entry tblockaddress:$disp, i64:$reg))]>, isPPC64;
```
- **EN**: Adds declarative TableGen records such as `LDtocCPT`, `LDtocBA` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LDtocCPT`, `LDtocBA`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1424-1433

```tablegen
def LDX  : XForm_1_memOp<31,  21, (outs g8rc:$RST), (ins (memrr $RA, $RB):$addr),
                        "ldx $RST, $addr", IIC_LdStLD,
                        [(set i64:$RST, (load XForm:$addr))]>, isPPC64;

let Predicates = [IsISA2_06] in {
def LDBRX : XForm_1_memOp<31,  532, (outs g8rc:$RST), (ins (memrr $RA, $RB):$addr),
                          "ldbrx $RST, $addr", IIC_LdStLoad,
                          [(set i64:$RST, (PPClbrx ForceXForm:$addr, i64))]>, isPPC64;
}
```
- **EN**: Adds declarative TableGen records such as `LDX`, `LDBRX` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LDX`, `LDBRX`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1434-1440

```tablegen
let mayLoad = 1, hasSideEffects = 0, isCodeGenOnly = 1 in {
def LHBRX8 : XForm_1_memOp<31, 790, (outs g8rc:$RST), (ins (memrr $RA, $RB):$addr),
                          "lhbrx $RST, $addr", IIC_LdStLoad, []>, ZExt32To64;
def LWBRX8 : XForm_1_memOp<31,  534, (outs g8rc:$RST), (ins (memrr $RA, $RB):$addr),
                          "lwbrx $RST, $addr", IIC_LdStLoad, []>, ZExt32To64;
}
```
- **EN**: Adds declarative TableGen records such as `LHBRX8`, `LWBRX8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LHBRX8`, `LWBRX8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1441-1452

```tablegen
let mayLoad = 1, hasSideEffects = 0 in {
def LDU  : DSForm_1<58, 1, (outs g8rc:$RST, ptr_rc_nor0:$ea_result),
                    (ins (memrix $D, $RA):$addr),
                    "ldu $RST, $addr", IIC_LdStLDU,
                    []>, RegConstraint<"$addr.reg = $ea_result">, isPPC64;

def LDUX : XForm_1_memOp<31, 53, (outs g8rc:$RST, ptr_rc_nor0:$ea_result),
                        (ins (memrr $RA, $RB):$addr),
                        "ldux $RST, $addr", IIC_LdStLDUX,
                        []>, RegConstraint<"$addr.ptrreg = $ea_result">, isPPC64;
}
```
- **EN**: Adds declarative TableGen records such as `LDU`, `LDUX` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LDU`, `LDUX`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1453-1467

```tablegen
let mayLoad = 1, hasNoSchedulingInfo = 1 in {
// Full 16-byte load.
// Early clobber $RTp to avoid assigned to the same register as RA.
// TODO: Add scheduling info.
def LQ   : DQForm_RTp5_RA17_MEM<56, 0,
                                (outs g8prc:$RTp),
                                (ins (memrix16 $DQ, $RA):$addr),
                                "lq $RTp, $addr", IIC_LdStLQ,
                                []>,
                                RegConstraint<"@earlyclobber $RTp">,
                                isPPC64;
// We don't really have LQX in the ISA, make a pseudo one so that we can
// handle x-form during isel. Make it pre-ra may expose
// oppotunities to some opts(CSE, LICM and etc.) for the result of adding
// RA and RB.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Full 16-byte load.". Scheduling or processor-model metadata is part of this section.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Full 16-byte load.”。 这一段包含调度或处理器模型元数据。

### Lines 1468-1476

```tablegen
def LQX_PSEUDO : PPCCustomInserterPseudo<(outs g8prc:$RTp),
                                         (ins memrr:$src), "#LQX_PSEUDO", []>;

def RESTORE_QUADWORD : PPCEmitTimePseudo<(outs g8prc:$RTp), (ins memrix:$src),
                                         "#RESTORE_QUADWORD", []>;
}

}
```
- **EN**: Adds declarative TableGen records such as `LQX_PSEUDO`, `RESTORE_QUADWORD` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LQX_PSEUDO`, `RESTORE_QUADWORD`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1477-1483

```tablegen
def : Pat<(int_ppc_atomic_load_i128 iaddrX16:$src),
          (SPLIT_QUADWORD (LQ memrix16:$src))>;

def : Pat<(int_ppc_atomic_load_i128 ForceXForm:$src),
          (SPLIT_QUADWORD (LQX_PSEUDO memrr:$src))>;

// Support for medium and large code model.
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1484-1492

```tablegen
let hasSideEffects = 0 in {
let isReMaterializable = 1 in {
def ADDIStocHA8: PPCEmitTimePseudo<(outs g8rc:$rD), (ins g8rc_nox0:$reg, tocentry:$disp),
                       "#ADDIStocHA8", []>, isPPC64;
def ADDItocL8: PPCEmitTimePseudo<(outs g8rc:$rD), (ins g8rc_nox0:$reg, tocentry:$disp),
                     "#ADDItocL8", []>, isPPC64;
}

// Local Data Transform
```
- **EN**: Adds declarative TableGen records such as `ADDIStocHA8`, `ADDItocL8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ADDIStocHA8`, `ADDItocL8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1493-1501

```tablegen
def ADDItoc8 : PPCEmitTimePseudo<(outs g8rc:$rD), (ins g8rc_nox0:$reg, tocentry:$disp),
                   "#ADDItoc8",
                   []>, isPPC64;
let mayLoad = 1 in
def LDtocL: PPCEmitTimePseudo<(outs g8rc:$rD), (ins tocentry:$disp, g8rc_nox0:$reg),
                   "#LDtocL", []>, isPPC64;
}

// Support for thread-local storage.
```
- **EN**: Adds declarative TableGen records such as `ADDItoc8`, `LDtocL` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ADDItoc8`, `LDtocL`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1502-1513

```tablegen
def ADDISgotTprelHA: PPCEmitTimePseudo<(outs g8rc:$rD), (ins g8rc_nox0:$reg, s16imm64:$disp),
                         "#ADDISgotTprelHA",
                         [(set i64:$rD,
                           (PPCaddisGotTprelHA i64:$reg,
                                               tglobaltlsaddr:$disp))]>,
                  isPPC64;
def LDgotTprelL: PPCEmitTimePseudo<(outs g8rc_nox0:$rD), (ins s16imm64:$disp, g8rc_nox0:$reg),
                        "#LDgotTprelL",
                        [(set i64:$rD,
                          (PPCldGotTprelL tglobaltlsaddr:$disp, i64:$reg))]>,
                 isPPC64;
```
- **EN**: Adds declarative TableGen records such as `ADDISgotTprelHA`, `LDgotTprelL` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ADDISgotTprelHA`, `LDgotTprelL`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1514-1523

```tablegen
let Defs = [CR7], Itinerary = IIC_LdStSync in
def CFENCE8 : PPCPostRAExpPseudo<(outs), (ins g8rc:$cr), "#CFENCE8", []>;

def : Pat<(PPCaddTls i64:$in, tglobaltlsaddr:$g),
          (ADD8TLS $in, tglobaltlsaddr:$g)>;
def ADDIStlsgdHA: PPCEmitTimePseudo<(outs g8rc:$rD), (ins g8rc_nox0:$reg, s16imm64:$disp),
                         "#ADDIStlsgdHA",
                         [(set i64:$rD,
                           (PPCaddisTlsgdHA i64:$reg, tglobaltlsaddr:$disp))]>,
                  isPPC64;
```
- **EN**: Adds declarative TableGen records such as `CFENCE8`, `ADDIStlsgdHA` that LLVM later expands into generated tables or helper code. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CFENCE8`, `ADDIStlsgdHA`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段包含调度或处理器模型元数据。

### Lines 1524-1534

```tablegen
def ADDItlsgdL : PPCEmitTimePseudo<(outs g8rc:$rD), (ins g8rc_nox0:$reg, s16imm64:$disp),
                       "#ADDItlsgdL",
                       [(set i64:$rD,
                         (PPCaddiTlsgdL i64:$reg, tglobaltlsaddr:$disp))]>,
                 isPPC64;

class GETtlsADDRPseudo <string asmstr> : PPCEmitTimePseudo<(outs g8rc:$rD), (ins g8rc:$reg, tlsgd:$sym),
                                             asmstr,
                                             [(set i64:$rD,
                                               (PPCgetTlsAddr i64:$reg, tglobaltlsaddr:$sym))]>,
                                      isPPC64;
```
- **EN**: Declares a backend-facing type `ADDItlsgdL`, `GETtlsADDRPseudo` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `ADDItlsgdL`, `GETtlsADDRPseudo`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1535-1545

```tablegen
class GETtlsldADDRPseudo <string asmstr> : PPCEmitTimePseudo<(outs g8rc:$rD), (ins g8rc:$reg, tlsgd:$sym),
                                             asmstr,
                                             [(set i64:$rD,
                                               (PPCgetTlsldAddr i64:$reg, tglobaltlsaddr:$sym))]>,
                                      isPPC64;

let hasExtraSrcRegAllocReq = 1, hasExtraDefRegAllocReq = 1 in {
// LR8 is a true define, while the rest of the Defs are clobbers. X3 is
// explicitly defined when this op is created, so not mentioned here.
// This is lowered to BL8_NOP_TLS by the assembly printer, so the size must be
// correct because the branch select pass is relying on it.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "LR8 is a true define, while the rest of the Defs are clobbers. X3 is". Notable symbols in this range include `GETtlsldADDRPseudo`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“LR8 is a true define, while the rest of the Defs are clobbers. X3 is”。 该区间中较显眼的符号包括 `GETtlsldADDRPseudo`。

### Lines 1546-1552

```tablegen
let Defs = [X0,X4,X5,X6,X7,X8,X9,X10,X11,X12,LR8,CTR8,CR0,CR1,CR5,CR6,CR7], Size = 8 in
def GETtlsADDR : GETtlsADDRPseudo <"#GETtlsADDR">;
let Defs = [X0,X2,X4,X5,X6,X7,X8,X9,X10,X11,X12,LR8,CTR8,CR0,CR1,CR5,CR6,CR7] in
def GETtlsADDRPCREL : GETtlsADDRPseudo <"#GETtlsADDRPCREL">;

// LR8 is a true define, while the rest of the Defs are clobbers. X3 is
// explicitly defined when this op is created, so not mentioned here.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "LR8 is a true define, while the rest of the Defs are clobbers. X3 is". Notable symbols in this range include `GETtlsADDR`, `GETtlsADDRPCREL`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“LR8 is a true define, while the rest of the Defs are clobbers. X3 is”。 该区间中较显眼的符号包括 `GETtlsADDR`, `GETtlsADDRPCREL`。

### Lines 1553-1562

```tablegen
let Defs = [X0,X4,X5,X6,X7,X8,X9,X10,X11,X12,LR8,CTR8,CR0,CR1,CR5,CR6,CR7], Size = 8 in
def GETtlsldADDR : GETtlsldADDRPseudo <"#GETtlsldADDR">;
let Defs = [X0,X2,X4,X5,X6,X7,X8,X9,X10,X11,X12,LR8,CTR8,CR0,CR1,CR5,CR6,CR7] in
def GETtlsldADDRPCREL : GETtlsldADDRPseudo <"#GETtlsldADDRPCREL">;

// On AIX, the call to __tls_get_addr needs two inputs in X3/X4 for the
// offset and region handle respectively. The call is not followed by a nop
// so we don't need to mark it with a size of 8 bytes. Finally, the assembly
// manual mentions this exact set of registers as the clobbered set, others
// are guaranteed not to be clobbered.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "On AIX, the call to __tls_get_addr needs two inputs in X3/X4 for the". Notable symbols in this range include `GETtlsldADDR`, `GETtlsldADDRPCREL`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“On AIX, the call to __tls_get_addr needs two inputs in X3/X4 for the”。 该区间中较显眼的符号包括 `GETtlsldADDR`, `GETtlsldADDRPCREL`。

### Lines 1563-1569

```tablegen
let Defs = [X0,X4,X5,X11,LR8,CR0] in {
def GETtlsADDR64AIX :
  PPCEmitTimePseudo<(outs g8rc:$rD),(ins g8rc:$offset, g8rc:$handle),
                    "GETtlsADDR64AIX",
                    [(set i64:$rD,
                      (PPCgetTlsAddr i64:$offset, i64:$handle))]>, isPPC64;
// On AIX, the call to .__tls_get_mod needs one input in X3 for the module handle.
```
- **EN**: Adds declarative TableGen records such as `GETtlsADDR64AIX` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `GETtlsADDR64AIX`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1570-1579

```tablegen
def GETtlsMOD64AIX :
  PPCEmitTimePseudo<(outs g8rc:$rD),(ins g8rc:$handle),
                    "GETtlsMOD64AIX",
                    [(set i64:$rD,
                      (PPCgetTlsMod i64:$handle))]>, isPPC64;
}
}

// Combined op for ADDItlsgdL and GETtlsADDR, late expanded.  X3 and LR8
// are true defines while the rest of the Defs are clobbers.
```
- **EN**: Adds declarative TableGen records such as `GETtlsMOD64AIX` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `GETtlsMOD64AIX`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1580-1590

```tablegen
let hasExtraSrcRegAllocReq = 1, hasExtraDefRegAllocReq = 1,
    Defs = [X0,X3,X4,X5,X6,X7,X8,X9,X10,X11,X12,LR8,CTR8,CR0,CR1,CR5,CR6,CR7]
    in
def ADDItlsgdLADDR : PPCEmitTimePseudo<(outs g8rc:$rD),
                            (ins g8rc_nox0:$reg, s16imm64:$disp, tlsgd:$sym),
                            "#ADDItlsgdLADDR",
                            [(set i64:$rD,
                              (PPCaddiTlsgdLAddr i64:$reg,
                                                 tglobaltlsaddr:$disp,
                                                 tglobaltlsaddr:$sym))]>,
                     isPPC64;
```
- **EN**: Adds declarative TableGen records such as `ADDItlsgdLADDR` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ADDItlsgdLADDR`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1591-1602

```tablegen
def ADDIStlsldHA: PPCEmitTimePseudo<(outs g8rc:$rD), (ins g8rc_nox0:$reg, s16imm64:$disp),
                         "#ADDIStlsldHA",
                         [(set i64:$rD,
                           (PPCaddisTlsldHA i64:$reg, tglobaltlsaddr:$disp))]>,
                  isPPC64;
def ADDItlsldL : PPCEmitTimePseudo<(outs g8rc:$rD), (ins g8rc_nox0:$reg, s16imm64:$disp),
                       "#ADDItlsldL",
                       [(set i64:$rD,
                         (PPCaddiTlsldL i64:$reg, tglobaltlsaddr:$disp))]>,
                 isPPC64;
// This pseudo is expanded to two copies to put the variable offset in R4 and
// the region handle in R3 and GETtlsADDR64AIX.
```
- **EN**: Adds declarative TableGen records such as `ADDIStlsldHA`, `ADDItlsldL` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ADDIStlsldHA`, `ADDItlsldL`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1603-1612

```tablegen
def TLSGDAIX8 :
  PPCEmitTimePseudo<(outs g8rc:$rD), (ins g8rc:$offset, g8rc:$handle),
                     "#TLSGDAIX8",
                     [(set i64:$rD,
                       (PPCTlsgdAIX i64:$offset, i64:$handle))]>;
// This pseudo is expanded to the call to GETtlsMOD64AIX.
def TLSLDAIX8 : PPCEmitTimePseudo<(outs g8rc:$rD), (ins g8rc:$handle),
                     "#TLSLDAIX8", [(set i64:$rD, (PPCTlsldAIX i64:$handle))]>;
// Combined op for ADDItlsldL and GETtlsADDR, late expanded.  X3 and LR8
// are true defines, while the rest of the Defs are clobbers.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This pseudo is expanded to the call to GETtlsMOD64AIX.". Notable symbols in this range include `TLSGDAIX8`, `TLSLDAIX8`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This pseudo is expanded to the call to GETtlsMOD64AIX.”。 该区间中较显眼的符号包括 `TLSGDAIX8`, `TLSLDAIX8`。

### Lines 1613-1623

```tablegen
let hasExtraSrcRegAllocReq = 1, hasExtraDefRegAllocReq = 1,
    Defs = [X0,X3,X4,X5,X6,X7,X8,X9,X10,X11,X12,LR8,CTR8,CR0,CR1,CR5,CR6,CR7]
    in
def ADDItlsldLADDR : PPCEmitTimePseudo<(outs g8rc:$rD),
                            (ins g8rc_nox0:$reg, s16imm64:$disp, tlsgd:$sym),
                            "#ADDItlsldLADDR",
                            [(set i64:$rD,
                              (PPCaddiTlsldLAddr i64:$reg,
                                                 tglobaltlsaddr:$disp,
                                                 tglobaltlsaddr:$sym))]>,
                     isPPC64;
```
- **EN**: Adds declarative TableGen records such as `ADDItlsldLADDR` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ADDItlsldLADDR`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1624-1634

```tablegen
def ADDISdtprelHA: PPCEmitTimePseudo<(outs g8rc:$rD), (ins g8rc_nox0:$reg, s16imm64:$disp),
                          "#ADDISdtprelHA",
                          [(set i64:$rD,
                            (PPCaddisDtprelHA i64:$reg,
                                              tglobaltlsaddr:$disp))]>,
                   isPPC64;
def ADDIdtprelL : PPCEmitTimePseudo<(outs g8rc:$rD), (ins g8rc_nox0:$reg, s16imm64:$disp),
                         "#ADDIdtprelL",
                         [(set i64:$rD,
                           (PPCaddiDtprelL i64:$reg, tglobaltlsaddr:$disp))]>,
                  isPPC64;
```
- **EN**: Adds declarative TableGen records such as `ADDISdtprelHA`, `ADDIdtprelL` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ADDISdtprelHA`, `ADDIdtprelL`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1635-1641

```tablegen
let Size = 8 in
def PADDIdtprel : PPCEmitTimePseudo<(outs g8rc:$rD), (ins g8rc_nox0:$reg, s16imm64:$disp),
                          "#PADDIdtprel",
                          [(set i64:$rD,
                            (PPCpaddiDtprel i64:$reg, tglobaltlsaddr:$disp))]>,
                  isPPC64;
```
- **EN**: Adds declarative TableGen records such as `PADDIdtprel` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PADDIdtprel`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1642-1650

```tablegen
let PPC970_Unit = 2 in {
let Interpretation64Bit = 1, isCodeGenOnly = 1 in {
// Truncating stores.
def STB8 : DForm_1<38, (outs), (ins g8rc:$RST, (memri $D, $RA):$addr),
                   "stb $RST, $addr", IIC_LdStStore,
                   [(truncstorei8 i64:$RST, DForm:$addr)]>;
def STH8 : DForm_1<44, (outs), (ins g8rc:$RST, (memri $D, $RA):$addr),
                   "sth $RST, $addr", IIC_LdStStore,
                   [(truncstorei16 i64:$RST, DForm:$addr)]>;
```
- **EN**: Adds declarative TableGen records such as `STB8`, `STH8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STB8`, `STH8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1651-1657

```tablegen
def STW8 : DForm_1<36, (outs), (ins g8rc:$RST, (memri $D, $RA):$addr),
                   "stw $RST, $addr", IIC_LdStStore,
                   [(truncstorei32 i64:$RST, DForm:$addr)]>;
def STBX8 : XForm_8_memOp<31, 215, (outs), (ins g8rc:$RST, (memrr $RA, $RB):$addr),
                          "stbx $RST, $addr", IIC_LdStStore,
                          [(truncstorei8 i64:$RST, XForm:$addr)]>,
                          PPC970_DGroup_Cracked;
```
- **EN**: Adds declarative TableGen records such as `STW8`, `STBX8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STW8`, `STBX8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1658-1668

```tablegen
def STHX8 : XForm_8_memOp<31, 407, (outs), (ins g8rc:$RST, (memrr $RA, $RB):$addr),
                          "sthx $RST, $addr", IIC_LdStStore,
                          [(truncstorei16 i64:$RST, XForm:$addr)]>,
                          PPC970_DGroup_Cracked;
def STWX8 : XForm_8_memOp<31, 151, (outs), (ins g8rc:$RST, (memrr $RA, $RB):$addr),
                          "stwx $RST, $addr", IIC_LdStStore,
                          [(truncstorei32 i64:$RST, XForm:$addr)]>,
                          PPC970_DGroup_Cracked;
} // Interpretation64Bit

// Normal 8-byte stores.
```
- **EN**: Adds declarative TableGen records such as `STHX8`, `STWX8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STHX8`, `STWX8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1669-1676

```tablegen
def STD  : DSForm_1<62, 0, (outs), (ins g8rc:$RST, (memrix $D, $RA):$addr),
                    "std $RST, $addr", IIC_LdStSTD,
                    [(store i64:$RST, DSForm:$addr)]>, isPPC64;
def STDX  : XForm_8_memOp<31, 149, (outs), (ins g8rc:$RST, (memrr $RA, $RB):$addr),
                          "stdx $RST, $addr", IIC_LdStSTD,
                          [(store i64:$RST, XForm:$addr)]>, isPPC64,
                          PPC970_DGroup_Cracked;
```
- **EN**: Adds declarative TableGen records such as `STD`, `STDX` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STD`, `STDX`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1677-1683

```tablegen
let Predicates = [IsISA2_06] in {
def STDBRX: XForm_8_memOp<31, 660, (outs), (ins g8rc:$RST, (memrr $RA, $RB):$addr),
                          "stdbrx $RST, $addr", IIC_LdStStore,
                          [(PPCstbrx i64:$RST, ForceXForm:$addr, i64)]>, isPPC64,
                          PPC970_DGroup_Cracked;
}
```
- **EN**: Adds declarative TableGen records such as `STDBRX` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STDBRX`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1684-1690

```tablegen
let mayStore = 1, hasNoSchedulingInfo = 1 in {
// Normal 16-byte stores.
// TODO: Add scheduling info.
def STQ : DSForm_1<62, 2, (outs), (ins g8prc:$RST, (memrix $D, $RA):$addr),
                   "stq $RST, $addr", IIC_LdStSTQ,
                   []>, isPPC64;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Normal 16-byte stores.". Scheduling or processor-model metadata is part of this section.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Normal 16-byte stores.”。 这一段包含调度或处理器模型元数据。

### Lines 1691-1700

```tablegen
def STQX_PSEUDO : PPCCustomInserterPseudo<(outs),
                                          (ins g8prc:$RSp, memrr:$dst),
                                          "#STQX_PSEUDO", []>;

def SPILL_QUADWORD : PPCEmitTimePseudo<(outs), (ins g8prc:$RSp, memrix:$dst),
                                       "#SPILL_QUADWORD", []>;
}

}
```
- **EN**: Adds declarative TableGen records such as `STQX_PSEUDO`, `SPILL_QUADWORD` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STQX_PSEUDO`, `SPILL_QUADWORD`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1701-1708

```tablegen
def BUILD_QUADWORD : PPCPostRAExpPseudo<
                       (outs g8prc:$RTp),
                       (ins g8rc:$lo, g8rc:$hi),
                       "#BUILD_QUADWORD", []>;

def : Pat<(int_ppc_atomic_store_i128 i64:$lo, i64:$hi, DSForm:$dst),
          (STQ (BUILD_QUADWORD g8rc:$lo, g8rc:$hi), memrix:$dst)>;
```
- **EN**: Adds declarative TableGen records such as `BUILD_QUADWORD` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `BUILD_QUADWORD`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1709-1717

```tablegen
def : Pat<(int_ppc_atomic_store_i128 i64:$lo, i64:$hi, ForceXForm:$dst),
          (STQX_PSEUDO (BUILD_QUADWORD g8rc:$lo, g8rc:$hi), memrr:$dst)>;

// Stores with Update (pre-inc).
let PPC970_Unit = 2, mayStore = 1, mayLoad = 0 in {
let Interpretation64Bit = 1, isCodeGenOnly = 1 in {
def STBU8 : DForm_1<39, (outs ptr_rc_nor0:$ea_res), (ins g8rc:$RST, (memri $D, $RA):$addr),
                   "stbu $RST, $addr", IIC_LdStSTU, []>,
                   RegConstraint<"$addr.reg = $ea_res">;
```
- **EN**: Adds declarative TableGen records such as `STBU8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STBU8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1718-1724

```tablegen
def STHU8 : DForm_1<45, (outs ptr_rc_nor0:$ea_res), (ins g8rc:$RST, (memri $D, $RA):$addr),
                   "sthu $RST, $addr", IIC_LdStSTU, []>,
                   RegConstraint<"$addr.reg = $ea_res">;
def STWU8 : DForm_1<37, (outs ptr_rc_nor0:$ea_res), (ins g8rc:$RST, (memri $D, $RA):$addr),
                   "stwu $RST, $addr", IIC_LdStSTU, []>,
                   RegConstraint<"$addr.reg = $ea_res">;
```
- **EN**: Adds declarative TableGen records such as `STHU8`, `STWU8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STHU8`, `STWU8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1725-1734

```tablegen
def STBUX8: XForm_8_memOp<31, 247, (outs ptr_rc_nor0:$ea_res),
                          (ins g8rc:$RST, (memrr $RA, $RB):$addr),
                          "stbux $RST, $addr", IIC_LdStSTUX, []>,
                          RegConstraint<"$addr.ptrreg = $ea_res">,
                          PPC970_DGroup_Cracked;
def STHUX8: XForm_8_memOp<31, 439, (outs ptr_rc_nor0:$ea_res),
                          (ins g8rc:$RST, (memrr $RA, $RB):$addr),
                          "sthux $RST, $addr", IIC_LdStSTUX, []>,
                          RegConstraint<"$addr.ptrreg = $ea_res">,
                          PPC970_DGroup_Cracked;
```
- **EN**: Adds declarative TableGen records such as `STBUX8`, `STHUX8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STBUX8`, `STHUX8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1735-1741

```tablegen
def STWUX8: XForm_8_memOp<31, 183, (outs ptr_rc_nor0:$ea_res),
                          (ins g8rc:$RST, (memrr $RA, $RB):$addr),
                          "stwux $RST, $addr", IIC_LdStSTUX, []>,
                          RegConstraint<"$addr.ptrreg = $ea_res">,
                          PPC970_DGroup_Cracked;
} // Interpretation64Bit
```
- **EN**: Adds declarative TableGen records such as `STWUX8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STWUX8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1742-1757

```tablegen
def STDU : DSForm_1<62, 1, (outs ptr_rc_nor0:$ea_res),
                   (ins g8rc:$RST, (memrix $D, $RA):$addr),
                   "stdu $RST, $addr", IIC_LdStSTU, []>,
                   RegConstraint<"$addr.reg = $ea_res">,
                   isPPC64;

def STDUX : XForm_8_memOp<31, 181, (outs ptr_rc_nor0:$ea_res),
                          (ins g8rc:$RST, (memrr $RA, $RB):$addr),
                          "stdux $RST, $addr", IIC_LdStSTUX, []>,
                          RegConstraint<"$addr.ptrreg = $ea_res">,
                          PPC970_DGroup_Cracked, isPPC64;
}

// Patterns to match the pre-inc stores.  We can't put the patterns on
// the instruction definitions directly as ISel wants the address base
// and offset to be separate operands, not a single complex operand.
```
- **EN**: Adds declarative TableGen records such as `STDU`, `STDUX` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STDU`, `STDUX`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1758-1766

```tablegen
def : Pat<(pre_truncsti8 i64:$rS, iPTR:$ptrreg, iaddroff:$ptroff),
          (STBU8 $rS, iaddroff:$ptroff, $ptrreg)>;
def : Pat<(pre_truncsti16 i64:$rS, iPTR:$ptrreg, iaddroff:$ptroff),
          (STHU8 $rS, iaddroff:$ptroff, $ptrreg)>;
def : Pat<(pre_truncsti32 i64:$rS, iPTR:$ptrreg, iaddroff:$ptroff),
          (STWU8 $rS, iaddroff:$ptroff, $ptrreg)>;
def : Pat<(DSFormPreStore i64:$rS, iPTR:$ptrreg, iaddroff:$ptroff),
          (STDU $rS, iaddroff:$ptroff, $ptrreg)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1767-1776

```tablegen
def : Pat<(pre_truncsti8 i64:$rS, iPTR:$ptrreg, iPTR:$ptroff),
          (STBUX8 $rS, $ptrreg, $ptroff)>;
def : Pat<(pre_truncsti16 i64:$rS, iPTR:$ptrreg, iPTR:$ptroff),
          (STHUX8 $rS, $ptrreg, $ptroff)>;
def : Pat<(pre_truncsti32 i64:$rS, iPTR:$ptrreg, iPTR:$ptroff),
          (STWUX8 $rS, $ptrreg, $ptroff)>;
def : Pat<(pre_store i64:$rS, iPTR:$ptrreg, iPTR:$ptroff),
          (STDUX $rS, $ptrreg, $ptroff)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1777-1783

```tablegen
//===----------------------------------------------------------------------===//
// Floating point instructions.
//


let PPC970_Unit = 3, hasSideEffects = 0, mayRaiseFPException = 1,
    Uses = [RM] in {  // FPU Operations.
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1784-1792

```tablegen
defm FCFID  : XForm_26r<63, 846, (outs f8rc:$RST), (ins f8rc:$RB),
                        "fcfid", "$RST, $RB", IIC_FPGeneral,
                        [(set f64:$RST, (PPCany_fcfid f64:$RB))]>, isPPC64;
defm FCTID  : XForm_26r<63, 814, (outs f8rc:$RST), (ins f8rc:$RB),
                        "fctid", "$RST, $RB", IIC_FPGeneral,
                        []>, isPPC64;
defm FCTIDU : XForm_26r<63, 942, (outs f8rc:$RST), (ins f8rc:$RB),
                        "fctidu", "$RST, $RB", IIC_FPGeneral,
                        []>, isPPC64;
```
- **EN**: Adds declarative TableGen records such as `FCFID`, `FCTID`, `FCTIDU` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `FCFID`, `FCTID`, `FCTIDU`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1793-1799

```tablegen
defm FCTIDZ : XForm_26r<63, 815, (outs f8rc:$RST), (ins f8rc:$RB),
                        "fctidz", "$RST, $RB", IIC_FPGeneral,
                        [(set f64:$RST, (PPCany_fctidz f64:$RB))]>, isPPC64;

defm FCFIDU  : XForm_26r<63, 974, (outs f8rc:$RST), (ins f8rc:$RB),
                        "fcfidu", "$RST, $RB", IIC_FPGeneral,
                        [(set f64:$RST, (PPCany_fcfidu f64:$RB))]>, isPPC64;
```
- **EN**: Adds declarative TableGen records such as `FCTIDZ`, `FCFIDU` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `FCTIDZ`, `FCFIDU`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1800-1808

```tablegen
defm FCFIDS  : XForm_26r<59, 846, (outs f4rc:$RST), (ins f8rc:$RB),
                        "fcfids", "$RST, $RB", IIC_FPGeneral,
                        [(set f32:$RST, (PPCany_fcfids f64:$RB))]>, isPPC64;
defm FCFIDUS : XForm_26r<59, 974, (outs f4rc:$RST), (ins f8rc:$RB),
                        "fcfidus", "$RST, $RB", IIC_FPGeneral,
                        [(set f32:$RST, (PPCany_fcfidus f64:$RB))]>, isPPC64;
defm FCTIDUZ : XForm_26r<63, 943, (outs f8rc:$RST), (ins f8rc:$RB),
                        "fctiduz", "$RST, $RB", IIC_FPGeneral,
                        [(set f64:$RST, (PPCany_fctiduz f64:$RB))]>, isPPC64;
```
- **EN**: Adds declarative TableGen records such as `FCFIDS`, `FCFIDUS`, `FCTIDUZ` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `FCFIDS`, `FCFIDUS`, `FCTIDUZ`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1809-1815

```tablegen
defm FCTIWUZ : XForm_26r<63, 143, (outs f8rc:$RST), (ins f8rc:$RB),
                        "fctiwuz", "$RST, $RB", IIC_FPGeneral,
                        [(set f64:$RST, (PPCany_fctiwuz f64:$RB))]>, isPPC64;
}

// These instructions store a hash computed from the value of the link register
// and the value of the stack pointer.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "These instructions store a hash computed from the value of the link register". Notable symbols in this range include `FCTIWUZ`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“These instructions store a hash computed from the value of the link register”。 该区间中较显眼的符号包括 `FCTIWUZ`。

### Lines 1816-1828

```tablegen
let mayStore = 1, Interpretation64Bit = 1, isCodeGenOnly = 1 in {
def HASHST8 : XForm_XD6_RA5_RB5<31, 722, (outs),
                                (ins g8rc:$RB, (memrihash $D, $RA):$addr),
                                "hashst $RB, $addr", IIC_IntGeneral, []>;
def HASHSTP8 : XForm_XD6_RA5_RB5<31, 658, (outs),
                                 (ins g8rc:$RB, (memrihash $D, $RA):$addr),
                                 "hashstp $RB, $addr", IIC_IntGeneral, []>;
}

// These instructions check a hash computed from the value of the link register
// and the value of the stack pointer. The hasSideEffects flag is needed as the
// instruction may TRAP if the hash does not match the hash stored at the
// specified address.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "These instructions check a hash computed from the value of the link register". Notable symbols in this range include `HASHST8`, `HASHSTP8`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“These instructions check a hash computed from the value of the link register”。 该区间中较显眼的符号包括 `HASHST8`, `HASHSTP8`。

### Lines 1829-1838

```tablegen
let mayLoad = 1, hasSideEffects = 1,
    Interpretation64Bit = 1, isCodeGenOnly = 1 in {
def HASHCHK8 : XForm_XD6_RA5_RB5<31, 754, (outs),
                                 (ins g8rc:$RB, (memrihash $D, $RA):$addr),
                                 "hashchk $RB, $addr", IIC_IntGeneral, []>;
def HASHCHKP8 : XForm_XD6_RA5_RB5<31, 690, (outs),
                                  (ins g8rc:$RB, (memrihash $D, $RA):$addr),
                                  "hashchkp $RB, $addr", IIC_IntGeneral, []>;
}
```
- **EN**: Adds declarative TableGen records such as `HASHCHK8`, `HASHCHKP8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `HASHCHK8`, `HASHCHKP8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1839-1845

```tablegen
let Interpretation64Bit = 1, isCodeGenOnly = 1, hasSideEffects = 1 in
def ADDEX8 : Z23Form_RTAB5_CY2<31, 170, (outs g8rc:$RT),
                              (ins g8rc:$RA, g8rc:$RB, u2imm:$CY),
                              "addex $RT, $RA, $RB, $CY", IIC_IntGeneral,
                              [(set i64:$RT, (int_ppc_addex i64:$RA, i64:$RB,
                                                            u2imm_timm:$CY))]>;
```
- **EN**: Adds declarative TableGen records such as `ADDEX8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ADDEX8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1846-1853

```tablegen
//===----------------------------------------------------------------------===//
// Instruction Patterns
//

// Extensions and truncates to/from 32-bit regs.
def : Pat<(i64 (zext i32:$in)),
          (RLDICL (INSERT_SUBREG (i64 (IMPLICIT_DEF)), $in, sub_32),
                  0, 32)>;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Instruction Patterns". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Instruction Patterns”。 这些声明会进入生成式模式匹配逻辑。

### Lines 1854-1861

```tablegen
def : Pat<(i64 (anyext i32:$in)),
          (INSERT_SUBREG (i64 (IMPLICIT_DEF)), $in, sub_32)>;
def : Pat<(i32 (trunc i64:$in)),
          (EXTRACT_SUBREG $in, sub_32)>;

// Implement the 'not' operation with the NOR instruction.
// (we could use the default xori pattern, but nor has lower latency on some
// cores (such as the A2)).
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Implement the 'not' operation with the NOR instruction.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Implement the 'not' operation with the NOR instruction.”。

### Lines 1862-1869

```tablegen
def i64not : OutPatFrag<(ops node:$in),
                        (NOR8 $in, $in)>;
def        : Pat<(not i64:$in),
                 (i64not $in)>;

// Extending loads with i64 targets.
def : Pat<(zextloadi1 DForm:$src),
          (LBZ8 DForm:$src)>;
```
- **EN**: Adds declarative TableGen records such as `i64not` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `i64not`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1870-1877

```tablegen
def : Pat<(zextloadi1 XForm:$src),
          (LBZX8 XForm:$src)>;
def : Pat<(extloadi1 DForm:$src),
          (LBZ8 DForm:$src)>;
def : Pat<(extloadi1 XForm:$src),
          (LBZX8 XForm:$src)>;
def : Pat<(extloadi8 DForm:$src),
          (LBZ8 DForm:$src)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1878-1885

```tablegen
def : Pat<(extloadi8 XForm:$src),
          (LBZX8 XForm:$src)>;
def : Pat<(extloadi16 DForm:$src),
          (LHZ8 DForm:$src)>;
def : Pat<(extloadi16 XForm:$src),
          (LHZX8 XForm:$src)>;
def : Pat<(extloadi32 DForm:$src),
          (LWZ8 DForm:$src)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1886-1893

```tablegen
def : Pat<(extloadi32 XForm:$src),
          (LWZX8 XForm:$src)>;

// Standard shifts.  These are represented separately from the real shifts above
// so that we can distinguish between shifts that allow 6-bit and 7-bit shift
// amounts.
def : Pat<(sra i64:$rS, i32:$rB),
          (SRAD $rS, $rB)>;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Standard shifts.  These are represented separately from the real shifts above".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Standard shifts.  These are represented separately from the real shifts above”。

### Lines 1894-1903

```tablegen
def : Pat<(srl i64:$rS, i32:$rB),
          (SRD $rS, $rB)>;
def : Pat<(shl i64:$rS, i32:$rB),
          (SLD $rS, $rB)>;

// SUBFIC
def : Pat<(sub imm64SExt16:$imm, i64:$in),
          (SUBFIC8 $in, imm:$imm)>;

// SHL/SRL
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1904-1911

```tablegen
def : Pat<(shl i64:$in, (i32 imm:$imm)),
          (RLDICR $in, imm:$imm, (SHL64 imm:$imm))>;
def : Pat<(srl i64:$in, (i32 imm:$imm)),
          (RLDICL $in, (SRL64 imm:$imm), imm:$imm)>;

// ROTL
def : Pat<(rotl i64:$in, i32:$sh),
          (RLDCL $in, $sh, 0)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1912-1918

```tablegen
def : Pat<(rotl i64:$in, (i32 imm:$imm)),
          (RLDICL $in, imm:$imm, 0)>;

// Hi and Lo for Darwin Global Addresses.
def : Pat<(PPChi tglobaladdr:$in, 0), (LIS8 tglobaladdr:$in)>;
def : Pat<(PPClo tglobaladdr:$in, 0), (LI8  tglobaladdr:$in)>;
def : Pat<(PPChi tconstpool:$in , 0), (LIS8 tconstpool:$in)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1919-1925

```tablegen
def : Pat<(PPClo tconstpool:$in , 0), (LI8  tconstpool:$in)>;
def : Pat<(PPChi tjumptable:$in , 0), (LIS8 tjumptable:$in)>;
def : Pat<(PPClo tjumptable:$in , 0), (LI8  tjumptable:$in)>;
def : Pat<(PPChi tblockaddress:$in, 0), (LIS8 tblockaddress:$in)>;
def : Pat<(PPClo tblockaddress:$in, 0), (LI8  tblockaddress:$in)>;
def : Pat<(PPChi tglobaltlsaddr:$g, i64:$in),
          (ADDIS8 $in, tglobaltlsaddr:$g)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1926-1933

```tablegen
def : Pat<(PPClo tglobaltlsaddr:$g, i64:$in),
          (ADDI8 $in, tglobaltlsaddr:$g)>;
def : Pat<(add i64:$in, (PPChi tglobaladdr:$g, 0)),
          (ADDIS8 $in, tglobaladdr:$g)>;
def : Pat<(add i64:$in, (PPChi tconstpool:$g, 0)),
          (ADDIS8 $in, tconstpool:$g)>;
def : Pat<(add i64:$in, (PPChi tjumptable:$g, 0)),
          (ADDIS8 $in, tjumptable:$g)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1934-1940

```tablegen
def : Pat<(add i64:$in, (PPChi tblockaddress:$g, 0)),
          (ADDIS8 $in, tblockaddress:$g)>;

// AIX 64-bit small code model TLS access.
// This is used for global dynamic accesses when loading the region handle and
// variable offset, and also for local-exec accesses to load the offset of a
// TLS variable from the TOC, prior to adding it to r13.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "AIX 64-bit small code model TLS access.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“AIX 64-bit small code model TLS access.”。

### Lines 1941-1948

```tablegen
def : Pat<(i64 (PPCtoc_entry tglobaltlsaddr:$disp, i64:$reg)),
          (i64 (LDtoc tglobaltlsaddr:$disp, i64:$reg))>;

// The following pattern matches 64-bit local- and initial-exec TLS accesses on AIX.
// PPCaddTls is used in local- and initial-exec accesses in order to:
//   - Get the address of a variable (adding the variable offset to the thread
//     pointer in r13).
//   - Create an opportunity to optimize the user of the loaded address.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "The following pattern matches 64-bit local- and initial-exec TLS accesses on AIX.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“The following pattern matches 64-bit local- and initial-exec TLS accesses on AIX.”。

### Lines 1949-1955

```tablegen
def : Pat<(PPCaddTls i64:$in, i64:$addr),
          (ADD8TLS $in, $addr)>;

// 64-bits atomic loads and stores
def : Pat<(atomic_load_nonext_64 DSForm:$src), (LD  memrix:$src)>;
def : Pat<(atomic_load_nonext_64 XForm:$src),  (LDX memrr:$src)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1956-1963

```tablegen
def : Pat<(atomic_store_64 i64:$val, DSForm:$ptr), (STD  g8rc:$val, memrix:$ptr)>;
def : Pat<(atomic_store_64 i64:$val, XForm:$ptr), (STDX g8rc:$val, memrr:$ptr)>;

let Predicates = [IsISA3_0, IsPPC64] in {
def : Pat<(i64 (int_ppc_cmpeqb g8rc:$a, g8rc:$b)),
          (i64 (SETB8 (CMPEQB $a, $b)))>;
def : Pat<(i64 (int_ppc_setb g8rc:$a, g8rc:$b)),
          (i64 (SETB8 (CMPD $a, $b)))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1964-1971

```tablegen
def : Pat<(i64 (int_ppc_maddhd g8rc:$a, g8rc:$b, g8rc:$c)),
          (i64 (MADDHD $a, $b, $c))>;
def : Pat<(i64 (int_ppc_maddhdu g8rc:$a, g8rc:$b, g8rc:$c)),
          (i64 (MADDHDU $a, $b, $c))>;
def : Pat<(i64 (int_ppc_maddld g8rc:$a, g8rc:$b, g8rc:$c)),
          (i64 (MADDLD8 $a, $b, $c))>;
}
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1972-1978

```tablegen
let Predicates = [IsPPC64] in {
def : Pat<(i64 (int_ppc_mulhd g8rc:$a, g8rc:$b)),
          (i64 (MULHD $a, $b))>;
def : Pat<(i64 (int_ppc_mulhdu g8rc:$a, g8rc:$b)),
          (i64 (MULHDU $a, $b))>;
def : Pat<(int_ppc_load8r ForceXForm:$ptr),
          (LDBRX ForceXForm:$ptr)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1979-1985

```tablegen
def : Pat<(int_ppc_store8r g8rc:$a, ForceXForm:$ptr),
          (STDBRX g8rc:$a, ForceXForm:$ptr)>;
}

def : Pat<(i64 (int_ppc_cmpb g8rc:$a, g8rc:$b)),
          (i64 (CMPB8 $a, $b))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1986-1992

```tablegen
let Predicates = [IsISA3_0] in {
// DARN (deliver random number)
// L=0 for 32-bit, L=1 for conditioned random, L=2 for raw random
def : Pat<(int_ppc_darn32), (EXTRACT_SUBREG (DARN 0), sub_32)>;
def : Pat<(int_ppc_darn), (DARN 1)>;
def : Pat<(int_ppc_darnraw), (DARN 2)>;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "DARN (deliver random number)". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“DARN (deliver random number)”。 这些声明会进入生成式模式匹配逻辑。

### Lines 1993-1999

```tablegen
class X_RA5_RB5<bits<6> opcode, bits<10> xo, string opc, RegisterOperand ty,
                   InstrItinClass itin, list<dag> pattern>
  : X_L1_RS5_RS5<opcode, xo, (outs), (ins ty:$RA, ty:$RB),
                 !strconcat(opc, " $RA, $RB"), itin, pattern>{
   let L = 1;
}
```
- **EN**: Declares a backend-facing type `X_RA5_RB5` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `X_RA5_RB5`，并勾勒出周边代码会依赖的接口或状态。

### Lines 2000-2006

```tablegen
class X_L1_RA5_RB5<bits<6> opcode, bits<10> xo, string opc, RegisterOperand ty,
                   InstrItinClass itin, list<dag> pattern>
  : X_L1_RS5_RS5<opcode, xo, (outs), (ins ty:$RA, ty:$RB, u1imm:$L),
                 !strconcat(opc, " $RA, $RB, $L"), itin, pattern>;

let Interpretation64Bit = 1, isCodeGenOnly = 1 in {
def CP_COPY8   : X_RA5_RB5<31, 774, "copy"  , g8rc, IIC_LdStCOPY, []>;
```
- **EN**: Declares a backend-facing type `X_L1_RA5_RB5`, `CP_COPY8` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `X_L1_RA5_RB5`, `CP_COPY8`，并勾勒出周边代码会依赖的接口或状态。

### Lines 2007-2013

```tablegen
def CP_PASTE8_rec : X_L1_RA5_RB5<31, 902, "paste.", g8rc, IIC_LdStPASTE, []>,isRecordForm;
}

// SLB Invalidate Entry Global
def SLBIEG : XForm_26<31, 466, (outs), (ins gprc:$RST, gprc:$RB),
                      "slbieg $RST, $RB", IIC_SprSLBIEG, []>;
// SLB Synchronize
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "SLB Invalidate Entry Global". Notable symbols in this range include `CP_PASTE8_rec`, `SLBIEG`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“SLB Invalidate Entry Global”。 该区间中较显眼的符号包括 `CP_PASTE8_rec`, `SLBIEG`。

### Lines 2014-2021

```tablegen
def SLBSYNC : XForm_0<31, 338, (outs), (ins), "slbsync", IIC_SprSLBSYNC, []>;

} // IsISA3_0

def : Pat<(int_ppc_ldarx ForceXForm:$ptr),
          (LDARX ForceXForm:$ptr)>;
def : Pat<(int_ppc_stdcx ForceXForm:$dst, g8rc:$A),
          (RLWINM (STDCX g8rc:$A, ForceXForm:$dst), 31, 31, 31)>;
```
- **EN**: Adds declarative TableGen records such as `SLBSYNC` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SLBSYNC`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2022-2028

```tablegen
def : Pat<(PPCStoreCond ForceXForm:$dst, g8rc:$A, 8),
          (RLWINM (STDCX g8rc:$A, ForceXForm:$dst), 31, 31, 31)>;

def : Pat<(i64 (int_ppc_mfspr timm:$SPR)),
          (MFSPR8 $SPR)>;
def : Pat<(int_ppc_mtspr timm:$SPR, g8rc:$RT),
          (MTSPR8 $SPR, $RT)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- TableGen declarations for the backend / 该后端的 TableGen 声明
- LLVM TableGen DSL / LLVM TableGen 领域特定语言
- SelectionDAG lowering / SelectionDAG lowering
- Assembly parsing / 汇编解析
- Declarative TableGen records / 声明式 TableGen 记录
- Atomic operation support / 原子操作支持
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- None explicitly included in the source text / 源码文本中没有显式包含项

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- Generated backend tables / 生成式后端表
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
