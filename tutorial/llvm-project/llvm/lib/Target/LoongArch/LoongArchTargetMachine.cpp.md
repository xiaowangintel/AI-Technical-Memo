# LoongArchTargetMachine.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/LoongArchTargetMachine.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file configures the target machine and code-generation pipeline for the LoongArch backend.
- **用途 (CN)**: 该文件用于 LoongArch 后端，负责配置目标机器与代码生成流水线。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===-- LoongArchTargetMachine.cpp - Define TargetMachine for LoongArch ---===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // Implements the info about LoongArch target spec.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "LoongArchTargetMachine.h"
  14: #include "LoongArch.h"
  15: #include "LoongArchMachineFunctionInfo.h"
  16: #include "LoongArchTargetTransformInfo.h"
  17: #include "MCTargetDesc/LoongArchBaseInfo.h"
  18: #include "TargetInfo/LoongArchTargetInfo.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `LoongArchTargetMachine.h`, `LoongArch.h`, `LoongArchMachineFunctionInfo.h`, `LoongArchTargetTransformInfo.h`, `LoongArchBaseInfo.h`, `LoongArchTargetInfo.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `LoongArchTargetMachine.h`, `LoongArch.h`, `LoongArchMachineFunctionInfo.h`, `LoongArchTargetTransformInfo.h`, `LoongArchBaseInfo.h`, `LoongArchTargetInfo.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 19-36 / 第 19-36 行
```cpp
  19: #include "llvm/Analysis/TargetTransformInfo.h"
  20: #include "llvm/CodeGen/Passes.h"
  21: #include "llvm/CodeGen/TargetLoweringObjectFileImpl.h"
  22: #include "llvm/CodeGen/TargetPassConfig.h"
  23: #include "llvm/MC/TargetRegistry.h"
  24: #include "llvm/Support/CodeGen.h"
  25: #include "llvm/Support/Compiler.h"
  26: #include "llvm/Transforms/Scalar.h"
  27: #include <optional>
  28: 
  29: using namespace llvm;
  30: 
  31: #define DEBUG_TYPE "loongarch"
  32: 
  33: extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
  34: LLVMInitializeLoongArchTarget() {
  35:   // Register the target.
  36:   RegisterTargetMachine<LoongArchTargetMachine> X(getTheLoongArch32Target());
```
- **EN**: It imports dependencies such as `TargetTransformInfo.h`, `Passes.h`, `TargetLoweringObjectFileImpl.h`, `TargetPassConfig.h`, `TargetRegistry.h`, `CodeGen.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 它引入了 `TargetTransformInfo.h`, `Passes.h`, `TargetLoweringObjectFileImpl.h`, `TargetPassConfig.h`, `TargetRegistry.h`, `CodeGen.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 37-54 / 第 37-54 行
```cpp
  37:   RegisterTargetMachine<LoongArchTargetMachine> Y(getTheLoongArch64Target());
  38:   auto *PR = PassRegistry::getPassRegistry();
  39:   initializeLoongArchDeadRegisterDefinitionsPass(*PR);
  40:   initializeLoongArchMergeBaseOffsetOptPass(*PR);
  41:   initializeLoongArchOptWInstrsPass(*PR);
  42:   initializeLoongArchPreRAExpandPseudoPass(*PR);
  43:   initializeLoongArchExpandPseudoPass(*PR);
  44:   initializeLoongArchDAGToDAGISelLegacyPass(*PR);
  45:   initializeLoongArchExpandAtomicPseudoPass(*PR);
  46: }
  47: 
  48: static cl::opt<bool> EnableLoongArchDeadRegisterElimination(
  49:     "loongarch-enable-dead-defs", cl::Hidden,
  50:     cl::desc("Enable the pass that removes dead"
  51:              " definitons and replaces stores to"
  52:              " them with stores to r0"),
  53:     cl::init(true));
  54: 
```
- **EN**: This span continues the file's main responsibility: this file configures the target machine and code-generation pipeline for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 55-72 / 第 55-72 行
```cpp
  55: static cl::opt<bool>
  56:     EnableLoopDataPrefetch("loongarch-enable-loop-data-prefetch", cl::Hidden,
  57:                            cl::desc("Enable the loop data prefetch pass"),
  58:                            cl::init(false));
  59: 
  60: static cl::opt<bool>
  61:     EnableMergeBaseOffset("loongarch-enable-merge-offset",
  62:                           cl::desc("Enable the merge base offset pass"),
  63:                           cl::init(true), cl::Hidden);
  64: 
  65: static cl::opt<bool>
  66:     EnableSinkFold("loongarch-enable-sink-fold",
  67:                    cl::desc("Enable sinking and folding of instruction copies"),
  68:                    cl::init(true), cl::Hidden);
  69: 
  70: static Reloc::Model getEffectiveRelocModel(std::optional<Reloc::Model> RM) {
  71:   return RM.value_or(Reloc::Static);
  72: }
```
- **EN**: The range implements or declares functions including `getEffectiveRelocModel`.
- **CN**: 这一段实现或声明了 `getEffectiveRelocModel` 等函数。

### Lines 73-90 / 第 73-90 行
```cpp
  73: 
  74: static CodeModel::Model
  75: getEffectiveLoongArchCodeModel(const Triple &TT,
  76:                                std::optional<CodeModel::Model> CM) {
  77:   if (!CM)
  78:     return TT.isArch64Bit() ? CodeModel::Medium : CodeModel::Small;
  79: 
  80:   switch (*CM) {
  81:   case CodeModel::Small:
  82:   case CodeModel::Medium:
  83:     return *CM;
  84:   case CodeModel::Large:
  85:     if (!TT.isArch64Bit())
  86:       report_fatal_error("Large code model requires LA64");
  87:     return *CM;
  88:   default:
  89:     report_fatal_error(
  90:         "Only small, medium and large code models are allowed on LoongArch");
```
- **EN**: The range implements or declares functions including `getEffectiveLoongArchCodeModel`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `getEffectiveLoongArchCodeModel` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 91-108 / 第 91-108 行
```cpp
  91:   }
  92: }
  93: 
  94: LoongArchTargetMachine::LoongArchTargetMachine(
  95:     const Target &T, const Triple &TT, StringRef CPU, StringRef FS,
  96:     const TargetOptions &Options, std::optional<Reloc::Model> RM,
  97:     std::optional<CodeModel::Model> CM, CodeGenOptLevel OL, bool JIT)
  98:     : CodeGenTargetMachineImpl(T, TT.computeDataLayout(), TT, CPU, FS, Options,
  99:                                getEffectiveRelocModel(RM),
 100:                                getEffectiveLoongArchCodeModel(TT, CM), OL),
 101:       TLOF(std::make_unique<TargetLoweringObjectFileELF>()) {
 102:   initAsmInfo();
 103: }
 104: 
 105: LoongArchTargetMachine::~LoongArchTargetMachine() = default;
 106: 
 107: const LoongArchSubtarget *
 108: LoongArchTargetMachine::getSubtargetImpl(const Function &F) const {
```
- **EN**: The range implements or declares functions including `CodeGenTargetMachineImpl`, `LoongArchTargetMachine::getSubtargetImpl`.
- **CN**: 这一段实现或声明了 `CodeGenTargetMachineImpl`, `LoongArchTargetMachine::getSubtargetImpl` 等函数。

### Lines 109-126 / 第 109-126 行
```cpp
 109:   Attribute CPUAttr = F.getFnAttribute("target-cpu");
 110:   Attribute TuneAttr = F.getFnAttribute("tune-cpu");
 111:   Attribute FSAttr = F.getFnAttribute("target-features");
 112: 
 113:   std::string CPU =
 114:       CPUAttr.isValid() ? CPUAttr.getValueAsString().str() : TargetCPU;
 115:   std::string TuneCPU =
 116:       TuneAttr.isValid() ? TuneAttr.getValueAsString().str() : CPU;
 117:   std::string FS =
 118:       FSAttr.isValid() ? FSAttr.getValueAsString().str() : TargetFS;
 119: 
 120:   std::string Key = CPU + TuneCPU + FS;
 121:   auto &I = SubtargetMap[Key];
 122:   if (!I) {
 123:     // This needs to be done before we create a new subtarget since any
 124:     // creation will depend on the TM and the code generation flags on the
 125:     // function that reside in TargetOptions.
 126:     resetTargetOptions(F);
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 127-144 / 第 127-144 行
```cpp
 127:     auto ABIName = Options.MCOptions.getABIName();
 128:     if (const MDString *ModuleTargetABI = dyn_cast_or_null<MDString>(
 129:             F.getParent()->getModuleFlag("target-abi"))) {
 130:       auto TargetABI = LoongArchABI::getTargetABI(ABIName);
 131:       if (TargetABI != LoongArchABI::ABI_Unknown &&
 132:           ModuleTargetABI->getString() != ABIName) {
 133:         report_fatal_error("-target-abi option != target-abi module flag");
 134:       }
 135:       ABIName = ModuleTargetABI->getString();
 136:     }
 137:     I = std::make_unique<LoongArchSubtarget>(TargetTriple, CPU, TuneCPU, FS,
 138:                                              ABIName, *this);
 139:   }
 140:   return I.get();
 141: }
 142: 
 143: MachineFunctionInfo *LoongArchTargetMachine::createMachineFunctionInfo(
 144:     BumpPtrAllocator &Allocator, const Function &F,
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 145-162 / 第 145-162 行
```cpp
 145:     const TargetSubtargetInfo *STI) const {
 146:   return LoongArchMachineFunctionInfo::create<LoongArchMachineFunctionInfo>(
 147:       Allocator, F, STI);
 148: }
 149: 
 150: namespace {
 151: class LoongArchPassConfig : public TargetPassConfig {
 152: public:
 153:   LoongArchPassConfig(LoongArchTargetMachine &TM, PassManagerBase &PM)
 154:       : TargetPassConfig(TM, PM) {
 155:     setEnableSinkAndFold(EnableSinkFold);
 156:   }
 157: 
 158:   LoongArchTargetMachine &getLoongArchTargetMachine() const {
 159:     return getTM<LoongArchTargetMachine>();
 160:   }
 161: 
 162:   void addIRPasses() override;
```
- **EN**: This block declares or refines TableGen records such as `LoongArchPassConfig`. The range implements or declares functions including `LoongArchPassConfig`.
- **CN**: 该代码块声明或细化了 `LoongArchPassConfig` 等 TableGen 记录。 这一段实现或声明了 `LoongArchPassConfig` 等函数。

### Lines 163-180 / 第 163-180 行
```cpp
 163:   void addCodeGenPrepare() override;
 164:   bool addInstSelector() override;
 165:   void addPreEmitPass() override;
 166:   void addPreEmitPass2() override;
 167:   void addMachineSSAOptimization() override;
 168:   void addPreRegAlloc() override;
 169:   bool addRegAssignAndRewriteFast() override;
 170:   bool addRegAssignAndRewriteOptimized() override;
 171: };
 172: } // end namespace
 173: 
 174: TargetPassConfig *
 175: LoongArchTargetMachine::createPassConfig(PassManagerBase &PM) {
 176:   return new LoongArchPassConfig(*this, PM);
 177: }
 178: 
 179: void LoongArchPassConfig::addIRPasses() {
 180:   // Run LoopDataPrefetch
```
- **EN**: The range implements or declares functions including `LoongArchTargetMachine::createPassConfig`, `LoongArchPassConfig::addIRPasses`.
- **CN**: 这一段实现或声明了 `LoongArchTargetMachine::createPassConfig`, `LoongArchPassConfig::addIRPasses` 等函数。

### Lines 181-198 / 第 181-198 行
```cpp
 181:   //
 182:   // Run this before LSR to remove the multiplies involved in computing the
 183:   // pointer values N iterations ahead.
 184:   if (TM->getOptLevel() != CodeGenOptLevel::None && EnableLoopDataPrefetch)
 185:     addPass(createLoopDataPrefetchPass());
 186:   addPass(createAtomicExpandLegacyPass());
 187: 
 188:   TargetPassConfig::addIRPasses();
 189: }
 190: 
 191: void LoongArchPassConfig::addCodeGenPrepare() {
 192:   if (getOptLevel() != CodeGenOptLevel::None)
 193:     addPass(createTypePromotionLegacyPass());
 194:   TargetPassConfig::addCodeGenPrepare();
 195: }
 196: 
 197: bool LoongArchPassConfig::addInstSelector() {
 198:   addPass(createLoongArchISelDag(getLoongArchTargetMachine(), getOptLevel()));
```
- **EN**: The range implements or declares functions including `LoongArchPassConfig::addCodeGenPrepare`, `LoongArchPassConfig::addInstSelector`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchPassConfig::addCodeGenPrepare`, `LoongArchPassConfig::addInstSelector` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 199-216 / 第 199-216 行
```cpp
 199: 
 200:   return false;
 201: }
 202: 
 203: TargetTransformInfo
 204: LoongArchTargetMachine::getTargetTransformInfo(const Function &F) const {
 205:   return TargetTransformInfo(std::make_unique<LoongArchTTIImpl>(this, F));
 206: }
 207: 
 208: void LoongArchPassConfig::addPreEmitPass() { addPass(&BranchRelaxationPassID); }
 209: 
 210: void LoongArchPassConfig::addPreEmitPass2() {
 211:   addPass(createLoongArchExpandPseudoPass());
 212:   // Schedule the expansion of AtomicPseudos at the last possible moment,
 213:   // avoiding the possibility for other passes to break the requirements for
 214:   // forward progress in the LL/SC block.
 215:   addPass(createLoongArchExpandAtomicPseudoPass());
 216: }
```
- **EN**: The range implements or declares functions including `LoongArchTargetMachine::getTargetTransformInfo`, `LoongArchPassConfig::addPreEmitPass`, `LoongArchPassConfig::addPreEmitPass2`.
- **CN**: 这一段实现或声明了 `LoongArchTargetMachine::getTargetTransformInfo`, `LoongArchPassConfig::addPreEmitPass`, `LoongArchPassConfig::addPreEmitPass2` 等函数。

### Lines 217-234 / 第 217-234 行
```cpp
 217: 
 218: void LoongArchPassConfig::addMachineSSAOptimization() {
 219:   TargetPassConfig::addMachineSSAOptimization();
 220: 
 221:   if (TM->getTargetTriple().isLoongArch64()) {
 222:     addPass(createLoongArchOptWInstrsPass());
 223:   }
 224: }
 225: 
 226: void LoongArchPassConfig::addPreRegAlloc() {
 227:   addPass(createLoongArchPreRAExpandPseudoPass());
 228:   if (TM->getOptLevel() != CodeGenOptLevel::None && EnableMergeBaseOffset)
 229:     addPass(createLoongArchMergeBaseOffsetOptPass());
 230: }
 231: 
 232: bool LoongArchPassConfig::addRegAssignAndRewriteFast() {
 233:   if (TM->getOptLevel() != CodeGenOptLevel::None &&
 234:       EnableLoongArchDeadRegisterElimination)
```
- **EN**: The range implements or declares functions including `LoongArchPassConfig::addMachineSSAOptimization`, `LoongArchPassConfig::addPreRegAlloc`, `LoongArchPassConfig::addRegAssignAndRewriteFast`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchPassConfig::addMachineSSAOptimization`, `LoongArchPassConfig::addPreRegAlloc`, `LoongArchPassConfig::addRegAssignAndRewriteFast` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 235-244 / 第 235-244 行
```cpp
 235:     addPass(createLoongArchDeadRegisterDefinitionsPass());
 236:   return TargetPassConfig::addRegAssignAndRewriteFast();
 237: }
 238: 
 239: bool LoongArchPassConfig::addRegAssignAndRewriteOptimized() {
 240:   if (TM->getOptLevel() != CodeGenOptLevel::None &&
 241:       EnableLoongArchDeadRegisterElimination)
 242:     addPass(createLoongArchDeadRegisterDefinitionsPass());
 243:   return TargetPassConfig::addRegAssignAndRewriteOptimized();
 244: }
```
- **EN**: The range implements or declares functions including `LoongArchPassConfig::addRegAssignAndRewriteOptimized`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchPassConfig::addRegAssignAndRewriteOptimized` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

## Key Concepts / 关键概念
- **GlobalISel**: Uses legalization, register banks, and instruction selection after generic IR lowering. / 在通用 IR 降低后使用合法化、寄存器银行和指令选择。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。
- **Scheduling model**: Provides instruction itineraries or scheduling classes for performance modelling. / 提供指令行程或调度类以支持性能建模。
- **Target machine**: Owns data layout, pass configuration, and backend-wide policy. / 管理数据布局、Pass 配置以及整个后端策略。

## Dependencies / 依赖关系
- `LoongArchTargetMachine.h`
- `LoongArch.h`
- `LoongArchMachineFunctionInfo.h`
- `LoongArchTargetTransformInfo.h`
- `MCTargetDesc/LoongArchBaseInfo.h`
- `TargetInfo/LoongArchTargetInfo.h`
- `llvm/Analysis/TargetTransformInfo.h`
- `llvm/CodeGen/Passes.h`
- `llvm/CodeGen/TargetLoweringObjectFileImpl.h`
- `llvm/CodeGen/TargetPassConfig.h`
- `llvm/MC/TargetRegistry.h`
- `llvm/Support/CodeGen.h`
- `llvm/Support/Compiler.h`
- `llvm/Transforms/Scalar.h`
- `optional`
