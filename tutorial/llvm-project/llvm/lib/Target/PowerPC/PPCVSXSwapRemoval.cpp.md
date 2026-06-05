# PPCVSXSwapRemoval.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCVSXSwapRemoval.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file provides backend implementation logic for the PowerPC backend.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCVSXSwapRemoval.cpp`，主要负责 PowerPC 后端的后端实现逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===----------- PPCVSXSwapRemoval.cpp - Remove VSX LE Swaps -------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-41

```cpp
//===---------------------------------------------------------------------===//
//
// This pass analyzes vector computations and removes unnecessary
// doubleword swaps (xxswapd instructions).  This pass is performed
// only for little-endian VSX code generation.
//
// For this specific case, loads and stores of v4i32, v4f32, v2i64,
// and v2f64 vectors are inefficient.  These are implemented using
// the lxvd2x and stxvd2x instructions, which invert the order of
// doublewords in a vector register.  Thus code generation inserts
// an xxswapd after each such load, and prior to each such store.
//
// The extra xxswapd instructions reduce performance.  The purpose
// of this pass is to reduce the number of xxswapd instructions
// required for correctness.
//
// The primary insight is that much code that operates on vectors
// does not care about the relative order of elements in a register,
// so long as the correct memory order is preserved.  If we have a
// computation where all input values are provided by lxvd2x/xxswapd,
// all outputs are stored using xxswapd/lxvd2x, and all intermediate
// computations are lane-insensitive (independent of element order),
// then all the xxswapd instructions associated with the loads and
// stores may be removed without changing observable semantics.
//
// This pass uses standard equivalence class infrastructure to create
// maximal webs of computations fitting the above description.  Each
// such web is then optimized by removing its unnecessary xxswapd
// instructions.
//
// There are some lane-sensitive operations for which we can still
// permit the optimization, provided we modify those operations
// accordingly.  Such operations are identified as using "special
// handling" within this module.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This pass analyzes vector computations and removes unnecessary". Notable symbols in this range include `swaps`, `insensitive`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This pass analyzes vector computations and removes unnecessary”。 该区间中较显眼的符号包括 `swaps`, `insensitive`。

### Lines 42-48

```cpp
//===---------------------------------------------------------------------===//

#include "PPC.h"
#include "PPCInstrInfo.h"
#include "PPCTargetMachine.h"
#include "llvm/ADT/DenseMap.h"
#include "llvm/ADT/EquivalenceClasses.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 49-58

```cpp
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/Config/llvm-config.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/Format.h"
#include "llvm/Support/raw_ostream.h"

using namespace llvm;
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 59-83

```cpp
#define DEBUG_TYPE "ppc-vsx-swaps"

namespace {

// A PPCVSXSwapEntry is created for each machine instruction that
// is relevant to a vector computation.
struct PPCVSXSwapEntry {
  // Pointer to the instruction.
  MachineInstr *VSEMI;

  // Unique ID (position in the swap vector).
  int VSEId;

  // Attributes of this node.
  unsigned int IsLoad : 1;
  unsigned int IsStore : 1;
  unsigned int IsSwap : 1;
  unsigned int MentionsPhysVR : 1;
  unsigned int IsSwappable : 1;
  unsigned int MentionsPartialVR : 1;
  unsigned int SpecialHandling : 3;
  unsigned int WebRejected : 1;
  unsigned int WillRemove : 1;
};
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 84-94

```cpp
enum SHValues {
  SH_NONE = 0,
  SH_EXTRACT,
  SH_INSERT,
  SH_NOSWAP_LD,
  SH_NOSWAP_ST,
  SH_SPLAT,
  SH_XXPERMDI,
  SH_COPYWIDEN
};
```
- **EN**: Introduces enumerated values such as `SHValues` to make target-specific cases explicit and type-safe.
- **CN**: 这里引入枚举值，例如 `SHValues`，用更显式且更安全的方式表达目标相关分支。

### Lines 95-132

```cpp
struct PPCVSXSwapRemoval : public MachineFunctionPass {

  static char ID;
  const PPCInstrInfo *TII;
  MachineFunction *MF;
  MachineRegisterInfo *MRI;

  // Swap entries are allocated in a vector for better performance.
  std::vector<PPCVSXSwapEntry> SwapVector;

  // A mapping is maintained between machine instructions and
  // their swap entries.  The key is the address of the MI.
  DenseMap<MachineInstr*, int> SwapMap;

  // Equivalence classes are used to gather webs of related computation.
  // Swap entries are represented by their VSEId fields.
  EquivalenceClasses<int> *EC;

  PPCVSXSwapRemoval() : MachineFunctionPass(ID) {}

private:
  // Initialize data structures.
  void initialize(MachineFunction &MFParm);

  // Walk the machine instructions to gather vector usage information.
  // Return true iff vector mentions are present.
  bool gatherVectorInstructions();

  // Add an entry to the swap vector and swap map.
  int addSwapEntry(MachineInstr *MI, PPCVSXSwapEntry &SwapEntry);

  // Hunt backwards through COPY and SUBREG_TO_REG chains for a
  // source register.  VecIdx indicates the swap vector entry to
  // mark as mentioning a physical register if the search leads
  // to one.
  unsigned lookThruCopyLike(unsigned SrcReg, unsigned VecIdx);

  // Generate equivalence classes for related computations (webs).
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Swap entries are allocated in a vector for better performance.". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Swap entries are allocated in a vector for better performance.”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 133-170

```cpp
  void formWebs();

  // Analyze webs and determine those that cannot be optimized.
  void recordUnoptimizableWebs();

  // Record which swap instructions can be safely removed.
  void markSwapsForRemoval();

  // Remove swaps and update other instructions requiring special
  // handling.  Return true iff any changes are made.
  bool removeSwaps();

  // Insert a swap instruction from SrcReg to DstReg at the given
  // InsertPoint.
  void insertSwap(MachineInstr *MI, MachineBasicBlock::iterator InsertPoint,
                  unsigned DstReg, unsigned SrcReg);

  // Update instructions requiring special handling.
  void handleSpecialSwappables(int EntryIdx);

  // Dump a description of the entries in the swap vector.
  void dumpSwapVector();

  // Return true iff the given register is in the given class.
  bool isRegInClass(unsigned Reg, const TargetRegisterClass *RC) {
    if (Register::isVirtualRegister(Reg))
      return RC->hasSubClassEq(MRI->getRegClass(Reg));
    return RC->contains(Reg);
  }

  // Return true iff the given register is a full vector register.
  bool isVecReg(unsigned Reg) {
    return (isRegInClass(Reg, &PPC::VSRCRegClass) ||
            isRegInClass(Reg, &PPC::VRRCRegClass));
  }

  // Return true iff the given register is a partial vector register.
  bool isScalarVecReg(unsigned Reg) {
```
- **EN**: Implements helper routine(s) `formWebs`, `recordUnoptimizableWebs`, `markSwapsForRemoval` for this portion of the PowerPC backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `formWebs`, `recordUnoptimizableWebs`, `markSwapsForRemoval`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 171-208

```cpp
    return (isRegInClass(Reg, &PPC::VSFRCRegClass) ||
            isRegInClass(Reg, &PPC::VSSRCRegClass));
  }

  // Return true iff the given register mentions all or part of a
  // vector register.  Also sets Partial to true if the mention
  // is for just the floating-point register overlap of the register.
  bool isAnyVecReg(unsigned Reg, bool &Partial) {
    if (isScalarVecReg(Reg))
      Partial = true;
    return isScalarVecReg(Reg) || isVecReg(Reg);
  }

public:
  // Main entry point for this pass.
  bool runOnMachineFunction(MachineFunction &MF) override {
    if (skipFunction(MF.getFunction()))
      return false;

    // If we don't have VSX on the subtarget, don't do anything.
    // Also, on Power 9 the load and store ops preserve element order and so
    // the swaps are not required.
    const PPCSubtarget &STI = MF.getSubtarget<PPCSubtarget>();
    if (!STI.hasVSX() || !STI.needsSwapsForVSXMemOps())
      return false;

    bool Changed = false;
    initialize(MF);

    if (gatherVectorInstructions()) {
      formWebs();
      recordUnoptimizableWebs();
      markSwapsForRemoval();
      Changed = removeSwaps();
    }

    // FIXME: See the allocation of EC in initialize().
    delete EC;
```
- **EN**: Implements helper routine(s) `isRegInClass`, `isAnyVecReg`, `isScalarVecReg` for this portion of the PowerPC backend backend implementation logic. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `isRegInClass`, `isAnyVecReg`, `isScalarVecReg`。 子目标特性裁剪会影响这里的行为。

### Lines 209-246

```cpp
    return Changed;
  }
};
} // end anonymous namespace

// Initialize data structures for this pass.  In particular, clear the
// swap vector and allocate the equivalence class mapping before
// processing each function.
void PPCVSXSwapRemoval::initialize(MachineFunction &MFParm) {
  MF = &MFParm;
  MRI = &MF->getRegInfo();
  TII = MF->getSubtarget<PPCSubtarget>().getInstrInfo();

  // An initial vector size of 256 appears to work well in practice.
  // Small/medium functions with vector content tend not to incur a
  // reallocation at this size.  Three of the vector tests in
  // projects/test-suite reallocate, which seems like a reasonable rate.
  const int InitialVectorSize(256);
  SwapVector.clear();
  SwapVector.reserve(InitialVectorSize);

  // FIXME: Currently we allocate EC each time because we don't have
  // access to the set representation on which to call clear().  Should
  // consider adding a clear() method to the EquivalenceClasses class.
  EC = new EquivalenceClasses<int>;
}

// Create an entry in the swap vector for each instruction that mentions
// a full vector register, recording various characteristics of the
// instructions there.
bool PPCVSXSwapRemoval::gatherVectorInstructions() {
  bool RelevantFunction = false;

  for (MachineBasicBlock &MBB : *MF) {
    for (MachineInstr &MI : MBB) {

      if (MI.isDebugInstr())
        continue;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Initialize data structures for this pass.  In particular, clear the". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Initialize data structures for this pass.  In particular, clear the”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 247-284

```cpp

      bool RelevantInstr = false;
      bool Partial = false;

      for (const MachineOperand &MO : MI.operands()) {
        if (!MO.isReg())
          continue;
        Register Reg = MO.getReg();
        // All operands need to be checked because there are instructions that
        // operate on a partial register and produce a full register (such as
        // XXPERMDIs).
        if (isAnyVecReg(Reg, Partial))
          RelevantInstr = true;
      }

      if (!RelevantInstr)
        continue;

      RelevantFunction = true;

      // Create a SwapEntry initialized to zeros, then fill in the
      // instruction and ID fields before pushing it to the back
      // of the swap vector.
      PPCVSXSwapEntry SwapEntry{};
      int VecIdx = addSwapEntry(&MI, SwapEntry);

      switch(MI.getOpcode()) {
      default:
        // Unless noted otherwise, an instruction is considered
        // safe for the optimization.  There are a large number of
        // such true-SIMD instructions (all vector math, logical,
        // select, compare, etc.).  However, if the instruction
        // mentions a partial vector register and does not have
        // special handling defined, it is not swappable.
        if (Partial)
          SwapVector[VecIdx].MentionsPartialVR = 1;
        else
          SwapVector[VecIdx].IsSwappable = 1;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "All operands need to be checked because there are instructions that". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“All operands need to be checked because there are instructions that”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 285-322

```cpp
        break;
      case PPC::XXPERMDI: {
        // This is a swap if it is of the form XXPERMDI t, s, s, 2.
        // Unfortunately, MachineCSE ignores COPY and SUBREG_TO_REG, so we
        // can also see XXPERMDI t, SUBREG_TO_REG(s), SUBREG_TO_REG(s), 2,
        // for example.  We have to look through chains of COPY and
        // SUBREG_TO_REG to find the real source value for comparison.
        // If the real source value is a physical register, then mark the
        // XXPERMDI as mentioning a physical register.
        int immed = MI.getOperand(3).getImm();
        if (immed == 2) {
          unsigned trueReg1 = lookThruCopyLike(MI.getOperand(1).getReg(),
                                               VecIdx);
          unsigned trueReg2 = lookThruCopyLike(MI.getOperand(2).getReg(),
                                               VecIdx);
          if (trueReg1 == trueReg2)
            SwapVector[VecIdx].IsSwap = 1;
          else {
            // We can still handle these if the two registers are not
            // identical, by adjusting the form of the XXPERMDI.
            SwapVector[VecIdx].IsSwappable = 1;
            SwapVector[VecIdx].SpecialHandling = SHValues::SH_XXPERMDI;
          }
        // This is a doubleword splat if it is of the form
        // XXPERMDI t, s, s, 0 or XXPERMDI t, s, s, 3.  As above we
        // must look through chains of copy-likes to find the source
        // register.  We turn off the marking for mention of a physical
        // register, because splatting it is safe; the optimization
        // will not swap the value in the physical register.  Whether
        // or not the two input registers are identical, we can handle
        // these by adjusting the form of the XXPERMDI.
        } else if (immed == 0 || immed == 3) {

          SwapVector[VecIdx].IsSwappable = 1;
          SwapVector[VecIdx].SpecialHandling = SHValues::SH_XXPERMDI;

          unsigned trueReg1 = lookThruCopyLike(MI.getOperand(1).getReg(),
                                               VecIdx);
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This is a swap if it is of the form XXPERMDI t, s, s, 2.". Notable symbols in this range include `SUBREG_TO_REG`, `getOperand`, `getImm`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This is a swap if it is of the form XXPERMDI t, s, s, 2.”。 该区间中较显眼的符号包括 `SUBREG_TO_REG`, `getOperand`, `getImm`。

### Lines 323-360

```cpp
          unsigned trueReg2 = lookThruCopyLike(MI.getOperand(2).getReg(),
                                               VecIdx);
          if (trueReg1 == trueReg2)
            SwapVector[VecIdx].MentionsPhysVR = 0;

        } else {
          // We can still handle these by adjusting the form of the XXPERMDI.
          SwapVector[VecIdx].IsSwappable = 1;
          SwapVector[VecIdx].SpecialHandling = SHValues::SH_XXPERMDI;
        }
        break;
      }
      case PPC::LVX:
        // Non-permuting loads are currently unsafe.  We can use special
        // handling for this in the future.  By not marking these as
        // IsSwap, we ensure computations containing them will be rejected
        // for now.
        SwapVector[VecIdx].IsLoad = 1;
        break;
      case PPC::LXVD2X:
      case PPC::LXVW4X:
        // Permuting loads are marked as both load and swap, and are
        // safe for optimization.
        SwapVector[VecIdx].IsLoad = 1;
        SwapVector[VecIdx].IsSwap = 1;
        break;
      case PPC::LXSDX:
      case PPC::LXSSPX:
      case PPC::XFLOADf64:
      case PPC::XFLOADf32:
        // A load of a floating-point value into the high-order half of
        // a vector register is safe, provided that we introduce a swap
        // following the load, which will be done by the SUBREG_TO_REG
        // support.  So just mark these as safe.
        SwapVector[VecIdx].IsLoad = 1;
        SwapVector[VecIdx].IsSwappable = 1;
        break;
      case PPC::STVX:
```
- **EN**: Implements helper routine(s) `lookThruCopyLike`, `getOperand`, `getReg` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `lookThruCopyLike`, `getOperand`, `getReg`。

### Lines 361-398

```cpp
        // Non-permuting stores are currently unsafe.  We can use special
        // handling for this in the future.  By not marking these as
        // IsSwap, we ensure computations containing them will be rejected
        // for now.
        SwapVector[VecIdx].IsStore = 1;
        break;
      case PPC::STXVD2X:
      case PPC::STXVW4X:
        // Permuting stores are marked as both store and swap, and are
        // safe for optimization.
        SwapVector[VecIdx].IsStore = 1;
        SwapVector[VecIdx].IsSwap = 1;
        break;
      case PPC::COPY:
        // These are fine provided they are moving between full vector
        // register classes.
        if (isVecReg(MI.getOperand(0).getReg()) &&
            isVecReg(MI.getOperand(1).getReg()))
          SwapVector[VecIdx].IsSwappable = 1;
        // If we have a copy from one scalar floating-point register
        // to another, we can accept this even if it is a physical
        // register.  The only way this gets involved is if it feeds
        // a SUBREG_TO_REG, which is handled by introducing a swap.
        else if (isScalarVecReg(MI.getOperand(0).getReg()) &&
                 isScalarVecReg(MI.getOperand(1).getReg()))
          SwapVector[VecIdx].IsSwappable = 1;
        break;
      case PPC::SUBREG_TO_REG: {
        // These are fine provided they are moving between full vector
        // register classes.  If they are moving from a scalar
        // floating-point class to a vector class, we can handle those
        // as well, provided we introduce a swap.  It is generally the
        // case that we will introduce fewer swaps than we remove, but
        // (FIXME) a cost model could be used.  However, introduced
        // swaps could potentially be CSEd, so this is not trivial.
        if (isVecReg(MI.getOperand(0).getReg()) &&
            isVecReg(MI.getOperand(1).getReg()))
          SwapVector[VecIdx].IsSwappable = 1;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Non-permuting stores are currently unsafe.  We can use special". Notable symbols in this range include `isVecReg`, `getOperand`, `getReg`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Non-permuting stores are currently unsafe.  We can use special”。 该区间中较显眼的符号包括 `isVecReg`, `getOperand`, `getReg`。

### Lines 399-436

```cpp
        else if (isVecReg(MI.getOperand(0).getReg()) &&
                 isScalarVecReg(MI.getOperand(1).getReg())) {
          SwapVector[VecIdx].IsSwappable = 1;
          SwapVector[VecIdx].SpecialHandling = SHValues::SH_COPYWIDEN;
        }
        break;
      }
      case PPC::VSPLTB:
      case PPC::VSPLTH:
      case PPC::VSPLTW:
      case PPC::XXSPLTW:
        // Splats are lane-sensitive, but we can use special handling
        // to adjust the source lane for the splat.
        SwapVector[VecIdx].IsSwappable = 1;
        SwapVector[VecIdx].SpecialHandling = SHValues::SH_SPLAT;
        break;
      // The presence of the following lane-sensitive operations in a
      // web will kill the optimization, at least for now.  For these
      // we do nothing, causing the optimization to fail.
      // FIXME: Some of these could be permitted with special handling,
      // and will be phased in as time permits.
      // FIXME: There is no simple and maintainable way to express a set
      // of opcodes having a common attribute in TableGen.  Should this
      // change, this is a prime candidate to use such a mechanism.
      case PPC::INLINEASM:
      case PPC::INLINEASM_BR:
      case PPC::EXTRACT_SUBREG:
      case PPC::INSERT_SUBREG:
      case PPC::COPY_TO_REGCLASS:
      case PPC::LVEBX:
      case PPC::LVEHX:
      case PPC::LVEWX:
      case PPC::LVSL:
      case PPC::LVSR:
      case PPC::LVXL:
      case PPC::STVEBX:
      case PPC::STVEHX:
      case PPC::STVEWX:
```
- **EN**: Implements helper routine(s) `isVecReg`, `getOperand`, `getReg` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `isVecReg`, `getOperand`, `getReg`。

### Lines 437-474

```cpp
      case PPC::STVXL:
        // We can handle STXSDX and STXSSPX similarly to LXSDX and LXSSPX,
        // by adding special handling for narrowing copies as well as
        // widening ones.  However, I've experimented with this, and in
        // practice we currently do not appear to use STXSDX fed by
        // a narrowing copy from a full vector register.  Since I can't
        // generate any useful test cases, I've left this alone for now.
      case PPC::STXSDX:
      case PPC::STXSSPX:
      case PPC::VCIPHER:
      case PPC::VCIPHERLAST:
      case PPC::VMRGHB:
      case PPC::VMRGHH:
      case PPC::VMRGHW:
      case PPC::VMRGLB:
      case PPC::VMRGLH:
      case PPC::VMRGLW:
      case PPC::VMULESB:
      case PPC::VMULESH:
      case PPC::VMULESW:
      case PPC::VMULEUB:
      case PPC::VMULEUH:
      case PPC::VMULEUW:
      case PPC::VMULOSB:
      case PPC::VMULOSH:
      case PPC::VMULOSW:
      case PPC::VMULOUB:
      case PPC::VMULOUH:
      case PPC::VMULOUW:
      case PPC::VNCIPHER:
      case PPC::VNCIPHERLAST:
      case PPC::VPERM:
      case PPC::VPERMXOR:
      case PPC::VPKPX:
      case PPC::VPKSHSS:
      case PPC::VPKSHUS:
      case PPC::VPKSDSS:
      case PPC::VPKSDUS:
```
- **EN**: Implements dispatch logic that chooses specialized target behavior for different opcodes, modes, or ABI cases.
- **CN**: 这一段实现分派逻辑，为不同操作码、模式或 ABI 情况选择特定的目标行为。

### Lines 475-512

```cpp
      case PPC::VPKSWSS:
      case PPC::VPKSWUS:
      case PPC::VPKUDUM:
      case PPC::VPKUDUS:
      case PPC::VPKUHUM:
      case PPC::VPKUHUS:
      case PPC::VPKUWUM:
      case PPC::VPKUWUS:
      case PPC::VPMSUMB:
      case PPC::VPMSUMD:
      case PPC::VPMSUMH:
      case PPC::VPMSUMW:
      case PPC::VRLB:
      case PPC::VRLD:
      case PPC::VRLH:
      case PPC::VRLW:
      case PPC::VSBOX:
      case PPC::VSHASIGMAD:
      case PPC::VSHASIGMAW:
      case PPC::VSL:
      case PPC::VSLDOI:
      case PPC::VSLO:
      case PPC::VSR:
      case PPC::VSRO:
      case PPC::VSUM2SWS:
      case PPC::VSUM4SBS:
      case PPC::VSUM4SHS:
      case PPC::VSUM4UBS:
      case PPC::VSUMSWS:
      case PPC::VUPKHPX:
      case PPC::VUPKHSB:
      case PPC::VUPKHSH:
      case PPC::VUPKHSW:
      case PPC::VUPKLPX:
      case PPC::VUPKLSB:
      case PPC::VUPKLSH:
      case PPC::VUPKLSW:
      case PPC::XXMRGHW:
```
- **EN**: Implements dispatch logic that chooses specialized target behavior for different opcodes, modes, or ABI cases.
- **CN**: 这一段实现分派逻辑，为不同操作码、模式或 ABI 情况选择特定的目标行为。

### Lines 513-550

```cpp
      case PPC::XXMRGLW:
      // XXSLDWI could be replaced by a general permute with one of three
      // permute control vectors (for shift values 1, 2, 3).  However,
      // VPERM has a more restrictive register class.
      case PPC::XXSLDWI:
      case PPC::XSCVDPSPN:
      case PPC::XSCVSPDPN:
      case PPC::MTVSCR:
      case PPC::MFVSCR:
        break;
      }
    }
  }

  if (RelevantFunction) {
    LLVM_DEBUG(dbgs() << "Swap vector when first built\n\n");
    LLVM_DEBUG(dumpSwapVector());
  }

  return RelevantFunction;
}

// Add an entry to the swap vector and swap map, and make a
// singleton equivalence class for the entry.
int PPCVSXSwapRemoval::addSwapEntry(MachineInstr *MI,
                                  PPCVSXSwapEntry& SwapEntry) {
  SwapEntry.VSEMI = MI;
  SwapEntry.VSEId = SwapVector.size();
  SwapVector.push_back(SwapEntry);
  EC->insert(SwapEntry.VSEId);
  SwapMap[MI] = SwapEntry.VSEId;
  return SwapEntry.VSEId;
}

// This is used to find the "true" source register for an
// XXPERMDI instruction, since MachineCSE does not handle the
// "copy-like" operations (Copy and SubregToReg).  Returns
// the original SrcReg unless it is the target of a copy-like
```
- **EN**: Implements helper routine(s) `vectors`, `dbgs`, `dumpSwapVector` for this portion of the PowerPC backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `vectors`, `dbgs`, `dumpSwapVector`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 551-588

```cpp
// operation, in which case we chain backwards through all
// such operations to the ultimate source register.  If a
// physical register is encountered, we stop the search and
// flag the swap entry indicated by VecIdx (the original
// XXPERMDI) as mentioning a physical register.
unsigned PPCVSXSwapRemoval::lookThruCopyLike(unsigned SrcReg,
                                             unsigned VecIdx) {
  MachineInstr *MI = MRI->getVRegDef(SrcReg);
  if (!MI->isCopyLike())
    return SrcReg;

  assert((MI->isCopy() || MI->isSubregToReg()) &&
         "bad opcode for lookThruCopyLike");
  unsigned CopySrcReg = MI->getOperand(1).getReg();

  if (!Register::isVirtualRegister(CopySrcReg)) {
    if (!isScalarVecReg(CopySrcReg))
      SwapVector[VecIdx].MentionsPhysVR = 1;
    return CopySrcReg;
  }

  return lookThruCopyLike(CopySrcReg, VecIdx);
}

// Generate equivalence classes for related computations (webs) by
// def-use relationships of virtual registers.  Mention of a physical
// register terminates the generation of equivalence classes as this
// indicates a use of a parameter, definition of a return value, use
// of a value returned from a call, or definition of a parameter to a
// call.  Computations with physical register mentions are flagged
// as such so their containing webs will not be optimized.
void PPCVSXSwapRemoval::formWebs() {

  LLVM_DEBUG(dbgs() << "\n*** Forming webs for swap removal ***\n\n");

  for (unsigned EntryIdx = 0; EntryIdx < SwapVector.size(); ++EntryIdx) {

    MachineInstr *MI = SwapVector[EntryIdx].VSEMI;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "operation, in which case we chain backwards through all". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“operation, in which case we chain backwards through all”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 589-626

```cpp

    LLVM_DEBUG(dbgs() << "\n" << SwapVector[EntryIdx].VSEId << " ");
    LLVM_DEBUG(MI->dump());

    // It's sufficient to walk vector uses and join them to their unique
    // definitions.  In addition, check full vector register operands
    // for physical regs.  We exclude partial-vector register operands
    // because we can handle them if copied to a full vector.
    for (const MachineOperand &MO : MI->operands()) {
      if (!MO.isReg())
        continue;

      Register Reg = MO.getReg();
      if (!isVecReg(Reg) && !isScalarVecReg(Reg))
        continue;

      if (!Reg.isVirtual()) {
        if (!(MI->isCopy() && isScalarVecReg(Reg)))
          SwapVector[EntryIdx].MentionsPhysVR = 1;
        continue;
      }

      if (!MO.isUse())
        continue;

      MachineInstr* DefMI = MRI->getVRegDef(Reg);
      assert(SwapMap.contains(DefMI) &&
             "Inconsistency: def of vector reg not found in swap map!");
      int DefIdx = SwapMap[DefMI];
      (void)EC->unionSets(SwapVector[DefIdx].VSEId,
                          SwapVector[EntryIdx].VSEId);

      LLVM_DEBUG(dbgs() << format("Unioning %d with %d\n",
                                  SwapVector[DefIdx].VSEId,
                                  SwapVector[EntryIdx].VSEId));
      LLVM_DEBUG(dbgs() << "  Def: ");
      LLVM_DEBUG(DefMI->dump());
    }
```
- **EN**: Implements helper routine(s) `dbgs`, `dump`, `operands` for this portion of the PowerPC backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `dbgs`, `dump`, `operands`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 627-664

```cpp
  }
}

// Walk the swap vector entries looking for conditions that prevent their
// containing computations from being optimized.  When such conditions are
// found, mark the representative of the computation's equivalence class
// as rejected.
void PPCVSXSwapRemoval::recordUnoptimizableWebs() {

  LLVM_DEBUG(dbgs() << "\n*** Rejecting webs for swap removal ***\n\n");

  for (unsigned EntryIdx = 0; EntryIdx < SwapVector.size(); ++EntryIdx) {
    int Repr = EC->getLeaderValue(SwapVector[EntryIdx].VSEId);

    // If representative is already rejected, don't waste further time.
    if (SwapVector[Repr].WebRejected)
      continue;

    // Reject webs containing mentions of physical or partial registers, or
    // containing operations that we don't know how to handle in a lane-
    // permuted region.
    if (SwapVector[EntryIdx].MentionsPhysVR ||
        SwapVector[EntryIdx].MentionsPartialVR ||
        !(SwapVector[EntryIdx].IsSwappable || SwapVector[EntryIdx].IsSwap)) {

      SwapVector[Repr].WebRejected = 1;

      LLVM_DEBUG(
          dbgs() << format("Web %d rejected for physreg, partial reg, or not "
                           "swap[pable]\n",
                           Repr));
      LLVM_DEBUG(dbgs() << "  in " << EntryIdx << ": ");
      LLVM_DEBUG(SwapVector[EntryIdx].VSEMI->dump());
      LLVM_DEBUG(dbgs() << "\n");
    }

    // Reject webs than contain swapping loads that feed something other
    // than a swap instruction.
```
- **EN**: Implements helper routine(s) `recordUnoptimizableWebs`, `dbgs`, `size` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `recordUnoptimizableWebs`, `dbgs`, `size`。

### Lines 665-702

```cpp
    else if (SwapVector[EntryIdx].IsLoad && SwapVector[EntryIdx].IsSwap) {
      MachineInstr *MI = SwapVector[EntryIdx].VSEMI;
      Register DefReg = MI->getOperand(0).getReg();

      // We skip debug instructions in the analysis.  (Note that debug
      // location information is still maintained by this optimization
      // because it remains on the LXVD2X and STXVD2X instructions after
      // the XXPERMDIs are removed.)
      for (MachineInstr &UseMI : MRI->use_nodbg_instructions(DefReg)) {
        int UseIdx = SwapMap[&UseMI];

        if (!SwapVector[UseIdx].IsSwap || SwapVector[UseIdx].IsLoad ||
            SwapVector[UseIdx].IsStore) {

          SwapVector[Repr].WebRejected = 1;

          LLVM_DEBUG(dbgs() << format(
                         "Web %d rejected for load not feeding swap\n", Repr));
          LLVM_DEBUG(dbgs() << "  def " << EntryIdx << ": ");
          LLVM_DEBUG(MI->dump());
          LLVM_DEBUG(dbgs() << "  use " << UseIdx << ": ");
          LLVM_DEBUG(UseMI.dump());
          LLVM_DEBUG(dbgs() << "\n");
        }

        // It is possible that the load feeds a swap and that swap feeds a
        // store. In such a case, the code is actually trying to store a swapped
        // vector. We must reject such webs.
        if (SwapVector[UseIdx].IsSwap && !SwapVector[UseIdx].IsLoad &&
            !SwapVector[UseIdx].IsStore) {
          Register SwapDefReg = UseMI.getOperand(0).getReg();
          for (MachineInstr &UseOfUseMI :
               MRI->use_nodbg_instructions(SwapDefReg)) {
            int UseOfUseIdx = SwapMap[&UseOfUseMI];
            if (SwapVector[UseOfUseIdx].IsStore) {
              SwapVector[Repr].WebRejected = 1;
              LLVM_DEBUG(
                  dbgs() << format(
```
- **EN**: Implements helper routine(s) `getOperand`, `getReg`, `use_nodbg_instructions` for this portion of the PowerPC backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `getOperand`, `getReg`, `use_nodbg_instructions`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 703-740

```cpp
                      "Web %d rejected for load/swap feeding a store\n", Repr));
              LLVM_DEBUG(dbgs() << "  def " << EntryIdx << ": ");
              LLVM_DEBUG(MI->dump());
              LLVM_DEBUG(dbgs() << "  use " << UseIdx << ": ");
              LLVM_DEBUG(UseMI.dump());
              LLVM_DEBUG(dbgs() << "\n");
            }
          }
        }
      }

    // Reject webs that contain swapping stores that are fed by something
    // other than a swap instruction.
    } else if (SwapVector[EntryIdx].IsStore && SwapVector[EntryIdx].IsSwap) {
      MachineInstr *MI = SwapVector[EntryIdx].VSEMI;
      Register UseReg = MI->getOperand(0).getReg();
      MachineInstr *DefMI = MRI->getVRegDef(UseReg);
      Register DefReg = DefMI->getOperand(0).getReg();
      int DefIdx = SwapMap[DefMI];

      if (!SwapVector[DefIdx].IsSwap || SwapVector[DefIdx].IsLoad ||
          SwapVector[DefIdx].IsStore) {

        SwapVector[Repr].WebRejected = 1;

        LLVM_DEBUG(dbgs() << format(
                       "Web %d rejected for store not fed by swap\n", Repr));
        LLVM_DEBUG(dbgs() << "  def " << DefIdx << ": ");
        LLVM_DEBUG(DefMI->dump());
        LLVM_DEBUG(dbgs() << "  use " << EntryIdx << ": ");
        LLVM_DEBUG(MI->dump());
        LLVM_DEBUG(dbgs() << "\n");
      }

      // Ensure all uses of the register defined by DefMI feed store
      // instructions
      for (MachineInstr &UseMI : MRI->use_nodbg_instructions(DefReg)) {
        int UseIdx = SwapMap[&UseMI];
```
- **EN**: Implements helper routine(s) `dbgs`, `dump`, `getOperand` for this portion of the PowerPC backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `dbgs`, `dump`, `getOperand`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 741-778

```cpp

        if (SwapVector[UseIdx].VSEMI->getOpcode() != MI->getOpcode()) {
          SwapVector[Repr].WebRejected = 1;

          LLVM_DEBUG(
              dbgs() << format(
                  "Web %d rejected for swap not feeding only stores\n", Repr));
          LLVM_DEBUG(dbgs() << "  def "
                            << " : ");
          LLVM_DEBUG(DefMI->dump());
          LLVM_DEBUG(dbgs() << "  use " << UseIdx << ": ");
          LLVM_DEBUG(SwapVector[UseIdx].VSEMI->dump());
          LLVM_DEBUG(dbgs() << "\n");
        }
      }
    }
  }

  LLVM_DEBUG(dbgs() << "Swap vector after web analysis:\n\n");
  LLVM_DEBUG(dumpSwapVector());
}

// Walk the swap vector entries looking for swaps fed by permuting loads
// and swaps that feed permuting stores.  If the containing computation
// has not been marked rejected, mark each such swap for removal.
// (Removal is delayed in case optimization has disturbed the pattern,
// such that multiple loads feed the same swap, etc.)
void PPCVSXSwapRemoval::markSwapsForRemoval() {

  LLVM_DEBUG(dbgs() << "\n*** Marking swaps for removal ***\n\n");

  for (unsigned EntryIdx = 0; EntryIdx < SwapVector.size(); ++EntryIdx) {

    if (SwapVector[EntryIdx].IsLoad && SwapVector[EntryIdx].IsSwap) {
      int Repr = EC->getLeaderValue(SwapVector[EntryIdx].VSEId);

      if (!SwapVector[Repr].WebRejected) {
        MachineInstr *MI = SwapVector[EntryIdx].VSEMI;
```
- **EN**: Implements helper routine(s) `getOpcode`, `dbgs`, `format` for this portion of the PowerPC backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `getOpcode`, `dbgs`, `format`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 779-816

```cpp
        Register DefReg = MI->getOperand(0).getReg();

        for (MachineInstr &UseMI : MRI->use_nodbg_instructions(DefReg)) {
          int UseIdx = SwapMap[&UseMI];
          SwapVector[UseIdx].WillRemove = 1;

          LLVM_DEBUG(dbgs() << "Marking swap fed by load for removal: ");
          LLVM_DEBUG(UseMI.dump());
        }
      }

    } else if (SwapVector[EntryIdx].IsStore && SwapVector[EntryIdx].IsSwap) {
      int Repr = EC->getLeaderValue(SwapVector[EntryIdx].VSEId);

      if (!SwapVector[Repr].WebRejected) {
        MachineInstr *MI = SwapVector[EntryIdx].VSEMI;
        Register UseReg = MI->getOperand(0).getReg();
        MachineInstr *DefMI = MRI->getVRegDef(UseReg);
        int DefIdx = SwapMap[DefMI];
        SwapVector[DefIdx].WillRemove = 1;

        LLVM_DEBUG(dbgs() << "Marking swap feeding store for removal: ");
        LLVM_DEBUG(DefMI->dump());
      }

    } else if (SwapVector[EntryIdx].IsSwappable &&
               SwapVector[EntryIdx].SpecialHandling != 0) {
      int Repr = EC->getLeaderValue(SwapVector[EntryIdx].VSEId);

      if (!SwapVector[Repr].WebRejected)
        handleSpecialSwappables(EntryIdx);
    }
  }
}

// Create an xxswapd instruction and insert it prior to the given point.
// MI is used to determine basic block and debug loc information.
// FIXME: When inserting a swap, we should check whether SrcReg is
```
- **EN**: Implements helper routine(s) `getOperand`, `getReg`, `use_nodbg_instructions` for this portion of the PowerPC backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `getOperand`, `getReg`, `use_nodbg_instructions`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 817-854

```cpp
// defined by another swap:  SrcReg = XXPERMDI Reg, Reg, 2;  If so,
// then instead we should generate a copy from Reg to DstReg.
void PPCVSXSwapRemoval::insertSwap(MachineInstr *MI,
                                   MachineBasicBlock::iterator InsertPoint,
                                   unsigned DstReg, unsigned SrcReg) {
  BuildMI(*MI->getParent(), InsertPoint, MI->getDebugLoc(),
          TII->get(PPC::XXPERMDI), DstReg)
    .addReg(SrcReg)
    .addReg(SrcReg)
    .addImm(2);
}

// The identified swap entry requires special handling to allow its
// containing computation to be optimized.  Perform that handling
// here.
// FIXME: Additional opportunities will be phased in with subsequent
// patches.
void PPCVSXSwapRemoval::handleSpecialSwappables(int EntryIdx) {
  switch (SwapVector[EntryIdx].SpecialHandling) {

  default:
    llvm_unreachable("Unexpected special handling type");

  // For splats based on an index into a vector, add N/2 modulo N
  // to the index, where N is the number of vector elements.
  case SHValues::SH_SPLAT: {
    MachineInstr *MI = SwapVector[EntryIdx].VSEMI;
    unsigned NElts;

    LLVM_DEBUG(dbgs() << "Changing splat: ");
    LLVM_DEBUG(MI->dump());

    switch (MI->getOpcode()) {
    default:
      llvm_unreachable("Unexpected splat opcode");
    case PPC::VSPLTB: NElts = 16; break;
    case PPC::VSPLTH: NElts = 8;  break;
    case PPC::VSPLTW:
```
- **EN**: Implements helper routine(s) `insertSwap`, `BuildMI`, `getParent` for this portion of the PowerPC backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `insertSwap`, `BuildMI`, `getParent`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 855-892

```cpp
    case PPC::XXSPLTW: NElts = 4;  break;
    }

    unsigned EltNo;
    if (MI->getOpcode() == PPC::XXSPLTW)
      EltNo = MI->getOperand(2).getImm();
    else
      EltNo = MI->getOperand(1).getImm();

    EltNo = (EltNo + NElts / 2) % NElts;
    if (MI->getOpcode() == PPC::XXSPLTW)
      MI->getOperand(2).setImm(EltNo);
    else
      MI->getOperand(1).setImm(EltNo);

    LLVM_DEBUG(dbgs() << "  Into: ");
    LLVM_DEBUG(MI->dump());
    break;
  }

  // For an XXPERMDI that isn't handled otherwise, we need to
  // reverse the order of the operands.  If the selector operand
  // has a value of 0 or 3, we need to change it to 3 or 0,
  // respectively.  Otherwise we should leave it alone.  (This
  // is equivalent to reversing the two bits of the selector
  // operand and complementing the result.)
  case SHValues::SH_XXPERMDI: {
    MachineInstr *MI = SwapVector[EntryIdx].VSEMI;

    LLVM_DEBUG(dbgs() << "Changing XXPERMDI: ");
    LLVM_DEBUG(MI->dump());

    unsigned Selector = MI->getOperand(3).getImm();
    if (Selector == 0 || Selector == 3)
      Selector = 3 - Selector;
    MI->getOperand(3).setImm(Selector);

    Register Reg1 = MI->getOperand(1).getReg();
```
- **EN**: Implements helper routine(s) `getOpcode`, `getOperand`, `getImm` for this portion of the PowerPC backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `getOpcode`, `getOperand`, `getImm`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 893-930

```cpp
    Register Reg2 = MI->getOperand(2).getReg();
    MI->getOperand(1).setReg(Reg2);
    MI->getOperand(2).setReg(Reg1);

    // We also need to swap kill flag associated with the register.
    bool IsKill1 = MI->getOperand(1).isKill();
    bool IsKill2 = MI->getOperand(2).isKill();
    MI->getOperand(1).setIsKill(IsKill2);
    MI->getOperand(2).setIsKill(IsKill1);

    LLVM_DEBUG(dbgs() << "  Into: ");
    LLVM_DEBUG(MI->dump());
    break;
  }

  // For a copy from a scalar floating-point register to a vector
  // register, removing swaps will leave the copied value in the
  // wrong lane.  Insert a swap following the copy to fix this.
  case SHValues::SH_COPYWIDEN: {
    MachineInstr *MI = SwapVector[EntryIdx].VSEMI;

    LLVM_DEBUG(dbgs() << "Changing SUBREG_TO_REG: ");
    LLVM_DEBUG(MI->dump());

    Register DstReg = MI->getOperand(0).getReg();
    const TargetRegisterClass *DstRC = MRI->getRegClass(DstReg);
    Register NewVReg = MRI->createVirtualRegister(DstRC);

    MI->getOperand(0).setReg(NewVReg);
    LLVM_DEBUG(dbgs() << "  Into: ");
    LLVM_DEBUG(MI->dump());

    auto InsertPoint = ++MachineBasicBlock::iterator(MI);

    // Note that an XXPERMDI requires a VSRC, so if the SUBREG_TO_REG
    // is copying to a VRRC, we need to be careful to avoid a register
    // assignment problem.  In this case we must copy from VRRC to VSRC
    // prior to the swap, and from VSRC to VRRC following the swap.
```
- **EN**: Implements helper routine(s) `getOperand`, `getReg`, `setReg` for this portion of the PowerPC backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `getOperand`, `getReg`, `setReg`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 931-968

```cpp
    // Coalescing will usually remove all this mess.
    if (DstRC == &PPC::VRRCRegClass) {
      Register VSRCTmp1 = MRI->createVirtualRegister(&PPC::VSRCRegClass);
      Register VSRCTmp2 = MRI->createVirtualRegister(&PPC::VSRCRegClass);

      BuildMI(*MI->getParent(), InsertPoint, MI->getDebugLoc(),
              TII->get(PPC::COPY), VSRCTmp1)
        .addReg(NewVReg);
      LLVM_DEBUG(std::prev(InsertPoint)->dump());

      insertSwap(MI, InsertPoint, VSRCTmp2, VSRCTmp1);
      LLVM_DEBUG(std::prev(InsertPoint)->dump());

      BuildMI(*MI->getParent(), InsertPoint, MI->getDebugLoc(),
              TII->get(PPC::COPY), DstReg)
        .addReg(VSRCTmp2);
      LLVM_DEBUG(std::prev(InsertPoint)->dump());

    } else {
      insertSwap(MI, InsertPoint, DstReg, NewVReg);
      LLVM_DEBUG(std::prev(InsertPoint)->dump());
    }
    break;
  }
  }
}

// Walk the swap vector and replace each entry marked for removal with
// a copy operation.
bool PPCVSXSwapRemoval::removeSwaps() {

  LLVM_DEBUG(dbgs() << "\n*** Removing swaps ***\n\n");

  bool Changed = false;

  for (unsigned EntryIdx = 0; EntryIdx < SwapVector.size(); ++EntryIdx) {
    if (SwapVector[EntryIdx].WillRemove) {
      Changed = true;
```
- **EN**: Implements helper routine(s) `createVirtualRegister`, `BuildMI`, `getParent` for this portion of the PowerPC backend backend implementation logic. Register assignment and register-class constraints matter here.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `createVirtualRegister`, `BuildMI`, `getParent`。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 969-1006

```cpp
      MachineInstr *MI = SwapVector[EntryIdx].VSEMI;
      MachineBasicBlock *MBB = MI->getParent();
      BuildMI(*MBB, MI, MI->getDebugLoc(), TII->get(TargetOpcode::COPY),
              MI->getOperand(0).getReg())
          .add(MI->getOperand(1));

      LLVM_DEBUG(dbgs() << format("Replaced %d with copy: ",
                                  SwapVector[EntryIdx].VSEId));
      LLVM_DEBUG(MI->dump());

      MI->eraseFromParent();
    }
  }

  return Changed;
}

#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
// For debug purposes, dump the contents of the swap vector.
LLVM_DUMP_METHOD void PPCVSXSwapRemoval::dumpSwapVector() {

  for (unsigned EntryIdx = 0; EntryIdx < SwapVector.size(); ++EntryIdx) {

    MachineInstr *MI = SwapVector[EntryIdx].VSEMI;
    int ID = SwapVector[EntryIdx].VSEId;

    dbgs() << format("%6d", ID);
    dbgs() << format("%6d", EC->getLeaderValue(ID));
    dbgs() << format(" %bb.%3d", MI->getParent()->getNumber());
    dbgs() << format("  %14s  ", TII->getName(MI->getOpcode()).str().c_str());

    if (SwapVector[EntryIdx].IsLoad)
      dbgs() << "load ";
    if (SwapVector[EntryIdx].IsStore)
      dbgs() << "store ";
    if (SwapVector[EntryIdx].IsSwap)
      dbgs() << "swap ";
    if (SwapVector[EntryIdx].MentionsPhysVR)
```
- **EN**: Implements helper routine(s) `getParent`, `BuildMI`, `getDebugLoc` for this portion of the PowerPC backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `getParent`, `BuildMI`, `getDebugLoc`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 1007-1044

```cpp
      dbgs() << "physreg ";
    if (SwapVector[EntryIdx].MentionsPartialVR)
      dbgs() << "partialreg ";

    if (SwapVector[EntryIdx].IsSwappable) {
      dbgs() << "swappable ";
      switch(SwapVector[EntryIdx].SpecialHandling) {
      default:
        dbgs() << "special:**unknown**";
        break;
      case SH_NONE:
        break;
      case SH_EXTRACT:
        dbgs() << "special:extract ";
        break;
      case SH_INSERT:
        dbgs() << "special:insert ";
        break;
      case SH_NOSWAP_LD:
        dbgs() << "special:load ";
        break;
      case SH_NOSWAP_ST:
        dbgs() << "special:store ";
        break;
      case SH_SPLAT:
        dbgs() << "special:splat ";
        break;
      case SH_XXPERMDI:
        dbgs() << "special:xxpermdi ";
        break;
      case SH_COPYWIDEN:
        dbgs() << "special:copywiden ";
        break;
      }
    }

    if (SwapVector[EntryIdx].WebRejected)
      dbgs() << "rejected ";
```
- **EN**: Implements helper routine(s) `dbgs` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `dbgs`。

### Lines 1045-1056

```cpp
    if (SwapVector[EntryIdx].WillRemove)
      dbgs() << "remove ";

    dbgs() << "\n";

    // For no-asserts builds.
    (void)MI;
    (void)ID;
  }

  dbgs() << "\n";
}
```
- **EN**: Declares function entry points including `dbgs` that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `dbgs`。

### Lines 1057-1066

```cpp
#endif

INITIALIZE_PASS_BEGIN(PPCVSXSwapRemoval, DEBUG_TYPE,
                      "PowerPC VSX Swap Removal", false, false)
INITIALIZE_PASS_END(PPCVSXSwapRemoval, DEBUG_TYPE,
                    "PowerPC VSX Swap Removal", false, false)

char PPCVSXSwapRemoval::ID = 0;
FunctionPass*
llvm::createPPCVSXSwapRemovalPass() { return new PPCVSXSwapRemoval(); }
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. Notable symbols in this range include `INITIALIZE_PASS_BEGIN`, `INITIALIZE_PASS_END`, `createPPCVSXSwapRemovalPass`.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 该区间中较显眼的符号包括 `INITIALIZE_PASS_BEGIN`, `INITIALIZE_PASS_END`, `createPPCVSXSwapRemovalPass`。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- Backend implementation logic / 后端实现逻辑
- Register modeling / 寄存器建模
- Subtarget features / 子目标特性
- Instruction selection or opcode handling / 指令选择或操作码处理
- Target machine configuration / 目标机器配置
- Declarative TableGen records / 声明式 TableGen 记录
- Vector or SIMD behavior / 向量或 SIMD 行为

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `PPC.h`
- `PPCInstrInfo.h`
- `PPCTargetMachine.h`
- `llvm/ADT/DenseMap.h`
- `llvm/ADT/EquivalenceClasses.h`
- `llvm/CodeGen/MachineFunctionPass.h`
- `llvm/CodeGen/MachineInstrBuilder.h`
- `llvm/CodeGen/MachineRegisterInfo.h`
- `llvm/Config/llvm-config.h`
- `llvm/Support/Debug.h`
- `llvm/Support/Format.h`
- `llvm/Support/raw_ostream.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- LLVM Support utilities / LLVM Support 工具层
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
