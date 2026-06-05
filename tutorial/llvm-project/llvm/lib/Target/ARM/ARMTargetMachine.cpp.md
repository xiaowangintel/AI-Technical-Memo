# ARMTargetMachine.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMTargetMachine.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `ARMTargetMachine` for the ARM backend, focusing on target-machine configuration and pass-pipeline integration.
- 用途 (CN): 实现 ARM 后端中的 `ARMTargetMachine`，重点处理目标机器配置与优化流水线集成。

## Line-by-Line Analysis / 逐行分析

### Lines 1-10
```cpp
//===-- ARMTargetMachine.cpp - Define TargetMachine for ARM ---------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 12-26
```cpp
#include "ARMTargetMachine.h"
#include "ARM.h"
#include "ARMLatencyMutations.h"
#include "ARMMachineFunctionInfo.h"
#include "ARMMacroFusion.h"
#include "ARMSubtarget.h"
#include "ARMTargetObjectFile.h"
#include "ARMTargetTransformInfo.h"
#include "MCTargetDesc/ARMMCTargetDesc.h"
#include "TargetInfo/ARMTargetInfo.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/Analysis/TargetTransformInfo.h"
#include "llvm/CodeGen/ExecutionDomainFix.h"
#include "llvm/CodeGen/GlobalISel/CSEInfo.h"
#include "llvm/CodeGen/GlobalISel/CallLowering.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 27-41
```cpp
#include "llvm/CodeGen/GlobalISel/IRTranslator.h"
#include "llvm/CodeGen/GlobalISel/InstructionSelect.h"
#include "llvm/CodeGen/GlobalISel/Legalizer.h"
#include "llvm/CodeGen/GlobalISel/LegalizerInfo.h"
#include "llvm/CodeGen/GlobalISel/RegBankSelect.h"
#include "llvm/CodeGen/MIRParser/MIParser.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineScheduler.h"
#include "llvm/CodeGen/Passes.h"
#include "llvm/CodeGen/TargetPassConfig.h"
#include "llvm/IR/Attributes.h"
#include "llvm/IR/DataLayout.h"
#include "llvm/IR/Function.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Pass.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 42-56
```cpp
#include "llvm/Passes/PassBuilder.h"
#include "llvm/Support/CodeGen.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/Compiler.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Target/TargetLoweringObjectFile.h"
#include "llvm/Target/TargetOptions.h"
#include "llvm/TargetParser/ARMTargetParser.h"
#include "llvm/TargetParser/TargetParser.h"
#include "llvm/TargetParser/Triple.h"
#include "llvm/Transforms/CFGuard.h"
#include "llvm/Transforms/IPO.h"
#include "llvm/Transforms/Scalar.h"
#include <cassert>
#include <memory>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 57-58
```cpp
#include <optional>
#include <string>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 60-60
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 62-65
```cpp
static cl::opt<bool>
DisableA15SDOptimization("disable-a15-sd-optimization", cl::Hidden,
                   cl::desc("Inhibit optimization of S->D register accesses on A15"),
                   cl::init(false));
```
- EN: Declares `cl::desc`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `cl::desc`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 67-71
```cpp
static cl::opt<bool>
EnableAtomicTidy("arm-atomic-cfg-tidy", cl::Hidden,
                 cl::desc("Run SimplifyCFG after expanding atomic operations"
                          " to make use of cmpxchg flow-based information"),
                 cl::init(true));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 73-76
```cpp
static cl::opt<bool>
EnableARMLoadStoreOpt("arm-load-store-opt", cl::Hidden,
                      cl::desc("Enable ARM load/store optimization pass"),
                      cl::init(true));
```
- EN: Declares `cl::desc`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `cl::desc`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 78-81
```cpp
// FIXME: Unify control over GlobalMerge.
static cl::opt<cl::boolOrDefault>
EnableGlobalMerge("arm-global-merge", cl::Hidden,
                  cl::desc("Enable the global merge pass"));
```
- EN: Declares `cl::desc`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `cl::desc`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 83-85
```cpp
namespace llvm {
  void initializeARMExecutionDomainFixPass(PassRegistry&);
}
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 87-92
```cpp
extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void LLVMInitializeARMTarget() {
  // Register the target.
  RegisterTargetMachine<ARMLETargetMachine> X(getTheARMLETarget());
  RegisterTargetMachine<ARMLETargetMachine> A(getTheThumbLETarget());
  RegisterTargetMachine<ARMBETargetMachine> Y(getTheARMBETarget());
  RegisterTargetMachine<ARMBETargetMachine> B(getTheThumbBETarget());
```
- EN: Implements `LLVMInitializeARMTarget`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `LLVMInitializeARMTarget`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 94-108
```cpp
  PassRegistry &Registry = *PassRegistry::getPassRegistry();
  initializeGlobalISel(Registry);
  initializeARMAsmPrinterPass(Registry);
  initializeARMLoadStoreOptLegacyPass(Registry);
  initializeARMPreAllocLoadStoreOptLegacyPass(Registry);
  initializeARMParallelDSPPass(Registry);
  initializeARMBranchTargetsPass(Registry);
  initializeARMConstantIslandsPass(Registry);
  initializeARMExecutionDomainFixPass(Registry);
  initializeARMExpandPseudoPass(Registry);
  initializeThumb2SizeReducePass(Registry);
  initializeMVEVPTBlockPass(Registry);
  initializeMVETPAndVPTOptimisationsPass(Registry);
  initializeMVETailPredicationPass(Registry);
  initializeARMLowOverheadLoopsPass(Registry);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 109-116
```cpp
  initializeARMBlockPlacementPass(Registry);
  initializeMVEGatherScatterLoweringPass(Registry);
  initializeARMSLSHardeningPass(Registry);
  initializeMVELaneInterleavingPass(Registry);
  initializeARMFixCortexA57AES1742098Pass(Registry);
  initializeARMDAGToDAGISelLegacyPass(Registry);
  initializeMachineKCFILegacyPass(Registry);
}
```
- EN: Declares `initializeARMBlockPlacementPass`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `initializeARMBlockPlacementPass`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 118-124
```cpp
static std::unique_ptr<TargetLoweringObjectFile> createTLOF(const Triple &TT) {
  if (TT.isOSBinFormatMachO())
    return std::make_unique<TargetLoweringObjectFileMachO>();
  if (TT.isOSWindows())
    return std::make_unique<TargetLoweringObjectFileCOFF>();
  return std::make_unique<ARMElfTargetObjectFile>();
}
```
- EN: Implements `createTLOF`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `createTLOF`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 126-130
```cpp
static Reloc::Model getEffectiveRelocModel(const Triple &TT,
                                           std::optional<Reloc::Model> RM) {
  if (!RM)
    // Default relocation model on Darwin is PIC.
    return TT.isOSBinFormatMachO() ? Reloc::PIC_ : Reloc::Static;
```
- EN: Implements `getEffectiveRelocModel`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getEffectiveRelocModel`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 132-134
```cpp
  if (*RM == Reloc::ROPI || *RM == Reloc::RWPI || *RM == Reloc::ROPI_RWPI)
    assert(TT.isOSBinFormatELF() &&
           "ROPI/RWPI currently only supported for ELF");
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 136-138
```cpp
  // DynamicNoPIC is only used on darwin.
  if (*RM == Reloc::DynamicNoPIC && !TT.isOSDarwin())
    return Reloc::Static;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 140-141
```cpp
  return *RM;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 143-156
```cpp
/// Create an ARM architecture model.
///
ARMBaseTargetMachine::ARMBaseTargetMachine(const Target &T, const Triple &TT,
                                           StringRef CPU, StringRef FS,
                                           const TargetOptions &Options,
                                           std::optional<Reloc::Model> RM,
                                           std::optional<CodeModel::Model> CM,
                                           CodeGenOptLevel OL)
    : CodeGenTargetMachineImpl(
          T, TT.computeDataLayout(Options.MCOptions.ABIName), TT, CPU, FS,
          Options, getEffectiveRelocModel(TT, RM),
          getEffectiveCodeModel(CM, CodeModel::Small), OL),
      TargetABI(ARM::computeTargetABI(TT, Options.MCOptions.ABIName)),
      TLOF(createTLOF(getTargetTriple())), isLittle(TT.isLittleEndian()) {
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 158-164
```cpp
  // Default to triple-appropriate float ABI
  if (Options.FloatABIType == FloatABI::Default) {
    if (isTargetHardFloat())
      this->Options.FloatABIType = FloatABI::Hard;
    else
      this->Options.FloatABIType = FloatABI::Soft;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 166-180
```cpp
  // Default to triple-appropriate EABI
  if (Options.EABIVersion == EABI::Default ||
      Options.EABIVersion == EABI::Unknown) {
    // musl is compatible with glibc with regard to EABI version
    if ((TargetTriple.getEnvironment() == Triple::GNUEABI ||
         TargetTriple.getEnvironment() == Triple::GNUEABIT64 ||
         TargetTriple.getEnvironment() == Triple::GNUEABIHF ||
         TargetTriple.getEnvironment() == Triple::GNUEABIHFT64 ||
         TargetTriple.getEnvironment() == Triple::MuslEABI ||
         TargetTriple.getEnvironment() == Triple::MuslEABIHF ||
         TargetTriple.getEnvironment() == Triple::OpenHOS) &&
        !(TargetTriple.isOSWindows() || TargetTriple.isOSDarwin()))
      this->Options.EABIVersion = EABI::GNU;
    else
      this->Options.EABIVersion = EABI::EABI5;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 181-181
```cpp
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 183-186
```cpp
  if (TT.isOSBinFormatMachO()) {
    this->Options.TrapUnreachable = true;
    this->Options.NoTrapAfterNoreturn = true;
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 188-189
```cpp
  // ARM supports the debug entry values.
  setSupportsDebugEntryValues(true);
```
- EN: Declares `setSupportsDebugEntryValues`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `setSupportsDebugEntryValues`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 191-191
```cpp
  initAsmInfo();
```
- EN: Declares `initAsmInfo`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `initAsmInfo`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 193-196
```cpp
  // ARM supports the MachineOutliner.
  setMachineOutliner(true);
  setSupportsDefaultOutlining(true);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 198-198
```cpp
ARMBaseTargetMachine::~ARMBaseTargetMachine() = default;
```
- EN: Declares `ARMBaseTargetMachine::~ARMBaseTargetMachine`, a target-specific routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `ARMBaseTargetMachine::~ARMBaseTargetMachine`，它是一个围绕目标机器策略展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 200-205
```cpp
MachineFunctionInfo *ARMBaseTargetMachine::createMachineFunctionInfo(
    BumpPtrAllocator &Allocator, const Function &F,
    const TargetSubtargetInfo *STI) const {
  return ARMFunctionInfo::create<ARMFunctionInfo>(
      Allocator, F, static_cast<const ARMSubtarget *>(STI));
}
```
- EN: Implements `ARMBaseTargetMachine::createMachineFunctionInfo`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMBaseTargetMachine::createMachineFunctionInfo`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 207-210
```cpp
const ARMSubtarget *
ARMBaseTargetMachine::getSubtargetImpl(const Function &F) const {
  Attribute CPUAttr = F.getFnAttribute("target-cpu");
  Attribute FSAttr = F.getFnAttribute("target-features");
```
- EN: Implements `ARMBaseTargetMachine::getSubtargetImpl`, a query/helper routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMBaseTargetMachine::getSubtargetImpl`，它是一个围绕目标机器策略展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 212-215
```cpp
  std::string CPU =
      CPUAttr.isValid() ? CPUAttr.getValueAsString().str() : TargetCPU;
  std::string FS =
      FSAttr.isValid() ? FSAttr.getValueAsString().str() : TargetFS;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 217-226
```cpp
  // FIXME: This is related to the code below to reset the target options,
  // we need to know whether or not the soft float flag is set on the
  // function before we can generate a subtarget. We also need to use
  // it as a key for the subtarget since that can be the only difference
  // between two functions.
  bool SoftFloat = F.getFnAttribute("use-soft-float").getValueAsBool();
  // If the soft float attribute is set on the function turn on the soft float
  // subtarget feature.
  if (SoftFloat)
    FS += FS.empty() ? "+soft-float" : ",+soft-float";
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 228-232
```cpp
  // Use the optminsize to identify the subtarget, but don't use it in the
  // feature string.
  std::string Key = CPU + FS;
  if (F.hasMinSize())
    Key += "+minsize";
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 234-236
```cpp
  DenormalMode DM = F.getDenormalFPEnv().DefaultMode;
  if (DM != DenormalMode::getIEEE())
    Key += "denormal-fp-math=" + DM.str();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 238-245
```cpp
  auto &I = SubtargetMap[Key];
  if (!I) {
    // This needs to be done before we create a new subtarget since any
    // creation will depend on the TM and the code generation flags on the
    // function that reside in TargetOptions.
    resetTargetOptions(F);
    I = std::make_unique<ARMSubtarget>(TargetTriple, CPU, FS, *this, isLittle,
                                       F.hasMinSize(), DM);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 247-250
```cpp
    if (!I->isThumb() && !I->hasARMOps())
      F.getContext().emitError("Function '" + F.getName() + "' uses ARM "
          "instructions, but the target does not support ARM mode execution.");
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 252-253
```cpp
  return I.get();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 255-258
```cpp
TargetTransformInfo
ARMBaseTargetMachine::getTargetTransformInfo(const Function &F) const {
  return TargetTransformInfo(std::make_unique<ARMTTIImpl>(this, F));
}
```
- EN: Implements `ARMBaseTargetMachine::getTargetTransformInfo`, a query/helper routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMBaseTargetMachine::getTargetTransformInfo`，它是一个围绕目标机器策略展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 260-268
```cpp
ScheduleDAGInstrs *
ARMBaseTargetMachine::createMachineScheduler(MachineSchedContext *C) const {
  ScheduleDAGMILive *DAG = createSchedLive(C);
  // add DAG Mutations here.
  const ARMSubtarget &ST = C->MF->getSubtarget<ARMSubtarget>();
  if (ST.hasFusion())
    DAG->addMutation(createARMMacroFusionDAGMutation());
  return DAG;
}
```
- EN: Implements `ARMBaseTargetMachine::createMachineScheduler`, a query/helper routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMBaseTargetMachine::createMachineScheduler`，它是一个围绕目标机器策略展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 270-280
```cpp
ScheduleDAGInstrs *
ARMBaseTargetMachine::createPostMachineScheduler(MachineSchedContext *C) const {
  ScheduleDAGMI *DAG = createSchedPostRA(C);
  // add DAG Mutations here.
  const ARMSubtarget &ST = C->MF->getSubtarget<ARMSubtarget>();
  if (ST.hasFusion())
    DAG->addMutation(createARMMacroFusionDAGMutation());
  if (auto Mutation = createARMLatencyMutations(ST, C->AA))
    DAG->addMutation(std::move(Mutation));
  return DAG;
}
```
- EN: Implements `ARMBaseTargetMachine::createPostMachineScheduler`, a query/helper routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMBaseTargetMachine::createPostMachineScheduler`，它是一个围绕目标机器策略展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 282-288
```cpp
ARMLETargetMachine::ARMLETargetMachine(const Target &T, const Triple &TT,
                                       StringRef CPU, StringRef FS,
                                       const TargetOptions &Options,
                                       std::optional<Reloc::Model> RM,
                                       std::optional<CodeModel::Model> CM,
                                       CodeGenOptLevel OL, bool JIT)
    : ARMBaseTargetMachine(T, TT, CPU, FS, Options, RM, CM, OL) {}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 290-296
```cpp
ARMBETargetMachine::ARMBETargetMachine(const Target &T, const Triple &TT,
                                       StringRef CPU, StringRef FS,
                                       const TargetOptions &Options,
                                       std::optional<Reloc::Model> RM,
                                       std::optional<CodeModel::Model> CM,
                                       CodeGenOptLevel OL, bool JIT)
    : ARMBaseTargetMachine(T, TT, CPU, FS, Options, RM, CM, OL) {}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 298-298
```cpp
namespace {
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 300-304
```cpp
/// ARM Code Generator Pass Configuration Options.
class ARMPassConfig : public TargetPassConfig {
public:
  ARMPassConfig(ARMBaseTargetMachine &TM, PassManagerBase &PM)
      : TargetPassConfig(TM, PM) {}
```
- EN: Declares `ARMPassConfig`, packaging target-specific state and APIs around `ARMTargetMachine`.
- CN: 这里声明 `ARMPassConfig`，把与 `ARMTargetMachine` 相关的目标特定状态和 API 组织在一起。

### Lines 306-308
```cpp
  ARMBaseTargetMachine &getARMTargetMachine() const {
    return getTM<ARMBaseTargetMachine>();
  }
```
- EN: Implements `getARMTargetMachine`, a query/helper routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getARMTargetMachine`，它是一个围绕目标机器策略展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 310-321
```cpp
  void addIRPasses() override;
  void addCodeGenPrepare() override;
  bool addPreISel() override;
  bool addInstSelector() override;
  bool addIRTranslator() override;
  bool addLegalizeMachineIR() override;
  bool addRegBankSelect() override;
  bool addGlobalInstructionSelect() override;
  void addPreRegAlloc() override;
  void addPreSched2() override;
  void addPreEmitPass() override;
  void addPreEmitPass2() override;
```
- EN: Declares `addIRPasses`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `addIRPasses`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 323-324
```cpp
  std::unique_ptr<CSEConfigBase> getCSEConfig() const override;
};
```
- EN: Declares `getCSEConfig`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getCSEConfig`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 326-334
```cpp
class ARMExecutionDomainFix : public ExecutionDomainFix {
public:
  static char ID;
  ARMExecutionDomainFix() : ExecutionDomainFix(ID, ARM::DPRRegClass) {}
  StringRef getPassName() const override {
    return "ARM Execution Domain Fix";
  }
};
char ARMExecutionDomainFix::ID;
```
- EN: Declares `ARMExecutionDomainFix`, packaging target-specific state and APIs around `ARMTargetMachine`.
- CN: 这里声明 `ARMExecutionDomainFix`，把与 `ARMTargetMachine` 相关的目标特定状态和 API 组织在一起。

### Lines 336-336
```cpp
} // end anonymous namespace
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 338-342
```cpp
INITIALIZE_PASS_BEGIN(ARMExecutionDomainFix, "arm-execution-domain-fix",
                      "ARM Execution Domain Fix", false, false)
INITIALIZE_PASS_DEPENDENCY(ReachingDefInfoWrapperPass)
INITIALIZE_PASS_END(ARMExecutionDomainFix, "arm-execution-domain-fix",
                    "ARM Execution Domain Fix", false, false)
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 344-347
```cpp
void ARMBaseTargetMachine::registerPassBuilderCallbacks(PassBuilder &PB) {
#define GET_PASS_REGISTRY "ARMPassRegistry.def"
#include "llvm/Passes/TargetPassRegistry.inc"
}
```
- EN: Implements `ARMBaseTargetMachine::registerPassBuilderCallbacks`, a target-specific routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMBaseTargetMachine::registerPassBuilderCallbacks`，它是一个围绕目标机器策略展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 349-351
```cpp
TargetPassConfig *ARMBaseTargetMachine::createPassConfig(PassManagerBase &PM) {
  return new ARMPassConfig(*this, PM);
}
```
- EN: Implements `ARMBaseTargetMachine::createPassConfig`, a query/helper routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMBaseTargetMachine::createPassConfig`，它是一个围绕目标机器策略展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 353-355
```cpp
std::unique_ptr<CSEConfigBase> ARMPassConfig::getCSEConfig() const {
  return getStandardCSEConfigForOpt(TM->getOptLevel());
}
```
- EN: Implements `ARMPassConfig::getCSEConfig`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMPassConfig::getCSEConfig`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 357-361
```cpp
void ARMPassConfig::addIRPasses() {
  if (TM->Options.ThreadModel == ThreadModel::Single)
    addPass(createLowerAtomicPass());
  else
    addPass(createAtomicExpandLegacyPass());
```
- EN: Implements `ARMPassConfig::addIRPasses`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMPassConfig::addIRPasses`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 363-372
```cpp
  // Cmpxchg instructions are often used with a subsequent comparison to
  // determine whether it succeeded. We can exploit existing control-flow in
  // ldrex/strex loops to simplify this, but it needs tidying up.
  if (TM->getOptLevel() != CodeGenOptLevel::None && EnableAtomicTidy)
    addPass(createCFGSimplificationPass(
        SimplifyCFGOptions().hoistCommonInsts(true).sinkCommonInsts(true),
        [this](const Function &F) {
          const auto &ST = this->TM->getSubtarget<ARMSubtarget>(F);
          return ST.hasAnyDataBarrier() && !ST.isThumb1Only();
        }));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 374-375
```cpp
  addPass(createMVEGatherScatterLoweringPass());
  addPass(createMVELaneInterleavingPass());
```
- EN: Declares `addPass`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `addPass`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 377-377
```cpp
  TargetPassConfig::addIRPasses();
```
- EN: Declares `TargetPassConfig::addIRPasses`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `TargetPassConfig::addIRPasses`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 379-381
```cpp
  // Run the parallel DSP pass.
  if (getOptLevel() == CodeGenOptLevel::Aggressive)
    addPass(createARMParallelDSPPass());
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 383-385
```cpp
  // Match complex arithmetic patterns
  if (TM->getOptLevel() >= CodeGenOptLevel::Default)
    addPass(createComplexDeinterleavingPass(TM));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 387-389
```cpp
  // Match interleaved memory accesses to ldN/stN intrinsics.
  if (TM->getOptLevel() != CodeGenOptLevel::None)
    addPass(createInterleavedAccessPass());
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 391-393
```cpp
  // Add Control Flow Guard checks.
  if (TM->getTargetTriple().isOSWindows())
    addPass(createCFGuardPass());
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 395-397
```cpp
  if (TM->Options.JMCInstrument)
    addPass(createJMCInstrumenterPass());
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 399-403
```cpp
void ARMPassConfig::addCodeGenPrepare() {
  if (getOptLevel() != CodeGenOptLevel::None)
    addPass(createTypePromotionLegacyPass());
  TargetPassConfig::addCodeGenPrepare();
}
```
- EN: Implements `ARMPassConfig::addCodeGenPrepare`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMPassConfig::addCodeGenPrepare`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 405-419
```cpp
bool ARMPassConfig::addPreISel() {
  if ((TM->getOptLevel() != CodeGenOptLevel::None &&
       EnableGlobalMerge == cl::BOU_UNSET) ||
      EnableGlobalMerge == cl::BOU_TRUE) {
    // FIXME: This is using the thumb1 only constant value for
    // maximal global offset for merging globals. We may want
    // to look into using the old value for non-thumb1 code of
    // 4095 based on the TargetMachine, but this starts to become
    // tricky when doing code gen per function.
    bool OnlyOptimizeForSize =
        (TM->getOptLevel() < CodeGenOptLevel::Aggressive) &&
        (EnableGlobalMerge == cl::BOU_UNSET);
    // Merging of extern globals is enabled by default on non-Mach-O as we
    // expect it to be generally either beneficial or harmless. On Mach-O it
    // is disabled as we emit the .subsections_via_symbols directive which
```
- EN: Implements `ARMPassConfig::addPreISel`, a target-specific routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMPassConfig::addPreISel`，它是一个围绕目标机器策略展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 420-424
```cpp
    // means that merging extern globals is not safe.
    bool MergeExternalByDefault = !TM->getTargetTriple().isOSBinFormatMachO();
    addPass(createGlobalMergePass(TM, 127, OnlyOptimizeForSize,
                                  MergeExternalByDefault));
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 426-437
```cpp
  if (TM->getOptLevel() != CodeGenOptLevel::None) {
    addPass(createHardwareLoopsLegacyPass());
    addPass(createMVETailPredicationPass());
    // FIXME: IR passes can delete address-taken basic blocks, deleting
    // corresponding blockaddresses. ARMConstantPoolConstant holds references to
    // address-taken basic blocks which can be invalidated if the function
    // containing the blockaddress has already been codegen'd and the basic
    // block is removed. Work around this by forcing all IR passes to run before
    // any ISel takes place. We should have a more principled way of handling
    // this. See D99707 for more details.
    addPass(createBarrierNoopPass());
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 439-440
```cpp
  return false;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 442-445
```cpp
bool ARMPassConfig::addInstSelector() {
  addPass(createARMISelDag(getARMTargetMachine(), getOptLevel()));
  return false;
}
```
- EN: Implements `ARMPassConfig::addInstSelector`, a target-specific routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMPassConfig::addInstSelector`，它是一个围绕目标机器策略展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 447-450
```cpp
bool ARMPassConfig::addIRTranslator() {
  addPass(new IRTranslator(getOptLevel()));
  return false;
}
```
- EN: Implements `ARMPassConfig::addIRTranslator`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMPassConfig::addIRTranslator`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 452-455
```cpp
bool ARMPassConfig::addLegalizeMachineIR() {
  addPass(new Legalizer());
  return false;
}
```
- EN: Implements `ARMPassConfig::addLegalizeMachineIR`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMPassConfig::addLegalizeMachineIR`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 457-460
```cpp
bool ARMPassConfig::addRegBankSelect() {
  addPass(new RegBankSelect());
  return false;
}
```
- EN: Implements `ARMPassConfig::addRegBankSelect`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMPassConfig::addRegBankSelect`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 462-465
```cpp
bool ARMPassConfig::addGlobalInstructionSelect() {
  addPass(new InstructionSelect(getOptLevel()));
  return false;
}
```
- EN: Implements `ARMPassConfig::addGlobalInstructionSelect`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMPassConfig::addGlobalInstructionSelect`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 467-470
```cpp
void ARMPassConfig::addPreRegAlloc() {
  if (getOptLevel() != CodeGenOptLevel::None) {
    if (getOptLevel() == CodeGenOptLevel::Aggressive)
      addPass(&MachinePipelinerID);
```
- EN: Implements `ARMPassConfig::addPreRegAlloc`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMPassConfig::addPreRegAlloc`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 472-472
```cpp
    addPass(createMVETPAndVPTOptimisationsPass());
```
- EN: Declares `addPass`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `addPass`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 474-474
```cpp
    addPass(createMLxExpansionPass());
```
- EN: Declares `addPass`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `addPass`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 476-477
```cpp
    if (EnableARMLoadStoreOpt)
      addPass(createARMLoadStoreOptLegacyPass(/* pre-register alloc */ true));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 479-482
```cpp
    if (!DisableA15SDOptimization)
      addPass(createA15SDOptimizerPass());
  }
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 484-487
```cpp
void ARMPassConfig::addPreSched2() {
  if (getOptLevel() != CodeGenOptLevel::None) {
    if (EnableARMLoadStoreOpt)
      addPass(createARMLoadStoreOptLegacyPass());
```
- EN: Implements `ARMPassConfig::addPreSched2`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMPassConfig::addPreSched2`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 489-491
```cpp
    addPass(new ARMExecutionDomainFix());
    addPass(createBreakFalseDeps());
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 493-495
```cpp
  // Expand some pseudo instructions into multiple instructions to allow
  // proper scheduling.
  addPass(createARMExpandPseudoPass());
```
- EN: Declares `addPass`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `addPass`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 497-498
```cpp
  // Emit KCFI checks for indirect calls.
  addPass(createKCFIPass());
```
- EN: Declares `addPass`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `addPass`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 500-507
```cpp
  if (getOptLevel() != CodeGenOptLevel::None) {
    // When optimising for size, always run the Thumb2SizeReduction pass before
    // IfConversion. Otherwise, check whether IT blocks are restricted
    // (e.g. in v8, IfConversion depends on Thumb instruction widths)
    addPass(createThumb2SizeReductionPass([this](const Function &F) {
      return this->TM->getSubtarget<ARMSubtarget>(F).hasMinSize() ||
             this->TM->getSubtarget<ARMSubtarget>(F).restrictIT();
    }));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 509-513
```cpp
    addPass(createIfConverter([](const MachineFunction &MF) {
      return !MF.getSubtarget<ARMSubtarget>().isThumb1Only();
    }));
  }
  addPass(createThumb2ITBlockPass());
```
- EN: Implements `addPass`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addPass`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 515-520
```cpp
  // Add both scheduling passes to give the subtarget an opportunity to pick
  // between them.
  if (getOptLevel() != CodeGenOptLevel::None) {
    addPass(&PostMachineSchedulerID);
    addPass(&PostRASchedulerID);
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 522-525
```cpp
  addPass(createMVEVPTBlockPass());
  addPass(createARMIndirectThunks());
  addPass(createARMSLSHardeningPass());
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 527-528
```cpp
void ARMPassConfig::addPreEmitPass() {
  addPass(createThumb2SizeReductionPass());
```
- EN: Implements `ARMPassConfig::addPreEmitPass`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMPassConfig::addPreEmitPass`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 530-536
```cpp
  // Unpack bundles for:
  // - Thumb2: Constant island pass requires unbundled instructions
  // - KCFI: KCFI_CHECK pseudo instructions need to be unbundled for AsmPrinter
  addPass(createUnpackMachineBundlesLegacy([](const MachineFunction &MF) {
    return MF.getSubtarget<ARMSubtarget>().isThumb2() ||
           MF.getFunction().getParent()->getModuleFlag("kcfi");
  }));
```
- EN: Implements `addPass`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `addPass`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 538-543
```cpp
  // Don't optimize barriers or block placement at -O0.
  if (getOptLevel() != CodeGenOptLevel::None) {
    addPass(createARMBlockPlacementPass());
    addPass(createARMOptimizeBarriersPass());
  }
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 545-545
```cpp
void ARMPassConfig::addPreEmitPass2() {
```
- EN: Implements `ARMPassConfig::addPreEmitPass2`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMPassConfig::addPreEmitPass2`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 547-561
```cpp
  // Inserts fixup instructions before unsafe AES operations. Instructions may
  // be inserted at the start of blocks and at within blocks so this pass has to
  // come before those below.
  addPass(createARMFixCortexA57AES1742098Pass());
  // Inserts BTIs at the start of functions and indirectly-called basic blocks,
  // so passes cannot add to the start of basic blocks once this has run.
  addPass(createARMBranchTargetsPass());
  // Inserts Constant Islands. Block sizes cannot be increased after this point,
  // as this may push the branch ranges and load offsets of accessing constant
  // pools out of range..
  addPass(createARMConstantIslandPass());
  // Finalises Low-Overhead Loops. This replaces pseudo instructions with real
  // instructions, but the pseudos all have conservative sizes so that block
  // sizes will only be decreased by this pass.
  addPass(createARMLowOverheadLoopsPass());
```
- EN: Declares `addPass`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `addPass`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 563-569
```cpp
  if (TM->getTargetTriple().isOSWindows()) {
    // Identify valid longjmp targets for Windows Control Flow Guard.
    addPass(createCFGuardLongjmpPass());
    // Identify valid eh continuation targets for Windows EHCont Guard.
    addPass(createEHContGuardTargetsPass());
  }
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 571-574
```cpp
yaml::MachineFunctionInfo *
ARMBaseTargetMachine::createDefaultFuncInfoYAML() const {
  return new yaml::ARMFunctionInfo();
}
```
- EN: Implements `ARMBaseTargetMachine::createDefaultFuncInfoYAML`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMBaseTargetMachine::createDefaultFuncInfoYAML`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 576-580
```cpp
yaml::MachineFunctionInfo *
ARMBaseTargetMachine::convertFuncInfoToYAML(const MachineFunction &MF) const {
  const auto *MFI = MF.getInfo<ARMFunctionInfo>();
  return new yaml::ARMFunctionInfo(*MFI);
}
```
- EN: Implements `ARMBaseTargetMachine::convertFuncInfoToYAML`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMBaseTargetMachine::convertFuncInfoToYAML`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 582-589
```cpp
bool ARMBaseTargetMachine::parseMachineFunctionInfo(
    const yaml::MachineFunctionInfo &MFI, PerFunctionMIParsingState &PFS,
    SMDiagnostic &Error, SMRange &SourceRange) const {
  const auto &YamlMFI = static_cast<const yaml::ARMFunctionInfo &>(MFI);
  MachineFunction &MF = PFS.MF;
  MF.getInfo<ARMFunctionInfo>()->initializeBaseYamlFields(YamlMFI);
  return false;
}
```
- EN: Implements `ARMBaseTargetMachine::parseMachineFunctionInfo`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMBaseTargetMachine::parseMachineFunctionInfo`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 591-591
```cpp
void ARMBaseTargetMachine::reset() { SubtargetMap.clear(); }
```
- EN: Implements `ARMBaseTargetMachine::reset`, a target-specific routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMBaseTargetMachine::reset`，它是一个围绕目标机器策略展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

## Key Concepts / 关键概念

- EN: Primary role: target-machine configuration and pass-pipeline integration.
  - CN: 核心职责：目标机器配置与优化流水线集成。
- EN: The MC layer is used to print, encode, or stream target instructions and symbols.
  - CN: MC 层被用于打印、编码或流式输出目标指令与符号。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。

## Dependencies / 依赖关系

- EN: Backend-local headers: `ARMTargetMachine.h`, `ARM.h`, `ARMLatencyMutations.h`, `ARMMachineFunctionInfo.h`, `ARMMacroFusion.h`, `ARMSubtarget.h`, `ARMTargetObjectFile.h`, `ARMTargetTransformInfo.h` ... (+2 more).
  - CN: 后端本地头文件：`ARMTargetMachine.h`, `ARM.h`, `ARMLatencyMutations.h`, `ARMMachineFunctionInfo.h`, `ARMMacroFusion.h`, `ARMSubtarget.h`, `ARMTargetObjectFile.h`, `ARMTargetTransformInfo.h` ... (+2 more)。
- EN: LLVM infrastructure headers: `llvm/ADT/StringRef.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/CodeGen/ExecutionDomainFix.h`, `llvm/CodeGen/GlobalISel/CSEInfo.h`, `llvm/CodeGen/GlobalISel/CallLowering.h`, `llvm/CodeGen/GlobalISel/IRTranslator.h`, `llvm/CodeGen/GlobalISel/InstructionSelect.h`, `llvm/CodeGen/GlobalISel/Legalizer.h` ... (+26 more).
  - CN: LLVM 基础设施头文件：`llvm/ADT/StringRef.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/CodeGen/ExecutionDomainFix.h`, `llvm/CodeGen/GlobalISel/CSEInfo.h`, `llvm/CodeGen/GlobalISel/CallLowering.h`, `llvm/CodeGen/GlobalISel/IRTranslator.h`, `llvm/CodeGen/GlobalISel/InstructionSelect.h`, `llvm/CodeGen/GlobalISel/Legalizer.h` ... (+26 more)。
- EN: Standard/system headers: `cassert`, `memory`, `optional`, `string`.
  - CN: 标准库/系统头文件：`cassert`, `memory`, `optional`, `string`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
- EN: The implementation also depends on LLVM MC layer classes for final assembly/object emission.
  - CN: 实现还依赖 LLVM 的 MC 层类来完成最终汇编/目标文件输出。
