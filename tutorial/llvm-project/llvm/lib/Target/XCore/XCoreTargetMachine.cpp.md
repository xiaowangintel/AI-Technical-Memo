# XCoreTargetMachine.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/XCore/XCoreTargetMachine.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Defines the target machine, data layout, pass pipeline hooks, and subtarget creation logic.
  - **CN**: 定义目标机器、数据布局、Pass 流水线钩子以及子目标创建逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- XCoreTargetMachine.cpp - Define TargetMachine for XCore -----------===//
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

#include "XCoreTargetMachine.h"
#include "TargetInfo/XCoreTargetInfo.h"
#include "XCore.h"
#include "XCoreMachineFunctionInfo.h"
#include "XCoreTargetObjectFile.h"
#include "XCoreTargetTransformInfo.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `XCoreTargetMachine.h`, `TargetInfo/XCoreTargetInfo.h`, `XCore.h`, `XCoreMachineFunctionInfo.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `XCoreTargetMachine.h`, `TargetInfo/XCoreTargetInfo.h`, `XCore.h`, `XCoreMachineFunctionInfo.h`。

### Lines 18-25
```cpp
#include "llvm/Analysis/TargetTransformInfo.h"
#include "llvm/CodeGen/Passes.h"
#include "llvm/CodeGen/TargetPassConfig.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/CodeGen.h"
#include "llvm/Support/Compiler.h"
#include <optional>

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/Analysis/TargetTransformInfo.h`, `llvm/CodeGen/Passes.h`, `llvm/CodeGen/TargetPassConfig.h`, `llvm/MC/TargetRegistry.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/Analysis/TargetTransformInfo.h`, `llvm/CodeGen/Passes.h`, `llvm/CodeGen/TargetPassConfig.h`, `llvm/MC/TargetRegistry.h`。

### Lines 26-31
```cpp
using namespace llvm;

static Reloc::Model getEffectiveRelocModel(std::optional<Reloc::Model> RM) {
  return RM.value_or(Reloc::Static);
}

```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 32-41
```cpp
static CodeModel::Model
getEffectiveXCoreCodeModel(std::optional<CodeModel::Model> CM) {
  if (CM) {
    if (*CM != CodeModel::Small && *CM != CodeModel::Large)
      report_fatal_error("Target only supports CodeModel Small or Large");
    return *CM;
  }
  return CodeModel::Small;
}

```
- **EN**: Implements logic around `getEffectiveXCoreCodeModel`, `report_fatal_error`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getEffectiveXCoreCodeModel`, `report_fatal_error` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 42-51
```cpp
/// Create an ILP32 architecture model
///
XCoreTargetMachine::XCoreTargetMachine(const Target &T, const Triple &TT,
                                       StringRef CPU, StringRef FS,
                                       const TargetOptions &Options,
                                       std::optional<Reloc::Model> RM,
                                       std::optional<CodeModel::Model> CM,
                                       CodeGenOptLevel OL, bool JIT)
    : CodeGenTargetMachineImpl(T, TT.computeDataLayout(), TT, CPU, FS, Options,
                               getEffectiveRelocModel(RM),
```
- **EN**: Implements logic around `XCoreTargetMachine`, `CodeGenTargetMachineImpl`, `getEffectiveRelocModel`; this block maps fixups or relocations.
- **CN**: 围绕 `XCoreTargetMachine`, `CodeGenTargetMachineImpl`, `getEffectiveRelocModel` 实现具体逻辑；这一段映射 fixup 或重定位。

### Lines 52-57
```cpp
                               getEffectiveXCoreCodeModel(CM), OL),
      TLOF(std::make_unique<XCoreTargetObjectFile>()),
      Subtarget(TT, std::string(CPU), std::string(FS), *this) {
  initAsmInfo();
}

```
- **EN**: Implements logic around `getEffectiveXCoreCodeModel`, `TLOF`, `Subtarget`, `initAsmInfo`.
- **CN**: 围绕 `getEffectiveXCoreCodeModel`, `TLOF`, `Subtarget`, `initAsmInfo` 实现具体逻辑。

### Lines 58-62
```cpp
XCoreTargetMachine::~XCoreTargetMachine() = default;

namespace {

/// XCore Code Generator Pass Configuration Options.
```
- **EN**: Implements logic around `~XCoreTargetMachine`.
- **CN**: 围绕 `~XCoreTargetMachine` 实现具体逻辑。

### Lines 63-67
```cpp
class XCorePassConfig : public TargetPassConfig {
public:
  XCorePassConfig(XCoreTargetMachine &TM, PassManagerBase &PM)
    : TargetPassConfig(TM, PM) {}

```
- **EN**: Introduces declarations for `XCorePassConfig`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `XCorePassConfig` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 68-77
```cpp
  XCoreTargetMachine &getXCoreTargetMachine() const {
    return getTM<XCoreTargetMachine>();
  }

  void addIRPasses() override;
  bool addPreISel() override;
  bool addInstSelector() override;
  void addPreEmitPass() override;
};

```
- **EN**: Implements logic around `getXCoreTargetMachine`, `getTM<XCoreTargetMachine>`, `addIRPasses`, `addPreISel`, ...; this block returns target-specific results.
- **CN**: 围绕 `getXCoreTargetMachine`, `getTM<XCoreTargetMachine>`, `addIRPasses`, `addPreISel`, ... 实现具体逻辑；这一段返回目标相关结果。

### Lines 78-83
```cpp
} // end anonymous namespace

TargetPassConfig *XCoreTargetMachine::createPassConfig(PassManagerBase &PM) {
  return new XCorePassConfig(*this, PM);
}

```
- **EN**: Implements logic around `createPassConfig`, `XCorePassConfig`; this block returns target-specific results.
- **CN**: 围绕 `createPassConfig`, `XCorePassConfig` 实现具体逻辑；这一段返回目标相关结果。

### Lines 84-89
```cpp
void XCorePassConfig::addIRPasses() {
  addPass(createAtomicExpandLegacyPass());

  TargetPassConfig::addIRPasses();
}

```
- **EN**: Implements logic around `addIRPasses`, `addPass`.
- **CN**: 围绕 `addIRPasses`, `addPass` 实现具体逻辑。

### Lines 90-94
```cpp
bool XCorePassConfig::addPreISel() {
  addPass(createXCoreLowerThreadLocalPass());
  return false;
}

```
- **EN**: Implements logic around `addPreISel`, `addPass`; this block returns target-specific results.
- **CN**: 围绕 `addPreISel`, `addPass` 实现具体逻辑；这一段返回目标相关结果。

### Lines 95-99
```cpp
bool XCorePassConfig::addInstSelector() {
  addPass(createXCoreISelDag(getXCoreTargetMachine(), getOptLevel()));
  return false;
}

```
- **EN**: Implements logic around `addInstSelector`, `addPass`; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `addInstSelector`, `addPass` 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 100-109
```cpp
void XCorePassConfig::addPreEmitPass() {
  addPass(createXCoreFrameToArgsOffsetEliminationPass());
}

// Force static initialization.
extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void LLVMInitializeXCoreTarget() {
  RegisterTargetMachine<XCoreTargetMachine> X(getTheXCoreTarget());
  PassRegistry &PR = *PassRegistry::getPassRegistry();
  initializeXCoreAsmPrinterPass(PR);
  initializeXCoreDAGToDAGISelLegacyPass(PR);
```
- **EN**: Implements logic around `addPreEmitPass`, `addPass`, `X`, `getPassRegistry`, ...; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `addPreEmitPass`, `addPass`, `X`, `getPassRegistry`, ... 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 110-117
```cpp
  initializeXCoreLowerThreadLocalPass(PR);
}

TargetTransformInfo
XCoreTargetMachine::getTargetTransformInfo(const Function &F) const {
  return TargetTransformInfo(std::make_unique<XCoreTTIImpl>(this, F));
}

```
- **EN**: Implements logic around `initializeXCoreLowerThreadLocalPass`, `getTargetTransformInfo`, `TargetTransformInfo`; this block returns target-specific results.
- **CN**: 围绕 `initializeXCoreLowerThreadLocalPass`, `getTargetTransformInfo`, `TargetTransformInfo` 实现具体逻辑；这一段返回目标相关结果。

### Lines 118-122
```cpp
MachineFunctionInfo *XCoreTargetMachine::createMachineFunctionInfo(
    BumpPtrAllocator &Allocator, const Function &F,
    const TargetSubtargetInfo *STI) const {
  return XCoreFunctionInfo::create<XCoreFunctionInfo>(Allocator, F, STI);
}
```
- **EN**: Implements logic around `createMachineFunctionInfo`, `create<XCoreFunctionInfo>`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `createMachineFunctionInfo`, `create<XCoreFunctionInfo>` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

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

- **Direct includes / 直接包含**: `XCoreTargetMachine.h`, `TargetInfo/XCoreTargetInfo.h`, `XCore.h`, `XCoreMachineFunctionInfo.h`, `XCoreTargetObjectFile.h`, `XCoreTargetTransformInfo.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/CodeGen/Passes.h`, `llvm/CodeGen/TargetPassConfig.h`, `llvm/MC/TargetRegistry.h`, `llvm/Support/CodeGen.h`, `llvm/Support/Compiler.h` ... (+1 more)
- **LLVM subsystems / LLVM 子系统**: MC, CodeGen, Support
