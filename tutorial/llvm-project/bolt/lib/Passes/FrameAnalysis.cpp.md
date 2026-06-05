# FrameAnalysis.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Passes/FrameAnalysis.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: bolt/Passes/FrameAnalysis.cpp This file implements the FrameAnalysis class.. It also sits in code that implements BOLT optimization and transformation passes. / 该文件实现 BOLT 优化与变换 Pass。 源码头部说明其职责是：bolt/Passes/FrameAnalysis.cpp This file implements the FrameAnalysis class.。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

```cpp
//===- bolt/Passes/FrameAnalysis.cpp --------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the FrameAnalysis class.
//
//===----------------------------------------------------------------------===//
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 13-20

```cpp
#include "bolt/Passes/FrameAnalysis.h"
#include "bolt/Core/CallGraphWalker.h"
#include "bolt/Core/ParallelUtilities.h"
#include "llvm/MC/MCRegisterInfo.h"
#include "llvm/Support/Timer.h"
#include <fstream>
#include <stack>
```

- EN: Pulls in 7 header(s) from local project, LLVM, system dependencies needed by this range. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里引入 7 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里的注释记录了设计假设、不变量或使用说明。

### Lines 21-28

```cpp
#define DEBUG_TYPE "fa"

using namespace llvm;

namespace opts {
extern cl::OptionCategory BoltOptCategory;
extern cl::opt<unsigned> Verbosity;
```

- EN: Establishes header-guard or prologue macros so the file can be compiled safely. Works inside namespace scope `llvm`, `opts` to organize symbols. Defines macros such as `DEBUG_TYPE` for constants or compile-time switches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里位于命名空间 `llvm`, `opts` 中，用于组织符号作用域。这里定义宏 `DEBUG_TYPE`，用于常量或编译期开关。这里的注释记录了设计假设、不变量或使用说明。

### Lines 29-37

```cpp
static cl::list<std::string>
    FrameOptFunctionNames("funcs-fop", cl::CommaSeparated,
                          cl::desc("list of functions to apply frame opts"),
                          cl::value_desc("func1,func2,func3,..."));

static cl::opt<std::string> FrameOptFunctionNamesFile(
    "funcs-file-fop",
    cl::desc("file with list of functions to frame optimize"));
```

- EN: Declares or implements routines including `desc`, `value_desc`. Notable symbols here include `desc`, `value_desc`.
- CN: 这里声明或实现函数，例如 `desc`, `value_desc`。这里较值得关注的符号包括 `desc`, `value_desc`。

### Lines 38-45

```cpp
static cl::opt<bool> TimeFA("time-fa", cl::desc("time frame analysis steps"),
                            cl::ReallyHidden, cl::cat(BoltOptCategory));

static cl::opt<bool>
    ExperimentalSW("experimental-shrink-wrapping",
                   cl::desc("process functions with stack pointer arithmetic"),
                   cl::ReallyHidden, cl::ZeroOrMore, cl::cat(BoltOptCategory));
```

- EN: Declares or implements routines including `TimeFA`, `cat`, `desc`. Notable symbols here include `TimeFA`, `cat`, `desc`.
- CN: 这里声明或实现函数，例如 `TimeFA`, `cat`, `desc`。这里较值得关注的符号包括 `TimeFA`, `cat`, `desc`。

### Lines 46-58

```cpp
bool shouldFrameOptimize(const llvm::bolt::BinaryFunction &Function) {
  if (Function.hasUnknownControlFlow())
    return false;

  if (!FrameOptFunctionNamesFile.empty()) {
    assert(!FrameOptFunctionNamesFile.empty() && "unexpected empty file name");
    std::ifstream FuncsFile(FrameOptFunctionNamesFile, std::ios::in);
    std::string FuncName;
    while (std::getline(FuncsFile, FuncName))
      FrameOptFunctionNames.push_back(FuncName);
    FrameOptFunctionNamesFile = "";
  }
```

- EN: Declares or implements routines including `shouldFrameOptimize`, `assert`, `FuncsFile`. Notable symbols here include `shouldFrameOptimize`, `assert`, `FuncsFile`.
- CN: 这里声明或实现函数，例如 `shouldFrameOptimize`, `assert`, `FuncsFile`。这里较值得关注的符号包括 `shouldFrameOptimize`, `assert`, `FuncsFile`。

### Lines 59-66

```cpp
  if (FrameOptFunctionNames.empty())
    return true;
  return llvm::any_of(FrameOptFunctionNames, [&](std::string &Name) {
    return Function.hasName(Name);
  });
}
} // namespace opts
```

- EN: Works inside namespace scope `opts` to organize symbols. Contains local control flow that updates state or selects among execution branches. Notable symbols here include `opts`.
- CN: 这里位于命名空间 `opts` 中，用于组织符号作用域。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里较值得关注的符号包括 `opts`。

### Lines 67-82

```cpp
namespace llvm {
namespace bolt {

raw_ostream &operator<<(raw_ostream &OS, const FrameIndexEntry &FIE) {
  OS << "FrameIndexEntry<IsLoad: " << FIE.IsLoad << ", IsStore: " << FIE.IsStore
     << ", IsStoreFromReg: " << FIE.IsStoreFromReg
     << ", RegOrImm: " << FIE.RegOrImm << ", StackOffset: ";
  if (FIE.StackOffset < 0)
    OS << "-" << Twine::utohexstr(-FIE.StackOffset);
  else
    OS << "+" << Twine::utohexstr(FIE.StackOffset);
  OS << ", Size: " << static_cast<int>(FIE.Size)
     << ", IsSimple: " << FIE.IsSimple << ">";
  return OS;
}
```

- EN: Works inside namespace scope `llvm`, `bolt` to organize symbols. Declares or implements routines including `utohexstr`. Notable symbols here include `utohexstr`, `llvm`, `bolt`.
- CN: 这里位于命名空间 `llvm`, `bolt` 中，用于组织符号作用域。这里声明或实现函数，例如 `utohexstr`。这里较值得关注的符号包括 `utohexstr`, `llvm`, `bolt`。

### Lines 83-95

```cpp
namespace {

/// This class should be used to iterate through basic blocks in layout order
/// to analyze instructions for frame accesses. The user should call
/// enterNewBB() whenever starting analyzing a new BB and doNext() for each
/// instruction. After doNext(), if isValidAccess() returns true, it means the
/// current instruction accesses the frame and getFIE() may be used to obtain
/// details about this access.
class FrameAccessAnalysis {
  /// We depend on Stack Pointer Tracking to figure out the current SP offset
  /// value at a given program point
  StackPointerTracking &SPT;
```

- EN: Introduces type definitions such as `should`, `FrameAccessAnalysis`. Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `should`, `FrameAccessAnalysis`.
- CN: 这里引入类型定义，例如 `should`, `FrameAccessAnalysis`。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `should`, `FrameAccessAnalysis`。

### Lines 96-112

```cpp
  /// Context vars
  const BinaryContext &BC;
  const BinaryFunction &BF;
  // Vars used for storing useful CFI info to give us a hint about how the stack
  // is used in this function
  int SPOffset{0};
  int FPOffset{0};
  int64_t CfaOffset{-8};
  uint16_t CfaReg{7};
  std::stack<std::pair<int64_t, uint16_t>> CFIStack;
  /// Our pointer to access SPT info
  const MCInst *Prev{nullptr};
  /// Info about the last frame access
  bool IsValidAccess{false};
  bool EscapesStackAddress{false};
  FrameIndexEntry FIE;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 113-123

```cpp
  bool decodeFrameAccess(const MCInst &Inst) {
    int32_t SrcImm = 0;
    MCPhysReg Reg = 0;
    int64_t StackOffset = 0;
    bool IsIndexed = false;
    if (!BC.MIB->isStackAccess(
            Inst, FIE.IsLoad, FIE.IsStore, FIE.IsStoreFromReg, Reg, SrcImm,
            FIE.StackPtrReg, StackOffset, FIE.Size, FIE.IsSimple, IsIndexed)) {
      return true;
    }
```

- EN: Declares or implements routines including `decodeFrameAccess`. Notable symbols here include `decodeFrameAccess`.
- CN: 这里声明或实现函数，例如 `decodeFrameAccess`。这里较值得关注的符号包括 `decodeFrameAccess`。

### Lines 124-131

```cpp
    if (IsIndexed || (!FIE.Size && (FIE.IsLoad || FIE.IsStore))) {
      LLVM_DEBUG(dbgs() << "Giving up on indexed memory access/unknown size\n");
      LLVM_DEBUG(dbgs() << "Blame insn: ");
      LLVM_DEBUG(BC.printInstruction(dbgs(), Inst, 0, &BF, true, false, false));
      LLVM_DEBUG(Inst.dump());
      return false;
    }
```

- EN: Declares or implements routines including `LLVM_DEBUG`. Notable symbols here include `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`。这里较值得关注的符号包括 `LLVM_DEBUG`。

### Lines 132-149

```cpp
    assert(FIE.Size != 0 || (!FIE.IsLoad && !FIE.IsStore));

    FIE.RegOrImm = SrcImm;
    if (FIE.IsLoad || FIE.IsStoreFromReg)
      FIE.RegOrImm = Reg;

    if (FIE.StackPtrReg == BC.MIB->getStackPointer() && SPOffset != SPT.EMPTY &&
        SPOffset != SPT.SUPERPOSITION) {
      LLVM_DEBUG(
          dbgs() << "Adding access via SP while CFA reg is another one\n");
      FIE.StackOffset = SPOffset + StackOffset;
    } else if (FIE.StackPtrReg == BC.MIB->getFramePointer() &&
               FPOffset != SPT.EMPTY && FPOffset != SPT.SUPERPOSITION) {
      LLVM_DEBUG(
          dbgs() << "Adding access via FP while CFA reg is another one\n");
      FIE.StackOffset = FPOffset + StackOffset;
    } else if (FIE.StackPtrReg ==
               *BC.MRI->getLLVMRegNum(CfaReg, /*isEH=*/false)) {
```

- EN: Declares or implements routines including `assert`, `dbgs`, `if`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `assert`, `dbgs`, `if`.
- CN: 这里声明或实现函数，例如 `assert`, `dbgs`, `if`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `assert`, `dbgs`, `if`。

### Lines 150-163

```cpp
      FIE.StackOffset = CfaOffset + StackOffset;
    } else {
      LLVM_DEBUG(
          dbgs() << "Found stack access with reg different than cfa reg.\n");
      LLVM_DEBUG(dbgs() << "\tCurrent CFA reg: " << CfaReg
                        << "\n\tStack access reg: " << FIE.StackPtrReg << "\n");
      LLVM_DEBUG(dbgs() << "Blame insn: ");
      LLVM_DEBUG(Inst.dump());
      return false;
    }
    IsValidAccess = true;
    return true;
  }
```

- EN: Declares or implements routines including `dbgs`, `LLVM_DEBUG`. Notable symbols here include `dbgs`, `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `dbgs`, `LLVM_DEBUG`。这里较值得关注的符号包括 `dbgs`, `LLVM_DEBUG`。

### Lines 164-173

```cpp
public:
  FrameAccessAnalysis(BinaryFunction &BF, StackPointerTracking &SPT)
      : SPT(SPT), BC(BF.getBinaryContext()), BF(BF) {}

  void enterNewBB() { Prev = nullptr; }
  const FrameIndexEntry &getFIE() const { return FIE; }
  int getSPOffset() const { return SPOffset; }
  bool isValidAccess() const { return IsValidAccess; }
  bool doesEscapeStackAddress() const { return EscapesStackAddress; }
```

- EN: Declares or implements routines including `FrameAccessAnalysis`, `SPT`, `enterNewBB`, `getFIE`, `getSPOffset`, and 2 more. Notable symbols here include `FrameAccessAnalysis`, `SPT`, `enterNewBB`, `getFIE`, `getSPOffset`, `isValidAccess`.
- CN: 这里声明或实现函数，例如 `FrameAccessAnalysis`, `SPT`, `enterNewBB`, `getFIE`, `getSPOffset`, and 2 more。这里较值得关注的符号包括 `FrameAccessAnalysis`, `SPT`, `enterNewBB`, `getFIE`, `getSPOffset`, `isValidAccess`。

### Lines 174-191

```cpp
  bool doNext(const BinaryBasicBlock &BB, const MCInst &Inst) {
    IsValidAccess = false;
    EscapesStackAddress = false;
    std::tie(SPOffset, FPOffset) =
        Prev ? *SPT.getStateAt(*Prev) : *SPT.getStateAt(BB);
    Prev = &Inst;
    // Use CFI information to keep track of which register is being used to
    // access the frame
    if (BC.MIB->isCFI(Inst)) {
      const MCCFIInstruction *CFI = BF.getCFIFor(Inst);
      switch (CFI->getOperation()) {
      case MCCFIInstruction::OpDefCfa:
        CfaOffset = CFI->getOffset();
        [[fallthrough]];
      case MCCFIInstruction::OpDefCfaRegister:
        CfaReg = CFI->getRegister();
        break;
      case MCCFIInstruction::OpDefCfaOffset:
```

- EN: Declares or implements routines including `doNext`, `tie`, `getOffset`, `getRegister`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `doNext`, `tie`, `getOffset`, `getRegister`.
- CN: 这里声明或实现函数，例如 `doNext`, `tie`, `getOffset`, `getRegister`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `doNext`, `tie`, `getOffset`, `getRegister`。

### Lines 192-209

```cpp
        CfaOffset = CFI->getOffset();
        break;
      case MCCFIInstruction::OpRememberState:
        CFIStack.push(std::make_pair(CfaOffset, CfaReg));
        break;
      case MCCFIInstruction::OpRestoreState: {
        if (CFIStack.empty())
          dbgs() << "Assertion is about to fail: " << BF.getPrintName() << "\n";
        assert(!CFIStack.empty() && "Corrupt CFI stack");
        std::pair<int64_t, uint16_t> Elem = CFIStack.top();
        CFIStack.pop();
        CfaOffset = Elem.first;
        CfaReg = Elem.second;
        break;
      }
      case MCCFIInstruction::OpAdjustCfaOffset:
        llvm_unreachable("Unhandled AdjustCfaOffset");
        break;
```

- EN: Declares or implements routines including `getOffset`, `dbgs`, `assert`, `llvm_unreachable`. Notable symbols here include `getOffset`, `dbgs`, `assert`, `llvm_unreachable`.
- CN: 这里声明或实现函数，例如 `getOffset`, `dbgs`, `assert`, `llvm_unreachable`。这里较值得关注的符号包括 `getOffset`, `dbgs`, `assert`, `llvm_unreachable`。

### Lines 210-226

```cpp
      default:
        break;
      }
      return true;
    }

    if (BC.MIB->escapesVariable(Inst, SPT.HasFramePointer)) {
      EscapesStackAddress = true;
      if (!opts::ExperimentalSW) {
        LLVM_DEBUG(
            dbgs() << "Leaked stack address, giving up on this function.\n");
        LLVM_DEBUG(dbgs() << "Blame insn: ");
        LLVM_DEBUG(Inst.dump());
        return false;
      }
    }
```

- EN: Declares or implements routines including `dbgs`, `LLVM_DEBUG`. Notable symbols here include `dbgs`, `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `dbgs`, `LLVM_DEBUG`。这里较值得关注的符号包括 `dbgs`, `LLVM_DEBUG`。

### Lines 227-244

```cpp
    return decodeFrameAccess(Inst);
  }
};

} // end anonymous namespace

void FrameAnalysis::addArgAccessesFor(MCInst &Inst, ArgAccesses &&AA) {
  if (ErrorOr<ArgAccesses &> OldAA = getArgAccessesFor(Inst)) {
    if (OldAA->AssumeEverything)
      return;
    *OldAA = std::move(AA);
    return;
  }
  if (AA.AssumeEverything) {
    // Index 0 in ArgAccessesVector represents an "assumeeverything" entry
    BC.MIB->addAnnotation(Inst, "ArgAccessEntry", 0U);
    return;
  }
```

- EN: Works inside namespace scope `void` to organize symbols. Declares or implements routines including `addArgAccessesFor`, `addAnnotation`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `addArgAccessesFor`, `addAnnotation`, `void`.
- CN: 这里位于命名空间 `void` 中，用于组织符号作用域。这里声明或实现函数，例如 `addArgAccessesFor`, `addAnnotation`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `addArgAccessesFor`, `addAnnotation`, `void`。

### Lines 245-262

```cpp
  BC.MIB->addAnnotation(Inst, "ArgAccessEntry",
                        (unsigned)ArgAccessesVector.size());
  ArgAccessesVector.emplace_back(std::move(AA));
}

void FrameAnalysis::addArgInStackAccessFor(MCInst &Inst,
                                           const ArgInStackAccess &Arg) {
  ErrorOr<ArgAccesses &> AA = getArgAccessesFor(Inst);
  if (!AA) {
    addArgAccessesFor(Inst, ArgAccesses(false));
    AA = getArgAccessesFor(Inst);
    assert(AA && "Object setup failed");
  }
  std::set<ArgInStackAccess> &Set = AA->Set;
  assert(!AA->AssumeEverything && "Adding arg to AssumeEverything set");
  Set.emplace(Arg);
}
```

- EN: Declares or implements routines including `getArgAccessesFor`, `addArgAccessesFor`, `assert`. Notable symbols here include `getArgAccessesFor`, `addArgAccessesFor`, `assert`.
- CN: 这里声明或实现函数，例如 `getArgAccessesFor`, `addArgAccessesFor`, `assert`。这里较值得关注的符号包括 `getArgAccessesFor`, `addArgAccessesFor`, `assert`。

### Lines 263-275

```cpp
void FrameAnalysis::addFIEFor(MCInst &Inst, const FrameIndexEntry &FIE) {
  BC.MIB->addAnnotation(Inst, "FrameAccessEntry", (unsigned)FIEVector.size());
  FIEVector.emplace_back(FIE);
}

ErrorOr<ArgAccesses &> FrameAnalysis::getArgAccessesFor(const MCInst &Inst) {
  if (auto Idx = BC.MIB->tryGetAnnotationAs<unsigned>(Inst, "ArgAccessEntry")) {
    assert(ArgAccessesVector.size() > *Idx && "Out of bounds");
    return ArgAccessesVector[*Idx];
  }
  return make_error_code(errc::result_out_of_range);
}
```

- EN: Declares or implements routines including `addFIEFor`, `addAnnotation`, `getArgAccessesFor`, `assert`. Notable symbols here include `addFIEFor`, `addAnnotation`, `getArgAccessesFor`, `assert`.
- CN: 这里声明或实现函数，例如 `addFIEFor`, `addAnnotation`, `getArgAccessesFor`, `assert`。这里较值得关注的符号包括 `addFIEFor`, `addAnnotation`, `getArgAccessesFor`, `assert`。

### Lines 276-284

```cpp
ErrorOr<const ArgAccesses &>
FrameAnalysis::getArgAccessesFor(const MCInst &Inst) const {
  if (auto Idx = BC.MIB->tryGetAnnotationAs<unsigned>(Inst, "ArgAccessEntry")) {
    assert(ArgAccessesVector.size() > *Idx && "Out of bounds");
    return ArgAccessesVector[*Idx];
  }
  return make_error_code(errc::result_out_of_range);
}
```

- EN: Declares or implements routines including `getArgAccessesFor`, `assert`. Notable symbols here include `getArgAccessesFor`, `assert`.
- CN: 这里声明或实现函数，例如 `getArgAccessesFor`, `assert`。这里较值得关注的符号包括 `getArgAccessesFor`, `assert`。

### Lines 285-294

```cpp
ErrorOr<const FrameIndexEntry &>
FrameAnalysis::getFIEFor(const MCInst &Inst) const {
  if (auto Idx =
          BC.MIB->tryGetAnnotationAs<unsigned>(Inst, "FrameAccessEntry")) {
    assert(FIEVector.size() > *Idx && "Out of bounds");
    return FIEVector[*Idx];
  }
  return make_error_code(errc::result_out_of_range);
}
```

- EN: Declares or implements routines including `getFIEFor`, `assert`. Notable symbols here include `getFIEFor`, `assert`.
- CN: 这里声明或实现函数，例如 `getFIEFor`, `assert`。这里较值得关注的符号包括 `getFIEFor`, `assert`。

### Lines 295-302

```cpp
void FrameAnalysis::traverseCG(BinaryFunctionCallGraph &CG) {
  CallGraphWalker CGWalker(CG);

  CGWalker.registerVisitor(
      [&](BinaryFunction *Func) -> bool { return computeArgsAccessed(*Func); });

  CGWalker.walk();
```

- EN: Declares or implements routines including `traverseCG`, `CGWalker`. Notable symbols here include `traverseCG`, `CGWalker`.
- CN: 这里声明或实现函数，例如 `traverseCG`, `CGWalker`。这里较值得关注的符号包括 `traverseCG`, `CGWalker`。

### Lines 303-317

```cpp
  DEBUG_WITH_TYPE("ra", {
    for (auto &MapEntry : ArgsTouchedMap) {
      const BinaryFunction *Func = MapEntry.first;
      const auto &Set = MapEntry.second;
      dbgs() << "Args accessed for " << Func->getPrintName() << ": ";
      if (!Set.empty() && Set.count(std::make_pair(-1, 0)))
        dbgs() << "assume everything";
      else
        for (const std::pair<int64_t, uint8_t> &Entry : Set)
          dbgs() << "[" << Entry.first << ", " << (int)Entry.second << "] ";
      dbgs() << "\n";
    }
  });
}
```

- EN: Declares or implements routines including `dbgs`. Notable symbols here include `dbgs`.
- CN: 这里声明或实现函数，例如 `dbgs`。这里较值得关注的符号包括 `dbgs`。

### Lines 318-333

```cpp
bool FrameAnalysis::updateArgsTouchedFor(const BinaryFunction &BF, MCInst &Inst,
                                         int CurOffset) {
  if (!BC.MIB->isCall(Inst))
    return false;

  const MCSymbol *TargetSymbol = BC.MIB->getTargetSymbol(Inst);
  // If indirect call, we conservatively assume it accesses all stack positions
  if (TargetSymbol == nullptr) {
    addArgAccessesFor(Inst, ArgAccesses(/*AssumeEverything=*/true));
    if (!FunctionsRequireAlignment.count(&BF)) {
      FunctionsRequireAlignment.insert(&BF);
      return true;
    }
    return false;
  }
```

- EN: Declares or implements routines including `getTargetSymbol`, `addArgAccessesFor`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getTargetSymbol`, `addArgAccessesFor`.
- CN: 这里声明或实现函数，例如 `getTargetSymbol`, `addArgAccessesFor`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getTargetSymbol`, `addArgAccessesFor`。

### Lines 334-345

```cpp
  const BinaryFunction *Function = BC.getFunctionForSymbol(TargetSymbol);
  // Call to a function without a BinaryFunction object. Conservatively assume
  // it accesses all stack positions
  if (Function == nullptr) {
    addArgAccessesFor(Inst, ArgAccesses(/*AssumeEverything=*/true));
    if (!FunctionsRequireAlignment.count(&BF)) {
      FunctionsRequireAlignment.insert(&BF);
      return true;
    }
    return false;
  }
```

- EN: Declares or implements routines including `addArgAccessesFor`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `addArgAccessesFor`.
- CN: 这里声明或实现函数，例如 `addArgAccessesFor`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `addArgAccessesFor`。

### Lines 346-363

```cpp
  auto Iter = ArgsTouchedMap.find(Function);

  bool Changed = false;
  if (BC.MIB->isTailCall(Inst) && Iter != ArgsTouchedMap.end()) {
    // Ignore checking CurOffset because we can't always reliably determine the
    // offset specially after an epilogue, where tailcalls happen. It should be
    // -8.
    for (std::pair<int64_t, uint8_t> Elem : Iter->second) {
      if (!llvm::is_contained(ArgsTouchedMap[&BF], Elem)) {
        ArgsTouchedMap[&BF].emplace(Elem);
        Changed = true;
      }
    }
  }
  if (FunctionsRequireAlignment.count(Function) &&
      !FunctionsRequireAlignment.count(&BF)) {
    Changed = true;
    FunctionsRequireAlignment.insert(&BF);
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 364-373

```cpp
  }
  if (Iter == ArgsTouchedMap.end())
    return Changed;

  if (CurOffset == StackPointerTracking::EMPTY ||
      CurOffset == StackPointerTracking::SUPERPOSITION) {
    addArgAccessesFor(Inst, ArgAccesses(/*AssumeEverything=*/true));
    return Changed;
  }
```

- EN: Declares or implements routines including `addArgAccessesFor`. Notable symbols here include `addArgAccessesFor`.
- CN: 这里声明或实现函数，例如 `addArgAccessesFor`。这里较值得关注的符号包括 `addArgAccessesFor`。

### Lines 374-387

```cpp
  for (std::pair<int64_t, uint8_t> Elem : Iter->second) {
    if (Elem.first == -1) {
      addArgAccessesFor(Inst, ArgAccesses(/*AssumeEverything=*/true));
      break;
    }
    LLVM_DEBUG(dbgs() << "Added arg in stack access annotation "
                      << CurOffset + Elem.first << "\n");
    addArgInStackAccessFor(
        Inst, ArgInStackAccess{/*StackOffset=*/CurOffset + Elem.first,
                               /*Size=*/Elem.second});
  }
  return Changed;
}
```

- EN: Declares or implements routines including `addArgAccessesFor`, `LLVM_DEBUG`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `addArgAccessesFor`, `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `addArgAccessesFor`, `LLVM_DEBUG`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `addArgAccessesFor`, `LLVM_DEBUG`。

### Lines 388-399

```cpp
bool FrameAnalysis::computeArgsAccessed(BinaryFunction &BF) {
  if (!BF.isSimple() || !BF.hasCFG()) {
    LLVM_DEBUG(dbgs() << "Treating " << BF.getPrintName()
                      << " conservatively.\n");
    ArgsTouchedMap[&BF].emplace(std::make_pair(-1, 0));
    if (!FunctionsRequireAlignment.count(&BF)) {
      FunctionsRequireAlignment.insert(&BF);
      return true;
    }
    return false;
  }
```

- EN: Declares or implements routines including `computeArgsAccessed`, `LLVM_DEBUG`. Notable symbols here include `computeArgsAccessed`, `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `computeArgsAccessed`, `LLVM_DEBUG`。这里较值得关注的符号包括 `computeArgsAccessed`, `LLVM_DEBUG`。

### Lines 400-408

```cpp
  LLVM_DEBUG(dbgs() << "Now computing args accessed for: " << BF.getPrintName()
                    << "\n");
  bool UpdatedArgsTouched = false;
  bool NoInfo = false;
  FrameAccessAnalysis FAA(BF, getSPT(BF));

  for (BinaryBasicBlock *BB : BF.getLayout().blocks()) {
    FAA.enterNewBB();
```

- EN: Declares or implements routines including `LLVM_DEBUG`, `FAA`. Notable symbols here include `LLVM_DEBUG`, `FAA`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`, `FAA`。这里较值得关注的符号包括 `LLVM_DEBUG`, `FAA`。

### Lines 409-420

```cpp
    for (MCInst &Inst : *BB) {
      if (!FAA.doNext(*BB, Inst) || FAA.doesEscapeStackAddress()) {
        ArgsTouchedMap[&BF].emplace(std::make_pair(-1, 0));
        NoInfo = true;
        break;
      }

      // Check for calls -- attach stack accessing info to them regarding their
      // target
      if (updateArgsTouchedFor(BF, Inst, FAA.getSPOffset()))
        UpdatedArgsTouched = true;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 421-431

```cpp
      // Check for stack accesses that affect callers
      if (!FAA.isValidAccess())
        continue;

      const FrameIndexEntry &FIE = FAA.getFIE();
      if (FIE.StackOffset < 0)
        continue;
      if (ArgsTouchedMap[&BF].find(std::make_pair(FIE.StackOffset, FIE.Size)) !=
          ArgsTouchedMap[&BF].end())
        continue;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 432-445

```cpp
      // Record accesses to the previous stack frame
      ArgsTouchedMap[&BF].emplace(std::make_pair(FIE.StackOffset, FIE.Size));
      UpdatedArgsTouched = true;
      LLVM_DEBUG({
        dbgs() << "Arg access offset " << FIE.StackOffset << " added to:\n";
        BC.printInstruction(dbgs(), Inst, 0, &BF, true);
      });
    }
    if (NoInfo)
      break;
  }
  if (FunctionsRequireAlignment.count(&BF))
    return UpdatedArgsTouched;
```

- EN: Declares or implements routines including `dbgs`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `dbgs`.
- CN: 这里声明或实现函数，例如 `dbgs`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `dbgs`。

### Lines 446-461

```cpp
  if (NoInfo) {
    FunctionsRequireAlignment.insert(&BF);
    return true;
  }

  for (BinaryBasicBlock &BB : BF) {
    for (MCInst &Inst : BB) {
      if (BC.MIB->requiresAlignedAddress(Inst)) {
        FunctionsRequireAlignment.insert(&BF);
        return true;
      }
    }
  }
  return UpdatedArgsTouched;
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 462-470

```cpp
bool FrameAnalysis::restoreFrameIndex(BinaryFunction &BF) {
  FrameAccessAnalysis FAA(BF, getSPT(BF));

  LLVM_DEBUG(dbgs() << "Restoring frame indices for \"" << BF.getPrintName()
                    << "\"\n");
  for (BinaryBasicBlock *BB : BF.getLayout().blocks()) {
    LLVM_DEBUG(dbgs() << "\tNow at BB " << BB->getName() << "\n");
    FAA.enterNewBB();
```

- EN: Declares or implements routines including `restoreFrameIndex`, `FAA`, `LLVM_DEBUG`. Notable symbols here include `restoreFrameIndex`, `FAA`, `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `restoreFrameIndex`, `FAA`, `LLVM_DEBUG`。这里较值得关注的符号包括 `restoreFrameIndex`, `FAA`, `LLVM_DEBUG`。

### Lines 471-479

```cpp
    for (MCInst &Inst : *BB) {
      if (!FAA.doNext(*BB, Inst))
        return false;
      LLVM_DEBUG({
        dbgs() << "\t\tNow at ";
        Inst.dump();
        dbgs() << "\t\t\tSP offset is " << FAA.getSPOffset() << "\n";
      });
```

- EN: Declares or implements routines including `dbgs`. Notable symbols here include `dbgs`.
- CN: 这里声明或实现函数，例如 `dbgs`。这里较值得关注的符号包括 `dbgs`。

### Lines 480-488

```cpp
      if (FAA.doesEscapeStackAddress()) {
        if (!FunctionsWithStackArithmetic.count(&BF))
          FunctionsWithStackArithmetic.insert(&BF);
        continue;
      }

      if (!FAA.isValidAccess())
        continue;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 489-500

```cpp
      const FrameIndexEntry &FIE = FAA.getFIE();

      addFIEFor(Inst, FIE);
      LLVM_DEBUG({
        dbgs() << "Frame index annotation " << FIE << " added to:\n";
        BC.printInstruction(dbgs(), Inst, 0, &BF, true);
      });
    }
  }
  return true;
}
```

- EN: Declares or implements routines including `addFIEFor`, `dbgs`. Notable symbols here include `addFIEFor`, `dbgs`.
- CN: 这里声明或实现函数，例如 `addFIEFor`, `dbgs`。这里较值得关注的符号包括 `addFIEFor`, `dbgs`。

### Lines 501-513

```cpp
void FrameAnalysis::cleanAnnotations() {
  NamedRegionTimer T("cleanannotations", "clean annotations", "FA",
                     "FA breakdown", opts::TimeFA);

  ParallelUtilities::WorkFuncTy CleanFunction = [&](BinaryFunction &BF) {
    for (BinaryBasicBlock &BB : BF) {
      for (MCInst &Inst : BB) {
        BC.MIB->removeAnnotation(Inst, "ArgAccessEntry");
        BC.MIB->removeAnnotation(Inst, "FrameAccessEntry");
      }
    }
  };
```

- EN: Declares or implements routines including `cleanAnnotations`, `removeAnnotation`. Notable symbols here include `cleanAnnotations`, `removeAnnotation`.
- CN: 这里声明或实现函数，例如 `cleanAnnotations`, `removeAnnotation`。这里较值得关注的符号包括 `cleanAnnotations`, `removeAnnotation`。

### Lines 514-524

```cpp
  ParallelUtilities::runOnEachFunction(
      BC, ParallelUtilities::SchedulingPolicy::SP_INST_LINEAR, CleanFunction,
      ParallelUtilities::PredicateTy(nullptr), "cleanAnnotations");
}

FrameAnalysis::FrameAnalysis(BinaryContext &BC, BinaryFunctionCallGraph &CG)
    : BC(BC) {
  // Position 0 of the vector should be always associated with "assume access
  // everything".
  ArgAccessesVector.emplace_back(ArgAccesses(/*AssumeEverything*/ true));
```

- EN: Declares or implements routines including `PredicateTy`, `FrameAnalysis`, `BC`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `PredicateTy`, `FrameAnalysis`, `BC`.
- CN: 这里声明或实现函数，例如 `PredicateTy`, `FrameAnalysis`, `BC`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `PredicateTy`, `FrameAnalysis`, `BC`。

### Lines 525-536

```cpp
  if (!opts::NoThreads) {
    NamedRegionTimer T1("precomputespt", "pre-compute spt", "FA",
                        "FA breakdown", opts::TimeFA);
    preComputeSPT();
  }

  {
    NamedRegionTimer T1("traversecg", "traverse call graph", "FA",
                        "FA breakdown", opts::TimeFA);
    traverseCG(CG);
  }
```

- EN: Declares or implements routines including `preComputeSPT`, `traverseCG`. Notable symbols here include `preComputeSPT`, `traverseCG`.
- CN: 这里声明或实现函数，例如 `preComputeSPT`, `traverseCG`。这里较值得关注的符号包括 `preComputeSPT`, `traverseCG`。

### Lines 537-546

```cpp
  for (auto &I : BC.getBinaryFunctions()) {
    CountDenominator += I.second.getFunctionScore();

    // "shouldOptimize" for passes that run after finalize
    if (!(I.second.isSimple() && I.second.hasCFG() && !I.second.isIgnored()) ||
        !opts::shouldFrameOptimize(I.second)) {
      ++NumFunctionsNotOptimized;
      continue;
    }
```

- EN: Declares or implements routines including `shouldFrameOptimize`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `shouldFrameOptimize`.
- CN: 这里声明或实现函数，例如 `shouldFrameOptimize`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `shouldFrameOptimize`。

### Lines 547-558

```cpp
    {
      NamedRegionTimer T1("restorefi", "restore frame index", "FA",
                          "FA breakdown", opts::TimeFA);
      if (!restoreFrameIndex(I.second)) {
        ++NumFunctionsFailedRestoreFI;
        CountFunctionsFailedRestoreFI += I.second.getFunctionScore();
        continue;
      }
    }
    AnalyzedFunctions.insert(&I.second);
  }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 559-576

```cpp
  {
    NamedRegionTimer T1("clearspt", "clear spt", "FA", "FA breakdown",
                        opts::TimeFA);
    clearSPTMap();
  }
}

void FrameAnalysis::printStats() {
  BC.outs() << "BOLT-INFO: FRAME ANALYSIS: " << NumFunctionsNotOptimized
            << " function(s) were not optimized.\n"
            << "BOLT-INFO: FRAME ANALYSIS: " << NumFunctionsFailedRestoreFI
            << " function(s) "
            << format(
                   "(%.1lf%% dyn cov)",
                   (100.0 * CountFunctionsFailedRestoreFI / CountDenominator))
            << " could not have its frame indices restored.\n";
}
```

- EN: Declares or implements routines including `clearSPTMap`, `printStats`, `function`. Notable symbols here include `clearSPTMap`, `printStats`, `function`.
- CN: 这里声明或实现函数，例如 `clearSPTMap`, `printStats`, `function`。这里较值得关注的符号包括 `clearSPTMap`, `printStats`, `function`。

### Lines 577-587

```cpp
void FrameAnalysis::clearSPTMap() {
  if (opts::NoThreads) {
    SPTMap.clear();
    return;
  }

  ParallelUtilities::WorkFuncTy ClearFunctionSPT = [&](BinaryFunction &BF) {
    std::unique_ptr<StackPointerTracking> &SPTPtr = SPTMap.find(&BF)->second;
    SPTPtr.reset();
  };
```

- EN: Declares or implements routines including `clearSPTMap`. Notable symbols here include `clearSPTMap`.
- CN: 这里声明或实现函数，例如 `clearSPTMap`。这里较值得关注的符号包括 `clearSPTMap`。

### Lines 588-595

```cpp
  ParallelUtilities::PredicateTy SkipFunc = [&](const BinaryFunction &BF) {
    return !BF.isSimple() || !BF.hasCFG();
  };

  ParallelUtilities::runOnEachFunction(
      BC, ParallelUtilities::SchedulingPolicy::SP_INST_LINEAR, ClearFunctionSPT,
      SkipFunc, "clearSPTMap");
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 596-610

```cpp
  SPTMap.clear();
}

void FrameAnalysis::preComputeSPT() {
  // Make sure that the SPTMap is empty
  assert(SPTMap.size() == 0);

  // Create map entries to allow lock-free parallel execution
  for (auto &BFI : BC.getBinaryFunctions()) {
    BinaryFunction &BF = BFI.second;
    if (!BF.isSimple() || !BF.hasCFG())
      continue;
    SPTMap.emplace(&BF, std::unique_ptr<StackPointerTracking>());
  }
```

- EN: Declares or implements routines including `preComputeSPT`, `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `preComputeSPT`, `assert`.
- CN: 这里声明或实现函数，例如 `preComputeSPT`, `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `preComputeSPT`, `assert`。

### Lines 611-623

```cpp
  // Create an index for the SPT annotation to allow lock-free parallel
  // execution
  BC.MIB->getOrCreateAnnotationIndex("StackPointerTracking");

  // Run SPT in parallel
  ParallelUtilities::WorkFuncWithAllocTy ProcessFunction =
      [&](BinaryFunction &BF, MCPlusBuilder::AllocatorIdTy AllocId) {
        std::unique_ptr<StackPointerTracking> &SPTPtr =
            SPTMap.find(&BF)->second;
        SPTPtr = std::make_unique<StackPointerTracking>(BF, AllocId);
        SPTPtr->run();
      };
```

- EN: Declares or implements routines including `getOrCreateAnnotationIndex`, `run`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getOrCreateAnnotationIndex`, `run`.
- CN: 这里声明或实现函数，例如 `getOrCreateAnnotationIndex`, `run`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getOrCreateAnnotationIndex`, `run`。

### Lines 624-632

```cpp
  ParallelUtilities::PredicateTy SkipPredicate = [&](const BinaryFunction &BF) {
    return !BF.isSimple() || !BF.hasCFG();
  };

  ParallelUtilities::runOnEachFunctionWithUniqueAllocId(
      BC, ParallelUtilities::SchedulingPolicy::SP_BB_QUADRATIC, ProcessFunction,
      SkipPredicate, "preComputeSPT");
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 633-634

```cpp
} // namespace bolt
} // namespace llvm
```

- EN: Works inside namespace scope `bolt`, `llvm` to organize symbols. Notable symbols here include `bolt`, `llvm`.
- CN: 这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里较值得关注的符号包括 `bolt`, `llvm`。

## Key Concepts / 关键概念

- `should`: class or struct interface / 类或结构体接口
- `FrameAccessAnalysis`: class or struct interface / 类或结构体接口
- `desc`: function or method entry point / 函数或方法入口
- `value_desc`: function or method entry point / 函数或方法入口
- `TimeFA`: function or method entry point / 函数或方法入口
- `cat`: function or method entry point / 函数或方法入口
- `shouldFrameOptimize`: function or method entry point / 函数或方法入口
- `DEBUG_TYPE`: macro or compile-time switch / 宏或编译期开关

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Passes/FrameAnalysis.h`, `bolt/Core/CallGraphWalker.h`, `bolt/Core/ParallelUtilities.h`
- LLVM headers / LLVM 头文件: `llvm/MC/MCRegisterInfo.h`, `llvm/Support/Timer.h`
- System headers / 系统头文件: `fstream`, `stack`
- Directory context / 目录上下文: `bolt/lib/Passes` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Passes` 下的相邻文件通常与本文件协作组成对应子系统
