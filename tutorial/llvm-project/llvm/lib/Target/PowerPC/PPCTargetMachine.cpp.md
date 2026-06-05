# PPCTargetMachine.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCTargetMachine.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file provides target machine configuration for the PowerPC backend.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCTargetMachine.cpp`，主要负责 PowerPC 后端的目标机器配置。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- PPCTargetMachine.cpp - Define TargetMachine for PowerPC -----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-10

```cpp
//===----------------------------------------------------------------------===//
//
// Top-level implementation for the PowerPC target.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Top-level implementation for the PowerPC target.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Top-level implementation for the PowerPC target.”。

### Lines 11-17

```cpp
//===----------------------------------------------------------------------===//

#include "PPCTargetMachine.h"
#include "MCTargetDesc/PPCMCTargetDesc.h"
#include "PPC.h"
#include "PPCMachineFunctionInfo.h"
#include "PPCMachineScheduler.h"
```
- **EN**: Pulls in direct dependencies required by this target machine configuration, so later declarations can reuse LLVM infrastructure and target-specific helpers. Scheduling or processor-model metadata is part of this section.
- **CN**: 这一段引入该目标机器配置所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这一段包含调度或处理器模型元数据。

### Lines 18-24

```cpp
#include "PPCMacroFusion.h"
#include "PPCSubtarget.h"
#include "PPCTargetObjectFile.h"
#include "PPCTargetTransformInfo.h"
#include "TargetInfo/PowerPCTargetInfo.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/Analysis/TargetTransformInfo.h"
```
- **EN**: Pulls in direct dependencies required by this target machine configuration, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该目标机器配置所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 25-31

```cpp
#include "llvm/CodeGen/GlobalISel/IRTranslator.h"
#include "llvm/CodeGen/GlobalISel/InstructionSelect.h"
#include "llvm/CodeGen/GlobalISel/Legalizer.h"
#include "llvm/CodeGen/GlobalISel/Localizer.h"
#include "llvm/CodeGen/GlobalISel/RegBankSelect.h"
#include "llvm/CodeGen/MachineScheduler.h"
#include "llvm/CodeGen/Passes.h"
```
- **EN**: Pulls in direct dependencies required by this target machine configuration, so later declarations can reuse LLVM infrastructure and target-specific helpers. This range participates in the GlobalISel pipeline.
- **CN**: 这一段引入该目标机器配置所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这一段参与 GlobalISel 流水线。

### Lines 32-38

```cpp
#include "llvm/CodeGen/TargetPassConfig.h"
#include "llvm/IR/Attributes.h"
#include "llvm/IR/DataLayout.h"
#include "llvm/IR/Function.h"
#include "llvm/InitializePasses.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Pass.h"
```
- **EN**: Pulls in direct dependencies required by this target machine configuration, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该目标机器配置所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 39-45

```cpp
#include "llvm/Support/CodeGen.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/Compiler.h"
#include "llvm/Target/TargetLoweringObjectFile.h"
#include "llvm/Target/TargetOptions.h"
#include "llvm/TargetParser/Triple.h"
#include "llvm/Transforms/Scalar.h"
```
- **EN**: Pulls in direct dependencies required by this target machine configuration, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该目标机器配置所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 46-83

```cpp
#include <cassert>
#include <memory>
#include <optional>
#include <string>

using namespace llvm;


static cl::opt<bool>
    EnableBranchCoalescing("enable-ppc-branch-coalesce", cl::Hidden,
                           cl::desc("enable coalescing of duplicate branches for PPC"));
static cl::
opt<bool> DisableCTRLoops("disable-ppc-ctrloops", cl::Hidden,
                        cl::desc("Disable CTR loops for PPC"));

static cl::
opt<bool> DisableInstrFormPrep("disable-ppc-instr-form-prep", cl::Hidden,
                            cl::desc("Disable PPC loop instr form prep"));

static cl::opt<bool>
VSXFMAMutateEarly("schedule-ppc-vsx-fma-mutation-early",
  cl::Hidden, cl::desc("Schedule VSX FMA instruction mutation early"));

static cl::
opt<bool> DisableVSXSwapRemoval("disable-ppc-vsx-swap-removal", cl::Hidden,
                                cl::desc("Disable VSX Swap Removal for PPC"));

static cl::
opt<bool> DisableMIPeephole("disable-ppc-peephole", cl::Hidden,
                            cl::desc("Disable machine peepholes for PPC"));

static cl::opt<bool>
EnableGEPOpt("ppc-gep-opt", cl::Hidden,
             cl::desc("Enable optimizations on complex GEPs"),
             cl::init(true));

static cl::opt<bool>
EnablePrefetch("enable-ppc-prefetching",
```
- **EN**: Pulls in direct dependencies required by this target machine configuration, so later declarations can reuse LLVM infrastructure and target-specific helpers. Scheduling or processor-model metadata is part of this section.
- **CN**: 这一段引入该目标机器配置所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这一段包含调度或处理器模型元数据。

### Lines 84-121

```cpp
                  cl::desc("enable software prefetching on PPC"),
                  cl::init(false), cl::Hidden);

static cl::opt<bool>
EnableExtraTOCRegDeps("enable-ppc-extra-toc-reg-deps",
                      cl::desc("Add extra TOC register dependencies"),
                      cl::init(true), cl::Hidden);

static cl::opt<bool>
EnableMachineCombinerPass("ppc-machine-combiner",
                          cl::desc("Enable the machine combiner pass"),
                          cl::init(true), cl::Hidden);

static cl::opt<bool>
  ReduceCRLogical("ppc-reduce-cr-logicals",
                  cl::desc("Expand eligible cr-logical binary ops to branches"),
                  cl::init(true), cl::Hidden);

static cl::opt<bool> EnablePPCGenScalarMASSEntries(
    "enable-ppc-gen-scalar-mass", cl::init(false),
    cl::desc("Enable lowering math functions to their corresponding MASS "
             "(scalar) entries"),
    cl::Hidden);

static cl::opt<bool>
    EnableGlobalMerge("ppc-global-merge", cl::Hidden, cl::init(false),
                      cl::desc("Enable the global merge pass"));

static cl::opt<unsigned>
    GlobalMergeMaxOffset("ppc-global-merge-max-offset", cl::Hidden,
                         cl::init(0x7fff),
                         cl::desc("Maximum global merge offset"));

extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
LLVMInitializePowerPCTarget() {
  // Register the targets
  RegisterTargetMachine<PPCTargetMachine> A(getThePPC32Target());
  RegisterTargetMachine<PPCTargetMachine> B(getThePPC32LETarget());
```
- **EN**: Implements helper routine(s) `desc`, `init`, `EnableExtraTOCRegDeps` for this portion of the PowerPC backend target machine configuration.
- **CN**: 这里实现了 PowerPC 后端该部分目标机器配置所需的辅助例程 `desc`, `init`, `EnableExtraTOCRegDeps`。

### Lines 122-159

```cpp
  RegisterTargetMachine<PPCTargetMachine> C(getThePPC64Target());
  RegisterTargetMachine<PPCTargetMachine> D(getThePPC64LETarget());

  PassRegistry &PR = *PassRegistry::getPassRegistry();
#ifndef NDEBUG
  initializePPCCTRLoopsVerifyPass(PR);
#endif
  initializePPCLoopInstrFormPrepPass(PR);
  initializePPCTOCRegDepsPass(PR);
  initializePPCEarlyReturnPass(PR);
  initializePPCVSXWACCCopyPass(PR);
  initializePPCVSXFMAMutatePass(PR);
  initializePPCVSXSwapRemovalPass(PR);
  initializePPCReduceCRLogicalsPass(PR);
  initializePPCBSelPass(PR);
  initializePPCBranchCoalescingPass(PR);
  initializePPCBoolRetToIntPass(PR);
  initializePPCPreEmitPeepholePass(PR);
  initializePPCTLSDynamicCallPass(PR);
  initializePPCMIPeepholePass(PR);
  initializePPCLowerMASSVEntriesPass(PR);
  initializePPCGenScalarMASSEntriesPass(PR);
  initializePPCExpandAtomicPseudoPass(PR);
  initializeGlobalISel(PR);
  initializePPCCTRLoopsPass(PR);
  initializePPCDAGToDAGISelLegacyPass(PR);
  initializePPCPrepareIFuncsOnAIXPass(PR);
  initializePPCLinuxAsmPrinterPass(PR);
  initializePPCAIXAsmPrinterPass(PR);
}

static std::string computeFSAdditions(StringRef FS, CodeGenOptLevel OL,
                                      const Triple &TT) {
  std::string FullFS = std::string(FS);

  // Make sure 64-bit features are available when CPUname is generic
  if (TT.getArch() == Triple::ppc64 || TT.getArch() == Triple::ppc64le) {
    if (!FullFS.empty())
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. This range participates in the GlobalISel pipeline.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 这一段参与 GlobalISel 流水线。

### Lines 160-197

```cpp
      FullFS = "+64bit," + FullFS;
    else
      FullFS = "+64bit";
  }

  if (OL >= CodeGenOptLevel::Default) {
    if (!FullFS.empty())
      FullFS = "+crbits," + FullFS;
    else
      FullFS = "+crbits";
  }

  if (OL != CodeGenOptLevel::None) {
    if (!FullFS.empty())
      FullFS = "+invariant-function-descriptors," + FullFS;
    else
      FullFS = "+invariant-function-descriptors";
  }

  if (TT.isOSAIX()) {
    if (!FullFS.empty())
      FullFS = "+aix," + FullFS;
    else
      FullFS = "+aix";
  }

  return FullFS;
}

static std::unique_ptr<TargetLoweringObjectFile> createTLOF(const Triple &TT) {
  if (TT.isOSAIX())
    return std::make_unique<TargetLoweringObjectFileXCOFF>();

  return std::make_unique<PPC64LinuxTargetObjectFile>();
}

static PPCTargetMachine::PPCABI computeTargetABI(const Triple &TT,
                                                 const TargetOptions &Options) {
```
- **EN**: Implements helper routine(s) `empty`, `isOSAIX`, `createTLOF` for this portion of the PowerPC backend target machine configuration.
- **CN**: 这里实现了 PowerPC 后端该部分目标机器配置所需的辅助例程 `empty`, `isOSAIX`, `createTLOF`。

### Lines 198-235

```cpp
  if (Options.MCOptions.getABIName().starts_with("elfv1"))
    return PPCTargetMachine::PPC_ABI_ELFv1;
  else if (Options.MCOptions.getABIName().starts_with("elfv2"))
    return PPCTargetMachine::PPC_ABI_ELFv2;

  assert(Options.MCOptions.getABIName().empty() &&
         "Unknown target-abi option!");

  switch (TT.getArch()) {
  case Triple::ppc64le:
    return PPCTargetMachine::PPC_ABI_ELFv2;
  case Triple::ppc64:
    if (TT.isPPC64ELFv2ABI())
      return PPCTargetMachine::PPC_ABI_ELFv2;
    else
      return PPCTargetMachine::PPC_ABI_ELFv1;
  default:
    return PPCTargetMachine::PPC_ABI_UNKNOWN;
  }
}

static Reloc::Model getEffectiveRelocModel(const Triple &TT,
                                           std::optional<Reloc::Model> RM) {
  if (TT.isOSAIX() && RM && *RM != Reloc::PIC_)
    report_fatal_error("invalid relocation model, AIX only supports PIC",
                       false);

  if (RM)
    return *RM;

  // Big Endian PPC and AIX default to PIC.
  if (TT.getArch() == Triple::ppc64 || TT.isOSAIX())
    return Reloc::PIC_;

  // Rest are static by default.
  return Reloc::Static;
}
```
- **EN**: Implements helper routine(s) `getABIName`, `starts_with`, `empty` for this portion of the PowerPC backend target machine configuration.
- **CN**: 这里实现了 PowerPC 后端该部分目标机器配置所需的辅助例程 `getABIName`, `starts_with`, `empty`。

### Lines 236-273

```cpp
static CodeModel::Model
getEffectivePPCCodeModel(const Triple &TT, std::optional<CodeModel::Model> CM,
                         bool JIT) {
  if (CM) {
    if (*CM == CodeModel::Tiny)
      report_fatal_error("Target does not support the tiny CodeModel", false);
    if (*CM == CodeModel::Kernel)
      report_fatal_error("Target does not support the kernel CodeModel", false);
    return *CM;
  }

  if (JIT)
    return CodeModel::Small;
  if (TT.isOSAIX())
    return CodeModel::Small;

  assert(TT.isOSBinFormatELF() && "All remaining PPC OSes are ELF based.");

  if (TT.isArch32Bit())
    return CodeModel::Small;

  assert(TT.isArch64Bit() && "Unsupported PPC architecture.");
  return CodeModel::Medium;
}


static ScheduleDAGInstrs *createPPCMachineScheduler(MachineSchedContext *C) {
  const PPCSubtarget &ST = C->MF->getSubtarget<PPCSubtarget>();
  ScheduleDAGMILive *DAG = ST.usePPCPreRASchedStrategy()
                               ? createSchedLive<PPCPreRASchedStrategy>(C)
                               : createSchedLive<GenericScheduler>(C);
  // add DAG Mutations here.
  if (ST.hasStoreFusion())
    DAG->addMutation(createStoreClusterDAGMutation(DAG->TII, DAG->TRI));
  if (ST.hasFusion())
    DAG->addMutation(createPowerPCMacroFusionDAGMutation());

  return DAG;
```
- **EN**: Implements helper routine(s) `getEffectivePPCCodeModel`, `report_fatal_error`, `isOSAIX` for this portion of the PowerPC backend target machine configuration. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 PowerPC 后端该部分目标机器配置所需的辅助例程 `getEffectivePPCCodeModel`, `report_fatal_error`, `isOSAIX`。 子目标特性裁剪会影响这里的行为。

### Lines 274-311

```cpp
}

static ScheduleDAGInstrs *
createPPCPostMachineScheduler(MachineSchedContext *C) {
  const PPCSubtarget &ST = C->MF->getSubtarget<PPCSubtarget>();
  ScheduleDAGMI *DAG = ST.usePPCPostRASchedStrategy()
                           ? createSchedPostRA<PPCPostRASchedStrategy>(C)
                           : createSchedPostRA<PostGenericScheduler>(C);
  // add DAG Mutations here.
  if (ST.hasStoreFusion())
    DAG->addMutation(createStoreClusterDAGMutation(DAG->TII, DAG->TRI));
  if (ST.hasFusion())
    DAG->addMutation(createPowerPCMacroFusionDAGMutation());
  return DAG;
}

// The FeatureString here is a little subtle. We are modifying the feature
// string with what are (currently) non-function specific overrides as it goes
// into the CodeGenTargetMachineImpl constructor and then using the stored value
// in the Subtarget constructor below it.
PPCTargetMachine::PPCTargetMachine(const Target &T, const Triple &TT,
                                   StringRef CPU, StringRef FS,
                                   const TargetOptions &Options,
                                   std::optional<Reloc::Model> RM,
                                   std::optional<CodeModel::Model> CM,
                                   CodeGenOptLevel OL, bool JIT)
    : CodeGenTargetMachineImpl(T,
                               TT.computeDataLayout(Options.MCOptions.ABIName),
                               TT, CPU, computeFSAdditions(FS, OL, TT), Options,
                               getEffectiveRelocModel(TT, RM),
                               getEffectivePPCCodeModel(TT, CM, JIT), OL),
      TLOF(createTLOF(getTargetTriple())),
      TargetABI(computeTargetABI(TT, Options)),
      Endianness(TT.isLittleEndian() ? Endian::LITTLE : Endian::BIG) {
  initAsmInfo();
}

PPCTargetMachine::~PPCTargetMachine() = default;
```
- **EN**: Implements helper routine(s) `createPPCPostMachineScheduler`, `usePPCPostRASchedStrategy`, `hasStoreFusion` for this portion of the PowerPC backend target machine configuration. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 PowerPC 后端该部分目标机器配置所需的辅助例程 `createPPCPostMachineScheduler`, `usePPCPostRASchedStrategy`, `hasStoreFusion`。 子目标特性裁剪会影响这里的行为。

### Lines 312-349

```cpp

const PPCSubtarget *
PPCTargetMachine::getSubtargetImpl(const Function &F) const {
  Attribute CPUAttr = F.getFnAttribute("target-cpu");
  Attribute TuneAttr = F.getFnAttribute("tune-cpu");
  Attribute FSAttr = F.getFnAttribute("target-features");

  std::string CPU =
      CPUAttr.isValid() ? CPUAttr.getValueAsString().str() : TargetCPU;
  std::string TuneCPU =
      TuneAttr.isValid() ? TuneAttr.getValueAsString().str() : CPU;
  std::string FS =
      FSAttr.isValid() ? FSAttr.getValueAsString().str() : TargetFS;

  // FIXME: This is related to the code below to reset the target options,
  // we need to know whether or not the soft float flag is set on the
  // function before we can generate a subtarget. We also need to use
  // it as a key for the subtarget since that can be the only difference
  // between two functions.
  bool SoftFloat = F.getFnAttribute("use-soft-float").getValueAsBool();
  // If the soft float attribute is set on the function turn on the soft float
  // subtarget feature.
  if (SoftFloat)
    FS += FS.empty() ? "-hard-float" : ",-hard-float";

  auto &I = SubtargetMap[CPU + TuneCPU + FS];
  if (!I) {
    // This needs to be done before we create a new subtarget since any
    // creation will depend on the TM and the code generation flags on the
    // function that reside in TargetOptions.
    resetTargetOptions(F);
    I = std::make_unique<PPCSubtarget>(
        TargetTriple, CPU, TuneCPU,
        // FIXME: It would be good to have the subtarget additions here
        // not necessary. Anything that turns them on/off (overrides) ends
        // up being put at the end of the feature string, but the defaults
        // shouldn't require adding them. Fixing this means pulling Feature64Bit
        // out of most of the target cpus in the .td file and making it set only
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "FIXME: This is related to the code below to reset the target options,". Subtarget feature gating influences the behavior here.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“FIXME: This is related to the code below to reset the target options,”。 子目标特性裁剪会影响这里的行为。

### Lines 350-365

```cpp
        // as part of initialization via the TargetTriple.
        computeFSAdditions(FS, getOptLevel(), getTargetTriple()), *this);
  }
  return I.get();
}

ScheduleDAGInstrs *
PPCTargetMachine::createMachineScheduler(MachineSchedContext *C) const {
  return createPPCMachineScheduler(C);
}

ScheduleDAGInstrs *
PPCTargetMachine::createPostMachineScheduler(MachineSchedContext *C) const {
  return createPPCPostMachineScheduler(C);
}
```
- **EN**: Implements helper routine(s) `computeFSAdditions`, `getOptLevel`, `getTargetTriple` for this portion of the PowerPC backend target machine configuration. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里实现了 PowerPC 后端该部分目标机器配置所需的辅助例程 `computeFSAdditions`, `getOptLevel`, `getTargetTriple`。 这一段包含调度或处理器模型元数据。

### Lines 366-367

```cpp
//===----------------------------------------------------------------------===//
// Pass Pipeline Configuration
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Pass Pipeline Configuration".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Pass Pipeline Configuration”。

### Lines 368-405

```cpp
//===----------------------------------------------------------------------===//

namespace {

/// PPC Code Generator Pass Configuration Options.
class PPCPassConfig : public TargetPassConfig {
public:
  PPCPassConfig(PPCTargetMachine &TM, PassManagerBase &PM)
    : TargetPassConfig(TM, PM) {
    // At any optimization level above -O0 we use the Machine Scheduler and not
    // the default Post RA List Scheduler.
    if (TM.getOptLevel() != CodeGenOptLevel::None)
      substitutePass(&PostRASchedulerID, &PostMachineSchedulerID);
  }

  PPCTargetMachine &getPPCTargetMachine() const {
    return getTM<PPCTargetMachine>();
  }

  void addIRPasses() override;
  bool addPreISel() override;
  bool addILPOpts() override;
  bool addInstSelector() override;
  void addMachineSSAOptimization() override;
  void addPreRegAlloc() override;
  void addPreSched2() override;
  void addPreEmitPass() override;
  void addPreEmitPass2() override;
  // GlobalISEL
  bool addIRTranslator() override;
  bool addLegalizeMachineIR() override;
  bool addRegBankSelect() override;
  bool addGlobalInstructionSelect() override;
};

} // end anonymous namespace

TargetPassConfig *PPCTargetMachine::createPassConfig(PassManagerBase &PM) {
```
- **EN**: Declares a backend-facing type `PPCPassConfig`, `TargetPassConfig`, `getOptLevel` and outlines the API or state that nearby code will rely on. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里声明面向后端的类型 `PPCPassConfig`, `TargetPassConfig`, `getOptLevel`，并勾勒出周边代码会依赖的接口或状态。 这一段包含调度或处理器模型元数据。

### Lines 406-443

```cpp
  return new PPCPassConfig(*this, PM);
}

void PPCPassConfig::addIRPasses() {
  if (TM->getOptLevel() != CodeGenOptLevel::None)
    addPass(createPPCBoolRetToIntPass());
  addPass(createAtomicExpandLegacyPass());

  // Lower generic MASSV routines to PowerPC subtarget-specific entries.
  addPass(createPPCLowerMASSVEntriesPass());

  // Generate PowerPC target-specific entries for scalar math functions
  // that are available in IBM MASS (scalar) library.
  if (TM->getOptLevel() == CodeGenOptLevel::Aggressive &&
      EnablePPCGenScalarMASSEntries) {
    TM->Options.PPCGenScalarMASSEntries = EnablePPCGenScalarMASSEntries;
    addPass(createPPCGenScalarMASSEntriesPass());
  }

  // If explicitly requested, add explicit data prefetch intrinsics.
  if (EnablePrefetch.getNumOccurrences() > 0)
    addPass(createLoopDataPrefetchPass());

  if (TM->getOptLevel() >= CodeGenOptLevel::Default && EnableGEPOpt) {
    // Call SeparateConstOffsetFromGEP pass to extract constants within indices
    // and lower a GEP with multiple indices to either arithmetic operations or
    // multiple GEPs with single index.
    addPass(createSeparateConstOffsetFromGEPPass(true));
    // Call EarlyCSE pass to find and remove subexpressions in the lowered
    // result.
    addPass(createEarlyCSEPass());
    // Do loop invariant code motion in case part of the lowered result is
    // invariant.
    addPass(createLICMPass());
  }

  if (TM->getTargetTriple().isOSAIX())
    addPass(createPPCPrepareIFuncsOnAIXPass());
```
- **EN**: Implements helper routine(s) `PPCPassConfig`, `addIRPasses`, `getOptLevel` for this portion of the PowerPC backend target machine configuration.
- **CN**: 这里实现了 PowerPC 后端该部分目标机器配置所需的辅助例程 `PPCPassConfig`, `addIRPasses`, `getOptLevel`。

### Lines 444-478

```cpp

  TargetPassConfig::addIRPasses();
}

bool PPCPassConfig::addPreISel() {
  // The GlobalMerge pass is intended to be on by default on AIX.
  // Specifying the command line option overrides the AIX default.
  if ((EnableGlobalMerge.getNumOccurrences() > 0)
          ? EnableGlobalMerge
          : getOptLevel() != CodeGenOptLevel::None)
    addPass(createGlobalMergePass(TM, GlobalMergeMaxOffset, false, false, true,
                                  true));

  if (!DisableInstrFormPrep && getOptLevel() != CodeGenOptLevel::None)
    addPass(createPPCLoopInstrFormPrepPass(getPPCTargetMachine()));

  if (!DisableCTRLoops && getOptLevel() != CodeGenOptLevel::None)
    addPass(createHardwareLoopsLegacyPass());

  return false;
}

bool PPCPassConfig::addILPOpts() {
  addPass(&EarlyIfConverterLegacyID);

  if (EnableMachineCombinerPass)
    addPass(&MachineCombinerID);

  return true;
}

bool PPCPassConfig::addInstSelector() {
  // Install an instruction selector.
  addPass(createPPCISelDag(getPPCTargetMachine(), getOptLevel()));
```
- **EN**: Implements helper routine(s) `addIRPasses`, `addPreISel`, `getNumOccurrences` for this portion of the PowerPC backend target machine configuration.
- **CN**: 这里实现了 PowerPC 后端该部分目标机器配置所需的辅助例程 `addIRPasses`, `addPreISel`, `getNumOccurrences`。

### Lines 479-516

```cpp
#ifndef NDEBUG
  if (!DisableCTRLoops && getOptLevel() != CodeGenOptLevel::None)
    addPass(createPPCCTRLoopsVerify());
#endif

  addPass(createPPCVSXWACCCopyPass());
  return false;
}

void PPCPassConfig::addMachineSSAOptimization() {
  // Run CTR loops pass before any cfg modification pass to prevent the
  // canonical form of hardware loop from being destroied.
  if (!DisableCTRLoops && getOptLevel() != CodeGenOptLevel::None)
    addPass(createPPCCTRLoopsPass());

  // PPCBranchCoalescingPass need to be done before machine sinking
  // since it merges empty blocks.
  if (EnableBranchCoalescing && getOptLevel() != CodeGenOptLevel::None)
    addPass(createPPCBranchCoalescingPass());
  TargetPassConfig::addMachineSSAOptimization();
  // For little endian, remove where possible the vector swap instructions
  // introduced at code generation to normalize vector element order.
  if (TM->getTargetTriple().getArch() == Triple::ppc64le &&
      !DisableVSXSwapRemoval)
    addPass(createPPCVSXSwapRemovalPass());
  // Reduce the number of cr-logical ops.
  if (ReduceCRLogical && getOptLevel() != CodeGenOptLevel::None)
    addPass(createPPCReduceCRLogicalsPass());
  // Target-specific peephole cleanups performed after instruction
  // selection.
  if (!DisableMIPeephole) {
    addPass(createPPCMIPeepholePass());
    addPass(&DeadMachineInstructionElimID);
  }
}

void PPCPassConfig::addPreRegAlloc() {
  if (getOptLevel() != CodeGenOptLevel::None) {
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 517-554

```cpp
    insertPass(VSXFMAMutateEarly ? &TwoAddressInstructionPassID
                                 : &MachineSchedulerID,
               &PPCVSXFMAMutateID);
  }

  // FIXME: We probably don't need to run these for -fPIE.
  if (getPPCTargetMachine().isPositionIndependent()) {
    // FIXME: LiveVariables should not be necessary here!
    // PPCTLSDynamicCallPass uses LiveIntervals which previously dependent on
    // LiveVariables. This (unnecessary) dependency has been removed now,
    // however a stage-2 clang build fails without LiveVariables computed here.
    addPass(&LiveVariablesID);
    addPass(createPPCTLSDynamicCallPass());
  }
  if (EnableExtraTOCRegDeps)
    addPass(createPPCTOCRegDepsPass());

  if (getOptLevel() != CodeGenOptLevel::None)
    addPass(&MachinePipelinerID);
}

void PPCPassConfig::addPreSched2() {
  if (getOptLevel() != CodeGenOptLevel::None)
    addPass(&IfConverterID);
}

void PPCPassConfig::addPreEmitPass() {
  addPass(createPPCPreEmitPeepholePass());

  if (getOptLevel() != CodeGenOptLevel::None)
    addPass(createPPCEarlyReturnPass());
}

void PPCPassConfig::addPreEmitPass2() {
  // Schedule the expansion of AMOs at the last possible moment, avoiding the
  // possibility for other passes to break the requirements for forward
  // progress in the LL/SC block.
  addPass(createPPCExpandAtomicPseudoPass());
```
- **EN**: Implements helper routine(s) `insertPass`, `getPPCTargetMachine`, `isPositionIndependent` for this portion of the PowerPC backend target machine configuration. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里实现了 PowerPC 后端该部分目标机器配置所需的辅助例程 `insertPass`, `getPPCTargetMachine`, `isPositionIndependent`。 这一段包含调度或处理器模型元数据。

### Lines 555-592

```cpp
  // Must run branch selection immediately preceding the asm printer.
  addPass(createPPCBranchSelectionPass());
}

TargetTransformInfo
PPCTargetMachine::getTargetTransformInfo(const Function &F) const {
  return TargetTransformInfo(std::make_unique<PPCTTIImpl>(this, F));
}

bool PPCTargetMachine::isLittleEndian() const {
  assert(Endianness != Endian::NOT_DETECTED &&
         "Unable to determine endianness");
  return Endianness == Endian::LITTLE;
}

MachineFunctionInfo *PPCTargetMachine::createMachineFunctionInfo(
    BumpPtrAllocator &Allocator, const Function &F,
    const TargetSubtargetInfo *STI) const {
  return PPCFunctionInfo::create<PPCFunctionInfo>(Allocator, F, STI);
}

static MachineSchedRegistry
PPCPreRASchedRegistry("ppc-prera",
                      "Run PowerPC PreRA specific scheduler",
                      createPPCMachineScheduler);

static MachineSchedRegistry
PPCPostRASchedRegistry("ppc-postra",
                       "Run PowerPC PostRA specific scheduler",
                       createPPCPostMachineScheduler);

// Global ISEL
bool PPCPassConfig::addIRTranslator() {
  addPass(new IRTranslator());
  return false;
}

bool PPCPassConfig::addLegalizeMachineIR() {
```
- **EN**: Implements helper routine(s) `addPass`, `createPPCBranchSelectionPass`, `getTargetTransformInfo` for this portion of the PowerPC backend target machine configuration. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 PowerPC 后端该部分目标机器配置所需的辅助例程 `addPass`, `createPPCBranchSelectionPass`, `getTargetTransformInfo`。 子目标特性裁剪会影响这里的行为。

### Lines 593-605

```cpp
  addPass(new Legalizer());
  return false;
}

bool PPCPassConfig::addRegBankSelect() {
  addPass(new RegBankSelect());
  return false;
}

bool PPCPassConfig::addGlobalInstructionSelect() {
  addPass(new InstructionSelect(getOptLevel()));
  return false;
}
```
- **EN**: Implements helper routine(s) `addPass`, `Legalizer`, `addRegBankSelect` for this portion of the PowerPC backend target machine configuration.
- **CN**: 这里实现了 PowerPC 后端该部分目标机器配置所需的辅助例程 `addPass`, `Legalizer`, `addRegBankSelect`。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- Target machine configuration / 目标机器配置
- GlobalISel pipeline / GlobalISel 流水线
- SelectionDAG lowering / SelectionDAG lowering
- Subtarget features / 子目标特性
- Assembly or MC emission / 汇编或 MC 发射
- Instruction selection or opcode handling / 指令选择或操作码处理
- Type or operation legalization / 类型或操作合法化

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `PPCTargetMachine.h`
- `MCTargetDesc/PPCMCTargetDesc.h`
- `PPC.h`
- `PPCMachineFunctionInfo.h`
- `PPCMachineScheduler.h`
- `PPCMacroFusion.h`
- `PPCSubtarget.h`
- `PPCTargetObjectFile.h`
- `PPCTargetTransformInfo.h`
- `TargetInfo/PowerPCTargetInfo.h`
- `llvm/ADT/StringRef.h`
- `llvm/Analysis/TargetTransformInfo.h`
- `llvm/CodeGen/GlobalISel/IRTranslator.h`
- `llvm/CodeGen/GlobalISel/InstructionSelect.h`
- `llvm/CodeGen/GlobalISel/Legalizer.h`
- `llvm/CodeGen/GlobalISel/Localizer.h`
- `llvm/CodeGen/GlobalISel/RegBankSelect.h`
- `llvm/CodeGen/MachineScheduler.h`
- `llvm/CodeGen/Passes.h`
- `llvm/CodeGen/TargetPassConfig.h`
- `llvm/IR/Attributes.h`
- `llvm/IR/DataLayout.h`
- `llvm/IR/Function.h`
- `llvm/InitializePasses.h`
- `llvm/MC/TargetRegistry.h`
- `llvm/Pass.h`
- `llvm/Support/CodeGen.h`
- `llvm/Support/CommandLine.h`
- `llvm/Support/Compiler.h`
- `llvm/Target/TargetLoweringObjectFile.h`
- `llvm/Target/TargetOptions.h`
- `llvm/TargetParser/Triple.h`
- `llvm/Transforms/Scalar.h`
- `cassert`
- `memory`
- `optional`
- `string`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- LLVM Support utilities / LLVM Support 工具层
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
