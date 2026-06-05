# HexagonSubtarget.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonSubtarget.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): This file implements the Hexagon specific subclass of TargetSubtarget.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```cpp
     1: //===- HexagonSubtarget.cpp - Hexagon Subtarget Information ---------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: //
     9: // This file implements the Hexagon specific subclass of TargetSubtarget.
    10: //
    11: //===----------------------------------------------------------------------===//
    12: 
    13: #include "HexagonSubtarget.h"
    14: #include "HexagonInstrInfo.h"
    15: #include "HexagonRegisterInfo.h"
    16: #include "MCTargetDesc/HexagonMCTargetDesc.h"
    17: #include "llvm/ADT/STLExtras.h"
    18: #include "llvm/ADT/SmallVector.h"
    19: #include "llvm/ADT/StringRef.h"
    20: #include "llvm/CodeGen/MachineInstr.h"
    21: #include "llvm/CodeGen/MachineOperand.h"
    22: #include "llvm/CodeGen/MachineScheduler.h"
    23: #include "llvm/CodeGen/ScheduleDAG.h"
    24: #include "llvm/CodeGen/ScheduleDAGInstrs.h"
    25: #include "llvm/IR/IntrinsicsHexagon.h"
    26: #include "llvm/Support/CommandLine.h"
    27: #include "llvm/Support/ErrorHandling.h"
    28: #include "llvm/Target/TargetMachine.h"
    29: #include <algorithm>
    30: #include <cassert>
    31: #include <optional>
    32: 
    33: using namespace llvm;
    34: 
    35: #define DEBUG_TYPE "hexagon-subtarget"
    36: 
    37: #define GET_SUBTARGETINFO_CTOR
    38: #define GET_SUBTARGETINFO_TARGET_DESC
    39: #include "HexagonGenSubtargetInfo.inc"
    40: 
    41: static cl::opt<bool> EnableBSBSched("enable-bsb-sched", cl::Hidden,
    42:                                     cl::init(true));
    43: 
    44: static cl::opt<bool> EnableTCLatencySched("enable-tc-latency-sched", cl::Hidden,
    45:                                           cl::init(false));
    46: 
    47: static cl::opt<bool>
    48:     EnableDotCurSched("enable-cur-sched", cl::Hidden, cl::init(true),
    49:                       cl::desc("Enable the scheduler to generate .cur"));
    50: 
```
- EN: It imports headers such as HexagonSubtarget.h, HexagonInstrInfo.h, HexagonRegisterInfo.h, MCTargetDesc/HexagonMCTargetDesc.h, ... (20 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as EnableBSBSched, EnableTCLatencySched, EnableDotCurSched, translating Hexagon-specific policy into reusable code paths.
- CN: 这里引入了 HexagonSubtarget.h, HexagonInstrInfo.h, HexagonRegisterInfo.h, MCTargetDesc/HexagonMCTargetDesc.h, ... (20 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 EnableBSBSched, EnableTCLatencySched, EnableDotCurSched 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 51-100 / 第 51-100 行

```cpp
    51: static cl::opt<bool>
    52:     DisableHexagonMISched("disable-hexagon-misched", cl::Hidden,
    53:                           cl::desc("Disable Hexagon MI Scheduling"));
    54: 
    55: static cl::opt<bool> OverrideLongCalls(
    56:     "hexagon-long-calls", cl::Hidden,
    57:     cl::desc("If present, forces/disables the use of long calls"));
    58: 
    59: static cl::opt<bool>
    60:     EnablePredicatedCalls("hexagon-pred-calls", cl::Hidden,
    61:                           cl::desc("Consider calls to be predicable"));
    62: 
    63: static cl::opt<bool> SchedPredsCloser("sched-preds-closer", cl::Hidden,
    64:                                       cl::init(true));
    65: 
    66: static cl::opt<bool> SchedRetvalOptimization("sched-retval-optimization",
    67:                                              cl::Hidden, cl::init(true));
    68: 
    69: static cl::opt<bool> EnableCheckBankConflict(
    70:     "hexagon-check-bank-conflict", cl::Hidden, cl::init(true),
    71:     cl::desc("Enable checking for cache bank conflicts"));
    72: 
    73: HexagonSubtarget::HexagonSubtarget(const Triple &TT, StringRef CPU,
    74:                                    StringRef FS, const TargetMachine &TM)
    75:     : HexagonGenSubtargetInfo(TT, CPU, /*TuneCPU*/ CPU, FS),
    76:       OptLevel(TM.getOptLevel()),
    77:       CPUString(std::string(Hexagon_MC::selectHexagonCPU(CPU))),
    78:       TargetTriple(TT), InstrInfo(initializeSubtargetDependencies(CPU, FS)),
    79:       TLInfo(TM, *this), InstrItins(getInstrItineraryForCPU(CPUString)) {
    80:   Hexagon_MC::addArchSubtarget(this, FS);
    81:   // Beware of the default constructor of InstrItineraryData: it will
    82:   // reset all members to 0.
    83:   assert(InstrItins.Itineraries != nullptr && "InstrItins not initialized");
    84: }
    85: 
    86: HexagonSubtarget &
    87: HexagonSubtarget::initializeSubtargetDependencies(StringRef CPU, StringRef FS) {
    88:   std::optional<Hexagon::ArchEnum> ArchVer = Hexagon::getCpu(CPUString);
    89:   if (ArchVer)
    90:     HexagonArchVersion = *ArchVer;
    91:   else
    92:     llvm_unreachable("Unrecognized Hexagon processor version");
    93: 
    94:   UseHVX128BOps = false;
    95:   UseHVX64BOps = false;
    96:   UseAudioOps = false;
    97:   UseLongCalls = false;
    98: 
    99:   SubtargetFeatures Features(FS);
   100: 
```
- EN: It declares or implements routines such as DisableHexagonMISched, OverrideLongCalls, EnablePredicatedCalls, SchedPredsCloser, ... (13 total), translating Hexagon-specific policy into reusable code paths. Command-line options appear here to gate diagnostics or target-specific tuning behavior. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里声明或实现了 DisableHexagonMISched, OverrideLongCalls, EnablePredicatedCalls, SchedPredsCloser, ... (13 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里出现了命令行选项，用于控制诊断开关或目标相关的调优行为。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 101-150 / 第 101-150 行

```cpp
   101:   // Turn on QFloat if the HVX version is v68+.
   102:   // The function ParseSubtargetFeatures will set feature bits and initialize
   103:   // subtarget's variables all in one, so there isn't a good way to preprocess
   104:   // the feature string, other than by tinkering with it directly.
   105:   auto IsQFloatFS = [](StringRef F) {
   106:     return F == "+hvx-qfloat" || F == "-hvx-qfloat";
   107:   };
   108:   if (!llvm::count_if(Features.getFeatures(), IsQFloatFS)) {
   109:     auto getHvxVersion = [&Features](StringRef FS) -> StringRef {
   110:       for (StringRef F : llvm::reverse(Features.getFeatures())) {
   111:         if (F.starts_with("+hvxv"))
   112:           return F;
   113:       }
   114:       for (StringRef F : llvm::reverse(Features.getFeatures())) {
   115:         if (F == "-hvx")
   116:           return StringRef();
   117:         if (F.starts_with("+hvx") || F == "-hvx")
   118:           return F.take_front(4);  // Return "+hvx" or "-hvx".
   119:       }
   120:       return StringRef();
   121:     };
   122: 
   123:     bool AddQFloat = false;
   124:     StringRef HvxVer = getHvxVersion(FS);
   125:     if (HvxVer.starts_with("+hvxv")) {
   126:       int Ver = 0;
   127:       if (!HvxVer.drop_front(5).consumeInteger(10, Ver) && Ver >= 68)
   128:         AddQFloat = true;
   129:     } else if (HvxVer == "+hvx") {
   130:       if (hasV68Ops())
   131:         AddQFloat = true;
   132:     }
   133: 
   134:     if (AddQFloat)
   135:       Features.AddFeature("+hvx-qfloat");
   136:   }
   137: 
   138:   std::string FeatureString = Features.getString();
   139:   ParseSubtargetFeatures(CPUString, /*TuneCPU*/ CPUString, FeatureString);
   140: 
   141:   if (useHVXV68Ops())
   142:     UseHVXFloatingPoint = UseHVXIEEEFPOps || UseHVXQFloatOps;
   143: 
   144:   if (UseHVXQFloatOps && UseHVXIEEEFPOps && UseHVXFloatingPoint)
   145:     LLVM_DEBUG(
   146:         dbgs() << "Behavior is undefined for simultaneous qfloat and ieee hvx codegen...");
   147: 
   148:   if (OverrideLongCalls.getPosition())
   149:     UseLongCalls = OverrideLongCalls;
   150: 
```
- EN: It declares or implements routines such as StringRef, getHvxVersion, getString, ParseSubtargetFeatures, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 StringRef, getHvxVersion, getString, ParseSubtargetFeatures 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 151-200 / 第 151-200 行

```cpp
   151:   UseBSBScheduling = hasV60Ops() && EnableBSBSched;
   152: 
   153:   if (isTinyCore()) {
   154:     // Tiny core has a single thread, so back-to-back scheduling is enabled by
   155:     // default.
   156:     if (!EnableBSBSched.getPosition())
   157:       UseBSBScheduling = false;
   158:   }
   159: 
   160:   FeatureBitset FeatureBits = getFeatureBits();
   161:   if (HexagonDisableDuplex)
   162:     setFeatureBits(FeatureBits.reset(Hexagon::FeatureDuplex));
   163:   setFeatureBits(Hexagon_MC::completeHVXFeatures(FeatureBits));
   164: 
   165:   return *this;
   166: }
   167: 
   168: bool HexagonSubtarget::isHVXElementType(MVT Ty, bool IncludeBool) const {
   169:   if (!useHVXOps())
   170:     return false;
   171:   if (Ty.isVector())
   172:     Ty = Ty.getVectorElementType();
   173:   if (IncludeBool && Ty == MVT::i1)
   174:     return true;
   175:   ArrayRef<MVT> ElemTypes = getHVXElementTypes();
   176:   return llvm::is_contained(ElemTypes, Ty);
   177: }
   178: 
   179: bool HexagonSubtarget::isHVXVectorType(EVT VecTy, bool IncludeBool) const {
   180:   if (!VecTy.isSimple())
   181:     return false;
   182:   if (!VecTy.isVector() || !useHVXOps() || VecTy.isScalableVector())
   183:     return false;
   184:   MVT ElemTy = VecTy.getSimpleVT().getVectorElementType();
   185:   if (!IncludeBool && ElemTy == MVT::i1)
   186:     return false;
   187: 
   188:   unsigned HwLen = getVectorLength();
   189:   unsigned NumElems = VecTy.getVectorNumElements();
   190:   ArrayRef<MVT> ElemTypes = getHVXElementTypes();
   191: 
   192:   if (IncludeBool && ElemTy == MVT::i1) {
   193:     // Boolean HVX vector types are formed from regular HVX vector types
   194:     // by replacing the element type with i1.
   195:     for (MVT T : ElemTypes)
   196:       if (NumElems * T.getSizeInBits() == 8 * HwLen)
   197:         return true;
   198:     return false;
   199:   }
   200: 
```
- EN: It declares or implements routines such as getFeatureBits, setFeatureBits, HexagonSubtarget::isHVXElementType, getHVXElementTypes, ... (9 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonDisableDuplex, Hexagon_MC, HexagonSubtarget, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getFeatureBits, setFeatureBits, HexagonSubtarget::isHVXElementType, getHVXElementTypes, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonDisableDuplex, Hexagon_MC, HexagonSubtarget，说明了它与同级后端组件的连接关系。

### Lines 201-250 / 第 201-250 行

```cpp
   201:   unsigned VecWidth = VecTy.getSizeInBits();
   202:   if (VecWidth != 8 * HwLen && VecWidth != 16 * HwLen)
   203:     return false;
   204:   return llvm::is_contained(ElemTypes, ElemTy);
   205: }
   206: 
   207: bool HexagonSubtarget::isTypeForHVX(Type *VecTy, bool IncludeBool) const {
   208:   if (!VecTy->isVectorTy() || isa<ScalableVectorType>(VecTy))
   209:     return false;
   210:   // Avoid types like <2 x i32*>.
   211:   Type *ScalTy = VecTy->getScalarType();
   212:   if (!ScalTy->isIntegerTy() &&
   213:       !(ScalTy->isFloatingPointTy() && useHVXFloatingPoint()))
   214:     return false;
   215:   // The given type may be something like <17 x i32>, which is not MVT,
   216:   // but can be represented as (non-simple) EVT.
   217:   EVT Ty = EVT::getEVT(VecTy, /*HandleUnknown*/false);
   218:   if (!Ty.getVectorElementType().isSimple())
   219:     return false;
   220: 
   221:   auto isHvxTy = [this, IncludeBool](MVT SimpleTy) {
   222:     if (isHVXVectorType(SimpleTy, IncludeBool))
   223:       return true;
   224:     auto Action = getTargetLowering()->getPreferredVectorAction(SimpleTy);
   225:     return Action == TargetLoweringBase::TypeWidenVector;
   226:   };
   227: 
   228:   // Round up EVT to have power-of-2 elements, and keep checking if it
   229:   // qualifies for HVX, dividing it in half after each step.
   230:   MVT ElemTy = Ty.getVectorElementType().getSimpleVT();
   231:   unsigned VecLen = PowerOf2Ceil(Ty.getVectorNumElements());
   232:   while (VecLen > 1) {
   233:     MVT SimpleTy = MVT::getVectorVT(ElemTy, VecLen);
   234:     if (SimpleTy.isValid() && isHvxTy(SimpleTy))
   235:       return true;
   236:     VecLen /= 2;
   237:   }
   238: 
   239:   return false;
   240: }
   241: 
   242: void HexagonSubtarget::UsrOverflowMutation::apply(ScheduleDAGInstrs *DAG) {
   243:   for (SUnit &SU : DAG->SUnits) {
   244:     if (!SU.isInstr())
   245:       continue;
   246:     SmallVector<SDep, 4> Erase;
   247:     for (auto &D : SU.Preds)
   248:       if (D.getKind() == SDep::Output && D.getReg() == Hexagon::USR_OVF)
   249:         Erase.push_back(D);
   250:     for (auto &E : Erase)
```
- EN: It declares or implements routines such as getSizeInBits, llvm::is_contained, HexagonSubtarget::isTypeForHVX, getScalarType, ... (10 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonSubtarget, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getSizeInBits, llvm::is_contained, HexagonSubtarget::isTypeForHVX, getScalarType, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonSubtarget，说明了它与同级后端组件的连接关系。

### Lines 251-300 / 第 251-300 行

```cpp
   251:       SU.removePred(E);
   252:   }
   253: }
   254: 
   255: void HexagonSubtarget::HVXMemLatencyMutation::apply(ScheduleDAGInstrs *DAG) {
   256:   for (SUnit &SU : DAG->SUnits) {
   257:     // Update the latency of chain edges between v60 vector load or store
   258:     // instructions to be 1. These instruction cannot be scheduled in the
   259:     // same packet.
   260:     MachineInstr &MI1 = *SU.getInstr();
   261:     auto *QII = static_cast<const HexagonInstrInfo*>(DAG->TII);
   262:     bool IsStoreMI1 = MI1.mayStore();
   263:     bool IsLoadMI1 = MI1.mayLoad();
   264:     if (!QII->isHVXVec(MI1) || !(IsStoreMI1 || IsLoadMI1))
   265:       continue;
   266:     for (SDep &SI : SU.Succs) {
   267:       if (SI.getKind() != SDep::Order || SI.getLatency() != 0)
   268:         continue;
   269:       MachineInstr &MI2 = *SI.getSUnit()->getInstr();
   270:       if (!QII->isHVXVec(MI2))
   271:         continue;
   272:       if ((IsStoreMI1 && MI2.mayStore()) || (IsLoadMI1 && MI2.mayLoad())) {
   273:         SI.setLatency(1);
   274:         SU.setHeightDirty();
   275:         // Change the dependence in the opposite direction too.
   276:         for (SDep &PI : SI.getSUnit()->Preds) {
   277:           if (PI.getSUnit() != &SU || PI.getKind() != SDep::Order)
   278:             continue;
   279:           PI.setLatency(1);
   280:           SI.getSUnit()->setDepthDirty();
   281:         }
   282:       }
   283:     }
   284:   }
   285: }
   286: 
   287: // Check if a call and subsequent A2_tfrpi instructions should maintain
   288: // scheduling affinity. We are looking for the TFRI to be consumed in
   289: // the next instruction. This should help reduce the instances of
   290: // double register pairs being allocated and scheduled before a call
   291: // when not used until after the call. This situation is exacerbated
   292: // by the fact that we allocate the pair from the callee saves list,
   293: // leading to excess spills and restores.
   294: bool HexagonSubtarget::CallMutation::shouldTFRICallBind(
   295:       const HexagonInstrInfo &HII, const SUnit &Inst1,
   296:       const SUnit &Inst2) const {
   297:   if (Inst1.getInstr()->getOpcode() != Hexagon::A2_tfrpi)
   298:     return false;
   299: 
   300:   // TypeXTYPE are 64 bit operations.
```
- EN: It declares or implements routines such as removePred, HexagonSubtarget::HVXMemLatencyMutation::apply, getInstr, mayStore, ... (9 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonSubtarget, HexagonInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 removePred, HexagonSubtarget::HVXMemLatencyMutation::apply, getInstr, mayStore, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonSubtarget, HexagonInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 301-350 / 第 301-350 行

```cpp
   301:   unsigned Type = HII.getType(*Inst2.getInstr());
   302:   return Type == HexagonII::TypeS_2op || Type == HexagonII::TypeS_3op ||
   303:          Type == HexagonII::TypeALU64 || Type == HexagonII::TypeM;
   304: }
   305: 
   306: void HexagonSubtarget::CallMutation::apply(ScheduleDAGInstrs *DAGInstrs) {
   307:   ScheduleDAGMI *DAG = static_cast<ScheduleDAGMI*>(DAGInstrs);
   308:   SUnit* LastSequentialCall = nullptr;
   309:   // Map from virtual register to physical register from the copy.
   310:   DenseMap<unsigned, unsigned> VRegHoldingReg;
   311:   // Map from the physical register to the instruction that uses virtual
   312:   // register. This is used to create the barrier edge.
   313:   DenseMap<unsigned, SUnit *> LastVRegUse;
   314:   auto &TRI = *DAG->MF.getSubtarget().getRegisterInfo();
   315:   auto &HII = *DAG->MF.getSubtarget<HexagonSubtarget>().getInstrInfo();
   316: 
   317:   // Currently we only catch the situation when compare gets scheduled
   318:   // before preceding call.
   319:   for (unsigned su = 0, e = DAG->SUnits.size(); su != e; ++su) {
   320:     // Remember the call.
   321:     if (DAG->SUnits[su].getInstr()->isCall())
   322:       LastSequentialCall = &DAG->SUnits[su];
   323:     // Look for a compare that defines a predicate.
   324:     else if (DAG->SUnits[su].getInstr()->isCompare() && LastSequentialCall)
   325:       DAG->addEdge(&DAG->SUnits[su], SDep(LastSequentialCall, SDep::Barrier));
   326:     // Look for call and tfri* instructions.
   327:     else if (SchedPredsCloser && LastSequentialCall && su > 1 && su < e-1 &&
   328:              shouldTFRICallBind(HII, DAG->SUnits[su], DAG->SUnits[su+1]))
   329:       DAG->addEdge(&DAG->SUnits[su], SDep(&DAG->SUnits[su-1], SDep::Barrier));
   330:     // Prevent redundant register copies due to reads and writes of physical
   331:     // registers. The original motivation for this was the code generated
   332:     // between two calls, which are caused both the return value and the
   333:     // argument for the next call being in %r0.
   334:     // Example:
   335:     //   1: <call1>
   336:     //   2: %vreg = COPY %r0
   337:     //   3: <use of %vreg>
   338:     //   4: %r0 = ...
   339:     //   5: <call2>
   340:     // The scheduler would often swap 3 and 4, so an additional register is
   341:     // needed. This code inserts a Barrier dependence between 3 & 4 to prevent
   342:     // this.
   343:     // The code below checks for all the physical registers, not just R0/D0/V0.
   344:     else if (SchedRetvalOptimization) {
   345:       const MachineInstr *MI = DAG->SUnits[su].getInstr();
   346:       if (MI->isCopy() && MI->getOperand(1).getReg().isPhysical()) {
   347:         // %vregX = COPY %r0
   348:         VRegHoldingReg[MI->getOperand(0).getReg()] = MI->getOperand(1).getReg();
   349:         LastVRegUse.erase(MI->getOperand(1).getReg());
   350:       } else {
```
- EN: It declares or implements routines such as getType, HexagonSubtarget::CallMutation::apply, getSubtarget, getSubtarget<HexagonSubtarget>, ... (7 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonII, HexagonSubtarget, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getType, HexagonSubtarget::CallMutation::apply, getSubtarget, getSubtarget<HexagonSubtarget>, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonII, HexagonSubtarget，说明了它与同级后端组件的连接关系。

### Lines 351-400 / 第 351-400 行

```cpp
   351:         for (const MachineOperand &MO : MI->operands()) {
   352:           if (!MO.isReg())
   353:             continue;
   354:           if (MO.isUse() && !MI->isCopy() &&
   355:               VRegHoldingReg.count(MO.getReg())) {
   356:             // <use of %vregX>
   357:             LastVRegUse[VRegHoldingReg[MO.getReg()]] = &DAG->SUnits[su];
   358:           } else if (MO.isDef() && MO.getReg().isPhysical()) {
   359:             for (MCRegAliasIterator AI(MO.getReg(), &TRI, true); AI.isValid();
   360:                  ++AI) {
   361:               if (auto It = LastVRegUse.find(*AI); It != LastVRegUse.end()) {
   362:                 if (It->second != &DAG->SUnits[su])
   363:                   // %r0 = ...
   364:                   DAG->addEdge(&DAG->SUnits[su],
   365:                                SDep(It->second, SDep::Barrier));
   366:                 LastVRegUse.erase(It);
   367:               }
   368:             }
   369:           }
   370:         }
   371:       }
   372:     }
   373:   }
   374: }
   375: 
   376: void HexagonSubtarget::BankConflictMutation::apply(ScheduleDAGInstrs *DAG) {
   377:   if (!EnableCheckBankConflict)
   378:     return;
   379: 
   380:   const auto &HII = static_cast<const HexagonInstrInfo&>(*DAG->TII);
   381: 
   382:   // Create artificial edges between loads that could likely cause a bank
   383:   // conflict. Since such loads would normally not have any dependency
   384:   // between them, we cannot rely on existing edges.
   385:   for (unsigned i = 0, e = DAG->SUnits.size(); i != e; ++i) {
   386:     SUnit &S0 = DAG->SUnits[i];
   387:     MachineInstr &L0 = *S0.getInstr();
   388:     if (!L0.mayLoad() || L0.mayStore() ||
   389:         HII.getAddrMode(L0) != HexagonII::BaseImmOffset)
   390:       continue;
   391:     int64_t Offset0;
   392:     LocationSize Size0 = LocationSize::precise(0);
   393:     MachineOperand *BaseOp0 = HII.getBaseAndOffset(L0, Offset0, Size0);
   394:     // Is the access size is longer than the L1 cache line, skip the check.
   395:     if (BaseOp0 == nullptr || !BaseOp0->isReg() || !Size0.hasValue() ||
   396:         Size0.getValue() >= 32)
   397:       continue;
   398:     // Scan only up to 32 instructions ahead (to avoid n^2 complexity).
   399:     for (unsigned j = i+1, m = std::min(i+32, e); j != m; ++j) {
   400:       SUnit &S1 = DAG->SUnits[j];
```
- EN: It declares or implements routines such as isValid, end, erase, HexagonSubtarget::BankConflictMutation::apply, ... (8 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonSubtarget, HexagonInstrInfo, HexagonII, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 isValid, end, erase, HexagonSubtarget::BankConflictMutation::apply, ... (8 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonSubtarget, HexagonInstrInfo, HexagonII，说明了它与同级后端组件的连接关系。

### Lines 401-450 / 第 401-450 行

```cpp
   401:       MachineInstr &L1 = *S1.getInstr();
   402:       if (!L1.mayLoad() || L1.mayStore() ||
   403:           HII.getAddrMode(L1) != HexagonII::BaseImmOffset)
   404:         continue;
   405:       int64_t Offset1;
   406:       LocationSize Size1 = LocationSize::precise(0);
   407:       MachineOperand *BaseOp1 = HII.getBaseAndOffset(L1, Offset1, Size1);
   408:       if (BaseOp1 == nullptr || !BaseOp1->isReg() || !Size0.hasValue() ||
   409:           Size1.getValue() >= 32 || BaseOp0->getReg() != BaseOp1->getReg())
   410:         continue;
   411:       // Check bits 3 and 4 of the offset: if they differ, a bank conflict
   412:       // is unlikely.
   413:       if (((Offset0 ^ Offset1) & 0x18) != 0)
   414:         continue;
   415:       // Bits 3 and 4 are the same, add an artificial edge and set extra
   416:       // latency.
   417:       SDep A(&S0, SDep::Artificial);
   418:       A.setLatency(1);
   419:       S1.addPred(A, true);
   420:     }
   421:   }
   422: }
   423: 
   424: /// Enable use of alias analysis during code generation (during MI
   425: /// scheduling, DAGCombine, etc.).
   426: bool HexagonSubtarget::useAA() const {
   427:   if (OptLevel != CodeGenOptLevel::None)
   428:     return true;
   429:   return false;
   430: }
   431: 
   432: /// Perform target specific adjustments to the latency of a schedule
   433: /// dependency.
   434: void HexagonSubtarget::adjustSchedDependency(
   435:     SUnit *Src, int SrcOpIdx, SUnit *Dst, int DstOpIdx, SDep &Dep,
   436:     const TargetSchedModel *SchedModel) const {
   437:   if (!Src->isInstr() || !Dst->isInstr())
   438:     return;
   439: 
   440:   MachineInstr *SrcInst = Src->getInstr();
   441:   MachineInstr *DstInst = Dst->getInstr();
   442:   const HexagonInstrInfo *QII = getInstrInfo();
   443: 
   444:   // Instructions with .new operands have zero latency.
   445:   SmallPtrSet<SUnit *, 4> ExclSrc;
   446:   SmallPtrSet<SUnit *, 4> ExclDst;
   447:   if (QII->canExecuteInBundle(*SrcInst, *DstInst) &&
   448:       isBestZeroLatency(Src, Dst, QII, ExclSrc, ExclDst)) {
   449:     Dep.setLatency(0);
   450:     return;
```
- EN: It declares or implements routines such as getInstr, LocationSize::precise, getBaseAndOffset, A, ... (9 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonII, HexagonSubtarget, HexagonInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getInstr, LocationSize::precise, getBaseAndOffset, A, ... (9 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonII, HexagonSubtarget, HexagonInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 451-500 / 第 451-500 行

```cpp
   451:   }
   452: 
   453:   // Set the latency for a copy to zero since we hope that is will get
   454:   // removed.
   455:   if (DstInst->isCopy())
   456:     Dep.setLatency(0);
   457: 
   458:   // If it's a REG_SEQUENCE/COPY, use its destination instruction to determine
   459:   // the correct latency.
   460:   // If there are multiple uses of the def of COPY/REG_SEQUENCE, set the latency
   461:   // only if the latencies on all the uses are equal, otherwise set it to
   462:   // default.
   463:   if ((DstInst->isRegSequence() || DstInst->isCopy())) {
   464:     Register DReg = DstInst->getOperand(0).getReg();
   465:     std::optional<unsigned> DLatency;
   466:     for (const auto &DDep : Dst->Succs) {
   467:       MachineInstr *DDst = DDep.getSUnit()->getInstr();
   468:       int UseIdx = -1;
   469:       for (unsigned OpNum = 0; OpNum < DDst->getNumOperands(); OpNum++) {
   470:         const MachineOperand &MO = DDst->getOperand(OpNum);
   471:         if (MO.isReg() && MO.getReg() && MO.isUse() && MO.getReg() == DReg) {
   472:           UseIdx = OpNum;
   473:           break;
   474:         }
   475:       }
   476: 
   477:       if (UseIdx == -1)
   478:         continue;
   479: 
   480:       std::optional<unsigned> Latency =
   481:           InstrInfo.getOperandLatency(&InstrItins, *SrcInst, 0, *DDst, UseIdx);
   482: 
   483:       // Set DLatency for the first time.
   484:       if (!DLatency)
   485:         DLatency = Latency;
   486: 
   487:       // For multiple uses, if the Latency is different across uses, reset
   488:       // DLatency.
   489:       if (DLatency != Latency) {
   490:         DLatency = std::nullopt;
   491:         break;
   492:       }
   493:     }
   494:     Dep.setLatency(DLatency.value_or(0));
   495:   }
   496: 
   497:   // Try to schedule uses near definitions to generate .cur.
   498:   ExclSrc.clear();
   499:   ExclDst.clear();
   500:   if (EnableDotCurSched && QII->isToBeScheduledASAP(*SrcInst, *DstInst) &&
```
- EN: It declares or implements routines such as getOperand, getSUnit, getNumOperands, getOperandLatency, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 getOperand, getSUnit, getNumOperands, getOperandLatency, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 501-550 / 第 501-550 行

```cpp
   501:       isBestZeroLatency(Src, Dst, QII, ExclSrc, ExclDst)) {
   502:     Dep.setLatency(0);
   503:     return;
   504:   }
   505:   int Latency = Dep.getLatency();
   506:   bool IsArtificial = Dep.isArtificial();
   507:   Latency = updateLatency(*SrcInst, *DstInst, IsArtificial, Latency);
   508:   Dep.setLatency(Latency);
   509: }
   510: 
   511: void HexagonSubtarget::getPostRAMutations(
   512:     std::vector<std::unique_ptr<ScheduleDAGMutation>> &Mutations) const {
   513:   Mutations.push_back(std::make_unique<UsrOverflowMutation>());
   514:   Mutations.push_back(std::make_unique<HVXMemLatencyMutation>());
   515:   Mutations.push_back(std::make_unique<BankConflictMutation>());
   516: }
   517: 
   518: void HexagonSubtarget::getSMSMutations(
   519:     std::vector<std::unique_ptr<ScheduleDAGMutation>> &Mutations) const {
   520:   Mutations.push_back(std::make_unique<UsrOverflowMutation>());
   521:   Mutations.push_back(std::make_unique<HVXMemLatencyMutation>());
   522: }
   523: 
   524: // Pin the vtable to this file.
   525: void HexagonSubtarget::anchor() {}
   526: 
   527: bool HexagonSubtarget::enableMachineScheduler() const {
   528:   if (DisableHexagonMISched.getNumOccurrences())
   529:     return !DisableHexagonMISched;
   530:   return true;
   531: }
   532: 
   533: bool HexagonSubtarget::usePredicatedCalls() const {
   534:   return EnablePredicatedCalls;
   535: }
   536: 
   537: int HexagonSubtarget::updateLatency(MachineInstr &SrcInst,
   538:                                     MachineInstr &DstInst, bool IsArtificial,
   539:                                     int Latency) const {
   540:   if (IsArtificial)
   541:     return 1;
   542:   if (!hasV60Ops())
   543:     return Latency;
   544: 
   545:   const HexagonInstrInfo &QII = *getInstrInfo();
   546:   // BSB scheduling.
   547:   if (QII.isHVXVec(SrcInst) || useBSBScheduling())
   548:     Latency = (Latency + 1) >> 1;
   549:   return Latency;
   550: }
```
- EN: It declares or implements routines such as isBestZeroLatency, setLatency, getLatency, isArtificial, ... (13 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonSubtarget, HexagonInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 isBestZeroLatency, setLatency, getLatency, isArtificial, ... (13 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonSubtarget, HexagonInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 551-600 / 第 551-600 行

```cpp
   551: 
   552: void HexagonSubtarget::restoreLatency(SUnit *Src, SUnit *Dst) const {
   553:   MachineInstr *SrcI = Src->getInstr();
   554:   for (auto &I : Src->Succs) {
   555:     if (!I.isAssignedRegDep() || I.getSUnit() != Dst)
   556:       continue;
   557:     Register DepR = I.getReg();
   558:     int DefIdx = -1;
   559:     for (unsigned OpNum = 0; OpNum < SrcI->getNumOperands(); OpNum++) {
   560:       const MachineOperand &MO = SrcI->getOperand(OpNum);
   561:       bool IsSameOrSubReg = false;
   562:       if (MO.isReg()) {
   563:         Register MOReg = MO.getReg();
   564:         if (DepR.isVirtual()) {
   565:           IsSameOrSubReg = (MOReg == DepR);
   566:         } else {
   567:           IsSameOrSubReg = getRegisterInfo()->isSubRegisterEq(DepR, MOReg);
   568:         }
   569:         if (MO.isDef() && IsSameOrSubReg)
   570:           DefIdx = OpNum;
   571:       }
   572:     }
   573:     assert(DefIdx >= 0 && "Def Reg not found in Src MI");
   574:     MachineInstr *DstI = Dst->getInstr();
   575:     SDep T = I;
   576:     for (unsigned OpNum = 0; OpNum < DstI->getNumOperands(); OpNum++) {
   577:       const MachineOperand &MO = DstI->getOperand(OpNum);
   578:       if (MO.isReg() && MO.isUse() && MO.getReg() == DepR) {
   579:         std::optional<unsigned> Latency = InstrInfo.getOperandLatency(
   580:             &InstrItins, *SrcI, DefIdx, *DstI, OpNum);
   581: 
   582:         // For some instructions (ex: COPY), we might end up with < 0 latency
   583:         // as they don't have any Itinerary class associated with them.
   584:         if (!Latency)
   585:           Latency = 0;
   586:         bool IsArtificial = I.isArtificial();
   587:         Latency = updateLatency(*SrcI, *DstI, IsArtificial, *Latency);
   588:         I.setLatency(*Latency);
   589:       }
   590:     }
   591: 
   592:     // Update the latency of opposite edge too.
   593:     T.setSUnit(Src);
   594:     auto F = find(Dst->Preds, T);
   595:     assert(F != Dst->Preds.end());
   596:     F->setLatency(I.getLatency());
   597:   }
   598: }
   599: 
   600: /// Change the latency between the two SUnits.
```
- EN: It declares or implements routines such as HexagonSubtarget::restoreLatency, getInstr, getReg, getNumOperands, ... (13 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonSubtarget, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonSubtarget::restoreLatency, getInstr, getReg, getNumOperands, ... (13 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonSubtarget，说明了它与同级后端组件的连接关系。

### Lines 601-650 / 第 601-650 行

```cpp
   601: void HexagonSubtarget::changeLatency(SUnit *Src, SUnit *Dst, unsigned Lat)
   602:       const {
   603:   for (auto &I : Src->Succs) {
   604:     if (!I.isAssignedRegDep() || I.getSUnit() != Dst)
   605:       continue;
   606:     SDep T = I;
   607:     I.setLatency(Lat);
   608: 
   609:     // Update the latency of opposite edge too.
   610:     T.setSUnit(Src);
   611:     auto F = find(Dst->Preds, T);
   612:     assert(F != Dst->Preds.end());
   613:     F->setLatency(Lat);
   614:   }
   615: }
   616: 
   617: /// If the SUnit has a zero latency edge, return the other SUnit.
   618: static SUnit *getZeroLatency(SUnit *N, SmallVector<SDep, 4> &Deps) {
   619:   for (auto &I : Deps)
   620:     if (I.isAssignedRegDep() && I.getLatency() == 0 &&
   621:         !I.getSUnit()->getInstr()->isPseudo())
   622:       return I.getSUnit();
   623:   return nullptr;
   624: }
   625: 
   626: // Return true if these are the best two instructions to schedule
   627: // together with a zero latency. Only one dependence should have a zero
   628: // latency. If there are multiple choices, choose the best, and change
   629: // the others, if needed.
   630: bool HexagonSubtarget::isBestZeroLatency(
   631:     SUnit *Src, SUnit *Dst, const HexagonInstrInfo *TII,
   632:     SmallPtrSet<SUnit *, 4> &ExclSrc, SmallPtrSet<SUnit *, 4> &ExclDst) const {
   633:   MachineInstr &SrcInst = *Src->getInstr();
   634:   MachineInstr &DstInst = *Dst->getInstr();
   635: 
   636:   // Ignore Boundary SU nodes as these have null instructions.
   637:   if (Dst->isBoundaryNode())
   638:     return false;
   639: 
   640:   if (SrcInst.isPHI() || DstInst.isPHI())
   641:     return false;
   642: 
   643:   if (!TII->isToBeScheduledASAP(SrcInst, DstInst) &&
   644:       !TII->canExecuteInBundle(SrcInst, DstInst))
   645:     return false;
   646: 
   647:   // The architecture doesn't allow three dependent instructions in the same
   648:   // packet. So, if the destination has a zero latency successor, then it's
   649:   // not a candidate for a zero latency predecessor.
   650:   if (getZeroLatency(Dst, Dst->Succs) != nullptr)
```
- EN: It declares or implements routines such as HexagonSubtarget::changeLatency, setLatency, setSUnit, find, ... (8 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonSubtarget, HexagonInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonSubtarget::changeLatency, setLatency, setSUnit, find, ... (8 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonSubtarget, HexagonInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 651-700 / 第 651-700 行

```cpp
   651:     return false;
   652: 
   653:   // Check if the Dst instruction is the best candidate first.
   654:   SUnit *Best = nullptr;
   655:   SUnit *DstBest = nullptr;
   656:   SUnit *SrcBest = getZeroLatency(Dst, Dst->Preds);
   657:   if (SrcBest == nullptr || Src->NodeNum >= SrcBest->NodeNum) {
   658:     // Check that Src doesn't have a better candidate.
   659:     DstBest = getZeroLatency(Src, Src->Succs);
   660:     if (DstBest == nullptr || Dst->NodeNum <= DstBest->NodeNum)
   661:       Best = Dst;
   662:   }
   663:   if (Best != Dst)
   664:     return false;
   665: 
   666:   // The caller frequently adds the same dependence twice. If so, then
   667:   // return true for this case too.
   668:   if ((Src == SrcBest && Dst == DstBest ) ||
   669:       (SrcBest == nullptr && Dst == DstBest) ||
   670:       (Src == SrcBest && Dst == nullptr))
   671:     return true;
   672: 
   673:   // Reassign the latency for the previous bests, which requires setting
   674:   // the dependence edge in both directions.
   675:   if (SrcBest != nullptr) {
   676:     if (!hasV60Ops())
   677:       changeLatency(SrcBest, Dst, 1);
   678:     else
   679:       restoreLatency(SrcBest, Dst);
   680:   }
   681:   if (DstBest != nullptr) {
   682:     if (!hasV60Ops())
   683:       changeLatency(Src, DstBest, 1);
   684:     else
   685:       restoreLatency(Src, DstBest);
   686:   }
   687: 
   688:   // Attempt to find another opportunity for zero latency in a different
   689:   // dependence.
   690:   if (SrcBest && DstBest)
   691:     // If there is an edge from SrcBest to DstBst, then try to change that
   692:     // to 0 now.
   693:     changeLatency(SrcBest, DstBest, 0);
   694:   else if (DstBest) {
   695:     // Check if the previous best destination instruction has a new zero
   696:     // latency dependence opportunity.
   697:     ExclSrc.insert(Src);
   698:     for (auto &I : DstBest->Preds)
   699:       if (ExclSrc.count(I.getSUnit()) == 0 &&
   700:           isBestZeroLatency(I.getSUnit(), DstBest, TII, ExclSrc, ExclDst))
```
- EN: It declares or implements routines such as getZeroLatency, restoreLatency, insert, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 getZeroLatency, restoreLatency, insert 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 701-750 / 第 701-750 行

```cpp
   701:         changeLatency(I.getSUnit(), DstBest, 0);
   702:   } else if (SrcBest) {
   703:     // Check if previous best source instruction has a new zero latency
   704:     // dependence opportunity.
   705:     ExclDst.insert(Dst);
   706:     for (auto &I : SrcBest->Succs)
   707:       if (ExclDst.count(I.getSUnit()) == 0 &&
   708:           isBestZeroLatency(SrcBest, I.getSUnit(), TII, ExclSrc, ExclDst))
   709:         changeLatency(SrcBest, I.getSUnit(), 0);
   710:   }
   711: 
   712:   return true;
   713: }
   714: 
   715: unsigned HexagonSubtarget::getL1CacheLineSize() const {
   716:   return 32;
   717: }
   718: 
   719: unsigned HexagonSubtarget::getL1PrefetchDistance() const {
   720:   return 32;
   721: }
   722: 
   723: bool HexagonSubtarget::enableSubRegLiveness() const { return true; }
   724: 
   725: Intrinsic::ID HexagonSubtarget::getIntrinsicId(unsigned Opc) const {
   726:   struct Scalar {
   727:     unsigned Opcode;
   728:     Intrinsic::ID IntId;
   729:   };
   730:   struct Hvx {
   731:     unsigned Opcode;
   732:     Intrinsic::ID Int64Id, Int128Id;
   733:   };
   734: 
   735:   static Scalar ScalarInts[] = {
   736: #define GET_SCALAR_INTRINSICS
   737: #include "HexagonDepInstrIntrinsics.inc"
   738: #undef GET_SCALAR_INTRINSICS
   739:   };
   740: 
   741:   static Hvx HvxInts[] = {
   742: #define GET_HVX_INTRINSICS
   743: #include "HexagonDepInstrIntrinsics.inc"
   744: #undef GET_HVX_INTRINSICS
   745:   };
   746: 
   747:   const auto CmpOpcode = [](auto A, auto B) { return A.Opcode < B.Opcode; };
   748:   [[maybe_unused]] static bool SortedScalar =
   749:       (llvm::sort(ScalarInts, CmpOpcode), true);
   750:   [[maybe_unused]] static bool SortedHvx =
```
- EN: It imports headers such as HexagonDepInstrIntrinsics.inc, establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It declares types such as Scalar, Hvx, which carry the state or API of this component. It declares or implements routines such as changeLatency, insert, HexagonSubtarget::getL1CacheLineSize, HexagonSubtarget::getL1PrefetchDistance, ... (7 total), translating Hexagon-specific policy into reusable code paths.
- CN: 这里引入了 HexagonDepInstrIntrinsics.inc 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里声明了 Scalar, Hvx 等类型，用来承载该组件的状态或接口。 这里声明或实现了 changeLatency, insert, HexagonSubtarget::getL1CacheLineSize, HexagonSubtarget::getL1PrefetchDistance, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 751-772 / 第 751-772 行

```cpp
   751:       (llvm::sort(HvxInts, CmpOpcode), true);
   752: 
   753:   auto [BS, ES] = std::make_pair(std::begin(ScalarInts), std::end(ScalarInts));
   754:   auto [BH, EH] = std::make_pair(std::begin(HvxInts), std::end(HvxInts));
   755: 
   756:   auto FoundScalar = std::lower_bound(BS, ES, Scalar{Opc, 0}, CmpOpcode);
   757:   if (FoundScalar != ES && FoundScalar->Opcode == Opc)
   758:     return FoundScalar->IntId;
   759: 
   760:   auto FoundHvx = std::lower_bound(BH, EH, Hvx{Opc, 0, 0}, CmpOpcode);
   761:   if (FoundHvx != EH && FoundHvx->Opcode == Opc) {
   762:     unsigned HwLen = getVectorLength();
   763:     if (HwLen == 64)
   764:       return FoundHvx->Int64Id;
   765:     if (HwLen == 128)
   766:       return FoundHvx->Int128Id;
   767:   }
   768: 
   769:   std::string error = "Invalid opcode (" + std::to_string(Opc) + ")";
   770:   llvm_unreachable(error.c_str());
   771:   return 0;
   772: }
```
- EN: It declares or implements routines such as llvm::sort, std::make_pair, getVectorLength, llvm_unreachable, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 llvm::sort, std::make_pair, getVectorLength, llvm_unreachable 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

## Key Concepts / 关键概念

- HVX vector ISA modeling / HVX 向量 ISA 建模
- instruction scheduling models / 指令调度模型
- register modeling / 寄存器建模
- instruction semantics / 指令语义
- MC-layer target description / MC 层目标描述
- target pipeline configuration / 目标流水线配置

## Dependencies / 依赖关系

- Direct includes / 直接包含: `HexagonSubtarget.h, HexagonInstrInfo.h, HexagonRegisterInfo.h, MCTargetDesc/HexagonMCTargetDesc.h, llvm/ADT/STLExtras.h, llvm/ADT/SmallVector.h, llvm/ADT/StringRef.h, llvm/CodeGen/MachineInstr.h, llvm/CodeGen/MachineOperand.h, llvm/CodeGen/MachineScheduler.h, ... (21 total)`
- Hexagon symbols / Hexagon 符号: `HexagonSubtarget, HexagonInstrInfo, HexagonRegisterInfo, HexagonMCTargetDesc, HexagonGenSubtargetInfo, Hexagon_MC, HexagonArchVersion, HexagonDisableDuplex, HexagonII, HexagonDepInstrIntrinsics`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
