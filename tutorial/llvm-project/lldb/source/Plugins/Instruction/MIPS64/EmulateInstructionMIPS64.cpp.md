# EmulateInstructionMIPS64.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/source/Plugins/Instruction/MIPS64/EmulateInstructionMIPS64.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements the LLDB support logic associated with `EmulateInstructionMIPS64`.
  - **CN**: 实现与 `EmulateInstructionMIPS64` 相关的 LLDB 支持逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-35
```cpp
//===-- EmulateInstructionMIPS64.cpp --------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "EmulateInstructionMIPS64.h"

#include <cstdlib>
#include <optional>

#include "lldb/Core/Address.h"
#include "lldb/Core/Opcode.h"
#include "lldb/Core/PluginManager.h"
#include "lldb/Host/PosixApi.h"
#include "lldb/Symbol/UnwindPlan.h"
#include "lldb/Utility/ArchSpec.h"
#include "lldb/Utility/ConstString.h"
#include "lldb/Utility/DataExtractor.h"
#include "lldb/Utility/RegisterValue.h"
#include "lldb/Utility/Stream.h"
#include "llvm-c/Disassembler.h"
#include "llvm/MC/MCAsmInfo.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCDisassembler/MCDisassembler.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCInstrInfo.h"
#include "llvm/MC/MCRegisterInfo.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/MC/MCTargetOptions.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/TargetSelect.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `EmulateInstructionMIPS64.h`, `cstdlib`, `optional`, `lldb/Core/Address.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `EmulateInstructionMIPS64.h`, `cstdlib`, `optional`, `lldb/Core/Address.h`。

### Lines 36-62
```cpp
#include "llvm/ADT/STLExtras.h"

#include "Plugins/Process/Utility/InstructionUtils.h"
#include "Plugins/Process/Utility/RegisterContext_mips.h"

using namespace lldb;
using namespace lldb_private;

LLDB_PLUGIN_DEFINE_ADV(EmulateInstructionMIPS64, InstructionMIPS64)

#define UInt(x) ((uint64_t)x)
#define integer int64_t

//
// EmulateInstructionMIPS64 implementation
//

#ifdef __mips__
extern "C" {
void LLVMInitializeMipsTargetInfo();
void LLVMInitializeMipsTarget();
void LLVMInitializeMipsAsmPrinter();
void LLVMInitializeMipsTargetMC();
void LLVMInitializeMipsDisassembler();
}
#endif

```
- **EN**: Pulls in the headers needed by this translation unit, including `llvm/ADT/STLExtras.h`, `Plugins/Process/Utility/InstructionUtils.h`, `Plugins/Process/Utility/RegisterContext_mips.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `llvm/ADT/STLExtras.h`, `Plugins/Process/Utility/InstructionUtils.h`, `Plugins/Process/Utility/RegisterContext_mips.h`。

### Lines 63-90
```cpp
EmulateInstructionMIPS64::EmulateInstructionMIPS64(
    const lldb_private::ArchSpec &arch)
    : EmulateInstruction(arch) {
  /* Create instance of llvm::MCDisassembler */
  std::string Status;
  llvm::Triple triple = arch.GetTriple();
  const llvm::Target *target =
      llvm::TargetRegistry::lookupTarget(triple, Status);

/*
 * If we fail to get the target then we haven't registered it. The
 * SystemInitializerCommon
 * does not initialize targets, MCs and disassemblers. However we need the
 * MCDisassembler
 * to decode the instructions so that the decoding complexity stays with LLVM.
 * Initialize the MIPS targets and disassemblers.
*/
#ifdef __mips__
  if (!target) {
    LLVMInitializeMipsTargetInfo();
    LLVMInitializeMipsTarget();
    LLVMInitializeMipsAsmPrinter();
    LLVMInitializeMipsTargetMC();
    LLVMInitializeMipsDisassembler();
    target = llvm::TargetRegistry::lookupTarget(triple, Status);
  }
#endif

```
- **EN**: Defines preprocessor-controlled structure, feature gates, or include-guard state.
- **CN**: 定义受预处理器控制的结构、特性开关或头文件保护状态。

### Lines 91-126
```cpp
  assert(target);

  llvm::StringRef cpu;

  switch (arch.GetCore()) {
  case ArchSpec::eCore_mips32:
  case ArchSpec::eCore_mips32el:
    cpu = "mips32";
    break;
  case ArchSpec::eCore_mips32r2:
  case ArchSpec::eCore_mips32r2el:
    cpu = "mips32r2";
    break;
  case ArchSpec::eCore_mips32r3:
  case ArchSpec::eCore_mips32r3el:
    cpu = "mips32r3";
    break;
  case ArchSpec::eCore_mips32r5:
  case ArchSpec::eCore_mips32r5el:
    cpu = "mips32r5";
    break;
  case ArchSpec::eCore_mips32r6:
  case ArchSpec::eCore_mips32r6el:
    cpu = "mips32r6";
    break;
  case ArchSpec::eCore_mips64:
  case ArchSpec::eCore_mips64el:
    cpu = "mips64";
    break;
  case ArchSpec::eCore_mips64r2:
  case ArchSpec::eCore_mips64r2el:
    cpu = "mips64r2";
    break;
  case ArchSpec::eCore_mips64r3:
  case ArchSpec::eCore_mips64r3el:
    cpu = "mips64r3";
```
- **EN**: Implements logic around `assert`, `GetCore`; this block coordinates debugger runtime objects, events, or asynchronous control flow; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `assert`, `GetCore` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 127-153
```cpp
    break;
  case ArchSpec::eCore_mips64r5:
  case ArchSpec::eCore_mips64r5el:
    cpu = "mips64r5";
    break;
  case ArchSpec::eCore_mips64r6:
  case ArchSpec::eCore_mips64r6el:
    cpu = "mips64r6";
    break;
  default:
    cpu = "generic";
    break;
  }

  std::string features;
  uint32_t arch_flags = arch.GetFlags();
  if (arch_flags & ArchSpec::eMIPSAse_msa)
    features += "+msa,";
  if (arch_flags & ArchSpec::eMIPSAse_dsp)
    features += "+dsp,";
  if (arch_flags & ArchSpec::eMIPSAse_dspr2)
    features += "+dspr2,";
  if (arch_flags & ArchSpec::eMIPSAse_mips16)
    features += "+mips16,";
  if (arch_flags & ArchSpec::eMIPSAse_micromips)
    features += "+micromips,";

```
- **EN**: Implements logic around `GetFlags`; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `GetFlags` 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 154-171
```cpp
  m_reg_info.reset(target->createMCRegInfo(triple));
  assert(m_reg_info.get());

  m_insn_info.reset(target->createMCInstrInfo());
  assert(m_insn_info.get());

  m_asm_info.reset(target->createMCAsmInfo(*m_reg_info, triple, m_mc_options));
  m_subtype_info.reset(target->createMCSubtargetInfo(triple, cpu, features));
  assert(m_asm_info.get() && m_subtype_info.get());

  m_context = std::make_unique<llvm::MCContext>(triple, *m_asm_info,
                                                *m_reg_info, *m_subtype_info);
  assert(m_context.get());

  m_disasm.reset(target->createMCDisassembler(*m_subtype_info, *m_context));
  assert(m_disasm.get());
}

```
- **EN**: Implements logic around `reset`, `assert`, `MCContext>`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `reset`, `assert`, `MCContext>` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 172-195
```cpp
void EmulateInstructionMIPS64::Initialize() {
  PluginManager::RegisterPlugin(GetPluginNameStatic(),
                                GetPluginDescriptionStatic(), CreateInstance);
}

void EmulateInstructionMIPS64::Terminate() {
  PluginManager::UnregisterPlugin(CreateInstance);
}

llvm::StringRef EmulateInstructionMIPS64::GetPluginDescriptionStatic() {
  return "Emulate instructions for the MIPS64 architecture.";
}

EmulateInstruction *
EmulateInstructionMIPS64::CreateInstance(const ArchSpec &arch,
                                         InstructionType inst_type) {
  if (EmulateInstructionMIPS64::SupportsEmulatingInstructionsOfTypeStatic(
          inst_type)) {
    if (arch.GetTriple().getArch() == llvm::Triple::mips64 ||
        arch.GetTriple().getArch() == llvm::Triple::mips64el) {
      return new EmulateInstructionMIPS64(arch);
    }
  }

```
- **EN**: Implements logic around `Initialize`, `RegisterPlugin`, `GetPluginDescriptionStatic`, `Terminate`, and 5 more symbols; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `Initialize`, `RegisterPlugin`, `GetPluginDescriptionStatic`, `Terminate`, and 5 more symbols 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 196-231
```cpp
  return nullptr;
}

bool EmulateInstructionMIPS64::SetTargetTriple(const ArchSpec &arch) {
  return arch.GetTriple().getArch() == llvm::Triple::mips64 ||
         arch.GetTriple().getArch() == llvm::Triple::mips64el;
}

const char *EmulateInstructionMIPS64::GetRegisterName(unsigned reg_num,
                                                      bool alternate_name) {
  if (alternate_name) {
    switch (reg_num) {
    case dwarf_sp_mips64:
      return "r29";
    case dwarf_r30_mips64:
      return "r30";
    case dwarf_ra_mips64:
      return "r31";
    case dwarf_f0_mips64:
      return "f0";
    case dwarf_f1_mips64:
      return "f1";
    case dwarf_f2_mips64:
      return "f2";
    case dwarf_f3_mips64:
      return "f3";
    case dwarf_f4_mips64:
      return "f4";
    case dwarf_f5_mips64:
      return "f5";
    case dwarf_f6_mips64:
      return "f6";
    case dwarf_f7_mips64:
      return "f7";
    case dwarf_f8_mips64:
      return "f8";
```
- **EN**: Implements logic around `SetTargetTriple`, `GetTriple`, `GetRegisterName`; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `SetTargetTriple`, `GetTriple`, `GetRegisterName` 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 232-267
```cpp
    case dwarf_f9_mips64:
      return "f9";
    case dwarf_f10_mips64:
      return "f10";
    case dwarf_f11_mips64:
      return "f11";
    case dwarf_f12_mips64:
      return "f12";
    case dwarf_f13_mips64:
      return "f13";
    case dwarf_f14_mips64:
      return "f14";
    case dwarf_f15_mips64:
      return "f15";
    case dwarf_f16_mips64:
      return "f16";
    case dwarf_f17_mips64:
      return "f17";
    case dwarf_f18_mips64:
      return "f18";
    case dwarf_f19_mips64:
      return "f19";
    case dwarf_f20_mips64:
      return "f20";
    case dwarf_f21_mips64:
      return "f21";
    case dwarf_f22_mips64:
      return "f22";
    case dwarf_f23_mips64:
      return "f23";
    case dwarf_f24_mips64:
      return "f24";
    case dwarf_f25_mips64:
      return "f25";
    case dwarf_f26_mips64:
      return "f26";
```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 268-303
```cpp
    case dwarf_f27_mips64:
      return "f27";
    case dwarf_f28_mips64:
      return "f28";
    case dwarf_f29_mips64:
      return "f29";
    case dwarf_f30_mips64:
      return "f30";
    case dwarf_f31_mips64:
      return "f31";
    case dwarf_w0_mips64:
      return "w0";
    case dwarf_w1_mips64:
      return "w1";
    case dwarf_w2_mips64:
      return "w2";
    case dwarf_w3_mips64:
      return "w3";
    case dwarf_w4_mips64:
      return "w4";
    case dwarf_w5_mips64:
      return "w5";
    case dwarf_w6_mips64:
      return "w6";
    case dwarf_w7_mips64:
      return "w7";
    case dwarf_w8_mips64:
      return "w8";
    case dwarf_w9_mips64:
      return "w9";
    case dwarf_w10_mips64:
      return "w10";
    case dwarf_w11_mips64:
      return "w11";
    case dwarf_w12_mips64:
      return "w12";
```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 304-339
```cpp
    case dwarf_w13_mips64:
      return "w13";
    case dwarf_w14_mips64:
      return "w14";
    case dwarf_w15_mips64:
      return "w15";
    case dwarf_w16_mips64:
      return "w16";
    case dwarf_w17_mips64:
      return "w17";
    case dwarf_w18_mips64:
      return "w18";
    case dwarf_w19_mips64:
      return "w19";
    case dwarf_w20_mips64:
      return "w20";
    case dwarf_w21_mips64:
      return "w21";
    case dwarf_w22_mips64:
      return "w22";
    case dwarf_w23_mips64:
      return "w23";
    case dwarf_w24_mips64:
      return "w24";
    case dwarf_w25_mips64:
      return "w25";
    case dwarf_w26_mips64:
      return "w26";
    case dwarf_w27_mips64:
      return "w27";
    case dwarf_w28_mips64:
      return "w28";
    case dwarf_w29_mips64:
      return "w29";
    case dwarf_w30_mips64:
      return "w30";
```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 340-375
```cpp
    case dwarf_w31_mips64:
      return "w31";
    case dwarf_mir_mips64:
      return "mir";
    case dwarf_mcsr_mips64:
      return "mcsr";
    case dwarf_config5_mips64:
      return "config5";
    default:
      break;
    }
    return nullptr;
  }

  switch (reg_num) {
  case dwarf_zero_mips64:
    return "r0";
  case dwarf_r1_mips64:
    return "r1";
  case dwarf_r2_mips64:
    return "r2";
  case dwarf_r3_mips64:
    return "r3";
  case dwarf_r4_mips64:
    return "r4";
  case dwarf_r5_mips64:
    return "r5";
  case dwarf_r6_mips64:
    return "r6";
  case dwarf_r7_mips64:
    return "r7";
  case dwarf_r8_mips64:
    return "r8";
  case dwarf_r9_mips64:
    return "r9";
  case dwarf_r10_mips64:
```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 376-411
```cpp
    return "r10";
  case dwarf_r11_mips64:
    return "r11";
  case dwarf_r12_mips64:
    return "r12";
  case dwarf_r13_mips64:
    return "r13";
  case dwarf_r14_mips64:
    return "r14";
  case dwarf_r15_mips64:
    return "r15";
  case dwarf_r16_mips64:
    return "r16";
  case dwarf_r17_mips64:
    return "r17";
  case dwarf_r18_mips64:
    return "r18";
  case dwarf_r19_mips64:
    return "r19";
  case dwarf_r20_mips64:
    return "r20";
  case dwarf_r21_mips64:
    return "r21";
  case dwarf_r22_mips64:
    return "r22";
  case dwarf_r23_mips64:
    return "r23";
  case dwarf_r24_mips64:
    return "r24";
  case dwarf_r25_mips64:
    return "r25";
  case dwarf_r26_mips64:
    return "r26";
  case dwarf_r27_mips64:
    return "r27";
  case dwarf_gp_mips64:
```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 412-447
```cpp
    return "gp";
  case dwarf_sp_mips64:
    return "sp";
  case dwarf_r30_mips64:
    return "fp";
  case dwarf_ra_mips64:
    return "ra";
  case dwarf_sr_mips64:
    return "sr";
  case dwarf_lo_mips64:
    return "lo";
  case dwarf_hi_mips64:
    return "hi";
  case dwarf_bad_mips64:
    return "bad";
  case dwarf_cause_mips64:
    return "cause";
  case dwarf_pc_mips64:
    return "pc";
  case dwarf_f0_mips64:
    return "f0";
  case dwarf_f1_mips64:
    return "f1";
  case dwarf_f2_mips64:
    return "f2";
  case dwarf_f3_mips64:
    return "f3";
  case dwarf_f4_mips64:
    return "f4";
  case dwarf_f5_mips64:
    return "f5";
  case dwarf_f6_mips64:
    return "f6";
  case dwarf_f7_mips64:
    return "f7";
  case dwarf_f8_mips64:
```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 448-483
```cpp
    return "f8";
  case dwarf_f9_mips64:
    return "f9";
  case dwarf_f10_mips64:
    return "f10";
  case dwarf_f11_mips64:
    return "f11";
  case dwarf_f12_mips64:
    return "f12";
  case dwarf_f13_mips64:
    return "f13";
  case dwarf_f14_mips64:
    return "f14";
  case dwarf_f15_mips64:
    return "f15";
  case dwarf_f16_mips64:
    return "f16";
  case dwarf_f17_mips64:
    return "f17";
  case dwarf_f18_mips64:
    return "f18";
  case dwarf_f19_mips64:
    return "f19";
  case dwarf_f20_mips64:
    return "f20";
  case dwarf_f21_mips64:
    return "f21";
  case dwarf_f22_mips64:
    return "f22";
  case dwarf_f23_mips64:
    return "f23";
  case dwarf_f24_mips64:
    return "f24";
  case dwarf_f25_mips64:
    return "f25";
  case dwarf_f26_mips64:
```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 484-519
```cpp
    return "f26";
  case dwarf_f27_mips64:
    return "f27";
  case dwarf_f28_mips64:
    return "f28";
  case dwarf_f29_mips64:
    return "f29";
  case dwarf_f30_mips64:
    return "f30";
  case dwarf_f31_mips64:
    return "f31";
  case dwarf_fcsr_mips64:
    return "fcsr";
  case dwarf_fir_mips64:
    return "fir";
  case dwarf_w0_mips64:
    return "w0";
  case dwarf_w1_mips64:
    return "w1";
  case dwarf_w2_mips64:
    return "w2";
  case dwarf_w3_mips64:
    return "w3";
  case dwarf_w4_mips64:
    return "w4";
  case dwarf_w5_mips64:
    return "w5";
  case dwarf_w6_mips64:
    return "w6";
  case dwarf_w7_mips64:
    return "w7";
  case dwarf_w8_mips64:
    return "w8";
  case dwarf_w9_mips64:
    return "w9";
  case dwarf_w10_mips64:
```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 520-555
```cpp
    return "w10";
  case dwarf_w11_mips64:
    return "w11";
  case dwarf_w12_mips64:
    return "w12";
  case dwarf_w13_mips64:
    return "w13";
  case dwarf_w14_mips64:
    return "w14";
  case dwarf_w15_mips64:
    return "w15";
  case dwarf_w16_mips64:
    return "w16";
  case dwarf_w17_mips64:
    return "w17";
  case dwarf_w18_mips64:
    return "w18";
  case dwarf_w19_mips64:
    return "w19";
  case dwarf_w20_mips64:
    return "w20";
  case dwarf_w21_mips64:
    return "w21";
  case dwarf_w22_mips64:
    return "w22";
  case dwarf_w23_mips64:
    return "w23";
  case dwarf_w24_mips64:
    return "w24";
  case dwarf_w25_mips64:
    return "w25";
  case dwarf_w26_mips64:
    return "w26";
  case dwarf_w27_mips64:
    return "w27";
  case dwarf_w28_mips64:
```
- **EN**: Finalizes a local computation, invariant check, or helper decision.
- **CN**: 完成局部计算、不变式检查或辅助决策。

### Lines 556-591
```cpp
    return "w28";
  case dwarf_w29_mips64:
    return "w29";
  case dwarf_w30_mips64:
    return "w30";
  case dwarf_w31_mips64:
    return "w31";
  case dwarf_mcsr_mips64:
    return "mcsr";
  case dwarf_mir_mips64:
    return "mir";
  case dwarf_config5_mips64:
    return "config5";
  }
  return nullptr;
}

std::optional<RegisterInfo>
EmulateInstructionMIPS64::GetRegisterInfo(RegisterKind reg_kind,
                                          uint32_t reg_num) {
  if (reg_kind == eRegisterKindGeneric) {
    switch (reg_num) {
    case LLDB_REGNUM_GENERIC_PC:
      reg_kind = eRegisterKindDWARF;
      reg_num = dwarf_pc_mips64;
      break;
    case LLDB_REGNUM_GENERIC_SP:
      reg_kind = eRegisterKindDWARF;
      reg_num = dwarf_sp_mips64;
      break;
    case LLDB_REGNUM_GENERIC_FP:
      reg_kind = eRegisterKindDWARF;
      reg_num = dwarf_r30_mips64;
      break;
    case LLDB_REGNUM_GENERIC_RA:
      reg_kind = eRegisterKindDWARF;
```
- **EN**: Implements logic around `GetRegisterInfo`.
- **CN**: 围绕 `GetRegisterInfo` 实现具体逻辑。

### Lines 592-627
```cpp
      reg_num = dwarf_ra_mips64;
      break;
    case LLDB_REGNUM_GENERIC_FLAGS:
      reg_kind = eRegisterKindDWARF;
      reg_num = dwarf_sr_mips64;
      break;
    default:
      return {};
    }
  }

  if (reg_kind == eRegisterKindDWARF) {
    RegisterInfo reg_info;
    ::memset(&reg_info, 0, sizeof(RegisterInfo));
    ::memset(reg_info.kinds, LLDB_INVALID_REGNUM, sizeof(reg_info.kinds));

    if (reg_num == dwarf_sr_mips64 || reg_num == dwarf_fcsr_mips64 ||
        reg_num == dwarf_fir_mips64 || reg_num == dwarf_mcsr_mips64 ||
        reg_num == dwarf_mir_mips64 || reg_num == dwarf_config5_mips64) {
      reg_info.byte_size = 4;
      reg_info.format = eFormatHex;
      reg_info.encoding = eEncodingUint;
    } else if ((int)reg_num >= dwarf_zero_mips64 &&
               (int)reg_num <= dwarf_f31_mips64) {
      reg_info.byte_size = 8;
      reg_info.format = eFormatHex;
      reg_info.encoding = eEncodingUint;
    } else if ((int)reg_num >= dwarf_w0_mips64 &&
               (int)reg_num <= dwarf_w31_mips64) {
      reg_info.byte_size = 16;
      reg_info.format = eFormatVectorOfUInt8;
      reg_info.encoding = eEncodingVector;
    } else {
      return {};
    }

```
- **EN**: Implements logic around `memset`; this block controls debugger-side formatting or synthetic presentation of values.
- **CN**: 围绕 `memset` 实现具体逻辑；该代码块控制调试器侧的值格式化或 synthetic 展示。

### Lines 628-655
```cpp
    reg_info.name = GetRegisterName(reg_num, false);
    reg_info.alt_name = GetRegisterName(reg_num, true);
    reg_info.kinds[eRegisterKindDWARF] = reg_num;

    switch (reg_num) {
    case dwarf_r30_mips64:
      reg_info.kinds[eRegisterKindGeneric] = LLDB_REGNUM_GENERIC_FP;
      break;
    case dwarf_ra_mips64:
      reg_info.kinds[eRegisterKindGeneric] = LLDB_REGNUM_GENERIC_RA;
      break;
    case dwarf_sp_mips64:
      reg_info.kinds[eRegisterKindGeneric] = LLDB_REGNUM_GENERIC_SP;
      break;
    case dwarf_pc_mips64:
      reg_info.kinds[eRegisterKindGeneric] = LLDB_REGNUM_GENERIC_PC;
      break;
    case dwarf_sr_mips64:
      reg_info.kinds[eRegisterKindGeneric] = LLDB_REGNUM_GENERIC_FLAGS;
      break;
    default:
      break;
    }
    return reg_info;
  }
  return {};
}

```
- **EN**: Implements logic around `GetRegisterName`.
- **CN**: 围绕 `GetRegisterName` 实现具体逻辑。

### Lines 656-675
```cpp
EmulateInstructionMIPS64::MipsOpcode *
EmulateInstructionMIPS64::GetOpcodeForInstruction(llvm::StringRef op_name) {
  static EmulateInstructionMIPS64::MipsOpcode g_opcodes[] = {
      // Prologue/Epilogue instructions
      {"DADDiu", &EmulateInstructionMIPS64::Emulate_DADDiu,
       "DADDIU rt, rs, immediate"},
      {"ADDiu", &EmulateInstructionMIPS64::Emulate_DADDiu,
       "ADDIU  rt, rs, immediate"},
      {"SD", &EmulateInstructionMIPS64::Emulate_SD, "SD     rt, offset(rs)"},
      {"LD", &EmulateInstructionMIPS64::Emulate_LD, "LD     rt, offset(base)"},
      {"DSUBU", &EmulateInstructionMIPS64::Emulate_DSUBU_DADDU,
       "DSUBU  rd, rs, rt"},
      {"SUBU", &EmulateInstructionMIPS64::Emulate_DSUBU_DADDU,
       "SUBU   rd, rs, rt"},
      {"DADDU", &EmulateInstructionMIPS64::Emulate_DSUBU_DADDU,
       "DADDU  rd, rs, rt"},
      {"ADDU", &EmulateInstructionMIPS64::Emulate_DSUBU_DADDU,
       "ADDU   rd, rs, rt"},
      {"LUI", &EmulateInstructionMIPS64::Emulate_LUI, "LUI    rt, immediate"},

```
- **EN**: Implements logic around `GetOpcodeForInstruction`, `offset`.
- **CN**: 围绕 `GetOpcodeForInstruction`, `offset` 实现具体逻辑。

### Lines 676-711
```cpp
      // Load/Store  instructions
      /* Following list of emulated instructions are required by implementation
         of hardware watchpoint
         for MIPS in lldb. As we just need the address accessed by instructions,
         we have generalised
         all these instructions in 2 functions depending on their addressing
         modes */

      {"LB", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "LB    rt, offset(base)"},
      {"LBE", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "LBE   rt, offset(base)"},
      {"LBU", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "LBU   rt, offset(base)"},
      {"LBUE", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "LBUE  rt, offset(base)"},
      {"LDC1", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "LDC1  ft, offset(base)"},
      {"LDL", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "LDL   rt, offset(base)"},
      {"LDR", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "LDR   rt, offset(base)"},
      {"LLD", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "LLD   rt, offset(base)"},
      {"LDC2", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "LDC2  rt, offset(base)"},
      {"LDXC1", &EmulateInstructionMIPS64::Emulate_LDST_Reg,
       "LDXC1 fd, index (base)"},
      {"LH", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "LH    rt, offset(base)"},
      {"LHE", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "LHE   rt, offset(base)"},
      {"LHU", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "LHU   rt, offset(base)"},
      {"LHUE", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "LHUE  rt, offset(base)"},
```
- **EN**: Implements logic around `offset`, `index`; this block tracks breakpoint state, stop conditions, or hit-processing policy; maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `offset`, `index` 实现具体逻辑；该代码块跟踪断点状态、停机条件或命中处理策略，并将可执行状态映射回模块、符号、节区或地址。

### Lines 712-736
```cpp
      {"LL", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "LL    rt, offset(base)"},
      {"LLE", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "LLE   rt, offset(base)"},
      {"LUXC1", &EmulateInstructionMIPS64::Emulate_LDST_Reg,
       "LUXC1 fd, index (base)"},
      {"LW", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "LW    rt, offset(rs)"},
      {"LWC1", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "LWC1  ft, offset(base)"},
      {"LWC2", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "LWC2  rt, offset(base)"},
      {"LWE", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "LWE   rt, offset(base)"},
      {"LWL", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "LWL   rt, offset(base)"},
      {"LWLE", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "LWLE  rt, offset(base)"},
      {"LWR", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "LWR   rt, offset(base)"},
      {"LWRE", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "LWRE  rt, offset(base)"},
      {"LWXC1", &EmulateInstructionMIPS64::Emulate_LDST_Reg,
       "LWXC1 fd, index (base)"},

```
- **EN**: Implements logic around `offset`, `index`.
- **CN**: 围绕 `offset`, `index` 实现具体逻辑。

### Lines 737-772
```cpp
      {"SB", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "SB    rt, offset(base)"},
      {"SBE", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "SBE   rt, offset(base)"},
      {"SC", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "SC    rt, offset(base)"},
      {"SCE", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "SCE   rt, offset(base)"},
      {"SCD", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "SCD   rt, offset(base)"},
      {"SDL", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "SDL   rt, offset(base)"},
      {"SDR", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "SDR   rt, offset(base)"},
      {"SDC1", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "SDC1  ft, offset(base)"},
      {"SDC2", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "SDC2  rt, offset(base)"},
      {"SDXC1", &EmulateInstructionMIPS64::Emulate_LDST_Reg,
       "SDXC1 fs, index (base)"},
      {"SH", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "SH    rt, offset(base)"},
      {"SHE", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "SHE   rt, offset(base)"},
      {"SUXC1", &EmulateInstructionMIPS64::Emulate_LDST_Reg,
       "SUXC1 fs, index (base)"},
      {"SW", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "SW    rt, offset(rs)"},
      {"SWC1", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "SWC1  ft, offset(base)"},
      {"SWC2", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "SWC2  rt, offset(base)"},
      {"SWE", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "SWE   rt, offset(base)"},
      {"SWL", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "SWL   rt, offset(base)"},
```
- **EN**: Implements logic around `offset`, `index`.
- **CN**: 围绕 `offset`, `index` 实现具体逻辑。

### Lines 773-808
```cpp
      {"SWLE", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "SWLE  rt, offset(base)"},
      {"SWR", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "SWR   rt, offset(base)"},
      {"SWRE", &EmulateInstructionMIPS64::Emulate_LDST_Imm,
       "SWRE  rt, offset(base)"},
      {"SWXC1", &EmulateInstructionMIPS64::Emulate_LDST_Reg,
       "SWXC1 fs, index (base)"},

      // Branch instructions
      {"BEQ", &EmulateInstructionMIPS64::Emulate_BXX_3ops, "BEQ rs,rt,offset"},
      {"BEQ64", &EmulateInstructionMIPS64::Emulate_BXX_3ops, "BEQ rs,rt,offset"},
      {"BNE", &EmulateInstructionMIPS64::Emulate_BXX_3ops, "BNE rs,rt,offset"},
      {"BNE64", &EmulateInstructionMIPS64::Emulate_BXX_3ops, "BNE rs,rt,offset"},
      {"BEQL", &EmulateInstructionMIPS64::Emulate_BXX_3ops,
       "BEQL rs,rt,offset"},
      {"BNEL", &EmulateInstructionMIPS64::Emulate_BXX_3ops,
       "BNEL rs,rt,offset"},
      {"BGEZALL", &EmulateInstructionMIPS64::Emulate_Bcond_Link,
       "BGEZALL rt,offset"},
      {"BAL", &EmulateInstructionMIPS64::Emulate_BAL, "BAL offset"},
      {"BGEZAL", &EmulateInstructionMIPS64::Emulate_Bcond_Link,
       "BGEZAL rs,offset"},
      {"BALC", &EmulateInstructionMIPS64::Emulate_BALC, "BALC offset"},
      {"BC", &EmulateInstructionMIPS64::Emulate_BC, "BC offset"},
      {"BGEZ", &EmulateInstructionMIPS64::Emulate_BXX_2ops, "BGEZ rs,offset"},
      {"BGEZ64", &EmulateInstructionMIPS64::Emulate_BXX_2ops, "BGEZ rs,offset"},
      {"BLEZALC", &EmulateInstructionMIPS64::Emulate_Bcond_Link_C,
       "BLEZALC rs,offset"},
      {"BGEZALC", &EmulateInstructionMIPS64::Emulate_Bcond_Link_C,
       "BGEZALC rs,offset"},
      {"BLTZALC", &EmulateInstructionMIPS64::Emulate_Bcond_Link_C,
       "BLTZALC rs,offset"},
      {"BGTZALC", &EmulateInstructionMIPS64::Emulate_Bcond_Link_C,
       "BGTZALC rs,offset"},
      {"BEQZALC", &EmulateInstructionMIPS64::Emulate_Bcond_Link_C,
```
- **EN**: Implements logic around `offset`, `index`.
- **CN**: 围绕 `offset`, `index` 实现具体逻辑。

### Lines 809-844
```cpp
       "BEQZALC rs,offset"},
      {"BNEZALC", &EmulateInstructionMIPS64::Emulate_Bcond_Link_C,
       "BNEZALC rs,offset"},
      {"BEQC", &EmulateInstructionMIPS64::Emulate_BXX_3ops_C,
       "BEQC rs,rt,offset"},
      {"BEQC64", &EmulateInstructionMIPS64::Emulate_BXX_3ops_C,
       "BEQC rs,rt,offset"},
      {"BNEC", &EmulateInstructionMIPS64::Emulate_BXX_3ops_C,
       "BNEC rs,rt,offset"},
      {"BNEC64", &EmulateInstructionMIPS64::Emulate_BXX_3ops_C,
       "BNEC rs,rt,offset"},
      {"BLTC", &EmulateInstructionMIPS64::Emulate_BXX_3ops_C,
       "BLTC rs,rt,offset"},
      {"BLTC64", &EmulateInstructionMIPS64::Emulate_BXX_3ops_C,
       "BLTC rs,rt,offset"},
      {"BGEC", &EmulateInstructionMIPS64::Emulate_BXX_3ops_C,
       "BGEC rs,rt,offset"},
      {"BGEC64", &EmulateInstructionMIPS64::Emulate_BXX_3ops_C,
       "BGEC rs,rt,offset"},
      {"BLTUC", &EmulateInstructionMIPS64::Emulate_BXX_3ops_C,
       "BLTUC rs,rt,offset"},
      {"BLTUC64", &EmulateInstructionMIPS64::Emulate_BXX_3ops_C,
       "BLTUC rs,rt,offset"},
      {"BGEUC", &EmulateInstructionMIPS64::Emulate_BXX_3ops_C,
       "BGEUC rs,rt,offset"},
      {"BGEUC64", &EmulateInstructionMIPS64::Emulate_BXX_3ops_C,
       "BGEUC rs,rt,offset"},
      {"BLTZC", &EmulateInstructionMIPS64::Emulate_BXX_2ops_C,
       "BLTZC rt,offset"},
      {"BLTZC64", &EmulateInstructionMIPS64::Emulate_BXX_2ops_C,
       "BLTZC rt,offset"},
      {"BLEZC", &EmulateInstructionMIPS64::Emulate_BXX_2ops_C,
       "BLEZC rt,offset"},
      {"BLEZC64", &EmulateInstructionMIPS64::Emulate_BXX_2ops_C,
       "BLEZC rt,offset"},
      {"BGEZC", &EmulateInstructionMIPS64::Emulate_BXX_2ops_C,
```
- **EN**: Contains supporting implementation details for the surrounding LLDB subsystem.
- **CN**: 包含周边 LLDB 子系统所需的辅助实现细节。

### Lines 845-880
```cpp
       "BGEZC rt,offset"},
      {"BGEZC64", &EmulateInstructionMIPS64::Emulate_BXX_2ops_C,
       "BGEZC rt,offset"},
      {"BGTZC", &EmulateInstructionMIPS64::Emulate_BXX_2ops_C,
       "BGTZC rt,offset"},
      {"BGTZC64", &EmulateInstructionMIPS64::Emulate_BXX_2ops_C,
       "BGTZC rt,offset"},
      {"BEQZC", &EmulateInstructionMIPS64::Emulate_BXX_2ops_C,
       "BEQZC rt,offset"},
      {"BEQZC64", &EmulateInstructionMIPS64::Emulate_BXX_2ops_C,
       "BEQZC rt,offset"},
      {"BNEZC", &EmulateInstructionMIPS64::Emulate_BXX_2ops_C,
       "BNEZC rt,offset"},
      {"BNEZC64", &EmulateInstructionMIPS64::Emulate_BXX_2ops_C,
       "BNEZC rt,offset"},
      {"BGEZL", &EmulateInstructionMIPS64::Emulate_BXX_2ops, "BGEZL rt,offset"},
      {"BGTZ", &EmulateInstructionMIPS64::Emulate_BXX_2ops, "BGTZ rt,offset"},
      {"BGTZ64", &EmulateInstructionMIPS64::Emulate_BXX_2ops, "BGTZ rt,offset"},
      {"BGTZL", &EmulateInstructionMIPS64::Emulate_BXX_2ops, "BGTZL rt,offset"},
      {"BLEZ", &EmulateInstructionMIPS64::Emulate_BXX_2ops, "BLEZ rt,offset"},
      {"BLEZ64", &EmulateInstructionMIPS64::Emulate_BXX_2ops, "BLEZ rt,offset"},
      {"BLEZL", &EmulateInstructionMIPS64::Emulate_BXX_2ops, "BLEZL rt,offset"},
      {"BLTZ", &EmulateInstructionMIPS64::Emulate_BXX_2ops, "BLTZ rt,offset"},
      {"BLTZ64", &EmulateInstructionMIPS64::Emulate_BXX_2ops, "BLTZ rt,offset"},
      {"BLTZAL", &EmulateInstructionMIPS64::Emulate_Bcond_Link,
       "BLTZAL rt,offset"},
      {"BLTZALL", &EmulateInstructionMIPS64::Emulate_Bcond_Link,
       "BLTZALL rt,offset"},
      {"BLTZL", &EmulateInstructionMIPS64::Emulate_BXX_2ops, "BLTZL rt,offset"},
      {"BOVC", &EmulateInstructionMIPS64::Emulate_BXX_3ops_C,
       "BOVC rs,rt,offset"},
      {"BNVC", &EmulateInstructionMIPS64::Emulate_BXX_3ops_C,
       "BNVC rs,rt,offset"},
      {"J", &EmulateInstructionMIPS64::Emulate_J, "J target"},
      {"JAL", &EmulateInstructionMIPS64::Emulate_JAL, "JAL target"},
      {"JALX", &EmulateInstructionMIPS64::Emulate_JAL, "JALX target"},
```
- **EN**: Contains supporting implementation details for the surrounding LLDB subsystem.
- **CN**: 包含周边 LLDB 子系统所需的辅助实现细节。

### Lines 881-916
```cpp
      {"JALR", &EmulateInstructionMIPS64::Emulate_JALR, "JALR target"},
      {"JALR64", &EmulateInstructionMIPS64::Emulate_JALR, "JALR target"},
      {"JALR_HB", &EmulateInstructionMIPS64::Emulate_JALR, "JALR.HB target"},
      {"JIALC", &EmulateInstructionMIPS64::Emulate_JIALC, "JIALC rt,offset"},
      {"JIALC64", &EmulateInstructionMIPS64::Emulate_JIALC, "JIALC rt,offset"},
      {"JIC", &EmulateInstructionMIPS64::Emulate_JIC, "JIC rt,offset"},
      {"JIC64", &EmulateInstructionMIPS64::Emulate_JIC, "JIC rt,offset"},
      {"JR", &EmulateInstructionMIPS64::Emulate_JR, "JR target"},
      {"JR64", &EmulateInstructionMIPS64::Emulate_JR, "JR target"},
      {"JR_HB", &EmulateInstructionMIPS64::Emulate_JR, "JR.HB target"},
      {"BC1F", &EmulateInstructionMIPS64::Emulate_FP_branch, "BC1F cc, offset"},
      {"BC1T", &EmulateInstructionMIPS64::Emulate_FP_branch, "BC1T cc, offset"},
      {"BC1FL", &EmulateInstructionMIPS64::Emulate_FP_branch,
       "BC1FL cc, offset"},
      {"BC1TL", &EmulateInstructionMIPS64::Emulate_FP_branch,
       "BC1TL cc, offset"},
      {"BC1EQZ", &EmulateInstructionMIPS64::Emulate_BC1EQZ,
       "BC1EQZ ft, offset"},
      {"BC1NEZ", &EmulateInstructionMIPS64::Emulate_BC1NEZ,
       "BC1NEZ ft, offset"},
      {"BC1ANY2F", &EmulateInstructionMIPS64::Emulate_3D_branch,
       "BC1ANY2F cc, offset"},
      {"BC1ANY2T", &EmulateInstructionMIPS64::Emulate_3D_branch,
       "BC1ANY2T cc, offset"},
      {"BC1ANY4F", &EmulateInstructionMIPS64::Emulate_3D_branch,
       "BC1ANY4F cc, offset"},
      {"BC1ANY4T", &EmulateInstructionMIPS64::Emulate_3D_branch,
       "BC1ANY4T cc, offset"},
      {"BNZ_B", &EmulateInstructionMIPS64::Emulate_BNZB, "BNZ.b wt,s16"},
      {"BNZ_H", &EmulateInstructionMIPS64::Emulate_BNZH, "BNZ.h wt,s16"},
      {"BNZ_W", &EmulateInstructionMIPS64::Emulate_BNZW, "BNZ.w wt,s16"},
      {"BNZ_D", &EmulateInstructionMIPS64::Emulate_BNZD, "BNZ.d wt,s16"},
      {"BZ_B", &EmulateInstructionMIPS64::Emulate_BZB, "BZ.b wt,s16"},
      {"BZ_H", &EmulateInstructionMIPS64::Emulate_BZH, "BZ.h wt,s16"},
      {"BZ_W", &EmulateInstructionMIPS64::Emulate_BZW, "BZ.w wt,s16"},
      {"BZ_D", &EmulateInstructionMIPS64::Emulate_BZD, "BZ.d wt,s16"},
```
- **EN**: Contains supporting implementation details for the surrounding LLDB subsystem.
- **CN**: 包含周边 LLDB 子系统所需的辅助实现细节。

### Lines 917-944
```cpp
      {"BNZ_V", &EmulateInstructionMIPS64::Emulate_BNZV, "BNZ.V wt,s16"},
      {"BZ_V", &EmulateInstructionMIPS64::Emulate_BZV, "BZ.V wt,s16"},
  };

  for (MipsOpcode &opcode : g_opcodes) {
    if (op_name.equals_insensitive(opcode.op_name))
      return &opcode;
  }
  return nullptr;
}

bool EmulateInstructionMIPS64::ReadInstruction() {
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
- **EN**: Implements logic around `equals_insensitive`, `ReadInstruction`, `ReadRegisterUnsigned`, `SetNoArgs`, and 3 more symbols.
- **CN**: 围绕 `equals_insensitive`, `ReadInstruction`, `ReadRegisterUnsigned`, `SetNoArgs`, and 3 more symbols 实现具体逻辑。

### Lines 945-967
```cpp
bool EmulateInstructionMIPS64::EvaluateInstruction(uint32_t evaluate_options) {
  bool success = false;
  llvm::MCInst mc_insn;
  uint64_t insn_size;
  DataExtractor data;

  /* Keep the complexity of the decode logic with the llvm::MCDisassembler
   * class. */
  if (m_opcode.GetData(data)) {
    llvm::MCDisassembler::DecodeStatus decode_status;
    llvm::ArrayRef<uint8_t> raw_insn(data.GetDataStart(), data.GetByteSize());
    decode_status = m_disasm->getInstruction(mc_insn, insn_size, raw_insn,
                                             m_addr, llvm::nulls());
    if (decode_status != llvm::MCDisassembler::Success)
      return false;
  }

  /*
   * mc_insn.getOpcode() returns decoded opcode. However to make use
   * of llvm::Mips::<insn> we would need "MipsGenInstrInfo.inc".
  */
  llvm::StringRef op_name = m_insn_info->getName(mc_insn.getOpcode());

```
- **EN**: Implements logic around `EvaluateInstruction`, `GetData`, `raw_insn`, `getInstruction`, and 2 more symbols.
- **CN**: 围绕 `EvaluateInstruction`, `GetData`, `raw_insn`, `getInstruction`, and 2 more symbols 实现具体逻辑。

### Lines 968-987
```cpp
  /*
   * Decoding has been done already. Just get the call-back function
   * and emulate the instruction.
  */
  MipsOpcode *opcode_data = GetOpcodeForInstruction(op_name);

  if (opcode_data == nullptr)
    return false;

  uint64_t old_pc = 0, new_pc = 0;
  const bool auto_advance_pc =
      evaluate_options & eEmulateInstructionOptionAutoAdvancePC;

  if (auto_advance_pc) {
    old_pc =
        ReadRegisterUnsigned(eRegisterKindDWARF, dwarf_pc_mips64, 0, &success);
    if (!success)
      return false;
  }

```
- **EN**: Implements logic around `GetOpcodeForInstruction`, `ReadRegisterUnsigned`; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `GetOpcodeForInstruction`, `ReadRegisterUnsigned` 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 988-1008
```cpp
  /* emulate instruction */
  success = (this->*opcode_data->callback)(mc_insn);
  if (!success)
    return false;

  if (auto_advance_pc) {
    new_pc =
        ReadRegisterUnsigned(eRegisterKindDWARF, dwarf_pc_mips64, 0, &success);
    if (!success)
      return false;

    /* If we haven't changed the PC, change it here */
    if (old_pc == new_pc) {
      new_pc += 4;
      Context context;
      if (!WriteRegisterUnsigned(context, eRegisterKindDWARF, dwarf_pc_mips64,
                                 new_pc))
        return false;
    }
  }

```
- **EN**: Implements logic around `ReadRegisterUnsigned`, `WriteRegisterUnsigned`; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `ReadRegisterUnsigned`, `WriteRegisterUnsigned` 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 1009-1026
```cpp
  return true;
}

bool EmulateInstructionMIPS64::CreateFunctionEntryUnwind(
    UnwindPlan &unwind_plan) {
  unwind_plan.Clear();
  unwind_plan.SetRegisterKind(eRegisterKindDWARF);

  UnwindPlan::Row row;
  const bool can_replace = false;

  // Our previous Call Frame Address is the stack pointer
  row.GetCFAValue().SetIsRegisterPlusOffset(dwarf_sp_mips64, 0);

  // Our previous PC is in the RA
  row.SetRegisterLocationToRegister(dwarf_pc_mips64, dwarf_ra_mips64,
                                    can_replace);

```
- **EN**: Implements logic around `CreateFunctionEntryUnwind`, `Clear`, `SetRegisterKind`, `GetCFAValue`, and 1 more symbols; this block maps executable state back to modules, symbols, sections, or addresses; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `CreateFunctionEntryUnwind`, `Clear`, `SetRegisterKind`, `GetCFAValue`, and 1 more symbols 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 1027-1059
```cpp
  unwind_plan.AppendRow(std::move(row));

  // All other registers are the same.
  unwind_plan.SetSourceName("EmulateInstructionMIPS64");
  unwind_plan.SetSourcedFromCompiler(eLazyBoolNo);
  unwind_plan.SetUnwindPlanValidAtAllInstructions(eLazyBoolYes);
  unwind_plan.SetUnwindPlanForSignalTrap(eLazyBoolNo);
  unwind_plan.SetReturnAddressRegister(dwarf_ra_mips64);

  return true;
}

bool EmulateInstructionMIPS64::nonvolatile_reg_p(uint64_t regnum) {
  switch (regnum) {
  case dwarf_r16_mips64:
  case dwarf_r17_mips64:
  case dwarf_r18_mips64:
  case dwarf_r19_mips64:
  case dwarf_r20_mips64:
  case dwarf_r21_mips64:
  case dwarf_r22_mips64:
  case dwarf_r23_mips64:
  case dwarf_gp_mips64:
  case dwarf_sp_mips64:
  case dwarf_r30_mips64:
  case dwarf_ra_mips64:
    return true;
  default:
    return false;
  }
  return false;
}

```
- **EN**: Implements logic around `AppendRow`, `SetSourceName`, `SetSourcedFromCompiler`, `SetUnwindPlanValidAtAllInstructions`, and 3 more symbols.
- **CN**: 围绕 `AppendRow`, `SetSourceName`, `SetSourcedFromCompiler`, `SetUnwindPlanValidAtAllInstructions`, and 3 more symbols 实现具体逻辑。

### Lines 1060-1078
```cpp
bool EmulateInstructionMIPS64::Emulate_DADDiu(llvm::MCInst &insn) {
  // DADDIU rt, rs, immediate
  // GPR[rt] <- GPR[rs] + sign_extend(immediate)

  uint8_t dst, src;
  bool success = false;
  const uint32_t imm16 = insn.getOperand(2).getImm();
  int64_t imm = SignedBits(imm16, 15, 0);

  dst = m_reg_info->getEncodingValue(insn.getOperand(0).getReg());
  src = m_reg_info->getEncodingValue(insn.getOperand(1).getReg());

  // If immediate is greater than 2^16 - 1 then clang generate LUI,
  // (D)ADDIU,(D)SUBU instructions in prolog. Example lui    $1, 0x2 daddiu $1,
  // $1, -0x5920 dsubu  $sp, $sp, $1 In this case, (D)ADDIU dst and src will be
  // same and not equal to sp
  if (dst == src) {
    Context context;

```
- **EN**: Implements logic around `Emulate_DADDiu`, `getOperand`, `SignedBits`, `getEncodingValue`; this block supports expression parsing, wrapping, or debug-time code generation.
- **CN**: 围绕 `Emulate_DADDiu`, `getOperand`, `SignedBits`, `getEncodingValue` 实现具体逻辑；该代码块支持表达式解析、包装或调试期代码生成。

### Lines 1079-1103
```cpp
    /* read <src> register */
    const uint64_t src_opd_val = ReadRegisterUnsigned(
        eRegisterKindDWARF, dwarf_zero_mips64 + src, 0, &success);
    if (!success)
      return false;

    /* Check if this is daddiu sp, sp, imm16 */
    if (dst == dwarf_sp_mips64) {
      /*
       * From the MIPS IV spec:
       *
       * The term “unsigned” in the instruction name is a misnomer; this
       * operation is 64-bit modulo arithmetic that does not trap on overflow.
       * It is appropriate for arithmetic which is not signed, such as address
       * arithmetic, or integer arithmetic environments that ignore overflow,
       * such as “C” language arithmetic.
       *
       * Assume 2's complement and rely on unsigned overflow here.
       */
      uint64_t result = src_opd_val + imm;
      std::optional<RegisterInfo> reg_info_sp =
          GetRegisterInfo(eRegisterKindDWARF, dwarf_sp_mips64);
      if (reg_info_sp)
        context.SetRegisterPlusOffset(*reg_info_sp, imm);

```
- **EN**: Implements logic around `ReadRegisterUnsigned`, `GetRegisterInfo`, `SetRegisterPlusOffset`; this block maps executable state back to modules, symbols, sections, or addresses; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `ReadRegisterUnsigned`, `GetRegisterInfo`, `SetRegisterPlusOffset` 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 1104-1123
```cpp
      /* We are allocating bytes on stack */
      context.type = eContextAdjustStackPointer;

      WriteRegisterUnsigned(context, eRegisterKindDWARF, dwarf_sp_mips64,
                            result);
      return true;
    }

    imm += src_opd_val;
    context.SetImmediateSigned(imm);
    context.type = eContextImmediate;

    if (!WriteRegisterUnsigned(context, eRegisterKindDWARF,
                               dwarf_zero_mips64 + dst, imm))
      return false;
  }

  return true;
}

```
- **EN**: Implements logic around `WriteRegisterUnsigned`, `SetImmediateSigned`; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `WriteRegisterUnsigned`, `SetImmediateSigned` 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 1124-1141
```cpp
bool EmulateInstructionMIPS64::Emulate_SD(llvm::MCInst &insn) {
  uint64_t address;
  bool success = false;
  uint32_t imm16 = insn.getOperand(2).getImm();
  uint64_t imm = SignedBits(imm16, 15, 0);
  uint32_t src, base;
  Context bad_vaddr_context;

  src = m_reg_info->getEncodingValue(insn.getOperand(0).getReg());
  base = m_reg_info->getEncodingValue(insn.getOperand(1).getReg());

  std::optional<RegisterInfo> reg_info_base =
      GetRegisterInfo(eRegisterKindDWARF, dwarf_zero_mips64 + base);
  std::optional<RegisterInfo> reg_info_src =
      GetRegisterInfo(eRegisterKindDWARF, dwarf_zero_mips64 + src);
  if (!reg_info_base || !reg_info_src)
    return false;

```
- **EN**: Implements logic around `Emulate_SD`, `getOperand`, `SignedBits`, `getEncodingValue`, and 1 more symbols; this block maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `Emulate_SD`, `getOperand`, `SignedBits`, `getEncodingValue`, and 1 more symbols 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址。

### Lines 1142-1160
```cpp
  /* read SP */
  address = ReadRegisterUnsigned(eRegisterKindDWARF, dwarf_zero_mips64 + base,
                                 0, &success);
  if (!success)
    return false;

  /* destination address */
  address = address + imm;

  /* We look for sp based non-volatile register stores */
  if (nonvolatile_reg_p(src)) {
    Context context;
    context.type = eContextPushRegisterOnStack;
    context.SetRegisterToRegisterPlusOffset(*reg_info_src, *reg_info_base, 0);

    std::optional<RegisterValue> data_src = ReadRegister(*reg_info_base);
    if (!data_src)
      return false;

```
- **EN**: Implements logic around `ReadRegisterUnsigned`, `nonvolatile_reg_p`, `SetRegisterToRegisterPlusOffset`, `ReadRegister`; this block maps executable state back to modules, symbols, sections, or addresses; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `ReadRegisterUnsigned`, `nonvolatile_reg_p`, `SetRegisterToRegisterPlusOffset`, `ReadRegister` 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 1161-1179
```cpp
    Status error;
    RegisterValue::BytesContainer buffer(reg_info_src->byte_size);
    if (data_src->GetAsMemoryData(*reg_info_src, buffer.data(),
                                  reg_info_src->byte_size, eByteOrderLittle,
                                  error) == 0)
      return false;

    if (!WriteMemory(context, address, buffer.data(), reg_info_src->byte_size))
      return false;
  }

  /* Set the bad_vaddr register with base address used in the instruction */
  bad_vaddr_context.type = eContextInvalid;
  WriteRegisterUnsigned(bad_vaddr_context, eRegisterKindDWARF, dwarf_bad_mips64,
                        address);

  return true;
}

```
- **EN**: Implements logic around `buffer`, `GetAsMemoryData`, `WriteMemory`, `WriteRegisterUnsigned`; this block propagates recoverable errors, status objects, or diagnostics; maps executable state back to modules, symbols, sections, or addresses; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `buffer`, `GetAsMemoryData`, `WriteMemory`, `WriteRegisterUnsigned` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并将可执行状态映射回模块、符号、节区或地址，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 1180-1198
```cpp
bool EmulateInstructionMIPS64::Emulate_LD(llvm::MCInst &insn) {
  bool success = false;
  uint32_t src, base;
  int64_t imm, address;
  Context bad_vaddr_context;

  src = m_reg_info->getEncodingValue(insn.getOperand(0).getReg());
  base = m_reg_info->getEncodingValue(insn.getOperand(1).getReg());
  imm = insn.getOperand(2).getImm();

  if (!GetRegisterInfo(eRegisterKindDWARF, dwarf_zero_mips64 + base))
    return false;

  /* read base register */
  address = ReadRegisterUnsigned(eRegisterKindDWARF, dwarf_zero_mips64 + base,
                                 0, &success);
  if (!success)
    return false;

```
- **EN**: Implements logic around `Emulate_LD`, `getEncodingValue`, `getOperand`, `GetRegisterInfo`, and 1 more symbols; this block maps executable state back to modules, symbols, sections, or addresses; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `Emulate_LD`, `getEncodingValue`, `getOperand`, `GetRegisterInfo`, and 1 more symbols 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 1199-1216
```cpp
  /* destination address */
  address = address + imm;

  /* Set the bad_vaddr register with base address used in the instruction */
  bad_vaddr_context.type = eContextInvalid;
  WriteRegisterUnsigned(bad_vaddr_context, eRegisterKindDWARF, dwarf_bad_mips64,
                        address);

  if (nonvolatile_reg_p(src)) {
    RegisterValue data_src;
    std::optional<RegisterInfo> reg_info_src =
        GetRegisterInfo(eRegisterKindDWARF, dwarf_zero_mips64 + src);
    if (!reg_info_src)
      return false;

    Context context;
    context.type = eContextRegisterLoad;

```
- **EN**: Implements logic around `WriteRegisterUnsigned`, `nonvolatile_reg_p`, `GetRegisterInfo`; this block maps executable state back to modules, symbols, sections, or addresses; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `WriteRegisterUnsigned`, `nonvolatile_reg_p`, `GetRegisterInfo` 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 1217-1235
```cpp
    return WriteRegister(context, *reg_info_src, data_src);
  }

  return false;
}

bool EmulateInstructionMIPS64::Emulate_LUI(llvm::MCInst &insn) {
  // LUI rt, immediate
  // GPR[rt] <- sign_extend(immediate << 16)

  const uint32_t imm32 = insn.getOperand(1).getImm() << 16;
  int64_t imm = SignedBits(imm32, 31, 0);
  uint8_t rt;
  Context context;

  rt = m_reg_info->getEncodingValue(insn.getOperand(0).getReg());
  context.SetImmediateSigned(imm);
  context.type = eContextImmediate;

```
- **EN**: Implements logic around `WriteRegister`, `Emulate_LUI`, `getOperand`, `SignedBits`, and 2 more symbols.
- **CN**: 围绕 `WriteRegister`, `Emulate_LUI`, `getOperand`, `SignedBits`, and 2 more symbols 实现具体逻辑。

### Lines 1236-1256
```cpp
  return WriteRegisterUnsigned(context, eRegisterKindDWARF,
                               dwarf_zero_mips64 + rt, imm);
}

bool EmulateInstructionMIPS64::Emulate_DSUBU_DADDU(llvm::MCInst &insn) {
  // DSUBU sp, <src>, <rt>
  // DADDU sp, <src>, <rt>
  // DADDU dst, sp, <rt>

  bool success = false;
  uint64_t result;
  uint8_t src, dst, rt;
  llvm::StringRef op_name = m_insn_info->getName(insn.getOpcode());

  dst = m_reg_info->getEncodingValue(insn.getOperand(0).getReg());
  src = m_reg_info->getEncodingValue(insn.getOperand(1).getReg());

  /* Check if sp is destination register */
  if (dst == dwarf_sp_mips64) {
    rt = m_reg_info->getEncodingValue(insn.getOperand(2).getReg());

```
- **EN**: Implements logic around `WriteRegisterUnsigned`, `Emulate_DSUBU_DADDU`, `getName`, `getEncodingValue`; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `WriteRegisterUnsigned`, `Emulate_DSUBU_DADDU`, `getName`, `getEncodingValue` 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 1257-1274
```cpp
    /* read <src> register */
    uint64_t src_opd_val = ReadRegisterUnsigned(
        eRegisterKindDWARF, dwarf_zero_mips64 + src, 0, &success);
    if (!success)
      return false;

    /* read <rt > register */
    uint64_t rt_opd_val = ReadRegisterUnsigned(
        eRegisterKindDWARF, dwarf_zero_mips64 + rt, 0, &success);
    if (!success)
      return false;

    if (op_name.equals_insensitive("DSUBU") ||
        op_name.equals_insensitive("SUBU"))
      result = src_opd_val - rt_opd_val;
    else
      result = src_opd_val + rt_opd_val;

```
- **EN**: Implements logic around `ReadRegisterUnsigned`, `equals_insensitive`; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `ReadRegisterUnsigned`, `equals_insensitive` 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 1275-1295
```cpp
    Context context;
    std::optional<RegisterInfo> reg_info_sp =
        GetRegisterInfo(eRegisterKindDWARF, dwarf_sp_mips64);
    if (reg_info_sp)
      context.SetRegisterPlusOffset(*reg_info_sp, rt_opd_val);

    /* We are allocating bytes on stack */
    context.type = eContextAdjustStackPointer;

    WriteRegisterUnsigned(context, eRegisterKindDWARF, dwarf_sp_mips64, result);

    return true;
  } else if (src == dwarf_sp_mips64) {
    rt = m_reg_info->getEncodingValue(insn.getOperand(2).getReg());

    /* read <src> register */
    uint64_t src_opd_val = ReadRegisterUnsigned(
        eRegisterKindDWARF, dwarf_zero_mips64 + src, 0, &success);
    if (!success)
      return false;

```
- **EN**: Implements logic around `GetRegisterInfo`, `SetRegisterPlusOffset`, `WriteRegisterUnsigned`, `getEncodingValue`, and 1 more symbols; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `GetRegisterInfo`, `SetRegisterPlusOffset`, `WriteRegisterUnsigned`, `getEncodingValue`, and 1 more symbols 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 1296-1317
```cpp
    /* read <rt> register */
    uint64_t rt_opd_val = ReadRegisterUnsigned(
        eRegisterKindDWARF, dwarf_zero_mips64 + rt, 0, &success);
    if (!success)
      return false;

    Context context;

    if (op_name.equals_insensitive("DSUBU") ||
        op_name.equals_insensitive("SUBU"))
      result = src_opd_val - rt_opd_val;
    else
      result = src_opd_val + rt_opd_val;

    context.SetImmediateSigned(result);
    context.type = eContextImmediate;

    if (!WriteRegisterUnsigned(context, eRegisterKindDWARF,
                               dwarf_zero_mips64 + dst, result))
      return false;
  }

```
- **EN**: Implements logic around `ReadRegisterUnsigned`, `equals_insensitive`, `SetImmediateSigned`, `WriteRegisterUnsigned`; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `ReadRegisterUnsigned`, `equals_insensitive`, `SetImmediateSigned`, `WriteRegisterUnsigned` 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 1318-1335
```cpp
  return true;
}

/*
    Emulate below MIPS branch instructions.
    BEQ, BNE : Branch on condition
    BEQL, BNEL : Branch likely
*/
bool EmulateInstructionMIPS64::Emulate_BXX_3ops(llvm::MCInst &insn) {
  bool success = false;
  uint32_t rs, rt;
  int64_t offset, pc, rs_val, rt_val, target = 0;
  llvm::StringRef op_name = m_insn_info->getName(insn.getOpcode());

  rs = m_reg_info->getEncodingValue(insn.getOperand(0).getReg());
  rt = m_reg_info->getEncodingValue(insn.getOperand(1).getReg());
  offset = insn.getOperand(2).getImm();

```
- **EN**: Implements logic around `Emulate_BXX_3ops`, `getName`, `getEncodingValue`, `getOperand`; this block tracks breakpoint state, stop conditions, or hit-processing policy; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `Emulate_BXX_3ops`, `getName`, `getEncodingValue`, `getOperand` 实现具体逻辑；该代码块跟踪断点状态、停机条件或命中处理策略，并协调调试器运行时对象、事件或异步控制流。

### Lines 1336-1364
```cpp
  pc = ReadRegisterUnsigned(eRegisterKindDWARF, dwarf_pc_mips64, 0, &success);
  if (!success)
    return false;

  rs_val = (int64_t)ReadRegisterUnsigned(eRegisterKindDWARF,
                                         dwarf_zero_mips64 + rs, 0, &success);
  if (!success)
    return false;

  rt_val = (int64_t)ReadRegisterUnsigned(eRegisterKindDWARF,
                                         dwarf_zero_mips64 + rt, 0, &success);
  if (!success)
    return false;

  if (op_name.equals_insensitive("BEQ") || op_name.equals_insensitive("BEQL") ||
      op_name.equals_insensitive("BEQ64")) {
    if (rs_val == rt_val)
      target = pc + offset;
    else
      target = pc + 8;
  } else if (op_name.equals_insensitive("BNE") ||
             op_name.equals_insensitive("BNEL") ||
             op_name.equals_insensitive("BNE64")) {
    if (rs_val != rt_val)
      target = pc + offset;
    else
      target = pc + 8;
  }

```
- **EN**: Implements logic around `ReadRegisterUnsigned`, `equals_insensitive`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `ReadRegisterUnsigned`, `equals_insensitive` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 1365-1384
```cpp
  Context context;
  context.type = eContextRelativeBranchImmediate;
  context.SetImmediate(offset);

  return WriteRegisterUnsigned(context, eRegisterKindDWARF, dwarf_pc_mips64,
                               target);
}

/*
    Emulate below MIPS Non-Compact conditional branch and link instructions.
    BLTZAL, BGEZAL      :
    BLTZALL, BGEZALL    : Branch likely
*/
bool EmulateInstructionMIPS64::Emulate_Bcond_Link(llvm::MCInst &insn) {
  bool success = false;
  uint32_t rs;
  int64_t offset, pc, target = 0;
  int64_t rs_val;
  llvm::StringRef op_name = m_insn_info->getName(insn.getOpcode());

```
- **EN**: Implements logic around `SetImmediate`, `WriteRegisterUnsigned`, `Emulate_Bcond_Link`, `getName`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `SetImmediate`, `WriteRegisterUnsigned`, `Emulate_Bcond_Link`, `getName` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 1385-1410
```cpp
  rs = m_reg_info->getEncodingValue(insn.getOperand(0).getReg());
  offset = insn.getOperand(1).getImm();

  pc = ReadRegisterUnsigned(eRegisterKindDWARF, dwarf_pc_mips64, 0, &success);
  if (!success)
    return false;

  rs_val = (int64_t)ReadRegisterUnsigned(eRegisterKindDWARF,
                                         dwarf_zero_mips64 + rs, 0, &success);
  if (!success)
    return false;

  if (op_name.equals_insensitive("BLTZAL") ||
      op_name.equals_insensitive("BLTZALL")) {
    if (rs_val < 0)
      target = pc + offset;
    else
      target = pc + 8;
  } else if (op_name.equals_insensitive("BGEZAL") ||
             op_name.equals_insensitive("BGEZALL")) {
    if (rs_val >= 0)
      target = pc + offset;
    else
      target = pc + 8;
  }

```
- **EN**: Implements logic around `getEncodingValue`, `getOperand`, `ReadRegisterUnsigned`, `equals_insensitive`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `getEncodingValue`, `getOperand`, `ReadRegisterUnsigned`, `equals_insensitive` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 1411-1435
```cpp
  Context context;

  if (!WriteRegisterUnsigned(context, eRegisterKindDWARF, dwarf_pc_mips64,
                             target))
    return false;

  if (!WriteRegisterUnsigned(context, eRegisterKindDWARF, dwarf_ra_mips64,
                             pc + 8))
    return false;

  return true;
}

bool EmulateInstructionMIPS64::Emulate_BAL(llvm::MCInst &insn) {
  bool success = false;
  int64_t offset, pc, target;

  /*
   * BAL offset
   *      offset = sign_ext (offset << 2)
   *      RA = PC + 8
   *      PC = PC + offset
  */
  offset = insn.getOperand(0).getImm();

```
- **EN**: Implements logic around `WriteRegisterUnsigned`, `Emulate_BAL`, `getOperand`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `WriteRegisterUnsigned`, `Emulate_BAL`, `getOperand` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 1436-1454
```cpp
  pc = ReadRegisterUnsigned(eRegisterKindDWARF, dwarf_pc_mips64, 0, &success);
  if (!success)
    return false;

  target = pc + offset;

  Context context;

  if (!WriteRegisterUnsigned(context, eRegisterKindDWARF, dwarf_pc_mips64,
                             target))
    return false;

  if (!WriteRegisterUnsigned(context, eRegisterKindDWARF, dwarf_ra_mips64,
                             pc + 8))
    return false;

  return true;
}

```
- **EN**: Implements logic around `ReadRegisterUnsigned`, `WriteRegisterUnsigned`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `ReadRegisterUnsigned`, `WriteRegisterUnsigned` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 1455-1472
```cpp
bool EmulateInstructionMIPS64::Emulate_BALC(llvm::MCInst &insn) {
  bool success = false;
  int64_t offset, pc, target;

  /*
   * BALC offset
   *      offset = sign_ext (offset << 2)
   *      RA = PC + 4
   *      PC = PC + 4 + offset
  */
  offset = insn.getOperand(0).getImm();

  pc = ReadRegisterUnsigned(eRegisterKindDWARF, dwarf_pc_mips64, 0, &success);
  if (!success)
    return false;

  target = pc + offset;

```
- **EN**: Implements logic around `Emulate_BALC`, `getOperand`, `ReadRegisterUnsigned`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `Emulate_BALC`, `getOperand`, `ReadRegisterUnsigned` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 1473-1495
```cpp
  Context context;

  if (!WriteRegisterUnsigned(context, eRegisterKindDWARF, dwarf_pc_mips64,
                             target))
    return false;

  if (!WriteRegisterUnsigned(context, eRegisterKindDWARF, dwarf_ra_mips64,
                             pc + 4))
    return false;

  return true;
}

/*
    Emulate below MIPS conditional branch and link instructions.
    BLEZALC, BGEZALC, BLTZALC, BGTZALC, BEQZALC, BNEZALC : Compact branches
*/
bool EmulateInstructionMIPS64::Emulate_Bcond_Link_C(llvm::MCInst &insn) {
  bool success = false;
  uint32_t rs;
  int64_t offset, pc, rs_val, target = 0;
  llvm::StringRef op_name = m_insn_info->getName(insn.getOpcode());

```
- **EN**: Implements logic around `WriteRegisterUnsigned`, `Emulate_Bcond_Link_C`, `getName`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `WriteRegisterUnsigned`, `Emulate_Bcond_Link_C`, `getName` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 1496-1531
```cpp
  rs = m_reg_info->getEncodingValue(insn.getOperand(0).getReg());
  offset = insn.getOperand(1).getImm();

  pc = ReadRegisterUnsigned(eRegisterKindDWARF, dwarf_pc_mips64, 0, &success);
  if (!success)
    return false;

  rs_val = (int64_t)ReadRegisterUnsigned(eRegisterKindDWARF,
                                         dwarf_zero_mips64 + rs, 0, &success);
  if (!success)
    return false;

  if (op_name.equals_insensitive("BLEZALC")) {
    if (rs_val <= 0)
      target = pc + offset;
    else
      target = pc + 4;
  } else if (op_name.equals_insensitive("BGEZALC")) {
    if (rs_val >= 0)
      target = pc + offset;
    else
      target = pc + 4;
  } else if (op_name.equals_insensitive("BLTZALC")) {
    if (rs_val < 0)
      target = pc + offset;
    else
      target = pc + 4;
  } else if (op_name.equals_insensitive("BGTZALC")) {
    if (rs_val > 0)
      target = pc + offset;
    else
      target = pc + 4;
  } else if (op_name.equals_insensitive("BEQZALC")) {
    if (rs_val == 0)
      target = pc + offset;
    else
```
- **EN**: Implements logic around `getEncodingValue`, `getOperand`, `ReadRegisterUnsigned`, `equals_insensitive`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `getEncodingValue`, `getOperand`, `ReadRegisterUnsigned`, `equals_insensitive` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 1532-1549
```cpp
      target = pc + 4;
  } else if (op_name.equals_insensitive("BNEZALC")) {
    if (rs_val != 0)
      target = pc + offset;
    else
      target = pc + 4;
  }

  Context context;

  if (!WriteRegisterUnsigned(context, eRegisterKindDWARF, dwarf_pc_mips64,
                             target))
    return false;

  if (!WriteRegisterUnsigned(context, eRegisterKindDWARF, dwarf_ra_mips64,
                             pc + 4))
    return false;

```
- **EN**: Implements logic around `equals_insensitive`, `WriteRegisterUnsigned`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `equals_insensitive`, `WriteRegisterUnsigned` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 1550-1570
```cpp
  return true;
}

/*
    Emulate below MIPS branch instructions.
    BLTZL, BGEZL, BGTZL, BLEZL : Branch likely
    BLTZ, BGEZ, BGTZ, BLEZ     : Non-compact branches
*/
bool EmulateInstructionMIPS64::Emulate_BXX_2ops(llvm::MCInst &insn) {
  bool success = false;
  uint32_t rs;
  int64_t offset, pc, rs_val, target = 0;
  llvm::StringRef op_name = m_insn_info->getName(insn.getOpcode());

  rs = m_reg_info->getEncodingValue(insn.getOperand(0).getReg());
  offset = insn.getOperand(1).getImm();

  pc = ReadRegisterUnsigned(eRegisterKindDWARF, dwarf_pc_mips64, 0, &success);
  if (!success)
    return false;

```
- **EN**: Implements logic around `Emulate_BXX_2ops`, `getName`, `getEncodingValue`, `getOperand`, and 1 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `Emulate_BXX_2ops`, `getName`, `getEncodingValue`, `getOperand`, and 1 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 1571-1605
```cpp
  rs_val = (int64_t)ReadRegisterUnsigned(eRegisterKindDWARF,
                                         dwarf_zero_mips64 + rs, 0, &success);
  if (!success)
    return false;

  if (op_name.equals_insensitive("BLTZL") ||
      op_name.equals_insensitive("BLTZ") ||
      op_name.equals_insensitive("BLTZ64")) {
    if (rs_val < 0)
      target = pc + offset;
    else
      target = pc + 8;
  } else if (op_name.equals_insensitive("BGEZL") ||
             op_name.equals_insensitive("BGEZ") ||
             op_name.equals_insensitive("BGEZ64")) {
    if (rs_val >= 0)
      target = pc + offset;
    else
      target = pc + 8;
  } else if (op_name.equals_insensitive("BGTZL") ||
             op_name.equals_insensitive("BGTZ") ||
             op_name.equals_insensitive("BGTZ64")) {
    if (rs_val > 0)
      target = pc + offset;
    else
      target = pc + 8;
  } else if (op_name.equals_insensitive("BLEZL") ||
             op_name.equals_insensitive("BLEZ") ||
             op_name.equals_insensitive("BLEZ64")) {
    if (rs_val <= 0)
      target = pc + offset;
    else
      target = pc + 8;
  }

```
- **EN**: Implements logic around `ReadRegisterUnsigned`, `equals_insensitive`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `ReadRegisterUnsigned`, `equals_insensitive` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 1606-1624
```cpp
  Context context;
  context.type = eContextRelativeBranchImmediate;
  context.SetImmediate(offset);

  return WriteRegisterUnsigned(context, eRegisterKindDWARF, dwarf_pc_mips64,
                               target);
}

bool EmulateInstructionMIPS64::Emulate_BC(llvm::MCInst &insn) {
  bool success = false;
  int64_t offset, pc, target;

  /*
   * BC offset
   *      offset = sign_ext (offset << 2)
   *      PC = PC + 4 + offset
  */
  offset = insn.getOperand(0).getImm();

```
- **EN**: Implements logic around `SetImmediate`, `WriteRegisterUnsigned`, `Emulate_BC`, `getOperand`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `SetImmediate`, `WriteRegisterUnsigned`, `Emulate_BC`, `getOperand` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 1625-1653
```cpp
  pc = ReadRegisterUnsigned(eRegisterKindDWARF, dwarf_pc_mips64, 0, &success);
  if (!success)
    return false;

  target = pc + offset;

  Context context;

  return WriteRegisterUnsigned(context, eRegisterKindDWARF, dwarf_pc_mips64,
                               target);
}

static int IsAdd64bitOverflow(int64_t a, int64_t b) {
  int64_t r = (uint64_t)a + (uint64_t)b;
  return (a < 0 && b < 0 && r >= 0) || (a >= 0 && b >= 0 && r < 0);
}

/*
    Emulate below MIPS branch instructions.
    BEQC, BNEC, BLTC, BGEC, BLTUC, BGEUC, BOVC, BNVC: Compact branch
   instructions with no delay slot
*/
bool EmulateInstructionMIPS64::Emulate_BXX_3ops_C(llvm::MCInst &insn) {
  bool success = false;
  uint32_t rs, rt;
  int64_t offset, pc, rs_val, rt_val, target = 0;
  llvm::StringRef op_name = m_insn_info->getName(insn.getOpcode());
  uint32_t current_inst_size = m_insn_info->get(insn.getOpcode()).getSize();

```
- **EN**: Implements logic around `ReadRegisterUnsigned`, `WriteRegisterUnsigned`, `IsAdd64bitOverflow`, `Emulate_BXX_3ops_C`, and 2 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `ReadRegisterUnsigned`, `WriteRegisterUnsigned`, `IsAdd64bitOverflow`, `Emulate_BXX_3ops_C`, and 2 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 1654-1671
```cpp
  rs = m_reg_info->getEncodingValue(insn.getOperand(0).getReg());
  rt = m_reg_info->getEncodingValue(insn.getOperand(1).getReg());
  offset = insn.getOperand(2).getImm();

  pc = ReadRegisterUnsigned(eRegisterKindDWARF, dwarf_pc_mips64, 0, &success);
  if (!success)
    return false;

  rs_val = (int64_t)ReadRegisterUnsigned(eRegisterKindDWARF,
                                         dwarf_zero_mips64 + rs, 0, &success);
  if (!success)
    return false;

  rt_val = (int64_t)ReadRegisterUnsigned(eRegisterKindDWARF,
                                         dwarf_zero_mips64 + rt, 0, &success);
  if (!success)
    return false;

```
- **EN**: Implements logic around `getEncodingValue`, `getOperand`, `ReadRegisterUnsigned`.
- **CN**: 围绕 `getEncodingValue`, `getOperand`, `ReadRegisterUnsigned` 实现具体逻辑。

### Lines 1672-1707
```cpp
  if (op_name.equals_insensitive("BEQC") ||
      op_name.equals_insensitive("BEQC64")) {
    if (rs_val == rt_val)
      target = pc + offset;
    else
      target = pc + 4;
  } else if (op_name.equals_insensitive("BNEC") ||
             op_name.equals_insensitive("BNEC64")) {
    if (rs_val != rt_val)
      target = pc + offset;
    else
      target = pc + 4;
  } else if (op_name.equals_insensitive("BLTC") ||
             op_name.equals_insensitive("BLTC64")) {
    if (rs_val < rt_val)
      target = pc + offset;
    else
      target = pc + 4;
  } else if (op_name.equals_insensitive("BGEC64") ||
             op_name.equals_insensitive("BGEC")) {
    if (rs_val >= rt_val)
      target = pc + offset;
    else
      target = pc + 4;
  } else if (op_name.equals_insensitive("BLTUC") ||
             op_name.equals_insensitive("BLTUC64")) {
    if (rs_val < rt_val)
      target = pc + offset;
    else
      target = pc + 4;
  } else if (op_name.equals_insensitive("BGEUC") ||
             op_name.equals_insensitive("BGEUC64")) {
    if ((uint32_t)rs_val >= (uint32_t)rt_val)
      target = pc + offset;
    else
      target = pc + 4;
```
- **EN**: Implements logic around `equals_insensitive`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `equals_insensitive` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 1708-1727
```cpp
  } else if (op_name.equals_insensitive("BOVC")) {
    if (IsAdd64bitOverflow(rs_val, rt_val))
      target = pc + offset;
    else
      target = pc + 4;
  } else if (op_name.equals_insensitive("BNVC")) {
    if (!IsAdd64bitOverflow(rs_val, rt_val))
      target = pc + offset;
    else
      target = pc + 4;
  }

  Context context;
  context.type = eContextRelativeBranchImmediate;
  context.SetImmediate(current_inst_size + offset);

  return WriteRegisterUnsigned(context, eRegisterKindDWARF, dwarf_pc_mips64,
                               target);
}

```
- **EN**: Implements logic around `equals_insensitive`, `IsAdd64bitOverflow`, `SetImmediate`, `WriteRegisterUnsigned`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `equals_insensitive`, `IsAdd64bitOverflow`, `SetImmediate`, `WriteRegisterUnsigned` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 1728-1746
```cpp
/*
    Emulate below MIPS branch instructions.
    BLTZC, BLEZC, BGEZC, BGTZC, BEQZC, BNEZC : Compact Branches
*/
bool EmulateInstructionMIPS64::Emulate_BXX_2ops_C(llvm::MCInst &insn) {
  bool success = false;
  uint32_t rs;
  int64_t offset, pc, target = 0;
  int64_t rs_val;
  llvm::StringRef op_name = m_insn_info->getName(insn.getOpcode());
  uint32_t current_inst_size = m_insn_info->get(insn.getOpcode()).getSize();

  rs = m_reg_info->getEncodingValue(insn.getOperand(0).getReg());
  offset = insn.getOperand(1).getImm();

  pc = ReadRegisterUnsigned(eRegisterKindDWARF, dwarf_pc_mips64, 0, &success);
  if (!success)
    return false;

```
- **EN**: Implements logic around `Emulate_BXX_2ops_C`, `getName`, `get`, `getEncodingValue`, and 2 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `Emulate_BXX_2ops_C`, `getName`, `get`, `getEncodingValue`, and 2 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 1747-1782
```cpp
  rs_val = (int64_t)ReadRegisterUnsigned(eRegisterKindDWARF,
                                         dwarf_zero_mips64 + rs, 0, &success);
  if (!success)
    return false;

  if (op_name.equals_insensitive("BLTZC") ||
      op_name.equals_insensitive("BLTZC64")) {
    if (rs_val < 0)
      target = pc + offset;
    else
      target = pc + 4;
  } else if (op_name.equals_insensitive("BLEZC") ||
             op_name.equals_insensitive("BLEZC64")) {
    if (rs_val <= 0)
      target = pc + offset;
    else
      target = pc + 4;
  } else if (op_name.equals_insensitive("BGEZC") ||
             op_name.equals_insensitive("BGEZC64")) {
    if (rs_val >= 0)
      target = pc + offset;
    else
      target = pc + 4;
  } else if (op_name.equals_insensitive("BGTZC") ||
             op_name.equals_insensitive("BGTZC64")) {
    if (rs_val > 0)
      target = pc + offset;
    else
      target = pc + 4;
  } else if (op_name.equals_insensitive("BEQZC") ||
             op_name.equals_insensitive("BEQZC64")) {
    if (rs_val == 0)
      target = pc + offset;
    else
      target = pc + 4;
  } else if (op_name.equals_insensitive("BNEZC") ||
```
- **EN**: Implements logic around `ReadRegisterUnsigned`, `equals_insensitive`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `ReadRegisterUnsigned`, `equals_insensitive` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 1783-1801
```cpp
             op_name.equals_insensitive("BNEZC64")) {
    if (rs_val != 0)
      target = pc + offset;
    else
      target = pc + 4;
  }

  Context context;
  context.type = eContextRelativeBranchImmediate;
  context.SetImmediate(current_inst_size + offset);

  return WriteRegisterUnsigned(context, eRegisterKindDWARF, dwarf_pc_mips64,
                               target);
}

bool EmulateInstructionMIPS64::Emulate_J(llvm::MCInst &insn) {
  bool success = false;
  uint64_t offset, pc;

```
- **EN**: Implements logic around `equals_insensitive`, `SetImmediate`, `WriteRegisterUnsigned`, `Emulate_J`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `equals_insensitive`, `SetImmediate`, `WriteRegisterUnsigned`, `Emulate_J` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 1802-1821
```cpp
  /*
   * J offset
   *      offset = sign_ext (offset << 2)
   *      PC = PC[63-28] | offset
  */
  offset = insn.getOperand(0).getImm();

  pc = ReadRegisterUnsigned(eRegisterKindDWARF, dwarf_pc_mips64, 0, &success);
  if (!success)
    return false;

  /* This is a PC-region branch and not PC-relative */
  pc = (pc & 0xFFFFFFFFF0000000ULL) | offset;

  Context context;

  return WriteRegisterUnsigned(context, eRegisterKindDWARF, dwarf_pc_mips64,
                               pc);
}

```
- **EN**: Implements logic around `getOperand`, `ReadRegisterUnsigned`, `WriteRegisterUnsigned`.
- **CN**: 围绕 `getOperand`, `ReadRegisterUnsigned`, `WriteRegisterUnsigned` 实现具体逻辑。

### Lines 1822-1839
```cpp
bool EmulateInstructionMIPS64::Emulate_JAL(llvm::MCInst &insn) {
  bool success = false;
  uint64_t offset, target, pc;

  /*
   * JAL offset
   *      offset = sign_ext (offset << 2)
   *      PC = PC[63-28] | offset
  */
  offset = insn.getOperand(0).getImm();

  pc = ReadRegisterUnsigned(eRegisterKindDWARF, dwarf_pc_mips64, 0, &success);
  if (!success)
    return false;

  /* This is a PC-region branch and not PC-relative */
  target = (pc & 0xFFFFFFFFF0000000ULL) | offset;

```
- **EN**: Implements logic around `Emulate_JAL`, `getOperand`, `ReadRegisterUnsigned`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `Emulate_JAL`, `getOperand`, `ReadRegisterUnsigned` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 1840-1857
```cpp
  Context context;

  if (!WriteRegisterUnsigned(context, eRegisterKindDWARF, dwarf_pc_mips64,
                             target))
    return false;

  if (!WriteRegisterUnsigned(context, eRegisterKindDWARF, dwarf_ra_mips64,
                             pc + 8))
    return false;

  return true;
}

bool EmulateInstructionMIPS64::Emulate_JALR(llvm::MCInst &insn) {
  bool success = false;
  uint32_t rs, rt;
  uint64_t pc, rs_val;

```
- **EN**: Implements logic around `WriteRegisterUnsigned`, `Emulate_JALR`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `WriteRegisterUnsigned`, `Emulate_JALR` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 1858-1876
```cpp
  /*
   * JALR rt, rs
   *      GPR[rt] = PC + 8
   *      PC = GPR[rs]
  */
  rt = m_reg_info->getEncodingValue(insn.getOperand(0).getReg());
  rs = m_reg_info->getEncodingValue(insn.getOperand(1).getReg());

  pc = ReadRegisterUnsigned(eRegisterKindDWARF, dwarf_pc_mips64, 0, &success);
  if (!success)
    return false;

  rs_val = ReadRegisterUnsigned(eRegisterKindDWARF, dwarf_zero_mips64 + rs, 0,
                                &success);
  if (!success)
    return false;

  Context context;

```
- **EN**: Implements logic around `getEncodingValue`, `ReadRegisterUnsigned`.
- **CN**: 围绕 `getEncodingValue`, `ReadRegisterUnsigned` 实现具体逻辑。

### Lines 1877-1901
```cpp
  if (!WriteRegisterUnsigned(context, eRegisterKindDWARF, dwarf_pc_mips64,
                             rs_val))
    return false;

  if (!WriteRegisterUnsigned(context, eRegisterKindDWARF,
                             dwarf_zero_mips64 + rt, pc + 8))
    return false;

  return true;
}

bool EmulateInstructionMIPS64::Emulate_JIALC(llvm::MCInst &insn) {
  bool success = false;
  uint32_t rt;
  int64_t target, offset, pc, rt_val;

  /*
   * JIALC rt, offset
   *      offset = sign_ext (offset)
   *      PC = GPR[rt] + offset
   *      RA = PC + 4
  */
  rt = m_reg_info->getEncodingValue(insn.getOperand(0).getReg());
  offset = insn.getOperand(1).getImm();

```
- **EN**: Implements logic around `WriteRegisterUnsigned`, `Emulate_JIALC`, `getEncodingValue`, `getOperand`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `WriteRegisterUnsigned`, `Emulate_JIALC`, `getEncodingValue`, `getOperand` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 1902-1922
```cpp
  pc = ReadRegisterUnsigned(eRegisterKindDWARF, dwarf_pc_mips64, 0, &success);
  if (!success)
    return false;

  rt_val = (int64_t)ReadRegisterUnsigned(eRegisterKindDWARF,
                                         dwarf_zero_mips64 + rt, 0, &success);
  if (!success)
    return false;

  target = rt_val + offset;

  Context context;

  if (!WriteRegisterUnsigned(context, eRegisterKindDWARF, dwarf_pc_mips64,
                             target))
    return false;

  if (!WriteRegisterUnsigned(context, eRegisterKindDWARF, dwarf_ra_mips64,
                             pc + 4))
    return false;

```
- **EN**: Implements logic around `ReadRegisterUnsigned`, `WriteRegisterUnsigned`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `ReadRegisterUnsigned`, `WriteRegisterUnsigned` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 1923-1943
```cpp
  return true;
}

bool EmulateInstructionMIPS64::Emulate_JIC(llvm::MCInst &insn) {
  bool success = false;
  uint32_t rt;
  int64_t target, offset, rt_val;

  /*
   * JIC rt, offset
   *      offset = sign_ext (offset)
   *      PC = GPR[rt] + offset
  */
  rt = m_reg_info->getEncodingValue(insn.getOperand(0).getReg());
  offset = insn.getOperand(1).getImm();

  rt_val = (int64_t)ReadRegisterUnsigned(eRegisterKindDWARF,
                                         dwarf_zero_mips64 + rt, 0, &success);
  if (!success)
    return false;

```
- **EN**: Implements logic around `Emulate_JIC`, `getEncodingValue`, `getOperand`, `ReadRegisterUnsigned`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `Emulate_JIC`, `getEncodingValue`, `getOperand`, `ReadRegisterUnsigned` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 1944-1962
```cpp
  target = rt_val + offset;

  Context context;

  return WriteRegisterUnsigned(context, eRegisterKindDWARF, dwarf_pc_mips64,
                               target);
}

bool EmulateInstructionMIPS64::Emulate_JR(llvm::MCInst &insn) {
  bool success = false;
  uint32_t rs;
  uint64_t rs_val;

  /*
   * JR rs
   *      PC = GPR[rs]
  */
  rs = m_reg_info->getEncodingValue(insn.getOperand(0).getReg());

```
- **EN**: Implements logic around `WriteRegisterUnsigned`, `Emulate_JR`, `getEncodingValue`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `WriteRegisterUnsigned`, `Emulate_JR`, `getEncodingValue` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 1963-1984
```cpp
  rs_val = ReadRegisterUnsigned(eRegisterKindDWARF, dwarf_zero_mips64 + rs, 0,
                                &success);
  if (!success)
    return false;

  Context context;

  return WriteRegisterUnsigned(context, eRegisterKindDWARF, dwarf_pc_mips64,
                               rs_val);
}

/*
    Emulate Branch on FP True/False
    BC1F, BC1FL :   Branch on FP False (L stands for branch likely)
    BC1T, BC1TL :   Branch on FP True  (L stands for branch likely)
*/
bool EmulateInstructionMIPS64::Emulate_FP_branch(llvm::MCInst &insn) {
  bool success = false;
  uint32_t cc, fcsr;
  int64_t pc, offset, target = 0;
  llvm::StringRef op_name = m_insn_info->getName(insn.getOpcode());

```
- **EN**: Implements logic around `ReadRegisterUnsigned`, `WriteRegisterUnsigned`, `False`, `True`, and 2 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `ReadRegisterUnsigned`, `WriteRegisterUnsigned`, `False`, `True`, and 2 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 1985-2003
```cpp
  /*
   * BC1F cc, offset
   *  condition <- (FPConditionCode(cc) == 0)
   *      if condition then
   *          offset = sign_ext (offset)
   *          PC = PC + offset
  */
  cc = m_reg_info->getEncodingValue(insn.getOperand(0).getReg());
  offset = insn.getOperand(1).getImm();

  pc = ReadRegisterUnsigned(eRegisterKindDWARF, dwarf_pc_mips64, 0, &success);
  if (!success)
    return false;

  fcsr =
      ReadRegisterUnsigned(eRegisterKindDWARF, dwarf_fcsr_mips64, 0, &success);
  if (!success)
    return false;

```
- **EN**: Implements logic around `getEncodingValue`, `getOperand`, `ReadRegisterUnsigned`; this block tracks breakpoint state, stop conditions, or hit-processing policy.
- **CN**: 围绕 `getEncodingValue`, `getOperand`, `ReadRegisterUnsigned` 实现具体逻辑；该代码块跟踪断点状态、停机条件或命中处理策略。

### Lines 2004-2022
```cpp
  /* fcsr[23], fcsr[25-31] are vaild condition bits */
  fcsr = ((fcsr >> 24) & 0xfe) | ((fcsr >> 23) & 0x01);

  if (op_name.equals_insensitive("BC1F") ||
      op_name.equals_insensitive("BC1FL")) {
    if ((fcsr & (1 << cc)) == 0)
      target = pc + offset;
    else
      target = pc + 8;
  } else if (op_name.equals_insensitive("BC1T") ||
             op_name.equals_insensitive("BC1TL")) {
    if ((fcsr & (1 << cc)) != 0)
      target = pc + offset;
    else
      target = pc + 8;
  }

  Context context;

```
- **EN**: Implements logic around `equals_insensitive`; this block tracks breakpoint state, stop conditions, or hit-processing policy; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `equals_insensitive` 实现具体逻辑；该代码块跟踪断点状态、停机条件或命中处理策略，并协调调试器运行时对象、事件或异步控制流。

### Lines 2023-2042
```cpp
  return WriteRegisterUnsigned(context, eRegisterKindDWARF, dwarf_pc_mips64,
                               target);
}

bool EmulateInstructionMIPS64::Emulate_BC1EQZ(llvm::MCInst &insn) {
  bool success = false;
  uint32_t ft;
  uint64_t ft_val;
  int64_t target, pc, offset;

  /*
   * BC1EQZ ft, offset
   *  condition <- (FPR[ft].bit0 == 0)
   *      if condition then
   *          offset = sign_ext (offset)
   *          PC = PC + 4 + offset
  */
  ft = m_reg_info->getEncodingValue(insn.getOperand(0).getReg());
  offset = insn.getOperand(1).getImm();

```
- **EN**: Implements logic around `WriteRegisterUnsigned`, `Emulate_BC1EQZ`, `getEncodingValue`, `getOperand`; this block tracks breakpoint state, stop conditions, or hit-processing policy; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `WriteRegisterUnsigned`, `Emulate_BC1EQZ`, `getEncodingValue`, `getOperand` 实现具体逻辑；该代码块跟踪断点状态、停机条件或命中处理策略，并协调调试器运行时对象、事件或异步控制流。

### Lines 2043-2062
```cpp
  pc = ReadRegisterUnsigned(eRegisterKindDWARF, dwarf_pc_mips64, 0, &success);
  if (!success)
    return false;

  ft_val = ReadRegisterUnsigned(eRegisterKindDWARF, dwarf_zero_mips64 + ft, 0,
                                &success);
  if (!success)
    return false;

  if ((ft_val & 1) == 0)
    target = pc + 4 + offset;
  else
    target = pc + 8;

  Context context;

  return WriteRegisterUnsigned(context, eRegisterKindDWARF, dwarf_pc_mips64,
                               target);
}

```
- **EN**: Implements logic around `ReadRegisterUnsigned`, `WriteRegisterUnsigned`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `ReadRegisterUnsigned`, `WriteRegisterUnsigned` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 2063-2082
```cpp
bool EmulateInstructionMIPS64::Emulate_BC1NEZ(llvm::MCInst &insn) {
  bool success = false;
  uint32_t ft;
  uint64_t ft_val;
  int64_t target, pc, offset;

  /*
   * BC1NEZ ft, offset
   *  condition <- (FPR[ft].bit0 != 0)
   *      if condition then
   *          offset = sign_ext (offset)
   *          PC = PC + 4 + offset
  */
  ft = m_reg_info->getEncodingValue(insn.getOperand(0).getReg());
  offset = insn.getOperand(1).getImm();

  pc = ReadRegisterUnsigned(eRegisterKindDWARF, dwarf_pc_mips64, 0, &success);
  if (!success)
    return false;

```
- **EN**: Implements logic around `Emulate_BC1NEZ`, `getEncodingValue`, `getOperand`, `ReadRegisterUnsigned`; this block tracks breakpoint state, stop conditions, or hit-processing policy; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `Emulate_BC1NEZ`, `getEncodingValue`, `getOperand`, `ReadRegisterUnsigned` 实现具体逻辑；该代码块跟踪断点状态、停机条件或命中处理策略，并协调调试器运行时对象、事件或异步控制流。

### Lines 2083-2111
```cpp
  ft_val = ReadRegisterUnsigned(eRegisterKindDWARF, dwarf_zero_mips64 + ft, 0,
                                &success);
  if (!success)
    return false;

  if ((ft_val & 1) != 0)
    target = pc + 4 + offset;
  else
    target = pc + 8;

  Context context;

  return WriteRegisterUnsigned(context, eRegisterKindDWARF, dwarf_pc_mips64,
                               target);
}

/*
    Emulate MIPS-3D Branch instructions
    BC1ANY2F, BC1ANY2T  : Branch on Any of Two Floating Point Condition Codes
   False/True
    BC1ANY4F, BC1ANY4T  : Branch on Any of Four Floating Point Condition Codes
   False/True
*/
bool EmulateInstructionMIPS64::Emulate_3D_branch(llvm::MCInst &insn) {
  bool success = false;
  uint32_t cc, fcsr;
  int64_t pc, offset, target = 0;
  llvm::StringRef op_name = m_insn_info->getName(insn.getOpcode());

```
- **EN**: Implements logic around `ReadRegisterUnsigned`, `WriteRegisterUnsigned`, `Emulate_3D_branch`, `getName`; this block tracks breakpoint state, stop conditions, or hit-processing policy; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `ReadRegisterUnsigned`, `WriteRegisterUnsigned`, `Emulate_3D_branch`, `getName` 实现具体逻辑；该代码块跟踪断点状态、停机条件或命中处理策略，并协调调试器运行时对象、事件或异步控制流。

### Lines 2112-2147
```cpp
  cc = m_reg_info->getEncodingValue(insn.getOperand(0).getReg());
  offset = insn.getOperand(1).getImm();

  pc = ReadRegisterUnsigned(eRegisterKindDWARF, dwarf_pc_mips64, 0, &success);
  if (!success)
    return false;

  fcsr = (uint32_t)ReadRegisterUnsigned(eRegisterKindDWARF, dwarf_fcsr_mips64,
                                        0, &success);
  if (!success)
    return false;

  /* fcsr[23], fcsr[25-31] are vaild condition bits */
  fcsr = ((fcsr >> 24) & 0xfe) | ((fcsr >> 23) & 0x01);

  if (op_name.equals_insensitive("BC1ANY2F")) {
    /* if any one bit is 0 */
    if (((fcsr >> cc) & 3) != 3)
      target = pc + offset;
    else
      target = pc + 8;
  } else if (op_name.equals_insensitive("BC1ANY2T")) {
    /* if any one bit is 1 */
    if (((fcsr >> cc) & 3) != 0)
      target = pc + offset;
    else
      target = pc + 8;
  } else if (op_name.equals_insensitive("BC1ANY4F")) {
    /* if any one bit is 0 */
    if (((fcsr >> cc) & 0xf) != 0xf)
      target = pc + offset;
    else
      target = pc + 8;
  } else if (op_name.equals_insensitive("BC1ANY4T")) {
    /* if any one bit is 1 */
    if (((fcsr >> cc) & 0xf) != 0)
```
- **EN**: Implements logic around `getEncodingValue`, `getOperand`, `ReadRegisterUnsigned`, `equals_insensitive`; this block tracks breakpoint state, stop conditions, or hit-processing policy; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `getEncodingValue`, `getOperand`, `ReadRegisterUnsigned`, `equals_insensitive` 实现具体逻辑；该代码块跟踪断点状态、停机条件或命中处理策略，并协调调试器运行时对象、事件或异步控制流。

### Lines 2148-2166
```cpp
      target = pc + offset;
    else
      target = pc + 8;
  }

  Context context;

  return WriteRegisterUnsigned(context, eRegisterKindDWARF, dwarf_pc_mips64,
                               target);
}

bool EmulateInstructionMIPS64::Emulate_BNZB(llvm::MCInst &insn) {
  return Emulate_MSA_Branch_DF(insn, 1, true);
}

bool EmulateInstructionMIPS64::Emulate_BNZH(llvm::MCInst &insn) {
  return Emulate_MSA_Branch_DF(insn, 2, true);
}

```
- **EN**: Implements logic around `WriteRegisterUnsigned`, `Emulate_BNZB`, `Emulate_MSA_Branch_DF`, `Emulate_BNZH`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `WriteRegisterUnsigned`, `Emulate_BNZB`, `Emulate_MSA_Branch_DF`, `Emulate_BNZH` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 2167-2186
```cpp
bool EmulateInstructionMIPS64::Emulate_BNZW(llvm::MCInst &insn) {
  return Emulate_MSA_Branch_DF(insn, 4, true);
}

bool EmulateInstructionMIPS64::Emulate_BNZD(llvm::MCInst &insn) {
  return Emulate_MSA_Branch_DF(insn, 8, true);
}

bool EmulateInstructionMIPS64::Emulate_BZB(llvm::MCInst &insn) {
  return Emulate_MSA_Branch_DF(insn, 1, false);
}

bool EmulateInstructionMIPS64::Emulate_BZH(llvm::MCInst &insn) {
  return Emulate_MSA_Branch_DF(insn, 2, false);
}

bool EmulateInstructionMIPS64::Emulate_BZW(llvm::MCInst &insn) {
  return Emulate_MSA_Branch_DF(insn, 4, false);
}

```
- **EN**: Implements logic around `Emulate_BNZW`, `Emulate_MSA_Branch_DF`, `Emulate_BNZD`, `Emulate_BZB`, and 2 more symbols.
- **CN**: 围绕 `Emulate_BNZW`, `Emulate_MSA_Branch_DF`, `Emulate_BNZD`, `Emulate_BZB`, and 2 more symbols 实现具体逻辑。

### Lines 2187-2206
```cpp
bool EmulateInstructionMIPS64::Emulate_BZD(llvm::MCInst &insn) {
  return Emulate_MSA_Branch_DF(insn, 8, false);
}

bool EmulateInstructionMIPS64::Emulate_MSA_Branch_DF(llvm::MCInst &insn,
                                                     int element_byte_size,
                                                     bool bnz) {
  bool success = false, branch_hit = true;
  int64_t target = 0;
  RegisterValue reg_value;
  const uint8_t *ptr = nullptr;

  uint32_t wt = m_reg_info->getEncodingValue(insn.getOperand(0).getReg());
  int64_t offset = insn.getOperand(1).getImm();

  int64_t pc =
      ReadRegisterUnsigned(eRegisterKindDWARF, dwarf_pc_mips64, 0, &success);
  if (!success)
    return false;

```
- **EN**: Implements logic around `Emulate_BZD`, `Emulate_MSA_Branch_DF`, `getEncodingValue`, `getOperand`, and 1 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `Emulate_BZD`, `Emulate_MSA_Branch_DF`, `getEncodingValue`, `getOperand`, and 1 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 2207-2238
```cpp
  if (ReadRegister(eRegisterKindDWARF, dwarf_w0_mips64 + wt, reg_value))
    ptr = (const uint8_t *)reg_value.GetBytes();
  else
    return false;

  for (int i = 0; i < 16 / element_byte_size; i++) {
    switch (element_byte_size) {
    case 1:
      if ((*ptr == 0 && bnz) || (*ptr != 0 && !bnz))
        branch_hit = false;
      break;
    case 2:
      if ((*(const uint16_t *)ptr == 0 && bnz) ||
          (*(const uint16_t *)ptr != 0 && !bnz))
        branch_hit = false;
      break;
    case 4:
      if ((*(const uint32_t *)ptr == 0 && bnz) ||
          (*(const uint32_t *)ptr != 0 && !bnz))
        branch_hit = false;
      break;
    case 8:
      if ((*(const uint64_t *)ptr == 0 && bnz) ||
          (*(const uint64_t *)ptr != 0 && !bnz))
        branch_hit = false;
      break;
    }
    if (!branch_hit)
      break;
    ptr = ptr + element_byte_size;
  }

```
- **EN**: Implements logic around `ReadRegister`, `GetBytes`.
- **CN**: 围绕 `ReadRegister`, `GetBytes` 实现具体逻辑。

### Lines 2239-2258
```cpp
  if (branch_hit)
    target = pc + offset;
  else
    target = pc + 8;

  Context context;
  context.type = eContextRelativeBranchImmediate;

  return WriteRegisterUnsigned(context, eRegisterKindDWARF, dwarf_pc_mips64,
                               target);
}

bool EmulateInstructionMIPS64::Emulate_BNZV(llvm::MCInst &insn) {
  return Emulate_MSA_Branch_V(insn, true);
}

bool EmulateInstructionMIPS64::Emulate_BZV(llvm::MCInst &insn) {
  return Emulate_MSA_Branch_V(insn, false);
}

```
- **EN**: Implements logic around `WriteRegisterUnsigned`, `Emulate_BNZV`, `Emulate_MSA_Branch_V`, `Emulate_BZV`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `WriteRegisterUnsigned`, `Emulate_BNZV`, `Emulate_MSA_Branch_V`, `Emulate_BZV` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 2259-2280
```cpp
bool EmulateInstructionMIPS64::Emulate_MSA_Branch_V(llvm::MCInst &insn,
                                                    bool bnz) {
  bool success = false;
  int64_t target = 0;
  llvm::APInt wr_val = llvm::APInt::getZero(128);
  llvm::APInt fail_value = llvm::APInt::getMaxValue(128);
  llvm::APInt zero_value = llvm::APInt::getZero(128);
  RegisterValue reg_value;

  uint32_t wt = m_reg_info->getEncodingValue(insn.getOperand(0).getReg());
  int64_t offset = insn.getOperand(1).getImm();

  int64_t pc =
      ReadRegisterUnsigned(eRegisterKindDWARF, dwarf_pc_mips64, 0, &success);
  if (!success)
    return false;

  if (ReadRegister(eRegisterKindDWARF, dwarf_w0_mips64 + wt, reg_value))
    wr_val = reg_value.GetAsUInt128(fail_value);
  else
    return false;

```
- **EN**: Implements logic around `Emulate_MSA_Branch_V`, `getZero`, `getMaxValue`, `getEncodingValue`, and 4 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `Emulate_MSA_Branch_V`, `getZero`, `getMaxValue`, `getEncodingValue`, and 4 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 2281-2299
```cpp
  if ((llvm::APInt::isSameValue(zero_value, wr_val) && !bnz) ||
      (!llvm::APInt::isSameValue(zero_value, wr_val) && bnz))
    target = pc + offset;
  else
    target = pc + 8;

  Context context;
  context.type = eContextRelativeBranchImmediate;

  return WriteRegisterUnsigned(context, eRegisterKindDWARF, dwarf_pc_mips64,
                               target);
}

bool EmulateInstructionMIPS64::Emulate_LDST_Imm(llvm::MCInst &insn) {
  bool success = false;
  uint32_t base;
  int64_t imm, address;
  Context bad_vaddr_context;

```
- **EN**: Implements logic around `isSameValue`, `WriteRegisterUnsigned`, `Emulate_LDST_Imm`; this block coordinates debugger runtime objects, events, or asynchronous control flow; maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `isSameValue`, `WriteRegisterUnsigned`, `Emulate_LDST_Imm` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并将可执行状态映射回模块、符号、节区或地址。

### Lines 2300-2321
```cpp
  uint32_t num_operands = insn.getNumOperands();
  base =
      m_reg_info->getEncodingValue(insn.getOperand(num_operands - 2).getReg());
  imm = insn.getOperand(num_operands - 1).getImm();

  if (!GetRegisterInfo(eRegisterKindDWARF, dwarf_zero_mips + base))
    return false;

  /* read base register */
  address = ReadRegisterUnsigned(eRegisterKindDWARF, dwarf_zero_mips + base, 0,
                                 &success);
  if (!success)
    return false;

  /* destination address */
  address = address + imm;

  /* Set the bad_vaddr register with base address used in the instruction */
  bad_vaddr_context.type = eContextInvalid;
  WriteRegisterUnsigned(bad_vaddr_context, eRegisterKindDWARF, dwarf_bad_mips,
                        address);

```
- **EN**: Implements logic around `getNumOperands`, `getEncodingValue`, `getOperand`, `GetRegisterInfo`, and 2 more symbols; this block maps executable state back to modules, symbols, sections, or addresses; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `getNumOperands`, `getEncodingValue`, `getOperand`, `GetRegisterInfo`, and 2 more symbols 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 2322-2339
```cpp
  return true;
}

bool EmulateInstructionMIPS64::Emulate_LDST_Reg(llvm::MCInst &insn) {
  bool success = false;
  uint32_t base, index;
  int64_t address, index_address;
  Context bad_vaddr_context;

  uint32_t num_operands = insn.getNumOperands();
  base =
      m_reg_info->getEncodingValue(insn.getOperand(num_operands - 2).getReg());
  index =
      m_reg_info->getEncodingValue(insn.getOperand(num_operands - 1).getReg());

  if (!GetRegisterInfo(eRegisterKindDWARF, dwarf_zero_mips + base))
    return false;

```
- **EN**: Implements logic around `Emulate_LDST_Reg`, `getNumOperands`, `getEncodingValue`, `GetRegisterInfo`; this block maps executable state back to modules, symbols, sections, or addresses.
- **CN**: 围绕 `Emulate_LDST_Reg`, `getNumOperands`, `getEncodingValue`, `GetRegisterInfo` 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址。

### Lines 2340-2357
```cpp
  if (!GetRegisterInfo(eRegisterKindDWARF, dwarf_zero_mips + index))
    return false;

  /* read base register */
  address = ReadRegisterUnsigned(eRegisterKindDWARF, dwarf_zero_mips + base, 0,
                                 &success);
  if (!success)
    return false;

  /* read index register */
  index_address = ReadRegisterUnsigned(eRegisterKindDWARF,
                                       dwarf_zero_mips + index, 0, &success);
  if (!success)
    return false;

  /* destination address */
  address = address + index_address;

```
- **EN**: Implements logic around `GetRegisterInfo`, `ReadRegisterUnsigned`; this block maps executable state back to modules, symbols, sections, or addresses; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `GetRegisterInfo`, `ReadRegisterUnsigned` 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 2358-2364
```cpp
  /* Set the bad_vaddr register with base address used in the instruction */
  bad_vaddr_context.type = eContextInvalid;
  WriteRegisterUnsigned(bad_vaddr_context, eRegisterKindDWARF, dwarf_bad_mips,
                        address);

  return true;
}
```
- **EN**: Implements logic around `WriteRegisterUnsigned`; this block maps executable state back to modules, symbols, sections, or addresses; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `WriteRegisterUnsigned` 实现具体逻辑；该代码块将可执行状态映射回模块、符号、节区或地址，并建模机器状态、栈检查或面向反汇编的元数据。

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
- **User expressions / 用户表达式**:
  - **EN**: Wraps and evaluates source snippets inside the debuggee context.
  - **CN**: 在被调试进程上下文中包装并求值源码片段。

## Dependencies / 依赖关系

- **Direct LLDB/LLVM includes / 直接的 LLDB/LLVM 包含**: `EmulateInstructionMIPS64.h`, `lldb/Core/Address.h`, `lldb/Core/Opcode.h`, `lldb/Core/PluginManager.h`, `lldb/Host/PosixApi.h`, `lldb/Symbol/UnwindPlan.h`, `lldb/Utility/ArchSpec.h`, `lldb/Utility/ConstString.h`, `lldb/Utility/DataExtractor.h`, `lldb/Utility/RegisterValue.h` ... (+15 more)
- **Standard-library headers / 标准库头文件**: `<cstdlib>`, `<optional>`
- **Subsystem categories / 子系统类别**: shared LLVM infrastructure / 共享 LLVM 基础设施 (9), shared LLDB utility classes / 共享 LLDB 工具类 (5), LLDB core debugger abstractions / LLDB 核心调试抽象 (3), host OS abstraction helpers / 主机操作系统抽象辅助组件 (1), symbol and debug-info abstractions / 符号与调试信息抽象 (1), LLVM support-library helpers / LLVM Support 库辅助组件 (1), LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (1)
