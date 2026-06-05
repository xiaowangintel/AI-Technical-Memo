# DeclareRuntimeLibcalls.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/DeclareRuntimeLibcalls.cpp` | `llvm/lib/Transforms/Utils/DeclareRuntimeLibcalls.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements Declare Runtime Libcalls within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 DeclareRuntimeLibcalls 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-17

```cpp
//===- DeclareRuntimeLibcalls.cpp -----------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Insert declarations for all runtime library calls known for the target.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/DeclareRuntimeLibcalls.h"
#include "llvm/Analysis/RuntimeLibcallInfo.h"
#include "llvm/IR/Module.h"
#include "llvm/IR/RuntimeLibcalls.h"

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。

### Lines 18-33

```cpp
using namespace llvm;

static void mergeAttributes(LLVMContext &Ctx, const Module &M,
                            const DataLayout &DL, const Triple &TT,
                            Function *Func, FunctionType *FuncTy,
                            AttributeList FuncAttrs) {
  AttributeList OldAttrs = Func->getAttributes();
  AttributeList NewAttrs = OldAttrs;

  {
    AttrBuilder OldBuilder(Ctx, OldAttrs.getFnAttrs());
    AttrBuilder NewBuilder(Ctx, FuncAttrs.getFnAttrs());
    OldBuilder.merge(NewBuilder);
    NewAttrs = NewAttrs.addFnAttributes(Ctx, OldBuilder);
  }

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 34-50

```cpp
  {
    AttrBuilder OldBuilder(Ctx, OldAttrs.getRetAttrs());
    AttrBuilder NewBuilder(Ctx, FuncAttrs.getRetAttrs());
    OldBuilder.merge(NewBuilder);
    NewAttrs = NewAttrs.addRetAttributes(Ctx, OldBuilder);
  }

  for (unsigned I = 0, E = FuncTy->getNumParams(); I != E; ++I) {
    AttrBuilder OldBuilder(Ctx, OldAttrs.getParamAttrs(I));
    AttrBuilder NewBuilder(Ctx, FuncAttrs.getParamAttrs(I));
    OldBuilder.merge(NewBuilder);
    NewAttrs = NewAttrs.addParamAttributes(Ctx, I, OldBuilder);
  }

  Func->setAttributes(NewAttrs);
}

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 51-65

```cpp
PreservedAnalyses DeclareRuntimeLibcallsPass::run(Module &M,
                                                  ModuleAnalysisManager &MAM) {
  const RTLIB::RuntimeLibcallsInfo &RTLCI =
      MAM.getResult<RuntimeLibraryAnalysis>(M);

  LLVMContext &Ctx = M.getContext();
  const DataLayout &DL = M.getDataLayout();
  const Triple &TT = M.getTargetTriple();

  for (RTLIB::LibcallImpl Impl : RTLIB::libcall_impls()) {
    if (!RTLCI.isAvailable(Impl))
      continue;

    auto [FuncTy, FuncAttrs] = RTLCI.getFunctionTy(Ctx, TT, DL, Impl);

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。

### Lines 66-79

```cpp
    // TODO: Declare with correct type, calling convention, and attributes.
    if (!FuncTy)
      FuncTy = FunctionType::get(Type::getVoidTy(Ctx), {}, /*IsVarArgs=*/true);

    StringRef FuncName = RTLCI.getLibcallImplName(Impl);

    Function *Func =
        cast<Function>(M.getOrInsertFunction(FuncName, FuncTy).getCallee());
    if (Func->getFunctionType() == FuncTy) {
      mergeAttributes(Ctx, M, DL, TT, Func, FuncTy, FuncAttrs);
      Func->setCallingConv(RTLCI.getLibcallImplCallingConv(Impl));
    }
  }

```
- EN: This region continues the DeclareRuntimeLibcalls implementation with local helper logic centered on TODO, Declare, FuncTy, FunctionType.
- CN: 这一段延续了 DeclareRuntimeLibcalls 的主体实现，围绕 TODO, Declare, FuncTy, FunctionType 等局部辅助逻辑展开。

### Lines 80-81

```cpp
  return PreservedAnalyses::none();
}
```
- EN: This region continues the DeclareRuntimeLibcalls implementation with local helper logic centered on PreservedAnalyses.
- CN: 这一段延续了 DeclareRuntimeLibcalls 的主体实现，围绕 PreservedAnalyses 等局部辅助逻辑展开。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Analysis coupling: it cooperates with `DataLayout` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DataLayout` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/RuntimeLibcallInfo.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/RuntimeLibcallInfo.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/Module.h`, `llvm/IR/RuntimeLibcalls.h`, `llvm/Transforms/Utils/DeclareRuntimeLibcalls.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Module.h`, `llvm/IR/RuntimeLibcalls.h`, `llvm/Transforms/Utils/DeclareRuntimeLibcalls.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Runtime analysis dependencies: `DataLayout` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DataLayout` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
