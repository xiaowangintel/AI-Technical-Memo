# EmulationStateARM.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/source/Plugins/Instruction/ARM/EmulationStateARM.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements the LLDB support logic associated with `EmulationStateARM`.
  - **CN**: 实现与 `EmulationStateARM` 相关的 LLDB 支持逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-10
```cpp
//===-- EmulationStateARM.cpp ---------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "EmulationStateARM.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `EmulationStateARM.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `EmulationStateARM.h`。

### Lines 11-22
```cpp
#include "lldb/Interpreter/OptionValueArray.h"
#include "lldb/Interpreter/OptionValueDictionary.h"
#include "lldb/Target/RegisterContext.h"
#include "lldb/Target/StackFrame.h"
#include "lldb/Utility/RegisterValue.h"
#include "lldb/Utility/Scalar.h"

#include "Utility/ARM_DWARF_Registers.h"

using namespace lldb;
using namespace lldb_private;

```
- **EN**: Pulls in the headers needed by this translation unit, including `lldb/Interpreter/OptionValueArray.h`, `lldb/Interpreter/OptionValueDictionary.h`, `lldb/Target/RegisterContext.h`, `lldb/Target/StackFrame.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `lldb/Interpreter/OptionValueArray.h`, `lldb/Interpreter/OptionValueDictionary.h`, `lldb/Target/RegisterContext.h`, `lldb/Target/StackFrame.h`。

### Lines 23-42
```cpp
EmulationStateARM::EmulationStateARM() : m_vfp_regs(), m_memory() {
  ClearPseudoRegisters();
}

EmulationStateARM::~EmulationStateARM() = default;

bool EmulationStateARM::StorePseudoRegisterValue(uint32_t reg_num,
                                                 uint64_t value) {
  if (reg_num <= dwarf_cpsr)
    m_gpr[reg_num - dwarf_r0] = (uint32_t)value;
  else if ((dwarf_s0 <= reg_num) && (reg_num <= dwarf_s31)) {
    uint32_t idx = reg_num - dwarf_s0;
    m_vfp_regs.s_regs[idx] = (uint32_t)value;
  } else if ((dwarf_d0 <= reg_num) && (reg_num <= dwarf_d31)) {
    uint32_t idx = reg_num - dwarf_d0;
    if (idx < 16) {
      m_vfp_regs.s_regs[idx * 2] = (uint32_t)value;
      m_vfp_regs.s_regs[idx * 2 + 1] = (uint32_t)(value >> 32);
    } else
      m_vfp_regs.d_regs[idx - 16] = value;
```
- **EN**: Implements logic around `EmulationStateARM`, `ClearPseudoRegisters`, `~EmulationStateARM`, `StorePseudoRegisterValue`.
- **CN**: 围绕 `EmulationStateARM`, `ClearPseudoRegisters`, `~EmulationStateARM`, `StorePseudoRegisterValue` 实现具体逻辑。

### Lines 43-53
```cpp
  } else
    return false;

  return true;
}

uint64_t EmulationStateARM::ReadPseudoRegisterValue(uint32_t reg_num,
                                                    bool &success) {
  uint64_t value = 0;
  success = true;

```
- **EN**: Implements logic around `ReadPseudoRegisterValue`.
- **CN**: 围绕 `ReadPseudoRegisterValue` 实现具体逻辑。

### Lines 54-68
```cpp
  if (reg_num <= dwarf_cpsr)
    value = m_gpr[reg_num - dwarf_r0];
  else if ((dwarf_s0 <= reg_num) && (reg_num <= dwarf_s31)) {
    uint32_t idx = reg_num - dwarf_s0;
    value = m_vfp_regs.s_regs[idx];
  } else if ((dwarf_d0 <= reg_num) && (reg_num <= dwarf_d31)) {
    uint32_t idx = reg_num - dwarf_d0;
    if (idx < 16)
      value = (uint64_t)m_vfp_regs.s_regs[idx * 2] |
              ((uint64_t)m_vfp_regs.s_regs[idx * 2 + 1] << 32);
    else
      value = m_vfp_regs.d_regs[idx - 16];
  } else
    success = false;

```
- **EN**: Contains supporting implementation details for the surrounding LLDB subsystem.
- **CN**: 包含周边 LLDB 子系统所需的辅助实现细节。

### Lines 69-78
```cpp
  return value;
}

void EmulationStateARM::ClearPseudoRegisters() {
  for (int i = 0; i < 17; ++i)
    m_gpr[i] = 0;

  for (int i = 0; i < 32; ++i)
    m_vfp_regs.s_regs[i] = 0;

```
- **EN**: Implements logic around `ClearPseudoRegisters`.
- **CN**: 围绕 `ClearPseudoRegisters` 实现具体逻辑。

### Lines 79-90
```cpp
  for (int i = 0; i < 16; ++i)
    m_vfp_regs.d_regs[i] = 0;
}

void EmulationStateARM::ClearPseudoMemory() { m_memory.clear(); }

bool EmulationStateARM::StoreToPseudoAddress(lldb::addr_t p_address,
                                             uint32_t value) {
  m_memory[p_address] = value;
  return true;
}

```
- **EN**: Implements logic around `ClearPseudoMemory`, `StoreToPseudoAddress`.
- **CN**: 围绕 `ClearPseudoMemory`, `StoreToPseudoAddress` 实现具体逻辑。

### Lines 91-102
```cpp
uint32_t EmulationStateARM::ReadFromPseudoAddress(lldb::addr_t p_address,
                                                  bool &success) {
  std::map<lldb::addr_t, uint32_t>::iterator pos;
  uint32_t ret_val = 0;

  success = true;
  pos = m_memory.find(p_address);
  if (pos != m_memory.end())
    ret_val = pos->second;
  else
    success = false;

```
- **EN**: Implements logic around `ReadFromPseudoAddress`, `find`, `end`.
- **CN**: 围绕 `ReadFromPseudoAddress`, `find`, `end` 实现具体逻辑。

### Lines 103-112
```cpp
  return ret_val;
}

size_t EmulationStateARM::ReadPseudoMemory(
    EmulateInstruction *instruction, void *baton,
    const EmulateInstruction::Context &context, lldb::addr_t addr, void *dst,
    size_t length) {
  if (!baton)
    return 0;

```
- **EN**: Implements logic around `ReadPseudoMemory`; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `ReadPseudoMemory` 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 113-127
```cpp
  bool success = true;
  EmulationStateARM *pseudo_state = (EmulationStateARM *)baton;
  if (length <= 4) {
    uint32_t value = pseudo_state->ReadFromPseudoAddress(addr, success);
    if (!success)
      return 0;

    if (endian::InlHostByteOrder() == lldb::eByteOrderBig)
      value = llvm::byteswap<uint32_t>(value);
    *((uint32_t *)dst) = value;
  } else if (length == 8) {
    uint32_t value1 = pseudo_state->ReadFromPseudoAddress(addr, success);
    if (!success)
      return 0;

```
- **EN**: Implements logic around `ReadFromPseudoAddress`, `InlHostByteOrder`, `byteswap`.
- **CN**: 围绕 `ReadFromPseudoAddress`, `InlHostByteOrder`, `byteswap` 实现具体逻辑。

### Lines 128-140
```cpp
    uint32_t value2 = pseudo_state->ReadFromPseudoAddress(addr + 4, success);
    if (!success)
      return 0;

    if (endian::InlHostByteOrder() == lldb::eByteOrderBig) {
      value1 = llvm::byteswap<uint32_t>(value1);
      value2 = llvm::byteswap<uint32_t>(value2);
    }
    ((uint32_t *)dst)[0] = value1;
    ((uint32_t *)dst)[1] = value2;
  } else
    success = false;

```
- **EN**: Implements logic around `ReadFromPseudoAddress`, `InlHostByteOrder`, `byteswap`.
- **CN**: 围绕 `ReadFromPseudoAddress`, `InlHostByteOrder`, `byteswap` 实现具体逻辑。

### Lines 141-153
```cpp
  if (success)
    return length;

  return 0;
}

size_t EmulationStateARM::WritePseudoMemory(
    EmulateInstruction *instruction, void *baton,
    const EmulateInstruction::Context &context, lldb::addr_t addr,
    const void *dst, size_t length) {
  if (!baton)
    return 0;

```
- **EN**: Implements logic around `WritePseudoMemory`; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `WritePseudoMemory` 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 154-173
```cpp
  EmulationStateARM *pseudo_state = (EmulationStateARM *)baton;

  if (length <= 4) {
    uint32_t value;
    memcpy (&value, dst, sizeof (uint32_t));
    if (endian::InlHostByteOrder() == lldb::eByteOrderBig)
      value = llvm::byteswap<uint32_t>(value);

    pseudo_state->StoreToPseudoAddress(addr, value);
    return length;
  } else if (length == 8) {
    uint32_t value1;
    uint32_t value2;
    memcpy (&value1, dst, sizeof (uint32_t));
    memcpy(&value2, static_cast<const uint8_t *>(dst) + sizeof(uint32_t),
           sizeof(uint32_t));
    if (endian::InlHostByteOrder() == lldb::eByteOrderBig) {
      value1 = llvm::byteswap<uint32_t>(value1);
      value2 = llvm::byteswap<uint32_t>(value2);
    }
```
- **EN**: Implements logic around `memcpy`, `InlHostByteOrder`, `byteswap`, `StoreToPseudoAddress`.
- **CN**: 围绕 `memcpy`, `InlHostByteOrder`, `byteswap`, `StoreToPseudoAddress` 实现具体逻辑。

### Lines 174-189
```cpp

    pseudo_state->StoreToPseudoAddress(addr, value1);
    pseudo_state->StoreToPseudoAddress(addr + 4, value2);
    return length;
  }

  return 0;
}

bool EmulationStateARM::ReadPseudoRegister(
    EmulateInstruction *instruction, void *baton,
    const lldb_private::RegisterInfo *reg_info,
    lldb_private::RegisterValue &reg_value) {
  if (!baton || !reg_info)
    return false;

```
- **EN**: Implements logic around `StoreToPseudoAddress`, `ReadPseudoRegister`; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `StoreToPseudoAddress`, `ReadPseudoRegister` 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 190-201
```cpp
  bool success = true;
  EmulationStateARM *pseudo_state = (EmulationStateARM *)baton;
  const uint32_t dwarf_reg_num = reg_info->kinds[eRegisterKindDWARF];
  assert(dwarf_reg_num != LLDB_INVALID_REGNUM);
  uint64_t reg_uval =
      pseudo_state->ReadPseudoRegisterValue(dwarf_reg_num, success);

  if (success)
    success = reg_value.SetUInt(reg_uval, reg_info->byte_size);
  return success;
}

```
- **EN**: Implements logic around `assert`, `ReadPseudoRegisterValue`, `SetUInt`.
- **CN**: 围绕 `assert`, `ReadPseudoRegisterValue`, `SetUInt` 实现具体逻辑。

### Lines 202-216
```cpp
bool EmulationStateARM::WritePseudoRegister(
    EmulateInstruction *instruction, void *baton,
    const EmulateInstruction::Context &context,
    const lldb_private::RegisterInfo *reg_info,
    const lldb_private::RegisterValue &reg_value) {
  if (!baton || !reg_info)
    return false;

  EmulationStateARM *pseudo_state = (EmulationStateARM *)baton;
  const uint32_t dwarf_reg_num = reg_info->kinds[eRegisterKindDWARF];
  assert(dwarf_reg_num != LLDB_INVALID_REGNUM);
  return pseudo_state->StorePseudoRegisterValue(dwarf_reg_num,
                                                reg_value.GetAsUInt64());
}

```
- **EN**: Implements logic around `WritePseudoRegister`, `assert`, `StorePseudoRegisterValue`, `GetAsUInt64`; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `WritePseudoRegister`, `assert`, `StorePseudoRegisterValue`, `GetAsUInt64` 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 217-228
```cpp
bool EmulationStateARM::CompareState(EmulationStateARM &other_state,
                                     Stream &out_stream) {
  bool match = true;

  for (int i = 0; match && i < 17; ++i) {
    if (m_gpr[i] != other_state.m_gpr[i]) {
      match = false;
      out_stream.Printf("r%d: 0x%x != 0x%x\n", i, m_gpr[i],
                        other_state.m_gpr[i]);
    }
  }

```
- **EN**: Implements logic around `CompareState`, `Printf`.
- **CN**: 围绕 `CompareState`, `Printf` 实现具体逻辑。

### Lines 229-244
```cpp
  for (int i = 0; match && i < 32; ++i) {
    if (m_vfp_regs.s_regs[i] != other_state.m_vfp_regs.s_regs[i]) {
      match = false;
      out_stream.Printf("s%d: 0x%x != 0x%x\n", i, m_vfp_regs.s_regs[i],
                        other_state.m_vfp_regs.s_regs[i]);
    }
  }

  for (int i = 0; match && i < 16; ++i) {
    if (m_vfp_regs.d_regs[i] != other_state.m_vfp_regs.d_regs[i]) {
      match = false;
      out_stream.Printf("d%d: 0x%" PRIx64 " != 0x%" PRIx64 "\n", i + 16,
                        m_vfp_regs.d_regs[i], other_state.m_vfp_regs.d_regs[i]);
    }
  }

```
- **EN**: Implements logic around `Printf`.
- **CN**: 围绕 `Printf` 实现具体逻辑。

### Lines 245-256
```cpp
  // other_state is the expected state. If it has memory, check it.
  if (!other_state.m_memory.empty() && m_memory != other_state.m_memory) {
    match = false;
    out_stream.Printf("memory does not match\n");
    out_stream.Printf("got memory:\n");
    for (auto p : m_memory)
      out_stream.Printf("0x%08" PRIx64 ": 0x%08x\n", p.first, p.second);
    out_stream.Printf("expected memory:\n");
    for (auto p : other_state.m_memory)
      out_stream.Printf("0x%08" PRIx64 ": 0x%08x\n", p.first, p.second);
  }

```
- **EN**: Implements logic around `empty`, `Printf`; this block propagates recoverable errors, status objects, or diagnostics; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `empty`, `Printf` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 257-272
```cpp
  return match;
}

bool EmulationStateARM::LoadRegistersStateFromDictionary(
    OptionValueDictionary *reg_dict, char kind, int first_reg, int num) {
  StreamString sstr;
  for (int i = 0; i < num; ++i) {
    sstr.Clear();
    sstr.Printf("%c%d", kind, i);
    OptionValueSP value_sp = reg_dict->GetValueForKey(sstr.GetString());
    if (value_sp.get() == nullptr)
      return false;
    uint64_t reg_value = value_sp->GetValueAs<uint64_t>().value_or(0);
    StorePseudoRegisterValue(first_reg + i, reg_value);
  }

```
- **EN**: Implements logic around `LoadRegistersStateFromDictionary`, `Clear`, `Printf`, `GetValueForKey`, and 3 more symbols.
- **CN**: 围绕 `LoadRegistersStateFromDictionary`, `Clear`, `Printf`, `GetValueForKey`, and 3 more symbols 实现具体逻辑。

### Lines 273-283
```cpp
  return true;
}

bool EmulationStateARM::LoadStateFromDictionary(
    OptionValueDictionary *test_data) {
  static constexpr llvm::StringLiteral memory_key("memory");
  static constexpr llvm::StringLiteral registers_key("registers");

  if (!test_data)
    return false;

```
- **EN**: Implements logic around `LoadStateFromDictionary`, `memory_key`, `registers_key`; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `LoadStateFromDictionary`, `memory_key`, `registers_key` 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 284-299
```cpp
  OptionValueSP value_sp = test_data->GetValueForKey(memory_key);

  // Load memory, if present.

  if (value_sp.get() != nullptr) {
    static constexpr llvm::StringLiteral address_key("address");
    static constexpr llvm::StringLiteral data_key("data");
    uint64_t start_address = 0;

    OptionValueDictionary *mem_dict = value_sp->GetAsDictionary();
    value_sp = mem_dict->GetValueForKey(address_key);
    if (value_sp.get() == nullptr)
      return false;
    else
      start_address = value_sp->GetValueAs<uint64_t>().value_or(0);

```
- **EN**: Implements logic around `GetValueForKey`, `get`, `address_key`, `data_key`, and 2 more symbols; this block maps executable state back to modules, symbols, sections, or addresses; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `GetValueForKey`, `get`, `address_key`, `data_key`, and 2 more symbols 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 300-317
```cpp
    value_sp = mem_dict->GetValueForKey(data_key);
    OptionValueArray *mem_array = value_sp->GetAsArray();
    if (!mem_array)
      return false;

    uint32_t num_elts = mem_array->GetSize();
    uint32_t address = (uint32_t)start_address;

    for (uint32_t i = 0; i < num_elts; ++i) {
      value_sp = mem_array->GetValueAtIndex(i);
      if (value_sp.get() == nullptr)
        return false;
      uint64_t value = value_sp->GetValueAs<uint64_t>().value_or(0);
      StoreToPseudoAddress(address, value);
      address = address + 4;
    }
  }

```
- **EN**: Implements logic around `GetValueForKey`, `GetAsArray`, `GetSize`, `GetValueAtIndex`, and 3 more symbols; this block maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `GetValueForKey`, `GetAsArray`, `GetSize`, `GetValueAtIndex`, and 3 more symbols 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址。

### Lines 318-327
```cpp
  value_sp = test_data->GetValueForKey(registers_key);
  if (value_sp.get() == nullptr)
    return false;

  // Load General Registers

  OptionValueDictionary *reg_dict = value_sp->GetAsDictionary();
  if (!LoadRegistersStateFromDictionary(reg_dict, 'r', dwarf_r0, 16))
    return false;

```
- **EN**: Implements logic around `GetValueForKey`, `get`, `GetAsDictionary`, `LoadRegistersStateFromDictionary`.
- **CN**: 围绕 `GetValueForKey`, `get`, `GetAsDictionary`, `LoadRegistersStateFromDictionary` 实现具体逻辑。

### Lines 328-343
```cpp
  static constexpr llvm::StringLiteral cpsr_name("cpsr");
  value_sp = reg_dict->GetValueForKey(cpsr_name);
  if (value_sp.get() == nullptr)
    return false;
  StorePseudoRegisterValue(dwarf_cpsr,
                           value_sp->GetValueAs<uint64_t>().value_or(0));

  // Load s/d Registers
  // To prevent you giving both types in a state and overwriting
  // one or the other, we'll expect to get either all S registers,
  // or all D registers. Not a mix of the two.
  bool found_s_registers =
      LoadRegistersStateFromDictionary(reg_dict, 's', dwarf_s0, 32);
  bool found_d_registers =
      LoadRegistersStateFromDictionary(reg_dict, 'd', dwarf_d0, 32);

```
- **EN**: Implements logic around `cpsr_name`, `GetValueForKey`, `get`, `StorePseudoRegisterValue`, and 2 more symbols.
- **CN**: 围绕 `cpsr_name`, `GetValueForKey`, `get`, `StorePseudoRegisterValue`, and 2 more symbols 实现具体逻辑。

### Lines 344-345
```cpp
  return found_s_registers != found_d_registers;
}
```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

## Key Concepts / 关键概念

- **LLDB architecture / LLDB 架构**:
  - **EN**: Explains how LLDB organizes debugger APIs, runtime objects, and platform-specific helpers.
  - **CN**: 说明 LLDB 如何组织调试器 API、运行时对象以及平台相关辅助组件。
- **Symbolication / 符号化**:
  - **EN**: Relates raw addresses back to modules, sections, symbols, and source locations.
  - **CN**: 将原始地址关联回模块、节区、符号与源码位置。
- **Runtime control / 运行时控制**:
  - **EN**: Models debugger ownership over targets, processes, threads, and event delivery.
  - **CN**: 建模调试器对目标、进程、线程以及事件投递的控制。

## Dependencies / 依赖关系

- **Direct LLDB/LLVM includes / 直接的 LLDB/LLVM 包含**: `EmulationStateARM.h`, `lldb/Interpreter/OptionValueArray.h`, `lldb/Interpreter/OptionValueDictionary.h`, `lldb/Target/RegisterContext.h`, `lldb/Target/StackFrame.h`, `lldb/Utility/RegisterValue.h`, `lldb/Utility/Scalar.h`, `Utility/ARM_DWARF_Registers.h`
- **Subsystem categories / 子系统类别**: command interpreter support / 命令解释器支持 (2), target, process, and thread control / 目标、进程与线程控制 (2), shared LLDB utility classes / 共享 LLDB 工具类 (2)
