# SystemZInstrInfo.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZInstrInfo.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file defines or implements target instruction information for the SystemZ backend.
- **用途 (CN)**: 该文件用于 SystemZ 后端，负责定义或实现目标指令信息。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
```cpp
   1: //===-- SystemZInstrInfo.h - SystemZ instruction information ----*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file contains the SystemZ implementation of the TargetInstrInfo class.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #ifndef LLVM_LIB_TARGET_SYSTEMZ_SYSTEMZINSTRINFO_H
  14: #define LLVM_LIB_TARGET_SYSTEMZ_SYSTEMZINSTRINFO_H
  15: 
  16: #include "SystemZ.h"
  17: #include "SystemZRegisterInfo.h"
  18: #include "llvm/ADT/ArrayRef.h"
  19: #include "llvm/CodeGen/MachineBasicBlock.h"
  20: #include "llvm/CodeGen/MachineFunction.h"
  21: #include "llvm/CodeGen/MachineInstrBuilder.h"
  22: #include "llvm/CodeGen/TargetInstrInfo.h"
  23: #include <cstdint>
  24: 
  25: #define GET_INSTRINFO_HEADER
  26: #include "SystemZGenInstrInfo.inc"
  27: 
  28: namespace llvm {
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `SystemZ.h`, `SystemZRegisterInfo.h`, `ArrayRef.h`, `MachineBasicBlock.h`, `MachineFunction.h`, `MachineInstrBuilder.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `SystemZ.h`, `SystemZRegisterInfo.h`, `ArrayRef.h`, `MachineBasicBlock.h`, `MachineFunction.h`, `MachineInstrBuilder.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 29-56 / 第 29-56 行
```cpp
  29: 
  30: class SystemZSubtarget;
  31: 
  32: namespace SystemZII {
  33: 
  34: enum {
  35:   // See comments in SystemZInstrFormats.td.
  36:   SimpleBDXLoad          = (1 << 0),
  37:   SimpleBDXStore         = (1 << 1),
  38:   Has20BitOffset         = (1 << 2),
  39:   HasIndex               = (1 << 3),
  40:   Is128Bit               = (1 << 4),
  41:   AccessSizeMask         = (31 << 5),
  42:   AccessSizeShift        = 5,
  43:   CCValuesMask           = (15 << 10),
  44:   CCValuesShift          = 10,
  45:   CompareZeroCCMaskMask  = (15 << 14),
  46:   CompareZeroCCMaskShift = 14,
  47:   CCMaskFirst            = (1 << 18),
  48:   CCMaskLast             = (1 << 19),
  49:   IsLogical              = (1 << 20),
  50:   CCIfNoSignedWrap       = (1 << 21)
  51: };
  52: 
  53: static inline unsigned getAccessSize(unsigned int Flags) {
  54:   return (Flags & AccessSizeMask) >> AccessSizeShift;
  55: }
  56: 
```
- **EN**: This block declares or refines TableGen records such as `SystemZSubtarget`. The range implements or declares functions including `getAccessSize`.
- **CN**: 该代码块声明或细化了 `SystemZSubtarget` 等 TableGen 记录。 这一段实现或声明了 `getAccessSize` 等函数。

### Lines 57-84 / 第 57-84 行
```cpp
  57: static inline unsigned getCCValues(unsigned int Flags) {
  58:   return (Flags & CCValuesMask) >> CCValuesShift;
  59: }
  60: 
  61: static inline unsigned getCompareZeroCCMask(unsigned int Flags) {
  62:   return (Flags & CompareZeroCCMaskMask) >> CompareZeroCCMaskShift;
  63: }
  64: 
  65: // SystemZ MachineOperand target flags.
  66: enum {
  67:   // Masks out the bits for the access model.
  68:   MO_SYMBOL_MODIFIER = (3 << 0),
  69: 
  70:   // @GOT (aka @GOTENT)
  71:   MO_GOT = (1 << 0),
  72: 
  73:   // @INDNTPOFF
  74:   MO_INDNTPOFF = (2 << 0),
  75: 
  76:   // z/OS XPLink specific: classifies the types of
  77:   // accesses to the ADA (Associated Data Area).
  78:   MO_ADA_DATA_SYMBOL_ADDR = (1 << 2),
  79:   MO_ADA_INDIRECT_FUNC_DESC = (2 << 2),
  80:   MO_ADA_DIRECT_FUNC_DESC = (3 << 2),
  81: };
  82: 
  83: // Classifies a branch.
  84: enum BranchType {
```
- **EN**: The range implements or declares functions including `getCCValues`, `getCompareZeroCCMask`.
- **CN**: 这一段实现或声明了 `getCCValues`, `getCompareZeroCCMask` 等函数。

### Lines 85-112 / 第 85-112 行
```cpp
  85:   // An instruction that branches on the current value of CC.
  86:   BranchNormal,
  87: 
  88:   // An instruction that peforms a 32-bit signed comparison and branches
  89:   // on the result.
  90:   BranchC,
  91: 
  92:   // An instruction that peforms a 32-bit unsigned comparison and branches
  93:   // on the result.
  94:   BranchCL,
  95: 
  96:   // An instruction that peforms a 64-bit signed comparison and branches
  97:   // on the result.
  98:   BranchCG,
  99: 
 100:   // An instruction that peforms a 64-bit unsigned comparison and branches
 101:   // on the result.
 102:   BranchCLG,
 103: 
 104:   // An instruction that decrements a 32-bit register and branches if
 105:   // the result is nonzero.
 106:   BranchCT,
 107: 
 108:   // An instruction that decrements a 64-bit register and branches if
 109:   // the result is nonzero.
 110:   BranchCTG,
 111: 
 112:   // An instruction representing an asm goto statement.
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 113-140 / 第 113-140 行
```cpp
 113:   AsmGoto
 114: };
 115: 
 116: // Information about a branch instruction.
 117: class Branch {
 118:   // The target of the branch. In case of INLINEASM_BR, this is nullptr.
 119:   const MachineOperand *Target;
 120: 
 121: public:
 122:   // The type of the branch.
 123:   BranchType Type;
 124: 
 125:   // CCMASK_<N> is set if CC might be equal to N.
 126:   unsigned CCValid;
 127: 
 128:   // CCMASK_<N> is set if the branch should be taken when CC == N.
 129:   unsigned CCMask;
 130: 
 131:   Branch(BranchType type, unsigned ccValid, unsigned ccMask,
 132:          const MachineOperand *target)
 133:     : Target(target), Type(type), CCValid(ccValid), CCMask(ccMask) {}
 134: 
 135:   bool isIndirect() { return Target != nullptr && Target->isReg(); }
 136:   bool hasMBBTarget() { return Target != nullptr && Target->isMBB(); }
 137:   MachineBasicBlock *getMBBTarget() {
 138:     return hasMBBTarget() ? Target->getMBB() : nullptr;
 139:   }
 140: };
```
- **EN**: This block declares or refines TableGen records such as `Branch`. The range implements or declares functions including `Branch`, `hasMBBTarget`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 该代码块声明或细化了 `Branch` 等 TableGen 记录。 这一段实现或声明了 `Branch`, `hasMBBTarget` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 141-168 / 第 141-168 行
```cpp
 141: 
 142: // Kinds of fused compares in compare-and-* instructions.  Together with type
 143: // of the converted compare, this identifies the compare-and-*
 144: // instruction.
 145: enum FusedCompareType {
 146:   // Relative branch - CRJ etc.
 147:   CompareAndBranch,
 148: 
 149:   // Indirect branch, used for return - CRBReturn etc.
 150:   CompareAndReturn,
 151: 
 152:   // Indirect branch, used for sibcall - CRBCall etc.
 153:   CompareAndSibcall,
 154: 
 155:   // Trap
 156:   CompareAndTrap
 157: };
 158: 
 159: } // end namespace SystemZII
 160: 
 161: namespace SystemZ {
 162: int32_t getTwoOperandOpcode(uint32_t Opcode);
 163: int32_t getTargetMemOpcode(uint32_t Opcode);
 164: 
 165: // Return a version of comparison CC mask CCMask in which the LT and GT
 166: // actions are swapped.
 167: unsigned reverseCCMask(unsigned CCMask);
 168: 
```
- **EN**: This span continues the file's main responsibility: this file defines or implements target instruction information for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 169-196 / 第 169-196 行
```cpp
 169: // Create a new basic block after MBB.
 170: MachineBasicBlock *emitBlockAfter(MachineBasicBlock *MBB);
 171: // Split MBB after MI and return the new block (the one that contains
 172: // instructions after MI).
 173: MachineBasicBlock *splitBlockAfter(MachineBasicBlock::iterator MI,
 174:                                    MachineBasicBlock *MBB);
 175: // Split MBB before MI and return the new block (the one that contains MI).
 176: MachineBasicBlock *splitBlockBefore(MachineBasicBlock::iterator MI,
 177:                                     MachineBasicBlock *MBB);
 178: }
 179: 
 180: class SystemZInstrInfo : public SystemZGenInstrInfo {
 181:   const SystemZRegisterInfo RI;
 182:   const SystemZSubtarget &STI;
 183: 
 184:   void splitMove(MachineBasicBlock::iterator MI, unsigned NewOpcode) const;
 185:   void splitAdjDynAlloc(MachineBasicBlock::iterator MI) const;
 186:   void expandRIPseudo(MachineInstr &MI, unsigned LowOpcode, unsigned HighOpcode,
 187:                       bool ConvertHigh) const;
 188:   void expandRIEPseudo(MachineInstr &MI, unsigned LowOpcode,
 189:                        unsigned LowOpcodeK, unsigned HighOpcode) const;
 190:   void expandRXYPseudo(MachineInstr &MI, unsigned LowOpcode,
 191:                        unsigned HighOpcode) const;
 192:   void expandLOCPseudo(MachineInstr &MI, unsigned LowOpcode,
 193:                        unsigned HighOpcode) const;
 194:   void expandZExtPseudo(MachineInstr &MI, unsigned LowOpcode,
 195:                         unsigned Size) const;
 196:   void expandLoadStackGuard(MachineInstr *MI) const;
```
- **EN**: This block declares or refines TableGen records such as `SystemZInstrInfo`.
- **CN**: 该代码块声明或细化了 `SystemZInstrInfo` 等 TableGen 记录。

### Lines 197-224 / 第 197-224 行
```cpp
 197: 
 198:   MachineInstrBuilder
 199:   emitGRX32Move(MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
 200:                 const DebugLoc &DL, unsigned DestReg, unsigned SrcReg,
 201:                 unsigned LowLowOpcode, unsigned Size, bool KillSrc,
 202:                 bool UndefSrc) const;
 203: 
 204:   virtual void anchor();
 205: 
 206: protected:
 207:   /// Commutes the operands in the given instruction by changing the operands
 208:   /// order and/or changing the instruction's opcode and/or the immediate value
 209:   /// operand.
 210:   ///
 211:   /// The arguments 'CommuteOpIdx1' and 'CommuteOpIdx2' specify the operands
 212:   /// to be commuted.
 213:   ///
 214:   /// Do not call this method for a non-commutable instruction or
 215:   /// non-commutable operands.
 216:   /// Even though the instruction is commutable, the method may still
 217:   /// fail to commute the operands, null pointer is returned in such cases.
 218:   MachineInstr *commuteInstructionImpl(MachineInstr &MI, bool NewMI,
 219:                                        unsigned CommuteOpIdx1,
 220:                                        unsigned CommuteOpIdx2) const override;
 221: 
 222: public:
 223:   explicit SystemZInstrInfo(const SystemZSubtarget &STI);
 224: 
```
- **EN**: This span continues the file's main responsibility: this file defines or implements target instruction information for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 225-252 / 第 225-252 行
```cpp
 225:   // Override TargetInstrInfo.
 226:   Register isLoadFromStackSlot(const MachineInstr &MI,
 227:                                int &FrameIndex) const override;
 228:   Register isStoreToStackSlot(const MachineInstr &MI,
 229:                               int &FrameIndex) const override;
 230:   Register isLoadFromStackSlotPostFE(const MachineInstr &MI,
 231:                                      int &FrameIndex) const override;
 232:   Register isStoreToStackSlotPostFE(const MachineInstr &MI,
 233:                                     int &FrameIndex) const override;
 234:   bool isStackSlotCopy(const MachineInstr &MI, int &DestFrameIndex,
 235:                        int &SrcFrameIndex) const override;
 236:   bool analyzeBranch(MachineBasicBlock &MBB, MachineBasicBlock *&TBB,
 237:                      MachineBasicBlock *&FBB,
 238:                      SmallVectorImpl<MachineOperand> &Cond,
 239:                      bool AllowModify) const override;
 240:   unsigned removeBranch(MachineBasicBlock &MBB,
 241:                         int *BytesRemoved = nullptr) const override;
 242:   unsigned insertBranch(MachineBasicBlock &MBB, MachineBasicBlock *TBB,
 243:                         MachineBasicBlock *FBB, ArrayRef<MachineOperand> Cond,
 244:                         const DebugLoc &DL,
 245:                         int *BytesAdded = nullptr) const override;
 246:   bool analyzeCompare(const MachineInstr &MI, Register &SrcReg,
 247:                       Register &SrcReg2, int64_t &Mask,
 248:                       int64_t &Value) const override;
 249:   bool canInsertSelect(const MachineBasicBlock &, ArrayRef<MachineOperand> Cond,
 250:                        Register, Register, Register, int &, int &,
 251:                        int &) const override;
 252:   void insertSelect(MachineBasicBlock &MBB, MachineBasicBlock::iterator MI,
```
- **EN**: This span continues the file's main responsibility: this file defines or implements target instruction information for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 253-280 / 第 253-280 行
```cpp
 253:                     const DebugLoc &DL, Register DstReg,
 254:                     ArrayRef<MachineOperand> Cond, Register TrueReg,
 255:                     Register FalseReg) const override;
 256:   bool foldImmediate(MachineInstr &UseMI, MachineInstr &DefMI, Register Reg,
 257:                      MachineRegisterInfo *MRI) const override;
 258: 
 259:   bool isPredicable(const MachineInstr &MI) const override;
 260:   bool isProfitableToIfCvt(MachineBasicBlock &MBB, unsigned NumCycles,
 261:                            unsigned ExtraPredCycles,
 262:                            BranchProbability Probability) const override;
 263:   bool isProfitableToIfCvt(MachineBasicBlock &TMBB,
 264:                            unsigned NumCyclesT, unsigned ExtraPredCyclesT,
 265:                            MachineBasicBlock &FMBB,
 266:                            unsigned NumCyclesF, unsigned ExtraPredCyclesF,
 267:                            BranchProbability Probability) const override;
 268:   bool isProfitableToDupForIfCvt(MachineBasicBlock &MBB, unsigned NumCycles,
 269:                             BranchProbability Probability) const override;
 270:   bool PredicateInstruction(MachineInstr &MI,
 271:                             ArrayRef<MachineOperand> Pred) const override;
 272:   void copyPhysReg(MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
 273:                    const DebugLoc &DL, Register DestReg, Register SrcReg,
 274:                    bool KillSrc, bool RenamableDest = false,
 275:                    bool RenamableSrc = false) const override;
 276:   void storeRegToStackSlot(
 277:       MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI, Register SrcReg,
 278:       bool isKill, int FrameIndex, const TargetRegisterClass *RC,
 279: 
 280:       Register VReg,
```
- **EN**: This span continues the file's main responsibility: this file defines or implements target instruction information for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 281-308 / 第 281-308 行
```cpp
 281:       MachineInstr::MIFlag Flags = MachineInstr::NoFlags) const override;
 282:   void loadRegFromStackSlot(
 283:       MachineBasicBlock &MBB, MachineBasicBlock::iterator MBBI,
 284:       Register DestReg, int FrameIdx, const TargetRegisterClass *RC,
 285:       Register VReg, unsigned SubReg = 0,
 286:       MachineInstr::MIFlag Flags = MachineInstr::NoFlags) const override;
 287:   MachineInstr *convertToThreeAddress(MachineInstr &MI, LiveVariables *LV,
 288:                                       LiveIntervals *LIS) const override;
 289: 
 290:   bool useMachineCombiner() const override { return true; }
 291:   bool isAssociativeAndCommutative(const MachineInstr &Inst,
 292:                                    bool Invert) const override;
 293:   std::optional<unsigned> getInverseOpcode(unsigned Opcode) const override;
 294: 
 295:   MachineInstr *foldMemoryOperandImpl(MachineFunction &MF, MachineInstr &MI,
 296:                                       ArrayRef<unsigned> Ops, int FrameIndex,
 297:                                       MachineInstr *&CopyMI,
 298:                                       LiveIntervals *LIS = nullptr,
 299:                                       VirtRegMap *VRM = nullptr) const override;
 300:   MachineInstr *foldMemoryOperandImpl(MachineFunction &MF, MachineInstr &MI,
 301:                                       ArrayRef<unsigned> Ops,
 302:                                       MachineInstr &LoadMI,
 303:                                       MachineInstr *&CopyMI,
 304:                                       LiveIntervals *LIS = nullptr,
 305:                                       VirtRegMap *VRM = nullptr) const override;
 306:   bool expandPostRAPseudo(MachineInstr &MBBI) const override;
 307:   bool reverseBranchCondition(SmallVectorImpl<MachineOperand> &Cond) const
 308:     override;
```
- **EN**: The range implements or declares functions including `reverseBranchCondition`.
- **CN**: 这一段实现或声明了 `reverseBranchCondition` 等函数。

### Lines 309-336 / 第 309-336 行
```cpp
 309: 
 310:   // Return the SystemZRegisterInfo, which this class owns.
 311:   const SystemZRegisterInfo &getRegisterInfo() const { return RI; }
 312: 
 313:   // Return the size in bytes of MI.
 314:   unsigned getInstSizeInBytes(const MachineInstr &MI) const override;
 315: 
 316:   // Return true if MI is a conditional or unconditional branch.
 317:   // When returning true, set Cond to the mask of condition-code
 318:   // values on which the instruction will branch, and set Target
 319:   // to the operand that contains the branch target.  This target
 320:   // can be a register or a basic block.
 321:   SystemZII::Branch getBranchInfo(const MachineInstr &MI) const;
 322: 
 323:   // Get the load and store opcodes for a given register class.
 324:   void getLoadStoreOpcodes(const TargetRegisterClass *RC,
 325:                            unsigned &LoadOpcode, unsigned &StoreOpcode) const;
 326: 
 327:   // Opcode is the opcode of an instruction that has an address operand,
 328:   // and the caller wants to perform that instruction's operation on an
 329:   // address that has displacement Offset.  Return the opcode of a suitable
 330:   // instruction (which might be Opcode itself) or 0 if no such instruction
 331:   // exists.  MI may be passed in order to allow examination of physical
 332:   // register operands (i.e. if a VR32/64 reg ended up as an FP or Vector reg).
 333:   unsigned getOpcodeForOffset(unsigned Opcode, int64_t Offset,
 334:                               const MachineInstr *MI = nullptr) const;
 335: 
 336:   // Return true if Opcode has a mapping in 12 <-> 20 bit displacements.
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 337-364 / 第 337-364 行
```cpp
 337:   bool hasDisplacementPairInsn(unsigned Opcode) const;
 338: 
 339:   // If Opcode is a load instruction that has a LOAD AND TEST form,
 340:   // return the opcode for the testing form, otherwise return 0.
 341:   unsigned getLoadAndTest(unsigned Opcode) const;
 342: 
 343:   // Return true if ROTATE AND ... SELECTED BITS can be used to select bits
 344:   // Mask of the R2 operand, given that only the low BitSize bits of Mask are
 345:   // significant.  Set Start and End to the I3 and I4 operands if so.
 346:   bool isRxSBGMask(uint64_t Mask, unsigned BitSize,
 347:                    unsigned &Start, unsigned &End) const;
 348: 
 349:   // If Opcode is a COMPARE opcode for which an associated fused COMPARE AND *
 350:   // operation exists, return the opcode for the latter, otherwise return 0.
 351:   // MI, if nonnull, is the compare instruction.
 352:   unsigned getFusedCompare(unsigned Opcode,
 353:                            SystemZII::FusedCompareType Type,
 354:                            const MachineInstr *MI = nullptr) const;
 355: 
 356:   // Return true if this is a load and test which can be optimized the
 357:   // same way as compare instruction.
 358:   bool isLoadAndTestAsCmp(const MachineInstr &MI) const;
 359: 
 360:   // Return true if Compare is a comparison against zero.
 361:   bool isCompareZero(const MachineInstr &Compare) const;
 362: 
 363:   // Return the source register of Compare, which is the unknown value
 364:   // being tested.
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 365-392 / 第 365-392 行
```cpp
 365:   Register getCompareSourceReg(const MachineInstr &Compare) const;
 366: 
 367:   // Try to find all CC users of the compare instruction (MBBI) and update
 368:   // all of them to maintain equivalent behavior after swapping the compare
 369:   // operands. Return false if not all users can be conclusively found and
 370:   // handled. The compare instruction is *not* changed.
 371:   bool prepareCompareSwapOperands(MachineBasicBlock::iterator MBBI) const;
 372: 
 373:   // If Opcode is a LOAD opcode for with an associated LOAD AND TRAP
 374:   // operation exists, returh the opcode for the latter, otherwise return 0.
 375:   unsigned getLoadAndTrap(unsigned Opcode) const;
 376: 
 377:   // Emit code before MBBI in MI to move immediate value Value into
 378:   // physical register Reg.
 379:   void loadImmediate(MachineBasicBlock &MBB,
 380:                      MachineBasicBlock::iterator MBBI,
 381:                      unsigned Reg, uint64_t Value) const;
 382: 
 383:   // Perform target specific instruction verification.
 384:   bool verifyInstruction(const MachineInstr &MI,
 385:                          StringRef &ErrInfo) const override;
 386: 
 387:   // Sometimes, it is possible for the target to tell, even without
 388:   // aliasing information, that two MIs access different memory
 389:   // addresses. This function returns true if two MIs access different
 390:   // memory addresses and false otherwise.
 391:   bool
 392:   areMemAccessesTriviallyDisjoint(const MachineInstr &MIa,
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 393-412 / 第 393-412 行
```cpp
 393:                                   const MachineInstr &MIb) const override;
 394: 
 395:   bool getConstValDefinedInReg(const MachineInstr &MI, const Register Reg,
 396:                                int64_t &ImmVal) const override;
 397: 
 398:   std::optional<DestSourcePair>
 399:   isCopyInstrImpl(const MachineInstr &MI) const override;
 400: 
 401:   std::pair<unsigned, unsigned>
 402:   decomposeMachineOperandsTargetFlags(unsigned TF) const override;
 403: 
 404:   ArrayRef<std::pair<unsigned, const char *>>
 405:   getSerializableDirectMachineOperandTargetFlags() const override;
 406: 
 407:   MCInst getNop() const override;
 408: };
 409: 
 410: } // end namespace llvm
 411: 
 412: #endif // LLVM_LIB_TARGET_SYSTEMZ_SYSTEMZINSTRINFO_H
```
- **EN**: This span continues the file's main responsibility: this file defines or implements target instruction information for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **MC layer**: Uses LLVM MC structures for encoding, printing, parsing, or disassembly. / 使用 LLVM MC 结构进行编码、打印、解析或反汇编。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Instruction metadata**: Captures opcodes, operands, patterns, and helper routines. / 描述操作码、操作数、匹配模式和辅助例程。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- `SystemZ.h`
- `SystemZRegisterInfo.h`
- `llvm/ADT/ArrayRef.h`
- `llvm/CodeGen/MachineBasicBlock.h`
- `llvm/CodeGen/MachineFunction.h`
- `llvm/CodeGen/MachineInstrBuilder.h`
- `llvm/CodeGen/TargetInstrInfo.h`
- `cstdint`
- `SystemZGenInstrInfo.inc`
