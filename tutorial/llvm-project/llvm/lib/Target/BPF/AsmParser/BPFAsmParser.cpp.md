# BPFAsmParser.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/BPF/AsmParser/BPFAsmParser.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Implements the target-specific assembly parser that turns textual assembly into MC operands and instructions.
- 目的（中文）: 实现目标专用的汇编解析器，将文本汇编转换为 MC 操作数与指令。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- BPFAsmParser.cpp - Parse BPF assembly to MCInst instructions --===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: #include "MCTargetDesc/BPFMCAsmInfo.h"
  10: #include "MCTargetDesc/BPFMCTargetDesc.h"
  11: #include "TargetInfo/BPFTargetInfo.h"
  12: #include "llvm/ADT/StringSwitch.h"
  13: #include "llvm/MC/MCContext.h"
  14: #include "llvm/MC/MCExpr.h"
  15: #include "llvm/MC/MCInst.h"
  16: #include "llvm/MC/MCInstrInfo.h"
  17: #include "llvm/MC/MCParser/AsmLexer.h"
  18: #include "llvm/MC/MCParser/MCParsedAsmOperand.h"
  19: #include "llvm/MC/MCParser/MCTargetAsmParser.h"
  20: #include "llvm/MC/MCStreamer.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 21-40

```cpp
  21: #include "llvm/MC/MCSubtargetInfo.h"
  22: #include "llvm/MC/TargetRegistry.h"
  23: #include "llvm/Support/Casting.h"
  24: #include "llvm/Support/Compiler.h"
  25: 
  26: using namespace llvm;
  27: 
  28: namespace {
  29: struct BPFOperand;
  30: 
  31: class BPFAsmParser : public MCTargetAsmParser {
  32: 
  33:   SMLoc getLoc() const { return getParser().getTok().getLoc(); }
  34: 
  35:   bool PreMatchCheck(OperandVector &Operands);
  36: 
  37:   bool matchAndEmitInstruction(SMLoc IDLoc, unsigned &Opcode,
  38:                                OperandVector &Operands, MCStreamer &Out,
  39:                                uint64_t &ErrorInfo,
  40:                                bool MatchingInlineAsm) override;
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. This chunk introduces interfaces or data structures such as BPFOperand, BPFAsmParser, which organize the target-specific behavior exposed by the file. Function bodies or method definitions such as getLoc contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 这一段引入了 BPFOperand, BPFAsmParser 等接口或数据结构，用于组织该文件暴露的目标专用行为。 getLoc 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 41-60

```cpp
  41: 
  42:   bool parseRegister(MCRegister &Reo, SMLoc &StartLoc, SMLoc &EndLoc) override;
  43:   ParseStatus tryParseRegister(MCRegister &Reg, SMLoc &StartLoc,
  44:                                SMLoc &EndLoc) override;
  45: 
  46:   bool parseInstruction(ParseInstructionInfo &Info, StringRef Name,
  47:                         SMLoc NameLoc, OperandVector &Operands) override;
  48: 
  49:   // "=" is used as assignment operator for assembly statment, so can't be used
  50:   // for symbol assignment.
  51:   bool equalIsAsmAssignment() override { return false; }
  52:   // "*" is used for dereferencing memory that it will be the start of
  53:   // statement.
  54:   bool tokenIsStartOfStatement(AsmToken::TokenKind Token) override {
  55:     return Token == AsmToken::Star;
  56:   }
  57: 
  58: #define GET_ASSEMBLER_HEADER
  59: #include "BPFGenAsmMatcher.inc"
  60: 
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 61-80

```cpp
  61:   ParseStatus parseImmediate(OperandVector &Operands);
  62:   ParseStatus parseRegister(OperandVector &Operands);
  63:   ParseStatus parseOperandAsOperator(OperandVector &Operands);
  64: 
  65: public:
  66:   enum BPFMatchResultTy {
  67:     Match_Dummy = FIRST_TARGET_MATCH_RESULT_TY,
  68: #define GET_OPERAND_DIAGNOSTIC_TYPES
  69: #include "BPFGenAsmMatcher.inc"
  70: #undef GET_OPERAND_DIAGNOSTIC_TYPES
  71:   };
  72: 
  73:   BPFAsmParser(const MCSubtargetInfo &STI, MCAsmParser &Parser,
  74:                const MCInstrInfo &MII)
  75:       : MCTargetAsmParser(STI, MII) {
  76:     setAvailableFeatures(ComputeAvailableFeatures(STI.getFeatureBits()));
  77:   }
  78: };
  79: 
  80: /// BPFOperand - Instances of this class represent a parsed machine
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. This chunk introduces interfaces or data structures such as represent, which organize the target-specific behavior exposed by the file. Function bodies or method definitions such as MCTargetAsmParser contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 这一段引入了 represent 等接口或数据结构，用于组织该文件暴露的目标专用行为。 MCTargetAsmParser 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 81-100

```cpp
  81: /// instruction
  82: struct BPFOperand : public MCParsedAsmOperand {
  83: 
  84:   enum KindTy {
  85:     Token,
  86:     Register,
  87:     Immediate,
  88:   } Kind;
  89: 
  90:   struct RegOp {
  91:     MCRegister RegNum;
  92:   };
  93: 
  94:   struct ImmOp {
  95:     const MCExpr *Val;
  96:   };
  97: 
  98:   SMLoc StartLoc, EndLoc;
  99:   union {
 100:     StringRef Tok;
```

- EN: This chunk introduces interfaces or data structures such as BPFOperand, RegOp, ImmOp, which organize the target-specific behavior exposed by the file. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 这一段引入了 BPFOperand, RegOp, ImmOp 等接口或数据结构，用于组织该文件暴露的目标专用行为。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 101-120

```cpp
 101:     RegOp Reg;
 102:     ImmOp Imm;
 103:   };
 104: 
 105:   BPFOperand(KindTy K) : Kind(K) {}
 106: 
 107: public:
 108:   BPFOperand(const BPFOperand &o) : MCParsedAsmOperand() {
 109:     Kind = o.Kind;
 110:     StartLoc = o.StartLoc;
 111:     EndLoc = o.EndLoc;
 112: 
 113:     switch (Kind) {
 114:     case Register:
 115:       Reg = o.Reg;
 116:       break;
 117:     case Immediate:
 118:       Imm = o.Imm;
 119:       break;
 120:     case Token:
```

- EN: Function bodies or method definitions such as BPFOperand contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: BPFOperand 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 121-140

```cpp
 121:       Tok = o.Tok;
 122:       break;
 123:     }
 124:   }
 125: 
 126:   bool isToken() const override { return Kind == Token; }
 127:   bool isReg() const override { return Kind == Register; }
 128:   bool isImm() const override { return Kind == Immediate; }
 129:   bool isMem() const override { return false; }
 130: 
 131:   bool isConstantImm() const {
 132:     return isImm() && isa<MCConstantExpr>(getImm());
 133:   }
 134: 
 135:   int64_t getConstantImm() const {
 136:     const MCExpr *Val = getImm();
 137:     return static_cast<const MCConstantExpr *>(Val)->getValue();
 138:   }
 139: 
 140:   bool isSImm16() const {
```

- EN: Function bodies or method definitions such as isConstantImm, getConstantImm, isSImm16 contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: isConstantImm, getConstantImm, isSImm16 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 141-160

```cpp
 141:     return (isConstantImm() && isInt<16>(getConstantImm()));
 142:   }
 143: 
 144:   bool isSymbolRef() const { return isImm() && isa<MCSymbolRefExpr>(getImm()); }
 145: 
 146:   bool isBrTarget() const { return isSymbolRef() || isSImm16(); }
 147: 
 148:   /// getStartLoc - Gets location of the first token of this operand
 149:   SMLoc getStartLoc() const override { return StartLoc; }
 150:   /// getEndLoc - Gets location of the last token of this operand
 151:   SMLoc getEndLoc() const override { return EndLoc; }
 152: 
 153:   MCRegister getReg() const override {
 154:     assert(Kind == Register && "Invalid type access!");
 155:     return Reg.RegNum;
 156:   }
 157: 
 158:   const MCExpr *getImm() const {
 159:     assert(Kind == Immediate && "Invalid type access!");
 160:     return Imm.Val;
```

- EN: Function bodies or method definitions such as isSymbolRef, isBrTarget contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: isSymbolRef, isBrTarget 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 161-180

```cpp
 161:   }
 162: 
 163:   StringRef getToken() const {
 164:     assert(Kind == Token && "Invalid type access!");
 165:     return Tok;
 166:   }
 167: 
 168:   void print(raw_ostream &OS, const MCAsmInfo &MAI) const override {
 169:     switch (Kind) {
 170:     case Immediate:
 171:       MAI.printExpr(OS, *getImm());
 172:       break;
 173:     case Register:
 174:       OS << "<register x";
 175:       OS << getReg().id() << ">";
 176:       break;
 177:     case Token:
 178:       OS << "'" << getToken() << "'";
 179:       break;
 180:     }
```

- EN: Function bodies or method definitions such as getToken contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: getToken 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 181-200

```cpp
 181:   }
 182: 
 183:   void addExpr(MCInst &Inst, const MCExpr *Expr) const {
 184:     assert(Expr && "Expr shouldn't be null!");
 185: 
 186:     if (auto *CE = dyn_cast<MCConstantExpr>(Expr))
 187:       Inst.addOperand(MCOperand::createImm(CE->getValue()));
 188:     else
 189:       Inst.addOperand(MCOperand::createExpr(Expr));
 190:   }
 191: 
 192:   // Used by the TableGen Code
 193:   void addRegOperands(MCInst &Inst, unsigned N) const {
 194:     assert(N == 1 && "Invalid number of operands!");
 195:     Inst.addOperand(MCOperand::createReg(getReg()));
 196:   }
 197: 
 198:   void addImmOperands(MCInst &Inst, unsigned N) const {
 199:     assert(N == 1 && "Invalid number of operands!");
 200:     addExpr(Inst, getImm());
```

- EN: Function bodies or method definitions such as addExpr, addRegOperands, addImmOperands contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: addExpr, addRegOperands, addImmOperands 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 201-220

```cpp
 201:   }
 202: 
 203:   static std::unique_ptr<BPFOperand> createToken(StringRef Str, SMLoc S) {
 204:     auto Op = std::make_unique<BPFOperand>(Token);
 205:     Op->Tok = Str;
 206:     Op->StartLoc = S;
 207:     Op->EndLoc = S;
 208:     return Op;
 209:   }
 210: 
 211:   static std::unique_ptr<BPFOperand> createReg(MCRegister Reg, SMLoc S,
 212:                                                SMLoc E) {
 213:     auto Op = std::make_unique<BPFOperand>(Register);
 214:     Op->Reg.RegNum = Reg;
 215:     Op->StartLoc = S;
 216:     Op->EndLoc = E;
 217:     return Op;
 218:   }
 219: 
 220:   static std::unique_ptr<BPFOperand> createImm(const MCExpr *Val, SMLoc S,
```

- EN: Function bodies or method definitions such as createToken, createReg contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: createToken, createReg 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 221-240

```cpp
 221:                                                SMLoc E) {
 222:     auto Op = std::make_unique<BPFOperand>(Immediate);
 223:     Op->Imm.Val = Val;
 224:     Op->StartLoc = S;
 225:     Op->EndLoc = E;
 226:     return Op;
 227:   }
 228: 
 229:   // Identifiers that can be used at the start of a statment.
 230:   static bool isValidIdAtStart(StringRef Name) {
 231:     return StringSwitch<bool>(Name.lower())
 232:         .Case("if", true)
 233:         .Case("call", true)
 234:         .Case("callx", true)
 235:         .Case("goto", true)
 236:         .Case("gotol", true)
 237:         .Case("gotox", true)
 238:         .Case("may_goto", true)
 239:         .Case("*", true)
 240:         .Case("exit", true)
```

- EN: Function bodies or method definitions such as isValidIdAtStart contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: isValidIdAtStart 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 241-260

```cpp
 241:         .Case("lock", true)
 242:         .Case("ld_pseudo", true)
 243:         .Case("store_release", true)
 244:         .Default(false);
 245:   }
 246: 
 247:   // Identifiers that can be used in the middle of a statment.
 248:   static bool isValidIdInMiddle(StringRef Name) {
 249:     return StringSwitch<bool>(Name.lower())
 250:         .Case("u64", true)
 251:         .Case("u32", true)
 252:         .Case("u16", true)
 253:         .Case("u8", true)
 254:         .Case("s32", true)
 255:         .Case("s16", true)
 256:         .Case("s8", true)
 257:         .Case("be64", true)
 258:         .Case("be32", true)
 259:         .Case("be16", true)
 260:         .Case("le64", true)
```

- EN: Function bodies or method definitions such as isValidIdInMiddle contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: isValidIdInMiddle 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 261-280

```cpp
 261:         .Case("le32", true)
 262:         .Case("le16", true)
 263:         .Case("bswap16", true)
 264:         .Case("bswap32", true)
 265:         .Case("bswap64", true)
 266:         .Case("goto", true)
 267:         .Case("ll", true)
 268:         .Case("skb", true)
 269:         .Case("s", true)
 270:         .Case("atomic_fetch_add", true)
 271:         .Case("atomic_fetch_and", true)
 272:         .Case("atomic_fetch_or", true)
 273:         .Case("atomic_fetch_xor", true)
 274:         .Case("xchg_64", true)
 275:         .Case("xchg32_32", true)
 276:         .Case("cmpxchg_64", true)
 277:         .Case("cmpxchg32_32", true)
 278:         .Case("addr_space_cast", true)
 279:         .Case("load_acquire", true)
 280:         .Default(false);
```

- EN: In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 281-300

```cpp
 281:   }
 282: };
 283: } // end anonymous namespace.
 284: 
 285: #define GET_REGISTER_MATCHER
 286: #define GET_MATCHER_IMPLEMENTATION
 287: #include "BPFGenAsmMatcher.inc"
 288: 
 289: bool BPFAsmParser::PreMatchCheck(OperandVector &Operands) {
 290: 
 291:   if (Operands.size() == 4) {
 292:     // check "reg1 = -reg2" and "reg1 = be16/be32/be64/le16/le32/le64 reg2",
 293:     // reg1 must be the same as reg2
 294:     BPFOperand &Op0 = (BPFOperand &)*Operands[0];
 295:     BPFOperand &Op1 = (BPFOperand &)*Operands[1];
 296:     BPFOperand &Op2 = (BPFOperand &)*Operands[2];
 297:     BPFOperand &Op3 = (BPFOperand &)*Operands[3];
 298:     if (Op0.isReg() && Op1.isToken() && Op2.isToken() && Op3.isReg()
 299:         && Op1.getToken() == "="
 300:         && (Op2.getToken() == "-" || Op2.getToken() == "be16"
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. Function bodies or method definitions such as PreMatchCheck contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 PreMatchCheck 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 301-320

```cpp
 301:             || Op2.getToken() == "be32" || Op2.getToken() == "be64"
 302:             || Op2.getToken() == "le16" || Op2.getToken() == "le32"
 303:             || Op2.getToken() == "le64")
 304:         && Op0.getReg() != Op3.getReg())
 305:       return true;
 306:   }
 307: 
 308:   return false;
 309: }
 310: 
 311: bool BPFAsmParser::matchAndEmitInstruction(SMLoc IDLoc, unsigned &Opcode,
 312:                                            OperandVector &Operands,
 313:                                            MCStreamer &Out, uint64_t &ErrorInfo,
 314:                                            bool MatchingInlineAsm) {
 315:   MCInst Inst;
 316:   SMLoc ErrorLoc;
 317: 
 318:   if (PreMatchCheck(Operands))
 319:     return Error(IDLoc, "additional inst constraint not met");
 320: 
```

- EN: Function bodies or method definitions such as matchAndEmitInstruction contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: matchAndEmitInstruction 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 321-340

```cpp
 321:   switch (MatchInstructionImpl(Operands, Inst, ErrorInfo, MatchingInlineAsm)) {
 322:   default:
 323:     break;
 324:   case Match_Success:
 325:     Inst.setLoc(IDLoc);
 326:     Out.emitInstruction(Inst, getSTI());
 327:     return false;
 328:   case Match_MissingFeature:
 329:     return Error(IDLoc, "instruction use requires an option to be enabled");
 330:   case Match_MnemonicFail:
 331:     return Error(IDLoc, "unrecognized instruction mnemonic");
 332:   case Match_InvalidOperand:
 333:     ErrorLoc = IDLoc;
 334: 
 335:     if (ErrorInfo != ~0U) {
 336:       if (ErrorInfo >= Operands.size())
 337:         return Error(ErrorLoc, "too few operands for instruction");
 338: 
 339:       ErrorLoc = ((BPFOperand &)*Operands[ErrorInfo]).getStartLoc();
 340: 
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 341-360

```cpp
 341:       if (ErrorLoc == SMLoc())
 342:         ErrorLoc = IDLoc;
 343:     }
 344: 
 345:     return Error(ErrorLoc, "invalid operand for instruction");
 346:   case Match_InvalidBrTarget:
 347:     return Error(Operands[ErrorInfo]->getStartLoc(),
 348:                  "operand is not an identifier or 16-bit signed integer");
 349:   case Match_InvalidSImm16:
 350:     return Error(Operands[ErrorInfo]->getStartLoc(),
 351:                  "operand is not a 16-bit signed integer");
 352:   case Match_InvalidTiedOperand:
 353:     return Error(Operands[ErrorInfo]->getStartLoc(),
 354:                  "operand is not the same as the dst register");
 355:   }
 356: 
 357:   llvm_unreachable("Unknown match type detected!");
 358: }
 359: 
 360: bool BPFAsmParser::parseRegister(MCRegister &Reg, SMLoc &StartLoc,
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 361-380

```cpp
 361:                                  SMLoc &EndLoc) {
 362:   if (!tryParseRegister(Reg, StartLoc, EndLoc).isSuccess())
 363:     return Error(StartLoc, "invalid register name");
 364:   return false;
 365: }
 366: 
 367: ParseStatus BPFAsmParser::tryParseRegister(MCRegister &Reg, SMLoc &StartLoc,
 368:                                            SMLoc &EndLoc) {
 369:   const AsmToken &Tok = getParser().getTok();
 370:   StartLoc = Tok.getLoc();
 371:   EndLoc = Tok.getEndLoc();
 372:   Reg = BPF::NoRegister;
 373:   StringRef Name = getLexer().getTok().getIdentifier();
 374: 
 375:   if (!MatchRegisterName(Name)) {
 376:     getParser().Lex(); // Eat identifier token.
 377:     return ParseStatus::Success;
 378:   }
 379: 
 380:   return ParseStatus::NoMatch;
```

- EN: Function bodies or method definitions such as tryParseRegister contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: tryParseRegister 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 381-400

```cpp
 381: }
 382: 
 383: ParseStatus BPFAsmParser::parseOperandAsOperator(OperandVector &Operands) {
 384:   SMLoc S = getLoc();
 385: 
 386:   if (getLexer().getKind() == AsmToken::Identifier) {
 387:     StringRef Name = getLexer().getTok().getIdentifier();
 388: 
 389:     if (BPFOperand::isValidIdInMiddle(Name)) {
 390:       getLexer().Lex();
 391:       Operands.push_back(BPFOperand::createToken(Name, S));
 392:       return ParseStatus::Success;
 393:     }
 394: 
 395:     return ParseStatus::NoMatch;
 396:   }
 397: 
 398:   switch (getLexer().getKind()) {
 399:   case AsmToken::Minus:
 400:   case AsmToken::Plus: {
```

- EN: Function bodies or method definitions such as parseOperandAsOperator contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: parseOperandAsOperator 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 401-420

```cpp
 401:     if (getLexer().peekTok().is(AsmToken::Integer))
 402:       return ParseStatus::NoMatch;
 403:     [[fallthrough]];
 404:   }
 405: 
 406:   case AsmToken::Equal:
 407:   case AsmToken::Greater:
 408:   case AsmToken::Less:
 409:   case AsmToken::Pipe:
 410:   case AsmToken::Star:
 411:   case AsmToken::LParen:
 412:   case AsmToken::RParen:
 413:   case AsmToken::LBrac:
 414:   case AsmToken::RBrac:
 415:   case AsmToken::Slash:
 416:   case AsmToken::Amp:
 417:   case AsmToken::Percent:
 418:   case AsmToken::Caret: {
 419:     StringRef Name = getLexer().getTok().getString();
 420:     getLexer().Lex();
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 421-440

```cpp
 421:     Operands.push_back(BPFOperand::createToken(Name, S));
 422: 
 423:     return ParseStatus::Success;
 424:   }
 425: 
 426:   case AsmToken::EqualEqual:
 427:   case AsmToken::ExclaimEqual:
 428:   case AsmToken::GreaterEqual:
 429:   case AsmToken::GreaterGreater:
 430:   case AsmToken::LessEqual:
 431:   case AsmToken::LessLess: {
 432:     Operands.push_back(BPFOperand::createToken(
 433:         getLexer().getTok().getString().substr(0, 1), S));
 434:     Operands.push_back(BPFOperand::createToken(
 435:         getLexer().getTok().getString().substr(1, 1), S));
 436:     getLexer().Lex();
 437: 
 438:     return ParseStatus::Success;
 439:   }
 440: 
```

- EN: In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 441-460

```cpp
 441:   default:
 442:     break;
 443:   }
 444: 
 445:   return ParseStatus::NoMatch;
 446: }
 447: 
 448: ParseStatus BPFAsmParser::parseRegister(OperandVector &Operands) {
 449:   SMLoc S = getLoc();
 450:   SMLoc E = SMLoc::getFromPointer(S.getPointer() - 1);
 451: 
 452:   switch (getLexer().getKind()) {
 453:   default:
 454:     return ParseStatus::NoMatch;
 455:   case AsmToken::Identifier:
 456:     StringRef Name = getLexer().getTok().getIdentifier();
 457:     MCRegister Reg = MatchRegisterName(Name);
 458: 
 459:     if (!Reg)
 460:       return ParseStatus::NoMatch;
```

- EN: Function bodies or method definitions such as parseRegister contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: parseRegister 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 461-480

```cpp
 461: 
 462:     getLexer().Lex();
 463:     Operands.push_back(BPFOperand::createReg(Reg, S, E));
 464:   }
 465:   return ParseStatus::Success;
 466: }
 467: 
 468: ParseStatus BPFAsmParser::parseImmediate(OperandVector &Operands) {
 469:   switch (getLexer().getKind()) {
 470:   default:
 471:     return ParseStatus::NoMatch;
 472:   case AsmToken::LParen:
 473:   case AsmToken::Minus:
 474:   case AsmToken::Plus:
 475:   case AsmToken::Integer:
 476:   case AsmToken::String:
 477:   case AsmToken::Identifier:
 478:     break;
 479:   }
 480: 
```

- EN: Function bodies or method definitions such as parseImmediate contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: parseImmediate 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 481-500

```cpp
 481:   const MCExpr *IdVal;
 482:   SMLoc S = getLoc();
 483: 
 484:   if (getParser().parseExpression(IdVal))
 485:     return ParseStatus::Failure;
 486: 
 487:   SMLoc E = SMLoc::getFromPointer(S.getPointer() - 1);
 488:   Operands.push_back(BPFOperand::createImm(IdVal, S, E));
 489: 
 490:   return ParseStatus::Success;
 491: }
 492: 
 493: /// Parse an BPF instruction which is in BPF verifier format.
 494: bool BPFAsmParser::parseInstruction(ParseInstructionInfo &Info, StringRef Name,
 495:                                     SMLoc NameLoc, OperandVector &Operands) {
 496:   // The first operand could be either register or actually an operator.
 497:   MCRegister Reg = MatchRegisterName(Name);
 498: 
 499:   if (Reg) {
 500:     SMLoc E = SMLoc::getFromPointer(NameLoc.getPointer() - 1);
```

- EN: Function bodies or method definitions such as parseInstruction contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: parseInstruction 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 501-520

```cpp
 501:     Operands.push_back(BPFOperand::createReg(Reg, NameLoc, E));
 502:   } else if (BPFOperand::isValidIdAtStart(Name))
 503:     Operands.push_back(BPFOperand::createToken(Name, NameLoc));
 504:   else
 505:     return Error(NameLoc, "invalid register/token name");
 506: 
 507:   while (!getLexer().is(AsmToken::EndOfStatement)) {
 508:     // Attempt to parse token as operator
 509:     if (parseOperandAsOperator(Operands).isSuccess())
 510:       continue;
 511: 
 512:     // Attempt to parse token as register
 513:     if (parseRegister(Operands).isSuccess())
 514:       continue;
 515: 
 516:     if (getLexer().is(AsmToken::Comma)) {
 517:       getLexer().Lex();
 518:       continue;
 519:     }
 520: 
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 521-540

```cpp
 521:     // Attempt to parse token as an immediate
 522:     if (!parseImmediate(Operands).isSuccess()) {
 523:       SMLoc Loc = getLexer().getLoc();
 524:       return Error(Loc, "unexpected token");
 525:     }
 526:   }
 527: 
 528:   if (getLexer().isNot(AsmToken::EndOfStatement)) {
 529:     SMLoc Loc = getLexer().getLoc();
 530: 
 531:     getParser().eatToEndOfStatement();
 532: 
 533:     return Error(Loc, "unexpected token");
 534:   }
 535: 
 536:   // Consume the EndOfStatement.
 537:   getParser().Lex();
 538:   return false;
 539: }
 540: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 541-545

```cpp
 541: extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void LLVMInitializeBPFAsmParser() {
 542:   RegisterMCAsmParser<BPFAsmParser> X(getTheBPFTarget());
 543:   RegisterMCAsmParser<BPFAsmParser> Y(getTheBPFleTarget());
 544:   RegisterMCAsmParser<BPFAsmParser> Z(getTheBPFbeTarget());
 545: }
```

- EN: In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

## Key Concepts / 关键概念

- Assembly syntax decoding / 汇编语法解码
- Operand construction / 操作数构造
- MC instruction representation / MC 指令表示
- MC streaming and emission / MC 流式输出
- CPU feature modelling / CPU 特性建模
- Assembly parsing / 汇编解析

## Dependencies / 依赖关系

- Direct includes / 直接包含: `MCTargetDesc/BPFMCAsmInfo.h`, `MCTargetDesc/BPFMCTargetDesc.h`, `TargetInfo/BPFTargetInfo.h`, `llvm/ADT/StringSwitch.h`, `llvm/MC/MCContext.h`, `llvm/MC/MCExpr.h`, `llvm/MC/MCInst.h`, `llvm/MC/MCInstrInfo.h`
- LLVM subsystems / LLVM 子系统: LLVM MC
- Generated or companion files / 生成或配套文件: `BPFGenAsmMatcher.inc`
