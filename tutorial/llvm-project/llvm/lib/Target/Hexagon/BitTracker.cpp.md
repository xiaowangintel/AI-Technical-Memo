# BitTracker.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/BitTracker.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon bit-level dataflow/value tracking.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-100 / 第 1-100 行

```cpp
     1: //===- BitTracker.cpp -----------------------------------------------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: 
     9: // SSA-based bit propagation.
    10: //
    11: // The purpose of this code is, for a given virtual register, to provide
    12: // information about the value of each bit in the register. The values
    13: // of bits are represented by the class BitValue, and take one of four
    14: // cases: 0, 1, "ref" and "bottom". The 0 and 1 are rather clear, the
    15: // "ref" value means that the bit is a copy of another bit (which itself
    16: // cannot be a copy of yet another bit---such chains are not allowed).
    17: // A "ref" value is associated with a BitRef structure, which indicates
    18: // which virtual register, and which bit in that register is the origin
    19: // of the value. For example, given an instruction
    20: //   %2 = ASL %1, 1
    21: // assuming that nothing is known about bits of %1, bit 1 of %2
    22: // will be a "ref" to (%1, 0). If there is a subsequent instruction
    23: //   %3 = ASL %2, 2
    24: // then bit 3 of %3 will be a "ref" to (%1, 0) as well.
    25: // The "bottom" case means that the bit's value cannot be determined,
    26: // and that this virtual register actually defines it. The "bottom" case
    27: // is discussed in detail in BitTracker.h. In fact, "bottom" is a "ref
    28: // to self", so for the %1 above, the bit 0 of it will be a "ref" to
    29: // (%1, 0), bit 1 will be a "ref" to (%1, 1), etc.
    30: //
    31: // The tracker implements the Wegman-Zadeck algorithm, originally developed
    32: // for SSA-based constant propagation. Each register is represented as
    33: // a sequence of bits, with the convention that bit 0 is the least signi-
    34: // ficant bit. Each bit is propagated individually. The class RegisterCell
    35: // implements the register's representation, and is also the subject of
    36: // the lattice operations in the tracker.
    37: //
    38: // The intended usage of the bit tracker is to create a target-specific
    39: // machine instruction evaluator, pass the evaluator to the BitTracker
    40: // object, and run the tracker. The tracker will then collect the bit
    41: // value information for a given machine function. After that, it can be
    42: // queried for the cells for each virtual register.
    43: // Sample code:
    44: //   const TargetSpecificEvaluator TSE(TRI, MRI);
    45: //   BitTracker BT(TSE, MF);
    46: //   BT.run();
    47: //   ...
    48: //   unsigned Reg = interestingRegister();
    49: //   RegisterCell RC = BT.get(Reg);
    50: //   if (RC[3].is(1))
    51: //      Reg0bit3 = 1;
    52: //
    53: // The code below is intended to be fully target-independent.
    54: 
    55: #include "BitTracker.h"
    56: #include "llvm/ADT/APInt.h"
    57: #include "llvm/ADT/BitVector.h"
    58: #include "llvm/CodeGen/MachineBasicBlock.h"
    59: #include "llvm/CodeGen/MachineFunction.h"
    60: #include "llvm/CodeGen/MachineInstr.h"
    61: #include "llvm/CodeGen/MachineOperand.h"
    62: #include "llvm/CodeGen/MachineRegisterInfo.h"
    63: #include "llvm/IR/Constants.h"
    64: #include "llvm/Support/Debug.h"
    65: 
    66: using namespace llvm;
    67: 
    68: using BT = BitTracker;
    69: 
    70: namespace {
    71: 
    72:   // Local trickery to pretty print a register (without the whole "%number"
    73:   // business).
    74:   struct printv {
    75:     printv(unsigned r) : R(r) {}
    76: 
    77:     unsigned R;
    78:   };
    79: 
    80:   raw_ostream &operator<< (raw_ostream &OS, const printv &PV) {
    81:     if (PV.R)
    82:       OS << 'v' << Register(PV.R).virtRegIndex();
    83:     else
    84:       OS << 's';
    85:     return OS;
    86:   }
    87: 
    88: } // end anonymous namespace
    89: 
    90: namespace llvm {
    91: 
    92:   raw_ostream &operator<<(raw_ostream &OS, const BT::BitValue &BV) {
    93:     switch (BV.Type) {
    94:       case BT::BitValue::Top:
    95:         OS << 'T';
    96:         break;
    97:       case BT::BitValue::Zero:
    98:         OS << '0';
    99:         break;
   100:       case BT::BitValue::One:
```
- EN: This range is dominated by comments or banner text that documents the surrounding section. It imports headers such as BitTracker.h, llvm/ADT/APInt.h, llvm/ADT/BitVector.h, llvm/CodeGen/MachineBasicBlock.h, ... (10 total), establishing the LLVM/Hexagon APIs used below. It opens namespaces (llvm, namespace) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as printv, which carry the state or API of this component.
- CN: 这一段主要由注释或分隔说明组成，用来解释后续代码区域。 这里引入了 BitTracker.h, llvm/ADT/APInt.h, llvm/ADT/BitVector.h, llvm/CodeGen/MachineBasicBlock.h, ... (10 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这里打开了命名空间（llvm, namespace），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 printv 等类型，用来承载该组件的状态或接口。

### Lines 101-200 / 第 101-200 行

```cpp
   101:         OS << '1';
   102:         break;
   103:       case BT::BitValue::Ref:
   104:         OS << printv(BV.RefI.Reg) << '[' << BV.RefI.Pos << ']';
   105:         break;
   106:     }
   107:     return OS;
   108:   }
   109: 
   110:   raw_ostream &operator<<(raw_ostream &OS, const BT::RegisterCell &RC) {
   111:     unsigned n = RC.Bits.size();
   112:     OS << "{ w:" << n;
   113:     // Instead of printing each bit value individually, try to group them
   114:     // into logical segments, such as sequences of 0 or 1 bits or references
   115:     // to consecutive bits (e.g. "bits 3-5 are same as bits 7-9 of reg xyz").
   116:     // "Start" will be the index of the beginning of the most recent segment.
   117:     unsigned Start = 0;
   118:     bool SeqRef = false;    // A sequence of refs to consecutive bits.
   119:     bool ConstRef = false;  // A sequence of refs to the same bit.
   120: 
   121:     for (unsigned i = 1, n = RC.Bits.size(); i < n; ++i) {
   122:       const BT::BitValue &V = RC[i];
   123:       const BT::BitValue &SV = RC[Start];
   124:       bool IsRef = (V.Type == BT::BitValue::Ref);
   125:       // If the current value is the same as Start, skip to the next one.
   126:       if (!IsRef && V == SV)
   127:         continue;
   128:       if (IsRef && SV.Type == BT::BitValue::Ref && V.RefI.Reg == SV.RefI.Reg) {
   129:         if (Start+1 == i) {
   130:           SeqRef = (V.RefI.Pos == SV.RefI.Pos+1);
   131:           ConstRef = (V.RefI.Pos == SV.RefI.Pos);
   132:         }
   133:         if (SeqRef && V.RefI.Pos == SV.RefI.Pos+(i-Start))
   134:           continue;
   135:         if (ConstRef && V.RefI.Pos == SV.RefI.Pos)
   136:           continue;
   137:       }
   138: 
   139:       // The current value is different. Print the previous one and reset
   140:       // the Start.
   141:       OS << " [" << Start;
   142:       unsigned Count = i - Start;
   143:       if (Count == 1) {
   144:         OS << "]:" << SV;
   145:       } else {
   146:         OS << '-' << i-1 << "]:";
   147:         if (SV.Type == BT::BitValue::Ref && SeqRef)
   148:           OS << printv(SV.RefI.Reg) << '[' << SV.RefI.Pos << '-'
   149:              << SV.RefI.Pos+(Count-1) << ']';
   150:         else
   151:           OS << SV;
   152:       }
   153:       Start = i;
   154:       SeqRef = ConstRef = false;
   155:     }
   156: 
   157:     OS << " [" << Start;
   158:     unsigned Count = n - Start;
   159:     if (n-Start == 1) {
   160:       OS << "]:" << RC[Start];
   161:     } else {
   162:       OS << '-' << n-1 << "]:";
   163:       const BT::BitValue &SV = RC[Start];
   164:       if (SV.Type == BT::BitValue::Ref && SeqRef)
   165:         OS << printv(SV.RefI.Reg) << '[' << SV.RefI.Pos << '-'
   166:            << SV.RefI.Pos+(Count-1) << ']';
   167:       else
   168:         OS << SV;
   169:     }
   170:     OS << " }";
   171: 
   172:     return OS;
   173:   }
   174: 
   175: } // end namespace llvm
   176: 
   177: void BitTracker::print_cells(raw_ostream &OS) const {
   178:   for (const std::pair<unsigned, RegisterCell> P : Map)
   179:     dbgs() << printReg(P.first, &ME.TRI) << " -> " << P.second << "\n";
   180: }
   181: 
   182: BitTracker::BitTracker(const MachineEvaluator &E, MachineFunction &F)
   183:     : ME(E), MF(F), MRI(F.getRegInfo()), Map(*new CellMapType), Trace(false) {
   184: }
   185: 
   186: BitTracker::~BitTracker() {
   187:   delete &Map;
   188: }
   189: 
   190: // If we were allowed to update a cell for a part of a register, the meet
   191: // operation would need to be parametrized by the register number and the
   192: // exact part of the register, so that the computer BitRefs correspond to
   193: // the actual bits of the "self" register.
   194: // While this cannot happen in the current implementation, I'm not sure
   195: // if this should be ruled out in the future.
   196: bool BT::RegisterCell::meet(const RegisterCell &RC, Register SelfR) {
   197:   // An example when "meet" can be invoked with SelfR == 0 is a phi node
   198:   // with a physical register as an operand.
   199:   assert(SelfR == 0 || SelfR.isVirtual());
   200:   bool Changed = false;
```
- EN: It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as size, BitTracker::print_cells, BitTracker::BitTracker, BitTracker::~BitTracker, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 size, BitTracker::print_cells, BitTracker::BitTracker, BitTracker::~BitTracker, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 201-300 / 第 201-300 行

```cpp
   201:   for (uint16_t i = 0, n = Bits.size(); i < n; ++i) {
   202:     const BitValue &RCV = RC[i];
   203:     Changed |= Bits[i].meet(RCV, BitRef(SelfR, i));
   204:   }
   205:   return Changed;
   206: }
   207: 
   208: // Insert the entire cell RC into the current cell at position given by M.
   209: BT::RegisterCell &BT::RegisterCell::insert(const BT::RegisterCell &RC,
   210:       const BitMask &M) {
   211:   uint16_t B = M.first(), E = M.last(), W = width();
   212:   // M must be a valid mask for *this.
   213:   assert(B < W && E < W);
   214:   // The masked part of *this must have the same number of bits
   215:   // as the source.
   216:   assert(B > E || E-B+1 == RC.width());      // B <= E  =>  E-B+1 = |RC|.
   217:   assert(B <= E || E+(W-B)+1 == RC.width()); // E < B   =>  E+(W-B)+1 = |RC|.
   218:   if (B <= E) {
   219:     for (uint16_t i = 0; i <= E-B; ++i)
   220:       Bits[i+B] = RC[i];
   221:   } else {
   222:     for (uint16_t i = 0; i < W-B; ++i)
   223:       Bits[i+B] = RC[i];
   224:     for (uint16_t i = 0; i <= E; ++i)
   225:       Bits[i] = RC[i+(W-B)];
   226:   }
   227:   return *this;
   228: }
   229: 
   230: BT::RegisterCell BT::RegisterCell::extract(const BitMask &M) const {
   231:   uint16_t B = M.first(), E = M.last(), W = width();
   232:   assert(B < W && E < W);
   233:   if (B <= E) {
   234:     RegisterCell RC(E-B+1);
   235:     for (uint16_t i = B; i <= E; ++i)
   236:       RC.Bits[i-B] = Bits[i];
   237:     return RC;
   238:   }
   239: 
   240:   RegisterCell RC(E+(W-B)+1);
   241:   for (uint16_t i = 0; i < W-B; ++i)
   242:     RC.Bits[i] = Bits[i+B];
   243:   for (uint16_t i = 0; i <= E; ++i)
   244:     RC.Bits[i+(W-B)] = Bits[i];
   245:   return RC;
   246: }
   247: 
   248: BT::RegisterCell &BT::RegisterCell::rol(uint16_t Sh) {
   249:   // Rotate left (i.e. towards increasing bit indices).
   250:   // Swap the two parts:  [0..W-Sh-1] [W-Sh..W-1]
   251:   uint16_t W = width();
   252:   Sh = Sh % W;
   253:   if (Sh == 0)
   254:     return *this;
   255: 
   256:   RegisterCell Tmp(W-Sh);
   257:   // Tmp = [0..W-Sh-1].
   258:   for (uint16_t i = 0; i < W-Sh; ++i)
   259:     Tmp[i] = Bits[i];
   260:   // Shift [W-Sh..W-1] to [0..Sh-1].
   261:   for (uint16_t i = 0; i < Sh; ++i)
   262:     Bits[i] = Bits[W-Sh+i];
   263:   // Copy Tmp to [Sh..W-1].
   264:   for (uint16_t i = 0; i < W-Sh; ++i)
   265:     Bits[i+Sh] = Tmp.Bits[i];
   266:   return *this;
   267: }
   268: 
   269: BT::RegisterCell &BT::RegisterCell::fill(uint16_t B, uint16_t E,
   270:       const BitValue &V) {
   271:   assert(B <= E);
   272:   while (B < E)
   273:     Bits[B++] = V;
   274:   return *this;
   275: }
   276: 
   277: BT::RegisterCell &BT::RegisterCell::cat(const RegisterCell &RC) {
   278:   // Append the cell given as the argument to the "this" cell.
   279:   // Bit 0 of RC becomes bit W of the result, where W is this->width().
   280:   uint16_t W = width(), WRC = RC.width();
   281:   Bits.resize(W+WRC);
   282:   for (uint16_t i = 0; i < WRC; ++i)
   283:     Bits[i+W] = RC.Bits[i];
   284:   return *this;
   285: }
   286: 
   287: uint16_t BT::RegisterCell::ct(bool B) const {
   288:   uint16_t W = width();
   289:   uint16_t C = 0;
   290:   BitValue V = B;
   291:   while (C < W && Bits[C] == V)
   292:     C++;
   293:   return C;
   294: }
   295: 
   296: uint16_t BT::RegisterCell::cl(bool B) const {
   297:   uint16_t W = width();
   298:   uint16_t C = 0;
   299:   BitValue V = B;
   300:   while (C < W && Bits[W-(C+1)] == V)
```
- EN: It declares or implements routines such as meet, BT::RegisterCell::insert, first, assert, ... (15 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 meet, BT::RegisterCell::insert, first, assert, ... (15 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 301-400 / 第 301-400 行

```cpp
   301:     C++;
   302:   return C;
   303: }
   304: 
   305: bool BT::RegisterCell::operator== (const RegisterCell &RC) const {
   306:   uint16_t W = Bits.size();
   307:   if (RC.Bits.size() != W)
   308:     return false;
   309:   for (uint16_t i = 0; i < W; ++i)
   310:     if (Bits[i] != RC[i])
   311:       return false;
   312:   return true;
   313: }
   314: 
   315: BT::RegisterCell &BT::RegisterCell::regify(unsigned R) {
   316:   for (unsigned i = 0, n = width(); i < n; ++i) {
   317:     const BitValue &V = Bits[i];
   318:     if (V.Type == BitValue::Ref && V.RefI.Reg == 0)
   319:       Bits[i].RefI = BitRef(R, i);
   320:   }
   321:   return *this;
   322: }
   323: 
   324: uint16_t BT::MachineEvaluator::getRegBitWidth(const RegisterRef &RR) const {
   325:   // The general problem is with finding a register class that corresponds
   326:   // to a given reference reg:sub. There can be several such classes, and
   327:   // since we only care about the register size, it does not matter which
   328:   // such class we would find.
   329:   // The easiest way to accomplish what we want is to
   330:   // 1. find a physical register PhysR from the same class as RR.Reg,
   331:   // 2. find a physical register PhysS that corresponds to PhysR:RR.Sub,
   332:   // 3. find a register class that contains PhysS.
   333:   if (RR.Reg.isVirtual()) {
   334:     const auto &VC = composeWithSubRegIndex(*MRI.getRegClass(RR.Reg), RR.Sub);
   335:     return TRI.getRegSizeInBits(VC);
   336:   }
   337:   assert(RR.Reg.isPhysical());
   338:   MCRegister PhysR =
   339:       (RR.Sub == 0) ? RR.Reg.asMCReg() : TRI.getSubReg(RR.Reg, RR.Sub);
   340:   return getPhysRegBitWidth(PhysR);
   341: }
   342: 
   343: BT::RegisterCell BT::MachineEvaluator::getCell(const RegisterRef &RR,
   344:       const CellMapType &M) const {
   345:   uint16_t BW = getRegBitWidth(RR);
   346: 
   347:   // Physical registers are assumed to be present in the map with an unknown
   348:   // value. Don't actually insert anything in the map, just return the cell.
   349:   if (RR.Reg.isPhysical())
   350:     return RegisterCell::self(0, BW);
   351: 
   352:   assert(RR.Reg.isVirtual());
   353:   // For virtual registers that belong to a class that is not tracked,
   354:   // generate an "unknown" value as well.
   355:   const TargetRegisterClass *C = MRI.getRegClass(RR.Reg);
   356:   if (!track(C))
   357:     return RegisterCell::self(0, BW);
   358: 
   359:   CellMapType::const_iterator F = M.find(RR.Reg);
   360:   if (F != M.end()) {
   361:     if (!RR.Sub)
   362:       return F->second;
   363:     BitMask M = mask(RR.Reg, RR.Sub);
   364:     return F->second.extract(M);
   365:   }
   366:   // If not found, create a "top" entry, but do not insert it in the map.
   367:   return RegisterCell::top(BW);
   368: }
   369: 
   370: void BT::MachineEvaluator::putCell(const RegisterRef &RR, RegisterCell RC,
   371:       CellMapType &M) const {
   372:   // While updating the cell map can be done in a meaningful way for
   373:   // a part of a register, it makes little sense to implement it as the
   374:   // SSA representation would never contain such "partial definitions".
   375:   if (!RR.Reg.isVirtual())
   376:     return;
   377:   assert(RR.Sub == 0 && "Unexpected sub-register in definition");
   378:   // Eliminate all ref-to-reg-0 bit values: replace them with "self".
   379:   M[RR.Reg] = RC.regify(RR.Reg);
   380: }
   381: 
   382: // Check if the cell represents a compile-time integer value.
   383: bool BT::MachineEvaluator::isInt(const RegisterCell &A) const {
   384:   uint16_t W = A.width();
   385:   for (uint16_t i = 0; i < W; ++i)
   386:     if (!A[i].is(0) && !A[i].is(1))
   387:       return false;
   388:   return true;
   389: }
   390: 
   391: // Convert a cell to the integer value. The result must fit in uint64_t.
   392: uint64_t BT::MachineEvaluator::toInt(const RegisterCell &A) const {
   393:   assert(isInt(A));
   394:   uint64_t Val = 0;
   395:   uint16_t W = A.width();
   396:   for (uint16_t i = 0; i < W; ++i) {
   397:     Val <<= 1;
   398:     Val |= A[i].is(1);
   399:   }
   400:   return Val;
```
- EN: It declares or implements routines such as size, BT::RegisterCell::regify, BT::MachineEvaluator::getRegBitWidth, composeWithSubRegIndex, ... (21 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 size, BT::RegisterCell::regify, BT::MachineEvaluator::getRegBitWidth, composeWithSubRegIndex, ... (21 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 401-500 / 第 401-500 行

```cpp
   401: }
   402: 
   403: // Evaluator helper functions. These implement some common operation on
   404: // register cells that can be used to implement target-specific instructions
   405: // in a target-specific evaluator.
   406: 
   407: BT::RegisterCell BT::MachineEvaluator::eIMM(int64_t V, uint16_t W) const {
   408:   RegisterCell Res(W);
   409:   // For bits beyond the 63rd, this will generate the sign bit of V.
   410:   for (uint16_t i = 0; i < W; ++i) {
   411:     Res[i] = BitValue(V & 1);
   412:     V >>= 1;
   413:   }
   414:   return Res;
   415: }
   416: 
   417: BT::RegisterCell BT::MachineEvaluator::eIMM(const ConstantInt *CI) const {
   418:   const APInt &A = CI->getValue();
   419:   uint16_t BW = A.getBitWidth();
   420:   assert((unsigned)BW == A.getBitWidth() && "BitWidth overflow");
   421:   RegisterCell Res(BW);
   422:   for (uint16_t i = 0; i < BW; ++i)
   423:     Res[i] = A[i];
   424:   return Res;
   425: }
   426: 
   427: BT::RegisterCell BT::MachineEvaluator::eADD(const RegisterCell &A1,
   428:       const RegisterCell &A2) const {
   429:   uint16_t W = A1.width();
   430:   assert(W == A2.width());
   431:   RegisterCell Res(W);
   432:   bool Carry = false;
   433:   uint16_t I;
   434:   for (I = 0; I < W; ++I) {
   435:     const BitValue &V1 = A1[I];
   436:     const BitValue &V2 = A2[I];
   437:     if (!V1.num() || !V2.num())
   438:       break;
   439:     unsigned S = bool(V1) + bool(V2) + Carry;
   440:     Res[I] = BitValue(S & 1);
   441:     Carry = (S > 1);
   442:   }
   443:   for (; I < W; ++I) {
   444:     const BitValue &V1 = A1[I];
   445:     const BitValue &V2 = A2[I];
   446:     // If the next bit is same as Carry, the result will be 0 plus the
   447:     // other bit. The Carry bit will remain unchanged.
   448:     if (V1.is(Carry))
   449:       Res[I] = BitValue::ref(V2);
   450:     else if (V2.is(Carry))
   451:       Res[I] = BitValue::ref(V1);
   452:     else
   453:       break;
   454:   }
   455:   for (; I < W; ++I)
   456:     Res[I] = BitValue::self();
   457:   return Res;
   458: }
   459: 
   460: BT::RegisterCell BT::MachineEvaluator::eSUB(const RegisterCell &A1,
   461:       const RegisterCell &A2) const {
   462:   uint16_t W = A1.width();
   463:   assert(W == A2.width());
   464:   RegisterCell Res(W);
   465:   bool Borrow = false;
   466:   uint16_t I;
   467:   for (I = 0; I < W; ++I) {
   468:     const BitValue &V1 = A1[I];
   469:     const BitValue &V2 = A2[I];
   470:     if (!V1.num() || !V2.num())
   471:       break;
   472:     unsigned S = bool(V1) - bool(V2) - Borrow;
   473:     Res[I] = BitValue(S & 1);
   474:     Borrow = (S > 1);
   475:   }
   476:   for (; I < W; ++I) {
   477:     const BitValue &V1 = A1[I];
   478:     const BitValue &V2 = A2[I];
   479:     if (V1.is(Borrow)) {
   480:       Res[I] = BitValue::ref(V2);
   481:       break;
   482:     }
   483:     if (V2.is(Borrow))
   484:       Res[I] = BitValue::ref(V1);
   485:     else
   486:       break;
   487:   }
   488:   for (; I < W; ++I)
   489:     Res[I] = BitValue::self();
   490:   return Res;
   491: }
   492: 
   493: BT::RegisterCell BT::MachineEvaluator::eMLS(const RegisterCell &A1,
   494:       const RegisterCell &A2) const {
   495:   uint16_t W = A1.width() + A2.width();
   496:   uint16_t Z = A1.ct(false) + A2.ct(false);
   497:   RegisterCell Res(W);
   498:   Res.fill(0, Z, BitValue::Zero);
   499:   Res.fill(Z, W, BitValue::self());
   500:   return Res;
```
- EN: It declares or implements routines such as BT::MachineEvaluator::eIMM, Res, BitValue, getValue, ... (14 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 BT::MachineEvaluator::eIMM, Res, BitValue, getValue, ... (14 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 501-600 / 第 501-600 行

```cpp
   501: }
   502: 
   503: BT::RegisterCell BT::MachineEvaluator::eMLU(const RegisterCell &A1,
   504:       const RegisterCell &A2) const {
   505:   uint16_t W = A1.width() + A2.width();
   506:   uint16_t Z = A1.ct(false) + A2.ct(false);
   507:   RegisterCell Res(W);
   508:   Res.fill(0, Z, BitValue::Zero);
   509:   Res.fill(Z, W, BitValue::self());
   510:   return Res;
   511: }
   512: 
   513: BT::RegisterCell BT::MachineEvaluator::eASL(const RegisterCell &A1,
   514:       uint16_t Sh) const {
   515:   assert(Sh <= A1.width());
   516:   RegisterCell Res = RegisterCell::ref(A1);
   517:   Res.rol(Sh);
   518:   Res.fill(0, Sh, BitValue::Zero);
   519:   return Res;
   520: }
   521: 
   522: BT::RegisterCell BT::MachineEvaluator::eLSR(const RegisterCell &A1,
   523:       uint16_t Sh) const {
   524:   uint16_t W = A1.width();
   525:   assert(Sh <= W);
   526:   RegisterCell Res = RegisterCell::ref(A1);
   527:   Res.rol(W-Sh);
   528:   Res.fill(W-Sh, W, BitValue::Zero);
   529:   return Res;
   530: }
   531: 
   532: BT::RegisterCell BT::MachineEvaluator::eASR(const RegisterCell &A1,
   533:       uint16_t Sh) const {
   534:   uint16_t W = A1.width();
   535:   assert(Sh <= W);
   536:   RegisterCell Res = RegisterCell::ref(A1);
   537:   BitValue Sign = Res[W-1];
   538:   Res.rol(W-Sh);
   539:   Res.fill(W-Sh, W, Sign);
   540:   return Res;
   541: }
   542: 
   543: BT::RegisterCell BT::MachineEvaluator::eAND(const RegisterCell &A1,
   544:       const RegisterCell &A2) const {
   545:   uint16_t W = A1.width();
   546:   assert(W == A2.width());
   547:   RegisterCell Res(W);
   548:   for (uint16_t i = 0; i < W; ++i) {
   549:     const BitValue &V1 = A1[i];
   550:     const BitValue &V2 = A2[i];
   551:     if (V1.is(1))
   552:       Res[i] = BitValue::ref(V2);
   553:     else if (V2.is(1))
   554:       Res[i] = BitValue::ref(V1);
   555:     else if (V1.is(0) || V2.is(0))
   556:       Res[i] = BitValue::Zero;
   557:     else if (V1 == V2)
   558:       Res[i] = V1;
   559:     else
   560:       Res[i] = BitValue::self();
   561:   }
   562:   return Res;
   563: }
   564: 
   565: BT::RegisterCell BT::MachineEvaluator::eORL(const RegisterCell &A1,
   566:       const RegisterCell &A2) const {
   567:   uint16_t W = A1.width();
   568:   assert(W == A2.width());
   569:   RegisterCell Res(W);
   570:   for (uint16_t i = 0; i < W; ++i) {
   571:     const BitValue &V1 = A1[i];
   572:     const BitValue &V2 = A2[i];
   573:     if (V1.is(1) || V2.is(1))
   574:       Res[i] = BitValue::One;
   575:     else if (V1.is(0))
   576:       Res[i] = BitValue::ref(V2);
   577:     else if (V2.is(0))
   578:       Res[i] = BitValue::ref(V1);
   579:     else if (V1 == V2)
   580:       Res[i] = V1;
   581:     else
   582:       Res[i] = BitValue::self();
   583:   }
   584:   return Res;
   585: }
   586: 
   587: BT::RegisterCell BT::MachineEvaluator::eXOR(const RegisterCell &A1,
   588:       const RegisterCell &A2) const {
   589:   uint16_t W = A1.width();
   590:   assert(W == A2.width());
   591:   RegisterCell Res(W);
   592:   for (uint16_t i = 0; i < W; ++i) {
   593:     const BitValue &V1 = A1[i];
   594:     const BitValue &V2 = A2[i];
   595:     if (V1.is(0))
   596:       Res[i] = BitValue::ref(V2);
   597:     else if (V2.is(0))
   598:       Res[i] = BitValue::ref(V1);
   599:     else if (V1 == V2)
   600:       Res[i] = BitValue::Zero;
```
- EN: It declares or implements routines such as BT::MachineEvaluator::eMLU, width, ct, Res, ... (15 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 BT::MachineEvaluator::eMLU, width, ct, Res, ... (15 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 601-700 / 第 601-700 行

```cpp
   601:     else
   602:       Res[i] = BitValue::self();
   603:   }
   604:   return Res;
   605: }
   606: 
   607: BT::RegisterCell BT::MachineEvaluator::eNOT(const RegisterCell &A1) const {
   608:   uint16_t W = A1.width();
   609:   RegisterCell Res(W);
   610:   for (uint16_t i = 0; i < W; ++i) {
   611:     const BitValue &V = A1[i];
   612:     if (V.is(0))
   613:       Res[i] = BitValue::One;
   614:     else if (V.is(1))
   615:       Res[i] = BitValue::Zero;
   616:     else
   617:       Res[i] = BitValue::self();
   618:   }
   619:   return Res;
   620: }
   621: 
   622: BT::RegisterCell BT::MachineEvaluator::eSET(const RegisterCell &A1,
   623:       uint16_t BitN) const {
   624:   assert(BitN < A1.width());
   625:   RegisterCell Res = RegisterCell::ref(A1);
   626:   Res[BitN] = BitValue::One;
   627:   return Res;
   628: }
   629: 
   630: BT::RegisterCell BT::MachineEvaluator::eCLR(const RegisterCell &A1,
   631:       uint16_t BitN) const {
   632:   assert(BitN < A1.width());
   633:   RegisterCell Res = RegisterCell::ref(A1);
   634:   Res[BitN] = BitValue::Zero;
   635:   return Res;
   636: }
   637: 
   638: BT::RegisterCell BT::MachineEvaluator::eCLB(const RegisterCell &A1, bool B,
   639:       uint16_t W) const {
   640:   uint16_t C = A1.cl(B), AW = A1.width();
   641:   // If the last leading non-B bit is not a constant, then we don't know
   642:   // the real count.
   643:   if ((C < AW && A1[AW-1-C].num()) || C == AW)
   644:     return eIMM(C, W);
   645:   return RegisterCell::self(0, W);
   646: }
   647: 
   648: BT::RegisterCell BT::MachineEvaluator::eCTB(const RegisterCell &A1, bool B,
   649:       uint16_t W) const {
   650:   uint16_t C = A1.ct(B), AW = A1.width();
   651:   // If the last trailing non-B bit is not a constant, then we don't know
   652:   // the real count.
   653:   if ((C < AW && A1[C].num()) || C == AW)
   654:     return eIMM(C, W);
   655:   return RegisterCell::self(0, W);
   656: }
   657: 
   658: BT::RegisterCell BT::MachineEvaluator::eSXT(const RegisterCell &A1,
   659:       uint16_t FromN) const {
   660:   uint16_t W = A1.width();
   661:   assert(FromN <= W);
   662:   RegisterCell Res = RegisterCell::ref(A1);
   663:   BitValue Sign = Res[FromN-1];
   664:   // Sign-extend "inreg".
   665:   Res.fill(FromN, W, Sign);
   666:   return Res;
   667: }
   668: 
   669: BT::RegisterCell BT::MachineEvaluator::eZXT(const RegisterCell &A1,
   670:       uint16_t FromN) const {
   671:   uint16_t W = A1.width();
   672:   assert(FromN <= W);
   673:   RegisterCell Res = RegisterCell::ref(A1);
   674:   Res.fill(FromN, W, BitValue::Zero);
   675:   return Res;
   676: }
   677: 
   678: BT::RegisterCell BT::MachineEvaluator::eXTR(const RegisterCell &A1,
   679:       uint16_t B, uint16_t E) const {
   680:   uint16_t W = A1.width();
   681:   assert(B < W && E <= W);
   682:   if (B == E)
   683:     return RegisterCell(0);
   684:   uint16_t Last = (E > 0) ? E-1 : W-1;
   685:   RegisterCell Res = RegisterCell::ref(A1).extract(BT::BitMask(B, Last));
   686:   // Return shorter cell.
   687:   return Res;
   688: }
   689: 
   690: BT::RegisterCell BT::MachineEvaluator::eINS(const RegisterCell &A1,
   691:       const RegisterCell &A2, uint16_t AtN) const {
   692:   uint16_t W1 = A1.width(), W2 = A2.width();
   693:   (void)W1;
   694:   assert(AtN < W1 && AtN+W2 <= W1);
   695:   // Copy bits from A1, insert A2 at position AtN.
   696:   RegisterCell Res = RegisterCell::ref(A1);
   697:   if (W2 > 0)
   698:     Res.insert(RegisterCell::ref(A2), BT::BitMask(AtN, AtN+W2-1));
   699:   return Res;
   700: }
```
- EN: It declares or implements routines such as BitValue::self, BT::MachineEvaluator::eNOT, width, Res, ... (18 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 BitValue::self, BT::MachineEvaluator::eNOT, width, Res, ... (18 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 701-800 / 第 701-800 行

```cpp
   701: 
   702: BT::BitMask BT::MachineEvaluator::mask(Register Reg, unsigned Sub) const {
   703:   assert(Sub == 0 && "Generic BitTracker::mask called for Sub != 0");
   704:   uint16_t W = getRegBitWidth(Reg);
   705:   assert(W > 0 && "Cannot generate mask for empty register");
   706:   return BitMask(0, W-1);
   707: }
   708: 
   709: uint16_t BT::MachineEvaluator::getPhysRegBitWidth(MCRegister Reg) const {
   710:   const TargetRegisterClass &PC = *TRI.getMinimalPhysRegClass(Reg);
   711:   return TRI.getRegSizeInBits(PC);
   712: }
   713: 
   714: bool BT::MachineEvaluator::evaluate(const MachineInstr &MI,
   715:                                     const CellMapType &Inputs,
   716:                                     CellMapType &Outputs) const {
   717:   unsigned Opc = MI.getOpcode();
   718:   switch (Opc) {
   719:     case TargetOpcode::REG_SEQUENCE: {
   720:       RegisterRef RD = MI.getOperand(0);
   721:       assert(RD.Sub == 0);
   722:       RegisterRef RS = MI.getOperand(1);
   723:       unsigned SS = MI.getOperand(2).getImm();
   724:       RegisterRef RT = MI.getOperand(3);
   725:       unsigned ST = MI.getOperand(4).getImm();
   726:       assert(SS != ST);
   727: 
   728:       uint16_t W = getRegBitWidth(RD);
   729:       RegisterCell Res(W);
   730:       Res.insert(RegisterCell::ref(getCell(RS, Inputs)), mask(RD.Reg, SS));
   731:       Res.insert(RegisterCell::ref(getCell(RT, Inputs)), mask(RD.Reg, ST));
   732:       putCell(RD, Res, Outputs);
   733:       break;
   734:     }
   735: 
   736:     case TargetOpcode::COPY: {
   737:       // COPY can transfer a smaller register into a wider one.
   738:       // If that is the case, fill the remaining high bits with 0.
   739:       RegisterRef RD = MI.getOperand(0);
   740:       RegisterRef RS = MI.getOperand(1);
   741:       assert(RD.Sub == 0);
   742:       uint16_t WD = getRegBitWidth(RD);
   743:       uint16_t WS = getRegBitWidth(RS);
   744:       assert(WD >= WS);
   745:       RegisterCell Src = getCell(RS, Inputs);
   746:       RegisterCell Res(WD);
   747:       Res.insert(Src, BitMask(0, WS - 1));
   748:       Res.fill(WS, WD, BitValue::Zero);
   749:       putCell(RD, Res, Outputs);
   750:       break;
   751:     }
   752: 
   753:     default:
   754:       return false;
   755:   }
   756: 
   757:   return true;
   758: }
   759: 
   760: bool BT::UseQueueType::Cmp::operator()(const MachineInstr *InstA,
   761:                                        const MachineInstr *InstB) const {
   762:   // This is a comparison function for a priority queue: give higher priority
   763:   // to earlier instructions.
   764:   // This operator is used as "less", so returning "true" gives InstB higher
   765:   // priority (because then InstA < InstB).
   766:   if (InstA == InstB)
   767:     return false;
   768:   const MachineBasicBlock *BA = InstA->getParent();
   769:   const MachineBasicBlock *BB = InstB->getParent();
   770:   if (BA != BB) {
   771:     // If the blocks are different, ideally the dominating block would
   772:     // have a higher priority, but it may be too expensive to check.
   773:     return BA->getNumber() > BB->getNumber();
   774:   }
   775: 
   776:   auto getDist = [this] (const MachineInstr *MI) {
   777:     auto F = Dist.find(MI);
   778:     if (F != Dist.end())
   779:       return F->second;
   780:     MachineBasicBlock::const_iterator I = MI->getParent()->begin();
   781:     MachineBasicBlock::const_iterator E = MI->getIterator();
   782:     unsigned D = std::distance(I, E);
   783:     Dist.insert(std::make_pair(MI, D));
   784:     return D;
   785:   };
   786: 
   787:   return getDist(InstA) > getDist(InstB);
   788: }
   789: 
   790: // Main W-Z implementation.
   791: 
   792: void BT::visitPHI(const MachineInstr &PI) {
   793:   int ThisN = PI.getParent()->getNumber();
   794:   if (Trace)
   795:     dbgs() << "Visit FI(" << printMBBReference(*PI.getParent()) << "): " << PI;
   796: 
   797:   const MachineOperand &MD = PI.getOperand(0);
   798:   assert(MD.getSubReg() == 0 && "Unexpected sub-register in definition");
   799:   RegisterRef DefRR(MD);
   800:   uint16_t DefBW = ME.getRegBitWidth(DefRR);
```
- EN: It declares or implements routines such as BT::MachineEvaluator::mask, assert, getRegBitWidth, BitMask, ... (23 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 BT::MachineEvaluator::mask, assert, getRegBitWidth, BitMask, ... (23 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 801-900 / 第 801-900 行

```cpp
   801: 
   802:   RegisterCell DefC = ME.getCell(DefRR, Map);
   803:   if (DefC == RegisterCell::self(DefRR.Reg, DefBW))    // XXX slow
   804:     return;
   805: 
   806:   bool Changed = false;
   807: 
   808:   for (unsigned i = 1, n = PI.getNumOperands(); i < n; i += 2) {
   809:     const MachineBasicBlock *PB = PI.getOperand(i + 1).getMBB();
   810:     int PredN = PB->getNumber();
   811:     if (Trace)
   812:       dbgs() << "  edge " << printMBBReference(*PB) << "->"
   813:              << printMBBReference(*PI.getParent());
   814:     if (!EdgeExec.count(CFGEdge(PredN, ThisN))) {
   815:       if (Trace)
   816:         dbgs() << " not executable\n";
   817:       continue;
   818:     }
   819: 
   820:     RegisterRef RU = PI.getOperand(i);
   821:     RegisterCell ResC = ME.getCell(RU, Map);
   822:     if (Trace)
   823:       dbgs() << " input reg: " << printReg(RU.Reg, &ME.TRI, RU.Sub)
   824:              << " cell: " << ResC << "\n";
   825:     Changed |= DefC.meet(ResC, DefRR.Reg);
   826:   }
   827: 
   828:   if (Changed) {
   829:     if (Trace)
   830:       dbgs() << "Output: " << printReg(DefRR.Reg, &ME.TRI, DefRR.Sub)
   831:              << " cell: " << DefC << "\n";
   832:     ME.putCell(DefRR, DefC, Map);
   833:     visitUsesOf(DefRR.Reg);
   834:   }
   835: }
   836: 
   837: void BT::visitNonBranch(const MachineInstr &MI) {
   838:   if (Trace)
   839:     dbgs() << "Visit MI(" << printMBBReference(*MI.getParent()) << "): " << MI;
   840:   if (MI.isDebugInstr())
   841:     return;
   842:   assert(!MI.isBranch() && "Unexpected branch instruction");
   843: 
   844:   CellMapType ResMap;
   845:   bool Eval = ME.evaluate(MI, Map, ResMap);
   846: 
   847:   if (Trace && Eval) {
   848:     for (const MachineOperand &MO : MI.operands()) {
   849:       if (!MO.isReg() || !MO.isUse())
   850:         continue;
   851:       RegisterRef RU(MO);
   852:       dbgs() << "  input reg: " << printReg(RU.Reg, &ME.TRI, RU.Sub)
   853:              << " cell: " << ME.getCell(RU, Map) << "\n";
   854:     }
   855:     dbgs() << "Outputs:\n";
   856:     for (const std::pair<const unsigned, RegisterCell> &P : ResMap) {
   857:       RegisterRef RD(P.first);
   858:       dbgs() << "  " << printReg(P.first, &ME.TRI) << " cell: "
   859:              << ME.getCell(RD, ResMap) << "\n";
   860:     }
   861:   }
   862: 
   863:   // Iterate over all definitions of the instruction, and update the
   864:   // cells accordingly.
   865:   for (const MachineOperand &MO : MI.operands()) {
   866:     // Visit register defs only.
   867:     if (!MO.isReg() || !MO.isDef())
   868:       continue;
   869:     RegisterRef RD(MO);
   870:     assert(RD.Sub == 0 && "Unexpected sub-register in definition");
   871:     if (!RD.Reg.isVirtual())
   872:       continue;
   873: 
   874:     bool Changed = false;
   875:     if (!Eval || ResMap.count(RD.Reg) == 0) {
   876:       // Set to "ref" (aka "bottom").
   877:       uint16_t DefBW = ME.getRegBitWidth(RD);
   878:       RegisterCell RefC = RegisterCell::self(RD.Reg, DefBW);
   879:       if (RefC != ME.getCell(RD, Map)) {
   880:         ME.putCell(RD, RefC, Map);
   881:         Changed = true;
   882:       }
   883:     } else {
   884:       RegisterCell DefC = ME.getCell(RD, Map);
   885:       RegisterCell ResC = ME.getCell(RD, ResMap);
   886:       // This is a non-phi instruction, so the values of the inputs come
   887:       // from the same registers each time this instruction is evaluated.
   888:       // During the propagation, the values of the inputs can become lowered
   889:       // in the sense of the lattice operation, which may cause different
   890:       // results to be calculated in subsequent evaluations. This should
   891:       // not cause the bottoming of the result in the map, since the new
   892:       // result is already reflecting the lowered inputs.
   893:       for (uint16_t i = 0, w = DefC.width(); i < w; ++i) {
   894:         BitValue &V = DefC[i];
   895:         // Bits that are already "bottom" should not be updated.
   896:         if (V.Type == BitValue::Ref && V.RefI.Reg == RD.Reg)
   897:           continue;
   898:         // Same for those that are identical in DefC and ResC.
   899:         if (V == ResC[i])
   900:           continue;
```
- EN: It declares or implements routines such as getCell, getOperand, getNumber, meet, ... (13 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 getCell, getOperand, getNumber, meet, ... (13 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 901-1000 / 第 901-1000 行

```cpp
   901:         V = ResC[i];
   902:         Changed = true;
   903:       }
   904:       if (Changed)
   905:         ME.putCell(RD, DefC, Map);
   906:     }
   907:     if (Changed)
   908:       visitUsesOf(RD.Reg);
   909:   }
   910: }
   911: 
   912: void BT::visitBranchesFrom(const MachineInstr &BI) {
   913:   const MachineBasicBlock &B = *BI.getParent();
   914:   MachineBasicBlock::const_iterator It = BI, End = B.end();
   915:   BranchTargetList Targets, BTs;
   916:   bool FallsThrough = true, DefaultToAll = false;
   917:   int ThisN = B.getNumber();
   918: 
   919:   do {
   920:     BTs.clear();
   921:     const MachineInstr &MI = *It;
   922:     if (Trace)
   923:       dbgs() << "Visit BR(" << printMBBReference(B) << "): " << MI;
   924:     assert(MI.isBranch() && "Expecting branch instruction");
   925:     InstrExec.insert(&MI);
   926:     bool Eval = ME.evaluate(MI, Map, BTs, FallsThrough);
   927:     if (!Eval) {
   928:       // If the evaluation failed, we will add all targets. Keep going in
   929:       // the loop to mark all executable branches as such.
   930:       DefaultToAll = true;
   931:       FallsThrough = true;
   932:       if (Trace)
   933:         dbgs() << "  failed to evaluate: will add all CFG successors\n";
   934:     } else if (!DefaultToAll) {
   935:       // If evaluated successfully add the targets to the cumulative list.
   936:       if (Trace) {
   937:         dbgs() << "  adding targets:";
   938:         for (const MachineBasicBlock *BT : BTs)
   939:           dbgs() << " " << printMBBReference(*BT);
   940:         if (FallsThrough)
   941:           dbgs() << "\n  falls through\n";
   942:         else
   943:           dbgs() << "\n  does not fall through\n";
   944:       }
   945:       Targets.insert_range(BTs);
   946:     }
   947:     ++It;
   948:   } while (FallsThrough && It != End);
   949: 
   950:   if (B.mayHaveInlineAsmBr())
   951:     DefaultToAll = true;
   952: 
   953:   if (!DefaultToAll) {
   954:     // Need to add all CFG successors that lead to EH landing pads.
   955:     // There won't be explicit branches to these blocks, but they must
   956:     // be processed.
   957:     for (const MachineBasicBlock *SB : B.successors()) {
   958:       if (SB->isEHPad())
   959:         Targets.insert(SB);
   960:     }
   961:     if (FallsThrough) {
   962:       MachineFunction::const_iterator BIt = B.getIterator();
   963:       MachineFunction::const_iterator Next = std::next(BIt);
   964:       if (Next != MF.end())
   965:         Targets.insert(&*Next);
   966:     }
   967:   } else {
   968:     Targets.insert_range(B.successors());
   969:   }
   970: 
   971:   for (const MachineBasicBlock *TB : Targets)
   972:     FlowQ.push(CFGEdge(ThisN, TB->getNumber()));
   973: }
   974: 
   975: void BT::visitUsesOf(Register Reg) {
   976:   if (Trace)
   977:     dbgs() << "queuing uses of modified reg " << printReg(Reg, &ME.TRI)
   978:            << " cell: " << ME.getCell(Reg, Map) << '\n';
   979: 
   980:   for (MachineInstr &UseI : MRI.use_nodbg_instructions(Reg))
   981:     UseQ.push(&UseI);
   982: }
   983: 
   984: BT::RegisterCell BT::get(RegisterRef RR) const {
   985:   return ME.getCell(RR, Map);
   986: }
   987: 
   988: void BT::put(RegisterRef RR, const RegisterCell &RC) {
   989:   ME.putCell(RR, RC, Map);
   990: }
   991: 
   992: // Replace all references to bits from OldRR with the corresponding bits
   993: // in NewRR.
   994: void BT::subst(RegisterRef OldRR, RegisterRef NewRR) {
   995:   assert(Map.count(OldRR.Reg) > 0 && "OldRR not present in map");
   996:   BitMask OM = ME.mask(OldRR.Reg, OldRR.Sub);
   997:   BitMask NM = ME.mask(NewRR.Reg, NewRR.Sub);
   998:   uint16_t OMB = OM.first(), OME = OM.last();
   999:   uint16_t NMB = NM.first(), NME = NM.last();
  1000:   (void)NME;
```
- EN: It declares or implements routines such as BT::visitBranchesFrom, getParent, end, getNumber, ... (19 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 BT::visitBranchesFrom, getParent, end, getNumber, ... (19 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 1001-1100 / 第 1001-1100 行

```cpp
  1001:   assert((OME-OMB == NME-NMB) &&
  1002:          "Substituting registers of different lengths");
  1003:   for (std::pair<const unsigned, RegisterCell> &P : Map) {
  1004:     RegisterCell &RC = P.second;
  1005:     for (uint16_t i = 0, w = RC.width(); i < w; ++i) {
  1006:       BitValue &V = RC[i];
  1007:       if (V.Type != BitValue::Ref || V.RefI.Reg != OldRR.Reg)
  1008:         continue;
  1009:       if (V.RefI.Pos < OMB || V.RefI.Pos > OME)
  1010:         continue;
  1011:       V.RefI.Reg = NewRR.Reg;
  1012:       V.RefI.Pos += NMB-OMB;
  1013:     }
  1014:   }
  1015: }
  1016: 
  1017: // Check if the block has been "executed" during propagation. (If not, the
  1018: // block is dead, but it may still appear to be reachable.)
  1019: bool BT::reached(const MachineBasicBlock *B) const {
  1020:   int BN = B->getNumber();
  1021:   assert(BN >= 0);
  1022:   return ReachedBB.count(BN);
  1023: }
  1024: 
  1025: // Visit an individual instruction. This could be a newly added instruction,
  1026: // or one that has been modified by an optimization.
  1027: void BT::visit(const MachineInstr &MI) {
  1028:   assert(!MI.isBranch() && "Only non-branches are allowed");
  1029:   InstrExec.insert(&MI);
  1030:   visitNonBranch(MI);
  1031:   // Make sure to flush all the pending use updates.
  1032:   runUseQueue();
  1033:   // The call to visitNonBranch could propagate the changes until a branch
  1034:   // is actually visited. This could result in adding CFG edges to the flow
  1035:   // queue. Since the queue won't be processed, clear it.
  1036:   while (!FlowQ.empty())
  1037:     FlowQ.pop();
  1038: }
  1039: 
  1040: void BT::reset() {
  1041:   EdgeExec.clear();
  1042:   InstrExec.clear();
  1043:   Map.clear();
  1044:   ReachedBB.clear();
  1045:   ReachedBB.reserve(MF.size());
  1046: }
  1047: 
  1048: void BT::runEdgeQueue(BitVector &BlockScanned) {
  1049:   while (!FlowQ.empty()) {
  1050:     CFGEdge Edge = FlowQ.front();
  1051:     FlowQ.pop();
  1052: 
  1053:     if (!EdgeExec.insert(Edge).second)
  1054:       return;
  1055:     ReachedBB.insert(Edge.second);
  1056: 
  1057:     const MachineBasicBlock &B = *MF.getBlockNumbered(Edge.second);
  1058:     MachineBasicBlock::const_iterator It = B.begin(), End = B.end();
  1059:     // Visit PHI nodes first.
  1060:     while (It != End && It->isPHI()) {
  1061:       const MachineInstr &PI = *It++;
  1062:       InstrExec.insert(&PI);
  1063:       visitPHI(PI);
  1064:     }
  1065: 
  1066:     // If this block has already been visited through a flow graph edge,
  1067:     // then the instructions have already been processed. Any updates to
  1068:     // the cells would now only happen through visitUsesOf...
  1069:     if (BlockScanned[Edge.second])
  1070:       return;
  1071:     BlockScanned[Edge.second] = true;
  1072: 
  1073:     // Visit non-branch instructions.
  1074:     while (It != End && !It->isBranch()) {
  1075:       const MachineInstr &MI = *It++;
  1076:       InstrExec.insert(&MI);
  1077:       visitNonBranch(MI);
  1078:     }
  1079:     // If block end has been reached, add the fall-through edge to the queue.
  1080:     if (It == End) {
  1081:       MachineFunction::const_iterator BIt = B.getIterator();
  1082:       MachineFunction::const_iterator Next = std::next(BIt);
  1083:       if (Next != MF.end() && B.isSuccessor(&*Next)) {
  1084:         int ThisN = B.getNumber();
  1085:         int NextN = Next->getNumber();
  1086:         FlowQ.push(CFGEdge(ThisN, NextN));
  1087:       }
  1088:     } else {
  1089:       // Handle the remaining sequence of branches. This function will update
  1090:       // the work queue.
  1091:       visitBranchesFrom(*It);
  1092:     }
  1093:   } // while (!FlowQ->empty())
  1094: }
  1095: 
  1096: void BT::runUseQueue() {
  1097:   while (!UseQ.empty()) {
  1098:     MachineInstr &UseI = *UseQ.front();
  1099:     UseQ.pop();
  1100: 
```
- EN: It declares or implements routines such as assert, BT::reached, getNumber, count, ... (22 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 assert, BT::reached, getNumber, count, ... (22 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 1101-1142 / 第 1101-1142 行

```cpp
  1101:     if (!InstrExec.count(&UseI))
  1102:       continue;
  1103:     if (UseI.isPHI())
  1104:       visitPHI(UseI);
  1105:     else if (!UseI.isBranch())
  1106:       visitNonBranch(UseI);
  1107:     else
  1108:       visitBranchesFrom(UseI);
  1109:   }
  1110: }
  1111: 
  1112: void BT::run() {
  1113:   reset();
  1114:   assert(FlowQ.empty());
  1115: 
  1116:   using MachineFlowGraphTraits = GraphTraits<const MachineFunction*>;
  1117:   const MachineBasicBlock *Entry = MachineFlowGraphTraits::getEntryNode(&MF);
  1118: 
  1119:   unsigned MaxBN = 0;
  1120:   for (const MachineBasicBlock &B : MF) {
  1121:     assert(B.getNumber() >= 0 && "Disconnected block");
  1122:     unsigned BN = B.getNumber();
  1123:     if (BN > MaxBN)
  1124:       MaxBN = BN;
  1125:   }
  1126: 
  1127:   // Keep track of visited blocks.
  1128:   BitVector BlockScanned(MaxBN+1);
  1129: 
  1130:   int EntryN = Entry->getNumber();
  1131:   // Generate a fake edge to get something to start with.
  1132:   FlowQ.push(CFGEdge(-1, EntryN));
  1133: 
  1134:   while (!FlowQ.empty() || !UseQ.empty()) {
  1135:     runEdgeQueue(BlockScanned);
  1136:     runUseQueue();
  1137:   }
  1138:   UseQ.reset();
  1139: 
  1140:   if (Trace)
  1141:     print_cells(dbgs() << "Cells after propagation:\n");
  1142: }
```
- EN: It declares or implements routines such as visitBranchesFrom, BT::run, reset, assert, ... (10 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 visitBranchesFrom, BT::run, reset, assert, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- instruction semantics / 指令语义

## Dependencies / 依赖关系

- Direct includes / 直接包含: `BitTracker.h, llvm/ADT/APInt.h, llvm/ADT/BitVector.h, llvm/CodeGen/MachineBasicBlock.h, llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineInstr.h, llvm/CodeGen/MachineOperand.h, llvm/CodeGen/MachineRegisterInfo.h, llvm/IR/Constants.h, llvm/Support/Debug.h`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
