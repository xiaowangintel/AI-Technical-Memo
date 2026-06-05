# AMDGPUMCResourceInfo.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUMCResourceInfo.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This header declares the interfaces, data structures, and pass entry points for AMDGPUMCResourceInfo in the LLVM AMDGPU backend. It defines the contracts consumed by other AMDGPU backend components. / 该头文件声明 LLVM AMDGPU 后端中 AMDGPUMCResourceInfo 的接口、数据结构与 Pass 入口。它定义了其他 AMDGPU 后端组件依赖的契约。

## Line-by-Line Analysis / 逐行分析
### Lines 1-22: File banner, includes, and setup
```cpp
//===- AMDGPUMCResourceInfo.h ----- MC Resource Info --------------*- C++ -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// \brief MC infrastructure to propagate the function level resource usage
/// info.
///
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AMDGPU_AMDGPUMCRESOURCEINFO_H
#define LLVM_LIB_TARGET_AMDGPU_AMDGPUMCRESOURCEINFO_H

#include "AMDGPUResourceUsageAnalysis.h"
#include "MCTargetDesc/AMDGPUMCExpr.h"

namespace llvm {

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 23-42: Declares class MCContext
```cpp
class MCContext;
class MCSymbol;
class StringRef;
class MachineFunction;

class MCResourceInfo {
public:
  enum ResourceInfoKind {
    RIK_NumVGPR,
    RIK_NumAGPR,
    RIK_NumSGPR,
    RIK_NumNamedBarrier,
    RIK_PrivateSegSize,
    RIK_UsesVCC,
    RIK_UsesFlatScratch,
    RIK_HasDynSizedStack,
    RIK_HasRecursion,
    RIK_HasIndirectCall
  };

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `MCContext`, `MCSymbol`, `StringRef`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`MCContext`, `MCSymbol`, `StringRef`。

### Lines 43-62: Defines assignResourceInfoExpr
```cpp
private:
  int32_t MaxVGPR = 0;
  int32_t MaxAGPR = 0;
  int32_t MaxSGPR = 0;
  int32_t MaxNamedBarrier = 0;

  // Whether the MCResourceInfo has been finalized through finalize(MCContext
  // &). Should only be called once, at the end of AsmPrinting to assign MaxXGPR
  // symbols to their final value.
  bool Finalized = false;

  void assignResourceInfoExpr(int64_t localValue, ResourceInfoKind RIK,
                              AMDGPUMCExpr::VariantKind Kind,
                              const MachineFunction &MF,
                              const SmallVectorImpl<const Function *> &Callees,
                              MCContext &OutContext);

  // Assigns expression for Max S/V/A-GPRs to the referenced symbols.
  void assignMaxRegs(MCContext &OutContext);

```
**EN:** This section contains concrete logic for assignResourceInfoExpr. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 assignResourceInfoExpr 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 63-83: Defines flattenedCycleMax
```cpp
  // Take flattened max of cyclic function calls' knowns. For example, for
  // a cycle A->B->C->D->A, take max(A, B, C, D) for A and have B, C, D have the
  // propgated value from A.
  const MCExpr *flattenedCycleMax(MCSymbol *RecSym, ResourceInfoKind RIK,
                                  MCContext &OutContext);

public:
  MCResourceInfo() = default;
  void addMaxVGPRCandidate(int32_t candidate) {
    MaxVGPR = std::max(MaxVGPR, candidate);
  }
  void addMaxAGPRCandidate(int32_t candidate) {
    MaxAGPR = std::max(MaxAGPR, candidate);
  }
  void addMaxSGPRCandidate(int32_t candidate) {
    MaxSGPR = std::max(MaxSGPR, candidate);
  }
  void addMaxNamedBarrierCandidate(int32_t candidate) {
    MaxNamedBarrier = std::max(MaxNamedBarrier, candidate);
  }

```
**EN:** This section contains concrete logic for flattenedCycleMax. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `std::max`.
**CN:** 本节包含与 flattenedCycleMax 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`std::max`。

### Lines 84-99: Defines getSymbol
```cpp
  MCSymbol *getSymbol(StringRef FuncName, ResourceInfoKind RIK,
                      MCContext &OutContext);
  const MCExpr *getSymRefExpr(StringRef FuncName, ResourceInfoKind RIK,
                              MCContext &Ctx);

  void reset();

  // Resolves the final symbols that requires the inter-function resource info
  // to be resolved.
  void finalize(MCContext &OutContext);

  MCSymbol *getMaxVGPRSymbol(MCContext &OutContext);
  MCSymbol *getMaxAGPRSymbol(MCContext &OutContext);
  MCSymbol *getMaxSGPRSymbol(MCContext &OutContext);
  MCSymbol *getMaxNamedBarrierSymbol(MCContext &OutContext);

```
**EN:** This section contains concrete logic for getSymbol. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 getSymbol 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 100-120: Preprocessor guards and macros
```cpp
  /// AMDGPUResourceUsageAnalysis gathers resource usage on a per-function
  /// granularity. However, some resource info has to be assigned the call
  /// transitive maximum or accumulative. For example, if A calls B and B's VGPR
  /// usage exceeds A's, A should be assigned B's VGPR usage. Furthermore,
  /// functions with indirect calls should be assigned the module level maximum.
  ///
  /// When link-time object linking is enabled, skip all call-transitive
  /// propagation and emit concrete per-function values for every resource
  /// symbol. Cross-TU aggregation is then the linker's responsibility.
  void gatherResourceInfo(
      const MachineFunction &MF,
      const AMDGPUResourceUsageAnalysisWrapperPass::FunctionResourceInfo &FRI,
      MCContext &OutContext);

  const MCExpr *createTotalNumVGPRs(const MachineFunction &MF, MCContext &Ctx);
  const MCExpr *createTotalNumSGPRs(const MachineFunction &MF, bool hasXnack,
                                    MCContext &Ctx);
};
} // namespace llvm

#endif // LLVM_LIB_TARGET_AMDGPU_AMDGPUMCRESOURCEINFO_H
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。

## Key Concepts / 关键概念
- **Language / 语言**: C++ header
- **Primary symbols / 主要符号**: `MCContext`, `MCSymbol`, `StringRef`, `MachineFunction`, `MCResourceInfo`, `ResourceInfoKind`
- **Main themes / 核心主题**: assembly handling / 汇编处理; LLVM pass integration / LLVM Pass 集成; machine-function state / MachineFunction 状态; hardware resource modeling / 硬件资源建模
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPUResourceUsageAnalysis.h"`
- `"MCTargetDesc/AMDGPUMCExpr.h"`
