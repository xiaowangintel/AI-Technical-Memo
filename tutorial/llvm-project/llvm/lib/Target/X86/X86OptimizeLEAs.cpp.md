# X86OptimizeLEAs.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/X86OptimizeLEAs.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements LEA optimization for the core X86 backend. / 实现X86 后端核心中的LEA 优化。

## Line-by-Line Analysis / 逐行分析
### Lines 1-50: File header, licensing, and opening context / 文件头、许可证与开场上下文
```cpp
//===- X86OptimizeLEAs.cpp - optimize usage of LEA instructions -----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the pass that performs some optimizations with LEA
// instructions in order to improve performance and code size.
// Currently, it does two things:
// 1) If there are two LEA instructions calculating addresses which only differ
//    by displacement inside a basic block, one of them is removed.
// 2) Address calculations in load and store instructions are replaced by
//    existing LEA def registers where possible.
//
//===----------------------------------------------------------------------===//

#include "MCTargetDesc/X86BaseInfo.h"
#include "X86.h"
#include "X86InstrInfo.h"
#include "X86Subtarget.h"
#include "llvm/ADT/DenseMap.h"
#include "llvm/ADT/DenseMapInfo.h"
#include "llvm/ADT/Hashing.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/Analysis/ProfileSummaryInfo.h"
#include "llvm/CodeGen/LazyMachineBlockFrequencyInfo.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineOperand.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/MachineSizeOpts.h"
#include "llvm/CodeGen/TargetOpcodes.h"
#include "llvm/CodeGen/TargetRegisterInfo.h"
#include "llvm/IR/DebugInfoMetadata.h"
#include "llvm/IR/DebugLoc.h"
#include "llvm/IR/Function.h"
#include "llvm/MC/MCInstrDesc.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/MathExtras.h"
#include "llvm/Support/raw_ostream.h"
#include <cassert>
#include <cstdint>
```
**EN:** This section implements file header, licensing, and opening context for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的文件头、许可证与开场上下文。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 51-100: Preprocessor directives and structural setup / 预处理指令与结构初始化
```cpp
#include <iterator>

using namespace llvm;

#define DEBUG_TYPE "x86-optimize-leas"

static cl::opt<bool>
    DisableX86LEAOpt("disable-x86-lea-opt", cl::Hidden,
                     cl::desc("X86: Disable LEA optimizations."),
                     cl::init(false));

STATISTIC(NumSubstLEAs, "Number of LEA instruction substitutions");
STATISTIC(NumRedundantLEAs, "Number of redundant LEA instructions removed");

/// Returns true if two machine operands are identical and they are not
/// physical registers.
static inline bool isIdenticalOp(const MachineOperand &MO1,
                                 const MachineOperand &MO2);

/// Returns true if two address displacement operands are of the same
/// type and use the same symbol/index/address regardless of the offset.
static bool isSimilarDispOp(const MachineOperand &MO1,
                            const MachineOperand &MO2);

/// Returns true if the instruction is LEA.
static inline bool isLEA(const MachineInstr &MI);

namespace {

/// A key based on instruction's memory operands.
class MemOpKey {
public:
  MemOpKey(const MachineOperand *Base, const MachineOperand *Scale,
           const MachineOperand *Index, const MachineOperand *Segment,
           const MachineOperand *Disp)
      : Disp(Disp) {
    Operands[0] = Base;
    Operands[1] = Scale;
    Operands[2] = Index;
    Operands[3] = Segment;
  }

  bool operator==(const MemOpKey &Other) const {
    // Addresses' bases, scales, indices and segments must be identical.
    for (int i = 0; i < 4; ++i)
      if (!isIdenticalOp(*Operands[i], *Other.Operands[i]))
        return false;

    // Addresses' displacements don't have to be exactly the same. It only
    // matters that they use the same symbol/index/address. Immediates' or
```
**EN:** This section implements preprocessor directives and structural setup for the core X86 backend. Key symbols include MemOpKey. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的预处理指令与结构初始化。关键符号包括 MemOpKey。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 101-150: Namespace scope management / 命名空间作用域管理
```cpp
    // offsets' differences will be taken care of during instruction
    // substitution.
    return isSimilarDispOp(*Disp, *Other.Disp);
  }

  // Address' base, scale, index and segment operands.
  const MachineOperand *Operands[4];

  // Address' displacement operand.
  const MachineOperand *Disp;
};

} // end anonymous namespace

namespace llvm {

/// Provide DenseMapInfo for MemOpKey.
template <> struct DenseMapInfo<MemOpKey> {
  using PtrInfo = DenseMapInfo<const MachineOperand *>;

  static inline MemOpKey getEmptyKey() {
    return MemOpKey(PtrInfo::getEmptyKey(), PtrInfo::getEmptyKey(),
                    PtrInfo::getEmptyKey(), PtrInfo::getEmptyKey(),
                    PtrInfo::getEmptyKey());
  }

  static inline MemOpKey getTombstoneKey() {
    return MemOpKey(PtrInfo::getTombstoneKey(), PtrInfo::getTombstoneKey(),
                    PtrInfo::getTombstoneKey(), PtrInfo::getTombstoneKey(),
                    PtrInfo::getTombstoneKey());
  }

  static unsigned getHashValue(const MemOpKey &Val) {
    // Checking any field of MemOpKey is enough to determine if the key is
    // empty or tombstone.
    assert(Val.Disp != PtrInfo::getEmptyKey() && "Cannot hash the empty key");
    assert(Val.Disp != PtrInfo::getTombstoneKey() &&
           "Cannot hash the tombstone key");

    hash_code Hash = hash_combine(*Val.Operands[0], *Val.Operands[1],
                                  *Val.Operands[2], *Val.Operands[3]);

    // If the address displacement is an immediate, it should not affect the
    // hash so that memory operands which differ only be immediate displacement
    // would have the same hash. If the address displacement is something else,
    // we should reflect symbol/index/address in the hash.
    switch (Val.Disp->getType()) {
    case MachineOperand::MO_Immediate:
      break;
    case MachineOperand::MO_ConstantPoolIndex:
```
**EN:** This section implements namespace scope management for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的命名空间作用域管理。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 151-200: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
    case MachineOperand::MO_JumpTableIndex:
      Hash = hash_combine(Hash, Val.Disp->getIndex());
      break;
    case MachineOperand::MO_ExternalSymbol:
      Hash = hash_combine(Hash, Val.Disp->getSymbolName());
      break;
    case MachineOperand::MO_GlobalAddress:
      Hash = hash_combine(Hash, Val.Disp->getGlobal());
      break;
    case MachineOperand::MO_BlockAddress:
      Hash = hash_combine(Hash, Val.Disp->getBlockAddress());
      break;
    case MachineOperand::MO_MCSymbol:
      Hash = hash_combine(Hash, Val.Disp->getMCSymbol());
      break;
    case MachineOperand::MO_MachineBasicBlock:
      Hash = hash_combine(Hash, Val.Disp->getMBB());
      break;
    default:
      llvm_unreachable("Invalid address displacement operand");
    }

    return (unsigned)Hash;
  }

  static bool isEqual(const MemOpKey &LHS, const MemOpKey &RHS) {
    // Checking any field of MemOpKey is enough to determine if the key is
    // empty or tombstone.
    if (RHS.Disp == PtrInfo::getEmptyKey())
      return LHS.Disp == PtrInfo::getEmptyKey();
    if (RHS.Disp == PtrInfo::getTombstoneKey())
      return LHS.Disp == PtrInfo::getTombstoneKey();
    return LHS == RHS;
  }
};

} // end namespace llvm

/// Returns a hash table key based on memory operands of \p MI. The
/// number of the first memory operand of \p MI is specified through \p N.
static inline MemOpKey getMemOpKey(const MachineInstr &MI, unsigned N) {
  assert((isLEA(MI) || MI.mayLoadOrStore()) &&
         "The instruction must be a LEA, a load or a store");
  return MemOpKey(&MI.getOperand(N + X86::AddrBaseReg),
                  &MI.getOperand(N + X86::AddrScaleAmt),
                  &MI.getOperand(N + X86::AddrIndexReg),
                  &MI.getOperand(N + X86::AddrSegmentReg),
                  &MI.getOperand(N + X86::AddrDisp));
}

```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 201-250: Preprocessor directives and structural setup / 预处理指令与结构初始化
```cpp
static inline bool isIdenticalOp(const MachineOperand &MO1,
                                 const MachineOperand &MO2) {
  return MO1.isIdenticalTo(MO2) && (!MO1.isReg() || !MO1.getReg().isPhysical());
}

#ifndef NDEBUG
static bool isValidDispOp(const MachineOperand &MO) {
  return MO.isImm() || MO.isCPI() || MO.isJTI() || MO.isSymbol() ||
         MO.isGlobal() || MO.isBlockAddress() || MO.isMCSymbol() || MO.isMBB();
}
#endif

static bool isSimilarDispOp(const MachineOperand &MO1,
                            const MachineOperand &MO2) {
  assert(isValidDispOp(MO1) && isValidDispOp(MO2) &&
         "Address displacement operand is not valid");
  return (MO1.isImm() && MO2.isImm()) ||
         (MO1.isCPI() && MO2.isCPI() && MO1.getIndex() == MO2.getIndex()) ||
         (MO1.isJTI() && MO2.isJTI() && MO1.getIndex() == MO2.getIndex()) ||
         (MO1.isSymbol() && MO2.isSymbol() &&
          MO1.getSymbolName() == MO2.getSymbolName()) ||
         (MO1.isGlobal() && MO2.isGlobal() &&
          MO1.getGlobal() == MO2.getGlobal()) ||
         (MO1.isBlockAddress() && MO2.isBlockAddress() &&
          MO1.getBlockAddress() == MO2.getBlockAddress()) ||
         (MO1.isMCSymbol() && MO2.isMCSymbol() &&
          MO1.getMCSymbol() == MO2.getMCSymbol()) ||
         (MO1.isMBB() && MO2.isMBB() && MO1.getMBB() == MO2.getMBB());
}

static inline bool isLEA(const MachineInstr &MI) {
  unsigned Opcode = MI.getOpcode();
  return Opcode == X86::LEA16r || Opcode == X86::LEA32r ||
         Opcode == X86::LEA64r || Opcode == X86::LEA64_32r;
}

namespace {

class X86OptimizeLEAsImpl {
public:
  bool runOnMachineFunction(MachineFunction &MF, ProfileSummaryInfo *PSI,
                            MachineBlockFrequencyInfo *MBFI);

private:
  using MemOpMap = DenseMap<MemOpKey, SmallVector<MachineInstr *, 16>>;

  /// Returns a distance between two instructions inside one basic block.
  /// Negative result means, that instructions occur in reverse order.
  int calcInstrDist(const MachineInstr &First, const MachineInstr &Last);

```
**EN:** This section implements preprocessor directives and structural setup for the core X86 backend. Key symbols include X86OptimizeLEAsImpl, isIdenticalOp. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的预处理指令与结构初始化。关键符号包括 X86OptimizeLEAsImpl, isIdenticalOp。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 251-300: Type declarations and shared abstractions / 类型声明与共享抽象
```cpp
  /// Choose the best \p LEA instruction from the \p List to replace
  /// address calculation in \p MI instruction. Return the address displacement
  /// and the distance between \p MI and the chosen \p BestLEA in
  /// \p AddrDispShift and \p Dist.
  bool chooseBestLEA(const SmallVectorImpl<MachineInstr *> &List,
                     const MachineInstr &MI, MachineInstr *&BestLEA,
                     int64_t &AddrDispShift, int &Dist);

  /// Returns the difference between addresses' displacements of \p MI1
  /// and \p MI2. The numbers of the first memory operands for the instructions
  /// are specified through \p N1 and \p N2.
  int64_t getAddrDispShift(const MachineInstr &MI1, unsigned N1,
                           const MachineInstr &MI2, unsigned N2) const;

  /// Returns true if the \p Last LEA instruction can be replaced by the
  /// \p First. The difference between displacements of the addresses calculated
  /// by these LEAs is returned in \p AddrDispShift. It'll be used for proper
  /// replacement of the \p Last LEA's uses with the \p First's def register.
  bool isReplaceable(const MachineInstr &First, const MachineInstr &Last,
                     int64_t &AddrDispShift) const;

  /// Find all LEA instructions in the basic block. Also, assign position
  /// numbers to all instructions in the basic block to speed up calculation of
  /// distance between them.
  void findLEAs(const MachineBasicBlock &MBB, MemOpMap &LEAs);

  /// Removes redundant address calculations.
  bool removeRedundantAddrCalc(MemOpMap &LEAs);

  /// Replace debug value MI with a new debug value instruction using register
  /// VReg with an appropriate offset and DIExpression to incorporate the
  /// address displacement AddrDispShift. Return new debug value instruction.
  MachineInstr *replaceDebugValue(MachineInstr &MI, Register OldReg,
                                  Register NewReg, int64_t AddrDispShift);

  /// Removes LEAs which calculate similar addresses.
  bool removeRedundantLEAs(MemOpMap &LEAs);

  DenseMap<const MachineInstr *, unsigned> InstrPos;

  MachineRegisterInfo *MRI = nullptr;
  const X86InstrInfo *TII = nullptr;
  const X86RegisterInfo *TRI = nullptr;
};

class X86OptimizeLEAsLegacy : public MachineFunctionPass {
public:
  X86OptimizeLEAsLegacy() : MachineFunctionPass(ID) {}

  StringRef getPassName() const override { return "X86 LEA Optimize"; }
```
**EN:** This section implements type declarations and shared abstractions for the core X86 backend. Key symbols include X86OptimizeLEAsLegacy. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的类型声明与共享抽象。关键符号包括 X86OptimizeLEAsLegacy。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 301-350: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp

  /// Loop over all of the basic blocks, replacing address
  /// calculations in load and store instructions, if it's already
  /// been calculated by LEA. Also, remove redundant LEAs.
  bool runOnMachineFunction(MachineFunction &MF) override;

  static char ID;

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.addRequired<ProfileSummaryInfoWrapperPass>();
    AU.addRequired<LazyMachineBlockFrequencyInfoPass>();
    MachineFunctionPass::getAnalysisUsage(AU);
  }
};

} // end anonymous namespace

char X86OptimizeLEAsLegacy::ID = 0;

FunctionPass *llvm::createX86OptimizeLEAsLegacyPass() {
  return new X86OptimizeLEAsLegacy();
}
INITIALIZE_PASS(X86OptimizeLEAsLegacy, DEBUG_TYPE, "X86 optimize LEA pass",
                false, false)

int X86OptimizeLEAsImpl::calcInstrDist(const MachineInstr &First,
                                       const MachineInstr &Last) {
  // Both instructions must be in the same basic block and they must be
  // presented in InstrPos.
  assert(Last.getParent() == First.getParent() &&
         "Instructions are in different basic blocks");
  assert(InstrPos.contains(&First) && InstrPos.contains(&Last) &&
         "Instructions' positions are undefined");

  return InstrPos[&Last] - InstrPos[&First];
}

// Find the best LEA instruction in the List to replace address recalculation in
// MI. Such LEA must meet these requirements:
// 1) The address calculated by the LEA differs only by the displacement from
//    the address used in MI.
// 2) The register class of the definition of the LEA is compatible with the
//    register class of the address base register of MI.
// 3) Displacement of the new memory operand should fit in 1 byte if possible.
// 4) The LEA should be as close to MI as possible, and prior to it if
//    possible.
bool X86OptimizeLEAsImpl::chooseBestLEA(
    const SmallVectorImpl<MachineInstr *> &List, const MachineInstr &MI,
    MachineInstr *&BestLEA, int64_t &AddrDispShift, int &Dist) {
  const MCInstrDesc &Desc = MI.getDesc();
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 351-400: Control flow, matching, and data updates / 控制流、匹配与数据更新
```cpp
  int MemOpNo = X86II::getMemoryOperandNo(Desc.TSFlags) +
                X86II::getOperandBias(Desc);

  BestLEA = nullptr;

  // Loop over all LEA instructions.
  for (auto *DefMI : List) {
    // Get new address displacement.
    int64_t AddrDispShiftTemp = getAddrDispShift(MI, MemOpNo, *DefMI, 1);

    // Make sure address displacement fits 4 bytes.
    if (!isInt<32>(AddrDispShiftTemp))
      continue;

    // Check that LEA def register can be used as MI address base. Some
    // instructions can use a limited set of registers as address base, for
    // example MOV8mr_NOREX. We could constrain the register class of the LEA
    // def to suit MI, however since this case is very rare and hard to
    // reproduce in a test it's just more reliable to skip the LEA.
    if (TII->getRegClass(Desc, MemOpNo + X86::AddrBaseReg) !=
        MRI->getRegClass(DefMI->getOperand(0).getReg()))
      continue;

    // Choose the closest LEA instruction from the list, prior to MI if
    // possible. Note that we took into account resulting address displacement
    // as well. Also note that the list is sorted by the order in which the LEAs
    // occur, so the break condition is pretty simple.
    int DistTemp = calcInstrDist(*DefMI, MI);
    assert(DistTemp != 0 &&
           "The distance between two different instructions cannot be zero");
    if (DistTemp > 0 || BestLEA == nullptr) {
      // Do not update return LEA, if the current one provides a displacement
      // which fits in 1 byte, while the new candidate does not.
      if (BestLEA != nullptr && !isInt<8>(AddrDispShiftTemp) &&
          isInt<8>(AddrDispShift))
        continue;

      BestLEA = DefMI;
      AddrDispShift = AddrDispShiftTemp;
      Dist = DistTemp;
    }

    // FIXME: Maybe we should not always stop at the first LEA after MI.
    if (DistTemp < 0)
      break;
  }

  return BestLEA != nullptr;
}

```
**EN:** This section implements control flow, matching, and data updates for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的控制流、匹配与数据更新。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 401-450: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
// Get the difference between the addresses' displacements of the two
// instructions \p MI1 and \p MI2. The numbers of the first memory operands are
// passed through \p N1 and \p N2.
int64_t X86OptimizeLEAsImpl::getAddrDispShift(const MachineInstr &MI1,
                                              unsigned N1,
                                              const MachineInstr &MI2,
                                              unsigned N2) const {
  const MachineOperand &Op1 = MI1.getOperand(N1 + X86::AddrDisp);
  const MachineOperand &Op2 = MI2.getOperand(N2 + X86::AddrDisp);

  assert(isSimilarDispOp(Op1, Op2) &&
         "Address displacement operands are not compatible");

  // After the assert above we can be sure that both operands are of the same
  // valid type and use the same symbol/index/address, thus displacement shift
  // calculation is rather simple.
  if (Op1.isJTI())
    return 0;
  return Op1.isImm() ? Op1.getImm() - Op2.getImm()
                     : Op1.getOffset() - Op2.getOffset();
}

// Check that the Last LEA can be replaced by the First LEA. To be so,
// these requirements must be met:
// 1) Addresses calculated by LEAs differ only by displacement.
// 2) Def registers of LEAs belong to the same class.
// 3) All uses of the Last LEA def register are replaceable, thus the
//    register is used only as address base.
bool X86OptimizeLEAsImpl::isReplaceable(const MachineInstr &First,
                                        const MachineInstr &Last,
                                        int64_t &AddrDispShift) const {
  assert(isLEA(First) && isLEA(Last) &&
         "The function works only with LEA instructions");

  // Make sure that LEA def registers belong to the same class. There may be
  // instructions (like MOV8mr_NOREX) which allow a limited set of registers to
  // be used as their operands, so we must be sure that replacing one LEA
  // with another won't lead to putting a wrong register in the instruction.
  if (MRI->getRegClass(First.getOperand(0).getReg()) !=
      MRI->getRegClass(Last.getOperand(0).getReg()))
    return false;

  // Get new address displacement.
  AddrDispShift = getAddrDispShift(Last, 1, First, 1);

  // Loop over all uses of the Last LEA to check that its def register is
  // used only as address base for memory accesses. If so, it can be
  // replaced, otherwise - no.
  for (auto &MO : MRI->use_nodbg_operands(Last.getOperand(0).getReg())) {
    MachineInstr &MI = *MO.getParent();
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 451-500: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp

    // Get the number of the first memory operand.
    const MCInstrDesc &Desc = MI.getDesc();
    int MemOpNo = X86II::getMemoryOperandNo(Desc.TSFlags);

    // If the use instruction has no memory operand - the LEA is not
    // replaceable.
    if (MemOpNo < 0)
      return false;

    MemOpNo += X86II::getOperandBias(Desc);

    // If the address base of the use instruction is not the LEA def register -
    // the LEA is not replaceable.
    if (!isIdenticalOp(MI.getOperand(MemOpNo + X86::AddrBaseReg), MO))
      return false;

    // If the LEA def register is used as any other operand of the use
    // instruction - the LEA is not replaceable.
    for (unsigned i = 0; i < MI.getNumOperands(); i++)
      if (i != (unsigned)(MemOpNo + X86::AddrBaseReg) &&
          isIdenticalOp(MI.getOperand(i), MO))
        return false;

    // Check that the new address displacement will fit 4 bytes.
    if (MI.getOperand(MemOpNo + X86::AddrDisp).isImm() &&
        !isInt<32>(MI.getOperand(MemOpNo + X86::AddrDisp).getImm() +
                   AddrDispShift))
      return false;
  }

  return true;
}

void X86OptimizeLEAsImpl::findLEAs(const MachineBasicBlock &MBB,
                                   MemOpMap &LEAs) {
  unsigned Pos = 0;
  for (auto &MI : MBB) {
    // Assign the position number to the instruction. Note that we are going to
    // move some instructions during the optimization however there will never
    // be a need to move two instructions before any selected instruction. So to
    // avoid multiple positions' updates during moves we just increase position
    // counter by two leaving a free space for instructions which will be moved.
    InstrPos[&MI] = Pos += 2;

    if (isLEA(MI))
      LEAs[getMemOpKey(MI, 1)].push_back(const_cast<MachineInstr *>(&MI));
  }
}

```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 501-550: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
// Try to find load and store instructions which recalculate addresses already
// calculated by some LEA and replace their memory operands with its def
// register.
bool X86OptimizeLEAsImpl::removeRedundantAddrCalc(MemOpMap &LEAs) {
  bool Changed = false;

  assert(!LEAs.empty());
  MachineBasicBlock *MBB = (*LEAs.begin()->second.begin())->getParent();

  // Process all instructions in basic block.
  for (MachineInstr &MI : llvm::make_early_inc_range(*MBB)) {
    // Instruction must be load or store.
    if (!MI.mayLoadOrStore())
      continue;

    // Get the number of the first memory operand.
    const MCInstrDesc &Desc = MI.getDesc();
    int MemOpNo = X86II::getMemoryOperandNo(Desc.TSFlags);

    // If instruction has no memory operand - skip it.
    if (MemOpNo < 0)
      continue;

    MemOpNo += X86II::getOperandBias(Desc);

    // Do not call chooseBestLEA if there was no matching LEA
    auto Insns = LEAs.find(getMemOpKey(MI, MemOpNo));
    if (Insns == LEAs.end())
      continue;

    // Get the best LEA instruction to replace address calculation.
    MachineInstr *DefMI;
    int64_t AddrDispShift;
    int Dist;
    if (!chooseBestLEA(Insns->second, MI, DefMI, AddrDispShift, Dist))
      continue;

    // If LEA occurs before current instruction, we can freely replace
    // the instruction. If LEA occurs after, we can lift LEA above the
    // instruction and this way to be able to replace it. Since LEA and the
    // instruction have similar memory operands (thus, the same def
    // instructions for these operands), we can always do that, without
    // worries of using registers before their defs.
    if (Dist < 0) {
      DefMI->removeFromParent();
      MBB->insert(MachineBasicBlock::iterator(&MI), DefMI);
      InstrPos[DefMI] = InstrPos[&MI] - 1;

      // Make sure the instructions' position numbers are sane.
      assert(((InstrPos[DefMI] == 1 &&
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 551-600: Control flow, matching, and data updates / 控制流、匹配与数据更新
```cpp
               MachineBasicBlock::iterator(DefMI) == MBB->begin()) ||
              InstrPos[DefMI] >
                  InstrPos[&*std::prev(MachineBasicBlock::iterator(DefMI))]) &&
             "Instruction positioning is broken");
    }

    // Since we can possibly extend register lifetime, clear kill flags.
    MRI->clearKillFlags(DefMI->getOperand(0).getReg());

    ++NumSubstLEAs;
    LLVM_DEBUG(dbgs() << "OptimizeLEAs: Candidate to replace: "; MI.dump(););

    // Change instruction operands.
    MI.getOperand(MemOpNo + X86::AddrBaseReg)
        .ChangeToRegister(DefMI->getOperand(0).getReg(), false);
    MI.getOperand(MemOpNo + X86::AddrScaleAmt).ChangeToImmediate(1);
    MI.getOperand(MemOpNo + X86::AddrIndexReg)
        .ChangeToRegister(X86::NoRegister, false);
    MI.getOperand(MemOpNo + X86::AddrDisp).ChangeToImmediate(AddrDispShift);
    MI.getOperand(MemOpNo + X86::AddrSegmentReg)
        .ChangeToRegister(X86::NoRegister, false);

    LLVM_DEBUG(dbgs() << "OptimizeLEAs: Replaced by: "; MI.dump(););

    Changed = true;
  }

  return Changed;
}

MachineInstr *X86OptimizeLEAsImpl::replaceDebugValue(MachineInstr &MI,
                                                     Register OldReg,
                                                     Register NewReg,
                                                     int64_t AddrDispShift) {
  const DIExpression *Expr = MI.getDebugExpression();
  if (AddrDispShift != 0) {
    if (MI.isNonListDebugValue()) {
      Expr =
          DIExpression::prepend(Expr, DIExpression::StackValue, AddrDispShift);
    } else {
      // Update the Expression, appending an offset of `AddrDispShift` to the
      // Op corresponding to `OldReg`.
      SmallVector<uint64_t, 3> Ops;
      DIExpression::appendOffset(Ops, AddrDispShift);
      for (MachineOperand &Op : MI.getDebugOperandsForReg(OldReg)) {
        unsigned OpIdx = MI.getDebugOperandIndex(&Op);
        Expr = DIExpression::appendOpsToArg(Expr, Ops, OpIdx);
      }
    }
  }
```
**EN:** This section implements control flow, matching, and data updates for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的控制流、匹配与数据更新。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 601-650: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp

  // Replace DBG_VALUE instruction with modified version.
  MachineBasicBlock *MBB = MI.getParent();
  DebugLoc DL = MI.getDebugLoc();
  bool IsIndirect = MI.isIndirectDebugValue();
  const MDNode *Var = MI.getDebugVariable();
  unsigned Opcode = MI.isNonListDebugValue() ? TargetOpcode::DBG_VALUE
                                             : TargetOpcode::DBG_VALUE_LIST;
  if (IsIndirect)
    assert(MI.getDebugOffset().getImm() == 0 &&
           "DBG_VALUE with nonzero offset");
  SmallVector<MachineOperand, 4> NewOps;
  // If we encounter an operand using the old register, replace it with an
  // operand that uses the new register; otherwise keep the old operand.
  auto replaceOldReg = [OldReg, NewReg](const MachineOperand &Op) {
    if (Op.isReg() && Op.getReg() == OldReg)
      return MachineOperand::CreateReg(NewReg, false, false, false, false,
                                       false, false, false, false, false,
                                       /*IsRenamable*/ true);
    return Op;
  };
  for (const MachineOperand &Op : MI.debug_operands())
    NewOps.push_back(replaceOldReg(Op));
  return BuildMI(*MBB, MBB->erase(&MI), DL, TII->get(Opcode), IsIndirect,
                 NewOps, Var, Expr);
}

// Try to find similar LEAs in the list and replace one with another.
bool X86OptimizeLEAsImpl::removeRedundantLEAs(MemOpMap &LEAs) {
  bool Changed = false;

  // Loop over all entries in the table.
  for (auto &E : LEAs) {
    auto &List = E.second;

    // Loop over all LEA pairs.
    auto I1 = List.begin();
    while (I1 != List.end()) {
      MachineInstr &First = **I1;
      auto I2 = std::next(I1);
      while (I2 != List.end()) {
        MachineInstr &Last = **I2;
        int64_t AddrDispShift;

        // LEAs should be in occurrence order in the list, so we can freely
        // replace later LEAs with earlier ones.
        assert(calcInstrDist(First, Last) > 0 &&
               "LEAs must be in occurrence order in the list");

        // Check that the Last LEA instruction can be replaced by the First.
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 651-700: Control flow, matching, and data updates / 控制流、匹配与数据更新
```cpp
        if (!isReplaceable(First, Last, AddrDispShift)) {
          ++I2;
          continue;
        }

        // Loop over all uses of the Last LEA and update their operands. Note
        // that the correctness of this has already been checked in the
        // isReplaceable function.
        Register FirstVReg = First.getOperand(0).getReg();
        Register LastVReg = Last.getOperand(0).getReg();
        // We use MRI->use_empty here instead of the combination of
        // llvm::make_early_inc_range and MRI->use_operands because we could
        // replace two or more uses in a debug instruction in one iteration, and
        // that would deeply confuse llvm::make_early_inc_range.
        while (!MRI->use_empty(LastVReg)) {
          MachineOperand &MO = *MRI->use_begin(LastVReg);
          MachineInstr &MI = *MO.getParent();

          if (MI.isDebugValue()) {
            // Replace DBG_VALUE instruction with modified version using the
            // register from the replacing LEA and the address displacement
            // between the LEA instructions.
            replaceDebugValue(MI, LastVReg, FirstVReg, AddrDispShift);
            continue;
          }

          // Get the number of the first memory operand.
          const MCInstrDesc &Desc = MI.getDesc();
          int MemOpNo =
              X86II::getMemoryOperandNo(Desc.TSFlags) +
              X86II::getOperandBias(Desc);

          // Update address base.
          MO.setReg(FirstVReg);

          // Update address disp.
          MachineOperand &Op = MI.getOperand(MemOpNo + X86::AddrDisp);
          if (Op.isImm())
            Op.setImm(Op.getImm() + AddrDispShift);
          else if (!Op.isJTI())
            Op.setOffset(Op.getOffset() + AddrDispShift);
        }

        // Since we can possibly extend register lifetime, clear kill flags.
        MRI->clearKillFlags(FirstVReg);

        ++NumRedundantLEAs;
        LLVM_DEBUG(dbgs() << "OptimizeLEAs: Remove redundant LEA: ";
                   Last.dump(););

```
**EN:** This section implements control flow, matching, and data updates for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的控制流、匹配与数据更新。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 701-750: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
        // By this moment, all of the Last LEA's uses must be replaced. So we
        // can freely remove it.
        assert(MRI->use_empty(LastVReg) &&
               "The LEA's def register must have no uses");
        Last.eraseFromParent();

        // Erase removed LEA from the list.
        I2 = List.erase(I2);

        Changed = true;
      }
      ++I1;
    }
  }

  return Changed;
}

bool X86OptimizeLEAsImpl::runOnMachineFunction(
    MachineFunction &MF, ProfileSummaryInfo *PSI,
    MachineBlockFrequencyInfo *MBFI) {
  bool Changed = false;

  if (DisableX86LEAOpt)
    return false;

  MRI = &MF.getRegInfo();
  TII = MF.getSubtarget<X86Subtarget>().getInstrInfo();
  TRI = MF.getSubtarget<X86Subtarget>().getRegisterInfo();

  // Process all basic blocks.
  for (auto &MBB : MF) {
    MemOpMap LEAs;
    InstrPos.clear();

    // Find all LEA instructions in basic block.
    findLEAs(MBB, LEAs);

    // If current basic block has no LEAs, move on to the next one.
    if (LEAs.empty())
      continue;

    // Remove redundant LEA instructions.
    Changed |= removeRedundantLEAs(LEAs);

    // Remove redundant address calculations. Do it only for -Os/-Oz since only
    // a code size gain is expected from this part of the pass.
    if (llvm::shouldOptimizeForSize(&MBB, PSI, MBFI))
      Changed |= removeRedundantAddrCalc(LEAs);
  }
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 751-784: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp

  return Changed;
}

bool X86OptimizeLEAsLegacy::runOnMachineFunction(MachineFunction &MF) {
  if (skipFunction(MF.getFunction()))
    return false;
  ProfileSummaryInfo *PSI =
      &getAnalysis<ProfileSummaryInfoWrapperPass>().getPSI();
  MachineBlockFrequencyInfo *MBFI =
      (PSI && PSI->hasProfileSummary())
          ? &getAnalysis<LazyMachineBlockFrequencyInfoPass>().getBFI()
          : nullptr;
  X86OptimizeLEAsImpl PassImpl;
  return PassImpl.runOnMachineFunction(MF, PSI, MBFI);
}

PreservedAnalyses
X86OptimizeLEAsPass::run(MachineFunction &MF,
                         MachineFunctionAnalysisManager &MFAM) {
  ProfileSummaryInfo *PSI =
      MFAM.getResult<ModuleAnalysisManagerMachineFunctionProxy>(MF)
          .getCachedResult<ProfileSummaryAnalysis>(
              *MF.getFunction().getParent());
  MachineBlockFrequencyInfo *MBFI =
      (PSI && PSI->hasProfileSummary())
          ? &MFAM.getResult<MachineBlockFrequencyAnalysis>(MF)
          : nullptr;
  X86OptimizeLEAsImpl PassImpl;
  bool Changed = PassImpl.runOnMachineFunction(MF, PSI, MBFI);
  if (!Changed)
    return PreservedAnalyses::all();
  return getMachineFunctionPassPreservedAnalyses().preserveSet<CFGAnalyses>();
}
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

## Key Concepts / 关键概念
- Primary topic: LEA optimization. / 核心主题：LEA 优化。
- Subsystem: the core X86 backend. / 所属子系统：X86 后端核心。
- Notable symbols: MemOpKey, X86OptimizeLEAsImpl, X86OptimizeLEAsLegacy. / 重要符号：MemOpKey, X86OptimizeLEAsImpl, X86OptimizeLEAsLegacy。
- The file emphasizes executable backend logic and target-specific decision making. / 该文件以可执行后端逻辑和目标特定决策为主。
- The implementation follows LLVM namespace conventions. / 实现遵循 LLVM 的命名空间约定。

## Dependencies / 依赖关系
- Direct includes: MCTargetDesc/X86BaseInfo.h, X86.h, X86InstrInfo.h, X86Subtarget.h, llvm/ADT/DenseMap.h, llvm/ADT/DenseMapInfo.h, llvm/ADT/Hashing.h, llvm/ADT/SmallVector.h, llvm/ADT/Statistic.h, llvm/Analysis/ProfileSummaryInfo.h. / 直接包含：MCTargetDesc/X86BaseInfo.h, X86.h, X86InstrInfo.h, X86Subtarget.h, llvm/ADT/DenseMap.h, llvm/ADT/DenseMapInfo.h, llvm/ADT/Hashing.h, llvm/ADT/SmallVector.h, llvm/ADT/Statistic.h, llvm/Analysis/ProfileSummaryInfo.h。
- Additional include dependencies: 23 more headers. / 额外包含依赖：还有 23 个头文件。
- Integrates with shared LLVM backend layers such as IR, CodeGen, MC, and X86 target support as needed. / 按需与 LLVM 的 IR、CodeGen、MC 和 X86 目标支持层集成。
