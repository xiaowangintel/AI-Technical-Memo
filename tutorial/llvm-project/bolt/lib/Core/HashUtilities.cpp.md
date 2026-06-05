# HashUtilities.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Core/HashUtilities.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: Misc hash utilities. It also sits in code that implements BOLT core binary-analysis logic. / 该文件实现 BOLT 核心二进制分析逻辑。 源码头部说明其职责是：Misc hash utilities。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```cpp
//===- bolt/Core/HashUtilities.cpp - Misc hash utilities ------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Computation of hash values over BinaryFunction and BinaryBasicBlock.
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 13-20

```cpp
#include "bolt/Core/HashUtilities.h"
#include "bolt/Core/BinaryContext.h"
#include "bolt/Utils/NameResolver.h"
#include "llvm/MC/MCInstPrinter.h"

namespace llvm {
namespace bolt {
```

- EN: Pulls in 4 header(s) from local project, LLVM dependencies needed by this range. Works inside namespace scope `llvm`, `bolt` to organize symbols. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `llvm`, `bolt`.
- CN: 这里引入 4 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里位于命名空间 `llvm`, `bolt` 中，用于组织符号作用域。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `llvm`, `bolt`。

### Lines 21-31

```cpp
std::string hashInteger(uint64_t Value) {
  std::string HashString;
  if (Value == 0)
    HashString.push_back(0);

  while (Value) {
    uint8_t LSB = Value & 0xff;
    HashString.push_back(LSB);
    Value >>= 8;
  }
```

- EN: Declares or implements routines including `hashInteger`. Notable symbols here include `hashInteger`.
- CN: 这里声明或实现函数，例如 `hashInteger`。这里较值得关注的符号包括 `hashInteger`。

### Lines 32-41

```cpp
  return HashString;
}

std::string hashSymbol(BinaryContext &BC, const MCSymbol &Symbol) {
  std::string HashString;

  // Ignore function references.
  if (BC.getFunctionForSymbol(&Symbol))
    return HashString;
```

- EN: Declares or implements routines including `hashSymbol`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `hashSymbol`.
- CN: 这里声明或实现函数，例如 `hashSymbol`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `hashSymbol`。

### Lines 42-49

```cpp
  llvm::ErrorOr<uint64_t> ErrorOrValue = BC.getSymbolValue(Symbol);
  if (!ErrorOrValue)
    return HashString;

  // Ignore jump table references.
  if (BC.getJumpTableContainingAddress(*ErrorOrValue))
    return HashString;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 50-67

```cpp
  return HashString.append(hashInteger(*ErrorOrValue));
}

std::string hashExpr(BinaryContext &BC, const MCExpr &Expr) {
  switch (Expr.getKind()) {
  case MCExpr::Constant:
    return hashInteger(cast<MCConstantExpr>(Expr).getValue());
  case MCExpr::SymbolRef:
    return hashSymbol(BC, cast<MCSymbolRefExpr>(Expr).getSymbol());
  case MCExpr::Unary: {
    const auto &UnaryExpr = cast<MCUnaryExpr>(Expr);
    return hashInteger(UnaryExpr.getOpcode())
        .append(hashExpr(BC, *UnaryExpr.getSubExpr()));
  }
  case MCExpr::Binary: {
    const auto &BinaryExpr = cast<MCBinaryExpr>(Expr);
    return hashExpr(BC, *BinaryExpr.getLHS())
        .append(hashInteger(BinaryExpr.getOpcode()))
```

- EN: Declares or implements routines including `hashExpr`. Notable symbols here include `hashExpr`.
- CN: 这里声明或实现函数，例如 `hashExpr`。这里较值得关注的符号包括 `hashExpr`。

### Lines 68-78

```cpp
        .append(hashExpr(BC, *BinaryExpr.getRHS()));
  }
  case MCExpr::Specifier: {
    const auto &SpecExpr = cast<MCSpecifierExpr>(Expr);
    return hashInteger(SpecExpr.getSpecifier())
        .append(hashExpr(BC, *SpecExpr.getSubExpr()));
  }
  case MCExpr::Target:
    return std::string();
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 79-89

```cpp
  llvm_unreachable("invalid expression kind");
}

std::string hashInstOperand(BinaryContext &BC, const MCOperand &Operand) {
  if (Operand.isImm())
    return hashInteger(Operand.getImm());
  if (Operand.isReg())
    return hashInteger(Operand.getReg());
  if (Operand.isExpr())
    return hashExpr(BC, *Operand.getExpr());
```

- EN: Declares or implements routines including `llvm_unreachable`, `hashInstOperand`. Notable symbols here include `llvm_unreachable`, `hashInstOperand`.
- CN: 这里声明或实现函数，例如 `llvm_unreachable`, `hashInstOperand`。这里较值得关注的符号包括 `llvm_unreachable`, `hashInstOperand`。

### Lines 90-100

```cpp
  return std::string();
}

std::string hashBlock(BinaryContext &BC, const BinaryBasicBlock &BB,
                      OperandHashFuncTy OperandHashFunc) {
  const bool IsX86 = BC.isX86();

  // The hash is computed by creating a string of all instruction opcodes and
  // possibly their operands and then hashing that string with std::hash.
  std::string HashString;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 101-112

```cpp
  for (const MCInst &Inst : BB) {
    if (BC.MIB->isPseudo(Inst))
      continue;

    unsigned Opcode = Inst.getOpcode();

    // Ignore unconditional jumps since we check CFG consistency by processing
    // basic blocks in order and do not rely on branches to be in-sync with
    // CFG. Note that we still use condition code of conditional jumps.
    if (BC.MIB->isUnconditionalBranch(Inst))
      continue;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 113-122

```cpp
    if (IsX86 && BC.MIB->isConditionalBranch(Inst))
      Opcode = BC.MIB->getShortBranchOpcode(Opcode);

    if (Opcode == 0) {
      HashString.push_back(0);
    } else {
      StringRef OpcodeName = BC.InstPrinter->getOpcodeName(Opcode);
      HashString.append(OpcodeName.str());
    }
```

- EN: Declares or implements routines including `getShortBranchOpcode`, `getOpcodeName`. Notable symbols here include `getShortBranchOpcode`, `getOpcodeName`.
- CN: 这里声明或实现函数，例如 `getShortBranchOpcode`, `getOpcodeName`。这里较值得关注的符号包括 `getShortBranchOpcode`, `getOpcodeName`。

### Lines 123-140

```cpp
    for (const MCOperand &Op : MCPlus::primeOperands(Inst))
      HashString.append(OperandHashFunc(Op));
  }
  return HashString;
}

/// A "loose" hash of a basic block to use with the stale profile matching. The
/// computed value will be the same for blocks with minor changes (such as
/// reordering of instructions or using different operands) but may result in
/// collisions that need to be resolved by a stronger hashing.
std::string hashBlockLoose(BinaryContext &BC, const BinaryBasicBlock &BB) {
  // The hash is computed by creating a string of all lexicographically ordered
  // instruction opcodes, which is then hashed with std::hash.
  std::set<std::string> Opcodes;
  for (const MCInst &Inst : BB) {
    // Skip pseudo instructions and nops.
    if (BC.MIB->isPseudo(Inst) || BC.MIB->isNoop(Inst))
      continue;
```

- EN: Declares or implements routines including `hashBlockLoose`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `hashBlockLoose`.
- CN: 这里声明或实现函数，例如 `hashBlockLoose`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `hashBlockLoose`。

### Lines 141-152

```cpp

    // Ignore unconditional jumps, as they can be added / removed as a result
    // of basic block reordering.
    if (BC.MIB->isUnconditionalBranch(Inst))
      continue;

    // Do not distinguish different types of conditional jumps.
    if (BC.MIB->isConditionalBranch(Inst)) {
      Opcodes.insert("JMP");
      continue;
    }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 153-163

```cpp
    std::string Mnemonic = BC.InstPrinter->getMnemonic(Inst).first;
    llvm::erase_if(Mnemonic, [](unsigned char ch) { return std::isspace(ch); });
    Opcodes.insert(Mnemonic);
  }

  std::string HashString;
  for (const std::string &Opcode : Opcodes)
    HashString.append(Opcode);
  return HashString;
}
```

- EN: Declares or implements routines including `getMnemonic`, `erase_if`. Notable symbols here include `getMnemonic`, `erase_if`.
- CN: 这里声明或实现函数，例如 `getMnemonic`, `erase_if`。这里较值得关注的符号包括 `getMnemonic`, `erase_if`。

### Lines 164-181

```cpp
/// An even looser hash level relative to $ hashBlockLoose to use with stale
/// profile matching, composed of the names of a block's called functions in
/// lexicographic order.
std::string hashBlockCalls(BinaryContext &BC, const BinaryBasicBlock &BB) {
  // The hash is computed by creating a string of all lexicographically ordered
  // called function names.
  std::vector<std::string> FunctionNames;
  for (const MCInst &Instr : BB) {
    // Skip non-call instructions.
    if (!BC.MIB->isCall(Instr))
      continue;
    const MCSymbol *CallSymbol = BC.MIB->getTargetSymbol(Instr);
    if (!CallSymbol)
      continue;
    FunctionNames.push_back(std::string(CallSymbol->getName()));
  }
  std::sort(FunctionNames.begin(), FunctionNames.end());
  std::string HashString;
```

- EN: Declares or implements routines including `hashBlockCalls`, `getTargetSymbol`, `sort`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `hashBlockCalls`, `getTargetSymbol`, `sort`.
- CN: 这里声明或实现函数，例如 `hashBlockCalls`, `getTargetSymbol`, `sort`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `hashBlockCalls`, `getTargetSymbol`, `sort`。

### Lines 182-199

```cpp
  for (const std::string &FunctionName : FunctionNames)
    HashString.append(FunctionName);

  return HashString;
}

/// The same as the $hashBlockCalls function, but for profiled functions.
std::string
hashBlockCalls(const DenseMap<uint32_t, yaml::bolt::BinaryFunctionProfile *>
                   &IdToYamlFunction,
               const yaml::bolt::BinaryBasicBlockProfile &YamlBB) {
  std::vector<std::string> FunctionNames;
  for (const yaml::bolt::CallSiteInfo &CallSiteInfo : YamlBB.CallSites) {
    auto It = IdToYamlFunction.find(CallSiteInfo.DestId);
    if (It == IdToYamlFunction.end())
      continue;
    StringRef Name = NameResolver::dropNumNames(It->second->Name);
    FunctionNames.push_back(std::string(Name));
```

- EN: Declares or implements routines including `dropNumNames`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `dropNumNames`.
- CN: 这里声明或实现函数，例如 `dropNumNames`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `dropNumNames`。

### Lines 200-208

```cpp
  }
  std::sort(FunctionNames.begin(), FunctionNames.end());
  std::string HashString;
  for (const std::string &FunctionName : FunctionNames)
    HashString.append(FunctionName);

  return HashString;
}
```

- EN: Declares or implements routines including `sort`. Notable symbols here include `sort`.
- CN: 这里声明或实现函数，例如 `sort`。这里较值得关注的符号包括 `sort`。

### Lines 209-210

```cpp
} // namespace bolt
} // namespace llvm
```

- EN: Works inside namespace scope `bolt`, `llvm` to organize symbols. Notable symbols here include `bolt`, `llvm`.
- CN: 这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里较值得关注的符号包括 `bolt`, `llvm`。

## Key Concepts / 关键概念

- `hashInteger`: function or method entry point / 函数或方法入口
- `hashSymbol`: function or method entry point / 函数或方法入口
- `hashExpr`: function or method entry point / 函数或方法入口
- `llvm_unreachable`: function or method entry point / 函数或方法入口
- `hashInstOperand`: function or method entry point / 函数或方法入口
- `llvm`: namespace scope / 命名空间作用域
- `bolt`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Core/HashUtilities.h`, `bolt/Core/BinaryContext.h`, `bolt/Utils/NameResolver.h`
- LLVM headers / LLVM 头文件: `llvm/MC/MCInstPrinter.h`
- Directory context / 目录上下文: `bolt/lib/Core` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Core` 下的相邻文件通常与本文件协作组成对应子系统
