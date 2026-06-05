# SIModeRegisterDefaults.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/SIModeRegisterDefaults.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This header declares the interfaces, data structures, and pass entry points for SIModeRegisterDefaults in the LLVM AMDGPU backend. It defines the contracts consumed by other AMDGPU backend components. / 该头文件声明 LLVM AMDGPU 后端中 SIModeRegisterDefaults 的接口、数据结构与 Pass 入口。它定义了其他 AMDGPU 后端组件依赖的契约。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: File banner, includes, and setup
```cpp
//===-- SIModeRegisterDefaults.h --------------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AMDGPU_SIMODEREGISTERDEFAULTS_H
#define LLVM_LIB_TARGET_AMDGPU_SIMODEREGISTERDEFAULTS_H

#include "Utils/AMDGPUBaseInfo.h"
#include "llvm/ADT/FloatingPointMode.h"

namespace llvm {

class GCNSubtarget;

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `GCNSubtarget`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`GCNSubtarget`。

### Lines 19-38: Declares struct SIModeRegisterDefaults
```cpp
// Track defaults for fields in the MODE register.
struct SIModeRegisterDefaults {
  /// Floating point opcodes that support exception flag gathering quiet and
  /// propagate signaling NaN inputs per IEEE 754-2008. Min_dx10 and max_dx10
  /// become IEEE 754- 2008 compliant due to signaling NaN propagation and
  /// quieting.
  bool IEEE : 1;

  /// Used by the vector ALU to force DX10-style treatment of NaNs: when set,
  /// clamp NaN to zero; otherwise, pass NaN through.
  bool DX10Clamp : 1;

  /// If this is set, neither input or output denormals are flushed for most f32
  /// instructions.
  DenormalMode FP32Denormals;

  /// If this is set, neither input or output denormals are flushed for both f64
  /// and f16/v2f16 instructions.
  DenormalMode FP64FP16Denormals;

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `SIModeRegisterDefaults`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`SIModeRegisterDefaults`。

### Lines 39-58: Defines SIModeRegisterDefaults
```cpp
  SIModeRegisterDefaults() :
    IEEE(true),
    DX10Clamp(true),
    FP32Denormals(DenormalMode::getIEEE()),
    FP64FP16Denormals(DenormalMode::getIEEE()) {}

  SIModeRegisterDefaults(const Function &F, const GCNSubtarget &ST);

  static SIModeRegisterDefaults getDefaultForCallingConv(CallingConv::ID CC) {
    SIModeRegisterDefaults Mode;
    Mode.IEEE = !AMDGPU::isShader(CC);
    return Mode;
  }

  bool operator==(const SIModeRegisterDefaults Other) const {
    return IEEE == Other.IEEE && DX10Clamp == Other.DX10Clamp &&
           FP32Denormals == Other.FP32Denormals &&
           FP64FP16Denormals == Other.FP64FP16Denormals;
  }

```
**EN:** This section contains concrete logic for SIModeRegisterDefaults. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `DenormalMode::getIEEE`, `AMDGPU::isShader`.
**CN:** 本节包含与 SIModeRegisterDefaults 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`DenormalMode::getIEEE`, `AMDGPU::isShader`。

### Lines 59-82: Defines fpDenormModeSPValue
```cpp
  /// Get the encoding value for the FP_DENORM bits of the mode register for the
  /// FP32 denormal mode.
  uint32_t fpDenormModeSPValue() const {
    if (FP32Denormals == DenormalMode::getPreserveSign())
      return FP_DENORM_FLUSH_IN_FLUSH_OUT;
    if (FP32Denormals.Output == DenormalMode::PreserveSign)
      return FP_DENORM_FLUSH_OUT;
    if (FP32Denormals.Input == DenormalMode::PreserveSign)
      return FP_DENORM_FLUSH_IN;
    return FP_DENORM_FLUSH_NONE;
  }

  /// Get the encoding value for the FP_DENORM bits of the mode register for the
  /// FP64/FP16 denormal mode.
  uint32_t fpDenormModeDPValue() const {
    if (FP64FP16Denormals == DenormalMode::getPreserveSign())
      return FP_DENORM_FLUSH_IN_FLUSH_OUT;
    if (FP64FP16Denormals.Output == DenormalMode::PreserveSign)
      return FP_DENORM_FLUSH_OUT;
    if (FP64FP16Denormals.Input == DenormalMode::PreserveSign)
      return FP_DENORM_FLUSH_IN;
    return FP_DENORM_FLUSH_NONE;
  }

```
**EN:** This section contains concrete logic for fpDenormModeSPValue. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `DenormalMode::getPreserveSign`.
**CN:** 本节包含与 fpDenormModeSPValue 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`DenormalMode::getPreserveSign`。

### Lines 83-106: Defines isInlineCompatible
```cpp
  // FIXME: Inlining should be OK for dx10-clamp, since the caller's mode should
  // be able to override.
  bool isInlineCompatible(SIModeRegisterDefaults CalleeMode) const {
    return DX10Clamp == CalleeMode.DX10Clamp && IEEE == CalleeMode.IEEE;
  }
};

namespace AMDGPU {

/// Return values used for llvm.get.rounding
///
/// When both the F32 and F64/F16 modes are the same, returns the standard
/// values. If they differ, returns an extended mode starting at 8.
enum AMDGPUFltRounds : int8_t {
  // Inherit everything from RoundingMode
  TowardZero = static_cast<int8_t>(RoundingMode::TowardZero),
  NearestTiesToEven = static_cast<int8_t>(RoundingMode::NearestTiesToEven),
  TowardPositive = static_cast<int8_t>(RoundingMode::TowardPositive),
  TowardNegative = static_cast<int8_t>(RoundingMode::TowardNegative),
  NearestTiesToAwayUnsupported =
      static_cast<int8_t>(RoundingMode::NearestTiesToAway),

  Dynamic = static_cast<int8_t>(RoundingMode::Dynamic),

```
**EN:** This section contains concrete logic for isInlineCompatible. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUFltRounds`.
**CN:** 本节包含与 isInlineCompatible 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUFltRounds`。

### Lines 107-129: Type declarations and aliases
```cpp
  // Permute the mismatched rounding mode cases.  If the modes are the same, use
  // the standard values, otherwise, these values are sorted such that higher
  // hardware encoded values have higher enum values.
  NearestTiesToEvenF32_NearestTiesToEvenF64 = NearestTiesToEven,
  NearestTiesToEvenF32_TowardPositiveF64 = 8,
  NearestTiesToEvenF32_TowardNegativeF64 = 9,
  NearestTiesToEvenF32_TowardZeroF64 = 10,

  TowardPositiveF32_NearestTiesToEvenF64 = 11,
  TowardPositiveF32_TowardPositiveF64 = TowardPositive,
  TowardPositiveF32_TowardNegativeF64 = 12,
  TowardPositiveF32_TowardZeroF64 = 13,

  TowardNegativeF32_NearestTiesToEvenF64 = 14,
  TowardNegativeF32_TowardPositiveF64 = 15,
  TowardNegativeF32_TowardNegativeF64 = TowardNegative,
  TowardNegativeF32_TowardZeroF64 = 16,

  TowardZeroF32_NearestTiesToEvenF64 = 17,
  TowardZeroF32_TowardPositiveF64 = 18,
  TowardZeroF32_TowardNegativeF64 = 19,
  TowardZeroF32_TowardZeroF64 = TowardZero,

```
**EN:** This section introduces supporting types or aliases that simplify later declarations and backend logic.
**CN:** 本节引入辅助类型或别名，以简化后续声明与后端逻辑。

### Lines 130-153: Implementation details and local logic
```cpp
  Invalid = static_cast<int8_t>(RoundingMode::Invalid)
};

/// Offset of nonstandard values for llvm.get.rounding results from the largest
/// supported mode.
static constexpr uint32_t ExtendedFltRoundOffset = 4;

/// Offset in mode register of f32 rounding mode.
static constexpr uint32_t F32FltRoundOffset = 0;

/// Offset in mode register of f64/f16 rounding mode.
static constexpr uint32_t F64FltRoundOffset = 2;

// Bit indexed table to convert from hardware rounding mode values to FLT_ROUNDS
// values.
extern const uint64_t FltRoundConversionTable;

// Bit indexed table to convert from FLT_ROUNDS values to hardware rounding mode
// values
extern const uint64_t FltRoundToHWConversionTable;

/// Read the hardware rounding mode equivalent of a AMDGPUFltRounds value.
uint32_t decodeFltRoundToHWConversionTable(uint32_t FltRounds);

```
**EN:** This section advances the file implementation by combining declarations, control flow, and helper operations that support the AMDGPU backend.
**CN:** 本节通过组合声明、控制流与辅助操作来推进文件实现，支撑 AMDGPU 后端的整体功能。

### Lines 154-158: Preprocessor guards and macros
```cpp
} // end namespace AMDGPU

} // end namespace llvm

#endif // LLVM_LIB_TARGET_AMDGPU_SIMODEREGISTERDEFAULTS_H
```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。

## Key Concepts / 关键概念
- **Language / 语言**: C++ header
- **Primary symbols / 主要符号**: `GCNSubtarget`, `SIModeRegisterDefaults`, `AMDGPUFltRounds`, `DenormalMode::getIEEE`, `AMDGPU::isShader`, `DenormalMode::getPreserveSign`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; LLVM pass integration / LLVM Pass 集成
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"Utils/AMDGPUBaseInfo.h"`
- `"llvm/ADT/FloatingPointMode.h"`
