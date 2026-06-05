# AArch64SMEAttributes.h — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/AArch64/AArch64SMEAttributes.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This file covers Helper for interpreting SME attributes. / 该文件实现 AArch64 后端中的SME 矩阵/向量支持。
## Line-by-Line Analysis / 逐行分析
### Lines 1-26: Documented code section
```cpp
//===-- AArch64SMEAttributes.h - Helper for interpreting SME attributes -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AARCH64_UTILS_AARCH64SMEATTRIBUTES_H
#define LLVM_LIB_TARGET_AARCH64_UTILS_AARCH64SMEATTRIBUTES_H

#include "llvm/IR/Function.h"

namespace llvm {
namespace RTLIB {
struct RuntimeLibcallsInfo;
}

class Function;
class CallBase;
class AttributeList;

/// SMEAttrs is a utility class to parse the SME ACLE attributes on functions.
/// It helps determine a function's requirements for PSTATE.ZA and PSTATE.SM.
class SMEAttrs {
  unsigned Bitmask = Normal;
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 27-50: Class StateValue
```cpp

public:
  enum class StateValue {
    None = 0,
    In = 1,        // aarch64_in_zt0
    Out = 2,       // aarch64_out_zt0
    InOut = 3,     // aarch64_inout_zt0
    Preserved = 4, // aarch64_preserves_zt0
    New = 5        // aarch64_new_zt0
  };

  // Enum with bitmasks for each individual SME feature.
  enum Mask {
    Normal = 0,
    SM_Enabled = 1 << 0,      // aarch64_pstate_sm_enabled
    SM_Compatible = 1 << 1,   // aarch64_pstate_sm_compatible
    SM_Body = 1 << 2,         // aarch64_pstate_sm_body
    SME_ABI_Routine = 1 << 3, // Used for SME ABI routines to avoid lazy saves
    ZA_State_Agnostic = 1 << 4,
    ZA_Shift = 6,
    ZA_Mask = 0b111 << ZA_Shift,
    ZT0_Shift = 9,
    ZT0_Mask = 0b111 << ZT0_Shift
  };
```
**EN:** This block defines StateValue, packaging state and behavior that the file reuses for SME matrix/vector support.  
**CN:** 该代码块定义 StateValue，把 SME 矩阵/向量支持 所需的状态与行为封装在一起供后续复用。
### Lines 51-72: Function SMEAttrs
```cpp

  SMEAttrs() = default;
  SMEAttrs(unsigned Mask) { set(Mask); }
  SMEAttrs(const Function &F, const RTLIB::RuntimeLibcallsInfo *RTLCI = nullptr)
      : SMEAttrs(F.getAttributes()) {
    if (RTLCI)
      addKnownFunctionAttrs(F.getName(), *RTLCI);
  }
  SMEAttrs(const AttributeList &L);
  SMEAttrs(StringRef FuncName, const RTLIB::RuntimeLibcallsInfo &RTLCI) {
    addKnownFunctionAttrs(FuncName, RTLCI);
  };

  void set(unsigned M, bool Enable = true) {
    if (Enable)
      Bitmask |= M;
    else
      Bitmask &= ~M;
#ifndef NDEBUG
    validate();
#endif
  }
```
**EN:** This block implements SMEAttrs, advancing the file's SME matrix/vector support flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 SMEAttrs，通过 AArch64 专用的决策与数据处理推进本文件的SME 矩阵/向量支持流程。
### Lines 73-96: Function hasStreamingBody
```cpp

  // Interfaces to query PSTATE.SM
  bool hasStreamingBody() const { return Bitmask & SM_Body; }
  bool hasStreamingInterface() const { return Bitmask & SM_Enabled; }
  bool hasStreamingInterfaceOrBody() const {
    return hasStreamingBody() || hasStreamingInterface();
  }
  bool hasStreamingCompatibleInterface() const {
    return Bitmask & SM_Compatible;
  }
  bool hasNonStreamingInterface() const {
    return !hasStreamingInterface() && !hasStreamingCompatibleInterface();
  }
  bool hasNonStreamingInterfaceAndBody() const {
    return hasNonStreamingInterface() && !hasStreamingBody();
  }

  // Interfaces to query ZA
  static StateValue decodeZAState(unsigned Bitmask) {
    return static_cast<StateValue>((Bitmask & ZA_Mask) >> ZA_Shift);
  }
  static unsigned encodeZAState(StateValue S) {
    return static_cast<unsigned>(S) << ZA_Shift;
  }
```
**EN:** This block implements hasStreamingBody, advancing the file's SME matrix/vector support flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 hasStreamingBody，通过 AArch64 专用的决策与数据处理推进本文件的SME 矩阵/向量支持流程。
### Lines 97-116: Function isNewZA
```cpp

  bool isNewZA() const { return decodeZAState(Bitmask) == StateValue::New; }
  bool isInZA() const { return decodeZAState(Bitmask) == StateValue::In; }
  bool isOutZA() const { return decodeZAState(Bitmask) == StateValue::Out; }
  bool isInOutZA() const { return decodeZAState(Bitmask) == StateValue::InOut; }
  bool isPreservesZA() const {
    return decodeZAState(Bitmask) == StateValue::Preserved;
  }
  bool sharesZA() const {
    StateValue State = decodeZAState(Bitmask);
    return State == StateValue::In || State == StateValue::Out ||
           State == StateValue::InOut || State == StateValue::Preserved;
  }
  bool hasAgnosticZAInterface() const { return Bitmask & ZA_State_Agnostic; }
  bool hasSharedZAInterface() const { return sharesZA() || sharesZT0(); }
  bool hasPrivateZAInterface() const {
    return !hasSharedZAInterface() && !hasAgnosticZAInterface();
  }
  bool hasZAState() const { return isNewZA() || sharesZA(); }
  bool isSMEABIRoutine() const { return Bitmask & SME_ABI_Routine; }
```
**EN:** This block implements isNewZA, advancing the file's SME matrix/vector support flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 isNewZA，通过 AArch64 专用的决策与数据处理推进本文件的SME 矩阵/向量支持流程。
### Lines 117-140: Function decodeZT0State
```cpp

  // Interfaces to query ZT0 State
  static StateValue decodeZT0State(unsigned Bitmask) {
    return static_cast<StateValue>((Bitmask & ZT0_Mask) >> ZT0_Shift);
  }
  static unsigned encodeZT0State(StateValue S) {
    return static_cast<unsigned>(S) << ZT0_Shift;
  }

  bool isNewZT0() const { return decodeZT0State(Bitmask) == StateValue::New; }
  bool isInZT0() const { return decodeZT0State(Bitmask) == StateValue::In; }
  bool isOutZT0() const { return decodeZT0State(Bitmask) == StateValue::Out; }
  bool isInOutZT0() const {
    return decodeZT0State(Bitmask) == StateValue::InOut;
  }
  bool isPreservesZT0() const {
    return decodeZT0State(Bitmask) == StateValue::Preserved;
  }
  bool sharesZT0() const {
    StateValue State = decodeZT0State(Bitmask);
    return State == StateValue::In || State == StateValue::Out ||
           State == StateValue::InOut || State == StateValue::Preserved;
  }
  bool hasZT0State() const { return isNewZT0() || sharesZT0(); }
```
**EN:** This block implements decodeZT0State, advancing the file's SME matrix/vector support flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 decodeZT0State，通过 AArch64 专用的决策与数据处理推进本文件的SME 矩阵/向量支持流程。
### Lines 141-165: Core AArch64 backend logic
```cpp

  SMEAttrs operator|(SMEAttrs Other) const {
    SMEAttrs Merged(*this);
    Merged.set(Other.Bitmask);
    return Merged;
  }

  bool operator==(SMEAttrs const &Other) const {
    return Bitmask == Other.Bitmask;
  }

private:
  void addKnownFunctionAttrs(StringRef FuncName,
                             const RTLIB::RuntimeLibcallsInfo &RTLCI);
  void validate() const;
};

/// SMECallAttrs is a utility class to hold the SMEAttrs for a callsite. It has
/// interfaces to query whether a streaming mode change or lazy-save mechanism
/// is required when going from one function to another (e.g. through a call).
class SMECallAttrs {
  SMEAttrs CallerFn;
  SMEAttrs CalledFn;
  SMEAttrs Callsite;
  bool IsIndirect = false;
```
**EN:** This block continues the file's main SME matrix/vector support logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的SME 矩阵/向量支持主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 166-190: Function SMECallAttrs
```cpp

public:
  SMECallAttrs(SMEAttrs Caller, SMEAttrs Callee,
               SMEAttrs Callsite = SMEAttrs::Normal)
      : CallerFn(Caller), CalledFn(Callee), Callsite(Callsite) {}

  SMECallAttrs(const CallBase &CB, const RTLIB::RuntimeLibcallsInfo *RTLCI);

  SMEAttrs &caller() { return CallerFn; }
  SMEAttrs &callee() { return IsIndirect ? Callsite : CalledFn; }
  SMEAttrs &callsite() { return Callsite; }
  SMEAttrs const &caller() const { return CallerFn; }
  SMEAttrs const &callee() const {
    return const_cast<SMECallAttrs *>(this)->callee();
  }
  SMEAttrs const &callsite() const { return Callsite; }

  /// \return true if a call from Caller -> Callee requires a change in
  /// streaming mode.
  bool requiresSMChange() const;

  bool requiresLazySave() const {
    return caller().hasZAState() && callee().hasPrivateZAInterface() &&
           !callee().isSMEABIRoutine();
  }
```
**EN:** This block implements SMECallAttrs, advancing the file's SME matrix/vector support flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 SMECallAttrs，通过 AArch64 专用的决策与数据处理推进本文件的SME 矩阵/向量支持流程。
### Lines 191-205: Function requiresPreservingZT0
```cpp

  bool requiresPreservingZT0() const {
    return caller().hasZT0State() && !callee().sharesZT0() &&
           !callee().hasAgnosticZAInterface();
  }

  bool requiresPreservingAllZAState() const {
    return caller().hasAgnosticZAInterface() &&
           !callee().hasAgnosticZAInterface() && !callee().isSMEABIRoutine();
  }
};

} // namespace llvm

#endif // LLVM_LIB_TARGET_AARCH64_UTILS_AARCH64SMEATTRIBUTES_H
```
**EN:** This block implements requiresPreservingZT0, advancing the file's SME matrix/vector support flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 requiresPreservingZT0，通过 AArch64 专用的决策与数据处理推进本文件的SME 矩阵/向量支持流程。
## Key Concepts / 关键概念
- **EN:** AArch64 target backend structure **CN:** AArch64 目标后端结构
- **EN:** Scalable Matrix Extension support **CN:** 可伸缩矩阵扩展 SME 支持
## Dependencies / 依赖关系
- **EN:** Core LLVM interfaces: llvm/IR/Function.h **CN:** 核心 LLVM 接口：llvm/IR/Function.h
- **EN:** Closely connected with neighboring AArch64 backend components responsible for SME matrix/vector support. **CN:** 与周边负责SME 矩阵/向量支持的 AArch64 后端组件紧密协作。
