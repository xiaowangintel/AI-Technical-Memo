# RISCVCustomBehaviour.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/MCA/RISCVCustomBehaviour.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements RISC-V-specific machine code analysis behavior for LLVM MCA. / 实现LLVM MCA 的 RISC-V 专用机器码分析行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Commentary and design intent / 注释与设计意图
```cpp
//===------------------- RISCVCustomBehaviour.cpp ---------------*-C++ -* -===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
/// \file
///
/// This file implements methods from the RISCVCustomBehaviour class.
///
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 13-22: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp

#include "RISCVCustomBehaviour.h"
#include "MCTargetDesc/RISCVMCTargetDesc.h"
#include "RISCV.h"
#include "TargetInfo/RISCVTargetInfo.h"
#include "llvm/MC/TargetRegistry.h"
#include "llvm/Support/Compiler.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/DebugLog.h"
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 23-32: Header guard and interface framing / 头文件保护与接口框架
```cpp
#define DEBUG_TYPE "llvm-mca-riscv-custombehaviour"

namespace llvm::RISCV {
struct VXMemOpInfo {
  unsigned Log2IdxEEW : 3;
  unsigned IsOrdered : 1;
  unsigned IsStore : 1;
  unsigned NFields : 4;
  unsigned BaseInstr;
};
```
**EN:** This block establishes include guards and the outer structure of the public interface.

**CN:** 该区段建立头文件保护并给出公共接口的外层结构。

### Lines 33-42: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp

#define GET_RISCVBaseVXMemOpTable_IMPL
#include "RISCVGenSearchableTables.inc"
} // namespace llvm::RISCV

namespace llvm {
namespace mca {

const llvm::StringRef RISCVLMULInstrument::DESC_NAME = "RISCV-LMUL";
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 43-62: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
bool RISCVLMULInstrument::isDataValid(llvm::StringRef Data) {
  // Return true if not one of the valid LMUL strings
  return StringSwitch<bool>(Data)
      .Cases({"M1", "M2", "M4", "M8", "MF2", "MF4", "MF8"}, true)
      .Default(false);
}

uint8_t RISCVLMULInstrument::getLMUL() const {
  // assertion prevents us from needing llvm_unreachable in the StringSwitch
  // below
  assert(isDataValid(getData()) &&
         "Cannot get LMUL because invalid Data value");
  // These are the LMUL values that are used in RISC-V tablegen
  return StringSwitch<uint8_t>(getData())
      .Case("M1", 0b000)
      .Case("M2", 0b001)
      .Case("M4", 0b010)
      .Case("M8", 0b011)
      .Case("MF2", 0b111)
      .Case("MF4", 0b110)
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 63-73: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
      .Case("MF8", 0b101);
}

const llvm::StringRef RISCVSEWInstrument::DESC_NAME = "RISCV-SEW";

bool RISCVSEWInstrument::isDataValid(llvm::StringRef Data) {
  // Return true if not one of the valid SEW strings
  return StringSwitch<bool>(Data)
      .Cases({"E8", "E16", "E32", "E64"}, true)
      .Default(false);
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 74-85: Function implementation: RISCVSEWInstrument::getSEW / 函数实现：RISCVSEWInstrument::getSEW
```cpp

uint8_t RISCVSEWInstrument::getSEW() const {
  // assertion prevents us from needing llvm_unreachable in the StringSwitch
  // below
  assert(isDataValid(getData()) && "Cannot get SEW because invalid Data value");
  // These are the LMUL values that are used in RISC-V tablegen
  return StringSwitch<uint8_t>(getData())
      .Case("E8", 8)
      .Case("E16", 16)
      .Case("E32", 32)
      .Case("E64", 64);
}
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 86-102: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

bool RISCVInstrumentManager::supportsInstrumentType(
    llvm::StringRef Type) const {
  return Type == RISCVLMULInstrument::DESC_NAME ||
         Type == RISCVSEWInstrument::DESC_NAME ||
         InstrumentManager::supportsInstrumentType(Type);
}

UniqueInstrument
RISCVInstrumentManager::createInstrument(llvm::StringRef Desc,
                                         llvm::StringRef Data) {
  if (Desc == RISCVLMULInstrument::DESC_NAME) {
    if (!RISCVLMULInstrument::isDataValid(Data)) {
      LDBG() << "RVCB: Bad data for instrument kind " << Desc << ": " << Data
             << '\n';
      return nullptr;
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 103-113: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    return std::make_unique<RISCVLMULInstrument>(Data);
  }

  if (Desc == RISCVSEWInstrument::DESC_NAME) {
    if (!RISCVSEWInstrument::isDataValid(Data)) {
      LDBG() << "RVCB: Bad data for instrument kind " << Desc << ": " << Data
             << '\n';
      return nullptr;
    }
    return std::make_unique<RISCVSEWInstrument>(Data);
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 114-127: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  LDBG() << "RVCB: Creating default instrument for Desc: " << Desc << '\n';
  return InstrumentManager::createInstrument(Desc, Data);
}

SmallVector<UniqueInstrument>
RISCVInstrumentManager::createInstruments(const MCInst &Inst) {
  if (Inst.getOpcode() == RISCV::VSETVLI ||
      Inst.getOpcode() == RISCV::VSETIVLI) {
    LDBG() << "RVCB: Found VSETVLI and creating instrument for it: " << Inst
           << "\n";
    unsigned VTypeI = Inst.getOperand(2).getImm();
    RISCVVType::VLMUL VLMUL = RISCVVType::getVLMUL(VTypeI);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 128-147: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    StringRef LMUL;
    switch (VLMUL) {
    case RISCVVType::LMUL_1:
      LMUL = "M1";
      break;
    case RISCVVType::LMUL_2:
      LMUL = "M2";
      break;
    case RISCVVType::LMUL_4:
      LMUL = "M4";
      break;
    case RISCVVType::LMUL_8:
      LMUL = "M8";
      break;
    case RISCVVType::LMUL_F2:
      LMUL = "MF2";
      break;
    case RISCVVType::LMUL_F4:
      LMUL = "MF4";
      break;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 148-157: Definitions and supporting logic / 定义与支撑逻辑
```cpp
    case RISCVVType::LMUL_F8:
      LMUL = "MF8";
      break;
    case RISCVVType::LMUL_RESERVED:
      llvm_unreachable("Cannot create instrument for LMUL_RESERVED");
    }
    SmallVector<UniqueInstrument> Instruments;
    Instruments.emplace_back(
        createInstrument(RISCVLMULInstrument::DESC_NAME, LMUL));
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 158-175: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    unsigned SEW = RISCVVType::getSEW(VTypeI);
    StringRef SEWStr;
    switch (SEW) {
    case 8:
      SEWStr = "E8";
      break;
    case 16:
      SEWStr = "E16";
      break;
    case 32:
      SEWStr = "E32";
      break;
    case 64:
      SEWStr = "E64";
      break;
    default:
      llvm_unreachable("Cannot create instrument for SEW");
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 176-195: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    Instruments.emplace_back(
        createInstrument(RISCVSEWInstrument::DESC_NAME, SEWStr));

    return Instruments;
  }
  return SmallVector<UniqueInstrument>();
}

static std::pair<uint8_t, uint8_t>
getEEWAndEMUL(unsigned Opcode, RISCVVType::VLMUL LMUL, uint8_t SEW) {
  uint8_t EEW;
  switch (Opcode) {
  case RISCV::VLM_V:
  case RISCV::VSM_V:
  case RISCV::VLE8_V:
  case RISCV::VSE8_V:
  case RISCV::VLSE8_V:
  case RISCV::VSSE8_V:
    EEW = 8;
    break;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 196-215: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  case RISCV::VLE16_V:
  case RISCV::VSE16_V:
  case RISCV::VLSE16_V:
  case RISCV::VSSE16_V:
    EEW = 16;
    break;
  case RISCV::VLE32_V:
  case RISCV::VSE32_V:
  case RISCV::VLSE32_V:
  case RISCV::VSSE32_V:
    EEW = 32;
    break;
  case RISCV::VLE64_V:
  case RISCV::VSE64_V:
  case RISCV::VLSE64_V:
  case RISCV::VSSE64_V:
    EEW = 64;
    break;
  default:
    llvm_unreachable("Could not determine EEW from Opcode");
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 216-235: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  }

  auto EMUL =
      RISCVVType::getSameRatioLMUL(RISCVVType::getSEWLMULRatio(SEW, LMUL), EEW);
  if (!EEW)
    llvm_unreachable("Invalid SEW or LMUL for new ratio");
  return std::make_pair(EEW, *EMUL);
}

static bool opcodeHasEEWAndEMULInfo(unsigned short Opcode) {
  return Opcode == RISCV::VLM_V || Opcode == RISCV::VSM_V ||
         Opcode == RISCV::VLE8_V || Opcode == RISCV::VSE8_V ||
         Opcode == RISCV::VLE16_V || Opcode == RISCV::VSE16_V ||
         Opcode == RISCV::VLE32_V || Opcode == RISCV::VSE32_V ||
         Opcode == RISCV::VLE64_V || Opcode == RISCV::VSE64_V ||
         Opcode == RISCV::VLSE8_V || Opcode == RISCV::VSSE8_V ||
         Opcode == RISCV::VLSE16_V || Opcode == RISCV::VSSE16_V ||
         Opcode == RISCV::VLSE32_V || Opcode == RISCV::VSSE32_V ||
         Opcode == RISCV::VLSE64_V || Opcode == RISCV::VSSE64_V;
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 236-251: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

unsigned RISCVInstrumentManager::getSchedClassID(
    const MCInstrInfo &MCII, const MCInst &MCI,
    const llvm::SmallVector<Instrument *> &IVec) const {
  unsigned short Opcode = MCI.getOpcode();
  unsigned SchedClassID = MCII.get(Opcode).getSchedClass();

  // Unpack all possible RISC-V instruments from IVec.
  RISCVLMULInstrument *LI = nullptr;
  RISCVSEWInstrument *SI = nullptr;
  for (auto &I : IVec) {
    if (I->getDesc() == RISCVLMULInstrument::DESC_NAME)
      LI = static_cast<RISCVLMULInstrument *>(I);
    else if (I->getDesc() == RISCVSEWInstrument::DESC_NAME)
      SI = static_cast<RISCVSEWInstrument *>(I);
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 252-265: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  // Need LMUL or LMUL, SEW in order to override opcode. If no LMUL is provided,
  // then no option to override.
  if (!LI) {
    LDBG() << "RVCB: Did not use instrumentation to override Opcode.\n";
    return SchedClassID;
  }
  uint8_t LMUL = LI->getLMUL();

  // getBaseInfo works with (Opcode, LMUL, 0) if no SEW instrument,
  // or (Opcode, LMUL, SEW) if SEW instrument is active, and depends on LMUL
  // and SEW, or (Opcode, LMUL, 0) if does not depend on SEW.
  uint8_t SEW = SI ? SI->getSEW() : 0;
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 266-284: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  std::optional<unsigned> VPOpcode;
  if (const auto *VXMO = RISCV::getVXMemOpInfo(Opcode)) {
    // Calculate the expected index EMUL. For indexed operations,
    // the DataEEW and DataEMUL are equal to SEW and LMUL, respectively.
    unsigned IndexEMUL = ((1 << VXMO->Log2IdxEEW) * LMUL) / SEW;

    if (!VXMO->NFields) {
      // Indexed Load / Store.
      if (VXMO->IsStore) {
        if (const auto *VXP = RISCV::getVSXPseudo(
                /*Masked=*/0, VXMO->IsOrdered, VXMO->Log2IdxEEW, LMUL,
                IndexEMUL))
          VPOpcode = VXP->Pseudo;
      } else {
        if (const auto *VXP = RISCV::getVLXPseudo(
                /*Masked=*/0, VXMO->IsOrdered, VXMO->Log2IdxEEW, LMUL,
                IndexEMUL))
          VPOpcode = VXP->Pseudo;
      }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 285-297: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    } else {
      // Segmented Indexed Load / Store.
      if (VXMO->IsStore) {
        if (const auto *VXP = RISCV::getVSXSEGPseudo(
                VXMO->NFields, /*Masked=*/0, VXMO->IsOrdered, VXMO->Log2IdxEEW,
                LMUL, IndexEMUL))
          VPOpcode = VXP->Pseudo;
      } else {
        if (const auto *VXP = RISCV::getVLXSEGPseudo(
                VXMO->NFields, /*Masked=*/0, VXMO->IsOrdered, VXMO->Log2IdxEEW,
                LMUL, IndexEMUL))
          VPOpcode = VXP->Pseudo;
      }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 298-311: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    }
  } else if (opcodeHasEEWAndEMULInfo(Opcode)) {
    RISCVVType::VLMUL VLMUL = static_cast<RISCVVType::VLMUL>(LMUL);
    auto [EEW, EMUL] = getEEWAndEMUL(Opcode, VLMUL, SEW);
    if (const auto *RVV =
            RISCVVInversePseudosTable::getBaseInfo(Opcode, EMUL, EEW))
      VPOpcode = RVV->Pseudo;
  } else {
    // Check if it depends on LMUL and SEW
    const auto *RVV = RISCVVInversePseudosTable::getBaseInfo(Opcode, LMUL, SEW);
    // Check if it depends only on LMUL
    if (!RVV)
      RVV = RISCVVInversePseudosTable::getBaseInfo(Opcode, LMUL, 0);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 312-325: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    if (RVV)
      VPOpcode = RVV->Pseudo;
  }

  // Not a RVV instr
  if (!VPOpcode) {
    LDBG() << "RVCB: Could not find PseudoInstruction for Opcode "
           << MCII.getName(Opcode)
           << ", LMUL=" << (LI ? LI->getData() : "Unspecified")
           << ", SEW=" << (SI ? SI->getData() : "Unspecified")
           << ". Ignoring instrumentation and using original SchedClassID="
           << SchedClassID << '\n';
    return SchedClassID;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 326-335: Definitions and supporting logic / 定义与支撑逻辑
```cpp

  // Override using pseudo
  LDBG() << "RVCB: Found Pseudo Instruction for Opcode " << MCII.getName(Opcode)
         << ", LMUL=" << LI->getData()
         << ", SEW=" << (SI ? SI->getData() : "Unspecified")
         << ". Overriding original SchedClassID=" << SchedClassID << " with "
         << MCII.getName(*VPOpcode) << '\n';
  return MCII.get(*VPOpcode).getSchedClass();
}
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 336-346: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
} // namespace mca
} // namespace llvm

using namespace llvm;
using namespace mca;

static InstrumentManager *
createRISCVInstrumentManager(const MCSubtargetInfo &STI,
                             const MCInstrInfo &MCII) {
  return new RISCVInstrumentManager(STI, MCII);
}
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 347-355: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

/// Extern function to initialize the targets for the RISC-V backend
extern "C" LLVM_ABI LLVM_EXTERNAL_VISIBILITY void
LLVMInitializeRISCVTargetMCA() {
  TargetRegistry::RegisterInstrumentManager(getTheRISCV32Target(),
                                            createRISCVInstrumentManager);
  TargetRegistry::RegisterInstrumentManager(getTheRISCV64Target(),
                                            createRISCVInstrumentManager);
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

## Key Concepts / 关键概念
- **Machine code analysis** / **机器码分析**

## Dependencies / 依赖关系
- `RISCVCustomBehaviour.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `MCTargetDesc/RISCVMCTargetDesc.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCV.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `TargetInfo/RISCVTargetInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/TargetRegistry.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/Support/Compiler.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/Support/Debug.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/Support/DebugLog.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVGenSearchableTables.inc` — Directly referenced by this file. / 该文件直接引用的依赖。
