# AVRAsmParser.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/AVR/AsmParser/AVRAsmParser.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Implements the target-specific assembly parser that turns textual assembly into MC operands and instructions.
- 目的（中文）: 实现目标专用的汇编解析器，将文本汇编转换为 MC 操作数与指令。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===---- AVRAsmParser.cpp - Parse AVR assembly to MCInst instructions ----===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: #include "AVRRegisterInfo.h"
  10: #include "MCTargetDesc/AVRMCAsmInfo.h"
  11: #include "MCTargetDesc/AVRMCELFStreamer.h"
  12: #include "MCTargetDesc/AVRMCTargetDesc.h"
  13: #include "TargetInfo/AVRTargetInfo.h"
  14: 
  15: #include "llvm/ADT/APInt.h"
  16: #include "llvm/MC/MCContext.h"
  17: #include "llvm/MC/MCExpr.h"
  18: #include "llvm/MC/MCInst.h"
  19: #include "llvm/MC/MCParser/AsmLexer.h"
  20: #include "llvm/MC/MCParser/MCParsedAsmOperand.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 21-40

```cpp
  21: #include "llvm/MC/MCParser/MCTargetAsmParser.h"
  22: #include "llvm/MC/MCStreamer.h"
  23: #include "llvm/MC/MCSubtargetInfo.h"
  24: #include "llvm/MC/MCSymbol.h"
  25: #include "llvm/MC/MCValue.h"
  26: #include "llvm/MC/TargetRegistry.h"
  27: #include "llvm/Support/Compiler.h"
  28: #include "llvm/Support/Debug.h"
  29: #include "llvm/Support/MathExtras.h"
  30: 
  31: #include <array>
  32: #include <sstream>
  33: 
  34: #define DEBUG_TYPE "avr-asm-parser"
  35: 
  36: using namespace llvm;
  37: 
  38: namespace {
  39: /// Parses AVR assembly from a stream.
  40: class AVRAsmParser : public MCTargetAsmParser {
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. This chunk introduces interfaces or data structures such as AVRAsmParser, which organize the target-specific behavior exposed by the file. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 这一段引入了 AVRAsmParser 等接口或数据结构，用于组织该文件暴露的目标专用行为。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 41-60

```cpp
  41:   MCAsmParser &Parser;
  42:   const MCRegisterInfo *MRI;
  43:   const std::string GENERATE_STUBS = "gs";
  44: 
  45:   enum AVRMatchResultTy {
  46:     Match_InvalidRegisterOnTiny = FIRST_TARGET_MATCH_RESULT_TY + 1,
  47:   };
  48: 
  49: #define GET_ASSEMBLER_HEADER
  50: #include "AVRGenAsmMatcher.inc"
  51: 
  52:   bool matchAndEmitInstruction(SMLoc IDLoc, unsigned &Opcode,
  53:                                OperandVector &Operands, MCStreamer &Out,
  54:                                uint64_t &ErrorInfo,
  55:                                bool MatchingInlineAsm) override;
  56: 
  57:   bool parseRegister(MCRegister &Reg, SMLoc &StartLoc, SMLoc &EndLoc) override;
  58:   ParseStatus tryParseRegister(MCRegister &Reg, SMLoc &StartLoc,
  59:                                SMLoc &EndLoc) override;
  60: 
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 61-80

```cpp
  61:   bool parseInstruction(ParseInstructionInfo &Info, StringRef Name,
  62:                         SMLoc NameLoc, OperandVector &Operands) override;
  63: 
  64:   ParseStatus parseDirective(AsmToken DirectiveID) override;
  65: 
  66:   ParseStatus parseMemriOperand(OperandVector &Operands);
  67: 
  68:   bool parseOperand(OperandVector &Operands, bool maybeReg);
  69:   MCRegister parseRegisterName(MCRegister (*matchFn)(StringRef));
  70:   MCRegister parseRegisterName();
  71:   MCRegister parseRegister(bool RestoreOnFailure = false);
  72:   bool tryParseRegisterOperand(OperandVector &Operands);
  73:   bool tryParseExpression(OperandVector &Operands, int64_t offset);
  74:   bool tryParseRelocExpression(OperandVector &Operands);
  75:   void eatComma();
  76: 
  77:   unsigned validateTargetOperandClass(MCParsedAsmOperand &Op,
  78:                                       unsigned Kind) override;
  79: 
  80:   MCRegister toDREG(MCRegister Reg, unsigned From = AVR::sub_lo) {
```

- EN: Function bodies or method definitions such as toDREG contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: toDREG 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 81-100

```cpp
  81:     MCRegisterClass const *Class = &AVRMCRegisterClasses[AVR::DREGSRegClassID];
  82:     return MRI->getMatchingSuperReg(Reg, From, Class);
  83:   }
  84: 
  85:   bool emit(MCInst &Instruction, SMLoc const &Loc, MCStreamer &Out) const;
  86:   bool invalidOperand(SMLoc const &Loc, OperandVector const &Operands,
  87:                       uint64_t const &ErrorInfo);
  88:   bool missingFeature(SMLoc const &Loc, uint64_t const &ErrorInfo);
  89: 
  90:   ParseStatus parseLiteralValues(unsigned SizeInBytes, SMLoc L);
  91: 
  92: public:
  93:   AVRAsmParser(const MCSubtargetInfo &STI, MCAsmParser &Parser,
  94:                const MCInstrInfo &MII)
  95:       : MCTargetAsmParser(STI, MII), Parser(Parser) {
  96:     MCAsmParserExtension::Initialize(Parser);
  97:     MRI = getContext().getRegisterInfo();
  98: 
  99:     setAvailableFeatures(ComputeAvailableFeatures(STI.getFeatureBits()));
 100:   }
```

- EN: Function bodies or method definitions such as AVRAsmParser contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: AVRAsmParser 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 101-120

```cpp
 101: 
 102:   MCAsmParser &getParser() const { return Parser; }
 103:   AsmLexer &getLexer() const { return Parser.getLexer(); }
 104: };
 105: 
 106: /// An parsed AVR assembly operand.
 107: class AVROperand : public MCParsedAsmOperand {
 108:   typedef MCParsedAsmOperand Base;
 109:   enum KindTy { k_Immediate, k_Register, k_Token, k_Memri } Kind;
 110: 
 111: public:
 112:   AVROperand(StringRef Tok, SMLoc const &S)
 113:       : Kind(k_Token), Tok(Tok), Start(S), End(S) {}
 114:   AVROperand(MCRegister Reg, SMLoc const &S, SMLoc const &E)
 115:       : Kind(k_Register), RegImm({Reg, nullptr}), Start(S), End(E) {}
 116:   AVROperand(MCExpr const *Imm, SMLoc const &S, SMLoc const &E)
 117:       : Kind(k_Immediate), RegImm({0, Imm}), Start(S), End(E) {}
 118:   AVROperand(MCRegister Reg, MCExpr const *Imm, SMLoc const &S, SMLoc const &E)
 119:       : Kind(k_Memri), RegImm({Reg, Imm}), Start(S), End(E) {}
 120: 
```

- EN: This chunk introduces interfaces or data structures such as AVROperand, which organize the target-specific behavior exposed by the file. Function bodies or method definitions such as AVROperand contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 这一段引入了 AVROperand 等接口或数据结构，用于组织该文件暴露的目标专用行为。 AVROperand 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 121-140

```cpp
 121:   struct RegisterImmediate {
 122:     MCRegister Reg;
 123:     MCExpr const *Imm;
 124:   };
 125:   union {
 126:     StringRef Tok;
 127:     RegisterImmediate RegImm;
 128:   };
 129: 
 130:   SMLoc Start, End;
 131: 
 132: public:
 133:   void addRegOperands(MCInst &Inst, unsigned N) const {
 134:     assert(Kind == k_Register && "Unexpected operand kind");
 135:     assert(N == 1 && "Invalid number of operands!");
 136: 
 137:     Inst.addOperand(MCOperand::createReg(getReg()));
 138:   }
 139: 
 140:   void addExpr(MCInst &Inst, const MCExpr *Expr) const {
```

- EN: This chunk introduces interfaces or data structures such as RegisterImmediate, which organize the target-specific behavior exposed by the file. Function bodies or method definitions such as addRegOperands, addExpr contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 这一段引入了 RegisterImmediate 等接口或数据结构，用于组织该文件暴露的目标专用行为。 addRegOperands, addExpr 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 141-160

```cpp
 141:     // Add as immediate when possible
 142:     if (!Expr)
 143:       Inst.addOperand(MCOperand::createImm(0));
 144:     else if (const MCConstantExpr *CE = dyn_cast<MCConstantExpr>(Expr))
 145:       Inst.addOperand(MCOperand::createImm(CE->getValue()));
 146:     else
 147:       Inst.addOperand(MCOperand::createExpr(Expr));
 148:   }
 149: 
 150:   void addImmOperands(MCInst &Inst, unsigned N) const {
 151:     assert(Kind == k_Immediate && "Unexpected operand kind");
 152:     assert(N == 1 && "Invalid number of operands!");
 153: 
 154:     const MCExpr *Expr = getImm();
 155:     addExpr(Inst, Expr);
 156:   }
 157: 
 158:   /// Adds the contained reg+imm operand to an instruction.
 159:   void addMemriOperands(MCInst &Inst, unsigned N) const {
 160:     assert(Kind == k_Memri && "Unexpected operand kind");
```

- EN: Function bodies or method definitions such as addImmOperands, addMemriOperands contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: addImmOperands, addMemriOperands 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 161-180

```cpp
 161:     assert(N == 2 && "Invalid number of operands");
 162: 
 163:     Inst.addOperand(MCOperand::createReg(getReg()));
 164:     addExpr(Inst, getImm());
 165:   }
 166: 
 167:   void addImmCom8Operands(MCInst &Inst, unsigned N) const {
 168:     assert(N == 1 && "Invalid number of operands!");
 169:     // The operand is actually a imm8, but we have its bitwise
 170:     // negation in the assembly source, so twiddle it here.
 171:     const auto *CE = cast<MCConstantExpr>(getImm());
 172:     Inst.addOperand(MCOperand::createImm(~(uint8_t)CE->getValue()));
 173:   }
 174: 
 175:   bool isImmCom8() const {
 176:     if (!isImm())
 177:       return false;
 178:     const auto *CE = dyn_cast<MCConstantExpr>(getImm());
 179:     if (!CE)
 180:       return false;
```

- EN: Function bodies or method definitions such as addImmCom8Operands, isImmCom8 contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: addImmCom8Operands, isImmCom8 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 181-200

```cpp
 181:     int64_t Value = CE->getValue();
 182:     return isUInt<8>(Value);
 183:   }
 184: 
 185:   bool isReg() const override { return Kind == k_Register; }
 186:   bool isImm() const override { return Kind == k_Immediate; }
 187:   bool isToken() const override { return Kind == k_Token; }
 188:   bool isMem() const override { return Kind == k_Memri; }
 189:   bool isMemri() const { return Kind == k_Memri; }
 190: 
 191:   StringRef getToken() const {
 192:     assert(Kind == k_Token && "Invalid access!");
 193:     return Tok;
 194:   }
 195: 
 196:   MCRegister getReg() const override {
 197:     assert((Kind == k_Register || Kind == k_Memri) && "Invalid access!");
 198: 
 199:     return RegImm.Reg;
 200:   }
```

- EN: Function bodies or method definitions such as isMemri, getToken contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: isMemri, getToken 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 201-220

```cpp
 201: 
 202:   const MCExpr *getImm() const {
 203:     assert((Kind == k_Immediate || Kind == k_Memri) && "Invalid access!");
 204:     return RegImm.Imm;
 205:   }
 206: 
 207:   static std::unique_ptr<AVROperand> CreateToken(StringRef Str, SMLoc S) {
 208:     return std::make_unique<AVROperand>(Str, S);
 209:   }
 210: 
 211:   static std::unique_ptr<AVROperand> CreateReg(MCRegister Reg, SMLoc S,
 212:                                                SMLoc E) {
 213:     return std::make_unique<AVROperand>(Reg, S, E);
 214:   }
 215: 
 216:   static std::unique_ptr<AVROperand> CreateImm(const MCExpr *Val, SMLoc S,
 217:                                                SMLoc E) {
 218:     return std::make_unique<AVROperand>(Val, S, E);
 219:   }
 220: 
```

- EN: Function bodies or method definitions such as CreateToken, CreateReg, CreateImm contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: CreateToken, CreateReg, CreateImm 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 221-240

```cpp
 221:   static std::unique_ptr<AVROperand>
 222:   CreateMemri(MCRegister Reg, const MCExpr *Val, SMLoc S, SMLoc E) {
 223:     return std::make_unique<AVROperand>(Reg, Val, S, E);
 224:   }
 225: 
 226:   void makeToken(StringRef Token) {
 227:     Kind = k_Token;
 228:     Tok = Token;
 229:   }
 230: 
 231:   void makeReg(MCRegister Reg) {
 232:     Kind = k_Register;
 233:     RegImm = {Reg, nullptr};
 234:   }
 235: 
 236:   void makeImm(MCExpr const *Ex) {
 237:     Kind = k_Immediate;
 238:     RegImm = {0, Ex};
 239:   }
 240: 
```

- EN: Function bodies or method definitions such as CreateMemri, makeToken, makeReg contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: CreateMemri, makeToken, makeReg 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 241-260

```cpp
 241:   void makeMemri(MCRegister Reg, MCExpr const *Imm) {
 242:     Kind = k_Memri;
 243:     RegImm = {Reg, Imm};
 244:   }
 245: 
 246:   SMLoc getStartLoc() const override { return Start; }
 247:   SMLoc getEndLoc() const override { return End; }
 248: 
 249:   void print(raw_ostream &O, const MCAsmInfo &MAI) const override {
 250:     switch (Kind) {
 251:     case k_Token:
 252:       O << "Token: \"" << getToken() << "\"";
 253:       break;
 254:     case k_Register:
 255:       O << "Register: " << getReg().id();
 256:       break;
 257:     case k_Immediate:
 258:       O << "Immediate: \"";
 259:       MAI.printExpr(O, *getImm());
 260:       O << "\"";
```

- EN: Function bodies or method definitions such as makeMemri contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: makeMemri 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 261-280

```cpp
 261:       break;
 262:     case k_Memri: {
 263:       // only manually print the size for non-negative values,
 264:       // as the sign is inserted automatically.
 265:       O << "Memri: \"" << getReg().id() << '+';
 266:       MAI.printExpr(O, *getImm());
 267:       O << "\"";
 268:       break;
 269:     }
 270:     }
 271:     O << "\n";
 272:   }
 273: };
 274: 
 275: } // end anonymous namespace.
 276: 
 277: // Auto-generated Match Functions
 278: 
 279: /// Maps from the set of all register names to a register number.
 280: /// \note Generated by TableGen.
```

- EN: In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 281-300

```cpp
 281: static MCRegister MatchRegisterName(StringRef Name);
 282: 
 283: /// Maps from the set of all alternative registernames to a register number.
 284: /// \note Generated by TableGen.
 285: static MCRegister MatchRegisterAltName(StringRef Name);
 286: 
 287: bool AVRAsmParser::invalidOperand(SMLoc const &Loc,
 288:                                   OperandVector const &Operands,
 289:                                   uint64_t const &ErrorInfo) {
 290:   SMLoc ErrorLoc = Loc;
 291:   char const *Diag = nullptr;
 292: 
 293:   if (ErrorInfo != ~0U) {
 294:     if (ErrorInfo >= Operands.size()) {
 295:       Diag = "too few operands for instruction.";
 296:     } else {
 297:       AVROperand const &Op = (AVROperand const &)*Operands[ErrorInfo];
 298: 
 299:       // TODO: See if we can do a better error than just "invalid ...".
 300:       if (Op.getStartLoc() != SMLoc()) {
```

- EN: Function bodies or method definitions such as invalidOperand contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: invalidOperand 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 301-320

```cpp
 301:         ErrorLoc = Op.getStartLoc();
 302:       }
 303:     }
 304:   }
 305: 
 306:   if (!Diag) {
 307:     Diag = "invalid operand for instruction";
 308:   }
 309: 
 310:   return Error(ErrorLoc, Diag);
 311: }
 312: 
 313: bool AVRAsmParser::missingFeature(llvm::SMLoc const &Loc,
 314:                                   uint64_t const &ErrorInfo) {
 315:   return Error(Loc, "instruction requires a CPU feature not currently enabled");
 316: }
 317: 
 318: bool AVRAsmParser::emit(MCInst &Inst, SMLoc const &Loc, MCStreamer &Out) const {
 319:   Inst.setLoc(Loc);
 320:   Out.emitInstruction(Inst, *STI);
```

- EN: Function bodies or method definitions such as missingFeature, emit contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: missingFeature, emit 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 321-340

```cpp
 321: 
 322:   return false;
 323: }
 324: 
 325: bool AVRAsmParser::matchAndEmitInstruction(SMLoc Loc, unsigned &Opcode,
 326:                                            OperandVector &Operands,
 327:                                            MCStreamer &Out, uint64_t &ErrorInfo,
 328:                                            bool MatchingInlineAsm) {
 329:   MCInst Inst;
 330:   unsigned MatchResult =
 331:       MatchInstructionImpl(Operands, Inst, ErrorInfo, MatchingInlineAsm);
 332: 
 333:   switch (MatchResult) {
 334:   case Match_Success:
 335:     return emit(Inst, Loc, Out);
 336:   case Match_MissingFeature:
 337:     return missingFeature(Loc, ErrorInfo);
 338:   case Match_InvalidOperand:
 339:     return invalidOperand(Loc, Operands, ErrorInfo);
 340:   case Match_MnemonicFail:
```

- EN: Function bodies or method definitions such as matchAndEmitInstruction contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: matchAndEmitInstruction 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 341-360

```cpp
 341:     return Error(Loc, "invalid instruction");
 342:   case Match_InvalidRegisterOnTiny:
 343:     return Error(Loc, "invalid register on avrtiny");
 344:   default:
 345:     return true;
 346:   }
 347: }
 348: 
 349: /// Parses a register name using a given matching function.
 350: /// Checks for lowercase or uppercase if necessary.
 351: MCRegister AVRAsmParser::parseRegisterName(MCRegister (*matchFn)(StringRef)) {
 352:   StringRef Name = Parser.getTok().getString();
 353: 
 354:   MCRegister Reg = matchFn(Name);
 355: 
 356:   // GCC supports case insensitive register names. Some of the AVR registers
 357:   // are all lower case, some are all upper case but non are mixed. We prefer
 358:   // to use the original names in the register definitions. That is why we
 359:   // have to test both upper and lower case here.
 360:   if (!Reg) {
```

- EN: Function bodies or method definitions such as parseRegisterName contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: parseRegisterName 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 361-380

```cpp
 361:     Reg = matchFn(Name.lower());
 362:   }
 363:   if (!Reg) {
 364:     Reg = matchFn(Name.upper());
 365:   }
 366: 
 367:   return Reg;
 368: }
 369: 
 370: MCRegister AVRAsmParser::parseRegisterName() {
 371:   MCRegister Reg = parseRegisterName(&MatchRegisterName);
 372: 
 373:   if (!Reg)
 374:     Reg = parseRegisterName(&MatchRegisterAltName);
 375: 
 376:   return Reg;
 377: }
 378: 
 379: MCRegister AVRAsmParser::parseRegister(bool RestoreOnFailure) {
 380:   MCRegister Reg;
```

- EN: Function bodies or method definitions such as parseRegisterName, parseRegister contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: parseRegisterName, parseRegister 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 381-400

```cpp
 381: 
 382:   if (Parser.getTok().is(AsmToken::Identifier)) {
 383:     // Check for register pair syntax
 384:     if (Parser.getLexer().peekTok().is(AsmToken::Colon)) {
 385:       AsmToken HighTok = Parser.getTok();
 386:       Parser.Lex();
 387:       AsmToken ColonTok = Parser.getTok();
 388:       Parser.Lex(); // Eat high (odd) register and colon
 389: 
 390:       if (Parser.getTok().is(AsmToken::Identifier)) {
 391:         // Convert lower (even) register to DREG
 392:         Reg = toDREG(parseRegisterName());
 393:       }
 394:       if (!Reg && RestoreOnFailure) {
 395:         getLexer().UnLex(std::move(ColonTok));
 396:         getLexer().UnLex(std::move(HighTok));
 397:       }
 398:     } else {
 399:       Reg = parseRegisterName();
 400:     }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 401-420

```cpp
 401:   }
 402:   return Reg;
 403: }
 404: 
 405: bool AVRAsmParser::tryParseRegisterOperand(OperandVector &Operands) {
 406:   MCRegister Reg = parseRegister();
 407: 
 408:   if (!Reg)
 409:     return true;
 410: 
 411:   // Reject R0~R15 on avrtiny.
 412:   if (AVR::R0 <= Reg && Reg <= AVR::R15 &&
 413:       STI->hasFeature(AVR::FeatureTinyEncoding))
 414:     return Error(Parser.getTok().getLoc(), "invalid register on avrtiny");
 415: 
 416:   AsmToken const &T = Parser.getTok();
 417:   Operands.push_back(AVROperand::CreateReg(Reg, T.getLoc(), T.getEndLoc()));
 418:   Parser.Lex(); // Eat register token.
 419: 
 420:   return false;
```

- EN: Function bodies or method definitions such as tryParseRegisterOperand contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: tryParseRegisterOperand 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 421-440

```cpp
 421: }
 422: 
 423: bool AVRAsmParser::tryParseExpression(OperandVector &Operands, int64_t offset) {
 424:   SMLoc S = Parser.getTok().getLoc();
 425: 
 426:   if (!tryParseRelocExpression(Operands))
 427:     return false;
 428: 
 429:   if ((Parser.getTok().getKind() == AsmToken::Plus ||
 430:        Parser.getTok().getKind() == AsmToken::Minus) &&
 431:       Parser.getLexer().peekTok().getKind() == AsmToken::Identifier) {
 432:     // Don't handle this case - it should be split into two
 433:     // separate tokens.
 434:     return true;
 435:   }
 436: 
 437:   // Parse (potentially inner) expression
 438:   MCExpr const *Expression;
 439:   if (getParser().parseExpression(Expression))
 440:     return true;
```

- EN: Function bodies or method definitions such as tryParseExpression contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: tryParseExpression 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 441-460

```cpp
 441: 
 442:   if (offset) {
 443:     Expression = MCBinaryExpr::createAdd(
 444:         Expression, MCConstantExpr::create(offset, getContext()), getContext());
 445:   }
 446: 
 447:   SMLoc E = SMLoc::getFromPointer(Parser.getTok().getLoc().getPointer() - 1);
 448:   Operands.push_back(AVROperand::CreateImm(Expression, S, E));
 449:   return false;
 450: }
 451: 
 452: bool AVRAsmParser::tryParseRelocExpression(OperandVector &Operands) {
 453:   bool isNegated = false;
 454:   AVR::Specifier ModifierKind = AVR::S_AVR_NONE;
 455: 
 456:   SMLoc S = Parser.getTok().getLoc();
 457: 
 458:   // Reject the form in which sign comes first. This behaviour is
 459:   // in accordance with avr-gcc.
 460:   AsmToken::TokenKind CurTok = Parser.getLexer().getKind();
```

- EN: Function bodies or method definitions such as tryParseRelocExpression contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: tryParseRelocExpression 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 461-480

```cpp
 461:   if (CurTok == AsmToken::Minus || CurTok == AsmToken::Plus)
 462:     return true;
 463: 
 464:   // Check for sign.
 465:   AsmToken tokens[2];
 466:   if (Parser.getLexer().peekTokens(tokens) == 2)
 467:     if (tokens[0].getKind() == AsmToken::LParen &&
 468:         tokens[1].getKind() == AsmToken::Minus)
 469:       isNegated = true;
 470: 
 471:   // Check if we have a target specific modifier (lo8, hi8, &c)
 472:   if (CurTok != AsmToken::Identifier ||
 473:       Parser.getLexer().peekTok().getKind() != AsmToken::LParen) {
 474:     // Not a reloc expr
 475:     return true;
 476:   }
 477:   StringRef ModifierName = Parser.getTok().getString();
 478:   ModifierKind = AVRMCExpr::parseSpecifier(ModifierName);
 479: 
 480:   if (ModifierKind != AVR::S_AVR_NONE) {
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 481-500

```cpp
 481:     Parser.Lex();
 482:     Parser.Lex(); // Eat modifier name and parenthesis
 483:     if (Parser.getTok().getString() == GENERATE_STUBS &&
 484:         Parser.getTok().getKind() == AsmToken::Identifier) {
 485:       std::string GSModName = ModifierName.str() + "_" + GENERATE_STUBS;
 486:       ModifierKind = AVRMCExpr::parseSpecifier(GSModName);
 487:       if (ModifierKind != AVR::S_AVR_NONE)
 488:         Parser.Lex(); // Eat gs modifier name
 489:     }
 490:   } else {
 491:     return Error(Parser.getTok().getLoc(), "unknown modifier");
 492:   }
 493: 
 494:   if (tokens[1].getKind() == AsmToken::Minus ||
 495:       tokens[1].getKind() == AsmToken::Plus) {
 496:     Parser.Lex();
 497:     assert(Parser.getTok().getKind() == AsmToken::LParen);
 498:     Parser.Lex(); // Eat the sign and parenthesis
 499:   }
 500: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 501-520

```cpp
 501:   MCExpr const *InnerExpression;
 502:   if (getParser().parseExpression(InnerExpression))
 503:     return true;
 504: 
 505:   if (tokens[1].getKind() == AsmToken::Minus ||
 506:       tokens[1].getKind() == AsmToken::Plus) {
 507:     assert(Parser.getTok().getKind() == AsmToken::RParen);
 508:     Parser.Lex(); // Eat closing parenthesis
 509:   }
 510: 
 511:   // If we have a modifier wrap the inner expression
 512:   assert(Parser.getTok().getKind() == AsmToken::RParen);
 513:   Parser.Lex(); // Eat closing parenthesis
 514: 
 515:   MCExpr const *Expression =
 516:       AVRMCExpr::create(ModifierKind, InnerExpression, isNegated, getContext());
 517: 
 518:   SMLoc E = SMLoc::getFromPointer(Parser.getTok().getLoc().getPointer() - 1);
 519:   Operands.push_back(AVROperand::CreateImm(Expression, S, E));
 520: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 521-540

```cpp
 521:   return false;
 522: }
 523: 
 524: bool AVRAsmParser::parseOperand(OperandVector &Operands, bool maybeReg) {
 525:   LLVM_DEBUG(dbgs() << "parseOperand\n");
 526: 
 527:   switch (getLexer().getKind()) {
 528:   default:
 529:     return Error(Parser.getTok().getLoc(), "unexpected token in operand");
 530: 
 531:   case AsmToken::Identifier:
 532:     // Try to parse a register, fall through to the next case if it fails.
 533:     if (maybeReg && !tryParseRegisterOperand(Operands)) {
 534:       return false;
 535:     }
 536:     [[fallthrough]];
 537:   case AsmToken::LParen:
 538:   case AsmToken::Integer:
 539:     return tryParseExpression(Operands, 0);
 540:   case AsmToken::Dot:
```

- EN: Function bodies or method definitions such as parseOperand contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: parseOperand 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 541-560

```cpp
 541:     return tryParseExpression(Operands, 2);
 542:   case AsmToken::Plus:
 543:   case AsmToken::Minus: {
 544:     // If the sign preceeds a number, parse the number,
 545:     // otherwise treat the sign a an independent token.
 546:     switch (getLexer().peekTok().getKind()) {
 547:     case AsmToken::Integer:
 548:     case AsmToken::BigNum:
 549:     case AsmToken::Identifier:
 550:     case AsmToken::Real:
 551:       if (!tryParseExpression(Operands, 0))
 552:         return false;
 553:       break;
 554:     default:
 555:       break;
 556:     }
 557:     // Treat the token as an independent token.
 558:     Operands.push_back(AVROperand::CreateToken(Parser.getTok().getString(),
 559:                                                Parser.getTok().getLoc()));
 560:     Parser.Lex(); // Eat the token.
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 561-580

```cpp
 561:     return false;
 562:   }
 563:   }
 564: 
 565:   // Could not parse operand
 566:   return true;
 567: }
 568: 
 569: ParseStatus AVRAsmParser::parseMemriOperand(OperandVector &Operands) {
 570:   LLVM_DEBUG(dbgs() << "parseMemriOperand()\n");
 571: 
 572:   SMLoc E, S;
 573:   MCExpr const *Expression;
 574:   MCRegister Reg;
 575: 
 576:   // Parse register.
 577:   {
 578:     Reg = parseRegister();
 579: 
 580:     if (!Reg)
```

- EN: Function bodies or method definitions such as parseMemriOperand contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: parseMemriOperand 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 581-600

```cpp
 581:       return ParseStatus::Failure;
 582: 
 583:     S = SMLoc::getFromPointer(Parser.getTok().getLoc().getPointer() - 1);
 584:     Parser.Lex(); // Eat register token.
 585:   }
 586: 
 587:   // Parse immediate;
 588:   {
 589:     if (getParser().parseExpression(Expression))
 590:       return ParseStatus::Failure;
 591: 
 592:     E = SMLoc::getFromPointer(Parser.getTok().getLoc().getPointer() - 1);
 593:   }
 594: 
 595:   Operands.push_back(AVROperand::CreateMemri(Reg, Expression, S, E));
 596: 
 597:   return ParseStatus::Success;
 598: }
 599: 
 600: bool AVRAsmParser::parseRegister(MCRegister &Reg, SMLoc &StartLoc,
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 601-620

```cpp
 601:                                  SMLoc &EndLoc) {
 602:   StartLoc = Parser.getTok().getLoc();
 603:   Reg = parseRegister(/*RestoreOnFailure=*/false);
 604:   EndLoc = Parser.getTok().getLoc();
 605: 
 606:   return Reg == AVR::NoRegister;
 607: }
 608: 
 609: ParseStatus AVRAsmParser::tryParseRegister(MCRegister &Reg, SMLoc &StartLoc,
 610:                                            SMLoc &EndLoc) {
 611:   StartLoc = Parser.getTok().getLoc();
 612:   Reg = parseRegister(/*RestoreOnFailure=*/true);
 613:   EndLoc = Parser.getTok().getLoc();
 614: 
 615:   if (Reg == AVR::NoRegister)
 616:     return ParseStatus::NoMatch;
 617:   return ParseStatus::Success;
 618: }
 619: 
 620: void AVRAsmParser::eatComma() {
```

- EN: Function bodies or method definitions such as tryParseRegister, eatComma contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: tryParseRegister, eatComma 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 621-640

```cpp
 621:   if (getLexer().is(AsmToken::Comma)) {
 622:     Parser.Lex();
 623:   } else {
 624:     // GCC allows commas to be omitted.
 625:   }
 626: }
 627: 
 628: bool AVRAsmParser::parseInstruction(ParseInstructionInfo &Info,
 629:                                     StringRef Mnemonic, SMLoc NameLoc,
 630:                                     OperandVector &Operands) {
 631:   Operands.push_back(AVROperand::CreateToken(Mnemonic, NameLoc));
 632: 
 633:   int OperandNum = -1;
 634:   while (getLexer().isNot(AsmToken::EndOfStatement)) {
 635:     OperandNum++;
 636:     if (OperandNum > 0)
 637:       eatComma();
 638: 
 639:     ParseStatus ParseRes = MatchOperandParserImpl(Operands, Mnemonic);
 640: 
```

- EN: Function bodies or method definitions such as parseInstruction contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: parseInstruction 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 641-660

```cpp
 641:     if (ParseRes.isSuccess())
 642:       continue;
 643: 
 644:     if (ParseRes.isFailure()) {
 645:       SMLoc Loc = getLexer().getLoc();
 646:       Parser.eatToEndOfStatement();
 647: 
 648:       return Error(Loc, "failed to parse register and immediate pair");
 649:     }
 650: 
 651:     // These specific operands should be treated as addresses/symbols/labels,
 652:     // other than registers.
 653:     bool maybeReg = true;
 654: 
 655:     if (OperandNum == 1) {
 656:       std::array<StringRef, 8> Insts = {"lds", "adiw", "sbiw", "ldi"};
 657:       for (auto Inst : Insts) {
 658:         if (Inst == Mnemonic) {
 659:           maybeReg = false;
 660:           break;
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 661-680

```cpp
 661:         }
 662:       }
 663:     } else if (OperandNum == 0) {
 664:       std::array<StringRef, 8> Insts = {"sts", "call", "rcall", "rjmp", "jmp"};
 665:       for (auto Inst : Insts) {
 666:         if (Inst == Mnemonic) {
 667:           maybeReg = false;
 668:           break;
 669:         }
 670:       }
 671:     }
 672: 
 673:     if (parseOperand(Operands, maybeReg)) {
 674:       SMLoc Loc = getLexer().getLoc();
 675:       Parser.eatToEndOfStatement();
 676:       return Error(Loc, "unexpected token in argument list");
 677:     }
 678:   }
 679:   Parser.Lex(); // Consume the EndOfStatement
 680:   return false;
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 681-700

```cpp
 681: }
 682: 
 683: ParseStatus AVRAsmParser::parseDirective(llvm::AsmToken DirectiveID) {
 684:   StringRef IDVal = DirectiveID.getIdentifier();
 685:   if (IDVal.lower() == ".long")
 686:     return parseLiteralValues(SIZE_LONG, DirectiveID.getLoc());
 687:   if (IDVal.lower() == ".word" || IDVal.lower() == ".short")
 688:     return parseLiteralValues(SIZE_WORD, DirectiveID.getLoc());
 689:   if (IDVal.lower() == ".byte")
 690:     return parseLiteralValues(1, DirectiveID.getLoc());
 691:   return ParseStatus::NoMatch;
 692: }
 693: 
 694: ParseStatus AVRAsmParser::parseLiteralValues(unsigned SizeInBytes, SMLoc L) {
 695:   MCAsmParser &Parser = getParser();
 696:   AVRMCELFStreamer &AVRStreamer =
 697:       static_cast<AVRMCELFStreamer &>(Parser.getStreamer());
 698:   AsmToken Tokens[2];
 699:   size_t ReadCount = Parser.getLexer().peekTokens(Tokens);
 700:   if (ReadCount == 2 && Parser.getTok().getKind() == AsmToken::Identifier &&
```

- EN: Function bodies or method definitions such as parseDirective, parseLiteralValues contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: parseDirective, parseLiteralValues 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 701-720

```cpp
 701:       Tokens[0].getKind() == AsmToken::Minus &&
 702:       Tokens[1].getKind() == AsmToken::Identifier) {
 703:     MCSymbol *Symbol = getContext().getOrCreateSymbol(".text");
 704:     AVRStreamer.emitValueForModiferKind(Symbol, SizeInBytes, L,
 705:                                         AVR::S_AVR_NONE);
 706:     return ParseStatus::NoMatch;
 707:   }
 708: 
 709:   if (Parser.getTok().getKind() == AsmToken::Identifier &&
 710:       Parser.getLexer().peekTok().getKind() == AsmToken::LParen) {
 711:     StringRef ModifierName = Parser.getTok().getString();
 712:     AVR::Specifier Spec = AVRMCExpr::parseSpecifier(ModifierName);
 713:     if (Spec != AVR::S_AVR_NONE) {
 714:       Parser.Lex();
 715:       Parser.Lex(); // Eat the modifier and parenthesis
 716:     } else {
 717:       return Error(Parser.getTok().getLoc(), "unknown modifier");
 718:     }
 719:     MCSymbol *Symbol =
 720:         getContext().getOrCreateSymbol(Parser.getTok().getString());
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 721-740

```cpp
 721:     AVRStreamer.emitValueForModiferKind(Symbol, SizeInBytes, L, Spec);
 722:     Lex(); // Eat the symbol name.
 723:     if (parseToken(AsmToken::RParen))
 724:       return ParseStatus::Failure;
 725:     return parseEOL();
 726:   }
 727: 
 728:   auto parseOne = [&]() -> bool {
 729:     const MCExpr *Value;
 730:     if (Parser.parseExpression(Value))
 731:       return true;
 732:     Parser.getStreamer().emitValue(Value, SizeInBytes, L);
 733:     return false;
 734:   };
 735:   return (parseMany(parseOne));
 736: }
 737: 
 738: extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void LLVMInitializeAVRAsmParser() {
 739:   RegisterMCAsmParser<AVRAsmParser> X(getTheAVRTarget());
 740: }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 741-760

```cpp
 741: 
 742: #define GET_REGISTER_MATCHER
 743: #define GET_MATCHER_IMPLEMENTATION
 744: #include "AVRGenAsmMatcher.inc"
 745: 
 746: // Uses enums defined in AVRGenAsmMatcher.inc
 747: unsigned AVRAsmParser::validateTargetOperandClass(MCParsedAsmOperand &AsmOp,
 748:                                                   unsigned ExpectedKind) {
 749:   AVROperand &Op = static_cast<AVROperand &>(AsmOp);
 750:   MatchClassKind Expected = static_cast<MatchClassKind>(ExpectedKind);
 751: 
 752:   // If need be, GCC converts bare numbers to register names
 753:   // It's ugly, but GCC supports it.
 754:   if (Op.isImm()) {
 755:     if (MCConstantExpr const *Const = dyn_cast<MCConstantExpr>(Op.getImm())) {
 756:       int64_t RegNum = Const->getValue();
 757: 
 758:       // Reject R0~R15 on avrtiny.
 759:       if (0 <= RegNum && RegNum <= 15 &&
 760:           STI->hasFeature(AVR::FeatureTinyEncoding))
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. Function bodies or method definitions such as validateTargetOperandClass contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 validateTargetOperandClass 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 761-780

```cpp
 761:         return Match_InvalidRegisterOnTiny;
 762: 
 763:       std::ostringstream RegName;
 764:       RegName << "r" << RegNum;
 765:       if (MCRegister Reg = MatchRegisterName(RegName.str())) {
 766:         Op.makeReg(Reg);
 767:         if (validateOperandClass(Op, Expected, *STI) == Match_Success) {
 768:           return Match_Success;
 769:         }
 770:       }
 771:       // Let the other quirks try their magic.
 772:     }
 773:   }
 774: 
 775:   if (Op.isReg()) {
 776:     // If the instruction uses a register pair but we got a single, lower
 777:     // register we perform a "class cast".
 778:     if (isSubclass(Expected, MCK_DREGS)) {
 779:       MCRegister correspondingDREG = toDREG(Op.getReg());
 780: 
```

- EN: This chunk introduces interfaces or data structures such as cast, which organize the target-specific behavior exposed by the file. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 这一段引入了 cast 等接口或数据结构，用于组织该文件暴露的目标专用行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 781-788

```cpp
 781:       if (correspondingDREG) {
 782:         Op.makeReg(correspondingDREG);
 783:         return validateOperandClass(Op, Expected, *STI);
 784:       }
 785:     }
 786:   }
 787:   return Match_InvalidOperand;
 788: }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

## Key Concepts / 关键概念

- Assembly syntax decoding / 汇编语法解码
- Operand construction / 操作数构造
- MC instruction representation / MC 指令表示
- MC streaming and emission / MC 流式输出
- Register classes / 寄存器类
- CPU feature modelling / CPU 特性建模
- Assembly parsing / 汇编解析
- ELF object support / ELF 目标文件支持

## Dependencies / 依赖关系

- Direct includes / 直接包含: `AVRRegisterInfo.h`, `MCTargetDesc/AVRMCAsmInfo.h`, `MCTargetDesc/AVRMCELFStreamer.h`, `MCTargetDesc/AVRMCTargetDesc.h`, `TargetInfo/AVRTargetInfo.h`, `llvm/ADT/APInt.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h`
- LLVM subsystems / LLVM 子系统: LLVM MC
- Generated or companion files / 生成或配套文件: `AVRGenAsmMatcher.inc`
