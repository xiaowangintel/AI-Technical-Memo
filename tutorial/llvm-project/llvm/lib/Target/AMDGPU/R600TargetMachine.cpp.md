# R600TargetMachine.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/R600TargetMachine.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements R600TargetMachine for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 R600TargetMachine 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-32: File banner, includes, and setup
```cpp
//===-- R600TargetMachine.cpp - TargetMachine for hw codegen targets-------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// This file contains both AMDGPU-R600 target machine and the CodeGen pass
/// builder. The target machine contains all of the hardware specific
/// information needed to emit code for R600 GPUs and the CodeGen pass builder
/// handles the pass pipeline for new pass manager.
//
//===----------------------------------------------------------------------===//

#include "R600TargetMachine.h"
#include "R600.h"
#include "R600MachineFunctionInfo.h"
#include "R600MachineScheduler.h"
#include "R600TargetTransformInfo.h"
#include "llvm/Passes/CodeGenPassBuilder.h"
#include "llvm/Transforms/Scalar.h"
#include <optional>

using namespace llvm;

static cl::opt<bool>
    EnableR600StructurizeCFG("r600-ir-structurize",
                             cl::desc("Use StructurizeCFG IR pass"),
                             cl::init(true));

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `cl::desc`, `cl::init`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`cl::desc`, `cl::init`。

### Lines 33-66: Declares class R600CodeGenPassBuilder
```cpp
static cl::opt<bool> EnableR600IfConvert("r600-if-convert",
                                         cl::desc("Use if conversion pass"),
                                         cl::ReallyHidden, cl::init(true));

static cl::opt<bool, true> EnableAMDGPUFunctionCallsOpt(
    "amdgpu-function-calls", cl::desc("Enable AMDGPU function call support"),
    cl::location(AMDGPUTargetMachine::EnableFunctionCalls), cl::init(true),
    cl::Hidden);

static ScheduleDAGInstrs *createR600MachineScheduler(MachineSchedContext *C) {
  return new ScheduleDAGMILive(C, std::make_unique<R600SchedStrategy>());
}

static MachineSchedRegistry R600SchedRegistry("r600",
                                              "Run R600's custom scheduler",
                                              createR600MachineScheduler);

//===----------------------------------------------------------------------===//
// R600 CodeGen Pass Builder interface.
//===----------------------------------------------------------------------===//

class R600CodeGenPassBuilder
    : public CodeGenPassBuilder<R600CodeGenPassBuilder, R600TargetMachine> {
public:
  R600CodeGenPassBuilder(R600TargetMachine &TM, const CGPassBuilderOption &Opts,
                         PassInstrumentationCallbacks *PIC);

  void addPreISel(PassManagerWrapper &PMW) const;
  void addAsmPrinterBegin(PassManagerWrapper &PMW) const;
  void addAsmPrinter(PassManagerWrapper &PMW) const;
  void addAsmPrinterEnd(PassManagerWrapper &PMW) const;
  Error addInstSelector(PassManagerWrapper &PMW) const;
};

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `R600CodeGenPassBuilder`, `cl::desc`, `cl::init`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`R600CodeGenPassBuilder`, `cl::desc`, `cl::init`。

### Lines 67-93: Implements R600TargetMachine::R600TargetMachine
```cpp
//===----------------------------------------------------------------------===//
// R600 Target Machine (R600 -> Cayman)
//===----------------------------------------------------------------------===//

R600TargetMachine::R600TargetMachine(const Target &T, const Triple &TT,
                                     StringRef CPU, StringRef FS,
                                     const TargetOptions &Options,
                                     std::optional<Reloc::Model> RM,
                                     std::optional<CodeModel::Model> CM,
                                     CodeGenOptLevel OL, bool JIT)
    : AMDGPUTargetMachine(T, TT, CPU, FS, Options, RM, CM, OL) {
  setRequiresStructuredCFG(true);

  // Override the default since calls aren't supported for r600.
  if (EnableFunctionCalls &&
      EnableAMDGPUFunctionCallsOpt.getNumOccurrences() == 0)
    EnableFunctionCalls = false;
}

const TargetSubtargetInfo *
R600TargetMachine::getSubtargetImpl(const Function &F) const {
  StringRef GPU = getGPUName(F);
  StringRef FS = getFeatureString(F);

  SmallString<128> SubtargetKey(GPU);
  SubtargetKey.append(FS);

```
**EN:** This section contains concrete logic for R600TargetMachine::R600TargetMachine. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600TargetMachine::R600TargetMachine`, `R600TargetMachine::getSubtargetImpl`.
**CN:** 本节包含与 R600TargetMachine::R600TargetMachine 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600TargetMachine::R600TargetMachine`, `R600TargetMachine::getSubtargetImpl`。

### Lines 94-121: Declares class R600PassConfig
```cpp
  auto &I = SubtargetMap[SubtargetKey];
  if (!I) {
    // This needs to be done before we create a new subtarget since any
    // creation will depend on the TM and the code generation flags on the
    // function that reside in TargetOptions.
    resetTargetOptions(F);
    I = std::make_unique<R600Subtarget>(TargetTriple, GPU, FS, *this);
  }

  return I.get();
}

TargetTransformInfo
R600TargetMachine::getTargetTransformInfo(const Function &F) const {
  return TargetTransformInfo(std::make_unique<R600TTIImpl>(this, F));
}

ScheduleDAGInstrs *
R600TargetMachine::createMachineScheduler(MachineSchedContext *C) const {
  return createR600MachineScheduler(C);
}

namespace {
class R600PassConfig final : public AMDGPUPassConfig {
public:
  R600PassConfig(TargetMachine &TM, PassManagerBase &PM)
      : AMDGPUPassConfig(TM, PM) {}

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `R600PassConfig`, `R600TargetMachine::getTargetTransformInfo`, `R600TargetMachine::createMachineScheduler`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`R600PassConfig`, `R600TargetMachine::getTargetTransformInfo`, `R600TargetMachine::createMachineScheduler`。

### Lines 122-155: Declares addPreISel
```cpp
  bool addPreISel() override;
  bool addInstSelector() override;
  void addPreRegAlloc() override;
  void addPreSched2() override;
  void addPreEmitPass() override;
};
} // namespace

//===----------------------------------------------------------------------===//
// R600 Pass Setup
//===----------------------------------------------------------------------===//

bool R600PassConfig::addPreISel() {
  AMDGPUPassConfig::addPreISel();

  if (EnableR600StructurizeCFG)
    addPass(createStructurizeCFGPass());
  return false;
}

bool R600PassConfig::addInstSelector() {
  addPass(createR600ISelDag(getAMDGPUTargetMachine(), getOptLevel()));
  return false;
}

void R600PassConfig::addPreRegAlloc() { addPass(createR600VectorRegMerger()); }

void R600PassConfig::addPreSched2() {
  addPass(createR600EmitClauseMarkers());
  if (EnableR600IfConvert)
    addPass(&IfConverterID);
  addPass(createR600ClauseMergePass());
}

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `R600PassConfig::addPreISel`, `AMDGPUPassConfig::addPreISel`, `R600PassConfig::addInstSelector`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`R600PassConfig::addPreISel`, `AMDGPUPassConfig::addPreISel`, `R600PassConfig::addInstSelector`。

### Lines 156-186: Implements R600PassConfig::addPreEmitPass
```cpp
void R600PassConfig::addPreEmitPass() {
  addPass(createR600MachineCFGStructurizerPass());
  addPass(createR600ExpandSpecialInstrsPass());
  addPass(createR600Packetizer());
  addPass(createR600ControlFlowFinalizer());
}

TargetPassConfig *R600TargetMachine::createPassConfig(PassManagerBase &PM) {
  return new R600PassConfig(*this, PM);
}

Error R600TargetMachine::buildCodeGenPipeline(
    ModulePassManager &MPM, ModuleAnalysisManager &MAM, raw_pwrite_stream &Out,
    raw_pwrite_stream *DwoOut, CodeGenFileType FileType,
    const CGPassBuilderOption &Opts, MCContext &Ctx,
    PassInstrumentationCallbacks *PIC) {
  R600CodeGenPassBuilder CGPB(*this, Opts, PIC);
  return CGPB.buildPipeline(MPM, MAM, Out, DwoOut, FileType, Ctx);
}

MachineFunctionInfo *R600TargetMachine::createMachineFunctionInfo(
    BumpPtrAllocator &Allocator, const Function &F,
    const TargetSubtargetInfo *STI) const {
  return R600MachineFunctionInfo::create<R600MachineFunctionInfo>(
      Allocator, F, static_cast<const R600Subtarget *>(STI));
}

//===----------------------------------------------------------------------===//
// R600 CodeGen Pass Builder interface.
//===----------------------------------------------------------------------===//

```
**EN:** This section contains concrete logic for R600PassConfig::addPreEmitPass. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600PassConfig::addPreEmitPass`, `R600TargetMachine::createPassConfig`, `R600TargetMachine::buildCodeGenPipeline`.
**CN:** 本节包含与 R600PassConfig::addPreEmitPass 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600PassConfig::addPreEmitPass`, `R600TargetMachine::createPassConfig`, `R600TargetMachine::buildCodeGenPipeline`。

### Lines 187-213: Implements R600CodeGenPassBuilder::R600CodeGenPassBuilder
```cpp
R600CodeGenPassBuilder::R600CodeGenPassBuilder(
    R600TargetMachine &TM, const CGPassBuilderOption &Opts,
    PassInstrumentationCallbacks *PIC)
    : CodeGenPassBuilder(TM, Opts, PIC) {
  Opt.RequiresCodeGenSCCOrder = true;
}

void R600CodeGenPassBuilder::addPreISel(PassManagerWrapper &PMW) const {
  // TODO: Add passes pre instruction selection.
}

void R600CodeGenPassBuilder::addAsmPrinterBegin(PassManagerWrapper &PMW) const {
  // TODO: Add AsmPrinterBegin
}

void R600CodeGenPassBuilder::addAsmPrinter(PassManagerWrapper &PMW) const {
  // TODO: Add AsmPrinter.
}

void R600CodeGenPassBuilder::addAsmPrinterEnd(PassManagerWrapper &PMW) const {
  // TODO: Add AsmPrinterEnd
}

Error R600CodeGenPassBuilder::addInstSelector(PassManagerWrapper &PMW) const {
  // TODO: Add instruction selector.
  return Error::success();
}
```
**EN:** This section contains concrete logic for R600CodeGenPassBuilder::R600CodeGenPassBuilder. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600CodeGenPassBuilder::R600CodeGenPassBuilder`, `R600CodeGenPassBuilder::addPreISel`, `R600CodeGenPassBuilder::addAsmPrinterBegin`.
**CN:** 本节包含与 R600CodeGenPassBuilder::R600CodeGenPassBuilder 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600CodeGenPassBuilder::R600CodeGenPassBuilder`, `R600CodeGenPassBuilder::addPreISel`, `R600CodeGenPassBuilder::addAsmPrinterBegin`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `R600CodeGenPassBuilder`, `R600PassConfig`, `cl::desc`, `cl::init`, `cl::location`, `R600TargetMachine::R600TargetMachine`
- **Main themes / 核心主题**: instruction semantics / 指令语义; subtarget modeling / 子目标建模; scheduling / 调度; assembly handling / 汇编处理
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"R600TargetMachine.h"`
- `"R600.h"`
- `"R600MachineFunctionInfo.h"`
- `"R600MachineScheduler.h"`
- `"R600TargetTransformInfo.h"`
- `"llvm/Passes/CodeGenPassBuilder.h"`
- `"llvm/Transforms/Scalar.h"`
- `<optional>`
