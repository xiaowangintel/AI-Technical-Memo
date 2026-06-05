# HexagonTfrCleanup.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonTfrCleanup.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Is32 = Hexagon::IntRegsRegClass.contains(Reg);
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```cpp
     1: //===------- HexagonTfrCleanup.cpp - Hexagon Transfer Cleanup Pass -------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: // This pass is to address a situation that appears after register allocation
     9: // every now and then, namely a register copy from a source that was defined
    10: // as an immediate value in the same block (usually just before the copy).
    11: //
    12: // Here is an example of actual code emitted that shows this problem:
    13: //
    14: //  .LBB0_5:
    15: //  {
    16: //    r5 = zxtb(r8)
    17: //    r6 = or(r6, ##12345)
    18: //  }
    19: //  {
    20: //    r3 = xor(r1, r2)
    21: //    r1 = #0               <-- r1 set to #0
    22: //  }
    23: //  {
    24: //    r7 = r1               <-- r7 set to r1
    25: //    r0 = zxtb(r3)
    26: //  }
    27: 
    28: #include "Hexagon.h"
    29: #include "HexagonTargetMachine.h"
    30: 
    31: #include "llvm/CodeGen/LiveIntervals.h"
    32: #include "llvm/CodeGen/MachineFunction.h"
    33: #include "llvm/CodeGen/MachineInstrBuilder.h"
    34: #include "llvm/CodeGen/Passes.h"
    35: #include "llvm/CodeGen/TargetInstrInfo.h"
    36: #include "llvm/CodeGen/TargetRegisterInfo.h"
    37: 
    38: using namespace llvm;
    39: 
    40: #define DEBUG_TYPE "tfr-cleanup"
    41: 
    42: namespace {
    43: class HexagonTfrCleanup : public MachineFunctionPass {
    44: public:
    45:   static char ID;
    46:   HexagonTfrCleanup() : MachineFunctionPass(ID) {}
    47:   StringRef getPassName() const override { return "Hexagon TFR Cleanup"; }
    48:   void getAnalysisUsage(AnalysisUsage &AU) const override {
    49:     AU.setPreservesAll();
    50:     MachineFunctionPass::getAnalysisUsage(AU);
```
- EN: It imports headers such as Hexagon.h, HexagonTargetMachine.h, llvm/CodeGen/LiveIntervals.h, llvm/CodeGen/MachineFunction.h, ... (8 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonTfrCleanup, which carry the state or API of this component.
- CN: 这里引入了 Hexagon.h, HexagonTargetMachine.h, llvm/CodeGen/LiveIntervals.h, llvm/CodeGen/MachineFunction.h, ... (8 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonTfrCleanup 等类型，用来承载该组件的状态或接口。

### Lines 51-100 / 第 51-100 行

```cpp
    51:   }
    52:   bool runOnMachineFunction(MachineFunction &MF) override;
    53: 
    54: private:
    55:   const HexagonInstrInfo *HII = nullptr;
    56:   const TargetRegisterInfo *TRI = nullptr;
    57: 
    58:   typedef DenseMap<unsigned, uint64_t> ImmediateMap;
    59: 
    60:   bool isIntReg(unsigned Reg, bool &Is32);
    61:   void setReg(unsigned R32, uint32_t V32, ImmediateMap &IMap);
    62:   bool getReg(unsigned Reg, uint64_t &Val, ImmediateMap &IMap);
    63:   bool updateImmMap(MachineInstr *MI, ImmediateMap &IMap);
    64:   bool rewriteIfImm(MachineInstr *MI, ImmediateMap &IMap, SlotIndexes *Indexes);
    65:   bool eraseIfRedundant(MachineInstr *MI, SlotIndexes *Indexes);
    66: };
    67: } // namespace
    68: 
    69: char HexagonTfrCleanup::ID = 0;
    70: 
    71: namespace llvm {
    72: char &HexagonTfrCleanupID = HexagonTfrCleanup::ID;
    73: }
    74: 
    75: bool HexagonTfrCleanup::isIntReg(unsigned Reg, bool &Is32) {
    76:   Is32 = Hexagon::IntRegsRegClass.contains(Reg);
    77:   return Is32 || Hexagon::DoubleRegsRegClass.contains(Reg);
    78: }
    79: 
    80: // Assign given value V32 to the specified the register R32 in the map. Only
    81: // 32-bit registers are valid arguments.
    82: void HexagonTfrCleanup::setReg(unsigned R32, uint32_t V32, ImmediateMap &IMap) {
    83:   IMap[R32] = V32;
    84: }
    85: 
    86: // Retrieve a value of the provided register Reg and store it into Val.
    87: // Return "true" if a value was found, "false" otherwise.
    88: bool HexagonTfrCleanup::getReg(unsigned Reg, uint64_t &Val,
    89:                                ImmediateMap &IMap) {
    90:   bool Is32;
    91:   if (!isIntReg(Reg, Is32))
    92:     return false;
    93: 
    94:   if (Is32) {
    95:     ImmediateMap::iterator F = IMap.find(Reg);
    96:     if (F == IMap.end())
    97:       return false;
    98:     Val = F->second;
    99:     return true;
   100:   }
```
- EN: It opens namespaces (char, llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as runOnMachineFunction, isIntReg, setReg, getReg, ... (12 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonInstrInfo, HexagonTfrCleanup, HexagonTfrCleanupID, showing how the code connects to sibling backend components.
- CN: 这里打开了命名空间（char, llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 runOnMachineFunction, isIntReg, setReg, getReg, ... (12 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonInstrInfo, HexagonTfrCleanup, HexagonTfrCleanupID，说明了它与同级后端组件的连接关系。

### Lines 101-150 / 第 101-150 行

```cpp
   101: 
   102:   // For 64-bit registers, compose the value from the values of its
   103:   // subregisters.
   104:   unsigned SubL = TRI->getSubReg(Reg, Hexagon::isub_lo);
   105:   unsigned SubH = TRI->getSubReg(Reg, Hexagon::isub_hi);
   106:   ImmediateMap::iterator FL = IMap.find(SubL), FH = IMap.find(SubH);
   107:   if (FL == IMap.end() || FH == IMap.end())
   108:     return false;
   109:   Val = (FH->second << 32) | FL->second;
   110:   return true;
   111: }
   112: 
   113: // Process an instruction and record the relevant information in the imme-
   114: // diate map.
   115: bool HexagonTfrCleanup::updateImmMap(MachineInstr *MI, ImmediateMap &IMap) {
   116:   using namespace Hexagon;
   117: 
   118:   if (MI->isCall()) {
   119:     IMap.clear();
   120:     return true;
   121:   }
   122: 
   123:   // If this is an instruction that loads a constant into a register,
   124:   // record this information in IMap.
   125:   unsigned Opc = MI->getOpcode();
   126:   if (Opc == A2_tfrsi || Opc == A2_tfrpi) {
   127:     unsigned DefR = MI->getOperand(0).getReg();
   128:     bool Is32;
   129:     if (!isIntReg(DefR, Is32))
   130:       return false;
   131:     if (!MI->getOperand(1).isImm()) {
   132:       if (!Is32) {
   133:         IMap.erase(TRI->getSubReg(DefR, isub_lo));
   134:         IMap.erase(TRI->getSubReg(DefR, isub_hi));
   135:       } else {
   136:         IMap.erase(DefR);
   137:       }
   138:       return false;
   139:     }
   140:     uint64_t Val = MI->getOperand(1).getImm();
   141:     // If it's a 64-bit register, break it up into subregisters.
   142:     if (!Is32) {
   143:       uint32_t VH = (Val >> 32), VL = (Val & 0xFFFFFFFFU);
   144:       setReg(TRI->getSubReg(DefR, isub_lo), VL, IMap);
   145:       setReg(TRI->getSubReg(DefR, isub_hi), VH, IMap);
   146:     } else {
   147:       setReg(DefR, Val, IMap);
   148:     }
   149:     return true;
   150:   }
```
- EN: It opens namespaces (Hexagon) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as getSubReg, find, HexagonTfrCleanup::updateImmMap, clear, ... (8 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonTfrCleanup, showing how the code connects to sibling backend components.
- CN: 这里打开了命名空间（Hexagon），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 getSubReg, find, HexagonTfrCleanup::updateImmMap, clear, ... (8 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonTfrCleanup，说明了它与同级后端组件的连接关系。

### Lines 151-200 / 第 151-200 行

```cpp
   151: 
   152:   // Not a A2_tfr[sp]i. Invalidate all modified registers in IMap.
   153:   for (MachineInstr::mop_iterator Mo = MI->operands_begin(),
   154:                                   E = MI->operands_end();
   155:        Mo != E; ++Mo) {
   156:     if (Mo->isRegMask()) {
   157:       IMap.clear();
   158:       return true;
   159:     }
   160:     if (!Mo->isReg() || !Mo->isDef())
   161:       continue;
   162:     unsigned R = Mo->getReg();
   163:     for (MCRegAliasIterator AR(R, TRI, true); AR.isValid(); ++AR)
   164:       IMap.erase(*AR);
   165:   }
   166:   return true;
   167: }
   168: 
   169: // Rewrite the instruction as A2_tfrsi/A2_tfrpi, it is a copy of a source that
   170: // has a known constant value.
   171: bool HexagonTfrCleanup::rewriteIfImm(MachineInstr *MI, ImmediateMap &IMap,
   172:                                      SlotIndexes *Indexes) {
   173:   using namespace Hexagon;
   174:   unsigned Opc = MI->getOpcode();
   175:   switch (Opc) {
   176:   case A2_tfr:
   177:   case A2_tfrp:
   178:   case COPY:
   179:     break;
   180:   default:
   181:     return false;
   182:   }
   183: 
   184:   unsigned DstR = MI->getOperand(0).getReg();
   185:   unsigned SrcR = MI->getOperand(1).getReg();
   186:   bool Tmp, Is32;
   187:   if (!isIntReg(DstR, Is32) || !isIntReg(SrcR, Tmp))
   188:     return false;
   189:   assert(Tmp == Is32 && "Register size mismatch");
   190:   uint64_t Val;
   191:   bool Found = getReg(SrcR, Val, IMap);
   192:   if (!Found)
   193:     return false;
   194: 
   195:   MachineBasicBlock &B = *MI->getParent();
   196:   DebugLoc DL = MI->getDebugLoc();
   197:   int64_t SVal = Is32 ? int32_t(Val) : Val;
   198:   auto &HST = B.getParent()->getSubtarget<HexagonSubtarget>();
   199:   MachineInstr *NewMI;
   200:   if (Is32)
```
- EN: It opens namespaces (Hexagon) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as clear, getReg, isValid, erase, ... (10 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这里打开了命名空间（Hexagon），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 clear, getReg, isValid, erase, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 201-250 / 第 201-250 行

```cpp
   201:     NewMI = BuildMI(B, MI, DL, HII->get(A2_tfrsi), DstR).addImm(SVal);
   202:   else if (isInt<8>(SVal))
   203:     NewMI = BuildMI(B, MI, DL, HII->get(A2_tfrpi), DstR).addImm(SVal);
   204:   else if (isInt<8>(SVal >> 32) && isInt<8>(int32_t(Val & 0xFFFFFFFFLL)))
   205:     NewMI = BuildMI(B, MI, DL, HII->get(A2_combineii), DstR)
   206:                 .addImm(int32_t(SVal >> 32))
   207:                 .addImm(int32_t(Val & 0xFFFFFFFFLL));
   208:   else if (HST.isTinyCore())
   209:     // Disable generating CONST64 since it requires load resource.
   210:     return false;
   211:   else
   212:     NewMI = BuildMI(B, MI, DL, HII->get(CONST64), DstR).addImm(Val);
   213: 
   214:   // Replace the MI to reuse the same slot index
   215:   if (Indexes)
   216:     Indexes->replaceMachineInstrInMaps(*MI, *NewMI);
   217:   MI->eraseFromParent();
   218:   return true;
   219: }
   220: 
   221: // Remove the instruction if it is a self-assignment.
   222: bool HexagonTfrCleanup::eraseIfRedundant(MachineInstr *MI,
   223:                                          SlotIndexes *Indexes) {
   224:   unsigned Opc = MI->getOpcode();
   225:   unsigned DefR, SrcR;
   226:   bool IsUndef = false;
   227:   switch (Opc) {
   228:   case Hexagon::A2_tfr:
   229:     // Rd = Rd
   230:     DefR = MI->getOperand(0).getReg();
   231:     SrcR = MI->getOperand(1).getReg();
   232:     IsUndef = MI->getOperand(1).isUndef();
   233:     break;
   234:   case Hexagon::A2_tfrt:
   235:   case Hexagon::A2_tfrf:
   236:     // if ([!]Pu) Rd = Rd
   237:     DefR = MI->getOperand(0).getReg();
   238:     SrcR = MI->getOperand(2).getReg();
   239:     IsUndef = MI->getOperand(2).isUndef();
   240:     break;
   241:   default:
   242:     return false;
   243:   }
   244:   if (DefR != SrcR)
   245:     return false;
   246:   if (IsUndef) {
   247:     MachineBasicBlock &B = *MI->getParent();
   248:     DebugLoc DL = MI->getDebugLoc();
   249:     auto DefI = BuildMI(B, MI, DL, HII->get(TargetOpcode::IMPLICIT_DEF), DefR);
   250:     for (auto &Op : MI->operands())
```
- EN: It declares or implements routines such as BuildMI, eraseFromParent, HexagonTfrCleanup::eraseIfRedundant, getOpcode, ... (7 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonTfrCleanup, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 BuildMI, eraseFromParent, HexagonTfrCleanup::eraseIfRedundant, getOpcode, ... (7 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonTfrCleanup，说明了它与同级后端组件的连接关系。

### Lines 251-300 / 第 251-300 行

```cpp
   251:       if (Op.isReg() && Op.isDef() && Op.isImplicit())
   252:         DefI->addOperand(Op);
   253:   }
   254: 
   255:   if (Indexes)
   256:     Indexes->removeMachineInstrFromMaps(*MI);
   257:   MI->eraseFromParent();
   258:   return true;
   259: }
   260: 
   261: bool HexagonTfrCleanup::runOnMachineFunction(MachineFunction &MF) {
   262:   bool Changed = false;
   263:   // Map: 32-bit register -> immediate value.
   264:   // 64-bit registers are stored through their subregisters.
   265:   ImmediateMap IMap;
   266:   auto *SIWrapper = getAnalysisIfAvailable<SlotIndexesWrapperPass>();
   267:   SlotIndexes *Indexes = SIWrapper ? &SIWrapper->getSI() : nullptr;
   268: 
   269:   auto &HST = MF.getSubtarget<HexagonSubtarget>();
   270:   HII = HST.getInstrInfo();
   271:   TRI = HST.getRegisterInfo();
   272: 
   273:   for (MachineBasicBlock &B : MF) {
   274:     MachineBasicBlock::iterator J, F, NextJ;
   275:     IMap.clear();
   276:     bool Inserted = false, Erased = false;
   277:     for (J = B.begin(), F = B.end(); J != F; J = NextJ) {
   278:       NextJ = std::next(J);
   279:       MachineInstr *MI = &*J;
   280:       bool E = eraseIfRedundant(MI, Indexes);
   281:       Erased |= E;
   282:       if (E)
   283:         continue;
   284:       Inserted |= rewriteIfImm(MI, IMap, Indexes);
   285:       MachineBasicBlock::iterator NewJ = std::prev(NextJ);
   286:       updateImmMap(&*NewJ, IMap);
   287:     }
   288:     bool BlockC = Inserted | Erased;
   289:     Changed |= BlockC;
   290:     if (BlockC && Indexes)
   291:       Indexes->repairIndexesInRange(&B, B.begin(), B.end());
   292:   }
   293: 
   294:   return Changed;
   295: }
   296: 
   297: //===----------------------------------------------------------------------===//
   298: //                         Public Constructor Functions
   299: //===----------------------------------------------------------------------===//
   300: INITIALIZE_PASS(HexagonTfrCleanup, "tfr-cleanup", "Hexagon TFR Cleanup", false,
```
- EN: It declares or implements routines such as eraseFromParent, HexagonTfrCleanup::runOnMachineFunction, getAnalysisIfAvailable<SlotIndexesWrapperPass>, getSubtarget<HexagonSubtarget>, ... (12 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonTfrCleanup, HexagonSubtarget, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 eraseFromParent, HexagonTfrCleanup::runOnMachineFunction, getAnalysisIfAvailable<SlotIndexesWrapperPass>, getSubtarget<HexagonSubtarget>, ... (12 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonTfrCleanup, HexagonSubtarget，说明了它与同级后端组件的连接关系。

### Lines 301-305 / 第 301-305 行

```cpp
   301:                 false)
   302: 
   303: FunctionPass *llvm::createHexagonTfrCleanup() {
   304:   return new HexagonTfrCleanup();
   305: }
```
- EN: It declares or implements routines such as llvm::createHexagonTfrCleanup, HexagonTfrCleanup, translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonTfrCleanup, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 llvm::createHexagonTfrCleanup, HexagonTfrCleanup 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonTfrCleanup，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- register modeling / 寄存器建模
- instruction semantics / 指令语义
- target pipeline configuration / 目标流水线配置

## Dependencies / 依赖关系

- Direct includes / 直接包含: `Hexagon.h, HexagonTargetMachine.h, llvm/CodeGen/LiveIntervals.h, llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineInstrBuilder.h, llvm/CodeGen/Passes.h, llvm/CodeGen/TargetInstrInfo.h, llvm/CodeGen/TargetRegisterInfo.h`
- Hexagon symbols / Hexagon 符号: `HexagonTfrCleanup, HexagonTargetMachine, HexagonInstrInfo, HexagonTfrCleanupID, HexagonSubtarget`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
