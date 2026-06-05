# SystemZLongBranch.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZLongBranch.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Provides target-specific implementation details for the SystemZ backend.
- **用途 (CN)**: 提供 SystemZ 后端的目标相关实现细节。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
```cpp
   1: //===-- SystemZLongBranch.cpp - Branch lengthening for SystemZ ------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This pass makes sure that all branches are in range.  There are several ways
  10: // in which this could be done.  One aggressive approach is to assume that all
  11: // branches are in range and successively replace those that turn out not
  12: // to be in range with a longer form (branch relaxation).  A simple
  13: // implementation is to continually walk through the function relaxing
  14: // branches until no more changes are needed and a fixed point is reached.
  15: // However, in the pathological worst case, this implementation is
  16: // quadratic in the number of blocks; relaxing branch N can make branch N-1
  17: // go out of range, which in turn can make branch N-2 go out of range,
  18: // and so on.
  19: //
  20: // An alternative approach is to assume that all branches must be
  21: // converted to their long forms, then reinstate the short forms of
  22: // branches that, even under this pessimistic assumption, turn out to be
  23: // in range (branch shortening).  This too can be implemented as a function
  24: // walk that is repeated until a fixed point is reached.  In general,
  25: // the result of shortening is not as good as that of relaxation, and
  26: // shortening is also quadratic in the worst case; shortening branch N
  27: // can bring branch N-1 in range of the short form, which in turn can do
  28: // the same for branch N-2, and so on.  The main advantage of shortening
```
- **EN**: This range is mainly descriptive commentary or banner text that frames the file before executable definitions begin.
- **CN**: 这一段主要是说明性注释或横幅文本，用于在可执行定义开始前说明文件背景。

### Lines 29-56 / 第 29-56 行
```cpp
  29: // is that each walk through the function produces valid code, so it is
  30: // possible to stop at any point after the first walk.  The quadraticness
  31: // could therefore be handled with a maximum pass count, although the
  32: // question then becomes: what maximum count should be used?
  33: //
  34: // On SystemZ, long branches are only needed for functions bigger than 64k,
  35: // which are relatively rare to begin with, and the long branch sequences
  36: // are actually relatively cheap.  It therefore doesn't seem worth spending
  37: // much compilation time on the problem.  Instead, the approach we take is:
  38: //
  39: // (1) Work out the address that each block would have if no branches
  40: //     need relaxing.  Exit the pass early if all branches are in range
  41: //     according to this assumption.
  42: //
  43: // (2) Work out the address that each block would have if all branches
  44: //     need relaxing.
  45: //
  46: // (3) Walk through the block calculating the final address of each instruction
  47: //     and relaxing those that need to be relaxed.  For backward branches,
  48: //     this check uses the final address of the target block, as calculated
  49: //     earlier in the walk.  For forward branches, this check uses the
  50: //     address of the target block that was calculated in (2).  Both checks
  51: //     give a conservatively-correct range.
  52: //
  53: //===----------------------------------------------------------------------===//
  54: 
  55: #include "SystemZ.h"
  56: #include "SystemZInstrInfo.h"
```
- **EN**: It imports dependencies such as `SystemZ.h`, `SystemZInstrInfo.h` that expose the LLVM and target interfaces used in later logic. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 它引入了 `SystemZ.h`, `SystemZInstrInfo.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 57-84 / 第 57-84 行
```cpp
  57: #include "SystemZTargetMachine.h"
  58: #include "llvm/ADT/SmallVector.h"
  59: #include "llvm/ADT/Statistic.h"
  60: #include "llvm/ADT/StringRef.h"
  61: #include "llvm/CodeGen/MachineBasicBlock.h"
  62: #include "llvm/CodeGen/MachineFunction.h"
  63: #include "llvm/CodeGen/MachineFunctionPass.h"
  64: #include "llvm/CodeGen/MachineInstr.h"
  65: #include "llvm/CodeGen/MachineInstrBuilder.h"
  66: #include "llvm/IR/DebugLoc.h"
  67: #include "llvm/Support/ErrorHandling.h"
  68: #include <cassert>
  69: #include <cstdint>
  70: 
  71: using namespace llvm;
  72: 
  73: #define DEBUG_TYPE "systemz-long-branch"
  74: 
  75: STATISTIC(LongBranches, "Number of long branches.");
  76: 
  77: namespace {
  78: 
  79: // Represents positional information about a basic block.
  80: struct MBBInfo {
  81:   // The address that we currently assume the block has.
  82:   uint64_t Address = 0;
  83: 
  84:   // The size of the block in bytes, excluding terminators.
```
- **EN**: It imports dependencies such as `SystemZTargetMachine.h`, `SmallVector.h`, `Statistic.h`, `StringRef.h`, `MachineBasicBlock.h`, `MachineFunction.h` that expose the LLVM and target interfaces used in later logic. It introduces interface types such as `MBBInfo`, shaping how other backend components interact with this file. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 它引入了 `SystemZTargetMachine.h`, `SmallVector.h`, `Statistic.h`, `StringRef.h`, `MachineBasicBlock.h`, `MachineFunction.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 它引入了 `MBBInfo` 等接口类型，定义了其他后端组件与本文件交互的方式。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 85-112 / 第 85-112 行
```cpp
  85:   // This value never changes.
  86:   uint64_t Size = 0;
  87: 
  88:   // The minimum alignment of the block.
  89:   // This value never changes.
  90:   Align Alignment;
  91: 
  92:   // The number of terminators in this block.  This value never changes.
  93:   unsigned NumTerminators = 0;
  94: 
  95:   MBBInfo() = default;
  96: };
  97: 
  98: // Represents the state of a block terminator.
  99: struct TerminatorInfo {
 100:   // If this terminator is a relaxable branch, this points to the branch
 101:   // instruction, otherwise it is null.
 102:   MachineInstr *Branch = nullptr;
 103: 
 104:   // The address that we currently assume the terminator has.
 105:   uint64_t Address = 0;
 106: 
 107:   // The current size of the terminator in bytes.
 108:   uint64_t Size = 0;
 109: 
 110:   // If Branch is nonnull, this is the number of the target block,
 111:   // otherwise it is unused.
 112:   unsigned TargetBlock = 0;
```
- **EN**: It introduces interface types such as `TerminatorInfo`, shaping how other backend components interact with this file.
- **CN**: 它引入了 `TerminatorInfo` 等接口类型，定义了其他后端组件与本文件交互的方式。

### Lines 113-140 / 第 113-140 行
```cpp
 113: 
 114:   // If Branch is nonnull, this is the length of the longest relaxed form,
 115:   // otherwise it is zero.
 116:   unsigned ExtraRelaxSize = 0;
 117: 
 118:   TerminatorInfo() = default;
 119: };
 120: 
 121: // Used to keep track of the current position while iterating over the blocks.
 122: struct BlockPosition {
 123:   // The address that we assume this position has.
 124:   uint64_t Address = 0;
 125: 
 126:   // The number of low bits in Address that are known to be the same
 127:   // as the runtime address.
 128:   unsigned KnownBits;
 129: 
 130:   BlockPosition(unsigned InitialLogAlignment)
 131:       : KnownBits(InitialLogAlignment) {}
 132: };
 133: 
 134: class SystemZLongBranch : public MachineFunctionPass {
 135: public:
 136:   static char ID;
 137: 
 138:   SystemZLongBranch() : MachineFunctionPass(ID) {}
 139: 
 140:   bool runOnMachineFunction(MachineFunction &F) override;
```
- **EN**: This block declares or refines TableGen records such as `SystemZLongBranch`. The range implements or declares functions including `BlockPosition`, `SystemZLongBranch`.
- **CN**: 该代码块声明或细化了 `SystemZLongBranch` 等 TableGen 记录。 这一段实现或声明了 `BlockPosition`, `SystemZLongBranch` 等函数。

### Lines 141-168 / 第 141-168 行
```cpp
 141: 
 142:   MachineFunctionProperties getRequiredProperties() const override {
 143:     return MachineFunctionProperties().setNoVRegs();
 144:   }
 145: 
 146: private:
 147:   void skipNonTerminators(BlockPosition &Position, MBBInfo &Block);
 148:   void skipTerminator(BlockPosition &Position, TerminatorInfo &Terminator,
 149:                       bool AssumeRelaxed);
 150:   TerminatorInfo describeTerminator(MachineInstr &MI);
 151:   uint64_t initMBBInfo();
 152:   bool mustRelaxBranch(const TerminatorInfo &Terminator, uint64_t Address);
 153:   bool mustRelaxABranch();
 154:   void setWorstCaseAddresses();
 155:   void splitBranchOnCount(MachineInstr *MI, unsigned AddOpcode);
 156:   void splitCompareBranch(MachineInstr *MI, unsigned CompareOpcode);
 157:   void relaxBranch(TerminatorInfo &Terminator);
 158:   void relaxBranches();
 159: 
 160:   const SystemZInstrInfo *TII = nullptr;
 161:   MachineFunction *MF = nullptr;
 162:   SmallVector<MBBInfo, 16> MBBs;
 163:   SmallVector<TerminatorInfo, 16> Terminators;
 164: };
 165: 
 166: char SystemZLongBranch::ID = 0;
 167: 
 168: const uint64_t MaxBackwardRange = 0x10000;
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 169-196 / 第 169-196 行
```cpp
 169: const uint64_t MaxForwardRange = 0xfffe;
 170: 
 171: } // end anonymous namespace
 172: 
 173: INITIALIZE_PASS(SystemZLongBranch, DEBUG_TYPE, "SystemZ Long Branch", false,
 174:                 false)
 175: 
 176: // Position describes the state immediately before Block.  Update Block
 177: // accordingly and move Position to the end of the block's non-terminator
 178: // instructions.
 179: void SystemZLongBranch::skipNonTerminators(BlockPosition &Position,
 180:                                            MBBInfo &Block) {
 181:   if (Log2(Block.Alignment) > Position.KnownBits) {
 182:     // When calculating the address of Block, we need to conservatively
 183:     // assume that Block had the worst possible misalignment.
 184:     Position.Address +=
 185:         (Block.Alignment.value() - (uint64_t(1) << Position.KnownBits));
 186:     Position.KnownBits = Log2(Block.Alignment);
 187:   }
 188: 
 189:   // Align the addresses.
 190:   Position.Address = alignTo(Position.Address, Block.Alignment);
 191: 
 192:   // Record the block's position.
 193:   Block.Address = Position.Address;
 194: 
 195:   // Move past the non-terminators in the block.
 196:   Position.Address += Block.Size;
```
- **EN**: The range implements or declares functions including `SystemZLongBranch::skipNonTerminators`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZLongBranch::skipNonTerminators` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 197-224 / 第 197-224 行
```cpp
 197: }
 198: 
 199: // Position describes the state immediately before Terminator.
 200: // Update Terminator accordingly and move Position past it.
 201: // Assume that Terminator will be relaxed if AssumeRelaxed.
 202: void SystemZLongBranch::skipTerminator(BlockPosition &Position,
 203:                                        TerminatorInfo &Terminator,
 204:                                        bool AssumeRelaxed) {
 205:   Terminator.Address = Position.Address;
 206:   Position.Address += Terminator.Size;
 207:   if (AssumeRelaxed)
 208:     Position.Address += Terminator.ExtraRelaxSize;
 209: }
 210: 
 211: static unsigned getInstSizeInBytes(const MachineInstr &MI,
 212:                                    const SystemZInstrInfo *TII) {
 213:   unsigned Size = TII->getInstSizeInBytes(MI);
 214:   assert((Size ||
 215:           // These do not have a size:
 216:           MI.isDebugOrPseudoInstr() || MI.isPosition() || MI.isKill() ||
 217:           MI.isImplicitDef() || MI.getOpcode() == TargetOpcode::MEMBARRIER ||
 218:           MI.getOpcode() == TargetOpcode::INIT_UNDEF || MI.isFakeUse() ||
 219:           MI.getOpcode() == TargetOpcode::RELOC_NONE ||
 220:           // These have a size that may be zero:
 221:           MI.isInlineAsm() || MI.getOpcode() == SystemZ::STACKMAP ||
 222:           MI.getOpcode() == SystemZ::PATCHPOINT ||
 223:           // EH_SjLj_Setup is a dummy terminator instruction of size 0,
 224:           // It is used to handle the clobber register for builtin setjmp.
```
- **EN**: The range implements or declares functions including `SystemZLongBranch::skipTerminator`, `getInstSizeInBytes`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `SystemZLongBranch::skipTerminator`, `getInstSizeInBytes` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 225-252 / 第 225-252 行
```cpp
 225:           MI.getOpcode() == SystemZ::EH_SjLj_Setup) &&
 226:          "Missing size value for instruction.");
 227:   return Size;
 228: }
 229: 
 230: // Return a description of terminator instruction MI.
 231: TerminatorInfo SystemZLongBranch::describeTerminator(MachineInstr &MI) {
 232:   TerminatorInfo Terminator;
 233:   Terminator.Size = getInstSizeInBytes(MI, TII);
 234:   if (MI.isConditionalBranch() || MI.isUnconditionalBranch()) {
 235:     switch (MI.getOpcode()) {
 236:     case SystemZ::J:
 237:       // Relaxes to JG, which is 2 bytes longer.
 238:       Terminator.ExtraRelaxSize = 2;
 239:       break;
 240:     case SystemZ::BRC:
 241:       // Relaxes to BRCL, which is 2 bytes longer.
 242:       Terminator.ExtraRelaxSize = 2;
 243:       break;
 244:     case SystemZ::BRCT:
 245:     case SystemZ::BRCTG:
 246:       // Relaxes to A(G)HI and BRCL, which is 6 bytes longer.
 247:       Terminator.ExtraRelaxSize = 6;
 248:       break;
 249:     case SystemZ::BRCTH:
 250:       // Never needs to be relaxed.
 251:       Terminator.ExtraRelaxSize = 0;
 252:       break;
```
- **EN**: The range implements or declares functions including `SystemZLongBranch::describeTerminator`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZLongBranch::describeTerminator` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 253-280 / 第 253-280 行
```cpp
 253:     case SystemZ::CRJ:
 254:     case SystemZ::CLRJ:
 255:       // Relaxes to a C(L)R/BRCL sequence, which is 2 bytes longer.
 256:       Terminator.ExtraRelaxSize = 2;
 257:       break;
 258:     case SystemZ::CGRJ:
 259:     case SystemZ::CLGRJ:
 260:       // Relaxes to a C(L)GR/BRCL sequence, which is 4 bytes longer.
 261:       Terminator.ExtraRelaxSize = 4;
 262:       break;
 263:     case SystemZ::CIJ:
 264:     case SystemZ::CGIJ:
 265:       // Relaxes to a C(G)HI/BRCL sequence, which is 4 bytes longer.
 266:       Terminator.ExtraRelaxSize = 4;
 267:       break;
 268:     case SystemZ::CLIJ:
 269:     case SystemZ::CLGIJ:
 270:       // Relaxes to a CL(G)FI/BRCL sequence, which is 6 bytes longer.
 271:       Terminator.ExtraRelaxSize = 6;
 272:       break;
 273:     default:
 274:       llvm_unreachable("Unrecognized branch instruction");
 275:     }
 276:     Terminator.Branch = &MI;
 277:     Terminator.TargetBlock =
 278:       TII->getBranchInfo(MI).getMBBTarget()->getNumber();
 279:   }
 280:   return Terminator;
```
- **EN**: The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 281-308 / 第 281-308 行
```cpp
 281: }
 282: 
 283: // Fill MBBs and Terminators, setting the addresses on the assumption
 284: // that no branches need relaxation.  Return the size of the function under
 285: // this assumption.
 286: uint64_t SystemZLongBranch::initMBBInfo() {
 287:   MF->RenumberBlocks();
 288:   unsigned NumBlocks = MF->size();
 289: 
 290:   MBBs.clear();
 291:   MBBs.resize(NumBlocks);
 292: 
 293:   Terminators.clear();
 294:   Terminators.reserve(NumBlocks);
 295: 
 296:   BlockPosition Position(Log2(MF->getAlignment()));
 297:   for (unsigned I = 0; I < NumBlocks; ++I) {
 298:     MachineBasicBlock *MBB = MF->getBlockNumbered(I);
 299:     MBBInfo &Block = MBBs[I];
 300: 
 301:     // Record the alignment, for quick access.
 302:     Block.Alignment = MBB->getAlignment();
 303: 
 304:     // Calculate the size of the fixed part of the block.
 305:     MachineBasicBlock::iterator MI = MBB->begin();
 306:     MachineBasicBlock::iterator End = MBB->end();
 307:     while (MI != End && !MI->isTerminator()) {
 308:       Block.Size += getInstSizeInBytes(*MI, TII);
```
- **EN**: The range implements or declares functions including `SystemZLongBranch::initMBBInfo`. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZLongBranch::initMBBInfo` 等函数。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 309-336 / 第 309-336 行
```cpp
 309:       ++MI;
 310:     }
 311:     skipNonTerminators(Position, Block);
 312: 
 313:     // Add the terminators.
 314:     while (MI != End) {
 315:       if (!MI->isDebugInstr()) {
 316:         assert(MI->isTerminator() && "Terminator followed by non-terminator");
 317:         Terminators.push_back(describeTerminator(*MI));
 318:         skipTerminator(Position, Terminators.back(), false);
 319:         ++Block.NumTerminators;
 320:       }
 321:       ++MI;
 322:     }
 323:   }
 324: 
 325:   return Position.Address;
 326: }
 327: 
 328: // Return true if, under current assumptions, Terminator would need to be
 329: // relaxed if it were placed at address Address.
 330: bool SystemZLongBranch::mustRelaxBranch(const TerminatorInfo &Terminator,
 331:                                         uint64_t Address) {
 332:   if (!Terminator.Branch || Terminator.ExtraRelaxSize == 0)
 333:     return false;
 334: 
 335:   const MBBInfo &Target = MBBs[Terminator.TargetBlock];
 336:   if (Address >= Target.Address) {
```
- **EN**: The range implements or declares functions including `SystemZLongBranch::mustRelaxBranch`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZLongBranch::mustRelaxBranch` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 337-364 / 第 337-364 行
```cpp
 337:     if (Address - Target.Address <= MaxBackwardRange)
 338:       return false;
 339:   } else {
 340:     if (Target.Address - Address <= MaxForwardRange)
 341:       return false;
 342:   }
 343: 
 344:   return true;
 345: }
 346: 
 347: // Return true if, under current assumptions, any terminator needs
 348: // to be relaxed.
 349: bool SystemZLongBranch::mustRelaxABranch() {
 350:   for (auto &Terminator : Terminators)
 351:     if (mustRelaxBranch(Terminator, Terminator.Address))
 352:       return true;
 353:   return false;
 354: }
 355: 
 356: // Set the address of each block on the assumption that all branches
 357: // must be long.
 358: void SystemZLongBranch::setWorstCaseAddresses() {
 359:   SmallVector<TerminatorInfo, 16>::iterator TI = Terminators.begin();
 360:   BlockPosition Position(Log2(MF->getAlignment()));
 361:   for (auto &Block : MBBs) {
 362:     skipNonTerminators(Position, Block);
 363:     for (unsigned BTI = 0, BTE = Block.NumTerminators; BTI != BTE; ++BTI) {
 364:       skipTerminator(Position, *TI, true);
```
- **EN**: The range implements or declares functions including `SystemZLongBranch::mustRelaxABranch`, `SystemZLongBranch::setWorstCaseAddresses`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZLongBranch::mustRelaxABranch`, `SystemZLongBranch::setWorstCaseAddresses` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 365-392 / 第 365-392 行
```cpp
 365:       ++TI;
 366:     }
 367:   }
 368: }
 369: 
 370: // Split BRANCH ON COUNT MI into the addition given by AddOpcode followed
 371: // by a BRCL on the result.
 372: void SystemZLongBranch::splitBranchOnCount(MachineInstr *MI,
 373:                                            unsigned AddOpcode) {
 374:   MachineBasicBlock *MBB = MI->getParent();
 375:   DebugLoc DL = MI->getDebugLoc();
 376:   MachineInstr *AddImm = BuildMI(*MBB, MI, DL, TII->get(AddOpcode))
 377:                              .add(MI->getOperand(0))
 378:                              .add(MI->getOperand(1))
 379:                              .addImm(-1);
 380:   MachineInstr *BRCL = BuildMI(*MBB, MI, DL, TII->get(SystemZ::BRCL))
 381:                            .addImm(SystemZ::CCMASK_ICMP)
 382:                            .addImm(SystemZ::CCMASK_CMP_NE)
 383:                            .add(MI->getOperand(2));
 384:   // The implicit use of CC is a killing use.
 385:   BRCL->addRegisterKilled(SystemZ::CC, &TII->getRegisterInfo());
 386:   // The result of the BRANCH ON COUNT MI is the new count in register 0, so the
 387:   // debug tracking needs to go to the result of the Add immediate.
 388:   MBB->getParent()->substituteDebugValuesForInst(*MI, *AddImm);
 389:   MI->eraseFromParent();
 390: }
 391: 
 392: // Split MI into the comparison given by CompareOpcode followed
```
- **EN**: The range implements or declares functions including `SystemZLongBranch::splitBranchOnCount`.
- **CN**: 这一段实现或声明了 `SystemZLongBranch::splitBranchOnCount` 等函数。

### Lines 393-420 / 第 393-420 行
```cpp
 393: // a BRCL on the result.
 394: void SystemZLongBranch::splitCompareBranch(MachineInstr *MI,
 395:                                            unsigned CompareOpcode) {
 396:   MachineBasicBlock *MBB = MI->getParent();
 397:   DebugLoc DL = MI->getDebugLoc();
 398:   BuildMI(*MBB, MI, DL, TII->get(CompareOpcode))
 399:       .add(MI->getOperand(0))
 400:       .add(MI->getOperand(1));
 401:   MachineInstr *BRCL = BuildMI(*MBB, MI, DL, TII->get(SystemZ::BRCL))
 402:                            .addImm(SystemZ::CCMASK_ICMP)
 403:                            .add(MI->getOperand(2))
 404:                            .add(MI->getOperand(3));
 405:   // The implicit use of CC is a killing use.
 406:   BRCL->addRegisterKilled(SystemZ::CC, &TII->getRegisterInfo());
 407:   // Since we are replacing branches that did not compute any value, no debug
 408:   // value substitution is necessary.
 409:   MI->eraseFromParent();
 410: }
 411: 
 412: // Relax the branch described by Terminator.
 413: void SystemZLongBranch::relaxBranch(TerminatorInfo &Terminator) {
 414:   MachineInstr *Branch = Terminator.Branch;
 415:   switch (Branch->getOpcode()) {
 416:   case SystemZ::J:
 417:     Branch->setDesc(TII->get(SystemZ::JG));
 418:     break;
 419:   case SystemZ::BRC:
 420:     Branch->setDesc(TII->get(SystemZ::BRCL));
```
- **EN**: The range implements or declares functions including `SystemZLongBranch::splitCompareBranch`, `BuildMI`, `SystemZLongBranch::relaxBranch`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这一段实现或声明了 `SystemZLongBranch::splitCompareBranch`, `BuildMI`, `SystemZLongBranch::relaxBranch` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 421-448 / 第 421-448 行
```cpp
 421:     break;
 422:   case SystemZ::BRCT:
 423:     splitBranchOnCount(Branch, SystemZ::AHI);
 424:     break;
 425:   case SystemZ::BRCTG:
 426:     splitBranchOnCount(Branch, SystemZ::AGHI);
 427:     break;
 428:   case SystemZ::CRJ:
 429:     splitCompareBranch(Branch, SystemZ::CR);
 430:     break;
 431:   case SystemZ::CGRJ:
 432:     splitCompareBranch(Branch, SystemZ::CGR);
 433:     break;
 434:   case SystemZ::CIJ:
 435:     splitCompareBranch(Branch, SystemZ::CHI);
 436:     break;
 437:   case SystemZ::CGIJ:
 438:     splitCompareBranch(Branch, SystemZ::CGHI);
 439:     break;
 440:   case SystemZ::CLRJ:
 441:     splitCompareBranch(Branch, SystemZ::CLR);
 442:     break;
 443:   case SystemZ::CLGRJ:
 444:     splitCompareBranch(Branch, SystemZ::CLGR);
 445:     break;
 446:   case SystemZ::CLIJ:
 447:     splitCompareBranch(Branch, SystemZ::CLFI);
 448:     break;
```
- **EN**: This span continues the file's main responsibility: provides target-specific implementation details for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 449-476 / 第 449-476 行
```cpp
 449:   case SystemZ::CLGIJ:
 450:     splitCompareBranch(Branch, SystemZ::CLGFI);
 451:     break;
 452:   default:
 453:     llvm_unreachable("Unrecognized branch");
 454:   }
 455: 
 456:   Terminator.Size += Terminator.ExtraRelaxSize;
 457:   Terminator.ExtraRelaxSize = 0;
 458:   Terminator.Branch = nullptr;
 459: 
 460:   ++LongBranches;
 461: }
 462: 
 463: // Run a shortening pass and relax any branches that need to be relaxed.
 464: void SystemZLongBranch::relaxBranches() {
 465:   SmallVector<TerminatorInfo, 16>::iterator TI = Terminators.begin();
 466:   BlockPosition Position(Log2(MF->getAlignment()));
 467:   for (auto &Block : MBBs) {
 468:     skipNonTerminators(Position, Block);
 469:     for (unsigned BTI = 0, BTE = Block.NumTerminators; BTI != BTE; ++BTI) {
 470:       assert(Position.Address <= TI->Address &&
 471:              "Addresses shouldn't go forwards");
 472:       if (mustRelaxBranch(*TI, Position.Address))
 473:         relaxBranch(*TI);
 474:       skipTerminator(Position, *TI, false);
 475:       ++TI;
 476:     }
```
- **EN**: The range implements or declares functions including `SystemZLongBranch::relaxBranches`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZLongBranch::relaxBranches` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 477-494 / 第 477-494 行
```cpp
 477:   }
 478: }
 479: 
 480: bool SystemZLongBranch::runOnMachineFunction(MachineFunction &F) {
 481:   TII = static_cast<const SystemZInstrInfo *>(F.getSubtarget().getInstrInfo());
 482:   MF = &F;
 483:   uint64_t Size = initMBBInfo();
 484:   if (Size <= MaxForwardRange || !mustRelaxABranch())
 485:     return false;
 486: 
 487:   setWorstCaseAddresses();
 488:   relaxBranches();
 489:   return true;
 490: }
 491: 
 492: FunctionPass *llvm::createSystemZLongBranchPass(SystemZTargetMachine &TM) {
 493:   return new SystemZLongBranch();
 494: }
```
- **EN**: The range implements or declares functions including `SystemZLongBranch::runOnMachineFunction`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZLongBranch::runOnMachineFunction` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

## Key Concepts / 关键概念
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。
- **Target machine**: Owns data layout, pass configuration, and backend-wide policy. / 管理数据布局、Pass 配置以及整个后端策略。

## Dependencies / 依赖关系
- `SystemZ.h`
- `SystemZInstrInfo.h`
- `SystemZTargetMachine.h`
- `llvm/ADT/SmallVector.h`
- `llvm/ADT/Statistic.h`
- `llvm/ADT/StringRef.h`
- `llvm/CodeGen/MachineBasicBlock.h`
- `llvm/CodeGen/MachineFunction.h`
- `llvm/CodeGen/MachineFunctionPass.h`
- `llvm/CodeGen/MachineInstr.h`
- `llvm/CodeGen/MachineInstrBuilder.h`
- `llvm/IR/DebugLoc.h`
- `llvm/Support/ErrorHandling.h`
- `cassert`
- `cstdint`
