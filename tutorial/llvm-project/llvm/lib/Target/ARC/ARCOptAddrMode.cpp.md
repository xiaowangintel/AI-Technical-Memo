# ARCOptAddrMode.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/ARC/ARCOptAddrMode.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Implements target-specific address-mode optimization or canonicalization.
- 目的（中文）: 实现目标专用的寻址模式优化或规范化。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===- ARCOptAddrMode.cpp ---------------------------------------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: /// \file
  10: /// This pass folds LD/ST + ADD pairs into Pre/Post-increment form  of
  11: /// load/store instructions.
  12: //===----------------------------------------------------------------------===//
  13: 
  14: #include "ARC.h"
  15: #define GET_INSTRMAP_INFO
  16: #include "ARCInstrInfo.h"
  17: #include "ARCTargetMachine.h"
  18: #include "llvm/CodeGen/MachineBasicBlock.h"
  19: #include "llvm/CodeGen/MachineDominators.h"
  20: #include "llvm/CodeGen/MachineFunctionPass.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。

### Lines 21-40

```cpp
  21: #include "llvm/CodeGen/MachineInstr.h"
  22: #include "llvm/CodeGen/MachineInstrBuilder.h"
  23: #include "llvm/CodeGen/TargetRegisterInfo.h"
  24: #include "llvm/IR/Function.h"
  25: #include "llvm/InitializePasses.h"
  26: #include "llvm/Support/CommandLine.h"
  27: #include "llvm/Support/Debug.h"
  28: #include "llvm/Support/raw_ostream.h"
  29: 
  30: using namespace llvm;
  31: 
  32: #define OPTADDRMODE_DESC "ARC load/store address mode"
  33: #define OPTADDRMODE_NAME "arc-addr-mode"
  34: #define DEBUG_TYPE "arc-addr-mode"
  35: 
  36: namespace llvm {
  37: 
  38: static cl::opt<unsigned> ArcKillAddrMode("arc-kill-addr-mode", cl::init(0),
  39:                                          cl::ReallyHidden);
  40: 
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。

### Lines 41-60

```cpp
  41: #define DUMP_BEFORE() ((ArcKillAddrMode & 0x0001) != 0)
  42: #define DUMP_AFTER() ((ArcKillAddrMode & 0x0002) != 0)
  43: #define VIEW_BEFORE() ((ArcKillAddrMode & 0x0004) != 0)
  44: #define VIEW_AFTER() ((ArcKillAddrMode & 0x0008) != 0)
  45: #define KILL_PASS() ((ArcKillAddrMode & 0x0010) != 0)
  46: 
  47: FunctionPass *createARCOptAddrMode();
  48: void initializeARCOptAddrModePass(PassRegistry &);
  49: } // end namespace llvm
  50: 
  51: namespace {
  52: class ARCOptAddrMode : public MachineFunctionPass {
  53: public:
  54:   static char ID;
  55: 
  56:   ARCOptAddrMode() : MachineFunctionPass(ID) {}
  57: 
  58:   StringRef getPassName() const override { return OPTADDRMODE_DESC; }
  59: 
  60:   void getAnalysisUsage(AnalysisUsage &AU) const override {
```

- EN: The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. This chunk introduces interfaces or data structures such as ARCOptAddrMode, which organize the target-specific behavior exposed by the file.
- 中文: 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 这一段引入了 ARCOptAddrMode 等接口或数据结构，用于组织该文件暴露的目标专用行为。

### Lines 61-80

```cpp
  61:     AU.setPreservesCFG();
  62:     MachineFunctionPass::getAnalysisUsage(AU);
  63:     AU.addRequired<MachineDominatorTreeWrapperPass>();
  64:     AU.addPreserved<MachineDominatorTreeWrapperPass>();
  65:   }
  66: 
  67:   bool runOnMachineFunction(MachineFunction &MF) override;
  68: 
  69: private:
  70:   const ARCSubtarget *AST = nullptr;
  71:   const ARCInstrInfo *AII = nullptr;
  72:   MachineRegisterInfo *MRI = nullptr;
  73:   MachineDominatorTree *MDT = nullptr;
  74: 
  75:   // Tries to combine \p Ldst with increment of its base register to form
  76:   // single post-increment instruction.
  77:   MachineInstr *tryToCombine(MachineInstr &Ldst);
  78: 
  79:   // Returns true if result of \p Add is not used before \p Ldst
  80:   bool noUseOfAddBeforeLoadOrStore(const MachineInstr *Add,
```

- EN: This range continues the implementation of the backend component described by ARCOptAddrMode.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 81-100

```cpp
  81:                                    const MachineInstr *Ldst);
  82: 
  83:   // Returns true if load/store instruction \p Ldst can be hoisted up to
  84:   // instruction \p To
  85:   bool canHoistLoadStoreTo(MachineInstr *Ldst, MachineInstr *To);
  86: 
  87:   // // Returns true if load/store instruction \p Ldst can be sunk down
  88:   // // to instruction \p To
  89:   // bool canSinkLoadStoreTo(MachineInstr *Ldst, MachineInstr *To);
  90: 
  91:   // Check if instructions \p Ldst and \p Add can be moved to become adjacent
  92:   // If they can return instruction which need not to move.
  93:   // If \p Uses is not null, fill it with instructions after \p Ldst which use
  94:   // \p Ldst's base register
  95:   MachineInstr *canJoinInstructions(MachineInstr *Ldst, MachineInstr *Add,
  96:                                     SmallVectorImpl<MachineInstr *> *Uses);
  97: 
  98:   // Returns true if all instruction in \p Uses array can be adjusted
  99:   // to accomodate increment of register \p BaseReg by \p Incr
 100:   bool canFixPastUses(const ArrayRef<MachineInstr *> &Uses,
```

- EN: This range continues the implementation of the backend component described by ARCOptAddrMode.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 101-120

```cpp
 101:                       MachineOperand &Incr, unsigned BaseReg);
 102: 
 103:   // Update all instructions in \p Uses to accomodate increment
 104:   // of \p BaseReg by \p Offset
 105:   void fixPastUses(ArrayRef<MachineInstr *> Uses, unsigned BaseReg,
 106:                    int64_t Offset);
 107: 
 108:   // Change instruction \p Ldst to postincrement form.
 109:   // \p NewBase is register to hold update base value
 110:   // \p NewOffset is instruction's new offset
 111:   void changeToAddrMode(MachineInstr &Ldst, unsigned NewOpcode,
 112:                         unsigned NewBase, MachineOperand &NewOffset);
 113: 
 114:   bool processBasicBlock(MachineBasicBlock &MBB);
 115: };
 116: 
 117: } // end anonymous namespace
 118: 
 119: char ARCOptAddrMode::ID = 0;
 120: INITIALIZE_PASS_BEGIN(ARCOptAddrMode, OPTADDRMODE_NAME, OPTADDRMODE_DESC, false,
```

- EN: This range continues the implementation of the backend component described by ARCOptAddrMode.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 121-140

```cpp
 121:                       false)
 122: INITIALIZE_PASS_DEPENDENCY(MachineDominatorTreeWrapperPass)
 123: INITIALIZE_PASS_END(ARCOptAddrMode, OPTADDRMODE_NAME, OPTADDRMODE_DESC, false,
 124:                     false)
 125: 
 126: // Return true if \p Off can be used as immediate offset
 127: // operand of load/store instruction (S9 literal)
 128: static bool isValidLoadStoreOffset(int64_t Off) { return isInt<9>(Off); }
 129: 
 130: // Return true if \p Off can be used as immediate operand of
 131: // ADD/SUB instruction (U6 literal)
 132: static bool isValidIncrementOffset(int64_t Off) { return isUInt<6>(Off); }
 133: 
 134: static bool isAddConstantOp(const MachineInstr &MI, int64_t &Amount) {
 135:   int64_t Sign = 1;
 136:   switch (MI.getOpcode()) {
 137:   case ARC::SUB_rru6:
 138:     Sign = -1;
 139:     [[fallthrough]];
 140:   case ARC::ADD_rru6:
```

- EN: Function bodies or method definitions such as isValidLoadStoreOffset, isValidIncrementOffset, isAddConstantOp contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition.
- 中文: isValidLoadStoreOffset, isValidIncrementOffset, isAddConstantOp 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。

### Lines 141-160

```cpp
 141:     assert(MI.getOperand(2).isImm() && "Expected immediate operand");
 142:     Amount = Sign * MI.getOperand(2).getImm();
 143:     return true;
 144:   default:
 145:     return false;
 146:   }
 147: }
 148: 
 149: // Return true if \p MI dominates of uses of virtual register \p VReg
 150: static bool dominatesAllUsesOf(const MachineInstr *MI, unsigned VReg,
 151:                                MachineDominatorTree *MDT,
 152:                                MachineRegisterInfo *MRI) {
 153: 
 154:   assert(Register::isVirtualRegister(VReg) && "Expected virtual register!");
 155: 
 156:   for (const MachineOperand &Use : MRI->use_nodbg_operands(VReg)) {
 157:     const MachineInstr *User = Use.getParent();
 158:     if (User->isPHI()) {
 159:       unsigned BBOperandIdx = Use.getOperandNo() + 1;
 160:       MachineBasicBlock *MBB = User->getOperand(BBOperandIdx).getMBB();
```

- EN: Function bodies or method definitions such as dominatesAllUsesOf contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: dominatesAllUsesOf 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 161-180

```cpp
 161:       if (MBB->empty()) {
 162:         const MachineBasicBlock *InstBB = MI->getParent();
 163:         assert(InstBB != MBB && "Instruction found in empty MBB");
 164:         if (!MDT->dominates(InstBB, MBB))
 165:           return false;
 166:         continue;
 167:       }
 168:       User = &*MBB->rbegin();
 169:     }
 170: 
 171:     if (!MDT->dominates(MI, User))
 172:       return false;
 173:   }
 174:   return true;
 175: }
 176: 
 177: // Return true if \p MI is load/store instruction with immediate offset
 178: // which can be adjusted by \p Disp
 179: static bool isLoadStoreThatCanHandleDisplacement(const TargetInstrInfo *TII,
 180:                                                  const MachineInstr &MI,
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 181-200

```cpp
 181:                                                  int64_t Disp) {
 182:   unsigned BasePos, OffPos;
 183:   if (!TII->getBaseAndOffsetPosition(MI, BasePos, OffPos))
 184:     return false;
 185:   const MachineOperand &MO = MI.getOperand(OffPos);
 186:   if (!MO.isImm())
 187:     return false;
 188:   int64_t Offset = MO.getImm() + Disp;
 189:   return isValidLoadStoreOffset(Offset);
 190: }
 191: 
 192: bool ARCOptAddrMode::noUseOfAddBeforeLoadOrStore(const MachineInstr *Add,
 193:                                                  const MachineInstr *Ldst) {
 194:   Register R = Add->getOperand(0).getReg();
 195:   return dominatesAllUsesOf(Ldst, R, MDT, MRI);
 196: }
 197: 
 198: MachineInstr *ARCOptAddrMode::tryToCombine(MachineInstr &Ldst) {
 199:   assert(Ldst.mayLoadOrStore() && "LD/ST instruction expected");
 200: 
```

- EN: Function bodies or method definitions such as noUseOfAddBeforeLoadOrStore contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: noUseOfAddBeforeLoadOrStore 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 201-220

```cpp
 201:   unsigned BasePos, OffsetPos;
 202: 
 203:   LLVM_DEBUG(dbgs() << "[ABAW] tryToCombine " << Ldst);
 204:   if (!AII->getBaseAndOffsetPosition(Ldst, BasePos, OffsetPos)) {
 205:     LLVM_DEBUG(dbgs() << "[ABAW] Not a recognized load/store\n");
 206:     return nullptr;
 207:   }
 208: 
 209:   MachineOperand &Base = Ldst.getOperand(BasePos);
 210:   MachineOperand &Offset = Ldst.getOperand(OffsetPos);
 211: 
 212:   assert(Base.isReg() && "Base operand must be register");
 213:   if (!Offset.isImm()) {
 214:     LLVM_DEBUG(dbgs() << "[ABAW] Offset is not immediate\n");
 215:     return nullptr;
 216:   }
 217: 
 218:   Register B = Base.getReg();
 219:   if (!Register::isVirtualRegister(B)) {
 220:     LLVM_DEBUG(dbgs() << "[ABAW] Base is not VReg\n");
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 221-240

```cpp
 221:     return nullptr;
 222:   }
 223: 
 224:   // TODO: try to generate address preincrement
 225:   if (Offset.getImm() != 0) {
 226:     LLVM_DEBUG(dbgs() << "[ABAW] Non-zero offset\n");
 227:     return nullptr;
 228:   }
 229: 
 230:   for (auto &Add : MRI->use_nodbg_instructions(B)) {
 231:     int64_t Incr;
 232:     if (!isAddConstantOp(Add, Incr))
 233:       continue;
 234:     if (!isValidLoadStoreOffset(Incr))
 235:       continue;
 236: 
 237:     SmallVector<MachineInstr *, 8> Uses;
 238:     MachineInstr *MoveTo = canJoinInstructions(&Ldst, &Add, &Uses);
 239: 
 240:     if (!MoveTo)
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 241-260

```cpp
 241:       continue;
 242: 
 243:     if (!canFixPastUses(Uses, Add.getOperand(2), B))
 244:       continue;
 245: 
 246:     LLVM_DEBUG(MachineInstr *First = &Ldst; MachineInstr *Last = &Add;
 247:                if (MDT->dominates(Last, First)) std::swap(First, Last);
 248:                dbgs() << "[ABAW] Instructions " << *First << " and " << *Last
 249:                       << " combined\n";
 250: 
 251:     );
 252: 
 253:     MachineInstr *Result = Ldst.getNextNode();
 254:     if (MoveTo == &Add) {
 255:       Ldst.removeFromParent();
 256:       Add.getParent()->insertAfter(Add.getIterator(), &Ldst);
 257:     }
 258:     if (Result == &Add)
 259:       Result = Result->getNextNode();
 260: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 261-280

```cpp
 261:     fixPastUses(Uses, B, Incr);
 262: 
 263:     int NewOpcode = ARC::getPostIncOpcode(Ldst.getOpcode());
 264:     assert(NewOpcode > 0 && "No postincrement form found");
 265:     unsigned NewBaseReg = Add.getOperand(0).getReg();
 266:     changeToAddrMode(Ldst, NewOpcode, NewBaseReg, Add.getOperand(2));
 267:     Add.eraseFromParent();
 268: 
 269:     return Result;
 270:   }
 271:   return nullptr;
 272: }
 273: 
 274: MachineInstr *
 275: ARCOptAddrMode::canJoinInstructions(MachineInstr *Ldst, MachineInstr *Add,
 276:                                     SmallVectorImpl<MachineInstr *> *Uses) {
 277:   assert(Ldst && Add && "NULL instruction passed");
 278: 
 279:   MachineInstr *First = Add;
 280:   MachineInstr *Last = Ldst;
```

- EN: Function bodies or method definitions such as canJoinInstructions contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: canJoinInstructions 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 281-300

```cpp
 281:   if (MDT->dominates(Ldst, Add))
 282:     std::swap(First, Last);
 283:   else if (!MDT->dominates(Add, Ldst))
 284:     return nullptr;
 285: 
 286:   LLVM_DEBUG(dbgs() << "canJoinInstructions: " << *First << *Last);
 287: 
 288:   unsigned BasePos, OffPos;
 289: 
 290:   if (!AII->getBaseAndOffsetPosition(*Ldst, BasePos, OffPos)) {
 291:     LLVM_DEBUG(
 292:         dbgs()
 293:         << "[canJoinInstructions] Cannot determine base/offset position\n");
 294:     return nullptr;
 295:   }
 296: 
 297:   Register BaseReg = Ldst->getOperand(BasePos).getReg();
 298: 
 299:   // prohibit this:
 300:   //   v1 = add v0, c
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 301-320

```cpp
 301:   //   st v1, [v0, 0]
 302:   // and this
 303:   //   st v0, [v0, 0]
 304:   //   v1 = add v0, c
 305:   if (Ldst->mayStore() && Ldst->getOperand(0).isReg()) {
 306:     Register StReg = Ldst->getOperand(0).getReg();
 307:     if (Add->getOperand(0).getReg() == StReg || BaseReg == StReg) {
 308:       LLVM_DEBUG(dbgs() << "[canJoinInstructions] Store uses result of Add\n");
 309:       return nullptr;
 310:     }
 311:   }
 312: 
 313:   SmallVector<MachineInstr *, 4> UsesAfterLdst;
 314:   SmallVector<MachineInstr *, 4> UsesAfterAdd;
 315:   for (MachineInstr &MI : MRI->use_nodbg_instructions(BaseReg)) {
 316:     if (&MI == Ldst || &MI == Add)
 317:       continue;
 318:     if (&MI != Add && MDT->dominates(Ldst, &MI))
 319:       UsesAfterLdst.push_back(&MI);
 320:     else if (!MDT->dominates(&MI, Ldst))
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 321-340

```cpp
 321:       return nullptr;
 322:     if (MDT->dominates(Add, &MI))
 323:       UsesAfterAdd.push_back(&MI);
 324:   }
 325: 
 326:   MachineInstr *Result = nullptr;
 327: 
 328:   if (First == Add) {
 329:     //  n = add b, i
 330:     //  ...
 331:     //  x = ld [b, o] or x = ld [n, o]
 332: 
 333:     if (noUseOfAddBeforeLoadOrStore(First, Last)) {
 334:       Result = Last;
 335:       LLVM_DEBUG(dbgs() << "[canJoinInstructions] Can sink Add down to Ldst\n");
 336:     } else if (canHoistLoadStoreTo(Ldst, Add)) {
 337:       Result = First;
 338:       LLVM_DEBUG(dbgs() << "[canJoinInstructions] Can hoist Ldst to Add\n");
 339:     }
 340:   } else {
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 341-360

```cpp
 341:     // x = ld [b, o]
 342:     // ...
 343:     // n = add b, i
 344:     Result = First;
 345:     LLVM_DEBUG(dbgs() << "[canJoinInstructions] Can hoist Add to Ldst\n");
 346:   }
 347:   if (Result && Uses)
 348:     *Uses = (Result == Ldst) ? UsesAfterLdst : UsesAfterAdd;
 349:   return Result;
 350: }
 351: 
 352: bool ARCOptAddrMode::canFixPastUses(const ArrayRef<MachineInstr *> &Uses,
 353:                                     MachineOperand &Incr, unsigned BaseReg) {
 354: 
 355:   assert(Incr.isImm() && "Expected immediate increment");
 356:   int64_t NewOffset = Incr.getImm();
 357:   for (MachineInstr *MI : Uses) {
 358:     int64_t Dummy;
 359:     if (isAddConstantOp(*MI, Dummy)) {
 360:       if (isValidIncrementOffset(Dummy + NewOffset))
```

- EN: Function bodies or method definitions such as canFixPastUses contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: canFixPastUses 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 361-380

```cpp
 361:         continue;
 362:       return false;
 363:     }
 364:     if (isLoadStoreThatCanHandleDisplacement(AII, *MI, -NewOffset))
 365:       continue;
 366:     LLVM_DEBUG(dbgs() << "Instruction cannot handle displacement " << -NewOffset
 367:                       << ": " << *MI);
 368:     return false;
 369:   }
 370:   return true;
 371: }
 372: 
 373: void ARCOptAddrMode::fixPastUses(ArrayRef<MachineInstr *> Uses,
 374:                                  unsigned NewBase, int64_t NewOffset) {
 375: 
 376:   for (MachineInstr *MI : Uses) {
 377:     int64_t Amount;
 378:     unsigned BasePos, OffPos;
 379:     if (isAddConstantOp(*MI, Amount)) {
 380:       NewOffset += Amount;
```

- EN: Function bodies or method definitions such as fixPastUses contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: fixPastUses 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 381-400

```cpp
 381:       assert(isValidIncrementOffset(NewOffset) &&
 382:              "New offset won't fit into ADD instr");
 383:       BasePos = 1;
 384:       OffPos = 2;
 385:     } else if (AII->getBaseAndOffsetPosition(*MI, BasePos, OffPos)) {
 386:       MachineOperand &MO = MI->getOperand(OffPos);
 387:       assert(MO.isImm() && "expected immediate operand");
 388:       NewOffset += MO.getImm();
 389:       assert(isValidLoadStoreOffset(NewOffset) &&
 390:              "New offset won't fit into LD/ST");
 391:     } else
 392:       llvm_unreachable("unexpected instruction");
 393: 
 394:     MI->getOperand(BasePos).setReg(NewBase);
 395:     MI->getOperand(OffPos).setImm(NewOffset);
 396:   }
 397: }
 398: 
 399: bool ARCOptAddrMode::canHoistLoadStoreTo(MachineInstr *Ldst, MachineInstr *To) {
 400:   if (Ldst->getParent() != To->getParent())
```

- EN: Function bodies or method definitions such as canHoistLoadStoreTo contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: canHoistLoadStoreTo 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 401-420

```cpp
 401:     return false;
 402:   MachineBasicBlock::const_iterator MI(To), ME(Ldst),
 403:       End(Ldst->getParent()->end());
 404: 
 405:   bool IsStore = Ldst->mayStore();
 406:   for (; MI != ME && MI != End; ++MI) {
 407:     if (MI->isDebugValue())
 408:       continue;
 409:     if (MI->mayStore() || MI->isCall() || MI->isInlineAsm() ||
 410:         MI->hasUnmodeledSideEffects())
 411:       return false;
 412:     if (IsStore && MI->mayLoad())
 413:       return false;
 414:   }
 415: 
 416:   for (auto &O : Ldst->explicit_operands()) {
 417:     if (!O.isReg() || !O.isUse())
 418:       continue;
 419:     MachineInstr *OpDef = MRI->getVRegDef(O.getReg());
 420:     if (!OpDef || !MDT->dominates(OpDef, To))
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 421-440

```cpp
 421:       return false;
 422:   }
 423:   return true;
 424: }
 425: 
 426: // bool ARCOptAddrMode::canSinkLoadStoreTo(MachineInstr *Ldst, MachineInstr *To) {
 427: //   // Can only sink load/store within same BB
 428: //   if (Ldst->getParent() != To->getParent())
 429: //     return false;
 430: //   MachineBasicBlock::const_iterator MI(Ldst), ME(To),
 431: //       End(Ldst->getParent()->end());
 432: 
 433: //   bool IsStore = Ldst->mayStore();
 434: //   bool IsLoad = Ldst->mayLoad();
 435: 
 436: //   Register ValReg = IsLoad ? Ldst->getOperand(0).getReg() : Register();
 437: //   for (; MI != ME && MI != End; ++MI) {
 438: //     if (MI->isDebugValue())
 439: //       continue;
 440: //     if (MI->mayStore() || MI->isCall() || MI->isInlineAsm() ||
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 441-460

```cpp
 441: //         MI->hasUnmodeledSideEffects())
 442: //       return false;
 443: //     if (IsStore && MI->mayLoad())
 444: //       return false;
 445: //     if (ValReg && MI->readsVirtualRegister(ValReg))
 446: //       return false;
 447: //   }
 448: //   return true;
 449: // }
 450: 
 451: void ARCOptAddrMode::changeToAddrMode(MachineInstr &Ldst, unsigned NewOpcode,
 452:                                       unsigned NewBase,
 453:                                       MachineOperand &NewOffset) {
 454:   bool IsStore = Ldst.mayStore();
 455:   unsigned BasePos, OffPos;
 456:   MachineOperand Src = MachineOperand::CreateImm(0xDEADBEEF);
 457:   AII->getBaseAndOffsetPosition(Ldst, BasePos, OffPos);
 458: 
 459:   Register BaseReg = Ldst.getOperand(BasePos).getReg();
 460: 
```

- EN: Function bodies or method definitions such as changeToAddrMode contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: changeToAddrMode 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 461-480

```cpp
 461:   Ldst.removeOperand(OffPos);
 462:   Ldst.removeOperand(BasePos);
 463: 
 464:   if (IsStore) {
 465:     Src = Ldst.getOperand(BasePos - 1);
 466:     Ldst.removeOperand(BasePos - 1);
 467:   }
 468: 
 469:   Ldst.setDesc(AST->getInstrInfo()->get(NewOpcode));
 470:   Ldst.addOperand(MachineOperand::CreateReg(NewBase, true));
 471:   if (IsStore)
 472:     Ldst.addOperand(Src);
 473:   Ldst.addOperand(MachineOperand::CreateReg(BaseReg, false));
 474:   Ldst.addOperand(NewOffset);
 475:   LLVM_DEBUG(dbgs() << "[ABAW] New Ldst: " << Ldst);
 476: }
 477: 
 478: bool ARCOptAddrMode::processBasicBlock(MachineBasicBlock &MBB) {
 479:   bool Changed = false;
 480:   for (auto MI = MBB.begin(), ME = MBB.end(); MI != ME; ++MI) {
```

- EN: Function bodies or method definitions such as processBasicBlock contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: processBasicBlock 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 481-500

```cpp
 481:     if (MI->isDebugValue())
 482:       continue;
 483:     if (!MI->mayLoad() && !MI->mayStore())
 484:       continue;
 485:     if (ARC::getPostIncOpcode(MI->getOpcode()) < 0)
 486:       continue;
 487:     MachineInstr *Res = tryToCombine(*MI);
 488:     if (Res) {
 489:       Changed = true;
 490:       // Res points to the next instruction. Rewind to process it
 491:       MI = std::prev(Res->getIterator());
 492:     }
 493:   }
 494:   return Changed;
 495: }
 496: 
 497: bool ARCOptAddrMode::runOnMachineFunction(MachineFunction &MF) {
 498:   if (skipFunction(MF.getFunction()) || KILL_PASS())
 499:     return false;
 500: 
```

- EN: Function bodies or method definitions such as runOnMachineFunction contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: runOnMachineFunction 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 501-520

```cpp
 501: #if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
 502:   if (DUMP_BEFORE())
 503:     MF.dump();
 504: #endif
 505:   if (VIEW_BEFORE())
 506:     MF.viewCFG();
 507: 
 508:   AST = &MF.getSubtarget<ARCSubtarget>();
 509:   AII = AST->getInstrInfo();
 510:   MRI = &MF.getRegInfo();
 511:   MDT = &getAnalysis<MachineDominatorTreeWrapperPass>().getDomTree();
 512: 
 513:   bool Changed = false;
 514:   for (auto &MBB : MF)
 515:     Changed |= processBasicBlock(MBB);
 516: 
 517: #if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
 518:   if (DUMP_AFTER())
 519:     MF.dump();
 520: #endif
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 521-530

```cpp
 521:   if (VIEW_AFTER())
 522:     MF.viewCFG();
 523:   return Changed;
 524: }
 525: 
 526: //===----------------------------------------------------------------------===//
 527: //                         Public Constructor Functions
 528: //===----------------------------------------------------------------------===//
 529: 
 530: FunctionPass *llvm::createARCOptAddrMode() { return new ARCOptAddrMode(); }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

## Key Concepts / 关键概念

- Machine instruction manipulation / 机器指令操作
- MachineFunction state / MachineFunction 状态
- Basic block level transformation / 基本块级转换
- Instruction semantics helpers / 指令语义辅助逻辑
- Target machine configuration / 目标机器配置
- CPU feature modelling / CPU 特性建模

## Dependencies / 依赖关系

- Direct includes / 直接包含: `ARC.h`, `ARCInstrInfo.h`, `ARCTargetMachine.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineDominators.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/MachineInstr.h`, `llvm/CodeGen/MachineInstrBuilder.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen
