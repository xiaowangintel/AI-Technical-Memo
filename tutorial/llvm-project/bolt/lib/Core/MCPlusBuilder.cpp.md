# MCPlusBuilder.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Core/MCPlusBuilder.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: Interface for MCPlus. It also sits in code that implements BOLT core binary-analysis logic. / 该文件实现 BOLT 核心二进制分析逻辑。 源码头部说明其职责是：Interface for MCPlus。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```cpp
//===- bolt/Core/MCPlusBuilder.cpp - Interface for MCPlus -----------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the MCPlusBuilder class.
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 13-25

```cpp
#include "bolt/Core/MCPlusBuilder.h"
#include "bolt/Core/MCPlus.h"
#include "bolt/Utils/CommandLineOpts.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCInstrAnalysis.h"
#include "llvm/MC/MCInstrDesc.h"
#include "llvm/MC/MCInstrInfo.h"
#include "llvm/MC/MCRegisterInfo.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/Debug.h"
#include <cstdint>
```

- EN: Pulls in 12 header(s) from local project, LLVM, system dependencies needed by this range. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 12 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里的注释记录了设计假设、不变量或使用说明。

### Lines 26-37

```cpp
#define DEBUG_TYPE "mcplus"

using namespace llvm;
using namespace bolt;
using namespace MCPlus;

namespace opts {
cl::opt<bool>
    TerminalHLT("terminal-x86-hlt",
                cl::desc("Assume that execution stops at x86 HLT instruction"),
                cl::init(true), cl::Hidden, cl::cat(BoltCategory));
```

- EN: Establishes header-guard or prologue macros so the file can be compiled safely. Works inside namespace scope `llvm`, `bolt`, `MCPlus`, `opts` to organize symbols. Declares or implements routines including `desc`, `init`. Defines macros such as `DEBUG_TYPE` for constants or compile-time switches.
- CN: 这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里位于命名空间 `llvm`, `bolt`, `MCPlus`, `opts` 中，用于组织符号作用域。这里声明或实现函数，例如 `desc`, `init`。这里定义宏 `DEBUG_TYPE`，用于常量或编译期开关。

### Lines 38-48

```cpp
cl::opt<bool>
    TerminalTrap("terminal-trap",
                 cl::desc("Assume that execution stops at trap instruction"),
                 cl::init(true), cl::Hidden, cl::cat(BoltCategory));
}

bool MCPlusBuilder::equals(const MCInst &A, const MCInst &B,
                           CompFuncTy Comp) const {
  if (A.getOpcode() != B.getOpcode())
    return false;
```

- EN: Declares or implements routines including `desc`, `init`. Notable symbols here include `desc`, `init`.
- CN: 这里声明或实现函数，例如 `desc`, `init`。这里较值得关注的符号包括 `desc`, `init`。

### Lines 49-56

```cpp
  unsigned NumOperands = MCPlus::getNumPrimeOperands(A);
  if (NumOperands != MCPlus::getNumPrimeOperands(B))
    return false;

  for (unsigned Index = 0; Index < NumOperands; ++Index)
    if (!equals(A.getOperand(Index), B.getOperand(Index), Comp))
      return false;
```

- EN: Declares or implements routines including `getNumPrimeOperands`. Notable symbols here include `getNumPrimeOperands`.
- CN: 这里声明或实现函数，例如 `getNumPrimeOperands`。这里较值得关注的符号包括 `getNumPrimeOperands`。

### Lines 57-74

```cpp
  return true;
}

bool MCPlusBuilder::equals(const MCOperand &A, const MCOperand &B,
                           CompFuncTy Comp) const {
  if (A.isReg()) {
    if (!B.isReg())
      return false;
    return A.getReg() == B.getReg();
  } else if (A.isImm()) {
    if (!B.isImm())
      return false;
    return A.getImm() == B.getImm();
  } else if (A.isSFPImm()) {
    if (!B.isSFPImm())
      return false;
    return A.getSFPImm() == B.getSFPImm();
  } else if (A.isDFPImm()) {
```

- EN: Declares or implements routines including `if`. Notable symbols here include `if`.
- CN: 这里声明或实现函数，例如 `if`。这里较值得关注的符号包括 `if`。

### Lines 75-87

```cpp
    if (!B.isDFPImm())
      return false;
    return A.getDFPImm() == B.getDFPImm();
  } else if (A.isExpr()) {
    if (!B.isExpr())
      return false;
    return equals(*A.getExpr(), *B.getExpr(), Comp);
  } else {
    llvm_unreachable("unexpected operand kind");
    return false;
  }
}
```

- EN: Declares or implements routines including `if`, `llvm_unreachable`. Notable symbols here include `if`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `if`, `llvm_unreachable`。这里较值得关注的符号包括 `if`, `llvm_unreachable`。

### Lines 88-99

```cpp
bool MCPlusBuilder::equals(const MCExpr &A, const MCExpr &B,
                           CompFuncTy Comp) const {
  if (A.getKind() != B.getKind())
    return false;

  switch (A.getKind()) {
  case MCExpr::Constant: {
    const auto &ConstA = cast<MCConstantExpr>(A);
    const auto &ConstB = cast<MCConstantExpr>(B);
    return ConstA.getValue() == ConstB.getValue();
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 100-113

```cpp
  case MCExpr::SymbolRef: {
    const MCSymbolRefExpr &SymbolA = cast<MCSymbolRefExpr>(A);
    const MCSymbolRefExpr &SymbolB = cast<MCSymbolRefExpr>(B);
    return SymbolA.getKind() == SymbolB.getKind() &&
           Comp(&SymbolA.getSymbol(), &SymbolB.getSymbol());
  }

  case MCExpr::Unary: {
    const auto &UnaryA = cast<MCUnaryExpr>(A);
    const auto &UnaryB = cast<MCUnaryExpr>(B);
    return UnaryA.getOpcode() == UnaryB.getOpcode() &&
           equals(*UnaryA.getSubExpr(), *UnaryB.getSubExpr(), Comp);
  }
```

- EN: Declares or implements routines including `Comp`, `equals`. Notable symbols here include `Comp`, `equals`.
- CN: 这里声明或实现函数，例如 `Comp`, `equals`。这里较值得关注的符号包括 `Comp`, `equals`。

### Lines 114-121

```cpp
  case MCExpr::Binary: {
    const auto &BinaryA = cast<MCBinaryExpr>(A);
    const auto &BinaryB = cast<MCBinaryExpr>(B);
    return BinaryA.getOpcode() == BinaryB.getOpcode() &&
           equals(*BinaryA.getLHS(), *BinaryB.getLHS(), Comp) &&
           equals(*BinaryA.getRHS(), *BinaryB.getRHS(), Comp);
  }
```

- EN: Declares or implements routines including `equals`. Notable symbols here include `equals`.
- CN: 这里声明或实现函数，例如 `equals`。这里较值得关注的符号包括 `equals`。

### Lines 122-130

```cpp
  case MCExpr::Specifier: {
    const auto &TargetExprA = cast<MCSpecifierExpr>(A);
    const auto &TargetExprB = cast<MCSpecifierExpr>(B);
    return equals(TargetExprA, TargetExprB, Comp);
  }
  case MCExpr::Target:
    llvm_unreachable("Not implemented");
  }
```

- EN: Declares or implements routines including `llvm_unreachable`. Notable symbols here include `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `llvm_unreachable`。这里较值得关注的符号包括 `llvm_unreachable`。

### Lines 131-138

```cpp
  llvm_unreachable("Invalid expression kind!");
}

bool MCPlusBuilder::equals(const MCSpecifierExpr &A, const MCSpecifierExpr &B,
                           CompFuncTy Comp) const {
  llvm_unreachable("target-specific expressions are unsupported");
}
```

- EN: Declares or implements routines including `llvm_unreachable`. Notable symbols here include `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `llvm_unreachable`。这里较值得关注的符号包括 `llvm_unreachable`。

### Lines 139-148

```cpp
bool MCPlusBuilder::isTerminator(const MCInst &Inst) const {
  if (isX86HLT(Inst))
    return opts::TerminalHLT;

  if (Info->get(Inst.getOpcode()).isTrap())
    return opts::TerminalTrap;

  return Analysis->isTerminator(Inst);
}
```

- EN: Declares or implements routines including `isTerminator`. Notable symbols here include `isTerminator`.
- CN: 这里声明或实现函数，例如 `isTerminator`。这里较值得关注的符号包括 `isTerminator`。

### Lines 149-161

```cpp
void MCPlusBuilder::setTailCall(MCInst &Inst) const {
  assert(!hasAnnotation(Inst, MCAnnotation::kTailCall));
  setAnnotationOpValue(Inst, MCAnnotation::kTailCall, true);
}

bool MCPlusBuilder::isTailCall(const MCInst &Inst) const {
  if (hasAnnotation(Inst, MCAnnotation::kTailCall))
    return true;
  if (getConditionalTailCall(Inst))
    return true;
  return false;
}
```

- EN: Declares or implements routines including `setTailCall`, `assert`, `setAnnotationOpValue`, `isTailCall`. Notable symbols here include `setTailCall`, `assert`, `setAnnotationOpValue`, `isTailCall`.
- CN: 这里声明或实现函数，例如 `setTailCall`, `assert`, `setAnnotationOpValue`, `isTailCall`。这里较值得关注的符号包括 `setTailCall`, `assert`, `setAnnotationOpValue`, `isTailCall`。

### Lines 162-170

```cpp
void MCPlusBuilder::setNegateRAState(MCInst &Inst) const {
  assert(!hasAnnotation(Inst, MCAnnotation::kNegateState));
  setAnnotationOpValue(Inst, MCAnnotation::kNegateState, true);
}

bool MCPlusBuilder::hasNegateRAState(const MCInst &Inst) const {
  return hasAnnotation(Inst, MCAnnotation::kNegateState);
}
```

- EN: Declares or implements routines including `setNegateRAState`, `assert`, `setAnnotationOpValue`, `hasNegateRAState`. Notable symbols here include `setNegateRAState`, `assert`, `setAnnotationOpValue`, `hasNegateRAState`.
- CN: 这里声明或实现函数，例如 `setNegateRAState`, `assert`, `setAnnotationOpValue`, `hasNegateRAState`。这里较值得关注的符号包括 `setNegateRAState`, `assert`, `setAnnotationOpValue`, `hasNegateRAState`。

### Lines 171-179

```cpp
void MCPlusBuilder::setRememberState(MCInst &Inst) const {
  assert(!hasAnnotation(Inst, MCAnnotation::kRememberState));
  setAnnotationOpValue(Inst, MCAnnotation::kRememberState, true);
}

bool MCPlusBuilder::hasRememberState(const MCInst &Inst) const {
  return hasAnnotation(Inst, MCAnnotation::kRememberState);
}
```

- EN: Declares or implements routines including `setRememberState`, `assert`, `setAnnotationOpValue`, `hasRememberState`. Notable symbols here include `setRememberState`, `assert`, `setAnnotationOpValue`, `hasRememberState`.
- CN: 这里声明或实现函数，例如 `setRememberState`, `assert`, `setAnnotationOpValue`, `hasRememberState`。这里较值得关注的符号包括 `setRememberState`, `assert`, `setAnnotationOpValue`, `hasRememberState`。

### Lines 180-188

```cpp
void MCPlusBuilder::setRestoreState(MCInst &Inst) const {
  assert(!hasAnnotation(Inst, MCAnnotation::kRestoreState));
  setAnnotationOpValue(Inst, MCAnnotation::kRestoreState, true);
}

bool MCPlusBuilder::hasRestoreState(const MCInst &Inst) const {
  return hasAnnotation(Inst, MCAnnotation::kRestoreState);
}
```

- EN: Declares or implements routines including `setRestoreState`, `assert`, `setAnnotationOpValue`, `hasRestoreState`. Notable symbols here include `setRestoreState`, `assert`, `setAnnotationOpValue`, `hasRestoreState`.
- CN: 这里声明或实现函数，例如 `setRestoreState`, `assert`, `setAnnotationOpValue`, `hasRestoreState`。这里较值得关注的符号包括 `setRestoreState`, `assert`, `setAnnotationOpValue`, `hasRestoreState`。

### Lines 189-197

```cpp
void MCPlusBuilder::setRAState(MCInst &Inst, bool State) const {
  assert(!hasAnnotation(Inst, MCAnnotation::kRASigned));
  assert(!hasAnnotation(Inst, MCAnnotation::kRAUnsigned));
  if (State)
    setAnnotationOpValue(Inst, MCAnnotation::kRASigned, true);
  else
    setAnnotationOpValue(Inst, MCAnnotation::kRAUnsigned, true);
}
```

- EN: Declares or implements routines including `setRAState`, `assert`, `setAnnotationOpValue`. Notable symbols here include `setRAState`, `assert`, `setAnnotationOpValue`.
- CN: 这里声明或实现函数，例如 `setRAState`, `assert`, `setAnnotationOpValue`。这里较值得关注的符号包括 `setRAState`, `assert`, `setAnnotationOpValue`。

### Lines 198-205

```cpp
std::optional<bool> MCPlusBuilder::getRAState(const MCInst &Inst) const {
  if (hasAnnotation(Inst, MCAnnotation::kRASigned))
    return true;
  if (hasAnnotation(Inst, MCAnnotation::kRAUnsigned))
    return false;
  return std::nullopt;
}
```

- EN: Declares or implements routines including `getRAState`. Notable symbols here include `getRAState`.
- CN: 这里声明或实现函数，例如 `getRAState`。这里较值得关注的符号包括 `getRAState`。

### Lines 206-217

```cpp
std::optional<MCLandingPad> MCPlusBuilder::getEHInfo(const MCInst &Inst) const {
  if (!isCall(Inst))
    return std::nullopt;
  std::optional<int64_t> LPSym =
      getAnnotationOpValue(Inst, MCAnnotation::kEHLandingPad);
  if (!LPSym)
    return std::nullopt;
  std::optional<int64_t> Action =
      getAnnotationOpValue(Inst, MCAnnotation::kEHAction);
  if (!Action)
    return std::nullopt;
```

- EN: Declares or implements routines including `getEHInfo`, `getAnnotationOpValue`. Notable symbols here include `getEHInfo`, `getAnnotationOpValue`.
- CN: 这里声明或实现函数，例如 `getEHInfo`, `getAnnotationOpValue`。这里较值得关注的符号包括 `getEHInfo`, `getAnnotationOpValue`。

### Lines 218-231

```cpp
  return std::make_pair(reinterpret_cast<const MCSymbol *>(*LPSym),
                        static_cast<uint64_t>(*Action));
}

void MCPlusBuilder::addEHInfo(MCInst &Inst, const MCLandingPad &LP) const {
  if (isCall(Inst)) {
    assert(!getEHInfo(Inst));
    setAnnotationOpValue(Inst, MCAnnotation::kEHLandingPad,
                         reinterpret_cast<int64_t>(LP.first));
    setAnnotationOpValue(Inst, MCAnnotation::kEHAction,
                         static_cast<int64_t>(LP.second));
  }
}
```

- EN: Declares or implements routines including `addEHInfo`, `assert`. Notable symbols here include `addEHInfo`, `assert`.
- CN: 这里声明或实现函数，例如 `addEHInfo`, `assert`。这里较值得关注的符号包括 `addEHInfo`, `assert`。

### Lines 232-242

```cpp
bool MCPlusBuilder::updateEHInfo(MCInst &Inst, const MCLandingPad &LP) const {
  if (!isInvoke(Inst))
    return false;

  setAnnotationOpValue(Inst, MCAnnotation::kEHLandingPad,
                       reinterpret_cast<int64_t>(LP.first));
  setAnnotationOpValue(Inst, MCAnnotation::kEHAction,
                       static_cast<int64_t>(LP.second));
  return true;
}
```

- EN: Declares or implements routines including `updateEHInfo`. Notable symbols here include `updateEHInfo`.
- CN: 这里声明或实现函数，例如 `updateEHInfo`。这里较值得关注的符号包括 `updateEHInfo`。

### Lines 243-250

```cpp
int64_t MCPlusBuilder::getGnuArgsSize(const MCInst &Inst) const {
  std::optional<int64_t> Value =
      getAnnotationOpValue(Inst, MCAnnotation::kGnuArgsSize);
  if (!Value)
    return -1LL;
  return *Value;
}
```

- EN: Declares or implements routines including `getGnuArgsSize`, `getAnnotationOpValue`. Notable symbols here include `getGnuArgsSize`, `getAnnotationOpValue`.
- CN: 这里声明或实现函数，例如 `getGnuArgsSize`, `getAnnotationOpValue`。这里较值得关注的符号包括 `getGnuArgsSize`, `getAnnotationOpValue`。

### Lines 251-258

```cpp
void MCPlusBuilder::addGnuArgsSize(MCInst &Inst, int64_t GnuArgsSize) const {
  assert(GnuArgsSize >= 0 && "cannot set GNU_args_size to negative value");
  assert(getGnuArgsSize(Inst) == -1LL && "GNU_args_size already set");
  assert(isInvoke(Inst) && "GNU_args_size can only be set for invoke");

  setAnnotationOpValue(Inst, MCAnnotation::kGnuArgsSize, GnuArgsSize);
}
```

- EN: Declares or implements routines including `addGnuArgsSize`, `assert`, `setAnnotationOpValue`. Notable symbols here include `addGnuArgsSize`, `assert`, `setAnnotationOpValue`.
- CN: 这里声明或实现函数，例如 `addGnuArgsSize`, `assert`, `setAnnotationOpValue`。这里较值得关注的符号包括 `addGnuArgsSize`, `assert`, `setAnnotationOpValue`。

### Lines 259-266

```cpp
uint64_t MCPlusBuilder::getJumpTable(const MCInst &Inst) const {
  std::optional<int64_t> Value =
      getAnnotationOpValue(Inst, MCAnnotation::kJumpTable);
  if (!Value)
    return 0;
  return *Value;
}
```

- EN: Declares or implements routines including `getJumpTable`, `getAnnotationOpValue`. Notable symbols here include `getJumpTable`, `getAnnotationOpValue`.
- CN: 这里声明或实现函数，例如 `getJumpTable`, `getAnnotationOpValue`。这里较值得关注的符号包括 `getJumpTable`, `getAnnotationOpValue`。

### Lines 267-279

```cpp
uint16_t MCPlusBuilder::getJumpTableIndexReg(const MCInst &Inst) const {
  return getAnnotationAs<uint16_t>(Inst, "JTIndexReg");
}

bool MCPlusBuilder::setJumpTable(MCInst &Inst, uint64_t Value,
                                 uint16_t IndexReg, AllocatorIdTy AllocId) {
  if (!isIndirectBranch(Inst))
    return false;
  setAnnotationOpValue(Inst, MCAnnotation::kJumpTable, Value);
  getOrCreateAnnotationAs<uint16_t>(Inst, "JTIndexReg", AllocId) = IndexReg;
  return true;
}
```

- EN: Declares or implements routines including `getJumpTableIndexReg`, `setAnnotationOpValue`. Notable symbols here include `getJumpTableIndexReg`, `setAnnotationOpValue`.
- CN: 这里声明或实现函数，例如 `getJumpTableIndexReg`, `setAnnotationOpValue`。这里较值得关注的符号包括 `getJumpTableIndexReg`, `setAnnotationOpValue`。

### Lines 280-287

```cpp
bool MCPlusBuilder::unsetJumpTable(MCInst &Inst) const {
  if (!getJumpTable(Inst))
    return false;
  removeAnnotation(Inst, MCAnnotation::kJumpTable);
  removeAnnotation(Inst, "JTIndexReg");
  return true;
}
```

- EN: Declares or implements routines including `unsetJumpTable`, `removeAnnotation`. Notable symbols here include `unsetJumpTable`, `removeAnnotation`.
- CN: 这里声明或实现函数，例如 `unsetJumpTable`, `removeAnnotation`。这里较值得关注的符号包括 `unsetJumpTable`, `removeAnnotation`。

### Lines 288-296

```cpp
std::optional<uint64_t>
MCPlusBuilder::getConditionalTailCall(const MCInst &Inst) const {
  std::optional<int64_t> Value =
      getAnnotationOpValue(Inst, MCAnnotation::kConditionalTailCall);
  if (!Value)
    return std::nullopt;
  return static_cast<uint64_t>(*Value);
}
```

- EN: Declares or implements routines including `getConditionalTailCall`, `getAnnotationOpValue`. Notable symbols here include `getConditionalTailCall`, `getAnnotationOpValue`.
- CN: 这里声明或实现函数，例如 `getConditionalTailCall`, `getAnnotationOpValue`。这里较值得关注的符号包括 `getConditionalTailCall`, `getAnnotationOpValue`。

### Lines 297-304

```cpp
bool MCPlusBuilder::setConditionalTailCall(MCInst &Inst, uint64_t Dest) const {
  if (!isConditionalBranch(Inst))
    return false;

  setAnnotationOpValue(Inst, MCAnnotation::kConditionalTailCall, Dest);
  return true;
}
```

- EN: Declares or implements routines including `setConditionalTailCall`, `setAnnotationOpValue`. Notable symbols here include `setConditionalTailCall`, `setAnnotationOpValue`.
- CN: 这里声明或实现函数，例如 `setConditionalTailCall`, `setAnnotationOpValue`。这里较值得关注的符号包括 `setConditionalTailCall`, `setAnnotationOpValue`。

### Lines 305-319

```cpp
bool MCPlusBuilder::unsetConditionalTailCall(MCInst &Inst) const {
  if (!getConditionalTailCall(Inst))
    return false;
  removeAnnotation(Inst, MCAnnotation::kConditionalTailCall);
  return true;
}

std::optional<uint32_t> MCPlusBuilder::getOffset(const MCInst &Inst) const {
  std::optional<int64_t> Value =
      getAnnotationOpValue(Inst, MCAnnotation::kOffset);
  if (!Value)
    return std::nullopt;
  return static_cast<uint32_t>(*Value);
}
```

- EN: Declares or implements routines including `unsetConditionalTailCall`, `removeAnnotation`, `getOffset`, `getAnnotationOpValue`. Notable symbols here include `unsetConditionalTailCall`, `removeAnnotation`, `getOffset`, `getAnnotationOpValue`.
- CN: 这里声明或实现函数，例如 `unsetConditionalTailCall`, `removeAnnotation`, `getOffset`, `getAnnotationOpValue`。这里较值得关注的符号包括 `unsetConditionalTailCall`, `removeAnnotation`, `getOffset`, `getAnnotationOpValue`。

### Lines 320-331

```cpp
uint32_t MCPlusBuilder::getOffsetWithDefault(const MCInst &Inst,
                                             uint32_t Default) const {
  if (std::optional<uint32_t> Offset = getOffset(Inst))
    return *Offset;
  return Default;
}

bool MCPlusBuilder::setOffset(MCInst &Inst, uint32_t Offset) const {
  setAnnotationOpValue(Inst, MCAnnotation::kOffset, Offset);
  return true;
}
```

- EN: Declares or implements routines including `setOffset`, `setAnnotationOpValue`. Notable symbols here include `setOffset`, `setAnnotationOpValue`.
- CN: 这里声明或实现函数，例如 `setOffset`, `setAnnotationOpValue`。这里较值得关注的符号包括 `setOffset`, `setAnnotationOpValue`。

### Lines 332-345

```cpp
bool MCPlusBuilder::clearOffset(MCInst &Inst) const {
  if (!hasAnnotation(Inst, MCAnnotation::kOffset))
    return false;
  removeAnnotation(Inst, MCAnnotation::kOffset);
  return true;
}

MCSymbol *MCPlusBuilder::getInstLabel(const MCInst &Inst) const {
  if (std::optional<int64_t> Label =
          getAnnotationOpValue(Inst, MCAnnotation::kLabel))
    return reinterpret_cast<MCSymbol *>(*Label);
  return nullptr;
}
```

- EN: Declares or implements routines including `clearOffset`, `removeAnnotation`, `getInstLabel`, `getAnnotationOpValue`. Notable symbols here include `clearOffset`, `removeAnnotation`, `getInstLabel`, `getAnnotationOpValue`.
- CN: 这里声明或实现函数，例如 `clearOffset`, `removeAnnotation`, `getInstLabel`, `getAnnotationOpValue`。这里较值得关注的符号包括 `clearOffset`, `removeAnnotation`, `getInstLabel`, `getAnnotationOpValue`。

### Lines 346-357

```cpp
MCSymbol *MCPlusBuilder::getOrCreateInstLabel(MCInst &Inst, const Twine &Name,
                                              MCContext *Ctx) const {
  MCSymbol *Label = getInstLabel(Inst);
  if (Label)
    return Label;

  Label = Ctx->createNamedTempSymbol(Name);
  setAnnotationOpValue(Inst, MCAnnotation::kLabel,
                       reinterpret_cast<int64_t>(Label));
  return Label;
}
```

- EN: Declares or implements routines including `getInstLabel`, `createNamedTempSymbol`. Notable symbols here include `getInstLabel`, `createNamedTempSymbol`.
- CN: 这里声明或实现函数，例如 `getInstLabel`, `createNamedTempSymbol`。这里较值得关注的符号包括 `getInstLabel`, `createNamedTempSymbol`。

### Lines 358-370

```cpp
void MCPlusBuilder::setInstLabel(MCInst &Inst, MCSymbol *Label) const {
  assert(!getInstLabel(Inst) && "Instruction already has assigned label.");
  setAnnotationOpValue(Inst, MCAnnotation::kLabel,
                       reinterpret_cast<int64_t>(Label));
}

std::optional<uint32_t> MCPlusBuilder::getSize(const MCInst &Inst) const {
  if (std::optional<int64_t> Value =
          getAnnotationOpValue(Inst, MCAnnotation::kSize))
    return static_cast<uint32_t>(*Value);
  return std::nullopt;
}
```

- EN: Declares or implements routines including `setInstLabel`, `assert`, `getSize`, `getAnnotationOpValue`. Notable symbols here include `setInstLabel`, `assert`, `getSize`, `getAnnotationOpValue`.
- CN: 这里声明或实现函数，例如 `setInstLabel`, `assert`, `getSize`, `getAnnotationOpValue`。这里较值得关注的符号包括 `setInstLabel`, `assert`, `getSize`, `getAnnotationOpValue`。

### Lines 371-381

```cpp
void MCPlusBuilder::setSize(MCInst &Inst, uint32_t Size) const {
  setAnnotationOpValue(Inst, MCAnnotation::kSize, Size);
}

bool MCPlusBuilder::isDynamicBranch(const MCInst &Inst) const {
  if (!hasAnnotation(Inst, MCAnnotation::kDynamicBranch))
    return false;
  assert(isBranch(Inst) && "Branch expected.");
  return true;
}
```

- EN: Declares or implements routines including `setSize`, `setAnnotationOpValue`, `isDynamicBranch`, `assert`. Notable symbols here include `setSize`, `setAnnotationOpValue`, `isDynamicBranch`, `assert`.
- CN: 这里声明或实现函数，例如 `setSize`, `setAnnotationOpValue`, `isDynamicBranch`, `assert`。这里较值得关注的符号包括 `setSize`, `setAnnotationOpValue`, `isDynamicBranch`, `assert`。

### Lines 382-391

```cpp
std::optional<uint32_t>
MCPlusBuilder::getDynamicBranchID(const MCInst &Inst) const {
  if (std::optional<int64_t> Value =
          getAnnotationOpValue(Inst, MCAnnotation::kDynamicBranch)) {
    assert(isBranch(Inst) && "Branch expected.");
    return static_cast<uint32_t>(*Value);
  }
  return std::nullopt;
}
```

- EN: Declares or implements routines including `getDynamicBranchID`, `getAnnotationOpValue`, `assert`. Notable symbols here include `getDynamicBranchID`, `getAnnotationOpValue`, `assert`.
- CN: 这里声明或实现函数，例如 `getDynamicBranchID`, `getAnnotationOpValue`, `assert`。这里较值得关注的符号包括 `getDynamicBranchID`, `getAnnotationOpValue`, `assert`。

### Lines 392-400

```cpp
void MCPlusBuilder::setDynamicBranch(MCInst &Inst, uint32_t ID) const {
  assert(isBranch(Inst) && "Branch expected.");
  setAnnotationOpValue(Inst, MCAnnotation::kDynamicBranch, ID);
}

bool MCPlusBuilder::hasAnnotation(const MCInst &Inst, unsigned Index) const {
  return (bool)getAnnotationOpValue(Inst, Index);
}
```

- EN: Declares or implements routines including `setDynamicBranch`, `assert`, `setAnnotationOpValue`, `hasAnnotation`. Notable symbols here include `setDynamicBranch`, `assert`, `setAnnotationOpValue`, `hasAnnotation`.
- CN: 这里声明或实现函数，例如 `setDynamicBranch`, `assert`, `setAnnotationOpValue`, `hasAnnotation`。这里较值得关注的符号包括 `setDynamicBranch`, `assert`, `setAnnotationOpValue`, `hasAnnotation`。

### Lines 401-415

```cpp
bool MCPlusBuilder::removeAnnotation(MCInst &Inst, unsigned Index) const {
  std::optional<unsigned> FirstAnnotationOp = getFirstAnnotationOpIndex(Inst);
  if (!FirstAnnotationOp)
    return false;

  for (unsigned I = Inst.getNumOperands() - 1; I >= *FirstAnnotationOp; --I) {
    const int64_t ImmValue = Inst.getOperand(I).getImm();
    if (extractAnnotationIndex(ImmValue) == Index) {
      Inst.erase(Inst.begin() + I);
      return true;
    }
  }
  return false;
}
```

- EN: Declares or implements routines including `removeAnnotation`, `getFirstAnnotationOpIndex`. Notable symbols here include `removeAnnotation`, `getFirstAnnotationOpIndex`.
- CN: 这里声明或实现函数，例如 `removeAnnotation`, `getFirstAnnotationOpIndex`。这里较值得关注的符号包括 `removeAnnotation`, `getFirstAnnotationOpIndex`。

### Lines 416-424

```cpp
void MCPlusBuilder::stripAnnotations(MCInst &Inst, bool KeepTC) const {
  KeepTC &= hasAnnotation(Inst, MCAnnotation::kTailCall);

  removeAnnotations(Inst);

  if (KeepTC)
    setTailCall(Inst);
}
```

- EN: Declares or implements routines including `stripAnnotations`, `hasAnnotation`, `removeAnnotations`, `setTailCall`. Notable symbols here include `stripAnnotations`, `hasAnnotation`, `removeAnnotations`, `setTailCall`.
- CN: 这里声明或实现函数，例如 `stripAnnotations`, `hasAnnotation`, `removeAnnotations`, `setTailCall`。这里较值得关注的符号包括 `stripAnnotations`, `hasAnnotation`, `removeAnnotations`, `setTailCall`。

### Lines 425-442

```cpp
void MCPlusBuilder::printAnnotations(const MCInst &Inst, raw_ostream &OS,
                                     bool PrintMemData) const {
  std::optional<unsigned> FirstAnnotationOp = getFirstAnnotationOpIndex(Inst);
  if (!FirstAnnotationOp)
    return;

  for (unsigned I = *FirstAnnotationOp; I < Inst.getNumOperands(); ++I) {
    const int64_t Imm = Inst.getOperand(I).getImm();
    const unsigned Index = extractAnnotationIndex(Imm);
    const int64_t Value = extractAnnotationValue(Imm);
    const auto *Annotation = reinterpret_cast<const MCAnnotation *>(Value);
    if (Index >= MCAnnotation::kGeneric) {
      std::string AnnotationName =
          AnnotationNames[Index - MCAnnotation::kGeneric];
      if (!PrintMemData && AnnotationName == "MemoryAccessProfile")
        continue;
      OS << " # " << AnnotationName << ": ";
      Annotation->print(OS);
```

- EN: Declares or implements routines including `getFirstAnnotationOpIndex`, `extractAnnotationIndex`, `extractAnnotationValue`, `print`. Notable symbols here include `getFirstAnnotationOpIndex`, `extractAnnotationIndex`, `extractAnnotationValue`, `print`.
- CN: 这里声明或实现函数，例如 `getFirstAnnotationOpIndex`, `extractAnnotationIndex`, `extractAnnotationValue`, `print`。这里较值得关注的符号包括 `getFirstAnnotationOpIndex`, `extractAnnotationIndex`, `extractAnnotationValue`, `print`。

### Lines 443-451

```cpp
    }
  }
}

void MCPlusBuilder::getClobberedRegs(const MCInst &Inst,
                                     BitVector &Regs) const {
  if (isPrefix(Inst) || isCFI(Inst))
    return;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 452-462

```cpp
  const MCInstrDesc &InstInfo = Info->get(Inst.getOpcode());

  for (MCPhysReg ImplicitDef : InstInfo.implicit_defs())
    Regs |= getAliases(ImplicitDef, /*OnlySmaller=*/false);

  for (const MCOperand &Operand : defOperands(Inst)) {
    assert(Operand.isReg());
    Regs |= getAliases(Operand.getReg(), /*OnlySmaller=*/false);
  }
}
```

- EN: Declares or implements routines including `get`, `getAliases`, `assert`. Notable symbols here include `get`, `getAliases`, `assert`.
- CN: 这里声明或实现函数，例如 `get`, `getAliases`, `assert`。这里较值得关注的符号包括 `get`, `getAliases`, `assert`。

### Lines 463-473

```cpp
void MCPlusBuilder::getTouchedRegs(const MCInst &Inst, BitVector &Regs) const {
  if (isPrefix(Inst) || isCFI(Inst))
    return;

  const MCInstrDesc &InstInfo = Info->get(Inst.getOpcode());

  for (MCPhysReg ImplicitDef : InstInfo.implicit_defs())
    Regs |= getAliases(ImplicitDef, /*OnlySmaller=*/false);
  for (MCPhysReg ImplicitUse : InstInfo.implicit_uses())
    Regs |= getAliases(ImplicitUse, /*OnlySmaller=*/false);
```

- EN: Declares or implements routines including `getTouchedRegs`, `get`, `getAliases`. Notable symbols here include `getTouchedRegs`, `get`, `getAliases`.
- CN: 这里声明或实现函数，例如 `getTouchedRegs`, `get`, `getAliases`。这里较值得关注的符号包括 `getTouchedRegs`, `get`, `getAliases`。

### Lines 474-484

```cpp
  for (unsigned I = 0, E = Inst.getNumOperands(); I != E; ++I) {
    if (!Inst.getOperand(I).isReg())
      continue;
    Regs |= getAliases(Inst.getOperand(I).getReg(), /*OnlySmaller=*/false);
  }
}

void MCPlusBuilder::getWrittenRegs(const MCInst &Inst, BitVector &Regs) const {
  if (isPrefix(Inst) || isCFI(Inst))
    return;
```

- EN: Declares or implements routines including `getAliases`, `getWrittenRegs`. Notable symbols here include `getAliases`, `getWrittenRegs`.
- CN: 这里声明或实现函数，例如 `getAliases`, `getWrittenRegs`。这里较值得关注的符号包括 `getAliases`, `getWrittenRegs`。

### Lines 485-495

```cpp
  const MCInstrDesc &InstInfo = Info->get(Inst.getOpcode());

  for (MCPhysReg ImplicitDef : InstInfo.implicit_defs())
    Regs |= getAliases(ImplicitDef, /*OnlySmaller=*/true);

  for (const MCOperand &Operand : defOperands(Inst)) {
    assert(Operand.isReg());
    Regs |= getAliases(Operand.getReg(), /*OnlySmaller=*/true);
  }
}
```

- EN: Declares or implements routines including `get`, `getAliases`, `assert`. Notable symbols here include `get`, `getAliases`, `assert`.
- CN: 这里声明或实现函数，例如 `get`, `getAliases`, `assert`。这里较值得关注的符号包括 `get`, `getAliases`, `assert`。

### Lines 496-504

```cpp
void MCPlusBuilder::getUsedRegs(const MCInst &Inst, BitVector &Regs) const {
  if (isPrefix(Inst) || isCFI(Inst))
    return;

  const MCInstrDesc &InstInfo = Info->get(Inst.getOpcode());

  for (MCPhysReg ImplicitUse : InstInfo.implicit_uses())
    Regs |= getAliases(ImplicitUse, /*OnlySmaller=*/true);
```

- EN: Declares or implements routines including `getUsedRegs`, `get`, `getAliases`. Notable symbols here include `getUsedRegs`, `get`, `getAliases`.
- CN: 这里声明或实现函数，例如 `getUsedRegs`, `get`, `getAliases`。这里较值得关注的符号包括 `getUsedRegs`, `get`, `getAliases`。

### Lines 505-515

```cpp
  for (const MCOperand &Operand : useOperands(Inst)) {
    if (!Operand.isReg())
      continue;
    Regs |= getAliases(Operand.getReg(), /*OnlySmaller=*/true);
  }
}

void MCPlusBuilder::getSrcRegs(const MCInst &Inst, BitVector &Regs) const {
  if (isPrefix(Inst) || isCFI(Inst))
    return;
```

- EN: Declares or implements routines including `getAliases`, `getSrcRegs`. Notable symbols here include `getAliases`, `getSrcRegs`.
- CN: 这里声明或实现函数，例如 `getAliases`, `getSrcRegs`。这里较值得关注的符号包括 `getAliases`, `getSrcRegs`。

### Lines 516-523

```cpp
  if (isCall(Inst)) {
    BitVector CallRegs = BitVector(Regs.size(), false);
    getCalleeSavedRegs(CallRegs);
    CallRegs.flip();
    Regs |= CallRegs;
    return;
  }
```

- EN: Declares or implements routines including `BitVector`, `getCalleeSavedRegs`. Notable symbols here include `BitVector`, `getCalleeSavedRegs`.
- CN: 这里声明或实现函数，例如 `BitVector`, `getCalleeSavedRegs`。这里较值得关注的符号包括 `BitVector`, `getCalleeSavedRegs`。

### Lines 524-531

```cpp
  if (isReturn(Inst)) {
    getDefaultLiveOut(Regs);
    return;
  }

  if (isRep(Inst))
    getRepRegs(Regs);
```

- EN: Declares or implements routines including `getDefaultLiveOut`, `getRepRegs`. Notable symbols here include `getDefaultLiveOut`, `getRepRegs`.
- CN: 这里声明或实现函数，例如 `getDefaultLiveOut`, `getRepRegs`。这里较值得关注的符号包括 `getDefaultLiveOut`, `getRepRegs`。

### Lines 532-541

```cpp
  const MCInstrDesc &InstInfo = Info->get(Inst.getOpcode());

  for (MCPhysReg ImplicitUse : InstInfo.implicit_uses())
    Regs |= getAliases(ImplicitUse, /*OnlySmaller=*/true);

  for (const MCOperand &Operand : useOperands(Inst))
    if (Operand.isReg())
      Regs |= getAliases(Operand.getReg(), /*OnlySmaller=*/true);
}
```

- EN: Declares or implements routines including `get`, `getAliases`. Notable symbols here include `get`, `getAliases`.
- CN: 这里声明或实现函数，例如 `get`, `getAliases`。这里较值得关注的符号包括 `get`, `getAliases`。

### Lines 542-559

```cpp
bool MCPlusBuilder::hasDefOfPhysReg(const MCInst &MI, unsigned Reg) const {
  const MCInstrDesc &InstInfo = Info->get(MI.getOpcode());
  return InstInfo.hasDefOfPhysReg(MI, Reg, *RegInfo);
}

bool MCPlusBuilder::hasUseOfPhysReg(const MCInst &MI, unsigned Reg) const {
  const MCInstrDesc &InstInfo = Info->get(MI.getOpcode());
  for (int I = InstInfo.NumDefs; I < InstInfo.NumOperands; ++I)
    if (MI.getOperand(I).isReg() && MI.getOperand(I).getReg() &&
        RegInfo->isSubRegisterEq(Reg, MI.getOperand(I).getReg()))
      return true;
  for (MCPhysReg ImplicitUse : InstInfo.implicit_uses()) {
    if (ImplicitUse == Reg || RegInfo->isSubRegister(Reg, ImplicitUse))
      return true;
  }
  return false;
}
```

- EN: Declares or implements routines including `hasDefOfPhysReg`, `get`, `hasUseOfPhysReg`, `isSubRegisterEq`. Notable symbols here include `hasDefOfPhysReg`, `get`, `hasUseOfPhysReg`, `isSubRegisterEq`.
- CN: 这里声明或实现函数，例如 `hasDefOfPhysReg`, `get`, `hasUseOfPhysReg`, `isSubRegisterEq`。这里较值得关注的符号包括 `hasDefOfPhysReg`, `get`, `hasUseOfPhysReg`, `isSubRegisterEq`。

### Lines 560-576

```cpp
const BitVector &MCPlusBuilder::getAliases(MCPhysReg Reg,
                                           bool OnlySmaller) const {
  if (OnlySmaller)
    return SmallerAliasMap[Reg];
  return AliasMap[Reg];
}

void MCPlusBuilder::initAliases() {
  assert(AliasMap.size() == 0 && SmallerAliasMap.size() == 0);
  // Build alias map
  for (MCPhysReg I = 0, E = RegInfo->getNumRegs(); I != E; ++I) {
    BitVector BV(RegInfo->getNumRegs(), false);
    BV.set(I);
    AliasMap.emplace_back(BV);
    SmallerAliasMap.emplace_back(BV);
  }
```

- EN: Declares or implements routines including `initAliases`, `assert`, `BV`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `initAliases`, `assert`, `BV`.
- CN: 这里声明或实现函数，例如 `initAliases`, `assert`, `BV`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `initAliases`, `assert`, `BV`。

### Lines 577-587

```cpp
  // Cache all aliases for each register
  for (MCPhysReg I = 1, E = RegInfo->getNumRegs(); I != E; ++I) {
    for (MCRegAliasIterator AI(I, RegInfo, true); AI.isValid(); ++AI)
      AliasMap[I].set(*AI);
  }

  // Propagate smaller alias info upwards. Skip reg 0 (mapped to NoRegister)
  for (MCPhysReg I = 1, E = RegInfo->getNumRegs(); I < E; ++I)
    for (MCSubRegIterator SI(I, RegInfo); SI.isValid(); ++SI)
      SmallerAliasMap[I] |= SmallerAliasMap[*SI];
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 588-602

```cpp
  LLVM_DEBUG({
    dbgs() << "Dumping reg alias table:\n";
    for (MCPhysReg I = 0, E = RegInfo->getNumRegs(); I != E; ++I) {
      dbgs() << "Reg " << I << ": ";
      const BitVector &BV = AliasMap[I];
      int Idx = BV.find_first();
      while (Idx != -1) {
        dbgs() << Idx << " ";
        Idx = BV.find_next(Idx);
      }
      dbgs() << "\n";
    }
  });
}
```

- EN: Declares or implements routines including `dbgs`. Notable symbols here include `dbgs`.
- CN: 这里声明或实现函数，例如 `dbgs`。这里较值得关注的符号包括 `dbgs`。

### Lines 603-610

```cpp
void MCPlusBuilder::initSizeMap() {
  SizeMap.resize(RegInfo->getNumRegs());
  // Build size map
  for (auto RC : RegInfo->regclasses())
    for (MCPhysReg Reg : RC)
      SizeMap[Reg] = RC.getSizeInBits() / 8;
}
```

- EN: Declares or implements routines including `initSizeMap`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `initSizeMap`.
- CN: 这里声明或实现函数，例如 `initSizeMap`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `initSizeMap`。

### Lines 611-628

```cpp
bool MCPlusBuilder::setOperandToSymbolRef(MCInst &Inst, int OpNum,
                                          const MCSymbol *Symbol,
                                          int64_t Addend, MCContext *Ctx,
                                          uint32_t RelType) const {
  MCOperand Operand;
  if (!Addend) {
    Operand = MCOperand::createExpr(getTargetExprFor(
        Inst, MCSymbolRefExpr::create(Symbol, *Ctx), *Ctx, RelType));
  } else {
    Operand = MCOperand::createExpr(getTargetExprFor(
        Inst,
        MCBinaryExpr::createAdd(MCSymbolRefExpr::create(Symbol, *Ctx),
                                MCConstantExpr::create(Addend, *Ctx), *Ctx),
        *Ctx, RelType));
  }
  Inst.getOperand(OpNum) = Operand;
  return true;
}
```

- EN: Declares or implements routines including `create`, `createAdd`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `create`, `createAdd`.
- CN: 这里声明或实现函数，例如 `create`, `createAdd`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `create`, `createAdd`。

## Key Concepts / 关键概念

- `desc`: function or method entry point / 函数或方法入口
- `init`: function or method entry point / 函数或方法入口
- `getNumPrimeOperands`: function or method entry point / 函数或方法入口
- `if`: function or method entry point / 函数或方法入口
- `llvm_unreachable`: function or method entry point / 函数或方法入口
- `DEBUG_TYPE`: macro or compile-time switch / 宏或编译期开关
- `llvm`: namespace scope / 命名空间作用域
- `bolt`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Core/MCPlusBuilder.h`, `bolt/Core/MCPlus.h`, `bolt/Utils/CommandLineOpts.h`
- LLVM headers / LLVM 头文件: `llvm/MC/MCContext.h`, `llvm/MC/MCInst.h`, `llvm/MC/MCInstrAnalysis.h`, `llvm/MC/MCInstrDesc.h`, `llvm/MC/MCInstrInfo.h`, `llvm/MC/MCRegisterInfo.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`
- System headers / 系统头文件: `cstdint`
- Directory context / 目录上下文: `bolt/lib/Core` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Core` 下的相邻文件通常与本文件协作组成对应子系统
