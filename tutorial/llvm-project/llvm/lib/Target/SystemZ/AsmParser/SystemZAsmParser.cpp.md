# SystemZAsmParser.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/AsmParser/SystemZAsmParser.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file parses target assembly syntax into MC/LLVM instructions for the SystemZ backend.
- **用途 (CN)**: 该文件用于 SystemZ 后端，负责将目标汇编语法解析为 MC/LLVM 指令。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40 / 第 1-40 行
```cpp
   1: //===-- SystemZAsmParser.cpp - Parse SystemZ assembly instructions --------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: #include "MCTargetDesc/SystemZGNUInstPrinter.h"
  10: #include "MCTargetDesc/SystemZMCAsmInfo.h"
  11: #include "MCTargetDesc/SystemZMCTargetDesc.h"
  12: #include "MCTargetDesc/SystemZTargetStreamer.h"
  13: #include "TargetInfo/SystemZTargetInfo.h"
  14: #include "llvm/ADT/STLExtras.h"
  15: #include "llvm/ADT/SmallVector.h"
  16: #include "llvm/ADT/StringExtras.h"
  17: #include "llvm/ADT/StringRef.h"
  18: #include "llvm/MC/MCAsmInfo.h"
  19: #include "llvm/MC/MCContext.h"
  20: #include "llvm/MC/MCExpr.h"
  21: #include "llvm/MC/MCInst.h"
  22: #include "llvm/MC/MCInstBuilder.h"
  23: #include "llvm/MC/MCInstrInfo.h"
  24: #include "llvm/MC/MCParser/AsmLexer.h"
  25: #include "llvm/MC/MCParser/MCAsmParser.h"
  26: #include "llvm/MC/MCParser/MCAsmParserExtension.h"
  27: #include "llvm/MC/MCParser/MCParsedAsmOperand.h"
  28: #include "llvm/MC/MCParser/MCTargetAsmParser.h"
  29: #include "llvm/MC/MCStreamer.h"
  30: #include "llvm/MC/MCSubtargetInfo.h"
  31: #include "llvm/MC/TargetRegistry.h"
  32: #include "llvm/Support/Casting.h"
  33: #include "llvm/Support/Compiler.h"
  34: #include "llvm/Support/ErrorHandling.h"
  35: #include "llvm/Support/SMLoc.h"
  36: #include "llvm/TargetParser/SubtargetFeature.h"
  37: #include <algorithm>
  38: #include <cassert>
  39: #include <cstddef>
  40: #include <cstdint>
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `SystemZGNUInstPrinter.h`, `SystemZMCAsmInfo.h`, `SystemZMCTargetDesc.h`, `SystemZTargetStreamer.h`, `SystemZTargetInfo.h`, `STLExtras.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `SystemZGNUInstPrinter.h`, `SystemZMCAsmInfo.h`, `SystemZMCTargetDesc.h`, `SystemZTargetStreamer.h`, `SystemZTargetInfo.h`, `STLExtras.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 41-80 / 第 41-80 行
```cpp
  41: #include <iterator>
  42: #include <memory>
  43: #include <string>
  44: 
  45: using namespace llvm;
  46: 
  47: // Return true if Expr is in the range [MinValue, MaxValue]. If AllowSymbol
  48: // is true any MCExpr is accepted (address displacement).
  49: static bool inRange(const MCExpr *Expr, int64_t MinValue, int64_t MaxValue,
  50:                     bool AllowSymbol = false) {
  51:   if (auto *CE = dyn_cast<MCConstantExpr>(Expr)) {
  52:     int64_t Value = CE->getValue();
  53:     return Value >= MinValue && Value <= MaxValue;
  54:   }
  55:   return AllowSymbol;
  56: }
  57: 
  58: namespace {
  59: 
  60: enum RegisterKind {
  61:   GR32Reg,
  62:   GRH32Reg,
  63:   GR64Reg,
  64:   GR128Reg,
  65:   FP16Reg,
  66:   FP32Reg,
  67:   FP64Reg,
  68:   FP128Reg,
  69:   VR16Reg,
  70:   VR32Reg,
  71:   VR64Reg,
  72:   VR128Reg,
  73:   AR32Reg,
  74:   CR64Reg,
  75: };
  76: 
  77: enum MemoryKind {
  78:   BDMem,
  79:   BDXMem,
  80:   BDLMem,
```
- **EN**: It imports dependencies such as `iterator`, `memory`, `string` that expose the LLVM and target interfaces used in later logic. The range implements or declares functions including `inRange`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 它引入了 `iterator`, `memory`, `string` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段实现或声明了 `inRange` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 81-120 / 第 81-120 行
```cpp
  81:   BDRMem,
  82:   BDVMem,
  83:   LXAMem
  84: };
  85: 
  86: class SystemZOperand : public MCParsedAsmOperand {
  87: private:
  88:   enum OperandKind {
  89:     KindInvalid,
  90:     KindToken,
  91:     KindReg,
  92:     KindImm,
  93:     KindImmTLS,
  94:     KindMem
  95:   };
  96: 
  97:   OperandKind Kind;
  98:   SMLoc StartLoc, EndLoc;
  99: 
 100:   // A string of length Length, starting at Data.
 101:   struct TokenOp {
 102:     const char *Data;
 103:     unsigned Length;
 104:   };
 105: 
 106:   // LLVM register Num, which has kind Kind.  In some ways it might be
 107:   // easier for this class to have a register bank (general, floating-point
 108:   // or access) and a raw register number (0-15).  This would postpone the
 109:   // interpretation of the operand to the add*() methods and avoid the need
 110:   // for context-dependent parsing.  However, we do things the current way
 111:   // because of the virtual getReg() method, which needs to distinguish
 112:   // between (say) %r0 used as a single register and %r0 used as a pair.
 113:   // Context-dependent parsing can also give us slightly better error
 114:   // messages when invalid pairs like %r1 are used.
 115:   struct RegOp {
 116:     RegisterKind Kind;
 117:     unsigned Num;
 118:   };
 119: 
 120:   // Base + Disp + Index, where Base and Index are LLVM registers or 0.
```
- **EN**: This block declares or refines TableGen records such as `SystemZOperand`.
- **CN**: 该代码块声明或细化了 `SystemZOperand` 等 TableGen 记录。

### Lines 121-160 / 第 121-160 行
```cpp
 121:   // MemKind says what type of memory this is and RegKind says what type
 122:   // the base register has (GR32Reg or GR64Reg).  Length is the operand
 123:   // length for D(L,B)-style operands, otherwise it is null.
 124:   struct MemOp {
 125:     unsigned Base : 12;
 126:     unsigned Index : 12;
 127:     unsigned MemKind : 4;
 128:     unsigned RegKind : 4;
 129:     const MCExpr *Disp;
 130:     union {
 131:       const MCExpr *Imm;
 132:       unsigned Reg;
 133:     } Length;
 134:   };
 135: 
 136:   // Imm is an immediate operand, and Sym is an optional TLS symbol
 137:   // for use with a __tls_get_offset marker relocation.
 138:   struct ImmTLSOp {
 139:     const MCExpr *Imm;
 140:     const MCExpr *Sym;
 141:   };
 142: 
 143:   union {
 144:     TokenOp Token;
 145:     RegOp Reg;
 146:     const MCExpr *Imm;
 147:     ImmTLSOp ImmTLS;
 148:     MemOp Mem;
 149:   };
 150: 
 151:   void addExpr(MCInst &Inst, const MCExpr *Expr) const {
 152:     // Add as immediates when possible.  Null MCExpr = 0.
 153:     if (!Expr)
 154:       Inst.addOperand(MCOperand::createImm(0));
 155:     else if (auto *CE = dyn_cast<MCConstantExpr>(Expr))
 156:       Inst.addOperand(MCOperand::createImm(CE->getValue()));
 157:     else
 158:       Inst.addOperand(MCOperand::createExpr(Expr));
 159:   }
 160: 
```
- **EN**: It introduces interface types such as `MemOp`, `ImmTLSOp`, shaping how other backend components interact with this file. The range implements or declares functions including `addExpr`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 它引入了 `MemOp`, `ImmTLSOp` 等接口类型，定义了其他后端组件与本文件交互的方式。 这一段实现或声明了 `addExpr` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 161-200 / 第 161-200 行
```cpp
 161: public:
 162:   SystemZOperand(OperandKind Kind, SMLoc StartLoc, SMLoc EndLoc)
 163:       : Kind(Kind), StartLoc(StartLoc), EndLoc(EndLoc) {}
 164: 
 165:   // Create particular kinds of operand.
 166:   static std::unique_ptr<SystemZOperand> createInvalid(SMLoc StartLoc,
 167:                                                        SMLoc EndLoc) {
 168:     return std::make_unique<SystemZOperand>(KindInvalid, StartLoc, EndLoc);
 169:   }
 170: 
 171:   static std::unique_ptr<SystemZOperand> createToken(StringRef Str, SMLoc Loc) {
 172:     auto Op = std::make_unique<SystemZOperand>(KindToken, Loc, Loc);
 173:     Op->Token.Data = Str.data();
 174:     Op->Token.Length = Str.size();
 175:     return Op;
 176:   }
 177: 
 178:   static std::unique_ptr<SystemZOperand>
 179:   createReg(RegisterKind Kind, unsigned Num, SMLoc StartLoc, SMLoc EndLoc) {
 180:     auto Op = std::make_unique<SystemZOperand>(KindReg, StartLoc, EndLoc);
 181:     Op->Reg.Kind = Kind;
 182:     Op->Reg.Num = Num;
 183:     return Op;
 184:   }
 185: 
 186:   static std::unique_ptr<SystemZOperand>
 187:   createImm(const MCExpr *Expr, SMLoc StartLoc, SMLoc EndLoc) {
 188:     auto Op = std::make_unique<SystemZOperand>(KindImm, StartLoc, EndLoc);
 189:     Op->Imm = Expr;
 190:     return Op;
 191:   }
 192: 
 193:   static std::unique_ptr<SystemZOperand>
 194:   createMem(MemoryKind MemKind, RegisterKind RegKind, unsigned Base,
 195:             const MCExpr *Disp, unsigned Index, const MCExpr *LengthImm,
 196:             unsigned LengthReg, SMLoc StartLoc, SMLoc EndLoc) {
 197:     auto Op = std::make_unique<SystemZOperand>(KindMem, StartLoc, EndLoc);
 198:     Op->Mem.MemKind = MemKind;
 199:     Op->Mem.RegKind = RegKind;
 200:     Op->Mem.Base = Base;
```
- **EN**: The range implements or declares functions including `SystemZOperand`, `createToken`, `createReg`, `createImm`, `createMem`.
- **CN**: 这一段实现或声明了 `SystemZOperand`, `createToken`, `createReg`, `createImm`, `createMem` 等函数。

### Lines 201-240 / 第 201-240 行
```cpp
 201:     Op->Mem.Index = Index;
 202:     Op->Mem.Disp = Disp;
 203:     if (MemKind == BDLMem)
 204:       Op->Mem.Length.Imm = LengthImm;
 205:     if (MemKind == BDRMem)
 206:       Op->Mem.Length.Reg = LengthReg;
 207:     return Op;
 208:   }
 209: 
 210:   static std::unique_ptr<SystemZOperand>
 211:   createImmTLS(const MCExpr *Imm, const MCExpr *Sym,
 212:                SMLoc StartLoc, SMLoc EndLoc) {
 213:     auto Op = std::make_unique<SystemZOperand>(KindImmTLS, StartLoc, EndLoc);
 214:     Op->ImmTLS.Imm = Imm;
 215:     Op->ImmTLS.Sym = Sym;
 216:     return Op;
 217:   }
 218: 
 219:   // Token operands
 220:   bool isToken() const override {
 221:     return Kind == KindToken;
 222:   }
 223:   StringRef getToken() const {
 224:     assert(Kind == KindToken && "Not a token");
 225:     return StringRef(Token.Data, Token.Length);
 226:   }
 227: 
 228:   // Register operands.
 229:   bool isReg() const override {
 230:     return Kind == KindReg;
 231:   }
 232:   bool isReg(RegisterKind RegKind) const {
 233:     return Kind == KindReg && Reg.Kind == RegKind;
 234:   }
 235:   MCRegister getReg() const override {
 236:     assert(Kind == KindReg && "Not a register");
 237:     return Reg.Num;
 238:   }
 239: 
 240:   // Immediate operands.
```
- **EN**: The range implements or declares functions including `createImmTLS`, `getToken`, `isReg`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `createImmTLS`, `getToken`, `isReg` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 241-280 / 第 241-280 行
```cpp
 241:   bool isImm() const override {
 242:     return Kind == KindImm;
 243:   }
 244:   bool isImm(int64_t MinValue, int64_t MaxValue) const {
 245:     return Kind == KindImm && inRange(Imm, MinValue, MaxValue, true);
 246:   }
 247:   const MCExpr *getImm() const {
 248:     assert(Kind == KindImm && "Not an immediate");
 249:     return Imm;
 250:   }
 251: 
 252:   // Immediate operands with optional TLS symbol.
 253:   bool isImmTLS() const {
 254:     return Kind == KindImmTLS;
 255:   }
 256: 
 257:   const ImmTLSOp getImmTLS() const {
 258:     assert(Kind == KindImmTLS && "Not a TLS immediate");
 259:     return ImmTLS;
 260:   }
 261: 
 262:   // Memory operands.
 263:   bool isMem() const override {
 264:     return Kind == KindMem;
 265:   }
 266:   bool isMem(MemoryKind MemKind) const {
 267:     return (Kind == KindMem &&
 268:             (Mem.MemKind == MemKind ||
 269:              // A BDMem can be treated as a BDXMem in which the index
 270:              // register field is 0.
 271:              (Mem.MemKind == BDMem && MemKind == BDXMem)));
 272:   }
 273:   bool isMem(MemoryKind MemKind, RegisterKind RegKind) const {
 274:     return isMem(MemKind) && Mem.RegKind == RegKind;
 275:   }
 276:   bool isMemDisp12(MemoryKind MemKind, RegisterKind RegKind) const {
 277:     return isMem(MemKind, RegKind) && inRange(Mem.Disp, 0, 0xfff, true);
 278:   }
 279:   bool isMemDisp20(MemoryKind MemKind, RegisterKind RegKind) const {
 280:     return isMem(MemKind, RegKind) && inRange(Mem.Disp, -524288, 524287, true);
```
- **EN**: The range implements or declares functions including `isImm`, `isImmTLS`, `getImmTLS`, `isMem`, `isMem`. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `isImm`, `isImmTLS`, `getImmTLS`, `isMem`, `isMem` 等函数。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 281-320 / 第 281-320 行
```cpp
 281:   }
 282:   bool isMemDisp12Len4(RegisterKind RegKind) const {
 283:     return isMemDisp12(BDLMem, RegKind) && inRange(Mem.Length.Imm, 1, 0x10);
 284:   }
 285:   bool isMemDisp12Len8(RegisterKind RegKind) const {
 286:     return isMemDisp12(BDLMem, RegKind) && inRange(Mem.Length.Imm, 1, 0x100);
 287:   }
 288: 
 289:   const MemOp& getMem() const {
 290:     assert(Kind == KindMem && "Not a Mem operand");
 291:     return Mem;
 292:   }
 293: 
 294:   // Override MCParsedAsmOperand.
 295:   SMLoc getStartLoc() const override { return StartLoc; }
 296:   SMLoc getEndLoc() const override { return EndLoc; }
 297:   void print(raw_ostream &OS, const MCAsmInfo &MAI) const override;
 298: 
 299:   /// getLocRange - Get the range between the first and last token of this
 300:   /// operand.
 301:   SMRange getLocRange() const { return SMRange(StartLoc, EndLoc); }
 302: 
 303:   // Used by the TableGen code to add particular types of operand
 304:   // to an instruction.
 305:   void addRegOperands(MCInst &Inst, unsigned N) const {
 306:     assert(N == 1 && "Invalid number of operands");
 307:     Inst.addOperand(MCOperand::createReg(getReg()));
 308:   }
 309:   void addImmOperands(MCInst &Inst, unsigned N) const {
 310:     assert(N == 1 && "Invalid number of operands");
 311:     addExpr(Inst, getImm());
 312:   }
 313:   void addBDAddrOperands(MCInst &Inst, unsigned N) const {
 314:     assert(N == 2 && "Invalid number of operands");
 315:     assert(isMem(BDMem) && "Invalid operand type");
 316:     Inst.addOperand(MCOperand::createReg(Mem.Base));
 317:     addExpr(Inst, Mem.Disp);
 318:   }
 319:   void addBDXAddrOperands(MCInst &Inst, unsigned N) const {
 320:     assert(N == 3 && "Invalid number of operands");
```
- **EN**: The range implements or declares functions including `isMemDisp12Len4`, `isMemDisp12Len8`, `getMem`, `getLocRange`, `addRegOperands`. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `isMemDisp12Len4`, `isMemDisp12Len8`, `getMem`, `getLocRange`, `addRegOperands` 等函数。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 321-360 / 第 321-360 行
```cpp
 321:     assert(isMem(BDXMem) && "Invalid operand type");
 322:     Inst.addOperand(MCOperand::createReg(Mem.Base));
 323:     addExpr(Inst, Mem.Disp);
 324:     Inst.addOperand(MCOperand::createReg(Mem.Index));
 325:   }
 326:   void addBDLAddrOperands(MCInst &Inst, unsigned N) const {
 327:     assert(N == 3 && "Invalid number of operands");
 328:     assert(isMem(BDLMem) && "Invalid operand type");
 329:     Inst.addOperand(MCOperand::createReg(Mem.Base));
 330:     addExpr(Inst, Mem.Disp);
 331:     addExpr(Inst, Mem.Length.Imm);
 332:   }
 333:   void addBDRAddrOperands(MCInst &Inst, unsigned N) const {
 334:     assert(N == 3 && "Invalid number of operands");
 335:     assert(isMem(BDRMem) && "Invalid operand type");
 336:     Inst.addOperand(MCOperand::createReg(Mem.Base));
 337:     addExpr(Inst, Mem.Disp);
 338:     Inst.addOperand(MCOperand::createReg(Mem.Length.Reg));
 339:   }
 340:   void addBDVAddrOperands(MCInst &Inst, unsigned N) const {
 341:     assert(N == 3 && "Invalid number of operands");
 342:     assert(isMem(BDVMem) && "Invalid operand type");
 343:     Inst.addOperand(MCOperand::createReg(Mem.Base));
 344:     addExpr(Inst, Mem.Disp);
 345:     Inst.addOperand(MCOperand::createReg(Mem.Index));
 346:   }
 347:   void addLXAAddrOperands(MCInst &Inst, unsigned N) const {
 348:     assert(N == 3 && "Invalid number of operands");
 349:     assert(isMem(LXAMem) && "Invalid operand type");
 350:     Inst.addOperand(MCOperand::createReg(Mem.Base));
 351:     addExpr(Inst, Mem.Disp);
 352:     Inst.addOperand(MCOperand::createReg(Mem.Index));
 353:   }
 354:   void addImmTLSOperands(MCInst &Inst, unsigned N) const {
 355:     assert(N == 2 && "Invalid number of operands");
 356:     assert(Kind == KindImmTLS && "Invalid operand type");
 357:     addExpr(Inst, ImmTLS.Imm);
 358:     if (ImmTLS.Sym)
 359:       addExpr(Inst, ImmTLS.Sym);
 360:   }
```
- **EN**: The range implements or declares functions including `addBDLAddrOperands`, `addBDRAddrOperands`, `addBDVAddrOperands`, `addLXAAddrOperands`, `addImmTLSOperands`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `addBDLAddrOperands`, `addBDRAddrOperands`, `addBDVAddrOperands`, `addLXAAddrOperands`, `addImmTLSOperands` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 361-400 / 第 361-400 行
```cpp
 361: 
 362:   // Used by the TableGen code to check for particular operand types.
 363:   bool isGR32() const { return isReg(GR32Reg); }
 364:   bool isGRH32() const { return isReg(GRH32Reg); }
 365:   bool isGRX32() const { return false; }
 366:   bool isGR64() const { return isReg(GR64Reg); }
 367:   bool isGR128() const { return isReg(GR128Reg); }
 368:   bool isADDR32() const { return isReg(GR32Reg); }
 369:   bool isADDR64() const { return isReg(GR64Reg); }
 370:   bool isADDR128() const { return false; }
 371:   bool isFP16() const { return isReg(FP16Reg); }
 372:   bool isFP32() const { return isReg(FP32Reg); }
 373:   bool isFP64() const { return isReg(FP64Reg); }
 374:   bool isFP128() const { return isReg(FP128Reg); }
 375:   bool isVR16() const { return isReg(VR16Reg); }
 376:   bool isVR32() const { return isReg(VR32Reg); }
 377:   bool isVR64() const { return isReg(VR64Reg); }
 378:   bool isVF128() const { return false; }
 379:   bool isVR128() const { return isReg(VR128Reg); }
 380:   bool isAR32() const { return isReg(AR32Reg); }
 381:   bool isCR64() const { return isReg(CR64Reg); }
 382:   bool isAnyReg() const { return (isReg() || isImm(0, 15)); }
 383:   bool isBDAddr32Disp12() const { return isMemDisp12(BDMem, GR32Reg); }
 384:   bool isBDAddr32Disp20() const { return isMemDisp20(BDMem, GR32Reg); }
 385:   bool isBDAddr64Disp12() const { return isMemDisp12(BDMem, GR64Reg); }
 386:   bool isBDAddr64Disp20() const { return isMemDisp20(BDMem, GR64Reg); }
 387:   bool isBDXAddr64Disp12() const { return isMemDisp12(BDXMem, GR64Reg); }
 388:   bool isBDXAddr64Disp20() const { return isMemDisp20(BDXMem, GR64Reg); }
 389:   bool isBDLAddr64Disp12Len4() const { return isMemDisp12Len4(GR64Reg); }
 390:   bool isBDLAddr64Disp12Len8() const { return isMemDisp12Len8(GR64Reg); }
 391:   bool isBDRAddr64Disp12() const { return isMemDisp12(BDRMem, GR64Reg); }
 392:   bool isBDVAddr64Disp12() const { return isMemDisp12(BDVMem, GR64Reg); }
 393:   bool isLXAAddr64Disp20() const { return isMemDisp20(LXAMem, GR64Reg); }
 394:   bool isU1Imm() const { return isImm(0, 1); }
 395:   bool isU2Imm() const { return isImm(0, 3); }
 396:   bool isU3Imm() const { return isImm(0, 7); }
 397:   bool isU4Imm() const { return isImm(0, 15); }
 398:   bool isU8Imm() const { return isImm(0, 255); }
 399:   bool isS8Imm() const { return isImm(-128, 127); }
 400:   bool isU12Imm() const { return isImm(0, 4095); }
```
- **EN**: The range implements or declares functions including `isGR32`, `isGRH32`, `isGRX32`, `isGR64`, `isGR128`.
- **CN**: 这一段实现或声明了 `isGR32`, `isGRH32`, `isGRX32`, `isGR64`, `isGR128` 等函数。

### Lines 401-440 / 第 401-440 行
```cpp
 401:   bool isU16Imm() const { return isImm(0, 65535); }
 402:   bool isS16Imm() const { return isImm(-32768, 32767); }
 403:   bool isU32Imm() const { return isImm(0, (1LL << 32) - 1); }
 404:   bool isS32Imm() const { return isImm(-(1LL << 31), (1LL << 31) - 1); }
 405:   bool isU48Imm() const { return isImm(0, (1LL << 48) - 1); }
 406: };
 407: 
 408: class SystemZAsmParser : public MCTargetAsmParser {
 409: #define GET_ASSEMBLER_HEADER
 410: #include "SystemZGenAsmMatcher.inc"
 411: 
 412: private:
 413:   MCAsmParser &Parser;
 414: 
 415:   // A vector to contain the stack of FeatureBitsets created by `.machine push`.
 416:   // `.machine pop` pops the top of the stack and uses `setAvailableFeatures` to
 417:   // apply the result.
 418:   SmallVector<FeatureBitset> MachineStack;
 419: 
 420:   enum RegisterGroup {
 421:     RegGR,
 422:     RegFP,
 423:     RegV,
 424:     RegAR,
 425:     RegCR
 426:   };
 427:   struct Register {
 428:     RegisterGroup Group;
 429:     unsigned Num;
 430:     SMLoc StartLoc, EndLoc;
 431:   };
 432: 
 433:   SystemZTargetStreamer &getTargetStreamer() {
 434:     assert(getParser().getStreamer().getTargetStreamer() &&
 435:            "do not have a target streamer");
 436:     MCTargetStreamer &TS = *getParser().getStreamer().getTargetStreamer();
 437:     return static_cast<SystemZTargetStreamer &>(TS);
 438:   }
 439: 
 440:   bool parseRegister(Register &Reg, bool RequirePercent,
```
- **EN**: It imports dependencies such as `SystemZGenAsmMatcher.inc` that expose the LLVM and target interfaces used in later logic. This block declares or refines TableGen records such as `SystemZAsmParser`. The range implements or declares functions including `isU16Imm`, `isS16Imm`, `isU32Imm`, `isS32Imm`, `isU48Imm`.
- **CN**: 它引入了 `SystemZGenAsmMatcher.inc` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 该代码块声明或细化了 `SystemZAsmParser` 等 TableGen 记录。 这一段实现或声明了 `isU16Imm`, `isS16Imm`, `isU32Imm`, `isS32Imm`, `isU48Imm` 等函数。

### Lines 441-480 / 第 441-480 行
```cpp
 441:                      bool RestoreOnFailure = false);
 442: 
 443:   bool parseIntegerRegister(Register &Reg, RegisterGroup Group);
 444: 
 445:   ParseStatus parseRegister(OperandVector &Operands, RegisterKind Kind);
 446: 
 447:   ParseStatus parseAnyRegister(OperandVector &Operands);
 448: 
 449:   bool parseAddress(bool &HaveReg1, Register &Reg1, bool &HaveReg2,
 450:                     Register &Reg2, const MCExpr *&Disp, const MCExpr *&Length,
 451:                     bool HasLength = false, bool HasVectorIndex = false);
 452:   bool parseAddressRegister(Register &Reg);
 453: 
 454:   bool parseDirectiveInsn(SMLoc L);
 455:   bool parseDirectiveMachine(SMLoc L);
 456:   bool parseGNUAttribute(SMLoc L);
 457: 
 458:   ParseStatus parseAddress(OperandVector &Operands, MemoryKind MemKind,
 459:                            RegisterKind RegKind);
 460: 
 461:   ParseStatus parsePCRel(OperandVector &Operands, int64_t MinVal,
 462:                          int64_t MaxVal, bool AllowTLS);
 463: 
 464:   bool parseOperand(OperandVector &Operands, StringRef Mnemonic);
 465: 
 466:   // Both the hlasm and gnu variants still rely on the basic gnu asm
 467:   // format with respect to inputs, clobbers, outputs etc.
 468:   //
 469:   // However, calling the overriden getAssemblerDialect() method in
 470:   // AsmParser is problematic. It either returns the AssemblerDialect field
 471:   // in the MCAsmInfo instance if the AssemblerDialect field in AsmParser is
 472:   // unset, otherwise it returns the private AssemblerDialect field in
 473:   // AsmParser.
 474:   //
 475:   // The problematic part is because, we forcibly set the inline asm dialect
 476:   // in the AsmParser instance in AsmPrinterInlineAsm.cpp. Soo any query
 477:   // to the overriden getAssemblerDialect function in AsmParser.cpp, will
 478:   // not return the assembler dialect set in the respective MCAsmInfo instance.
 479:   //
 480:   // For this purpose, we explicitly query the SystemZMCAsmInfo instance
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 481-520 / 第 481-520 行
```cpp
 481:   // here, to get the "correct" assembler dialect, and use it in various
 482:   // functions.
 483:   unsigned getMAIAssemblerDialect() {
 484:     return Parser.getContext().getAsmInfo().getAssemblerDialect();
 485:   }
 486: 
 487:   // An alphabetic character in HLASM is a letter from 'A' through 'Z',
 488:   // or from 'a' through 'z', or '$', '_','#', or '@'.
 489:   inline bool isHLASMAlpha(char C) {
 490:     return isAlpha(C) || llvm::is_contained("_@#$", C);
 491:   }
 492: 
 493:   // A digit in HLASM is a number from 0 to 9.
 494:   inline bool isHLASMAlnum(char C) { return isHLASMAlpha(C) || isDigit(C); }
 495: 
 496:   // Are we parsing using the AD_HLASM dialect?
 497:   inline bool isParsingHLASM() { return getMAIAssemblerDialect() == AD_HLASM; }
 498: 
 499:   // Are we parsing using the AD_GNU dialect?
 500:   inline bool isParsingGNU() { return getMAIAssemblerDialect() == AD_GNU; }
 501: 
 502: public:
 503:   SystemZAsmParser(const MCSubtargetInfo &sti, MCAsmParser &parser,
 504:                    const MCInstrInfo &MII)
 505:       : MCTargetAsmParser(sti, MII), Parser(parser) {
 506:     MCAsmParserExtension::Initialize(Parser);
 507: 
 508:     // Alias the .word directive to .short.
 509:     parser.addAliasForDirective(".word", ".short");
 510: 
 511:     // Initialize the set of available features.
 512:     setAvailableFeatures(ComputeAvailableFeatures(getSTI().getFeatureBits()));
 513:   }
 514: 
 515:   // Override MCTargetAsmParser.
 516:   ParseStatus parseDirective(AsmToken DirectiveID) override;
 517:   bool parseRegister(MCRegister &Reg, SMLoc &StartLoc, SMLoc &EndLoc) override;
 518:   bool ParseRegister(MCRegister &RegNo, SMLoc &StartLoc, SMLoc &EndLoc,
 519:                      bool RequirePercent, bool RestoreOnFailure);
 520:   ParseStatus tryParseRegister(MCRegister &Reg, SMLoc &StartLoc,
```
- **EN**: The range implements or declares functions including `getMAIAssemblerDialect`, `isHLASMAlpha`, `isHLASMAlnum`, `isParsingHLASM`, `isParsingGNU`.
- **CN**: 这一段实现或声明了 `getMAIAssemblerDialect`, `isHLASMAlpha`, `isHLASMAlnum`, `isParsingHLASM`, `isParsingGNU` 等函数。

### Lines 521-560 / 第 521-560 行
```cpp
 521:                                SMLoc &EndLoc) override;
 522:   bool parseInstruction(ParseInstructionInfo &Info, StringRef Name,
 523:                         SMLoc NameLoc, OperandVector &Operands) override;
 524:   bool matchAndEmitInstruction(SMLoc IDLoc, unsigned &Opcode,
 525:                                OperandVector &Operands, MCStreamer &Out,
 526:                                uint64_t &ErrorInfo,
 527:                                bool MatchingInlineAsm) override;
 528:   bool isLabel(AsmToken &Token) override;
 529: 
 530:   // Used by the TableGen code to parse particular operand types.
 531:   ParseStatus parseGR32(OperandVector &Operands) {
 532:     return parseRegister(Operands, GR32Reg);
 533:   }
 534:   ParseStatus parseGRH32(OperandVector &Operands) {
 535:     return parseRegister(Operands, GRH32Reg);
 536:   }
 537:   ParseStatus parseGRX32(OperandVector &Operands) {
 538:     llvm_unreachable("GRX32 should only be used for pseudo instructions");
 539:   }
 540:   ParseStatus parseGR64(OperandVector &Operands) {
 541:     return parseRegister(Operands, GR64Reg);
 542:   }
 543:   ParseStatus parseGR128(OperandVector &Operands) {
 544:     return parseRegister(Operands, GR128Reg);
 545:   }
 546:   ParseStatus parseADDR32(OperandVector &Operands) {
 547:     // For the AsmParser, we will accept %r0 for ADDR32 as well.
 548:     return parseRegister(Operands, GR32Reg);
 549:   }
 550:   ParseStatus parseADDR64(OperandVector &Operands) {
 551:     // For the AsmParser, we will accept %r0 for ADDR64 as well.
 552:     return parseRegister(Operands, GR64Reg);
 553:   }
 554:   ParseStatus parseADDR128(OperandVector &Operands) {
 555:     llvm_unreachable("Shouldn't be used as an operand");
 556:   }
 557:   ParseStatus parseFP16(OperandVector &Operands) {
 558:     return parseRegister(Operands, FP16Reg);
 559:   }
 560:   ParseStatus parseFP32(OperandVector &Operands) {
```
- **EN**: The range implements or declares functions including `parseGR32`, `parseGRH32`, `parseGRX32`, `parseGR64`, `parseGR128`. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `parseGR32`, `parseGRH32`, `parseGRX32`, `parseGR64`, `parseGR128` 等函数。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 561-600 / 第 561-600 行
```cpp
 561:     return parseRegister(Operands, FP32Reg);
 562:   }
 563:   ParseStatus parseFP64(OperandVector &Operands) {
 564:     return parseRegister(Operands, FP64Reg);
 565:   }
 566:   ParseStatus parseFP128(OperandVector &Operands) {
 567:     return parseRegister(Operands, FP128Reg);
 568:   }
 569:   ParseStatus parseVR16(OperandVector &Operands) {
 570:     return parseRegister(Operands, VR16Reg);
 571:   }
 572:   ParseStatus parseVR32(OperandVector &Operands) {
 573:     return parseRegister(Operands, VR32Reg);
 574:   }
 575:   ParseStatus parseVR64(OperandVector &Operands) {
 576:     return parseRegister(Operands, VR64Reg);
 577:   }
 578:   ParseStatus parseVF128(OperandVector &Operands) {
 579:     llvm_unreachable("Shouldn't be used as an operand");
 580:   }
 581:   ParseStatus parseVR128(OperandVector &Operands) {
 582:     return parseRegister(Operands, VR128Reg);
 583:   }
 584:   ParseStatus parseAR32(OperandVector &Operands) {
 585:     return parseRegister(Operands, AR32Reg);
 586:   }
 587:   ParseStatus parseCR64(OperandVector &Operands) {
 588:     return parseRegister(Operands, CR64Reg);
 589:   }
 590:   ParseStatus parseAnyReg(OperandVector &Operands) {
 591:     return parseAnyRegister(Operands);
 592:   }
 593:   ParseStatus parseBDAddr32(OperandVector &Operands) {
 594:     return parseAddress(Operands, BDMem, GR32Reg);
 595:   }
 596:   ParseStatus parseBDAddr64(OperandVector &Operands) {
 597:     return parseAddress(Operands, BDMem, GR64Reg);
 598:   }
 599:   ParseStatus parseBDXAddr64(OperandVector &Operands) {
 600:     return parseAddress(Operands, BDXMem, GR64Reg);
```
- **EN**: The range implements or declares functions including `parseFP64`, `parseFP128`, `parseVR16`, `parseVR32`, `parseVR64`. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `parseFP64`, `parseFP128`, `parseVR16`, `parseVR32`, `parseVR64` 等函数。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 601-640 / 第 601-640 行
```cpp
 601:   }
 602:   ParseStatus parseBDLAddr64(OperandVector &Operands) {
 603:     return parseAddress(Operands, BDLMem, GR64Reg);
 604:   }
 605:   ParseStatus parseBDRAddr64(OperandVector &Operands) {
 606:     return parseAddress(Operands, BDRMem, GR64Reg);
 607:   }
 608:   ParseStatus parseBDVAddr64(OperandVector &Operands) {
 609:     return parseAddress(Operands, BDVMem, GR64Reg);
 610:   }
 611:   ParseStatus parseLXAAddr64(OperandVector &Operands) {
 612:     return parseAddress(Operands, LXAMem, GR64Reg);
 613:   }
 614:   ParseStatus parsePCRel12(OperandVector &Operands) {
 615:     return parsePCRel(Operands, -(1LL << 12), (1LL << 12) - 1, false);
 616:   }
 617:   ParseStatus parsePCRel16(OperandVector &Operands) {
 618:     return parsePCRel(Operands, -(1LL << 16), (1LL << 16) - 1, false);
 619:   }
 620:   ParseStatus parsePCRel24(OperandVector &Operands) {
 621:     return parsePCRel(Operands, -(1LL << 24), (1LL << 24) - 1, false);
 622:   }
 623:   ParseStatus parsePCRel32(OperandVector &Operands) {
 624:     return parsePCRel(Operands, -(1LL << 32), (1LL << 32) - 1, false);
 625:   }
 626:   ParseStatus parsePCRelTLS16(OperandVector &Operands) {
 627:     return parsePCRel(Operands, -(1LL << 16), (1LL << 16) - 1, true);
 628:   }
 629:   ParseStatus parsePCRelTLS32(OperandVector &Operands) {
 630:     return parsePCRel(Operands, -(1LL << 32), (1LL << 32) - 1, true);
 631:   }
 632: };
 633: 
 634: } // end anonymous namespace
 635: 
 636: #define GET_REGISTER_MATCHER
 637: #define GET_SUBTARGET_FEATURE_NAME
 638: #define GET_MATCHER_IMPLEMENTATION
 639: #define GET_MNEMONIC_SPELL_CHECKER
 640: #include "SystemZGenAsmMatcher.inc"
```
- **EN**: It imports dependencies such as `SystemZGenAsmMatcher.inc` that expose the LLVM and target interfaces used in later logic. The range implements or declares functions including `parseBDLAddr64`, `parseBDRAddr64`, `parseBDVAddr64`, `parseLXAAddr64`, `parsePCRel12`. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 它引入了 `SystemZGenAsmMatcher.inc` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段实现或声明了 `parseBDLAddr64`, `parseBDRAddr64`, `parseBDVAddr64`, `parseLXAAddr64`, `parsePCRel12` 等函数。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 641-680 / 第 641-680 行
```cpp
 641: 
 642: // Used for the .insn directives; contains information needed to parse the
 643: // operands in the directive.
 644: struct InsnMatchEntry {
 645:   StringRef Format;
 646:   uint64_t Opcode;
 647:   int32_t NumOperands;
 648:   MatchClassKind OperandKinds[7];
 649: };
 650: 
 651: // For equal_range comparison.
 652: struct CompareInsn {
 653:   bool operator() (const InsnMatchEntry &LHS, StringRef RHS) {
 654:     return LHS.Format < RHS;
 655:   }
 656:   bool operator() (StringRef LHS, const InsnMatchEntry &RHS) {
 657:     return LHS < RHS.Format;
 658:   }
 659:   bool operator() (const InsnMatchEntry &LHS, const InsnMatchEntry &RHS) {
 660:     return LHS.Format < RHS.Format;
 661:   }
 662: };
 663: 
 664: // Table initializing information for parsing the .insn directive.
 665: static struct InsnMatchEntry InsnMatchTable[] = {
 666:   /* Format, Opcode, NumOperands, OperandKinds */
 667:   { "e", SystemZ::InsnE, 1,
 668:     { MCK_U16Imm } },
 669:   { "ri", SystemZ::InsnRI, 3,
 670:     { MCK_U32Imm, MCK_AnyReg, MCK_S16Imm } },
 671:   { "rie", SystemZ::InsnRIE, 4,
 672:     { MCK_U48Imm, MCK_AnyReg, MCK_AnyReg, MCK_PCRel16 } },
 673:   { "ril", SystemZ::InsnRIL, 3,
 674:     { MCK_U48Imm, MCK_AnyReg, MCK_PCRel32 } },
 675:   { "rilu", SystemZ::InsnRILU, 3,
 676:     { MCK_U48Imm, MCK_AnyReg, MCK_U32Imm } },
 677:   { "ris", SystemZ::InsnRIS, 5,
 678:     { MCK_U48Imm, MCK_AnyReg, MCK_S8Imm, MCK_U4Imm, MCK_BDAddr64Disp12 } },
 679:   { "rr", SystemZ::InsnRR, 3,
 680:     { MCK_U16Imm, MCK_AnyReg, MCK_AnyReg } },
```
- **EN**: It introduces interface types such as `InsnMatchEntry`, `CompareInsn`, shaping how other backend components interact with this file. The range implements or declares functions including `operator`, `operator`, `operator`.
- **CN**: 它引入了 `InsnMatchEntry`, `CompareInsn` 等接口类型，定义了其他后端组件与本文件交互的方式。 这一段实现或声明了 `operator`, `operator`, `operator` 等函数。

### Lines 681-720 / 第 681-720 行
```cpp
 681:   { "rre", SystemZ::InsnRRE, 3,
 682:     { MCK_U32Imm, MCK_AnyReg, MCK_AnyReg } },
 683:   { "rrf", SystemZ::InsnRRF, 5,
 684:     { MCK_U32Imm, MCK_AnyReg, MCK_AnyReg, MCK_AnyReg, MCK_U4Imm } },
 685:   { "rrs", SystemZ::InsnRRS, 5,
 686:     { MCK_U48Imm, MCK_AnyReg, MCK_AnyReg, MCK_U4Imm, MCK_BDAddr64Disp12 } },
 687:   { "rs", SystemZ::InsnRS, 4,
 688:     { MCK_U32Imm, MCK_AnyReg, MCK_AnyReg, MCK_BDAddr64Disp12 } },
 689:   { "rse", SystemZ::InsnRSE, 4,
 690:     { MCK_U48Imm, MCK_AnyReg, MCK_AnyReg, MCK_BDAddr64Disp12 } },
 691:   { "rsi", SystemZ::InsnRSI, 4,
 692:     { MCK_U48Imm, MCK_AnyReg, MCK_AnyReg, MCK_PCRel16 } },
 693:   { "rsy", SystemZ::InsnRSY, 4,
 694:     { MCK_U48Imm, MCK_AnyReg, MCK_AnyReg, MCK_BDAddr64Disp20 } },
 695:   { "rx", SystemZ::InsnRX, 3,
 696:     { MCK_U32Imm, MCK_AnyReg, MCK_BDXAddr64Disp12 } },
 697:   { "rxe", SystemZ::InsnRXE, 3,
 698:     { MCK_U48Imm, MCK_AnyReg, MCK_BDXAddr64Disp12 } },
 699:   { "rxf", SystemZ::InsnRXF, 4,
 700:     { MCK_U48Imm, MCK_AnyReg, MCK_AnyReg, MCK_BDXAddr64Disp12 } },
 701:   { "rxy", SystemZ::InsnRXY, 3,
 702:     { MCK_U48Imm, MCK_AnyReg, MCK_BDXAddr64Disp20 } },
 703:   { "s", SystemZ::InsnS, 2,
 704:     { MCK_U32Imm, MCK_BDAddr64Disp12 } },
 705:   { "si", SystemZ::InsnSI, 3,
 706:     { MCK_U32Imm, MCK_BDAddr64Disp12, MCK_S8Imm } },
 707:   { "sil", SystemZ::InsnSIL, 3,
 708:     { MCK_U48Imm, MCK_BDAddr64Disp12, MCK_U16Imm } },
 709:   { "siy", SystemZ::InsnSIY, 3,
 710:     { MCK_U48Imm, MCK_BDAddr64Disp20, MCK_U8Imm } },
 711:   { "ss", SystemZ::InsnSS, 4,
 712:     { MCK_U48Imm, MCK_BDXAddr64Disp12, MCK_BDAddr64Disp12, MCK_AnyReg } },
 713:   { "sse", SystemZ::InsnSSE, 3,
 714:     { MCK_U48Imm, MCK_BDAddr64Disp12, MCK_BDAddr64Disp12 } },
 715:   { "ssf", SystemZ::InsnSSF, 4,
 716:     { MCK_U48Imm, MCK_BDAddr64Disp12, MCK_BDAddr64Disp12, MCK_AnyReg } },
 717:   { "vri", SystemZ::InsnVRI, 6,
 718:     { MCK_U48Imm, MCK_VR128, MCK_VR128, MCK_U12Imm, MCK_U4Imm, MCK_U4Imm } },
 719:   { "vrr", SystemZ::InsnVRR, 7,
 720:     { MCK_U48Imm, MCK_VR128, MCK_VR128, MCK_VR128, MCK_U4Imm, MCK_U4Imm,
```
- **EN**: This span continues the file's main responsibility: this file parses target assembly syntax into MC/LLVM instructions for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 721-760 / 第 721-760 行
```cpp
 721:       MCK_U4Imm } },
 722:   { "vrs", SystemZ::InsnVRS, 5,
 723:     { MCK_U48Imm, MCK_AnyReg, MCK_VR128, MCK_BDAddr64Disp12, MCK_U4Imm } },
 724:   { "vrv", SystemZ::InsnVRV, 4,
 725:     { MCK_U48Imm, MCK_VR128, MCK_BDVAddr64Disp12, MCK_U4Imm } },
 726:   { "vrx", SystemZ::InsnVRX, 4,
 727:     { MCK_U48Imm, MCK_VR128, MCK_BDXAddr64Disp12, MCK_U4Imm } },
 728:   { "vsi", SystemZ::InsnVSI, 4,
 729:     { MCK_U48Imm, MCK_VR128, MCK_BDAddr64Disp12, MCK_U8Imm } }
 730: };
 731: 
 732: void SystemZOperand::print(raw_ostream &OS, const MCAsmInfo &MAI) const {
 733:   switch (Kind) {
 734:   case KindToken:
 735:     OS << "Token:" << getToken();
 736:     break;
 737:   case KindReg:
 738:     OS << "Reg:" << SystemZGNUInstPrinter::getRegisterName(getReg());
 739:     break;
 740:   case KindImm:
 741:     OS << "Imm:";
 742:     MAI.printExpr(OS, *getImm());
 743:     break;
 744:   case KindImmTLS:
 745:     OS << "ImmTLS:";
 746:     MAI.printExpr(OS, *getImmTLS().Imm);
 747:     if (getImmTLS().Sym) {
 748:       OS << ", ";
 749:       MAI.printExpr(OS, *getImmTLS().Sym);
 750:     }
 751:     break;
 752:   case KindMem: {
 753:     const MemOp &Op = getMem();
 754:     OS << "Mem:";
 755:     MAI.printExpr(OS, *cast<MCConstantExpr>(Op.Disp));
 756:     if (Op.Base) {
 757:       OS << "(";
 758:       if (Op.MemKind == BDLMem) {
 759:         MAI.printExpr(OS, *cast<MCConstantExpr>(Op.Length.Imm));
 760:         OS << ',';
```
- **EN**: The range implements or declares functions including `SystemZOperand::print`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZOperand::print` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 761-800 / 第 761-800 行
```cpp
 761:       } else if (Op.MemKind == BDRMem)
 762:         OS << SystemZGNUInstPrinter::getRegisterName(Op.Length.Reg) << ",";
 763:       if (Op.Index)
 764:         OS << SystemZGNUInstPrinter::getRegisterName(Op.Index) << ",";
 765:       OS << SystemZGNUInstPrinter::getRegisterName(Op.Base);
 766:       OS << ")";
 767:     }
 768:     break;
 769:   }
 770:   case KindInvalid:
 771:     break;
 772:   }
 773: }
 774: 
 775: // Parse one register of the form %<prefix><number>.
 776: bool SystemZAsmParser::parseRegister(Register &Reg, bool RequirePercent,
 777:                                      bool RestoreOnFailure) {
 778:   const AsmToken &PercentTok = Parser.getTok();
 779:   bool HasPercent = PercentTok.is(AsmToken::Percent);
 780: 
 781:   Reg.StartLoc = PercentTok.getLoc();
 782: 
 783:   if (RequirePercent && PercentTok.isNot(AsmToken::Percent))
 784:     return Error(PercentTok.getLoc(), "register expected");
 785: 
 786:   if (HasPercent) {
 787:     Parser.Lex(); // Eat percent token.
 788:   }
 789: 
 790:   // Expect a register name.
 791:   if (Parser.getTok().isNot(AsmToken::Identifier)) {
 792:     if (RestoreOnFailure && HasPercent)
 793:       getLexer().UnLex(PercentTok);
 794:     return Error(Reg.StartLoc,
 795:                  HasPercent ? "invalid register" : "register expected");
 796:   }
 797: 
 798:   // Check that there's a prefix.
 799:   StringRef Name = Parser.getTok().getString();
 800:   if (Name.size() < 2) {
```
- **EN**: The range implements or declares functions including `SystemZAsmParser::parseRegister`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZAsmParser::parseRegister` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 801-840 / 第 801-840 行
```cpp
 801:     if (RestoreOnFailure && HasPercent)
 802:       getLexer().UnLex(PercentTok);
 803:     return Error(Reg.StartLoc, "invalid register");
 804:   }
 805:   char Prefix = Name[0];
 806: 
 807:   // Treat the rest of the register name as a register number.
 808:   if (Name.substr(1).getAsInteger(10, Reg.Num)) {
 809:     if (RestoreOnFailure && HasPercent)
 810:       getLexer().UnLex(PercentTok);
 811:     return Error(Reg.StartLoc, "invalid register");
 812:   }
 813: 
 814:   // Look for valid combinations of prefix and number.
 815:   if (Prefix == 'r' && Reg.Num < 16)
 816:     Reg.Group = RegGR;
 817:   else if (Prefix == 'f' && Reg.Num < 16)
 818:     Reg.Group = RegFP;
 819:   else if (Prefix == 'v' && Reg.Num < 32)
 820:     Reg.Group = RegV;
 821:   else if (Prefix == 'a' && Reg.Num < 16)
 822:     Reg.Group = RegAR;
 823:   else if (Prefix == 'c' && Reg.Num < 16)
 824:     Reg.Group = RegCR;
 825:   else {
 826:     if (RestoreOnFailure && HasPercent)
 827:       getLexer().UnLex(PercentTok);
 828:     return Error(Reg.StartLoc, "invalid register");
 829:   }
 830: 
 831:   Reg.EndLoc = Parser.getTok().getLoc();
 832:   Parser.Lex();
 833:   return false;
 834: }
 835: 
 836: // Parse a register of kind Kind and add it to Operands.
 837: ParseStatus SystemZAsmParser::parseRegister(OperandVector &Operands,
 838:                                             RegisterKind Kind) {
 839:   Register Reg;
 840:   RegisterGroup Group;
```
- **EN**: The range implements or declares functions including `SystemZAsmParser::parseRegister`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZAsmParser::parseRegister` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 841-880 / 第 841-880 行
```cpp
 841:   switch (Kind) {
 842:   case GR32Reg:
 843:   case GRH32Reg:
 844:   case GR64Reg:
 845:   case GR128Reg:
 846:     Group = RegGR;
 847:     break;
 848:   case FP16Reg:
 849:   case FP32Reg:
 850:   case FP64Reg:
 851:   case FP128Reg:
 852:     Group = RegFP;
 853:     break;
 854:   case VR16Reg:
 855:   case VR32Reg:
 856:   case VR64Reg:
 857:   case VR128Reg:
 858:     Group = RegV;
 859:     break;
 860:   case AR32Reg:
 861:     Group = RegAR;
 862:     break;
 863:   case CR64Reg:
 864:     Group = RegCR;
 865:     break;
 866:   }
 867: 
 868:   // Handle register names of the form %<prefix><number>
 869:   if (isParsingGNU() && Parser.getTok().is(AsmToken::Percent)) {
 870:     if (parseRegister(Reg, /*RequirePercent=*/true))
 871:       return ParseStatus::Failure;
 872: 
 873:     // Check the parsed register group "Reg.Group" with the expected "Group"
 874:     // Have to error out if user specified wrong prefix.
 875:     switch (Group) {
 876:     case RegGR:
 877:     case RegFP:
 878:     case RegAR:
 879:     case RegCR:
 880:       if (Group != Reg.Group)
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 881-920 / 第 881-920 行
```cpp
 881:         return Error(Reg.StartLoc, "invalid operand for instruction");
 882:       break;
 883:     case RegV:
 884:       if (Reg.Group != RegV && Reg.Group != RegFP)
 885:         return Error(Reg.StartLoc, "invalid operand for instruction");
 886:       break;
 887:     }
 888:   } else if (Parser.getTok().is(AsmToken::Integer)) {
 889:     if (parseIntegerRegister(Reg, Group))
 890:       return ParseStatus::Failure;
 891:   }
 892:   // Otherwise we didn't match a register operand.
 893:   else
 894:     return ParseStatus::NoMatch;
 895: 
 896:   // Determine the LLVM register number according to Kind.
 897:   // clang-format off
 898:   const unsigned *Regs;
 899:   switch (Kind) {
 900:   case GR32Reg:  Regs = SystemZMC::GR32Regs;  break;
 901:   case GRH32Reg: Regs = SystemZMC::GRH32Regs; break;
 902:   case GR64Reg:  Regs = SystemZMC::GR64Regs;  break;
 903:   case GR128Reg: Regs = SystemZMC::GR128Regs; break;
 904:   case FP16Reg:  Regs = SystemZMC::FP16Regs;  break;
 905:   case FP32Reg:  Regs = SystemZMC::FP32Regs;  break;
 906:   case FP64Reg:  Regs = SystemZMC::FP64Regs;  break;
 907:   case FP128Reg: Regs = SystemZMC::FP128Regs; break;
 908:   case VR16Reg:  Regs = SystemZMC::VR16Regs;  break;
 909:   case VR32Reg:  Regs = SystemZMC::VR32Regs;  break;
 910:   case VR64Reg:  Regs = SystemZMC::VR64Regs;  break;
 911:   case VR128Reg: Regs = SystemZMC::VR128Regs; break;
 912:   case AR32Reg:  Regs = SystemZMC::AR32Regs;  break;
 913:   case CR64Reg:  Regs = SystemZMC::CR64Regs;  break;
 914:   }
 915:   // clang-format on
 916:   if (Regs[Reg.Num] == 0)
 917:     return Error(Reg.StartLoc, "invalid register pair");
 918: 
 919:   Operands.push_back(
 920:       SystemZOperand::createReg(Kind, Regs[Reg.Num], Reg.StartLoc, Reg.EndLoc));
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 921-960 / 第 921-960 行
```cpp
 921:   return ParseStatus::Success;
 922: }
 923: 
 924: // Parse any type of register (including integers) and add it to Operands.
 925: ParseStatus SystemZAsmParser::parseAnyRegister(OperandVector &Operands) {
 926:   SMLoc StartLoc = Parser.getTok().getLoc();
 927: 
 928:   // Handle integer values.
 929:   if (Parser.getTok().is(AsmToken::Integer)) {
 930:     const MCExpr *Register;
 931:     if (Parser.parseExpression(Register))
 932:       return ParseStatus::Failure;
 933: 
 934:     if (auto *CE = dyn_cast<MCConstantExpr>(Register)) {
 935:       int64_t Value = CE->getValue();
 936:       if (Value < 0 || Value > 15)
 937:         return Error(StartLoc, "invalid register");
 938:     }
 939: 
 940:     SMLoc EndLoc =
 941:       SMLoc::getFromPointer(Parser.getTok().getLoc().getPointer() - 1);
 942: 
 943:     Operands.push_back(SystemZOperand::createImm(Register, StartLoc, EndLoc));
 944:   }
 945:   else {
 946:     if (isParsingHLASM())
 947:       return ParseStatus::NoMatch;
 948: 
 949:     Register Reg;
 950:     if (parseRegister(Reg, /*RequirePercent=*/true))
 951:       return ParseStatus::Failure;
 952: 
 953:     if (Reg.Num > 15)
 954:       return Error(StartLoc, "invalid register");
 955: 
 956:     // Map to the correct register kind.
 957:     RegisterKind Kind;
 958:     unsigned RegNo;
 959:     if (Reg.Group == RegGR) {
 960:       Kind = GR64Reg;
```
- **EN**: The range implements or declares functions including `SystemZAsmParser::parseAnyRegister`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZAsmParser::parseAnyRegister` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 961-1000 / 第 961-1000 行
```cpp
 961:       RegNo = SystemZMC::GR64Regs[Reg.Num];
 962:     }
 963:     else if (Reg.Group == RegFP) {
 964:       Kind = FP64Reg;
 965:       RegNo = SystemZMC::FP64Regs[Reg.Num];
 966:     }
 967:     else if (Reg.Group == RegV) {
 968:       Kind = VR128Reg;
 969:       RegNo = SystemZMC::VR128Regs[Reg.Num];
 970:     }
 971:     else if (Reg.Group == RegAR) {
 972:       Kind = AR32Reg;
 973:       RegNo = SystemZMC::AR32Regs[Reg.Num];
 974:     }
 975:     else if (Reg.Group == RegCR) {
 976:       Kind = CR64Reg;
 977:       RegNo = SystemZMC::CR64Regs[Reg.Num];
 978:     }
 979:     else {
 980:       return ParseStatus::Failure;
 981:     }
 982: 
 983:     Operands.push_back(SystemZOperand::createReg(Kind, RegNo,
 984:                                                  Reg.StartLoc, Reg.EndLoc));
 985:   }
 986:   return ParseStatus::Success;
 987: }
 988: 
 989: bool SystemZAsmParser::parseIntegerRegister(Register &Reg,
 990:                                             RegisterGroup Group) {
 991:   Reg.StartLoc = Parser.getTok().getLoc();
 992:   // We have an integer token
 993:   const MCExpr *Register;
 994:   if (Parser.parseExpression(Register))
 995:     return true;
 996: 
 997:   const auto *CE = dyn_cast<MCConstantExpr>(Register);
 998:   if (!CE)
 999:     return true;
1000: 
```
- **EN**: The range implements or declares functions including `SystemZAsmParser::parseIntegerRegister`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZAsmParser::parseIntegerRegister` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1001-1040 / 第 1001-1040 行
```cpp
1001:   int64_t MaxRegNum = (Group == RegV) ? 31 : 15;
1002:   int64_t Value = CE->getValue();
1003:   if (Value < 0 || Value > MaxRegNum) {
1004:     Error(Parser.getTok().getLoc(), "invalid register");
1005:     return true;
1006:   }
1007: 
1008:   // Assign the Register Number
1009:   Reg.Num = (unsigned)Value;
1010:   Reg.Group = Group;
1011:   Reg.EndLoc = SMLoc::getFromPointer(Parser.getTok().getLoc().getPointer() - 1);
1012: 
1013:   // At this point, successfully parsed an integer register.
1014:   return false;
1015: }
1016: 
1017: // Parse a memory operand into Reg1, Reg2, Disp, and Length.
1018: bool SystemZAsmParser::parseAddress(bool &HaveReg1, Register &Reg1,
1019:                                     bool &HaveReg2, Register &Reg2,
1020:                                     const MCExpr *&Disp, const MCExpr *&Length,
1021:                                     bool HasLength, bool HasVectorIndex) {
1022:   // Parse the displacement, which must always be present.
1023:   if (getParser().parseExpression(Disp))
1024:     return true;
1025: 
1026:   // Parse the optional base and index.
1027:   HaveReg1 = false;
1028:   HaveReg2 = false;
1029:   Length = nullptr;
1030: 
1031:   // If we have a scenario as below:
1032:   //   vgef %v0, 0(0), 0
1033:   // This is an example of a "BDVMem" instruction type.
1034:   //
1035:   // So when we parse this as an integer register, the register group
1036:   // needs to be tied to "RegV". Usually when the prefix is passed in
1037:   // as %<prefix><reg-number> its easy to check which group it should belong to
1038:   // However, if we're passing in just the integer there's no real way to
1039:   // "check" what register group it should belong to.
1040:   //
```
- **EN**: The range implements or declares functions including `SystemZAsmParser::parseAddress`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZAsmParser::parseAddress` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1041-1080 / 第 1041-1080 行
```cpp
1041:   // When the user passes in the register as an integer, the user assumes that
1042:   // the compiler is responsible for substituting it as the right kind of
1043:   // register. Whereas, when the user specifies a "prefix", the onus is on
1044:   // the user to make sure they pass in the right kind of register.
1045:   //
1046:   // The restriction only applies to the first Register (i.e. Reg1). Reg2 is
1047:   // always a general register. Reg1 should be of group RegV if "HasVectorIndex"
1048:   // (i.e. insn is of type BDVMem) is true.
1049:   RegisterGroup RegGroup = HasVectorIndex ? RegV : RegGR;
1050: 
1051:   if (getLexer().is(AsmToken::LParen)) {
1052:     Parser.Lex();
1053: 
1054:     if (isParsingGNU() && getLexer().is(AsmToken::Percent)) {
1055:       // Parse the first register.
1056:       HaveReg1 = true;
1057:       if (parseRegister(Reg1, /*RequirePercent=*/true))
1058:         return true;
1059:     }
1060:     // So if we have an integer as the first token in ([tok1], ..), it could:
1061:     // 1. Refer to a "Register" (i.e X,R,V fields in BD[X|R|V]Mem type of
1062:     // instructions)
1063:     // 2. Refer to a "Length" field (i.e L field in BDLMem type of instructions)
1064:     else if (getLexer().is(AsmToken::Integer)) {
1065:       if (HasLength) {
1066:         // Instruction has a "Length" field, safe to parse the first token as
1067:         // the "Length" field
1068:         if (getParser().parseExpression(Length))
1069:           return true;
1070:       } else {
1071:         // Otherwise, if the instruction has no "Length" field, parse the
1072:         // token as a "Register". We don't have to worry about whether the
1073:         // instruction is invalid here, because the caller will take care of
1074:         // error reporting.
1075:         HaveReg1 = true;
1076:         if (parseIntegerRegister(Reg1, RegGroup))
1077:           return true;
1078:       }
1079:     } else {
1080:       // If its not an integer or a percent token, then if the instruction
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1081-1120 / 第 1081-1120 行
```cpp
1081:       // is reported to have a "Length" then, parse it as "Length".
1082:       if (HasLength) {
1083:         if (getParser().parseExpression(Length))
1084:           return true;
1085:       }
1086:     }
1087: 
1088:     // Check whether there's a second register.
1089:     if (getLexer().is(AsmToken::Comma)) {
1090:       Parser.Lex();
1091:       HaveReg2 = true;
1092: 
1093:       if (getLexer().is(AsmToken::Integer)) {
1094:         if (parseIntegerRegister(Reg2, RegGR))
1095:           return true;
1096:       } else if (isParsingGNU()) {
1097:         if (Parser.getTok().is(AsmToken::Percent)) {
1098:           if (parseRegister(Reg2, /*RequirePercent=*/true))
1099:             return true;
1100:         } else {
1101:           // GAS allows ",)" to indicate a missing base register.
1102:           Reg2.Num = 0;
1103:           Reg2.Group = RegGR;
1104:           Reg2.StartLoc = Reg2.EndLoc = Parser.getTok().getLoc();
1105:         }
1106:       }
1107:     }
1108: 
1109:     // Consume the closing bracket.
1110:     if (getLexer().isNot(AsmToken::RParen))
1111:       return Error(Parser.getTok().getLoc(), "unexpected token in address");
1112:     Parser.Lex();
1113:   }
1114:   return false;
1115: }
1116: 
1117: // Verify that Reg is a valid address register (base or index).
1118: bool
1119: SystemZAsmParser::parseAddressRegister(Register &Reg) {
1120:   if (Reg.Group == RegV) {
```
- **EN**: The range implements or declares functions including `SystemZAsmParser::parseAddressRegister`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZAsmParser::parseAddressRegister` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1121-1160 / 第 1121-1160 行
```cpp
1121:     Error(Reg.StartLoc, "invalid use of vector addressing");
1122:     return true;
1123:   }
1124:   if (Reg.Group != RegGR) {
1125:     Error(Reg.StartLoc, "invalid address register");
1126:     return true;
1127:   }
1128:   return false;
1129: }
1130: 
1131: // Parse a memory operand and add it to Operands.  The other arguments
1132: // are as above.
1133: ParseStatus SystemZAsmParser::parseAddress(OperandVector &Operands,
1134:                                            MemoryKind MemKind,
1135:                                            RegisterKind RegKind) {
1136:   SMLoc StartLoc = Parser.getTok().getLoc();
1137:   unsigned Base = 0, Index = 0, LengthReg = 0;
1138:   Register Reg1, Reg2;
1139:   bool HaveReg1, HaveReg2;
1140:   const MCExpr *Disp;
1141:   const MCExpr *Length;
1142: 
1143:   bool HasLength = (MemKind == BDLMem) ? true : false;
1144:   bool HasVectorIndex = (MemKind == BDVMem) ? true : false;
1145:   if (parseAddress(HaveReg1, Reg1, HaveReg2, Reg2, Disp, Length, HasLength,
1146:                    HasVectorIndex))
1147:     return ParseStatus::Failure;
1148: 
1149:   const unsigned *Regs;
1150:   switch (RegKind) {
1151:   case GR32Reg: Regs = SystemZMC::GR32Regs; break;
1152:   case GR64Reg: Regs = SystemZMC::GR64Regs; break;
1153:   default: llvm_unreachable("invalid RegKind");
1154:   }
1155: 
1156:   switch (MemKind) {
1157:   case BDMem:
1158:     // If we have Reg1, it must be an address register.
1159:     if (HaveReg1) {
1160:       if (parseAddressRegister(Reg1))
```
- **EN**: The range implements or declares functions including `SystemZAsmParser::parseAddress`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZAsmParser::parseAddress` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1161-1200 / 第 1161-1200 行
```cpp
1161:         return ParseStatus::Failure;
1162:       Base = Reg1.Num == 0 ? 0 : Regs[Reg1.Num];
1163:     }
1164:     // There must be no Reg2.
1165:     if (HaveReg2)
1166:       return Error(StartLoc, "invalid use of indexed addressing");
1167:     break;
1168:   case BDXMem:
1169:   case LXAMem:
1170:     // If we have Reg1, it must be an address register.
1171:     if (HaveReg1) {
1172:       const unsigned *IndexRegs = Regs;
1173:       if (MemKind == LXAMem)
1174:         IndexRegs = SystemZMC::GR32Regs;
1175: 
1176:       if (parseAddressRegister(Reg1))
1177:         return ParseStatus::Failure;
1178:       // If there are two registers, the first one is the index and the
1179:       // second is the base.  If there is only a single register, it is
1180:       // used as base with GAS and as index with HLASM.
1181:       if (HaveReg2 || isParsingHLASM())
1182:         Index = Reg1.Num == 0 ? 0 : IndexRegs[Reg1.Num];
1183:       else
1184:         Base = Reg1.Num == 0 ? 0 : Regs[Reg1.Num];
1185:     }
1186:     // If we have Reg2, it must be an address register.
1187:     if (HaveReg2) {
1188:       if (parseAddressRegister(Reg2))
1189:         return ParseStatus::Failure;
1190:       Base = Reg2.Num == 0 ? 0 : Regs[Reg2.Num];
1191:     }
1192:     break;
1193:   case BDLMem:
1194:     // If we have Reg2, it must be an address register.
1195:     if (HaveReg2) {
1196:       if (parseAddressRegister(Reg2))
1197:         return ParseStatus::Failure;
1198:       Base = Reg2.Num == 0 ? 0 : Regs[Reg2.Num];
1199:     }
1200:     // We cannot support base+index addressing.
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1201-1240 / 第 1201-1240 行
```cpp
1201:     if (HaveReg1 && HaveReg2)
1202:       return Error(StartLoc, "invalid use of indexed addressing");
1203:     // We must have a length.
1204:     if (!Length)
1205:       return Error(StartLoc, "missing length in address");
1206:     break;
1207:   case BDRMem:
1208:     // We must have Reg1, and it must be a GPR.
1209:     if (!HaveReg1 || Reg1.Group != RegGR)
1210:       return Error(StartLoc, "invalid operand for instruction");
1211:     LengthReg = SystemZMC::GR64Regs[Reg1.Num];
1212:     // If we have Reg2, it must be an address register.
1213:     if (HaveReg2) {
1214:       if (parseAddressRegister(Reg2))
1215:         return ParseStatus::Failure;
1216:       Base = Reg2.Num == 0 ? 0 : Regs[Reg2.Num];
1217:     }
1218:     break;
1219:   case BDVMem:
1220:     // We must have Reg1, and it must be a vector register.
1221:     if (!HaveReg1 || Reg1.Group != RegV)
1222:       return Error(StartLoc, "vector index required in address");
1223:     Index = SystemZMC::VR128Regs[Reg1.Num];
1224:     // In GAS mode, we must have Reg2, since a single register would be
1225:     // interpreted as base register, which cannot be a vector register.
1226:     if (isParsingGNU() && !HaveReg2)
1227:       return Error(Reg1.StartLoc, "invalid use of vector addressing");
1228:     // If we have Reg2, it must be an address register.
1229:     if (HaveReg2) {
1230:       if (parseAddressRegister(Reg2))
1231:         return ParseStatus::Failure;
1232:       Base = Reg2.Num == 0 ? 0 : Regs[Reg2.Num];
1233:     }
1234:     break;
1235:   }
1236: 
1237:   SMLoc EndLoc =
1238:       SMLoc::getFromPointer(Parser.getTok().getLoc().getPointer() - 1);
1239:   Operands.push_back(SystemZOperand::createMem(MemKind, RegKind, Base, Disp,
1240:                                                Index, Length, LengthReg,
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1241-1280 / 第 1241-1280 行
```cpp
1241:                                                StartLoc, EndLoc));
1242:   return ParseStatus::Success;
1243: }
1244: 
1245: ParseStatus SystemZAsmParser::parseDirective(AsmToken DirectiveID) {
1246:   StringRef IDVal = DirectiveID.getIdentifier();
1247: 
1248:   if (IDVal == ".insn")
1249:     return parseDirectiveInsn(DirectiveID.getLoc());
1250:   if (IDVal == ".machine")
1251:     return parseDirectiveMachine(DirectiveID.getLoc());
1252:   if (IDVal.starts_with(".gnu_attribute"))
1253:     return parseGNUAttribute(DirectiveID.getLoc());
1254: 
1255:   return ParseStatus::NoMatch;
1256: }
1257: 
1258: /// ParseDirectiveInsn
1259: /// ::= .insn [ format, encoding, (operands (, operands)*) ]
1260: bool SystemZAsmParser::parseDirectiveInsn(SMLoc L) {
1261:   MCAsmParser &Parser = getParser();
1262: 
1263:   // Expect instruction format as identifier.
1264:   StringRef Format;
1265:   SMLoc ErrorLoc = Parser.getTok().getLoc();
1266:   if (Parser.parseIdentifier(Format))
1267:     return Error(ErrorLoc, "expected instruction format");
1268: 
1269:   SmallVector<std::unique_ptr<MCParsedAsmOperand>, 8> Operands;
1270: 
1271:   // Find entry for this format in InsnMatchTable.
1272:   auto EntryRange =
1273:     std::equal_range(std::begin(InsnMatchTable), std::end(InsnMatchTable),
1274:                      Format, CompareInsn());
1275: 
1276:   // If first == second, couldn't find a match in the table.
1277:   if (EntryRange.first == EntryRange.second)
1278:     return Error(ErrorLoc, "unrecognized format");
1279: 
1280:   struct InsnMatchEntry *Entry = EntryRange.first;
```
- **EN**: It introduces interface types such as `InsnMatchEntry`, shaping how other backend components interact with this file. The range implements or declares functions including `SystemZAsmParser::parseDirective`, `SystemZAsmParser::parseDirectiveInsn`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 它引入了 `InsnMatchEntry` 等接口类型，定义了其他后端组件与本文件交互的方式。 这一段实现或声明了 `SystemZAsmParser::parseDirective`, `SystemZAsmParser::parseDirectiveInsn` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1281-1320 / 第 1281-1320 行
```cpp
1281: 
1282:   // Format should match from equal_range.
1283:   assert(Entry->Format == Format);
1284: 
1285:   // Parse the following operands using the table's information.
1286:   for (int I = 0; I < Entry->NumOperands; I++) {
1287:     MatchClassKind Kind = Entry->OperandKinds[I];
1288: 
1289:     SMLoc StartLoc = Parser.getTok().getLoc();
1290: 
1291:     // Always expect commas as separators for operands.
1292:     if (getLexer().isNot(AsmToken::Comma))
1293:       return Error(StartLoc, "unexpected token in directive");
1294:     Lex();
1295: 
1296:     // Parse operands.
1297:     ParseStatus ResTy;
1298:     if (Kind == MCK_AnyReg)
1299:       ResTy = parseAnyReg(Operands);
1300:     else if (Kind == MCK_VR128)
1301:       ResTy = parseVR128(Operands);
1302:     else if (Kind == MCK_BDXAddr64Disp12 || Kind == MCK_BDXAddr64Disp20)
1303:       ResTy = parseBDXAddr64(Operands);
1304:     else if (Kind == MCK_BDAddr64Disp12 || Kind == MCK_BDAddr64Disp20)
1305:       ResTy = parseBDAddr64(Operands);
1306:     else if (Kind == MCK_BDVAddr64Disp12)
1307:       ResTy = parseBDVAddr64(Operands);
1308:     else if (Kind == MCK_LXAAddr64Disp20)
1309:       ResTy = parseLXAAddr64(Operands);
1310:     else if (Kind == MCK_PCRel32)
1311:       ResTy = parsePCRel32(Operands);
1312:     else if (Kind == MCK_PCRel16)
1313:       ResTy = parsePCRel16(Operands);
1314:     else {
1315:       // Only remaining operand kind is an immediate.
1316:       const MCExpr *Expr;
1317:       SMLoc StartLoc = Parser.getTok().getLoc();
1318: 
1319:       // Expect immediate expression.
1320:       if (Parser.parseExpression(Expr))
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 1321-1360 / 第 1321-1360 行
```cpp
1321:         return Error(StartLoc, "unexpected token in directive");
1322: 
1323:       SMLoc EndLoc =
1324:         SMLoc::getFromPointer(Parser.getTok().getLoc().getPointer() - 1);
1325: 
1326:       Operands.push_back(SystemZOperand::createImm(Expr, StartLoc, EndLoc));
1327:       ResTy = ParseStatus::Success;
1328:     }
1329: 
1330:     if (!ResTy.isSuccess())
1331:       return true;
1332:   }
1333: 
1334:   // Build the instruction with the parsed operands.
1335:   MCInst Inst = MCInstBuilder(Entry->Opcode);
1336: 
1337:   for (size_t I = 0; I < Operands.size(); I++) {
1338:     MCParsedAsmOperand &Operand = *Operands[I];
1339:     MatchClassKind Kind = Entry->OperandKinds[I];
1340: 
1341:     // Verify operand.
1342:     unsigned Res = validateOperandClass(Operand, Kind, *STI);
1343:     if (Res != Match_Success)
1344:       return Error(Operand.getStartLoc(), "unexpected operand type");
1345: 
1346:     // Add operands to instruction.
1347:     SystemZOperand &ZOperand = static_cast<SystemZOperand &>(Operand);
1348:     if (ZOperand.isReg())
1349:       ZOperand.addRegOperands(Inst, 1);
1350:     else if (ZOperand.isMem(BDMem))
1351:       ZOperand.addBDAddrOperands(Inst, 2);
1352:     else if (ZOperand.isMem(BDXMem))
1353:       ZOperand.addBDXAddrOperands(Inst, 3);
1354:     else if (ZOperand.isMem(BDVMem))
1355:       ZOperand.addBDVAddrOperands(Inst, 3);
1356:     else if (ZOperand.isMem(LXAMem))
1357:       ZOperand.addLXAAddrOperands(Inst, 3);
1358:     else if (ZOperand.isImm())
1359:       ZOperand.addImmOperands(Inst, 1);
1360:     else
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1361-1400 / 第 1361-1400 行
```cpp
1361:       llvm_unreachable("unexpected operand type");
1362:   }
1363: 
1364:   // Emit as a regular instruction.
1365:   Parser.getStreamer().emitInstruction(Inst, getSTI());
1366: 
1367:   return false;
1368: }
1369: 
1370: /// ParseDirectiveMachine
1371: /// ::= .machine [ mcpu ]
1372: bool SystemZAsmParser::parseDirectiveMachine(SMLoc L) {
1373:   MCAsmParser &Parser = getParser();
1374:   if (Parser.getTok().isNot(AsmToken::Identifier) &&
1375:       Parser.getTok().isNot(AsmToken::String))
1376:     return TokError("unexpected token in '.machine' directive");
1377: 
1378:   StringRef Id = Parser.getTok().getIdentifier();
1379:   SMLoc IdLoc = Parser.getTok().getLoc();
1380: 
1381:   Parser.Lex();
1382:   if (parseEOL())
1383:     return true;
1384: 
1385:   // Parse push and pop directives first
1386:   if (Id == "push") {
1387:     // Push the Current FeatureBitSet onto the stack.
1388:     MachineStack.push_back(getAvailableFeatures());
1389:   } else if (Id == "pop") {
1390:     // If the stack is not empty pop the topmost FeatureBitset and use it.
1391:     if (MachineStack.empty())
1392:       return Error(IdLoc,
1393:                    "pop without corresponding push in '.machine' directive");
1394:     setAvailableFeatures(MachineStack.back());
1395:     MachineStack.pop_back();
1396:   } else {
1397:     // Try to interpret the Identifier as a CPU spec and derive the
1398:     // FeatureBitset from that.
1399:     MCSubtargetInfo &STI = copySTI();
1400:     STI.setDefaultFeatures(Id, /*TuneCPU*/ Id, "");
```
- **EN**: The range implements or declares functions including `SystemZAsmParser::parseDirectiveMachine`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `SystemZAsmParser::parseDirectiveMachine` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 1401-1440 / 第 1401-1440 行
```cpp
1401:     setAvailableFeatures(ComputeAvailableFeatures(STI.getFeatureBits()));
1402:   }
1403:   getTargetStreamer().emitMachine(Id);
1404: 
1405:   return false;
1406: }
1407: 
1408: bool SystemZAsmParser::parseGNUAttribute(SMLoc L) {
1409:   int64_t Tag;
1410:   int64_t IntegerValue;
1411:   if (!Parser.parseGNUAttribute(L, Tag, IntegerValue))
1412:     return Error(L, "malformed .gnu_attribute directive");
1413: 
1414:   // Tag_GNU_S390_ABI_Vector tag is '8' and can be 0, 1, or 2.
1415:   if (Tag != 8 || (IntegerValue < 0 || IntegerValue > 2))
1416:     return Error(L, "unrecognized .gnu_attribute tag/value pair.");
1417: 
1418:   Parser.getStreamer().emitGNUAttribute(Tag, IntegerValue);
1419: 
1420:   return parseEOL();
1421: }
1422: 
1423: bool SystemZAsmParser::ParseRegister(MCRegister &RegNo, SMLoc &StartLoc,
1424:                                      SMLoc &EndLoc, bool RequirePercent,
1425:                                      bool RestoreOnFailure) {
1426:   Register Reg;
1427:   if (parseRegister(Reg, RequirePercent, RestoreOnFailure))
1428:     return true;
1429:   if (Reg.Group == RegGR)
1430:     RegNo = SystemZMC::GR64Regs[Reg.Num];
1431:   else if (Reg.Group == RegFP)
1432:     RegNo = SystemZMC::FP64Regs[Reg.Num];
1433:   else if (Reg.Group == RegV)
1434:     RegNo = SystemZMC::VR128Regs[Reg.Num];
1435:   else if (Reg.Group == RegAR)
1436:     RegNo = SystemZMC::AR32Regs[Reg.Num];
1437:   else if (Reg.Group == RegCR)
1438:     RegNo = SystemZMC::CR64Regs[Reg.Num];
1439:   StartLoc = Reg.StartLoc;
1440:   EndLoc = Reg.EndLoc;
```
- **EN**: The range implements or declares functions including `SystemZAsmParser::parseGNUAttribute`, `SystemZAsmParser::ParseRegister`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZAsmParser::parseGNUAttribute`, `SystemZAsmParser::ParseRegister` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1441-1480 / 第 1441-1480 行
```cpp
1441:   return false;
1442: }
1443: 
1444: bool SystemZAsmParser::parseRegister(MCRegister &Reg, SMLoc &StartLoc,
1445:                                      SMLoc &EndLoc) {
1446:   return ParseRegister(Reg, StartLoc, EndLoc, /*RequirePercent=*/false,
1447:                        /*RestoreOnFailure=*/false);
1448: }
1449: 
1450: ParseStatus SystemZAsmParser::tryParseRegister(MCRegister &Reg, SMLoc &StartLoc,
1451:                                                SMLoc &EndLoc) {
1452:   bool Result = ParseRegister(Reg, StartLoc, EndLoc, /*RequirePercent=*/false,
1453:                               /*RestoreOnFailure=*/true);
1454:   bool PendingErrors = getParser().hasPendingError();
1455:   getParser().clearPendingErrors();
1456:   if (PendingErrors)
1457:     return ParseStatus::Failure;
1458:   if (Result)
1459:     return ParseStatus::NoMatch;
1460:   return ParseStatus::Success;
1461: }
1462: 
1463: bool SystemZAsmParser::parseInstruction(ParseInstructionInfo &Info,
1464:                                         StringRef Name, SMLoc NameLoc,
1465:                                         OperandVector &Operands) {
1466: 
1467:   // Apply mnemonic aliases first, before doing anything else, in
1468:   // case the target uses it.
1469:   applyMnemonicAliases(Name, getAvailableFeatures(), getMAIAssemblerDialect());
1470: 
1471:   Operands.push_back(SystemZOperand::createToken(Name, NameLoc));
1472: 
1473:   // Read the remaining operands.
1474:   if (getLexer().isNot(AsmToken::EndOfStatement)) {
1475:     // Read the first operand.
1476:     if (parseOperand(Operands, Name)) {
1477:       return true;
1478:     }
1479: 
1480:     // Read any subsequent operands.
```
- **EN**: The range implements or declares functions including `SystemZAsmParser::parseRegister`, `SystemZAsmParser::tryParseRegister`, `SystemZAsmParser::parseInstruction`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZAsmParser::parseRegister`, `SystemZAsmParser::tryParseRegister`, `SystemZAsmParser::parseInstruction` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1481-1520 / 第 1481-1520 行
```cpp
1481:     while (getLexer().is(AsmToken::Comma)) {
1482:       Parser.Lex();
1483: 
1484:       if (isParsingHLASM() && getLexer().is(AsmToken::Space))
1485:         return Error(
1486:             Parser.getTok().getLoc(),
1487:             "No space allowed between comma that separates operand entries");
1488: 
1489:       if (parseOperand(Operands, Name)) {
1490:         return true;
1491:       }
1492:     }
1493: 
1494:     // Under the HLASM variant, we could have the remark field
1495:     // The remark field occurs after the operation entries
1496:     // There is a space that separates the operation entries and the
1497:     // remark field.
1498:     if (isParsingHLASM() && getTok().is(AsmToken::Space)) {
1499:       // We've confirmed that there is a Remark field.
1500:       StringRef Remark(getLexer().LexUntilEndOfStatement());
1501:       Parser.Lex();
1502: 
1503:       // If there is nothing after the space, then there is nothing to emit
1504:       // We could have a situation as this:
1505:       // "  \n"
1506:       // After lexing above, we will have
1507:       // "\n"
1508:       // This isn't an explicit remark field, so we don't have to output
1509:       // this as a comment.
1510:       if (Remark.size())
1511:         // Output the entire Remarks Field as a comment
1512:         getStreamer().AddComment(Remark);
1513:     }
1514: 
1515:     if (getLexer().isNot(AsmToken::EndOfStatement)) {
1516:       SMLoc Loc = getLexer().getLoc();
1517:       return Error(Loc, "unexpected token in argument list");
1518:     }
1519:   }
1520: 
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1521-1560 / 第 1521-1560 行
```cpp
1521:   // Consume the EndOfStatement.
1522:   Parser.Lex();
1523:   return false;
1524: }
1525: 
1526: bool SystemZAsmParser::parseOperand(OperandVector &Operands,
1527:                                     StringRef Mnemonic) {
1528:   // Check if the current operand has a custom associated parser, if so, try to
1529:   // custom parse the operand, or fallback to the general approach.  Force all
1530:   // features to be available during the operand check, or else we will fail to
1531:   // find the custom parser, and then we will later get an InvalidOperand error
1532:   // instead of a MissingFeature errror.
1533:   FeatureBitset AvailableFeatures = getAvailableFeatures();
1534:   FeatureBitset All;
1535:   All.set();
1536:   setAvailableFeatures(All);
1537:   ParseStatus Res = MatchOperandParserImpl(Operands, Mnemonic);
1538:   setAvailableFeatures(AvailableFeatures);
1539:   if (Res.isSuccess())
1540:     return false;
1541: 
1542:   // If there wasn't a custom match, try the generic matcher below. Otherwise,
1543:   // there was a match, but an error occurred, in which case, just return that
1544:   // the operand parsing failed.
1545:   if (Res.isFailure())
1546:     return true;
1547: 
1548:   // Check for a register.  All real register operands should have used
1549:   // a context-dependent parse routine, which gives the required register
1550:   // class.  The code is here to mop up other cases, like those where
1551:   // the instruction isn't recognized.
1552:   if (isParsingGNU() && Parser.getTok().is(AsmToken::Percent)) {
1553:     Register Reg;
1554:     if (parseRegister(Reg, /*RequirePercent=*/true))
1555:       return true;
1556:     Operands.push_back(SystemZOperand::createInvalid(Reg.StartLoc, Reg.EndLoc));
1557:     return false;
1558:   }
1559: 
1560:   // The only other type of operand is an immediate or address.  As above,
```
- **EN**: The range implements or declares functions including `SystemZAsmParser::parseOperand`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZAsmParser::parseOperand` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1561-1600 / 第 1561-1600 行
```cpp
1561:   // real address operands should have used a context-dependent parse routine,
1562:   // so we treat any plain expression as an immediate.
1563:   SMLoc StartLoc = Parser.getTok().getLoc();
1564:   Register Reg1, Reg2;
1565:   bool HaveReg1, HaveReg2;
1566:   const MCExpr *Expr;
1567:   const MCExpr *Length;
1568:   if (parseAddress(HaveReg1, Reg1, HaveReg2, Reg2, Expr, Length,
1569:                    /*HasLength*/ true, /*HasVectorIndex*/ true))
1570:     return true;
1571:   // If the register combination is not valid for any instruction, reject it.
1572:   // Otherwise, fall back to reporting an unrecognized instruction.
1573:   if (HaveReg1 && Reg1.Group != RegGR && Reg1.Group != RegV
1574:       && parseAddressRegister(Reg1))
1575:     return true;
1576:   if (HaveReg2 && parseAddressRegister(Reg2))
1577:     return true;
1578: 
1579:   SMLoc EndLoc =
1580:     SMLoc::getFromPointer(Parser.getTok().getLoc().getPointer() - 1);
1581:   if (HaveReg1 || HaveReg2 || Length)
1582:     Operands.push_back(SystemZOperand::createInvalid(StartLoc, EndLoc));
1583:   else
1584:     Operands.push_back(SystemZOperand::createImm(Expr, StartLoc, EndLoc));
1585:   return false;
1586: }
1587: 
1588: bool SystemZAsmParser::matchAndEmitInstruction(SMLoc IDLoc, unsigned &Opcode,
1589:                                                OperandVector &Operands,
1590:                                                MCStreamer &Out,
1591:                                                uint64_t &ErrorInfo,
1592:                                                bool MatchingInlineAsm) {
1593:   MCInst Inst;
1594:   unsigned MatchResult;
1595: 
1596:   unsigned Dialect = getMAIAssemblerDialect();
1597: 
1598:   FeatureBitset MissingFeatures;
1599:   MatchResult = MatchInstructionImpl(Operands, Inst, ErrorInfo, MissingFeatures,
1600:                                      MatchingInlineAsm, Dialect);
```
- **EN**: The range implements or declares functions including `SystemZAsmParser::matchAndEmitInstruction`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZAsmParser::matchAndEmitInstruction` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1601-1640 / 第 1601-1640 行
```cpp
1601:   switch (MatchResult) {
1602:   case Match_Success:
1603:     Inst.setLoc(IDLoc);
1604:     Out.emitInstruction(Inst, getSTI());
1605:     return false;
1606: 
1607:   case Match_MissingFeature: {
1608:     assert(MissingFeatures.any() && "Unknown missing feature!");
1609:     // Special case the error message for the very common case where only
1610:     // a single subtarget feature is missing
1611:     std::string Msg = "instruction requires:";
1612:     for (unsigned I = 0, E = MissingFeatures.size(); I != E; ++I) {
1613:       if (MissingFeatures[I]) {
1614:         Msg += " ";
1615:         Msg += getSubtargetFeatureName(I);
1616:       }
1617:     }
1618:     return Error(IDLoc, Msg);
1619:   }
1620: 
1621:   case Match_InvalidOperand: {
1622:     SMLoc ErrorLoc = IDLoc;
1623:     if (ErrorInfo != ~0ULL) {
1624:       if (ErrorInfo >= Operands.size())
1625:         return Error(IDLoc, "too few operands for instruction");
1626: 
1627:       ErrorLoc = ((SystemZOperand &)*Operands[ErrorInfo]).getStartLoc();
1628:       if (ErrorLoc == SMLoc())
1629:         ErrorLoc = IDLoc;
1630:     }
1631:     return Error(ErrorLoc, "invalid operand for instruction");
1632:   }
1633: 
1634:   case Match_MnemonicFail: {
1635:     FeatureBitset FBS = ComputeAvailableFeatures(getSTI().getFeatureBits());
1636:     std::string Suggestion = SystemZMnemonicSpellCheck(
1637:         ((SystemZOperand &)*Operands[0]).getToken(), FBS, Dialect);
1638:     return Error(IDLoc, "invalid instruction" + Suggestion,
1639:                  ((SystemZOperand &)*Operands[0]).getLocRange());
1640:   }
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1641-1680 / 第 1641-1680 行
```cpp
1641:   }
1642: 
1643:   llvm_unreachable("Unexpected match type");
1644: }
1645: 
1646: ParseStatus SystemZAsmParser::parsePCRel(OperandVector &Operands,
1647:                                          int64_t MinVal, int64_t MaxVal,
1648:                                          bool AllowTLS) {
1649:   MCContext &Ctx = getContext();
1650:   MCStreamer &Out = getStreamer();
1651:   const MCExpr *Expr;
1652:   SMLoc StartLoc = Parser.getTok().getLoc();
1653:   if (getParser().parseExpression(Expr))
1654:     return ParseStatus::NoMatch;
1655: 
1656:   auto IsOutOfRangeConstant = [&](const MCExpr *E, bool Negate) -> bool {
1657:     if (auto *CE = dyn_cast<MCConstantExpr>(E)) {
1658:       int64_t Value = CE->getValue();
1659:       if (Negate)
1660:         Value = -Value;
1661:       if ((Value & 1) || Value < MinVal || Value > MaxVal)
1662:         return true;
1663:     }
1664:     return false;
1665:   };
1666: 
1667:   // For consistency with the GNU assembler, treat immediates as offsets
1668:   // from ".".
1669:   if (auto *CE = dyn_cast<MCConstantExpr>(Expr)) {
1670:     if (isParsingHLASM())
1671:       return Error(StartLoc, "Expected PC-relative expression");
1672:     if (IsOutOfRangeConstant(CE, false))
1673:       return Error(StartLoc, "offset out of range");
1674:     int64_t Value = CE->getValue();
1675:     MCSymbol *Sym = Ctx.createTempSymbol();
1676:     Out.emitLabel(Sym);
1677:     const MCExpr *Base = MCSymbolRefExpr::create(Sym, Ctx);
1678:     Expr = Value == 0 ? Base : MCBinaryExpr::createAdd(Base, Expr, Ctx);
1679:   }
1680: 
```
- **EN**: The range implements or declares functions including `SystemZAsmParser::parsePCRel`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `SystemZAsmParser::parsePCRel` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 1681-1720 / 第 1681-1720 行
```cpp
1681:   // For consistency with the GNU assembler, conservatively assume that a
1682:   // constant offset must by itself be within the given size range.
1683:   if (const auto *BE = dyn_cast<MCBinaryExpr>(Expr))
1684:     if (IsOutOfRangeConstant(BE->getLHS(), false) ||
1685:         IsOutOfRangeConstant(BE->getRHS(),
1686:                              BE->getOpcode() == MCBinaryExpr::Sub))
1687:       return Error(StartLoc, "offset out of range");
1688: 
1689:   // Optionally match :tls_gdcall: or :tls_ldcall: followed by a TLS symbol.
1690:   const MCExpr *Sym = nullptr;
1691:   if (AllowTLS && getLexer().is(AsmToken::Colon)) {
1692:     Parser.Lex();
1693: 
1694:     if (Parser.getTok().isNot(AsmToken::Identifier))
1695:       return Error(Parser.getTok().getLoc(), "unexpected token");
1696: 
1697:     auto Kind = SystemZ::S_None;
1698:     StringRef Name = Parser.getTok().getString();
1699:     if (Name == "tls_gdcall")
1700:       Kind = SystemZ::S_TLSGD;
1701:     else if (Name == "tls_ldcall")
1702:       Kind = SystemZ::S_TLSLDM;
1703:     else
1704:       return Error(Parser.getTok().getLoc(), "unknown TLS tag");
1705:     Parser.Lex();
1706: 
1707:     if (Parser.getTok().isNot(AsmToken::Colon))
1708:       return Error(Parser.getTok().getLoc(), "unexpected token");
1709:     Parser.Lex();
1710: 
1711:     if (Parser.getTok().isNot(AsmToken::Identifier))
1712:       return Error(Parser.getTok().getLoc(), "unexpected token");
1713: 
1714:     StringRef Identifier = Parser.getTok().getString();
1715:     Sym = MCSymbolRefExpr::create(Ctx.getOrCreateSymbol(Identifier),
1716:                                   Kind, Ctx);
1717:     Parser.Lex();
1718:   }
1719: 
1720:   SMLoc EndLoc =
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1721-1760 / 第 1721-1760 行
```cpp
1721:     SMLoc::getFromPointer(Parser.getTok().getLoc().getPointer() - 1);
1722: 
1723:   if (AllowTLS)
1724:     Operands.push_back(SystemZOperand::createImmTLS(Expr, Sym,
1725:                                                     StartLoc, EndLoc));
1726:   else
1727:     Operands.push_back(SystemZOperand::createImm(Expr, StartLoc, EndLoc));
1728: 
1729:   return ParseStatus::Success;
1730: }
1731: 
1732: bool SystemZAsmParser::isLabel(AsmToken &Token) {
1733:   if (isParsingGNU())
1734:     return true;
1735: 
1736:   // HLASM labels are ordinary symbols.
1737:   // An HLASM label always starts at column 1.
1738:   // An ordinary symbol syntax is laid out as follows:
1739:   // Rules:
1740:   // 1. Has to start with an "alphabetic character". Can be followed by up to
1741:   //    62 alphanumeric characters. An "alphabetic character", in this scenario,
1742:   //    is a letter from 'A' through 'Z', or from 'a' through 'z',
1743:   //    or '$', '_', '#', or '@'
1744:   // 2. Labels are case-insensitive. E.g. "lab123", "LAB123", "lAb123", etc.
1745:   //    are all treated as the same symbol. However, the processing for the case
1746:   //    folding will not be done in this function.
1747:   StringRef RawLabel = Token.getString();
1748:   SMLoc Loc = Token.getLoc();
1749: 
1750:   // An HLASM label cannot be empty.
1751:   if (!RawLabel.size())
1752:     return !Error(Loc, "HLASM Label cannot be empty");
1753: 
1754:   // An HLASM label cannot exceed greater than 63 characters.
1755:   if (RawLabel.size() > 63)
1756:     return !Error(Loc, "Maximum length for HLASM Label is 63 characters");
1757: 
1758:   // A label must start with an "alphabetic character".
1759:   if (!isHLASMAlpha(RawLabel[0]))
1760:     return !Error(Loc, "HLASM Label has to start with an alphabetic "
```
- **EN**: The range implements or declares functions including `SystemZAsmParser::isLabel`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZAsmParser::isLabel` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1761-1778 / 第 1761-1778 行
```cpp
1761:                        "character or the underscore character");
1762: 
1763:   // Now, we've established that the length is valid
1764:   // and the first character is alphabetic.
1765:   // Check whether remaining string is alphanumeric.
1766:   for (unsigned I = 1; I < RawLabel.size(); ++I)
1767:     if (!isHLASMAlnum(RawLabel[I]))
1768:       return !Error(Loc, "HLASM Label has to be alphanumeric");
1769: 
1770:   return true;
1771: }
1772: 
1773: // Force static initialization.
1774: // NOLINTNEXTLINE(readability-identifier-naming)
1775: extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
1776: LLVMInitializeSystemZAsmParser() {
1777:   RegisterMCAsmParser<SystemZAsmParser> X(getTheSystemZTarget());
1778: }
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。
- **MC layer**: Uses LLVM MC structures for encoding, printing, parsing, or disassembly. / 使用 LLVM MC 结构进行编码、打印、解析或反汇编。
- **MC streaming**: Emits sections, symbols, expressions, and encoded bytes to object or assembly output. / 向目标文件或汇编输出节、符号、表达式和编码字节。
- **Assembly emission**: Prints target instructions and directives in textual assembly form. / 以文本汇编形式输出目标指令和伪指令。
- **Assembly parsing**: Reads textual assembly operands, directives, and mnemonics. / 读取文本汇编的操作数、伪指令和助记符。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。

## Dependencies / 依赖关系
- `MCTargetDesc/SystemZGNUInstPrinter.h`
- `MCTargetDesc/SystemZMCAsmInfo.h`
- `MCTargetDesc/SystemZMCTargetDesc.h`
- `MCTargetDesc/SystemZTargetStreamer.h`
- `TargetInfo/SystemZTargetInfo.h`
- `llvm/ADT/STLExtras.h`
- `llvm/ADT/SmallVector.h`
- `llvm/ADT/StringExtras.h`
- `llvm/ADT/StringRef.h`
- `llvm/MC/MCAsmInfo.h`
- `llvm/MC/MCContext.h`
- `llvm/MC/MCExpr.h`
- `llvm/MC/MCInst.h`
- `llvm/MC/MCInstBuilder.h`
- `llvm/MC/MCInstrInfo.h`
- `llvm/MC/MCParser/AsmLexer.h`
- `...` (20 more include dependencies omitted for brevity / 其余 20 个 include 依赖已省略)
