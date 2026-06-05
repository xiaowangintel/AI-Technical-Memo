# XCOFF.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/lib/BinaryFormat/XCOFF.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements helpers for concrete object-file and binary metadata formats.
  - **CN**: 实现具体目标文件与二进制元数据格式的辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16
```cpp
//===-- llvm/BinaryFormat/XCOFF.cpp - The XCOFF file format -----*- C++/-*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/BinaryFormat/XCOFF.h"
#include "llvm/ADT/SmallString.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/ADT/StringSwitch.h"
#include "llvm/Support/Errc.h"
#include "llvm/Support/Error.h"
#include "llvm/TargetParser/PPCTargetParser.h"

```
- **EN**: Pulls in the headers needed by this translation unit, including `llvm/BinaryFormat/XCOFF.h`, `llvm/ADT/SmallString.h`, `llvm/ADT/StringRef.h`, `llvm/ADT/StringSwitch.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `llvm/BinaryFormat/XCOFF.h`, `llvm/ADT/SmallString.h`, `llvm/ADT/StringRef.h`, `llvm/ADT/StringSwitch.h`。

### Lines 17-36
```cpp
using namespace llvm;

#define SMC_CASE(A)                                                            \
  case XCOFF::XMC_##A:                                                         \
    return #A;
StringRef XCOFF::getMappingClassString(XCOFF::StorageMappingClass SMC) {
  switch (SMC) {
    SMC_CASE(PR)
    SMC_CASE(RO)
    SMC_CASE(DB)
    SMC_CASE(GL)
    SMC_CASE(XO)
    SMC_CASE(SV)
    SMC_CASE(SV64)
    SMC_CASE(SV3264)
    SMC_CASE(TI)
    SMC_CASE(TB)
    SMC_CASE(RW)
    SMC_CASE(TC0)
    SMC_CASE(TC)
```
- **EN**: Defines preprocessor-controlled structure, feature gates, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、特性开关或编译期常量。

### Lines 37-47
```cpp
    SMC_CASE(TD)
    SMC_CASE(DS)
    SMC_CASE(UA)
    SMC_CASE(BS)
    SMC_CASE(UC)
    SMC_CASE(TL)
    SMC_CASE(UL)
    SMC_CASE(TE)
#undef SMC_CASE
  }

```
- **EN**: Implements logic around `SMC_CASE`.
- **CN**: 围绕 `SMC_CASE` 实现具体逻辑。

### Lines 48-67
```cpp
  // TODO: need to add a test case for "Unknown" and other SMC.
  return "Unknown";
}

#define RELOC_CASE(A)                                                          \
  case XCOFF::A:                                                               \
    return #A;
StringRef XCOFF::getRelocationTypeString(XCOFF::RelocationType Type) {
  switch (Type) {
    RELOC_CASE(R_POS)
    RELOC_CASE(R_RL)
    RELOC_CASE(R_RLA)
    RELOC_CASE(R_NEG)
    RELOC_CASE(R_REL)
    RELOC_CASE(R_TOC)
    RELOC_CASE(R_TRL)
    RELOC_CASE(R_TRLA)
    RELOC_CASE(R_GL)
    RELOC_CASE(R_TCL)
    RELOC_CASE(R_REF)
```
- **EN**: Defines preprocessor-controlled structure, feature gates, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、特性开关或编译期常量。

### Lines 68-84
```cpp
    RELOC_CASE(R_BA)
    RELOC_CASE(R_BR)
    RELOC_CASE(R_RBA)
    RELOC_CASE(R_RBR)
    RELOC_CASE(R_TLS)
    RELOC_CASE(R_TLS_IE)
    RELOC_CASE(R_TLS_LD)
    RELOC_CASE(R_TLS_LE)
    RELOC_CASE(R_TLSM)
    RELOC_CASE(R_TLSML)
    RELOC_CASE(R_TOCU)
    RELOC_CASE(R_TOCL)
  }
  return "Unknown";
}
#undef RELOC_CASE

```
- **EN**: Implements logic around `RELOC_CASE`.
- **CN**: 围绕 `RELOC_CASE` 实现具体逻辑。

### Lines 85-104
```cpp
#define LANG_CASE(A)                                                           \
  case XCOFF::TracebackTable::A:                                               \
    return #A;

StringRef XCOFF::getNameForTracebackTableLanguageId(
    XCOFF::TracebackTable::LanguageID LangId) {
  switch (LangId) {
    LANG_CASE(C)
    LANG_CASE(Fortran)
    LANG_CASE(Pascal)
    LANG_CASE(Ada)
    LANG_CASE(PL1)
    LANG_CASE(Basic)
    LANG_CASE(Lisp)
    LANG_CASE(Cobol)
    LANG_CASE(Modula2)
    LANG_CASE(Rpg)
    LANG_CASE(PL8)
    LANG_CASE(Assembly)
    LANG_CASE(Java)
```
- **EN**: Defines preprocessor-controlled structure, feature gates, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、特性开关或编译期常量。

### Lines 105-124
```cpp
    LANG_CASE(ObjectiveC)
    LANG_CASE(CPlusPlus)
  }
  return "Unknown";
}
#undef LANG_CASE

XCOFF::CFileCpuId XCOFF::getCpuID(StringRef CPUName) {
  StringRef CPU = PPC::normalizeCPUName(CPUName);
  return StringSwitch<XCOFF::CFileCpuId>(CPU)
      .Cases({"generic", "COM"}, XCOFF::TCPU_COM)
      .Case("601", XCOFF::TCPU_601)
      .Cases({"602", "603", "603e", "603ev"}, XCOFF::TCPU_603)
      .Cases({"604", "604e"}, XCOFF::TCPU_604)
      .Case("620", XCOFF::TCPU_620)
      .Case("970", XCOFF::TCPU_970)
      .Cases({"a2", "g3", "g4", "g5", "e500"}, XCOFF::TCPU_COM)
      .Cases({"pwr3", "pwr4"}, XCOFF::TCPU_COM)
      .Cases({"pwr5", "PWR5"}, XCOFF::TCPU_PWR5)
      .Cases({"pwr5x", "PWR5X"}, XCOFF::TCPU_PWR5X)
```
- **EN**: Implements logic around `LANG_CASE`, `getCpuID`, `normalizeCPUName`, `CFileCpuId>`, and 2 more symbols; this block applies object-format-specific rules.
- **CN**: 围绕 `LANG_CASE`, `getCpuID`, `normalizeCPUName`, `CFileCpuId>`, and 2 more symbols 实现具体逻辑；该代码块应用目标文件格式专用规则。

### Lines 125-137
```cpp
      .Cases({"pwr6", "PWR6"}, XCOFF::TCPU_PWR6)
      .Cases({"pwr6x", "PWR6E"}, XCOFF::TCPU_PWR6E)
      .Cases({"pwr7", "PWR7"}, XCOFF::TCPU_PWR7)
      .Cases({"pwr8", "PWR8"}, XCOFF::TCPU_PWR8)
      .Cases({"pwr9", "PWR9"}, XCOFF::TCPU_PWR9)
      .Cases({"pwr10", "PWR10"}, XCOFF::TCPU_PWR10)
      .Cases({"ppc", "PPC", "ppc32", "ppc64"}, XCOFF::TCPU_COM)
      .Case("ppc64le", XCOFF::TCPU_PWR8)
      .Case("future", XCOFF::TCPU_PWR10)
      .Cases({"any", "ANY"}, XCOFF::TCPU_ANY)
      .Default(XCOFF::TCPU_INVALID);
}

```
- **EN**: Implements logic around `Cases`, `Case`, `Default`; this block applies object-format-specific rules.
- **CN**: 围绕 `Cases`, `Case`, `Default` 实现具体逻辑；该代码块应用目标文件格式专用规则。

### Lines 138-157
```cpp
#define TCPU_CASE(A)                                                           \
  case XCOFF::TCPU_##A:                                                        \
    return #A;
StringRef XCOFF::getTCPUString(XCOFF::CFileCpuId TCPU) {
  switch (TCPU) {
    TCPU_CASE(INVALID)
    TCPU_CASE(PPC)
    TCPU_CASE(PPC64)
    TCPU_CASE(COM)
    TCPU_CASE(PWR)
    TCPU_CASE(ANY)
    TCPU_CASE(601)
    TCPU_CASE(603)
    TCPU_CASE(604)
    TCPU_CASE(620)
    TCPU_CASE(A35)
    TCPU_CASE(PWR5)
    TCPU_CASE(970)
    TCPU_CASE(PWR6)
    TCPU_CASE(PWR5X)
```
- **EN**: Defines preprocessor-controlled structure, feature gates, or compile-time constants.
- **CN**: 定义受预处理器控制的结构、特性开关或编译期常量。

### Lines 158-168
```cpp
    TCPU_CASE(PWR6E)
    TCPU_CASE(PWR7)
    TCPU_CASE(PWR8)
    TCPU_CASE(PWR9)
    TCPU_CASE(PWR10)
    TCPU_CASE(PWRX)
  }
  return "INVALID";
}
#undef TCPU_CASE

```
- **EN**: Implements logic around `TCPU_CASE`.
- **CN**: 围绕 `TCPU_CASE` 实现具体逻辑。

### Lines 169-178
```cpp
Expected<SmallString<32>> XCOFF::parseParmsType(uint32_t Value,
                                                unsigned FixedParmsNum,
                                                unsigned FloatingParmsNum) {
  SmallString<32> ParmsType;
  int Bits = 0;
  unsigned ParsedFixedNum = 0;
  unsigned ParsedFloatingNum = 0;
  unsigned ParsedNum = 0;
  unsigned ParmsNum = FixedParmsNum + FloatingParmsNum;

```
- **EN**: Implements logic around `parseParmsType`; this block parses or classifies structured input; applies object-format-specific rules.
- **CN**: 围绕 `parseParmsType` 实现具体逻辑；该代码块解析或分类结构化输入，并应用目标文件格式专用规则。

### Lines 179-198
```cpp
  // In the function PPCFunctionInfo::getParmsType(), when there are no vector
  // parameters, the 31st bit of ParmsType is always zero even if it indicates a
  // floating point parameter. The parameter type information is lost. There
  // are only 8 GPRs used for parameters passing, the floating parameters
  // also occupy GPRs if there are available, so the 31st bit can never be a
  // fixed parameter. At the same time, we also do not know whether the zero of
  // the 31st bit indicates a float or double parameter type here. Therefore, we
  // ignore the 31st bit.
  while (Bits < 31 && ParsedNum < ParmsNum) {
    if (++ParsedNum > 1)
      ParmsType += ", ";
    if ((Value & TracebackTable::ParmTypeIsFloatingBit) == 0) {
      // Fixed parameter type.
      ParmsType += "i";
      ++ParsedFixedNum;
      Value <<= 1;
      ++Bits;
    } else {
      if ((Value & TracebackTable::ParmTypeFloatingIsDoubleBit) == 0)
        // Float parameter type.
```
- **EN**: Contains supporting implementation details for the surrounding LLVM library component.
- **CN**: 包含周边 LLVM 库组件所需的辅助实现细节。

### Lines 199-208
```cpp
        ParmsType += "f";
      else
        // Double parameter type.
        ParmsType += "d";
      ++ParsedFloatingNum;
      Value <<= 2;
      Bits += 2;
    }
  }

```
- **EN**: Contains supporting implementation details for the surrounding LLVM library component.
- **CN**: 包含周边 LLVM 库组件所需的辅助实现细节。

### Lines 209-220
```cpp
  // We have more parameters than the 32 Bits could encode.
  if (ParsedNum < ParmsNum)
    ParmsType += ", ...";

  if (Value != 0u || ParsedFixedNum > FixedParmsNum ||
      ParsedFloatingNum > FloatingParmsNum)
    return createStringError(errc::invalid_argument,
                             "ParmsType encodes can not map to ParmsNum "
                             "parameters in parseParmsType.");
  return ParmsType;
}

```
- **EN**: Implements logic around `createStringError`; this block propagates recoverable errors through LLVM error utilities; parses or classifies structured input.
- **CN**: 围绕 `createStringError` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并解析或分类结构化输入。

### Lines 221-236
```cpp
SmallString<32> XCOFF::getExtendedTBTableFlagString(uint8_t Flag) {
  SmallString<32> Res;

  if (Flag & ExtendedTBTableFlag::TB_OS1)
    Res += "TB_OS1 ";
  if (Flag & ExtendedTBTableFlag::TB_RESERVED)
    Res += "TB_RESERVED ";
  if (Flag & ExtendedTBTableFlag::TB_SSP_CANARY)
    Res += "TB_SSP_CANARY ";
  if (Flag & ExtendedTBTableFlag::TB_OS2)
    Res += "TB_OS2 ";
  if (Flag & ExtendedTBTableFlag::TB_EH_INFO)
    Res += "TB_EH_INFO ";
  if (Flag & ExtendedTBTableFlag::TB_LONGTBTABLE2)
    Res += "TB_LONGTBTABLE2 ";

```
- **EN**: Implements logic around `getExtendedTBTableFlagString`; this block applies object-format-specific rules.
- **CN**: 围绕 `getExtendedTBTableFlagString` 实现具体逻辑；该代码块应用目标文件格式专用规则。

### Lines 237-251
```cpp
  // Two of the bits that haven't got used in the mask.
  if (Flag & 0x06)
    Res += "Unknown ";

  // Pop the last space.
  Res.pop_back();
  return Res;
}

Expected<SmallString<32>>
XCOFF::parseParmsTypeWithVecInfo(uint32_t Value, unsigned FixedParmsNum,
                                 unsigned FloatingParmsNum,
                                 unsigned VectorParmsNum) {
  SmallString<32> ParmsType;

```
- **EN**: Implements logic around `pop_back`, `parseParmsTypeWithVecInfo`; this block parses or classifies structured input; applies object-format-specific rules.
- **CN**: 围绕 `pop_back`, `parseParmsTypeWithVecInfo` 实现具体逻辑；该代码块解析或分类结构化输入，并应用目标文件格式专用规则。

### Lines 252-261
```cpp
  unsigned ParsedFixedNum = 0;
  unsigned ParsedFloatingNum = 0;
  unsigned ParsedVectorNum = 0;
  unsigned ParsedNum = 0;
  unsigned ParmsNum = FixedParmsNum + FloatingParmsNum + VectorParmsNum;

  for (int Bits = 0; Bits < 32 && ParsedNum < ParmsNum; Bits += 2) {
    if (++ParsedNum > 1)
      ParmsType += ", ";

```
- **EN**: Contains supporting implementation details for the surrounding LLVM library component.
- **CN**: 包含周边 LLVM 库组件所需的辅助实现细节。

### Lines 262-281
```cpp
    switch (Value & TracebackTable::ParmTypeMask) {
    case TracebackTable::ParmTypeIsFixedBits:
      ParmsType += "i";
      ++ParsedFixedNum;
      break;
    case TracebackTable::ParmTypeIsVectorBits:
      ParmsType += "v";
      ++ParsedVectorNum;
      break;
    case TracebackTable::ParmTypeIsFloatingBits:
      ParmsType += "f";
      ++ParsedFloatingNum;
      break;
    case TracebackTable::ParmTypeIsDoubleBits:
      ParmsType += "d";
      ++ParsedFloatingNum;
      break;
    default:
      assert(false && "Unrecognized bits in ParmsType.");
    }
```
- **EN**: Implements logic around `assert`; this block uses `switch`-style dispatch; parses or classifies structured input.
- **CN**: 围绕 `assert` 实现具体逻辑；该代码块使用 `switch` 风格分派，并解析或分类结构化输入。

### Lines 282-295
```cpp
    Value <<= 2;
  }

  // We have more parameters than the 32 Bits could encode.
  if (ParsedNum < ParmsNum)
    ParmsType += ", ...";

  if (Value != 0u || ParsedFixedNum > FixedParmsNum ||
      ParsedFloatingNum > FloatingParmsNum || ParsedVectorNum > VectorParmsNum)
    return createStringError(
        errc::invalid_argument,
        "ParmsType encodes can not map to ParmsNum parameters "
        "in parseParmsTypeWithVecInfo.");

```
- **EN**: Implements logic around `createStringError`; this block propagates recoverable errors through LLVM error utilities; parses or classifies structured input.
- **CN**: 围绕 `createStringError` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并解析或分类结构化输入。

### Lines 296-310
```cpp
  return ParmsType;
}

Expected<SmallString<32>> XCOFF::parseVectorParmsType(uint32_t Value,
                                                      unsigned ParmsNum) {
  SmallString<32> ParmsType;
  unsigned ParsedNum = 0;
  for (int Bits = 0; ParsedNum < ParmsNum && Bits < 32; Bits += 2) {
    if (++ParsedNum > 1)
      ParmsType += ", ";
    switch (Value & TracebackTable::ParmTypeMask) {
    case TracebackTable::ParmTypeIsVectorCharBit:
      ParmsType += "vc";
      break;

```
- **EN**: Implements logic around `parseVectorParmsType`; this block uses `switch`-style dispatch; parses or classifies structured input; applies object-format-specific rules.
- **CN**: 围绕 `parseVectorParmsType` 实现具体逻辑；该代码块使用 `switch` 风格分派，并解析或分类结构化输入，并应用目标文件格式专用规则。

### Lines 311-323
```cpp
    case TracebackTable::ParmTypeIsVectorShortBit:
      ParmsType += "vs";
      break;

    case TracebackTable::ParmTypeIsVectorIntBit:
      ParmsType += "vi";
      break;

    case TracebackTable::ParmTypeIsVectorFloatBit:
      ParmsType += "vf";
      break;
    }

```
- **EN**: Contains supporting implementation details for the surrounding LLVM library component.
- **CN**: 包含周边 LLVM 库组件所需的辅助实现细节。

### Lines 324-336
```cpp
    Value <<= 2;
  }

  // We have more parameters than the 32 Bits could encode.
  if (ParsedNum < ParmsNum)
    ParmsType += ", ...";

  if (Value != 0u)
    return createStringError(errc::invalid_argument,
                             "ParmsType encodes more than ParmsNum parameters "
                             "in parseVectorParmsType.");
  return ParmsType;
}
```
- **EN**: Implements logic around `createStringError`; this block propagates recoverable errors through LLVM error utilities; parses or classifies structured input.
- **CN**: 围绕 `createStringError` 实现具体逻辑；该代码块通过 LLVM 错误工具传播可恢复错误，并解析或分类结构化输入。

## Key Concepts / 关键概念

- **Binary metadata formats / 二进制元数据格式**:
  - **EN**: Describes constants, records, and helpers for concrete file formats.
  - **CN**: 描述具体文件格式的常量、记录与辅助逻辑。

## Dependencies / 依赖关系

- **Direct LLVM/local includes / 直接的 LLVM/本地包含**: `llvm/BinaryFormat/XCOFF.h`, `llvm/ADT/SmallString.h`, `llvm/ADT/StringRef.h`, `llvm/ADT/StringSwitch.h`, `llvm/Support/Errc.h`, `llvm/Support/Error.h`, `llvm/TargetParser/PPCTargetParser.h`
- **Subsystem categories / 子系统类别**: LLVM ADT containers and utility types / LLVM ADT 容器与工具类型 (3), support-library helpers / Support 库辅助功能 (2), binary-format constants and record definitions / 二进制格式常量与记录定义 (1)
