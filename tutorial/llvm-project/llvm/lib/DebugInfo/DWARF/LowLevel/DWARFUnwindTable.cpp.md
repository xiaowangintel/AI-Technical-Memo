# DWARFUnwindTable.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/DebugInfo/DWARF/LowLevel/DWARFUnwindTable.cpp`
- Repository: `llvm-project`
- Purpose (EN): All the unwinding rows parsed during processing of the CFI program.
- Purpose (CN): 该文件位于 LLVM 的 `DebugInfo/DWARF/LowLevel` 目录中，主要实现与 `DWARFUnwindTable` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===----------------------------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/DebugInfo/DWARF/LowLevel/DWARFUnwindTable.h"
#include "llvm/Support/Errc.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/raw_ostream.h"
#include <cassert>
#include <cinttypes>
#include <cstdint>
#include <optional>

using namespace llvm;
using namespace dwarf;

UnwindLocation UnwindLocation::createUnspecified() { return {Unspecified}; }

UnwindLocation UnwindLocation::createUndefined() { return {Undefined}; }

UnwindLocation UnwindLocation::createSame() { return {Same}; }

UnwindLocation UnwindLocation::createIsConstant(int32_t Value) {
  return {Constant, InvalidRegisterNumber, Value, std::nullopt, false};
}

UnwindLocation UnwindLocation::createIsCFAPlusOffset(int32_t Offset) {
  return {CFAPlusOffset, InvalidRegisterNumber, Offset, std::nullopt, false};
}

UnwindLocation UnwindLocation::createAtCFAPlusOffset(int32_t Offset) {
  return {CFAPlusOffset, InvalidRegisterNumber, Offset, std::nullopt, true};
}

UnwindLocation
UnwindLocation::createIsRegisterPlusOffset(uint32_t RegNum, int32_t Offset,
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 8 direct dependencies, including `llvm/DebugInfo/DWARF/LowLevel/DWARFUnwindTable.h`, `llvm/Support/Errc.h`, `llvm/Support/ErrorHandling.h`, `llvm/Support/raw_ostream.h`.
  CN: 引入了 8 个直接依赖，其中包括 `llvm/DebugInfo/DWARF/LowLevel/DWARFUnwindTable.h`, `llvm/Support/Errc.h`, `llvm/Support/ErrorHandling.h`, `llvm/Support/raw_ostream.h`。
- EN: This section centers on `createUnspecified`, `createUndefined`, `createSame` and creates and initializes supporting objects or state.
  CN: 这一段主要围绕 `createUnspecified`, `createUndefined`, `createSame` 等符号展开，负责创建并初始化辅助对象或状态。

### Lines 41-80

```cpp
                                           std::optional<uint32_t> AddrSpace) {
  return {RegPlusOffset, RegNum, Offset, AddrSpace, false};
}

UnwindLocation
UnwindLocation::createAtRegisterPlusOffset(uint32_t RegNum, int32_t Offset,
                                           std::optional<uint32_t> AddrSpace) {
  return {RegPlusOffset, RegNum, Offset, AddrSpace, true};
}

UnwindLocation UnwindLocation::createIsDWARFExpression(DWARFExpression Expr) {
  return {Expr, false};
}

UnwindLocation UnwindLocation::createAtDWARFExpression(DWARFExpression Expr) {
  return {Expr, true};
}

bool UnwindLocation::operator==(const UnwindLocation &RHS) const {
  if (Kind != RHS.Kind)
    return false;
  switch (Kind) {
  case Unspecified:
  case Undefined:
  case Same:
    return true;
  case CFAPlusOffset:
    return Offset == RHS.Offset && Dereference == RHS.Dereference;
  case RegPlusOffset:
    return RegNum == RHS.RegNum && Offset == RHS.Offset &&
           Dereference == RHS.Dereference;
  case DWARFExpr:
    return *Expr == *RHS.Expr && Dereference == RHS.Dereference;
  case Constant:
    return Offset == RHS.Offset;
  }
  return false;
}

Expected<UnwindTable::RowContainer>
```
- EN: This section centers on `createAtRegisterPlusOffset`, `createIsDWARFExpression`, `createAtDWARFExpression` and creates and initializes supporting objects or state.
  CN: 这一段主要围绕 `createAtRegisterPlusOffset`, `createIsDWARFExpression`, `createAtDWARFExpression` 等符号展开，负责创建并初始化辅助对象或状态。
- EN: In this range, the code uses a `switch` to dispatch behavior by kind or encoding and checks conditions and handles edge cases.
  CN: 在这一段中，代码使用 `switch` 按类型或编码分派行为，并检查条件并处理边界情况。

### Lines 81-120

```cpp
llvm::dwarf::parseRows(const CFIProgram &CFIP, UnwindRow &Row,
                       const RegisterLocations *InitialLocs) {
  // All the unwinding rows parsed during processing of the CFI program.
  UnwindTable::RowContainer Rows;

  // State consists of CFA value and register locations.
  std::vector<std::pair<UnwindLocation, RegisterLocations>> States;
  for (const CFIProgram::Instruction &Inst : CFIP) {
    switch (Inst.Opcode) {
    case dwarf::DW_CFA_set_loc: {
      // The DW_CFA_set_loc instruction takes a single operand that
      // represents a target address. The required action is to create a new
      // table row using the specified address as the location. All other
      // values in the new row are initially identical to the current row.
      // The new location value is always greater than the current one. If
      // the segment_size field of this FDE's CIE is non- zero, the initial
      // location is preceded by a segment selector of the given length
      llvm::Expected<uint64_t> NewAddress = Inst.getOperandAsUnsigned(CFIP, 0);
      if (!NewAddress)
        return NewAddress.takeError();
      if (*NewAddress <= Row.getAddress())
        return createStringError(
            errc::invalid_argument,
            "%s with adrress 0x%" PRIx64 " which must be greater than the "
            "current row address 0x%" PRIx64,
            CFIP.callFrameString(Inst.Opcode).str().c_str(), *NewAddress,
            Row.getAddress());
      Rows.push_back(Row);
      Row.setAddress(*NewAddress);
      break;
    }

    case dwarf::DW_CFA_advance_loc:
    case dwarf::DW_CFA_advance_loc1:
    case dwarf::DW_CFA_advance_loc2:
    case dwarf::DW_CFA_advance_loc4: {
      // The DW_CFA_advance instruction takes a single operand that
      // represents a constant delta. The required action is to create a new
      // table row with a location value that is computed by taking the
      // current entry’s location value and adding the value of delta *
```
- EN: This section centers on `parseRows` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `parseRows` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code uses a `switch` to dispatch behavior by kind or encoding and iterates over collections, ranges, or records.
  CN: 在这一段中，代码使用 `switch` 按类型或编码分派行为，并遍历集合、区间或记录。

### Lines 121-160

```cpp
      // code_alignment_factor. All other values in the new row are initially
      // identical to the current row.
      Rows.push_back(Row);
      llvm::Expected<uint64_t> Offset = Inst.getOperandAsUnsigned(CFIP, 0);
      if (!Offset)
        return Offset.takeError();
      Row.slideAddress(*Offset);
      break;
    }

    case dwarf::DW_CFA_restore:
    case dwarf::DW_CFA_restore_extended: {
      // The DW_CFA_restore instruction takes a single operand (encoded with
      // the opcode) that represents a register number. The required action
      // is to change the rule for the indicated register to the rule
      // assigned it by the initial_instructions in the CIE.
      if (InitialLocs == nullptr)
        return createStringError(
            errc::invalid_argument, "%s encountered while parsing a CIE",
            CFIP.callFrameString(Inst.Opcode).str().c_str());
      llvm::Expected<uint64_t> RegNum = Inst.getOperandAsUnsigned(CFIP, 0);
      if (!RegNum)
        return RegNum.takeError();
      if (std::optional<UnwindLocation> O =
              InitialLocs->getRegisterLocation(*RegNum))
        Row.getRegisterLocations().setRegisterLocation(*RegNum, *O);
      else
        Row.getRegisterLocations().removeRegisterLocation(*RegNum);
      break;
    }

    case dwarf::DW_CFA_offset:
    case dwarf::DW_CFA_offset_extended:
    case dwarf::DW_CFA_offset_extended_sf: {
      llvm::Expected<uint64_t> RegNum = Inst.getOperandAsUnsigned(CFIP, 0);
      if (!RegNum)
        return RegNum.takeError();
      llvm::Expected<int64_t> Offset = Inst.getOperandAsSigned(CFIP, 1);
      if (!Offset)
        return Offset.takeError();
```
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 161-200

```cpp
      Row.getRegisterLocations().setRegisterLocation(
          *RegNum, UnwindLocation::createAtCFAPlusOffset(*Offset));
      break;
    }

    case dwarf::DW_CFA_nop:
      break;

    case dwarf::DW_CFA_remember_state:
      States.push_back(
          std::make_pair(Row.getCFAValue(), Row.getRegisterLocations()));
      break;

    case dwarf::DW_CFA_restore_state:
      if (States.empty())
        return createStringError(errc::invalid_argument,
                                 "DW_CFA_restore_state without a matching "
                                 "previous DW_CFA_remember_state");
      Row.getCFAValue() = States.back().first;
      Row.getRegisterLocations() = States.back().second;
      States.pop_back();
      break;

    case dwarf::DW_CFA_GNU_window_save:
      switch (CFIP.triple()) {
      case Triple::aarch64:
      case Triple::aarch64_be:
      case Triple::aarch64_32: {
        // DW_CFA_GNU_window_save is used for different things on different
        // architectures. For aarch64 it is known as
        // DW_CFA_AARCH64_negate_ra_state. The action is to toggle the
        // value of the return address state between 1 and 0. If there is
        // no rule for the AARCH64_DWARF_PAUTH_RA_STATE register, then it
        // should be initially set to 1.
        constexpr uint32_t AArch64DWARFPAuthRaState = 34;
        auto LRLoc = Row.getRegisterLocations().getRegisterLocation(
            AArch64DWARFPAuthRaState);
        if (LRLoc) {
          if (LRLoc->getLocation() == UnwindLocation::Constant) {
            // Toggle the constant value from 0 to 1 or 1 to 0.
```
- EN: This section centers on `make_pair` and creates and initializes supporting objects or state.
  CN: 这一段主要围绕 `make_pair` 等符号展开，负责创建并初始化辅助对象或状态。
- EN: In this range, the code uses a `switch` to dispatch behavior by kind or encoding and checks conditions and handles edge cases.
  CN: 在这一段中，代码使用 `switch` 按类型或编码分派行为，并检查条件并处理边界情况。

### Lines 201-240

```cpp
            LRLoc->setConstant(LRLoc->getConstant() ^ 1);
            Row.getRegisterLocations().setRegisterLocation(
                AArch64DWARFPAuthRaState, *LRLoc);
          } else {
            return createStringError(
                errc::invalid_argument,
                "%s encountered when existing rule for this register is not "
                "a constant",
                CFIP.callFrameString(Inst.Opcode).str().c_str());
          }
        } else {
          Row.getRegisterLocations().setRegisterLocation(
              AArch64DWARFPAuthRaState, UnwindLocation::createIsConstant(1));
        }
        break;
      }

      case Triple::sparc:
      case Triple::sparcv9:
      case Triple::sparcel:
        for (uint32_t RegNum = 16; RegNum < 32; ++RegNum) {
          Row.getRegisterLocations().setRegisterLocation(
              RegNum, UnwindLocation::createAtCFAPlusOffset((RegNum - 16) * 8));
        }
        break;

      default: {
        return createStringError(
            errc::not_supported,
            "DW_CFA opcode %#x is not supported for architecture %s",
            Inst.Opcode, Triple::getArchTypeName(CFIP.triple()).str().c_str());

        break;
      }
      }
      break;

    case dwarf::DW_CFA_AARCH64_negate_ra_state_with_pc: {
      constexpr uint32_t AArch64DWARFPAuthRaState = 34;
      auto LRLoc = Row.getRegisterLocations().getRegisterLocation(
```
- EN: This section centers on `createStringError`, `createIsConstant`, `createAtCFAPlusOffset` and creates and initializes supporting objects or state.
  CN: 这一段主要围绕 `createStringError`, `createIsConstant`, `createAtCFAPlusOffset` 等符号展开，负责创建并初始化辅助对象或状态。
- EN: In this range, the code iterates over collections, ranges, or records and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码遍历集合、区间或记录，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 241-280

```cpp
          AArch64DWARFPAuthRaState);
      if (LRLoc) {
        if (LRLoc->getLocation() == UnwindLocation::Constant) {
          // Toggle the constant value of bits[1:0] from 0 to 1 or 1 to 0.
          LRLoc->setConstant(LRLoc->getConstant() ^ 0x3);
        } else {
          return createStringError(
              errc::invalid_argument,
              "%s encountered when existing rule for this register is not "
              "a constant",
              CFIP.callFrameString(Inst.Opcode).str().c_str());
        }
      } else {
        Row.getRegisterLocations().setRegisterLocation(
            AArch64DWARFPAuthRaState, UnwindLocation::createIsConstant(0x3));
      }
      break;
    }

    case dwarf::DW_CFA_undefined: {
      llvm::Expected<uint64_t> RegNum = Inst.getOperandAsUnsigned(CFIP, 0);
      if (!RegNum)
        return RegNum.takeError();
      Row.getRegisterLocations().setRegisterLocation(
          *RegNum, UnwindLocation::createUndefined());
      break;
    }

    case dwarf::DW_CFA_same_value: {
      llvm::Expected<uint64_t> RegNum = Inst.getOperandAsUnsigned(CFIP, 0);
      if (!RegNum)
        return RegNum.takeError();
      Row.getRegisterLocations().setRegisterLocation(
          *RegNum, UnwindLocation::createSame());
      break;
    }

    case dwarf::DW_CFA_GNU_args_size:
      break;

```
- EN: This section centers on `createStringError`, `createIsConstant` and creates and initializes supporting objects or state.
  CN: 这一段主要围绕 `createStringError`, `createIsConstant` 等符号展开，负责创建并初始化辅助对象或状态。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 281-320

```cpp
    case dwarf::DW_CFA_register: {
      llvm::Expected<uint64_t> RegNum = Inst.getOperandAsUnsigned(CFIP, 0);
      if (!RegNum)
        return RegNum.takeError();
      llvm::Expected<uint64_t> NewRegNum = Inst.getOperandAsUnsigned(CFIP, 1);
      if (!NewRegNum)
        return NewRegNum.takeError();
      Row.getRegisterLocations().setRegisterLocation(
          *RegNum, UnwindLocation::createIsRegisterPlusOffset(*NewRegNum, 0));
      break;
    }

    case dwarf::DW_CFA_val_offset:
    case dwarf::DW_CFA_val_offset_sf: {
      llvm::Expected<uint64_t> RegNum = Inst.getOperandAsUnsigned(CFIP, 0);
      if (!RegNum)
        return RegNum.takeError();
      llvm::Expected<int64_t> Offset = Inst.getOperandAsSigned(CFIP, 1);
      if (!Offset)
        return Offset.takeError();
      Row.getRegisterLocations().setRegisterLocation(
          *RegNum, UnwindLocation::createIsCFAPlusOffset(*Offset));
      break;
    }

    case dwarf::DW_CFA_expression: {
      llvm::Expected<uint64_t> RegNum = Inst.getOperandAsUnsigned(CFIP, 0);
      if (!RegNum)
        return RegNum.takeError();
      Row.getRegisterLocations().setRegisterLocation(
          *RegNum, UnwindLocation::createAtDWARFExpression(*Inst.Expression));
      break;
    }

    case dwarf::DW_CFA_val_expression: {
      llvm::Expected<uint64_t> RegNum = Inst.getOperandAsUnsigned(CFIP, 0);
      if (!RegNum)
        return RegNum.takeError();
      Row.getRegisterLocations().setRegisterLocation(
          *RegNum, UnwindLocation::createIsDWARFExpression(*Inst.Expression));
```
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 321-360

```cpp
      break;
    }

    case dwarf::DW_CFA_def_cfa_register: {
      llvm::Expected<uint64_t> RegNum = Inst.getOperandAsUnsigned(CFIP, 0);
      if (!RegNum)
        return RegNum.takeError();
      if (Row.getCFAValue().getLocation() != UnwindLocation::RegPlusOffset)
        Row.getCFAValue() =
            UnwindLocation::createIsRegisterPlusOffset(*RegNum, 0);
      else
        Row.getCFAValue().setRegister(*RegNum);
      break;
    }

    case dwarf::DW_CFA_def_cfa_offset:
    case dwarf::DW_CFA_def_cfa_offset_sf: {
      llvm::Expected<int64_t> Offset = Inst.getOperandAsSigned(CFIP, 0);
      if (!Offset)
        return Offset.takeError();
      if (Row.getCFAValue().getLocation() != UnwindLocation::RegPlusOffset) {
        return createStringError(
            errc::invalid_argument,
            "%s found when CFA rule was not RegPlusOffset",
            CFIP.callFrameString(Inst.Opcode).str().c_str());
      }
      Row.getCFAValue().setOffset(*Offset);
      break;
    }

    case dwarf::DW_CFA_def_cfa:
    case dwarf::DW_CFA_def_cfa_sf: {
      llvm::Expected<uint64_t> RegNum = Inst.getOperandAsUnsigned(CFIP, 0);
      if (!RegNum)
        return RegNum.takeError();
      llvm::Expected<int64_t> Offset = Inst.getOperandAsSigned(CFIP, 1);
      if (!Offset)
        return Offset.takeError();
      Row.getCFAValue() =
          UnwindLocation::createIsRegisterPlusOffset(*RegNum, *Offset);
```
- EN: This section centers on `createStringError`, `createIsRegisterPlusOffset` and creates and initializes supporting objects or state.
  CN: 这一段主要围绕 `createStringError`, `createIsRegisterPlusOffset` 等符号展开，负责创建并初始化辅助对象或状态。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 361-388

```cpp
      break;
    }

    case dwarf::DW_CFA_LLVM_def_aspace_cfa:
    case dwarf::DW_CFA_LLVM_def_aspace_cfa_sf: {
      llvm::Expected<uint64_t> RegNum = Inst.getOperandAsUnsigned(CFIP, 0);
      if (!RegNum)
        return RegNum.takeError();
      llvm::Expected<int64_t> Offset = Inst.getOperandAsSigned(CFIP, 1);
      if (!Offset)
        return Offset.takeError();
      llvm::Expected<uint32_t> CFAAddrSpace =
          Inst.getOperandAsUnsigned(CFIP, 2);
      if (!CFAAddrSpace)
        return CFAAddrSpace.takeError();
      Row.getCFAValue() = UnwindLocation::createIsRegisterPlusOffset(
          *RegNum, *Offset, *CFAAddrSpace);
      break;
    }

    case dwarf::DW_CFA_def_cfa_expression:
      Row.getCFAValue() =
          UnwindLocation::createIsDWARFExpression(*Inst.Expression);
      break;
    }
  }
  return Rows;
}
```
- EN: This section centers on `createIsDWARFExpression` and creates and initializes supporting objects or state.
  CN: 这一段主要围绕 `createIsDWARFExpression` 等符号展开，负责创建并初始化辅助对象或状态。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

## Key Concepts / 关键概念
- Domain / 领域: DWARF debug information / DWARF 调试信息
- Core symbols / 核心符号: `createUnspecified`, `createUndefined`, `createSame`, `createIsConstant` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Helper routines coordinate local state, control flow, and result construction. / 通过辅助例程协调局部状态、控制流与结果构造。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。
- Data structures / 数据结构: Relies on LLVM or STL containers for compact state management. / 依赖 LLVM 或 STL 容器管理紧凑状态。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/DebugInfo/DWARF/LowLevel/DWARFUnwindTable.h`, `llvm/Support/Errc.h`, `llvm/Support/ErrorHandling.h`, `llvm/Support/raw_ostream.h`
- Standard library / 标准库: `cstdint`, `optional`
- Other/system headers / 其他或系统头文件: `cassert`, `cinttypes`
- Related symbols / 相关符号: `createUnspecified`, `createUndefined`, `createSame`, `createIsConstant`, `createIsCFAPlusOffset`
