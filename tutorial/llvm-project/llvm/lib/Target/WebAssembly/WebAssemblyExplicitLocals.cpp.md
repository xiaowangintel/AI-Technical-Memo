# WebAssemblyExplicitLocals.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyExplicitLocals.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file converts any remaining registers into WebAssembly locals.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyExplicitLocals.cpp`，主要负责 WebAssembly 后端的后端实现逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- WebAssemblyExplicitLocals.cpp - Make Locals Explicit --------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-15

```cpp
//===----------------------------------------------------------------------===//
///
/// \file
/// This file converts any remaining registers into WebAssembly locals.
///
/// After register stackification and register coloring, convert non-stackified
/// registers into locals, inserting explicit local.get and local.set
/// instructions.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 16-22

```cpp
//===----------------------------------------------------------------------===//

#include "MCTargetDesc/WebAssemblyMCTargetDesc.h"
#include "WebAssembly.h"
#include "WebAssemblyDebugValueManager.h"
#include "WebAssemblyMachineFunctionInfo.h"
#include "WebAssemblySubtarget.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 23-31

```cpp
#include "WebAssemblyUtilities.h"
#include "llvm/CodeGen/MachineBlockFrequencyInfo.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/Passes.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/raw_ostream.h"
using namespace llvm;
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 32-69

```cpp
#define DEBUG_TYPE "wasm-explicit-locals"

namespace {
class WebAssemblyExplicitLocals final : public MachineFunctionPass {
  StringRef getPassName() const override {
    return "WebAssembly Explicit Locals";
  }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    AU.addPreserved<MachineBlockFrequencyInfoWrapperPass>();
    MachineFunctionPass::getAnalysisUsage(AU);
  }

  bool runOnMachineFunction(MachineFunction &MF) override;

public:
  static char ID; // Pass identification, replacement for typeid
  WebAssemblyExplicitLocals() : MachineFunctionPass(ID) {}
};
} // end anonymous namespace

char WebAssemblyExplicitLocals::ID = 0;
INITIALIZE_PASS(WebAssemblyExplicitLocals, DEBUG_TYPE,
                "Convert registers to WebAssembly locals", false, false)

FunctionPass *llvm::createWebAssemblyExplicitLocals() {
  return new WebAssemblyExplicitLocals();
}

static void checkFrameBase(WebAssemblyFunctionInfo &MFI, unsigned Local,
                           unsigned Reg) {
  // Mark a local for the frame base vreg.
  if (MFI.isFrameBaseVirtual() && Reg == MFI.getFrameBaseVreg()) {
    LLVM_DEBUG({
      dbgs() << "Allocating local " << Local << "for VReg "
             << Register(Reg).virtRegIndex() << '\n';
    });
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. Notable symbols in this range include `WebAssemblyExplicitLocals`, `getPassName`, `getAnalysisUsage`.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 该区间中较显眼的符号包括 `WebAssemblyExplicitLocals`, `getPassName`, `getAnalysisUsage`。

### Lines 70-107

```cpp
    MFI.setFrameBaseLocal(Local);
  }
}

/// Return a local id number for the given register, assigning it a new one
/// if it doesn't yet have one.
static unsigned getLocalId(DenseMap<unsigned, unsigned> &Reg2Local,
                           WebAssemblyFunctionInfo &MFI, unsigned &CurLocal,
                           unsigned Reg) {
  auto P = Reg2Local.insert(std::make_pair(Reg, CurLocal));
  if (P.second) {
    checkFrameBase(MFI, CurLocal, Reg);
    ++CurLocal;
  }
  return P.first->second;
}

/// Get the appropriate drop opcode for the given register class.
static unsigned getDropOpcode(const TargetRegisterClass *RC) {
  if (RC == &WebAssembly::I32RegClass)
    return WebAssembly::DROP_I32;
  if (RC == &WebAssembly::I64RegClass)
    return WebAssembly::DROP_I64;
  if (RC == &WebAssembly::F32RegClass)
    return WebAssembly::DROP_F32;
  if (RC == &WebAssembly::F64RegClass)
    return WebAssembly::DROP_F64;
  if (RC == &WebAssembly::V128RegClass)
    return WebAssembly::DROP_V128;
  if (RC == &WebAssembly::FUNCREFRegClass)
    return WebAssembly::DROP_FUNCREF;
  if (RC == &WebAssembly::EXTERNREFRegClass)
    return WebAssembly::DROP_EXTERNREF;
  if (RC == &WebAssembly::EXNREFRegClass)
    return WebAssembly::DROP_EXNREF;
  llvm_unreachable("Unexpected register class");
}
```
- **EN**: Implements helper routine(s) `setFrameBaseLocal`, `getLocalId`, `insert` for this portion of the WebAssembly backend backend implementation logic. Register assignment and register-class constraints matter here.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `setFrameBaseLocal`, `getLocalId`, `insert`。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 108-145

```cpp
/// Get the appropriate local.get opcode for the given register class.
static unsigned getLocalGetOpcode(const TargetRegisterClass *RC) {
  if (RC == &WebAssembly::I32RegClass)
    return WebAssembly::LOCAL_GET_I32;
  if (RC == &WebAssembly::I64RegClass)
    return WebAssembly::LOCAL_GET_I64;
  if (RC == &WebAssembly::F32RegClass)
    return WebAssembly::LOCAL_GET_F32;
  if (RC == &WebAssembly::F64RegClass)
    return WebAssembly::LOCAL_GET_F64;
  if (RC == &WebAssembly::V128RegClass)
    return WebAssembly::LOCAL_GET_V128;
  if (RC == &WebAssembly::FUNCREFRegClass)
    return WebAssembly::LOCAL_GET_FUNCREF;
  if (RC == &WebAssembly::EXTERNREFRegClass)
    return WebAssembly::LOCAL_GET_EXTERNREF;
  if (RC == &WebAssembly::EXNREFRegClass)
    return WebAssembly::LOCAL_GET_EXNREF;
  llvm_unreachable("Unexpected register class");
}

/// Get the appropriate local.set opcode for the given register class.
static unsigned getLocalSetOpcode(const TargetRegisterClass *RC) {
  if (RC == &WebAssembly::I32RegClass)
    return WebAssembly::LOCAL_SET_I32;
  if (RC == &WebAssembly::I64RegClass)
    return WebAssembly::LOCAL_SET_I64;
  if (RC == &WebAssembly::F32RegClass)
    return WebAssembly::LOCAL_SET_F32;
  if (RC == &WebAssembly::F64RegClass)
    return WebAssembly::LOCAL_SET_F64;
  if (RC == &WebAssembly::V128RegClass)
    return WebAssembly::LOCAL_SET_V128;
  if (RC == &WebAssembly::FUNCREFRegClass)
    return WebAssembly::LOCAL_SET_FUNCREF;
  if (RC == &WebAssembly::EXTERNREFRegClass)
    return WebAssembly::LOCAL_SET_EXTERNREF;
  if (RC == &WebAssembly::EXNREFRegClass)
```
- **EN**: Implements helper routine(s) `getLocalGetOpcode`, `llvm_unreachable`, `getLocalSetOpcode` for this portion of the WebAssembly backend backend implementation logic. Register assignment and register-class constraints matter here.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `getLocalGetOpcode`, `llvm_unreachable`, `getLocalSetOpcode`。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 146-183

```cpp
    return WebAssembly::LOCAL_SET_EXNREF;
  llvm_unreachable("Unexpected register class");
}

/// Get the appropriate local.tee opcode for the given register class.
static unsigned getLocalTeeOpcode(const TargetRegisterClass *RC) {
  if (RC == &WebAssembly::I32RegClass)
    return WebAssembly::LOCAL_TEE_I32;
  if (RC == &WebAssembly::I64RegClass)
    return WebAssembly::LOCAL_TEE_I64;
  if (RC == &WebAssembly::F32RegClass)
    return WebAssembly::LOCAL_TEE_F32;
  if (RC == &WebAssembly::F64RegClass)
    return WebAssembly::LOCAL_TEE_F64;
  if (RC == &WebAssembly::V128RegClass)
    return WebAssembly::LOCAL_TEE_V128;
  if (RC == &WebAssembly::FUNCREFRegClass)
    return WebAssembly::LOCAL_TEE_FUNCREF;
  if (RC == &WebAssembly::EXTERNREFRegClass)
    return WebAssembly::LOCAL_TEE_EXTERNREF;
  if (RC == &WebAssembly::EXNREFRegClass)
    return WebAssembly::LOCAL_TEE_EXNREF;
  llvm_unreachable("Unexpected register class");
}

/// Get the type associated with the given register class.
static MVT typeForRegClass(const TargetRegisterClass *RC) {
  if (RC == &WebAssembly::I32RegClass)
    return MVT::i32;
  if (RC == &WebAssembly::I64RegClass)
    return MVT::i64;
  if (RC == &WebAssembly::F32RegClass)
    return MVT::f32;
  if (RC == &WebAssembly::F64RegClass)
    return MVT::f64;
  if (RC == &WebAssembly::V128RegClass)
    return MVT::v16i8;
  if (RC == &WebAssembly::FUNCREFRegClass)
```
- **EN**: Implements helper routine(s) `llvm_unreachable`, `getLocalTeeOpcode`, `typeForRegClass` for this portion of the WebAssembly backend backend implementation logic. Register assignment and register-class constraints matter here.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `llvm_unreachable`, `getLocalTeeOpcode`, `typeForRegClass`。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 184-221

```cpp
    return MVT::funcref;
  if (RC == &WebAssembly::EXTERNREFRegClass)
    return MVT::externref;
  if (RC == &WebAssembly::EXNREFRegClass)
    return MVT::exnref;
  llvm_unreachable("unrecognized register class");
}

/// Given a MachineOperand of a stackified vreg, return the instruction at the
/// start of the expression tree.
static MachineInstr *findStartOfTree(MachineOperand &MO,
                                     MachineRegisterInfo &MRI,
                                     const WebAssemblyFunctionInfo &MFI) {
  Register Reg = MO.getReg();
  assert(MFI.isVRegStackified(Reg));
  MachineInstr *Def = MRI.getVRegDef(Reg);

  // If this instruction has any non-stackified defs, it is the start
  for (auto DefReg : Def->defs()) {
    if (!MFI.isVRegStackified(DefReg.getReg())) {
      return Def;
    }
  }

  // Find the first stackified use and proceed from there.
  for (MachineOperand &DefMO : Def->explicit_uses()) {
    if (!DefMO.isReg())
      continue;
    return findStartOfTree(DefMO, MRI, MFI);
  }

  // If there were no stackified uses, we've reached the start.
  return Def;
}

// FAKE_USEs are no-ops, so remove them here so that the values used by them
// will be correctly dropped later.
static void removeFakeUses(MachineFunction &MF) {
```
- **EN**: Implements helper routine(s) `llvm_unreachable`, `findStartOfTree`, `getReg` for this portion of the WebAssembly backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `llvm_unreachable`, `findStartOfTree`, `getReg`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 222-259

```cpp
  SmallVector<MachineInstr *> ToDelete;
  for (auto &MBB : MF)
    for (auto &MI : MBB)
      if (MI.isFakeUse())
        ToDelete.push_back(&MI);
  for (auto *MI : ToDelete)
    MI->eraseFromParent();
}

bool WebAssemblyExplicitLocals::runOnMachineFunction(MachineFunction &MF) {
  LLVM_DEBUG(dbgs() << "********** Make Locals Explicit **********\n"
                       "********** Function: "
                    << MF.getName() << '\n');

  bool Changed = false;
  MachineRegisterInfo &MRI = MF.getRegInfo();
  WebAssemblyFunctionInfo &MFI = *MF.getInfo<WebAssemblyFunctionInfo>();
  const auto *TII = MF.getSubtarget<WebAssemblySubtarget>().getInstrInfo();

  removeFakeUses(MF);

  // Map non-stackified virtual registers to their local ids.
  DenseMap<unsigned, unsigned> Reg2Local;

  // Handle ARGUMENTS first to ensure that they get the designated numbers.
  for (MachineBasicBlock::iterator I = MF.begin()->begin(),
                                   E = MF.begin()->end();
       I != E;) {
    MachineInstr &MI = *I++;
    if (!WebAssembly::isArgument(MI.getOpcode()))
      break;
    Register Reg = MI.getOperand(0).getReg();
    assert(!MFI.isVRegStackified(Reg));
    auto Local = static_cast<unsigned>(MI.getOperand(1).getImm());
    Reg2Local[Reg] = Local;
    checkFrameBase(MFI, Local, Reg);

    // Update debug value to point to the local before removing.
```
- **EN**: Implements helper routine(s) `isFakeUse`, `push_back`, `eraseFromParent` for this portion of the WebAssembly backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `isFakeUse`, `push_back`, `eraseFromParent`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 260-297

```cpp
    WebAssemblyDebugValueManager(&MI).replaceWithLocal(Local);

    MI.eraseFromParent();
    Changed = true;
  }

  // Start assigning local numbers after the last parameter and after any
  // already-assigned locals.
  unsigned CurLocal = static_cast<unsigned>(MFI.getParams().size());
  CurLocal += static_cast<unsigned>(MFI.getLocals().size());

  // Precompute the set of registers that are unused, so that we can insert
  // drops to their defs.
  // And unstackify any stackified registers that don't have any uses, so that
  // they can be dropped later. This can happen when transformations after
  // RegStackify remove instructions using stackified registers.
  BitVector UseEmpty(MRI.getNumVirtRegs());
  for (unsigned I = 0, E = MRI.getNumVirtRegs(); I < E; ++I) {
    Register Reg = Register::index2VirtReg(I);
    if (MRI.use_empty(Reg)) {
      UseEmpty[I] = true;
      MFI.unstackifyVReg(Reg);
    }
  }

  // Visit each instruction in the function.
  for (MachineBasicBlock &MBB : MF) {
    for (MachineInstr &MI : llvm::make_early_inc_range(MBB)) {
      assert(!WebAssembly::isArgument(MI.getOpcode()));

      if (MI.isDebugInstr() || MI.isLabel())
        continue;

      if (MI.getOpcode() == WebAssembly::IMPLICIT_DEF) {
        MI.eraseFromParent();
        Changed = true;
        continue;
      }
```
- **EN**: Implements helper routine(s) `WebAssemblyDebugValueManager`, `replaceWithLocal`, `eraseFromParent` for this portion of the WebAssembly backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `WebAssemblyDebugValueManager`, `replaceWithLocal`, `eraseFromParent`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 298-335

```cpp

      // Replace tee instructions with local.tee. The difference is that tee
      // instructions have two defs, while local.tee instructions have one def
      // and an index of a local to write to.
      //
      // - Before:
      // TeeReg, Reg = TEE DefReg
      // INST ..., TeeReg, ...
      // INST ..., Reg, ...
      // INST ..., Reg, ...
      // * DefReg: may or may not be stackified
      // * Reg: not stackified
      // * TeeReg: stackified
      //
      // - After (when DefReg was already stackified):
      // TeeReg = LOCAL_TEE LocalId1, DefReg
      // INST ..., TeeReg, ...
      // INST ..., Reg, ...
      // INST ..., Reg, ...
      // * Reg: mapped to LocalId1
      // * TeeReg: stackified
      //
      // - After (when DefReg was not already stackified):
      // NewReg = LOCAL_GET LocalId1
      // TeeReg = LOCAL_TEE LocalId2, NewReg
      // INST ..., TeeReg, ...
      // INST ..., Reg, ...
      // INST ..., Reg, ...
      // * DefReg: mapped to LocalId1
      // * Reg: mapped to LocalId2
      // * TeeReg: stackified
      if (WebAssembly::isTee(MI.getOpcode())) {
        assert(MFI.isVRegStackified(MI.getOperand(0).getReg()));
        assert(!MFI.isVRegStackified(MI.getOperand(1).getReg()));
        Register DefReg = MI.getOperand(2).getReg();
        const TargetRegisterClass *RC = MRI.getRegClass(DefReg);

        // Stackify the input if it isn't stackified yet.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Replace tee instructions with local.tee. The difference is that tee". Register assignment and register-class constraints matter here.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Replace tee instructions with local.tee. The difference is that tee”。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 336-373

```cpp
        if (!MFI.isVRegStackified(DefReg)) {
          unsigned LocalId = getLocalId(Reg2Local, MFI, CurLocal, DefReg);
          Register NewReg = MRI.createVirtualRegister(RC);
          unsigned Opc = getLocalGetOpcode(RC);
          BuildMI(MBB, &MI, MI.getDebugLoc(), TII->get(Opc), NewReg)
              .addImm(LocalId);
          MI.getOperand(2).setReg(NewReg);
          MFI.stackifyVReg(MRI, NewReg);
        }

        // Replace the TEE with a LOCAL_TEE.
        unsigned LocalId =
            getLocalId(Reg2Local, MFI, CurLocal, MI.getOperand(1).getReg());
        unsigned Opc = getLocalTeeOpcode(RC);
        BuildMI(MBB, &MI, MI.getDebugLoc(), TII->get(Opc),
                MI.getOperand(0).getReg())
            .addImm(LocalId)
            .addReg(MI.getOperand(2).getReg());

        WebAssemblyDebugValueManager(&MI).replaceWithLocal(LocalId);

        MI.eraseFromParent();
        Changed = true;
        continue;
      }

      // Insert local.sets for any defs that aren't stackified yet.
      for (auto &Def : MI.defs()) {
        Register OldReg = Def.getReg();
        if (!MFI.isVRegStackified(OldReg)) {
          const TargetRegisterClass *RC = MRI.getRegClass(OldReg);
          Register NewReg = MRI.createVirtualRegister(RC);
          auto InsertPt = std::next(MI.getIterator());
          if (UseEmpty[OldReg.virtRegIndex()]) {
            unsigned Opc = getDropOpcode(RC);
            MachineInstr *Drop =
                BuildMI(MBB, InsertPt, MI.getDebugLoc(), TII->get(Opc))
                    .addReg(NewReg);
```
- **EN**: Implements helper routine(s) `isVRegStackified`, `getLocalId`, `createVirtualRegister` for this portion of the WebAssembly backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `isVRegStackified`, `getLocalId`, `createVirtualRegister`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 374-411

```cpp
            // After the drop instruction, this reg operand will not be used
            Drop->getOperand(0).setIsKill();
            if (MFI.isFrameBaseVirtual() && OldReg == MFI.getFrameBaseVreg())
              MFI.clearFrameBaseVreg();
          } else {
            unsigned LocalId = getLocalId(Reg2Local, MFI, CurLocal, OldReg);
            unsigned Opc = getLocalSetOpcode(RC);

            WebAssemblyDebugValueManager(&MI).replaceWithLocal(LocalId);

            BuildMI(MBB, InsertPt, MI.getDebugLoc(), TII->get(Opc))
                .addImm(LocalId)
                .addReg(NewReg);
          }
          // This register operand of the original instruction is now being used
          // by the inserted drop or local.set instruction, so make it not dead
          // yet.
          Def.setReg(NewReg);
          Def.setIsDead(false);
          MFI.stackifyVReg(MRI, NewReg);
          Changed = true;
        }
      }

      // Insert local.gets for any uses that aren't stackified yet.
      MachineInstr *InsertPt = &MI;
      for (MachineOperand &MO : reverse(MI.explicit_uses())) {
        if (!MO.isReg())
          continue;

        Register OldReg = MO.getReg();

        // Inline asm may have a def in the middle of the operands. Our contract
        // with inline asm register operands is to provide local indices as
        // immediates.
        if (MO.isDef()) {
          assert(MI.isInlineAsm());
          unsigned LocalId = getLocalId(Reg2Local, MFI, CurLocal, OldReg);
```
- **EN**: Implements helper routine(s) `getOperand`, `setIsKill`, `isFrameBaseVirtual` for this portion of the WebAssembly backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `getOperand`, `setIsKill`, `isFrameBaseVirtual`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 412-449

```cpp
          // If this register operand is tied to another operand, we can't
          // change it to an immediate. Untie it first.
          MI.untieRegOperand(MO.getOperandNo());
          MO.ChangeToImmediate(LocalId);
          continue;
        }

        // If we see a stackified register, prepare to insert subsequent
        // local.gets before the start of its tree.
        if (MFI.isVRegStackified(OldReg)) {
          InsertPt = findStartOfTree(MO, MRI, MFI);
          continue;
        }

        // Our contract with inline asm register operands is to provide local
        // indices as immediates.
        if (MI.isInlineAsm()) {
          unsigned LocalId = getLocalId(Reg2Local, MFI, CurLocal, OldReg);
          // Untie it first if this reg operand is tied to another operand.
          MI.untieRegOperand(MO.getOperandNo());
          MO.ChangeToImmediate(LocalId);
          continue;
        }

        // Insert a local.get.
        unsigned LocalId = getLocalId(Reg2Local, MFI, CurLocal, OldReg);
        const TargetRegisterClass *RC = MRI.getRegClass(OldReg);
        Register NewReg = MRI.createVirtualRegister(RC);
        unsigned Opc = getLocalGetOpcode(RC);
        // Use a InsertPt as our DebugLoc, since MI may be discontinuous from
        // the where this local is being inserted, causing non-linear stepping
        // in the debugger or function entry points where variables aren't live
        // yet. Alternative is previous instruction, but that is strictly worse
        // since it can point at the previous statement.
        // See crbug.com/1251909, crbug.com/1249745
        InsertPt = BuildMI(MBB, InsertPt, InsertPt->getDebugLoc(),
                           TII->get(Opc), NewReg).addImm(LocalId);
        MO.setReg(NewReg);
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "If this register operand is tied to another operand, we can't". Register assignment and register-class constraints matter here.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“If this register operand is tied to another operand, we can't”。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 450-477

```cpp
        MFI.stackifyVReg(MRI, NewReg);
        Changed = true;
      }

      // Coalesce and eliminate COPY instructions.
      if (WebAssembly::isCopy(MI.getOpcode())) {
        MRI.replaceRegWith(MI.getOperand(1).getReg(),
                           MI.getOperand(0).getReg());
        MI.eraseFromParent();
        Changed = true;
      }
    }
  }

  // Define the locals.
  // TODO: Sort the locals for better compression.
  MFI.setNumLocals(CurLocal - MFI.getParams().size());
  for (unsigned I = 0, E = MRI.getNumVirtRegs(); I < E; ++I) {
    Register Reg = Register::index2VirtReg(I);
    auto RL = Reg2Local.find(Reg);
    if (RL == Reg2Local.end() || RL->second < MFI.getParams().size())
      continue;

    MFI.setLocal(RL->second - MFI.getParams().size(),
                 typeForRegClass(MRI.getRegClass(Reg)));
    Changed = true;
  }
```
- **EN**: Implements helper routine(s) `stackifyVReg`, `isCopy`, `getOpcode` for this portion of the WebAssembly backend backend implementation logic. Register assignment and register-class constraints matter here.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `stackifyVReg`, `isCopy`, `getOpcode`。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 478-491

```cpp
#ifndef NDEBUG
  // Assert that all registers have been stackified at this point.
  for (const MachineBasicBlock &MBB : MF) {
    for (const MachineInstr &MI : MBB) {
      if (MI.isDebugInstr() || MI.isLabel())
        continue;
      for (const MachineOperand &MO : MI.explicit_operands()) {
        assert(
            (!MO.isReg() || MRI.use_empty(MO.getReg()) ||
             MFI.isVRegStackified(MO.getReg())) &&
            "WebAssemblyExplicitLocals failed to stackify a register operand");
      }
    }
  }
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 492-495

```cpp
#endif

  return Changed;
}
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Backend implementation logic / 后端实现逻辑
- Register modeling / 寄存器建模
- Stack frame management / 栈帧管理
- Subtarget features / 子目标特性
- Instruction selection or opcode handling / 指令选择或操作码处理
- Declarative TableGen records / 声明式 TableGen 记录
- Vector or SIMD behavior / 向量或 SIMD 行为

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `MCTargetDesc/WebAssemblyMCTargetDesc.h`
- `WebAssembly.h`
- `WebAssemblyDebugValueManager.h`
- `WebAssemblyMachineFunctionInfo.h`
- `WebAssemblySubtarget.h`
- `WebAssemblyUtilities.h`
- `llvm/CodeGen/MachineBlockFrequencyInfo.h`
- `llvm/CodeGen/MachineInstrBuilder.h`
- `llvm/CodeGen/MachineRegisterInfo.h`
- `llvm/CodeGen/Passes.h`
- `llvm/Support/Debug.h`
- `llvm/Support/raw_ostream.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- LLVM Support utilities / LLVM Support 工具层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
