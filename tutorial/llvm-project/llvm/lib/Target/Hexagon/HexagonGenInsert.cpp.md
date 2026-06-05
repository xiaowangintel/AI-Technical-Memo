# HexagonGenInsert.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonGenInsert.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon generation of insert operations.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-100 / 第 1-100 行

```cpp
     1: //===- HexagonGenInsert.cpp -----------------------------------------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: 
     9: #include "BitTracker.h"
    10: #include "Hexagon.h"
    11: #include "HexagonBitTracker.h"
    12: #include "HexagonInstrInfo.h"
    13: #include "HexagonRegisterInfo.h"
    14: #include "HexagonSubtarget.h"
    15: #include "llvm/ADT/BitVector.h"
    16: #include "llvm/ADT/DenseMap.h"
    17: #include "llvm/ADT/GraphTraits.h"
    18: #include "llvm/ADT/PostOrderIterator.h"
    19: #include "llvm/ADT/STLExtras.h"
    20: #include "llvm/ADT/SmallVector.h"
    21: #include "llvm/ADT/StringRef.h"
    22: #include "llvm/CodeGen/MachineBasicBlock.h"
    23: #include "llvm/CodeGen/MachineDominators.h"
    24: #include "llvm/CodeGen/MachineFunction.h"
    25: #include "llvm/CodeGen/MachineFunctionPass.h"
    26: #include "llvm/CodeGen/MachineInstr.h"
    27: #include "llvm/CodeGen/MachineInstrBuilder.h"
    28: #include "llvm/CodeGen/MachineOperand.h"
    29: #include "llvm/CodeGen/MachineRegisterInfo.h"
    30: #include "llvm/CodeGen/TargetRegisterInfo.h"
    31: #include "llvm/IR/DebugLoc.h"
    32: #include "llvm/InitializePasses.h"
    33: #include "llvm/Pass.h"
    34: #include "llvm/Support/CommandLine.h"
    35: #include "llvm/Support/Debug.h"
    36: #include "llvm/Support/MathExtras.h"
    37: #include "llvm/Support/Timer.h"
    38: #include "llvm/Support/raw_ostream.h"
    39: #include <algorithm>
    40: #include <cassert>
    41: #include <cstdint>
    42: #include <iterator>
    43: #include <utility>
    44: #include <vector>
    45: 
    46: #define DEBUG_TYPE "hexinsert"
    47: 
    48: using namespace llvm;
    49: 
    50: static cl::opt<unsigned>
    51:     VRegIndexCutoff("insert-vreg-cutoff", cl::init(~0U), cl::Hidden,
    52:                     cl::desc("Vreg# cutoff for insert generation."));
    53: // The distance cutoff is selected based on the precheckin-perf results:
    54: // cutoffs 20, 25, 35, and 40 are worse than 30.
    55: static cl::opt<unsigned>
    56:     VRegDistCutoff("insert-dist-cutoff", cl::init(30U), cl::Hidden,
    57:                    cl::desc("Vreg distance cutoff for insert "
    58:                             "generation."));
    59: 
    60: // Limit the container sizes for extreme cases where we run out of memory.
    61: static cl::opt<unsigned>
    62:     MaxORLSize("insert-max-orl", cl::init(4096), cl::Hidden,
    63:                cl::desc("Maximum size of OrderedRegisterList"));
    64: static cl::opt<unsigned> MaxIFMSize("insert-max-ifmap", cl::init(1024),
    65:                                     cl::Hidden,
    66:                                     cl::desc("Maximum size of IFMap"));
    67: 
    68: static cl::opt<bool> OptTiming("insert-timing", cl::Hidden,
    69:                                cl::desc("Enable timing of insert generation"));
    70: static cl::opt<bool>
    71:     OptTimingDetail("insert-timing-detail", cl::Hidden,
    72:                     cl::desc("Enable detailed timing of insert "
    73:                              "generation"));
    74: 
    75: static cl::opt<bool> OptSelectAll0("insert-all0", cl::init(false), cl::Hidden);
    76: static cl::opt<bool> OptSelectHas0("insert-has0", cl::init(false), cl::Hidden);
    77: // Whether to construct constant values via "insert". Could eliminate constant
    78: // extenders, but often not practical.
    79: static cl::opt<bool> OptConst("insert-const", cl::init(false), cl::Hidden);
    80: 
    81: // The preprocessor gets confused when the DEBUG macro is passed larger
    82: // chunks of code. Use this function to detect debugging.
    83: inline static bool isDebug() {
    84: #ifndef NDEBUG
    85:   return DebugFlag && isCurrentDebugType(DEBUG_TYPE);
    86: #else
    87:   return false;
    88: #endif
    89: }
    90: 
    91: namespace {
    92: 
    93:   // Set of virtual registers, based on BitVector.
    94:   struct RegisterSet : private BitVector {
    95:     RegisterSet() = default;
    96:     explicit RegisterSet(unsigned s, bool t = false) : BitVector(s, t) {}
    97:     RegisterSet(const RegisterSet &RS) = default;
    98:     RegisterSet &operator=(const RegisterSet &RS) = default;
    99: 
   100:     using BitVector::clear;
```
- EN: It imports headers such as BitTracker.h, Hexagon.h, HexagonBitTracker.h, HexagonInstrInfo.h, ... (36 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as RegisterSet, which carry the state or API of this component.
- CN: 这里引入了 BitTracker.h, Hexagon.h, HexagonBitTracker.h, HexagonInstrInfo.h, ... (36 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 RegisterSet 等类型，用来承载该组件的状态或接口。

### Lines 101-200 / 第 101-200 行

```cpp
   101: 
   102:     unsigned find_first() const {
   103:       int First = BitVector::find_first();
   104:       if (First < 0)
   105:         return 0;
   106:       return x2v(First);
   107:     }
   108: 
   109:     unsigned find_next(unsigned Prev) const {
   110:       int Next = BitVector::find_next(v2x(Prev));
   111:       if (Next < 0)
   112:         return 0;
   113:       return x2v(Next);
   114:     }
   115: 
   116:     RegisterSet &insert(unsigned R) {
   117:       unsigned Idx = v2x(R);
   118:       ensure(Idx);
   119:       return static_cast<RegisterSet&>(BitVector::set(Idx));
   120:     }
   121:     RegisterSet &remove(unsigned R) {
   122:       unsigned Idx = v2x(R);
   123:       if (Idx >= size())
   124:         return *this;
   125:       return static_cast<RegisterSet&>(BitVector::reset(Idx));
   126:     }
   127: 
   128:     RegisterSet &insert(const RegisterSet &Rs) {
   129:       return static_cast<RegisterSet&>(BitVector::operator|=(Rs));
   130:     }
   131:     RegisterSet &remove(const RegisterSet &Rs) {
   132:       return static_cast<RegisterSet&>(BitVector::reset(Rs));
   133:     }
   134: 
   135:     reference operator[](unsigned R) {
   136:       unsigned Idx = v2x(R);
   137:       ensure(Idx);
   138:       return BitVector::operator[](Idx);
   139:     }
   140:     bool operator[](unsigned R) const {
   141:       unsigned Idx = v2x(R);
   142:       assert(Idx < size());
   143:       return BitVector::operator[](Idx);
   144:     }
   145:     bool has(unsigned R) const {
   146:       unsigned Idx = v2x(R);
   147:       if (Idx >= size())
   148:         return false;
   149:       return BitVector::test(Idx);
   150:     }
   151: 
   152:     bool empty() const {
   153:       return !BitVector::any();
   154:     }
   155:     bool includes(const RegisterSet &Rs) const {
   156:       return Rs.BitVector::subsetOf(*this);
   157:     }
   158:     bool intersects(const RegisterSet &Rs) const {
   159:       return BitVector::anyCommon(Rs);
   160:     }
   161: 
   162:   private:
   163:     void ensure(unsigned Idx) {
   164:       if (size() <= Idx)
   165:         resize(std::max(Idx+1, 32U));
   166:     }
   167: 
   168:     static inline unsigned v2x(unsigned v) {
   169:       return Register(v).virtRegIndex();
   170:     }
   171: 
   172:     static inline unsigned x2v(unsigned x) {
   173:       return Register::index2VirtReg(x);
   174:     }
   175:   };
   176: 
   177:   struct PrintRegSet {
   178:     PrintRegSet(const RegisterSet &S, const TargetRegisterInfo *RI)
   179:       : RS(S), TRI(RI) {}
   180: 
   181:     friend raw_ostream &operator<< (raw_ostream &OS,
   182:           const PrintRegSet &P);
   183: 
   184:   private:
   185:     const RegisterSet &RS;
   186:     const TargetRegisterInfo *TRI;
   187:   };
   188: 
   189:   raw_ostream &operator<< (raw_ostream &OS, const PrintRegSet &P) {
   190:     OS << '{';
   191:     for (unsigned R = P.RS.find_first(); R; R = P.RS.find_next(R))
   192:       OS << ' ' << printReg(R, P.TRI);
   193:     OS << " }";
   194:     return OS;
   195:   }
   196: 
   197:   // A convenience class to associate unsigned numbers (such as virtual
   198:   // registers) with unsigned numbers.
   199:   struct UnsignedMap : public DenseMap<unsigned,unsigned> {
   200:     UnsignedMap() = default;
```
- EN: It declares types such as PrintRegSet, UnsignedMap, which carry the state or API of this component. It declares or implements routines such as find_first, BitVector::find_first, x2v, find_next, ... (24 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明了 PrintRegSet, UnsignedMap 等类型，用来承载该组件的状态或接口。 这里声明或实现了 find_first, BitVector::find_first, x2v, find_next, ... (24 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 201-300 / 第 201-300 行

```cpp
   201: 
   202:   private:
   203:     using BaseType = DenseMap<unsigned, unsigned>;
   204:   };
   205: 
   206:   // A utility to establish an ordering between virtual registers:
   207:   // VRegA < VRegB  <=>  RegisterOrdering[VRegA] < RegisterOrdering[VRegB]
   208:   // This is meant as a cache for the ordering of virtual registers defined
   209:   // by a potentially expensive comparison function, or obtained by a proce-
   210:   // dure that should not be repeated each time two registers are compared.
   211:   struct RegisterOrdering : public UnsignedMap {
   212:     RegisterOrdering() = default;
   213: 
   214:     unsigned operator[](unsigned VR) const {
   215:       const_iterator F = find(VR);
   216:       assert(F != end());
   217:       return F->second;
   218:     }
   219: 
   220:     // Add operator(), so that objects of this class can be used as
   221:     // comparators in std::sort et al.
   222:     bool operator() (unsigned VR1, unsigned VR2) const {
   223:       return operator[](VR1) < operator[](VR2);
   224:     }
   225:   };
   226: 
   227:   // Ordering of bit values. This class does not have operator[], but
   228:   // is supplies a comparison operator() for use in std:: algorithms.
   229:   // The order is as follows:
   230:   // - 0 < 1 < ref
   231:   // - ref1 < ref2, if ord(ref1.Reg) < ord(ref2.Reg),
   232:   //   or ord(ref1.Reg) == ord(ref2.Reg), and ref1.Pos < ref2.Pos.
   233:   struct BitValueOrdering {
   234:     BitValueOrdering(const RegisterOrdering &RB) : BaseOrd(RB) {}
   235: 
   236:     bool operator() (const BitTracker::BitValue &V1,
   237:           const BitTracker::BitValue &V2) const;
   238: 
   239:     const RegisterOrdering &BaseOrd;
   240:   };
   241: 
   242: } // end anonymous namespace
   243: 
   244: bool BitValueOrdering::operator() (const BitTracker::BitValue &V1,
   245:       const BitTracker::BitValue &V2) const {
   246:   if (V1 == V2)
   247:     return false;
   248:   // V1==0 => true, V2==0 => false
   249:   if (V1.is(0) || V2.is(0))
   250:     return V1.is(0);
   251:   // Neither of V1,V2 is 0, and V1!=V2.
   252:   // V2==1 => false, V1==1 => true
   253:   if (V2.is(1) || V1.is(1))
   254:     return !V2.is(1);
   255:   // Both V1,V2 are refs.
   256:   unsigned Ind1 = BaseOrd[V1.RefI.Reg], Ind2 = BaseOrd[V2.RefI.Reg];
   257:   if (Ind1 != Ind2)
   258:     return Ind1 < Ind2;
   259:   // If V1.Pos==V2.Pos
   260:   assert(V1.RefI.Pos != V2.RefI.Pos && "Bit values should be different");
   261:   return V1.RefI.Pos < V2.RefI.Pos;
   262: }
   263: 
   264: namespace {
   265: 
   266:   // Cache for the BitTracker's cell map. Map lookup has a logarithmic
   267:   // complexity, this class will memoize the lookup results to reduce
   268:   // the access time for repeated lookups of the same cell.
   269:   struct CellMapShadow {
   270:     CellMapShadow(const BitTracker &T) : BT(T) {}
   271: 
   272:     const BitTracker::RegisterCell &lookup(unsigned VR) {
   273:       unsigned RInd = Register(VR).virtRegIndex();
   274:       // Grow the vector to at least 32 elements.
   275:       if (RInd >= CVect.size())
   276:         CVect.resize(std::max(RInd+16, 32U), nullptr);
   277:       const BitTracker::RegisterCell *CP = CVect[RInd];
   278:       if (CP == nullptr)
   279:         CP = CVect[RInd] = &BT.lookup(VR);
   280:       return *CP;
   281:     }
   282: 
   283:     const BitTracker &BT;
   284: 
   285:   private:
   286:     using CellVectType = std::vector<const BitTracker::RegisterCell *>;
   287: 
   288:     CellVectType CVect;
   289:   };
   290: 
   291:   // Comparator class for lexicographic ordering of virtual registers
   292:   // according to the corresponding BitTracker::RegisterCell objects.
   293:   struct RegisterCellLexCompare {
   294:     RegisterCellLexCompare(const BitValueOrdering &BO, CellMapShadow &M)
   295:       : BitOrd(BO), CM(M) {}
   296: 
   297:     bool operator() (unsigned VR1, unsigned VR2) const;
   298: 
   299:   private:
   300:     const BitValueOrdering &BitOrd;
```
- EN: It opens namespaces (bool) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as RegisterOrdering, BitValueOrdering, CellMapShadow, RegisterCellLexCompare, which carry the state or API of this component. It declares or implements routines such as RegisterOrdering, find, assert, BitValueOrdering, ... (8 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里打开了命名空间（bool），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 RegisterOrdering, BitValueOrdering, CellMapShadow, RegisterCellLexCompare 等类型，用来承载该组件的状态或接口。 这里声明或实现了 RegisterOrdering, find, assert, BitValueOrdering, ... (8 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 301-400 / 第 301-400 行

```cpp
   301:     CellMapShadow &CM;
   302:   };
   303: 
   304:   // Comparator class for lexicographic ordering of virtual registers
   305:   // according to the specified bits of the corresponding BitTracker::
   306:   // RegisterCell objects.
   307:   // Specifically, this class will be used to compare bit B of a register
   308:   // cell for a selected virtual register R with bit N of any register
   309:   // other than R.
   310:   struct RegisterCellBitCompareSel {
   311:     RegisterCellBitCompareSel(unsigned R, unsigned B, unsigned N,
   312:           const BitValueOrdering &BO, CellMapShadow &M)
   313:       : SelR(R), SelB(B), BitN(N), BitOrd(BO), CM(M) {}
   314: 
   315:     bool operator() (unsigned VR1, unsigned VR2) const;
   316: 
   317:   private:
   318:     const unsigned SelR, SelB;
   319:     const unsigned BitN;
   320:     const BitValueOrdering &BitOrd;
   321:     CellMapShadow &CM;
   322:   };
   323: 
   324: } // end anonymous namespace
   325: 
   326: bool RegisterCellLexCompare::operator() (unsigned VR1, unsigned VR2) const {
   327:   // Ordering of registers, made up from two given orderings:
   328:   // - the ordering of the register numbers, and
   329:   // - the ordering of register cells.
   330:   // Def. R1 < R2 if:
   331:   // - cell(R1) < cell(R2), or
   332:   // - cell(R1) == cell(R2), and index(R1) < index(R2).
   333:   //
   334:   // For register cells, the ordering is lexicographic, with index 0 being
   335:   // the most significant.
   336:   if (VR1 == VR2)
   337:     return false;
   338: 
   339:   const BitTracker::RegisterCell &RC1 = CM.lookup(VR1), &RC2 = CM.lookup(VR2);
   340:   uint16_t W1 = RC1.width(), W2 = RC2.width();
   341:   for (uint16_t i = 0, w = std::min(W1, W2); i < w; ++i) {
   342:     const BitTracker::BitValue &V1 = RC1[i], &V2 = RC2[i];
   343:     if (V1 != V2)
   344:       return BitOrd(V1, V2);
   345:   }
   346:   // Cells are equal up until the common length.
   347:   if (W1 != W2)
   348:     return W1 < W2;
   349: 
   350:   return BitOrd.BaseOrd[VR1] < BitOrd.BaseOrd[VR2];
   351: }
   352: 
   353: bool RegisterCellBitCompareSel::operator() (unsigned VR1, unsigned VR2) const {
   354:   if (VR1 == VR2)
   355:     return false;
   356:   const BitTracker::RegisterCell &RC1 = CM.lookup(VR1);
   357:   const BitTracker::RegisterCell &RC2 = CM.lookup(VR2);
   358:   uint16_t W1 = RC1.width(), W2 = RC2.width();
   359:   uint16_t Bit1 = (VR1 == SelR) ? SelB : BitN;
   360:   uint16_t Bit2 = (VR2 == SelR) ? SelB : BitN;
   361:   // If Bit1 exceeds the width of VR1, then:
   362:   // - return false, if at the same time Bit2 exceeds VR2, or
   363:   // - return true, otherwise.
   364:   // (I.e. "a bit value that does not exist is less than any bit value
   365:   // that does exist".)
   366:   if (W1 <= Bit1)
   367:     return Bit2 < W2;
   368:   // If Bit1 is within VR1, but Bit2 is not within VR2, return false.
   369:   if (W2 <= Bit2)
   370:     return false;
   371: 
   372:   const BitTracker::BitValue &V1 = RC1[Bit1], V2 = RC2[Bit2];
   373:   if (V1 != V2)
   374:     return BitOrd(V1, V2);
   375:   return false;
   376: }
   377: 
   378: namespace {
   379: 
   380:   class OrderedRegisterList {
   381:     using ListType = std::vector<unsigned>;
   382:     const unsigned MaxSize;
   383: 
   384:   public:
   385:     OrderedRegisterList(const RegisterOrdering &RO)
   386:       : MaxSize(MaxORLSize), Ord(RO) {}
   387: 
   388:     void insert(unsigned VR);
   389:     void remove(unsigned VR);
   390: 
   391:     unsigned operator[](unsigned Idx) const {
   392:       assert(Idx < Seq.size());
   393:       return Seq[Idx];
   394:     }
   395: 
   396:     unsigned size() const {
   397:       return Seq.size();
   398:     }
   399: 
   400:     using iterator = ListType::iterator;
```
- EN: It opens namespaces (bool) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as RegisterCellBitCompareSel, OrderedRegisterList, which carry the state or API of this component. It defines declarative TableGen records like OrderedRegisterList; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as RegisterCellBitCompareSel, lookup, width, OrderedRegisterList, ... (8 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里打开了命名空间（bool），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 RegisterCellBitCompareSel, OrderedRegisterList 等类型，用来承载该组件的状态或接口。 这里定义了 OrderedRegisterList 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 RegisterCellBitCompareSel, lookup, width, OrderedRegisterList, ... (8 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 401-500 / 第 401-500 行

```cpp
   401:     using const_iterator = ListType::const_iterator;
   402: 
   403:     iterator begin() { return Seq.begin(); }
   404:     iterator end() { return Seq.end(); }
   405:     const_iterator begin() const { return Seq.begin(); }
   406:     const_iterator end() const { return Seq.end(); }
   407: 
   408:     // Convenience function to convert an iterator to the corresponding index.
   409:     unsigned idx(iterator It) const { return It-begin(); }
   410: 
   411:   private:
   412:     ListType Seq;
   413:     const RegisterOrdering &Ord;
   414:   };
   415: 
   416:   struct PrintORL {
   417:     PrintORL(const OrderedRegisterList &L, const TargetRegisterInfo *RI)
   418:       : RL(L), TRI(RI) {}
   419: 
   420:     friend raw_ostream &operator<< (raw_ostream &OS, const PrintORL &P);
   421: 
   422:   private:
   423:     const OrderedRegisterList &RL;
   424:     const TargetRegisterInfo *TRI;
   425:   };
   426: 
   427:   raw_ostream &operator<< (raw_ostream &OS, const PrintORL &P) {
   428:     OS << '(';
   429:     OrderedRegisterList::const_iterator B = P.RL.begin(), E = P.RL.end();
   430:     for (OrderedRegisterList::const_iterator I = B; I != E; ++I) {
   431:       if (I != B)
   432:         OS << ", ";
   433:       OS << printReg(*I, P.TRI);
   434:     }
   435:     OS << ')';
   436:     return OS;
   437:   }
   438: 
   439: } // end anonymous namespace
   440: 
   441: void OrderedRegisterList::insert(unsigned VR) {
   442:   iterator L = llvm::lower_bound(Seq, VR, Ord);
   443:   if (L == Seq.end())
   444:     Seq.push_back(VR);
   445:   else
   446:     Seq.insert(L, VR);
   447: 
   448:   unsigned S = Seq.size();
   449:   if (S > MaxSize)
   450:     Seq.resize(MaxSize);
   451:   assert(Seq.size() <= MaxSize);
   452: }
   453: 
   454: void OrderedRegisterList::remove(unsigned VR) {
   455:   iterator L = llvm::lower_bound(Seq, VR, Ord);
   456:   if (L != Seq.end())
   457:     Seq.erase(L);
   458: }
   459: 
   460: namespace {
   461: 
   462:   // A record of the insert form. The fields correspond to the operands
   463:   // of the "insert" instruction:
   464:   // ... = insert(SrcR, InsR, #Wdh, #Off)
   465:   struct IFRecord {
   466:     IFRecord(unsigned SR = 0, unsigned IR = 0, uint16_t W = 0, uint16_t O = 0)
   467:       : SrcR(SR), InsR(IR), Wdh(W), Off(O) {}
   468: 
   469:     unsigned SrcR, InsR;
   470:     uint16_t Wdh, Off;
   471:   };
   472: 
   473:   struct PrintIFR {
   474:     PrintIFR(const IFRecord &R, const TargetRegisterInfo *RI)
   475:       : IFR(R), TRI(RI) {}
   476: 
   477:   private:
   478:     friend raw_ostream &operator<< (raw_ostream &OS, const PrintIFR &P);
   479: 
   480:     const IFRecord &IFR;
   481:     const TargetRegisterInfo *TRI;
   482:   };
   483: 
   484:   raw_ostream &operator<< (raw_ostream &OS, const PrintIFR &P) {
   485:     unsigned SrcR = P.IFR.SrcR, InsR = P.IFR.InsR;
   486:     OS << '(' << printReg(SrcR, P.TRI) << ',' << printReg(InsR, P.TRI)
   487:        << ",#" << P.IFR.Wdh << ",#" << P.IFR.Off << ')';
   488:     return OS;
   489:   }
   490: 
   491:   using IFRecordWithRegSet = std::pair<IFRecord, RegisterSet>;
   492: 
   493: } // end anonymous namespace
   494: 
   495: namespace {
   496: 
   497:   class HexagonGenInsert : public MachineFunctionPass {
   498:   public:
   499:     static char ID;
   500: 
```
- EN: It opens namespaces (void, namespace) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as PrintORL, IFRecord, PrintIFR, HexagonGenInsert, which carry the state or API of this component. It defines declarative TableGen records like HexagonGenInsert; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as begin, end, idx, PrintORL, ... (13 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里打开了命名空间（void, namespace），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 PrintORL, IFRecord, PrintIFR, HexagonGenInsert 等类型，用来承载该组件的状态或接口。 这里定义了 HexagonGenInsert 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 begin, end, idx, PrintORL, ... (13 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 501-600 / 第 501-600 行

```cpp
   501:     HexagonGenInsert() : MachineFunctionPass(ID) {}
   502: 
   503:     StringRef getPassName() const override {
   504:       return "Hexagon generate \"insert\" instructions";
   505:     }
   506: 
   507:     void getAnalysisUsage(AnalysisUsage &AU) const override {
   508:       AU.addRequired<MachineDominatorTreeWrapperPass>();
   509:       AU.addPreserved<MachineDominatorTreeWrapperPass>();
   510:       MachineFunctionPass::getAnalysisUsage(AU);
   511:     }
   512: 
   513:     bool runOnMachineFunction(MachineFunction &MF) override;
   514: 
   515:   private:
   516:     using PairMapType = DenseMap<std::pair<unsigned, unsigned>, unsigned>;
   517: 
   518:     void buildOrderingMF(RegisterOrdering &RO) const;
   519:     void buildOrderingBT(RegisterOrdering &RB, RegisterOrdering &RO) const;
   520:     bool isIntClass(const TargetRegisterClass *RC) const;
   521:     bool isConstant(unsigned VR) const;
   522:     bool isSmallConstant(unsigned VR) const;
   523:     bool isValidInsertForm(unsigned DstR, unsigned SrcR, unsigned InsR,
   524:           uint16_t L, uint16_t S) const;
   525:     bool findSelfReference(unsigned VR) const;
   526:     bool findNonSelfReference(unsigned VR) const;
   527:     void getInstrDefs(const MachineInstr *MI, RegisterSet &Defs) const;
   528:     void getInstrUses(const MachineInstr *MI, RegisterSet &Uses) const;
   529:     unsigned distance(const MachineBasicBlock *FromB,
   530:           const MachineBasicBlock *ToB, const UnsignedMap &RPO,
   531:           PairMapType &M) const;
   532:     unsigned distance(MachineBasicBlock::const_iterator FromI,
   533:           MachineBasicBlock::const_iterator ToI, const UnsignedMap &RPO,
   534:           PairMapType &M) const;
   535:     bool findRecordInsertForms(unsigned VR, OrderedRegisterList &AVs);
   536:     void collectInBlock(MachineBasicBlock *B, OrderedRegisterList &AVs);
   537:     void findRemovableRegisters(unsigned VR, IFRecord IF,
   538:           RegisterSet &RMs) const;
   539:     void computeRemovableRegisters();
   540: 
   541:     void pruneEmptyLists();
   542:     void pruneCoveredSets(unsigned VR);
   543:     void pruneUsesTooFar(unsigned VR, const UnsignedMap &RPO, PairMapType &M);
   544:     void pruneRegCopies(unsigned VR);
   545:     void pruneCandidates();
   546:     void selectCandidates();
   547:     bool generateInserts();
   548: 
   549:     bool removeDeadCode(MachineDomTreeNode *N);
   550: 
   551:     // IFRecord coupled with a set of potentially removable registers:
   552:     using IFListType = std::vector<IFRecordWithRegSet>;
   553:     using IFMapType = DenseMap<unsigned, IFListType>; // vreg -> IFListType
   554: 
   555:     void dump_map() const;
   556: 
   557:     const HexagonInstrInfo *HII = nullptr;
   558:     const HexagonRegisterInfo *HRI = nullptr;
   559: 
   560:     MachineFunction *MFN;
   561:     MachineRegisterInfo *MRI;
   562:     MachineDominatorTree *MDT;
   563:     CellMapShadow *CMS;
   564: 
   565:     RegisterOrdering BaseOrd;
   566:     RegisterOrdering CellOrd;
   567:     IFMapType IFMap;
   568:   };
   569: 
   570: } // end anonymous namespace
   571: 
   572: char HexagonGenInsert::ID = 0;
   573: 
   574: void HexagonGenInsert::dump_map() const {
   575:   for (const auto &I : IFMap) {
   576:     dbgs() << "  " << printReg(I.first, HRI) << ":\n";
   577:     const IFListType &LL = I.second;
   578:     for (const auto &J : LL)
   579:       dbgs() << "    " << PrintIFR(J.first, HRI) << ", "
   580:              << PrintRegSet(J.second, HRI) << '\n';
   581:   }
   582: }
   583: 
   584: void HexagonGenInsert::buildOrderingMF(RegisterOrdering &RO) const {
   585:   unsigned Index = 0;
   586: 
   587:   for (const MachineBasicBlock &B : *MFN) {
   588:     if (!CMS->BT.reached(&B))
   589:       continue;
   590: 
   591:     for (const MachineInstr &MI : B) {
   592:       for (const MachineOperand &MO : MI.operands()) {
   593:         if (MO.isReg() && MO.isDef()) {
   594:           Register R = MO.getReg();
   595:           assert(MO.getSubReg() == 0 && "Unexpected subregister in definition");
   596:           if (R.isVirtual())
   597:             RO.insert(std::make_pair(R, Index++));
   598:         }
   599:       }
   600:     }
```
- EN: It opens namespaces (char) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as HexagonGenInsert, getPassName, getAnalysisUsage, addRequired<MachineDominatorTreeWrapperPass>, ... (35 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里打开了命名空间（char），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 HexagonGenInsert, getPassName, getAnalysisUsage, addRequired<MachineDominatorTreeWrapperPass>, ... (35 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 601-700 / 第 601-700 行

```cpp
   601:   }
   602:   // Since some virtual registers may have had their def and uses eliminated,
   603:   // they are no longer referenced in the code, and so they will not appear
   604:   // in the map.
   605: }
   606: 
   607: void HexagonGenInsert::buildOrderingBT(RegisterOrdering &RB,
   608:       RegisterOrdering &RO) const {
   609:   // Create a vector of all virtual registers (collect them from the base
   610:   // ordering RB), and then sort it using the RegisterCell comparator.
   611:   BitValueOrdering BVO(RB);
   612:   RegisterCellLexCompare LexCmp(BVO, *CMS);
   613: 
   614:   using SortableVectorType = std::vector<unsigned>;
   615: 
   616:   SortableVectorType VRs;
   617:   for (auto &I : RB)
   618:     VRs.push_back(I.first);
   619:   llvm::sort(VRs, LexCmp);
   620:   // Transfer the results to the outgoing register ordering.
   621:   for (unsigned i = 0, n = VRs.size(); i < n; ++i)
   622:     RO.insert(std::make_pair(VRs[i], i));
   623: }
   624: 
   625: inline bool HexagonGenInsert::isIntClass(const TargetRegisterClass *RC) const {
   626:   return RC == &Hexagon::IntRegsRegClass || RC == &Hexagon::DoubleRegsRegClass;
   627: }
   628: 
   629: bool HexagonGenInsert::isConstant(unsigned VR) const {
   630:   const BitTracker::RegisterCell &RC = CMS->lookup(VR);
   631:   uint16_t W = RC.width();
   632:   for (uint16_t i = 0; i < W; ++i) {
   633:     const BitTracker::BitValue &BV = RC[i];
   634:     if (BV.is(0) || BV.is(1))
   635:       continue;
   636:     return false;
   637:   }
   638:   return true;
   639: }
   640: 
   641: bool HexagonGenInsert::isSmallConstant(unsigned VR) const {
   642:   const BitTracker::RegisterCell &RC = CMS->lookup(VR);
   643:   uint16_t W = RC.width();
   644:   if (W > 64)
   645:     return false;
   646:   uint64_t V = 0, B = 1;
   647:   for (uint16_t i = 0; i < W; ++i) {
   648:     const BitTracker::BitValue &BV = RC[i];
   649:     if (BV.is(1))
   650:       V |= B;
   651:     else if (!BV.is(0))
   652:       return false;
   653:     B <<= 1;
   654:   }
   655: 
   656:   // For 32-bit registers, consider: Rd = #s16.
   657:   if (W == 32)
   658:     return isInt<16>(V);
   659: 
   660:   // For 64-bit registers, it's Rdd = #s8 or Rdd = combine(#s8,#s8)
   661:   return isInt<8>(Lo_32(V)) && isInt<8>(Hi_32(V));
   662: }
   663: 
   664: bool HexagonGenInsert::isValidInsertForm(unsigned DstR, unsigned SrcR,
   665:       unsigned InsR, uint16_t L, uint16_t S) const {
   666:   const TargetRegisterClass *DstRC = MRI->getRegClass(DstR);
   667:   const TargetRegisterClass *SrcRC = MRI->getRegClass(SrcR);
   668:   const TargetRegisterClass *InsRC = MRI->getRegClass(InsR);
   669:   // Only integet (32-/64-bit) register classes.
   670:   if (!isIntClass(DstRC) || !isIntClass(SrcRC) || !isIntClass(InsRC))
   671:     return false;
   672:   // The "source" register must be of the same class as DstR.
   673:   if (DstRC != SrcRC)
   674:     return false;
   675:   if (DstRC == InsRC)
   676:     return true;
   677:   // A 64-bit register can only be generated from other 64-bit registers.
   678:   if (DstRC == &Hexagon::DoubleRegsRegClass)
   679:     return false;
   680:   // Otherwise, the L and S cannot span 32-bit word boundary.
   681:   if (S < 32 && S+L > 32)
   682:     return false;
   683:   return true;
   684: }
   685: 
   686: bool HexagonGenInsert::findSelfReference(unsigned VR) const {
   687:   const BitTracker::RegisterCell &RC = CMS->lookup(VR);
   688:   for (uint16_t i = 0, w = RC.width(); i < w; ++i) {
   689:     const BitTracker::BitValue &V = RC[i];
   690:     if (V.Type == BitTracker::BitValue::Ref && V.RefI.Reg == VR)
   691:       return true;
   692:   }
   693:   return false;
   694: }
   695: 
   696: bool HexagonGenInsert::findNonSelfReference(unsigned VR) const {
   697:   BitTracker::RegisterCell RC = CMS->lookup(VR);
   698:   for (uint16_t i = 0, w = RC.width(); i < w; ++i) {
   699:     const BitTracker::BitValue &V = RC[i];
   700:     if (V.Type == BitTracker::BitValue::Ref && V.RefI.Reg != VR)
```
- EN: It declares or implements routines such as HexagonGenInsert::buildOrderingBT, registers, LexCmp, llvm::sort, ... (15 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonGenInsert, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonGenInsert::buildOrderingBT, registers, LexCmp, llvm::sort, ... (15 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonGenInsert，说明了它与同级后端组件的连接关系。

### Lines 701-800 / 第 701-800 行

```cpp
   701:       return true;
   702:   }
   703:   return false;
   704: }
   705: 
   706: void HexagonGenInsert::getInstrDefs(const MachineInstr *MI,
   707:       RegisterSet &Defs) const {
   708:   for (const MachineOperand &MO : MI->operands()) {
   709:     if (!MO.isReg() || !MO.isDef())
   710:       continue;
   711:     Register R = MO.getReg();
   712:     if (!R.isVirtual())
   713:       continue;
   714:     Defs.insert(R);
   715:   }
   716: }
   717: 
   718: void HexagonGenInsert::getInstrUses(const MachineInstr *MI,
   719:       RegisterSet &Uses) const {
   720:   for (const MachineOperand &MO : MI->operands()) {
   721:     if (!MO.isReg() || !MO.isUse())
   722:       continue;
   723:     Register R = MO.getReg();
   724:     if (!R.isVirtual())
   725:       continue;
   726:     Uses.insert(R);
   727:   }
   728: }
   729: 
   730: unsigned HexagonGenInsert::distance(const MachineBasicBlock *FromB,
   731:       const MachineBasicBlock *ToB, const UnsignedMap &RPO,
   732:       PairMapType &M) const {
   733:   // Forward distance from the end of a block to the beginning of it does
   734:   // not make sense. This function should not be called with FromB == ToB.
   735:   assert(FromB != ToB);
   736: 
   737:   unsigned FromN = FromB->getNumber(), ToN = ToB->getNumber();
   738:   // If we have already computed it, return the cached result.
   739:   PairMapType::iterator F = M.find(std::make_pair(FromN, ToN));
   740:   if (F != M.end())
   741:     return F->second;
   742:   unsigned ToRPO = RPO.lookup(ToN);
   743: 
   744:   unsigned MaxD = 0;
   745: 
   746:   for (const MachineBasicBlock *PB : ToB->predecessors()) {
   747:     // Skip back edges. Also, if FromB is a predecessor of ToB, the distance
   748:     // along that path will be 0, and we don't need to do any calculations
   749:     // on it.
   750:     if (PB == FromB || RPO.lookup(PB->getNumber()) >= ToRPO)
   751:       continue;
   752:     unsigned D = PB->size() + distance(FromB, PB, RPO, M);
   753:     if (D > MaxD)
   754:       MaxD = D;
   755:   }
   756: 
   757:   // Memoize the result for later lookup.
   758:   M.insert(std::make_pair(std::make_pair(FromN, ToN), MaxD));
   759:   return MaxD;
   760: }
   761: 
   762: unsigned HexagonGenInsert::distance(MachineBasicBlock::const_iterator FromI,
   763:       MachineBasicBlock::const_iterator ToI, const UnsignedMap &RPO,
   764:       PairMapType &M) const {
   765:   const MachineBasicBlock *FB = FromI->getParent(), *TB = ToI->getParent();
   766:   if (FB == TB)
   767:     return std::distance(FromI, ToI);
   768:   unsigned D1 = std::distance(TB->begin(), ToI);
   769:   unsigned D2 = distance(FB, TB, RPO, M);
   770:   unsigned D3 = std::distance(FromI, FB->end());
   771:   return D1+D2+D3;
   772: }
   773: 
   774: bool HexagonGenInsert::findRecordInsertForms(unsigned VR,
   775:       OrderedRegisterList &AVs) {
   776:   if (isDebug()) {
   777:     dbgs() << __func__ << ": " << printReg(VR, HRI)
   778:            << "  AVs: " << PrintORL(AVs, HRI) << "\n";
   779:   }
   780:   if (AVs.size() == 0)
   781:     return false;
   782: 
   783:   using iterator = OrderedRegisterList::iterator;
   784: 
   785:   BitValueOrdering BVO(BaseOrd);
   786:   const BitTracker::RegisterCell &RC = CMS->lookup(VR);
   787:   uint16_t W = RC.width();
   788: 
   789:   using RSRecord = std::pair<unsigned, uint16_t>; // (reg,shift)
   790:   using RSListType = std::vector<RSRecord>;
   791:   // Have a map, with key being the matching prefix length, and the value
   792:   // being the list of pairs (R,S), where R's prefix matches VR at S.
   793:   // (DenseMap<uint16_t,RSListType> fails to instantiate.)
   794:   using LRSMapType = DenseMap<unsigned, RSListType>;
   795:   LRSMapType LM;
   796: 
   797:   // Conceptually, rotate the cell RC right (i.e. towards the LSB) by S,
   798:   // and find matching prefixes from AVs with the rotated RC. Such a prefix
   799:   // would match a string of bits (of length L) in RC starting at S.
   800:   for (uint16_t S = 0; S < W; ++S) {
```
- EN: It declares or implements routines such as HexagonGenInsert::getInstrDefs, getReg, insert, HexagonGenInsert::getInstrUses, ... (16 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonGenInsert, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonGenInsert::getInstrDefs, getReg, insert, HexagonGenInsert::getInstrUses, ... (16 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonGenInsert，说明了它与同级后端组件的连接关系。

### Lines 801-900 / 第 801-900 行

```cpp
   801:     iterator B = AVs.begin(), E = AVs.end();
   802:     // The registers in AVs are ordered according to the lexical order of
   803:     // the corresponding register cells. This means that the range of regis-
   804:     // ters in AVs that match a prefix of length L+1 will be contained in
   805:     // the range that matches a prefix of length L. This means that we can
   806:     // keep narrowing the search space as the prefix length goes up. This
   807:     // helps reduce the overall complexity of the search.
   808:     uint16_t L;
   809:     for (L = 0; L < W-S; ++L) {
   810:       // Compare against VR's bits starting at S, which emulates rotation
   811:       // of VR by S.
   812:       RegisterCellBitCompareSel RCB(VR, S+L, L, BVO, *CMS);
   813:       iterator NewB = std::lower_bound(B, E, VR, RCB);
   814:       iterator NewE = std::upper_bound(NewB, E, VR, RCB);
   815:       // For the registers that are eliminated from the next range, L is
   816:       // the longest prefix matching VR at position S (their prefixes
   817:       // differ from VR at S+L). If L>0, record this information for later
   818:       // use.
   819:       if (L > 0) {
   820:         for (iterator I = B; I != NewB; ++I)
   821:           LM[L].push_back(std::make_pair(*I, S));
   822:         for (iterator I = NewE; I != E; ++I)
   823:           LM[L].push_back(std::make_pair(*I, S));
   824:       }
   825:       B = NewB, E = NewE;
   826:       if (B == E)
   827:         break;
   828:     }
   829:     // Record the final register range. If this range is non-empty, then
   830:     // L=W-S.
   831:     assert(B == E || L == W-S);
   832:     if (B != E) {
   833:       for (iterator I = B; I != E; ++I)
   834:         LM[L].push_back(std::make_pair(*I, S));
   835:       // If B!=E, then we found a range of registers whose prefixes cover the
   836:       // rest of VR from position S. There is no need to further advance S.
   837:       break;
   838:     }
   839:   }
   840: 
   841:   if (isDebug()) {
   842:     dbgs() << "Prefixes matching register " << printReg(VR, HRI) << "\n";
   843:     for (const auto &I : LM) {
   844:       dbgs() << "  L=" << I.first << ':';
   845:       const RSListType &LL = I.second;
   846:       for (const auto &J : LL)
   847:         dbgs() << " (" << printReg(J.first, HRI) << ",@" << J.second << ')';
   848:       dbgs() << '\n';
   849:     }
   850:   }
   851: 
   852:   bool Recorded = false;
   853: 
   854:   for (unsigned SrcR : AVs) {
   855:     int FDi = -1, LDi = -1;   // First/last different bit.
   856:     const BitTracker::RegisterCell &AC = CMS->lookup(SrcR);
   857:     uint16_t AW = AC.width();
   858:     for (uint16_t i = 0, w = std::min(W, AW); i < w; ++i) {
   859:       if (RC[i] == AC[i])
   860:         continue;
   861:       if (FDi == -1)
   862:         FDi = i;
   863:       LDi = i;
   864:     }
   865:     if (FDi == -1)
   866:       continue;  // TODO (future): Record identical registers.
   867:     // Look for a register whose prefix could patch the range [FD..LD]
   868:     // where VR and SrcR differ.
   869:     uint16_t FD = FDi, LD = LDi;  // Switch to unsigned type.
   870:     uint16_t MinL = LD-FD+1;
   871:     for (uint16_t L = MinL; L < W; ++L) {
   872:       LRSMapType::iterator F = LM.find(L);
   873:       if (F == LM.end())
   874:         continue;
   875:       RSListType &LL = F->second;
   876:       for (const auto &I : LL) {
   877:         uint16_t S = I.second;
   878:         // MinL is the minimum length of the prefix. Any length above MinL
   879:         // allows some flexibility as to where the prefix can start:
   880:         // given the extra length EL=L-MinL, the prefix must start between
   881:         // max(0,FD-EL) and FD.
   882:         if (S > FD)   // Starts too late.
   883:           continue;
   884:         uint16_t EL = L-MinL;
   885:         uint16_t LowS = (EL < FD) ? FD-EL : 0;
   886:         if (S < LowS) // Starts too early.
   887:           continue;
   888:         unsigned InsR = I.first;
   889:         if (!isValidInsertForm(VR, SrcR, InsR, L, S))
   890:           continue;
   891:         if (isDebug()) {
   892:           dbgs() << printReg(VR, HRI) << " = insert(" << printReg(SrcR, HRI)
   893:                  << ',' << printReg(InsR, HRI) << ",#" << L << ",#"
   894:                  << S << ")\n";
   895:         }
   896:         IFRecordWithRegSet RR(IFRecord(SrcR, InsR, L, S), RegisterSet());
   897:         IFMap[VR].push_back(RR);
   898:         Recorded = true;
   899:       }
   900:     }
```
- EN: It declares or implements routines such as begin, RCB, std::lower_bound, std::upper_bound, ... (11 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 begin, RCB, std::lower_bound, std::upper_bound, ... (11 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 901-1000 / 第 901-1000 行

```cpp
   901:   }
   902: 
   903:   return Recorded;
   904: }
   905: 
   906: void HexagonGenInsert::collectInBlock(MachineBasicBlock *B,
   907:       OrderedRegisterList &AVs) {
   908:   if (isDebug())
   909:     dbgs() << "visiting block " << printMBBReference(*B) << "\n";
   910: 
   911:   // First, check if this block is reachable at all. If not, the bit tracker
   912:   // will not have any information about registers in it.
   913:   if (!CMS->BT.reached(B))
   914:     return;
   915: 
   916:   bool DoConst = OptConst;
   917:   // Keep a separate set of registers defined in this block, so that we
   918:   // can remove them from the list of available registers once all DT
   919:   // successors have been processed.
   920:   RegisterSet BlockDefs, InsDefs;
   921:   for (MachineInstr &MI : *B) {
   922:     // Stop if the map size is too large.
   923:     if (IFMap.size() >= MaxIFMSize)
   924:       break;
   925: 
   926:     InsDefs.clear();
   927:     getInstrDefs(&MI, InsDefs);
   928:     // Leave those alone. They are more transparent than "insert".
   929:     bool Skip = MI.isCopy() || MI.isRegSequence();
   930: 
   931:     if (!Skip) {
   932:       // Visit all defined registers, and attempt to find the corresponding
   933:       // "insert" representations.
   934:       for (unsigned VR = InsDefs.find_first(); VR; VR = InsDefs.find_next(VR)) {
   935:         // Do not collect registers that are known to be compile-time cons-
   936:         // tants, unless requested.
   937:         if (!DoConst && isConstant(VR))
   938:           continue;
   939:         // If VR's cell contains a reference to VR, then VR cannot be defined
   940:         // via "insert". If VR is a constant that can be generated in a single
   941:         // instruction (without constant extenders), generating it via insert
   942:         // makes no sense.
   943:         if (findSelfReference(VR) || isSmallConstant(VR))
   944:           continue;
   945: 
   946:         findRecordInsertForms(VR, AVs);
   947:         // Stop if the map size is too large.
   948:         if (IFMap.size() >= MaxIFMSize)
   949:           break;
   950:       }
   951:     }
   952: 
   953:     // Insert the defined registers into the list of available registers
   954:     // after they have been processed.
   955:     for (unsigned VR = InsDefs.find_first(); VR; VR = InsDefs.find_next(VR))
   956:       AVs.insert(VR);
   957:     BlockDefs.insert(InsDefs);
   958:   }
   959: 
   960:   for (auto *DTN : children<MachineDomTreeNode*>(MDT->getNode(B))) {
   961:     MachineBasicBlock *SB = DTN->getBlock();
   962:     collectInBlock(SB, AVs);
   963:   }
   964: 
   965:   for (unsigned VR = BlockDefs.find_first(); VR; VR = BlockDefs.find_next(VR))
   966:     AVs.remove(VR);
   967: }
   968: 
   969: void HexagonGenInsert::findRemovableRegisters(unsigned VR, IFRecord IF,
   970:       RegisterSet &RMs) const {
   971:   // For a given register VR and a insert form, find the registers that are
   972:   // used by the current definition of VR, and which would no longer be
   973:   // needed for it after the definition of VR is replaced with the insert
   974:   // form. These are the registers that could potentially become dead.
   975:   RegisterSet Regs[2];
   976: 
   977:   unsigned S = 0;  // Register set selector.
   978:   Regs[S].insert(VR);
   979: 
   980:   while (!Regs[S].empty()) {
   981:     // Breadth-first search.
   982:     unsigned OtherS = 1-S;
   983:     Regs[OtherS].clear();
   984:     for (unsigned R = Regs[S].find_first(); R; R = Regs[S].find_next(R)) {
   985:       Regs[S].remove(R);
   986:       if (R == IF.SrcR || R == IF.InsR)
   987:         continue;
   988:       // Check if a given register has bits that are references to any other
   989:       // registers. This is to detect situations where the instruction that
   990:       // defines register R takes register Q as an operand, but R itself does
   991:       // not contain any bits from Q. Loads are examples of how this could
   992:       // happen:
   993:       //   R = load Q
   994:       // In this case (assuming we do not have any knowledge about the loaded
   995:       // value), we must not treat R as a "conveyance" of the bits from Q.
   996:       // (The information in BT about R's bits would have them as constants,
   997:       // in case of zero-extending loads, or refs to R.)
   998:       if (!findNonSelfReference(R))
   999:         continue;
  1000:       RMs.insert(R);
```
- EN: It declares or implements routines such as HexagonGenInsert::collectInBlock, clear, getInstrDefs, isCopy, ... (11 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonGenInsert, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonGenInsert::collectInBlock, clear, getInstrDefs, isCopy, ... (11 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonGenInsert，说明了它与同级后端组件的连接关系。

### Lines 1001-1100 / 第 1001-1100 行

```cpp
  1001:       const MachineInstr *DefI = MRI->getVRegDef(R);
  1002:       assert(DefI);
  1003:       // Do not iterate past PHI nodes to avoid infinite loops. This can
  1004:       // make the final set a bit less accurate, but the removable register
  1005:       // sets are an approximation anyway.
  1006:       if (DefI->isPHI())
  1007:         continue;
  1008:       getInstrUses(DefI, Regs[OtherS]);
  1009:     }
  1010:     S = OtherS;
  1011:   }
  1012:   // The register VR is added to the list as a side-effect of the algorithm,
  1013:   // but it is not "potentially removable". A potentially removable register
  1014:   // is one that may become unused (dead) after conversion to the insert form
  1015:   // IF, and obviously VR (or its replacement) will not become dead by apply-
  1016:   // ing IF.
  1017:   RMs.remove(VR);
  1018: }
  1019: 
  1020: void HexagonGenInsert::computeRemovableRegisters() {
  1021:   for (auto &I : IFMap) {
  1022:     IFListType &LL = I.second;
  1023:     for (auto &J : LL)
  1024:       findRemovableRegisters(I.first, J.first, J.second);
  1025:   }
  1026: }
  1027: 
  1028: void HexagonGenInsert::pruneEmptyLists() {
  1029:   // Remove all entries from the map, where the register has no insert forms
  1030:   // associated with it.
  1031:   using IterListType = SmallVector<IFMapType::iterator, 16>;
  1032:   IterListType Prune;
  1033:   for (IFMapType::iterator I = IFMap.begin(), E = IFMap.end(); I != E; ++I) {
  1034:     if (I->second.empty())
  1035:       Prune.push_back(I);
  1036:   }
  1037:   for (const auto &It : Prune)
  1038:     IFMap.erase(It);
  1039: }
  1040: 
  1041: void HexagonGenInsert::pruneCoveredSets(unsigned VR) {
  1042:   IFMapType::iterator F = IFMap.find(VR);
  1043:   assert(F != IFMap.end());
  1044:   IFListType &LL = F->second;
  1045: 
  1046:   // First, examine the IF candidates for register VR whose removable-regis-
  1047:   // ter sets are empty. This means that a given candidate will not help eli-
  1048:   // minate any registers, but since "insert" is not a constant-extendable
  1049:   // instruction, using such a candidate may reduce code size if the defini-
  1050:   // tion of VR is constant-extended.
  1051:   // If there exists a candidate with a non-empty set, the ones with empty
  1052:   // sets will not be used and can be removed.
  1053:   MachineInstr *DefVR = MRI->getVRegDef(VR);
  1054:   bool DefEx = HII->isConstExtended(*DefVR);
  1055:   bool HasNE = false;
  1056:   for (const auto &I : LL) {
  1057:     if (I.second.empty())
  1058:       continue;
  1059:     HasNE = true;
  1060:     break;
  1061:   }
  1062:   if (!DefEx || HasNE) {
  1063:     // The definition of VR is not constant-extended, or there is a candidate
  1064:     // with a non-empty set. Remove all candidates with empty sets.
  1065:     auto IsEmpty = [] (const IFRecordWithRegSet &IR) -> bool {
  1066:       return IR.second.empty();
  1067:     };
  1068:     llvm::erase_if(LL, IsEmpty);
  1069:   } else {
  1070:     // The definition of VR is constant-extended, and all candidates have
  1071:     // empty removable-register sets. Pick the maximum candidate, and remove
  1072:     // all others. The "maximum" does not have any special meaning here, it
  1073:     // is only so that the candidate that will remain on the list is selec-
  1074:     // ted deterministically.
  1075:     IFRecord MaxIF = LL[0].first;
  1076:     for (unsigned i = 1, n = LL.size(); i < n; ++i) {
  1077:       // If LL[MaxI] < LL[i], then MaxI = i.
  1078:       const IFRecord &IF = LL[i].first;
  1079:       unsigned M0 = BaseOrd[MaxIF.SrcR], M1 = BaseOrd[MaxIF.InsR];
  1080:       unsigned R0 = BaseOrd[IF.SrcR], R1 = BaseOrd[IF.InsR];
  1081:       if (M0 > R0)
  1082:         continue;
  1083:       if (M0 == R0) {
  1084:         if (M1 > R1)
  1085:           continue;
  1086:         if (M1 == R1) {
  1087:           if (MaxIF.Wdh > IF.Wdh)
  1088:             continue;
  1089:           if (MaxIF.Wdh == IF.Wdh && MaxIF.Off >= IF.Off)
  1090:             continue;
  1091:         }
  1092:       }
  1093:       // MaxIF < IF.
  1094:       MaxIF = IF;
  1095:     }
  1096:     // Remove everything except the maximum candidate. All register sets
  1097:     // are empty, so no need to preserve anything.
  1098:     LL.clear();
  1099:     LL.push_back(std::make_pair(MaxIF, RegisterSet()));
  1100:   }
```
- EN: It declares or implements routines such as getVRegDef, assert, getInstrUses, unused, ... (13 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonGenInsert, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getVRegDef, assert, getInstrUses, unused, ... (13 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonGenInsert，说明了它与同级后端组件的连接关系。

### Lines 1101-1200 / 第 1101-1200 行

```cpp
  1101: 
  1102:   // Now, remove those whose sets of potentially removable registers are
  1103:   // contained in another IF candidate for VR. For example, given these
  1104:   // candidates for %45,
  1105:   //   %45:
  1106:   //     (%44,%41,#9,#8), { %42 }
  1107:   //     (%43,%41,#9,#8), { %42 %44 }
  1108:   // remove the first one, since it is contained in the second one.
  1109:   for (unsigned i = 0, n = LL.size(); i < n; ) {
  1110:     const RegisterSet &RMi = LL[i].second;
  1111:     unsigned j = 0;
  1112:     while (j < n) {
  1113:       if (j != i && LL[j].second.includes(RMi))
  1114:         break;
  1115:       j++;
  1116:     }
  1117:     if (j == n) {   // RMi not contained in anything else.
  1118:       i++;
  1119:       continue;
  1120:     }
  1121:     LL.erase(LL.begin()+i);
  1122:     n = LL.size();
  1123:   }
  1124: }
  1125: 
  1126: void HexagonGenInsert::pruneUsesTooFar(unsigned VR, const UnsignedMap &RPO,
  1127:       PairMapType &M) {
  1128:   IFMapType::iterator F = IFMap.find(VR);
  1129:   assert(F != IFMap.end());
  1130:   IFListType &LL = F->second;
  1131:   unsigned Cutoff = VRegDistCutoff;
  1132:   const MachineInstr *DefV = MRI->getVRegDef(VR);
  1133: 
  1134:   for (unsigned i = LL.size(); i > 0; --i) {
  1135:     unsigned SR = LL[i-1].first.SrcR, IR = LL[i-1].first.InsR;
  1136:     const MachineInstr *DefS = MRI->getVRegDef(SR);
  1137:     const MachineInstr *DefI = MRI->getVRegDef(IR);
  1138:     unsigned DSV = distance(DefS, DefV, RPO, M);
  1139:     if (DSV < Cutoff) {
  1140:       unsigned DIV = distance(DefI, DefV, RPO, M);
  1141:       if (DIV < Cutoff)
  1142:         continue;
  1143:     }
  1144:     LL.erase(LL.begin()+(i-1));
  1145:   }
  1146: }
  1147: 
  1148: void HexagonGenInsert::pruneRegCopies(unsigned VR) {
  1149:   IFMapType::iterator F = IFMap.find(VR);
  1150:   assert(F != IFMap.end());
  1151:   IFListType &LL = F->second;
  1152: 
  1153:   auto IsCopy = [] (const IFRecordWithRegSet &IR) -> bool {
  1154:     return IR.first.Wdh == 32 && (IR.first.Off == 0 || IR.first.Off == 32);
  1155:   };
  1156:   llvm::erase_if(LL, IsCopy);
  1157: }
  1158: 
  1159: void HexagonGenInsert::pruneCandidates() {
  1160:   // Remove candidates that are not beneficial, regardless of the final
  1161:   // selection method.
  1162:   // First, remove candidates whose potentially removable set is a subset
  1163:   // of another candidate's set.
  1164:   for (const auto &I : IFMap)
  1165:     pruneCoveredSets(I.first);
  1166: 
  1167:   UnsignedMap RPO;
  1168: 
  1169:   using RPOTType = ReversePostOrderTraversal<const MachineFunction *>;
  1170: 
  1171:   RPOTType RPOT(MFN);
  1172:   unsigned RPON = 0;
  1173:   for (const auto &I : RPOT)
  1174:     RPO[I->getNumber()] = RPON++;
  1175: 
  1176:   PairMapType Memo; // Memoization map for distance calculation.
  1177:   // Remove candidates that would use registers defined too far away.
  1178:   for (const auto &I : IFMap)
  1179:     pruneUsesTooFar(I.first, RPO, Memo);
  1180: 
  1181:   pruneEmptyLists();
  1182: 
  1183:   for (const auto &I : IFMap)
  1184:     pruneRegCopies(I.first);
  1185: }
  1186: 
  1187: namespace {
  1188: 
  1189:   // Class for comparing IF candidates for registers that have multiple of
  1190:   // them. The smaller the candidate, according to this ordering, the better.
  1191:   // First, compare the number of zeros in the associated potentially remova-
  1192:   // ble register sets. "Zero" indicates that the register is very likely to
  1193:   // become dead after this transformation.
  1194:   // Second, compare "averages", i.e. use-count per size. The lower wins.
  1195:   // After that, it does not really matter which one is smaller. Resolve
  1196:   // the tie in some deterministic way.
  1197:   struct IFOrdering {
  1198:     IFOrdering(const UnsignedMap &UC, const RegisterOrdering &BO)
  1199:       : UseC(UC), BaseOrd(BO) {}
  1200: 
```
- EN: It declares types such as IFOrdering, which carry the state or API of this component. It declares or implements routines such as erase, size, HexagonGenInsert::pruneUsesTooFar, find, ... (13 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明了 IFOrdering 等类型，用来承载该组件的状态或接口。 这里声明或实现了 erase, size, HexagonGenInsert::pruneUsesTooFar, find, ... (13 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 1201-1300 / 第 1201-1300 行

```cpp
  1201:     bool operator() (const IFRecordWithRegSet &A,
  1202:                      const IFRecordWithRegSet &B) const;
  1203: 
  1204:   private:
  1205:     void stats(const RegisterSet &Rs, unsigned &Size, unsigned &Zero,
  1206:           unsigned &Sum) const;
  1207: 
  1208:     const UnsignedMap &UseC;
  1209:     const RegisterOrdering &BaseOrd;
  1210:   };
  1211: 
  1212: } // end anonymous namespace
  1213: 
  1214: bool IFOrdering::operator() (const IFRecordWithRegSet &A,
  1215:       const IFRecordWithRegSet &B) const {
  1216:   unsigned SizeA = 0, ZeroA = 0, SumA = 0;
  1217:   unsigned SizeB = 0, ZeroB = 0, SumB = 0;
  1218:   stats(A.second, SizeA, ZeroA, SumA);
  1219:   stats(B.second, SizeB, ZeroB, SumB);
  1220: 
  1221:   // We will pick the minimum element. The more zeros, the better.
  1222:   if (ZeroA != ZeroB)
  1223:     return ZeroA > ZeroB;
  1224:   // Compare SumA/SizeA with SumB/SizeB, lower is better.
  1225:   uint64_t AvgA = SumA*SizeB, AvgB = SumB*SizeA;
  1226:   if (AvgA != AvgB)
  1227:     return AvgA < AvgB;
  1228: 
  1229:   // The sets compare identical so far. Resort to comparing the IF records.
  1230:   // The actual values don't matter, this is only for determinism.
  1231:   unsigned OSA = BaseOrd[A.first.SrcR], OSB = BaseOrd[B.first.SrcR];
  1232:   if (OSA != OSB)
  1233:     return OSA < OSB;
  1234:   unsigned OIA = BaseOrd[A.first.InsR], OIB = BaseOrd[B.first.InsR];
  1235:   if (OIA != OIB)
  1236:     return OIA < OIB;
  1237:   if (A.first.Wdh != B.first.Wdh)
  1238:     return A.first.Wdh < B.first.Wdh;
  1239:   return A.first.Off < B.first.Off;
  1240: }
  1241: 
  1242: void IFOrdering::stats(const RegisterSet &Rs, unsigned &Size, unsigned &Zero,
  1243:       unsigned &Sum) const {
  1244:   for (unsigned R = Rs.find_first(); R; R = Rs.find_next(R)) {
  1245:     UnsignedMap::const_iterator F = UseC.find(R);
  1246:     assert(F != UseC.end());
  1247:     unsigned UC = F->second;
  1248:     if (UC == 0)
  1249:       Zero++;
  1250:     Sum += UC;
  1251:     Size++;
  1252:   }
  1253: }
  1254: 
  1255: void HexagonGenInsert::selectCandidates() {
  1256:   // Some registers may have multiple valid candidates. Pick the best one
  1257:   // (or decide not to use any).
  1258: 
  1259:   // Compute the "removability" measure of R:
  1260:   // For each potentially removable register R, record the number of regis-
  1261:   // ters with IF candidates, where R appears in at least one set.
  1262:   RegisterSet AllRMs;
  1263:   UnsignedMap UseC, RemC;
  1264:   IFMapType::iterator End = IFMap.end();
  1265: 
  1266:   for (IFMapType::iterator I = IFMap.begin(); I != End; ++I) {
  1267:     const IFListType &LL = I->second;
  1268:     RegisterSet TT;
  1269:     for (const auto &J : LL)
  1270:       TT.insert(J.second);
  1271:     for (unsigned R = TT.find_first(); R; R = TT.find_next(R))
  1272:       RemC[R]++;
  1273:     AllRMs.insert(TT);
  1274:   }
  1275: 
  1276:   for (unsigned R = AllRMs.find_first(); R; R = AllRMs.find_next(R)) {
  1277:     using use_iterator = MachineRegisterInfo::use_nodbg_iterator;
  1278:     using InstrSet = SmallPtrSet<const MachineInstr *, 16>;
  1279: 
  1280:     InstrSet UIs;
  1281:     // Count as the number of instructions in which R is used, not the
  1282:     // number of operands.
  1283:     use_iterator E = MRI->use_nodbg_end();
  1284:     for (use_iterator I = MRI->use_nodbg_begin(R); I != E; ++I)
  1285:       UIs.insert(I->getParent());
  1286:     unsigned C = UIs.size();
  1287:     // Calculate a measure, which is the number of instructions using R,
  1288:     // minus the "removability" count computed earlier.
  1289:     unsigned D = RemC[R];
  1290:     UseC[R] = (C > D) ? C-D : 0;  // doz
  1291:   }
  1292: 
  1293:   bool SelectAll0 = OptSelectAll0, SelectHas0 = OptSelectHas0;
  1294:   if (!SelectAll0 && !SelectHas0)
  1295:     SelectAll0 = true;
  1296: 
  1297:   // The smaller the number UseC for a given register R, the "less used"
  1298:   // R is aside from the opportunities for removal offered by generating
  1299:   // "insert" instructions.
  1300:   // Iterate over the IF map, and for those registers that have multiple
```
- EN: It opens namespaces (bool) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as stats, IFOrdering::stats, find_next, find, ... (10 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里打开了命名空间（bool），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 stats, IFOrdering::stats, find_next, find, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 1301-1400 / 第 1301-1400 行

```cpp
  1301:   // candidates, pick the minimum one according to IFOrdering.
  1302:   IFOrdering IFO(UseC, BaseOrd);
  1303:   for (IFMapType::iterator I = IFMap.begin(); I != End; ++I) {
  1304:     IFListType &LL = I->second;
  1305:     if (LL.empty())
  1306:       continue;
  1307:     // Get the minimum element, remember it and clear the list. If the
  1308:     // element found is adequate, we will put it back on the list, other-
  1309:     // wise the list will remain empty, and the entry for this register
  1310:     // will be removed (i.e. this register will not be replaced by insert).
  1311:     IFListType::iterator MinI = llvm::min_element(LL, IFO);
  1312:     assert(MinI != LL.end());
  1313:     IFRecordWithRegSet M = *MinI;
  1314:     LL.clear();
  1315: 
  1316:     // We want to make sure that this replacement will have a chance to be
  1317:     // beneficial, and that means that we want to have indication that some
  1318:     // register will be removed. The most likely registers to be eliminated
  1319:     // are the use operands in the definition of I->first. Accept/reject a
  1320:     // candidate based on how many of its uses it can potentially eliminate.
  1321: 
  1322:     RegisterSet Us;
  1323:     const MachineInstr *DefI = MRI->getVRegDef(I->first);
  1324:     getInstrUses(DefI, Us);
  1325:     bool Accept = false;
  1326: 
  1327:     if (SelectAll0) {
  1328:       bool All0 = true;
  1329:       for (unsigned R = Us.find_first(); R; R = Us.find_next(R)) {
  1330:         if (UseC[R] == 0)
  1331:           continue;
  1332:         All0 = false;
  1333:         break;
  1334:       }
  1335:       Accept = All0;
  1336:     } else if (SelectHas0) {
  1337:       bool Has0 = false;
  1338:       for (unsigned R = Us.find_first(); R; R = Us.find_next(R)) {
  1339:         if (UseC[R] != 0)
  1340:           continue;
  1341:         Has0 = true;
  1342:         break;
  1343:       }
  1344:       Accept = Has0;
  1345:     }
  1346:     if (Accept)
  1347:       LL.push_back(M);
  1348:   }
  1349: 
  1350:   // Remove candidates that add uses of removable registers, unless the
  1351:   // removable registers are among replacement candidates.
  1352:   // Recompute the removable registers, since some candidates may have
  1353:   // been eliminated.
  1354:   AllRMs.clear();
  1355:   for (IFMapType::iterator I = IFMap.begin(); I != End; ++I) {
  1356:     const IFListType &LL = I->second;
  1357:     if (!LL.empty())
  1358:       AllRMs.insert(LL[0].second);
  1359:   }
  1360:   for (IFMapType::iterator I = IFMap.begin(); I != End; ++I) {
  1361:     IFListType &LL = I->second;
  1362:     if (LL.empty())
  1363:       continue;
  1364:     unsigned SR = LL[0].first.SrcR, IR = LL[0].first.InsR;
  1365:     if (AllRMs[SR] || AllRMs[IR])
  1366:       LL.clear();
  1367:   }
  1368: 
  1369:   pruneEmptyLists();
  1370: }
  1371: 
  1372: bool HexagonGenInsert::generateInserts() {
  1373:   // Create a new register for each one from IFMap, and store them in the
  1374:   // map.
  1375:   UnsignedMap RegMap;
  1376:   for (auto &I : IFMap) {
  1377:     unsigned VR = I.first;
  1378:     const TargetRegisterClass *RC = MRI->getRegClass(VR);
  1379:     Register NewVR = MRI->createVirtualRegister(RC);
  1380:     RegMap[VR] = NewVR;
  1381:   }
  1382: 
  1383:   // We can generate the "insert" instructions using potentially stale re-
  1384:   // gisters: SrcR and InsR for a given VR may be among other registers that
  1385:   // are also replaced. This is fine, we will do the mass "rauw" a bit later.
  1386:   for (auto &I : IFMap) {
  1387:     MachineInstr *MI = MRI->getVRegDef(I.first);
  1388:     MachineBasicBlock &B = *MI->getParent();
  1389:     DebugLoc DL = MI->getDebugLoc();
  1390:     unsigned NewR = RegMap[I.first];
  1391:     bool R32 = MRI->getRegClass(NewR) == &Hexagon::IntRegsRegClass;
  1392:     const MCInstrDesc &D = R32 ? HII->get(Hexagon::S2_insert)
  1393:                                : HII->get(Hexagon::S2_insertp);
  1394:     IFRecord IF = I.second[0].first;
  1395:     unsigned Wdh = IF.Wdh, Off = IF.Off;
  1396:     unsigned InsS = 0;
  1397:     if (R32 && MRI->getRegClass(IF.InsR) == &Hexagon::DoubleRegsRegClass) {
  1398:       InsS = Hexagon::isub_lo;
  1399:       if (Off >= 32) {
  1400:         InsS = Hexagon::isub_hi;
```
- EN: It declares or implements routines such as IFO, removed, assert, clear, ... (14 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonGenInsert, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 IFO, removed, assert, clear, ... (14 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonGenInsert，说明了它与同级后端组件的连接关系。

### Lines 1401-1500 / 第 1401-1500 行

```cpp
  1401:         Off -= 32;
  1402:       }
  1403:     }
  1404:     // Advance to the proper location for inserting instructions. This could
  1405:     // be B.end().
  1406:     MachineBasicBlock::iterator At = MI;
  1407:     if (MI->isPHI())
  1408:       At = B.getFirstNonPHI();
  1409: 
  1410:     BuildMI(B, At, DL, D, NewR)
  1411:         .addReg(IF.SrcR)
  1412:         .addReg(IF.InsR, {}, InsS)
  1413:         .addImm(Wdh)
  1414:         .addImm(Off);
  1415: 
  1416:     MRI->clearKillFlags(IF.SrcR);
  1417:     MRI->clearKillFlags(IF.InsR);
  1418:   }
  1419: 
  1420:   for (const auto &I : IFMap) {
  1421:     MachineInstr *DefI = MRI->getVRegDef(I.first);
  1422:     MRI->replaceRegWith(I.first, RegMap[I.first]);
  1423:     DefI->eraseFromParent();
  1424:   }
  1425: 
  1426:   return true;
  1427: }
  1428: 
  1429: bool HexagonGenInsert::removeDeadCode(MachineDomTreeNode *N) {
  1430:   bool Changed = false;
  1431: 
  1432:   for (auto *DTN : children<MachineDomTreeNode*>(N))
  1433:     Changed |= removeDeadCode(DTN);
  1434: 
  1435:   MachineBasicBlock *B = N->getBlock();
  1436:   std::vector<MachineInstr*> Instrs;
  1437:   for (MachineInstr &MI : llvm::reverse(*B))
  1438:     Instrs.push_back(&MI);
  1439: 
  1440:   for (MachineInstr *MI : Instrs) {
  1441:     unsigned Opc = MI->getOpcode();
  1442:     // Do not touch lifetime markers. This is why the target-independent DCE
  1443:     // cannot be used.
  1444:     if (Opc == TargetOpcode::LIFETIME_START ||
  1445:         Opc == TargetOpcode::LIFETIME_END)
  1446:       continue;
  1447:     bool Store = false;
  1448:     if (MI->isInlineAsm() || !MI->isSafeToMove(Store))
  1449:       continue;
  1450: 
  1451:     bool AllDead = true;
  1452:     SmallVector<unsigned,2> Regs;
  1453:     for (const MachineOperand &MO : MI->operands()) {
  1454:       if (!MO.isReg() || !MO.isDef())
  1455:         continue;
  1456:       Register R = MO.getReg();
  1457:       if (!R.isVirtual() || !MRI->use_nodbg_empty(R)) {
  1458:         AllDead = false;
  1459:         break;
  1460:       }
  1461:       Regs.push_back(R);
  1462:     }
  1463:     if (!AllDead)
  1464:       continue;
  1465: 
  1466:     B->erase(MI);
  1467:     for (unsigned Reg : Regs)
  1468:       MRI->markUsesInDebugValueAsUndef(Reg);
  1469:     Changed = true;
  1470:   }
  1471: 
  1472:   return Changed;
  1473: }
  1474: 
  1475: bool HexagonGenInsert::runOnMachineFunction(MachineFunction &MF) {
  1476:   if (skipFunction(MF.getFunction()))
  1477:     return false;
  1478: 
  1479:   bool Timing = OptTiming, TimingDetail = Timing && OptTimingDetail;
  1480:   bool Changed = false;
  1481: 
  1482:   // Verify: one, but not both.
  1483:   assert(!OptSelectAll0 || !OptSelectHas0);
  1484: 
  1485:   IFMap.clear();
  1486:   BaseOrd.clear();
  1487:   CellOrd.clear();
  1488: 
  1489:   const auto &ST = MF.getSubtarget<HexagonSubtarget>();
  1490:   HII = ST.getInstrInfo();
  1491:   HRI = ST.getRegisterInfo();
  1492:   MFN = &MF;
  1493:   MRI = &MF.getRegInfo();
  1494:   MDT = &getAnalysis<MachineDominatorTreeWrapperPass>().getDomTree();
  1495: 
  1496:   // Clean up before any further processing, so that dead code does not
  1497:   // get used in a newly generated "insert" instruction. Have a custom
  1498:   // version of DCE that preserves lifetime markers. Without it, merging
  1499:   // of stack objects can fail to recognize and merge disjoint objects
  1500:   // leading to unnecessary stack growth.
```
- EN: It declares or implements routines such as addImm, clearKillFlags, getVRegDef, replaceRegWith, ... (19 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonGenInsert, HexagonSubtarget, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 addImm, clearKillFlags, getVRegDef, replaceRegWith, ... (19 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonGenInsert, HexagonSubtarget，说明了它与同级后端组件的连接关系。

### Lines 1501-1600 / 第 1501-1600 行

```cpp
  1501:   Changed = removeDeadCode(MDT->getRootNode());
  1502: 
  1503:   const HexagonEvaluator HE(*HRI, *MRI, *HII, MF);
  1504:   BitTracker BTLoc(HE, MF);
  1505:   BTLoc.trace(isDebug());
  1506:   BTLoc.run();
  1507:   CellMapShadow MS(BTLoc);
  1508:   CMS = &MS;
  1509: 
  1510:   buildOrderingMF(BaseOrd);
  1511:   buildOrderingBT(BaseOrd, CellOrd);
  1512: 
  1513:   if (isDebug()) {
  1514:     dbgs() << "Cell ordering:\n";
  1515:     for (const auto &I : CellOrd) {
  1516:       unsigned VR = I.first, Pos = I.second;
  1517:       dbgs() << printReg(VR, HRI) << " -> " << Pos << "\n";
  1518:     }
  1519:   }
  1520: 
  1521:   // Collect candidates for conversion into the insert forms.
  1522:   MachineBasicBlock *RootB = MDT->getRoot();
  1523:   OrderedRegisterList AvailR(CellOrd);
  1524: 
  1525:   const char *const TGName = "hexinsert";
  1526:   const char *const TGDesc = "Generate Insert Instructions";
  1527: 
  1528:   {
  1529:     NamedRegionTimer _T("collection", "collection", TGName, TGDesc,
  1530:                         TimingDetail);
  1531:     collectInBlock(RootB, AvailR);
  1532:     // Complete the information gathered in IFMap.
  1533:     computeRemovableRegisters();
  1534:   }
  1535: 
  1536:   if (isDebug()) {
  1537:     dbgs() << "Candidates after collection:\n";
  1538:     dump_map();
  1539:   }
  1540: 
  1541:   if (IFMap.empty())
  1542:     return Changed;
  1543: 
  1544:   {
  1545:     NamedRegionTimer _T("pruning", "pruning", TGName, TGDesc, TimingDetail);
  1546:     pruneCandidates();
  1547:   }
  1548: 
  1549:   if (isDebug()) {
  1550:     dbgs() << "Candidates after pruning:\n";
  1551:     dump_map();
  1552:   }
  1553: 
  1554:   if (IFMap.empty())
  1555:     return Changed;
  1556: 
  1557:   {
  1558:     NamedRegionTimer _T("selection", "selection", TGName, TGDesc, TimingDetail);
  1559:     selectCandidates();
  1560:   }
  1561: 
  1562:   if (isDebug()) {
  1563:     dbgs() << "Candidates after selection:\n";
  1564:     dump_map();
  1565:   }
  1566: 
  1567:   // Filter out vregs beyond the cutoff.
  1568:   if (VRegIndexCutoff.getPosition()) {
  1569:     unsigned Cutoff = VRegIndexCutoff;
  1570: 
  1571:     using IterListType = SmallVector<IFMapType::iterator, 16>;
  1572: 
  1573:     IterListType Out;
  1574:     for (IFMapType::iterator I = IFMap.begin(), E = IFMap.end(); I != E; ++I) {
  1575:       unsigned Idx = Register(I->first).virtRegIndex();
  1576:       if (Idx >= Cutoff)
  1577:         Out.push_back(I);
  1578:     }
  1579:     for (const auto &It : Out)
  1580:       IFMap.erase(It);
  1581:   }
  1582:   if (IFMap.empty())
  1583:     return Changed;
  1584: 
  1585:   {
  1586:     NamedRegionTimer _T("generation", "generation", TGName, TGDesc,
  1587:                         TimingDetail);
  1588:     generateInserts();
  1589:   }
  1590: 
  1591:   return true;
  1592: }
  1593: 
  1594: FunctionPass *llvm::createHexagonGenInsert() {
  1595:   return new HexagonGenInsert();
  1596: }
  1597: 
  1598: //===----------------------------------------------------------------------===//
  1599: //                         Public Constructor Functions
  1600: //===----------------------------------------------------------------------===//
```
- EN: It declares or implements routines such as removeDeadCode, HE, BTLoc, trace, ... (20 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonEvaluator, HexagonGenInsert, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 removeDeadCode, HE, BTLoc, trace, ... (20 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonEvaluator, HexagonGenInsert，说明了它与同级后端组件的连接关系。

### Lines 1601-1606 / 第 1601-1606 行

```cpp
  1601: 
  1602: INITIALIZE_PASS_BEGIN(HexagonGenInsert, "hexinsert",
  1603:   "Hexagon generate \"insert\" instructions", false, false)
  1604: INITIALIZE_PASS_DEPENDENCY(MachineDominatorTreeWrapperPass)
  1605: INITIALIZE_PASS_END(HexagonGenInsert, "hexinsert",
  1606:   "Hexagon generate \"insert\" instructions", false, false)
```
- EN: Notable Hexagon symbols referenced here include HexagonGenInsert, showing how the code connects to sibling backend components.
- CN: 这里引用的重要 Hexagon 符号包括 HexagonGenInsert，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- instruction semantics / 指令语义
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `BitTracker.h, Hexagon.h, HexagonBitTracker.h, HexagonInstrInfo.h, HexagonRegisterInfo.h, HexagonSubtarget.h, llvm/ADT/BitVector.h, llvm/ADT/DenseMap.h, llvm/ADT/GraphTraits.h, llvm/ADT/PostOrderIterator.h, ... (36 total)`
- Hexagon symbols / Hexagon 符号: `HexagonGenInsert, HexagonBitTracker, HexagonInstrInfo, HexagonRegisterInfo, HexagonSubtarget, HexagonEvaluator`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
