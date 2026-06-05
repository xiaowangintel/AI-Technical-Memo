# HexagonAsmParser.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/AsmParser/HexagonAsmParser.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Parses Hexagon assembly text into MC instructions and directives.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。 重点涉及汇编/MC 层处理。 重点涉及指令语义与选择。

## Line-by-Line Analysis / 逐行分析

### Lines 1-200 / 第 1-200 行

```cpp
     1: //===-- HexagonAsmParser.cpp - Parse Hexagon asm to MCInst instructions----===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: 
     9: #include "MCTargetDesc/HexagonMCAsmInfo.h"
    10: #include "MCTargetDesc/HexagonMCChecker.h"
    11: #include "MCTargetDesc/HexagonMCELFStreamer.h"
    12: #include "MCTargetDesc/HexagonMCExpr.h"
    13: #include "MCTargetDesc/HexagonMCInstrInfo.h"
    14: #include "MCTargetDesc/HexagonMCTargetDesc.h"
    15: #include "MCTargetDesc/HexagonShuffler.h"
    16: #include "MCTargetDesc/HexagonTargetStreamer.h"
    17: #include "TargetInfo/HexagonTargetInfo.h"
    18: #include "llvm/ADT/STLExtras.h"
    19: #include "llvm/ADT/SmallVector.h"
    20: #include "llvm/ADT/StringExtras.h"
    21: #include "llvm/ADT/StringRef.h"
    22: #include "llvm/ADT/Twine.h"
    23: #include "llvm/BinaryFormat/ELF.h"
    24: #include "llvm/MC/MCAssembler.h"
    25: #include "llvm/MC/MCContext.h"
    26: #include "llvm/MC/MCDirectives.h"
    27: #include "llvm/MC/MCELFStreamer.h"
    28: #include "llvm/MC/MCExpr.h"
    29: #include "llvm/MC/MCInst.h"
    30: #include "llvm/MC/MCParser/AsmLexer.h"
    31: #include "llvm/MC/MCParser/MCAsmParser.h"
    32: #include "llvm/MC/MCParser/MCAsmParserExtension.h"
    33: #include "llvm/MC/MCParser/MCParsedAsmOperand.h"
    34: #include "llvm/MC/MCParser/MCTargetAsmParser.h"
    35: #include "llvm/MC/MCRegisterInfo.h"
    36: #include "llvm/MC/MCSectionELF.h"
    37: #include "llvm/MC/MCStreamer.h"
    38: #include "llvm/MC/MCSubtargetInfo.h"
    39: #include "llvm/MC/MCSymbol.h"
    40: #include "llvm/MC/MCValue.h"
    41: #include "llvm/MC/TargetRegistry.h"
    42: #include "llvm/Support/Casting.h"
    43: #include "llvm/Support/CommandLine.h"
    44: #include "llvm/Support/Compiler.h"
    45: #include "llvm/Support/Debug.h"
    46: #include "llvm/Support/ErrorHandling.h"
    47: #include "llvm/Support/Format.h"
    48: #include "llvm/Support/HexagonAttributes.h"
    49: #include "llvm/Support/MathExtras.h"
    50: #include "llvm/Support/SMLoc.h"
    51: #include "llvm/Support/SourceMgr.h"
    52: #include "llvm/Support/raw_ostream.h"
    53: #include <cassert>
    54: #include <cctype>
    55: #include <cstddef>
    56: #include <cstdint>
    57: #include <memory>
    58: #include <string>
    59: #include <utility>
    60: 
    61: #define DEBUG_TYPE "mcasmparser"
    62: 
    63: using namespace llvm;
    64: 
    65: static cl::opt<bool> WarnMissingParenthesis(
    66:     "mwarn-missing-parenthesis",
    67:     cl::desc("Warn for missing parenthesis around predicate registers"),
    68:     cl::init(true));
    69: static cl::opt<bool> ErrorMissingParenthesis(
    70:     "merror-missing-parenthesis",
    71:     cl::desc("Error for missing parenthesis around predicate registers"),
    72:     cl::init(false));
    73: static cl::opt<bool> WarnSignedMismatch(
    74:     "mwarn-sign-mismatch",
    75:     cl::desc("Warn for mismatching a signed and unsigned value"),
    76:     cl::init(false));
    77: static cl::opt<bool> WarnNoncontigiousRegister(
    78:     "mwarn-noncontigious-register",
    79:     cl::desc("Warn for register names that aren't contigious"), cl::init(true));
    80: static cl::opt<bool> ErrorNoncontigiousRegister(
    81:     "merror-noncontigious-register",
    82:     cl::desc("Error for register names that aren't contigious"),
    83:     cl::init(false));
    84: static cl::opt<bool> AddBuildAttributes("hexagon-add-build-attributes");
    85: namespace {
    86: 
    87: struct HexagonOperand;
    88: 
    89: class HexagonAsmParser : public MCTargetAsmParser {
    90: 
    91:   HexagonTargetStreamer &getTargetStreamer() {
    92:     MCTargetStreamer &TS = *Parser.getStreamer().getTargetStreamer();
    93:     return static_cast<HexagonTargetStreamer &>(TS);
    94:   }
    95: 
    96:   MCAsmParser &Parser;
    97:   MCInst MCB;
    98:   bool InBrackets;
    99: 
   100:   MCAsmParser &getParser() const { return Parser; }
   101:   MCAssembler *getAssembler() const {
   102:     MCAssembler *Assembler = nullptr;
   103:     // FIXME: need better way to detect AsmStreamer (upstream removed getKind())
   104:     if (!Parser.getStreamer().hasRawTextSupport()) {
   105:       MCELFStreamer *MES = static_cast<MCELFStreamer *>(&Parser.getStreamer());
   106:       Assembler = &MES->getAssembler();
   107:     }
   108:     return Assembler;
   109:   }
   110: 
   111:   AsmLexer &getLexer() const { return Parser.getLexer(); }
   112: 
   113:   bool equalIsAsmAssignment() override { return false; }
   114:   bool isLabel(AsmToken &Token) override;
   115:   bool tokenIsStartOfStatement(AsmToken::TokenKind Token) override;
   116: 
   117:   void Warning(SMLoc L, const Twine &Msg) { Parser.Warning(L, Msg); }
   118:   bool Error(SMLoc L, const Twine &Msg) { return Parser.Error(L, Msg); }
   119:   bool ParseDirectiveFalign(unsigned Size, SMLoc L);
   120: 
   121:   bool parseRegister(MCRegister &Reg, SMLoc &StartLoc, SMLoc &EndLoc) override;
   122:   ParseStatus tryParseRegister(MCRegister &Reg, SMLoc &StartLoc,
   123:                                SMLoc &EndLoc) override;
   124:   bool ParseDirectiveSubsection(SMLoc L);
   125:   bool ParseDirectiveComm(bool IsLocal, SMLoc L);
   126: 
   127:   bool parseDirectiveAttribute(SMLoc L);
   128: 
   129:   bool RegisterMatchesArch(MCRegister MatchNum) const;
   130: 
   131:   bool matchBundleOptions();
   132:   bool handleNoncontigiousRegister(bool Contigious, SMLoc &Loc);
   133:   bool finishBundle(SMLoc IDLoc, MCStreamer &Out);
   134:   void canonicalizeImmediates(MCInst &MCI);
   135:   bool matchOneInstruction(MCInst &MCB, SMLoc IDLoc,
   136:                            OperandVector &InstOperands, uint64_t &ErrorInfo,
   137:                            bool MatchingInlineAsm);
   138:   void eatToEndOfPacket();
   139:   bool matchAndEmitInstruction(SMLoc IDLoc, unsigned &Opcode,
   140:                                OperandVector &Operands, MCStreamer &Out,
   141:                                uint64_t &ErrorInfo,
   142:                                bool MatchingInlineAsm) override;
   143: 
   144:   unsigned validateTargetOperandClass(MCParsedAsmOperand &Op,
   145:                                       unsigned Kind) override;
   146:   bool OutOfRange(SMLoc IDLoc, long long Val, long long Max);
   147:   int processInstruction(MCInst &Inst, OperandVector const &Operands,
   148:                          SMLoc IDLoc);
   149: 
   150:   MCRegister matchRegister(StringRef Name);
   151: 
   152:   /// @name Auto-generated Match Functions
   153:   /// {
   154: 
   155: #define GET_ASSEMBLER_HEADER
   156: #include "HexagonGenAsmMatcher.inc"
   157: 
   158:   /// }
   159: 
   160: public:
   161:   HexagonAsmParser(const MCSubtargetInfo &_STI, MCAsmParser &_Parser,
   162:                    const MCInstrInfo &MII)
   163:       : MCTargetAsmParser(_STI, MII), Parser(_Parser), InBrackets(false) {
   164:     MCB.setOpcode(Hexagon::BUNDLE);
   165:     setAvailableFeatures(ComputeAvailableFeatures(getSTI().getFeatureBits()));
   166: 
   167:     Parser.addAliasForDirective(".half", ".2byte");
   168:     Parser.addAliasForDirective(".hword", ".2byte");
   169:     Parser.addAliasForDirective(".word", ".4byte");
   170: 
   171:     MCAsmParserExtension::Initialize(_Parser);
   172: 
   173:     if (AddBuildAttributes)
   174:       getTargetStreamer().emitTargetAttributes(*STI);
   175:   }
   176: 
   177:   bool splitIdentifier(OperandVector &Operands);
   178:   bool parseOperand(OperandVector &Operands);
   179:   bool parseInstruction(OperandVector &Operands);
   180:   bool implicitExpressionLocation(OperandVector &Operands);
   181:   bool parseExpressionOrOperand(OperandVector &Operands);
   182:   bool parseExpression(MCExpr const *&Expr);
   183: 
   184:   bool parseInstruction(ParseInstructionInfo &Info, StringRef Name,
   185:                         SMLoc NameLoc, OperandVector &Operands) override {
   186:     llvm_unreachable("Unimplemented");
   187:   }
   188: 
   189:   bool parseInstruction(ParseInstructionInfo &Info, StringRef Name, AsmToken ID,
   190:                         OperandVector &Operands) override;
   191: 
   192:   bool ParseDirective(AsmToken DirectiveID) override;
   193: };
   194: 
   195: /// HexagonOperand - Instances of this class represent a parsed Hexagon machine
   196: /// instruction.
   197: struct HexagonOperand : public MCParsedAsmOperand {
   198:   enum KindTy { Token, Immediate, Register } Kind;
   199:   MCContext &Context;
   200: 
```
- EN: It imports headers such as MCTargetDesc/HexagonMCAsmInfo.h, MCTargetDesc/HexagonMCChecker.h, MCTargetDesc/HexagonMCELFStreamer.h, MCTargetDesc/HexagonMCExpr.h, ... (52 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonOperand, HexagonAsmParser, KindTy, which carry the state or API of this component.
- CN: 这里引入了 MCTargetDesc/HexagonMCAsmInfo.h, MCTargetDesc/HexagonMCChecker.h, MCTargetDesc/HexagonMCELFStreamer.h, MCTargetDesc/HexagonMCExpr.h, ... (52 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonOperand, HexagonAsmParser, KindTy 等类型，用来承载该组件的状态或接口。

### Lines 201-400 / 第 201-400 行

```cpp
   201:   SMLoc StartLoc, EndLoc;
   202: 
   203:   struct TokTy {
   204:     const char *Data;
   205:     unsigned Length;
   206:   };
   207: 
   208:   struct RegTy {
   209:     MCRegister RegNum;
   210:   };
   211: 
   212:   struct ImmTy {
   213:     const MCExpr *Val;
   214:   };
   215: 
   216:   union {
   217:     struct TokTy Tok;
   218:     struct RegTy Reg;
   219:     struct ImmTy Imm;
   220:   };
   221: 
   222:   HexagonOperand(KindTy K, MCContext &Context) : Kind(K), Context(Context) {}
   223: 
   224: public:
   225:   HexagonOperand(const HexagonOperand &o)
   226:       : MCParsedAsmOperand(), Context(o.Context) {
   227:     Kind = o.Kind;
   228:     StartLoc = o.StartLoc;
   229:     EndLoc = o.EndLoc;
   230:     switch (Kind) {
   231:     case Register:
   232:       Reg = o.Reg;
   233:       break;
   234:     case Immediate:
   235:       Imm = o.Imm;
   236:       break;
   237:     case Token:
   238:       Tok = o.Tok;
   239:       break;
   240:     }
   241:   }
   242: 
   243:   /// getStartLoc - Get the location of the first token of this operand.
   244:   SMLoc getStartLoc() const override { return StartLoc; }
   245: 
   246:   /// getEndLoc - Get the location of the last token of this operand.
   247:   SMLoc getEndLoc() const override { return EndLoc; }
   248: 
   249:   MCRegister getReg() const override {
   250:     assert(Kind == Register && "Invalid access!");
   251:     return Reg.RegNum;
   252:   }
   253: 
   254:   const MCExpr *getImm() const {
   255:     assert(Kind == Immediate && "Invalid access!");
   256:     return Imm.Val;
   257:   }
   258: 
   259:   bool isToken() const override { return Kind == Token; }
   260:   bool isImm() const override { return Kind == Immediate; }
   261:   bool isMem() const override { llvm_unreachable("No isMem"); }
   262:   bool isReg() const override { return Kind == Register; }
   263: 
   264:   bool CheckImmRange(int immBits, int zeroBits, bool isSigned,
   265:                      bool isRelocatable, bool Extendable) const {
   266:     if (Kind == Immediate) {
   267:       const MCExpr *myMCExpr = &HexagonMCInstrInfo::getExpr(*getImm());
   268:       if (HexagonMCInstrInfo::mustExtend(*Imm.Val) && !Extendable)
   269:         return false;
   270:       int64_t Res;
   271:       if (myMCExpr->evaluateAsAbsolute(Res)) {
   272:         int bits = immBits + zeroBits;
   273:         // Field bit range is zerobits + bits
   274:         // zeroBits must be 0
   275:         if (Res & ((1 << zeroBits) - 1))
   276:           return false;
   277:         if (isSigned) {
   278:           if (Res < (1LL << (bits - 1)) && Res >= -(1LL << (bits - 1)))
   279:             return true;
   280:         } else {
   281:           if (bits == 64)
   282:             return true;
   283:           if (Res >= 0)
   284:             return ((uint64_t)Res < (uint64_t)(1ULL << bits));
   285:           else {
   286:             const int64_t high_bit_set = 1ULL << 63;
   287:             const uint64_t mask = (high_bit_set >> (63 - bits));
   288:             return (((uint64_t)Res & mask) == mask);
   289:           }
   290:         }
   291:       } else if (myMCExpr->getKind() == MCExpr::SymbolRef && isRelocatable)
   292:         return true;
   293:       else if (myMCExpr->getKind() == MCExpr::Binary ||
   294:                myMCExpr->getKind() == MCExpr::Unary)
   295:         return true;
   296:     }
   297:     return false;
   298:   }
   299: 
   300:   bool isa30_2Imm() const { return CheckImmRange(30, 2, true, true, true); }
   301:   bool isb30_2Imm() const { return CheckImmRange(30, 2, true, true, true); }
   302:   bool isb15_2Imm() const { return CheckImmRange(15, 2, true, true, false); }
   303:   bool isb13_2Imm() const { return CheckImmRange(13, 2, true, true, false); }
   304: 
   305:   bool ism32_0Imm() const { return true; }
   306: 
   307:   bool isf32Imm() const { return false; }
   308:   bool isf64Imm() const { return false; }
   309:   bool iss32_0Imm() const { return true; }
   310:   bool iss31_1Imm() const { return true; }
   311:   bool iss30_2Imm() const { return true; }
   312:   bool iss29_3Imm() const { return true; }
   313:   bool iss27_2Imm() const { return CheckImmRange(27, 2, true, true, false); }
   314:   bool iss10_0Imm() const { return CheckImmRange(10, 0, true, false, false); }
   315:   bool iss10_6Imm() const { return CheckImmRange(10, 6, true, false, false); }
   316:   bool iss9_0Imm() const { return CheckImmRange(9, 0, true, false, false); }
   317:   bool iss8_0Imm() const { return CheckImmRange(8, 0, true, false, false); }
   318:   bool iss8_0Imm64() const { return CheckImmRange(8, 0, true, true, false); }
   319:   bool iss7_0Imm() const { return CheckImmRange(7, 0, true, false, false); }
   320:   bool iss6_0Imm() const { return CheckImmRange(6, 0, true, false, false); }
   321:   bool iss6_3Imm() const { return CheckImmRange(6, 3, true, false, false); }
   322:   bool iss4_0Imm() const { return CheckImmRange(4, 0, true, false, false); }
   323:   bool iss4_1Imm() const { return CheckImmRange(4, 1, true, false, false); }
   324:   bool iss4_2Imm() const { return CheckImmRange(4, 2, true, false, false); }
   325:   bool iss4_3Imm() const { return CheckImmRange(4, 3, true, false, false); }
   326:   bool iss3_0Imm() const { return CheckImmRange(3, 0, true, false, false); }
   327: 
   328:   bool isu64_0Imm() const { return CheckImmRange(64, 0, false, true, true); }
   329:   bool isu32_0Imm() const { return true; }
   330:   bool isu31_1Imm() const { return true; }
   331:   bool isu30_2Imm() const { return true; }
   332:   bool isu29_3Imm() const { return true; }
   333:   bool isu26_6Imm() const { return CheckImmRange(26, 6, false, true, false); }
   334:   bool isu16_0Imm() const { return CheckImmRange(16, 0, false, true, false); }
   335:   bool isu16_1Imm() const { return CheckImmRange(16, 1, false, true, false); }
   336:   bool isu16_2Imm() const { return CheckImmRange(16, 2, false, true, false); }
   337:   bool isu16_3Imm() const { return CheckImmRange(16, 3, false, true, false); }
   338:   bool isu11_3Imm() const { return CheckImmRange(11, 3, false, false, false); }
   339:   bool isu10_0Imm() const { return CheckImmRange(10, 0, false, false, false); }
   340:   bool isu9_0Imm() const { return CheckImmRange(9, 0, false, false, false); }
   341:   bool isu8_0Imm() const { return CheckImmRange(8, 0, false, false, false); }
   342:   bool isu7_0Imm() const { return CheckImmRange(7, 0, false, false, false); }
   343:   bool isu6_0Imm() const { return CheckImmRange(6, 0, false, false, false); }
   344:   bool isu6_1Imm() const { return CheckImmRange(6, 1, false, false, false); }
   345:   bool isu6_2Imm() const { return CheckImmRange(6, 2, false, false, false); }
   346:   bool isu6_3Imm() const { return CheckImmRange(6, 3, false, false, false); }
   347:   bool isu5_0Imm() const { return CheckImmRange(5, 0, false, false, false); }
   348:   bool isu5_2Imm() const { return CheckImmRange(5, 2, false, false, false); }
   349:   bool isu5_3Imm() const { return CheckImmRange(5, 3, false, false, false); }
   350:   bool isu4_0Imm() const { return CheckImmRange(4, 0, false, false, false); }
   351:   bool isu4_2Imm() const { return CheckImmRange(4, 2, false, false, false); }
   352:   bool isu3_0Imm() const { return CheckImmRange(3, 0, false, false, false); }
   353:   bool isu3_1Imm() const { return CheckImmRange(3, 1, false, false, false); }
   354:   bool isu2_0Imm() const { return CheckImmRange(2, 0, false, false, false); }
   355:   bool isu1_0Imm() const { return CheckImmRange(1, 0, false, false, false); }
   356: 
   357:   bool isn1Const() const {
   358:     if (!isImm())
   359:       return false;
   360:     int64_t Value;
   361:     if (!getImm()->evaluateAsAbsolute(Value))
   362:       return false;
   363:     return Value == -1;
   364:   }
   365:   bool issgp10Const() const {
   366:     if (!isReg())
   367:       return false;
   368:     return getReg() == Hexagon::SGP1_0;
   369:   }
   370:   bool iss11_0Imm() const {
   371:     return CheckImmRange(11 + 26, 0, true, true, true);
   372:   }
   373:   bool iss11_1Imm() const {
   374:     return CheckImmRange(11 + 26, 1, true, true, true);
   375:   }
   376:   bool iss11_2Imm() const {
   377:     return CheckImmRange(11 + 26, 2, true, true, true);
   378:   }
   379:   bool iss11_3Imm() const {
   380:     return CheckImmRange(11 + 26, 3, true, true, true);
   381:   }
   382:   bool isu32_0MustExt() const { return isImm(); }
   383: 
   384:   void addRegOperands(MCInst &Inst, unsigned N) const {
   385:     assert(N == 1 && "Invalid number of operands!");
   386:     Inst.addOperand(MCOperand::createReg(getReg()));
   387:   }
   388: 
   389:   void addImmOperands(MCInst &Inst, unsigned N) const {
   390:     assert(N == 1 && "Invalid number of operands!");
   391:     Inst.addOperand(MCOperand::createExpr(getImm()));
   392:   }
   393: 
   394:   void addSignedImmOperands(MCInst &Inst, unsigned N) const {
   395:     assert(N == 1 && "Invalid number of operands!");
   396:     HexagonMCExpr *Expr =
   397:         const_cast<HexagonMCExpr *>(cast<HexagonMCExpr>(getImm()));
   398:     int64_t Value;
   399:     if (!Expr->evaluateAsAbsolute(Value)) {
   400:       Inst.addOperand(MCOperand::createExpr(Expr));
```
- EN: It declares types such as TokTy, RegTy, ImmTy, which carry the state or API of this component. It declares or implements routines such as HexagonOperand, getStartLoc, getEndLoc, getReg, ... (78 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明了 TokTy, RegTy, ImmTy 等类型，用来承载该组件的状态或接口。 这里声明或实现了 HexagonOperand, getStartLoc, getEndLoc, getReg, ... (78 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 401-600 / 第 401-600 行

```cpp
   401:       return;
   402:     }
   403:     int64_t Extended = SignExtend64(Value, 32);
   404:     HexagonMCExpr *NewExpr = HexagonMCExpr::create(
   405:         MCConstantExpr::create(Extended, Context), Context);
   406:     if ((Extended < 0) != (Value < 0))
   407:       NewExpr->setSignMismatch();
   408:     NewExpr->setMustExtend(Expr->mustExtend());
   409:     NewExpr->setMustNotExtend(Expr->mustNotExtend());
   410:     Inst.addOperand(MCOperand::createExpr(NewExpr));
   411:   }
   412: 
   413:   void addn1ConstOperands(MCInst &Inst, unsigned N) const {
   414:     addImmOperands(Inst, N);
   415:   }
   416:   void addsgp10ConstOperands(MCInst &Inst, unsigned N) const {
   417:     addRegOperands(Inst, N);
   418:   }
   419: 
   420:   StringRef getToken() const {
   421:     assert(Kind == Token && "Invalid access!");
   422:     return StringRef(Tok.Data, Tok.Length);
   423:   }
   424: 
   425:   void print(raw_ostream &OS, const MCAsmInfo &MAI) const override;
   426: 
   427:   static std::unique_ptr<HexagonOperand> CreateToken(MCContext &Context,
   428:                                                      StringRef Str, SMLoc S) {
   429:     HexagonOperand *Op = new HexagonOperand(Token, Context);
   430:     Op->Tok.Data = Str.data();
   431:     Op->Tok.Length = Str.size();
   432:     Op->StartLoc = S;
   433:     Op->EndLoc = S;
   434:     return std::unique_ptr<HexagonOperand>(Op);
   435:   }
   436: 
   437:   static std::unique_ptr<HexagonOperand>
   438:   CreateReg(MCContext &Context, MCRegister Reg, SMLoc S, SMLoc E) {
   439:     HexagonOperand *Op = new HexagonOperand(Register, Context);
   440:     Op->Reg.RegNum = Reg;
   441:     Op->StartLoc = S;
   442:     Op->EndLoc = E;
   443:     return std::unique_ptr<HexagonOperand>(Op);
   444:   }
   445: 
   446:   static std::unique_ptr<HexagonOperand>
   447:   CreateImm(MCContext &Context, const MCExpr *Val, SMLoc S, SMLoc E) {
   448:     HexagonOperand *Op = new HexagonOperand(Immediate, Context);
   449:     Op->Imm.Val = Val;
   450:     Op->StartLoc = S;
   451:     Op->EndLoc = E;
   452:     return std::unique_ptr<HexagonOperand>(Op);
   453:   }
   454: };
   455: 
   456: } // end anonymous namespace
   457: 
   458: void HexagonOperand::print(raw_ostream &OS, const MCAsmInfo &MAI) const {
   459:   switch (Kind) {
   460:   case Immediate:
   461:     MAI.printExpr(OS, *getImm());
   462:     break;
   463:   case Register:
   464:     OS << "<register R";
   465:     OS << getReg().id() << ">";
   466:     break;
   467:   case Token:
   468:     OS << "'" << getToken() << "'";
   469:     break;
   470:   }
   471: }
   472: 
   473: bool HexagonAsmParser::finishBundle(SMLoc IDLoc, MCStreamer &Out) {
   474:   LLVM_DEBUG(dbgs() << "Bundle:");
   475:   LLVM_DEBUG(MCB.dump_pretty(dbgs()));
   476:   LLVM_DEBUG(dbgs() << "--\n");
   477: 
   478:   MCB.setLoc(IDLoc);
   479: 
   480:   // Check the bundle for errors.
   481:   const MCRegisterInfo *RI = getContext().getRegisterInfo();
   482:   MCSubtargetInfo const &STI = getSTI();
   483: 
   484:   MCInst OrigBundle = MCB;
   485:   HexagonMCChecker Check(getContext(), MII, STI, MCB, *RI, true);
   486: 
   487:   bool CheckOk = HexagonMCInstrInfo::canonicalizePacket(
   488:       MII, STI, getContext(), MCB, &Check, true);
   489: 
   490:   if (CheckOk) {
   491:     if (HexagonMCInstrInfo::bundleSize(MCB) == 0) {
   492:       assert(!HexagonMCInstrInfo::isInnerLoop(MCB));
   493:       assert(!HexagonMCInstrInfo::isOuterLoop(MCB));
   494:       // Empty packets are valid yet aren't emitted
   495:       return false;
   496:     }
   497: 
   498:     assert(HexagonMCInstrInfo::isBundle(MCB));
   499: 
   500:     Out.emitInstruction(MCB, STI);
   501:   } else
   502:     return true; // Error
   503: 
   504:   return false; // No error
   505: }
   506: 
   507: bool HexagonAsmParser::matchBundleOptions() {
   508:   MCAsmParser &Parser = getParser();
   509:   while (true) {
   510:     if (!Parser.getTok().is(AsmToken::Colon))
   511:       return false;
   512:     Lex();
   513:     char const *MemNoShuffMsg =
   514:         "invalid instruction packet: mem_noshuf specifier not "
   515:         "supported with this architecture";
   516:     StringRef Option = Parser.getTok().getString();
   517:     auto IDLoc = Parser.getTok().getLoc();
   518:     if (Option.compare_insensitive("endloop01") == 0) {
   519:       HexagonMCInstrInfo::setInnerLoop(MCB);
   520:       HexagonMCInstrInfo::setOuterLoop(MCB);
   521:     } else if (Option.compare_insensitive("endloop0") == 0) {
   522:       HexagonMCInstrInfo::setInnerLoop(MCB);
   523:     } else if (Option.compare_insensitive("endloop1") == 0) {
   524:       HexagonMCInstrInfo::setOuterLoop(MCB);
   525:     } else if (Option.compare_insensitive("mem_noshuf") == 0) {
   526:       if (getSTI().hasFeature(Hexagon::FeatureMemNoShuf))
   527:         HexagonMCInstrInfo::setMemReorderDisabled(MCB);
   528:       else
   529:         return getParser().Error(IDLoc, MemNoShuffMsg);
   530:     } else if (Option.compare_insensitive("mem_no_order") == 0) {
   531:       // Nothing.
   532:     } else
   533:       return getParser().Error(IDLoc, llvm::Twine("'") + Option +
   534:                                           "' is not a valid bundle option");
   535:     Lex();
   536:   }
   537: }
   538: 
   539: // For instruction aliases, immediates are generated rather than
   540: // MCConstantExpr.  Convert them for uniform MCExpr.
   541: // Also check for signed/unsigned mismatches and warn
   542: void HexagonAsmParser::canonicalizeImmediates(MCInst &MCI) {
   543:   MCInst NewInst;
   544:   NewInst.setOpcode(MCI.getOpcode());
   545:   for (MCOperand &I : MCI)
   546:     if (I.isImm()) {
   547:       int64_t Value(I.getImm());
   548:       NewInst.addOperand(MCOperand::createExpr(HexagonMCExpr::create(
   549:           MCConstantExpr::create(Value, getContext()), getContext())));
   550:     } else {
   551:       if (I.isExpr() && cast<HexagonMCExpr>(I.getExpr())->signMismatch() &&
   552:           WarnSignedMismatch)
   553:         Warning(MCI.getLoc(), "Signed/Unsigned mismatch");
   554:       NewInst.addOperand(I);
   555:     }
   556:   MCI = NewInst;
   557: }
   558: 
   559: bool HexagonAsmParser::matchOneInstruction(MCInst &MCI, SMLoc IDLoc,
   560:                                            OperandVector &InstOperands,
   561:                                            uint64_t &ErrorInfo,
   562:                                            bool MatchingInlineAsm) {
   563:   // Perform matching with tablegen asmmatcher generated function
   564:   int result =
   565:       MatchInstructionImpl(InstOperands, MCI, ErrorInfo, MatchingInlineAsm);
   566:   if (result == Match_Success) {
   567:     MCI.setLoc(IDLoc);
   568:     canonicalizeImmediates(MCI);
   569:     result = processInstruction(MCI, InstOperands, IDLoc);
   570: 
   571:     LLVM_DEBUG(dbgs() << "Insn:");
   572:     LLVM_DEBUG(MCI.dump_pretty(dbgs()));
   573:     LLVM_DEBUG(dbgs() << "\n\n");
   574: 
   575:     MCI.setLoc(IDLoc);
   576:   }
   577: 
   578:   // Create instruction operand for bundle instruction
   579:   //   Break this into a separate function Code here is less readable
   580:   //   Think about how to get an instruction error to report correctly.
   581:   //   SMLoc will return the "{"
   582:   switch (result) {
   583:   default:
   584:     break;
   585:   case Match_Success:
   586:     return false;
   587:   case Match_MissingFeature:
   588:     return Error(IDLoc, "invalid instruction");
   589:   case Match_MnemonicFail:
   590:     return Error(IDLoc, "unrecognized instruction");
   591:   case Match_InvalidOperand:
   592:     [[fallthrough]];
   593:   case Match_InvalidTiedOperand:
   594:     SMLoc ErrorLoc = IDLoc;
   595:     if (ErrorInfo != ~0U) {
   596:       if (ErrorInfo >= InstOperands.size())
   597:         return Error(IDLoc, "too few operands for instruction");
   598: 
   599:       ErrorLoc = (static_cast<HexagonOperand *>(InstOperands[ErrorInfo].get()))
   600:                      ->getStartLoc();
```
- EN: It opens namespaces (void) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as SignExtend64, HexagonMCExpr::create, setMustExtend, setMustNotExtend, ... (45 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里打开了命名空间（void），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 SignExtend64, HexagonMCExpr::create, setMustExtend, setMustNotExtend, ... (45 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 601-800 / 第 601-800 行

```cpp
   601:       if (ErrorLoc == SMLoc())
   602:         ErrorLoc = IDLoc;
   603:     }
   604:     return Error(ErrorLoc, "invalid operand for instruction");
   605:   }
   606:   llvm_unreachable("Implement any new match types added!");
   607: }
   608: 
   609: void HexagonAsmParser::eatToEndOfPacket() {
   610:   assert(InBrackets);
   611:   AsmLexer &Lexer = getLexer();
   612:   while (!Lexer.is(AsmToken::RCurly))
   613:     Lexer.Lex();
   614:   Lexer.Lex();
   615:   InBrackets = false;
   616: }
   617: 
   618: bool HexagonAsmParser::matchAndEmitInstruction(SMLoc IDLoc, unsigned &Opcode,
   619:                                                OperandVector &Operands,
   620:                                                MCStreamer &Out,
   621:                                                uint64_t &ErrorInfo,
   622:                                                bool MatchingInlineAsm) {
   623:   if (!InBrackets) {
   624:     MCB.clear();
   625:     MCB.addOperand(MCOperand::createImm(0));
   626:   }
   627:   HexagonOperand &FirstOperand = static_cast<HexagonOperand &>(*Operands[0]);
   628:   if (FirstOperand.isToken() && FirstOperand.getToken() == "{") {
   629:     assert(Operands.size() == 1 && "Brackets should be by themselves");
   630:     if (InBrackets) {
   631:       getParser().Error(IDLoc, "Already in a packet");
   632:       InBrackets = false;
   633:       return true;
   634:     }
   635:     InBrackets = true;
   636:     return false;
   637:   }
   638:   if (FirstOperand.isToken() && FirstOperand.getToken() == "}") {
   639:     assert(Operands.size() == 1 && "Brackets should be by themselves");
   640:     if (!InBrackets) {
   641:       getParser().Error(IDLoc, "Not in a packet");
   642:       return true;
   643:     }
   644:     InBrackets = false;
   645:     if (matchBundleOptions())
   646:       return true;
   647:     return finishBundle(IDLoc, Out);
   648:   }
   649:   MCInst *SubInst = getParser().getContext().createMCInst();
   650:   if (matchOneInstruction(*SubInst, IDLoc, Operands, ErrorInfo,
   651:                           MatchingInlineAsm)) {
   652:     if (InBrackets)
   653:       eatToEndOfPacket();
   654:     return true;
   655:   }
   656:   HexagonMCInstrInfo::extendIfNeeded(
   657:       getParser().getContext(), MII, MCB, *SubInst);
   658:   MCB.addOperand(MCOperand::createInst(SubInst));
   659:   if (!InBrackets)
   660:     return finishBundle(IDLoc, Out);
   661:   return false;
   662: }
   663: /// parseDirectiveAttribute
   664: ///  ::= .attribute int, int
   665: ///  ::= .attribute Tag_name, int
   666: bool HexagonAsmParser::parseDirectiveAttribute(SMLoc L) {
   667:   MCAsmParser &Parser = getParser();
   668:   int64_t Tag;
   669:   SMLoc TagLoc = Parser.getTok().getLoc();
   670:   if (Parser.getTok().is(AsmToken::Identifier)) {
   671:     StringRef Name = Parser.getTok().getIdentifier();
   672:     std::optional<unsigned> Ret = ELFAttrs::attrTypeFromString(
   673:         Name, HexagonAttrs::getHexagonAttributeTags());
   674:     if (!Ret)
   675:       return Error(TagLoc, "attribute name not recognized: " + Name);
   676:     Tag = *Ret;
   677:     Parser.Lex();
   678:   } else {
   679:     const MCExpr *AttrExpr;
   680: 
   681:     TagLoc = Parser.getTok().getLoc();
   682:     if (Parser.parseExpression(AttrExpr))
   683:       return true;
   684: 
   685:     const MCConstantExpr *CE = dyn_cast<MCConstantExpr>(AttrExpr);
   686:     if (check(!CE, TagLoc, "expected numeric constant"))
   687:       return true;
   688: 
   689:     Tag = CE->getValue();
   690:   }
   691: 
   692:   if (Parser.parseComma())
   693:     return true;
   694: 
   695:   // We currently only have integer values.
   696:   int64_t IntegerValue = 0;
   697:   SMLoc ValueExprLoc = Parser.getTok().getLoc();
   698:   const MCExpr *ValueExpr;
   699:   if (Parser.parseExpression(ValueExpr))
   700:     return true;
   701: 
   702:   const MCConstantExpr *CE = dyn_cast<MCConstantExpr>(ValueExpr);
   703:   if (!CE)
   704:     return Error(ValueExprLoc, "expected numeric constant");
   705:   IntegerValue = CE->getValue();
   706: 
   707:   if (Parser.parseEOL())
   708:     return true;
   709: 
   710:   getTargetStreamer().emitAttribute(Tag, IntegerValue);
   711:   return false;
   712: }
   713: 
   714: /// ParseDirective parses the Hexagon specific directives
   715: bool HexagonAsmParser::ParseDirective(AsmToken DirectiveID) {
   716:   StringRef IDVal = DirectiveID.getIdentifier();
   717:   if (IDVal.lower() == ".falign")
   718:     return ParseDirectiveFalign(256, DirectiveID.getLoc());
   719:   if ((IDVal.lower() == ".lcomm") || (IDVal.lower() == ".lcommon"))
   720:     return ParseDirectiveComm(true, DirectiveID.getLoc());
   721:   if ((IDVal.lower() == ".comm") || (IDVal.lower() == ".common"))
   722:     return ParseDirectiveComm(false, DirectiveID.getLoc());
   723:   if (IDVal.lower() == ".subsection")
   724:     return ParseDirectiveSubsection(DirectiveID.getLoc());
   725:   if (IDVal == ".attribute")
   726:     return parseDirectiveAttribute(DirectiveID.getLoc());
   727: 
   728:   return true;
   729: }
   730: bool HexagonAsmParser::ParseDirectiveSubsection(SMLoc L) {
   731:   const MCExpr *Subsection = nullptr;
   732:   int64_t Res;
   733: 
   734:   assert((getLexer().isNot(AsmToken::EndOfStatement)) &&
   735:          "Invalid subsection directive");
   736:   getParser().parseExpression(Subsection);
   737: 
   738:   if (!Subsection->evaluateAsAbsolute(Res))
   739:     return Error(L, "Cannot evaluate subsection number");
   740: 
   741:   if (getLexer().isNot(AsmToken::EndOfStatement))
   742:     return TokError("unexpected token in directive");
   743: 
   744:   // 0-8192 is the hard-coded range in MCObjectStreamper.cpp, this keeps the
   745:   // negative subsections together and in the same order but at the opposite
   746:   // end of the section.  Only legacy hexagon-gcc created assembly code
   747:   // used negative subsections.
   748:   if ((Res < 0) && (Res > -8193))
   749:     Res += 8192;
   750:   getStreamer().switchSection(getStreamer().getCurrentSectionOnly(), Res);
   751:   return false;
   752: }
   753: 
   754: ///  ::= .falign [expression]
   755: bool HexagonAsmParser::ParseDirectiveFalign(unsigned Size, SMLoc L) {
   756: 
   757:   int64_t MaxBytesToFill = 15;
   758: 
   759:   // if there is an argument
   760:   if (getLexer().isNot(AsmToken::EndOfStatement)) {
   761:     const MCExpr *Value;
   762:     SMLoc ExprLoc = L;
   763: 
   764:     // Make sure we have a number (false is returned if expression is a number)
   765:     if (!getParser().parseExpression(Value)) {
   766:       // Make sure this is a number that is in range
   767:       auto *MCE = cast<MCConstantExpr>(Value);
   768:       uint64_t IntValue = MCE->getValue();
   769:       if (!isUIntN(Size, IntValue) && !isIntN(Size, IntValue))
   770:         return Error(ExprLoc, "literal value out of range (256) for falign");
   771:       MaxBytesToFill = IntValue;
   772:       Lex();
   773:     } else {
   774:       return Error(ExprLoc, "not a valid expression for falign directive");
   775:     }
   776:   }
   777: 
   778:   getTargetStreamer().emitFAlign(16, MaxBytesToFill);
   779:   Lex();
   780: 
   781:   return false;
   782: }
   783: 
   784: // This is largely a copy of AsmParser's ParseDirectiveComm extended to
   785: // accept a 3rd argument, AccessAlignment which indicates the smallest
   786: // memory access made to the symbol, expressed in bytes.  If no
   787: // AccessAlignment is specified it defaults to the Alignment Value.
   788: // Hexagon's .lcomm:
   789: //   .lcomm Symbol, Length, Alignment, AccessAlignment
   790: bool HexagonAsmParser::ParseDirectiveComm(bool IsLocal, SMLoc Loc) {
   791:   // FIXME: need better way to detect if AsmStreamer (upstream removed
   792:   // getKind())
   793:   if (getStreamer().hasRawTextSupport())
   794:     return true; // Only object file output requires special treatment.
   795: 
   796:   StringRef Name;
   797:   if (getParser().parseIdentifier(Name))
   798:     return TokError("expected identifier in directive");
   799:   // Handle the identifier as the key symbol.
   800:   MCSymbol *Sym = getContext().getOrCreateSymbol(Name);
```
- EN: It declares or implements routines such as Error, llvm_unreachable, HexagonAsmParser::eatToEndOfPacket, assert, ... (27 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonAsmParser, HexagonOperand, HexagonMCInstrInfo, HexagonAttrs, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 Error, llvm_unreachable, HexagonAsmParser::eatToEndOfPacket, assert, ... (27 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonAsmParser, HexagonOperand, HexagonMCInstrInfo, HexagonAttrs，说明了它与同级后端组件的连接关系。

### Lines 801-1000 / 第 801-1000 行

```cpp
   801: 
   802:   if (getLexer().isNot(AsmToken::Comma))
   803:     return TokError("unexpected token in directive");
   804:   Lex();
   805: 
   806:   int64_t Size;
   807:   SMLoc SizeLoc = getLexer().getLoc();
   808:   if (getParser().parseAbsoluteExpression(Size))
   809:     return true;
   810: 
   811:   int64_t ByteAlignment = 1;
   812:   SMLoc ByteAlignmentLoc;
   813:   if (getLexer().is(AsmToken::Comma)) {
   814:     Lex();
   815:     ByteAlignmentLoc = getLexer().getLoc();
   816:     if (getParser().parseAbsoluteExpression(ByteAlignment))
   817:       return true;
   818:     if (!isPowerOf2_64(ByteAlignment))
   819:       return Error(ByteAlignmentLoc, "alignment must be a power of 2");
   820:   }
   821: 
   822:   int64_t AccessAlignment = 0;
   823:   if (getLexer().is(AsmToken::Comma)) {
   824:     // The optional access argument specifies the size of the smallest memory
   825:     //   access to be made to the symbol, expressed in bytes.
   826:     SMLoc AccessAlignmentLoc;
   827:     Lex();
   828:     AccessAlignmentLoc = getLexer().getLoc();
   829:     if (getParser().parseAbsoluteExpression(AccessAlignment))
   830:       return true;
   831: 
   832:     if (!isPowerOf2_64(AccessAlignment))
   833:       return Error(AccessAlignmentLoc, "access alignment must be a power of 2");
   834:   }
   835: 
   836:   if (getLexer().isNot(AsmToken::EndOfStatement))
   837:     return TokError("unexpected token in '.comm' or '.lcomm' directive");
   838: 
   839:   Lex();
   840: 
   841:   // NOTE: a size of zero for a .comm should create a undefined symbol
   842:   // but a size of .lcomm creates a bss symbol of size zero.
   843:   if (Size < 0)
   844:     return Error(SizeLoc, "invalid '.comm' or '.lcomm' directive size, can't "
   845:                           "be less than zero");
   846: 
   847:   // NOTE: The alignment in the directive is a power of 2 value, the assembler
   848:   // may internally end up wanting an alignment in bytes.
   849:   // FIXME: Diagnose overflow.
   850:   if (ByteAlignment < 0)
   851:     return Error(ByteAlignmentLoc, "invalid '.comm' or '.lcomm' directive "
   852:                                    "alignment, can't be less than zero");
   853: 
   854:   if (!Sym->isUndefined())
   855:     return Error(Loc, "invalid symbol redefinition");
   856: 
   857:   HexagonMCELFStreamer &HexagonELFStreamer =
   858:       static_cast<HexagonMCELFStreamer &>(getStreamer());
   859:   if (IsLocal) {
   860:     HexagonELFStreamer.HexagonMCEmitLocalCommonSymbol(
   861:         Sym, Size, Align(ByteAlignment), AccessAlignment);
   862:     return false;
   863:   }
   864: 
   865:   HexagonELFStreamer.HexagonMCEmitCommonSymbol(Sym, Size, Align(ByteAlignment),
   866:                                                AccessAlignment);
   867:   return false;
   868: }
   869: 
   870: // validate register against architecture
   871: bool HexagonAsmParser::RegisterMatchesArch(MCRegister MatchNum) const {
   872:   if (HexagonMCRegisterClasses[Hexagon::V62RegsRegClassID].contains(MatchNum))
   873:     if (!getSTI().hasFeature(Hexagon::ArchV62))
   874:       return false;
   875:   return true;
   876: }
   877: 
   878: // extern "C" LLVM_EXTERNAL_VISIBILITY void LLVMInitializeHexagonAsmLexer();
   879: 
   880: /// Force static initialization.
   881: extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
   882: LLVMInitializeHexagonAsmParser() {
   883:   RegisterMCAsmParser<HexagonAsmParser> X(getTheHexagonTarget());
   884: }
   885: 
   886: #define GET_MATCHER_IMPLEMENTATION
   887: #define GET_REGISTER_MATCHER
   888: #include "HexagonGenAsmMatcher.inc"
   889: 
   890: static bool previousEqual(OperandVector &Operands, size_t Index,
   891:                           StringRef String) {
   892:   if (Index >= Operands.size())
   893:     return false;
   894:   MCParsedAsmOperand &Operand = *Operands[Operands.size() - Index - 1];
   895:   if (!Operand.isToken())
   896:     return false;
   897:   return static_cast<HexagonOperand &>(Operand).getToken().equals_insensitive(
   898:       String);
   899: }
   900: 
   901: static bool previousIsLoop(OperandVector &Operands, size_t Index) {
   902:   return previousEqual(Operands, Index, "loop0") ||
   903:          previousEqual(Operands, Index, "loop1") ||
   904:          previousEqual(Operands, Index, "sp1loop0") ||
   905:          previousEqual(Operands, Index, "sp2loop0") ||
   906:          previousEqual(Operands, Index, "sp3loop0");
   907: }
   908: 
   909: bool HexagonAsmParser::splitIdentifier(OperandVector &Operands) {
   910:   AsmToken const &Token = getParser().getTok();
   911:   StringRef String = Token.getString();
   912:   SMLoc Loc = Token.getLoc();
   913:   Lex();
   914:   do {
   915:     std::pair<StringRef, StringRef> HeadTail = String.split('.');
   916:     if (!HeadTail.first.empty())
   917:       Operands.push_back(
   918:           HexagonOperand::CreateToken(getContext(), HeadTail.first, Loc));
   919:     if (!HeadTail.second.empty())
   920:       Operands.push_back(HexagonOperand::CreateToken(
   921:           getContext(), String.substr(HeadTail.first.size(), 1), Loc));
   922:     String = HeadTail.second;
   923:   } while (!String.empty());
   924:   return false;
   925: }
   926: 
   927: bool HexagonAsmParser::parseOperand(OperandVector &Operands) {
   928:   MCRegister Register;
   929:   SMLoc Begin;
   930:   SMLoc End;
   931:   AsmLexer &Lexer = getLexer();
   932:   if (!parseRegister(Register, Begin, End)) {
   933:     if (!ErrorMissingParenthesis)
   934:       switch (Register.id()) {
   935:       default:
   936:         break;
   937:       case Hexagon::P0:
   938:       case Hexagon::P1:
   939:       case Hexagon::P2:
   940:       case Hexagon::P3:
   941:         if (previousEqual(Operands, 0, "if")) {
   942:           if (WarnMissingParenthesis)
   943:             Warning(Begin, "Missing parenthesis around predicate register");
   944:           static char const *LParen = "(";
   945:           static char const *RParen = ")";
   946:           Operands.push_back(
   947:               HexagonOperand::CreateToken(getContext(), LParen, Begin));
   948:           Operands.push_back(
   949:               HexagonOperand::CreateReg(getContext(), Register, Begin, End));
   950:           const AsmToken &MaybeDotNew = Lexer.getTok();
   951:           if (MaybeDotNew.is(AsmToken::TokenKind::Identifier) &&
   952:               MaybeDotNew.getString().equals_insensitive(".new"))
   953:             splitIdentifier(Operands);
   954:           Operands.push_back(
   955:               HexagonOperand::CreateToken(getContext(), RParen, Begin));
   956:           return false;
   957:         }
   958:         if (previousEqual(Operands, 0, "!") &&
   959:             previousEqual(Operands, 1, "if")) {
   960:           if (WarnMissingParenthesis)
   961:             Warning(Begin, "Missing parenthesis around predicate register");
   962:           static char const *LParen = "(";
   963:           static char const *RParen = ")";
   964:           Operands.insert(Operands.end() - 1, HexagonOperand::CreateToken(
   965:                                                   getContext(), LParen, Begin));
   966:           Operands.push_back(
   967:               HexagonOperand::CreateReg(getContext(), Register, Begin, End));
   968:           const AsmToken &MaybeDotNew = Lexer.getTok();
   969:           if (MaybeDotNew.is(AsmToken::TokenKind::Identifier) &&
   970:               MaybeDotNew.getString().equals_insensitive(".new"))
   971:             splitIdentifier(Operands);
   972:           Operands.push_back(
   973:               HexagonOperand::CreateToken(getContext(), RParen, Begin));
   974:           return false;
   975:         }
   976:         break;
   977:       }
   978:     Operands.push_back(
   979:         HexagonOperand::CreateReg(getContext(), Register, Begin, End));
   980:     return false;
   981:   }
   982:   return splitIdentifier(Operands);
   983: }
   984: 
   985: bool HexagonAsmParser::isLabel(AsmToken &Token) {
   986:   AsmLexer &Lexer = getLexer();
   987:   AsmToken const &Second = Lexer.getTok();
   988:   AsmToken Third = Lexer.peekTok();
   989:   StringRef String = Token.getString();
   990:   if (Token.is(AsmToken::TokenKind::LCurly) ||
   991:       Token.is(AsmToken::TokenKind::RCurly))
   992:     return false;
   993:   // special case for parsing vwhist256:sat
   994:   if (String.lower() == "vwhist256" && Second.is(AsmToken::Colon) &&
   995:       Third.getString().lower() == "sat")
   996:     return false;
   997:   if (!Token.is(AsmToken::TokenKind::Identifier))
   998:     return true;
   999:   if (!matchRegister(String.lower()))
  1000:     return true;
```
- EN: It imports headers such as HexagonGenAsmMatcher.inc, establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It declares or implements routines such as Lex, getLexer, getStreamer, HexagonMCEmitLocalCommonSymbol, ... (24 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里引入了 HexagonGenAsmMatcher.inc 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里声明或实现了 Lex, getLexer, getStreamer, HexagonMCEmitLocalCommonSymbol, ... (24 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 1001-1200 / 第 1001-1200 行

```cpp
  1001:   assert(Second.is(AsmToken::Colon));
  1002:   StringRef Raw(String.data(), Third.getString().data() - String.data() +
  1003:                                    Third.getString().size());
  1004:   std::string Collapsed = std::string(Raw);
  1005:   llvm::erase_if(Collapsed, isSpace);
  1006:   StringRef Whole = Collapsed;
  1007:   std::pair<StringRef, StringRef> DotSplit = Whole.split('.');
  1008:   if (!matchRegister(DotSplit.first.lower()))
  1009:     return true;
  1010:   return false;
  1011: }
  1012: 
  1013: bool HexagonAsmParser::tokenIsStartOfStatement(AsmToken::TokenKind Token) {
  1014:   return Token == AsmToken::LCurly || Token == AsmToken::RCurly;
  1015: }
  1016: 
  1017: bool HexagonAsmParser::handleNoncontigiousRegister(bool Contigious,
  1018:                                                    SMLoc &Loc) {
  1019:   if (!Contigious && ErrorNoncontigiousRegister) {
  1020:     Error(Loc, "Register name is not contigious");
  1021:     return true;
  1022:   }
  1023:   if (!Contigious && WarnNoncontigiousRegister)
  1024:     Warning(Loc, "Register name is not contigious");
  1025:   return false;
  1026: }
  1027: 
  1028: bool HexagonAsmParser::parseRegister(MCRegister &Reg, SMLoc &StartLoc,
  1029:                                      SMLoc &EndLoc) {
  1030:   return !tryParseRegister(Reg, StartLoc, EndLoc).isSuccess();
  1031: }
  1032: 
  1033: ParseStatus HexagonAsmParser::tryParseRegister(MCRegister &Reg, SMLoc &StartLoc,
  1034:                                                SMLoc &EndLoc) {
  1035:   AsmLexer &Lexer = getLexer();
  1036:   StartLoc = getLexer().getLoc();
  1037:   SmallVector<AsmToken, 5> Lookahead;
  1038:   StringRef RawString(Lexer.getTok().getString().data(), 0);
  1039:   bool Again = Lexer.is(AsmToken::Identifier);
  1040:   bool NeededWorkaround = false;
  1041:   while (Again) {
  1042:     AsmToken const &Token = Lexer.getTok();
  1043:     RawString = StringRef(RawString.data(), Token.getString().data() -
  1044:                                                 RawString.data() +
  1045:                                                 Token.getString().size());
  1046:     Lookahead.push_back(Token);
  1047:     Lexer.Lex();
  1048:     bool Contigious = Lexer.getTok().getString().data() ==
  1049:                       Lookahead.back().getString().data() +
  1050:                           Lookahead.back().getString().size();
  1051:     bool Type = Lexer.is(AsmToken::Identifier) || Lexer.is(AsmToken::Dot) ||
  1052:                 Lexer.is(AsmToken::Integer) || Lexer.is(AsmToken::Real) ||
  1053:                 Lexer.is(AsmToken::Colon);
  1054:     bool Workaround =
  1055:         Lexer.is(AsmToken::Colon) || Lookahead.back().is(AsmToken::Colon);
  1056:     Again = (Contigious && Type) || (Workaround && Type);
  1057:     NeededWorkaround = NeededWorkaround || (Again && !(Contigious && Type));
  1058:   }
  1059:   std::string Collapsed = std::string(RawString);
  1060:   llvm::erase_if(Collapsed, isSpace);
  1061:   StringRef FullString = Collapsed;
  1062:   std::pair<StringRef, StringRef> DotSplit = FullString.split('.');
  1063:   MCRegister DotReg = matchRegister(DotSplit.first.lower());
  1064:   if (DotReg && RegisterMatchesArch(DotReg)) {
  1065:     if (DotSplit.second.empty()) {
  1066:       Reg = DotReg;
  1067:       EndLoc = Lexer.getLoc();
  1068:       if (handleNoncontigiousRegister(!NeededWorkaround, StartLoc))
  1069:         return ParseStatus::NoMatch;
  1070:       return ParseStatus::Success;
  1071:     } else {
  1072:       Reg = DotReg;
  1073:       size_t First = RawString.find('.');
  1074:       StringRef DotString (RawString.data() + First, RawString.size() - First);
  1075:       Lexer.UnLex(AsmToken(AsmToken::Identifier, DotString));
  1076:       EndLoc = Lexer.getLoc();
  1077:       if (handleNoncontigiousRegister(!NeededWorkaround, StartLoc))
  1078:         return ParseStatus::NoMatch;
  1079:       return ParseStatus::Success;
  1080:     }
  1081:   }
  1082:   std::pair<StringRef, StringRef> ColonSplit = StringRef(FullString).split(':');
  1083:   MCRegister ColonReg = matchRegister(ColonSplit.first.lower());
  1084:   if (ColonReg && RegisterMatchesArch(DotReg)) {
  1085:     do {
  1086:       Lexer.UnLex(Lookahead.pop_back_val());
  1087:     } while (!Lookahead.empty() && !Lexer.is(AsmToken::Colon));
  1088:     Reg = ColonReg;
  1089:     EndLoc = Lexer.getLoc();
  1090:     if (handleNoncontigiousRegister(!NeededWorkaround, StartLoc))
  1091:       return ParseStatus::NoMatch;
  1092:     return ParseStatus::Success;
  1093:   }
  1094:   while (!Lookahead.empty()) {
  1095:     Lexer.UnLex(Lookahead.pop_back_val());
  1096:   }
  1097:   return ParseStatus::NoMatch;
  1098: }
  1099: 
  1100: bool HexagonAsmParser::implicitExpressionLocation(OperandVector &Operands) {
  1101:   if (previousEqual(Operands, 0, "call"))
  1102:     return true;
  1103:   if (previousEqual(Operands, 0, "jump"))
  1104:     if (!getLexer().getTok().is(AsmToken::Colon))
  1105:       return true;
  1106:   if (previousEqual(Operands, 0, "(") && previousIsLoop(Operands, 1))
  1107:     return true;
  1108:   if (previousEqual(Operands, 1, ":") && previousEqual(Operands, 2, "jump") &&
  1109:       (previousEqual(Operands, 0, "nt") || previousEqual(Operands, 0, "t")))
  1110:     return true;
  1111:   return false;
  1112: }
  1113: 
  1114: bool HexagonAsmParser::parseExpression(MCExpr const *&Expr) {
  1115:   SmallVector<AsmToken, 4> Tokens;
  1116:   AsmLexer &Lexer = getLexer();
  1117:   bool Done = false;
  1118:   static char const *Comma = ",";
  1119:   do {
  1120:     Tokens.emplace_back(Lexer.getTok());
  1121:     Lex();
  1122:     switch (Tokens.back().getKind()) {
  1123:     case AsmToken::TokenKind::Hash:
  1124:       if (Tokens.size() > 1)
  1125:         if ((Tokens.end() - 2)->getKind() == AsmToken::TokenKind::Plus) {
  1126:           Tokens.insert(Tokens.end() - 2,
  1127:                         AsmToken(AsmToken::TokenKind::Comma, Comma));
  1128:           Done = true;
  1129:         }
  1130:       break;
  1131:     case AsmToken::TokenKind::RCurly:
  1132:     case AsmToken::TokenKind::EndOfStatement:
  1133:     case AsmToken::TokenKind::Eof:
  1134:       Done = true;
  1135:       break;
  1136:     default:
  1137:       break;
  1138:     }
  1139:   } while (!Done);
  1140:   while (!Tokens.empty()) {
  1141:     Lexer.UnLex(Tokens.back());
  1142:     Tokens.pop_back();
  1143:   }
  1144:   SMLoc Loc = Lexer.getLoc();
  1145:   return getParser().parseExpression(Expr, Loc);
  1146: }
  1147: 
  1148: bool HexagonAsmParser::parseExpressionOrOperand(OperandVector &Operands) {
  1149:   if (implicitExpressionLocation(Operands)) {
  1150:     MCAsmParser &Parser = getParser();
  1151:     SMLoc Loc = Parser.getLexer().getLoc();
  1152:     MCExpr const *Expr = nullptr;
  1153:     bool Error = parseExpression(Expr);
  1154:     Expr = HexagonMCExpr::create(Expr, getContext());
  1155:     if (!Error)
  1156:       Operands.push_back(
  1157:           HexagonOperand::CreateImm(getContext(), Expr, Loc, Loc));
  1158:     return Error;
  1159:   }
  1160:   return parseOperand(Operands);
  1161: }
  1162: 
  1163: /// Parse an instruction.
  1164: bool HexagonAsmParser::parseInstruction(OperandVector &Operands) {
  1165:   MCAsmParser &Parser = getParser();
  1166:   AsmLexer &Lexer = getLexer();
  1167:   while (true) {
  1168:     AsmToken const &Token = Parser.getTok();
  1169:     switch (Token.getKind()) {
  1170:     case AsmToken::Eof:
  1171:     case AsmToken::EndOfStatement: {
  1172:       Lex();
  1173:       return false;
  1174:     }
  1175:     case AsmToken::LCurly: {
  1176:       if (!Operands.empty())
  1177:         return true;
  1178:       Operands.push_back(HexagonOperand::CreateToken(
  1179:           getContext(), Token.getString(), Token.getLoc()));
  1180:       Lex();
  1181:       return false;
  1182:     }
  1183:     case AsmToken::RCurly: {
  1184:       if (Operands.empty()) {
  1185:         Operands.push_back(HexagonOperand::CreateToken(
  1186:             getContext(), Token.getString(), Token.getLoc()));
  1187:         Lex();
  1188:       }
  1189:       return false;
  1190:     }
  1191:     case AsmToken::Comma: {
  1192:       Lex();
  1193:       continue;
  1194:     }
  1195:     case AsmToken::EqualEqual:
  1196:     case AsmToken::ExclaimEqual:
  1197:     case AsmToken::GreaterEqual:
  1198:     case AsmToken::GreaterGreater:
  1199:     case AsmToken::LessEqual:
  1200:     case AsmToken::LessLess: {
```
- EN: It declares or implements routines such as assert, Raw, std::string, llvm::erase_if, ... (34 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonAsmParser, HexagonMCExpr, HexagonOperand, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 assert, Raw, std::string, llvm::erase_if, ... (34 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonAsmParser, HexagonMCExpr, HexagonOperand，说明了它与同级后端组件的连接关系。

### Lines 1201-1400 / 第 1201-1400 行

```cpp
  1201:       Operands.push_back(HexagonOperand::CreateToken(
  1202:           getContext(), Token.getString().substr(0, 1), Token.getLoc()));
  1203:       Operands.push_back(HexagonOperand::CreateToken(
  1204:           getContext(), Token.getString().substr(1, 1), Token.getLoc()));
  1205:       Lex();
  1206:       continue;
  1207:     }
  1208:     case AsmToken::Hash: {
  1209:       bool MustNotExtend = false;
  1210:       bool ImplicitExpression = implicitExpressionLocation(Operands);
  1211:       SMLoc ExprLoc = Lexer.getLoc();
  1212:       if (!ImplicitExpression)
  1213:         Operands.push_back(HexagonOperand::CreateToken(
  1214:             getContext(), Token.getString(), Token.getLoc()));
  1215:       Lex();
  1216:       bool MustExtend = false;
  1217:       bool HiOnly = false;
  1218:       bool LoOnly = false;
  1219:       if (Lexer.is(AsmToken::Hash)) {
  1220:         Lex();
  1221:         MustExtend = true;
  1222:       } else if (ImplicitExpression)
  1223:         MustNotExtend = true;
  1224:       AsmToken const &Token = Parser.getTok();
  1225:       if (Token.is(AsmToken::Identifier)) {
  1226:         StringRef String = Token.getString();
  1227:         if (String.lower() == "hi") {
  1228:           HiOnly = true;
  1229:         } else if (String.lower() == "lo") {
  1230:           LoOnly = true;
  1231:         }
  1232:         if (HiOnly || LoOnly) {
  1233:           AsmToken LParen = Lexer.peekTok();
  1234:           if (!LParen.is(AsmToken::LParen)) {
  1235:             HiOnly = false;
  1236:             LoOnly = false;
  1237:           } else {
  1238:             Lex();
  1239:           }
  1240:         }
  1241:       }
  1242:       MCExpr const *Expr = nullptr;
  1243:       if (parseExpression(Expr))
  1244:         return true;
  1245:       int64_t Value;
  1246:       MCContext &Context = Parser.getContext();
  1247:       assert(Expr != nullptr);
  1248:       if (Expr->evaluateAsAbsolute(Value)) {
  1249:         if (HiOnly)
  1250:           Expr = MCBinaryExpr::createLShr(
  1251:               Expr, MCConstantExpr::create(16, Context), Context);
  1252:         if (HiOnly || LoOnly)
  1253:           Expr = MCBinaryExpr::createAnd(
  1254:               Expr, MCConstantExpr::create(0xffff, Context), Context);
  1255:       } else {
  1256:         MCValue Value;
  1257:         if (Expr->evaluateAsRelocatable(Value, nullptr)) {
  1258:           if (!Value.isAbsolute()) {
  1259:             switch (HexagonMCExpr::VariantKind(Value.getSpecifier())) {
  1260:             case HexagonMCExpr::VK_TPREL:
  1261:             case HexagonMCExpr::VK_DTPREL:
  1262:               // Don't lazy extend these expression variants
  1263:               MustNotExtend = !MustExtend;
  1264:               break;
  1265:             default:
  1266:               break;
  1267:             }
  1268:           }
  1269:         }
  1270:       }
  1271:       Expr = HexagonMCExpr::create(Expr, Context);
  1272:       HexagonMCInstrInfo::setMustNotExtend(*Expr, MustNotExtend);
  1273:       HexagonMCInstrInfo::setMustExtend(*Expr, MustExtend);
  1274:       std::unique_ptr<HexagonOperand> Operand =
  1275:           HexagonOperand::CreateImm(getContext(), Expr, ExprLoc, ExprLoc);
  1276:       Operands.push_back(std::move(Operand));
  1277:       continue;
  1278:     }
  1279:     default:
  1280:       break;
  1281:     }
  1282:     if (parseExpressionOrOperand(Operands))
  1283:       return true;
  1284:   }
  1285: }
  1286: 
  1287: bool HexagonAsmParser::parseInstruction(ParseInstructionInfo &Info,
  1288:                                         StringRef Name, AsmToken ID,
  1289:                                         OperandVector &Operands) {
  1290:   getLexer().UnLex(ID);
  1291:   return parseInstruction(Operands);
  1292: }
  1293: 
  1294: static MCInst makeCombineInst(int opCode, MCOperand &Rdd, MCOperand &MO1,
  1295:                               MCOperand &MO2) {
  1296:   MCInst TmpInst;
  1297:   TmpInst.setOpcode(opCode);
  1298:   TmpInst.addOperand(Rdd);
  1299:   TmpInst.addOperand(MO1);
  1300:   TmpInst.addOperand(MO2);
  1301: 
  1302:   return TmpInst;
  1303: }
  1304: 
  1305: // Define this matcher function after the auto-generated include so we
  1306: // have the match class enum definitions.
  1307: unsigned HexagonAsmParser::validateTargetOperandClass(MCParsedAsmOperand &AsmOp,
  1308:                                                       unsigned Kind) {
  1309:   HexagonOperand *Op = static_cast<HexagonOperand *>(&AsmOp);
  1310: 
  1311:   switch (Kind) {
  1312:   case MCK_0: {
  1313:     int64_t Value;
  1314:     return Op->isImm() && Op->Imm.Val->evaluateAsAbsolute(Value) && Value == 0
  1315:                ? Match_Success
  1316:                : Match_InvalidOperand;
  1317:   }
  1318:   case MCK_1: {
  1319:     int64_t Value;
  1320:     return Op->isImm() && Op->Imm.Val->evaluateAsAbsolute(Value) && Value == 1
  1321:                ? Match_Success
  1322:                : Match_InvalidOperand;
  1323:   }
  1324:   }
  1325:   if (Op->Kind == HexagonOperand::Token && Kind != InvalidMatchClass) {
  1326:     StringRef myStringRef = StringRef(Op->Tok.Data, Op->Tok.Length);
  1327:     if (matchTokenString(myStringRef.lower()) == (MatchClassKind)Kind)
  1328:       return Match_Success;
  1329:     if (matchTokenString(myStringRef.upper()) == (MatchClassKind)Kind)
  1330:       return Match_Success;
  1331:   }
  1332: 
  1333:   LLVM_DEBUG(dbgs() << "Unmatched Operand:");
  1334:   LLVM_DEBUG(Op->dump());
  1335:   LLVM_DEBUG(dbgs() << "\n");
  1336: 
  1337:   return Match_InvalidOperand;
  1338: }
  1339: 
  1340: // FIXME: Calls to OutOfRange should propagate failure up to parseStatement.
  1341: bool HexagonAsmParser::OutOfRange(SMLoc IDLoc, long long Val, long long Max) {
  1342:   std::string errStr;
  1343:   raw_string_ostream ES(errStr);
  1344:   ES << "value " << Val << "(" << format_hex(Val, 0) << ") out of range: ";
  1345:   if (Max >= 0)
  1346:     ES << "0-" << Max;
  1347:   else
  1348:     ES << Max << "-" << (-Max - 1);
  1349:   return Parser.printError(IDLoc, ES.str());
  1350: }
  1351: 
  1352: int HexagonAsmParser::processInstruction(MCInst &Inst,
  1353:                                          OperandVector const &Operands,
  1354:                                          SMLoc IDLoc) {
  1355:   MCContext &Context = getParser().getContext();
  1356:   const MCRegisterInfo *RI = getContext().getRegisterInfo();
  1357:   const std::string r = "r";
  1358:   const std::string Colon = ":";
  1359:   using RegPairVals = std::pair<unsigned, unsigned>;
  1360:   auto GetRegPair = [this, r](RegPairVals RegPair) {
  1361:     const std::string R1 = r + utostr(RegPair.first);
  1362:     const std::string R2 = r + utostr(RegPair.second);
  1363: 
  1364:     return std::make_pair(matchRegister(R1), matchRegister(R2));
  1365:   };
  1366:   auto GetScalarRegs = [RI, GetRegPair](MCRegister RegPair) {
  1367:     const unsigned Lower = RI->getEncodingValue(RegPair);
  1368:     const RegPairVals RegPair_ = std::make_pair(Lower + 1, Lower);
  1369: 
  1370:     return GetRegPair(RegPair_);
  1371:   };
  1372:   auto GetVecRegs = [GetRegPair](MCRegister VecRegPair) {
  1373:     const RegPairVals RegPair =
  1374:         HexagonMCInstrInfo::GetVecRegPairIndices(VecRegPair);
  1375: 
  1376:     return GetRegPair(RegPair);
  1377:   };
  1378: 
  1379:   bool is32bit = false; // used to distinguish between CONST32 and CONST64
  1380:   switch (Inst.getOpcode()) {
  1381:   default:
  1382:     if (HexagonMCInstrInfo::getDesc(MII, Inst).isPseudo()) {
  1383:       SMDiagnostic Diag = getSourceManager().GetMessage(
  1384:           IDLoc, SourceMgr::DK_Error,
  1385:           "Found pseudo instruction with no expansion");
  1386:       Diag.print("", errs());
  1387:       report_fatal_error("Invalid pseudo instruction");
  1388:     }
  1389:     break;
  1390: 
  1391:   case Hexagon::J2_trap1:
  1392:     if (!getSTI().hasFeature(Hexagon::ArchV65)) {
  1393:       MCOperand &Rx = Inst.getOperand(0);
  1394:       MCOperand &Ry = Inst.getOperand(1);
  1395:       if (Rx.getReg() != Hexagon::R0 || Ry.getReg() != Hexagon::R0) {
  1396:         Error(IDLoc, "trap1 can only have register r0 as operand");
  1397:         return Match_InvalidOperand;
  1398:       }
  1399:     }
  1400:     break;
```
- EN: It declares or implements routines such as push_back, Lex, implicitExpressionLocation, getLoc, ... (37 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonOperand, HexagonMCExpr, HexagonMCInstrInfo, HexagonAsmParser, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 push_back, Lex, implicitExpressionLocation, getLoc, ... (37 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonOperand, HexagonMCExpr, HexagonMCInstrInfo, HexagonAsmParser，说明了它与同级后端组件的连接关系。

### Lines 1401-1600 / 第 1401-1600 行

```cpp
  1401: 
  1402:   case Hexagon::A2_iconst: {
  1403:     Inst.setOpcode(Hexagon::A2_addi);
  1404:     MCOperand Reg = Inst.getOperand(0);
  1405:     MCOperand S27 = Inst.getOperand(1);
  1406:     HexagonMCInstrInfo::setMustNotExtend(*S27.getExpr());
  1407:     HexagonMCInstrInfo::setS27_2_reloc(*S27.getExpr());
  1408:     Inst.clear();
  1409:     Inst.addOperand(Reg);
  1410:     Inst.addOperand(MCOperand::createReg(Hexagon::R0));
  1411:     Inst.addOperand(S27);
  1412:     break;
  1413:   }
  1414:   case Hexagon::M4_mpyrr_addr:
  1415:   case Hexagon::S4_addi_asl_ri:
  1416:   case Hexagon::S4_addi_lsr_ri:
  1417:   case Hexagon::S4_andi_asl_ri:
  1418:   case Hexagon::S4_andi_lsr_ri:
  1419:   case Hexagon::S4_ori_asl_ri:
  1420:   case Hexagon::S4_ori_lsr_ri:
  1421:   case Hexagon::S4_or_andix:
  1422:   case Hexagon::S4_subi_asl_ri:
  1423:   case Hexagon::S4_subi_lsr_ri: {
  1424:     MCOperand &Ry = Inst.getOperand(0);
  1425:     MCOperand &src = Inst.getOperand(2);
  1426:     if (RI->getEncodingValue(Ry.getReg()) != RI->getEncodingValue(src.getReg()))
  1427:       return Match_InvalidOperand;
  1428:     break;
  1429:   }
  1430: 
  1431:   case Hexagon::C2_cmpgei: {
  1432:     MCOperand &MO = Inst.getOperand(2);
  1433:     MO.setExpr(HexagonMCExpr::create(
  1434:         MCBinaryExpr::createSub(MO.getExpr(),
  1435:                                 MCConstantExpr::create(1, Context), Context),
  1436:         Context));
  1437:     Inst.setOpcode(Hexagon::C2_cmpgti);
  1438:     break;
  1439:   }
  1440: 
  1441:   case Hexagon::C2_cmpgeui: {
  1442:     MCOperand &MO = Inst.getOperand(2);
  1443:     int64_t Value;
  1444:     bool Success = MO.getExpr()->evaluateAsAbsolute(Value);
  1445:     (void)Success;
  1446:     assert(Success && "Assured by matcher");
  1447:     if (Value == 0) {
  1448:       MCInst TmpInst;
  1449:       MCOperand &Pd = Inst.getOperand(0);
  1450:       MCOperand &Rt = Inst.getOperand(1);
  1451:       TmpInst.setOpcode(Hexagon::C2_cmpeq);
  1452:       TmpInst.addOperand(Pd);
  1453:       TmpInst.addOperand(Rt);
  1454:       TmpInst.addOperand(Rt);
  1455:       Inst = TmpInst;
  1456:     } else {
  1457:       MO.setExpr(HexagonMCExpr::create(
  1458:           MCBinaryExpr::createSub(MO.getExpr(),
  1459:                                   MCConstantExpr::create(1, Context), Context),
  1460:           Context));
  1461:       Inst.setOpcode(Hexagon::C2_cmpgtui);
  1462:     }
  1463:     break;
  1464:   }
  1465: 
  1466:   // Translate a "$Rdd = $Rss" to "$Rdd = combine($Rs, $Rt)"
  1467:   case Hexagon::A2_tfrp: {
  1468:     MCOperand &MO = Inst.getOperand(1);
  1469:     const std::pair<MCRegister, MCRegister> RegPair =
  1470:         GetScalarRegs(MO.getReg());
  1471:     MO.setReg(RegPair.first);
  1472:     Inst.addOperand(MCOperand::createReg(RegPair.second));
  1473:     Inst.setOpcode(Hexagon::A2_combinew);
  1474:     break;
  1475:   }
  1476: 
  1477:   case Hexagon::A2_tfrpt:
  1478:   case Hexagon::A2_tfrpf: {
  1479:     MCOperand &MO = Inst.getOperand(2);
  1480:     const std::pair<MCRegister, MCRegister> RegPair =
  1481:         GetScalarRegs(MO.getReg());
  1482:     MO.setReg(RegPair.first);
  1483:     Inst.addOperand(MCOperand::createReg(RegPair.second));
  1484:     Inst.setOpcode((Inst.getOpcode() == Hexagon::A2_tfrpt)
  1485:                        ? Hexagon::C2_ccombinewt
  1486:                        : Hexagon::C2_ccombinewf);
  1487:     break;
  1488:   }
  1489:   case Hexagon::A2_tfrptnew:
  1490:   case Hexagon::A2_tfrpfnew: {
  1491:     MCOperand &MO = Inst.getOperand(2);
  1492:     const std::pair<MCRegister, MCRegister> RegPair =
  1493:         GetScalarRegs(MO.getReg());
  1494:     MO.setReg(RegPair.first);
  1495:     Inst.addOperand(MCOperand::createReg(RegPair.second));
  1496:     Inst.setOpcode((Inst.getOpcode() == Hexagon::A2_tfrptnew)
  1497:                        ? Hexagon::C2_ccombinewnewt
  1498:                        : Hexagon::C2_ccombinewnewf);
  1499:     break;
  1500:   }
  1501: 
  1502:   // Translate a "$Vdd = $Vss" to "$Vdd = vcombine($Vs, $Vt)"
  1503:   case Hexagon::V6_vassignp: {
  1504:     MCOperand &MO = Inst.getOperand(1);
  1505:     const std::pair<MCRegister, MCRegister> RegPair = GetVecRegs(MO.getReg());
  1506:     MO.setReg(RegPair.first);
  1507:     Inst.addOperand(MCOperand::createReg(RegPair.second));
  1508:     Inst.setOpcode(Hexagon::V6_vcombine);
  1509:     break;
  1510:   }
  1511: 
  1512:   // Translate a "$Rx =  CONST32(#imm)" to "$Rx = memw(gp+#LABEL) "
  1513:   case Hexagon::CONST32:
  1514:     is32bit = true;
  1515:     [[fallthrough]];
  1516:   // Translate a "$Rx:y =  CONST64(#imm)" to "$Rx:y = memd(gp+#LABEL) "
  1517:   case Hexagon::CONST64:
  1518:     // FIXME: need better way to detect AsmStreamer (upstream removed getKind())
  1519:     if (!Parser.getStreamer().hasRawTextSupport()) {
  1520:       MCELFStreamer *MES = static_cast<MCELFStreamer *>(&Parser.getStreamer());
  1521:       MCOperand &MO_1 = Inst.getOperand(1);
  1522:       MCOperand &MO_0 = Inst.getOperand(0);
  1523: 
  1524:       // push section onto section stack
  1525:       MES->pushSection();
  1526: 
  1527:       std::string myCharStr;
  1528:       MCSectionELF *mySection;
  1529: 
  1530:       // check if this as an immediate or a symbol
  1531:       int64_t Value;
  1532:       bool Absolute = MO_1.getExpr()->evaluateAsAbsolute(Value);
  1533:       if (Absolute) {
  1534:         // Create a new section - one for each constant
  1535:         // Some or all of the zeros are replaced with the given immediate.
  1536:         if (is32bit) {
  1537:           std::string myImmStr = utohexstr(static_cast<uint32_t>(Value));
  1538:           myCharStr = StringRef(".gnu.linkonce.l4.CONST_00000000")
  1539:                           .drop_back(myImmStr.size())
  1540:                           .str() +
  1541:                       myImmStr;
  1542:         } else {
  1543:           std::string myImmStr = utohexstr(Value);
  1544:           myCharStr = StringRef(".gnu.linkonce.l8.CONST_0000000000000000")
  1545:                           .drop_back(myImmStr.size())
  1546:                           .str() +
  1547:                       myImmStr;
  1548:         }
  1549: 
  1550:         mySection = getContext().getELFSection(myCharStr, ELF::SHT_PROGBITS,
  1551:                                                ELF::SHF_ALLOC | ELF::SHF_WRITE);
  1552:       } else if (MO_1.isExpr()) {
  1553:         // .lita - for expressions
  1554:         myCharStr = ".lita";
  1555:         mySection = getContext().getELFSection(myCharStr, ELF::SHT_PROGBITS,
  1556:                                                ELF::SHF_ALLOC | ELF::SHF_WRITE);
  1557:       } else
  1558:         llvm_unreachable("unexpected type of machine operand!");
  1559: 
  1560:       MES->switchSection(mySection);
  1561:       unsigned byteSize = is32bit ? 4 : 8;
  1562:       getStreamer().emitCodeAlignment(Align(byteSize), &getSTI(), byteSize);
  1563: 
  1564:       MCSymbol *Sym;
  1565: 
  1566:       // for symbols, get rid of prepended ".gnu.linkonce.lx."
  1567: 
  1568:       // emit symbol if needed
  1569:       if (Absolute) {
  1570:         Sym = getContext().getOrCreateSymbol(StringRef(myCharStr.c_str() + 16));
  1571:         if (Sym->isUndefined()) {
  1572:           getStreamer().emitLabel(Sym);
  1573:           getStreamer().emitSymbolAttribute(Sym, MCSA_Global);
  1574:           getStreamer().emitIntValue(Value, byteSize);
  1575:         }
  1576:       } else if (MO_1.isExpr()) {
  1577:         const char *StringStart = nullptr;
  1578:         const char *StringEnd = nullptr;
  1579:         if (*Operands[4]->getStartLoc().getPointer() == '#') {
  1580:           StringStart = Operands[5]->getStartLoc().getPointer();
  1581:           StringEnd = Operands[6]->getStartLoc().getPointer();
  1582:         } else { // no pound
  1583:           StringStart = Operands[4]->getStartLoc().getPointer();
  1584:           StringEnd = Operands[5]->getStartLoc().getPointer();
  1585:         }
  1586: 
  1587:         unsigned size = StringEnd - StringStart;
  1588:         std::string DotConst = ".CONST_";
  1589:         Sym = getContext().getOrCreateSymbol(DotConst +
  1590:                                              StringRef(StringStart, size));
  1591: 
  1592:         if (Sym->isUndefined()) {
  1593:           // case where symbol is not yet defined: emit symbol
  1594:           getStreamer().emitLabel(Sym);
  1595:           getStreamer().emitSymbolAttribute(Sym, MCSA_Local);
  1596:           getStreamer().emitValue(MO_1.getExpr(), 4);
  1597:         }
  1598:       } else
  1599:         llvm_unreachable("unexpected type of machine operand!");
  1600: 
```
- EN: It declares or implements routines such as setOpcode, getOperand, HexagonMCInstrInfo::setMustNotExtend, HexagonMCInstrInfo::setS27_2_reloc, ... (20 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonMCInstrInfo, HexagonMCExpr, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 setOpcode, getOperand, HexagonMCInstrInfo::setMustNotExtend, HexagonMCInstrInfo::setS27_2_reloc, ... (20 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonMCInstrInfo, HexagonMCExpr，说明了它与同级后端组件的连接关系。

### Lines 1601-1800 / 第 1601-1800 行

```cpp
  1601:       MES->popSection();
  1602: 
  1603:       if (Sym) {
  1604:         MCInst TmpInst;
  1605:         if (is32bit) // 32 bit
  1606:           TmpInst.setOpcode(Hexagon::L2_loadrigp);
  1607:         else // 64 bit
  1608:           TmpInst.setOpcode(Hexagon::L2_loadrdgp);
  1609: 
  1610:         TmpInst.addOperand(MO_0);
  1611:         TmpInst.addOperand(MCOperand::createExpr(HexagonMCExpr::create(
  1612:             MCSymbolRefExpr::create(Sym, getContext()), getContext())));
  1613:         Inst = TmpInst;
  1614:       }
  1615:     }
  1616:     break;
  1617: 
  1618:   // Translate a "$Rdd = #-imm" to "$Rdd = combine(#[-1,0], #-imm)"
  1619:   case Hexagon::A2_tfrpi: {
  1620:     MCOperand &Rdd = Inst.getOperand(0);
  1621:     MCOperand &MO = Inst.getOperand(1);
  1622:     int64_t Value;
  1623:     int sVal = (MO.getExpr()->evaluateAsAbsolute(Value) && Value < 0) ? -1 : 0;
  1624:     MCOperand imm(MCOperand::createExpr(
  1625:         HexagonMCExpr::create(MCConstantExpr::create(sVal, Context), Context)));
  1626:     Inst = makeCombineInst(Hexagon::A2_combineii, Rdd, imm, MO);
  1627:     break;
  1628:   }
  1629: 
  1630:   // Translate a "$Rdd = [#]#imm" to "$Rdd = combine(#, [#]#imm)"
  1631:   case Hexagon::TFRI64_V4: {
  1632:     MCOperand &Rdd = Inst.getOperand(0);
  1633:     MCOperand &MO = Inst.getOperand(1);
  1634:     int64_t Value;
  1635:     if (MO.getExpr()->evaluateAsAbsolute(Value)) {
  1636:       int s8 = Hi_32(Value);
  1637:       if (!isInt<8>(s8))
  1638:         OutOfRange(IDLoc, s8, -128);
  1639:       MCOperand imm(MCOperand::createExpr(HexagonMCExpr::create(
  1640:           MCConstantExpr::create(s8, Context), Context))); // upper 32
  1641:       auto Expr = HexagonMCExpr::create(
  1642:           MCConstantExpr::create(Lo_32(Value), Context), Context);
  1643:       HexagonMCInstrInfo::setMustExtend(
  1644:           *Expr, HexagonMCInstrInfo::mustExtend(*MO.getExpr()));
  1645:       MCOperand imm2(MCOperand::createExpr(Expr)); // lower 32
  1646:       Inst = makeCombineInst(Hexagon::A4_combineii, Rdd, imm, imm2);
  1647:     } else {
  1648:       MCOperand imm(MCOperand::createExpr(HexagonMCExpr::create(
  1649:           MCConstantExpr::create(0, Context), Context))); // upper 32
  1650:       Inst = makeCombineInst(Hexagon::A4_combineii, Rdd, imm, MO);
  1651:     }
  1652:     break;
  1653:   }
  1654: 
  1655:   // Handle $Rdd = combine(##imm, #imm)"
  1656:   case Hexagon::TFRI64_V2_ext: {
  1657:     MCOperand &Rdd = Inst.getOperand(0);
  1658:     MCOperand &MO1 = Inst.getOperand(1);
  1659:     MCOperand &MO2 = Inst.getOperand(2);
  1660:     int64_t Value;
  1661:     if (MO2.getExpr()->evaluateAsAbsolute(Value)) {
  1662:       int s8 = Value;
  1663:       if (s8 < -128 || s8 > 127)
  1664:         OutOfRange(IDLoc, s8, -128);
  1665:     }
  1666:     Inst = makeCombineInst(Hexagon::A2_combineii, Rdd, MO1, MO2);
  1667:     break;
  1668:   }
  1669: 
  1670:   // Handle $Rdd = combine(#imm, ##imm)"
  1671:   case Hexagon::A4_combineii: {
  1672:     MCOperand &Rdd = Inst.getOperand(0);
  1673:     MCOperand &MO1 = Inst.getOperand(1);
  1674:     int64_t Value;
  1675:     if (MO1.getExpr()->evaluateAsAbsolute(Value)) {
  1676:       int s8 = Value;
  1677:       if (s8 < -128 || s8 > 127)
  1678:         OutOfRange(IDLoc, s8, -128);
  1679:     }
  1680:     MCOperand &MO2 = Inst.getOperand(2);
  1681:     Inst = makeCombineInst(Hexagon::A4_combineii, Rdd, MO1, MO2);
  1682:     break;
  1683:   }
  1684: 
  1685:   case Hexagon::S2_tableidxb_goodsyntax:
  1686:     Inst.setOpcode(Hexagon::S2_tableidxb);
  1687:     break;
  1688: 
  1689:   case Hexagon::S2_tableidxh_goodsyntax: {
  1690:     MCInst TmpInst;
  1691:     MCOperand &Rx = Inst.getOperand(0);
  1692:     MCOperand &Rs = Inst.getOperand(2);
  1693:     MCOperand &Imm4 = Inst.getOperand(3);
  1694:     MCOperand &Imm6 = Inst.getOperand(4);
  1695:     Imm6.setExpr(HexagonMCExpr::create(
  1696:         MCBinaryExpr::createSub(Imm6.getExpr(),
  1697:                                 MCConstantExpr::create(1, Context), Context),
  1698:         Context));
  1699:     TmpInst.setOpcode(Hexagon::S2_tableidxh);
  1700:     TmpInst.addOperand(Rx);
  1701:     TmpInst.addOperand(Rx);
  1702:     TmpInst.addOperand(Rs);
  1703:     TmpInst.addOperand(Imm4);
  1704:     TmpInst.addOperand(Imm6);
  1705:     Inst = TmpInst;
  1706:     break;
  1707:   }
  1708: 
  1709:   case Hexagon::S2_tableidxw_goodsyntax: {
  1710:     MCInst TmpInst;
  1711:     MCOperand &Rx = Inst.getOperand(0);
  1712:     MCOperand &Rs = Inst.getOperand(2);
  1713:     MCOperand &Imm4 = Inst.getOperand(3);
  1714:     MCOperand &Imm6 = Inst.getOperand(4);
  1715:     Imm6.setExpr(HexagonMCExpr::create(
  1716:         MCBinaryExpr::createSub(Imm6.getExpr(),
  1717:                                 MCConstantExpr::create(2, Context), Context),
  1718:         Context));
  1719:     TmpInst.setOpcode(Hexagon::S2_tableidxw);
  1720:     TmpInst.addOperand(Rx);
  1721:     TmpInst.addOperand(Rx);
  1722:     TmpInst.addOperand(Rs);
  1723:     TmpInst.addOperand(Imm4);
  1724:     TmpInst.addOperand(Imm6);
  1725:     Inst = TmpInst;
  1726:     break;
  1727:   }
  1728: 
  1729:   case Hexagon::S2_tableidxd_goodsyntax: {
  1730:     MCInst TmpInst;
  1731:     MCOperand &Rx = Inst.getOperand(0);
  1732:     MCOperand &Rs = Inst.getOperand(2);
  1733:     MCOperand &Imm4 = Inst.getOperand(3);
  1734:     MCOperand &Imm6 = Inst.getOperand(4);
  1735:     Imm6.setExpr(HexagonMCExpr::create(
  1736:         MCBinaryExpr::createSub(Imm6.getExpr(),
  1737:                                 MCConstantExpr::create(3, Context), Context),
  1738:         Context));
  1739:     TmpInst.setOpcode(Hexagon::S2_tableidxd);
  1740:     TmpInst.addOperand(Rx);
  1741:     TmpInst.addOperand(Rx);
  1742:     TmpInst.addOperand(Rs);
  1743:     TmpInst.addOperand(Imm4);
  1744:     TmpInst.addOperand(Imm6);
  1745:     Inst = TmpInst;
  1746:     break;
  1747:   }
  1748: 
  1749:   case Hexagon::M2_mpyui:
  1750:     Inst.setOpcode(Hexagon::M2_mpyi);
  1751:     break;
  1752:   case Hexagon::M2_mpysmi: {
  1753:     MCInst TmpInst;
  1754:     MCOperand &Rd = Inst.getOperand(0);
  1755:     MCOperand &Rs = Inst.getOperand(1);
  1756:     MCOperand &Imm = Inst.getOperand(2);
  1757:     int64_t Value;
  1758:     MCExpr const &Expr = *Imm.getExpr();
  1759:     bool Absolute = Expr.evaluateAsAbsolute(Value);
  1760:     if (!Absolute)
  1761:       return Match_InvalidOperand;
  1762:     if (!HexagonMCInstrInfo::mustExtend(Expr) &&
  1763:         ((Value <= -256) || Value >= 256))
  1764:       return Match_InvalidOperand;
  1765:     if (Value < 0 && Value > -256) {
  1766:       Imm.setExpr(HexagonMCExpr::create(
  1767:           MCConstantExpr::create(Value * -1, Context), Context));
  1768:       TmpInst.setOpcode(Hexagon::M2_mpysin);
  1769:     } else
  1770:       TmpInst.setOpcode(Hexagon::M2_mpysip);
  1771:     TmpInst.addOperand(Rd);
  1772:     TmpInst.addOperand(Rs);
  1773:     TmpInst.addOperand(Imm);
  1774:     Inst = TmpInst;
  1775:     break;
  1776:   }
  1777: 
  1778:   case Hexagon::S2_asr_i_r_rnd_goodsyntax: {
  1779:     MCOperand &Imm = Inst.getOperand(2);
  1780:     MCInst TmpInst;
  1781:     int64_t Value;
  1782:     bool Absolute = Imm.getExpr()->evaluateAsAbsolute(Value);
  1783:     if (!Absolute)
  1784:       return Match_InvalidOperand;
  1785:     if (Value == 0) { // convert to $Rd = $Rs
  1786:       TmpInst.setOpcode(Hexagon::A2_tfr);
  1787:       MCOperand &Rd = Inst.getOperand(0);
  1788:       MCOperand &Rs = Inst.getOperand(1);
  1789:       TmpInst.addOperand(Rd);
  1790:       TmpInst.addOperand(Rs);
  1791:     } else {
  1792:       Imm.setExpr(HexagonMCExpr::create(
  1793:           MCBinaryExpr::createSub(Imm.getExpr(),
  1794:                                   MCConstantExpr::create(1, Context), Context),
  1795:           Context));
  1796:       TmpInst.setOpcode(Hexagon::S2_asr_i_r_rnd);
  1797:       MCOperand &Rd = Inst.getOperand(0);
  1798:       MCOperand &Rs = Inst.getOperand(1);
  1799:       TmpInst.addOperand(Rd);
  1800:       TmpInst.addOperand(Rs);
```
- EN: It declares or implements routines such as popSection, setOpcode, addOperand, getOperand, ... (13 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonMCExpr, HexagonMCInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 popSection, setOpcode, addOperand, getOperand, ... (13 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonMCExpr, HexagonMCInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 1801-2000 / 第 1801-2000 行

```cpp
  1801:       TmpInst.addOperand(Imm);
  1802:     }
  1803:     Inst = TmpInst;
  1804:     break;
  1805:   }
  1806: 
  1807:   case Hexagon::S2_asr_i_p_rnd_goodsyntax: {
  1808:     MCOperand &Rdd = Inst.getOperand(0);
  1809:     MCOperand &Rss = Inst.getOperand(1);
  1810:     MCOperand &Imm = Inst.getOperand(2);
  1811:     int64_t Value;
  1812:     bool Absolute = Imm.getExpr()->evaluateAsAbsolute(Value);
  1813:     if (!Absolute)
  1814:       return Match_InvalidOperand;
  1815:     if (Value == 0) { // convert to $Rdd = combine ($Rs[0], $Rs[1])
  1816:       MCInst TmpInst;
  1817:       unsigned int RegPairNum = RI->getEncodingValue(Rss.getReg());
  1818:       std::string R1 = r + utostr(RegPairNum + 1);
  1819:       StringRef Reg1(R1);
  1820:       Rss.setReg(matchRegister(Reg1));
  1821:       // Add a new operand for the second register in the pair.
  1822:       std::string R2 = r + utostr(RegPairNum);
  1823:       StringRef Reg2(R2);
  1824:       TmpInst.setOpcode(Hexagon::A2_combinew);
  1825:       TmpInst.addOperand(Rdd);
  1826:       TmpInst.addOperand(Rss);
  1827:       TmpInst.addOperand(MCOperand::createReg(matchRegister(Reg2)));
  1828:       Inst = TmpInst;
  1829:     } else {
  1830:       Imm.setExpr(HexagonMCExpr::create(
  1831:           MCBinaryExpr::createSub(Imm.getExpr(),
  1832:                                   MCConstantExpr::create(1, Context), Context),
  1833:           Context));
  1834:       Inst.setOpcode(Hexagon::S2_asr_i_p_rnd);
  1835:     }
  1836:     break;
  1837:   }
  1838: 
  1839:   case Hexagon::A4_boundscheck: {
  1840:     MCOperand &Rs = Inst.getOperand(1);
  1841:     unsigned int RegNum = RI->getEncodingValue(Rs.getReg());
  1842:     if (RegNum & 1) { // Odd mapped to raw:hi, regpair is rodd:odd-1, like r3:2
  1843:       Inst.setOpcode(Hexagon::A4_boundscheck_hi);
  1844:       std::string Name = r + utostr(RegNum) + Colon + utostr(RegNum - 1);
  1845:       StringRef RegPair = Name;
  1846:       Rs.setReg(matchRegister(RegPair));
  1847:     } else { // raw:lo
  1848:       Inst.setOpcode(Hexagon::A4_boundscheck_lo);
  1849:       std::string Name = r + utostr(RegNum + 1) + Colon + utostr(RegNum);
  1850:       StringRef RegPair = Name;
  1851:       Rs.setReg(matchRegister(RegPair));
  1852:     }
  1853:     break;
  1854:   }
  1855: 
  1856:   case Hexagon::A2_addsp: {
  1857:     MCOperand &Rs = Inst.getOperand(1);
  1858:     unsigned int RegNum = RI->getEncodingValue(Rs.getReg());
  1859:     if (RegNum & 1) { // Odd mapped to raw:hi
  1860:       Inst.setOpcode(Hexagon::A2_addsph);
  1861:       std::string Name = r + utostr(RegNum) + Colon + utostr(RegNum - 1);
  1862:       StringRef RegPair = Name;
  1863:       Rs.setReg(matchRegister(RegPair));
  1864:     } else { // Even mapped raw:lo
  1865:       Inst.setOpcode(Hexagon::A2_addspl);
  1866:       std::string Name = r + utostr(RegNum + 1) + Colon + utostr(RegNum);
  1867:       StringRef RegPair = Name;
  1868:       Rs.setReg(matchRegister(RegPair));
  1869:     }
  1870:     break;
  1871:   }
  1872: 
  1873:   case Hexagon::M2_vrcmpys_s1: {
  1874:     MCOperand &Rt = Inst.getOperand(2);
  1875:     unsigned int RegNum = RI->getEncodingValue(Rt.getReg());
  1876:     if (RegNum & 1) { // Odd mapped to sat:raw:hi
  1877:       Inst.setOpcode(Hexagon::M2_vrcmpys_s1_h);
  1878:       std::string Name = r + utostr(RegNum) + Colon + utostr(RegNum - 1);
  1879:       StringRef RegPair = Name;
  1880:       Rt.setReg(matchRegister(RegPair));
  1881:     } else { // Even mapped sat:raw:lo
  1882:       Inst.setOpcode(Hexagon::M2_vrcmpys_s1_l);
  1883:       std::string Name = r + utostr(RegNum + 1) + Colon + utostr(RegNum);
  1884:       StringRef RegPair = Name;
  1885:       Rt.setReg(matchRegister(RegPair));
  1886:     }
  1887:     break;
  1888:   }
  1889: 
  1890:   case Hexagon::M2_vrcmpys_acc_s1: {
  1891:     MCInst TmpInst;
  1892:     MCOperand &Rxx = Inst.getOperand(0);
  1893:     MCOperand &Rss = Inst.getOperand(2);
  1894:     MCOperand &Rt = Inst.getOperand(3);
  1895:     unsigned int RegNum = RI->getEncodingValue(Rt.getReg());
  1896:     if (RegNum & 1) { // Odd mapped to sat:raw:hi
  1897:       TmpInst.setOpcode(Hexagon::M2_vrcmpys_acc_s1_h);
  1898:       std::string Name = r + utostr(RegNum) + Colon + utostr(RegNum - 1);
  1899:       StringRef RegPair = Name;
  1900:       Rt.setReg(matchRegister(RegPair));
  1901:     } else { // Even mapped sat:raw:lo
  1902:       TmpInst.setOpcode(Hexagon::M2_vrcmpys_acc_s1_l);
  1903:       std::string Name = r + utostr(RegNum + 1) + Colon + utostr(RegNum);
  1904:       StringRef RegPair = Name;
  1905:       Rt.setReg(matchRegister(RegPair));
  1906:     }
  1907:     // Registers are in different positions
  1908:     TmpInst.addOperand(Rxx);
  1909:     TmpInst.addOperand(Rxx);
  1910:     TmpInst.addOperand(Rss);
  1911:     TmpInst.addOperand(Rt);
  1912:     Inst = TmpInst;
  1913:     break;
  1914:   }
  1915: 
  1916:   case Hexagon::M2_vrcmpys_s1rp: {
  1917:     MCOperand &Rt = Inst.getOperand(2);
  1918:     unsigned int RegNum = RI->getEncodingValue(Rt.getReg());
  1919:     if (RegNum & 1) { // Odd mapped to rnd:sat:raw:hi
  1920:       Inst.setOpcode(Hexagon::M2_vrcmpys_s1rp_h);
  1921:       std::string Name = r + utostr(RegNum) + Colon + utostr(RegNum - 1);
  1922:       StringRef RegPair = Name;
  1923:       Rt.setReg(matchRegister(RegPair));
  1924:     } else { // Even mapped rnd:sat:raw:lo
  1925:       Inst.setOpcode(Hexagon::M2_vrcmpys_s1rp_l);
  1926:       std::string Name = r + utostr(RegNum + 1) + Colon + utostr(RegNum);
  1927:       StringRef RegPair = Name;
  1928:       Rt.setReg(matchRegister(RegPair));
  1929:     }
  1930:     break;
  1931:   }
  1932: 
  1933:   case Hexagon::S5_asrhub_rnd_sat_goodsyntax: {
  1934:     MCOperand &Imm = Inst.getOperand(2);
  1935:     int64_t Value;
  1936:     bool Absolute = Imm.getExpr()->evaluateAsAbsolute(Value);
  1937:     if (!Absolute)
  1938:       return Match_InvalidOperand;
  1939:     if (Value == 0)
  1940:       Inst.setOpcode(Hexagon::S2_vsathub);
  1941:     else {
  1942:       Imm.setExpr(HexagonMCExpr::create(
  1943:           MCBinaryExpr::createSub(Imm.getExpr(),
  1944:                                   MCConstantExpr::create(1, Context), Context),
  1945:           Context));
  1946:       Inst.setOpcode(Hexagon::S5_asrhub_rnd_sat);
  1947:     }
  1948:     break;
  1949:   }
  1950: 
  1951:   case Hexagon::S5_vasrhrnd_goodsyntax: {
  1952:     MCOperand &Rdd = Inst.getOperand(0);
  1953:     MCOperand &Rss = Inst.getOperand(1);
  1954:     MCOperand &Imm = Inst.getOperand(2);
  1955:     int64_t Value;
  1956:     bool Absolute = Imm.getExpr()->evaluateAsAbsolute(Value);
  1957:     if (!Absolute)
  1958:       return Match_InvalidOperand;
  1959:     if (Value == 0) {
  1960:       MCInst TmpInst;
  1961:       unsigned int RegPairNum = RI->getEncodingValue(Rss.getReg());
  1962:       std::string R1 = r + utostr(RegPairNum + 1);
  1963:       StringRef Reg1(R1);
  1964:       Rss.setReg(matchRegister(Reg1));
  1965:       // Add a new operand for the second register in the pair.
  1966:       std::string R2 = r + utostr(RegPairNum);
  1967:       StringRef Reg2(R2);
  1968:       TmpInst.setOpcode(Hexagon::A2_combinew);
  1969:       TmpInst.addOperand(Rdd);
  1970:       TmpInst.addOperand(Rss);
  1971:       TmpInst.addOperand(MCOperand::createReg(matchRegister(Reg2)));
  1972:       Inst = TmpInst;
  1973:     } else {
  1974:       Imm.setExpr(HexagonMCExpr::create(
  1975:           MCBinaryExpr::createSub(Imm.getExpr(),
  1976:                                   MCConstantExpr::create(1, Context), Context),
  1977:           Context));
  1978:       Inst.setOpcode(Hexagon::S5_vasrhrnd);
  1979:     }
  1980:     break;
  1981:   }
  1982: 
  1983:   case Hexagon::A2_not: {
  1984:     MCInst TmpInst;
  1985:     MCOperand &Rd = Inst.getOperand(0);
  1986:     MCOperand &Rs = Inst.getOperand(1);
  1987:     TmpInst.setOpcode(Hexagon::A2_subri);
  1988:     TmpInst.addOperand(Rd);
  1989:     TmpInst.addOperand(MCOperand::createExpr(
  1990:         HexagonMCExpr::create(MCConstantExpr::create(-1, Context), Context)));
  1991:     TmpInst.addOperand(Rs);
  1992:     Inst = TmpInst;
  1993:     break;
  1994:   }
  1995:   case Hexagon::PS_loadrubabs:
  1996:     if (!HexagonMCInstrInfo::mustExtend(*Inst.getOperand(1).getExpr()))
  1997:       Inst.setOpcode(Hexagon::L2_loadrubgp);
  1998:     break;
  1999:   case Hexagon::PS_loadrbabs:
  2000:     if (!HexagonMCInstrInfo::mustExtend(*Inst.getOperand(1).getExpr()))
```
- EN: It declares or implements routines such as addOperand, getOperand, getExpr, getEncodingValue, ... (10 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonMCExpr, HexagonMCInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 addOperand, getOperand, getExpr, getEncodingValue, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonMCExpr, HexagonMCInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 2001-2066 / 第 2001-2066 行

```cpp
  2001:       Inst.setOpcode(Hexagon::L2_loadrbgp);
  2002:     break;
  2003:   case Hexagon::PS_loadruhabs:
  2004:     if (!HexagonMCInstrInfo::mustExtend(*Inst.getOperand(1).getExpr()))
  2005:       Inst.setOpcode(Hexagon::L2_loadruhgp);
  2006:     break;
  2007:   case Hexagon::PS_loadrhabs:
  2008:     if (!HexagonMCInstrInfo::mustExtend(*Inst.getOperand(1).getExpr()))
  2009:       Inst.setOpcode(Hexagon::L2_loadrhgp);
  2010:     break;
  2011:   case Hexagon::PS_loadriabs:
  2012:     if (!HexagonMCInstrInfo::mustExtend(*Inst.getOperand(1).getExpr()))
  2013:       Inst.setOpcode(Hexagon::L2_loadrigp);
  2014:     break;
  2015:   case Hexagon::PS_loadrdabs:
  2016:     if (!HexagonMCInstrInfo::mustExtend(*Inst.getOperand(1).getExpr()))
  2017:       Inst.setOpcode(Hexagon::L2_loadrdgp);
  2018:     break;
  2019:   case Hexagon::PS_storerbabs:
  2020:     if (!HexagonMCInstrInfo::mustExtend(*Inst.getOperand(0).getExpr()))
  2021:       Inst.setOpcode(Hexagon::S2_storerbgp);
  2022:     break;
  2023:   case Hexagon::PS_storerhabs:
  2024:     if (!HexagonMCInstrInfo::mustExtend(*Inst.getOperand(0).getExpr()))
  2025:       Inst.setOpcode(Hexagon::S2_storerhgp);
  2026:     break;
  2027:   case Hexagon::PS_storerfabs:
  2028:     if (!HexagonMCInstrInfo::mustExtend(*Inst.getOperand(0).getExpr()))
  2029:       Inst.setOpcode(Hexagon::S2_storerfgp);
  2030:     break;
  2031:   case Hexagon::PS_storeriabs:
  2032:     if (!HexagonMCInstrInfo::mustExtend(*Inst.getOperand(0).getExpr()))
  2033:       Inst.setOpcode(Hexagon::S2_storerigp);
  2034:     break;
  2035:   case Hexagon::PS_storerdabs:
  2036:     if (!HexagonMCInstrInfo::mustExtend(*Inst.getOperand(0).getExpr()))
  2037:       Inst.setOpcode(Hexagon::S2_storerdgp);
  2038:     break;
  2039:   case Hexagon::PS_storerbnewabs:
  2040:     if (!HexagonMCInstrInfo::mustExtend(*Inst.getOperand(0).getExpr()))
  2041:       Inst.setOpcode(Hexagon::S2_storerbnewgp);
  2042:     break;
  2043:   case Hexagon::PS_storerhnewabs:
  2044:     if (!HexagonMCInstrInfo::mustExtend(*Inst.getOperand(0).getExpr()))
  2045:       Inst.setOpcode(Hexagon::S2_storerhnewgp);
  2046:     break;
  2047:   case Hexagon::PS_storerinewabs:
  2048:     if (!HexagonMCInstrInfo::mustExtend(*Inst.getOperand(0).getExpr()))
  2049:       Inst.setOpcode(Hexagon::S2_storerinewgp);
  2050:     break;
  2051:   case Hexagon::A2_zxtb: {
  2052:     Inst.setOpcode(Hexagon::A2_andir);
  2053:     Inst.addOperand(
  2054:         MCOperand::createExpr(MCConstantExpr::create(255, Context)));
  2055:     break;
  2056:   }
  2057:   } // switch
  2058: 
  2059:   return Match_Success;
  2060: }
  2061: 
  2062: MCRegister HexagonAsmParser::matchRegister(StringRef Name) {
  2063:   if (MCRegister Reg = MatchRegisterName(Name))
  2064:     return Reg;
  2065:   return MatchRegisterAltName(Name);
  2066: }
```
- EN: It declares or implements routines such as setOpcode, addOperand, HexagonAsmParser::matchRegister, MatchRegisterAltName, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonMCInstrInfo, HexagonAsmParser, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 setOpcode, addOperand, HexagonAsmParser::matchRegister, MatchRegisterAltName 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonMCInstrInfo, HexagonAsmParser，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- instruction semantics / 指令语义
- assembly/MC integration / 汇编/MC 集成
- MC-layer target description / MC 层目标描述
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `MCTargetDesc/HexagonMCAsmInfo.h, MCTargetDesc/HexagonMCChecker.h, MCTargetDesc/HexagonMCELFStreamer.h, MCTargetDesc/HexagonMCExpr.h, MCTargetDesc/HexagonMCInstrInfo.h, MCTargetDesc/HexagonMCTargetDesc.h, MCTargetDesc/HexagonShuffler.h, MCTargetDesc/HexagonTargetStreamer.h, TargetInfo/HexagonTargetInfo.h, llvm/ADT/STLExtras.h, ... (52 total)`
- Hexagon symbols / Hexagon 符号: `HexagonAsmParser, HexagonMCAsmInfo, HexagonMCChecker, HexagonMCELFStreamer, HexagonMCExpr, HexagonMCInstrInfo, HexagonMCTargetDesc, HexagonShuffler, HexagonTargetStreamer, HexagonTargetInfo, ... (18 total)`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
