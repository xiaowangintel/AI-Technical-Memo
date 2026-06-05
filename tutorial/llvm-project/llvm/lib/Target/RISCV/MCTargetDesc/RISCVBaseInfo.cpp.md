# RISCVBaseInfo.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/MCTargetDesc/RISCVBaseInfo.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements shared RISC-V target constants, enums, and helper utilities. / 实现共享的 RISC-V 目标常量、枚举与辅助工具。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Commentary and design intent / 注释与设计意图
```cpp
//===-- RISCVBaseInfo.cpp - Top level definitions for RISC-V MC -----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains small standalone enum definitions for the RISC-V target
// useful for the compiler back-end and the MC libraries.
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 13-22: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp

#include "RISCVBaseInfo.h"
#include "RISCVInstrInfo.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCRegisterInfo.h"
#include "llvm/MC/MCSubtargetInfo.h"
#include "llvm/Support/raw_ostream.h"
#include "llvm/TargetParser/TargetParser.h"
#include "llvm/TargetParser/Triple.h"
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 23-36: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp
namespace llvm {

extern const SubtargetFeatureKV RISCVFeatureKV[RISCV::NumSubtargetFeatures];

namespace RISCVSysReg {
#define GET_SysRegsList_IMPL
#include "RISCVGenSearchableTables.inc"
} // namespace RISCVSysReg

namespace RISCVInsnOpcode {
#define GET_RISCVOpcodesList_IMPL
#include "RISCVGenSearchableTables.inc"
} // namespace RISCVInsnOpcode
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 37-55: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp
namespace RISCVVInversePseudosTable {
using namespace RISCV;
#define GET_RISCVVInversePseudosTable_IMPL
#include "RISCVGenSearchableTables.inc"
} // namespace RISCVVInversePseudosTable

namespace RISCV {
#define GET_RISCVVSSEGTable_IMPL
#define GET_RISCVVLSEGTable_IMPL
#define GET_RISCVVLXSEGTable_IMPL
#define GET_RISCVVSXSEGTable_IMPL
#define GET_RISCVVLETable_IMPL
#define GET_RISCVVSETable_IMPL
#define GET_RISCVVLXTable_IMPL
#define GET_RISCVVSXTable_IMPL
#define GET_RISCVNDSVLNTable_IMPL
#include "RISCVGenSearchableTables.inc"
} // namespace RISCV
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 56-75: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
namespace RISCVABI {
ABI computeTargetABI(const Triple &TT, const FeatureBitset &FeatureBits,
                     StringRef ABIName) {
  auto TargetABI = getTargetABI(ABIName);
  bool IsRV64 = TT.isArch64Bit();
  bool IsRVE = FeatureBits[RISCV::FeatureStdExtE];

  if (!ABIName.empty() && TargetABI == ABI_Unknown) {
    errs()
        << "'" << ABIName
        << "' is not a recognized ABI for this target (ignoring target-abi)\n";
  } else if (ABIName.starts_with("ilp32") && IsRV64) {
    errs() << "32-bit ABIs are not supported for 64-bit targets (ignoring "
              "target-abi)\n";
    TargetABI = ABI_Unknown;
  } else if (ABIName.starts_with("lp64") && !IsRV64) {
    errs() << "64-bit ABIs are not supported for 32-bit targets (ignoring "
              "target-abi)\n";
    TargetABI = ABI_Unknown;
  } else if (!IsRV64 && IsRVE && TargetABI != ABI_ILP32E &&
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 76-87: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
             TargetABI != ABI_Unknown) {
    // TODO: move this checking to RISCVTargetLowering and RISCVAsmParser
    errs()
        << "Only the ilp32e ABI is supported for RV32E (ignoring target-abi)\n";
    TargetABI = ABI_Unknown;
  } else if (IsRV64 && IsRVE && TargetABI != ABI_LP64E &&
             TargetABI != ABI_Unknown) {
    // TODO: move this checking to RISCVTargetLowering and RISCVAsmParser
    errs()
        << "Only the lp64e ABI is supported for RV64E (ignoring target-abi)\n";
    TargetABI = ABI_Unknown;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 88-102: Definitions and supporting logic / 定义与支撑逻辑
```cpp

  if ((TargetABI == RISCVABI::ABI::ABI_ILP32E ||
       (TargetABI == ABI_Unknown && IsRVE && !IsRV64)) &&
      FeatureBits[RISCV::FeatureStdExtD])
    reportFatalUsageError("ILP32E cannot be used with the D ISA extension");

  if (TargetABI != ABI_Unknown)
    return TargetABI;

  // If no explicit ABI is given, try to compute the default ABI.
  auto ISAInfo = RISCVFeatures::parseFeatureBits(IsRV64, FeatureBits);
  if (!ISAInfo)
    reportFatalUsageError(ISAInfo.takeError());
  return getTargetABI((*ISAInfo)->computeDefaultABI());
}
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 103-122: Function implementation: getTargetABI / 函数实现：getTargetABI
```cpp

ABI getTargetABI(StringRef ABIName) {
  auto TargetABI = StringSwitch<ABI>(ABIName)
                       .Case("ilp32", ABI_ILP32)
                       .Case("ilp32f", ABI_ILP32F)
                       .Case("ilp32d", ABI_ILP32D)
                       .Case("ilp32e", ABI_ILP32E)
                       .Case("il32pc64", ABI_IL32PC64)
                       .Case("il32pc64f", ABI_IL32PC64F)
                       .Case("il32pc64d", ABI_IL32PC64D)
                       .Case("il32pc64e", ABI_IL32PC64E)
                       .Case("lp64", ABI_LP64)
                       .Case("lp64f", ABI_LP64F)
                       .Case("lp64d", ABI_LP64D)
                       .Case("lp64e", ABI_LP64E)
                       .Case("l64pc128", ABI_L64PC128)
                       .Case("l64pc128f", ABI_L64PC128F)
                       .Case("l64pc128d", ABI_L64PC128D)
                       .Default(ABI_Unknown);
  return TargetABI;
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 123-132: Function implementation: getBPReg / 函数实现：getBPReg
```cpp
}

// To avoid the BP value clobbered by a function call, we need to choose a
// callee saved register to save the value. RV32E only has X8 and X9 as callee
// saved registers and X8 will be used as fp. So we choose X9 as bp.
MCRegister getBPReg() { return RISCV::X9; }

// Returns the register holding shadow call stack pointer.
MCRegister getSCSPReg() { return RISCV::X3; }
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 133-145: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
} // namespace RISCVABI

namespace RISCVFeatures {

void validate(const Triple &TT, const FeatureBitset &FeatureBits) {
  if (TT.isArch64Bit() && !FeatureBits[RISCV::Feature64Bit])
    reportFatalUsageError("RV64 target requires an RV64 CPU");
  if (!TT.isArch64Bit() && !FeatureBits[RISCV::Feature32Bit])
    reportFatalUsageError("RV32 target requires an RV32 CPU");
  if (FeatureBits[RISCV::Feature32Bit] &&
      FeatureBits[RISCV::Feature64Bit])
    reportFatalUsageError("RV32 and RV64 can't be combined");
}
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 146-156: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

llvm::Expected<std::unique_ptr<RISCVISAInfo>>
parseFeatureBits(bool IsRV64, const FeatureBitset &FeatureBits) {
  unsigned XLen = IsRV64 ? 64 : 32;
  std::vector<std::string> FeatureVector;
  // Convert FeatureBitset to FeatureVector.
  for (auto Feature : RISCVFeatureKV) {
    if (FeatureBits[Feature.Value] &&
        llvm::RISCVISAInfo::isSupportedExtensionFeature(Feature.Key))
      FeatureVector.push_back(std::string("+") + Feature.Key);
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 157-166: File prologue and imported dependencies / 文件前言与导入依赖
```cpp
  return llvm::RISCVISAInfo::parseFeatures(XLen, FeatureVector);
}

} // namespace RISCVFeatures

// Include the auto-generated portion of the compress emitter.
#define GEN_UNCOMPRESS_INSTR
#define GEN_COMPRESS_INSTR
#include "RISCVGenCompressInstEmitter.inc"
```
**EN:** This opening block combines banner comments with the first wave of includes, giving readers both context and the main compile-time dependencies.

**CN:** 该开头区段同时包含说明性注释与首批 include，让读者先看到文件定位，再看到主要编译期依赖。

### Lines 167-176: Function implementation: RISCVRVC::compress / 函数实现：RISCVRVC::compress
```cpp
bool RISCVRVC::compress(MCInst &OutInst, const MCInst &MI,
                        const MCSubtargetInfo &STI) {
  return compressInst(OutInst, MI, STI);
}

bool RISCVRVC::uncompress(MCInst &OutInst, const MCInst &MI,
                          const MCSubtargetInfo &STI) {
  return uncompressInst(OutInst, MI, STI);
}
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 177-189: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
// Lookup table for fli.s for entries 2-31.
static constexpr std::pair<uint8_t, uint8_t> LoadFP32ImmArr[] = {
    {0b01101111, 0b00}, {0b01110000, 0b00}, {0b01110111, 0b00},
    {0b01111000, 0b00}, {0b01111011, 0b00}, {0b01111100, 0b00},
    {0b01111101, 0b00}, {0b01111101, 0b01}, {0b01111101, 0b10},
    {0b01111101, 0b11}, {0b01111110, 0b00}, {0b01111110, 0b01},
    {0b01111110, 0b10}, {0b01111110, 0b11}, {0b01111111, 0b00},
    {0b01111111, 0b01}, {0b01111111, 0b10}, {0b01111111, 0b11},
    {0b10000000, 0b00}, {0b10000000, 0b01}, {0b10000000, 0b10},
    {0b10000001, 0b00}, {0b10000010, 0b00}, {0b10000011, 0b00},
    {0b10000110, 0b00}, {0b10000111, 0b00}, {0b10001110, 0b00},
    {0b10001111, 0b00}, {0b11111111, 0b00}, {0b11111111, 0b10},
};
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 190-200: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

int RISCVLoadFPImm::getLoadFPImm(APFloat FPImm) {
  assert((&FPImm.getSemantics() == &APFloat::IEEEsingle() ||
          &FPImm.getSemantics() == &APFloat::IEEEdouble() ||
          &FPImm.getSemantics() == &APFloat::IEEEhalf()) &&
         "Unexpected semantics");

  // Handle the minimum normalized value which is different for each type.
  if (FPImm.isSmallestNormalized() && !FPImm.isNegative())
    return 1;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 201-212: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  // Convert to single precision to use its lookup table.
  bool LosesInfo;
  APFloat::opStatus Status = FPImm.convert(
      APFloat::IEEEsingle(), APFloat::rmNearestTiesToEven, &LosesInfo);
  if (Status != APFloat::opOK || LosesInfo)
    return -1;

  APInt Imm = FPImm.bitcastToAPInt();

  if (Imm.extractBitsAsZExtValue(21, 0) != 0)
    return -1;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 213-224: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  bool Sign = Imm.extractBitsAsZExtValue(1, 31);
  uint8_t Mantissa = Imm.extractBitsAsZExtValue(2, 21);
  uint8_t Exp = Imm.extractBitsAsZExtValue(8, 23);

  auto EMI = llvm::lower_bound(LoadFP32ImmArr, std::make_pair(Exp, Mantissa));
  if (EMI == std::end(LoadFP32ImmArr) || EMI->first != Exp ||
      EMI->second != Mantissa)
    return -1;

  // Table doesn't have entry 0 or 1.
  int Entry = std::distance(std::begin(LoadFP32ImmArr), EMI) + 2;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 225-234: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  // The only legal negative value is -1.0(entry 0). 1.0 is entry 16.
  if (Sign) {
    if (Entry == 16)
      return 0;
    return -1;
  }

  return Entry;
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 235-244: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
float RISCVLoadFPImm::getFPImm(unsigned Imm) {
  assert(Imm != 1 && Imm != 30 && Imm != 31 && "Unsupported immediate");

  // Entry 0 is -1.0, the only negative value. Entry 16 is 1.0.
  uint32_t Sign = 0;
  if (Imm == 0) {
    Sign = 0b1;
    Imm = 16;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 245-262: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  uint32_t Exp = LoadFP32ImmArr[Imm - 2].first;
  uint32_t Mantissa = LoadFP32ImmArr[Imm - 2].second;

  uint32_t I = Sign << 31 | Exp << 23 | Mantissa << 21;
  return bit_cast<float>(I);
}

void RISCVZC::printRegList(unsigned RlistEncode, raw_ostream &OS) {
  assert(RlistEncode >= RLISTENCODE::RA &&
         RlistEncode <= RLISTENCODE::RA_S0_S11 && "Invalid Rlist");
  OS << "{ra";
  if (RlistEncode > RISCVZC::RA) {
    OS << ", s0";
    if (RlistEncode == RISCVZC::RA_S0_S11)
      OS << "-s11";
    else if (RlistEncode > RISCVZC::RA_S0 && RlistEncode <= RISCVZC::RA_S0_S11)
      OS << "-s" << (RlistEncode - RISCVZC::RA_S0);
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 263-266: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
  OS << "}";
}

} // namespace llvm
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

## Key Concepts / 关键概念
- **MC layer target description** / **MC 层目标描述**

## Dependencies / 依赖关系
- `RISCVBaseInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVInstrInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCInst.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCRegisterInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCSubtargetInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/Support/raw_ostream.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/TargetParser/TargetParser.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/TargetParser/Triple.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVGenSearchableTables.inc` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVGenCompressInstEmitter.inc` — Directly referenced by this file. / 该文件直接引用的依赖。
