# HexagonConstExtenders.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonConstExtenders.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Bool contains(int32_t V) const {
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-200 / 第 1-200 行

```cpp
     1: //===- HexagonConstExtenders.cpp ------------------------------------------===//
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
    12: #include "llvm/ADT/SetVector.h"
    13: #include "llvm/ADT/SmallVector.h"
    14: #include "llvm/CodeGen/MachineDominators.h"
    15: #include "llvm/CodeGen/MachineFunctionPass.h"
    16: #include "llvm/CodeGen/MachineInstrBuilder.h"
    17: #include "llvm/CodeGen/MachineRegisterInfo.h"
    18: #include "llvm/CodeGen/Register.h"
    19: #include "llvm/InitializePasses.h"
    20: #include "llvm/Pass.h"
    21: #include "llvm/Support/CommandLine.h"
    22: #include "llvm/Support/raw_ostream.h"
    23: #include <map>
    24: #include <set>
    25: #include <utility>
    26: #include <vector>
    27: 
    28: #define DEBUG_TYPE "hexagon-cext-opt"
    29: 
    30: using namespace llvm;
    31: 
    32: static cl::opt<unsigned> CountThreshold(
    33:     "hexagon-cext-threshold", cl::init(3), cl::Hidden,
    34:     cl::desc("Minimum number of extenders to trigger replacement"));
    35: 
    36: static cl::opt<unsigned>
    37:     ReplaceLimit("hexagon-cext-limit", cl::init(0), cl::Hidden,
    38:                  cl::desc("Maximum number of replacements"));
    39: 
    40: static int32_t adjustUp(int32_t V, uint8_t A, uint8_t O) {
    41:   assert(isPowerOf2_32(A));
    42:   int32_t U = (V & -A) + O;
    43:   return U >= V ? U : U+A;
    44: }
    45: 
    46: static int32_t adjustDown(int32_t V, uint8_t A, uint8_t O) {
    47:   assert(isPowerOf2_32(A));
    48:   int32_t U = (V & -A) + O;
    49:   return U <= V ? U : U-A;
    50: }
    51: 
    52: namespace {
    53:   struct OffsetRange {
    54:     // The range of values between Min and Max that are of form Align*N+Offset,
    55:     // for some integer N. Min and Max are required to be of that form as well,
    56:     // except in the case of an empty range.
    57:     int32_t Min = INT_MIN, Max = INT_MAX;
    58:     uint8_t Align = 1;
    59:     uint8_t Offset = 0;
    60: 
    61:     OffsetRange() = default;
    62:     OffsetRange(int32_t L, int32_t H, uint8_t A, uint8_t O = 0)
    63:       : Min(L), Max(H), Align(A), Offset(O) {}
    64:     OffsetRange &intersect(OffsetRange A) {
    65:       if (Align < A.Align)
    66:         std::swap(*this, A);
    67: 
    68:       // Align >= A.Align.
    69:       if (Offset >= A.Offset && (Offset - A.Offset) % A.Align == 0) {
    70:         Min = adjustUp(std::max(Min, A.Min), Align, Offset);
    71:         Max = adjustDown(std::min(Max, A.Max), Align, Offset);
    72:       } else {
    73:         // Make an empty range.
    74:         Min = 0;
    75:         Max = -1;
    76:       }
    77:       // Canonicalize empty ranges.
    78:       if (Min > Max)
    79:         std::tie(Min, Max, Align) = std::make_tuple(0, -1, 1);
    80:       return *this;
    81:     }
    82:     OffsetRange &shift(int32_t S) {
    83:       Min += S;
    84:       Max += S;
    85:       Offset = (Offset+S) % Align;
    86:       return *this;
    87:     }
    88:     OffsetRange &extendBy(int32_t D) {
    89:       // If D < 0, extend Min, otherwise extend Max.
    90:       assert(D % Align == 0);
    91:       if (D < 0)
    92:         Min = (INT_MIN-D < Min) ? Min+D : INT_MIN;
    93:       else
    94:         Max = (INT_MAX-D > Max) ? Max+D : INT_MAX;
    95:       return *this;
    96:     }
    97:     bool empty() const {
    98:       return Min > Max;
    99:     }
   100:     bool contains(int32_t V) const {
   101:       return Min <= V && V <= Max && (V-Offset) % Align == 0;
   102:     }
   103:     bool operator==(const OffsetRange &R) const {
   104:       return Min == R.Min && Max == R.Max && Align == R.Align;
   105:     }
   106:     bool operator!=(const OffsetRange &R) const {
   107:       return !operator==(R);
   108:     }
   109:     bool operator<(const OffsetRange &R) const {
   110:       return std::tie(Min, Max, Align) < std::tie(R.Min, R.Max, R.Align);
   111:     }
   112:     static OffsetRange zero() { return {0, 0, 1}; }
   113:   };
   114: 
   115:   struct RangeTree {
   116:     struct Node {
   117:       Node(const OffsetRange &R) : MaxEnd(R.Max), Range(R) {}
   118:       unsigned Height = 1;
   119:       unsigned Count = 1;
   120:       int32_t MaxEnd;
   121:       const OffsetRange &Range;
   122:       Node *Left = nullptr, *Right = nullptr;
   123:     };
   124: 
   125:     Node *Root = nullptr;
   126: 
   127:     void add(const OffsetRange &R) {
   128:       Root = add(Root, R);
   129:     }
   130:     void erase(const Node *N) {
   131:       Root = remove(Root, N);
   132:       delete N;
   133:     }
   134:     void order(SmallVectorImpl<Node*> &Seq) const {
   135:       order(Root, Seq);
   136:     }
   137:     SmallVector<Node*,8> nodesWith(int32_t P, bool CheckAlign = true) {
   138:       SmallVector<Node*,8> Nodes;
   139:       nodesWith(Root, P, CheckAlign, Nodes);
   140:       return Nodes;
   141:     }
   142:     void dump() const;
   143:     ~RangeTree() {
   144:       SmallVector<Node*,8> Nodes;
   145:       order(Nodes);
   146:       for (Node *N : Nodes)
   147:         delete N;
   148:     }
   149: 
   150:   private:
   151:     void dump(const Node *N) const;
   152:     void order(Node *N, SmallVectorImpl<Node*> &Seq) const;
   153:     void nodesWith(Node *N, int32_t P, bool CheckA,
   154:                    SmallVectorImpl<Node*> &Seq) const;
   155: 
   156:     Node *add(Node *N, const OffsetRange &R);
   157:     Node *remove(Node *N, const Node *D);
   158:     Node *rotateLeft(Node *Lower, Node *Higher);
   159:     Node *rotateRight(Node *Lower, Node *Higher);
   160:     unsigned height(Node *N) {
   161:       return N != nullptr ? N->Height : 0;
   162:     }
   163:     Node *update(Node *N) {
   164:       assert(N != nullptr);
   165:       N->Height = 1 + std::max(height(N->Left), height(N->Right));
   166:       if (N->Left)
   167:         N->MaxEnd = std::max(N->MaxEnd, N->Left->MaxEnd);
   168:       if (N->Right)
   169:         N->MaxEnd = std::max(N->MaxEnd, N->Right->MaxEnd);
   170:       return N;
   171:     }
   172:     Node *rebalance(Node *N) {
   173:       assert(N != nullptr);
   174:       int32_t Balance = height(N->Right) - height(N->Left);
   175:       if (Balance < -1)
   176:         return rotateRight(N->Left, N);
   177:       if (Balance > 1)
   178:         return rotateLeft(N->Right, N);
   179:       return N;
   180:     }
   181:   };
   182: 
   183:   struct Loc {
   184:     MachineBasicBlock *Block = nullptr;
   185:     MachineBasicBlock::iterator At;
   186: 
   187:     Loc(MachineBasicBlock *B, MachineBasicBlock::iterator It)
   188:       : Block(B), At(It) {
   189:       if (B->end() == It) {
   190:         Pos = -1;
   191:       } else {
   192:         assert(It->getParent() == B);
   193:         Pos = std::distance(B->begin(), It);
   194:       }
   195:     }
   196:     bool operator<(Loc A) const {
   197:       if (Block != A.Block)
   198:         return Block->getNumber() < A.Block->getNumber();
   199:       if (A.Pos == -1)
   200:         return Pos != A.Pos;
```
- EN: It imports headers such as HexagonInstrInfo.h, HexagonRegisterInfo.h, HexagonSubtarget.h, llvm/ADT/SetVector.h, ... (18 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as OffsetRange, RangeTree, Node, Loc, which carry the state or API of this component.
- CN: 这里引入了 HexagonInstrInfo.h, HexagonRegisterInfo.h, HexagonSubtarget.h, llvm/ADT/SetVector.h, ... (18 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 OffsetRange, RangeTree, Node, Loc 等类型，用来承载该组件的状态或接口。

### Lines 201-400 / 第 201-400 行

```cpp
   201:       return Pos != -1 && Pos < A.Pos;
   202:     }
   203:   private:
   204:     int Pos = 0;
   205:   };
   206: 
   207:   struct HexagonConstExtenders : public MachineFunctionPass {
   208:     static char ID;
   209:     HexagonConstExtenders() : MachineFunctionPass(ID) {}
   210: 
   211:     void getAnalysisUsage(AnalysisUsage &AU) const override {
   212:       AU.addRequired<MachineDominatorTreeWrapperPass>();
   213:       AU.addPreserved<MachineDominatorTreeWrapperPass>();
   214:       MachineFunctionPass::getAnalysisUsage(AU);
   215:     }
   216: 
   217:     StringRef getPassName() const override {
   218:       return "Hexagon constant-extender optimization";
   219:     }
   220:     bool runOnMachineFunction(MachineFunction &MF) override;
   221: 
   222:   private:
   223:     struct Register {
   224:       Register() = default;
   225:       Register(llvm::Register R, unsigned S) : Reg(R), Sub(S) {}
   226:       Register(const MachineOperand &Op)
   227:         : Reg(Op.getReg()), Sub(Op.getSubReg()) {}
   228:       Register &operator=(const MachineOperand &Op) {
   229:         if (Op.isReg()) {
   230:           Reg = Op.getReg();
   231:           Sub = Op.getSubReg();
   232:         } else if (Op.isFI()) {
   233:           Reg = llvm::Register::index2StackSlot(Op.getIndex());
   234:         }
   235:         return *this;
   236:       }
   237:       bool isVReg() const {
   238:         return Reg != 0 && !Reg.isStack() && Reg.isVirtual();
   239:       }
   240:       bool isSlot() const { return Reg != 0 && Reg.isStack(); }
   241:       operator MachineOperand() const {
   242:         if (isVReg())
   243:           return MachineOperand::CreateReg(Reg, /*Def*/false, /*Imp*/false,
   244:                           /*Kill*/false, /*Dead*/false, /*Undef*/false,
   245:                           /*EarlyClobber*/false, Sub);
   246:         if (Reg.isStack()) {
   247:           int FI = Reg.stackSlotIndex();
   248:           return MachineOperand::CreateFI(FI);
   249:         }
   250:         llvm_unreachable("Cannot create MachineOperand");
   251:       }
   252:       bool operator==(Register R) const { return Reg == R.Reg && Sub == R.Sub; }
   253:       bool operator!=(Register R) const { return !operator==(R); }
   254:       bool operator<(Register R) const {
   255:         // For std::map.
   256:         return std::tie(Reg, Sub) < std::tie(R.Reg, R.Sub);
   257:       }
   258:       llvm::Register Reg;
   259:       unsigned Sub = 0;
   260:     };
   261: 
   262:     struct ExtExpr {
   263:       // A subexpression in which the extender is used. In general, this
   264:       // represents an expression where adding D to the extender will be
   265:       // equivalent to adding D to the expression as a whole. In other
   266:       // words, expr(add(##V,D) = add(expr(##V),D).
   267: 
   268:       // The original motivation for this are the io/ur addressing modes,
   269:       // where the offset is extended. Consider the io example:
   270:       // In memw(Rs+##V), the ##V could be replaced by a register Rt to
   271:       // form the rr mode: memw(Rt+Rs<<0). In such case, however, the
   272:       // register Rt must have exactly the value of ##V. If there was
   273:       // another instruction memw(Rs+##V+4), it would need a different Rt.
   274:       // Now, if Rt was initialized as "##V+Rs<<0", both of these
   275:       // instructions could use the same Rt, just with different offsets.
   276:       // Here it's clear that "initializer+4" should be the same as if
   277:       // the offset 4 was added to the ##V in the initializer.
   278: 
   279:       // The only kinds of expressions that support the requirement of
   280:       // commuting with addition are addition and subtraction from ##V.
   281:       // Include shifting the Rs to account for the ur addressing mode:
   282:       //   ##Val + Rs << S
   283:       //   ##Val - Rs
   284:       Register Rs;
   285:       unsigned S = 0;
   286:       bool Neg = false;
   287: 
   288:       ExtExpr() = default;
   289:       ExtExpr(Register RS, bool NG, unsigned SH) : Rs(RS), S(SH), Neg(NG) {}
   290:       // Expression is trivial if it does not modify the extender.
   291:       bool trivial() const {
   292:         return Rs.Reg == 0;
   293:       }
   294:       bool operator==(const ExtExpr &Ex) const {
   295:         return Rs == Ex.Rs && S == Ex.S && Neg == Ex.Neg;
   296:       }
   297:       bool operator!=(const ExtExpr &Ex) const {
   298:         return !operator==(Ex);
   299:       }
   300:       bool operator<(const ExtExpr &Ex) const {
   301:         return std::tie(Rs, S, Neg) < std::tie(Ex.Rs, Ex.S, Ex.Neg);
   302:       }
   303:     };
   304: 
   305:     struct ExtDesc {
   306:       MachineInstr *UseMI = nullptr;
   307:       unsigned OpNum = -1u;
   308:       // The subexpression in which the extender is used (e.g. address
   309:       // computation).
   310:       ExtExpr Expr;
   311:       // Optional register that is assigned the value of Expr.
   312:       Register Rd;
   313:       // Def means that the output of the instruction may differ from the
   314:       // original by a constant c, and that the difference can be corrected
   315:       // by adding/subtracting c in all users of the defined register.
   316:       bool IsDef = false;
   317: 
   318:       MachineOperand &getOp() {
   319:         return UseMI->getOperand(OpNum);
   320:       }
   321:       const MachineOperand &getOp() const {
   322:         return UseMI->getOperand(OpNum);
   323:       }
   324:     };
   325: 
   326:     struct ExtRoot {
   327:       union {
   328:         const ConstantFP *CFP;  // MO_FPImmediate
   329:         const char *SymbolName; // MO_ExternalSymbol
   330:         const GlobalValue *GV;  // MO_GlobalAddress
   331:         const BlockAddress *BA; // MO_BlockAddress
   332:         int64_t ImmVal;         // MO_Immediate, MO_TargetIndex,
   333:                                 // and MO_ConstantPoolIndex
   334:       } V;
   335:       unsigned Kind;            // Same as in MachineOperand.
   336:       unsigned char TF;         // TargetFlags.
   337: 
   338:       ExtRoot(const MachineOperand &Op);
   339:       bool operator==(const ExtRoot &ER) const {
   340:         return Kind == ER.Kind && V.ImmVal == ER.V.ImmVal;
   341:       }
   342:       bool operator!=(const ExtRoot &ER) const {
   343:         return !operator==(ER);
   344:       }
   345:       bool operator<(const ExtRoot &ER) const;
   346:     };
   347: 
   348:     struct ExtValue : public ExtRoot {
   349:       int32_t Offset;
   350: 
   351:       ExtValue(const MachineOperand &Op);
   352:       ExtValue(const ExtDesc &ED) : ExtValue(ED.getOp()) {}
   353:       ExtValue(const ExtRoot &ER, int32_t Off) : ExtRoot(ER), Offset(Off) {}
   354:       bool operator<(const ExtValue &EV) const;
   355:       bool operator==(const ExtValue &EV) const {
   356:         return ExtRoot(*this) == ExtRoot(EV) && Offset == EV.Offset;
   357:       }
   358:       bool operator!=(const ExtValue &EV) const {
   359:         return !operator==(EV);
   360:       }
   361:       explicit operator MachineOperand() const;
   362:     };
   363: 
   364:     using IndexList = SetVector<unsigned>;
   365:     using ExtenderInit = std::pair<ExtValue, ExtExpr>;
   366:     using AssignmentMap = std::map<ExtenderInit, IndexList>;
   367:     using LocDefList = std::vector<std::pair<Loc, IndexList>>;
   368: 
   369:     const HexagonSubtarget *HST = nullptr;
   370:     const HexagonInstrInfo *HII = nullptr;
   371:     const HexagonRegisterInfo *HRI = nullptr;
   372:     MachineDominatorTree *MDT = nullptr;
   373:     MachineRegisterInfo *MRI = nullptr;
   374:     std::vector<ExtDesc> Extenders;
   375:     std::vector<unsigned> NewRegs;
   376: 
   377:     bool isStoreImmediate(unsigned Opc) const;
   378:     bool isRegOffOpcode(unsigned ExtOpc) const ;
   379:     unsigned getRegOffOpcode(unsigned ExtOpc) const;
   380:     unsigned getDirectRegReplacement(unsigned ExtOpc) const;
   381:     OffsetRange getOffsetRange(Register R, const MachineInstr &MI) const;
   382:     OffsetRange getOffsetRange(const ExtDesc &ED) const;
   383:     OffsetRange getOffsetRange(Register Rd) const;
   384: 
   385:     void recordExtender(MachineInstr &MI, unsigned OpNum);
   386:     void collectInstr(MachineInstr &MI);
   387:     void collect(MachineFunction &MF);
   388:     void assignInits(const ExtRoot &ER, unsigned Begin, unsigned End,
   389:                      AssignmentMap &IMap);
   390:     void calculatePlacement(const ExtenderInit &ExtI, const IndexList &Refs,
   391:                             LocDefList &Defs);
   392:     Register insertInitializer(Loc DefL, const ExtenderInit &ExtI);
   393:     bool replaceInstrExact(const ExtDesc &ED, Register ExtR);
   394:     bool replaceInstrExpr(const ExtDesc &ED, const ExtenderInit &ExtI,
   395:                           Register ExtR, int32_t &Diff);
   396:     bool replaceInstr(unsigned Idx, Register ExtR, const ExtenderInit &ExtI);
   397:     bool replaceExtenders(const AssignmentMap &IMap);
   398: 
   399:     unsigned getOperandIndex(const MachineInstr &MI,
   400:                              const MachineOperand &Op) const;
```
- EN: It declares types such as HexagonConstExtenders, Register, ExtExpr, ExtDesc, ... (6 total), which carry the state or API of this component. It declares or implements routines such as HexagonConstExtenders, getAnalysisUsage, addRequired<MachineDominatorTreeWrapperPass>, addPreserved<MachineDominatorTreeWrapperPass>, ... (41 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonConstExtenders, HexagonSubtarget, HexagonInstrInfo, HexagonRegisterInfo, showing how the code connects to sibling backend components.
- CN: 这里声明了 HexagonConstExtenders, Register, ExtExpr, ExtDesc, ... (6 total) 等类型，用来承载该组件的状态或接口。 这里声明或实现了 HexagonConstExtenders, getAnalysisUsage, addRequired<MachineDominatorTreeWrapperPass>, addPreserved<MachineDominatorTreeWrapperPass>, ... (41 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonConstExtenders, HexagonSubtarget, HexagonInstrInfo, HexagonRegisterInfo，说明了它与同级后端组件的连接关系。

### Lines 401-600 / 第 401-600 行

```cpp
   401:     const MachineOperand &getPredicateOp(const MachineInstr &MI) const;
   402:     const MachineOperand &getLoadResultOp(const MachineInstr &MI) const;
   403:     const MachineOperand &getStoredValueOp(const MachineInstr &MI) const;
   404: 
   405:     friend struct PrintRegister;
   406:     friend struct PrintExpr;
   407:     friend struct PrintInit;
   408:     friend struct PrintIMap;
   409:     friend raw_ostream &operator<< (raw_ostream &OS,
   410:                                     const struct PrintRegister &P);
   411:     friend raw_ostream &operator<< (raw_ostream &OS, const struct PrintExpr &P);
   412:     friend raw_ostream &operator<< (raw_ostream &OS, const struct PrintInit &P);
   413:     friend raw_ostream &operator<< (raw_ostream &OS, const ExtDesc &ED);
   414:     friend raw_ostream &operator<< (raw_ostream &OS, const ExtRoot &ER);
   415:     friend raw_ostream &operator<< (raw_ostream &OS, const ExtValue &EV);
   416:     friend raw_ostream &operator<< (raw_ostream &OS, const OffsetRange &OR);
   417:     friend raw_ostream &operator<< (raw_ostream &OS, const struct PrintIMap &P);
   418:   };
   419: 
   420:   using HCE = HexagonConstExtenders;
   421: 
   422:   [[maybe_unused]]
   423:   raw_ostream &operator<<(raw_ostream &OS, const OffsetRange &OR) {
   424:     if (OR.Min > OR.Max)
   425:       OS << '!';
   426:     OS << '[' << OR.Min << ',' << OR.Max << "]a" << unsigned(OR.Align)
   427:        << '+' << unsigned(OR.Offset);
   428:     return OS;
   429:   }
   430: 
   431:   struct PrintRegister {
   432:     PrintRegister(HCE::Register R, const HexagonRegisterInfo &I)
   433:       : Rs(R), HRI(I) {}
   434:     HCE::Register Rs;
   435:     const HexagonRegisterInfo &HRI;
   436:   };
   437: 
   438:   [[maybe_unused]]
   439:   raw_ostream &operator<<(raw_ostream &OS, const PrintRegister &P) {
   440:     if (P.Rs.Reg != 0)
   441:       OS << printReg(P.Rs.Reg, &P.HRI, P.Rs.Sub);
   442:     else
   443:       OS << "noreg";
   444:     return OS;
   445:   }
   446: 
   447:   struct PrintExpr {
   448:     PrintExpr(const HCE::ExtExpr &E, const HexagonRegisterInfo &I)
   449:       : Ex(E), HRI(I) {}
   450:     const HCE::ExtExpr &Ex;
   451:     const HexagonRegisterInfo &HRI;
   452:   };
   453: 
   454:   [[maybe_unused]]
   455:   raw_ostream &operator<<(raw_ostream &OS, const PrintExpr &P) {
   456:     OS << "## " << (P.Ex.Neg ? "- " : "+ ");
   457:     if (P.Ex.Rs.Reg != 0)
   458:       OS << printReg(P.Ex.Rs.Reg, &P.HRI, P.Ex.Rs.Sub);
   459:     else
   460:       OS << "__";
   461:     OS << " << " << P.Ex.S;
   462:     return OS;
   463:   }
   464: 
   465:   struct PrintInit {
   466:     PrintInit(const HCE::ExtenderInit &EI, const HexagonRegisterInfo &I)
   467:       : ExtI(EI), HRI(I) {}
   468:     const HCE::ExtenderInit &ExtI;
   469:     const HexagonRegisterInfo &HRI;
   470:   };
   471: 
   472:   [[maybe_unused]]
   473:   raw_ostream &operator<<(raw_ostream &OS, const PrintInit &P) {
   474:     OS << '[' << P.ExtI.first << ", "
   475:        << PrintExpr(P.ExtI.second, P.HRI) << ']';
   476:     return OS;
   477:   }
   478: 
   479:   [[maybe_unused]]
   480:   raw_ostream &operator<<(raw_ostream &OS, const HCE::ExtDesc &ED) {
   481:     assert(ED.OpNum != -1u);
   482:     const MachineBasicBlock &MBB = *ED.getOp().getParent()->getParent();
   483:     const MachineFunction &MF = *MBB.getParent();
   484:     const auto &HRI = *MF.getSubtarget<HexagonSubtarget>().getRegisterInfo();
   485:     OS << "bb#" << MBB.getNumber() << ": ";
   486:     if (ED.Rd.Reg != 0)
   487:       OS << printReg(ED.Rd.Reg, &HRI, ED.Rd.Sub);
   488:     else
   489:       OS << "__";
   490:     OS << " = " << PrintExpr(ED.Expr, HRI);
   491:     if (ED.IsDef)
   492:       OS << ", def";
   493:     return OS;
   494:   }
   495: 
   496:   [[maybe_unused]]
   497:   raw_ostream &operator<<(raw_ostream &OS, const HCE::ExtRoot &ER) {
   498:     switch (ER.Kind) {
   499:       case MachineOperand::MO_Immediate:
   500:         OS << "imm:" << ER.V.ImmVal;
   501:         break;
   502:       case MachineOperand::MO_FPImmediate:
   503:         OS << "fpi:" << *ER.V.CFP;
   504:         break;
   505:       case MachineOperand::MO_ExternalSymbol:
   506:         OS << "sym:" << *ER.V.SymbolName;
   507:         break;
   508:       case MachineOperand::MO_GlobalAddress:
   509:         OS << "gad:" << ER.V.GV->getName();
   510:         break;
   511:       case MachineOperand::MO_BlockAddress:
   512:         OS << "blk:" << *ER.V.BA;
   513:         break;
   514:       case MachineOperand::MO_TargetIndex:
   515:         OS << "tgi:" << ER.V.ImmVal;
   516:         break;
   517:       case MachineOperand::MO_ConstantPoolIndex:
   518:         OS << "cpi:" << ER.V.ImmVal;
   519:         break;
   520:       case MachineOperand::MO_JumpTableIndex:
   521:         OS << "jti:" << ER.V.ImmVal;
   522:         break;
   523:       default:
   524:         OS << "???:" << ER.V.ImmVal;
   525:         break;
   526:     }
   527:     return OS;
   528:   }
   529: 
   530:   [[maybe_unused]]
   531:   raw_ostream &operator<<(raw_ostream &OS, const HCE::ExtValue &EV) {
   532:     OS << HCE::ExtRoot(EV) << "  off:" << EV.Offset;
   533:     return OS;
   534:   }
   535: 
   536:   struct PrintIMap {
   537:     PrintIMap(const HCE::AssignmentMap &M, const HexagonRegisterInfo &I)
   538:       : IMap(M), HRI(I) {}
   539:     const HCE::AssignmentMap &IMap;
   540:     const HexagonRegisterInfo &HRI;
   541:   };
   542: 
   543:   [[maybe_unused]]
   544:   raw_ostream &operator<<(raw_ostream &OS, const PrintIMap &P) {
   545:     OS << "{\n";
   546:     for (const std::pair<const HCE::ExtenderInit, HCE::IndexList> &Q : P.IMap) {
   547:       OS << "  " << PrintInit(Q.first, P.HRI) << " -> {";
   548:       for (unsigned I : Q.second)
   549:         OS << ' ' << I;
   550:       OS << " }\n";
   551:     }
   552:     OS << "}\n";
   553:     return OS;
   554:   }
   555: }
   556: 
   557: INITIALIZE_PASS_BEGIN(HexagonConstExtenders, "hexagon-cext-opt",
   558:       "Hexagon constant-extender optimization", false, false)
   559: INITIALIZE_PASS_DEPENDENCY(MachineDominatorTreeWrapperPass)
   560: INITIALIZE_PASS_END(HexagonConstExtenders, "hexagon-cext-opt",
   561:       "Hexagon constant-extender optimization", false, false)
   562: 
   563: static unsigned ReplaceCounter = 0;
   564: 
   565: char HCE::ID = 0;
   566: 
   567: #ifndef NDEBUG
   568: LLVM_DUMP_METHOD void RangeTree::dump() const {
   569:   dbgs() << "Root: " << Root << '\n';
   570:   if (Root)
   571:     dump(Root);
   572: }
   573: 
   574: LLVM_DUMP_METHOD void RangeTree::dump(const Node *N) const {
   575:   dbgs() << "Node: " << N << '\n';
   576:   dbgs() << "  Height: " << N->Height << '\n';
   577:   dbgs() << "  Count: " << N->Count << '\n';
   578:   dbgs() << "  MaxEnd: " << N->MaxEnd << '\n';
   579:   dbgs() << "  Range: " << N->Range << '\n';
   580:   dbgs() << "  Left: " << N->Left << '\n';
   581:   dbgs() << "  Right: " << N->Right << "\n\n";
   582: 
   583:   if (N->Left)
   584:     dump(N->Left);
   585:   if (N->Right)
   586:     dump(N->Right);
   587: }
   588: #endif
   589: 
   590: void RangeTree::order(Node *N, SmallVectorImpl<Node*> &Seq) const {
   591:   if (N == nullptr)
   592:     return;
   593:   order(N->Left, Seq);
   594:   Seq.push_back(N);
   595:   order(N->Right, Seq);
   596: }
   597: 
   598: void RangeTree::nodesWith(Node *N, int32_t P, bool CheckA,
   599:       SmallVectorImpl<Node*> &Seq) const {
   600:   if (N == nullptr || N->MaxEnd < P)
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. It declares types such as PrintRegister, PrintExpr, PrintInit, PrintIMap, which carry the state or API of this component. It declares or implements routines such as getPredicateOp, getLoadResultOp, getStoredValueOp, unsigned, ... (18 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里声明了 PrintRegister, PrintExpr, PrintInit, PrintIMap 等类型，用来承载该组件的状态或接口。 这里声明或实现了 getPredicateOp, getLoadResultOp, getStoredValueOp, unsigned, ... (18 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 601-800 / 第 601-800 行

```cpp
   601:     return;
   602:   nodesWith(N->Left, P, CheckA, Seq);
   603:   if (N->Range.Min <= P) {
   604:     if ((CheckA && N->Range.contains(P)) || (!CheckA && P <= N->Range.Max))
   605:       Seq.push_back(N);
   606:     nodesWith(N->Right, P, CheckA, Seq);
   607:   }
   608: }
   609: 
   610: RangeTree::Node *RangeTree::add(Node *N, const OffsetRange &R) {
   611:   if (N == nullptr)
   612:     return new Node(R);
   613: 
   614:   if (N->Range == R) {
   615:     N->Count++;
   616:     return N;
   617:   }
   618: 
   619:   if (R < N->Range)
   620:     N->Left = add(N->Left, R);
   621:   else
   622:     N->Right = add(N->Right, R);
   623:   return rebalance(update(N));
   624: }
   625: 
   626: RangeTree::Node *RangeTree::remove(Node *N, const Node *D) {
   627:   assert(N != nullptr);
   628: 
   629:   if (N != D) {
   630:     assert(N->Range != D->Range && "N and D should not be equal");
   631:     if (D->Range < N->Range)
   632:       N->Left = remove(N->Left, D);
   633:     else
   634:       N->Right = remove(N->Right, D);
   635:     return rebalance(update(N));
   636:   }
   637: 
   638:   // We got to the node we need to remove. If any of its children are
   639:   // missing, simply replace it with the other child.
   640:   if (N->Left == nullptr || N->Right == nullptr)
   641:     return (N->Left == nullptr) ? N->Right : N->Left;
   642: 
   643:   // Find the rightmost child of N->Left, remove it and plug it in place
   644:   // of N.
   645:   Node *M = N->Left;
   646:   while (M->Right)
   647:     M = M->Right;
   648:   M->Left = remove(N->Left, M);
   649:   M->Right = N->Right;
   650:   return rebalance(update(M));
   651: }
   652: 
   653: RangeTree::Node *RangeTree::rotateLeft(Node *Lower, Node *Higher) {
   654:   assert(Higher->Right == Lower);
   655:   // The Lower node is on the right from Higher. Make sure that Lower's
   656:   // balance is greater to the right. Otherwise the rotation will create
   657:   // an unbalanced tree again.
   658:   if (height(Lower->Left) > height(Lower->Right))
   659:     Lower = rotateRight(Lower->Left, Lower);
   660:   assert(height(Lower->Left) <= height(Lower->Right));
   661:   Higher->Right = Lower->Left;
   662:   update(Higher);
   663:   Lower->Left = Higher;
   664:   update(Lower);
   665:   return Lower;
   666: }
   667: 
   668: RangeTree::Node *RangeTree::rotateRight(Node *Lower, Node *Higher) {
   669:   assert(Higher->Left == Lower);
   670:   // The Lower node is on the left from Higher. Make sure that Lower's
   671:   // balance is greater to the left. Otherwise the rotation will create
   672:   // an unbalanced tree again.
   673:   if (height(Lower->Left) < height(Lower->Right))
   674:     Lower = rotateLeft(Lower->Right, Lower);
   675:   assert(height(Lower->Left) >= height(Lower->Right));
   676:   Higher->Left = Lower->Right;
   677:   update(Higher);
   678:   Lower->Right = Higher;
   679:   update(Lower);
   680:   return Lower;
   681: }
   682: 
   683: 
   684: HCE::ExtRoot::ExtRoot(const MachineOperand &Op) {
   685:   // Always store ImmVal, since it's the field used for comparisons.
   686:   V.ImmVal = 0;
   687:   if (Op.isImm())
   688:     ; // Keep 0. Do not use Op.getImm() for value here (treat 0 as the root).
   689:   else if (Op.isFPImm())
   690:     V.CFP = Op.getFPImm();
   691:   else if (Op.isSymbol())
   692:     V.SymbolName = Op.getSymbolName();
   693:   else if (Op.isGlobal())
   694:     V.GV = Op.getGlobal();
   695:   else if (Op.isBlockAddress())
   696:     V.BA = Op.getBlockAddress();
   697:   else if (Op.isCPI() || Op.isTargetIndex() || Op.isJTI())
   698:     V.ImmVal = Op.getIndex();
   699:   else
   700:     llvm_unreachable("Unexpected operand type");
   701: 
   702:   Kind = Op.getType();
   703:   TF = Op.getTargetFlags();
   704: }
   705: 
   706: bool HCE::ExtRoot::operator< (const HCE::ExtRoot &ER) const {
   707:   if (Kind != ER.Kind)
   708:     return Kind < ER.Kind;
   709:   switch (Kind) {
   710:     case MachineOperand::MO_Immediate:
   711:     case MachineOperand::MO_TargetIndex:
   712:     case MachineOperand::MO_ConstantPoolIndex:
   713:     case MachineOperand::MO_JumpTableIndex:
   714:       return V.ImmVal < ER.V.ImmVal;
   715:     case MachineOperand::MO_FPImmediate: {
   716:       const APFloat &ThisF = V.CFP->getValueAPF();
   717:       const APFloat &OtherF = ER.V.CFP->getValueAPF();
   718:       return ThisF.bitcastToAPInt().ult(OtherF.bitcastToAPInt());
   719:     }
   720:     case MachineOperand::MO_ExternalSymbol:
   721:       return StringRef(V.SymbolName) < StringRef(ER.V.SymbolName);
   722:     case MachineOperand::MO_GlobalAddress:
   723:       // Do not use GUIDs, since they depend on the source path. Moving the
   724:       // source file to a different directory could cause different GUID
   725:       // values for a pair of given symbols. These symbols could then compare
   726:       // "less" in one directory, but "greater" in another.
   727:       assert(!V.GV->getName().empty() && !ER.V.GV->getName().empty());
   728:       return V.GV->getName() < ER.V.GV->getName();
   729:     case MachineOperand::MO_BlockAddress: {
   730:       const BasicBlock *ThisB = V.BA->getBasicBlock();
   731:       const BasicBlock *OtherB = ER.V.BA->getBasicBlock();
   732:       assert(ThisB->getParent() == OtherB->getParent());
   733:       const Function &F = *ThisB->getParent();
   734:       return std::distance(F.begin(), ThisB->getIterator()) <
   735:              std::distance(F.begin(), OtherB->getIterator());
   736:     }
   737:   }
   738:   return V.ImmVal < ER.V.ImmVal;
   739: }
   740: 
   741: HCE::ExtValue::ExtValue(const MachineOperand &Op) : ExtRoot(Op) {
   742:   if (Op.isImm())
   743:     Offset = Op.getImm();
   744:   else if (Op.isFPImm() || Op.isJTI())
   745:     Offset = 0;
   746:   else if (Op.isSymbol() || Op.isGlobal() || Op.isBlockAddress() ||
   747:            Op.isCPI() || Op.isTargetIndex())
   748:     Offset = Op.getOffset();
   749:   else
   750:     llvm_unreachable("Unexpected operand type");
   751: }
   752: 
   753: bool HCE::ExtValue::operator< (const HCE::ExtValue &EV) const {
   754:   const ExtRoot &ER = *this;
   755:   if (!(ER == ExtRoot(EV)))
   756:     return ER < EV;
   757:   return Offset < EV.Offset;
   758: }
   759: 
   760: HCE::ExtValue::operator MachineOperand() const {
   761:   switch (Kind) {
   762:     case MachineOperand::MO_Immediate:
   763:       return MachineOperand::CreateImm(V.ImmVal + Offset);
   764:     case MachineOperand::MO_FPImmediate:
   765:       assert(Offset == 0);
   766:       return MachineOperand::CreateFPImm(V.CFP);
   767:     case MachineOperand::MO_ExternalSymbol:
   768:       assert(Offset == 0);
   769:       return MachineOperand::CreateES(V.SymbolName, TF);
   770:     case MachineOperand::MO_GlobalAddress:
   771:       return MachineOperand::CreateGA(V.GV, Offset, TF);
   772:     case MachineOperand::MO_BlockAddress:
   773:       return MachineOperand::CreateBA(V.BA, Offset, TF);
   774:     case MachineOperand::MO_TargetIndex:
   775:       return MachineOperand::CreateTargetIndex(V.ImmVal, Offset, TF);
   776:     case MachineOperand::MO_ConstantPoolIndex:
   777:       return MachineOperand::CreateCPI(V.ImmVal, Offset, TF);
   778:     case MachineOperand::MO_JumpTableIndex:
   779:       assert(Offset == 0);
   780:       return MachineOperand::CreateJTI(V.ImmVal, TF);
   781:     default:
   782:       llvm_unreachable("Unhandled kind");
   783:  }
   784: }
   785: 
   786: bool HCE::isStoreImmediate(unsigned Opc) const {
   787:   switch (Opc) {
   788:     case Hexagon::S4_storeirbt_io:
   789:     case Hexagon::S4_storeirbf_io:
   790:     case Hexagon::S4_storeirht_io:
   791:     case Hexagon::S4_storeirhf_io:
   792:     case Hexagon::S4_storeirit_io:
   793:     case Hexagon::S4_storeirif_io:
   794:     case Hexagon::S4_storeirb_io:
   795:     case Hexagon::S4_storeirh_io:
   796:     case Hexagon::S4_storeiri_io:
   797:       return true;
   798:     default:
   799:       break;
   800:   }
```
- EN: It declares or implements routines such as nodesWith, RangeTree::add, add, rebalance, ... (33 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 nodesWith, RangeTree::add, add, rebalance, ... (33 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 801-1000 / 第 801-1000 行

```cpp
   801:   return false;
   802: }
   803: 
   804: bool HCE::isRegOffOpcode(unsigned Opc) const {
   805:   switch (Opc) {
   806:     case Hexagon::L2_loadrub_io:
   807:     case Hexagon::L2_loadrb_io:
   808:     case Hexagon::L2_loadruh_io:
   809:     case Hexagon::L2_loadrh_io:
   810:     case Hexagon::L2_loadri_io:
   811:     case Hexagon::L2_loadrd_io:
   812:     case Hexagon::L2_loadbzw2_io:
   813:     case Hexagon::L2_loadbzw4_io:
   814:     case Hexagon::L2_loadbsw2_io:
   815:     case Hexagon::L2_loadbsw4_io:
   816:     case Hexagon::L2_loadalignh_io:
   817:     case Hexagon::L2_loadalignb_io:
   818:     case Hexagon::L2_ploadrubt_io:
   819:     case Hexagon::L2_ploadrubf_io:
   820:     case Hexagon::L2_ploadrbt_io:
   821:     case Hexagon::L2_ploadrbf_io:
   822:     case Hexagon::L2_ploadruht_io:
   823:     case Hexagon::L2_ploadruhf_io:
   824:     case Hexagon::L2_ploadrht_io:
   825:     case Hexagon::L2_ploadrhf_io:
   826:     case Hexagon::L2_ploadrit_io:
   827:     case Hexagon::L2_ploadrif_io:
   828:     case Hexagon::L2_ploadrdt_io:
   829:     case Hexagon::L2_ploadrdf_io:
   830:     case Hexagon::S2_storerb_io:
   831:     case Hexagon::S2_storerh_io:
   832:     case Hexagon::S2_storerf_io:
   833:     case Hexagon::S2_storeri_io:
   834:     case Hexagon::S2_storerd_io:
   835:     case Hexagon::S2_pstorerbt_io:
   836:     case Hexagon::S2_pstorerbf_io:
   837:     case Hexagon::S2_pstorerht_io:
   838:     case Hexagon::S2_pstorerhf_io:
   839:     case Hexagon::S2_pstorerft_io:
   840:     case Hexagon::S2_pstorerff_io:
   841:     case Hexagon::S2_pstorerit_io:
   842:     case Hexagon::S2_pstorerif_io:
   843:     case Hexagon::S2_pstorerdt_io:
   844:     case Hexagon::S2_pstorerdf_io:
   845:     case Hexagon::A2_addi:
   846:       return true;
   847:     default:
   848:       break;
   849:   }
   850:   return false;
   851: }
   852: 
   853: unsigned HCE::getRegOffOpcode(unsigned ExtOpc) const {
   854:   // If there exists an instruction that takes a register and offset,
   855:   // that corresponds to the ExtOpc, return it, otherwise return 0.
   856:   using namespace Hexagon;
   857:   switch (ExtOpc) {
   858:     case A2_tfrsi:    return A2_addi;
   859:     default:
   860:       break;
   861:   }
   862:   const MCInstrDesc &D = HII->get(ExtOpc);
   863:   if (D.mayLoad() || D.mayStore()) {
   864:     uint64_t F = D.TSFlags;
   865:     unsigned AM = (F >> HexagonII::AddrModePos) & HexagonII::AddrModeMask;
   866:     switch (AM) {
   867:       case HexagonII::Absolute:
   868:       case HexagonII::AbsoluteSet:
   869:       case HexagonII::BaseLongOffset:
   870:         switch (ExtOpc) {
   871:           case PS_loadrubabs:
   872:           case L4_loadrub_ap:
   873:           case L4_loadrub_ur:     return L2_loadrub_io;
   874:           case PS_loadrbabs:
   875:           case L4_loadrb_ap:
   876:           case L4_loadrb_ur:      return L2_loadrb_io;
   877:           case PS_loadruhabs:
   878:           case L4_loadruh_ap:
   879:           case L4_loadruh_ur:     return L2_loadruh_io;
   880:           case PS_loadrhabs:
   881:           case L4_loadrh_ap:
   882:           case L4_loadrh_ur:      return L2_loadrh_io;
   883:           case PS_loadriabs:
   884:           case L4_loadri_ap:
   885:           case L4_loadri_ur:      return L2_loadri_io;
   886:           case PS_loadrdabs:
   887:           case L4_loadrd_ap:
   888:           case L4_loadrd_ur:      return L2_loadrd_io;
   889:           case L4_loadbzw2_ap:
   890:           case L4_loadbzw2_ur:    return L2_loadbzw2_io;
   891:           case L4_loadbzw4_ap:
   892:           case L4_loadbzw4_ur:    return L2_loadbzw4_io;
   893:           case L4_loadbsw2_ap:
   894:           case L4_loadbsw2_ur:    return L2_loadbsw2_io;
   895:           case L4_loadbsw4_ap:
   896:           case L4_loadbsw4_ur:    return L2_loadbsw4_io;
   897:           case L4_loadalignh_ap:
   898:           case L4_loadalignh_ur:  return L2_loadalignh_io;
   899:           case L4_loadalignb_ap:
   900:           case L4_loadalignb_ur:  return L2_loadalignb_io;
   901:           case L4_ploadrubt_abs:  return L2_ploadrubt_io;
   902:           case L4_ploadrubf_abs:  return L2_ploadrubf_io;
   903:           case L4_ploadrbt_abs:   return L2_ploadrbt_io;
   904:           case L4_ploadrbf_abs:   return L2_ploadrbf_io;
   905:           case L4_ploadruht_abs:  return L2_ploadruht_io;
   906:           case L4_ploadruhf_abs:  return L2_ploadruhf_io;
   907:           case L4_ploadrht_abs:   return L2_ploadrht_io;
   908:           case L4_ploadrhf_abs:   return L2_ploadrhf_io;
   909:           case L4_ploadrit_abs:   return L2_ploadrit_io;
   910:           case L4_ploadrif_abs:   return L2_ploadrif_io;
   911:           case L4_ploadrdt_abs:   return L2_ploadrdt_io;
   912:           case L4_ploadrdf_abs:   return L2_ploadrdf_io;
   913:           case PS_storerbabs:
   914:           case S4_storerb_ap:
   915:           case S4_storerb_ur:     return S2_storerb_io;
   916:           case PS_storerhabs:
   917:           case S4_storerh_ap:
   918:           case S4_storerh_ur:     return S2_storerh_io;
   919:           case PS_storerfabs:
   920:           case S4_storerf_ap:
   921:           case S4_storerf_ur:     return S2_storerf_io;
   922:           case PS_storeriabs:
   923:           case S4_storeri_ap:
   924:           case S4_storeri_ur:     return S2_storeri_io;
   925:           case PS_storerdabs:
   926:           case S4_storerd_ap:
   927:           case S4_storerd_ur:     return S2_storerd_io;
   928:           case S4_pstorerbt_abs:  return S2_pstorerbt_io;
   929:           case S4_pstorerbf_abs:  return S2_pstorerbf_io;
   930:           case S4_pstorerht_abs:  return S2_pstorerht_io;
   931:           case S4_pstorerhf_abs:  return S2_pstorerhf_io;
   932:           case S4_pstorerft_abs:  return S2_pstorerft_io;
   933:           case S4_pstorerff_abs:  return S2_pstorerff_io;
   934:           case S4_pstorerit_abs:  return S2_pstorerit_io;
   935:           case S4_pstorerif_abs:  return S2_pstorerif_io;
   936:           case S4_pstorerdt_abs:  return S2_pstorerdt_io;
   937:           case S4_pstorerdf_abs:  return S2_pstorerdf_io;
   938:           default:
   939:             break;
   940:         }
   941:         break;
   942:       case HexagonII::BaseImmOffset:
   943:         if (!isStoreImmediate(ExtOpc))
   944:           return ExtOpc;
   945:         break;
   946:       default:
   947:         break;
   948:     }
   949:   }
   950:   return 0;
   951: }
   952: 
   953: unsigned HCE::getDirectRegReplacement(unsigned ExtOpc) const {
   954:   switch (ExtOpc) {
   955:     case Hexagon::A2_addi:          return Hexagon::A2_add;
   956:     case Hexagon::A2_andir:         return Hexagon::A2_and;
   957:     case Hexagon::A2_combineii:     return Hexagon::A4_combineri;
   958:     case Hexagon::A2_orir:          return Hexagon::A2_or;
   959:     case Hexagon::A2_paddif:        return Hexagon::A2_paddf;
   960:     case Hexagon::A2_paddit:        return Hexagon::A2_paddt;
   961:     case Hexagon::A2_subri:         return Hexagon::A2_sub;
   962:     case Hexagon::A2_tfrsi:         return TargetOpcode::COPY;
   963:     case Hexagon::A4_cmpbeqi:       return Hexagon::A4_cmpbeq;
   964:     case Hexagon::A4_cmpbgti:       return Hexagon::A4_cmpbgt;
   965:     case Hexagon::A4_cmpbgtui:      return Hexagon::A4_cmpbgtu;
   966:     case Hexagon::A4_cmpheqi:       return Hexagon::A4_cmpheq;
   967:     case Hexagon::A4_cmphgti:       return Hexagon::A4_cmphgt;
   968:     case Hexagon::A4_cmphgtui:      return Hexagon::A4_cmphgtu;
   969:     case Hexagon::A4_combineii:     return Hexagon::A4_combineir;
   970:     case Hexagon::A4_combineir:     return TargetOpcode::REG_SEQUENCE;
   971:     case Hexagon::A4_combineri:     return TargetOpcode::REG_SEQUENCE;
   972:     case Hexagon::A4_rcmpeqi:       return Hexagon::A4_rcmpeq;
   973:     case Hexagon::A4_rcmpneqi:      return Hexagon::A4_rcmpneq;
   974:     case Hexagon::C2_cmoveif:       return Hexagon::A2_tfrpf;
   975:     case Hexagon::C2_cmoveit:       return Hexagon::A2_tfrpt;
   976:     case Hexagon::C2_cmpeqi:        return Hexagon::C2_cmpeq;
   977:     case Hexagon::C2_cmpgti:        return Hexagon::C2_cmpgt;
   978:     case Hexagon::C2_cmpgtui:       return Hexagon::C2_cmpgtu;
   979:     case Hexagon::C2_muxii:         return Hexagon::C2_muxir;
   980:     case Hexagon::C2_muxir:         return Hexagon::C2_mux;
   981:     case Hexagon::C2_muxri:         return Hexagon::C2_mux;
   982:     case Hexagon::C4_cmpltei:       return Hexagon::C4_cmplte;
   983:     case Hexagon::C4_cmplteui:      return Hexagon::C4_cmplteu;
   984:     case Hexagon::C4_cmpneqi:       return Hexagon::C4_cmpneq;
   985:     case Hexagon::M2_accii:         return Hexagon::M2_acci;        // T -> T
   986:     /* No M2_macsin */
   987:     case Hexagon::M2_macsip:        return Hexagon::M2_maci;        // T -> T
   988:     case Hexagon::M2_mpysin:        return Hexagon::M2_mpyi;
   989:     case Hexagon::M2_mpysip:        return Hexagon::M2_mpyi;
   990:     case Hexagon::M2_mpysmi:        return Hexagon::M2_mpyi;
   991:     case Hexagon::M2_naccii:        return Hexagon::M2_nacci;       // T -> T
   992:     case Hexagon::M4_mpyri_addi:    return Hexagon::M4_mpyri_addr;
   993:     case Hexagon::M4_mpyri_addr:    return Hexagon::M4_mpyrr_addr;  // _ -> T
   994:     case Hexagon::M4_mpyrr_addi:    return Hexagon::M4_mpyrr_addr;  // _ -> T
   995:     case Hexagon::S4_addaddi:       return Hexagon::M2_acci;        // _ -> T
   996:     case Hexagon::S4_addi_asl_ri:   return Hexagon::S2_asl_i_r_acc; // T -> T
   997:     case Hexagon::S4_addi_lsr_ri:   return Hexagon::S2_lsr_i_r_acc; // T -> T
   998:     case Hexagon::S4_andi_asl_ri:   return Hexagon::S2_asl_i_r_and; // T -> T
   999:     case Hexagon::S4_andi_lsr_ri:   return Hexagon::S2_lsr_i_r_and; // T -> T
  1000:     case Hexagon::S4_ori_asl_ri:    return Hexagon::S2_asl_i_r_or;  // T -> T
```
- EN: It opens namespaces (Hexagon) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as HCE::isRegOffOpcode, HCE::getRegOffOpcode, get, HCE::getDirectRegReplacement, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonII, showing how the code connects to sibling backend components.
- CN: 这里打开了命名空间（Hexagon），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 HCE::isRegOffOpcode, HCE::getRegOffOpcode, get, HCE::getDirectRegReplacement 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonII，说明了它与同级后端组件的连接关系。

### Lines 1001-1200 / 第 1001-1200 行

```cpp
  1001:     case Hexagon::S4_ori_lsr_ri:    return Hexagon::S2_lsr_i_r_or;  // T -> T
  1002:     case Hexagon::S4_subaddi:       return Hexagon::M2_subacc;      // _ -> T
  1003:     case Hexagon::S4_subi_asl_ri:   return Hexagon::S2_asl_i_r_nac; // T -> T
  1004:     case Hexagon::S4_subi_lsr_ri:   return Hexagon::S2_lsr_i_r_nac; // T -> T
  1005: 
  1006:     // Store-immediates:
  1007:     case Hexagon::S4_storeirbf_io:  return Hexagon::S2_pstorerbf_io;
  1008:     case Hexagon::S4_storeirb_io:   return Hexagon::S2_storerb_io;
  1009:     case Hexagon::S4_storeirbt_io:  return Hexagon::S2_pstorerbt_io;
  1010:     case Hexagon::S4_storeirhf_io:  return Hexagon::S2_pstorerhf_io;
  1011:     case Hexagon::S4_storeirh_io:   return Hexagon::S2_storerh_io;
  1012:     case Hexagon::S4_storeirht_io:  return Hexagon::S2_pstorerht_io;
  1013:     case Hexagon::S4_storeirif_io:  return Hexagon::S2_pstorerif_io;
  1014:     case Hexagon::S4_storeiri_io:   return Hexagon::S2_storeri_io;
  1015:     case Hexagon::S4_storeirit_io:  return Hexagon::S2_pstorerit_io;
  1016: 
  1017:     default:
  1018:       break;
  1019:   }
  1020:   return 0;
  1021: }
  1022: 
  1023: // Return the allowable deviation from the current value of Rb (i.e. the
  1024: // range of values that can be added to the current value) which the
  1025: // instruction MI can accommodate.
  1026: // The instruction MI is a user of register Rb, which is defined via an
  1027: // extender. It may be possible for MI to be tweaked to work for a register
  1028: // defined with a slightly different value. For example
  1029: //   ... = L2_loadrub_io Rb, 1
  1030: // can be modified to be
  1031: //   ... = L2_loadrub_io Rb', 0
  1032: // if Rb' = Rb+1.
  1033: // The range for Rb would be [Min+1, Max+1], where [Min, Max] is a range
  1034: // for L2_loadrub with offset 0. That means that Rb could be replaced with
  1035: // Rc, where Rc-Rb belongs to [Min+1, Max+1].
  1036: OffsetRange HCE::getOffsetRange(Register Rb, const MachineInstr &MI) const {
  1037:   unsigned Opc = MI.getOpcode();
  1038:   // Instructions that are constant-extended may be replaced with something
  1039:   // else that no longer offers the same range as the original.
  1040:   if (!isRegOffOpcode(Opc) || HII->isConstExtended(MI))
  1041:     return OffsetRange::zero();
  1042: 
  1043:   if (Opc == Hexagon::A2_addi) {
  1044:     const MachineOperand &Op1 = MI.getOperand(1), &Op2 = MI.getOperand(2);
  1045:     if (Rb != Register(Op1) || !Op2.isImm())
  1046:       return OffsetRange::zero();
  1047:     OffsetRange R = { -(1<<15)+1, (1<<15)-1, 1 };
  1048:     return R.shift(Op2.getImm());
  1049:   }
  1050: 
  1051:   // HII::getBaseAndOffsetPosition returns the increment position as "offset".
  1052:   if (HII->isPostIncrement(MI))
  1053:     return OffsetRange::zero();
  1054: 
  1055:   const MCInstrDesc &D = HII->get(Opc);
  1056:   assert(D.mayLoad() || D.mayStore());
  1057: 
  1058:   unsigned BaseP, OffP;
  1059:   if (!HII->getBaseAndOffsetPosition(MI, BaseP, OffP) ||
  1060:       Rb != Register(MI.getOperand(BaseP)) ||
  1061:       !MI.getOperand(OffP).isImm())
  1062:     return OffsetRange::zero();
  1063: 
  1064:   uint64_t F = (D.TSFlags >> HexagonII::MemAccessSizePos) &
  1065:                   HexagonII::MemAccesSizeMask;
  1066:   uint8_t A = HexagonII::getMemAccessSizeInBytes(HexagonII::MemAccessSize(F));
  1067:   unsigned L = Log2_32(A);
  1068:   unsigned S = 10+L;  // sint11_L
  1069:   int32_t Min = -alignDown((1<<S)-1, A);
  1070: 
  1071:   // The range will be shifted by Off. To prefer non-negative offsets,
  1072:   // adjust Max accordingly.
  1073:   int32_t Off = MI.getOperand(OffP).getImm();
  1074:   int32_t Max = Off >= 0 ? 0 : -Off;
  1075: 
  1076:   OffsetRange R = { Min, Max, A };
  1077:   return R.shift(Off);
  1078: }
  1079: 
  1080: // Return the allowable deviation from the current value of the extender ED,
  1081: // for which the instruction corresponding to ED can be modified without
  1082: // using an extender.
  1083: // The instruction uses the extender directly. It will be replaced with
  1084: // another instruction, say MJ, where the extender will be replaced with a
  1085: // register. MJ can allow some variability with respect to the value of
  1086: // that register, as is the case with indexed memory instructions.
  1087: OffsetRange HCE::getOffsetRange(const ExtDesc &ED) const {
  1088:   // The only way that there can be a non-zero range available is if
  1089:   // the instruction using ED will be converted to an indexed memory
  1090:   // instruction.
  1091:   unsigned IdxOpc = getRegOffOpcode(ED.UseMI->getOpcode());
  1092:   switch (IdxOpc) {
  1093:     case 0:
  1094:       return OffsetRange::zero();
  1095:     case Hexagon::A2_addi:    // s16
  1096:       return { -32767, 32767, 1 };
  1097:     case Hexagon::A2_subri:   // s10
  1098:       return { -511, 511, 1 };
  1099:   }
  1100: 
  1101:   if (!ED.UseMI->mayLoad() && !ED.UseMI->mayStore())
  1102:     return OffsetRange::zero();
  1103:   const MCInstrDesc &D = HII->get(IdxOpc);
  1104:   uint64_t F = (D.TSFlags >> HexagonII::MemAccessSizePos) &
  1105:                   HexagonII::MemAccesSizeMask;
  1106:   uint8_t A = HexagonII::getMemAccessSizeInBytes(HexagonII::MemAccessSize(F));
  1107:   unsigned L = Log2_32(A);
  1108:   unsigned S = 10+L;  // sint11_L
  1109:   int32_t Min = -alignDown((1<<S)-1, A);
  1110:   int32_t Max = 0;  // Force non-negative offsets.
  1111:   return { Min, Max, A };
  1112: }
  1113: 
  1114: // Get the allowable deviation from the current value of Rd by checking
  1115: // all uses of Rd.
  1116: OffsetRange HCE::getOffsetRange(Register Rd) const {
  1117:   OffsetRange Range;
  1118:   for (const MachineOperand &Op : MRI->use_operands(Rd.Reg)) {
  1119:     // Make sure that the register being used by this operand is identical
  1120:     // to the register that was defined: using a different subregister
  1121:     // precludes any non-trivial range.
  1122:     if (Rd != Register(Op))
  1123:       return OffsetRange::zero();
  1124:     Range.intersect(getOffsetRange(Rd, *Op.getParent()));
  1125:   }
  1126:   return Range;
  1127: }
  1128: 
  1129: void HCE::recordExtender(MachineInstr &MI, unsigned OpNum) {
  1130:   unsigned Opc = MI.getOpcode();
  1131:   ExtDesc ED;
  1132:   ED.OpNum = OpNum;
  1133: 
  1134:   bool IsLoad = MI.mayLoad();
  1135:   bool IsStore = MI.mayStore();
  1136: 
  1137:   // Fixed stack slots have negative indexes, and they cannot be used
  1138:   // with Register::stackSlotIndex and Register::index2StackSlot. This is
  1139:   // somewhat unfortunate, but should not be a frequent thing.
  1140:   for (MachineOperand &Op : MI.operands())
  1141:     if (Op.isFI() && Op.getIndex() < 0)
  1142:       return;
  1143: 
  1144:   if (IsLoad || IsStore) {
  1145:     unsigned AM = HII->getAddrMode(MI);
  1146:     switch (AM) {
  1147:       // (Re: ##Off + Rb<<S) = Rd: ##Val
  1148:       case HexagonII::Absolute:       // (__: ## + __<<_)
  1149:         break;
  1150:       case HexagonII::AbsoluteSet:    // (Rd: ## + __<<_)
  1151:         ED.Rd = MI.getOperand(OpNum-1);
  1152:         ED.IsDef = true;
  1153:         break;
  1154:       case HexagonII::BaseImmOffset:  // (__: ## + Rs<<0)
  1155:         // Store-immediates are treated as non-memory operations, since
  1156:         // it's the value being stored that is extended (as opposed to
  1157:         // a part of the address).
  1158:         if (!isStoreImmediate(Opc))
  1159:           ED.Expr.Rs = MI.getOperand(OpNum-1);
  1160:         break;
  1161:       case HexagonII::BaseLongOffset: // (__: ## + Rs<<S)
  1162:         ED.Expr.Rs = MI.getOperand(OpNum-2);
  1163:         ED.Expr.S = MI.getOperand(OpNum-1).getImm();
  1164:         break;
  1165:       default:
  1166:         llvm_unreachable("Unhandled memory instruction");
  1167:     }
  1168:   } else {
  1169:     switch (Opc) {
  1170:       case Hexagon::A2_tfrsi:         // (Rd: ## + __<<_)
  1171:         ED.Rd = MI.getOperand(0);
  1172:         ED.IsDef = true;
  1173:         break;
  1174:       case Hexagon::A2_combineii:     // (Rd: ## + __<<_)
  1175:       case Hexagon::A4_combineir:
  1176:         ED.Rd = { MI.getOperand(0).getReg(), Hexagon::isub_hi };
  1177:         ED.IsDef = true;
  1178:         break;
  1179:       case Hexagon::A4_combineri:     // (Rd: ## + __<<_)
  1180:         ED.Rd = { MI.getOperand(0).getReg(), Hexagon::isub_lo };
  1181:         ED.IsDef = true;
  1182:         break;
  1183:       case Hexagon::A2_addi:          // (Rd: ## + Rs<<0)
  1184:         ED.Rd = MI.getOperand(0);
  1185:         ED.Expr.Rs = MI.getOperand(OpNum-1);
  1186:         break;
  1187:       case Hexagon::M2_accii:         // (__: ## + Rs<<0)
  1188:       case Hexagon::M2_naccii:
  1189:       case Hexagon::S4_addaddi:
  1190:         ED.Expr.Rs = MI.getOperand(OpNum-1);
  1191:         break;
  1192:       case Hexagon::A2_subri:         // (Rd: ## - Rs<<0)
  1193:         ED.Rd = MI.getOperand(0);
  1194:         ED.Expr.Rs = MI.getOperand(OpNum+1);
  1195:         ED.Expr.Neg = true;
  1196:         break;
  1197:       case Hexagon::S4_subaddi:       // (__: ## - Rs<<0)
  1198:         ED.Expr.Rs = MI.getOperand(OpNum+1);
  1199:         ED.Expr.Neg = true;
  1200:         break;
```
- EN: It declares or implements routines such as Rb, getOpcode, getOperand, shift, ... (19 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 Rb, getOpcode, getOperand, shift, ... (19 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonII，说明了它与同级后端组件的连接关系。

### Lines 1201-1400 / 第 1201-1400 行

```cpp
  1201:       default:                        // (__: ## + __<<_)
  1202:         break;
  1203:     }
  1204:   }
  1205: 
  1206:   ED.UseMI = &MI;
  1207: 
  1208:   // Ignore unnamed globals.
  1209:   ExtRoot ER(ED.getOp());
  1210:   if (ER.Kind == MachineOperand::MO_GlobalAddress)
  1211:     if (ER.V.GV->getName().empty())
  1212:       return;
  1213:   // Ignore block address that points to block in another function
  1214:   if (ER.Kind == MachineOperand::MO_BlockAddress)
  1215:     if (ER.V.BA->getFunction() != &(MI.getMF()->getFunction()))
  1216:       return;
  1217:   Extenders.push_back(ED);
  1218: }
  1219: 
  1220: void HCE::collectInstr(MachineInstr &MI) {
  1221:   if (!HII->isConstExtended(MI))
  1222:     return;
  1223: 
  1224:   // Skip some non-convertible instructions.
  1225:   unsigned Opc = MI.getOpcode();
  1226:   switch (Opc) {
  1227:     case Hexagon::M2_macsin:  // There is no Rx -= mpyi(Rs,Rt).
  1228:     case Hexagon::C4_addipc:
  1229:     case Hexagon::S4_or_andi:
  1230:     case Hexagon::S4_or_andix:
  1231:     case Hexagon::S4_or_ori:
  1232:       return;
  1233:   }
  1234:   recordExtender(MI, HII->getCExtOpNum(MI));
  1235: }
  1236: 
  1237: void HCE::collect(MachineFunction &MF) {
  1238:   Extenders.clear();
  1239:   for (MachineBasicBlock &MBB : MF) {
  1240:     // Skip unreachable blocks.
  1241:     if (MBB.getNumber() == -1)
  1242:       continue;
  1243:     for (MachineInstr &MI : MBB)
  1244:       collectInstr(MI);
  1245:   }
  1246: }
  1247: 
  1248: void HCE::assignInits(const ExtRoot &ER, unsigned Begin, unsigned End,
  1249:       AssignmentMap &IMap) {
  1250:   // Basic correctness: make sure that all extenders in the range [Begin..End)
  1251:   // share the same root ER.
  1252:   for (unsigned I = Begin; I != End; ++I)
  1253:     assert(ER == ExtRoot(Extenders[I].getOp()));
  1254: 
  1255:   // Construct the list of ranges, such that for each P in Ranges[I],
  1256:   // a register Reg = ER+P can be used in place of Extender[I]. If the
  1257:   // instruction allows, uses in the form of Reg+Off are considered
  1258:   // (here, Off = required_value - P).
  1259:   std::vector<OffsetRange> Ranges(End-Begin);
  1260: 
  1261:   // For each extender that is a def, visit all uses of the defined register,
  1262:   // and produce an offset range that works for all uses. The def doesn't
  1263:   // have to be checked, because it can become dead if all uses can be updated
  1264:   // to use a different reg/offset.
  1265:   for (unsigned I = Begin; I != End; ++I) {
  1266:     const ExtDesc &ED = Extenders[I];
  1267:     if (!ED.IsDef)
  1268:       continue;
  1269:     ExtValue EV(ED);
  1270:     LLVM_DEBUG(dbgs() << " =" << I << ". " << EV << "  " << ED << '\n');
  1271:     assert(ED.Rd.Reg != 0);
  1272:     Ranges[I-Begin] = getOffsetRange(ED.Rd).shift(EV.Offset);
  1273:     // A2_tfrsi is a special case: it will be replaced with A2_addi, which
  1274:     // has a 16-bit signed offset. This means that A2_tfrsi not only has a
  1275:     // range coming from its uses, but also from the fact that its replacement
  1276:     // has a range as well.
  1277:     if (ED.UseMI->getOpcode() == Hexagon::A2_tfrsi) {
  1278:       int32_t D = alignDown(32767, Ranges[I-Begin].Align); // XXX hardcoded
  1279:       Ranges[I-Begin].extendBy(-D).extendBy(D);
  1280:     }
  1281:   }
  1282: 
  1283:   // Visit all non-def extenders. For each one, determine the offset range
  1284:   // available for it.
  1285:   for (unsigned I = Begin; I != End; ++I) {
  1286:     const ExtDesc &ED = Extenders[I];
  1287:     if (ED.IsDef)
  1288:       continue;
  1289:     ExtValue EV(ED);
  1290:     LLVM_DEBUG(dbgs() << "  " << I << ". " << EV << "  " << ED << '\n');
  1291:     OffsetRange Dev = getOffsetRange(ED);
  1292:     Ranges[I-Begin].intersect(Dev.shift(EV.Offset));
  1293:   }
  1294: 
  1295:   // Here for each I there is a corresponding Range[I]. Construct the
  1296:   // inverse map, that to each range will assign the set of indexes in
  1297:   // [Begin..End) that this range corresponds to.
  1298:   std::map<OffsetRange, IndexList> RangeMap;
  1299:   for (unsigned I = Begin; I != End; ++I)
  1300:     RangeMap[Ranges[I-Begin]].insert(I);
  1301: 
  1302:   LLVM_DEBUG({
  1303:     dbgs() << "Ranges\n";
  1304:     for (unsigned I = Begin; I != End; ++I)
  1305:       dbgs() << "  " << I << ". " << Ranges[I-Begin] << '\n';
  1306:     dbgs() << "RangeMap\n";
  1307:     for (auto &P : RangeMap) {
  1308:       dbgs() << "  " << P.first << " ->";
  1309:       for (unsigned I : P.second)
  1310:         dbgs() << ' ' << I;
  1311:       dbgs() << '\n';
  1312:     }
  1313:   });
  1314: 
  1315:   // Select the definition points, and generate the assignment between
  1316:   // these points and the uses.
  1317: 
  1318:   RangeTree Tree;
  1319:   for (const OffsetRange &R : Ranges)
  1320:     Tree.add(R);
  1321:   SmallVector<RangeTree::Node*,8> Nodes;
  1322:   Tree.order(Nodes);
  1323: 
  1324:   auto MaxAlign = [](const SmallVectorImpl<RangeTree::Node*> &Nodes,
  1325:                      uint8_t Align, uint8_t Offset) {
  1326:     for (RangeTree::Node *N : Nodes) {
  1327:       if (N->Range.Align <= Align || N->Range.Offset < Offset)
  1328:         continue;
  1329:       if ((N->Range.Offset - Offset) % Align != 0)
  1330:         continue;
  1331:       Align = N->Range.Align;
  1332:       Offset = N->Range.Offset;
  1333:     }
  1334:     return std::make_pair(Align, Offset);
  1335:   };
  1336: 
  1337:   // Construct the set of all potential definition points from the endpoints
  1338:   // of the ranges. If a given endpoint also belongs to a different range,
  1339:   // but with a higher alignment, also consider the more-highly-aligned
  1340:   // value of this endpoint.
  1341:   std::set<int32_t> CandSet;
  1342:   for (RangeTree::Node *N : Nodes) {
  1343:     const OffsetRange &R = N->Range;
  1344:     auto P0 = MaxAlign(Tree.nodesWith(R.Min, false), R.Align, R.Offset);
  1345:     CandSet.insert(R.Min);
  1346:     if (R.Align < P0.first)
  1347:       CandSet.insert(adjustUp(R.Min, P0.first, P0.second));
  1348:     auto P1 = MaxAlign(Tree.nodesWith(R.Max, false), R.Align, R.Offset);
  1349:     CandSet.insert(R.Max);
  1350:     if (R.Align < P1.first)
  1351:       CandSet.insert(adjustDown(R.Max, P1.first, P1.second));
  1352:   }
  1353: 
  1354:   // Build the assignment map: candidate C -> { list of extender indexes }.
  1355:   // This has to be done iteratively:
  1356:   // - pick the candidate that covers the maximum number of extenders,
  1357:   // - add the candidate to the map,
  1358:   // - remove the extenders from the pool.
  1359:   while (true) {
  1360:     using CMap = std::map<int32_t,unsigned>;
  1361:     CMap Counts;
  1362:     for (auto It = CandSet.begin(), Et = CandSet.end(); It != Et; ) {
  1363:       auto &&V = Tree.nodesWith(*It);
  1364:       unsigned N = std::accumulate(V.begin(), V.end(), 0u,
  1365:                     [](unsigned Acc, const RangeTree::Node *N) {
  1366:                       return Acc + N->Count;
  1367:                     });
  1368:       if (N != 0)
  1369:         Counts.insert({*It, N});
  1370:       It = (N != 0) ? std::next(It) : CandSet.erase(It);
  1371:     }
  1372:     if (Counts.empty())
  1373:       break;
  1374: 
  1375:     // Find the best candidate with respect to the number of extenders covered.
  1376:     auto BestIt = llvm::max_element(
  1377:         Counts, [](const CMap::value_type &A, const CMap::value_type &B) {
  1378:           return A.second < B.second || (A.second == B.second && A < B);
  1379:         });
  1380:     int32_t Best = BestIt->first;
  1381:     ExtValue BestV(ER, Best);
  1382:     for (RangeTree::Node *N : Tree.nodesWith(Best)) {
  1383:       for (unsigned I : RangeMap[N->Range])
  1384:         IMap[{BestV,Extenders[I].Expr}].insert(I);
  1385:       Tree.erase(N);
  1386:     }
  1387:   }
  1388: 
  1389:   LLVM_DEBUG(dbgs() << "IMap (before fixup) = " << PrintIMap(IMap, *HRI));
  1390: 
  1391:   // There is some ambiguity in what initializer should be used, if the
  1392:   // descriptor's subexpression is non-trivial: it can be the entire
  1393:   // subexpression (which is what has been done so far), or it can be
  1394:   // the extender's value itself, if all corresponding extenders have the
  1395:   // exact value of the initializer (i.e. require offset of 0).
  1396: 
  1397:   // To reduce the number of initializers, merge such special cases.
  1398:   for (std::pair<const ExtenderInit,IndexList> &P : IMap) {
  1399:     // Skip trivial initializers.
  1400:     if (P.first.second.trivial())
```
- EN: It declares or implements routines such as ER, push_back, HCE::collectInstr, getOpcode, ... (27 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 ER, push_back, HCE::collectInstr, getOpcode, ... (27 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 1401-1600 / 第 1401-1600 行

```cpp
  1401:       continue;
  1402:     // If the corresponding trivial initializer does not exist, skip this
  1403:     // entry.
  1404:     const ExtValue &EV = P.first.first;
  1405:     AssignmentMap::iterator F = IMap.find({EV, ExtExpr()});
  1406:     if (F == IMap.end())
  1407:       continue;
  1408: 
  1409:     // Finally, check if all extenders have the same value as the initializer.
  1410:     // Make sure that extenders that are a part of a stack address are not
  1411:     // merged with those that aren't. Stack addresses need an offset field
  1412:     // (to be used by frame index elimination), while non-stack expressions
  1413:     // can be replaced with forms (such as rr) that do not have such a field.
  1414:     // Example:
  1415:     //
  1416:     // Collected 3 extenders
  1417:     //  =2. imm:0  off:32968  bb#2: %7 = ## + __ << 0, def
  1418:     //   0. imm:0  off:267  bb#0: __ = ## + SS#1 << 0
  1419:     //   1. imm:0  off:267  bb#1: __ = ## + SS#1 << 0
  1420:     // Ranges
  1421:     //   0. [-756,267]a1+0
  1422:     //   1. [-756,267]a1+0
  1423:     //   2. [201,65735]a1+0
  1424:     // RangeMap
  1425:     //   [-756,267]a1+0 -> 0 1
  1426:     //   [201,65735]a1+0 -> 2
  1427:     // IMap (before fixup) = {
  1428:     //   [imm:0  off:267, ## + __ << 0] -> { 2 }
  1429:     //   [imm:0  off:267, ## + SS#1 << 0] -> { 0 1 }
  1430:     // }
  1431:     // IMap (after fixup) = {
  1432:     //   [imm:0  off:267, ## + __ << 0] -> { 2 0 1 }
  1433:     //   [imm:0  off:267, ## + SS#1 << 0] -> { }
  1434:     // }
  1435:     // Inserted def in bb#0 for initializer: [imm:0  off:267, ## + __ << 0]
  1436:     //   %12:intregs = A2_tfrsi 267
  1437:     //
  1438:     // The result was
  1439:     //   %12:intregs = A2_tfrsi 267
  1440:     //   S4_pstorerbt_rr %3, %12, %stack.1, 0, killed %4
  1441:     // Which became
  1442:     //   r0 = #267
  1443:     //   if (p0.new) memb(r0+r29<<#4) = r2
  1444: 
  1445:     bool IsStack = any_of(F->second, [this](unsigned I) {
  1446:                       return Extenders[I].Expr.Rs.isSlot();
  1447:                    });
  1448:     auto SameValue = [&EV,this,IsStack](unsigned I) {
  1449:       const ExtDesc &ED = Extenders[I];
  1450:       return ED.Expr.Rs.isSlot() == IsStack &&
  1451:              ExtValue(ED).Offset == EV.Offset;
  1452:     };
  1453:     if (all_of(P.second, SameValue)) {
  1454:       F->second.insert_range(P.second);
  1455:       P.second.clear();
  1456:     }
  1457:   }
  1458: 
  1459:   LLVM_DEBUG(dbgs() << "IMap (after fixup) = " << PrintIMap(IMap, *HRI));
  1460: }
  1461: 
  1462: void HCE::calculatePlacement(const ExtenderInit &ExtI, const IndexList &Refs,
  1463:       LocDefList &Defs) {
  1464:   if (Refs.empty())
  1465:     return;
  1466: 
  1467:   // The placement calculation is somewhat simple right now: it finds a
  1468:   // single location for the def that dominates all refs. Since this may
  1469:   // place the def far from the uses, producing several locations for
  1470:   // defs that collectively dominate all refs could be better.
  1471:   // For now only do the single one.
  1472:   DenseSet<MachineBasicBlock*> Blocks;
  1473:   DenseSet<MachineInstr*> RefMIs;
  1474:   const ExtDesc &ED0 = Extenders[Refs[0]];
  1475:   MachineBasicBlock *DomB = ED0.UseMI->getParent();
  1476:   RefMIs.insert(ED0.UseMI);
  1477:   Blocks.insert(DomB);
  1478:   for (unsigned i = 1, e = Refs.size(); i != e; ++i) {
  1479:     const ExtDesc &ED = Extenders[Refs[i]];
  1480:     MachineBasicBlock *MBB = ED.UseMI->getParent();
  1481:     RefMIs.insert(ED.UseMI);
  1482:     DomB = MDT->findNearestCommonDominator(DomB, MBB);
  1483:     Blocks.insert(MBB);
  1484:   }
  1485: 
  1486: #ifndef NDEBUG
  1487:   // The block DomB should be dominated by the def of each register used
  1488:   // in the initializer.
  1489:   Register Rs = ExtI.second.Rs;  // Only one reg allowed now.
  1490:   const MachineInstr *DefI = Rs.isVReg() ? MRI->getVRegDef(Rs.Reg) : nullptr;
  1491: 
  1492:   // This should be guaranteed given that the entire expression is used
  1493:   // at each instruction in Refs. Add an assertion just in case.
  1494:   assert(!DefI || MDT->dominates(DefI->getParent(), DomB));
  1495: #endif
  1496: 
  1497:   MachineBasicBlock::iterator It;
  1498:   if (Blocks.count(DomB)) {
  1499:     // Try to find the latest possible location for the def.
  1500:     MachineBasicBlock::iterator End = DomB->end();
  1501:     for (It = DomB->begin(); It != End; ++It)
  1502:       if (RefMIs.count(&*It))
  1503:         break;
  1504:     assert(It != End && "Should have found a ref in DomB");
  1505:   } else {
  1506:     // DomB does not contain any refs.
  1507:     It = DomB->getFirstTerminator();
  1508:   }
  1509:   Loc DefLoc(DomB, It);
  1510:   Defs.emplace_back(DefLoc, Refs);
  1511: }
  1512: 
  1513: HCE::Register HCE::insertInitializer(Loc DefL, const ExtenderInit &ExtI) {
  1514:   llvm::Register DefR = MRI->createVirtualRegister(&Hexagon::IntRegsRegClass);
  1515:   MachineBasicBlock &MBB = *DefL.Block;
  1516:   MachineBasicBlock::iterator At = DefL.At;
  1517:   DebugLoc dl = DefL.Block->findDebugLoc(DefL.At);
  1518:   const ExtValue &EV = ExtI.first;
  1519:   MachineOperand ExtOp(EV);
  1520: 
  1521:   const ExtExpr &Ex = ExtI.second;
  1522:   const MachineInstr *InitI = nullptr;
  1523: 
  1524:   if (Ex.Rs.isSlot()) {
  1525:     assert(Ex.S == 0 && "Cannot have a shift of a stack slot");
  1526:     assert(!Ex.Neg && "Cannot subtract a stack slot");
  1527:     // DefR = PS_fi Rb,##EV
  1528:     InitI = BuildMI(MBB, At, dl, HII->get(Hexagon::PS_fi), DefR)
  1529:               .add(MachineOperand(Ex.Rs))
  1530:               .add(ExtOp);
  1531:   } else {
  1532:     assert((Ex.Rs.Reg == 0 || Ex.Rs.isVReg()) && "Expecting virtual register");
  1533:     if (Ex.trivial()) {
  1534:       // DefR = ##EV
  1535:       InitI = BuildMI(MBB, At, dl, HII->get(Hexagon::A2_tfrsi), DefR)
  1536:                 .add(ExtOp);
  1537:     } else if (Ex.S == 0) {
  1538:       if (Ex.Neg) {
  1539:         // DefR = sub(##EV,Rb)
  1540:         InitI = BuildMI(MBB, At, dl, HII->get(Hexagon::A2_subri), DefR)
  1541:                   .add(ExtOp)
  1542:                   .add(MachineOperand(Ex.Rs));
  1543:       } else {
  1544:         // DefR = add(Rb,##EV)
  1545:         InitI = BuildMI(MBB, At, dl, HII->get(Hexagon::A2_addi), DefR)
  1546:                   .add(MachineOperand(Ex.Rs))
  1547:                   .add(ExtOp);
  1548:       }
  1549:     } else {
  1550:       if (HST->useCompound()) {
  1551:         unsigned NewOpc = Ex.Neg ? Hexagon::S4_subi_asl_ri
  1552:                                  : Hexagon::S4_addi_asl_ri;
  1553:         // DefR = add(##EV,asl(Rb,S))
  1554:         InitI = BuildMI(MBB, At, dl, HII->get(NewOpc), DefR)
  1555:                   .add(ExtOp)
  1556:                   .add(MachineOperand(Ex.Rs))
  1557:                   .addImm(Ex.S);
  1558:       } else {
  1559:         // No compounds are available. It is not clear whether we should
  1560:         // even process such extenders where the initializer cannot be
  1561:         // a single instruction, but do it for now.
  1562:         llvm::Register TmpR = MRI->createVirtualRegister(&Hexagon::IntRegsRegClass);
  1563:         BuildMI(MBB, At, dl, HII->get(Hexagon::S2_asl_i_r), TmpR)
  1564:           .add(MachineOperand(Ex.Rs))
  1565:           .addImm(Ex.S);
  1566:         if (Ex.Neg)
  1567:           InitI = BuildMI(MBB, At, dl, HII->get(Hexagon::A2_subri), DefR)
  1568:                     .add(ExtOp)
  1569:                     .add(MachineOperand(Register(TmpR, 0)));
  1570:         else
  1571:           InitI = BuildMI(MBB, At, dl, HII->get(Hexagon::A2_addi), DefR)
  1572:                     .add(MachineOperand(Register(TmpR, 0)))
  1573:                     .add(ExtOp);
  1574:       }
  1575:     }
  1576:   }
  1577: 
  1578:   assert(InitI);
  1579:   (void)InitI;
  1580:   LLVM_DEBUG(dbgs() << "Inserted def in bb#" << MBB.getNumber()
  1581:                     << " for initializer: " << PrintInit(ExtI, *HRI) << "\n  "
  1582:                     << *InitI);
  1583:   return { DefR, 0 };
  1584: }
  1585: 
  1586: // Replace the extender at index Idx with the register ExtR.
  1587: bool HCE::replaceInstrExact(const ExtDesc &ED, Register ExtR) {
  1588:   MachineInstr &MI = *ED.UseMI;
  1589:   MachineBasicBlock &MBB = *MI.getParent();
  1590:   MachineBasicBlock::iterator At = MI.getIterator();
  1591:   DebugLoc dl = MI.getDebugLoc();
  1592:   unsigned ExtOpc = MI.getOpcode();
  1593: 
  1594:   // With a few exceptions, direct replacement amounts to creating an
  1595:   // instruction with a corresponding register opcode, with all operands
  1596:   // the same, except for the register used in place of the extender.
  1597:   unsigned RegOpc = getDirectRegReplacement(ExtOpc);
  1598: 
  1599:   if (RegOpc == TargetOpcode::REG_SEQUENCE) {
  1600:     if (ExtOpc == Hexagon::A4_combineri)
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. It declares or implements routines such as isSlot, insert_range, clear, LLVM_DEBUG, ... (25 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里声明或实现了 isSlot, insert_range, clear, LLVM_DEBUG, ... (25 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 1601-1800 / 第 1601-1800 行

```cpp
  1601:       BuildMI(MBB, At, dl, HII->get(RegOpc))
  1602:         .add(MI.getOperand(0))
  1603:         .add(MI.getOperand(1))
  1604:         .addImm(Hexagon::isub_hi)
  1605:         .add(MachineOperand(ExtR))
  1606:         .addImm(Hexagon::isub_lo);
  1607:     else if (ExtOpc == Hexagon::A4_combineir)
  1608:       BuildMI(MBB, At, dl, HII->get(RegOpc))
  1609:         .add(MI.getOperand(0))
  1610:         .add(MachineOperand(ExtR))
  1611:         .addImm(Hexagon::isub_hi)
  1612:         .add(MI.getOperand(2))
  1613:         .addImm(Hexagon::isub_lo);
  1614:     else
  1615:       llvm_unreachable("Unexpected opcode became REG_SEQUENCE");
  1616:     MBB.erase(MI);
  1617:     return true;
  1618:   }
  1619:   if (ExtOpc == Hexagon::C2_cmpgei || ExtOpc == Hexagon::C2_cmpgeui) {
  1620:     unsigned NewOpc = ExtOpc == Hexagon::C2_cmpgei ? Hexagon::C2_cmplt
  1621:                                                    : Hexagon::C2_cmpltu;
  1622:     BuildMI(MBB, At, dl, HII->get(NewOpc))
  1623:       .add(MI.getOperand(0))
  1624:       .add(MachineOperand(ExtR))
  1625:       .add(MI.getOperand(1));
  1626:     MBB.erase(MI);
  1627:     return true;
  1628:   }
  1629: 
  1630:   if (RegOpc != 0) {
  1631:     MachineInstrBuilder MIB = BuildMI(MBB, At, dl, HII->get(RegOpc));
  1632:     unsigned RegN = ED.OpNum;
  1633:     // Copy all operands except the one that has the extender.
  1634:     for (unsigned i = 0, e = MI.getNumOperands(); i != e; ++i) {
  1635:       if (i != RegN)
  1636:         MIB.add(MI.getOperand(i));
  1637:       else
  1638:         MIB.add(MachineOperand(ExtR));
  1639:     }
  1640:     MIB.cloneMemRefs(MI);
  1641:     MBB.erase(MI);
  1642:     return true;
  1643:   }
  1644: 
  1645:   if (MI.mayLoadOrStore() && !isStoreImmediate(ExtOpc)) {
  1646:     // For memory instructions, there is an asymmetry in the addressing
  1647:     // modes. Addressing modes allowing extenders can be replaced with
  1648:     // addressing modes that use registers, but the order of operands
  1649:     // (or even their number) may be different.
  1650:     // Replacements:
  1651:     //   BaseImmOffset (io)  -> BaseRegOffset (rr)
  1652:     //   BaseLongOffset (ur) -> BaseRegOffset (rr)
  1653:     unsigned RegOpc, Shift;
  1654:     unsigned AM = HII->getAddrMode(MI);
  1655:     if (AM == HexagonII::BaseImmOffset) {
  1656:       RegOpc = HII->changeAddrMode_io_rr(ExtOpc);
  1657:       Shift = 0;
  1658:     } else if (AM == HexagonII::BaseLongOffset) {
  1659:       // Loads:  Rd = L4_loadri_ur Rs, S, ##
  1660:       // Stores: S4_storeri_ur Rs, S, ##, Rt
  1661:       RegOpc = HII->changeAddrMode_ur_rr(ExtOpc);
  1662:       Shift = MI.getOperand(MI.mayLoad() ? 2 : 1).getImm();
  1663:     } else {
  1664:       llvm_unreachable("Unexpected addressing mode");
  1665:     }
  1666: #ifndef NDEBUG
  1667:     if (RegOpc == -1u) {
  1668:       dbgs() << "\nExtOpc: " << HII->getName(ExtOpc) << " has no rr version\n";
  1669:       llvm_unreachable("No corresponding rr instruction");
  1670:     }
  1671: #endif
  1672: 
  1673:     unsigned BaseP, OffP;
  1674:     HII->getBaseAndOffsetPosition(MI, BaseP, OffP);
  1675: 
  1676:     // Build an rr instruction: (RegOff + RegBase<<0)
  1677:     MachineInstrBuilder MIB = BuildMI(MBB, At, dl, HII->get(RegOpc));
  1678:     // First, add the def for loads.
  1679:     if (MI.mayLoad())
  1680:       MIB.add(getLoadResultOp(MI));
  1681:     // Handle possible predication.
  1682:     if (HII->isPredicated(MI))
  1683:       MIB.add(getPredicateOp(MI));
  1684:     // Build the address.
  1685:     MIB.add(MachineOperand(ExtR));      // RegOff
  1686:     MIB.add(MI.getOperand(BaseP));      // RegBase
  1687:     MIB.addImm(Shift);                  // << Shift
  1688:     // Add the stored value for stores.
  1689:     if (MI.mayStore())
  1690:       MIB.add(getStoredValueOp(MI));
  1691:     MIB.cloneMemRefs(MI);
  1692:     MBB.erase(MI);
  1693:     return true;
  1694:   }
  1695: 
  1696: #ifndef NDEBUG
  1697:   dbgs() << '\n' << MI;
  1698: #endif
  1699:   llvm_unreachable("Unhandled exact replacement");
  1700:   return false;
  1701: }
  1702: 
  1703: // Replace the extender ED with a form corresponding to the initializer ExtI.
  1704: bool HCE::replaceInstrExpr(const ExtDesc &ED, const ExtenderInit &ExtI,
  1705:       Register ExtR, int32_t &Diff) {
  1706:   MachineInstr &MI = *ED.UseMI;
  1707:   MachineBasicBlock &MBB = *MI.getParent();
  1708:   MachineBasicBlock::iterator At = MI.getIterator();
  1709:   DebugLoc dl = MI.getDebugLoc();
  1710:   unsigned ExtOpc = MI.getOpcode();
  1711: 
  1712:   if (ExtOpc == Hexagon::A2_tfrsi) {
  1713:     // A2_tfrsi is a special case: it's replaced with A2_addi, which introduces
  1714:     // another range. One range is the one that's common to all tfrsi's uses,
  1715:     // this one is the range of immediates in A2_addi. When calculating ranges,
  1716:     // the addi's 16-bit argument was included, so now we need to make it such
  1717:     // that the produced value is in the range for the uses alone.
  1718:     // Most of the time, simply adding Diff will make the addi produce exact
  1719:     // result, but if Diff is outside of the 16-bit range, some adjustment
  1720:     // will be needed.
  1721:     unsigned IdxOpc = getRegOffOpcode(ExtOpc);
  1722:     assert(IdxOpc == Hexagon::A2_addi);
  1723: 
  1724:     // Clamp Diff to the 16 bit range.
  1725:     int32_t D = isInt<16>(Diff) ? Diff : (Diff > 0 ? 32767 : -32768);
  1726:     if (Diff > 32767) {
  1727:       // Split Diff into two values: one that is close to min/max int16,
  1728:       // and the other being the rest, and such that both have the same
  1729:       // "alignment" as Diff.
  1730:       uint32_t UD = Diff;
  1731:       OffsetRange R = getOffsetRange(MI.getOperand(0));
  1732:       uint32_t A = std::min<uint32_t>(R.Align, 1u << llvm::countr_zero(UD));
  1733:       D &= ~(A-1);
  1734:     }
  1735:     BuildMI(MBB, At, dl, HII->get(IdxOpc))
  1736:       .add(MI.getOperand(0))
  1737:       .add(MachineOperand(ExtR))
  1738:       .addImm(D);
  1739:     Diff -= D;
  1740: #ifndef NDEBUG
  1741:     // Make sure the output is within allowable range for uses.
  1742:     // "Diff" is a difference in the "opposite direction", i.e. Ext - DefV,
  1743:     // not DefV - Ext, as the getOffsetRange would calculate.
  1744:     OffsetRange Uses = getOffsetRange(MI.getOperand(0));
  1745:     if (!Uses.contains(-Diff))
  1746:       dbgs() << "Diff: " << -Diff << " out of range " << Uses
  1747:              << " for " << MI;
  1748:     assert(Uses.contains(-Diff));
  1749: #endif
  1750:     MBB.erase(MI);
  1751:     return true;
  1752:   }
  1753: 
  1754:   const ExtValue &EV = ExtI.first; (void)EV;
  1755:   const ExtExpr &Ex = ExtI.second; (void)Ex;
  1756: 
  1757:   if (ExtOpc == Hexagon::A2_addi || ExtOpc == Hexagon::A2_subri) {
  1758:     // If addi/subri are replaced with the exactly matching initializer,
  1759:     // they amount to COPY.
  1760:     // Check that the initializer is an exact match (for simplicity).
  1761: #ifndef NDEBUG
  1762:     bool IsAddi = ExtOpc == Hexagon::A2_addi;
  1763:     const MachineOperand &RegOp = MI.getOperand(IsAddi ? 1 : 2);
  1764:     const MachineOperand &ImmOp = MI.getOperand(IsAddi ? 2 : 1);
  1765:     assert(Ex.Rs == RegOp && EV == ImmOp && Ex.Neg != IsAddi &&
  1766:            "Initializer mismatch");
  1767: #endif
  1768:     BuildMI(MBB, At, dl, HII->get(TargetOpcode::COPY))
  1769:       .add(MI.getOperand(0))
  1770:       .add(MachineOperand(ExtR));
  1771:     Diff = 0;
  1772:     MBB.erase(MI);
  1773:     return true;
  1774:   }
  1775:   if (ExtOpc == Hexagon::M2_accii || ExtOpc == Hexagon::M2_naccii ||
  1776:       ExtOpc == Hexagon::S4_addaddi || ExtOpc == Hexagon::S4_subaddi) {
  1777:     // M2_accii:    add(Rt,add(Rs,V)) (tied)
  1778:     // M2_naccii:   sub(Rt,add(Rs,V))
  1779:     // S4_addaddi:  add(Rt,add(Rs,V))
  1780:     // S4_subaddi:  add(Rt,sub(V,Rs))
  1781:     // Check that Rs and V match the initializer expression. The Rs+V is the
  1782:     // combination that is considered "subexpression" for V, although Rx+V
  1783:     // would also be valid.
  1784: #ifndef NDEBUG
  1785:     bool IsSub = ExtOpc == Hexagon::S4_subaddi;
  1786:     Register Rs = MI.getOperand(IsSub ? 3 : 2);
  1787:     ExtValue V = MI.getOperand(IsSub ? 2 : 3);
  1788:     assert(EV == V && Rs == Ex.Rs && IsSub == Ex.Neg && "Initializer mismatch");
  1789: #endif
  1790:     unsigned NewOpc = ExtOpc == Hexagon::M2_naccii ? Hexagon::A2_sub
  1791:                                                    : Hexagon::A2_add;
  1792:     BuildMI(MBB, At, dl, HII->get(NewOpc))
  1793:       .add(MI.getOperand(0))
  1794:       .add(MI.getOperand(1))
  1795:       .add(MachineOperand(ExtR));
  1796:     MBB.erase(MI);
  1797:     return true;
  1798:   }
  1799: 
  1800:   if (MI.mayLoadOrStore()) {
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. It declares or implements routines such as BuildMI, llvm_unreachable, erase, add, ... (22 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里声明或实现了 BuildMI, llvm_unreachable, erase, add, ... (22 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 1801-2000 / 第 1801-2000 行

```cpp
  1801:     unsigned IdxOpc = getRegOffOpcode(ExtOpc);
  1802:     assert(IdxOpc && "Expecting indexed opcode");
  1803:     MachineInstrBuilder MIB = BuildMI(MBB, At, dl, HII->get(IdxOpc));
  1804:     // Construct the new indexed instruction.
  1805:     // First, add the def for loads.
  1806:     if (MI.mayLoad())
  1807:       MIB.add(getLoadResultOp(MI));
  1808:     // Handle possible predication.
  1809:     if (HII->isPredicated(MI))
  1810:       MIB.add(getPredicateOp(MI));
  1811:     // Build the address.
  1812:     MIB.add(MachineOperand(ExtR));
  1813:     MIB.addImm(Diff);
  1814:     // Add the stored value for stores.
  1815:     if (MI.mayStore())
  1816:       MIB.add(getStoredValueOp(MI));
  1817:     MIB.cloneMemRefs(MI);
  1818:     MBB.erase(MI);
  1819:     return true;
  1820:   }
  1821: 
  1822: #ifndef NDEBUG
  1823:   dbgs() << '\n' << PrintInit(ExtI, *HRI) << "  " << MI;
  1824: #endif
  1825:   llvm_unreachable("Unhandled expr replacement");
  1826:   return false;
  1827: }
  1828: 
  1829: bool HCE::replaceInstr(unsigned Idx, Register ExtR, const ExtenderInit &ExtI) {
  1830:   if (ReplaceLimit.getNumOccurrences()) {
  1831:     if (ReplaceLimit <= ReplaceCounter)
  1832:       return false;
  1833:     ++ReplaceCounter;
  1834:   }
  1835:   const ExtDesc &ED = Extenders[Idx];
  1836:   assert((!ED.IsDef || ED.Rd.Reg != 0) && "Missing Rd for def");
  1837:   const ExtValue &DefV = ExtI.first;
  1838:   assert(ExtRoot(ExtValue(ED)) == ExtRoot(DefV) && "Extender root mismatch");
  1839:   const ExtExpr &DefEx = ExtI.second;
  1840: 
  1841:   ExtValue EV(ED);
  1842:   int32_t Diff = EV.Offset - DefV.Offset;
  1843:   const MachineInstr &MI = *ED.UseMI;
  1844:   LLVM_DEBUG(dbgs() << __func__ << " Idx:" << Idx << " ExtR:"
  1845:                     << PrintRegister(ExtR, *HRI) << " Diff:" << Diff << '\n');
  1846: 
  1847:   // These two addressing modes must be converted into indexed forms
  1848:   // regardless of what the initializer looks like.
  1849:   bool IsAbs = false, IsAbsSet = false;
  1850:   if (MI.mayLoadOrStore()) {
  1851:     unsigned AM = HII->getAddrMode(MI);
  1852:     IsAbs = AM == HexagonII::Absolute;
  1853:     IsAbsSet = AM == HexagonII::AbsoluteSet;
  1854:   }
  1855: 
  1856:   // If it's a def, remember all operands that need to be updated.
  1857:   // If ED is a def, and Diff is not 0, then all uses of the register Rd
  1858:   // defined by ED must be in the form (Rd, imm), i.e. the immediate offset
  1859:   // must follow the Rd in the operand list.
  1860:   std::vector<std::pair<MachineInstr*,unsigned>> RegOps;
  1861:   if (ED.IsDef && Diff != 0) {
  1862:     for (MachineOperand &Op : MRI->use_operands(ED.Rd.Reg)) {
  1863:       MachineInstr &UI = *Op.getParent();
  1864:       RegOps.push_back({&UI, getOperandIndex(UI, Op)});
  1865:     }
  1866:   }
  1867: 
  1868:   // Replace the instruction.
  1869:   bool Replaced = false;
  1870:   if (Diff == 0 && DefEx.trivial() && !IsAbs && !IsAbsSet)
  1871:     Replaced = replaceInstrExact(ED, ExtR);
  1872:   else
  1873:     Replaced = replaceInstrExpr(ED, ExtI, ExtR, Diff);
  1874: 
  1875:   if (Diff != 0 && Replaced && ED.IsDef) {
  1876:     // Update offsets of the def's uses.
  1877:     for (std::pair<MachineInstr*,unsigned> P : RegOps) {
  1878:       unsigned J = P.second;
  1879:       assert(P.first->getNumOperands() > J+1 &&
  1880:              P.first->getOperand(J+1).isImm());
  1881:       MachineOperand &ImmOp = P.first->getOperand(J+1);
  1882:       ImmOp.setImm(ImmOp.getImm() + Diff);
  1883:     }
  1884:     // If it was an absolute-set instruction, the "set" part has been removed.
  1885:     // ExtR will now be the register with the extended value, and since all
  1886:     // users of Rd have been updated, all that needs to be done is to replace
  1887:     // Rd with ExtR.
  1888:     if (IsAbsSet) {
  1889:       assert(ED.Rd.Sub == 0 && ExtR.Sub == 0);
  1890:       MRI->replaceRegWith(ED.Rd.Reg, ExtR.Reg);
  1891:     }
  1892:   }
  1893: 
  1894:   return Replaced;
  1895: }
  1896: 
  1897: bool HCE::replaceExtenders(const AssignmentMap &IMap) {
  1898:   LocDefList Defs;
  1899:   bool Changed = false;
  1900: 
  1901:   for (const std::pair<const ExtenderInit, IndexList> &P : IMap) {
  1902:     const IndexList &Idxs = P.second;
  1903:     if (Idxs.size() < CountThreshold)
  1904:       continue;
  1905: 
  1906:     Defs.clear();
  1907:     calculatePlacement(P.first, Idxs, Defs);
  1908:     for (const std::pair<Loc,IndexList> &Q : Defs) {
  1909:       Register DefR = insertInitializer(Q.first, P.first);
  1910:       NewRegs.push_back(DefR.Reg);
  1911:       for (unsigned I : Q.second)
  1912:         Changed |= replaceInstr(I, DefR, P.first);
  1913:     }
  1914:   }
  1915:   return Changed;
  1916: }
  1917: 
  1918: unsigned HCE::getOperandIndex(const MachineInstr &MI,
  1919:       const MachineOperand &Op) const {
  1920:   for (unsigned i = 0, n = MI.getNumOperands(); i != n; ++i)
  1921:     if (&MI.getOperand(i) == &Op)
  1922:       return i;
  1923:   llvm_unreachable("Not an operand of MI");
  1924: }
  1925: 
  1926: const MachineOperand &HCE::getPredicateOp(const MachineInstr &MI) const {
  1927:   assert(HII->isPredicated(MI));
  1928:   for (const MachineOperand &Op : MI.operands()) {
  1929:     if (!Op.isReg() || !Op.isUse() ||
  1930:         MRI->getRegClass(Op.getReg()) != &Hexagon::PredRegsRegClass)
  1931:       continue;
  1932:     assert(Op.getSubReg() == 0 && "Predicate register with a subregister");
  1933:     return Op;
  1934:   }
  1935:   llvm_unreachable("Predicate operand not found");
  1936: }
  1937: 
  1938: const MachineOperand &HCE::getLoadResultOp(const MachineInstr &MI) const {
  1939:   assert(MI.mayLoad());
  1940:   return MI.getOperand(0);
  1941: }
  1942: 
  1943: const MachineOperand &HCE::getStoredValueOp(const MachineInstr &MI) const {
  1944:   assert(MI.mayStore());
  1945:   return MI.getOperand(MI.getNumExplicitOperands()-1);
  1946: }
  1947: 
  1948: bool HCE::runOnMachineFunction(MachineFunction &MF) {
  1949:   if (skipFunction(MF.getFunction()))
  1950:     return false;
  1951:   if (MF.getFunction().hasPersonalityFn()) {
  1952:     LLVM_DEBUG(dbgs() << getPassName() << ": skipping " << MF.getName()
  1953:                       << " due to exception handling\n");
  1954:     return false;
  1955:   }
  1956:   LLVM_DEBUG(MF.print(dbgs() << "Before " << getPassName() << '\n', nullptr));
  1957: 
  1958:   HST = &MF.getSubtarget<HexagonSubtarget>();
  1959:   HII = HST->getInstrInfo();
  1960:   HRI = HST->getRegisterInfo();
  1961:   MDT = &getAnalysis<MachineDominatorTreeWrapperPass>().getDomTree();
  1962:   MRI = &MF.getRegInfo();
  1963:   AssignmentMap IMap;
  1964: 
  1965:   collect(MF);
  1966:   llvm::sort(Extenders, [this](const ExtDesc &A, const ExtDesc &B) {
  1967:     ExtValue VA(A), VB(B);
  1968:     if (VA != VB)
  1969:       return VA < VB;
  1970:     const MachineInstr *MA = A.UseMI;
  1971:     const MachineInstr *MB = B.UseMI;
  1972:     if (MA == MB) {
  1973:       // If it's the same instruction, compare operand numbers.
  1974:       return A.OpNum < B.OpNum;
  1975:     }
  1976: 
  1977:     const MachineBasicBlock *BA = MA->getParent();
  1978:     const MachineBasicBlock *BB = MB->getParent();
  1979:     assert(BA->getNumber() != -1 && BB->getNumber() != -1);
  1980:     if (BA != BB)
  1981:       return BA->getNumber() < BB->getNumber();
  1982:     return MDT->dominates(MA, MB);
  1983:   });
  1984: 
  1985:   bool Changed = false;
  1986:   LLVM_DEBUG(dbgs() << "Collected " << Extenders.size() << " extenders\n");
  1987:   for (unsigned I = 0, E = Extenders.size(); I != E; ) {
  1988:     unsigned B = I;
  1989:     const ExtRoot &T = Extenders[B].getOp();
  1990:     while (I != E && ExtRoot(Extenders[I].getOp()) == T)
  1991:       ++I;
  1992: 
  1993:     IMap.clear();
  1994:     assignInits(T, B, I, IMap);
  1995:     Changed |= replaceExtenders(IMap);
  1996:   }
  1997: 
  1998:   LLVM_DEBUG({
  1999:     if (Changed)
  2000:       MF.print(dbgs() << "After " << getPassName() << '\n', nullptr);
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. It declares or implements routines such as getRegOffOpcode, assert, BuildMI, add, ... (39 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里声明或实现了 getRegOffOpcode, assert, BuildMI, add, ... (39 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 2001-2009 / 第 2001-2009 行

```cpp
  2001:     else
  2002:       dbgs() << "No changes\n";
  2003:   });
  2004:   return Changed;
  2005: }
  2006: 
  2007: FunctionPass *llvm::createHexagonConstExtenders() {
  2008:   return new HexagonConstExtenders();
  2009: }
```
- EN: It declares or implements routines such as llvm::createHexagonConstExtenders, HexagonConstExtenders, translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonConstExtenders, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 llvm::createHexagonConstExtenders, HexagonConstExtenders 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonConstExtenders，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- instruction semantics / 指令语义
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `HexagonInstrInfo.h, HexagonRegisterInfo.h, HexagonSubtarget.h, llvm/ADT/SetVector.h, llvm/ADT/SmallVector.h, llvm/CodeGen/MachineDominators.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstrBuilder.h, llvm/CodeGen/MachineRegisterInfo.h, llvm/CodeGen/Register.h, ... (18 total)`
- Hexagon symbols / Hexagon 符号: `HexagonConstExtenders, HexagonInstrInfo, HexagonRegisterInfo, HexagonSubtarget, HexagonII`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
