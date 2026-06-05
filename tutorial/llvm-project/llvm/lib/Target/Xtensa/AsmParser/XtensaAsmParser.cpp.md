# XtensaAsmParser.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Xtensa/AsmParser/XtensaAsmParser.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements the target-specific assembly parser that turns textual assembly into MC operands and MC instructions.
  - **CN**: 实现目标相关的汇编解析器，把文本汇编转换为 MC 操作数和 MC 指令。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24
```cpp
//===- XtensaAsmParser.cpp - Parse Xtensa assembly to MCInst instructions -===//
//
//                     The LLVM Compiler Infrastructure
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "MCTargetDesc/XtensaMCAsmInfo.h"
#include "MCTargetDesc/XtensaMCTargetDesc.h"
#include "MCTargetDesc/XtensaTargetStreamer.h"
#include "TargetInfo/XtensaTargetInfo.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/ADT/StringSwitch.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCInstrInfo.h"
#include "llvm/MC/MCParser/AsmLexer.h"
#include "llvm/MC/MCParser/MCParsedAsmOperand.h"
#include "llvm/MC/MCParser/MCTargetAsmParser.h"
#include "llvm/MC/MCRegisterInfo.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `MCTargetDesc/XtensaMCAsmInfo.h`, `MCTargetDesc/XtensaMCTargetDesc.h`, `MCTargetDesc/XtensaTargetStreamer.h`, `TargetInfo/XtensaTargetInfo.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `MCTargetDesc/XtensaMCAsmInfo.h`, `MCTargetDesc/XtensaMCTargetDesc.h`, `MCTargetDesc/XtensaTargetStreamer.h`, `TargetInfo/XtensaTargetInfo.h`。

### Lines 25-36
```cpp
#include "llvm/MC/MCStreamer.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/MC/MCSymbol.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/Casting.h"

using namespace llvm;

#define DEBUG_TYPE "xtensa-asm-parser"

struct XtensaOperand;

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/MC/MCStreamer.h`, `llvm/MC/MCSubtargetInfo.h`, `llvm/MC/MCSymbol.h`, `llvm/MC/TargetRegistry.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/MC/MCStreamer.h`, `llvm/MC/MCSubtargetInfo.h`, `llvm/MC/MCSymbol.h`, `llvm/MC/TargetRegistry.h`。

### Lines 37-58
```cpp
class XtensaAsmParser : public MCTargetAsmParser {
  const MCRegisterInfo &MRI;

  enum XtensaRegisterType { Xtensa_Generic, Xtensa_SR, Xtensa_UR };
  SMLoc getLoc() const { return getParser().getTok().getLoc(); }

  XtensaTargetStreamer &getTargetStreamer() {
    MCTargetStreamer &TS = *getParser().getStreamer().getTargetStreamer();
    return static_cast<XtensaTargetStreamer &>(TS);
  }

  ParseStatus parseDirective(AsmToken DirectiveID) override;
  bool parseRegister(MCRegister &Reg, SMLoc &StartLoc, SMLoc &EndLoc) override;
  bool parseInstruction(ParseInstructionInfo &Info, StringRef Name,
                        SMLoc NameLoc, OperandVector &Operands) override;
  bool matchAndEmitInstruction(SMLoc IDLoc, unsigned &Opcode,
                               OperandVector &Operands, MCStreamer &Out,
                               uint64_t &ErrorInfo,
                               bool MatchingInlineAsm) override;
  unsigned validateTargetOperandClass(MCParsedAsmOperand &Op,
                                      unsigned Kind) override;

```
- **EN**: Introduces declarations for `XtensaAsmParser`, `XtensaRegisterType`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `XtensaAsmParser`, `XtensaRegisterType` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 59-82
```cpp
  bool processInstruction(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out,
                          const MCSubtargetInfo *STI);

// Auto-generated instruction matching functions
#define GET_ASSEMBLER_HEADER
#include "XtensaGenAsmMatcher.inc"

  ParseStatus parseImmediate(OperandVector &Operands);
  ParseStatus
  parseRegister(OperandVector &Operands, bool AllowParens = false,
                XtensaRegisterType SR = Xtensa_Generic,
                Xtensa::RegisterAccessType RAType = Xtensa::REGISTER_EXCHANGE);
  ParseStatus parseOperandWithModifier(OperandVector &Operands);
  bool
  parseOperand(OperandVector &Operands, StringRef Mnemonic,
               XtensaRegisterType SR = Xtensa_Generic,
               Xtensa::RegisterAccessType RAType = Xtensa::REGISTER_EXCHANGE);
  bool ParseInstructionWithSR(ParseInstructionInfo &Info, StringRef Name,
                              SMLoc NameLoc, OperandVector &Operands);
  ParseStatus tryParseRegister(MCRegister &Reg, SMLoc &StartLoc,
                               SMLoc &EndLoc) override {
    return ParseStatus::NoMatch;
  }

```
- **EN**: Pulls in the headers needed for this implementation, including `XtensaGenAsmMatcher.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `XtensaGenAsmMatcher.inc`。

### Lines 83-100
```cpp
  ParseStatus parsePCRelTarget(OperandVector &Operands);
  bool parseLiteralDirective(SMLoc L);

public:
  enum XtensaMatchResultTy {
    Match_Dummy = FIRST_TARGET_MATCH_RESULT_TY,
#define GET_OPERAND_DIAGNOSTIC_TYPES
#include "XtensaGenAsmMatcher.inc"
#undef GET_OPERAND_DIAGNOSTIC_TYPES
  };

  XtensaAsmParser(const MCSubtargetInfo &STI, MCAsmParser &Parser,
                  const MCInstrInfo &MII)
      : MCTargetAsmParser(STI, MII),
        MRI(*Parser.getContext().getRegisterInfo()) {
    setAvailableFeatures(ComputeAvailableFeatures(STI.getFeatureBits()));
  }

```
- **EN**: Pulls in the headers needed for this implementation, including `XtensaGenAsmMatcher.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `XtensaGenAsmMatcher.inc`。

### Lines 101-114
```cpp
  bool hasWindowed() const {
    return getSTI().getFeatureBits()[Xtensa::FeatureWindowed];
  };
};

// Return true if Expr is in the range [MinValue, MaxValue].
static bool inRange(const MCExpr *Expr, int64_t MinValue, int64_t MaxValue) {
  if (auto *CE = dyn_cast<MCConstantExpr>(Expr)) {
    int64_t Value = CE->getValue();
    return Value >= MinValue && Value <= MaxValue;
  }
  return false;
}

```
- **EN**: Implements logic around `hasWindowed`, `getSTI`, `inRange`, `getValue`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `hasWindowed`, `getSTI`, `inRange`, `getValue` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 115-126
```cpp
struct XtensaOperand : public MCParsedAsmOperand {

  enum KindTy {
    Token,
    Register,
    Immediate,
  } Kind;

  struct RegOp {
    unsigned RegNum;
  };

```
- **EN**: Introduces declarations for `XtensaOperand`, `KindTy`, `RegOp`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `XtensaOperand`, `KindTy`, `RegOp` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 127-139
```cpp
  struct ImmOp {
    const MCExpr *Val;
  };

  SMLoc StartLoc, EndLoc;
  union {
    StringRef Tok;
    RegOp Reg;
    ImmOp Imm;
  };

  XtensaOperand(KindTy K) : MCParsedAsmOperand(), Kind(K) {}

```
- **EN**: Introduces declarations for `ImmOp`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `ImmOp` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 140-157
```cpp
public:
  XtensaOperand(const XtensaOperand &o) : MCParsedAsmOperand() {
    Kind = o.Kind;
    StartLoc = o.StartLoc;
    EndLoc = o.EndLoc;
    switch (Kind) {
    case Register:
      Reg = o.Reg;
      break;
    case Immediate:
      Imm = o.Imm;
      break;
    case Token:
      Tok = o.Tok;
      break;
    }
  }

```
- **EN**: Implements logic around `XtensaOperand`; this block uses `switch`-based dispatch.
- **CN**: 围绕 `XtensaOperand` 实现具体逻辑；这一段使用 `switch` 分派。

### Lines 158-173
```cpp
  bool isToken() const override { return Kind == Token; }
  bool isReg() const override { return Kind == Register; }
  bool isImm() const override { return Kind == Immediate; }
  bool isMem() const override { return false; }

  bool isImm(int64_t MinValue, int64_t MaxValue) const {
    return Kind == Immediate && inRange(getImm(), MinValue, MaxValue);
  }

  bool isImm8() const { return isImm(-128, 127); }

  bool isImm8_sh8() const {
    return isImm(-32768, 32512) &&
           ((cast<MCConstantExpr>(getImm())->getValue() & 0xFF) == 0);
  }

```
- **EN**: Implements logic around `isToken`, `isReg`, `isImm`, `isMem`, ...; this block returns target-specific results.
- **CN**: 围绕 `isToken`, `isReg`, `isImm`, `isMem`, ... 实现具体逻辑；这一段返回目标相关结果。

### Lines 174-185
```cpp
  bool isImm12() const { return isImm(-2048, 2047); }

  // Convert MOVI to literal load, when immediate is not in range (-2048, 2047)
  bool isImm12m() const { return Kind == Immediate; }

  bool isOffset4m32() const {
    return isImm(0, 60) &&
           ((cast<MCConstantExpr>(getImm())->getValue() & 0x3) == 0);
  }

  bool isOffset8m8() const { return isImm(0, 255); }

```
- **EN**: Implements logic around `isImm12`, `isImm12m`, `isOffset4m32`, `isImm`, ...; this block returns target-specific results.
- **CN**: 围绕 `isImm12`, `isImm12m`, `isOffset4m32`, `isImm`, ... 实现具体逻辑；这一段返回目标相关结果。

### Lines 186-200
```cpp
  bool isOffset8m16() const {
    return isImm(0, 510) &&
           ((cast<MCConstantExpr>(getImm())->getValue() & 0x1) == 0);
  }

  bool isOffset8m32() const {
    return isImm(0, 1020) &&
           ((cast<MCConstantExpr>(getImm())->getValue() & 0x3) == 0);
  }

  bool isentry_imm12() const {
    return isImm(0, 32760) &&
           ((cast<MCConstantExpr>(getImm())->getValue() % 8) == 0);
  }

```
- **EN**: Implements logic around `isOffset8m16`, `isImm`, `cast<MCConstantExpr>`, `isOffset8m32`, ...; this block returns target-specific results.
- **CN**: 围绕 `isOffset8m16`, `isImm`, `cast<MCConstantExpr>`, `isOffset8m32`, ... 实现具体逻辑；这一段返回目标相关结果。

### Lines 201-212
```cpp
  bool isUimm4() const { return isImm(0, 15); }

  bool isUimm5() const { return isImm(0, 31); }

  bool isImm8n_7() const { return isImm(-8, 7); }

  bool isShimm1_31() const { return isImm(1, 31); }

  bool isImm16_31() const { return isImm(16, 31); }

  bool isImm1_16() const { return isImm(1, 16); }

```
- **EN**: Implements logic around `isUimm4`, `isUimm5`, `isImm8n_7`, `isShimm1_31`, ...; this block returns target-specific results.
- **CN**: 围绕 `isUimm4`, `isUimm5`, `isImm8n_7`, `isShimm1_31`, ... 实现具体逻辑；这一段返回目标相关结果。

### Lines 213-236
```cpp
  // Check that value is either equals (-1) or from [1,15] range.
  bool isImm1n_15() const { return isImm(1, 15) || isImm(-1, -1); }

  bool isImm32n_95() const { return isImm(-32, 95); }

  bool isImm64n_4n() const {
    return isImm(-64, -4) &&
           ((cast<MCConstantExpr>(getImm())->getValue() & 0x3) == 0);
  }

  bool isB4const() const {
    if (Kind != Immediate)
      return false;
    if (auto *CE = dyn_cast<MCConstantExpr>(getImm())) {
      int64_t Value = CE->getValue();
      switch (Value) {
      case -1:
      case 1:
      case 2:
      case 3:
      case 4:
      case 5:
      case 6:
      case 7:
```
- **EN**: Implements logic around `isImm1n_15`, `isImm32n_95`, `isImm64n_4n`, `isImm`, ...; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `isImm1n_15`, `isImm32n_95`, `isImm64n_4n`, `isImm`, ... 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果。

### Lines 237-252
```cpp
      case 8:
      case 10:
      case 12:
      case 16:
      case 32:
      case 64:
      case 128:
      case 256:
        return true;
      default:
        return false;
      }
    }
    return false;
  }

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 253-276
```cpp
  bool isB4constu() const {
    if (Kind != Immediate)
      return false;
    if (auto *CE = dyn_cast<MCConstantExpr>(getImm())) {
      int64_t Value = CE->getValue();
      switch (Value) {
      case 32768:
      case 65536:
      case 2:
      case 3:
      case 4:
      case 5:
      case 6:
      case 7:
      case 8:
      case 10:
      case 12:
      case 16:
      case 32:
      case 64:
      case 128:
      case 256:
        return true;
      default:
```
- **EN**: Implements logic around `isB4constu`, `getValue`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `isB4constu`, `getValue` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果。

### Lines 277-289
```cpp
        return false;
      }
    }
    return false;
  }

  bool isimm7_22() const { return isImm(7, 22); }

  /// getStartLoc - Gets location of the first token of this operand
  SMLoc getStartLoc() const override { return StartLoc; }
  /// getEndLoc - Gets location of the last token of this operand
  SMLoc getEndLoc() const override { return EndLoc; }

```
- **EN**: Implements logic around `isimm7_22`, `getStartLoc`, `getEndLoc`; this block returns target-specific results.
- **CN**: 围绕 `isimm7_22`, `getStartLoc`, `getEndLoc` 实现具体逻辑；这一段返回目标相关结果。

### Lines 290-304
```cpp
  MCRegister getReg() const override {
    assert(Kind == Register && "Invalid type access!");
    return Reg.RegNum;
  }

  const MCExpr *getImm() const {
    assert(Kind == Immediate && "Invalid type access!");
    return Imm.Val;
  }

  StringRef getToken() const {
    assert(Kind == Token && "Invalid type access!");
    return Tok;
  }

```
- **EN**: Implements logic around `getReg`, `assert`, `getImm`, `getToken`; this block returns target-specific results.
- **CN**: 围绕 `getReg`, `assert`, `getImm`, `getToken` 实现具体逻辑；这一段返回目标相关结果。

### Lines 305-319
```cpp
  void print(raw_ostream &OS, const MCAsmInfo &MAI) const override {
    switch (Kind) {
    case Immediate:
      MAI.printExpr(OS, *getImm());
      break;
    case Register:
      OS << "<register x";
      OS << getReg() << ">";
      break;
    case Token:
      OS << "'" << getToken() << "'";
      break;
    }
  }

```
- **EN**: Implements logic around `print`, `printExpr`, `getReg`, `getToken`; this block uses `switch`-based dispatch.
- **CN**: 围绕 `print`, `printExpr`, `getReg`, `getToken` 实现具体逻辑；这一段使用 `switch` 分派。

### Lines 320-336
```cpp
  static std::unique_ptr<XtensaOperand> createToken(StringRef Str, SMLoc S) {
    auto Op = std::make_unique<XtensaOperand>(Token);
    Op->Tok = Str;
    Op->StartLoc = S;
    Op->EndLoc = S;
    return Op;
  }

  static std::unique_ptr<XtensaOperand> createReg(unsigned RegNo, SMLoc S,
                                                  SMLoc E) {
    auto Op = std::make_unique<XtensaOperand>(Register);
    Op->Reg.RegNum = RegNo;
    Op->StartLoc = S;
    Op->EndLoc = E;
    return Op;
  }

```
- **EN**: Implements logic around `createToken`, `make_unique<XtensaOperand>`, `createReg`; this block returns target-specific results.
- **CN**: 围绕 `createToken`, `make_unique<XtensaOperand>`, `createReg` 实现具体逻辑；这一段返回目标相关结果。

### Lines 337-350
```cpp
  static std::unique_ptr<XtensaOperand> createImm(const MCExpr *Val, SMLoc S,
                                                  SMLoc E) {
    auto Op = std::make_unique<XtensaOperand>(Immediate);
    Op->Imm.Val = Val;
    Op->StartLoc = S;
    Op->EndLoc = E;
    return Op;
  }

  void addExpr(MCInst &Inst, const MCExpr *Expr) const {
    assert(Expr && "Expr shouldn't be null!");
    int64_t Imm = 0;
    bool IsConstant = false;

```
- **EN**: Implements logic around `createImm`, `make_unique<XtensaOperand>`, `addExpr`, `assert`; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `createImm`, `make_unique<XtensaOperand>`, `addExpr`, `assert` 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 351-367
```cpp
    if (auto *CE = dyn_cast<MCConstantExpr>(Expr)) {
      IsConstant = true;
      Imm = CE->getValue();
    }

    if (IsConstant)
      Inst.addOperand(MCOperand::createImm(Imm));
    else
      Inst.addOperand(MCOperand::createExpr(Expr));
  }

  // Used by the TableGen Code
  void addRegOperands(MCInst &Inst, unsigned N) const {
    assert(N == 1 && "Invalid number of operands!");
    Inst.addOperand(MCOperand::createReg(getReg()));
  }

```
- **EN**: Implements logic around `getValue`, `addOperand`, `addRegOperands`, `assert`; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `getValue`, `addOperand`, `addRegOperands`, `assert` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 368-382
```cpp
  void addImmOperands(MCInst &Inst, unsigned N) const {
    assert(N == 1 && "Invalid number of operands!");
    addExpr(Inst, getImm());
  }
};

#define GET_REGISTER_MATCHER
#define GET_MATCHER_IMPLEMENTATION
#include "XtensaGenAsmMatcher.inc"

unsigned XtensaAsmParser::validateTargetOperandClass(MCParsedAsmOperand &AsmOp,
                                                     unsigned Kind) {
  return Match_InvalidOperand;
}

```
- **EN**: Pulls in the headers needed for this implementation, including `XtensaGenAsmMatcher.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `XtensaGenAsmMatcher.inc`。

### Lines 383-406
```cpp
static SMLoc RefineErrorLoc(const SMLoc Loc, const OperandVector &Operands,
                            uint64_t ErrorInfo) {
  if (ErrorInfo != ~0ULL && ErrorInfo < Operands.size()) {
    SMLoc ErrorLoc = Operands[ErrorInfo]->getStartLoc();
    if (ErrorLoc == SMLoc())
      return Loc;
    return ErrorLoc;
  }
  return Loc;
}

bool XtensaAsmParser::processInstruction(MCInst &Inst, SMLoc IDLoc,
                                         MCStreamer &Out,
                                         const MCSubtargetInfo *STI) {
  Inst.setLoc(IDLoc);
  const unsigned Opcode = Inst.getOpcode();
  switch (Opcode) {
  case Xtensa::L32R: {
    const MCSymbolRefExpr *OpExpr =
        static_cast<const MCSymbolRefExpr *>(Inst.getOperand(1).getExpr());
    Inst.getOperand(1).setExpr(OpExpr);
    break;
  }
  case Xtensa::MOVI: {
```
- **EN**: Implements logic around `RefineErrorLoc`, `getStartLoc`, `processInstruction`, `setLoc`, ...; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results; works at the MC layer.
- **CN**: 围绕 `RefineErrorLoc`, `getStartLoc`, `processInstruction`, `setLoc`, ... 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果，工作在 MC 层。

### Lines 407-430
```cpp
    XtensaTargetStreamer &TS = this->getTargetStreamer();

    // Expand MOVI operand
    if (!Inst.getOperand(1).isExpr()) {
      uint64_t ImmOp64 = Inst.getOperand(1).getImm();
      int32_t Imm = ImmOp64;
      if (!isInt<12>(Imm)) {
        XtensaTargetStreamer &TS = this->getTargetStreamer();
        MCInst TmpInst;
        TmpInst.setLoc(IDLoc);
        TmpInst.setOpcode(Xtensa::L32R);
        const MCExpr *Value = MCConstantExpr::create(ImmOp64, getContext());
        MCSymbol *Sym = getContext().createTempSymbol();
        const MCExpr *Expr = MCSymbolRefExpr::create(Sym, getContext());
        TmpInst.addOperand(Inst.getOperand(0));
        MCOperand Op1 = MCOperand::createExpr(Expr);
        TmpInst.addOperand(Op1);
        TS.emitLiteral(Sym, Value, true, IDLoc);
        Inst = TmpInst;
      }
    } else {
      MCInst TmpInst;
      TmpInst.setLoc(IDLoc);
      TmpInst.setOpcode(Xtensa::L32R);
```
- **EN**: Implements logic around `getTargetStreamer`, `getOperand`, `setLoc`, `setOpcode`, ...; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `getTargetStreamer`, `getOperand`, `setLoc`, `setOpcode`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 431-445
```cpp
      const MCExpr *Value = Inst.getOperand(1).getExpr();
      MCSymbol *Sym = getContext().createTempSymbol();
      const MCExpr *Expr = MCSymbolRefExpr::create(Sym, getContext());
      TmpInst.addOperand(Inst.getOperand(0));
      MCOperand Op1 = MCOperand::createExpr(Expr);
      TmpInst.addOperand(Op1);
      Inst = TmpInst;
      TS.emitLiteral(Sym, Value, true, IDLoc);
    }
    break;
  }
  default:
    break;
  }

```
- **EN**: Implements logic around `getOperand`, `getContext`, `create`, `addOperand`, ....
- **CN**: 围绕 `getOperand`, `getContext`, `create`, `addOperand`, ... 实现具体逻辑。

### Lines 446-457
```cpp
  return true;
}

bool XtensaAsmParser::matchAndEmitInstruction(SMLoc IDLoc, unsigned &Opcode,
                                              OperandVector &Operands,
                                              MCStreamer &Out,
                                              uint64_t &ErrorInfo,
                                              bool MatchingInlineAsm) {
  MCInst Inst;
  auto Result =
      MatchInstructionImpl(Operands, Inst, ErrorInfo, MatchingInlineAsm);

```
- **EN**: Implements logic around `matchAndEmitInstruction`, `MatchInstructionImpl`; this block returns target-specific results; works at the MC layer.
- **CN**: 围绕 `matchAndEmitInstruction`, `MatchInstructionImpl` 实现具体逻辑；这一段返回目标相关结果，工作在 MC 层。

### Lines 458-475
```cpp
  switch (Result) {
  default:
    break;
  case Match_Success:
    processInstruction(Inst, IDLoc, Out, STI);
    Inst.setLoc(IDLoc);
    Out.emitInstruction(Inst, getSTI());
    return false;
  case Match_MissingFeature:
    return Error(IDLoc, "instruction use requires an option to be enabled");
  case Match_MnemonicFail:
    return Error(IDLoc, "unrecognized instruction mnemonic");
  case Match_InvalidOperand: {
    SMLoc ErrorLoc = IDLoc;
    if (ErrorInfo != ~0U) {
      if (ErrorInfo >= Operands.size())
        return Error(ErrorLoc, "too few operands for instruction");

```
- **EN**: Implements logic around `processInstruction`, `setLoc`, `emitInstruction`, `Error`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `processInstruction`, `setLoc`, `emitInstruction`, `Error` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果。

### Lines 476-499
```cpp
      ErrorLoc = ((XtensaOperand &)*Operands[ErrorInfo]).getStartLoc();
      if (ErrorLoc == SMLoc())
        ErrorLoc = IDLoc;
    }
    return Error(ErrorLoc, "invalid operand for instruction");
  }
  case Match_InvalidImm8:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected immediate in range [-128, 127]");
  case Match_InvalidImm8_sh8:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected immediate in range [-32768, 32512], first 8 bits "
                 "should be zero");
  case Match_InvalidB4const:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected b4const immediate");
  case Match_InvalidB4constu:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected b4constu immediate");
  case Match_InvalidImm12:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected immediate in range [-2048, 2047]");
  case Match_InvalidImm12m:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
```
- **EN**: Implements logic around `getStartLoc`, `Error`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getStartLoc`, `Error` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 500-523
```cpp
                 "expected immediate in range [-2048, 2047]");
  case Match_InvalidImm1_16:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected immediate in range [1, 16]");
  case Match_InvalidImm1n_15:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected immediate in range [-1, 15] except 0");
  case Match_InvalidImm32n_95:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected immediate in range [-32, 95]");
  case Match_InvalidImm64n_4n:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected immediate in range [-64, -4]");
  case Match_InvalidImm8n_7:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected immediate in range [-8, 7]");
  case Match_InvalidShimm1_31:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected immediate in range [1, 31]");
  case Match_InvalidUimm4:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected immediate in range [0, 15]");
  case Match_InvalidUimm5:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
```
- **EN**: Implements logic around `Error`; this block returns target-specific results.
- **CN**: 围绕 `Error` 实现具体逻辑；这一段返回目标相关结果。

### Lines 524-547
```cpp
                 "expected immediate in range [0, 31]");
  case Match_InvalidOffset8m8:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected immediate in range [0, 255]");
  case Match_InvalidOffset8m16:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected immediate in range [0, 510], first bit "
                 "should be zero");
  case Match_InvalidOffset8m32:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected immediate in range [0, 1020], first 2 bits "
                 "should be zero");
  case Match_InvalidOffset4m32:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected immediate in range [0, 60], first 2 bits "
                 "should be zero");
  case Match_Invalidentry_imm12:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected immediate in range [0, 32760], first 3 bits "
                 "should be zero");
  case Match_Invalidimm7_22:
    return Error(RefineErrorLoc(IDLoc, Operands, ErrorInfo),
                 "expected immediate in range [7, 22]");
  }
```
- **EN**: Implements logic around `Error`; this block returns target-specific results.
- **CN**: 围绕 `Error` 实现具体逻辑；这一段返回目标相关结果。

### Lines 548-564
```cpp

  report_fatal_error("Unknown match type detected!");
}

ParseStatus XtensaAsmParser::parsePCRelTarget(OperandVector &Operands) {
  MCAsmParser &Parser = getParser();
  LLVM_DEBUG(dbgs() << "parsePCRelTarget\n");

  SMLoc S = getLexer().getLoc();

  // Expressions are acceptable
  const MCExpr *Expr = nullptr;
  if (Parser.parseExpression(Expr)) {
    // We have no way of knowing if a symbol was consumed so we must ParseFail
    return ParseStatus::Failure;
  }

```
- **EN**: Implements logic around `report_fatal_error`, `parsePCRelTarget`, `getParser`, `getLexer`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `report_fatal_error`, `parsePCRelTarget`, `getParser`, `getLexer` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 565-580
```cpp
  // Currently not support constants
  if (Expr->getKind() == MCExpr::ExprKind::Constant)
    return Error(getLoc(), "unknown operand");

  Operands.push_back(XtensaOperand::createImm(Expr, S, getLexer().getLoc()));
  return ParseStatus::Success;
}

bool XtensaAsmParser::parseRegister(MCRegister &Reg, SMLoc &StartLoc,
                                    SMLoc &EndLoc) {
  const AsmToken &Tok = getParser().getTok();
  StartLoc = Tok.getLoc();
  EndLoc = Tok.getEndLoc();
  Reg = Xtensa::NoRegister;
  StringRef Name = getLexer().getTok().getIdentifier();

```
- **EN**: Implements logic around `Error`, `push_back`, `parseRegister`, `getParser`, ...; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `Error`, `push_back`, `parseRegister`, `getParser`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 581-597
```cpp
  if (!MatchRegisterName(Name) && !MatchRegisterAltName(Name)) {
    getParser().Lex(); // Eat identifier token.
    return false;
  }

  return Error(StartLoc, "invalid register name");
}

ParseStatus XtensaAsmParser::parseRegister(OperandVector &Operands,
                                           bool AllowParens,
                                           XtensaRegisterType RegType,
                                           Xtensa::RegisterAccessType RAType) {
  SMLoc FirstS = getLoc();
  bool HadParens = false;
  AsmToken Buf[2];
  StringRef RegName;

```
- **EN**: Implements logic around `getParser`, `Error`, `parseRegister`, `getLoc`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getParser`, `Error`, `parseRegister`, `getLoc` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 598-610
```cpp
  // If this a parenthesised register name is allowed, parse it atomically
  if (AllowParens && getLexer().is(AsmToken::LParen)) {
    size_t ReadCount = getLexer().peekTokens(Buf);
    if (ReadCount == 2 && Buf[1].getKind() == AsmToken::RParen) {
      if (Buf[0].getKind() == AsmToken::Integer && RegType == Xtensa_Generic)
        return ParseStatus::NoMatch;
      HadParens = true;
      getParser().Lex(); // Eat '('
    }
  }

  MCRegister RegNo = 0;

```
- **EN**: Implements logic around `getLexer`, `getParser`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getLexer`, `getParser` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 611-634
```cpp
  switch (getLexer().getKind()) {
  default:
    return ParseStatus::NoMatch;
  case AsmToken::Integer:
    if (RegType == Xtensa_Generic)
      return ParseStatus::NoMatch;

    // Parse case when we expect UR register code as special case,
    // because SR and UR registers may have the same number
    // and such situation may lead to confilct
    if (RegType == Xtensa_UR) {
      int64_t RegCode = getLexer().getTok().getIntVal();
      RegNo = Xtensa::getUserRegister(RegCode, MRI);
    } else {
      RegName = getLexer().getTok().getString();
      RegNo = MatchRegisterAltName(RegName);
    }
    break;
  case AsmToken::Identifier:
    RegName = getLexer().getTok().getIdentifier();
    RegNo = MatchRegisterName(RegName);
    if (RegNo == 0)
      RegNo = MatchRegisterAltName(RegName);
    break;
```
- **EN**: Implements logic around `getLexer`, `getUserRegister`, `MatchRegisterAltName`, `MatchRegisterName`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getLexer`, `getUserRegister`, `MatchRegisterAltName`, `MatchRegisterName` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果。

### Lines 635-652
```cpp
  }

  if (RegNo == 0) {
    if (HadParens)
      getLexer().UnLex(Buf[0]);
    return ParseStatus::NoMatch;
  }

  if (!Xtensa::checkRegister(RegNo, getSTI().getFeatureBits(), RAType))
    return ParseStatus::NoMatch;

  if (HadParens)
    Operands.push_back(XtensaOperand::createToken("(", FirstS));
  SMLoc S = getLoc();
  SMLoc E = getParser().getTok().getEndLoc();
  getLexer().Lex();
  Operands.push_back(XtensaOperand::createReg(RegNo, S, E));

```
- **EN**: Implements logic around `getLexer`, `push_back`, `getLoc`, `getParser`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getLexer`, `push_back`, `getLoc`, `getParser` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 653-665
```cpp
  if (HadParens) {
    getParser().Lex(); // Eat ')'
    Operands.push_back(XtensaOperand::createToken(")", getLoc()));
  }

  return ParseStatus::Success;
}

ParseStatus XtensaAsmParser::parseImmediate(OperandVector &Operands) {
  SMLoc S = getLoc();
  SMLoc E;
  const MCExpr *Res;

```
- **EN**: Implements logic around `getParser`, `push_back`, `parseImmediate`, `getLoc`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getParser`, `push_back`, `parseImmediate`, `getLoc` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 666-682
```cpp
  switch (getLexer().getKind()) {
  default:
    return ParseStatus::NoMatch;
  case AsmToken::LParen:
  case AsmToken::Minus:
  case AsmToken::Plus:
  case AsmToken::Tilde:
  case AsmToken::Integer:
  case AsmToken::String:
    if (getParser().parseExpression(Res))
      return ParseStatus::Failure;
    break;
  case AsmToken::Identifier: {
    StringRef Identifier;
    if (getParser().parseIdentifier(Identifier))
      return ParseStatus::Failure;

```
- **EN**: Implements target-specific case analysis using a `switch` over opcodes, fixups, or enum values.
- **CN**: 通过对 opcode、fixup 或枚举值执行 `switch` 分析来实现目标相关逻辑。

### Lines 683-695
```cpp
    MCSymbol *Sym = getContext().getOrCreateSymbol(Identifier);
    Res = MCSymbolRefExpr::create(Sym, getContext());
    break;
  }
  case AsmToken::Percent:
    return parseOperandWithModifier(Operands);
  }

  E = SMLoc::getFromPointer(S.getPointer() - 1);
  Operands.push_back(XtensaOperand::createImm(Res, S, E));
  return ParseStatus::Success;
}

```
- **EN**: Implements logic around `getContext`, `create`, `parseOperandWithModifier`, `getFromPointer`, ...; this block returns target-specific results.
- **CN**: 围绕 `getContext`, `create`, `parseOperandWithModifier`, `getFromPointer`, ... 实现具体逻辑；这一段返回目标相关结果。

### Lines 696-711
```cpp
ParseStatus XtensaAsmParser::parseOperandWithModifier(OperandVector &Operands) {
  return ParseStatus::Failure;
}

/// Looks at a token type and creates the relevant operand
/// from this information, adding to Operands.
/// If operand was parsed, returns false, else true.
bool XtensaAsmParser::parseOperand(OperandVector &Operands, StringRef Mnemonic,
                                   XtensaRegisterType RegType,
                                   Xtensa::RegisterAccessType RAType) {
  // Check if the current operand has a custom associated parser, if so, try to
  // custom parse the operand, or fallback to the general approach.
  ParseStatus Res = MatchOperandParserImpl(Operands, Mnemonic);
  if (Res.isSuccess())
    return false;

```
- **EN**: Implements logic around `parseOperandWithModifier`, `parseOperand`, `MatchOperandParserImpl`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `parseOperandWithModifier`, `parseOperand`, `MatchOperandParserImpl` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 712-725
```cpp
  // If there wasn't a custom match, try the generic matcher below. Otherwise,
  // there was a match, but an error occurred, in which case, just return that
  // the operand parsing failed.
  if (Res.isFailure())
    return true;

  // Attempt to parse token as register
  if (parseRegister(Operands, true, RegType, RAType).isSuccess())
    return false;

  // Attempt to parse token as an immediate
  if (parseImmediate(Operands).isSuccess())
    return false;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 726-737
```cpp
  // Finally we have exhausted all options and must declare defeat.
  return Error(getLoc(), "unknown operand");
}

bool XtensaAsmParser::ParseInstructionWithSR(ParseInstructionInfo &Info,
                                             StringRef Name, SMLoc NameLoc,
                                             OperandVector &Operands) {
  Xtensa::RegisterAccessType RAType =
      Name[0] == 'w' ? Xtensa::REGISTER_WRITE
                     : (Name[0] == 'r' ? Xtensa::REGISTER_READ
                                       : Xtensa::REGISTER_EXCHANGE);

```
- **EN**: Implements logic around `Error`, `ParseInstructionWithSR`; this block returns target-specific results.
- **CN**: 围绕 `Error`, `ParseInstructionWithSR` 实现具体逻辑；这一段返回目标相关结果。

### Lines 738-750
```cpp
  if ((Name.size() > 4) && Name[3] == '.') {
    // Parse case when instruction name is concatenated with SR/UR register
    // name, like "wsr.sar a1" or "wur.fcr a1"

    // First operand is token for instruction
    Operands.push_back(XtensaOperand::createToken(Name.take_front(3), NameLoc));

    StringRef RegName = Name.drop_front(4);
    unsigned RegNo = MatchRegisterName(RegName);

    if (RegNo == 0)
      RegNo = MatchRegisterAltName(RegName);

```
- **EN**: Implements logic around `push_back`, `drop_front`, `MatchRegisterName`, `MatchRegisterAltName`; this block applies conditional target rules.
- **CN**: 围绕 `push_back`, `drop_front`, `MatchRegisterName`, `MatchRegisterAltName` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 751-764
```cpp
    if (!Xtensa::checkRegister(RegNo, getSTI().getFeatureBits(), RAType))
      return Error(NameLoc, "invalid register name");

    // Parse operand
    if (parseOperand(Operands, Name))
      return true;

    SMLoc S = getLoc();
    SMLoc E = SMLoc::getFromPointer(S.getPointer() - 1);
    Operands.push_back(XtensaOperand::createReg(RegNo, S, E));
  } else {
    // First operand is token for instruction
    Operands.push_back(XtensaOperand::createToken(Name, NameLoc));

```
- **EN**: Implements logic around `Error`, `getLoc`, `getFromPointer`, `push_back`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `Error`, `getLoc`, `getFromPointer`, `push_back` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 765-780
```cpp
    // Parse first operand
    if (parseOperand(Operands, Name))
      return true;

    if (!parseOptionalToken(AsmToken::Comma)) {
      SMLoc Loc = getLexer().getLoc();
      getParser().eatToEndOfStatement();
      return Error(Loc, "unexpected token");
    }

    // Parse second operand
    if (parseOperand(Operands, Name, Name[1] == 's' ? Xtensa_SR : Xtensa_UR,
                     RAType))
      return true;
  }

```
- **EN**: Implements logic around `getLexer`, `getParser`, `Error`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getLexer`, `getParser`, `Error` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 781-799
```cpp
  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    SMLoc Loc = getLexer().getLoc();
    getParser().eatToEndOfStatement();
    return Error(Loc, "unexpected token");
  }

  getParser().Lex(); // Consume the EndOfStatement.
  return false;
}

bool XtensaAsmParser::parseInstruction(ParseInstructionInfo &Info,
                                       StringRef Name, SMLoc NameLoc,
                                       OperandVector &Operands) {
  if (Name.starts_with("wsr") || Name.starts_with("rsr") ||
      Name.starts_with("xsr") || Name.starts_with("rur") ||
      Name.starts_with("wur")) {
    return ParseInstructionWithSR(Info, Name, NameLoc, Operands);
  }

```
- **EN**: Implements logic around `getLexer`, `getParser`, `Error`, `parseInstruction`, ...; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getLexer`, `getParser`, `Error`, `parseInstruction`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 800-815
```cpp
  // First operand is token for instruction
  Operands.push_back(XtensaOperand::createToken(Name, NameLoc));

  // If there are no more operands, then finish
  if (getLexer().is(AsmToken::EndOfStatement))
    return false;

  // Parse first operand
  if (parseOperand(Operands, Name))
    return true;

  // Parse until end of statement, consuming commas between operands
  while (parseOptionalToken(AsmToken::Comma))
    if (parseOperand(Operands, Name))
      return true;

```
- **EN**: Implements logic around `push_back`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `push_back` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 816-831
```cpp
  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    SMLoc Loc = getLexer().getLoc();
    getParser().eatToEndOfStatement();
    return Error(Loc, "unexpected token");
  }

  getParser().Lex(); // Consume the EndOfStatement.
  return false;
}

bool XtensaAsmParser::parseLiteralDirective(SMLoc L) {
  MCAsmParser &Parser = getParser();
  const MCExpr *Value;
  SMLoc LiteralLoc = getLexer().getLoc();
  XtensaTargetStreamer &TS = this->getTargetStreamer();

```
- **EN**: Implements logic around `getLexer`, `getParser`, `Error`, `parseLiteralDirective`, ...; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getLexer`, `getParser`, `Error`, `parseLiteralDirective`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 832-846
```cpp
  if (Parser.parseExpression(Value))
    return true;

  const MCSymbolRefExpr *SE = dyn_cast<MCSymbolRefExpr>(Value);

  if (!SE)
    return Error(LiteralLoc, "literal label must be a symbol");

  if (Parser.parseComma())
    return true;

  SMLoc OpcodeLoc = getLexer().getLoc();
  if (parseOptionalToken(AsmToken::EndOfStatement))
    return Error(OpcodeLoc, "expected value");

```
- **EN**: Implements logic around `dyn_cast<MCSymbolRefExpr>`, `Error`, `getLexer`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `dyn_cast<MCSymbolRefExpr>`, `Error`, `getLexer` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 847-859
```cpp
  if (Parser.parseExpression(Value))
    return true;

  if (parseEOL())
    return true;

  MCSymbol *Sym = getContext().getOrCreateSymbol(SE->getSymbol().getName());

  TS.emitLiteral(Sym, Value, true, LiteralLoc);

  return false;
}

```
- **EN**: Implements logic around `getContext`, `emitLiteral`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getContext`, `emitLiteral` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 860-873
```cpp
ParseStatus XtensaAsmParser::parseDirective(AsmToken DirectiveID) {
  StringRef IDVal = DirectiveID.getString();
  SMLoc Loc = getLexer().getLoc();

  if (IDVal == ".literal_position") {
    XtensaTargetStreamer &TS = this->getTargetStreamer();
    TS.emitLiteralPosition();
    return parseEOL();
  }

  if (IDVal == ".literal") {
    return parseLiteralDirective(Loc);
  }

```
- **EN**: Implements logic around `parseDirective`, `getString`, `getLexer`, `getTargetStreamer`, ...; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `parseDirective`, `getString`, `getLexer`, `getTargetStreamer`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 874-880
```cpp
  return ParseStatus::NoMatch;
}

// Force static initialization.
extern "C" LLVM_EXTERNAL_VISIBILITY void LLVMInitializeXtensaAsmParser() {
  RegisterMCAsmParser<XtensaAsmParser> X(getTheXtensaTarget());
}
```
- **EN**: Implements logic around `X`; this block returns target-specific results.
- **CN**: 围绕 `X` 实现具体逻辑；这一段返回目标相关结果。

## Key Concepts / 关键概念

- **Assembly parsing / 汇编解析**:
  - **EN**: Parses textual assembly into operands and instructions
  - **CN**: 将文本汇编解析为操作数和指令

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `MCTargetDesc/XtensaMCAsmInfo.h`, `MCTargetDesc/XtensaMCTargetDesc.h`, `MCTargetDesc/XtensaTargetStreamer.h`, `TargetInfo/XtensaTargetInfo.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/StringSwitch.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCInst.h`, `llvm/MC/MCInstrInfo.h`, `llvm/MC/MCParser/AsmLexer.h`, `llvm/MC/MCParser/MCParsedAsmOperand.h` ... (+8 more)
- **LLVM subsystems / LLVM 子系统**: MC, Support
- **Generated macros / 生成宏**: `GET_ASSEMBLER_HEADER`, `GET_MATCHER_IMPLEMENTATION`, `GET_MATCH_RESULT_TY`, `GET_OPERAND_DIAGNOSTIC_TYPES`, `GET_REGISTER_MATCHER`
