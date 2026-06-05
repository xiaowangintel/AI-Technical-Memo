# NativeRegisterContextLinux.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/source/Plugins/Process/Linux/NativeRegisterContextLinux.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements the LLDB support logic associated with `NativeRegisterContextLinux`.
  - **CN**: 实现与 `NativeRegisterContextLinux` 相关的 LLDB 支持逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- NativeRegisterContextLinux.cpp ------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/LLDB file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM/LLDB 标准文件头、许可证声明以及文件的高层说明。

### Lines 8-19
```cpp

#include "NativeRegisterContextLinux.h"

#include "Plugins/Process/Linux/NativeProcessLinux.h"
#include "Plugins/Process/POSIX/ProcessPOSIXLog.h"
#include "lldb/Host/HostInfo.h"
#include "lldb/Host/common/NativeProcessProtocol.h"
#include "lldb/Host/common/NativeThreadProtocol.h"
#include "lldb/Host/linux/Ptrace.h"
#include "lldb/Utility/RegisterValue.h"
#include <sys/uio.h>

```
- **EN**: Pulls in the headers needed by this translation unit, including `NativeRegisterContextLinux.h`, `Plugins/Process/Linux/NativeProcessLinux.h`, `Plugins/Process/POSIX/ProcessPOSIXLog.h`, `lldb/Host/HostInfo.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `NativeRegisterContextLinux.h`, `Plugins/Process/Linux/NativeProcessLinux.h`, `Plugins/Process/POSIX/ProcessPOSIXLog.h`, `lldb/Host/HostInfo.h`。

### Lines 20-26
```cpp
using namespace lldb_private;
using namespace lldb_private::process_linux;

lldb::ByteOrder NativeRegisterContextLinux::GetByteOrder() const {
  return m_thread.GetProcess().GetByteOrder();
}

```
- **EN**: Implements logic around `GetByteOrder`, `GetProcess`.
- **CN**: 围绕 `GetByteOrder`, `GetProcess` 实现具体逻辑。

### Lines 27-33
```cpp
Status NativeRegisterContextLinux::ReadRegisterRaw(uint32_t reg_index,
                                                   RegisterValue &reg_value) {
  const RegisterInfo *const reg_info = GetRegisterInfoAtIndex(reg_index);
  if (!reg_info)
    return Status::FromErrorStringWithFormat("register %" PRIu32 " not found",
                                             reg_index);

```
- **EN**: Implements logic around `ReadRegisterRaw`, `GetRegisterInfoAtIndex`, `FromErrorStringWithFormat`; this block propagates recoverable errors, status objects, or diagnostics; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `ReadRegisterRaw`, `GetRegisterInfoAtIndex`, `FromErrorStringWithFormat` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 34-43
```cpp
  return DoReadRegisterValue(GetPtraceOffset(reg_index), reg_info->name,
                             reg_info->byte_size, reg_value);
}

Status
NativeRegisterContextLinux::WriteRegisterRaw(uint32_t reg_index,
                                             const RegisterValue &reg_value) {
  uint32_t reg_to_write = reg_index;
  RegisterValue value_to_write = reg_value;

```
- **EN**: Implements logic around `DoReadRegisterValue`, `WriteRegisterRaw`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `DoReadRegisterValue`, `WriteRegisterRaw` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 44-50
```cpp
  // Check if this is a subregister of a full register.
  const RegisterInfo *reg_info = GetRegisterInfoAtIndex(reg_index);
  assert(reg_info && "Expected valid register info for reg_index.");
  if (reg_info->invalidate_regs &&
      (reg_info->invalidate_regs[0] != LLDB_INVALID_REGNUM)) {
    Status error;

```
- **EN**: Implements logic around `GetRegisterInfoAtIndex`, `assert`; this block propagates recoverable errors, status objects, or diagnostics; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `GetRegisterInfoAtIndex`, `assert` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 51-61
```cpp
    RegisterValue full_value;
    uint32_t full_reg = reg_info->invalidate_regs[0];
    const RegisterInfo *full_reg_info = GetRegisterInfoAtIndex(full_reg);

    // Read the full register.
    error = ReadRegister(full_reg_info, full_value);
    if (error.Fail()) {
      // full_reg_info was nullptr, or we couldn't read the register.
      return error;
    }

```
- **EN**: Implements logic around `GetRegisterInfoAtIndex`, `ReadRegister`, `Fail`; this block propagates recoverable errors, status objects, or diagnostics; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `GetRegisterInfoAtIndex`, `ReadRegister`, `Fail` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 62-70
```cpp
    lldb::ByteOrder byte_order = GetByteOrder();
    RegisterValue::BytesContainer dst(full_reg_info->byte_size);

    // Get the bytes for the full register.
    const uint32_t dest_size = full_value.GetAsMemoryData(
        *full_reg_info, dst.data(), dst.size(), byte_order, error);
    if (error.Success() && dest_size) {
      RegisterValue::BytesContainer src(reg_info->byte_size);

```
- **EN**: Implements logic around `GetByteOrder`, `dst`, `GetAsMemoryData`, `Success`, and 1 more symbols; this block propagates recoverable errors, status objects, or diagnostics; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `GetByteOrder`, `dst`, `GetAsMemoryData`, `Success`, and 1 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 71-84
```cpp
      // Get the bytes for the source data.
      const uint32_t src_size = reg_value.GetAsMemoryData(
          *reg_info, src.data(), src.size(), byte_order, error);
      if (error.Success() && src_size && (src_size < dest_size)) {
        // Copy the src bytes to the destination.
        memcpy(dst.data() + (reg_info->byte_offset & 0x1), src.data(),
               src_size);
        // Set this full register as the value to write.
        value_to_write.SetBytes(dst.data(), full_value.GetByteSize(),
                                byte_order);
        value_to_write.SetType(*full_reg_info);
        reg_to_write = full_reg;
      }
    }
```
- **EN**: Implements logic around `GetAsMemoryData`, `Success`, `memcpy`, `SetBytes`, and 1 more symbols; this block propagates recoverable errors, status objects, or diagnostics; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `GetAsMemoryData`, `Success`, `memcpy`, `SetBytes`, and 1 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 85-96
```cpp
  }

  const RegisterInfo *const register_to_write_info_p =
      GetRegisterInfoAtIndex(reg_to_write);
  assert(register_to_write_info_p &&
         "register to write does not have valid RegisterInfo");
  if (!register_to_write_info_p)
    return Status::FromErrorStringWithFormat(
        "NativeRegisterContextLinux::%s failed to get RegisterInfo "
        "for write register index %" PRIu32,
        __FUNCTION__, reg_to_write);

```
- **EN**: Implements logic around `GetRegisterInfoAtIndex`, `assert`, `FromErrorStringWithFormat`; this block propagates recoverable errors, status objects, or diagnostics; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `GetRegisterInfoAtIndex`, `assert`, `FromErrorStringWithFormat` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 97-105
```cpp
  return DoWriteRegisterValue(GetPtraceOffset(reg_index), reg_info->name,
                              reg_value);
}

Status NativeRegisterContextLinux::ReadGPR() {
  return NativeProcessLinux::PtraceWrapper(
      PTRACE_GETREGS, m_thread.GetID(), nullptr, GetGPRBuffer(), GetGPRSize());
}

```
- **EN**: Implements logic around `DoWriteRegisterValue`, `ReadGPR`, `PtraceWrapper`, `GetID`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `DoWriteRegisterValue`, `ReadGPR`, `PtraceWrapper`, `GetID` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 106-116
```cpp
Status NativeRegisterContextLinux::WriteGPR() {
  return NativeProcessLinux::PtraceWrapper(
      PTRACE_SETREGS, m_thread.GetID(), nullptr, GetGPRBuffer(), GetGPRSize());
}

Status NativeRegisterContextLinux::ReadFPR() {
  return NativeProcessLinux::PtraceWrapper(PTRACE_GETFPREGS, m_thread.GetID(),
                                           nullptr, GetFPRBuffer(),
                                           GetFPRSize());
}

```
- **EN**: Implements logic around `WriteGPR`, `PtraceWrapper`, `GetID`, `ReadFPR`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `WriteGPR`, `PtraceWrapper`, `GetID`, `ReadFPR`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 117-129
```cpp
Status NativeRegisterContextLinux::WriteFPR() {
  return NativeProcessLinux::PtraceWrapper(PTRACE_SETFPREGS, m_thread.GetID(),
                                           nullptr, GetFPRBuffer(),
                                           GetFPRSize());
}

Status NativeRegisterContextLinux::ReadRegisterSet(void *buf, size_t buf_size,
                                                   unsigned int regset) {
  return NativeProcessLinux::PtraceWrapper(PTRACE_GETREGSET, m_thread.GetID(),
                                           static_cast<void *>(&regset), buf,
                                           buf_size);
}

```
- **EN**: Implements logic around `WriteFPR`, `PtraceWrapper`, `GetFPRBuffer`, `GetFPRSize`, and 1 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `WriteFPR`, `PtraceWrapper`, `GetFPRBuffer`, `GetFPRSize`, and 1 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 130-136
```cpp
Status NativeRegisterContextLinux::WriteRegisterSet(void *buf, size_t buf_size,
                                                    unsigned int regset) {
  return NativeProcessLinux::PtraceWrapper(PTRACE_SETREGSET, m_thread.GetID(),
                                           static_cast<void *>(&regset), buf,
                                           buf_size);
}

```
- **EN**: Implements logic around `WriteRegisterSet`, `PtraceWrapper`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `WriteRegisterSet`, `PtraceWrapper` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 137-147
```cpp
Status NativeRegisterContextLinux::DoReadRegisterValue(uint32_t offset,
                                                       const char *reg_name,
                                                       uint32_t size,
                                                       RegisterValue &value) {
  Log *log = GetLog(POSIXLog::Registers);

  long data;
  Status error = NativeProcessLinux::PtraceWrapper(
      PTRACE_PEEKUSER, m_thread.GetID(), reinterpret_cast<void *>(offset),
      nullptr, 0, &data);

```
- **EN**: Implements logic around `DoReadRegisterValue`, `GetLog`, `PtraceWrapper`, `GetID`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `DoReadRegisterValue`, `GetLog`, `PtraceWrapper`, `GetID` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 148-155
```cpp
  if (error.Success())
    // First cast to an unsigned of the same size to avoid sign extension.
    value.SetUInt(static_cast<unsigned long>(data), size);

  LLDB_LOG(log, "{0}: {1:x}", reg_name, data);
  return error;
}

```
- **EN**: Implements logic around `Success`, `SetUInt`, `LLDB_LOG`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `Success`, `SetUInt`, `LLDB_LOG` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 156-162
```cpp
Status NativeRegisterContextLinux::DoWriteRegisterValue(
    uint32_t offset, const char *reg_name, const RegisterValue &value) {
  Log *log = GetLog(POSIXLog::Registers);

  void *buf = reinterpret_cast<void *>(value.GetAsUInt64());
  LLDB_LOG(log, "{0}: {1}", reg_name, buf);

```
- **EN**: Implements logic around `DoWriteRegisterValue`, `GetLog`, `GetAsUInt64`, `LLDB_LOG`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `DoWriteRegisterValue`, `GetLog`, `GetAsUInt64`, `LLDB_LOG` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 163-171
```cpp
  return NativeProcessLinux::PtraceWrapper(
      PTRACE_POKEUSER, m_thread.GetID(), reinterpret_cast<void *>(offset), buf);
}

llvm::Expected<ArchSpec>
NativeRegisterContextLinux::DetermineArchitectureViaGPR(lldb::tid_t tid,
                                                        size_t gpr64_size) {
  std::unique_ptr<uint8_t[]> data = std::make_unique<uint8_t[]>(gpr64_size);
  struct iovec iov;
```
- **EN**: Introduces declarations for `iovec`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `iovec` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 172-181
```cpp
  iov.iov_base = data.get();
  iov.iov_len = gpr64_size;
  unsigned int regset = llvm::ELF::NT_PRSTATUS;
  Status ST = NativeProcessLinux::PtraceWrapper(PTRACE_GETREGSET, tid, &regset,
                                                &iov, sizeof(iov));
  if (ST.Fail())
    return ST.ToError();
  return HostInfo::GetArchitecture(
      iov.iov_len < gpr64_size ? HostInfo::eArchKind32 : HostInfo::eArchKind64);
}
```
- **EN**: Implements logic around `get`, `PtraceWrapper`, `Fail`, `ToError`, and 1 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `get`, `PtraceWrapper`, `Fail`, `ToError`, and 1 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

## Key Concepts / 关键概念

- **LLDB architecture / LLDB 架构**:
  - **EN**: Explains how LLDB organizes debugger APIs, runtime objects, and platform-specific helpers.
  - **CN**: 说明 LLDB 如何组织调试器 API、运行时对象以及平台相关辅助组件。
- **Runtime control / 运行时控制**:
  - **EN**: Models debugger ownership over targets, processes, threads, and event delivery.
  - **CN**: 建模调试器对目标、进程、线程以及事件投递的控制。
- **OS integration / 操作系统集成**:
  - **EN**: Bridges LLDB abstractions to host files, sockets, terminals, and native process facilities.
  - **CN**: 将 LLDB 抽象连接到主机文件、套接字、终端与本地进程设施。

## Dependencies / 依赖关系

- **Direct LLDB/LLVM includes / 直接的 LLDB/LLVM 包含**: `NativeRegisterContextLinux.h`, `Plugins/Process/Linux/NativeProcessLinux.h`, `Plugins/Process/POSIX/ProcessPOSIXLog.h`, `lldb/Host/HostInfo.h`, `lldb/Host/common/NativeProcessProtocol.h`, `lldb/Host/common/NativeThreadProtocol.h`, `lldb/Host/linux/Ptrace.h`, `lldb/Utility/RegisterValue.h`
- **Standard-library headers / 标准库头文件**: `<sys/uio.h>`
- **Subsystem categories / 子系统类别**: host OS abstraction helpers / 主机操作系统抽象辅助组件 (4), shared LLDB utility classes / 共享 LLDB 工具类 (1)
