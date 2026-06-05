# HexagonShuffler.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/MCTargetDesc/HexagonShuffler.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon MC-layer target description glue.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```cpp
     1: //===- HexagonShuffler.cpp - Instruction bundle shuffling -----------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // This implements the shuffling of insns inside a bundle according to the
    10: // packet formation rules of the Hexagon ISA.
    11: //
    12: //===----------------------------------------------------------------------===//
    13: 
    14: #include "MCTargetDesc/HexagonShuffler.h"
    15: #include "MCTargetDesc/HexagonBaseInfo.h"
    16: #include "MCTargetDesc/HexagonMCInstrInfo.h"
    17: #include "MCTargetDesc/HexagonMCTargetDesc.h"
    18: #include "llvm/ADT/SmallVector.h"
    19: #include "llvm/ADT/StringExtras.h"
    20: #include "llvm/ADT/Twine.h"
    21: #include "llvm/MC/MCContext.h"
    22: #include "llvm/MC/MCInst.h"
    23: #include "llvm/MC/MCInstrDesc.h"
    24: #include "llvm/MC/MCSubtargetInfo.h"
    25: #include "llvm/Support/Compiler.h"
    26: #include "llvm/Support/Debug.h"
    27: #include "llvm/Support/SourceMgr.h"
    28: #include "llvm/Support/raw_ostream.h"
    29: #include <algorithm>
    30: #include <cassert>
    31: #include <optional>
    32: #include <utility>
    33: 
    34: #define DEBUG_TYPE "hexagon-shuffle"
    35: 
    36: using namespace llvm;
    37: 
    38: namespace {
    39: 
    40: // Insn shuffling priority.
    41: class HexagonBid {
    42:   // The priority is directly proportional to how restricted the insn is based
    43:   // on its flexibility to run on the available slots.  So, the fewer slots it
    44:   // may run on, the higher its priority.
    45:   enum { MAX = 360360 }; // LCD of 1/2, 1/3, 1/4,... 1/15.
    46:   unsigned Bid = 0;
    47: 
    48: public:
    49:   HexagonBid() = default;
    50:   HexagonBid(unsigned B) { Bid = B ? MAX / llvm::popcount(B) : 0; }
```
- EN: It imports headers such as MCTargetDesc/HexagonShuffler.h, MCTargetDesc/HexagonBaseInfo.h, MCTargetDesc/HexagonMCInstrInfo.h, MCTargetDesc/HexagonMCTargetDesc.h, ... (19 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonBid, which carry the state or API of this component.
- CN: 这里引入了 MCTargetDesc/HexagonShuffler.h, MCTargetDesc/HexagonBaseInfo.h, MCTargetDesc/HexagonMCInstrInfo.h, MCTargetDesc/HexagonMCTargetDesc.h, ... (19 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonBid 等类型，用来承载该组件的状态或接口。

### Lines 51-100 / 第 51-100 行

```cpp
    51: 
    52:   // Check if the insn priority is overflowed.
    53:   bool isSold() const { return (Bid >= MAX); }
    54: 
    55:   HexagonBid &operator+=(const HexagonBid &B) {
    56:     Bid += B.Bid;
    57:     return *this;
    58:   }
    59: };
    60: 
    61: // Slot shuffling allocation.
    62: class HexagonUnitAuction {
    63:   HexagonBid Scores[HEXAGON_PACKET_SIZE];
    64:   // Mask indicating which slot is unavailable.
    65:   unsigned isSold : HEXAGON_PACKET_SIZE;
    66: 
    67: public:
    68:   HexagonUnitAuction(unsigned cs = 0) : isSold(cs) {}
    69: 
    70:   // Allocate slots.
    71:   bool bid(unsigned B) {
    72:     // Exclude already auctioned slots from the bid.
    73:     unsigned b = B & ~isSold;
    74:     if (b) {
    75:       for (unsigned i = 0; i < HEXAGON_PACKET_SIZE; ++i)
    76:         if (b & (1 << i)) {
    77:           // Request candidate slots.
    78:           Scores[i] += HexagonBid(b);
    79:           isSold |= Scores[i].isSold() << i;
    80:         }
    81:       return true;
    82:     } else
    83:       // Error if the desired slots are already full.
    84:       return false;
    85:   }
    86: };
    87: 
    88: } // end anonymous namespace
    89: 
    90: unsigned HexagonResource::setWeight(unsigned s) {
    91:   const unsigned SlotWeight = 8;
    92:   const unsigned MaskWeight = SlotWeight - 1;
    93:   unsigned Units = getUnits();
    94:   unsigned Key = ((1u << s) & Units) != 0;
    95: 
    96:   // Calculate relative weight of the insn for the given slot, weighing it the
    97:   // heavier the more restrictive the insn is and the lowest the slots that the
    98:   // insn may be executed in.
    99:   if (Key == 0 || Units == 0 || (SlotWeight * s >= 32))
   100:     return Weight = 0;
```
- EN: It opens namespaces (unsigned) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonUnitAuction, which carry the state or API of this component. It defines declarative TableGen records like HexagonUnitAuction; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as isSold, HexagonUnitAuction, bid, HexagonBid, ... (6 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里打开了命名空间（unsigned），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonUnitAuction 等类型，用来承载该组件的状态或接口。 这里定义了 HexagonUnitAuction 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 isSold, HexagonUnitAuction, bid, HexagonBid, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 101-150 / 第 101-150 行

```cpp
   101: 
   102:   unsigned Ctpop = llvm::popcount(Units);
   103:   unsigned Cttz = llvm::countr_zero(Units);
   104:   Weight = (1u << (SlotWeight * s)) * ((MaskWeight - Ctpop) << Cttz);
   105:   return Weight;
   106: }
   107: 
   108: HexagonCVIResource::HexagonCVIResource(MCInstrInfo const &MCII,
   109:                                        MCSubtargetInfo const &STI,
   110:                                        unsigned s,
   111:                                        MCInst const *id)
   112:     : HexagonResource(s) {
   113: 
   114:   const unsigned ItinUnits = HexagonMCInstrInfo::getCVIResources(MCII, STI, *id);
   115:   unsigned Lanes;
   116:   const unsigned Units = HexagonConvertUnits(ItinUnits, &Lanes);
   117: 
   118:   if (Units == 0 && Lanes == 0) {
   119:     // For core insns.
   120:     Valid = false;
   121:     setUnits(0);
   122:     setLanes(0);
   123:     setLoad(false);
   124:     setStore(false);
   125:   } else {
   126:     // For an HVX insn.
   127:     Valid = true;
   128:     setUnits(Units);
   129:     setLanes(Lanes);
   130:     setLoad(HexagonMCInstrInfo::getDesc(MCII, *id).mayLoad());
   131:     setStore(HexagonMCInstrInfo::getDesc(MCII, *id).mayStore());
   132:   }
   133: }
   134: 
   135: struct CVIUnits {
   136:   unsigned Units;
   137:   unsigned Lanes;
   138: };
   139: using HVXInstsT = SmallVector<struct CVIUnits, 8>;
   140: 
   141: static unsigned makeAllBits(unsigned startBit, unsigned Lanes)
   142: {
   143:   for (unsigned i = 1; i < Lanes; ++i)
   144:     startBit = (startBit << 1) | startBit;
   145:   return startBit;
   146: }
   147: 
   148: static bool checkHVXPipes(const HVXInstsT &hvxInsts, unsigned startIdx,
   149:                           unsigned usedUnits) {
   150:   if (startIdx < hvxInsts.size()) {
```
- EN: It declares types such as CVIUnits, which carry the state or API of this component. It declares or implements routines such as llvm::popcount, llvm::countr_zero, HexagonCVIResource::HexagonCVIResource, HexagonMCInstrInfo::getCVIResources, ... (11 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonCVIResource, HexagonResource, HexagonMCInstrInfo, HexagonConvertUnits, showing how the code connects to sibling backend components.
- CN: 这里声明了 CVIUnits 等类型，用来承载该组件的状态或接口。 这里声明或实现了 llvm::popcount, llvm::countr_zero, HexagonCVIResource::HexagonCVIResource, HexagonMCInstrInfo::getCVIResources, ... (11 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonCVIResource, HexagonResource, HexagonMCInstrInfo, HexagonConvertUnits，说明了它与同级后端组件的连接关系。

### Lines 151-200 / 第 151-200 行

```cpp
   151:     if (!hvxInsts[startIdx].Units)
   152:       return checkHVXPipes(hvxInsts, startIdx + 1, usedUnits);
   153:     for (unsigned b = 0x1; b <= 0x8; b <<= 1) {
   154:       if ((hvxInsts[startIdx].Units & b) == 0)
   155:         continue;
   156:       unsigned allBits = makeAllBits(b, hvxInsts[startIdx].Lanes);
   157:       if ((allBits & usedUnits) == 0) {
   158:         if (checkHVXPipes(hvxInsts, startIdx + 1, usedUnits | allBits))
   159:           return true;
   160:       }
   161:     }
   162:     return false;
   163:   }
   164:   return true;
   165: }
   166: 
   167: HexagonShuffler::HexagonShuffler(MCContext &Context, bool ReportErrors,
   168:                                  MCInstrInfo const &MCII,
   169:                                  MCSubtargetInfo const &STI)
   170:     : Context(Context), BundleFlags(), MCII(MCII), STI(STI),
   171:       ReportErrors(ReportErrors), CheckFailure() {
   172:   reset();
   173: }
   174: 
   175: void HexagonShuffler::reset() {
   176:   Packet.clear();
   177:   BundleFlags = 0;
   178:   CheckFailure = false;
   179: }
   180: 
   181: void HexagonShuffler::append(MCInst const &ID, MCInst const *Extender,
   182:                              unsigned S) {
   183:   HexagonInstr PI(MCII, STI, &ID, Extender, S);
   184: 
   185:   Packet.push_back(PI);
   186: }
   187: 
   188: 
   189: static const unsigned Slot0Mask = 1 << 0;
   190: static const unsigned Slot1Mask = 1 << 1;
   191: static const unsigned Slot3Mask = 1 << 3;
   192: static const unsigned slotSingleLoad = Slot0Mask;
   193: static const unsigned slotSingleStore = Slot0Mask;
   194: 
   195: void HexagonShuffler::restrictSlot1AOK(HexagonPacketSummary const &Summary) {
   196:   if (Summary.Slot1AOKLoc)
   197:     for (HexagonInstr &ISJ : insts()) {
   198:       MCInst const &Inst = ISJ.getDesc();
   199:       const unsigned Type = HexagonMCInstrInfo::getType(MCII, Inst);
   200:       if (Type != HexagonII::TypeALU32_2op &&
```
- EN: It declares or implements routines such as makeAllBits, HexagonShuffler::HexagonShuffler, reset, HexagonShuffler::reset, ... (11 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonShuffler, HexagonInstr, HexagonPacketSummary, HexagonMCInstrInfo, ... (5 total), showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 makeAllBits, HexagonShuffler::HexagonShuffler, reset, HexagonShuffler::reset, ... (11 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonShuffler, HexagonInstr, HexagonPacketSummary, HexagonMCInstrInfo, ... (5 total)，说明了它与同级后端组件的连接关系。

### Lines 201-250 / 第 201-250 行

```cpp
   201:           Type != HexagonII::TypeALU32_3op &&
   202:           Type != HexagonII::TypeALU32_ADDI) {
   203:         const unsigned Units = ISJ.Core.getUnits();
   204: 
   205:         if (Units & Slot1Mask) {
   206:           AppliedRestrictions.push_back(std::make_pair(
   207:               Inst.getLoc(),
   208:               "Instruction was restricted from being in slot 1"));
   209:           AppliedRestrictions.push_back(std::make_pair(
   210:               *Summary.Slot1AOKLoc, "Instruction can only be combined "
   211:                                     "with an ALU instruction in slot 1"));
   212:           ISJ.Core.setUnits(Units & ~Slot1Mask);
   213:         }
   214:       }
   215:     }
   216: }
   217: 
   218: void HexagonShuffler::restrictNoSlot1Store(
   219:     HexagonPacketSummary const &Summary) {
   220:   // If this packet contains an instruction that bars slot-1 stores,
   221:   // we should mask off slot 1 from all of the store instructions in
   222:   // this packet.
   223: 
   224:   if (!Summary.NoSlot1StoreLoc)
   225:     return;
   226: 
   227:   bool AppliedRestriction = false;
   228: 
   229:   for (HexagonInstr &ISJ : insts()) {
   230:     MCInst const &Inst = ISJ.getDesc();
   231:     if (HexagonMCInstrInfo::getDesc(MCII, Inst).mayStore()) {
   232:       unsigned Units = ISJ.Core.getUnits();
   233:       if (Units & Slot1Mask) {
   234:         AppliedRestriction = true;
   235:         AppliedRestrictions.push_back(std::make_pair(
   236:             Inst.getLoc(), "Instruction was restricted from being in slot 1"));
   237:         ISJ.Core.setUnits(Units & ~Slot1Mask);
   238:       }
   239:     }
   240:   }
   241: 
   242:   if (AppliedRestriction)
   243:     AppliedRestrictions.push_back(
   244:         std::make_pair(*Summary.NoSlot1StoreLoc,
   245:                        "Instruction does not allow a store in slot 1"));
   246: }
   247: 
   248: bool HexagonShuffler::applySlotRestrictions(HexagonPacketSummary const &Summary,
   249:                                             const bool DoShuffle) {
   250:   // These restrictions can modify the slot masks in the instructions
```
- EN: It declares or implements routines such as getUnits, push_back, setUnits, HexagonShuffler::restrictNoSlot1Store, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonII, HexagonShuffler, HexagonPacketSummary, HexagonInstr, ... (5 total), showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getUnits, push_back, setUnits, HexagonShuffler::restrictNoSlot1Store, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonII, HexagonShuffler, HexagonPacketSummary, HexagonInstr, ... (5 total)，说明了它与同级后端组件的连接关系。

### Lines 251-300 / 第 251-300 行

```cpp
   251:   // in the Packet member.  They should run unconditionally and their
   252:   // order does not matter.
   253:   restrictSlot1AOK(Summary);
   254:   restrictNoSlot1Store(Summary);
   255: 
   256:   permitNonSlot();
   257: 
   258:   // These restrictions can modify the slot masks in the instructions
   259:   // in the Packet member, but they can also detect constraint failures
   260:   // which are fatal.
   261:   if (!CheckFailure)
   262:     restrictStoreLoadOrder(Summary);
   263:   if (!CheckFailure)
   264:     restrictBranchOrder(Summary);
   265:   if (!CheckFailure)
   266:     restrictPreferSlot3(Summary, DoShuffle);
   267:   return !CheckFailure;
   268: }
   269: 
   270: void HexagonShuffler::restrictBranchOrder(HexagonPacketSummary const &Summary) {
   271:   // preserve branch order
   272:   const bool HasMultipleBranches = Summary.branchInsts.size() > 1;
   273:   if (!HasMultipleBranches)
   274:     return;
   275: 
   276:   if (Summary.branchInsts.size() > 2) {
   277:     reportError(Twine("too many branches in packet"));
   278:     return;
   279:   }
   280: 
   281:   const static std::pair<unsigned, unsigned> jumpSlots[] = {
   282:       {8, 4}, {8, 2}, {8, 1}, {4, 2}, {4, 1}, {2, 1}};
   283:   // try all possible choices
   284:   for (std::pair<unsigned, unsigned> jumpSlot : jumpSlots) {
   285:     // validate first jump with this slot rule
   286:     if (!(jumpSlot.first & Summary.branchInsts[0]->Core.getUnits()))
   287:       continue;
   288: 
   289:     // validate second jump with this slot rule
   290:     if (!(jumpSlot.second & Summary.branchInsts[1]->Core.getUnits()))
   291:       continue;
   292: 
   293:     // both valid for this configuration, set new slot rules
   294:     const HexagonPacket PacketSave = Packet;
   295:     Summary.branchInsts[0]->Core.setUnits(jumpSlot.first);
   296:     Summary.branchInsts[1]->Core.setUnits(jumpSlot.second);
   297: 
   298:     const bool HasShuffledPacket = tryAuction(Summary).has_value();
   299:     if (HasShuffledPacket)
   300:       return;
```
- EN: It declares or implements routines such as restrictSlot1AOK, restrictNoSlot1Store, permitNonSlot, HexagonShuffler::restrictBranchOrder, ... (7 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonShuffler, HexagonPacketSummary, HexagonPacket, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 restrictSlot1AOK, restrictNoSlot1Store, permitNonSlot, HexagonShuffler::restrictBranchOrder, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonShuffler, HexagonPacketSummary, HexagonPacket，说明了它与同级后端组件的连接关系。

### Lines 301-350 / 第 301-350 行

```cpp
   301: 
   302:     // if yes, great, if not then restore original slot mask
   303:     // restore original values
   304:     Packet = PacketSave;
   305:   }
   306: 
   307:   reportResourceError(Summary, "out of slots");
   308: }
   309: 
   310: void HexagonShuffler::permitNonSlot() {
   311:   for (HexagonInstr &ISJ : insts()) {
   312:     const bool RequiresSlot = HexagonMCInstrInfo::requiresSlot(STI, *ISJ.ID);
   313:     if (!RequiresSlot)
   314:       ISJ.Core.setAllUnits();
   315:   }
   316: }
   317: 
   318: bool HexagonShuffler::ValidResourceUsage(HexagonPacketSummary const &Summary) {
   319:   std::optional<HexagonPacket> ShuffledPacket = tryAuction(Summary);
   320: 
   321:   if (!ShuffledPacket) {
   322:     reportResourceError(Summary, "slot error");
   323:     return false;
   324:   }
   325: 
   326:   // Verify the CVI slot subscriptions.
   327:   llvm::stable_sort(*ShuffledPacket, HexagonInstr::lessCVI);
   328:   // create vector of hvx instructions to check
   329:   HVXInstsT hvxInsts;
   330:   hvxInsts.clear();
   331:   for (const auto &I : *ShuffledPacket) {
   332:     struct CVIUnits inst;
   333:     inst.Units = I.CVI.getUnits();
   334:     inst.Lanes = I.CVI.getLanes();
   335:     if (inst.Units == 0)
   336:       continue; // not an hvx inst or an hvx inst that doesn't uses any pipes
   337:     hvxInsts.push_back(inst);
   338:   }
   339: 
   340:   // if there are any hvx instructions in this packet, check pipe usage
   341:   if (hvxInsts.size() > 0) {
   342:     unsigned startIdx, usedUnits;
   343:     startIdx = usedUnits = 0x0;
   344:     if (!checkHVXPipes(hvxInsts, startIdx, usedUnits)) {
   345:       // too many pipes used to be valid
   346:       reportError(Twine("invalid instruction packet: slot error"));
   347:       return false;
   348:     }
   349:   }
   350: 
```
- EN: It declares types such as CVIUnits, which carry the state or API of this component. It declares or implements routines such as reportResourceError, HexagonShuffler::permitNonSlot, HexagonMCInstrInfo::requiresSlot, HexagonShuffler::ValidResourceUsage, ... (11 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonShuffler, HexagonInstr, HexagonMCInstrInfo, HexagonPacketSummary, ... (5 total), showing how the code connects to sibling backend components.
- CN: 这里声明了 CVIUnits 等类型，用来承载该组件的状态或接口。 这里声明或实现了 reportResourceError, HexagonShuffler::permitNonSlot, HexagonMCInstrInfo::requiresSlot, HexagonShuffler::ValidResourceUsage, ... (11 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonShuffler, HexagonInstr, HexagonMCInstrInfo, HexagonPacketSummary, ... (5 total)，说明了它与同级后端组件的连接关系。

### Lines 351-400 / 第 351-400 行

```cpp
   351:   Packet = *ShuffledPacket;
   352: 
   353:   return true;
   354: }
   355: 
   356: bool HexagonShuffler::restrictStoreLoadOrder(
   357:     HexagonPacketSummary const &Summary) {
   358:   // Modify packet accordingly.
   359:   // TODO: need to reserve slots #0 and #1 for duplex insns.
   360:   static const unsigned slotFirstLoadStore = Slot1Mask;
   361:   static const unsigned slotLastLoadStore = Slot0Mask;
   362:   unsigned slotLoadStore = slotFirstLoadStore;
   363: 
   364:   for (iterator ISJ = begin(); ISJ != end(); ++ISJ) {
   365:     MCInst const &ID = ISJ->getDesc();
   366: 
   367:     if (!ISJ->Core.getUnits())
   368:       // Error if insn may not be executed in any slot.
   369:       return false;
   370: 
   371:     // A single load must use slot #0.
   372:     if (HexagonMCInstrInfo::getDesc(MCII, ID).mayLoad()) {
   373:       if (Summary.loads == 1 && Summary.loads == Summary.memory &&
   374:           Summary.memops == 0)
   375:         // Pin the load to slot #0.
   376:         switch (ID.getOpcode()) {
   377:         case Hexagon::V6_vgathermw:
   378:         case Hexagon::V6_vgathermh:
   379:         case Hexagon::V6_vgathermhw:
   380:         case Hexagon::V6_vgathermwq:
   381:         case Hexagon::V6_vgathermhq:
   382:         case Hexagon::V6_vgathermhwq:
   383:           // Slot1 only loads
   384:           break;
   385:         default:
   386:           ISJ->Core.setUnits(ISJ->Core.getUnits() & slotSingleLoad);
   387:           break;
   388:         }
   389:       else if (Summary.loads >= 1 && isMemReorderDisabled()) { // }:mem_noshuf
   390:         // Loads must keep the original order ONLY if
   391:         // isMemReorderDisabled() == true
   392:         if (slotLoadStore < slotLastLoadStore) {
   393:           // Error if no more slots available for loads.
   394:           reportError("invalid instruction packet: too many loads");
   395:           return false;
   396:         }
   397:         // Pin the load to the highest slot available to it.
   398:         ISJ->Core.setUnits(ISJ->Core.getUnits() & slotLoadStore);
   399:         // Update the next highest slot available to loads.
   400:         slotLoadStore >>= 1;
```
- EN: It declares or implements routines such as HexagonShuffler::restrictStoreLoadOrder, end, getDesc, setUnits, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonShuffler, HexagonPacketSummary, HexagonMCInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonShuffler::restrictStoreLoadOrder, end, getDesc, setUnits, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonShuffler, HexagonPacketSummary, HexagonMCInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 401-450 / 第 401-450 行

```cpp
   401:       }
   402:     }
   403: 
   404:     // A single store must use slot #0.
   405:     if (HexagonMCInstrInfo::getDesc(MCII, ID).mayStore()) {
   406:       if (!Summary.store0) {
   407:         const bool PacketHasNoOnlySlot0 =
   408:             llvm::none_of(insts(), [&](HexagonInstr const &I) {
   409:               return I.Core.getUnits() == Slot0Mask &&
   410:                      I.ID->getOpcode() != ID.getOpcode();
   411:             });
   412:         const bool SafeToMoveToSlot0 =
   413:             (Summary.loads == 0) ||
   414:             (!isMemReorderDisabled() && PacketHasNoOnlySlot0);
   415: 
   416:         if (Summary.stores == 1 && SafeToMoveToSlot0)
   417:           // Pin the store to slot #0 only if isMemReorderDisabled() == false
   418:           ISJ->Core.setUnits(ISJ->Core.getUnits() & slotSingleStore);
   419:         else if (Summary.stores >= 1) {
   420:           if (slotLoadStore < slotLastLoadStore) {
   421:             // Error if no more slots available for stores.
   422:             reportError("invalid instruction packet: too many stores");
   423:             return false;
   424:           }
   425:           // Pin the store to the highest slot available to it.
   426:           ISJ->Core.setUnits(ISJ->Core.getUnits() & slotLoadStore);
   427:           // Update the next highest slot available to stores.
   428:           slotLoadStore >>= 1;
   429:         }
   430:       }
   431:       if (Summary.store1 && Summary.stores > 1) {
   432:         // Error if a single store with another store.
   433:         reportError("invalid instruction packet: too many stores");
   434:         return false;
   435:       }
   436:     }
   437:   }
   438: 
   439:   return true;
   440: }
   441: 
   442: static std::string SlotMaskToText(unsigned SlotMask) {
   443:     SmallVector<std::string, HEXAGON_PRESHUFFLE_PACKET_SIZE> Slots;
   444:     for (unsigned SlotNum = 0; SlotNum < HEXAGON_PACKET_SIZE; SlotNum++)
   445:         if ((SlotMask & (1 << SlotNum)) != 0)
   446:             Slots.push_back(utostr(SlotNum));
   447: 
   448:     return llvm::join(Slots, StringRef(", "));
   449: }
   450: 
```
- EN: It declares or implements routines such as llvm::none_of, getUnits, isMemReorderDisabled, reportError, ... (7 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonMCInstrInfo, HexagonInstr, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 llvm::none_of, getUnits, isMemReorderDisabled, reportError, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonMCInstrInfo, HexagonInstr，说明了它与同级后端组件的连接关系。

### Lines 451-500 / 第 451-500 行

```cpp
   451: HexagonShuffler::HexagonPacketSummary HexagonShuffler::GetPacketSummary() {
   452:   HexagonPacketSummary Summary = HexagonPacketSummary();
   453: 
   454:   // Collect information from the insns in the packet.
   455:   for (iterator ISJ = begin(); ISJ != end(); ++ISJ) {
   456:     MCInst const &ID = ISJ->getDesc();
   457: 
   458:     if (HexagonMCInstrInfo::isRestrictSlot1AOK(MCII, ID))
   459:       Summary.Slot1AOKLoc = ID.getLoc();
   460:     if (HexagonMCInstrInfo::isRestrictNoSlot1Store(MCII, ID))
   461:       Summary.NoSlot1StoreLoc = ID.getLoc();
   462: 
   463:     if (HexagonMCInstrInfo::prefersSlot3(MCII, ID)) {
   464:       ++Summary.pSlot3Cnt;
   465:       Summary.PrefSlot3Inst = ISJ;
   466:     }
   467:     const unsigned ReservedSlots =
   468:         HexagonMCInstrInfo::getOtherReservedSlots(MCII, STI, ID);
   469:     Summary.ReservedSlotMask |= ReservedSlots;
   470:     if (ReservedSlots != 0)
   471:       AppliedRestrictions.push_back(std::make_pair(ID.getLoc(),
   472:                   (Twine("Instruction has reserved slots: ") +
   473:                    SlotMaskToText(ReservedSlots)).str()));
   474: 
   475:     switch (HexagonMCInstrInfo::getType(MCII, ID)) {
   476:     case HexagonII::TypeS_2op:
   477:     case HexagonII::TypeS_3op:
   478:     case HexagonII::TypeALU64:
   479:       break;
   480:     case HexagonII::TypeJ:
   481:       if (HexagonMCInstrInfo::IsABranchingInst(MCII, STI, *ISJ->ID))
   482:         Summary.branchInsts.push_back(ISJ);
   483:       break;
   484:     case HexagonII::TypeCVI_VM_VP_LDU:
   485:     case HexagonII::TypeCVI_VM_LD:
   486:     case HexagonII::TypeCVI_VM_TMP_LD:
   487:     case HexagonII::TypeCVI_GATHER:
   488:     case HexagonII::TypeCVI_GATHER_DV:
   489:     case HexagonII::TypeCVI_GATHER_RST:
   490:       ++Summary.NonZCVIloads;
   491:       [[fallthrough]];
   492:     case HexagonII::TypeCVI_ZW:
   493:       ++Summary.AllCVIloads;
   494:       [[fallthrough]];
   495:     case HexagonII::TypeLD:
   496:       ++Summary.loads;
   497:       ++Summary.memory;
   498:       if (ISJ->Core.getUnits() == slotSingleLoad ||
   499:           HexagonMCInstrInfo::getType(MCII, ID) == HexagonII::TypeCVI_VM_VP_LDU)
   500:         ++Summary.load0;
```
- EN: It declares or implements routines such as HexagonShuffler::GetPacketSummary, HexagonPacketSummary, end, getDesc, ... (5 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonShuffler, HexagonPacketSummary, HexagonMCInstrInfo, HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonShuffler::GetPacketSummary, HexagonPacketSummary, end, getDesc, ... (5 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonShuffler, HexagonPacketSummary, HexagonMCInstrInfo, HexagonII，说明了它与同级后端组件的连接关系。

### Lines 501-550 / 第 501-550 行

```cpp
   501:       if (HexagonMCInstrInfo::getDesc(MCII, ID).isReturn())
   502:         Summary.branchInsts.push_back(ISJ);
   503:       break;
   504:     case HexagonII::TypeCVI_VM_STU:
   505:     case HexagonII::TypeCVI_VM_ST:
   506:     case HexagonII::TypeCVI_VM_NEW_ST:
   507:     case HexagonII::TypeCVI_SCATTER:
   508:     case HexagonII::TypeCVI_SCATTER_DV:
   509:     case HexagonII::TypeCVI_SCATTER_RST:
   510:     case HexagonII::TypeCVI_SCATTER_NEW_RST:
   511:     case HexagonII::TypeCVI_SCATTER_NEW_ST:
   512:       ++Summary.CVIstores;
   513:       [[fallthrough]];
   514:     case HexagonII::TypeST:
   515:       ++Summary.stores;
   516:       ++Summary.memory;
   517:       if (ISJ->Core.getUnits() == slotSingleStore ||
   518:           HexagonMCInstrInfo::getType(MCII, ID) == HexagonII::TypeCVI_VM_STU)
   519:         ++Summary.store0;
   520:       break;
   521:     case HexagonII::TypeV4LDST:
   522:       ++Summary.loads;
   523:       ++Summary.stores;
   524:       ++Summary.store1;
   525:       ++Summary.memops;
   526:       ++Summary.memory;
   527:       break;
   528:     case HexagonII::TypeNCJ:
   529:       ++Summary.memory; // NV insns are memory-like.
   530:       Summary.branchInsts.push_back(ISJ);
   531:       break;
   532:     case HexagonII::TypeV2LDST:
   533:       if (HexagonMCInstrInfo::getDesc(MCII, ID).mayLoad()) {
   534:         ++Summary.loads;
   535:         ++Summary.memory;
   536:         if (ISJ->Core.getUnits() == slotSingleLoad ||
   537:             HexagonMCInstrInfo::getType(MCII, ID) ==
   538:                 HexagonII::TypeCVI_VM_VP_LDU)
   539:           ++Summary.load0;
   540:       } else {
   541:         assert(HexagonMCInstrInfo::getDesc(MCII, ID).mayStore());
   542:         ++Summary.memory;
   543:         ++Summary.stores;
   544:       }
   545:       break;
   546:     case HexagonII::TypeCR:
   547:     // Legacy conditional branch predicated on a register.
   548:     case HexagonII::TypeCJ:
   549:       if (HexagonMCInstrInfo::getDesc(MCII, ID).isBranch())
   550:         Summary.branchInsts.push_back(ISJ);
```
- EN: It declares or implements routines such as push_back, assert, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonMCInstrInfo, HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 push_back, assert 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonMCInstrInfo, HexagonII，说明了它与同级后端组件的连接关系。

### Lines 551-600 / 第 551-600 行

```cpp
   551:       break;
   552:     case HexagonII::TypeDUPLEX: {
   553:       ++Summary.duplex;
   554:       MCInst const &Inst0 = *ID.getOperand(0).getInst();
   555:       MCInst const &Inst1 = *ID.getOperand(1).getInst();
   556:       if (HexagonMCInstrInfo::getDesc(MCII, Inst0).isBranch())
   557:         Summary.branchInsts.push_back(ISJ);
   558:       if (HexagonMCInstrInfo::getDesc(MCII, Inst1).isBranch())
   559:         Summary.branchInsts.push_back(ISJ);
   560:       if (HexagonMCInstrInfo::getDesc(MCII, Inst0).isReturn())
   561:         Summary.branchInsts.push_back(ISJ);
   562:       if (HexagonMCInstrInfo::getDesc(MCII, Inst1).isReturn())
   563:         Summary.branchInsts.push_back(ISJ);
   564:       break;
   565:     }
   566:     }
   567:   }
   568:   return Summary;
   569: }
   570: 
   571: bool HexagonShuffler::ValidPacketMemoryOps(
   572:     HexagonPacketSummary const &Summary) const {
   573:   // Check if the packet is legal.
   574:   const unsigned ZCVIloads = Summary.AllCVIloads - Summary.NonZCVIloads;
   575:   const bool ValidHVXMem =
   576:       Summary.NonZCVIloads <= 1 && ZCVIloads <= 1 && Summary.CVIstores <= 1;
   577:   const bool InvalidPacket =
   578:       ((Summary.load0 > 1 || Summary.store0 > 1 || !ValidHVXMem) ||
   579:        (Summary.duplex > 1 || (Summary.duplex && Summary.memory)));
   580: 
   581:   return !InvalidPacket;
   582: }
   583: 
   584: void HexagonShuffler::restrictPreferSlot3(HexagonPacketSummary const &Summary,
   585:                                           const bool DoShuffle) {
   586:   // flag if an instruction requires to be in slot 3
   587:   const bool HasOnlySlot3 = llvm::any_of(insts(), [&](HexagonInstr const &I) {
   588:     return (I.Core.getUnits() == Slot3Mask);
   589:   });
   590:   const bool NeedsPrefSlot3Shuffle = Summary.branchInsts.size() <= 1 &&
   591:                                      !HasOnlySlot3 && Summary.pSlot3Cnt == 1 &&
   592:                                      Summary.PrefSlot3Inst && DoShuffle;
   593: 
   594:   if (!NeedsPrefSlot3Shuffle)
   595:     return;
   596: 
   597:   HexagonInstr *PrefSlot3Inst = *Summary.PrefSlot3Inst;
   598:   // save off slot mask of instruction marked with A_PREFER_SLOT3
   599:   // and then pin it to slot #3
   600:   const unsigned saveUnits = PrefSlot3Inst->Core.getUnits();
```
- EN: It declares or implements routines such as getOperand, HexagonShuffler::ValidPacketMemoryOps, HexagonShuffler::restrictPreferSlot3, llvm::any_of, ... (5 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonII, HexagonMCInstrInfo, HexagonShuffler, HexagonPacketSummary, ... (5 total), showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getOperand, HexagonShuffler::ValidPacketMemoryOps, HexagonShuffler::restrictPreferSlot3, llvm::any_of, ... (5 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonII, HexagonMCInstrInfo, HexagonShuffler, HexagonPacketSummary, ... (5 total)，说明了它与同级后端组件的连接关系。

### Lines 601-650 / 第 601-650 行

```cpp
   601:   PrefSlot3Inst->Core.setUnits(saveUnits & Slot3Mask);
   602:   const bool HasShuffledPacket = tryAuction(Summary).has_value();
   603:   if (HasShuffledPacket)
   604:     return;
   605: 
   606:   PrefSlot3Inst->Core.setUnits(saveUnits);
   607: }
   608: 
   609: /// Check that the packet is legal and enforce relative insn order.
   610: bool HexagonShuffler::check(const bool RequireShuffle) {
   611:   const HexagonPacketSummary Summary = GetPacketSummary();
   612:   if (!applySlotRestrictions(Summary, RequireShuffle))
   613:     return false;
   614: 
   615:   if (!ValidPacketMemoryOps(Summary)) {
   616:     reportError("invalid instruction packet");
   617:     return false;
   618:   }
   619: 
   620:   if (RequireShuffle)
   621:     ValidResourceUsage(Summary);
   622: 
   623:   return !CheckFailure;
   624: }
   625: 
   626: std::optional<HexagonShuffler::HexagonPacket>
   627: HexagonShuffler::tryAuction(HexagonPacketSummary const &Summary) {
   628:   HexagonPacket PacketResult = Packet;
   629:   HexagonUnitAuction AuctionCore(Summary.ReservedSlotMask);
   630:   llvm::stable_sort(PacketResult, HexagonInstr::lessCore);
   631: 
   632:   const bool ValidSlots =
   633:       llvm::all_of(insts(PacketResult), [&AuctionCore](HexagonInstr const &I) {
   634:         return AuctionCore.bid(I.Core.getUnits());
   635:       });
   636: 
   637:   LLVM_DEBUG(
   638:     dbgs() << "Shuffle attempt: " << (ValidSlots ? "passed" : "failed")
   639:            << "\n";
   640:     for (HexagonInstr const &ISJ : insts(PacketResult))
   641:       dbgs() << "\t" << HexagonMCInstrInfo::getName(MCII, *ISJ.ID) << ": "
   642:              << llvm::format_hex(ISJ.Core.getUnits(), 4, true) << "\n";
   643:   );
   644: 
   645:   std::optional<HexagonPacket> Res;
   646:   if (ValidSlots)
   647:     Res = PacketResult;
   648: 
   649:   return Res;
   650: }
```
- EN: It declares or implements routines such as setUnits, tryAuction, HexagonShuffler::check, GetPacketSummary, ... (10 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonShuffler, HexagonPacketSummary, HexagonPacket, HexagonUnitAuction, ... (6 total), showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 setUnits, tryAuction, HexagonShuffler::check, GetPacketSummary, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonShuffler, HexagonPacketSummary, HexagonPacket, HexagonUnitAuction, ... (6 total)，说明了它与同级后端组件的连接关系。

### Lines 651-700 / 第 651-700 行

```cpp
   651: 
   652: bool HexagonShuffler::shuffle() {
   653:   if (size() > HEXAGON_PACKET_SIZE) {
   654:     // Ignore a packet with more than what a packet can hold
   655:     // or with compound or duplex insns for now.
   656:     reportError("invalid instruction packet");
   657:     return false;
   658:   }
   659: 
   660:   // Check and prepare packet.
   661:   bool Ok = check();
   662:   if (size() > 1 && Ok)
   663:     // Reorder the handles for each slot.
   664:     for (unsigned nSlot = 0, emptySlots = 0; nSlot < HEXAGON_PACKET_SIZE;
   665:          ++nSlot) {
   666:       iterator ISJ, ISK;
   667:       unsigned slotSkip, slotWeight;
   668: 
   669:       // Prioritize the handles considering their restrictions.
   670:       for (ISJ = ISK = Packet.begin(), slotSkip = slotWeight = 0;
   671:            ISK != Packet.end(); ++ISK, ++slotSkip)
   672:         if (slotSkip < nSlot - emptySlots)
   673:           // Note which handle to begin at.
   674:           ++ISJ;
   675:         else
   676:           // Calculate the weight of the slot.
   677:           slotWeight += ISK->Core.setWeight(HEXAGON_PACKET_SIZE - nSlot - 1);
   678: 
   679:       if (slotWeight)
   680:         // Sort the packet, favoring source order,
   681:         // beginning after the previous slot.
   682:         std::stable_sort(ISJ, Packet.end());
   683:       else
   684:         // Skip unused slot.
   685:         ++emptySlots;
   686:     }
   687: 
   688:   LLVM_DEBUG(
   689:     for (HexagonInstr const &ISJ : insts()) {
   690:       dbgs().write_hex(ISJ.Core.getUnits());
   691:       if (ISJ.CVI.isValid()) {
   692:         dbgs() << '/';
   693:         dbgs().write_hex(ISJ.CVI.getUnits()) << '|';
   694:         dbgs() << ISJ.CVI.getLanes();
   695:       }
   696:       dbgs() << ':'
   697:              << HexagonMCInstrInfo::getDesc(MCII, ISJ.getDesc()).getOpcode()
   698:              << '\n';
   699:     } dbgs() << '\n';
   700:   );
```
- EN: It declares or implements routines such as HexagonShuffler::shuffle, reportError, check, end, ... (7 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonShuffler, HexagonInstr, HexagonMCInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonShuffler::shuffle, reportError, check, end, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonShuffler, HexagonInstr, HexagonMCInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 701-741 / 第 701-741 行

```cpp
   701: 
   702:   return Ok;
   703: }
   704: 
   705: void HexagonShuffler::reportResourceError(HexagonPacketSummary const &Summary, StringRef Err) {
   706:   if (ReportErrors)
   707:     reportResourceUsage(Summary);
   708:   reportError(Twine("invalid instruction packet: ") + Err);
   709: }
   710: 
   711: 
   712: void HexagonShuffler::reportResourceUsage(HexagonPacketSummary const &Summary) {
   713:   auto SM = Context.getSourceManager();
   714:   if (SM) {
   715:     for (HexagonInstr const &I : insts()) {
   716:       const unsigned Units = I.Core.getUnits();
   717: 
   718:       if (HexagonMCInstrInfo::requiresSlot(STI, *I.ID)) {
   719:         const std::string UnitsText = Units ? SlotMaskToText(Units) : "<None>";
   720:         SM->PrintMessage(I.ID->getLoc(), SourceMgr::DK_Note,
   721:                 Twine("Instruction can utilize slots: ") +
   722:                 UnitsText);
   723:       }
   724:       else if (!HexagonMCInstrInfo::isImmext(*I.ID))
   725:         SM->PrintMessage(I.ID->getLoc(), SourceMgr::DK_Note,
   726:                        "Instruction does not require a slot");
   727:     }
   728:   }
   729: }
   730: 
   731: void HexagonShuffler::reportError(Twine const &Msg) {
   732:   CheckFailure = true;
   733:   if (ReportErrors) {
   734:     for (auto const &I : AppliedRestrictions) {
   735:       auto SM = Context.getSourceManager();
   736:       if (SM)
   737:         SM->PrintMessage(I.first, SourceMgr::DK_Note, I.second);
   738:     }
   739:     Context.reportError(Loc, Msg);
   740:   }
   741: }
```
- EN: It declares or implements routines such as HexagonShuffler::reportResourceError, reportError, HexagonShuffler::reportResourceUsage, getSourceManager, ... (7 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonShuffler, HexagonPacketSummary, HexagonInstr, HexagonMCInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonShuffler::reportResourceError, reportError, HexagonShuffler::reportResourceUsage, getSourceManager, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonShuffler, HexagonPacketSummary, HexagonInstr, HexagonMCInstrInfo，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- HVX vector ISA modeling / HVX 向量 ISA 建模
- instruction semantics / 指令语义
- MC-layer target description / MC 层目标描述
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `MCTargetDesc/HexagonShuffler.h, MCTargetDesc/HexagonBaseInfo.h, MCTargetDesc/HexagonMCInstrInfo.h, MCTargetDesc/HexagonMCTargetDesc.h, llvm/ADT/SmallVector.h, llvm/ADT/StringExtras.h, llvm/ADT/Twine.h, llvm/MC/MCContext.h, llvm/MC/MCInst.h, llvm/MC/MCInstrDesc.h, ... (19 total)`
- Hexagon symbols / Hexagon 符号: `HexagonShuffler, HexagonBaseInfo, HexagonMCInstrInfo, HexagonMCTargetDesc, HexagonBid, HexagonUnitAuction, HexagonResource, HexagonCVIResource, HexagonConvertUnits, HexagonInstr, ... (13 total)`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
