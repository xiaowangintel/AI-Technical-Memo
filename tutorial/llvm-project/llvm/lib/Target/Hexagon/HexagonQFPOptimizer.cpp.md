# HexagonQFPOptimizer.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonQFPOptimizer.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon QFP-specific optimization logic.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-50 / 第 1-50 行

```cpp
     1: //===----- HexagonQFPOptimizer.cpp - Qualcomm-FP to IEEE-FP conversions
     2: // optimizer ------------------===//
     3: //
     4: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     5: // See https://llvm.org/LICENSE.txt for license information.
     6: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     7: //
     8: //===----------------------------------------------------------------------===//
     9: //
    10: // Basic infrastructure for optimizing intermediate conversion instructions
    11: // generated while performing vector floating point operations.
    12: // Currently run at the starting of the code generation for Hexagon, cleans
    13: // up redundant conversion instructions and replaces the uses of conversion
    14: // with appropriate machine operand. Liveness is preserved after this pass.
    15: //
    16: // @note: The redundant conversion instructions are not eliminated in this pass.
    17: // In this pass, we are only trying to replace the uses of conversion
    18: // instructions with its appropriate QFP instruction. We are leaving the job to
    19: // Dead instruction Elimination pass to remove redundant conversion
    20: // instructions.
    21: //
    22: // Brief overview of working of this QFP optimizer.
    23: // This version of Hexagon QFP optimizer basically iterates over each
    24: // instruction, checks whether if it belongs to hexagon floating point HVX
    25: // arithmetic instruction category(Add, Sub, Mul). And then it finds the unique
    26: // definition for the machine operands corresponding to the instruction.
    27: //
    28: // Example:
    29: // MachineInstruction *MI be the HVX vadd instruction
    30: // MI -> $v0 = V6_vadd_sf $v1, $v2
    31: // MachineOperand *DefMI1 = MRI->getVRegDef(MI->getOperand(1).getReg());
    32: // MachineOperand *DefMI2 = MRI->getVRegDef(MI->getOperand(2).getReg());
    33: //
    34: // In the above example, DefMI1 and DefMI2 gives the unique definitions
    35: // corresponding to the operands($v1 and &v2 respectively) of instruction MI.
    36: //
    37: // If both of the definitions are not conversion instructions(V6_vconv_sf_qf32,
    38: // V6_vconv_hf_qf16), then it will skip optimizing the current instruction and
    39: // iterates over next instruction.
    40: //
    41: // If one the definitions is conversion instruction then our pass will replace
    42: // the arithmetic instruction with its corresponding mix variant.
    43: // In the above example, if $v1 is conversion instruction
    44: // DefMI1 -> $v1 = V6_vconv_sf_qf32 $v3
    45: // After Transformation:
    46: // MI -> $v0 = V6_vadd_qf32_mix $v3, $v2 ($v1 is replaced with $v3)
    47: //
    48: // If both the definitions are conversion instructions then the instruction will
    49: // be replaced with its qf variant
    50: // In the above example, if $v1 and $v2 are conversion instructions
```
- EN: This range is dominated by comments or banner text that documents the surrounding section. It declares or implements routines such as category, getVRegDef, translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonQFPOptimizer, showing how the code connects to sibling backend components.
- CN: 这一段主要由注释或分隔说明组成，用来解释后续代码区域。 这里声明或实现了 category, getVRegDef 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonQFPOptimizer，说明了它与同级后端组件的连接关系。

### Lines 51-100 / 第 51-100 行

```cpp
    51: // DefMI1 -> $v1 = V6_vconv_sf_qf32 $v3
    52: // DefMI2 -> $v2 = V6_vconv_sf_qf32 $v4
    53: // After Transformation:
    54: // MI -> $v0 = V6_vadd_qf32 $v3, $v4 ($v1 is replaced with $v3, $v2 is replaced
    55: // with $v4)
    56: //
    57: // Currently, in this pass, we are not handling the case when the definitions
    58: // are PHI inst.
    59: //
    60: //===----------------------------------------------------------------------===//
    61: 
    62: #define HEXAGON_QFP_OPTIMIZER "QFP optimizer pass"
    63: 
    64: #include "Hexagon.h"
    65: #include "HexagonInstrInfo.h"
    66: #include "HexagonSubtarget.h"
    67: #include "llvm/ADT/SmallVector.h"
    68: #include "llvm/ADT/StringRef.h"
    69: #include "llvm/CodeGen/MachineBasicBlock.h"
    70: #include "llvm/CodeGen/MachineFunction.h"
    71: #include "llvm/CodeGen/MachineFunctionPass.h"
    72: #include "llvm/CodeGen/MachineInstr.h"
    73: #include "llvm/CodeGen/MachineOperand.h"
    74: #include "llvm/CodeGen/Passes.h"
    75: #include "llvm/Pass.h"
    76: #include "llvm/Support/CommandLine.h"
    77: #include "llvm/Support/Debug.h"
    78: #include "llvm/Support/raw_ostream.h"
    79: #include <map>
    80: 
    81: #define DEBUG_TYPE "hexagon-qfp-optimizer"
    82: 
    83: using namespace llvm;
    84: 
    85: cl::opt<bool>
    86:     DisableQFOptimizer("disable-qfp-opt", cl::init(false),
    87:                        cl::desc("Disable optimization of Qfloat operations."));
    88: cl::opt<bool> DisableQFOptForMul(
    89:     "disable-qfp-opt-mul", cl::init(true),
    90:     cl::desc("Disable optimization of Qfloat operations for multiply."));
    91: 
    92: namespace {
    93: const std::map<unsigned short, unsigned short> QFPInstMap{
    94:     {Hexagon::V6_vadd_hf, Hexagon::V6_vadd_qf16_mix},
    95:     {Hexagon::V6_vadd_qf16_mix, Hexagon::V6_vadd_qf16},
    96:     {Hexagon::V6_vadd_sf, Hexagon::V6_vadd_qf32_mix},
    97:     {Hexagon::V6_vadd_qf32_mix, Hexagon::V6_vadd_qf32},
    98:     {Hexagon::V6_vsub_hf, Hexagon::V6_vsub_qf16_mix},
    99:     {Hexagon::V6_vsub_qf16_mix, Hexagon::V6_vsub_qf16},
   100:     {Hexagon::V6_vsub_sf, Hexagon::V6_vsub_qf32_mix},
```
- EN: It imports headers such as Hexagon.h, HexagonInstrInfo.h, HexagonSubtarget.h, llvm/ADT/SmallVector.h, ... (16 total), establishing the LLVM/Hexagon APIs used below. Header guards in this range prevent duplicate inclusion and define the interface boundary. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as DisableQFOptimizer, DisableQFOptForMul, translating Hexagon-specific policy into reusable code paths.
- CN: 这里引入了 Hexagon.h, HexagonInstrInfo.h, HexagonSubtarget.h, llvm/ADT/SmallVector.h, ... (16 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 DisableQFOptimizer, DisableQFOptForMul 等例程，把 Hexagon 特定策略落实为可复用的代码路径。

### Lines 101-150 / 第 101-150 行

```cpp
   101:     {Hexagon::V6_vsub_qf32_mix, Hexagon::V6_vsub_qf32},
   102:     {Hexagon::V6_vmpy_qf16_hf, Hexagon::V6_vmpy_qf16_mix_hf},
   103:     {Hexagon::V6_vmpy_qf16_mix_hf, Hexagon::V6_vmpy_qf16},
   104:     {Hexagon::V6_vmpy_qf32_hf, Hexagon::V6_vmpy_qf32_mix_hf},
   105:     {Hexagon::V6_vmpy_qf32_mix_hf, Hexagon::V6_vmpy_qf32_qf16},
   106:     {Hexagon::V6_vmpy_qf32_sf, Hexagon::V6_vmpy_qf32},
   107:     {Hexagon::V6_vilog2_sf, Hexagon::V6_vilog2_qf32},
   108:     {Hexagon::V6_vilog2_hf, Hexagon::V6_vilog2_qf16},
   109:     {Hexagon::V6_vabs_qf32_sf, Hexagon::V6_vabs_qf32_qf32},
   110:     {Hexagon::V6_vabs_qf16_hf, Hexagon::V6_vabs_qf16_qf16},
   111:     {Hexagon::V6_vneg_qf32_sf, Hexagon::V6_vneg_qf32_qf32},
   112:     {Hexagon::V6_vneg_qf16_hf, Hexagon::V6_vneg_qf16_qf16}};
   113: } // namespace
   114: 
   115: namespace {
   116: struct HexagonQFPOptimizer : public MachineFunctionPass {
   117: public:
   118:   static char ID;
   119: 
   120:   HexagonQFPOptimizer() : MachineFunctionPass(ID) {}
   121: 
   122:   bool runOnMachineFunction(MachineFunction &MF) override;
   123: 
   124:   bool optimizeQfp(MachineInstr *MI, MachineBasicBlock *MBB);
   125: 
   126:   bool optimizeQfpTwoOp(MachineInstr *MI, MachineBasicBlock *MBB);
   127: 
   128:   bool optimizeQfpOneOp(MachineInstr *MI, MachineBasicBlock *MBB);
   129: 
   130:   StringRef getPassName() const override { return HEXAGON_QFP_OPTIMIZER; }
   131: 
   132:   void getAnalysisUsage(AnalysisUsage &AU) const override {
   133:     AU.setPreservesCFG();
   134:     MachineFunctionPass::getAnalysisUsage(AU);
   135:   }
   136: 
   137: private:
   138:   const HexagonSubtarget *HST = nullptr;
   139:   const HexagonInstrInfo *HII = nullptr;
   140:   const MachineRegisterInfo *MRI = nullptr;
   141: };
   142: 
   143: char HexagonQFPOptimizer::ID = 0;
   144: } // namespace
   145: 
   146: INITIALIZE_PASS(HexagonQFPOptimizer, "hexagon-qfp-optimizer",
   147:                 HEXAGON_QFP_OPTIMIZER, false, false)
   148: 
   149: FunctionPass *llvm::createHexagonQFPOptimizer() {
   150:   return new HexagonQFPOptimizer();
```
- EN: It opens namespaces (namespace, INITIALIZE_PASS) to keep the implementation scoped to LLVM/Hexagon components. It declares types such as HexagonQFPOptimizer, which carry the state or API of this component. It declares or implements routines such as HexagonQFPOptimizer, runOnMachineFunction, optimizeQfp, optimizeQfpTwoOp, ... (10 total), translating Hexagon-specific policy into reusable code paths. Notable Hexagon symbols referenced here include HexagonQFPOptimizer, HexagonSubtarget, HexagonInstrInfo, showing how the code connects to sibling backend components.
- CN: 这里打开了命名空间（namespace, INITIALIZE_PASS），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明了 HexagonQFPOptimizer 等类型，用来承载该组件的状态或接口。 这里声明或实现了 HexagonQFPOptimizer, runOnMachineFunction, optimizeQfp, optimizeQfpTwoOp, ... (10 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里引用的重要 Hexagon 符号包括 HexagonQFPOptimizer, HexagonSubtarget, HexagonInstrInfo，说明了它与同级后端组件的连接关系。

### Lines 151-200 / 第 151-200 行

```cpp
   151: }
   152: 
   153: bool HexagonQFPOptimizer::optimizeQfp(MachineInstr *MI,
   154:                                       MachineBasicBlock *MBB) {
   155: 
   156:   if (MI->getNumOperands() == 2)
   157:     return optimizeQfpOneOp(MI, MBB);
   158:   else if (MI->getNumOperands() == 3)
   159:     return optimizeQfpTwoOp(MI, MBB);
   160:   else
   161:     return false;
   162: }
   163: 
   164: bool HexagonQFPOptimizer::optimizeQfpOneOp(MachineInstr *MI,
   165:                                            MachineBasicBlock *MBB) {
   166: 
   167:   RegState Op0F = {};
   168:   auto It = QFPInstMap.find(MI->getOpcode());
   169:   if (It == QFPInstMap.end())
   170:     return false;
   171: 
   172:   unsigned short InstTy = It->second;
   173:   // Get the reachind defs of MI
   174:   MachineInstr *DefMI = MRI->getVRegDef(MI->getOperand(1).getReg());
   175:   MachineOperand &Res = MI->getOperand(0);
   176:   if (!Res.isReg())
   177:     return false;
   178: 
   179:   LLVM_DEBUG(dbgs() << "\n[Reaching Defs of operands]: "; DefMI->dump());
   180:   MachineInstr *ReachDefDef = nullptr;
   181: 
   182:   // Get the reaching def of the reaching def to check for W reg def
   183:   if (DefMI->getNumOperands() > 1 && DefMI->getOperand(1).isReg() &&
   184:       DefMI->getOperand(1).getReg().isVirtual())
   185:     ReachDefDef = MRI->getVRegDef(DefMI->getOperand(1).getReg());
   186:   unsigned ReachDefOp = DefMI->getOpcode();
   187:   MachineInstrBuilder MIB;
   188: 
   189:   // Check if the reaching def is a conversion
   190:   if (ReachDefOp == Hexagon::V6_vconv_sf_qf32 ||
   191:       ReachDefOp == Hexagon::V6_vconv_hf_qf16) {
   192: 
   193:     // Return if the reaching def of reaching def is W type
   194:     if (ReachDefDef && MRI->getRegClass(ReachDefDef->getOperand(0).getReg()) ==
   195:                            &Hexagon::HvxWRRegClass)
   196:       return false;
   197: 
   198:     // Analyze the use operands of the conversion to get their KILL status
   199:     MachineOperand &SrcOp = DefMI->getOperand(1);
   200:     Op0F = getKillRegState(SrcOp.isKill());
```
- EN: It declares or implements routines such as HexagonQFPOptimizer::optimizeQfp, HexagonQFPOptimizer::optimizeQfpOneOp, find, getVRegDef, ... (8 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonQFPOptimizer, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonQFPOptimizer::optimizeQfp, HexagonQFPOptimizer::optimizeQfpOneOp, find, getVRegDef, ... (8 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonQFPOptimizer，说明了它与同级后端组件的连接关系。

### Lines 201-250 / 第 201-250 行

```cpp
   201:     SrcOp.setIsKill(false);
   202:     MIB = BuildMI(*MBB, MI, MI->getDebugLoc(), HII->get(InstTy), Res.getReg())
   203:               .addReg(SrcOp.getReg(), Op0F, SrcOp.getSubReg());
   204:     LLVM_DEBUG(dbgs() << "\n[Inserting]: "; MIB.getInstr()->dump());
   205:     return true;
   206:   }
   207:   return false;
   208: }
   209: 
   210: bool HexagonQFPOptimizer::optimizeQfpTwoOp(MachineInstr *MI,
   211:                                            MachineBasicBlock *MBB) {
   212: 
   213:   RegState Op0F = {};
   214:   RegState Op1F = {};
   215:   auto It = QFPInstMap.find(MI->getOpcode());
   216:   if (It == QFPInstMap.end())
   217:     return false;
   218:   unsigned short InstTy = It->second;
   219:   // Get the reaching defs of MI, DefMI1 and DefMI2
   220:   MachineInstr *DefMI1 = nullptr;
   221:   MachineInstr *DefMI2 = nullptr;
   222: 
   223:   if (MI->getOperand(1).isReg())
   224:     DefMI1 = MRI->getVRegDef(MI->getOperand(1).getReg());
   225:   if (MI->getOperand(2).isReg())
   226:     DefMI2 = MRI->getVRegDef(MI->getOperand(2).getReg());
   227:   if (!DefMI1 || !DefMI2)
   228:     return false;
   229: 
   230:   MachineOperand &Res = MI->getOperand(0);
   231:   if (!Res.isReg())
   232:     return false;
   233: 
   234:   MachineInstr *Inst1 = nullptr;
   235:   MachineInstr *Inst2 = nullptr;
   236:   LLVM_DEBUG(dbgs() << "\n[Reaching Defs of operands]: "; DefMI1->dump();
   237:              DefMI2->dump());
   238: 
   239:   // Get the reaching defs of DefMI
   240:   if (DefMI1->getNumOperands() > 1 && DefMI1->getOperand(1).isReg() &&
   241:       DefMI1->getOperand(1).getReg().isVirtual())
   242:     Inst1 = MRI->getVRegDef(DefMI1->getOperand(1).getReg());
   243: 
   244:   if (DefMI2->getNumOperands() > 1 && DefMI2->getOperand(1).isReg() &&
   245:       DefMI2->getOperand(1).getReg().isVirtual())
   246:     Inst2 = MRI->getVRegDef(DefMI2->getOperand(1).getReg());
   247: 
   248:   unsigned Def1OP = DefMI1->getOpcode();
   249:   unsigned Def2OP = DefMI2->getOpcode();
   250: 
```
- EN: It declares or implements routines such as setIsKill, BuildMI, getInstr, HexagonQFPOptimizer::optimizeQfpTwoOp, ... (8 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonQFPOptimizer, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 setIsKill, BuildMI, getInstr, HexagonQFPOptimizer::optimizeQfpTwoOp, ... (8 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonQFPOptimizer，说明了它与同级后端组件的连接关系。

### Lines 251-300 / 第 251-300 行

```cpp
   251:   MachineInstrBuilder MIB;
   252: 
   253:   // Check if the both the reaching defs of MI are qf to sf/hf conversions
   254:   if ((Def1OP == Hexagon::V6_vconv_sf_qf32 &&
   255:        Def2OP == Hexagon::V6_vconv_sf_qf32) ||
   256:       (Def1OP == Hexagon::V6_vconv_hf_qf16 &&
   257:        Def2OP == Hexagon::V6_vconv_hf_qf16)) {
   258: 
   259:     // If the reaching defs of DefMI are W register type, we return
   260:     if ((Inst1 && Inst1->getNumOperands() > 0 && Inst1->getOperand(0).isReg() &&
   261:          MRI->getRegClass(Inst1->getOperand(0).getReg()) ==
   262:              &Hexagon::HvxWRRegClass) ||
   263:         (Inst2 && Inst2->getNumOperands() > 0 && Inst2->getOperand(0).isReg() &&
   264:          MRI->getRegClass(Inst2->getOperand(0).getReg()) ==
   265:              &Hexagon::HvxWRRegClass))
   266:       return false;
   267: 
   268:     // Analyze the use operands of the conversion to get their KILL status
   269:     MachineOperand &Src1 = DefMI1->getOperand(1);
   270:     MachineOperand &Src2 = DefMI2->getOperand(1);
   271: 
   272:     Op0F = getKillRegState(Src1.isKill());
   273:     Src1.setIsKill(false);
   274: 
   275:     Op1F = getKillRegState(Src2.isKill());
   276:     Src2.setIsKill(false);
   277: 
   278:     if (MI->getOpcode() != Hexagon::V6_vmpy_qf32_sf) {
   279:       auto OuterIt = QFPInstMap.find(MI->getOpcode());
   280:       if (OuterIt == QFPInstMap.end())
   281:         return false;
   282:       auto InnerIt = QFPInstMap.find(OuterIt->second);
   283:       if (InnerIt == QFPInstMap.end())
   284:         return false;
   285:       InstTy = InnerIt->second;
   286:     }
   287: 
   288:     MIB = BuildMI(*MBB, MI, MI->getDebugLoc(), HII->get(InstTy), Res.getReg())
   289:               .addReg(Src1.getReg(), Op0F, Src1.getSubReg())
   290:               .addReg(Src2.getReg(), Op1F, Src2.getSubReg());
   291:     LLVM_DEBUG(dbgs() << "\n[Inserting]: "; MIB.getInstr()->dump());
   292:     return true;
   293: 
   294:     // Check if left operand's reaching def is a conversion to sf/hf
   295:   } else if (((Def1OP == Hexagon::V6_vconv_sf_qf32 &&
   296:                Def2OP != Hexagon::V6_vconv_sf_qf32) ||
   297:               (Def1OP == Hexagon::V6_vconv_hf_qf16 &&
   298:                Def2OP != Hexagon::V6_vconv_hf_qf16)) &&
   299:              !DefMI2->isPHI() &&
   300:              (MI->getOpcode() != Hexagon::V6_vmpy_qf32_sf)) {
```
- EN: It declares or implements routines such as getOperand, getKillRegState, setIsKill, find, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 getOperand, getKillRegState, setIsKill, find, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 301-350 / 第 301-350 行

```cpp
   301: 
   302:     if (Inst1 && MRI->getRegClass(Inst1->getOperand(0).getReg()) ==
   303:                      &Hexagon::HvxWRRegClass)
   304:       return false;
   305: 
   306:     MachineOperand &Src1 = DefMI1->getOperand(1);
   307:     MachineOperand &Src2 = MI->getOperand(2);
   308: 
   309:     Op0F = getKillRegState(Src1.isKill());
   310:     Src1.setIsKill(false);
   311:     Op1F = getKillRegState(Src2.isKill());
   312:     MIB = BuildMI(*MBB, MI, MI->getDebugLoc(), HII->get(InstTy), Res.getReg())
   313:               .addReg(Src1.getReg(), Op0F, Src1.getSubReg())
   314:               .addReg(Src2.getReg(), Op1F, Src2.getSubReg());
   315:     LLVM_DEBUG(dbgs() << "\n[Inserting]: "; MIB.getInstr()->dump());
   316:     return true;
   317: 
   318:     // Check if right operand's reaching def is a conversion to sf/hf
   319:   } else if (((Def1OP != Hexagon::V6_vconv_sf_qf32 &&
   320:                Def2OP == Hexagon::V6_vconv_sf_qf32) ||
   321:               (Def1OP != Hexagon::V6_vconv_hf_qf16 &&
   322:                Def2OP == Hexagon::V6_vconv_hf_qf16)) &&
   323:              !DefMI1->isPHI() &&
   324:              (MI->getOpcode() != Hexagon::V6_vmpy_qf32_sf)) {
   325:     // The second operand of original instruction is converted.
   326:     if (Inst2 && MRI->getRegClass(Inst2->getOperand(0).getReg()) ==
   327:                      &Hexagon::HvxWRRegClass)
   328:       return false;
   329: 
   330:     MachineOperand &Src1 = MI->getOperand(1);
   331:     MachineOperand &Src2 = DefMI2->getOperand(1);
   332: 
   333:     Op1F = getKillRegState(Src2.isKill());
   334:     Src2.setIsKill(false);
   335:     Op0F = getKillRegState(Src1.isKill());
   336:     if (InstTy == Hexagon::V6_vsub_qf16_mix ||
   337:         InstTy == Hexagon::V6_vsub_qf32_mix) {
   338:       if (!HST->useHVXV81Ops())
   339:         // vsub_(hf|sf)_mix insts are only avlbl on hvx81+
   340:         return false;
   341:       // vsub is not commutative w.r.t. operands -> treat it as a special case
   342:       // to choose the correct mix instruction.
   343:       if (Def2OP == Hexagon::V6_vconv_sf_qf32)
   344:         InstTy = Hexagon::V6_vsub_sf_mix;
   345:       else if (Def2OP == Hexagon::V6_vconv_hf_qf16)
   346:         InstTy = Hexagon::V6_vsub_hf_mix;
   347:       MIB = BuildMI(*MBB, MI, MI->getDebugLoc(), HII->get(InstTy), Res.getReg())
   348:                 .addReg(Src1.getReg(), Op0F, Src1.getSubReg())
   349:                 .addReg(Src2.getReg(), Op1F, Src2.getSubReg());
   350:     } else {
```
- EN: It declares or implements routines such as getOperand, getKillRegState, setIsKill, BuildMI, ... (5 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal.
- CN: 这里声明或实现了 getOperand, getKillRegState, setIsKill, BuildMI, ... (5 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。

### Lines 351-400 / 第 351-400 行

```cpp
   351:       MIB = BuildMI(*MBB, MI, MI->getDebugLoc(), HII->get(InstTy), Res.getReg())
   352:                 .addReg(Src2.getReg(), Op1F,
   353:                         Src2.getSubReg()) // Notice the operands are flipped.
   354:                 .addReg(Src1.getReg(), Op0F, Src1.getSubReg());
   355:     }
   356:     LLVM_DEBUG(dbgs() << "\n[Inserting]: "; MIB.getInstr()->dump());
   357:     return true;
   358:   }
   359: 
   360:   return false;
   361: }
   362: 
   363: bool HexagonQFPOptimizer::runOnMachineFunction(MachineFunction &MF) {
   364: 
   365:   bool Changed = false;
   366: 
   367:   if (DisableQFOptimizer)
   368:     return Changed;
   369: 
   370:   HST = &MF.getSubtarget<HexagonSubtarget>();
   371:   if (!HST->useHVXV68Ops() || !HST->usePackets() ||
   372:       skipFunction(MF.getFunction()))
   373:     return false;
   374:   HII = HST->getInstrInfo();
   375:   MRI = &MF.getRegInfo();
   376: 
   377:   MachineFunction::iterator MBBI = MF.begin();
   378:   LLVM_DEBUG(dbgs() << "\n=== Running QFPOptimzer Pass for : " << MF.getName()
   379:                     << " Optimize intermediate conversions ===\n");
   380:   while (MBBI != MF.end()) {
   381:     MachineBasicBlock *MBB = &*MBBI;
   382:     MachineBasicBlock::iterator MII = MBBI->instr_begin();
   383:     while (MII != MBBI->instr_end()) {
   384:       MachineInstr *MI = &*MII;
   385:       ++MII; // As MI might be removed.
   386:       if (QFPInstMap.count(MI->getOpcode())) {
   387:         auto OpC = MI->getOpcode();
   388:         if (DisableQFOptForMul && HII->isQFPMul(MI))
   389:           continue;
   390:         if (OpC != Hexagon::V6_vconv_sf_qf32 &&
   391:             OpC != Hexagon::V6_vconv_hf_qf16) {
   392:           LLVM_DEBUG(dbgs() << "\n###Analyzing for removal: "; MI->dump());
   393:           if (optimizeQfp(MI, MBB)) {
   394:             MI->eraseFromParent();
   395:             LLVM_DEBUG(dbgs() << "\t....Removing....");
   396:             Changed = true;
   397:           }
   398:         }
   399:       }
   400:     }
```
- EN: It declares or implements routines such as BuildMI, getInstr, HexagonQFPOptimizer::runOnMachineFunction, getSubtarget<HexagonSubtarget>, ... (12 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonQFPOptimizer, HexagonSubtarget, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 BuildMI, getInstr, HexagonQFPOptimizer::runOnMachineFunction, getSubtarget<HexagonSubtarget>, ... (12 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonQFPOptimizer, HexagonSubtarget，说明了它与同级后端组件的连接关系。

### Lines 401-404 / 第 401-404 行

```cpp
   401:     ++MBBI;
   402:   }
   403:   return Changed;
   404: }
```
- EN: This range contains executable implementation details for a Hexagon backend subsystem.
- CN: 这一段包含了某个 Hexagon 后端子系统的可执行实现细节。

## Key Concepts / 关键概念

- HVX vector ISA modeling / HVX 向量 ISA 建模
- register modeling / 寄存器建模
- instruction semantics / 指令语义
- subtarget feature gating / 子目标特性控制

## Dependencies / 依赖关系

- Direct includes / 直接包含: `Hexagon.h, HexagonInstrInfo.h, HexagonSubtarget.h, llvm/ADT/SmallVector.h, llvm/ADT/StringRef.h, llvm/CodeGen/MachineBasicBlock.h, llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstr.h, llvm/CodeGen/MachineOperand.h, ... (16 total)`
- Hexagon symbols / Hexagon 符号: `HexagonQFPOptimizer, HexagonInstrInfo, HexagonSubtarget`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
