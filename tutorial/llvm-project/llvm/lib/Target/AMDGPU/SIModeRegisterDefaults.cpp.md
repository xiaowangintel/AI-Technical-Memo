# SIModeRegisterDefaults.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/SIModeRegisterDefaults.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements SIModeRegisterDefaults for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 SIModeRegisterDefaults 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-33: File banner, includes, and setup
```cpp
//===-- SIModeRegisterDefaults.cpp ------------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "SIModeRegisterDefaults.h"
#include "GCNSubtarget.h"

using namespace llvm;

SIModeRegisterDefaults::SIModeRegisterDefaults(const Function &F,
                                               const GCNSubtarget &ST) {
  *this = getDefaultForCallingConv(F.getCallingConv());

  if (ST.hasFeature(AMDGPU::FeatureDX10ClampAndIEEEMode)) {
    StringRef IEEEAttr = F.getFnAttribute("amdgpu-ieee").getValueAsString();
    if (!IEEEAttr.empty())
      IEEE = IEEEAttr == "true";

    StringRef DX10ClampAttr =
        F.getFnAttribute("amdgpu-dx10-clamp").getValueAsString();
    if (!DX10ClampAttr.empty())
      DX10Clamp = DX10ClampAttr == "true";
  }

  DenormalFPEnv FPEnv = F.getDenormalFPEnv();
  FP64FP16Denormals = FPEnv.DefaultMode;
  FP32Denormals = FPEnv.F32Mode;
}

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation. Main symbols: `SIModeRegisterDefaults::SIModeRegisterDefaults`.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。 主要符号：`SIModeRegisterDefaults::SIModeRegisterDefaults`。

### Lines 34-59: Defines getModeRegisterRoundMode
```cpp
using namespace AMDGPU;

/// Combine f32 and f64 rounding modes into a combined rounding mode value.
static constexpr uint32_t getModeRegisterRoundMode(uint32_t HWFP32Val,
                                                   uint32_t HWFP64Val) {
  return HWFP32Val << F32FltRoundOffset | HWFP64Val << F64FltRoundOffset;
}

static constexpr uint64_t encodeFltRoundsTable(uint32_t FltRoundsVal,
                                               uint32_t HWF32Val,
                                               uint32_t HWF64Val) {
  uint32_t ModeVal = getModeRegisterRoundMode(HWF32Val, HWF64Val);
  if (FltRoundsVal > TowardNegative)
    FltRoundsVal -= ExtendedFltRoundOffset;

  uint32_t BitIndex = ModeVal << 2;
  return static_cast<uint64_t>(FltRoundsVal) << BitIndex;
}

// Encode FLT_ROUNDS value where the two rounding modes are the same and use a
// standard value
static constexpr uint64_t
encodeFltRoundsTableSame(AMDGPUFltRounds FltRoundsMode, uint32_t HWVal) {
  return encodeFltRoundsTable(FltRoundsMode, HWVal, HWVal);
}

```
**EN:** This section contains concrete logic for getModeRegisterRoundMode. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 getModeRegisterRoundMode 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 60-89: Defines decodeIndexFltRoundConversionTable
```cpp
// Convert mode register encoded rounding mode to AMDGPUFltRounds
static constexpr AMDGPUFltRounds
decodeIndexFltRoundConversionTable(uint32_t HWMode) {
  uint32_t TableRead = (FltRoundConversionTable >> (HWMode << 2)) & 0xf;
  if (TableRead > TowardNegative)
    TableRead += ExtendedFltRoundOffset;
  return static_cast<AMDGPUFltRounds>(TableRead);
}

static constexpr uint32_t HWTowardZero = FP_ROUND_ROUND_TO_ZERO;
static constexpr uint32_t HWNearestTiesToEven = FP_ROUND_ROUND_TO_NEAREST;
static constexpr uint32_t HWTowardPositive = FP_ROUND_ROUND_TO_INF;
static constexpr uint32_t HWTowardNegative = FP_ROUND_ROUND_TO_NEGINF;

const uint64_t AMDGPU::FltRoundConversionTable =
    encodeFltRoundsTableSame(TowardZeroF32_TowardZeroF64, HWTowardZero) |
    encodeFltRoundsTableSame(NearestTiesToEvenF32_NearestTiesToEvenF64,
                             HWNearestTiesToEven) |
    encodeFltRoundsTableSame(TowardPositiveF32_TowardPositiveF64,
                             HWTowardPositive) |
    encodeFltRoundsTableSame(TowardNegativeF32_TowardNegativeF64,
                             HWTowardNegative) |

    encodeFltRoundsTable(TowardZeroF32_NearestTiesToEvenF64, HWTowardZero,
                         HWNearestTiesToEven) |
    encodeFltRoundsTable(TowardZeroF32_TowardPositiveF64, HWTowardZero,
                         HWTowardPositive) |
    encodeFltRoundsTable(TowardZeroF32_TowardNegativeF64, HWTowardZero,
                         HWTowardNegative) |

```
**EN:** This section contains concrete logic for decodeIndexFltRoundConversionTable. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 decodeIndexFltRoundConversionTable 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 90-112: Implementation details and local logic
```cpp
    encodeFltRoundsTable(NearestTiesToEvenF32_TowardZeroF64,
                         HWNearestTiesToEven, HWTowardZero) |
    encodeFltRoundsTable(NearestTiesToEvenF32_TowardPositiveF64,
                         HWNearestTiesToEven, HWTowardPositive) |
    encodeFltRoundsTable(NearestTiesToEvenF32_TowardNegativeF64,
                         HWNearestTiesToEven, HWTowardNegative) |

    encodeFltRoundsTable(TowardPositiveF32_TowardZeroF64, HWTowardPositive,
                         HWTowardZero) |
    encodeFltRoundsTable(TowardPositiveF32_NearestTiesToEvenF64,
                         HWTowardPositive, HWNearestTiesToEven) |
    encodeFltRoundsTable(TowardPositiveF32_TowardNegativeF64, HWTowardPositive,
                         HWTowardNegative) |

    encodeFltRoundsTable(TowardNegativeF32_TowardZeroF64, HWTowardNegative,
                         HWTowardZero) |
    encodeFltRoundsTable(TowardNegativeF32_NearestTiesToEvenF64,
                         HWTowardNegative, HWNearestTiesToEven) |
    encodeFltRoundsTable(TowardNegativeF32_TowardPositiveF64, HWTowardNegative,
                         HWTowardPositive);

// Verify evaluation of FltRoundConversionTable

```
**EN:** This section advances the file implementation by combining declarations, control flow, and helper operations that support the AMDGPU backend.
**CN:** 本节通过组合声明、控制流与辅助操作来推进文件实现，支撑 AMDGPU 后端的整体功能。

### Lines 113-145: Defines static_assert
```cpp
// If both modes are the same, should return the standard values.
static_assert(decodeIndexFltRoundConversionTable(getModeRegisterRoundMode(
                  HWTowardZero, HWTowardZero)) == AMDGPUFltRounds::TowardZero);
static_assert(decodeIndexFltRoundConversionTable(getModeRegisterRoundMode(
                  HWNearestTiesToEven, HWNearestTiesToEven)) ==
              AMDGPUFltRounds::NearestTiesToEven);
static_assert(decodeIndexFltRoundConversionTable(getModeRegisterRoundMode(
                  HWTowardPositive, HWTowardPositive)) ==
              AMDGPUFltRounds::TowardPositive);
static_assert(decodeIndexFltRoundConversionTable(getModeRegisterRoundMode(
                  HWTowardNegative, HWTowardNegative)) ==
              AMDGPUFltRounds::TowardNegative);

static_assert(decodeIndexFltRoundConversionTable(getModeRegisterRoundMode(
                  HWTowardZero, HWNearestTiesToEven)) ==
              TowardZeroF32_NearestTiesToEvenF64);
static_assert(decodeIndexFltRoundConversionTable(
                  getModeRegisterRoundMode(HWTowardZero, HWTowardPositive)) ==
              TowardZeroF32_TowardPositiveF64);
static_assert(decodeIndexFltRoundConversionTable(
                  getModeRegisterRoundMode(HWTowardZero, HWTowardNegative)) ==
              TowardZeroF32_TowardNegativeF64);

static_assert(decodeIndexFltRoundConversionTable(getModeRegisterRoundMode(
                  HWNearestTiesToEven, HWTowardZero)) ==
              NearestTiesToEvenF32_TowardZeroF64);
static_assert(decodeIndexFltRoundConversionTable(getModeRegisterRoundMode(
                  HWNearestTiesToEven, HWTowardPositive)) ==
              NearestTiesToEvenF32_TowardPositiveF64);
static_assert(decodeIndexFltRoundConversionTable(getModeRegisterRoundMode(
                  HWNearestTiesToEven, HWTowardNegative)) ==
              NearestTiesToEvenF32_TowardNegativeF64);

```
**EN:** This section contains concrete logic for static_assert. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 static_assert 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 146-176: Defines static_assert
```cpp
static_assert(decodeIndexFltRoundConversionTable(
                  getModeRegisterRoundMode(HWTowardPositive, HWTowardZero)) ==
              TowardPositiveF32_TowardZeroF64);
static_assert(decodeIndexFltRoundConversionTable(getModeRegisterRoundMode(
                  HWTowardPositive, HWNearestTiesToEven)) ==
              TowardPositiveF32_NearestTiesToEvenF64);
static_assert(decodeIndexFltRoundConversionTable(getModeRegisterRoundMode(
                  HWTowardPositive, HWTowardNegative)) ==
              TowardPositiveF32_TowardNegativeF64);

static_assert(decodeIndexFltRoundConversionTable(
                  getModeRegisterRoundMode(HWTowardNegative, HWTowardZero)) ==
              TowardNegativeF32_TowardZeroF64);
static_assert(decodeIndexFltRoundConversionTable(getModeRegisterRoundMode(
                  HWTowardNegative, HWNearestTiesToEven)) ==
              TowardNegativeF32_NearestTiesToEvenF64);
static_assert(decodeIndexFltRoundConversionTable(getModeRegisterRoundMode(
                  HWTowardNegative, HWTowardPositive)) ==
              TowardNegativeF32_TowardPositiveF64);

// Decode FLT_ROUNDS into the hardware value where the two rounding modes are
// the same and use a standard value
static constexpr uint64_t encodeFltRoundsToHWTableSame(uint32_t HWVal,
                                                       uint32_t FltRoundsVal) {
  if (FltRoundsVal > TowardNegative)
    FltRoundsVal -= ExtendedFltRoundOffset;

  return static_cast<uint64_t>(getModeRegisterRoundMode(HWVal, HWVal))
         << (FltRoundsVal << 2);
}

```
**EN:** This section contains concrete logic for static_assert. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 static_assert 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 177-210: Defines encodeFltRoundsToHWTable
```cpp
/// Decode FLT_ROUNDS into the hardware value where the two rounding modes
/// different and use an extended value.
static constexpr uint64_t encodeFltRoundsToHWTable(uint32_t HWF32Val,
                                                   uint32_t HWF64Val,
                                                   uint32_t FltRoundsVal) {
  if (FltRoundsVal > TowardNegative)
    FltRoundsVal -= ExtendedFltRoundOffset;
  return static_cast<uint64_t>(getModeRegisterRoundMode(HWF32Val, HWF64Val))
         << (FltRoundsVal << 2);
}

const uint64_t AMDGPU::FltRoundToHWConversionTable =
    encodeFltRoundsToHWTableSame(HWTowardZero, TowardZeroF32_TowardZeroF64) |
    encodeFltRoundsToHWTableSame(HWNearestTiesToEven,
                                 NearestTiesToEvenF32_NearestTiesToEvenF64) |
    encodeFltRoundsToHWTableSame(HWTowardPositive,
                                 TowardPositiveF32_TowardPositiveF64) |
    encodeFltRoundsToHWTableSame(HWTowardNegative,
                                 TowardNegativeF32_TowardNegativeF64) |

    encodeFltRoundsToHWTable(HWTowardZero, HWNearestTiesToEven,
                             TowardZeroF32_NearestTiesToEvenF64) |
    encodeFltRoundsToHWTable(HWTowardZero, HWTowardPositive,
                             TowardZeroF32_TowardPositiveF64) |
    encodeFltRoundsToHWTable(HWTowardZero, HWTowardNegative,
                             TowardZeroF32_TowardNegativeF64) |

    encodeFltRoundsToHWTable(HWNearestTiesToEven, HWTowardZero,
                             NearestTiesToEvenF32_TowardZeroF64) |
    encodeFltRoundsToHWTable(HWNearestTiesToEven, HWTowardPositive,
                             NearestTiesToEvenF32_TowardPositiveF64) |
    encodeFltRoundsToHWTable(HWNearestTiesToEven, HWTowardNegative,
                             NearestTiesToEvenF32_TowardNegativeF64) |

```
**EN:** This section contains concrete logic for encodeFltRoundsToHWTable. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 encodeFltRoundsToHWTable 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 211-244: Conditional logic and checks
```cpp
    encodeFltRoundsToHWTable(HWTowardPositive, HWTowardZero,
                             TowardPositiveF32_TowardZeroF64) |
    encodeFltRoundsToHWTable(HWTowardPositive, HWNearestTiesToEven,
                             TowardPositiveF32_NearestTiesToEvenF64) |
    encodeFltRoundsToHWTable(HWTowardPositive, HWTowardNegative,
                             TowardPositiveF32_TowardNegativeF64) |

    encodeFltRoundsToHWTable(HWTowardNegative, HWTowardZero,
                             TowardNegativeF32_TowardZeroF64) |
    encodeFltRoundsToHWTable(HWTowardNegative, HWNearestTiesToEven,
                             TowardNegativeF32_NearestTiesToEvenF64) |
    encodeFltRoundsToHWTable(HWTowardNegative, HWTowardPositive,
                             TowardNegativeF32_TowardPositiveF64);

/// Read the hardware rounding mode equivalent of a AMDGPUFltRounds value.
static constexpr uint32_t
decodeFltRoundToHWConversionTable(uint64_t FltRoundToHWConversionTable,
                                  uint32_t FltRounds) {
  uint32_t IndexVal = FltRounds;
  if (IndexVal > TowardNegative)
    IndexVal -= ExtendedFltRoundOffset;
  return (FltRoundToHWConversionTable >> (IndexVal << 2)) & 0xf;
}

uint32_t AMDGPU::decodeFltRoundToHWConversionTable(uint32_t FltRounds) {
  return ::decodeFltRoundToHWConversionTable(FltRoundToHWConversionTable,
                                             FltRounds);
}

static constexpr uint32_t decodeFltRoundToHW(uint32_t FltRounds) {
  return ::decodeFltRoundToHWConversionTable(FltRoundToHWConversionTable,
                                             FltRounds);
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `AMDGPU::decodeFltRoundToHWConversionTable`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`AMDGPU::decodeFltRoundToHWConversionTable`。

### Lines 245-277: Defines static_assert
```cpp
// Verify evaluation of FltRoundToHWConversionTable

static_assert(decodeFltRoundToHW(AMDGPUFltRounds::TowardZero) ==
              getModeRegisterRoundMode(HWTowardZero, HWTowardZero));
static_assert(decodeFltRoundToHW(AMDGPUFltRounds::NearestTiesToEven) ==
              getModeRegisterRoundMode(HWNearestTiesToEven,
                                       HWNearestTiesToEven));
static_assert(decodeFltRoundToHW(AMDGPUFltRounds::TowardPositive) ==
              getModeRegisterRoundMode(HWTowardPositive, HWTowardPositive));
static_assert(decodeFltRoundToHW(AMDGPUFltRounds::TowardNegative) ==
              getModeRegisterRoundMode(HWTowardNegative, HWTowardNegative));

static_assert(decodeFltRoundToHW(NearestTiesToEvenF32_TowardPositiveF64) ==
              getModeRegisterRoundMode(HWNearestTiesToEven, HWTowardPositive));
static_assert(decodeFltRoundToHW(NearestTiesToEvenF32_TowardNegativeF64) ==
              getModeRegisterRoundMode(HWNearestTiesToEven, HWTowardNegative));
static_assert(decodeFltRoundToHW(NearestTiesToEvenF32_TowardZeroF64) ==
              getModeRegisterRoundMode(HWNearestTiesToEven, HWTowardZero));

static_assert(decodeFltRoundToHW(TowardPositiveF32_NearestTiesToEvenF64) ==
              getModeRegisterRoundMode(HWTowardPositive, HWNearestTiesToEven));
static_assert(decodeFltRoundToHW(TowardPositiveF32_TowardNegativeF64) ==
              getModeRegisterRoundMode(HWTowardPositive, HWTowardNegative));
static_assert(decodeFltRoundToHW(TowardPositiveF32_TowardZeroF64) ==
              getModeRegisterRoundMode(HWTowardPositive, HWTowardZero));

static_assert(decodeFltRoundToHW(TowardNegativeF32_NearestTiesToEvenF64) ==
              getModeRegisterRoundMode(HWTowardNegative, HWNearestTiesToEven));
static_assert(decodeFltRoundToHW(TowardNegativeF32_TowardPositiveF64) ==
              getModeRegisterRoundMode(HWTowardNegative, HWTowardPositive));
static_assert(decodeFltRoundToHW(TowardNegativeF32_TowardZeroF64) ==
              getModeRegisterRoundMode(HWTowardNegative, HWTowardZero));

```
**EN:** This section contains concrete logic for static_assert. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 static_assert 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

### Lines 278-283: Defines static_assert
```cpp
static_assert(decodeFltRoundToHW(TowardZeroF32_NearestTiesToEvenF64) ==
              getModeRegisterRoundMode(HWTowardZero, HWNearestTiesToEven));
static_assert(decodeFltRoundToHW(TowardZeroF32_TowardPositiveF64) ==
              getModeRegisterRoundMode(HWTowardZero, HWTowardPositive));
static_assert(decodeFltRoundToHW(TowardZeroF32_TowardNegativeF64) ==
              getModeRegisterRoundMode(HWTowardZero, HWTowardNegative));
```
**EN:** This section contains concrete logic for static_assert. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline.
**CN:** 本节包含与 static_assert 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `SIModeRegisterDefaults::SIModeRegisterDefaults`, `AMDGPU::decodeFltRoundToHWConversionTable`
- **Main themes / 核心主题**: register management / 寄存器管理; subtarget modeling / 子目标建模; feature description / 特性描述
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"SIModeRegisterDefaults.h"`
- `"GCNSubtarget.h"`
