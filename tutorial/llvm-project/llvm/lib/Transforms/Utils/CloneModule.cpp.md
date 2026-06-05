# CloneModule.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/CloneModule.cpp` | `llvm/lib/Transforms/Utils/CloneModule.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements clone an entire module within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 CloneModule 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

```cpp
//===- CloneModule.cpp - Clone an entire module ---------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the CloneModule interface which makes a copy of an
// entire module.
//
//===----------------------------------------------------------------------===//

#include "llvm-c/Core.h"
#include "llvm/IR/DerivedTypes.h"
#include "llvm/IR/Module.h"
#include "llvm/Transforms/Utils/Cloning.h"
#include "llvm/Transforms/Utils/ValueMapper.h"
using namespace llvm;

namespace llvm {
class Constant;
}

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include Constant, which carry the local state, declarations, or transformation entry points for this region.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 Constant，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 25-51

```cpp
static void copyComdat(GlobalObject *Dst, const GlobalObject *Src) {
  const Comdat *SC = Src->getComdat();
  if (!SC)
    return;
  Comdat *DC = Dst->getParent()->getOrInsertComdat(SC->getName());
  DC->setSelectionKind(SC->getSelectionKind());
  Dst->setComdat(DC);
}

/// This is not as easy as it might seem because we have to worry about making
/// copies of global variables and functions, and making their (initializers and
/// references, respectively) refer to the right globals.
///
/// Cloning un-materialized modules is not currently supported, so any
/// modules initialized via lazy loading should be materialized before cloning
std::unique_ptr<Module> llvm::CloneModule(const Module &M) {
  // Create the value map that maps things from the old module over to the new
  // module.
  ValueToValueMapTy VMap;
  return CloneModule(M, VMap);
}

std::unique_ptr<Module> llvm::CloneModule(const Module &M,
                                          ValueToValueMapTy &VMap) {
  return CloneModule(M, VMap, [](const GlobalValue *GV) { return true; });
}

```
- EN: Core entities appearing here include copyComdat, CloneModule, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 copyComdat, CloneModule，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 52-78

```cpp
std::unique_ptr<Module> llvm::CloneModule(
    const Module &M, ValueToValueMapTy &VMap,
    function_ref<bool(const GlobalValue *)> ShouldCloneDefinition) {

  assert(M.isMaterialized() && "Module must be materialized before cloning!");

  // First off, we need to create the new module.
  std::unique_ptr<Module> New =
      std::make_unique<Module>(M.getModuleIdentifier(), M.getContext());
  New->setSourceFileName(M.getSourceFileName());
  New->setDataLayout(M.getDataLayout());
  New->setTargetTriple(M.getTargetTriple());
  New->setModuleInlineAsm(M.getModuleInlineAsm());

  // Loop over all of the global variables, making corresponding globals in the
  // new module.  Here we add them to the VMap and to the new Module.  We
  // don't worry about attributes or initializers, they will come later.
  //
  for (const GlobalVariable &I : M.globals()) {
    GlobalVariable *NewGV = new GlobalVariable(
        *New, I.getValueType(), I.isConstant(), I.getLinkage(),
        (Constant *)nullptr, I.getName(), (GlobalVariable *)nullptr,
        I.getThreadLocalMode(), I.getType()->getAddressSpace());
    NewGV->copyAttributesFrom(&I);
    VMap[&I] = NewGV;
  }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 79-101

```cpp
  // Loop over the functions in the module, making external functions as before
  for (const Function &I : M) {
    Function *NF =
        Function::Create(I.getFunctionType(), I.getLinkage(),
                         I.getAddressSpace(), I.getName(), New.get());
    NF->copyAttributesFrom(&I);
    VMap[&I] = NF;
  }

  // Loop over the aliases in the module
  for (const GlobalAlias &I : M.aliases()) {
    if (!ShouldCloneDefinition(&I)) {
      // An alias cannot act as an external reference, so we need to create
      // either a function or a global variable depending on the value type.
      // FIXME: Once pointee types are gone we can probably pick one or the
      // other.
      GlobalValue *GV;
      if (I.getValueType()->isFunctionTy())
        GV = Function::Create(cast<FunctionType>(I.getValueType()),
                              GlobalValue::ExternalLinkage, I.getAddressSpace(),
                              I.getName(), New.get());
      else
        GV = new GlobalVariable(*New, I.getValueType(), false,
```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 102-126

```cpp
                                GlobalValue::ExternalLinkage, nullptr,
                                I.getName(), nullptr, I.getThreadLocalMode(),
                                I.getType()->getAddressSpace());
      VMap[&I] = GV;
      // We do not copy attributes (mainly because copying between different
      // kinds of globals is forbidden), but this is generally not required for
      // correctness.
      continue;
    }
    auto *GA = GlobalAlias::create(I.getValueType(),
                                   I.getType()->getPointerAddressSpace(),
                                   I.getLinkage(), I.getName(), New.get());
    GA->copyAttributesFrom(&I);
    VMap[&I] = GA;
  }

  for (const GlobalIFunc &I : M.ifuncs()) {
    // Defer setting the resolver function until after functions are cloned.
    auto *GI =
        GlobalIFunc::create(I.getValueType(), I.getAddressSpace(),
                            I.getLinkage(), I.getName(), nullptr, New.get());
    GI->copyAttributesFrom(&I);
    VMap[&I] = GI;
  }

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 127-149

```cpp
  // Similarly, copy over function bodies now...
  //
  for (const Function &I : M) {
    Function *F = cast<Function>(VMap[&I]);

    if (I.isDeclaration()) {
      // Copy over metadata for declarations since we're not doing it below in
      // CloneFunctionInto().
      SmallVector<std::pair<unsigned, MDNode *>, 1> MDs;
      I.getAllMetadata(MDs);
      for (auto MD : MDs)
        F->addMetadata(MD.first, *MapMetadata(MD.second, VMap));
      continue;
    }

    if (!ShouldCloneDefinition(&I)) {
      // Skip after setting the correct linkage for an external reference.
      F->setLinkage(GlobalValue::ExternalLinkage);
      // Personality function is not valid on a declaration.
      F->setPersonalityFn(nullptr);
      continue;
    }

```
- EN: This region continues the CloneModule implementation with local helper logic centered on Similarly, Function, VMap, Copy.
- CN: 这一段延续了 CloneModule 的主体实现，围绕 Similarly, Function, VMap, Copy 等局部辅助逻辑展开。

### Lines 150-175

```cpp
    Function::arg_iterator DestI = F->arg_begin();
    for (const Argument &J : I.args()) {
      DestI->setName(J.getName());
      VMap[&J] = &*DestI++;
    }

    SmallVector<ReturnInst *, 8> Returns; // Ignore returns cloned.
    CloneFunctionInto(F, &I, VMap, CloneFunctionChangeType::ClonedModule,
                      Returns);

    if (I.hasPersonalityFn())
      F->setPersonalityFn(MapValue(I.getPersonalityFn(), VMap));

    copyComdat(F, &I);
  }

  // And aliases
  for (const GlobalAlias &I : M.aliases()) {
    // We already dealt with undefined aliases above.
    if (!ShouldCloneDefinition(&I))
      continue;
    GlobalAlias *GA = cast<GlobalAlias>(VMap[&I]);
    if (const Constant *C = I.getAliasee())
      GA->setAliasee(MapValue(C, VMap));
  }

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 176-200

```cpp
  for (const GlobalIFunc &I : M.ifuncs()) {
    GlobalIFunc *GI = cast<GlobalIFunc>(VMap[&I]);
    if (const Constant *Resolver = I.getResolver())
      GI->setResolver(MapValue(Resolver, VMap));
  }

  // And named metadata....
  for (const NamedMDNode &NMD : M.named_metadata()) {
    NamedMDNode *NewNMD = New->getOrInsertNamedMetadata(NMD.getName());
    for (const MDNode *N : NMD.operands())
      NewNMD->addOperand(MapMetadata(N, VMap));
  }

  // Now that all of the things that global variable initializer can refer to
  // have been created, loop through and copy the global variable referrers
  // over...  We also set the attributes on the global now.
  //
  for (const GlobalVariable &G : M.globals()) {
    GlobalVariable *GV = cast<GlobalVariable>(VMap[&G]);

    SmallVector<std::pair<unsigned, MDNode *>, 1> MDs;
    G.getAllMetadata(MDs);
    for (auto MD : MDs)
      GV->addMetadata(MD.first, *MapMetadata(MD.second, VMap));

```
- EN: This region continues the CloneModule implementation with local helper logic centered on GlobalIFunc, VMap, Constant, Resolver.
- CN: 这一段延续了 CloneModule 的主体实现，围绕 GlobalIFunc, VMap, Constant, Resolver 等局部辅助逻辑展开。

### Lines 201-223

```cpp
    if (G.isDeclaration())
      continue;

    if (!ShouldCloneDefinition(&G)) {
      // Skip after setting the correct linkage for an external reference.
      GV->setLinkage(GlobalValue::ExternalLinkage);
      continue;
    }
    if (G.hasInitializer())
      GV->setInitializer(MapValue(G.getInitializer(), VMap));

    copyComdat(GV, &G);
  }

  return New;
}

extern "C" {

LLVMModuleRef LLVMCloneModule(LLVMModuleRef M) {
  return wrap(CloneModule(*unwrap(M)).release());
}

```
- EN: Core entities appearing here include LLVMCloneModule, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 LLVMCloneModule，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 224-224

```cpp
}
```
- EN: This region continues the CloneModule implementation with local helper logic centered on CloneModule.
- CN: 这一段延续了 CloneModule 的主体实现，围绕 CloneModule 等局部辅助逻辑展开。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `Constant, copyComdat, CloneModule, LLVMCloneModule` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`Constant, copyComdat, CloneModule, LLVMCloneModule` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DataLayout` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DataLayout` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: IR/transform headers: `llvm/IR/DerivedTypes.h`, `llvm/IR/Module.h`, `llvm/Transforms/Utils/Cloning.h`, `llvm/Transforms/Utils/ValueMapper.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/DerivedTypes.h`, `llvm/IR/Module.h`, `llvm/Transforms/Utils/Cloning.h`, `llvm/Transforms/Utils/ValueMapper.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Standard/other headers: `llvm-c/Core.h` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`llvm-c/Core.h` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `DataLayout` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DataLayout` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
