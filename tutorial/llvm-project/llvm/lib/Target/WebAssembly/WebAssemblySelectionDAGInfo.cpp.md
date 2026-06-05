# WebAssemblySelectionDAGInfo.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblySelectionDAGInfo.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file implements the WebAssemblySelectionDAGInfo class.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblySelectionDAGInfo.cpp`，主要负责 WebAssembly 后端的后端实现逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- WebAssemblySelectionDAGInfo.cpp - WebAssembly SelectionDAG Info ---===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 7-11

```cpp
//===----------------------------------------------------------------------===//
///
/// \file
/// This file implements the WebAssemblySelectionDAGInfo class.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 12-21

```cpp
//===----------------------------------------------------------------------===//

#include "WebAssemblySelectionDAGInfo.h"
#include "WebAssemblyTargetMachine.h"

#define GET_SDNODE_DESC
#include "WebAssemblyGenSDNodeInfo.inc"

using namespace llvm;
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 22-59

```cpp
#define DEBUG_TYPE "wasm-selectiondag-info"

WebAssemblySelectionDAGInfo::WebAssemblySelectionDAGInfo()
    : SelectionDAGGenTargetInfo(WebAssemblyGenSDNodeInfo) {}

WebAssemblySelectionDAGInfo::~WebAssemblySelectionDAGInfo() = default; // anchor

const char *
WebAssemblySelectionDAGInfo::getTargetNodeName(unsigned Opcode) const {
  switch (static_cast<WebAssemblyISD::NodeType>(Opcode)) {
  case WebAssemblyISD::CALL:
    return "WebAssemblyISD::CALL";
  case WebAssemblyISD::RET_CALL:
    return "WebAssemblyISD::RET_CALL";
  }

  return SelectionDAGGenTargetInfo::getTargetNodeName(Opcode);
}

SDValue WebAssemblySelectionDAGInfo::EmitTargetCodeForMemcpy(
    SelectionDAG &DAG, const SDLoc &DL, SDValue Chain, SDValue Dst, SDValue Src,
    SDValue Size, Align Alignment, bool IsVolatile, bool AlwaysInline,
    MachinePointerInfo DstPtrInfo, MachinePointerInfo SrcPtrInfo) const {
  auto &ST = DAG.getMachineFunction().getSubtarget<WebAssemblySubtarget>();
  if (!ST.hasBulkMemoryOpt())
    return SDValue();

  SDValue MemIdx = DAG.getConstant(0, DL, MVT::i32);
  auto LenMVT = ST.hasAddr64() ? MVT::i64 : MVT::i32;

  // Use `MEMCPY` here instead of `MEMORY_COPY` because `memory.copy` traps
  // if the pointers are invalid even if the length is zero. `MEMCPY` gets
  // extra code to handle this in the way that LLVM IR expects.
  return DAG.getNode(
      WebAssemblyISD::MEMCPY, DL, MVT::Other,
      {Chain, MemIdx, MemIdx, Dst, Src, DAG.getZExtOrTrunc(Size, DL, LenMVT)});
}
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 60-88

```cpp
SDValue WebAssemblySelectionDAGInfo::EmitTargetCodeForMemmove(
    SelectionDAG &DAG, const SDLoc &DL, SDValue Chain, SDValue Op1, SDValue Op2,
    SDValue Op3, Align Alignment, bool IsVolatile,
    MachinePointerInfo DstPtrInfo, MachinePointerInfo SrcPtrInfo) const {
  return EmitTargetCodeForMemcpy(DAG, DL, Chain, Op1, Op2, Op3,
                                 Alignment, IsVolatile, false,
                                 DstPtrInfo, SrcPtrInfo);
}

SDValue WebAssemblySelectionDAGInfo::EmitTargetCodeForMemset(
    SelectionDAG &DAG, const SDLoc &DL, SDValue Chain, SDValue Dst, SDValue Val,
    SDValue Size, Align Alignment, bool IsVolatile, bool AlwaysInline,
    MachinePointerInfo DstPtrInfo) const {
  auto &ST = DAG.getMachineFunction().getSubtarget<WebAssemblySubtarget>();
  if (!ST.hasBulkMemoryOpt())
    return SDValue();

  SDValue MemIdx = DAG.getConstant(0, DL, MVT::i32);
  auto LenMVT = ST.hasAddr64() ? MVT::i64 : MVT::i32;

  // Use `MEMSET` here instead of `MEMORY_FILL` because `memory.fill` traps
  // if the pointers are invalid even if the length is zero. `MEMSET` gets
  // extra code to handle this in the way that LLVM IR expects.
  //
  // Only low byte matters for val argument, so anyext the i8
  return DAG.getNode(WebAssemblyISD::MEMSET, DL, MVT::Other, Chain, MemIdx, Dst,
                     DAG.getAnyExtOrTrunc(Val, DL, MVT::i32),
                     DAG.getZExtOrTrunc(Size, DL, LenMVT));
}
```
- **EN**: Implements helper routine(s) `EmitTargetCodeForMemmove`, `EmitTargetCodeForMemcpy`, `EmitTargetCodeForMemset` for this portion of the WebAssembly backend backend implementation logic. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `EmitTargetCodeForMemmove`, `EmitTargetCodeForMemcpy`, `EmitTargetCodeForMemset`。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Backend implementation logic / 后端实现逻辑
- SelectionDAG lowering / SelectionDAG lowering
- Subtarget features / 子目标特性
- Instruction selection or opcode handling / 指令选择或操作码处理
- Target machine configuration / 目标机器配置
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `WebAssemblySelectionDAGInfo.h`
- `WebAssemblyTargetMachine.h`
- `WebAssemblyGenSDNodeInfo.inc`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
