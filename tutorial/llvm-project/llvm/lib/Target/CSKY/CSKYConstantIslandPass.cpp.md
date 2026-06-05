# CSKYConstantIslandPass.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/CSKY/CSKYConstantIslandPass.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Provides target-specific support code, declarations, or helper routines used by the backend.
- 目的（中文）: 提供目标后端使用的专用支持代码、声明或辅助例程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===- CSKYConstantIslandPass.cpp - Emit PC Relative loads ----------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: //
  10: // Loading constants inline is expensive on CSKY and it's in general better
  11: // to place the constant nearby in code space and then it can be loaded with a
  12: // simple 16/32 bit load instruction like lrw.
  13: //
  14: // The constants can be not just numbers but addresses of functions and labels.
  15: // This can be particularly helpful in static relocation mode for embedded
  16: // non-linux targets.
  17: //
  18: //===----------------------------------------------------------------------===//
  19: 
  20: #include "CSKY.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。

### Lines 21-40

```cpp
  21: #include "CSKYConstantPoolValue.h"
  22: #include "CSKYMachineFunctionInfo.h"
  23: #include "CSKYSubtarget.h"
  24: #include "llvm/ADT/STLExtras.h"
  25: #include "llvm/ADT/SmallSet.h"
  26: #include "llvm/ADT/SmallVector.h"
  27: #include "llvm/ADT/Statistic.h"
  28: #include "llvm/ADT/StringRef.h"
  29: #include "llvm/CodeGen/MachineBasicBlock.h"
  30: #include "llvm/CodeGen/MachineConstantPool.h"
  31: #include "llvm/CodeGen/MachineFrameInfo.h"
  32: #include "llvm/CodeGen/MachineFunction.h"
  33: #include "llvm/CodeGen/MachineFunctionPass.h"
  34: #include "llvm/CodeGen/MachineInstr.h"
  35: #include "llvm/CodeGen/MachineInstrBuilder.h"
  36: #include "llvm/CodeGen/MachineOperand.h"
  37: #include "llvm/CodeGen/MachineRegisterInfo.h"
  38: #include "llvm/Config/llvm-config.h"
  39: #include "llvm/IR/Constants.h"
  40: #include "llvm/IR/DataLayout.h"
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。

### Lines 41-60

```cpp
  41: #include "llvm/IR/DebugLoc.h"
  42: #include "llvm/IR/Function.h"
  43: #include "llvm/IR/Type.h"
  44: #include "llvm/Support/CommandLine.h"
  45: #include "llvm/Support/Compiler.h"
  46: #include "llvm/Support/Debug.h"
  47: #include "llvm/Support/ErrorHandling.h"
  48: #include "llvm/Support/Format.h"
  49: #include "llvm/Support/MathExtras.h"
  50: #include "llvm/Support/raw_ostream.h"
  51: #include <cassert>
  52: #include <cstdint>
  53: #include <iterator>
  54: #include <vector>
  55: 
  56: using namespace llvm;
  57: 
  58: #define DEBUG_TYPE "CSKY-constant-islands"
  59: 
  60: STATISTIC(NumCPEs, "Number of constpool entries");
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。

### Lines 61-80

```cpp
  61: STATISTIC(NumSplit, "Number of uncond branches inserted");
  62: STATISTIC(NumCBrFixed, "Number of cond branches fixed");
  63: STATISTIC(NumUBrFixed, "Number of uncond branches fixed");
  64: 
  65: namespace {
  66: 
  67: using Iter = MachineBasicBlock::iterator;
  68: using ReverseIter = MachineBasicBlock::reverse_iterator;
  69: 
  70: /// CSKYConstantIslands - Due to limited PC-relative displacements, CSKY
  71: /// requires constant pool entries to be scattered among the instructions
  72: /// inside a function.  To do this, it completely ignores the normal LLVM
  73: /// constant pool; instead, it places constants wherever it feels like with
  74: /// special instructions.
  75: ///
  76: /// The terminology used in this pass includes:
  77: ///   Islands - Clumps of constants placed in the function.
  78: ///   Water   - Potential places where an island could be formed.
  79: ///   CPE     - A constant pool entry that has been placed somewhere, which
  80: ///             tracks a list of users.
```

- EN: This range continues the implementation of the backend component described by CSKYConstantIslandPass.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 81-100

```cpp
  81: 
  82: class CSKYConstantIslands : public MachineFunctionPass {
  83:   /// BasicBlockInfo - Information about the offset and size of a single
  84:   /// basic block.
  85:   struct BasicBlockInfo {
  86:     /// Offset - Distance from the beginning of the function to the beginning
  87:     /// of this basic block.
  88:     ///
  89:     /// Offsets are computed assuming worst case padding before an aligned
  90:     /// block. This means that subtracting basic block offsets always gives a
  91:     /// conservative estimate of the real distance which may be smaller.
  92:     ///
  93:     /// Because worst case padding is used, the computed offset of an aligned
  94:     /// block may not actually be aligned.
  95:     unsigned Offset = 0;
  96: 
  97:     /// Size - Size of the basic block in bytes.  If the block contains
  98:     /// inline assembly, this is a worst case estimate.
  99:     ///
 100:     /// The size does not include any alignment padding whether from the
```

- EN: This chunk introduces interfaces or data structures such as CSKYConstantIslands, BasicBlockInfo, which organize the target-specific behavior exposed by the file.
- 中文: 这一段引入了 CSKYConstantIslands, BasicBlockInfo 等接口或数据结构，用于组织该文件暴露的目标专用行为。

### Lines 101-120

```cpp
 101:     /// beginning of the block, or from an aligned jump table at the end.
 102:     unsigned Size = 0;
 103: 
 104:     BasicBlockInfo() = default;
 105: 
 106:     unsigned postOffset() const { return Offset + Size; }
 107:   };
 108: 
 109:   std::vector<BasicBlockInfo> BBInfo;
 110: 
 111:   /// WaterList - A sorted list of basic blocks where islands could be placed
 112:   /// (i.e. blocks that don't fall through to the following block, due
 113:   /// to a return, unreachable, or unconditional branch).
 114:   std::vector<MachineBasicBlock *> WaterList;
 115: 
 116:   /// NewWaterList - The subset of WaterList that was created since the
 117:   /// previous iteration by inserting unconditional branches.
 118:   SmallPtrSet<MachineBasicBlock *, 4> NewWaterList;
 119: 
 120:   using water_iterator = std::vector<MachineBasicBlock *>::iterator;
```

- EN: Function bodies or method definitions such as postOffset contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: postOffset 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 121-140

```cpp
 121: 
 122:   /// CPUser - One user of a constant pool, keeping the machine instruction
 123:   /// pointer, the constant pool being referenced, and the max displacement
 124:   /// allowed from the instruction to the CP.  The HighWaterMark records the
 125:   /// highest basic block where a new CPEntry can be placed.  To ensure this
 126:   /// pass terminates, the CP entries are initially placed at the end of the
 127:   /// function and then move monotonically to lower addresses.  The
 128:   /// exception to this rule is when the current CP entry for a particular
 129:   /// CPUser is out of range, but there is another CP entry for the same
 130:   /// constant value in range.  We want to use the existing in-range CP
 131:   /// entry, but if it later moves out of range, the search for new water
 132:   /// should resume where it left off.  The HighWaterMark is used to record
 133:   /// that point.
 134:   struct CPUser {
 135:     MachineInstr *MI;
 136:     MachineInstr *CPEMI;
 137:     MachineBasicBlock *HighWaterMark;
 138: 
 139:   private:
 140:     unsigned MaxDisp;
```

- EN: This chunk introduces interfaces or data structures such as CPUser, which organize the target-specific behavior exposed by the file.
- 中文: 这一段引入了 CPUser 等接口或数据结构，用于组织该文件暴露的目标专用行为。

### Lines 141-160

```cpp
 141: 
 142:   public:
 143:     bool NegOk;
 144: 
 145:     CPUser(MachineInstr *Mi, MachineInstr *Cpemi, unsigned Maxdisp, bool Neg)
 146:         : MI(Mi), CPEMI(Cpemi), MaxDisp(Maxdisp), NegOk(Neg) {
 147:       HighWaterMark = CPEMI->getParent();
 148:     }
 149: 
 150:     /// getMaxDisp - Returns the maximum displacement supported by MI.
 151:     unsigned getMaxDisp() const { return MaxDisp - 16; }
 152: 
 153:     void setMaxDisp(unsigned Val) { MaxDisp = Val; }
 154:   };
 155: 
 156:   /// CPUsers - Keep track of all of the machine instructions that use various
 157:   /// constant pools and their max displacement.
 158:   std::vector<CPUser> CPUsers;
 159: 
 160:   /// CPEntry - One per constant pool entry, keeping the machine instruction
```

- EN: Function bodies or method definitions such as MI, getMaxDisp, setMaxDisp contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: MI, getMaxDisp, setMaxDisp 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 161-180

```cpp
 161:   /// pointer, the constpool index, and the number of CPUser's which
 162:   /// reference this entry.
 163:   struct CPEntry {
 164:     MachineInstr *CPEMI;
 165:     unsigned CPI;
 166:     unsigned RefCount;
 167: 
 168:     CPEntry(MachineInstr *Cpemi, unsigned Cpi, unsigned Rc = 0)
 169:         : CPEMI(Cpemi), CPI(Cpi), RefCount(Rc) {}
 170:   };
 171: 
 172:   /// CPEntries - Keep track of all of the constant pool entry machine
 173:   /// instructions. For each original constpool index (i.e. those that
 174:   /// existed upon entry to this pass), it keeps a vector of entries.
 175:   /// Original elements are cloned as we go along; the clones are
 176:   /// put in the vector of the original element, but have distinct CPIs.
 177:   std::vector<std::vector<CPEntry>> CPEntries;
 178: 
 179:   /// ImmBranch - One per immediate branch, keeping the machine instruction
 180:   /// pointer, conditional or unconditional, the max displacement,
```

- EN: This chunk introduces interfaces or data structures such as CPEntry, which organize the target-specific behavior exposed by the file. Function bodies or method definitions such as CPEMI contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: 这一段引入了 CPEntry 等接口或数据结构，用于组织该文件暴露的目标专用行为。 CPEMI 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 181-200

```cpp
 181:   /// and (if isCond is true) the corresponding unconditional branch
 182:   /// opcode.
 183:   struct ImmBranch {
 184:     MachineInstr *MI;
 185:     unsigned MaxDisp : 31;
 186:     LLVM_PREFERRED_TYPE(bool)
 187:     unsigned IsCond : 1;
 188:     int UncondBr;
 189: 
 190:     ImmBranch(MachineInstr *Mi, unsigned Maxdisp, bool Cond, int Ubr)
 191:         : MI(Mi), MaxDisp(Maxdisp), IsCond(Cond), UncondBr(Ubr) {}
 192:   };
 193: 
 194:   /// ImmBranches - Keep track of all the immediate branch instructions.
 195:   ///
 196:   std::vector<ImmBranch> ImmBranches;
 197: 
 198:   const CSKYSubtarget *STI = nullptr;
 199:   const CSKYInstrInfo *TII;
 200:   CSKYMachineFunctionInfo *MFI;
```

- EN: This chunk introduces interfaces or data structures such as ImmBranch, which organize the target-specific behavior exposed by the file. Function bodies or method definitions such as MI contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: 这一段引入了 ImmBranch 等接口或数据结构，用于组织该文件暴露的目标专用行为。 MI 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 201-220

```cpp
 201:   MachineFunction *MF = nullptr;
 202:   MachineConstantPool *MCP = nullptr;
 203: 
 204:   unsigned PICLabelUId;
 205: 
 206:   void initPICLabelUId(unsigned UId) { PICLabelUId = UId; }
 207: 
 208:   unsigned createPICLabelUId() { return PICLabelUId++; }
 209: 
 210: public:
 211:   static char ID;
 212: 
 213:   CSKYConstantIslands() : MachineFunctionPass(ID) {}
 214: 
 215:   StringRef getPassName() const override { return "CSKY Constant Islands"; }
 216: 
 217:   bool runOnMachineFunction(MachineFunction &F) override;
 218: 
 219:   MachineFunctionProperties getRequiredProperties() const override {
 220:     return MachineFunctionProperties().setNoVRegs();
```

- EN: Function bodies or method definitions such as initPICLabelUId, createPICLabelUId contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: initPICLabelUId, createPICLabelUId 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 221-240

```cpp
 221:   }
 222: 
 223:   void doInitialPlacement(std::vector<MachineInstr *> &CPEMIs);
 224:   CPEntry *findConstPoolEntry(unsigned CPI, const MachineInstr *CPEMI);
 225:   Align getCPEAlign(const MachineInstr &CPEMI);
 226:   void initializeFunctionInfo(const std::vector<MachineInstr *> &CPEMIs);
 227:   unsigned getOffsetOf(MachineInstr *MI) const;
 228:   unsigned getUserOffset(CPUser &) const;
 229:   void dumpBBs();
 230: 
 231:   bool isOffsetInRange(unsigned UserOffset, unsigned TrialOffset, unsigned Disp,
 232:                        bool NegativeOK);
 233:   bool isOffsetInRange(unsigned UserOffset, unsigned TrialOffset,
 234:                        const CPUser &U);
 235: 
 236:   void computeBlockSize(MachineBasicBlock *MBB);
 237:   MachineBasicBlock *splitBlockBeforeInstr(MachineInstr &MI);
 238:   void updateForInsertedWaterBlock(MachineBasicBlock *NewBB);
 239:   void adjustBBOffsetsAfter(MachineBasicBlock *BB);
 240:   bool decrementCPEReferenceCount(unsigned CPI, MachineInstr *CPEMI);
```

- EN: This range continues the implementation of the backend component described by CSKYConstantIslandPass.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 241-260

```cpp
 241:   int findInRangeCPEntry(CPUser &U, unsigned UserOffset);
 242:   bool findAvailableWater(CPUser &U, unsigned UserOffset,
 243:                           water_iterator &WaterIter);
 244:   void createNewWater(unsigned CPUserIndex, unsigned UserOffset,
 245:                       MachineBasicBlock *&NewMBB);
 246:   bool handleConstantPoolUser(unsigned CPUserIndex);
 247:   void removeDeadCPEMI(MachineInstr *CPEMI);
 248:   bool removeUnusedCPEntries();
 249:   bool isCPEntryInRange(MachineInstr *MI, unsigned UserOffset,
 250:                         MachineInstr *CPEMI, unsigned Disp, bool NegOk,
 251:                         bool DoDump = false);
 252:   bool isWaterInRange(unsigned UserOffset, MachineBasicBlock *Water, CPUser &U,
 253:                       unsigned &Growth);
 254:   bool isBBInRange(MachineInstr *MI, MachineBasicBlock *BB, unsigned Disp);
 255:   bool fixupImmediateBr(ImmBranch &Br);
 256:   bool fixupConditionalBr(ImmBranch &Br);
 257:   bool fixupUnconditionalBr(ImmBranch &Br);
 258: };
 259: } // end anonymous namespace
 260: 
```

- EN: This range continues the implementation of the backend component described by CSKYConstantIslandPass.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 261-280

```cpp
 261: char CSKYConstantIslands::ID = 0;
 262: 
 263: bool CSKYConstantIslands::isOffsetInRange(unsigned UserOffset,
 264:                                           unsigned TrialOffset,
 265:                                           const CPUser &U) {
 266:   return isOffsetInRange(UserOffset, TrialOffset, U.getMaxDisp(), U.NegOk);
 267: }
 268: 
 269: #if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
 270: /// print block size and offset information - debugging
 271: LLVM_DUMP_METHOD void CSKYConstantIslands::dumpBBs() {
 272:   for (unsigned J = 0, E = BBInfo.size(); J != E; ++J) {
 273:     const BasicBlockInfo &BBI = BBInfo[J];
 274:     dbgs() << format("%08x %bb.%u\t", BBI.Offset, J)
 275:            << format(" size=%#x\n", BBInfo[J].Size);
 276:   }
 277: }
 278: #endif
 279: 
 280: bool CSKYConstantIslands::runOnMachineFunction(MachineFunction &Mf) {
```

- EN: Function bodies or method definitions such as isOffsetInRange, dumpBBs, runOnMachineFunction contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis.
- 中文: isOffsetInRange, dumpBBs, runOnMachineFunction 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。

### Lines 281-300

```cpp
 281:   MF = &Mf;
 282:   MCP = Mf.getConstantPool();
 283:   STI = &Mf.getSubtarget<CSKYSubtarget>();
 284: 
 285:   LLVM_DEBUG(dbgs() << "***** CSKYConstantIslands: "
 286:                     << MCP->getConstants().size() << " CP entries, aligned to "
 287:                     << MCP->getConstantPoolAlign().value() << " bytes *****\n");
 288: 
 289:   TII = STI->getInstrInfo();
 290:   MFI = MF->getInfo<CSKYMachineFunctionInfo>();
 291: 
 292:   // This pass invalidates liveness information when it splits basic blocks.
 293:   MF->getRegInfo().invalidateLiveness();
 294: 
 295:   // Renumber all of the machine basic blocks in the function, guaranteeing that
 296:   // the numbers agree with the position of the block in the function.
 297:   MF->RenumberBlocks();
 298: 
 299:   bool MadeChange = false;
 300: 
```

- EN: This range continues the implementation of the backend component described by CSKYConstantIslandPass.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 301-320

```cpp
 301:   // Perform the initial placement of the constant pool entries.  To start with,
 302:   // we put them all at the end of the function.
 303:   std::vector<MachineInstr *> CPEMIs;
 304:   if (!MCP->isEmpty())
 305:     doInitialPlacement(CPEMIs);
 306: 
 307:   /// The next UID to take is the first unused one.
 308:   initPICLabelUId(CPEMIs.size());
 309: 
 310:   // Do the initial scan of the function, building up information about the
 311:   // sizes of each block, the location of all the water, and finding all of the
 312:   // constant pool users.
 313:   initializeFunctionInfo(CPEMIs);
 314:   CPEMIs.clear();
 315:   LLVM_DEBUG(dumpBBs());
 316: 
 317:   /// Remove dead constant pool entries.
 318:   MadeChange |= removeUnusedCPEntries();
 319: 
 320:   // Iteratively place constant pool entries and fix up branches until there
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 321-340

```cpp
 321:   // is no change.
 322:   unsigned NoCPIters = 0, NoBRIters = 0;
 323:   (void)NoBRIters;
 324:   while (true) {
 325:     LLVM_DEBUG(dbgs() << "Beginning CP iteration #" << NoCPIters << '\n');
 326:     bool CPChange = false;
 327:     for (unsigned I = 0, E = CPUsers.size(); I != E; ++I)
 328:       CPChange |= handleConstantPoolUser(I);
 329:     if (CPChange && ++NoCPIters > 30)
 330:       report_fatal_error("Constant Island pass failed to converge!");
 331:     LLVM_DEBUG(dumpBBs());
 332: 
 333:     // Clear NewWaterList now.  If we split a block for branches, it should
 334:     // appear as "new water" for the next iteration of constant pool placement.
 335:     NewWaterList.clear();
 336: 
 337:     LLVM_DEBUG(dbgs() << "Beginning BR iteration #" << NoBRIters << '\n');
 338:     bool BRChange = false;
 339:     for (unsigned I = 0, E = ImmBranches.size(); I != E; ++I)
 340:       BRChange |= fixupImmediateBr(ImmBranches[I]);
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 341-360

```cpp
 341:     if (BRChange && ++NoBRIters > 30)
 342:       report_fatal_error("Branch Fix Up pass failed to converge!");
 343:     LLVM_DEBUG(dumpBBs());
 344:     if (!CPChange && !BRChange)
 345:       break;
 346:     MadeChange = true;
 347:   }
 348: 
 349:   LLVM_DEBUG(dbgs() << '\n'; dumpBBs());
 350: 
 351:   BBInfo.clear();
 352:   WaterList.clear();
 353:   CPUsers.clear();
 354:   CPEntries.clear();
 355:   ImmBranches.clear();
 356:   return MadeChange;
 357: }
 358: 
 359: /// doInitialPlacement - Perform the initial placement of the constant pool
 360: /// entries.  To start with, we put them all at the end of the function.
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 361-380

```cpp
 361: void CSKYConstantIslands::doInitialPlacement(
 362:     std::vector<MachineInstr *> &CPEMIs) {
 363:   // Create the basic block to hold the CPE's.
 364:   MachineBasicBlock *BB = MF->CreateMachineBasicBlock();
 365:   MF->push_back(BB);
 366: 
 367:   // MachineConstantPool measures alignment in bytes. We measure in log2(bytes).
 368:   const Align MaxAlign = MCP->getConstantPoolAlign();
 369: 
 370:   // Mark the basic block as required by the const-pool.
 371:   BB->setAlignment(Align(2));
 372: 
 373:   // The function needs to be as aligned as the basic blocks. The linker may
 374:   // move functions around based on their alignment.
 375:   MF->ensureAlignment(BB->getAlignment());
 376: 
 377:   // Order the entries in BB by descending alignment.  That ensures correct
 378:   // alignment of all entries as long as BB is sufficiently aligned.  Keep
 379:   // track of the insertion point for each alignment.  We are going to bucket
 380:   // sort the entries as they are created.
```

- EN: Function bodies or method definitions such as doInitialPlacement contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: doInitialPlacement 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 381-400

```cpp
 381:   SmallVector<MachineBasicBlock::iterator, 8> InsPoint(Log2(MaxAlign) + 1,
 382:                                                        BB->end());
 383: 
 384:   // Add all of the constants from the constant pool to the end block, use an
 385:   // identity mapping of CPI's to CPE's.
 386:   const std::vector<MachineConstantPoolEntry> &CPs = MCP->getConstants();
 387: 
 388:   const DataLayout &TD = MF->getDataLayout();
 389:   for (unsigned I = 0, E = CPs.size(); I != E; ++I) {
 390:     unsigned Size = CPs[I].getSizeInBytes(TD);
 391:     assert(Size >= 4 && "Too small constant pool entry");
 392:     Align Alignment = CPs[I].getAlign();
 393:     // Verify that all constant pool entries are a multiple of their alignment.
 394:     // If not, we would have to pad them out so that instructions stay aligned.
 395:     assert(isAligned(Alignment, Size) && "CP Entry not multiple of 4 bytes!");
 396: 
 397:     // Insert CONSTPOOL_ENTRY before entries with a smaller alignment.
 398:     unsigned LogAlign = Log2(Alignment);
 399:     MachineBasicBlock::iterator InsAt = InsPoint[LogAlign];
 400: 
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。

### Lines 401-420

```cpp
 401:     MachineInstr *CPEMI =
 402:         BuildMI(*BB, InsAt, DebugLoc(), TII->get(CSKY::CONSTPOOL_ENTRY))
 403:             .addImm(I)
 404:             .addConstantPoolIndex(I)
 405:             .addImm(Size);
 406: 
 407:     CPEMIs.push_back(CPEMI);
 408: 
 409:     // Ensure that future entries with higher alignment get inserted before
 410:     // CPEMI. This is bucket sort with iterators.
 411:     for (unsigned A = LogAlign + 1; A <= Log2(MaxAlign); ++A)
 412:       if (InsPoint[A] == InsAt)
 413:         InsPoint[A] = CPEMI;
 414:     // Add a new CPEntry, but no corresponding CPUser yet.
 415:     CPEntries.emplace_back(1, CPEntry(CPEMI, I));
 416:     ++NumCPEs;
 417:     LLVM_DEBUG(dbgs() << "Moved CPI#" << I << " to end of function, size = "
 418:                       << Size << ", align = " << Alignment.value() << '\n');
 419:   }
 420:   LLVM_DEBUG(BB->dump());
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 421-440

```cpp
 421: }
 422: 
 423: /// BBHasFallthrough - Return true if the specified basic block can fallthrough
 424: /// into the block immediately after it.
 425: static bool bbHasFallthrough(MachineBasicBlock *MBB) {
 426:   // Get the next machine basic block in the function.
 427:   MachineFunction::iterator MBBI = MBB->getIterator();
 428:   // Can't fall off end of function.
 429:   if (std::next(MBBI) == MBB->getParent()->end())
 430:     return false;
 431: 
 432:   MachineBasicBlock *NextBB = &*std::next(MBBI);
 433:   for (MachineBasicBlock::succ_iterator I = MBB->succ_begin(),
 434:                                         E = MBB->succ_end();
 435:        I != E; ++I)
 436:     if (*I == NextBB)
 437:       return true;
 438: 
 439:   return false;
 440: }
```

- EN: Function bodies or method definitions such as bbHasFallthrough contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: bbHasFallthrough 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 441-460

```cpp
 441: 
 442: /// findConstPoolEntry - Given the constpool index and CONSTPOOL_ENTRY MI,
 443: /// look up the corresponding CPEntry.
 444: CSKYConstantIslands::CPEntry *
 445: CSKYConstantIslands::findConstPoolEntry(unsigned CPI,
 446:                                         const MachineInstr *CPEMI) {
 447:   std::vector<CPEntry> &CPEs = CPEntries[CPI];
 448:   // Number of entries per constpool index should be small, just do a
 449:   // linear search.
 450:   for (unsigned I = 0, E = CPEs.size(); I != E; ++I) {
 451:     if (CPEs[I].CPEMI == CPEMI)
 452:       return &CPEs[I];
 453:   }
 454:   return nullptr;
 455: }
 456: 
 457: /// getCPEAlign - Returns the required alignment of the constant pool entry
 458: /// represented by CPEMI.  Alignment is measured in log2(bytes) units.
 459: Align CSKYConstantIslands::getCPEAlign(const MachineInstr &CPEMI) {
 460:   assert(CPEMI.getOpcode() == CSKY::CONSTPOOL_ENTRY);
```

- EN: Function bodies or method definitions such as findConstPoolEntry, getCPEAlign contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: findConstPoolEntry, getCPEAlign 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 461-480

```cpp
 461: 
 462:   unsigned CPI = CPEMI.getOperand(1).getIndex();
 463:   assert(CPI < MCP->getConstants().size() && "Invalid constant pool index.");
 464:   return MCP->getConstants()[CPI].getAlign();
 465: }
 466: 
 467: /// initializeFunctionInfo - Do the initial scan of the function, building up
 468: /// information about the sizes of each block, the location of all the water,
 469: /// and finding all of the constant pool users.
 470: void CSKYConstantIslands::initializeFunctionInfo(
 471:     const std::vector<MachineInstr *> &CPEMIs) {
 472:   BBInfo.clear();
 473:   BBInfo.resize(MF->getNumBlockIDs());
 474: 
 475:   // First thing, compute the size of all basic blocks, and see if the function
 476:   // has any inline assembly in it. If so, we have to be conservative about
 477:   // alignment assumptions, as we don't know for sure the size of any
 478:   // instructions in the inline assembly.
 479:   for (MachineFunction::iterator I = MF->begin(), E = MF->end(); I != E; ++I)
 480:     computeBlockSize(&*I);
```

- EN: Function bodies or method definitions such as initializeFunctionInfo contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis.
- 中文: initializeFunctionInfo 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。

### Lines 481-500

```cpp
 481: 
 482:   // Compute block offsets.
 483:   adjustBBOffsetsAfter(&MF->front());
 484: 
 485:   // Now go back through the instructions and build up our data structures.
 486:   for (MachineBasicBlock &MBB : *MF) {
 487:     // If this block doesn't fall through into the next MBB, then this is
 488:     // 'water' that a constant pool island could be placed.
 489:     if (!bbHasFallthrough(&MBB))
 490:       WaterList.push_back(&MBB);
 491:     for (MachineInstr &MI : MBB) {
 492:       if (MI.isDebugInstr())
 493:         continue;
 494: 
 495:       int Opc = MI.getOpcode();
 496:       if (MI.isBranch() && !MI.isIndirectBranch()) {
 497:         bool IsCond = MI.isConditionalBranch();
 498:         unsigned Bits = 0;
 499:         unsigned Scale = 1;
 500:         int UOpc = CSKY::BR32;
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 501-520

```cpp
 501: 
 502:         switch (MI.getOpcode()) {
 503:         case CSKY::BR16:
 504:         case CSKY::BF16:
 505:         case CSKY::BT16:
 506:           Bits = 10;
 507:           Scale = 2;
 508:           break;
 509:         default:
 510:           Bits = 16;
 511:           Scale = 2;
 512:           break;
 513:         }
 514: 
 515:         // Record this immediate branch.
 516:         unsigned MaxOffs = ((1 << (Bits - 1)) - 1) * Scale;
 517:         ImmBranches.push_back(ImmBranch(&MI, MaxOffs, IsCond, UOpc));
 518:       }
 519: 
 520:       if (Opc == CSKY::CONSTPOOL_ENTRY)
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 521-540

```cpp
 521:         continue;
 522: 
 523:       // Scan the instructions for constant pool operands.
 524:       for (unsigned Op = 0, E = MI.getNumOperands(); Op != E; ++Op)
 525:         if (MI.getOperand(Op).isCPI()) {
 526:           // We found one.  The addressing mode tells us the max displacement
 527:           // from the PC that this instruction permits.
 528: 
 529:           // Basic size info comes from the TSFlags field.
 530:           unsigned Bits = 0;
 531:           unsigned Scale = 1;
 532:           bool NegOk = false;
 533: 
 534:           switch (Opc) {
 535:           default:
 536:             llvm_unreachable("Unknown addressing mode for CP reference!");
 537:           case CSKY::MOVIH32:
 538:           case CSKY::ORI32:
 539:             continue;
 540:           case CSKY::PseudoTLSLA32:
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 541-560

```cpp
 541:           case CSKY::JSRI32:
 542:           case CSKY::JMPI32:
 543:           case CSKY::LRW32:
 544:           case CSKY::LRW32_Gen:
 545:             Bits = 16;
 546:             Scale = 4;
 547:             break;
 548:           case CSKY::f2FLRW_S:
 549:           case CSKY::f2FLRW_D:
 550:             Bits = 8;
 551:             Scale = 4;
 552:             break;
 553:           case CSKY::GRS32:
 554:             Bits = 17;
 555:             Scale = 2;
 556:             NegOk = true;
 557:             break;
 558:           }
 559:           // Remember that this is a user of a CP entry.
 560:           unsigned CPI = MI.getOperand(Op).getIndex();
```

- EN: This range continues the implementation of the backend component described by CSKYConstantIslandPass.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 561-580

```cpp
 561:           MachineInstr *CPEMI = CPEMIs[CPI];
 562:           unsigned MaxOffs = ((1 << Bits) - 1) * Scale;
 563:           CPUsers.push_back(CPUser(&MI, CPEMI, MaxOffs, NegOk));
 564: 
 565:           // Increment corresponding CPEntry reference count.
 566:           CPEntry *CPE = findConstPoolEntry(CPI, CPEMI);
 567:           assert(CPE && "Cannot find a corresponding CPEntry!");
 568:           CPE->RefCount++;
 569:         }
 570:     }
 571:   }
 572: }
 573: 
 574: /// computeBlockSize - Compute the size and some alignment information for MBB.
 575: /// This function updates BBInfo directly.
 576: void CSKYConstantIslands::computeBlockSize(MachineBasicBlock *MBB) {
 577:   BasicBlockInfo &BBI = BBInfo[MBB->getNumber()];
 578:   BBI.Size = 0;
 579: 
 580:   for (const MachineInstr &MI : *MBB)
```

- EN: Function bodies or method definitions such as computeBlockSize contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis.
- 中文: computeBlockSize 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。

### Lines 581-600

```cpp
 581:     BBI.Size += TII->getInstSizeInBytes(MI);
 582: }
 583: 
 584: /// getOffsetOf - Return the current offset of the specified machine instruction
 585: /// from the start of the function.  This offset changes as stuff is moved
 586: /// around inside the function.
 587: unsigned CSKYConstantIslands::getOffsetOf(MachineInstr *MI) const {
 588:   MachineBasicBlock *MBB = MI->getParent();
 589: 
 590:   // The offset is composed of two things: the sum of the sizes of all MBB's
 591:   // before this instruction's block, and the offset from the start of the block
 592:   // it is in.
 593:   unsigned Offset = BBInfo[MBB->getNumber()].Offset;
 594: 
 595:   // Sum instructions before MI in MBB.
 596:   for (MachineBasicBlock::iterator I = MBB->begin(); &*I != MI; ++I) {
 597:     assert(I != MBB->end() && "Didn't find MI in its own basic block?");
 598:     Offset += TII->getInstSizeInBytes(*I);
 599:   }
 600:   return Offset;
```

- EN: Function bodies or method definitions such as getOffsetOf contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis.
- 中文: getOffsetOf 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。

### Lines 601-620

```cpp
 601: }
 602: 
 603: /// CompareMBBNumbers - Little predicate function to sort the WaterList by MBB
 604: /// ID.
 605: static bool compareMbbNumbers(const MachineBasicBlock *LHS,
 606:                               const MachineBasicBlock *RHS) {
 607:   return LHS->getNumber() < RHS->getNumber();
 608: }
 609: 
 610: /// updateForInsertedWaterBlock - When a block is newly inserted into the
 611: /// machine function, it upsets all of the block numbers.  Renumber the blocks
 612: /// and update the arrays that parallel this numbering.
 613: void CSKYConstantIslands::updateForInsertedWaterBlock(
 614:     MachineBasicBlock *NewBB) {
 615:   // Renumber the MBB's to keep them consecutive.
 616:   NewBB->getParent()->RenumberBlocks(NewBB);
 617: 
 618:   // Insert an entry into BBInfo to align it properly with the (newly
 619:   // renumbered) block numbers.
 620:   BBInfo.insert(BBInfo.begin() + NewBB->getNumber(), BasicBlockInfo());
```

- EN: Function bodies or method definitions such as compareMbbNumbers, updateForInsertedWaterBlock contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: compareMbbNumbers, updateForInsertedWaterBlock 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 621-640

```cpp
 621: 
 622:   // Next, update WaterList.  Specifically, we need to add NewMBB as having
 623:   // available water after it.
 624:   water_iterator IP = llvm::lower_bound(WaterList, NewBB, compareMbbNumbers);
 625:   WaterList.insert(IP, NewBB);
 626: }
 627: 
 628: unsigned CSKYConstantIslands::getUserOffset(CPUser &U) const {
 629:   unsigned UserOffset = getOffsetOf(U.MI);
 630: 
 631:   UserOffset &= ~3u;
 632: 
 633:   return UserOffset;
 634: }
 635: 
 636: /// Split the basic block containing MI into two blocks, which are joined by
 637: /// an unconditional branch.  Update data structures and renumber blocks to
 638: /// account for this change and returns the newly created block.
 639: MachineBasicBlock *
 640: CSKYConstantIslands::splitBlockBeforeInstr(MachineInstr &MI) {
```

- EN: Function bodies or method definitions such as getUserOffset, splitBlockBeforeInstr contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: getUserOffset, splitBlockBeforeInstr 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 641-660

```cpp
 641:   MachineBasicBlock *OrigBB = MI.getParent();
 642: 
 643:   // Create a new MBB for the code after the OrigBB.
 644:   MachineBasicBlock *NewBB =
 645:       MF->CreateMachineBasicBlock(OrigBB->getBasicBlock());
 646:   MachineFunction::iterator MBBI = ++OrigBB->getIterator();
 647:   MF->insert(MBBI, NewBB);
 648: 
 649:   // Splice the instructions starting with MI over to NewBB.
 650:   NewBB->splice(NewBB->end(), OrigBB, MI, OrigBB->end());
 651: 
 652:   // Add an unconditional branch from OrigBB to NewBB.
 653:   // Note the new unconditional branch is not being recorded.
 654:   // There doesn't seem to be meaningful DebugInfo available; this doesn't
 655:   // correspond to anything in the source.
 656: 
 657:   // TODO: Add support for 16bit instr.
 658:   BuildMI(OrigBB, DebugLoc(), TII->get(CSKY::BR32)).addMBB(NewBB);
 659:   ++NumSplit;
 660: 
```

- EN: This range continues the implementation of the backend component described by CSKYConstantIslandPass.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 661-680

```cpp
 661:   // Update the CFG.  All succs of OrigBB are now succs of NewBB.
 662:   NewBB->transferSuccessors(OrigBB);
 663: 
 664:   // OrigBB branches to NewBB.
 665:   OrigBB->addSuccessor(NewBB);
 666: 
 667:   // Update internal data structures to account for the newly inserted MBB.
 668:   // This is almost the same as updateForInsertedWaterBlock, except that
 669:   // the Water goes after OrigBB, not NewBB.
 670:   MF->RenumberBlocks(NewBB);
 671: 
 672:   // Insert an entry into BBInfo to align it properly with the (newly
 673:   // renumbered) block numbers.
 674:   BBInfo.insert(BBInfo.begin() + NewBB->getNumber(), BasicBlockInfo());
 675: 
 676:   // Next, update WaterList.  Specifically, we need to add OrigMBB as having
 677:   // available water after it (but not if it's already there, which happens
 678:   // when splitting before a conditional branch that is followed by an
 679:   // unconditional branch - in that case we want to insert NewBB).
 680:   water_iterator IP = llvm::lower_bound(WaterList, OrigBB, compareMbbNumbers);
```

- EN: This range continues the implementation of the backend component described by CSKYConstantIslandPass.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 681-700

```cpp
 681:   MachineBasicBlock *WaterBB = *IP;
 682:   if (WaterBB == OrigBB)
 683:     WaterList.insert(std::next(IP), NewBB);
 684:   else
 685:     WaterList.insert(IP, OrigBB);
 686:   NewWaterList.insert(OrigBB);
 687: 
 688:   // Figure out how large the OrigBB is.  As the first half of the original
 689:   // block, it cannot contain a tablejump.  The size includes
 690:   // the new jump we added.  (It should be possible to do this without
 691:   // recounting everything, but it's very confusing, and this is rarely
 692:   // executed.)
 693:   computeBlockSize(OrigBB);
 694: 
 695:   // Figure out how large the NewMBB is.  As the second half of the original
 696:   // block, it may contain a tablejump.
 697:   computeBlockSize(NewBB);
 698: 
 699:   // All BBOffsets following these blocks must be modified.
 700:   adjustBBOffsetsAfter(OrigBB);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 701-720

```cpp
 701: 
 702:   return NewBB;
 703: }
 704: 
 705: /// isOffsetInRange - Checks whether UserOffset (the location of a constant pool
 706: /// reference) is within MaxDisp of TrialOffset (a proposed location of a
 707: /// constant pool entry).
 708: bool CSKYConstantIslands::isOffsetInRange(unsigned UserOffset,
 709:                                           unsigned TrialOffset,
 710:                                           unsigned MaxDisp, bool NegativeOK) {
 711:   if (UserOffset <= TrialOffset) {
 712:     // User before the Trial.
 713:     if (TrialOffset - UserOffset <= MaxDisp)
 714:       return true;
 715:   } else if (NegativeOK) {
 716:     if (UserOffset - TrialOffset <= MaxDisp)
 717:       return true;
 718:   }
 719:   return false;
 720: }
```

- EN: Function bodies or method definitions such as isOffsetInRange contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: isOffsetInRange 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 721-740

```cpp
 721: 
 722: /// isWaterInRange - Returns true if a CPE placed after the specified
 723: /// Water (a basic block) will be in range for the specific MI.
 724: ///
 725: /// Compute how much the function will grow by inserting a CPE after Water.
 726: bool CSKYConstantIslands::isWaterInRange(unsigned UserOffset,
 727:                                          MachineBasicBlock *Water, CPUser &U,
 728:                                          unsigned &Growth) {
 729:   unsigned CPEOffset = BBInfo[Water->getNumber()].postOffset();
 730:   unsigned NextBlockOffset;
 731:   Align NextBlockAlignment;
 732:   MachineFunction::const_iterator NextBlock = ++Water->getIterator();
 733:   if (NextBlock == MF->end()) {
 734:     NextBlockOffset = BBInfo[Water->getNumber()].postOffset();
 735:     NextBlockAlignment = Align(4);
 736:   } else {
 737:     NextBlockOffset = BBInfo[NextBlock->getNumber()].Offset;
 738:     NextBlockAlignment = NextBlock->getAlignment();
 739:   }
 740:   unsigned Size = U.CPEMI->getOperand(2).getImm();
```

- EN: Function bodies or method definitions such as isWaterInRange contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: isWaterInRange 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 741-760

```cpp
 741:   unsigned CPEEnd = CPEOffset + Size;
 742: 
 743:   // The CPE may be able to hide in the alignment padding before the next
 744:   // block. It may also cause more padding to be required if it is more aligned
 745:   // that the next block.
 746:   if (CPEEnd > NextBlockOffset) {
 747:     Growth = CPEEnd - NextBlockOffset;
 748:     // Compute the padding that would go at the end of the CPE to align the next
 749:     // block.
 750:     Growth += offsetToAlignment(CPEEnd, NextBlockAlignment);
 751: 
 752:     // If the CPE is to be inserted before the instruction, that will raise
 753:     // the offset of the instruction. Also account for unknown alignment padding
 754:     // in blocks between CPE and the user.
 755:     if (CPEOffset < UserOffset)
 756:       UserOffset += Growth;
 757:   } else
 758:     // CPE fits in existing padding.
 759:     Growth = 0;
 760: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 761-780

```cpp
 761:   return isOffsetInRange(UserOffset, CPEOffset, U);
 762: }
 763: 
 764: /// isCPEntryInRange - Returns true if the distance between specific MI and
 765: /// specific ConstPool entry instruction can fit in MI's displacement field.
 766: bool CSKYConstantIslands::isCPEntryInRange(MachineInstr *MI,
 767:                                            unsigned UserOffset,
 768:                                            MachineInstr *CPEMI,
 769:                                            unsigned MaxDisp, bool NegOk,
 770:                                            bool DoDump) {
 771:   unsigned CPEOffset = getOffsetOf(CPEMI);
 772: 
 773:   if (DoDump) {
 774:     LLVM_DEBUG({
 775:       unsigned Block = MI->getParent()->getNumber();
 776:       const BasicBlockInfo &BBI = BBInfo[Block];
 777:       dbgs() << "User of CPE#" << CPEMI->getOperand(0).getImm()
 778:              << " max delta=" << MaxDisp
 779:              << format(" insn address=%#x", UserOffset) << " in "
 780:              << printMBBReference(*MI->getParent()) << ": "
```

- EN: Function bodies or method definitions such as isCPEntryInRange contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: isCPEntryInRange 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 781-800

```cpp
 781:              << format("%#x-%x\t", BBI.Offset, BBI.postOffset()) << *MI
 782:              << format("CPE address=%#x offset=%+d: ", CPEOffset,
 783:                        int(CPEOffset - UserOffset));
 784:     });
 785:   }
 786: 
 787:   return isOffsetInRange(UserOffset, CPEOffset, MaxDisp, NegOk);
 788: }
 789: 
 790: #ifndef NDEBUG
 791: /// BBIsJumpedOver - Return true of the specified basic block's only predecessor
 792: /// unconditionally branches to its only successor.
 793: static bool bbIsJumpedOver(MachineBasicBlock *MBB) {
 794:   if (MBB->pred_size() != 1 || MBB->succ_size() != 1)
 795:     return false;
 796:   MachineBasicBlock *Succ = *MBB->succ_begin();
 797:   MachineBasicBlock *Pred = *MBB->pred_begin();
 798:   MachineInstr *PredMI = &Pred->back();
 799:   if (PredMI->getOpcode() == CSKY::BR32 /*TODO: change to 16bit instr. */)
 800:     return PredMI->getOperand(0).getMBB() == Succ;
```

- EN: Function bodies or method definitions such as bbIsJumpedOver contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: bbIsJumpedOver 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 801-820

```cpp
 801:   return false;
 802: }
 803: #endif
 804: 
 805: void CSKYConstantIslands::adjustBBOffsetsAfter(MachineBasicBlock *BB) {
 806:   unsigned BBNum = BB->getNumber();
 807:   for (unsigned I = BBNum + 1, E = MF->getNumBlockIDs(); I < E; ++I) {
 808:     // Get the offset and known bits at the end of the layout predecessor.
 809:     // Include the alignment of the current block.
 810:     unsigned Offset = BBInfo[I - 1].Offset + BBInfo[I - 1].Size;
 811:     BBInfo[I].Offset = Offset;
 812:   }
 813: }
 814: 
 815: /// decrementCPEReferenceCount - find the constant pool entry with index CPI
 816: /// and instruction CPEMI, and decrement its refcount.  If the refcount
 817: /// becomes 0 remove the entry and instruction.  Returns true if we removed
 818: /// the entry, false if we didn't.
 819: bool CSKYConstantIslands::decrementCPEReferenceCount(unsigned CPI,
 820:                                                      MachineInstr *CPEMI) {
```

- EN: Function bodies or method definitions such as adjustBBOffsetsAfter, decrementCPEReferenceCount contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis.
- 中文: adjustBBOffsetsAfter, decrementCPEReferenceCount 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。

### Lines 821-840

```cpp
 821:   // Find the old entry. Eliminate it if it is no longer used.
 822:   CPEntry *CPE = findConstPoolEntry(CPI, CPEMI);
 823:   assert(CPE && "Unexpected!");
 824:   if (--CPE->RefCount == 0) {
 825:     removeDeadCPEMI(CPEMI);
 826:     CPE->CPEMI = nullptr;
 827:     --NumCPEs;
 828:     return true;
 829:   }
 830:   return false;
 831: }
 832: 
 833: /// LookForCPEntryInRange - see if the currently referenced CPE is in range;
 834: /// if not, see if an in-range clone of the CPE is in range, and if so,
 835: /// change the data structures so the user references the clone.  Returns:
 836: /// 0 = no existing entry found
 837: /// 1 = entry found, and there were no code insertions or deletions
 838: /// 2 = entry found, and there were code insertions or deletions
 839: int CSKYConstantIslands::findInRangeCPEntry(CPUser &U, unsigned UserOffset) {
 840:   MachineInstr *UserMI = U.MI;
```

- EN: Function bodies or method definitions such as findInRangeCPEntry contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: findInRangeCPEntry 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 841-860

```cpp
 841:   MachineInstr *CPEMI = U.CPEMI;
 842: 
 843:   // Check to see if the CPE is already in-range.
 844:   if (isCPEntryInRange(UserMI, UserOffset, CPEMI, U.getMaxDisp(), U.NegOk,
 845:                        true)) {
 846:     LLVM_DEBUG(dbgs() << "In range\n");
 847:     return 1;
 848:   }
 849: 
 850:   // No.  Look for previously created clones of the CPE that are in range.
 851:   unsigned CPI = CPEMI->getOperand(1).getIndex();
 852:   std::vector<CPEntry> &CPEs = CPEntries[CPI];
 853:   for (unsigned I = 0, E = CPEs.size(); I != E; ++I) {
 854:     // We already tried this one
 855:     if (CPEs[I].CPEMI == CPEMI)
 856:       continue;
 857:     // Removing CPEs can leave empty entries, skip
 858:     if (CPEs[I].CPEMI == nullptr)
 859:       continue;
 860:     if (isCPEntryInRange(UserMI, UserOffset, CPEs[I].CPEMI, U.getMaxDisp(),
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 861-880

```cpp
 861:                          U.NegOk)) {
 862:       LLVM_DEBUG(dbgs() << "Replacing CPE#" << CPI << " with CPE#"
 863:                         << CPEs[I].CPI << "\n");
 864:       // Point the CPUser node to the replacement
 865:       U.CPEMI = CPEs[I].CPEMI;
 866:       // Change the CPI in the instruction operand to refer to the clone.
 867:       for (unsigned J = 0, E = UserMI->getNumOperands(); J != E; ++J)
 868:         if (UserMI->getOperand(J).isCPI()) {
 869:           UserMI->getOperand(J).setIndex(CPEs[I].CPI);
 870:           break;
 871:         }
 872:       // Adjust the refcount of the clone...
 873:       CPEs[I].RefCount++;
 874:       // ...and the original.  If we didn't remove the old entry, none of the
 875:       // addresses changed, so we don't need another pass.
 876:       return decrementCPEReferenceCount(CPI, CPEMI) ? 2 : 1;
 877:     }
 878:   }
 879:   return 0;
 880: }
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 881-900

```cpp
 881: 
 882: /// getUnconditionalBrDisp - Returns the maximum displacement that can fit in
 883: /// the specific unconditional branch instruction.
 884: static inline unsigned getUnconditionalBrDisp(int Opc) {
 885:   unsigned Bits, Scale;
 886: 
 887:   switch (Opc) {
 888:   case CSKY::BR16:
 889:     Bits = 10;
 890:     Scale = 2;
 891:     break;
 892:   case CSKY::BR32:
 893:     Bits = 16;
 894:     Scale = 2;
 895:     break;
 896:   default:
 897:     llvm_unreachable("");
 898:   }
 899: 
 900:   unsigned MaxOffs = ((1 << (Bits - 1)) - 1) * Scale;
```

- EN: Function bodies or method definitions such as getUnconditionalBrDisp contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition.
- 中文: getUnconditionalBrDisp 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。

### Lines 901-920

```cpp
 901:   return MaxOffs;
 902: }
 903: 
 904: /// findAvailableWater - Look for an existing entry in the WaterList in which
 905: /// we can place the CPE referenced from U so it's within range of U's MI.
 906: /// Returns true if found, false if not.  If it returns true, WaterIter
 907: /// is set to the WaterList entry.
 908: /// To ensure that this pass
 909: /// terminates, the CPE location for a particular CPUser is only allowed to
 910: /// move to a lower address, so search backward from the end of the list and
 911: /// prefer the first water that is in range.
 912: bool CSKYConstantIslands::findAvailableWater(CPUser &U, unsigned UserOffset,
 913:                                              water_iterator &WaterIter) {
 914:   if (WaterList.empty())
 915:     return false;
 916: 
 917:   unsigned BestGrowth = ~0u;
 918:   for (water_iterator IP = std::prev(WaterList.end()), B = WaterList.begin();;
 919:        --IP) {
 920:     MachineBasicBlock *WaterBB = *IP;
```

- EN: Function bodies or method definitions such as findAvailableWater contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: findAvailableWater 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 921-940

```cpp
 921:     // Check if water is in range and is either at a lower address than the
 922:     // current "high water mark" or a new water block that was created since
 923:     // the previous iteration by inserting an unconditional branch.  In the
 924:     // latter case, we want to allow resetting the high water mark back to
 925:     // this new water since we haven't seen it before.  Inserting branches
 926:     // should be relatively uncommon and when it does happen, we want to be
 927:     // sure to take advantage of it for all the CPEs near that block, so that
 928:     // we don't insert more branches than necessary.
 929:     unsigned Growth;
 930:     if (isWaterInRange(UserOffset, WaterBB, U, Growth) &&
 931:         (WaterBB->getNumber() < U.HighWaterMark->getNumber() ||
 932:          NewWaterList.count(WaterBB)) &&
 933:         Growth < BestGrowth) {
 934:       // This is the least amount of required padding seen so far.
 935:       BestGrowth = Growth;
 936:       WaterIter = IP;
 937:       LLVM_DEBUG(dbgs() << "Found water after " << printMBBReference(*WaterBB)
 938:                         << " Growth=" << Growth << '\n');
 939: 
 940:       // Keep looking unless it is perfect.
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 941-960

```cpp
 941:       if (BestGrowth == 0)
 942:         return true;
 943:     }
 944:     if (IP == B)
 945:       break;
 946:   }
 947:   return BestGrowth != ~0u;
 948: }
 949: 
 950: /// createNewWater - No existing WaterList entry will work for
 951: /// CPUsers[CPUserIndex], so create a place to put the CPE.  The end of the
 952: /// block is used if in range, and the conditional branch munged so control
 953: /// flow is correct.  Otherwise the block is split to create a hole with an
 954: /// unconditional branch around it.  In either case NewMBB is set to a
 955: /// block following which the new island can be inserted (the WaterList
 956: /// is not adjusted).
 957: void CSKYConstantIslands::createNewWater(unsigned CPUserIndex,
 958:                                          unsigned UserOffset,
 959:                                          MachineBasicBlock *&NewMBB) {
 960:   CPUser &U = CPUsers[CPUserIndex];
```

- EN: Function bodies or method definitions such as createNewWater contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: createNewWater 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 961-980

```cpp
 961:   MachineInstr *UserMI = U.MI;
 962:   MachineInstr *CPEMI = U.CPEMI;
 963:   MachineBasicBlock *UserMBB = UserMI->getParent();
 964:   const BasicBlockInfo &UserBBI = BBInfo[UserMBB->getNumber()];
 965: 
 966:   // If the block does not end in an unconditional branch already, and if the
 967:   // end of the block is within range, make new water there.
 968:   if (bbHasFallthrough(UserMBB)) {
 969:     // Size of branch to insert.
 970:     unsigned Delta = 4;
 971:     // Compute the offset where the CPE will begin.
 972:     unsigned CPEOffset = UserBBI.postOffset() + Delta;
 973: 
 974:     if (isOffsetInRange(UserOffset, CPEOffset, U)) {
 975:       LLVM_DEBUG(dbgs() << "Split at end of " << printMBBReference(*UserMBB)
 976:                         << format(", expected CPE offset %#x\n", CPEOffset));
 977:       NewMBB = &*++UserMBB->getIterator();
 978:       // Add an unconditional branch from UserMBB to fallthrough block.  Record
 979:       // it for branch lengthening; this new branch will not get out of range,
 980:       // but if the preceding conditional branch is out of range, the targets
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 981-1000

```cpp
 981:       // will be exchanged, and the altered branch may be out of range, so the
 982:       // machinery has to know about it.
 983: 
 984:       // TODO: Add support for 16bit instr.
 985:       int UncondBr = CSKY::BR32;
 986:       auto *NewMI = BuildMI(UserMBB, DebugLoc(), TII->get(UncondBr))
 987:                         .addMBB(NewMBB)
 988:                         .getInstr();
 989:       unsigned MaxDisp = getUnconditionalBrDisp(UncondBr);
 990:       ImmBranches.push_back(
 991:           ImmBranch(&UserMBB->back(), MaxDisp, false, UncondBr));
 992:       BBInfo[UserMBB->getNumber()].Size += TII->getInstSizeInBytes(*NewMI);
 993:       adjustBBOffsetsAfter(UserMBB);
 994:       return;
 995:     }
 996:   }
 997: 
 998:   // What a big block.  Find a place within the block to split it.
 999: 
1000:   // Try to split the block so it's fully aligned.  Compute the latest split
```

- EN: This range continues the implementation of the backend component described by CSKYConstantIslandPass.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 1001-1020

```cpp
1001:   // point where we can add a 4-byte branch instruction, and then align to
1002:   // Align which is the largest possible alignment in the function.
1003:   const Align Align = MF->getAlignment();
1004:   unsigned BaseInsertOffset = UserOffset + U.getMaxDisp();
1005:   LLVM_DEBUG(dbgs() << format("Split in middle of big block before %#x",
1006:                               BaseInsertOffset));
1007: 
1008:   // The 4 in the following is for the unconditional branch we'll be inserting
1009:   // Alignment of the island is handled
1010:   // inside isOffsetInRange.
1011:   BaseInsertOffset -= 4;
1012: 
1013:   LLVM_DEBUG(dbgs() << format(", adjusted to %#x", BaseInsertOffset)
1014:                     << " la=" << Log2(Align) << '\n');
1015: 
1016:   // This could point off the end of the block if we've already got constant
1017:   // pool entries following this block; only the last one is in the water list.
1018:   // Back past any possible branches (allow for a conditional and a maximally
1019:   // long unconditional).
1020:   if (BaseInsertOffset + 8 >= UserBBI.postOffset()) {
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 1021-1040

```cpp
1021:     BaseInsertOffset = UserBBI.postOffset() - 8;
1022:     LLVM_DEBUG(dbgs() << format("Move inside block: %#x\n", BaseInsertOffset));
1023:   }
1024:   unsigned EndInsertOffset =
1025:       BaseInsertOffset + 4 + CPEMI->getOperand(2).getImm();
1026:   MachineBasicBlock::iterator MI = UserMI;
1027:   ++MI;
1028:   unsigned CPUIndex = CPUserIndex + 1;
1029:   unsigned NumCPUsers = CPUsers.size();
1030:   for (unsigned Offset = UserOffset + TII->getInstSizeInBytes(*UserMI);
1031:        Offset < BaseInsertOffset;
1032:        Offset += TII->getInstSizeInBytes(*MI), MI = std::next(MI)) {
1033:     assert(MI != UserMBB->end() && "Fell off end of block");
1034:     if (CPUIndex < NumCPUsers && CPUsers[CPUIndex].MI == MI) {
1035:       CPUser &U = CPUsers[CPUIndex];
1036:       if (!isOffsetInRange(Offset, EndInsertOffset, U)) {
1037:         // Shift intertion point by one unit of alignment so it is within reach.
1038:         BaseInsertOffset -= Align.value();
1039:         EndInsertOffset -= Align.value();
1040:       }
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 1041-1060

```cpp
1041:       // This is overly conservative, as we don't account for CPEMIs being
1042:       // reused within the block, but it doesn't matter much.  Also assume CPEs
1043:       // are added in order with alignment padding.  We may eventually be able
1044:       // to pack the aligned CPEs better.
1045:       EndInsertOffset += U.CPEMI->getOperand(2).getImm();
1046:       CPUIndex++;
1047:     }
1048:   }
1049: 
1050:   NewMBB = splitBlockBeforeInstr(*--MI);
1051: }
1052: 
1053: /// handleConstantPoolUser - Analyze the specified user, checking to see if it
1054: /// is out-of-range.  If so, pick up the constant pool value and move it some
1055: /// place in-range.  Return true if we changed any addresses (thus must run
1056: /// another pass of branch lengthening), false otherwise.
1057: bool CSKYConstantIslands::handleConstantPoolUser(unsigned CPUserIndex) {
1058:   CPUser &U = CPUsers[CPUserIndex];
1059:   MachineInstr *UserMI = U.MI;
1060:   MachineInstr *CPEMI = U.CPEMI;
```

- EN: Function bodies or method definitions such as handleConstantPoolUser contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: handleConstantPoolUser 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 1061-1080

```cpp
1061:   unsigned CPI = CPEMI->getOperand(1).getIndex();
1062:   unsigned Size = CPEMI->getOperand(2).getImm();
1063:   // Compute this only once, it's expensive.
1064:   unsigned UserOffset = getUserOffset(U);
1065: 
1066:   // See if the current entry is within range, or there is a clone of it
1067:   // in range.
1068:   int result = findInRangeCPEntry(U, UserOffset);
1069:   if (result == 1)
1070:     return false;
1071:   if (result == 2)
1072:     return true;
1073: 
1074:   // Look for water where we can place this CPE.
1075:   MachineBasicBlock *NewIsland = MF->CreateMachineBasicBlock();
1076:   MachineBasicBlock *NewMBB;
1077:   water_iterator IP;
1078:   if (findAvailableWater(U, UserOffset, IP)) {
1079:     LLVM_DEBUG(dbgs() << "Found water in range\n");
1080:     MachineBasicBlock *WaterBB = *IP;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 1081-1100

```cpp
1081: 
1082:     // If the original WaterList entry was "new water" on this iteration,
1083:     // propagate that to the new island.  This is just keeping NewWaterList
1084:     // updated to match the WaterList, which will be updated below.
1085:     if (NewWaterList.erase(WaterBB))
1086:       NewWaterList.insert(NewIsland);
1087: 
1088:     // The new CPE goes before the following block (NewMBB).
1089:     NewMBB = &*++WaterBB->getIterator();
1090:   } else {
1091:     LLVM_DEBUG(dbgs() << "No water found\n");
1092:     createNewWater(CPUserIndex, UserOffset, NewMBB);
1093: 
1094:     // splitBlockBeforeInstr adds to WaterList, which is important when it is
1095:     // called while handling branches so that the water will be seen on the
1096:     // next iteration for constant pools, but in this context, we don't want
1097:     // it.  Check for this so it will be removed from the WaterList.
1098:     // Also remove any entry from NewWaterList.
1099:     MachineBasicBlock *WaterBB = &*--NewMBB->getIterator();
1100:     IP = llvm::find(WaterList, WaterBB);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 1101-1120

```cpp
1101:     if (IP != WaterList.end())
1102:       NewWaterList.erase(WaterBB);
1103: 
1104:     // We are adding new water.  Update NewWaterList.
1105:     NewWaterList.insert(NewIsland);
1106:   }
1107: 
1108:   // Remove the original WaterList entry; we want subsequent insertions in
1109:   // this vicinity to go after the one we're about to insert.  This
1110:   // considerably reduces the number of times we have to move the same CPE
1111:   // more than once and is also important to ensure the algorithm terminates.
1112:   if (IP != WaterList.end())
1113:     WaterList.erase(IP);
1114: 
1115:   // Okay, we know we can put an island before NewMBB now, do it!
1116:   MF->insert(NewMBB->getIterator(), NewIsland);
1117: 
1118:   // Update internal data structures to account for the newly inserted MBB.
1119:   updateForInsertedWaterBlock(NewIsland);
1120: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 1121-1140

```cpp
1121:   // Decrement the old entry, and remove it if refcount becomes 0.
1122:   decrementCPEReferenceCount(CPI, CPEMI);
1123: 
1124:   // No existing clone of this CPE is within range.
1125:   // We will be generating a new clone.  Get a UID for it.
1126:   unsigned ID = createPICLabelUId();
1127: 
1128:   // Now that we have an island to add the CPE to, clone the original CPE and
1129:   // add it to the island.
1130:   U.HighWaterMark = NewIsland;
1131:   U.CPEMI = BuildMI(NewIsland, DebugLoc(), TII->get(CSKY::CONSTPOOL_ENTRY))
1132:                 .addImm(ID)
1133:                 .addConstantPoolIndex(CPI)
1134:                 .addImm(Size);
1135:   CPEntries[CPI].push_back(CPEntry(U.CPEMI, ID, 1));
1136:   ++NumCPEs;
1137: 
1138:   // Mark the basic block as aligned as required by the const-pool entry.
1139:   NewIsland->setAlignment(getCPEAlign(*U.CPEMI));
1140: 
```

- EN: This range continues the implementation of the backend component described by CSKYConstantIslandPass.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 1141-1160

```cpp
1141:   // Increase the size of the island block to account for the new entry.
1142:   BBInfo[NewIsland->getNumber()].Size += Size;
1143:   adjustBBOffsetsAfter(&*--NewIsland->getIterator());
1144: 
1145:   // Finally, change the CPI in the instruction operand to be ID.
1146:   for (unsigned I = 0, E = UserMI->getNumOperands(); I != E; ++I)
1147:     if (UserMI->getOperand(I).isCPI()) {
1148:       UserMI->getOperand(I).setIndex(ID);
1149:       break;
1150:     }
1151: 
1152:   LLVM_DEBUG(
1153:       dbgs() << "  Moved CPE to #" << ID << " CPI=" << CPI
1154:              << format(" offset=%#x\n", BBInfo[NewIsland->getNumber()].Offset));
1155: 
1156:   return true;
1157: }
1158: 
1159: /// removeDeadCPEMI - Remove a dead constant pool entry instruction. Update
1160: /// sizes and offsets of impacted basic blocks.
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 1161-1180

```cpp
1161: void CSKYConstantIslands::removeDeadCPEMI(MachineInstr *CPEMI) {
1162:   MachineBasicBlock *CPEBB = CPEMI->getParent();
1163:   unsigned Size = CPEMI->getOperand(2).getImm();
1164:   CPEMI->eraseFromParent();
1165:   BBInfo[CPEBB->getNumber()].Size -= Size;
1166:   // All succeeding offsets have the current size value added in, fix this.
1167:   if (CPEBB->empty()) {
1168:     BBInfo[CPEBB->getNumber()].Size = 0;
1169: 
1170:     // This block no longer needs to be aligned.
1171:     CPEBB->setAlignment(Align(4));
1172:   } else {
1173:     // Entries are sorted by descending alignment, so realign from the front.
1174:     CPEBB->setAlignment(getCPEAlign(*CPEBB->begin()));
1175:   }
1176: 
1177:   adjustBBOffsetsAfter(CPEBB);
1178:   // An island has only one predecessor BB and one successor BB. Check if
1179:   // this BB's predecessor jumps directly to this BB's successor. This
1180:   // shouldn't happen currently.
```

- EN: Function bodies or method definitions such as removeDeadCPEMI contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: removeDeadCPEMI 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 1181-1200

```cpp
1181:   assert(!bbIsJumpedOver(CPEBB) && "How did this happen?");
1182:   // FIXME: remove the empty blocks after all the work is done?
1183: }
1184: 
1185: /// removeUnusedCPEntries - Remove constant pool entries whose refcounts
1186: /// are zero.
1187: bool CSKYConstantIslands::removeUnusedCPEntries() {
1188:   unsigned MadeChange = false;
1189:   for (unsigned I = 0, E = CPEntries.size(); I != E; ++I) {
1190:     std::vector<CPEntry> &CPEs = CPEntries[I];
1191:     for (unsigned J = 0, Ee = CPEs.size(); J != Ee; ++J) {
1192:       if (CPEs[J].RefCount == 0 && CPEs[J].CPEMI) {
1193:         removeDeadCPEMI(CPEs[J].CPEMI);
1194:         CPEs[J].CPEMI = nullptr;
1195:         MadeChange = true;
1196:       }
1197:     }
1198:   }
1199:   return MadeChange;
1200: }
```

- EN: Function bodies or method definitions such as removeUnusedCPEntries contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: removeUnusedCPEntries 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 1201-1220

```cpp
1201: 
1202: /// isBBInRange - Returns true if the distance between specific MI and
1203: /// specific BB can fit in MI's displacement field.
1204: bool CSKYConstantIslands::isBBInRange(MachineInstr *MI,
1205:                                       MachineBasicBlock *DestBB,
1206:                                       unsigned MaxDisp) {
1207:   unsigned BrOffset = getOffsetOf(MI);
1208:   unsigned DestOffset = BBInfo[DestBB->getNumber()].Offset;
1209: 
1210:   LLVM_DEBUG(dbgs() << "Branch of destination " << printMBBReference(*DestBB)
1211:                     << " from " << printMBBReference(*MI->getParent())
1212:                     << " max delta=" << MaxDisp << " from " << getOffsetOf(MI)
1213:                     << " to " << DestOffset << " offset "
1214:                     << int(DestOffset - BrOffset) << "\t" << *MI);
1215: 
1216:   if (BrOffset <= DestOffset) {
1217:     // Branch before the Dest.
1218:     if (DestOffset - BrOffset <= MaxDisp)
1219:       return true;
1220:   } else {
```

- EN: Function bodies or method definitions such as isBBInRange contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: isBBInRange 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 1221-1240

```cpp
1221:     if (BrOffset - DestOffset <= MaxDisp)
1222:       return true;
1223:   }
1224:   return false;
1225: }
1226: 
1227: /// fixupImmediateBr - Fix up an immediate branch whose destination is too far
1228: /// away to fit in its displacement field.
1229: bool CSKYConstantIslands::fixupImmediateBr(ImmBranch &Br) {
1230:   MachineInstr *MI = Br.MI;
1231:   MachineBasicBlock *DestBB = TII->getBranchDestBlock(*MI);
1232: 
1233:   // Check to see if the DestBB is already in-range.
1234:   if (isBBInRange(MI, DestBB, Br.MaxDisp))
1235:     return false;
1236: 
1237:   if (!Br.IsCond)
1238:     return fixupUnconditionalBr(Br);
1239:   return fixupConditionalBr(Br);
1240: }
```

- EN: Function bodies or method definitions such as fixupImmediateBr contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: fixupImmediateBr 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 1241-1260

```cpp
1241: 
1242: /// fixupUnconditionalBr - Fix up an unconditional branch whose destination is
1243: /// too far away to fit in its displacement field. If the LR register has been
1244: /// spilled in the epilogue, then we can use BSR to implement a far jump.
1245: /// Otherwise, add an intermediate branch instruction to a branch.
1246: bool CSKYConstantIslands::fixupUnconditionalBr(ImmBranch &Br) {
1247:   MachineInstr *MI = Br.MI;
1248:   MachineBasicBlock *MBB = MI->getParent();
1249: 
1250:   if (!MFI->isLRSpilled())
1251:     report_fatal_error("underestimated function size");
1252: 
1253:   // Use BSR to implement far jump.
1254:   Br.MaxDisp = ((1 << (26 - 1)) - 1) * 2;
1255:   MI->setDesc(TII->get(CSKY::BSR32_BR));
1256:   BBInfo[MBB->getNumber()].Size += 4;
1257:   adjustBBOffsetsAfter(MBB);
1258:   ++NumUBrFixed;
1259: 
1260:   LLVM_DEBUG(dbgs() << "  Changed B to long jump " << *MI);
```

- EN: Function bodies or method definitions such as fixupUnconditionalBr contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: fixupUnconditionalBr 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 1261-1280

```cpp
1261: 
1262:   return true;
1263: }
1264: 
1265: /// fixupConditionalBr - Fix up a conditional branch whose destination is too
1266: /// far away to fit in its displacement field. It is converted to an inverse
1267: /// conditional branch + an unconditional branch to the destination.
1268: bool CSKYConstantIslands::fixupConditionalBr(ImmBranch &Br) {
1269:   MachineInstr *MI = Br.MI;
1270:   MachineBasicBlock *DestBB = TII->getBranchDestBlock(*MI);
1271: 
1272:   SmallVector<MachineOperand, 4> Cond;
1273:   Cond.push_back(MachineOperand::CreateImm(MI->getOpcode()));
1274:   Cond.push_back(MI->getOperand(0));
1275:   TII->reverseBranchCondition(Cond);
1276: 
1277:   // Add an unconditional branch to the destination and invert the branch
1278:   // condition to jump over it:
1279:   // bteqz L1
1280:   // =>
```

- EN: Function bodies or method definitions such as fixupConditionalBr contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: fixupConditionalBr 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 1281-1300

```cpp
1281:   // bnez L2
1282:   // b   L1
1283:   // L2:
1284: 
1285:   // If the branch is at the end of its MBB and that has a fall-through block,
1286:   // direct the updated conditional branch to the fall-through block. Otherwise,
1287:   // split the MBB before the next instruction.
1288:   MachineBasicBlock *MBB = MI->getParent();
1289:   MachineInstr *BMI = &MBB->back();
1290:   bool NeedSplit = (BMI != MI) || !bbHasFallthrough(MBB);
1291: 
1292:   ++NumCBrFixed;
1293:   if (BMI != MI) {
1294:     if (std::next(MachineBasicBlock::iterator(MI)) == std::prev(MBB->end()) &&
1295:         BMI->isUnconditionalBranch()) {
1296:       // Last MI in the BB is an unconditional branch. Can we simply invert the
1297:       // condition and swap destinations:
1298:       // beqz L1
1299:       // b   L2
1300:       // =>
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 1301-1320

```cpp
1301:       // bnez L2
1302:       // b   L1
1303:       MachineBasicBlock *NewDest = TII->getBranchDestBlock(*BMI);
1304:       if (isBBInRange(MI, NewDest, Br.MaxDisp)) {
1305:         LLVM_DEBUG(
1306:             dbgs() << "  Invert Bcc condition and swap its destination with "
1307:                    << *BMI);
1308:         BMI->getOperand(BMI->getNumExplicitOperands() - 1).setMBB(DestBB);
1309:         MI->getOperand(MI->getNumExplicitOperands() - 1).setMBB(NewDest);
1310: 
1311:         MI->setDesc(TII->get(Cond[0].getImm()));
1312:         return true;
1313:       }
1314:     }
1315:   }
1316: 
1317:   if (NeedSplit) {
1318:     splitBlockBeforeInstr(*MI);
1319:     // No need for the branch to the next block. We're adding an unconditional
1320:     // branch to the destination.
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 1321-1340

```cpp
1321:     int Delta = TII->getInstSizeInBytes(MBB->back());
1322:     BBInfo[MBB->getNumber()].Size -= Delta;
1323:     MBB->back().eraseFromParent();
1324:     // BBInfo[SplitBB].Offset is wrong temporarily, fixed below
1325: 
1326:     // The conditional successor will be swapped between the BBs after this, so
1327:     // update CFG.
1328:     MBB->addSuccessor(DestBB);
1329:     std::next(MBB->getIterator())->removeSuccessor(DestBB);
1330:   }
1331:   MachineBasicBlock *NextBB = &*++MBB->getIterator();
1332: 
1333:   LLVM_DEBUG(dbgs() << "  Insert B to " << printMBBReference(*DestBB)
1334:                     << " also invert condition and change dest. to "
1335:                     << printMBBReference(*NextBB) << "\n");
1336: 
1337:   // Insert a new conditional branch and a new unconditional branch.
1338:   // Also update the ImmBranch as well as adding a new entry for the new branch.
1339: 
1340:   BuildMI(MBB, DebugLoc(), TII->get(Cond[0].getImm()))
```

- EN: This range continues the implementation of the backend component described by CSKYConstantIslandPass.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 1341-1360

```cpp
1341:       .addReg(MI->getOperand(0).getReg())
1342:       .addMBB(NextBB);
1343: 
1344:   Br.MI = &MBB->back();
1345:   BBInfo[MBB->getNumber()].Size += TII->getInstSizeInBytes(MBB->back());
1346:   BuildMI(MBB, DebugLoc(), TII->get(Br.UncondBr)).addMBB(DestBB);
1347:   BBInfo[MBB->getNumber()].Size += TII->getInstSizeInBytes(MBB->back());
1348:   unsigned MaxDisp = getUnconditionalBrDisp(Br.UncondBr);
1349:   ImmBranches.push_back(ImmBranch(&MBB->back(), MaxDisp, false, Br.UncondBr));
1350: 
1351:   // Remove the old conditional branch.  It may or may not still be in MBB.
1352:   BBInfo[MI->getParent()->getNumber()].Size -= TII->getInstSizeInBytes(*MI);
1353:   MI->eraseFromParent();
1354:   adjustBBOffsetsAfter(MBB);
1355:   return true;
1356: }
1357: 
1358: /// Returns a pass that converts branches to long branches.
1359: FunctionPass *llvm::createCSKYConstantIslandPass() {
1360:   return new CSKYConstantIslands();
```

- EN: This range continues the implementation of the backend component described by CSKYConstantIslandPass.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 1361-1365

```cpp
1361: }
1362: 
1363: INITIALIZE_PASS(CSKYConstantIslands, DEBUG_TYPE,
1364:                 "CSKY constant island placement and branch shortening pass",
1365:                 false, false)
```

- EN: This range continues the implementation of the backend component described by CSKYConstantIslandPass.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

## Key Concepts / 关键概念

- Machine instruction manipulation / 机器指令操作
- MachineFunction state / MachineFunction 状态
- Basic block level transformation / 基本块级转换
- CPU feature modelling / CPU 特性建模
- Pseudo-instruction handling / 伪指令处理

## Dependencies / 依赖关系

- Direct includes / 直接包含: `CSKY.h`, `CSKYConstantPoolValue.h`, `CSKYMachineFunctionInfo.h`, `CSKYSubtarget.h`, `llvm/ADT/STLExtras.h`, `llvm/ADT/SmallSet.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Statistic.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen
