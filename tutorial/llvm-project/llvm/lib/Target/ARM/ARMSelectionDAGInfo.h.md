# ARMSelectionDAGInfo.h — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMSelectionDAGInfo.h`
- Repository: `llvm-project`
- Purpose (EN): This file defines the ARM subclass for SelectionDAGTargetInfo.
- 用途 (CN): 声明 ARM 后端中的 `ARMSelectionDAGInfo`，并提供与SelectionDAG 目标钩子相关的接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- ARMSelectionDAGInfo.h - ARM SelectionDAG Info -----------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the ARM subclass for SelectionDAGTargetInfo.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 13-14
```cpp
#ifndef LLVM_LIB_TARGET_ARM_ARMSELECTIONDAGINFO_H
#define LLVM_LIB_TARGET_ARM_ARMSELECTIONDAGINFO_H
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 16-18
```cpp
#include "MCTargetDesc/ARMAddressingModes.h"
#include "llvm/CodeGen/RuntimeLibcallUtil.h"
#include "llvm/CodeGen/SelectionDAGTargetInfo.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 20-21
```cpp
#define GET_SDNODE_ENUM
#include "ARMGenSDNodeInfo.inc"
```
- EN: Connects this file to TableGen-generated declarations so target-specific enums and helper tables become available to C++ code.
- CN: 这里把文件连接到 TableGen 生成的声明，使目标相关枚举和辅助表能够在 C++ 代码中使用。

### Lines 23-24
```cpp
namespace llvm {
namespace ARMISD {
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 26-27
```cpp
enum NodeType : unsigned {
  DYN_ALLOC = GENERATED_OPCODE_END, // Dynamic allocation on the stack.
```
- EN: Defines enumeration `NodeType` to name backend-specific modes, cases, or flags used elsewhere in the file.
- CN: 这里定义枚举 `NodeType`，用于命名本文件后续会使用的后端模式、分支情况或标志。

### Lines 29-31
```cpp
  MVESEXT,  // Legalization aids for extending a vector into two/four vectors.
  MVEZEXT,  //  or truncating two/four vectors into one. Eventually becomes
  MVETRUNC, //  stack store/load sequence, if not optimized to anything else.
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 33-38
```cpp
  // Operands of the standard BUILD_VECTOR node are not legalized, which
  // is fine if BUILD_VECTORs are always lowered to shuffles or other
  // operations, but for ARM some BUILD_VECTORs are legal as-is and their
  // operands need to be legalized.  Define an ARM-specific version of
  // BUILD_VECTOR for this purpose.
  BUILD_VECTOR,
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 40-45
```cpp
  // Vector load N-element structure to all lanes:
  FIRST_MEMORY_OPCODE,
  VLD1DUP = FIRST_MEMORY_OPCODE,
  VLD2DUP,
  VLD3DUP,
  VLD4DUP,
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 47-61
```cpp
  // NEON loads with post-increment base updates:
  VLD1_UPD,
  VLD2_UPD,
  VLD3_UPD,
  VLD4_UPD,
  VLD2LN_UPD,
  VLD3LN_UPD,
  VLD4LN_UPD,
  VLD1DUP_UPD,
  VLD2DUP_UPD,
  VLD3DUP_UPD,
  VLD4DUP_UPD,
  VLD1x2_UPD,
  VLD1x3_UPD,
  VLD1x4_UPD,
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 63-73
```cpp
  // NEON stores with post-increment base updates:
  VST1_UPD,
  VST3_UPD,
  VST2LN_UPD,
  VST3LN_UPD,
  VST4LN_UPD,
  VST1x2_UPD,
  VST1x3_UPD,
  VST1x4_UPD,
  LAST_MEMORY_OPCODE = VST1x4_UPD,
};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 75-75
```cpp
} // namespace ARMISD
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 77-91
```cpp
namespace ARM_AM {
  static inline ShiftOpc getShiftOpcForNode(unsigned Opcode) {
    switch (Opcode) {
    default:          return ARM_AM::no_shift;
    case ISD::SHL:    return ARM_AM::lsl;
    case ISD::SRL:    return ARM_AM::lsr;
    case ISD::SRA:    return ARM_AM::asr;
    case ISD::ROTR:   return ARM_AM::ror;
    //case ISD::ROTL:  // Only if imm -> turn into ROTR.
    // Can't handle RRX here, because it would require folding a flag into
    // the addressing mode.  :(  This causes us to miss certain things.
    //case ARMISD::RRX: return ARM_AM::rrx;
    }
  }
}  // end namespace ARM_AM
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 93-95
```cpp
class ARMSelectionDAGInfo : public SelectionDAGGenTargetInfo {
public:
  ARMSelectionDAGInfo();
```
- EN: Declares `ARMSelectionDAGInfo`, packaging target-specific state and APIs around `ARMSelectionDAGInfo`.
- CN: 这里声明 `ARMSelectionDAGInfo`，把与 `ARMSelectionDAGInfo` 相关的目标特定状态和 API 组织在一起。

### Lines 97-97
```cpp
  const char *getTargetNodeName(unsigned Opcode) const override;
```
- EN: Declares `getTargetNodeName`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getTargetNodeName`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 99-99
```cpp
  bool isTargetMemoryOpcode(unsigned Opcode) const override;
```
- EN: Declares `isTargetMemoryOpcode`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isTargetMemoryOpcode`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 101-102
```cpp
  void verifyTargetNode(const SelectionDAG &DAG,
                        const SDNode *N) const override;
```
- EN: Declares `verifyTargetNode`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `verifyTargetNode`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 104-109
```cpp
  SDValue EmitTargetCodeForMemcpy(SelectionDAG &DAG, const SDLoc &dl,
                                  SDValue Chain, SDValue Dst, SDValue Src,
                                  SDValue Size, Align Alignment,
                                  bool isVolatile, bool AlwaysInline,
                                  MachinePointerInfo DstPtrInfo,
                                  MachinePointerInfo SrcPtrInfo) const override;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 111-116
```cpp
  SDValue
  EmitTargetCodeForMemmove(SelectionDAG &DAG, const SDLoc &dl, SDValue Chain,
                           SDValue Dst, SDValue Src, SDValue Size,
                           Align Alignment, bool isVolatile,
                           MachinePointerInfo DstPtrInfo,
                           MachinePointerInfo SrcPtrInfo) const override;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 118-123
```cpp
  // Adjust parameters for memset, see RTABI section 4.3.4
  SDValue EmitTargetCodeForMemset(SelectionDAG &DAG, const SDLoc &dl,
                                  SDValue Chain, SDValue Op1, SDValue Op2,
                                  SDValue Op3, Align Alignment, bool isVolatile,
                                  bool AlwaysInline,
                                  MachinePointerInfo DstPtrInfo) const override;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 125-129
```cpp
  SDValue EmitSpecializedLibcall(SelectionDAG &DAG, const SDLoc &dl,
                                 SDValue Chain, SDValue Dst, SDValue Src,
                                 SDValue Size, unsigned Align,
                                 RTLIB::Libcall LC) const;
};
```
- EN: Declares `EmitSpecializedLibcall`, a emission/printing routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `EmitSpecializedLibcall`，它是一个围绕SelectionDAG 降级展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 131-131
```cpp
} // namespace llvm
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 133-133
```cpp
#endif
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

## Key Concepts / 关键概念

- EN: Primary role: SelectionDAG target hooks.
  - CN: 核心职责：SelectionDAG 目标钩子。
- EN: Stack-frame and spill/reload concerns are reflected in the API or implementation.
  - CN: API 或实现中体现了栈帧以及溢出/重载相关问题。

## Dependencies / 依赖关系

- EN: Backend-local headers: `MCTargetDesc/ARMAddressingModes.h`, `ARMGenSDNodeInfo.inc`.
  - CN: 后端本地头文件：`MCTargetDesc/ARMAddressingModes.h`, `ARMGenSDNodeInfo.inc`。
- EN: LLVM infrastructure headers: `llvm/CodeGen/RuntimeLibcallUtil.h`, `llvm/CodeGen/SelectionDAGTargetInfo.h`.
  - CN: LLVM 基础设施头文件：`llvm/CodeGen/RuntimeLibcallUtil.h`, `llvm/CodeGen/SelectionDAGTargetInfo.h`。
