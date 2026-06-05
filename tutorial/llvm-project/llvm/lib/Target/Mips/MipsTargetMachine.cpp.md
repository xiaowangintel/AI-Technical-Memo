# MipsTargetMachine.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MipsTargetMachine.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `MipsTargetMachine` for the Mips backend, focusing on target-machine configuration and pass-pipeline integration.
- 用途 (CN): 实现 Mips 后端中的 `MipsTargetMachine`，重点处理目标机器配置与优化流水线集成。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- MipsTargetMachine.cpp - Define TargetMachine for Mips -------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Implements the info about Mips target spec.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 13-27
```cpp
#include "MipsTargetMachine.h"
#include "MCTargetDesc/MipsABIInfo.h"
#include "MCTargetDesc/MipsMCTargetDesc.h"
#include "Mips.h"
#include "Mips16ISelDAGToDAG.h"
#include "MipsMachineFunction.h"
#include "MipsSEISelDAGToDAG.h"
#include "MipsSubtarget.h"
#include "MipsTargetObjectFile.h"
#include "MipsTargetTransformInfo.h"
#include "TargetInfo/MipsTargetInfo.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/Analysis/TargetTransformInfo.h"
#include "llvm/CodeGen/BasicTTIImpl.h"
#include "llvm/CodeGen/GlobalISel/CSEInfo.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 28-42
```cpp
#include "llvm/CodeGen/GlobalISel/IRTranslator.h"
#include "llvm/CodeGen/GlobalISel/InstructionSelect.h"
#include "llvm/CodeGen/GlobalISel/Legalizer.h"
#include "llvm/CodeGen/GlobalISel/RegBankSelect.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/Passes.h"
#include "llvm/CodeGen/TargetPassConfig.h"
#include "llvm/IR/Attributes.h"
#include "llvm/IR/Function.h"
#include "llvm/InitializePasses.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/CodeGen.h"
#include "llvm/Support/Compiler.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/raw_ostream.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 43-45
```cpp
#include "llvm/Target/TargetOptions.h"
#include <optional>
#include <string>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 47-47
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 49-49
```cpp
#define DEBUG_TYPE "mips"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 51-53
```cpp
static cl::opt<bool>
    EnableMulMulFix("mfix4300", cl::init(false),
                    cl::desc("Enable the VR4300 mulmul bug fix."), cl::Hidden);
```
- EN: Declares `cl::init`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `cl::init`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 55-60
```cpp
extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void LLVMInitializeMipsTarget() {
  // Register the target.
  RegisterTargetMachine<MipsebTargetMachine> X(getTheMipsTarget());
  RegisterTargetMachine<MipselTargetMachine> Y(getTheMipselTarget());
  RegisterTargetMachine<MipsebTargetMachine> A(getTheMips64Target());
  RegisterTargetMachine<MipselTargetMachine> B(getTheMips64elTarget());
```
- EN: Implements `LLVMInitializeMipsTarget`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `LLVMInitializeMipsTarget`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 62-73
```cpp
  PassRegistry *PR = PassRegistry::getPassRegistry();
  initializeGlobalISel(*PR);
  initializeMipsAsmPrinterPass(*PR);
  initializeMipsDelaySlotFillerPass(*PR);
  initializeMipsBranchExpansionPass(*PR);
  initializeMicroMipsSizeReducePass(*PR);
  initializeMipsPreLegalizerCombinerPass(*PR);
  initializeMipsPostLegalizerCombinerPass(*PR);
  initializeMipsMulMulBugFixPass(*PR);
  initializeMipsDAGToDAGISelLegacyPass(*PR);
  initializeMipsSetMachineRegisterFlagsPass(*PR);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 75-79
```cpp
static std::unique_ptr<TargetLoweringObjectFile> createTLOF(const Triple &TT) {
  if (TT.isOSBinFormatCOFF())
    return std::make_unique<TargetLoweringObjectFileCOFF>();
  return std::make_unique<MipsTargetObjectFile>();
}
```
- EN: Implements `createTLOF`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `createTLOF`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 81-86
```cpp
static Reloc::Model getEffectiveRelocModel(bool JIT,
                                           std::optional<Reloc::Model> RM) {
  if (!RM || JIT)
    return Reloc::Static;
  return *RM;
}
```
- EN: Implements `getEffectiveRelocModel`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getEffectiveRelocModel`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 88-102
```cpp
// On function prologue, the stack is created by decrementing
// its pointer. Once decremented, all references are done with positive
// offset from the stack/frame pointer, using StackGrowsUp enables
// an easier handling.
// Using CodeModel::Large enables different CALL behavior.
MipsTargetMachine::MipsTargetMachine(const Target &T, const Triple &TT,
                                     StringRef CPU, StringRef FS,
                                     const TargetOptions &Options,
                                     std::optional<Reloc::Model> RM,
                                     std::optional<CodeModel::Model> CM,
                                     CodeGenOptLevel OL, bool JIT,
                                     bool isLittle)
    : CodeGenTargetMachineImpl(
          T, TT.computeDataLayout(Options.MCOptions.getABIName()), TT, CPU, FS,
          Options, getEffectiveRelocModel(JIT, RM),
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 103-113
```cpp
          getEffectiveCodeModel(CM, CodeModel::Small), OL),
      isLittle(isLittle), TLOF(createTLOF(getTargetTriple())),
      ABI(MipsABIInfo::computeTargetABI(TT, Options.MCOptions.getABIName())),
      Subtarget(nullptr),
      DefaultSubtarget(TT, CPU, FS, isLittle, *this, std::nullopt),
      NoMips16Subtarget(TT, CPU, FS.empty() ? "-mips16" : FS.str() + ",-mips16",
                        isLittle, *this, std::nullopt),
      Mips16Subtarget(TT, CPU, FS.empty() ? "+mips16" : FS.str() + ",+mips16",
                      isLittle, *this, std::nullopt) {
  Subtarget = &DefaultSubtarget;
  initAsmInfo();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 115-117
```cpp
  // Mips supports the debug entry values.
  setSupportsDebugEntryValues(true);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 119-119
```cpp
MipsTargetMachine::~MipsTargetMachine() = default;
```
- EN: Declares `MipsTargetMachine::~MipsTargetMachine`, a target-specific routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `MipsTargetMachine::~MipsTargetMachine`，它是一个围绕目标机器策略展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 121-121
```cpp
void MipsebTargetMachine::anchor() {}
```
- EN: Implements `MipsebTargetMachine::anchor`, a target-specific routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsebTargetMachine::anchor`，它是一个围绕目标机器策略展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 123-129
```cpp
MipsebTargetMachine::MipsebTargetMachine(const Target &T, const Triple &TT,
                                         StringRef CPU, StringRef FS,
                                         const TargetOptions &Options,
                                         std::optional<Reloc::Model> RM,
                                         std::optional<CodeModel::Model> CM,
                                         CodeGenOptLevel OL, bool JIT)
    : MipsTargetMachine(T, TT, CPU, FS, Options, RM, CM, OL, JIT, false) {}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 131-131
```cpp
void MipselTargetMachine::anchor() {}
```
- EN: Implements `MipselTargetMachine::anchor`, a target-specific routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipselTargetMachine::anchor`，它是一个围绕目标机器策略展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 133-139
```cpp
MipselTargetMachine::MipselTargetMachine(const Target &T, const Triple &TT,
                                         StringRef CPU, StringRef FS,
                                         const TargetOptions &Options,
                                         std::optional<Reloc::Model> RM,
                                         std::optional<CodeModel::Model> CM,
                                         CodeGenOptLevel OL, bool JIT)
    : MipsTargetMachine(T, TT, CPU, FS, Options, RM, CM, OL, JIT, true) {}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 141-144
```cpp
const MipsSubtarget *
MipsTargetMachine::getSubtargetImpl(const Function &F) const {
  Attribute CPUAttr = F.getFnAttribute("target-cpu");
  Attribute FSAttr = F.getFnAttribute("target-features");
```
- EN: Implements `MipsTargetMachine::getSubtargetImpl`, a query/helper routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetMachine::getSubtargetImpl`，它是一个围绕目标机器策略展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 146-151
```cpp
  std::string CPU =
      CPUAttr.isValid() ? CPUAttr.getValueAsString().str() : TargetCPU;
  std::string FS =
      FSAttr.isValid() ? FSAttr.getValueAsString().str() : TargetFS;
  bool hasMips16Attr = F.getFnAttribute("mips16").isValid();
  bool hasNoMips16Attr = F.getFnAttribute("nomips16").isValid();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 153-154
```cpp
  bool HasMicroMipsAttr = F.getFnAttribute("micromips").isValid();
  bool HasNoMicroMipsAttr = F.getFnAttribute("nomicromips").isValid();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 156-159
```cpp
  // FIXME: This is related to the code below to reset the target options,
  // we need to know whether or not the soft float flag is set on the
  // function, so we can enable it as a subtarget feature.
  bool softFloat = F.getFnAttribute("use-soft-float").getValueAsBool();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 161-170
```cpp
  if (hasMips16Attr)
    FS += FS.empty() ? "+mips16" : ",+mips16";
  else if (hasNoMips16Attr)
    FS += FS.empty() ? "-mips16" : ",-mips16";
  if (HasMicroMipsAttr)
    FS += FS.empty() ? "+micromips" : ",+micromips";
  else if (HasNoMicroMipsAttr)
    FS += FS.empty() ? "-micromips" : ",-micromips";
  if (softFloat)
    FS += FS.empty() ? "+soft-float" : ",+soft-float";
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 172-183
```cpp
  auto &I = SubtargetMap[CPU + FS];
  if (!I) {
    // This needs to be done before we create a new subtarget since any
    // creation will depend on the TM and the code generation flags on the
    // function that reside in TargetOptions.
    resetTargetOptions(F);
    I = std::make_unique<MipsSubtarget>(
        TargetTriple, CPU, FS, isLittle, *this,
        MaybeAlign(F.getParent()->getOverrideStackAlignment()));
  }
  return I.get();
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 185-186
```cpp
void MipsTargetMachine::resetSubtarget(MachineFunction *MF) {
  LLVM_DEBUG(dbgs() << "resetSubtarget\n");
```
- EN: Implements `MipsTargetMachine::resetSubtarget`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetMachine::resetSubtarget`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 188-189
```cpp
  Subtarget = &MF->getSubtarget<MipsSubtarget>();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 191-191
```cpp
namespace {
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 193-204
```cpp
/// Mips Code Generator Pass Configuration Options.
class MipsPassConfig : public TargetPassConfig {
public:
  MipsPassConfig(MipsTargetMachine &TM, PassManagerBase &PM)
      : TargetPassConfig(TM, PM) {
    // The current implementation of long branch pass requires a scratch
    // register ($at) to be available before branch instructions. Tail merging
    // can break this requirement, so disable it when long branch pass is
    // enabled.
    EnableTailMerge = !getMipsSubtarget().enableLongBranchPass();
    EnableLoopTermFold = true;
  }
```
- EN: Declares `MipsPassConfig`, packaging target-specific state and APIs around `MipsTargetMachine`.
- CN: 这里声明 `MipsPassConfig`，把与 `MipsTargetMachine` 相关的目标特定状态和 API 组织在一起。

### Lines 206-208
```cpp
  MipsTargetMachine &getMipsTargetMachine() const {
    return getTM<MipsTargetMachine>();
  }
```
- EN: Implements `getMipsTargetMachine`, a query/helper routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getMipsTargetMachine`，它是一个围绕目标机器策略展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 210-212
```cpp
  const MipsSubtarget &getMipsSubtarget() const {
    return *getMipsTargetMachine().getSubtargetImpl();
  }
```
- EN: Implements `getMipsSubtarget`, a query/helper routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getMipsSubtarget`，它是一个围绕目标机器策略展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 214-223
```cpp
  void addIRPasses() override;
  bool addInstSelector() override;
  void addPreEmitPass() override;
  void addPreRegAlloc() override;
  bool addIRTranslator() override;
  void addPreLegalizeMachineIR() override;
  bool addLegalizeMachineIR() override;
  void addPreRegBankSelect() override;
  bool addRegBankSelect() override;
  bool addGlobalInstructionSelect() override;
```
- EN: Declares `addIRPasses`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `addIRPasses`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 225-226
```cpp
  std::unique_ptr<CSEConfigBase> getCSEConfig() const override;
};
```
- EN: Declares `getCSEConfig`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getCSEConfig`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 228-228
```cpp
} // end anonymous namespace
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 230-232
```cpp
TargetPassConfig *MipsTargetMachine::createPassConfig(PassManagerBase &PM) {
  return new MipsPassConfig(*this, PM);
}
```
- EN: Implements `MipsTargetMachine::createPassConfig`, a query/helper routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetMachine::createPassConfig`，它是一个围绕目标机器策略展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 234-236
```cpp
std::unique_ptr<CSEConfigBase> MipsPassConfig::getCSEConfig() const {
  return getStandardCSEConfigForOpt(TM->getOptLevel());
}
```
- EN: Implements `MipsPassConfig::getCSEConfig`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsPassConfig::getCSEConfig`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 238-252
```cpp
void MipsPassConfig::addIRPasses() {
  TargetPassConfig::addIRPasses();
  addPass(createAtomicExpandLegacyPass());
  if (getMipsSubtarget().os16())
    addPass(createMipsOs16Pass());
  if (getMipsSubtarget().inMips16HardFloat())
    addPass(createMips16HardFloatPass());
}
// Install an instruction selector pass using
// the ISelDag to gen Mips code.
bool MipsPassConfig::addInstSelector() {
  addPass(createMipsModuleISelDagPass());
  addPass(createMips16ISelDag(getMipsTargetMachine(), getOptLevel()));
  addPass(createMipsSEISelDag(getMipsTargetMachine(), getOptLevel()));
  return false;
```
- EN: Implements `MipsPassConfig::addIRPasses`, a target-specific routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsPassConfig::addIRPasses`，它是一个围绕目标机器策略展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 253-253
```cpp
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 255-258
```cpp
void MipsPassConfig::addPreRegAlloc() {
  addPass(createMipsOptimizePICCallPass());
  addPass(createMipsSetMachineRegisterFlagsPass());
}
```
- EN: Implements `MipsPassConfig::addPreRegAlloc`, a target-specific routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsPassConfig::addPreRegAlloc`，它是一个围绕寄存器管理展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 260-266
```cpp
TargetTransformInfo
MipsTargetMachine::getTargetTransformInfo(const Function &F) const {
  if (Subtarget->allowMixed16_32()) {
    LLVM_DEBUG(errs() << "No Target Transform Info Pass Added\n");
    // FIXME: This is no longer necessary as the TTI returned is per-function.
    return TargetTransformInfo(F.getDataLayout());
  }
```
- EN: Implements `MipsTargetMachine::getTargetTransformInfo`, a query/helper routine centered on target-machine policy. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetMachine::getTargetTransformInfo`，它是一个围绕目标机器策略展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 268-270
```cpp
  LLVM_DEBUG(errs() << "Target Transform Info Pass Added\n");
  return TargetTransformInfo(std::make_unique<MipsTTIImpl>(this, F));
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 272-276
```cpp
MachineFunctionInfo *MipsTargetMachine::createMachineFunctionInfo(
    BumpPtrAllocator &Allocator, const Function &F,
    const TargetSubtargetInfo *STI) const {
  return MipsFunctionInfo::create<MipsFunctionInfo>(Allocator, F, STI);
}
```
- EN: Implements `MipsTargetMachine::createMachineFunctionInfo`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsTargetMachine::createMachineFunctionInfo`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 278-282
```cpp
// Implemented by targets that want to run passes immediately before
// machine code is emitted.
void MipsPassConfig::addPreEmitPass() {
  // Expand pseudo instructions that are sensitive to register allocation.
  addPass(createMipsExpandPseudoPass());
```
- EN: Implements `MipsPassConfig::addPreEmitPass`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsPassConfig::addPreEmitPass`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 284-286
```cpp
  // The microMIPS size reduction pass performs instruction reselection for
  // instructions which can be remapped to a 16 bit instruction.
  addPass(createMicroMipsSizeReducePass());
```
- EN: Declares `addPass`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `addPass`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 288-291
```cpp
  // This pass inserts a nop instruction between two back-to-back multiplication
  // instructions when the "mfix4300" flag is passed.
  if (EnableMulMulFix)
    addPass(createMipsMulMulBugPass());
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 293-295
```cpp
  // The delay slot filler pass can potientially create forbidden slot hazards
  // for MIPSR6 and therefore it should go before MipsBranchExpansion pass.
  addPass(createMipsDelaySlotFillerPass());
```
- EN: Declares `addPass`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `addPass`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 297-305
```cpp
  // This pass expands branches and takes care about the forbidden slot hazards.
  // Expanding branches may potentially create forbidden slot hazards for
  // MIPSR6, and fixing such hazard may potentially break a branch by extending
  // its offset out of range. That's why this pass combine these two tasks, and
  // runs them alternately until one of them finishes without any changes. Only
  // then we can be sure that all branches are expanded properly and no hazards
  // exists.
  // Any new pass should go before this pass.
  addPass(createMipsBranchExpansion());
```
- EN: Declares `addPass`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `addPass`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 307-308
```cpp
  addPass(createMipsConstantIslandPass());
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 310-313
```cpp
bool MipsPassConfig::addIRTranslator() {
  addPass(new IRTranslator(getOptLevel()));
  return false;
}
```
- EN: Implements `MipsPassConfig::addIRTranslator`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsPassConfig::addIRTranslator`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 315-317
```cpp
void MipsPassConfig::addPreLegalizeMachineIR() {
  addPass(createMipsPreLegalizeCombiner());
}
```
- EN: Implements `MipsPassConfig::addPreLegalizeMachineIR`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsPassConfig::addPreLegalizeMachineIR`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 319-322
```cpp
bool MipsPassConfig::addLegalizeMachineIR() {
  addPass(new Legalizer());
  return false;
}
```
- EN: Implements `MipsPassConfig::addLegalizeMachineIR`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsPassConfig::addLegalizeMachineIR`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 324-327
```cpp
void MipsPassConfig::addPreRegBankSelect() {
  bool IsOptNone = getOptLevel() == CodeGenOptLevel::None;
  addPass(createMipsPostLegalizeCombiner(IsOptNone));
}
```
- EN: Implements `MipsPassConfig::addPreRegBankSelect`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsPassConfig::addPreRegBankSelect`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 329-332
```cpp
bool MipsPassConfig::addRegBankSelect() {
  addPass(new RegBankSelect());
  return false;
}
```
- EN: Implements `MipsPassConfig::addRegBankSelect`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsPassConfig::addRegBankSelect`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 334-337
```cpp
bool MipsPassConfig::addGlobalInstructionSelect() {
  addPass(new InstructionSelect(getOptLevel()));
  return false;
}
```
- EN: Implements `MipsPassConfig::addGlobalInstructionSelect`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MipsPassConfig::addGlobalInstructionSelect`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

## Key Concepts / 关键概念

- EN: Primary role: target-machine configuration and pass-pipeline integration.
  - CN: 核心职责：目标机器配置与优化流水线集成。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。
- EN: Stack-frame and spill/reload concerns are reflected in the API or implementation.
  - CN: API 或实现中体现了栈帧以及溢出/重载相关问题。

## Dependencies / 依赖关系

- EN: Backend-local headers: `MipsTargetMachine.h`, `MCTargetDesc/MipsABIInfo.h`, `MCTargetDesc/MipsMCTargetDesc.h`, `Mips.h`, `Mips16ISelDAGToDAG.h`, `MipsMachineFunction.h`, `MipsSEISelDAGToDAG.h`, `MipsSubtarget.h` ... (+3 more).
  - CN: 后端本地头文件：`MipsTargetMachine.h`, `MCTargetDesc/MipsABIInfo.h`, `MCTargetDesc/MipsMCTargetDesc.h`, `Mips.h`, `Mips16ISelDAGToDAG.h`, `MipsMachineFunction.h`, `MipsSEISelDAGToDAG.h`, `MipsSubtarget.h` ... (+3 more)。
- EN: LLVM infrastructure headers: `llvm/ADT/StringRef.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/CodeGen/BasicTTIImpl.h`, `llvm/CodeGen/GlobalISel/CSEInfo.h`, `llvm/CodeGen/GlobalISel/IRTranslator.h`, `llvm/CodeGen/GlobalISel/InstructionSelect.h`, `llvm/CodeGen/GlobalISel/Legalizer.h`, `llvm/CodeGen/GlobalISel/RegBankSelect.h` ... (+12 more).
  - CN: LLVM 基础设施头文件：`llvm/ADT/StringRef.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/CodeGen/BasicTTIImpl.h`, `llvm/CodeGen/GlobalISel/CSEInfo.h`, `llvm/CodeGen/GlobalISel/IRTranslator.h`, `llvm/CodeGen/GlobalISel/InstructionSelect.h`, `llvm/CodeGen/GlobalISel/Legalizer.h`, `llvm/CodeGen/GlobalISel/RegBankSelect.h` ... (+12 more)。
- EN: Standard/system headers: `optional`, `string`.
  - CN: 标准库/系统头文件：`optional`, `string`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
