# HexagonConstPropagation.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonConstPropagation.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Lattice cell, based on that was described in the W-Z paper on constant
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-200 / 第 1-200 行

```cpp
     1: //===- HexagonConstPropagation.cpp ----------------------------------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: 
     9: #include "HexagonInstrInfo.h"
    10: #include "HexagonRegisterInfo.h"
    11: #include "HexagonSubtarget.h"
    12: #include "llvm/ADT/APFloat.h"
    13: #include "llvm/ADT/APInt.h"
    14: #include "llvm/ADT/PostOrderIterator.h"
    15: #include "llvm/ADT/SetVector.h"
    16: #include "llvm/ADT/SmallVector.h"
    17: #include "llvm/ADT/StringRef.h"
    18: #include "llvm/CodeGen/MachineBasicBlock.h"
    19: #include "llvm/CodeGen/MachineFunction.h"
    20: #include "llvm/CodeGen/MachineFunctionPass.h"
    21: #include "llvm/CodeGen/MachineInstr.h"
    22: #include "llvm/CodeGen/MachineInstrBuilder.h"
    23: #include "llvm/CodeGen/MachineOperand.h"
    24: #include "llvm/CodeGen/MachineRegisterInfo.h"
    25: #include "llvm/CodeGen/TargetRegisterInfo.h"
    26: #include "llvm/CodeGen/TargetSubtargetInfo.h"
    27: #include "llvm/IR/Constants.h"
    28: #include "llvm/IR/Type.h"
    29: #include "llvm/Pass.h"
    30: #include "llvm/Support/Casting.h"
    31: #include "llvm/Support/Compiler.h"
    32: #include "llvm/Support/Debug.h"
    33: #include "llvm/Support/ErrorHandling.h"
    34: #include "llvm/Support/MathExtras.h"
    35: #include "llvm/Support/raw_ostream.h"
    36: #include <cassert>
    37: #include <cstdint>
    38: #include <cstring>
    39: #include <iterator>
    40: #include <map>
    41: #include <queue>
    42: #include <set>
    43: #include <utility>
    44: #include <vector>
    45: 
    46: #define DEBUG_TYPE "hcp"
    47: 
    48: using namespace llvm;
    49: 
    50: namespace {
    51: 
    52:   // Properties of a value that are tracked by the propagation.
    53:   // A property that is marked as present (i.e. bit is set) dentes that the
    54:   // value is known (proven) to have this property. Not all combinations
    55:   // of bits make sense, for example Zero and NonZero are mutually exclusive,
    56:   // but on the other hand, Zero implies Finite. In this case, whenever
    57:   // the Zero property is present, Finite should also be present.
    58:   class ConstantProperties {
    59:   public:
    60:     enum {
    61:       Unknown   = 0x0000,
    62:       Zero      = 0x0001,
    63:       NonZero   = 0x0002,
    64:       Finite    = 0x0004,
    65:       Infinity  = 0x0008,
    66:       NaN       = 0x0010,
    67:       SignedZero = 0x0020,
    68:       NumericProperties = (Zero|NonZero|Finite|Infinity|NaN|SignedZero),
    69:       PosOrZero       = 0x0100,
    70:       NegOrZero       = 0x0200,
    71:       SignProperties  = (PosOrZero|NegOrZero),
    72:       Everything      = (NumericProperties|SignProperties)
    73:     };
    74: 
    75:     // For a given constant, deduce the set of trackable properties that this
    76:     // constant has.
    77:     static uint32_t deduce(const Constant *C);
    78:   };
    79: 
    80:   using RegSubRegPair = TargetInstrInfo::RegSubRegPair;
    81: 
    82:   // Lattice cell, based on that was described in the W-Z paper on constant
    83:   // propagation.
    84:   // Lattice cell will be allowed to hold multiple constant values. While
    85:   // multiple values would normally indicate "bottom", we can still derive
    86:   // some useful information from them. For example, comparison X > 0
    87:   // could be folded if all the values in the cell associated with X are
    88:   // positive.
    89:   class LatticeCell {
    90:   private:
    91:     enum { Normal, Top, Bottom };
    92: 
    93:     static const unsigned MaxCellSize = 4;
    94: 
    95:     unsigned Kind:2;
    96:     unsigned Size:3;
    97:     unsigned IsSpecial:1;
    98:     unsigned :0;
    99: 
   100:   public:
   101:     union {
   102:       uint32_t Properties;
   103:       const Constant *Value;
   104:       const Constant *Values[MaxCellSize];
   105:     };
   106: 
   107:     LatticeCell() : Kind(Top), Size(0), IsSpecial(false) {
   108:       llvm::fill(Values, nullptr);
   109:     }
   110: 
   111:     bool meet(const LatticeCell &L);
   112:     bool add(const Constant *C);
   113:     bool add(uint32_t Property);
   114:     uint32_t properties() const;
   115:     unsigned size() const { return Size; }
   116: 
   117:     LatticeCell(const LatticeCell &L) {
   118:       // This memcpy also copies Properties (when L.Size == 0).
   119:       uint32_t N =
   120:           L.IsSpecial ? sizeof L.Properties : L.Size * sizeof(const Constant *);
   121:       memcpy(Values, L.Values, N);
   122:       Kind = L.Kind;
   123:       Size = L.Size;
   124:       IsSpecial = L.IsSpecial;
   125:     }
   126: 
   127:     LatticeCell &operator=(const LatticeCell &L) {
   128:       if (this != &L) {
   129:         // This memcpy also copies Properties (when L.Size == 0).
   130:         uint32_t N = L.IsSpecial ? sizeof L.Properties
   131:                                  : L.Size * sizeof(const Constant *);
   132:         memcpy(Values, L.Values, N);
   133:         Kind = L.Kind;
   134:         Size = L.Size;
   135:         IsSpecial = L.IsSpecial;
   136:       }
   137:       return *this;
   138:     }
   139: 
   140:     bool isSingle() const { return size() == 1; }
   141:     bool isProperty() const { return IsSpecial; }
   142:     bool isTop() const { return Kind == Top; }
   143:     bool isBottom() const { return Kind == Bottom; }
   144: 
   145:     bool setBottom() {
   146:       bool Changed = (Kind != Bottom);
   147:       Kind = Bottom;
   148:       Size = 0;
   149:       IsSpecial = false;
   150:       return Changed;
   151:     }
   152: 
   153:     void print(raw_ostream &os) const;
   154: 
   155:   private:
   156:     void setProperty() {
   157:       IsSpecial = true;
   158:       Size = 0;
   159:       Kind = Normal;
   160:     }
   161: 
   162:     bool convertToProperty();
   163:   };
   164: 
   165: #ifndef NDEBUG
   166:   raw_ostream &operator<< (raw_ostream &os, const LatticeCell &L) {
   167:     L.print(os);
   168:     return os;
   169:   }
   170: #endif
   171: 
   172:   class MachineConstEvaluator;
   173: 
   174:   class MachineConstPropagator {
   175:   public:
   176:     MachineConstPropagator(MachineConstEvaluator &E) : MCE(E) {
   177:       Bottom.setBottom();
   178:     }
   179: 
   180:     // Mapping: vreg -> cell
   181:     // The keys are registers _without_ subregisters. This won't allow
   182:     // definitions in the form of "vreg:subreg = ...". Such definitions
   183:     // would be questionable from the point of view of SSA, since the "vreg"
   184:     // could not be initialized in its entirety (specifically, an instruction
   185:     // defining the "other part" of "vreg" would also count as a definition
   186:     // of "vreg", which would violate the SSA).
   187:     // If a value of a pair vreg:subreg needs to be obtained, the cell for
   188:     // "vreg" needs to be looked up, and then the value of subregister "subreg"
   189:     // needs to be evaluated.
   190:     class CellMap {
   191:     public:
   192:       CellMap() {
   193:         assert(Top.isTop());
   194:         Bottom.setBottom();
   195:       }
   196: 
   197:       void clear() { Map.clear(); }
   198: 
   199:       bool has(Register R) const {
   200:         // All non-virtual registers are considered "bottom".
```
- EN: It imports headers such as HexagonInstrInfo.h, HexagonRegisterInfo.h, HexagonSubtarget.h, llvm/ADT/APFloat.h, ... (36 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as ConstantProperties, LatticeCell, MachineConstEvaluator, MachineConstPropagator, ... (5 total), which carry the state or API of this component.
- CN: 这里引入了 HexagonInstrInfo.h, HexagonRegisterInfo.h, HexagonSubtarget.h, llvm/ADT/APFloat.h, ... (36 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 ConstantProperties, LatticeCell, MachineConstEvaluator, MachineConstPropagator, ... (5 total) 等类型，用来承载该组件的状态或接口。

### Lines 201-400 / 第 201-400 行

```cpp
   201:         if (!R.isVirtual())
   202:           return true;
   203:         MapType::const_iterator F = Map.find(R);
   204:         return F != Map.end();
   205:       }
   206: 
   207:       const LatticeCell &get(Register R) const {
   208:         if (!R.isVirtual())
   209:           return Bottom;
   210:         MapType::const_iterator F = Map.find(R);
   211:         if (F != Map.end())
   212:           return F->second;
   213:         return Top;
   214:       }
   215: 
   216:       // Invalidates any const references.
   217:       void update(Register R, const LatticeCell &L) { Map[R] = L; }
   218: 
   219:       void print(raw_ostream &os, const TargetRegisterInfo &TRI) const;
   220: 
   221:     private:
   222:       using MapType = std::map<Register, LatticeCell>;
   223: 
   224:       MapType Map;
   225:       // To avoid creating "top" entries, return a const reference to
   226:       // this cell in "get". Also, have a "Bottom" cell to return from
   227:       // get when a value of a physical register is requested.
   228:       LatticeCell Top, Bottom;
   229: 
   230:     public:
   231:       using const_iterator = MapType::const_iterator;
   232: 
   233:       const_iterator begin() const { return Map.begin(); }
   234:       const_iterator end() const { return Map.end(); }
   235:     };
   236: 
   237:     bool run(MachineFunction &MF);
   238: 
   239:   private:
   240:     void visitPHI(const MachineInstr &PN);
   241:     void visitNonBranch(const MachineInstr &MI);
   242:     void visitBranchesFrom(const MachineInstr &BrI);
   243:     void visitUsesOf(unsigned R);
   244:     bool computeBlockSuccessors(const MachineBasicBlock *MB,
   245:           SetVector<const MachineBasicBlock*> &Targets);
   246:     void removeCFGEdge(MachineBasicBlock *From, MachineBasicBlock *To);
   247: 
   248:     void propagate(MachineFunction &MF);
   249:     bool rewrite(MachineFunction &MF);
   250: 
   251:     MachineRegisterInfo      *MRI = nullptr;
   252:     MachineConstEvaluator    &MCE;
   253: 
   254:     using CFGEdge = std::pair<unsigned, unsigned>;
   255:     using SetOfCFGEdge = std::set<CFGEdge>;
   256:     using SetOfInstr = std::set<const MachineInstr *>;
   257:     using QueueOfCFGEdge = std::queue<CFGEdge>;
   258: 
   259:     LatticeCell     Bottom;
   260:     CellMap         Cells;
   261:     SetOfCFGEdge    EdgeExec;
   262:     SetOfInstr      InstrExec;
   263:     QueueOfCFGEdge  FlowQ;
   264:   };
   265: 
   266:   // The "evaluator/rewriter" of machine instructions. This is an abstract
   267:   // base class that provides the interface that the propagator will use,
   268:   // as well as some helper functions that are target-independent.
   269:   class MachineConstEvaluator {
   270:   public:
   271:     MachineConstEvaluator(MachineFunction &Fn)
   272:       : TRI(*Fn.getSubtarget().getRegisterInfo()),
   273:         MF(Fn), CX(Fn.getFunction().getContext()) {}
   274:     virtual ~MachineConstEvaluator() = default;
   275: 
   276:     // The required interface:
   277:     // - A set of three "evaluate" functions. Each returns "true" if the
   278:     //       computation succeeded, "false" otherwise.
   279:     //   (1) Given an instruction MI, and the map with input values "Inputs",
   280:     //       compute the set of output values "Outputs". An example of when
   281:     //       the computation can "fail" is if MI is not an instruction that
   282:     //       is recognized by the evaluator.
   283:     //   (2) Given a register R (as reg:subreg), compute the cell that
   284:     //       corresponds to the "subreg" part of the given register.
   285:     //   (3) Given a branch instruction BrI, compute the set of target blocks.
   286:     //       If the branch can fall-through, add null (0) to the list of
   287:     //       possible targets.
   288:     // - A function "rewrite", that given the cell map after propagation,
   289:     //   could rewrite instruction MI in a more beneficial form. Return
   290:     //   "true" if a change has been made, "false" otherwise.
   291:     using CellMap = MachineConstPropagator::CellMap;
   292:     virtual bool evaluate(const MachineInstr &MI, const CellMap &Inputs,
   293:                           CellMap &Outputs) = 0;
   294:     virtual bool evaluate(const RegSubRegPair &R, const LatticeCell &SrcC,
   295:                           LatticeCell &Result) = 0;
   296:     virtual bool evaluate(const MachineInstr &BrI, const CellMap &Inputs,
   297:                           SetVector<const MachineBasicBlock*> &Targets,
   298:                           bool &CanFallThru) = 0;
   299:     virtual bool rewrite(MachineInstr &MI, const CellMap &Inputs) = 0;
   300: 
   301:     const TargetRegisterInfo &TRI;
   302: 
   303:   protected:
   304:     MachineFunction &MF;
   305:     LLVMContext     &CX;
   306: 
   307:     struct Comparison {
   308:       enum {
   309:         Unk = 0x00,
   310:         EQ  = 0x01,
   311:         NE  = 0x02,
   312:         L   = 0x04, // Less-than property.
   313:         G   = 0x08, // Greater-than property.
   314:         U   = 0x40, // Unsigned property.
   315:         LTs = L,
   316:         LEs = L | EQ,
   317:         GTs = G,
   318:         GEs = G | EQ,
   319:         LTu = L      | U,
   320:         LEu = L | EQ | U,
   321:         GTu = G      | U,
   322:         GEu = G | EQ | U
   323:       };
   324: 
   325:       static uint32_t negate(uint32_t Cmp) {
   326:         if (Cmp == EQ)
   327:           return NE;
   328:         if (Cmp == NE)
   329:           return EQ;
   330:         assert((Cmp & (L|G)) != (L|G));
   331:         return Cmp ^ (L|G);
   332:       }
   333:     };
   334: 
   335:     // Helper functions.
   336: 
   337:     bool getCell(const RegSubRegPair &R, const CellMap &Inputs,
   338:                  LatticeCell &RC);
   339:     bool constToInt(const Constant *C, APInt &Val) const;
   340:     const ConstantInt *intToConst(const APInt &Val) const;
   341: 
   342:     // Compares.
   343:     bool evaluateCMPrr(uint32_t Cmp, const RegSubRegPair &R1,
   344:                        const RegSubRegPair &R2, const CellMap &Inputs,
   345:                        bool &Result);
   346:     bool evaluateCMPri(uint32_t Cmp, const RegSubRegPair &R1, const APInt &A2,
   347:                        const CellMap &Inputs, bool &Result);
   348:     bool evaluateCMPrp(uint32_t Cmp, const RegSubRegPair &R1, uint64_t Props2,
   349:                        const CellMap &Inputs, bool &Result);
   350:     bool evaluateCMPii(uint32_t Cmp, const APInt &A1, const APInt &A2,
   351:           bool &Result);
   352:     bool evaluateCMPpi(uint32_t Cmp, uint32_t Props, const APInt &A2,
   353:           bool &Result);
   354:     bool evaluateCMPpp(uint32_t Cmp, uint32_t Props1, uint32_t Props2,
   355:           bool &Result);
   356: 
   357:     bool evaluateCOPY(const RegSubRegPair &R1, const CellMap &Inputs,
   358:                       LatticeCell &Result);
   359: 
   360:     // Logical operations.
   361:     bool evaluateANDrr(const RegSubRegPair &R1, const RegSubRegPair &R2,
   362:                        const CellMap &Inputs, LatticeCell &Result);
   363:     bool evaluateANDri(const RegSubRegPair &R1, const APInt &A2,
   364:                        const CellMap &Inputs, LatticeCell &Result);
   365:     bool evaluateANDii(const APInt &A1, const APInt &A2, APInt &Result);
   366:     bool evaluateORrr(const RegSubRegPair &R1, const RegSubRegPair &R2,
   367:                       const CellMap &Inputs, LatticeCell &Result);
   368:     bool evaluateORri(const RegSubRegPair &R1, const APInt &A2,
   369:                       const CellMap &Inputs, LatticeCell &Result);
   370:     bool evaluateORii(const APInt &A1, const APInt &A2, APInt &Result);
   371:     bool evaluateXORrr(const RegSubRegPair &R1, const RegSubRegPair &R2,
   372:                        const CellMap &Inputs, LatticeCell &Result);
   373:     bool evaluateXORri(const RegSubRegPair &R1, const APInt &A2,
   374:                        const CellMap &Inputs, LatticeCell &Result);
   375:     bool evaluateXORii(const APInt &A1, const APInt &A2, APInt &Result);
   376: 
   377:     // Extensions.
   378:     bool evaluateZEXTr(const RegSubRegPair &R1, unsigned Width, unsigned Bits,
   379:                        const CellMap &Inputs, LatticeCell &Result);
   380:     bool evaluateZEXTi(const APInt &A1, unsigned Width, unsigned Bits,
   381:           APInt &Result);
   382:     bool evaluateSEXTr(const RegSubRegPair &R1, unsigned Width, unsigned Bits,
   383:                        const CellMap &Inputs, LatticeCell &Result);
   384:     bool evaluateSEXTi(const APInt &A1, unsigned Width, unsigned Bits,
   385:           APInt &Result);
   386: 
   387:     // Leading/trailing bits.
   388:     bool evaluateCLBr(const RegSubRegPair &R1, bool Zeros, bool Ones,
   389:                       const CellMap &Inputs, LatticeCell &Result);
   390:     bool evaluateCLBi(const APInt &A1, bool Zeros, bool Ones, APInt &Result);
   391:     bool evaluateCTBr(const RegSubRegPair &R1, bool Zeros, bool Ones,
   392:                       const CellMap &Inputs, LatticeCell &Result);
   393:     bool evaluateCTBi(const APInt &A1, bool Zeros, bool Ones, APInt &Result);
   394: 
   395:     // Bitfield extract.
   396:     bool evaluateEXTRACTr(const RegSubRegPair &R1, unsigned Width,
   397:                           unsigned Bits, unsigned Offset, bool Signed,
   398:                           const CellMap &Inputs, LatticeCell &Result);
   399:     bool evaluateEXTRACTi(const APInt &A1, unsigned Bits, unsigned Offset,
   400:           bool Signed, APInt &Result);
```
- EN: It declares types such as MachineConstEvaluator, Comparison, which carry the state or API of this component. It defines declarative TableGen records like MachineConstEvaluator; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as find, end, get, update, ... (47 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明了 MachineConstEvaluator, Comparison 等类型，用来承载该组件的状态或接口。 这里定义了 MachineConstEvaluator 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 find, end, get, update, ... (47 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 401-600 / 第 401-600 行

```cpp
   401:     // Vector operations.
   402:     bool evaluateSplatr(const RegSubRegPair &R1, unsigned Bits, unsigned Count,
   403:                         const CellMap &Inputs, LatticeCell &Result);
   404:     bool evaluateSplati(const APInt &A1, unsigned Bits, unsigned Count,
   405:           APInt &Result);
   406:   };
   407: 
   408: } // end anonymous namespace
   409: 
   410: uint32_t ConstantProperties::deduce(const Constant *C) {
   411:   if (isa<ConstantInt>(C)) {
   412:     const ConstantInt *CI = cast<ConstantInt>(C);
   413:     if (CI->isZero())
   414:       return Zero | PosOrZero | NegOrZero | Finite;
   415:     uint32_t Props = (NonZero | Finite);
   416:     if (CI->isNegative())
   417:       return Props | NegOrZero;
   418:     return Props | PosOrZero;
   419:   }
   420: 
   421:   if (isa<ConstantFP>(C)) {
   422:     const ConstantFP *CF = cast<ConstantFP>(C);
   423:     uint32_t Props = CF->isNegative() ? (NegOrZero|NonZero)
   424:                                       : PosOrZero;
   425:     if (CF->isZero())
   426:       return (Props & ~NumericProperties) | (Zero|Finite);
   427:     Props = (Props & ~NumericProperties) | NonZero;
   428:     if (CF->isNaN())
   429:       return (Props & ~NumericProperties) | NaN;
   430:     const APFloat &Val = CF->getValueAPF();
   431:     if (Val.isInfinity())
   432:       return (Props & ~NumericProperties) | Infinity;
   433:     Props |= Finite;
   434:     return Props;
   435:   }
   436: 
   437:   return Unknown;
   438: }
   439: 
   440: // Convert a cell from a set of specific values to a cell that tracks
   441: // properties.
   442: bool LatticeCell::convertToProperty() {
   443:   if (isProperty())
   444:     return false;
   445:   // Corner case: converting a fresh (top) cell to "special".
   446:   // This can happen, when adding a property to a top cell.
   447:   uint32_t Everything = ConstantProperties::Everything;
   448:   uint32_t Ps = !isTop() ? properties()
   449:                          : Everything;
   450:   if (Ps != ConstantProperties::Unknown) {
   451:     Properties = Ps;
   452:     setProperty();
   453:   } else {
   454:     setBottom();
   455:   }
   456:   return true;
   457: }
   458: 
   459: #ifndef NDEBUG
   460: void LatticeCell::print(raw_ostream &os) const {
   461:   if (isProperty()) {
   462:     os << "{ ";
   463:     uint32_t Ps = properties();
   464:     if (Ps & ConstantProperties::Zero)
   465:       os << "zero ";
   466:     if (Ps & ConstantProperties::NonZero)
   467:       os << "nonzero ";
   468:     if (Ps & ConstantProperties::Finite)
   469:       os << "finite ";
   470:     if (Ps & ConstantProperties::Infinity)
   471:       os << "infinity ";
   472:     if (Ps & ConstantProperties::NaN)
   473:       os << "nan ";
   474:     if (Ps & ConstantProperties::PosOrZero)
   475:       os << "poz ";
   476:     if (Ps & ConstantProperties::NegOrZero)
   477:       os << "nez ";
   478:     os << '}';
   479:     return;
   480:   }
   481: 
   482:   os << "{ ";
   483:   if (isBottom()) {
   484:     os << "bottom";
   485:   } else if (isTop()) {
   486:     os << "top";
   487:   } else {
   488:     for (unsigned i = 0; i < size(); ++i) {
   489:       const Constant *C = Values[i];
   490:       if (i != 0)
   491:         os << ", ";
   492:       C->print(os);
   493:     }
   494:   }
   495:   os << " }";
   496: }
   497: #endif
   498: 
   499: // "Meet" operation on two cells. This is the key of the propagation
   500: // algorithm.
   501: bool LatticeCell::meet(const LatticeCell &L) {
   502:   bool Changed = false;
   503:   if (L.isBottom())
   504:     Changed = setBottom();
   505:   if (isBottom() || L.isTop())
   506:     return Changed;
   507:   if (isTop()) {
   508:     *this = L;
   509:     // L can be neither Top nor Bottom, so *this must have changed.
   510:     return true;
   511:   }
   512: 
   513:   // Top/bottom cases covered. Need to integrate L's set into ours.
   514:   if (L.isProperty())
   515:     return add(L.properties());
   516:   for (unsigned i = 0; i < L.size(); ++i) {
   517:     const Constant *LC = L.Values[i];
   518:     Changed |= add(LC);
   519:   }
   520:   return Changed;
   521: }
   522: 
   523: // Add a new constant to the cell. This is actually where the cell update
   524: // happens. If a cell has room for more constants, the new constant is added.
   525: // Otherwise, the cell is converted to a "property" cell (i.e. a cell that
   526: // will track properties of the associated values, and not the values
   527: // themselves. Care is taken to handle special cases, like "bottom", etc.
   528: bool LatticeCell::add(const Constant *LC) {
   529:   assert(LC);
   530:   if (isBottom())
   531:     return false;
   532: 
   533:   if (!isProperty()) {
   534:     // Cell is not special. Try to add the constant here first,
   535:     // if there is room.
   536:     unsigned Index = 0;
   537:     while (Index < Size) {
   538:       const Constant *C = Values[Index];
   539:       // If the constant is already here, no change is needed.
   540:       if (C == LC)
   541:         return false;
   542:       Index++;
   543:     }
   544:     if (Index < MaxCellSize) {
   545:       Values[Index] = LC;
   546:       Kind = Normal;
   547:       Size++;
   548:       return true;
   549:     }
   550:   }
   551: 
   552:   bool Changed = false;
   553: 
   554:   // This cell is special, or is not special, but is full. After this
   555:   // it will be special.
   556:   Changed = convertToProperty();
   557:   uint32_t Ps = properties();
   558:   uint32_t NewPs = Ps & ConstantProperties::deduce(LC);
   559:   if (NewPs == ConstantProperties::Unknown) {
   560:     setBottom();
   561:     return true;
   562:   }
   563:   if (Ps != NewPs) {
   564:     Properties = NewPs;
   565:     Changed = true;
   566:   }
   567:   return Changed;
   568: }
   569: 
   570: // Add a property to the cell. This will force the cell to become a property-
   571: // tracking cell.
   572: bool LatticeCell::add(uint32_t Property) {
   573:   bool Changed = convertToProperty();
   574:   uint32_t Ps = properties();
   575:   if (Ps == (Ps & Property))
   576:     return Changed;
   577:   Properties = Property & Ps;
   578:   return true;
   579: }
   580: 
   581: // Return the properties of the values in the cell. This is valid for any
   582: // cell, and does not alter the cell itself.
   583: uint32_t LatticeCell::properties() const {
   584:   if (isProperty())
   585:     return Properties;
   586:   assert(!isTop() && "Should not call this for a top cell");
   587:   if (isBottom())
   588:     return ConstantProperties::Unknown;
   589: 
   590:   assert(size() > 0 && "Empty cell");
   591:   uint32_t Ps = ConstantProperties::deduce(Values[0]);
   592:   for (unsigned i = 1; i < size(); ++i) {
   593:     if (Ps == ConstantProperties::Unknown)
   594:       break;
   595:     Ps &= ConstantProperties::deduce(Values[i]);
   596:   }
   597:   return Ps;
   598: }
   599: 
   600: #ifndef NDEBUG
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (uint32_t) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as evaluateSplatr, evaluateSplati, ConstantProperties::deduce, cast<ConstantInt>, ... (20 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（uint32_t），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 evaluateSplatr, evaluateSplati, ConstantProperties::deduce, cast<ConstantInt>, ... (20 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 601-800 / 第 601-800 行

```cpp
   601: void MachineConstPropagator::CellMap::print(raw_ostream &os,
   602:       const TargetRegisterInfo &TRI) const {
   603:   for (auto &I : Map)
   604:     dbgs() << "  " << printReg(I.first, &TRI) << " -> " << I.second << '\n';
   605: }
   606: #endif
   607: 
   608: void MachineConstPropagator::visitPHI(const MachineInstr &PN) {
   609:   const MachineBasicBlock *MB = PN.getParent();
   610:   unsigned MBN = MB->getNumber();
   611:   LLVM_DEBUG(dbgs() << "Visiting FI(" << printMBBReference(*MB) << "): " << PN);
   612: 
   613:   const MachineOperand &MD = PN.getOperand(0);
   614:   RegSubRegPair DefR(getRegSubRegPair(MD));
   615:   assert(DefR.Reg.isVirtual());
   616: 
   617:   bool Changed = false;
   618: 
   619:   // If the def has a sub-register, set the corresponding cell to "bottom".
   620:   if (DefR.SubReg) {
   621: Bottomize:
   622:     const LatticeCell &T = Cells.get(DefR.Reg);
   623:     Changed = !T.isBottom();
   624:     Cells.update(DefR.Reg, Bottom);
   625:     if (Changed)
   626:       visitUsesOf(DefR.Reg);
   627:     return;
   628:   }
   629: 
   630:   LatticeCell DefC = Cells.get(DefR.Reg);
   631: 
   632:   for (unsigned i = 1, n = PN.getNumOperands(); i < n; i += 2) {
   633:     const MachineBasicBlock *PB = PN.getOperand(i+1).getMBB();
   634:     unsigned PBN = PB->getNumber();
   635:     if (!EdgeExec.count(CFGEdge(PBN, MBN))) {
   636:       LLVM_DEBUG(dbgs() << "  edge " << printMBBReference(*PB) << "->"
   637:                         << printMBBReference(*MB) << " not executable\n");
   638:       continue;
   639:     }
   640:     const MachineOperand &SO = PN.getOperand(i);
   641:     RegSubRegPair UseR(getRegSubRegPair(SO));
   642:     // If the input is not a virtual register, we don't really know what
   643:     // value it holds.
   644:     if (!UseR.Reg.isVirtual())
   645:       goto Bottomize;
   646:     // If there is no cell for an input register, it means top.
   647:     if (!Cells.has(UseR.Reg))
   648:       continue;
   649: 
   650:     LatticeCell SrcC;
   651:     bool Eval = MCE.evaluate(UseR, Cells.get(UseR.Reg), SrcC);
   652:     LLVM_DEBUG(dbgs() << "  edge from " << printMBBReference(*PB) << ": "
   653:                       << printReg(UseR.Reg, &MCE.TRI, UseR.SubReg) << SrcC
   654:                       << '\n');
   655:     Changed |= Eval ? DefC.meet(SrcC)
   656:                     : DefC.setBottom();
   657:     Cells.update(DefR.Reg, DefC);
   658:     if (DefC.isBottom())
   659:       break;
   660:   }
   661:   if (Changed)
   662:     visitUsesOf(DefR.Reg);
   663: }
   664: 
   665: void MachineConstPropagator::visitNonBranch(const MachineInstr &MI) {
   666:   LLVM_DEBUG(dbgs() << "Visiting MI(" << printMBBReference(*MI.getParent())
   667:                     << "): " << MI);
   668:   CellMap Outputs;
   669:   bool Eval = MCE.evaluate(MI, Cells, Outputs);
   670:   LLVM_DEBUG({
   671:     if (Eval) {
   672:       dbgs() << "  outputs:";
   673:       for (auto &I : Outputs)
   674:         dbgs() << ' ' << I.second;
   675:       dbgs() << '\n';
   676:     }
   677:   });
   678: 
   679:   // Update outputs. If the value was not computed, set all the
   680:   // def cells to bottom.
   681:   for (const MachineOperand &MO : MI.operands()) {
   682:     if (!MO.isReg() || !MO.isDef())
   683:       continue;
   684:     RegSubRegPair DefR(getRegSubRegPair(MO));
   685:     // Only track virtual registers.
   686:     if (!DefR.Reg.isVirtual())
   687:       continue;
   688:     bool Changed = false;
   689:     // If the evaluation failed, set cells for all output registers to bottom.
   690:     if (!Eval) {
   691:       const LatticeCell &T = Cells.get(DefR.Reg);
   692:       Changed = !T.isBottom();
   693:       Cells.update(DefR.Reg, Bottom);
   694:     } else {
   695:       // Find the corresponding cell in the computed outputs.
   696:       // If it's not there, go on to the next def.
   697:       if (!Outputs.has(DefR.Reg))
   698:         continue;
   699:       LatticeCell RC = Cells.get(DefR.Reg);
   700:       Changed = RC.meet(Outputs.get(DefR.Reg));
   701:       Cells.update(DefR.Reg, RC);
   702:     }
   703:     if (Changed)
   704:       visitUsesOf(DefR.Reg);
   705:   }
   706: }
   707: 
   708: // Starting at a given branch, visit remaining branches in the block.
   709: // Traverse over the subsequent branches for as long as the preceding one
   710: // can fall through. Add all the possible targets to the flow work queue,
   711: // including the potential fall-through to the layout-successor block.
   712: void MachineConstPropagator::visitBranchesFrom(const MachineInstr &BrI) {
   713:   const MachineBasicBlock &B = *BrI.getParent();
   714:   unsigned MBN = B.getNumber();
   715:   MachineBasicBlock::const_iterator It = BrI.getIterator();
   716:   MachineBasicBlock::const_iterator End = B.end();
   717: 
   718:   SetVector<const MachineBasicBlock*> Targets;
   719:   bool EvalOk = true, FallsThru = true;
   720:   while (It != End) {
   721:     const MachineInstr &MI = *It;
   722:     InstrExec.insert(&MI);
   723:     LLVM_DEBUG(dbgs() << "Visiting " << (EvalOk ? "BR" : "br") << "("
   724:                       << printMBBReference(B) << "): " << MI);
   725:     // Do not evaluate subsequent branches if the evaluation of any of the
   726:     // previous branches failed. Keep iterating over the branches only
   727:     // to mark them as executable.
   728:     EvalOk = EvalOk && MCE.evaluate(MI, Cells, Targets, FallsThru);
   729:     if (!EvalOk)
   730:       FallsThru = true;
   731:     if (!FallsThru)
   732:       break;
   733:     ++It;
   734:   }
   735: 
   736:   if (B.mayHaveInlineAsmBr())
   737:     EvalOk = false;
   738: 
   739:   if (EvalOk) {
   740:     // Need to add all CFG successors that lead to EH landing pads.
   741:     // There won't be explicit branches to these blocks, but they must
   742:     // be processed.
   743:     for (const MachineBasicBlock *SB : B.successors()) {
   744:       if (SB->isEHPad())
   745:         Targets.insert(SB);
   746:     }
   747:     if (FallsThru) {
   748:       const MachineFunction &MF = *B.getParent();
   749:       MachineFunction::const_iterator BI = B.getIterator();
   750:       MachineFunction::const_iterator Next = std::next(BI);
   751:       if (Next != MF.end())
   752:         Targets.insert(&*Next);
   753:     }
   754:   } else {
   755:     // If the evaluation of the branches failed, make "Targets" to be the
   756:     // set of all successors of the block from the CFG.
   757:     // If the evaluation succeeded for all visited branches, then if the
   758:     // last one set "FallsThru", then add an edge to the layout successor
   759:     // to the targets.
   760:     Targets.clear();
   761:     LLVM_DEBUG(dbgs() << "  failed to evaluate a branch...adding all CFG "
   762:                          "successors\n");
   763:     Targets.insert_range(B.successors());
   764:   }
   765: 
   766:   for (const MachineBasicBlock *TB : Targets) {
   767:     unsigned TBN = TB->getNumber();
   768:     LLVM_DEBUG(dbgs() << "  pushing edge " << printMBBReference(B) << " -> "
   769:                       << printMBBReference(*TB) << "\n");
   770:     FlowQ.push(CFGEdge(MBN, TBN));
   771:   }
   772: }
   773: 
   774: void MachineConstPropagator::visitUsesOf(unsigned Reg) {
   775:   LLVM_DEBUG(dbgs() << "Visiting uses of " << printReg(Reg, &MCE.TRI)
   776:                     << Cells.get(Reg) << '\n');
   777:   for (MachineInstr &MI : MRI->use_nodbg_instructions(Reg)) {
   778:     // Do not process non-executable instructions. They can become executable
   779:     // later (via a flow-edge in the work queue). In such case, the instruc-
   780:     // tion will be visited at that time.
   781:     if (!InstrExec.count(&MI))
   782:       continue;
   783:     if (MI.isPHI())
   784:       visitPHI(MI);
   785:     else if (!MI.isBranch())
   786:       visitNonBranch(MI);
   787:     else
   788:       visitBranchesFrom(MI);
   789:   }
   790: }
   791: 
   792: bool MachineConstPropagator::computeBlockSuccessors(const MachineBasicBlock *MB,
   793:       SetVector<const MachineBasicBlock*> &Targets) {
   794:   Targets.clear();
   795: 
   796:   MachineBasicBlock::const_iterator FirstBr = MB->end();
   797:   for (const MachineInstr &MI : *MB) {
   798:     if (MI.getOpcode() == TargetOpcode::INLINEASM_BR)
   799:       return false;
   800:     if (MI.isDebugInstr())
```
- EN: It declares or implements routines such as MachineConstPropagator::CellMap::print, MachineConstPropagator::visitPHI, getParent, getNumber, ... (26 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 MachineConstPropagator::CellMap::print, MachineConstPropagator::visitPHI, getParent, getNumber, ... (26 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 801-1000 / 第 801-1000 行

```cpp
   801:       continue;
   802:     if (MI.isBranch()) {
   803:       FirstBr = MI.getIterator();
   804:       break;
   805:     }
   806:   }
   807: 
   808:   MachineBasicBlock::const_iterator End = MB->end();
   809: 
   810:   bool DoNext = true;
   811:   for (MachineBasicBlock::const_iterator I = FirstBr; I != End; ++I) {
   812:     const MachineInstr &MI = *I;
   813:     // Can there be debug instructions between branches?
   814:     if (MI.isDebugInstr())
   815:       continue;
   816:     if (!InstrExec.count(&MI))
   817:       continue;
   818:     bool Eval = MCE.evaluate(MI, Cells, Targets, DoNext);
   819:     if (!Eval)
   820:       return false;
   821:     if (!DoNext)
   822:       break;
   823:   }
   824:   // If the last branch could fall-through, add block's layout successor.
   825:   if (DoNext) {
   826:     MachineFunction::const_iterator BI = MB->getIterator();
   827:     MachineFunction::const_iterator NextI = std::next(BI);
   828:     if (NextI != MB->getParent()->end())
   829:       Targets.insert(&*NextI);
   830:   }
   831: 
   832:   // Add all the EH landing pads.
   833:   for (const MachineBasicBlock *SB : MB->successors())
   834:     if (SB->isEHPad())
   835:       Targets.insert(SB);
   836: 
   837:   return true;
   838: }
   839: 
   840: void MachineConstPropagator::removeCFGEdge(MachineBasicBlock *From,
   841:       MachineBasicBlock *To) {
   842:   // First, remove the CFG successor/predecessor information.
   843:   From->removeSuccessor(To);
   844:   // Remove all corresponding PHI operands in the To block.
   845:   for (MachineInstr &PN : To->phis()) {
   846:     // reg0 = PHI reg1, bb2, reg3, bb4, ...
   847:     int N = PN.getNumOperands() - 2;
   848:     while (N > 0) {
   849:       if (PN.getOperand(N + 1).getMBB() == From) {
   850:         PN.removeOperand(N + 1);
   851:         PN.removeOperand(N);
   852:       }
   853:       N -= 2;
   854:     }
   855:   }
   856: }
   857: 
   858: void MachineConstPropagator::propagate(MachineFunction &MF) {
   859:   MachineBasicBlock *Entry = GraphTraits<MachineFunction*>::getEntryNode(&MF);
   860:   unsigned EntryNum = Entry->getNumber();
   861: 
   862:   // Start with a fake edge, just to process the entry node.
   863:   FlowQ.push(CFGEdge(EntryNum, EntryNum));
   864: 
   865:   while (!FlowQ.empty()) {
   866:     CFGEdge Edge = FlowQ.front();
   867:     FlowQ.pop();
   868: 
   869:     LLVM_DEBUG(
   870:         dbgs() << "Picked edge "
   871:                << printMBBReference(*MF.getBlockNumbered(Edge.first)) << "->"
   872:                << printMBBReference(*MF.getBlockNumbered(Edge.second)) << '\n');
   873:     if (Edge.first != EntryNum)
   874:       if (EdgeExec.count(Edge))
   875:         continue;
   876:     EdgeExec.insert(Edge);
   877:     MachineBasicBlock *SB = MF.getBlockNumbered(Edge.second);
   878: 
   879:     // Process the block in three stages:
   880:     // - visit all PHI nodes,
   881:     // - visit all non-branch instructions,
   882:     // - visit block branches.
   883:     MachineBasicBlock::const_iterator It = SB->begin(), End = SB->end();
   884: 
   885:     // Visit PHI nodes in the successor block.
   886:     while (It != End && It->isPHI()) {
   887:       InstrExec.insert(&*It);
   888:       visitPHI(*It);
   889:       ++It;
   890:     }
   891: 
   892:     // If the successor block just became executable, visit all instructions.
   893:     // To see if this is the first time we're visiting it, check the first
   894:     // non-debug instruction to see if it is executable.
   895:     while (It != End && It->isDebugInstr())
   896:       ++It;
   897:     assert(It == End || !It->isPHI());
   898:     // If this block has been visited, go on to the next one.
   899:     if (It != End && InstrExec.count(&*It))
   900:       continue;
   901:     // For now, scan all non-branch instructions. Branches require different
   902:     // processing.
   903:     while (It != End && !It->isBranch()) {
   904:       if (!It->isDebugInstr()) {
   905:         InstrExec.insert(&*It);
   906:         visitNonBranch(*It);
   907:       }
   908:       ++It;
   909:     }
   910: 
   911:     // Time to process the end of the block. This is different from
   912:     // processing regular (non-branch) instructions, because there can
   913:     // be multiple branches in a block, and they can cause the block to
   914:     // terminate early.
   915:     if (It != End) {
   916:       visitBranchesFrom(*It);
   917:     } else {
   918:       // If the block didn't have a branch, add all successor edges to the
   919:       // work queue. (There should really be only one successor in such case.)
   920:       unsigned SBN = SB->getNumber();
   921:       for (const MachineBasicBlock *SSB : SB->successors())
   922:         FlowQ.push(CFGEdge(SBN, SSB->getNumber()));
   923:     }
   924:   } // while (FlowQ)
   925: 
   926:   LLVM_DEBUG({
   927:     dbgs() << "Cells after propagation:\n";
   928:     Cells.print(dbgs(), MCE.TRI);
   929:     dbgs() << "Dead CFG edges:\n";
   930:     for (const MachineBasicBlock &B : MF) {
   931:       unsigned BN = B.getNumber();
   932:       for (const MachineBasicBlock *SB : B.successors()) {
   933:         unsigned SN = SB->getNumber();
   934:         if (!EdgeExec.count(CFGEdge(BN, SN)))
   935:           dbgs() << "  " << printMBBReference(B) << " -> "
   936:                  << printMBBReference(*SB) << '\n';
   937:       }
   938:     }
   939:   });
   940: }
   941: 
   942: bool MachineConstPropagator::rewrite(MachineFunction &MF) {
   943:   bool Changed = false;
   944:   // Rewrite all instructions based on the collected cell information.
   945:   //
   946:   // Traverse the instructions in a post-order, so that rewriting an
   947:   // instruction can make changes "downstream" in terms of control-flow
   948:   // without affecting the rewriting process. (We should not change
   949:   // instructions that have not yet been visited by the rewriter.)
   950:   // The reason for this is that the rewriter can introduce new vregs,
   951:   // and replace uses of old vregs (which had corresponding cells
   952:   // computed during propagation) with these new vregs (which at this
   953:   // point would not have any cells, and would appear to be "top").
   954:   // If an attempt was made to evaluate an instruction with a fresh
   955:   // "top" vreg, it would cause an error (abend) in the evaluator.
   956: 
   957:   // Collect the post-order-traversal block ordering. The subsequent
   958:   // traversal/rewrite will update block successors, so it's safer
   959:   // if the visiting order it computed ahead of time.
   960:   std::vector<MachineBasicBlock*> POT;
   961:   for (MachineBasicBlock *B : post_order(&MF))
   962:     if (!B->empty())
   963:       POT.push_back(B);
   964: 
   965:   for (MachineBasicBlock *B : POT) {
   966:     // Walk the block backwards (which usually begin with the branches).
   967:     // If any branch is rewritten, we may need to update the successor
   968:     // information for this block. Unless the block's successors can be
   969:     // precisely determined (which may not be the case for indirect
   970:     // branches), we cannot modify any branch.
   971: 
   972:     // Compute the successor information.
   973:     SetVector<const MachineBasicBlock*> Targets;
   974:     bool HaveTargets = computeBlockSuccessors(B, Targets);
   975:     // Rewrite the executable instructions. Skip branches if we don't
   976:     // have block successor information.
   977:     for (MachineInstr &MI : llvm::reverse(*B)) {
   978:       if (InstrExec.count(&MI)) {
   979:         if (MI.isBranch() && !HaveTargets)
   980:           continue;
   981:         Changed |= MCE.rewrite(MI, Cells);
   982:       }
   983:     }
   984:     // The rewriting could rewrite PHI nodes to non-PHI nodes, causing
   985:     // regular instructions to appear in between PHI nodes. Bring all
   986:     // the PHI nodes to the beginning of the block.
   987:     for (auto I = B->begin(), E = B->end(); I != E; ++I) {
   988:       if (I->isPHI())
   989:         continue;
   990:       // I is not PHI. Find the next PHI node P.
   991:       auto P = I;
   992:       while (++P != E)
   993:         if (P->isPHI())
   994:           break;
   995:       // Not found.
   996:       if (P == E)
   997:         break;
   998:       // Splice P right before I.
   999:       B->splice(I, B, P);
  1000:       // Reset I to point at the just spliced PHI node.
```
- EN: It declares or implements routines such as getIterator, end, evaluate, std::next, ... (27 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 getIterator, end, evaluate, std::next, ... (27 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 1001-1200 / 第 1001-1200 行

```cpp
  1001:       --I;
  1002:     }
  1003:     // Update the block successor information: remove unnecessary successors.
  1004:     if (HaveTargets) {
  1005:       SmallVector<MachineBasicBlock*,2> ToRemove;
  1006:       for (MachineBasicBlock *SB : B->successors()) {
  1007:         if (!Targets.count(SB))
  1008:           ToRemove.push_back(SB);
  1009:         Targets.remove(SB);
  1010:       }
  1011:       for (MachineBasicBlock *MBB : ToRemove)
  1012:         removeCFGEdge(B, MBB);
  1013:       // If there are any blocks left in the computed targets, it means that
  1014:       // we think that the block could go somewhere, but the CFG does not.
  1015:       // This could legitimately happen in blocks that have non-returning
  1016:       // calls---we would think that the execution can continue, but the
  1017:       // CFG will not have a successor edge.
  1018:     }
  1019:   }
  1020:   // Need to do some final post-processing.
  1021:   // If a branch was not executable, it will not get rewritten, but should
  1022:   // be removed (or replaced with something equivalent to a A2_nop). We can't
  1023:   // erase instructions during rewriting, so this needs to be delayed until
  1024:   // now.
  1025:   for (MachineBasicBlock &B : MF) {
  1026:     for (MachineInstr &MI : llvm::make_early_inc_range(B))
  1027:       if (MI.isBranch() && !InstrExec.count(&MI))
  1028:         B.erase(&MI);
  1029:   }
  1030:   return Changed;
  1031: }
  1032: 
  1033: // This is the constant propagation algorithm as described by Wegman-Zadeck.
  1034: // Most of the terminology comes from there.
  1035: bool MachineConstPropagator::run(MachineFunction &MF) {
  1036:   LLVM_DEBUG(MF.print(dbgs() << "Starting MachineConstPropagator\n", nullptr));
  1037: 
  1038:   MRI = &MF.getRegInfo();
  1039: 
  1040:   Cells.clear();
  1041:   EdgeExec.clear();
  1042:   InstrExec.clear();
  1043:   assert(FlowQ.empty());
  1044: 
  1045:   propagate(MF);
  1046:   bool Changed = rewrite(MF);
  1047: 
  1048:   LLVM_DEBUG({
  1049:     dbgs() << "End of MachineConstPropagator (Changed=" << Changed << ")\n";
  1050:     if (Changed)
  1051:       MF.print(dbgs(), nullptr);
  1052:   });
  1053:   return Changed;
  1054: }
  1055: 
  1056: // --------------------------------------------------------------------
  1057: // Machine const evaluator.
  1058: 
  1059: bool MachineConstEvaluator::getCell(const RegSubRegPair &R,
  1060:                                     const CellMap &Inputs, LatticeCell &RC) {
  1061:   if (!R.Reg.isVirtual())
  1062:     return false;
  1063:   const LatticeCell &L = Inputs.get(R.Reg);
  1064:   if (!R.SubReg) {
  1065:     RC = L;
  1066:     return !RC.isBottom();
  1067:   }
  1068:   bool Eval = evaluate(R, L, RC);
  1069:   return Eval && !RC.isBottom();
  1070: }
  1071: 
  1072: bool MachineConstEvaluator::constToInt(const Constant *C,
  1073:       APInt &Val) const {
  1074:   const ConstantInt *CI = dyn_cast<ConstantInt>(C);
  1075:   if (!CI)
  1076:     return false;
  1077:   Val = CI->getValue();
  1078:   return true;
  1079: }
  1080: 
  1081: const ConstantInt *MachineConstEvaluator::intToConst(const APInt &Val) const {
  1082:   return ConstantInt::get(CX, Val);
  1083: }
  1084: 
  1085: bool MachineConstEvaluator::evaluateCMPrr(uint32_t Cmp, const RegSubRegPair &R1,
  1086:                                           const RegSubRegPair &R2,
  1087:                                           const CellMap &Inputs, bool &Result) {
  1088:   assert(Inputs.has(R1.Reg) && Inputs.has(R2.Reg));
  1089:   LatticeCell LS1, LS2;
  1090:   if (!getCell(R1, Inputs, LS1) || !getCell(R2, Inputs, LS2))
  1091:     return false;
  1092: 
  1093:   bool IsProp1 = LS1.isProperty();
  1094:   bool IsProp2 = LS2.isProperty();
  1095:   if (IsProp1) {
  1096:     uint32_t Prop1 = LS1.properties();
  1097:     if (IsProp2)
  1098:       return evaluateCMPpp(Cmp, Prop1, LS2.properties(), Result);
  1099:     uint32_t NegCmp = Comparison::negate(Cmp);
  1100:     return evaluateCMPrp(NegCmp, R2, Prop1, Inputs, Result);
  1101:   }
  1102:   if (IsProp2) {
  1103:     uint32_t Prop2 = LS2.properties();
  1104:     return evaluateCMPrp(Cmp, R1, Prop2, Inputs, Result);
  1105:   }
  1106: 
  1107:   APInt A;
  1108:   bool IsTrue = true, IsFalse = true;
  1109:   for (unsigned i = 0; i < LS2.size(); ++i) {
  1110:     bool Res;
  1111:     bool Computed = constToInt(LS2.Values[i], A) &&
  1112:                     evaluateCMPri(Cmp, R1, A, Inputs, Res);
  1113:     if (!Computed)
  1114:       return false;
  1115:     IsTrue &= Res;
  1116:     IsFalse &= !Res;
  1117:   }
  1118:   assert(!IsTrue || !IsFalse);
  1119:   // The actual logical value of the comparison is same as IsTrue.
  1120:   Result = IsTrue;
  1121:   // Return true if the result was proven to be true or proven to be false.
  1122:   return IsTrue || IsFalse;
  1123: }
  1124: 
  1125: bool MachineConstEvaluator::evaluateCMPri(uint32_t Cmp, const RegSubRegPair &R1,
  1126:                                           const APInt &A2,
  1127:                                           const CellMap &Inputs, bool &Result) {
  1128:   assert(Inputs.has(R1.Reg));
  1129:   LatticeCell LS;
  1130:   if (!getCell(R1, Inputs, LS))
  1131:     return false;
  1132:   if (LS.isProperty())
  1133:     return evaluateCMPpi(Cmp, LS.properties(), A2, Result);
  1134: 
  1135:   APInt A;
  1136:   bool IsTrue = true, IsFalse = true;
  1137:   for (unsigned i = 0; i < LS.size(); ++i) {
  1138:     bool Res;
  1139:     bool Computed = constToInt(LS.Values[i], A) &&
  1140:                     evaluateCMPii(Cmp, A, A2, Res);
  1141:     if (!Computed)
  1142:       return false;
  1143:     IsTrue &= Res;
  1144:     IsFalse &= !Res;
  1145:   }
  1146:   assert(!IsTrue || !IsFalse);
  1147:   // The actual logical value of the comparison is same as IsTrue.
  1148:   Result = IsTrue;
  1149:   // Return true if the result was proven to be true or proven to be false.
  1150:   return IsTrue || IsFalse;
  1151: }
  1152: 
  1153: bool MachineConstEvaluator::evaluateCMPrp(uint32_t Cmp, const RegSubRegPair &R1,
  1154:                                           uint64_t Props2,
  1155:                                           const CellMap &Inputs, bool &Result) {
  1156:   assert(Inputs.has(R1.Reg));
  1157:   LatticeCell LS;
  1158:   if (!getCell(R1, Inputs, LS))
  1159:     return false;
  1160:   if (LS.isProperty())
  1161:     return evaluateCMPpp(Cmp, LS.properties(), Props2, Result);
  1162: 
  1163:   APInt A;
  1164:   uint32_t NegCmp = Comparison::negate(Cmp);
  1165:   bool IsTrue = true, IsFalse = true;
  1166:   for (unsigned i = 0; i < LS.size(); ++i) {
  1167:     bool Res;
  1168:     bool Computed = constToInt(LS.Values[i], A) &&
  1169:                     evaluateCMPpi(NegCmp, Props2, A, Res);
  1170:     if (!Computed)
  1171:       return false;
  1172:     IsTrue &= Res;
  1173:     IsFalse &= !Res;
  1174:   }
  1175:   assert(!IsTrue || !IsFalse);
  1176:   Result = IsTrue;
  1177:   return IsTrue || IsFalse;
  1178: }
  1179: 
  1180: bool MachineConstEvaluator::evaluateCMPii(uint32_t Cmp, const APInt &A1,
  1181:       const APInt &A2, bool &Result) {
  1182:   // NE is a special kind of comparison (not composed of smaller properties).
  1183:   if (Cmp == Comparison::NE) {
  1184:     Result = !APInt::isSameValue(A1, A2);
  1185:     return true;
  1186:   }
  1187:   if (Cmp == Comparison::EQ) {
  1188:     Result = APInt::isSameValue(A1, A2);
  1189:     return true;
  1190:   }
  1191:   if (Cmp & Comparison::EQ) {
  1192:     if (APInt::isSameValue(A1, A2))
  1193:       return (Result = true);
  1194:   }
  1195:   assert((Cmp & (Comparison::L | Comparison::G)) && "Malformed comparison");
  1196:   Result = false;
  1197: 
  1198:   unsigned W1 = A1.getBitWidth();
  1199:   unsigned W2 = A2.getBitWidth();
  1200:   unsigned MaxW = (W1 >= W2) ? W1 : W2;
```
- EN: It declares or implements routines such as remove, removed, MachineConstPropagator::run, LLVM_DEBUG, ... (31 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 remove, removed, MachineConstPropagator::run, LLVM_DEBUG, ... (31 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 1201-1400 / 第 1201-1400 行

```cpp
  1201:   if (Cmp & Comparison::U) {
  1202:     APInt Zx1 = A1.zext(MaxW);
  1203:     APInt Zx2 = A2.zext(MaxW);
  1204:     if (Cmp & Comparison::L)
  1205:       Result = Zx1.ult(Zx2);
  1206:     else if (Cmp & Comparison::G)
  1207:       Result = Zx2.ult(Zx1);
  1208:     return true;
  1209:   }
  1210: 
  1211:   // Signed comparison.
  1212:   APInt Sx1 = A1.sext(MaxW);
  1213:   APInt Sx2 = A2.sext(MaxW);
  1214:   if (Cmp & Comparison::L)
  1215:     Result = Sx1.slt(Sx2);
  1216:   else if (Cmp & Comparison::G)
  1217:     Result = Sx2.slt(Sx1);
  1218:   return true;
  1219: }
  1220: 
  1221: bool MachineConstEvaluator::evaluateCMPpi(uint32_t Cmp, uint32_t Props,
  1222:       const APInt &A2, bool &Result) {
  1223:   if (Props == ConstantProperties::Unknown)
  1224:     return false;
  1225: 
  1226:   // Should never see NaN here, but check for it for completeness.
  1227:   if (Props & ConstantProperties::NaN)
  1228:     return false;
  1229:   // Infinity could theoretically be compared to a number, but the
  1230:   // presence of infinity here would be very suspicious. If we don't
  1231:   // know for sure that the number is finite, bail out.
  1232:   if (!(Props & ConstantProperties::Finite))
  1233:     return false;
  1234: 
  1235:   // Let X be a number that has properties Props.
  1236: 
  1237:   if (Cmp & Comparison::U) {
  1238:     // In case of unsigned comparisons, we can only compare against 0.
  1239:     if (A2 == 0) {
  1240:       // Any x!=0 will be considered >0 in an unsigned comparison.
  1241:       if (Props & ConstantProperties::Zero)
  1242:         Result = (Cmp & Comparison::EQ);
  1243:       else if (Props & ConstantProperties::NonZero)
  1244:         Result = (Cmp & Comparison::G) || (Cmp == Comparison::NE);
  1245:       else
  1246:         return false;
  1247:       return true;
  1248:     }
  1249:     // A2 is not zero. The only handled case is if X = 0.
  1250:     if (Props & ConstantProperties::Zero) {
  1251:       Result = (Cmp & Comparison::L) || (Cmp == Comparison::NE);
  1252:       return true;
  1253:     }
  1254:     return false;
  1255:   }
  1256: 
  1257:   // Signed comparisons are different.
  1258:   if (Props & ConstantProperties::Zero) {
  1259:     if (A2 == 0)
  1260:       Result = (Cmp & Comparison::EQ);
  1261:     else
  1262:       Result = (Cmp == Comparison::NE) ||
  1263:                ((Cmp & Comparison::L) && !A2.isNegative()) ||
  1264:                ((Cmp & Comparison::G) &&  A2.isNegative());
  1265:     return true;
  1266:   }
  1267:   if (Props & ConstantProperties::PosOrZero) {
  1268:     // X >= 0 and !(A2 < 0) => cannot compare
  1269:     if (!A2.isNegative())
  1270:       return false;
  1271:     // X >= 0 and A2 < 0
  1272:     Result = (Cmp & Comparison::G) || (Cmp == Comparison::NE);
  1273:     return true;
  1274:   }
  1275:   if (Props & ConstantProperties::NegOrZero) {
  1276:     // X <= 0 and Src1 < 0 => cannot compare
  1277:     if (A2 == 0 || A2.isNegative())
  1278:       return false;
  1279:     // X <= 0 and A2 > 0
  1280:     Result = (Cmp & Comparison::L) || (Cmp == Comparison::NE);
  1281:     return true;
  1282:   }
  1283: 
  1284:   return false;
  1285: }
  1286: 
  1287: bool MachineConstEvaluator::evaluateCMPpp(uint32_t Cmp, uint32_t Props1,
  1288:       uint32_t Props2, bool &Result) {
  1289:   using P = ConstantProperties;
  1290: 
  1291:   if ((Props1 & P::NaN) && (Props2 & P::NaN))
  1292:     return false;
  1293:   if (!(Props1 & P::Finite) || !(Props2 & P::Finite))
  1294:     return false;
  1295: 
  1296:   bool Zero1 = (Props1 & P::Zero), Zero2 = (Props2 & P::Zero);
  1297:   bool NonZero1 = (Props1 & P::NonZero), NonZero2 = (Props2 & P::NonZero);
  1298:   if (Zero1 && Zero2) {
  1299:     Result = (Cmp & Comparison::EQ);
  1300:     return true;
  1301:   }
  1302:   if (Cmp == Comparison::NE) {
  1303:     if ((Zero1 && NonZero2) || (NonZero1 && Zero2))
  1304:       return (Result = true);
  1305:     return false;
  1306:   }
  1307: 
  1308:   if (Cmp & Comparison::U) {
  1309:     // In unsigned comparisons, we can only compare against a known zero,
  1310:     // or a known non-zero.
  1311:     if (Zero1 && NonZero2) {
  1312:       Result = (Cmp & Comparison::L);
  1313:       return true;
  1314:     }
  1315:     if (NonZero1 && Zero2) {
  1316:       Result = (Cmp & Comparison::G);
  1317:       return true;
  1318:     }
  1319:     return false;
  1320:   }
  1321: 
  1322:   // Signed comparison. The comparison is not NE.
  1323:   bool Poz1 = (Props1 & P::PosOrZero), Poz2 = (Props2 & P::PosOrZero);
  1324:   bool Nez1 = (Props1 & P::NegOrZero), Nez2 = (Props2 & P::NegOrZero);
  1325:   if (Nez1 && Poz2) {
  1326:     if (NonZero1 || NonZero2) {
  1327:       Result = (Cmp & Comparison::L);
  1328:       return true;
  1329:     }
  1330:     // Either (or both) could be zero. Can only say that X <= Y.
  1331:     if ((Cmp & Comparison::EQ) && (Cmp & Comparison::L))
  1332:       return (Result = true);
  1333:   }
  1334:   if (Poz1 && Nez2) {
  1335:     if (NonZero1 || NonZero2) {
  1336:       Result = (Cmp & Comparison::G);
  1337:       return true;
  1338:     }
  1339:     // Either (or both) could be zero. Can only say that X >= Y.
  1340:     if ((Cmp & Comparison::EQ) && (Cmp & Comparison::G))
  1341:       return (Result = true);
  1342:   }
  1343: 
  1344:   return false;
  1345: }
  1346: 
  1347: bool MachineConstEvaluator::evaluateCOPY(const RegSubRegPair &R1,
  1348:                                          const CellMap &Inputs,
  1349:                                          LatticeCell &Result) {
  1350:   return getCell(R1, Inputs, Result);
  1351: }
  1352: 
  1353: bool MachineConstEvaluator::evaluateANDrr(const RegSubRegPair &R1,
  1354:                                           const RegSubRegPair &R2,
  1355:                                           const CellMap &Inputs,
  1356:                                           LatticeCell &Result) {
  1357:   assert(Inputs.has(R1.Reg) && Inputs.has(R2.Reg));
  1358:   const LatticeCell &L1 = Inputs.get(R2.Reg);
  1359:   const LatticeCell &L2 = Inputs.get(R2.Reg);
  1360:   // If both sources are bottom, exit. Otherwise try to evaluate ANDri
  1361:   // with the non-bottom argument passed as the immediate. This is to
  1362:   // catch cases of ANDing with 0.
  1363:   if (L2.isBottom()) {
  1364:     if (L1.isBottom())
  1365:       return false;
  1366:     return evaluateANDrr(R2, R1, Inputs, Result);
  1367:   }
  1368:   LatticeCell LS2;
  1369:   if (!evaluate(R2, L2, LS2))
  1370:     return false;
  1371:   if (LS2.isBottom() || LS2.isProperty())
  1372:     return false;
  1373: 
  1374:   APInt A;
  1375:   for (unsigned i = 0; i < LS2.size(); ++i) {
  1376:     LatticeCell RC;
  1377:     bool Eval = constToInt(LS2.Values[i], A) &&
  1378:                 evaluateANDri(R1, A, Inputs, RC);
  1379:     if (!Eval)
  1380:       return false;
  1381:     Result.meet(RC);
  1382:   }
  1383:   return !Result.isBottom();
  1384: }
  1385: 
  1386: bool MachineConstEvaluator::evaluateANDri(const RegSubRegPair &R1,
  1387:                                           const APInt &A2,
  1388:                                           const CellMap &Inputs,
  1389:                                           LatticeCell &Result) {
  1390:   assert(Inputs.has(R1.Reg));
  1391:   if (A2 == -1)
  1392:     return getCell(R1, Inputs, Result);
  1393:   if (A2 == 0) {
  1394:     LatticeCell RC;
  1395:     RC.add(intToConst(A2));
  1396:     // Overwrite Result.
  1397:     Result = RC;
  1398:     return true;
  1399:   }
  1400:   LatticeCell LS1;
```
- EN: It declares or implements routines such as zext, sext, MachineConstEvaluator::evaluateCMPpi, isNegative, ... (18 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 zext, sext, MachineConstEvaluator::evaluateCMPpi, isNegative, ... (18 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 1401-1600 / 第 1401-1600 行

```cpp
  1401:   if (!getCell(R1, Inputs, LS1))
  1402:     return false;
  1403:   if (LS1.isBottom() || LS1.isProperty())
  1404:     return false;
  1405: 
  1406:   APInt A, ResA;
  1407:   for (unsigned i = 0; i < LS1.size(); ++i) {
  1408:     bool Eval = constToInt(LS1.Values[i], A) &&
  1409:                 evaluateANDii(A, A2, ResA);
  1410:     if (!Eval)
  1411:       return false;
  1412:     const Constant *C = intToConst(ResA);
  1413:     Result.add(C);
  1414:   }
  1415:   return !Result.isBottom();
  1416: }
  1417: 
  1418: bool MachineConstEvaluator::evaluateANDii(const APInt &A1,
  1419:       const APInt &A2, APInt &Result) {
  1420:   Result = A1 & A2;
  1421:   return true;
  1422: }
  1423: 
  1424: bool MachineConstEvaluator::evaluateORrr(const RegSubRegPair &R1,
  1425:                                          const RegSubRegPair &R2,
  1426:                                          const CellMap &Inputs,
  1427:                                          LatticeCell &Result) {
  1428:   assert(Inputs.has(R1.Reg) && Inputs.has(R2.Reg));
  1429:   const LatticeCell &L1 = Inputs.get(R2.Reg);
  1430:   const LatticeCell &L2 = Inputs.get(R2.Reg);
  1431:   // If both sources are bottom, exit. Otherwise try to evaluate ORri
  1432:   // with the non-bottom argument passed as the immediate. This is to
  1433:   // catch cases of ORing with -1.
  1434:   if (L2.isBottom()) {
  1435:     if (L1.isBottom())
  1436:       return false;
  1437:     return evaluateORrr(R2, R1, Inputs, Result);
  1438:   }
  1439:   LatticeCell LS2;
  1440:   if (!evaluate(R2, L2, LS2))
  1441:     return false;
  1442:   if (LS2.isBottom() || LS2.isProperty())
  1443:     return false;
  1444: 
  1445:   APInt A;
  1446:   for (unsigned i = 0; i < LS2.size(); ++i) {
  1447:     LatticeCell RC;
  1448:     bool Eval = constToInt(LS2.Values[i], A) &&
  1449:                 evaluateORri(R1, A, Inputs, RC);
  1450:     if (!Eval)
  1451:       return false;
  1452:     Result.meet(RC);
  1453:   }
  1454:   return !Result.isBottom();
  1455: }
  1456: 
  1457: bool MachineConstEvaluator::evaluateORri(const RegSubRegPair &R1,
  1458:                                          const APInt &A2, const CellMap &Inputs,
  1459:                                          LatticeCell &Result) {
  1460:   assert(Inputs.has(R1.Reg));
  1461:   if (A2 == 0)
  1462:     return getCell(R1, Inputs, Result);
  1463:   if (A2 == -1) {
  1464:     LatticeCell RC;
  1465:     RC.add(intToConst(A2));
  1466:     // Overwrite Result.
  1467:     Result = RC;
  1468:     return true;
  1469:   }
  1470:   LatticeCell LS1;
  1471:   if (!getCell(R1, Inputs, LS1))
  1472:     return false;
  1473:   if (LS1.isBottom() || LS1.isProperty())
  1474:     return false;
  1475: 
  1476:   APInt A, ResA;
  1477:   for (unsigned i = 0; i < LS1.size(); ++i) {
  1478:     bool Eval = constToInt(LS1.Values[i], A) &&
  1479:                 evaluateORii(A, A2, ResA);
  1480:     if (!Eval)
  1481:       return false;
  1482:     const Constant *C = intToConst(ResA);
  1483:     Result.add(C);
  1484:   }
  1485:   return !Result.isBottom();
  1486: }
  1487: 
  1488: bool MachineConstEvaluator::evaluateORii(const APInt &A1,
  1489:       const APInt &A2, APInt &Result) {
  1490:   Result = A1 | A2;
  1491:   return true;
  1492: }
  1493: 
  1494: bool MachineConstEvaluator::evaluateXORrr(const RegSubRegPair &R1,
  1495:                                           const RegSubRegPair &R2,
  1496:                                           const CellMap &Inputs,
  1497:                                           LatticeCell &Result) {
  1498:   assert(Inputs.has(R1.Reg) && Inputs.has(R2.Reg));
  1499:   LatticeCell LS1, LS2;
  1500:   if (!getCell(R1, Inputs, LS1) || !getCell(R2, Inputs, LS2))
  1501:     return false;
  1502:   if (LS1.isProperty()) {
  1503:     if (LS1.properties() & ConstantProperties::Zero)
  1504:       return !(Result = LS2).isBottom();
  1505:     return false;
  1506:   }
  1507:   if (LS2.isProperty()) {
  1508:     if (LS2.properties() & ConstantProperties::Zero)
  1509:       return !(Result = LS1).isBottom();
  1510:     return false;
  1511:   }
  1512: 
  1513:   APInt A;
  1514:   for (unsigned i = 0; i < LS2.size(); ++i) {
  1515:     LatticeCell RC;
  1516:     bool Eval = constToInt(LS2.Values[i], A) &&
  1517:                 evaluateXORri(R1, A, Inputs, RC);
  1518:     if (!Eval)
  1519:       return false;
  1520:     Result.meet(RC);
  1521:   }
  1522:   return !Result.isBottom();
  1523: }
  1524: 
  1525: bool MachineConstEvaluator::evaluateXORri(const RegSubRegPair &R1,
  1526:                                           const APInt &A2,
  1527:                                           const CellMap &Inputs,
  1528:                                           LatticeCell &Result) {
  1529:   assert(Inputs.has(R1.Reg));
  1530:   LatticeCell LS1;
  1531:   if (!getCell(R1, Inputs, LS1))
  1532:     return false;
  1533:   if (LS1.isProperty()) {
  1534:     if (LS1.properties() & ConstantProperties::Zero) {
  1535:       const Constant *C = intToConst(A2);
  1536:       Result.add(C);
  1537:       return !Result.isBottom();
  1538:     }
  1539:     return false;
  1540:   }
  1541: 
  1542:   APInt A, XA;
  1543:   for (unsigned i = 0; i < LS1.size(); ++i) {
  1544:     bool Eval = constToInt(LS1.Values[i], A) &&
  1545:                 evaluateXORii(A, A2, XA);
  1546:     if (!Eval)
  1547:       return false;
  1548:     const Constant *C = intToConst(XA);
  1549:     Result.add(C);
  1550:   }
  1551:   return !Result.isBottom();
  1552: }
  1553: 
  1554: bool MachineConstEvaluator::evaluateXORii(const APInt &A1,
  1555:       const APInt &A2, APInt &Result) {
  1556:   Result = A1 ^ A2;
  1557:   return true;
  1558: }
  1559: 
  1560: bool MachineConstEvaluator::evaluateZEXTr(const RegSubRegPair &R1,
  1561:                                           unsigned Width, unsigned Bits,
  1562:                                           const CellMap &Inputs,
  1563:                                           LatticeCell &Result) {
  1564:   assert(Inputs.has(R1.Reg));
  1565:   LatticeCell LS1;
  1566:   if (!getCell(R1, Inputs, LS1))
  1567:     return false;
  1568:   if (LS1.isProperty())
  1569:     return false;
  1570: 
  1571:   APInt A, XA;
  1572:   for (unsigned i = 0; i < LS1.size(); ++i) {
  1573:     bool Eval = constToInt(LS1.Values[i], A) &&
  1574:                 evaluateZEXTi(A, Width, Bits, XA);
  1575:     if (!Eval)
  1576:       return false;
  1577:     const Constant *C = intToConst(XA);
  1578:     Result.add(C);
  1579:   }
  1580:   return true;
  1581: }
  1582: 
  1583: bool MachineConstEvaluator::evaluateZEXTi(const APInt &A1, unsigned Width,
  1584:       unsigned Bits, APInt &Result) {
  1585:   unsigned BW = A1.getBitWidth();
  1586:   (void)BW;
  1587:   assert(Width >= Bits && BW >= Bits);
  1588:   APInt Mask = APInt::getLowBitsSet(Width, Bits);
  1589:   Result = A1.zextOrTrunc(Width) & Mask;
  1590:   return true;
  1591: }
  1592: 
  1593: bool MachineConstEvaluator::evaluateSEXTr(const RegSubRegPair &R1,
  1594:                                           unsigned Width, unsigned Bits,
  1595:                                           const CellMap &Inputs,
  1596:                                           LatticeCell &Result) {
  1597:   assert(Inputs.has(R1.Reg));
  1598:   LatticeCell LS1;
  1599:   if (!getCell(R1, Inputs, LS1))
  1600:     return false;
```
- EN: It declares or implements routines such as size, constToInt, intToConst, add, ... (21 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 size, constToInt, intToConst, add, ... (21 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 1601-1800 / 第 1601-1800 行

```cpp
  1601:   if (LS1.isBottom() || LS1.isProperty())
  1602:     return false;
  1603: 
  1604:   APInt A, XA;
  1605:   for (unsigned i = 0; i < LS1.size(); ++i) {
  1606:     bool Eval = constToInt(LS1.Values[i], A) &&
  1607:                 evaluateSEXTi(A, Width, Bits, XA);
  1608:     if (!Eval)
  1609:       return false;
  1610:     const Constant *C = intToConst(XA);
  1611:     Result.add(C);
  1612:   }
  1613:   return true;
  1614: }
  1615: 
  1616: bool MachineConstEvaluator::evaluateSEXTi(const APInt &A1, unsigned Width,
  1617:       unsigned Bits, APInt &Result) {
  1618:   unsigned BW = A1.getBitWidth();
  1619:   assert(Width >= Bits && BW >= Bits);
  1620:   // Special case to make things faster for smaller source widths.
  1621:   // Sign extension of 0 bits generates 0 as a result. This is consistent
  1622:   // with what the HW does.
  1623:   if (Bits == 0) {
  1624:     Result = APInt(Width, 0);
  1625:     return true;
  1626:   }
  1627:   // In C, shifts by 64 invoke undefined behavior: handle that case in APInt.
  1628:   if (BW <= 64 && Bits != 0) {
  1629:     int64_t V = A1.getSExtValue();
  1630:     switch (Bits) {
  1631:       case 8:
  1632:         V = static_cast<int8_t>(V);
  1633:         break;
  1634:       case 16:
  1635:         V = static_cast<int16_t>(V);
  1636:         break;
  1637:       case 32:
  1638:         V = static_cast<int32_t>(V);
  1639:         break;
  1640:       default:
  1641:         // Shift left to lose all bits except lower "Bits" bits, then shift
  1642:         // the value back, replicating what was a sign bit after the first
  1643:         // shift.
  1644:         V = (V << (64-Bits)) >> (64-Bits);
  1645:         break;
  1646:     }
  1647:     // V is a 64-bit sign-extended value. Convert it to APInt of desired
  1648:     // width.
  1649:     Result = APInt(Width, V, true);
  1650:     return true;
  1651:   }
  1652:   // Slow case: the value doesn't fit in int64_t.
  1653:   if (Bits < BW)
  1654:     Result = A1.trunc(Bits).sext(Width);
  1655:   else // Bits == BW
  1656:     Result = A1.sext(Width);
  1657:   return true;
  1658: }
  1659: 
  1660: bool MachineConstEvaluator::evaluateCLBr(const RegSubRegPair &R1, bool Zeros,
  1661:                                          bool Ones, const CellMap &Inputs,
  1662:                                          LatticeCell &Result) {
  1663:   assert(Inputs.has(R1.Reg));
  1664:   LatticeCell LS1;
  1665:   if (!getCell(R1, Inputs, LS1))
  1666:     return false;
  1667:   if (LS1.isBottom() || LS1.isProperty())
  1668:     return false;
  1669: 
  1670:   APInt A, CA;
  1671:   for (unsigned i = 0; i < LS1.size(); ++i) {
  1672:     bool Eval = constToInt(LS1.Values[i], A) &&
  1673:                 evaluateCLBi(A, Zeros, Ones, CA);
  1674:     if (!Eval)
  1675:       return false;
  1676:     const Constant *C = intToConst(CA);
  1677:     Result.add(C);
  1678:   }
  1679:   return true;
  1680: }
  1681: 
  1682: bool MachineConstEvaluator::evaluateCLBi(const APInt &A1, bool Zeros,
  1683:       bool Ones, APInt &Result) {
  1684:   unsigned BW = A1.getBitWidth();
  1685:   if (!Zeros && !Ones)
  1686:     return false;
  1687:   unsigned Count = 0;
  1688:   if (Zeros && (Count == 0))
  1689:     Count = A1.countl_zero();
  1690:   if (Ones && (Count == 0))
  1691:     Count = A1.countl_one();
  1692:   Result = APInt(BW, static_cast<uint64_t>(Count), false);
  1693:   return true;
  1694: }
  1695: 
  1696: bool MachineConstEvaluator::evaluateCTBr(const RegSubRegPair &R1, bool Zeros,
  1697:                                          bool Ones, const CellMap &Inputs,
  1698:                                          LatticeCell &Result) {
  1699:   assert(Inputs.has(R1.Reg));
  1700:   LatticeCell LS1;
  1701:   if (!getCell(R1, Inputs, LS1))
  1702:     return false;
  1703:   if (LS1.isBottom() || LS1.isProperty())
  1704:     return false;
  1705: 
  1706:   APInt A, CA;
  1707:   for (unsigned i = 0; i < LS1.size(); ++i) {
  1708:     bool Eval = constToInt(LS1.Values[i], A) &&
  1709:                 evaluateCTBi(A, Zeros, Ones, CA);
  1710:     if (!Eval)
  1711:       return false;
  1712:     const Constant *C = intToConst(CA);
  1713:     Result.add(C);
  1714:   }
  1715:   return true;
  1716: }
  1717: 
  1718: bool MachineConstEvaluator::evaluateCTBi(const APInt &A1, bool Zeros,
  1719:       bool Ones, APInt &Result) {
  1720:   unsigned BW = A1.getBitWidth();
  1721:   if (!Zeros && !Ones)
  1722:     return false;
  1723:   unsigned Count = 0;
  1724:   if (Zeros && (Count == 0))
  1725:     Count = A1.countr_zero();
  1726:   if (Ones && (Count == 0))
  1727:     Count = A1.countr_one();
  1728:   Result = APInt(BW, static_cast<uint64_t>(Count), false);
  1729:   return true;
  1730: }
  1731: 
  1732: bool MachineConstEvaluator::evaluateEXTRACTr(const RegSubRegPair &R1,
  1733:                                              unsigned Width, unsigned Bits,
  1734:                                              unsigned Offset, bool Signed,
  1735:                                              const CellMap &Inputs,
  1736:                                              LatticeCell &Result) {
  1737:   assert(Inputs.has(R1.Reg));
  1738:   assert(Bits+Offset <= Width);
  1739:   LatticeCell LS1;
  1740:   if (!getCell(R1, Inputs, LS1))
  1741:     return false;
  1742:   if (LS1.isBottom())
  1743:     return false;
  1744:   if (LS1.isProperty()) {
  1745:     uint32_t Ps = LS1.properties();
  1746:     if (Ps & ConstantProperties::Zero) {
  1747:       const Constant *C = intToConst(APInt(Width, 0, false));
  1748:       Result.add(C);
  1749:       return true;
  1750:     }
  1751:     return false;
  1752:   }
  1753: 
  1754:   APInt A, CA;
  1755:   for (unsigned i = 0; i < LS1.size(); ++i) {
  1756:     bool Eval = constToInt(LS1.Values[i], A) &&
  1757:                 evaluateEXTRACTi(A, Bits, Offset, Signed, CA);
  1758:     if (!Eval)
  1759:       return false;
  1760:     const Constant *C = intToConst(CA);
  1761:     Result.add(C);
  1762:   }
  1763:   return true;
  1764: }
  1765: 
  1766: bool MachineConstEvaluator::evaluateEXTRACTi(const APInt &A1, unsigned Bits,
  1767:       unsigned Offset, bool Signed, APInt &Result) {
  1768:   unsigned BW = A1.getBitWidth();
  1769:   assert(Bits+Offset <= BW);
  1770:   // Extracting 0 bits generates 0 as a result (as indicated by the HW people).
  1771:   if (Bits == 0) {
  1772:     Result = APInt(BW, 0);
  1773:     return true;
  1774:   }
  1775:   if (BW <= 64) {
  1776:     uint64_t U = A1.getZExtValue();
  1777:     U <<= (64 - Bits - Offset);
  1778:     int64_t V;
  1779:     if (Signed)
  1780:       V = static_cast<int64_t>(U) >> (64 - Bits);
  1781:     else
  1782:       V = static_cast<int64_t>(U >> (64 - Bits));
  1783:     Result = APInt(BW, V, Signed);
  1784:     return true;
  1785:   }
  1786:   if (Signed)
  1787:     Result = A1.shl(BW-Bits-Offset).ashr(BW-Bits);
  1788:   else
  1789:     Result = A1.shl(BW-Bits-Offset).lshr(BW-Bits);
  1790:   return true;
  1791: }
  1792: 
  1793: bool MachineConstEvaluator::evaluateSplatr(const RegSubRegPair &R1,
  1794:                                            unsigned Bits, unsigned Count,
  1795:                                            const CellMap &Inputs,
  1796:                                            LatticeCell &Result) {
  1797:   assert(Inputs.has(R1.Reg));
  1798:   LatticeCell LS1;
  1799:   if (!getCell(R1, Inputs, LS1))
  1800:     return false;
```
- EN: It declares or implements routines such as size, constToInt, intToConst, add, ... (25 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 size, constToInt, intToConst, add, ... (25 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 1801-2000 / 第 1801-2000 行

```cpp
  1801:   if (LS1.isBottom() || LS1.isProperty())
  1802:     return false;
  1803: 
  1804:   APInt A, SA;
  1805:   for (unsigned i = 0; i < LS1.size(); ++i) {
  1806:     bool Eval = constToInt(LS1.Values[i], A) &&
  1807:                 evaluateSplati(A, Bits, Count, SA);
  1808:     if (!Eval)
  1809:       return false;
  1810:     const Constant *C = intToConst(SA);
  1811:     Result.add(C);
  1812:   }
  1813:   return true;
  1814: }
  1815: 
  1816: bool MachineConstEvaluator::evaluateSplati(const APInt &A1, unsigned Bits,
  1817:       unsigned Count, APInt &Result) {
  1818:   assert(Count > 0);
  1819:   unsigned BW = A1.getBitWidth(), SW = Count*Bits;
  1820:   APInt LoBits = (Bits < BW) ? A1.trunc(Bits) : A1.zext(Bits);
  1821:   if (Count > 1)
  1822:     LoBits = LoBits.zext(SW);
  1823: 
  1824:   APInt Res(SW, 0, false);
  1825:   for (unsigned i = 0; i < Count; ++i) {
  1826:     Res <<= Bits;
  1827:     Res |= LoBits;
  1828:   }
  1829:   Result = Res;
  1830:   return true;
  1831: }
  1832: 
  1833: // ----------------------------------------------------------------------
  1834: // Hexagon-specific code.
  1835: 
  1836: namespace {
  1837: 
  1838:   class HexagonConstEvaluator : public MachineConstEvaluator {
  1839:   public:
  1840:     HexagonConstEvaluator(MachineFunction &Fn);
  1841: 
  1842:     bool evaluate(const MachineInstr &MI, const CellMap &Inputs,
  1843:           CellMap &Outputs) override;
  1844:     bool evaluate(const RegSubRegPair &R, const LatticeCell &SrcC,
  1845:                   LatticeCell &Result) override;
  1846:     bool evaluate(const MachineInstr &BrI, const CellMap &Inputs,
  1847:           SetVector<const MachineBasicBlock*> &Targets, bool &FallsThru)
  1848:           override;
  1849:     bool rewrite(MachineInstr &MI, const CellMap &Inputs) override;
  1850: 
  1851:   private:
  1852:     unsigned getRegBitWidth(unsigned Reg) const;
  1853: 
  1854:     static uint32_t getCmp(unsigned Opc);
  1855:     static APInt getCmpImm(unsigned Opc, unsigned OpX,
  1856:           const MachineOperand &MO);
  1857:     void replaceWithNop(MachineInstr &MI);
  1858: 
  1859:     bool evaluateHexRSEQ32(RegSubRegPair RL, RegSubRegPair RH,
  1860:                            const CellMap &Inputs, LatticeCell &Result);
  1861:     bool evaluateHexCompare(const MachineInstr &MI, const CellMap &Inputs,
  1862:           CellMap &Outputs);
  1863:     // This is suitable to be called for compare-and-jump instructions.
  1864:     bool evaluateHexCompare2(uint32_t Cmp, const MachineOperand &Src1,
  1865:           const MachineOperand &Src2, const CellMap &Inputs, bool &Result);
  1866:     bool evaluateHexLogical(const MachineInstr &MI, const CellMap &Inputs,
  1867:           CellMap &Outputs);
  1868:     bool evaluateHexCondMove(const MachineInstr &MI, const CellMap &Inputs,
  1869:           CellMap &Outputs);
  1870:     bool evaluateHexExt(const MachineInstr &MI, const CellMap &Inputs,
  1871:           CellMap &Outputs);
  1872:     bool evaluateHexVector1(const MachineInstr &MI, const CellMap &Inputs,
  1873:           CellMap &Outputs);
  1874:     bool evaluateHexVector2(const MachineInstr &MI, const CellMap &Inputs,
  1875:           CellMap &Outputs);
  1876: 
  1877:     void replaceAllRegUsesWith(Register FromReg, Register ToReg);
  1878:     bool rewriteHexBranch(MachineInstr &BrI, const CellMap &Inputs);
  1879:     bool rewriteHexConstDefs(MachineInstr &MI, const CellMap &Inputs,
  1880:           bool &AllDefs);
  1881:     bool rewriteHexConstUses(MachineInstr &MI, const CellMap &Inputs);
  1882: 
  1883:     MachineRegisterInfo *MRI;
  1884:     const HexagonInstrInfo &HII;
  1885:     const HexagonRegisterInfo &HRI;
  1886:   };
  1887: 
  1888:   class HexagonConstPropagation : public MachineFunctionPass {
  1889:   public:
  1890:     static char ID;
  1891: 
  1892:     HexagonConstPropagation() : MachineFunctionPass(ID) {}
  1893: 
  1894:     StringRef getPassName() const override {
  1895:       return "Hexagon Constant Propagation";
  1896:     }
  1897: 
  1898:     bool runOnMachineFunction(MachineFunction &MF) override {
  1899:       const Function &F = MF.getFunction();
  1900:       if (skipFunction(F))
  1901:         return false;
  1902: 
  1903:       HexagonConstEvaluator HCE(MF);
  1904:       return MachineConstPropagator(HCE).run(MF);
  1905:     }
  1906:   };
  1907: 
  1908: } // end anonymous namespace
  1909: 
  1910: char HexagonConstPropagation::ID = 0;
  1911: 
  1912: INITIALIZE_PASS(HexagonConstPropagation, "hexagon-constp",
  1913:   "Hexagon Constant Propagation", false, false)
  1914: 
  1915: HexagonConstEvaluator::HexagonConstEvaluator(MachineFunction &Fn)
  1916:   : MachineConstEvaluator(Fn),
  1917:     HII(*Fn.getSubtarget<HexagonSubtarget>().getInstrInfo()),
  1918:     HRI(*Fn.getSubtarget<HexagonSubtarget>().getRegisterInfo()) {
  1919:   MRI = &Fn.getRegInfo();
  1920: }
  1921: 
  1922: bool HexagonConstEvaluator::evaluate(const MachineInstr &MI,
  1923:       const CellMap &Inputs, CellMap &Outputs) {
  1924:   if (MI.isCall())
  1925:     return false;
  1926:   if (MI.getNumOperands() == 0 || !MI.getOperand(0).isReg())
  1927:     return false;
  1928:   const MachineOperand &MD = MI.getOperand(0);
  1929:   if (!MD.isDef())
  1930:     return false;
  1931: 
  1932:   unsigned Opc = MI.getOpcode();
  1933:   RegSubRegPair DefR(getRegSubRegPair(MD));
  1934:   assert(!DefR.SubReg);
  1935:   if (!DefR.Reg.isVirtual())
  1936:     return false;
  1937: 
  1938:   if (MI.isCopy()) {
  1939:     LatticeCell RC;
  1940:     RegSubRegPair SrcR(getRegSubRegPair(MI.getOperand(1)));
  1941:     bool Eval = evaluateCOPY(SrcR, Inputs, RC);
  1942:     if (!Eval)
  1943:       return false;
  1944:     Outputs.update(DefR.Reg, RC);
  1945:     return true;
  1946:   }
  1947:   if (MI.isRegSequence()) {
  1948:     unsigned Sub1 = MI.getOperand(2).getImm();
  1949:     unsigned Sub2 = MI.getOperand(4).getImm();
  1950:     const TargetRegisterClass &DefRC = *MRI->getRegClass(DefR.Reg);
  1951:     unsigned SubLo = HRI.getHexagonSubRegIndex(DefRC, Hexagon::ps_sub_lo);
  1952:     unsigned SubHi = HRI.getHexagonSubRegIndex(DefRC, Hexagon::ps_sub_hi);
  1953:     if (Sub1 != SubLo && Sub1 != SubHi)
  1954:       return false;
  1955:     if (Sub2 != SubLo && Sub2 != SubHi)
  1956:       return false;
  1957:     assert(Sub1 != Sub2);
  1958:     bool LoIs1 = (Sub1 == SubLo);
  1959:     const MachineOperand &OpLo = LoIs1 ? MI.getOperand(1) : MI.getOperand(3);
  1960:     const MachineOperand &OpHi = LoIs1 ? MI.getOperand(3) : MI.getOperand(1);
  1961:     LatticeCell RC;
  1962:     RegSubRegPair SrcRL(getRegSubRegPair(OpLo)), SrcRH(getRegSubRegPair(OpHi));
  1963:     bool Eval = evaluateHexRSEQ32(SrcRL, SrcRH, Inputs, RC);
  1964:     if (!Eval)
  1965:       return false;
  1966:     Outputs.update(DefR.Reg, RC);
  1967:     return true;
  1968:   }
  1969:   if (MI.isCompare()) {
  1970:     bool Eval = evaluateHexCompare(MI, Inputs, Outputs);
  1971:     return Eval;
  1972:   }
  1973: 
  1974:   switch (Opc) {
  1975:     default:
  1976:       return false;
  1977:     case Hexagon::A2_tfrsi:
  1978:     case Hexagon::A2_tfrpi:
  1979:     case Hexagon::CONST32:
  1980:     case Hexagon::CONST64:
  1981:     {
  1982:       const MachineOperand &VO = MI.getOperand(1);
  1983:       // The operand of CONST32 can be a blockaddress, e.g.
  1984:       //   %0 = CONST32 <blockaddress(@eat, %l)>
  1985:       // Do this check for all instructions for safety.
  1986:       if (!VO.isImm())
  1987:         return false;
  1988:       int64_t V = MI.getOperand(1).getImm();
  1989:       unsigned W = getRegBitWidth(DefR.Reg);
  1990:       if (W != 32 && W != 64)
  1991:         return false;
  1992:       IntegerType *Ty = (W == 32) ? Type::getInt32Ty(CX)
  1993:                                   : Type::getInt64Ty(CX);
  1994:       const ConstantInt *CI =
  1995:           ConstantInt::get(Ty, V, /*IsSigned=*/true, /*ImplicitTrunc=*/true);
  1996:       LatticeCell RC = Outputs.get(DefR.Reg);
  1997:       RC.add(CI);
  1998:       Outputs.update(DefR.Reg, RC);
  1999:       break;
  2000:     }
```
- EN: It opens namespaces (char) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonConstEvaluator, HexagonConstPropagation, which carry the state or API of this component. It defines declarative TableGen records like HexagonConstEvaluator, HexagonConstPropagation; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as size, constToInt, intToConst, add, ... (48 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里打开了命名空间（char），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonConstEvaluator, HexagonConstPropagation 等类型，用来承载该组件的状态或接口。 这里定义了 HexagonConstEvaluator, HexagonConstPropagation 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 size, constToInt, intToConst, add, ... (48 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 2001-2200 / 第 2001-2200 行

```cpp
  2001: 
  2002:     case Hexagon::PS_true:
  2003:     case Hexagon::PS_false:
  2004:     {
  2005:       LatticeCell RC = Outputs.get(DefR.Reg);
  2006:       bool NonZero = (Opc == Hexagon::PS_true);
  2007:       uint32_t P = NonZero ? ConstantProperties::NonZero
  2008:                            : ConstantProperties::Zero;
  2009:       RC.add(P);
  2010:       Outputs.update(DefR.Reg, RC);
  2011:       break;
  2012:     }
  2013: 
  2014:     case Hexagon::A2_and:
  2015:     case Hexagon::A2_andir:
  2016:     case Hexagon::A2_andp:
  2017:     case Hexagon::A2_or:
  2018:     case Hexagon::A2_orir:
  2019:     case Hexagon::A2_orp:
  2020:     case Hexagon::A2_xor:
  2021:     case Hexagon::A2_xorp:
  2022:     {
  2023:       bool Eval = evaluateHexLogical(MI, Inputs, Outputs);
  2024:       if (!Eval)
  2025:         return false;
  2026:       break;
  2027:     }
  2028: 
  2029:     case Hexagon::A2_combineii:  // combine(#s8Ext, #s8)
  2030:     case Hexagon::A4_combineii:  // combine(#s8, #u6Ext)
  2031:     {
  2032:       if (!MI.getOperand(1).isImm() || !MI.getOperand(2).isImm())
  2033:         return false;
  2034:       uint64_t Hi = MI.getOperand(1).getImm();
  2035:       uint64_t Lo = MI.getOperand(2).getImm();
  2036:       uint64_t Res = (Hi << 32) | (Lo & 0xFFFFFFFF);
  2037:       IntegerType *Ty = Type::getInt64Ty(CX);
  2038:       const ConstantInt *CI = ConstantInt::get(Ty, Res, false);
  2039:       LatticeCell RC = Outputs.get(DefR.Reg);
  2040:       RC.add(CI);
  2041:       Outputs.update(DefR.Reg, RC);
  2042:       break;
  2043:     }
  2044: 
  2045:     case Hexagon::S2_setbit_i:
  2046:     {
  2047:       int64_t B = MI.getOperand(2).getImm();
  2048:       assert(B >=0 && B < 32);
  2049:       APInt A(32, (1ull << B), false);
  2050:       RegSubRegPair R(getRegSubRegPair(MI.getOperand(1)));
  2051:       LatticeCell RC = Outputs.get(DefR.Reg);
  2052:       bool Eval = evaluateORri(R, A, Inputs, RC);
  2053:       if (!Eval)
  2054:         return false;
  2055:       Outputs.update(DefR.Reg, RC);
  2056:       break;
  2057:     }
  2058: 
  2059:     case Hexagon::C2_mux:
  2060:     case Hexagon::C2_muxir:
  2061:     case Hexagon::C2_muxri:
  2062:     case Hexagon::C2_muxii:
  2063:     {
  2064:       bool Eval = evaluateHexCondMove(MI, Inputs, Outputs);
  2065:       if (!Eval)
  2066:         return false;
  2067:       break;
  2068:     }
  2069: 
  2070:     case Hexagon::A2_sxtb:
  2071:     case Hexagon::A2_sxth:
  2072:     case Hexagon::A2_sxtw:
  2073:     case Hexagon::A2_zxtb:
  2074:     case Hexagon::A2_zxth:
  2075:     {
  2076:       bool Eval = evaluateHexExt(MI, Inputs, Outputs);
  2077:       if (!Eval)
  2078:         return false;
  2079:       break;
  2080:     }
  2081: 
  2082:     case Hexagon::S2_ct0:
  2083:     case Hexagon::S2_ct0p:
  2084:     case Hexagon::S2_ct1:
  2085:     case Hexagon::S2_ct1p:
  2086:     {
  2087:       using namespace Hexagon;
  2088: 
  2089:       bool Ones = (Opc == S2_ct1) || (Opc == S2_ct1p);
  2090:       RegSubRegPair R1(getRegSubRegPair(MI.getOperand(1)));
  2091:       assert(Inputs.has(R1.Reg));
  2092:       LatticeCell T;
  2093:       bool Eval = evaluateCTBr(R1, !Ones, Ones, Inputs, T);
  2094:       if (!Eval)
  2095:         return false;
  2096:       // All of these instructions return a 32-bit value. The evaluate
  2097:       // will generate the same type as the operand, so truncate the
  2098:       // result if necessary.
  2099:       APInt C;
  2100:       LatticeCell RC = Outputs.get(DefR.Reg);
  2101:       for (unsigned i = 0; i < T.size(); ++i) {
  2102:         const Constant *CI = T.Values[i];
  2103:         if (constToInt(CI, C) && C.getBitWidth() > 32)
  2104:           CI = intToConst(C.trunc(32));
  2105:         RC.add(CI);
  2106:       }
  2107:       Outputs.update(DefR.Reg, RC);
  2108:       break;
  2109:     }
  2110: 
  2111:     case Hexagon::S2_cl0:
  2112:     case Hexagon::S2_cl0p:
  2113:     case Hexagon::S2_cl1:
  2114:     case Hexagon::S2_cl1p:
  2115:     case Hexagon::S2_clb:
  2116:     case Hexagon::S2_clbp:
  2117:     {
  2118:       using namespace Hexagon;
  2119: 
  2120:       bool OnlyZeros = (Opc == S2_cl0) || (Opc == S2_cl0p);
  2121:       bool OnlyOnes =  (Opc == S2_cl1) || (Opc == S2_cl1p);
  2122:       RegSubRegPair R1(getRegSubRegPair(MI.getOperand(1)));
  2123:       assert(Inputs.has(R1.Reg));
  2124:       LatticeCell T;
  2125:       bool Eval = evaluateCLBr(R1, !OnlyOnes, !OnlyZeros, Inputs, T);
  2126:       if (!Eval)
  2127:         return false;
  2128:       // All of these instructions return a 32-bit value. The evaluate
  2129:       // will generate the same type as the operand, so truncate the
  2130:       // result if necessary.
  2131:       APInt C;
  2132:       LatticeCell RC = Outputs.get(DefR.Reg);
  2133:       for (unsigned i = 0; i < T.size(); ++i) {
  2134:         const Constant *CI = T.Values[i];
  2135:         if (constToInt(CI, C) && C.getBitWidth() > 32)
  2136:           CI = intToConst(C.trunc(32));
  2137:         RC.add(CI);
  2138:       }
  2139:       Outputs.update(DefR.Reg, RC);
  2140:       break;
  2141:     }
  2142: 
  2143:     case Hexagon::S4_extract:
  2144:     case Hexagon::S4_extractp:
  2145:     case Hexagon::S2_extractu:
  2146:     case Hexagon::S2_extractup:
  2147:     {
  2148:       bool Signed = (Opc == Hexagon::S4_extract) ||
  2149:                     (Opc == Hexagon::S4_extractp);
  2150:       RegSubRegPair R1(getRegSubRegPair(MI.getOperand(1)));
  2151:       unsigned BW = getRegBitWidth(R1.Reg);
  2152:       unsigned Bits = MI.getOperand(2).getImm();
  2153:       unsigned Offset = MI.getOperand(3).getImm();
  2154:       LatticeCell RC = Outputs.get(DefR.Reg);
  2155:       if (Offset >= BW) {
  2156:         APInt Zero(BW, 0, false);
  2157:         RC.add(intToConst(Zero));
  2158:         break;
  2159:       }
  2160:       if (Offset+Bits > BW) {
  2161:         // If the requested bitfield extends beyond the most significant bit,
  2162:         // the extra bits are treated as 0s. To emulate this behavior, reduce
  2163:         // the number of requested bits, and make the extract unsigned.
  2164:         Bits = BW-Offset;
  2165:         Signed = false;
  2166:       }
  2167:       bool Eval = evaluateEXTRACTr(R1, BW, Bits, Offset, Signed, Inputs, RC);
  2168:       if (!Eval)
  2169:         return false;
  2170:       Outputs.update(DefR.Reg, RC);
  2171:       break;
  2172:     }
  2173: 
  2174:     case Hexagon::S2_vsplatrb:
  2175:     case Hexagon::S2_vsplatrh:
  2176:     // vabsh, vabsh:sat
  2177:     // vabsw, vabsw:sat
  2178:     // vconj:sat
  2179:     // vrndwh, vrndwh:sat
  2180:     // vsathb, vsathub, vsatwuh
  2181:     // vsxtbh, vsxthw
  2182:     // vtrunehb, vtrunohb
  2183:     // vzxtbh, vzxthw
  2184:     {
  2185:       bool Eval = evaluateHexVector1(MI, Inputs, Outputs);
  2186:       if (!Eval)
  2187:         return false;
  2188:       break;
  2189:     }
  2190: 
  2191:     // TODO:
  2192:     // A2_vaddh
  2193:     // A2_vaddhs
  2194:     // A2_vaddw
  2195:     // A2_vaddws
  2196:   }
  2197: 
  2198:   return true;
  2199: }
  2200: 
```
- EN: It opens namespaces (Hexagon) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as get, add, update, evaluateHexLogical, ... (22 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里打开了命名空间（Hexagon），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 get, add, update, evaluateHexLogical, ... (22 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 2201-2400 / 第 2201-2400 行

```cpp
  2201: bool HexagonConstEvaluator::evaluate(const RegSubRegPair &R,
  2202:                                      const LatticeCell &Input,
  2203:                                      LatticeCell &Result) {
  2204:   if (!R.SubReg) {
  2205:     Result = Input;
  2206:     return true;
  2207:   }
  2208:   const TargetRegisterClass *RC = MRI->getRegClass(R.Reg);
  2209:   if (RC != &Hexagon::DoubleRegsRegClass)
  2210:     return false;
  2211:   if (R.SubReg != Hexagon::isub_lo && R.SubReg != Hexagon::isub_hi)
  2212:     return false;
  2213: 
  2214:   assert(!Input.isTop());
  2215:   if (Input.isBottom())
  2216:     return false;
  2217: 
  2218:   using P = ConstantProperties;
  2219: 
  2220:   if (Input.isProperty()) {
  2221:     uint32_t Ps = Input.properties();
  2222:     if (Ps & (P::Zero|P::NaN)) {
  2223:       uint32_t Ns = (Ps & (P::Zero|P::NaN|P::SignProperties));
  2224:       Result.add(Ns);
  2225:       return true;
  2226:     }
  2227:     if (R.SubReg == Hexagon::isub_hi) {
  2228:       uint32_t Ns = (Ps & P::SignProperties);
  2229:       Result.add(Ns);
  2230:       return true;
  2231:     }
  2232:     return false;
  2233:   }
  2234: 
  2235:   // The Input cell contains some known values. Pick the word corresponding
  2236:   // to the subregister.
  2237:   APInt A;
  2238:   for (unsigned i = 0; i < Input.size(); ++i) {
  2239:     const Constant *C = Input.Values[i];
  2240:     if (!constToInt(C, A))
  2241:       return false;
  2242:     if (!A.isIntN(64))
  2243:       return false;
  2244:     uint64_t U = A.getZExtValue();
  2245:     if (R.SubReg == Hexagon::isub_hi)
  2246:       U >>= 32;
  2247:     U &= 0xFFFFFFFFULL;
  2248:     uint32_t U32 = Lo_32(U);
  2249:     int32_t V32;
  2250:     memcpy(&V32, &U32, sizeof V32);
  2251:     IntegerType *Ty = Type::getInt32Ty(CX);
  2252:     const ConstantInt *C32 =
  2253:         ConstantInt::getSigned(Ty, static_cast<int64_t>(V32));
  2254:     Result.add(C32);
  2255:   }
  2256:   return true;
  2257: }
  2258: 
  2259: bool HexagonConstEvaluator::evaluate(const MachineInstr &BrI,
  2260:       const CellMap &Inputs, SetVector<const MachineBasicBlock*> &Targets,
  2261:       bool &FallsThru) {
  2262:   // We need to evaluate one branch at a time. TII::analyzeBranch checks
  2263:   // all the branches in a basic block at once, so we cannot use it.
  2264:   unsigned Opc = BrI.getOpcode();
  2265:   bool SimpleBranch = false;
  2266:   bool Negated = false;
  2267:   switch (Opc) {
  2268:     case Hexagon::J2_jumpf:
  2269:     case Hexagon::J2_jumpfnew:
  2270:     case Hexagon::J2_jumpfnewpt:
  2271:       Negated = true;
  2272:       [[fallthrough]];
  2273:     case Hexagon::J2_jumpt:
  2274:     case Hexagon::J2_jumptnew:
  2275:     case Hexagon::J2_jumptnewpt:
  2276:       // Simple branch:  if([!]Pn) jump ...
  2277:       // i.e. Op0 = predicate, Op1 = branch target.
  2278:       SimpleBranch = true;
  2279:       break;
  2280:     case Hexagon::J2_jump:
  2281:       Targets.insert(BrI.getOperand(0).getMBB());
  2282:       FallsThru = false;
  2283:       return true;
  2284:     default:
  2285: Undetermined:
  2286:       // If the branch is of unknown type, assume that all successors are
  2287:       // executable.
  2288:       FallsThru = !BrI.isUnconditionalBranch();
  2289:       return false;
  2290:   }
  2291: 
  2292:   if (SimpleBranch) {
  2293:     const MachineOperand &MD = BrI.getOperand(0);
  2294:     RegSubRegPair PR(getRegSubRegPair(MD));
  2295:     // If the condition operand has a subregister, this is not something
  2296:     // we currently recognize.
  2297:     if (PR.SubReg)
  2298:       goto Undetermined;
  2299:     assert(Inputs.has(PR.Reg));
  2300:     const LatticeCell &PredC = Inputs.get(PR.Reg);
  2301:     if (PredC.isBottom())
  2302:       goto Undetermined;
  2303: 
  2304:     uint32_t Props = PredC.properties();
  2305:     bool CTrue = false, CFalse = false;
  2306:     if (Props & ConstantProperties::Zero)
  2307:       CFalse = true;
  2308:     else if (Props & ConstantProperties::NonZero)
  2309:       CTrue = true;
  2310:     // If the condition is not known to be either, bail out.
  2311:     if (!CTrue && !CFalse)
  2312:       goto Undetermined;
  2313: 
  2314:     const MachineBasicBlock *BranchTarget = BrI.getOperand(1).getMBB();
  2315: 
  2316:     FallsThru = false;
  2317:     if ((!Negated && CTrue) || (Negated && CFalse))
  2318:       Targets.insert(BranchTarget);
  2319:     else if ((!Negated && CFalse) || (Negated && CTrue))
  2320:       FallsThru = true;
  2321:     else
  2322:       goto Undetermined;
  2323:   }
  2324: 
  2325:   return true;
  2326: }
  2327: 
  2328: bool HexagonConstEvaluator::rewrite(MachineInstr &MI, const CellMap &Inputs) {
  2329:   if (MI.isBranch())
  2330:     return rewriteHexBranch(MI, Inputs);
  2331: 
  2332:   unsigned Opc = MI.getOpcode();
  2333:   switch (Opc) {
  2334:     default:
  2335:       break;
  2336:     case Hexagon::A2_tfrsi:
  2337:     case Hexagon::A2_tfrpi:
  2338:     case Hexagon::CONST32:
  2339:     case Hexagon::CONST64:
  2340:     case Hexagon::PS_true:
  2341:     case Hexagon::PS_false:
  2342:       return false;
  2343:   }
  2344: 
  2345:   unsigned NumOp = MI.getNumOperands();
  2346:   if (NumOp == 0)
  2347:     return false;
  2348: 
  2349:   bool AllDefs, Changed;
  2350:   Changed = rewriteHexConstDefs(MI, Inputs, AllDefs);
  2351:   // If not all defs have been rewritten (i.e. the instruction defines
  2352:   // a register that is not compile-time constant), then try to rewrite
  2353:   // register operands that are known to be constant with immediates.
  2354:   if (!AllDefs)
  2355:     Changed |= rewriteHexConstUses(MI, Inputs);
  2356: 
  2357:   return Changed;
  2358: }
  2359: 
  2360: unsigned HexagonConstEvaluator::getRegBitWidth(unsigned Reg) const {
  2361:   const TargetRegisterClass *RC = MRI->getRegClass(Reg);
  2362:   if (Hexagon::IntRegsRegClass.hasSubClassEq(RC))
  2363:     return 32;
  2364:   if (Hexagon::DoubleRegsRegClass.hasSubClassEq(RC))
  2365:     return 64;
  2366:   if (Hexagon::PredRegsRegClass.hasSubClassEq(RC))
  2367:     return 8;
  2368:   llvm_unreachable("Invalid register");
  2369:   return 0;
  2370: }
  2371: 
  2372: uint32_t HexagonConstEvaluator::getCmp(unsigned Opc) {
  2373:   switch (Opc) {
  2374:     case Hexagon::C2_cmpeq:
  2375:     case Hexagon::C2_cmpeqp:
  2376:     case Hexagon::A4_cmpbeq:
  2377:     case Hexagon::A4_cmpheq:
  2378:     case Hexagon::A4_cmpbeqi:
  2379:     case Hexagon::A4_cmpheqi:
  2380:     case Hexagon::C2_cmpeqi:
  2381:     case Hexagon::J4_cmpeqn1_t_jumpnv_nt:
  2382:     case Hexagon::J4_cmpeqn1_t_jumpnv_t:
  2383:     case Hexagon::J4_cmpeqi_t_jumpnv_nt:
  2384:     case Hexagon::J4_cmpeqi_t_jumpnv_t:
  2385:     case Hexagon::J4_cmpeq_t_jumpnv_nt:
  2386:     case Hexagon::J4_cmpeq_t_jumpnv_t:
  2387:       return Comparison::EQ;
  2388: 
  2389:     case Hexagon::C4_cmpneq:
  2390:     case Hexagon::C4_cmpneqi:
  2391:     case Hexagon::J4_cmpeqn1_f_jumpnv_nt:
  2392:     case Hexagon::J4_cmpeqn1_f_jumpnv_t:
  2393:     case Hexagon::J4_cmpeqi_f_jumpnv_nt:
  2394:     case Hexagon::J4_cmpeqi_f_jumpnv_t:
  2395:     case Hexagon::J4_cmpeq_f_jumpnv_nt:
  2396:     case Hexagon::J4_cmpeq_f_jumpnv_t:
  2397:       return Comparison::NE;
  2398: 
  2399:     case Hexagon::C2_cmpgt:
  2400:     case Hexagon::C2_cmpgtp:
```
- EN: It declares or implements routines such as HexagonConstEvaluator::evaluate, getRegClass, assert, properties, ... (24 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonConstEvaluator, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonConstEvaluator::evaluate, getRegClass, assert, properties, ... (24 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonConstEvaluator，说明了它与同级后端组件的连接关系。

### Lines 2401-2600 / 第 2401-2600 行

```cpp
  2401:     case Hexagon::A4_cmpbgt:
  2402:     case Hexagon::A4_cmphgt:
  2403:     case Hexagon::A4_cmpbgti:
  2404:     case Hexagon::A4_cmphgti:
  2405:     case Hexagon::C2_cmpgti:
  2406:     case Hexagon::J4_cmpgtn1_t_jumpnv_nt:
  2407:     case Hexagon::J4_cmpgtn1_t_jumpnv_t:
  2408:     case Hexagon::J4_cmpgti_t_jumpnv_nt:
  2409:     case Hexagon::J4_cmpgti_t_jumpnv_t:
  2410:     case Hexagon::J4_cmpgt_t_jumpnv_nt:
  2411:     case Hexagon::J4_cmpgt_t_jumpnv_t:
  2412:       return Comparison::GTs;
  2413: 
  2414:     case Hexagon::C4_cmplte:
  2415:     case Hexagon::C4_cmpltei:
  2416:     case Hexagon::J4_cmpgtn1_f_jumpnv_nt:
  2417:     case Hexagon::J4_cmpgtn1_f_jumpnv_t:
  2418:     case Hexagon::J4_cmpgti_f_jumpnv_nt:
  2419:     case Hexagon::J4_cmpgti_f_jumpnv_t:
  2420:     case Hexagon::J4_cmpgt_f_jumpnv_nt:
  2421:     case Hexagon::J4_cmpgt_f_jumpnv_t:
  2422:       return Comparison::LEs;
  2423: 
  2424:     case Hexagon::C2_cmpgtu:
  2425:     case Hexagon::C2_cmpgtup:
  2426:     case Hexagon::A4_cmpbgtu:
  2427:     case Hexagon::A4_cmpbgtui:
  2428:     case Hexagon::A4_cmphgtu:
  2429:     case Hexagon::A4_cmphgtui:
  2430:     case Hexagon::C2_cmpgtui:
  2431:     case Hexagon::J4_cmpgtui_t_jumpnv_nt:
  2432:     case Hexagon::J4_cmpgtui_t_jumpnv_t:
  2433:     case Hexagon::J4_cmpgtu_t_jumpnv_nt:
  2434:     case Hexagon::J4_cmpgtu_t_jumpnv_t:
  2435:       return Comparison::GTu;
  2436: 
  2437:     case Hexagon::J4_cmpltu_f_jumpnv_nt:
  2438:     case Hexagon::J4_cmpltu_f_jumpnv_t:
  2439:       return Comparison::GEu;
  2440: 
  2441:     case Hexagon::J4_cmpltu_t_jumpnv_nt:
  2442:     case Hexagon::J4_cmpltu_t_jumpnv_t:
  2443:       return Comparison::LTu;
  2444: 
  2445:     case Hexagon::J4_cmplt_f_jumpnv_nt:
  2446:     case Hexagon::J4_cmplt_f_jumpnv_t:
  2447:       return Comparison::GEs;
  2448: 
  2449:     case Hexagon::C4_cmplteu:
  2450:     case Hexagon::C4_cmplteui:
  2451:     case Hexagon::J4_cmpgtui_f_jumpnv_nt:
  2452:     case Hexagon::J4_cmpgtui_f_jumpnv_t:
  2453:     case Hexagon::J4_cmpgtu_f_jumpnv_nt:
  2454:     case Hexagon::J4_cmpgtu_f_jumpnv_t:
  2455:       return Comparison::LEu;
  2456: 
  2457:     case Hexagon::J4_cmplt_t_jumpnv_nt:
  2458:     case Hexagon::J4_cmplt_t_jumpnv_t:
  2459:       return Comparison::LTs;
  2460: 
  2461:     default:
  2462:       break;
  2463:   }
  2464:   return Comparison::Unk;
  2465: }
  2466: 
  2467: APInt HexagonConstEvaluator::getCmpImm(unsigned Opc, unsigned OpX,
  2468:       const MachineOperand &MO) {
  2469:   bool Signed = false;
  2470:   switch (Opc) {
  2471:     case Hexagon::A4_cmpbgtui:   // u7
  2472:     case Hexagon::A4_cmphgtui:   // u7
  2473:       break;
  2474:     case Hexagon::A4_cmpheqi:    // s8
  2475:     case Hexagon::C4_cmpneqi:   // s8
  2476:       Signed = true;
  2477:       break;
  2478:     case Hexagon::A4_cmpbeqi:    // u8
  2479:       break;
  2480:     case Hexagon::C2_cmpgtui:      // u9
  2481:     case Hexagon::C4_cmplteui:  // u9
  2482:       break;
  2483:     case Hexagon::C2_cmpeqi:       // s10
  2484:     case Hexagon::C2_cmpgti:       // s10
  2485:     case Hexagon::C4_cmpltei:   // s10
  2486:       Signed = true;
  2487:       break;
  2488:     case Hexagon::J4_cmpeqi_f_jumpnv_nt:   // u5
  2489:     case Hexagon::J4_cmpeqi_f_jumpnv_t:    // u5
  2490:     case Hexagon::J4_cmpeqi_t_jumpnv_nt:   // u5
  2491:     case Hexagon::J4_cmpeqi_t_jumpnv_t:    // u5
  2492:     case Hexagon::J4_cmpgti_f_jumpnv_nt:   // u5
  2493:     case Hexagon::J4_cmpgti_f_jumpnv_t:    // u5
  2494:     case Hexagon::J4_cmpgti_t_jumpnv_nt:   // u5
  2495:     case Hexagon::J4_cmpgti_t_jumpnv_t:    // u5
  2496:     case Hexagon::J4_cmpgtui_f_jumpnv_nt:  // u5
  2497:     case Hexagon::J4_cmpgtui_f_jumpnv_t:   // u5
  2498:     case Hexagon::J4_cmpgtui_t_jumpnv_nt:  // u5
  2499:     case Hexagon::J4_cmpgtui_t_jumpnv_t:   // u5
  2500:       break;
  2501:     default:
  2502:       llvm_unreachable("Unhandled instruction");
  2503:       break;
  2504:   }
  2505: 
  2506:   uint64_t Val = MO.getImm();
  2507:   // TODO: Is implicitTrunc correct here?
  2508:   return APInt(32, Val, Signed, /*implicitTrunc=*/true);
  2509: }
  2510: 
  2511: void HexagonConstEvaluator::replaceWithNop(MachineInstr &MI) {
  2512:   MI.setDesc(HII.get(Hexagon::A2_nop));
  2513:   while (MI.getNumOperands() > 0)
  2514:     MI.removeOperand(0);
  2515: }
  2516: 
  2517: bool HexagonConstEvaluator::evaluateHexRSEQ32(RegSubRegPair RL,
  2518:                                               RegSubRegPair RH,
  2519:                                               const CellMap &Inputs,
  2520:                                               LatticeCell &Result) {
  2521:   assert(Inputs.has(RL.Reg) && Inputs.has(RH.Reg));
  2522:   LatticeCell LSL, LSH;
  2523:   if (!getCell(RL, Inputs, LSL) || !getCell(RH, Inputs, LSH))
  2524:     return false;
  2525:   if (LSL.isProperty() || LSH.isProperty())
  2526:     return false;
  2527: 
  2528:   unsigned LN = LSL.size(), HN = LSH.size();
  2529:   SmallVector<APInt,4> LoVs(LN), HiVs(HN);
  2530:   for (unsigned i = 0; i < LN; ++i) {
  2531:     bool Eval = constToInt(LSL.Values[i], LoVs[i]);
  2532:     if (!Eval)
  2533:       return false;
  2534:     assert(LoVs[i].getBitWidth() == 32);
  2535:   }
  2536:   for (unsigned i = 0; i < HN; ++i) {
  2537:     bool Eval = constToInt(LSH.Values[i], HiVs[i]);
  2538:     if (!Eval)
  2539:       return false;
  2540:     assert(HiVs[i].getBitWidth() == 32);
  2541:   }
  2542: 
  2543:   for (unsigned i = 0; i < HiVs.size(); ++i) {
  2544:     APInt HV = HiVs[i].zext(64) << 32;
  2545:     for (unsigned j = 0; j < LoVs.size(); ++j) {
  2546:       APInt LV = LoVs[j].zext(64);
  2547:       const Constant *C = intToConst(HV | LV);
  2548:       Result.add(C);
  2549:       if (Result.isBottom())
  2550:         return false;
  2551:     }
  2552:   }
  2553:   return !Result.isBottom();
  2554: }
  2555: 
  2556: bool HexagonConstEvaluator::evaluateHexCompare(const MachineInstr &MI,
  2557:       const CellMap &Inputs, CellMap &Outputs) {
  2558:   unsigned Opc = MI.getOpcode();
  2559:   bool Classic = false;
  2560:   switch (Opc) {
  2561:     case Hexagon::C2_cmpeq:
  2562:     case Hexagon::C2_cmpeqp:
  2563:     case Hexagon::C2_cmpgt:
  2564:     case Hexagon::C2_cmpgtp:
  2565:     case Hexagon::C2_cmpgtu:
  2566:     case Hexagon::C2_cmpgtup:
  2567:     case Hexagon::C2_cmpeqi:
  2568:     case Hexagon::C2_cmpgti:
  2569:     case Hexagon::C2_cmpgtui:
  2570:       // Classic compare:  Dst0 = CMP Src1, Src2
  2571:       Classic = true;
  2572:       break;
  2573:     default:
  2574:       // Not handling other compare instructions now.
  2575:       return false;
  2576:   }
  2577: 
  2578:   if (Classic) {
  2579:     const MachineOperand &Src1 = MI.getOperand(1);
  2580:     const MachineOperand &Src2 = MI.getOperand(2);
  2581: 
  2582:     bool Result;
  2583:     unsigned Opc = MI.getOpcode();
  2584:     bool Computed = evaluateHexCompare2(Opc, Src1, Src2, Inputs, Result);
  2585:     if (Computed) {
  2586:       // Only create a zero/non-zero cell. At this time there isn't really
  2587:       // much need for specific values.
  2588:       RegSubRegPair DefR(getRegSubRegPair(MI.getOperand(0)));
  2589:       LatticeCell L = Outputs.get(DefR.Reg);
  2590:       uint32_t P = Result ? ConstantProperties::NonZero
  2591:                           : ConstantProperties::Zero;
  2592:       L.add(P);
  2593:       Outputs.update(DefR.Reg, L);
  2594:       return true;
  2595:     }
  2596:   }
  2597: 
  2598:   return false;
  2599: }
  2600: 
```
- EN: It declares or implements routines such as HexagonConstEvaluator::getCmpImm, llvm_unreachable, getImm, APInt, ... (22 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonConstEvaluator, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonConstEvaluator::getCmpImm, llvm_unreachable, getImm, APInt, ... (22 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonConstEvaluator，说明了它与同级后端组件的连接关系。

### Lines 2601-2800 / 第 2601-2800 行

```cpp
  2601: bool HexagonConstEvaluator::evaluateHexCompare2(unsigned Opc,
  2602:       const MachineOperand &Src1, const MachineOperand &Src2,
  2603:       const CellMap &Inputs, bool &Result) {
  2604:   uint32_t Cmp = getCmp(Opc);
  2605:   bool Reg1 = Src1.isReg(), Reg2 = Src2.isReg();
  2606:   bool Imm1 = Src1.isImm(), Imm2 = Src2.isImm();
  2607:   if (Reg1) {
  2608:     RegSubRegPair R1(getRegSubRegPair(Src1));
  2609:     if (Reg2) {
  2610:       RegSubRegPair R2(getRegSubRegPair(Src2));
  2611:       return evaluateCMPrr(Cmp, R1, R2, Inputs, Result);
  2612:     } else if (Imm2) {
  2613:       APInt A2 = getCmpImm(Opc, 2, Src2);
  2614:       return evaluateCMPri(Cmp, R1, A2, Inputs, Result);
  2615:     }
  2616:   } else if (Imm1) {
  2617:     APInt A1 = getCmpImm(Opc, 1, Src1);
  2618:     if (Reg2) {
  2619:       RegSubRegPair R2(getRegSubRegPair(Src2));
  2620:       uint32_t NegCmp = Comparison::negate(Cmp);
  2621:       return evaluateCMPri(NegCmp, R2, A1, Inputs, Result);
  2622:     } else if (Imm2) {
  2623:       APInt A2 = getCmpImm(Opc, 2, Src2);
  2624:       return evaluateCMPii(Cmp, A1, A2, Result);
  2625:     }
  2626:   }
  2627:   // Unknown kind of comparison.
  2628:   return false;
  2629: }
  2630: 
  2631: bool HexagonConstEvaluator::evaluateHexLogical(const MachineInstr &MI,
  2632:       const CellMap &Inputs, CellMap &Outputs) {
  2633:   unsigned Opc = MI.getOpcode();
  2634:   if (MI.getNumOperands() != 3)
  2635:     return false;
  2636:   const MachineOperand &Src1 = MI.getOperand(1);
  2637:   const MachineOperand &Src2 = MI.getOperand(2);
  2638:   RegSubRegPair R1(getRegSubRegPair(Src1));
  2639:   bool Eval = false;
  2640:   LatticeCell RC;
  2641:   switch (Opc) {
  2642:     default:
  2643:       return false;
  2644:     case Hexagon::A2_and:
  2645:     case Hexagon::A2_andp:
  2646:       Eval =
  2647:           evaluateANDrr(R1, RegSubRegPair(getRegSubRegPair(Src2)), Inputs, RC);
  2648:       break;
  2649:     case Hexagon::A2_andir: {
  2650:       if (!Src2.isImm())
  2651:         return false;
  2652:       APInt A(32, Src2.getImm(), true);
  2653:       Eval = evaluateANDri(R1, A, Inputs, RC);
  2654:       break;
  2655:     }
  2656:     case Hexagon::A2_or:
  2657:     case Hexagon::A2_orp:
  2658:       Eval =
  2659:           evaluateORrr(R1, RegSubRegPair(getRegSubRegPair(Src2)), Inputs, RC);
  2660:       break;
  2661:     case Hexagon::A2_orir: {
  2662:       if (!Src2.isImm())
  2663:         return false;
  2664:       APInt A(32, Src2.getImm(), true);
  2665:       Eval = evaluateORri(R1, A, Inputs, RC);
  2666:       break;
  2667:     }
  2668:     case Hexagon::A2_xor:
  2669:     case Hexagon::A2_xorp:
  2670:       Eval =
  2671:           evaluateXORrr(R1, RegSubRegPair(getRegSubRegPair(Src2)), Inputs, RC);
  2672:       break;
  2673:   }
  2674:   if (Eval) {
  2675:     RegSubRegPair DefR(getRegSubRegPair(MI.getOperand(0)));
  2676:     Outputs.update(DefR.Reg, RC);
  2677:   }
  2678:   return Eval;
  2679: }
  2680: 
  2681: bool HexagonConstEvaluator::evaluateHexCondMove(const MachineInstr &MI,
  2682:       const CellMap &Inputs, CellMap &Outputs) {
  2683:   // Dst0 = Cond1 ? Src2 : Src3
  2684:   RegSubRegPair CR(getRegSubRegPair(MI.getOperand(1)));
  2685:   assert(Inputs.has(CR.Reg));
  2686:   LatticeCell LS;
  2687:   if (!getCell(CR, Inputs, LS))
  2688:     return false;
  2689:   uint32_t Ps = LS.properties();
  2690:   unsigned TakeOp;
  2691:   if (Ps & ConstantProperties::Zero)
  2692:     TakeOp = 3;
  2693:   else if (Ps & ConstantProperties::NonZero)
  2694:     TakeOp = 2;
  2695:   else
  2696:     return false;
  2697: 
  2698:   const MachineOperand &ValOp = MI.getOperand(TakeOp);
  2699:   RegSubRegPair DefR(getRegSubRegPair(MI.getOperand(0)));
  2700:   LatticeCell RC = Outputs.get(DefR.Reg);
  2701: 
  2702:   if (ValOp.isImm()) {
  2703:     int64_t V = ValOp.getImm();
  2704:     unsigned W = getRegBitWidth(DefR.Reg);
  2705:     APInt A(W, V, true);
  2706:     const Constant *C = intToConst(A);
  2707:     RC.add(C);
  2708:     Outputs.update(DefR.Reg, RC);
  2709:     return true;
  2710:   }
  2711:   if (ValOp.isReg()) {
  2712:     RegSubRegPair R(getRegSubRegPair(ValOp));
  2713:     const LatticeCell &LR = Inputs.get(R.Reg);
  2714:     LatticeCell LSR;
  2715:     if (!evaluate(R, LR, LSR))
  2716:       return false;
  2717:     RC.meet(LSR);
  2718:     Outputs.update(DefR.Reg, RC);
  2719:     return true;
  2720:   }
  2721:   return false;
  2722: }
  2723: 
  2724: bool HexagonConstEvaluator::evaluateHexExt(const MachineInstr &MI,
  2725:       const CellMap &Inputs, CellMap &Outputs) {
  2726:   // Dst0 = ext R1
  2727:   RegSubRegPair R1(getRegSubRegPair(MI.getOperand(1)));
  2728:   assert(Inputs.has(R1.Reg));
  2729: 
  2730:   unsigned Opc = MI.getOpcode();
  2731:   unsigned Bits;
  2732:   switch (Opc) {
  2733:     case Hexagon::A2_sxtb:
  2734:     case Hexagon::A2_zxtb:
  2735:       Bits = 8;
  2736:       break;
  2737:     case Hexagon::A2_sxth:
  2738:     case Hexagon::A2_zxth:
  2739:       Bits = 16;
  2740:       break;
  2741:     case Hexagon::A2_sxtw:
  2742:       Bits = 32;
  2743:       break;
  2744:     default:
  2745:       llvm_unreachable("Unhandled extension opcode");
  2746:   }
  2747: 
  2748:   bool Signed = false;
  2749:   switch (Opc) {
  2750:     case Hexagon::A2_sxtb:
  2751:     case Hexagon::A2_sxth:
  2752:     case Hexagon::A2_sxtw:
  2753:       Signed = true;
  2754:       break;
  2755:   }
  2756: 
  2757:   RegSubRegPair DefR(getRegSubRegPair(MI.getOperand(0)));
  2758:   unsigned BW = getRegBitWidth(DefR.Reg);
  2759:   LatticeCell RC = Outputs.get(DefR.Reg);
  2760:   bool Eval = Signed ? evaluateSEXTr(R1, BW, Bits, Inputs, RC)
  2761:                      : evaluateZEXTr(R1, BW, Bits, Inputs, RC);
  2762:   if (!Eval)
  2763:     return false;
  2764:   Outputs.update(DefR.Reg, RC);
  2765:   return true;
  2766: }
  2767: 
  2768: bool HexagonConstEvaluator::evaluateHexVector1(const MachineInstr &MI,
  2769:       const CellMap &Inputs, CellMap &Outputs) {
  2770:   // DefR = op R1
  2771:   RegSubRegPair DefR(getRegSubRegPair(MI.getOperand(0)));
  2772:   RegSubRegPair R1(getRegSubRegPair(MI.getOperand(1)));
  2773:   assert(Inputs.has(R1.Reg));
  2774:   LatticeCell RC = Outputs.get(DefR.Reg);
  2775:   bool Eval;
  2776: 
  2777:   unsigned Opc = MI.getOpcode();
  2778:   switch (Opc) {
  2779:     case Hexagon::S2_vsplatrb:
  2780:       // Rd = 4 times Rs:0..7
  2781:       Eval = evaluateSplatr(R1, 8, 4, Inputs, RC);
  2782:       break;
  2783:     case Hexagon::S2_vsplatrh:
  2784:       // Rdd = 4 times Rs:0..15
  2785:       Eval = evaluateSplatr(R1, 16, 4, Inputs, RC);
  2786:       break;
  2787:     default:
  2788:       return false;
  2789:   }
  2790: 
  2791:   if (!Eval)
  2792:     return false;
  2793:   Outputs.update(DefR.Reg, RC);
  2794:   return true;
  2795: }
  2796: 
  2797: bool HexagonConstEvaluator::rewriteHexConstDefs(MachineInstr &MI,
  2798:       const CellMap &Inputs, bool &AllDefs) {
  2799:   AllDefs = false;
  2800: 
```
- EN: It declares or implements routines such as HexagonConstEvaluator::evaluateHexCompare2, getCmp, isReg, isImm, ... (39 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonConstEvaluator, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonConstEvaluator::evaluateHexCompare2, getCmp, isReg, isImm, ... (39 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonConstEvaluator，说明了它与同级后端组件的连接关系。

### Lines 2801-3000 / 第 2801-3000 行

```cpp
  2801:   // Some diagnostics.
  2802:   // LLVM_DEBUG({...}) gets confused with all this code as an argument.
  2803: #ifndef NDEBUG
  2804:   bool Debugging = DebugFlag && isCurrentDebugType(DEBUG_TYPE);
  2805:   if (Debugging) {
  2806:     bool Const = true, HasUse = false;
  2807:     for (const MachineOperand &MO : MI.operands()) {
  2808:       if (!MO.isReg() || !MO.isUse() || MO.isImplicit())
  2809:         continue;
  2810:       RegSubRegPair R(getRegSubRegPair(MO));
  2811:       if (!R.Reg.isVirtual())
  2812:         continue;
  2813:       HasUse = true;
  2814:       // PHIs can legitimately have "top" cells after propagation.
  2815:       if (!MI.isPHI() && !Inputs.has(R.Reg)) {
  2816:         dbgs() << "Top " << printReg(R.Reg, &HRI, R.SubReg)
  2817:                << " in MI: " << MI;
  2818:         continue;
  2819:       }
  2820:       const LatticeCell &L = Inputs.get(R.Reg);
  2821:       Const &= L.isSingle();
  2822:       if (!Const)
  2823:         break;
  2824:     }
  2825:     if (HasUse && Const) {
  2826:       if (!MI.isCopy()) {
  2827:         dbgs() << "CONST: " << MI;
  2828:         for (const MachineOperand &MO : MI.operands()) {
  2829:           if (!MO.isReg() || !MO.isUse() || MO.isImplicit())
  2830:             continue;
  2831:           Register R = MO.getReg();
  2832:           dbgs() << printReg(R, &TRI) << ": " << Inputs.get(R) << "\n";
  2833:         }
  2834:       }
  2835:     }
  2836:   }
  2837: #endif
  2838: 
  2839:   // Avoid generating TFRIs for register transfers---this will keep the
  2840:   // coalescing opportunities.
  2841:   if (MI.isCopy())
  2842:     return false;
  2843: 
  2844:   MachineFunction *MF = MI.getParent()->getParent();
  2845:   auto &HST = MF->getSubtarget<HexagonSubtarget>();
  2846: 
  2847:   // Collect all virtual register-def operands.
  2848:   SmallVector<unsigned,2> DefRegs;
  2849:   for (const MachineOperand &MO : MI.operands()) {
  2850:     if (!MO.isReg() || !MO.isDef())
  2851:       continue;
  2852:     Register R = MO.getReg();
  2853:     if (!R.isVirtual())
  2854:       continue;
  2855:     assert(!MO.getSubReg());
  2856:     assert(Inputs.has(R));
  2857:     DefRegs.push_back(R);
  2858:   }
  2859: 
  2860:   MachineBasicBlock &B = *MI.getParent();
  2861:   const DebugLoc &DL = MI.getDebugLoc();
  2862:   unsigned ChangedNum = 0;
  2863: #ifndef NDEBUG
  2864:   SmallVector<const MachineInstr*,4> NewInstrs;
  2865: #endif
  2866: 
  2867:   // For each defined register, if it is a constant, create an instruction
  2868:   //   NewR = const
  2869:   // and replace all uses of the defined register with NewR.
  2870:   for (unsigned R : DefRegs) {
  2871:     const LatticeCell &L = Inputs.get(R);
  2872:     if (L.isBottom())
  2873:       continue;
  2874:     const TargetRegisterClass *RC = MRI->getRegClass(R);
  2875:     MachineBasicBlock::iterator At = MI.getIterator();
  2876: 
  2877:     if (!L.isSingle()) {
  2878:       // If this a zero/non-zero cell, we can fold a definition
  2879:       // of a predicate register.
  2880:       using P = ConstantProperties;
  2881: 
  2882:       uint64_t Ps = L.properties();
  2883:       if (!(Ps & (P::Zero|P::NonZero)))
  2884:         continue;
  2885:       const TargetRegisterClass *PredRC = &Hexagon::PredRegsRegClass;
  2886:       if (RC != PredRC)
  2887:         continue;
  2888:       const MCInstrDesc *NewD = (Ps & P::Zero) ?
  2889:         &HII.get(Hexagon::PS_false) :
  2890:         &HII.get(Hexagon::PS_true);
  2891:       Register NewR = MRI->createVirtualRegister(PredRC);
  2892:       const MachineInstrBuilder &MIB = BuildMI(B, At, DL, *NewD, NewR);
  2893:       (void)MIB;
  2894: #ifndef NDEBUG
  2895:       NewInstrs.push_back(&*MIB);
  2896: #endif
  2897:       replaceAllRegUsesWith(R, NewR);
  2898:     } else {
  2899:       // This cell has a single value.
  2900:       APInt A;
  2901:       if (!constToInt(L.Value, A) || !A.isSignedIntN(64))
  2902:         continue;
  2903:       const TargetRegisterClass *NewRC;
  2904:       const MCInstrDesc *NewD;
  2905: 
  2906:       unsigned W = getRegBitWidth(R);
  2907:       int64_t V = A.getSExtValue();
  2908:       assert(W == 32 || W == 64);
  2909:       if (W == 32)
  2910:         NewRC = &Hexagon::IntRegsRegClass;
  2911:       else
  2912:         NewRC = &Hexagon::DoubleRegsRegClass;
  2913:       Register NewR = MRI->createVirtualRegister(NewRC);
  2914:       const MachineInstr *NewMI;
  2915: 
  2916:       if (W == 32) {
  2917:         NewD = &HII.get(Hexagon::A2_tfrsi);
  2918:         NewMI = BuildMI(B, At, DL, *NewD, NewR)
  2919:                   .addImm(V);
  2920:       } else {
  2921:         if (A.isSignedIntN(8)) {
  2922:           NewD = &HII.get(Hexagon::A2_tfrpi);
  2923:           NewMI = BuildMI(B, At, DL, *NewD, NewR)
  2924:                     .addImm(V);
  2925:         } else {
  2926:           int32_t Hi = V >> 32;
  2927:           int32_t Lo = V & 0xFFFFFFFFLL;
  2928:           if (isInt<8>(Hi) && isInt<8>(Lo)) {
  2929:             NewD = &HII.get(Hexagon::A2_combineii);
  2930:             NewMI = BuildMI(B, At, DL, *NewD, NewR)
  2931:                       .addImm(Hi)
  2932:                       .addImm(Lo);
  2933:           } else if (MF->getFunction().hasOptSize() || !HST.isTinyCore()) {
  2934:             // Disable CONST64 for tiny core since it takes a LD resource.
  2935:             NewD = &HII.get(Hexagon::CONST64);
  2936:             NewMI = BuildMI(B, At, DL, *NewD, NewR)
  2937:                       .addImm(V);
  2938:           } else
  2939:             return false;
  2940:         }
  2941:       }
  2942:       (void)NewMI;
  2943: #ifndef NDEBUG
  2944:       NewInstrs.push_back(NewMI);
  2945: #endif
  2946:       replaceAllRegUsesWith(R, NewR);
  2947:     }
  2948:     ChangedNum++;
  2949:   }
  2950: 
  2951:   LLVM_DEBUG({
  2952:     if (!NewInstrs.empty()) {
  2953:       MachineFunction &MF = *MI.getParent()->getParent();
  2954:       dbgs() << "In function: " << MF.getName() << "\n";
  2955:       dbgs() << "Rewrite: for " << MI << "  created " << *NewInstrs[0];
  2956:       for (unsigned i = 1; i < NewInstrs.size(); ++i)
  2957:         dbgs() << "          " << *NewInstrs[i];
  2958:     }
  2959:   });
  2960: 
  2961:   AllDefs = (ChangedNum == DefRegs.size());
  2962:   return ChangedNum > 0;
  2963: }
  2964: 
  2965: bool HexagonConstEvaluator::rewriteHexConstUses(MachineInstr &MI,
  2966:       const CellMap &Inputs) {
  2967:   bool Changed = false;
  2968:   unsigned Opc = MI.getOpcode();
  2969:   MachineBasicBlock &B = *MI.getParent();
  2970:   const DebugLoc &DL = MI.getDebugLoc();
  2971:   MachineBasicBlock::iterator At = MI.getIterator();
  2972:   MachineInstr *NewMI = nullptr;
  2973: 
  2974:   switch (Opc) {
  2975:     case Hexagon::M2_maci:
  2976:     // Convert DefR += mpyi(R2, R3)
  2977:     //   to   DefR += mpyi(R, #imm),
  2978:     //   or   DefR -= mpyi(R, #imm).
  2979:     {
  2980:       RegSubRegPair DefR(getRegSubRegPair(MI.getOperand(0)));
  2981:       assert(!DefR.SubReg);
  2982:       RegSubRegPair R2(getRegSubRegPair(MI.getOperand(2)));
  2983:       RegSubRegPair R3(getRegSubRegPair(MI.getOperand(3)));
  2984:       assert(Inputs.has(R2.Reg) && Inputs.has(R3.Reg));
  2985:       LatticeCell LS2, LS3;
  2986:       // It is enough to get one of the input cells, since we will only try
  2987:       // to replace one argument---whichever happens to be a single constant.
  2988:       bool HasC2 = getCell(R2, Inputs, LS2), HasC3 = getCell(R3, Inputs, LS3);
  2989:       if (!HasC2 && !HasC3)
  2990:         return false;
  2991:       bool Zero = ((HasC2 && (LS2.properties() & ConstantProperties::Zero)) ||
  2992:                    (HasC3 && (LS3.properties() & ConstantProperties::Zero)));
  2993:       // If one of the operands is zero, eliminate the multiplication.
  2994:       if (Zero) {
  2995:         // DefR == R1 (tied operands).
  2996:         MachineOperand &Acc = MI.getOperand(1);
  2997:         RegSubRegPair R1(getRegSubRegPair(Acc));
  2998:         unsigned NewR = R1.Reg;
  2999:         if (R1.SubReg) {
  3000:           // Generate COPY. FIXME: Replace with the register:subregister.
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. It declares or implements routines such as isCurrentDebugType, R, get, isSingle, ... (26 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里声明或实现了 isCurrentDebugType, R, get, isSingle, ... (26 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 3001-3193 / 第 3001-3193 行

```cpp
  3001:           const TargetRegisterClass *RC = MRI->getRegClass(DefR.Reg);
  3002:           NewR = MRI->createVirtualRegister(RC);
  3003:           NewMI = BuildMI(B, At, DL, HII.get(TargetOpcode::COPY), NewR)
  3004:                     .addReg(R1.Reg, getRegState(Acc), R1.SubReg);
  3005:         }
  3006:         replaceAllRegUsesWith(DefR.Reg, NewR);
  3007:         MRI->clearKillFlags(NewR);
  3008:         Changed = true;
  3009:         break;
  3010:       }
  3011: 
  3012:       bool Swap = false;
  3013:       if (!LS3.isSingle()) {
  3014:         if (!LS2.isSingle())
  3015:           return false;
  3016:         Swap = true;
  3017:       }
  3018:       const LatticeCell &LI = Swap ? LS2 : LS3;
  3019:       const MachineOperand &OpR2 = Swap ? MI.getOperand(3)
  3020:                                         : MI.getOperand(2);
  3021:       // LI is single here.
  3022:       APInt A;
  3023:       if (!constToInt(LI.Value, A) || !A.isSignedIntN(8))
  3024:         return false;
  3025:       int64_t V = A.getSExtValue();
  3026:       const MCInstrDesc &D = (V >= 0) ? HII.get(Hexagon::M2_macsip)
  3027:                                       : HII.get(Hexagon::M2_macsin);
  3028:       if (V < 0)
  3029:         V = -V;
  3030:       const TargetRegisterClass *RC = MRI->getRegClass(DefR.Reg);
  3031:       Register NewR = MRI->createVirtualRegister(RC);
  3032:       const MachineOperand &Src1 = MI.getOperand(1);
  3033:       NewMI = BuildMI(B, At, DL, D, NewR)
  3034:                 .addReg(Src1.getReg(), getRegState(Src1), Src1.getSubReg())
  3035:                 .addReg(OpR2.getReg(), getRegState(OpR2), OpR2.getSubReg())
  3036:                 .addImm(V);
  3037:       replaceAllRegUsesWith(DefR.Reg, NewR);
  3038:       Changed = true;
  3039:       break;
  3040:     }
  3041: 
  3042:     case Hexagon::A2_and:
  3043:     {
  3044:       RegSubRegPair R1(getRegSubRegPair(MI.getOperand(1)));
  3045:       RegSubRegPair R2(getRegSubRegPair(MI.getOperand(2)));
  3046:       assert(Inputs.has(R1.Reg) && Inputs.has(R2.Reg));
  3047:       LatticeCell LS1, LS2;
  3048:       unsigned CopyOf = 0;
  3049:       // Check if any of the operands is -1 (i.e. all bits set).
  3050:       if (getCell(R1, Inputs, LS1) && LS1.isSingle()) {
  3051:         APInt M1;
  3052:         if (constToInt(LS1.Value, M1) && !~M1)
  3053:           CopyOf = 2;
  3054:       }
  3055:       else if (getCell(R2, Inputs, LS2) && LS2.isSingle()) {
  3056:         APInt M1;
  3057:         if (constToInt(LS2.Value, M1) && !~M1)
  3058:           CopyOf = 1;
  3059:       }
  3060:       if (!CopyOf)
  3061:         return false;
  3062:       MachineOperand &SO = MI.getOperand(CopyOf);
  3063:       RegSubRegPair SR(getRegSubRegPair(SO));
  3064:       RegSubRegPair DefR(getRegSubRegPair(MI.getOperand(0)));
  3065:       unsigned NewR = SR.Reg;
  3066:       if (SR.SubReg) {
  3067:         const TargetRegisterClass *RC = MRI->getRegClass(DefR.Reg);
  3068:         NewR = MRI->createVirtualRegister(RC);
  3069:         NewMI = BuildMI(B, At, DL, HII.get(TargetOpcode::COPY), NewR)
  3070:                   .addReg(SR.Reg, getRegState(SO), SR.SubReg);
  3071:       }
  3072:       replaceAllRegUsesWith(DefR.Reg, NewR);
  3073:       MRI->clearKillFlags(NewR);
  3074:       Changed = true;
  3075:     }
  3076:     break;
  3077: 
  3078:     case Hexagon::A2_or:
  3079:     {
  3080:       RegSubRegPair R1(getRegSubRegPair(MI.getOperand(1)));
  3081:       RegSubRegPair R2(getRegSubRegPair(MI.getOperand(2)));
  3082:       assert(Inputs.has(R1.Reg) && Inputs.has(R2.Reg));
  3083:       LatticeCell LS1, LS2;
  3084:       unsigned CopyOf = 0;
  3085: 
  3086:       using P = ConstantProperties;
  3087: 
  3088:       if (getCell(R1, Inputs, LS1) && (LS1.properties() & P::Zero))
  3089:         CopyOf = 2;
  3090:       else if (getCell(R2, Inputs, LS2) && (LS2.properties() & P::Zero))
  3091:         CopyOf = 1;
  3092:       if (!CopyOf)
  3093:         return false;
  3094:       MachineOperand &SO = MI.getOperand(CopyOf);
  3095:       RegSubRegPair SR(getRegSubRegPair(SO));
  3096:       RegSubRegPair DefR(getRegSubRegPair(MI.getOperand(0)));
  3097:       unsigned NewR = SR.Reg;
  3098:       if (SR.SubReg) {
  3099:         const TargetRegisterClass *RC = MRI->getRegClass(DefR.Reg);
  3100:         NewR = MRI->createVirtualRegister(RC);
  3101:         NewMI = BuildMI(B, At, DL, HII.get(TargetOpcode::COPY), NewR)
  3102:                   .addReg(SR.Reg, getRegState(SO), SR.SubReg);
  3103:       }
  3104:       replaceAllRegUsesWith(DefR.Reg, NewR);
  3105:       MRI->clearKillFlags(NewR);
  3106:       Changed = true;
  3107:     }
  3108:     break;
  3109:   }
  3110: 
  3111:   if (NewMI) {
  3112:     // clear all the kill flags of this new instruction.
  3113:     for (MachineOperand &MO : NewMI->operands())
  3114:       if (MO.isReg() && MO.isUse())
  3115:         MO.setIsKill(false);
  3116:   }
  3117: 
  3118:   LLVM_DEBUG({
  3119:     if (NewMI) {
  3120:       dbgs() << "Rewrite: for " << MI;
  3121:       if (NewMI != &MI)
  3122:         dbgs() << "  created " << *NewMI;
  3123:       else
  3124:         dbgs() << "  modified the instruction itself and created:" << *NewMI;
  3125:     }
  3126:   });
  3127: 
  3128:   return Changed;
  3129: }
  3130: 
  3131: void HexagonConstEvaluator::replaceAllRegUsesWith(Register FromReg,
  3132:                                                   Register ToReg) {
  3133:   assert(FromReg.isVirtual());
  3134:   assert(ToReg.isVirtual());
  3135:   for (MachineOperand &O :
  3136:        llvm::make_early_inc_range(MRI->use_operands(FromReg)))
  3137:     O.setReg(ToReg);
  3138: }
  3139: 
  3140: bool HexagonConstEvaluator::rewriteHexBranch(MachineInstr &BrI,
  3141:       const CellMap &Inputs) {
  3142:   MachineBasicBlock &B = *BrI.getParent();
  3143:   unsigned NumOp = BrI.getNumOperands();
  3144:   if (!NumOp)
  3145:     return false;
  3146: 
  3147:   bool FallsThru;
  3148:   SetVector<const MachineBasicBlock*> Targets;
  3149:   bool Eval = evaluate(BrI, Inputs, Targets, FallsThru);
  3150:   unsigned NumTargets = Targets.size();
  3151:   if (!Eval || NumTargets > 1 || (NumTargets == 1 && FallsThru))
  3152:     return false;
  3153:   if (BrI.getOpcode() == Hexagon::J2_jump)
  3154:     return false;
  3155: 
  3156:   LLVM_DEBUG(dbgs() << "Rewrite(" << printMBBReference(B) << "):" << BrI);
  3157:   bool Rewritten = false;
  3158:   if (NumTargets > 0) {
  3159:     assert(!FallsThru && "This should have been checked before");
  3160:     // MIB.addMBB needs non-const pointer.
  3161:     MachineBasicBlock *TargetB = const_cast<MachineBasicBlock*>(Targets[0]);
  3162:     bool Moot = B.isLayoutSuccessor(TargetB);
  3163:     if (!Moot) {
  3164:       // If we build a branch here, we must make sure that it won't be
  3165:       // erased as "non-executable". We can't mark any new instructions
  3166:       // as executable here, so we need to overwrite the BrI, which we
  3167:       // know is executable.
  3168:       const MCInstrDesc &JD = HII.get(Hexagon::J2_jump);
  3169:       auto NI = BuildMI(B, BrI.getIterator(), BrI.getDebugLoc(), JD)
  3170:                   .addMBB(TargetB);
  3171:       BrI.setDesc(JD);
  3172:       while (BrI.getNumOperands() > 0)
  3173:         BrI.removeOperand(0);
  3174:       // This ensures that all implicit operands (e.g. implicit-def %r31, etc)
  3175:       // are present in the rewritten branch.
  3176:       for (auto &Op : NI->operands())
  3177:         BrI.addOperand(Op);
  3178:       NI->eraseFromParent();
  3179:       Rewritten = true;
  3180:     }
  3181:   }
  3182: 
  3183:   // Do not erase instructions. A newly created instruction could get
  3184:   // the same address as an instruction marked as executable during the
  3185:   // propagation.
  3186:   if (!Rewritten)
  3187:     replaceWithNop(BrI);
  3188:   return true;
  3189: }
  3190: 
  3191: FunctionPass *llvm::createHexagonConstPropagationPass() {
  3192:   return new HexagonConstPropagation();
  3193: }
```
- EN: It declares or implements routines such as getRegClass, createVirtualRegister, BuildMI, replaceAllRegUsesWith, ... (26 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonConstEvaluator, HexagonConstPropagation, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getRegClass, createVirtualRegister, BuildMI, replaceAllRegUsesWith, ... (26 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonConstEvaluator, HexagonConstPropagation，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- instruction semantics / 指令语义
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `HexagonInstrInfo.h, HexagonRegisterInfo.h, HexagonSubtarget.h, llvm/ADT/APFloat.h, llvm/ADT/APInt.h, llvm/ADT/PostOrderIterator.h, llvm/ADT/SetVector.h, llvm/ADT/SmallVector.h, llvm/ADT/StringRef.h, llvm/CodeGen/MachineBasicBlock.h, ... (36 total)`
- Hexagon symbols / Hexagon 符号: `HexagonConstPropagation, HexagonInstrInfo, HexagonRegisterInfo, HexagonSubtarget, HexagonConstEvaluator`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
