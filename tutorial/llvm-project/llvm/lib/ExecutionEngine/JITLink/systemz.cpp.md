# systemz.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/JITLink/systemz.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements Generic JITLink systemz edge kinds, utilities.
  - **CN**: 实现面向即时编译代码的 JITLink 图链接、重定位处理以及特定文件格式的链接图 pass。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===---- systemz.cpp - Generic JITLink systemz edge kinds, utilities -----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 8-14
```cpp
//
// Generic utilities for graphs representing systemz objects.
//
//===----------------------------------------------------------------------===//

#include "llvm/ExecutionEngine/JITLink/systemz.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ExecutionEngine/JITLink/systemz.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ExecutionEngine/JITLink/systemz.h`。

### Lines 15-23
```cpp
#define DEBUG_TYPE "jitlink"

namespace llvm {
namespace jitlink {
namespace systemz {

const char NullPointerContent[8] = {0x00, 0x00, 0x00, 0x00,
                                    0x00, 0x00, 0x00, 0x00};

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 24-34
```cpp
const char Pointer64JumpStubContent[8] = {
    static_cast<char>(0xC4u),
    0x18,
    0x00,
    0x00,
    0x00,
    0x00, // lgrl r1
    static_cast<char>(0x07u),
    static_cast<char>(0xF1u), // BCR 15, 1
};

```
- **EN**: Implements logic around `static_cast<char>`.
- **CN**: 围绕 `static_cast<char>` 实现具体逻辑。

### Lines 35-48
```cpp
const char *getEdgeKindName(Edge::Kind R) {
  switch (R) {
  case Pointer64:
    return "Pointer64";
  case Pointer32:
    return "Pointer32";
  case Pointer20:
    return "Pointer20";
  case Pointer16:
    return "Pointer16";
  case Pointer12:
    return "Pointer12";
  case Pointer8:
    return "Pointer8";
```
- **EN**: Implements logic around `getEdgeKindName`; this block manipulates JITLink graph structures or link-time passes; uses `switch`-based dispatch over enums, opcodes, or kinds; returns subsystem-specific computed results.
- **CN**: 围绕 `getEdgeKindName` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，使用 `switch` 对枚举、opcode 或 kind 进行分派，返回子系统相关的计算结果。

### Lines 49-62
```cpp
  case Delta64:
    return "Delta64";
  case Delta32:
    return "Delta32";
  case Delta16:
    return "Delta16";
  case Delta32dbl:
    return "Delta32dbl";
  case Delta24dbl:
    return "Delta24dbl";
  case Delta16dbl:
    return "Delta16dbl";
  case Delta12dbl:
    return "Delta12dbl";
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 63-76
```cpp
  case NegDelta64:
    return "NegDelta64";
  case NegDelta32:
    return "NegDelta32";
  case DeltaPLT32dbl:
    return "DeltaPLT32dbl";
  case DeltaPLT24dbl:
    return "DeltaPLT24dbl";
  case DeltaPLT16dbl:
    return "DeltaPLT16dbl";
  case DeltaPLT12dbl:
    return "DeltaPLT12dbl";
  case DeltaPLT64:
    return "DeltaPLT64";
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 77-90
```cpp
  case DeltaPLT32:
    return "DeltaPLT32";
  case Delta64FromGOT:
    return "Delta64FromGOT";
  case Delta32FromGOT:
    return "Delta32FromGOT";
  case Delta16FromGOT:
    return "Delta16FromGOT";
  case Delta64PLTFromGOT:
    return "Delta64PLTFromGOT";
  case Delta32PLTFromGOT:
    return "Delta32PLTFromGOT";
  case Delta16PLTFromGOT:
    return "Delta16PLTFromGOT";
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 91-104
```cpp
  case Delta32GOTBase:
    return "Delta32GOTBase";
  case Delta32dblGOTBase:
    return "Delta32dblGOTBase";
  case RequestGOTAndTransformToDelta64FromGOT:
    return "RequestGOTAndTransformToDelta64FromGOT";
  case RequestGOTAndTransformToDelta32FromGOT:
    return "RequestGOTAndTransformToDelta32FromGOT";
  case RequestGOTAndTransformToDelta20FromGOT:
    return "RequestGOTAndTransformToDelta20FromGOT";
  case RequestGOTAndTransformToDelta16FromGOT:
    return "RequestGOTAndTransformToDelta16FromGOT";
  case RequestGOTAndTransformToDelta12FromGOT:
    return "RequestGOTAndTransformToDelta12FromGOT";
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 105-113
```cpp
  case RequestGOTAndTransformToDelta32dbl:
    return "RequestGOTAndTransformToDelta32dbl";
  case RequestTLSDescInGOTAndTransformToDelta64FromGOT:
    return "RequestTLSDescInGOTAndTransformToDelta64FromGOT";
  default:
    return getGenericEdgeKindName(static_cast<Edge::Kind>(R));
  }
}

```
- **EN**: Implements logic around `getGenericEdgeKindName`; this block manipulates JITLink graph structures or link-time passes; returns subsystem-specific computed results.
- **CN**: 围绕 `getGenericEdgeKindName` 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass，返回子系统相关的计算结果。

### Lines 114-127
```cpp
Error optimizeGOTAndStubAccesses(LinkGraph &G) {
  LLVM_DEBUG(dbgs() << "Optimizing GOT entries and stubs:\n");

  for (auto *B : G.blocks())
    for (auto &E : B->edges()) {
      if (E.getKind() == systemz::DeltaPLT32dbl) {
        auto &StubBlock = E.getTarget().getBlock();
        if (StubBlock.getSize() == sizeof(Pointer64JumpStubContent) &&
            StubBlock.edges_size() == 1) {
          auto &GOTBlock = StubBlock.edges().begin()->getTarget().getBlock();
          assert(GOTBlock.getSize() == G.getPointerSize() &&
                 "GOT block should be pointer sized");
          assert(GOTBlock.edges_size() == 1 &&
                 "GOT block should only have one outgoing edge");
```
- **EN**: Implements logic around `optimizeGOTAndStubAccesses`, `getTarget`, `edges_size`, `edges`, and 1 more symbols; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `optimizeGOTAndStubAccesses`, `getTarget`, `edges_size`, `edges`, and 1 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 128-141
```cpp

          auto &GOTTarget = GOTBlock.edges().begin()->getTarget();
          orc::ExecutorAddr EdgeAddr = B->getAddress() + E.getOffset();
          orc::ExecutorAddr TargetAddr = GOTTarget.getAddress();

          int64_t Displacement = TargetAddr + E.getAddend() - EdgeAddr;
          if (isInt<33>(Displacement)) {
            E.setKind(systemz::Delta32dbl);
            E.setTarget(GOTTarget);
            LLVM_DEBUG({
              dbgs() << "  Replaced stub branch with direct branch:\n    ";
              printEdge(dbgs(), *B, E, getEdgeKindName(E.getKind()));
              dbgs() << "\n";
            });
```
- **EN**: Implements logic around `edges`, `getAddress`, `getAddend`, `setKind`, and 3 more symbols; this block manipulates JITLink graph structures or link-time passes.
- **CN**: 围绕 `edges`, `getAddress`, `getAddend`, `setKind`, and 3 more symbols 实现具体逻辑；这一段操作 JITLink 图结构或链接阶段 pass。

### Lines 142-149
```cpp
          }
        }
      }
    }

  return Error::success();
}

```
- **EN**: Implements logic around `success`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `success` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 150-152
```cpp
} // namespace systemz
} // namespace jitlink
} // namespace llvm
```
- **EN**: Introduces declarations for `systemz`, `jitlink`, `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `systemz`, `jitlink`, `llvm` 等声明，定义本文件后续使用的数据结构或接口。

## Key Concepts / 关键概念

- **JITLink graph linking / JITLink 图链接**:
  - **EN**: Represents object code as graphs of blocks and edges so passes can rewrite relocations before final linking
  - **CN**: 把目标代码表示为块与边构成的图，使 pass 能在最终链接前重写重定位
- **Link graphs / 链接图**:
  - **EN**: Represents atoms of code/data and relocation edges explicitly for JIT-time rewriting
  - **CN**: 显式表示代码/数据原子及其重定位边，以便在 JIT 期重写
- **Debug/unwind metadata / 调试与展开元数据**:
  - **EN**: Emits or manages metadata needed for debugging and stack unwinding
  - **CN**: 输出或管理调试与栈展开所需的元数据

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/ExecutionEngine/JITLink/systemz.h`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine
