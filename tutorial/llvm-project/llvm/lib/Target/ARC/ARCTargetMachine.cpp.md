# ARCTargetMachine.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/ARC/ARCTargetMachine.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Defines the top-level TargetMachine configuration, pass pipeline hooks, and data layout integration.
- 目的（中文）: 定义顶层 TargetMachine 配置、Pass 管线钩子以及数据布局集成。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===- ARCTargetMachine.cpp - Define TargetMachine for ARC ------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: //
  10: //===----------------------------------------------------------------------===//
  11: 
  12: #include "ARCTargetMachine.h"
  13: #include "ARC.h"
  14: #include "ARCMachineFunctionInfo.h"
  15: #include "ARCTargetTransformInfo.h"
  16: #include "TargetInfo/ARCTargetInfo.h"
  17: #include "llvm/CodeGen/Passes.h"
  18: #include "llvm/CodeGen/TargetLoweringObjectFileImpl.h"
  19: #include "llvm/CodeGen/TargetPassConfig.h"
  20: #include "llvm/MC/TargetRegistry.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. At the backend level, this chunk helps wire the target into LLVM’s pass pipeline and code-generation configuration.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 在后端层面，这一段帮助把目标接入 LLVM 的 Pass 管线和代码生成配置。

### Lines 21-40

```cpp
  21: #include <optional>
  22: 
  23: using namespace llvm;
  24: 
  25: static Reloc::Model getRelocModel(std::optional<Reloc::Model> RM) {
  26:   return RM.value_or(Reloc::Static);
  27: }
  28: 
  29: /// ARCTargetMachine ctor - Create an ILP32 architecture model
  30: ARCTargetMachine::ARCTargetMachine(const Target &T, const Triple &TT,
  31:                                    StringRef CPU, StringRef FS,
  32:                                    const TargetOptions &Options,
  33:                                    std::optional<Reloc::Model> RM,
  34:                                    std::optional<CodeModel::Model> CM,
  35:                                    CodeGenOptLevel OL, bool JIT)
  36:     : CodeGenTargetMachineImpl(T, TT.computeDataLayout(), TT, CPU, FS, Options,
  37:                                getRelocModel(RM),
  38:                                getEffectiveCodeModel(CM, CodeModel::Small), OL),
  39:       TLOF(std::make_unique<TargetLoweringObjectFileELF>()),
  40:       Subtarget(TT, std::string(CPU), std::string(FS), *this) {
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. Function bodies or method definitions such as getRelocModel, CodeGenTargetMachineImpl contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk helps wire the target into LLVM’s pass pipeline and code-generation configuration.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 getRelocModel, CodeGenTargetMachineImpl 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段帮助把目标接入 LLVM 的 Pass 管线和代码生成配置。

### Lines 41-60

```cpp
  41:   initAsmInfo();
  42: }
  43: 
  44: ARCTargetMachine::~ARCTargetMachine() = default;
  45: 
  46: namespace {
  47: 
  48: /// ARC Code Generator Pass Configuration Options.
  49: class ARCPassConfig : public TargetPassConfig {
  50: public:
  51:   ARCPassConfig(ARCTargetMachine &TM, PassManagerBase &PM)
  52:       : TargetPassConfig(TM, PM) {}
  53: 
  54:   ARCTargetMachine &getARCTargetMachine() const {
  55:     return getTM<ARCTargetMachine>();
  56:   }
  57: 
  58:   void addIRPasses() override;
  59:   bool addInstSelector() override;
  60:   void addPreEmitPass() override;
```

- EN: This chunk introduces interfaces or data structures such as ARCPassConfig, which organize the target-specific behavior exposed by the file. Function bodies or method definitions such as ARCPassConfig contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk helps wire the target into LLVM’s pass pipeline and code-generation configuration.
- 中文: 这一段引入了 ARCPassConfig 等接口或数据结构，用于组织该文件暴露的目标专用行为。 ARCPassConfig 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段帮助把目标接入 LLVM 的 Pass 管线和代码生成配置。

### Lines 61-80

```cpp
  61:   void addPreRegAlloc() override;
  62: };
  63: 
  64: } // end anonymous namespace
  65: 
  66: TargetPassConfig *ARCTargetMachine::createPassConfig(PassManagerBase &PM) {
  67:   return new ARCPassConfig(*this, PM);
  68: }
  69: 
  70: void ARCPassConfig::addIRPasses() {
  71:   addPass(createAtomicExpandLegacyPass());
  72: 
  73:   TargetPassConfig::addIRPasses();
  74: }
  75: 
  76: bool ARCPassConfig::addInstSelector() {
  77:   addPass(createARCISelDag(getARCTargetMachine(), getOptLevel()));
  78:   return false;
  79: }
  80: 
```

- EN: Function bodies or method definitions such as addIRPasses, addInstSelector contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk helps wire the target into LLVM’s pass pipeline and code-generation configuration.
- 中文: addIRPasses, addInstSelector 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段帮助把目标接入 LLVM 的 Pass 管线和代码生成配置。

### Lines 81-100

```cpp
  81: void ARCPassConfig::addPreEmitPass() { addPass(createARCBranchFinalizePass()); }
  82: 
  83: void ARCPassConfig::addPreRegAlloc() {
  84:     addPass(createARCExpandPseudosPass());
  85:     addPass(createARCOptAddrMode());
  86: }
  87: 
  88: MachineFunctionInfo *ARCTargetMachine::createMachineFunctionInfo(
  89:     BumpPtrAllocator &Allocator, const Function &F,
  90:     const TargetSubtargetInfo *STI) const {
  91:     return ARCFunctionInfo::create<ARCFunctionInfo>(Allocator, F, STI);
  92: }
  93: 
  94: // Force static initialization.
  95: extern "C" LLVM_EXTERNAL_VISIBILITY void LLVMInitializeARCTarget() {
  96:   RegisterTargetMachine<ARCTargetMachine> X(getTheARCTarget());
  97:   PassRegistry &PR = *PassRegistry::getPassRegistry();
  98:   initializeARCAsmPrinterPass(PR);
  99:   initializeARCDAGToDAGISelLegacyPass(PR);
 100: }
```

- EN: Function bodies or method definitions such as addPreEmitPass, addPreRegAlloc contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk helps wire the target into LLVM’s pass pipeline and code-generation configuration.
- 中文: addPreEmitPass, addPreRegAlloc 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段帮助把目标接入 LLVM 的 Pass 管线和代码生成配置。

### Lines 101-105

```cpp
 101: 
 102: TargetTransformInfo
 103: ARCTargetMachine::getTargetTransformInfo(const Function &F) const {
 104:   return TargetTransformInfo(std::make_unique<ARCTTIImpl>(this, F));
 105: }
```

- EN: Function bodies or method definitions such as getTargetTransformInfo contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk helps wire the target into LLVM’s pass pipeline and code-generation configuration.
- 中文: getTargetTransformInfo 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段帮助把目标接入 LLVM 的 Pass 管线和代码生成配置。

## Key Concepts / 关键概念

- Pass pipeline integration / Pass 管线集成
- Data layout / 数据布局
- MachineFunction state / MachineFunction 状态
- IR to target lowering / IR 到目标降级
- Target machine configuration / 目标机器配置
- CPU feature modelling / CPU 特性建模
- Assembly printing / 汇编打印
- ELF object support / ELF 目标文件支持

## Dependencies / 依赖关系

- Direct includes / 直接包含: `ARCTargetMachine.h`, `ARC.h`, `ARCMachineFunctionInfo.h`, `ARCTargetTransformInfo.h`, `TargetInfo/ARCTargetInfo.h`, `llvm/CodeGen/Passes.h`, `llvm/CodeGen/TargetLoweringObjectFileImpl.h`, `llvm/CodeGen/TargetPassConfig.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen, SelectionDAG
- Local companions / 本地配套文件: `ARCTargetMachine.h`
