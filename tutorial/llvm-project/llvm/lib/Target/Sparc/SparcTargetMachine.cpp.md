# SparcTargetMachine.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Sparc/SparcTargetMachine.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Defines the target machine, data layout, pass pipeline hooks, and subtarget creation logic.
  - **CN**: 定义目标机器、数据布局、Pass 流水线钩子以及子目标创建逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- SparcTargetMachine.cpp - Define TargetMachine for Sparc -----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-21
```cpp
//
//
//===----------------------------------------------------------------------===//

#include "SparcTargetMachine.h"
#include "LeonPasses.h"
#include "Sparc.h"
#include "SparcMachineFunctionInfo.h"
#include "SparcTargetObjectFile.h"
#include "SparcTargetTransformInfo.h"
#include "TargetInfo/SparcTargetInfo.h"
#include "llvm/CodeGen/Passes.h"
#include "llvm/CodeGen/TargetPassConfig.h"
#include "llvm/MC/TargetRegistry.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `SparcTargetMachine.h`, `LeonPasses.h`, `Sparc.h`, `SparcMachineFunctionInfo.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `SparcTargetMachine.h`, `LeonPasses.h`, `Sparc.h`, `SparcMachineFunctionInfo.h`。

### Lines 22-31
```cpp
#include "llvm/Support/Compiler.h"
#include <optional>
using namespace llvm;

extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void LLVMInitializeSparcTarget() {
  // Register the target.
  RegisterTargetMachine<SparcV8TargetMachine> X(getTheSparcTarget());
  RegisterTargetMachine<SparcV9TargetMachine> Y(getTheSparcV9Target());
  RegisterTargetMachine<SparcelTargetMachine> Z(getTheSparcelTarget());

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/Support/Compiler.h`, `optional`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/Support/Compiler.h`, `optional`。

### Lines 32-41
```cpp
  PassRegistry &PR = *PassRegistry::getPassRegistry();
  initializeSparcAsmPrinterPass(PR);
  initializeSparcDAGToDAGISelLegacyPass(PR);
  initializeErrataWorkaroundPass(PR);
}

static cl::opt<bool>
    BranchRelaxation("sparc-enable-branch-relax", cl::Hidden, cl::init(true),
                     cl::desc("Relax out of range conditional branches"));

```
- **EN**: Implements logic around `getPassRegistry`, `initializeSparcAsmPrinterPass`, `initializeSparcDAGToDAGISelLegacyPass`, `initializeErrataWorkaroundPass`, ...; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `getPassRegistry`, `initializeSparcAsmPrinterPass`, `initializeSparcDAGToDAGISelLegacyPass`, `initializeErrataWorkaroundPass`, ... 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 42-55
```cpp
static Reloc::Model getEffectiveRelocModel(std::optional<Reloc::Model> RM) {
  return RM.value_or(Reloc::Static);
}

// Code models. Some only make sense for 64-bit code.
//
// SunCC  Reloc   CodeModel  Constraints
// abs32  Static  Small      text+data+bss linked below 2^32 bytes
// abs44  Static  Medium     text+data+bss linked below 2^44 bytes
// abs64  Static  Large      text smaller than 2^31 bytes
// pic13  PIC_    Small      GOT < 2^13 bytes
// pic32  PIC_    Medium     GOT < 2^32 bytes
//
// All code models require that the text segment is smaller than 2GB.
```
- **EN**: Implements logic around `getEffectiveRelocModel`, `value_or`; this block returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `getEffectiveRelocModel`, `value_or` 实现具体逻辑；这一段返回目标相关结果，映射 fixup 或重定位。

### Lines 56-69
```cpp
static CodeModel::Model
getEffectiveSparcCodeModel(std::optional<CodeModel::Model> CM, Reloc::Model RM,
                           bool Is64Bit, bool JIT) {
  if (CM) {
    if (*CM == CodeModel::Tiny)
      report_fatal_error("Target does not support the tiny CodeModel", false);
    if (*CM == CodeModel::Kernel)
      report_fatal_error("Target does not support the kernel CodeModel", false);
    return *CM;
  }
  if (Is64Bit) {
    if (JIT)
      return CodeModel::Large;
    return RM == Reloc::PIC_ ? CodeModel::Small : CodeModel::Medium;
```
- **EN**: Implements logic around `getEffectiveSparcCodeModel`, `report_fatal_error`; this block applies conditional target rules; returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `getEffectiveSparcCodeModel`, `report_fatal_error` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位。

### Lines 70-83
```cpp
  }
  return CodeModel::Small;
}

/// Create an ILP32 architecture model
SparcTargetMachine::SparcTargetMachine(const Target &T, const Triple &TT,
                                       StringRef CPU, StringRef FS,
                                       const TargetOptions &Options,
                                       std::optional<Reloc::Model> RM,
                                       std::optional<CodeModel::Model> CM,
                                       CodeGenOptLevel OL, bool JIT)
    : CodeGenTargetMachineImpl(
          T, TT.computeDataLayout(), TT, CPU, FS, Options,
          getEffectiveRelocModel(RM),
```
- **EN**: Implements logic around `SparcTargetMachine`, `CodeGenTargetMachineImpl`, `computeDataLayout`, `getEffectiveRelocModel`; this block returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `SparcTargetMachine`, `CodeGenTargetMachineImpl`, `computeDataLayout`, `getEffectiveRelocModel` 实现具体逻辑；这一段返回目标相关结果，映射 fixup 或重定位。

### Lines 84-90
```cpp
          getEffectiveSparcCodeModel(CM, getEffectiveRelocModel(RM),
                                     TT.isSPARC64(), JIT),
          OL),
      TLOF(std::make_unique<SparcELFTargetObjectFile>()) {
  initAsmInfo();
}

```
- **EN**: Implements logic around `getEffectiveSparcCodeModel`, `isSPARC64`, `TLOF`, `initAsmInfo`; this block maps fixups or relocations.
- **CN**: 围绕 `getEffectiveSparcCodeModel`, `isSPARC64`, `TLOF`, `initAsmInfo` 实现具体逻辑；这一段映射 fixup 或重定位。

### Lines 91-98
```cpp
SparcTargetMachine::~SparcTargetMachine() = default;

const SparcSubtarget *
SparcTargetMachine::getSubtargetImpl(const Function &F) const {
  Attribute CPUAttr = F.getFnAttribute("target-cpu");
  Attribute TuneAttr = F.getFnAttribute("tune-cpu");
  Attribute FSAttr = F.getFnAttribute("target-features");

```
- **EN**: Implements logic around `~SparcTargetMachine`, `getSubtargetImpl`, `getFnAttribute`.
- **CN**: 围绕 `~SparcTargetMachine`, `getSubtargetImpl`, `getFnAttribute` 实现具体逻辑。

### Lines 99-105
```cpp
  std::string CPU =
      CPUAttr.isValid() ? CPUAttr.getValueAsString().str() : TargetCPU;
  std::string TuneCPU =
      TuneAttr.isValid() ? TuneAttr.getValueAsString().str() : CPU;
  std::string FS =
      FSAttr.isValid() ? FSAttr.getValueAsString().str() : TargetFS;

```
- **EN**: Implements logic around `isValid`.
- **CN**: 围绕 `isValid` 实现具体逻辑。

### Lines 106-113
```cpp
  // FIXME: This is related to the code below to reset the target options,
  // we need to know whether or not the soft float flag is set on the
  // function, so we can enable it as a subtarget feature.
  bool softFloat = F.getFnAttribute("use-soft-float").getValueAsBool();

  if (softFloat)
    FS += FS.empty() ? "+soft-float" : ",+soft-float";

```
- **EN**: Implements logic around `getFnAttribute`, `empty`; this block applies conditional target rules.
- **CN**: 围绕 `getFnAttribute`, `empty` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 114-124
```cpp
  auto &I = SubtargetMap[CPU + FS];
  if (!I) {
    // This needs to be done before we create a new subtarget since any
    // creation will depend on the TM and the code generation flags on the
    // function that reside in TargetOptions.
    resetTargetOptions(F);
    I = std::make_unique<SparcSubtarget>(CPU, TuneCPU, FS, *this);
  }
  return I.get();
}

```
- **EN**: Implements logic around `resetTargetOptions`, `make_unique<SparcSubtarget>`, `get`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `resetTargetOptions`, `make_unique<SparcSubtarget>`, `get` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 125-131
```cpp
MachineFunctionInfo *SparcTargetMachine::createMachineFunctionInfo(
    BumpPtrAllocator &Allocator, const Function &F,
    const TargetSubtargetInfo *STI) const {
  return SparcMachineFunctionInfo::create<SparcMachineFunctionInfo>(Allocator,
                                                                    F, STI);
}

```
- **EN**: Implements logic around `createMachineFunctionInfo`, `create<SparcMachineFunctionInfo>`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `createMachineFunctionInfo`, `create<SparcMachineFunctionInfo>` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 132-138
```cpp
namespace {
/// Sparc Code Generator Pass Configuration Options.
class SparcPassConfig : public TargetPassConfig {
public:
  SparcPassConfig(SparcTargetMachine &TM, PassManagerBase &PM)
    : TargetPassConfig(TM, PM) {}

```
- **EN**: Introduces declarations for `SparcPassConfig`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `SparcPassConfig` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 139-148
```cpp
  SparcTargetMachine &getSparcTargetMachine() const {
    return getTM<SparcTargetMachine>();
  }

  void addIRPasses() override;
  bool addInstSelector() override;
  void addPreEmitPass() override;
};
} // namespace

```
- **EN**: Implements logic around `getSparcTargetMachine`, `getTM<SparcTargetMachine>`, `addIRPasses`, `addInstSelector`, ...; this block returns target-specific results.
- **CN**: 围绕 `getSparcTargetMachine`, `getTM<SparcTargetMachine>`, `addIRPasses`, `addInstSelector`, ... 实现具体逻辑；这一段返回目标相关结果。

### Lines 149-157
```cpp
TargetPassConfig *SparcTargetMachine::createPassConfig(PassManagerBase &PM) {
  return new SparcPassConfig(*this, PM);
}

TargetTransformInfo
SparcTargetMachine::getTargetTransformInfo(const Function &F) const {
  return TargetTransformInfo(std::make_unique<SparcTTIImpl>(this, F));
}

```
- **EN**: Implements logic around `createPassConfig`, `SparcPassConfig`, `getTargetTransformInfo`, `TargetTransformInfo`; this block returns target-specific results.
- **CN**: 围绕 `createPassConfig`, `SparcPassConfig`, `getTargetTransformInfo`, `TargetTransformInfo` 实现具体逻辑；这一段返回目标相关结果。

### Lines 158-168
```cpp
void SparcPassConfig::addIRPasses() {
  addPass(createAtomicExpandLegacyPass());

  TargetPassConfig::addIRPasses();
}

bool SparcPassConfig::addInstSelector() {
  addPass(createSparcISelDag(getSparcTargetMachine()));
  return false;
}

```
- **EN**: Implements logic around `addIRPasses`, `addPass`, `addInstSelector`; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `addIRPasses`, `addPass`, `addInstSelector` 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 169-179
```cpp
void SparcPassConfig::addPreEmitPass(){
  if (BranchRelaxation)
    addPass(&BranchRelaxationPassID);

  addPass(createSparcDelaySlotFillerPass());
  addPass(new InsertNOPLoad());
  addPass(new DetectRoundChange());
  addPass(new FixAllFDIVSQRT());
  addPass(new ErrataWorkaround());
}

```
- **EN**: Implements logic around `addPreEmitPass`, `addPass`; this block applies conditional target rules.
- **CN**: 围绕 `addPreEmitPass`, `addPass` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 180-189
```cpp
void SparcV8TargetMachine::anchor() { }

SparcV8TargetMachine::SparcV8TargetMachine(const Target &T, const Triple &TT,
                                           StringRef CPU, StringRef FS,
                                           const TargetOptions &Options,
                                           std::optional<Reloc::Model> RM,
                                           std::optional<CodeModel::Model> CM,
                                           CodeGenOptLevel OL, bool JIT)
    : SparcTargetMachine(T, TT, CPU, FS, Options, RM, CM, OL, JIT) {}

```
- **EN**: Implements logic around `anchor`, `SparcV8TargetMachine`, `SparcTargetMachine`; this block maps fixups or relocations.
- **CN**: 围绕 `anchor`, `SparcV8TargetMachine`, `SparcTargetMachine` 实现具体逻辑；这一段映射 fixup 或重定位。

### Lines 190-199
```cpp
void SparcV9TargetMachine::anchor() { }

SparcV9TargetMachine::SparcV9TargetMachine(const Target &T, const Triple &TT,
                                           StringRef CPU, StringRef FS,
                                           const TargetOptions &Options,
                                           std::optional<Reloc::Model> RM,
                                           std::optional<CodeModel::Model> CM,
                                           CodeGenOptLevel OL, bool JIT)
    : SparcTargetMachine(T, TT, CPU, FS, Options, RM, CM, OL, JIT) {}

```
- **EN**: Implements logic around `anchor`, `SparcV9TargetMachine`, `SparcTargetMachine`; this block maps fixups or relocations.
- **CN**: 围绕 `anchor`, `SparcV9TargetMachine`, `SparcTargetMachine` 实现具体逻辑；这一段映射 fixup 或重定位。

### Lines 200-208
```cpp
void SparcelTargetMachine::anchor() {}

SparcelTargetMachine::SparcelTargetMachine(const Target &T, const Triple &TT,
                                           StringRef CPU, StringRef FS,
                                           const TargetOptions &Options,
                                           std::optional<Reloc::Model> RM,
                                           std::optional<CodeModel::Model> CM,
                                           CodeGenOptLevel OL, bool JIT)
    : SparcTargetMachine(T, TT, CPU, FS, Options, RM, CM, OL, JIT) {}
```
- **EN**: Implements logic around `anchor`, `SparcelTargetMachine`, `SparcTargetMachine`; this block maps fixups or relocations.
- **CN**: 围绕 `anchor`, `SparcelTargetMachine`, `SparcTargetMachine` 实现具体逻辑；这一段映射 fixup 或重定位。

## Key Concepts / 关键概念

- **Target configuration / 目标配置**:
  - **EN**: Owns data layout, passes, and per-function subtarget selection
  - **CN**: 管理数据布局、Pass 与按函数的子目标选择
- **Fixups and relocations / Fixup 与重定位**:
  - **EN**: Bridges symbolic references to concrete relocation records
  - **CN**: 把符号引用连接到具体重定位记录
- **Machine-level codegen / 机器级代码生成**:
  - **EN**: Operates after instruction selection on machine instructions and blocks
  - **CN**: 在指令选择后处理机器指令与基本块

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `SparcTargetMachine.h`, `LeonPasses.h`, `Sparc.h`, `SparcMachineFunctionInfo.h`, `SparcTargetObjectFile.h`, `SparcTargetTransformInfo.h`, `TargetInfo/SparcTargetInfo.h`, `llvm/CodeGen/Passes.h`, `llvm/CodeGen/TargetPassConfig.h`, `llvm/MC/TargetRegistry.h`, `llvm/Support/Compiler.h`, `optional`
- **LLVM subsystems / LLVM 子系统**: MC, CodeGen, Support
