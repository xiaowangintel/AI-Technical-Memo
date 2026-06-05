# HexagonDisassembler.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/Disassembler/HexagonDisassembler.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Decodes Hexagon machine code bytes into LLVM MC instructions.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。 重点涉及指令语义与选择。

## Line-by-Line Analysis / 逐行分析

### Lines 1-100 / 第 1-100 行

```cpp
     1: //===- HexagonDisassembler.cpp - Disassembler for Hexagon ISA -------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: 
     9: #include "MCTargetDesc/HexagonBaseInfo.h"
    10: #include "MCTargetDesc/HexagonMCChecker.h"
    11: #include "MCTargetDesc/HexagonMCInstrInfo.h"
    12: #include "MCTargetDesc/HexagonMCTargetDesc.h"
    13: #include "TargetInfo/HexagonTargetInfo.h"
    14: #include "llvm/ADT/ArrayRef.h"
    15: #include "llvm/MC/MCContext.h"
    16: #include "llvm/MC/MCDecoder.h"
    17: #include "llvm/MC/MCDecoderOps.h"
    18: #include "llvm/MC/MCDisassembler/MCDisassembler.h"
    19: #include "llvm/MC/MCExpr.h"
    20: #include "llvm/MC/MCInst.h"
    21: #include "llvm/MC/MCInstrInfo.h"
    22: #include "llvm/MC/MCRegisterInfo.h"
    23: #include "llvm/MC/MCSubtargetInfo.h"
    24: #include "llvm/MC/TargetRegistry.h"
    25: #include "llvm/Support/Compiler.h"
    26: #include "llvm/Support/Endian.h"
    27: #include "llvm/Support/MathExtras.h"
    28: #include "llvm/Support/raw_ostream.h"
    29: #include <cassert>
    30: #include <cstddef>
    31: #include <cstdint>
    32: #include <memory>
    33: 
    34: #define DEBUG_TYPE "hexagon-disassembler"
    35: 
    36: using namespace llvm;
    37: using namespace llvm::MCD;
    38: using namespace Hexagon;
    39: 
    40: using DecodeStatus = MCDisassembler::DecodeStatus;
    41: 
    42: namespace {
    43: 
    44: /// Hexagon disassembler for all Hexagon platforms.
    45: class HexagonDisassembler : public MCDisassembler {
    46: public:
    47:   std::unique_ptr<MCInstrInfo const> const MCII;
    48:   mutable std::unique_ptr<MCInst> CurrentBundle;
    49:   mutable MCInst const *CurrentExtender;
    50: 
    51:   HexagonDisassembler(const MCSubtargetInfo &STI, MCContext &Ctx,
    52:                       MCInstrInfo const *MCII)
    53:       : MCDisassembler(STI, Ctx), MCII(MCII), CurrentBundle(nullptr),
    54:         CurrentExtender(nullptr) {}
    55: 
    56:   DecodeStatus getSingleInstruction(MCInst &Instr, MCInst &MCB,
    57:                                     ArrayRef<uint8_t> Bytes, uint64_t Address,
    58:                                     raw_ostream &CStream, bool &Complete) const;
    59:   DecodeStatus getInstruction(MCInst &Instr, uint64_t &Size,
    60:                               ArrayRef<uint8_t> Bytes, uint64_t Address,
    61:                               raw_ostream &CStream) const override;
    62: 
    63:   DecodeStatus getInstructionBundle(MCInst &Instr, uint64_t &Size,
    64:                                     ArrayRef<uint8_t> Bytes, uint64_t Address,
    65:                                     raw_ostream &CStream) const override;
    66: 
    67:   void remapInstruction(MCInst &Instr) const;
    68: 
    69:   Expected<bool> onSymbolStart(SymbolInfoTy &Symbol, uint64_t &Size,
    70:                                ArrayRef<uint8_t> Bytes,
    71:                                uint64_t Address) const override;
    72: 
    73: private:
    74:   bool makeBundle(ArrayRef<uint8_t> Bytes, uint64_t Address,
    75:                   uint64_t &BytesToSkip, raw_ostream &CS) const;
    76: 
    77:   void resetBundle() const {
    78:     CurrentBundle.reset();
    79:     CurrentInstruction = nullptr;
    80:   }
    81: 
    82:   mutable MCOperand *CurrentInstruction = nullptr;
    83: };
    84: 
    85: static uint64_t fullValue(HexagonDisassembler const &Disassembler, MCInst &MI,
    86:                           int64_t Value) {
    87:   MCInstrInfo MCII = *Disassembler.MCII;
    88:   if (!Disassembler.CurrentExtender ||
    89:       MI.size() != HexagonMCInstrInfo::getExtendableOp(MCII, MI))
    90:     return Value;
    91:   unsigned Alignment = HexagonMCInstrInfo::getExtentAlignment(MCII, MI);
    92:   uint32_t Lower6 = static_cast<uint32_t>(Value >> Alignment) & 0x3f;
    93:   int64_t Bits;
    94:   bool Success =
    95:       Disassembler.CurrentExtender->getOperand(0).getExpr()->evaluateAsAbsolute(
    96:           Bits);
    97:   assert(Success);
    98:   (void)Success;
    99:   uint64_t Upper26 = static_cast<uint64_t>(Bits);
   100:   uint64_t Operand = Upper26 | Lower6;
```
- EN: It imports headers such as MCTargetDesc/HexagonBaseInfo.h, MCTargetDesc/HexagonMCChecker.h, MCTargetDesc/HexagonMCInstrInfo.h, MCTargetDesc/HexagonMCTargetDesc.h, ... (24 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm, Hexagon) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonDisassembler, which carry the state or API of this component.
- CN: 这里引入了 MCTargetDesc/HexagonBaseInfo.h, MCTargetDesc/HexagonMCChecker.h, MCTargetDesc/HexagonMCInstrInfo.h, MCTargetDesc/HexagonMCTargetDesc.h, ... (24 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm, Hexagon），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonDisassembler 等类型，用来承载该组件的状态或接口。

### Lines 101-200 / 第 101-200 行

```cpp
   101:   return Operand;
   102: }
   103: static HexagonDisassembler const &disassembler(const MCDisassembler *Decoder) {
   104:   return *static_cast<HexagonDisassembler const *>(Decoder);
   105: }
   106: template <size_t T>
   107: static void signedDecoder(MCInst &MI, unsigned tmp,
   108:                           const MCDisassembler *Decoder) {
   109:   HexagonDisassembler const &Disassembler = disassembler(Decoder);
   110:   int64_t FullValue = fullValue(Disassembler, MI, SignExtend64<T>(tmp));
   111:   int64_t Extended = SignExtend64<32>(FullValue);
   112:   HexagonMCInstrInfo::addConstant(MI, Extended, Disassembler.getContext());
   113: }
   114: }
   115: 
   116: // Forward declare these because the auto-generated code will reference them.
   117: // Definitions are further down.
   118: 
   119: static DecodeStatus DecodeIntRegsRegisterClass(MCInst &Inst, unsigned RegNo,
   120:                                                uint64_t Address,
   121:                                                const MCDisassembler *Decoder);
   122: static DecodeStatus
   123: DecodeGeneralSubRegsRegisterClass(MCInst &Inst, unsigned RegNo,
   124:                                   uint64_t Address,
   125:                                   const MCDisassembler *Decoder);
   126: static DecodeStatus
   127: DecodeIntRegsLow8RegisterClass(MCInst &Inst, unsigned RegNo, uint64_t Address,
   128:                                const MCDisassembler *Decoder);
   129: static DecodeStatus DecodeHvxVRRegisterClass(MCInst &Inst, unsigned RegNo,
   130:                                              uint64_t Address,
   131:                                              const MCDisassembler *Decoder);
   132: static DecodeStatus
   133: DecodeDoubleRegsRegisterClass(MCInst &Inst, unsigned RegNo, uint64_t Address,
   134:                               const MCDisassembler *Decoder);
   135: static DecodeStatus
   136: DecodeGeneralDoubleLow8RegsRegisterClass(MCInst &Inst, unsigned RegNo,
   137:                                          uint64_t Address,
   138:                                          const MCDisassembler *Decoder);
   139: static DecodeStatus DecodeHvxWRRegisterClass(MCInst &Inst, unsigned RegNo,
   140:                                              uint64_t Address,
   141:                                              const MCDisassembler *Decoder);
   142: static DecodeStatus DecodeHvxVQRRegisterClass(MCInst &Inst, unsigned RegNo,
   143:                                               uint64_t Address,
   144:                                               const MCDisassembler *Decoder);
   145: static DecodeStatus DecodePredRegsRegisterClass(MCInst &Inst, unsigned RegNo,
   146:                                                 uint64_t Address,
   147:                                                 const MCDisassembler *Decoder);
   148: static DecodeStatus DecodeHvxQRRegisterClass(MCInst &Inst, unsigned RegNo,
   149:                                              uint64_t Address,
   150:                                              const MCDisassembler *Decoder);
   151: static DecodeStatus DecodeCtrRegsRegisterClass(MCInst &Inst, unsigned RegNo,
   152:                                                uint64_t Address,
   153:                                                const MCDisassembler *Decoder);
   154: static DecodeStatus DecodeGuestRegsRegisterClass(MCInst &Inst, unsigned RegNo,
   155:                                                  uint64_t Address,
   156:                                                  const MCDisassembler *Decoder);
   157: static DecodeStatus DecodeSysRegsRegisterClass(MCInst &Inst, unsigned RegNo,
   158:                                                uint64_t Address,
   159:                                                const MCDisassembler *Decoder);
   160: static DecodeStatus DecodeModRegsRegisterClass(MCInst &Inst, unsigned RegNo,
   161:                                                uint64_t Address,
   162:                                                const MCDisassembler *Decoder);
   163: static DecodeStatus DecodeCtrRegs64RegisterClass(MCInst &Inst, unsigned RegNo,
   164:                                                  uint64_t Address,
   165:                                                  const MCDisassembler *Decoder);
   166: static DecodeStatus
   167: DecodeGuestRegs64RegisterClass(MCInst &Inst, unsigned RegNo, uint64_t Address,
   168:                                const MCDisassembler *Decoder);
   169: static DecodeStatus DecodeSysRegs64RegisterClass(MCInst &Inst, unsigned RegNo,
   170:                                                  uint64_t Address,
   171:                                                  const MCDisassembler *Decoder);
   172: 
   173: static DecodeStatus unsignedImmDecoder(MCInst &MI, unsigned tmp,
   174:                                        uint64_t Address,
   175:                                        const MCDisassembler *Decoder);
   176: static DecodeStatus s32_0ImmDecoder(MCInst &MI, unsigned tmp,
   177:                                     uint64_t /*Address*/,
   178:                                     const MCDisassembler *Decoder);
   179: static DecodeStatus brtargetDecoder(MCInst &MI, unsigned tmp, uint64_t Address,
   180:                                     const MCDisassembler *Decoder);
   181: 
   182: static DecodeStatus n1ConstDecoder(MCInst &MI, const MCDisassembler *Decoder) {
   183:   MCContext &Ctx = Decoder->getContext();
   184:   MI.addOperand(MCOperand::createExpr(MCConstantExpr::create(-1, Ctx)));
   185:   return DecodeStatus::Success;
   186: }
   187: 
   188: static DecodeStatus sgp10ConstDecoder(MCInst &MI,
   189:                                       const MCDisassembler *Decoder) {
   190:   MI.addOperand(MCOperand::createReg(Hexagon::SGP1_0));
   191:   return DecodeStatus::Success;
   192: }
   193: 
   194: #include "HexagonDepDecoders.inc"
   195: #include "HexagonGenDisassemblerTables.inc"
   196: 
   197: static MCDisassembler *createHexagonDisassembler(const Target &T,
   198:                                                  const MCSubtargetInfo &STI,
   199:                                                  MCContext &Ctx) {
   200:   return new HexagonDisassembler(STI, Ctx, T.createMCInstrInfo());
```
- EN: It imports headers such as HexagonDepDecoders.inc, HexagonGenDisassemblerTables.inc, establishing the LLVM/Hexagon APIs used below. It declares or implements routines such as disassembler, signedDecoder, fullValue, SignExtend64<32>, ... (31 total), translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonDisassembler, HexagonMCInstrInfo, HexagonDepDecoders, HexagonGenDisassemblerTables, showing how the code connects to sibling backend components. Generated include hooks bridge handwritten logic with TableGen-produced tables in this range.
- CN: 这里引入了 HexagonDepDecoders.inc, HexagonGenDisassemblerTables.inc 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这里声明或实现了 disassembler, signedDecoder, fullValue, SignExtend64<32>, ... (31 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonDisassembler, HexagonMCInstrInfo, HexagonDepDecoders, HexagonGenDisassemblerTables，说明了它与同级后端组件的连接关系。 这里通过生成的 include 钩子把手写逻辑与 TableGen 生成表连接起来。

### Lines 201-300 / 第 201-300 行

```cpp
   201: }
   202: 
   203: extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
   204: LLVMInitializeHexagonDisassembler() {
   205:   TargetRegistry::RegisterMCDisassembler(getTheHexagonTarget(),
   206:                                          createHexagonDisassembler);
   207: }
   208: 
   209: bool HexagonDisassembler::makeBundle(ArrayRef<uint8_t> Bytes, uint64_t Address,
   210:                                      uint64_t &BytesToSkip,
   211:                                      raw_ostream &CS) const {
   212:   bool Complete = false;
   213:   DecodeStatus Result = DecodeStatus::Success;
   214: 
   215:   CurrentBundle.reset(new MCInst);
   216:   CurrentBundle->setOpcode(Hexagon::BUNDLE);
   217:   CurrentBundle->addOperand(MCOperand::createImm(0));
   218:   while (Result == Success && !Complete) {
   219:     if (Bytes.size() < HEXAGON_INSTR_SIZE)
   220:       return false;
   221:     MCInst *Inst = getContext().createMCInst();
   222:     Result = getSingleInstruction(*Inst, *CurrentBundle, Bytes, Address, CS,
   223:                                   Complete);
   224:     CurrentBundle->addOperand(MCOperand::createInst(Inst));
   225:     BytesToSkip += HEXAGON_INSTR_SIZE;
   226:     Bytes = Bytes.slice(HEXAGON_INSTR_SIZE);
   227:   }
   228:   if (Result == MCDisassembler::Fail)
   229:     return false;
   230:   if (BytesToSkip > HEXAGON_MAX_PACKET_SIZE)
   231:     return false;
   232: 
   233:   const auto ArchSTI = Hexagon_MC::getArchSubtarget(&STI);
   234:   const auto STI_ = (ArchSTI != nullptr) ? *ArchSTI : STI;
   235:   HexagonMCChecker Checker(getContext(), *MCII, STI_, *CurrentBundle,
   236:                            *getContext().getRegisterInfo(), false);
   237:   if (!Checker.check())
   238:     return false;
   239:   remapInstruction(*CurrentBundle);
   240:   return true;
   241: }
   242: 
   243: DecodeStatus HexagonDisassembler::getInstruction(MCInst &MI, uint64_t &Size,
   244:                                                  ArrayRef<uint8_t> Bytes,
   245:                                                  uint64_t Address,
   246:                                                  raw_ostream &CS) const {
   247:   CommentStream = &CS;
   248: 
   249:   Size = 0;
   250:   uint64_t BytesToSkip = 0;
   251: 
   252:   if (!CurrentBundle) {
   253:     if (!makeBundle(Bytes, Address, BytesToSkip, CS)) {
   254:       Size = BytesToSkip;
   255:       resetBundle();
   256:       return MCDisassembler::Fail;
   257:     }
   258:     CurrentInstruction = (CurrentBundle->begin() + 1);
   259:   }
   260: 
   261:   MI = *(CurrentInstruction->getInst());
   262:   Size = HEXAGON_INSTR_SIZE;
   263:   if (++CurrentInstruction == CurrentBundle->end())
   264:     resetBundle();
   265:   return MCDisassembler::Success;
   266: }
   267: 
   268: DecodeStatus HexagonDisassembler::getInstructionBundle(MCInst &MI,
   269:                                                        uint64_t &Size,
   270:                                                        ArrayRef<uint8_t> Bytes,
   271:                                                        uint64_t Address,
   272:                                                        raw_ostream &CS) const {
   273:   CommentStream = &CS;
   274:   Size = 0;
   275:   uint64_t BytesToSkip = 0;
   276:   assert(!CurrentBundle);
   277: 
   278:   if (!makeBundle(Bytes, Address, BytesToSkip, CS)) {
   279:     Size = BytesToSkip;
   280:     resetBundle();
   281:     return MCDisassembler::Fail;
   282:   }
   283: 
   284:   MI = *CurrentBundle;
   285:   Size = HEXAGON_INSTR_SIZE * HexagonMCInstrInfo::bundleSize(MI);
   286:   resetBundle();
   287: 
   288:   return Success;
   289: }
   290: 
   291: void HexagonDisassembler::remapInstruction(MCInst &Instr) const {
   292:   for (auto I: HexagonMCInstrInfo::bundleInstructions(Instr)) {
   293:     auto &MI = const_cast<MCInst &>(*I.getInst());
   294:     switch (MI.getOpcode()) {
   295:     case Hexagon::S2_allocframe:
   296:       if (MI.getOperand(0).getReg() == Hexagon::R29) {
   297:         MI.setOpcode(Hexagon::S6_allocframe_to_raw);
   298:         MI.erase(MI.begin () + 1);
   299:         MI.erase(MI.begin ());
   300:       }
```
- EN: It declares or implements routines such as LLVMInitializeHexagonDisassembler, TargetRegistry::RegisterMCDisassembler, HexagonDisassembler::makeBundle, reset, ... (21 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonDisassembler, Hexagon_MC, HexagonMCChecker, HexagonMCInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 LLVMInitializeHexagonDisassembler, TargetRegistry::RegisterMCDisassembler, HexagonDisassembler::makeBundle, reset, ... (21 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonDisassembler, Hexagon_MC, HexagonMCChecker, HexagonMCInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 301-400 / 第 301-400 行

```cpp
   301:       break;
   302:     case Hexagon::L2_deallocframe:
   303:       if (MI.getOperand(0).getReg() == Hexagon::D15 &&
   304:           MI.getOperand(1).getReg() == Hexagon::R30) {
   305:         MI.setOpcode(L6_deallocframe_map_to_raw);
   306:         MI.erase(MI.begin () + 1);
   307:         MI.erase(MI.begin ());
   308:       }
   309:       break;
   310:     case Hexagon::L4_return:
   311:       if (MI.getOperand(0).getReg() == Hexagon::D15 &&
   312:           MI.getOperand(1).getReg() == Hexagon::R30) {
   313:         MI.setOpcode(L6_return_map_to_raw);
   314:         MI.erase(MI.begin () + 1);
   315:         MI.erase(MI.begin ());
   316:       }
   317:       break;
   318:     case Hexagon::L4_return_t:
   319:       if (MI.getOperand(0).getReg() == Hexagon::D15 &&
   320:           MI.getOperand(2).getReg() == Hexagon::R30) {
   321:         MI.setOpcode(L4_return_map_to_raw_t);
   322:         MI.erase(MI.begin () + 2);
   323:         MI.erase(MI.begin ());
   324:       }
   325:       break;
   326:     case Hexagon::L4_return_f:
   327:       if (MI.getOperand(0).getReg() == Hexagon::D15 &&
   328:           MI.getOperand(2).getReg() == Hexagon::R30) {
   329:         MI.setOpcode(L4_return_map_to_raw_f);
   330:         MI.erase(MI.begin () + 2);
   331:         MI.erase(MI.begin ());
   332:       }
   333:       break;
   334:     case Hexagon::L4_return_tnew_pt:
   335:       if (MI.getOperand(0).getReg() == Hexagon::D15 &&
   336:           MI.getOperand(2).getReg() == Hexagon::R30) {
   337:         MI.setOpcode(L4_return_map_to_raw_tnew_pt);
   338:         MI.erase(MI.begin () + 2);
   339:         MI.erase(MI.begin ());
   340:       }
   341:       break;
   342:     case Hexagon::L4_return_fnew_pt:
   343:       if (MI.getOperand(0).getReg() == Hexagon::D15 &&
   344:           MI.getOperand(2).getReg() == Hexagon::R30) {
   345:         MI.setOpcode(L4_return_map_to_raw_fnew_pt);
   346:         MI.erase(MI.begin () + 2);
   347:         MI.erase(MI.begin ());
   348:       }
   349:       break;
   350:     case Hexagon::L4_return_tnew_pnt:
   351:       if (MI.getOperand(0).getReg() == Hexagon::D15 &&
   352:           MI.getOperand(2).getReg() == Hexagon::R30) {
   353:         MI.setOpcode(L4_return_map_to_raw_tnew_pnt);
   354:         MI.erase(MI.begin () + 2);
   355:         MI.erase(MI.begin ());
   356:       }
   357:       break;
   358:     case Hexagon::L4_return_fnew_pnt:
   359:       if (MI.getOperand(0).getReg() == Hexagon::D15 &&
   360:           MI.getOperand(2).getReg() == Hexagon::R30) {
   361:         MI.setOpcode(L4_return_map_to_raw_fnew_pnt);
   362:         MI.erase(MI.begin () + 2);
   363:         MI.erase(MI.begin ());
   364:       }
   365:       break;
   366:     }
   367:   }
   368: }
   369: 
   370: DecodeStatus HexagonDisassembler::getSingleInstruction(MCInst &MI, MCInst &MCB,
   371:                                                        ArrayRef<uint8_t> Bytes,
   372:                                                        uint64_t Address,
   373:                                                        raw_ostream &cs,
   374:                                                        bool &Complete) const {
   375:   assert(Bytes.size() >= HEXAGON_INSTR_SIZE);
   376: 
   377:   uint32_t Instruction = support::endian::read32le(Bytes.data());
   378: 
   379:   auto BundleSize = HexagonMCInstrInfo::bundleSize(MCB);
   380:   if ((Instruction & HexagonII::INST_PARSE_MASK) ==
   381:       HexagonII::INST_PARSE_LOOP_END) {
   382:     if (BundleSize == 0)
   383:       HexagonMCInstrInfo::setInnerLoop(MCB);
   384:     else if (BundleSize == 1)
   385:       HexagonMCInstrInfo::setOuterLoop(MCB);
   386:     else
   387:       return DecodeStatus::Fail;
   388:   }
   389: 
   390:   CurrentExtender = HexagonMCInstrInfo::extenderForIndex(
   391:       MCB, HexagonMCInstrInfo::bundleSize(MCB));
   392: 
   393:   DecodeStatus Result = DecodeStatus::Fail;
   394:   if ((Instruction & HexagonII::INST_PARSE_MASK) ==
   395:       HexagonII::INST_PARSE_DUPLEX) {
   396:     unsigned duplexIClass;
   397:     uint8_t const *DecodeLow, *DecodeHigh;
   398:     duplexIClass = ((Instruction >> 28) & 0xe) | ((Instruction >> 13) & 0x1);
   399:     switch (duplexIClass) {
   400:     default:
```
- EN: It declares or implements routines such as setOpcode, erase, HexagonDisassembler::getSingleInstruction, assert, ... (7 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonDisassembler, HexagonMCInstrInfo, HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 setOpcode, erase, HexagonDisassembler::getSingleInstruction, assert, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonDisassembler, HexagonMCInstrInfo, HexagonII，说明了它与同级后端组件的连接关系。

### Lines 401-500 / 第 401-500 行

```cpp
   401:       return MCDisassembler::Fail;
   402:     case 0:
   403:       DecodeLow = DecoderTableSUBINSN_L132;
   404:       DecodeHigh = DecoderTableSUBINSN_L132;
   405:       break;
   406:     case 1:
   407:       DecodeLow = DecoderTableSUBINSN_L232;
   408:       DecodeHigh = DecoderTableSUBINSN_L132;
   409:       break;
   410:     case 2:
   411:       DecodeLow = DecoderTableSUBINSN_L232;
   412:       DecodeHigh = DecoderTableSUBINSN_L232;
   413:       break;
   414:     case 3:
   415:       DecodeLow = DecoderTableSUBINSN_A32;
   416:       DecodeHigh = DecoderTableSUBINSN_A32;
   417:       break;
   418:     case 4:
   419:       DecodeLow = DecoderTableSUBINSN_L132;
   420:       DecodeHigh = DecoderTableSUBINSN_A32;
   421:       break;
   422:     case 5:
   423:       DecodeLow = DecoderTableSUBINSN_L232;
   424:       DecodeHigh = DecoderTableSUBINSN_A32;
   425:       break;
   426:     case 6:
   427:       DecodeLow = DecoderTableSUBINSN_S132;
   428:       DecodeHigh = DecoderTableSUBINSN_A32;
   429:       break;
   430:     case 7:
   431:       DecodeLow = DecoderTableSUBINSN_S232;
   432:       DecodeHigh = DecoderTableSUBINSN_A32;
   433:       break;
   434:     case 8:
   435:       DecodeLow = DecoderTableSUBINSN_S132;
   436:       DecodeHigh = DecoderTableSUBINSN_L132;
   437:       break;
   438:     case 9:
   439:       DecodeLow = DecoderTableSUBINSN_S132;
   440:       DecodeHigh = DecoderTableSUBINSN_L232;
   441:       break;
   442:     case 10:
   443:       DecodeLow = DecoderTableSUBINSN_S132;
   444:       DecodeHigh = DecoderTableSUBINSN_S132;
   445:       break;
   446:     case 11:
   447:       DecodeLow = DecoderTableSUBINSN_S232;
   448:       DecodeHigh = DecoderTableSUBINSN_S132;
   449:       break;
   450:     case 12:
   451:       DecodeLow = DecoderTableSUBINSN_S232;
   452:       DecodeHigh = DecoderTableSUBINSN_L132;
   453:       break;
   454:     case 13:
   455:       DecodeLow = DecoderTableSUBINSN_S232;
   456:       DecodeHigh = DecoderTableSUBINSN_L232;
   457:       break;
   458:     case 14:
   459:       DecodeLow = DecoderTableSUBINSN_S232;
   460:       DecodeHigh = DecoderTableSUBINSN_S232;
   461:       break;
   462:     }
   463:     MI.setOpcode(Hexagon::DuplexIClass0 + duplexIClass);
   464:     MCInst *MILow = getContext().createMCInst();
   465:     MCInst *MIHigh = getContext().createMCInst();
   466:     auto TmpExtender = CurrentExtender;
   467:     CurrentExtender =
   468:         nullptr; // constant extenders in duplex must always be in slot 1
   469:     Result = decodeInstruction(DecodeLow, *MILow, Instruction & 0x1fff, Address,
   470:                                this, STI);
   471:     CurrentExtender = TmpExtender;
   472:     if (Result != DecodeStatus::Success)
   473:       return DecodeStatus::Fail;
   474:     Result = decodeInstruction(
   475:         DecodeHigh, *MIHigh, (Instruction >> 16) & 0x1fff, Address, this, STI);
   476:     if (Result != DecodeStatus::Success)
   477:       return DecodeStatus::Fail;
   478:     MCOperand OPLow = MCOperand::createInst(MILow);
   479:     MCOperand OPHigh = MCOperand::createInst(MIHigh);
   480:     MI.addOperand(OPLow);
   481:     MI.addOperand(OPHigh);
   482:     Complete = true;
   483:   } else {
   484:     if ((Instruction & HexagonII::INST_PARSE_MASK) ==
   485:         HexagonII::INST_PARSE_PACKET_END)
   486:       Complete = true;
   487: 
   488:     if (CurrentExtender != nullptr)
   489:       Result = decodeInstruction(DecoderTableMustExtend32, MI, Instruction,
   490:                                  Address, this, STI);
   491: 
   492:     if (Result != MCDisassembler::Success)
   493:       Result = decodeInstruction(DecoderTable32, MI, Instruction, Address, this,
   494:                                  STI);
   495: 
   496:     if (Result != MCDisassembler::Success &&
   497:         STI.hasFeature(Hexagon::ExtensionHVX))
   498:       Result = decodeInstruction(DecoderTableEXT_mmvec32, MI, Instruction,
   499:                                  Address, this, STI);
   500: 
```
- EN: It declares or implements routines such as setOpcode, getContext, decodeInstruction, MCOperand::createInst, ... (5 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 setOpcode, getContext, decodeInstruction, MCOperand::createInst, ... (5 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonII，说明了它与同级后端组件的连接关系。

### Lines 501-600 / 第 501-600 行

```cpp
   501:   }
   502: 
   503:   if (HexagonMCInstrInfo::isNewValue(*MCII, MI)) {
   504:     unsigned OpIndex = HexagonMCInstrInfo::getNewValueOp(*MCII, MI);
   505:     MCOperand &MCO = MI.getOperand(OpIndex);
   506:     assert(MCO.isReg() && "New value consumers must be registers");
   507:     unsigned Register =
   508:         getContext().getRegisterInfo()->getEncodingValue(MCO.getReg());
   509:     if ((Register & 0x6) == 0)
   510:       // HexagonPRM 10.11 Bit 1-2 == 0 is reserved
   511:       return MCDisassembler::Fail;
   512:     unsigned Lookback = (Register & 0x6) >> 1;
   513:     unsigned Offset = 1;
   514:     bool Vector = HexagonMCInstrInfo::isVector(*MCII, MI);
   515:     bool PrevVector = false;
   516:     auto Instructions = HexagonMCInstrInfo::bundleInstructions(*CurrentBundle);
   517:     auto i = Instructions.end() - 1;
   518:     for (auto n = Instructions.begin() - 1;; --i, ++Offset) {
   519:       if (i == n)
   520:         // Couldn't find producer
   521:         return MCDisassembler::Fail;
   522:       bool CurrentVector = HexagonMCInstrInfo::isVector(*MCII, *i->getInst());
   523:       if (Vector && !CurrentVector)
   524:         // Skip scalars when calculating distances for vectors
   525:         ++Lookback;
   526:       if (HexagonMCInstrInfo::isImmext(*i->getInst()) && (Vector == PrevVector))
   527:         ++Lookback;
   528:       PrevVector = CurrentVector;
   529:       if (Offset == Lookback)
   530:         break;
   531:     }
   532:     auto const &Inst = *i->getInst();
   533:     bool SubregBit = (Register & 0x1) != 0;
   534:     if (HexagonMCInstrInfo::hasNewValue2(*MCII, Inst)) {
   535:       // If subreg bit is set we're selecting the second produced newvalue
   536:       MCRegister Producer =
   537:           SubregBit
   538:               ? HexagonMCInstrInfo::getNewValueOperand(*MCII, Inst).getReg()
   539:               : HexagonMCInstrInfo::getNewValueOperand2(*MCII, Inst).getReg();
   540:       assert(Producer != Hexagon::NoRegister);
   541:       MCO.setReg(Producer);
   542:     } else if (HexagonMCInstrInfo::hasNewValue(*MCII, Inst)) {
   543:       MCRegister Producer =
   544:           HexagonMCInstrInfo::getNewValueOperand(*MCII, Inst).getReg();
   545: 
   546:       if (HexagonMCInstrInfo::IsVecRegPair(Producer)) {
   547:         const bool Rev = HexagonMCInstrInfo::IsReverseVecRegPair(Producer);
   548:         const unsigned ProdPairIndex =
   549:             Rev ? Producer - Hexagon::WR0 : Producer - Hexagon::W0;
   550:         if (Rev)
   551:           SubregBit = !SubregBit;
   552:         Producer = (ProdPairIndex << 1) + SubregBit + Hexagon::V0;
   553:       } else if (SubregBit)
   554:         // Hexagon PRM 10.11 New-value operands
   555:         // Nt[0] is reserved and should always be encoded as zero.
   556:         return MCDisassembler::Fail;
   557:       assert(Producer != Hexagon::NoRegister);
   558:       MCO.setReg(Producer);
   559:     } else
   560:       return MCDisassembler::Fail;
   561:   }
   562: 
   563:   if (CurrentExtender != nullptr) {
   564:     MCInst const &Inst = HexagonMCInstrInfo::isDuplex(*MCII, MI)
   565:                              ? *MI.getOperand(1).getInst()
   566:                              : MI;
   567:     if (!HexagonMCInstrInfo::isExtendable(*MCII, Inst) &&
   568:         !HexagonMCInstrInfo::isExtended(*MCII, Inst))
   569:       return MCDisassembler::Fail;
   570:   }
   571:   return Result;
   572: }
   573: 
   574: Expected<bool> HexagonDisassembler::onSymbolStart(SymbolInfoTy &Symbol,
   575:                                                   uint64_t &Size,
   576:                                                   ArrayRef<uint8_t> Bytes,
   577:                                                   uint64_t Address) const {
   578:   // At the start of a symbol, force a fresh packet by resetting any
   579:   // in-progress bundle state. This prevents packets from straddling label
   580:   // boundaries when data (e.g. jump tables) appears in between.
   581:   Size = 0;
   582:   resetBundle();
   583:   return true;
   584: }
   585: 
   586: static DecodeStatus DecodeRegisterClass(MCInst &Inst, unsigned RegNo,
   587:                                         ArrayRef<MCPhysReg> Table) {
   588:   if (RegNo < Table.size()) {
   589:     Inst.addOperand(MCOperand::createReg(Table[RegNo]));
   590:     return MCDisassembler::Success;
   591:   }
   592: 
   593:   return MCDisassembler::Fail;
   594: }
   595: 
   596: static DecodeStatus
   597: DecodeIntRegsLow8RegisterClass(MCInst &Inst, unsigned RegNo, uint64_t Address,
   598:                                const MCDisassembler *Decoder) {
   599:   return DecodeIntRegsRegisterClass(Inst, RegNo, Address, Decoder);
   600: }
```
- EN: It declares or implements routines such as HexagonMCInstrInfo::getNewValueOp, getOperand, assert, getContext, ... (16 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonMCInstrInfo, HexagonPRM, HexagonDisassembler, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonMCInstrInfo::getNewValueOp, getOperand, assert, getContext, ... (16 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonMCInstrInfo, HexagonPRM, HexagonDisassembler，说明了它与同级后端组件的连接关系。

### Lines 601-700 / 第 601-700 行

```cpp
   601: 
   602: static DecodeStatus DecodeIntRegsRegisterClass(MCInst &Inst, unsigned RegNo,
   603:                                                uint64_t Address,
   604:                                                const MCDisassembler *Decoder) {
   605:   static const MCPhysReg IntRegDecoderTable[] = {
   606:       Hexagon::R0,  Hexagon::R1,  Hexagon::R2,  Hexagon::R3,  Hexagon::R4,
   607:       Hexagon::R5,  Hexagon::R6,  Hexagon::R7,  Hexagon::R8,  Hexagon::R9,
   608:       Hexagon::R10, Hexagon::R11, Hexagon::R12, Hexagon::R13, Hexagon::R14,
   609:       Hexagon::R15, Hexagon::R16, Hexagon::R17, Hexagon::R18, Hexagon::R19,
   610:       Hexagon::R20, Hexagon::R21, Hexagon::R22, Hexagon::R23, Hexagon::R24,
   611:       Hexagon::R25, Hexagon::R26, Hexagon::R27, Hexagon::R28, Hexagon::R29,
   612:       Hexagon::R30, Hexagon::R31};
   613: 
   614:   return DecodeRegisterClass(Inst, RegNo, IntRegDecoderTable);
   615: }
   616: 
   617: static DecodeStatus
   618: DecodeGeneralSubRegsRegisterClass(MCInst &Inst, unsigned RegNo,
   619:                                   uint64_t Address,
   620:                                   const MCDisassembler *Decoder) {
   621:   static const MCPhysReg GeneralSubRegDecoderTable[] = {
   622:       Hexagon::R0,  Hexagon::R1,  Hexagon::R2,  Hexagon::R3,
   623:       Hexagon::R4,  Hexagon::R5,  Hexagon::R6,  Hexagon::R7,
   624:       Hexagon::R16, Hexagon::R17, Hexagon::R18, Hexagon::R19,
   625:       Hexagon::R20, Hexagon::R21, Hexagon::R22, Hexagon::R23,
   626:   };
   627: 
   628:   return DecodeRegisterClass(Inst, RegNo, GeneralSubRegDecoderTable);
   629: }
   630: 
   631: static DecodeStatus DecodeHvxVRRegisterClass(MCInst &Inst, unsigned RegNo,
   632:                                              uint64_t /*Address*/,
   633:                                              const MCDisassembler *Decoder) {
   634:   static const MCPhysReg HvxVRDecoderTable[] = {
   635:       Hexagon::V0,  Hexagon::V1,  Hexagon::V2,  Hexagon::V3,  Hexagon::V4,
   636:       Hexagon::V5,  Hexagon::V6,  Hexagon::V7,  Hexagon::V8,  Hexagon::V9,
   637:       Hexagon::V10, Hexagon::V11, Hexagon::V12, Hexagon::V13, Hexagon::V14,
   638:       Hexagon::V15, Hexagon::V16, Hexagon::V17, Hexagon::V18, Hexagon::V19,
   639:       Hexagon::V20, Hexagon::V21, Hexagon::V22, Hexagon::V23, Hexagon::V24,
   640:       Hexagon::V25, Hexagon::V26, Hexagon::V27, Hexagon::V28, Hexagon::V29,
   641:       Hexagon::V30, Hexagon::V31};
   642: 
   643:   return DecodeRegisterClass(Inst, RegNo, HvxVRDecoderTable);
   644: }
   645: 
   646: static DecodeStatus
   647: DecodeDoubleRegsRegisterClass(MCInst &Inst, unsigned RegNo,
   648:                               uint64_t /*Address*/,
   649:                               const MCDisassembler *Decoder) {
   650:   static const MCPhysReg DoubleRegDecoderTable[] = {
   651:       Hexagon::D0,  Hexagon::D1,  Hexagon::D2,  Hexagon::D3,
   652:       Hexagon::D4,  Hexagon::D5,  Hexagon::D6,  Hexagon::D7,
   653:       Hexagon::D8,  Hexagon::D9,  Hexagon::D10, Hexagon::D11,
   654:       Hexagon::D12, Hexagon::D13, Hexagon::D14, Hexagon::D15};
   655: 
   656:   return DecodeRegisterClass(Inst, RegNo >> 1, DoubleRegDecoderTable);
   657: }
   658: 
   659: static DecodeStatus
   660: DecodeGeneralDoubleLow8RegsRegisterClass(MCInst &Inst, unsigned RegNo,
   661:                                          uint64_t /*Address*/,
   662:                                          const MCDisassembler *Decoder) {
   663:   static const MCPhysReg GeneralDoubleLow8RegDecoderTable[] = {
   664:       Hexagon::D0, Hexagon::D1, Hexagon::D2,  Hexagon::D3,
   665:       Hexagon::D8, Hexagon::D9, Hexagon::D10, Hexagon::D11};
   666: 
   667:   return DecodeRegisterClass(Inst, RegNo, GeneralDoubleLow8RegDecoderTable);
   668: }
   669: 
   670: static DecodeStatus DecodeHvxWRRegisterClass(MCInst &Inst, unsigned RegNo,
   671:                                              uint64_t /*Address*/,
   672:                                              const MCDisassembler *Decoder) {
   673:   static const MCPhysReg HvxWRDecoderTable[] = {
   674:       Hexagon::W0,   Hexagon::WR0,  Hexagon::W1,   Hexagon::WR1,  Hexagon::W2,
   675:       Hexagon::WR2,  Hexagon::W3,   Hexagon::WR3,  Hexagon::W4,   Hexagon::WR4,
   676:       Hexagon::W5,   Hexagon::WR5,  Hexagon::W6,   Hexagon::WR6,  Hexagon::W7,
   677:       Hexagon::WR7,  Hexagon::W8,   Hexagon::WR8,  Hexagon::W9,   Hexagon::WR9,
   678:       Hexagon::W10,  Hexagon::WR10, Hexagon::W11,  Hexagon::WR11, Hexagon::W12,
   679:       Hexagon::WR12, Hexagon::W13,  Hexagon::WR13, Hexagon::W14,  Hexagon::WR14,
   680:       Hexagon::W15,  Hexagon::WR15,
   681:   };
   682: 
   683:   return DecodeRegisterClass(Inst, RegNo, HvxWRDecoderTable);
   684: }
   685: 
   686: [[maybe_unused]] // Suppress warning temporarily.
   687: static DecodeStatus DecodeHvxVQRRegisterClass(MCInst &Inst, unsigned RegNo,
   688:                                               uint64_t /*Address*/,
   689:                                               const MCDisassembler *Decoder) {
   690:   static const MCPhysReg HvxVQRDecoderTable[] = {
   691:       Hexagon::VQ0,  Hexagon::VQ1,  Hexagon::VQ2,  Hexagon::VQ3,
   692:       Hexagon::VQ4,  Hexagon::VQ5,  Hexagon::VQ6,  Hexagon::VQ7};
   693: 
   694:   return DecodeRegisterClass(Inst, RegNo >> 2, HvxVQRDecoderTable);
   695: }
   696: 
   697: static DecodeStatus DecodePredRegsRegisterClass(MCInst &Inst, unsigned RegNo,
   698:                                                 uint64_t /*Address*/,
   699:                                                 const MCDisassembler *Decoder) {
   700:   static const MCPhysReg PredRegDecoderTable[] = {Hexagon::P0, Hexagon::P1,
```
- EN: It declares or implements routines such as DecodeIntRegsRegisterClass, DecodeRegisterClass, DecodeGeneralSubRegsRegisterClass, DecodeHvxVRRegisterClass, ... (9 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里声明或实现了 DecodeIntRegsRegisterClass, DecodeRegisterClass, DecodeGeneralSubRegsRegisterClass, DecodeHvxVRRegisterClass, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 701-800 / 第 701-800 行

```cpp
   701:                                                   Hexagon::P2, Hexagon::P3};
   702: 
   703:   return DecodeRegisterClass(Inst, RegNo, PredRegDecoderTable);
   704: }
   705: 
   706: static DecodeStatus DecodeHvxQRRegisterClass(MCInst &Inst, unsigned RegNo,
   707:                                              uint64_t /*Address*/,
   708:                                              const MCDisassembler *Decoder) {
   709:   static const MCPhysReg HvxQRDecoderTable[] = {Hexagon::Q0, Hexagon::Q1,
   710:                                                 Hexagon::Q2, Hexagon::Q3};
   711: 
   712:   return DecodeRegisterClass(Inst, RegNo, HvxQRDecoderTable);
   713: }
   714: 
   715: static DecodeStatus DecodeCtrRegsRegisterClass(MCInst &Inst, unsigned RegNo,
   716:                                                uint64_t /*Address*/,
   717:                                                const MCDisassembler *Decoder) {
   718:   using namespace Hexagon;
   719: 
   720:   static const MCPhysReg CtrlRegDecoderTable[] = {
   721:     /*  0 */  SA0,        LC0,        SA1,        LC1,
   722:     /*  4 */  P3_0,       C5,         M0,         M1,
   723:     /*  8 */  USR,        PC,         UGP,        GP,
   724:     /* 12 */  CS0,        CS1,        UPCYCLELO,  UPCYCLEHI,
   725:     /* 16 */  FRAMELIMIT, FRAMEKEY,   PKTCOUNTLO, PKTCOUNTHI,
   726:     /* 20 */  0,          0,          0,          0,
   727:     /* 24 */  0,          0,          0,          0,
   728:     /* 28 */  0,          0,          UTIMERLO,   UTIMERHI
   729:   };
   730: 
   731:   if (RegNo >= std::size(CtrlRegDecoderTable))
   732:     return MCDisassembler::Fail;
   733: 
   734:   static_assert(NoRegister == 0, "Expecting NoRegister to be 0");
   735:   if (CtrlRegDecoderTable[RegNo] == NoRegister)
   736:     return MCDisassembler::Fail;
   737: 
   738:   unsigned Register = CtrlRegDecoderTable[RegNo];
   739:   Inst.addOperand(MCOperand::createReg(Register));
   740:   return MCDisassembler::Success;
   741: }
   742: 
   743: static DecodeStatus
   744: DecodeCtrRegs64RegisterClass(MCInst &Inst, unsigned RegNo, uint64_t /*Address*/,
   745:                              const MCDisassembler *Decoder) {
   746:   using namespace Hexagon;
   747: 
   748:   static const MCPhysReg CtrlReg64DecoderTable[] = {
   749:     /*  0 */  C1_0,       0,          C3_2,       0,
   750:     /*  4 */  C5_4,       0,          C7_6,       0,
   751:     /*  8 */  C9_8,       0,          C11_10,     0,
   752:     /* 12 */  CS,         0,          UPCYCLE,    0,
   753:     /* 16 */  C17_16,     0,          PKTCOUNT,   0,
   754:     /* 20 */  0,          0,          0,          0,
   755:     /* 24 */  0,          0,          0,          0,
   756:     /* 28 */  0,          0,          UTIMER,     0
   757:   };
   758: 
   759:   if (RegNo >= std::size(CtrlReg64DecoderTable))
   760:     return MCDisassembler::Fail;
   761: 
   762:   static_assert(NoRegister == 0, "Expecting NoRegister to be 0");
   763:   if (CtrlReg64DecoderTable[RegNo] == NoRegister)
   764:     return MCDisassembler::Fail;
   765: 
   766:   unsigned Register = CtrlReg64DecoderTable[RegNo];
   767:   Inst.addOperand(MCOperand::createReg(Register));
   768:   return MCDisassembler::Success;
   769: }
   770: 
   771: static DecodeStatus DecodeModRegsRegisterClass(MCInst &Inst, unsigned RegNo,
   772:                                                uint64_t /*Address*/,
   773:                                                const MCDisassembler *Decoder) {
   774:   unsigned Register = 0;
   775:   switch (RegNo) {
   776:   case 0:
   777:     Register = Hexagon::M0;
   778:     break;
   779:   case 1:
   780:     Register = Hexagon::M1;
   781:     break;
   782:   default:
   783:     return MCDisassembler::Fail;
   784:   }
   785:   Inst.addOperand(MCOperand::createReg(Register));
   786:   return MCDisassembler::Success;
   787: }
   788: 
   789: static DecodeStatus unsignedImmDecoder(MCInst &MI, unsigned tmp,
   790:                                        uint64_t /*Address*/,
   791:                                        const MCDisassembler *Decoder) {
   792:   HexagonDisassembler const &Disassembler = disassembler(Decoder);
   793:   int64_t FullValue = fullValue(Disassembler, MI, tmp);
   794:   assert(FullValue >= 0 && "Negative in unsigned decoder");
   795:   HexagonMCInstrInfo::addConstant(MI, FullValue, Disassembler.getContext());
   796:   return MCDisassembler::Success;
   797: }
   798: 
   799: static DecodeStatus s32_0ImmDecoder(MCInst &MI, unsigned tmp,
   800:                                     uint64_t /*Address*/,
```
- EN: It opens namespaces (Hexagon) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as DecodeRegisterClass, DecodeHvxQRRegisterClass, DecodeCtrRegsRegisterClass, static_assert, ... (12 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里打开了命名空间（Hexagon），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 DecodeRegisterClass, DecodeHvxQRRegisterClass, DecodeCtrRegsRegisterClass, static_assert, ... (12 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 801-900 / 第 801-900 行

```cpp
   801:                                     const MCDisassembler *Decoder) {
   802:   HexagonDisassembler const &Disassembler = disassembler(Decoder);
   803:   unsigned Bits = HexagonMCInstrInfo::getExtentBits(*Disassembler.MCII, MI);
   804:   tmp = SignExtend64(tmp, Bits);
   805:   signedDecoder<32>(MI, tmp, Decoder);
   806:   return MCDisassembler::Success;
   807: }
   808: 
   809: // custom decoder for various jump/call immediates
   810: static DecodeStatus brtargetDecoder(MCInst &MI, unsigned tmp, uint64_t Address,
   811:                                     const MCDisassembler *Decoder) {
   812:   HexagonDisassembler const &Disassembler = disassembler(Decoder);
   813:   unsigned Bits = HexagonMCInstrInfo::getExtentBits(*Disassembler.MCII, MI);
   814:   // r13_2 is not extendable, so if there are no extent bits, it's r13_2
   815:   if (Bits == 0)
   816:     Bits = 15;
   817:   uint64_t FullValue = fullValue(Disassembler, MI, SignExtend64(tmp, Bits));
   818:   uint32_t Extended = FullValue + Address;
   819:   if (!Disassembler.tryAddingSymbolicOperand(MI, Extended, Address, true, 0, 0,
   820:                                              4))
   821:     HexagonMCInstrInfo::addConstant(MI, Extended, Disassembler.getContext());
   822:   return MCDisassembler::Success;
   823: }
   824: 
   825: static const uint16_t SysRegDecoderTable[] = {
   826:     Hexagon::SGP0,       Hexagon::SGP1,      Hexagon::STID,
   827:     Hexagon::ELR,        Hexagon::BADVA0,    Hexagon::BADVA1,
   828:     Hexagon::SSR,        Hexagon::CCR,       Hexagon::HTID,
   829:     Hexagon::BADVA,      Hexagon::IMASK,     Hexagon::S11,
   830:     Hexagon::S12,        Hexagon::S13,       Hexagon::S14,
   831:     Hexagon::S15,        Hexagon::EVB,       Hexagon::MODECTL,
   832:     Hexagon::SYSCFG,     Hexagon::S19,       Hexagon::S20,
   833:     Hexagon::VID,        Hexagon::S22,       Hexagon::S23,
   834:     Hexagon::S24,        Hexagon::S25,       Hexagon::S26,
   835:     Hexagon::CFGBASE,    Hexagon::DIAG,      Hexagon::REV,
   836:     Hexagon::PCYCLELO,   Hexagon::PCYCLEHI,  Hexagon::ISDBST,
   837:     Hexagon::ISDBCFG0,   Hexagon::ISDBCFG1,  Hexagon::S35,
   838:     Hexagon::BRKPTPC0,   Hexagon::BRKPTCFG0, Hexagon::BRKPTPC1,
   839:     Hexagon::BRKPTCFG1,  Hexagon::ISDBMBXIN, Hexagon::ISDBMBXOUT,
   840:     Hexagon::ISDBEN,     Hexagon::ISDBGPR,   Hexagon::S44,
   841:     Hexagon::S45,        Hexagon::S46,       Hexagon::S47,
   842:     Hexagon::PMUCNT0,    Hexagon::PMUCNT1,   Hexagon::PMUCNT2,
   843:     Hexagon::PMUCNT3,    Hexagon::PMUEVTCFG, Hexagon::PMUCFG,
   844:     Hexagon::S54,        Hexagon::S55,       Hexagon::S56,
   845:     Hexagon::S57,        Hexagon::S58,       Hexagon::S59,
   846:     Hexagon::S60,        Hexagon::S61,       Hexagon::S62,
   847:     Hexagon::S63,        Hexagon::S64,       Hexagon::S65,
   848:     Hexagon::S66,        Hexagon::S67,       Hexagon::S68,
   849:     Hexagon::S69,        Hexagon::S70,       Hexagon::S71,
   850:     Hexagon::S72,        Hexagon::S73,       Hexagon::S74,
   851:     Hexagon::S75,        Hexagon::S76,       Hexagon::S77,
   852:     Hexagon::S78,        Hexagon::S79,       Hexagon::S80,
   853: };
   854: 
   855: static DecodeStatus DecodeSysRegsRegisterClass(MCInst &Inst, unsigned RegNo,
   856:                                                uint64_t /*Address*/,
   857:                                                const MCDisassembler *Decoder) {
   858:   if (RegNo >= std::size(SysRegDecoderTable))
   859:     return MCDisassembler::Fail;
   860: 
   861:   if (SysRegDecoderTable[RegNo] == Hexagon::NoRegister)
   862:     return MCDisassembler::Fail;
   863: 
   864:   unsigned Register = SysRegDecoderTable[RegNo];
   865:   Inst.addOperand(MCOperand::createReg(Register));
   866:   return MCDisassembler::Success;
   867: }
   868: 
   869: static const uint16_t SysReg64DecoderTable[] = {
   870:     Hexagon::SGP1_0, Hexagon::S3_2,   Hexagon::S5_4,   Hexagon::S7_6,
   871:     Hexagon::S9_8,   Hexagon::S11_10, Hexagon::S13_12, Hexagon::S15_14,
   872:     Hexagon::S17_16, Hexagon::S19_18, Hexagon::S21_20, Hexagon::S23_22,
   873:     Hexagon::S25_24, Hexagon::S27_26, Hexagon::S29_28, Hexagon::S31_30,
   874:     Hexagon::S33_32, Hexagon::S35_34, Hexagon::S37_36, Hexagon::S39_38,
   875:     Hexagon::S41_40, Hexagon::S43_42, Hexagon::S45_44, Hexagon::S47_46,
   876:     Hexagon::S49_48, Hexagon::S51_50, Hexagon::S53_52, Hexagon::S55_54,
   877:     Hexagon::S57_56, Hexagon::S59_58, Hexagon::S61_60, Hexagon::S63_62,
   878:     Hexagon::S65_64, Hexagon::S67_66, Hexagon::S69_68, Hexagon::S71_70,
   879:     Hexagon::S73_72, Hexagon::S75_74, Hexagon::S77_76, Hexagon::S79_78,
   880: };
   881: 
   882: static DecodeStatus
   883: DecodeSysRegs64RegisterClass(MCInst &Inst, unsigned RegNo, uint64_t /*Address*/,
   884:                              const MCDisassembler *Decoder) {
   885:   RegNo = RegNo >> 1;
   886:   if (RegNo >= std::size(SysReg64DecoderTable))
   887:     return MCDisassembler::Fail;
   888: 
   889:   if (SysReg64DecoderTable[RegNo] == Hexagon::NoRegister)
   890:     return MCDisassembler::Fail;
   891: 
   892:   unsigned Register = SysReg64DecoderTable[RegNo];
   893:   Inst.addOperand(MCOperand::createReg(Register));
   894:   return MCDisassembler::Success;
   895: }
   896: 
   897: static DecodeStatus
   898: DecodeGuestRegsRegisterClass(MCInst &Inst, unsigned RegNo, uint64_t /*Address*/,
   899:                              const MCDisassembler *Decoder) {
   900:   using namespace Hexagon;
```
- EN: It opens namespaces (Hexagon) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as disassembler, HexagonMCInstrInfo::getExtentBits, SignExtend64, signedDecoder<32>, ... (10 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonDisassembler, HexagonMCInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里打开了命名空间（Hexagon），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 disassembler, HexagonMCInstrInfo::getExtentBits, SignExtend64, signedDecoder<32>, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonDisassembler, HexagonMCInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 901-948 / 第 901-948 行

```cpp
   901: 
   902:   static const MCPhysReg GuestRegDecoderTable[] = {
   903:     /*  0 */ GELR,      GSR,        GOSP,       G3,
   904:     /*  4 */ G4,        G5,         G6,         G7,
   905:     /*  8 */ G8,        G9,         G10,        G11,
   906:     /* 12 */ G12,       G13,        G14,        G15,
   907:     /* 16 */ GPMUCNT4,  GPMUCNT5,   GPMUCNT6,   GPMUCNT7,
   908:     /* 20 */ G20,       G21,        G22,        G23,
   909:     /* 24 */ GPCYCLELO, GPCYCLEHI,  GPMUCNT0,   GPMUCNT1,
   910:     /* 28 */ GPMUCNT2,  GPMUCNT3,   G30,        G31
   911:   };
   912: 
   913:   if (RegNo >= std::size(GuestRegDecoderTable))
   914:     return MCDisassembler::Fail;
   915:   if (GuestRegDecoderTable[RegNo] == Hexagon::NoRegister)
   916:     return MCDisassembler::Fail;
   917: 
   918:   unsigned Register = GuestRegDecoderTable[RegNo];
   919:   Inst.addOperand(MCOperand::createReg(Register));
   920:   return MCDisassembler::Success;
   921: }
   922: 
   923: static DecodeStatus
   924: DecodeGuestRegs64RegisterClass(MCInst &Inst, unsigned RegNo,
   925:                                uint64_t /*Address*/,
   926:                                const MCDisassembler *Decoder) {
   927:   using namespace Hexagon;
   928: 
   929:   static const MCPhysReg GuestReg64DecoderTable[] = {
   930:     /*  0 */ G1_0,      0,          G3_2,       0,
   931:     /*  4 */ G5_4,      0,          G7_6,       0,
   932:     /*  8 */ G9_8,      0,          G11_10,     0,
   933:     /* 12 */ G13_12,    0,          G15_14,     0,
   934:     /* 16 */ G17_16,    0,          G19_18,     0,
   935:     /* 20 */ G21_20,    0,          G23_22,     0,
   936:     /* 24 */ G25_24,    0,          G27_26,     0,
   937:     /* 28 */ G29_28,    0,          G31_30,     0
   938:   };
   939: 
   940:   if (RegNo >= std::size(GuestReg64DecoderTable))
   941:     return MCDisassembler::Fail;
   942:   if (GuestReg64DecoderTable[RegNo] == Hexagon::NoRegister)
   943:     return MCDisassembler::Fail;
   944: 
   945:   unsigned Register = GuestReg64DecoderTable[RegNo];
   946:   Inst.addOperand(MCOperand::createReg(Register));
   947:   return MCDisassembler::Success;
   948: }
```
- EN: It opens namespaces (Hexagon) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as addOperand, DecodeGuestRegs64RegisterClass, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里打开了命名空间（Hexagon），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 addOperand, DecodeGuestRegs64RegisterClass 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- instruction semantics / 指令语义
- MC-layer target description / MC 层目标描述
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `MCTargetDesc/HexagonBaseInfo.h, MCTargetDesc/HexagonMCChecker.h, MCTargetDesc/HexagonMCInstrInfo.h, MCTargetDesc/HexagonMCTargetDesc.h, TargetInfo/HexagonTargetInfo.h, llvm/ADT/ArrayRef.h, llvm/MC/MCContext.h, llvm/MC/MCDecoder.h, llvm/MC/MCDecoderOps.h, llvm/MC/MCDisassembler/MCDisassembler.h, ... (26 total)`
- Hexagon symbols / Hexagon 符号: `HexagonDisassembler, HexagonBaseInfo, HexagonMCChecker, HexagonMCInstrInfo, HexagonMCTargetDesc, HexagonTargetInfo, HexagonDepDecoders, HexagonGenDisassemblerTables, Hexagon_MC, HexagonII, ... (11 total)`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
