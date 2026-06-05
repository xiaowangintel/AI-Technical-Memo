# HexagonBlockRanges.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonBlockRanges.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): If A contains start(), or "this" contains A.start(), then overlap.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```cpp
     1: //===- HexagonBlockRanges.cpp ---------------------------------------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: 
     9: #include "HexagonBlockRanges.h"
    10: #include "HexagonInstrInfo.h"
    11: #include "HexagonSubtarget.h"
    12: #include "llvm/ADT/BitVector.h"
    13: #include "llvm/ADT/STLExtras.h"
    14: #include "llvm/CodeGen/MachineBasicBlock.h"
    15: #include "llvm/CodeGen/MachineFunction.h"
    16: #include "llvm/CodeGen/MachineInstr.h"
    17: #include "llvm/CodeGen/MachineOperand.h"
    18: #include "llvm/CodeGen/MachineRegisterInfo.h"
    19: #include "llvm/CodeGen/TargetRegisterInfo.h"
    20: #include "llvm/MC/MCRegisterInfo.h"
    21: #include "llvm/Support/Debug.h"
    22: #include "llvm/Support/raw_ostream.h"
    23: #include <cassert>
    24: #include <cstdint>
    25: #include <iterator>
    26: #include <map>
    27: 
    28: using namespace llvm;
    29: 
    30: #define DEBUG_TYPE "hbr"
    31: 
    32: bool HexagonBlockRanges::IndexRange::overlaps(const IndexRange &A) const {
    33:   // If A contains start(), or "this" contains A.start(), then overlap.
    34:   IndexType S = start(), E = end(), AS = A.start(), AE = A.end();
    35:   if (AS == S)
    36:     return true;
    37:   bool SbAE = (S < AE) || (S == AE && A.TiedEnd);  // S-before-AE.
    38:   bool ASbE = (AS < E) || (AS == E && TiedEnd);    // AS-before-E.
    39:   if ((AS < S && SbAE) || (S < AS && ASbE))
    40:     return true;
    41:   // Otherwise no overlap.
    42:   return false;
    43: }
    44: 
    45: bool HexagonBlockRanges::IndexRange::contains(const IndexRange &A) const {
    46:   if (start() <= A.start()) {
    47:     // Treat "None" in the range end as equal to the range start.
    48:     IndexType E = (end() != IndexType::None) ? end() : start();
    49:     IndexType AE = (A.end() != IndexType::None) ? A.end() : A.start();
    50:     if (AE <= E)
```
- EN: It imports headers such as HexagonBlockRanges.h, HexagonInstrInfo.h, HexagonSubtarget.h, llvm/ADT/BitVector.h, ... (18 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as HexagonBlockRanges::IndexRange::overlaps, start, HexagonBlockRanges::IndexRange::contains, end, translating Hexagon-specific policy into reusable code paths.
- CN: 这里引入了 HexagonBlockRanges.h, HexagonInstrInfo.h, HexagonSubtarget.h, llvm/ADT/BitVector.h, ... (18 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 HexagonBlockRanges::IndexRange::overlaps, start, HexagonBlockRanges::IndexRange::contains, end 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 51-100 / 第 51-100 行

```cpp
    51:       return true;
    52:   }
    53:   return false;
    54: }
    55: 
    56: void HexagonBlockRanges::IndexRange::merge(const IndexRange &A) {
    57:   // Allow merging adjacent ranges.
    58:   assert(end() == A.start() || overlaps(A));
    59:   IndexType AS = A.start(), AE = A.end();
    60:   if (AS < start() || start() == IndexType::None)
    61:     setStart(AS);
    62:   if (end() < AE || end() == IndexType::None) {
    63:     setEnd(AE);
    64:     TiedEnd = A.TiedEnd;
    65:   } else {
    66:     if (end() == AE)
    67:       TiedEnd |= A.TiedEnd;
    68:   }
    69:   if (A.Fixed)
    70:     Fixed = true;
    71: }
    72: 
    73: void HexagonBlockRanges::RangeList::include(const RangeList &RL) {
    74:   for (const auto &R : RL)
    75:     if (!is_contained(*this, R))
    76:       push_back(R);
    77: }
    78: 
    79: // Merge all overlapping ranges in the list, so that all that remains
    80: // is a list of disjoint ranges.
    81: void HexagonBlockRanges::RangeList::unionize(bool MergeAdjacent) {
    82:   if (empty())
    83:     return;
    84: 
    85:   llvm::sort(*this);
    86:   iterator Iter = begin();
    87: 
    88:   while (Iter != end()-1) {
    89:     iterator Next = std::next(Iter);
    90:     // If MergeAdjacent is true, merge ranges A and B, where A.end == B.start.
    91:     // This allows merging dead ranges, but is not valid for live ranges.
    92:     bool Merge = MergeAdjacent && (Iter->end() == Next->start());
    93:     if (Merge || Iter->overlaps(*Next)) {
    94:       Iter->merge(*Next);
    95:       erase(Next);
    96:       continue;
    97:     }
    98:     ++Iter;
    99:   }
   100: }
```
- EN: It declares or implements routines such as HexagonBlockRanges::IndexRange::merge, assert, start, setEnd, ... (12 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonBlockRanges, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonBlockRanges::IndexRange::merge, assert, start, setEnd, ... (12 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonBlockRanges，说明了它与同级后端组件的连接关系。

### Lines 101-150 / 第 101-150 行

```cpp
   101: 
   102: // Compute a range A-B and add it to the list.
   103: void HexagonBlockRanges::RangeList::addsub(const IndexRange &A,
   104:       const IndexRange &B) {
   105:   // Exclusion of non-overlapping ranges makes some checks simpler
   106:   // later in this function.
   107:   if (!A.overlaps(B)) {
   108:     // A - B = A.
   109:     add(A);
   110:     return;
   111:   }
   112: 
   113:   IndexType AS = A.start(), AE = A.end();
   114:   IndexType BS = B.start(), BE = B.end();
   115: 
   116:   // If AE is None, then A is included in B, since A and B overlap.
   117:   // The result of subtraction if empty, so just return.
   118:   if (AE == IndexType::None)
   119:     return;
   120: 
   121:   if (AS < BS) {
   122:     // A starts before B.
   123:     // AE cannot be None since A and B overlap.
   124:     assert(AE != IndexType::None);
   125:     // Add the part of A that extends on the "less" side of B.
   126:     add(AS, BS, A.Fixed, false);
   127:   }
   128: 
   129:   if (BE < AE) {
   130:     // BE cannot be Exit here.
   131:     if (BE == IndexType::None)
   132:       add(BS, AE, A.Fixed, false);
   133:     else
   134:       add(BE, AE, A.Fixed, false);
   135:   }
   136: }
   137: 
   138: // Subtract a given range from each element in the list.
   139: void HexagonBlockRanges::RangeList::subtract(const IndexRange &Range) {
   140:   // Cannot assume that the list is unionized (i.e. contains only non-
   141:   // overlapping ranges.
   142:   RangeList T;
   143:   for (iterator Next, I = begin(); I != end(); I = Next) {
   144:     IndexRange &Rg = *I;
   145:     if (Rg.overlaps(Range)) {
   146:       T.addsub(Rg, Range);
   147:       Next = this->erase(I);
   148:     } else {
   149:       Next = std::next(I);
   150:     }
```
- EN: It declares or implements routines such as HexagonBlockRanges::RangeList::addsub, add, start, assert, ... (9 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonBlockRanges, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonBlockRanges::RangeList::addsub, add, start, assert, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonBlockRanges，说明了它与同级后端组件的连接关系。

### Lines 151-200 / 第 151-200 行

```cpp
   151:   }
   152:   include(T);
   153: }
   154: 
   155: HexagonBlockRanges::InstrIndexMap::InstrIndexMap(MachineBasicBlock &B)
   156:     : Block(B) {
   157:   IndexType Idx = IndexType::First;
   158:   First = Idx;
   159:   for (auto &In : B) {
   160:     if (In.isDebugInstr())
   161:       continue;
   162:     assert(getIndex(&In) == IndexType::None && "Instruction already in map");
   163:     Map.insert(std::make_pair(Idx, &In));
   164:     ++Idx;
   165:   }
   166:   Last = B.empty() ? IndexType::None : unsigned(Idx)-1;
   167: }
   168: 
   169: MachineInstr *HexagonBlockRanges::InstrIndexMap::getInstr(IndexType Idx) const {
   170:   auto F = Map.find(Idx);
   171:   return (F != Map.end()) ? F->second : nullptr;
   172: }
   173: 
   174: HexagonBlockRanges::IndexType HexagonBlockRanges::InstrIndexMap::getIndex(
   175:       MachineInstr *MI) const {
   176:   for (const auto &I : Map)
   177:     if (I.second == MI)
   178:       return I.first;
   179:   return IndexType::None;
   180: }
   181: 
   182: HexagonBlockRanges::IndexType HexagonBlockRanges::InstrIndexMap::getPrevIndex(
   183:       IndexType Idx) const {
   184:   assert (Idx != IndexType::None);
   185:   if (Idx == IndexType::Entry)
   186:     return IndexType::None;
   187:   if (Idx == IndexType::Exit)
   188:     return Last;
   189:   if (Idx == First)
   190:     return IndexType::Entry;
   191:   return unsigned(Idx)-1;
   192: }
   193: 
   194: HexagonBlockRanges::IndexType HexagonBlockRanges::InstrIndexMap::getNextIndex(
   195:       IndexType Idx) const {
   196:   assert (Idx != IndexType::None);
   197:   if (Idx == IndexType::Entry)
   198:     return IndexType::First;
   199:   if (Idx == IndexType::Exit || Idx == Last)
   200:     return IndexType::None;
```
- EN: It declares or implements routines such as include, HexagonBlockRanges::InstrIndexMap::InstrIndexMap, assert, insert, ... (9 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonBlockRanges, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 include, HexagonBlockRanges::InstrIndexMap::InstrIndexMap, assert, insert, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonBlockRanges，说明了它与同级后端组件的连接关系。

### Lines 201-250 / 第 201-250 行

```cpp
   201:   return unsigned(Idx)+1;
   202: }
   203: 
   204: void HexagonBlockRanges::InstrIndexMap::replaceInstr(MachineInstr *OldMI,
   205:       MachineInstr *NewMI) {
   206:   for (auto &I : Map) {
   207:     if (I.second != OldMI)
   208:       continue;
   209:     if (NewMI != nullptr)
   210:       I.second = NewMI;
   211:     else
   212:       Map.erase(I.first);
   213:     break;
   214:   }
   215: }
   216: 
   217: HexagonBlockRanges::HexagonBlockRanges(MachineFunction &mf)
   218:   : MF(mf), HST(mf.getSubtarget<HexagonSubtarget>()),
   219:     TII(*HST.getInstrInfo()), TRI(*HST.getRegisterInfo()),
   220:     Reserved(TRI.getReservedRegs(mf)) {
   221:   // Consider all non-allocatable registers as reserved.
   222:   for (const TargetRegisterClass *RC : TRI.regclasses()) {
   223:     if (RC->isAllocatable())
   224:       continue;
   225:     for (unsigned R : *RC)
   226:       Reserved[R] = true;
   227:   }
   228: }
   229: 
   230: HexagonBlockRanges::RegisterSet HexagonBlockRanges::getLiveIns(
   231:       const MachineBasicBlock &B, const MachineRegisterInfo &MRI,
   232:       const TargetRegisterInfo &TRI) {
   233:   RegisterSet LiveIns;
   234:   RegisterSet Tmp;
   235: 
   236:   for (auto I : B.liveins()) {
   237:     MCSubRegIndexIterator S(I.PhysReg, &TRI);
   238:     if (I.LaneMask.all() || (I.LaneMask.any() && !S.isValid())) {
   239:       Tmp.insert({I.PhysReg, 0});
   240:       continue;
   241:     }
   242:     for (; S.isValid(); ++S) {
   243:       unsigned SI = S.getSubRegIndex();
   244:       if ((I.LaneMask & TRI.getSubRegIndexLaneMask(SI)).any())
   245:         Tmp.insert({S.getSubReg(), 0});
   246:     }
   247:   }
   248: 
   249:   for (auto R : Tmp) {
   250:     if (!Reserved[R.Reg])
```
- EN: It declares or implements routines such as HexagonBlockRanges::InstrIndexMap::replaceInstr, erase, HexagonBlockRanges::HexagonBlockRanges, HexagonBlockRanges::getLiveIns, ... (7 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonBlockRanges, HexagonSubtarget, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonBlockRanges::InstrIndexMap::replaceInstr, erase, HexagonBlockRanges::HexagonBlockRanges, HexagonBlockRanges::getLiveIns, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonBlockRanges, HexagonSubtarget，说明了它与同级后端组件的连接关系。

### Lines 251-300 / 第 251-300 行

```cpp
   251:       LiveIns.insert(R);
   252:     for (auto S : expandToSubRegs(R, MRI, TRI))
   253:       if (!Reserved[S.Reg])
   254:         LiveIns.insert(S);
   255:   }
   256:   return LiveIns;
   257: }
   258: 
   259: HexagonBlockRanges::RegisterSet HexagonBlockRanges::expandToSubRegs(
   260:       RegisterRef R, const MachineRegisterInfo &MRI,
   261:       const TargetRegisterInfo &TRI) {
   262:   RegisterSet SRs;
   263: 
   264:   if (R.Sub != 0) {
   265:     SRs.insert(R);
   266:     return SRs;
   267:   }
   268: 
   269:   if (R.Reg.isPhysical()) {
   270:     if (TRI.subregs(R.Reg).empty())
   271:       SRs.insert({R.Reg, 0});
   272:     for (MCPhysReg I : TRI.subregs(R.Reg))
   273:       SRs.insert({I, 0});
   274:   } else {
   275:     assert(R.Reg.isVirtual());
   276:     auto &RC = *MRI.getRegClass(R.Reg);
   277:     unsigned PReg = *RC.begin();
   278:     MCSubRegIndexIterator I(PReg, &TRI);
   279:     if (!I.isValid())
   280:       SRs.insert({R.Reg, 0});
   281:     for (; I.isValid(); ++I)
   282:       SRs.insert({R.Reg, I.getSubRegIndex()});
   283:   }
   284:   return SRs;
   285: }
   286: 
   287: void HexagonBlockRanges::computeInitialLiveRanges(InstrIndexMap &IndexMap,
   288:       RegToRangeMap &LiveMap) {
   289:   std::map<RegisterRef,IndexType> LastDef, LastUse;
   290:   RegisterSet LiveOnEntry;
   291:   MachineBasicBlock &B = IndexMap.getBlock();
   292:   MachineRegisterInfo &MRI = B.getParent()->getRegInfo();
   293: 
   294:   for (auto R : getLiveIns(B, MRI, TRI))
   295:     LiveOnEntry.insert(R);
   296: 
   297:   for (auto R : LiveOnEntry)
   298:     LastDef[R] = IndexType::Entry;
   299: 
   300:   auto closeRange = [&LastUse,&LastDef,&LiveMap] (RegisterRef R) -> void {
```
- EN: It declares or implements routines such as insert, HexagonBlockRanges::expandToSubRegs, assert, getRegClass, ... (10 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonBlockRanges, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 insert, HexagonBlockRanges::expandToSubRegs, assert, getRegClass, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonBlockRanges，说明了它与同级后端组件的连接关系。

### Lines 301-350 / 第 301-350 行

```cpp
   301:     auto LD = LastDef[R], LU = LastUse[R];
   302:     if (LD == IndexType::None)
   303:       LD = IndexType::Entry;
   304:     if (LU == IndexType::None)
   305:       LU = IndexType::Exit;
   306:     LiveMap[R].add(LD, LU, false, false);
   307:     LastUse[R] = LastDef[R] = IndexType::None;
   308:   };
   309: 
   310:   RegisterSet Defs, Clobbers;
   311: 
   312:   for (auto &In : B) {
   313:     if (In.isDebugInstr())
   314:       continue;
   315:     IndexType Index = IndexMap.getIndex(&In);
   316:     // Process uses first.
   317:     for (auto &Op : In.operands()) {
   318:       if (!Op.isReg() || !Op.isUse() || Op.isUndef())
   319:         continue;
   320:       RegisterRef R = { Op.getReg(), Op.getSubReg() };
   321:       if (R.Reg.isPhysical() && Reserved[R.Reg])
   322:         continue;
   323:       bool IsKill = Op.isKill();
   324:       for (auto S : expandToSubRegs(R, MRI, TRI)) {
   325:         LastUse[S] = Index;
   326:         if (IsKill)
   327:           closeRange(S);
   328:       }
   329:     }
   330:     // Process defs and clobbers.
   331:     Defs.clear();
   332:     Clobbers.clear();
   333:     for (auto &Op : In.operands()) {
   334:       if (!Op.isReg() || !Op.isDef() || Op.isUndef())
   335:         continue;
   336:       RegisterRef R = { Op.getReg(), Op.getSubReg() };
   337:       for (auto S : expandToSubRegs(R, MRI, TRI)) {
   338:         if (S.Reg.isPhysical() && Reserved[S.Reg])
   339:           continue;
   340:         if (Op.isDead())
   341:           Clobbers.insert(S);
   342:         else
   343:           Defs.insert(S);
   344:       }
   345:     }
   346: 
   347:     for (auto &Op : In.operands()) {
   348:       if (!Op.isRegMask())
   349:         continue;
   350:       const uint32_t *BM = Op.getRegMask();
```
- EN: It declares or implements routines such as add, getIndex, isKill, clear, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 add, getIndex, isKill, clear, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 351-400 / 第 351-400 行

```cpp
   351:       for (unsigned PR = 1, N = TRI.getNumRegs(); PR != N; ++PR) {
   352:         // Skip registers that have subregisters. A register is preserved
   353:         // iff its bit is set in the regmask, so if R1:0 was preserved, both
   354:         // R1 and R0 would also be present.
   355:         if (!TRI.subregs(PR).empty())
   356:           continue;
   357:         if (Reserved[PR])
   358:           continue;
   359:         if (BM[PR/32] & (1u << (PR%32)))
   360:           continue;
   361:         RegisterRef R = { PR, 0 };
   362:         if (!Defs.count(R))
   363:           Clobbers.insert(R);
   364:       }
   365:     }
   366:     // Defs and clobbers can overlap, e.g.
   367:     // dead %d0 = COPY %5, implicit-def %r0, implicit-def %r1
   368:     for (RegisterRef R : Defs)
   369:       Clobbers.erase(R);
   370: 
   371:     // Update maps for defs.
   372:     for (RegisterRef S : Defs) {
   373:       // Defs should already be expanded into subregs.
   374:       assert(!S.Reg.isPhysical() || TRI.subregs(S.Reg).empty());
   375:       if (LastDef[S] != IndexType::None || LastUse[S] != IndexType::None)
   376:         closeRange(S);
   377:       LastDef[S] = Index;
   378:     }
   379:     // Update maps for clobbers.
   380:     for (RegisterRef S : Clobbers) {
   381:       // Clobbers should already be expanded into subregs.
   382:       assert(!S.Reg.isPhysical() || TRI.subregs(S.Reg).empty());
   383:       if (LastDef[S] != IndexType::None || LastUse[S] != IndexType::None)
   384:         closeRange(S);
   385:       // Create a single-instruction range.
   386:       LastDef[S] = LastUse[S] = Index;
   387:       closeRange(S);
   388:     }
   389:   }
   390: 
   391:   // Collect live-on-exit.
   392:   RegisterSet LiveOnExit;
   393:   for (auto *SB : B.successors())
   394:     for (auto R : getLiveIns(*SB, MRI, TRI))
   395:       LiveOnExit.insert(R);
   396: 
   397:   for (auto R : LiveOnExit)
   398:     LastUse[R] = IndexType::Exit;
   399: 
   400:   // Process remaining registers.
```
- EN: It declares or implements routines such as assert, closeRange, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 assert, closeRange 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 401-450 / 第 401-450 行

```cpp
   401:   RegisterSet Left;
   402:   for (auto &I : LastUse)
   403:     if (I.second != IndexType::None)
   404:       Left.insert(I.first);
   405:   for (auto &I : LastDef)
   406:     if (I.second != IndexType::None)
   407:       Left.insert(I.first);
   408:   for (auto R : Left)
   409:     closeRange(R);
   410: 
   411:   // Finalize the live ranges.
   412:   for (auto &P : LiveMap)
   413:     P.second.unionize();
   414: }
   415: 
   416: HexagonBlockRanges::RegToRangeMap HexagonBlockRanges::computeLiveMap(
   417:       InstrIndexMap &IndexMap) {
   418:   RegToRangeMap LiveMap;
   419:   LLVM_DEBUG(dbgs() << __func__ << ": index map\n" << IndexMap << '\n');
   420:   computeInitialLiveRanges(IndexMap, LiveMap);
   421:   LLVM_DEBUG(dbgs() << __func__ << ": live map\n"
   422:                     << PrintRangeMap(LiveMap, TRI) << '\n');
   423:   return LiveMap;
   424: }
   425: 
   426: HexagonBlockRanges::RegToRangeMap HexagonBlockRanges::computeDeadMap(
   427:       InstrIndexMap &IndexMap, RegToRangeMap &LiveMap) {
   428:   RegToRangeMap DeadMap;
   429: 
   430:   auto addDeadRanges = [&IndexMap,&LiveMap,&DeadMap] (RegisterRef R) -> void {
   431:     auto F = LiveMap.find(R);
   432:     if (F == LiveMap.end() || F->second.empty()) {
   433:       DeadMap[R].add(IndexType::Entry, IndexType::Exit, false, false);
   434:       return;
   435:     }
   436: 
   437:     RangeList &RL = F->second;
   438:     RangeList::iterator A = RL.begin(), Z = RL.end()-1;
   439: 
   440:     // Try to create the initial range.
   441:     if (A->start() != IndexType::Entry) {
   442:       IndexType DE = IndexMap.getPrevIndex(A->start());
   443:       if (DE != IndexType::Entry)
   444:         DeadMap[R].add(IndexType::Entry, DE, false, false);
   445:     }
   446: 
   447:     while (A != Z) {
   448:       // Creating a dead range that follows A.  Pay attention to empty
   449:       // ranges (i.e. those ending with "None").
   450:       IndexType AE = (A->end() == IndexType::None) ? A->start() : A->end();
```
- EN: It declares or implements routines such as HexagonBlockRanges::computeLiveMap, LLVM_DEBUG, computeInitialLiveRanges, HexagonBlockRanges::computeDeadMap, ... (8 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonBlockRanges, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonBlockRanges::computeLiveMap, LLVM_DEBUG, computeInitialLiveRanges, HexagonBlockRanges::computeDeadMap, ... (8 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonBlockRanges，说明了它与同级后端组件的连接关系。

### Lines 451-500 / 第 451-500 行

```cpp
   451:       IndexType DS = IndexMap.getNextIndex(AE);
   452:       ++A;
   453:       IndexType DE = IndexMap.getPrevIndex(A->start());
   454:       if (DS < DE)
   455:         DeadMap[R].add(DS, DE, false, false);
   456:     }
   457: 
   458:     // Try to create the final range.
   459:     if (Z->end() != IndexType::Exit) {
   460:       IndexType ZE = (Z->end() == IndexType::None) ? Z->start() : Z->end();
   461:       IndexType DS = IndexMap.getNextIndex(ZE);
   462:       if (DS < IndexType::Exit)
   463:         DeadMap[R].add(DS, IndexType::Exit, false, false);
   464:     }
   465:   };
   466: 
   467:   MachineFunction &MF = *IndexMap.getBlock().getParent();
   468:   auto &MRI = MF.getRegInfo();
   469:   unsigned NumRegs = TRI.getNumRegs();
   470:   BitVector Visited(NumRegs);
   471:   for (unsigned R = 1; R < NumRegs; ++R) {
   472:     for (auto S : expandToSubRegs({R,0}, MRI, TRI)) {
   473:       if (Reserved[S.Reg] || Visited[S.Reg])
   474:         continue;
   475:       addDeadRanges(S);
   476:       Visited[S.Reg] = true;
   477:     }
   478:   }
   479:   for (auto &P : LiveMap)
   480:     if (P.first.Reg.isVirtual())
   481:       addDeadRanges(P.first);
   482: 
   483:   LLVM_DEBUG(dbgs() << __func__ << ": dead map\n"
   484:                     << PrintRangeMap(DeadMap, TRI) << '\n');
   485:   return DeadMap;
   486: }
   487: 
   488: raw_ostream &llvm::operator<<(raw_ostream &OS,
   489:                               HexagonBlockRanges::IndexType Idx) {
   490:   if (Idx == HexagonBlockRanges::IndexType::None)
   491:     return OS << '-';
   492:   if (Idx == HexagonBlockRanges::IndexType::Entry)
   493:     return OS << 'n';
   494:   if (Idx == HexagonBlockRanges::IndexType::Exit)
   495:     return OS << 'x';
   496:   return OS << unsigned(Idx)-HexagonBlockRanges::IndexType::First+1;
   497: }
   498: 
   499: // A mapping to translate between instructions and their indices.
   500: raw_ostream &llvm::operator<<(raw_ostream &OS,
```
- EN: It declares or implements routines such as getNextIndex, getPrevIndex, end, getBlock, ... (9 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonBlockRanges, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getNextIndex, getPrevIndex, end, getBlock, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonBlockRanges，说明了它与同级后端组件的连接关系。

### Lines 501-531 / 第 501-531 行

```cpp
   501:                               const HexagonBlockRanges::IndexRange &IR) {
   502:   OS << '[' << IR.start() << ':' << IR.end() << (IR.TiedEnd ? '}' : ']');
   503:   if (IR.Fixed)
   504:     OS << '!';
   505:   return OS;
   506: }
   507: 
   508: raw_ostream &llvm::operator<<(raw_ostream &OS,
   509:                               const HexagonBlockRanges::RangeList &RL) {
   510:   for (const auto &R : RL)
   511:     OS << R << " ";
   512:   return OS;
   513: }
   514: 
   515: raw_ostream &llvm::operator<<(raw_ostream &OS,
   516:                               const HexagonBlockRanges::InstrIndexMap &M) {
   517:   for (auto &In : M.Block) {
   518:     HexagonBlockRanges::IndexType Idx = M.getIndex(&In);
   519:     OS << Idx << (Idx == M.Last ? ". " : "  ") << In;
   520:   }
   521:   return OS;
   522: }
   523: 
   524: raw_ostream &llvm::operator<<(raw_ostream &OS,
   525:                               const HexagonBlockRanges::PrintRangeMap &P) {
   526:   for (const auto &I : P.Map) {
   527:     const HexagonBlockRanges::RangeList &RL = I.second;
   528:     OS << printReg(I.first.Reg, &P.TRI, I.first.Sub) << " -> " << RL << "\n";
   529:   }
   530:   return OS;
   531: }
```
- EN: It declares or implements routines such as getIndex, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonBlockRanges, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getIndex 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonBlockRanges，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- instruction semantics / 指令语义
- MC-layer target description / MC 层目标描述
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `HexagonBlockRanges.h, HexagonInstrInfo.h, HexagonSubtarget.h, llvm/ADT/BitVector.h, llvm/ADT/STLExtras.h, llvm/CodeGen/MachineBasicBlock.h, llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineInstr.h, llvm/CodeGen/MachineOperand.h, llvm/CodeGen/MachineRegisterInfo.h, ... (18 total)`
- Hexagon symbols / Hexagon 符号: `HexagonBlockRanges, HexagonInstrInfo, HexagonSubtarget`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
