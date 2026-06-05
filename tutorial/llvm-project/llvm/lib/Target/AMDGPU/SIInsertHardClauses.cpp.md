# SIInsertHardClauses.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/SIInsertHardClauses.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements SIInsertHardClauses for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 SIInsertHardClauses 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-33: File banner, license, and overview
```cpp
//===- SIInsertHardClauses.cpp - Insert Hard Clauses ----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// Insert s_clause instructions to form hard clauses.
///
/// Clausing load instructions can give cache coherency benefits. Before gfx10,
/// the hardware automatically detected "soft clauses", which were sequences of
/// memory instructions of the same type. In gfx10 this detection was removed,
/// and the s_clause instruction was introduced to explicitly mark "hard
/// clauses".
///
/// It's the scheduler's job to form the clauses by putting similar memory
/// instructions next to each other. Our job is just to insert an s_clause
/// instruction to mark the start of each clause.
///
/// Note that hard clauses are very similar to, but logically distinct from, the
/// groups of instructions that have to be restartable when XNACK is enabled.
/// The rules are slightly different in each case. For example an s_nop
/// instruction breaks a restartable group, but can appear in the middle of a
/// hard clause. (Before gfx10 there wasn't a distinction, and both were called
/// "soft clauses" or just "clauses".)
///
/// The SIFormMemoryClauses pass and GCNHazardRecognizer deal with restartable
/// groups, not hard clauses.
//
//===----------------------------------------------------------------------===//

```
**EN:** This opening section establishes the file identity, licensing terms, and high-level intent before the backend-specific code begins.
**CN:** 开头部分先给出文件标识、许可证信息以及总体意图，为后续后端专用代码建立上下文。

### Lines 34-62: Header dependencies and setup
```cpp
#include "AMDGPU.h"
#include "GCNSubtarget.h"
#include "MCTargetDesc/AMDGPUMCTargetDesc.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachinePassManager.h"

using namespace llvm;

#define DEBUG_TYPE "si-insert-hard-clauses"

static cl::opt<unsigned>
    HardClauseLengthLimit("amdgpu-hard-clause-length-limit",
                          cl::desc("Maximum number of memory instructions to "
                                   "place in the same hard clause"),
                          cl::Hidden);

namespace {

enum HardClauseType {
  // For GFX10 and GFX1250:

  // Texture, buffer, global or scratch memory instructions.
  HARDCLAUSE_VMEM,
  // Flat (not global or scratch) memory instructions.
  HARDCLAUSE_FLAT,

  // For GFX11:

```
**EN:** These includes pull in the LLVM and AMDGPU declarations required by the rest of the file, making later symbols and helper APIs available. Main symbols: `HardClauseType`, `cl::desc`.
**CN:** 这些头文件引入本文件后续实现所需的 LLVM 与 AMDGPU 声明，使后面的符号和辅助 API 可以直接使用。 主要符号：`HardClauseType`, `cl::desc`。

### Lines 63-88: Implementation details and local logic
```cpp
  // Texture memory instructions.
  HARDCLAUSE_MIMG_LOAD,
  HARDCLAUSE_MIMG_STORE,
  HARDCLAUSE_MIMG_ATOMIC,
  HARDCLAUSE_MIMG_SAMPLE,
  // Buffer, global or scratch memory instructions.
  HARDCLAUSE_VMEM_LOAD,
  HARDCLAUSE_VMEM_STORE,
  HARDCLAUSE_VMEM_ATOMIC,
  // Flat (not global or scratch) memory instructions.
  HARDCLAUSE_FLAT_LOAD,
  HARDCLAUSE_FLAT_STORE,
  HARDCLAUSE_FLAT_ATOMIC,
  // BVH instructions.
  HARDCLAUSE_BVH,

  // Common:

  // Instructions that access LDS.
  HARDCLAUSE_LDS,
  // Scalar memory instructions.
  HARDCLAUSE_SMEM,
  // VALU instructions.
  HARDCLAUSE_VALU,
  LAST_REAL_HARDCLAUSE_TYPE = HARDCLAUSE_VALU,

```
**EN:** This section advances the file implementation by combining declarations, control flow, and helper operations that support the AMDGPU backend.
**CN:** 本节通过组合声明、控制流与辅助操作来推进文件实现，支撑 AMDGPU 后端的整体功能。

### Lines 89-122: Declares class SIInsertHardClauses
```cpp
  // Internal instructions, which are allowed in the middle of a hard clause,
  // except for s_waitcnt.
  HARDCLAUSE_INTERNAL,
  // Meta instructions that do not result in any ISA like KILL.
  HARDCLAUSE_IGNORE,
  // Instructions that are not allowed in a hard clause: SALU, export, branch,
  // message, GDS, s_waitcnt and anything else not mentioned above.
  HARDCLAUSE_ILLEGAL,
};

class SIInsertHardClauses {
public:
  const GCNSubtarget *ST = nullptr;

  HardClauseType getHardClauseType(const MachineInstr &MI) {
    if (MI.mayLoad() || (MI.mayStore() && ST->shouldClusterStores())) {
      if (ST->getGeneration() == AMDGPUSubtarget::GFX10 ||
          ST->hasGFX1250Insts()) {
        if ((SIInstrInfo::isVMEM(MI) && !SIInstrInfo::isFLAT(MI)) ||
            SIInstrInfo::isSegmentSpecificFLAT(MI)) {
          if (ST->hasNSAClauseBug()) {
            const AMDGPU::MIMGInfo *Info = AMDGPU::getMIMGInfo(MI.getOpcode());
            if (Info && Info->MIMGEncoding == AMDGPU::MIMGEncGfx10NSA)
              return HARDCLAUSE_ILLEGAL;
          }
          return HARDCLAUSE_VMEM;
        }
        if (SIInstrInfo::isFLAT(MI))
          return HARDCLAUSE_FLAT;
      } else {
        if (SIInstrInfo::isMIMG(MI)) {
          const AMDGPU::MIMGInfo *Info = AMDGPU::getMIMGInfo(MI.getOpcode());
          const AMDGPU::MIMGBaseOpcodeInfo *BaseInfo =
              AMDGPU::getMIMGBaseOpcodeInfo(Info->BaseOpcode);
```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `SIInsertHardClauses`, `SIInstrInfo::isVMEM`, `SIInstrInfo::isFLAT`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`SIInsertHardClauses`, `SIInstrInfo::isVMEM`, `SIInstrInfo::isFLAT`。

### Lines 123-149: Conditional logic and checks
```cpp
          if (BaseInfo->BVH)
            return HARDCLAUSE_BVH;
          if (BaseInfo->Sampler || BaseInfo->MSAA)
            return HARDCLAUSE_MIMG_SAMPLE;
          return MI.mayLoad() ? MI.mayStore() ? HARDCLAUSE_MIMG_ATOMIC
                                              : HARDCLAUSE_MIMG_LOAD
                              : HARDCLAUSE_MIMG_STORE;
        }
        if ((SIInstrInfo::isVMEM(MI) && !SIInstrInfo::isFLAT(MI)) ||
            SIInstrInfo::isSegmentSpecificFLAT(MI)) {
          return MI.mayLoad() ? MI.mayStore() ? HARDCLAUSE_VMEM_ATOMIC
                                              : HARDCLAUSE_VMEM_LOAD
                              : HARDCLAUSE_VMEM_STORE;
        }
        if (SIInstrInfo::isFLAT(MI)) {
          return MI.mayLoad() ? MI.mayStore() ? HARDCLAUSE_FLAT_ATOMIC
                                              : HARDCLAUSE_FLAT_LOAD
                              : HARDCLAUSE_FLAT_STORE;
        }
      }
      // TODO: LDS
      if (SIInstrInfo::isSMRD(MI))
        return HARDCLAUSE_SMEM;
    }

    // Don't form VALU clauses. It's not clear what benefit they give, if any.

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `SIInstrInfo::isVMEM`, `SIInstrInfo::isFLAT`, `SIInstrInfo::isSegmentSpecificFLAT`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`SIInstrInfo::isVMEM`, `SIInstrInfo::isFLAT`, `SIInstrInfo::isSegmentSpecificFLAT`。

### Lines 150-183: Declares struct ClauseInfo
```cpp
    // In practice s_nop is the only internal instruction we're likely to see.
    // It's safe to treat the rest as illegal.
    if (MI.getOpcode() == AMDGPU::S_NOP)
      return HARDCLAUSE_INTERNAL;
    if (MI.isMetaInstruction())
      return HARDCLAUSE_IGNORE;
    return HARDCLAUSE_ILLEGAL;
  }

  // Track information about a clause as we discover it.
  struct ClauseInfo {
    // The type of all (non-internal) instructions in the clause.
    HardClauseType Type = HARDCLAUSE_ILLEGAL;
    // The first (necessarily non-internal) instruction in the clause.
    MachineInstr *First = nullptr;
    // The last non-internal instruction in the clause.
    MachineInstr *Last = nullptr;
    // The length of the clause including any internal instructions in the
    // middle (but not at the end) of the clause.
    unsigned Length = 0;
    // Internal instructions at the and of a clause should not be included in
    // the clause. Count them in TrailingInternalLength until a new memory
    // instruction is added.
    unsigned TrailingInternalLength = 0;
    // The base operands of *Last.
    SmallVector<const MachineOperand *, 4> BaseOps;
  };

  bool emitClause(const ClauseInfo &CI, const SIInstrInfo *SII) {
    if (CI.First == CI.Last)
      return false;
    assert(CI.Length <= ST->maxHardClauseLength() &&
           "Hard clause is too long!");

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `ClauseInfo`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`ClauseInfo`。

### Lines 184-208: Defines run
```cpp
    auto &MBB = *CI.First->getParent();
    auto ClauseMI =
        BuildMI(MBB, *CI.First, DebugLoc(), SII->get(AMDGPU::S_CLAUSE))
            .addImm(CI.Length - 1);
    finalizeBundle(MBB, ClauseMI->getIterator(),
                   std::next(CI.Last->getIterator()));
    return true;
  }

  bool run(MachineFunction &MF) {
    ST = &MF.getSubtarget<GCNSubtarget>();
    if (!ST->hasHardClauses())
      return false;

    unsigned MaxClauseLength = MF.getFunction().getFnAttributeAsParsedInteger(
        "amdgpu-hard-clause-length-limit", 255);
    if (HardClauseLengthLimit.getNumOccurrences())
      MaxClauseLength = HardClauseLengthLimit;
    MaxClauseLength = std::min(MaxClauseLength, ST->maxHardClauseLength());
    if (MaxClauseLength <= 1)
      return false;

    const SIInstrInfo *SII = ST->getInstrInfo();
    const TargetRegisterInfo *TRI = ST->getRegisterInfo();

```
**EN:** This section contains concrete logic for run. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `std::next`, `std::min`.
**CN:** 本节包含与 run 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`std::next`, `std::min`。

### Lines 209-240: Conditional logic and checks
```cpp
    bool Changed = false;
    for (auto &MBB : MF) {
      ClauseInfo CI;
      unsigned ExistingClauseRemaining = 0;
      for (auto &MI : MBB) {
        HardClauseType Type;
        if (ExistingClauseRemaining) {
          if (!MI.isMetaInstruction())
            ExistingClauseRemaining--;
          Type = HARDCLAUSE_ILLEGAL;
        } else if (MI.getOpcode() == AMDGPU::S_CLAUSE) {
          // Respect existing explicit clauses. Re-clausing instructions that
          // are already covered by an S_CLAUSE can create nested clauses.
          ExistingClauseRemaining = (MI.getOperand(0).getImm() & 63) + 1;
          Type = HARDCLAUSE_ILLEGAL;
        } else {
          Type = getHardClauseType(MI);
        }

        int64_t Dummy1;
        bool Dummy2;
        LocationSize Dummy3 = LocationSize::precise(0);
        SmallVector<const MachineOperand *, 4> BaseOps;
        if (Type <= LAST_REAL_HARDCLAUSE_TYPE) {
          if (!SII->getMemOperandsWithOffsetWidth(MI, BaseOps, Dummy1, Dummy2,
                                                  Dummy3, TRI)) {
            // We failed to get the base operands, so we'll never clause this
            // instruction with any other, so pretend it's illegal.
            Type = HARDCLAUSE_ILLEGAL;
          }
        }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `LocationSize::precise`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`LocationSize::precise`。

### Lines 241-258: Conditional logic and checks
```cpp
        if (CI.Length == MaxClauseLength ||
            (CI.Length && Type != HARDCLAUSE_INTERNAL &&
             Type != HARDCLAUSE_IGNORE &&
             (Type != CI.Type ||
              // Note that we lie to shouldClusterMemOps about the size of the
              // cluster. When shouldClusterMemOps is called from the machine
              // scheduler it limits the size of the cluster to avoid increasing
              // register pressure too much, but this pass runs after register
              // allocation so there is no need for that kind of limit.
              // We also lie about the Offset and OffsetIsScalable parameters,
              // as they aren't used in the SIInstrInfo implementation.
              !SII->shouldClusterMemOps(CI.BaseOps, 0, false, BaseOps, 0, false,
                                        2, 2)))) {
          // Finish the current clause.
          Changed |= emitClause(CI, SII);
          CI = ClauseInfo();
        }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 259-291: Declares class SIInsertHardClausesLegacy
```cpp
        if (CI.Length) {
          // Extend the current clause.
          if (Type != HARDCLAUSE_IGNORE) {
            if (Type == HARDCLAUSE_INTERNAL) {
              ++CI.TrailingInternalLength;
            } else {
              ++CI.Length;
              CI.Length += CI.TrailingInternalLength;
              CI.TrailingInternalLength = 0;
              CI.Last = &MI;
              CI.BaseOps = std::move(BaseOps);
            }
          }
        } else if (Type <= LAST_REAL_HARDCLAUSE_TYPE) {
          // Start a new clause.
          CI = ClauseInfo{Type, &MI, &MI, 1, 0, std::move(BaseOps)};
        }
      }

      // Finish the last clause in the basic block if any.
      if (CI.Length)
        Changed |= emitClause(CI, SII);
    }

    return Changed;
  }
};

class SIInsertHardClausesLegacy : public MachineFunctionPass {
public:
  static char ID;
  SIInsertHardClausesLegacy() : MachineFunctionPass(ID) {}

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `SIInsertHardClausesLegacy`, `std::move`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`SIInsertHardClausesLegacy`, `std::move`。

### Lines 292-323: Registers LLVM passes
```cpp
  bool runOnMachineFunction(MachineFunction &MF) override {
    if (skipFunction(MF.getFunction()))
      return false;

    return SIInsertHardClauses().run(MF);
  }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    MachineFunctionPass::getAnalysisUsage(AU);
  }
};

} // namespace

PreservedAnalyses
llvm::SIInsertHardClausesPass::run(MachineFunction &MF,
                                   MachineFunctionAnalysisManager &MFAM) {
  if (!SIInsertHardClauses().run(MF))
    return PreservedAnalyses::all();

  auto PA = getMachineFunctionPassPreservedAnalyses();
  PA.preserveSet<CFGAnalyses>();
  return PA;
}

char SIInsertHardClausesLegacy::ID = 0;

char &llvm::SIInsertHardClausesID = SIInsertHardClausesLegacy::ID;

INITIALIZE_PASS(SIInsertHardClausesLegacy, DEBUG_TYPE, "SI Insert Hard Clauses",
                false, false)
```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `MachineFunctionPass::getAnalysisUsage`, `SIInsertHardClausesPass::run`, `PreservedAnalyses::all`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`MachineFunctionPass::getAnalysisUsage`, `SIInsertHardClausesPass::run`, `PreservedAnalyses::all`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `HardClauseType`, `SIInsertHardClauses`, `ClauseInfo`, `SIInsertHardClausesLegacy`, `cl::desc`, `SIInstrInfo::isVMEM`
- **Main themes / 核心主题**: instruction semantics / 指令语义; subtarget modeling / 子目标建模; legalization / 合法化; scheduling / 调度
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPU.h"`
- `"GCNSubtarget.h"`
- `"MCTargetDesc/AMDGPUMCTargetDesc.h"`
- `"llvm/ADT/SmallVector.h"`
- `"llvm/CodeGen/MachineFunctionPass.h"`
- `"llvm/CodeGen/MachinePassManager.h"`
