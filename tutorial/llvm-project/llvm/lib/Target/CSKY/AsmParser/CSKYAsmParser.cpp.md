# CSKYAsmParser.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/CSKY/AsmParser/CSKYAsmParser.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Implements the target-specific assembly parser that turns textual assembly into MC operands and instructions.
- 目的（中文）: 实现目标专用的汇编解析器，将文本汇编转换为 MC 操作数与指令。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===---- CSKYAsmParser.cpp - Parse CSKY assembly to MCInst instructions --===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: #include "MCTargetDesc/CSKYInstPrinter.h"
  10: #include "MCTargetDesc/CSKYMCAsmInfo.h"
  11: #include "MCTargetDesc/CSKYMCTargetDesc.h"
  12: #include "MCTargetDesc/CSKYTargetStreamer.h"
  13: #include "TargetInfo/CSKYTargetInfo.h"
  14: #include "llvm/ADT/STLExtras.h"
  15: #include "llvm/ADT/Statistic.h"
  16: #include "llvm/ADT/StringExtras.h"
  17: #include "llvm/ADT/StringSwitch.h"
  18: #include "llvm/BinaryFormat/ELF.h"
  19: #include "llvm/CodeGen/Register.h"
  20: #include "llvm/MC/MCContext.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 21-40

```cpp
  21: #include "llvm/MC/MCExpr.h"
  22: #include "llvm/MC/MCInst.h"
  23: #include "llvm/MC/MCInstrInfo.h"
  24: #include "llvm/MC/MCParser/AsmLexer.h"
  25: #include "llvm/MC/MCParser/MCParsedAsmOperand.h"
  26: #include "llvm/MC/MCParser/MCTargetAsmParser.h"
  27: #include "llvm/MC/MCRegisterInfo.h"
  28: #include "llvm/MC/MCSectionELF.h"
  29: #include "llvm/MC/MCStreamer.h"
  30: #include "llvm/MC/MCSubtargetInfo.h"
  31: #include "llvm/MC/TargetRegistry.h"
  32: #include "llvm/Support/CSKYAttributes.h"
  33: #include "llvm/Support/Casting.h"
  34: #include "llvm/Support/CommandLine.h"
  35: #include "llvm/Support/Debug.h"
  36: #include "llvm/TargetParser/CSKYTargetParser.h"
  37: 
  38: using namespace llvm;
  39: 
  40: #define DEBUG_TYPE "csky-asm-parser"
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 41-60

```cpp
  41: 
  42: // Include the auto-generated portion of the compress emitter.
  43: #define GEN_COMPRESS_INSTR
  44: #include "CSKYGenCompressInstEmitter.inc"
  45: 
  46: STATISTIC(CSKYNumInstrsCompressed,
  47:           "Number of C-SKY Compressed instructions emitted");
  48: 
  49: static cl::opt<bool>
  50:     EnableCompressedInst("enable-csky-asm-compressed-inst", cl::Hidden,
  51:                          cl::init(false),
  52:                          cl::desc("Enable C-SKY asm compressed instruction"));
  53: 
  54: namespace {
  55: struct CSKYOperand;
  56: 
  57: class CSKYAsmParser : public MCTargetAsmParser {
  58: 
  59:   const MCRegisterInfo *MRI;
  60: 
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. This chunk introduces interfaces or data structures such as CSKYOperand, CSKYAsmParser, which organize the target-specific behavior exposed by the file. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 这一段引入了 CSKYOperand, CSKYAsmParser 等接口或数据结构，用于组织该文件暴露的目标专用行为。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 61-80

```cpp
  61:   unsigned validateTargetOperandClass(MCParsedAsmOperand &Op,
  62:                                       unsigned Kind) override;
  63: 
  64:   bool generateImmOutOfRangeError(OperandVector &Operands, uint64_t ErrorInfo,
  65:                                   int64_t Lower, int64_t Upper,
  66:                                   const Twine &Msg);
  67: 
  68:   SMLoc getLoc() const { return getParser().getTok().getLoc(); }
  69: 
  70:   bool matchAndEmitInstruction(SMLoc IDLoc, unsigned &Opcode,
  71:                                OperandVector &Operands, MCStreamer &Out,
  72:                                uint64_t &ErrorInfo,
  73:                                bool MatchingInlineAsm) override;
  74: 
  75:   bool parseRegister(MCRegister &Reg, SMLoc &StartLoc, SMLoc &EndLoc) override;
  76: 
  77:   bool parseInstruction(ParseInstructionInfo &Info, StringRef Name,
  78:                         SMLoc NameLoc, OperandVector &Operands) override;
  79: 
  80:   ParseStatus parseDirective(AsmToken DirectiveID) override;
```

- EN: Function bodies or method definitions such as getLoc contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: getLoc 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 81-100

```cpp
  81: 
  82:   // Helper to actually emit an instruction to the MCStreamer. Also, when
  83:   // possible, compression of the instruction is performed.
  84:   void emitToStreamer(MCStreamer &S, const MCInst &Inst);
  85: 
  86:   ParseStatus tryParseRegister(MCRegister &Reg, SMLoc &StartLoc,
  87:                                SMLoc &EndLoc) override;
  88: 
  89:   bool processInstruction(MCInst &Inst, SMLoc IDLoc, OperandVector &Operands,
  90:                           MCStreamer &Out);
  91:   bool processLRW(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out);
  92:   bool processJSRI(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out);
  93:   bool processJMPI(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out);
  94: 
  95:   CSKYTargetStreamer &getTargetStreamer() {
  96:     assert(getParser().getStreamer().getTargetStreamer() &&
  97:            "do not have a target streamer");
  98:     MCTargetStreamer &TS = *getParser().getStreamer().getTargetStreamer();
  99:     return static_cast<CSKYTargetStreamer &>(TS);
 100:   }
```

- EN: In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 101-120

```cpp
 101: 
 102: // Auto-generated instruction matching functions
 103: #define GET_ASSEMBLER_HEADER
 104: #include "CSKYGenAsmMatcher.inc"
 105: 
 106:   ParseStatus parseImmediate(OperandVector &Operands);
 107:   ParseStatus parseRegister(OperandVector &Operands);
 108:   ParseStatus parseBaseRegImm(OperandVector &Operands);
 109:   ParseStatus parseCSKYSymbol(OperandVector &Operands);
 110:   ParseStatus parseConstpoolSymbol(OperandVector &Operands);
 111:   ParseStatus parseDataSymbol(OperandVector &Operands);
 112:   ParseStatus parsePSRFlag(OperandVector &Operands);
 113:   ParseStatus parseRegSeq(OperandVector &Operands);
 114:   ParseStatus parseRegList(OperandVector &Operands);
 115: 
 116:   bool parseOperand(OperandVector &Operands, StringRef Mnemonic);
 117: 
 118:   bool parseDirectiveAttribute();
 119: 
 120: public:
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 121-140

```cpp
 121:   enum CSKYMatchResultTy {
 122:     Match_Dummy = FIRST_TARGET_MATCH_RESULT_TY,
 123:     Match_RequiresSameSrcAndDst,
 124:     Match_InvalidRegOutOfRange,
 125: #define GET_OPERAND_DIAGNOSTIC_TYPES
 126: #include "CSKYGenAsmMatcher.inc"
 127: #undef GET_OPERAND_DIAGNOSTIC_TYPES
 128:   };
 129: 
 130:   CSKYAsmParser(const MCSubtargetInfo &STI, MCAsmParser &Parser,
 131:                 const MCInstrInfo &MII)
 132:       : MCTargetAsmParser(STI, MII) {
 133: 
 134:     MCAsmParserExtension::Initialize(Parser);
 135: 
 136:     // Cache the MCRegisterInfo.
 137:     MRI = getContext().getRegisterInfo();
 138: 
 139:     setAvailableFeatures(ComputeAvailableFeatures(STI.getFeatureBits()));
 140:     getTargetStreamer().emitTargetAttributes(STI);
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. Function bodies or method definitions such as MCTargetAsmParser contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 MCTargetAsmParser 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 141-160

```cpp
 141:   }
 142: };
 143: 
 144: /// Instances of this class represent a parsed machine instruction.
 145: struct CSKYOperand : public MCParsedAsmOperand {
 146: 
 147:   enum KindTy {
 148:     Token,
 149:     Register,
 150:     Immediate,
 151:     RegisterSeq,
 152:     CPOP,
 153:     RegisterList
 154:   } Kind;
 155: 
 156:   struct RegOp {
 157:     MCRegister RegNum;
 158:   };
 159: 
 160:   struct ImmOp {
```

- EN: This chunk introduces interfaces or data structures such as represent, CSKYOperand, RegOp, which organize the target-specific behavior exposed by the file. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 这一段引入了 represent, CSKYOperand, RegOp 等接口或数据结构，用于组织该文件暴露的目标专用行为。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 161-180

```cpp
 161:     const MCExpr *Val;
 162:   };
 163: 
 164:   struct ConstpoolOp {
 165:     const MCExpr *Val;
 166:   };
 167: 
 168:   struct RegSeqOp {
 169:     MCRegister RegNumFrom;
 170:     MCRegister RegNumTo;
 171:   };
 172: 
 173:   struct RegListOp {
 174:     MCRegister List1From;
 175:     MCRegister List1To;
 176:     MCRegister List2From;
 177:     MCRegister List2To;
 178:     MCRegister List3From;
 179:     MCRegister List3To;
 180:     MCRegister List4From;
```

- EN: This chunk introduces interfaces or data structures such as ConstpoolOp, RegSeqOp, RegListOp, which organize the target-specific behavior exposed by the file. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 这一段引入了 ConstpoolOp, RegSeqOp, RegListOp 等接口或数据结构，用于组织该文件暴露的目标专用行为。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 181-200

```cpp
 181:     MCRegister List4To;
 182:   };
 183: 
 184:   SMLoc StartLoc, EndLoc;
 185:   union {
 186:     StringRef Tok;
 187:     RegOp Reg;
 188:     ImmOp Imm;
 189:     ConstpoolOp CPool;
 190:     RegSeqOp RegSeq;
 191:     RegListOp RegList;
 192:   };
 193: 
 194:   CSKYOperand(KindTy K) : MCParsedAsmOperand(), Kind(K) {}
 195: 
 196: public:
 197:   CSKYOperand(const CSKYOperand &o) : MCParsedAsmOperand() {
 198:     Kind = o.Kind;
 199:     StartLoc = o.StartLoc;
 200:     EndLoc = o.EndLoc;
```

- EN: Function bodies or method definitions such as CSKYOperand contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: CSKYOperand 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 201-220

```cpp
 201:     switch (Kind) {
 202:     case Register:
 203:       Reg = o.Reg;
 204:       break;
 205:     case RegisterSeq:
 206:       RegSeq = o.RegSeq;
 207:       break;
 208:     case CPOP:
 209:       CPool = o.CPool;
 210:       break;
 211:     case Immediate:
 212:       Imm = o.Imm;
 213:       break;
 214:     case Token:
 215:       Tok = o.Tok;
 216:       break;
 217:     case RegisterList:
 218:       RegList = o.RegList;
 219:       break;
 220:     }
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 221-240

```cpp
 221:   }
 222: 
 223:   bool isToken() const override { return Kind == Token; }
 224:   bool isReg() const override { return Kind == Register; }
 225:   bool isImm() const override { return Kind == Immediate; }
 226:   bool isRegisterSeq() const { return Kind == RegisterSeq; }
 227:   bool isRegisterList() const { return Kind == RegisterList; }
 228:   bool isConstPoolOp() const { return Kind == CPOP; }
 229: 
 230:   bool isMem() const override { return false; }
 231: 
 232:   static bool evaluateConstantImm(const MCExpr *Expr, int64_t &Imm) {
 233:     if (auto CE = dyn_cast<MCConstantExpr>(Expr)) {
 234:       Imm = CE->getValue();
 235:       return true;
 236:     }
 237: 
 238:     return false;
 239:   }
 240: 
```

- EN: Function bodies or method definitions such as isRegisterSeq, isRegisterList, isConstPoolOp contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: isRegisterSeq, isRegisterList, isConstPoolOp 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 241-260

```cpp
 241:   template <unsigned num, unsigned shift = 0> bool isUImm() const {
 242:     if (!isImm())
 243:       return false;
 244: 
 245:     int64_t Imm;
 246:     bool IsConstantImm = evaluateConstantImm(getImm(), Imm);
 247:     return IsConstantImm && isShiftedUInt<num, shift>(Imm);
 248:   }
 249: 
 250:   template <unsigned num> bool isOImm() const {
 251:     if (!isImm())
 252:       return false;
 253: 
 254:     int64_t Imm;
 255:     bool IsConstantImm = evaluateConstantImm(getImm(), Imm);
 256:     return IsConstantImm && isUInt<num>(Imm - 1);
 257:   }
 258: 
 259:   template <unsigned num, unsigned shift = 0> bool isSImm() const {
 260:     if (!isImm())
```

- EN: Function bodies or method definitions such as isOImm contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: isOImm 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 261-280

```cpp
 261:       return false;
 262: 
 263:     int64_t Imm;
 264:     bool IsConstantImm = evaluateConstantImm(getImm(), Imm);
 265:     return IsConstantImm && isShiftedInt<num, shift>(Imm);
 266:   }
 267: 
 268:   bool isUImm1() const { return isUImm<1>(); }
 269:   bool isUImm2() const { return isUImm<2>(); }
 270:   bool isUImm3() const { return isUImm<3>(); }
 271:   bool isUImm4() const { return isUImm<4>(); }
 272:   bool isUImm5() const { return isUImm<5>(); }
 273:   bool isUImm6() const { return isUImm<6>(); }
 274:   bool isUImm7() const { return isUImm<7>(); }
 275:   bool isUImm8() const { return isUImm<8>(); }
 276:   bool isUImm12() const { return isUImm<12>(); }
 277:   bool isUImm16() const { return isUImm<16>(); }
 278:   bool isUImm20() const { return isUImm<20>(); }
 279:   bool isUImm24() const { return isUImm<24>(); }
 280: 
```

- EN: Function bodies or method definitions such as isUImm1, isUImm2, isUImm3 contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: isUImm1, isUImm2, isUImm3 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 281-300

```cpp
 281:   bool isOImm3() const { return isOImm<3>(); }
 282:   bool isOImm4() const { return isOImm<4>(); }
 283:   bool isOImm5() const { return isOImm<5>(); }
 284:   bool isOImm6() const { return isOImm<6>(); }
 285:   bool isOImm8() const { return isOImm<8>(); }
 286:   bool isOImm12() const { return isOImm<12>(); }
 287:   bool isOImm16() const { return isOImm<16>(); }
 288: 
 289:   bool isSImm8() const { return isSImm<8>(); }
 290: 
 291:   bool isUImm5Shift1() { return isUImm<5, 1>(); }
 292:   bool isUImm5Shift2() { return isUImm<5, 2>(); }
 293:   bool isUImm7Shift1() { return isUImm<7, 1>(); }
 294:   bool isUImm7Shift2() { return isUImm<7, 2>(); }
 295:   bool isUImm7Shift3() { return isUImm<7, 3>(); }
 296:   bool isUImm8Shift2() { return isUImm<8, 2>(); }
 297:   bool isUImm8Shift3() { return isUImm<8, 3>(); }
 298:   bool isUImm8Shift8() { return isUImm<8, 8>(); }
 299:   bool isUImm8Shift16() { return isUImm<8, 16>(); }
 300:   bool isUImm8Shift24() { return isUImm<8, 24>(); }
```

- EN: Function bodies or method definitions such as isOImm3, isOImm4, isOImm5 contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: isOImm3, isOImm4, isOImm5 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 301-320

```cpp
 301:   bool isUImm12Shift1() { return isUImm<12, 1>(); }
 302:   bool isUImm12Shift2() { return isUImm<12, 2>(); }
 303:   bool isUImm16Shift8() { return isUImm<16, 8>(); }
 304:   bool isUImm16Shift16() { return isUImm<16, 16>(); }
 305:   bool isUImm24Shift8() { return isUImm<24, 8>(); }
 306: 
 307:   bool isSImm16Shift1() { return isSImm<16, 1>(); }
 308: 
 309:   bool isCSKYSymbol() const { return isImm(); }
 310: 
 311:   bool isConstpool() const { return isConstPoolOp(); }
 312:   bool isDataSymbol() const { return isConstPoolOp(); }
 313: 
 314:   bool isPSRFlag() const {
 315:     int64_t Imm;
 316:     // Must be of 'immediate' type and a constant.
 317:     if (!isImm() || !evaluateConstantImm(getImm(), Imm))
 318:       return false;
 319: 
 320:     return isUInt<5>(Imm);
```

- EN: Function bodies or method definitions such as isUImm12Shift1, isUImm12Shift2, isUImm16Shift8 contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: isUImm12Shift1, isUImm12Shift2, isUImm16Shift8 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 321-340

```cpp
 321:   }
 322: 
 323:   template <unsigned MIN, unsigned MAX> bool isRegSeqTemplate() const {
 324:     if (!isRegisterSeq())
 325:       return false;
 326: 
 327:     std::pair<unsigned, unsigned> regSeq = getRegSeq();
 328: 
 329:     return MIN <= regSeq.first && regSeq.first <= regSeq.second &&
 330:            regSeq.second <= MAX;
 331:   }
 332: 
 333:   bool isRegSeq() const { return isRegSeqTemplate<CSKY::R0, CSKY::R31>(); }
 334: 
 335:   bool isRegSeqV1() const {
 336:     return isRegSeqTemplate<CSKY::F0_32, CSKY::F15_32>();
 337:   }
 338: 
 339:   bool isRegSeqV2() const {
 340:     return isRegSeqTemplate<CSKY::F0_32, CSKY::F31_32>();
```

- EN: Function bodies or method definitions such as isRegSeq, isRegSeqV1, isRegSeqV2 contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: isRegSeq, isRegSeqV1, isRegSeqV2 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 341-360

```cpp
 341:   }
 342: 
 343:   static bool isLegalRegList(unsigned from, unsigned to) {
 344:     if (from == 0 && to == 0)
 345:       return true;
 346: 
 347:     if (from == to) {
 348:       if (from != CSKY::R4 && from != CSKY::R15 && from != CSKY::R16 &&
 349:           from != CSKY::R28)
 350:         return false;
 351: 
 352:       return true;
 353:     } else {
 354:       if (from != CSKY::R4 && from != CSKY::R16)
 355:         return false;
 356: 
 357:       if (from == CSKY::R4 && to > CSKY::R4 && to < CSKY::R12)
 358:         return true;
 359:       else if (from == CSKY::R16 && to > CSKY::R16 && to < CSKY::R18)
 360:         return true;
```

- EN: Function bodies or method definitions such as isLegalRegList contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: isLegalRegList 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 361-380

```cpp
 361:       else
 362:         return false;
 363:     }
 364:   }
 365: 
 366:   bool isRegList() const {
 367:     if (!isRegisterList())
 368:       return false;
 369: 
 370:     auto regList = getRegList();
 371: 
 372:     if (!isLegalRegList(regList.List1From, regList.List1To))
 373:       return false;
 374:     if (!isLegalRegList(regList.List2From, regList.List2To))
 375:       return false;
 376:     if (!isLegalRegList(regList.List3From, regList.List3To))
 377:       return false;
 378:     if (!isLegalRegList(regList.List4From, regList.List4To))
 379:       return false;
 380: 
```

- EN: Function bodies or method definitions such as isRegList contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: isRegList 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 381-400

```cpp
 381:     return true;
 382:   }
 383: 
 384:   bool isExtImm6() {
 385:     if (!isImm())
 386:       return false;
 387: 
 388:     int64_t Imm;
 389:     bool IsConstantImm = evaluateConstantImm(getImm(), Imm);
 390:     if (!IsConstantImm)
 391:       return false;
 392: 
 393:     int uimm4 = Imm & 0xf;
 394: 
 395:     return isShiftedUInt<6, 0>(Imm) && uimm4 >= 0 && uimm4 <= 14;
 396:   }
 397: 
 398:   /// Gets location of the first token of this operand.
 399:   SMLoc getStartLoc() const override { return StartLoc; }
 400:   /// Gets location of the last token of this operand.
```

- EN: Function bodies or method definitions such as isExtImm6 contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: isExtImm6 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 401-420

```cpp
 401:   SMLoc getEndLoc() const override { return EndLoc; }
 402: 
 403:   MCRegister getReg() const override {
 404:     assert(Kind == Register && "Invalid type access!");
 405:     return Reg.RegNum;
 406:   }
 407: 
 408:   std::pair<MCRegister, MCRegister> getRegSeq() const {
 409:     assert(Kind == RegisterSeq && "Invalid type access!");
 410:     return {RegSeq.RegNumFrom, RegSeq.RegNumTo};
 411:   }
 412: 
 413:   RegListOp getRegList() const {
 414:     assert(Kind == RegisterList && "Invalid type access!");
 415:     return RegList;
 416:   }
 417: 
 418:   const MCExpr *getImm() const {
 419:     assert(Kind == Immediate && "Invalid type access!");
 420:     return Imm.Val;
```

- EN: Function bodies or method definitions such as getRegList contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: getRegList 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 421-440

```cpp
 421:   }
 422: 
 423:   const MCExpr *getConstpoolOp() const {
 424:     assert(Kind == CPOP && "Invalid type access!");
 425:     return CPool.Val;
 426:   }
 427: 
 428:   StringRef getToken() const {
 429:     assert(Kind == Token && "Invalid type access!");
 430:     return Tok;
 431:   }
 432: 
 433:   void print(raw_ostream &OS, const MCAsmInfo &MAI) const override {
 434:     auto RegName = [](MCRegister Reg) {
 435:       if (Reg)
 436:         return CSKYInstPrinter::getRegisterName(Reg);
 437:       else
 438:         return "noreg";
 439:     };
 440: 
```

- EN: Function bodies or method definitions such as getToken contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: getToken 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 441-460

```cpp
 441:     switch (Kind) {
 442:     case CPOP:
 443:       MAI.printExpr(OS, *getConstpoolOp());
 444:       break;
 445:     case Immediate:
 446:       MAI.printExpr(OS, *getImm());
 447:       break;
 448:     case KindTy::Register:
 449:       OS << "<register " << RegName(getReg()) << ">";
 450:       break;
 451:     case RegisterSeq:
 452:       OS << "<register-seq ";
 453:       OS << RegName(getRegSeq().first) << "-" << RegName(getRegSeq().second)
 454:          << ">";
 455:       break;
 456:     case RegisterList:
 457:       OS << "<register-list ";
 458:       OS << RegName(getRegList().List1From) << "-"
 459:          << RegName(getRegList().List1To) << ",";
 460:       OS << RegName(getRegList().List2From) << "-"
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 461-480

```cpp
 461:          << RegName(getRegList().List2To) << ",";
 462:       OS << RegName(getRegList().List3From) << "-"
 463:          << RegName(getRegList().List3To) << ",";
 464:       OS << RegName(getRegList().List4From) << "-"
 465:          << RegName(getRegList().List4To);
 466:       break;
 467:     case Token:
 468:       OS << "'" << getToken() << "'";
 469:       break;
 470:     }
 471:   }
 472: 
 473:   static std::unique_ptr<CSKYOperand> createToken(StringRef Str, SMLoc S) {
 474:     auto Op = std::make_unique<CSKYOperand>(Token);
 475:     Op->Tok = Str;
 476:     Op->StartLoc = S;
 477:     Op->EndLoc = S;
 478:     return Op;
 479:   }
 480: 
```

- EN: Function bodies or method definitions such as createToken contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: createToken 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 481-500

```cpp
 481:   static std::unique_ptr<CSKYOperand> createReg(MCRegister RegNo, SMLoc S,
 482:                                                 SMLoc E) {
 483:     auto Op = std::make_unique<CSKYOperand>(Register);
 484:     Op->Reg.RegNum = RegNo;
 485:     Op->StartLoc = S;
 486:     Op->EndLoc = E;
 487:     return Op;
 488:   }
 489: 
 490:   static std::unique_ptr<CSKYOperand>
 491:   createRegSeq(MCRegister RegNoFrom, MCRegister RegNoTo, SMLoc S) {
 492:     auto Op = std::make_unique<CSKYOperand>(RegisterSeq);
 493:     Op->RegSeq.RegNumFrom = RegNoFrom;
 494:     Op->RegSeq.RegNumTo = RegNoTo;
 495:     Op->StartLoc = S;
 496:     Op->EndLoc = S;
 497:     return Op;
 498:   }
 499: 
 500:   static std::unique_ptr<CSKYOperand>
```

- EN: Function bodies or method definitions such as createReg, createRegSeq contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: createReg, createRegSeq 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 501-520

```cpp
 501:   createRegList(const SmallVector<MCRegister, 4> &reglist, SMLoc S) {
 502:     auto Op = std::make_unique<CSKYOperand>(RegisterList);
 503:     Op->RegList.List1From = 0;
 504:     Op->RegList.List1To = 0;
 505:     Op->RegList.List2From = 0;
 506:     Op->RegList.List2To = 0;
 507:     Op->RegList.List3From = 0;
 508:     Op->RegList.List3To = 0;
 509:     Op->RegList.List4From = 0;
 510:     Op->RegList.List4To = 0;
 511: 
 512:     for (unsigned i = 0; i < reglist.size(); i += 2) {
 513:       if (Op->RegList.List1From == 0) {
 514:         Op->RegList.List1From = reglist[i];
 515:         Op->RegList.List1To = reglist[i + 1];
 516:       } else if (Op->RegList.List2From == 0) {
 517:         Op->RegList.List2From = reglist[i];
 518:         Op->RegList.List2To = reglist[i + 1];
 519:       } else if (Op->RegList.List3From == 0) {
 520:         Op->RegList.List3From = reglist[i];
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 521-540

```cpp
 521:         Op->RegList.List3To = reglist[i + 1];
 522:       } else if (Op->RegList.List4From == 0) {
 523:         Op->RegList.List4From = reglist[i];
 524:         Op->RegList.List4To = reglist[i + 1];
 525:       } else {
 526:         assert(0);
 527:       }
 528:     }
 529: 
 530:     Op->StartLoc = S;
 531:     Op->EndLoc = S;
 532:     return Op;
 533:   }
 534: 
 535:   static std::unique_ptr<CSKYOperand> createImm(const MCExpr *Val, SMLoc S,
 536:                                                 SMLoc E) {
 537:     auto Op = std::make_unique<CSKYOperand>(Immediate);
 538:     Op->Imm.Val = Val;
 539:     Op->StartLoc = S;
 540:     Op->EndLoc = E;
```

- EN: Function bodies or method definitions such as createImm contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: createImm 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 541-560

```cpp
 541:     return Op;
 542:   }
 543: 
 544:   static std::unique_ptr<CSKYOperand> createConstpoolOp(const MCExpr *Val,
 545:                                                         SMLoc S, SMLoc E) {
 546:     auto Op = std::make_unique<CSKYOperand>(CPOP);
 547:     Op->CPool.Val = Val;
 548:     Op->StartLoc = S;
 549:     Op->EndLoc = E;
 550:     return Op;
 551:   }
 552: 
 553:   void addExpr(MCInst &Inst, const MCExpr *Expr) const {
 554:     assert(Expr && "Expr shouldn't be null!");
 555:     if (auto *CE = dyn_cast<MCConstantExpr>(Expr))
 556:       Inst.addOperand(MCOperand::createImm(CE->getValue()));
 557:     else
 558:       Inst.addOperand(MCOperand::createExpr(Expr));
 559:   }
 560: 
```

- EN: Function bodies or method definitions such as createConstpoolOp, addExpr contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: createConstpoolOp, addExpr 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 561-580

```cpp
 561:   // Used by the TableGen Code.
 562:   void addRegOperands(MCInst &Inst, unsigned N) const {
 563:     assert(N == 1 && "Invalid number of operands!");
 564:     Inst.addOperand(MCOperand::createReg(getReg()));
 565:   }
 566: 
 567:   void addImmOperands(MCInst &Inst, unsigned N) const {
 568:     assert(N == 1 && "Invalid number of operands!");
 569:     addExpr(Inst, getImm());
 570:   }
 571: 
 572:   void addConstpoolOperands(MCInst &Inst, unsigned N) const {
 573:     assert(N == 1 && "Invalid number of operands!");
 574:     Inst.addOperand(MCOperand::createExpr(getConstpoolOp()));
 575:   }
 576: 
 577:   void addRegSeqOperands(MCInst &Inst, unsigned N) const {
 578:     assert(N == 2 && "Invalid number of operands!");
 579:     auto regSeq = getRegSeq();
 580: 
```

- EN: Function bodies or method definitions such as addRegOperands, addImmOperands, addConstpoolOperands contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: addRegOperands, addImmOperands, addConstpoolOperands 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 581-600

```cpp
 581:     Inst.addOperand(MCOperand::createReg(regSeq.first));
 582:     Inst.addOperand(MCOperand::createReg(regSeq.second));
 583:   }
 584: 
 585:   static unsigned getListValue(unsigned ListFrom, unsigned ListTo) {
 586:     if (ListFrom == ListTo && ListFrom == CSKY::R15)
 587:       return (1 << 4);
 588:     else if (ListFrom == ListTo && ListFrom == CSKY::R28)
 589:       return (1 << 8);
 590:     else if (ListFrom == CSKY::R4)
 591:       return ListTo - ListFrom + 1;
 592:     else if (ListFrom == CSKY::R16)
 593:       return ((ListTo - ListFrom + 1) << 5);
 594:     else
 595:       return 0;
 596:   }
 597: 
 598:   void addRegListOperands(MCInst &Inst, unsigned N) const {
 599:     assert(N == 1 && "Invalid number of operands!");
 600:     auto regList = getRegList();
```

- EN: Function bodies or method definitions such as getListValue, addRegListOperands contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: getListValue, addRegListOperands 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 601-620

```cpp
 601: 
 602:     unsigned V = 0;
 603: 
 604:     unsigned T = getListValue(regList.List1From, regList.List1To);
 605:     if (T != 0)
 606:       V = V | T;
 607: 
 608:     T = getListValue(regList.List2From, regList.List2To);
 609:     if (T != 0)
 610:       V = V | T;
 611: 
 612:     T = getListValue(regList.List3From, regList.List3To);
 613:     if (T != 0)
 614:       V = V | T;
 615: 
 616:     T = getListValue(regList.List4From, regList.List4To);
 617:     if (T != 0)
 618:       V = V | T;
 619: 
 620:     Inst.addOperand(MCOperand::createImm(V));
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 621-640

```cpp
 621:   }
 622: 
 623:   bool isValidForTie(const CSKYOperand &Other) const {
 624:     if (Kind != Other.Kind)
 625:       return false;
 626: 
 627:     switch (Kind) {
 628:     default:
 629:       llvm_unreachable("Unexpected kind");
 630:       return false;
 631:     case Register:
 632:       return Reg.RegNum == Other.Reg.RegNum;
 633:     }
 634:   }
 635: };
 636: } // end anonymous namespace.
 637: 
 638: #define GET_REGISTER_MATCHER
 639: #define GET_SUBTARGET_FEATURE_NAME
 640: #define GET_MATCHER_IMPLEMENTATION
```

- EN: Function bodies or method definitions such as isValidForTie contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: isValidForTie 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 641-660

```cpp
 641: #define GET_MNEMONIC_SPELL_CHECKER
 642: #include "CSKYGenAsmMatcher.inc"
 643: 
 644: static MCRegister convertFPR32ToFPR64(MCRegister Reg) {
 645:   assert(Reg >= CSKY::F0_32 && Reg <= CSKY::F31_32 && "Invalid register");
 646:   return Reg - CSKY::F0_32 + CSKY::F0_64;
 647: }
 648: 
 649: static std::string CSKYMnemonicSpellCheck(StringRef S, const FeatureBitset &FBS,
 650:                                           unsigned VariantID = 0);
 651: 
 652: bool CSKYAsmParser::generateImmOutOfRangeError(
 653:     OperandVector &Operands, uint64_t ErrorInfo, int64_t Lower, int64_t Upper,
 654:     const Twine &Msg = "immediate must be an integer in the range") {
 655:   SMLoc ErrorLoc = ((CSKYOperand &)*Operands[ErrorInfo]).getStartLoc();
 656:   return Error(ErrorLoc, Msg + " [" + Twine(Lower) + ", " + Twine(Upper) + "]");
 657: }
 658: 
 659: bool CSKYAsmParser::matchAndEmitInstruction(SMLoc IDLoc, unsigned &Opcode,
 660:                                             OperandVector &Operands,
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. Function bodies or method definitions such as convertFPR32ToFPR64, generateImmOutOfRangeError contain the concrete backend logic executed by LLVM passes or MC helpers. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 convertFPR32ToFPR64, generateImmOutOfRangeError 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 661-680

```cpp
 661:                                             MCStreamer &Out,
 662:                                             uint64_t &ErrorInfo,
 663:                                             bool MatchingInlineAsm) {
 664:   MCInst Inst;
 665:   FeatureBitset MissingFeatures;
 666: 
 667:   auto Result = MatchInstructionImpl(Operands, Inst, ErrorInfo, MissingFeatures,
 668:                                      MatchingInlineAsm);
 669:   switch (Result) {
 670:   default:
 671:     break;
 672:   case Match_Success:
 673:     return processInstruction(Inst, IDLoc, Operands, Out);
 674:   case Match_MissingFeature: {
 675:     assert(MissingFeatures.any() && "Unknown missing features!");
 676:     ListSeparator LS;
 677:     std::string Msg = "instruction requires the following: ";
 678:     for (unsigned i = 0, e = MissingFeatures.size(); i != e; ++i) {
 679:       if (MissingFeatures[i]) {
 680:         Msg += LS;
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 681-700

```cpp
 681:         Msg += getSubtargetFeatureName(i);
 682:       }
 683:     }
 684:     return Error(IDLoc, Msg);
 685:   }
 686:   case Match_MnemonicFail: {
 687:     FeatureBitset FBS = ComputeAvailableFeatures(getSTI().getFeatureBits());
 688:     std::string Suggestion =
 689:         CSKYMnemonicSpellCheck(((CSKYOperand &)*Operands[0]).getToken(), FBS);
 690:     return Error(IDLoc, "unrecognized instruction mnemonic" + Suggestion);
 691:   }
 692:   case Match_InvalidTiedOperand:
 693:   case Match_InvalidOperand: {
 694:     SMLoc ErrorLoc = IDLoc;
 695:     if (ErrorInfo != ~0U) {
 696:       if (ErrorInfo >= Operands.size())
 697:         return Error(ErrorLoc, "too few operands for instruction");
 698: 
 699:       ErrorLoc = ((CSKYOperand &)*Operands[ErrorInfo]).getStartLoc();
 700:       if (ErrorLoc == SMLoc())
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 701-720

```cpp
 701:         ErrorLoc = IDLoc;
 702:     }
 703:     return Error(ErrorLoc, "invalid operand for instruction");
 704:   }
 705:   }
 706: 
 707:   // Handle the case when the error message is of specific type
 708:   // other than the generic Match_InvalidOperand, and the
 709:   // corresponding operand is missing.
 710:   if (Result > FIRST_TARGET_MATCH_RESULT_TY) {
 711:     SMLoc ErrorLoc = IDLoc;
 712:     if (ErrorInfo != ~0U && ErrorInfo >= Operands.size())
 713:       return Error(ErrorLoc, "too few operands for instruction");
 714:   }
 715: 
 716:   switch (Result) {
 717:   default:
 718:     break;
 719:   case Match_InvalidSImm8:
 720:     return generateImmOutOfRangeError(Operands, ErrorInfo, -(1 << 7),
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 721-740

```cpp
 721:                                       (1 << 7) - 1);
 722:   case Match_InvalidOImm3:
 723:     return generateImmOutOfRangeError(Operands, ErrorInfo, 1, (1 << 3));
 724:   case Match_InvalidOImm4:
 725:     return generateImmOutOfRangeError(Operands, ErrorInfo, 1, (1 << 4));
 726:   case Match_InvalidOImm5:
 727:     return generateImmOutOfRangeError(Operands, ErrorInfo, 1, (1 << 5));
 728:   case Match_InvalidOImm6:
 729:     return generateImmOutOfRangeError(Operands, ErrorInfo, 1, (1 << 6));
 730:   case Match_InvalidOImm8:
 731:     return generateImmOutOfRangeError(Operands, ErrorInfo, 1, (1 << 8));
 732:   case Match_InvalidOImm12:
 733:     return generateImmOutOfRangeError(Operands, ErrorInfo, 1, (1 << 12));
 734:   case Match_InvalidOImm16:
 735:     return generateImmOutOfRangeError(Operands, ErrorInfo, 1, (1 << 16));
 736:   case Match_InvalidUImm1:
 737:     return generateImmOutOfRangeError(Operands, ErrorInfo, 0, (1 << 1) - 1);
 738:   case Match_InvalidUImm2:
 739:     return generateImmOutOfRangeError(Operands, ErrorInfo, 0, (1 << 2) - 1);
 740:   case Match_InvalidUImm3:
```

- EN: In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 741-760

```cpp
 741:     return generateImmOutOfRangeError(Operands, ErrorInfo, 0, (1 << 3) - 1);
 742:   case Match_InvalidUImm4:
 743:     return generateImmOutOfRangeError(Operands, ErrorInfo, 0, (1 << 4) - 1);
 744:   case Match_InvalidUImm5:
 745:     return generateImmOutOfRangeError(Operands, ErrorInfo, 0, (1 << 5) - 1);
 746:   case Match_InvalidUImm6:
 747:     return generateImmOutOfRangeError(Operands, ErrorInfo, 0, (1 << 6) - 1);
 748:   case Match_InvalidUImm7:
 749:     return generateImmOutOfRangeError(Operands, ErrorInfo, 0, (1 << 7) - 1);
 750:   case Match_InvalidUImm8:
 751:     return generateImmOutOfRangeError(Operands, ErrorInfo, 0, (1 << 8) - 1);
 752:   case Match_InvalidUImm12:
 753:     return generateImmOutOfRangeError(Operands, ErrorInfo, 0, (1 << 12) - 1);
 754:   case Match_InvalidUImm16:
 755:     return generateImmOutOfRangeError(Operands, ErrorInfo, 0, (1 << 16) - 1);
 756:   case Match_InvalidUImm5Shift1:
 757:     return generateImmOutOfRangeError(
 758:         Operands, ErrorInfo, 0, (1 << 5) - 2,
 759:         "immediate must be a multiple of 2 bytes in the range");
 760:   case Match_InvalidUImm12Shift1:
```

- EN: In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 761-780

```cpp
 761:     return generateImmOutOfRangeError(
 762:         Operands, ErrorInfo, 0, (1 << 12) - 2,
 763:         "immediate must be a multiple of 2 bytes in the range");
 764:   case Match_InvalidUImm5Shift2:
 765:     return generateImmOutOfRangeError(
 766:         Operands, ErrorInfo, 0, (1 << 5) - 4,
 767:         "immediate must be a multiple of 4 bytes in the range");
 768:   case Match_InvalidUImm7Shift1:
 769:     return generateImmOutOfRangeError(
 770:         Operands, ErrorInfo, 0, (1 << 7) - 2,
 771:         "immediate must be a multiple of 2 bytes in the range");
 772:   case Match_InvalidUImm7Shift2:
 773:     return generateImmOutOfRangeError(
 774:         Operands, ErrorInfo, 0, (1 << 7) - 4,
 775:         "immediate must be a multiple of 4 bytes in the range");
 776:   case Match_InvalidUImm8Shift2:
 777:     return generateImmOutOfRangeError(
 778:         Operands, ErrorInfo, 0, (1 << 8) - 4,
 779:         "immediate must be a multiple of 4 bytes in the range");
 780:   case Match_InvalidUImm8Shift3:
```

- EN: In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 781-800

```cpp
 781:     return generateImmOutOfRangeError(
 782:         Operands, ErrorInfo, 0, (1 << 8) - 8,
 783:         "immediate must be a multiple of 8 bytes in the range");
 784:   case Match_InvalidUImm8Shift8:
 785:     return generateImmOutOfRangeError(
 786:         Operands, ErrorInfo, 0, (1 << 8) - 256,
 787:         "immediate must be a multiple of 256 bytes in the range");
 788:   case Match_InvalidUImm12Shift2:
 789:     return generateImmOutOfRangeError(
 790:         Operands, ErrorInfo, 0, (1 << 12) - 4,
 791:         "immediate must be a multiple of 4 bytes in the range");
 792:   case Match_InvalidCSKYSymbol: {
 793:     SMLoc ErrorLoc = ((CSKYOperand &)*Operands[ErrorInfo]).getStartLoc();
 794:     return Error(ErrorLoc, "operand must be a symbol name");
 795:   }
 796:   case Match_InvalidConstpool: {
 797:     SMLoc ErrorLoc = ((CSKYOperand &)*Operands[ErrorInfo]).getStartLoc();
 798:     return Error(ErrorLoc, "operand must be a constpool symbol name");
 799:   }
 800:   case Match_InvalidPSRFlag: {
```

- EN: In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 801-820

```cpp
 801:     SMLoc ErrorLoc = ((CSKYOperand &)*Operands[ErrorInfo]).getStartLoc();
 802:     return Error(ErrorLoc, "psrset operand is not valid");
 803:   }
 804:   case Match_InvalidRegSeq: {
 805:     SMLoc ErrorLoc = ((CSKYOperand &)*Operands[ErrorInfo]).getStartLoc();
 806:     return Error(ErrorLoc, "Register sequence is not valid");
 807:   }
 808:   case Match_InvalidRegOutOfRange: {
 809:     SMLoc ErrorLoc = ((CSKYOperand &)*Operands[ErrorInfo]).getStartLoc();
 810:     return Error(ErrorLoc, "register is out of range");
 811:   }
 812:   case Match_RequiresSameSrcAndDst: {
 813:     SMLoc ErrorLoc = ((CSKYOperand &)*Operands[ErrorInfo]).getStartLoc();
 814:     return Error(ErrorLoc, "src and dst operand must be same");
 815:   }
 816:   case Match_InvalidRegList: {
 817:     SMLoc ErrorLoc = ((CSKYOperand &)*Operands[ErrorInfo]).getStartLoc();
 818:     return Error(ErrorLoc, "invalid register list");
 819:   }
 820:   }
```

- EN: In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 821-840

```cpp
 821:   LLVM_DEBUG(dbgs() << "Result = " << Result);
 822:   llvm_unreachable("Unknown match type detected!");
 823: }
 824: 
 825: bool CSKYAsmParser::processLRW(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out) {
 826:   Inst.setLoc(IDLoc);
 827: 
 828:   unsigned Opcode;
 829:   MCOperand Op;
 830:   if (Inst.getOpcode() == CSKY::PseudoLRW16)
 831:     Opcode = CSKY::LRW16;
 832:   else
 833:     Opcode = CSKY::LRW32;
 834: 
 835:   if (Inst.getOperand(1).isImm()) {
 836:     if (isUInt<8>(Inst.getOperand(1).getImm()) &&
 837:         Inst.getOperand(0).getReg() <= CSKY::R7) {
 838:       Opcode = CSKY::MOVI16;
 839:     } else if (getSTI().hasFeature(CSKY::HasE2) &&
 840:                isUInt<16>(Inst.getOperand(1).getImm())) {
```

- EN: Function bodies or method definitions such as processLRW contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: processLRW 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 841-860

```cpp
 841:       Opcode = CSKY::MOVI32;
 842:     } else {
 843:       auto *Expr = getTargetStreamer().addConstantPoolEntry(
 844:           MCConstantExpr::create(Inst.getOperand(1).getImm(), getContext()),
 845:           Inst.getLoc());
 846:       Inst.erase(std::prev(Inst.end()));
 847:       Inst.addOperand(MCOperand::createExpr(Expr));
 848:     }
 849:   } else {
 850:     const MCExpr *AdjustExpr = nullptr;
 851:     if (const auto *CSKYExpr =
 852:             dyn_cast<MCSpecifierExpr>(Inst.getOperand(1).getExpr())) {
 853:       if (CSKYExpr->getSpecifier() == CSKY::S_TLSGD ||
 854:           CSKYExpr->getSpecifier() == CSKY::S_TLSIE ||
 855:           CSKYExpr->getSpecifier() == CSKY::S_TLSLDM) {
 856:         MCSymbol *Dot = getContext().createNamedTempSymbol();
 857:         Out.emitLabel(Dot);
 858:         AdjustExpr = MCSymbolRefExpr::create(Dot, getContext());
 859:       }
 860:     }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 861-880

```cpp
 861:     auto *Expr = getTargetStreamer().addConstantPoolEntry(
 862:         Inst.getOperand(1).getExpr(), Inst.getLoc(), AdjustExpr);
 863:     Inst.erase(std::prev(Inst.end()));
 864:     Inst.addOperand(MCOperand::createExpr(Expr));
 865:   }
 866: 
 867:   Inst.setOpcode(Opcode);
 868: 
 869:   Out.emitInstruction(Inst, getSTI());
 870:   return false;
 871: }
 872: 
 873: bool CSKYAsmParser::processJSRI(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out) {
 874:   Inst.setLoc(IDLoc);
 875: 
 876:   if (Inst.getOperand(0).isImm()) {
 877:     const MCExpr *Expr = getTargetStreamer().addConstantPoolEntry(
 878:         MCConstantExpr::create(Inst.getOperand(0).getImm(), getContext()),
 879:         Inst.getLoc());
 880:     Inst.setOpcode(CSKY::JSRI32);
```

- EN: Function bodies or method definitions such as processJSRI contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: processJSRI 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 881-900

```cpp
 881:     Inst.erase(std::prev(Inst.end()));
 882:     Inst.addOperand(MCOperand::createExpr(Expr));
 883:   } else {
 884:     const MCExpr *Expr = getTargetStreamer().addConstantPoolEntry(
 885:         Inst.getOperand(0).getExpr(), Inst.getLoc());
 886:     Inst.setOpcode(CSKY::JBSR32);
 887:     Inst.addOperand(MCOperand::createExpr(Expr));
 888:   }
 889: 
 890:   Out.emitInstruction(Inst, getSTI());
 891:   return false;
 892: }
 893: 
 894: bool CSKYAsmParser::processJMPI(MCInst &Inst, SMLoc IDLoc, MCStreamer &Out) {
 895:   Inst.setLoc(IDLoc);
 896: 
 897:   if (Inst.getOperand(0).isImm()) {
 898:     const MCExpr *Expr = getTargetStreamer().addConstantPoolEntry(
 899:         MCConstantExpr::create(Inst.getOperand(0).getImm(), getContext()),
 900:         Inst.getLoc());
```

- EN: Function bodies or method definitions such as processJMPI contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: processJMPI 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 901-920

```cpp
 901:     Inst.setOpcode(CSKY::JMPI32);
 902:     Inst.erase(std::prev(Inst.end()));
 903:     Inst.addOperand(MCOperand::createExpr(Expr));
 904:   } else {
 905:     const MCExpr *Expr = getTargetStreamer().addConstantPoolEntry(
 906:         Inst.getOperand(0).getExpr(), Inst.getLoc());
 907:     Inst.setOpcode(CSKY::JBR32);
 908:     Inst.addOperand(MCOperand::createExpr(Expr));
 909:   }
 910: 
 911:   Out.emitInstruction(Inst, getSTI());
 912:   return false;
 913: }
 914: 
 915: bool CSKYAsmParser::processInstruction(MCInst &Inst, SMLoc IDLoc,
 916:                                        OperandVector &Operands,
 917:                                        MCStreamer &Out) {
 918: 
 919:   switch (Inst.getOpcode()) {
 920:   default:
```

- EN: Function bodies or method definitions such as processInstruction contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: processInstruction 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 921-940

```cpp
 921:     break;
 922:   case CSKY::LDQ32:
 923:   case CSKY::STQ32:
 924:     if (Inst.getOperand(1).getReg() != CSKY::R4 ||
 925:         Inst.getOperand(2).getReg() != CSKY::R7) {
 926:       return Error(IDLoc, "Register sequence is not valid. 'r4-r7' expected");
 927:     }
 928:     Inst.setOpcode(Inst.getOpcode() == CSKY::LDQ32 ? CSKY::LDM32 : CSKY::STM32);
 929:     break;
 930:   case CSKY::SEXT32:
 931:   case CSKY::ZEXT32:
 932:     if (Inst.getOperand(2).getImm() < Inst.getOperand(3).getImm())
 933:       return Error(IDLoc, "msb must be greater or equal to lsb");
 934:     break;
 935:   case CSKY::INS32:
 936:     if (Inst.getOperand(3).getImm() < Inst.getOperand(4).getImm())
 937:       return Error(IDLoc, "msb must be greater or equal to lsb");
 938:     break;
 939:   case CSKY::IDLY32:
 940:     if (Inst.getOperand(0).getImm() > 32 || Inst.getOperand(0).getImm() < 0)
```

- EN: Function bodies or method definitions such as if contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: if 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 941-960

```cpp
 941:       return Error(IDLoc, "n must be in range [0,32]");
 942:     break;
 943:   case CSKY::ADDC32:
 944:   case CSKY::SUBC32:
 945:   case CSKY::ADDC16:
 946:   case CSKY::SUBC16:
 947:     Inst.erase(std::next(Inst.begin()));
 948:     Inst.erase(std::prev(Inst.end()));
 949:     Inst.insert(std::next(Inst.begin()), MCOperand::createReg(CSKY::C));
 950:     Inst.insert(Inst.end(), MCOperand::createReg(CSKY::C));
 951:     break;
 952:   case CSKY::CMPNEI32:
 953:   case CSKY::CMPNEI16:
 954:   case CSKY::CMPNE32:
 955:   case CSKY::CMPNE16:
 956:   case CSKY::CMPHSI32:
 957:   case CSKY::CMPHSI16:
 958:   case CSKY::CMPHS32:
 959:   case CSKY::CMPHS16:
 960:   case CSKY::CMPLTI32:
```

- EN: In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 961-980

```cpp
 961:   case CSKY::CMPLTI16:
 962:   case CSKY::CMPLT32:
 963:   case CSKY::CMPLT16:
 964:   case CSKY::BTSTI32:
 965:     Inst.erase(Inst.begin());
 966:     Inst.insert(Inst.begin(), MCOperand::createReg(CSKY::C));
 967:     break;
 968:   case CSKY::MVCV32:
 969:     Inst.erase(std::next(Inst.begin()));
 970:     Inst.insert(Inst.end(), MCOperand::createReg(CSKY::C));
 971:     break;
 972:   case CSKY::PseudoLRW16:
 973:   case CSKY::PseudoLRW32:
 974:     return processLRW(Inst, IDLoc, Out);
 975:   case CSKY::PseudoJSRI32:
 976:     return processJSRI(Inst, IDLoc, Out);
 977:   case CSKY::PseudoJMPI32:
 978:     return processJMPI(Inst, IDLoc, Out);
 979:   case CSKY::JBSR32:
 980:   case CSKY::JBR16:
```

- EN: In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 981-1000

```cpp
 981:   case CSKY::JBT16:
 982:   case CSKY::JBF16:
 983:   case CSKY::JBR32:
 984:   case CSKY::JBT32:
 985:   case CSKY::JBF32:
 986:     unsigned Num = Inst.getNumOperands() - 1;
 987:     assert(Inst.getOperand(Num).isExpr());
 988: 
 989:     const MCExpr *Expr = getTargetStreamer().addConstantPoolEntry(
 990:         Inst.getOperand(Num).getExpr(), Inst.getLoc());
 991: 
 992:     Inst.addOperand(MCOperand::createExpr(Expr));
 993:     break;
 994:   }
 995: 
 996:   emitToStreamer(Out, Inst);
 997:   return false;
 998: }
 999: 
1000: // Attempts to match Name as a register (either using the default name or
```

- EN: In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 1001-1020

```cpp
1001: // alternative ABI names), setting RegNo to the matching register. Upon
1002: // failure, returns true and sets RegNo to 0.
1003: static bool matchRegisterNameHelper(const MCSubtargetInfo &STI, MCRegister &Reg,
1004:                                     StringRef Name) {
1005:   Reg = MatchRegisterName(Name);
1006: 
1007:   if (Reg == CSKY::NoRegister)
1008:     Reg = MatchRegisterAltName(Name);
1009: 
1010:   return Reg == CSKY::NoRegister;
1011: }
1012: 
1013: bool CSKYAsmParser::parseRegister(MCRegister &Reg, SMLoc &StartLoc,
1014:                                   SMLoc &EndLoc) {
1015:   const AsmToken &Tok = getParser().getTok();
1016:   StartLoc = Tok.getLoc();
1017:   EndLoc = Tok.getEndLoc();
1018:   StringRef Name = getLexer().getTok().getIdentifier();
1019: 
1020:   if (!matchRegisterNameHelper(getSTI(), Reg, Name)) {
```

- EN: Function bodies or method definitions such as matchRegisterNameHelper, parseRegister contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: matchRegisterNameHelper, parseRegister 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 1021-1040

```cpp
1021:     getParser().Lex(); // Eat identifier token.
1022:     return false;
1023:   }
1024: 
1025:   return true;
1026: }
1027: 
1028: ParseStatus CSKYAsmParser::parseRegister(OperandVector &Operands) {
1029:   SMLoc S = getLoc();
1030:   SMLoc E = SMLoc::getFromPointer(S.getPointer() - 1);
1031: 
1032:   switch (getLexer().getKind()) {
1033:   default:
1034:     return ParseStatus::NoMatch;
1035:   case AsmToken::Identifier: {
1036:     StringRef Name = getLexer().getTok().getIdentifier();
1037:     MCRegister Reg;
1038: 
1039:     if (matchRegisterNameHelper(getSTI(), Reg, Name))
1040:       return ParseStatus::NoMatch;
```

- EN: Function bodies or method definitions such as parseRegister contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: parseRegister 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 1041-1060

```cpp
1041: 
1042:     getLexer().Lex();
1043:     Operands.push_back(CSKYOperand::createReg(Reg, S, E));
1044: 
1045:     return ParseStatus::Success;
1046:   }
1047:   }
1048: }
1049: 
1050: ParseStatus CSKYAsmParser::parseBaseRegImm(OperandVector &Operands) {
1051:   assert(getLexer().is(AsmToken::LParen));
1052: 
1053:   Operands.push_back(CSKYOperand::createToken("(", getLoc()));
1054: 
1055:   auto Tok = getParser().Lex(); // Eat '('
1056: 
1057:   if (!parseRegister(Operands).isSuccess()) {
1058:     getLexer().UnLex(Tok);
1059:     Operands.pop_back();
1060:     return ParseStatus::NoMatch;
```

- EN: Function bodies or method definitions such as parseBaseRegImm contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: parseBaseRegImm 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 1061-1080

```cpp
1061:   }
1062: 
1063:   if (getLexer().is(AsmToken::RParen)) {
1064:     Operands.push_back(CSKYOperand::createToken(")", getLoc()));
1065:     getParser().Lex(); // Eat ')'
1066:     return ParseStatus::Success;
1067:   }
1068: 
1069:   if (getLexer().isNot(AsmToken::Comma))
1070:     return Error(getLoc(), "expected ','");
1071: 
1072:   getParser().Lex(); // Eat ','
1073: 
1074:   if (parseRegister(Operands).isSuccess()) {
1075:     if (getLexer().isNot(AsmToken::LessLess))
1076:       return Error(getLoc(), "expected '<<'");
1077: 
1078:     Operands.push_back(CSKYOperand::createToken("<<", getLoc()));
1079: 
1080:     getParser().Lex(); // Eat '<<'
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 1081-1100

```cpp
1081: 
1082:     if (!parseImmediate(Operands).isSuccess())
1083:       return Error(getLoc(), "expected imm");
1084: 
1085:   } else if (!parseImmediate(Operands).isSuccess()) {
1086:     return Error(getLoc(), "expected imm");
1087:   }
1088: 
1089:   if (getLexer().isNot(AsmToken::RParen))
1090:     return Error(getLoc(), "expected ')'");
1091: 
1092:   Operands.push_back(CSKYOperand::createToken(")", getLoc()));
1093: 
1094:   getParser().Lex(); // Eat ')'
1095: 
1096:   return ParseStatus::Success;
1097: }
1098: 
1099: ParseStatus CSKYAsmParser::parseImmediate(OperandVector &Operands) {
1100:   switch (getLexer().getKind()) {
```

- EN: Function bodies or method definitions such as parseImmediate contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: parseImmediate 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 1101-1120

```cpp
1101:   default:
1102:     return ParseStatus::NoMatch;
1103:   case AsmToken::LParen:
1104:   case AsmToken::Minus:
1105:   case AsmToken::Plus:
1106:   case AsmToken::Integer:
1107:   case AsmToken::String:
1108:     break;
1109:   }
1110: 
1111:   const MCExpr *IdVal;
1112:   SMLoc S = getLoc();
1113:   if (getParser().parseExpression(IdVal))
1114:     return Error(getLoc(), "unknown expression");
1115: 
1116:   SMLoc E = SMLoc::getFromPointer(S.getPointer() - 1);
1117:   Operands.push_back(CSKYOperand::createImm(IdVal, S, E));
1118:   return ParseStatus::Success;
1119: }
1120: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 1121-1140

```cpp
1121: /// Looks at a token type and creates the relevant operand from this
1122: /// information, adding to Operands. If operand was parsed, returns false, else
1123: /// true.
1124: bool CSKYAsmParser::parseOperand(OperandVector &Operands, StringRef Mnemonic) {
1125:   // Check if the current operand has a custom associated parser, if so, try to
1126:   // custom parse the operand, or fallback to the general approach.
1127:   ParseStatus Result =
1128:       MatchOperandParserImpl(Operands, Mnemonic, /*ParseForAllFeatures=*/true);
1129:   if (Result.isSuccess())
1130:     return false;
1131:   if (Result.isFailure())
1132:     return true;
1133: 
1134:   // Attempt to parse token as register
1135:   auto Res = parseRegister(Operands);
1136:   if (Res.isSuccess())
1137:     return false;
1138:   if (Res.isFailure())
1139:     return true;
1140: 
```

- EN: Function bodies or method definitions such as parseOperand contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: parseOperand 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 1141-1160

```cpp
1141:   // Attempt to parse token as (register, imm)
1142:   if (getLexer().is(AsmToken::LParen)) {
1143:     Res = parseBaseRegImm(Operands);
1144:     if (Res.isSuccess())
1145:       return false;
1146:     if (Res.isFailure())
1147:       return true;
1148:   }
1149: 
1150:   Res = parseImmediate(Operands);
1151:   if (Res.isSuccess())
1152:     return false;
1153:   if (Res.isFailure())
1154:     return true;
1155: 
1156:   // Finally we have exhausted all options and must declare defeat.
1157:   Error(getLoc(), "unknown operand");
1158:   return true;
1159: }
1160: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 1161-1180

```cpp
1161: ParseStatus CSKYAsmParser::parseCSKYSymbol(OperandVector &Operands) {
1162:   SMLoc S = getLoc();
1163:   SMLoc E = SMLoc::getFromPointer(S.getPointer() - 1);
1164:   const MCExpr *Res;
1165: 
1166:   if (getLexer().getKind() != AsmToken::Identifier)
1167:     return ParseStatus::NoMatch;
1168: 
1169:   StringRef Identifier;
1170:   AsmToken Tok = getLexer().getTok();
1171: 
1172:   if (getParser().parseIdentifier(Identifier))
1173:     return Error(getLoc(), "unknown identifier");
1174: 
1175:   CSKY::Specifier Kind = CSKY::S_None;
1176:   if (Identifier.consume_back("@GOT"))
1177:     Kind = CSKY::S_GOT;
1178:   else if (Identifier.consume_back("@GOTOFF"))
1179:     Kind = CSKY::S_GOTOFF;
1180:   else if (Identifier.consume_back("@PLT"))
```

- EN: Function bodies or method definitions such as parseCSKYSymbol contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: parseCSKYSymbol 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 1181-1200

```cpp
1181:     Kind = CSKY::S_PLT;
1182:   else if (Identifier.consume_back("@GOTPC"))
1183:     Kind = CSKY::S_GOTPC;
1184:   else if (Identifier.consume_back("@TLSGD32"))
1185:     Kind = CSKY::S_TLSGD;
1186:   else if (Identifier.consume_back("@GOTTPOFF"))
1187:     Kind = CSKY::S_TLSIE;
1188:   else if (Identifier.consume_back("@TPOFF"))
1189:     Kind = CSKY::S_TLSLE;
1190:   else if (Identifier.consume_back("@TLSLDM32"))
1191:     Kind = CSKY::S_TLSLDM;
1192:   else if (Identifier.consume_back("@TLSLDO32"))
1193:     Kind = CSKY::S_TLSLDO;
1194: 
1195:   MCSymbol *Sym = getContext().getInlineAsmLabel(Identifier);
1196: 
1197:   if (!Sym)
1198:     Sym = getContext().getOrCreateSymbol(Identifier);
1199: 
1200:   if (Sym->isVariable()) {
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 1201-1220

```cpp
1201:     const MCExpr *V = Sym->getVariableValue();
1202:     if (!isa<MCSymbolRefExpr>(V)) {
1203:       getLexer().UnLex(Tok); // Put back if it's not a bare symbol.
1204:       return Error(getLoc(), "unknown symbol");
1205:     }
1206:     Res = V;
1207:   } else
1208:     Res = MCSymbolRefExpr::create(Sym, getContext());
1209: 
1210:   MCBinaryExpr::Opcode Opcode;
1211:   switch (getLexer().getKind()) {
1212:   default:
1213:     if (Kind != CSKY::S_None)
1214:       Res = MCSpecifierExpr::create(Res, Kind, getContext());
1215: 
1216:     Operands.push_back(CSKYOperand::createImm(Res, S, E));
1217:     return ParseStatus::Success;
1218:   case AsmToken::Plus:
1219:     Opcode = MCBinaryExpr::Add;
1220:     break;
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 1221-1240

```cpp
1221:   case AsmToken::Minus:
1222:     Opcode = MCBinaryExpr::Sub;
1223:     break;
1224:   }
1225: 
1226:   getLexer().Lex(); // eat + or -
1227: 
1228:   const MCExpr *Expr;
1229:   if (getParser().parseExpression(Expr))
1230:     return Error(getLoc(), "unknown expression");
1231:   Res = MCBinaryExpr::create(Opcode, Res, Expr, getContext());
1232:   Operands.push_back(CSKYOperand::createImm(Res, S, E));
1233:   return ParseStatus::Success;
1234: }
1235: 
1236: ParseStatus CSKYAsmParser::parseDataSymbol(OperandVector &Operands) {
1237:   SMLoc S = getLoc();
1238:   SMLoc E = SMLoc::getFromPointer(S.getPointer() - 1);
1239:   const MCExpr *Res;
1240: 
```

- EN: Function bodies or method definitions such as parseDataSymbol contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: parseDataSymbol 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 1241-1260

```cpp
1241:   if (!parseOptionalToken(AsmToken::LBrac))
1242:     return ParseStatus::NoMatch;
1243:   if (getLexer().getKind() != AsmToken::Identifier) {
1244:     const MCExpr *Expr;
1245:     if (getParser().parseExpression(Expr))
1246:       return Error(getLoc(), "unknown expression");
1247: 
1248:     if (parseToken(AsmToken::RBrac, "expected ']'"))
1249:       return ParseStatus::Failure;
1250: 
1251:     Operands.push_back(CSKYOperand::createConstpoolOp(Expr, S, E));
1252:     return ParseStatus::Success;
1253:   }
1254: 
1255:   AsmToken Tok = getLexer().getTok();
1256:   StringRef Identifier;
1257: 
1258:   if (getParser().parseIdentifier(Identifier))
1259:     return Error(getLoc(), "unknown identifier " + Identifier);
1260: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 1261-1280

```cpp
1261:   CSKY::Specifier Kind = CSKY::S_None;
1262:   if (Identifier.consume_back("@GOT"))
1263:     Kind = CSKY::S_GOT_IMM18_BY4;
1264:   else if (Identifier.consume_back("@PLT"))
1265:     Kind = CSKY::S_PLT_IMM18_BY4;
1266: 
1267:   MCSymbol *Sym = getContext().getInlineAsmLabel(Identifier);
1268: 
1269:   if (!Sym)
1270:     Sym = getContext().getOrCreateSymbol(Identifier);
1271: 
1272:   if (Sym->isVariable()) {
1273:     const MCExpr *V = Sym->getVariableValue();
1274:     if (!isa<MCSymbolRefExpr>(V)) {
1275:       getLexer().UnLex(Tok); // Put back if it's not a bare symbol.
1276:       return Error(getLoc(), "unknown symbol");
1277:     }
1278:     Res = V;
1279:   } else {
1280:     Res = MCSymbolRefExpr::create(Sym, getContext());
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 1281-1300

```cpp
1281:   }
1282: 
1283:   MCBinaryExpr::Opcode Opcode;
1284:   switch (getLexer().getKind()) {
1285:   default:
1286:     return Error(getLoc(), "unknown symbol");
1287:   case AsmToken::RBrac:
1288: 
1289:     getLexer().Lex(); // Eat ']'.
1290: 
1291:     if (Kind != CSKY::S_None)
1292:       Res = MCSpecifierExpr::create(Res, Kind, getContext());
1293: 
1294:     Operands.push_back(CSKYOperand::createConstpoolOp(Res, S, E));
1295:     return ParseStatus::Success;
1296:   case AsmToken::Plus:
1297:     Opcode = MCBinaryExpr::Add;
1298:     break;
1299:   case AsmToken::Minus:
1300:     Opcode = MCBinaryExpr::Sub;
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 1301-1320

```cpp
1301:     break;
1302:   }
1303: 
1304:   getLexer().Lex(); // eat + or -
1305: 
1306:   const MCExpr *Expr;
1307:   if (getParser().parseExpression(Expr))
1308:     return Error(getLoc(), "unknown expression");
1309:   if (parseToken(AsmToken::RBrac, "expected ']'"))
1310:     return ParseStatus::Failure;
1311: 
1312:   Res = MCBinaryExpr::create(Opcode, Res, Expr, getContext());
1313:   Operands.push_back(CSKYOperand::createConstpoolOp(Res, S, E));
1314:   return ParseStatus::Success;
1315: }
1316: 
1317: ParseStatus CSKYAsmParser::parseConstpoolSymbol(OperandVector &Operands) {
1318:   SMLoc S = getLoc();
1319:   SMLoc E = SMLoc::getFromPointer(S.getPointer() - 1);
1320:   const MCExpr *Res;
```

- EN: Function bodies or method definitions such as parseConstpoolSymbol contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: parseConstpoolSymbol 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 1321-1340

```cpp
1321: 
1322:   if (!parseOptionalToken(AsmToken::LBrac))
1323:     return ParseStatus::NoMatch;
1324: 
1325:   if (getLexer().getKind() != AsmToken::Identifier) {
1326:     const MCExpr *Expr;
1327:     if (getParser().parseExpression(Expr))
1328:       return Error(getLoc(), "unknown expression");
1329:     if (parseToken(AsmToken::RBrac))
1330:       return ParseStatus::Failure;
1331: 
1332:     Operands.push_back(CSKYOperand::createConstpoolOp(Expr, S, E));
1333:     return ParseStatus::Success;
1334:   }
1335: 
1336:   AsmToken Tok = getLexer().getTok();
1337:   StringRef Identifier;
1338: 
1339:   if (getParser().parseIdentifier(Identifier))
1340:     return Error(getLoc(), "unknown identifier");
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 1341-1360

```cpp
1341: 
1342:   MCSymbol *Sym = getContext().getInlineAsmLabel(Identifier);
1343: 
1344:   if (!Sym)
1345:     Sym = getContext().getOrCreateSymbol(Identifier);
1346: 
1347:   if (Sym->isVariable()) {
1348:     const MCExpr *V = Sym->getVariableValue();
1349:     if (!isa<MCSymbolRefExpr>(V)) {
1350:       getLexer().UnLex(Tok); // Put back if it's not a bare symbol.
1351:       return Error(getLoc(), "unknown symbol");
1352:     }
1353:     Res = V;
1354:   } else {
1355:     Res = MCSymbolRefExpr::create(Sym, getContext());
1356:   }
1357: 
1358:   MCBinaryExpr::Opcode Opcode;
1359:   switch (getLexer().getKind()) {
1360:   default:
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 1361-1380

```cpp
1361:     return Error(getLoc(), "unknown symbol");
1362:   case AsmToken::RBrac:
1363: 
1364:     getLexer().Lex(); // Eat ']'.
1365: 
1366:     Operands.push_back(CSKYOperand::createConstpoolOp(Res, S, E));
1367:     return ParseStatus::Success;
1368:   case AsmToken::Plus:
1369:     Opcode = MCBinaryExpr::Add;
1370:     break;
1371:   case AsmToken::Minus:
1372:     Opcode = MCBinaryExpr::Sub;
1373:     break;
1374:   }
1375: 
1376:   getLexer().Lex(); // eat + or -
1377: 
1378:   const MCExpr *Expr;
1379:   if (getParser().parseExpression(Expr))
1380:     return Error(getLoc(), "unknown expression");
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 1381-1400

```cpp
1381:   if (parseToken(AsmToken::RBrac, "expected ']'"))
1382:     return ParseStatus::Failure;
1383: 
1384:   Res = MCBinaryExpr::create(Opcode, Res, Expr, getContext());
1385:   Operands.push_back(CSKYOperand::createConstpoolOp(Res, S, E));
1386:   return ParseStatus::Success;
1387: }
1388: 
1389: ParseStatus CSKYAsmParser::parsePSRFlag(OperandVector &Operands) {
1390:   SMLoc S = getLoc();
1391:   SMLoc E = SMLoc::getFromPointer(S.getPointer() - 1);
1392: 
1393:   unsigned Flag = 0;
1394: 
1395:   while (getLexer().isNot(AsmToken::EndOfStatement)) {
1396:     StringRef Identifier;
1397:     if (getParser().parseIdentifier(Identifier))
1398:       return Error(getLoc(), "unknown identifier " + Identifier);
1399: 
1400:     if (Identifier == "sie")
```

- EN: Function bodies or method definitions such as parsePSRFlag contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: parsePSRFlag 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 1401-1420

```cpp
1401:       Flag = (1 << 4) | Flag;
1402:     else if (Identifier == "ee")
1403:       Flag = (1 << 3) | Flag;
1404:     else if (Identifier == "ie")
1405:       Flag = (1 << 2) | Flag;
1406:     else if (Identifier == "fe")
1407:       Flag = (1 << 1) | Flag;
1408:     else if (Identifier == "af")
1409:       Flag = (1 << 0) | Flag;
1410:     else
1411:       return Error(getLoc(), "expected " + Identifier);
1412: 
1413:     if (getLexer().is(AsmToken::EndOfStatement))
1414:       break;
1415: 
1416:     if (parseToken(AsmToken::Comma, "expected ','"))
1417:       return ParseStatus::Failure;
1418:   }
1419: 
1420:   Operands.push_back(
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 1421-1440

```cpp
1421:       CSKYOperand::createImm(MCConstantExpr::create(Flag, getContext()), S, E));
1422:   return ParseStatus::Success;
1423: }
1424: 
1425: ParseStatus CSKYAsmParser::parseRegSeq(OperandVector &Operands) {
1426:   SMLoc S = getLoc();
1427: 
1428:   if (!parseRegister(Operands).isSuccess())
1429:     return ParseStatus::NoMatch;
1430: 
1431:   auto Ry = Operands.back()->getReg();
1432:   Operands.pop_back();
1433: 
1434:   if (parseToken(AsmToken::Minus, "expected '-'"))
1435:     return ParseStatus::Failure;
1436:   if (!parseRegister(Operands).isSuccess())
1437:     return Error(getLoc(), "invalid register");
1438: 
1439:   auto Rz = Operands.back()->getReg();
1440:   Operands.pop_back();
```

- EN: Function bodies or method definitions such as parseRegSeq contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: parseRegSeq 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 1441-1460

```cpp
1441: 
1442:   Operands.push_back(CSKYOperand::createRegSeq(Ry, Rz, S));
1443:   return ParseStatus::Success;
1444: }
1445: 
1446: ParseStatus CSKYAsmParser::parseRegList(OperandVector &Operands) {
1447:   SMLoc S = getLoc();
1448:   SmallVector<MCRegister, 4> reglist;
1449:   while (true) {
1450: 
1451:     if (!parseRegister(Operands).isSuccess())
1452:       return Error(getLoc(), "invalid register");
1453: 
1454:     auto Ry = Operands.back()->getReg();
1455:     Operands.pop_back();
1456: 
1457:     if (parseOptionalToken(AsmToken::Minus)) {
1458:       if (!parseRegister(Operands).isSuccess())
1459:         return Error(getLoc(), "invalid register");
1460: 
```

- EN: Function bodies or method definitions such as parseRegList contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: parseRegList 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 1461-1480

```cpp
1461:       auto Rz = Operands.back()->getReg();
1462:       Operands.pop_back();
1463: 
1464:       reglist.push_back(Ry);
1465:       reglist.push_back(Rz);
1466: 
1467:       if (getLexer().is(AsmToken::EndOfStatement))
1468:         break;
1469:       (void)parseOptionalToken(AsmToken::Comma);
1470:     } else if (parseOptionalToken(AsmToken::Comma)) {
1471:       reglist.push_back(Ry);
1472:       reglist.push_back(Ry);
1473:     } else if (getLexer().is(AsmToken::EndOfStatement)) {
1474:       reglist.push_back(Ry);
1475:       reglist.push_back(Ry);
1476:       break;
1477:     } else {
1478:       return Error(getLoc(), "invalid register list");
1479:     }
1480:   }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 1481-1500

```cpp
1481: 
1482:   Operands.push_back(CSKYOperand::createRegList(reglist, S));
1483:   return ParseStatus::Success;
1484: }
1485: 
1486: bool CSKYAsmParser::parseInstruction(ParseInstructionInfo &Info, StringRef Name,
1487:                                      SMLoc NameLoc, OperandVector &Operands) {
1488:   // First operand is token for instruction.
1489:   Operands.push_back(CSKYOperand::createToken(Name, NameLoc));
1490: 
1491:   // If there are no more operands, then finish.
1492:   if (getLexer().is(AsmToken::EndOfStatement))
1493:     return false;
1494: 
1495:   // Parse first operand.
1496:   if (parseOperand(Operands, Name))
1497:     return true;
1498: 
1499:   // Parse until end of statement, consuming commas between operands.
1500:   while (parseOptionalToken(AsmToken::Comma))
```

- EN: Function bodies or method definitions such as parseInstruction contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: parseInstruction 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 1501-1520

```cpp
1501:     if (parseOperand(Operands, Name))
1502:       return true;
1503: 
1504:   if (getLexer().isNot(AsmToken::EndOfStatement)) {
1505:     SMLoc Loc = getLexer().getLoc();
1506:     getParser().eatToEndOfStatement();
1507:     return Error(Loc, "unexpected token");
1508:   }
1509: 
1510:   getParser().Lex(); // Consume the EndOfStatement.
1511:   return false;
1512: }
1513: 
1514: ParseStatus CSKYAsmParser::tryParseRegister(MCRegister &Reg, SMLoc &StartLoc,
1515:                                             SMLoc &EndLoc) {
1516:   const AsmToken &Tok = getParser().getTok();
1517:   StartLoc = Tok.getLoc();
1518:   EndLoc = Tok.getEndLoc();
1519: 
1520:   StringRef Name = getLexer().getTok().getIdentifier();
```

- EN: Function bodies or method definitions such as tryParseRegister contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: tryParseRegister 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 1521-1540

```cpp
1521: 
1522:   if (matchRegisterNameHelper(getSTI(), Reg, Name))
1523:     return ParseStatus::NoMatch;
1524: 
1525:   getParser().Lex(); // Eat identifier token.
1526:   return ParseStatus::Success;
1527: }
1528: 
1529: ParseStatus CSKYAsmParser::parseDirective(AsmToken DirectiveID) {
1530:   StringRef IDVal = DirectiveID.getString();
1531: 
1532:   if (IDVal == ".csky_attribute")
1533:     return parseDirectiveAttribute();
1534: 
1535:   return ParseStatus::NoMatch;
1536: }
1537: 
1538: /// parseDirectiveAttribute
1539: ///  ::= .attribute expression ',' ( expression | "string" )
1540: bool CSKYAsmParser::parseDirectiveAttribute() {
```

- EN: Function bodies or method definitions such as parseDirective, parseDirectiveAttribute contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: parseDirective, parseDirectiveAttribute 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 1541-1560

```cpp
1541:   MCAsmParser &Parser = getParser();
1542:   int64_t Tag;
1543:   SMLoc TagLoc;
1544:   TagLoc = Parser.getTok().getLoc();
1545:   if (Parser.getTok().is(AsmToken::Identifier)) {
1546:     StringRef Name = Parser.getTok().getIdentifier();
1547:     std::optional<unsigned> Ret =
1548:         ELFAttrs::attrTypeFromString(Name, CSKYAttrs::getCSKYAttributeTags());
1549:     if (!Ret)
1550:       return Error(TagLoc, "attribute name not recognised: " + Name);
1551:     Tag = *Ret;
1552:     Parser.Lex();
1553:   } else {
1554:     const MCExpr *AttrExpr;
1555: 
1556:     TagLoc = Parser.getTok().getLoc();
1557:     if (Parser.parseExpression(AttrExpr))
1558:       return true;
1559: 
1560:     const MCConstantExpr *CE = dyn_cast<MCConstantExpr>(AttrExpr);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 1561-1580

```cpp
1561:     if (!CE)
1562:       return Error(TagLoc, "expected numeric constant");
1563: 
1564:     Tag = CE->getValue();
1565:   }
1566: 
1567:   if (Parser.parseComma())
1568:     return true;
1569: 
1570:   StringRef StringValue;
1571:   int64_t IntegerValue = 0;
1572:   bool IsIntegerValue = ((Tag != CSKYAttrs::CSKY_ARCH_NAME) &&
1573:                          (Tag != CSKYAttrs::CSKY_CPU_NAME) &&
1574:                          (Tag != CSKYAttrs::CSKY_FPU_NUMBER_MODULE));
1575: 
1576:   SMLoc ValueExprLoc = Parser.getTok().getLoc();
1577:   if (IsIntegerValue) {
1578:     const MCExpr *ValueExpr;
1579:     if (Parser.parseExpression(ValueExpr))
1580:       return true;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 1581-1600

```cpp
1581: 
1582:     const MCConstantExpr *CE = dyn_cast<MCConstantExpr>(ValueExpr);
1583:     if (!CE)
1584:       return Error(ValueExprLoc, "expected numeric constant");
1585:     IntegerValue = CE->getValue();
1586:   } else {
1587:     if (Parser.getTok().isNot(AsmToken::String))
1588:       return Error(Parser.getTok().getLoc(), "expected string constant");
1589: 
1590:     StringValue = Parser.getTok().getStringContents();
1591:     Parser.Lex();
1592:   }
1593: 
1594:   if (Parser.parseEOL())
1595:     return true;
1596: 
1597:   if (IsIntegerValue)
1598:     getTargetStreamer().emitAttribute(Tag, IntegerValue);
1599:   else if (Tag != CSKYAttrs::CSKY_ARCH_NAME && Tag != CSKYAttrs::CSKY_CPU_NAME)
1600:     getTargetStreamer().emitTextAttribute(Tag, StringValue);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 1601-1620

```cpp
1601:   else {
1602:     CSKY::ArchKind ID = (Tag == CSKYAttrs::CSKY_ARCH_NAME)
1603:                             ? CSKY::parseArch(StringValue)
1604:                             : CSKY::parseCPUArch(StringValue);
1605:     if (ID == CSKY::ArchKind::INVALID)
1606:       return Error(ValueExprLoc, (Tag == CSKYAttrs::CSKY_ARCH_NAME)
1607:                                      ? "unknown arch name"
1608:                                      : "unknown cpu name");
1609: 
1610:     getTargetStreamer().emitTextAttribute(Tag, StringValue);
1611:   }
1612: 
1613:   return false;
1614: }
1615: 
1616: unsigned CSKYAsmParser::validateTargetOperandClass(MCParsedAsmOperand &AsmOp,
1617:                                                    unsigned Kind) {
1618:   CSKYOperand &Op = static_cast<CSKYOperand &>(AsmOp);
1619: 
1620:   if (!Op.isReg())
```

- EN: Function bodies or method definitions such as validateTargetOperandClass contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: validateTargetOperandClass 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 1621-1640

```cpp
1621:     return Match_InvalidOperand;
1622: 
1623:   MCRegister Reg = Op.getReg();
1624: 
1625:   if (CSKYMCRegisterClasses[CSKY::FPR32RegClassID].contains(Reg)) {
1626:     // As the parser couldn't differentiate an FPR64 from an FPR32, coerce the
1627:     // register from FPR32 to FPR64 if necessary.
1628:     if (Kind == MCK_FPR64 || Kind == MCK_sFPR64) {
1629:       Op.Reg.RegNum = convertFPR32ToFPR64(Reg);
1630:       if (Kind == MCK_sFPR64 &&
1631:           (Op.Reg.RegNum < CSKY::F0_64 || Op.Reg.RegNum > CSKY::F15_64))
1632:         return Match_InvalidRegOutOfRange;
1633:       if (Kind == MCK_FPR64 &&
1634:           (Op.Reg.RegNum < CSKY::F0_64 || Op.Reg.RegNum > CSKY::F31_64))
1635:         return Match_InvalidRegOutOfRange;
1636:       return Match_Success;
1637:     }
1638:   }
1639: 
1640:   if (CSKYMCRegisterClasses[CSKY::GPRRegClassID].contains(Reg)) {
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 1641-1660

```cpp
1641:     if (Kind == MCK_GPRPair) {
1642:       Op.Reg.RegNum = MRI->getEncodingValue(Reg) + CSKY::R0_R1;
1643:       return Match_Success;
1644:     }
1645:   }
1646: 
1647:   return Match_InvalidOperand;
1648: }
1649: 
1650: void CSKYAsmParser::emitToStreamer(MCStreamer &S, const MCInst &Inst) {
1651:   MCInst CInst;
1652:   bool Res = false;
1653:   if (EnableCompressedInst)
1654:     Res = compressInst(CInst, Inst, getSTI());
1655:   if (Res)
1656:     ++CSKYNumInstrsCompressed;
1657:   S.emitInstruction((Res ? CInst : Inst), getSTI());
1658: }
1659: 
1660: extern "C" LLVM_EXTERNAL_VISIBILITY void LLVMInitializeCSKYAsmParser() {
```

- EN: Function bodies or method definitions such as emitToStreamer contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: emitToStreamer 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

### Lines 1661-1662

```cpp
1661:   RegisterMCAsmParser<CSKYAsmParser> X(getTheCSKYTarget());
1662: }
```

- EN: In file-level terms, this portion contributes to parsing assembly syntax into structured operands and instructions.
- 中文: 从文件角色看，这一部分负责把汇编语法解析为结构化操作数与指令。

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

- Direct includes / 直接包含: `MCTargetDesc/CSKYInstPrinter.h`, `MCTargetDesc/CSKYMCAsmInfo.h`, `MCTargetDesc/CSKYMCTargetDesc.h`, `MCTargetDesc/CSKYTargetStreamer.h`, `TargetInfo/CSKYTargetInfo.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/Statistic.h`, `llvm/ADT/StringExtras.h`
- LLVM subsystems / LLVM 子系统: LLVM MC
- Generated or companion files / 生成或配套文件: `CSKYGenCompressInstEmitter.inc`, `CSKYGenAsmMatcher.inc`
