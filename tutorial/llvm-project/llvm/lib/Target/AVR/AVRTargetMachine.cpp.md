# AVRTargetMachine.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/AVR/AVRTargetMachine.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file defines the AVR specific subclass of TargetMachine.
- 目的（中文）: 定义顶层 TargetMachine 配置、Pass 管线钩子以及数据布局集成。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- AVRTargetMachine.cpp - Define TargetMachine for AVR ---------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file defines the AVR specific subclass of TargetMachine.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "AVRTargetMachine.h"
  14: 
  15: #include "llvm/CodeGen/Passes.h"
  16: #include "llvm/CodeGen/TargetPassConfig.h"
  17: #include "llvm/MC/TargetRegistry.h"
  18: #include "llvm/Support/Compiler.h"
  19: 
  20: #include "AVR.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. At the backend level, this chunk helps wire the target into LLVM’s pass pipeline and code-generation configuration.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 在后端层面，这一段帮助把目标接入 LLVM 的 Pass 管线和代码生成配置。

### Lines 21-40

```cpp
  21: #include "AVRMachineFunctionInfo.h"
  22: #include "AVRTargetObjectFile.h"
  23: #include "AVRTargetTransformInfo.h"
  24: #include "MCTargetDesc/AVRMCTargetDesc.h"
  25: #include "TargetInfo/AVRTargetInfo.h"
  26: 
  27: #include <optional>
  28: 
  29: namespace llvm {
  30: 
  31: /// Processes a CPU name.
  32: static StringRef getCPU(StringRef CPU) {
  33:   if (CPU.empty() || CPU == "generic") {
  34:     return "avr2";
  35:   }
  36: 
  37:   return CPU;
  38: }
  39: 
  40: static Reloc::Model getEffectiveRelocModel(std::optional<Reloc::Model> RM) {
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. Function bodies or method definitions such as getCPU, getEffectiveRelocModel contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk helps wire the target into LLVM’s pass pipeline and code-generation configuration.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 getCPU, getEffectiveRelocModel 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段帮助把目标接入 LLVM 的 Pass 管线和代码生成配置。

### Lines 41-60

```cpp
  41:   return RM.value_or(Reloc::Static);
  42: }
  43: 
  44: AVRTargetMachine::AVRTargetMachine(const Target &T, const Triple &TT,
  45:                                    StringRef CPU, StringRef FS,
  46:                                    const TargetOptions &Options,
  47:                                    std::optional<Reloc::Model> RM,
  48:                                    std::optional<CodeModel::Model> CM,
  49:                                    CodeGenOptLevel OL, bool JIT)
  50:     : CodeGenTargetMachineImpl(T, TT.computeDataLayout(), TT, getCPU(CPU), FS,
  51:                                Options, getEffectiveRelocModel(RM),
  52:                                getEffectiveCodeModel(CM, CodeModel::Small), OL),
  53:       SubTarget(TT, std::string(getCPU(CPU)), std::string(FS), *this) {
  54:   this->TLOF = std::make_unique<AVRTargetObjectFile>();
  55:   initAsmInfo();
  56: }
  57: 
  58: namespace {
  59: /// AVR Code Generator Pass Configuration Options.
  60: class AVRPassConfig : public TargetPassConfig {
```

- EN: This chunk introduces interfaces or data structures such as AVRPassConfig, which organize the target-specific behavior exposed by the file. Function bodies or method definitions such as CodeGenTargetMachineImpl contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk helps wire the target into LLVM’s pass pipeline and code-generation configuration.
- 中文: 这一段引入了 AVRPassConfig 等接口或数据结构，用于组织该文件暴露的目标专用行为。 CodeGenTargetMachineImpl 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段帮助把目标接入 LLVM 的 Pass 管线和代码生成配置。

### Lines 61-80

```cpp
  61: public:
  62:   AVRPassConfig(AVRTargetMachine &TM, PassManagerBase &PM)
  63:       : TargetPassConfig(TM, PM) {
  64:     EnableLoopTermFold = true;
  65:   }
  66: 
  67:   AVRTargetMachine &getAVRTargetMachine() const {
  68:     return getTM<AVRTargetMachine>();
  69:   }
  70: 
  71:   void addIRPasses() override;
  72:   bool addInstSelector() override;
  73:   void addPreSched2() override;
  74:   void addPreEmitPass() override;
  75: };
  76: } // namespace
  77: 
  78: TargetPassConfig *AVRTargetMachine::createPassConfig(PassManagerBase &PM) {
  79:   return new AVRPassConfig(*this, PM);
  80: }
```

- EN: Function bodies or method definitions such as AVRPassConfig contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk helps wire the target into LLVM’s pass pipeline and code-generation configuration.
- 中文: AVRPassConfig 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段帮助把目标接入 LLVM 的 Pass 管线和代码生成配置。

### Lines 81-100

```cpp
  81: 
  82: void AVRPassConfig::addIRPasses() {
  83:   // Expand instructions like
  84:   //   %result = shl i32 %n, %amount
  85:   // to a loop so that library calls are avoided.
  86:   addPass(createAVRShiftExpandPass());
  87: 
  88:   TargetPassConfig::addIRPasses();
  89: }
  90: 
  91: extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void LLVMInitializeAVRTarget() {
  92:   // Register the target.
  93:   RegisterTargetMachine<AVRTargetMachine> X(getTheAVRTarget());
  94: 
  95:   auto &PR = *PassRegistry::getPassRegistry();
  96:   initializeAVRAsmPrinterPass(PR);
  97:   initializeAVRExpandPseudoPass(PR);
  98:   initializeAVRShiftExpandPass(PR);
  99:   initializeAVRDAGToDAGISelLegacyPass(PR);
 100: }
```

- EN: Function bodies or method definitions such as addIRPasses contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk helps wire the target into LLVM’s pass pipeline and code-generation configuration.
- 中文: addIRPasses 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段帮助把目标接入 LLVM 的 Pass 管线和代码生成配置。

### Lines 101-120

```cpp
 101: 
 102: const AVRSubtarget *AVRTargetMachine::getSubtargetImpl() const {
 103:   return &SubTarget;
 104: }
 105: 
 106: const AVRSubtarget *AVRTargetMachine::getSubtargetImpl(const Function &) const {
 107:   return &SubTarget;
 108: }
 109: 
 110: TargetTransformInfo
 111: AVRTargetMachine::getTargetTransformInfo(const Function &F) const {
 112:   return TargetTransformInfo(std::make_unique<AVRTTIImpl>(this, F));
 113: }
 114: 
 115: MachineFunctionInfo *AVRTargetMachine::createMachineFunctionInfo(
 116:     BumpPtrAllocator &Allocator, const Function &F,
 117:     const TargetSubtargetInfo *STI) const {
 118:   return AVRMachineFunctionInfo::create<AVRMachineFunctionInfo>(Allocator, F,
 119:                                                                 STI);
 120: }
```

- EN: Function bodies or method definitions such as getTargetTransformInfo contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk helps wire the target into LLVM’s pass pipeline and code-generation configuration.
- 中文: getTargetTransformInfo 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段帮助把目标接入 LLVM 的 Pass 管线和代码生成配置。

### Lines 121-140

```cpp
 121: 
 122: //===----------------------------------------------------------------------===//
 123: // Pass Pipeline Configuration
 124: //===----------------------------------------------------------------------===//
 125: 
 126: bool AVRPassConfig::addInstSelector() {
 127:   // Install an instruction selector.
 128:   addPass(createAVRISelDag(getAVRTargetMachine(), getOptLevel()));
 129:   // Create the frame analyzer pass used by the PEI pass.
 130:   addPass(createAVRFrameAnalyzerPass());
 131: 
 132:   return false;
 133: }
 134: 
 135: void AVRPassConfig::addPreSched2() { addPass(createAVRExpandPseudoPass()); }
 136: 
 137: void AVRPassConfig::addPreEmitPass() {
 138:   // Must run branch selection immediately preceding the asm printer.
 139:   addPass(&BranchRelaxationPassID);
 140: }
```

- EN: Function bodies or method definitions such as addInstSelector, addPreSched2, addPreEmitPass contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk helps wire the target into LLVM’s pass pipeline and code-generation configuration.
- 中文: addInstSelector, addPreSched2, addPreEmitPass 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段帮助把目标接入 LLVM 的 Pass 管线和代码生成配置。

### Lines 141-142

```cpp
 141: 
 142: } // end of namespace llvm
```

- EN: The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. At the backend level, this chunk helps wire the target into LLVM’s pass pipeline and code-generation configuration.
- 中文: 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 在后端层面，这一段帮助把目标接入 LLVM 的 Pass 管线和代码生成配置。

## Key Concepts / 关键概念

- Pass pipeline integration / Pass 管线集成
- Data layout / 数据布局
- MachineFunction state / MachineFunction 状态
- Target machine configuration / 目标机器配置
- CPU feature modelling / CPU 特性建模
- Assembly printing / 汇编打印
- Pseudo-instruction handling / 伪指令处理
- Scheduling model / 调度模型

## Dependencies / 依赖关系

- Direct includes / 直接包含: `AVRTargetMachine.h`, `llvm/CodeGen/Passes.h`, `llvm/CodeGen/TargetPassConfig.h`, `llvm/MC/TargetRegistry.h`, `llvm/Support/Compiler.h`, `AVR.h`, `AVRMachineFunctionInfo.h`, `AVRTargetObjectFile.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen
- Local companions / 本地配套文件: `AVRTargetMachine.h`
