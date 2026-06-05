# MSP430AsmParser.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/MSP430/AsmParser/MSP430AsmParser.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements the target-specific assembly parser that turns textual assembly into MC operands and MC instructions.
  - **CN**: 实现目标相关的汇编解析器，把文本汇编转换为 MC 操作数和 MC 指令。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```cpp
//===- MSP430AsmParser.cpp - Parse MSP430 assembly to MCInst instructions -===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "MCTargetDesc/MSP430MCAsmInfo.h"
#include "MCTargetDesc/MSP430MCTargetDesc.h"
#include "MSP430.h"
#include "MSP430RegisterInfo.h"
#include "TargetInfo/MSP430TargetInfo.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `MCTargetDesc/MSP430MCAsmInfo.h`, `MCTargetDesc/MSP430MCTargetDesc.h`, `MSP430.h`, `MSP430RegisterInfo.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `MCTargetDesc/MSP430MCAsmInfo.h`, `MCTargetDesc/MSP430MCTargetDesc.h`, `MSP430.h`, `MSP430RegisterInfo.h`。

### Lines 15-30
```cpp
#include "llvm/ADT/APInt.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCInstrInfo.h"
#include "llvm/MC/MCParser/AsmLexer.h"
#include "llvm/MC/MCParser/MCParsedAsmOperand.h"
#include "llvm/MC/MCParser/MCTargetAsmParser.h"
#include "llvm/MC/MCStreamer.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/MC/MCSymbol.h"
#include "llvm/MC/MCValue.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/Compiler.h"
#include "llvm/Support/Debug.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ADT/APInt.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCInst.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ADT/APInt.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCInst.h`。

### Lines 31-41
```cpp
#define DEBUG_TYPE "msp430-asm-parser"

using namespace llvm;

namespace {

/// Parses MSP430 assembly from a stream.
class MSP430AsmParser : public MCTargetAsmParser {
  MCAsmParser &Parser;
  const MCRegisterInfo *MRI;

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 42-50
```cpp
  bool matchAndEmitInstruction(SMLoc IDLoc, unsigned &Opcode,
                               OperandVector &Operands, MCStreamer &Out,
                               uint64_t &ErrorInfo,
                               bool MatchingInlineAsm) override;

  bool parseRegister(MCRegister &Reg, SMLoc &StartLoc, SMLoc &EndLoc) override;
  ParseStatus tryParseRegister(MCRegister &Reg, SMLoc &StartLoc,
                               SMLoc &EndLoc) override;

```
- **EN**: Implements logic around `matchAndEmitInstruction`, `parseRegister`, `tryParseRegister`.
- **CN**: 围绕 `matchAndEmitInstruction`, `parseRegister`, `tryParseRegister` 实现具体逻辑。

### Lines 51-59
```cpp
  bool parseInstruction(ParseInstructionInfo &Info, StringRef Name,
                        SMLoc NameLoc, OperandVector &Operands) override;

  ParseStatus parseDirective(AsmToken DirectiveID) override;
  bool ParseDirectiveRefSym(AsmToken DirectiveID);

  unsigned validateTargetOperandClass(MCParsedAsmOperand &Op,
                                      unsigned Kind) override;

```
- **EN**: Implements logic around `parseInstruction`, `parseDirective`, `ParseDirectiveRefSym`, `validateTargetOperandClass`.
- **CN**: 围绕 `parseInstruction`, `parseDirective`, `ParseDirectiveRefSym`, `validateTargetOperandClass` 实现具体逻辑。

### Lines 60-69
```cpp
  bool parseJccInstruction(ParseInstructionInfo &Info, StringRef Name,
                           SMLoc NameLoc, OperandVector &Operands);

  bool ParseOperand(OperandVector &Operands);

  bool ParseLiteralValues(unsigned Size, SMLoc L);

  MCAsmParser &getParser() const { return Parser; }
  AsmLexer &getLexer() const { return Parser.getLexer(); }

```
- **EN**: Implements logic around `parseJccInstruction`, `ParseOperand`, `ParseLiteralValues`, `getParser`, ...; this block returns target-specific results.
- **CN**: 围绕 `parseJccInstruction`, `ParseOperand`, `ParseLiteralValues`, `getParser`, ... 实现具体逻辑；这一段返回目标相关结果。

### Lines 70-84
```cpp
  /// @name Auto-generated Matcher Functions
  /// {

#define GET_ASSEMBLER_HEADER
#include "MSP430GenAsmMatcher.inc"

  /// }

public:
  MSP430AsmParser(const MCSubtargetInfo &STI, MCAsmParser &Parser,
                  const MCInstrInfo &MII)
      : MCTargetAsmParser(STI, MII), Parser(Parser) {
    MCAsmParserExtension::Initialize(Parser);
    MRI = getContext().getRegisterInfo();

```
- **EN**: Pulls in the headers needed for this implementation, including `MSP430GenAsmMatcher.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `MSP430GenAsmMatcher.inc`。

### Lines 85-101
```cpp
    setAvailableFeatures(ComputeAvailableFeatures(STI.getFeatureBits()));
  }
};

/// A parsed MSP430 assembly operand.
class MSP430Operand : public MCParsedAsmOperand {
  typedef MCParsedAsmOperand Base;

  enum KindTy {
    k_Imm,
    k_Reg,
    k_Tok,
    k_Mem,
    k_IndReg,
    k_PostIndReg
  } Kind;

```
- **EN**: Introduces declarations for `MSP430Operand`, `KindTy`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `MSP430Operand`, `KindTy` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 102-112
```cpp
  struct Memory {
    MCRegister Reg;
    const MCExpr *Offset;
  };
  union {
    const MCExpr *Imm;
    MCRegister    Reg;
    StringRef     Tok;
    Memory        Mem;
  };

```
- **EN**: Introduces declarations for `Memory`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `Memory` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 113-125
```cpp
  SMLoc Start, End;

public:
  MSP430Operand(StringRef Tok, SMLoc const &S)
      : Kind(k_Tok), Tok(Tok), Start(S), End(S) {}
  MSP430Operand(KindTy Kind, MCRegister Reg, SMLoc const &S, SMLoc const &E)
      : Kind(Kind), Reg(Reg), Start(S), End(E) {}
  MSP430Operand(MCExpr const *Imm, SMLoc const &S, SMLoc const &E)
      : Kind(k_Imm), Imm(Imm), Start(S), End(E) {}
  MSP430Operand(MCRegister Reg, MCExpr const *Expr, SMLoc const &S,
                SMLoc const &E)
      : Kind(k_Mem), Mem({Reg, Expr}), Start(S), End(E) {}

```
- **EN**: Implements logic around `MSP430Operand`, `Kind`.
- **CN**: 围绕 `MSP430Operand`, `Kind` 实现具体逻辑。

### Lines 126-143
```cpp
  void addRegOperands(MCInst &Inst, unsigned N) const {
    assert((Kind == k_Reg || Kind == k_IndReg || Kind == k_PostIndReg) &&
        "Unexpected operand kind");
    assert(N == 1 && "Invalid number of operands!");

    Inst.addOperand(MCOperand::createReg(Reg));
  }

  void addExprOperand(MCInst &Inst, const MCExpr *Expr) const {
    // Add as immediate when possible
    if (!Expr)
      Inst.addOperand(MCOperand::createImm(0));
    else if (const MCConstantExpr *CE = dyn_cast<MCConstantExpr>(Expr))
      Inst.addOperand(MCOperand::createImm(CE->getValue()));
    else
      Inst.addOperand(MCOperand::createExpr(Expr));
  }

```
- **EN**: Implements logic around `addRegOperands`, `assert`, `addOperand`, `addExprOperand`; this block applies conditional target rules; works at the MC layer.
- **CN**: 围绕 `addRegOperands`, `assert`, `addOperand`, `addExprOperand` 实现具体逻辑；这一段应用条件化的目标规则，工作在 MC 层。

### Lines 144-154
```cpp
  void addImmOperands(MCInst &Inst, unsigned N) const {
    assert(Kind == k_Imm && "Unexpected operand kind");
    assert(N == 1 && "Invalid number of operands!");

    addExprOperand(Inst, Imm);
  }

  void addMemOperands(MCInst &Inst, unsigned N) const {
    assert(Kind == k_Mem && "Unexpected operand kind");
    assert(N == 2 && "Invalid number of operands");

```
- **EN**: Implements logic around `addImmOperands`, `assert`, `addExprOperand`, `addMemOperands`; this block works at the MC layer.
- **CN**: 围绕 `addImmOperands`, `assert`, `addExprOperand`, `addMemOperands` 实现具体逻辑；这一段工作在 MC 层。

### Lines 155-165
```cpp
    Inst.addOperand(MCOperand::createReg(Mem.Reg));
    addExprOperand(Inst, Mem.Offset);
  }

  bool isReg()   const override { return Kind == k_Reg; }
  bool isImm()   const override { return Kind == k_Imm; }
  bool isToken() const override { return Kind == k_Tok; }
  bool isMem()   const override { return Kind == k_Mem; }
  bool isIndReg()         const { return Kind == k_IndReg; }
  bool isPostIndReg()     const { return Kind == k_PostIndReg; }

```
- **EN**: Implements logic around `addOperand`, `addExprOperand`, `isReg`, `isImm`, ...; this block returns target-specific results.
- **CN**: 围绕 `addOperand`, `addExprOperand`, `isReg`, `isImm`, ... 实现具体逻辑；这一段返回目标相关结果。

### Lines 166-176
```cpp
  bool isCGImm() const {
    if (Kind != k_Imm)
      return false;

    int64_t Val;
    if (!Imm->evaluateAsAbsolute(Val))
      return false;
    
    if (Val == 0 || Val == 1 || Val == 2 || Val == 4 || Val == 8 || Val == -1)
      return true;

```
- **EN**: Implements logic around `isCGImm`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `isCGImm` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 177-189
```cpp
    return false;
  }

  StringRef getToken() const {
    assert(Kind == k_Tok && "Invalid access!");
    return Tok;
  }

  MCRegister getReg() const override {
    assert(Kind == k_Reg && "Invalid access!");
    return Reg;
  }

```
- **EN**: Implements logic around `getToken`, `assert`, `getReg`; this block returns target-specific results.
- **CN**: 围绕 `getToken`, `assert`, `getReg` 实现具体逻辑；这一段返回目标相关结果。

### Lines 190-198
```cpp
  void setReg(MCRegister RegNo) {
    assert(Kind == k_Reg && "Invalid access!");
    Reg = RegNo;
  }

  static std::unique_ptr<MSP430Operand> CreateToken(StringRef Str, SMLoc S) {
    return std::make_unique<MSP430Operand>(Str, S);
  }

```
- **EN**: Implements logic around `setReg`, `assert`, `CreateToken`, `make_unique<MSP430Operand>`; this block returns target-specific results.
- **CN**: 围绕 `setReg`, `assert`, `CreateToken`, `make_unique<MSP430Operand>` 实现具体逻辑；这一段返回目标相关结果。

### Lines 199-208
```cpp
  static std::unique_ptr<MSP430Operand> CreateReg(MCRegister Reg, SMLoc S,
                                                  SMLoc E) {
    return std::make_unique<MSP430Operand>(k_Reg, Reg, S, E);
  }

  static std::unique_ptr<MSP430Operand> CreateImm(const MCExpr *Val, SMLoc S,
                                                  SMLoc E) {
    return std::make_unique<MSP430Operand>(Val, S, E);
  }

```
- **EN**: Implements logic around `CreateReg`, `make_unique<MSP430Operand>`, `CreateImm`; this block returns target-specific results.
- **CN**: 围绕 `CreateReg`, `make_unique<MSP430Operand>`, `CreateImm` 实现具体逻辑；这一段返回目标相关结果。

### Lines 209-218
```cpp
  static std::unique_ptr<MSP430Operand>
  CreateMem(MCRegister Reg, const MCExpr *Val, SMLoc S, SMLoc E) {
    return std::make_unique<MSP430Operand>(Reg, Val, S, E);
  }

  static std::unique_ptr<MSP430Operand> CreateIndReg(MCRegister Reg, SMLoc S,
                                                     SMLoc E) {
    return std::make_unique<MSP430Operand>(k_IndReg, Reg, S, E);
  }

```
- **EN**: Implements logic around `CreateMem`, `make_unique<MSP430Operand>`, `CreateIndReg`; this block returns target-specific results.
- **CN**: 围绕 `CreateMem`, `make_unique<MSP430Operand>`, `CreateIndReg` 实现具体逻辑；这一段返回目标相关结果。

### Lines 219-236
```cpp
  static std::unique_ptr<MSP430Operand> CreatePostIndReg(MCRegister Reg,
                                                         SMLoc S, SMLoc E) {
    return std::make_unique<MSP430Operand>(k_PostIndReg, Reg, S, E);
  }

  SMLoc getStartLoc() const override { return Start; }
  SMLoc getEndLoc() const override { return End; }

  void print(raw_ostream &O, const MCAsmInfo &MAI) const override {
    switch (Kind) {
    case k_Tok:
      O << "Token " << Tok;
      break;
    case k_Reg:
      O << "Register " << Reg.id();
      break;
    case k_Imm:
      O << "Immediate ";
```
- **EN**: Implements logic around `CreatePostIndReg`, `make_unique<MSP430Operand>`, `getStartLoc`, `getEndLoc`, ...; this block uses `switch`-based dispatch; returns target-specific results.
- **CN**: 围绕 `CreatePostIndReg`, `make_unique<MSP430Operand>`, `getStartLoc`, `getEndLoc`, ... 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果。

### Lines 237-253
```cpp
      MAI.printExpr(O, *Imm);
      break;
    case k_Mem:
      O << "Memory ";
      MAI.printExpr(O, *Mem.Offset);
      break;
    case k_IndReg:
      O << "RegInd " << Reg.id();
      break;
    case k_PostIndReg:
      O << "PostInc " << Reg.id();
      break;
    }
  }
};
} // end anonymous namespace

```
- **EN**: Implements logic around `printExpr`, `id`.
- **CN**: 围绕 `printExpr`, `id` 实现具体逻辑。

### Lines 254-262
```cpp
bool MSP430AsmParser::matchAndEmitInstruction(SMLoc Loc, unsigned &Opcode,
                                              OperandVector &Operands,
                                              MCStreamer &Out,
                                              uint64_t &ErrorInfo,
                                              bool MatchingInlineAsm) {
  MCInst Inst;
  unsigned MatchResult =
      MatchInstructionImpl(Operands, Inst, ErrorInfo, MatchingInlineAsm);

```
- **EN**: Implements logic around `matchAndEmitInstruction`, `MatchInstructionImpl`; this block works at the MC layer.
- **CN**: 围绕 `matchAndEmitInstruction`, `MatchInstructionImpl` 实现具体逻辑；这一段工作在 MC 层。

### Lines 263-275
```cpp
  switch (MatchResult) {
  case Match_Success:
    Inst.setLoc(Loc);
    Out.emitInstruction(Inst, *STI);
    return false;
  case Match_MnemonicFail:
    return Error(Loc, "invalid instruction mnemonic");
  case Match_InvalidOperand: {
    SMLoc ErrorLoc = Loc;
    if (ErrorInfo != ~0U) {
      if (ErrorInfo >= Operands.size())
        return Error(ErrorLoc, "too few operands for instruction");

```
- **EN**: Implements logic around `setLoc`, `emitInstruction`, `Error`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `setLoc`, `emitInstruction`, `Error` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果。

### Lines 276-286
```cpp
      ErrorLoc = ((MSP430Operand &)*Operands[ErrorInfo]).getStartLoc();
      if (ErrorLoc == SMLoc())
        ErrorLoc = Loc;
    }
    return Error(ErrorLoc, "invalid operand for instruction");
  }
  default:
    return true;
  }
}

```
- **EN**: Implements logic around `getStartLoc`, `Error`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getStartLoc`, `Error` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 287-300
```cpp
// Auto-generated by TableGen
static MCRegister MatchRegisterName(StringRef Name);
static MCRegister MatchRegisterAltName(StringRef Name);

bool MSP430AsmParser::parseRegister(MCRegister &Reg, SMLoc &StartLoc,
                                    SMLoc &EndLoc) {
  ParseStatus Res = tryParseRegister(Reg, StartLoc, EndLoc);
  if (Res.isFailure())
    return Error(StartLoc, "invalid register name");
  if (Res.isSuccess())
    return false;
  if (Res.isNoMatch())
    return true;

```
- **EN**: Implements logic around `MatchRegisterName`, `MatchRegisterAltName`, `parseRegister`, `tryParseRegister`, ...; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `MatchRegisterName`, `MatchRegisterAltName`, `parseRegister`, `tryParseRegister`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 301-314
```cpp
  llvm_unreachable("unknown parse status");
}

ParseStatus MSP430AsmParser::tryParseRegister(MCRegister &Reg, SMLoc &StartLoc,
                                              SMLoc &EndLoc) {
  if (getLexer().getKind() == AsmToken::Identifier) {
    auto Name = getLexer().getTok().getIdentifier().lower();
    Reg = MatchRegisterName(Name);
    if (Reg == MSP430::NoRegister) {
      Reg = MatchRegisterAltName(Name);
      if (Reg == MSP430::NoRegister)
        return ParseStatus::NoMatch;
    }

```
- **EN**: Implements logic around `llvm_unreachable`, `tryParseRegister`, `getLexer`, `MatchRegisterName`, ...; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `llvm_unreachable`, `tryParseRegister`, `getLexer`, `MatchRegisterName`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 315-325
```cpp
    AsmToken const &T = getParser().getTok();
    StartLoc = T.getLoc();
    EndLoc = T.getEndLoc();
    getLexer().Lex(); // eat register token

    return ParseStatus::Success;
  }

  return ParseStatus::Failure;
}

```
- **EN**: Implements logic around `getParser`, `getLoc`, `getEndLoc`, `getLexer`; this block returns target-specific results.
- **CN**: 围绕 `getParser`, `getLoc`, `getEndLoc`, `getLexer` 实现具体逻辑；这一段返回目标相关结果。

### Lines 326-343
```cpp
bool MSP430AsmParser::parseJccInstruction(ParseInstructionInfo &Info,
                                          StringRef Name, SMLoc NameLoc,
                                          OperandVector &Operands) {
  if (!Name.starts_with_insensitive("j"))
    return true;

  auto CC = Name.drop_front().lower();
  unsigned CondCode;
  if (CC == "ne" || CC == "nz")
    CondCode = MSP430CC::COND_NE;
  else if (CC == "eq" || CC == "z")
    CondCode = MSP430CC::COND_E;
  else if (CC == "lo" || CC == "nc")
    CondCode = MSP430CC::COND_LO;
  else if (CC == "hs" || CC == "c")
    CondCode = MSP430CC::COND_HS;
  else if (CC == "n")
    CondCode = MSP430CC::COND_N;
```
- **EN**: Implements logic around `parseJccInstruction`, `drop_front`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `parseJccInstruction`, `drop_front` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 344-352
```cpp
  else if (CC == "ge")
    CondCode = MSP430CC::COND_GE;
  else if (CC == "l")
    CondCode = MSP430CC::COND_L;
  else if (CC == "mp")
    CondCode = MSP430CC::COND_NONE;
  else
    return Error(NameLoc, "unknown instruction");

```
- **EN**: Implements logic around `Error`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `Error` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 353-363
```cpp
  if (CondCode == (unsigned)MSP430CC::COND_NONE)
    Operands.push_back(MSP430Operand::CreateToken("jmp", NameLoc));
  else {
    Operands.push_back(MSP430Operand::CreateToken("j", NameLoc));
    const MCExpr *CCode = MCConstantExpr::create(CondCode, getContext());
    Operands.push_back(MSP430Operand::CreateImm(CCode, SMLoc(), SMLoc()));
  }

  // Skip optional '$' sign.
  (void)parseOptionalToken(AsmToken::Dollar);

```
- **EN**: Implements logic around `push_back`, `create`, `parseOptionalToken`; this block applies conditional target rules.
- **CN**: 围绕 `push_back`, `create`, `parseOptionalToken` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 364-373
```cpp
  const MCExpr *Val;
  SMLoc ExprLoc = getLexer().getLoc();
  if (getParser().parseExpression(Val))
    return Error(ExprLoc, "expected expression operand");

  int64_t Res;
  if (Val->evaluateAsAbsolute(Res))
    if (Res < -512 || Res > 511)
      return Error(ExprLoc, "invalid jump offset");

```
- **EN**: Implements logic around `getLexer`, `Error`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getLexer`, `Error` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 374-382
```cpp
  Operands.push_back(MSP430Operand::CreateImm(Val, ExprLoc,
    getLexer().getLoc()));

  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    SMLoc Loc = getLexer().getLoc();
    getParser().eatToEndOfStatement();
    return Error(Loc, "unexpected token");
  }

```
- **EN**: Implements logic around `push_back`, `getLexer`, `getParser`, `Error`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `push_back`, `getLexer`, `getParser`, `Error` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 383-393
```cpp
  getParser().Lex(); // Consume the EndOfStatement.
  return false;
}

bool MSP430AsmParser::parseInstruction(ParseInstructionInfo &Info,
                                       StringRef Name, SMLoc NameLoc,
                                       OperandVector &Operands) {
  // Drop .w suffix
  if (Name.ends_with_insensitive(".w"))
    Name = Name.drop_back(2);

```
- **EN**: Implements logic around `getParser`, `parseInstruction`, `drop_back`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getParser`, `parseInstruction`, `drop_back` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 394-403
```cpp
  if (!parseJccInstruction(Info, Name, NameLoc, Operands))
    return false;

  // First operand is instruction mnemonic
  Operands.push_back(MSP430Operand::CreateToken(Name, NameLoc));

  // If there are no more operands, then finish
  if (getLexer().is(AsmToken::EndOfStatement))
    return false;

```
- **EN**: Implements logic around `push_back`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `push_back` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 404-417
```cpp
  // Parse first operand
  if (ParseOperand(Operands))
    return true;

  // Parse second operand if any
  if (parseOptionalToken(AsmToken::Comma) && ParseOperand(Operands))
    return true;

  if (getLexer().isNot(AsmToken::EndOfStatement)) {
    SMLoc Loc = getLexer().getLoc();
    getParser().eatToEndOfStatement();
    return Error(Loc, "unexpected token");
  }

```
- **EN**: Implements logic around `getLexer`, `getParser`, `Error`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getLexer`, `getParser`, `Error` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 418-426
```cpp
  getParser().Lex(); // Consume the EndOfStatement.
  return false;
}

bool MSP430AsmParser::ParseDirectiveRefSym(AsmToken DirectiveID) {
  StringRef Name;
  if (getParser().parseIdentifier(Name))
    return TokError("expected identifier in directive");

```
- **EN**: Implements logic around `getParser`, `ParseDirectiveRefSym`, `TokError`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getParser`, `ParseDirectiveRefSym`, `TokError` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 427-444
```cpp
  MCSymbol *Sym = getContext().getOrCreateSymbol(Name);
  getStreamer().emitSymbolAttribute(Sym, MCSA_Global);
  return parseEOL();
}

ParseStatus MSP430AsmParser::parseDirective(AsmToken DirectiveID) {
  StringRef IDVal = DirectiveID.getIdentifier();
  if (IDVal.lower() == ".long")
    return ParseLiteralValues(4, DirectiveID.getLoc());
  if (IDVal.lower() == ".word" || IDVal.lower() == ".short")
    return ParseLiteralValues(2, DirectiveID.getLoc());
  if (IDVal.lower() == ".byte")
    return ParseLiteralValues(1, DirectiveID.getLoc());
  if (IDVal.lower() == ".refsym")
    return ParseDirectiveRefSym(DirectiveID);
  return ParseStatus::NoMatch;
}

```
- **EN**: Implements logic around `getContext`, `getStreamer`, `parseEOL`, `parseDirective`, ...; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getContext`, `getStreamer`, `parseEOL`, `parseDirective`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 445-462
```cpp
bool MSP430AsmParser::ParseOperand(OperandVector &Operands) {
  switch (getLexer().getKind()) {
    default: return true;
    case AsmToken::Identifier: {
      // try rN
      MCRegister RegNo;
      SMLoc StartLoc, EndLoc;
      if (!parseRegister(RegNo, StartLoc, EndLoc)) {
        Operands.push_back(MSP430Operand::CreateReg(RegNo, StartLoc, EndLoc));
        return false;
      }
      [[fallthrough]];
    }
    case AsmToken::Integer:
    case AsmToken::Plus:
    case AsmToken::Minus: {
      SMLoc StartLoc = getParser().getTok().getLoc();
      const MCExpr *Val;
```
- **EN**: Implements logic around `ParseOperand`, `push_back`, `getParser`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `ParseOperand`, `push_back`, `getParser` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果。

### Lines 463-480
```cpp
      // Try constexpr[(rN)]
      if (!getParser().parseExpression(Val)) {
        MCRegister RegNo = MSP430::PC;
        SMLoc EndLoc = getParser().getTok().getLoc();
        // Try (rN)
        if (parseOptionalToken(AsmToken::LParen)) {
          SMLoc RegStartLoc;
          if (parseRegister(RegNo, RegStartLoc, EndLoc))
            return true;
          EndLoc = getParser().getTok().getEndLoc();
          if (!parseOptionalToken(AsmToken::RParen))
            return true;
        }
        Operands.push_back(MSP430Operand::CreateMem(RegNo, Val, StartLoc,
          EndLoc));
        return false;
      }
      return true;
```
- **EN**: Implements logic around `getParser`, `push_back`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getParser`, `push_back` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 481-498
```cpp
    }
    case AsmToken::Amp: {
      // Try &constexpr
      SMLoc StartLoc = getParser().getTok().getLoc();
      getLexer().Lex(); // Eat '&'
      const MCExpr *Val;
      if (!getParser().parseExpression(Val)) {
        SMLoc EndLoc = getParser().getTok().getLoc();
        Operands.push_back(MSP430Operand::CreateMem(MSP430::SR, Val, StartLoc,
          EndLoc));
        return false;
      }
      return true;
    }
    case AsmToken::At: {
      // Try @rN[+]
      SMLoc StartLoc = getParser().getTok().getLoc();
      getLexer().Lex(); // Eat '@'
```
- **EN**: Implements logic around `getParser`, `getLexer`, `push_back`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getParser`, `getLexer`, `push_back` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 499-516
```cpp
      MCRegister RegNo;
      SMLoc RegStartLoc, EndLoc;
      if (parseRegister(RegNo, RegStartLoc, EndLoc))
        return true;
      if (parseOptionalToken(AsmToken::Plus)) {
        Operands.push_back(MSP430Operand::CreatePostIndReg(RegNo, StartLoc, EndLoc));
        return false;
      }
      if (Operands.size() > 1) // Emulate @rd in destination position as 0(rd)
        Operands.push_back(MSP430Operand::CreateMem(RegNo,
            MCConstantExpr::create(0, getContext()), StartLoc, EndLoc));
      else
        Operands.push_back(MSP430Operand::CreateIndReg(RegNo, StartLoc, EndLoc));
      return false;
    }
    case AsmToken::Hash:
      // Try #constexpr
      SMLoc StartLoc = getParser().getTok().getLoc();
```
- **EN**: Implements logic around `push_back`, `create`, `getParser`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `push_back`, `create`, `getParser` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 517-527
```cpp
      getLexer().Lex(); // Eat '#'
      const MCExpr *Val;
      if (!getParser().parseExpression(Val)) {
        SMLoc EndLoc = getParser().getTok().getLoc();
        Operands.push_back(MSP430Operand::CreateImm(Val, StartLoc, EndLoc));
        return false;
      }
      return true;
  }
}

```
- **EN**: Implements logic around `getLexer`, `getParser`, `push_back`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getLexer`, `getParser`, `push_back` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 528-538
```cpp
bool MSP430AsmParser::ParseLiteralValues(unsigned Size, SMLoc L) {
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
- **EN**: Implements logic around `ParseLiteralValues`, `getParser`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `ParseLiteralValues`, `getParser` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 539-547
```cpp
extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
LLVMInitializeMSP430AsmParser() {
  RegisterMCAsmParser<MSP430AsmParser> X(getTheMSP430Target());
}

#define GET_REGISTER_MATCHER
#define GET_MATCHER_IMPLEMENTATION
#include "MSP430GenAsmMatcher.inc"

```
- **EN**: Pulls in the headers needed for this implementation, including `MSP430GenAsmMatcher.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `MSP430GenAsmMatcher.inc`。

### Lines 548-565
```cpp
static MCRegister convertGR16ToGR8(MCRegister Reg) {
  switch (Reg.id()) {
  default:
    llvm_unreachable("Unknown GR16 register");
  case MSP430::PC:  return MSP430::PCB;
  case MSP430::SP:  return MSP430::SPB;
  case MSP430::SR:  return MSP430::SRB;
  case MSP430::CG:  return MSP430::CGB;
  case MSP430::R4:  return MSP430::R4B;
  case MSP430::R5:  return MSP430::R5B;
  case MSP430::R6:  return MSP430::R6B;
  case MSP430::R7:  return MSP430::R7B;
  case MSP430::R8:  return MSP430::R8B;
  case MSP430::R9:  return MSP430::R9B;
  case MSP430::R10: return MSP430::R10B;
  case MSP430::R11: return MSP430::R11B;
  case MSP430::R12: return MSP430::R12B;
  case MSP430::R13: return MSP430::R13B;
```
- **EN**: Implements logic around `convertGR16ToGR8`, `llvm_unreachable`; this block uses `switch`-based dispatch; returns target-specific results.
- **CN**: 围绕 `convertGR16ToGR8`, `llvm_unreachable` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果。

### Lines 566-574
```cpp
  case MSP430::R14: return MSP430::R14B;
  case MSP430::R15: return MSP430::R15B;
  }
}

unsigned MSP430AsmParser::validateTargetOperandClass(MCParsedAsmOperand &AsmOp,
                                                     unsigned Kind) {
  MSP430Operand &Op = static_cast<MSP430Operand &>(AsmOp);

```
- **EN**: Implements logic around `validateTargetOperandClass`; this block returns target-specific results.
- **CN**: 围绕 `validateTargetOperandClass` 实现具体逻辑；这一段返回目标相关结果。

### Lines 575-586
```cpp
  if (!Op.isReg())
    return Match_InvalidOperand;

  MCRegister Reg = Op.getReg();
  bool isGR16 =
      MSP430MCRegisterClasses[MSP430::GR16RegClassID].contains(Reg);

  if (isGR16 && (Kind == MCK_GR8)) {
    Op.setReg(convertGR16ToGR8(Reg));
    return Match_Success;
  }

```
- **EN**: Implements logic around `getReg`, `contains`, `setReg`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getReg`, `contains`, `setReg` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 587-588
```cpp
  return Match_InvalidOperand;
}
```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

## Key Concepts / 关键概念

- **Assembly parsing / 汇编解析**:
  - **EN**: Parses textual assembly into operands and instructions
  - **CN**: 将文本汇编解析为操作数和指令

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `MCTargetDesc/MSP430MCAsmInfo.h`, `MCTargetDesc/MSP430MCTargetDesc.h`, `MSP430.h`, `MSP430RegisterInfo.h`, `TargetInfo/MSP430TargetInfo.h`, `llvm/ADT/APInt.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCInst.h`, `llvm/MC/MCInstrInfo.h`, `llvm/MC/MCParser/AsmLexer.h`, `llvm/MC/MCParser/MCParsedAsmOperand.h` ... (+9 more)
- **LLVM subsystems / LLVM 子系统**: MC, Support
- **Generated macros / 生成宏**: `GET_ASSEMBLER_HEADER`, `GET_MATCHER_IMPLEMENTATION`, `GET_REGISTER_MATCHER`
