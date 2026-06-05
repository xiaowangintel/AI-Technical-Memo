# SplitModuleByCategory.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/SplitModuleByCategory.cpp` | `llvm/lib/Transforms/Utils/SplitModuleByCategory.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements split a module by categories within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 SplitModuleByCategory 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-33

```cpp
//===-------- SplitModuleByCategory.cpp - split a module by categories ----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
// See comments in the header.
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/SplitModuleByCategory.h"
#include "llvm/ADT/SetVector.h"
#include "llvm/ADT/SmallPtrSet.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/InstIterator.h"
#include "llvm/IR/Instructions.h"
#include "llvm/IR/Module.h"
#include "llvm/Support/Debug.h"
#include "llvm/Transforms/Utils/Cloning.h"

#include <map>
#include <utility>

using namespace llvm;

#define DEBUG_TYPE "split-module-by-category"

namespace {

// A vector that contains a group of function with the same category.
using EntryPointSet = SetVector<const Function *>;

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 34-65

```cpp
/// Represents a group of functions with one category.
struct EntryPointGroup {
  int ID;
  EntryPointSet Functions;

  EntryPointGroup() = default;

  EntryPointGroup(int ID, EntryPointSet &&Functions = EntryPointSet())
      : ID(ID), Functions(std::move(Functions)) {}

  void clear() { Functions.clear(); }

#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
  LLVM_DUMP_METHOD void dump() const {
    constexpr size_t INDENT = 4;
    dbgs().indent(INDENT) << "ENTRY POINTS"
                          << " " << ID << " {\n";
    for (const Function *F : Functions)
      dbgs().indent(INDENT) << "  " << F->getName() << "\n";

    dbgs().indent(INDENT) << "}\n";
  }
#endif
};

/// Annotates an llvm::Module with information necessary to perform and track
/// the result of code (llvm::Module instances) splitting:
/// - entry points group from the module.
class ModuleDesc {
  std::unique_ptr<Module> M;
  EntryPointGroup EntryPoints;

```
- EN: Core entities appearing here include EntryPointGroup, dump, ModuleDesc, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 EntryPointGroup, dump, ModuleDesc，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 66-97

```cpp
public:
  ModuleDesc(std::unique_ptr<Module> M,
             EntryPointGroup &&EntryPoints = EntryPointGroup())
      : M(std::move(M)), EntryPoints(std::move(EntryPoints)) {
    assert(this->M && "Module should be non-null");
  }

  Module &getModule() { return *M; }
  const Module &getModule() const { return *M; }

  std::unique_ptr<Module> releaseModule() {
    EntryPoints.clear();
    return std::move(M);
  }

#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
  LLVM_DUMP_METHOD void dump() const {
    dbgs() << "ModuleDesc[" << M->getName() << "] {\n";
    EntryPoints.dump();
    dbgs() << "}\n";
  }
#endif
};

// Represents "dependency" or "use" graph of global objects (functions and
// global variables) in a module. It is used during code split to
// understand which global variables and functions (other than entry points)
// should be included into a split module.
//
// Nodes of the graph represent LLVM's GlobalObjects, edges "A" -> "B" represent
// the fact that if "A" is included into a module, then "B" should be included
// as well.
```
- EN: Core entities appearing here include M, releaseModule, dump, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 M, releaseModule, dump，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 98-132

```cpp
//
// Examples of dependencies which are represented in this graph:
// - Function FA calls function FB
// - Function FA uses global variable GA
// - Global variable GA references (initialized with) function FB
// - Function FA stores address of a function FB somewhere
//
// The following cases are treated as dependencies between global objects:
// 1. Global object A is used by a global object B in any way (store,
//    bitcast, phi node, call, etc.): "A" -> "B" edge will be added to the
//    graph;
// 2. function A performs an indirect call of a function with signature S and
//    there is a function B with signature S. "A" -> "B" edge will be added to
//    the graph;
class DependencyGraph {
public:
  using GlobalSet = SmallPtrSet<const GlobalValue *, 16>;

  DependencyGraph(const Module &M) {
    // Group functions by their signature to handle case (2) described above
    DenseMap<const FunctionType *, DependencyGraph::GlobalSet>
        FuncTypeToFuncsMap;
    for (const Function &F : M.functions()) {
      // Kernels can't be called (either directly or indirectly).
      if (F.hasKernelCallingConv())
        continue;

      FuncTypeToFuncsMap[F.getFunctionType()].insert(&F);
    }

    for (const Function &F : M.functions()) {
      // case (1), see comment above the class definition
      for (const Value *U : F.users())
        addUserToGraphRecursively(cast<const User>(U), &F);

```
- EN: Core entities appearing here include DependencyGraph, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 DependencyGraph，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 133-163

```cpp
      // case (2), see comment above the class definition
      for (const Instruction &I : instructions(F)) {
        const CallBase *CB = dyn_cast<CallBase>(&I);
        if (!CB || !CB->isIndirectCall()) // Direct calls were handled above
          continue;

        const FunctionType *Signature = CB->getFunctionType();
        GlobalSet &PotentialCallees = FuncTypeToFuncsMap[Signature];
        Graph[&F].insert(PotentialCallees.begin(), PotentialCallees.end());
      }
    }

    // And every global variable (but their handling is a bit simpler)
    for (const GlobalVariable &GV : M.globals())
      for (const Value *U : GV.users())
        addUserToGraphRecursively(cast<const User>(U), &GV);
  }

  iterator_range<GlobalSet::const_iterator>
  dependencies(const GlobalValue *Val) const {
    auto It = Graph.find(Val);
    return (It == Graph.end())
               ? make_range(EmptySet.begin(), EmptySet.end())
               : make_range(It->second.begin(), It->second.end());
  }

private:
  void addUserToGraphRecursively(const User *Root, const GlobalValue *V) {
    SmallVector<const User *, 8> WorkList;
    WorkList.push_back(Root);

```
- EN: Core entities appearing here include dependencies, addUserToGraphRecursively, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 dependencies, addUserToGraphRecursively，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 164-198

```cpp
    while (!WorkList.empty()) {
      const User *U = WorkList.pop_back_val();
      if (const auto *I = dyn_cast<const Instruction>(U)) {
        const Function *UFunc = I->getFunction();
        Graph[UFunc].insert(V);
      } else if (isa<const Constant>(U)) {
        if (const auto *GV = dyn_cast<const GlobalVariable>(U))
          Graph[GV].insert(V);
        // This could be a global variable or some constant expression (like
        // bitcast or gep). We trace users of this constant further to reach
        // global objects they are used by and add them to the graph.
        for (const User *UU : U->users())
          WorkList.push_back(UU);
      } else {
        llvm_unreachable("Unhandled type of function user");
      }
    }
  }

  DenseMap<const GlobalValue *, GlobalSet> Graph;
  SmallPtrSet<const GlobalValue *, 1> EmptySet;
};

void collectFunctionsAndGlobalVariablesToExtract(
    SetVector<const GlobalValue *> &GVs, const Module &M,
    const EntryPointGroup &ModuleEntryPoints, const DependencyGraph &DG) {
  // We start with module entry points
  for (const Function *F : ModuleEntryPoints.Functions)
    GVs.insert(F);

  // Non-discardable global variables are also include into the initial set
  for (const GlobalVariable &GV : M.globals())
    if (!GV.isDiscardableIfUnused())
      GVs.insert(&GV);

```
- EN: This region continues the SplitModuleByCategory implementation with local helper logic centered on WorkList, User, Instruction, Function.
- CN: 这一段延续了 SplitModuleByCategory 的主体实现，围绕 WorkList, User, Instruction, Function 等局部辅助逻辑展开。

### Lines 199-232

```cpp
  // GVs has SetVector type. This type inserts a value only if it is not yet
  // present there. So, recursion is not expected here.
  size_t Idx = 0;
  while (Idx < GVs.size()) {
    const GlobalValue *Obj = GVs[Idx++];

    for (const GlobalValue *Dep : DG.dependencies(Obj)) {
      if (const auto *Func = dyn_cast<const Function>(Dep)) {
        if (!Func->isDeclaration())
          GVs.insert(Func);
      } else {
        GVs.insert(Dep); // Global variables are added unconditionally
      }
    }
  }
}

ModuleDesc extractSubModule(const Module &M,
                            const SetVector<const GlobalValue *> &GVs,
                            EntryPointGroup &&ModuleEntryPoints) {
  ValueToValueMapTy VMap;
  // Clone definitions only for needed globals. Others will be added as
  // declarations and removed later.
  std::unique_ptr<Module> SubM = CloneModule(
      M, VMap, [&](const GlobalValue *GV) { return GVs.contains(GV); });
  // Replace entry points with cloned ones.
  EntryPointSet NewEPs;
  const EntryPointSet &EPs = ModuleEntryPoints.Functions;
  llvm::for_each(
      EPs, [&](const Function *F) { NewEPs.insert(cast<Function>(VMap[F])); });
  ModuleEntryPoints.Functions = std::move(NewEPs);
  return ModuleDesc{std::move(SubM), std::move(ModuleEntryPoints)};
}

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 233-266

```cpp
// The function produces a copy of input LLVM IR module M with only those
// functions and globals that can be called from entry points that are specified
// in ModuleEntryPoints vector, in addition to the entry point functions.
ModuleDesc extractCallGraph(const Module &M,
                            EntryPointGroup &&ModuleEntryPoints,
                            const DependencyGraph &DG) {
  SetVector<const GlobalValue *> GVs;
  collectFunctionsAndGlobalVariablesToExtract(GVs, M, ModuleEntryPoints, DG);

  ModuleDesc SplitM = extractSubModule(M, GVs, std::move(ModuleEntryPoints));
  LLVM_DEBUG(SplitM.dump());
  return SplitM;
}

using EntryPointGroupVec = SmallVector<EntryPointGroup>;

/// Module Splitter.
/// It gets a module and a collection of entry points groups.
/// Each group specifies subset entry points from input module that should be
/// included in a split module.
class ModuleSplitter {
private:
  std::unique_ptr<Module> M;
  EntryPointGroupVec Groups;
  DependencyGraph DG;

private:
  EntryPointGroup drawEntryPointGroup() {
    assert(Groups.size() > 0 && "Reached end of entry point groups list.");
    EntryPointGroup Group = std::move(Groups.back());
    Groups.pop_back();
    return Group;
  }

```
- EN: Core entities appearing here include ModuleSplitter, drawEntryPointGroup, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as CallGraph.
- CN: 此处出现的核心实体包括 ModuleSplitter, drawEntryPointGroup，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 CallGraph 等分析结果。

### Lines 267-299

```cpp
public:
  ModuleSplitter(std::unique_ptr<Module> Module, EntryPointGroupVec &&GroupVec)
      : M(std::move(Module)), Groups(std::move(GroupVec)), DG(*M) {
    assert(!Groups.empty() && "Entry points groups collection is empty!");
  }

  /// Gets next subsequence of entry points in an input module and provides
  /// split submodule containing these entry points and their dependencies.
  ModuleDesc getNextSplit() {
    return extractCallGraph(*M, drawEntryPointGroup(), DG);
  }

  /// Check that there are still submodules to split.
  bool hasMoreSplits() const { return Groups.size() > 0; }
};

EntryPointGroupVec selectEntryPointGroups(
    const Module &M, function_ref<std::optional<int>(const Function &F)> EPC) {
  // std::map is used here to ensure stable ordering of entry point groups,
  // which is based on their contents, this greatly helps LIT tests
  // Note: EPC is allowed to return big identifiers. Therefore, we use
  // std::map + SmallVector approach here.
  std::map<int, EntryPointSet> EntryPointsMap;

  for (const auto &F : M.functions())
    if (std::optional<int> Category = EPC(F); Category)
      EntryPointsMap[*Category].insert(&F);

  EntryPointGroupVec Groups;
  Groups.reserve(EntryPointsMap.size());
  for (auto &[Key, EntryPoints] : EntryPointsMap)
    Groups.emplace_back(Key, std::move(EntryPoints));

```
- EN: Core entities appearing here include M, getNextSplit, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as CallGraph.
- CN: 此处出现的核心实体包括 M, getNextSplit，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 CallGraph 等分析结果。

### Lines 300-317

```cpp
  return Groups;
}

} // namespace

Error llvm::splitModuleTransitiveFromEntryPoints(
    std::unique_ptr<Module> M,
    function_ref<std::optional<int>(const Function &F)> EntryPointCategorizer,
    function_ref<Error(std::unique_ptr<Module> Part)> Callback) {
  EntryPointGroupVec Groups = selectEntryPointGroups(*M, EntryPointCategorizer);
  ModuleSplitter Splitter(std::move(M), std::move(Groups));
  while (Splitter.hasMoreSplits()) {
    ModuleDesc MD = Splitter.getNextSplit();
    if (Error E = Callback(MD.releaseModule()))
      return E;
  }
  return Error::success();
}
```
- EN: This region continues the SplitModuleByCategory implementation with local helper logic centered on Groups, Error, Module, Function.
- CN: 这一段延续了 SplitModuleByCategory 的主体实现，围绕 Groups, Error, Module, Function 等局部辅助逻辑展开。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `EntryPointGroup, ModuleDesc, DependencyGraph, ModuleSplitter, dump, M, releaseModule, dependencies` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`EntryPointGroup, ModuleDesc, DependencyGraph, ModuleSplitter, dump, M, releaseModule, dependencies` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `CallGraph` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `CallGraph` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: IR/transform headers: `llvm/IR/Function.h`, `llvm/IR/InstIterator.h`, `llvm/IR/Instructions.h`, `llvm/IR/Module.h`, `llvm/Transforms/Utils/SplitModuleByCategory.h`, `llvm/Transforms/Utils/Cloning.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Function.h`, `llvm/IR/InstIterator.h`, `llvm/IR/Instructions.h`, `llvm/IR/Module.h`, `llvm/Transforms/Utils/SplitModuleByCategory.h`, `llvm/Transforms/Utils/Cloning.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Debug.h`, `llvm/ADT/SetVector.h`, `llvm/ADT/SmallPtrSet.h`, `llvm/ADT/StringExtras.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Debug.h`, `llvm/ADT/SetVector.h`, `llvm/ADT/SmallPtrSet.h`, `llvm/ADT/StringExtras.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `map`, `utility` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`map`, `utility` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `CallGraph` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`CallGraph` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
