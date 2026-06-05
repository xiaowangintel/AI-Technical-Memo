# AMDGPUMachineFunctionInfo.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUMachineFunctionInfo.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This header declares the interfaces, data structures, and pass entry points for AMDGPUMachineFunctionInfo in the LLVM AMDGPU backend. It defines the contracts consumed by other AMDGPU backend components. / 该头文件声明 LLVM AMDGPU 后端中 AMDGPUMachineFunctionInfo 的接口、数据结构与 Pass 入口。它定义了其他 AMDGPU 后端组件依赖的契约。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: File banner, includes, and setup
```cpp
//===-- AMDGPUMachineFunctionInfo.h -------------------------------*- C++ -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AMDGPU_AMDGPUMACHINEFUNCTION_H
#define LLVM_LIB_TARGET_AMDGPU_AMDGPUMACHINEFUNCTION_H

#include "Utils/AMDGPUBaseInfo.h"
#include "llvm/ADT/DenseMap.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/IR/DataLayout.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/GlobalValue.h"
#include "llvm/IR/GlobalVariable.h"

namespace llvm {

class AMDGPUSubtarget;

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `AMDGPUSubtarget`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`AMDGPUSubtarget`。

### Lines 24-41: Declares class AMDGPUMachineFunctionInfo
```cpp
class AMDGPUMachineFunctionInfo : public MachineFunctionInfo {
  /// A map to keep track of local memory objects and their offsets within the
  /// local memory space.
  SmallDenseMap<const GlobalValue *, unsigned, 4> LocalMemoryObjects;

protected:
  uint64_t ExplicitKernArgSize = 0; // Cache for this.
  Align MaxKernArgAlign;            // Cache for this.

  /// Number of bytes in the LDS that are being used.
  uint32_t LDSSize = 0;
  uint32_t GDSSize = 0;

  /// Number of bytes in the LDS allocated statically. This field is only used
  /// in the instruction selector and not part of the machine function info.
  uint32_t StaticLDSSize = 0;
  uint32_t StaticGDSSize = 0;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUMachineFunctionInfo`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUMachineFunctionInfo`。

### Lines 42-63: Implementation details and local logic
```cpp
  /// Align for dynamic shared memory if any. Dynamic shared memory is
  /// allocated directly after the static one, i.e., LDSSize. Need to pad
  /// LDSSize to ensure that dynamic one is aligned accordingly.
  /// The maximal alignment is updated during IR translation or lowering
  /// stages.
  Align DynLDSAlign;

  // Flag to check dynamic LDS usage by kernel.
  bool UsesDynamicLDS = false;

  uint32_t NumNamedBarriers = 0;

  // Kernels + shaders. i.e. functions called by the hardware and not called
  // by other functions.
  bool IsEntryFunction = false;

  // Entry points called by other functions instead of directly by the hardware.
  bool IsModuleEntryFunction = false;

  // Functions with the amdgpu_cs_chain or amdgpu_cs_chain_preserve CC.
  bool IsChainFunction = false;

```
**EN:** This section advances the file implementation by combining declarations, control flow, and helper operations that support the AMDGPU backend.
**CN:** 本节通过组合声明、控制流与辅助操作来推进文件实现，支撑 AMDGPU 后端的整体功能。

### Lines 64-82: Declares AMDGPUMachineFunctionInfo
```cpp
  // Function may be memory bound.
  bool MemoryBound = false;

  // Kernel may need limited waves per EU for better performance.
  bool WaveLimiter = false;

  bool HasInitWholeWave = false;

public:
  AMDGPUMachineFunctionInfo(const Function &F, const AMDGPUSubtarget &ST);

  uint64_t getExplicitKernArgSize() const { return ExplicitKernArgSize; }

  Align getMaxKernArgAlign() const { return MaxKernArgAlign; }

  uint32_t getLDSSize() const { return LDSSize; }

  uint32_t getGDSSize() const { return GDSSize; }

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 83-104: Defines recordNumNamedBarriers
```cpp
  void recordNumNamedBarriers(uint32_t GVAddr, unsigned BarCnt) {
    NumNamedBarriers =
        std::max(NumNamedBarriers, ((GVAddr & 0x1ff) >> 4) + BarCnt - 1);
  }
  uint32_t getNumNamedBarriers() const { return NumNamedBarriers; }

  bool isEntryFunction() const { return IsEntryFunction; }

  bool isModuleEntryFunction() const { return IsModuleEntryFunction; }

  bool isChainFunction() const { return IsChainFunction; }

  // The stack is empty upon entry to this function.
  bool isBottomOfStack() const { return isEntryFunction(); }

  bool isMemoryBound() const { return MemoryBound; }

  bool needsWaveLimiter() const { return WaveLimiter; }

  bool hasInitWholeWave() const { return HasInitWholeWave; }
  void setInitWholeWave() { HasInitWholeWave = true; }

```
**EN:** This section contains concrete logic for recordNumNamedBarriers. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `std::max`.
**CN:** 本节包含与 recordNumNamedBarriers 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`std::max`。

### Lines 105-125: Preprocessor guards and macros
```cpp
  unsigned allocateLDSGlobal(const DataLayout &DL, const GlobalVariable &GV) {
    return allocateLDSGlobal(DL, GV, DynLDSAlign);
  }

  unsigned allocateLDSGlobal(const DataLayout &DL, const GlobalVariable &GV,
                             Align Trailing);

  static std::optional<uint32_t> getLDSKernelIdMetadata(const Function &F);
  static std::optional<uint32_t> getLDSAbsoluteAddress(const GlobalValue &GV);

  Align getDynLDSAlign() const { return DynLDSAlign; }

  void setDynLDSAlign(const Function &F, const GlobalVariable &GV);

  void setUsesDynamicLDS(bool DynLDS);

  bool isDynamicLDSUsed() const;
};

} // namespace llvm
#endif
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。

## Key Concepts / 关键概念
- **Language / 语言**: C++ header
- **Primary symbols / 主要符号**: `AMDGPUSubtarget`, `AMDGPUMachineFunctionInfo`, `std::max`
- **Main themes / 核心主题**: instruction semantics / 指令语义; subtarget modeling / 子目标建模; lowering / 降低; machine-function state / MachineFunction 状态
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"Utils/AMDGPUBaseInfo.h"`
- `"llvm/ADT/DenseMap.h"`
- `"llvm/CodeGen/MachineFunction.h"`
- `"llvm/IR/DataLayout.h"`
- `"llvm/IR/Function.h"`
- `"llvm/IR/GlobalValue.h"`
- `"llvm/IR/GlobalVariable.h"`
