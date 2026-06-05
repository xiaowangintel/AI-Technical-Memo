# AMDGPU.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPU.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This header declares the interfaces, data structures, and pass entry points for AMDGPU in the LLVM AMDGPU backend. It defines the contracts consumed by other AMDGPU backend components. / 该头文件声明 LLVM AMDGPU 后端中 AMDGPU 的接口、数据结构与 Pass 入口。它定义了其他 AMDGPU 后端组件依赖的契约。

## Line-by-Line Analysis / 逐行分析
### Lines 1-26: File banner, includes, and setup
```cpp
//===-- AMDGPU.h - MachineFunction passes hw codegen --------------*- C++ -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
/// \file
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AMDGPU_AMDGPU_H
#define LLVM_LIB_TARGET_AMDGPU_AMDGPU_H

#include "llvm/Analysis/CGSCCPassManager.h"
#include "llvm/CodeGen/MachinePassManager.h"
#include "llvm/IR/PassManager.h"
#include "llvm/Pass.h"
#include "llvm/Support/AMDGPUAddrSpace.h"
#include "llvm/Support/CodeGen.h"

namespace llvm {

class AMDGPUTargetMachine;
class LazyCallGraph;
class GCNTargetMachine;
class TargetMachine;

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `AMDGPUTargetMachine`, `LazyCallGraph`, `GCNTargetMachine`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`AMDGPUTargetMachine`, `LazyCallGraph`, `GCNTargetMachine`。

### Lines 27-58: Declares initializeAMDGPUPreLegalizerCombinerPass
```cpp
// GlobalISel passes
void initializeAMDGPUPreLegalizerCombinerPass(PassRegistry &);
FunctionPass *createAMDGPUPreLegalizeCombiner(bool IsOptNone);
void initializeAMDGPUPostLegalizerCombinerPass(PassRegistry &);
FunctionPass *createAMDGPUPostLegalizeCombiner(bool IsOptNone);
FunctionPass *createAMDGPURegBankCombiner(bool IsOptNone);
void initializeAMDGPURegBankCombinerPass(PassRegistry &);
FunctionPass *createAMDGPUGlobalISelDivergenceLoweringPass();
FunctionPass *createAMDGPURegBankSelectPass();
FunctionPass *createAMDGPURegBankLegalizePass();

// SI Passes
FunctionPass *createGCNDPPCombinePass();
FunctionPass *createSIAnnotateControlFlowLegacyPass();
FunctionPass *createSIFoldOperandsLegacyPass();
FunctionPass *createSIPeepholeSDWALegacyPass();
FunctionPass *createSILowerI1CopiesLegacyPass();
FunctionPass *createSIShrinkInstructionsLegacyPass();
FunctionPass *createSILoadStoreOptimizerLegacyPass();
FunctionPass *createSIWholeQuadModeLegacyPass();
FunctionPass *createSIFixControlFlowLiveIntervalsPass();
FunctionPass *createSIOptimizeExecMaskingPreRAPass();
FunctionPass *createSIOptimizeVGPRLiveRangeLegacyPass();
FunctionPass *createAMDGPUNextUseAnalysisLegacyPass();
FunctionPass *createAMDGPUNextUseAnalysisPrinterLegacyPass();
FunctionPass *createSIFixSGPRCopiesLegacyPass();
FunctionPass *createLowerWWMCopiesPass();
FunctionPass *createSIMemoryLegalizerPass();
FunctionPass *createSIInsertWaitcntsPass();
FunctionPass *createSIPreAllocateWWMRegsLegacyPass();
FunctionPass *createSIFormMemoryClausesLegacyPass();

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 59-89: Declares struct AMDGPUSimplifyLibCallsPass
```cpp
FunctionPass *createSIPostRABundlerPass();
FunctionPass *createAMDGPUImageIntrinsicOptimizerPass(const TargetMachine *);
ModulePass *createAMDGPURemoveIncompatibleFunctionsPass(const TargetMachine *);
FunctionPass *createAMDGPUCodeGenPreparePass();
FunctionPass *createAMDGPULateCodeGenPrepareLegacyPass();
FunctionPass *createAMDGPUReserveWWMRegsPass();
FunctionPass *createAMDGPURewriteOutArgumentsPass();
ModulePass *
createAMDGPULowerModuleLDSLegacyPass(const AMDGPUTargetMachine *TM = nullptr);
ModulePass *createAMDGPULowerBufferFatPointersPass();
ModulePass *createAMDGPULowerIntrinsicsLegacyPass();
FunctionPass *createSIModeRegisterPass();
FunctionPass *createGCNPreRAOptimizationsLegacyPass();
FunctionPass *createAMDGPUPreloadKernArgPrologLegacyPass();
ModulePass *createAMDGPUPreloadKernelArgumentsLegacyPass(const TargetMachine *);

struct AMDGPUSimplifyLibCallsPass
    : OptionalPassInfoMixin<AMDGPUSimplifyLibCallsPass> {
  AMDGPUSimplifyLibCallsPass() = default;
  PreservedAnalyses run(Function &F, FunctionAnalysisManager &AM);
};

struct AMDGPUImageIntrinsicOptimizerPass
    : OptionalPassInfoMixin<AMDGPUImageIntrinsicOptimizerPass> {
  AMDGPUImageIntrinsicOptimizerPass(TargetMachine &TM) : TM(TM) {}
  PreservedAnalyses run(Function &F, FunctionAnalysisManager &AM);

private:
  TargetMachine &TM;
};

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUSimplifyLibCallsPass`, `AMDGPUImageIntrinsicOptimizerPass`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUSimplifyLibCallsPass`, `AMDGPUImageIntrinsicOptimizerPass`。

### Lines 90-122: Declares struct AMDGPUUseNativeCallsPass
```cpp
struct AMDGPUUseNativeCallsPass
    : OptionalPassInfoMixin<AMDGPUUseNativeCallsPass> {
  PreservedAnalyses run(Function &F, FunctionAnalysisManager &AM);
};

class SILowerI1CopiesPass : public OptionalPassInfoMixin<SILowerI1CopiesPass> {
public:
  SILowerI1CopiesPass() = default;
  PreservedAnalyses run(MachineFunction &MF,
                        MachineFunctionAnalysisManager &MFAM);
};

void initializeAMDGPUDAGToDAGISelLegacyPass(PassRegistry &);

void initializeAMDGPUAlwaysInlinePass(PassRegistry&);

void initializeAMDGPUAsmPrinterPass(PassRegistry &);

// DPP/Iterative option enables the atomic optimizer with given strategy
// whereas None disables the atomic optimizer.
enum class ScanOptions { DPP, Iterative, None };
FunctionPass *createAMDGPUAtomicOptimizerPass(ScanOptions ScanStrategy);
void initializeAMDGPUAtomicOptimizerPass(PassRegistry &);
extern char &AMDGPUAtomicOptimizerID;

ModulePass *createAMDGPUCtorDtorLoweringLegacyPass();
void initializeAMDGPUCtorDtorLoweringLegacyPass(PassRegistry &);
extern char &AMDGPUCtorDtorLoweringLegacyPassID;

FunctionPass *createAMDGPULowerKernelArgumentsPass();
void initializeAMDGPULowerKernelArgumentsPass(PassRegistry &);
extern char &AMDGPULowerKernelArgumentsID;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUUseNativeCallsPass`, `SILowerI1CopiesPass`, `ScanOptions`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUUseNativeCallsPass`, `SILowerI1CopiesPass`, `ScanOptions`。

### Lines 123-154: Declares struct AMDGPUPromoteKernelArgumentsPass
```cpp
FunctionPass *createAMDGPUPromoteKernelArgumentsPass();
void initializeAMDGPUPromoteKernelArgumentsPass(PassRegistry &);
extern char &AMDGPUPromoteKernelArgumentsID;

struct AMDGPUPromoteKernelArgumentsPass
    : OptionalPassInfoMixin<AMDGPUPromoteKernelArgumentsPass> {
  PreservedAnalyses run(Function &F, FunctionAnalysisManager &AM);
};

ModulePass *createAMDGPULowerKernelAttributesPass();
void initializeAMDGPULowerKernelAttributesPass(PassRegistry &);
extern char &AMDGPULowerKernelAttributesID;

struct AMDGPULowerKernelAttributesPass
    : OptionalPassInfoMixin<AMDGPULowerKernelAttributesPass> {
  PreservedAnalyses run(Function &F, FunctionAnalysisManager &AM);
};

void initializeAMDGPULowerModuleLDSLegacyPass(PassRegistry &);
extern char &AMDGPULowerModuleLDSLegacyPassID;

struct AMDGPULowerModuleLDSPass
    : OptionalPassInfoMixin<AMDGPULowerModuleLDSPass> {
  const AMDGPUTargetMachine &TM;
  AMDGPULowerModuleLDSPass(const AMDGPUTargetMachine &TM_) : TM(TM_) {}

  PreservedAnalyses run(Module &M, ModuleAnalysisManager &AM);
};

void initializeAMDGPULowerBufferFatPointersPass(PassRegistry &);
extern char &AMDGPULowerBufferFatPointersID;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUPromoteKernelArgumentsPass`, `AMDGPULowerKernelAttributesPass`, `AMDGPULowerModuleLDSPass`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUPromoteKernelArgumentsPass`, `AMDGPULowerKernelAttributesPass`, `AMDGPULowerModuleLDSPass`。

### Lines 155-186: Declares struct AMDGPULowerBufferFatPointersPass
```cpp
struct AMDGPULowerBufferFatPointersPass
    : OptionalPassInfoMixin<AMDGPULowerBufferFatPointersPass> {
  AMDGPULowerBufferFatPointersPass(const TargetMachine &TM) : TM(TM) {}
  PreservedAnalyses run(Module &M, ModuleAnalysisManager &AM);

private:
  const TargetMachine &TM;
};

void initializeAMDGPULowerIntrinsicsLegacyPass(PassRegistry &);

struct AMDGPULowerIntrinsicsPass
    : OptionalPassInfoMixin<AMDGPULowerIntrinsicsPass> {
  AMDGPULowerIntrinsicsPass(const AMDGPUTargetMachine &TM) : TM(TM) {}
  PreservedAnalyses run(Module &M, ModuleAnalysisManager &MAM);

private:
  const AMDGPUTargetMachine &TM;
};

void initializeAMDGPUPrepareAGPRAllocLegacyPass(PassRegistry &);
extern char &AMDGPUPrepareAGPRAllocLegacyID;

void initializeAMDGPUReserveWWMRegsLegacyPass(PassRegistry &);
extern char &AMDGPUReserveWWMRegsLegacyID;

void initializeAMDGPURewriteOutArgumentsPass(PassRegistry &);
extern char &AMDGPURewriteOutArgumentsID;

void initializeGCNDPPCombineLegacyPass(PassRegistry &);
extern char &GCNDPPCombineLegacyID;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPULowerBufferFatPointersPass`, `AMDGPULowerIntrinsicsPass`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPULowerBufferFatPointersPass`, `AMDGPULowerIntrinsicsPass`。

### Lines 187-219: Declares initializeSIFoldOperandsLegacyPass
```cpp
void initializeSIFoldOperandsLegacyPass(PassRegistry &);
extern char &SIFoldOperandsLegacyID;

void initializeSIPeepholeSDWALegacyPass(PassRegistry &);
extern char &SIPeepholeSDWALegacyID;

void initializeSIShrinkInstructionsLegacyPass(PassRegistry &);
extern char &SIShrinkInstructionsLegacyID;

void initializeSIFixSGPRCopiesLegacyPass(PassRegistry &);
extern char &SIFixSGPRCopiesLegacyID;

void initializeSIFixVGPRCopiesLegacyPass(PassRegistry &);
extern char &SIFixVGPRCopiesID;

void initializeAMDGPUNextUseAnalysisLegacyPassPass(PassRegistry &);
extern char &AMDGPUNextUseAnalysisLegacyID;

void initializeAMDGPUNextUseAnalysisPrinterLegacyPassPass(PassRegistry &);
extern char &AMDGPUNextUseAnalysisPrinterLegacyID;

void initializeSILowerWWMCopiesLegacyPass(PassRegistry &);
extern char &SILowerWWMCopiesLegacyID;

void initializeSILowerI1CopiesLegacyPass(PassRegistry &);
extern char &SILowerI1CopiesLegacyID;

void initializeAMDGPUGlobalISelDivergenceLoweringPass(PassRegistry &);
extern char &AMDGPUGlobalISelDivergenceLoweringID;

void initializeAMDGPURegBankSelectPass(PassRegistry &);
extern char &AMDGPURegBankSelectID;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 220-252: Declares initializeAMDGPURegBankLegalizePass
```cpp
void initializeAMDGPURegBankLegalizePass(PassRegistry &);
extern char &AMDGPURegBankLegalizeID;

void initializeAMDGPUMarkLastScratchLoadLegacyPass(PassRegistry &);
extern char &AMDGPUMarkLastScratchLoadID;

void initializeSILowerSGPRSpillsLegacyPass(PassRegistry &);
extern char &SILowerSGPRSpillsLegacyID;

void initializeSILoadStoreOptimizerLegacyPass(PassRegistry &);
extern char &SILoadStoreOptimizerLegacyID;

void initializeSIWholeQuadModeLegacyPass(PassRegistry &);
extern char &SIWholeQuadModeID;

void initializeSILowerControlFlowLegacyPass(PassRegistry &);
extern char &SILowerControlFlowLegacyID;

void initializeSIPreEmitPeepholeLegacyPass(PassRegistry &);
extern char &SIPreEmitPeepholeID;

void initializeSILateBranchLoweringLegacyPass(PassRegistry &);
extern char &SILateBranchLoweringPassID;

void initializeSIOptimizeExecMaskingLegacyPass(PassRegistry &);
extern char &SIOptimizeExecMaskingLegacyID;

void initializeSIPreAllocateWWMRegsLegacyPass(PassRegistry &);
extern char &SIPreAllocateWWMRegsLegacyID;

void initializeAMDGPUImageIntrinsicOptimizerPass(PassRegistry &);
extern char &AMDGPUImageIntrinsicOptimizerID;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 253-283: Declares struct AMDGPUPromoteAllocaPass
```cpp
void initializeAMDGPUPerfHintAnalysisLegacyPass(PassRegistry &);
extern char &AMDGPUPerfHintAnalysisLegacyID;

void initializeGCNRegPressurePrinterPass(PassRegistry &);
extern char &GCNRegPressurePrinterID;

void initializeAMDGPUPreloadKernArgPrologLegacyPass(PassRegistry &);
extern char &AMDGPUPreloadKernArgPrologLegacyID;

void initializeAMDGPUPreloadKernelArgumentsLegacyPass(PassRegistry &);
extern char &AMDGPUPreloadKernelArgumentsLegacyID;

// Passes common to R600 and SI
FunctionPass *createAMDGPUPromoteAlloca();
void initializeAMDGPUPromoteAllocaPass(PassRegistry&);
extern char &AMDGPUPromoteAllocaID;

struct AMDGPUPromoteAllocaPass
    : OptionalPassInfoMixin<AMDGPUPromoteAllocaPass> {
  AMDGPUPromoteAllocaPass(TargetMachine &TM) : TM(TM) {}
  PreservedAnalyses run(Function &F, FunctionAnalysisManager &AM);

private:
  TargetMachine &TM;
};

struct AMDGPUPromoteAllocaToVectorPass
    : OptionalPassInfoMixin<AMDGPUPromoteAllocaToVectorPass> {
  AMDGPUPromoteAllocaToVectorPass(TargetMachine &TM) : TM(TM) {}
  PreservedAnalyses run(Function &F, FunctionAnalysisManager &AM);

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUPromoteAllocaPass`, `AMDGPUPromoteAllocaToVectorPass`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUPromoteAllocaPass`, `AMDGPUPromoteAllocaToVectorPass`。

### Lines 284-316: Declares struct AMDGPUAtomicOptimizerPass
```cpp
private:
  TargetMachine &TM;
};

struct AMDGPUAtomicOptimizerPass
    : OptionalPassInfoMixin<AMDGPUAtomicOptimizerPass> {
  AMDGPUAtomicOptimizerPass(TargetMachine &TM, ScanOptions ScanImpl)
      : TM(TM), ScanImpl(ScanImpl) {}
  PreservedAnalyses run(Function &F, FunctionAnalysisManager &AM);

private:
  TargetMachine &TM;
  ScanOptions ScanImpl;
};

struct AMDGPUInsertDelayAluPass
    : public OptionalPassInfoMixin<AMDGPUInsertDelayAluPass> {
  PreservedAnalyses run(MachineFunction &F,
                        MachineFunctionAnalysisManager &MFAM);
};

Pass *createAMDGPUStructurizeCFGPass();
FunctionPass *createAMDGPUISelDag(TargetMachine &TM, CodeGenOptLevel OptLevel);
ModulePass *createAMDGPUAlwaysInlinePass(bool GlobalOpt = true);

struct AMDGPUAlwaysInlinePass : OptionalPassInfoMixin<AMDGPUAlwaysInlinePass> {
  AMDGPUAlwaysInlinePass(bool GlobalOpt = true) : GlobalOpt(GlobalOpt) {}
  PreservedAnalyses run(Module &M, ModuleAnalysisManager &AM);

private:
  bool GlobalOpt;
};

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUAtomicOptimizerPass`, `AMDGPUInsertDelayAluPass`, `AMDGPUAlwaysInlinePass`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUAtomicOptimizerPass`, `AMDGPUInsertDelayAluPass`, `AMDGPUAlwaysInlinePass`。

### Lines 317-347: Declares struct AMDGPULowerExecSyncPass
```cpp
void initializeAMDGPULowerExecSyncLegacyPass(PassRegistry &);
extern char &AMDGPULowerExecSyncLegacyPassID;
ModulePass *createAMDGPULowerExecSyncLegacyPass();

struct AMDGPULowerExecSyncPass
    : OptionalPassInfoMixin<AMDGPULowerExecSyncPass> {
  AMDGPULowerExecSyncPass() {}
  PreservedAnalyses run(Module &M, ModuleAnalysisManager &AM);
};

void initializeAMDGPUSwLowerLDSLegacyPass(PassRegistry &);
extern char &AMDGPUSwLowerLDSLegacyPassID;
ModulePass *
createAMDGPUSwLowerLDSLegacyPass(const AMDGPUTargetMachine *TM = nullptr);

struct AMDGPUSwLowerLDSPass : OptionalPassInfoMixin<AMDGPUSwLowerLDSPass> {
  const AMDGPUTargetMachine &TM;
  AMDGPUSwLowerLDSPass(const AMDGPUTargetMachine &TM_) : TM(TM_) {}
  PreservedAnalyses run(Module &M, ModuleAnalysisManager &AM);
};

class AMDGPUCodeGenPreparePass
    : public OptionalPassInfoMixin<AMDGPUCodeGenPreparePass> {
private:
  TargetMachine &TM;

public:
  AMDGPUCodeGenPreparePass(TargetMachine &TM) : TM(TM){};
  PreservedAnalyses run(Function &, FunctionAnalysisManager &);
};

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPULowerExecSyncPass`, `AMDGPUSwLowerLDSPass`, `AMDGPUCodeGenPreparePass`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPULowerExecSyncPass`, `AMDGPUSwLowerLDSPass`, `AMDGPUCodeGenPreparePass`。

### Lines 348-380: Declares class AMDGPULateCodeGenPreparePass
```cpp
class AMDGPULateCodeGenPreparePass
    : public OptionalPassInfoMixin<AMDGPULateCodeGenPreparePass> {
private:
  const GCNTargetMachine &TM;

public:
  AMDGPULateCodeGenPreparePass(const GCNTargetMachine &TM) : TM(TM) {};
  PreservedAnalyses run(Function &, FunctionAnalysisManager &);
};

class AMDGPULowerKernelArgumentsPass
    : public OptionalPassInfoMixin<AMDGPULowerKernelArgumentsPass> {
private:
  TargetMachine &TM;

public:
  AMDGPULowerKernelArgumentsPass(TargetMachine &TM) : TM(TM){};
  PreservedAnalyses run(Function &, FunctionAnalysisManager &);
};

struct AMDGPUAttributorOptions {
  bool IsClosedWorld = false;
};

class AMDGPUAttributorPass
    : public OptionalPassInfoMixin<AMDGPUAttributorPass> {
private:
  TargetMachine &TM;

  AMDGPUAttributorOptions Options;

  const ThinOrFullLTOPhase LTOPhase;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPULateCodeGenPreparePass`, `AMDGPULowerKernelArgumentsPass`, `AMDGPUAttributorOptions`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPULateCodeGenPreparePass`, `AMDGPULowerKernelArgumentsPass`, `AMDGPUAttributorOptions`。

### Lines 381-408: Declares class AMDGPUAttributorCGSCCPass
```cpp
public:
  AMDGPUAttributorPass(TargetMachine &TM, AMDGPUAttributorOptions Options,
                       ThinOrFullLTOPhase LTOPhase = ThinOrFullLTOPhase::None)
      : TM(TM), Options(Options), LTOPhase(LTOPhase) {};
  PreservedAnalyses run(Module &M, ModuleAnalysisManager &AM);
};

class AMDGPUAttributorCGSCCPass
    : public OptionalPassInfoMixin<AMDGPUAttributorCGSCCPass> {
private:
  GCNTargetMachine &TM;

public:
  AMDGPUAttributorCGSCCPass(GCNTargetMachine &TM) : TM(TM) {}
  PreservedAnalyses run(LazyCallGraph::SCC &C, CGSCCAnalysisManager &AM,
                        LazyCallGraph &CG, CGSCCUpdateResult &UR);
};

class AMDGPUPreloadKernelArgumentsPass
    : public OptionalPassInfoMixin<AMDGPUPreloadKernelArgumentsPass> {
  const TargetMachine &TM;

public:
  explicit AMDGPUPreloadKernelArgumentsPass(const TargetMachine &TM) : TM(TM) {}

  PreservedAnalyses run(Module &M, ModuleAnalysisManager &AM);
};

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUAttributorCGSCCPass`, `AMDGPUPreloadKernelArgumentsPass`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUAttributorCGSCCPass`, `AMDGPUPreloadKernelArgumentsPass`。

### Lines 409-441: Declares class AMDGPUAnnotateUniformValuesPass
```cpp
class AMDGPUAnnotateUniformValuesPass
    : public OptionalPassInfoMixin<AMDGPUAnnotateUniformValuesPass> {
public:
  AMDGPUAnnotateUniformValuesPass() = default;
  PreservedAnalyses run(Function &F, FunctionAnalysisManager &AM);
};

class SIModeRegisterPass : public OptionalPassInfoMixin<SIModeRegisterPass> {
public:
  SIModeRegisterPass() = default;
  PreservedAnalyses run(MachineFunction &F, MachineFunctionAnalysisManager &AM);
};

class SIMemoryLegalizerPass
    : public RequiredPassInfoMixin<SIMemoryLegalizerPass> {
public:
  PreservedAnalyses run(MachineFunction &MF,
                        MachineFunctionAnalysisManager &MFAM);
};

class GCNCreateVOPDPass : public OptionalPassInfoMixin<GCNCreateVOPDPass> {
public:
  PreservedAnalyses run(MachineFunction &MF,
                        MachineFunctionAnalysisManager &AM);
};

class AMDGPUMarkLastScratchLoadPass
    : public OptionalPassInfoMixin<AMDGPUMarkLastScratchLoadPass> {
public:
  PreservedAnalyses run(MachineFunction &MF,
                        MachineFunctionAnalysisManager &AM);
};

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUAnnotateUniformValuesPass`, `SIModeRegisterPass`, `SIMemoryLegalizerPass`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUAnnotateUniformValuesPass`, `SIModeRegisterPass`, `SIMemoryLegalizerPass`。

### Lines 442-469: Declares class SIInsertWaitcntsPass
```cpp
class SIInsertWaitcntsPass
    : public RequiredPassInfoMixin<SIInsertWaitcntsPass> {
public:
  PreservedAnalyses run(MachineFunction &MF,
                        MachineFunctionAnalysisManager &MFAM);
};

class SIInsertHardClausesPass
    : public OptionalPassInfoMixin<SIInsertHardClausesPass> {
public:
  PreservedAnalyses run(MachineFunction &MF,
                        MachineFunctionAnalysisManager &MFAM);
};

class SILateBranchLoweringPass
    : public RequiredPassInfoMixin<SILateBranchLoweringPass> {
public:
  PreservedAnalyses run(MachineFunction &MF,
                        MachineFunctionAnalysisManager &MFAM);
};

class SIPreEmitPeepholePass
    : public RequiredPassInfoMixin<SIPreEmitPeepholePass> {
public:
  PreservedAnalyses run(MachineFunction &MF,
                        MachineFunctionAnalysisManager &MFAM);
};

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `SIInsertWaitcntsPass`, `SIInsertHardClausesPass`, `SILateBranchLoweringPass`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`SIInsertWaitcntsPass`, `SIInsertHardClausesPass`, `SILateBranchLoweringPass`。

### Lines 470-502: Declares class AMDGPUSetWavePriorityPass
```cpp
class AMDGPUSetWavePriorityPass
    : public OptionalPassInfoMixin<AMDGPUSetWavePriorityPass> {
public:
  PreservedAnalyses run(MachineFunction &MF,
                        MachineFunctionAnalysisManager &MFAM);
};

FunctionPass *createAMDGPUAnnotateUniformValuesLegacy();

ModulePass *createAMDGPUPrintfRuntimeBinding();
void initializeAMDGPUPrintfRuntimeBindingPass(PassRegistry&);
extern char &AMDGPUPrintfRuntimeBindingID;

void initializeAMDGPUResourceUsageAnalysisWrapperPassPass(PassRegistry &);
extern char &AMDGPUResourceUsageAnalysisID;

struct AMDGPUPrintfRuntimeBindingPass
    : OptionalPassInfoMixin<AMDGPUPrintfRuntimeBindingPass> {
  PreservedAnalyses run(Module &M, ModuleAnalysisManager &AM);
};

void initializeSIOptimizeExecMaskingPreRALegacyPass(PassRegistry &);
extern char &SIOptimizeExecMaskingPreRAID;

void initializeSIOptimizeVGPRLiveRangeLegacyPass(PassRegistry &);
extern char &SIOptimizeVGPRLiveRangeLegacyID;

void initializeAMDGPUAnnotateUniformValuesLegacyPass(PassRegistry &);
extern char &AMDGPUAnnotateUniformValuesLegacyPassID;

void initializeAMDGPUCodeGenPreparePass(PassRegistry&);
extern char &AMDGPUCodeGenPrepareID;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUSetWavePriorityPass`, `AMDGPUPrintfRuntimeBindingPass`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUSetWavePriorityPass`, `AMDGPUPrintfRuntimeBindingPass`。

### Lines 503-535: Declares class AMDGPURewriteUndefForPHIPass
```cpp
void initializeAMDGPURemoveIncompatibleFunctionsLegacyPass(PassRegistry &);
extern char &AMDGPURemoveIncompatibleFunctionsID;

void initializeAMDGPULateCodeGenPrepareLegacyPass(PassRegistry &);
extern char &AMDGPULateCodeGenPrepareLegacyID;

FunctionPass *createAMDGPURewriteUndefForPHILegacyPass();
void initializeAMDGPURewriteUndefForPHILegacyPass(PassRegistry &);
extern char &AMDGPURewriteUndefForPHILegacyPassID;

class AMDGPURewriteUndefForPHIPass
    : public OptionalPassInfoMixin<AMDGPURewriteUndefForPHIPass> {
public:
  AMDGPURewriteUndefForPHIPass() = default;
  PreservedAnalyses run(Function &F, FunctionAnalysisManager &AM);
};

class SIAnnotateControlFlowPass
    : public OptionalPassInfoMixin<SIAnnotateControlFlowPass> {
private:
  const AMDGPUTargetMachine &TM;

public:
  SIAnnotateControlFlowPass(const AMDGPUTargetMachine &TM) : TM(TM) {}
  PreservedAnalyses run(Function &F, FunctionAnalysisManager &AM);
};

void initializeSIAnnotateControlFlowLegacyPass(PassRegistry &);
extern char &SIAnnotateControlFlowLegacyPassID;

void initializeSIMemoryLegalizerLegacyPass(PassRegistry &);
extern char &SIMemoryLegalizerID;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPURewriteUndefForPHIPass`, `SIAnnotateControlFlowPass`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPURewriteUndefForPHIPass`, `SIAnnotateControlFlowPass`。

### Lines 536-567: Declares initializeSIModeRegisterLegacyPass
```cpp
void initializeSIModeRegisterLegacyPass(PassRegistry &);
extern char &SIModeRegisterID;

void initializeAMDGPUInsertDelayAluLegacyPass(PassRegistry &);
extern char &AMDGPUInsertDelayAluID;

void initializeAMDGPULowerVGPREncodingLegacyPass(PassRegistry &);
extern char &AMDGPULowerVGPREncodingLegacyID;

void initializeSIInsertHardClausesLegacyPass(PassRegistry &);
extern char &SIInsertHardClausesID;

void initializeSIInsertWaitcntsLegacyPass(PassRegistry &);
extern char &SIInsertWaitcntsID;

void initializeSIFormMemoryClausesLegacyPass(PassRegistry &);
extern char &SIFormMemoryClausesID;

void initializeSIPostRABundlerLegacyPass(PassRegistry &);
extern char &SIPostRABundlerLegacyID;

void initializeGCNCreateVOPDLegacyPass(PassRegistry &);
extern char &GCNCreateVOPDID;

void initializeAMDGPUUnifyDivergentExitNodesLegacyPass(PassRegistry &);
extern char &AMDGPUUnifyDivergentExitNodesID;

ImmutablePass *createAMDGPUAAWrapperPass();
void initializeAMDGPUAAWrapperPassPass(PassRegistry&);
ImmutablePass *createAMDGPUExternalAAWrapperPass();
void initializeAMDGPUExternalAAWrapperPass(PassRegistry&);

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 568-600: Declares class AMDGPURewriteAGPRCopyMFMAPass
```cpp
ModulePass *createAMDGPUExportKernelRuntimeHandlesLegacyPass();
void initializeAMDGPUExportKernelRuntimeHandlesLegacyPass(PassRegistry &);
extern char &AMDGPUExportKernelRuntimeHandlesLegacyID;

void initializeGCNNSAReassignLegacyPass(PassRegistry &);
extern char &GCNNSAReassignID;

void initializeGCNPreRALongBranchRegLegacyPass(PassRegistry &);
extern char &GCNPreRALongBranchRegID;

void initializeGCNPreRAOptimizationsLegacyPass(PassRegistry &);
extern char &GCNPreRAOptimizationsID;

FunctionPass *createAMDGPUSetWavePriorityPass();
void initializeAMDGPUSetWavePriorityLegacyPass(PassRegistry &);

void initializeGCNRewritePartialRegUsesLegacyPass(llvm::PassRegistry &);
extern char &GCNRewritePartialRegUsesID;

void initializeAMDGPUWaitSGPRHazardsLegacyPass(PassRegistry &);
extern char &AMDGPUWaitSGPRHazardsLegacyID;

class AMDGPURewriteAGPRCopyMFMAPass
    : public OptionalPassInfoMixin<AMDGPURewriteAGPRCopyMFMAPass> {
public:
  AMDGPURewriteAGPRCopyMFMAPass() = default;
  PreservedAnalyses run(MachineFunction &MF,
                        MachineFunctionAnalysisManager &MFAM);
};

void initializeAMDGPURewriteAGPRCopyMFMALegacyPass(PassRegistry &);
extern char &AMDGPURewriteAGPRCopyMFMALegacyID;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPURewriteAGPRCopyMFMAPass`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPURewriteAGPRCopyMFMAPass`。

### Lines 601-622: Declares struct AMDGPUUniformIntrinsicCombinePass
```cpp
void initializeAMDGPUUniformIntrinsicCombineLegacyPass(PassRegistry &);
extern char &AMDGPUUniformIntrinsicCombineLegacyPassID;
FunctionPass *createAMDGPUUniformIntrinsicCombineLegacyPass();

struct AMDGPUUniformIntrinsicCombinePass
    : public OptionalPassInfoMixin<AMDGPUUniformIntrinsicCombinePass> {
  PreservedAnalyses run(Function &F, FunctionAnalysisManager &AM);
};

namespace AMDGPU {
enum TargetIndex {
  TI_CONSTDATA_START,
  TI_SCRATCH_RSRC_DWORD0,
  TI_SCRATCH_RSRC_DWORD1,
  TI_SCRATCH_RSRC_DWORD2,
  TI_SCRATCH_RSRC_DWORD3
};

static inline bool addrspacesMayAlias(unsigned AS1, unsigned AS2) {
  if (AS1 > AMDGPUAS::MAX_AMDGPU_ADDRESS || AS2 > AMDGPUAS::MAX_AMDGPU_ADDRESS)
    return true;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUUniformIntrinsicCombinePass`, `TargetIndex`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUUniformIntrinsicCombinePass`, `TargetIndex`。

### Lines 623-647: Preprocessor guards and macros
```cpp
  // clang-format off
  static const bool ASAliasRules[][AMDGPUAS::MAX_AMDGPU_ADDRESS + 1] = {
    /*                       Flat   Global Region  Local Constant Private Const32 BufFatPtr BufRsrc BufStrdPtr */
    /* Flat     */            {true,  true,  false, true,  true,  true,  true,  true,  true,  true},
    /* Global   */            {true,  true,  false, false, true,  false, true,  true,  true,  true},
    /* Region   */            {false, false, true,  false, false, false, false, false, false, false},
    /* Local    */            {true,  false, false, true,  false, false, false, false, false, false},
    /* Constant */            {true,  true,  false, false, false, false, true,  true,  true,  true},
    /* Private  */            {true,  false, false, false, false, true,  false, false, false, false},
    /* Constant 32-bit */     {true,  true,  false, false, true,  false, false, true,  true,  true},
    /* Buffer Fat Ptr  */     {true,  true,  false, false, true,  false, true,  true,  true,  true},
    /* Buffer Resource */     {true,  true,  false, false, true,  false, true,  true,  true,  true},
    /* Buffer Strided Ptr  */ {true,  true,  false, false, true,  false, true,  true,  true,  true},
  };
  // clang-format on
  static_assert(std::size(ASAliasRules) == AMDGPUAS::MAX_AMDGPU_ADDRESS + 1);

  return ASAliasRules[AS1][AS2];
}

}

} // End namespace llvm

#endif
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled. Main symbols: `std::size`.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。 主要符号：`std::size`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ header
- **Primary symbols / 主要符号**: `AMDGPUTargetMachine`, `LazyCallGraph`, `GCNTargetMachine`, `TargetMachine`, `AMDGPUSimplifyLibCallsPass`, `AMDGPUImageIntrinsicOptimizerPass`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; lowering / 降低; legalization / 合法化
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"llvm/Analysis/CGSCCPassManager.h"`
- `"llvm/CodeGen/MachinePassManager.h"`
- `"llvm/IR/PassManager.h"`
- `"llvm/Pass.h"`
- `"llvm/Support/AMDGPUAddrSpace.h"`
- `"llvm/Support/CodeGen.h"`
