# X86TargetMachine.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/X86TargetMachine.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements target machine setup for the core X86 backend. / 实现X86 后端核心中的目标机器初始化。

## Line-by-Line Analysis / 逐行分析
### Lines 1-50: File header, licensing, and opening context / 文件头、许可证与开场上下文
```cpp
//===-- X86TargetMachine.cpp - Define TargetMachine for the X86 -----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the X86 specific subclass of TargetMachine.
//
//===----------------------------------------------------------------------===//

#include "X86TargetMachine.h"
#include "MCTargetDesc/X86MCTargetDesc.h"
#include "TargetInfo/X86TargetInfo.h"
#include "X86.h"
#include "X86MachineFunctionInfo.h"
#include "X86MacroFusion.h"
#include "X86Subtarget.h"
#include "X86TargetObjectFile.h"
#include "X86TargetTransformInfo.h"
#include "llvm-c/Visibility.h"
#include "llvm/ADT/SmallString.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/Analysis/TargetTransformInfo.h"
#include "llvm/CodeGen/ExecutionDomainFix.h"
#include "llvm/CodeGen/GlobalISel/CSEInfo.h"
#include "llvm/CodeGen/GlobalISel/CallLowering.h"
#include "llvm/CodeGen/GlobalISel/IRTranslator.h"
#include "llvm/CodeGen/GlobalISel/InstructionSelect.h"
#include "llvm/CodeGen/GlobalISel/Legalizer.h"
#include "llvm/CodeGen/GlobalISel/RegBankSelect.h"
#include "llvm/CodeGen/MIRParser/MIParser.h"
#include "llvm/CodeGen/MIRYamlMapping.h"
#include "llvm/CodeGen/MachineScheduler.h"
#include "llvm/CodeGen/Passes.h"
#include "llvm/CodeGen/TargetPassConfig.h"
#include "llvm/IR/Attributes.h"
#include "llvm/IR/DataLayout.h"
#include "llvm/IR/Function.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Pass.h"
#include "llvm/Support/CodeGen.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Target/TargetLoweringObjectFile.h"
#include "llvm/Target/TargetOptions.h"
#include "llvm/TargetParser/Triple.h"
#include "llvm/Transforms/CFGuard.h"
```
**EN:** This section implements file header, licensing, and opening context for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的文件头、许可证与开场上下文。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 51-100: Preprocessor directives and structural setup / 预处理指令与结构初始化
```cpp
#include <memory>
#include <optional>

using namespace llvm;

cl::opt<bool>
    X86EnableMachineCombinerPass("x86-machine-combiner",
                                 cl::desc("Enable the machine combiner pass"),
                                 cl::init(true), cl::Hidden);

static cl::opt<bool>
    EnableTileRAPass("x86-tile-ra",
                     cl::desc("Enable the tile register allocation pass"),
                     cl::init(true), cl::Hidden);

extern "C" LLVM_C_ABI void LLVMInitializeX86Target() {
  // Register the target.
  RegisterTargetMachine<X86TargetMachine> X(getTheX86_32Target());
  RegisterTargetMachine<X86TargetMachine> Y(getTheX86_64Target());

  PassRegistry &PR = *PassRegistry::getPassRegistry();
  initializeX86LowerAMXIntrinsicsLegacyPassPass(PR);
  initializeX86LowerAMXTypeLegacyPassPass(PR);
  initializeX86PreTileConfigLegacyPass(PR);
  initializeGlobalISel(PR);
  initializeWinEHStateLegacyPass(PR);
  initializeX86FixupBWInstLegacyPass(PR);
  initializeCompressEVEXLegacyPass(PR);
  initializeFixupLEAsLegacyPass(PR);
  initializeX86FPStackifierLegacyPass(PR);
  initializeX86FixupSetCCLegacyPass(PR);
  initializeX86CallFrameOptimizationLegacyPass(PR);
  initializeX86CmovConversionLegacyPass(PR);
  initializeX86TileConfigLegacyPass(PR);
  initializeX86FastPreTileConfigLegacyPass(PR);
  initializeX86FastTileConfigLegacyPass(PR);
  initializeMachineKCFILegacyPass(PR);
  initializeX86LowerTileCopyLegacyPass(PR);
  initializeX86ExpandPseudoLegacyPass(PR);
  initializeX86ExecutionDomainFixPass(PR);
  initializeX86DomainReassignmentLegacyPass(PR);
  initializeX86AvoidSFBLegacyPass(PR);
  initializeX86AvoidTrailingCallLegacyPassPass(PR);
  initializeX86SpeculativeLoadHardeningLegacyPass(PR);
  initializeX86SpeculativeExecutionSideEffectSuppressionLegacyPass(PR);
  initializeX86FlagsCopyLoweringLegacyPass(PR);
  initializeX86LoadValueInjectionLoadHardeningLegacyPass(PR);
  initializeX86LoadValueInjectionRetHardeningLegacyPass(PR);
  initializeX86OptimizeLEAsLegacyPass(PR);
  initializeX86PartialReductionLegacyPass(PR);
```
**EN:** This section implements preprocessor directives and structural setup for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的预处理指令与结构初始化。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 101-150: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
  initializeX86ReturnThunksLegacyPass(PR);
  initializeX86DAGToDAGISelLegacyPass(PR);
  initializeX86ArgumentStackSlotLegacyPass(PR);
  initializeX86AsmPrinterPass(PR);
  initializeX86FixupInstTuningLegacyPass(PR);
  initializeX86FixupVectorConstantsLegacyPass(PR);
  initializeX86DynAllocaExpanderLegacyPass(PR);
  initializeX86SuppressAPXForRelocationLegacyPass(PR);
  initializeX86WinEHUnwindV2LegacyPass(PR);
  initializeX86PreLegalizerCombinerLegacyPass(PR);
  initializeX86PostLegalizerCombinerLegacyPass(PR);
}

static std::unique_ptr<TargetLoweringObjectFile> createTLOF(const Triple &TT) {
  if (TT.isOSBinFormatMachO()) {
    if (TT.isX86_64())
      return std::make_unique<X86_64MachoTargetObjectFile>();
    return std::make_unique<TargetLoweringObjectFileMachO>();
  }

  if (TT.isOSBinFormatCOFF())
    return std::make_unique<TargetLoweringObjectFileCOFF>();

  if (TT.isX86_64())
    return std::make_unique<X86_64ELFTargetObjectFile>();
  return std::make_unique<X86ELFTargetObjectFile>();
}

static Reloc::Model getEffectiveRelocModel(const Triple &TT, bool JIT,
                                           std::optional<Reloc::Model> RM) {
  bool is64Bit = TT.isX86_64();
  if (!RM) {
    // JIT codegen should use static relocations by default, since it's
    // typically executed in process and not relocatable.
    if (JIT)
      return Reloc::Static;

    // Darwin defaults to PIC in 64 bit mode and dynamic-no-pic in 32 bit mode.
    // Win64 requires rip-rel addressing, thus we force it to PIC. Otherwise we
    // use static relocation model by default.
    if (TT.isOSDarwin()) {
      if (is64Bit)
        return Reloc::PIC_;
      return Reloc::DynamicNoPIC;
    }
    if (TT.isOSWindows() && is64Bit)
      return Reloc::PIC_;
    return Reloc::Static;
  }

```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 151-200: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
  // ELF and X86-64 don't have a distinct DynamicNoPIC model.  DynamicNoPIC
  // is defined as a model for code which may be used in static or dynamic
  // executables but not necessarily a shared library. On X86-32 we just
  // compile in -static mode, in x86-64 we use PIC.
  if (*RM == Reloc::DynamicNoPIC) {
    if (is64Bit)
      return Reloc::PIC_;
    if (!TT.isOSDarwin())
      return Reloc::Static;
  }

  // If we are on Darwin, disallow static relocation model in X86-64 mode, since
  // the Mach-O file format doesn't support it.
  if (*RM == Reloc::Static && TT.isOSDarwin() && is64Bit)
    return Reloc::PIC_;

  return *RM;
}

static CodeModel::Model
getEffectiveX86CodeModel(const Triple &TT, std::optional<CodeModel::Model> CM,
                         bool JIT) {
  bool Is64Bit = TT.isX86_64();
  if (CM) {
    if (*CM == CodeModel::Tiny)
      reportFatalUsageError("target does not support the tiny CodeModel");
    return *CM;
  }
  if (JIT)
    return Is64Bit ? CodeModel::Large : CodeModel::Small;
  return CodeModel::Small;
}

/// Create an X86 target.
///
X86TargetMachine::X86TargetMachine(const Target &T, const Triple &TT,
                                   StringRef CPU, StringRef FS,
                                   const TargetOptions &Options,
                                   std::optional<Reloc::Model> RM,
                                   std::optional<CodeModel::Model> CM,
                                   CodeGenOptLevel OL, bool JIT)
    : CodeGenTargetMachineImpl(T, TT.computeDataLayout(), TT, CPU, FS, Options,
                               getEffectiveRelocModel(TT, JIT, RM),
                               getEffectiveX86CodeModel(TT, CM, JIT), OL),
      TLOF(createTLOF(getTargetTriple())), IsJIT(JIT) {
  // On PS4/PS5, the "return address" of a 'noreturn' call must still be within
  // the calling function. Note that this also includes __stack_chk_fail,
  // so there was some target-specific logic in the instruction selectors
  // to handle that. That code has since been generalized, so the only thing
  // needed is to set TrapUnreachable here.
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 201-250: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
  if (TT.isPS() || TT.isOSBinFormatMachO()) {
    this->Options.TrapUnreachable = true;
    this->Options.NoTrapAfterNoreturn = TT.isOSBinFormatMachO();
  }

  setMachineOutliner(true);

  // x86 supports the debug entry values.
  setSupportsDebugEntryValues(true);

  initAsmInfo();
}

X86TargetMachine::~X86TargetMachine() = default;

const X86Subtarget *
X86TargetMachine::getSubtargetImpl(const Function &F) const {
  Attribute CPUAttr = F.getFnAttribute("target-cpu");
  Attribute TuneAttr = F.getFnAttribute("tune-cpu");
  Attribute FSAttr = F.getFnAttribute("target-features");

  StringRef CPU =
      CPUAttr.isValid() ? CPUAttr.getValueAsString() : (StringRef)TargetCPU;
  // "x86-64" is a default target setting for many front ends. In these cases,
  // they actually request for "generic" tuning unless the "tune-cpu" was
  // specified.
  StringRef TuneCPU = TuneAttr.isValid() ? TuneAttr.getValueAsString()
                      : CPU == "x86-64"  ? "generic"
                                         : (StringRef)CPU;
  StringRef FS =
      FSAttr.isValid() ? FSAttr.getValueAsString() : (StringRef)TargetFS;

  SmallString<512> Key;
  // The additions here are ordered so that the definitely short strings are
  // added first so we won't exceed the small size. We append the
  // much longer FS string at the end so that we only heap allocate at most
  // one time.

  // Extract prefer-vector-width attribute.
  unsigned PreferVectorWidthOverride = 0;
  Attribute PreferVecWidthAttr = F.getFnAttribute("prefer-vector-width");
  if (PreferVecWidthAttr.isValid()) {
    StringRef Val = PreferVecWidthAttr.getValueAsString();
    unsigned Width;
    if (!Val.getAsInteger(0, Width)) {
      Key += 'p';
      Key += Val;
      PreferVectorWidthOverride = Width;
    }
  }
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 251-300: Control flow, matching, and data updates / 控制流、匹配与数据更新
```cpp

  // Extract min-legal-vector-width attribute.
  unsigned RequiredVectorWidth = UINT32_MAX;
  Attribute MinLegalVecWidthAttr = F.getFnAttribute("min-legal-vector-width");
  if (MinLegalVecWidthAttr.isValid()) {
    StringRef Val = MinLegalVecWidthAttr.getValueAsString();
    unsigned Width;
    if (!Val.getAsInteger(0, Width)) {
      Key += 'm';
      Key += Val;
      RequiredVectorWidth = Width;
    }
  }

  // Add CPU to the Key.
  Key += CPU;

  // Add tune CPU to the Key.
  Key += TuneCPU;

  // Keep track of the start of the feature portion of the string.
  unsigned FSStart = Key.size();

  // FIXME: This is related to the code below to reset the target options,
  // we need to know whether or not the soft float flag is set on the
  // function before we can generate a subtarget. We also need to use
  // it as a key for the subtarget since that can be the only difference
  // between two functions.
  bool SoftFloat = F.getFnAttribute("use-soft-float").getValueAsBool();
  // If the soft float attribute is set on the function turn on the soft float
  // subtarget feature.
  if (SoftFloat)
    Key += FS.empty() ? "+soft-float" : "+soft-float,";

  Key += FS;

  // We may have added +soft-float to the features so move the StringRef to
  // point to the full string in the Key.
  FS = Key.substr(FSStart);

  auto &I = SubtargetMap[Key];
  if (!I) {
    // This needs to be done before we create a new subtarget since any
    // creation will depend on the TM and the code generation flags on the
    // function that reside in TargetOptions.
    resetTargetOptions(F);
    I = std::make_unique<X86Subtarget>(
        TargetTriple, CPU, TuneCPU, FS, *this,
        MaybeAlign(F.getParent()->getOverrideStackAlignment()),
        PreferVectorWidthOverride, RequiredVectorWidth);
```
**EN:** This section implements control flow, matching, and data updates for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的控制流、匹配与数据更新。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 301-350: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
  }
  return I.get();
}

yaml::MachineFunctionInfo *X86TargetMachine::createDefaultFuncInfoYAML() const {
  return new yaml::X86MachineFunctionInfo();
}

yaml::MachineFunctionInfo *
X86TargetMachine::convertFuncInfoToYAML(const MachineFunction &MF) const {
  const auto *MFI = MF.getInfo<X86MachineFunctionInfo>();
  return new yaml::X86MachineFunctionInfo(*MFI);
}

bool X86TargetMachine::parseMachineFunctionInfo(
    const yaml::MachineFunctionInfo &MFI, PerFunctionMIParsingState &PFS,
    SMDiagnostic &Error, SMRange &SourceRange) const {
  const auto &YamlMFI = static_cast<const yaml::X86MachineFunctionInfo &>(MFI);
  PFS.MF.getInfo<X86MachineFunctionInfo>()->initializeBaseYamlFields(YamlMFI);
  return false;
}

bool X86TargetMachine::isNoopAddrSpaceCast(unsigned SrcAS,
                                           unsigned DestAS) const {
  assert(SrcAS != DestAS && "Expected different address spaces!");
  if (getPointerSize(SrcAS) != getPointerSize(DestAS))
    return false;
  return SrcAS < 256 && DestAS < 256;
}

void X86TargetMachine::reset() { SubtargetMap.clear(); }

ScheduleDAGInstrs *
X86TargetMachine::createMachineScheduler(MachineSchedContext *C) const {
  ScheduleDAGMILive *DAG = createSchedLive(C);
  DAG->addMutation(createX86MacroFusionDAGMutation());
  return DAG;
}

ScheduleDAGInstrs *
X86TargetMachine::createPostMachineScheduler(MachineSchedContext *C) const {
  ScheduleDAGMI *DAG = createSchedPostRA(C);
  DAG->addMutation(createX86MacroFusionDAGMutation());
  return DAG;
}

//===----------------------------------------------------------------------===//
// X86 TTI query.
//===----------------------------------------------------------------------===//

```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 351-400: Namespace scope management / 命名空间作用域管理
```cpp
TargetTransformInfo
X86TargetMachine::getTargetTransformInfo(const Function &F) const {
  return TargetTransformInfo(std::make_unique<X86TTIImpl>(this, F));
}

//===----------------------------------------------------------------------===//
// Pass Pipeline Configuration
//===----------------------------------------------------------------------===//

namespace {

/// X86 Code Generator Pass Configuration Options.
class X86PassConfig : public TargetPassConfig {
public:
  X86PassConfig(X86TargetMachine &TM, PassManagerBase &PM)
    : TargetPassConfig(TM, PM) {}

  X86TargetMachine &getX86TargetMachine() const {
    return getTM<X86TargetMachine>();
  }

  void addIRPasses() override;
  bool addInstSelector() override;
  bool addIRTranslator() override;
  bool addLegalizeMachineIR() override;
  void addPreRegBankSelect() override;
  bool addRegBankSelect() override;
  bool addGlobalInstructionSelect() override;
  void addPreLegalizeMachineIR() override;
  bool addILPOpts() override;
  bool addPreISel() override;
  void addMachineSSAOptimization() override;
  void addPreRegAlloc() override;
  bool addPostFastRegAllocRewrite() override;
  void addPostRegAlloc() override;
  void addPreEmitPass() override;
  void addPreEmitPass2() override;
  void addPreSched2() override;
  bool addRegAssignAndRewriteOptimized() override;

  std::unique_ptr<CSEConfigBase> getCSEConfig() const override;
};

class X86ExecutionDomainFix : public ExecutionDomainFix {
public:
  static char ID;
  X86ExecutionDomainFix() : ExecutionDomainFix(ID, X86::VR128XRegClass) {}
  StringRef getPassName() const override {
    return "X86 Execution Dependency Fix";
  }
```
**EN:** This section implements namespace scope management for the core X86 backend. Key symbols include X86PassConfig, X86ExecutionDomainFix, X86TargetMachine::getTargetTransformInfo. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的命名空间作用域管理。关键符号包括 X86PassConfig, X86ExecutionDomainFix, X86TargetMachine::getTargetTransformInfo。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 401-450: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
};
char X86ExecutionDomainFix::ID;

} // end anonymous namespace

INITIALIZE_PASS_BEGIN(X86ExecutionDomainFix, "x86-execution-domain-fix",
  "X86 Execution Domain Fix", false, false)
INITIALIZE_PASS_DEPENDENCY(ReachingDefInfoWrapperPass)
INITIALIZE_PASS_END(X86ExecutionDomainFix, "x86-execution-domain-fix",
  "X86 Execution Domain Fix", false, false)

TargetPassConfig *X86TargetMachine::createPassConfig(PassManagerBase &PM) {
  return new X86PassConfig(*this, PM);
}

MachineFunctionInfo *X86TargetMachine::createMachineFunctionInfo(
    BumpPtrAllocator &Allocator, const Function &F,
    const TargetSubtargetInfo *STI) const {
  return X86MachineFunctionInfo::create<X86MachineFunctionInfo>(Allocator, F,
                                                                STI);
}

void X86PassConfig::addIRPasses() {
  addPass(createAtomicExpandLegacyPass());

  // We add both pass anyway and when these two passes run, we skip the pass
  // based on the option level and option attribute.
  addPass(createX86LowerAMXIntrinsicsLegacyPass());
  addPass(createX86LowerAMXTypeLegacyPass());

  TargetPassConfig::addIRPasses();

  if (TM->getOptLevel() != CodeGenOptLevel::None) {
    addPass(createInterleavedAccessPass());
    addPass(createX86PartialReductionLegacyPass());
  }

  // Add passes that handle indirect branch removal and insertion of a retpoline
  // thunk. These will be a no-op unless a function subtarget has the retpoline
  // feature enabled.
  addPass(createIndirectBrExpandPass());

  // Add Control Flow Guard checks.
  const Triple &TT = TM->getTargetTriple();
  if (TT.isOSWindows()) {
    addPass(createCFGuardPass());
  }

  if (TM->Options.JMCInstrument)
    addPass(createJMCInstrumenterPass());
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 451-500: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
}

bool X86PassConfig::addInstSelector() {
  // Install an instruction selector.
  addPass(createX86ISelDag(getX86TargetMachine(), getOptLevel()));

  // For ELF, cleanup any local-dynamic TLS accesses.
  if (TM->getTargetTriple().isOSBinFormatELF() &&
      getOptLevel() != CodeGenOptLevel::None)
    addPass(createCleanupLocalDynamicTLSLegacyPass());

  addPass(createX86GlobalBaseRegLegacyPass());
  addPass(createX86ArgumentStackSlotLegacyPass());
  return false;
}

bool X86PassConfig::addIRTranslator() {
  addPass(new IRTranslator(getOptLevel()));
  return false;
}

void X86PassConfig::addPreRegBankSelect() {
  bool IsOptNone = getOptLevel() == CodeGenOptLevel::None;
  if (!IsOptNone) {
    addPass(createX86PostLegalizerCombinerLegacy());
  }
}
bool X86PassConfig::addLegalizeMachineIR() {
  addPass(new Legalizer());
  return false;
}

bool X86PassConfig::addRegBankSelect() {
  addPass(new RegBankSelect());
  return false;
}

bool X86PassConfig::addGlobalInstructionSelect() {
  addPass(new InstructionSelect(getOptLevel()));
  // Add GlobalBaseReg in case there is no SelectionDAG passes afterwards
  if (isGlobalISelAbortEnabled())
    addPass(createX86GlobalBaseRegLegacyPass());
  return false;
}

void X86PassConfig::addPreLegalizeMachineIR() {
  if (getOptLevel() != CodeGenOptLevel::None) {
    addPass(createX86PreLegalizerCombinerLegacy());
  }
}
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 501-550: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp

bool X86PassConfig::addILPOpts() {
  addPass(&EarlyIfConverterLegacyID);
  if (X86EnableMachineCombinerPass)
    addPass(&MachineCombinerID);
  addPass(createX86CmovConversionLegacyPass());
  return true;
}

bool X86PassConfig::addPreISel() {
  // Only add this pass for 32-bit x86 Windows.
  const Triple &TT = TM->getTargetTriple();
  if (TT.isOSWindows() && TT.isX86_32())
    addPass(createX86WinEHStateLegacyPass());
  return true;
}

void X86PassConfig::addPreRegAlloc() {
  if (getOptLevel() != CodeGenOptLevel::None) {
    addPass(&LiveRangeShrinkID);
    addPass(createX86FixupSetCCLegacyPass());
    addPass(createX86OptimizeLEAsLegacyPass());
    addPass(createX86CallFrameOptimizationLegacyPass());
    addPass(createX86AvoidStoreForwardingBlocksLegacyPass());
  }

  addPass(createX86SuppressAPXForRelocationLegacyPass());

  addPass(createX86SpeculativeLoadHardeningLegacyPass());
  addPass(createX86FlagsCopyLoweringLegacyPass());
  addPass(createX86DynAllocaExpanderLegacyPass());

  if (getOptLevel() != CodeGenOptLevel::None)
    addPass(createX86PreTileConfigLegacyPass());
  else
    addPass(createX86FastPreTileConfigLegacyPass());
}

void X86PassConfig::addMachineSSAOptimization() {
  addPass(createX86DomainReassignmentLegacyPass());
  TargetPassConfig::addMachineSSAOptimization();
}

void X86PassConfig::addPostRegAlloc() {
  addPass(createX86LowerTileCopyLegacyPass());
  addPass(createX86FPStackifierLegacyPass());
  // When -O0 is enabled, the Load Value Injection Hardening pass will fall back
  // to using the Speculative Execution Side Effect Suppression pass for
  // mitigation. This is to prevent slow downs due to
  // analyses needed by the LVIHardening pass when compiling at -O0.
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. Key symbols include X86PassConfig::addILPOpts. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。关键符号包括 X86PassConfig::addILPOpts。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 551-600: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
  if (getOptLevel() != CodeGenOptLevel::None)
    addPass(createX86LoadValueInjectionLoadHardeningLegacyPass());
}

void X86PassConfig::addPreSched2() {
  addPass(createX86ExpandPseudoLegacyPass());
  addPass(createKCFIPass());
}

void X86PassConfig::addPreEmitPass() {
  if (getOptLevel() != CodeGenOptLevel::None) {
    addPass(new X86ExecutionDomainFix());
    addPass(createBreakFalseDeps());
  }

  addPass(createX86IndirectBranchTrackingLegacyPass());

  addPass(createX86InsertVZeroUpperLegacyPass());

  if (getOptLevel() != CodeGenOptLevel::None) {
    addPass(createX86FixupBWInstsLegacyPass());
    addPass(createX86PadShortFunctions());
    addPass(createX86FixupLEAsLegacyPass());
    addPass(createX86FixupInstTuningLegacyPass());
    addPass(createX86FixupVectorConstantsLegacyPass());
  }
  addPass(createX86CompressEVEXLegacyPass());
  addPass(createX86InsertX87WaitLegacyPass());
}

void X86PassConfig::addPreEmitPass2() {
  const Triple &TT = TM->getTargetTriple();
  const MCAsmInfo &MAI = TM->getMCAsmInfo();

  // The X86 Speculative Execution Pass must run after all control
  // flow graph modifying passes. As a result it was listed to run right before
  // the X86 Retpoline Thunks pass. The reason it must run after control flow
  // graph modifications is that the model of LFENCE in LLVM has to be updated
  // (FIXME: https://bugs.llvm.org/show_bug.cgi?id=45167). Currently the
  // placement of this pass was hand checked to ensure that the subsequent
  // passes don't move the code around the LFENCEs in a way that will hurt the
  // correctness of this pass. This placement has been shown to work based on
  // hand inspection of the codegen output.
  addPass(createX86SpeculativeExecutionSideEffectSuppressionLegacyPass());
  addPass(createX86IndirectThunksPass());
  addPass(createX86ReturnThunksLegacyPass());

  // Insert extra int3 instructions after trailing call instructions to avoid
  // issues in the unwinder.
  if (TT.isOSWindows() && TT.isX86_64())
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 601-650: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
    addPass(createX86AvoidTrailingCallLegacyPass());

  // Verify basic block incoming and outgoing cfa offset and register values and
  // correct CFA calculation rule where needed by inserting appropriate CFI
  // instructions.
  if (!TT.isOSDarwin() &&
      (!TT.isOSWindows() ||
       MAI.getExceptionHandlingType() == ExceptionHandling::DwarfCFI))
    addPass(createCFIInstrInserter());

  if (TT.isOSWindows()) {
    // Identify valid longjmp targets for Windows Control Flow Guard.
    addPass(createCFGuardLongjmpPass());
    // Identify valid eh continuation targets for Windows EHCont Guard.
    addPass(createEHContGuardTargetsPass());
  }
  addPass(createX86LoadValueInjectionRetHardeningLegacyPass());

  // Insert pseudo probe annotation for callsite profiling
  addPass(createPseudoProbeInserter());

  // KCFI indirect call checks are lowered to a bundle, and on Darwin platforms,
  // also CALL_RVMARKER.
  addPass(createUnpackMachineBundlesLegacy([&TT](const MachineFunction &MF) {
    // Only run bundle expansion if the module uses kcfi, or there are relevant
    // ObjC runtime functions present in the module.
    const Function &F = MF.getFunction();
    const Module *M = F.getParent();
    return M->getModuleFlag("kcfi") ||
           (TT.isOSDarwin() &&
            (M->getFunction("objc_retainAutoreleasedReturnValue") ||
             M->getFunction("objc_unsafeClaimAutoreleasedReturnValue")));
  }));

  // Analyzes and emits pseudos to support Win x64 Unwind V2. This pass must run
  // after all real instructions have been added to the epilog.
  if (TT.isOSWindows() && TT.isX86_64())
    addPass(createX86WinEHUnwindV2LegacyPass());
}

bool X86PassConfig::addPostFastRegAllocRewrite() {
  addPass(createX86FastTileConfigLegacyPass());
  return true;
}

std::unique_ptr<CSEConfigBase> X86PassConfig::getCSEConfig() const {
  return getStandardCSEConfigForOpt(TM->getOptLevel());
}

static bool onlyAllocateTileRegisters(const TargetRegisterInfo &TRI,
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 651-665: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
                                      const MachineRegisterInfo &MRI,
                                      const Register Reg) {
  const TargetRegisterClass *RC = MRI.getRegClass(Reg);
  return static_cast<const X86RegisterInfo &>(TRI).isTileRegisterClass(RC);
}

bool X86PassConfig::addRegAssignAndRewriteOptimized() {
  // Don't support tile RA when RA is specified by command line "-regalloc".
  if (!isCustomizedRegAlloc() && EnableTileRAPass) {
    // Allocate tile register first.
    addPass(createGreedyRegisterAllocator(onlyAllocateTileRegisters));
    addPass(createX86TileConfigLegacyPass());
  }
  return TargetPassConfig::addRegAssignAndRewriteOptimized();
}
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

## Key Concepts / 关键概念
- Primary topic: target machine setup. / 核心主题：目标机器初始化。
- Subsystem: the core X86 backend. / 所属子系统：X86 后端核心。
- Notable symbols: X86PassConfig, X86ExecutionDomainFix. / 重要符号：X86PassConfig, X86ExecutionDomainFix。
- The file emphasizes executable backend logic and target-specific decision making. / 该文件以可执行后端逻辑和目标特定决策为主。
- The implementation follows LLVM namespace conventions. / 实现遵循 LLVM 的命名空间约定。

## Dependencies / 依赖关系
- Direct includes: X86TargetMachine.h, MCTargetDesc/X86MCTargetDesc.h, TargetInfo/X86TargetInfo.h, X86.h, X86MachineFunctionInfo.h, X86MacroFusion.h, X86Subtarget.h, X86TargetObjectFile.h, X86TargetTransformInfo.h, llvm-c/Visibility.h. / 直接包含：X86TargetMachine.h, MCTargetDesc/X86MCTargetDesc.h, TargetInfo/X86TargetInfo.h, X86.h, X86MachineFunctionInfo.h, X86MacroFusion.h, X86Subtarget.h, X86TargetObjectFile.h, X86TargetTransformInfo.h, llvm-c/Visibility.h。
- Additional include dependencies: 30 more headers. / 额外包含依赖：还有 30 个头文件。
- Integrates with shared LLVM backend layers such as IR, CodeGen, MC, and X86 target support as needed. / 按需与 LLVM 的 IR、CodeGen、MC 和 X86 目标支持层集成。
