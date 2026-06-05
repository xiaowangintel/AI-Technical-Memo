# X86CodeGenPassBuilder.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/X86CodeGenPassBuilder.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements X86 target definitions for the core X86 backend. / 实现X86 后端核心中的X86 目标定义。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: File header, licensing, and opening context / 文件头、许可证与开场上下文
```cpp
//===-- X86CodeGenPassBuilder.cpp ---------------------------------*- C++ -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
/// \file
/// This file contains X86 CodeGen pipeline builder.
/// TODO: Port CodeGen passes to new pass manager.
//===----------------------------------------------------------------------===//

#include "X86.h"
#include "X86AsmPrinter.h"
#include "X86ISelDAGToDAG.h"
#include "X86TargetMachine.h"

#include "llvm/CodeGen/AtomicExpand.h"
#include "llvm/CodeGen/EarlyIfConversion.h"
#include "llvm/CodeGen/IndirectBrExpand.h"
#include "llvm/CodeGen/InterleavedAccess.h"
#include "llvm/CodeGen/JMCInstrumenter.h"
#include "llvm/CodeGen/KCFI.h"
#include "llvm/MC/MCStreamer.h"
#include "llvm/Passes/CodeGenPassBuilder.h"
#include "llvm/Passes/PassBuilder.h"
#include "llvm/Support/CodeGen.h"
#include "llvm/Transforms/CFGuard.h"

using namespace llvm;
```
**EN:** This section implements file header, licensing, and opening context for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的文件头、许可证与开场上下文。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 31-60: Namespace scope management / 命名空间作用域管理
```cpp

extern cl::opt<bool> X86EnableMachineCombinerPass;

namespace {

class X86CodeGenPassBuilder
    : public CodeGenPassBuilder<X86CodeGenPassBuilder, X86TargetMachine> {
  using Base = CodeGenPassBuilder<X86CodeGenPassBuilder, X86TargetMachine>;

public:
  explicit X86CodeGenPassBuilder(X86TargetMachine &TM,
                                 const CGPassBuilderOption &Opts,
                                 PassInstrumentationCallbacks *PIC)
      : CodeGenPassBuilder(TM, Opts, PIC) {}

  void addIRPasses(PassManagerWrapper &PMW) const;
  void addPreISel(PassManagerWrapper &PMW) const;
  Error addInstSelector(PassManagerWrapper &PMW) const;
  void addPreLegalizeMachineIR(PassManagerWrapper &PMW) const;
  void addILPOpts(PassManagerWrapper &PMW) const;
  void addPreRegBankSelect(PassManagerWrapper &PMW) const;
  void addMachineSSAOptimization(PassManagerWrapper &PMW) const;
  void addPreRegAlloc(PassManagerWrapper &PMW) const;
  // TODO(boomanaiden154): We need to add addPostFastRegAllocRewrite here once
  // it is available to support AMX.
  void addPostRegAlloc(PassManagerWrapper &PMW) const;
  void addPreSched2(PassManagerWrapper &PMW) const;
  void addPreEmitPass(PassManagerWrapper &PMW) const;
  void addPreEmitPass2(PassManagerWrapper &PMW) const;
  // TODO(boomanaiden154): We need to add addRegAssignAndRewriteOptimized here
```
**EN:** This section implements namespace scope management for the core X86 backend. Key symbols include X86CodeGenPassBuilder. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的命名空间作用域管理。关键符号包括 X86CodeGenPassBuilder。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 61-90: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
  // once it is available to support AMX.
  void addAsmPrinterBegin(PassManagerWrapper &PMW) const;
  void addAsmPrinter(PassManagerWrapper &PMW) const;
  void addAsmPrinterEnd(PassManagerWrapper &PMW) const;
};

void X86CodeGenPassBuilder::addIRPasses(PassManagerWrapper &PMW) const {
  addFunctionPass(AtomicExpandPass(TM), PMW);

  // We add both pass anyway and when these two passes run, one will be a
  // no-op based on the optimization level/attributes.
  addFunctionPass(X86LowerAMXIntrinsicsPass(&TM), PMW);
  addFunctionPass(X86LowerAMXTypePass(&TM), PMW);

  Base::addIRPasses(PMW);

  if (getOptLevel() != CodeGenOptLevel::None) {
    addFunctionPass(InterleavedAccessPass(TM), PMW);
    addFunctionPass(X86PartialReductionPass(&TM), PMW);
  }

  // Add passes that handle indirect branch removal and insertion of a retpoline
  // thunk. These will be a no-op unless a function subtarget has the retpoline
  // feature enabled.
  addFunctionPass(IndirectBrExpandPass(TM), PMW);

  // Add Control Flow Guard checks.
  const Triple &TT = TM.getTargetTriple();
  if (TT.isOSWindows())
    addFunctionPass(CFGuardPass(), PMW);
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 91-120: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp

  if (TM.Options.JMCInstrument) {
    flushFPMsToMPM(PMW);
    addModulePass(JMCInstrumenterPass(), PMW);
  }
}

void X86CodeGenPassBuilder::addPreISel(PassManagerWrapper &PMW) const {
  // Only add this pass for 32-bit x86 Windows.
  const Triple &TT = TM.getTargetTriple();
  if (TT.isOSWindows() && TT.isX86_32()) {
    flushFPMsToMPM(PMW);
    addModulePass(X86WinEHStatePass(), PMW);
  }
}

Error X86CodeGenPassBuilder::addInstSelector(PassManagerWrapper &PMW) const {
  addMachineFunctionPass(X86ISelDAGToDAGPass(TM), PMW);

  // For ELF, cleanup any local-dynamic TLS accesses
  if (TM.getTargetTriple().isOSBinFormatELF() &&
      getOptLevel() != CodeGenOptLevel::None) {
    addMachineFunctionPass(X86CleanupLocalDynamicTLSPass(), PMW);
  }

  addMachineFunctionPass(X86GlobalBaseRegPass(), PMW);
  addMachineFunctionPass(X86ArgumentStackSlotPass(), PMW);
  return Error::success();
}

```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 121-150: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
void X86CodeGenPassBuilder::addPreLegalizeMachineIR(
    PassManagerWrapper &PMW) const {
  addMachineFunctionPass(X86PreLegalizerCombinerPass(), PMW);
}

void X86CodeGenPassBuilder::addILPOpts(PassManagerWrapper &PMW) const {
  addMachineFunctionPass(EarlyIfConverterPass(), PMW);
  if (X86EnableMachineCombinerPass) {
    // TODO(boomanaiden154): Add the MachineCombinerPass here once it has been
    // ported to the new pass manager.
  }
  addMachineFunctionPass(X86CmovConversionPass(), PMW);
}

void X86CodeGenPassBuilder::addPreRegBankSelect(PassManagerWrapper &PMW) const {
  addMachineFunctionPass(X86PostLegalizerCombinerPass(), PMW);
}

void X86CodeGenPassBuilder::addMachineSSAOptimization(
    PassManagerWrapper &PMW) const {
  addMachineFunctionPass(X86DomainReassignmentPass(), PMW);
  Base::addMachineSSAOptimization(PMW);
}

void X86CodeGenPassBuilder::addPreRegAlloc(PassManagerWrapper &PMW) const {
  if (getOptLevel() != CodeGenOptLevel::None) {
    addMachineFunctionPass(LiveRangeShrinkPass(), PMW);
    addMachineFunctionPass(X86FixupSetCCPass(), PMW);
    addMachineFunctionPass(X86CallFrameOptimizationPass(), PMW);
    addMachineFunctionPass(X86AvoidStoreForwardingBlocksPass(), PMW);
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. Key symbols include X86CodeGenPassBuilder::addPreLegalizeMachineIR. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。关键符号包括 X86CodeGenPassBuilder::addPreLegalizeMachineIR。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 151-180: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
  }

  addMachineFunctionPass(X86SuppressAPXForRelocationPass(), PMW);
  addMachineFunctionPass(X86SpeculativeLoadHardeningPass(), PMW);
  addMachineFunctionPass(X86FlagsCopyLoweringPass(), PMW);
  addMachineFunctionPass(X86DynAllocaExpanderPass(), PMW);

  if (getOptLevel() != CodeGenOptLevel::None)
    addMachineFunctionPass(X86PreTileConfigPass(), PMW);
  else
    addMachineFunctionPass(X86FastPreTileConfigPass(), PMW);
}

void X86CodeGenPassBuilder::addPostRegAlloc(PassManagerWrapper &PMW) const {
  addMachineFunctionPass(X86LowerTileCopyPass(), PMW);
  addMachineFunctionPass(X86FPStackifierPass(), PMW);
  // When -O0 is enabled, the Load Value Injection Hardening pass will fall back
  // to using the Speculative Execution Side Effect Suppression pass for
  // mitigation. This is to prevent slow downs due to
  // analyses needed by the LVIHardening pass when compiling at -O0.
  if (getOptLevel() != CodeGenOptLevel::None) {
    addMachineFunctionPass(X86LoadValueInjectionLoadHardeningPass(), PMW);
  }
}

void X86CodeGenPassBuilder::addPreSched2(PassManagerWrapper &PMW) const {
  addMachineFunctionPass(X86ExpandPseudoPass(), PMW);
  addMachineFunctionPass(MachineKCFIPass(), PMW);
}

```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 181-210: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
void X86CodeGenPassBuilder::addPreEmitPass(PassManagerWrapper &PMW) const {
  if (getOptLevel() != CodeGenOptLevel::None) {
    // TODO(boomanaiden154): Add X86ExecutionDomainFixPass here once it has
    // been ported.
    addMachineFunctionPass(BreakFalseDepsPass(), PMW);
  }

  addMachineFunctionPass(X86IndirectBranchTrackingPass(), PMW);
  addMachineFunctionPass(X86InsertVZeroUpperPass(), PMW);

  if (getOptLevel() != CodeGenOptLevel::None) {
    addMachineFunctionPass(X86FixupBWInstsPass(), PMW);
    // TODO(boomanaiden154): Add X86PadShortFunctionsPass here once it has been
    // ported.
    addMachineFunctionPass(X86FixupLEAsPass(), PMW);
    addMachineFunctionPass(X86FixupInstTuningPass(), PMW);
    addMachineFunctionPass(X86FixupVectorConstantsPass(), PMW);
  }
  addMachineFunctionPass(X86CompressEVEXPass(), PMW);
  addMachineFunctionPass(X86InsertX87WaitPass(), PMW);
}

void X86CodeGenPassBuilder::addPreEmitPass2(PassManagerWrapper &PMW) const {
  const Triple &TT = TM.getTargetTriple();
  const MCAsmInfo &MAI = TM.getMCAsmInfo();

  // The X86 Speculative Execution Pass must run after all control
  // flow graph modifying passes. As a result it was listed to run right before
  // the X86 Retpoline Thunks pass. The reason it must run after control flow
  // graph modifications is that the model of LFENCE in LLVM has to be updated
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. Key symbols include X86CodeGenPassBuilder::addPreEmitPass. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。关键符号包括 X86CodeGenPassBuilder::addPreEmitPass。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 211-240: Control flow, matching, and data updates / 控制流、匹配与数据更新
```cpp
  // (FIXME: https://bugs.llvm.org/show_bug.cgi?id=45167). Currently the
  // placement of this pass was hand checked to ensure that the subsequent
  // passes don't move the code around the LFENCEs in a way that will hurt the
  // correctness of this pass. This placement has been shown to work based on
  // hand inspection of the codegen output.
  addMachineFunctionPass(X86SpeculativeExecutionSideEffectSuppressionPass(),
                         PMW);
  // TODO(boomanaiden154): Add X86IndirectThunksPass here
  // once it has been ported.
  addMachineFunctionPass(X86ReturnThunksPass(), PMW);

  // Insert extra int3 instructions after trailing call instructions to avoid
  // issues in the unwinder.
  if (TT.isOSWindows() && TT.isX86_64())
    addMachineFunctionPass(X86AvoidTrailingCallPass(), PMW);

  // Verify basic block incoming and outgoing cfa offset and register values and
  // correct CFA calculation rule where needed by inserting appropriate CFI
  // instructions.
  if (!TT.isOSDarwin() &&
      (!TT.isOSWindows() ||
       MAI.getExceptionHandlingType() == ExceptionHandling::DwarfCFI)) {
    // TODO(boomanaiden154): Add CFInstrInserterPass here when it has been
    // ported.
  }

  if (TT.isOSWindows()) {
    // Identify valid longjmp targets for Windows Control Flow Guard.
    // TODO(boomanaiden154): Add CFGuardLongjmpPass here when it has been
    // ported.
```
**EN:** This section implements control flow, matching, and data updates for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的控制流、匹配与数据更新。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 241-270: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
    // Identify valid eh continuation targets for Windows EHCont Guard.
    // TODO(boomanaiden154): Add EHContGuardTargetsPass when it has been
    // ported.
  }

  addMachineFunctionPass(X86LoadValueInjectionRetHardeningPass(), PMW);

  // Insert pseudo probe annotation for callsite profiling
  // TODO(boomanaiden154): Add PseudoProberInserterPass here once it has been
  // ported.

  // KCFI indirect call checks are lowered to a bundle, and on Darwin platforms,
  // also CALL_RVMARKER.
  // TODO(boomanaiden154): Add UnpackMachineBundlesPass here once it has been
  // ported.

  // Analyzes and emits pseudos to support Win x64 Unwind V2. This pass must run
  // after all real instructions have been added to the epilog.
  if (TT.isOSWindows() && TT.isX86_64()) {
    addMachineFunctionPass(X86WinEHUnwindV2Pass(), PMW);
  }
}

void X86CodeGenPassBuilder::addAsmPrinterBegin(PassManagerWrapper &PMW) const {
  addModulePass(X86AsmPrinterBeginPass(), PMW, /*Force=*/true);
}

void X86CodeGenPassBuilder::addAsmPrinter(PassManagerWrapper &PMW) const {
  addMachineFunctionPass(X86AsmPrinterPass(), PMW);
}
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 271-299: Preprocessor directives and structural setup / 预处理指令与结构初始化
```cpp

void X86CodeGenPassBuilder::addAsmPrinterEnd(PassManagerWrapper &PMW) const {
  addModulePass(X86AsmPrinterEndPass(), PMW, /*Force=*/true);
}

} // namespace

void X86TargetMachine::registerPassBuilderCallbacks(PassBuilder &PB) {
#define GET_PASS_REGISTRY "X86PassRegistry.def"
#include "llvm/Passes/TargetPassRegistry.inc"
  // TODO(boomanaiden154): Move this into the base CodeGenPassBuilder once all
  // targets that currently implement it have a ported asm-printer pass.
  if (PIC) {
    PIC->addClassToPassName(X86AsmPrinterBeginPass::name(),
                            "x86-asm-printer-begin");
    PIC->addClassToPassName(X86AsmPrinterPass::name(), "x86-asm-printer");
    PIC->addClassToPassName(X86AsmPrinterEndPass::name(),
                            "x86-asm-printer-end");
  }
}

Error X86TargetMachine::buildCodeGenPipeline(
    ModulePassManager &MPM, ModuleAnalysisManager &MAM, raw_pwrite_stream &Out,
    raw_pwrite_stream *DwoOut, CodeGenFileType FileType,
    const CGPassBuilderOption &Opt, MCContext &Ctx,
    PassInstrumentationCallbacks *PIC) {
  auto CGPB = X86CodeGenPassBuilder(*this, Opt, PIC);
  return CGPB.buildPipeline(MPM, MAM, Out, DwoOut, FileType, Ctx);
}
```
**EN:** This section implements preprocessor directives and structural setup for the core X86 backend. Key symbols include X86CodeGenPassBuilder::addAsmPrinterEnd. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的预处理指令与结构初始化。关键符号包括 X86CodeGenPassBuilder::addAsmPrinterEnd。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

## Key Concepts / 关键概念
- Primary topic: X86 target definitions. / 核心主题：X86 目标定义。
- Subsystem: the core X86 backend. / 所属子系统：X86 后端核心。
- Notable symbols: X86CodeGenPassBuilder. / 重要符号：X86CodeGenPassBuilder。
- The file emphasizes executable backend logic and target-specific decision making. / 该文件以可执行后端逻辑和目标特定决策为主。
- The implementation follows LLVM namespace conventions. / 实现遵循 LLVM 的命名空间约定。

## Dependencies / 依赖关系
- Direct includes: X86.h, X86AsmPrinter.h, X86ISelDAGToDAG.h, X86TargetMachine.h, llvm/CodeGen/AtomicExpand.h, llvm/CodeGen/EarlyIfConversion.h, llvm/CodeGen/IndirectBrExpand.h, llvm/CodeGen/InterleavedAccess.h, llvm/CodeGen/JMCInstrumenter.h, llvm/CodeGen/KCFI.h. / 直接包含：X86.h, X86AsmPrinter.h, X86ISelDAGToDAG.h, X86TargetMachine.h, llvm/CodeGen/AtomicExpand.h, llvm/CodeGen/EarlyIfConversion.h, llvm/CodeGen/IndirectBrExpand.h, llvm/CodeGen/InterleavedAccess.h, llvm/CodeGen/JMCInstrumenter.h, llvm/CodeGen/KCFI.h。
- Additional include dependencies: 6 more headers. / 额外包含依赖：还有 6 个头文件。
- Integrates with shared LLVM backend layers such as IR, CodeGen, MC, and X86 target support as needed. / 按需与 LLVM 的 IR、CodeGen、MC 和 X86 目标支持层集成。
