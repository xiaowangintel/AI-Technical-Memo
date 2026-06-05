# SystemZTargetMachine.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZTargetMachine.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file configures the target machine and code-generation pipeline for the SystemZ backend.
- **用途 (CN)**: 该文件用于 SystemZ 后端，负责配置目标机器与代码生成流水线。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===-- SystemZTargetMachine.cpp - Define TargetMachine for SystemZ -------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: #include "SystemZTargetMachine.h"
  10: #include "MCTargetDesc/SystemZMCTargetDesc.h"
  11: #include "SystemZ.h"
  12: #include "SystemZMachineFunctionInfo.h"
  13: #include "SystemZMachineScheduler.h"
  14: #include "SystemZTargetObjectFile.h"
  15: #include "SystemZTargetTransformInfo.h"
  16: #include "TargetInfo/SystemZTargetInfo.h"
  17: #include "llvm/ADT/StringRef.h"
  18: #include "llvm/Analysis/TargetTransformInfo.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `SystemZTargetMachine.h`, `SystemZMCTargetDesc.h`, `SystemZ.h`, `SystemZMachineFunctionInfo.h`, `SystemZMachineScheduler.h`, `SystemZTargetObjectFile.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `SystemZTargetMachine.h`, `SystemZMCTargetDesc.h`, `SystemZ.h`, `SystemZMachineFunctionInfo.h`, `SystemZMachineScheduler.h`, `SystemZTargetObjectFile.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 19-36 / 第 19-36 行
```cpp
  19: #include "llvm/CodeGen/Passes.h"
  20: #include "llvm/CodeGen/TargetLoweringObjectFileImpl.h"
  21: #include "llvm/CodeGen/TargetPassConfig.h"
  22: #include "llvm/IR/DataLayout.h"
  23: #include "llvm/MC/TargetRegistry.h"
  24: #include "llvm/Support/CodeGen.h"
  25: #include "llvm/Support/Compiler.h"
  26: #include "llvm/Target/TargetLoweringObjectFile.h"
  27: #include "llvm/Transforms/Scalar.h"
  28: #include <memory>
  29: #include <optional>
  30: #include <string>
  31: 
  32: using namespace llvm;
  33: 
  34: static cl::opt<bool> EnableMachineCombinerPass(
  35:     "systemz-machine-combiner",
  36:     cl::desc("Enable the machine combiner pass"),
```
- **EN**: It imports dependencies such as `Passes.h`, `TargetLoweringObjectFileImpl.h`, `TargetPassConfig.h`, `DataLayout.h`, `TargetRegistry.h`, `CodeGen.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 它引入了 `Passes.h`, `TargetLoweringObjectFileImpl.h`, `TargetPassConfig.h`, `DataLayout.h`, `TargetRegistry.h`, `CodeGen.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 37-54 / 第 37-54 行
```cpp
  37:     cl::init(true), cl::Hidden);
  38: 
  39: static cl::opt<bool> GenericSched(
  40:     "generic-sched", cl::Hidden, cl::init(false),
  41:     cl::desc("Run the generic pre-ra scheduler instead of the SystemZ "
  42:              "scheduler."));
  43: 
  44: // NOLINTNEXTLINE(readability-identifier-naming)
  45: extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
  46: LLVMInitializeSystemZTarget() {
  47:   // Register the target.
  48:   RegisterTargetMachine<SystemZTargetMachine> X(getTheSystemZTarget());
  49:   auto &PR = *PassRegistry::getPassRegistry();
  50:   initializeSystemZAsmPrinterPass(PR);
  51:   initializeSystemZElimComparePass(PR);
  52:   initializeSystemZShortenInstPass(PR);
  53:   initializeSystemZLongBranchPass(PR);
  54:   initializeSystemZLDCleanupPass(PR);
```
- **EN**: This span continues the file's main responsibility: this file configures the target machine and code-generation pipeline for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 55-72 / 第 55-72 行
```cpp
  55:   initializeSystemZShortenInstPass(PR);
  56:   initializeSystemZPostRewritePass(PR);
  57:   initializeSystemZTDCPassPass(PR);
  58:   initializeSystemZDAGToDAGISelLegacyPass(PR);
  59:   initializeSystemZCopyPhysRegsPass(PR);
  60: }
  61: 
  62: static std::unique_ptr<TargetLoweringObjectFile> createTLOF(const Triple &TT) {
  63:   if (TT.isOSzOS())
  64:     return std::make_unique<TargetLoweringObjectFileGOFF>();
  65: 
  66:   // Note: Some times run with -triple s390x-unknown.
  67:   // In this case, default to ELF unless z/OS specifically provided.
  68:   return std::make_unique<SystemZELFTargetObjectFile>();
  69: }
  70: 
  71: static Reloc::Model getEffectiveRelocModel(std::optional<Reloc::Model> RM) {
  72:   // Static code is suitable for use in a dynamic executable; there is no
```
- **EN**: The range implements or declares functions including `createTLOF`, `getEffectiveRelocModel`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `createTLOF`, `getEffectiveRelocModel` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 73-90 / 第 73-90 行
```cpp
  73:   // separate DynamicNoPIC model.
  74:   if (!RM || *RM == Reloc::DynamicNoPIC)
  75:     return Reloc::Static;
  76:   return *RM;
  77: }
  78: 
  79: // For SystemZ we define the models as follows:
  80: //
  81: // Small:  BRASL can call any function and will use a stub if necessary.
  82: //         Locally-binding symbols will always be in range of LARL.
  83: //
  84: // Medium: BRASL can call any function and will use a stub if necessary.
  85: //         GOT slots and locally-defined text will always be in range
  86: //         of LARL, but other symbols might not be.
  87: //
  88: // Large:  Equivalent to Medium for now.
  89: //
  90: // Kernel: Equivalent to Medium for now.
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 91-108 / 第 91-108 行
```cpp
  91: //
  92: // This means that any PIC module smaller than 4GB meets the
  93: // requirements of Small, so Small seems like the best default there.
  94: //
  95: // All symbols bind locally in a non-PIC module, so the choice is less
  96: // obvious.  There are two cases:
  97: //
  98: // - When creating an executable, PLTs and copy relocations allow
  99: //   us to treat external symbols as part of the executable.
 100: //   Any executable smaller than 4GB meets the requirements of Small,
 101: //   so that seems like the best default.
 102: //
 103: // - When creating JIT code, stubs will be in range of BRASL if the
 104: //   image is less than 4GB in size.  GOT entries will likewise be
 105: //   in range of LARL.  However, the JIT environment has no equivalent
 106: //   of copy relocs, so locally-binding data symbols might not be in
 107: //   the range of LARL.  We need the Medium model in that case.
 108: static CodeModel::Model
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 109-126 / 第 109-126 行
```cpp
 109: getEffectiveSystemZCodeModel(std::optional<CodeModel::Model> CM,
 110:                              Reloc::Model RM, bool JIT) {
 111:   if (CM) {
 112:     if (*CM == CodeModel::Tiny)
 113:       report_fatal_error("Target does not support the tiny CodeModel", false);
 114:     if (*CM == CodeModel::Kernel)
 115:       report_fatal_error("Target does not support the kernel CodeModel", false);
 116:     return *CM;
 117:   }
 118:   if (JIT)
 119:     return RM == Reloc::PIC_ ? CodeModel::Small : CodeModel::Medium;
 120:   return CodeModel::Small;
 121: }
 122: 
 123: SystemZTargetMachine::SystemZTargetMachine(const Target &T, const Triple &TT,
 124:                                            StringRef CPU, StringRef FS,
 125:                                            const TargetOptions &Options,
 126:                                            std::optional<Reloc::Model> RM,
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 127-144 / 第 127-144 行
```cpp
 127:                                            std::optional<CodeModel::Model> CM,
 128:                                            CodeGenOptLevel OL, bool JIT)
 129:     : CodeGenTargetMachineImpl(
 130:           T, TT.computeDataLayout(), TT, CPU, FS, Options,
 131:           getEffectiveRelocModel(RM),
 132:           getEffectiveSystemZCodeModel(CM, getEffectiveRelocModel(RM), JIT),
 133:           OL),
 134:       TLOF(createTLOF(getTargetTriple())) {
 135:   initAsmInfo();
 136: }
 137: 
 138: SystemZTargetMachine::~SystemZTargetMachine() = default;
 139: 
 140: const SystemZSubtarget *
 141: SystemZTargetMachine::getSubtargetImpl(const Function &F) const {
 142:   Attribute CPUAttr = F.getFnAttribute("target-cpu");
 143:   Attribute TuneAttr = F.getFnAttribute("tune-cpu");
 144:   Attribute FSAttr = F.getFnAttribute("target-features");
```
- **EN**: The range implements or declares functions including `CodeGenTargetMachineImpl`, `SystemZTargetMachine::getSubtargetImpl`.
- **CN**: 这一段实现或声明了 `CodeGenTargetMachineImpl`, `SystemZTargetMachine::getSubtargetImpl` 等函数。

### Lines 145-162 / 第 145-162 行
```cpp
 145: 
 146:   std::string CPU =
 147:       CPUAttr.isValid() ? CPUAttr.getValueAsString().str() : TargetCPU;
 148:   std::string TuneCPU =
 149:       TuneAttr.isValid() ? TuneAttr.getValueAsString().str() : CPU;
 150:   std::string FS =
 151:       FSAttr.isValid() ? FSAttr.getValueAsString().str() : TargetFS;
 152: 
 153:   // FIXME: This is related to the code below to reset the target options,
 154:   // we need to know whether the soft float and backchain flags are set on the
 155:   // function, so we can enable them as subtarget features.
 156:   bool SoftFloat = F.getFnAttribute("use-soft-float").getValueAsBool();
 157:   if (SoftFloat)
 158:     FS += FS.empty() ? "+soft-float" : ",+soft-float";
 159:   bool BackChain = F.hasFnAttribute("backchain");
 160:   if (BackChain)
 161:     FS += FS.empty() ? "+backchain" : ",+backchain";
 162: 
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 163-180 / 第 163-180 行
```cpp
 163:   auto &I = SubtargetMap[CPU + TuneCPU + FS];
 164:   if (!I) {
 165:     // This needs to be done before we create a new subtarget since any
 166:     // creation will depend on the TM and the code generation flags on the
 167:     // function that reside in TargetOptions.
 168:     resetTargetOptions(F);
 169:     I = std::make_unique<SystemZSubtarget>(TargetTriple, CPU, TuneCPU, FS,
 170:                                            *this);
 171:   }
 172: 
 173:   return I.get();
 174: }
 175: 
 176: ScheduleDAGInstrs *
 177: SystemZTargetMachine::createMachineScheduler(MachineSchedContext *C) const {
 178:   // Use GenericScheduler if requested on CL or for Z10 which has no sched
 179:   // model.
 180:   if (GenericSched ||
```
- **EN**: The range implements or declares functions including `SystemZTargetMachine::createMachineScheduler`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZTargetMachine::createMachineScheduler` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 181-198 / 第 181-198 行
```cpp
 181:       !C->MF->getSubtarget().getSchedModel().hasInstrSchedModel())
 182:     return nullptr;
 183: 
 184:   return createSchedLive<SystemZPreRASchedStrategy>(C);
 185: }
 186: 
 187: ScheduleDAGInstrs *
 188: SystemZTargetMachine::createPostMachineScheduler(MachineSchedContext *C) const {
 189:   return createSchedPostRA<SystemZPostRASchedStrategy>(C);
 190: }
 191: 
 192: namespace {
 193: 
 194: /// SystemZ Code Generator Pass Configuration Options.
 195: class SystemZPassConfig : public TargetPassConfig {
 196: public:
 197:   SystemZPassConfig(SystemZTargetMachine &TM, PassManagerBase &PM)
 198:     : TargetPassConfig(TM, PM) {}
```
- **EN**: This block declares or refines TableGen records such as `SystemZPassConfig`. The range implements or declares functions including `SystemZTargetMachine::createPostMachineScheduler`, `SystemZPassConfig`.
- **CN**: 该代码块声明或细化了 `SystemZPassConfig` 等 TableGen 记录。 这一段实现或声明了 `SystemZTargetMachine::createPostMachineScheduler`, `SystemZPassConfig` 等函数。

### Lines 199-216 / 第 199-216 行
```cpp
 199: 
 200:   SystemZTargetMachine &getSystemZTargetMachine() const {
 201:     return getTM<SystemZTargetMachine>();
 202:   }
 203: 
 204:   void addIRPasses() override;
 205:   bool addInstSelector() override;
 206:   bool addILPOpts() override;
 207:   void addPreRegAlloc() override;
 208:   void addPostRewrite() override;
 209:   void addPostRegAlloc() override;
 210:   void addPreSched2() override;
 211:   void addPreEmitPass() override;
 212: };
 213: 
 214: } // end anonymous namespace
 215: 
 216: void SystemZPassConfig::addIRPasses() {
```
- **EN**: The range implements or declares functions including `SystemZPassConfig::addIRPasses`.
- **CN**: 这一段实现或声明了 `SystemZPassConfig::addIRPasses` 等函数。

### Lines 217-234 / 第 217-234 行
```cpp
 217:   if (getOptLevel() != CodeGenOptLevel::None) {
 218:     addPass(createSystemZTDCPass());
 219:     addPass(createLoopDataPrefetchPass());
 220:   }
 221: 
 222:   addPass(createAtomicExpandLegacyPass());
 223: 
 224:   TargetPassConfig::addIRPasses();
 225: }
 226: 
 227: bool SystemZPassConfig::addInstSelector() {
 228:   addPass(createSystemZISelDag(getSystemZTargetMachine(), getOptLevel()));
 229: 
 230:   if (getOptLevel() != CodeGenOptLevel::None)
 231:     addPass(createSystemZLDCleanupPass(getSystemZTargetMachine()));
 232: 
 233:   return false;
 234: }
```
- **EN**: The range implements or declares functions including `SystemZPassConfig::addInstSelector`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZPassConfig::addInstSelector` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 235-252 / 第 235-252 行
```cpp
 235: 
 236: bool SystemZPassConfig::addILPOpts() {
 237:   addPass(&EarlyIfConverterLegacyID);
 238: 
 239:   if (EnableMachineCombinerPass)
 240:     addPass(&MachineCombinerID);
 241: 
 242:   return true;
 243: }
 244: 
 245: void SystemZPassConfig::addPreRegAlloc() {
 246:   addPass(createSystemZCopyPhysRegsPass(getSystemZTargetMachine()));
 247: }
 248: 
 249: void SystemZPassConfig::addPostRewrite() {
 250:   addPass(createSystemZPostRewritePass(getSystemZTargetMachine()));
 251: }
 252: 
```
- **EN**: The range implements or declares functions including `SystemZPassConfig::addILPOpts`, `SystemZPassConfig::addPreRegAlloc`, `SystemZPassConfig::addPostRewrite`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZPassConfig::addILPOpts`, `SystemZPassConfig::addPreRegAlloc`, `SystemZPassConfig::addPostRewrite` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 253-270 / 第 253-270 行
```cpp
 253: void SystemZPassConfig::addPostRegAlloc() {
 254:   // PostRewrite needs to be run at -O0 also (in which case addPostRewrite()
 255:   // is not called).
 256:   if (getOptLevel() == CodeGenOptLevel::None)
 257:     addPass(createSystemZPostRewritePass(getSystemZTargetMachine()));
 258: }
 259: 
 260: void SystemZPassConfig::addPreSched2() {
 261:   if (getOptLevel() != CodeGenOptLevel::None)
 262:     addPass(&IfConverterID);
 263: }
 264: 
 265: void SystemZPassConfig::addPreEmitPass() {
 266:   // Do instruction shortening before compare elimination because some
 267:   // vector instructions will be shortened into opcodes that compare
 268:   // elimination recognizes.
 269:   if (getOptLevel() != CodeGenOptLevel::None)
 270:     addPass(createSystemZShortenInstPass(getSystemZTargetMachine()));
```
- **EN**: The range implements or declares functions including `SystemZPassConfig::addPostRegAlloc`, `SystemZPassConfig::addPreSched2`, `SystemZPassConfig::addPreEmitPass`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZPassConfig::addPostRegAlloc`, `SystemZPassConfig::addPreSched2`, `SystemZPassConfig::addPreEmitPass` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 271-288 / 第 271-288 行
```cpp
 271: 
 272:   // We eliminate comparisons here rather than earlier because some
 273:   // transformations can change the set of available CC values and we
 274:   // generally want those transformations to have priority.  This is
 275:   // especially true in the commonest case where the result of the comparison
 276:   // is used by a single in-range branch instruction, since we will then
 277:   // be able to fuse the compare and the branch instead.
 278:   //
 279:   // For example, two-address NILF can sometimes be converted into
 280:   // three-address RISBLG.  NILF produces a CC value that indicates whether
 281:   // the low word is zero, but RISBLG does not modify CC at all.  On the
 282:   // other hand, 64-bit ANDs like NILL can sometimes be converted to RISBG.
 283:   // The CC value produced by NILL isn't useful for our purposes, but the
 284:   // value produced by RISBG can be used for any comparison with zero
 285:   // (not just equality).  So there are some transformations that lose
 286:   // CC values (while still being worthwhile) and others that happen to make
 287:   // the CC result more useful than it was originally.
 288:   //
```
- **EN**: This range is mainly descriptive commentary or banner text that frames the file before executable definitions begin.
- **CN**: 这一段主要是说明性注释或横幅文本，用于在可执行定义开始前说明文件背景。

### Lines 289-306 / 第 289-306 行
```cpp
 289:   // Another reason is that we only want to use BRANCH ON COUNT in cases
 290:   // where we know that the count register is not going to be spilled.
 291:   //
 292:   // Doing it so late makes it more likely that a register will be reused
 293:   // between the comparison and the branch, but it isn't clear whether
 294:   // preventing that would be a win or not.
 295:   if (getOptLevel() != CodeGenOptLevel::None)
 296:     addPass(createSystemZElimComparePass(getSystemZTargetMachine()));
 297:   addPass(createSystemZLongBranchPass(getSystemZTargetMachine()));
 298: 
 299:   // Do final scheduling after all other optimizations, to get an
 300:   // optimal input for the decoder (branch relaxation must happen
 301:   // after block placement).
 302:   if (getOptLevel() != CodeGenOptLevel::None)
 303:     addPass(&PostMachineSchedulerID);
 304: }
 305: 
 306: TargetPassConfig *SystemZTargetMachine::createPassConfig(PassManagerBase &PM) {
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 307-320 / 第 307-320 行
```cpp
 307:   return new SystemZPassConfig(*this, PM);
 308: }
 309: 
 310: TargetTransformInfo
 311: SystemZTargetMachine::getTargetTransformInfo(const Function &F) const {
 312:   return TargetTransformInfo(std::make_unique<SystemZTTIImpl>(this, F));
 313: }
 314: 
 315: MachineFunctionInfo *SystemZTargetMachine::createMachineFunctionInfo(
 316:     BumpPtrAllocator &Allocator, const Function &F,
 317:     const TargetSubtargetInfo *STI) const {
 318:   return SystemZMachineFunctionInfo::create<SystemZMachineFunctionInfo>(
 319:       Allocator, F, STI);
 320: }
```
- **EN**: The range implements or declares functions including `SystemZTargetMachine::getTargetTransformInfo`.
- **CN**: 这一段实现或声明了 `SystemZTargetMachine::getTargetTransformInfo` 等函数。

## Key Concepts / 关键概念
- **GlobalISel**: Uses legalization, register banks, and instruction selection after generic IR lowering. / 在通用 IR 降低后使用合法化、寄存器银行和指令选择。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Assembly emission**: Prints target instructions and directives in textual assembly form. / 以文本汇编形式输出目标指令和伪指令。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。
- **Scheduling model**: Provides instruction itineraries or scheduling classes for performance modelling. / 提供指令行程或调度类以支持性能建模。

## Dependencies / 依赖关系
- `SystemZTargetMachine.h`
- `MCTargetDesc/SystemZMCTargetDesc.h`
- `SystemZ.h`
- `SystemZMachineFunctionInfo.h`
- `SystemZMachineScheduler.h`
- `SystemZTargetObjectFile.h`
- `SystemZTargetTransformInfo.h`
- `TargetInfo/SystemZTargetInfo.h`
- `llvm/ADT/StringRef.h`
- `llvm/Analysis/TargetTransformInfo.h`
- `llvm/CodeGen/Passes.h`
- `llvm/CodeGen/TargetLoweringObjectFileImpl.h`
- `llvm/CodeGen/TargetPassConfig.h`
- `llvm/IR/DataLayout.h`
- `llvm/MC/TargetRegistry.h`
- `llvm/Support/CodeGen.h`
- `...` (6 more include dependencies omitted for brevity / 其余 6 个 include 依赖已省略)
