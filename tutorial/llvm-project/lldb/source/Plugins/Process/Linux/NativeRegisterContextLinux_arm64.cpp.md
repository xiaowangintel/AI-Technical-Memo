# NativeRegisterContextLinux_arm64.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `lldb/source/Plugins/Process/Linux/NativeRegisterContextLinux_arm64.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: if defined(__arm64__) || defined(__aarch64__).
  - **CN**: 实现与 `NativeRegisterContextLinux_arm64` 相关的 LLDB 支持逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-22
```cpp
//===-- NativeRegisterContextLinux_arm64.cpp ------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#if defined(__arm64__) || defined(__aarch64__)

#include "NativeRegisterContextLinux_arm64.h"
#include "NativeRegisterContextLinux_arm.h"
#include "NativeRegisterContextLinux_arm64dbreg.h"

#include "lldb/Host/HostInfo.h"
#include "lldb/Host/common/NativeProcessProtocol.h"
#include "lldb/Host/linux/Ptrace.h"
#include "lldb/Utility/DataBufferHeap.h"
#include "lldb/Utility/Log.h"
#include "lldb/Utility/RegisterValue.h"
#include "lldb/Utility/Status.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `NativeRegisterContextLinux_arm64.h`, `NativeRegisterContextLinux_arm.h`, `NativeRegisterContextLinux_arm64dbreg.h`, `lldb/Host/HostInfo.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `NativeRegisterContextLinux_arm64.h`, `NativeRegisterContextLinux_arm.h`, `NativeRegisterContextLinux_arm64dbreg.h`, `lldb/Host/HostInfo.h`。

### Lines 23-41
```cpp
#include "Plugins/Process/Linux/NativeProcessLinux.h"
#include "Plugins/Process/Linux/Procfs.h"
#include "Plugins/Process/POSIX/ProcessPOSIXLog.h"
#include "Plugins/Process/Utility/MemoryTagManagerAArch64MTE.h"
#include "Plugins/Process/Utility/RegisterFlagsDetector_arm64.h"
#include "Plugins/Process/Utility/RegisterInfoPOSIX_arm64.h"

// System includes - They have to be included after framework includes because
// they define some macros which collide with variable names in other modules
#include <sys/uio.h>
// NT_PRSTATUS and NT_FPREGSET definition
#include <elf.h>
#include <mutex>
#include <optional>

#ifndef NT_ARM_SVE
#define NT_ARM_SVE 0x405 /* ARM Scalable Vector Extension */
#endif

```
- **EN**: Pulls in the headers needed by this translation unit, including `Plugins/Process/Linux/NativeProcessLinux.h`, `Plugins/Process/Linux/Procfs.h`, `Plugins/Process/POSIX/ProcessPOSIXLog.h`, `Plugins/Process/Utility/MemoryTagManagerAArch64MTE.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `Plugins/Process/Linux/NativeProcessLinux.h`, `Plugins/Process/Linux/Procfs.h`, `Plugins/Process/POSIX/ProcessPOSIXLog.h`, `Plugins/Process/Utility/MemoryTagManagerAArch64MTE.h`。

### Lines 42-59
```cpp
#ifndef NT_ARM_SSVE
#define NT_ARM_SSVE                                                            \
  0x40b /* ARM Scalable Matrix Extension, Streaming SVE mode */
#endif

#ifndef NT_ARM_ZA
#define NT_ARM_ZA 0x40c /* ARM Scalable Matrix Extension, Array Storage */
#endif

#ifndef NT_ARM_ZT
#define NT_ARM_ZT                                                              \
  0x40d /* ARM Scalable Matrix Extension 2, lookup table register */
#endif

#ifndef NT_ARM_PAC_MASK
#define NT_ARM_PAC_MASK 0x406 /* Pointer authentication code masks */
#endif

```
- **EN**: Defines preprocessor-controlled structure, feature gates, or include-guard state.
- **CN**: 定义受预处理器控制的结构、特性开关或头文件保护状态。

### Lines 60-79
```cpp
#ifndef NT_ARM_TAGGED_ADDR_CTRL
#define NT_ARM_TAGGED_ADDR_CTRL 0x409 /* Tagged address control register */
#endif

#ifndef NT_ARM_FPMR
#define NT_ARM_FPMR 0x40e /* Floating point mode register */
#endif

#ifndef NT_ARM_POE
#define NT_ARM_POE 0x40f /* Permission Overlay registers */
#endif

#ifndef NT_ARM_GCS
#define NT_ARM_GCS 0x410 /* Guarded Control Stack control registers */
#endif

#ifndef HWCAP_PACA
#define HWCAP_PACA (1 << 30)
#endif

```
- **EN**: Defines preprocessor-controlled structure, feature gates, or include-guard state.
- **CN**: 定义受预处理器控制的结构、特性开关或头文件保护状态。

### Lines 80-99
```cpp
#ifndef HWCAP_GCS
#define HWCAP_GCS (1UL << 32)
#endif

#ifndef HWCAP2_MTE
#define HWCAP2_MTE (1 << 18)
#endif

#ifndef HWCAP2_FPMR
#define HWCAP2_FPMR (1UL << 48)
#endif

#ifndef HWCAP2_POE
#define HWCAP2_POE (1ULL << 63)
#endif

using namespace lldb;
using namespace lldb_private;
using namespace lldb_private::process_linux;

```
- **EN**: Defines preprocessor-controlled structure, feature gates, or include-guard state.
- **CN**: 定义受预处理器控制的结构、特性开关或头文件保护状态。

### Lines 100-117
```cpp
// A NativeRegisterContext is constructed per thread, but all threads' registers
// will contain the same fields. Therefore this mutex prevents each instance
// competing with the other, and subsequent instances from having to detect the
// fields all over again.
static std::mutex g_register_flags_detector_mutex;
static Arm64RegisterFlagsDetector g_register_flags_detector;

std::unique_ptr<NativeRegisterContextLinux>
NativeRegisterContextLinux::CreateHostNativeRegisterContextLinux(
    const ArchSpec &target_arch, NativeThreadLinux &native_thread) {
  switch (target_arch.GetMachine()) {
  case llvm::Triple::arm:
    return std::make_unique<NativeRegisterContextLinux_arm>(target_arch,
                                                            native_thread);
  case llvm::Triple::aarch64: {
    // Configure register sets supported by this AArch64 target.
    // Read SVE header to check for SVE support.
    struct sve::user_sve_header sve_header;
```
- **EN**: Introduces declarations for `sve::user_sve_header`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `sve::user_sve_header` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 118-139
```cpp
    struct iovec ioVec;
    ioVec.iov_base = &sve_header;
    ioVec.iov_len = sizeof(sve_header);
    unsigned int regset = NT_ARM_SVE;

    Flags opt_regsets;
    if (NativeProcessLinux::PtraceWrapper(PTRACE_GETREGSET,
                                          native_thread.GetID(), &regset,
                                          &ioVec, sizeof(sve_header))
            .Success())
      opt_regsets.Set(RegisterInfoPOSIX_arm64::eRegsetMaskSVE);

    // We may have the Scalable Matrix Extension (SME) which adds a
    // streaming SVE mode. Systems can have SVE and/or SME.
    ioVec.iov_len = sizeof(sve_header);
    regset = NT_ARM_SSVE;
    if (NativeProcessLinux::PtraceWrapper(PTRACE_GETREGSET,
                                          native_thread.GetID(), &regset,
                                          &ioVec, sizeof(sve_header))
            .Success())
      opt_regsets.Set(RegisterInfoPOSIX_arm64::eRegsetMaskSSVE);

```
- **EN**: Introduces declarations for `iovec`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `iovec` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 140-160
```cpp
    sve::user_za_header za_header;
    ioVec.iov_base = &za_header;
    ioVec.iov_len = sizeof(za_header);
    regset = NT_ARM_ZA;
    if (NativeProcessLinux::PtraceWrapper(PTRACE_GETREGSET,
                                          native_thread.GetID(), &regset,
                                          &ioVec, sizeof(za_header))
            .Success())
      opt_regsets.Set(RegisterInfoPOSIX_arm64::eRegsetMaskZA);

    // SME's ZT0 is a 512 bit register.
    std::array<uint8_t, 64> zt_reg;
    ioVec.iov_base = zt_reg.data();
    ioVec.iov_len = zt_reg.size();
    regset = NT_ARM_ZT;
    if (NativeProcessLinux::PtraceWrapper(PTRACE_GETREGSET,
                                          native_thread.GetID(), &regset,
                                          &ioVec, zt_reg.size())
            .Success())
      opt_regsets.Set(RegisterInfoPOSIX_arm64::eRegsetMaskZT);

```
- **EN**: Implements logic around `PtraceWrapper`, `GetID`, `Success`, `Set`, and 2 more symbols; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `PtraceWrapper`, `GetID`, `Success`, `Set`, and 2 more symbols 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 161-180
```cpp
    NativeProcessLinux &process = native_thread.GetProcess();

    std::optional<uint64_t> auxv_at_hwcap =
        process.GetAuxValue(AuxVector::AUXV_AT_HWCAP);
    if (auxv_at_hwcap && (*auxv_at_hwcap & HWCAP_PACA))
      opt_regsets.Set(RegisterInfoPOSIX_arm64::eRegsetMaskPAuth);

    std::optional<uint64_t> auxv_at_hwcap2 =
        process.GetAuxValue(AuxVector::AUXV_AT_HWCAP2);
    if (auxv_at_hwcap2) {
      if (*auxv_at_hwcap2 & HWCAP2_MTE)
        opt_regsets.Set(RegisterInfoPOSIX_arm64::eRegsetMaskMTE);
      if (*auxv_at_hwcap2 & HWCAP2_FPMR)
        opt_regsets.Set(RegisterInfoPOSIX_arm64::eRegsetMaskFPMR);
      if (*auxv_at_hwcap & HWCAP_GCS)
        opt_regsets.Set(RegisterInfoPOSIX_arm64::eRegsetMaskGCS);
      if (*auxv_at_hwcap2 & HWCAP2_POE)
        opt_regsets.Set(RegisterInfoPOSIX_arm64::eRegsetMaskPOE);
    }

```
- **EN**: Implements logic around `GetProcess`, `GetAuxValue`, `Set`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `GetProcess`, `GetAuxValue`, `Set` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 181-200
```cpp
    opt_regsets.Set(RegisterInfoPOSIX_arm64::eRegsetMaskTLS);

    std::optional<uint64_t> auxv_at_hwcap3 =
        process.GetAuxValue(AuxVector::AUXV_AT_HWCAP3);
    std::lock_guard<std::mutex> lock(g_register_flags_detector_mutex);
    if (!g_register_flags_detector.HasDetected())
      g_register_flags_detector.DetectFields(auxv_at_hwcap.value_or(0),
                                             auxv_at_hwcap2.value_or(0),
                                             auxv_at_hwcap3.value_or(0));

    auto register_info_up =
        std::make_unique<RegisterInfoPOSIX_arm64>(target_arch, opt_regsets);
    return std::make_unique<NativeRegisterContextLinux_arm64>(
        target_arch, native_thread, std::move(register_info_up));
  }
  default:
    llvm_unreachable("have no register context for architecture");
  }
}

```
- **EN**: Implements logic around `Set`, `GetAuxValue`, `lock`, `HasDetected`, and 5 more symbols; this block coordinates debugger runtime objects, events, or asynchronous control flow; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `Set`, `GetAuxValue`, `lock`, `HasDetected`, and 5 more symbols 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 201-220
```cpp
llvm::Expected<ArchSpec>
NativeRegisterContextLinux::DetermineArchitecture(lldb::tid_t tid) {
  return DetermineArchitectureViaGPR(
      tid, RegisterInfoPOSIX_arm64::GetGPRSizeStatic());
}

NativeRegisterContextLinux_arm64::NativeRegisterContextLinux_arm64(
    const ArchSpec &target_arch, NativeThreadProtocol &native_thread,
    std::unique_ptr<RegisterInfoPOSIX_arm64> register_info_up)
    : NativeRegisterContextRegisterInfo(native_thread,
                                        register_info_up.release()),
      NativeRegisterContextLinux(native_thread) {
  g_register_flags_detector.UpdateRegisterInfo(
      GetRegisterInfoInterface().GetRegisterInfo(),
      GetRegisterInfoInterface().GetRegisterCount());

  // 16 is just a maximum value, query hardware for actual watchpoint count
  m_max_hwp_supported = 16;
  m_max_hbp_supported = 16;

```
- **EN**: Implements logic around `DetermineArchitecture`, `DetermineArchitectureViaGPR`, `GetGPRSizeStatic`, `NativeRegisterContextLinux_arm64`, and 5 more symbols; this block propagates recoverable errors, status objects, or diagnostics; tracks breakpoint state, stop conditions, or hit-processing policy.
- **CN**: 围绕 `DetermineArchitecture`, `DetermineArchitectureViaGPR`, `GetGPRSizeStatic`, `NativeRegisterContextLinux_arm64`, and 5 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并跟踪断点状态、停机条件或命中处理策略。

### Lines 221-242
```cpp
  m_gpr_is_valid = false;
  m_fpu_is_valid = false;
  m_sve_buffer_is_valid = false;
  m_sve_header_is_valid = false;
  m_pac_mask_is_valid = false;
  m_mte_ctrl_is_valid = false;
  m_tls_is_valid = false;
  m_zt_buffer_is_valid = false;
  m_fpmr_is_valid = false;
  m_gcs_is_valid = false;
  m_poe_is_valid = false;

  // SME adds the tpidr2 register
  m_tls_size = GetRegisterInfo().IsSSVEPresent() ? sizeof(m_tls_regs)
                                                 : sizeof(m_tls_regs.tpidr_reg);

  if (GetRegisterInfo().IsSVEPresent() || GetRegisterInfo().IsSSVEPresent())
    m_sve_state = SVEState::Unknown;
  else
    m_sve_state = SVEState::Disabled;
}

```
- **EN**: Implements logic around `GetRegisterInfo`; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `GetRegisterInfo` 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 243-263
```cpp
RegisterInfoPOSIX_arm64 &
NativeRegisterContextLinux_arm64::GetRegisterInfo() const {
  return static_cast<RegisterInfoPOSIX_arm64 &>(*m_register_info_interface_up);
}

uint32_t NativeRegisterContextLinux_arm64::GetRegisterSetCount() const {
  return GetRegisterInfo().GetRegisterSetCount();
}

const RegisterSet *
NativeRegisterContextLinux_arm64::GetRegisterSet(uint32_t set_index) const {
  return GetRegisterInfo().GetRegisterSet(set_index);
}

uint32_t NativeRegisterContextLinux_arm64::GetUserRegisterCount() const {
  uint32_t count = 0;
  for (uint32_t set_index = 0; set_index < GetRegisterSetCount(); ++set_index)
    count += GetRegisterSet(set_index)->num_registers;
  return count;
}

```
- **EN**: Implements logic around `GetRegisterInfo`, `GetRegisterSetCount`, `GetRegisterSet`, `GetUserRegisterCount`.
- **CN**: 围绕 `GetRegisterInfo`, `GetRegisterSetCount`, `GetRegisterSet`, `GetUserRegisterCount` 实现具体逻辑。

### Lines 264-285
```cpp
Status
NativeRegisterContextLinux_arm64::ReadRegister(const RegisterInfo *reg_info,
                                               RegisterValue &reg_value) {
  Status error;

  if (!reg_info) {
    error = Status::FromErrorString("reg_info NULL");
    return error;
  }

  const uint32_t reg = reg_info->kinds[lldb::eRegisterKindLLDB];

  if (reg == LLDB_INVALID_REGNUM)
    return Status::FromErrorStringWithFormat(
        "no lldb regnum for %s",
        reg_info && reg_info->name ? reg_info->name : "<unknown register>");

  uint8_t *src;
  uint32_t offset = LLDB_INVALID_INDEX32;
  uint64_t sve_vg;
  std::vector<uint8_t> sve_reg_non_live;

```
- **EN**: Implements logic around `ReadRegister`, `FromErrorString`, `FromErrorStringWithFormat`; this block propagates recoverable errors, status objects, or diagnostics; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `ReadRegister`, `FromErrorString`, `FromErrorStringWithFormat` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 286-304
```cpp
  if (GetRegisterInfo().IsGPR(reg)) {
    error = ReadGPR();
    if (error.Fail())
      return error;

    offset = reg_info->byte_offset;
    assert(offset < GetGPRSize());
    src = (uint8_t *)GetGPRBuffer() + offset;

  } else if (GetRegisterInfo().IsFPR(reg)) {
    if (m_sve_state == SVEState::Disabled ||
        m_sve_state == SVEState::StreamingFPSIMD) {
      // FP registers come from the FP register set when:
      // * We only have SVE in streaming mode, and we are in non-streaming mode.
      // * We only have SIMD, no SVE in any mode.
      error = ReadFPR();
      if (error.Fail())
        return error;

```
- **EN**: Implements logic around `GetRegisterInfo`, `ReadGPR`, `Fail`, `assert`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `GetRegisterInfo`, `ReadGPR`, `Fail`, `assert`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 305-340
```cpp
      offset = CalculateFprOffset(reg_info,
                                  m_sve_state == SVEState::StreamingFPSIMD);
      assert(offset < GetFPRSize());
      src = (uint8_t *)GetFPRBuffer() + offset;
    } else {
      // SVE or SSVE enabled, we will read and cache SVE ptrace data.
      // In SIMD or Full mode, the data comes from the SVE regset. In streaming
      // mode it comes from the streaming SVE regset.
      error = ReadAllSVE();
      if (error.Fail())
        return error;

      // FPSR and FPCR will be located right after Z registers in
      // SVEState::FPSIMD while in SVEState::Full or SVEState::Streaming they
      // will be located at the end of register data after an alignment
      // correction based on currently selected vector length.
      uint32_t sve_reg_num = LLDB_INVALID_REGNUM;
      if (reg == GetRegisterInfo().GetRegNumFPSR()) {
        sve_reg_num = reg;
        if (m_sve_state == SVEState::Full || m_sve_state == SVEState::Streaming)
          offset = sve::PTraceFPSROffset(sve::vq_from_vl(m_sve_header.vl));
        else if (m_sve_state == SVEState::FPSIMD)
          offset = sve::ptrace_fpsimd_offset + (32 * 16);
      } else if (reg == GetRegisterInfo().GetRegNumFPCR()) {
        sve_reg_num = reg;
        if (m_sve_state == SVEState::Full || m_sve_state == SVEState::Streaming)
          offset = sve::PTraceFPCROffset(sve::vq_from_vl(m_sve_header.vl));
        else if (m_sve_state == SVEState::FPSIMD)
          offset = sve::ptrace_fpsimd_offset + (32 * 16) + 4;
      } else {
        // Extract SVE Z register value register number for this reg_info
        if (reg_info->value_regs &&
            reg_info->value_regs[0] != LLDB_INVALID_REGNUM)
          sve_reg_num = reg_info->value_regs[0];
        offset = CalculateSVEOffset(GetRegisterInfoAtIndex(sve_reg_num));
      }
```
- **EN**: Implements logic around `CalculateFprOffset`, `assert`, `GetFPRBuffer`, `ReadAllSVE`, and 5 more symbols; this block propagates recoverable errors, status objects, or diagnostics; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `CalculateFprOffset`, `assert`, `GetFPRBuffer`, `ReadAllSVE`, and 5 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 341-361
```cpp

      assert(offset < GetSVEBufferSize());
      src = (uint8_t *)GetSVEBuffer() + offset;
    }
  } else if (GetRegisterInfo().IsTLSReg(reg)) {
    error = ReadTLS();
    if (error.Fail())
      return error;

    offset = reg_info->byte_offset - GetRegisterInfo().GetTLSOffset();
    assert(offset < GetTLSBufferSize());
    src = (uint8_t *)GetTLSBuffer() + offset;
  } else if (GetRegisterInfo().IsSVEReg(reg)) {
    if (m_sve_state == SVEState::Disabled || m_sve_state == SVEState::Unknown)
      return Status::FromErrorString("SVE disabled or not supported");

    if (GetRegisterInfo().IsSVERegVG(reg)) {
      error = ReadSVEHeader();
      if (error.Fail())
        return error;

```
- **EN**: Implements logic around `assert`, `GetSVEBuffer`, `GetRegisterInfo`, `ReadTLS`, and 4 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `assert`, `GetSVEBuffer`, `GetRegisterInfo`, `ReadTLS`, and 4 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 362-380
```cpp
      sve_vg = GetSVERegVG();
      src = (uint8_t *)&sve_vg;
    } else if (m_sve_state == SVEState::StreamingFPSIMD) {
      // When we only have streaming SVE and we are in non-streaming mode,
      // we cannot read streaming SVE registers.

      // P and FFR show as 0s.
      if (GetRegisterInfo().IsSVEPReg(reg) ||
          GetRegisterInfo().IsSVERegFFR(reg)) {
        std::vector<uint8_t> fake_reg(reg_info->byte_size, 0);
        reg_value.SetFromMemoryData(*reg_info, &fake_reg[0],
                                    reg_info->byte_size, eByteOrderLittle,
                                    error);
        return error;
      }

      // For Z registers, zero extend the 128-bit FP register to Z register
      // size.

```
- **EN**: Implements logic around `GetSVERegVG`, `GetRegisterInfo`, `fake_reg`, `SetFromMemoryData`; this block propagates recoverable errors, status objects, or diagnostics; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `GetSVERegVG`, `GetRegisterInfo`, `fake_reg`, `SetFromMemoryData` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 381-398
```cpp
      error = ReadFPR();
      if (error.Fail())
        return error;

      // As we told the client we have Z registers, our own internal offsets
      // are set as if we were using an SVE context. We need to work out
      // an offset within the FP context instead:
      // struct user_fpsimd_state {
      // 	__uint128_t	vregs[32];
      // 	__u32		fpsr;
      // 	__u32		fpcr;
      // 	__u32		__reserved[2];
      // };
      const uint32_t z_num = reg - GetRegisterInfo().GetRegNumSVEZ0();
      offset = z_num * 16;
      assert(offset < GetFPRSize());
      src = (uint8_t *)GetFPRBuffer() + offset;

```
- **EN**: Implements logic around `ReadFPR`, `Fail`, `GetRegisterInfo`, `assert`, and 1 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `ReadFPR`, `Fail`, `GetRegisterInfo`, `assert`, and 1 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 399-418
```cpp
      // Copy from FP into a fake Z value.
      std::vector<uint8_t> fake_z(reg_info->byte_size, 0);
      std::memcpy(&fake_z[0], src, 16 /* 128 bits */);
      reg_value.SetFromMemoryData(*reg_info, &fake_z[0], reg_info->byte_size,
                                  eByteOrderLittle, error);

      return error;
    } else {
      // SVE enabled, we will read and cache SVE ptrace data
      error = ReadAllSVE();
      if (error.Fail())
        return error;

      if (m_sve_state == SVEState::FPSIMD) {
        // In FPSIMD state SVE payload mirrors legacy fpsimd struct and so
        // just copy 16 bytes of v register to the start of z register. All
        // other SVE register will be set to zero.
        sve_reg_non_live.resize(reg_info->byte_size, 0);
        src = sve_reg_non_live.data();

```
- **EN**: Implements logic around `fake_z`, `memcpy`, `SetFromMemoryData`, `ReadAllSVE`, and 3 more symbols; this block propagates recoverable errors, status objects, or diagnostics; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `fake_z`, `memcpy`, `SetFromMemoryData`, `ReadAllSVE`, and 3 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 419-443
```cpp
        if (GetRegisterInfo().IsSVEZReg(reg)) {
          offset = CalculateSVEOffset(reg_info);
          assert(offset < GetSVEBufferSize());
          ::memcpy(sve_reg_non_live.data(), (uint8_t *)GetSVEBuffer() + offset,
                   16);
        }
      } else {
        offset = CalculateSVEOffset(reg_info);
        assert(offset < GetSVEBufferSize());
        src = (uint8_t *)GetSVEBuffer() + offset;
      }
    }
  } else if (GetRegisterInfo().IsPAuthReg(reg)) {
    error = ReadPAuthMask();
    if (error.Fail())
      return error;

    offset = reg_info->byte_offset - GetRegisterInfo().GetPAuthOffset();
    assert(offset < GetPACMaskSize());
    src = (uint8_t *)GetPACMask() + offset;
  } else if (GetRegisterInfo().IsMTEReg(reg)) {
    error = ReadMTEControl();
    if (error.Fail())
      return error;

```
- **EN**: Implements logic around `GetRegisterInfo`, `CalculateSVEOffset`, `assert`, `memcpy`, and 5 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `GetRegisterInfo`, `CalculateSVEOffset`, `assert`, `memcpy`, and 5 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 444-468
```cpp
    offset = reg_info->byte_offset - GetRegisterInfo().GetMTEOffset();
    assert(offset < GetMTEControlSize());
    src = (uint8_t *)GetMTEControl() + offset;
  } else if (GetRegisterInfo().IsSMEReg(reg)) {
    if (GetRegisterInfo().IsSMERegZA(reg)) {
      error = ReadZAHeader();
      if (error.Fail())
        return error;

      // If there is only a header and no registers, ZA is inactive. Read as 0
      // in this case.
      if (m_za_header.size == sizeof(m_za_header)) {
        // This will get reconfigured/reset later, so we are safe to use it.
        // ZA is a square of VL * VL and the ptrace buffer also includes the
        // header itself.
        m_za_ptrace_payload.resize(((m_za_header.vl) * (m_za_header.vl)) +
                                   GetZAHeaderSize());
        std::fill(m_za_ptrace_payload.begin(), m_za_ptrace_payload.end(), 0);
      } else {
        // ZA is active, read the real register.
        error = ReadZA();
        if (error.Fail())
          return error;
      }

```
- **EN**: Implements logic around `GetRegisterInfo`, `assert`, `GetMTEControl`, `ReadZAHeader`, and 5 more symbols; this block propagates recoverable errors, status objects, or diagnostics; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `GetRegisterInfo`, `assert`, `GetMTEControl`, `ReadZAHeader`, and 5 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 469-486
```cpp
      // ZA is part of the SME set but uses a separate member buffer for
      // storage. Therefore its effective byte offset is always 0 even if it
      // isn't 0 within the SME register set.
      src = (uint8_t *)GetZABuffer() + GetZAHeaderSize();
    } else if (GetRegisterInfo().IsSMERegZT(reg)) {
      // Unlike ZA, the kernel will return register data for ZT0 when ZA is not
      // enabled. This data will be all 0s so we don't have to invent anything
      // like we did for ZA.
      error = ReadZT();
      if (error.Fail())
        return error;

      src = (uint8_t *)GetZTBuffer();
    } else {
      error = ReadSMESVG();
      if (error.Fail())
        return error;

```
- **EN**: Implements logic around `GetZABuffer`, `GetRegisterInfo`, `ReadZT`, `Fail`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `GetZABuffer`, `GetRegisterInfo`, `ReadZT`, `Fail`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 487-506
```cpp
      // This is a psuedo so it never fails.
      ReadSMEControl();

      offset = reg_info->byte_offset - GetRegisterInfo().GetSMEOffset();
      assert(offset < GetSMEPseudoBufferSize());
      src = (uint8_t *)GetSMEPseudoBuffer() + offset;
    }
  } else if (GetRegisterInfo().IsFPMRReg(reg)) {
    error = ReadFPMR();
    if (error.Fail())
      return error;

    offset = reg_info->byte_offset - GetRegisterInfo().GetFPMROffset();
    assert(offset < GetFPMRBufferSize());
    src = (uint8_t *)GetFPMRBuffer() + offset;
  } else if (GetRegisterInfo().IsGCSReg(reg)) {
    error = ReadGCS();
    if (error.Fail())
      return error;

```
- **EN**: Implements logic around `ReadSMEControl`, `GetRegisterInfo`, `assert`, `GetSMEPseudoBuffer`, and 4 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `ReadSMEControl`, `GetRegisterInfo`, `assert`, `GetSMEPseudoBuffer`, and 4 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 507-525
```cpp
    offset = reg_info->byte_offset - GetRegisterInfo().GetGCSOffset();
    assert(offset < GetGCSBufferSize());
    src = (uint8_t *)GetGCSBuffer() + offset;
  } else if (GetRegisterInfo().IsPOEReg(reg)) {
    error = ReadPOE();
    if (error.Fail())
      return error;

    offset = reg_info->byte_offset - GetRegisterInfo().GetPOEOffset();
    assert(offset < GetPOEBufferSize());
    src = (uint8_t *)GetPOEBuffer() + offset;
  } else
    return Status::FromErrorString(
        "failed - register wasn't recognized to be a GPR or an FPR, "
        "write strategy unknown");

  reg_value.SetFromMemoryData(*reg_info, src, reg_info->byte_size,
                              eByteOrderLittle, error);

```
- **EN**: Implements logic around `GetRegisterInfo`, `assert`, `GetGCSBuffer`, `ReadPOE`, and 4 more symbols; this block propagates recoverable errors, status objects, or diagnostics; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `GetRegisterInfo`, `assert`, `GetGCSBuffer`, `ReadPOE`, and 4 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 526-546
```cpp
  return error;
}

Status NativeRegisterContextLinux_arm64::WriteRegister(
    const RegisterInfo *reg_info, const RegisterValue &reg_value) {
  Status error;

  if (!reg_info)
    return Status::FromErrorString("reg_info NULL");

  const uint32_t reg = reg_info->kinds[lldb::eRegisterKindLLDB];

  if (reg == LLDB_INVALID_REGNUM)
    return Status::FromErrorStringWithFormat(
        "no lldb regnum for %s",
        reg_info && reg_info->name ? reg_info->name : "<unknown register>");

  uint8_t *dst;
  uint32_t offset = LLDB_INVALID_INDEX32;
  std::vector<uint8_t> sve_reg_non_live;

```
- **EN**: Implements logic around `WriteRegister`, `FromErrorString`, `FromErrorStringWithFormat`; this block propagates recoverable errors, status objects, or diagnostics; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `WriteRegister`, `FromErrorString`, `FromErrorStringWithFormat` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 547-566
```cpp
  if (GetRegisterInfo().IsGPR(reg)) {
    error = ReadGPR();
    if (error.Fail())
      return error;

    assert(reg_info->byte_offset < GetGPRSize());
    dst = (uint8_t *)GetGPRBuffer() + reg_info->byte_offset;
    ::memcpy(dst, reg_value.GetBytes(), reg_info->byte_size);

    return WriteGPR();
  } else if (GetRegisterInfo().IsFPR(reg)) {
    if (m_sve_state == SVEState::Disabled ||
        m_sve_state == SVEState::StreamingFPSIMD) {
      // SVE is not present, or we only have it in streaming mode and are
      // currently outside of streaming mode. Take normal route for FPU register
      // access.
      error = ReadFPR();
      if (error.Fail())
        return error;

```
- **EN**: Implements logic around `GetRegisterInfo`, `ReadGPR`, `Fail`, `assert`, and 4 more symbols; this block propagates recoverable errors, status objects, or diagnostics; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `GetRegisterInfo`, `ReadGPR`, `Fail`, `assert`, and 4 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 567-602
```cpp
      offset = CalculateFprOffset(reg_info,
                                  m_sve_state == SVEState::StreamingFPSIMD);
      assert(offset < GetFPRSize());
      dst = (uint8_t *)GetFPRBuffer() + offset;
      ::memcpy(dst, reg_value.GetBytes(), reg_info->byte_size);

      return WriteFPR();
    } else {
      // SVE enabled, we will read and cache SVE ptrace data.
      error = ReadAllSVE();
      if (error.Fail())
        return error;

      // FPSR and FPCR will be located right after Z registers in
      // SVEState::FPSIMD while in SVEState::Full or SVEState::Streaming they
      // will be located at the end of register data after an alignment
      // correction based on currently selected vector length.
      uint32_t sve_reg_num = LLDB_INVALID_REGNUM;
      if (reg == GetRegisterInfo().GetRegNumFPSR()) {
        sve_reg_num = reg;
        if (m_sve_state == SVEState::Full || m_sve_state == SVEState::Streaming)
          offset = sve::PTraceFPSROffset(sve::vq_from_vl(m_sve_header.vl));
        else if (m_sve_state == SVEState::FPSIMD)
          offset = sve::ptrace_fpsimd_offset + (32 * 16);
      } else if (reg == GetRegisterInfo().GetRegNumFPCR()) {
        sve_reg_num = reg;
        if (m_sve_state == SVEState::Full || m_sve_state == SVEState::Streaming)
          offset = sve::PTraceFPCROffset(sve::vq_from_vl(m_sve_header.vl));
        else if (m_sve_state == SVEState::FPSIMD)
          offset = sve::ptrace_fpsimd_offset + (32 * 16) + 4;
      } else {
        // Extract SVE Z register value register number for this reg_info
        if (reg_info->value_regs &&
            reg_info->value_regs[0] != LLDB_INVALID_REGNUM)
          sve_reg_num = reg_info->value_regs[0];
        offset = CalculateSVEOffset(GetRegisterInfoAtIndex(sve_reg_num));
```
- **EN**: Implements logic around `CalculateFprOffset`, `assert`, `GetFPRBuffer`, `memcpy`, and 7 more symbols; this block propagates recoverable errors, status objects, or diagnostics; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `CalculateFprOffset`, `assert`, `GetFPRBuffer`, `memcpy`, and 7 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 603-635
```cpp
      }

      assert(offset < GetSVEBufferSize());
      dst = (uint8_t *)GetSVEBuffer() + offset;
      ::memcpy(dst, reg_value.GetBytes(), reg_info->byte_size);
      return WriteAllSVE();
    }
  } else if (GetRegisterInfo().IsSVEReg(reg)) {
    if (m_sve_state == SVEState::Disabled || m_sve_state == SVEState::Unknown) {
      return Status::FromErrorString("SVE disabled or not supported");
    } else if (m_sve_state == SVEState::StreamingFPSIMD) {
      // When a target has SVE (in any state), the client is told that it has
      // real SVE registers and that the FP registers are just subregisters
      // of those SVE registers. This means that any FP write will be converted
      // into an SVE write.
      //
      // If we get here, it did that, but we are outside of streaming mode
      // on an SME only system. Meaning there's no way at all to write to actual
      // SVE registers.
      //
      // Instead we will extract the bottom 128 bits of the register,
      // write that via the standard FP route and then return the fake SVE
      // values as usual.
      //
      // We can only do this for Z registers. P, FFR and VG have no SIMD
      // equivalent.
      if (GetRegisterInfo().IsSVERegVG(reg) ||
          GetRegisterInfo().IsSVEPReg(reg) ||
          GetRegisterInfo().IsSVERegFFR(reg))
        return Status::FromErrorString(
            "Cannot write SVE VG, P or FFR registers while outside of "
            "streaming mode.");

```
- **EN**: Implements logic around `assert`, `GetSVEBuffer`, `memcpy`, `WriteAllSVE`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `assert`, `GetSVEBuffer`, `memcpy`, `WriteAllSVE`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 636-659
```cpp
      // We have told the client that we only have Z registers and the V
      // registers are subsets of Z. This means that the V byte offsets are
      // actually for the SVE register context, which we cannot access right
      // now. That is, v0 is offset 16, v1 is 16+vlen, and so on. So we will
      // manually patch this data into the FP context and write it.
      error = ReadFPR();
      if (error.Fail())
        return error;

      uint32_t z_num = reg - GetRegisterInfo().GetRegNumSVEZ0();
      offset = z_num * 16;
      assert(offset < GetFPRSize());
      dst = (uint8_t *)GetFPRBuffer() + offset;
      // If we get here we must have a Z register. Assume we have 16 bytes aka
      // 128 bits at least, enough to fill an FP V register.
      ::memcpy(dst, reg_value.GetBytes(), 16);

      return WriteFPR();
    } else {
      // Target has SVE enabled, we will read and cache SVE ptrace data
      error = ReadAllSVE();
      if (error.Fail())
        return error;

```
- **EN**: Implements logic around `ReadFPR`, `Fail`, `GetRegisterInfo`, `assert`, and 4 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `ReadFPR`, `Fail`, `GetRegisterInfo`, `assert`, and 4 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 660-680
```cpp
      if (GetRegisterInfo().IsSVERegVG(reg)) {
        uint64_t vg_value = reg_value.GetAsUInt64();

        if (sve::vl_valid(vg_value * 8)) {
          if (m_sve_header_is_valid && vg_value == GetSVERegVG())
            return error;

          SetSVERegVG(vg_value);

          error = WriteSVEHeader();
          if (error.Success()) {
            // Changing VG during streaming mode also changes the size of ZA.
            if (m_sve_state == SVEState::Streaming)
              m_za_header_is_valid = false;
            ConfigureRegisterContext();
          }

          if (m_sve_header_is_valid && vg_value == GetSVERegVG())
            return error;
        }

```
- **EN**: Implements logic around `GetRegisterInfo`, `GetAsUInt64`, `vl_valid`, `GetSVERegVG`, and 4 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `GetRegisterInfo`, `GetAsUInt64`, `vl_valid`, `GetSVERegVG`, and 4 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 681-706
```cpp
        return Status::FromErrorString("SVE vector length update failed.");
      }

      // If target supports SVE but currently in FPSIMD mode.
      if (m_sve_state == SVEState::FPSIMD) {
        // Here we will check if writing this SVE register enables
        // SVEState::Full
        bool set_sve_state_full = false;
        const uint8_t *reg_bytes = (const uint8_t *)reg_value.GetBytes();
        if (GetRegisterInfo().IsSVEZReg(reg)) {
          for (uint32_t i = 16; i < reg_info->byte_size; i++) {
            if (reg_bytes[i]) {
              set_sve_state_full = true;
              break;
            }
          }
        } else if (GetRegisterInfo().IsSVEPReg(reg) ||
                   reg == GetRegisterInfo().GetRegNumSVEFFR()) {
          for (uint32_t i = 0; i < reg_info->byte_size; i++) {
            if (reg_bytes[i]) {
              set_sve_state_full = true;
              break;
            }
          }
        }

```
- **EN**: Implements logic around `FromErrorString`, `GetBytes`, `GetRegisterInfo`; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `FromErrorString`, `GetBytes`, `GetRegisterInfo` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 707-731
```cpp
        if (!set_sve_state_full && GetRegisterInfo().IsSVEZReg(reg)) {
          // We are writing a Z register which is zero beyond 16 bytes so copy
          // first 16 bytes only as SVE payload mirrors legacy fpsimd structure
          offset = CalculateSVEOffset(reg_info);
          assert(offset < GetSVEBufferSize());
          dst = (uint8_t *)GetSVEBuffer() + offset;
          ::memcpy(dst, reg_value.GetBytes(), 16);

          return WriteAllSVE();
        } else
          return Status::FromErrorString(
              "SVE state change operation not supported");
      } else {
        offset = CalculateSVEOffset(reg_info);
        assert(offset < GetSVEBufferSize());
        dst = (uint8_t *)GetSVEBuffer() + offset;
        ::memcpy(dst, reg_value.GetBytes(), reg_info->byte_size);
        return WriteAllSVE();
      }
    }
  } else if (GetRegisterInfo().IsMTEReg(reg)) {
    error = ReadMTEControl();
    if (error.Fail())
      return error;

```
- **EN**: Implements logic around `GetRegisterInfo`, `CalculateSVEOffset`, `assert`, `GetSVEBuffer`, and 5 more symbols; this block propagates recoverable errors, status objects, or diagnostics; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `GetRegisterInfo`, `CalculateSVEOffset`, `assert`, `GetSVEBuffer`, and 5 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 732-754
```cpp
    offset = reg_info->byte_offset - GetRegisterInfo().GetMTEOffset();
    assert(offset < GetMTEControlSize());
    dst = (uint8_t *)GetMTEControl() + offset;
    ::memcpy(dst, reg_value.GetBytes(), reg_info->byte_size);

    return WriteMTEControl();
  } else if (GetRegisterInfo().IsTLSReg(reg)) {
    error = ReadTLS();
    if (error.Fail())
      return error;

    offset = reg_info->byte_offset - GetRegisterInfo().GetTLSOffset();
    assert(offset < GetTLSBufferSize());
    dst = (uint8_t *)GetTLSBuffer() + offset;
    ::memcpy(dst, reg_value.GetBytes(), reg_info->byte_size);

    return WriteTLS();
  } else if (GetRegisterInfo().IsSMEReg(reg)) {
    if (GetRegisterInfo().IsSMERegZA(reg)) {
      error = ReadZA();
      if (error.Fail())
        return error;

```
- **EN**: Implements logic around `GetRegisterInfo`, `assert`, `GetMTEControl`, `memcpy`, and 6 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `GetRegisterInfo`, `assert`, `GetMTEControl`, `memcpy`, and 6 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 755-780
```cpp
      // ZA is part of the SME set but not stored with the other SME registers.
      // So its byte offset is effectively always 0.
      dst = (uint8_t *)GetZABuffer() + GetZAHeaderSize();
      ::memcpy(dst, reg_value.GetBytes(), reg_info->byte_size);

      // While this is writing a header that contains a vector length, the only
      // way to change that is via the vg register. So here we assume the length
      // will always be the current length and no reconfigure is needed.
      return WriteZA();
    } else if (GetRegisterInfo().IsSMERegZT(reg)) {
      error = ReadZT();
      if (error.Fail())
        return error;

      dst = (uint8_t *)GetZTBuffer();
      ::memcpy(dst, reg_value.GetBytes(), reg_info->byte_size);

      return WriteZT();
    } else
      return Status::FromErrorString(
          "Writing to SVG or SVCR is not supported.");
  } else if (GetRegisterInfo().IsFPMRReg(reg)) {
    error = ReadFPMR();
    if (error.Fail())
      return error;

```
- **EN**: Implements logic around `GetZABuffer`, `memcpy`, `WriteZA`, `GetRegisterInfo`, and 6 more symbols; this block propagates recoverable errors, status objects, or diagnostics; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `GetZABuffer`, `memcpy`, `WriteZA`, `GetRegisterInfo`, and 6 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 781-802
```cpp
    offset = reg_info->byte_offset - GetRegisterInfo().GetFPMROffset();
    assert(offset < GetFPMRBufferSize());
    dst = (uint8_t *)GetFPMRBuffer() + offset;
    ::memcpy(dst, reg_value.GetBytes(), reg_info->byte_size);

    return WriteFPMR();
  } else if (GetRegisterInfo().IsGCSReg(reg)) {
    error = ReadGCS();
    if (error.Fail())
      return error;

    offset = reg_info->byte_offset - GetRegisterInfo().GetGCSOffset();
    assert(offset < GetGCSBufferSize());
    dst = (uint8_t *)GetGCSBuffer() + offset;
    ::memcpy(dst, reg_value.GetBytes(), reg_info->byte_size);

    return WriteGCS();
  } else if (GetRegisterInfo().IsPOEReg(reg)) {
    error = ReadPOE();
    if (error.Fail())
      return error;

```
- **EN**: Implements logic around `GetRegisterInfo`, `assert`, `GetFPMRBuffer`, `memcpy`, and 6 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `GetRegisterInfo`, `assert`, `GetFPMRBuffer`, `memcpy`, and 6 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 803-828
```cpp
    offset = reg_info->byte_offset - GetRegisterInfo().GetPOEOffset();
    assert(offset < GetPOEBufferSize());
    dst = (uint8_t *)GetPOEBuffer() + offset;
    ::memcpy(dst, reg_value.GetBytes(), reg_info->byte_size);

    return WritePOE();
  }

  return Status::FromErrorString("Failed to write register value");
}

enum RegisterSetType : uint32_t {
  GPR, // General purpose registers.
  SVE, // Used for SVE registers in streaming or non-streaming mode.
  FPR, // When there is no SVE, or SVE in FPSIMD mode, or streaming only SVE
       // that is in non-streaming mode.
  // Pointer authentication registers are read only, so not included here.
  MTE,  // Memory tagging control registers.
  TLS,  // Thread local storage registers.
  SME,  // ZA only, because SVCR and SVG are pseudo registers.
  SME2, // ZT only.
  FPMR, // Floating point mode control registers.
  GCS,  // Guarded Control Stack registers.
  POE,  // Permission Overlay registers.
};

```
- **EN**: Introduces declarations for `RegisterSetType`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `RegisterSetType` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 829-846
```cpp
static uint8_t *AddRegisterSetType(uint8_t *dst,
                                   RegisterSetType register_set_type) {
  *(reinterpret_cast<uint32_t *>(dst)) = register_set_type;
  return dst + sizeof(uint32_t);
}

static uint8_t *AddSavedRegistersData(uint8_t *dst, void *src, size_t size) {
  ::memcpy(dst, src, size);
  return dst + size;
}

static uint8_t *AddSavedRegisters(uint8_t *dst,
                                  enum RegisterSetType register_set_type,
                                  void *src, size_t size) {
  dst = AddRegisterSetType(dst, register_set_type);
  return AddSavedRegistersData(dst, src, size);
}

```
- **EN**: Introduces declarations for `RegisterSetType`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `RegisterSetType` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 847-870
```cpp
Status
NativeRegisterContextLinux_arm64::CacheAllRegisters(uint32_t &cached_size) {
  Status error;
  cached_size = sizeof(RegisterSetType) + GetGPRBufferSize();
  error = ReadGPR();
  if (error.Fail())
    return error;

  if (GetRegisterInfo().IsZAPresent()) {
    error = ReadZAHeader();
    if (error.Fail())
      return error;
    // Use header size here because the buffer may contain fake data when ZA is
    // disabled. We do not want to write this fake data (all 0s) because this
    // would tell the kernel that we want ZA to become active. Which is the
    // opposite of what we want in the case where it is currently inactive.
    cached_size += sizeof(RegisterSetType) + m_za_header.size;
    // For the same reason, we need to force it to be re-read so that it will
    // always contain the real header.
    m_za_buffer_is_valid = false;
    error = ReadZA();
    if (error.Fail())
      return error;

```
- **EN**: Implements logic around `CacheAllRegisters`, `GetGPRBufferSize`, `ReadGPR`, `Fail`, and 3 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `CacheAllRegisters`, `GetGPRBufferSize`, `ReadGPR`, `Fail`, and 3 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 871-902
```cpp
    // We will only be restoring ZT data if ZA is active. As writing to an
    // inactive ZT enables ZA, which may not be desireable.
    if (
        // If we have ZT0, or in other words, if we have SME2.
        GetRegisterInfo().IsZTPresent() &&
        // And ZA is active, which means that ZT0 is also active.
        m_za_header.size > sizeof(m_za_header)) {
      cached_size += sizeof(RegisterSetType) + GetZTBufferSize();
      // The kernel handles an inactive ZT0 for us, and it will read as 0s if
      // inactive (unlike ZA where we fake that behaviour).
      error = ReadZT();
      if (error.Fail())
        return error;
    }
  }

  // If SVE is enabled we need not copy FPR separately, unless we are in the
  // non-streaming mode of a streaming only process (as its non-streaming mode
  // is FPSIMD, rather than SVE).
  if ((GetRegisterInfo().IsSVEPresent() || GetRegisterInfo().IsSSVEPresent()) &&
      m_sve_state != SVEState::StreamingFPSIMD) {
    // Store mode and register data.
    cached_size +=
        sizeof(RegisterSetType) + sizeof(m_sve_state) + GetSVEBufferSize();
    error = ReadAllSVE();
  } else {
    cached_size += sizeof(RegisterSetType) + GetFPRSize();
    error = ReadFPR();
  }
  if (error.Fail())
    return error;

```
- **EN**: Implements logic around `GetRegisterInfo`, `GetZTBufferSize`, `ReadZT`, `Fail`, and 4 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `GetRegisterInfo`, `GetZTBufferSize`, `ReadZT`, `Fail`, and 4 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 903-923
```cpp
  if (GetRegisterInfo().IsMTEPresent()) {
    cached_size += sizeof(RegisterSetType) + GetMTEControlSize();
    error = ReadMTEControl();
    if (error.Fail())
      return error;
  }

  if (GetRegisterInfo().IsFPMRPresent()) {
    cached_size += sizeof(RegisterSetType) + GetFPMRBufferSize();
    error = ReadFPMR();
    if (error.Fail())
      return error;
  }

  if (GetRegisterInfo().IsGCSPresent()) {
    cached_size += sizeof(RegisterSetType) + GetGCSBufferSize();
    error = ReadGCS();
    if (error.Fail())
      return error;
  }

```
- **EN**: Implements logic around `GetRegisterInfo`, `GetMTEControlSize`, `ReadMTEControl`, `Fail`, and 4 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `GetRegisterInfo`, `GetMTEControlSize`, `ReadMTEControl`, `Fail`, and 4 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 924-944
```cpp
  if (GetRegisterInfo().IsPOEPresent()) {
    cached_size += sizeof(RegisterSetType) + GetPOEBufferSize();
    error = ReadPOE();
    if (error.Fail())
      return error;
  }

  // tpidr is always present but tpidr2 depends on SME.
  cached_size += sizeof(RegisterSetType) + GetTLSBufferSize();
  error = ReadTLS();

  return error;
}

Status NativeRegisterContextLinux_arm64::ReadAllRegisterValues(
    lldb::WritableDataBufferSP &data_sp) {
  // AArch64 register data must contain GPRs and either FPR or SVE registers.
  // SVE registers can be non-streaming (aka SVE) or streaming (aka SSVE).
  // Finally an optional MTE register. Pointer Authentication (PAC) registers
  // are read-only and will be skipped.

```
- **EN**: Implements logic around `GetRegisterInfo`, `GetPOEBufferSize`, `ReadPOE`, `Fail`, and 3 more symbols; this block propagates recoverable errors, status objects, or diagnostics; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `GetRegisterInfo`, `GetPOEBufferSize`, `ReadPOE`, `Fail`, and 3 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 945-980
```cpp
  // In order to create register data checkpoint we first read all register
  // values if not done already and calculate total size of register set data.
  // We store all register values in data_sp by copying full PTrace data that
  // corresponds to register sets enabled by current register context.

  uint32_t reg_data_byte_size = 0;
  Status error = CacheAllRegisters(reg_data_byte_size);
  if (error.Fail())
    return error;

  data_sp.reset(new DataBufferHeap(reg_data_byte_size, 0));
  uint8_t *dst = data_sp->GetBytes();

  dst = AddSavedRegisters(dst, RegisterSetType::GPR, GetGPRBuffer(),
                          GetGPRBufferSize());

  // Streaming SVE and the ZA register both use the streaming vector length.
  // When you change this, the kernel will invalidate parts of the process
  // state. Therefore we need a specific order of restoration for each mode, if
  // we also have ZA to restore.
  //
  // Streaming mode enabled, ZA enabled:
  // * Write streaming registers. This sets SVCR.SM and clears SVCR.ZA.
  // * Write ZA, this set SVCR.ZA. The register data we provide is written to
  // ZA.
  // * Result is SVCR.SM and SVCR.ZA set, with the expected data in both
  //   register sets.
  //
  // Streaming mode disabled, ZA enabled:
  // * Write ZA. This sets SVCR.ZA, and the ZA content. In the majority of cases
  //   the streaming vector length is changing, so the thread is converted into
  //   an FPSIMD thread if it is not already one. This also clears SVCR.SM.
  // * Write SVE registers, which also clears SVCR.SM but most importantly, puts
  //   us into full SVE mode instead of FPSIMD mode (where the registers are
  //   actually the 128 bit Neon registers).
  // * Result is we have SVCR.SM = 0, SVCR.ZA = 1 and the expected register
```
- **EN**: Implements logic around `CacheAllRegisters`, `Fail`, `reset`, `GetBytes`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `CacheAllRegisters`, `Fail`, `reset`, `GetBytes`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 981-999
```cpp
  //   state.
  //
  // Restoring in different orders leads to things like the SVE registers being
  // truncated due to the FPSIMD mode and ZA being disabled or filled with 0s
  // (disabled and 0s looks the same from inside lldb since we fake the value
  // when it's disabled).
  //
  // For more information on this, look up the uses of the relevant NT_ARM_
  // constants and the functions vec_set_vector_length, sve_set_common and
  // za_set in the Linux Kernel.

  if ((m_sve_state != SVEState::Streaming) && GetRegisterInfo().IsZAPresent()) {
    // Use the header size not the buffer size, as we may be using the buffer
    // for fake data, which we do not want to write out.
    assert(m_za_header.size <= GetZABufferSize());
    dst = AddSavedRegisters(dst, RegisterSetType::SME, GetZABuffer(),
                            m_za_header.size);
  }

```
- **EN**: Implements logic around `GetRegisterInfo`, `assert`, `AddSavedRegisters`.
- **CN**: 围绕 `GetRegisterInfo`, `assert`, `AddSavedRegisters` 实现具体逻辑。

### Lines 1000-1029
```cpp
  if ((GetRegisterInfo().IsSVEPresent() || GetRegisterInfo().IsSSVEPresent()) &&
      m_sve_state != SVEState::StreamingFPSIMD) {
    dst = AddRegisterSetType(dst, RegisterSetType::SVE);
    *(reinterpret_cast<SVEState *>(dst)) = m_sve_state;
    dst += sizeof(m_sve_state);
    dst = AddSavedRegistersData(dst, GetSVEBuffer(), GetSVEBufferSize());
  } else {
    dst = AddSavedRegisters(dst, RegisterSetType::FPR, GetFPRBuffer(),
                            GetFPRSize());
  }

  if ((m_sve_state == SVEState::Streaming) && GetRegisterInfo().IsZAPresent()) {
    assert(m_za_header.size <= GetZABufferSize());
    dst = AddSavedRegisters(dst, RegisterSetType::SME, GetZABuffer(),
                            m_za_header.size);
  }

  // If ZT0 is present and we are going to be restoring an active ZA (which
  // implies an active ZT0), then restore ZT0 after ZA has been set. This
  // prevents us enabling ZA accidentally after the restore of ZA disabled it.
  // If we leave ZA/ZT0 inactive and read ZT0, the kernel returns 0s. Therefore
  // there's nothing for us to restore if ZA was originally inactive.
  if (
      // If we have SME2 and therefore ZT0.
      GetRegisterInfo().IsZTPresent() &&
      // And ZA is enabled.
      m_za_header.size > sizeof(m_za_header))
    dst = AddSavedRegisters(dst, RegisterSetType::SME2, GetZTBuffer(),
                            GetZTBufferSize());

```
- **EN**: Implements logic around `GetRegisterInfo`, `AddRegisterSetType`, `AddSavedRegistersData`, `AddSavedRegisters`, and 3 more symbols.
- **CN**: 围绕 `GetRegisterInfo`, `AddRegisterSetType`, `AddSavedRegistersData`, `AddSavedRegisters`, and 3 more symbols 实现具体逻辑。

### Lines 1030-1049
```cpp
  if (GetRegisterInfo().IsMTEPresent()) {
    dst = AddSavedRegisters(dst, RegisterSetType::MTE, GetMTEControl(),
                            GetMTEControlSize());
  }

  if (GetRegisterInfo().IsFPMRPresent()) {
    dst = AddSavedRegisters(dst, RegisterSetType::FPMR, GetFPMRBuffer(),
                            GetFPMRBufferSize());
  }

  if (GetRegisterInfo().IsGCSPresent()) {
    dst = AddSavedRegisters(dst, RegisterSetType::GCS, GetGCSBuffer(),
                            GetGCSBufferSize());
  }

  if (GetRegisterInfo().IsPOEPresent()) {
    dst = AddSavedRegisters(dst, RegisterSetType::POE, GetPOEBuffer(),
                            GetPOEBufferSize());
  }

```
- **EN**: Implements logic around `GetRegisterInfo`, `AddSavedRegisters`, `GetMTEControlSize`, `GetFPMRBufferSize`, and 2 more symbols.
- **CN**: 围绕 `GetRegisterInfo`, `AddSavedRegisters`, `GetMTEControlSize`, `GetFPMRBufferSize`, and 2 more symbols 实现具体逻辑。

### Lines 1050-1069
```cpp
  dst = AddSavedRegisters(dst, RegisterSetType::TLS, GetTLSBuffer(),
                          GetTLSBufferSize());

  return error;
}

static Status RestoreRegisters(void *buffer, const uint8_t **src, size_t len,
                               bool &is_valid, std::function<Status()> writer) {
  ::memcpy(buffer, *src, len);
  is_valid = true;
  *src += len;
  return writer();
}

Status NativeRegisterContextLinux_arm64::WriteAllRegisterValues(
    const lldb::DataBufferSP &data_sp) {
  // AArch64 register data must contain GPRs, either FPR or SVE registers
  // (which can be streaming or non-streaming) and optional MTE register.
  // Pointer Authentication (PAC) registers are read-only and will be skipped.

```
- **EN**: Implements logic around `AddSavedRegisters`, `GetTLSBufferSize`, `RestoreRegisters`, `function`, and 3 more symbols; this block propagates recoverable errors, status objects, or diagnostics; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `AddSavedRegisters`, `GetTLSBufferSize`, `RestoreRegisters`, `function`, and 3 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 1070-1094
```cpp
  // We store all register values in data_sp by copying full PTrace data that
  // corresponds to register sets enabled by current register context. In order
  // to restore from register data checkpoint we will first restore GPRs, based
  // on size of remaining register data either SVE or FPRs should be restored
  // next. SVE is not enabled if we have register data size less than or equal
  // to size of GPR + FPR + MTE.

  Status error;
  if (!data_sp) {
    error = Status::FromErrorStringWithFormat(
        "NativeRegisterContextLinux_arm64::%s invalid data_sp provided",
        __FUNCTION__);
    return error;
  }

  const uint8_t *src = data_sp->GetBytes();
  if (src == nullptr) {
    error = Status::FromErrorStringWithFormat(
        "NativeRegisterContextLinux_arm64::%s "
        "DataBuffer::GetBytes() returned a null "
        "pointer",
        __FUNCTION__);
    return error;
  }

```
- **EN**: Implements logic around `FromErrorStringWithFormat`, `GetBytes`; this block propagates recoverable errors, status objects, or diagnostics; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `FromErrorStringWithFormat`, `GetBytes` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 1095-1121
```cpp
  uint64_t reg_data_min_size =
      GetGPRBufferSize() + GetFPRSize() + 2 * (sizeof(RegisterSetType));
  if (data_sp->GetByteSize() < reg_data_min_size) {
    error = Status::FromErrorStringWithFormat(
        "NativeRegisterContextLinux_arm64::%s data_sp contained insufficient "
        "register data bytes, expected at least %" PRIu64 ", actual %" PRIu64,
        __FUNCTION__, reg_data_min_size, data_sp->GetByteSize());
    return error;
  }

  const uint8_t *end = src + data_sp->GetByteSize();
  while (src < end) {
    const RegisterSetType kind =
        *reinterpret_cast<const RegisterSetType *>(src);
    src += sizeof(RegisterSetType);

    switch (kind) {
    case RegisterSetType::GPR:
      error = RestoreRegisters(
          GetGPRBuffer(), &src, GetGPRBufferSize(), m_gpr_is_valid,
          std::bind(&NativeRegisterContextLinux_arm64::WriteGPR, this));
      break;
    case RegisterSetType::SVE:
      // Restore to the correct mode, streaming or not.
      m_sve_state = static_cast<SVEState>(*src);
      src += sizeof(m_sve_state);

```
- **EN**: Implements logic around `GetGPRBufferSize`, `GetByteSize`, `FromErrorStringWithFormat`, `RestoreRegisters`, and 3 more symbols; this block propagates recoverable errors, status objects, or diagnostics; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `GetGPRBufferSize`, `GetByteSize`, `FromErrorStringWithFormat`, `RestoreRegisters`, and 3 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 1122-1142
```cpp
      // First write SVE header. We do not use RestoreRegisters because we do
      // not want src to be modified yet.
      ::memcpy(GetSVEHeader(), src, GetSVEHeaderSize());
      if (!sve::vl_valid(m_sve_header.vl)) {
        m_sve_header_is_valid = false;
        error = Status::FromErrorStringWithFormat(
            "NativeRegisterContextLinux_arm64::%s "
            "Invalid SVE header in data_sp",
            __FUNCTION__);
        return error;
      }
      m_sve_header_is_valid = true;
      error = WriteSVEHeader();
      if (error.Fail())
        return error;

      // SVE header has been written configure SVE vector length if needed.
      // This could change ZA data too, but that will be restored again later
      // anyway.
      ConfigureRegisterContext();

```
- **EN**: Implements logic around `memcpy`, `vl_valid`, `FromErrorStringWithFormat`, `WriteSVEHeader`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `memcpy`, `vl_valid`, `FromErrorStringWithFormat`, `WriteSVEHeader`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 1143-1169
```cpp
      // Write header and register data, incrementing src this time.
      error = RestoreRegisters(
          GetSVEBuffer(), &src, GetSVEBufferSize(), m_sve_buffer_is_valid,
          std::bind(&NativeRegisterContextLinux_arm64::WriteAllSVE, this));
      break;
    case RegisterSetType::FPR: {
      m_sve_buffer_is_valid = false;
      m_sve_header_is_valid = false;
      m_sve_state = SVEState::Unknown;
      ConfigureRegisterContext();

      // If we are on an SME only system and currently in streaming mode, about
      // to restore non-streaming FP data.
      if (!GetRegisterInfo().IsSVEPresent() &&
          GetRegisterInfo().IsSSVEPresent() &&
          m_sve_state == SVEState::Streaming) {
        // We can only restore this data on kernel versions >= 6.19, so
        // attempt it and if it fails, we will skip restoring the data.
        //
        // To attempt the restore we write FPSIMD format data to NT_ARM_SVE,
        // with the vector length set to 0. If supported, this will switch
        // modes from streaming to non-streaming and update the FP registers
        // with the values we provided.
        //
        // This interface is only used by LLDB in this one specific
        // circumstance.

```
- **EN**: Implements logic around `RestoreRegisters`, `GetSVEBuffer`, `bind`, `ConfigureRegisterContext`, and 1 more symbols; this block propagates recoverable errors, status objects, or diagnostics; controls debugger-side formatting or synthetic presentation of values; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `RestoreRegisters`, `GetSVEBuffer`, `bind`, `ConfigureRegisterContext`, and 1 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并控制调试器侧的值格式化或 synthetic 展示，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 1170-1187
```cpp
        size_t data_size = sve::ptrace_fpsimd_offset + GetFPRSize();
        // NT_ARM_SVE data must be a multiple of 128 bits, and the FPU data size
        // is not, round up.
        data_size =
            (data_size + sve::vq_bytes - 1) / sve::vq_bytes * sve::vq_bytes;
        std::vector<uint8_t> sve_fpsimd_data(data_size);

        sve::user_sve_header *header =
            reinterpret_cast<sve::user_sve_header *>(sve_fpsimd_data.data());
        std::memset(header, 0, sizeof(sve::user_sve_header));
        header->size = sve_fpsimd_data.size();
        // VL = 0 tells the process to exit streaming mode.
        header->vl = 0;
        header->flags = sve::ptrace_regs_fpsimd;
        std::memcpy(&sve_fpsimd_data[sve::ptrace_fpsimd_offset], src,
                    GetFPRSize());

        struct iovec ioVec;
```
- **EN**: Introduces declarations for `iovec`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `iovec` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 1188-1223
```cpp
        ioVec.iov_base = sve_fpsimd_data.data();
        ioVec.iov_len = sve_fpsimd_data.size();

        // Even though the system does not have SVE, NT_ARM_SVE is used when
        // exiting streaming mode.
        error = WriteRegisterSet(&ioVec, sve_fpsimd_data.size(), NT_ARM_SVE);

        // Consume FP register set.
        src += GetFPRSize();

        if (error.Success()) {
          // Wrote FPU, and SVE overlaps FPU.
          m_fpu_is_valid = false;
          m_sve_buffer_is_valid = false;
          m_sve_header_is_valid = false;

          m_sve_state = SVEState::Unknown;
          ConfigureRegisterContext();
        }
        // Else we failed to restore these registers, but we will try to restore
        // the others.
      } else {
        error = RestoreRegisters(
            GetFPRBuffer(), &src, GetFPRSize(), m_fpu_is_valid,
            std::bind(&NativeRegisterContextLinux_arm64::WriteFPR, this));
      }
      break;
    }
    case RegisterSetType::MTE:
      error = RestoreRegisters(
          GetMTEControl(), &src, GetMTEControlSize(), m_mte_ctrl_is_valid,
          std::bind(&NativeRegisterContextLinux_arm64::WriteMTEControl, this));
      break;
    case RegisterSetType::TLS:
      error = RestoreRegisters(
          GetTLSBuffer(), &src, GetTLSBufferSize(), m_tls_is_valid,
```
- **EN**: Implements logic around `data`, `size`, `WriteRegisterSet`, `GetFPRSize`, and 7 more symbols; this block propagates recoverable errors, status objects, or diagnostics; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `data`, `size`, `WriteRegisterSet`, `GetFPRSize`, and 7 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 1224-1243
```cpp
          std::bind(&NativeRegisterContextLinux_arm64::WriteTLS, this));
      break;
    case RegisterSetType::SME:
      // To enable or disable ZA you write the regset with or without register
      // data. The kernel detects this by looking at the ioVec's length, not the
      // ZA header size you pass in. Therefore we must write header and register
      // data (if present) in one go every time. Read the header only first just
      // to get the size.
      ::memcpy(GetZAHeader(), src, GetZAHeaderSize());
      // Read the header and register data. Can't use the buffer size here, it
      // may be incorrect due to being filled with dummy data previously. Resize
      // this so WriteZA uses the correct size.
      m_za_ptrace_payload.resize(m_za_header.size);
      ::memcpy(GetZABuffer(), src, GetZABufferSize());
      m_za_buffer_is_valid = true;

      error = WriteZA();
      if (error.Fail())
        return error;

```
- **EN**: Implements logic around `bind`, `memcpy`, `resize`, `WriteZA`, and 1 more symbols; this block propagates recoverable errors, status objects, or diagnostics; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `bind`, `memcpy`, `resize`, `WriteZA`, and 1 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 1244-1275
```cpp
      // Update size of ZA, which resizes the ptrace payload potentially
      // trashing our copy of the data we just wrote.
      ConfigureRegisterContext();

      // ZA buffer now has proper size, read back the data we wrote above, from
      // ptrace.
      error = ReadZA();
      src += GetZABufferSize();
      break;
    case RegisterSetType::SME2:
      // Doing this would activate an inactive ZA, however we will only get here
      // if the state we are restoring had an active ZA. Restoring ZT0 will
      // always come after restoring ZA.
      error = RestoreRegisters(
          GetZTBuffer(), &src, GetZTBufferSize(), m_zt_buffer_is_valid,
          std::bind(&NativeRegisterContextLinux_arm64::WriteZT, this));
      break;
    case RegisterSetType::FPMR:
      error = RestoreRegisters(
          GetFPMRBuffer(), &src, GetFPMRBufferSize(), m_fpmr_is_valid,
          std::bind(&NativeRegisterContextLinux_arm64::WriteFPMR, this));
      break;
    case RegisterSetType::GCS: {
      // It is not permitted to enable GCS via ptrace. We can disable it, but
      // to keep things simple we will not revert any change to the
      // PR_SHADOW_STACK_ENABLE bit. Instead patch in the current enable bit
      // into the registers we are about to restore.
      m_gcs_is_valid = false;
      error = ReadGCS();
      if (error.Fail())
        return error;

```
- **EN**: Implements logic around `ConfigureRegisterContext`, `ReadZA`, `GetZABufferSize`, `RestoreRegisters`, and 5 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `ConfigureRegisterContext`, `ReadZA`, `GetZABufferSize`, `RestoreRegisters`, and 5 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 1276-1296
```cpp
      uint64_t enable_bit = m_gcs_regs.features_enabled & 1UL;
      gcs_regs new_gcs_regs = *reinterpret_cast<const gcs_regs *>(src);
      new_gcs_regs.features_enabled =
          (new_gcs_regs.features_enabled & ~1UL) | enable_bit;

      const uint8_t *new_gcs_src =
          reinterpret_cast<const uint8_t *>(&new_gcs_regs);
      error = RestoreRegisters(
          GetGCSBuffer(), &new_gcs_src, GetGCSBufferSize(), m_gcs_is_valid,
          std::bind(&NativeRegisterContextLinux_arm64::WriteGCS, this));
      src += GetGCSBufferSize();

      break;
    }
    case RegisterSetType::POE:
      error = RestoreRegisters(
          GetPOEBuffer(), &src, GetPOEBufferSize(), m_poe_is_valid,
          std::bind(&NativeRegisterContextLinux_arm64::WritePOE, this));
      break;
    }

```
- **EN**: Implements logic around `RestoreRegisters`, `GetGCSBuffer`, `bind`, `GetGCSBufferSize`, and 1 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `RestoreRegisters`, `GetGCSBuffer`, `bind`, `GetGCSBufferSize`, and 1 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 1297-1315
```cpp
    if (error.Fail())
      return error;
  }

  return error;
}

llvm::Error NativeRegisterContextLinux_arm64::ReadHardwareDebugInfo() {
  if (!m_refresh_hwdebug_info) {
    return llvm::Error::success();
  }

  ::pid_t tid = m_thread.GetID();

  Status error = arm64::ReadHardwareDebugInfo(tid, m_max_hwp_supported,
                                              m_max_hbp_supported);
  if (error.Fail())
    return error.ToError();

```
- **EN**: Implements logic around `Fail`, `ReadHardwareDebugInfo`, `success`, `GetID`, and 1 more symbols; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `Fail`, `ReadHardwareDebugInfo`, `success`, `GetID`, and 1 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 1316-1333
```cpp
  m_refresh_hwdebug_info = false;

  return llvm::Error::success();
}

llvm::Error
NativeRegisterContextLinux_arm64::WriteHardwareDebugRegs(DREGType hwbType) {
  uint32_t max_supported =
      (hwbType == eDREGTypeWATCH) ? m_max_hwp_supported : m_max_hbp_supported;
  auto &regs = (hwbType == eDREGTypeWATCH) ? m_hwp_regs : m_hbp_regs;
  return arm64::WriteHardwareDebugRegs(hwbType, m_thread.GetID(), max_supported,
                                       regs)
      .ToError();
}

Status NativeRegisterContextLinux_arm64::ReadGPR() {
  Status error;

```
- **EN**: Implements logic around `success`, `WriteHardwareDebugRegs`, `ToError`, `ReadGPR`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `success`, `WriteHardwareDebugRegs`, `ToError`, `ReadGPR` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 1334-1353
```cpp
  if (m_gpr_is_valid)
    return error;

  struct iovec ioVec;
  ioVec.iov_base = GetGPRBuffer();
  ioVec.iov_len = GetGPRBufferSize();

  error = ReadRegisterSet(&ioVec, GetGPRBufferSize(), NT_PRSTATUS);

  if (error.Success())
    m_gpr_is_valid = true;

  return error;
}

Status NativeRegisterContextLinux_arm64::WriteGPR() {
  Status error = ReadGPR();
  if (error.Fail())
    return error;

```
- **EN**: Introduces declarations for `iovec`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `iovec` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 1354-1372
```cpp
  struct iovec ioVec;
  ioVec.iov_base = GetGPRBuffer();
  ioVec.iov_len = GetGPRBufferSize();

  m_gpr_is_valid = false;

  return WriteRegisterSet(&ioVec, GetGPRBufferSize(), NT_PRSTATUS);
}

Status NativeRegisterContextLinux_arm64::ReadFPR() {
  Status error;

  if (m_fpu_is_valid)
    return error;

  struct iovec ioVec;
  ioVec.iov_base = GetFPRBuffer();
  ioVec.iov_len = GetFPRSize();

```
- **EN**: Introduces declarations for `iovec`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `iovec` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 1373-1393
```cpp
  error = ReadRegisterSet(&ioVec, GetFPRSize(), NT_FPREGSET);
  if (error.Success())
    m_fpu_is_valid = true;

  return error;
}

Status NativeRegisterContextLinux_arm64::WriteFPR() {
  Status error = ReadFPR();
  if (error.Fail())
    return error;

  struct iovec ioVec;
  ioVec.iov_base = GetFPRBuffer();
  ioVec.iov_len = GetFPRSize();

  m_fpu_is_valid = false;
  // SVE Z registers overlap the FP registers.
  m_sve_buffer_is_valid = false;
  m_sve_header_is_valid = false;

```
- **EN**: Introduces declarations for `iovec`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `iovec` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 1394-1411
```cpp
  return WriteRegisterSet(&ioVec, GetFPRSize(), NT_FPREGSET);
}

void NativeRegisterContextLinux_arm64::InvalidateAllRegisters() {
  m_gpr_is_valid = false;
  m_fpu_is_valid = false;
  m_sve_buffer_is_valid = false;
  m_sve_header_is_valid = false;
  m_za_buffer_is_valid = false;
  m_za_header_is_valid = false;
  m_pac_mask_is_valid = false;
  m_mte_ctrl_is_valid = false;
  m_tls_is_valid = false;
  m_zt_buffer_is_valid = false;
  m_fpmr_is_valid = false;
  m_gcs_is_valid = false;
  m_poe_is_valid = false;

```
- **EN**: Implements logic around `WriteRegisterSet`, `InvalidateAllRegisters`.
- **CN**: 围绕 `WriteRegisterSet`, `InvalidateAllRegisters` 实现具体逻辑。

### Lines 1412-1431
```cpp
  // Update SVE and ZA registers in case there is change in configuration.
  ConfigureRegisterContext();
}

unsigned NativeRegisterContextLinux_arm64::GetSVERegSet() {
  switch (m_sve_state) {
  case SVEState::Streaming:
  case SVEState::StreamingFPSIMD:
    return NT_ARM_SSVE;
  default:
    return NT_ARM_SVE;
  }
}

Status NativeRegisterContextLinux_arm64::ReadSVEHeader() {
  Status error;

  if (m_sve_header_is_valid)
    return error;

```
- **EN**: Implements logic around `ConfigureRegisterContext`, `GetSVERegSet`, `ReadSVEHeader`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `ConfigureRegisterContext`, `GetSVERegSet`, `ReadSVEHeader` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 1432-1449
```cpp
  struct iovec ioVec;
  ioVec.iov_base = GetSVEHeader();
  ioVec.iov_len = GetSVEHeaderSize();

  error = ReadRegisterSet(&ioVec, GetSVEHeaderSize(), GetSVERegSet());

  if (error.Success())
    m_sve_header_is_valid = true;

  return error;
}

Status NativeRegisterContextLinux_arm64::ReadPAuthMask() {
  Status error;

  if (m_pac_mask_is_valid)
    return error;

```
- **EN**: Introduces declarations for `iovec`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `iovec` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 1450-1468
```cpp
  struct iovec ioVec;
  ioVec.iov_base = GetPACMask();
  ioVec.iov_len = GetPACMaskSize();

  error = ReadRegisterSet(&ioVec, GetPACMaskSize(), NT_ARM_PAC_MASK);

  if (error.Success())
    m_pac_mask_is_valid = true;

  return error;
}

Status NativeRegisterContextLinux_arm64::WriteSVEHeader() {
  Status error;

  error = ReadSVEHeader();
  if (error.Fail())
    return error;

```
- **EN**: Introduces declarations for `iovec`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `iovec` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 1469-1488
```cpp
  struct iovec ioVec;
  ioVec.iov_base = GetSVEHeader();
  ioVec.iov_len = GetSVEHeaderSize();

  m_sve_buffer_is_valid = false;
  m_sve_header_is_valid = false;
  m_fpu_is_valid = false;

  return WriteRegisterSet(&ioVec, GetSVEHeaderSize(), GetSVERegSet());
}

Status NativeRegisterContextLinux_arm64::ReadAllSVE() {
  Status error;
  if (m_sve_buffer_is_valid)
    return error;

  struct iovec ioVec;
  ioVec.iov_base = GetSVEBuffer();
  ioVec.iov_len = GetSVEBufferSize();

```
- **EN**: Introduces declarations for `iovec`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `iovec` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 1489-1508
```cpp
  error = ReadRegisterSet(&ioVec, GetSVEBufferSize(), GetSVERegSet());

  if (error.Success())
    m_sve_buffer_is_valid = true;

  return error;
}

Status NativeRegisterContextLinux_arm64::WriteAllSVE() {
  Status error;

  error = ReadAllSVE();
  if (error.Fail())
    return error;

  struct iovec ioVec;

  ioVec.iov_base = GetSVEBuffer();
  ioVec.iov_len = GetSVEBufferSize();

```
- **EN**: Introduces declarations for `iovec`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `iovec` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 1509-1530
```cpp
  m_sve_buffer_is_valid = false;
  m_sve_header_is_valid = false;
  m_fpu_is_valid = false;

  return WriteRegisterSet(&ioVec, GetSVEBufferSize(), GetSVERegSet());
}

Status NativeRegisterContextLinux_arm64::ReadSMEControl() {
  // The real register is SVCR and is accessible from EL0. However we don't want
  // to have to JIT code into the target process so we'll just recreate it using
  // what we know from ptrace.

  // Bit 0 indicates whether streaming mode is active.
  m_sme_pseudo_regs.ctrl_reg = m_sve_state == SVEState::Streaming;

  // Bit 1 indicates whether the array storage is active.
  // It is active if we can read the header and the size field tells us that
  // there is register data following it.
  Status error = ReadZAHeader();
  if (error.Success() && (m_za_header.size > sizeof(m_za_header)))
    m_sme_pseudo_regs.ctrl_reg |= 2;

```
- **EN**: Implements logic around `WriteRegisterSet`, `ReadSMEControl`, `ReadZAHeader`, `Success`; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow; supports expression parsing, wrapping, or debug-time code generation; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `WriteRegisterSet`, `ReadSMEControl`, `ReadZAHeader`, `Success` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流，并支持表达式解析、包装或调试期代码生成，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 1531-1548
```cpp
  return error;
}

Status NativeRegisterContextLinux_arm64::ReadMTEControl() {
  Status error;

  if (m_mte_ctrl_is_valid)
    return error;

  struct iovec ioVec;
  ioVec.iov_base = GetMTEControl();
  ioVec.iov_len = GetMTEControlSize();

  error = ReadRegisterSet(&ioVec, GetMTEControlSize(), NT_ARM_TAGGED_ADDR_CTRL);

  if (error.Success())
    m_mte_ctrl_is_valid = true;

```
- **EN**: Introduces declarations for `iovec`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `iovec` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 1549-1567
```cpp
  return error;
}

Status NativeRegisterContextLinux_arm64::WriteMTEControl() {
  Status error;

  error = ReadMTEControl();
  if (error.Fail())
    return error;

  struct iovec ioVec;
  ioVec.iov_base = GetMTEControl();
  ioVec.iov_len = GetMTEControlSize();

  m_mte_ctrl_is_valid = false;

  return WriteRegisterSet(&ioVec, GetMTEControlSize(), NT_ARM_TAGGED_ADDR_CTRL);
}

```
- **EN**: Introduces declarations for `iovec`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `iovec` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 1568-1585
```cpp
Status NativeRegisterContextLinux_arm64::ReadTLS() {
  Status error;

  if (m_tls_is_valid)
    return error;

  struct iovec ioVec;
  ioVec.iov_base = GetTLSBuffer();
  ioVec.iov_len = GetTLSBufferSize();

  error = ReadRegisterSet(&ioVec, GetTLSBufferSize(), NT_ARM_TLS);

  if (error.Success())
    m_tls_is_valid = true;

  return error;
}

```
- **EN**: Introduces declarations for `iovec`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `iovec` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 1586-1604
```cpp
Status NativeRegisterContextLinux_arm64::WriteTLS() {
  Status error;

  error = ReadTLS();
  if (error.Fail())
    return error;

  struct iovec ioVec;
  ioVec.iov_base = GetTLSBuffer();
  ioVec.iov_len = GetTLSBufferSize();

  m_tls_is_valid = false;

  return WriteRegisterSet(&ioVec, GetTLSBufferSize(), NT_ARM_TLS);
}

Status NativeRegisterContextLinux_arm64::ReadGCS() {
  Status error;

```
- **EN**: Introduces declarations for `iovec`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `iovec` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 1605-1622
```cpp
  if (m_gcs_is_valid)
    return error;

  struct iovec ioVec;
  ioVec.iov_base = GetGCSBuffer();
  ioVec.iov_len = GetGCSBufferSize();

  error = ReadRegisterSet(&ioVec, GetGCSBufferSize(), NT_ARM_GCS);

  if (error.Success())
    m_gcs_is_valid = true;

  return error;
}

Status NativeRegisterContextLinux_arm64::WriteGCS() {
  Status error;

```
- **EN**: Introduces declarations for `iovec`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `iovec` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 1623-1641
```cpp
  error = ReadGCS();
  if (error.Fail())
    return error;

  struct iovec ioVec;
  ioVec.iov_base = GetGCSBuffer();
  ioVec.iov_len = GetGCSBufferSize();

  m_gcs_is_valid = false;

  return WriteRegisterSet(&ioVec, GetGCSBufferSize(), NT_ARM_GCS);
}

Status NativeRegisterContextLinux_arm64::ReadZAHeader() {
  Status error;

  if (m_za_header_is_valid)
    return error;

```
- **EN**: Introduces declarations for `iovec`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `iovec` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 1642-1659
```cpp
  struct iovec ioVec;
  ioVec.iov_base = GetZAHeader();
  ioVec.iov_len = GetZAHeaderSize();

  error = ReadRegisterSet(&ioVec, GetZAHeaderSize(), NT_ARM_ZA);

  if (error.Success())
    m_za_header_is_valid = true;

  return error;
}

Status NativeRegisterContextLinux_arm64::ReadZA() {
  Status error;

  if (m_za_buffer_is_valid)
    return error;

```
- **EN**: Introduces declarations for `iovec`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `iovec` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 1660-1677
```cpp
  struct iovec ioVec;
  ioVec.iov_base = GetZABuffer();
  ioVec.iov_len = GetZABufferSize();

  error = ReadRegisterSet(&ioVec, GetZABufferSize(), NT_ARM_ZA);

  if (error.Success())
    m_za_buffer_is_valid = true;

  return error;
}

Status NativeRegisterContextLinux_arm64::WriteZA() {
  // Note that because the ZA ptrace payload contains the header also, this
  // method will write both. This is done because writing only the header
  // will disable ZA, even if .size in the header is correct for an enabled ZA.
  Status error;

```
- **EN**: Introduces declarations for `iovec`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `iovec` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 1678-1696
```cpp
  error = ReadZA();
  if (error.Fail())
    return error;

  struct iovec ioVec;
  ioVec.iov_base = GetZABuffer();
  ioVec.iov_len = GetZABufferSize();

  m_za_buffer_is_valid = false;
  m_za_header_is_valid = false;
  // Writing to ZA may enable ZA, which means ZT0 may change too.
  m_zt_buffer_is_valid = false;

  return WriteRegisterSet(&ioVec, GetZABufferSize(), NT_ARM_ZA);
}

Status NativeRegisterContextLinux_arm64::ReadZT() {
  Status error;

```
- **EN**: Introduces declarations for `iovec`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `iovec` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 1697-1716
```cpp
  if (m_zt_buffer_is_valid)
    return error;

  struct iovec ioVec;
  ioVec.iov_base = GetZTBuffer();
  ioVec.iov_len = GetZTBufferSize();

  error = ReadRegisterSet(&ioVec, GetZTBufferSize(), NT_ARM_ZT);
  m_zt_buffer_is_valid = error.Success();

  return error;
}

Status NativeRegisterContextLinux_arm64::WriteZT() {
  Status error;

  error = ReadZT();
  if (error.Fail())
    return error;

```
- **EN**: Introduces declarations for `iovec`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `iovec` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 1717-1735
```cpp
  struct iovec ioVec;
  ioVec.iov_base = GetZTBuffer();
  ioVec.iov_len = GetZTBufferSize();

  m_zt_buffer_is_valid = false;
  // Writing to an inactive ZT0 will enable ZA as well, which invalidates our
  // current copy of it.
  m_za_buffer_is_valid = false;
  m_za_header_is_valid = false;

  return WriteRegisterSet(&ioVec, GetZTBufferSize(), NT_ARM_ZT);
}

Status NativeRegisterContextLinux_arm64::ReadFPMR() {
  Status error;

  if (m_fpmr_is_valid)
    return error;

```
- **EN**: Introduces declarations for `iovec`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `iovec` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 1736-1754
```cpp
  struct iovec ioVec;
  ioVec.iov_base = GetFPMRBuffer();
  ioVec.iov_len = GetFPMRBufferSize();

  error = ReadRegisterSet(&ioVec, GetFPMRBufferSize(), NT_ARM_FPMR);

  if (error.Success())
    m_fpmr_is_valid = true;

  return error;
}

Status NativeRegisterContextLinux_arm64::WriteFPMR() {
  Status error;

  error = ReadFPMR();
  if (error.Fail())
    return error;

```
- **EN**: Introduces declarations for `iovec`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `iovec` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 1755-1773
```cpp
  struct iovec ioVec;
  ioVec.iov_base = GetFPMRBuffer();
  ioVec.iov_len = GetFPMRBufferSize();

  m_fpmr_is_valid = false;

  return WriteRegisterSet(&ioVec, GetFPMRBufferSize(), NT_ARM_FPMR);
}

Status NativeRegisterContextLinux_arm64::ReadPOE() {
  Status error;

  if (m_poe_is_valid)
    return error;

  struct iovec ioVec;
  ioVec.iov_base = GetPOEBuffer();
  ioVec.iov_len = GetPOEBufferSize();

```
- **EN**: Introduces declarations for `iovec`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `iovec` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 1774-1792
```cpp
  error = ReadRegisterSet(&ioVec, GetPOEBufferSize(), NT_ARM_POE);

  if (error.Success())
    m_poe_is_valid = true;

  return error;
}

Status NativeRegisterContextLinux_arm64::WritePOE() {
  Status error;

  error = ReadPOE();
  if (error.Fail())
    return error;

  struct iovec ioVec;
  ioVec.iov_base = GetPOEBuffer();
  ioVec.iov_len = GetPOEBufferSize();

```
- **EN**: Introduces declarations for `iovec`, establishing the types or namespaces used later in the file.
- **CN**: 引入 `iovec` 等声明，建立本文件后续使用的类型或命名空间。

### Lines 1793-1814
```cpp
  m_poe_is_valid = false;

  return WriteRegisterSet(&ioVec, GetPOEBufferSize(), NT_ARM_POE);
}

void NativeRegisterContextLinux_arm64::ConfigureRegisterContext() {
  // ConfigureRegisterContext gets called from InvalidateAllRegisters
  // on every stop and configures SVE vector length and whether we are in
  // streaming SVE mode.
  // If m_sve_state is set to SVEState::Disabled on first stop, code below will
  // be deemed non operational for the lifetime of current process.
  if (!m_sve_header_is_valid && m_sve_state != SVEState::Disabled) {
    // Systems may have SVE and/or SME. If they are SME only, the SVE regset
    // cannot be read from but the SME one can. If they have both SVE and SME,
    // only the active mode will return valid register data.

    // Check for SME.
    m_sve_header_is_valid = false;
    m_sve_buffer_is_valid = false;
    m_sve_state = SVEState::Streaming;
    Status error = ReadSVEHeader();

```
- **EN**: Implements logic around `WriteRegisterSet`, `ConfigureRegisterContext`, `ReadSVEHeader`; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `WriteRegisterSet`, `ConfigureRegisterContext`, `ReadSVEHeader` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 1815-1835
```cpp
    bool has_sme = error.Success();
    bool sme_is_active =
        has_sme &&
        ((m_sve_header.flags & sve::ptrace_regs_mask) == sve::ptrace_regs_sve);

    // Check for SVE.
    m_sve_header_is_valid = false;
    m_sve_buffer_is_valid = false;
    m_sve_state = SVEState::Full;
    error = ReadSVEHeader();

    bool has_sve = error.Success();
    bool sve_is_active =
        has_sve &&
        ((m_sve_header.flags & sve::ptrace_regs_mask) == sve::ptrace_regs_sve);
    // We do not check this for streaming mode because the streaming mode regset
    // will never be in FP format.
    bool fp_is_active =
        has_sve && ((m_sve_header.flags & sve::ptrace_regs_mask) ==
                    sve::ptrace_regs_fpsimd);

```
- **EN**: Implements logic around `Success`, `ReadSVEHeader`; this block propagates recoverable errors, status objects, or diagnostics; controls debugger-side formatting or synthetic presentation of values.
- **CN**: 围绕 `Success`, `ReadSVEHeader` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并控制调试器侧的值格式化或 synthetic 展示。

### Lines 1836-1855
```cpp
    if (sme_is_active)
      m_sve_state = SVEState::Streaming;
    else if (sve_is_active)
      m_sve_state = SVEState::Full;
    else if (fp_is_active)
      m_sve_state = SVEState::FPSIMD;
    else if (has_sme) {
      // We are in the non-streaming mode of an SME only system.
      m_sve_state = SVEState::StreamingFPSIMD;
    } else
      m_sve_state = SVEState::Disabled;

    if (m_sve_state == SVEState::Full || m_sve_state == SVEState::FPSIMD ||
        m_sve_state == SVEState::Streaming ||
        m_sve_state == SVEState::StreamingFPSIMD) {

      m_sve_header_is_valid = false;
      m_sve_buffer_is_valid = false;
      error = ReadSVEHeader();

```
- **EN**: Implements logic around `ReadSVEHeader`; this block propagates recoverable errors, status objects, or diagnostics.
- **CN**: 围绕 `ReadSVEHeader` 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息。

### Lines 1856-1873
```cpp
      // On every stop we configure SVE vector length by calling
      // ConfigureVectorLengthSVE regardless of current SVEState of this thread.
      uint32_t vq = RegisterInfoPOSIX_arm64::eVectorQuadwordAArch64SVE;
      if (sve::vl_valid(m_sve_header.vl))
        vq = sve::vq_from_vl(m_sve_header.vl);

      GetRegisterInfo().ConfigureVectorLengthSVE(vq);
      m_sve_ptrace_payload.resize(sve::PTraceSize(vq, sve::ptrace_regs_sve));
    }
  }

  if (!m_za_header_is_valid) {
    Status error = ReadZAHeader();
    if (error.Success()) {
      uint32_t vq = RegisterInfoPOSIX_arm64::eVectorQuadwordAArch64SVE;
      if (sve::vl_valid(m_za_header.vl))
        vq = sve::vq_from_vl(m_za_header.vl);

```
- **EN**: Implements logic around `vl_valid`, `vq_from_vl`, `GetRegisterInfo`, `resize`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics; coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `vl_valid`, `vq_from_vl`, `GetRegisterInfo`, `resize`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并协调调试器运行时对象、事件或异步控制流。

### Lines 1874-1904
```cpp
      GetRegisterInfo().ConfigureVectorLengthZA(vq);
      m_za_ptrace_payload.resize(m_za_header.size);
      m_za_buffer_is_valid = false;
    }
  }
}

uint32_t NativeRegisterContextLinux_arm64::CalculateFprOffset(
    const RegisterInfo *reg_info, bool streaming_fpsimd) const {
  uint32_t offset = reg_info->byte_offset - GetGPRSize();
  if (!streaming_fpsimd)
    return offset;

  // If we're outside of streaming mode on a streaming only target, the offsets
  // are relative to an SVE context. We need the offset into the actual FPR
  // context:
  // struct user_fpsimd_state {
  // 	__uint128_t	vregs[32];
  // 	__u32		fpsr;
  // 	__u32		fpcr;
  // 	__u32		__reserved[2];
  // };
  const size_t fpsr_offset = 16 * 32;
  const uint32_t reg = reg_info->kinds[lldb::eRegisterKindLLDB];
  if (reg == GetRegisterInfo().GetRegNumFPSR())
    offset = fpsr_offset;
  else if (reg == GetRegisterInfo().GetRegNumFPCR())
    offset = fpsr_offset + 4;
  else
    offset = 16 * (reg - GetRegisterInfo().GetRegNumFPV0());

```
- **EN**: Implements logic around `GetRegisterInfo`, `resize`, `CalculateFprOffset`, `GetGPRSize`; this block coordinates debugger runtime objects, events, or asynchronous control flow.
- **CN**: 围绕 `GetRegisterInfo`, `resize`, `CalculateFprOffset`, `GetGPRSize` 实现具体逻辑；该代码块协调调试器运行时对象、事件或异步控制流。

### Lines 1905-1925
```cpp
  return offset;
}

uint32_t NativeRegisterContextLinux_arm64::CalculateSVEOffset(
    const RegisterInfo *reg_info) const {
  // Start of Z0 data is after GPRs plus 8 bytes of vg register
  uint32_t sve_reg_offset = LLDB_INVALID_INDEX32;
  if (m_sve_state == SVEState::FPSIMD) {
    const uint32_t reg = reg_info->kinds[lldb::eRegisterKindLLDB];
    sve_reg_offset = sve::ptrace_fpsimd_offset +
                     (reg - GetRegisterInfo().GetRegNumSVEZ0()) * 16;
    // Between non-streaming and streaming mode, the layout is identical.
  } else if (m_sve_state == SVEState::Full ||
             m_sve_state == SVEState::Streaming) {
    uint32_t sve_z0_offset = GetGPRSize() + 16;
    sve_reg_offset =
        sve::SigRegsOffset() + reg_info->byte_offset - sve_z0_offset;
  }
  return sve_reg_offset;
}

```
- **EN**: Implements logic around `CalculateSVEOffset`, `GetRegisterInfo`, `GetGPRSize`, `SigRegsOffset`; this block models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `CalculateSVEOffset`, `GetRegisterInfo`, `GetGPRSize`, `SigRegsOffset` 实现具体逻辑；该代码块建模机器状态、栈检查或面向反汇编的元数据。

### Lines 1926-1947
```cpp
Status NativeRegisterContextLinux_arm64::ReadSMESVG() {
  // This register is the streaming vector length, so we will get it from
  // NT_ARM_ZA regardless of the current streaming mode.
  Status error = ReadZAHeader();
  if (error.Success())
    m_sme_pseudo_regs.svg_reg = m_za_header.vl / 8;

  return error;
}

std::vector<uint32_t> NativeRegisterContextLinux_arm64::GetExpeditedRegisters(
    ExpeditedRegs expType) const {
  std::vector<uint32_t> expedited_reg_nums =
      NativeRegisterContext::GetExpeditedRegisters(expType);
  // SVE, non-streaming vector length.
  if (m_sve_state == SVEState::FPSIMD || m_sve_state == SVEState::Full)
    expedited_reg_nums.push_back(GetRegisterInfo().GetRegNumSVEVG());
  // SME, streaming vector length. This is used by the ZA register which is
  // present even when streaming mode is not enabled.
  if (GetRegisterInfo().IsSSVEPresent())
    expedited_reg_nums.push_back(GetRegisterInfo().GetRegNumSMESVG());

```
- **EN**: Implements logic around `ReadSMESVG`, `ReadZAHeader`, `Success`, `GetExpeditedRegisters`, and 2 more symbols; this block propagates recoverable errors, status objects, or diagnostics; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `ReadSMESVG`, `ReadZAHeader`, `Success`, `GetExpeditedRegisters`, and 2 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 1948-1967
```cpp
  return expedited_reg_nums;
}

llvm::Expected<NativeRegisterContextLinux::MemoryTaggingDetails>
NativeRegisterContextLinux_arm64::GetMemoryTaggingDetails(int32_t type) {
  if (type == MemoryTagManagerAArch64MTE::eMTE_allocation) {
    return MemoryTaggingDetails{std::make_unique<MemoryTagManagerAArch64MTE>(),
                                PTRACE_PEEKMTETAGS, PTRACE_POKEMTETAGS};
  }

  return llvm::createStringError(llvm::inconvertibleErrorCode(),
                                 "Unknown AArch64 memory tag type %d", type);
}

lldb::addr_t NativeRegisterContextLinux_arm64::FixWatchpointHitAddress(
    lldb::addr_t hit_addr) {
  // Linux configures user-space virtual addresses with top byte ignored.
  // We set default value of mask such that top byte is masked out.
  lldb::addr_t mask = ~((1ULL << 56) - 1);

```
- **EN**: Implements logic around `GetMemoryTaggingDetails`, `make_unique`, `createStringError`, `FixWatchpointHitAddress`, and 1 more symbols; this block propagates recoverable errors, status objects, or diagnostics; models machine state, stack inspection, or disassembly-oriented metadata.
- **CN**: 围绕 `GetMemoryTaggingDetails`, `make_unique`, `createStringError`, `FixWatchpointHitAddress`, and 1 more symbols 实现具体逻辑；该代码块传播可恢复错误、状态对象或诊断信息，并建模机器状态、栈检查或面向反汇编的元数据。

### Lines 1968-1977
```cpp
  // Try to read pointer authentication data_mask register and calculate a
  // consolidated data address mask after ignoring the top byte.
  if (ReadPAuthMask().Success())
    mask |= m_pac_mask.data_mask;

  return hit_addr & ~mask;
  ;
}

#endif // defined (__arm64__) || defined (__aarch64__)
```
- **EN**: Defines preprocessor-controlled structure, feature gates, or include-guard state.
- **CN**: 定义受预处理器控制的结构、特性开关或头文件保护状态。

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
- **OS integration / 操作系统集成**:
  - **EN**: Bridges LLDB abstractions to host files, sockets, terminals, and native process facilities.
  - **CN**: 将 LLDB 抽象连接到主机文件、套接字、终端与本地进程设施。

## Dependencies / 依赖关系

- **Direct LLDB/LLVM includes / 直接的 LLDB/LLVM 包含**: `NativeRegisterContextLinux_arm64.h`, `NativeRegisterContextLinux_arm.h`, `NativeRegisterContextLinux_arm64dbreg.h`, `lldb/Host/HostInfo.h`, `lldb/Host/common/NativeProcessProtocol.h`, `lldb/Host/linux/Ptrace.h`, `lldb/Utility/DataBufferHeap.h`, `lldb/Utility/Log.h`, `lldb/Utility/RegisterValue.h`, `lldb/Utility/Status.h` ... (+6 more)
- **Standard-library headers / 标准库头文件**: `<sys/uio.h>`, `<elf.h>`, `<mutex>`, `<optional>`
- **Subsystem categories / 子系统类别**: shared LLDB utility classes / 共享 LLDB 工具类 (4), host OS abstraction helpers / 主机操作系统抽象辅助组件 (3)
