# LoongArchAsmParser.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/AsmParser/LoongArchAsmParser.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file parses target assembly syntax into MC/LLVM instructions for the LoongArch backend.
- **用途 (CN)**: 该文件用于 LoongArch 后端，负责将目标汇编语法解析为 MC/LLVM 指令。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40 / 第 1-40 行
```cpp
   1: // LoongArchAsmParser.cpp - Parse LoongArch assembly to MCInst instructions -=//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: #include "MCTargetDesc/LoongArchBaseInfo.h"
  10: #include "MCTargetDesc/LoongArchInstPrinter.h"
  11: #include "MCTargetDesc/LoongArchMCAsmInfo.h"
  12: #include "MCTargetDesc/LoongArchMCTargetDesc.h"
  13: #include "MCTargetDesc/LoongArchMatInt.h"
  14: #include "MCTargetDesc/LoongArchTargetStreamer.h"
  15: #include "TargetInfo/LoongArchTargetInfo.h"
  16: #include "llvm/BinaryFormat/ELF.h"
  17: #include "llvm/MC/MCContext.h"
  18: #include "llvm/MC/MCInstBuilder.h"
  19: #include "llvm/MC/MCInstrInfo.h"
  20: #include "llvm/MC/MCParser/AsmLexer.h"
  21: #include "llvm/MC/MCParser/MCParsedAsmOperand.h"
  22: #include "llvm/MC/MCParser/MCTargetAsmParser.h"
  23: #include "llvm/MC/MCRegisterInfo.h"
  24: #include "llvm/MC/MCStreamer.h"
  25: #include "llvm/MC/MCSubtargetInfo.h"
  26: #include "llvm/MC/MCValue.h"
  27: #include "llvm/MC/TargetRegistry.h"
  28: #include "llvm/Support/Casting.h"
  29: #include "llvm/Support/Compiler.h"
  30: 
  31: using namespace llvm;
  32: 
  33: #define DEBUG_TYPE "loongarch-asm-parser"
  34: 
  35: namespace {
  36: class LoongArchAsmParser : public MCTargetAsmParser {
  37:   SmallVector<FeatureBitset, 4> FeatureBitStack;
  38: 
  39:   SMLoc getLoc() const { return getParser().getTok().getLoc(); }
  40:   bool is64Bit() const { return getSTI().hasFeature(LoongArch::Feature64Bit); }
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `LoongArchBaseInfo.h`, `LoongArchInstPrinter.h`, `LoongArchMCAsmInfo.h`, `LoongArchMCTargetDesc.h`, `LoongArchMatInt.h`, `LoongArchTargetStreamer.h` that expose the LLVM and target interfaces used in later logic. This block declares or refines TableGen records such as `LoongArchAsmParser`.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `LoongArchBaseInfo.h`, `LoongArchInstPrinter.h`, `LoongArchMCAsmInfo.h`, `LoongArchMCTargetDesc.h`, `LoongArchMatInt.h`, `LoongArchTargetStreamer.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 该代码块声明或细化了 `LoongArchAsmParser` 等 TableGen 记录。

### Lines 41-80 / 第 41-80 行
```cpp
  41:   bool has32S() const { return getSTI().hasFeature(LoongArch::Feature32S); }
  42:   LoongArchTargetStreamer &getTargetStreamer() {
  43:     assert(getParser().getStreamer().getTargetStreamer() &&
  44:            "do not have a target streamer");
  45:     MCTargetStreamer &TS = *getParser().getStreamer().getTargetStreamer();
  46:     return static_cast<LoongArchTargetStreamer &>(TS);
  47:   }
  48: 
  49:   struct Inst {
  50:     unsigned Opc;
  51:     uint16_t Specifier;
  52:     Inst(unsigned Opc, uint16_t VK = 0) : Opc(Opc), Specifier(VK) {}
  53:   };
  54:   using InstSeq = SmallVector<Inst>;
  55: 
  56:   /// Parse a register as used in CFI directives.
  57:   bool parseRegister(MCRegister &Reg, SMLoc &StartLoc, SMLoc &EndLoc) override;
  58:   ParseStatus tryParseRegister(MCRegister &Reg, SMLoc &StartLoc,
  59:                                SMLoc &EndLoc) override;
  60: 
  61:   bool parseInstruction(ParseInstructionInfo &Info, StringRef Name,
  62:                         SMLoc NameLoc, OperandVector &Operands) override;
  63: 
  64:   bool matchAndEmitInstruction(SMLoc IDLoc, unsigned &Opcode,
  65:                                OperandVector &Operands, MCStreamer &Out,
  66:                                uint64_t &ErrorInfo,
  67:                                bool MatchingInlineAsm) override;
  68: 
  69:   unsigned checkTargetMatchPredicate(MCInst &Inst) override;
  70: 
  71:   unsigned validateTargetOperandClass(MCParsedAsmOperand &Op,
  72:                                       unsigned Kind) override;
  73: 
  74:   ParseStatus parseDirective(AsmToken DirectiveID) override;
  75: 
  76:   bool generateImmOutOfRangeError(OperandVector &Operands, uint64_t ErrorInfo,
  77:                                   int64_t Lower, int64_t Upper,
  78:                                   const Twine &Msg);
  79: 
  80:   /// Helper for processing MC instructions that have been successfully matched
```
- **EN**: It introduces interface types such as `Inst`, shaping how other backend components interact with this file. The range implements or declares functions including `has32S`, `Inst`. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 它引入了 `Inst` 等接口类型，定义了其他后端组件与本文件交互的方式。 这一段实现或声明了 `has32S`, `Inst` 等函数。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 81-120 / 第 81-120 行
```cpp
  81:   /// by matchAndEmitInstruction.
  82:   bool processInstruction(MCInst &Inst, SMLoc IDLoc, OperandVector &Operands,
  83:                           MCStreamer &Out);
  84: 
  85: // Auto-generated instruction matching functions.
  86: #define GET_ASSEMBLER_HEADER
  87: #include "LoongArchGenAsmMatcher.inc"
  88: 
  89:   ParseStatus parseRegister(OperandVector &Operands);
  90:   ParseStatus parseImmediate(OperandVector &Operands);
  91:   ParseStatus parseOperandWithModifier(OperandVector &Operands);
  92:   ParseStatus parseSImm26Operand(OperandVector &Operands);
  93:   ParseStatus parseAtomicMemOp(OperandVector &Operands);
  94: 
  95:   bool parseOperand(OperandVector &Operands, StringRef Mnemonic);
  96: 
  97:   bool parseDirectiveOption();
  98: 
  99:   void setFeatureBits(uint64_t Feature, StringRef FeatureString) {
 100:     if (!(getSTI().hasFeature(Feature))) {
 101:       MCSubtargetInfo &STI = copySTI();
 102:       setAvailableFeatures(
 103:           ComputeAvailableFeatures(STI.ToggleFeature(FeatureString)));
 104:     }
 105:   }
 106: 
 107:   void clearFeatureBits(uint64_t Feature, StringRef FeatureString) {
 108:     if (getSTI().hasFeature(Feature)) {
 109:       MCSubtargetInfo &STI = copySTI();
 110:       setAvailableFeatures(
 111:           ComputeAvailableFeatures(STI.ToggleFeature(FeatureString)));
 112:     }
 113:   }
 114: 
 115:   void pushFeatureBits() {
 116:     FeatureBitStack.push_back(getSTI().getFeatureBits());
 117:   }
 118: 
 119:   bool popFeatureBits() {
 120:     if (FeatureBitStack.empty())
```
- **EN**: It imports dependencies such as `LoongArchGenAsmMatcher.inc` that expose the LLVM and target interfaces used in later logic. The range implements or declares functions including `setFeatureBits`, `clearFeatureBits`, `pushFeatureBits`, `popFeatureBits`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 它引入了 `LoongArchGenAsmMatcher.inc` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段实现或声明了 `setFeatureBits`, `clearFeatureBits`, `pushFeatureBits`, `popFeatureBits` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 121-160 / 第 121-160 行
```cpp
 121:       return true;
 122: 
 123:     FeatureBitset FeatureBits = FeatureBitStack.pop_back_val();
 124:     copySTI().setFeatureBits(FeatureBits);
 125:     setAvailableFeatures(ComputeAvailableFeatures(FeatureBits));
 126: 
 127:     return false;
 128:   }
 129: 
 130:   // Helper to emit the sequence of instructions generated by the
 131:   // "emitLoadAddress*" functions.
 132:   void emitLAInstSeq(MCRegister DestReg, MCRegister TmpReg,
 133:                      const MCExpr *Symbol, SmallVectorImpl<Inst> &Insts,
 134:                      SMLoc IDLoc, MCStreamer &Out, bool RelaxHint = false);
 135: 
 136:   // Helper to emit pseudo instruction "la.abs $rd, sym".
 137:   void emitLoadAddressAbs(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out);
 138: 
 139:   // Helper to emit pseudo instruction "la.pcrel $rd, sym".
 140:   void emitLoadAddressPcrel(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out);
 141:   // Helper to emit pseudo instruction "la.pcrel $rd, $rj, sym".
 142:   void emitLoadAddressPcrelLarge(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out);
 143: 
 144:   // Helper to emit pseudo instruction "la.got $rd, sym".
 145:   void emitLoadAddressGot(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out);
 146:   // Helper to emit pseudo instruction "la.got $rd, $rj, sym".
 147:   void emitLoadAddressGotLarge(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out);
 148: 
 149:   // Helper to emit pseudo instruction "la.tls.le $rd, sym".
 150:   void emitLoadAddressTLSLE(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out);
 151: 
 152:   // Helper to emit pseudo instruction "la.tls.ie $rd, sym".
 153:   void emitLoadAddressTLSIE(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out);
 154:   // Helper to emit pseudo instruction "la.tls.ie $rd, $rj, sym".
 155:   void emitLoadAddressTLSIELarge(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out);
 156: 
 157:   // Helper to emit pseudo instruction "la.tls.ld $rd, sym".
 158:   void emitLoadAddressTLSLD(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out);
 159:   // Helper to emit pseudo instruction "la.tls.ld $rd, $rj, sym".
 160:   void emitLoadAddressTLSLDLarge(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out);
```
- **EN**: This span continues the file's main responsibility: this file parses target assembly syntax into MC/LLVM instructions for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 161-200 / 第 161-200 行
```cpp
 161: 
 162:   // Helper to emit pseudo instruction "la.tls.gd $rd, sym".
 163:   void emitLoadAddressTLSGD(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out);
 164:   // Helper to emit pseudo instruction "la.tls.gd $rd, $rj, sym".
 165:   void emitLoadAddressTLSGDLarge(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out);
 166: 
 167:   // Helper to emit pseudo instruction "la.tls.desc $rd, sym".
 168:   void emitLoadAddressTLSDesc(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out);
 169:   // Helper to emit pseudo instruction "la.tls.desc $rd, $rj, sym".
 170:   void emitLoadAddressTLSDescLarge(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out);
 171: 
 172:   // Helper to emit pseudo instruction "li.w/d $rd, $imm".
 173:   void emitLoadImm(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out);
 174: 
 175:   // Helper to emit pseudo instruction "call{3x} sym" or "tail{3x} $rj, sym".
 176:   void emitFuncCall(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out, bool IsTailCall,
 177:                     bool IsCall36);
 178: 
 179: public:
 180:   enum LoongArchMatchResultTy {
 181:     Match_Dummy = FIRST_TARGET_MATCH_RESULT_TY,
 182:     Match_RequiresMsbNotLessThanLsb,
 183:     Match_RequiresOpnd2NotR0R1,
 184:     Match_RequiresAMORdDifferRkRj,
 185:     Match_RequiresLAORdDifferRj,
 186:     Match_RequiresLAORdR4,
 187: #define GET_OPERAND_DIAGNOSTIC_TYPES
 188: #include "LoongArchGenAsmMatcher.inc"
 189: #undef GET_OPERAND_DIAGNOSTIC_TYPES
 190:   };
 191: 
 192:   static bool classifySymbolRef(const MCExpr *Expr,
 193:                                 LoongArchMCExpr::Specifier &Kind);
 194: 
 195:   LoongArchAsmParser(const MCSubtargetInfo &STI, MCAsmParser &Parser,
 196:                      const MCInstrInfo &MII)
 197:       : MCTargetAsmParser(STI, MII) {
 198:     Parser.addAliasForDirective(".half", ".2byte");
 199:     Parser.addAliasForDirective(".hword", ".2byte");
 200:     Parser.addAliasForDirective(".word", ".4byte");
```
- **EN**: It imports dependencies such as `LoongArchGenAsmMatcher.inc` that expose the LLVM and target interfaces used in later logic. The range implements or declares functions including `LoongArchAsmParser`. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 它引入了 `LoongArchGenAsmMatcher.inc` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段实现或声明了 `LoongArchAsmParser` 等函数。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 201-240 / 第 201-240 行
```cpp
 201:     Parser.addAliasForDirective(".dword", ".8byte");
 202: 
 203:     // Initialize the set of available features.
 204:     setAvailableFeatures(ComputeAvailableFeatures(STI.getFeatureBits()));
 205:   }
 206: };
 207: 
 208: // Instances of this class represent a parsed LoongArch machine instruction.
 209: class LoongArchOperand : public MCParsedAsmOperand {
 210:   enum class KindTy {
 211:     Token,
 212:     Register,
 213:     Immediate,
 214:   } Kind;
 215: 
 216:   struct RegOp {
 217:     MCRegister RegNum;
 218:   };
 219: 
 220:   struct ImmOp {
 221:     const MCExpr *Val;
 222:   };
 223: 
 224:   SMLoc StartLoc, EndLoc;
 225:   union {
 226:     StringRef Tok;
 227:     struct RegOp Reg;
 228:     struct ImmOp Imm;
 229:   };
 230: 
 231: public:
 232:   LoongArchOperand(KindTy K) : MCParsedAsmOperand(), Kind(K) {}
 233: 
 234:   bool isToken() const override { return Kind == KindTy::Token; }
 235:   bool isReg() const override { return Kind == KindTy::Register; }
 236:   bool isImm() const override { return Kind == KindTy::Immediate; }
 237:   bool isMem() const override { return false; }
 238:   void setReg(MCRegister PhysReg) { Reg.RegNum = PhysReg; }
 239:   bool isGPR() const {
 240:     return Kind == KindTy::Register &&
```
- **EN**: This block declares or refines TableGen records such as `LoongArchOperand`. The range implements or declares functions including `LoongArchOperand`, `setReg`, `isGPR`.
- **CN**: 该代码块声明或细化了 `LoongArchOperand` 等 TableGen 记录。 这一段实现或声明了 `LoongArchOperand`, `setReg`, `isGPR` 等函数。

### Lines 241-280 / 第 241-280 行
```cpp
 241:            LoongArchMCRegisterClasses[LoongArch::GPRRegClassID].contains(
 242:                Reg.RegNum);
 243:   }
 244: 
 245:   static bool evaluateConstantImm(const MCExpr *Expr, int64_t &Imm,
 246:                                   LoongArchMCExpr::Specifier &VK) {
 247:     if (auto *LE = dyn_cast<LoongArchMCExpr>(Expr)) {
 248:       VK = LE->getSpecifier();
 249:       return false;
 250:     }
 251: 
 252:     if (auto CE = dyn_cast<MCConstantExpr>(Expr)) {
 253:       Imm = CE->getValue();
 254:       return true;
 255:     }
 256: 
 257:     return false;
 258:   }
 259: 
 260:   template <unsigned N, int P = 0> bool isUImm() const {
 261:     if (!isImm())
 262:       return false;
 263: 
 264:     int64_t Imm;
 265:     LoongArchMCExpr::Specifier VK = LoongArchMCExpr::VK_None;
 266:     bool IsConstantImm = evaluateConstantImm(getImm(), Imm, VK);
 267:     return IsConstantImm && isUInt<N>(Imm - P) &&
 268:            VK == LoongArchMCExpr::VK_None;
 269:   }
 270: 
 271:   template <unsigned N, unsigned S = 0> bool isSImm() const {
 272:     if (!isImm())
 273:       return false;
 274: 
 275:     int64_t Imm;
 276:     LoongArchMCExpr::Specifier VK = LoongArchMCExpr::VK_None;
 277:     bool IsConstantImm = evaluateConstantImm(getImm(), Imm, VK);
 278:     return IsConstantImm && isShiftedInt<N, S>(Imm) &&
 279:            VK == LoongArchMCExpr::VK_None;
 280:   }
```
- **EN**: The range implements or declares functions including `evaluateConstantImm`, `isUImm`, `isSImm`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `evaluateConstantImm`, `isUImm`, `isSImm` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 281-320 / 第 281-320 行
```cpp
 281: 
 282:   bool isBareSymbol() const {
 283:     int64_t Imm;
 284:     LoongArchMCExpr::Specifier VK = LoongArchMCExpr::VK_None;
 285:     // Must be of 'immediate' type but not a constant.
 286:     if (!isImm() || evaluateConstantImm(getImm(), Imm, VK))
 287:       return false;
 288:     return LoongArchAsmParser::classifySymbolRef(getImm(), VK) &&
 289:            VK == LoongArchMCExpr::VK_None;
 290:   }
 291: 
 292:   bool isTPRelAddSymbol() const {
 293:     int64_t Imm;
 294:     LoongArchMCExpr::Specifier VK = LoongArchMCExpr::VK_None;
 295:     // Must be of 'immediate' type but not a constant.
 296:     if (!isImm() || evaluateConstantImm(getImm(), Imm, VK))
 297:       return false;
 298:     return LoongArchAsmParser::classifySymbolRef(getImm(), VK) &&
 299:            VK == ELF::R_LARCH_TLS_LE_ADD_R;
 300:   }
 301: 
 302:   bool isUImm1() const { return isUImm<1>(); }
 303:   bool isUImm2() const { return isUImm<2>(); }
 304:   bool isUImm2plus1() const { return isUImm<2, 1>(); }
 305:   bool isUImm3() const { return isUImm<3>(); }
 306:   bool isUImm4() const { return isUImm<4>(); }
 307:   bool isSImm5() const { return isSImm<5>(); }
 308:   bool isUImm5() const { return isUImm<5>(); }
 309:   bool isUImm6() const { return isUImm<6>(); }
 310:   bool isUImm7() const { return isUImm<7>(); }
 311:   bool isSImm8() const { return isSImm<8>(); }
 312:   bool isSImm8lsl1() const { return isSImm<8, 1>(); }
 313:   bool isSImm8lsl2() const { return isSImm<8, 2>(); }
 314:   bool isSImm8lsl3() const { return isSImm<8, 3>(); }
 315:   bool isUImm8() const { return isUImm<8>(); }
 316:   bool isSImm9lsl3() const { return isSImm<9, 3>(); }
 317:   bool isSImm10() const { return isSImm<10>(); }
 318:   bool isSImm10lsl2() const { return isSImm<10, 2>(); }
 319:   bool isSImm11lsl1() const { return isSImm<11, 1>(); }
 320:   bool isSImm12() const { return isSImm<12>(); }
```
- **EN**: The range implements or declares functions including `isBareSymbol`, `isTPRelAddSymbol`, `isUImm1`, `isUImm2`, `isUImm2plus1`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `isBareSymbol`, `isTPRelAddSymbol`, `isUImm1`, `isUImm2`, `isUImm2plus1` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 321-360 / 第 321-360 行
```cpp
 321: 
 322:   bool isSImm12addlike() const {
 323:     if (!isImm())
 324:       return false;
 325: 
 326:     int64_t Imm;
 327:     LoongArchMCExpr::Specifier VK = LoongArchMCExpr::VK_None;
 328:     bool IsConstantImm = evaluateConstantImm(getImm(), Imm, VK);
 329:     bool IsValidKind =
 330:         VK == LoongArchMCExpr::VK_None || VK == ELF::R_LARCH_PCALA_LO12 ||
 331:         VK == ELF::R_LARCH_GOT_PC_LO12 || VK == ELF::R_LARCH_TLS_IE_PC_LO12 ||
 332:         VK == ELF::R_LARCH_TLS_LE_LO12_R || VK == ELF::R_LARCH_TLS_DESC_LD ||
 333:         VK == ELF::R_LARCH_TLS_DESC_PC_LO12 || VK == ELF::R_LARCH_PCADD_LO12 ||
 334:         VK == ELF::R_LARCH_GOT_PCADD_LO12 ||
 335:         VK == ELF::R_LARCH_TLS_IE_PCADD_LO12 ||
 336:         VK == ELF::R_LARCH_TLS_LD_PCADD_LO12 ||
 337:         VK == ELF::R_LARCH_TLS_GD_PCADD_LO12 ||
 338:         VK == ELF::R_LARCH_TLS_DESC_PCADD_LO12;
 339:     return IsConstantImm
 340:                ? isInt<12>(Imm) && IsValidKind
 341:                : LoongArchAsmParser::classifySymbolRef(getImm(), VK) &&
 342:                      IsValidKind;
 343:   }
 344: 
 345:   bool isSImm12lu52id() const {
 346:     if (!isImm())
 347:       return false;
 348: 
 349:     int64_t Imm;
 350:     LoongArchMCExpr::Specifier VK = LoongArchMCExpr::VK_None;
 351:     bool IsConstantImm = evaluateConstantImm(getImm(), Imm, VK);
 352:     bool IsValidKind =
 353:         VK == LoongArchMCExpr::VK_None || VK == ELF::R_LARCH_ABS64_HI12 ||
 354:         VK == ELF::R_LARCH_PCALA64_HI12 || VK == ELF::R_LARCH_GOT64_HI12 ||
 355:         VK == ELF::R_LARCH_GOT64_PC_HI12 || VK == ELF::R_LARCH_TLS_LE64_HI12 ||
 356:         VK == ELF::R_LARCH_TLS_IE64_HI12 ||
 357:         VK == ELF::R_LARCH_TLS_IE64_PC_HI12 ||
 358:         VK == ELF::R_LARCH_TLS_DESC64_HI12 ||
 359:         VK == ELF::R_LARCH_TLS_DESC64_PC_HI12;
 360:     return IsConstantImm
```
- **EN**: The range implements or declares functions including `isSImm12addlike`, `isSImm12lu52id`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `isSImm12addlike`, `isSImm12lu52id` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 361-400 / 第 361-400 行
```cpp
 361:                ? isInt<12>(Imm) && IsValidKind
 362:                : LoongArchAsmParser::classifySymbolRef(getImm(), VK) &&
 363:                      IsValidKind;
 364:   }
 365: 
 366:   bool isUImm12() const { return isUImm<12>(); }
 367: 
 368:   bool isUImm12ori() const {
 369:     if (!isImm())
 370:       return false;
 371: 
 372:     int64_t Imm;
 373:     LoongArchMCExpr::Specifier VK = LoongArchMCExpr::VK_None;
 374:     bool IsConstantImm = evaluateConstantImm(getImm(), Imm, VK);
 375:     bool IsValidKind =
 376:         VK == LoongArchMCExpr::VK_None || VK == ELF::R_LARCH_ABS_LO12 ||
 377:         VK == ELF::R_LARCH_PCALA_LO12 || VK == ELF::R_LARCH_GOT_LO12 ||
 378:         VK == ELF::R_LARCH_GOT_PC_LO12 || VK == ELF::R_LARCH_TLS_LE_LO12 ||
 379:         VK == ELF::R_LARCH_TLS_IE_LO12 || VK == ELF::R_LARCH_TLS_IE_PC_LO12 ||
 380:         VK == ELF::R_LARCH_TLS_DESC_LO12;
 381:     return IsConstantImm
 382:                ? isUInt<12>(Imm) && IsValidKind
 383:                : LoongArchAsmParser::classifySymbolRef(getImm(), VK) &&
 384:                      IsValidKind;
 385:   }
 386: 
 387:   bool isSImm13() const { return isSImm<13>(); }
 388:   bool isUImm14() const { return isUImm<14>(); }
 389:   bool isUImm15() const { return isUImm<15>(); }
 390: 
 391:   bool isSImm14lsl2() const { return isSImm<14, 2>(); }
 392:   bool isSImm16() const { return isSImm<16>(); }
 393: 
 394:   bool isSImm16lsl2() const {
 395:     if (!isImm())
 396:       return false;
 397: 
 398:     int64_t Imm;
 399:     LoongArchMCExpr::Specifier VK = LoongArchMCExpr::VK_None;
 400:     bool IsConstantImm = evaluateConstantImm(getImm(), Imm, VK);
```
- **EN**: The range implements or declares functions including `isUImm12`, `isUImm12ori`, `isSImm13`, `isUImm14`, `isUImm15`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `isUImm12`, `isUImm12ori`, `isSImm13`, `isUImm14`, `isUImm15` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 401-440 / 第 401-440 行
```cpp
 401:     bool IsValidKind =
 402:         VK == LoongArchMCExpr::VK_None || VK == ELF::R_LARCH_B16 ||
 403:         VK == ELF::R_LARCH_PCALA_LO12 || VK == ELF::R_LARCH_TLS_DESC_CALL;
 404:     return IsConstantImm
 405:                ? isShiftedInt<16, 2>(Imm) && IsValidKind
 406:                : LoongArchAsmParser::classifySymbolRef(getImm(), VK) &&
 407:                      IsValidKind;
 408:   }
 409: 
 410:   bool isSImm20() const { return isSImm<20>(); }
 411: 
 412:   bool isSImm20pcalau12i() const {
 413:     if (!isImm())
 414:       return false;
 415: 
 416:     int64_t Imm;
 417:     LoongArchMCExpr::Specifier VK = LoongArchMCExpr::VK_None;
 418:     bool IsConstantImm = evaluateConstantImm(getImm(), Imm, VK);
 419:     bool IsValidKind =
 420:         VK == LoongArchMCExpr::VK_None || VK == ELF::R_LARCH_PCALA_HI20 ||
 421:         VK == ELF::R_LARCH_GOT_PC_HI20 || VK == ELF::R_LARCH_TLS_IE_PC_HI20 ||
 422:         VK == ELF::R_LARCH_TLS_LD_PC_HI20 ||
 423:         VK == ELF::R_LARCH_TLS_GD_PC_HI20 ||
 424:         VK == ELF::R_LARCH_TLS_DESC_PC_HI20;
 425:     return IsConstantImm
 426:                ? isInt<20>(Imm) && IsValidKind
 427:                : LoongArchAsmParser::classifySymbolRef(getImm(), VK) &&
 428:                      IsValidKind;
 429:   }
 430: 
 431:   bool isSImm20lu12iw() const {
 432:     if (!isImm())
 433:       return false;
 434: 
 435:     int64_t Imm;
 436:     LoongArchMCExpr::Specifier VK = LoongArchMCExpr::VK_None;
 437:     bool IsConstantImm = evaluateConstantImm(getImm(), Imm, VK);
 438:     bool IsValidKind =
 439:         VK == LoongArchMCExpr::VK_None || VK == ELF::R_LARCH_ABS_HI20 ||
 440:         VK == ELF::R_LARCH_GOT_HI20 || VK == ELF::R_LARCH_TLS_GD_HI20 ||
```
- **EN**: The range implements or declares functions including `isSImm20`, `isSImm20pcalau12i`, `isSImm20lu12iw`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `isSImm20`, `isSImm20pcalau12i`, `isSImm20lu12iw` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 441-480 / 第 441-480 行
```cpp
 441:         VK == ELF::R_LARCH_TLS_LD_HI20 || VK == ELF::R_LARCH_TLS_IE_HI20 ||
 442:         VK == ELF::R_LARCH_TLS_LE_HI20 || VK == ELF::R_LARCH_TLS_LE_HI20_R ||
 443:         VK == ELF::R_LARCH_TLS_DESC_HI20;
 444:     return IsConstantImm
 445:                ? isInt<20>(Imm) && IsValidKind
 446:                : LoongArchAsmParser::classifySymbolRef(getImm(), VK) &&
 447:                      IsValidKind;
 448:   }
 449: 
 450:   bool isSImm20lu32id() const {
 451:     if (!isImm())
 452:       return false;
 453: 
 454:     int64_t Imm;
 455:     LoongArchMCExpr::Specifier VK = LoongArchMCExpr::VK_None;
 456:     bool IsConstantImm = evaluateConstantImm(getImm(), Imm, VK);
 457:     bool IsValidKind =
 458:         VK == LoongArchMCExpr::VK_None || VK == ELF::R_LARCH_ABS64_LO20 ||
 459:         VK == ELF::R_LARCH_PCALA64_LO20 || VK == ELF::R_LARCH_GOT64_LO20 ||
 460:         VK == ELF::R_LARCH_GOT64_PC_LO20 || VK == ELF::R_LARCH_TLS_IE64_LO20 ||
 461:         VK == ELF::R_LARCH_TLS_IE64_PC_LO20 ||
 462:         VK == ELF::R_LARCH_TLS_LE64_LO20 ||
 463:         VK == ELF::R_LARCH_TLS_DESC64_PC_LO20 ||
 464:         VK == ELF::R_LARCH_TLS_DESC64_LO20;
 465: 
 466:     return IsConstantImm
 467:                ? isInt<20>(Imm) && IsValidKind
 468:                : LoongArchAsmParser::classifySymbolRef(getImm(), VK) &&
 469:                      IsValidKind;
 470:   }
 471: 
 472:   bool isSImm20pcaddu12i() const {
 473:     if (!isImm())
 474:       return false;
 475: 
 476:     int64_t Imm;
 477:     LoongArchMCExpr::Specifier VK = LoongArchMCExpr::VK_None;
 478:     bool IsConstantImm = evaluateConstantImm(getImm(), Imm, VK);
 479:     bool IsValidKind =
 480:         VK == LoongArchMCExpr::VK_None || VK == ELF::R_LARCH_CALL30 ||
```
- **EN**: The range implements or declares functions including `isSImm20lu32id`, `isSImm20pcaddu12i`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `isSImm20lu32id`, `isSImm20pcaddu12i` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 481-520 / 第 481-520 行
```cpp
 481:         VK == ELF::R_LARCH_PCADD_HI20 || VK == ELF::R_LARCH_GOT_PCADD_HI20 ||
 482:         VK == ELF::R_LARCH_TLS_IE_PCADD_HI20 ||
 483:         VK == ELF::R_LARCH_TLS_LD_PCADD_HI20 ||
 484:         VK == ELF::R_LARCH_TLS_GD_PCADD_HI20 ||
 485:         VK == ELF::R_LARCH_TLS_DESC_PCADD_HI20;
 486: 
 487:     return IsConstantImm
 488:                ? isInt<20>(Imm) && IsValidKind
 489:                : LoongArchAsmParser::classifySymbolRef(getImm(), VK) &&
 490:                      IsValidKind;
 491:   }
 492: 
 493:   bool isSImm20pcaddu18i() const {
 494:     if (!isImm())
 495:       return false;
 496: 
 497:     int64_t Imm;
 498:     LoongArchMCExpr::Specifier VK = LoongArchMCExpr::VK_None;
 499:     bool IsConstantImm = evaluateConstantImm(getImm(), Imm, VK);
 500:     bool IsValidKind =
 501:         VK == LoongArchMCExpr::VK_None || VK == ELF::R_LARCH_CALL36;
 502: 
 503:     return IsConstantImm
 504:                ? isInt<20>(Imm) && IsValidKind
 505:                : LoongArchAsmParser::classifySymbolRef(getImm(), VK) &&
 506:                      IsValidKind;
 507:   }
 508: 
 509:   bool isSImm20pcaddi() const {
 510:     if (!isImm())
 511:       return false;
 512: 
 513:     int64_t Imm;
 514:     LoongArchMCExpr::Specifier VK = LoongArchMCExpr::VK_None;
 515:     bool IsConstantImm = evaluateConstantImm(getImm(), Imm, VK);
 516:     bool IsValidKind = VK == LoongArchMCExpr::VK_None ||
 517:                        VK == ELF::R_LARCH_PCREL20_S2 ||
 518:                        VK == ELF::R_LARCH_TLS_LD_PCREL20_S2 ||
 519:                        VK == ELF::R_LARCH_TLS_GD_PCREL20_S2 ||
 520:                        VK == ELF::R_LARCH_TLS_DESC_PCREL20_S2;
```
- **EN**: The range implements or declares functions including `isSImm20pcaddu18i`, `isSImm20pcaddi`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `isSImm20pcaddu18i`, `isSImm20pcaddi` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 521-560 / 第 521-560 行
```cpp
 521:     return IsConstantImm
 522:                ? isInt<20>(Imm) && IsValidKind
 523:                : LoongArchAsmParser::classifySymbolRef(getImm(), VK) &&
 524:                      IsValidKind;
 525:   }
 526: 
 527:   bool isSImm21lsl2() const {
 528:     if (!isImm())
 529:       return false;
 530: 
 531:     int64_t Imm;
 532:     LoongArchMCExpr::Specifier VK = LoongArchMCExpr::VK_None;
 533:     bool IsConstantImm = evaluateConstantImm(getImm(), Imm, VK);
 534:     bool IsValidKind = VK == LoongArchMCExpr::VK_None || VK == ELF::R_LARCH_B21;
 535:     return IsConstantImm
 536:                ? isShiftedInt<21, 2>(Imm) && IsValidKind
 537:                : LoongArchAsmParser::classifySymbolRef(getImm(), VK) &&
 538:                      IsValidKind;
 539:   }
 540: 
 541:   bool isSImm26Operand() const {
 542:     if (!isImm())
 543:       return false;
 544: 
 545:     int64_t Imm;
 546:     LoongArchMCExpr::Specifier VK = LoongArchMCExpr::VK_None;
 547:     bool IsConstantImm = evaluateConstantImm(getImm(), Imm, VK);
 548:     bool IsValidKind = VK == LoongArchMCExpr::VK_None || VK == ELF::R_LARCH_B26;
 549:     return IsConstantImm
 550:                ? isShiftedInt<26, 2>(Imm) && IsValidKind
 551:                : LoongArchAsmParser::classifySymbolRef(getImm(), VK) &&
 552:                      IsValidKind;
 553:   }
 554: 
 555:   bool isImm32() const { return isSImm<32>() || isUImm<32>(); }
 556:   bool isImm64() const {
 557:     if (!isImm())
 558:       return false;
 559:     int64_t Imm;
 560:     LoongArchMCExpr::Specifier VK = LoongArchMCExpr::VK_None;
```
- **EN**: The range implements or declares functions including `isSImm21lsl2`, `isSImm26Operand`, `isImm32`, `isImm64`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `isSImm21lsl2`, `isSImm26Operand`, `isImm32`, `isImm64` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 561-600 / 第 561-600 行
```cpp
 561:     bool IsConstantImm = evaluateConstantImm(getImm(), Imm, VK);
 562:     return IsConstantImm && VK == LoongArchMCExpr::VK_None;
 563:   }
 564: 
 565:   /// Gets location of the first token of this operand.
 566:   SMLoc getStartLoc() const override { return StartLoc; }
 567:   /// Gets location of the last token of this operand.
 568:   SMLoc getEndLoc() const override { return EndLoc; }
 569: 
 570:   MCRegister getReg() const override {
 571:     assert(Kind == KindTy::Register && "Invalid type access!");
 572:     return Reg.RegNum;
 573:   }
 574: 
 575:   const MCExpr *getImm() const {
 576:     assert(Kind == KindTy::Immediate && "Invalid type access!");
 577:     return Imm.Val;
 578:   }
 579: 
 580:   StringRef getToken() const {
 581:     assert(Kind == KindTy::Token && "Invalid type access!");
 582:     return Tok;
 583:   }
 584: 
 585:   void print(raw_ostream &OS, const MCAsmInfo &MAI) const override {
 586:     auto RegName = [](MCRegister Reg) {
 587:       if (Reg)
 588:         return LoongArchInstPrinter::getRegisterName(Reg);
 589:       else
 590:         return "noreg";
 591:     };
 592: 
 593:     switch (Kind) {
 594:     case KindTy::Immediate:
 595:       MAI.printExpr(OS, *getImm());
 596:       break;
 597:     case KindTy::Register:
 598:       OS << "<register " << RegName(getReg()) << ">";
 599:       break;
 600:     case KindTy::Token:
```
- **EN**: The range implements or declares functions including `getToken`, `print`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `getToken`, `print` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 601-640 / 第 601-640 行
```cpp
 601:       OS << "'" << getToken() << "'";
 602:       break;
 603:     }
 604:   }
 605: 
 606:   static std::unique_ptr<LoongArchOperand> createToken(StringRef Str, SMLoc S) {
 607:     auto Op = std::make_unique<LoongArchOperand>(KindTy::Token);
 608:     Op->Tok = Str;
 609:     Op->StartLoc = S;
 610:     Op->EndLoc = S;
 611:     return Op;
 612:   }
 613: 
 614:   static std::unique_ptr<LoongArchOperand> createReg(MCRegister Reg, SMLoc S,
 615:                                                      SMLoc E) {
 616:     auto Op = std::make_unique<LoongArchOperand>(KindTy::Register);
 617:     Op->Reg.RegNum = Reg;
 618:     Op->StartLoc = S;
 619:     Op->EndLoc = E;
 620:     return Op;
 621:   }
 622: 
 623:   static std::unique_ptr<LoongArchOperand> createImm(const MCExpr *Val, SMLoc S,
 624:                                                      SMLoc E) {
 625:     auto Op = std::make_unique<LoongArchOperand>(KindTy::Immediate);
 626:     Op->Imm.Val = Val;
 627:     Op->StartLoc = S;
 628:     Op->EndLoc = E;
 629:     return Op;
 630:   }
 631: 
 632:   void addExpr(MCInst &Inst, const MCExpr *Expr) const {
 633:     if (auto CE = dyn_cast<MCConstantExpr>(Expr))
 634:       Inst.addOperand(MCOperand::createImm(CE->getValue()));
 635:     else
 636:       Inst.addOperand(MCOperand::createExpr(Expr));
 637:   }
 638: 
 639:   // Used by the TableGen Code.
 640:   void addRegOperands(MCInst &Inst, unsigned N) const {
```
- **EN**: The range implements or declares functions including `createToken`, `createReg`, `createImm`, `addExpr`, `addRegOperands`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `createToken`, `createReg`, `createImm`, `addExpr`, `addRegOperands` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 641-680 / 第 641-680 行
```cpp
 641:     assert(N == 1 && "Invalid number of operands!");
 642:     Inst.addOperand(MCOperand::createReg(getReg()));
 643:   }
 644:   void addImmOperands(MCInst &Inst, unsigned N) const {
 645:     assert(N == 1 && "Invalid number of operands!");
 646:     addExpr(Inst, getImm());
 647:   }
 648: };
 649: } // end namespace
 650: 
 651: #define GET_REGISTER_MATCHER
 652: #define GET_SUBTARGET_FEATURE_NAME
 653: #define GET_MATCHER_IMPLEMENTATION
 654: #define GET_MNEMONIC_SPELL_CHECKER
 655: #include "LoongArchGenAsmMatcher.inc"
 656: 
 657: static MCRegister convertFPR32ToFPR64(MCRegister Reg) {
 658:   assert(Reg >= LoongArch::F0 && Reg <= LoongArch::F31 && "Invalid register");
 659:   return Reg - LoongArch::F0 + LoongArch::F0_64;
 660: }
 661: 
 662: // Attempts to match Name as a register (either using the default name or
 663: // alternative ABI names), setting RegNo to the matching register. Upon
 664: // failure, returns true and sets RegNo to 0.
 665: static bool matchRegisterNameHelper(MCRegister &RegNo, StringRef Name) {
 666:   RegNo = MatchRegisterName(Name);
 667:   // The 32-bit and 64-bit FPRs have the same asm name. Check that the initial
 668:   // match always matches the 32-bit variant, and not the 64-bit one.
 669:   assert(!(RegNo >= LoongArch::F0_64 && RegNo <= LoongArch::F31_64));
 670:   // The default FPR register class is based on the tablegen enum ordering.
 671:   static_assert(LoongArch::F0 < LoongArch::F0_64,
 672:                 "FPR matching must be updated");
 673:   if (RegNo == LoongArch::NoRegister)
 674:     RegNo = MatchRegisterAltName(Name);
 675: 
 676:   return RegNo == LoongArch::NoRegister;
 677: }
 678: 
 679: bool LoongArchAsmParser::parseRegister(MCRegister &Reg, SMLoc &StartLoc,
 680:                                        SMLoc &EndLoc) {
```
- **EN**: It imports dependencies such as `LoongArchGenAsmMatcher.inc` that expose the LLVM and target interfaces used in later logic. The range implements or declares functions including `addImmOperands`, `convertFPR32ToFPR64`, `matchRegisterNameHelper`, `LoongArchAsmParser::parseRegister`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 它引入了 `LoongArchGenAsmMatcher.inc` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段实现或声明了 `addImmOperands`, `convertFPR32ToFPR64`, `matchRegisterNameHelper`, `LoongArchAsmParser::parseRegister` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 681-720 / 第 681-720 行
```cpp
 681:   if (!tryParseRegister(Reg, StartLoc, EndLoc).isSuccess())
 682:     return Error(getLoc(), "invalid register name");
 683: 
 684:   if (!LoongArchMCRegisterClasses[LoongArch::GPRRegClassID].contains(Reg) &&
 685:       !LoongArchMCRegisterClasses[LoongArch::FPR32RegClassID].contains(Reg))
 686:     return Error(getLoc(), "invalid register name");
 687: 
 688:   return false;
 689: }
 690: 
 691: ParseStatus LoongArchAsmParser::tryParseRegister(MCRegister &Reg,
 692:                                                  SMLoc &StartLoc,
 693:                                                  SMLoc &EndLoc) {
 694:   const AsmToken &Tok = getParser().getTok();
 695:   StartLoc = Tok.getLoc();
 696:   EndLoc = Tok.getEndLoc();
 697: 
 698:   parseOptionalToken(AsmToken::Dollar);
 699:   if (getLexer().getKind() != AsmToken::Identifier)
 700:     return ParseStatus::NoMatch;
 701: 
 702:   StringRef Name = Tok.getIdentifier();
 703:   if (matchRegisterNameHelper(Reg, Name))
 704:     return ParseStatus::NoMatch;
 705: 
 706:   getParser().Lex(); // Eat identifier token.
 707:   return ParseStatus::Success;
 708: }
 709: 
 710: bool LoongArchAsmParser::classifySymbolRef(const MCExpr *Expr,
 711:                                            LoongArchMCExpr::Specifier &Kind) {
 712:   Kind = LoongArchMCExpr::VK_None;
 713: 
 714:   if (const LoongArchMCExpr *RE = dyn_cast<LoongArchMCExpr>(Expr)) {
 715:     Kind = RE->getSpecifier();
 716:     Expr = RE->getSubExpr();
 717:   }
 718: 
 719:   MCValue Res;
 720:   if (Expr->evaluateAsRelocatable(Res, nullptr))
```
- **EN**: The range implements or declares functions including `LoongArchAsmParser::tryParseRegister`, `LoongArchAsmParser::classifySymbolRef`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchAsmParser::tryParseRegister`, `LoongArchAsmParser::classifySymbolRef` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 721-760 / 第 721-760 行
```cpp
 721:     return Res.getSpecifier() == LoongArchMCExpr::VK_None;
 722:   return false;
 723: }
 724: 
 725: ParseStatus LoongArchAsmParser::parseRegister(OperandVector &Operands) {
 726:   if (!parseOptionalToken(AsmToken::Dollar))
 727:     return ParseStatus::NoMatch;
 728:   if (getLexer().getKind() != AsmToken::Identifier)
 729:     return ParseStatus::NoMatch;
 730: 
 731:   StringRef Name = getLexer().getTok().getIdentifier();
 732:   MCRegister RegNo;
 733:   matchRegisterNameHelper(RegNo, Name);
 734:   if (RegNo == LoongArch::NoRegister)
 735:     return ParseStatus::NoMatch;
 736: 
 737:   SMLoc S = getLoc();
 738:   SMLoc E = SMLoc::getFromPointer(S.getPointer() + Name.size());
 739:   getLexer().Lex();
 740:   Operands.push_back(LoongArchOperand::createReg(RegNo, S, E));
 741: 
 742:   return ParseStatus::Success;
 743: }
 744: 
 745: ParseStatus LoongArchAsmParser::parseImmediate(OperandVector &Operands) {
 746:   SMLoc S = getLoc();
 747:   SMLoc E;
 748:   const MCExpr *Res;
 749: 
 750:   switch (getLexer().getKind()) {
 751:   default:
 752:     return ParseStatus::NoMatch;
 753:   case AsmToken::LParen:
 754:   case AsmToken::Dot:
 755:   case AsmToken::Minus:
 756:   case AsmToken::Plus:
 757:   case AsmToken::Exclaim:
 758:   case AsmToken::Tilde:
 759:   case AsmToken::Integer:
 760:   case AsmToken::String:
```
- **EN**: The range implements or declares functions including `LoongArchAsmParser::parseRegister`, `LoongArchAsmParser::parseImmediate`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchAsmParser::parseRegister`, `LoongArchAsmParser::parseImmediate` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 761-800 / 第 761-800 行
```cpp
 761:   case AsmToken::Identifier:
 762:     if (getParser().parseExpression(Res, E))
 763:       return ParseStatus::Failure;
 764:     break;
 765:   case AsmToken::Percent:
 766:     return parseOperandWithModifier(Operands);
 767:   }
 768: 
 769:   Operands.push_back(LoongArchOperand::createImm(Res, S, E));
 770:   return ParseStatus::Success;
 771: }
 772: 
 773: ParseStatus
 774: LoongArchAsmParser::parseOperandWithModifier(OperandVector &Operands) {
 775:   SMLoc S = getLoc();
 776:   SMLoc E;
 777: 
 778:   if (getLexer().getKind() != AsmToken::Percent)
 779:     return Error(getLoc(), "expected '%' for operand modifier");
 780: 
 781:   getParser().Lex(); // Eat '%'
 782: 
 783:   if (getLexer().getKind() != AsmToken::Identifier)
 784:     return Error(getLoc(), "expected valid identifier for operand modifier");
 785:   StringRef Identifier = getParser().getTok().getIdentifier();
 786:   auto VK = LoongArch::parseSpecifier(Identifier);
 787:   if (VK == LoongArchMCExpr::VK_None)
 788:     return Error(getLoc(), "invalid relocation specifier");
 789: 
 790:   getParser().Lex(); // Eat the identifier
 791:   if (getLexer().getKind() != AsmToken::LParen)
 792:     return Error(getLoc(), "expected '('");
 793:   getParser().Lex(); // Eat '('
 794: 
 795:   const MCExpr *SubExpr;
 796:   if (getParser().parseParenExpression(SubExpr, E))
 797:     return ParseStatus::Failure;
 798: 
 799:   const MCExpr *ModExpr = LoongArchMCExpr::create(SubExpr, VK, getContext());
 800:   Operands.push_back(LoongArchOperand::createImm(ModExpr, S, E));
```
- **EN**: The range implements or declares functions including `LoongArchAsmParser::parseOperandWithModifier`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchAsmParser::parseOperandWithModifier` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 801-840 / 第 801-840 行
```cpp
 801:   return ParseStatus::Success;
 802: }
 803: 
 804: ParseStatus LoongArchAsmParser::parseSImm26Operand(OperandVector &Operands) {
 805:   SMLoc S = getLoc();
 806:   const MCExpr *Res;
 807: 
 808:   if (getLexer().getKind() == AsmToken::Percent)
 809:     return parseOperandWithModifier(Operands);
 810: 
 811:   if (getLexer().getKind() != AsmToken::Identifier)
 812:     return ParseStatus::NoMatch;
 813: 
 814:   StringRef Identifier;
 815:   if (getParser().parseIdentifier(Identifier))
 816:     return ParseStatus::Failure;
 817: 
 818:   SMLoc E = SMLoc::getFromPointer(S.getPointer() + Identifier.size());
 819: 
 820:   MCSymbol *Sym = getContext().getOrCreateSymbol(Identifier);
 821:   Res = MCSymbolRefExpr::create(Sym, getContext());
 822:   Operands.push_back(LoongArchOperand::createImm(Res, S, E));
 823:   return ParseStatus::Success;
 824: }
 825: 
 826: ParseStatus LoongArchAsmParser::parseAtomicMemOp(OperandVector &Operands) {
 827:   // Parse "$r*".
 828:   if (!parseRegister(Operands).isSuccess())
 829:     return ParseStatus::NoMatch;
 830: 
 831:   // If there is a next operand and it is 0, ignore it. Otherwise print a
 832:   // diagnostic message.
 833:   if (parseOptionalToken(AsmToken::Comma)) {
 834:     int64_t ImmVal;
 835:     SMLoc ImmStart = getLoc();
 836:     if (getParser().parseIntToken(ImmVal, "expected optional integer offset"))
 837:       return ParseStatus::Failure;
 838:     if (ImmVal)
 839:       return Error(ImmStart, "optional integer offset must be 0");
 840:   }
```
- **EN**: The range implements or declares functions including `LoongArchAsmParser::parseSImm26Operand`, `LoongArchAsmParser::parseAtomicMemOp`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchAsmParser::parseSImm26Operand`, `LoongArchAsmParser::parseAtomicMemOp` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 841-880 / 第 841-880 行
```cpp
 841: 
 842:   return ParseStatus::Success;
 843: }
 844: /// Looks at a token type and creates the relevant operand from this
 845: /// information, adding to Operands. Return true upon an error.
 846: bool LoongArchAsmParser::parseOperand(OperandVector &Operands,
 847:                                       StringRef Mnemonic) {
 848:   // Check if the current operand has a custom associated parser, if so, try to
 849:   // custom parse the operand, or fallback to the general approach.
 850:   ParseStatus Result =
 851:       MatchOperandParserImpl(Operands, Mnemonic, /*ParseForAllFeatures=*/true);
 852:   if (Result.isSuccess())
 853:     return false;
 854:   if (Result.isFailure())
 855:     return true;
 856: 
 857:   if (parseRegister(Operands).isSuccess() ||
 858:       parseImmediate(Operands).isSuccess())
 859:     return false;
 860: 
 861:   // Finally we have exhausted all options and must declare defeat.
 862:   return Error(getLoc(), "unknown operand");
 863: }
 864: 
 865: bool LoongArchAsmParser::parseInstruction(ParseInstructionInfo &Info,
 866:                                           StringRef Name, SMLoc NameLoc,
 867:                                           OperandVector &Operands) {
 868:   // First operand in MCInst is instruction mnemonic.
 869:   Operands.push_back(LoongArchOperand::createToken(Name, NameLoc));
 870: 
 871:   // If there are no more operands, then finish.
 872:   if (parseOptionalToken(AsmToken::EndOfStatement))
 873:     return false;
 874: 
 875:   // Parse first operand.
 876:   if (parseOperand(Operands, Name))
 877:     return true;
 878: 
 879:   // Parse until end of statement, consuming commas between operands.
 880:   while (parseOptionalToken(AsmToken::Comma))
```
- **EN**: The range implements or declares functions including `LoongArchAsmParser::parseOperand`, `LoongArchAsmParser::parseInstruction`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `LoongArchAsmParser::parseOperand`, `LoongArchAsmParser::parseInstruction` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 881-920 / 第 881-920 行
```cpp
 881:     if (parseOperand(Operands, Name))
 882:       return true;
 883: 
 884:   // Parse end of statement and return successfully.
 885:   if (parseOptionalToken(AsmToken::EndOfStatement))
 886:     return false;
 887: 
 888:   SMLoc Loc = getLexer().getLoc();
 889:   getParser().eatToEndOfStatement();
 890:   return Error(Loc, "unexpected token");
 891: }
 892: 
 893: void LoongArchAsmParser::emitLAInstSeq(MCRegister DestReg, MCRegister TmpReg,
 894:                                        const MCExpr *Symbol,
 895:                                        SmallVectorImpl<Inst> &Insts,
 896:                                        SMLoc IDLoc, MCStreamer &Out,
 897:                                        bool RelaxHint) {
 898:   MCContext &Ctx = getContext();
 899:   MCSymbol *PCALabel = nullptr;
 900:   for (LoongArchAsmParser::Inst &Inst : Insts) {
 901:     unsigned Opc = Inst.Opc;
 902:     auto VK = LoongArchMCExpr::Specifier(Inst.Specifier);
 903:     const LoongArchMCExpr *LE =
 904:         LoongArchMCExpr::create(Symbol, VK, Ctx, RelaxHint);
 905:     switch (Opc) {
 906:     default:
 907:       llvm_unreachable("unexpected opcode");
 908:     case LoongArch::PCADDU12I:
 909:       PCALabel = Ctx.createNamedTempSymbol("pcadd_hi");
 910:       Out.emitLabel(PCALabel);
 911:       LLVM_FALLTHROUGH;
 912:     case LoongArch::PCALAU12I:
 913:     case LoongArch::LU12I_W:
 914:       Out.emitInstruction(MCInstBuilder(Opc).addReg(DestReg).addExpr(LE),
 915:                           getSTI());
 916:       break;
 917:     case LoongArch::ORI:
 918:     case LoongArch::ADDI_W:
 919:     case LoongArch::LD_W:
 920:     case LoongArch::LD_D: {
```
- **EN**: The range implements or declares functions including `LoongArchAsmParser::emitLAInstSeq`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchAsmParser::emitLAInstSeq` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 921-960 / 第 921-960 行
```cpp
 921:       if (VK == LoongArchMCExpr::VK_None) {
 922:         Out.emitInstruction(
 923:             MCInstBuilder(Opc).addReg(DestReg).addReg(DestReg).addImm(0),
 924:             getSTI());
 925:         continue;
 926:       } else if (VK == ELF::R_LARCH_TLS_DESC_LD) {
 927:         Out.emitInstruction(MCInstBuilder(Opc)
 928:                                 .addReg(LoongArch::R1)
 929:                                 .addReg(DestReg)
 930:                                 .addExpr(LE),
 931:                             getSTI());
 932:         continue;
 933:       } else if (VK == ELF::R_LARCH_PCADD_LO12 ||
 934:                  VK == ELF::R_LARCH_GOT_PCADD_LO12 ||
 935:                  VK == ELF::R_LARCH_TLS_IE_PCADD_LO12 ||
 936:                  VK == ELF::R_LARCH_TLS_LD_PCADD_LO12 ||
 937:                  VK == ELF::R_LARCH_TLS_GD_PCADD_LO12 ||
 938:                  VK == ELF::R_LARCH_TLS_DESC_PCADD_LO12) {
 939:         Out.emitInstruction(
 940:             MCInstBuilder(Opc).addReg(DestReg).addReg(DestReg).addExpr(
 941:                 LoongArchMCExpr::create(MCSymbolRefExpr::create(PCALabel, Ctx),
 942:                                         VK, Ctx, RelaxHint)),
 943:             getSTI());
 944:         continue;
 945:       }
 946:       Out.emitInstruction(
 947:           MCInstBuilder(Opc).addReg(DestReg).addReg(DestReg).addExpr(LE),
 948:           getSTI());
 949:       break;
 950:     }
 951:     case LoongArch::LU32I_D:
 952:       Out.emitInstruction(MCInstBuilder(Opc)
 953:                               .addReg(DestReg == TmpReg ? DestReg : TmpReg)
 954:                               .addReg(DestReg == TmpReg ? DestReg : TmpReg)
 955:                               .addExpr(LE),
 956:                           getSTI());
 957:       break;
 958:     case LoongArch::LU52I_D:
 959:       Out.emitInstruction(
 960:           MCInstBuilder(Opc).addReg(TmpReg).addReg(TmpReg).addExpr(LE),
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 961-1000 / 第 961-1000 行
```cpp
 961:           getSTI());
 962:       break;
 963:     case LoongArch::ADDI_D:
 964:       Out.emitInstruction(
 965:           MCInstBuilder(Opc)
 966:               .addReg(TmpReg)
 967:               .addReg(DestReg == TmpReg ? TmpReg : LoongArch::R0)
 968:               .addExpr(LE),
 969:           getSTI());
 970:       break;
 971:     case LoongArch::ADD_D:
 972:     case LoongArch::LDX_D:
 973:       Out.emitInstruction(
 974:           MCInstBuilder(Opc).addReg(DestReg).addReg(DestReg).addReg(TmpReg),
 975:           getSTI());
 976:       break;
 977:     case LoongArch::JIRL:
 978:       Out.emitInstruction(MCInstBuilder(Opc)
 979:                               .addReg(LoongArch::R1)
 980:                               .addReg(LoongArch::R1)
 981:                               .addExpr(LE),
 982:                           getSTI());
 983:       break;
 984:     }
 985:   }
 986: }
 987: 
 988: void LoongArchAsmParser::emitLoadAddressAbs(MCInst &Inst, SMLoc IDLoc,
 989:                                             MCStreamer &Out) {
 990:   // la.abs $rd, sym
 991:   // expands to:
 992:   //   lu12i.w $rd, %abs_hi20(sym)
 993:   //   ori     $rd, $rd, %abs_lo12(sym)
 994:   //
 995:   // for 64bit appends:
 996:   //   lu32i.d $rd, %abs64_lo20(sym)
 997:   //   lu52i.d $rd, $rd, %abs64_hi12(sym)
 998:   MCRegister DestReg = Inst.getOperand(0).getReg();
 999:   const MCExpr *Symbol = Inst.getOpcode() == LoongArch::PseudoLA_ABS
1000:                              ? Inst.getOperand(1).getExpr()
```
- **EN**: The range implements or declares functions including `MCInstBuilder`, `LoongArchAsmParser::emitLoadAddressAbs`.
- **CN**: 这一段实现或声明了 `MCInstBuilder`, `LoongArchAsmParser::emitLoadAddressAbs` 等函数。

### Lines 1001-1040 / 第 1001-1040 行
```cpp
1001:                              : Inst.getOperand(2).getExpr();
1002:   InstSeq Insts;
1003: 
1004:   // To distinguish between la.abs and %abs_hi20, la.abs will generate
1005:   // R_LARCH_MARK_LA and R_LARCH_ABS_HI20 relocations.
1006:   Insts.push_back(
1007:       LoongArchAsmParser::Inst(LoongArch::LU12I_W, ELF::R_LARCH_MARK_LA));
1008:   Insts.push_back(
1009:       LoongArchAsmParser::Inst(LoongArch::ORI, ELF::R_LARCH_ABS_LO12));
1010: 
1011:   if (is64Bit()) {
1012:     Insts.push_back(
1013:         LoongArchAsmParser::Inst(LoongArch::LU32I_D, ELF::R_LARCH_ABS64_LO20));
1014:     Insts.push_back(
1015:         LoongArchAsmParser::Inst(LoongArch::LU52I_D, ELF::R_LARCH_ABS64_HI12));
1016:   }
1017: 
1018:   emitLAInstSeq(DestReg, DestReg, Symbol, Insts, IDLoc, Out);
1019: }
1020: 
1021: void LoongArchAsmParser::emitLoadAddressPcrel(MCInst &Inst, SMLoc IDLoc,
1022:                                               MCStreamer &Out) {
1023:   // la.pcrel $rd, sym
1024:   //
1025:   // for la32r expands to:
1026:   // .Lpcadd_hi:
1027:   //   pcaddu12i $rd, %pcadd_hi20(sym)
1028:   //   addi.w    $rd, rd, %pcadd_lo12(.Lpcadd_hi)
1029:   //
1030:   // for la32s and la64 expands to:
1031:   //   pcalau12i $rd, %pc_hi20(sym)
1032:   //   addi.w/d  $rd, rd, %pc_lo12(sym)
1033:   MCRegister DestReg = Inst.getOperand(0).getReg();
1034:   const MCExpr *Symbol = Inst.getOperand(1).getExpr();
1035:   InstSeq Insts;
1036:   unsigned PCAI = has32S() ? LoongArch::PCALAU12I : LoongArch::PCADDU12I;
1037:   unsigned ADDI = is64Bit() ? LoongArch::ADDI_D : LoongArch::ADDI_W;
1038:   unsigned PCAIRel =
1039:       has32S() ? ELF::R_LARCH_PCALA_HI20 : ELF::R_LARCH_PCADD_HI20;
1040:   unsigned ADDIRel =
```
- **EN**: The range implements or declares functions including `LoongArchAsmParser::emitLoadAddressPcrel`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchAsmParser::emitLoadAddressPcrel` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1041-1080 / 第 1041-1080 行
```cpp
1041:       has32S() ? ELF::R_LARCH_PCALA_LO12 : ELF::R_LARCH_PCADD_LO12;
1042: 
1043:   Insts.push_back(LoongArchAsmParser::Inst(PCAI, PCAIRel));
1044:   Insts.push_back(LoongArchAsmParser::Inst(ADDI, ADDIRel));
1045: 
1046:   emitLAInstSeq(DestReg, DestReg, Symbol, Insts, IDLoc, Out,
1047:                 /*RelaxHint=*/true);
1048: }
1049: 
1050: void LoongArchAsmParser::emitLoadAddressPcrelLarge(MCInst &Inst, SMLoc IDLoc,
1051:                                                    MCStreamer &Out) {
1052:   // la.pcrel $rd, $rj, sym
1053:   // expands to:
1054:   //   pcalau12i $rd, %pc_hi20(sym)
1055:   //   addi.d    $rj, $r0, %pc_lo12(sym)
1056:   //   lu32i.d   $rj, %pc64_lo20(sym)
1057:   //   lu52i.d   $rj, $rj, %pc64_hi12(sym)
1058:   //   add.d     $rd, $rd, $rj
1059:   MCRegister DestReg = Inst.getOperand(0).getReg();
1060:   MCRegister TmpReg = Inst.getOperand(1).getReg();
1061:   const MCExpr *Symbol = Inst.getOperand(2).getExpr();
1062:   InstSeq Insts;
1063: 
1064:   Insts.push_back(
1065:       LoongArchAsmParser::Inst(LoongArch::PCALAU12I, ELF::R_LARCH_PCALA_HI20));
1066:   Insts.push_back(
1067:       LoongArchAsmParser::Inst(LoongArch::ADDI_D, ELF::R_LARCH_PCALA_LO12));
1068:   Insts.push_back(
1069:       LoongArchAsmParser::Inst(LoongArch::LU32I_D, ELF::R_LARCH_PCALA64_LO20));
1070:   Insts.push_back(
1071:       LoongArchAsmParser::Inst(LoongArch::LU52I_D, ELF::R_LARCH_PCALA64_HI12));
1072:   Insts.push_back(LoongArchAsmParser::Inst(LoongArch::ADD_D));
1073: 
1074:   emitLAInstSeq(DestReg, TmpReg, Symbol, Insts, IDLoc, Out);
1075: }
1076: 
1077: void LoongArchAsmParser::emitLoadAddressGot(MCInst &Inst, SMLoc IDLoc,
1078:                                             MCStreamer &Out) {
1079:   // la.got $rd, sym
1080:   MCRegister DestReg = Inst.getOperand(0).getReg();
```
- **EN**: The range implements or declares functions including `LoongArchAsmParser::emitLoadAddressPcrelLarge`, `LoongArchAsmParser::emitLoadAddressGot`.
- **CN**: 这一段实现或声明了 `LoongArchAsmParser::emitLoadAddressPcrelLarge`, `LoongArchAsmParser::emitLoadAddressGot` 等函数。

### Lines 1081-1120 / 第 1081-1120 行
```cpp
1081:   const MCExpr *Symbol = Inst.getOperand(1).getExpr();
1082:   InstSeq Insts;
1083:   unsigned PCAI = has32S() ? LoongArch::PCALAU12I : LoongArch::PCADDU12I;
1084:   unsigned LD = is64Bit() ? LoongArch::LD_D : LoongArch::LD_W;
1085:   unsigned PCAIRel =
1086:       has32S() ? ELF::R_LARCH_GOT_PC_HI20 : ELF::R_LARCH_GOT_PCADD_HI20;
1087:   unsigned LDRel =
1088:       has32S() ? ELF::R_LARCH_GOT_PC_LO12 : ELF::R_LARCH_GOT_PCADD_LO12;
1089: 
1090:   if (getSTI().hasFeature(LoongArch::LaGlobalWithAbs)) {
1091:     // with feature: +la-glabal-with-abs
1092:     // for 32bit:
1093:     //   lu12i.w $rd, %got_hi20(sym)
1094:     //   ori     $rd, $rd, %got_lo12(sym)
1095:     //   ld.w    $rd, $rd, 0
1096:     //
1097:     // for 64bit:
1098:     //   lu12i.w $rd, %got_hi20(sym)
1099:     //   ori     $rd, $rd, %got_lo12(sym)
1100:     //   lu32i.d $rd, %got64_lo20(sym)
1101:     //   lu52i.d $rd, $rd, %got64_hi12(sym)
1102:     //   ld.d    $rd, $rd, 0
1103:     Insts.push_back(
1104:         LoongArchAsmParser::Inst(LoongArch::LU12I_W, ELF::R_LARCH_GOT_HI20));
1105:     Insts.push_back(
1106:         LoongArchAsmParser::Inst(LoongArch::ORI, ELF::R_LARCH_GOT_LO12));
1107: 
1108:     if (is64Bit()) {
1109:       Insts.push_back(LoongArchAsmParser::Inst(LoongArch::LU32I_D,
1110:                                                ELF::R_LARCH_GOT64_LO20));
1111:       Insts.push_back(LoongArchAsmParser::Inst(LoongArch::LU52I_D,
1112:                                                ELF::R_LARCH_GOT64_HI12));
1113:     }
1114:     Insts.push_back(LoongArchAsmParser::Inst(LD));
1115:     emitLAInstSeq(DestReg, DestReg, Symbol, Insts, IDLoc, Out);
1116:     return;
1117:   }
1118:   // for la32r expands to:
1119:   // .Lpcadd_hi:
1120:   //   pcaddu12i $rd, %got_pcadd_hi20(sym)
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1121-1160 / 第 1121-1160 行
```cpp
1121:   //   ld.w      $rd, $rd, %got_pcadd_lo12(.Lpcadd_hi)
1122:   //
1123:   // for la32s and la64 expands to:
1124:   //   pcalau12i $rd, %got_pc_hi20(sym)
1125:   //   ld.w/d    $rd, $rd, %got_pc_lo12(sym)
1126:   Insts.push_back(LoongArchAsmParser::Inst(PCAI, PCAIRel));
1127:   Insts.push_back(LoongArchAsmParser::Inst(LD, LDRel));
1128: 
1129:   emitLAInstSeq(DestReg, DestReg, Symbol, Insts, IDLoc, Out,
1130:                 /*RelaxHint=*/true);
1131: }
1132: 
1133: void LoongArchAsmParser::emitLoadAddressGotLarge(MCInst &Inst, SMLoc IDLoc,
1134:                                                  MCStreamer &Out) {
1135:   // la.got $rd, $rj, sym
1136:   // expands to:
1137:   //   pcalau12i $rd, %got_pc_hi20(sym)
1138:   //   addi.d    $rj, $r0, %got_pc_lo12(sym)
1139:   //   lu32i.d   $rj, %got64_pc_lo20(sym)
1140:   //   lu52i.d   $rj, $rj, %got64_pc_hi12(sym)
1141:   //   ldx.d     $rd, $rd, $rj
1142:   MCRegister DestReg = Inst.getOperand(0).getReg();
1143:   MCRegister TmpReg = Inst.getOperand(1).getReg();
1144:   const MCExpr *Symbol = Inst.getOperand(2).getExpr();
1145:   InstSeq Insts;
1146: 
1147:   Insts.push_back(
1148:       LoongArchAsmParser::Inst(LoongArch::PCALAU12I, ELF::R_LARCH_GOT_PC_HI20));
1149:   Insts.push_back(
1150:       LoongArchAsmParser::Inst(LoongArch::ADDI_D, ELF::R_LARCH_GOT_PC_LO12));
1151:   Insts.push_back(
1152:       LoongArchAsmParser::Inst(LoongArch::LU32I_D, ELF::R_LARCH_GOT64_PC_LO20));
1153:   Insts.push_back(
1154:       LoongArchAsmParser::Inst(LoongArch::LU52I_D, ELF::R_LARCH_GOT64_PC_HI12));
1155:   Insts.push_back(LoongArchAsmParser::Inst(LoongArch::LDX_D));
1156: 
1157:   emitLAInstSeq(DestReg, TmpReg, Symbol, Insts, IDLoc, Out);
1158: }
1159: 
1160: void LoongArchAsmParser::emitLoadAddressTLSLE(MCInst &Inst, SMLoc IDLoc,
```
- **EN**: The range implements or declares functions including `LoongArchAsmParser::emitLoadAddressGotLarge`.
- **CN**: 这一段实现或声明了 `LoongArchAsmParser::emitLoadAddressGotLarge` 等函数。

### Lines 1161-1200 / 第 1161-1200 行
```cpp
1161:                                               MCStreamer &Out) {
1162:   // la.tls.le $rd, sym
1163:   // expands to:
1164:   //   lu12i.w $rd, %le_hi20(sym)
1165:   //   ori     $rd, $rd, %le_lo12(sym)
1166:   MCRegister DestReg = Inst.getOperand(0).getReg();
1167:   const MCExpr *Symbol = Inst.getOperand(1).getExpr();
1168:   InstSeq Insts;
1169: 
1170:   Insts.push_back(
1171:       LoongArchAsmParser::Inst(LoongArch::LU12I_W, ELF::R_LARCH_TLS_LE_HI20));
1172:   Insts.push_back(
1173:       LoongArchAsmParser::Inst(LoongArch::ORI, ELF::R_LARCH_TLS_LE_LO12));
1174: 
1175:   emitLAInstSeq(DestReg, DestReg, Symbol, Insts, IDLoc, Out);
1176: }
1177: 
1178: void LoongArchAsmParser::emitLoadAddressTLSIE(MCInst &Inst, SMLoc IDLoc,
1179:                                               MCStreamer &Out) {
1180:   // la.tls.ie $rd, sym
1181:   MCRegister DestReg = Inst.getOperand(0).getReg();
1182:   const MCExpr *Symbol = Inst.getOperand(1).getExpr();
1183:   InstSeq Insts;
1184:   unsigned PCAI = has32S() ? LoongArch::PCALAU12I : LoongArch::PCADDU12I;
1185:   unsigned LD = is64Bit() ? LoongArch::LD_D : LoongArch::LD_W;
1186:   unsigned PCAIRel =
1187:       has32S() ? ELF::R_LARCH_TLS_IE_PC_HI20 : ELF::R_LARCH_TLS_IE_PCADD_HI20;
1188:   unsigned LDRel =
1189:       has32S() ? ELF::R_LARCH_TLS_IE_PC_LO12 : ELF::R_LARCH_TLS_IE_PCADD_LO12;
1190: 
1191:   if (getSTI().hasFeature(LoongArch::LaGlobalWithAbs)) {
1192:     // with feature: +la-glabal-with-abs
1193:     // for 32bit:
1194:     //   lu12i.w $rd, %ie_hi20(sym)
1195:     //   ori     $rd, $rd, %ie_lo12(sym)
1196:     //   ld.w    $rd, $rd, 0
1197:     //
1198:     // for 64bit:
1199:     //   lu12i.w $rd, %ie_hi20(sym)
1200:     //   ori     $rd, $rd, %ie_lo12(sym)
```
- **EN**: The range implements or declares functions including `LoongArchAsmParser::emitLoadAddressTLSIE`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchAsmParser::emitLoadAddressTLSIE` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1201-1240 / 第 1201-1240 行
```cpp
1201:     //   lu32i.d $rd, %ie64_lo20(sym)
1202:     //   lu52i.d $rd, $rd, %ie64_hi12(sym)
1203:     //   ld.d    $rd, $rd, 0
1204:     Insts.push_back(
1205:         LoongArchAsmParser::Inst(LoongArch::LU12I_W, ELF::R_LARCH_TLS_IE_HI20));
1206:     Insts.push_back(
1207:         LoongArchAsmParser::Inst(LoongArch::ORI, ELF::R_LARCH_TLS_IE_LO12));
1208: 
1209:     if (is64Bit()) {
1210:       Insts.push_back(LoongArchAsmParser::Inst(LoongArch::LU32I_D,
1211:                                                ELF::R_LARCH_TLS_IE64_LO20));
1212:       Insts.push_back(LoongArchAsmParser::Inst(LoongArch::LU52I_D,
1213:                                                ELF::R_LARCH_TLS_IE64_HI12));
1214:     }
1215:     Insts.push_back(LoongArchAsmParser::Inst(LD));
1216:     emitLAInstSeq(DestReg, DestReg, Symbol, Insts, IDLoc, Out);
1217:     return;
1218:   }
1219: 
1220:   // for la32r expands to:
1221:   // .Lpcadd_hi:
1222:   //   pcaddu12i $rd, %ie_pcadd_hi20(sym)
1223:   //   ld.w      $rd, $rd, %ie_pcadd_lo12(.Lpcadd_hi)
1224:   //
1225:   // for la32s and la64 expands to:
1226:   //   pcalau12i $rd, %ie_pc_hi20(sym)
1227:   //   ld.w/d    $rd, $rd, %ie_pc_lo12(sym)
1228:   Insts.push_back(LoongArchAsmParser::Inst(PCAI, PCAIRel));
1229:   Insts.push_back(LoongArchAsmParser::Inst(LD, LDRel));
1230: 
1231:   emitLAInstSeq(DestReg, DestReg, Symbol, Insts, IDLoc, Out,
1232:                 /*RelaxHint=*/true);
1233: }
1234: 
1235: void LoongArchAsmParser::emitLoadAddressTLSIELarge(MCInst &Inst, SMLoc IDLoc,
1236:                                                    MCStreamer &Out) {
1237:   // la.tls.ie $rd, $rj, sym
1238:   // expands to:
1239:   //   pcalau12i $rd, %ie_pc_hi20(sym)
1240:   //   addi.d    $rj, $r0, %ie_pc_lo12(sym)
```
- **EN**: The range implements or declares functions including `LoongArchAsmParser::emitLoadAddressTLSIELarge`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchAsmParser::emitLoadAddressTLSIELarge` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1241-1280 / 第 1241-1280 行
```cpp
1241:   //   lu32i.d   $rj, %ie64_pc_lo20(sym)
1242:   //   lu52i.d   $rj, $rj, %ie64_pc_hi12(sym)
1243:   //   ldx.d     $rd, $rd, $rj
1244:   MCRegister DestReg = Inst.getOperand(0).getReg();
1245:   MCRegister TmpReg = Inst.getOperand(1).getReg();
1246:   const MCExpr *Symbol = Inst.getOperand(2).getExpr();
1247:   InstSeq Insts;
1248: 
1249:   Insts.push_back(LoongArchAsmParser::Inst(LoongArch::PCALAU12I,
1250:                                            ELF::R_LARCH_TLS_IE_PC_HI20));
1251:   Insts.push_back(
1252:       LoongArchAsmParser::Inst(LoongArch::ADDI_D, ELF::R_LARCH_TLS_IE_PC_LO12));
1253:   Insts.push_back(LoongArchAsmParser::Inst(LoongArch::LU32I_D,
1254:                                            ELF::R_LARCH_TLS_IE64_PC_LO20));
1255:   Insts.push_back(LoongArchAsmParser::Inst(LoongArch::LU52I_D,
1256:                                            ELF::R_LARCH_TLS_IE64_PC_HI12));
1257:   Insts.push_back(LoongArchAsmParser::Inst(LoongArch::LDX_D));
1258: 
1259:   emitLAInstSeq(DestReg, TmpReg, Symbol, Insts, IDLoc, Out);
1260: }
1261: 
1262: void LoongArchAsmParser::emitLoadAddressTLSLD(MCInst &Inst, SMLoc IDLoc,
1263:                                               MCStreamer &Out) {
1264:   // la.tls.ld $rd, sym
1265:   MCRegister DestReg = Inst.getOperand(0).getReg();
1266:   const MCExpr *Symbol = Inst.getOperand(1).getExpr();
1267:   InstSeq Insts;
1268:   unsigned PCAI = has32S() ? LoongArch::PCALAU12I : LoongArch::PCADDU12I;
1269:   unsigned ADDI = is64Bit() ? LoongArch::ADDI_D : LoongArch::ADDI_W;
1270:   unsigned PCAIRel =
1271:       has32S() ? ELF::R_LARCH_TLS_LD_PC_HI20 : ELF::R_LARCH_TLS_LD_PCADD_HI20;
1272:   unsigned ADDIRel =
1273:       has32S() ? ELF::R_LARCH_GOT_PC_LO12 : ELF::R_LARCH_TLS_LD_PCADD_LO12;
1274: 
1275:   if (getSTI().hasFeature(LoongArch::LaGlobalWithAbs)) {
1276:     // with feature: +la-glabal-with-abs
1277:     // for 32bit:
1278:     //   lu12i.w $rd, %ld_hi20(sym)
1279:     //   ori     $rd, $rd, %got_lo12(sym)
1280:     //
```
- **EN**: The range implements or declares functions including `LoongArchAsmParser::emitLoadAddressTLSLD`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchAsmParser::emitLoadAddressTLSLD` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1281-1320 / 第 1281-1320 行
```cpp
1281:     // for 64bit:
1282:     //   lu12i.w $rd, %ld_hi20(sym)
1283:     //   ori     $rd, $rd, %got_lo12(sym)
1284:     //   lu32i.d $rd, %got64_lo20(sym)
1285:     //   lu52i.d $rd, $rd, %got64_hi12(sym)
1286:     Insts.push_back(
1287:         LoongArchAsmParser::Inst(LoongArch::LU12I_W, ELF::R_LARCH_TLS_LD_HI20));
1288:     Insts.push_back(
1289:         LoongArchAsmParser::Inst(LoongArch::ORI, ELF::R_LARCH_GOT_LO12));
1290: 
1291:     if (is64Bit()) {
1292:       Insts.push_back(LoongArchAsmParser::Inst(LoongArch::LU32I_D,
1293:                                                ELF::R_LARCH_GOT64_LO20));
1294:       Insts.push_back(LoongArchAsmParser::Inst(LoongArch::LU52I_D,
1295:                                                ELF::R_LARCH_GOT64_HI12));
1296:     }
1297:     emitLAInstSeq(DestReg, DestReg, Symbol, Insts, IDLoc, Out);
1298:     return;
1299:   }
1300: 
1301:   // for la32r expands to:
1302:   // .Lpcadd_hi:
1303:   //   pcaddu12i $rd, %ld_pcadd_hi20(sym)
1304:   //   addi.w    $rd, $rd, %ld_pcadd_lo12(.Lpcadd_hi)
1305:   //
1306:   // for la32s and la64 expands to:
1307:   //   pcalau12i $rd, %ld_pc_hi20(sym)
1308:   //   addi.w/d  $rd, $rd, %got_pc_lo12(sym)
1309:   Insts.push_back(LoongArchAsmParser::Inst(PCAI, PCAIRel));
1310:   Insts.push_back(LoongArchAsmParser::Inst(ADDI, ADDIRel));
1311: 
1312:   emitLAInstSeq(DestReg, DestReg, Symbol, Insts, IDLoc, Out,
1313:                 /*RelaxHint=*/true);
1314: }
1315: 
1316: void LoongArchAsmParser::emitLoadAddressTLSLDLarge(MCInst &Inst, SMLoc IDLoc,
1317:                                                    MCStreamer &Out) {
1318:   // la.tls.ld $rd, $rj, sym
1319:   // expands to:
1320:   //   pcalau12i $rd, %ld_pc_hi20(sym)
```
- **EN**: The range implements or declares functions including `LoongArchAsmParser::emitLoadAddressTLSLDLarge`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchAsmParser::emitLoadAddressTLSLDLarge` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1321-1360 / 第 1321-1360 行
```cpp
1321:   //   addi.d    $rj, $r0, %got_pc_lo12(sym)
1322:   //   lu32i.d   $rj, %got64_pc_lo20(sym)
1323:   //   lu52i.d   $rj, $rj, %got64_pc_hi12(sym)
1324:   //   add.d     $rd, $rd, $rj
1325:   MCRegister DestReg = Inst.getOperand(0).getReg();
1326:   MCRegister TmpReg = Inst.getOperand(1).getReg();
1327:   const MCExpr *Symbol = Inst.getOperand(2).getExpr();
1328:   InstSeq Insts;
1329: 
1330:   Insts.push_back(LoongArchAsmParser::Inst(LoongArch::PCALAU12I,
1331:                                            ELF::R_LARCH_TLS_LD_PC_HI20));
1332:   Insts.push_back(
1333:       LoongArchAsmParser::Inst(LoongArch::ADDI_D, ELF::R_LARCH_GOT_PC_LO12));
1334:   Insts.push_back(
1335:       LoongArchAsmParser::Inst(LoongArch::LU32I_D, ELF::R_LARCH_GOT64_PC_LO20));
1336:   Insts.push_back(
1337:       LoongArchAsmParser::Inst(LoongArch::LU52I_D, ELF::R_LARCH_GOT64_PC_HI12));
1338:   Insts.push_back(LoongArchAsmParser::Inst(LoongArch::ADD_D));
1339: 
1340:   emitLAInstSeq(DestReg, TmpReg, Symbol, Insts, IDLoc, Out);
1341: }
1342: 
1343: void LoongArchAsmParser::emitLoadAddressTLSGD(MCInst &Inst, SMLoc IDLoc,
1344:                                               MCStreamer &Out) {
1345:   // la.tls.gd $rd, sym
1346:   MCRegister DestReg = Inst.getOperand(0).getReg();
1347:   const MCExpr *Symbol = Inst.getOperand(1).getExpr();
1348:   InstSeq Insts;
1349:   unsigned PCAI = has32S() ? LoongArch::PCALAU12I : LoongArch::PCADDU12I;
1350:   unsigned ADDI = is64Bit() ? LoongArch::ADDI_D : LoongArch::ADDI_W;
1351:   unsigned PCAIRel =
1352:       has32S() ? ELF::R_LARCH_TLS_GD_PC_HI20 : ELF::R_LARCH_TLS_GD_PCADD_HI20;
1353:   unsigned ADDIRel =
1354:       has32S() ? ELF::R_LARCH_GOT_PC_LO12 : ELF::R_LARCH_TLS_GD_PCADD_LO12;
1355: 
1356:   if (getSTI().hasFeature(LoongArch::LaGlobalWithAbs)) {
1357:     // with feature: +la-glabal-with-abs
1358:     // for 32bit:
1359:     //   lu12i.w $rd, %gd_hi20(sym)
1360:     //   ori     $rd, $rd, %got_lo12(sym)
```
- **EN**: The range implements or declares functions including `LoongArchAsmParser::emitLoadAddressTLSGD`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchAsmParser::emitLoadAddressTLSGD` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1361-1400 / 第 1361-1400 行
```cpp
1361:     //
1362:     // for 64bit:
1363:     //   lu12i.w $rd, %gd_hi20(sym)
1364:     //   ori     $rd, $rd, %got_lo12(sym)
1365:     //   lu32i.d $rd, %got64_lo20(sym)
1366:     //   lu52i.d $rd, $rd, %got64_hi12(sym)
1367:     Insts.push_back(
1368:         LoongArchAsmParser::Inst(LoongArch::LU12I_W, ELF::R_LARCH_TLS_GD_HI20));
1369:     Insts.push_back(
1370:         LoongArchAsmParser::Inst(LoongArch::ORI, ELF::R_LARCH_GOT_LO12));
1371: 
1372:     if (is64Bit()) {
1373:       Insts.push_back(LoongArchAsmParser::Inst(LoongArch::LU32I_D,
1374:                                                ELF::R_LARCH_GOT64_LO20));
1375:       Insts.push_back(LoongArchAsmParser::Inst(LoongArch::LU52I_D,
1376:                                                ELF::R_LARCH_GOT64_HI12));
1377:     }
1378:     emitLAInstSeq(DestReg, DestReg, Symbol, Insts, IDLoc, Out);
1379:     return;
1380:   }
1381: 
1382:   // for la32r expands to:
1383:   // .Lpcadd_hi:
1384:   //   pcaddu12i $rd, %gd_pcadd_hi20(sym)
1385:   //   addi.w    $rd, $rd, %gd_pcadd_lo12(.Lpcadd_hi)
1386:   //
1387:   // for la32s and la64 expands to:
1388:   //   pcalau12i $rd, %gd_pc_hi20(sym)
1389:   //   addi.w/d  $rd, $rd, %got_pc_lo12(sym)
1390:   Insts.push_back(LoongArchAsmParser::Inst(PCAI, PCAIRel));
1391:   Insts.push_back(LoongArchAsmParser::Inst(ADDI, ADDIRel));
1392: 
1393:   emitLAInstSeq(DestReg, DestReg, Symbol, Insts, IDLoc, Out,
1394:                 /*RelaxHint=*/true);
1395: }
1396: 
1397: void LoongArchAsmParser::emitLoadAddressTLSGDLarge(MCInst &Inst, SMLoc IDLoc,
1398:                                                    MCStreamer &Out) {
1399:   // la.tls.gd $rd, $rj, sym
1400:   // expands to:
```
- **EN**: The range implements or declares functions including `LoongArchAsmParser::emitLoadAddressTLSGDLarge`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchAsmParser::emitLoadAddressTLSGDLarge` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1401-1440 / 第 1401-1440 行
```cpp
1401:   //   pcalau12i $rd, %gd_pc_hi20(sym)
1402:   //   addi.d    $rj, $r0, %got_pc_lo12(sym)
1403:   //   lu32i.d   $rj, %got64_pc_lo20(sym)
1404:   //   lu52i.d   $rj, $rj, %got64_pc_hi12(sym)
1405:   //   add.d     $rd, $rd, $rj
1406:   MCRegister DestReg = Inst.getOperand(0).getReg();
1407:   MCRegister TmpReg = Inst.getOperand(1).getReg();
1408:   const MCExpr *Symbol = Inst.getOperand(2).getExpr();
1409:   InstSeq Insts;
1410: 
1411:   Insts.push_back(LoongArchAsmParser::Inst(LoongArch::PCALAU12I,
1412:                                            ELF::R_LARCH_TLS_GD_PC_HI20));
1413:   Insts.push_back(
1414:       LoongArchAsmParser::Inst(LoongArch::ADDI_D, ELF::R_LARCH_GOT_PC_LO12));
1415:   Insts.push_back(
1416:       LoongArchAsmParser::Inst(LoongArch::LU32I_D, ELF::R_LARCH_GOT64_PC_LO20));
1417:   Insts.push_back(
1418:       LoongArchAsmParser::Inst(LoongArch::LU52I_D, ELF::R_LARCH_GOT64_PC_HI12));
1419:   Insts.push_back(LoongArchAsmParser::Inst(LoongArch::ADD_D));
1420: 
1421:   emitLAInstSeq(DestReg, TmpReg, Symbol, Insts, IDLoc, Out);
1422: }
1423: 
1424: void LoongArchAsmParser::emitLoadAddressTLSDesc(MCInst &Inst, SMLoc IDLoc,
1425:                                                 MCStreamer &Out) {
1426:   // la.tls.desc $rd, sym
1427:   MCRegister DestReg = Inst.getOperand(0).getReg();
1428:   const MCExpr *Symbol = Inst.getOperand(1).getExpr();
1429:   unsigned PCAI = has32S() ? LoongArch::PCALAU12I : LoongArch::PCADDU12I;
1430:   unsigned ADDI = is64Bit() ? LoongArch::ADDI_D : LoongArch::ADDI_W;
1431:   unsigned LD = is64Bit() ? LoongArch::LD_D : LoongArch::LD_W;
1432:   unsigned PCAIRel = has32S() ? ELF::R_LARCH_TLS_DESC_PC_HI20
1433:                               : ELF::R_LARCH_TLS_DESC_PCADD_HI20;
1434:   unsigned ADDIRel = has32S() ? ELF::R_LARCH_TLS_DESC_PC_LO12
1435:                               : ELF::R_LARCH_TLS_DESC_PCADD_LO12;
1436:   InstSeq Insts;
1437: 
1438:   if (getSTI().hasFeature(LoongArch::LaGlobalWithAbs)) {
1439:     // with feature: +la-glabal-with-abs
1440:     // for la32 expands to:
```
- **EN**: The range implements or declares functions including `LoongArchAsmParser::emitLoadAddressTLSDesc`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchAsmParser::emitLoadAddressTLSDesc` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1441-1480 / 第 1441-1480 行
```cpp
1441:     //   lu12i.w $rd, %desc_hi20(sym)
1442:     //   ori     $rd, $rd, %desc_lo12(sym)
1443:     //   ld.w    $ra, $rd, %desc_ld(sym)
1444:     //   jirl    $ra, $ra, %desc_call(sym)
1445:     //
1446:     // for la64 expands to:
1447:     //   lu12i.w $rd, %desc_hi20(sym)
1448:     //   ori     $rd, $rd, %desc_lo12(sym)
1449:     //   lu32i.d $rd, %desc64_lo20(sym)
1450:     //   lu52i.d $rd, $rd, %desc64_hi12(sym)
1451:     //   ld.d    $ra, $rd, %desc_ld(sym)
1452:     //   jirl    $ra, $ra, %desc_call(sym)
1453:     Insts.push_back(LoongArchAsmParser::Inst(LoongArch::LU12I_W,
1454:                                              ELF::R_LARCH_TLS_DESC_HI20));
1455:     Insts.push_back(
1456:         LoongArchAsmParser::Inst(LoongArch::ORI, ELF::R_LARCH_TLS_DESC_LO12));
1457: 
1458:     if (is64Bit()) {
1459:       Insts.push_back(LoongArchAsmParser::Inst(LoongArch::LU32I_D,
1460:                                                ELF::R_LARCH_TLS_DESC64_LO20));
1461:       Insts.push_back(LoongArchAsmParser::Inst(LoongArch::LU52I_D,
1462:                                                ELF::R_LARCH_TLS_DESC64_HI12));
1463:     }
1464: 
1465:     Insts.push_back(LoongArchAsmParser::Inst(LD, ELF::R_LARCH_TLS_DESC_LD));
1466:     Insts.push_back(
1467:         LoongArchAsmParser::Inst(LoongArch::JIRL, ELF::R_LARCH_TLS_DESC_CALL));
1468: 
1469:     emitLAInstSeq(DestReg, DestReg, Symbol, Insts, IDLoc, Out);
1470:     return;
1471:   }
1472: 
1473:   // for la32r expands to:
1474:   // .Lpcadd_hi:
1475:   //   pcaddu12i $rd, %desc_pcadd_hi20(sym)
1476:   //   addi.w    $rd, $rd, %desc_pcadd_lo12(.Lpcadd_hi)
1477:   //   ld.w      $ra, $rd, %desc_ld(sym)
1478:   //   jirl      $ra, $ra, %desc_call(sym)
1479:   //
1480:   // for la32s and la64 expands to:
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1481-1520 / 第 1481-1520 行
```cpp
1481:   //   pcalau12i $rd, %desc_pc_hi20(sym)
1482:   //   addi.w/d  $rd, $rd, %desc_pc_lo12(sym)
1483:   //   ld.w/d    $ra, $rd, %desc_ld(sym)
1484:   //   jirl      $ra, $ra, %desc_call(sym)
1485:   Insts.push_back(LoongArchAsmParser::Inst(PCAI, PCAIRel));
1486:   Insts.push_back(LoongArchAsmParser::Inst(ADDI, ADDIRel));
1487:   Insts.push_back(LoongArchAsmParser::Inst(LD, ELF::R_LARCH_TLS_DESC_LD));
1488:   Insts.push_back(
1489:       LoongArchAsmParser::Inst(LoongArch::JIRL, ELF::R_LARCH_TLS_DESC_CALL));
1490: 
1491:   emitLAInstSeq(DestReg, DestReg, Symbol, Insts, IDLoc, Out,
1492:                 /*RelaxHint=*/true);
1493: }
1494: 
1495: void LoongArchAsmParser::emitLoadAddressTLSDescLarge(MCInst &Inst, SMLoc IDLoc,
1496:                                                      MCStreamer &Out) {
1497:   // la.tls.desc $rd, $rj, sym
1498:   // expands to:
1499:   //   pcalau12i $rd, %desc_pc_hi20(sym)
1500:   //   addi.d    $rj, $r0, %desc_pc_lo12(sym)
1501:   //   lu32i.d   $rj, %desc64_pc_lo20(sym)
1502:   //   lu52i.d   $rj, $rj, %desc64_pc_hi12(sym)
1503:   //   add.d     $rd, $rd, $rj
1504:   //   ld.w/d    $ra, $rd, %desc_ld(sym)
1505:   //   jirl      $ra, $ra, %desc_call(sym)
1506:   MCRegister DestReg = Inst.getOperand(0).getReg();
1507:   MCRegister TmpReg = Inst.getOperand(1).getReg();
1508:   const MCExpr *Symbol = Inst.getOperand(2).getExpr();
1509:   InstSeq Insts;
1510: 
1511:   Insts.push_back(LoongArchAsmParser::Inst(LoongArch::PCALAU12I,
1512:                                            ELF::R_LARCH_TLS_DESC_PC_HI20));
1513:   Insts.push_back(LoongArchAsmParser::Inst(LoongArch::ADDI_D,
1514:                                            ELF::R_LARCH_TLS_DESC_PC_LO12));
1515:   Insts.push_back(LoongArchAsmParser::Inst(LoongArch::LU32I_D,
1516:                                            ELF::R_LARCH_TLS_DESC64_PC_LO20));
1517:   Insts.push_back(LoongArchAsmParser::Inst(LoongArch::LU52I_D,
1518:                                            ELF::R_LARCH_TLS_DESC64_PC_HI12));
1519:   Insts.push_back(LoongArchAsmParser::Inst(LoongArch::ADD_D));
1520:   Insts.push_back(
```
- **EN**: The range implements or declares functions including `LoongArchAsmParser::emitLoadAddressTLSDescLarge`.
- **CN**: 这一段实现或声明了 `LoongArchAsmParser::emitLoadAddressTLSDescLarge` 等函数。

### Lines 1521-1560 / 第 1521-1560 行
```cpp
1521:       LoongArchAsmParser::Inst(LoongArch::LD_D, ELF::R_LARCH_TLS_DESC_LD));
1522:   Insts.push_back(
1523:       LoongArchAsmParser::Inst(LoongArch::JIRL, ELF::R_LARCH_TLS_DESC_CALL));
1524: 
1525:   emitLAInstSeq(DestReg, TmpReg, Symbol, Insts, IDLoc, Out);
1526: }
1527: 
1528: void LoongArchAsmParser::emitLoadImm(MCInst &Inst, SMLoc IDLoc,
1529:                                      MCStreamer &Out) {
1530:   MCRegister DestReg = Inst.getOperand(0).getReg();
1531:   int64_t Imm = Inst.getOperand(1).getImm();
1532:   MCRegister SrcReg = LoongArch::R0;
1533: 
1534:   if (Inst.getOpcode() == LoongArch::PseudoLI_W)
1535:     Imm = SignExtend64<32>(Imm);
1536: 
1537:   for (LoongArchMatInt::Inst &Inst : LoongArchMatInt::generateInstSeq(Imm)) {
1538:     switch (Inst.Opc) {
1539:     case LoongArch::LU12I_W:
1540:       Out.emitInstruction(
1541:           MCInstBuilder(Inst.Opc).addReg(DestReg).addImm(Inst.Imm), getSTI());
1542:       break;
1543:     case LoongArch::ADDI_W:
1544:     case LoongArch::ORI:
1545:     case LoongArch::LU32I_D:
1546:     case LoongArch::LU52I_D:
1547:       Out.emitInstruction(
1548:           MCInstBuilder(Inst.Opc).addReg(DestReg).addReg(SrcReg).addImm(
1549:               Inst.Imm),
1550:           getSTI());
1551:       break;
1552:     case LoongArch::BSTRINS_D:
1553:       Out.emitInstruction(MCInstBuilder(Inst.Opc)
1554:                               .addReg(DestReg)
1555:                               .addReg(SrcReg)
1556:                               .addReg(SrcReg)
1557:                               .addImm(Inst.Imm >> 32)
1558:                               .addImm(Inst.Imm & 0xFF),
1559:                           getSTI());
1560:       break;
```
- **EN**: The range implements or declares functions including `LoongArchAsmParser::emitLoadImm`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchAsmParser::emitLoadImm` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1561-1600 / 第 1561-1600 行
```cpp
1561:     default:
1562:       llvm_unreachable("unexpected opcode generated by LoongArchMatInt");
1563:     }
1564:     SrcReg = DestReg;
1565:   }
1566: }
1567: 
1568: void LoongArchAsmParser::emitFuncCall(MCInst &Inst, SMLoc IDLoc,
1569:                                       MCStreamer &Out, bool IsTailCall,
1570:                                       bool IsCall36) {
1571:   // call30 sym
1572:   // expands to:
1573:   //   pcaddu12i $ra, %call30(sym)
1574:   //   jirl      $ra, $ra, 0
1575:   //
1576:   // tail30 $rj, sym
1577:   // expands to:
1578:   //   pcaddu12i $rj, %call30(sym)
1579:   //   jirl      $r0, $rj, 0
1580:   //
1581:   // call36 sym
1582:   // expands to:
1583:   //   pcaddu18i $ra, %call36(sym)
1584:   //   jirl      $ra, $ra, 0
1585:   //
1586:   // tail36 $rj, sym
1587:   // expands to:
1588:   //   pcaddu18i $rj, %call36(sym)
1589:   //   jirl      $r0, $rj, 0
1590:   MCRegister ScratchReg =
1591:       IsTailCall ? Inst.getOperand(0).getReg() : MCRegister(LoongArch::R1);
1592:   unsigned PCAI = IsCall36 ? LoongArch::PCADDU18I : LoongArch::PCADDU12I;
1593:   unsigned Rel = IsCall36 ? ELF::R_LARCH_CALL36 : ELF::R_LARCH_CALL30;
1594:   const MCExpr *Sym =
1595:       IsTailCall ? Inst.getOperand(1).getExpr() : Inst.getOperand(0).getExpr();
1596:   const LoongArchMCExpr *LE =
1597:       LoongArchMCExpr::create(Sym, Rel, getContext(), /*RelaxHint=*/true);
1598: 
1599:   Out.emitInstruction(MCInstBuilder(PCAI).addReg(ScratchReg).addExpr(LE),
1600:                       getSTI());
```
- **EN**: The range implements or declares functions including `LoongArchAsmParser::emitFuncCall`. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `LoongArchAsmParser::emitFuncCall` 等函数。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 1601-1640 / 第 1601-1640 行
```cpp
1601:   Out.emitInstruction(
1602:       MCInstBuilder(LoongArch::JIRL)
1603:           .addReg(IsTailCall ? MCRegister(LoongArch::R0) : ScratchReg)
1604:           .addReg(ScratchReg)
1605:           .addImm(0),
1606:       getSTI());
1607: }
1608: 
1609: bool LoongArchAsmParser::processInstruction(MCInst &Inst, SMLoc IDLoc,
1610:                                             OperandVector &Operands,
1611:                                             MCStreamer &Out) {
1612:   Inst.setLoc(IDLoc);
1613:   switch (Inst.getOpcode()) {
1614:   default:
1615:     break;
1616:   case LoongArch::PseudoLA_ABS:
1617:   case LoongArch::PseudoLA_ABS_LARGE:
1618:     emitLoadAddressAbs(Inst, IDLoc, Out);
1619:     return false;
1620:   case LoongArch::PseudoLA_PCREL:
1621:     emitLoadAddressPcrel(Inst, IDLoc, Out);
1622:     return false;
1623:   case LoongArch::PseudoLA_PCREL_LARGE:
1624:     emitLoadAddressPcrelLarge(Inst, IDLoc, Out);
1625:     return false;
1626:   case LoongArch::PseudoLA_GOT:
1627:     emitLoadAddressGot(Inst, IDLoc, Out);
1628:     return false;
1629:   case LoongArch::PseudoLA_GOT_LARGE:
1630:     emitLoadAddressGotLarge(Inst, IDLoc, Out);
1631:     return false;
1632:   case LoongArch::PseudoLA_TLS_LE:
1633:     emitLoadAddressTLSLE(Inst, IDLoc, Out);
1634:     return false;
1635:   case LoongArch::PseudoLA_TLS_IE:
1636:     emitLoadAddressTLSIE(Inst, IDLoc, Out);
1637:     return false;
1638:   case LoongArch::PseudoLA_TLS_IE_LARGE:
1639:     emitLoadAddressTLSIELarge(Inst, IDLoc, Out);
1640:     return false;
```
- **EN**: The range implements or declares functions including `MCInstBuilder`, `LoongArchAsmParser::processInstruction`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这一段实现或声明了 `MCInstBuilder`, `LoongArchAsmParser::processInstruction` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 1641-1680 / 第 1641-1680 行
```cpp
1641:   case LoongArch::PseudoLA_TLS_LD:
1642:     emitLoadAddressTLSLD(Inst, IDLoc, Out);
1643:     return false;
1644:   case LoongArch::PseudoLA_TLS_LD_LARGE:
1645:     emitLoadAddressTLSLDLarge(Inst, IDLoc, Out);
1646:     return false;
1647:   case LoongArch::PseudoLA_TLS_GD:
1648:     emitLoadAddressTLSGD(Inst, IDLoc, Out);
1649:     return false;
1650:   case LoongArch::PseudoLA_TLS_GD_LARGE:
1651:     emitLoadAddressTLSGDLarge(Inst, IDLoc, Out);
1652:     return false;
1653:   case LoongArch::PseudoLA_TLS_DESC:
1654:     emitLoadAddressTLSDesc(Inst, IDLoc, Out);
1655:     return false;
1656:   case LoongArch::PseudoLA_TLS_DESC_LARGE:
1657:     emitLoadAddressTLSDescLarge(Inst, IDLoc, Out);
1658:     return false;
1659:   case LoongArch::PseudoLI_W:
1660:   case LoongArch::PseudoLI_D:
1661:     emitLoadImm(Inst, IDLoc, Out);
1662:     return false;
1663:   case LoongArch::PseudoCALL:
1664:     emitFuncCall(Inst, IDLoc, Out, /*IsTailCall=*/false,
1665:                  /*IsCall36=*/is64Bit());
1666:     return false;
1667:   case LoongArch::PseudoCALL30:
1668:     emitFuncCall(Inst, IDLoc, Out, /*IsTailCall=*/false, /*IsCall36=*/false);
1669:     return false;
1670:   case LoongArch::PseudoCALL36:
1671:     emitFuncCall(Inst, IDLoc, Out, /*IsTailCall=*/false, /*IsCall36=*/true);
1672:     return false;
1673:   case LoongArch::PseudoTAIL:
1674:     emitFuncCall(Inst, IDLoc, Out, /*IsTailCall=*/true, /*IsCall36=*/is64Bit());
1675:     return false;
1676:   case LoongArch::PseudoTAIL30:
1677:     emitFuncCall(Inst, IDLoc, Out, /*IsTailCall=*/true, /*IsCall36=*/false);
1678:     return false;
1679:   case LoongArch::PseudoTAIL36:
1680:     emitFuncCall(Inst, IDLoc, Out, /*IsTailCall=*/true, /*IsCall36=*/true);
```
- **EN**: This span continues the file's main responsibility: this file parses target assembly syntax into MC/LLVM instructions for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1681-1720 / 第 1681-1720 行
```cpp
1681:     return false;
1682:   }
1683:   Out.emitInstruction(Inst, getSTI());
1684:   return false;
1685: }
1686: 
1687: unsigned LoongArchAsmParser::checkTargetMatchPredicate(MCInst &Inst) {
1688:   unsigned Opc = Inst.getOpcode();
1689:   const MCInstrDesc &MCID = MII.get(Opc);
1690:   switch (Opc) {
1691:   default:
1692:     if (LoongArchII::isSubjectToAMORdConstraint(MCID.TSFlags)) {
1693:       const bool IsAMCAS = LoongArchII::isAMCAS(MCID.TSFlags);
1694:       MCRegister Rd = Inst.getOperand(0).getReg();
1695:       MCRegister Rk = Inst.getOperand(IsAMCAS ? 2 : 1).getReg();
1696:       MCRegister Rj = Inst.getOperand(IsAMCAS ? 3 : 2).getReg();
1697:       if ((Rd == Rk || Rd == Rj) && Rd != LoongArch::R0)
1698:         return Match_RequiresAMORdDifferRkRj;
1699:     }
1700:     break;
1701:   case LoongArch::PseudoLA_TLS_DESC:
1702:   case LoongArch::PseudoLA_TLS_DESC_LARGE: {
1703:     MCRegister Rd = Inst.getOperand(0).getReg();
1704:     if (Rd != LoongArch::R4)
1705:       return Match_RequiresLAORdR4;
1706:     break;
1707:   }
1708:   case LoongArch::PseudoLA_PCREL_LARGE:
1709:   case LoongArch::PseudoLA_GOT_LARGE:
1710:   case LoongArch::PseudoLA_TLS_IE_LARGE:
1711:   case LoongArch::PseudoLA_TLS_LD_LARGE:
1712:   case LoongArch::PseudoLA_TLS_GD_LARGE: {
1713:     MCRegister Rd = Inst.getOperand(0).getReg();
1714:     MCRegister Rj = Inst.getOperand(1).getReg();
1715:     if (Rd == Rj)
1716:       return Match_RequiresLAORdDifferRj;
1717:     break;
1718:   }
1719:   case LoongArch::CSRXCHG:
1720:   case LoongArch::GCSRXCHG: {
```
- **EN**: The range implements or declares functions including `LoongArchAsmParser::checkTargetMatchPredicate`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchAsmParser::checkTargetMatchPredicate` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1721-1760 / 第 1721-1760 行
```cpp
1721:     MCRegister Rj = Inst.getOperand(2).getReg();
1722:     if (Rj == LoongArch::R0 || Rj == LoongArch::R1)
1723:       return Match_RequiresOpnd2NotR0R1;
1724:     return Match_Success;
1725:   }
1726:   case LoongArch::BSTRINS_W:
1727:   case LoongArch::BSTRINS_D:
1728:   case LoongArch::BSTRPICK_W:
1729:   case LoongArch::BSTRPICK_D: {
1730:     unsigned Opc = Inst.getOpcode();
1731:     const signed Msb =
1732:         (Opc == LoongArch::BSTRINS_W || Opc == LoongArch::BSTRINS_D)
1733:             ? Inst.getOperand(3).getImm()
1734:             : Inst.getOperand(2).getImm();
1735:     const signed Lsb =
1736:         (Opc == LoongArch::BSTRINS_W || Opc == LoongArch::BSTRINS_D)
1737:             ? Inst.getOperand(4).getImm()
1738:             : Inst.getOperand(3).getImm();
1739:     if (Msb < Lsb)
1740:       return Match_RequiresMsbNotLessThanLsb;
1741:     return Match_Success;
1742:   }
1743:   }
1744: 
1745:   return Match_Success;
1746: }
1747: 
1748: unsigned
1749: LoongArchAsmParser::validateTargetOperandClass(MCParsedAsmOperand &AsmOp,
1750:                                                unsigned Kind) {
1751:   LoongArchOperand &Op = static_cast<LoongArchOperand &>(AsmOp);
1752:   if (!Op.isReg())
1753:     return Match_InvalidOperand;
1754: 
1755:   MCRegister Reg = Op.getReg();
1756:   // As the parser couldn't differentiate an FPR32 from an FPR64, coerce the
1757:   // register from FPR32 to FPR64 if necessary.
1758:   if (LoongArchMCRegisterClasses[LoongArch::FPR32RegClassID].contains(Reg) &&
1759:       Kind == MCK_FPR64) {
1760:     Op.setReg(convertFPR32ToFPR64(Reg));
```
- **EN**: The range implements or declares functions including `LoongArchAsmParser::validateTargetOperandClass`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchAsmParser::validateTargetOperandClass` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1761-1800 / 第 1761-1800 行
```cpp
1761:     return Match_Success;
1762:   }
1763: 
1764:   if (Kind == MCK_GPRNoR0R1 && (Reg == LoongArch::R0 || Reg == LoongArch::R1))
1765:     return Match_RequiresOpnd2NotR0R1;
1766: 
1767:   return Match_InvalidOperand;
1768: }
1769: 
1770: bool LoongArchAsmParser::generateImmOutOfRangeError(
1771:     OperandVector &Operands, uint64_t ErrorInfo, int64_t Lower, int64_t Upper,
1772:     const Twine &Msg = "immediate must be an integer in the range") {
1773:   SMLoc ErrorLoc = ((LoongArchOperand &)*Operands[ErrorInfo]).getStartLoc();
1774:   return Error(ErrorLoc, Msg + " [" + Twine(Lower) + ", " + Twine(Upper) + "]");
1775: }
1776: 
1777: bool LoongArchAsmParser::matchAndEmitInstruction(SMLoc IDLoc, unsigned &Opcode,
1778:                                                  OperandVector &Operands,
1779:                                                  MCStreamer &Out,
1780:                                                  uint64_t &ErrorInfo,
1781:                                                  bool MatchingInlineAsm) {
1782:   MCInst Inst;
1783:   FeatureBitset MissingFeatures;
1784: 
1785:   auto Result = MatchInstructionImpl(Operands, Inst, ErrorInfo, MissingFeatures,
1786:                                      MatchingInlineAsm);
1787:   switch (Result) {
1788:   default:
1789:     break;
1790:   case Match_Success:
1791:     return processInstruction(Inst, IDLoc, Operands, Out);
1792:   case Match_MissingFeature: {
1793:     assert(MissingFeatures.any() && "Unknown missing features!");
1794:     bool FirstFeature = true;
1795:     std::string Msg = "instruction requires the following:";
1796:     for (unsigned i = 0, e = MissingFeatures.size(); i != e; ++i) {
1797:       if (MissingFeatures[i]) {
1798:         Msg += FirstFeature ? " " : ", ";
1799:         Msg += getSubtargetFeatureName(i);
1800:         FirstFeature = false;
```
- **EN**: The range implements or declares functions including `LoongArchAsmParser::generateImmOutOfRangeError`, `LoongArchAsmParser::matchAndEmitInstruction`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchAsmParser::generateImmOutOfRangeError`, `LoongArchAsmParser::matchAndEmitInstruction` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1801-1840 / 第 1801-1840 行
```cpp
1801:       }
1802:     }
1803:     return Error(IDLoc, Msg);
1804:   }
1805:   case Match_MnemonicFail: {
1806:     FeatureBitset FBS = ComputeAvailableFeatures(getSTI().getFeatureBits());
1807:     std::string Suggestion = LoongArchMnemonicSpellCheck(
1808:         ((LoongArchOperand &)*Operands[0]).getToken(), FBS, 0);
1809:     return Error(IDLoc, "unrecognized instruction mnemonic" + Suggestion);
1810:   }
1811:   case Match_InvalidOperand: {
1812:     SMLoc ErrorLoc = IDLoc;
1813:     if (ErrorInfo != ~0ULL) {
1814:       if (ErrorInfo >= Operands.size())
1815:         return Error(ErrorLoc, "too few operands for instruction");
1816: 
1817:       ErrorLoc = ((LoongArchOperand &)*Operands[ErrorInfo]).getStartLoc();
1818:       if (ErrorLoc == SMLoc())
1819:         ErrorLoc = IDLoc;
1820:     }
1821:     return Error(ErrorLoc, "invalid operand for instruction");
1822:   }
1823:   }
1824: 
1825:   // Handle the case when the error message is of specific type
1826:   // other than the generic Match_InvalidOperand, and the
1827:   // corresponding operand is missing.
1828:   if (Result > FIRST_TARGET_MATCH_RESULT_TY) {
1829:     SMLoc ErrorLoc = IDLoc;
1830:     if (ErrorInfo != ~0ULL && ErrorInfo >= Operands.size())
1831:       return Error(ErrorLoc, "too few operands for instruction");
1832:   }
1833: 
1834:   switch (Result) {
1835:   default:
1836:     break;
1837:   case Match_RequiresMsbNotLessThanLsb: {
1838:     SMLoc ErrorStart = Operands[3]->getStartLoc();
1839:     return Error(ErrorStart, "msb is less than lsb",
1840:                  SMRange(ErrorStart, Operands[4]->getEndLoc()));
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1841-1880 / 第 1841-1880 行
```cpp
1841:   }
1842:   case Match_RequiresOpnd2NotR0R1:
1843:     return Error(Operands[2]->getStartLoc(), "must not be $r0 or $r1");
1844:   case Match_RequiresAMORdDifferRkRj:
1845:     return Error(Operands[1]->getStartLoc(),
1846:                  "$rd must be different from both $rk and $rj");
1847:   case Match_RequiresLAORdDifferRj:
1848:     return Error(Operands[1]->getStartLoc(), "$rd must be different from $rj");
1849:   case Match_RequiresLAORdR4:
1850:     return Error(Operands[1]->getStartLoc(), "$rd must be $r4");
1851:   case Match_InvalidUImm1:
1852:     return generateImmOutOfRangeError(Operands, ErrorInfo, /*Lower=*/0,
1853:                                       /*Upper=*/(1 << 1) - 1);
1854:   case Match_InvalidUImm2:
1855:     return generateImmOutOfRangeError(Operands, ErrorInfo, /*Lower=*/0,
1856:                                       /*Upper=*/(1 << 2) - 1);
1857:   case Match_InvalidUImm2plus1:
1858:     return generateImmOutOfRangeError(Operands, ErrorInfo, /*Lower=*/1,
1859:                                       /*Upper=*/(1 << 2));
1860:   case Match_InvalidUImm3:
1861:     return generateImmOutOfRangeError(Operands, ErrorInfo, /*Lower=*/0,
1862:                                       /*Upper=*/(1 << 3) - 1);
1863:   case Match_InvalidUImm4:
1864:     return generateImmOutOfRangeError(Operands, ErrorInfo, /*Lower=*/0,
1865:                                       /*Upper=*/(1 << 4) - 1);
1866:   case Match_InvalidUImm5:
1867:     return generateImmOutOfRangeError(Operands, ErrorInfo, /*Lower=*/0,
1868:                                       /*Upper=*/(1 << 5) - 1);
1869:   case Match_InvalidUImm6:
1870:     return generateImmOutOfRangeError(Operands, ErrorInfo, /*Lower=*/0,
1871:                                       /*Upper=*/(1 << 6) - 1);
1872:   case Match_InvalidUImm7:
1873:     return generateImmOutOfRangeError(Operands, ErrorInfo, /*Lower=*/0,
1874:                                       /*Upper=*/(1 << 7) - 1);
1875:   case Match_InvalidUImm8:
1876:     return generateImmOutOfRangeError(Operands, ErrorInfo, /*Lower=*/0,
1877:                                       /*Upper=*/(1 << 8) - 1);
1878:   case Match_InvalidUImm12:
1879:     return generateImmOutOfRangeError(Operands, ErrorInfo, /*Lower=*/0,
1880:                                       /*Upper=*/(1 << 12) - 1);
```
- **EN**: This span continues the file's main responsibility: this file parses target assembly syntax into MC/LLVM instructions for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1881-1920 / 第 1881-1920 行
```cpp
1881:   case Match_InvalidUImm12ori:
1882:     return generateImmOutOfRangeError(
1883:         Operands, ErrorInfo, /*Lower=*/0,
1884:         /*Upper=*/(1 << 12) - 1,
1885:         "operand must be a symbol with modifier (e.g. %abs_lo12) or an "
1886:         "integer in the range");
1887:   case Match_InvalidUImm14:
1888:     return generateImmOutOfRangeError(Operands, ErrorInfo, /*Lower=*/0,
1889:                                       /*Upper=*/(1 << 14) - 1);
1890:   case Match_InvalidUImm15:
1891:     return generateImmOutOfRangeError(Operands, ErrorInfo, /*Lower=*/0,
1892:                                       /*Upper=*/(1 << 15) - 1);
1893:   case Match_InvalidSImm5:
1894:     return generateImmOutOfRangeError(Operands, ErrorInfo, /*Lower=*/-(1 << 4),
1895:                                       /*Upper=*/(1 << 4) - 1);
1896:   case Match_InvalidSImm8:
1897:     return generateImmOutOfRangeError(Operands, ErrorInfo, /*Lower=*/-(1 << 7),
1898:                                       /*Upper=*/(1 << 7) - 1);
1899:   case Match_InvalidSImm8lsl1:
1900:     return generateImmOutOfRangeError(
1901:         Operands, ErrorInfo, /*Lower=*/-(1 << 8), /*Upper=*/(1 << 8) - 2,
1902:         "immediate must be a multiple of 2 in the range");
1903:   case Match_InvalidSImm8lsl2:
1904:     return generateImmOutOfRangeError(
1905:         Operands, ErrorInfo, /*Lower=*/-(1 << 9), /*Upper=*/(1 << 9) - 4,
1906:         "immediate must be a multiple of 4 in the range");
1907:   case Match_InvalidSImm10:
1908:     return generateImmOutOfRangeError(Operands, ErrorInfo, /*Lower=*/-(1 << 9),
1909:                                       /*Upper=*/(1 << 9) - 1);
1910:   case Match_InvalidSImm8lsl3:
1911:     return generateImmOutOfRangeError(
1912:         Operands, ErrorInfo, /*Lower=*/-(1 << 10), /*Upper=*/(1 << 10) - 8,
1913:         "immediate must be a multiple of 8 in the range");
1914:   case Match_InvalidSImm9lsl3:
1915:     return generateImmOutOfRangeError(
1916:         Operands, ErrorInfo, /*Lower=*/-(1 << 11), /*Upper=*/(1 << 11) - 8,
1917:         "immediate must be a multiple of 8 in the range");
1918:   case Match_InvalidSImm10lsl2:
1919:     return generateImmOutOfRangeError(
1920:         Operands, ErrorInfo, /*Lower=*/-(1 << 11), /*Upper=*/(1 << 11) - 4,
```
- **EN**: This span continues the file's main responsibility: this file parses target assembly syntax into MC/LLVM instructions for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1921-1960 / 第 1921-1960 行
```cpp
1921:         "immediate must be a multiple of 4 in the range");
1922:   case Match_InvalidSImm11lsl1:
1923:     return generateImmOutOfRangeError(
1924:         Operands, ErrorInfo, /*Lower=*/-(1 << 11), /*Upper=*/(1 << 11) - 2,
1925:         "immediate must be a multiple of 2 in the range");
1926:   case Match_InvalidSImm12:
1927:     return generateImmOutOfRangeError(Operands, ErrorInfo, /*Lower=*/-(1 << 11),
1928:                                       /*Upper=*/(1 << 11) - 1);
1929:   case Match_InvalidSImm12addlike:
1930:     return generateImmOutOfRangeError(
1931:         Operands, ErrorInfo, /*Lower=*/-(1 << 11),
1932:         /*Upper=*/(1 << 11) - 1,
1933:         "operand must be a symbol with modifier (e.g. %pc_lo12) or an integer "
1934:         "in the range");
1935:   case Match_InvalidSImm12lu52id:
1936:     return generateImmOutOfRangeError(
1937:         Operands, ErrorInfo, /*Lower=*/-(1 << 11),
1938:         /*Upper=*/(1 << 11) - 1,
1939:         "operand must be a symbol with modifier (e.g. %pc64_hi12) or an "
1940:         "integer in the range");
1941:   case Match_InvalidSImm13:
1942:     return generateImmOutOfRangeError(Operands, ErrorInfo, /*Lower=*/-(1 << 12),
1943:                                       /*Upper=*/(1 << 12) - 1);
1944:   case Match_InvalidSImm14lsl2:
1945:     return generateImmOutOfRangeError(
1946:         Operands, ErrorInfo, /*Lower=*/-(1 << 15), /*Upper=*/(1 << 15) - 4,
1947:         "immediate must be a multiple of 4 in the range");
1948:   case Match_InvalidSImm16:
1949:     return generateImmOutOfRangeError(Operands, ErrorInfo, /*Lower=*/-(1 << 15),
1950:                                       /*Upper=*/(1 << 15) - 1);
1951:   case Match_InvalidSImm16lsl2:
1952:     return generateImmOutOfRangeError(
1953:         Operands, ErrorInfo, /*Lower=*/-(1 << 17), /*Upper=*/(1 << 17) - 4,
1954:         "operand must be a symbol with modifier (e.g. %b16) or an integer "
1955:         "in the range");
1956:   case Match_InvalidSImm20:
1957:     return generateImmOutOfRangeError(Operands, ErrorInfo, /*Lower=*/-(1 << 19),
1958:                                       /*Upper=*/(1 << 19) - 1);
1959:   case Match_InvalidSImm20lu12iw:
1960:     return generateImmOutOfRangeError(
```
- **EN**: This span continues the file's main responsibility: this file parses target assembly syntax into MC/LLVM instructions for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 1961-2000 / 第 1961-2000 行
```cpp
1961:         Operands, ErrorInfo, /*Lower=*/-(1 << 19),
1962:         /*Upper=*/(1 << 19) - 1,
1963:         "operand must be a symbol with modifier (e.g. %abs_hi20) or an integer "
1964:         "in the range");
1965:   case Match_InvalidSImm20lu32id:
1966:     return generateImmOutOfRangeError(
1967:         Operands, ErrorInfo, /*Lower=*/-(1 << 19),
1968:         /*Upper=*/(1 << 19) - 1,
1969:         "operand must be a symbol with modifier (e.g. %abs64_lo20) or an "
1970:         "integer in the range");
1971:   case Match_InvalidSImm20pcalau12i:
1972:     return generateImmOutOfRangeError(
1973:         Operands, ErrorInfo, /*Lower=*/-(1 << 19),
1974:         /*Upper=*/(1 << 19) - 1,
1975:         "operand must be a symbol with modifier (e.g. %pc_hi20) or an integer "
1976:         "in the range");
1977:   case Match_InvalidSImm20pcaddu12i:
1978:     return generateImmOutOfRangeError(
1979:         Operands, ErrorInfo, /*Lower=*/-(1 << 19),
1980:         /*Upper=*/(1 << 19) - 1,
1981:         "operand must be a symbol with modifier (e.g. %call30 or %pcadd_hi20) "
1982:         "or an integer in the range");
1983:   case Match_InvalidSImm20pcaddu18i:
1984:     return generateImmOutOfRangeError(
1985:         Operands, ErrorInfo, /*Lower=*/-(1 << 19),
1986:         /*Upper=*/(1 << 19) - 1,
1987:         "operand must be a symbol with modifier (e.g. %call36) or an integer "
1988:         "in the range");
1989:   case Match_InvalidSImm20pcaddi:
1990:     return generateImmOutOfRangeError(
1991:         Operands, ErrorInfo, /*Lower=*/-(1 << 19),
1992:         /*Upper=*/(1 << 19) - 1,
1993:         "operand must be a symbol with modifier (e.g. %pcrel_20) or an integer "
1994:         "in the range");
1995:   case Match_InvalidSImm21lsl2:
1996:     return generateImmOutOfRangeError(
1997:         Operands, ErrorInfo, /*Lower=*/-(1 << 22), /*Upper=*/(1 << 22) - 4,
1998:         "operand must be a symbol with modifier (e.g. %b21) or an integer "
1999:         "in the range");
2000:   case Match_InvalidSImm26Operand:
```
- **EN**: This span continues the file's main responsibility: this file parses target assembly syntax into MC/LLVM instructions for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 2001-2040 / 第 2001-2040 行
```cpp
2001:     return generateImmOutOfRangeError(
2002:         Operands, ErrorInfo, /*Lower=*/-(1 << 27), /*Upper=*/(1 << 27) - 4,
2003:         "operand must be a bare symbol name or an immediate must be a multiple "
2004:         "of 4 in the range");
2005:   case Match_InvalidImm32: {
2006:     SMLoc ErrorLoc = ((LoongArchOperand &)*Operands[ErrorInfo]).getStartLoc();
2007:     return Error(ErrorLoc, "operand must be a 32 bit immediate");
2008:   }
2009:   case Match_InvalidImm64: {
2010:     SMLoc ErrorLoc = ((LoongArchOperand &)*Operands[ErrorInfo]).getStartLoc();
2011:     return Error(ErrorLoc, "operand must be a 64 bit immediate");
2012:   }
2013:   case Match_InvalidBareSymbol: {
2014:     SMLoc ErrorLoc = ((LoongArchOperand &)*Operands[ErrorInfo]).getStartLoc();
2015:     return Error(ErrorLoc, "operand must be a bare symbol name");
2016:   }
2017:   case Match_InvalidTPRelAddSymbol: {
2018:     SMLoc ErrorLoc = ((LoongArchOperand &)*Operands[ErrorInfo]).getStartLoc();
2019:     return Error(ErrorLoc, "operand must be a symbol with %le_add_r modifier");
2020:   }
2021:   }
2022:   llvm_unreachable("Unknown match type detected!");
2023: }
2024: 
2025: bool LoongArchAsmParser::parseDirectiveOption() {
2026:   MCAsmParser &Parser = getParser();
2027:   // Get the option token.
2028:   AsmToken Tok = Parser.getTok();
2029: 
2030:   // At the moment only identifiers are supported.
2031:   if (parseToken(AsmToken::Identifier, "expected identifier"))
2032:     return true;
2033: 
2034:   StringRef Option = Tok.getIdentifier();
2035: 
2036:   if (Option == "push") {
2037:     if (Parser.parseEOL())
2038:       return true;
2039: 
2040:     getTargetStreamer().emitDirectiveOptionPush();
```
- **EN**: The range implements or declares functions including `LoongArchAsmParser::parseDirectiveOption`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `LoongArchAsmParser::parseDirectiveOption` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 2041-2080 / 第 2041-2080 行
```cpp
2041:     pushFeatureBits();
2042:     return false;
2043:   }
2044: 
2045:   if (Option == "pop") {
2046:     SMLoc StartLoc = Parser.getTok().getLoc();
2047:     if (Parser.parseEOL())
2048:       return true;
2049: 
2050:     getTargetStreamer().emitDirectiveOptionPop();
2051:     if (popFeatureBits())
2052:       return Error(StartLoc, ".option pop with no .option push");
2053: 
2054:     return false;
2055:   }
2056: 
2057:   if (Option == "relax") {
2058:     if (Parser.parseEOL())
2059:       return true;
2060: 
2061:     getTargetStreamer().emitDirectiveOptionRelax();
2062:     setFeatureBits(LoongArch::FeatureRelax, "relax");
2063:     return false;
2064:   }
2065: 
2066:   if (Option == "norelax") {
2067:     if (Parser.parseEOL())
2068:       return true;
2069: 
2070:     getTargetStreamer().emitDirectiveOptionNoRelax();
2071:     clearFeatureBits(LoongArch::FeatureRelax, "relax");
2072:     return false;
2073:   }
2074: 
2075:   // Unknown option.
2076:   Warning(Parser.getTok().getLoc(),
2077:           "unknown option, expected 'push', 'pop', 'relax' or 'norelax'");
2078:   Parser.eatToEndOfStatement();
2079:   return false;
2080: }
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 2081-2093 / 第 2081-2093 行
```cpp
2081: 
2082: ParseStatus LoongArchAsmParser::parseDirective(AsmToken DirectiveID) {
2083:   if (DirectiveID.getString() == ".option")
2084:     return parseDirectiveOption();
2085: 
2086:   return ParseStatus::NoMatch;
2087: }
2088: 
2089: extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
2090: LLVMInitializeLoongArchAsmParser() {
2091:   RegisterMCAsmParser<LoongArchAsmParser> X(getTheLoongArch32Target());
2092:   RegisterMCAsmParser<LoongArchAsmParser> Y(getTheLoongArch64Target());
2093: }
```
- **EN**: The range implements or declares functions including `LoongArchAsmParser::parseDirective`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchAsmParser::parseDirective` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。
- **MC layer**: Uses LLVM MC structures for encoding, printing, parsing, or disassembly. / 使用 LLVM MC 结构进行编码、打印、解析或反汇编。
- **MC streaming**: Emits sections, symbols, expressions, and encoded bytes to object or assembly output. / 向目标文件或汇编输出节、符号、表达式和编码字节。
- **Assembly parsing**: Reads textual assembly operands, directives, and mnemonics. / 读取文本汇编的操作数、伪指令和助记符。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Register classes**: Groups registers by capability so instruction operands and allocators can constrain them. / 按能力对寄存器分组，以便指令操作数和分配器施加约束。

## Dependencies / 依赖关系
- `MCTargetDesc/LoongArchBaseInfo.h`
- `MCTargetDesc/LoongArchInstPrinter.h`
- `MCTargetDesc/LoongArchMCAsmInfo.h`
- `MCTargetDesc/LoongArchMCTargetDesc.h`
- `MCTargetDesc/LoongArchMatInt.h`
- `MCTargetDesc/LoongArchTargetStreamer.h`
- `TargetInfo/LoongArchTargetInfo.h`
- `llvm/BinaryFormat/ELF.h`
- `llvm/MC/MCContext.h`
- `llvm/MC/MCInstBuilder.h`
- `llvm/MC/MCInstrInfo.h`
- `llvm/MC/MCParser/AsmLexer.h`
- `llvm/MC/MCParser/MCParsedAsmOperand.h`
- `llvm/MC/MCParser/MCTargetAsmParser.h`
- `llvm/MC/MCRegisterInfo.h`
- `llvm/MC/MCStreamer.h`
- `...` (6 more include dependencies omitted for brevity / 其余 6 个 include 依赖已省略)
