# NativeProcessSoftwareSingleStep.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/source/Plugins/Process/Utility/NativeProcessSoftwareSingleStep.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements the LLDB support logic associated with `NativeProcessSoftwareSingleStep`.
  - **CN**: 实现与 `NativeProcessSoftwareSingleStep` 相关的 LLDB 支持逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- NativeProcessSoftwareSingleStep.cpp -------------------------------===//
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

#include "NativeProcessSoftwareSingleStep.h"

#include "lldb/Core/EmulateInstruction.h"
#include "lldb/Host/common/NativeRegisterContext.h"
#include "lldb/Utility/RegisterValue.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `NativeProcessSoftwareSingleStep.h`, `lldb/Core/EmulateInstruction.h`, `lldb/Host/common/NativeRegisterContext.h`, `lldb/Utility/RegisterValue.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `NativeProcessSoftwareSingleStep.h`, `lldb/Core/EmulateInstruction.h`, `lldb/Host/common/NativeRegisterContext.h`, `lldb/Utility/RegisterValue.h`。

### Lines 15-19
```cpp
#include <unordered_map>

using namespace lldb;
using namespace lldb_private;

```
- **EN**: Pulls in the headers needed by this translation unit, including `unordered_map`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `unordered_map`。

### Lines 20-25
```cpp
namespace {

struct EmulatorBaton {
  NativeProcessProtocol &m_process;
  NativeRegisterContext &m_reg_context;

```
- **EN**: Introduces declarations for `EmulatorBaton`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `EmulatorBaton` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 26-33
```cpp
  // eRegisterKindDWARF -> RegsiterValue
  std::unordered_map<uint32_t, RegisterValue> m_register_values;

  EmulatorBaton(NativeProcessProtocol &process,
                NativeRegisterContext &reg_context)
      : m_process(process), m_reg_context(reg_context) {}
};

```
- **EN**: Implements logic around `EmulatorBaton`, `m_process`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `EmulatorBaton`, `m_process` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 34-40
```cpp
} // anonymous namespace

static size_t ReadMemoryCallback(EmulateInstruction *instruction, void *baton,
                                 const EmulateInstruction::Context &context,
                                 lldb::addr_t addr, void *dst, size_t length) {
  EmulatorBaton *emulator_baton = static_cast<EmulatorBaton *>(baton);

```
- **EN**: Implements logic around `ReadMemoryCallback`; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `ReadMemoryCallback` 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 41-45
```cpp
  size_t bytes_read;
  emulator_baton->m_process.ReadMemory(addr, dst, length, bytes_read);
  return bytes_read;
}

```
- **EN**: Implements logic around `ReadMemory`.
- **CN**: 围绕 `ReadMemory` 实现具体逻辑。

### Lines 46-50
```cpp
static bool ReadRegisterCallback(EmulateInstruction *instruction, void *baton,
                                 const RegisterInfo *reg_info,
                                 RegisterValue &reg_value) {
  EmulatorBaton *emulator_baton = static_cast<EmulatorBaton *>(baton);

```
- **EN**: Implements logic around `ReadRegisterCallback`; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `ReadRegisterCallback` 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 51-57
```cpp
  auto it = emulator_baton->m_register_values.find(
      reg_info->kinds[eRegisterKindDWARF]);
  if (it != emulator_baton->m_register_values.end()) {
    reg_value = it->second;
    return true;
  }

```
- **EN**: Implements logic around `find`, `end`.
- **CN**: 围绕 `find`, `end` 实现具体逻辑。

### Lines 58-64
```cpp
  // The emulator only fill in the dwarf regsiter numbers (and in some case the
  // generic register numbers). Get the full register info from the register
  // context based on the dwarf register numbers.
  const RegisterInfo *full_reg_info =
      emulator_baton->m_reg_context.GetRegisterInfo(
          eRegisterKindDWARF, reg_info->kinds[eRegisterKindDWARF]);

```
- **EN**: Implements logic around `GetRegisterInfo`; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `GetRegisterInfo` 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 65-69
```cpp
  Status error =
      emulator_baton->m_reg_context.ReadRegister(full_reg_info, reg_value);
  if (error.Success())
    return true;

```
- **EN**: Implements logic around `ReadRegister`, `Success`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `ReadRegister`, `Success` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 70-79
```cpp
  return false;
}

static bool WriteRegisterCallback(EmulateInstruction *instruction, void *baton,
                                  const EmulateInstruction::Context &context,
                                  const RegisterInfo *reg_info,
                                  const RegisterValue &reg_value) {
  EmulatorBaton *emulator_baton = static_cast<EmulatorBaton *>(baton);
  emulator_baton->m_register_values[reg_info->kinds[eRegisterKindDWARF]] =
      reg_value;
```
- **EN**: Implements logic around `WriteRegisterCallback`; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `WriteRegisterCallback` 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 80-89
```cpp
  return true;
}

static size_t WriteMemoryCallback(EmulateInstruction *instruction, void *baton,
                                  const EmulateInstruction::Context &context,
                                  lldb::addr_t addr, const void *dst,
                                  size_t length) {
  return length;
}

```
- **EN**: Implements logic around `WriteMemoryCallback`; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `WriteMemoryCallback` 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 90-95
```cpp
Status NativeProcessSoftwareSingleStep::SetupSoftwareSingleStepping(
    NativeThreadProtocol &thread) {
  NativeProcessProtocol &process = thread.GetProcess();
  NativeRegisterContext &register_context = thread.GetRegisterContext();
  const ArchSpec &arch = process.GetArchitecture();

```
- **EN**: Implements logic around `SetupSoftwareSingleStepping`, `GetProcess`, `GetRegisterContext`, `GetArchitecture`; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `SetupSoftwareSingleStepping`, `GetProcess`, `GetRegisterContext`, `GetArchitecture` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 96-101
```cpp
  std::unique_ptr<EmulateInstruction> emulator_up(
      EmulateInstruction::FindPlugin(arch, eInstructionTypePCModifying,
                                     nullptr));
  if (emulator_up == nullptr)
    return Status::FromErrorString("Instruction emulator not found!");

```
- **EN**: Implements logic around `emulator_up`, `FindPlugin`, `FromErrorString`; this block propagates recoverable errors, status objects, or diagnostics; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `emulator_up`, `FindPlugin`, `FromErrorString` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 102-108
```cpp
  EmulatorBaton baton(process, register_context);
  emulator_up->SetBaton(&baton);
  emulator_up->SetReadMemCallback(&ReadMemoryCallback);
  emulator_up->SetReadRegCallback(&ReadRegisterCallback);
  emulator_up->SetWriteMemCallback(&WriteMemoryCallback);
  emulator_up->SetWriteRegCallback(&WriteRegisterCallback);

```
- **EN**: Implements logic around `baton`, `SetBaton`, `SetReadMemCallback`, `SetReadRegCallback`, and 2 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `baton`, `SetBaton`, `SetReadMemCallback`, `SetReadRegCallback`, and 2 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 109-117
```cpp
  auto bp_locations_predictor =
      EmulateInstruction::CreateBreakpointLocationPredictor(
          std::move(emulator_up));

  llvm::Expected<BreakpointLocations> candidates =
      bp_locations_predictor->GetBreakpointLocations();
  if (!candidates)
    return Status::FromError(candidates.takeError());

```
- **EN**: Implements logic around `CreateBreakpointLocationPredictor`, `move`, `GetBreakpointLocations`, `FromError`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `CreateBreakpointLocationPredictor`, `move`, `GetBreakpointLocations`, `FromError` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 118-125
```cpp
  for (addr_t bp_addr : *candidates) {
    if (process.HasSoftwareBreakpoint(bp_addr))
      continue;
    llvm::Expected<unsigned> bp_size =
        bp_locations_predictor->GetBreakpointSize(bp_addr);
    if (auto err = bp_size.takeError())
      return Status::FromError(std::move(err));

```
- **EN**: Implements logic around `HasSoftwareBreakpoint`, `GetBreakpointSize`, `takeError`, `FromError`; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `HasSoftwareBreakpoint`, `GetBreakpointSize`, `takeError`, `FromError` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 126-134
```cpp
    Status error = process.SetBreakpoint(bp_addr, *bp_size, /*hardware=*/false);

    // If setting the breakpoint fails because pc is out of the address
    // space, ignore it and let the debugee segfault.
    if (error.GetError() == EIO || error.GetError() == EFAULT)
      continue;
    if (error.Fail())
      return error;

```
- **EN**: Implements logic around `SetBreakpoint`, `GetError`, `Fail`; this block propagates recoverable errors, status objects, or diagnostics; tracks breakpoint state, stop conditions, or hit-processing policy; coordinates debugger runtime objects, events, or asynchronous control flow; maps executable state back to modules, symbols, sections, or addresses; defines user-visible settings, options, or policy flags.
- **CN**: 围绕 `SetBreakpoint`, `GetError`, `Fail` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并跟踪断点状态、停机条件或命中处理策略，并协调调试器运行时对象、事件或异步控制流，并将可执行状态映射回模块、符号、节区或地址，并定义用户可见的设置、选项或策略标志。

### Lines 135-141
```cpp
    m_step_breakpoints.emplace(bp_addr);
  }

  m_threads_stepping_with_breakpoint.emplace(thread.GetID(),
                                             std::move(*candidates));
  return Status();
}
```
- **EN**: Implements logic around `emplace`, `move`, `Status`; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `emplace`, `move`, `Status` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

## Key Concepts / 关键概念

- **LLDB architecture / LLDB 架构**:
  - **EN**: Explains how LLDB organizes debugger APIs, runtime objects, and platform-specific helpers.
  - **CN**: 说明 LLDB 如何组织调试器 API、运行时对象以及平台相关辅助组件。
- **Stop control / 停机控制**:
  - **EN**: Represents stop reasons, conditions, callbacks, and breakpoint resolution state.
  - **CN**: 表示停机原因、条件、回调以及断点解析状态。
- **Symbolication / 符号化**:
  - **EN**: Relates raw addresses back to modules, sections, symbols, and source locations.
  - **CN**: 将原始地址关联回模块、节区、符号与源码位置。
- **Runtime control / 运行时控制**:
  - **EN**: Models debugger ownership over targets, processes, threads, and event delivery.
  - **CN**: 建模调试器对目标、进程、线程以及事件投递的控制。
- **OS integration / 操作系统集成**:
  - **EN**: Bridges LLDB abstractions to host files, sockets, terminals, and native process facilities.
  - **CN**: 将 LLDB 抽象连接到主机文件、套接字、终端与本地进程设施。

## Dependencies / 依赖关系

- **Direct LLDB/LLVM includes / 直接的 LLDB/LLVM 包含**: `NativeProcessSoftwareSingleStep.h`, `lldb/Core/EmulateInstruction.h`, `lldb/Host/common/NativeRegisterContext.h`, `lldb/Utility/RegisterValue.h`
- **Standard-library headers / 标准库头文件**: `<unordered_map>`
- **Subsystem categories / 子系统类别**: LLDB core debugger abstractions / LLDB 核心调试抽象 (1), host OS abstraction helpers / 主机操作系统抽象辅助组件 (1), shared LLDB utility classes / 共享 LLDB 工具类 (1)
