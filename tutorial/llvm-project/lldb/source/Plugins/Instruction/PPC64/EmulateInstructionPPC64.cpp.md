# EmulateInstructionPPC64.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/source/Plugins/Instruction/PPC64/EmulateInstructionPPC64.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements the LLDB support logic associated with `EmulateInstructionPPC64`.
  - **CN**: 实现与 `EmulateInstructionPPC64` 相关的 LLDB 支持逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-10
```cpp
//===-- EmulateInstructionPPC64.cpp ---------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "EmulateInstructionPPC64.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `EmulateInstructionPPC64.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `EmulateInstructionPPC64.h`。

### Lines 11-20
```cpp
#include <cstdlib>
#include <optional>

#include "Plugins/Process/Utility/lldb-ppc64le-register-enums.h"
#include "lldb/Core/PluginManager.h"
#include "lldb/Symbol/UnwindPlan.h"
#include "lldb/Utility/ArchSpec.h"
#include "lldb/Utility/ConstString.h"
#include "lldb/Utility/LLDBLog.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `cstdlib`, `optional`, `Plugins/Process/Utility/lldb-ppc64le-register-enums.h`, `lldb/Core/PluginManager.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `cstdlib`, `optional`, `Plugins/Process/Utility/lldb-ppc64le-register-enums.h`, `lldb/Core/PluginManager.h`。

### Lines 21-30
```cpp
#define DECLARE_REGISTER_INFOS_PPC64LE_STRUCT
#include "Plugins/Process/Utility/RegisterInfos_ppc64le.h"

#include "Plugins/Process/Utility/InstructionUtils.h"

using namespace lldb;
using namespace lldb_private;

LLDB_PLUGIN_DEFINE_ADV(EmulateInstructionPPC64, InstructionPPC64)

```
- **EN**: Pulls in the headers needed by this translation unit, including `Plugins/Process/Utility/RegisterInfos_ppc64le.h`, `Plugins/Process/Utility/InstructionUtils.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `Plugins/Process/Utility/RegisterInfos_ppc64le.h`, `Plugins/Process/Utility/InstructionUtils.h`。

### Lines 31-42
```cpp
EmulateInstructionPPC64::EmulateInstructionPPC64(const ArchSpec &arch)
    : EmulateInstruction(arch) {}

void EmulateInstructionPPC64::Initialize() {
  PluginManager::RegisterPlugin(GetPluginNameStatic(),
                                GetPluginDescriptionStatic(), CreateInstance);
}

void EmulateInstructionPPC64::Terminate() {
  PluginManager::UnregisterPlugin(CreateInstance);
}

```
- **EN**: Implements logic around `EmulateInstructionPPC64`, `EmulateInstruction`, `Initialize`, `RegisterPlugin`, and 3 more symbols; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `EmulateInstructionPPC64`, `EmulateInstruction`, `Initialize`, `RegisterPlugin`, and 3 more symbols 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 43-54
```cpp
llvm::StringRef EmulateInstructionPPC64::GetPluginDescriptionStatic() {
  return "Emulate instructions for the PPC64 architecture.";
}

EmulateInstruction *
EmulateInstructionPPC64::CreateInstance(const ArchSpec &arch,
                                        InstructionType inst_type) {
  if (EmulateInstructionPPC64::SupportsEmulatingInstructionsOfTypeStatic(
          inst_type))
    if (arch.GetTriple().isPPC64())
      return new EmulateInstructionPPC64(arch);

```
- **EN**: Implements logic around `GetPluginDescriptionStatic`, `CreateInstance`, `SupportsEmulatingInstructionsOfTypeStatic`, `GetTriple`, and 1 more symbols; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `GetPluginDescriptionStatic`, `CreateInstance`, `SupportsEmulatingInstructionsOfTypeStatic`, `GetTriple`, and 1 more symbols 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 55-67
```cpp
  return nullptr;
}

bool EmulateInstructionPPC64::SetTargetTriple(const ArchSpec &arch) {
  return arch.GetTriple().isPPC64();
}

static std::optional<RegisterInfo> LLDBTableGetRegisterInfo(uint32_t reg_num) {
  if (reg_num >= std::size(g_register_infos_ppc64le))
    return {};
  return g_register_infos_ppc64le[reg_num];
}

```
- **EN**: Implements logic around `SetTargetTriple`, `GetTriple`, `LLDBTableGetRegisterInfo`, `size`; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `SetTargetTriple`, `GetTriple`, `LLDBTableGetRegisterInfo`, `size` 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 68-87
```cpp
std::optional<RegisterInfo>
EmulateInstructionPPC64::GetRegisterInfo(RegisterKind reg_kind,
                                         uint32_t reg_num) {
  if (reg_kind == eRegisterKindGeneric) {
    switch (reg_num) {
    case LLDB_REGNUM_GENERIC_PC:
      reg_kind = eRegisterKindLLDB;
      reg_num = gpr_pc_ppc64le;
      break;
    case LLDB_REGNUM_GENERIC_SP:
      reg_kind = eRegisterKindLLDB;
      reg_num = gpr_r1_ppc64le;
      break;
    case LLDB_REGNUM_GENERIC_RA:
      reg_kind = eRegisterKindLLDB;
      reg_num = gpr_lr_ppc64le;
      break;
    case LLDB_REGNUM_GENERIC_FLAGS:
      reg_kind = eRegisterKindLLDB;
      reg_num = gpr_cr_ppc64le;
```
- **EN**: Implements logic around `GetRegisterInfo`.
- **CN**: 围绕 `GetRegisterInfo` 实现具体逻辑。

### Lines 88-99
```cpp
      break;

    default:
      return {};
    }
  }

  if (reg_kind == eRegisterKindLLDB)
    return LLDBTableGetRegisterInfo(reg_num);
  return {};
}

```
- **EN**: Implements logic around `LLDBTableGetRegisterInfo`.
- **CN**: 围绕 `LLDBTableGetRegisterInfo` 实现具体逻辑。

### Lines 100-115
```cpp
bool EmulateInstructionPPC64::ReadInstruction() {
  bool success = false;
  m_addr = ReadRegisterUnsigned(eRegisterKindGeneric, LLDB_REGNUM_GENERIC_PC,
                                LLDB_INVALID_ADDRESS, &success);
  if (success) {
    Context ctx;
    ctx.type = eContextReadOpcode;
    ctx.SetNoArgs();
    m_opcode.SetOpcode32(ReadMemoryUnsigned(ctx, m_addr, 4, 0, &success),
                         GetByteOrder());
  }
  if (!success)
    m_addr = LLDB_INVALID_ADDRESS;
  return success;
}

```
- **EN**: Implements logic around `ReadInstruction`, `ReadRegisterUnsigned`, `SetNoArgs`, `SetOpcode32`, and 1 more symbols.
- **CN**: 围绕 `ReadInstruction`, `ReadRegisterUnsigned`, `SetNoArgs`, `SetOpcode32`, and 1 more symbols 实现具体逻辑。

### Lines 116-125
```cpp
bool EmulateInstructionPPC64::CreateFunctionEntryUnwind(
    UnwindPlan &unwind_plan) {
  unwind_plan.Clear();
  unwind_plan.SetRegisterKind(eRegisterKindLLDB);

  UnwindPlan::Row row;

  // Our previous Call Frame Address is the stack pointer
  row.GetCFAValue().SetIsRegisterPlusOffset(gpr_r1_ppc64le, 0);

```
- **EN**: Implements logic around `CreateFunctionEntryUnwind`, `Clear`, `SetRegisterKind`, `GetCFAValue`; this block maps executable state back to modules, symbols, sections, or addresses; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `CreateFunctionEntryUnwind`, `Clear`, `SetRegisterKind`, `GetCFAValue` 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 126-145
```cpp
  unwind_plan.AppendRow(std::move(row));
  unwind_plan.SetSourceName("EmulateInstructionPPC64");
  unwind_plan.SetSourcedFromCompiler(eLazyBoolNo);
  unwind_plan.SetUnwindPlanValidAtAllInstructions(eLazyBoolYes);
  unwind_plan.SetUnwindPlanForSignalTrap(eLazyBoolNo);
  unwind_plan.SetReturnAddressRegister(gpr_lr_ppc64le);
  return true;
}

EmulateInstructionPPC64::Opcode *
EmulateInstructionPPC64::GetOpcodeForInstruction(uint32_t opcode) {
  static EmulateInstructionPPC64::Opcode g_opcodes[] = {
      {0xfc0007ff, 0x7c0002a6, &EmulateInstructionPPC64::EmulateMFSPR,
       "mfspr RT, SPR"},
      {0xfc000003, 0xf8000000, &EmulateInstructionPPC64::EmulateSTD,
       "std RS, DS(RA)"},
      {0xfc000003, 0xf8000001, &EmulateInstructionPPC64::EmulateSTD,
       "stdu RS, DS(RA)"},
      {0xfc0007fe, 0x7c000378, &EmulateInstructionPPC64::EmulateOR,
       "or RA, RS, RB"},
```
- **EN**: Implements logic around `AppendRow`, `SetSourceName`, `SetSourcedFromCompiler`, `SetUnwindPlanValidAtAllInstructions`, and 4 more symbols.
- **CN**: 围绕 `AppendRow`, `SetSourceName`, `SetSourcedFromCompiler`, `SetUnwindPlanValidAtAllInstructions`, and 4 more symbols 实现具体逻辑。

### Lines 146-158
```cpp
      {0xfc000000, 0x38000000, &EmulateInstructionPPC64::EmulateADDI,
       "addi RT, RA, SI"},
      {0xfc000003, 0xe8000000, &EmulateInstructionPPC64::EmulateLD,
       "ld RT, DS(RA)"}};
  static const size_t k_num_ppc_opcodes = std::size(g_opcodes);

  for (size_t i = 0; i < k_num_ppc_opcodes; ++i) {
    if ((g_opcodes[i].mask & opcode) == g_opcodes[i].value)
      return &g_opcodes[i];
  }
  return nullptr;
}

```
- **EN**: Implements logic around `DS`, `size`.
- **CN**: 围绕 `DS`, `size` 实现具体逻辑。

### Lines 159-169
```cpp
bool EmulateInstructionPPC64::EvaluateInstruction(uint32_t evaluate_options) {
  const uint32_t opcode = m_opcode.GetOpcode32();
  // LLDB_LOG(log, "PPC64::EvaluateInstruction: opcode={0:X+8}", opcode);
  Opcode *opcode_data = GetOpcodeForInstruction(opcode);
  if (!opcode_data)
    return false;

  // LLDB_LOG(log, "PPC64::EvaluateInstruction: {0}", opcode_data->name);
  const bool auto_advance_pc =
      evaluate_options & eEmulateInstructionOptionAutoAdvancePC;

```
- **EN**: Implements logic around `EvaluateInstruction`, `GetOpcode32`, `GetOpcodeForInstruction`.
- **CN**: 围绕 `EvaluateInstruction`, `GetOpcode32`, `GetOpcodeForInstruction` 实现具体逻辑。

### Lines 170-179
```cpp
  bool success = false;

  uint32_t orig_pc_value = 0;
  if (auto_advance_pc) {
    orig_pc_value =
        ReadRegisterUnsigned(eRegisterKindLLDB, gpr_pc_ppc64le, 0, &success);
    if (!success)
      return false;
  }

```
- **EN**: Implements logic around `ReadRegisterUnsigned`.
- **CN**: 围绕 `ReadRegisterUnsigned` 实现具体逻辑。

### Lines 180-190
```cpp
  // Call the Emulate... function.
  success = (this->*opcode_data->callback)(opcode);
  if (!success)
    return false;

  if (auto_advance_pc) {
    uint32_t new_pc_value =
        ReadRegisterUnsigned(eRegisterKindLLDB, gpr_pc_ppc64le, 0, &success);
    if (!success)
      return false;

```
- **EN**: Implements logic around `ReadRegisterUnsigned`.
- **CN**: 围绕 `ReadRegisterUnsigned` 实现具体逻辑。

### Lines 191-202
```cpp
    if (new_pc_value == orig_pc_value) {
      EmulateInstruction::Context context;
      context.type = eContextAdvancePC;
      context.SetNoArgs();
      if (!WriteRegisterUnsigned(context, eRegisterKindLLDB, gpr_pc_ppc64le,
                                 orig_pc_value + 4))
        return false;
    }
  }
  return true;
}

```
- **EN**: Implements logic around `SetNoArgs`, `WriteRegisterUnsigned`.
- **CN**: 围绕 `SetNoArgs`, `WriteRegisterUnsigned` 实现具体逻辑。

### Lines 203-212
```cpp
bool EmulateInstructionPPC64::EmulateMFSPR(uint32_t opcode) {
  uint32_t rt = Bits32(opcode, 25, 21);
  uint32_t spr = Bits32(opcode, 20, 11);

  enum { SPR_LR = 0x100 };

  // For now, we're only insterested in 'mfspr r0, lr'
  if (rt != gpr_r0_ppc64le || spr != SPR_LR)
    return false;

```
- **EN**: Implements logic around `EmulateMFSPR`, `Bits32`.
- **CN**: 围绕 `EmulateMFSPR`, `Bits32` 实现具体逻辑。

### Lines 213-227
```cpp
  Log *log = GetLog(LLDBLog::Unwind);
  LLDB_LOG(log, "EmulateMFSPR: {0:X+8}: mfspr r0, lr", m_addr);

  bool success;
  uint64_t lr =
      ReadRegisterUnsigned(eRegisterKindLLDB, gpr_lr_ppc64le, 0, &success);
  if (!success)
    return false;
  Context context;
  context.type = eContextWriteRegisterRandomBits;
  WriteRegisterUnsigned(context, eRegisterKindLLDB, gpr_r0_ppc64le, lr);
  LLDB_LOG(log, "EmulateMFSPR: success!");
  return true;
}

```
- **EN**: Implements logic around `GetLog`, `LLDB_LOG`, `ReadRegisterUnsigned`, `WriteRegisterUnsigned`.
- **CN**: 围绕 `GetLog`, `LLDB_LOG`, `ReadRegisterUnsigned`, `WriteRegisterUnsigned` 实现具体逻辑。

### Lines 228-239
```cpp
bool EmulateInstructionPPC64::EmulateLD(uint32_t opcode) {
  uint32_t rt = Bits32(opcode, 25, 21);
  uint32_t ra = Bits32(opcode, 20, 16);
  uint32_t ds = Bits32(opcode, 15, 2);

  int32_t ids = llvm::SignExtend32<16>(ds << 2);

  // For now, tracking only loads from 0(r1) to r1 (0(r1) is the ABI defined
  // location to save previous SP)
  if (ra != gpr_r1_ppc64le || rt != gpr_r1_ppc64le || ids != 0)
    return false;

```
- **EN**: Implements logic around `EmulateLD`, `Bits32`, `SignExtend32`.
- **CN**: 围绕 `EmulateLD`, `Bits32`, `SignExtend32` 实现具体逻辑。

### Lines 240-252
```cpp
  Log *log = GetLog(LLDBLog::Unwind);
  LLDB_LOG(log, "EmulateLD: {0:X+8}: ld r{1}, {2}(r{3})", m_addr, rt, ids, ra);

  std::optional<RegisterInfo> r1_info =
      GetRegisterInfo(eRegisterKindLLDB, gpr_r1_ppc64le);
  if (!r1_info)
    return false;

  // restore SP
  Context ctx;
  ctx.type = eContextRestoreStackPointer;
  ctx.SetRegisterToRegisterPlusOffset(*r1_info, *r1_info, 0);

```
- **EN**: Implements logic around `GetLog`, `LLDB_LOG`, `GetRegisterInfo`, `SetRegisterToRegisterPlusOffset`.
- **CN**: 围绕 `GetLog`, `LLDB_LOG`, `GetRegisterInfo`, `SetRegisterToRegisterPlusOffset` 实现具体逻辑。

### Lines 253-263
```cpp
  WriteRegisterUnsigned(ctx, eRegisterKindLLDB, gpr_r1_ppc64le, 0);
  LLDB_LOG(log, "EmulateLD: success!");
  return true;
}

bool EmulateInstructionPPC64::EmulateSTD(uint32_t opcode) {
  uint32_t rs = Bits32(opcode, 25, 21);
  uint32_t ra = Bits32(opcode, 20, 16);
  uint32_t ds = Bits32(opcode, 15, 2);
  uint32_t u = Bits32(opcode, 1, 0);

```
- **EN**: Implements logic around `WriteRegisterUnsigned`, `LLDB_LOG`, `EmulateSTD`, `Bits32`.
- **CN**: 围绕 `WriteRegisterUnsigned`, `LLDB_LOG`, `EmulateSTD`, `Bits32` 实现具体逻辑。

### Lines 264-276
```cpp
  // For now, tracking only stores to r1
  if (ra != gpr_r1_ppc64le)
    return false;
  // ... and only stores of SP, FP and LR (moved into r0 by a previous mfspr)
  if (rs != gpr_r1_ppc64le && rs != gpr_r31_ppc64le && rs != gpr_r30_ppc64le &&
      rs != gpr_r0_ppc64le)
    return false;

  bool success;
  uint64_t rs_val = ReadRegisterUnsigned(eRegisterKindLLDB, rs, 0, &success);
  if (!success)
    return false;

```
- **EN**: Implements logic around `ReadRegisterUnsigned`.
- **CN**: 围绕 `ReadRegisterUnsigned` 实现具体逻辑。

### Lines 277-292
```cpp
  int32_t ids = llvm::SignExtend32<16>(ds << 2);
  Log *log = GetLog(LLDBLog::Unwind);
  LLDB_LOG(log, "EmulateSTD: {0:X+8}: std{1} r{2}, {3}(r{4})", m_addr,
           u ? "u" : "", rs, ids, ra);

  // Make sure that r0 is really holding LR value (this won't catch unlikely
  // cases, such as r0 being overwritten after mfspr)
  uint32_t rs_num = rs;
  if (rs == gpr_r0_ppc64le) {
    uint64_t lr =
        ReadRegisterUnsigned(eRegisterKindLLDB, gpr_lr_ppc64le, 0, &success);
    if (!success || lr != rs_val)
      return false;
    rs_num = gpr_lr_ppc64le;
  }

```
- **EN**: Implements logic around `SignExtend32`, `GetLog`, `LLDB_LOG`, `ReadRegisterUnsigned`.
- **CN**: 围绕 `SignExtend32`, `GetLog`, `LLDB_LOG`, `ReadRegisterUnsigned` 实现具体逻辑。

### Lines 293-305
```cpp
  // set context
  std::optional<RegisterInfo> rs_info =
      GetRegisterInfo(eRegisterKindLLDB, rs_num);
  if (!rs_info)
    return false;
  std::optional<RegisterInfo> ra_info = GetRegisterInfo(eRegisterKindLLDB, ra);
  if (!ra_info)
    return false;

  Context ctx;
  ctx.type = eContextPushRegisterOnStack;
  ctx.SetRegisterToRegisterPlusOffset(*rs_info, *ra_info, ids);

```
- **EN**: Implements logic around `GetRegisterInfo`, `SetRegisterToRegisterPlusOffset`.
- **CN**: 围绕 `GetRegisterInfo`, `SetRegisterToRegisterPlusOffset` 实现具体逻辑。

### Lines 306-321
```cpp
  // store
  uint64_t ra_val = ReadRegisterUnsigned(eRegisterKindLLDB, ra, 0, &success);
  if (!success)
    return false;

  lldb::addr_t addr = ra_val + ids;
  WriteMemory(ctx, addr, &rs_val, sizeof(rs_val));

  // update RA?
  if (u) {
    Context ctx;
    // NOTE Currently, RA will always be equal to SP(r1)
    ctx.type = eContextAdjustStackPointer;
    WriteRegisterUnsigned(ctx, eRegisterKindLLDB, ra, addr);
  }

```
- **EN**: Implements logic around `ReadRegisterUnsigned`, `WriteMemory`, `WriteRegisterUnsigned`.
- **CN**: 围绕 `ReadRegisterUnsigned`, `WriteMemory`, `WriteRegisterUnsigned` 实现具体逻辑。

### Lines 322-335
```cpp
  LLDB_LOG(log, "EmulateSTD: success!");
  return true;
}

bool EmulateInstructionPPC64::EmulateOR(uint32_t opcode) {
  uint32_t rs = Bits32(opcode, 25, 21);
  uint32_t ra = Bits32(opcode, 20, 16);
  uint32_t rb = Bits32(opcode, 15, 11);

  // to be safe, process only the known 'mr r31/r30, r1' prologue instructions
  if (m_fp != LLDB_INVALID_REGNUM || rs != rb ||
      (ra != gpr_r30_ppc64le && ra != gpr_r31_ppc64le) || rb != gpr_r1_ppc64le)
    return false;

```
- **EN**: Implements logic around `LLDB_LOG`, `EmulateOR`, `Bits32`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `LLDB_LOG`, `EmulateOR`, `Bits32` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 336-347
```cpp
  Log *log = GetLog(LLDBLog::Unwind);
  LLDB_LOG(log, "EmulateOR: {0:X+8}: mr r{1}, r{2}", m_addr, ra, rb);

  // set context
  std::optional<RegisterInfo> ra_info = GetRegisterInfo(eRegisterKindLLDB, ra);
  if (!ra_info)
    return false;

  Context ctx;
  ctx.type = eContextSetFramePointer;
  ctx.SetRegister(*ra_info);

```
- **EN**: Implements logic around `GetLog`, `LLDB_LOG`, `GetRegisterInfo`, `SetRegister`.
- **CN**: 围绕 `GetLog`, `LLDB_LOG`, `GetRegisterInfo`, `SetRegister` 实现具体逻辑。

### Lines 348-358
```cpp
  // move
  bool success;
  uint64_t rb_val = ReadRegisterUnsigned(eRegisterKindLLDB, rb, 0, &success);
  if (!success)
    return false;
  WriteRegisterUnsigned(ctx, eRegisterKindLLDB, ra, rb_val);
  m_fp = ra;
  LLDB_LOG(log, "EmulateOR: success!");
  return true;
}

```
- **EN**: Implements logic around `ReadRegisterUnsigned`, `WriteRegisterUnsigned`, `LLDB_LOG`.
- **CN**: 围绕 `ReadRegisterUnsigned`, `WriteRegisterUnsigned`, `LLDB_LOG` 实现具体逻辑。

### Lines 359-369
```cpp
bool EmulateInstructionPPC64::EmulateADDI(uint32_t opcode) {
  uint32_t rt = Bits32(opcode, 25, 21);
  uint32_t ra = Bits32(opcode, 20, 16);
  uint32_t si = Bits32(opcode, 15, 0);

  // handle stack adjustments only
  // (this is a typical epilogue operation, with ra == r1. If it's
  //  something else, then we won't know the correct value of ra)
  if (rt != gpr_r1_ppc64le || ra != gpr_r1_ppc64le)
    return false;

```
- **EN**: Implements logic around `EmulateADDI`, `Bits32`; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `EmulateADDI`, `Bits32` 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 370-379
```cpp
  int32_t si_val = llvm::SignExtend32<16>(si);
  Log *log = GetLog(LLDBLog::Unwind);
  LLDB_LOG(log, "EmulateADDI: {0:X+8}: addi r1, r1, {1}", m_addr, si_val);

  // set context
  std::optional<RegisterInfo> r1_info =
      GetRegisterInfo(eRegisterKindLLDB, gpr_r1_ppc64le);
  if (!r1_info)
    return false;

```
- **EN**: Implements logic around `SignExtend32`, `GetLog`, `LLDB_LOG`, `GetRegisterInfo`.
- **CN**: 围绕 `SignExtend32`, `GetLog`, `LLDB_LOG`, `GetRegisterInfo` 实现具体逻辑。

### Lines 380-393
```cpp
  Context ctx;
  ctx.type = eContextRestoreStackPointer;
  ctx.SetRegisterToRegisterPlusOffset(*r1_info, *r1_info, 0);

  // adjust SP
  bool success;
  uint64_t r1 =
      ReadRegisterUnsigned(eRegisterKindLLDB, gpr_r1_ppc64le, 0, &success);
  if (!success)
    return false;
  WriteRegisterUnsigned(ctx, eRegisterKindLLDB, gpr_r1_ppc64le, r1 + si_val);
  LLDB_LOG(log, "EmulateADDI: success!");
  return true;
}
```
- **EN**: Implements logic around `SetRegisterToRegisterPlusOffset`, `ReadRegisterUnsigned`, `WriteRegisterUnsigned`, `LLDB_LOG`.
- **CN**: 围绕 `SetRegisterToRegisterPlusOffset`, `ReadRegisterUnsigned`, `WriteRegisterUnsigned`, `LLDB_LOG` 实现具体逻辑。

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

- **Direct LLDB/LLVM includes / 直接的 LLDB/LLVM 包含**: `EmulateInstructionPPC64.h`, `Plugins/Process/Utility/lldb-ppc64le-register-enums.h`, `lldb/Core/PluginManager.h`, `lldb/Symbol/UnwindPlan.h`, `lldb/Utility/ArchSpec.h`, `lldb/Utility/ConstString.h`, `lldb/Utility/LLDBLog.h`, `Plugins/Process/Utility/RegisterInfos_ppc64le.h`, `Plugins/Process/Utility/InstructionUtils.h`
- **Standard-library headers / 标准库头文件**: `<cstdlib>`, `<optional>`
- **Subsystem categories / 子系统类别**: shared LLDB utility classes / 共享 LLDB 工具类 (3), LLDB core debugger abstractions / LLDB 核心调试抽象 (1), symbol and debug-info abstractions / 符号与调试信息抽象 (1)
