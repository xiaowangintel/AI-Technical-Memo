# LanaiTargetMachine.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/Lanai/LanaiTargetMachine.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Defines the top-level TargetMachine configuration, pass pipeline hooks, and data layout integration.
- 目的（中文）: 定义顶层 TargetMachine 配置、Pass 管线钩子以及数据布局集成。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- LanaiTargetMachine.cpp - Define TargetMachine for Lanai ---------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // Implements the info about Lanai target spec.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "LanaiTargetMachine.h"
  14: 
  15: #include "Lanai.h"
  16: #include "LanaiMachineFunctionInfo.h"
  17: #include "LanaiTargetObjectFile.h"
  18: #include "LanaiTargetTransformInfo.h"
  19: #include "TargetInfo/LanaiTargetInfo.h"
  20: #include "llvm/Analysis/TargetTransformInfo.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. At the backend level, this chunk helps wire the target into LLVM’s pass pipeline and code-generation configuration.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 在后端层面，这一段帮助把目标接入 LLVM 的 Pass 管线和代码生成配置。

### Lines 21-40

```cpp
  21: #include "llvm/CodeGen/Passes.h"
  22: #include "llvm/CodeGen/TargetPassConfig.h"
  23: #include "llvm/MC/TargetRegistry.h"
  24: #include "llvm/Support/Compiler.h"
  25: #include "llvm/Target/TargetOptions.h"
  26: #include <optional>
  27: 
  28: using namespace llvm;
  29: 
  30: extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void LLVMInitializeLanaiTarget() {
  31:   // Register the target.
  32:   RegisterTargetMachine<LanaiTargetMachine> registered_target(
  33:       getTheLanaiTarget());
  34:   PassRegistry &PR = *PassRegistry::getPassRegistry();
  35:   initializeLanaiAsmPrinterPass(PR);
  36:   initializeLanaiDAGToDAGISelLegacyPass(PR);
  37:   initializeLanaiMemAluCombinerPass(PR);
  38: }
  39: 
  40: static Reloc::Model getEffectiveRelocModel(std::optional<Reloc::Model> RM) {
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. Function bodies or method definitions such as getEffectiveRelocModel contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk helps wire the target into LLVM’s pass pipeline and code-generation configuration.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 getEffectiveRelocModel 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段帮助把目标接入 LLVM 的 Pass 管线和代码生成配置。

### Lines 41-60

```cpp
  41:   return RM.value_or(Reloc::PIC_);
  42: }
  43: 
  44: LanaiTargetMachine::LanaiTargetMachine(
  45:     const Target &T, const Triple &TT, StringRef Cpu, StringRef FeatureString,
  46:     const TargetOptions &Options, std::optional<Reloc::Model> RM,
  47:     std::optional<CodeModel::Model> CodeModel, CodeGenOptLevel OptLevel,
  48:     bool JIT)
  49:     : CodeGenTargetMachineImpl(
  50:           T, TT.computeDataLayout(), TT, Cpu, FeatureString, Options,
  51:           getEffectiveRelocModel(RM),
  52:           getEffectiveCodeModel(CodeModel, CodeModel::Medium), OptLevel),
  53:       Subtarget(TT, Cpu, FeatureString, *this, Options, getCodeModel(),
  54:                 OptLevel),
  55:       TLOF(new LanaiTargetObjectFile()) {
  56:   initAsmInfo();
  57: }
  58: 
  59: TargetTransformInfo
  60: LanaiTargetMachine::getTargetTransformInfo(const Function &F) const {
```

- EN: Function bodies or method definitions such as CodeGenTargetMachineImpl, getTargetTransformInfo contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk helps wire the target into LLVM’s pass pipeline and code-generation configuration.
- 中文: CodeGenTargetMachineImpl, getTargetTransformInfo 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段帮助把目标接入 LLVM 的 Pass 管线和代码生成配置。

### Lines 61-80

```cpp
  61:   return TargetTransformInfo(std::make_unique<LanaiTTIImpl>(this, F));
  62: }
  63: 
  64: MachineFunctionInfo *LanaiTargetMachine::createMachineFunctionInfo(
  65:     BumpPtrAllocator &Allocator, const Function &F,
  66:     const TargetSubtargetInfo *STI) const {
  67:   return LanaiMachineFunctionInfo::create<LanaiMachineFunctionInfo>(Allocator,
  68:                                                                     F, STI);
  69: }
  70: 
  71: namespace {
  72: // Lanai Code Generator Pass Configuration Options.
  73: class LanaiPassConfig : public TargetPassConfig {
  74: public:
  75:   LanaiPassConfig(LanaiTargetMachine &TM, PassManagerBase *PassManager)
  76:       : TargetPassConfig(TM, *PassManager) {}
  77: 
  78:   LanaiTargetMachine &getLanaiTargetMachine() const {
  79:     return getTM<LanaiTargetMachine>();
  80:   }
```

- EN: This chunk introduces interfaces or data structures such as LanaiPassConfig, which organize the target-specific behavior exposed by the file. Function bodies or method definitions such as LanaiPassConfig contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk helps wire the target into LLVM’s pass pipeline and code-generation configuration.
- 中文: 这一段引入了 LanaiPassConfig 等接口或数据结构，用于组织该文件暴露的目标专用行为。 LanaiPassConfig 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段帮助把目标接入 LLVM 的 Pass 管线和代码生成配置。

### Lines 81-100

```cpp
  81: 
  82:   void addIRPasses() override;
  83:   bool addInstSelector() override;
  84:   void addPreSched2() override;
  85:   void addPreEmitPass() override;
  86: };
  87: } // namespace
  88: 
  89: TargetPassConfig *
  90: LanaiTargetMachine::createPassConfig(PassManagerBase &PassManager) {
  91:   return new LanaiPassConfig(*this, &PassManager);
  92: }
  93: 
  94: void LanaiPassConfig::addIRPasses() {
  95:   addPass(createAtomicExpandLegacyPass());
  96: 
  97:   TargetPassConfig::addIRPasses();
  98: }
  99: 
 100: // Install an instruction selector pass.
```

- EN: Function bodies or method definitions such as createPassConfig, addIRPasses contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk helps wire the target into LLVM’s pass pipeline and code-generation configuration.
- 中文: createPassConfig, addIRPasses 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段帮助把目标接入 LLVM 的 Pass 管线和代码生成配置。

### Lines 101-116

```cpp
 101: bool LanaiPassConfig::addInstSelector() {
 102:   addPass(createLanaiISelDag(getLanaiTargetMachine()));
 103:   return false;
 104: }
 105: 
 106: // Implemented by targets that want to run passes immediately before
 107: // machine code is emitted.
 108: void LanaiPassConfig::addPreEmitPass() {
 109:   addPass(createLanaiDelaySlotFillerPass(getLanaiTargetMachine()));
 110: }
 111: 
 112: // Run passes after prolog-epilog insertion and before the second instruction
 113: // scheduling pass.
 114: void LanaiPassConfig::addPreSched2() {
 115:   addPass(createLanaiMemAluCombinerPass());
 116: }
```

- EN: Function bodies or method definitions such as addInstSelector, addPreEmitPass, addPreSched2 contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk helps wire the target into LLVM’s pass pipeline and code-generation configuration.
- 中文: addInstSelector, addPreEmitPass, addPreSched2 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段帮助把目标接入 LLVM 的 Pass 管线和代码生成配置。

## Key Concepts / 关键概念

- Pass pipeline integration / Pass 管线集成
- Data layout / 数据布局
- MachineFunction state / MachineFunction 状态
- Target machine configuration / 目标机器配置
- CPU feature modelling / CPU 特性建模
- Assembly printing / 汇编打印
- Scheduling model / 调度模型

## Dependencies / 依赖关系

- Direct includes / 直接包含: `LanaiTargetMachine.h`, `Lanai.h`, `LanaiMachineFunctionInfo.h`, `LanaiTargetObjectFile.h`, `LanaiTargetTransformInfo.h`, `TargetInfo/LanaiTargetInfo.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/CodeGen/Passes.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen
- Local companions / 本地配套文件: `LanaiTargetMachine.h`
