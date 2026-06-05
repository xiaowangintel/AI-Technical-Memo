# HexagonGenPredicate.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonGenPredicate.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon generation of predicate-producing operations.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```cpp
     1: //===- HexagonGenPredicate.cpp --------------------------------------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: 
     9: #include "Hexagon.h"
    10: #include "HexagonInstrInfo.h"
    11: #include "HexagonSubtarget.h"
    12: #include "llvm/ADT/SetVector.h"
    13: #include "llvm/ADT/StringRef.h"
    14: #include "llvm/CodeGen/MachineBasicBlock.h"
    15: #include "llvm/CodeGen/MachineDominators.h"
    16: #include "llvm/CodeGen/MachineFunction.h"
    17: #include "llvm/CodeGen/MachineFunctionPass.h"
    18: #include "llvm/CodeGen/MachineInstr.h"
    19: #include "llvm/CodeGen/MachineInstrBuilder.h"
    20: #include "llvm/CodeGen/MachineOperand.h"
    21: #include "llvm/CodeGen/MachineRegisterInfo.h"
    22: #include "llvm/CodeGen/TargetRegisterInfo.h"
    23: #include "llvm/IR/DebugLoc.h"
    24: #include "llvm/InitializePasses.h"
    25: #include "llvm/Pass.h"
    26: #include "llvm/Support/Compiler.h"
    27: #include "llvm/Support/Debug.h"
    28: #include "llvm/Support/ErrorHandling.h"
    29: #include "llvm/Support/raw_ostream.h"
    30: #include <cassert>
    31: #include <iterator>
    32: #include <queue>
    33: 
    34: #define DEBUG_TYPE "gen-pred"
    35: 
    36: using namespace llvm;
    37: 
    38: namespace {
    39: 
    40: using RegSubRegPair = TargetInstrInfo::RegSubRegPair;
    41: 
    42: struct PrintRegister {
    43:   friend raw_ostream &operator<<(raw_ostream &OS, const PrintRegister &PR);
    44: 
    45:   PrintRegister(RegSubRegPair R, const TargetRegisterInfo &I)
    46:       : Reg(R), TRI(I) {}
    47: 
    48: private:
    49:   RegSubRegPair Reg;
    50:   const TargetRegisterInfo &TRI;
```
- EN: It imports headers such as Hexagon.h, HexagonInstrInfo.h, HexagonSubtarget.h, llvm/ADT/SetVector.h, ... (24 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as PrintRegister, which carry the state or API of this component.
- CN: 这里引入了 Hexagon.h, HexagonInstrInfo.h, HexagonSubtarget.h, llvm/ADT/SetVector.h, ... (24 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 PrintRegister 等类型，用来承载该组件的状态或接口。

### Lines 51-100 / 第 51-100 行

```cpp
    51: };
    52: 
    53: [[maybe_unused]] raw_ostream &operator<<(raw_ostream &OS,
    54:                                          const PrintRegister &PR);
    55: raw_ostream &operator<<(raw_ostream &OS, const PrintRegister &PR) {
    56:   return OS << printReg(PR.Reg.Reg, &PR.TRI, PR.Reg.SubReg);
    57: }
    58: 
    59:   class HexagonGenPredicate : public MachineFunctionPass {
    60:   public:
    61:     static char ID;
    62: 
    63:     HexagonGenPredicate() : MachineFunctionPass(ID) {}
    64: 
    65:     StringRef getPassName() const override {
    66:       return "Hexagon generate predicate operations";
    67:     }
    68: 
    69:     void getAnalysisUsage(AnalysisUsage &AU) const override {
    70:       AU.addRequired<MachineDominatorTreeWrapperPass>();
    71:       AU.addPreserved<MachineDominatorTreeWrapperPass>();
    72:       MachineFunctionPass::getAnalysisUsage(AU);
    73:     }
    74: 
    75:     bool runOnMachineFunction(MachineFunction &MF) override;
    76: 
    77:   private:
    78:     using VectOfInst = SetVector<MachineInstr *>;
    79:     using SetOfReg = SetVector<RegSubRegPair>;
    80:     using RegToRegMap = DenseMap<RegSubRegPair, RegSubRegPair>;
    81: 
    82:     const HexagonInstrInfo *TII = nullptr;
    83:     const HexagonRegisterInfo *TRI = nullptr;
    84:     MachineRegisterInfo *MRI = nullptr;
    85:     SetOfReg PredGPRs;
    86:     VectOfInst PUsers;
    87:     RegToRegMap G2P;
    88: 
    89:     bool isPredReg(Register R);
    90:     void collectPredicateGPR(MachineFunction &MF);
    91:     void processPredicateGPR(const RegSubRegPair &Reg);
    92:     unsigned getPredForm(unsigned Opc);
    93:     bool isConvertibleToPredForm(const MachineInstr *MI);
    94:     bool isScalarCmp(unsigned Opc);
    95:     bool isScalarPred(RegSubRegPair PredReg);
    96:     RegSubRegPair getPredRegFor(const RegSubRegPair &Reg);
    97:     bool convertToPredForm(MachineInstr *MI);
    98:     bool eliminatePredCopies(MachineFunction &MF);
    99:   };
   100: 
```
- EN: It declares types such as HexagonGenPredicate, which carry the state or API of this component. It defines declarative TableGen records like HexagonGenPredicate; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as printReg, HexagonGenPredicate, getPassName, getAnalysisUsage, ... (18 total), translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonGenPredicate, HexagonInstrInfo, HexagonRegisterInfo, showing how the code connects to sibling backend components.
- CN: 这里声明了 HexagonGenPredicate 等类型，用来承载该组件的状态或接口。 这里定义了 HexagonGenPredicate 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 printReg, HexagonGenPredicate, getPassName, getAnalysisUsage, ... (18 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonGenPredicate, HexagonInstrInfo, HexagonRegisterInfo，说明了它与同级后端组件的连接关系。

### Lines 101-150 / 第 101-150 行

```cpp
   101: } // end anonymous namespace
   102: 
   103: char HexagonGenPredicate::ID = 0;
   104: 
   105: INITIALIZE_PASS_BEGIN(HexagonGenPredicate, "hexagon-gen-pred-pass",
   106:                       "Hexagon generate predicate operations", false, false)
   107: INITIALIZE_PASS_DEPENDENCY(MachineDominatorTreeWrapperPass)
   108: INITIALIZE_PASS_END(HexagonGenPredicate, "hexagon-gen-pred-pass",
   109:                     "Hexagon generate predicate operations", false, false)
   110: 
   111: bool HexagonGenPredicate::isPredReg(Register R) {
   112:   if (!R.isVirtual())
   113:     return false;
   114:   const TargetRegisterClass *RC = MRI->getRegClass(R);
   115:   return RC == &Hexagon::PredRegsRegClass;
   116: }
   117: 
   118: unsigned HexagonGenPredicate::getPredForm(unsigned Opc) {
   119:   using namespace Hexagon;
   120: 
   121:   switch (Opc) {
   122:     case A2_and:
   123:     case A2_andp:
   124:       return C2_and;
   125:     case A4_andn:
   126:     case A4_andnp:
   127:       return C2_andn;
   128:     case M4_and_and:
   129:       return C4_and_and;
   130:     case M4_and_andn:
   131:       return C4_and_andn;
   132:     case M4_and_or:
   133:       return C4_and_or;
   134: 
   135:     case A2_or:
   136:     case A2_orp:
   137:       return C2_or;
   138:     case A4_orn:
   139:     case A4_ornp:
   140:       return C2_orn;
   141:     case M4_or_and:
   142:       return C4_or_and;
   143:     case M4_or_andn:
   144:       return C4_or_andn;
   145:     case M4_or_or:
   146:       return C4_or_or;
   147: 
   148:     case A2_xor:
   149:     case A2_xorp:
   150:       return C2_xor;
```
- EN: It opens namespaces (char, Hexagon) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as INITIALIZE_PASS_BEGIN, getRegClass, HexagonGenPredicate::getPredForm, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonGenPredicate, showing how the code connects to sibling backend components.
- CN: 这里打开了命名空间（char, Hexagon），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 INITIALIZE_PASS_BEGIN, getRegClass, HexagonGenPredicate::getPredForm 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonGenPredicate，说明了它与同级后端组件的连接关系。

### Lines 151-200 / 第 151-200 行

```cpp
   151: 
   152:     case C2_tfrrp:
   153:       return COPY;
   154:   }
   155:   // The opcode corresponding to 0 is TargetOpcode::PHI. We can use 0 here
   156:   // to denote "none", but we need to make sure that none of the valid opcodes
   157:   // that we return will ever be 0.
   158:   static_assert(PHI == 0, "Use different value for <none>");
   159:   return 0;
   160: }
   161: 
   162: bool HexagonGenPredicate::isConvertibleToPredForm(const MachineInstr *MI) {
   163:   unsigned Opc = MI->getOpcode();
   164:   if (getPredForm(Opc) != 0)
   165:     return true;
   166: 
   167:   // Comparisons against 0 are also convertible. This does not apply to
   168:   // A4_rcmpeqi or A4_rcmpneqi, since they produce values 0 or 1, which
   169:   // may not match the value that the predicate register would have if
   170:   // it was converted to a predicate form.
   171:   switch (Opc) {
   172:     case Hexagon::C2_cmpeqi:
   173:     case Hexagon::C4_cmpneqi:
   174:       if (MI->getOperand(2).isImm() && MI->getOperand(2).getImm() == 0)
   175:         return true;
   176:       break;
   177:   }
   178:   return false;
   179: }
   180: 
   181: void HexagonGenPredicate::collectPredicateGPR(MachineFunction &MF) {
   182:   for (MachineBasicBlock &B : MF) {
   183:     for (MachineInstr &MI : B) {
   184:       unsigned Opc = MI.getOpcode();
   185:       switch (Opc) {
   186:         case Hexagon::C2_tfrpr:
   187:         case TargetOpcode::COPY:
   188:           if (isPredReg(MI.getOperand(1).getReg())) {
   189:             RegSubRegPair RD = getRegSubRegPair(MI.getOperand(0));
   190:             if (RD.Reg.isVirtual())
   191:               PredGPRs.insert(RD);
   192:           }
   193:           break;
   194:       }
   195:     }
   196:   }
   197: }
   198: 
   199: void HexagonGenPredicate::processPredicateGPR(const RegSubRegPair &Reg) {
   200:   LLVM_DEBUG(dbgs() << __func__ << ": " << printReg(Reg.Reg, TRI, Reg.SubReg)
```
- EN: It declares or implements routines such as static_assert, HexagonGenPredicate::isConvertibleToPredForm, getOpcode, HexagonGenPredicate::collectPredicateGPR, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonGenPredicate, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 static_assert, HexagonGenPredicate::isConvertibleToPredForm, getOpcode, HexagonGenPredicate::collectPredicateGPR, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonGenPredicate，说明了它与同级后端组件的连接关系。

### Lines 201-250 / 第 201-250 行

```cpp
   201:                     << "\n");
   202:   using use_iterator = MachineRegisterInfo::use_iterator;
   203: 
   204:   use_iterator I = MRI->use_begin(Reg.Reg), E = MRI->use_end();
   205:   if (I == E) {
   206:     LLVM_DEBUG(dbgs() << "Dead reg: " << printReg(Reg.Reg, TRI, Reg.SubReg)
   207:                       << '\n');
   208:     MachineInstr *DefI = MRI->getVRegDef(Reg.Reg);
   209:     DefI->eraseFromParent();
   210:     return;
   211:   }
   212: 
   213:   for (; I != E; ++I) {
   214:     MachineInstr *UseI = I->getParent();
   215:     if (isConvertibleToPredForm(UseI))
   216:       PUsers.insert(UseI);
   217:   }
   218: }
   219: 
   220: RegSubRegPair HexagonGenPredicate::getPredRegFor(const RegSubRegPair &Reg) {
   221:   // Create a predicate register for a given Reg. The newly created register
   222:   // will have its value copied from Reg, so that it can be later used as
   223:   // an operand in other instructions.
   224:   assert(Reg.Reg.isVirtual());
   225:   RegToRegMap::iterator F = G2P.find(Reg);
   226:   if (F != G2P.end())
   227:     return F->second;
   228: 
   229:   LLVM_DEBUG(dbgs() << __func__ << ": " << PrintRegister(Reg, *TRI));
   230:   MachineInstr *DefI = MRI->getVRegDef(Reg.Reg);
   231:   assert(DefI);
   232:   unsigned Opc = DefI->getOpcode();
   233:   if (Opc == Hexagon::C2_tfrpr || Opc == TargetOpcode::COPY) {
   234:     assert(DefI->getOperand(0).isDef() && DefI->getOperand(1).isUse());
   235:     RegSubRegPair PR = getRegSubRegPair(DefI->getOperand(1));
   236:     G2P.insert(std::make_pair(Reg, PR));
   237:     LLVM_DEBUG(dbgs() << " -> " << PrintRegister(PR, *TRI) << '\n');
   238:     return PR;
   239:   }
   240: 
   241:   MachineBasicBlock &B = *DefI->getParent();
   242:   DebugLoc DL = DefI->getDebugLoc();
   243:   const TargetRegisterClass *PredRC = &Hexagon::PredRegsRegClass;
   244:   Register NewPR = MRI->createVirtualRegister(PredRC);
   245: 
   246:   // For convertible instructions, do not modify them, so that they can
   247:   // be converted later.  Generate a copy from Reg to NewPR.
   248:   if (isConvertibleToPredForm(DefI)) {
   249:     MachineBasicBlock::iterator DefIt = DefI;
   250:     BuildMI(B, std::next(DefIt), DL, TII->get(TargetOpcode::COPY), NewPR)
```
- EN: It declares or implements routines such as use_begin, LLVM_DEBUG, getVRegDef, eraseFromParent, ... (13 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonGenPredicate, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 use_begin, LLVM_DEBUG, getVRegDef, eraseFromParent, ... (13 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonGenPredicate，说明了它与同级后端组件的连接关系。

### Lines 251-300 / 第 251-300 行

```cpp
   251:         .addReg(Reg.Reg, {}, Reg.SubReg);
   252:     G2P.insert(std::make_pair(Reg, RegSubRegPair(NewPR)));
   253:     LLVM_DEBUG(dbgs() << " -> !" << PrintRegister(RegSubRegPair(NewPR), *TRI)
   254:                       << '\n');
   255:     return RegSubRegPair(NewPR);
   256:   }
   257: 
   258:   llvm_unreachable("Invalid argument");
   259: }
   260: 
   261: bool HexagonGenPredicate::isScalarCmp(unsigned Opc) {
   262:   switch (Opc) {
   263:     case Hexagon::C2_cmpeq:
   264:     case Hexagon::C2_cmpgt:
   265:     case Hexagon::C2_cmpgtu:
   266:     case Hexagon::C2_cmpeqp:
   267:     case Hexagon::C2_cmpgtp:
   268:     case Hexagon::C2_cmpgtup:
   269:     case Hexagon::C2_cmpeqi:
   270:     case Hexagon::C2_cmpgti:
   271:     case Hexagon::C2_cmpgtui:
   272:     case Hexagon::C2_cmpgei:
   273:     case Hexagon::C2_cmpgeui:
   274:     case Hexagon::C4_cmpneqi:
   275:     case Hexagon::C4_cmpltei:
   276:     case Hexagon::C4_cmplteui:
   277:     case Hexagon::C4_cmpneq:
   278:     case Hexagon::C4_cmplte:
   279:     case Hexagon::C4_cmplteu:
   280:     case Hexagon::A4_cmpbeq:
   281:     case Hexagon::A4_cmpbeqi:
   282:     case Hexagon::A4_cmpbgtu:
   283:     case Hexagon::A4_cmpbgtui:
   284:     case Hexagon::A4_cmpbgt:
   285:     case Hexagon::A4_cmpbgti:
   286:     case Hexagon::A4_cmpheq:
   287:     case Hexagon::A4_cmphgt:
   288:     case Hexagon::A4_cmphgtu:
   289:     case Hexagon::A4_cmpheqi:
   290:     case Hexagon::A4_cmphgti:
   291:     case Hexagon::A4_cmphgtui:
   292:       return true;
   293:   }
   294:   return false;
   295: }
   296: 
   297: bool HexagonGenPredicate::isScalarPred(RegSubRegPair PredReg) {
   298:   std::queue<RegSubRegPair> WorkQ;
   299:   WorkQ.push(PredReg);
   300: 
```
- EN: It declares or implements routines such as insert, LLVM_DEBUG, RegSubRegPair, llvm_unreachable, ... (7 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonGenPredicate, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 insert, LLVM_DEBUG, RegSubRegPair, llvm_unreachable, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonGenPredicate，说明了它与同级后端组件的连接关系。

### Lines 301-350 / 第 301-350 行

```cpp
   301:   while (!WorkQ.empty()) {
   302:     RegSubRegPair PR = WorkQ.front();
   303:     WorkQ.pop();
   304:     const MachineInstr *DefI = MRI->getVRegDef(PR.Reg);
   305:     if (!DefI)
   306:       return false;
   307:     unsigned DefOpc = DefI->getOpcode();
   308:     switch (DefOpc) {
   309:       case TargetOpcode::COPY: {
   310:         const TargetRegisterClass *PredRC = &Hexagon::PredRegsRegClass;
   311:         if (MRI->getRegClass(PR.Reg) != PredRC)
   312:           return false;
   313:         // If it is a copy between two predicate registers, fall through.
   314:         [[fallthrough]];
   315:       }
   316:       case Hexagon::C2_and:
   317:       case Hexagon::C2_andn:
   318:       case Hexagon::C4_and_and:
   319:       case Hexagon::C4_and_andn:
   320:       case Hexagon::C4_and_or:
   321:       case Hexagon::C2_or:
   322:       case Hexagon::C2_orn:
   323:       case Hexagon::C4_or_and:
   324:       case Hexagon::C4_or_andn:
   325:       case Hexagon::C4_or_or:
   326:       case Hexagon::C4_or_orn:
   327:       case Hexagon::C2_xor:
   328:         // Add operands to the queue.
   329:         for (const MachineOperand &MO : DefI->operands())
   330:           if (MO.isReg() && MO.isUse())
   331:             WorkQ.push(RegSubRegPair(MO.getReg()));
   332:         break;
   333: 
   334:       // All non-vector compares are ok, everything else is bad.
   335:       default:
   336:         return isScalarCmp(DefOpc);
   337:     }
   338:   }
   339: 
   340:   return true;
   341: }
   342: 
   343: bool HexagonGenPredicate::convertToPredForm(MachineInstr *MI) {
   344:   LLVM_DEBUG(dbgs() << __func__ << ": " << MI << " " << *MI);
   345: 
   346:   unsigned Opc = MI->getOpcode();
   347:   assert(isConvertibleToPredForm(MI));
   348:   unsigned NumOps = MI->getNumOperands();
   349:   for (unsigned i = 0; i < NumOps; ++i) {
   350:     MachineOperand &MO = MI->getOperand(i);
```
- EN: It declares or implements routines such as front, pop, getVRegDef, getOpcode, ... (10 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonGenPredicate, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 front, pop, getVRegDef, getOpcode, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonGenPredicate，说明了它与同级后端组件的连接关系。

### Lines 351-400 / 第 351-400 行

```cpp
   351:     if (!MO.isReg() || !MO.isUse())
   352:       continue;
   353:     RegSubRegPair Reg(getRegSubRegPair(MO));
   354:     if (Reg.SubReg && Reg.SubReg != Hexagon::isub_lo)
   355:       return false;
   356:     if (!PredGPRs.count(Reg))
   357:       return false;
   358:   }
   359: 
   360:   MachineBasicBlock &B = *MI->getParent();
   361:   DebugLoc DL = MI->getDebugLoc();
   362: 
   363:   unsigned NewOpc = getPredForm(Opc);
   364:   // Special case for comparisons against 0.
   365:   if (NewOpc == 0) {
   366:     switch (Opc) {
   367:       case Hexagon::C2_cmpeqi:
   368:         NewOpc = Hexagon::C2_not;
   369:         break;
   370:       case Hexagon::C4_cmpneqi:
   371:         NewOpc = TargetOpcode::COPY;
   372:         break;
   373:       default:
   374:         return false;
   375:     }
   376: 
   377:     // If it's a scalar predicate register, then all bits in it are
   378:     // the same. Otherwise, to determine whether all bits are 0 or not
   379:     // we would need to use any8.
   380:     RegSubRegPair PR = getPredRegFor(getRegSubRegPair(MI->getOperand(1)));
   381:     if (!isScalarPred(PR))
   382:       return false;
   383:     // This will skip the immediate argument when creating the predicate
   384:     // version instruction.
   385:     NumOps = 2;
   386:   }
   387: 
   388:   // Check that def is in operand #0.
   389:   MachineOperand &Op0 = MI->getOperand(0);
   390:   assert(Op0.isDef());
   391:   RegSubRegPair OutR(getRegSubRegPair(Op0));
   392: 
   393:   // Don't use getPredRegFor, since it will create an association between
   394:   // the argument and a created predicate register (i.e. it will insert a
   395:   // copy if a new predicate register is created).
   396:   const TargetRegisterClass *PredRC = &Hexagon::PredRegsRegClass;
   397:   RegSubRegPair NewPR = MRI->createVirtualRegister(PredRC);
   398:   MachineInstrBuilder MIB = BuildMI(B, MI, DL, TII->get(NewOpc), NewPR.Reg);
   399: 
   400:   // Add predicate counterparts of the GPRs.
```
- EN: It declares or implements routines such as Reg, getParent, getDebugLoc, getPredForm, ... (10 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 Reg, getParent, getDebugLoc, getPredForm, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 401-450 / 第 401-450 行

```cpp
   401:   for (unsigned i = 1; i < NumOps; ++i) {
   402:     RegSubRegPair GPR = getRegSubRegPair(MI->getOperand(i));
   403:     RegSubRegPair Pred = getPredRegFor(GPR);
   404:     MIB.addReg(Pred.Reg, {}, Pred.SubReg);
   405:   }
   406:   LLVM_DEBUG(dbgs() << "generated: " << *MIB);
   407: 
   408:   // Generate a copy-out: NewGPR = NewPR, and replace all uses of OutR
   409:   // with NewGPR.
   410:   const TargetRegisterClass *RC = MRI->getRegClass(OutR.Reg);
   411:   Register NewOutR = MRI->createVirtualRegister(RC);
   412:   BuildMI(B, MI, DL, TII->get(TargetOpcode::COPY), NewOutR)
   413:       .addReg(NewPR.Reg, {}, NewPR.SubReg);
   414:   MRI->replaceRegWith(OutR.Reg, NewOutR);
   415:   MI->eraseFromParent();
   416: 
   417:   // If the processed instruction was C2_tfrrp (i.e. Rn = Pm; Pk = Rn),
   418:   // then the output will be a predicate register.  Do not visit the
   419:   // users of it.
   420:   if (!isPredReg(NewOutR)) {
   421:     RegSubRegPair R(NewOutR);
   422:     PredGPRs.insert(R);
   423:     processPredicateGPR(R);
   424:   }
   425:   return true;
   426: }
   427: 
   428: bool HexagonGenPredicate::eliminatePredCopies(MachineFunction &MF) {
   429:   LLVM_DEBUG(dbgs() << __func__ << "\n");
   430:   const TargetRegisterClass *PredRC = &Hexagon::PredRegsRegClass;
   431:   bool Changed = false;
   432:   VectOfInst Erase;
   433: 
   434:   // First, replace copies
   435:   //   IntR = PredR1
   436:   //   PredR2 = IntR
   437:   // with
   438:   //   PredR2 = PredR1
   439:   // Such sequences can be generated when a copy-into-pred is generated from
   440:   // a gpr register holding a result of a convertible instruction. After
   441:   // the convertible instruction is converted, its predicate result will be
   442:   // copied back into the original gpr.
   443: 
   444:   for (MachineBasicBlock &MBB : MF) {
   445:     for (MachineInstr &MI : MBB) {
   446:       if (MI.getOpcode() != TargetOpcode::COPY)
   447:         continue;
   448:       RegSubRegPair DR = getRegSubRegPair(MI.getOperand(0));
   449:       RegSubRegPair SR = getRegSubRegPair(MI.getOperand(1));
   450:       if (!DR.Reg.isVirtual())
```
- EN: It declares or implements routines such as getRegSubRegPair, getPredRegFor, LLVM_DEBUG, getRegClass, ... (11 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonGenPredicate, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getRegSubRegPair, getPredRegFor, LLVM_DEBUG, getRegClass, ... (11 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonGenPredicate，说明了它与同级后端组件的连接关系。

### Lines 451-500 / 第 451-500 行

```cpp
   451:         continue;
   452:       if (!SR.Reg.isVirtual())
   453:         continue;
   454:       if (MRI->getRegClass(DR.Reg) != PredRC)
   455:         continue;
   456:       if (MRI->getRegClass(SR.Reg) != PredRC)
   457:         continue;
   458:       assert(!DR.SubReg && !SR.SubReg && "Unexpected subregister");
   459:       MRI->replaceRegWith(DR.Reg, SR.Reg);
   460:       Erase.insert(&MI);
   461:       Changed = true;
   462:     }
   463:   }
   464: 
   465:   for (MachineInstr *MI : Erase)
   466:     MI->eraseFromParent();
   467: 
   468:   return Changed;
   469: }
   470: 
   471: bool HexagonGenPredicate::runOnMachineFunction(MachineFunction &MF) {
   472:   if (skipFunction(MF.getFunction()))
   473:     return false;
   474: 
   475:   TII = MF.getSubtarget<HexagonSubtarget>().getInstrInfo();
   476:   TRI = MF.getSubtarget<HexagonSubtarget>().getRegisterInfo();
   477:   MRI = &MF.getRegInfo();
   478:   PredGPRs.clear();
   479:   PUsers.clear();
   480:   G2P.clear();
   481: 
   482:   bool Changed = false;
   483:   collectPredicateGPR(MF);
   484:   for (const RegSubRegPair &R : PredGPRs)
   485:     processPredicateGPR(R);
   486: 
   487:   bool Again;
   488:   do {
   489:     Again = false;
   490:     VectOfInst Processed, Copy;
   491: 
   492:     Copy = PUsers;
   493:     for (MachineInstr *MI : Copy) {
   494:       bool Done = convertToPredForm(MI);
   495:       if (Done) {
   496:         Processed.insert(MI);
   497:         Again = true;
   498:       }
   499:     }
   500:     Changed |= Again;
```
- EN: It declares or implements routines such as assert, replaceRegWith, insert, HexagonGenPredicate::runOnMachineFunction, ... (9 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonGenPredicate, HexagonSubtarget, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 assert, replaceRegWith, insert, HexagonGenPredicate::runOnMachineFunction, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonGenPredicate, HexagonSubtarget，说明了它与同级后端组件的连接关系。

### Lines 501-514 / 第 501-514 行

```cpp
   501: 
   502:     auto Done = [Processed] (MachineInstr *MI) -> bool {
   503:       return Processed.count(MI);
   504:     };
   505:     PUsers.remove_if(Done);
   506:   } while (Again);
   507: 
   508:   Changed |= eliminatePredCopies(MF);
   509:   return Changed;
   510: }
   511: 
   512: FunctionPass *llvm::createHexagonGenPredicate() {
   513:   return new HexagonGenPredicate();
   514: }
```
- EN: It declares or implements routines such as count, remove_if, eliminatePredCopies, llvm::createHexagonGenPredicate, ... (5 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonGenPredicate, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 count, remove_if, eliminatePredCopies, llvm::createHexagonGenPredicate, ... (5 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonGenPredicate，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- instruction semantics / 指令语义
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `Hexagon.h, HexagonInstrInfo.h, HexagonSubtarget.h, llvm/ADT/SetVector.h, llvm/ADT/StringRef.h, llvm/CodeGen/MachineBasicBlock.h, llvm/CodeGen/MachineDominators.h, llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstr.h, ... (24 total)`
- Hexagon symbols / Hexagon 符号: `HexagonGenPredicate, HexagonInstrInfo, HexagonSubtarget, HexagonRegisterInfo`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
