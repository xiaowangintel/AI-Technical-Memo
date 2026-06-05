# EmulateInstructionLoongArch.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/source/Plugins/Instruction/LoongArch/EmulateInstructionLoongArch.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements the LLDB support logic associated with `EmulateInstructionLoongArch`.
  - **CN**: 实现与 `EmulateInstructionLoongArch` 相关的 LLDB 支持逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===---EmulateInstructionLoongArch.cpp------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include <cstdlib>
#include <optional>

```
- **EN**: Pulls in the headers needed by this translation unit, including `cstdlib`, `optional`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `cstdlib`, `optional`。

### Lines 12-27
```cpp
#include "EmulateInstructionLoongArch.h"
#include "Plugins/Process/Utility/InstructionUtils.h"
#include "Plugins/Process/Utility/RegisterInfoPOSIX_loongarch64.h"
#include "Plugins/Process/Utility/lldb-loongarch-register-enums.h"
#include "lldb/Core/Address.h"
#include "lldb/Core/PluginManager.h"
#include "lldb/Interpreter/OptionValueArray.h"
#include "lldb/Interpreter/OptionValueDictionary.h"
#include "lldb/Symbol/UnwindPlan.h"
#include "lldb/Utility/ArchSpec.h"
#include "lldb/Utility/LLDBLog.h"
#include "lldb/Utility/RegisterValue.h"
#include "lldb/Utility/Stream.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/Support/MathExtras.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `EmulateInstructionLoongArch.h`, `Plugins/Process/Utility/InstructionUtils.h`, `Plugins/Process/Utility/RegisterInfoPOSIX_loongarch64.h`, `Plugins/Process/Utility/lldb-loongarch-register-enums.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `EmulateInstructionLoongArch.h`, `Plugins/Process/Utility/InstructionUtils.h`, `Plugins/Process/Utility/RegisterInfoPOSIX_loongarch64.h`, `Plugins/Process/Utility/lldb-loongarch-register-enums.h`。

### Lines 28-47
```cpp
using namespace lldb;
using namespace lldb_private;

LLDB_PLUGIN_DEFINE_ADV(EmulateInstructionLoongArch, InstructionLoongArch)

namespace lldb_private {

EmulateInstructionLoongArch::Opcode *
EmulateInstructionLoongArch::GetOpcodeForInstruction(uint32_t inst) {
  // TODO: Add the mask for other instruction.
  static EmulateInstructionLoongArch::Opcode g_opcodes[] = {
      {0xfc000000, 0x40000000, &EmulateInstructionLoongArch::EmulateBEQZ,
       "beqz rj, offs21"},
      {0xfc000000, 0x44000000, &EmulateInstructionLoongArch::EmulateBNEZ,
       "bnez rj, offs21"},
      {0xfc000300, 0x48000000, &EmulateInstructionLoongArch::EmulateBCEQZ,
       "bceqz cj, offs21"},
      {0xfc000300, 0x48000100, &EmulateInstructionLoongArch::EmulateBCNEZ,
       "bcnez cj, offs21"},
      {0xfc000000, 0x4c000000, &EmulateInstructionLoongArch::EmulateJIRL,
```
- **EN**: Introduces declarations for `lldb_private`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `lldb_private` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 48-67
```cpp
       "jirl rd, rj, offs16"},
      {0xfc000000, 0x50000000, &EmulateInstructionLoongArch::EmulateB,
       " b  offs26"},
      {0xfc000000, 0x54000000, &EmulateInstructionLoongArch::EmulateBL,
       "bl  offs26"},
      {0xfc000000, 0x58000000, &EmulateInstructionLoongArch::EmulateBEQ,
       "beq  rj, rd, offs16"},
      {0xfc000000, 0x5c000000, &EmulateInstructionLoongArch::EmulateBNE,
       "bne  rj, rd, offs16"},
      {0xfc000000, 0x60000000, &EmulateInstructionLoongArch::EmulateBLT,
       "blt  rj, rd, offs16"},
      {0xfc000000, 0x64000000, &EmulateInstructionLoongArch::EmulateBGE,
       "bge  rj, rd, offs16"},
      {0xfc000000, 0x68000000, &EmulateInstructionLoongArch::EmulateBLTU,
       "bltu rj, rd, offs16"},
      {0xfc000000, 0x6c000000, &EmulateInstructionLoongArch::EmulateBGEU,
       "bgeu rj, rd, offs16"},
      {0x00000000, 0x00000000, &EmulateInstructionLoongArch::EmulateNonJMP,
       "NonJMP"}};
  static const size_t num_loongarch_opcodes = std::size(g_opcodes);
```
- **EN**: Implements logic around `size`.
- **CN**: 围绕 `size` 实现具体逻辑。

### Lines 68-84
```cpp

  for (size_t i = 0; i < num_loongarch_opcodes; ++i)
    if ((g_opcodes[i].mask & inst) == g_opcodes[i].value)
      return &g_opcodes[i];
  return nullptr;
}

bool EmulateInstructionLoongArch::TestExecute(uint32_t inst) {
  Opcode *opcode_data = GetOpcodeForInstruction(inst);
  if (!opcode_data)
    return false;
  // Call the Emulate... function.
  if (!(this->*opcode_data->callback)(inst))
    return false;
  return true;
}

```
- **EN**: Implements logic around `TestExecute`, `GetOpcodeForInstruction`.
- **CN**: 围绕 `TestExecute`, `GetOpcodeForInstruction` 实现具体逻辑。

### Lines 85-101
```cpp
bool EmulateInstructionLoongArch::EvaluateInstruction(uint32_t options) {
  uint32_t inst_size = m_opcode.GetByteSize();
  uint32_t inst = m_opcode.GetOpcode32();
  bool increase_pc = options & eEmulateInstructionOptionAutoAdvancePC;

  Opcode *opcode_data = GetOpcodeForInstruction(inst);
  if (!opcode_data)
    return false;

  lldb::addr_t old_pc = 0;
  if (increase_pc) {
    auto addr = ReadPC();
    if (!addr)
      return false;
    old_pc = *addr;
  }

```
- **EN**: Implements logic around `EvaluateInstruction`, `GetByteSize`, `GetOpcode32`, `GetOpcodeForInstruction`, and 1 more symbols.
- **CN**: 围绕 `EvaluateInstruction`, `GetByteSize`, `GetOpcode32`, `GetOpcodeForInstruction`, and 1 more symbols 实现具体逻辑。

### Lines 102-111
```cpp
  // Call the Emulate... function.
  if (!(this->*opcode_data->callback)(inst))
    return false;

  if (increase_pc) {
    auto addr = ReadPC();
    if (!addr)
      return false;
    lldb::addr_t new_pc = *addr;

```
- **EN**: Implements logic around `ReadPC`.
- **CN**: 围绕 `ReadPC` 实现具体逻辑。

### Lines 112-125
```cpp
    if (new_pc == old_pc && !WritePC(old_pc + inst_size))
      return false;
  }
  return true;
}

bool EmulateInstructionLoongArch::ReadInstruction() {
  auto addr = ReadPC();
  if (!addr) {
    m_addr = LLDB_INVALID_ADDRESS;
    return false;
  }
  m_addr = *addr;

```
- **EN**: Implements logic around `WritePC`, `ReadInstruction`, `ReadPC`.
- **CN**: 围绕 `WritePC`, `ReadInstruction`, `ReadPC` 实现具体逻辑。

### Lines 126-135
```cpp
  bool success = false;
  Context ctx;
  ctx.type = eContextReadOpcode;
  ctx.SetNoArgs();
  uint32_t inst = (uint32_t)ReadMemoryUnsigned(ctx, m_addr, 4, 0, &success);
  m_opcode.SetOpcode32(inst, GetByteOrder());

  return true;
}

```
- **EN**: Implements logic around `SetNoArgs`, `ReadMemoryUnsigned`, `SetOpcode32`.
- **CN**: 围绕 `SetNoArgs`, `ReadMemoryUnsigned`, `SetOpcode32` 实现具体逻辑。

### Lines 136-155
```cpp
std::optional<RegisterInfo>
EmulateInstructionLoongArch::GetRegisterInfo(lldb::RegisterKind reg_kind,
                                             uint32_t reg_index) {
  if (reg_kind == eRegisterKindGeneric) {
    switch (reg_index) {
    case LLDB_REGNUM_GENERIC_PC:
      reg_kind = eRegisterKindLLDB;
      reg_index = gpr_pc_loongarch;
      break;
    case LLDB_REGNUM_GENERIC_SP:
      reg_kind = eRegisterKindLLDB;
      reg_index = gpr_sp_loongarch;
      break;
    case LLDB_REGNUM_GENERIC_FP:
      reg_kind = eRegisterKindLLDB;
      reg_index = gpr_fp_loongarch;
      break;
    case LLDB_REGNUM_GENERIC_RA:
      reg_kind = eRegisterKindLLDB;
      reg_index = gpr_ra_loongarch;
```
- **EN**: Implements logic around `GetRegisterInfo`.
- **CN**: 围绕 `GetRegisterInfo` 实现具体逻辑。

### Lines 156-168
```cpp
      break;
    // We may handle LLDB_REGNUM_GENERIC_ARGx when more instructions are
    // supported.
    default:
      llvm_unreachable("unsupported register");
    }
  }

  const RegisterInfo *array =
      RegisterInfoPOSIX_loongarch64::GetRegisterInfoPtr(m_arch);
  const uint32_t length =
      RegisterInfoPOSIX_loongarch64::GetRegisterInfoCount(m_arch);

```
- **EN**: Implements logic around `llvm_unreachable`, `GetRegisterInfoPtr`, `GetRegisterInfoCount`; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `llvm_unreachable`, `GetRegisterInfoPtr`, `GetRegisterInfoCount` 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 169-182
```cpp
  if (reg_index >= length || reg_kind != eRegisterKindLLDB)
    return {};
  return array[reg_index];
}

bool EmulateInstructionLoongArch::SetTargetTriple(const ArchSpec &arch) {
  return SupportsThisArch(arch);
}

bool EmulateInstructionLoongArch::TestEmulation(
    Stream &out_stream, ArchSpec &arch, OptionValueDictionary *test_data) {
  return false;
}

```
- **EN**: Implements logic around `SetTargetTriple`, `SupportsThisArch`, `TestEmulation`; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `SetTargetTriple`, `SupportsThisArch`, `TestEmulation` 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 183-200
```cpp
void EmulateInstructionLoongArch::Initialize() {
  PluginManager::RegisterPlugin(GetPluginNameStatic(),
                                GetPluginDescriptionStatic(), CreateInstance);
}

void EmulateInstructionLoongArch::Terminate() {
  PluginManager::UnregisterPlugin(CreateInstance);
}

lldb_private::EmulateInstruction *
EmulateInstructionLoongArch::CreateInstance(const ArchSpec &arch,
                                            InstructionType inst_type) {
  if (EmulateInstructionLoongArch::SupportsThisInstructionType(inst_type) &&
      SupportsThisArch(arch))
    return new EmulateInstructionLoongArch(arch);
  return nullptr;
}

```
- **EN**: Implements logic around `Initialize`, `RegisterPlugin`, `GetPluginDescriptionStatic`, `Terminate`, and 5 more symbols; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `Initialize`, `RegisterPlugin`, `GetPluginDescriptionStatic`, `Terminate`, and 5 more symbols 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 201-212
```cpp
bool EmulateInstructionLoongArch::SupportsThisArch(const ArchSpec &arch) {
  return arch.GetTriple().isLoongArch();
}

bool EmulateInstructionLoongArch::EmulateBEQZ(uint32_t inst) {
  return IsLoongArch64() ? EmulateBEQZ64(inst) : false;
}

bool EmulateInstructionLoongArch::EmulateBNEZ(uint32_t inst) {
  return IsLoongArch64() ? EmulateBNEZ64(inst) : false;
}

```
- **EN**: Implements logic around `SupportsThisArch`, `GetTriple`, `EmulateBEQZ`, `IsLoongArch64`, and 1 more symbols; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `SupportsThisArch`, `GetTriple`, `EmulateBEQZ`, `IsLoongArch64`, and 1 more symbols 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 213-224
```cpp
bool EmulateInstructionLoongArch::EmulateBCEQZ(uint32_t inst) {
  return IsLoongArch64() ? EmulateBCEQZ64(inst) : false;
}

bool EmulateInstructionLoongArch::EmulateBCNEZ(uint32_t inst) {
  return IsLoongArch64() ? EmulateBCNEZ64(inst) : false;
}

bool EmulateInstructionLoongArch::EmulateJIRL(uint32_t inst) {
  return IsLoongArch64() ? EmulateJIRL64(inst) : false;
}

```
- **EN**: Implements logic around `EmulateBCEQZ`, `IsLoongArch64`, `EmulateBCNEZ`, `EmulateJIRL`.
- **CN**: 围绕 `EmulateBCEQZ`, `IsLoongArch64`, `EmulateBCNEZ`, `EmulateJIRL` 实现具体逻辑。

### Lines 225-236
```cpp
bool EmulateInstructionLoongArch::EmulateB(uint32_t inst) {
  return IsLoongArch64() ? EmulateB64(inst) : false;
}

bool EmulateInstructionLoongArch::EmulateBL(uint32_t inst) {
  return IsLoongArch64() ? EmulateBL64(inst) : false;
}

bool EmulateInstructionLoongArch::EmulateBEQ(uint32_t inst) {
  return IsLoongArch64() ? EmulateBEQ64(inst) : false;
}

```
- **EN**: Implements logic around `EmulateB`, `IsLoongArch64`, `EmulateBL`, `EmulateBEQ`.
- **CN**: 围绕 `EmulateB`, `IsLoongArch64`, `EmulateBL`, `EmulateBEQ` 实现具体逻辑。

### Lines 237-248
```cpp
bool EmulateInstructionLoongArch::EmulateBNE(uint32_t inst) {
  return IsLoongArch64() ? EmulateBNE64(inst) : false;
}

bool EmulateInstructionLoongArch::EmulateBLT(uint32_t inst) {
  return IsLoongArch64() ? EmulateBLT64(inst) : false;
}

bool EmulateInstructionLoongArch::EmulateBGE(uint32_t inst) {
  return IsLoongArch64() ? EmulateBGE64(inst) : false;
}

```
- **EN**: Implements logic around `EmulateBNE`, `IsLoongArch64`, `EmulateBLT`, `EmulateBGE`.
- **CN**: 围绕 `EmulateBNE`, `IsLoongArch64`, `EmulateBLT`, `EmulateBGE` 实现具体逻辑。

### Lines 249-258
```cpp
bool EmulateInstructionLoongArch::EmulateBLTU(uint32_t inst) {
  return IsLoongArch64() ? EmulateBLTU64(inst) : false;
}

bool EmulateInstructionLoongArch::EmulateBGEU(uint32_t inst) {
  return IsLoongArch64() ? EmulateBGEU64(inst) : false;
}

bool EmulateInstructionLoongArch::EmulateNonJMP(uint32_t inst) { return false; }

```
- **EN**: Implements logic around `EmulateBLTU`, `IsLoongArch64`, `EmulateBGEU`, `EmulateNonJMP`.
- **CN**: 围绕 `EmulateBLTU`, `IsLoongArch64`, `EmulateBGEU`, `EmulateNonJMP` 实现具体逻辑。

### Lines 259-270
```cpp
// beqz rj, offs21
// if GR[rj] == 0:
//   PC = PC + SignExtend({offs21, 2'b0}, GRLEN)
bool EmulateInstructionLoongArch::EmulateBEQZ64(uint32_t inst) {
  bool success = false;
  uint32_t rj = Bits32(inst, 9, 5);

  auto addr = ReadPC();
  if (!addr)
    return false;
  uint64_t pc = *addr;

```
- **EN**: Implements logic around `EmulateBEQZ64`, `Bits32`, `ReadPC`.
- **CN**: 围绕 `EmulateBEQZ64`, `Bits32`, `ReadPC` 实现具体逻辑。

### Lines 271-281
```cpp
  uint32_t offs21 = Bits32(inst, 25, 10) + (Bits32(inst, 4, 0) << 16);
  uint64_t rj_val = ReadRegisterUnsigned(eRegisterKindLLDB, rj, 0, &success);
  if (!success)
    return false;
  if (rj_val == 0) {
    uint64_t next_pc = pc + llvm::SignExtend64<23>(offs21 << 2);
    return WritePC(next_pc);
  } else
    return WritePC(pc + 4);
}

```
- **EN**: Implements logic around `Bits32`, `ReadRegisterUnsigned`, `SignExtend64`, `WritePC`.
- **CN**: 围绕 `Bits32`, `ReadRegisterUnsigned`, `SignExtend64`, `WritePC` 实现具体逻辑。

### Lines 282-293
```cpp
// bnez rj, offs21
// if GR[rj] != 0:
//   PC = PC + SignExtend({offs21, 2'b0}, GRLEN)
bool EmulateInstructionLoongArch::EmulateBNEZ64(uint32_t inst) {
  bool success = false;
  uint32_t rj = Bits32(inst, 9, 5);

  auto addr = ReadPC();
  if (!addr)
    return false;
  uint64_t pc = *addr;

```
- **EN**: Implements logic around `EmulateBNEZ64`, `Bits32`, `ReadPC`.
- **CN**: 围绕 `EmulateBNEZ64`, `Bits32`, `ReadPC` 实现具体逻辑。

### Lines 294-304
```cpp
  uint32_t offs21 = Bits32(inst, 25, 10) + (Bits32(inst, 4, 0) << 16);
  uint64_t rj_val = ReadRegisterUnsigned(eRegisterKindLLDB, rj, 0, &success);
  if (!success)
    return false;
  if (rj_val != 0) {
    uint64_t next_pc = pc + llvm::SignExtend64<23>(offs21 << 2);
    return WritePC(next_pc);
  } else
    return WritePC(pc + 4);
}

```
- **EN**: Implements logic around `Bits32`, `ReadRegisterUnsigned`, `SignExtend64`, `WritePC`.
- **CN**: 围绕 `Bits32`, `ReadRegisterUnsigned`, `SignExtend64`, `WritePC` 实现具体逻辑。

### Lines 305-316
```cpp
// bceqz cj, offs21
// if CFR[cj] == 0:
//	PC = PC + SignExtend({offs21, 2'b0}, GRLEN)
bool EmulateInstructionLoongArch::EmulateBCEQZ64(uint32_t inst) {
  bool success = false;
  uint32_t cj = Bits32(inst, 7, 5) + fpr_fcc0_loongarch;

  auto addr = ReadPC();
  if (!addr)
    return false;
  uint64_t pc = *addr;

```
- **EN**: Implements logic around `EmulateBCEQZ64`, `Bits32`, `ReadPC`.
- **CN**: 围绕 `EmulateBCEQZ64`, `Bits32`, `ReadPC` 实现具体逻辑。

### Lines 317-329
```cpp
  uint32_t offs21 = Bits32(inst, 25, 10) + (Bits32(inst, 4, 0) << 16);
  uint8_t cj_val =
      (uint8_t)ReadRegisterUnsigned(eRegisterKindLLDB, cj, 0, &success);
  if (!success)
    return false;
  if (cj_val == 0) {
    uint64_t next_pc = pc + llvm::SignExtend64<23>(offs21 << 2);
    return WritePC(next_pc);
  } else
    return WritePC(pc + 4);
  return false;
}

```
- **EN**: Implements logic around `Bits32`, `ReadRegisterUnsigned`, `SignExtend64`, `WritePC`.
- **CN**: 围绕 `Bits32`, `ReadRegisterUnsigned`, `SignExtend64`, `WritePC` 实现具体逻辑。

### Lines 330-341
```cpp
// bcnez cj, offs21
// if CFR[cj] != 0:
//	PC = PC + SignExtend({offs21, 2'b0}, GRLEN)
bool EmulateInstructionLoongArch::EmulateBCNEZ64(uint32_t inst) {
  bool success = false;
  uint32_t cj = Bits32(inst, 7, 5) + fpr_fcc0_loongarch;

  auto addr = ReadPC();
  if (!addr)
    return false;
  uint64_t pc = *addr;

```
- **EN**: Implements logic around `EmulateBCNEZ64`, `Bits32`, `ReadPC`.
- **CN**: 围绕 `EmulateBCNEZ64`, `Bits32`, `ReadPC` 实现具体逻辑。

### Lines 342-354
```cpp
  uint32_t offs21 = Bits32(inst, 25, 10) + (Bits32(inst, 4, 0) << 16);
  uint8_t cj_val =
      (uint8_t)ReadRegisterUnsigned(eRegisterKindLLDB, cj, 0, &success);
  if (!success)
    return false;
  if (cj_val != 0) {
    uint64_t next_pc = pc + llvm::SignExtend64<23>(offs21 << 2);
    return WritePC(next_pc);
  } else
    return WritePC(pc + 4);
  return false;
}

```
- **EN**: Implements logic around `Bits32`, `ReadRegisterUnsigned`, `SignExtend64`, `WritePC`.
- **CN**: 围绕 `Bits32`, `ReadRegisterUnsigned`, `SignExtend64`, `WritePC` 实现具体逻辑。

### Lines 355-367
```cpp
// jirl rd, rj, offs16
// GR[rd] = PC + 4
// PC = GR[rj] + SignExtend({offs16, 2'b0}, GRLEN)
bool EmulateInstructionLoongArch::EmulateJIRL64(uint32_t inst) {
  uint32_t rj = Bits32(inst, 9, 5);
  uint32_t rd = Bits32(inst, 4, 0);
  bool success = false;

  auto addr = ReadPC();
  if (!addr)
    return false;
  uint64_t pc = *addr;

```
- **EN**: Implements logic around `EmulateJIRL64`, `Bits32`, `ReadPC`.
- **CN**: 围绕 `EmulateJIRL64`, `Bits32`, `ReadPC` 实现具体逻辑。

### Lines 368-377
```cpp
  EmulateInstruction::Context ctx;
  if (!WriteRegisterUnsigned(ctx, eRegisterKindLLDB, rd, pc + 4))
    return false;
  uint64_t rj_val = ReadRegisterUnsigned(eRegisterKindLLDB, rj, 0, &success);
  if (!success)
    return false;
  uint64_t next_pc = rj_val + llvm::SignExtend64<18>(Bits32(inst, 25, 10) << 2);
  return WritePC(next_pc);
}

```
- **EN**: Implements logic around `WriteRegisterUnsigned`, `ReadRegisterUnsigned`, `SignExtend64`, `WritePC`.
- **CN**: 围绕 `WriteRegisterUnsigned`, `ReadRegisterUnsigned`, `SignExtend64`, `WritePC` 实现具体逻辑。

### Lines 378-390
```cpp
// b offs26
// PC = PC + SignExtend({offs26, 2' b0}, GRLEN)
bool EmulateInstructionLoongArch::EmulateB64(uint32_t inst) {
  auto addr = ReadPC();
  if (!addr)
    return false;
  uint64_t pc = *addr;

  uint32_t offs26 = Bits32(inst, 25, 10) + (Bits32(inst, 9, 0) << 16);
  uint64_t next_pc = pc + llvm::SignExtend64<28>(offs26 << 2);
  return WritePC(next_pc);
}

```
- **EN**: Implements logic around `EmulateB64`, `ReadPC`, `Bits32`, `SignExtend64`, and 1 more symbols.
- **CN**: 围绕 `EmulateB64`, `ReadPC`, `Bits32`, `SignExtend64`, and 1 more symbols 实现具体逻辑。

### Lines 391-407
```cpp
// bl offs26
// GR[1] = PC + 4
// PC = PC + SignExtend({offs26, 2'b0}, GRLEN)
bool EmulateInstructionLoongArch::EmulateBL64(uint32_t inst) {
  auto addr = ReadPC();
  if (!addr)
    return false;
  uint64_t pc = *addr;

  EmulateInstruction::Context ctx;
  if (!WriteRegisterUnsigned(ctx, eRegisterKindLLDB, gpr_r1_loongarch, pc + 4))
    return false;
  uint32_t offs26 = Bits32(inst, 25, 10) + (Bits32(inst, 9, 0) << 16);
  uint64_t next_pc = pc + llvm::SignExtend64<28>(offs26 << 2);
  return WritePC(next_pc);
}

```
- **EN**: Implements logic around `EmulateBL64`, `ReadPC`, `WriteRegisterUnsigned`, `Bits32`, and 2 more symbols.
- **CN**: 围绕 `EmulateBL64`, `ReadPC`, `WriteRegisterUnsigned`, `Bits32`, and 2 more symbols 实现具体逻辑。

### Lines 408-420
```cpp
// beq rj, rd, offs16
// if GR[rj] == GR[rd]:
//   PC = PC + SignExtend({offs16, 2'b0}, GRLEN)
bool EmulateInstructionLoongArch::EmulateBEQ64(uint32_t inst) {
  bool success = false;
  uint32_t rj = Bits32(inst, 9, 5);
  uint32_t rd = Bits32(inst, 4, 0);

  auto addr = ReadPC();
  if (!addr)
    return false;
  uint64_t pc = *addr;

```
- **EN**: Implements logic around `EmulateBEQ64`, `Bits32`, `ReadPC`.
- **CN**: 围绕 `EmulateBEQ64`, `Bits32`, `ReadPC` 实现具体逻辑。

### Lines 421-433
```cpp
  uint64_t rj_val = ReadRegisterUnsigned(eRegisterKindLLDB, rj, 0, &success);
  if (!success)
    return false;
  uint64_t rd_val = ReadRegisterUnsigned(eRegisterKindLLDB, rd, 0, &success);
  if (!success)
    return false;
  if (rj_val == rd_val) {
    uint64_t next_pc = pc + llvm::SignExtend64<18>(Bits32(inst, 25, 10) << 2);
    return WritePC(next_pc);
  } else
    return WritePC(pc + 4);
}

```
- **EN**: Implements logic around `ReadRegisterUnsigned`, `SignExtend64`, `WritePC`.
- **CN**: 围绕 `ReadRegisterUnsigned`, `SignExtend64`, `WritePC` 实现具体逻辑。

### Lines 434-446
```cpp
// bne rj, rd, offs16
// if GR[rj] != GR[rd]:
//   PC = PC + SignExtend({offs16, 2'b0}, GRLEN)
bool EmulateInstructionLoongArch::EmulateBNE64(uint32_t inst) {
  bool success = false;
  uint32_t rj = Bits32(inst, 9, 5);
  uint32_t rd = Bits32(inst, 4, 0);

  auto addr = ReadPC();
  if (!addr)
    return false;
  uint64_t pc = *addr;

```
- **EN**: Implements logic around `EmulateBNE64`, `Bits32`, `ReadPC`.
- **CN**: 围绕 `EmulateBNE64`, `Bits32`, `ReadPC` 实现具体逻辑。

### Lines 447-459
```cpp
  uint64_t rj_val = ReadRegisterUnsigned(eRegisterKindLLDB, rj, 0, &success);
  if (!success)
    return false;
  uint64_t rd_val = ReadRegisterUnsigned(eRegisterKindLLDB, rd, 0, &success);
  if (!success)
    return false;
  if (rj_val != rd_val) {
    uint64_t next_pc = pc + llvm::SignExtend64<18>(Bits32(inst, 25, 10) << 2);
    return WritePC(next_pc);
  } else
    return WritePC(pc + 4);
}

```
- **EN**: Implements logic around `ReadRegisterUnsigned`, `SignExtend64`, `WritePC`.
- **CN**: 围绕 `ReadRegisterUnsigned`, `SignExtend64`, `WritePC` 实现具体逻辑。

### Lines 460-472
```cpp
// blt rj, rd, offs16
// if signed(GR[rj]) < signed(GR[rd]):
//   PC = PC + SignExtend({offs16, 2'b0}, GRLEN)
bool EmulateInstructionLoongArch::EmulateBLT64(uint32_t inst) {
  bool success = false;
  uint32_t rj = Bits32(inst, 9, 5);
  uint32_t rd = Bits32(inst, 4, 0);

  auto addr = ReadPC();
  if (!addr)
    return false;
  uint64_t pc = *addr;

```
- **EN**: Implements logic around `EmulateBLT64`, `Bits32`, `ReadPC`.
- **CN**: 围绕 `EmulateBLT64`, `Bits32`, `ReadPC` 实现具体逻辑。

### Lines 473-487
```cpp
  int64_t rj_val =
      (int64_t)ReadRegisterUnsigned(eRegisterKindLLDB, rj, 0, &success);
  if (!success)
    return false;
  int64_t rd_val =
      (int64_t)ReadRegisterUnsigned(eRegisterKindLLDB, rd, 0, &success);
  if (!success)
    return false;
  if (rj_val < rd_val) {
    uint64_t next_pc = pc + llvm::SignExtend64<18>(Bits32(inst, 25, 10) << 2);
    return WritePC(next_pc);
  } else
    return WritePC(pc + 4);
}

```
- **EN**: Implements logic around `ReadRegisterUnsigned`, `SignExtend64`, `WritePC`.
- **CN**: 围绕 `ReadRegisterUnsigned`, `SignExtend64`, `WritePC` 实现具体逻辑。

### Lines 488-500
```cpp
// bge rj, rd, offs16
// if signed(GR[rj]) >= signed(GR[rd]):
//   PC = PC + SignExtend({offs16, 2'b0}, GRLEN)
bool EmulateInstructionLoongArch::EmulateBGE64(uint32_t inst) {
  bool success = false;
  uint32_t rj = Bits32(inst, 9, 5);
  uint32_t rd = Bits32(inst, 4, 0);

  auto addr = ReadPC();
  if (!addr)
    return false;
  uint64_t pc = *addr;

```
- **EN**: Implements logic around `EmulateBGE64`, `Bits32`, `ReadPC`.
- **CN**: 围绕 `EmulateBGE64`, `Bits32`, `ReadPC` 实现具体逻辑。

### Lines 501-515
```cpp
  int64_t rj_val =
      (int64_t)ReadRegisterUnsigned(eRegisterKindLLDB, rj, 0, &success);
  if (!success)
    return false;
  int64_t rd_val =
      (int64_t)ReadRegisterUnsigned(eRegisterKindLLDB, rd, 0, &success);
  if (!success)
    return false;
  if (rj_val >= rd_val) {
    uint64_t next_pc = pc + llvm::SignExtend64<18>(Bits32(inst, 25, 10) << 2);
    return WritePC(next_pc);
  } else
    return WritePC(pc + 4);
}

```
- **EN**: Implements logic around `ReadRegisterUnsigned`, `SignExtend64`, `WritePC`.
- **CN**: 围绕 `ReadRegisterUnsigned`, `SignExtend64`, `WritePC` 实现具体逻辑。

### Lines 516-528
```cpp
// bltu rj, rd, offs16
// if unsigned(GR[rj]) < unsigned(GR[rd]):
//   PC = PC + SignExtend({offs16, 2'b0}, GRLEN)
bool EmulateInstructionLoongArch::EmulateBLTU64(uint32_t inst) {
  bool success = false;
  uint32_t rj = Bits32(inst, 9, 5);
  uint32_t rd = Bits32(inst, 4, 0);

  auto addr = ReadPC();
  if (!addr)
    return false;
  uint64_t pc = *addr;

```
- **EN**: Implements logic around `EmulateBLTU64`, `Bits32`, `ReadPC`.
- **CN**: 围绕 `EmulateBLTU64`, `Bits32`, `ReadPC` 实现具体逻辑。

### Lines 529-541
```cpp
  uint64_t rj_val = ReadRegisterUnsigned(eRegisterKindLLDB, rj, 0, &success);
  if (!success)
    return false;
  uint64_t rd_val = ReadRegisterUnsigned(eRegisterKindLLDB, rd, 0, &success);
  if (!success)
    return false;
  if (rj_val < rd_val) {
    uint64_t next_pc = pc + llvm::SignExtend64<18>(Bits32(inst, 25, 10) << 2);
    return WritePC(next_pc);
  } else
    return WritePC(pc + 4);
}

```
- **EN**: Implements logic around `ReadRegisterUnsigned`, `SignExtend64`, `WritePC`.
- **CN**: 围绕 `ReadRegisterUnsigned`, `SignExtend64`, `WritePC` 实现具体逻辑。

### Lines 542-554
```cpp
// bgeu rj, rd, offs16
// if unsigned(GR[rj]) >= unsigned(GR[rd]):
//   PC = PC + SignExtend({offs16, 2'b0}, GRLEN)
bool EmulateInstructionLoongArch::EmulateBGEU64(uint32_t inst) {
  bool success = false;
  uint32_t rj = Bits32(inst, 9, 5);
  uint32_t rd = Bits32(inst, 4, 0);

  auto addr = ReadPC();
  if (!addr)
    return false;
  uint64_t pc = *addr;

```
- **EN**: Implements logic around `EmulateBGEU64`, `Bits32`, `ReadPC`.
- **CN**: 围绕 `EmulateBGEU64`, `Bits32`, `ReadPC` 实现具体逻辑。

### Lines 555-567
```cpp
  uint64_t rj_val = ReadRegisterUnsigned(eRegisterKindLLDB, rj, 0, &success);
  if (!success)
    return false;
  uint64_t rd_val = ReadRegisterUnsigned(eRegisterKindLLDB, rd, 0, &success);
  if (!success)
    return false;
  if (rj_val >= rd_val) {
    uint64_t next_pc = pc + llvm::SignExtend64<18>(Bits32(inst, 25, 10) << 2);
    return WritePC(next_pc);
  } else
    return WritePC(pc + 4);
}

```
- **EN**: Implements logic around `ReadRegisterUnsigned`, `SignExtend64`, `WritePC`.
- **CN**: 围绕 `ReadRegisterUnsigned`, `SignExtend64`, `WritePC` 实现具体逻辑。

### Lines 568-568
```cpp
} // namespace lldb_private
```
- **EN**: Contains supporting implementation details for the surrounding LLDB subsystem.
- **CN**: 包含周边 LLDB 子系统所需的辅助实现细节。

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

- **Direct LLDB/LLVM includes / 直接的 LLDB/LLVM 包含**: `EmulateInstructionLoongArch.h`, `Plugins/Process/Utility/InstructionUtils.h`, `Plugins/Process/Utility/RegisterInfoPOSIX_loongarch64.h`, `Plugins/Process/Utility/lldb-loongarch-register-enums.h`, `lldb/Core/Address.h`, `lldb/Core/PluginManager.h`, `lldb/Interpreter/OptionValueArray.h`, `lldb/Interpreter/OptionValueDictionary.h`, `lldb/Symbol/UnwindPlan.h`, `lldb/Utility/ArchSpec.h` ... (+5 more)
- **Standard-library headers / 标准库头文件**: `<cstdlib>`, `<optional>`
- **Subsystem categories / 子系统类别**: shared LLDB utility classes / 共享 LLDB 工具类 (4), LLDB core debugger abstractions / LLDB 核心调试抽象 (2), command interpreter support / 命令解释器支持 (2), symbol and debug-info abstractions / 符号与调试信息抽象 (1), LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (1), LLVM support-library helpers / LLVM Support 库辅助组件 (1)
