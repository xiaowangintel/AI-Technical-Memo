# EmulateInstructionARM64.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/source/Plugins/Instruction/ARM64/EmulateInstructionARM64.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements the LLDB support logic associated with `EmulateInstructionARM64`.
  - **CN**: 实现与 `EmulateInstructionARM64` 相关的 LLDB 支持逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-19
```cpp
//===-- EmulateInstructionARM64.cpp ---------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "EmulateInstructionARM64.h"

#include "lldb/Core/Address.h"
#include "lldb/Core/PluginManager.h"
#include "lldb/Symbol/UnwindPlan.h"
#include "lldb/Utility/ArchSpec.h"
#include "lldb/Utility/RegisterValue.h"
#include "lldb/Utility/Stream.h"

#include "llvm/Support/CheckedArithmetic.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `EmulateInstructionARM64.h`, `lldb/Core/Address.h`, `lldb/Core/PluginManager.h`, `lldb/Symbol/UnwindPlan.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `EmulateInstructionARM64.h`, `lldb/Core/Address.h`, `lldb/Core/PluginManager.h`, `lldb/Symbol/UnwindPlan.h`。

### Lines 20-40
```cpp
#include "Plugins/Process/Utility/ARMDefines.h"
#include "Plugins/Process/Utility/ARMUtils.h"
#include "Plugins/Process/Utility/lldb-arm64-register-enums.h"

#include <algorithm>
#include <cstdlib>
#include <optional>

#define GPR_OFFSET(idx) ((idx)*8)
#define GPR_OFFSET_NAME(reg) 0
#define FPU_OFFSET(idx) ((idx)*16)
#define FPU_OFFSET_NAME(reg) 0
#define EXC_OFFSET_NAME(reg) 0
#define DBG_OFFSET_NAME(reg) 0
#define DBG_OFFSET_NAME(reg) 0
#define DEFINE_DBG(re, y)                                                      \
  "na", nullptr, 8, 0, lldb::eEncodingUint, lldb::eFormatHex,                  \
      {LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM,          \
       LLDB_INVALID_REGNUM, LLDB_INVALID_REGNUM},                              \
      nullptr, nullptr, nullptr

```
- **EN**: Pulls in the headers needed by this translation unit, including `Plugins/Process/Utility/ARMDefines.h`, `Plugins/Process/Utility/ARMUtils.h`, `Plugins/Process/Utility/lldb-arm64-register-enums.h`, `algorithm`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `Plugins/Process/Utility/ARMDefines.h`, `Plugins/Process/Utility/ARMUtils.h`, `Plugins/Process/Utility/lldb-arm64-register-enums.h`, `algorithm`。

### Lines 41-60
```cpp
#define DECLARE_REGISTER_INFOS_ARM64_STRUCT

#include "Plugins/Process/Utility/RegisterInfos_arm64.h"

#include "llvm/ADT/STLExtras.h"
#include "llvm/Support/MathExtras.h"

#include "Plugins/Process/Utility/InstructionUtils.h"

using namespace lldb;
using namespace lldb_private;

LLDB_PLUGIN_DEFINE_ADV(EmulateInstructionARM64, InstructionARM64)

static std::optional<RegisterInfo> LLDBTableGetRegisterInfo(uint32_t reg_num) {
  if (reg_num >= std::size(g_register_infos_arm64_le))
    return {};
  return g_register_infos_arm64_le[reg_num];
}

```
- **EN**: Pulls in the headers needed by this translation unit, including `Plugins/Process/Utility/RegisterInfos_arm64.h`, `llvm/ADT/STLExtras.h`, `llvm/Support/MathExtras.h`, `Plugins/Process/Utility/InstructionUtils.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `Plugins/Process/Utility/RegisterInfos_arm64.h`, `llvm/ADT/STLExtras.h`, `llvm/Support/MathExtras.h`, `Plugins/Process/Utility/InstructionUtils.h`。

### Lines 61-78
```cpp
#define No_VFP 0
#define VFPv1 (1u << 1)
#define VFPv2 (1u << 2)
#define VFPv3 (1u << 3)
#define AdvancedSIMD (1u << 4)

#define VFPv1_ABOVE (VFPv1 | VFPv2 | VFPv3 | AdvancedSIMD)
#define VFPv2_ABOVE (VFPv2 | VFPv3 | AdvancedSIMD)
#define VFPv2v3 (VFPv2 | VFPv3)

#define UInt(x) ((uint64_t)x)
#define SInt(x) ((int64_t)x)
#define bit bool
#define boolean bool
#define integer int64_t

static inline bool IsZero(uint64_t x) { return x == 0; }

```
- **EN**: Defines preprocessor-controlled structure, feature gates, or include-guard state.
- **CN**: 定义受预处理器控制的结构、特性开关或头文件保护状态。

### Lines 79-106
```cpp
static inline uint64_t NOT(uint64_t x) { return ~x; }

// LSL()
// =====

static inline uint64_t LSL(uint64_t x, integer shift) {
  if (shift == 0)
    return x;
  return x << shift;
}

// ConstrainUnpredictable()
// ========================

EmulateInstructionARM64::ConstraintType
ConstrainUnpredictable(EmulateInstructionARM64::Unpredictable which) {
  EmulateInstructionARM64::ConstraintType result =
      EmulateInstructionARM64::Constraint_UNKNOWN;
  switch (which) {
  case EmulateInstructionARM64::Unpredictable_WBOVERLAP:
  case EmulateInstructionARM64::Unpredictable_LDPOVERLAP:
    // TODO: don't know what to really do here? Pseudo code says:
    // set result to one of above Constraint behaviours or UNDEFINED
    break;
  }
  return result;
}

```
- **EN**: Implements logic around `NOT`, `LSL`, `ConstrainUnpredictable`.
- **CN**: 围绕 `NOT`, `LSL`, `ConstrainUnpredictable` 实现具体逻辑。

### Lines 107-134
```cpp
//
// EmulateInstructionARM implementation
//

void EmulateInstructionARM64::Initialize() {
  PluginManager::RegisterPlugin(GetPluginNameStatic(),
                                GetPluginDescriptionStatic(), CreateInstance);
}

void EmulateInstructionARM64::Terminate() {
  PluginManager::UnregisterPlugin(CreateInstance);
}

llvm::StringRef EmulateInstructionARM64::GetPluginDescriptionStatic() {
  return "Emulate instructions for the ARM64 architecture.";
}

EmulateInstruction *
EmulateInstructionARM64::CreateInstance(const ArchSpec &arch,
                                        InstructionType inst_type) {
  if (EmulateInstructionARM64::SupportsEmulatingInstructionsOfTypeStatic(
          inst_type)) {
    if (arch.GetTriple().getArch() == llvm::Triple::aarch64 ||
        arch.GetTriple().getArch() == llvm::Triple::aarch64_32) {
      return new EmulateInstructionARM64(arch);
    }
  }

```
- **EN**: Implements logic around `Initialize`, `RegisterPlugin`, `GetPluginDescriptionStatic`, `Terminate`, and 5 more symbols; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `Initialize`, `RegisterPlugin`, `GetPluginDescriptionStatic`, `Terminate`, and 5 more symbols 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 135-170
```cpp
  return nullptr;
}

bool EmulateInstructionARM64::SetTargetTriple(const ArchSpec &arch) {
  if (arch.GetTriple().getArch() == llvm::Triple::arm)
    return true;
  else if (arch.GetTriple().getArch() == llvm::Triple::thumb)
    return true;

  return false;
}

std::optional<RegisterInfo>
EmulateInstructionARM64::GetRegisterInfo(RegisterKind reg_kind,
                                         uint32_t reg_num) {
  if (reg_kind == eRegisterKindGeneric) {
    switch (reg_num) {
    case LLDB_REGNUM_GENERIC_PC:
      reg_kind = eRegisterKindLLDB;
      reg_num = gpr_pc_arm64;
      break;
    case LLDB_REGNUM_GENERIC_SP:
      reg_kind = eRegisterKindLLDB;
      reg_num = gpr_sp_arm64;
      break;
    case LLDB_REGNUM_GENERIC_FP:
      reg_kind = eRegisterKindLLDB;
      reg_num = gpr_fp_arm64;
      break;
    case LLDB_REGNUM_GENERIC_RA:
      reg_kind = eRegisterKindLLDB;
      reg_num = gpr_lr_arm64;
      break;
    case LLDB_REGNUM_GENERIC_FLAGS:
      reg_kind = eRegisterKindLLDB;
      reg_num = gpr_cpsr_arm64;
```
- **EN**: Implements logic around `SetTargetTriple`, `GetTriple`, `GetRegisterInfo`; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `SetTargetTriple`, `GetTriple`, `GetRegisterInfo` 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 171-201
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

EmulateInstructionARM64::Opcode *
EmulateInstructionARM64::GetOpcodeForInstruction(const uint32_t opcode) {
  static EmulateInstructionARM64::Opcode g_opcodes[] = {
      // Prologue instructions

      // push register(s)
      {0xff000000, 0xd1000000, No_VFP,
       &EmulateInstructionARM64::EmulateADDSUBImm,
       "SUB  <Xd|SP>, <Xn|SP>, #<imm> {, <shift>}"},
      {0xff000000, 0xf1000000, No_VFP,
       &EmulateInstructionARM64::EmulateADDSUBImm,
       "SUBS  <Xd>, <Xn|SP>, #<imm> {, <shift>}"},
      {0xff000000, 0x91000000, No_VFP,
       &EmulateInstructionARM64::EmulateADDSUBImm,
       "ADD  <Xd|SP>, <Xn|SP>, #<imm> {, <shift>}"},
      {0xff000000, 0xb1000000, No_VFP,
       &EmulateInstructionARM64::EmulateADDSUBImm,
       "ADDS  <Xd>, <Xn|SP>, #<imm> {, <shift>}"},

```
- **EN**: Implements logic around `LLDBTableGetRegisterInfo`, `GetOpcodeForInstruction`; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `LLDBTableGetRegisterInfo`, `GetOpcodeForInstruction` 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 202-230
```cpp
      {0xff000000, 0x51000000, No_VFP,
       &EmulateInstructionARM64::EmulateADDSUBImm,
       "SUB  <Wd|WSP>, <Wn|WSP>, #<imm> {, <shift>}"},
      {0xff000000, 0x71000000, No_VFP,
       &EmulateInstructionARM64::EmulateADDSUBImm,
       "SUBS  <Wd>, <Wn|WSP>, #<imm> {, <shift>}"},
      {0xff000000, 0x11000000, No_VFP,
       &EmulateInstructionARM64::EmulateADDSUBImm,
       "ADD  <Wd|WSP>, <Wn|WSP>, #<imm> {, <shift>}"},
      {0xff000000, 0x31000000, No_VFP,
       &EmulateInstructionARM64::EmulateADDSUBImm,
       "ADDS  <Wd>, <Wn|WSP>, #<imm> {, <shift>}"},

      {0xffc00000, 0x29000000, No_VFP,
       &EmulateInstructionARM64::EmulateLDPSTP<AddrMode_OFF>,
       "STP  <Wt>, <Wt2>, [<Xn|SP>{, #<imm>}]"},
      {0xffc00000, 0xa9000000, No_VFP,
       &EmulateInstructionARM64::EmulateLDPSTP<AddrMode_OFF>,
       "STP  <Xt>, <Xt2>, [<Xn|SP>{, #<imm>}]"},
      {0xffc00000, 0x2d000000, No_VFP,
       &EmulateInstructionARM64::EmulateLDPSTP<AddrMode_OFF>,
       "STP  <St>, <St2>, [<Xn|SP>{, #<imm>}]"},
      {0xffc00000, 0x6d000000, No_VFP,
       &EmulateInstructionARM64::EmulateLDPSTP<AddrMode_OFF>,
       "STP  <Dt>, <Dt2>, [<Xn|SP>{, #<imm>}]"},
      {0xffc00000, 0xad000000, No_VFP,
       &EmulateInstructionARM64::EmulateLDPSTP<AddrMode_OFF>,
       "STP  <Qt>, <Qt2>, [<Xn|SP>{, #<imm>}]"},

```
- **EN**: Contains supporting implementation details for the surrounding LLDB subsystem.
- **CN**: 包含周边 LLDB 子系统所需的辅助实现细节。

### Lines 231-262
```cpp
      {0xffc00000, 0x29800000, No_VFP,
       &EmulateInstructionARM64::EmulateLDPSTP<AddrMode_PRE>,
       "STP  <Wt>, <Wt2>, [<Xn|SP>, #<imm>]!"},
      {0xffc00000, 0xa9800000, No_VFP,
       &EmulateInstructionARM64::EmulateLDPSTP<AddrMode_PRE>,
       "STP  <Xt>, <Xt2>, [<Xn|SP>, #<imm>]!"},
      {0xffc00000, 0x2d800000, No_VFP,
       &EmulateInstructionARM64::EmulateLDPSTP<AddrMode_PRE>,
       "STP  <St>, <St2>, [<Xn|SP>, #<imm>]!"},
      {0xffc00000, 0x6d800000, No_VFP,
       &EmulateInstructionARM64::EmulateLDPSTP<AddrMode_PRE>,
       "STP  <Dt>, <Dt2>, [<Xn|SP>, #<imm>]!"},
      {0xffc00000, 0xad800000, No_VFP,
       &EmulateInstructionARM64::EmulateLDPSTP<AddrMode_PRE>,
       "STP  <Qt>, <Qt2>, [<Xn|SP>, #<imm>]!"},

      {0xffc00000, 0x28800000, No_VFP,
       &EmulateInstructionARM64::EmulateLDPSTP<AddrMode_POST>,
       "STP  <Wt>, <Wt2>, [<Xn|SP>, #<imm>]!"},
      {0xffc00000, 0xa8800000, No_VFP,
       &EmulateInstructionARM64::EmulateLDPSTP<AddrMode_POST>,
       "STP  <Xt>, <Xt2>, [<Xn|SP>, #<imm>]!"},
      {0xffc00000, 0x2c800000, No_VFP,
       &EmulateInstructionARM64::EmulateLDPSTP<AddrMode_POST>,
       "STP  <St>, <St2>, [<Xn|SP>, #<imm>]!"},
      {0xffc00000, 0x6c800000, No_VFP,
       &EmulateInstructionARM64::EmulateLDPSTP<AddrMode_POST>,
       "STP  <Dt>, <Dt2>, [<Xn|SP>, #<imm>]!"},
      {0xffc00000, 0xac800000, No_VFP,
       &EmulateInstructionARM64::EmulateLDPSTP<AddrMode_POST>,
       "STP  <Qt>, <Qt2>, [<Xn|SP>, #<imm>]!"},

```
- **EN**: Contains supporting implementation details for the surrounding LLDB subsystem.
- **CN**: 包含周边 LLDB 子系统所需的辅助实现细节。

### Lines 263-294
```cpp
      {0xffc00000, 0x29400000, No_VFP,
       &EmulateInstructionARM64::EmulateLDPSTP<AddrMode_OFF>,
       "LDP  <Wt>, <Wt2>, [<Xn|SP>{, #<imm>}]"},
      {0xffc00000, 0xa9400000, No_VFP,
       &EmulateInstructionARM64::EmulateLDPSTP<AddrMode_OFF>,
       "LDP  <Xt>, <Xt2>, [<Xn|SP>{, #<imm>}]"},
      {0xffc00000, 0x2d400000, No_VFP,
       &EmulateInstructionARM64::EmulateLDPSTP<AddrMode_OFF>,
       "LDP  <St>, <St2>, [<Xn|SP>{, #<imm>}]"},
      {0xffc00000, 0x6d400000, No_VFP,
       &EmulateInstructionARM64::EmulateLDPSTP<AddrMode_OFF>,
       "LDP  <Dt>, <Dt2>, [<Xn|SP>{, #<imm>}]"},
      {0xffc00000, 0xad400000, No_VFP,
       &EmulateInstructionARM64::EmulateLDPSTP<AddrMode_OFF>,
       "LDP  <Qt>, <Qt2>, [<Xn|SP>{, #<imm>}]"},

      {0xffc00000, 0x29c00000, No_VFP,
       &EmulateInstructionARM64::EmulateLDPSTP<AddrMode_PRE>,
       "LDP  <Wt>, <Wt2>, [<Xn|SP>, #<imm>]!"},
      {0xffc00000, 0xa9c00000, No_VFP,
       &EmulateInstructionARM64::EmulateLDPSTP<AddrMode_PRE>,
       "LDP  <Xt>, <Xt2>, [<Xn|SP>, #<imm>]!"},
      {0xffc00000, 0x2dc00000, No_VFP,
       &EmulateInstructionARM64::EmulateLDPSTP<AddrMode_PRE>,
       "LDP  <St>, <St2>, [<Xn|SP>, #<imm>]!"},
      {0xffc00000, 0x6dc00000, No_VFP,
       &EmulateInstructionARM64::EmulateLDPSTP<AddrMode_PRE>,
       "LDP  <Dt>, <Dt2>, [<Xn|SP>, #<imm>]!"},
      {0xffc00000, 0xadc00000, No_VFP,
       &EmulateInstructionARM64::EmulateLDPSTP<AddrMode_PRE>,
       "LDP  <Qt>, <Qt2>, [<Xn|SP>, #<imm>]!"},

```
- **EN**: Contains supporting implementation details for the surrounding LLDB subsystem.
- **CN**: 包含周边 LLDB 子系统所需的辅助实现细节。

### Lines 295-329
```cpp
      {0xffc00000, 0x28c00000, No_VFP,
       &EmulateInstructionARM64::EmulateLDPSTP<AddrMode_POST>,
       "LDP  <Wt>, <Wt2>, [<Xn|SP>, #<imm>]!"},
      {0xffc00000, 0xa8c00000, No_VFP,
       &EmulateInstructionARM64::EmulateLDPSTP<AddrMode_POST>,
       "LDP  <Xt>, <Xt2>, [<Xn|SP>, #<imm>]!"},
      {0xffc00000, 0x2cc00000, No_VFP,
       &EmulateInstructionARM64::EmulateLDPSTP<AddrMode_POST>,
       "LDP  <St>, <St2>, [<Xn|SP>, #<imm>]!"},
      {0xffc00000, 0x6cc00000, No_VFP,
       &EmulateInstructionARM64::EmulateLDPSTP<AddrMode_POST>,
       "LDP  <Dt>, <Dt2>, [<Xn|SP>, #<imm>]!"},
      {0xffc00000, 0xacc00000, No_VFP,
       &EmulateInstructionARM64::EmulateLDPSTP<AddrMode_POST>,
       "LDP  <Qt>, <Qt2>, [<Xn|SP>, #<imm>]!"},

      {0xffe00c00, 0xb8000400, No_VFP,
       &EmulateInstructionARM64::EmulateLDRSTRImm<AddrMode_POST>,
       "STR <Wt>, [<Xn|SP>], #<simm>"},
      {0xffe00c00, 0xf8000400, No_VFP,
       &EmulateInstructionARM64::EmulateLDRSTRImm<AddrMode_POST>,
       "STR <Xt>, [<Xn|SP>], #<simm>"},
      {0xffe00c00, 0xb8000c00, No_VFP,
       &EmulateInstructionARM64::EmulateLDRSTRImm<AddrMode_PRE>,
       "STR <Wt>, [<Xn|SP>, #<simm>]!"},
      {0xffe00c00, 0xf8000c00, No_VFP,
       &EmulateInstructionARM64::EmulateLDRSTRImm<AddrMode_PRE>,
       "STR <Xt>, [<Xn|SP>, #<simm>]!"},
      {0xffc00000, 0xb9000000, No_VFP,
       &EmulateInstructionARM64::EmulateLDRSTRImm<AddrMode_OFF>,
       "STR <Wt>, [<Xn|SP>{, #<pimm>}]"},
      {0xffc00000, 0xf9000000, No_VFP,
       &EmulateInstructionARM64::EmulateLDRSTRImm<AddrMode_OFF>,
       "STR <Xt>, [<Xn|SP>{, #<pimm>}]"},

```
- **EN**: Contains supporting implementation details for the surrounding LLDB subsystem.
- **CN**: 包含周边 LLDB 子系统所需的辅助实现细节。

### Lines 330-348
```cpp
      {0xffe00c00, 0xb8400400, No_VFP,
       &EmulateInstructionARM64::EmulateLDRSTRImm<AddrMode_POST>,
       "LDR <Wt>, [<Xn|SP>], #<simm>"},
      {0xffe00c00, 0xf8400400, No_VFP,
       &EmulateInstructionARM64::EmulateLDRSTRImm<AddrMode_POST>,
       "LDR <Xt>, [<Xn|SP>], #<simm>"},
      {0xffe00c00, 0xb8400c00, No_VFP,
       &EmulateInstructionARM64::EmulateLDRSTRImm<AddrMode_PRE>,
       "LDR <Wt>, [<Xn|SP>, #<simm>]!"},
      {0xffe00c00, 0xf8400c00, No_VFP,
       &EmulateInstructionARM64::EmulateLDRSTRImm<AddrMode_PRE>,
       "LDR <Xt>, [<Xn|SP>, #<simm>]!"},
      {0xffc00000, 0xb9400000, No_VFP,
       &EmulateInstructionARM64::EmulateLDRSTRImm<AddrMode_OFF>,
       "LDR <Wt>, [<Xn|SP>{, #<pimm>}]"},
      {0xffc00000, 0xf9400000, No_VFP,
       &EmulateInstructionARM64::EmulateLDRSTRImm<AddrMode_OFF>,
       "LDR <Xt>, [<Xn|SP>{, #<pimm>}]"},

```
- **EN**: Contains supporting implementation details for the surrounding LLDB subsystem.
- **CN**: 包含周边 LLDB 子系统所需的辅助实现细节。

### Lines 349-371
```cpp
      {0x3f200c00, 0x3c000400, No_VFP,
       &EmulateInstructionARM64::EmulateLDRSTRImm<AddrMode_POST>,
       "LDR|STR <Bt|Ht|St|Dt|Qt>, [<Xn|SP>], #<simm>"},
      {0x3f200c00, 0x3c000c00, No_VFP,
       &EmulateInstructionARM64::EmulateLDRSTRImm<AddrMode_PRE>,
       "LDR|STR <Bt|Ht|St|Dt|Qt>, [<Xn|SP>, #<simm>]!"},
      {0x3f000000, 0x3d000000, No_VFP,
       &EmulateInstructionARM64::EmulateLDRSTRImm<AddrMode_OFF>,
       "LDR|STR <Bt|Ht|St|Dt|Qt>, [<Xn|SP>{, #<pimm>}]"},

      {0xfc000000, 0x14000000, No_VFP, &EmulateInstructionARM64::EmulateB,
       "B <label>"},
      {0xff000010, 0x54000000, No_VFP, &EmulateInstructionARM64::EmulateBcond,
       "B.<cond> <label>"},
      {0x7f000000, 0x34000000, No_VFP, &EmulateInstructionARM64::EmulateCBZ,
       "CBZ <Wt>, <label>"},
      {0x7f000000, 0x35000000, No_VFP, &EmulateInstructionARM64::EmulateCBZ,
       "CBNZ <Wt>, <label>"},
      {0x7f000000, 0x36000000, No_VFP, &EmulateInstructionARM64::EmulateTBZ,
       "TBZ <R><t>, #<imm>, <label>"},
      {0x7f000000, 0x37000000, No_VFP, &EmulateInstructionARM64::EmulateTBZ,
       "TBNZ <R><t>, #<imm>, <label>"},

```
- **EN**: Contains supporting implementation details for the surrounding LLDB subsystem.
- **CN**: 包含周边 LLDB 子系统所需的辅助实现细节。

### Lines 372-398
```cpp
  };
  static const size_t k_num_arm_opcodes = std::size(g_opcodes);

  for (size_t i = 0; i < k_num_arm_opcodes; ++i) {
    if ((g_opcodes[i].mask & opcode) == g_opcodes[i].value)
      return &g_opcodes[i];
  }
  return nullptr;
}

bool EmulateInstructionARM64::ReadInstruction() {
  bool success = false;
  m_addr = ReadRegisterUnsigned(eRegisterKindGeneric, LLDB_REGNUM_GENERIC_PC,
                                LLDB_INVALID_ADDRESS, &success);
  if (success) {
    Context read_inst_context;
    read_inst_context.type = eContextReadOpcode;
    read_inst_context.SetNoArgs();
    m_opcode.SetOpcode32(
        ReadMemoryUnsigned(read_inst_context, m_addr, 4, 0, &success),
        GetByteOrder());
  }
  if (!success)
    m_addr = LLDB_INVALID_ADDRESS;
  return success;
}

```
- **EN**: Implements logic around `size`, `ReadInstruction`, `ReadRegisterUnsigned`, `SetNoArgs`, and 3 more symbols.
- **CN**: 围绕 `size`, `ReadInstruction`, `ReadRegisterUnsigned`, `SetNoArgs`, and 3 more symbols 实现具体逻辑。

### Lines 399-416
```cpp
bool EmulateInstructionARM64::EvaluateInstruction(uint32_t evaluate_options) {
  const uint32_t opcode = m_opcode.GetOpcode32();
  Opcode *opcode_data = GetOpcodeForInstruction(opcode);
  if (opcode_data == nullptr)
    return false;

  const bool auto_advance_pc =
      evaluate_options & eEmulateInstructionOptionAutoAdvancePC;
  m_ignore_conditions =
      evaluate_options & eEmulateInstructionOptionIgnoreConditions;

  bool success = false;

  // Only return false if we are unable to read the CPSR if we care about
  // conditions
  if (!success && !m_ignore_conditions)
    return false;

```
- **EN**: Implements logic around `EvaluateInstruction`, `GetOpcode32`, `GetOpcodeForInstruction`.
- **CN**: 围绕 `EvaluateInstruction`, `GetOpcode32`, `GetOpcodeForInstruction` 实现具体逻辑。

### Lines 417-435
```cpp
  uint64_t orig_pc_value = 0;
  if (auto_advance_pc) {
    orig_pc_value =
        ReadRegisterUnsigned(eRegisterKindLLDB, gpr_pc_arm64, 0, &success);
    if (!success)
      return false;
  }

  // Call the Emulate... function.
  success = (this->*opcode_data->callback)(opcode);
  if (!success)
    return false;

  if (auto_advance_pc) {
    uint64_t new_pc_value =
        ReadRegisterUnsigned(eRegisterKindLLDB, gpr_pc_arm64, 0, &success);
    if (!success)
      return false;

```
- **EN**: Implements logic around `ReadRegisterUnsigned`.
- **CN**: 围绕 `ReadRegisterUnsigned` 实现具体逻辑。

### Lines 436-454
```cpp
    if (new_pc_value == orig_pc_value) {
      EmulateInstruction::Context context;
      context.type = eContextAdvancePC;
      context.SetNoArgs();
      if (!WriteRegisterUnsigned(context, eRegisterKindLLDB, gpr_pc_arm64,
                                 orig_pc_value + 4))
        return false;
    }
  }
  return true;
}

bool EmulateInstructionARM64::CreateFunctionEntryUnwind(
    UnwindPlan &unwind_plan) {
  unwind_plan.Clear();
  unwind_plan.SetRegisterKind(eRegisterKindLLDB);

  UnwindPlan::Row row;

```
- **EN**: Implements logic around `SetNoArgs`, `WriteRegisterUnsigned`, `CreateFunctionEntryUnwind`, `Clear`, and 1 more symbols.
- **CN**: 围绕 `SetNoArgs`, `WriteRegisterUnsigned`, `CreateFunctionEntryUnwind`, `Clear`, and 1 more symbols 实现具体逻辑。

### Lines 455-472
```cpp
  // Our previous Call Frame Address is the stack pointer
  row.GetCFAValue().SetIsRegisterPlusOffset(gpr_sp_arm64, 0);
  row.SetRegisterLocationToSame(gpr_lr_arm64, /*must_replace=*/false);
  row.SetRegisterLocationToSame(gpr_fp_arm64, /*must_replace=*/false);

  unwind_plan.AppendRow(std::move(row));
  unwind_plan.SetSourceName("EmulateInstructionARM64");
  unwind_plan.SetSourcedFromCompiler(eLazyBoolNo);
  unwind_plan.SetUnwindPlanValidAtAllInstructions(eLazyBoolYes);
  unwind_plan.SetUnwindPlanForSignalTrap(eLazyBoolNo);
  unwind_plan.SetReturnAddressRegister(gpr_lr_arm64);
  return true;
}

uint32_t EmulateInstructionARM64::GetFramePointerRegisterNumber() const {
  if (m_arch.GetTriple().isAndroid())
    return LLDB_INVALID_REGNUM; // Don't use frame pointer on android

```
- **EN**: Implements logic around `GetCFAValue`, `SetRegisterLocationToSame`, `AppendRow`, `SetSourceName`, and 6 more symbols; this block maps executable state back to modules, symbols, sections, or addresses; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `GetCFAValue`, `SetRegisterLocationToSame`, `AppendRow`, `SetSourceName`, and 6 more symbols 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 473-508
```cpp
  return gpr_fp_arm64;
}

bool EmulateInstructionARM64::UsingAArch32() {
  bool aarch32 = m_opcode_pstate.RW == 1;
  // if !HaveAnyAArch32() then assert !aarch32;
  // if HighestELUsingAArch32() then assert aarch32;
  return aarch32;
}

bool EmulateInstructionARM64::BranchTo(const Context &context, uint32_t N,
                                       addr_t target) {
#if 0
    // Set program counter to a new address, with a branch reason hint for
    // possible use by hardware fetching the next instruction.
    BranchTo(bits(N) target, BranchType branch_type)
        Hint_Branch(branch_type);
        if N == 32 then
            assert UsingAArch32();
            _PC = ZeroExtend(target);
        else
            assert N == 64 && !UsingAArch32();
            // Remove the tag bits from a tagged target
            case PSTATE.EL of
                when EL0, EL1
                    if target<55> == '1' && TCR_EL1.TBI1 == '1' then
                        target<63:56> = '11111111';
                    if target<55> == '0' && TCR_EL1.TBI0 == '1' then
                        target<63:56> = '00000000';
                when EL2
                    if TCR_EL2.TBI == '1' then
                        target<63:56> = '00000000';
                when EL3
                    if TCR_EL3.TBI == '1' then
                        target<63:56> = '00000000';
        _PC = target<63:0>;
```
- **EN**: Defines preprocessor-controlled structure, feature gates, or include-guard state.
- **CN**: 定义受预处理器控制的结构、特性开关或头文件保护状态。

### Lines 509-526
```cpp
        return;
#endif

  addr_t addr;

  // Hint_Branch(branch_type);
  if (N == 32) {
    if (!UsingAArch32())
      return false;
    addr = target;
  } else if (N == 64) {
    if (UsingAArch32())
      return false;
    // TODO: Remove the tag bits from a tagged target
    addr = target;
  } else
    return false;

```
- **EN**: Defines preprocessor-controlled structure, feature gates, or include-guard state.
- **CN**: 定义受预处理器控制的结构、特性开关或头文件保护状态。

### Lines 527-562
```cpp
  return WriteRegisterUnsigned(context, eRegisterKindGeneric,
                               LLDB_REGNUM_GENERIC_PC, addr);
}

bool EmulateInstructionARM64::ConditionHolds(const uint32_t cond) {
  // If we are ignoring conditions, then always return true. this allows us to
  // iterate over disassembly code and still emulate an instruction even if we
  // don't have all the right bits set in the CPSR register...
  if (m_ignore_conditions)
    return true;

  bool result = false;
  switch (UnsignedBits(cond, 3, 1)) {
  case 0:
    result = (m_opcode_pstate.Z == 1);
    break;
  case 1:
    result = (m_opcode_pstate.C == 1);
    break;
  case 2:
    result = (m_opcode_pstate.N == 1);
    break;
  case 3:
    result = (m_opcode_pstate.V == 1);
    break;
  case 4:
    result = (m_opcode_pstate.C == 1 && m_opcode_pstate.Z == 0);
    break;
  case 5:
    result = (m_opcode_pstate.N == m_opcode_pstate.V);
    break;
  case 6:
    result = (m_opcode_pstate.N == m_opcode_pstate.V && m_opcode_pstate.Z == 0);
    break;
  case 7:
    // Always execute (cond == 0b1110, or the special 0b1111 which gives
```
- **EN**: Implements logic around `WriteRegisterUnsigned`, `ConditionHolds`, `UnsignedBits`; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `WriteRegisterUnsigned`, `ConditionHolds`, `UnsignedBits` 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 563-589
```cpp
    // opcodes different meanings, but always means execution happens.
    return true;
  }

  if (cond & 1)
    result = !result;
  return result;
}

uint64_t EmulateInstructionARM64::
AddWithCarry(uint32_t N, uint64_t x, uint64_t y, bit carry_in,
             EmulateInstructionARM64::ProcState &proc_state) {
  uint64_t unsigned_sum = UInt(x) + UInt(y) + UInt(carry_in);
  std::optional<int64_t> signed_sum = llvm::checkedAdd(SInt(x), SInt(y));
  bool overflow = !signed_sum;
  if (!overflow)
    overflow |= !llvm::checkedAdd(*signed_sum, SInt(carry_in));
  uint64_t result = unsigned_sum;
  if (N < 64)
    result = Bits64(result, N - 1, 0);
  proc_state.N = Bit64(result, N - 1);
  proc_state.Z = IsZero(result);
  proc_state.C = UInt(result) != unsigned_sum;
  proc_state.V = overflow;
  return result;
}

```
- **EN**: Implements logic around `AddWithCarry`, `UInt`, `checkedAdd`, `Bits64`, and 2 more symbols.
- **CN**: 围绕 `AddWithCarry`, `UInt`, `checkedAdd`, `Bits64`, and 2 more symbols 实现具体逻辑。

### Lines 590-625
```cpp
bool EmulateInstructionARM64::EmulateADDSUBImm(const uint32_t opcode) {
  // integer d = UInt(Rd);
  // integer n = UInt(Rn);
  // integer datasize = if sf == 1 then 64 else 32;
  // boolean sub_op = (op == 1);
  // boolean setflags = (S == 1);
  // bits(datasize) imm;
  //
  // case shift of
  //     when '00' imm = ZeroExtend(imm12, datasize);
  //     when '01' imm = ZeroExtend(imm12 : Zeros(12), datasize);
  //    when '1x' UNDEFINED;
  //
  //
  // bits(datasize) result;
  // bits(datasize) operand1 = if n == 31 then SP[] else X[n];
  // bits(datasize) operand2 = imm;
  // bits(4) nzcv;
  // bit carry_in;
  //
  // if sub_op then
  //     operand2 = NOT(operand2);
  //     carry_in = 1;
  // else
  //     carry_in = 0;
  //
  // (result, nzcv) = AddWithCarry(operand1, operand2, carry_in);
  //
  // if setflags then
  //     PSTATE.NZCV = nzcv;
  //
  // if d == 31 && !setflags then
  //     SP[] = result;
  // else
  //     X[d] = result;

```
- **EN**: Implements logic around `EmulateADDSUBImm`.
- **CN**: 围绕 `EmulateADDSUBImm` 实现具体逻辑。

### Lines 626-658
```cpp
  const uint32_t sf = Bit32(opcode, 31);
  const uint32_t op = Bit32(opcode, 30);
  const uint32_t S = Bit32(opcode, 29);
  const uint32_t shift = Bits32(opcode, 23, 22);
  const uint32_t imm12 = Bits32(opcode, 21, 10);
  const uint32_t Rn = Bits32(opcode, 9, 5);
  const uint32_t Rd = Bits32(opcode, 4, 0);

  bool success = false;

  const uint32_t d = UInt(Rd);
  const uint32_t n = UInt(Rn);
  const uint32_t datasize = (sf == 1) ? 64 : 32;
  boolean sub_op = op == 1;
  boolean setflags = S == 1;
  uint64_t imm;

  switch (shift) {
  case 0:
    imm = imm12;
    break;
  case 1:
    imm = static_cast<uint64_t>(imm12) << 12;
    break;
  default:
    return false; // UNDEFINED;
  }
  uint64_t result;
  uint64_t operand1 =
      ReadRegisterUnsigned(eRegisterKindLLDB, gpr_x0_arm64 + n, 0, &success);
  uint64_t operand2 = imm;
  bit carry_in;

```
- **EN**: Implements logic around `Bit32`, `Bits32`, `UInt`, `static_cast`, and 1 more symbols.
- **CN**: 围绕 `Bit32`, `Bits32`, `UInt`, `static_cast`, and 1 more symbols 实现具体逻辑。

### Lines 659-677
```cpp
  if (sub_op) {
    operand2 = NOT(operand2);
    carry_in = true;
    imm = -imm; // For the Register plug offset context below
  } else {
    carry_in = false;
  }

  ProcState proc_state;

  result = AddWithCarry(datasize, operand1, operand2, carry_in, proc_state);

  if (setflags) {
    m_emulated_pstate.N = proc_state.N;
    m_emulated_pstate.Z = proc_state.Z;
    m_emulated_pstate.C = proc_state.C;
    m_emulated_pstate.V = proc_state.V;
  }

```
- **EN**: Implements logic around `NOT`, `AddWithCarry`; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `NOT`, `AddWithCarry` 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 678-697
```cpp
  Context context;
  std::optional<RegisterInfo> reg_info_Rn =
      GetRegisterInfo(eRegisterKindLLDB, n);
  if (reg_info_Rn)
    context.SetRegisterPlusOffset(*reg_info_Rn, imm);

  if (n == GetFramePointerRegisterNumber() && d == gpr_sp_arm64 && !setflags) {
    // 'mov sp, fp' - common epilogue instruction, CFA is now in terms of the
    // stack pointer, instead of frame pointer.
    context.type = EmulateInstruction::eContextRestoreStackPointer;
  } else if ((n == gpr_sp_arm64 || n == GetFramePointerRegisterNumber()) &&
             d == gpr_sp_arm64 && !setflags) {
    context.type = EmulateInstruction::eContextAdjustStackPointer;
  } else if (d == GetFramePointerRegisterNumber() && n == gpr_sp_arm64 &&
             !setflags) {
    context.type = EmulateInstruction::eContextSetFramePointer;
  } else {
    context.type = EmulateInstruction::eContextImmediate;
  }

```
- **EN**: Implements logic around `GetRegisterInfo`, `SetRegisterPlusOffset`, `GetFramePointerRegisterNumber`; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `GetRegisterInfo`, `SetRegisterPlusOffset`, `GetFramePointerRegisterNumber` 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 698-719
```cpp
  // If setflags && d == gpr_sp_arm64 then d = WZR/XZR. See CMN, CMP
  if (!setflags || d != gpr_sp_arm64)
    WriteRegisterUnsigned(context, eRegisterKindLLDB, gpr_x0_arm64 + d, result);

  return false;
}

template <EmulateInstructionARM64::AddrMode a_mode>
bool EmulateInstructionARM64::EmulateLDPSTP(const uint32_t opcode) {
  uint32_t opc = Bits32(opcode, 31, 30);
  uint32_t V = Bit32(opcode, 26);
  uint32_t L = Bit32(opcode, 22);
  uint32_t imm7 = Bits32(opcode, 21, 15);
  uint32_t Rt2 = Bits32(opcode, 14, 10);
  uint32_t Rn = Bits32(opcode, 9, 5);
  uint32_t Rt = Bits32(opcode, 4, 0);

  integer n = UInt(Rn);
  integer t = UInt(Rt);
  integer t2 = UInt(Rt2);
  uint64_t idx;

```
- **EN**: Implements logic around `WriteRegisterUnsigned`, `EmulateLDPSTP`, `Bits32`, `Bit32`, and 1 more symbols.
- **CN**: 围绕 `WriteRegisterUnsigned`, `EmulateLDPSTP`, `Bits32`, `Bit32`, and 1 more symbols 实现具体逻辑。

### Lines 720-741
```cpp
  MemOp memop = L == 1 ? MemOp_LOAD : MemOp_STORE;
  boolean vector = (V == 1);
  // AccType acctype = AccType_NORMAL;
  boolean is_signed = false;
  boolean wback = a_mode != AddrMode_OFF;
  boolean wb_unknown = false;
  boolean rt_unknown = false;
  integer scale;
  integer size;

  if (opc == 3)
    return false; // UNDEFINED

  if (vector) {
    scale = 2 + UInt(opc);
  } else {
    scale = (opc & 2) ? 3 : 2;
    is_signed = (opc & 1) != 0;
    if (is_signed && memop == MemOp_STORE)
      return false; // UNDEFINED
  }

```
- **EN**: Implements logic around `UInt`.
- **CN**: 围绕 `UInt` 实现具体逻辑。

### Lines 742-761
```cpp
  if (!vector && wback && ((t == n) || (t2 == n))) {
    switch (ConstrainUnpredictable(Unpredictable_WBOVERLAP)) {
    case Constraint_UNKNOWN:
      wb_unknown = true; // writeback is UNKNOWN
      break;

    case Constraint_SUPPRESSWB:
      wback = false; // writeback is suppressed
      break;

    case Constraint_NOP:
      memop = MemOp_NOP; // do nothing
      wback = false;
      break;

    case Constraint_NONE:
      break;
    }
  }

```
- **EN**: Implements logic around `ConstrainUnpredictable`.
- **CN**: 围绕 `ConstrainUnpredictable` 实现具体逻辑。

### Lines 762-783
```cpp
  if (memop == MemOp_LOAD && t == t2) {
    switch (ConstrainUnpredictable(Unpredictable_LDPOVERLAP)) {
    case Constraint_UNKNOWN:
      rt_unknown = true; // result is UNKNOWN
      break;

    case Constraint_NOP:
      memop = MemOp_NOP; // do nothing
      wback = false;
      break;

    default:
      break;
    }
  }

  idx = LSL(llvm::SignExtend64<7>(imm7), scale);
  size = (integer)1 << scale;
  uint64_t datasize = size * 8;
  uint64_t address;
  uint64_t wb_address;

```
- **EN**: Implements logic around `ConstrainUnpredictable`, `LSL`; this block maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `ConstrainUnpredictable`, `LSL` 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址。

### Lines 784-802
```cpp
  std::optional<RegisterInfo> reg_info_base =
      GetRegisterInfo(eRegisterKindLLDB, gpr_x0_arm64 + n);
  if (!reg_info_base)
    return false;

  std::optional<RegisterInfo> reg_info_Rt;
  std::optional<RegisterInfo> reg_info_Rt2;

  if (vector) {
    reg_info_Rt = GetRegisterInfo(eRegisterKindLLDB, fpu_d0_arm64 + t);
    reg_info_Rt2 = GetRegisterInfo(eRegisterKindLLDB, fpu_d0_arm64 + t2);
  } else {
    reg_info_Rt = GetRegisterInfo(eRegisterKindLLDB, gpr_x0_arm64 + t);
    reg_info_Rt2 = GetRegisterInfo(eRegisterKindLLDB, gpr_x0_arm64 + t2);
  }

  if (!reg_info_Rt || !reg_info_Rt2)
    return false;

```
- **EN**: Implements logic around `GetRegisterInfo`.
- **CN**: 围绕 `GetRegisterInfo` 实现具体逻辑。

### Lines 803-821
```cpp
  bool success = false;
  if (n == 31) {
    // CheckSPAlignment();
    address =
        ReadRegisterUnsigned(eRegisterKindLLDB, gpr_sp_arm64, 0, &success);
  } else
    address =
        ReadRegisterUnsigned(eRegisterKindLLDB, gpr_x0_arm64 + n, 0, &success);

  wb_address = address + idx;
  if (a_mode != AddrMode_POST)
    address = wb_address;

  Context context_t;
  Context context_t2;

  RegisterValue::BytesContainer buffer;
  Status error;

```
- **EN**: Implements logic around `ReadRegisterUnsigned`; this block propagates recoverable errors, status objects, or diagnostics; maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `ReadRegisterUnsigned` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并将可执行状态映射回模块、符号、节区或地址。

### Lines 822-841
```cpp
  switch (memop) {
  case MemOp_STORE: {
    if (n == 31 || n == GetFramePointerRegisterNumber()) // if this store is
                                                         // based off of the sp
                                                         // or fp register
    {
      context_t.type = eContextPushRegisterOnStack;
      context_t2.type = eContextPushRegisterOnStack;
    } else {
      context_t.type = eContextRegisterStore;
      context_t2.type = eContextRegisterStore;
    }
    context_t.SetRegisterToRegisterPlusOffset(*reg_info_Rt, *reg_info_base, 0);
    context_t2.SetRegisterToRegisterPlusOffset(*reg_info_Rt2, *reg_info_base,
                                               size);

    std::optional<RegisterValue> data_Rt = ReadRegister(*reg_info_Rt);
    if (!data_Rt)
      return false;

```
- **EN**: Implements logic around `GetFramePointerRegisterNumber`, `SetRegisterToRegisterPlusOffset`, `ReadRegister`; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `GetFramePointerRegisterNumber`, `SetRegisterToRegisterPlusOffset`, `ReadRegister` 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 842-861
```cpp
    buffer.resize(reg_info_Rt->byte_size);
    if (data_Rt->GetAsMemoryData(*reg_info_Rt, buffer.data(),
                                 reg_info_Rt->byte_size, eByteOrderLittle,
                                 error) == 0)
      return false;

    if (!WriteMemory(context_t, address + 0, buffer.data(),
                     reg_info_Rt->byte_size))
      return false;

    std::optional<RegisterValue> data_Rt2 = ReadRegister(*reg_info_Rt2);
    if (!data_Rt2)
      return false;

    buffer.resize(reg_info_Rt2->byte_size);
    if (data_Rt2->GetAsMemoryData(*reg_info_Rt2, buffer.data(),
                                  reg_info_Rt2->byte_size, eByteOrderLittle,
                                  error) == 0)
      return false;

```
- **EN**: Implements logic around `resize`, `GetAsMemoryData`, `WriteMemory`, `ReadRegister`; this block propagates recoverable errors, status objects, or diagnostics; maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `resize`, `GetAsMemoryData`, `WriteMemory`, `ReadRegister` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并将可执行状态映射回模块、符号、节区或地址。

### Lines 862-880
```cpp
    if (!WriteMemory(context_t2, address + size, buffer.data(),
                     reg_info_Rt2->byte_size))
      return false;
  } break;

  case MemOp_LOAD: {
    if (n == 31 || n == GetFramePointerRegisterNumber()) // if this load is
                                                         // based off of the sp
                                                         // or fp register
    {
      context_t.type = eContextPopRegisterOffStack;
      context_t2.type = eContextPopRegisterOffStack;
    } else {
      context_t.type = eContextRegisterLoad;
      context_t2.type = eContextRegisterLoad;
    }
    context_t.SetAddress(address);
    context_t2.SetAddress(address + size);

```
- **EN**: Implements logic around `WriteMemory`, `GetFramePointerRegisterNumber`, `SetAddress`; this block maps executable state back to modules, symbols, sections, or addresses; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `WriteMemory`, `GetFramePointerRegisterNumber`, `SetAddress` 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 881-898
```cpp
    buffer.resize(reg_info_Rt->byte_size);
    if (rt_unknown)
      std::fill(buffer.begin(), buffer.end(), 'U');
    else {
      if (!ReadMemory(context_t, address, buffer.data(),
                      reg_info_Rt->byte_size))
        return false;
    }

    RegisterValue data_Rt;
    if (data_Rt.SetFromMemoryData(*reg_info_Rt, buffer.data(),
                                  reg_info_Rt->byte_size, eByteOrderLittle,
                                  error) == 0)
      return false;

    if (!vector && is_signed && !data_Rt.SignExtend(datasize))
      return false;

```
- **EN**: Implements logic around `resize`, `fill`, `ReadMemory`, `SetFromMemoryData`, and 1 more symbols; this block propagates recoverable errors, status objects, or diagnostics; maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `resize`, `fill`, `ReadMemory`, `SetFromMemoryData`, and 1 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并将可执行状态映射回模块、符号、节区或地址。

### Lines 899-916
```cpp
    if (!WriteRegister(context_t, *reg_info_Rt, data_Rt))
      return false;

    buffer.resize(reg_info_Rt2->byte_size);
    if (!rt_unknown)
      if (!ReadMemory(context_t2, address + size, buffer.data(),
                      reg_info_Rt2->byte_size))
        return false;

    RegisterValue data_Rt2;
    if (data_Rt2.SetFromMemoryData(*reg_info_Rt2, buffer.data(),
                                   reg_info_Rt2->byte_size, eByteOrderLittle,
                                   error) == 0)
      return false;

    if (!vector && is_signed && !data_Rt2.SignExtend(datasize))
      return false;

```
- **EN**: Implements logic around `WriteRegister`, `resize`, `ReadMemory`, `SetFromMemoryData`, and 1 more symbols; this block propagates recoverable errors, status objects, or diagnostics; maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `WriteRegister`, `resize`, `ReadMemory`, `SetFromMemoryData`, and 1 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并将可执行状态映射回模块、符号、节区或地址。

### Lines 917-938
```cpp
    if (!WriteRegister(context_t2, *reg_info_Rt2, data_Rt2))
      return false;
  } break;

  default:
    break;
  }

  if (wback) {
    if (wb_unknown)
      wb_address = LLDB_INVALID_ADDRESS;
    Context context;
    context.SetImmediateSigned(idx);
    if (n == 31)
      context.type = eContextAdjustStackPointer;
    else
      context.type = eContextAdjustBaseRegister;
    WriteRegisterUnsigned(context, *reg_info_base, wb_address);
  }
  return true;
}

```
- **EN**: Implements logic around `WriteRegister`, `SetImmediateSigned`, `WriteRegisterUnsigned`.
- **CN**: 围绕 `WriteRegister`, `SetImmediateSigned`, `WriteRegisterUnsigned` 实现具体逻辑。

### Lines 939-965
```cpp
template <EmulateInstructionARM64::AddrMode a_mode>
bool EmulateInstructionARM64::EmulateLDRSTRImm(const uint32_t opcode) {
  uint32_t size = Bits32(opcode, 31, 30);
  uint32_t opc = Bits32(opcode, 23, 22);
  uint32_t vr = Bit32(opcode, 26);
  uint32_t n = Bits32(opcode, 9, 5);
  uint32_t t = Bits32(opcode, 4, 0);

  MemOp memop;
  if (vr) {
    // opc<1> == 1 && size != 0 is an undefined encoding.
    if (Bit32(opc, 1) == 1 && size != 0)
      return false;
    // opc<1> == 1 && size == 0 encode the 128-bit variant.
    if (Bit32(opc, 1) == 1)
      size = 4;
    memop = Bit32(opc, 0) == 1 ? MemOp_LOAD : MemOp_STORE;
  } else {
    if (Bit32(opc, 1) == 0) {
      memop = Bit32(opc, 0) == 1 ? MemOp_LOAD : MemOp_STORE;
    } else {
      memop = MemOp_LOAD;
      if (size == 2 && Bit32(opc, 0) == 1)
        return false;
    }
  }

```
- **EN**: Implements logic around `EmulateLDRSTRImm`, `Bits32`, `Bit32`.
- **CN**: 围绕 `EmulateLDRSTRImm`, `Bits32`, `Bit32` 实现具体逻辑。

### Lines 966-987
```cpp
  bool wback;
  bool postindex;
  uint64_t offset;

  switch (a_mode) {
  case AddrMode_POST:
    wback = true;
    postindex = true;
    offset = llvm::SignExtend64<9>(Bits32(opcode, 20, 12));
    break;
  case AddrMode_PRE:
    wback = true;
    postindex = false;
    offset = llvm::SignExtend64<9>(Bits32(opcode, 20, 12));
    break;
  case AddrMode_OFF:
    wback = false;
    postindex = false;
    offset = LSL(Bits32(opcode, 21, 10), size);
    break;
  }

```
- **EN**: Implements logic around `SignExtend64`, `LSL`.
- **CN**: 围绕 `SignExtend64`, `LSL` 实现具体逻辑。

### Lines 988-1005
```cpp
  Status error;
  bool success = false;
  uint64_t address;
  RegisterValue::BytesContainer buffer;

  if (n == 31)
    address =
        ReadRegisterUnsigned(eRegisterKindLLDB, gpr_sp_arm64, 0, &success);
  else
    address =
        ReadRegisterUnsigned(eRegisterKindLLDB, gpr_x0_arm64 + n, 0, &success);

  if (!success)
    return false;

  if (!postindex)
    address += offset;

```
- **EN**: Implements logic around `ReadRegisterUnsigned`; this block propagates recoverable errors, status objects, or diagnostics; maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `ReadRegisterUnsigned` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并将可执行状态映射回模块、符号、节区或地址。

### Lines 1006-1028
```cpp
  std::optional<RegisterInfo> reg_info_base =
      GetRegisterInfo(eRegisterKindLLDB, gpr_x0_arm64 + n);
  if (!reg_info_base)
    return false;

  std::optional<RegisterInfo> reg_info_Rt =
      vr ? GetRegisterInfo(eRegisterKindLLDB, fpu_d0_arm64 + t)
         : GetRegisterInfo(eRegisterKindLLDB, gpr_x0_arm64 + t);
  if (!reg_info_Rt)
    return false;

  Context context;
  switch (memop) {
  case MemOp_STORE: {
    if (n == 31 || n == GetFramePointerRegisterNumber()) // if this store is
                                                         // based off of the sp
                                                         // or fp register
      context.type = eContextPushRegisterOnStack;
    else
      context.type = eContextRegisterStore;
    context.SetRegisterToRegisterPlusOffset(*reg_info_Rt, *reg_info_base,
                                            postindex ? 0 : offset);

```
- **EN**: Implements logic around `GetRegisterInfo`, `GetFramePointerRegisterNumber`, `SetRegisterToRegisterPlusOffset`; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `GetRegisterInfo`, `GetFramePointerRegisterNumber`, `SetRegisterToRegisterPlusOffset` 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 1029-1051
```cpp
    std::optional<RegisterValue> data_Rt = ReadRegister(*reg_info_Rt);
    if (!data_Rt)
      return false;

    buffer.resize(reg_info_Rt->byte_size);
    if (data_Rt->GetAsMemoryData(*reg_info_Rt, buffer.data(),
                                 reg_info_Rt->byte_size, eByteOrderLittle,
                                 error) == 0)
      return false;

    if (!WriteMemory(context, address, buffer.data(), reg_info_Rt->byte_size))
      return false;
  } break;

  case MemOp_LOAD: {
    if (n == 31 || n == GetFramePointerRegisterNumber()) // if this store is
                                                         // based off of the sp
                                                         // or fp register
      context.type = eContextPopRegisterOffStack;
    else
      context.type = eContextRegisterLoad;
    context.SetAddress(address);

```
- **EN**: Implements logic around `ReadRegister`, `resize`, `GetAsMemoryData`, `WriteMemory`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics; maps executable state back to modules, symbols, sections, or addresses; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `ReadRegister`, `resize`, `GetAsMemoryData`, `WriteMemory`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并将可执行状态映射回模块、符号、节区或地址，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 1052-1072
```cpp
    buffer.resize(reg_info_Rt->byte_size);
    if (!ReadMemory(context, address, buffer.data(), reg_info_Rt->byte_size))
      return false;

    RegisterValue data_Rt;
    if (data_Rt.SetFromMemoryData(*reg_info_Rt, buffer.data(),
                                  reg_info_Rt->byte_size, eByteOrderLittle,
                                  error) == 0)
      return false;

    if (!WriteRegister(context, *reg_info_Rt, data_Rt))
      return false;
  } break;
  default:
    return false;
  }

  if (wback) {
    if (postindex)
      address += offset;

```
- **EN**: Implements logic around `resize`, `ReadMemory`, `SetFromMemoryData`, `WriteRegister`; this block propagates recoverable errors, status objects, or diagnostics; maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `resize`, `ReadMemory`, `SetFromMemoryData`, `WriteRegister` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并将可执行状态映射回模块、符号、节区或地址。

### Lines 1073-1091
```cpp
    if (n == 31)
      context.type = eContextAdjustStackPointer;
    else
      context.type = eContextAdjustBaseRegister;
    context.SetImmediateSigned(offset);

    if (!WriteRegisterUnsigned(context, *reg_info_base, address))
      return false;
  }
  return true;
}

bool EmulateInstructionARM64::EmulateB(const uint32_t opcode) {
#if 0
    // ARM64 pseudo code...
    if branch_type == BranchType_CALL then X[30] = PC[] + 4;
    BranchTo(PC[] + offset, branch_type);
#endif

```
- **EN**: Defines preprocessor-controlled structure, feature gates, or include-guard state.
- **CN**: 定义受预处理器控制的结构、特性开关或头文件保护状态。

### Lines 1092-1117
```cpp
  bool success = false;

  EmulateInstruction::Context context;
  context.type = EmulateInstruction::eContextRelativeBranchImmediate;
  const uint64_t pc = ReadRegisterUnsigned(eRegisterKindGeneric,
                                           LLDB_REGNUM_GENERIC_PC, 0, &success);
  if (!success)
    return false;

  int64_t offset = llvm::SignExtend64<28>(Bits32(opcode, 25, 0) << 2);
  BranchType branch_type = Bit32(opcode, 31) ? BranchType_CALL : BranchType_JMP;
  addr_t target = pc + offset;
  context.SetImmediateSigned(offset);

  switch (branch_type) {
  case BranchType_CALL: {
    addr_t x30 = pc + 4;
    if (!WriteRegisterUnsigned(context, eRegisterKindLLDB, gpr_lr_arm64, x30))
      return false;
  } break;
  case BranchType_JMP:
    break;
  default:
    return false;
  }

```
- **EN**: Implements logic around `ReadRegisterUnsigned`, `SignExtend64`, `Bit32`, `SetImmediateSigned`, and 1 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `ReadRegisterUnsigned`, `SignExtend64`, `Bit32`, `SetImmediateSigned`, and 1 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 1118-1137
```cpp
  return BranchTo(context, 64, target);
}

bool EmulateInstructionARM64::EmulateBcond(const uint32_t opcode) {
#if 0
    // ARM64 pseudo code...
    bits(64) offset = SignExtend(imm19:'00', 64);
    bits(4) condition = cond;
    if ConditionHolds(condition) then
        BranchTo(PC[] + offset, BranchType_JMP);
#endif

  if (ConditionHolds(Bits32(opcode, 3, 0))) {
    bool success = false;

    const uint64_t pc = ReadRegisterUnsigned(
        eRegisterKindGeneric, LLDB_REGNUM_GENERIC_PC, 0, &success);
    if (!success)
      return false;

```
- **EN**: Defines preprocessor-controlled structure, feature gates, or include-guard state.
- **CN**: 定义受预处理器控制的结构、特性开关或头文件保护状态。

### Lines 1138-1156
```cpp
    int64_t offset = llvm::SignExtend64<21>(Bits32(opcode, 23, 5) << 2);
    addr_t target = pc + offset;

    EmulateInstruction::Context context;
    context.type = EmulateInstruction::eContextRelativeBranchImmediate;
    context.SetImmediateSigned(offset);
    if (!BranchTo(context, 64, target))
      return false;
  }
  return true;
}

bool EmulateInstructionARM64::EmulateCBZ(const uint32_t opcode) {
#if 0
    integer t = UInt(Rt);
    integer datasize = if sf == '1' then 64 else 32;
    boolean iszero = (op == '0');
    bits(64) offset = SignExtend(imm19:'00', 64);

```
- **EN**: Defines preprocessor-controlled structure, feature gates, or include-guard state.
- **CN**: 定义受预处理器控制的结构、特性开关或头文件保护状态。

### Lines 1157-1178
```cpp
    bits(datasize) operand1 = X[t];
    if IsZero(operand1) == iszero then
        BranchTo(PC[] + offset, BranchType_JMP);
#endif

  bool success = false;

  uint32_t t = Bits32(opcode, 4, 0);
  bool is_zero = Bit32(opcode, 24) == 0;
  int32_t offset = llvm::SignExtend64<21>(Bits32(opcode, 23, 5) << 2);

  const uint64_t operand =
      ReadRegisterUnsigned(eRegisterKindLLDB, gpr_x0_arm64 + t, 0, &success);
  if (!success)
    return false;

  if (m_ignore_conditions || ((operand == 0) == is_zero)) {
    const uint64_t pc = ReadRegisterUnsigned(
        eRegisterKindGeneric, LLDB_REGNUM_GENERIC_PC, 0, &success);
    if (!success)
      return false;

```
- **EN**: Defines preprocessor-controlled structure, feature gates, or include-guard state.
- **CN**: 定义受预处理器控制的结构、特性开关或头文件保护状态。

### Lines 1179-1196
```cpp
    EmulateInstruction::Context context;
    context.type = EmulateInstruction::eContextRelativeBranchImmediate;
    context.SetImmediateSigned(offset);
    if (!BranchTo(context, 64, pc + offset))
      return false;
  }
  return true;
}

bool EmulateInstructionARM64::EmulateTBZ(const uint32_t opcode) {
#if 0
    integer t = UInt(Rt);
    integer datasize = if b5 == '1' then 64 else 32;
    integer bit_pos = UInt(b5:b40);
    bit bit_val = op;
    bits(64) offset = SignExtend(imm14:'00', 64);
#endif

```
- **EN**: Defines preprocessor-controlled structure, feature gates, or include-guard state.
- **CN**: 定义受预处理器控制的结构、特性开关或头文件保护状态。

### Lines 1197-1214
```cpp
  bool success = false;

  uint32_t t = Bits32(opcode, 4, 0);
  uint32_t bit_pos = (Bit32(opcode, 31) << 6) | (Bits32(opcode, 23, 19));
  uint32_t bit_val = Bit32(opcode, 24);
  int64_t offset = llvm::SignExtend64<16>(Bits32(opcode, 18, 5) << 2);

  const uint64_t operand =
      ReadRegisterUnsigned(eRegisterKindLLDB, gpr_x0_arm64 + t, 0, &success);
  if (!success)
    return false;

  if (m_ignore_conditions || Bit32(operand, bit_pos) == bit_val) {
    const uint64_t pc = ReadRegisterUnsigned(
        eRegisterKindGeneric, LLDB_REGNUM_GENERIC_PC, 0, &success);
    if (!success)
      return false;

```
- **EN**: Implements logic around `Bits32`, `Bit32`, `SignExtend64`, `ReadRegisterUnsigned`.
- **CN**: 围绕 `Bits32`, `Bit32`, `SignExtend64`, `ReadRegisterUnsigned` 实现具体逻辑。

### Lines 1215-1222
```cpp
    EmulateInstruction::Context context;
    context.type = EmulateInstruction::eContextRelativeBranchImmediate;
    context.SetImmediateSigned(offset);
    if (!BranchTo(context, 64, pc + offset))
      return false;
  }
  return true;
}
```
- **EN**: Implements logic around `SetImmediateSigned`, `BranchTo`.
- **CN**: 围绕 `SetImmediateSigned`, `BranchTo` 实现具体逻辑。

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

- **Direct LLDB/LLVM includes / 直接的 LLDB/LLVM 包含**: `EmulateInstructionARM64.h`, `lldb/Core/Address.h`, `lldb/Core/PluginManager.h`, `lldb/Symbol/UnwindPlan.h`, `lldb/Utility/ArchSpec.h`, `lldb/Utility/RegisterValue.h`, `lldb/Utility/Stream.h`, `llvm/Support/CheckedArithmetic.h`, `Plugins/Process/Utility/ARMDefines.h`, `Plugins/Process/Utility/ARMUtils.h` ... (+5 more)
- **Standard-library headers / 标准库头文件**: `<algorithm>`, `<cstdlib>`, `<optional>`
- **Subsystem categories / 子系统类别**: shared LLDB utility classes / 共享 LLDB 工具类 (3), LLDB core debugger abstractions / LLDB 核心调试抽象 (2), LLVM support-library helpers / LLVM Support 库辅助组件 (2), symbol and debug-info abstractions / 符号与调试信息抽象 (1), LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (1)
