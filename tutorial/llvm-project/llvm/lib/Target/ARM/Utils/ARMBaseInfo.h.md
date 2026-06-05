# ARMBaseInfo.h — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/Utils/ARMBaseInfo.h`
- Repository: `llvm-project`
- Purpose (EN): This file contains small standalone helper functions and enum definitions for the ARM target useful for the compiler back-end and the MC libraries.
- 用途 (CN): 声明 ARM 后端中的 `ARMBaseInfo`，并提供与后端工具辅助逻辑相关的接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```cpp
//===-- ARMBaseInfo.h - Top level definitions for ARM ---*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains small standalone helper functions and enum definitions for
// the ARM target useful for the compiler back-end and the MC libraries.
// As such, it deliberately does not include references to LLVM core
// code gen types, passes, etc..
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 16-17
```cpp
#ifndef LLVM_LIB_TARGET_ARM_UTILS_ARMBASEINFO_H
#define LLVM_LIB_TARGET_ARM_UTILS_ARMBASEINFO_H
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 19-22
```cpp
#include "llvm/ADT/StringSwitch.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/TargetParser/SubtargetFeature.h"
#include "MCTargetDesc/ARMMCTargetDesc.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 24-24
```cpp
namespace llvm {
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 26-40
```cpp
// Enums corresponding to ARM condition codes
namespace ARMCC {
// The CondCodes constants map directly to the 4-bit encoding of the
// condition field for predicated instructions.
enum CondCodes { // Meaning (integer)          Meaning (floating-point)
  EQ,            // Equal                      Equal
  NE,            // Not equal                  Not equal, or unordered
  HS,            // Carry set                  >, ==, or unordered
  LO,            // Carry clear                Less than
  MI,            // Minus, negative            Less than
  PL,            // Plus, positive or zero     >, ==, or unordered
  VS,            // Overflow                   Unordered
  VC,            // No overflow                Not unordered
  HI,            // Unsigned higher            Greater than, or unordered
  LS,            // Unsigned lower or same     Less than or equal
```
- EN: Defines enumeration `CondCodes` to name backend-specific modes, cases, or flags used elsewhere in the file.
- CN: 这里定义枚举 `CondCodes`，用于命名本文件后续会使用的后端模式、分支情况或标志。

### Lines 41-46
```cpp
  GE,            // Greater than or equal      Greater than or equal
  LT,            // Less than                  Less than, or unordered
  GT,            // Greater than               Greater than
  LE,            // Less than or equal         <, ==, or unordered
  AL             // Always (unconditional)     Always (unconditional)
};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 48-62
```cpp
inline static CondCodes getOppositeCondition(CondCodes CC) {
  switch (CC) {
  default: llvm_unreachable("Unknown condition code");
  case EQ: return NE;
  case NE: return EQ;
  case HS: return LO;
  case LO: return HS;
  case MI: return PL;
  case PL: return MI;
  case VS: return VC;
  case VC: return VS;
  case HI: return LS;
  case LS: return HI;
  case GE: return LT;
  case LT: return GE;
```
- EN: Implements `getOppositeCondition`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getOppositeCondition`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 63-66
```cpp
  case GT: return LE;
  case LE: return GT;
  }
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 68-82
```cpp
/// getSwappedCondition - assume the flags are set by MI(a,b), return
/// the condition code if we modify the instructions such that flags are
/// set by MI(b,a).
inline static ARMCC::CondCodes getSwappedCondition(ARMCC::CondCodes CC) {
  switch (CC) {
  default: return ARMCC::AL;
  case ARMCC::EQ: return ARMCC::EQ;
  case ARMCC::NE: return ARMCC::NE;
  case ARMCC::HS: return ARMCC::LS;
  case ARMCC::LO: return ARMCC::HI;
  case ARMCC::HI: return ARMCC::LO;
  case ARMCC::LS: return ARMCC::HS;
  case ARMCC::GE: return ARMCC::LE;
  case ARMCC::LT: return ARMCC::GT;
  case ARMCC::GT: return ARMCC::LT;
```
- EN: Implements `getSwappedCondition`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getSwappedCondition`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 83-86
```cpp
  case ARMCC::LE: return ARMCC::GE;
  }
}
} // end namespace ARMCC
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 88-94
```cpp
namespace ARMVCC {
  enum VPTCodes {
    None = 0,
    Then,
    Else
  };
} // namespace ARMVCC
```
- EN: Defines enumeration `VPTCodes` to name backend-specific modes, cases, or flags used elsewhere in the file.
- CN: 这里定义枚举 `VPTCodes`，用于命名本文件后续会使用的后端模式、分支情况或标志。

### Lines 96-110
```cpp
namespace ARM {
  /// Mask values for IT and VPT Blocks, to be used by MCOperands.
  /// Note that this is different from the "real" encoding used by the
  /// instructions. In this encoding, the lowest set bit indicates the end of
  /// the encoding, and above that, "1" indicates an else, while "0" indicates
  /// a then.
  ///   Tx = x100
  ///   Txy = xy10
  ///   Txyz = xyz1
  enum class PredBlockMask {
    T = 0b1000,
    TT = 0b0100,
    TE = 0b1100,
    TTT = 0b0010,
    TTE = 0b0110,
```
- EN: Declares `PredBlockMask`, packaging target-specific state and APIs around `ARMBaseInfo`.
- CN: 这里声明 `PredBlockMask`，把与 `ARMBaseInfo` 相关的目标特定状态和 API 组织在一起。

### Lines 111-122
```cpp
    TEE = 0b1110,
    TET = 0b1010,
    TTTT = 0b0001,
    TTTE = 0b0011,
    TTEE = 0b0111,
    TTET = 0b0101,
    TEEE = 0b1111,
    TEET = 0b1101,
    TETT = 0b1001,
    TETE = 0b1011
  };
} // namespace ARM
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 124-128
```cpp
// Expands a PredBlockMask by adding an E or a T at the end, depending on Kind.
// e.g ExpandPredBlockMask(T, Then) = TT, ExpandPredBlockMask(TT, Else) = TTE,
// and so on.
ARM::PredBlockMask expandPredBlockMask(ARM::PredBlockMask BlockMask,
                                       ARMVCC::VPTCodes Kind);
```
- EN: Declares `expandPredBlockMask`, a mutation/build routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `expandPredBlockMask`，它是一个围绕目标相关状态展开的构造/变换例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 130-137
```cpp
inline static const char *ARMVPTPredToString(ARMVCC::VPTCodes CC) {
  switch (CC) {
  case ARMVCC::None:  return "none";
  case ARMVCC::Then:  return "t";
  case ARMVCC::Else:  return "e";
  }
  llvm_unreachable("Unknown VPT code");
}
```
- EN: Implements `ARMVPTPredToString`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMVPTPredToString`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 139-144
```cpp
inline static unsigned ARMVectorCondCodeFromString(StringRef CC) {
  return StringSwitch<unsigned>(CC.lower())
    .Case("t", ARMVCC::Then)
    .Case("e", ARMVCC::Else)
    .Default(~0U);
}
```
- EN: Implements `ARMVectorCondCodeFromString`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMVectorCondCodeFromString`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 146-160
```cpp
inline static const char *ARMCondCodeToString(ARMCC::CondCodes CC) {
  switch (CC) {
  case ARMCC::EQ:  return "eq";
  case ARMCC::NE:  return "ne";
  case ARMCC::HS:  return "hs";
  case ARMCC::LO:  return "lo";
  case ARMCC::MI:  return "mi";
  case ARMCC::PL:  return "pl";
  case ARMCC::VS:  return "vs";
  case ARMCC::VC:  return "vc";
  case ARMCC::HI:  return "hi";
  case ARMCC::LS:  return "ls";
  case ARMCC::GE:  return "ge";
  case ARMCC::LT:  return "lt";
  case ARMCC::GT:  return "gt";
```
- EN: Implements `ARMCondCodeToString`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMCondCodeToString`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 161-165
```cpp
  case ARMCC::LE:  return "le";
  case ARMCC::AL:  return "al";
  }
  llvm_unreachable("Unknown condition code");
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 167-181
```cpp
inline static unsigned ARMCondCodeFromString(StringRef CC) {
  return StringSwitch<unsigned>(CC.lower())
    .Case("eq", ARMCC::EQ)
    .Case("ne", ARMCC::NE)
    .Case("hs", ARMCC::HS)
    .Case("cs", ARMCC::HS)
    .Case("lo", ARMCC::LO)
    .Case("cc", ARMCC::LO)
    .Case("mi", ARMCC::MI)
    .Case("pl", ARMCC::PL)
    .Case("vs", ARMCC::VS)
    .Case("vc", ARMCC::VC)
    .Case("hi", ARMCC::HI)
    .Case("ls", ARMCC::LS)
    .Case("ge", ARMCC::GE)
```
- EN: Implements `ARMCondCodeFromString`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMCondCodeFromString`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 182-187
```cpp
    .Case("lt", ARMCC::LT)
    .Case("gt", ARMCC::GT)
    .Case("le", ARMCC::LE)
    .Case("al", ARMCC::AL)
    .Default(~0U);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 189-196
```cpp
// System Registers
namespace ARMSysReg {
  struct MClassSysReg {
    const char Name[32];
    uint16_t M1Encoding12;
    uint16_t M2M3Encoding8;
    uint16_t Encoding;
    FeatureBitset FeaturesRequired;
```
- EN: Declares `MClassSysReg`, packaging target-specific state and APIs around `ARMBaseInfo`.
- CN: 这里声明 `MClassSysReg`，把与 `ARMBaseInfo` 相关的目标特定状态和 API 组织在一起。

### Lines 198-201
```cpp
    // return true if FeaturesRequired are all present in ActiveFeatures
    bool hasRequiredFeatures(FeatureBitset ActiveFeatures) const {
      return (FeaturesRequired & ActiveFeatures) == FeaturesRequired;
    }
```
- EN: Implements `hasRequiredFeatures`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `hasRequiredFeatures`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 203-207
```cpp
    // returns true if TestFeatures are all present in FeaturesRequired
    bool isInRequiredFeatures(FeatureBitset TestFeatures) const {
      return (FeaturesRequired & TestFeatures) == TestFeatures;
    }
  };
```
- EN: Implements `isInRequiredFeatures`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isInRequiredFeatures`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 209-210
```cpp
#define GET_MClassSysRegsList_DECL
#include "ARMGenSystemRegister.inc"
```
- EN: Connects this file to TableGen-generated declarations so target-specific enums and helper tables become available to C++ code.
- CN: 这里把文件连接到 TableGen 生成的声明，使目标相关枚举和辅助表能够在 C++ 代码中使用。

### Lines 212-214
```cpp
  // lookup system register using 12-bit SYSm value.
  // Note: the search is uniqued using M1 mask
  const MClassSysReg *lookupMClassSysRegBy12bitSYSmValue(unsigned SYSm);
```
- EN: Declares `lookupMClassSysRegBy12bitSYSmValue`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `lookupMClassSysRegBy12bitSYSmValue`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 216-218
```cpp
  // returns APSR with _<bits> qualifier.
  // Note: ARMv7-M deprecates using MSR APSR without a _<bits> qualifier
  const MClassSysReg *lookupMClassSysRegAPSRNonDeprecated(unsigned SYSm);
```
- EN: Declares `lookupMClassSysRegAPSRNonDeprecated`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `lookupMClassSysRegAPSRNonDeprecated`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 220-221
```cpp
  // lookup system registers using 8-bit SYSm value
  const MClassSysReg *lookupMClassSysRegBy8bitSYSmValue(unsigned SYSm);
```
- EN: Declares `lookupMClassSysRegBy8bitSYSmValue`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `lookupMClassSysRegBy8bitSYSmValue`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 223-223
```cpp
} // end namespace ARMSysReg
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 225-233
```cpp
// Banked Registers
namespace ARMBankedReg {
  struct BankedReg {
    const char *Name;
    uint16_t Encoding;
  };
#define GET_BankedRegsList_DECL
#include "ARMGenSystemRegister.inc"
} // end namespace ARMBankedReg
```
- EN: Declares `BankedReg`, packaging target-specific state and APIs around `ARMBaseInfo`.
- CN: 这里声明 `BankedReg`，把与 `ARMBaseInfo` 相关的目标特定状态和 API 组织在一起。

### Lines 235-235
```cpp
} // end namespace llvm
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 237-237
```cpp
#endif // LLVM_LIB_TARGET_ARM_UTILS_ARMBASEINFO_H
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

## Key Concepts / 关键概念

- EN: Primary role: backend utility helpers.
  - CN: 核心职责：后端工具辅助逻辑。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。

## Dependencies / 依赖关系

- EN: Backend-local headers: `MCTargetDesc/ARMMCTargetDesc.h`, `ARMGenSystemRegister.inc`, `ARMGenSystemRegister.inc`.
  - CN: 后端本地头文件：`MCTargetDesc/ARMMCTargetDesc.h`, `ARMGenSystemRegister.inc`, `ARMGenSystemRegister.inc`。
- EN: LLVM infrastructure headers: `llvm/ADT/StringSwitch.h`, `llvm/Support/ErrorHandling.h`, `llvm/TargetParser/SubtargetFeature.h`.
  - CN: LLVM 基础设施头文件：`llvm/ADT/StringSwitch.h`, `llvm/Support/ErrorHandling.h`, `llvm/TargetParser/SubtargetFeature.h`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
