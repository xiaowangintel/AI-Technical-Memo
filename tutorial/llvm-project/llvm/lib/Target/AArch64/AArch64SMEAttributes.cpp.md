# AArch64SMEAttributes.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/AArch64/AArch64SMEAttributes.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This file covers Helper for interpreting SME attributes. / 该文件实现 AArch64 后端中的SME 矩阵/向量支持。
## Line-by-Line Analysis / 逐行分析
### Lines 1-24: Documented code section
```cpp
//===-- AArch64SMEAttributes.cpp - Helper for interpreting SME attributes -===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "AArch64SMEAttributes.h"
#include "AArch64ISelLowering.h"
#include "llvm/IR/InstrTypes.h"
#include "llvm/IR/RuntimeLibcalls.h"
#include <cassert>

using namespace llvm;

void SMEAttrs::validate() const {
  // Streaming Mode Attrs
  assert(!(hasStreamingInterface() && hasStreamingCompatibleInterface()) &&
         "SM_Enabled and SM_Compatible are mutually exclusive");

  // ZA Attrs
  assert(!(isNewZA() && (Bitmask & SME_ABI_Routine)) &&
         "ZA_New and SME_ABI_Routine are mutually exclusive");
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 25-41: Core AArch64 backend logic
```cpp

  assert(
      (isNewZA() + isInZA() + isOutZA() + isInOutZA() + isPreservesZA()) <= 1 &&
      "Attributes 'aarch64_new_za', 'aarch64_in_za', 'aarch64_out_za', "
      "'aarch64_inout_za' and 'aarch64_preserves_za' are mutually exclusive");

  // ZT0 Attrs
  assert(
      (isNewZT0() + isInZT0() + isOutZT0() + isInOutZT0() + isPreservesZT0()) <=
          1 &&
      "Attributes 'aarch64_new_zt0', 'aarch64_in_zt0', 'aarch64_out_zt0', "
      "'aarch64_inout_zt0' and 'aarch64_preserves_zt0' are mutually exclusive");

  assert(!(hasAgnosticZAInterface() && hasSharedZAInterface()) &&
         "Function cannot have a shared-ZA interface and an agnostic-ZA "
         "interface");
}
```
**EN:** This block continues the file's main SME matrix/vector support logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的SME 矩阵/向量支持主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 42-69: Function SMEAttrs::SMEAttrs
```cpp

SMEAttrs::SMEAttrs(const AttributeList &Attrs) {
  // Note: 'aarch64_zt0_undef' was previously used (and subsequently removed).
  // To avoid introducing any compatibility issues don't reuse
  // 'aarch64_zt0_undef' for another purpose.
  Bitmask = 0;
  if (Attrs.hasFnAttr("aarch64_pstate_sm_enabled"))
    Bitmask |= SM_Enabled;
  if (Attrs.hasFnAttr("aarch64_pstate_sm_compatible"))
    Bitmask |= SM_Compatible;
  if (Attrs.hasFnAttr("aarch64_pstate_sm_body"))
    Bitmask |= SM_Body;
  if (Attrs.hasFnAttr("aarch64_za_state_agnostic"))
    Bitmask |= ZA_State_Agnostic;
  if (Attrs.hasFnAttr("aarch64_in_za"))
    Bitmask |= encodeZAState(StateValue::In);
  if (Attrs.hasFnAttr("aarch64_out_za"))
    Bitmask |= encodeZAState(StateValue::Out);
  if (Attrs.hasFnAttr("aarch64_inout_za"))
    Bitmask |= encodeZAState(StateValue::InOut);
  if (Attrs.hasFnAttr("aarch64_preserves_za"))
    Bitmask |= encodeZAState(StateValue::Preserved);
  if (Attrs.hasFnAttr("aarch64_new_za"))
    Bitmask |= encodeZAState(StateValue::New);
  if (Attrs.hasFnAttr("aarch64_in_zt0"))
    Bitmask |= encodeZT0State(StateValue::In);
  if (Attrs.hasFnAttr("aarch64_out_zt0"))
    Bitmask |= encodeZT0State(StateValue::Out);
```
**EN:** This block implements SMEAttrs::SMEAttrs, advancing the file's SME matrix/vector support flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 SMEAttrs::SMEAttrs，通过 AArch64 专用的决策与数据处理推进本文件的SME 矩阵/向量支持流程。
### Lines 70-97: Function SMEAttrs::addKnownFunctionAttrs
```cpp
  if (Attrs.hasFnAttr("aarch64_inout_zt0"))
    Bitmask |= encodeZT0State(StateValue::InOut);
  if (Attrs.hasFnAttr("aarch64_preserves_zt0"))
    Bitmask |= encodeZT0State(StateValue::Preserved);
  if (Attrs.hasFnAttr("aarch64_new_zt0"))
    Bitmask |= encodeZT0State(StateValue::New);
}

void SMEAttrs::addKnownFunctionAttrs(StringRef FuncName,
                                     const RTLIB::RuntimeLibcallsInfo &RTLCI) {
  RTLIB::LibcallImpl Impl = RTLCI.getSupportedLibcallImpl(FuncName);
  if (Impl == RTLIB::Unsupported)
    return;
  unsigned KnownAttrs = SMEAttrs::Normal;
  RTLIB::Libcall LC = RTLIB::RuntimeLibcallsInfo::getLibcallFromImpl(Impl);
  switch (LC) {
  case RTLIB::SMEABI_SME_STATE:
  case RTLIB::SMEABI_TPIDR2_SAVE:
  case RTLIB::SMEABI_GET_CURRENT_VG:
  case RTLIB::SMEABI_SME_STATE_SIZE:
  case RTLIB::SMEABI_SME_SAVE:
  case RTLIB::SMEABI_SME_RESTORE:
    KnownAttrs |= SMEAttrs::SM_Compatible | SMEAttrs::SME_ABI_Routine;
    break;
  case RTLIB::SMEABI_ZA_DISABLE:
  case RTLIB::SMEABI_TPIDR2_RESTORE:
    KnownAttrs |= SMEAttrs::SM_Compatible | encodeZAState(StateValue::In) |
                  SMEAttrs::SME_ABI_Routine;
```
**EN:** This block implements SMEAttrs::addKnownFunctionAttrs, advancing the file's SME matrix/vector support flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 SMEAttrs::addKnownFunctionAttrs，通过 AArch64 专用的决策与数据处理推进本文件的SME 矩阵/向量支持流程。
### Lines 98-123: Core AArch64 backend logic
```cpp
    break;
  case RTLIB::SC_MEMCPY:
  case RTLIB::SC_MEMMOVE:
  case RTLIB::SC_MEMSET:
  case RTLIB::SC_MEMCHR:
    KnownAttrs |= SMEAttrs::SM_Compatible;
    break;
  default:
    break;
  }
  set(KnownAttrs);
}

bool SMECallAttrs::requiresSMChange() const {
  if (callee().hasStreamingCompatibleInterface())
    return false;

  // Both non-streaming
  if (caller().hasNonStreamingInterfaceAndBody() &&
      callee().hasNonStreamingInterface())
    return false;

  // Both streaming
  if (caller().hasStreamingInterfaceOrBody() &&
      callee().hasStreamingInterface())
    return false;
```
**EN:** This block continues the file's main SME matrix/vector support logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的SME 矩阵/向量支持主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 124-145: Function SMECallAttrs::SMECallAttrs
```cpp

  return true;
}

SMECallAttrs::SMECallAttrs(const CallBase &CB,
                           const RTLIB::RuntimeLibcallsInfo *RTLCI)
    : CallerFn(*CB.getFunction()), CalledFn(SMEAttrs::Normal),
      Callsite(CB.getAttributes()), IsIndirect(CB.isIndirectCall()) {
  if (auto *CalledFunction = CB.getCalledFunction())
    CalledFn = SMEAttrs(*CalledFunction, RTLCI);

  // FIXME: We probably should not allow SME attributes on direct calls but
  // clang duplicates streaming mode attributes at each callsite.
  assert((IsIndirect || ((Callsite | CalledFn) == CalledFn)) &&
         "SME attributes at callsite do not match declaration");

  // An `invoke` of an agnostic ZA function may not return normally (it may
  // resume in an exception block). In this case, it acts like a private ZA
  // callee and may require a ZA save to be set up before it is called.
  if (isa<InvokeInst>(CB))
    CalledFn.set(SMEAttrs::ZA_State_Agnostic, /*Enable=*/false);
}
```
**EN:** This block implements SMECallAttrs::SMECallAttrs, advancing the file's SME matrix/vector support flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 SMECallAttrs::SMECallAttrs，通过 AArch64 专用的决策与数据处理推进本文件的SME 矩阵/向量支持流程。
## Key Concepts / 关键概念
- **EN:** AArch64 target backend structure **CN:** AArch64 目标后端结构
- **EN:** Scalable Matrix Extension support **CN:** 可伸缩矩阵扩展 SME 支持
## Dependencies / 依赖关系
- **EN:** Target-local includes: AArch64SMEAttributes.h, AArch64ISelLowering.h **CN:** 目标本地依赖：AArch64SMEAttributes.h, AArch64ISelLowering.h
- **EN:** Core LLVM interfaces: llvm/IR/InstrTypes.h, llvm/IR/RuntimeLibcalls.h **CN:** 核心 LLVM 接口：llvm/IR/InstrTypes.h, llvm/IR/RuntimeLibcalls.h
- **EN:** Standard-library support: cassert **CN:** 标准库支持：cassert
- **EN:** Closely connected with neighboring AArch64 backend components responsible for SME matrix/vector support. **CN:** 与周边负责SME 矩阵/向量支持的 AArch64 后端组件紧密协作。
