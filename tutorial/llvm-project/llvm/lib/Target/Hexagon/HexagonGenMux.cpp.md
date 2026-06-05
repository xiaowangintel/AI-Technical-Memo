# HexagonGenMux.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonGenMux.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Return Hexagon::DoubleRegsRegClass.contains(Reg);
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```cpp
     1: //===- HexagonGenMux.cpp --------------------------------------------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: 
     9: // During instruction selection, MUX instructions are generated for
    10: // conditional assignments. Since such assignments often present an
    11: // opportunity to predicate instructions, HexagonExpandCondsets
    12: // expands MUXes into pairs of conditional transfers, and then proceeds
    13: // with predication of the producers/consumers of the registers involved.
    14: // This happens after exiting from the SSA form, but before the machine
    15: // instruction scheduler. After the scheduler and after the register
    16: // allocation there can be cases of pairs of conditional transfers
    17: // resulting from a MUX where neither of them was further predicated. If
    18: // these transfers are now placed far enough from the instruction defining
    19: // the predicate register, they cannot use the .new form. In such cases it
    20: // is better to collapse them back to a single MUX instruction.
    21: 
    22: #include "Hexagon.h"
    23: #include "HexagonInstrInfo.h"
    24: #include "HexagonRegisterInfo.h"
    25: #include "HexagonSubtarget.h"
    26: #include "llvm/ADT/BitVector.h"
    27: #include "llvm/ADT/DenseMap.h"
    28: #include "llvm/ADT/SmallVector.h"
    29: #include "llvm/ADT/StringRef.h"
    30: #include "llvm/CodeGen/LiveRegUnits.h"
    31: #include "llvm/CodeGen/MachineBasicBlock.h"
    32: #include "llvm/CodeGen/MachineFunction.h"
    33: #include "llvm/CodeGen/MachineFunctionPass.h"
    34: #include "llvm/CodeGen/MachineInstr.h"
    35: #include "llvm/CodeGen/MachineInstrBuilder.h"
    36: #include "llvm/CodeGen/MachineOperand.h"
    37: #include "llvm/IR/DebugLoc.h"
    38: #include "llvm/MC/MCInstrDesc.h"
    39: #include "llvm/Pass.h"
    40: #include "llvm/Support/CommandLine.h"
    41: #include "llvm/Support/MathExtras.h"
    42: #include <algorithm>
    43: #include <cassert>
    44: #include <iterator>
    45: #include <limits>
    46: 
    47: #define DEBUG_TYPE "hexmux"
    48: 
    49: using namespace llvm;
    50: 
```
- EN: It imports headers such as Hexagon.h, HexagonInstrInfo.h, HexagonRegisterInfo.h, HexagonSubtarget.h, ... (24 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里引入了 Hexagon.h, HexagonInstrInfo.h, HexagonRegisterInfo.h, HexagonSubtarget.h, ... (24 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 51-100 / 第 51-100 行

```cpp
    51: // Initialize this to 0 to always prefer generating mux by default.
    52: static cl::opt<unsigned> MinPredDist("hexagon-gen-mux-threshold", cl::Hidden,
    53:   cl::init(0), cl::desc("Minimum distance between predicate definition and "
    54:   "farther of the two predicated uses"));
    55: 
    56: namespace {
    57: 
    58:   class HexagonGenMux : public MachineFunctionPass {
    59:   public:
    60:     static char ID;
    61: 
    62:     HexagonGenMux() : MachineFunctionPass(ID) {}
    63: 
    64:     StringRef getPassName() const override {
    65:       return "Hexagon generate mux instructions";
    66:     }
    67: 
    68:     void getAnalysisUsage(AnalysisUsage &AU) const override {
    69:       MachineFunctionPass::getAnalysisUsage(AU);
    70:     }
    71: 
    72:     bool runOnMachineFunction(MachineFunction &MF) override;
    73: 
    74:     MachineFunctionProperties getRequiredProperties() const override {
    75:       return MachineFunctionProperties().setNoVRegs();
    76:     }
    77: 
    78:   private:
    79:     const HexagonInstrInfo *HII = nullptr;
    80:     const HexagonRegisterInfo *HRI = nullptr;
    81: 
    82:     struct CondsetInfo {
    83:       unsigned PredR = 0;
    84:       unsigned TrueX = std::numeric_limits<unsigned>::max();
    85:       unsigned FalseX = std::numeric_limits<unsigned>::max();
    86: 
    87:       CondsetInfo() = default;
    88:     };
    89: 
    90:     struct DefUseInfo {
    91:       BitVector Defs, Uses;
    92: 
    93:       DefUseInfo() = default;
    94:       DefUseInfo(const BitVector &D, const BitVector &U) : Defs(D), Uses(U) {}
    95:     };
    96: 
    97:     struct MuxInfo {
    98:       MachineBasicBlock::iterator At;
    99:       unsigned DefR, PredR;
   100:       MachineOperand *SrcT, *SrcF;
```
- EN: It declares types such as HexagonGenMux, CondsetInfo, DefUseInfo, MuxInfo, which carry the state or API of this component. It defines declarative TableGen records like HexagonGenMux; these records are consumed by TableGen instead of executed directly. It declares or implements routines such as MinPredDist, HexagonGenMux, getPassName, getAnalysisUsage, ... (11 total), translating Hexagon-specific policy into reusable code paths. Command-line options appear here to gate diagnostics or target-specific tuning behavior.
- CN: 这里声明了 HexagonGenMux, CondsetInfo, DefUseInfo, MuxInfo 等类型，用来承载该组件的状态或接口。 这里定义了 HexagonGenMux 等声明式 TableGen 记录；这些记录由 TableGen 消费，而不是直接执行。 这里声明或实现了 MinPredDist, HexagonGenMux, getPassName, getAnalysisUsage, ... (11 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里出现了命令行选项，用于控制诊断开关或目标相关的调优行为。

### Lines 101-150 / 第 101-150 行

```cpp
   101:       MachineInstr *Def1, *Def2;
   102: 
   103:       MuxInfo(MachineBasicBlock::iterator It, unsigned DR, unsigned PR,
   104:               MachineOperand *TOp, MachineOperand *FOp, MachineInstr &D1,
   105:               MachineInstr &D2)
   106:           : At(It), DefR(DR), PredR(PR), SrcT(TOp), SrcF(FOp), Def1(&D1),
   107:             Def2(&D2) {}
   108:     };
   109: 
   110:     using InstrIndexMap = DenseMap<MachineInstr *, unsigned>;
   111:     using DefUseInfoMap = DenseMap<unsigned, DefUseInfo>;
   112:     using MuxInfoList = SmallVector<MuxInfo, 4>;
   113: 
   114:     bool isRegPair(unsigned Reg) const {
   115:       return Hexagon::DoubleRegsRegClass.contains(Reg);
   116:     }
   117: 
   118:     void getSubRegs(unsigned Reg, BitVector &SRs) const;
   119:     void expandReg(unsigned Reg, BitVector &Set) const;
   120:     void getDefsUses(const MachineInstr *MI, BitVector &Defs,
   121:           BitVector &Uses) const;
   122:     void buildMaps(MachineBasicBlock &B, InstrIndexMap &I2X,
   123:           DefUseInfoMap &DUM);
   124:     bool isCondTransfer(unsigned Opc) const;
   125:     unsigned getMuxOpcode(const MachineOperand &Src1,
   126:           const MachineOperand &Src2) const;
   127:     bool genMuxInBlock(MachineBasicBlock &B);
   128:   };
   129: 
   130: } // end anonymous namespace
   131: 
   132: char HexagonGenMux::ID = 0;
   133: 
   134: INITIALIZE_PASS(HexagonGenMux, "hexagon-gen-mux",
   135:   "Hexagon generate mux instructions", false, false)
   136: 
   137: void HexagonGenMux::getSubRegs(unsigned Reg, BitVector &SRs) const {
   138:   for (MCPhysReg I : HRI->subregs(Reg))
   139:     SRs[I] = true;
   140: }
   141: 
   142: void HexagonGenMux::expandReg(unsigned Reg, BitVector &Set) const {
   143:   if (isRegPair(Reg))
   144:     getSubRegs(Reg, Set);
   145:   else
   146:     Set[Reg] = true;
   147: }
   148: 
   149: void HexagonGenMux::getDefsUses(const MachineInstr *MI, BitVector &Defs,
   150:       BitVector &Uses) const {
```
- EN: It opens namespaces (char) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as MuxInfo, isRegPair, contains, getSubRegs, ... (13 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonGenMux, showing how the code connects to sibling backend components.
- CN: 这里打开了命名空间（char），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 MuxInfo, isRegPair, contains, getSubRegs, ... (13 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonGenMux，说明了它与同级后端组件的连接关系。

### Lines 151-200 / 第 151-200 行

```cpp
   151:   // First, get the implicit defs and uses for this instruction.
   152:   unsigned Opc = MI->getOpcode();
   153:   const MCInstrDesc &D = HII->get(Opc);
   154:   for (MCPhysReg R : D.implicit_defs())
   155:     expandReg(R, Defs);
   156:   for (MCPhysReg R : D.implicit_uses())
   157:     expandReg(R, Uses);
   158: 
   159:   // Look over all operands, and collect explicit defs and uses.
   160:   for (const MachineOperand &MO : MI->operands()) {
   161:     if (!MO.isReg() || MO.isImplicit())
   162:       continue;
   163:     Register R = MO.getReg();
   164:     BitVector &Set = MO.isDef() ? Defs : Uses;
   165:     expandReg(R, Set);
   166:   }
   167: }
   168: 
   169: void HexagonGenMux::buildMaps(MachineBasicBlock &B, InstrIndexMap &I2X,
   170:       DefUseInfoMap &DUM) {
   171:   unsigned Index = 0;
   172:   unsigned NR = HRI->getNumRegs();
   173:   BitVector Defs(NR), Uses(NR);
   174: 
   175:   for (MachineInstr &MI : B) {
   176:     I2X.insert(std::make_pair(&MI, Index));
   177:     Defs.reset();
   178:     Uses.reset();
   179:     getDefsUses(&MI, Defs, Uses);
   180:     DUM.insert(std::make_pair(Index, DefUseInfo(Defs, Uses)));
   181:     Index++;
   182:   }
   183: }
   184: 
   185: bool HexagonGenMux::isCondTransfer(unsigned Opc) const {
   186:   switch (Opc) {
   187:     case Hexagon::A2_tfrt:
   188:     case Hexagon::A2_tfrf:
   189:     case Hexagon::C2_cmoveit:
   190:     case Hexagon::C2_cmoveif:
   191:       return true;
   192:   }
   193:   return false;
   194: }
   195: 
   196: unsigned HexagonGenMux::getMuxOpcode(const MachineOperand &Src1,
   197:       const MachineOperand &Src2) const {
   198:   bool IsReg1 = Src1.isReg(), IsReg2 = Src2.isReg();
   199:   if (IsReg1)
   200:     return IsReg2 ? Hexagon::C2_mux : Hexagon::C2_muxir;
```
- EN: It declares or implements routines such as getOpcode, get, getReg, expandReg, ... (13 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonGenMux, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getOpcode, get, getReg, expandReg, ... (13 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonGenMux，说明了它与同级后端组件的连接关系。

### Lines 201-250 / 第 201-250 行

```cpp
   201:   if (IsReg2)
   202:     return Hexagon::C2_muxri;
   203: 
   204:   // Neither is a register. The first source is extendable, but the second
   205:   // is not (s8).
   206:   if (Src2.isImm() && isInt<8>(Src2.getImm()))
   207:     return Hexagon::C2_muxii;
   208: 
   209:   return 0;
   210: }
   211: 
   212: bool HexagonGenMux::genMuxInBlock(MachineBasicBlock &B) {
   213:   bool Changed = false;
   214:   InstrIndexMap I2X;
   215:   DefUseInfoMap DUM;
   216:   buildMaps(B, I2X, DUM);
   217: 
   218:   using CondsetMap = DenseMap<unsigned, CondsetInfo>;
   219: 
   220:   CondsetMap CM;
   221:   MuxInfoList ML;
   222: 
   223:   for (MachineInstr &MI : llvm::make_early_inc_range(B)) {
   224:     unsigned Opc = MI.getOpcode();
   225:     if (!isCondTransfer(Opc))
   226:       continue;
   227:     Register DR = MI.getOperand(0).getReg();
   228:     if (isRegPair(DR))
   229:       continue;
   230:     MachineOperand &PredOp = MI.getOperand(1);
   231:     if (PredOp.isUndef())
   232:       continue;
   233: 
   234:     Register PR = PredOp.getReg();
   235:     unsigned Idx = I2X.lookup(&MI);
   236:     CondsetMap::iterator F = CM.find(DR);
   237:     bool IfTrue = HII->isPredicatedTrue(Opc);
   238: 
   239:     // If there is no record of a conditional transfer for this register,
   240:     // or the predicate register differs, create a new record for it.
   241:     if (F != CM.end() && F->second.PredR != PR) {
   242:       CM.erase(F);
   243:       F = CM.end();
   244:     }
   245:     if (F == CM.end()) {
   246:       F = CM.try_emplace(DR).first;
   247:       F->second.PredR = PR;
   248:     }
   249:     CondsetInfo &CI = F->second;
   250:     if (IfTrue)
```
- EN: It declares or implements routines such as HexagonGenMux::genMuxInBlock, buildMaps, getOpcode, getOperand, ... (10 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonGenMux, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonGenMux::genMuxInBlock, buildMaps, getOpcode, getOperand, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonGenMux，说明了它与同级后端组件的连接关系。

### Lines 251-300 / 第 251-300 行

```cpp
   251:       CI.TrueX = Idx;
   252:     else
   253:       CI.FalseX = Idx;
   254:     if (CI.TrueX == std::numeric_limits<unsigned>::max() ||
   255:         CI.FalseX == std::numeric_limits<unsigned>::max())
   256:       continue;
   257: 
   258:     // There is now a complete definition of DR, i.e. we have the predicate
   259:     // register, the definition if-true, and definition if-false.
   260: 
   261:     // First, check if the definitions are far enough from the definition
   262:     // of the predicate register.
   263:     unsigned MinX = std::min(CI.TrueX, CI.FalseX);
   264:     unsigned MaxX = std::max(CI.TrueX, CI.FalseX);
   265:     // Specifically, check if the predicate definition is within a prescribed
   266:     // distance from the farther of the two predicated instructions.
   267:     unsigned SearchX = (MaxX >= MinPredDist) ? MaxX-MinPredDist : 0;
   268:     bool NearDef = false;
   269:     for (unsigned X = SearchX; X < MaxX; ++X) {
   270:       const DefUseInfo &DU = DUM.lookup(X);
   271:       if (!DU.Defs[PR])
   272:         continue;
   273:       NearDef = true;
   274:       break;
   275:     }
   276:     if (NearDef)
   277:       continue;
   278: 
   279:     // The predicate register is not defined in the last few instructions.
   280:     // Check if the conversion to MUX is possible (either "up", i.e. at the
   281:     // place of the earlier partial definition, or "down", where the later
   282:     // definition is located). Examine all defs and uses between these two
   283:     // definitions.
   284:     // SR1, SR2 - source registers from the first and the second definition.
   285:     MachineBasicBlock::iterator It1 = B.begin(), It2 = B.begin();
   286:     std::advance(It1, MinX);
   287:     std::advance(It2, MaxX);
   288:     MachineInstr &Def1 = *It1, &Def2 = *It2;
   289:     MachineOperand *Src1 = &Def1.getOperand(2), *Src2 = &Def2.getOperand(2);
   290:     Register SR1 = Src1->isReg() ? Src1->getReg() : Register();
   291:     Register SR2 = Src2->isReg() ? Src2->getReg() : Register();
   292:     bool Failure = false, CanUp = true, CanDown = true;
   293:     for (unsigned X = MinX+1; X < MaxX; X++) {
   294:       const DefUseInfo &DU = DUM.lookup(X);
   295:       if (DU.Defs[PR] || DU.Defs[DR] || DU.Uses[DR]) {
   296:         Failure = true;
   297:         break;
   298:       }
   299:       if (CanDown && DU.Defs[SR1])
   300:         CanDown = false;
```
- EN: It declares or implements routines such as std::min, std::max, lookup, possible, ... (7 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 std::min, std::max, lookup, possible, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 301-350 / 第 301-350 行

```cpp
   301:       if (CanUp && DU.Defs[SR2])
   302:         CanUp = false;
   303:     }
   304:     if (Failure || (!CanUp && !CanDown))
   305:       continue;
   306: 
   307:     MachineOperand *SrcT = (MinX == CI.TrueX) ? Src1 : Src2;
   308:     MachineOperand *SrcF = (MinX == CI.FalseX) ? Src1 : Src2;
   309:     // Prefer "down", since this will move the MUX farther away from the
   310:     // predicate definition.
   311:     MachineBasicBlock::iterator At = CanDown ? Def2 : Def1;
   312:     ML.push_back(MuxInfo(At, DR, PR, SrcT, SrcF, Def1, Def2));
   313:   }
   314: 
   315:   for (MuxInfo &MX : ML) {
   316:     unsigned MxOpc = getMuxOpcode(*MX.SrcT, *MX.SrcF);
   317:     if (!MxOpc)
   318:       continue;
   319:     // Basic correctness check: since we are deleting instructions, validate the
   320:     // iterators. There is a possibility that one of Def1 or Def2 is translated
   321:     // to "mux" and being considered for other "mux" instructions.
   322:     if (!MX.At->getParent() || !MX.Def1->getParent() || !MX.Def2->getParent())
   323:       continue;
   324: 
   325:     MachineBasicBlock &B = *MX.At->getParent();
   326:     const DebugLoc &DL = B.findDebugLoc(MX.At);
   327:     auto NewMux = BuildMI(B, MX.At, DL, HII->get(MxOpc), MX.DefR)
   328:                       .addReg(MX.PredR)
   329:                       .add(*MX.SrcT)
   330:                       .add(*MX.SrcF);
   331:     NewMux->clearKillInfo();
   332:     B.remove(MX.Def1);
   333:     B.remove(MX.Def2);
   334:     Changed = true;
   335:   }
   336: 
   337:   // Fix up kill flags.
   338: 
   339:   LiveRegUnits LPR(*HRI);
   340:   LPR.addLiveOuts(B);
   341:   for (MachineInstr &I : llvm::reverse(B)) {
   342:     if (I.isDebugInstr())
   343:       continue;
   344:     // This isn't 100% accurate, but it's safe.
   345:     // It won't detect (as a kill) a case like this
   346:     //   r0 = add r0, 1    <-- r0 should be "killed"
   347:     //   ... = r0
   348:     for (MachineOperand &Op : I.operands()) {
   349:       if (!Op.isReg() || !Op.isUse())
   350:         continue;
```
- EN: It declares or implements routines such as push_back, getMuxOpcode, getParent, findDebugLoc, ... (10 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 push_back, getMuxOpcode, getParent, findDebugLoc, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 351-374 / 第 351-374 行

```cpp
   351:       assert(Op.getSubReg() == 0 && "Should have physical registers only");
   352:       bool Live = !LPR.available(Op.getReg());
   353:       Op.setIsKill(!Live);
   354:     }
   355:     LPR.stepBackward(I);
   356:   }
   357: 
   358:   return Changed;
   359: }
   360: 
   361: bool HexagonGenMux::runOnMachineFunction(MachineFunction &MF) {
   362:   if (skipFunction(MF.getFunction()))
   363:     return false;
   364:   HII = MF.getSubtarget<HexagonSubtarget>().getInstrInfo();
   365:   HRI = MF.getSubtarget<HexagonSubtarget>().getRegisterInfo();
   366:   bool Changed = false;
   367:   for (auto &I : MF)
   368:     Changed |= genMuxInBlock(I);
   369:   return Changed;
   370: }
   371: 
   372: FunctionPass *llvm::createHexagonGenMux() {
   373:   return new HexagonGenMux();
   374: }
```
- EN: It declares or implements routines such as assert, available, setIsKill, stepBackward, ... (8 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonGenMux, HexagonSubtarget, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 assert, available, setIsKill, stepBackward, ... (8 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonGenMux, HexagonSubtarget，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- instruction semantics / 指令语义
- MC-layer target description / MC 层目标描述
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `Hexagon.h, HexagonInstrInfo.h, HexagonRegisterInfo.h, HexagonSubtarget.h, llvm/ADT/BitVector.h, llvm/ADT/DenseMap.h, llvm/ADT/SmallVector.h, llvm/ADT/StringRef.h, llvm/CodeGen/LiveRegUnits.h, llvm/CodeGen/MachineBasicBlock.h, ... (24 total)`
- Hexagon symbols / Hexagon 符号: `HexagonGenMux, HexagonExpandCondsets, HexagonInstrInfo, HexagonRegisterInfo, HexagonSubtarget`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
