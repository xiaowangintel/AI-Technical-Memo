# BPFTargetMachine.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/BPF/BPFTargetMachine.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Defines the top-level TargetMachine configuration, pass pipeline hooks, and data layout integration.
- 目的（中文）: 定义顶层 TargetMachine 配置、Pass 管线钩子以及数据布局集成。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- BPFTargetMachine.cpp - Define TargetMachine for BPF ---------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // Implements the info about BPF target spec.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "BPFTargetMachine.h"
  14: #include "BPF.h"
  15: #include "BPFTargetLoweringObjectFile.h"
  16: #include "BPFTargetTransformInfo.h"
  17: #include "MCTargetDesc/BPFMCAsmInfo.h"
  18: #include "TargetInfo/BPFTargetInfo.h"
  19: #include "llvm/CodeGen/GlobalISel/IRTranslator.h"
  20: #include "llvm/CodeGen/GlobalISel/InstructionSelect.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. At the backend level, this chunk helps wire the target into LLVM’s pass pipeline and code-generation configuration.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 在后端层面，这一段帮助把目标接入 LLVM 的 Pass 管线和代码生成配置。

### Lines 21-40

```cpp
  21: #include "llvm/CodeGen/GlobalISel/Legalizer.h"
  22: #include "llvm/CodeGen/GlobalISel/RegBankSelect.h"
  23: #include "llvm/CodeGen/Passes.h"
  24: #include "llvm/CodeGen/TargetLoweringObjectFileImpl.h"
  25: #include "llvm/CodeGen/TargetPassConfig.h"
  26: #include "llvm/IR/PassManager.h"
  27: #include "llvm/InitializePasses.h"
  28: #include "llvm/MC/TargetRegistry.h"
  29: #include "llvm/Passes/PassBuilder.h"
  30: #include "llvm/Support/Compiler.h"
  31: #include "llvm/Target/TargetOptions.h"
  32: #include "llvm/Transforms/Scalar.h"
  33: #include "llvm/Transforms/Scalar/SimplifyCFG.h"
  34: #include "llvm/Transforms/Utils/SimplifyCFGOptions.h"
  35: #include <optional>
  36: using namespace llvm;
  37: 
  38: static cl::
  39: opt<bool> DisableMIPeephole("disable-bpf-peephole", cl::Hidden,
  40:                             cl::desc("Disable machine peepholes for BPF"));
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. At the backend level, this chunk helps wire the target into LLVM’s pass pipeline and code-generation configuration.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 在后端层面，这一段帮助把目标接入 LLVM 的 Pass 管线和代码生成配置。

### Lines 41-60

```cpp
  41: 
  42: static cl::opt<bool>
  43:     DisableCheckUnreachable("bpf-disable-trap-unreachable", cl::Hidden,
  44:                             cl::desc("Disable Trap Unreachable for BPF"));
  45: 
  46: extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void LLVMInitializeBPFTarget() {
  47:   // Register the target.
  48:   RegisterTargetMachine<BPFTargetMachine> X(getTheBPFleTarget());
  49:   RegisterTargetMachine<BPFTargetMachine> Y(getTheBPFbeTarget());
  50:   RegisterTargetMachine<BPFTargetMachine> Z(getTheBPFTarget());
  51: 
  52:   PassRegistry &PR = *PassRegistry::getPassRegistry();
  53:   initializeGlobalISel(PR);
  54:   initializeBPFAsmPrinterPass(PR);
  55:   initializeBPFCheckAndAdjustIRPass(PR);
  56:   initializeBPFMIPeepholePass(PR);
  57:   initializeBPFMIPreEmitPeepholePass(PR);
  58:   initializeBPFDAGToDAGISelLegacyPass(PR);
  59:   initializeBPFMISimplifyPatchablePass(PR);
  60:   initializeBPFMIPreEmitCheckingPass(PR);
```

- EN: At the backend level, this chunk helps wire the target into LLVM’s pass pipeline and code-generation configuration.
- 中文: 在后端层面，这一段帮助把目标接入 LLVM 的 Pass 管线和代码生成配置。

### Lines 61-80

```cpp
  61: }
  62: 
  63: static Reloc::Model getEffectiveRelocModel(std::optional<Reloc::Model> RM) {
  64:   return RM.value_or(Reloc::PIC_);
  65: }
  66: 
  67: BPFTargetMachine::BPFTargetMachine(const Target &T, const Triple &TT,
  68:                                    StringRef CPU, StringRef FS,
  69:                                    const TargetOptions &Options,
  70:                                    std::optional<Reloc::Model> RM,
  71:                                    std::optional<CodeModel::Model> CM,
  72:                                    CodeGenOptLevel OL, bool JIT)
  73:     : CodeGenTargetMachineImpl(T, TT.computeDataLayout(), TT, CPU, FS, Options,
  74:                                getEffectiveRelocModel(RM),
  75:                                getEffectiveCodeModel(CM, CodeModel::Small), OL),
  76:       TLOF(std::make_unique<BPFTargetLoweringObjectFileELF>()),
  77:       Subtarget(TT, std::string(CPU), std::string(FS), *this) {
  78:   if (!DisableCheckUnreachable) {
  79:     this->Options.TrapUnreachable = true;
  80:     this->Options.NoTrapAfterNoreturn = true;
```

- EN: Function bodies or method definitions such as getEffectiveRelocModel, CodeGenTargetMachineImpl contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk helps wire the target into LLVM’s pass pipeline and code-generation configuration.
- 中文: getEffectiveRelocModel, CodeGenTargetMachineImpl 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段帮助把目标接入 LLVM 的 Pass 管线和代码生成配置。

### Lines 81-100

```cpp
  81:   }
  82: 
  83:   initAsmInfo();
  84: 
  85:   BPFMCAsmInfo *MAI =
  86:       static_cast<BPFMCAsmInfo *>(const_cast<MCAsmInfo *>(AsmInfo.get()));
  87:   MAI->setDwarfUsesRelocationsAcrossSections(!Subtarget.getUseDwarfRIS());
  88: }
  89: 
  90: namespace {
  91: // BPF Code Generator Pass Configuration Options.
  92: class BPFPassConfig : public TargetPassConfig {
  93: public:
  94:   BPFPassConfig(BPFTargetMachine &TM, PassManagerBase &PM)
  95:       : TargetPassConfig(TM, PM) {}
  96: 
  97:   BPFTargetMachine &getBPFTargetMachine() const {
  98:     return getTM<BPFTargetMachine>();
  99:   }
 100: 
```

- EN: This chunk introduces interfaces or data structures such as BPFPassConfig, which organize the target-specific behavior exposed by the file. Function bodies or method definitions such as BPFPassConfig contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk helps wire the target into LLVM’s pass pipeline and code-generation configuration.
- 中文: 这一段引入了 BPFPassConfig 等接口或数据结构，用于组织该文件暴露的目标专用行为。 BPFPassConfig 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段帮助把目标接入 LLVM 的 Pass 管线和代码生成配置。

### Lines 101-120

```cpp
 101:   void addIRPasses() override;
 102:   bool addInstSelector() override;
 103:   void addMachineSSAOptimization() override;
 104:   void addPreEmitPass() override;
 105: 
 106:   bool addIRTranslator() override;
 107:   bool addLegalizeMachineIR() override;
 108:   bool addRegBankSelect() override;
 109:   bool addGlobalInstructionSelect() override;
 110: };
 111: }
 112: 
 113: TargetPassConfig *BPFTargetMachine::createPassConfig(PassManagerBase &PM) {
 114:   return new BPFPassConfig(*this, PM);
 115: }
 116: 
 117: static Expected<bool> parseBPFPreserveStaticOffsetOptions(StringRef Params) {
 118:   return PassBuilder::parseSinglePassOption(Params, "allow-partial",
 119:                                             "BPFPreserveStaticOffsetPass");
 120: }
```

- EN: Function bodies or method definitions such as parseBPFPreserveStaticOffsetOptions contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk helps wire the target into LLVM’s pass pipeline and code-generation configuration.
- 中文: parseBPFPreserveStaticOffsetOptions 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段帮助把目标接入 LLVM 的 Pass 管线和代码生成配置。

### Lines 121-140

```cpp
 121: 
 122: void BPFTargetMachine::registerPassBuilderCallbacks(PassBuilder &PB) {
 123: #define GET_PASS_REGISTRY "BPFPassRegistry.def"
 124: #include "llvm/Passes/TargetPassRegistry.inc"
 125: 
 126:   PB.registerPipelineStartEPCallback(
 127:       [=](ModulePassManager &MPM, OptimizationLevel) {
 128:         FunctionPassManager FPM;
 129:         FPM.addPass(BPFPreserveStaticOffsetPass(true));
 130:         FPM.addPass(BPFAbstractMemberAccessPass(this));
 131:         FPM.addPass(BPFPreserveDITypePass());
 132:         FPM.addPass(BPFIRPeepholePass());
 133:         MPM.addPass(createModuleToFunctionPassAdaptor(std::move(FPM)));
 134:       });
 135:   PB.registerPeepholeEPCallback([=](FunctionPassManager &FPM,
 136:                                     OptimizationLevel Level) {
 137:     FPM.addPass(SimplifyCFGPass(SimplifyCFGOptions().hoistCommonInsts(true)));
 138:     FPM.addPass(BPFASpaceCastSimplifyPass());
 139:   });
 140:   PB.registerScalarOptimizerLateEPCallback(
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. Function bodies or method definitions such as registerPassBuilderCallbacks contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk helps wire the target into LLVM’s pass pipeline and code-generation configuration.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 registerPassBuilderCallbacks 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段帮助把目标接入 LLVM 的 Pass 管线和代码生成配置。

### Lines 141-160

```cpp
 141:       [=](FunctionPassManager &FPM, OptimizationLevel Level) {
 142:         // Run this after loop unrolling but before
 143:         // SimplifyCFGPass(... .sinkCommonInsts(true))
 144:         FPM.addPass(BPFPreserveStaticOffsetPass(false));
 145:       });
 146:   PB.registerPipelineEarlySimplificationEPCallback(
 147:       [=](ModulePassManager &MPM, OptimizationLevel, ThinOrFullLTOPhase) {
 148:         MPM.addPass(BPFAdjustOptPass());
 149:       });
 150: }
 151: 
 152: void BPFPassConfig::addIRPasses() {
 153:   addPass(createAtomicExpandLegacyPass());
 154:   addPass(createBPFCheckAndAdjustIR());
 155: 
 156:   TargetPassConfig::addIRPasses();
 157: }
 158: 
 159: TargetTransformInfo
 160: BPFTargetMachine::getTargetTransformInfo(const Function &F) const {
```

- EN: Function bodies or method definitions such as addIRPasses, getTargetTransformInfo contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk helps wire the target into LLVM’s pass pipeline and code-generation configuration.
- 中文: addIRPasses, getTargetTransformInfo 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段帮助把目标接入 LLVM 的 Pass 管线和代码生成配置。

### Lines 161-180

```cpp
 161:   return TargetTransformInfo(std::make_unique<BPFTTIImpl>(this, F));
 162: }
 163: 
 164: // Install an instruction selector pass using
 165: // the ISelDag to gen BPF code.
 166: bool BPFPassConfig::addInstSelector() {
 167:   addPass(createBPFISelDag(getBPFTargetMachine()));
 168: 
 169:   return false;
 170: }
 171: 
 172: void BPFPassConfig::addMachineSSAOptimization() {
 173:   addPass(createBPFMISimplifyPatchablePass());
 174: 
 175:   // The default implementation must be called first as we want eBPF
 176:   // Peephole ran at last.
 177:   TargetPassConfig::addMachineSSAOptimization();
 178: 
 179:   const BPFSubtarget *Subtarget = getBPFTargetMachine().getSubtargetImpl();
 180:   if (!DisableMIPeephole) {
```

- EN: Function bodies or method definitions such as addInstSelector, addMachineSSAOptimization contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk helps wire the target into LLVM’s pass pipeline and code-generation configuration.
- 中文: addInstSelector, addMachineSSAOptimization 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段帮助把目标接入 LLVM 的 Pass 管线和代码生成配置。

### Lines 181-200

```cpp
 181:     if (Subtarget->getHasAlu32())
 182:       addPass(createBPFMIPeepholePass());
 183:   }
 184: }
 185: 
 186: void BPFPassConfig::addPreEmitPass() {
 187:   addPass(createBPFMIPreEmitCheckingPass());
 188:   if (getOptLevel() != CodeGenOptLevel::None)
 189:     if (!DisableMIPeephole)
 190:       addPass(createBPFMIPreEmitPeepholePass());
 191: }
 192: 
 193: bool BPFPassConfig::addIRTranslator() {
 194:   addPass(new IRTranslator());
 195:   return false;
 196: }
 197: 
 198: bool BPFPassConfig::addLegalizeMachineIR() {
 199:   addPass(new Legalizer());
 200:   return false;
```

- EN: Function bodies or method definitions such as addPreEmitPass, addIRTranslator, addLegalizeMachineIR contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk helps wire the target into LLVM’s pass pipeline and code-generation configuration.
- 中文: addPreEmitPass, addIRTranslator, addLegalizeMachineIR 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段帮助把目标接入 LLVM 的 Pass 管线和代码生成配置。

### Lines 201-211

```cpp
 201: }
 202: 
 203: bool BPFPassConfig::addRegBankSelect() {
 204:   addPass(new RegBankSelect());
 205:   return false;
 206: }
 207: 
 208: bool BPFPassConfig::addGlobalInstructionSelect() {
 209:   addPass(new InstructionSelect(getOptLevel()));
 210:   return false;
 211: }
```

- EN: Function bodies or method definitions such as addRegBankSelect, addGlobalInstructionSelect contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk helps wire the target into LLVM’s pass pipeline and code-generation configuration.
- 中文: addRegBankSelect, addGlobalInstructionSelect 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段帮助把目标接入 LLVM 的 Pass 管线和代码生成配置。

## Key Concepts / 关键概念

- Pass pipeline integration / Pass 管线集成
- Data layout / 数据布局
- GlobalISel pipeline / GlobalISel 管线
- IR to target lowering / IR 到目标降级
- Target machine configuration / 目标机器配置
- CPU feature modelling / CPU 特性建模
- Assembly printing / 汇编打印
- ELF object support / ELF 目标文件支持

## Dependencies / 依赖关系

- Direct includes / 直接包含: `BPFTargetMachine.h`, `BPF.h`, `BPFTargetLoweringObjectFile.h`, `BPFTargetTransformInfo.h`, `MCTargetDesc/BPFMCAsmInfo.h`, `TargetInfo/BPFTargetInfo.h`, `llvm/CodeGen/GlobalISel/IRTranslator.h`, `llvm/CodeGen/GlobalISel/InstructionSelect.h`
- LLVM subsystems / LLVM 子系统: SelectionDAG, GlobalISel
- Local companions / 本地配套文件: `BPFTargetMachine.h`
