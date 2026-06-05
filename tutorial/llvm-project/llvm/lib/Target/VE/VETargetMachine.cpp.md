# VETargetMachine.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/VE/VETargetMachine.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Defines the target machine, data layout, pass pipeline hooks, and subtarget creation logic.
  - **CN**: 定义目标机器、数据布局、Pass 流水线钩子以及子目标创建逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- VETargetMachine.cpp - Define TargetMachine for VE -----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-17
```cpp
//
//
//===----------------------------------------------------------------------===//

#include "VETargetMachine.h"
#include "TargetInfo/VETargetInfo.h"
#include "VE.h"
#include "VEMachineFunctionInfo.h"
#include "VETargetTransformInfo.h"
#include "llvm/CodeGen/Passes.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `VETargetMachine.h`, `TargetInfo/VETargetInfo.h`, `VE.h`, `VEMachineFunctionInfo.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `VETargetMachine.h`, `TargetInfo/VETargetInfo.h`, `VE.h`, `VEMachineFunctionInfo.h`。

### Lines 18-24
```cpp
#include "llvm/CodeGen/TargetLoweringObjectFileImpl.h"
#include "llvm/CodeGen/TargetPassConfig.h"
#include "llvm/IR/LegacyPassManager.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/Compiler.h"
#include <optional>

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/CodeGen/TargetLoweringObjectFileImpl.h`, `llvm/CodeGen/TargetPassConfig.h`, `llvm/IR/LegacyPassManager.h`, `llvm/MC/TargetRegistry.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/CodeGen/TargetLoweringObjectFileImpl.h`, `llvm/CodeGen/TargetPassConfig.h`, `llvm/IR/LegacyPassManager.h`, `llvm/MC/TargetRegistry.h`。

### Lines 25-32
```cpp
using namespace llvm;

#define DEBUG_TYPE "ve"

extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void LLVMInitializeVETarget() {
  // Register the target.
  RegisterTargetMachine<VETargetMachine> X(getTheVETarget());

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 33-37
```cpp
  PassRegistry &PR = *PassRegistry::getPassRegistry();
  initializeVEAsmPrinterPass(PR);
  initializeVEDAGToDAGISelLegacyPass(PR);
}

```
- **EN**: Implements logic around `getPassRegistry`, `initializeVEAsmPrinterPass`, `initializeVEDAGToDAGISelLegacyPass`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `getPassRegistry`, `initializeVEAsmPrinterPass`, `initializeVEDAGToDAGISelLegacyPass` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 38-47
```cpp
static Reloc::Model getEffectiveRelocModel(std::optional<Reloc::Model> RM) {
  return RM.value_or(Reloc::Static);
}

namespace {
class VEELFTargetObjectFile : public TargetLoweringObjectFileELF {
  void Initialize(MCContext &Ctx, const TargetMachine &TM) override {
    TargetLoweringObjectFileELF::Initialize(Ctx, TM);
    InitializeELF(TM.Options.UseInitArray);
  }
```
- **EN**: Introduces declarations for `VEELFTargetObjectFile`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `VEELFTargetObjectFile` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 48-54
```cpp
};
} // namespace

static std::unique_ptr<TargetLoweringObjectFile> createTLOF() {
  return std::make_unique<VEELFTargetObjectFile>();
}

```
- **EN**: Implements logic around `createTLOF`, `make_unique<VEELFTargetObjectFile>`; this block returns target-specific results.
- **CN**: 围绕 `createTLOF`, `make_unique<VEELFTargetObjectFile>` 实现具体逻辑；这一段返回目标相关结果。

### Lines 55-64
```cpp
/// Create an Aurora VE architecture model
VETargetMachine::VETargetMachine(const Target &T, const Triple &TT,
                                 StringRef CPU, StringRef FS,
                                 const TargetOptions &Options,
                                 std::optional<Reloc::Model> RM,
                                 std::optional<CodeModel::Model> CM,
                                 CodeGenOptLevel OL, bool JIT)
    : CodeGenTargetMachineImpl(T, TT.computeDataLayout(), TT, CPU, FS, Options,
                               getEffectiveRelocModel(RM),
                               getEffectiveCodeModel(CM, CodeModel::Small), OL),
```
- **EN**: Implements logic around `VETargetMachine`, `CodeGenTargetMachineImpl`, `getEffectiveRelocModel`, `getEffectiveCodeModel`; this block maps fixups or relocations.
- **CN**: 围绕 `VETargetMachine`, `CodeGenTargetMachineImpl`, `getEffectiveRelocModel`, `getEffectiveCodeModel` 实现具体逻辑；这一段映射 fixup 或重定位。

### Lines 65-69
```cpp
      TLOF(createTLOF()),
      Subtarget(TT, std::string(CPU), std::string(FS), *this) {
  initAsmInfo();
}

```
- **EN**: Implements logic around `TLOF`, `Subtarget`, `initAsmInfo`.
- **CN**: 围绕 `TLOF`, `Subtarget`, `initAsmInfo` 实现具体逻辑。

### Lines 70-76
```cpp
VETargetMachine::~VETargetMachine() = default;

TargetTransformInfo
VETargetMachine::getTargetTransformInfo(const Function &F) const {
  return TargetTransformInfo(std::make_unique<VETTIImpl>(this, F));
}

```
- **EN**: Implements logic around `~VETargetMachine`, `getTargetTransformInfo`, `TargetTransformInfo`; this block returns target-specific results.
- **CN**: 围绕 `~VETargetMachine`, `getTargetTransformInfo`, `TargetTransformInfo` 实现具体逻辑；这一段返回目标相关结果。

### Lines 77-83
```cpp
MachineFunctionInfo *VETargetMachine::createMachineFunctionInfo(
    BumpPtrAllocator &Allocator, const Function &F,
    const TargetSubtargetInfo *STI) const {
  return VEMachineFunctionInfo::create<VEMachineFunctionInfo>(Allocator, F,
                                                              STI);
}

```
- **EN**: Implements logic around `createMachineFunctionInfo`, `create<VEMachineFunctionInfo>`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `createMachineFunctionInfo`, `create<VEMachineFunctionInfo>` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 84-90
```cpp
namespace {
/// VE Code Generator Pass Configuration Options.
class VEPassConfig : public TargetPassConfig {
public:
  VEPassConfig(VETargetMachine &TM, PassManagerBase &PM)
      : TargetPassConfig(TM, PM) {}

```
- **EN**: Introduces declarations for `VEPassConfig`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `VEPassConfig` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 91-100
```cpp
  VETargetMachine &getVETargetMachine() const {
    return getTM<VETargetMachine>();
  }

  void addIRPasses() override;
  bool addInstSelector() override;
  void addPreEmitPass() override;
};
} // namespace

```
- **EN**: Implements logic around `getVETargetMachine`, `getTM<VETargetMachine>`, `addIRPasses`, `addInstSelector`, ...; this block returns target-specific results.
- **CN**: 围绕 `getVETargetMachine`, `getTM<VETargetMachine>`, `addIRPasses`, `addInstSelector`, ... 实现具体逻辑；这一段返回目标相关结果。

### Lines 101-110
```cpp
TargetPassConfig *VETargetMachine::createPassConfig(PassManagerBase &PM) {
  return new VEPassConfig(*this, PM);
}

void VEPassConfig::addIRPasses() {
  // VE requires atomic expand pass.
  addPass(createAtomicExpandLegacyPass());
  TargetPassConfig::addIRPasses();
}

```
- **EN**: Implements logic around `createPassConfig`, `VEPassConfig`, `addIRPasses`, `addPass`; this block returns target-specific results.
- **CN**: 围绕 `createPassConfig`, `VEPassConfig`, `addIRPasses`, `addPass` 实现具体逻辑；这一段返回目标相关结果。

### Lines 111-115
```cpp
bool VEPassConfig::addInstSelector() {
  addPass(createVEISelDag(getVETargetMachine()));
  return false;
}

```
- **EN**: Implements logic around `addInstSelector`, `addPass`; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `addInstSelector`, `addPass` 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 116-119
```cpp
void VEPassConfig::addPreEmitPass() {
  // LVLGen should be called after scheduling and register allocation
  addPass(createLVLGenPass());
}
```
- **EN**: Implements logic around `addPreEmitPass`, `addPass`.
- **CN**: 围绕 `addPreEmitPass`, `addPass` 实现具体逻辑。

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

- **Direct includes / 直接包含**: `VETargetMachine.h`, `TargetInfo/VETargetInfo.h`, `VE.h`, `VEMachineFunctionInfo.h`, `VETargetTransformInfo.h`, `llvm/CodeGen/Passes.h`, `llvm/CodeGen/TargetLoweringObjectFileImpl.h`, `llvm/CodeGen/TargetPassConfig.h`, `llvm/IR/LegacyPassManager.h`, `llvm/MC/TargetRegistry.h`, `llvm/Support/Compiler.h`, `optional`
- **LLVM subsystems / LLVM 子系统**: MC, CodeGen, IR, Support
