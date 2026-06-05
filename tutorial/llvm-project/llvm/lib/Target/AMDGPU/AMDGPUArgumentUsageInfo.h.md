# AMDGPUArgumentUsageInfo.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUArgumentUsageInfo.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This header declares the interfaces, data structures, and pass entry points for AMDGPUArgumentUsageInfo in the LLVM AMDGPU backend. It defines the contracts consumed by other AMDGPU backend components. / 该头文件声明 LLVM AMDGPU 后端中 AMDGPUArgumentUsageInfo 的接口、数据结构与 Pass 入口。它定义了其他 AMDGPU 后端组件依赖的契约。

## Line-by-Line Analysis / 逐行分析
### Lines 1-22: File banner, includes, and setup
```cpp
//==- AMDGPUArgumentrUsageInfo.h - Function Arg Usage Info -------*- C++ -*-==//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AMDGPU_AMDGPUARGUMENTUSAGEINFO_H
#define LLVM_LIB_TARGET_AMDGPU_AMDGPUARGUMENTUSAGEINFO_H

#include "MCTargetDesc/AMDGPUMCTargetDesc.h"
#include "llvm/CodeGen/Register.h"
#include <variant>

namespace llvm {

class LLT;
class raw_ostream;
class TargetRegisterClass;
class TargetRegisterInfo;

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `LLT`, `raw_ostream`, `TargetRegisterClass`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`LLT`, `raw_ostream`, `TargetRegisterClass`。

### Lines 23-46: Declares struct ArgDescriptor
```cpp
struct ArgDescriptor {
private:
  friend struct AMDGPUFunctionArgInfo;

  std::variant<std::monostate, MCRegister, unsigned> Val;

  // Bitmask to locate argument within the register.
  unsigned Mask;

public:
  ArgDescriptor(unsigned Mask = ~0u) : Mask(Mask) {}

  static ArgDescriptor createRegister(Register Reg, unsigned Mask = ~0u) {
    ArgDescriptor Ret(Mask);
    Ret.Val = Reg.asMCReg();
    return Ret;
  }

  static ArgDescriptor createStack(unsigned Offset, unsigned Mask = ~0u) {
    ArgDescriptor Ret(Mask);
    Ret.Val = Offset;
    return Ret;
  }

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `ArgDescriptor`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`ArgDescriptor`。

### Lines 47-65: Defines createArg
```cpp
  static ArgDescriptor createArg(const ArgDescriptor &Arg, unsigned Mask) {
    // Copy the descriptor, then change the mask.
    ArgDescriptor Ret(Arg);
    Ret.Mask = Mask;
    return Ret;
  }

  bool isSet() const { return !std::holds_alternative<std::monostate>(Val); }

  explicit operator bool() const {
    return isSet();
  }

  bool isRegister() const { return std::holds_alternative<MCRegister>(Val); }

  MCRegister getRegister() const { return std::get<MCRegister>(Val); }

  unsigned getStackOffset() const { return std::get<unsigned>(Val); }

```
**EN:** This section contains concrete logic for createArg. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 createArg 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 66-88: Declares struct KernArgPreloadDescriptor
```cpp
  unsigned getMask() const {
    // None of the target SGPRs or VGPRs are expected to have a 'zero' mask.
    assert(Mask && "Invalid mask.");
    return Mask;
  }

  bool isMasked() const {
    return Mask != ~0u;
  }

  void print(raw_ostream &OS, const TargetRegisterInfo *TRI = nullptr) const;
};

inline raw_ostream &operator<<(raw_ostream &OS, const ArgDescriptor &Arg) {
  Arg.print(OS);
  return OS;
}

struct KernArgPreloadDescriptor : public ArgDescriptor {
  KernArgPreloadDescriptor() = default;
  SmallVector<MCRegister> Regs;
};

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `KernArgPreloadDescriptor`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`KernArgPreloadDescriptor`。

### Lines 89-112: Declares struct AMDGPUFunctionArgInfo
```cpp
struct AMDGPUFunctionArgInfo {
  // clang-format off
  enum PreloadedValue {
    // SGPRS:
    PRIVATE_SEGMENT_BUFFER = 0,
    DISPATCH_PTR        =  1,
    QUEUE_PTR           =  2,
    KERNARG_SEGMENT_PTR =  3,
    DISPATCH_ID         =  4,
    FLAT_SCRATCH_INIT   =  5,
    LDS_KERNEL_ID       =  6, // LLVM internal, not part of the ABI
    WORKGROUP_ID_X      = 10, // Also used for cluster ID X.
    WORKGROUP_ID_Y      = 11, // Also used for cluster ID Y.
    WORKGROUP_ID_Z      = 12, // Also used for cluster ID Z.
    PRIVATE_SEGMENT_WAVE_BYTE_OFFSET = 14,
    IMPLICIT_BUFFER_PTR = 15,
    IMPLICIT_ARG_PTR = 16,
    PRIVATE_SEGMENT_SIZE = 17,
    CLUSTER_WORKGROUP_ID_X = 21,
    CLUSTER_WORKGROUP_ID_Y = 22,
    CLUSTER_WORKGROUP_ID_Z = 23,
    CLUSTER_WORKGROUP_MAX_ID_X = 24,
    CLUSTER_WORKGROUP_MAX_ID_Y = 25,
    CLUSTER_WORKGROUP_MAX_ID_Z = 26,
```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `AMDGPUFunctionArgInfo`, `PreloadedValue`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`AMDGPUFunctionArgInfo`, `PreloadedValue`。

### Lines 113-135: Implementation details and local logic
```cpp
    CLUSTER_WORKGROUP_MAX_FLAT_ID = 27,

    // VGPRS:
    WORKITEM_ID_X       = 28,
    WORKITEM_ID_Y       = 29,
    WORKITEM_ID_Z       = 30,
    FIRST_VGPR_VALUE    = WORKITEM_ID_X
  };
  // clang-format on

  // Kernel input registers setup for the HSA ABI in allocation order.

  // User SGPRs in kernels
  // XXX - Can these require argument spills?
  ArgDescriptor PrivateSegmentBuffer;
  ArgDescriptor DispatchPtr;
  ArgDescriptor QueuePtr;
  ArgDescriptor KernargSegmentPtr;
  ArgDescriptor DispatchID;
  ArgDescriptor FlatScratchInit;
  ArgDescriptor PrivateSegmentSize;
  ArgDescriptor LDSKernelId;

```
**EN:** This section advances the file implementation by combining declarations, control flow, and helper operations that support the AMDGPU backend.
**CN:** 本节通过组合声明、控制流与辅助操作来推进文件实现，支撑 AMDGPU 后端的整体功能。

### Lines 136-155: Implementation details and local logic
```cpp
  // System SGPRs in kernels.
  ArgDescriptor WorkGroupIDX;
  ArgDescriptor WorkGroupIDY;
  ArgDescriptor WorkGroupIDZ;
  ArgDescriptor WorkGroupInfo;
  ArgDescriptor PrivateSegmentWaveByteOffset;

  // Pointer with offset from kernargsegmentptr to where special ABI arguments
  // are passed to callable functions.
  ArgDescriptor ImplicitArgPtr;

  // Input registers for non-HSA ABI
  ArgDescriptor ImplicitBufferPtr;

  // VGPRs inputs. For entry functions these are either v0, v1 and v2 or packed
  // into v0, 10 bits per dimension if packed-tid is set.
  ArgDescriptor WorkItemIDX;
  ArgDescriptor WorkItemIDY;
  ArgDescriptor WorkItemIDZ;

```
**EN:** This section advances the file implementation by combining declarations, control flow, and helper operations that support the AMDGPU backend.
**CN:** 本节通过组合声明、控制流与辅助操作来推进文件实现，支撑 AMDGPU 后端的整体功能。

### Lines 156-170: Preprocessor guards and macros
```cpp
  // Map the index of preloaded kernel arguments to its descriptor.
  SmallDenseMap<int, KernArgPreloadDescriptor> PreloadKernArgs{};
  // The first user SGPR allocated for kernarg preloading.
  Register FirstKernArgPreloadReg;

  std::tuple<const ArgDescriptor *, const TargetRegisterClass *, LLT>
  getPreloadedValue(PreloadedValue Value) const;

  static AMDGPUFunctionArgInfo fixedABILayout();
  static const AMDGPUFunctionArgInfo FixedABIFunctionInfo;
};

} // end namespace llvm

#endif
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。

## Key Concepts / 关键概念
- **Language / 语言**: C++ header
- **Primary symbols / 主要符号**: `LLT`, `raw_ostream`, `TargetRegisterClass`, `TargetRegisterInfo`, `ArgDescriptor`, `KernArgPreloadDescriptor`
- **Main themes / 核心主题**: register management / 寄存器管理; assembly handling / 汇编处理; LLVM pass integration / LLVM Pass 集成
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"MCTargetDesc/AMDGPUMCTargetDesc.h"`
- `"llvm/CodeGen/Register.h"`
- `<variant>`
