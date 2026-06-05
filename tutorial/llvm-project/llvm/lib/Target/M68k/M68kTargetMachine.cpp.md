# M68kTargetMachine.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/M68k/M68kTargetMachine.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file configures the target machine and code-generation pipeline for the M68k backend.
- **用途 (CN)**: 该文件用于 M68k 后端，负责配置目标机器与代码生成流水线。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===-- M68kTargetMachine.cpp - M68k Target Machine -------------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: ///
   9: /// \file
  10: /// This file contains implementation for M68k target machine.
  11: ///
  12: //===----------------------------------------------------------------------===//
  13: 
  14: #include "M68kTargetMachine.h"
  15: #include "M68k.h"
  16: #include "M68kMachineFunction.h"
  17: #include "M68kSubtarget.h"
  18: #include "M68kTargetObjectFile.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `M68kTargetMachine.h`, `M68k.h`, `M68kMachineFunction.h`, `M68kSubtarget.h`, `M68kTargetObjectFile.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `M68kTargetMachine.h`, `M68k.h`, `M68kMachineFunction.h`, `M68kSubtarget.h`, `M68kTargetObjectFile.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 19-36 / 第 19-36 行
```cpp
  19: #include "TargetInfo/M68kTargetInfo.h"
  20: #include "llvm/CodeGen/GlobalISel/IRTranslator.h"
  21: #include "llvm/CodeGen/GlobalISel/InstructionSelect.h"
  22: #include "llvm/CodeGen/GlobalISel/Legalizer.h"
  23: #include "llvm/CodeGen/GlobalISel/RegBankSelect.h"
  24: #include "llvm/CodeGen/Passes.h"
  25: #include "llvm/CodeGen/TargetPassConfig.h"
  26: #include "llvm/InitializePasses.h"
  27: #include "llvm/MC/TargetRegistry.h"
  28: #include "llvm/PassRegistry.h"
  29: #include <memory>
  30: #include <optional>
  31: 
  32: using namespace llvm;
  33: 
  34: #define DEBUG_TYPE "m68k"
  35: 
  36: extern "C" LLVM_EXTERNAL_VISIBILITY void LLVMInitializeM68kTarget() {
```
- **EN**: It imports dependencies such as `M68kTargetInfo.h`, `IRTranslator.h`, `InstructionSelect.h`, `Legalizer.h`, `RegBankSelect.h`, `Passes.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 它引入了 `M68kTargetInfo.h`, `IRTranslator.h`, `InstructionSelect.h`, `Legalizer.h`, `RegBankSelect.h`, `Passes.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 37-54 / 第 37-54 行
```cpp
  37:   RegisterTargetMachine<M68kTargetMachine> X(getTheM68kTarget());
  38:   auto *PR = PassRegistry::getPassRegistry();
  39:   initializeGlobalISel(*PR);
  40:   initializeM68kAsmPrinterPass(*PR);
  41:   initializeM68kDAGToDAGISelLegacyPass(*PR);
  42:   initializeM68kExpandPseudoPass(*PR);
  43:   initializeM68kGlobalBaseRegPass(*PR);
  44:   initializeM68kCollapseMOVEMPass(*PR);
  45: }
  46: 
  47: namespace {
  48: 
  49: Reloc::Model getEffectiveRelocModel(std::optional<Reloc::Model> RM) {
  50:   // If not defined we default to static
  51:   return RM.value_or(Reloc::Static);
  52: }
  53: 
  54: CodeModel::Model getEffectiveCodeModel(std::optional<CodeModel::Model> CM,
```
- **EN**: The range implements or declares functions including `getEffectiveRelocModel`.
- **CN**: 这一段实现或声明了 `getEffectiveRelocModel` 等函数。

### Lines 55-72 / 第 55-72 行
```cpp
  55:                                        bool JIT) {
  56:   if (!CM) {
  57:     return CodeModel::Small;
  58:   } else if (CM == CodeModel::Kernel) {
  59:     llvm_unreachable("Kernel code model is not implemented yet");
  60:   }
  61:   return CM.value();
  62: }
  63: } // end anonymous namespace
  64: 
  65: M68kTargetMachine::M68kTargetMachine(const Target &T, const Triple &TT,
  66:                                      StringRef CPU, StringRef FS,
  67:                                      const TargetOptions &Options,
  68:                                      std::optional<Reloc::Model> RM,
  69:                                      std::optional<CodeModel::Model> CM,
  70:                                      CodeGenOptLevel OL, bool JIT)
  71:     : CodeGenTargetMachineImpl(T, TT.computeDataLayout(), TT, CPU, FS, Options,
  72:                                getEffectiveRelocModel(RM),
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 73-90 / 第 73-90 行
```cpp
  73:                                ::getEffectiveCodeModel(CM, JIT), OL),
  74:       TLOF(std::make_unique<M68kELFTargetObjectFile>()),
  75:       Subtarget(TT, CPU, FS, *this) {
  76:   initAsmInfo();
  77: }
  78: 
  79: M68kTargetMachine::~M68kTargetMachine() {}
  80: 
  81: const M68kSubtarget *
  82: M68kTargetMachine::getSubtargetImpl(const Function &F) const {
  83:   Attribute CPUAttr = F.getFnAttribute("target-cpu");
  84:   Attribute FSAttr = F.getFnAttribute("target-features");
  85: 
  86:   auto CPU = CPUAttr.isValid() ? CPUAttr.getValueAsString().str() : TargetCPU;
  87:   auto FS = FSAttr.isValid() ? FSAttr.getValueAsString().str() : TargetFS;
  88: 
  89:   auto &I = SubtargetMap[CPU + FS];
  90:   if (!I) {
```
- **EN**: The range implements or declares functions including `TLOF`, `M68kTargetMachine::getSubtargetImpl`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `TLOF`, `M68kTargetMachine::getSubtargetImpl` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 91-108 / 第 91-108 行
```cpp
  91:     // This needs to be done before we create a new subtarget since any
  92:     // creation will depend on the TM and the code generation flags on the
  93:     // function that reside in TargetOptions.
  94:     resetTargetOptions(F);
  95:     I = std::make_unique<M68kSubtarget>(TargetTriple, CPU, FS, *this);
  96:   }
  97:   return I.get();
  98: }
  99: 
 100: MachineFunctionInfo *M68kTargetMachine::createMachineFunctionInfo(
 101:     BumpPtrAllocator &Allocator, const Function &F,
 102:     const TargetSubtargetInfo *STI) const {
 103:   return M68kMachineFunctionInfo::create<M68kMachineFunctionInfo>(Allocator, F,
 104:                                                                   STI);
 105: }
 106: 
 107: //===----------------------------------------------------------------------===//
 108: // Pass Pipeline Configuration
```
- **EN**: This span continues the file's main responsibility: this file configures the target machine and code-generation pipeline for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 109-126 / 第 109-126 行
```cpp
 109: //===----------------------------------------------------------------------===//
 110: 
 111: namespace {
 112: class M68kPassConfig : public TargetPassConfig {
 113: public:
 114:   M68kPassConfig(M68kTargetMachine &TM, PassManagerBase &PM)
 115:       : TargetPassConfig(TM, PM) {}
 116: 
 117:   M68kTargetMachine &getM68kTargetMachine() const {
 118:     return getTM<M68kTargetMachine>();
 119:   }
 120: 
 121:   const M68kSubtarget &getM68kSubtarget() const {
 122:     return *getM68kTargetMachine().getSubtargetImpl();
 123:   }
 124:   void addIRPasses() override;
 125:   bool addIRTranslator() override;
 126:   bool addLegalizeMachineIR() override;
```
- **EN**: This block declares or refines TableGen records such as `M68kPassConfig`. The range implements or declares functions including `M68kPassConfig`.
- **CN**: 该代码块声明或细化了 `M68kPassConfig` 等 TableGen 记录。 这一段实现或声明了 `M68kPassConfig` 等函数。

### Lines 127-144 / 第 127-144 行
```cpp
 127:   bool addRegBankSelect() override;
 128:   bool addGlobalInstructionSelect() override;
 129:   bool addInstSelector() override;
 130:   void addPreSched2() override;
 131:   void addPreEmitPass() override;
 132: };
 133: } // namespace
 134: 
 135: TargetPassConfig *M68kTargetMachine::createPassConfig(PassManagerBase &PM) {
 136:   return new M68kPassConfig(*this, PM);
 137: }
 138: 
 139: void M68kPassConfig::addIRPasses() {
 140:   addPass(createAtomicExpandLegacyPass());
 141:   TargetPassConfig::addIRPasses();
 142: }
 143: 
 144: bool M68kPassConfig::addInstSelector() {
```
- **EN**: The range implements or declares functions including `M68kPassConfig::addIRPasses`, `M68kPassConfig::addInstSelector`.
- **CN**: 这一段实现或声明了 `M68kPassConfig::addIRPasses`, `M68kPassConfig::addInstSelector` 等函数。

### Lines 145-162 / 第 145-162 行
```cpp
 145:   // Install an instruction selector.
 146:   addPass(createM68kISelDag(getM68kTargetMachine()));
 147:   addPass(createM68kGlobalBaseRegPass());
 148:   return false;
 149: }
 150: 
 151: bool M68kPassConfig::addIRTranslator() {
 152:   addPass(new IRTranslator());
 153:   return false;
 154: }
 155: 
 156: bool M68kPassConfig::addLegalizeMachineIR() {
 157:   addPass(new Legalizer());
 158:   return false;
 159: }
 160: 
 161: bool M68kPassConfig::addRegBankSelect() {
 162:   addPass(new RegBankSelect());
```
- **EN**: The range implements or declares functions including `M68kPassConfig::addIRTranslator`, `M68kPassConfig::addLegalizeMachineIR`, `M68kPassConfig::addRegBankSelect`.
- **CN**: 这一段实现或声明了 `M68kPassConfig::addIRTranslator`, `M68kPassConfig::addLegalizeMachineIR`, `M68kPassConfig::addRegBankSelect` 等函数。

### Lines 163-175 / 第 163-175 行
```cpp
 163:   return false;
 164: }
 165: 
 166: bool M68kPassConfig::addGlobalInstructionSelect() {
 167:   addPass(new InstructionSelect());
 168:   return false;
 169: }
 170: 
 171: void M68kPassConfig::addPreSched2() { addPass(createM68kExpandPseudoPass()); }
 172: 
 173: void M68kPassConfig::addPreEmitPass() {
 174:   addPass(createM68kCollapseMOVEMPass());
 175: }
```
- **EN**: The range implements or declares functions including `M68kPassConfig::addGlobalInstructionSelect`, `M68kPassConfig::addPreSched2`, `M68kPassConfig::addPreEmitPass`.
- **CN**: 这一段实现或声明了 `M68kPassConfig::addGlobalInstructionSelect`, `M68kPassConfig::addPreSched2`, `M68kPassConfig::addPreEmitPass` 等函数。

## Key Concepts / 关键概念
- **GlobalISel**: Uses legalization, register banks, and instruction selection after generic IR lowering. / 在通用 IR 降低后使用合法化、寄存器银行和指令选择。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Assembly emission**: Prints target instructions and directives in textual assembly form. / 以文本汇编形式输出目标指令和伪指令。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。
- **Target machine**: Owns data layout, pass configuration, and backend-wide policy. / 管理数据布局、Pass 配置以及整个后端策略。

## Dependencies / 依赖关系
- `M68kTargetMachine.h`
- `M68k.h`
- `M68kMachineFunction.h`
- `M68kSubtarget.h`
- `M68kTargetObjectFile.h`
- `TargetInfo/M68kTargetInfo.h`
- `llvm/CodeGen/GlobalISel/IRTranslator.h`
- `llvm/CodeGen/GlobalISel/InstructionSelect.h`
- `llvm/CodeGen/GlobalISel/Legalizer.h`
- `llvm/CodeGen/GlobalISel/RegBankSelect.h`
- `llvm/CodeGen/Passes.h`
- `llvm/CodeGen/TargetPassConfig.h`
- `llvm/InitializePasses.h`
- `llvm/MC/TargetRegistry.h`
- `llvm/PassRegistry.h`
- `memory`
- `...` (1 more include dependencies omitted for brevity / 其余 1 个 include 依赖已省略)
