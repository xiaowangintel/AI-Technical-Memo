# Layer.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/ExecutionEngine/Orc/Layer.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: This file implements Layer interfaces.
  - **CN**: 实现 ORC JIT 基础设施，例如 ExecutionSession、JITDylib、物化流程、符号查找以及执行辅助组件。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-------------------- Layer.cpp - Layer interfaces --------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及文件级说明。

### Lines 8-16
```cpp

#include "llvm/ExecutionEngine/Orc/Layer.h"

#include "llvm/ExecutionEngine/Orc/DebugUtils.h"
#include "llvm/ExecutionEngine/Orc/ExecutionUtils.h"
#include "llvm/ExecutionEngine/Orc/ObjectFileInterface.h"
#include "llvm/IR/Constants.h"
#include "llvm/Support/Debug.h"

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/ExecutionEngine/Orc/Layer.h`, `llvm/ExecutionEngine/Orc/DebugUtils.h`, `llvm/ExecutionEngine/Orc/ExecutionUtils.h`, `llvm/ExecutionEngine/Orc/ObjectFileInterface.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/ExecutionEngine/Orc/Layer.h`, `llvm/ExecutionEngine/Orc/DebugUtils.h`, `llvm/ExecutionEngine/Orc/ExecutionUtils.h`, `llvm/ExecutionEngine/Orc/ObjectFileInterface.h`。

### Lines 17-23
```cpp
#define DEBUG_TYPE "orc"

namespace llvm {
namespace orc {

IRLayer::~IRLayer() = default;

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 24-31
```cpp
Error IRLayer::add(ResourceTrackerSP RT, ThreadSafeModule TSM) {
  assert(RT && "RT can not be null");
  auto &JD = RT->getJITDylib();
  return JD.define(std::make_unique<BasicIRLayerMaterializationUnit>(
                       *this, *getManglingOptions(), std::move(TSM)),
                   std::move(RT));
}

```
- **EN**: Implements logic around `add`, `assert`, `getJITDylib`, `define`, and 2 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `add`, `assert`, `getJITDylib`, `define`, and 2 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 32-38
```cpp
IRMaterializationUnit::IRMaterializationUnit(
    ExecutionSession &ES, const IRSymbolMapper::ManglingOptions &MO,
    ThreadSafeModule TSM)
    : MaterializationUnit(Interface()), TSM(std::move(TSM)) {

  assert(this->TSM && "Module must not be null");

```
- **EN**: Implements logic around `IRMaterializationUnit`, `MaterializationUnit`, `assert`; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `IRMaterializationUnit`, `MaterializationUnit`, `assert` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 39-47
```cpp
  MangleAndInterner Mangle(ES, this->TSM.getModuleUnlocked()->getDataLayout());
  this->TSM.withModuleDo([&](Module &M) {
    for (auto &G : M.global_values()) {
      // Skip globals that don't generate symbols.

      if (!G.hasName() || G.isDeclaration() || G.hasLocalLinkage() ||
          G.hasAvailableExternallyLinkage() || G.hasAppendingLinkage())
        continue;

```
- **EN**: Implements logic around `Mangle`, `withModuleDo`, `hasAvailableExternallyLinkage`; this block drives emission, layout, or binary encoding behavior.
- **CN**: 围绕 `Mangle`, `withModuleDo`, `hasAvailableExternallyLinkage` 实现具体逻辑；这一段驱动输出、布局或二进制编码行为。

### Lines 48-54
```cpp
      // thread locals generate different symbols depending on whether or not
      // emulated TLS is enabled.
      if (G.isThreadLocal() && MO.EmulatedTLS) {
        auto &GV = cast<GlobalVariable>(G);

        auto Flags = JITSymbolFlags::fromGlobalValue(GV);

```
- **EN**: Implements logic around `cast<GlobalVariable>`, `fromGlobalValue`.
- **CN**: 围绕 `cast<GlobalVariable>`, `fromGlobalValue` 实现具体逻辑。

### Lines 55-63
```cpp
        auto EmuTLSV = Mangle(("__emutls_v." + GV.getName()).str());
        SymbolFlags[EmuTLSV] = Flags;
        SymbolToDefinition[EmuTLSV] = &GV;

        // If this GV has a non-zero initializer we'll need to emit an
        // __emutls.t symbol too.
        if (GV.hasInitializer()) {
          const auto *InitVal = GV.getInitializer();

```
- **EN**: Implements logic around `Mangle`, `getInitializer`.
- **CN**: 围绕 `Mangle`, `getInitializer` 实现具体逻辑。

### Lines 64-70
```cpp
          // Skip zero-initializers.
          if (isa<ConstantAggregateZero>(InitVal))
            continue;
          const auto *InitIntValue = dyn_cast<ConstantInt>(InitVal);
          if (InitIntValue && InitIntValue->isZero())
            continue;

```
- **EN**: Implements logic around `dyn_cast<ConstantInt>`.
- **CN**: 围绕 `dyn_cast<ConstantInt>` 实现具体逻辑。

### Lines 71-84
```cpp
          auto EmuTLST = Mangle(("__emutls_t." + GV.getName()).str());
          SymbolFlags[EmuTLST] = Flags;
        }
        continue;
      }

      // Otherwise we just need a normal linker mangling.
      auto MangledName = Mangle(G.getName());
      auto &Flags = SymbolFlags[MangledName];
      Flags = JITSymbolFlags::fromGlobalValue(G);
      if (G.getComdat() &&
          G.getComdat()->getSelectionKind() != Comdat::NoDeduplicate)
        Flags |= JITSymbolFlags::Weak;
      SymbolToDefinition[MangledName] = &G;
```
- **EN**: Implements logic around `Mangle`, `fromGlobalValue`, `getComdat`.
- **CN**: 围绕 `Mangle`, `fromGlobalValue`, `getComdat` 实现具体逻辑。

### Lines 85-97
```cpp
    }

    // If we need an init symbol for this module then create one.
    if (!getStaticInitGVs(M).empty()) {
      size_t Counter = 0;

      do {
        std::string InitSymbolName;
        raw_string_ostream(InitSymbolName)
            << "$." << M.getModuleIdentifier() << ".__inits." << Counter++;
        InitSymbol = ES.intern(InitSymbolName);
      } while (SymbolFlags.count(InitSymbol));

```
- **EN**: Implements logic around `raw_string_ostream`, `getModuleIdentifier`, `intern`.
- **CN**: 围绕 `raw_string_ostream`, `getModuleIdentifier`, `intern` 实现具体逻辑。

### Lines 98-108
```cpp
      SymbolFlags[InitSymbol] = JITSymbolFlags::MaterializationSideEffectsOnly;
    }
  });
}

IRMaterializationUnit::IRMaterializationUnit(
    ThreadSafeModule TSM, Interface I,
    SymbolNameToDefinitionMap SymbolToDefinition)
    : MaterializationUnit(std::move(I)), TSM(std::move(TSM)),
      SymbolToDefinition(std::move(SymbolToDefinition)) {}

```
- **EN**: Implements logic around `IRMaterializationUnit`, `MaterializationUnit`, `SymbolToDefinition`; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `IRMaterializationUnit`, `MaterializationUnit`, `SymbolToDefinition` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 109-115
```cpp
StringRef IRMaterializationUnit::getName() const {
  if (TSM)
    return TSM.withModuleDo(
        [](const Module &M) -> StringRef { return M.getModuleIdentifier(); });
  return "<null module>";
}

```
- **EN**: Implements logic around `getName`, `withModuleDo`, `getModuleIdentifier`; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `getName`, `withModuleDo`, `getModuleIdentifier` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 116-122
```cpp
void IRMaterializationUnit::discard(const JITDylib &JD,
                                    const SymbolStringPtr &Name) {
  LLVM_DEBUG(JD.getExecutionSession().runSessionLocked([&]() {
    dbgs() << "In " << JD.getName() << " discarding " << *Name << " from MU@"
           << this << " (" << getName() << ")\n";
  }););

```
- **EN**: Implements logic around `discard`, `dbgs`, `getName`; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `discard`, `dbgs`, `getName` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 123-135
```cpp
  auto I = SymbolToDefinition.find(Name);
  assert(I != SymbolToDefinition.end() &&
         "Symbol not provided by this MU, or previously discarded");
  assert(!I->second->isDeclaration() &&
         "Discard should only apply to definitions");
  I->second->setLinkage(GlobalValue::AvailableExternallyLinkage);
  // According to the IR verifier, "Declaration[s] may not be in a Comdat!"
  // Remove it, if this is a GlobalObject.
  if (auto *GO = dyn_cast<GlobalObject>(I->second))
    GO->setComdat(nullptr);
  SymbolToDefinition.erase(I);
}

```
- **EN**: Implements logic around `find`, `assert`, `setLinkage`, `setComdat`, and 1 more symbols.
- **CN**: 围绕 `find`, `assert`, `setLinkage`, `setComdat`, and 1 more symbols 实现具体逻辑。

### Lines 136-143
```cpp
BasicIRLayerMaterializationUnit::BasicIRLayerMaterializationUnit(
    IRLayer &L, const IRSymbolMapper::ManglingOptions &MO, ThreadSafeModule TSM)
    : IRMaterializationUnit(L.getExecutionSession(), MO, std::move(TSM)), L(L) {
}

void BasicIRLayerMaterializationUnit::materialize(
    std::unique_ptr<MaterializationResponsibility> R) {

```
- **EN**: Implements logic around `BasicIRLayerMaterializationUnit`, `IRMaterializationUnit`, `materialize`; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `BasicIRLayerMaterializationUnit`, `IRMaterializationUnit`, `materialize` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 144-151
```cpp
  // Throw away the SymbolToDefinition map: it's not usable after we hand
  // off the module.
  SymbolToDefinition.clear();

  // If cloneToNewContextOnEmit is set, clone the module now.
  if (L.getCloneToNewContextOnEmit())
    TSM = cloneToNewContext(TSM);

```
- **EN**: Implements logic around `clear`, `cloneToNewContext`.
- **CN**: 围绕 `clear`, `cloneToNewContext` 实现具体逻辑。

### Lines 152-164
```cpp
#ifndef NDEBUG
  auto &ES = R->getTargetJITDylib().getExecutionSession();
  auto &N = R->getTargetJITDylib().getName();
#endif // NDEBUG

  LLVM_DEBUG(ES.runSessionLocked(
      [&]() { dbgs() << "Emitting, for " << N << ", " << *this << "\n"; }););
  L.emit(std::move(R), std::move(TSM));
  LLVM_DEBUG(ES.runSessionLocked([&]() {
    dbgs() << "Finished emitting, for " << N << ", " << *this << "\n";
  }););
}

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 165-178
```cpp
char ObjectLayer::ID;

ObjectLayer::ObjectLayer(ExecutionSession &ES) : ES(ES) {}

ObjectLayer::~ObjectLayer() = default;

Error ObjectLayer::add(ResourceTrackerSP RT, std::unique_ptr<MemoryBuffer> O,
                       MaterializationUnit::Interface I) {
  assert(RT && "RT can not be null");
  auto &JD = RT->getJITDylib();
  return JD.define(std::make_unique<BasicObjectLayerMaterializationUnit>(
                       *this, std::move(O), std::move(I)),
                   std::move(RT));
}
```
- **EN**: Implements logic around `ObjectLayer`, `~ObjectLayer`, `add`, `assert`, and 3 more symbols; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `ObjectLayer`, `~ObjectLayer`, `add`, `assert`, and 3 more symbols 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 179-186
```cpp

Error ObjectLayer::add(ResourceTrackerSP RT, std::unique_ptr<MemoryBuffer> O) {
  auto I = getObjectFileInterface(getExecutionSession(), O->getMemBufferRef());
  if (!I)
    return I.takeError();
  return add(std::move(RT), std::move(O), std::move(*I));
}

```
- **EN**: Implements logic around `add`, `getObjectFileInterface`, `takeError`; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `add`, `getObjectFileInterface`, `takeError` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 187-193
```cpp
Error ObjectLayer::add(JITDylib &JD, std::unique_ptr<MemoryBuffer> O) {
  auto I = getObjectFileInterface(getExecutionSession(), O->getMemBufferRef());
  if (!I)
    return I.takeError();
  return add(JD, std::move(O), std::move(*I));
}

```
- **EN**: Implements logic around `add`, `getObjectFileInterface`, `takeError`; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `add`, `getObjectFileInterface`, `takeError` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 194-200
```cpp
Expected<std::unique_ptr<BasicObjectLayerMaterializationUnit>>
BasicObjectLayerMaterializationUnit::Create(ObjectLayer &L,
                                            std::unique_ptr<MemoryBuffer> O) {

  auto ObjInterface =
      getObjectFileInterface(L.getExecutionSession(), O->getMemBufferRef());

```
- **EN**: Implements logic around `Create`, `getObjectFileInterface`; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `Create`, `getObjectFileInterface` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 201-207
```cpp
  if (!ObjInterface)
    return ObjInterface.takeError();

  return std::make_unique<BasicObjectLayerMaterializationUnit>(
      L, std::move(O), std::move(*ObjInterface));
}

```
- **EN**: Implements logic around `takeError`, `make_unique<BasicObjectLayerMaterializationUnit>`, `move`; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `takeError`, `make_unique<BasicObjectLayerMaterializationUnit>`, `move` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 208-217
```cpp
BasicObjectLayerMaterializationUnit::BasicObjectLayerMaterializationUnit(
    ObjectLayer &L, std::unique_ptr<MemoryBuffer> O, Interface I)
    : MaterializationUnit(std::move(I)), L(L), O(std::move(O)) {}

StringRef BasicObjectLayerMaterializationUnit::getName() const {
  if (O)
    return O->getBufferIdentifier();
  return "<null object>";
}

```
- **EN**: Implements logic around `BasicObjectLayerMaterializationUnit`, `MaterializationUnit`, `getName`, `getBufferIdentifier`; this block coordinates ORC symbol lookup or materialization state; returns subsystem-specific computed results.
- **CN**: 围绕 `BasicObjectLayerMaterializationUnit`, `MaterializationUnit`, `getName`, `getBufferIdentifier` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态，返回子系统相关的计算结果。

### Lines 218-228
```cpp
void BasicObjectLayerMaterializationUnit::materialize(
    std::unique_ptr<MaterializationResponsibility> R) {
  L.emit(std::move(R), std::move(O));
}

void BasicObjectLayerMaterializationUnit::discard(const JITDylib &JD,
                                                  const SymbolStringPtr &Name) {
  // This is a no-op for object files: Having removed 'Name' from SymbolFlags
  // the symbol will be dead-stripped by the JIT linker.
}

```
- **EN**: Implements logic around `materialize`, `emit`, `discard`; this block coordinates ORC symbol lookup or materialization state.
- **CN**: 围绕 `materialize`, `emit`, `discard` 实现具体逻辑；这一段协调 ORC 符号查找或物化状态。

### Lines 229-230
```cpp
} // End namespace orc.
} // End namespace llvm.
```
- **EN**: Introduces declarations for `orc`, `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `orc`, `llvm` 等声明，定义本文件后续使用的数据结构或接口。

## Key Concepts / 关键概念

- **ORC JIT / ORC JIT**:
  - **EN**: Coordinates JITDylibs, symbol materialization, execution sessions, and asynchronous compilation flows
  - **CN**: 协调 JITDylib、符号物化、ExecutionSession 与异步编译流程
- **Symbol materialization / 符号物化**:
  - **EN**: Defers code or data generation until symbols are requested
  - **CN**: 将代码或数据的生成延迟到符号真正被请求时
- **Debug/unwind metadata / 调试与展开元数据**:
  - **EN**: Emits or manages metadata needed for debugging and stack unwinding
  - **CN**: 输出或管理调试与栈展开所需的元数据

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `llvm/ExecutionEngine/Orc/Layer.h`, `llvm/ExecutionEngine/Orc/DebugUtils.h`, `llvm/ExecutionEngine/Orc/ExecutionUtils.h`, `llvm/ExecutionEngine/Orc/ObjectFileInterface.h`, `llvm/IR/Constants.h`, `llvm/Support/Debug.h`
- **LLVM subsystems / LLVM 子系统**: ExecutionEngine, IR, Support
