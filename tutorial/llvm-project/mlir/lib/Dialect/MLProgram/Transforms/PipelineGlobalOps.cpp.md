# PipelineGlobalOps.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Dialect/MLProgram/Transforms/PipelineGlobalOps.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**:
  - **EN**: Implements MLProgram dialect support for rewrite patterns, passes, and IR-to-IR transformation logic, centered on `PipelineGlobalOps`.
  - **CN**: 实现 MLProgram 方言中围绕 `PipelineGlobalOps` 的重写模式、Pass 以及 IR 到 IR 的变换逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- PipelineGlobalOpsPass.cpp - Pipeline Global Ops Pass ---------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM/MLIR file banner, license notice, and file-level summary.
- **CN**: 包含 LLVM/MLIR 标准文件头、许可证声明以及文件级摘要。

### Lines 8-14
```cpp

#include "mlir/Dialect/MLProgram/Transforms/Passes.h"

#include "mlir/Dialect/MLProgram/IR/MLProgram.h"
#include "mlir/IR/BuiltinOps.h"

namespace mlir {
```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/MLProgram/Transforms/Passes.h`, `mlir/Dialect/MLProgram/IR/MLProgram.h`, `mlir/IR/BuiltinOps.h`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/MLProgram/Transforms/Passes.h`, `mlir/Dialect/MLProgram/IR/MLProgram.h`, `mlir/IR/BuiltinOps.h`。

### Lines 15-21
```cpp
namespace ml_program {
#define GEN_PASS_DEF_MLPROGRAMPIPELINEGLOBALSPASS
#include "mlir/Dialect/MLProgram/Transforms/Passes.h.inc"

namespace {

class MLProgramPipelineGlobals
```
- **EN**: Pulls in the headers needed by this translation unit, including `mlir/Dialect/MLProgram/Transforms/Passes.h.inc`.
- **CN**: 引入该编译单元所需的头文件，其中包括 `mlir/Dialect/MLProgram/Transforms/Passes.h.inc`。

### Lines 22-28
```cpp
    : public impl::MLProgramPipelineGlobalsPassBase<MLProgramPipelineGlobals> {
public:
  void runOnOperation() override;

private:
  LogicalResult buildGlobalMap(ModuleOp op);

```
- **EN**: Implements logic around `runOnOperation`, `buildGlobalMap`; this block packages logic as an MLIR pass or pass helper.
- **CN**: 围绕 `runOnOperation`, `buildGlobalMap` 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或 Pass 辅助组件。

### Lines 29-35
```cpp
  void processBlock(Block &block, llvm::DenseSet<SymbolRefAttr> &symbolLoad,
                    llvm::DenseSet<SymbolRefAttr> &symbolStore);

  llvm::DenseMap<SymbolRefAttr, llvm::DenseSet<SymbolRefAttr>> loadSymbolsMap;
  llvm::DenseMap<SymbolRefAttr, llvm::DenseSet<SymbolRefAttr>> storeSymbolsMap;
};

```
- **EN**: Implements logic around `processBlock`.
- **CN**: 围绕 `processBlock` 实现具体逻辑。

### Lines 36-45
```cpp
// Traverses upwards searching for the operation mapped by the symbol.
static Operation *getFromSymbol(Operation *baseOp, SymbolRefAttr symbol) {
  for (auto *op = baseOp; op; op = op->getParentOp()) {
    auto *lookup = SymbolTable::lookupNearestSymbolFrom(op, symbol);
    if (lookup)
      return lookup;
  }
  return nullptr;
}

```
- **EN**: Implements logic around `getFromSymbol`, `getParentOp`, `lookupNearestSymbolFrom`.
- **CN**: 围绕 `getFromSymbol`, `getParentOp`, `lookupNearestSymbolFrom` 实现具体逻辑。

### Lines 46-57
```cpp
// Builds map from a symbol to MLProgram global symbols loaded or stored
// during processing.
LogicalResult MLProgramPipelineGlobals::buildGlobalMap(ModuleOp module) {
  llvm::DenseMap<SymbolRefAttr, Operation *> callableMap;
  auto res = module->walk([&](Operation *op) {
    if (auto caller = mlir::dyn_cast<CallOpInterface>(op)) {
      auto callable = caller.getCallableForCallee();
      // For now we do not know how to handle Value based tracing, so fail.
      if (mlir::isa<Value>(callable)) {
        return WalkResult::interrupt();
      }

```
- **EN**: Implements logic around `buildGlobalMap`, `walk`, `getCallableForCallee`, `interrupt`.
- **CN**: 围绕 `buildGlobalMap`, `walk`, `getCallableForCallee`, `interrupt` 实现具体逻辑。

### Lines 58-67
```cpp
      auto symbol = mlir::dyn_cast<SymbolRefAttr>(callable);
      auto *func = getFromSymbol(op, symbol);
      // If the callee cannot be resolved, we cannot safely analyze the IR.
      if (!func)
        return WalkResult::interrupt();
      callableMap[symbol] = func;
    }
    return WalkResult::advance();
  });

```
- **EN**: Implements logic around `getFromSymbol`, `interrupt`, `advance`.
- **CN**: 围绕 `getFromSymbol`, `interrupt`, `advance` 实现具体逻辑。

### Lines 68-79
```cpp
  if (res.wasInterrupted()) {
    return failure();
  }

  // First grab all symbols loaded or stored by each function. This
  // will not handle calls initially.
  llvm::DenseMap<SymbolRefAttr, llvm::DenseSet<SymbolRefAttr>> opLoadSymbols;
  llvm::DenseMap<SymbolRefAttr, llvm::DenseSet<SymbolRefAttr>> opStoreSymbols;
  for (auto callable : callableMap) {
    llvm::DenseSet<SymbolRefAttr> loadSymbols;
    llvm::DenseSet<SymbolRefAttr> storeSymbols;

```
- **EN**: Implements logic around `wasInterrupted`, `failure`.
- **CN**: 围绕 `wasInterrupted`, `failure` 实现具体逻辑。

### Lines 80-89
```cpp
    callable.getSecond()->walk(
        [&](GlobalLoadOp op) { loadSymbols.insert(op.getGlobal()); });

    callable.getSecond()->walk(
        [&](GlobalStoreOp op) { storeSymbols.insert(op.getGlobal()); });

    opLoadSymbols[callable.getFirst()] = std::move(loadSymbols);
    opStoreSymbols[callable.getFirst()] = std::move(storeSymbols);
  }

```
- **EN**: Implements logic around `getSecond`, `insert`, `getFirst`.
- **CN**: 围绕 `getSecond`, `insert`, `getFirst` 实现具体逻辑。

### Lines 90-99
```cpp
  // For each callable function we find each global loaded/stored within the
  // function or a nested called function. This includes recursion checking to
  // avoid infinitely recursing.
  for (auto callable : callableMap) {
    SymbolRefAttr thisSymbol = llvm::dyn_cast<SymbolRefAttr>(callable.first);
    llvm::SmallVector<SymbolRefAttr> work = {thisSymbol};
    llvm::DenseSet<SymbolRefAttr> visited = {thisSymbol};
    llvm::DenseSet<SymbolRefAttr> loadSymbols;
    llvm::DenseSet<SymbolRefAttr> storeSymbols;

```
- **EN**: Contains supporting implementation details for the surrounding MLIR dialect component.
- **CN**: 包含周边 MLIR 方言组件所需的辅助实现细节。

### Lines 100-111
```cpp
    for (size_t i = 0; i < work.size(); ++i) {
      // Defensive: symbols in `work` should always be in `callableMap` since
      // buildGlobalMap interrupted on any unresolvable callee, but use find to
      // avoid inserting null entries via operator[].
      auto it = callableMap.find(work[i]);
      assert(it != callableMap.end() && "Expected callable in callableMap");
      it->second->walk([&](CallOpInterface call) {
        auto symbol = cast<SymbolRefAttr>(call.getCallableForCallee());
        if (visited.insert(symbol).second)
          work.push_back(symbol);
      });

```
- **EN**: Implements logic around `size`, `find`, `assert`, `walk`, and 3 more symbols.
- **CN**: 围绕 `size`, `find`, `assert`, `walk`, and 3 more symbols 实现具体逻辑。

### Lines 112-120
```cpp
      loadSymbols.insert_range(opLoadSymbols[work[i]]);

      storeSymbols.insert_range(opStoreSymbols[work[i]]);
    }

    loadSymbolsMap[thisSymbol] = std::move(loadSymbols);
    storeSymbolsMap[thisSymbol] = std::move(storeSymbols);
  }

```
- **EN**: Implements logic around `insert_range`, `move`.
- **CN**: 围绕 `insert_range`, `move` 实现具体逻辑。

### Lines 121-129
```cpp
  return success();
}

// Process each operation in the block deleting unneeded loads / stores,
// recursing on subblocks and checking function calls.
void MLProgramPipelineGlobals::processBlock(
    Block &block, llvm::DenseSet<SymbolRefAttr> &symbolLoad,
    llvm::DenseSet<SymbolRefAttr> &symbolStore) {

```
- **EN**: Implements logic around `success`, `processBlock`.
- **CN**: 围绕 `success`, `processBlock` 实现具体逻辑。

### Lines 130-143
```cpp
  llvm::DenseMap<SymbolRefAttr, Value> previousLoads;
  llvm::DenseMap<SymbolRefAttr, Operation *> previousStores;
  llvm::SmallVector<Operation *> toDelete;
  for (auto &op : block) {
    // If this is a global load, remap to a previous value if known
    // and delete this load. Remember that this value is the currently
    // known load.
    if (auto load = mlir::dyn_cast<GlobalLoadOp>(op)) {
      auto ref = load.getGlobal();
      symbolLoad.insert(ref);
      if (previousLoads.contains(ref)) {
        toDelete.push_back(&op);
        load.getResult().replaceAllUsesWith(previousLoads[ref]);
      } else {
```
- **EN**: Implements logic around `getGlobal`, `insert`, `contains`, `push_back`, and 1 more symbols.
- **CN**: 围绕 `getGlobal`, `insert`, `contains`, `push_back`, and 1 more symbols 实现具体逻辑。

### Lines 144-157
```cpp
        previousLoads[ref] = load.getResult();
      }
      continue;
    }

    // Delete a previous store if it exists and is not needed, update
    // the most recent known value for this global ref.
    if (auto store = mlir::dyn_cast<GlobalStoreOp>(op)) {
      auto ref = store.getGlobal();
      symbolStore.insert(ref);
      auto it = previousStores.find(ref);
      if (it != previousStores.end()) {
        toDelete.push_back(it->getSecond());
      }
```
- **EN**: Implements logic around `getResult`, `getGlobal`, `insert`, `find`, and 2 more symbols.
- **CN**: 围绕 `getResult`, `getGlobal`, `insert`, `find`, and 2 more symbols 实现具体逻辑。

### Lines 158-171
```cpp

      previousLoads[ref] = store.getValue();
      previousStores[ref] = &op;
      continue;
    }

    // If a function is called, clear known values for loads/stores used by
    // the function or its sub-functions.
    if (auto call = mlir::dyn_cast<CallOpInterface>(op)) {
      auto loadSymbols =
          loadSymbolsMap[dyn_cast<SymbolRefAttr>(call.getCallableForCallee())];
      auto storeSymbols =
          storeSymbolsMap[dyn_cast<SymbolRefAttr>(call.getCallableForCallee())];

```
- **EN**: Implements logic around `getValue`, `getCallableForCallee`.
- **CN**: 围绕 `getValue`, `getCallableForCallee` 实现具体逻辑。

### Lines 172-182
```cpp
      for (auto sym : loadSymbols) {
        previousStores.erase(sym);
      }

      for (auto sym : storeSymbols) {
        previousLoads.erase(sym);
        previousStores.erase(sym);
      }
      continue;
    }

```
- **EN**: Implements logic around `erase`.
- **CN**: 围绕 `erase` 实现具体逻辑。

### Lines 183-192
```cpp
    // If the op has sub-regions, recurse inside. We make no guarantees whether
    // the recursion occurs.
    llvm::DenseSet<SymbolRefAttr> opSymbolLoad;
    llvm::DenseSet<SymbolRefAttr> opSymbolStore;
    for (auto &region : op.getRegions()) {
      for (auto &block : region) {
        processBlock(block, opSymbolLoad, opSymbolStore);
      }
    }

```
- **EN**: Implements logic around `getRegions`, `processBlock`.
- **CN**: 围绕 `getRegions`, `processBlock` 实现具体逻辑。

### Lines 193-205
```cpp
    // Update current state from the subblock.
    for (auto change : opSymbolLoad) {
      symbolLoad.insert(change);
      previousStores.erase(change);
    }

    for (auto change : opSymbolStore) {
      symbolStore.insert(change);
      previousLoads.erase(change);
      previousStores.erase(change);
    }
  }

```
- **EN**: Implements logic around `insert`, `erase`.
- **CN**: 围绕 `insert`, `erase` 实现具体逻辑。

### Lines 206-216
```cpp
  for (auto *op : toDelete) {
    op->erase();
  }
}

void MLProgramPipelineGlobals::runOnOperation() {
  auto targetOp = getOperation();
  if (failed(buildGlobalMap(targetOp))) {
    return;
  }

```
- **EN**: Implements logic around `erase`, `runOnOperation`, `getOperation`, `failed`; this block packages logic as an MLIR pass or pass helper.
- **CN**: 围绕 `erase`, `runOnOperation`, `getOperation`, `failed` 实现具体逻辑；该代码块将逻辑组织为 MLIR Pass 或 Pass 辅助组件。

### Lines 217-227
```cpp
  for (auto &funcOp : *targetOp.getBody()) {
    for (auto &region : funcOp.getRegions()) {
      for (auto &block : region.getBlocks()) {
        llvm::DenseSet<SymbolRefAttr> symbolsLoaded;
        llvm::DenseSet<SymbolRefAttr> symbolsStored;
        processBlock(block, symbolsLoaded, symbolsStored);
      }
    }
  }
}

```
- **EN**: Implements logic around `getBody`, `getRegions`, `getBlocks`, `processBlock`.
- **CN**: 围绕 `getBody`, `getRegions`, `getBlocks`, `processBlock` 实现具体逻辑。

### Lines 228-231
```cpp
} // namespace

} // namespace ml_program
} // namespace mlir
```
- **EN**: Introduces declarations for `ml_program`, `mlir`, establishing the dialect-level types or pattern records used later.
- **CN**: 引入 `ml_program`, `mlir` 等声明，建立后续使用的方言级类型或模式记录。

## Key Concepts / 关键概念

- **Rewrite and lowering / 重写与 lowering**:
  - **EN**: Uses MLIR pattern infrastructure or passes to canonicalize, legalize, or lower operations.
  - **CN**: 使用 MLIR 模式基础设施或 Pass 对操作进行规范化、合法化或 lowering。
- **Pass infrastructure / Pass 基础设施**:
  - **EN**: Wraps transformations as reusable passes that can be scheduled in MLIR pipelines.
  - **CN**: 将变换封装为可在 MLIR 流水线中调度的可复用 Pass。
- **Interface-based extensibility / 基于接口的可扩展性**:
  - **EN**: Models reusable capabilities that can be queried across dialect boundaries.
  - **CN**: 建模可跨方言查询的可复用能力。
- **Structured IR coordination / 结构化 IR 协同**:
  - **EN**: Interacts with structured MLIR dialects that model loops, tensors, memory, and vector semantics.
  - **CN**: 与建模循环、张量、内存和向量语义的结构化 MLIR 方言交互。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM/local includes / 直接的 MLIR/LLVM/本地包含**: `mlir/Dialect/MLProgram/Transforms/Passes.h`, `mlir/Dialect/MLProgram/IR/MLProgram.h`, `mlir/IR/BuiltinOps.h`, `mlir/Dialect/MLProgram/Transforms/Passes.h.inc`
- **Subsystem categories / 子系统类别**: dialect-specific operation, type, attribute, or transform declarations / 方言相关的操作、类型、属性或变换声明 (3), MLIR core IR types and infrastructural utilities / MLIR 核心 IR 类型与基础设施工具 (1)
