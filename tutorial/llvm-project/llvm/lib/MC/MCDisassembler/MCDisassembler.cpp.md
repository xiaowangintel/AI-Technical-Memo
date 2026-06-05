# MCDisassembler.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/MC/MCDisassembler/MCDisassembler.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements Disassembler interface.
  - **CN**: 实现 LLVM MC 反汇编支持，把机器码字节解码为 MC 指令与符号化操作数。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- MCDisassembler.cpp - Disassembler interface ------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 8-13
```cpp

#include "llvm/MC/MCDisassembler/MCDisassembler.h"
#include "llvm/ADT/ArrayRef.h"

using namespace llvm;

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/MC/MCDisassembler/MCDisassembler.h`, `llvm/ADT/ArrayRef.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/MC/MCDisassembler/MCDisassembler.h`, `llvm/ADT/ArrayRef.h`。

### Lines 14-22
```cpp
MCDisassembler::~MCDisassembler() = default;

Expected<bool> MCDisassembler::onSymbolStart(SymbolInfoTy &Symbol,
                                             uint64_t &Size,
                                             ArrayRef<uint8_t> Bytes,
                                             uint64_t Address) const {
  return false;
}

```
- **EN**: Implements logic around `~MCDisassembler`, `onSymbolStart`; this block decodes machine-code bytes into symbolic instruction form; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `~MCDisassembler`, `onSymbolStart` 实现具体逻辑；这一段把机器码字节解码为符号化指令形式，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 23-27
```cpp
uint64_t MCDisassembler::suggestBytesToSkip(ArrayRef<uint8_t> Bytes,
                                            uint64_t Address) const {
  return 1;
}

```
- **EN**: Implements logic around `suggestBytesToSkip`; this block decodes machine-code bytes into symbolic instruction form; returns subsystem-specific computed results.
- **CN**: 围绕 `suggestBytesToSkip` 实现具体逻辑；这一段把机器码字节解码为符号化指令形式，返回子系统相关的计算结果。

### Lines 28-37
```cpp
bool MCDisassembler::tryAddingSymbolicOperand(MCInst &Inst, int64_t Value,
                                              uint64_t Address, bool IsBranch,
                                              uint64_t Offset, uint64_t OpSize,
                                              uint64_t InstSize) const {
  if (Symbolizer) {
    assert(CommentStream && "CommentStream is not set.");
    return Symbolizer->tryAddingSymbolicOperand(Inst, *CommentStream, Value,
                                                Address, IsBranch, Offset,
                                                OpSize, InstSize);
  }
```
- **EN**: Implements logic around `tryAddingSymbolicOperand`, `assert`; this block parses assembly syntax or operands; decodes machine-code bytes into symbolic instruction form; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `tryAddingSymbolicOperand`, `assert` 实现具体逻辑；这一段解析汇编语法或操作数，把机器码字节解码为符号化指令形式，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 38-47
```cpp
  return false;
}

void MCDisassembler::tryAddingPcLoadReferenceComment(int64_t Value,
                                                     uint64_t Address) const {
  if (Symbolizer) {
    assert(CommentStream && "CommentStream is not set.");
    Symbolizer->tryAddingPcLoadReferenceComment(*CommentStream, Value, Address);
  }
}
```
- **EN**: Implements logic around `tryAddingPcLoadReferenceComment`, `assert`; this block decodes machine-code bytes into symbolic instruction form; updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `tryAddingPcLoadReferenceComment`, `assert` 实现具体逻辑；这一段把机器码字节解码为符号化指令形式，更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 48-52
```cpp

void MCDisassembler::setSymbolizer(std::unique_ptr<MCSymbolizer> Symzer) {
  Symbolizer = std::move(Symzer);
}

```
- **EN**: Implements logic around `setSymbolizer`, `move`; this block decodes machine-code bytes into symbolic instruction form; updates MC section or symbol state.
- **CN**: 围绕 `setSymbolizer`, `move` 实现具体逻辑；这一段把机器码字节解码为符号化指令形式，更新 MC 节区或符号状态。

### Lines 53-62
```cpp
#define SMC_PCASE(A, P)                                                         \
  case XCOFF::XMC_##A:                                                         \
    return P;

static uint8_t getSMCPriority(XCOFF::StorageMappingClass SMC) {
  switch (SMC) {
    SMC_PCASE(PR, 1)
    SMC_PCASE(RO, 1)
    SMC_PCASE(DB, 1)
    SMC_PCASE(GL, 1)
```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 63-72
```cpp
    SMC_PCASE(XO, 1)
    SMC_PCASE(SV, 1)
    SMC_PCASE(SV64, 1)
    SMC_PCASE(SV3264, 1)
    SMC_PCASE(TI, 1)
    SMC_PCASE(TB, 1)
    SMC_PCASE(RW, 1)
    SMC_PCASE(TC0, 0)
    SMC_PCASE(TC, 1)
    SMC_PCASE(TD, 1)
```
- **EN**: Implements logic around `SMC_PCASE`.
- **CN**: 围绕 `SMC_PCASE` 实现具体逻辑。

### Lines 73-82
```cpp
    SMC_PCASE(DS, 1)
    SMC_PCASE(UA, 1)
    SMC_PCASE(BS, 1)
    SMC_PCASE(UC, 1)
    SMC_PCASE(TL, 1)
    SMC_PCASE(UL, 1)
    SMC_PCASE(TE, 1)
#undef SMC_PCASE
  }
  return 0;
```
- **EN**: Implements logic around `SMC_PCASE`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `SMC_PCASE` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 83-87
```cpp
}

/// The function is for symbol sorting when symbols have the same address.
/// The symbols in the same section are sorted in ascending order.
/// llvm-objdump -D will choose the highest priority symbol to display when
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

### Lines 88-93
```cpp
/// there are symbols with the same address.
bool XCOFFSymbolInfoTy::operator<(const XCOFFSymbolInfoTy &SymInfo) const {
  // Label symbols have higher priority than non-label symbols.
  if (IsLabel != SymInfo.IsLabel)
    return SymInfo.IsLabel;

```
- **EN**: Implements logic around `operator<`; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `operator<` 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 94-98
```cpp
  // Symbols with a StorageMappingClass have higher priority than those without.
  if (StorageMappingClass.has_value() !=
      SymInfo.StorageMappingClass.has_value())
    return SymInfo.StorageMappingClass.has_value();

```
- **EN**: Implements logic around `has_value`; this block updates MC section or symbol state; returns subsystem-specific computed results.
- **CN**: 围绕 `has_value` 实现具体逻辑；这一段更新 MC 节区或符号状态，返回子系统相关的计算结果。

### Lines 99-103
```cpp
  if (StorageMappingClass) {
    return getSMCPriority(*StorageMappingClass) <
           getSMCPriority(*SymInfo.StorageMappingClass);
  }

```
- **EN**: Implements logic around `getSMCPriority`; this block returns subsystem-specific computed results.
- **CN**: 围绕 `getSMCPriority` 实现具体逻辑；这一段返回子系统相关的计算结果。

### Lines 104-105
```cpp
  return false;
}
```
- **EN**: Contains supporting implementation details for the surrounding subsystem logic.
- **CN**: 包含周边子系统逻辑所需的辅助实现细节。

## Key Concepts / 关键概念

- **Disassembly / 反汇编**:
  - **EN**: Decodes raw bytes into MCInst objects and tries to recover symbolic references
  - **CN**: 将原始字节解码为 MCInst，并尝试恢复符号引用
- **MC instruction model / MC 指令模型**:
  - **EN**: Uses MCInst/MCOperand abstractions to represent machine instructions generically
  - **CN**: 使用 MCInst/MCOperand 抽象以通用方式表示机器指令
- **Object layout / 目标文件布局**:
  - **EN**: Organizes fragments into sections and computes final offsets
  - **CN**: 把片段组织到节区中并计算最终偏移

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/MC/MCDisassembler/MCDisassembler.h`, `llvm/ADT/ArrayRef.h`
- **LLVM subsystems / LLVM 子系统**: MC
