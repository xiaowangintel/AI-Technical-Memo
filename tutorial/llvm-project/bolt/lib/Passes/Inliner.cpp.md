# Inliner.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/lib/Passes/Inliner.cpp`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: Inlining pass for low-level binary IR. It also sits in code that implements BOLT optimization and transformation passes. / 该文件实现 BOLT 优化与变换 Pass。 源码头部说明其职责是：Inlining pass for low-level binary IR。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18

```cpp
//===- bolt/Passes/Inliner.cpp - Inlining pass for low-level binary IR ----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the Inliner class used for inlining binary functions.
//
// The current inliner has a limited callee support
// (see Inliner::getInliningInfo() for the most up-to-date details):
//
//  * No exception handling
//  * No jump tables
//  * Single entry point
//  * CFI update not supported - breaks unwinding
//  * Regular Call Sites:
```

- EN: Introduces type definitions such as `used`. Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `used`.
- CN: 这里引入类型定义，例如 `used`。这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `used`。

### Lines 19-26

```cpp
//    - only leaf functions (or callees with only tail calls)
//      * no invokes (they can't be tail calls)
//    - no direct use of %rsp
//  * Tail Call Sites:
//    - since the stack is unmodified, the regular call limitations are lifted
//
//===----------------------------------------------------------------------===//
```

- EN: Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里的注释记录了设计假设、不变量或使用说明。

### Lines 27-34

```cpp
#include "bolt/Passes/Inliner.h"
#include "bolt/Core/MCPlus.h"
#include "llvm/Support/CommandLine.h"

#define DEBUG_TYPE "bolt-inliner"

using namespace llvm;
```

- EN: Pulls in 3 header(s) from local project, LLVM dependencies needed by this range. Establishes header-guard or prologue macros so the file can be compiled safely. Works inside namespace scope `llvm` to organize symbols. Defines macros such as `DEBUG_TYPE` for constants or compile-time switches.
- CN: 这里引入 3 个头文件，为本段提供本地模块、LLVM 或系统层面的依赖。这里建立头文件保护或前导宏，保证源码可被安全编译与重复包含控制。这里位于命名空间 `llvm` 中，用于组织符号作用域。这里定义宏 `DEBUG_TYPE`，用于常量或编译期开关。

### Lines 35-43

```cpp
namespace opts {

extern cl::OptionCategory BoltOptCategory;

static cl::opt<bool>
    AdjustProfile("inline-ap",
                  cl::desc("adjust function profile after inlining"),
                  cl::cat(BoltOptCategory));
```

- EN: Works inside namespace scope `opts` to organize symbols. Declares or implements routines including `desc`, `cat`. Notable symbols here include `desc`, `cat`, `opts`.
- CN: 这里位于命名空间 `opts` 中，用于组织符号作用域。这里声明或实现函数，例如 `desc`, `cat`。这里较值得关注的符号包括 `desc`, `cat`, `opts`。

### Lines 44-51

```cpp
static cl::list<std::string>
ForceInlineFunctions("force-inline",
  cl::CommaSeparated,
  cl::desc("list of functions to always consider for inlining"),
  cl::value_desc("func1,func2,func3,..."),
  cl::Hidden,
  cl::cat(BoltOptCategory));
```

- EN: Declares or implements routines including `desc`, `value_desc`, `cat`. Notable symbols here include `desc`, `value_desc`, `cat`.
- CN: 这里声明或实现函数，例如 `desc`, `value_desc`, `cat`。这里较值得关注的符号包括 `desc`, `value_desc`, `cat`。

### Lines 52-60

```cpp
static cl::list<std::string> SkipInlineFunctions(
    "skip-inline", cl::CommaSeparated,
    cl::desc("list of functions to never consider for inlining"),
    cl::value_desc("func1,func2,func3,..."), cl::Hidden,
    cl::cat(BoltOptCategory));

static cl::opt<bool> InlineAll("inline-all", cl::desc("inline all functions"),
                               cl::cat(BoltOptCategory));
```

- EN: Declares or implements routines including `desc`, `value_desc`, `cat`, `InlineAll`. Notable symbols here include `desc`, `value_desc`, `cat`, `InlineAll`.
- CN: 这里声明或实现函数，例如 `desc`, `value_desc`, `cat`, `InlineAll`。这里较值得关注的符号包括 `desc`, `value_desc`, `cat`, `InlineAll`。

### Lines 61-71

```cpp
static cl::opt<bool> InlineIgnoreLeafCFI(
    "inline-ignore-leaf-cfi",
    cl::desc("inline leaf functions with CFI programs (can break unwinding)"),
    cl::init(true), cl::ReallyHidden, cl::cat(BoltOptCategory));

static cl::opt<bool> InlineIgnoreCFI(
    "inline-ignore-cfi",
    cl::desc(
        "inline functions with CFI programs (can break exception handling)"),
    cl::ReallyHidden, cl::cat(BoltOptCategory));
```

- EN: Declares or implements routines including `desc`, `init`, `programs`, `cat`. Notable symbols here include `desc`, `init`, `programs`, `cat`.
- CN: 这里声明或实现函数，例如 `desc`, `init`, `programs`, `cat`。这里较值得关注的符号包括 `desc`, `init`, `programs`, `cat`。

### Lines 72-81

```cpp
static cl::opt<unsigned>
    InlineLimit("inline-limit",
                cl::desc("maximum number of call sites to inline"), cl::init(0),
                cl::Hidden, cl::cat(BoltOptCategory));

static cl::opt<unsigned>
    InlineMaxIters("inline-max-iters",
                   cl::desc("maximum number of inline iterations"), cl::init(3),
                   cl::Hidden, cl::cat(BoltOptCategory));
```

- EN: Declares or implements routines including `desc`, `cat`. Notable symbols here include `desc`, `cat`.
- CN: 这里声明或实现函数，例如 `desc`, `cat`。这里较值得关注的符号包括 `desc`, `cat`。

### Lines 82-93

```cpp
static cl::opt<bool> InlineSmallFunctions(
    "inline-small-functions",
    cl::desc("inline functions if increase in size is less than defined by "
             "-inline-small-functions-bytes"),
    cl::cat(BoltOptCategory));

static cl::opt<unsigned> InlineSmallFunctionsBytes(
    "inline-small-functions-bytes",
    cl::desc("max number of bytes for the function to be considered small for "
             "inlining purposes"),
    cl::init(4), cl::Hidden, cl::cat(BoltOptCategory));
```

- EN: Declares or implements routines including `cat`, `init`. Notable symbols here include `cat`, `init`.
- CN: 这里声明或实现函数，例如 `cat`, `init`。这里较值得关注的符号包括 `cat`, `init`。

### Lines 94-106

```cpp
static cl::opt<bool> NoInline(
    "no-inline",
    cl::desc("disable all inlining (overrides other inlining options)"),
    cl::cat(BoltOptCategory));

/// This function returns true if any of inlining options are specified and the
/// inlining pass should be executed. Whenever a new inlining option is added,
/// this function should reflect the change.
bool inliningEnabled() {
  return !NoInline &&
         (InlineAll || InlineSmallFunctions || !ForceInlineFunctions.empty());
}
```

- EN: Declares or implements routines including `desc`, `cat`, `inliningEnabled`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `desc`, `cat`, `inliningEnabled`.
- CN: 这里声明或实现函数，例如 `desc`, `cat`, `inliningEnabled`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `desc`, `cat`, `inliningEnabled`。

### Lines 107-119

```cpp
bool mustConsider(const llvm::bolt::BinaryFunction &Function) {
  for (std::string &Name : opts::ForceInlineFunctions)
    if (Function.hasName(Name))
      return true;
  return false;
}

bool mustSkip(const llvm::bolt::BinaryFunction &Function) {
  return llvm::any_of(opts::SkipInlineFunctions, [&](const std::string &Name) {
    return Function.hasName(Name);
  });
}
```

- EN: Declares or implements routines including `mustConsider`, `mustSkip`. Notable symbols here include `mustConsider`, `mustSkip`.
- CN: 这里声明或实现函数，例如 `mustConsider`, `mustSkip`。这里较值得关注的符号包括 `mustConsider`, `mustSkip`。

### Lines 120-127

```cpp
void syncOptions() {
  if (opts::InlineIgnoreCFI)
    opts::InlineIgnoreLeafCFI = true;

  if (opts::InlineAll)
    opts::InlineSmallFunctions = true;
}
```

- EN: Declares or implements routines including `syncOptions`. Notable symbols here include `syncOptions`.
- CN: 这里声明或实现函数，例如 `syncOptions`。这里较值得关注的符号包括 `syncOptions`。

### Lines 128-135

```cpp
} // namespace opts

namespace llvm {
namespace bolt {

uint64_t Inliner::SizeOfCallInst;
uint64_t Inliner::SizeOfTailCallInst;
```

- EN: Works inside namespace scope `opts`, `llvm`, `bolt` to organize symbols. Notable symbols here include `opts`, `llvm`, `bolt`.
- CN: 这里位于命名空间 `opts`, `llvm`, `bolt` 中，用于组织符号作用域。这里较值得关注的符号包括 `opts`, `llvm`, `bolt`。

### Lines 136-143

```cpp
uint64_t Inliner::getSizeOfCallInst(const BinaryContext &BC) {
  if (SizeOfCallInst)
    return SizeOfCallInst;

  MCInst Inst;
  BC.MIB->createCall(Inst, BC.Ctx->createNamedTempSymbol(), BC.Ctx.get());
  SizeOfCallInst = BC.computeInstructionSize(Inst);
```

- EN: Declares or implements routines including `getSizeOfCallInst`, `createCall`. Notable symbols here include `getSizeOfCallInst`, `createCall`.
- CN: 这里声明或实现函数，例如 `getSizeOfCallInst`, `createCall`。这里较值得关注的符号包括 `getSizeOfCallInst`, `createCall`。

### Lines 144-154

```cpp
  return SizeOfCallInst;
}

uint64_t Inliner::getSizeOfTailCallInst(const BinaryContext &BC) {
  if (SizeOfTailCallInst)
    return SizeOfTailCallInst;

  MCInst Inst;
  BC.MIB->createTailCall(Inst, BC.Ctx->createNamedTempSymbol(), BC.Ctx.get());
  SizeOfTailCallInst = BC.computeInstructionSize(Inst);
```

- EN: Declares or implements routines including `getSizeOfTailCallInst`, `createTailCall`. Notable symbols here include `getSizeOfTailCallInst`, `createTailCall`.
- CN: 这里声明或实现函数，例如 `getSizeOfTailCallInst`, `createTailCall`。这里较值得关注的符号包括 `getSizeOfTailCallInst`, `createTailCall`。

### Lines 155-163

```cpp
  return SizeOfTailCallInst;
}

InliningInfo getInliningInfo(const BinaryFunction &BF) {
  const BinaryContext &BC = BF.getBinaryContext();
  bool DirectSP = false;
  bool HasCFI = false;
  bool IsLeaf = true;
```

- EN: Declares or implements routines including `getInliningInfo`. Notable symbols here include `getInliningInfo`.
- CN: 这里声明或实现函数，例如 `getInliningInfo`。这里较值得关注的符号包括 `getInliningInfo`。

### Lines 164-171

```cpp
  // Perform necessary checks unless the option overrides it.
  if (!opts::mustConsider(BF)) {
    if (BF.hasSDTMarker())
      return INL_NONE;

    if (BF.hasEHRanges())
      return INL_NONE;
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 172-185

```cpp
    if (BF.isMultiEntry())
      return INL_NONE;

    if (BF.hasJumpTables())
      return INL_NONE;

    const MCPhysReg SPReg = BC.MIB->getStackPointer();
    for (const BinaryBasicBlock &BB : BF) {
      for (const MCInst &Inst : BB) {
        // Tail calls are marked as implicitly using the stack pointer and they
        // could be inlined.
        if (BC.MIB->isTailCall(Inst))
          break;
```

- EN: Declares or implements routines including `getStackPointer`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getStackPointer`.
- CN: 这里声明或实现函数，例如 `getStackPointer`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getStackPointer`。

### Lines 186-193

```cpp
        if (BC.MIB->isCFI(Inst)) {
          HasCFI = true;
          continue;
        }

        if (BC.MIB->isCall(Inst))
          IsLeaf = false;
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 194-204

```cpp
        // Push/pop instructions are straightforward to handle.
        if (BC.MIB->isPush(Inst) || BC.MIB->isPop(Inst))
          continue;

        // Pointer signing and authenticatin instructions are used around
        // Push and Pop. These are also straightforward to handle.
        if (BC.isAArch64() &&
            (BC.MIB->isPSignOnLR(Inst) || BC.MIB->isPAuthOnLR(Inst) ||
             BC.MIB->isPAuthAndRet(Inst)))
          continue;
```

- EN: Declares or implements routines including `isPAuthAndRet`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isPAuthAndRet`.
- CN: 这里声明或实现函数，例如 `isPAuthAndRet`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isPAuthAndRet`。

### Lines 205-214

```cpp
        DirectSP |= BC.MIB->hasDefOfPhysReg(Inst, SPReg) ||
                    BC.MIB->hasUseOfPhysReg(Inst, SPReg);
      }
    }
  }

  if (HasCFI) {
    if (!opts::InlineIgnoreLeafCFI)
      return INL_NONE;
```

- EN: Declares or implements routines including `hasDefOfPhysReg`, `hasUseOfPhysReg`. Notable symbols here include `hasDefOfPhysReg`, `hasUseOfPhysReg`.
- CN: 这里声明或实现函数，例如 `hasDefOfPhysReg`, `hasUseOfPhysReg`。这里较值得关注的符号包括 `hasDefOfPhysReg`, `hasUseOfPhysReg`。

### Lines 215-222

```cpp
    if (!IsLeaf && !opts::InlineIgnoreCFI)
      return INL_NONE;
  }

  InliningInfo Info(DirectSP ? INL_TAILCALL : INL_ANY);

  size_t Size = BF.estimateSize();
```

- EN: Declares or implements routines including `Info`. Notable symbols here include `Info`.
- CN: 这里声明或实现函数，例如 `Info`。这里较值得关注的符号包括 `Info`。

### Lines 223-238

```cpp
  Info.SizeAfterInlining = Size;
  Info.SizeAfterTailCallInlining = Size;

  // Handle special case of the known size reduction.
  if (BF.size() == 1) {
    // For a regular call the last return instruction could be removed
    // (or converted to a branch).
    const MCInst *LastInst = BF.back().getLastNonPseudoInstr();
    if (LastInst && BC.MIB->isReturn(*LastInst) &&
        !BC.MIB->isTailCall(*LastInst)) {
      const uint64_t RetInstSize = BC.computeInstructionSize(*LastInst);
      assert(Size >= RetInstSize);
      Info.SizeAfterInlining -= RetInstSize;
    }
  }
```

- EN: Declares or implements routines including `isTailCall`, `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `isTailCall`, `assert`.
- CN: 这里声明或实现函数，例如 `isTailCall`, `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `isTailCall`, `assert`。

### Lines 239-252

```cpp
  return Info;
}

void Inliner::findInliningCandidates(BinaryContext &BC) {
  for (const auto &BFI : BC.getBinaryFunctions()) {
    const BinaryFunction &Function = BFI.second;
    if (!shouldOptimize(Function) || opts::mustSkip(Function))
      continue;
    const InliningInfo InlInfo = getInliningInfo(Function);
    if (InlInfo.Type != INL_NONE)
      InliningCandidates[&Function] = InlInfo;
  }
}
```

- EN: Declares or implements routines including `findInliningCandidates`, `getInliningInfo`. Notable symbols here include `findInliningCandidates`, `getInliningInfo`.
- CN: 这里声明或实现函数，例如 `findInliningCandidates`, `getInliningInfo`。这里较值得关注的符号包括 `findInliningCandidates`, `getInliningInfo`。

### Lines 253-260

```cpp
std::pair<BinaryBasicBlock *, BinaryBasicBlock::iterator>
Inliner::inlineCall(BinaryBasicBlock &CallerBB,
                    BinaryBasicBlock::iterator CallInst,
                    const BinaryFunction &Callee) {
  BinaryFunction &CallerFunction = *CallerBB.getFunction();
  BinaryContext &BC = CallerFunction.getBinaryContext();
  auto &MIB = *BC.MIB;
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 261-273

```cpp
  assert(MIB.isCall(*CallInst) && "can only inline a call or a tail call");
  assert(!Callee.isMultiEntry() &&
         "cannot inline function with multiple entries");
  assert(!Callee.hasJumpTables() &&
         "cannot inline function with jump table(s)");

  // Get information about the call site.
  const bool CSIsInvoke = BC.MIB->isInvoke(*CallInst);
  const bool CSIsTailCall = BC.MIB->isTailCall(*CallInst);
  const int64_t CSGNUArgsSize = BC.MIB->getGnuArgsSize(*CallInst);
  const std::optional<MCPlus::MCLandingPad> CSEHInfo =
      BC.MIB->getEHInfo(*CallInst);
```

- EN: Declares or implements routines including `assert`, `table`, `isInvoke`, `isTailCall`, `getGnuArgsSize`, and 1 more. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `assert`, `table`, `isInvoke`, `isTailCall`, `getGnuArgsSize`, `getEHInfo`.
- CN: 这里声明或实现函数，例如 `assert`, `table`, `isInvoke`, `isTailCall`, `getGnuArgsSize`, and 1 more。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `assert`, `table`, `isInvoke`, `isTailCall`, `getGnuArgsSize`, `getEHInfo`。

### Lines 274-281

```cpp
  // Split basic block at the call site if there will be more incoming edges
  // coming from the callee.
  BinaryBasicBlock *FirstInlinedBB = &CallerBB;
  if (Callee.front().pred_size() && CallInst != CallerBB.begin()) {
    FirstInlinedBB = CallerBB.splitAt(CallInst);
    CallInst = FirstInlinedBB->begin();
  }
```

- EN: Declares or implements routines including `begin`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `begin`.
- CN: 这里声明或实现函数，例如 `begin`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `begin`。

### Lines 282-294

```cpp
  // Split basic block after the call instruction unless the callee is trivial
  // (i.e. consists of a single basic block). If necessary, obtain a basic block
  // for return instructions in the callee to redirect to.
  BinaryBasicBlock *NextBB = nullptr;
  if (Callee.size() > 1) {
    if (std::next(CallInst) != FirstInlinedBB->end())
      NextBB = FirstInlinedBB->splitAt(std::next(CallInst));
    else
      NextBB = FirstInlinedBB->getSuccessor();
  }
  if (NextBB)
    FirstInlinedBB->removeSuccessor(NextBB);
```

- EN: Declares or implements routines including `splitAt`, `getSuccessor`, `removeSuccessor`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `splitAt`, `getSuccessor`, `removeSuccessor`.
- CN: 这里声明或实现函数，例如 `splitAt`, `getSuccessor`, `removeSuccessor`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `splitAt`, `getSuccessor`, `removeSuccessor`。

### Lines 295-302

```cpp
  // Remove the call instruction.
  auto InsertII = FirstInlinedBB->eraseInstruction(CallInst);

  double ProfileRatio = 0;
  if (uint64_t CalleeExecCount = Callee.getKnownExecutionCount())
    ProfileRatio =
        (double)FirstInlinedBB->getKnownExecutionCount() / CalleeExecCount;
```

- EN: Declares or implements routines including `eraseInstruction`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `eraseInstruction`.
- CN: 这里声明或实现函数，例如 `eraseInstruction`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `eraseInstruction`。

### Lines 303-319

```cpp
  // Save execution count of the first block as we don't want it to change
  // later due to profile adjustment rounding errors.
  const uint64_t FirstInlinedBBCount = FirstInlinedBB->getKnownExecutionCount();

  // Copy basic blocks and maintain a map from their origin.
  std::unordered_map<const BinaryBasicBlock *, BinaryBasicBlock *> InlinedBBMap;
  InlinedBBMap[&Callee.front()] = FirstInlinedBB;
  for (const BinaryBasicBlock &BB : llvm::drop_begin(Callee)) {
    BinaryBasicBlock *InlinedBB = CallerFunction.addBasicBlock();
    InlinedBBMap[&BB] = InlinedBB;
    InlinedBB->setCFIState(FirstInlinedBB->getCFIState());
    if (Callee.hasValidProfile())
      InlinedBB->setExecutionCount(BB.getKnownExecutionCount());
    else
      InlinedBB->setExecutionCount(FirstInlinedBBCount);
  }
```

- EN: Declares or implements routines including `getKnownExecutionCount`, `setCFIState`, `setExecutionCount`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getKnownExecutionCount`, `setCFIState`, `setExecutionCount`.
- CN: 这里声明或实现函数，例如 `getKnownExecutionCount`, `setCFIState`, `setExecutionCount`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getKnownExecutionCount`, `setCFIState`, `setExecutionCount`。

### Lines 320-331

```cpp
  // Copy over instructions and edges.
  for (const BinaryBasicBlock &BB : Callee) {
    BinaryBasicBlock *InlinedBB = InlinedBBMap[&BB];

    if (InlinedBB != FirstInlinedBB)
      InsertII = InlinedBB->begin();

    // Copy over instructions making any necessary mods.
    for (MCInst Inst : BB) {
      if (MIB.isPseudo(Inst))
        continue;
```

- EN: Declares or implements routines including `begin`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `begin`.
- CN: 这里声明或实现函数，例如 `begin`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `begin`。

### Lines 332-347

```cpp
      MIB.stripAnnotations(Inst, /*KeepTC=*/BC.isX86() || BC.isAArch64());

      // Fix branch target. Strictly speaking, we don't have to do this as
      // targets of direct branches will be fixed later and don't matter
      // in the CFG state. However, disassembly may look misleading, and
      // hence we do the fixing.
      if (MIB.isBranch(Inst) && !MIB.isTailCall(Inst)) {
        assert(!MIB.isIndirectBranch(Inst) &&
               "unexpected indirect branch in callee");
        const BinaryBasicBlock *TargetBB =
            Callee.getBasicBlockForLabel(MIB.getTargetSymbol(Inst));
        assert(TargetBB && "cannot find target block in callee");
        MIB.replaceBranchTarget(Inst, InlinedBBMap[TargetBB]->getLabel(),
                                BC.Ctx.get());
      }
```

- EN: Declares or implements routines including `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `assert`.
- CN: 这里声明或实现函数，例如 `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `assert`。

### Lines 348-365

```cpp
      // Handling fused authentication and return instructions (Armv8.3-A):
      // if the Callee does not end in a tailcall, the return will be removed
      // from the inlined block. If that return is RETA(A|B), we have to keep
      // the authentication part.
      // RETAA -> AUTIASP
      // RETAB -> AUTIBSP
      if (!CSIsTailCall && BC.isAArch64() && BC.MIB->isPAuthAndRet(Inst)) {
        MCInst Auth;
        BC.MIB->createMatchingAuth(Inst, Auth);
        InsertII =
            std::next(InlinedBB->insertInstruction(InsertII, std::move(Auth)));
      }
      if (CSIsTailCall || (!MIB.isCall(Inst) && !MIB.isReturn(Inst))) {
        InsertII =
            std::next(InlinedBB->insertInstruction(InsertII, std::move(Inst)));
        continue;
      }
```

- EN: Declares or implements routines including `createMatchingAuth`, `next`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `createMatchingAuth`, `next`.
- CN: 这里声明或实现函数，例如 `createMatchingAuth`, `next`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `createMatchingAuth`, `next`。

### Lines 366-373

```cpp
      // Handle special instructions for a non-tail call site.
      if (!MIB.isCall(Inst)) {
        // Returns are removed.
        break;
      }

      MIB.convertTailCallToCall(Inst);
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 374-384

```cpp
      // Propagate EH-related info to call instructions.
      if (CSIsInvoke) {
        MIB.addEHInfo(Inst, *CSEHInfo);
        if (CSGNUArgsSize >= 0)
          MIB.addGnuArgsSize(Inst, CSGNUArgsSize);
      }

      InsertII =
          std::next(InlinedBB->insertInstruction(InsertII, std::move(Inst)));
    }
```

- EN: Declares or implements routines including `next`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `next`.
- CN: 这里声明或实现函数，例如 `next`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `next`。

### Lines 385-393

```cpp
    // Add CFG edges to the basic blocks of the inlined instance.
    std::vector<BinaryBasicBlock *> Successors(BB.succ_size());
    llvm::transform(BB.successors(), Successors.begin(),
                    [&InlinedBBMap](const BinaryBasicBlock *BB) {
                      auto It = InlinedBBMap.find(BB);
                      assert(It != InlinedBBMap.end());
                      return It->second;
                    });
```

- EN: Declares or implements routines including `Successors`, `transform`, `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `Successors`, `transform`, `assert`.
- CN: 这里声明或实现函数，例如 `Successors`, `transform`, `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `Successors`, `transform`, `assert`。

### Lines 394-404

```cpp
    if (CallerFunction.hasValidProfile() && Callee.hasValidProfile())
      InlinedBB->addSuccessors(Successors.begin(), Successors.end(),
                               BB.branch_info_begin(), BB.branch_info_end());
    else
      InlinedBB->addSuccessors(Successors.begin(), Successors.end());

    if (!CSIsTailCall && BB.succ_size() == 0 && NextBB) {
      // Either it's a return block or the last instruction never returns.
      InlinedBB->addSuccessor(NextBB, InlinedBB->getExecutionCount());
    }
```

- EN: Declares or implements routines including `addSuccessors`, `addSuccessor`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `addSuccessors`, `addSuccessor`.
- CN: 这里声明或实现函数，例如 `addSuccessors`, `addSuccessor`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `addSuccessors`, `addSuccessor`。

### Lines 405-414

```cpp
    // Scale profiling info for blocks and edges after inlining.
    if (CallerFunction.hasValidProfile() && Callee.size() > 1) {
      if (opts::AdjustProfile)
        InlinedBB->adjustExecutionCount(ProfileRatio);
      else
        InlinedBB->setExecutionCount(InlinedBB->getKnownExecutionCount() *
                                     ProfileRatio);
    }
  }
```

- EN: Declares or implements routines including `adjustExecutionCount`, `setExecutionCount`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `adjustExecutionCount`, `setExecutionCount`.
- CN: 这里声明或实现函数，例如 `adjustExecutionCount`, `setExecutionCount`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `adjustExecutionCount`, `setExecutionCount`。

### Lines 415-422

```cpp
  // Restore the original execution count of the first inlined basic block.
  FirstInlinedBB->setExecutionCount(FirstInlinedBBCount);

  CallerFunction.recomputeLandingPads();

  if (NextBB)
    return std::make_pair(NextBB, NextBB->begin());
```

- EN: Declares or implements routines including `setExecutionCount`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `setExecutionCount`.
- CN: 这里声明或实现函数，例如 `setExecutionCount`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `setExecutionCount`。

### Lines 423-437

```cpp
  if (Callee.size() == 1)
    return std::make_pair(FirstInlinedBB, InsertII);

  return std::make_pair(FirstInlinedBB, FirstInlinedBB->end());
}

bool Inliner::inlineCallsInFunction(BinaryFunction &Function) {
  BinaryContext &BC = Function.getBinaryContext();
  std::vector<BinaryBasicBlock *> Blocks(Function.getLayout().block_begin(),
                                         Function.getLayout().block_end());
  llvm::sort(
      Blocks, [](const BinaryBasicBlock *BB1, const BinaryBasicBlock *BB2) {
        return BB1->getKnownExecutionCount() > BB2->getKnownExecutionCount();
      });
```

- EN: Declares or implements routines including `inlineCallsInFunction`, `Blocks`. Notable symbols here include `inlineCallsInFunction`, `Blocks`.
- CN: 这里声明或实现函数，例如 `inlineCallsInFunction`, `Blocks`。这里较值得关注的符号包括 `inlineCallsInFunction`, `Blocks`。

### Lines 438-447

```cpp
  bool DidInlining = false;
  for (BinaryBasicBlock *BB : Blocks) {
    for (auto InstIt = BB->begin(); InstIt != BB->end();) {
      MCInst &Inst = *InstIt;
      if (!BC.MIB->isCall(Inst) || MCPlus::getNumPrimeOperands(Inst) != 1 ||
          !Inst.getOperand(0).isExpr()) {
        ++InstIt;
        continue;
      }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 448-459

```cpp
      const MCSymbol *TargetSymbol = BC.MIB->getTargetSymbol(Inst);
      assert(TargetSymbol && "target symbol expected for direct call");

      // Don't inline calls to a secondary entry point in a target function.
      uint64_t EntryID = 0;
      BinaryFunction *TargetFunction =
          BC.getFunctionForSymbol(TargetSymbol, &EntryID);
      if (!TargetFunction || EntryID != 0) {
        ++InstIt;
        continue;
      }
```

- EN: Declares or implements routines including `getTargetSymbol`, `assert`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getTargetSymbol`, `assert`.
- CN: 这里声明或实现函数，例如 `getTargetSymbol`, `assert`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getTargetSymbol`, `assert`。

### Lines 460-471

```cpp
      // Don't do recursive inlining.
      if (TargetFunction == &Function) {
        ++InstIt;
        continue;
      }

      auto IInfo = InliningCandidates.find(TargetFunction);
      if (IInfo == InliningCandidates.end()) {
        ++InstIt;
        continue;
      }
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 472-485

```cpp
      const bool IsTailCall = BC.MIB->isTailCall(Inst);
      if (!IsTailCall && IInfo->second.Type == INL_TAILCALL) {
        ++InstIt;
        continue;
      }

      int64_t SizeAfterInlining;
      if (IsTailCall)
        SizeAfterInlining =
            IInfo->second.SizeAfterTailCallInlining - getSizeOfTailCallInst(BC);
      else
        SizeAfterInlining =
            IInfo->second.SizeAfterInlining - getSizeOfCallInst(BC);
```

- EN: Declares or implements routines including `isTailCall`, `getSizeOfTailCallInst`, `getSizeOfCallInst`. Notable symbols here include `isTailCall`, `getSizeOfTailCallInst`, `getSizeOfCallInst`.
- CN: 这里声明或实现函数，例如 `isTailCall`, `getSizeOfTailCallInst`, `getSizeOfCallInst`。这里较值得关注的符号包括 `isTailCall`, `getSizeOfTailCallInst`, `getSizeOfCallInst`。

### Lines 486-493

```cpp
      if (!opts::InlineAll && !opts::mustConsider(*TargetFunction)) {
        if (!opts::InlineSmallFunctions ||
            SizeAfterInlining > opts::InlineSmallFunctionsBytes) {
          ++InstIt;
          continue;
        }
      }
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 494-510

```cpp
      // AArch64 BTI:
      // If the callee has an indirect tailcall (BR), we would transform it to
      // an indirect call (BLR) in InlineCall. Because of this, we would have to
      // update the BTI at the target of the tailcall. However, these targets
      // are not known. Instead, we skip inlining blocks with indirect
      // tailcalls.
      auto HasIndirectTailCall = [&](const BinaryFunction &BF) -> bool {
        for (const auto &BB : BF) {
          for (const auto &II : BB) {
            if (BC.MIB->isIndirectBranch(II) && BC.MIB->isTailCall(II)) {
              return true;
            }
          }
        }
        return false;
      };
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 511-519

```cpp
      if (BC.isAArch64() && BC.usesBTI() &&
          HasIndirectTailCall(*TargetFunction)) {
        ++InstIt;
        LLVM_DEBUG(dbgs() << "BOLT-DEBUG: Skipping inlining block with tailcall"
                          << " in " << Function << " : " << BB->getName()
                          << " to keep BTIs consistent.\n");
        continue;
      }
```

- EN: Declares or implements routines including `HasIndirectTailCall`, `LLVM_DEBUG`, `getName`. Notable symbols here include `HasIndirectTailCall`, `LLVM_DEBUG`, `getName`.
- CN: 这里声明或实现函数，例如 `HasIndirectTailCall`, `LLVM_DEBUG`, `getName`。这里较值得关注的符号包括 `HasIndirectTailCall`, `LLVM_DEBUG`, `getName`。

### Lines 520-527

```cpp
      LLVM_DEBUG(dbgs() << "BOLT-DEBUG: inlining call to " << *TargetFunction
                        << " in " << Function << " : " << BB->getName()
                        << ". Count: " << BB->getKnownExecutionCount()
                        << ". Size change: " << SizeAfterInlining
                        << " bytes.\n");

      std::tie(BB, InstIt) = inlineCall(*BB, InstIt, *TargetFunction);
```

- EN: Declares or implements routines including `LLVM_DEBUG`, `getName`, `getKnownExecutionCount`, `tie`. Notable symbols here include `LLVM_DEBUG`, `getName`, `getKnownExecutionCount`, `tie`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`, `getName`, `getKnownExecutionCount`, `tie`。这里较值得关注的符号包括 `LLVM_DEBUG`, `getName`, `getKnownExecutionCount`, `tie`。

### Lines 528-537

```cpp
      DidInlining = true;
      TotalInlinedBytes += SizeAfterInlining;

      ++NumInlinedCallSites;
      NumInlinedDynamicCalls += BB->getExecutionCount();

      // Subtract basic block execution count from the callee execution count.
      if (opts::AdjustProfile)
        TargetFunction->adjustExecutionCount(BB->getKnownExecutionCount());
```

- EN: Declares or implements routines including `getExecutionCount`, `adjustExecutionCount`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `getExecutionCount`, `adjustExecutionCount`.
- CN: 这里声明或实现函数，例如 `getExecutionCount`, `adjustExecutionCount`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `getExecutionCount`, `adjustExecutionCount`。

### Lines 538-548

```cpp
      // Check if the caller inlining status has to be adjusted.
      if (IInfo->second.Type == INL_TAILCALL) {
        auto CallerIInfo = InliningCandidates.find(&Function);
        if (CallerIInfo != InliningCandidates.end() &&
            CallerIInfo->second.Type == INL_ANY) {
          LLVM_DEBUG(dbgs() << "adjusting inlining status for function "
                            << Function << '\n');
          CallerIInfo->second.Type = INL_TAILCALL;
        }
      }
```

- EN: Declares or implements routines including `LLVM_DEBUG`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `LLVM_DEBUG`.
- CN: 这里声明或实现函数，例如 `LLVM_DEBUG`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `LLVM_DEBUG`。

### Lines 549-556

```cpp
      if (NumInlinedCallSites == opts::InlineLimit)
        return true;
    }
  }

  return DidInlining;
}
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 557-568

```cpp
Error Inliner::runOnFunctions(BinaryContext &BC) {
  opts::syncOptions();

  if (!opts::inliningEnabled())
    return Error::success();

  bool InlinedOnce;
  unsigned NumIters = 0;
  do {
    if (opts::InlineLimit && NumInlinedCallSites >= opts::InlineLimit)
      break;
```

- EN: Declares or implements routines including `runOnFunctions`, `syncOptions`. Notable symbols here include `runOnFunctions`, `syncOptions`.
- CN: 这里声明或实现函数，例如 `runOnFunctions`, `syncOptions`。这里较值得关注的符号包括 `runOnFunctions`, `syncOptions`。

### Lines 569-586

```cpp
    InlinedOnce = false;

    InliningCandidates.clear();
    findInliningCandidates(BC);

    BinaryFunctionListType ConsideredFunctions;
    for (auto &BFI : BC.getBinaryFunctions()) {
      BinaryFunction &Function = BFI.second;
      if (!shouldOptimize(Function))
        continue;
      ConsideredFunctions.push_back(&Function);
    }
    llvm::sort(ConsideredFunctions, [](const BinaryFunction *A,
                                       const BinaryFunction *B) {
      return B->getKnownExecutionCount() < A->getKnownExecutionCount();
    });
    for (BinaryFunction *Function : ConsideredFunctions) {
      if (opts::InlineLimit && NumInlinedCallSites >= opts::InlineLimit)
```

- EN: Declares or implements routines including `findInliningCandidates`. Notable symbols here include `findInliningCandidates`.
- CN: 这里声明或实现函数，例如 `findInliningCandidates`。这里较值得关注的符号包括 `findInliningCandidates`。

### Lines 587-596

```cpp
        break;

      const bool DidInline = inlineCallsInFunction(*Function);

      if (DidInline)
        Modified.insert(Function);

      InlinedOnce |= DidInline;
    }
```

- EN: Declares or implements routines including `inlineCallsInFunction`. Notable symbols here include `inlineCallsInFunction`.
- CN: 这里声明或实现函数，例如 `inlineCallsInFunction`。这里较值得关注的符号包括 `inlineCallsInFunction`。

### Lines 597-607

```cpp
    ++NumIters;
  } while (InlinedOnce && NumIters < opts::InlineMaxIters);

  if (NumInlinedCallSites)
    BC.outs() << "BOLT-INFO: inlined " << NumInlinedDynamicCalls << " calls at "
              << NumInlinedCallSites << " call sites in " << NumIters
              << " iteration(s). Change in binary size: " << TotalInlinedBytes
              << " bytes.\n";
  return Error::success();
}
```

- EN: Declares or implements routines including `while`, `iteration`. Notable symbols here include `while`, `iteration`.
- CN: 这里声明或实现函数，例如 `while`, `iteration`。这里较值得关注的符号包括 `while`, `iteration`。

### Lines 608-609

```cpp
} // namespace bolt
} // namespace llvm
```

- EN: Works inside namespace scope `bolt`, `llvm` to organize symbols. Notable symbols here include `bolt`, `llvm`.
- CN: 这里位于命名空间 `bolt`, `llvm` 中，用于组织符号作用域。这里较值得关注的符号包括 `bolt`, `llvm`。

## Key Concepts / 关键概念

- `used`: class or struct interface / 类或结构体接口
- `desc`: function or method entry point / 函数或方法入口
- `cat`: function or method entry point / 函数或方法入口
- `value_desc`: function or method entry point / 函数或方法入口
- `InlineAll`: function or method entry point / 函数或方法入口
- `init`: function or method entry point / 函数或方法入口
- `DEBUG_TYPE`: macro or compile-time switch / 宏或编译期开关
- `llvm`: namespace scope / 命名空间作用域

## Dependencies / 依赖关系

- Local headers / 本地头文件: `bolt/Passes/Inliner.h`, `bolt/Core/MCPlus.h`
- LLVM headers / LLVM 头文件: `llvm/Support/CommandLine.h`
- Directory context / 目录上下文: `bolt/lib/Passes` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/lib/Passes` 下的相邻文件通常与本文件协作组成对应子系统
