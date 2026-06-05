# NativeRegisterContext.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/include/lldb/Host/common/NativeRegisterContext.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Declares cross-platform LLDB host helpers for native processes, sockets, and diagnostics.
  - **CN**: 声明跨平台 LLDB 主机辅助组件，用于本地进程、套接字与诊断。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- NativeRegisterContext.h ---------------------------------*- C++ -*-===//
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

#ifndef LLDB_HOST_COMMON_NATIVEREGISTERCONTEXT_H
#define LLDB_HOST_COMMON_NATIVEREGISTERCONTEXT_H

#include "lldb/Host/common/NativeWatchpointList.h"
#include "lldb/lldb-private.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `lldb/Host/common/NativeWatchpointList.h`, `lldb/lldb-private.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `lldb/Host/common/NativeWatchpointList.h`, `lldb/lldb-private.h`。

### Lines 15-21
```cpp
namespace lldb_private {

class NativeThreadProtocol;

enum class ExpeditedRegs { Minimal, Full };

class NativeRegisterContext
```
- **EN**: Introduces declarations for `lldb_private`, `NativeThreadProtocol`, `ExpeditedRegs`, `NativeRegisterContext`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `lldb_private`, `NativeThreadProtocol`, `ExpeditedRegs`, `NativeRegisterContext` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 22-28
```cpp
    : public std::enable_shared_from_this<NativeRegisterContext> {
public:
  // Constructors and Destructors
  NativeRegisterContext(NativeThreadProtocol &thread);

  virtual ~NativeRegisterContext();

```
- **EN**: Implements logic around `NativeRegisterContext`, `~NativeRegisterContext`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `NativeRegisterContext`, `~NativeRegisterContext` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 29-35
```cpp
  // void
  // InvalidateIfNeeded (bool force);

  // Subclasses must override these functions
  // virtual void
  // InvalidateAllRegisters () = 0;

```
- **EN**: Documents the surrounding declarations, example workflow, or design constraints for this LLDB component.
- **CN**: 说明周边声明、示例流程，或记录该 LLDB 组件的设计约束。

### Lines 36-43
```cpp
  virtual uint32_t GetRegisterCount() const = 0;

  virtual uint32_t GetUserRegisterCount() const = 0;

  virtual const RegisterInfo *GetRegisterInfoAtIndex(uint32_t reg) const = 0;

  const char *GetRegisterSetNameForRegisterAtIndex(uint32_t reg_index) const;

```
- **EN**: Declares APIs around `GetRegisterCount`, `GetUserRegisterCount`, `GetRegisterInfoAtIndex`, `GetRegisterSetNameForRegisterAtIndex`.
- **CN**: 声明与 `GetRegisterCount`, `GetUserRegisterCount`, `GetRegisterInfoAtIndex`, `GetRegisterSetNameForRegisterAtIndex` 相关的 API。

### Lines 44-50
```cpp
  virtual uint32_t GetRegisterSetCount() const = 0;

  virtual const RegisterSet *GetRegisterSet(uint32_t set_index) const = 0;

  virtual Status ReadRegister(const RegisterInfo *reg_info,
                              RegisterValue &reg_value) = 0;

```
- **EN**: Declares APIs around `GetRegisterSetCount`, `GetRegisterSet`, `ReadRegister`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 声明与 `GetRegisterSetCount`, `GetRegisterSet`, `ReadRegister` 相关的 API；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 51-57
```cpp
  virtual Status WriteRegister(const RegisterInfo *reg_info,
                               const RegisterValue &reg_value) = 0;

  virtual Status ReadAllRegisterValues(lldb::WritableDataBufferSP &data_sp) = 0;

  virtual Status WriteAllRegisterValues(const lldb::DataBufferSP &data_sp) = 0;

```
- **EN**: Declares APIs around `WriteRegister`, `ReadAllRegisterValues`, `WriteAllRegisterValues`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 声明与 `WriteRegister`, `ReadAllRegisterValues`, `WriteAllRegisterValues` 相关的 API；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 58-65
```cpp
  uint32_t ConvertRegisterKindToRegisterNumber(uint32_t kind,
                                               uint32_t num) const;

  // Subclasses can override these functions if desired
  virtual uint32_t NumSupportedHardwareBreakpoints();

  virtual uint32_t SetHardwareBreakpoint(lldb::addr_t addr, size_t size);

```
- **EN**: Declares APIs around `ConvertRegisterKindToRegisterNumber`, `NumSupportedHardwareBreakpoints`, `SetHardwareBreakpoint`.
- **CN**: 声明与 `ConvertRegisterKindToRegisterNumber`, `NumSupportedHardwareBreakpoints`, `SetHardwareBreakpoint` 相关的 API。

### Lines 66-72
```cpp
  virtual bool ClearHardwareBreakpoint(uint32_t hw_idx);

  virtual Status ClearAllHardwareBreakpoints();

  virtual Status GetHardwareBreakHitIndex(uint32_t &bp_index,
                                          lldb::addr_t trap_addr);

```
- **EN**: Declares APIs around `ClearHardwareBreakpoint`, `ClearAllHardwareBreakpoints`, `GetHardwareBreakHitIndex`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 声明与 `ClearHardwareBreakpoint`, `ClearAllHardwareBreakpoints`, `GetHardwareBreakHitIndex` 相关的 API；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 73-79
```cpp
  virtual uint32_t NumSupportedHardwareWatchpoints();

  virtual uint32_t SetHardwareWatchpoint(lldb::addr_t addr, size_t size,
                                         uint32_t watch_flags);

  virtual bool ClearHardwareWatchpoint(uint32_t hw_index);

```
- **EN**: Declares APIs around `NumSupportedHardwareWatchpoints`, `SetHardwareWatchpoint`, `ClearHardwareWatchpoint`.
- **CN**: 声明与 `NumSupportedHardwareWatchpoints`, `SetHardwareWatchpoint`, `ClearHardwareWatchpoint` 相关的 API。

### Lines 80-88
```cpp
  virtual Status ClearWatchpointHit(uint32_t hw_index);

  virtual Status ClearAllHardwareWatchpoints();

  virtual Status IsWatchpointHit(uint32_t wp_index, bool &is_hit);

  virtual Status GetWatchpointHitIndex(uint32_t &wp_index,
                                       lldb::addr_t trap_addr);

```
- **EN**: Declares APIs around `ClearWatchpointHit`, `ClearAllHardwareWatchpoints`, `IsWatchpointHit`, `GetWatchpointHitIndex`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 声明与 `ClearWatchpointHit`, `ClearAllHardwareWatchpoints`, `IsWatchpointHit`, `GetWatchpointHitIndex` 相关的 API；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 89-102
```cpp
  virtual Status IsWatchpointVacant(uint32_t wp_index, bool &is_vacant);

  virtual lldb::addr_t GetWatchpointAddress(uint32_t wp_index);

  // MIPS Linux kernel returns a masked address (last 3bits are masked)
  // when a HW watchpoint is hit. However user may not have set a watchpoint on
  // this address. This function emulates the instruction at PC and finds the
  // base address used in the load/store instruction. This gives the exact
  // address used to read/write the variable being watched. For example: 'n' is
  // at 0x120010d00 and 'm' is 0x120010d04. When a watchpoint is set at 'm',
  // then watch exception is generated even when 'n' is read/written. This
  // function returns address of 'n' so that client can check whether a
  // watchpoint is set on this address or not.
  virtual lldb::addr_t GetWatchpointHitAddress(uint32_t wp_index);
```
- **EN**: Declares APIs around `IsWatchpointVacant`, `GetWatchpointAddress`, `GetWatchpointHitAddress`; this block propagates recoverable errors, status objects, or diagnostics; tracks breakpoint state, stop conditions, or hit-processing policy; maps executable state back to modules, symbols, sections, or addresses; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 声明与 `IsWatchpointVacant`, `GetWatchpointAddress`, `GetWatchpointHitAddress` 相关的 API；该代码块传播可恢复错误、状态对象或诊断信息，并跟踪断点状态、停机条件或命中处理策略，并将可执行状态映射回模块、符号、节区或地址，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 103-110
```cpp

  virtual bool HardwareSingleStep(bool enable);

  virtual Status
  ReadRegisterValueFromMemory(const lldb_private::RegisterInfo *reg_info,
                              lldb::addr_t src_addr, size_t src_len,
                              RegisterValue &reg_value);

```
- **EN**: Declares APIs around `HardwareSingleStep`, `ReadRegisterValueFromMemory`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 声明与 `HardwareSingleStep`, `ReadRegisterValueFromMemory` 相关的 API；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 111-118
```cpp
  virtual Status
  WriteRegisterValueToMemory(const lldb_private::RegisterInfo *reg_info,
                             lldb::addr_t dst_addr, size_t dst_len,
                             const RegisterValue &reg_value);

  // Subclasses should not override these
  virtual lldb::tid_t GetThreadID() const;

```
- **EN**: Declares APIs around `WriteRegisterValueToMemory`, `GetThreadID`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 声明与 `WriteRegisterValueToMemory`, `GetThreadID` 相关的 API；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 119-125
```cpp
  virtual NativeThreadProtocol &GetThread() { return m_thread; }

  virtual std::vector<uint32_t>
  GetExpeditedRegisters(ExpeditedRegs expType) const;

  virtual bool RegisterOffsetIsDynamic() const { return false; }

```
- **EN**: Implements logic around `GetThread`, `GetExpeditedRegisters`, `RegisterOffsetIsDynamic`.
- **CN**: 围绕 `GetThread`, `GetExpeditedRegisters`, `RegisterOffsetIsDynamic` 实现具体逻辑。

### Lines 126-132
```cpp
  const RegisterInfo *GetRegisterInfoByName(llvm::StringRef reg_name,
                                            uint32_t start_idx = 0);

  const RegisterInfo *GetRegisterInfo(uint32_t reg_kind, uint32_t reg_num);

  lldb::addr_t GetPC(lldb::addr_t fail_value = LLDB_INVALID_ADDRESS);

```
- **EN**: Declares APIs around `GetRegisterInfoByName`, `GetRegisterInfo`, `GetPC`.
- **CN**: 声明与 `GetRegisterInfoByName`, `GetRegisterInfo`, `GetPC` 相关的 API。

### Lines 133-139
```cpp
  virtual lldb::addr_t
  GetPCfromBreakpointLocation(lldb::addr_t fail_value = LLDB_INVALID_ADDRESS);

  Status SetPC(lldb::addr_t pc);

  lldb::addr_t GetSP(lldb::addr_t fail_value = LLDB_INVALID_ADDRESS);

```
- **EN**: Declares APIs around `GetPCfromBreakpointLocation`, `SetPC`, `GetSP`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 声明与 `GetPCfromBreakpointLocation`, `SetPC`, `GetSP` 相关的 API；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 140-147
```cpp
  Status SetSP(lldb::addr_t sp);

  lldb::addr_t GetFP(lldb::addr_t fail_value = LLDB_INVALID_ADDRESS);

  Status SetFP(lldb::addr_t fp);

  const char *GetRegisterName(uint32_t reg);

```
- **EN**: Declares APIs around `SetSP`, `GetFP`, `SetFP`, `GetRegisterName`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 声明与 `SetSP`, `GetFP`, `SetFP`, `GetRegisterName` 相关的 API；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 148-156
```cpp
  lldb::addr_t GetReturnAddress(lldb::addr_t fail_value = LLDB_INVALID_ADDRESS);

  lldb::addr_t GetFlags(lldb::addr_t fail_value = 0);

  lldb::addr_t ReadRegisterAsUnsigned(uint32_t reg, lldb::addr_t fail_value);

  lldb::addr_t ReadRegisterAsUnsigned(const RegisterInfo *reg_info,
                                      lldb::addr_t fail_value);

```
- **EN**: Declares APIs around `GetReturnAddress`, `GetFlags`, `ReadRegisterAsUnsigned`.
- **CN**: 声明与 `GetReturnAddress`, `GetFlags`, `ReadRegisterAsUnsigned` 相关的 API。

### Lines 157-166
```cpp
  Status WriteRegisterFromUnsigned(uint32_t reg, uint64_t uval);

  Status WriteRegisterFromUnsigned(const RegisterInfo *reg_info, uint64_t uval);

  // uint32_t
  // GetStopID () const
  // {
  //     return m_stop_id;
  // }

```
- **EN**: Implements logic around `WriteRegisterFromUnsigned`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `WriteRegisterFromUnsigned` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 167-173
```cpp
  // void
  // SetStopID (uint32_t stop_id)
  // {
  //     m_stop_id = stop_id;
  // }

protected:
```
- **EN**: Transitions between access-control regions inside a class or struct definition.
- **CN**: 在类或结构体定义内部切换访问控制区域。

### Lines 174-180
```cpp
  // Classes that inherit from RegisterContext can see and modify these
  NativeThreadProtocol
      &m_thread; // The thread that this register context belongs to.
  // uint32_t m_stop_id;             // The stop ID that any data in this
  // context is valid for

private:
```
- **EN**: Transitions between access-control regions inside a class or struct definition.
- **CN**: 在类或结构体定义内部切换访问控制区域。

### Lines 181-188
```cpp
  // For RegisterContext only
  NativeRegisterContext(const NativeRegisterContext &) = delete;
  const NativeRegisterContext &
  operator=(const NativeRegisterContext &) = delete;
};

} // namespace lldb_private

```
- **EN**: Declares APIs around `NativeRegisterContext`.
- **CN**: 声明与 `NativeRegisterContext` 相关的 API。

### Lines 189-189
```cpp
#endif // LLDB_HOST_COMMON_NATIVEREGISTERCONTEXT_H
```
- **EN**: Defines preprocessor-controlled structure, feature gates, or include-guard state.
- **CN**: 定义受预处理器控制的结构、特性开关或头文件保护状态。

## Key Concepts / 关键概念

- **LLDB architecture / LLDB 架构**:
  - **EN**: Explains how LLDB organizes debugger APIs, runtime objects, and platform-specific helpers.
  - **CN**: 说明 LLDB 如何组织调试器 API、运行时对象以及平台相关辅助组件。
- **Host abstraction / 主机抽象层**:
  - **EN**: Separates debugger logic from OS-specific file, process, terminal, and socket details.
  - **CN**: 将调试器逻辑与操作系统相关的文件、进程、终端和套接字细节解耦。
- **Stop control / 停机控制**:
  - **EN**: Represents stop reasons, conditions, callbacks, and breakpoint resolution state.
  - **CN**: 表示停机原因、条件、回调以及断点解析状态。
- **Symbolication / 符号化**:
  - **EN**: Relates raw addresses back to modules, sections, symbols, and source locations.
  - **CN**: 将原始地址关联回模块、节区、符号与源码位置。
- **Runtime control / 运行时控制**:
  - **EN**: Models debugger ownership over targets, processes, threads, and event delivery.
  - **CN**: 建模调试器对目标、进程、线程以及事件投递的控制。

## Dependencies / 依赖关系

- **Direct LLDB/LLVM includes / 直接的 LLDB/LLVM 包含**: `lldb/Host/common/NativeWatchpointList.h`, `lldb/lldb-private.h`
- **Subsystem categories / 子系统类别**: host OS abstraction helpers / 主机操作系统抽象辅助组件 (1), shared LLDB public/private type declarations / 共享 LLDB 公共/私有类型声明 (1)
