# CSKYTargetMachine.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/CSKY/CSKYTargetMachine.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Defines the top-level TargetMachine configuration, pass pipeline hooks, and data layout integration.
- 目的（中文）: 定义顶层 TargetMachine 配置、Pass 管线钩子以及数据布局集成。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===--- CSKYTargetMachine.cpp - Define TargetMachine for CSKY ------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // Implements the info about CSKY target spec.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "CSKYTargetMachine.h"
  14: #include "CSKY.h"
  15: #include "CSKYMachineFunctionInfo.h"
  16: #include "CSKYSubtarget.h"
  17: #include "CSKYTargetObjectFile.h"
  18: #include "TargetInfo/CSKYTargetInfo.h"
  19: #include "llvm/CodeGen/MachineFrameInfo.h"
  20: #include "llvm/CodeGen/TargetLoweringObjectFileImpl.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. At the backend level, this chunk helps wire the target into LLVM’s pass pipeline and code-generation configuration.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 在后端层面，这一段帮助把目标接入 LLVM 的 Pass 管线和代码生成配置。

### Lines 21-40

```cpp
  21: #include "llvm/CodeGen/TargetPassConfig.h"
  22: #include "llvm/CodeGen/TargetSubtargetInfo.h"
  23: #include "llvm/MC/TargetRegistry.h"
  24: #include <optional>
  25: 
  26: using namespace llvm;
  27: 
  28: extern "C" LLVM_EXTERNAL_VISIBILITY void LLVMInitializeCSKYTarget() {
  29:   RegisterTargetMachine<CSKYTargetMachine> X(getTheCSKYTarget());
  30: 
  31:   PassRegistry *Registry = PassRegistry::getPassRegistry();
  32:   initializeCSKYConstantIslandsPass(*Registry);
  33:   initializeCSKYDAGToDAGISelLegacyPass(*Registry);
  34: }
  35: 
  36: CSKYTargetMachine::CSKYTargetMachine(const Target &T, const Triple &TT,
  37:                                      StringRef CPU, StringRef FS,
  38:                                      const TargetOptions &Options,
  39:                                      std::optional<Reloc::Model> RM,
  40:                                      std::optional<CodeModel::Model> CM,
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. At the backend level, this chunk helps wire the target into LLVM’s pass pipeline and code-generation configuration.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 在后端层面，这一段帮助把目标接入 LLVM 的 Pass 管线和代码生成配置。

### Lines 41-60

```cpp
  41:                                      CodeGenOptLevel OL, bool JIT)
  42:     : CodeGenTargetMachineImpl(T, TT.computeDataLayout(), TT, CPU, FS, Options,
  43:                                RM.value_or(Reloc::Static),
  44:                                getEffectiveCodeModel(CM, CodeModel::Small), OL),
  45:       TLOF(std::make_unique<CSKYELFTargetObjectFile>()) {
  46:   initAsmInfo();
  47: }
  48: 
  49: const CSKYSubtarget *
  50: CSKYTargetMachine::getSubtargetImpl(const Function &F) const {
  51:   Attribute CPUAttr = F.getFnAttribute("target-cpu");
  52:   Attribute TuneAttr = F.getFnAttribute("tune-cpu");
  53:   Attribute FSAttr = F.getFnAttribute("target-features");
  54: 
  55:   std::string CPU =
  56:       CPUAttr.isValid() ? CPUAttr.getValueAsString().str() : TargetCPU;
  57:   std::string TuneCPU =
  58:       TuneAttr.isValid() ? TuneAttr.getValueAsString().str() : CPU;
  59:   std::string FS =
  60:       FSAttr.isValid() ? FSAttr.getValueAsString().str() : TargetFS;
```

- EN: Function bodies or method definitions such as CodeGenTargetMachineImpl, getSubtargetImpl contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk helps wire the target into LLVM’s pass pipeline and code-generation configuration.
- 中文: CodeGenTargetMachineImpl, getSubtargetImpl 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段帮助把目标接入 LLVM 的 Pass 管线和代码生成配置。

### Lines 61-80

```cpp
  61: 
  62:   std::string Key = CPU + TuneCPU + FS;
  63:   auto &I = SubtargetMap[Key];
  64:   if (!I) {
  65:     // This needs to be done before we create a new subtarget since any
  66:     // creation will depend on the TM and the code generation flags on the
  67:     // function that reside in TargetOptions.
  68:     resetTargetOptions(F);
  69:     I = std::make_unique<CSKYSubtarget>(TargetTriple, CPU, TuneCPU, FS, *this);
  70:     if (I->useHardFloat() && !I->hasAnyFloatExt())
  71:       errs() << "Hard-float can't be used with current CPU,"
  72:                 " set to Soft-float\n";
  73:   }
  74:   return I.get();
  75: }
  76: 
  77: MachineFunctionInfo *CSKYTargetMachine::createMachineFunctionInfo(
  78:     BumpPtrAllocator &Allocator, const Function &F,
  79:     const TargetSubtargetInfo *STI) const {
  80:   return CSKYMachineFunctionInfo::create<CSKYMachineFunctionInfo>(Allocator, F,
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk helps wire the target into LLVM’s pass pipeline and code-generation configuration.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段帮助把目标接入 LLVM 的 Pass 管线和代码生成配置。

### Lines 81-100

```cpp
  81:                                                                   STI);
  82: }
  83: 
  84: namespace {
  85: class CSKYPassConfig : public TargetPassConfig {
  86: public:
  87:   CSKYPassConfig(CSKYTargetMachine &TM, PassManagerBase &PM)
  88:       : TargetPassConfig(TM, PM) {}
  89: 
  90:   CSKYTargetMachine &getCSKYTargetMachine() const {
  91:     return getTM<CSKYTargetMachine>();
  92:   }
  93: 
  94:   void addIRPasses() override;
  95:   bool addInstSelector() override;
  96:   void addPreEmitPass() override;
  97: };
  98: 
  99: } // namespace
 100: 
```

- EN: This chunk introduces interfaces or data structures such as CSKYPassConfig, which organize the target-specific behavior exposed by the file. Function bodies or method definitions such as CSKYPassConfig contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk helps wire the target into LLVM’s pass pipeline and code-generation configuration.
- 中文: 这一段引入了 CSKYPassConfig 等接口或数据结构，用于组织该文件暴露的目标专用行为。 CSKYPassConfig 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段帮助把目标接入 LLVM 的 Pass 管线和代码生成配置。

### Lines 101-118

```cpp
 101: TargetPassConfig *CSKYTargetMachine::createPassConfig(PassManagerBase &PM) {
 102:   return new CSKYPassConfig(*this, PM);
 103: }
 104: 
 105: void CSKYPassConfig::addIRPasses() {
 106:   addPass(createAtomicExpandLegacyPass());
 107:   TargetPassConfig::addIRPasses();
 108: }
 109: 
 110: bool CSKYPassConfig::addInstSelector() {
 111:   addPass(createCSKYISelDag(getCSKYTargetMachine(), getOptLevel()));
 112: 
 113:   return false;
 114: }
 115: 
 116: void CSKYPassConfig::addPreEmitPass() {
 117:   addPass(createCSKYConstantIslandPass());
 118: }
```

- EN: Function bodies or method definitions such as addIRPasses, addInstSelector, addPreEmitPass contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk helps wire the target into LLVM’s pass pipeline and code-generation configuration.
- 中文: addIRPasses, addInstSelector, addPreEmitPass 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段帮助把目标接入 LLVM 的 Pass 管线和代码生成配置。

## Key Concepts / 关键概念

- Pass pipeline integration / Pass 管线集成
- Data layout / 数据布局
- MachineFunction state / MachineFunction 状态
- IR to target lowering / IR 到目标降级
- Target machine configuration / 目标机器配置
- CPU feature modelling / CPU 特性建模
- ELF object support / ELF 目标文件支持

## Dependencies / 依赖关系

- Direct includes / 直接包含: `CSKYTargetMachine.h`, `CSKY.h`, `CSKYMachineFunctionInfo.h`, `CSKYSubtarget.h`, `CSKYTargetObjectFile.h`, `TargetInfo/CSKYTargetInfo.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/TargetLoweringObjectFileImpl.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen, SelectionDAG
- Local companions / 本地配套文件: `CSKYTargetMachine.h`
