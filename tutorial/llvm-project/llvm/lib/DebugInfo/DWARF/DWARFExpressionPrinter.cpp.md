# DWARFExpressionPrinter.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/DebugInfo/DWARF/DWARFExpressionPrinter.cpp`
- Repository: `llvm-project`
- Purpose (EN): In "register-only" mode, still show simple constant-valued locations.
- Purpose (CN): 该文件位于 LLVM 的 `DebugInfo/DWARF` 目录中，主要实现与 `DWARFExpressionPrinter` 相关的接口、数据结构和辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-40

```cpp
//===-- DWARFExpression.cpp -----------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "llvm/DebugInfo/DWARF/DWARFExpressionPrinter.h"
#include "llvm/ADT/SmallString.h"
#include "llvm/DebugInfo/DWARF/DWARFUnit.h"
#include "llvm/DebugInfo/DWARF/LowLevel/DWARFExpression.h"
#include "llvm/Support/Format.h"
#include "llvm/Support/FormatVariadic.h"
#include <cassert>
#include <cstdint>

using namespace llvm;
using namespace dwarf;

namespace llvm {

typedef DWARFExpression::Operation Op;
typedef Op::Description Desc;

static void prettyPrintBaseTypeRef(DWARFUnit *U, raw_ostream &OS,
                                   DIDumpOptions DumpOpts,
                                   ArrayRef<uint64_t> Operands,
                                   unsigned Operand) {
  assert(Operand < Operands.size() && "operand out of bounds");
  if (!U) {
    OS << formatv(" <base_type ref: {0:x}>", Operands[Operand]);
    return;
  }
  auto Die = U->getDIEForOffset(U->getOffset() + Operands[Operand]);
  if (Die && Die.getTag() == dwarf::DW_TAG_base_type) {
    OS << " (";
    if (DumpOpts.Verbose)
      OS << formatv("{0:x8} -> ", Operands[Operand]);
    OS << formatv("{0:x8})", U->getOffset() + Operands[Operand]);
```
- EN: Introduces the file banner, comments, and the initial setup for this translation unit.
  CN: 这一段给出文件横幅、注释说明以及该编译单元的初始设置。
- EN: Brings in 8 direct dependencies, including `llvm/DebugInfo/DWARF/DWARFExpressionPrinter.h`, `llvm/ADT/SmallString.h`, `llvm/DebugInfo/DWARF/DWARFUnit.h`, `llvm/DebugInfo/DWARF/LowLevel/DWARFExpression.h`.
  CN: 引入了 8 个直接依赖，其中包括 `llvm/DebugInfo/DWARF/DWARFExpressionPrinter.h`, `llvm/ADT/SmallString.h`, `llvm/DebugInfo/DWARF/DWARFUnit.h`, `llvm/DebugInfo/DWARF/LowLevel/DWARFExpression.h`。
- EN: This section centers on `prettyPrintBaseTypeRef`, `assert` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `prettyPrintBaseTypeRef`, `assert` 等符号展开，负责为下游使用方写出、格式化或序列化结果。

### Lines 41-80

```cpp
    if (auto Name = dwarf::toString(Die.find(dwarf::DW_AT_name)))
      OS << " \"" << *Name << "\"";
  } else {
    OS << formatv(" <invalid base_type ref: {0:x}>", Operands[Operand]);
  }
}

static bool printOp(const DWARFExpression::Operation *Op, raw_ostream &OS,
                    DIDumpOptions DumpOpts, const DWARFExpression *Expr,
                    DWARFUnit *U) {
  if (Op->isError()) {
    if (!DumpOpts.PrintRegisterOnly)
      OS << "<decoding error>";
    return false;
  }

  std::optional<unsigned> SubOpcode = Op->getSubCode();

  // In "register-only" mode, still show simple constant-valued locations.
  // This lets clients print annotations like "i = 0" when the location is
  // a constant (e.g. DW_OP_constu/consts ... DW_OP_stack_value).
  // We continue to suppress all other non-register ops in this mode.
  if (DumpOpts.PrintRegisterOnly) {
    // First, try pretty-printing registers (existing behavior below also does
    // this, but we need to short-circuit here to avoid printing opcode names).
    if ((Op->getCode() >= DW_OP_breg0 && Op->getCode() <= DW_OP_breg31) ||
        (Op->getCode() >= DW_OP_reg0 && Op->getCode() <= DW_OP_reg31) ||
        Op->getCode() == DW_OP_bregx || Op->getCode() == DW_OP_regx ||
        Op->getCode() == DW_OP_regval_type ||
        SubOpcode == DW_OP_LLVM_call_frame_entry_reg ||
        SubOpcode == DW_OP_LLVM_aspace_bregx) {
      if (prettyPrintRegisterOp(U, OS, DumpOpts, Op->getCode(),
                                Op->getRawOperands()))
        return true;
      // If we couldn't pretty-print, fall through and suppress.
    }

    // Show constants (decimal), suppress everything else.
    if (Op->getCode() == DW_OP_constu) {
      OS << (uint64_t)Op->getRawOperand(0);
```
- EN: This section centers on `printOp` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `printOp` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 81-120

```cpp
      return true;
    }
    if (Op->getCode() == DW_OP_consts) {
      OS << (int64_t)Op->getRawOperand(0);
      return true;
    }
    if (Op->getCode() >= DW_OP_lit0 && Op->getCode() <= DW_OP_lit31) {
      OS << (unsigned)(Op->getCode() - DW_OP_lit0);
      return true;
    }
    if (Op->getCode() == DW_OP_stack_value)
      return true; // metadata; don't print a token

    return true; // suppress other opcodes silently in register-only mode
  }

  if (!DumpOpts.PrintRegisterOnly) {
    StringRef Name = OperationEncodingString(Op->getCode());
    assert(!Name.empty() && "DW_OP has no name!");
    OS << Name;

    if (SubOpcode) {
      StringRef SubName = SubOperationEncodingString(Op->getCode(), *SubOpcode);
      assert(!SubName.empty() && "DW_OP SubOp has no name!");
      OS << ' ' << SubName;
    }
  }

  if ((Op->getCode() >= DW_OP_breg0 && Op->getCode() <= DW_OP_breg31) ||
      (Op->getCode() >= DW_OP_reg0 && Op->getCode() <= DW_OP_reg31) ||
      Op->getCode() == DW_OP_bregx || Op->getCode() == DW_OP_regx ||
      Op->getCode() == DW_OP_regval_type ||
      SubOpcode == DW_OP_LLVM_call_frame_entry_reg ||
      SubOpcode == DW_OP_LLVM_aspace_bregx)
    if (prettyPrintRegisterOp(U, OS, DumpOpts, Op->getCode(),
                              Op->getRawOperands()))
      return true;

  if (!DumpOpts.PrintRegisterOnly) {
    for (unsigned Operand = 0; Operand < Op->getDescription().Op.size();
```
- EN: This section centers on `assert` and parses input and converts raw data into structured form.
  CN: 这一段主要围绕 `assert` 等符号展开，负责解析输入并把原始数据转换成结构化形式。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 121-160

```cpp
         ++Operand) {
      unsigned Size = Op->getDescription().Op[Operand];
      unsigned Signed = Size & DWARFExpression::Operation::SignBit;

      if (Size == DWARFExpression::Operation::SizeSubOpLEB) {
        assert(Operand == 0 && "DW_OP SubOp must be the first operand");
        assert(SubOpcode && "DW_OP SubOp description is inconsistent");
      } else if (Size == DWARFExpression::Operation::BaseTypeRef && U) {
        // For DW_OP_convert the operand may be 0 to indicate that conversion to
        // the generic type should be done. The same holds for
        // DW_OP_reinterpret, which is currently not supported.
        if (Op->getCode() == DW_OP_convert && Op->getRawOperand(Operand) == 0)
          OS << " 0x0";
        else
          prettyPrintBaseTypeRef(U, OS, DumpOpts, Op->getRawOperands(),
                                 Operand);
      } else if (Size == DWARFExpression::Operation::WasmLocationArg) {
        assert(Operand == 1);
        switch (Op->getRawOperand(0)) {
        case 0:
        case 1:
        case 2:
        case 3: // global as uint32
        case 4:
          OS << formatv(" {0:x}", Op->getRawOperand(Operand));
          break;
        default:
          assert(false);
        }
      } else if (Size == DWARFExpression::Operation::SizeBlock) {
        uint64_t Offset = Op->getRawOperand(Operand);
        for (unsigned i = 0; i < Op->getRawOperand(Operand - 1); ++i)
          OS << formatv(" {0:x2}",
                        static_cast<uint8_t>(Expr->getData()[Offset++]));
      } else {
        if (Signed)
          OS << formatv(" {0:+d}", (int64_t)Op->getRawOperand(Operand));
        else if (Op->getCode() != DW_OP_entry_value &&
                 Op->getCode() != DW_OP_GNU_entry_value)
          OS << formatv(" {0:x}", Op->getRawOperand(Operand));
```
- EN: This section centers on `assert`, `prettyPrintBaseTypeRef` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `assert`, `prettyPrintBaseTypeRef` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: In this range, the code uses a `switch` to dispatch behavior by kind or encoding and iterates over collections, ranges, or records.
  CN: 在这一段中，代码使用 `switch` 按类型或编码分派行为，并遍历集合、区间或记录。

### Lines 161-200

```cpp
      }
    }
  }
  return true;
}

void printDwarfExpression(const DWARFExpression *E, raw_ostream &OS,
                          DIDumpOptions DumpOpts, DWARFUnit *U, bool IsEH) {
  uint32_t EntryValExprSize = 0;
  uint64_t EntryValStartOffset = 0;
  if (E->getData().empty())
    OS << "<empty>";

  for (auto &Op : *E) {
    DumpOpts.IsEH = IsEH;
    if (!printOp(&Op, OS, DumpOpts, E, U) && !DumpOpts.PrintRegisterOnly) {
      uint64_t FailOffset = Op.getEndOffset();
      while (FailOffset < E->getData().size())
        OS << formatv(" {0:x-2}",
                      static_cast<uint8_t>(E->getData()[FailOffset++]));
      return;
    }
    if (!DumpOpts.PrintRegisterOnly) {
      if (Op.getCode() == DW_OP_entry_value ||
          Op.getCode() == DW_OP_GNU_entry_value) {
        OS << "(";
        EntryValExprSize = Op.getRawOperand(0);
        EntryValStartOffset = Op.getEndOffset();
        continue;
      }

      if (EntryValExprSize) {
        EntryValExprSize -= Op.getEndOffset() - EntryValStartOffset;
        if (EntryValExprSize == 0)
          OS << ")";
      }

      if (Op.getEndOffset() < E->getData().size())
        OS << ", ";
    }
```
- EN: This section centers on `printDwarfExpression` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `printDwarfExpression` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 201-240

```cpp
  }
}

/// A user-facing string representation of a DWARF expression. This might be an
/// Address expression, in which case it will be implicitly dereferenced, or a
/// Value expression.
struct PrintedExpr {
  enum ExprKind {
    Address,
    Value,
  };
  ExprKind Kind;
  SmallString<16> String;

  PrintedExpr(ExprKind K = Address) : Kind(K) {}
};

static bool printCompactDWARFExpr(
    raw_ostream &OS, DWARFExpression::iterator I,
    const DWARFExpression::iterator E,
    std::function<StringRef(uint64_t RegNum, bool IsEH)> GetNameForDWARFReg =
        nullptr) {
  SmallVector<PrintedExpr, 4> Stack;

  auto UnknownOpcode = [](raw_ostream &OS, uint8_t Opcode,
                          std::optional<unsigned> SubOpcode) -> bool {
    // If we hit an unknown operand, we don't know its effect on the stack,
    // so bail out on the whole expression.
    OS << "<unknown op " << dwarf::OperationEncodingString(Opcode) << " ("
       << (int)Opcode;
    if (SubOpcode)
      OS << ") subop " << dwarf::SubOperationEncodingString(Opcode, *SubOpcode)
         << " (" << *SubOpcode;
    OS << ")>";
    return false;
  };

  while (I != E) {
    const DWARFExpression::Operation &Op = *I;
    uint8_t Opcode = Op.getCode();
```
- EN: This section centers on `PrintedExpr`, `printCompactDWARFExpr` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `PrintedExpr`, `printCompactDWARFExpr` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: In this range, the code iterates over collections, ranges, or records and checks conditions and handles edge cases.
  CN: 在这一段中，代码遍历集合、区间或记录，并检查条件并处理边界情况。

### Lines 241-280

```cpp
    switch (Opcode) {
    case dwarf::DW_OP_regx: {
      // DW_OP_regx: A register, with the register num given as an operand.
      // Printed as the plain register name.
      uint64_t DwarfRegNum = Op.getRawOperand(0);
      auto RegName = GetNameForDWARFReg(DwarfRegNum, false);
      if (RegName.empty())
        return false;
      raw_svector_ostream S(Stack.emplace_back(PrintedExpr::Value).String);
      S << RegName;
      break;
    }
    case dwarf::DW_OP_bregx: {
      int DwarfRegNum = Op.getRawOperand(0);
      int64_t Offset = Op.getRawOperand(1);
      auto RegName = GetNameForDWARFReg(DwarfRegNum, false);
      if (RegName.empty())
        return false;
      raw_svector_ostream S(Stack.emplace_back().String);
      S << RegName;
      if (Offset)
        S << formatv("{0:+d}", Offset);
      break;
    }
    case dwarf::DW_OP_entry_value:
    case dwarf::DW_OP_GNU_entry_value: {
      // DW_OP_entry_value contains a sub-expression which must be rendered
      // separately.
      uint64_t SubExprLength = Op.getRawOperand(0);
      DWARFExpression::iterator SubExprEnd = I.skipBytes(SubExprLength);
      ++I;
      raw_svector_ostream S(Stack.emplace_back().String);
      S << "entry(";
      printCompactDWARFExpr(S, I, SubExprEnd, GetNameForDWARFReg);
      S << ")";
      I = SubExprEnd;
      continue;
    }
    case dwarf::DW_OP_stack_value: {
      // The top stack entry should be treated as the actual value of tne
```
- EN: This section centers on `S`, `printCompactDWARFExpr` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `S`, `printCompactDWARFExpr` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: In this range, the code uses a `switch` to dispatch behavior by kind or encoding and checks conditions and handles edge cases.
  CN: 在这一段中，代码使用 `switch` 按类型或编码分派行为，并检查条件并处理边界情况。

### Lines 281-320

```cpp
      // variable, rather than the address of the variable in memory.
      assert(!Stack.empty());
      Stack.back().Kind = PrintedExpr::Value;
      break;
    }
    case dwarf::DW_OP_nop: {
      break;
    }
    case dwarf::DW_OP_LLVM_user: {
      std::optional<unsigned> SubOpcode = Op.getSubCode();
      if (SubOpcode == dwarf::DW_OP_LLVM_nop)
        break;
      return UnknownOpcode(OS, Opcode, SubOpcode);
    }
    default:
      if (Opcode >= dwarf::DW_OP_reg0 && Opcode <= dwarf::DW_OP_reg31) {
        // DW_OP_reg<N>: A register, with the register num implied by the
        // opcode. Printed as the plain register name.
        uint64_t DwarfRegNum = Opcode - dwarf::DW_OP_reg0;
        auto RegName = GetNameForDWARFReg(DwarfRegNum, false);
        if (RegName.empty())
          return false;
        raw_svector_ostream S(Stack.emplace_back(PrintedExpr::Value).String);
        S << RegName;
      } else if (Opcode >= dwarf::DW_OP_breg0 &&
                 Opcode <= dwarf::DW_OP_breg31) {
        int DwarfRegNum = Opcode - dwarf::DW_OP_breg0;
        int64_t Offset = Op.getRawOperand(0);
        auto RegName = GetNameForDWARFReg(DwarfRegNum, false);
        if (RegName.empty())
          return false;
        raw_svector_ostream S(Stack.emplace_back().String);
        S << RegName;
        if (Offset)
          S << formatv("{0:+d}", Offset);
      } else {
        return UnknownOpcode(OS, Opcode, std::nullopt);
      }
      break;
    }
```
- EN: This section centers on `assert`, `UnknownOpcode`, `S` and implements the local control flow and bookkeeping.
  CN: 这一段主要围绕 `assert`, `UnknownOpcode`, `S` 等符号展开，负责实现局部控制流程与状态维护。
- EN: In this range, the code checks conditions and handles edge cases and propagates LLVM-style errors and invariants.
  CN: 在这一段中，代码检查条件并处理边界情况，并传播 LLVM 风格的错误处理并维护不变量。

### Lines 321-360

```cpp
    ++I;
  }

  if (Stack.size() != 1) {
    OS << "<stack of size " << Stack.size() << ", expected 1>";
    return false;
  }

  if (Stack.front().Kind == PrintedExpr::Address)
    OS << "[" << Stack.front().String << "]";
  else
    OS << Stack.front().String;

  return true;
}

bool printDwarfExpressionCompact(
    const DWARFExpression *E, raw_ostream &OS,
    std::function<StringRef(uint64_t RegNum, bool IsEH)> GetNameForDWARFReg) {
  return printCompactDWARFExpr(OS, E->begin(), E->end(), GetNameForDWARFReg);
}

bool prettyPrintRegisterOp(DWARFUnit *U, raw_ostream &OS,
                           DIDumpOptions DumpOpts, uint8_t Opcode,
                           ArrayRef<uint64_t> Operands) {
  if (!DumpOpts.GetNameForDWARFReg)
    return false;

  uint64_t DwarfRegNum;
  unsigned OpNum = 0;

  std::optional<unsigned> SubOpcode;
  if (Opcode == DW_OP_LLVM_user)
    SubOpcode = Operands[OpNum++];

  if (Opcode == DW_OP_bregx || Opcode == DW_OP_regx ||
      Opcode == DW_OP_regval_type || SubOpcode == DW_OP_LLVM_aspace_bregx ||
      SubOpcode == DW_OP_LLVM_call_frame_entry_reg)
    DwarfRegNum = Operands[OpNum++];
  else if (Opcode >= DW_OP_breg0 && Opcode < DW_OP_bregx)
```
- EN: This section centers on `printDwarfExpressionCompact`, `printCompactDWARFExpr`, `prettyPrintRegisterOp` and writes, formats, or serializes results for downstream consumers.
  CN: 这一段主要围绕 `printDwarfExpressionCompact`, `printCompactDWARFExpr`, `prettyPrintRegisterOp` 等符号展开，负责为下游使用方写出、格式化或序列化结果。
- EN: In this range, the code checks conditions and handles edge cases and returns the resulting value to its callers.
  CN: 在这一段中，代码检查条件并处理边界情况，并将结果返回给调用方。

### Lines 361-381

```cpp
    DwarfRegNum = Opcode - DW_OP_breg0;
  else
    DwarfRegNum = Opcode - DW_OP_reg0;

  auto RegName = DumpOpts.GetNameForDWARFReg(DwarfRegNum, DumpOpts.IsEH);
  if (!RegName.empty()) {
    if ((Opcode >= DW_OP_breg0 && Opcode <= DW_OP_breg31) ||
        Opcode == DW_OP_bregx || SubOpcode == DW_OP_LLVM_aspace_bregx)
      OS << ' ' << RegName << formatv("{0:+d}", int64_t(Operands[OpNum]));
    else
      OS << ' ' << RegName.data();

    if (Opcode == DW_OP_regval_type)
      prettyPrintBaseTypeRef(U, OS, DumpOpts, Operands, 1);
    return true;
  }

  return false;
}

} // namespace llvm
```
- EN: In this range, the code checks conditions and handles edge cases and returns the resulting value to its callers.
  CN: 在这一段中，代码检查条件并处理边界情况，并将结果返回给调用方。

## Key Concepts / 关键概念
- Domain / 领域: DWARF debug information / DWARF 调试信息
- Core symbols / 核心符号: `PrintedExpr`, `ExprKind`, `prettyPrintBaseTypeRef`, `assert`, `printOp`, `printDwarfExpression` / 该文件围绕这些类型或函数组织主要逻辑。
- Data flow / 数据流: Formatting, emission, and outward serialization. / 重点关注格式化、输出与序列化。
- Error model / 错误模型: LLVM-style `Error`/`Expected` handling and invariant checks. / 使用 LLVM 风格的 `Error`/`Expected` 处理与不变量检查。
- Data structures / 数据结构: Relies on LLVM or STL containers for compact state management. / 依赖 LLVM 或 STL 容器管理紧凑状态。

## Dependencies / 依赖关系
- LLVM headers / LLVM 头文件: `llvm/DebugInfo/DWARF/DWARFExpressionPrinter.h`, `llvm/ADT/SmallString.h`, `llvm/DebugInfo/DWARF/DWARFUnit.h`, `llvm/DebugInfo/DWARF/LowLevel/DWARFExpression.h`, `llvm/Support/Format.h`, `llvm/Support/FormatVariadic.h`
- Standard library / 标准库: `cstdint`
- Other/system headers / 其他或系统头文件: `cassert`
- Related symbols / 相关符号: `PrintedExpr`, `ExprKind`, `prettyPrintBaseTypeRef`, `assert`, `printOp`, `printDwarfExpression`
