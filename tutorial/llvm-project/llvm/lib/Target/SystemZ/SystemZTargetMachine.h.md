# SystemZTargetMachine.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZTargetMachine.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file configures the target machine and code-generation pipeline for the SystemZ backend.
- **用途 (CN)**: 该文件用于 SystemZ 后端，负责配置目标机器与代码生成流水线。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12 / 第 1-12 行
```cpp
   1: //=- SystemZTargetMachine.h - Define TargetMachine for SystemZ ----*- C++ -*-=//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file declares the SystemZ specific subclass of TargetMachine.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
```
- **EN**: This range is mainly descriptive commentary or banner text that frames the file before executable definitions begin.
- **CN**: 这一段主要是说明性注释或横幅文本，用于在可执行定义开始前说明文件背景。

### Lines 13-24 / 第 13-24 行
```cpp
  13: 
  14: #ifndef LLVM_LIB_TARGET_SYSTEMZ_SYSTEMZTARGETMACHINE_H
  15: #define LLVM_LIB_TARGET_SYSTEMZ_SYSTEMZTARGETMACHINE_H
  16: 
  17: #include "SystemZSubtarget.h"
  18: #include "llvm/ADT/StringRef.h"
  19: #include "llvm/Analysis/TargetTransformInfo.h"
  20: #include "llvm/CodeGen/CodeGenTargetMachineImpl.h"
  21: #include "llvm/Support/CodeGen.h"
  22: #include "llvm/Target/TargetMachine.h"
  23: #include <memory>
  24: #include <optional>
```
- **EN**: It imports dependencies such as `SystemZSubtarget.h`, `StringRef.h`, `TargetTransformInfo.h`, `CodeGenTargetMachineImpl.h`, `CodeGen.h`, `TargetMachine.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 它引入了 `SystemZSubtarget.h`, `StringRef.h`, `TargetTransformInfo.h`, `CodeGenTargetMachineImpl.h`, `CodeGen.h`, `TargetMachine.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 25-36 / 第 25-36 行
```cpp
  25: 
  26: namespace llvm {
  27: 
  28: class SystemZTargetMachine : public CodeGenTargetMachineImpl {
  29:   std::unique_ptr<TargetLoweringObjectFile> TLOF;
  30: 
  31:   mutable StringMap<std::unique_ptr<SystemZSubtarget>> SubtargetMap;
  32: 
  33: public:
  34:   SystemZTargetMachine(const Target &T, const Triple &TT, StringRef CPU,
  35:                        StringRef FS, const TargetOptions &Options,
  36:                        std::optional<Reloc::Model> RM,
```
- **EN**: This block declares or refines TableGen records such as `SystemZTargetMachine`.
- **CN**: 该代码块声明或细化了 `SystemZTargetMachine` 等 TableGen 记录。

### Lines 37-48 / 第 37-48 行
```cpp
  37:                        std::optional<CodeModel::Model> CM, CodeGenOptLevel OL,
  38:                        bool JIT);
  39:   ~SystemZTargetMachine() override;
  40: 
  41:   const SystemZSubtarget *getSubtargetImpl(const Function &) const override;
  42:   // DO NOT IMPLEMENT: There is no such thing as a valid default subtarget,
  43:   // subtargets are per-function entities based on the target-specific
  44:   // attributes of each function.
  45:   const SystemZSubtarget *getSubtargetImpl() const = delete;
  46: 
  47:   // Override CodeGenTargetMachineImpl
  48:   TargetPassConfig *createPassConfig(PassManagerBase &PM) override;
```
- **EN**: This span continues the file's main responsibility: this file configures the target machine and code-generation pipeline for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 49-60 / 第 49-60 行
```cpp
  49:   TargetTransformInfo getTargetTransformInfo(const Function &F) const override;
  50: 
  51:   TargetLoweringObjectFile *getObjFileLowering() const override {
  52:     return TLOF.get();
  53:   }
  54: 
  55:   MachineFunctionInfo *
  56:   createMachineFunctionInfo(BumpPtrAllocator &Allocator, const Function &F,
  57:                             const TargetSubtargetInfo *STI) const override;
  58: 
  59:   ScheduleDAGInstrs *
  60:   createMachineScheduler(MachineSchedContext *C) const override;
```
- **EN**: This span continues the file's main responsibility: this file configures the target machine and code-generation pipeline for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 61-70 / 第 61-70 行
```cpp
  61: 
  62:   ScheduleDAGInstrs *
  63:   createPostMachineScheduler(MachineSchedContext *C) const override;
  64: 
  65:   bool targetSchedulesPostRAScheduling() const override { return true; };
  66: };
  67: 
  68: } // end namespace llvm
  69: 
  70: #endif // LLVM_LIB_TARGET_SYSTEMZ_SYSTEMZTARGETMACHINE_H
```
- **EN**: This span continues the file's main responsibility: this file configures the target machine and code-generation pipeline for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。
- **Scheduling model**: Provides instruction itineraries or scheduling classes for performance modelling. / 提供指令行程或调度类以支持性能建模。
- **Target machine**: Owns data layout, pass configuration, and backend-wide policy. / 管理数据布局、Pass 配置以及整个后端策略。

## Dependencies / 依赖关系
- `SystemZSubtarget.h`
- `llvm/ADT/StringRef.h`
- `llvm/Analysis/TargetTransformInfo.h`
- `llvm/CodeGen/CodeGenTargetMachineImpl.h`
- `llvm/Support/CodeGen.h`
- `llvm/Target/TargetMachine.h`
- `memory`
- `optional`
