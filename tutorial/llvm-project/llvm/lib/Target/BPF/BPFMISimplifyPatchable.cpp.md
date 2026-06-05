# BPFMISimplifyPatchable.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/BPF/BPFMISimplifyPatchable.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Provides target-specific support code, declarations, or helper routines used by the backend.
- 目的（中文）: 提供目标后端使用的专用支持代码、声明或辅助例程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===----- BPFMISimplifyPatchable.cpp - MI Simplify Patchable Insts -------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This pass targets a subset of instructions like below
  10: //    ld_imm64 r1, @global
  11: //    ldd r2, r1, 0
  12: //    add r3, struct_base_reg, r2
  13: //
  14: // Here @global should represent an AMA (abstruct member access).
  15: // Such an access is subject to bpf load time patching. After this pass, the
  16: // code becomes
  17: //    ld_imm64 r1, @global
  18: //    add r3, struct_base_reg, r1
  19: //
  20: // Eventually, at BTF output stage, a relocation record will be generated
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。

### Lines 21-40

```cpp
  21: // for ld_imm64 which should be replaced later by bpf loader:
  22: //    r1 = <calculated field_info>
  23: //    add r3, struct_base_reg, r1
  24: //
  25: // This pass also removes the intermediate load generated in IR pass for
  26: // __builtin_btf_type_id() intrinsic.
  27: //
  28: //===----------------------------------------------------------------------===//
  29: 
  30: #include "BPF.h"
  31: #include "BPFCORE.h"
  32: #include "BPFInstrInfo.h"
  33: #include "BPFTargetMachine.h"
  34: #include "llvm/CodeGen/MachineFunctionPass.h"
  35: #include "llvm/CodeGen/MachineInstrBuilder.h"
  36: #include "llvm/CodeGen/MachineRegisterInfo.h"
  37: #include "llvm/IR/GlobalVariable.h"
  38: #include "llvm/Support/Debug.h"
  39: #include <set>
  40: 
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。

### Lines 41-60

```cpp
  41: using namespace llvm;
  42: 
  43: #define DEBUG_TYPE "bpf-mi-simplify-patchable"
  44: 
  45: static cl::opt<bool>
  46:     DisableCOREOptimization("disable-bpf-core-optimization", cl::Hidden,
  47:                             cl::desc("Disable CORE relocation optimization"));
  48: 
  49: namespace {
  50: 
  51: struct BPFMISimplifyPatchable : public MachineFunctionPass {
  52: 
  53:   static char ID;
  54:   const BPFInstrInfo *TII;
  55:   MachineFunction *MF;
  56: 
  57:   BPFMISimplifyPatchable() : MachineFunctionPass(ID) {}
  58: 
  59: private:
  60:   std::set<MachineInstr *> SkipInsts;
```

- EN: The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. This chunk introduces interfaces or data structures such as BPFMISimplifyPatchable, which organize the target-specific behavior exposed by the file.
- 中文: 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 这一段引入了 BPFMISimplifyPatchable 等接口或数据结构，用于组织该文件暴露的目标专用行为。

### Lines 61-80

```cpp
  61: 
  62:   // Initialize class variables.
  63:   void initialize(MachineFunction &MFParm);
  64: 
  65:   bool isLoadInst(unsigned Opcode);
  66:   bool removeLD();
  67:   void processCandidate(MachineRegisterInfo *MRI, MachineBasicBlock &MBB,
  68:                         MachineInstr &MI, Register &SrcReg, Register &DstReg,
  69:                         const GlobalValue *GVal, bool IsAma);
  70:   void processDstReg(MachineRegisterInfo *MRI, Register &DstReg,
  71:                      Register &SrcReg, const GlobalValue *GVal,
  72:                      bool doSrcRegProp, bool IsAma);
  73:   void processInst(MachineRegisterInfo *MRI, MachineInstr *Inst,
  74:                    MachineOperand *RelocOp, const GlobalValue *GVal);
  75:   void checkADDrr(MachineRegisterInfo *MRI, MachineOperand *RelocOp,
  76:                   const GlobalValue *GVal);
  77:   void checkShift(MachineRegisterInfo *MRI, MachineBasicBlock &MBB,
  78:                   MachineOperand *RelocOp, const GlobalValue *GVal,
  79:                   unsigned Opcode);
  80: 
```

- EN: This chunk introduces interfaces or data structures such as variables, which organize the target-specific behavior exposed by the file.
- 中文: 这一段引入了 variables 等接口或数据结构，用于组织该文件暴露的目标专用行为。

### Lines 81-100

```cpp
  81: public:
  82:   // Main entry point for this pass.
  83:   bool runOnMachineFunction(MachineFunction &MF) override {
  84:     if (skipFunction(MF.getFunction()))
  85:       return false;
  86: 
  87:     initialize(MF);
  88:     return removeLD();
  89:   }
  90: };
  91: 
  92: // Initialize class variables.
  93: void BPFMISimplifyPatchable::initialize(MachineFunction &MFParm) {
  94:   MF = &MFParm;
  95:   TII = MF->getSubtarget<BPFSubtarget>().getInstrInfo();
  96:   LLVM_DEBUG(dbgs() << "*** BPF simplify patchable insts pass ***\n\n");
  97: }
  98: 
  99: static bool isStoreImm(unsigned Opcode) {
 100:   return Opcode == BPF::STB_imm || Opcode == BPF::STH_imm ||
```

- EN: This chunk introduces interfaces or data structures such as variables, which organize the target-specific behavior exposed by the file. Function bodies or method definitions such as initialize, isStoreImm contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这一段引入了 variables 等接口或数据结构，用于组织该文件暴露的目标专用行为。 initialize, isStoreImm 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 101-120

```cpp
 101:          Opcode == BPF::STW_imm || Opcode == BPF::STD_imm;
 102: }
 103: 
 104: static bool isStore32(unsigned Opcode) {
 105:   return Opcode == BPF::STB32 || Opcode == BPF::STH32 || Opcode == BPF::STW32 ||
 106:          Opcode == BPF::STBREL32 || Opcode == BPF::STHREL32 ||
 107:          Opcode == BPF::STWREL32;
 108: }
 109: 
 110: static bool isStore64(unsigned Opcode) {
 111:   return Opcode == BPF::STB || Opcode == BPF::STH || Opcode == BPF::STW ||
 112:          Opcode == BPF::STD || Opcode == BPF::STDREL;
 113: }
 114: 
 115: static bool isLoad32(unsigned Opcode) {
 116:   return Opcode == BPF::LDB32 || Opcode == BPF::LDH32 || Opcode == BPF::LDW32 ||
 117:          Opcode == BPF::LDBACQ32 || Opcode == BPF::LDHACQ32 ||
 118:          Opcode == BPF::LDWACQ32;
 119: }
 120: 
```

- EN: Function bodies or method definitions such as isStore32, isStore64, isLoad32 contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: isStore32, isStore64, isLoad32 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 121-140

```cpp
 121: static bool isLoad64(unsigned Opcode) {
 122:   return Opcode == BPF::LDB || Opcode == BPF::LDH || Opcode == BPF::LDW ||
 123:          Opcode == BPF::LDD || Opcode == BPF::LDDACQ;
 124: }
 125: 
 126: static bool isLoadSext(unsigned Opcode) {
 127:   return Opcode == BPF::LDBSX || Opcode == BPF::LDHSX || Opcode == BPF::LDWSX;
 128: }
 129: 
 130: bool BPFMISimplifyPatchable::isLoadInst(unsigned Opcode) {
 131:   return isLoad32(Opcode) || isLoad64(Opcode) || isLoadSext(Opcode);
 132: }
 133: 
 134: void BPFMISimplifyPatchable::checkADDrr(MachineRegisterInfo *MRI,
 135:     MachineOperand *RelocOp, const GlobalValue *GVal) {
 136:   const MachineInstr *Inst = RelocOp->getParent();
 137:   const MachineOperand *Op1 = &Inst->getOperand(1);
 138:   const MachineOperand *Op2 = &Inst->getOperand(2);
 139:   const MachineOperand *BaseOp = (RelocOp == Op1) ? Op2 : Op1;
 140: 
```

- EN: Function bodies or method definitions such as isLoad64, isLoadSext, isLoadInst contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: isLoad64, isLoadSext, isLoadInst 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 141-160

```cpp
 141:   // Go through all uses of %1 as in %1 = ADD_rr %2, %3
 142:   const MachineOperand Op0 = Inst->getOperand(0);
 143:   for (MachineOperand &MO :
 144:        llvm::make_early_inc_range(MRI->use_operands(Op0.getReg()))) {
 145:     // The candidate needs to have a unique definition.
 146:     if (!MRI->getUniqueVRegDef(MO.getReg()))
 147:       continue;
 148: 
 149:     MachineInstr *DefInst = MO.getParent();
 150:     unsigned Opcode = DefInst->getOpcode();
 151:     unsigned COREOp;
 152:     if (isLoad64(Opcode) || isLoadSext(Opcode))
 153:       COREOp = BPF::CORE_LD64;
 154:     else if (isLoad32(Opcode))
 155:       COREOp = BPF::CORE_LD32;
 156:     else if (isStore64(Opcode) || isStore32(Opcode) || isStoreImm(Opcode))
 157:       COREOp = BPF::CORE_ST;
 158:     else
 159:       continue;
 160: 
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 161-180

```cpp
 161:     // It must be a form of %2 = *(type *)(%1 + 0) or *(type *)(%1 + 0) = %2.
 162:     const MachineOperand &ImmOp = DefInst->getOperand(2);
 163:     if (!ImmOp.isImm() || ImmOp.getImm() != 0)
 164:       continue;
 165: 
 166:     // Reject the form:
 167:     //   %1 = ADD_rr %2, %3
 168:     //   *(type *)(%2 + 0) = %1
 169:     if (isStore64(Opcode) || isStore32(Opcode)) {
 170:       const MachineOperand &Opnd = DefInst->getOperand(0);
 171:       if (Opnd.isReg() && Opnd.getReg() == MO.getReg())
 172:         continue;
 173:     }
 174: 
 175:     BuildMI(*DefInst->getParent(), *DefInst, DefInst->getDebugLoc(), TII->get(COREOp))
 176:         .add(DefInst->getOperand(0)).addImm(Opcode).add(*BaseOp)
 177:         .addGlobalAddress(GVal);
 178:     DefInst->eraseFromParent();
 179:   }
 180: }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 181-200

```cpp
 181: 
 182: void BPFMISimplifyPatchable::checkShift(MachineRegisterInfo *MRI,
 183:     MachineBasicBlock &MBB, MachineOperand *RelocOp, const GlobalValue *GVal,
 184:     unsigned Opcode) {
 185:   // Relocation operand should be the operand #2.
 186:   MachineInstr *Inst = RelocOp->getParent();
 187:   if (RelocOp != &Inst->getOperand(2))
 188:     return;
 189: 
 190:   BuildMI(MBB, *Inst, Inst->getDebugLoc(), TII->get(BPF::CORE_SHIFT))
 191:       .add(Inst->getOperand(0)).addImm(Opcode)
 192:       .add(Inst->getOperand(1)).addGlobalAddress(GVal);
 193:   Inst->eraseFromParent();
 194: }
 195: 
 196: void BPFMISimplifyPatchable::processCandidate(MachineRegisterInfo *MRI,
 197:     MachineBasicBlock &MBB, MachineInstr &MI, Register &SrcReg,
 198:     Register &DstReg, const GlobalValue *GVal, bool IsAma) {
 199:   if (MRI->getRegClass(DstReg) == &BPF::GPR32RegClass) {
 200:     if (IsAma) {
```

- EN: Function bodies or method definitions such as checkShift, processCandidate contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: checkShift, processCandidate 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 201-220

```cpp
 201:       // We can optimize such a pattern:
 202:       //  %1:gpr = LD_imm64 @"llvm.s:0:4$0:2"
 203:       //  %2:gpr32 = LDW32 %1:gpr, 0
 204:       //  %3:gpr = SUBREG_TO_REG %2:gpr32, %subreg.sub_32
 205:       //  %4:gpr = ADD_rr %0:gpr, %3:gpr
 206:       //  or similar patterns below for non-alu32 case.
 207:       auto Begin = MRI->use_begin(DstReg), End = MRI->use_end();
 208:       decltype(End) NextI;
 209:       for (auto I = Begin; I != End; I = NextI) {
 210:         NextI = std::next(I);
 211:         if (!MRI->getUniqueVRegDef(I->getReg()))
 212:           continue;
 213: 
 214:         unsigned Opcode = I->getParent()->getOpcode();
 215:         if (Opcode == BPF::SUBREG_TO_REG) {
 216:           Register TmpReg = I->getParent()->getOperand(0).getReg();
 217:           processDstReg(MRI, TmpReg, DstReg, GVal, false, IsAma);
 218:         }
 219:       }
 220:     }
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 221-240

```cpp
 221: 
 222:     BuildMI(MBB, MI, MI.getDebugLoc(), TII->get(BPF::COPY), DstReg)
 223:         .addReg(SrcReg, {}, BPF::sub_32);
 224:     return;
 225:   }
 226: 
 227:   // All uses of DstReg replaced by SrcReg
 228:   processDstReg(MRI, DstReg, SrcReg, GVal, true, IsAma);
 229: }
 230: 
 231: void BPFMISimplifyPatchable::processDstReg(MachineRegisterInfo *MRI,
 232:     Register &DstReg, Register &SrcReg, const GlobalValue *GVal,
 233:     bool doSrcRegProp, bool IsAma) {
 234:   auto Begin = MRI->use_begin(DstReg), End = MRI->use_end();
 235:   decltype(End) NextI;
 236:   for (auto I = Begin; I != End; I = NextI) {
 237:     NextI = std::next(I);
 238:     if (doSrcRegProp) {
 239:       // In situations like below it is not known if usage is a kill
 240:       // after setReg():
```

- EN: Function bodies or method definitions such as processDstReg contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: processDstReg 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 241-260

```cpp
 241:       //
 242:       // .-> %2:gpr = LD_imm64 @"llvm.t:0:0$0:0"
 243:       // |
 244:       // |`----------------.
 245:       // |   %3:gpr = LDD %2:gpr, 0
 246:       // |   %4:gpr = ADD_rr %0:gpr(tied-def 0), killed %3:gpr <--- (1)
 247:       // |   %5:gpr = LDD killed %4:gpr, 0       ^^^^^^^^^^^^^
 248:       // |   STD killed %5:gpr, %1:gpr, 0         this is I
 249:       //  `----------------.
 250:       //     %6:gpr = LDD %2:gpr, 0
 251:       //     %7:gpr = ADD_rr %0:gpr(tied-def 0), killed %6:gpr <--- (2)
 252:       //     %8:gpr = LDD killed %7:gpr, 0       ^^^^^^^^^^^^^
 253:       //     STD killed %8:gpr, %1:gpr, 0         this is I
 254:       //
 255:       // Instructions (1) and (2) would be updated by setReg() to:
 256:       //
 257:       //     ADD_rr %0:gpr(tied-def 0), %2:gpr
 258:       //
 259:       // %2:gpr is not killed at (1), so it is necessary to remove kill flag
 260:       // from I.
```

- EN: This range continues the implementation of the backend component described by BPFMISimplifyPatchable.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 261-280

```cpp
 261:       I->setReg(SrcReg);
 262:       I->setIsKill(false);
 263:     }
 264: 
 265:     // The candidate needs to have a unique definition.
 266:     if (IsAma && MRI->getUniqueVRegDef(I->getReg()))
 267:       processInst(MRI, I->getParent(), &*I, GVal);
 268:   }
 269: }
 270: 
 271: // Check to see whether we could do some optimization
 272: // to attach relocation to downstream dependent instructions.
 273: // Two kinds of patterns are recognized below:
 274: // Pattern 1:
 275: //   %1 = LD_imm64 @"llvm.b:0:4$0:1"  <== patch_imm = 4
 276: //   %2 = LDD %1, 0  <== this insn will be removed
 277: //   %3 = ADD_rr %0, %2
 278: //   %4 = LDW[32] %3, 0 OR STW[32] %4, %3, 0
 279: //   The `%4 = ...` will be transformed to
 280: //      CORE_[ALU32_]MEM(%4, mem_opcode, %0, @"llvm.b:0:4$0:1")
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 281-300

```cpp
 281: //   and later on, BTF emit phase will translate to
 282: //      %4 = LDW[32] %0, 4 STW[32] %4, %0, 4
 283: //   and attach a relocation to it.
 284: // Pattern 2:
 285: //    %15 = LD_imm64 @"llvm.t:5:63$0:2" <== relocation type 5
 286: //    %16 = LDD %15, 0   <== this insn will be removed
 287: //    %17 = SRA_rr %14, %16
 288: //    The `%17 = ...` will be transformed to
 289: //       %17 = CORE_SHIFT(SRA_ri, %14, @"llvm.t:5:63$0:2")
 290: //    and later on, BTF emit phase will translate to
 291: //       %r4 = SRA_ri %r4, 63
 292: void BPFMISimplifyPatchable::processInst(MachineRegisterInfo *MRI,
 293:     MachineInstr *Inst, MachineOperand *RelocOp, const GlobalValue *GVal) {
 294:   unsigned Opcode = Inst->getOpcode();
 295:   if (isLoadInst(Opcode)) {
 296:     SkipInsts.insert(Inst);
 297:     return;
 298:   }
 299: 
 300:   if (DisableCOREOptimization)
```

- EN: Function bodies or method definitions such as processInst contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: processInst 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 301-320

```cpp
 301:     return;
 302: 
 303:   if (Opcode == BPF::ADD_rr) {
 304:     // If the struct offset is greater than INT16_MAX, skip optimization.
 305:     StringRef AccessPattern = GVal->getName();
 306:     size_t FirstDollar = AccessPattern.find_first_of('$');
 307:     size_t FirstColon = AccessPattern.find_first_of(':');
 308:     size_t SecondColon = AccessPattern.find_first_of(':', FirstColon + 1);
 309:     StringRef PatchImmStr =
 310:         AccessPattern.substr(SecondColon + 1, FirstDollar - SecondColon);
 311:     int PatchImm = std::stoll(std::string(PatchImmStr));
 312:     if (PatchImm <= INT16_MAX)
 313:       checkADDrr(MRI, RelocOp, GVal);
 314:     return;
 315:   }
 316: 
 317:   if (Opcode == BPF::SLL_rr)
 318:     checkShift(MRI, *Inst->getParent(), RelocOp, GVal, BPF::SLL_ri);
 319:   else if (Opcode == BPF::SRA_rr)
 320:     checkShift(MRI, *Inst->getParent(), RelocOp, GVal, BPF::SRA_ri);
```

- EN: This chunk introduces interfaces or data structures such as offset, which organize the target-specific behavior exposed by the file. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这一段引入了 offset 等接口或数据结构，用于组织该文件暴露的目标专用行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 321-340

```cpp
 321:   else if (Opcode == BPF::SRL_rr)
 322:     checkShift(MRI, *Inst->getParent(), RelocOp, GVal, BPF::SRL_ri);
 323: }
 324: 
 325: /// Remove unneeded Load instructions.
 326: bool BPFMISimplifyPatchable::removeLD() {
 327:   MachineRegisterInfo *MRI = &MF->getRegInfo();
 328:   MachineInstr *ToErase = nullptr;
 329:   bool Changed = false;
 330: 
 331:   for (MachineBasicBlock &MBB : *MF) {
 332:     for (MachineInstr &MI : MBB) {
 333:       if (ToErase) {
 334:         ToErase->eraseFromParent();
 335:         ToErase = nullptr;
 336:       }
 337: 
 338:       // Ensure the register format is LOAD <reg>, <reg>, 0
 339:       if (!isLoadInst(MI.getOpcode()))
 340:         continue;
```

- EN: Function bodies or method definitions such as removeLD contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: removeLD 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 341-360

```cpp
 341: 
 342:       if (SkipInsts.find(&MI) != SkipInsts.end())
 343:         continue;
 344: 
 345:       if (!MI.getOperand(0).isReg() || !MI.getOperand(1).isReg())
 346:         continue;
 347: 
 348:       if (!MI.getOperand(2).isImm() || MI.getOperand(2).getImm())
 349:         continue;
 350: 
 351:       Register DstReg = MI.getOperand(0).getReg();
 352:       Register SrcReg = MI.getOperand(1).getReg();
 353: 
 354:       MachineInstr *DefInst = MRI->getUniqueVRegDef(SrcReg);
 355:       if (!DefInst)
 356:         continue;
 357: 
 358:       if (DefInst->getOpcode() != BPF::LD_imm64)
 359:         continue;
 360: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 361-380

```cpp
 361:       const MachineOperand &MO = DefInst->getOperand(1);
 362:       if (!MO.isGlobal())
 363:         continue;
 364: 
 365:       const GlobalValue *GVal = MO.getGlobal();
 366:       auto *GVar = dyn_cast<GlobalVariable>(GVal);
 367:       if (!GVar)
 368:         continue;
 369: 
 370:       // Global variables representing structure offset or type id.
 371:       bool IsAma = false;
 372:       if (GVar->hasAttribute(BPFCoreSharedInfo::AmaAttr))
 373:         IsAma = true;
 374:       else if (!GVar->hasAttribute(BPFCoreSharedInfo::TypeIdAttr))
 375:         continue;
 376: 
 377:       processCandidate(MRI, MBB, MI, SrcReg, DstReg, GVal, IsAma);
 378: 
 379:       ToErase = &MI;
 380:       Changed = true;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 381-395

```cpp
 381:     }
 382:   }
 383: 
 384:   return Changed;
 385: }
 386: 
 387: } // namespace
 388: 
 389: INITIALIZE_PASS(BPFMISimplifyPatchable, DEBUG_TYPE,
 390:                 "BPF PreEmit SimplifyPatchable", false, false)
 391: 
 392: char BPFMISimplifyPatchable::ID = 0;
 393: FunctionPass *llvm::createBPFMISimplifyPatchablePass() {
 394:   return new BPFMISimplifyPatchable();
 395: }
```

- EN: This range continues the implementation of the backend component described by BPFMISimplifyPatchable.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

## Key Concepts / 关键概念

- Machine instruction manipulation / 机器指令操作
- MachineFunction state / MachineFunction 状态
- Basic block level transformation / 基本块级转换
- Target machine configuration / 目标机器配置
- CPU feature modelling / CPU 特性建模

## Dependencies / 依赖关系

- Direct includes / 直接包含: `BPF.h`, `BPFCORE.h`, `BPFInstrInfo.h`, `BPFTargetMachine.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineRegisterInfo.h`, `llvm/IR/GlobalVariable.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen
