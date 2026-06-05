# WebAssemblyReduceToAnyAllTrue.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyReduceToAnyAllTrue.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file provides backend implementation logic for the WebAssembly backend.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyReduceToAnyAllTrue.cpp`，主要负责 WebAssembly 后端的后端实现逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7

```cpp
#include "WebAssembly.h"
#include "WebAssemblySubtarget.h"
#include "WebAssemblyTargetMachine.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/IR/IntrinsicsWebAssembly.h"
#include "llvm/IR/Module.h"
#include "llvm/IR/PatternMatch.h"
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file. Subtarget feature gating influences the behavior here.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。 子目标特性裁剪会影响这里的行为。

### Lines 8-45

```cpp
#include "llvm/Pass.h"

using namespace llvm;
using namespace llvm::PatternMatch;

namespace {
struct WebAssemblyReduceToAnyAllTrue final : FunctionPass {
  static char ID;

  WebAssemblyTargetMachine &TM;
  const Module *CachedModule = nullptr;
  bool ModuleHasInterestingIntrinsics = false;

  WebAssemblyReduceToAnyAllTrue(WebAssemblyTargetMachine &TM)
      : FunctionPass(ID), TM(TM) {}

  StringRef getPassName() const override {
    return "WebAssembly convert reduce to any_true/all_true";
  }

  bool hasInterestingIntrinsics(const Module &M) {
    if (CachedModule == &M)
      return ModuleHasInterestingIntrinsics;

    CachedModule = &M;
    ModuleHasInterestingIntrinsics = false;

    for (const Function &Fn : M.functions()) {
      switch (Fn.getIntrinsicID()) {
      case Intrinsic::vector_reduce_or:
      case Intrinsic::vector_reduce_and:
        ModuleHasInterestingIntrinsics = true;
        return true;
      default:
        break;
      }
    }
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. These declarations feed generated pattern-matching logic.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这些声明会进入生成式模式匹配逻辑。

### Lines 46-83

```cpp
    return false;
  }

  bool runOnFunction(Function &F) override {
    if (!TM.getSubtarget<WebAssemblySubtarget>(F).hasSIMD128())
      return false;

    if (!hasInterestingIntrinsics(*F.getParent()))
      return false;

    bool Changed = false;

    for (auto &BB : F) {
      for (auto It = BB.begin(), E = BB.end(); It != E;) {
        Instruction *I = &*It++;
        auto *Cmp = dyn_cast<ICmpInst>(I);
        if (!Cmp || Cmp->getPredicate() != ICmpInst::ICMP_NE)
          continue;

        Value *Reduce = nullptr;
        if (!match(Cmp, m_ICmp(m_Value(Reduce), m_ZeroInt())))
          continue;

        auto *II = dyn_cast<IntrinsicInst>(Reduce);
        if (!II || !II->hasOneUse())
          continue;

        IRBuilder<> B(Cmp);
        Value *Vec = II->getArgOperand(0);
        Module *M = F.getParent();

        auto makeIntrinsic = [&](Intrinsic::ID ID, Value *Arg) {
          Function *Fn =
              Intrinsic::getOrInsertDeclaration(M, ID, {Arg->getType()});
          return B.CreateCall(Fn, {Arg});
        };

        Value *New = nullptr;
```
- **EN**: Implements helper routine(s) `runOnFunction`, `hasSIMD128`, `hasInterestingIntrinsics` for this portion of the WebAssembly backend backend implementation logic. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `runOnFunction`, `hasSIMD128`, `hasInterestingIntrinsics`。 子目标特性裁剪会影响这里的行为。

### Lines 84-121

```cpp

        switch (II->getIntrinsicID()) {
        case Intrinsic::vector_reduce_or: {
          // reduce.or(X) != 0  -> anytrue(X)
          Value *Any = makeIntrinsic(Intrinsic::wasm_anytrue, Vec);
          New = B.CreateICmpNE(Any, ConstantInt::get(Any->getType(), 0));
          break;
        }

        case Intrinsic::vector_reduce_and: {
          // reduce.and(zext (icmp ne X, zeroinitializer)) != 0  -> alltrue(X)

          // Match: zext (icmp ne X, 0) from <N x i1> to <N x iX>
          CmpPredicate Pred;
          Value *LHS = nullptr;
          if (!match(Vec, m_ZExt(m_c_ICmp(Pred, m_Value(LHS), m_Zero()))))
            continue;
          if (Pred != ICmpInst::ICMP_NE)
            continue;

          Value *All = makeIntrinsic(Intrinsic::wasm_alltrue, LHS);
          New = B.CreateICmpNE(All, ConstantInt::get(All->getType(), 0));
          break;
        }

        default:
          continue;
        }

        Cmp->replaceAllUsesWith(New);
        Cmp->eraseFromParent();

        if (II->use_empty())
          II->eraseFromParent();

        Changed = true;
      }
    }
```
- **EN**: Implements helper routine(s) `getIntrinsicID`, `or`, `anytrue` for this portion of the WebAssembly backend backend implementation logic. These declarations feed generated pattern-matching logic.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `getIntrinsicID`, `or`, `anytrue`。 这些声明会进入生成式模式匹配逻辑。

### Lines 122-133

```cpp

    return Changed;
  }
};
} // end anonymous namespace

char WebAssemblyReduceToAnyAllTrue::ID = 0;

FunctionPass *
llvm::createWebAssemblyReduceToAnyAllTrue(WebAssemblyTargetMachine &TM) {
  return new WebAssemblyReduceToAnyAllTrue(TM);
}
```
- **EN**: Implements helper routine(s) `createWebAssemblyReduceToAnyAllTrue`, `WebAssemblyReduceToAnyAllTrue` for this portion of the WebAssembly backend backend implementation logic.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `createWebAssemblyReduceToAnyAllTrue`, `WebAssemblyReduceToAnyAllTrue`。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Backend implementation logic / 后端实现逻辑
- Subtarget features / 子目标特性
- Target machine configuration / 目标机器配置
- Vector or SIMD behavior / 向量或 SIMD 行为

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `WebAssembly.h`
- `WebAssemblySubtarget.h`
- `WebAssemblyTargetMachine.h`
- `llvm/IR/IRBuilder.h`
- `llvm/IR/IntrinsicsWebAssembly.h`
- `llvm/IR/Module.h`
- `llvm/IR/PatternMatch.h`
- `llvm/Pass.h`

### Important Collaborators / 重要协作组件

- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
