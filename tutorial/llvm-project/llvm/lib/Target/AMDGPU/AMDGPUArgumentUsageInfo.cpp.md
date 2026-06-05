# AMDGPUArgumentUsageInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUArgumentUsageInfo.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements AMDGPUArgumentUsageInfo for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 AMDGPUArgumentUsageInfo 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: File banner, includes, and setup
```cpp
//===----------------------------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "AMDGPUArgumentUsageInfo.h"
#include "AMDGPU.h"
#include "MCTargetDesc/AMDGPUMCTargetDesc.h"
#include "SIRegisterInfo.h"
#include "llvm/CodeGen/TargetRegisterInfo.h"
#include "llvm/Support/NativeFormatting.h"
#include "llvm/Support/raw_ostream.h"

using namespace llvm;

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 19-42: Implements ArgDescriptor::print
```cpp
void ArgDescriptor::print(raw_ostream &OS,
                          const TargetRegisterInfo *TRI) const {
  if (!isSet()) {
    OS << "<not set>\n";
    return;
  }

  if (isRegister())
    OS << "Reg " << printReg(getRegister(), TRI);
  else
    OS << "Stack offset " << getStackOffset();

  if (isMasked()) {
    OS << " & ";
    llvm::write_hex(OS, Mask, llvm::HexPrintStyle::PrefixLower);
  }

  OS << '\n';
}

// Hardcoded registers from fixed function ABI
const AMDGPUFunctionArgInfo AMDGPUFunctionArgInfo::FixedABIFunctionInfo =
    AMDGPUFunctionArgInfo::fixedABILayout();

```
**EN:** This section contains concrete logic for ArgDescriptor::print. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `ArgDescriptor::print`, `llvm::write_hex`, `AMDGPUFunctionArgInfo::fixedABILayout`.
**CN:** 本节包含与 ArgDescriptor::print 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`ArgDescriptor::print`, `llvm::write_hex`, `AMDGPUFunctionArgInfo::fixedABILayout`。

### Lines 43-66: Implements AMDGPUFunctionArgInfo::getPreloadedValue
```cpp
std::tuple<const ArgDescriptor *, const TargetRegisterClass *, LLT>
AMDGPUFunctionArgInfo::getPreloadedValue(
    AMDGPUFunctionArgInfo::PreloadedValue Value) const {
  switch (Value) {
  case AMDGPUFunctionArgInfo::PRIVATE_SEGMENT_BUFFER: {
    return std::tuple(PrivateSegmentBuffer ? &PrivateSegmentBuffer : nullptr,
                      &AMDGPU::SGPR_128RegClass, LLT::fixed_vector(4, 32));
  }
  case AMDGPUFunctionArgInfo::IMPLICIT_BUFFER_PTR:
    return std::tuple(ImplicitBufferPtr ? &ImplicitBufferPtr : nullptr,
                      &AMDGPU::SGPR_64RegClass,
                      LLT::pointer(AMDGPUAS::CONSTANT_ADDRESS, 64));
  case AMDGPUFunctionArgInfo::WORKGROUP_ID_X:
    return std::tuple(WorkGroupIDX ? &WorkGroupIDX : nullptr,
                      &AMDGPU::SGPR_32RegClass, LLT::scalar(32));
  case AMDGPUFunctionArgInfo::WORKGROUP_ID_Y:
    return std::tuple(WorkGroupIDY ? &WorkGroupIDY : nullptr,
                      &AMDGPU::SGPR_32RegClass, LLT::scalar(32));
  case AMDGPUFunctionArgInfo::WORKGROUP_ID_Z:
    return std::tuple(WorkGroupIDZ ? &WorkGroupIDZ : nullptr,
                      &AMDGPU::SGPR_32RegClass, LLT::scalar(32));
  case AMDGPUFunctionArgInfo::CLUSTER_WORKGROUP_ID_X:
  case AMDGPUFunctionArgInfo::CLUSTER_WORKGROUP_ID_Y:
  case AMDGPUFunctionArgInfo::CLUSTER_WORKGROUP_ID_Z:
```
**EN:** This section contains concrete logic for AMDGPUFunctionArgInfo::getPreloadedValue. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUFunctionArgInfo::getPreloadedValue`, `std::tuple`, `LLT::fixed_vector`.
**CN:** 本节包含与 AMDGPUFunctionArgInfo::getPreloadedValue 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUFunctionArgInfo::getPreloadedValue`, `std::tuple`, `LLT::fixed_vector`。

### Lines 67-90: Result computation and returns
```cpp
  case AMDGPUFunctionArgInfo::CLUSTER_WORKGROUP_MAX_ID_X:
  case AMDGPUFunctionArgInfo::CLUSTER_WORKGROUP_MAX_ID_Y:
  case AMDGPUFunctionArgInfo::CLUSTER_WORKGROUP_MAX_ID_Z:
  case AMDGPUFunctionArgInfo::CLUSTER_WORKGROUP_MAX_FLAT_ID:
    return std::tuple(nullptr, &AMDGPU::SGPR_32RegClass, LLT::scalar(32));
  case AMDGPUFunctionArgInfo::LDS_KERNEL_ID:
    return std::tuple(LDSKernelId ? &LDSKernelId : nullptr,
                      &AMDGPU::SGPR_32RegClass, LLT::scalar(32));
  case AMDGPUFunctionArgInfo::PRIVATE_SEGMENT_WAVE_BYTE_OFFSET:
    return std::tuple(
        PrivateSegmentWaveByteOffset ? &PrivateSegmentWaveByteOffset : nullptr,
        &AMDGPU::SGPR_32RegClass, LLT::scalar(32));
  case AMDGPUFunctionArgInfo::PRIVATE_SEGMENT_SIZE:
    return {PrivateSegmentSize ? &PrivateSegmentSize : nullptr,
            &AMDGPU::SGPR_32RegClass, LLT::scalar(32)};
  case AMDGPUFunctionArgInfo::KERNARG_SEGMENT_PTR:
    return std::tuple(KernargSegmentPtr ? &KernargSegmentPtr : nullptr,
                      &AMDGPU::SGPR_64RegClass,
                      LLT::pointer(AMDGPUAS::CONSTANT_ADDRESS, 64));
  case AMDGPUFunctionArgInfo::IMPLICIT_ARG_PTR:
    return std::tuple(ImplicitArgPtr ? &ImplicitArgPtr : nullptr,
                      &AMDGPU::SGPR_64RegClass,
                      LLT::pointer(AMDGPUAS::CONSTANT_ADDRESS, 64));
  case AMDGPUFunctionArgInfo::DISPATCH_ID:
```
**EN:** This section finalizes intermediate state and returns the value that the caller or pass pipeline needs. Main symbols: `std::tuple`, `LLT::scalar`, `LLT::pointer`.
**CN:** 本节整理中间状态并返回调用者或 Pass 流水线所需的结果。 主要符号：`std::tuple`, `LLT::scalar`, `LLT::pointer`。

### Lines 91-114: Result computation and returns
```cpp
    return std::tuple(DispatchID ? &DispatchID : nullptr,
                      &AMDGPU::SGPR_64RegClass, LLT::scalar(64));
  case AMDGPUFunctionArgInfo::FLAT_SCRATCH_INIT:
    return std::tuple(FlatScratchInit ? &FlatScratchInit : nullptr,
                      &AMDGPU::SGPR_64RegClass, LLT::scalar(64));
  case AMDGPUFunctionArgInfo::DISPATCH_PTR:
    return std::tuple(DispatchPtr ? &DispatchPtr : nullptr,
                      &AMDGPU::SGPR_64RegClass,
                      LLT::pointer(AMDGPUAS::CONSTANT_ADDRESS, 64));
  case AMDGPUFunctionArgInfo::QUEUE_PTR:
    return std::tuple(QueuePtr ? &QueuePtr : nullptr, &AMDGPU::SGPR_64RegClass,
                      LLT::pointer(AMDGPUAS::CONSTANT_ADDRESS, 64));
  case AMDGPUFunctionArgInfo::WORKITEM_ID_X:
    return std::tuple(WorkItemIDX ? &WorkItemIDX : nullptr,
                      &AMDGPU::VGPR_32RegClass, LLT::scalar(32));
  case AMDGPUFunctionArgInfo::WORKITEM_ID_Y:
    return std::tuple(WorkItemIDY ? &WorkItemIDY : nullptr,
                      &AMDGPU::VGPR_32RegClass, LLT::scalar(32));
  case AMDGPUFunctionArgInfo::WORKITEM_ID_Z:
    return std::tuple(WorkItemIDZ ? &WorkItemIDZ : nullptr,
                      &AMDGPU::VGPR_32RegClass, LLT::scalar(32));
  }
  llvm_unreachable("unexpected preloaded value type");
}
```
**EN:** This section finalizes intermediate state and returns the value that the caller or pass pipeline needs. Main symbols: `std::tuple`, `LLT::scalar`, `LLT::pointer`.
**CN:** 本节整理中间状态并返回调用者或 Pass 流水线所需的结果。 主要符号：`std::tuple`, `LLT::scalar`, `LLT::pointer`。

### Lines 115-133: Implements AMDGPUFunctionArgInfo::fixedABILayout
```cpp

AMDGPUFunctionArgInfo AMDGPUFunctionArgInfo::fixedABILayout() {
  AMDGPUFunctionArgInfo AI;
  AI.PrivateSegmentBuffer
    = ArgDescriptor::createRegister(AMDGPU::SGPR0_SGPR1_SGPR2_SGPR3);
  AI.DispatchPtr = ArgDescriptor::createRegister(AMDGPU::SGPR4_SGPR5);
  AI.QueuePtr = ArgDescriptor::createRegister(AMDGPU::SGPR6_SGPR7);

  // Do not pass kernarg segment pointer, only pass increment version in its
  // place.
  AI.ImplicitArgPtr = ArgDescriptor::createRegister(AMDGPU::SGPR8_SGPR9);
  AI.DispatchID = ArgDescriptor::createRegister(AMDGPU::SGPR10_SGPR11);

  // Skip FlatScratchInit/PrivateSegmentSize
  AI.WorkGroupIDX = ArgDescriptor::createRegister(AMDGPU::SGPR12);
  AI.WorkGroupIDY = ArgDescriptor::createRegister(AMDGPU::SGPR13);
  AI.WorkGroupIDZ = ArgDescriptor::createRegister(AMDGPU::SGPR14);
  AI.LDSKernelId = ArgDescriptor::createRegister(AMDGPU::SGPR15);

```
**EN:** This section contains concrete logic for AMDGPUFunctionArgInfo::fixedABILayout. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUFunctionArgInfo::fixedABILayout`, `ArgDescriptor::createRegister`.
**CN:** 本节包含与 AMDGPUFunctionArgInfo::fixedABILayout 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUFunctionArgInfo::fixedABILayout`, `ArgDescriptor::createRegister`。

### Lines 134-139: Implements ArgDescriptor::createRegister
```cpp
  const unsigned Mask = 0x3ff;
  AI.WorkItemIDX = ArgDescriptor::createRegister(AMDGPU::VGPR31, Mask);
  AI.WorkItemIDY = ArgDescriptor::createRegister(AMDGPU::VGPR31, Mask << 10);
  AI.WorkItemIDZ = ArgDescriptor::createRegister(AMDGPU::VGPR31, Mask << 20);
  return AI;
}
```
**EN:** This section contains concrete logic for ArgDescriptor::createRegister. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `ArgDescriptor::createRegister`.
**CN:** 本节包含与 ArgDescriptor::createRegister 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`ArgDescriptor::createRegister`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `ArgDescriptor::print`, `llvm::write_hex`, `AMDGPUFunctionArgInfo::fixedABILayout`, `AMDGPUFunctionArgInfo::getPreloadedValue`, `std::tuple`, `LLT::fixed_vector`
- **Main themes / 核心主题**: register management / 寄存器管理; lowering / 降低
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"AMDGPUArgumentUsageInfo.h"`
- `"AMDGPU.h"`
- `"MCTargetDesc/AMDGPUMCTargetDesc.h"`
- `"SIRegisterInfo.h"`
- `"llvm/CodeGen/TargetRegisterInfo.h"`
- `"llvm/Support/NativeFormatting.h"`
- `"llvm/Support/raw_ostream.h"`
