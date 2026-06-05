# VEAsmParser.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/VE/AsmParser/VEAsmParser.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements the target-specific assembly parser that turns textual assembly into MC operands and MC instructions.
  - **CN**: 实现目标相关的汇编解析器，把文本汇编转换为 MC 操作数和 MC 指令。

## Line-by-Line Analysis / 逐行分析

### Lines 1-31
```cpp
//===-- VEAsmParser.cpp - Parse VE assembly to MCInst instructions --------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "MCTargetDesc/VEMCAsmInfo.h"
#include "MCTargetDesc/VEMCTargetDesc.h"
#include "TargetInfo/VETargetInfo.h"
#include "VE.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/ADT/Twine.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCInstrInfo.h"
#include "llvm/MC/MCParser/AsmLexer.h"
#include "llvm/MC/MCParser/MCAsmParser.h"
#include "llvm/MC/MCParser/MCParsedAsmOperand.h"
#include "llvm/MC/MCParser/MCTargetAsmParser.h"
#include "llvm/MC/MCStreamer.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/MC/MCSymbol.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/Compiler.h"
#include "llvm/Support/raw_ostream.h"
#include <memory>

```
- **EN**: Pulls in the headers needed for this implementation, including `MCTargetDesc/VEMCAsmInfo.h`, `MCTargetDesc/VEMCTargetDesc.h`, `TargetInfo/VETargetInfo.h`, `VE.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `MCTargetDesc/VEMCAsmInfo.h`, `MCTargetDesc/VEMCTargetDesc.h`, `TargetInfo/VETargetInfo.h`, `VE.h`。

### Lines 32-48
```cpp
using namespace llvm;

#define DEBUG_TYPE "ve-asmparser"

namespace {

class VEOperand;

class VEAsmParser : public MCTargetAsmParser {
  MCAsmParser &Parser;

  /// @name Auto-generated Match Functions
  /// {

#define GET_ASSEMBLER_HEADER
#include "VEGenAsmMatcher.inc"

```
- **EN**: Pulls in the headers needed for this implementation, including `VEGenAsmMatcher.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `VEGenAsmMatcher.inc`。

### Lines 49-66
```cpp
  /// }

  // public interface of the MCTargetAsmParser.
  bool matchAndEmitInstruction(SMLoc IDLoc, unsigned &Opcode,
                               OperandVector &Operands, MCStreamer &Out,
                               uint64_t &ErrorInfo,
                               bool MatchingInlineAsm) override;
  bool parseRegister(MCRegister &Reg, SMLoc &StartLoc, SMLoc &EndLoc) override;
  MCRegister parseRegisterName(MCRegister (*matchFn)(StringRef));
  ParseStatus tryParseRegister(MCRegister &Reg, SMLoc &StartLoc,
                               SMLoc &EndLoc) override;
  bool parseInstruction(ParseInstructionInfo &Info, StringRef Name,
                        SMLoc NameLoc, OperandVector &Operands) override;
  ParseStatus parseDirective(AsmToken DirectiveID) override;

  unsigned validateTargetOperandClass(MCParsedAsmOperand &Op,
                                      unsigned Kind) override;

```
- **EN**: Implements logic around `matchAndEmitInstruction`, `parseRegister`, `parseRegisterName`, `tryParseRegister`, ....
- **CN**: 围绕 `matchAndEmitInstruction`, `parseRegister`, `parseRegisterName`, `tryParseRegister`, ... 实现具体逻辑。

### Lines 67-83
```cpp
  // Custom parse functions for VE specific operands.
  ParseStatus parseMEMOperand(OperandVector &Operands);
  ParseStatus parseMEMAsOperand(OperandVector &Operands);
  ParseStatus parseCCOpOperand(OperandVector &Operands);
  ParseStatus parseRDOpOperand(OperandVector &Operands);
  ParseStatus parseMImmOperand(OperandVector &Operands);
  ParseStatus parseOperand(OperandVector &Operands, StringRef Name);
  ParseStatus parseVEAsmOperand(std::unique_ptr<VEOperand> &Operand);

  // Helper function to parse expression with a symbol.
  const MCExpr *extractSpecifier(const MCExpr *E, VE::Specifier &Variant);
  bool parseExpression(const MCExpr *&EVal);

  // Split the mnemonic stripping conditional code and quantifiers
  StringRef splitMnemonic(StringRef Name, SMLoc NameLoc,
                          OperandVector *Operands);

```
- **EN**: Implements logic around `parseMEMOperand`, `parseMEMAsOperand`, `parseCCOpOperand`, `parseRDOpOperand`, ....
- **CN**: 围绕 `parseMEMOperand`, `parseMEMAsOperand`, `parseCCOpOperand`, `parseRDOpOperand`, ... 实现具体逻辑。

### Lines 84-108
```cpp
  bool parseLiteralValues(unsigned Size, SMLoc L);

public:
  VEAsmParser(const MCSubtargetInfo &sti, MCAsmParser &parser,
              const MCInstrInfo &MII)
      : MCTargetAsmParser(sti, MII), Parser(parser) {
    // Initialize the set of available features.
    setAvailableFeatures(ComputeAvailableFeatures(getSTI().getFeatureBits()));
  }
};

} // end anonymous namespace

static const MCPhysReg I32Regs[64] = {
    VE::SW0,  VE::SW1,  VE::SW2,  VE::SW3,  VE::SW4,  VE::SW5,  VE::SW6,
    VE::SW7,  VE::SW8,  VE::SW9,  VE::SW10, VE::SW11, VE::SW12, VE::SW13,
    VE::SW14, VE::SW15, VE::SW16, VE::SW17, VE::SW18, VE::SW19, VE::SW20,
    VE::SW21, VE::SW22, VE::SW23, VE::SW24, VE::SW25, VE::SW26, VE::SW27,
    VE::SW28, VE::SW29, VE::SW30, VE::SW31, VE::SW32, VE::SW33, VE::SW34,
    VE::SW35, VE::SW36, VE::SW37, VE::SW38, VE::SW39, VE::SW40, VE::SW41,
    VE::SW42, VE::SW43, VE::SW44, VE::SW45, VE::SW46, VE::SW47, VE::SW48,
    VE::SW49, VE::SW50, VE::SW51, VE::SW52, VE::SW53, VE::SW54, VE::SW55,
    VE::SW56, VE::SW57, VE::SW58, VE::SW59, VE::SW60, VE::SW61, VE::SW62,
    VE::SW63};

```
- **EN**: Implements logic around `parseLiteralValues`, `VEAsmParser`, `MCTargetAsmParser`, `setAvailableFeatures`; this block works at the MC layer.
- **CN**: 围绕 `parseLiteralValues`, `VEAsmParser`, `MCTargetAsmParser`, `setAvailableFeatures` 实现具体逻辑；这一段工作在 MC 层。

### Lines 109-126
```cpp
static const MCPhysReg F32Regs[64] = {
    VE::SF0,  VE::SF1,  VE::SF2,  VE::SF3,  VE::SF4,  VE::SF5,  VE::SF6,
    VE::SF7,  VE::SF8,  VE::SF9,  VE::SF10, VE::SF11, VE::SF12, VE::SF13,
    VE::SF14, VE::SF15, VE::SF16, VE::SF17, VE::SF18, VE::SF19, VE::SF20,
    VE::SF21, VE::SF22, VE::SF23, VE::SF24, VE::SF25, VE::SF26, VE::SF27,
    VE::SF28, VE::SF29, VE::SF30, VE::SF31, VE::SF32, VE::SF33, VE::SF34,
    VE::SF35, VE::SF36, VE::SF37, VE::SF38, VE::SF39, VE::SF40, VE::SF41,
    VE::SF42, VE::SF43, VE::SF44, VE::SF45, VE::SF46, VE::SF47, VE::SF48,
    VE::SF49, VE::SF50, VE::SF51, VE::SF52, VE::SF53, VE::SF54, VE::SF55,
    VE::SF56, VE::SF57, VE::SF58, VE::SF59, VE::SF60, VE::SF61, VE::SF62,
    VE::SF63};

static const MCPhysReg F128Regs[32] = {
    VE::Q0,  VE::Q1,  VE::Q2,  VE::Q3,  VE::Q4,  VE::Q5,  VE::Q6,  VE::Q7,
    VE::Q8,  VE::Q9,  VE::Q10, VE::Q11, VE::Q12, VE::Q13, VE::Q14, VE::Q15,
    VE::Q16, VE::Q17, VE::Q18, VE::Q19, VE::Q20, VE::Q21, VE::Q22, VE::Q23,
    VE::Q24, VE::Q25, VE::Q26, VE::Q27, VE::Q28, VE::Q29, VE::Q30, VE::Q31};

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 127-142
```cpp
static const MCPhysReg VM512Regs[8] = {VE::VMP0, VE::VMP1, VE::VMP2, VE::VMP3,
                                       VE::VMP4, VE::VMP5, VE::VMP6, VE::VMP7};

static const MCPhysReg MISCRegs[31] = {
    VE::USRCC,      VE::PSW,        VE::SAR,        VE::NoRegister,
    VE::NoRegister, VE::NoRegister, VE::NoRegister, VE::PMMR,
    VE::PMCR0,      VE::PMCR1,      VE::PMCR2,      VE::PMCR3,
    VE::NoRegister, VE::NoRegister, VE::NoRegister, VE::NoRegister,
    VE::PMC0,       VE::PMC1,       VE::PMC2,       VE::PMC3,
    VE::PMC4,       VE::PMC5,       VE::PMC6,       VE::PMC7,
    VE::PMC8,       VE::PMC9,       VE::PMC10,      VE::PMC11,
    VE::PMC12,      VE::PMC13,      VE::PMC14};

namespace {

/// VEOperand - Instances of this class represent a parsed VE machine
```
- **EN**: Introduces declarations for `represent`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `represent` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 143-163
```cpp
/// instruction.
class VEOperand : public MCParsedAsmOperand {
private:
  enum KindTy {
    k_Token,
    k_Register,
    k_Immediate,
    // SX-Aurora ASX form is disp(index, base).
    k_MemoryRegRegImm,  // base=reg, index=reg, disp=imm
    k_MemoryRegImmImm,  // base=reg, index=imm, disp=imm
    k_MemoryZeroRegImm, // base=0, index=reg, disp=imm
    k_MemoryZeroImmImm, // base=0, index=imm, disp=imm
    // SX-Aurora AS form is disp(base).
    k_MemoryRegImm,  // base=reg, disp=imm
    k_MemoryZeroImm, // base=0, disp=imm
    // Other special cases for Aurora VE
    k_CCOp,   // condition code
    k_RDOp,   // rounding mode
    k_MImmOp, // Special immediate value of sequential bit stream of 0 or 1.
  } Kind;

```
- **EN**: Introduces declarations for `VEOperand`, `KindTy`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `VEOperand`, `KindTy` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 164-185
```cpp
  SMLoc StartLoc, EndLoc;

  struct Token {
    const char *Data;
    unsigned Length;
  };

  struct RegOp {
    MCRegister Reg;
  };

  struct ImmOp {
    const MCExpr *Val;
  };

  struct MemOp {
    MCRegister Base;
    MCRegister IndexReg;
    const MCExpr *Index;
    const MCExpr *Offset;
  };

```
- **EN**: Introduces declarations for `Token`, `RegOp`, `ImmOp`, `MemOp`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `Token`, `RegOp`, `ImmOp`, `MemOp` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 186-208
```cpp
  struct CCOp {
    unsigned CCVal;
  };

  struct RDOp {
    unsigned RDVal;
  };

  struct MImmOp {
    const MCExpr *Val;
    bool M0Flag;
  };

  union {
    struct Token Tok;
    struct RegOp Reg;
    struct ImmOp Imm;
    struct MemOp Mem;
    struct CCOp CC;
    struct RDOp RD;
    struct MImmOp MImm;
  };

```
- **EN**: Introduces declarations for `CCOp`, `RDOp`, `MImmOp`, `Token`, ..., defining the data structures or interfaces used later in the file.
- **CN**: 引入 `CCOp`, `RDOp`, `MImmOp`, `Token`, ... 等声明，定义本文件后续使用的数据结构或接口。

### Lines 209-230
```cpp
public:
  VEOperand(KindTy K) : Kind(K) {}

  bool isToken() const override { return Kind == k_Token; }
  bool isReg() const override { return Kind == k_Register; }
  bool isImm() const override { return Kind == k_Immediate; }
  bool isMem() const override {
    return isMEMrri() || isMEMrii() || isMEMzri() || isMEMzii() || isMEMri() ||
           isMEMzi();
  }
  bool isMEMrri() const { return Kind == k_MemoryRegRegImm; }
  bool isMEMrii() const { return Kind == k_MemoryRegImmImm; }
  bool isMEMzri() const { return Kind == k_MemoryZeroRegImm; }
  bool isMEMzii() const { return Kind == k_MemoryZeroImmImm; }
  bool isMEMri() const { return Kind == k_MemoryRegImm; }
  bool isMEMzi() const { return Kind == k_MemoryZeroImm; }
  bool isCCOp() const { return Kind == k_CCOp; }
  bool isRDOp() const { return Kind == k_RDOp; }
  bool isZero() {
    if (!isImm())
      return false;

```
- **EN**: Implements logic around `VEOperand`, `isToken`, `isReg`, `isImm`, ...; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `VEOperand`, `isToken`, `isReg`, `isImm`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 231-252
```cpp
    // Constant case
    if (const auto *ConstExpr = dyn_cast<MCConstantExpr>(Imm.Val)) {
      int64_t Value = ConstExpr->getValue();
      return Value == 0;
    }
    return false;
  }
  bool isUImm0to2() {
    if (!isImm())
      return false;

    // Constant case
    if (const auto *ConstExpr = dyn_cast<MCConstantExpr>(Imm.Val)) {
      int64_t Value = ConstExpr->getValue();
      return Value >= 0 && Value < 3;
    }
    return false;
  }
  bool isUImm1() {
    if (!isImm())
      return false;

```
- **EN**: Implements logic around `getValue`, `isUImm0to2`, `isUImm1`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getValue`, `isUImm0to2`, `isUImm1` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 253-274
```cpp
    // Constant case
    if (const auto *ConstExpr = dyn_cast<MCConstantExpr>(Imm.Val)) {
      int64_t Value = ConstExpr->getValue();
      return isUInt<1>(Value);
    }
    return false;
  }
  bool isUImm2() {
    if (!isImm())
      return false;

    // Constant case
    if (const auto *ConstExpr = dyn_cast<MCConstantExpr>(Imm.Val)) {
      int64_t Value = ConstExpr->getValue();
      return isUInt<2>(Value);
    }
    return false;
  }
  bool isUImm3() {
    if (!isImm())
      return false;

```
- **EN**: Implements logic around `getValue`, `isUInt<1>`, `isUImm2`, `isUInt<2>`, ...; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getValue`, `isUInt<1>`, `isUImm2`, `isUInt<2>`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 275-296
```cpp
    // Constant case
    if (const auto *ConstExpr = dyn_cast<MCConstantExpr>(Imm.Val)) {
      int64_t Value = ConstExpr->getValue();
      return isUInt<3>(Value);
    }
    return false;
  }
  bool isUImm4() {
    if (!isImm())
      return false;

    // Constant case
    if (const auto *ConstExpr = dyn_cast<MCConstantExpr>(Imm.Val)) {
      int64_t Value = ConstExpr->getValue();
      return isUInt<4>(Value);
    }
    return false;
  }
  bool isUImm6() {
    if (!isImm())
      return false;

```
- **EN**: Implements logic around `getValue`, `isUInt<3>`, `isUImm4`, `isUInt<4>`, ...; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getValue`, `isUInt<3>`, `isUImm4`, `isUInt<4>`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 297-318
```cpp
    // Constant case
    if (const auto *ConstExpr = dyn_cast<MCConstantExpr>(Imm.Val)) {
      int64_t Value = ConstExpr->getValue();
      return isUInt<6>(Value);
    }
    return false;
  }
  bool isUImm7() {
    if (!isImm())
      return false;

    // Constant case
    if (const auto *ConstExpr = dyn_cast<MCConstantExpr>(Imm.Val)) {
      int64_t Value = ConstExpr->getValue();
      return isUInt<7>(Value);
    }
    return false;
  }
  bool isSImm7() {
    if (!isImm())
      return false;

```
- **EN**: Implements logic around `getValue`, `isUInt<6>`, `isUImm7`, `isUInt<7>`, ...; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getValue`, `isUInt<6>`, `isUImm7`, `isUInt<7>`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 319-337
```cpp
    // Constant case
    if (const auto *ConstExpr = dyn_cast<MCConstantExpr>(Imm.Val)) {
      int64_t Value = ConstExpr->getValue();
      return isInt<7>(Value);
    }
    return false;
  }
  bool isMImm() const {
    if (Kind != k_MImmOp)
      return false;

    // Constant case
    if (const auto *ConstExpr = dyn_cast<MCConstantExpr>(MImm.Val)) {
      int64_t Value = ConstExpr->getValue();
      return isUInt<6>(Value);
    }
    return false;
  }

```
- **EN**: Implements logic around `getValue`, `isInt<7>`, `isMImm`, `isUInt<6>`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getValue`, `isInt<7>`, `isMImm`, `isUInt<6>` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 338-359
```cpp
  StringRef getToken() const {
    assert(Kind == k_Token && "Invalid access!");
    return StringRef(Tok.Data, Tok.Length);
  }

  MCRegister getReg() const override {
    assert((Kind == k_Register) && "Invalid access!");
    return Reg.Reg;
  }

  const MCExpr *getImm() const {
    assert((Kind == k_Immediate) && "Invalid access!");
    return Imm.Val;
  }

  MCRegister getMemBase() const {
    assert((Kind == k_MemoryRegRegImm || Kind == k_MemoryRegImmImm ||
            Kind == k_MemoryRegImm) &&
           "Invalid access!");
    return Mem.Base;
  }

```
- **EN**: Implements logic around `getToken`, `assert`, `StringRef`, `getReg`, ...; this block returns target-specific results.
- **CN**: 围绕 `getToken`, `assert`, `StringRef`, `getReg`, ... 实现具体逻辑；这一段返回目标相关结果。

### Lines 360-379
```cpp
  MCRegister getMemIndexReg() const {
    assert((Kind == k_MemoryRegRegImm || Kind == k_MemoryZeroRegImm) &&
           "Invalid access!");
    return Mem.IndexReg;
  }

  const MCExpr *getMemIndex() const {
    assert((Kind == k_MemoryRegImmImm || Kind == k_MemoryZeroImmImm) &&
           "Invalid access!");
    return Mem.Index;
  }

  const MCExpr *getMemOffset() const {
    assert((Kind == k_MemoryRegRegImm || Kind == k_MemoryRegImmImm ||
            Kind == k_MemoryZeroImmImm || Kind == k_MemoryZeroRegImm ||
            Kind == k_MemoryRegImm || Kind == k_MemoryZeroImm) &&
           "Invalid access!");
    return Mem.Offset;
  }

```
- **EN**: Implements logic around `getMemIndexReg`, `assert`, `getMemIndex`, `getMemOffset`; this block returns target-specific results.
- **CN**: 围绕 `getMemIndexReg`, `assert`, `getMemIndex`, `getMemOffset` 实现具体逻辑；这一段返回目标相关结果。

### Lines 380-397
```cpp
  void setMemOffset(const MCExpr *off) {
    assert((Kind == k_MemoryRegRegImm || Kind == k_MemoryRegImmImm ||
            Kind == k_MemoryZeroImmImm || Kind == k_MemoryZeroRegImm ||
            Kind == k_MemoryRegImm || Kind == k_MemoryZeroImm) &&
           "Invalid access!");
    Mem.Offset = off;
  }

  unsigned getCCVal() const {
    assert((Kind == k_CCOp) && "Invalid access!");
    return CC.CCVal;
  }

  unsigned getRDVal() const {
    assert((Kind == k_RDOp) && "Invalid access!");
    return RD.RDVal;
  }

```
- **EN**: Implements logic around `setMemOffset`, `assert`, `getCCVal`, `getRDVal`; this block returns target-specific results.
- **CN**: 围绕 `setMemOffset`, `assert`, `getCCVal`, `getRDVal` 实现具体逻辑；这一段返回目标相关结果。

### Lines 398-429
```cpp
  const MCExpr *getMImmVal() const {
    assert((Kind == k_MImmOp) && "Invalid access!");
    return MImm.Val;
  }
  bool getM0Flag() const {
    assert((Kind == k_MImmOp) && "Invalid access!");
    return MImm.M0Flag;
  }

  /// getStartLoc - Get the location of the first token of this operand.
  SMLoc getStartLoc() const override { return StartLoc; }
  /// getEndLoc - Get the location of the last token of this operand.
  SMLoc getEndLoc() const override { return EndLoc; }

  void print(raw_ostream &OS, const MCAsmInfo &MAI) const override {
    switch (Kind) {
    case k_Token:
      OS << "Token: " << getToken() << "\n";
      break;
    case k_Register:
      OS << "Reg: #" << getReg().id() << "\n";
      break;
    case k_Immediate:
      OS << "Imm: " << getImm() << "\n";
      break;
    case k_MemoryRegRegImm:
      assert(getMemOffset() != nullptr);
      OS << "Mem: #" << getMemBase().id() << "+#" << getMemIndexReg().id()
         << "+";
      MAI.printExpr(OS, *getMemOffset());
      OS << "\n";
      break;
```
- **EN**: Implements logic around `getMImmVal`, `assert`, `getM0Flag`, `getStartLoc`, ...; this block uses `switch`-based dispatch; returns target-specific results.
- **CN**: 围绕 `getMImmVal`, `assert`, `getM0Flag`, `getStartLoc`, ... 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果。

### Lines 430-461
```cpp
    case k_MemoryRegImmImm:
      assert(getMemIndex() != nullptr && getMemOffset() != nullptr);
      OS << "Mem: #" << getMemBase().id() << "+";
      MAI.printExpr(OS, *getMemIndex());
      OS << "+";
      MAI.printExpr(OS, *getMemOffset());
      OS << "\n";
      break;
    case k_MemoryZeroRegImm:
      assert(getMemOffset() != nullptr);
      OS << "Mem: 0+#" << getMemIndexReg().id() << "+";
      MAI.printExpr(OS, *getMemOffset());
      OS << "\n";
      break;
    case k_MemoryZeroImmImm:
      assert(getMemIndex() != nullptr && getMemOffset() != nullptr);
      OS << "Mem: 0+";
      MAI.printExpr(OS, *getMemIndex());
      OS << "+";
      MAI.printExpr(OS, *getMemOffset());
      OS << "\n";
      break;
    case k_MemoryRegImm:
      assert(getMemOffset() != nullptr);
      OS << "Mem: #" << getMemBase().id() << "+";
      MAI.printExpr(OS, *getMemOffset());
      OS << "\n";
      break;
    case k_MemoryZeroImm:
      assert(getMemOffset() != nullptr);
      OS << "Mem: 0+";
      MAI.printExpr(OS, *getMemOffset());
```
- **EN**: Implements logic around `assert`, `getMemBase`, `printExpr`, `getMemIndexReg`.
- **CN**: 围绕 `assert`, `getMemBase`, `printExpr`, `getMemIndexReg` 实现具体逻辑。

### Lines 462-480
```cpp
      OS << "\n";
      break;
    case k_CCOp:
      OS << "CCOp: " << getCCVal() << "\n";
      break;
    case k_RDOp:
      OS << "RDOp: " << getRDVal() << "\n";
      break;
    case k_MImmOp:
      OS << "MImm: (" << getMImmVal() << (getM0Flag() ? ")0" : ")1") << "\n";
      break;
    }
  }

  void addRegOperands(MCInst &Inst, unsigned N) const {
    assert(N == 1 && "Invalid number of operands!");
    Inst.addOperand(MCOperand::createReg(getReg()));
  }

```
- **EN**: Implements logic around `getCCVal`, `getRDVal`, `MImm:`, `addRegOperands`, ...; this block works at the MC layer.
- **CN**: 围绕 `getCCVal`, `getRDVal`, `MImm:`, `addRegOperands`, ... 实现具体逻辑；这一段工作在 MC 层。

### Lines 481-498
```cpp
  void addImmOperands(MCInst &Inst, unsigned N) const {
    assert(N == 1 && "Invalid number of operands!");
    const MCExpr *Expr = getImm();
    addExpr(Inst, Expr);
  }

  void addZeroOperands(MCInst &Inst, unsigned N) const {
    addImmOperands(Inst, N);
  }

  void addUImm0to2Operands(MCInst &Inst, unsigned N) const {
    addImmOperands(Inst, N);
  }

  void addUImm1Operands(MCInst &Inst, unsigned N) const {
    addImmOperands(Inst, N);
  }

```
- **EN**: Implements logic around `addImmOperands`, `assert`, `getImm`, `addExpr`, ...; this block works at the MC layer.
- **CN**: 围绕 `addImmOperands`, `assert`, `getImm`, `addExpr`, ... 实现具体逻辑；这一段工作在 MC 层。

### Lines 499-514
```cpp
  void addUImm2Operands(MCInst &Inst, unsigned N) const {
    addImmOperands(Inst, N);
  }

  void addUImm3Operands(MCInst &Inst, unsigned N) const {
    addImmOperands(Inst, N);
  }

  void addUImm4Operands(MCInst &Inst, unsigned N) const {
    addImmOperands(Inst, N);
  }

  void addUImm6Operands(MCInst &Inst, unsigned N) const {
    addImmOperands(Inst, N);
  }

```
- **EN**: Implements logic around `addUImm2Operands`, `addImmOperands`, `addUImm3Operands`, `addUImm4Operands`, ...; this block works at the MC layer.
- **CN**: 围绕 `addUImm2Operands`, `addImmOperands`, `addUImm3Operands`, `addUImm4Operands`, ... 实现具体逻辑；这一段工作在 MC 层。

### Lines 515-532
```cpp
  void addUImm7Operands(MCInst &Inst, unsigned N) const {
    addImmOperands(Inst, N);
  }

  void addSImm7Operands(MCInst &Inst, unsigned N) const {
    addImmOperands(Inst, N);
  }

  void addExpr(MCInst &Inst, const MCExpr *Expr) const {
    // Add as immediate when possible.  Null MCExpr = 0.
    if (!Expr)
      Inst.addOperand(MCOperand::createImm(0));
    else if (const auto *CE = dyn_cast<MCConstantExpr>(Expr))
      Inst.addOperand(MCOperand::createImm(CE->getValue()));
    else
      Inst.addOperand(MCOperand::createExpr(Expr));
  }

```
- **EN**: Implements logic around `addUImm7Operands`, `addImmOperands`, `addSImm7Operands`, `addExpr`, ...; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `addUImm7Operands`, `addImmOperands`, `addSImm7Operands`, `addExpr`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 533-548
```cpp
  void addMEMrriOperands(MCInst &Inst, unsigned N) const {
    assert(N == 3 && "Invalid number of operands!");

    Inst.addOperand(MCOperand::createReg(getMemBase()));
    Inst.addOperand(MCOperand::createReg(getMemIndexReg()));
    addExpr(Inst, getMemOffset());
  }

  void addMEMriiOperands(MCInst &Inst, unsigned N) const {
    assert(N == 3 && "Invalid number of operands!");

    Inst.addOperand(MCOperand::createReg(getMemBase()));
    addExpr(Inst, getMemIndex());
    addExpr(Inst, getMemOffset());
  }

```
- **EN**: Implements logic around `addMEMrriOperands`, `assert`, `addOperand`, `addExpr`, ...; this block works at the MC layer.
- **CN**: 围绕 `addMEMrriOperands`, `assert`, `addOperand`, `addExpr`, ... 实现具体逻辑；这一段工作在 MC 层。

### Lines 549-564
```cpp
  void addMEMzriOperands(MCInst &Inst, unsigned N) const {
    assert(N == 3 && "Invalid number of operands!");

    Inst.addOperand(MCOperand::createImm(0));
    Inst.addOperand(MCOperand::createReg(getMemIndexReg()));
    addExpr(Inst, getMemOffset());
  }

  void addMEMziiOperands(MCInst &Inst, unsigned N) const {
    assert(N == 3 && "Invalid number of operands!");

    Inst.addOperand(MCOperand::createImm(0));
    addExpr(Inst, getMemIndex());
    addExpr(Inst, getMemOffset());
  }

```
- **EN**: Implements logic around `addMEMzriOperands`, `assert`, `addOperand`, `addExpr`, ...; this block works at the MC layer.
- **CN**: 围绕 `addMEMzriOperands`, `assert`, `addOperand`, `addExpr`, ... 实现具体逻辑；这一段工作在 MC 层。

### Lines 565-581
```cpp
  void addMEMriOperands(MCInst &Inst, unsigned N) const {
    assert(N == 2 && "Invalid number of operands!");

    Inst.addOperand(MCOperand::createReg(getMemBase()));
    addExpr(Inst, getMemOffset());
  }

  void addMEMziOperands(MCInst &Inst, unsigned N) const {
    assert(N == 2 && "Invalid number of operands!");

    Inst.addOperand(MCOperand::createImm(0));
    addExpr(Inst, getMemOffset());
  }

  void addCCOpOperands(MCInst &Inst, unsigned N) const {
    assert(N == 1 && "Invalid number of operands!");

```
- **EN**: Implements logic around `addMEMriOperands`, `assert`, `addOperand`, `addExpr`, ...; this block works at the MC layer.
- **CN**: 围绕 `addMEMriOperands`, `assert`, `addOperand`, `addExpr`, ... 实现具体逻辑；这一段工作在 MC 层。

### Lines 582-600
```cpp
    Inst.addOperand(MCOperand::createImm(getCCVal()));
  }

  void addRDOpOperands(MCInst &Inst, unsigned N) const {
    assert(N == 1 && "Invalid number of operands!");

    Inst.addOperand(MCOperand::createImm(getRDVal()));
  }

  void addMImmOperands(MCInst &Inst, unsigned N) const {
    assert(N == 1 && "Invalid number of operands!");
    const auto *ConstExpr = dyn_cast<MCConstantExpr>(getMImmVal());
    assert(ConstExpr && "Null operands!");
    int64_t Value = ConstExpr->getValue();
    if (getM0Flag())
      Value += 64;
    Inst.addOperand(MCOperand::createImm(Value));
  }

```
- **EN**: Implements logic around `addOperand`, `addRDOpOperands`, `assert`, `addMImmOperands`, ...; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `addOperand`, `addRDOpOperands`, `assert`, `addMImmOperands`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 601-618
```cpp
  static std::unique_ptr<VEOperand> CreateToken(StringRef Str, SMLoc S) {
    auto Op = std::make_unique<VEOperand>(k_Token);
    Op->Tok.Data = Str.data();
    Op->Tok.Length = Str.size();
    Op->StartLoc = S;
    Op->EndLoc = S;
    return Op;
  }

  static std::unique_ptr<VEOperand> CreateReg(MCRegister Reg, SMLoc S,
                                              SMLoc E) {
    auto Op = std::make_unique<VEOperand>(k_Register);
    Op->Reg.Reg = Reg;
    Op->StartLoc = S;
    Op->EndLoc = E;
    return Op;
  }

```
- **EN**: Implements logic around `CreateToken`, `make_unique<VEOperand>`, `data`, `size`, ...; this block returns target-specific results.
- **CN**: 围绕 `CreateToken`, `make_unique<VEOperand>`, `data`, `size`, ... 实现具体逻辑；这一段返回目标相关结果。

### Lines 619-636
```cpp
  static std::unique_ptr<VEOperand> CreateImm(const MCExpr *Val, SMLoc S,
                                              SMLoc E) {
    auto Op = std::make_unique<VEOperand>(k_Immediate);
    Op->Imm.Val = Val;
    Op->StartLoc = S;
    Op->EndLoc = E;
    return Op;
  }

  static std::unique_ptr<VEOperand> CreateCCOp(unsigned CCVal, SMLoc S,
                                               SMLoc E) {
    auto Op = std::make_unique<VEOperand>(k_CCOp);
    Op->CC.CCVal = CCVal;
    Op->StartLoc = S;
    Op->EndLoc = E;
    return Op;
  }

```
- **EN**: Implements logic around `CreateImm`, `make_unique<VEOperand>`, `CreateCCOp`; this block returns target-specific results.
- **CN**: 围绕 `CreateImm`, `make_unique<VEOperand>`, `CreateCCOp` 实现具体逻辑；这一段返回目标相关结果。

### Lines 637-655
```cpp
  static std::unique_ptr<VEOperand> CreateRDOp(unsigned RDVal, SMLoc S,
                                               SMLoc E) {
    auto Op = std::make_unique<VEOperand>(k_RDOp);
    Op->RD.RDVal = RDVal;
    Op->StartLoc = S;
    Op->EndLoc = E;
    return Op;
  }

  static std::unique_ptr<VEOperand> CreateMImm(const MCExpr *Val, bool Flag,
                                               SMLoc S, SMLoc E) {
    auto Op = std::make_unique<VEOperand>(k_MImmOp);
    Op->MImm.Val = Val;
    Op->MImm.M0Flag = Flag;
    Op->StartLoc = S;
    Op->EndLoc = E;
    return Op;
  }

```
- **EN**: Implements logic around `CreateRDOp`, `make_unique<VEOperand>`, `CreateMImm`; this block returns target-specific results.
- **CN**: 围绕 `CreateRDOp`, `make_unique<VEOperand>`, `CreateMImm` 实现具体逻辑；这一段返回目标相关结果。

### Lines 656-673
```cpp
  static bool MorphToI32Reg(VEOperand &Op) {
    MCRegister Reg = Op.getReg();
    unsigned regIdx = Reg - VE::SX0;
    if (regIdx > 63)
      return false;
    Op.Reg.Reg = I32Regs[regIdx];
    return true;
  }

  static bool MorphToF32Reg(VEOperand &Op) {
    MCRegister Reg = Op.getReg();
    unsigned regIdx = Reg - VE::SX0;
    if (regIdx > 63)
      return false;
    Op.Reg.Reg = F32Regs[regIdx];
    return true;
  }

```
- **EN**: Implements logic around `MorphToI32Reg`, `getReg`, `MorphToF32Reg`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `MorphToI32Reg`, `getReg`, `MorphToF32Reg` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 674-691
```cpp
  static bool MorphToF128Reg(VEOperand &Op) {
    MCRegister Reg = Op.getReg();
    unsigned regIdx = Reg - VE::SX0;
    if (regIdx % 2 || regIdx > 63)
      return false;
    Op.Reg.Reg = F128Regs[regIdx / 2];
    return true;
  }

  static bool MorphToVM512Reg(VEOperand &Op) {
    MCRegister Reg = Op.getReg();
    unsigned regIdx = Reg - VE::VM0;
    if (regIdx % 2 || regIdx > 15)
      return false;
    Op.Reg.Reg = VM512Regs[regIdx / 2];
    return true;
  }

```
- **EN**: Implements logic around `MorphToF128Reg`, `getReg`, `MorphToVM512Reg`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `MorphToF128Reg`, `getReg`, `MorphToVM512Reg` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 692-714
```cpp
  static bool MorphToMISCReg(VEOperand &Op) {
    const auto *ConstExpr = dyn_cast<MCConstantExpr>(Op.getImm());
    if (!ConstExpr)
      return false;
    unsigned regIdx = ConstExpr->getValue();
    if (regIdx >= std::size(MISCRegs) || MISCRegs[regIdx] == VE::NoRegister)
      return false;
    Op.Kind = k_Register;
    Op.Reg.Reg = MISCRegs[regIdx];
    return true;
  }

  static std::unique_ptr<VEOperand>
  MorphToMEMri(MCRegister Base, std::unique_ptr<VEOperand> Op) {
    const MCExpr *Imm = Op->getImm();
    Op->Kind = k_MemoryRegImm;
    Op->Mem.Base = Base;
    Op->Mem.IndexReg = MCRegister();
    Op->Mem.Index = nullptr;
    Op->Mem.Offset = Imm;
    return Op;
  }

```
- **EN**: Implements logic around `MorphToMISCReg`, `dyn_cast<MCConstantExpr>`, `getValue`, `MorphToMEMri`, ...; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `MorphToMISCReg`, `dyn_cast<MCConstantExpr>`, `getValue`, `MorphToMEMri`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 715-737
```cpp
  static std::unique_ptr<VEOperand>
  MorphToMEMzi(std::unique_ptr<VEOperand> Op) {
    const MCExpr *Imm = Op->getImm();
    Op->Kind = k_MemoryZeroImm;
    Op->Mem.Base = MCRegister();
    Op->Mem.IndexReg = MCRegister();
    Op->Mem.Index = nullptr;
    Op->Mem.Offset = Imm;
    return Op;
  }

  static std::unique_ptr<VEOperand>
  MorphToMEMrri(MCRegister Base, MCRegister Index,
                std::unique_ptr<VEOperand> Op) {
    const MCExpr *Imm = Op->getImm();
    Op->Kind = k_MemoryRegRegImm;
    Op->Mem.Base = Base;
    Op->Mem.IndexReg = Index;
    Op->Mem.Index = nullptr;
    Op->Mem.Offset = Imm;
    return Op;
  }

```
- **EN**: Implements logic around `MorphToMEMzi`, `getImm`, `MCRegister`, `MorphToMEMrri`; this block returns target-specific results.
- **CN**: 围绕 `MorphToMEMzi`, `getImm`, `MCRegister`, `MorphToMEMrri` 实现具体逻辑；这一段返回目标相关结果。

### Lines 738-760
```cpp
  static std::unique_ptr<VEOperand>
  MorphToMEMrii(MCRegister Base, const MCExpr *Index,
                std::unique_ptr<VEOperand> Op) {
    const MCExpr *Imm = Op->getImm();
    Op->Kind = k_MemoryRegImmImm;
    Op->Mem.Base = Base;
    Op->Mem.IndexReg = MCRegister();
    Op->Mem.Index = Index;
    Op->Mem.Offset = Imm;
    return Op;
  }

  static std::unique_ptr<VEOperand>
  MorphToMEMzri(MCRegister Index, std::unique_ptr<VEOperand> Op) {
    const MCExpr *Imm = Op->getImm();
    Op->Kind = k_MemoryZeroRegImm;
    Op->Mem.Base = MCRegister();
    Op->Mem.IndexReg = Index;
    Op->Mem.Index = nullptr;
    Op->Mem.Offset = Imm;
    return Op;
  }

```
- **EN**: Implements logic around `MorphToMEMrii`, `getImm`, `MCRegister`, `MorphToMEMzri`; this block returns target-specific results.
- **CN**: 围绕 `MorphToMEMrii`, `getImm`, `MCRegister`, `MorphToMEMzri` 实现具体逻辑；这一段返回目标相关结果。

### Lines 761-787
```cpp
  static std::unique_ptr<VEOperand>
  MorphToMEMzii(const MCExpr *Index, std::unique_ptr<VEOperand> Op) {
    const MCExpr *Imm = Op->getImm();
    Op->Kind = k_MemoryZeroImmImm;
    Op->Mem.Base = MCRegister();
    Op->Mem.IndexReg = MCRegister();
    Op->Mem.Index = Index;
    Op->Mem.Offset = Imm;
    return Op;
  }
};

} // end anonymous namespace

bool VEAsmParser::matchAndEmitInstruction(SMLoc IDLoc, unsigned &Opcode,
                                          OperandVector &Operands,
                                          MCStreamer &Out, uint64_t &ErrorInfo,
                                          bool MatchingInlineAsm) {
  MCInst Inst;
  unsigned MatchResult =
      MatchInstructionImpl(Operands, Inst, ErrorInfo, MatchingInlineAsm);
  switch (MatchResult) {
  case Match_Success:
    Inst.setLoc(IDLoc);
    Out.emitInstruction(Inst, getSTI());
    return false;

```
- **EN**: Implements logic around `MorphToMEMzii`, `getImm`, `MCRegister`, `matchAndEmitInstruction`, ...; this block uses `switch`-based dispatch; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `MorphToMEMzii`, `getImm`, `MCRegister`, `matchAndEmitInstruction`, ... 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果，工作在 MC 层。

### Lines 788-810
```cpp
  case Match_MissingFeature:
    return Error(IDLoc,
                 "instruction requires a CPU feature not currently enabled");

  case Match_InvalidOperand: {
    SMLoc ErrorLoc = IDLoc;
    if (ErrorInfo != ~0ULL) {
      if (ErrorInfo >= Operands.size())
        return Error(IDLoc, "too few operands for instruction");

      ErrorLoc = ((VEOperand &)*Operands[ErrorInfo]).getStartLoc();
      if (ErrorLoc == SMLoc())
        ErrorLoc = IDLoc;
    }

    return Error(ErrorLoc, "invalid operand for instruction");
  }
  case Match_MnemonicFail:
    return Error(IDLoc, "invalid instruction mnemonic");
  }
  llvm_unreachable("Implement any new match types added!");
}

```
- **EN**: Implements logic around `Error`, `getStartLoc`, `llvm_unreachable`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `Error`, `getStartLoc`, `llvm_unreachable` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 811-830
```cpp
bool VEAsmParser::parseRegister(MCRegister &Reg, SMLoc &StartLoc,
                                SMLoc &EndLoc) {
  if (!tryParseRegister(Reg, StartLoc, EndLoc).isSuccess())
    return Error(StartLoc, "invalid register name");
  return false;
}

/// Parses a register name using a given matching function.
/// Checks for lowercase or uppercase if necessary.
MCRegister VEAsmParser::parseRegisterName(MCRegister (*matchFn)(StringRef)) {
  StringRef Name = Parser.getTok().getString();

  MCRegister RegNum = matchFn(Name);

  // GCC supports case insensitive register names. All of the VE registers
  // are all lower case.
  if (!RegNum) {
    RegNum = matchFn(Name.lower());
  }

```
- **EN**: Implements logic around `parseRegister`, `Error`, `parseRegisterName`, `getTok`, ...; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `parseRegister`, `Error`, `parseRegisterName`, `getTok`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 831-851
```cpp
  return RegNum;
}

/// Maps from the set of all register names to a register number.
/// \note Generated by TableGen.
static MCRegister MatchRegisterName(StringRef Name);

/// Maps from the set of all alternative registernames to a register number.
/// \note Generated by TableGen.
static MCRegister MatchRegisterAltName(StringRef Name);

ParseStatus VEAsmParser::tryParseRegister(MCRegister &Reg, SMLoc &StartLoc,
                                          SMLoc &EndLoc) {
  const AsmToken Tok = Parser.getTok();
  StartLoc = Tok.getLoc();
  EndLoc = Tok.getEndLoc();
  Reg = VE::NoRegister;
  if (getLexer().getKind() != AsmToken::Percent)
    return ParseStatus::NoMatch;
  Parser.Lex();

```
- **EN**: Implements logic around `MatchRegisterName`, `MatchRegisterAltName`, `tryParseRegister`, `getTok`, ...; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `MatchRegisterName`, `MatchRegisterAltName`, `tryParseRegister`, `getTok`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 852-873
```cpp
  Reg = parseRegisterName(&MatchRegisterName);
  if (Reg == VE::NoRegister)
    Reg = parseRegisterName(&MatchRegisterAltName);

  if (Reg != VE::NoRegister) {
    Parser.Lex();
    return ParseStatus::Success;
  }

  getLexer().UnLex(Tok);
  return ParseStatus::NoMatch;
}

static StringRef parseCC(StringRef Name, unsigned Prefix, unsigned Suffix,
                         bool IntegerCC, bool OmitCC, SMLoc NameLoc,
                         OperandVector *Operands) {
  // Parse instructions with a conditional code. For example, 'bne' is
  // converted into two operands 'b' and 'ne'.
  StringRef Cond = Name.slice(Prefix, Suffix);
  VECC::CondCode CondCode =
      IntegerCC ? stringToVEICondCode(Cond) : stringToVEFCondCode(Cond);

```
- **EN**: Implements logic around `parseRegisterName`, `Lex`, `getLexer`, `parseCC`, ...; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `parseRegisterName`, `Lex`, `getLexer`, `parseCC`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 874-893
```cpp
  // If OmitCC is enabled, CC_AT and CC_AF is treated as a part of mnemonic.
  if (CondCode != VECC::UNKNOWN &&
      (!OmitCC || (CondCode != VECC::CC_AT && CondCode != VECC::CC_AF))) {
    StringRef SuffixStr = Name.substr(Suffix);
    // Push "b".
    Name = Name.slice(0, Prefix);
    Operands->push_back(VEOperand::CreateToken(Name, NameLoc));
    // Push $cond part.
    SMLoc CondLoc = SMLoc::getFromPointer(NameLoc.getPointer() + Prefix);
    SMLoc SuffixLoc = SMLoc::getFromPointer(NameLoc.getPointer() + Suffix);
    Operands->push_back(VEOperand::CreateCCOp(CondCode, CondLoc, SuffixLoc));
    // push suffix like ".l.t"
    if (!SuffixStr.empty())
      Operands->push_back(VEOperand::CreateToken(SuffixStr, SuffixLoc));
  } else {
    Operands->push_back(VEOperand::CreateToken(Name, NameLoc));
  }
  return Name;
}

```
- **EN**: Implements logic around `substr`, `slice`, `push_back`, `getFromPointer`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `substr`, `slice`, `push_back`, `getFromPointer` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 894-917
```cpp
static StringRef parseRD(StringRef Name, unsigned Prefix, SMLoc NameLoc,
                         OperandVector *Operands) {
  // Parse instructions with a conditional code. For example, 'cvt.w.d.sx.rz'
  // is converted into two operands 'cvt.w.d.sx' and '.rz'.
  StringRef RD = Name.substr(Prefix);
  VERD::RoundingMode RoundingMode = stringToVERD(RD);

  if (RoundingMode != VERD::UNKNOWN) {
    Name = Name.slice(0, Prefix);
    // push 1st like `cvt.w.d.sx`
    Operands->push_back(VEOperand::CreateToken(Name, NameLoc));
    SMLoc SuffixLoc =
        SMLoc::getFromPointer(NameLoc.getPointer() + (RD.data() - Name.data()));
    SMLoc SuffixEnd =
        SMLoc::getFromPointer(NameLoc.getPointer() + (RD.end() - Name.data()));
    // push $round if it has rounding mode
    Operands->push_back(
        VEOperand::CreateRDOp(RoundingMode, SuffixLoc, SuffixEnd));
  } else {
    Operands->push_back(VEOperand::CreateToken(Name, NameLoc));
  }
  return Name;
}

```
- **EN**: Implements logic around `parseRD`, `substr`, `stringToVERD`, `slice`, ...; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `parseRD`, `substr`, `stringToVERD`, `slice`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 918-949
```cpp
// Split the mnemonic into ASM operand, conditional code and instruction
// qualifier (half-word, byte).
StringRef VEAsmParser::splitMnemonic(StringRef Name, SMLoc NameLoc,
                                     OperandVector *Operands) {
  // Create the leading tokens for the mnemonic
  StringRef Mnemonic = Name;

  if (Name[0] == 'b') {
    // Match b?? or br??.
    size_t Start = 1;
    size_t Next = Name.find('.');
    // Adjust position of CondCode.
    if (Name.size() > 1 && Name[1] == 'r')
      Start = 2;
    // Check suffix.
    bool ICC = true;
    if (Next + 1 < Name.size() &&
        (Name[Next + 1] == 'd' || Name[Next + 1] == 's'))
      ICC = false;
    Mnemonic = parseCC(Name, Start, Next, ICC, true, NameLoc, Operands);
  } else if (Name.starts_with("cmov.l.") || Name.starts_with("cmov.w.") ||
             Name.starts_with("cmov.d.") || Name.starts_with("cmov.s.")) {
    bool ICC = Name[5] == 'l' || Name[5] == 'w';
    Mnemonic = parseCC(Name, 7, Name.size(), ICC, false, NameLoc, Operands);
  } else if (Name.starts_with("cvt.w.d.sx") || Name.starts_with("cvt.w.d.zx") ||
             Name.starts_with("cvt.w.s.sx") || Name.starts_with("cvt.w.s.zx")) {
    Mnemonic = parseRD(Name, 10, NameLoc, Operands);
  } else if (Name.starts_with("cvt.l.d")) {
    Mnemonic = parseRD(Name, 7, NameLoc, Operands);
  } else if (Name.starts_with("vcvt.w.d.sx") ||
             Name.starts_with("vcvt.w.d.zx") ||
             Name.starts_with("vcvt.w.s.sx") ||
```
- **EN**: Implements logic around `splitMnemonic`, `find`, `parseCC`, `starts_with`, ...; this block applies conditional target rules.
- **CN**: 围绕 `splitMnemonic`, `find`, `parseCC`, `starts_with`, ... 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 950-972
```cpp
             Name.starts_with("vcvt.w.s.zx")) {
    Mnemonic = parseRD(Name, 11, NameLoc, Operands);
  } else if (Name.starts_with("vcvt.l.d")) {
    Mnemonic = parseRD(Name, 8, NameLoc, Operands);
  } else if (Name.starts_with("pvcvt.w.s.lo") ||
             Name.starts_with("pvcvt.w.s.up")) {
    Mnemonic = parseRD(Name, 12, NameLoc, Operands);
  } else if (Name.starts_with("pvcvt.w.s")) {
    Mnemonic = parseRD(Name, 9, NameLoc, Operands);
  } else if (Name.starts_with("vfmk.l.") || Name.starts_with("vfmk.w.") ||
             Name.starts_with("vfmk.d.") || Name.starts_with("vfmk.s.")) {
    bool ICC = Name[5] == 'l' || Name[5] == 'w' ? true : false;
    Mnemonic = parseCC(Name, 7, Name.size(), ICC, true, NameLoc, Operands);
  } else if (Name.starts_with("pvfmk.w.lo.") ||
             Name.starts_with("pvfmk.w.up.") ||
             Name.starts_with("pvfmk.s.lo.") ||
             Name.starts_with("pvfmk.s.up.")) {
    bool ICC = Name[6] == 'l' || Name[6] == 'w' ? true : false;
    Mnemonic = parseCC(Name, 11, Name.size(), ICC, true, NameLoc, Operands);
  } else {
    Operands->push_back(VEOperand::CreateToken(Mnemonic, NameLoc));
  }

```
- **EN**: Implements logic around `starts_with`, `parseRD`, `parseCC`, `push_back`; this block applies conditional target rules.
- **CN**: 围绕 `starts_with`, `parseRD`, `parseCC`, `push_back` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 973-990
```cpp
  return Mnemonic;
}

static void applyMnemonicAliases(StringRef &Mnemonic,
                                 const FeatureBitset &Features,
                                 unsigned VariantID);

bool VEAsmParser::parseInstruction(ParseInstructionInfo &Info, StringRef Name,
                                   SMLoc NameLoc, OperandVector &Operands) {
  // If the target architecture uses MnemonicAlias, call it here to parse
  // operands correctly.
  applyMnemonicAliases(Name, getAvailableFeatures(), 0);

  // Split name to first token and the rest, e.g. "bgt.l.t" to "b", "gt", and
  // ".l.t".  We treat "b" as a mnemonic, "gt" as first operand, and ".l.t"
  // as second operand.
  StringRef Mnemonic = splitMnemonic(Name, NameLoc, &Operands);

```
- **EN**: Implements logic around `applyMnemonicAliases`, `parseInstruction`, `splitMnemonic`; this block returns target-specific results.
- **CN**: 围绕 `applyMnemonicAliases`, `parseInstruction`, `splitMnemonic` 实现具体逻辑；这一段返回目标相关结果。

### Lines 991-1014
```cpp
  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    // Read the first operand.
    if (!parseOperand(Operands, Mnemonic).isSuccess()) {
      SMLoc Loc = getLexer().getLoc();
      return Error(Loc, "unexpected token");
    }

    while (getLexer().is(AsmToken::Comma)) {
      Parser.Lex(); // Eat the comma.
      // Parse and remember the operand.
      if (!parseOperand(Operands, Mnemonic).isSuccess()) {
        SMLoc Loc = getLexer().getLoc();
        return Error(Loc, "unexpected token");
      }
    }
  }
  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    SMLoc Loc = getLexer().getLoc();
    return Error(Loc, "unexpected token");
  }
  Parser.Lex(); // Consume the EndOfStatement.
  return false;
}

```
- **EN**: Implements logic around `getLexer`, `Error`, `Lex`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getLexer`, `Error`, `Lex` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 1015-1033
```cpp
ParseStatus VEAsmParser::parseDirective(AsmToken DirectiveID) {
  std::string IDVal = DirectiveID.getIdentifier().lower();

  // Defines VE specific directives.  Reference is "Vector Engine Assembly
  // Language Reference Manual":
  // https://www.hpc.nec/documents/sdk/pdfs/VectorEngine-as-manual-v1.3.pdf

  // The .word is 4 bytes long on VE.
  if (IDVal == ".word")
    return parseLiteralValues(4, DirectiveID.getLoc());

  // The .long is 8 bytes long on VE.
  if (IDVal == ".long")
    return parseLiteralValues(8, DirectiveID.getLoc());

  // The .llong is 8 bytes long on VE.
  if (IDVal == ".llong")
    return parseLiteralValues(8, DirectiveID.getLoc());

```
- **EN**: Implements logic around `parseDirective`, `getIdentifier`, `parseLiteralValues`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `parseDirective`, `getIdentifier`, `parseLiteralValues` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 1034-1052
```cpp
  // Let the MC layer to handle other directives.
  return ParseStatus::NoMatch;
}

/// parseLiteralValues
///  ::= .word expression [, expression]*
///  ::= .long expression [, expression]*
///  ::= .llong expression [, expression]*
bool VEAsmParser::parseLiteralValues(unsigned Size, SMLoc L) {
  auto parseOne = [&]() -> bool {
    const MCExpr *Value;
    if (getParser().parseExpression(Value))
      return true;
    getParser().getStreamer().emitValue(Value, Size, L);
    return false;
  };
  return (parseMany(parseOne));
}

```
- **EN**: Implements logic around `parseLiteralValues`, `getParser`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `parseLiteralValues`, `getParser` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 1053-1070
```cpp
/// Extract \code @lo32/@hi32/etc \endcode specifier from expression.
/// Recursively scan the expression and check for VK_HI32/LO32/etc
/// symbol variants.  If all symbols with modifier use the same
/// variant, return the corresponding VE::Specifier,
/// and a modified expression using the default symbol variant.
/// Otherwise, return NULL.
const MCExpr *VEAsmParser::extractSpecifier(const MCExpr *E,
                                            VE::Specifier &Variant) {
  MCContext &Context = getParser().getContext();
  Variant = VE::S_None;

  switch (E->getKind()) {
  case MCExpr::Target:
  case MCExpr::Constant:
    return nullptr;
  case MCExpr::Specifier:
    llvm_unreachable("unused by this backend");

```
- **EN**: Implements logic around `extractSpecifier`, `getParser`, `llvm_unreachable`; this block uses `switch`-based dispatch; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `extractSpecifier`, `getParser`, `llvm_unreachable` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果，工作在 MC 层。

### Lines 1071-1102
```cpp
  case MCExpr::SymbolRef: {
    const MCSymbolRefExpr *SRE = cast<MCSymbolRefExpr>(E);

    switch (SRE->getSpecifier()) {
    case VE::S_None:
      // Use VK_REFLONG to a symbol without modifiers.
      Variant = VE::S_REFLONG;
      break;
    case VE::S_HI32:
      Variant = VE::S_HI32;
      break;
    case VE::S_LO32:
      Variant = VE::S_LO32;
      break;
    case VE::S_PC_HI32:
      Variant = VE::S_PC_HI32;
      break;
    case VE::S_PC_LO32:
      Variant = VE::S_PC_LO32;
      break;
    case VE::S_GOT_HI32:
      Variant = VE::S_GOT_HI32;
      break;
    case VE::S_GOT_LO32:
      Variant = VE::S_GOT_LO32;
      break;
    case VE::S_GOTOFF_HI32:
      Variant = VE::S_GOTOFF_HI32;
      break;
    case VE::S_GOTOFF_LO32:
      Variant = VE::S_GOTOFF_LO32;
      break;
```
- **EN**: Implements logic around `cast<MCSymbolRefExpr>`; this block uses `switch`-based dispatch.
- **CN**: 围绕 `cast<MCSymbolRefExpr>` 实现具体逻辑；这一段使用 `switch` 分派。

### Lines 1103-1124
```cpp
    case VE::S_PLT_HI32:
      Variant = VE::S_PLT_HI32;
      break;
    case VE::S_PLT_LO32:
      Variant = VE::S_PLT_LO32;
      break;
    case VE::S_TLS_GD_HI32:
      Variant = VE::S_TLS_GD_HI32;
      break;
    case VE::S_TLS_GD_LO32:
      Variant = VE::S_TLS_GD_LO32;
      break;
    case VE::S_TPOFF_HI32:
      Variant = VE::S_TPOFF_HI32;
      break;
    case VE::S_TPOFF_LO32:
      Variant = VE::S_TPOFF_LO32;
      break;
    default:
      return nullptr;
    }

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 1125-1141
```cpp
    return MCSymbolRefExpr::create(&SRE->getSymbol(), Context);
  }

  case MCExpr::Unary: {
    const MCUnaryExpr *UE = cast<MCUnaryExpr>(E);
    const MCExpr *Sub = extractSpecifier(UE->getSubExpr(), Variant);
    if (!Sub)
      return nullptr;
    return MCUnaryExpr::create(UE->getOpcode(), Sub, Context);
  }

  case MCExpr::Binary: {
    const MCBinaryExpr *BE = cast<MCBinaryExpr>(E);
    VE::Specifier LHSVariant, RHSVariant;
    const MCExpr *LHS = extractSpecifier(BE->getLHS(), LHSVariant);
    const MCExpr *RHS = extractSpecifier(BE->getRHS(), RHSVariant);

```
- **EN**: Implements logic around `create`, `cast<MCUnaryExpr>`, `extractSpecifier`, `cast<MCBinaryExpr>`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `create`, `cast<MCUnaryExpr>`, `extractSpecifier`, `cast<MCBinaryExpr>` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 1142-1158
```cpp
    if (!LHS && !RHS)
      return nullptr;

    if (!LHS)
      LHS = BE->getLHS();
    if (!RHS)
      RHS = BE->getRHS();

    if (LHSVariant == VE::S_None)
      Variant = RHSVariant;
    else if (RHSVariant == VE::S_None)
      Variant = LHSVariant;
    else if (LHSVariant == RHSVariant)
      Variant = LHSVariant;
    else
      return nullptr;

```
- **EN**: Implements logic around `getLHS`, `getRHS`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getLHS`, `getRHS` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 1159-1178
```cpp
    return MCBinaryExpr::create(BE->getOpcode(), LHS, RHS, Context);
  }
  }

  llvm_unreachable("Invalid expression kind!");
}

/// This differs from the default "parseExpression" in that it handles
/// relocation specifiers.
bool VEAsmParser::parseExpression(const MCExpr *&EVal) {
  // Handle \code symbol @lo32/@hi32/etc \endcode.
  if (getParser().parseExpression(EVal))
    return true;

  // Convert MCSymbolRefExpr with specifier to MCSpecifierExpr.
  VE::Specifier Specifier;
  const MCExpr *E = extractSpecifier(EVal, Specifier);
  if (E)
    EVal = MCSpecifierExpr::create(E, Specifier, getParser().getContext());

```
- **EN**: Implements logic around `create`, `llvm_unreachable`, `parseExpression`, `extractSpecifier`; this block applies conditional target rules; returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `create`, `llvm_unreachable`, `parseExpression`, `extractSpecifier` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位。

### Lines 1179-1195
```cpp
  return false;
}

ParseStatus VEAsmParser::parseMEMOperand(OperandVector &Operands) {
  LLVM_DEBUG(dbgs() << "parseMEMOperand\n");
  const AsmToken &Tok = Parser.getTok();
  SMLoc S = Tok.getLoc();
  SMLoc E = Tok.getEndLoc();
  // Parse ASX format
  //   disp
  //   disp(, base)
  //   disp(index)
  //   disp(index, base)
  //   (, base)
  //   (index)
  //   (index, base)

```
- **EN**: Implements logic around `parseMEMOperand`, `getTok`, `getLoc`, `getEndLoc`; this block returns target-specific results.
- **CN**: 围绕 `parseMEMOperand`, `getTok`, `getLoc`, `getEndLoc` 实现具体逻辑；这一段返回目标相关结果。

### Lines 1196-1212
```cpp
  std::unique_ptr<VEOperand> Offset;
  switch (getLexer().getKind()) {
  default:
    return ParseStatus::NoMatch;

  case AsmToken::Minus:
  case AsmToken::Integer:
  case AsmToken::Dot:
  case AsmToken::Identifier: {
    const MCExpr *EVal;
    if (!parseExpression(EVal))
      Offset = VEOperand::CreateImm(EVal, S, E);
    else
      return ParseStatus::NoMatch;
    break;
  }

```
- **EN**: Implements logic around `CreateImm`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `CreateImm` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果。

### Lines 1213-1228
```cpp
  case AsmToken::LParen:
    // empty disp (= 0)
    Offset =
        VEOperand::CreateImm(MCConstantExpr::create(0, getContext()), S, E);
    break;
  }

  switch (getLexer().getKind()) {
  default:
    return ParseStatus::Failure;

  case AsmToken::EndOfStatement:
    Operands.push_back(VEOperand::MorphToMEMzii(
        MCConstantExpr::create(0, getContext()), std::move(Offset)));
    return ParseStatus::Success;

```
- **EN**: Implements logic around `CreateImm`, `push_back`, `create`; this block uses `switch`-based dispatch; returns target-specific results.
- **CN**: 围绕 `CreateImm`, `push_back`, `create` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果。

### Lines 1229-1249
```cpp
  case AsmToken::LParen:
    Parser.Lex(); // Eat the (
    break;
  }

  const MCExpr *IndexValue = nullptr;
  MCRegister IndexReg;

  switch (getLexer().getKind()) {
  default:
    if (parseRegister(IndexReg, S, E))
      return ParseStatus::Failure;
    break;

  case AsmToken::Minus:
  case AsmToken::Integer:
  case AsmToken::Dot:
    if (getParser().parseExpression(IndexValue, E))
      return ParseStatus::Failure;
    break;

```
- **EN**: Implements logic around `Lex`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `Lex` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果。

### Lines 1250-1266
```cpp
  case AsmToken::Comma:
    // empty index
    IndexValue = MCConstantExpr::create(0, getContext());
    break;
  }

  switch (getLexer().getKind()) {
  default:
    return ParseStatus::Failure;

  case AsmToken::RParen:
    Parser.Lex(); // Eat the )
    Operands.push_back(
        IndexValue ? VEOperand::MorphToMEMzii(IndexValue, std::move(Offset))
                   : VEOperand::MorphToMEMzri(IndexReg, std::move(Offset)));
    return ParseStatus::Success;

```
- **EN**: Implements logic around `create`, `Lex`, `push_back`, `MorphToMEMzii`, ...; this block uses `switch`-based dispatch; returns target-specific results.
- **CN**: 围绕 `create`, `Lex`, `push_back`, `MorphToMEMzii`, ... 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果。

### Lines 1267-1284
```cpp
  case AsmToken::Comma:
    Parser.Lex(); // Eat the ,
    break;
  }

  MCRegister BaseReg;
  if (parseRegister(BaseReg, S, E))
    return ParseStatus::Failure;

  if (!Parser.getTok().is(AsmToken::RParen))
    return ParseStatus::Failure;

  Parser.Lex(); // Eat the )
  Operands.push_back(
      IndexValue
          ? VEOperand::MorphToMEMrii(BaseReg, IndexValue, std::move(Offset))
          : VEOperand::MorphToMEMrri(BaseReg, IndexReg, std::move(Offset)));

```
- **EN**: Implements logic around `Lex`, `push_back`, `MorphToMEMrii`, `MorphToMEMrri`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `Lex`, `push_back`, `MorphToMEMrii`, `MorphToMEMrri` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 1285-1301
```cpp
  return ParseStatus::Success;
}

ParseStatus VEAsmParser::parseMEMAsOperand(OperandVector &Operands) {
  LLVM_DEBUG(dbgs() << "parseMEMAsOperand\n");
  const AsmToken &Tok = Parser.getTok();
  SMLoc S = Tok.getLoc();
  SMLoc E = Tok.getEndLoc();
  // Parse AS format
  //   disp
  //   disp(, base)
  //   disp(base)
  //   disp()
  //   (, base)
  //   (base)
  //   base

```
- **EN**: Implements logic around `parseMEMAsOperand`, `getTok`, `getLoc`, `getEndLoc`; this block returns target-specific results.
- **CN**: 围绕 `parseMEMAsOperand`, `getTok`, `getLoc`, `getEndLoc` 实现具体逻辑；这一段返回目标相关结果。

### Lines 1302-1319
```cpp
  MCRegister BaseReg;
  std::unique_ptr<VEOperand> Offset;
  switch (getLexer().getKind()) {
  default:
    return ParseStatus::NoMatch;

  case AsmToken::Minus:
  case AsmToken::Integer:
  case AsmToken::Dot:
  case AsmToken::Identifier: {
    const MCExpr *EVal;
    if (!parseExpression(EVal))
      Offset = VEOperand::CreateImm(EVal, S, E);
    else
      return ParseStatus::NoMatch;
    break;
  }

```
- **EN**: Implements logic around `CreateImm`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `CreateImm` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果。

### Lines 1320-1337
```cpp
  case AsmToken::Percent:
    if (parseRegister(BaseReg, S, E))
      return ParseStatus::NoMatch;
    Offset =
        VEOperand::CreateImm(MCConstantExpr::create(0, getContext()), S, E);
    break;

  case AsmToken::LParen:
    // empty disp (= 0)
    Offset =
        VEOperand::CreateImm(MCConstantExpr::create(0, getContext()), S, E);
    break;
  }

  switch (getLexer().getKind()) {
  default:
    return ParseStatus::Failure;

```
- **EN**: Implements logic around `CreateImm`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `CreateImm` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果。

### Lines 1338-1357
```cpp
  case AsmToken::EndOfStatement:
  case AsmToken::Comma:
    Operands.push_back(BaseReg != VE::NoRegister
                           ? VEOperand::MorphToMEMri(BaseReg, std::move(Offset))
                           : VEOperand::MorphToMEMzi(std::move(Offset)));
    return ParseStatus::Success;

  case AsmToken::LParen:
    if (BaseReg != VE::NoRegister)
      return ParseStatus::Failure;
    Parser.Lex(); // Eat the (
    break;
  }

  switch (getLexer().getKind()) {
  default:
    if (parseRegister(BaseReg, S, E))
      return ParseStatus::Failure;
    break;

```
- **EN**: Implements logic around `push_back`, `MorphToMEMri`, `MorphToMEMzi`, `Lex`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `push_back`, `MorphToMEMri`, `MorphToMEMzi`, `Lex` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果。

### Lines 1358-1375
```cpp
  case AsmToken::Comma:
    Parser.Lex(); // Eat the ,
    if (parseRegister(BaseReg, S, E))
      return ParseStatus::Failure;
    break;

  case AsmToken::RParen:
    break;
  }

  if (!Parser.getTok().is(AsmToken::RParen))
    return ParseStatus::Failure;

  Parser.Lex(); // Eat the )
  Operands.push_back(BaseReg != VE::NoRegister
                         ? VEOperand::MorphToMEMri(BaseReg, std::move(Offset))
                         : VEOperand::MorphToMEMzi(std::move(Offset)));

```
- **EN**: Implements logic around `Lex`, `push_back`, `MorphToMEMri`, `MorphToMEMzi`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `Lex`, `push_back`, `MorphToMEMri`, `MorphToMEMzi` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 1376-1396
```cpp
  return ParseStatus::Success;
}

ParseStatus VEAsmParser::parseMImmOperand(OperandVector &Operands) {
  LLVM_DEBUG(dbgs() << "parseMImmOperand\n");

  // Parsing "(" + number + ")0/1"
  const AsmToken Tok1 = Parser.getTok();
  if (!Tok1.is(AsmToken::LParen))
    return ParseStatus::NoMatch;

  Parser.Lex(); // Eat the '('.

  const AsmToken Tok2 = Parser.getTok();
  SMLoc E;
  const MCExpr *EVal;
  if (!Tok2.is(AsmToken::Integer) || getParser().parseExpression(EVal, E)) {
    getLexer().UnLex(Tok1);
    return ParseStatus::NoMatch;
  }

```
- **EN**: Implements logic around `parseMImmOperand`, `getTok`, `Lex`, `getLexer`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `parseMImmOperand`, `getTok`, `Lex`, `getLexer` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 1397-1419
```cpp
  const AsmToken Tok3 = Parser.getTok();
  if (!Tok3.is(AsmToken::RParen)) {
    getLexer().UnLex(Tok2);
    getLexer().UnLex(Tok1);
    return ParseStatus::NoMatch;
  }
  Parser.Lex(); // Eat the ')'.

  const AsmToken &Tok4 = Parser.getTok();
  StringRef Suffix = Tok4.getString();
  if (Suffix != "1" && Suffix != "0") {
    getLexer().UnLex(Tok3);
    getLexer().UnLex(Tok2);
    getLexer().UnLex(Tok1);
    return ParseStatus::NoMatch;
  }
  Parser.Lex(); // Eat the value.
  SMLoc EndLoc = SMLoc::getFromPointer(Suffix.end());
  Operands.push_back(
      VEOperand::CreateMImm(EVal, Suffix == "0", Tok1.getLoc(), EndLoc));
  return ParseStatus::Success;
}

```
- **EN**: Implements logic around `getTok`, `getLexer`, `Lex`, `getString`, ...; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getTok`, `getLexer`, `Lex`, `getString`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 1420-1436
```cpp
ParseStatus VEAsmParser::parseOperand(OperandVector &Operands,
                                      StringRef Mnemonic) {
  LLVM_DEBUG(dbgs() << "parseOperand\n");
  ParseStatus Res = MatchOperandParserImpl(Operands, Mnemonic);

  // If there wasn't a custom match, try the generic matcher below. Otherwise,
  // there was a match, but an error occurred, in which case, just return that
  // the operand parsing failed.
  if (Res.isSuccess() || Res.isFailure())
    return Res;

  switch (getLexer().getKind()) {
  case AsmToken::LParen: {
    // Parsing "(" + %vreg + ", " + %vreg + ")"
    const AsmToken Tok1 = Parser.getTok();
    Parser.Lex(); // Eat the '('.

```
- **EN**: Implements logic around `parseOperand`, `MatchOperandParserImpl`, `getTok`, `Lex`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `parseOperand`, `MatchOperandParserImpl`, `getTok`, `Lex` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果。

### Lines 1437-1452
```cpp
    MCRegister Reg1;
    SMLoc S1, E1;
    if (!tryParseRegister(Reg1, S1, E1).isSuccess()) {
      getLexer().UnLex(Tok1);
      return ParseStatus::NoMatch;
    }

    if (!Parser.getTok().is(AsmToken::Comma))
      return ParseStatus::Failure;
    Parser.Lex(); // Eat the ','.

    MCRegister Reg2;
    SMLoc S2, E2;
    if (!tryParseRegister(Reg2, S2, E2).isSuccess())
      return ParseStatus::Failure;

```
- **EN**: Implements logic around `getLexer`, `Lex`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getLexer`, `Lex` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 1453-1469
```cpp
    if (!Parser.getTok().is(AsmToken::RParen))
      return ParseStatus::Failure;

    Operands.push_back(VEOperand::CreateToken(Tok1.getString(), Tok1.getLoc()));
    Operands.push_back(VEOperand::CreateReg(Reg1, S1, E1));
    Operands.push_back(VEOperand::CreateReg(Reg2, S2, E2));
    Operands.push_back(VEOperand::CreateToken(Parser.getTok().getString(),
                                              Parser.getTok().getLoc()));
    Parser.Lex(); // Eat the ')'.
    break;
  }
  default: {
    std::unique_ptr<VEOperand> Op;
    Res = parseVEAsmOperand(Op);
    if (!Res.isSuccess() || !Op)
      return ParseStatus::Failure;

```
- **EN**: Implements logic around `push_back`, `getTok`, `Lex`, `parseVEAsmOperand`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `push_back`, `getTok`, `Lex`, `parseVEAsmOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 1470-1485
```cpp
    // Push the parsed operand into the list of operands
    Operands.push_back(std::move(Op));

    if (!Parser.getTok().is(AsmToken::LParen))
      break;

    // Parsing %vec-reg + "(" + %sclar-reg/number + ")"
    std::unique_ptr<VEOperand> Op1 = VEOperand::CreateToken(
        Parser.getTok().getString(), Parser.getTok().getLoc());
    Parser.Lex(); // Eat the '('.

    std::unique_ptr<VEOperand> Op2;
    Res = parseVEAsmOperand(Op2);
    if (!Res.isSuccess() || !Op2)
      return ParseStatus::Failure;

```
- **EN**: Implements logic around `push_back`, `CreateToken`, `getTok`, `Lex`, ...; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `push_back`, `CreateToken`, `getTok`, `Lex`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 1486-1506
```cpp
    if (!Parser.getTok().is(AsmToken::RParen))
      return ParseStatus::Failure;

    Operands.push_back(std::move(Op1));
    Operands.push_back(std::move(Op2));
    Operands.push_back(VEOperand::CreateToken(Parser.getTok().getString(),
                                              Parser.getTok().getLoc()));
    Parser.Lex(); // Eat the ')'.
    break;
  }
  }

  return ParseStatus::Success;
}

ParseStatus VEAsmParser::parseVEAsmOperand(std::unique_ptr<VEOperand> &Op) {
  LLVM_DEBUG(dbgs() << "parseVEAsmOperand\n");
  SMLoc S = Parser.getTok().getLoc();
  SMLoc E = SMLoc::getFromPointer(Parser.getTok().getLoc().getPointer() - 1);
  const MCExpr *EVal;

```
- **EN**: Implements logic around `push_back`, `getTok`, `Lex`, `parseVEAsmOperand`, ...; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `push_back`, `getTok`, `Lex`, `parseVEAsmOperand`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 1507-1528
```cpp
  Op = nullptr;
  switch (getLexer().getKind()) {
  default:
    break;

  case AsmToken::Percent: {
    MCRegister Reg;
    if (tryParseRegister(Reg, S, E).isSuccess())
      Op = VEOperand::CreateReg(Reg, S, E);
    break;
  }
  case AsmToken::Minus:
  case AsmToken::Integer:
  case AsmToken::Dot:
  case AsmToken::Identifier:
    if (!parseExpression(EVal))
      Op = VEOperand::CreateImm(EVal, S, E);
    break;
  }
  return Op ? ParseStatus::Success : ParseStatus::Failure;
}

```
- **EN**: Implements logic around `CreateReg`, `CreateImm`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `CreateReg`, `CreateImm` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果。

### Lines 1529-1560
```cpp
// Force static initialization.
extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void LLVMInitializeVEAsmParser() {
  RegisterMCAsmParser<VEAsmParser> A(getTheVETarget());
}

#define GET_REGISTER_MATCHER
#define GET_MATCHER_IMPLEMENTATION
#include "VEGenAsmMatcher.inc"

unsigned VEAsmParser::validateTargetOperandClass(MCParsedAsmOperand &GOp,
                                                 unsigned Kind) {
  VEOperand &Op = (VEOperand &)GOp;

  // VE uses identical register name for all registers like both
  // F32 and I32 uses "%s23".  Need to convert the name of them
  // for validation.
  switch (Kind) {
  default:
    break;
  case MCK_F32:
    if (Op.isReg() && VEOperand::MorphToF32Reg(Op))
      return MCTargetAsmParser::Match_Success;
    break;
  case MCK_I32:
    if (Op.isReg() && VEOperand::MorphToI32Reg(Op))
      return MCTargetAsmParser::Match_Success;
    break;
  case MCK_F128:
    if (Op.isReg() && VEOperand::MorphToF128Reg(Op))
      return MCTargetAsmParser::Match_Success;
    break;
  case MCK_VM512:
```
- **EN**: Pulls in the headers needed for this implementation, including `VEGenAsmMatcher.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `VEGenAsmMatcher.inc`。

### Lines 1561-1570
```cpp
    if (Op.isReg() && VEOperand::MorphToVM512Reg(Op))
      return MCTargetAsmParser::Match_Success;
    break;
  case MCK_MISC:
    if (Op.isImm() && VEOperand::MorphToMISCReg(Op))
      return MCTargetAsmParser::Match_Success;
    break;
  }
  return Match_InvalidOperand;
}
```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

## Key Concepts / 关键概念

- **Assembly parsing / 汇编解析**:
  - **EN**: Parses textual assembly into operands and instructions
  - **CN**: 将文本汇编解析为操作数和指令
- **Fixups and relocations / Fixup 与重定位**:
  - **EN**: Bridges symbolic references to concrete relocation records
  - **CN**: 把符号引用连接到具体重定位记录

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `MCTargetDesc/VEMCAsmInfo.h`, `MCTargetDesc/VEMCTargetDesc.h`, `TargetInfo/VETargetInfo.h`, `VE.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/StringRef.h`, `llvm/ADT/Twine.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCInst.h`, `llvm/MC/MCInstrInfo.h`, `llvm/MC/MCParser/AsmLexer.h` ... (+11 more)
- **LLVM subsystems / LLVM 子系统**: MC, Support
- **Generated macros / 生成宏**: `GET_ASSEMBLER_HEADER`, `GET_MATCHER_IMPLEMENTATION`, `GET_REGISTER_MATCHER`
