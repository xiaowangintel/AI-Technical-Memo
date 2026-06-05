# HexagonMCInstrInfo.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/MCTargetDesc/HexagonMCInstrInfo.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon instruction semantics, helpers, and packetization-aware queries.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。 重点涉及指令语义与选择。 重点涉及调度或成包。

## Line-by-Line Analysis / 逐行分析

### Lines 1-100 / 第 1-100 行

```cpp
     1: //===- HexagonMCInstrInfo.cpp - Hexagon sub-class of MCInst ---------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // This class extends MCInstrInfo to allow Hexagon specific MCInstr queries
    10: //
    11: //===----------------------------------------------------------------------===//
    12: 
    13: #include "MCTargetDesc/HexagonMCInstrInfo.h"
    14: #include "MCTargetDesc/HexagonBaseInfo.h"
    15: #include "MCTargetDesc/HexagonMCChecker.h"
    16: #include "MCTargetDesc/HexagonMCExpr.h"
    17: #include "MCTargetDesc/HexagonMCShuffler.h"
    18: #include "MCTargetDesc/HexagonMCTargetDesc.h"
    19: #include "llvm/ADT/SmallVector.h"
    20: #include "llvm/ADT/StringSwitch.h"
    21: #include "llvm/MC/MCContext.h"
    22: #include "llvm/MC/MCExpr.h"
    23: #include "llvm/MC/MCInst.h"
    24: #include "llvm/MC/MCInstrInfo.h"
    25: #include "llvm/MC/MCInstrItineraries.h"
    26: #include "llvm/MC/MCSubtargetInfo.h"
    27: #include "llvm/Support/Casting.h"
    28: #include "llvm/Support/ErrorHandling.h"
    29: #include <cassert>
    30: #include <cstdint>
    31: #include <limits>
    32: 
    33: using namespace llvm;
    34: 
    35: bool HexagonMCInstrInfo::PredicateInfo::isPredicated() const {
    36:   return Register != Hexagon::NoRegister;
    37: }
    38: 
    39: Hexagon::PacketIterator::PacketIterator(MCInstrInfo const &MCII,
    40:                                         MCInst const &Inst)
    41:     : MCII(MCII), BundleCurrent(Inst.begin() +
    42:                                 HexagonMCInstrInfo::bundleInstructionsOffset),
    43:       BundleEnd(Inst.end()), DuplexCurrent(Inst.end()), DuplexEnd(Inst.end()) {}
    44: 
    45: Hexagon::PacketIterator::PacketIterator(MCInstrInfo const &MCII,
    46:                                         MCInst const &Inst, std::nullptr_t)
    47:     : MCII(MCII), BundleCurrent(Inst.end()), BundleEnd(Inst.end()),
    48:       DuplexCurrent(Inst.end()), DuplexEnd(Inst.end()) {}
    49: 
    50: Hexagon::PacketIterator &Hexagon::PacketIterator::operator++() {
    51:   if (DuplexCurrent != DuplexEnd) {
    52:     ++DuplexCurrent;
    53:     if (DuplexCurrent == DuplexEnd) {
    54:       DuplexCurrent = BundleEnd;
    55:       DuplexEnd = BundleEnd;
    56:       ++BundleCurrent;
    57:     }
    58:     return *this;
    59:   }
    60:   ++BundleCurrent;
    61:   if (BundleCurrent != BundleEnd) {
    62:     MCInst const &Inst = *BundleCurrent->getInst();
    63:     if (HexagonMCInstrInfo::isDuplex(MCII, Inst)) {
    64:       DuplexCurrent = Inst.begin();
    65:       DuplexEnd = Inst.end();
    66:     }
    67:   }
    68:   return *this;
    69: }
    70: 
    71: MCInst const &Hexagon::PacketIterator::operator*() const {
    72:   if (DuplexCurrent != DuplexEnd)
    73:     return *DuplexCurrent->getInst();
    74:   return *BundleCurrent->getInst();
    75: }
    76: 
    77: bool Hexagon::PacketIterator::operator==(PacketIterator const &Other) const {
    78:   return BundleCurrent == Other.BundleCurrent && BundleEnd == Other.BundleEnd &&
    79:          DuplexCurrent == Other.DuplexCurrent && DuplexEnd == Other.DuplexEnd;
    80: }
    81: 
    82: void HexagonMCInstrInfo::addConstant(MCInst &MI, uint64_t Value,
    83:                                      MCContext &Context) {
    84:   MI.addOperand(MCOperand::createExpr(MCConstantExpr::create(Value, Context)));
    85: }
    86: 
    87: void HexagonMCInstrInfo::addConstExtender(MCContext &Context,
    88:                                           MCInstrInfo const &MCII, MCInst &MCB,
    89:                                           MCInst const &MCI) {
    90:   assert(HexagonMCInstrInfo::isBundle(MCB));
    91:   MCOperand const &exOp =
    92:       MCI.getOperand(HexagonMCInstrInfo::getExtendableOp(MCII, MCI));
    93: 
    94:   // Create the extender.
    95:   MCInst *XMCI =
    96:       new (Context) MCInst(HexagonMCInstrInfo::deriveExtender(MCII, MCI, exOp));
    97:   XMCI->setLoc(MCI.getLoc());
    98: 
    99:   MCB.addOperand(MCOperand::createInst(XMCI));
   100: }
```
- EN: It imports headers such as MCTargetDesc/HexagonMCInstrInfo.h, MCTargetDesc/HexagonBaseInfo.h, MCTargetDesc/HexagonMCChecker.h, MCTargetDesc/HexagonMCExpr.h, ... (19 total), establishing the LLVM/Hexagon APIs used below. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as HexagonMCInstrInfo::PredicateInfo::isPredicated, Hexagon::PacketIterator::PacketIterator, getInst, begin, ... (12 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里引入了 MCTargetDesc/HexagonMCInstrInfo.h, MCTargetDesc/HexagonBaseInfo.h, MCTargetDesc/HexagonMCChecker.h, MCTargetDesc/HexagonMCExpr.h, ... (19 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 HexagonMCInstrInfo::PredicateInfo::isPredicated, Hexagon::PacketIterator::PacketIterator, getInst, begin, ... (12 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 101-200 / 第 101-200 行

```cpp
   101: 
   102: iterator_range<Hexagon::PacketIterator>
   103: HexagonMCInstrInfo::bundleInstructions(MCInstrInfo const &MCII,
   104:                                        MCInst const &MCI) {
   105:   assert(isBundle(MCI));
   106:   return make_range(Hexagon::PacketIterator(MCII, MCI),
   107:                     Hexagon::PacketIterator(MCII, MCI, nullptr));
   108: }
   109: 
   110: iterator_range<MCInst::const_iterator>
   111: HexagonMCInstrInfo::bundleInstructions(MCInst const &MCI) {
   112:   assert(isBundle(MCI));
   113:   return drop_begin(MCI, bundleInstructionsOffset);
   114: }
   115: 
   116: size_t HexagonMCInstrInfo::bundleSize(MCInst const &MCI) {
   117:   if (HexagonMCInstrInfo::isBundle(MCI))
   118:     return (MCI.size() - bundleInstructionsOffset);
   119:   else
   120:     return (1);
   121: }
   122: 
   123: namespace {
   124: bool canonicalizePacketImpl(MCInstrInfo const &MCII, MCSubtargetInfo const &STI,
   125:                             MCContext &Context, MCInst &MCB,
   126:                             HexagonMCChecker *Check) {
   127:   // Check the bundle for errors.
   128:   bool CheckOk = Check ? Check->check(false) : true;
   129:   if (!CheckOk)
   130:     return false;
   131: 
   132:   MCInst OrigMCB = MCB;
   133: 
   134:   // Examine the packet and convert pairs of instructions to compound
   135:   // instructions when possible.
   136:   if (!HexagonDisableCompound)
   137:     HexagonMCInstrInfo::tryCompound(MCII, STI, Context, MCB);
   138:   HexagonMCShuffle(Context, false, MCII, STI, MCB);
   139: 
   140:   const SmallVector<DuplexCandidate, 8> possibleDuplexes =
   141:       (STI.hasFeature(Hexagon::FeatureDuplex))
   142:           ? HexagonMCInstrInfo::getDuplexPossibilties(MCII, STI, MCB)
   143:           : SmallVector<DuplexCandidate, 8>();
   144: 
   145:   // Examine the packet and convert pairs of instructions to duplex
   146:   // instructions when possible.
   147:   HexagonMCShuffle(Context, MCII, STI, MCB, possibleDuplexes);
   148: 
   149:   // Examines packet and pad the packet, if needed, when an
   150:   // end-loop is in the bundle.
   151:   HexagonMCInstrInfo::padEndloop(MCB, Context);
   152: 
   153:   // If compounding and duplexing didn't reduce the size below
   154:   // 4 or less we have a packet that is too big.
   155:   if (HexagonMCInstrInfo::bundleSize(MCB) > HEXAGON_PACKET_SIZE) {
   156:     if (Check)
   157:       Check->reportError("invalid instruction packet: out of slots");
   158:     return false;
   159:   }
   160:   // Check the bundle for errors.
   161:   CheckOk = Check ? Check->check(true) : true;
   162:   if (!CheckOk)
   163:     return false;
   164: 
   165:   HexagonMCShuffle(Context, true, MCII, STI, MCB);
   166: 
   167:   return true;
   168: }
   169: } // namespace
   170: 
   171: bool HexagonMCInstrInfo::canonicalizePacket(MCInstrInfo const &MCII,
   172:                                             MCSubtargetInfo const &STI,
   173:                                             MCContext &Context, MCInst &MCB,
   174:                                             HexagonMCChecker *Check,
   175:                                             bool AttemptCompatibility) {
   176:   auto ArchSTI = Hexagon_MC::getArchSubtarget(&STI);
   177:   if (!AttemptCompatibility || ArchSTI == nullptr)
   178:     return canonicalizePacketImpl(MCII, STI, Context, MCB, Check);
   179: 
   180:   const MCRegisterInfo *RI = Context.getRegisterInfo();
   181:   HexagonMCChecker DefaultCheck(Context, MCII, STI, MCB, *RI, false);
   182:   HexagonMCChecker *BaseCheck = (Check == nullptr) ? &DefaultCheck : Check;
   183:   HexagonMCChecker PerfCheck(*BaseCheck, STI, false);
   184:   if (canonicalizePacketImpl(MCII, STI, Context, MCB, &PerfCheck))
   185:     return true;
   186: 
   187:   HexagonMCChecker ArchCheck(*BaseCheck, *ArchSTI, true);
   188:   return canonicalizePacketImpl(MCII, *ArchSTI, Context, MCB, &ArchCheck);
   189: }
   190: 
   191: MCInst HexagonMCInstrInfo::deriveExtender(MCInstrInfo const &MCII,
   192:                                           MCInst const &Inst,
   193:                                           MCOperand const &MO) {
   194:   assert(HexagonMCInstrInfo::isExtendable(MCII, Inst) ||
   195:          HexagonMCInstrInfo::isExtended(MCII, Inst));
   196: 
   197:   MCInst XMI;
   198:   XMI.setOpcode(Hexagon::A4_ext);
   199:   if (MO.isImm())
   200:     XMI.addOperand(MCOperand::createImm(MO.getImm() & (~0x3f)));
```
- EN: It opens namespaces (bool) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as HexagonMCInstrInfo::bundleInstructions, assert, make_range, drop_begin, ... (17 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里打开了命名空间（bool），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 HexagonMCInstrInfo::bundleInstructions, assert, make_range, drop_begin, ... (17 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 201-300 / 第 201-300 行

```cpp
   201:   else if (MO.isExpr())
   202:     XMI.addOperand(MCOperand::createExpr(MO.getExpr()));
   203:   else
   204:     llvm_unreachable("invalid extendable operand");
   205:   return XMI;
   206: }
   207: 
   208: MCInst *HexagonMCInstrInfo::deriveDuplex(MCContext &Context, unsigned iClass,
   209:                                          MCInst const &inst0,
   210:                                          MCInst const &inst1) {
   211:   assert((iClass <= 0xf) && "iClass must have range of 0 to 0xf");
   212:   MCInst *duplexInst = new (Context) MCInst;
   213:   duplexInst->setOpcode(Hexagon::DuplexIClass0 + iClass);
   214: 
   215:   MCInst *SubInst0 = new (Context) MCInst(deriveSubInst(inst0));
   216:   MCInst *SubInst1 = new (Context) MCInst(deriveSubInst(inst1));
   217:   duplexInst->addOperand(MCOperand::createInst(SubInst0));
   218:   duplexInst->addOperand(MCOperand::createInst(SubInst1));
   219:   return duplexInst;
   220: }
   221: 
   222: MCInst const *HexagonMCInstrInfo::extenderForIndex(MCInst const &MCB,
   223:                                                    size_t Index) {
   224:   assert(Index <= bundleSize(MCB));
   225:   if (Index == 0)
   226:     return nullptr;
   227:   MCInst const *Inst =
   228:       MCB.getOperand(Index + bundleInstructionsOffset - 1).getInst();
   229:   if (isImmext(*Inst))
   230:     return Inst;
   231:   return nullptr;
   232: }
   233: 
   234: void HexagonMCInstrInfo::extendIfNeeded(MCContext &Context,
   235:                                         MCInstrInfo const &MCII, MCInst &MCB,
   236:                                         MCInst const &MCI) {
   237:   if (isConstExtended(MCII, MCI))
   238:     addConstExtender(Context, MCII, MCB, MCI);
   239: }
   240: 
   241: unsigned HexagonMCInstrInfo::getMemAccessSize(MCInstrInfo const &MCII,
   242:       MCInst const &MCI) {
   243:   uint64_t F = HexagonMCInstrInfo::getDesc(MCII, MCI).TSFlags;
   244:   unsigned S = (F >> HexagonII::MemAccessSizePos) & HexagonII::MemAccesSizeMask;
   245:   return HexagonII::getMemAccessSizeInBytes(HexagonII::MemAccessSize(S));
   246: }
   247: 
   248: unsigned HexagonMCInstrInfo::getAddrMode(MCInstrInfo const &MCII,
   249:                                          MCInst const &MCI) {
   250:   const uint64_t F = HexagonMCInstrInfo::getDesc(MCII, MCI).TSFlags;
   251:   return static_cast<unsigned>((F >> HexagonII::AddrModePos) &
   252:                                HexagonII::AddrModeMask);
   253: }
   254: 
   255: MCInstrDesc const &HexagonMCInstrInfo::getDesc(MCInstrInfo const &MCII,
   256:                                                MCInst const &MCI) {
   257:   return MCII.get(MCI.getOpcode());
   258: }
   259: 
   260: unsigned HexagonMCInstrInfo::getDuplexRegisterNumbering(MCRegister Reg) {
   261:   using namespace Hexagon;
   262: 
   263:   switch (Reg.id()) {
   264:   default:
   265:     llvm_unreachable("unknown duplex register");
   266:   // Rs       Rss
   267:   case R0:
   268:   case D0:
   269:     return 0;
   270:   case R1:
   271:   case D1:
   272:     return 1;
   273:   case R2:
   274:   case D2:
   275:     return 2;
   276:   case R3:
   277:   case D3:
   278:     return 3;
   279:   case R4:
   280:   case D8:
   281:     return 4;
   282:   case R5:
   283:   case D9:
   284:     return 5;
   285:   case R6:
   286:   case D10:
   287:     return 6;
   288:   case R7:
   289:   case D11:
   290:     return 7;
   291:   case R16:
   292:     return 8;
   293:   case R17:
   294:     return 9;
   295:   case R18:
   296:     return 10;
   297:   case R19:
   298:     return 11;
   299:   case R20:
   300:     return 12;
```
- EN: It opens namespaces (Hexagon) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as llvm_unreachable, HexagonMCInstrInfo::deriveDuplex, assert, setOpcode, ... (16 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里打开了命名空间（Hexagon），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 llvm_unreachable, HexagonMCInstrInfo::deriveDuplex, assert, setOpcode, ... (16 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 301-400 / 第 301-400 行

```cpp
   301:   case R21:
   302:     return 13;
   303:   case R22:
   304:     return 14;
   305:   case R23:
   306:     return 15;
   307:   }
   308: }
   309: 
   310: MCExpr const &HexagonMCInstrInfo::getExpr(MCExpr const &Expr) {
   311:   const auto &HExpr = cast<HexagonMCExpr>(Expr);
   312:   assert(HExpr.getExpr());
   313:   return *HExpr.getExpr();
   314: }
   315: 
   316: unsigned short HexagonMCInstrInfo::getExtendableOp(MCInstrInfo const &MCII,
   317:                                                    MCInst const &MCI) {
   318:   const uint64_t F = HexagonMCInstrInfo::getDesc(MCII, MCI).TSFlags;
   319:   return ((F >> HexagonII::ExtendableOpPos) & HexagonII::ExtendableOpMask);
   320: }
   321: 
   322: MCOperand const &
   323: HexagonMCInstrInfo::getExtendableOperand(MCInstrInfo const &MCII,
   324:                                          MCInst const &MCI) {
   325:   unsigned O = HexagonMCInstrInfo::getExtendableOp(MCII, MCI);
   326:   MCOperand const &MO = MCI.getOperand(O);
   327: 
   328:   assert((HexagonMCInstrInfo::isExtendable(MCII, MCI) ||
   329:           HexagonMCInstrInfo::isExtended(MCII, MCI)) &&
   330:          (MO.isImm() || MO.isExpr()));
   331:   return (MO);
   332: }
   333: 
   334: unsigned HexagonMCInstrInfo::getExtentAlignment(MCInstrInfo const &MCII,
   335:                                                 MCInst const &MCI) {
   336:   const uint64_t F = HexagonMCInstrInfo::getDesc(MCII, MCI).TSFlags;
   337:   return ((F >> HexagonII::ExtentAlignPos) & HexagonII::ExtentAlignMask);
   338: }
   339: 
   340: unsigned HexagonMCInstrInfo::getExtentBits(MCInstrInfo const &MCII,
   341:                                            MCInst const &MCI) {
   342:   const uint64_t F = HexagonMCInstrInfo::getDesc(MCII, MCI).TSFlags;
   343:   return ((F >> HexagonII::ExtentBitsPos) & HexagonII::ExtentBitsMask);
   344: }
   345: 
   346: bool HexagonMCInstrInfo::isExtentSigned(MCInstrInfo const &MCII,
   347:                                         MCInst const &MCI) {
   348:   const uint64_t F = HexagonMCInstrInfo::getDesc(MCII, MCI).TSFlags;
   349:   return (F >> HexagonII::ExtentSignedPos) & HexagonII::ExtentSignedMask;
   350: }
   351: 
   352: /// Return the maximum value of an extendable operand.
   353: int HexagonMCInstrInfo::getMaxValue(MCInstrInfo const &MCII,
   354:                                     MCInst const &MCI) {
   355:   assert(HexagonMCInstrInfo::isExtendable(MCII, MCI) ||
   356:          HexagonMCInstrInfo::isExtended(MCII, MCI));
   357: 
   358:   if (HexagonMCInstrInfo::isExtentSigned(MCII, MCI)) // if value is signed
   359:     return (1 << (HexagonMCInstrInfo::getExtentBits(MCII, MCI) - 1)) - 1;
   360:   return (1 << HexagonMCInstrInfo::getExtentBits(MCII, MCI)) - 1;
   361: }
   362: 
   363: /// Return the minimum value of an extendable operand.
   364: int HexagonMCInstrInfo::getMinValue(MCInstrInfo const &MCII,
   365:                                     MCInst const &MCI) {
   366:   assert(HexagonMCInstrInfo::isExtendable(MCII, MCI) ||
   367:          HexagonMCInstrInfo::isExtended(MCII, MCI));
   368: 
   369:   if (HexagonMCInstrInfo::isExtentSigned(MCII, MCI)) // if value is signed
   370:     return -(1 << (HexagonMCInstrInfo::getExtentBits(MCII, MCI) - 1));
   371:   return 0;
   372: }
   373: 
   374: StringRef HexagonMCInstrInfo::getName(MCInstrInfo const &MCII,
   375:                                       MCInst const &MCI) {
   376:   return MCII.getName(MCI.getOpcode());
   377: }
   378: 
   379: unsigned short HexagonMCInstrInfo::getNewValueOp(MCInstrInfo const &MCII,
   380:                                                  MCInst const &MCI) {
   381:   const uint64_t F = HexagonMCInstrInfo::getDesc(MCII, MCI).TSFlags;
   382:   return ((F >> HexagonII::NewValueOpPos) & HexagonII::NewValueOpMask);
   383: }
   384: 
   385: MCOperand const &HexagonMCInstrInfo::getNewValueOperand(MCInstrInfo const &MCII,
   386:                                                         MCInst const &MCI) {
   387:   if (HexagonMCInstrInfo::hasTmpDst(MCII, MCI)) {
   388:     // VTMP doesn't actually exist in the encodings for these 184
   389:     // 3 instructions so go ahead and create it here.
   390:     static MCOperand MCO = MCOperand::createReg(Hexagon::VTMP);
   391:     return (MCO);
   392:   } else {
   393:     unsigned O = HexagonMCInstrInfo::getNewValueOp(MCII, MCI);
   394:     MCOperand const &MCO = MCI.getOperand(O);
   395: 
   396:     assert((HexagonMCInstrInfo::isNewValue(MCII, MCI) ||
   397:             HexagonMCInstrInfo::hasNewValue(MCII, MCI)) &&
   398:            MCO.isReg());
   399:     return (MCO);
   400:   }
```
- EN: It declares or implements routines such as HexagonMCInstrInfo::getExpr, cast<HexagonMCExpr>, assert, getExpr, ... (17 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonMCInstrInfo, HexagonMCExpr, HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonMCInstrInfo::getExpr, cast<HexagonMCExpr>, assert, getExpr, ... (17 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonMCInstrInfo, HexagonMCExpr, HexagonII，说明了它与同级后端组件的连接关系。

### Lines 401-500 / 第 401-500 行

```cpp
   401: }
   402: 
   403: /// Return the new value or the newly produced value.
   404: unsigned short HexagonMCInstrInfo::getNewValueOp2(MCInstrInfo const &MCII,
   405:                                                   MCInst const &MCI) {
   406:   const uint64_t F = HexagonMCInstrInfo::getDesc(MCII, MCI).TSFlags;
   407:   return ((F >> HexagonII::NewValueOpPos2) & HexagonII::NewValueOpMask2);
   408: }
   409: 
   410: MCOperand const &
   411: HexagonMCInstrInfo::getNewValueOperand2(MCInstrInfo const &MCII,
   412:                                         MCInst const &MCI) {
   413:   unsigned O = HexagonMCInstrInfo::getNewValueOp2(MCII, MCI);
   414:   MCOperand const &MCO = MCI.getOperand(O);
   415: 
   416:   assert((HexagonMCInstrInfo::isNewValue(MCII, MCI) ||
   417:           HexagonMCInstrInfo::hasNewValue2(MCII, MCI)) &&
   418:          MCO.isReg());
   419:   return (MCO);
   420: }
   421: 
   422: /// Return the Hexagon ISA class for the insn.
   423: unsigned HexagonMCInstrInfo::getType(MCInstrInfo const &MCII,
   424:                                      MCInst const &MCI) {
   425:   const uint64_t F = MCII.get(MCI.getOpcode()).TSFlags;
   426:   return ((F >> HexagonII::TypePos) & HexagonII::TypeMask);
   427: }
   428: 
   429: /// Return the resources used by this instruction
   430: unsigned HexagonMCInstrInfo::getCVIResources(MCInstrInfo const &MCII,
   431:                                       MCSubtargetInfo const &STI,
   432:                                       MCInst const &MCI) {
   433: 
   434:   const InstrItinerary *II = STI.getSchedModel().InstrItineraries;
   435:   int SchedClass = HexagonMCInstrInfo::getDesc(MCII, MCI).getSchedClass();
   436:   int Size = II[SchedClass].LastStage - II[SchedClass].FirstStage;
   437: 
   438:   // HVX resources used are currently located at the second to last stage.
   439:   // This could also be done with a linear search of the stages looking for:
   440:   // CVI_ALL, CVI_MPY01, CVI_XLSHF, CVI_MPY0, CVI_MPY1, CVI_SHIFT, CVI_XLANE,
   441:   // CVI_ZW
   442:   unsigned Stage = II[SchedClass].LastStage - 1;
   443: 
   444:   if (Size < 2)
   445:     return 0;
   446:   return ((Stage + HexagonStages)->getUnits());
   447: }
   448: 
   449: /// Return the slots this instruction can execute out of
   450: unsigned HexagonMCInstrInfo::getUnits(MCInstrInfo const &MCII,
   451:                                       MCSubtargetInfo const &STI,
   452:                                       MCInst const &MCI) {
   453:   const InstrItinerary *II = STI.getSchedModel().InstrItineraries;
   454:   int SchedClass = HexagonMCInstrInfo::getDesc(MCII, MCI).getSchedClass();
   455:   return ((II[SchedClass].FirstStage + HexagonStages)->getUnits());
   456: }
   457: 
   458: /// Return the slots this instruction consumes in addition to
   459: /// the slot(s) it can execute out of
   460: 
   461: unsigned HexagonMCInstrInfo::getOtherReservedSlots(MCInstrInfo const &MCII,
   462:                                                    MCSubtargetInfo const &STI,
   463:                                                    MCInst const &MCI) {
   464:   const InstrItinerary *II = STI.getSchedModel().InstrItineraries;
   465:   int SchedClass = HexagonMCInstrInfo::getDesc(MCII, MCI).getSchedClass();
   466:   unsigned Slots = 0;
   467: 
   468:   // FirstStage are slots that this instruction can execute in.
   469:   // FirstStage+1 are slots that are also consumed by this instruction.
   470:   // For example: vmemu can only execute in slot 0 but also consumes slot 1.
   471:   for (unsigned Stage = II[SchedClass].FirstStage + 1;
   472:        Stage < II[SchedClass].LastStage; ++Stage) {
   473:     unsigned Units = (Stage + HexagonStages)->getUnits();
   474:     if (Units > HexagonGetLastSlot())
   475:       break;
   476:     // fyi: getUnits() will return 0x1, 0x2, 0x4 or 0x8
   477:     Slots |= Units;
   478:   }
   479: 
   480:   // if 0 is returned, then no additional slots are consumed by this inst.
   481:   return Slots;
   482: }
   483: 
   484: bool HexagonMCInstrInfo::hasDuplex(MCInstrInfo const &MCII, MCInst const &MCI) {
   485:   if (!HexagonMCInstrInfo::isBundle(MCI))
   486:     return false;
   487: 
   488:   for (auto const &I : HexagonMCInstrInfo::bundleInstructions(MCI)) {
   489:     if (HexagonMCInstrInfo::isDuplex(MCII, *I.getInst()))
   490:       return true;
   491:   }
   492: 
   493:   return false;
   494: }
   495: 
   496: bool HexagonMCInstrInfo::hasExtenderForIndex(MCInst const &MCB, size_t Index) {
   497:   return extenderForIndex(MCB, Index) != nullptr;
   498: }
   499: 
   500: bool HexagonMCInstrInfo::hasImmExt(MCInst const &MCI) {
```
- EN: It declares or implements routines such as HexagonMCInstrInfo::getNewValueOp2, HexagonMCInstrInfo::getNewValueOperand2, getOperand, assert, ... (13 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonMCInstrInfo, HexagonII, HexagonStages, HexagonGetLastSlot, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonMCInstrInfo::getNewValueOp2, HexagonMCInstrInfo::getNewValueOperand2, getOperand, assert, ... (13 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonMCInstrInfo, HexagonII, HexagonStages, HexagonGetLastSlot，说明了它与同级后端组件的连接关系。

### Lines 501-600 / 第 501-600 行

```cpp
   501:   if (!HexagonMCInstrInfo::isBundle(MCI))
   502:     return false;
   503: 
   504:   for (const auto &I : HexagonMCInstrInfo::bundleInstructions(MCI)) {
   505:     if (isImmext(*I.getInst()))
   506:       return true;
   507:   }
   508: 
   509:   return false;
   510: }
   511: 
   512: /// Return whether the insn produces a value.
   513: bool HexagonMCInstrInfo::hasNewValue(MCInstrInfo const &MCII,
   514:                                      MCInst const &MCI) {
   515:   const uint64_t F = HexagonMCInstrInfo::getDesc(MCII, MCI).TSFlags;
   516:   return ((F >> HexagonII::hasNewValuePos) & HexagonII::hasNewValueMask);
   517: }
   518: 
   519: /// Return whether the insn produces a second value.
   520: bool HexagonMCInstrInfo::hasNewValue2(MCInstrInfo const &MCII,
   521:                                       MCInst const &MCI) {
   522:   const uint64_t F = HexagonMCInstrInfo::getDesc(MCII, MCI).TSFlags;
   523:   return ((F >> HexagonII::hasNewValuePos2) & HexagonII::hasNewValueMask2);
   524: }
   525: 
   526: MCInst const &HexagonMCInstrInfo::instruction(MCInst const &MCB, size_t Index) {
   527:   assert(isBundle(MCB));
   528:   assert(Index < HEXAGON_PRESHUFFLE_PACKET_SIZE);
   529:   return *MCB.getOperand(bundleInstructionsOffset + Index).getInst();
   530: }
   531: 
   532: /// Return where the instruction is an accumulator.
   533: bool HexagonMCInstrInfo::isAccumulator(MCInstrInfo const &MCII,
   534:                                        MCInst const &MCI) {
   535:   const uint64_t F = HexagonMCInstrInfo::getDesc(MCII, MCI).TSFlags;
   536:   return ((F >> HexagonII::AccumulatorPos) & HexagonII::AccumulatorMask);
   537: }
   538: 
   539: bool HexagonMCInstrInfo::isBundle(MCInst const &MCI) {
   540:   auto Result = Hexagon::BUNDLE == MCI.getOpcode();
   541:   assert(!Result || (MCI.size() > 0 && MCI.getOperand(0).isImm()));
   542:   return Result;
   543: }
   544: 
   545: bool HexagonMCInstrInfo::isConstExtended(MCInstrInfo const &MCII,
   546:                                          MCInst const &MCI) {
   547:   if (HexagonMCInstrInfo::isExtended(MCII, MCI))
   548:     return true;
   549:   if (!HexagonMCInstrInfo::isExtendable(MCII, MCI))
   550:     return false;
   551:   MCOperand const &MO = HexagonMCInstrInfo::getExtendableOperand(MCII, MCI);
   552:   if (isa<HexagonMCExpr>(MO.getExpr()) &&
   553:       HexagonMCInstrInfo::mustExtend(*MO.getExpr()))
   554:     return true;
   555:   // Branch insns are handled as necessary by relaxation.
   556:   if ((HexagonMCInstrInfo::getType(MCII, MCI) == HexagonII::TypeJ) ||
   557:       (HexagonMCInstrInfo::getType(MCII, MCI) == HexagonII::TypeCJ &&
   558:        HexagonMCInstrInfo::getDesc(MCII, MCI).isBranch()) ||
   559:       (HexagonMCInstrInfo::getType(MCII, MCI) == HexagonII::TypeNCJ &&
   560:        HexagonMCInstrInfo::getDesc(MCII, MCI).isBranch()))
   561:     return false;
   562:   // Otherwise loop instructions and other CR insts are handled by relaxation
   563:   else if ((HexagonMCInstrInfo::getType(MCII, MCI) == HexagonII::TypeCR) &&
   564:            (MCI.getOpcode() != Hexagon::C4_addipc))
   565:     return false;
   566: 
   567:   assert(!MO.isImm());
   568:   if (isa<HexagonMCExpr>(MO.getExpr()) &&
   569:       HexagonMCInstrInfo::mustNotExtend(*MO.getExpr()))
   570:     return false;
   571: 
   572:   int64_t Value;
   573:   if (!MO.getExpr()->evaluateAsAbsolute(Value))
   574:     return true;
   575:   if (HexagonMCInstrInfo::isExtentSigned(MCII, MCI)) {
   576:     int32_t SValue = Value;
   577:     int32_t MinValue = HexagonMCInstrInfo::getMinValue(MCII, MCI);
   578:     int32_t MaxValue = HexagonMCInstrInfo::getMaxValue(MCII, MCI);
   579:     return SValue < MinValue || SValue > MaxValue;
   580:   }
   581:   uint32_t UValue = Value;
   582:   uint32_t MinValue = HexagonMCInstrInfo::getMinValue(MCII, MCI);
   583:   uint32_t MaxValue = HexagonMCInstrInfo::getMaxValue(MCII, MCI);
   584:   return UValue < MinValue || UValue > MaxValue;
   585: }
   586: 
   587: bool HexagonMCInstrInfo::isCanon(MCInstrInfo const &MCII, MCInst const &MCI) {
   588:   return !HexagonMCInstrInfo::getDesc(MCII, MCI).isPseudo() &&
   589:          !HexagonMCInstrInfo::isPrefix(MCII, MCI);
   590: }
   591: 
   592: bool HexagonMCInstrInfo::isCofMax1(MCInstrInfo const &MCII, MCInst const &MCI) {
   593:   const uint64_t F = HexagonMCInstrInfo::getDesc(MCII, MCI).TSFlags;
   594:   return ((F >> HexagonII::CofMax1Pos) & HexagonII::CofMax1Mask);
   595: }
   596: 
   597: bool HexagonMCInstrInfo::isCofRelax1(MCInstrInfo const &MCII,
   598:                                      MCInst const &MCI) {
   599:   const uint64_t F = HexagonMCInstrInfo::getDesc(MCII, MCI).TSFlags;
   600:   return ((F >> HexagonII::CofRelax1Pos) & HexagonII::CofRelax1Mask);
```
- EN: It declares or implements routines such as HexagonMCInstrInfo::hasNewValue, HexagonMCInstrInfo::hasNewValue2, HexagonMCInstrInfo::instruction, assert, ... (16 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonMCInstrInfo, HexagonII, HexagonMCExpr, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonMCInstrInfo::hasNewValue, HexagonMCInstrInfo::hasNewValue2, HexagonMCInstrInfo::instruction, assert, ... (16 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonMCInstrInfo, HexagonII, HexagonMCExpr，说明了它与同级后端组件的连接关系。

### Lines 601-700 / 第 601-700 行

```cpp
   601: }
   602: 
   603: bool HexagonMCInstrInfo::isCofRelax2(MCInstrInfo const &MCII,
   604:                                      MCInst const &MCI) {
   605:   const uint64_t F = HexagonMCInstrInfo::getDesc(MCII, MCI).TSFlags;
   606:   return ((F >> HexagonII::CofRelax2Pos) & HexagonII::CofRelax2Mask);
   607: }
   608: 
   609: bool HexagonMCInstrInfo::isCompound(MCInstrInfo const &MCII,
   610:                                     MCInst const &MCI) {
   611:   return (getType(MCII, MCI) == HexagonII::TypeCJ);
   612: }
   613: 
   614: bool HexagonMCInstrInfo::isCVINew(MCInstrInfo const &MCII, MCInst const &MCI) {
   615:   const uint64_t F = HexagonMCInstrInfo::getDesc(MCII, MCI).TSFlags;
   616:   return ((F >> HexagonII::CVINewPos) & HexagonII::CVINewMask);
   617: }
   618: 
   619: bool HexagonMCInstrInfo::isDblRegForSubInst(MCRegister Reg) {
   620:   return ((Reg >= Hexagon::D0 && Reg <= Hexagon::D3) ||
   621:           (Reg >= Hexagon::D8 && Reg <= Hexagon::D11));
   622: }
   623: 
   624: bool HexagonMCInstrInfo::isDuplex(MCInstrInfo const &MCII, MCInst const &MCI) {
   625:   return HexagonII::TypeDUPLEX == HexagonMCInstrInfo::getType(MCII, MCI);
   626: }
   627: 
   628: bool HexagonMCInstrInfo::isExtendable(MCInstrInfo const &MCII,
   629:                                       MCInst const &MCI) {
   630:   uint64_t const F = HexagonMCInstrInfo::getDesc(MCII, MCI).TSFlags;
   631:   return (F >> HexagonII::ExtendablePos) & HexagonII::ExtendableMask;
   632: }
   633: 
   634: bool HexagonMCInstrInfo::isExtended(MCInstrInfo const &MCII,
   635:                                     MCInst const &MCI) {
   636:   uint64_t const F = HexagonMCInstrInfo::getDesc(MCII, MCI).TSFlags;
   637:   return (F >> HexagonII::ExtendedPos) & HexagonII::ExtendedMask;
   638: }
   639: 
   640: bool HexagonMCInstrInfo::isFloat(MCInstrInfo const &MCII, MCInst const &MCI) {
   641:   const uint64_t F = HexagonMCInstrInfo::getDesc(MCII, MCI).TSFlags;
   642:   return ((F >> HexagonII::FPPos) & HexagonII::FPMask);
   643: }
   644: 
   645: bool HexagonMCInstrInfo::isHVX(MCInstrInfo const &MCII, MCInst const &MCI) {
   646:   const uint64_t V = getType(MCII, MCI);
   647:   return HexagonII::TypeCVI_FIRST <= V && V <= HexagonII::TypeCVI_LAST;
   648: }
   649: 
   650: bool HexagonMCInstrInfo::isImmext(MCInst const &MCI) {
   651:   return MCI.getOpcode() == Hexagon::A4_ext;
   652: }
   653: 
   654: bool HexagonMCInstrInfo::isInnerLoop(MCInst const &MCI) {
   655:   assert(isBundle(MCI));
   656:   int64_t Flags = MCI.getOperand(0).getImm();
   657:   return (Flags & innerLoopMask) != 0;
   658: }
   659: 
   660: bool HexagonMCInstrInfo::isIntReg(MCRegister Reg) {
   661:   return (Reg >= Hexagon::R0 && Reg <= Hexagon::R31);
   662: }
   663: 
   664: bool HexagonMCInstrInfo::isIntRegForSubInst(MCRegister Reg) {
   665:   return ((Reg >= Hexagon::R0 && Reg <= Hexagon::R7) ||
   666:           (Reg >= Hexagon::R16 && Reg <= Hexagon::R23));
   667: }
   668: 
   669: /// Return whether the insn expects newly produced value.
   670: bool HexagonMCInstrInfo::isNewValue(MCInstrInfo const &MCII,
   671:                                     MCInst const &MCI) {
   672:   const uint64_t F = HexagonMCInstrInfo::getDesc(MCII, MCI).TSFlags;
   673:   return ((F >> HexagonII::NewValuePos) & HexagonII::NewValueMask);
   674: }
   675: 
   676: bool HexagonMCInstrInfo::isNewValueStore(MCInstrInfo const &MCII,
   677:                                          MCInst const &MCI) {
   678:   const uint64_t F = HexagonMCInstrInfo::getDesc(MCII, MCI).TSFlags;
   679:   return (F >> HexagonII::NVStorePos) & HexagonII::NVStoreMask;
   680: }
   681: 
   682: /// Return whether the operand is extendable.
   683: bool HexagonMCInstrInfo::isOpExtendable(MCInstrInfo const &MCII,
   684:                                         MCInst const &MCI, unsigned short O) {
   685:   return (O == HexagonMCInstrInfo::getExtendableOp(MCII, MCI));
   686: }
   687: 
   688: bool HexagonMCInstrInfo::isOuterLoop(MCInst const &MCI) {
   689:   assert(isBundle(MCI));
   690:   int64_t Flags = MCI.getOperand(0).getImm();
   691:   return (Flags & outerLoopMask) != 0;
   692: }
   693: 
   694: bool HexagonMCInstrInfo::IsVecRegPair(MCRegister VecReg) {
   695:   return (VecReg >= Hexagon::W0 && VecReg <= Hexagon::W15) ||
   696:          (VecReg >= Hexagon::WR0 && VecReg <= Hexagon::WR15);
   697: }
   698: 
   699: bool HexagonMCInstrInfo::IsReverseVecRegPair(MCRegister VecReg) {
   700:   return (VecReg >= Hexagon::WR0 && VecReg <= Hexagon::WR15);
```
- EN: It declares or implements routines such as HexagonMCInstrInfo::isCofRelax2, HexagonMCInstrInfo::isCompound, HexagonMCInstrInfo::isCVINew, HexagonMCInstrInfo::isDblRegForSubInst, ... (23 total), translating Hexagon-specific policy into reusable code paths. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonMCInstrInfo, HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonMCInstrInfo::isCofRelax2, HexagonMCInstrInfo::isCompound, HexagonMCInstrInfo::isCVINew, HexagonMCInstrInfo::isDblRegForSubInst, ... (23 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonMCInstrInfo, HexagonII，说明了它与同级后端组件的连接关系。

### Lines 701-800 / 第 701-800 行

```cpp
   701: }
   702: 
   703: bool HexagonMCInstrInfo::IsVecRegSingle(MCRegister VecReg) {
   704:   return (VecReg >= Hexagon::V0 && VecReg <= Hexagon::V31);
   705: }
   706: 
   707: std::pair<unsigned, unsigned>
   708: HexagonMCInstrInfo::GetVecRegPairIndices(MCRegister VecRegPair) {
   709:   assert(IsVecRegPair(VecRegPair) &&
   710:          "VecRegPair must be a vector register pair");
   711: 
   712:   const bool IsRev = IsReverseVecRegPair(VecRegPair);
   713:   const unsigned PairIndex =
   714:       2 * (IsRev ? VecRegPair - Hexagon::WR0 : VecRegPair - Hexagon::W0);
   715: 
   716:   return IsRev ? std::make_pair(PairIndex, PairIndex + 1)
   717:                : std::make_pair(PairIndex + 1, PairIndex);
   718: }
   719: 
   720: bool HexagonMCInstrInfo::IsSingleConsumerRefPairProducer(MCRegister Producer,
   721:                                                          MCRegister Consumer) {
   722:   if (IsVecRegPair(Producer) && IsVecRegSingle(Consumer)) {
   723:     const unsigned ProdPairIndex = IsReverseVecRegPair(Producer)
   724:                                        ? Producer - Hexagon::WR0
   725:                                        : Producer - Hexagon::W0;
   726:     const unsigned ConsumerSingleIndex = (Consumer - Hexagon::V0) >> 1;
   727: 
   728:     return ConsumerSingleIndex == ProdPairIndex;
   729:   }
   730:   return false;
   731: }
   732: 
   733: bool HexagonMCInstrInfo::isPredicated(MCInstrInfo const &MCII,
   734:                                       MCInst const &MCI) {
   735:   const uint64_t F = HexagonMCInstrInfo::getDesc(MCII, MCI).TSFlags;
   736:   return ((F >> HexagonII::PredicatedPos) & HexagonII::PredicatedMask);
   737: }
   738: 
   739: bool HexagonMCInstrInfo::isPrefix(MCInstrInfo const &MCII, MCInst const &MCI) {
   740:   return HexagonII::TypeEXTENDER == HexagonMCInstrInfo::getType(MCII, MCI);
   741: }
   742: 
   743: bool HexagonMCInstrInfo::isPredicateLate(MCInstrInfo const &MCII,
   744:                                          MCInst const &MCI) {
   745:   const uint64_t F = HexagonMCInstrInfo::getDesc(MCII, MCI).TSFlags;
   746:   return (F >> HexagonII::PredicateLatePos & HexagonII::PredicateLateMask);
   747: }
   748: 
   749: /// Return whether the insn is newly predicated.
   750: bool HexagonMCInstrInfo::isPredicatedNew(MCInstrInfo const &MCII,
   751:                                          MCInst const &MCI) {
   752:   const uint64_t F = HexagonMCInstrInfo::getDesc(MCII, MCI).TSFlags;
   753:   return ((F >> HexagonII::PredicatedNewPos) & HexagonII::PredicatedNewMask);
   754: }
   755: 
   756: bool HexagonMCInstrInfo::isPredicatedTrue(MCInstrInfo const &MCII,
   757:                                           MCInst const &MCI) {
   758:   const uint64_t F = HexagonMCInstrInfo::getDesc(MCII, MCI).TSFlags;
   759:   return (
   760:       !((F >> HexagonII::PredicatedFalsePos) & HexagonII::PredicatedFalseMask));
   761: }
   762: 
   763: bool HexagonMCInstrInfo::isPredReg(MCRegisterInfo const &MRI, MCRegister Reg) {
   764:   auto &PredRegClass = MRI.getRegClass(Hexagon::PredRegsRegClassID);
   765:   return PredRegClass.contains(Reg);
   766: }
   767: 
   768: bool HexagonMCInstrInfo::isPredRegister(MCInstrInfo const &MCII,
   769:                                         MCInst const &Inst, unsigned I) {
   770:   MCInstrDesc const &Desc = HexagonMCInstrInfo::getDesc(MCII, Inst);
   771: 
   772:   return Inst.getOperand(I).isReg() &&
   773:          Desc.operands()[I].RegClass == Hexagon::PredRegsRegClassID;
   774: }
   775: 
   776: /// Return whether the insn can be packaged only with A and X-type insns.
   777: bool HexagonMCInstrInfo::isSoloAX(MCInstrInfo const &MCII, MCInst const &MCI) {
   778:   const uint64_t F = HexagonMCInstrInfo::getDesc(MCII, MCI).TSFlags;
   779:   return ((F >> HexagonII::SoloAXPos) & HexagonII::SoloAXMask);
   780: }
   781: 
   782: /// Return whether the insn can be packaged only with an A-type insn in slot #1.
   783: bool HexagonMCInstrInfo::isRestrictSlot1AOK(MCInstrInfo const &MCII,
   784:                                             MCInst const &MCI) {
   785:   const uint64_t F = HexagonMCInstrInfo::getDesc(MCII, MCI).TSFlags;
   786:   return ((F >> HexagonII::RestrictSlot1AOKPos) &
   787:           HexagonII::RestrictSlot1AOKMask);
   788: }
   789: 
   790: bool HexagonMCInstrInfo::isRestrictNoSlot1Store(MCInstrInfo const &MCII,
   791:                                                 MCInst const &MCI) {
   792:   const uint64_t F = HexagonMCInstrInfo::getDesc(MCII, MCI).TSFlags;
   793:   return ((F >> HexagonII::RestrictNoSlot1StorePos) &
   794:           HexagonII::RestrictNoSlot1StoreMask);
   795: }
   796: 
   797: /// Return whether the insn is solo, i.e., cannot be in a packet.
   798: bool HexagonMCInstrInfo::isSolo(MCInstrInfo const &MCII, MCInst const &MCI) {
   799:   const uint64_t F = MCII.get(MCI.getOpcode()).TSFlags;
   800:   return ((F >> HexagonII::SoloPos) & HexagonII::SoloMask);
```
- EN: It declares or implements routines such as HexagonMCInstrInfo::IsVecRegSingle, HexagonMCInstrInfo::GetVecRegPairIndices, assert, IsReverseVecRegPair, ... (21 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonMCInstrInfo, HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonMCInstrInfo::IsVecRegSingle, HexagonMCInstrInfo::GetVecRegPairIndices, assert, IsReverseVecRegPair, ... (21 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonMCInstrInfo, HexagonII，说明了它与同级后端组件的连接关系。

### Lines 801-900 / 第 801-900 行

```cpp
   801: }
   802: 
   803: bool HexagonMCInstrInfo::isMemReorderDisabled(MCInst const &MCI) {
   804:   assert(isBundle(MCI));
   805:   auto Flags = MCI.getOperand(0).getImm();
   806:   return (Flags & memReorderDisabledMask) != 0;
   807: }
   808: 
   809: bool HexagonMCInstrInfo::isSubInstruction(MCInst const &MCI) {
   810:   switch (MCI.getOpcode()) {
   811:   default:
   812:     return false;
   813:   case Hexagon::SA1_addi:
   814:   case Hexagon::SA1_addrx:
   815:   case Hexagon::SA1_addsp:
   816:   case Hexagon::SA1_and1:
   817:   case Hexagon::SA1_clrf:
   818:   case Hexagon::SA1_clrfnew:
   819:   case Hexagon::SA1_clrt:
   820:   case Hexagon::SA1_clrtnew:
   821:   case Hexagon::SA1_cmpeqi:
   822:   case Hexagon::SA1_combine0i:
   823:   case Hexagon::SA1_combine1i:
   824:   case Hexagon::SA1_combine2i:
   825:   case Hexagon::SA1_combine3i:
   826:   case Hexagon::SA1_combinerz:
   827:   case Hexagon::SA1_combinezr:
   828:   case Hexagon::SA1_dec:
   829:   case Hexagon::SA1_inc:
   830:   case Hexagon::SA1_seti:
   831:   case Hexagon::SA1_setin1:
   832:   case Hexagon::SA1_sxtb:
   833:   case Hexagon::SA1_sxth:
   834:   case Hexagon::SA1_tfr:
   835:   case Hexagon::SA1_zxtb:
   836:   case Hexagon::SA1_zxth:
   837:   case Hexagon::SL1_loadri_io:
   838:   case Hexagon::SL1_loadrub_io:
   839:   case Hexagon::SL2_deallocframe:
   840:   case Hexagon::SL2_jumpr31:
   841:   case Hexagon::SL2_jumpr31_f:
   842:   case Hexagon::SL2_jumpr31_fnew:
   843:   case Hexagon::SL2_jumpr31_t:
   844:   case Hexagon::SL2_jumpr31_tnew:
   845:   case Hexagon::SL2_loadrb_io:
   846:   case Hexagon::SL2_loadrd_sp:
   847:   case Hexagon::SL2_loadrh_io:
   848:   case Hexagon::SL2_loadri_sp:
   849:   case Hexagon::SL2_loadruh_io:
   850:   case Hexagon::SL2_return:
   851:   case Hexagon::SL2_return_f:
   852:   case Hexagon::SL2_return_fnew:
   853:   case Hexagon::SL2_return_t:
   854:   case Hexagon::SL2_return_tnew:
   855:   case Hexagon::SS1_storeb_io:
   856:   case Hexagon::SS1_storew_io:
   857:   case Hexagon::SS2_allocframe:
   858:   case Hexagon::SS2_storebi0:
   859:   case Hexagon::SS2_storebi1:
   860:   case Hexagon::SS2_stored_sp:
   861:   case Hexagon::SS2_storeh_io:
   862:   case Hexagon::SS2_storew_sp:
   863:   case Hexagon::SS2_storewi0:
   864:   case Hexagon::SS2_storewi1:
   865:     return true;
   866:   }
   867: }
   868: 
   869: bool HexagonMCInstrInfo::isVector(MCInstrInfo const &MCII, MCInst const &MCI) {
   870:   const uint64_t F = HexagonMCInstrInfo::getDesc(MCII, MCI).TSFlags;
   871:   return (F >> HexagonII::isCVIPos) & HexagonII::isCVIMask;
   872: }
   873: 
   874: int64_t HexagonMCInstrInfo::minConstant(MCInst const &MCI, size_t Index) {
   875:   auto Sentinel = static_cast<int64_t>(std::numeric_limits<uint32_t>::max())
   876:                   << 8;
   877:   if (MCI.size() <= Index)
   878:     return Sentinel;
   879:   MCOperand const &MCO = MCI.getOperand(Index);
   880:   if (!MCO.isExpr())
   881:     return Sentinel;
   882:   int64_t Value;
   883:   if (!MCO.getExpr()->evaluateAsAbsolute(Value))
   884:     return Sentinel;
   885:   return Value;
   886: }
   887: 
   888: void HexagonMCInstrInfo::setMustExtend(MCExpr const &Expr, bool Val) {
   889:   HexagonMCExpr &HExpr = const_cast<HexagonMCExpr &>(cast<HexagonMCExpr>(Expr));
   890:   HExpr.setMustExtend(Val);
   891: }
   892: 
   893: bool HexagonMCInstrInfo::mustExtend(MCExpr const &Expr) {
   894:   HexagonMCExpr const &HExpr = cast<HexagonMCExpr>(Expr);
   895:   return HExpr.mustExtend();
   896: }
   897: void HexagonMCInstrInfo::setMustNotExtend(MCExpr const &Expr, bool Val) {
   898:   HexagonMCExpr &HExpr = const_cast<HexagonMCExpr &>(cast<HexagonMCExpr>(Expr));
   899:   HExpr.setMustNotExtend(Val);
   900: }
```
- EN: It declares or implements routines such as HexagonMCInstrInfo::isMemReorderDisabled, assert, getOperand, HexagonMCInstrInfo::isSubInstruction, ... (13 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonMCInstrInfo, HexagonII, HexagonMCExpr, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonMCInstrInfo::isMemReorderDisabled, assert, getOperand, HexagonMCInstrInfo::isSubInstruction, ... (13 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonMCInstrInfo, HexagonII, HexagonMCExpr，说明了它与同级后端组件的连接关系。

### Lines 901-1000 / 第 901-1000 行

```cpp
   901: bool HexagonMCInstrInfo::mustNotExtend(MCExpr const &Expr) {
   902:   HexagonMCExpr const &HExpr = cast<HexagonMCExpr>(Expr);
   903:   return HExpr.mustNotExtend();
   904: }
   905: void HexagonMCInstrInfo::setS27_2_reloc(MCExpr const &Expr, bool Val) {
   906:   HexagonMCExpr &HExpr =
   907:       const_cast<HexagonMCExpr &>(*cast<HexagonMCExpr>(&Expr));
   908:   HExpr.setS27_2_reloc(Val);
   909: }
   910: bool HexagonMCInstrInfo::s27_2_reloc(MCExpr const &Expr) {
   911:   HexagonMCExpr const *HExpr = dyn_cast<HexagonMCExpr>(&Expr);
   912:   if (!HExpr)
   913:     return false;
   914:   return HExpr->s27_2_reloc();
   915: }
   916: 
   917: unsigned HexagonMCInstrInfo::packetSizeSlots(MCSubtargetInfo const &STI) {
   918:   const bool IsTiny = STI.hasFeature(Hexagon::ProcTinyCore);
   919: 
   920:   return IsTiny ? (HEXAGON_PACKET_SIZE - 1) : HEXAGON_PACKET_SIZE;
   921: }
   922: 
   923: unsigned HexagonMCInstrInfo::packetSize(StringRef CPU) {
   924:   return llvm::StringSwitch<unsigned>(CPU)
   925:       .Case("hexagonv67t", 3)
   926:       .Default(4);
   927: }
   928: 
   929: void HexagonMCInstrInfo::padEndloop(MCInst &MCB, MCContext &Context) {
   930:   MCInst Nop;
   931:   Nop.setOpcode(Hexagon::A2_nop);
   932:   assert(isBundle(MCB));
   933:   while (LoopNeedsPadding(MCB))
   934:     MCB.addOperand(MCOperand::createInst(new (Context) MCInst(Nop)));
   935: }
   936: 
   937: HexagonMCInstrInfo::PredicateInfo
   938: HexagonMCInstrInfo::predicateInfo(MCInstrInfo const &MCII, MCInst const &MCI) {
   939:   if (!isPredicated(MCII, MCI))
   940:     return {0, 0, false};
   941:   MCInstrDesc const &Desc = getDesc(MCII, MCI);
   942:   for (auto I = Desc.getNumDefs(), N = Desc.getNumOperands(); I != N; ++I)
   943:     if (Desc.operands()[I].RegClass == Hexagon::PredRegsRegClassID)
   944:       return {MCI.getOperand(I).getReg(), I, isPredicatedTrue(MCII, MCI)};
   945:   return {0, 0, false};
   946: }
   947: 
   948: bool HexagonMCInstrInfo::prefersSlot3(MCInstrInfo const &MCII,
   949:                                       MCInst const &MCI) {
   950:   const uint64_t F = HexagonMCInstrInfo::getDesc(MCII, MCI).TSFlags;
   951:   return (F >> HexagonII::PrefersSlot3Pos) & HexagonII::PrefersSlot3Mask;
   952: }
   953: 
   954: bool HexagonMCInstrInfo::hasTmpDst(MCInstrInfo const &MCII, MCInst const &MCI) {
   955:   switch (MCI.getOpcode()) {
   956:   default:
   957:     return false;
   958:   case Hexagon::V6_vgathermh:
   959:   case Hexagon::V6_vgathermhq:
   960:   case Hexagon::V6_vgathermhw:
   961:   case Hexagon::V6_vgathermhwq:
   962:   case Hexagon::V6_vgathermw:
   963:   case Hexagon::V6_vgathermwq:
   964:     return true;
   965:   }
   966:   return false;
   967: }
   968: 
   969: bool HexagonMCInstrInfo::hasHvxTmp(MCInstrInfo const &MCII, MCInst const &MCI) {
   970:   const uint64_t F = HexagonMCInstrInfo::getDesc(MCII, MCI).TSFlags;
   971:   return (F >> HexagonII::HasHvxTmpPos) & HexagonII::HasHvxTmpMask;
   972: }
   973: 
   974: bool HexagonMCInstrInfo::requiresSlot(MCSubtargetInfo const &STI,
   975:                                       MCInst const &MCI) {
   976:   const unsigned OpCode = MCI.getOpcode();
   977:   const bool IsTiny = STI.getFeatureBits() [Hexagon::ProcTinyCore];
   978:   const bool NoSlotReqd = Hexagon::A4_ext == OpCode ||
   979:                           (IsTiny && Hexagon::A2_nop == OpCode) ||
   980:                           (IsTiny && Hexagon::J4_hintjumpr == OpCode);
   981: 
   982:   return !NoSlotReqd;
   983: }
   984: 
   985: unsigned HexagonMCInstrInfo::slotsConsumed(MCInstrInfo const &MCII,
   986:                                            MCSubtargetInfo const &STI,
   987:                                            MCInst const &MCI) {
   988:   unsigned slotsUsed = 0;
   989:   for (auto HMI : bundleInstructions(MCI)) {
   990:     MCInst const &MCI = *HMI.getInst();
   991:     if (!requiresSlot(STI, MCI))
   992:       continue;
   993:     if (isDuplex(MCII, MCI))
   994:       slotsUsed += 2;
   995:     else
   996:       ++slotsUsed;
   997:   }
   998:   return slotsUsed;
   999: }
  1000: 
```
- EN: It declares or implements routines such as HexagonMCInstrInfo::mustNotExtend, cast<HexagonMCExpr>, mustNotExtend, HexagonMCInstrInfo::setS27_2_reloc, ... (24 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonMCInstrInfo, HexagonMCExpr, HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonMCInstrInfo::mustNotExtend, cast<HexagonMCExpr>, mustNotExtend, HexagonMCInstrInfo::setS27_2_reloc, ... (24 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonMCInstrInfo, HexagonMCExpr, HexagonII，说明了它与同级后端组件的连接关系。

### Lines 1001-1062 / 第 1001-1062 行

```cpp
  1001: void HexagonMCInstrInfo::replaceDuplex(MCContext &Context, MCInst &MCB,
  1002:                                        DuplexCandidate Candidate) {
  1003:   assert(Candidate.packetIndexI < MCB.size());
  1004:   assert(Candidate.packetIndexJ < MCB.size());
  1005:   assert(isBundle(MCB));
  1006:   MCInst *Duplex =
  1007:       deriveDuplex(Context, Candidate.iClass,
  1008:                    *MCB.getOperand(Candidate.packetIndexJ).getInst(),
  1009:                    *MCB.getOperand(Candidate.packetIndexI).getInst());
  1010:   assert(Duplex != nullptr);
  1011:   MCB.getOperand(Candidate.packetIndexI).setInst(Duplex);
  1012:   MCB.erase(MCB.begin() + Candidate.packetIndexJ);
  1013: }
  1014: 
  1015: void HexagonMCInstrInfo::setInnerLoop(MCInst &MCI) {
  1016:   assert(isBundle(MCI));
  1017:   MCOperand &Operand = MCI.getOperand(0);
  1018:   Operand.setImm(Operand.getImm() | innerLoopMask);
  1019: }
  1020: 
  1021: void HexagonMCInstrInfo::setMemReorderDisabled(MCInst &MCI) {
  1022:   assert(isBundle(MCI));
  1023:   MCOperand &Operand = MCI.getOperand(0);
  1024:   Operand.setImm(Operand.getImm() | memReorderDisabledMask);
  1025:   assert(isMemReorderDisabled(MCI));
  1026: }
  1027: 
  1028: void HexagonMCInstrInfo::setOuterLoop(MCInst &MCI) {
  1029:   assert(isBundle(MCI));
  1030:   MCOperand &Operand = MCI.getOperand(0);
  1031:   Operand.setImm(Operand.getImm() | outerLoopMask);
  1032: }
  1033: 
  1034: unsigned HexagonMCInstrInfo::SubregisterBit(MCRegister Consumer,
  1035:                                             MCRegister Producer,
  1036:                                             MCRegister Producer2) {
  1037:   // If we're a single vector consumer of a double producer, set subreg bit
  1038:   // based on if we're accessing the lower or upper register component
  1039:   if (IsVecRegPair(Producer) && IsVecRegSingle(Consumer)) {
  1040:     unsigned Rev = IsReverseVecRegPair(Producer);
  1041:     return ((Consumer - Hexagon::V0) & 0x1) ^ Rev;
  1042:   }
  1043:   if (Producer2 != Hexagon::NoRegister)
  1044:     return Consumer == Producer;
  1045:   return 0;
  1046: }
  1047: 
  1048: bool HexagonMCInstrInfo::LoopNeedsPadding(MCInst const &MCB) {
  1049:   return (
  1050:       (HexagonMCInstrInfo::isInnerLoop(MCB) &&
  1051:        (HexagonMCInstrInfo::bundleSize(MCB) < HEXAGON_PACKET_INNER_SIZE)) ||
  1052:       ((HexagonMCInstrInfo::isOuterLoop(MCB) &&
  1053:         (HexagonMCInstrInfo::bundleSize(MCB) < HEXAGON_PACKET_OUTER_SIZE))));
  1054: }
  1055: 
  1056: bool HexagonMCInstrInfo::IsABranchingInst(MCInstrInfo const &MCII,
  1057:                                           MCSubtargetInfo const &STI,
  1058:                                           MCInst const &I) {
  1059:   assert(!HexagonMCInstrInfo::isBundle(I));
  1060:   MCInstrDesc const &Desc = HexagonMCInstrInfo::getDesc(MCII, I);
  1061:   return (Desc.isBranch() || Desc.isCall() || Desc.isReturn());
  1062: }
```
- EN: It declares or implements routines such as HexagonMCInstrInfo::replaceDuplex, assert, deriveDuplex, getOperand, ... (14 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonMCInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonMCInstrInfo::replaceDuplex, assert, deriveDuplex, getOperand, ... (14 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonMCInstrInfo，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- VLIW packetization / VLIW 成包
- register modeling / 寄存器建模
- instruction semantics / 指令语义
- MC-layer target description / MC 层目标描述
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `MCTargetDesc/HexagonMCInstrInfo.h, MCTargetDesc/HexagonBaseInfo.h, MCTargetDesc/HexagonMCChecker.h, MCTargetDesc/HexagonMCExpr.h, MCTargetDesc/HexagonMCShuffler.h, MCTargetDesc/HexagonMCTargetDesc.h, llvm/ADT/SmallVector.h, llvm/ADT/StringSwitch.h, llvm/MC/MCContext.h, llvm/MC/MCExpr.h, ... (19 total)`
- Hexagon symbols / Hexagon 符号: `HexagonMCInstrInfo, HexagonBaseInfo, HexagonMCChecker, HexagonMCExpr, HexagonMCShuffler, HexagonMCTargetDesc, HexagonDisableCompound, HexagonMCShuffle, Hexagon_MC, HexagonII, ... (12 total)`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
