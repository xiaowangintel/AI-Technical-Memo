# SIProgramInfo.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/SIProgramInfo.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This header declares the interfaces, data structures, and pass entry points for SIProgramInfo in the LLVM AMDGPU backend. It defines the contracts consumed by other AMDGPU backend components. / 该头文件声明 LLVM AMDGPU 后端中 SIProgramInfo 的接口、数据结构与 Pass 入口。它定义了其他 AMDGPU 后端组件依赖的契约。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: File banner, includes, and setup
```cpp
//===--- SIProgramInfo.h ----------------------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// Defines struct to track resource usage and hardware flags for kernels and
/// entry functions.
///
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AMDGPU_SIPROGRAMINFO_H
#define LLVM_LIB_TARGET_AMDGPU_SIPROGRAMINFO_H

#include "llvm/IR/CallingConv.h"
#include "llvm/Support/Compiler.h"
#include <cstdint>
#include <optional>

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 24-47: Declares class GCNSubtarget
```cpp
namespace llvm {

class GCNSubtarget;
class MCContext;
class MCExpr;
class MachineFunction;

/// Track resource usage for kernels / entry functions.
struct LLVM_EXTERNAL_VISIBILITY SIProgramInfo {
  std::optional<uint64_t> CodeSizeInBytes;

  // Fields set in PGM_RSRC1 pm4 packet.
  const MCExpr *VGPRBlocks = nullptr;
  const MCExpr *SGPRBlocks = nullptr;
  uint32_t Priority = 0;
  uint32_t FloatMode = 0;
  uint32_t Priv = 0;
  uint32_t DX10Clamp = 0;
  uint32_t DebugMode = 0;
  uint32_t IEEEMode = 0;
  uint32_t WgpMode = 0;     // GFX10+
  uint32_t MemOrdered = 0;  // GFX10+
  uint32_t FwdProgress = 0; // GFX10+
  uint32_t RrWgMode = 0;    // GFX12+
```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `GCNSubtarget`, `MCContext`, `MCExpr`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`GCNSubtarget`, `MCContext`, `MCExpr`。

### Lines 48-68: Implementation details and local logic
```cpp
  const MCExpr *ScratchSize = nullptr;

  // State used to calculate fields set in PGM_RSRC2 pm4 packet.
  uint32_t LDSBlocks = 0;
  const MCExpr *ScratchBlocks = nullptr;

  // Fields set in PGM_RSRC2 pm4 packet
  const MCExpr *ScratchEnable = nullptr;
  uint32_t UserSGPR = 0;
  uint32_t TrapHandlerEnable = 0;
  uint32_t TGIdXEnable = 0;
  uint32_t TGIdYEnable = 0;
  uint32_t TGIdZEnable = 0;
  uint32_t TGSizeEnable = 0;
  uint32_t TIdIGCompCount = 0;
  uint32_t EXCPEnMSB = 0;
  uint32_t LdsSize = 0;
  uint32_t EXCPEnable = 0;

  const MCExpr *ComputePGMRSrc3 = nullptr;

```
**EN:** This section advances the file implementation by combining declarations, control flow, and helper operations that support the AMDGPU backend.
**CN:** 本节通过组合声明、控制流与辅助操作来推进文件实现，支撑 AMDGPU 后端的整体功能。

### Lines 69-91: Implementation details and local logic
```cpp
  const MCExpr *NumVGPR = nullptr;
  const MCExpr *NumArchVGPR = nullptr;
  const MCExpr *NumAccVGPR = nullptr;
  const MCExpr *AccumOffset = nullptr;
  uint32_t TgSplit = 0;
  const MCExpr *NumSGPR = nullptr;
  unsigned SGPRSpill = 0;
  unsigned VGPRSpill = 0;
  uint32_t LDSSize = 0;
  const MCExpr *FlatUsed = nullptr;

  // Number of SGPRs that meets number of waves per execution unit request.
  const MCExpr *NumSGPRsForWavesPerEU = nullptr;

  // Number of VGPRs that meets number of waves per execution unit request.
  const MCExpr *NumVGPRsForWavesPerEU = nullptr;

  // Number of named barriers used by the kernel.
  const MCExpr *NamedBarCnt = nullptr;

  // Final occupancy.
  const MCExpr *Occupancy = nullptr;

```
**EN:** This section advances the file implementation by combining declarations, control flow, and helper operations that support the AMDGPU backend.
**CN:** 本节通过组合声明、控制流与辅助操作来推进文件实现，支撑 AMDGPU 后端的整体功能。

### Lines 92-112: Declares SIProgramInfo
```cpp
  // Whether there is recursion, dynamic allocas, indirect calls or some other
  // reason there may be statically unknown stack usage.
  const MCExpr *DynamicCallStack = nullptr;

  // Bonus information for debugging.
  const MCExpr *VCCUsed = nullptr;

  SIProgramInfo() = default;

  // The constructor sets the values for each member as shown in the struct.
  // However, setting the MCExpr members to their zero value equivalent
  // happens in reset together with (duplicated) value re-set for the
  // non-MCExpr members.
  void reset(const MachineFunction &MF);

  // Get function code size and cache the value.
  // If \p IsLowerBound is set it returns a minimal code size which is safe
  // to address.
  uint64_t getFunctionCodeSize(const MachineFunction &MF,
                               bool IsLowerBound = false);

```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。

### Lines 113-128: Preprocessor guards and macros
```cpp
  /// Compute the value of the ComputePGMRsrc1 register.
  const MCExpr *getComputePGMRSrc1(const GCNSubtarget &ST,
                                   MCContext &Ctx) const;
  const MCExpr *getPGMRSrc1(CallingConv::ID CC, const GCNSubtarget &ST,
                            MCContext &Ctx) const;

  /// Compute the value of the ComputePGMRsrc2 register.
  const MCExpr *getComputePGMRSrc2(const GCNSubtarget &ST,
                                   MCContext &Ctx) const;
  const MCExpr *getPGMRSrc2(CallingConv::ID CC, const GCNSubtarget &ST,
                            MCContext &Ctx) const;
};

} // namespace llvm

#endif // LLVM_LIB_TARGET_AMDGPU_SIPROGRAMINFO_H
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。

## Key Concepts / 关键概念
- **Language / 语言**: C++ header
- **Primary symbols / 主要符号**: `GCNSubtarget`, `MCContext`, `MCExpr`, `MachineFunction`, `LLVM_EXTERNAL_VISIBILITY`
- **Main themes / 核心主题**: register management / 寄存器管理; subtarget modeling / 子目标建模; lowering / 降低; machine-function state / MachineFunction 状态
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"llvm/IR/CallingConv.h"`
- `"llvm/Support/Compiler.h"`
- `<cstdint>`
- `<optional>`
