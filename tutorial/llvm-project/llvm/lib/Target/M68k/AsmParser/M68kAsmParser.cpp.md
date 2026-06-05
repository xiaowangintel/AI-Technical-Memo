# M68kAsmParser.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/M68k/AsmParser/M68kAsmParser.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file parses target assembly syntax into MC/LLVM instructions for the M68k backend.
- **用途 (CN)**: 该文件用于 M68k 后端，负责将目标汇编语法解析为 MC/LLVM 指令。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
```cpp
   1: //===-- M68kAsmParser.cpp - Parse M68k assembly to MCInst instructions ----===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: #include "M68kInstrInfo.h"
  10: #include "M68kRegisterInfo.h"
  11: #include "MCTargetDesc/M68kMCAsmInfo.h"
  12: #include "TargetInfo/M68kTargetInfo.h"
  13: 
  14: #include "llvm/MC/MCContext.h"
  15: #include "llvm/MC/MCParser/AsmLexer.h"
  16: #include "llvm/MC/MCParser/MCParsedAsmOperand.h"
  17: #include "llvm/MC/MCParser/MCTargetAsmParser.h"
  18: #include "llvm/MC/MCStreamer.h"
  19: #include "llvm/MC/TargetRegistry.h"
  20: 
  21: #include <sstream>
  22: 
  23: #define DEBUG_TYPE "m68k-asm-parser"
  24: 
  25: using namespace llvm;
  26: 
  27: static cl::opt<bool> RegisterPrefixOptional(
  28:     "m68k-register-prefix-optional", cl::Hidden,
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `M68kInstrInfo.h`, `M68kRegisterInfo.h`, `M68kMCAsmInfo.h`, `M68kTargetInfo.h`, `MCContext.h`, `AsmLexer.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `M68kInstrInfo.h`, `M68kRegisterInfo.h`, `M68kMCAsmInfo.h`, `M68kTargetInfo.h`, `MCContext.h`, `AsmLexer.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 29-56 / 第 29-56 行
```cpp
  29:     cl::desc("Enable specifying registers without the % prefix"),
  30:     cl::init(false));
  31: 
  32: namespace {
  33: /// Parses M68k assembly from a stream.
  34: class M68kAsmParser : public MCTargetAsmParser {
  35:   MCAsmParser &Parser;
  36:   const MCRegisterInfo *MRI;
  37: 
  38: #define GET_ASSEMBLER_HEADER
  39: #include "M68kGenAsmMatcher.inc"
  40: 
  41:   // Helpers for Match&Emit.
  42:   bool invalidOperand(SMLoc Loc, const OperandVector &Operands,
  43:                       const uint64_t &ErrorInfo);
  44:   bool missingFeature(SMLoc Loc, const uint64_t &ErrorInfo);
  45:   bool emit(MCInst &Inst, SMLoc Loc, MCStreamer &Out) const;
  46:   bool parseRegisterName(MCRegister &RegNo, SMLoc Loc, StringRef RegisterName);
  47:   ParseStatus parseRegister(MCRegister &RegNo);
  48: 
  49:   // Parser functions.
  50:   void eatComma();
  51: 
  52:   bool isExpr();
  53:   ParseStatus parseImm(OperandVector &Operands);
  54:   ParseStatus parseMemOp(OperandVector &Operands);
  55:   ParseStatus parseRegOrMoveMask(OperandVector &Operands);
  56: 
```
- **EN**: It imports dependencies such as `M68kGenAsmMatcher.inc` that expose the LLVM and target interfaces used in later logic. This block declares or refines TableGen records such as `M68kAsmParser`. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 它引入了 `M68kGenAsmMatcher.inc` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 该代码块声明或细化了 `M68kAsmParser` 等 TableGen 记录。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 57-84 / 第 57-84 行
```cpp
  57: public:
  58:   M68kAsmParser(const MCSubtargetInfo &STI, MCAsmParser &Parser,
  59:                 const MCInstrInfo &MII)
  60:       : MCTargetAsmParser(STI, MII), Parser(Parser) {
  61:     MCAsmParserExtension::Initialize(Parser);
  62:     MRI = getContext().getRegisterInfo();
  63: 
  64:     setAvailableFeatures(ComputeAvailableFeatures(STI.getFeatureBits()));
  65:   }
  66: 
  67:   unsigned validateTargetOperandClass(MCParsedAsmOperand &Op,
  68:                                       unsigned Kind) override;
  69:   bool parseRegister(MCRegister &Reg, SMLoc &StartLoc, SMLoc &EndLoc) override;
  70:   ParseStatus tryParseRegister(MCRegister &Reg, SMLoc &StartLoc,
  71:                                SMLoc &EndLoc) override;
  72:   bool parseInstruction(ParseInstructionInfo &Info, StringRef Name,
  73:                         SMLoc NameLoc, OperandVector &Operands) override;
  74:   bool matchAndEmitInstruction(SMLoc IDLoc, unsigned &Opcode,
  75:                                OperandVector &Operands, MCStreamer &Out,
  76:                                uint64_t &ErrorInfo,
  77:                                bool MatchingInlineAsm) override;
  78: };
  79: 
  80: struct M68kMemOp {
  81:   enum class Kind {
  82:     Addr,
  83:     RegMask,
  84:     Reg,
```
- **EN**: It introduces interface types such as `M68kMemOp`, shaping how other backend components interact with this file. The range implements or declares functions including `M68kAsmParser`.
- **CN**: 它引入了 `M68kMemOp` 等接口类型，定义了其他后端组件与本文件交互的方式。 这一段实现或声明了 `M68kAsmParser` 等函数。

### Lines 85-112 / 第 85-112 行
```cpp
  85:     RegIndirect,
  86:     RegPostIncrement,
  87:     RegPreDecrement,
  88:     RegIndirectDisplacement,
  89:     RegIndirectDisplacementIndex,
  90:   };
  91: 
  92:   // These variables are used for the following forms:
  93:   // Addr: (OuterDisp)
  94:   // RegMask: RegMask (as register mask)
  95:   // Reg: %OuterReg
  96:   // RegIndirect: (%OuterReg)
  97:   // RegPostIncrement: (%OuterReg)+
  98:   // RegPreDecrement: -(%OuterReg)
  99:   // RegIndirectDisplacement: OuterDisp(%OuterReg)
 100:   // RegIndirectDisplacementIndex:
 101:   //   OuterDisp(%OuterReg, %InnerReg.Size * Scale, InnerDisp)
 102: 
 103:   Kind Op;
 104:   MCRegister OuterReg;
 105:   MCRegister InnerReg;
 106:   const MCExpr *OuterDisp;
 107:   const MCExpr *InnerDisp;
 108:   uint8_t Size : 4;
 109:   uint8_t Scale : 4;
 110:   const MCExpr *Expr;
 111:   uint16_t RegMask;
 112: 
```
- **EN**: This span continues the file's main responsibility: this file parses target assembly syntax into MC/LLVM instructions for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 113-140 / 第 113-140 行
```cpp
 113:   M68kMemOp() {}
 114:   M68kMemOp(Kind Op) : Op(Op) {}
 115: 
 116:   void print(raw_ostream &OS) const;
 117: };
 118: 
 119: /// An parsed M68k assembly operand.
 120: class M68kOperand : public MCParsedAsmOperand {
 121:   typedef MCParsedAsmOperand Base;
 122: 
 123:   enum class KindTy {
 124:     Invalid,
 125:     Token,
 126:     Imm,
 127:     MemOp,
 128:   };
 129: 
 130:   KindTy Kind;
 131:   SMLoc Start, End;
 132:   union {
 133:     StringRef Token;
 134:     const MCExpr *Expr;
 135:     M68kMemOp MemOp;
 136:   };
 137: 
 138:   template <unsigned N> bool isAddrN() const;
 139: 
 140: public:
```
- **EN**: This block declares or refines TableGen records such as `M68kOperand`. The range implements or declares functions including `M68kMemOp`.
- **CN**: 该代码块声明或细化了 `M68kOperand` 等 TableGen 记录。 这一段实现或声明了 `M68kMemOp` 等函数。

### Lines 141-168 / 第 141-168 行
```cpp
 141:   M68kOperand(KindTy Kind, SMLoc Start, SMLoc End)
 142:       : Base(), Kind(Kind), Start(Start), End(End) {}
 143: 
 144:   SMLoc getStartLoc() const override { return Start; }
 145:   SMLoc getEndLoc() const override { return End; }
 146: 
 147:   void print(raw_ostream &OS, const MCAsmInfo &MAI) const override;
 148: 
 149:   bool isMem() const override { return false; }
 150:   bool isMemOp() const { return Kind == KindTy::MemOp; }
 151: 
 152:   static void addExpr(MCInst &Inst, const MCExpr *Expr);
 153: 
 154:   // Reg
 155:   bool isReg() const override;
 156:   bool isAReg() const;
 157:   bool isDReg() const;
 158:   bool isFPDReg() const;
 159:   bool isFPCReg() const;
 160:   MCRegister getReg() const override;
 161:   void addRegOperands(MCInst &Inst, unsigned N) const;
 162: 
 163:   static std::unique_ptr<M68kOperand> createMemOp(M68kMemOp MemOp, SMLoc Start,
 164:                                                   SMLoc End);
 165: 
 166:   // Token
 167:   bool isToken() const override;
 168:   StringRef getToken() const;
```
- **EN**: The range implements or declares functions including `M68kOperand`, `isMemOp`.
- **CN**: 这一段实现或声明了 `M68kOperand`, `isMemOp` 等函数。

### Lines 169-196 / 第 169-196 行
```cpp
 169:   static std::unique_ptr<M68kOperand> createToken(StringRef Token, SMLoc Start,
 170:                                                   SMLoc End);
 171: 
 172:   // Imm
 173:   bool isImm() const override;
 174:   void addImmOperands(MCInst &Inst, unsigned N) const;
 175: 
 176:   static std::unique_ptr<M68kOperand> createImm(const MCExpr *Expr, SMLoc Start,
 177:                                                 SMLoc End);
 178: 
 179:   // Imm for TRAP instruction
 180:   bool isTrapImm() const;
 181:   // Imm for BKPT instruction
 182:   bool isBkptImm() const;
 183: 
 184:   // MoveMask
 185:   bool isMoveMask() const;
 186:   void addMoveMaskOperands(MCInst &Inst, unsigned N) const;
 187: 
 188:   // Addr
 189:   bool isAddr() const;
 190:   bool isAddr8() const { return isAddrN<8>(); }
 191:   bool isAddr16() const { return isAddrN<16>(); }
 192:   bool isAddr32() const { return isAddrN<32>(); }
 193:   void addAddrOperands(MCInst &Inst, unsigned N) const;
 194: 
 195:   // ARI
 196:   bool isARI() const;
```
- **EN**: The range implements or declares functions including `isAddr8`, `isAddr16`, `isAddr32`.
- **CN**: 这一段实现或声明了 `isAddr8`, `isAddr16`, `isAddr32` 等函数。

### Lines 197-224 / 第 197-224 行
```cpp
 197:   void addARIOperands(MCInst &Inst, unsigned N) const;
 198: 
 199:   // ARID
 200:   bool isARID() const;
 201:   void addARIDOperands(MCInst &Inst, unsigned N) const;
 202: 
 203:   // ARII
 204:   bool isARII() const;
 205:   void addARIIOperands(MCInst &Inst, unsigned N) const;
 206: 
 207:   // ARIPD
 208:   bool isARIPD() const;
 209:   void addARIPDOperands(MCInst &Inst, unsigned N) const;
 210: 
 211:   // ARIPI
 212:   bool isARIPI() const;
 213:   void addARIPIOperands(MCInst &Inst, unsigned N) const;
 214: 
 215:   // PCD
 216:   bool isPCD() const;
 217:   void addPCDOperands(MCInst &Inst, unsigned N) const;
 218: 
 219:   // PCI
 220:   bool isPCI() const;
 221:   void addPCIOperands(MCInst &Inst, unsigned N) const;
 222: };
 223: 
 224: } // end anonymous namespace.
```
- **EN**: This span continues the file's main responsibility: this file parses target assembly syntax into MC/LLVM instructions for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 225-252 / 第 225-252 行
```cpp
 225: 
 226: extern "C" LLVM_EXTERNAL_VISIBILITY void LLVMInitializeM68kAsmParser() {
 227:   RegisterMCAsmParser<M68kAsmParser> X(getTheM68kTarget());
 228: }
 229: 
 230: #define GET_REGISTER_MATCHER
 231: #define GET_MATCHER_IMPLEMENTATION
 232: #include "M68kGenAsmMatcher.inc"
 233: 
 234: static inline unsigned getRegisterByIndex(unsigned RegisterIndex) {
 235:   static unsigned RegistersByIndex[] = {
 236:       M68k::D0,  M68k::D1,  M68k::D2,  M68k::D3,  M68k::D4,  M68k::D5,
 237:       M68k::D6,  M68k::D7,  M68k::A0,  M68k::A1,  M68k::A2,  M68k::A3,
 238:       M68k::A4,  M68k::A5,  M68k::A6,  M68k::SP,  M68k::FP0, M68k::FP1,
 239:       M68k::FP2, M68k::FP3, M68k::FP4, M68k::FP5, M68k::FP6, M68k::FP7};
 240:   assert(RegisterIndex <=
 241:          sizeof(RegistersByIndex) / sizeof(RegistersByIndex[0]));
 242:   return RegistersByIndex[RegisterIndex];
 243: }
 244: 
 245: static inline unsigned getRegisterIndex(unsigned Register) {
 246:   if (Register >= M68k::D0 && Register <= M68k::D7)
 247:     return Register - M68k::D0;
 248:   if (Register >= M68k::A0 && Register <= M68k::A6)
 249:     return Register - M68k::A0 + 8;
 250:   if (Register >= M68k::FP0 && Register <= M68k::FP7)
 251:     return Register - M68k::FP0 + 16;
 252: 
```
- **EN**: It imports dependencies such as `M68kGenAsmMatcher.inc` that expose the LLVM and target interfaces used in later logic. The range implements or declares functions including `getRegisterByIndex`, `getRegisterIndex`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 它引入了 `M68kGenAsmMatcher.inc` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段实现或声明了 `getRegisterByIndex`, `getRegisterIndex` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 253-280 / 第 253-280 行
```cpp
 253:   switch (Register) {
 254:   case M68k::SP:
 255:     // SP is sadly not contiguous with the rest of the An registers
 256:     return 15;
 257: 
 258:   // We don't care about the indices of these registers.
 259:   case M68k::PC:
 260:   case M68k::CCR:
 261:   case M68k::SR:
 262:   case M68k::FPC:
 263:   case M68k::FPS:
 264:   case M68k::FPIAR:
 265:     return UINT_MAX;
 266: 
 267:   default:
 268:     llvm_unreachable("unexpected register number");
 269:   }
 270: }
 271: 
 272: void M68kMemOp::print(raw_ostream &OS) const {
 273:   switch (Op) {
 274:   case Kind::Addr:
 275:     OS << OuterDisp;
 276:     break;
 277:   case Kind::RegMask:
 278:     OS << "RegMask(" << format("%04x", RegMask) << ")";
 279:     break;
 280:   case Kind::Reg:
```
- **EN**: The range implements or declares functions including `M68kMemOp::print`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `M68kMemOp::print` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 281-308 / 第 281-308 行
```cpp
 281:     OS << '%' << OuterReg;
 282:     break;
 283:   case Kind::RegIndirect:
 284:     OS << "(%" << OuterReg << ')';
 285:     break;
 286:   case Kind::RegPostIncrement:
 287:     OS << "(%" << OuterReg << ")+";
 288:     break;
 289:   case Kind::RegPreDecrement:
 290:     OS << "-(%" << OuterReg << ")";
 291:     break;
 292:   case Kind::RegIndirectDisplacement:
 293:     OS << OuterDisp << "(%" << OuterReg << ")";
 294:     break;
 295:   case Kind::RegIndirectDisplacementIndex:
 296:     OS << OuterDisp << "(%" << OuterReg << ", " << InnerReg << "." << Size
 297:        << ", " << InnerDisp << ")";
 298:     break;
 299:   }
 300: }
 301: 
 302: void M68kOperand::addExpr(MCInst &Inst, const MCExpr *Expr) {
 303:   if (auto Const = dyn_cast<MCConstantExpr>(Expr)) {
 304:     Inst.addOperand(MCOperand::createImm(Const->getValue()));
 305:     return;
 306:   }
 307: 
 308:   Inst.addOperand(MCOperand::createExpr(Expr));
```
- **EN**: The range implements or declares functions including `M68kOperand::addExpr`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kOperand::addExpr` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 309-336 / 第 309-336 行
```cpp
 309: }
 310: 
 311: // Reg
 312: bool M68kOperand::isReg() const {
 313:   return Kind == KindTy::MemOp && MemOp.Op == M68kMemOp::Kind::Reg;
 314: }
 315: 
 316: MCRegister M68kOperand::getReg() const {
 317:   assert(isReg());
 318:   return MemOp.OuterReg;
 319: }
 320: 
 321: void M68kOperand::addRegOperands(MCInst &Inst, unsigned N) const {
 322:   assert(isReg() && "wrong operand kind");
 323:   assert((N == 1) && "can only handle one register operand");
 324: 
 325:   Inst.addOperand(MCOperand::createReg(getReg()));
 326: }
 327: 
 328: std::unique_ptr<M68kOperand> M68kOperand::createMemOp(M68kMemOp MemOp,
 329:                                                       SMLoc Start, SMLoc End) {
 330:   auto Op = std::make_unique<M68kOperand>(KindTy::MemOp, Start, End);
 331:   Op->MemOp = MemOp;
 332:   return Op;
 333: }
 334: 
 335: // Token
 336: bool M68kOperand::isToken() const { return Kind == KindTy::Token; }
```
- **EN**: The range implements or declares functions including `M68kOperand::isReg`, `M68kOperand::getReg`, `M68kOperand::addRegOperands`, `M68kOperand::createMemOp`, `M68kOperand::isToken`. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `M68kOperand::isReg`, `M68kOperand::getReg`, `M68kOperand::addRegOperands`, `M68kOperand::createMemOp`, `M68kOperand::isToken` 等函数。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 337-364 / 第 337-364 行
```cpp
 337: StringRef M68kOperand::getToken() const {
 338:   assert(isToken());
 339:   return Token;
 340: }
 341: 
 342: std::unique_ptr<M68kOperand> M68kOperand::createToken(StringRef Token,
 343:                                                       SMLoc Start, SMLoc End) {
 344:   auto Op = std::make_unique<M68kOperand>(KindTy::Token, Start, End);
 345:   Op->Token = Token;
 346:   return Op;
 347: }
 348: 
 349: // Imm
 350: bool M68kOperand::isImm() const { return Kind == KindTy::Imm; }
 351: void M68kOperand::addImmOperands(MCInst &Inst, unsigned N) const {
 352:   assert(isImm() && "wrong operand kind");
 353:   assert((N == 1) && "can only handle one register operand");
 354: 
 355:   M68kOperand::addExpr(Inst, Expr);
 356: }
 357: 
 358: std::unique_ptr<M68kOperand> M68kOperand::createImm(const MCExpr *Expr,
 359:                                                     SMLoc Start, SMLoc End) {
 360:   auto Op = std::make_unique<M68kOperand>(KindTy::Imm, Start, End);
 361:   Op->Expr = Expr;
 362:   return Op;
 363: }
 364: 
```
- **EN**: The range implements or declares functions including `M68kOperand::getToken`, `M68kOperand::createToken`, `M68kOperand::isImm`, `M68kOperand::addImmOperands`, `M68kOperand::createImm`. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `M68kOperand::getToken`, `M68kOperand::createToken`, `M68kOperand::isImm`, `M68kOperand::addImmOperands`, `M68kOperand::createImm` 等函数。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 365-392 / 第 365-392 行
```cpp
 365: bool M68kOperand::isTrapImm() const {
 366:   int64_t Value;
 367:   if (!isImm() || !Expr->evaluateAsAbsolute(Value))
 368:     return false;
 369: 
 370:   return isUInt<4>(Value);
 371: }
 372: 
 373: bool M68kOperand::isBkptImm() const {
 374:   int64_t Value;
 375:   if (!isImm() || !Expr->evaluateAsAbsolute(Value))
 376:     return false;
 377: 
 378:   return isUInt<3>(Value);
 379: }
 380: 
 381: // MoveMask
 382: bool M68kOperand::isMoveMask() const {
 383:   if (!isMemOp())
 384:     return false;
 385: 
 386:   if (MemOp.Op == M68kMemOp::Kind::RegMask)
 387:     return true;
 388: 
 389:   if (MemOp.Op != M68kMemOp::Kind::Reg)
 390:     return false;
 391: 
 392:   // Only regular address / data registers are allowed to be used
```
- **EN**: The range implements or declares functions including `M68kOperand::isTrapImm`, `M68kOperand::isBkptImm`, `M68kOperand::isMoveMask`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kOperand::isTrapImm`, `M68kOperand::isBkptImm`, `M68kOperand::isMoveMask` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 393-420 / 第 393-420 行
```cpp
 393:   // in register masks.
 394:   return getRegisterIndex(MemOp.OuterReg) < 16;
 395: }
 396: 
 397: void M68kOperand::addMoveMaskOperands(MCInst &Inst, unsigned N) const {
 398:   assert(isMoveMask() && "wrong operand kind");
 399:   assert((N == 1) && "can only handle one immediate operand");
 400: 
 401:   uint16_t MoveMask = MemOp.RegMask;
 402:   if (MemOp.Op == M68kMemOp::Kind::Reg)
 403:     MoveMask = 1 << getRegisterIndex(MemOp.OuterReg);
 404: 
 405:   Inst.addOperand(MCOperand::createImm(MoveMask));
 406: }
 407: 
 408: // Addr
 409: bool M68kOperand::isAddr() const {
 410:   return isMemOp() && MemOp.Op == M68kMemOp::Kind::Addr;
 411: }
 412: // TODO: Maybe we can also store the size of OuterDisp
 413: // in Size?
 414: template <unsigned N> bool M68kOperand::isAddrN() const {
 415:   if (isAddr()) {
 416:     int64_t Res;
 417:     if (MemOp.OuterDisp->evaluateAsAbsolute(Res))
 418:       return isInt<N>(Res);
 419:     return true;
 420:   }
```
- **EN**: The range implements or declares functions including `M68kOperand::addMoveMaskOperands`, `M68kOperand::isAddr`, `M68kOperand::isAddrN`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `M68kOperand::addMoveMaskOperands`, `M68kOperand::isAddr`, `M68kOperand::isAddrN` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 421-448 / 第 421-448 行
```cpp
 421:   return false;
 422: }
 423: void M68kOperand::addAddrOperands(MCInst &Inst, unsigned N) const {
 424:   M68kOperand::addExpr(Inst, MemOp.OuterDisp);
 425: }
 426: 
 427: // ARI
 428: bool M68kOperand::isARI() const {
 429:   return isMemOp() && MemOp.Op == M68kMemOp::Kind::RegIndirect &&
 430:          M68k::AR32RegClass.contains(MemOp.OuterReg);
 431: }
 432: void M68kOperand::addARIOperands(MCInst &Inst, unsigned N) const {
 433:   Inst.addOperand(MCOperand::createReg(MemOp.OuterReg));
 434: }
 435: 
 436: // ARID
 437: bool M68kOperand::isARID() const {
 438:   return isMemOp() && MemOp.Op == M68kMemOp::Kind::RegIndirectDisplacement &&
 439:          M68k::AR32RegClass.contains(MemOp.OuterReg);
 440: }
 441: void M68kOperand::addARIDOperands(MCInst &Inst, unsigned N) const {
 442:   M68kOperand::addExpr(Inst, MemOp.OuterDisp);
 443:   Inst.addOperand(MCOperand::createReg(MemOp.OuterReg));
 444: }
 445: 
 446: // ARII
 447: bool M68kOperand::isARII() const {
 448:   return isMemOp() &&
```
- **EN**: The range implements or declares functions including `M68kOperand::addAddrOperands`, `M68kOperand::isARI`, `M68kOperand::addARIOperands`, `M68kOperand::isARID`, `M68kOperand::addARIDOperands`.
- **CN**: 这一段实现或声明了 `M68kOperand::addAddrOperands`, `M68kOperand::isARI`, `M68kOperand::addARIOperands`, `M68kOperand::isARID`, `M68kOperand::addARIDOperands` 等函数。

### Lines 449-476 / 第 449-476 行
```cpp
 449:          MemOp.Op == M68kMemOp::Kind::RegIndirectDisplacementIndex &&
 450:          M68k::AR32RegClass.contains(MemOp.OuterReg);
 451: }
 452: void M68kOperand::addARIIOperands(MCInst &Inst, unsigned N) const {
 453:   M68kOperand::addExpr(Inst, MemOp.OuterDisp);
 454:   Inst.addOperand(MCOperand::createReg(MemOp.OuterReg));
 455:   Inst.addOperand(MCOperand::createReg(MemOp.InnerReg));
 456: }
 457: 
 458: // ARIPD
 459: bool M68kOperand::isARIPD() const {
 460:   return isMemOp() && MemOp.Op == M68kMemOp::Kind::RegPreDecrement &&
 461:          M68k::AR32RegClass.contains(MemOp.OuterReg);
 462: }
 463: void M68kOperand::addARIPDOperands(MCInst &Inst, unsigned N) const {
 464:   Inst.addOperand(MCOperand::createReg(MemOp.OuterReg));
 465: }
 466: 
 467: // ARIPI
 468: bool M68kOperand::isARIPI() const {
 469:   return isMemOp() && MemOp.Op == M68kMemOp::Kind::RegPostIncrement &&
 470:          M68k::AR32RegClass.contains(MemOp.OuterReg);
 471: }
 472: void M68kOperand::addARIPIOperands(MCInst &Inst, unsigned N) const {
 473:   Inst.addOperand(MCOperand::createReg(MemOp.OuterReg));
 474: }
 475: 
 476: // PCD
```
- **EN**: The range implements or declares functions including `M68kOperand::addARIIOperands`, `M68kOperand::isARIPD`, `M68kOperand::addARIPDOperands`, `M68kOperand::isARIPI`, `M68kOperand::addARIPIOperands`.
- **CN**: 这一段实现或声明了 `M68kOperand::addARIIOperands`, `M68kOperand::isARIPD`, `M68kOperand::addARIPDOperands`, `M68kOperand::isARIPI`, `M68kOperand::addARIPIOperands` 等函数。

### Lines 477-504 / 第 477-504 行
```cpp
 477: bool M68kOperand::isPCD() const {
 478:   return isMemOp() && MemOp.Op == M68kMemOp::Kind::RegIndirectDisplacement &&
 479:          MemOp.OuterReg == M68k::PC;
 480: }
 481: void M68kOperand::addPCDOperands(MCInst &Inst, unsigned N) const {
 482:   M68kOperand::addExpr(Inst, MemOp.OuterDisp);
 483: }
 484: 
 485: // PCI
 486: bool M68kOperand::isPCI() const {
 487:   return isMemOp() &&
 488:          MemOp.Op == M68kMemOp::Kind::RegIndirectDisplacementIndex &&
 489:          MemOp.OuterReg == M68k::PC;
 490: }
 491: void M68kOperand::addPCIOperands(MCInst &Inst, unsigned N) const {
 492:   M68kOperand::addExpr(Inst, MemOp.OuterDisp);
 493:   Inst.addOperand(MCOperand::createReg(MemOp.InnerReg));
 494: }
 495: 
 496: static inline bool checkRegisterClass(unsigned RegNo, bool Data, bool Address,
 497:                                       bool SP, bool FPDR = false,
 498:                                       bool FPCR = false) {
 499:   switch (RegNo) {
 500:   case M68k::A0:
 501:   case M68k::A1:
 502:   case M68k::A2:
 503:   case M68k::A3:
 504:   case M68k::A4:
```
- **EN**: The range implements or declares functions including `M68kOperand::isPCD`, `M68kOperand::addPCDOperands`, `M68kOperand::isPCI`, `M68kOperand::addPCIOperands`, `checkRegisterClass`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这一段实现或声明了 `M68kOperand::isPCD`, `M68kOperand::addPCDOperands`, `M68kOperand::isPCI`, `M68kOperand::addPCIOperands`, `checkRegisterClass` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 505-532 / 第 505-532 行
```cpp
 505:   case M68k::A5:
 506:   case M68k::A6:
 507:     return Address;
 508: 
 509:   case M68k::SP:
 510:     return SP;
 511: 
 512:   case M68k::D0:
 513:   case M68k::D1:
 514:   case M68k::D2:
 515:   case M68k::D3:
 516:   case M68k::D4:
 517:   case M68k::D5:
 518:   case M68k::D6:
 519:   case M68k::D7:
 520:     return Data;
 521: 
 522:   case M68k::SR:
 523:   case M68k::CCR:
 524:     return false;
 525: 
 526:   case M68k::FP0:
 527:   case M68k::FP1:
 528:   case M68k::FP2:
 529:   case M68k::FP3:
 530:   case M68k::FP4:
 531:   case M68k::FP5:
 532:   case M68k::FP6:
```
- **EN**: This span continues the file's main responsibility: this file parses target assembly syntax into MC/LLVM instructions for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 533-560 / 第 533-560 行
```cpp
 533:   case M68k::FP7:
 534:     return FPDR;
 535: 
 536:   case M68k::FPC:
 537:   case M68k::FPS:
 538:   case M68k::FPIAR:
 539:     return FPCR;
 540: 
 541:   default:
 542:     llvm_unreachable("unexpected register type");
 543:     return false;
 544:   }
 545: }
 546: 
 547: bool M68kOperand::isAReg() const {
 548:   return isReg() && checkRegisterClass(getReg(),
 549:                                        /*Data=*/false,
 550:                                        /*Address=*/true, /*SP=*/true);
 551: }
 552: 
 553: bool M68kOperand::isDReg() const {
 554:   return isReg() && checkRegisterClass(getReg(),
 555:                                        /*Data=*/true,
 556:                                        /*Address=*/false, /*SP=*/false);
 557: }
 558: 
 559: bool M68kOperand::isFPDReg() const {
 560:   return isReg() && checkRegisterClass(getReg(),
```
- **EN**: The range implements or declares functions including `M68kOperand::isAReg`, `M68kOperand::isDReg`, `M68kOperand::isFPDReg`. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `M68kOperand::isAReg`, `M68kOperand::isDReg`, `M68kOperand::isFPDReg` 等函数。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 561-588 / 第 561-588 行
```cpp
 561:                                        /*Data=*/false,
 562:                                        /*Address=*/false, /*SP=*/false,
 563:                                        /*FPDR=*/true);
 564: }
 565: 
 566: bool M68kOperand::isFPCReg() const {
 567:   return isReg() && checkRegisterClass(getReg(),
 568:                                        /*Data=*/false,
 569:                                        /*Address=*/false, /*SP=*/false,
 570:                                        /*FPDR=*/false, /*FPCR=*/true);
 571: }
 572: 
 573: unsigned M68kAsmParser::validateTargetOperandClass(MCParsedAsmOperand &Op,
 574:                                                    unsigned Kind) {
 575:   M68kOperand &Operand = (M68kOperand &)Op;
 576: 
 577:   switch (Kind) {
 578:   case MCK_XR16:
 579:   case MCK_SPILL:
 580:     if (Operand.isReg() &&
 581:         checkRegisterClass(Operand.getReg(), true, true, true)) {
 582:       return Match_Success;
 583:     }
 584:     break;
 585: 
 586:   case MCK_AR16:
 587:   case MCK_AR32:
 588:     if (Operand.isReg() &&
```
- **EN**: The range implements or declares functions including `M68kOperand::isFPCReg`, `M68kAsmParser::validateTargetOperandClass`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kOperand::isFPCReg`, `M68kAsmParser::validateTargetOperandClass` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 589-616 / 第 589-616 行
```cpp
 589:         checkRegisterClass(Operand.getReg(), false, true, true)) {
 590:       return Match_Success;
 591:     }
 592:     break;
 593: 
 594:   case MCK_AR32_NOSP:
 595:     if (Operand.isReg() &&
 596:         checkRegisterClass(Operand.getReg(), false, true, false)) {
 597:       return Match_Success;
 598:     }
 599:     break;
 600: 
 601:   case MCK_DR8:
 602:   case MCK_DR16:
 603:   case MCK_DR32:
 604:     if (Operand.isReg() &&
 605:         checkRegisterClass(Operand.getReg(), true, false, false)) {
 606:       return Match_Success;
 607:     }
 608:     break;
 609: 
 610:   case MCK_AR16_TC:
 611:     if (Operand.isReg() &&
 612:         ((Operand.getReg() == M68k::A0) || (Operand.getReg() == M68k::A1))) {
 613:       return Match_Success;
 614:     }
 615:     break;
 616: 
```
- **EN**: The range implements or declares functions including `checkRegisterClass`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `checkRegisterClass` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 617-644 / 第 617-644 行
```cpp
 617:   case MCK_DR16_TC:
 618:     if (Operand.isReg() &&
 619:         ((Operand.getReg() == M68k::D0) || (Operand.getReg() == M68k::D1))) {
 620:       return Match_Success;
 621:     }
 622:     break;
 623: 
 624:   case MCK_XR16_TC:
 625:     if (Operand.isReg() &&
 626:         ((Operand.getReg() == M68k::D0) || (Operand.getReg() == M68k::D1) ||
 627:          (Operand.getReg() == M68k::A0) || (Operand.getReg() == M68k::A1))) {
 628:       return Match_Success;
 629:     }
 630:     break;
 631:   }
 632: 
 633:   return Match_InvalidOperand;
 634: }
 635: 
 636: bool M68kAsmParser::parseRegisterName(MCRegister &RegNo, SMLoc Loc,
 637:                                       StringRef RegisterName) {
 638:   auto RegisterNameLower = RegisterName.lower();
 639: 
 640:   // CCR and SR register
 641:   if (RegisterNameLower == "ccr") {
 642:     RegNo = M68k::CCR;
 643:     return true;
 644:   } else if (RegisterNameLower == "sr") {
```
- **EN**: The range implements or declares functions including `M68kAsmParser::parseRegisterName`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kAsmParser::parseRegisterName` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 645-672 / 第 645-672 行
```cpp
 645:     RegNo = M68k::SR;
 646:     return true;
 647:   }
 648: 
 649:   // Parse simple general-purpose registers.
 650:   if (RegisterNameLower.size() == 2) {
 651: 
 652:     switch (RegisterNameLower[0]) {
 653:     case 'd':
 654:     case 'a': {
 655:       if (isdigit(RegisterNameLower[1])) {
 656:         unsigned IndexOffset = (RegisterNameLower[0] == 'a') ? 8 : 0;
 657:         unsigned RegIndex = (unsigned)(RegisterNameLower[1] - '0');
 658:         if (RegIndex < 8) {
 659:           RegNo = getRegisterByIndex(IndexOffset + RegIndex);
 660:           return true;
 661:         }
 662:       }
 663:       break;
 664:     }
 665: 
 666:     case 's':
 667:       if (RegisterNameLower[1] == 'p') {
 668:         RegNo = M68k::SP;
 669:         return true;
 670:       } else if (RegisterNameLower[1] == 'r') {
 671:         RegNo = M68k::SR;
 672:         return true;
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 673-700 / 第 673-700 行
```cpp
 673:       }
 674:       break;
 675: 
 676:     case 'p':
 677:       if (RegisterNameLower[1] == 'c') {
 678:         RegNo = M68k::PC;
 679:         return true;
 680:       }
 681:       break;
 682:     }
 683:   } else if (StringRef(RegisterNameLower).starts_with("fp") &&
 684:              RegisterNameLower.size() > 2) {
 685:     auto RegIndex = unsigned(RegisterNameLower[2] - '0');
 686:     if (RegIndex < 8 && RegisterNameLower.size() == 3) {
 687:       // Floating point data register.
 688:       RegNo = getRegisterByIndex(16 + RegIndex);
 689:       return true;
 690:     } else {
 691:       // Floating point control register.
 692:       RegNo = StringSwitch<unsigned>(RegisterNameLower)
 693:                   .Cases({"fpc", "fpcr"}, M68k::FPC)
 694:                   .Cases({"fps", "fpsr"}, M68k::FPS)
 695:                   .Cases({"fpi", "fpiar"}, M68k::FPIAR)
 696:                   .Default(M68k::NoRegister);
 697:       assert(RegNo != M68k::NoRegister &&
 698:              "Unrecognized FP control register name");
 699:       return true;
 700:     }
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 701-728 / 第 701-728 行
```cpp
 701:   }
 702: 
 703:   return false;
 704: }
 705: 
 706: ParseStatus M68kAsmParser::parseRegister(MCRegister &RegNo) {
 707:   bool HasPercent = false;
 708:   AsmToken PercentToken;
 709: 
 710:   LLVM_DEBUG(dbgs() << "parseRegister "; getTok().dump(dbgs()); dbgs() << "\n");
 711: 
 712:   if (getTok().is(AsmToken::Percent)) {
 713:     HasPercent = true;
 714:     PercentToken = Lex();
 715:   } else if (!RegisterPrefixOptional.getValue()) {
 716:     return ParseStatus::NoMatch;
 717:   }
 718: 
 719:   if (!Parser.getTok().is(AsmToken::Identifier)) {
 720:     if (HasPercent) {
 721:       getLexer().UnLex(PercentToken);
 722:     }
 723:     return ParseStatus::NoMatch;
 724:   }
 725: 
 726:   auto RegisterName = Parser.getTok().getString();
 727:   if (!parseRegisterName(RegNo, Parser.getLexer().getLoc(), RegisterName)) {
 728:     if (HasPercent) {
```
- **EN**: The range implements or declares functions including `M68kAsmParser::parseRegister`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kAsmParser::parseRegister` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 729-756 / 第 729-756 行
```cpp
 729:       getLexer().UnLex(PercentToken);
 730:     }
 731:     return ParseStatus::NoMatch;
 732:   }
 733: 
 734:   Parser.Lex();
 735:   return ParseStatus::Success;
 736: }
 737: 
 738: bool M68kAsmParser::parseRegister(MCRegister &Reg, SMLoc &StartLoc,
 739:                                   SMLoc &EndLoc) {
 740:   ParseStatus Result = tryParseRegister(Reg, StartLoc, EndLoc);
 741:   if (!Result.isSuccess())
 742:     return Error(StartLoc, "expected register");
 743: 
 744:   return false;
 745: }
 746: 
 747: ParseStatus M68kAsmParser::tryParseRegister(MCRegister &Reg, SMLoc &StartLoc,
 748:                                             SMLoc &EndLoc) {
 749:   StartLoc = getLexer().getLoc();
 750:   ParseStatus Result = parseRegister(Reg);
 751:   EndLoc = getLexer().getLoc();
 752:   return Result;
 753: }
 754: 
 755: bool M68kAsmParser::isExpr() {
 756:   switch (Parser.getTok().getKind()) {
```
- **EN**: The range implements or declares functions including `M68kAsmParser::parseRegister`, `M68kAsmParser::tryParseRegister`, `M68kAsmParser::isExpr`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kAsmParser::parseRegister`, `M68kAsmParser::tryParseRegister`, `M68kAsmParser::isExpr` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 757-784 / 第 757-784 行
```cpp
 757:   case AsmToken::Identifier:
 758:   case AsmToken::Integer:
 759:     return true;
 760:   case AsmToken::Minus:
 761:     return getLexer().peekTok().getKind() == AsmToken::Integer;
 762: 
 763:   default:
 764:     return false;
 765:   }
 766: }
 767: 
 768: ParseStatus M68kAsmParser::parseImm(OperandVector &Operands) {
 769:   if (getLexer().isNot(AsmToken::Hash))
 770:     return ParseStatus::NoMatch;
 771:   SMLoc Start = getLexer().getLoc();
 772:   Parser.Lex();
 773: 
 774:   SMLoc End;
 775:   const MCExpr *Expr;
 776: 
 777:   if (getParser().parseExpression(Expr, End))
 778:     return ParseStatus::Failure;
 779: 
 780:   Operands.push_back(M68kOperand::createImm(Expr, Start, End));
 781:   return ParseStatus::Success;
 782: }
 783: 
 784: ParseStatus M68kAsmParser::parseMemOp(OperandVector &Operands) {
```
- **EN**: The range implements or declares functions including `M68kAsmParser::parseImm`, `M68kAsmParser::parseMemOp`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kAsmParser::parseImm`, `M68kAsmParser::parseMemOp` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 785-812 / 第 785-812 行
```cpp
 785:   SMLoc Start = getLexer().getLoc();
 786:   bool IsPD = false;
 787:   M68kMemOp MemOp;
 788: 
 789:   // Check for a plain register or register mask.
 790:   ParseStatus Result = parseRegOrMoveMask(Operands);
 791:   if (!Result.isNoMatch())
 792:     return Result;
 793: 
 794:   // Check for pre-decrement & outer displacement.
 795:   bool HasDisplacement = false;
 796:   if (getLexer().is(AsmToken::Minus)) {
 797:     IsPD = true;
 798:     Parser.Lex();
 799:   } else if (isExpr()) {
 800:     if (Parser.parseExpression(MemOp.OuterDisp))
 801:       return ParseStatus::Failure;
 802:     HasDisplacement = true;
 803:   }
 804: 
 805:   if (getLexer().isNot(AsmToken::LParen)) {
 806:     if (HasDisplacement) {
 807:       MemOp.Op = M68kMemOp::Kind::Addr;
 808:       Operands.push_back(
 809:           M68kOperand::createMemOp(MemOp, Start, getLexer().getLoc()));
 810:       return ParseStatus::Success;
 811:     }
 812:     if (IsPD)
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 813-840 / 第 813-840 行
```cpp
 813:       return Error(getLexer().getLoc(), "expected (");
 814: 
 815:     return ParseStatus::NoMatch;
 816:   }
 817:   Parser.Lex();
 818: 
 819:   // Check for constant dereference & MIT-style displacement
 820:   if (!HasDisplacement && isExpr()) {
 821:     if (Parser.parseExpression(MemOp.OuterDisp))
 822:       return ParseStatus::Failure;
 823:     HasDisplacement = true;
 824: 
 825:     // If we're not followed by a comma, we're a constant dereference.
 826:     if (getLexer().isNot(AsmToken::Comma)) {
 827:       MemOp.Op = M68kMemOp::Kind::Addr;
 828:       Operands.push_back(
 829:           M68kOperand::createMemOp(MemOp, Start, getLexer().getLoc()));
 830:       return ParseStatus::Success;
 831:     }
 832: 
 833:     Parser.Lex();
 834:   }
 835: 
 836:   Result = parseRegister(MemOp.OuterReg);
 837:   if (Result.isFailure())
 838:     return ParseStatus::Failure;
 839: 
 840:   if (!Result.isSuccess())
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 841-868 / 第 841-868 行
```cpp
 841:     return Error(getLexer().getLoc(), "expected register");
 842: 
 843:   // Check for Index.
 844:   bool HasIndex = false;
 845:   if (Parser.getTok().is(AsmToken::Comma)) {
 846:     Parser.Lex();
 847: 
 848:     Result = parseRegister(MemOp.InnerReg);
 849:     if (Result.isFailure())
 850:       return Result;
 851: 
 852:     if (Result.isNoMatch())
 853:       return Error(getLexer().getLoc(), "expected register");
 854: 
 855:     // TODO: parse size, scale and inner displacement.
 856:     MemOp.Size = 4;
 857:     MemOp.Scale = 1;
 858:     MemOp.InnerDisp = MCConstantExpr::create(0, Parser.getContext(), true, 4);
 859:     HasIndex = true;
 860:   }
 861: 
 862:   if (Parser.getTok().isNot(AsmToken::RParen))
 863:     return Error(getLexer().getLoc(), "expected )");
 864:   Parser.Lex();
 865: 
 866:   bool IsPI = false;
 867:   if (!IsPD && Parser.getTok().is(AsmToken::Plus)) {
 868:     Parser.Lex();
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 869-896 / 第 869-896 行
```cpp
 869:     IsPI = true;
 870:   }
 871: 
 872:   SMLoc End = getLexer().getLoc();
 873: 
 874:   unsigned OpCount = IsPD + IsPI + (HasIndex || HasDisplacement);
 875:   if (OpCount > 1)
 876:     return Error(Start, "only one of post-increment, pre-decrement or "
 877:                         "displacement can be used");
 878: 
 879:   if (IsPD) {
 880:     MemOp.Op = M68kMemOp::Kind::RegPreDecrement;
 881:   } else if (IsPI) {
 882:     MemOp.Op = M68kMemOp::Kind::RegPostIncrement;
 883:   } else if (HasIndex) {
 884:     MemOp.Op = M68kMemOp::Kind::RegIndirectDisplacementIndex;
 885:   } else if (HasDisplacement) {
 886:     MemOp.Op = M68kMemOp::Kind::RegIndirectDisplacement;
 887:   } else {
 888:     MemOp.Op = M68kMemOp::Kind::RegIndirect;
 889:   }
 890: 
 891:   Operands.push_back(M68kOperand::createMemOp(MemOp, Start, End));
 892:   return ParseStatus::Success;
 893: }
 894: 
 895: ParseStatus M68kAsmParser::parseRegOrMoveMask(OperandVector &Operands) {
 896:   SMLoc Start = getLexer().getLoc();
```
- **EN**: The range implements or declares functions including `M68kAsmParser::parseRegOrMoveMask`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kAsmParser::parseRegOrMoveMask` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 897-924 / 第 897-924 行
```cpp
 897:   M68kMemOp MemOp(M68kMemOp::Kind::RegMask);
 898:   MemOp.RegMask = 0;
 899: 
 900:   for (;;) {
 901:     bool IsFirstRegister =
 902:         (MemOp.Op == M68kMemOp::Kind::RegMask) && (MemOp.RegMask == 0);
 903: 
 904:     MCRegister FirstRegister;
 905:     ParseStatus Result = parseRegister(FirstRegister);
 906:     if (IsFirstRegister && Result.isNoMatch())
 907:       return ParseStatus::NoMatch;
 908:     if (!Result.isSuccess())
 909:       return Error(getLexer().getLoc(), "expected start register");
 910: 
 911:     MCRegister LastRegister = FirstRegister;
 912:     if (parseOptionalToken(AsmToken::Minus)) {
 913:       Result = parseRegister(LastRegister);
 914:       if (!Result.isSuccess())
 915:         return Error(getLexer().getLoc(), "expected end register");
 916:     }
 917: 
 918:     unsigned FirstRegisterIndex = getRegisterIndex(FirstRegister);
 919:     unsigned LastRegisterIndex = getRegisterIndex(LastRegister);
 920: 
 921:     uint16_t NumNewBits = LastRegisterIndex - FirstRegisterIndex + 1;
 922:     uint16_t NewMaskBits = ((1 << NumNewBits) - 1) << FirstRegisterIndex;
 923: 
 924:     if (IsFirstRegister && (FirstRegister == LastRegister)) {
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 925-952 / 第 925-952 行
```cpp
 925:       // First register range is a single register, simplify to just Reg
 926:       // so that it matches more operands.
 927:       MemOp.Op = M68kMemOp::Kind::Reg;
 928:       MemOp.OuterReg = FirstRegister;
 929:     } else {
 930:       if (MemOp.Op == M68kMemOp::Kind::Reg) {
 931:         // This is the second register being specified - expand the Reg operand
 932:         // into a mask first.
 933:         MemOp.Op = M68kMemOp::Kind::RegMask;
 934:         MemOp.RegMask = 1 << getRegisterIndex(MemOp.OuterReg);
 935: 
 936:         if (MemOp.RegMask == 0)
 937:           return Error(getLexer().getLoc(),
 938:                        "special registers cannot be used in register masks");
 939:       }
 940: 
 941:       if ((FirstRegisterIndex >= 16) || (LastRegisterIndex >= 16))
 942:         return Error(getLexer().getLoc(),
 943:                      "special registers cannot be used in register masks");
 944: 
 945:       if (NewMaskBits & MemOp.RegMask)
 946:         return Error(getLexer().getLoc(), "conflicting masked registers");
 947: 
 948:       MemOp.RegMask |= NewMaskBits;
 949:     }
 950: 
 951:     if (!parseOptionalToken(AsmToken::Slash))
 952:       break;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 953-980 / 第 953-980 行
```cpp
 953:   }
 954: 
 955:   Operands.push_back(
 956:       M68kOperand::createMemOp(MemOp, Start, getLexer().getLoc()));
 957:   return ParseStatus::Success;
 958: }
 959: 
 960: void M68kAsmParser::eatComma() {
 961:   if (Parser.getTok().is(AsmToken::Comma)) {
 962:     Parser.Lex();
 963:   }
 964: }
 965: 
 966: bool M68kAsmParser::parseInstruction(ParseInstructionInfo &Info, StringRef Name,
 967:                                      SMLoc NameLoc, OperandVector &Operands) {
 968:   SMLoc Start = getLexer().getLoc();
 969:   Operands.push_back(M68kOperand::createToken(Name, Start, Start));
 970: 
 971:   bool First = true;
 972:   while (Parser.getTok().isNot(AsmToken::EndOfStatement)) {
 973:     if (!First) {
 974:       eatComma();
 975:     } else {
 976:       First = false;
 977:     }
 978: 
 979:     ParseStatus MatchResult = MatchOperandParserImpl(Operands, Name);
 980:     if (MatchResult.isSuccess())
```
- **EN**: The range implements or declares functions including `M68kAsmParser::eatComma`, `M68kAsmParser::parseInstruction`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `M68kAsmParser::eatComma`, `M68kAsmParser::parseInstruction` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 981-1008 / 第 981-1008 行
```cpp
 981:       continue;
 982: 
 983:     // Add custom operand formats here...
 984:     SMLoc Loc = getLexer().getLoc();
 985:     Parser.eatToEndOfStatement();
 986:     return Error(Loc, "unexpected token parsing operands");
 987:   }
 988: 
 989:   // Eat EndOfStatement.
 990:   Parser.Lex();
 991:   return false;
 992: }
 993: 
 994: bool M68kAsmParser::invalidOperand(SMLoc Loc, OperandVector const &Operands,
 995:                                    uint64_t const &ErrorInfo) {
 996:   SMLoc ErrorLoc = Loc;
 997:   char const *Diag = 0;
 998: 
 999:   if (ErrorInfo != ~0U) {
1000:     if (ErrorInfo >= Operands.size()) {
1001:       Diag = "too few operands for instruction.";
1002:     } else {
1003:       auto const &Op = (M68kOperand const &)*Operands[ErrorInfo];
1004:       if (Op.getStartLoc() != SMLoc()) {
1005:         ErrorLoc = Op.getStartLoc();
1006:       }
1007:     }
1008:   }
```
- **EN**: The range implements or declares functions including `M68kAsmParser::invalidOperand`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kAsmParser::invalidOperand` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1009-1036 / 第 1009-1036 行
```cpp
1009: 
1010:   if (!Diag) {
1011:     Diag = "invalid operand for instruction";
1012:   }
1013: 
1014:   return Error(ErrorLoc, Diag);
1015: }
1016: 
1017: bool M68kAsmParser::missingFeature(SMLoc Loc, uint64_t const &ErrorInfo) {
1018:   return Error(Loc, "instruction requires a CPU feature not currently enabled");
1019: }
1020: 
1021: bool M68kAsmParser::emit(MCInst &Inst, SMLoc Loc, MCStreamer &Out) const {
1022:   Inst.setLoc(Loc);
1023:   Out.emitInstruction(Inst, *STI);
1024: 
1025:   return false;
1026: }
1027: 
1028: bool M68kAsmParser::matchAndEmitInstruction(SMLoc Loc, unsigned &Opcode,
1029:                                             OperandVector &Operands,
1030:                                             MCStreamer &Out,
1031:                                             uint64_t &ErrorInfo,
1032:                                             bool MatchingInlineAsm) {
1033:   MCInst Inst;
1034:   unsigned MatchResult =
1035:       MatchInstructionImpl(Operands, Inst, ErrorInfo, MatchingInlineAsm);
1036: 
```
- **EN**: The range implements or declares functions including `M68kAsmParser::missingFeature`, `M68kAsmParser::emit`, `M68kAsmParser::matchAndEmitInstruction`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kAsmParser::missingFeature`, `M68kAsmParser::emit`, `M68kAsmParser::matchAndEmitInstruction` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1037-1064 / 第 1037-1064 行
```cpp
1037:   switch (MatchResult) {
1038:   case Match_Success:
1039:     return emit(Inst, Loc, Out);
1040:   case Match_MissingFeature:
1041:     return missingFeature(Loc, ErrorInfo);
1042:   case Match_InvalidOperand:
1043:     return invalidOperand(Loc, Operands, ErrorInfo);
1044:   case Match_MnemonicFail:
1045:     return Error(Loc, "invalid instruction");
1046:   default:
1047:     return true;
1048:   }
1049: }
1050: 
1051: void M68kOperand::print(raw_ostream &OS, const MCAsmInfo &MAI) const {
1052:   switch (Kind) {
1053:   case KindTy::Invalid:
1054:     OS << "invalid";
1055:     break;
1056: 
1057:   case KindTy::Token:
1058:     OS << "token '" << Token << "'";
1059:     break;
1060: 
1061:   case KindTy::Imm: {
1062:     int64_t Value;
1063:     Expr->evaluateAsAbsolute(Value);
1064:     OS << "immediate " << Value;
```
- **EN**: The range implements or declares functions including `M68kOperand::print`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这一段实现或声明了 `M68kOperand::print` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 1065-1072 / 第 1065-1072 行
```cpp
1065:     break;
1066:   }
1067: 
1068:   case KindTy::MemOp:
1069:     MemOp.print(OS);
1070:     break;
1071:   }
1072: }
```
- **EN**: This span continues the file's main responsibility: this file parses target assembly syntax into MC/LLVM instructions for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **MC layer**: Uses LLVM MC structures for encoding, printing, parsing, or disassembly. / 使用 LLVM MC 结构进行编码、打印、解析或反汇编。
- **MC streaming**: Emits sections, symbols, expressions, and encoded bytes to object or assembly output. / 向目标文件或汇编输出节、符号、表达式和编码字节。
- **Assembly parsing**: Reads textual assembly operands, directives, and mnemonics. / 读取文本汇编的操作数、伪指令和助记符。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Register classes**: Groups registers by capability so instruction operands and allocators can constrain them. / 按能力对寄存器分组，以便指令操作数和分配器施加约束。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。

## Dependencies / 依赖关系
- `M68kInstrInfo.h`
- `M68kRegisterInfo.h`
- `MCTargetDesc/M68kMCAsmInfo.h`
- `TargetInfo/M68kTargetInfo.h`
- `llvm/MC/MCContext.h`
- `llvm/MC/MCParser/AsmLexer.h`
- `llvm/MC/MCParser/MCParsedAsmOperand.h`
- `llvm/MC/MCParser/MCTargetAsmParser.h`
- `llvm/MC/MCStreamer.h`
- `llvm/MC/TargetRegistry.h`
- `sstream`
- `M68kGenAsmMatcher.inc`
