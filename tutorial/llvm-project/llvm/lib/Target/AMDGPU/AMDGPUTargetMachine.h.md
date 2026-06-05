# AMDGPUTargetMachine.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUTargetMachine.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This header declares the interfaces, data structures, and pass entry points for AMDGPUTargetMachine in the LLVM AMDGPU backend. It defines the contracts consumed by other AMDGPU backend components. / 该头文件声明 LLVM AMDGPU 后端中 AMDGPUTargetMachine 的接口、数据结构与 Pass 入口。它定义了其他 AMDGPU 后端组件依赖的契约。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: File banner, includes, and setup
```cpp
//===-- AMDGPUTargetMachine.h - AMDGPU TargetMachine Interface --*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// The AMDGPU TargetMachine interface definition for hw codegen targets.
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AMDGPU_AMDGPUTARGETMACHINE_H
#define LLVM_LIB_TARGET_AMDGPU_AMDGPUTARGETMACHINE_H

#include "GCNSubtarget.h"
#include "llvm/CodeGen/CodeGenTargetMachineImpl.h"
#include "llvm/CodeGen/TargetPassConfig.h"
#include "llvm/MC/MCStreamer.h"
#include <optional>
#include <utility>

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 24-45: Declares class AMDGPUTargetMachine
```cpp
namespace llvm {

//===----------------------------------------------------------------------===//
// AMDGPU Target Machine (R600+)
//===----------------------------------------------------------------------===//

namespace AMDGPU {
StringRef getSchedStrategy(const Function &F);
}

class AMDGPUTargetMachine : public CodeGenTargetMachineImpl {
protected:
  std::unique_ptr<TargetLoweringObjectFile> TLOF;

  StringRef getGPUName(const Function &F) const;
  StringRef getFeatureString(const Function &F) const;

public:
  static bool EnableFunctionCalls;
  static bool EnableObjectLinking;
  static bool EnableLowerModuleLDS;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUTargetMachine`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUTargetMachine`。

### Lines 46-69: Defines AMDGPUTargetMachine
```cpp
  AMDGPUTargetMachine(const Target &T, const Triple &TT, StringRef CPU,
                      StringRef FS, const TargetOptions &Options,
                      std::optional<Reloc::Model> RM,
                      std::optional<CodeModel::Model> CM, CodeGenOptLevel OL);
  ~AMDGPUTargetMachine() override;

  const TargetSubtargetInfo *getSubtargetImpl() const;
  const TargetSubtargetInfo *
  getSubtargetImpl(const Function &) const override = 0;

  TargetLoweringObjectFile *getObjFileLowering() const override {
    return TLOF.get();
  }

  void registerPassBuilderCallbacks(PassBuilder &PB) override;
  void registerDefaultAliasAnalyses(AAManager &) override;

  bool isNoopAddrSpaceCast(unsigned SrcAS, unsigned DestAS) const override;

  unsigned getAssumedAddrSpace(const Value *V) const override;

  std::pair<const Value *, unsigned>
  getPredicatedAddrSpace(const Value *V) const override;

```
**EN:** This section contains concrete logic for AMDGPUTargetMachine. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 AMDGPUTargetMachine 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 70-93: Declares class GCNTargetMachine
```cpp
  unsigned getAddressSpaceForPseudoSourceKind(unsigned Kind) const override;

  bool splitModule(Module &M, unsigned NumParts,
                   function_ref<void(std::unique_ptr<Module> MPart)>
                       ModuleCallback) override;
  ScheduleDAGInstrs *
  createMachineScheduler(MachineSchedContext *C) const override;
};

//===----------------------------------------------------------------------===//
// GCN Target Machine (SI+)
//===----------------------------------------------------------------------===//

class GCNTargetMachine final : public AMDGPUTargetMachine {
private:
  mutable StringMap<std::unique_ptr<GCNSubtarget>> SubtargetMap;

public:
  GCNTargetMachine(const Target &T, const Triple &TT, StringRef CPU,
                   StringRef FS, const TargetOptions &Options,
                   std::optional<Reloc::Model> RM,
                   std::optional<CodeModel::Model> CM, CodeGenOptLevel OL,
                   bool JIT);

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `GCNTargetMachine`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`GCNTargetMachine`。

### Lines 94-113: Declares createPassConfig
```cpp
  TargetPassConfig *createPassConfig(PassManagerBase &PM) override;

  const TargetSubtargetInfo *getSubtargetImpl(const Function &) const override;

  TargetTransformInfo getTargetTransformInfo(const Function &F) const override;

  bool useIPRA() const override { return true; }

  Error buildCodeGenPipeline(ModulePassManager &MPM, ModuleAnalysisManager &MAM,
                             raw_pwrite_stream &Out, raw_pwrite_stream *DwoOut,
                             CodeGenFileType FileType,
                             const CGPassBuilderOption &Opts, MCContext &Ctx,
                             PassInstrumentationCallbacks *PIC) override;

  void registerMachineRegisterInfoCallback(MachineFunction &MF) const override;

  MachineFunctionInfo *
  createMachineFunctionInfo(BumpPtrAllocator &Allocator, const Function &F,
                            const TargetSubtargetInfo *STI) const override;

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 114-134: Declares class AMDGPUPassConfig
```cpp
  yaml::MachineFunctionInfo *createDefaultFuncInfoYAML() const override;
  yaml::MachineFunctionInfo *
  convertFuncInfoToYAML(const MachineFunction &MF) const override;
  bool parseMachineFunctionInfo(const yaml::MachineFunctionInfo &,
                                PerFunctionMIParsingState &PFS,
                                SMDiagnostic &Error,
                                SMRange &SourceRange) const override;
  ScheduleDAGInstrs *
  createMachineScheduler(MachineSchedContext *C) const override;
  ScheduleDAGInstrs *
  createPostMachineScheduler(MachineSchedContext *C) const override;
};

//===----------------------------------------------------------------------===//
// AMDGPU Pass Setup - For Legacy Pass Manager.
//===----------------------------------------------------------------------===//

class AMDGPUPassConfig : public TargetPassConfig {
public:
  AMDGPUPassConfig(TargetMachine &TM, PassManagerBase &PM);

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUPassConfig`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUPassConfig`。

### Lines 135-147: Defines getAMDGPUTargetMachine
```cpp
  AMDGPUTargetMachine &getAMDGPUTargetMachine() const {
    return getTM<AMDGPUTargetMachine>();
  }
  void addEarlyCSEOrGVNPass();
  void addStraightLineScalarOptimizationPasses();
  void addIRPasses() override;
  void addCodeGenPrepare() override;
  bool addPreISel() override;
  bool addInstSelector() override;
  bool addGCPasses() override;

  std::unique_ptr<CSEConfigBase> getCSEConfig() const override;

```
**EN:** This section contains concrete logic for getAMDGPUTargetMachine. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 getAMDGPUTargetMachine 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 148-164: Preprocessor guards and macros
```cpp
  /// Check if a pass is enabled given \p Opt option. The option always
  /// overrides defaults if explicitly used. Otherwise its default will
  /// be used given that a pass shall work at an optimization \p Level
  /// minimum.
  bool isPassEnabled(const cl::opt<bool> &Opt,
                     CodeGenOptLevel Level = CodeGenOptLevel::Default) const {
    if (Opt.getNumOccurrences())
      return Opt;
    if (TM->getOptLevel() < Level)
      return false;
    return Opt;
  }
};

} // end namespace llvm

#endif // LLVM_LIB_TARGET_AMDGPU_AMDGPUTARGETMACHINE_H
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。

## Key Concepts / 关键概念
- **Language / 语言**: C++ header
- **Primary symbols / 主要符号**: `AMDGPUTargetMachine`, `GCNTargetMachine`, `AMDGPUPassConfig`
- **Main themes / 核心主题**: alias analysis / 别名分析; register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"GCNSubtarget.h"`
- `"llvm/CodeGen/CodeGenTargetMachineImpl.h"`
- `"llvm/CodeGen/TargetPassConfig.h"`
- `"llvm/MC/MCStreamer.h"`
- `<optional>`
- `<utility>`
