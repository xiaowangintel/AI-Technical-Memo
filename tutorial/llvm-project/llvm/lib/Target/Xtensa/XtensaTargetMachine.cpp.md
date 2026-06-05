# XtensaTargetMachine.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Xtensa/XtensaTargetMachine.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Defines the target machine, data layout, pass pipeline hooks, and subtarget creation logic.
  - **CN**: 定义目标机器、数据布局、Pass 流水线钩子以及子目标创建逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-9
```cpp
//===- XtensaTargetMachine.cpp - Define TargetMachine for Xtensa ----------===//
//
//                     The LLVM Compiler Infrastructure
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 10-14
```cpp
//
// Implements the info about Xtensa target spec.
//
//===----------------------------------------------------------------------===//

```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 15-24
```cpp
#include "XtensaTargetMachine.h"
#include "TargetInfo/XtensaTargetInfo.h"
#include "XtensaMachineFunctionInfo.h"
#include "llvm/CodeGen/Passes.h"
#include "llvm/CodeGen/TargetLoweringObjectFileImpl.h"
#include "llvm/CodeGen/TargetPassConfig.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/PassRegistry.h"
#include "llvm/Transforms/Scalar.h"
#include <optional>
```
- **EN**: Pulls in the headers needed for this implementation, including `XtensaTargetMachine.h`, `TargetInfo/XtensaTargetInfo.h`, `XtensaMachineFunctionInfo.h`, `llvm/CodeGen/Passes.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `XtensaTargetMachine.h`, `TargetInfo/XtensaTargetInfo.h`, `XtensaMachineFunctionInfo.h`, `llvm/CodeGen/Passes.h`。

### Lines 25-34
```cpp

using namespace llvm;

extern "C" LLVM_EXTERNAL_VISIBILITY void LLVMInitializeXtensaTarget() {
  // Register the target.
  RegisterTargetMachine<XtensaTargetMachine> A(getTheXtensaTarget());
  PassRegistry &PR = *PassRegistry::getPassRegistry();
  initializeXtensaAsmPrinterPass(PR);
}

```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 35-41
```cpp
static Reloc::Model getEffectiveRelocModel(bool JIT,
                                           std::optional<Reloc::Model> RM) {
  if (!RM || JIT)
     return Reloc::Static;
  return *RM;
}

```
- **EN**: Implements logic around `getEffectiveRelocModel`; this block applies conditional target rules; returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `getEffectiveRelocModel` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位。

### Lines 42-51
```cpp
XtensaTargetMachine::XtensaTargetMachine(const Target &T, const Triple &TT,
                                         StringRef CPU, StringRef FS,
                                         const TargetOptions &Options,
                                         std::optional<Reloc::Model> RM,
                                         std::optional<CodeModel::Model> CM,
                                         CodeGenOptLevel OL, bool JIT,
                                         bool IsLittle)
    : CodeGenTargetMachineImpl(T, TT.computeDataLayout(), TT, CPU, FS, Options,
                               getEffectiveRelocModel(JIT, RM),
                               getEffectiveCodeModel(CM, CodeModel::Small), OL),
```
- **EN**: Implements logic around `XtensaTargetMachine`, `CodeGenTargetMachineImpl`, `getEffectiveRelocModel`, `getEffectiveCodeModel`; this block maps fixups or relocations.
- **CN**: 围绕 `XtensaTargetMachine`, `CodeGenTargetMachineImpl`, `getEffectiveRelocModel`, `getEffectiveCodeModel` 实现具体逻辑；这一段映射 fixup 或重定位。

### Lines 52-61
```cpp
      TLOF(std::make_unique<TargetLoweringObjectFileELF>()) {
  initAsmInfo();
}

XtensaTargetMachine::XtensaTargetMachine(const Target &T, const Triple &TT,
                                         StringRef CPU, StringRef FS,
                                         const TargetOptions &Options,
                                         std::optional<Reloc::Model> RM,
                                         std::optional<CodeModel::Model> CM,
                                         CodeGenOptLevel OL, bool JIT)
```
- **EN**: Implements logic around `TLOF`, `initAsmInfo`, `XtensaTargetMachine`; this block maps fixups or relocations.
- **CN**: 围绕 `TLOF`, `initAsmInfo`, `XtensaTargetMachine` 实现具体逻辑；这一段映射 fixup 或重定位。

### Lines 62-68
```cpp
    : XtensaTargetMachine(T, TT, CPU, FS, Options, RM, CM, OL, JIT, true) {}

const XtensaSubtarget *
XtensaTargetMachine::getSubtargetImpl(const Function &F) const {
  Attribute CPUAttr = F.getFnAttribute("target-cpu");
  Attribute FSAttr = F.getFnAttribute("target-features");

```
- **EN**: Implements logic around `XtensaTargetMachine`, `getSubtargetImpl`, `getFnAttribute`.
- **CN**: 围绕 `XtensaTargetMachine`, `getSubtargetImpl`, `getFnAttribute` 实现具体逻辑。

### Lines 69-78
```cpp
  auto CPU = CPUAttr.isValid() ? CPUAttr.getValueAsString().str() : TargetCPU;
  auto FS = FSAttr.isValid() ? FSAttr.getValueAsString().str() : TargetFS;

  auto &I = SubtargetMap[CPU + FS];
  if (!I) {
    // This needs to be done before we create a new subtarget since any
    // creation will depend on the TM and the code generation flags on the
    // function that reside in TargetOptions.
    resetTargetOptions(F);
    I = std::make_unique<XtensaSubtarget>(TargetTriple, CPU, FS, *this);
```
- **EN**: Implements logic around `isValid`, `resetTargetOptions`, `make_unique<XtensaSubtarget>`; this block applies conditional target rules.
- **CN**: 围绕 `isValid`, `resetTargetOptions`, `make_unique<XtensaSubtarget>` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 79-88
```cpp
  }
  return I.get();
}

MachineFunctionInfo *XtensaTargetMachine::createMachineFunctionInfo(
    BumpPtrAllocator &Allocator, const Function &F,
    const TargetSubtargetInfo *STI) const {
  return XtensaMachineFunctionInfo::create<XtensaMachineFunctionInfo>(Allocator,
                                                                      F, STI);
}
```
- **EN**: Implements logic around `get`, `createMachineFunctionInfo`, `create<XtensaMachineFunctionInfo>`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `get`, `createMachineFunctionInfo`, `create<XtensaMachineFunctionInfo>` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 89-96
```cpp

namespace {
/// Xtensa Code Generator Pass Configuration Options.
class XtensaPassConfig : public TargetPassConfig {
public:
  XtensaPassConfig(XtensaTargetMachine &TM, PassManagerBase &PM)
      : TargetPassConfig(TM, PM) {}

```
- **EN**: Introduces declarations for `XtensaPassConfig`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `XtensaPassConfig` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 97-106
```cpp
  XtensaTargetMachine &getXtensaTargetMachine() const {
    return getTM<XtensaTargetMachine>();
  }

  bool addInstSelector() override;
  void addIRPasses() override;
  void addPreEmitPass() override;
};
} // end anonymous namespace

```
- **EN**: Implements logic around `getXtensaTargetMachine`, `getTM<XtensaTargetMachine>`, `addInstSelector`, `addIRPasses`, ...; this block returns target-specific results.
- **CN**: 围绕 `getXtensaTargetMachine`, `getTM<XtensaTargetMachine>`, `addInstSelector`, `addIRPasses`, ... 实现具体逻辑；这一段返回目标相关结果。

### Lines 107-111
```cpp
bool XtensaPassConfig::addInstSelector() {
  addPass(createXtensaISelDag(getXtensaTargetMachine(), getOptLevel()));
  return false;
}

```
- **EN**: Implements logic around `addInstSelector`, `addPass`; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `addInstSelector`, `addPass` 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 112-116
```cpp
void XtensaPassConfig::addIRPasses() {
  addPass(createAtomicExpandLegacyPass());
  TargetPassConfig::addIRPasses();
}

```
- **EN**: Implements logic around `addIRPasses`, `addPass`.
- **CN**: 围绕 `addIRPasses`, `addPass` 实现具体逻辑。

### Lines 117-121
```cpp
void XtensaPassConfig::addPreEmitPass() { addPass(&BranchRelaxationPassID); }

TargetPassConfig *XtensaTargetMachine::createPassConfig(PassManagerBase &PM) {
  return new XtensaPassConfig(*this, PM);
}
```
- **EN**: Implements logic around `addPreEmitPass`, `createPassConfig`, `XtensaPassConfig`; this block returns target-specific results.
- **CN**: 围绕 `addPreEmitPass`, `createPassConfig`, `XtensaPassConfig` 实现具体逻辑；这一段返回目标相关结果。

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

- **Direct includes / 直接包含**: `XtensaTargetMachine.h`, `TargetInfo/XtensaTargetInfo.h`, `XtensaMachineFunctionInfo.h`, `llvm/CodeGen/Passes.h`, `llvm/CodeGen/TargetLoweringObjectFileImpl.h`, `llvm/CodeGen/TargetPassConfig.h`, `llvm/MC/TargetRegistry.h`, `llvm/PassRegistry.h`, `llvm/Transforms/Scalar.h`, `optional`
- **LLVM subsystems / LLVM 子系统**: MC, CodeGen
