# Opcode.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/include/lldb/Core/Opcode.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Declares LLDB core debugger abstractions such as modules, addresses, targets, and settings.
  - **CN**: 声明 LLDB 核心调试抽象，例如模块、地址、目标与设置。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- Opcode.h ------------------------------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/LLDB file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/LLDB 标准文件头、许可证声明以及文件的高层说明。

### Lines 8-14
```cpp

#ifndef LLDB_CORE_OPCODE_H
#define LLDB_CORE_OPCODE_H

#include "lldb/Utility/Endian.h"
#include "lldb/lldb-enumerations.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `lldb/Utility/Endian.h`, `lldb/lldb-enumerations.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `lldb/Utility/Endian.h`, `lldb/lldb-enumerations.h`。

### Lines 15-21
```cpp
#include "llvm/ADT/bit.h"

#include <cassert>
#include <cstdint>
#include <cstring>

namespace lldb {
```
- **EN**: Pulls in the headers needed by this translation unit, including `llvm/ADT/bit.h`, `cassert`, `cstdint`, `cstring`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `llvm/ADT/bit.h`, `cassert`, `cstdint`, `cstring`。

### Lines 22-28
```cpp
class SBInstruction;
}

namespace lldb_private {
class DataExtractor;
class Stream;

```
- **EN**: Introduces declarations for `SBInstruction`, `lldb_private`, `DataExtractor`, `Stream`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `SBInstruction`, `lldb_private`, `DataExtractor`, `Stream` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 29-42
```cpp
class Opcode {
public:
  enum Type {
    eTypeInvalid,
    eType8,
    eType16,
    eType16_2,        // a 32-bit Thumb instruction, made up of two words
    eType16_32Tuples, // RISC-V that can have 2, 4, 6, 8 etc byte long
                      // instructions which will be printed in combinations of
                      // 16 & 32-bit words.
    eType32,
    eType64,
    eTypeBytes
  };
```
- **EN**: Introduces declarations for `Opcode`, `Type`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `Opcode`, `Type` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 43-50
```cpp

  Opcode() = default;

  Opcode(uint8_t inst, lldb::ByteOrder order)
      : m_byte_order(order), m_type(eType8) {
    m_data.inst8 = inst;
  }

```
- **EN**: Implements logic around `Opcode`, `m_byte_order`.
- **CN**: 围绕 `Opcode`, `m_byte_order` 实现具体逻辑。

### Lines 51-60
```cpp
  Opcode(uint16_t inst, lldb::ByteOrder order)
      : m_byte_order(order), m_type(eType16) {
    m_data.inst16 = inst;
  }

  Opcode(uint32_t inst, lldb::ByteOrder order)
      : m_byte_order(order), m_type(eType32) {
    m_data.inst32 = inst;
  }

```
- **EN**: Implements logic around `Opcode`, `m_byte_order`.
- **CN**: 围绕 `Opcode`, `m_byte_order` 实现具体逻辑。

### Lines 61-70
```cpp
  Opcode(uint64_t inst, lldb::ByteOrder order)
      : m_byte_order(order), m_type(eType64) {
    m_data.inst64 = inst;
  }

  Opcode(uint8_t *bytes, size_t length, Opcode::Type type,
         lldb::ByteOrder order) {
    DoSetOpcodeBytes(bytes, length, type, order);
  }

```
- **EN**: Implements logic around `Opcode`, `m_byte_order`, `DoSetOpcodeBytes`.
- **CN**: 围绕 `Opcode`, `m_byte_order`, `DoSetOpcodeBytes` 实现具体逻辑。

### Lines 71-77
```cpp
  void Clear() {
    m_byte_order = lldb::eByteOrderInvalid;
    m_type = Opcode::eTypeInvalid;
  }

  Opcode::Type GetType() const { return m_type; }

```
- **EN**: Implements logic around `Clear`, `GetType`.
- **CN**: 围绕 `Clear`, `GetType` 实现具体逻辑。

### Lines 78-91
```cpp
  uint8_t GetOpcode8(uint8_t invalid_opcode = UINT8_MAX) const {
    switch (m_type) {
    case Opcode::eTypeInvalid:
      break;
    case Opcode::eType8:
      return m_data.inst8;
    case Opcode::eType16:
      break;
    case Opcode::eType16_2:
      break;
    case Opcode::eType16_32Tuples:
      break;
    case Opcode::eType32:
      break;
```
- **EN**: Implements logic around `GetOpcode8`.
- **CN**: 围绕 `GetOpcode8` 实现具体逻辑。

### Lines 92-99
```cpp
    case Opcode::eType64:
      break;
    case Opcode::eTypeBytes:
      break;
    }
    return invalid_opcode;
  }

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 100-113
```cpp
  uint16_t GetOpcode16(uint16_t invalid_opcode = UINT16_MAX) const {
    switch (m_type) {
    case Opcode::eTypeInvalid:
      break;
    case Opcode::eType8:
      return m_data.inst8;
    case Opcode::eType16:
      return GetEndianSwap() ? llvm::byteswap<uint16_t>(m_data.inst16)
                             : m_data.inst16;
    case Opcode::eType16_2:
      break;
    case Opcode::eType16_32Tuples:
      break;
    case Opcode::eType32:
```
- **EN**: Implements logic around `GetOpcode16`, `GetEndianSwap`.
- **CN**: 围绕 `GetOpcode16`, `GetEndianSwap` 实现具体逻辑。

### Lines 114-122
```cpp
      break;
    case Opcode::eType64:
      break;
    case Opcode::eTypeBytes:
      break;
    }
    return invalid_opcode;
  }

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 123-136
```cpp
  uint32_t GetOpcode32(uint32_t invalid_opcode = UINT32_MAX) const {
    switch (m_type) {
    case Opcode::eTypeInvalid:
      break;
    case Opcode::eType8:
      return m_data.inst8;
    case Opcode::eType16:
      return GetEndianSwap() ? llvm::byteswap<uint16_t>(m_data.inst16)
                             : m_data.inst16;
    case Opcode::eType16_32Tuples:
      break;
    case Opcode::eType16_2: // passthrough
    case Opcode::eType32:
      return GetEndianSwap() ? llvm::byteswap<uint32_t>(m_data.inst32)
```
- **EN**: Implements logic around `GetOpcode32`, `GetEndianSwap`.
- **CN**: 围绕 `GetOpcode32`, `GetEndianSwap` 实现具体逻辑。

### Lines 137-145
```cpp
                             : m_data.inst32;
    case Opcode::eType64:
      break;
    case Opcode::eTypeBytes:
      break;
    }
    return invalid_opcode;
  }

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 146-159
```cpp
  uint64_t GetOpcode64(uint64_t invalid_opcode = UINT64_MAX) const {
    switch (m_type) {
    case Opcode::eTypeInvalid:
      break;
    case Opcode::eType8:
      return m_data.inst8;
    case Opcode::eType16:
      return GetEndianSwap() ? llvm::byteswap<uint16_t>(m_data.inst16)
                             : m_data.inst16;
    case Opcode::eType16_32Tuples:
      break;
    case Opcode::eType16_2: // passthrough
    case Opcode::eType32:
      return GetEndianSwap() ? llvm::byteswap<uint32_t>(m_data.inst32)
```
- **EN**: Implements logic around `GetOpcode64`, `GetEndianSwap`.
- **CN**: 围绕 `GetOpcode64`, `GetEndianSwap` 实现具体逻辑。

### Lines 160-169
```cpp
                             : m_data.inst32;
    case Opcode::eType64:
      return GetEndianSwap() ? llvm::byteswap<uint64_t>(m_data.inst64)
                             : m_data.inst64;
    case Opcode::eTypeBytes:
      break;
    }
    return invalid_opcode;
  }

```
- **EN**: Declares APIs around `GetEndianSwap`.
- **CN**: 声明与 `GetEndianSwap` 相关的 API。

### Lines 170-181
```cpp
  void SetOpcode8(uint8_t inst, lldb::ByteOrder order) {
    m_type = eType8;
    m_data.inst8 = inst;
    m_byte_order = order;
  }

  void SetOpcode16(uint16_t inst, lldb::ByteOrder order) {
    m_type = eType16;
    m_data.inst16 = inst;
    m_byte_order = order;
  }

```
- **EN**: Implements logic around `SetOpcode8`, `SetOpcode16`.
- **CN**: 围绕 `SetOpcode8`, `SetOpcode16` 实现具体逻辑。

### Lines 182-193
```cpp
  void SetOpcode16_2(uint32_t inst, lldb::ByteOrder order) {
    m_type = eType16_2;
    m_data.inst32 = inst;
    m_byte_order = order;
  }

  void SetOpcode32(uint32_t inst, lldb::ByteOrder order) {
    m_type = eType32;
    m_data.inst32 = inst;
    m_byte_order = order;
  }

```
- **EN**: Implements logic around `SetOpcode16_2`, `SetOpcode32`.
- **CN**: 围绕 `SetOpcode16_2`, `SetOpcode32` 实现具体逻辑。

### Lines 194-204
```cpp
  void SetOpcode64(uint64_t inst, lldb::ByteOrder order) {
    m_type = eType64;
    m_data.inst64 = inst;
    m_byte_order = order;
  }

  void SetOpcode16_32TupleBytes(const void *bytes, size_t length,
                                lldb::ByteOrder order) {
    DoSetOpcodeBytes(bytes, length, eType16_32Tuples, order);
  }

```
- **EN**: Implements logic around `SetOpcode64`, `SetOpcode16_32TupleBytes`, `DoSetOpcodeBytes`.
- **CN**: 围绕 `SetOpcode64`, `SetOpcode16_32TupleBytes`, `DoSetOpcodeBytes` 实现具体逻辑。

### Lines 205-218
```cpp
  void SetOpcodeBytes(const void *bytes, size_t length) {
    DoSetOpcodeBytes(bytes, length, eTypeBytes, lldb::eByteOrderInvalid);
  }

  void DoSetOpcodeBytes(const void *bytes, size_t length, Opcode::Type type,
                        lldb::ByteOrder order) {
    if (bytes != nullptr && length > 0) {
      m_type = type;
      m_data.inst.length = length;
      assert(length <= sizeof(m_data.inst.bytes));
      memcpy(m_data.inst.bytes, bytes, length);
      m_byte_order = order;
    } else {
      m_type = eTypeInvalid;
```
- **EN**: Implements logic around `SetOpcodeBytes`, `DoSetOpcodeBytes`, `assert`, `memcpy`.
- **CN**: 围绕 `SetOpcodeBytes`, `DoSetOpcodeBytes`, `assert`, `memcpy` 实现具体逻辑。

### Lines 219-230
```cpp
      m_data.inst.length = 0;
    }
  }

  int Dump(Stream *s, uint32_t min_byte_width) const;

  const void *GetOpcodeBytes() const {
    return ((m_type == Opcode::eTypeBytes || m_type == Opcode::eType16_32Tuples)
                ? m_data.inst.bytes
                : nullptr);
  }

```
- **EN**: Implements logic around `Dump`, `GetOpcodeBytes`.
- **CN**: 围绕 `Dump`, `GetOpcodeBytes` 实现具体逻辑。

### Lines 231-244
```cpp
  uint32_t GetByteSize() const {
    switch (m_type) {
    case Opcode::eTypeInvalid:
      break;
    case Opcode::eType8:
      return sizeof(m_data.inst8);
    case Opcode::eType16:
      return sizeof(m_data.inst16);
    case Opcode::eType16_32Tuples:
      return m_data.inst.length;
    case Opcode::eType16_2: // passthrough
    case Opcode::eType32:
      return sizeof(m_data.inst32);
    case Opcode::eType64:
```
- **EN**: Implements logic around `GetByteSize`.
- **CN**: 围绕 `GetByteSize` 实现具体逻辑。

### Lines 245-251
```cpp
      return sizeof(m_data.inst64);
    case Opcode::eTypeBytes:
      return m_data.inst.length;
    }
    return 0;
  }

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 252-265
```cpp
  // Get the opcode exactly as it would be laid out in memory.
  uint32_t GetData(DataExtractor &data) const;

protected:
  friend class lldb::SBInstruction;

  const void *GetOpcodeDataBytes() const {
    switch (m_type) {
    case Opcode::eTypeInvalid:
      break;
    case Opcode::eType8:
      return &m_data.inst8;
    case Opcode::eType16:
      return &m_data.inst16;
```
- **EN**: Implements logic around `GetData`, `GetOpcodeDataBytes`; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `GetData`, `GetOpcodeDataBytes` 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 266-278
```cpp
    case Opcode::eType16_32Tuples:
      return m_data.inst.bytes;
    case Opcode::eType16_2: // passthrough
    case Opcode::eType32:
      return &m_data.inst32;
    case Opcode::eType64:
      return &m_data.inst64;
    case Opcode::eTypeBytes:
      return m_data.inst.bytes;
    }
    return nullptr;
  }

```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 279-287
```cpp
  lldb::ByteOrder GetDataByteOrder() const;

  bool GetEndianSwap() const {
    return (m_byte_order == lldb::eByteOrderBig &&
            endian::InlHostByteOrder() == lldb::eByteOrderLittle) ||
           (m_byte_order == lldb::eByteOrderLittle &&
            endian::InlHostByteOrder() == lldb::eByteOrderBig);
  }

```
- **EN**: Implements logic around `GetDataByteOrder`, `GetEndianSwap`, `InlHostByteOrder`.
- **CN**: 围绕 `GetDataByteOrder`, `GetEndianSwap`, `InlHostByteOrder` 实现具体逻辑。

### Lines 288-296
```cpp
  lldb::ByteOrder m_byte_order = lldb::eByteOrderInvalid;

  Opcode::Type m_type = eTypeInvalid;
  union {
    uint8_t inst8;
    uint16_t inst16;
    uint32_t inst32;
    uint64_t inst64;
    struct {
```
- **EN**: Contains supporting implementation details for the surrounding LLDB subsystem.
- **CN**: 包含周边 LLDB 子系统所需的辅助实现细节。

### Lines 297-303
```cpp
      uint8_t bytes[16]; // This must be big enough to handle any opcode for any
                         // supported target.
      uint8_t length;
    } inst;
  } m_data;
};

```
- **EN**: Contains supporting implementation details for the surrounding LLDB subsystem.
- **CN**: 包含周边 LLDB 子系统所需的辅助实现细节。

### Lines 304-306
```cpp
} // namespace lldb_private

#endif // LLDB_CORE_OPCODE_H
```
- **EN**: Defines preprocessor-controlled structure, feature gates, or include-guard state.
- **CN**: 定义受预处理器控制的结构、特性开关或头文件保护状态。

## Key Concepts / 关键概念

- **LLDB architecture / LLDB 架构**:
  - **EN**: Explains how LLDB organizes debugger APIs, runtime objects, and platform-specific helpers.
  - **CN**: 说明 LLDB 如何组织调试器 API、运行时对象以及平台相关辅助组件。
- **Debugger core model / 调试器核心模型**:
  - **EN**: Defines long-lived objects such as modules, addresses, symbol contexts, and debugger settings.
  - **CN**: 定义模块、地址、符号上下文与调试器设置等长生命周期对象。
- **Runtime control / 运行时控制**:
  - **EN**: Models debugger ownership over targets, processes, threads, and event delivery.
  - **CN**: 建模调试器对目标、进程、线程以及事件投递的控制。

## Dependencies / 依赖关系

- **Direct LLDB/LLVM includes / 直接的 LLDB/LLVM 包含**: `lldb/Utility/Endian.h`, `lldb/lldb-enumerations.h`, `llvm/ADT/bit.h`
- **Standard-library headers / 标准库头文件**: `<cassert>`, `<cstdint>`, `<cstring>`
- **Subsystem categories / 子系统类别**: shared LLDB utility classes / 共享 LLDB 工具类 (1), shared LLDB public/private type declarations / 共享 LLDB 公共/私有类型声明 (1), LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (1)
