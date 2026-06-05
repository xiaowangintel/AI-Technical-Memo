# AMDGPUMIRFormatter.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUMIRFormatter.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This header declares the interfaces, data structures, and pass entry points for AMDGPUMIRFormatter in the LLVM AMDGPU backend. It defines the contracts consumed by other AMDGPU backend components. / 该头文件声明 LLVM AMDGPU 后端中 AMDGPUMIRFormatter 的接口、数据结构与 Pass 入口。它定义了其他 AMDGPU 后端组件依赖的契约。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: File banner, license, and overview
```cpp
//===-- llvm/Target/AMDGPU/AMDGPUMIRFormatter.h -----------------*- C++ -*-===//
//
//                     The LLVM Compiler Infrastructure
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// AMDGPU specific overrides of MIRFormatter.
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AMDGPUMIRFORMATTER_H
#define LLVM_LIB_TARGET_AMDGPUMIRFORMATTER_H

```
**EN:** This opening section establishes the file identity, licensing terms, and high-level intent before the backend-specific code begins.
**CN:** 开头部分先给出文件标识、许可证信息以及总体意图，为后续后端专用代码建立上下文。

### Lines 19-31: Header dependencies and setup
```cpp
#include "Utils/AMDGPUBaseInfo.h"
#include "llvm/CodeGen/MIRFormatter.h"

namespace llvm {

class MachineFunction;
struct PerFunctionMIParsingState;

class AMDGPUMIRFormatter final : public MIRFormatter {
public:
  explicit AMDGPUMIRFormatter(const MCSubtargetInfo &STI) : STI(STI) {}
  ~AMDGPUMIRFormatter() override = default;

```
**EN:** These includes pull in the LLVM and AMDGPU declarations required by the rest of the file, making later symbols and helper APIs available. Main symbols: `MachineFunction`, `PerFunctionMIParsingState`, `AMDGPUMIRFormatter`.
**CN:** 这些头文件引入本文件后续实现所需的 LLVM 与 AMDGPU 声明，使后面的符号和辅助 API 可以直接使用。 主要符号：`MachineFunction`, `PerFunctionMIParsingState`, `AMDGPUMIRFormatter`。

### Lines 32-43: Defines printImm
```cpp
  /// Implement target specific printing for machine operand immediate value, so
  /// that we can have more meaningful mnemonic than a 64-bit integer. Passing
  /// None to OpIdx means the index is unknown.
  void printImm(raw_ostream &OS, const MachineInstr &MI,
                std::optional<unsigned> OpIdx, int64_t Imm) const override;

  /// Implement target specific parsing of immediate mnemonics. The mnemonic is
  /// a string with a leading dot.
  bool parseImmMnemonic(const unsigned OpCode, const unsigned OpIdx,
                        StringRef Src, int64_t &Imm,
                        ErrorCallbackType ErrorCallback) const override;

```
**EN:** This section contains concrete logic for printImm. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 printImm 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 44-59: Defines parseCustomPseudoSourceValue
```cpp
  /// Implement target specific parsing of target custom pseudo source value.
  bool
  parseCustomPseudoSourceValue(StringRef Src, MachineFunction &MF,
                               PerFunctionMIParsingState &PFS,
                               const PseudoSourceValue *&PSV,
                               ErrorCallbackType ErrorCallback) const override;

private:
  const MCSubtargetInfo &STI;
  /// Prints the string to represent s_wait_alu immediate value.
  void printSWaitAluImm(uint64_t Imm, raw_ostream &OS) const;
  /// Prints the string to represent s_waitcnt immediate value.
  void printSWaitcntImm(uint64_t Imm, raw_ostream &OS) const;
  /// Print the string to represent s_delay_alu immediate value
  void printSDelayAluImm(int64_t Imm, llvm::raw_ostream &OS) const;

```
**EN:** This section contains concrete logic for parseCustomPseudoSourceValue. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 parseCustomPseudoSourceValue 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 60-76: Defines parseSWaitAluImmMnemonic
```cpp
  /// Parse the immediate pseudo literal for s_wait_alu
  bool parseSWaitAluImmMnemonic(
      const unsigned int OpIdx, int64_t &Imm, StringRef &Src,
      MIRFormatter::ErrorCallbackType &ErrorCallback) const;

  /// Parse the immediate pseudo literal for s_waitcnt
  bool parseSWaitcntImmMnemonic(
      const unsigned int OpIdx, int64_t &Imm, StringRef &Src,
      MIRFormatter::ErrorCallbackType &ErrorCallback) const;

  /// Parse the immediate pseudo literal for s_delay_alu
  bool parseSDelayAluImmMnemonic(
      const unsigned int OpIdx, int64_t &Imm, llvm::StringRef &Src,
      llvm::MIRFormatter::ErrorCallbackType &ErrorCallback) const;

};

```
**EN:** This section contains concrete logic for parseSWaitAluImmMnemonic. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 parseSWaitAluImmMnemonic 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 77-79: Preprocessor guards and macros
```cpp
} // end namespace llvm

#endif
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。

## Key Concepts / 关键概念
- **Language / 语言**: C++ header
- **Primary symbols / 主要符号**: `MachineFunction`, `PerFunctionMIParsingState`, `AMDGPUMIRFormatter`
- **Main themes / 核心主题**: instruction semantics / 指令语义; subtarget modeling / 子目标建模; LLVM pass integration / LLVM Pass 集成; machine-function state / MachineFunction 状态
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"Utils/AMDGPUBaseInfo.h"`
- `"llvm/CodeGen/MIRFormatter.h"`
